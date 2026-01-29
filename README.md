<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>五韻發音儀 - 台灣聲調教學</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>
    <script src="https://unpkg.com/ml5@latest/dist/ml5.min.js"></script>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; --accent: #38bdf8; --red: #fb7185; }
        body { font-family: 'PingFang TC', sans-serif; background: var(--bg); color: #fff; margin: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; overflow: hidden; }
        
        .header { margin-bottom: 20px; }
        .mic-status { position: absolute; top: 20px; right: 20px; display: flex; align-items: center; gap: 10px; font-size: 0.9em; }
        .mic-dot { width: 12px; height: 12px; border-radius: 50%; background: #444; }
        .mic-on { background: #22c55e; box-shadow: 0 0 10px #22c55e; animation: blink 1.5s infinite; }

        .btn-group { display: flex; gap: 15px; margin-bottom: 20px; }
        button { padding: 12px 25px; border: 2px solid #334155; background: var(--card); color: #cbd5e1; border-radius: 50px; cursor: pointer; transition: 0.3s; font-size: 1.1em; }
        button.active { border-color: var(--accent); color: var(--accent); box-shadow: 0 0 15px rgba(56, 189, 248, 0.3); }
        
        #canvas-wrap { border-radius: 20px; box-shadow: 0 20px 50px rgba(0,0,0,0.5); border: 1px solid #334155; position: relative; }
        .hint-text { margin-top: 15px; color: #64748b; font-size: 0.9em; }
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }
    </style>
</head>
<body>

    <div class="mic-status">
        <div id="micIcon" class="mic-dot"></div>
        <span id="micText">麥克風未開啟</span>
    </div>

    <div class="header">
        <h1>五韻· ˇˋˊ</h1>
    </div>
    
    <div class="btn-group">
        <button onclick="setTone(1)" id="t1">一聲</button>
        <button onclick="setTone(2)" id="t2">二聲</button>
        <button onclick="setTone(3)" id="t3">三聲</button>
        <button onclick="setTone(4)" id="t4">四聲</button>
    </div>

    <div id="canvas-wrap"></div>
    <div class="hint-text">請按「空白鍵」校準你的音域基準點</div>

    <script>
        const MODELS = {
            1: [5, 5, 5, 5, 5],
            2: [3, 3.5, 4, 4.5, 5],
            3: [2, 1, 1, 2, 4],
            4: [5, 4, 3, 2, 1]
        };

        let currentTone = 1;
        let pitch;
        let mic;
        let points = [];
        let baseLog = null;
        let calibrated = false;

        function setup() {
            let cnv = createCanvas(850, 480);
            cnv.parent('canvas-wrap');
            mic = new p5.AudioIn();
            mic.start(() => {
                document.getElementById('micIcon').classList.add('mic-on');
                document.getElementById('micText').innerText = "麥克風監聽中";
                pitch = ml5.pitchDetection('https://cdn.jsdelivr.net/gh/ml5js/ml5-data-and-models/models/pitch-detection/crepe/', getAudioContext(), mic.stream, () => {});
            });
            setTone(1);
        }

        function setTone(t) {
            currentTone = t;
            document.querySelectorAll('button').forEach(b => b.classList.remove('active'));
            document.getElementById('t' + t).classList.add('active');
        }

        function draw() {
            background(30, 41, 59);
            drawZones();
            drawGuidePath();

            if (pitch && calibrated) {
                pitch.getPitch((err, freq) => {
                    if (freq && freq > 50) {
                        let logF = Math.log2(freq);
                        let val = map(logF, baseLog - 0.4, baseLog + 0.4, 1, 5);
                        points.push(val);
                        if (points.length > 60) points.shift();
                    } else {
                        points.push(null);
                        if (points.length > 60) points.shift();
                    }
                });
            }

            drawUserPath();
        }

        function drawZones() {
            noStroke();
            for (let i = 1; i <= 5; i++) {
                fill(255, 255, 255, i % 2 === 0 ? 5 : 2);
                let y = map(i, 0.5, 5.5, height, 0);
                let h = height / 5;
                rect(0, y - h/2, width, h);
            }
        }

        function drawGuidePath() {
            let data = MODELS[currentTone];
            noFill();
            // 繪製半透明引導區
            stroke(56, 189, 248, 30);
            strokeWeight(50);
            strokeJoin(ROUND);
            renderPath(data);
            
            // 繪製虛線核心
            stroke(56, 189, 248, 100);
            strokeWeight(2);
            drawingContext.setLineDash([10, 10]);
            renderPath(data);
            drawingContext.setLineDash([]);
        }

        function drawUserPath() {
            noFill();
            stroke(251, 113, 133);
            strokeWeight(6);
            beginShape();
            points.forEach((p, i) => {
                if (p !== null) {
                    let x = map(i, 0, 60, 0, width * 0.85);
                    let y = map(p, 0.5, 5.5, height, 0);
                    vertex(x, y);
                } else {
                    endShape();
                    beginShape();
                }
            });
            endShape();
        }

        function renderPath(data) {
            beginShape();
            data.forEach((v, i) => {
                let x = map(i, 0, data.length - 1, 0, width * 0.85);
                let y = map(v, 0.5, 5.5, height, 0);
                vertex(x, y);
            });
            endShape();
        }

        function keyPressed() {
            if (key === ' ') {
                pitch.getPitch((err, freq) => {
                    if (freq) {
                        baseLog = Math.log2(freq);
                        calibrated = true;
                    }
                });
            }
        }
    </script>
</body>
</html>
