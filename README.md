<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>五度標記法即時追蹤 - 專業版</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>
    <script src="https://unpkg.com/ml5@latest/dist/ml5.min.js"></script>
    <style>
        body { font-family: 'PingFang TC', sans-serif; text-align: center; background: #1a1a1a; color: #fff; }
        #canvas-container { margin: 20px auto; border: 3px solid #444; width: 800px; border-radius: 8px; overflow: hidden; }
        .controls { margin-bottom: 10px; color: #aaa; }
    </style>
</head>
<body>
    <h1>五度標記法即時發音辨識</h1>
    <div class="controls">
        狀態：<span id="status">模型載入中...</span> | 
        頻率：<span id="freq">0</span> Hz
    </div>
    <div id="canvas-container"></div>

    <script>
        let pitch;
        let audioContext;
        let mic;
        let points = []; // 儲存繪製點
        const maxPoints = 400; // 畫面最多呈現的點數
        let userBaseLog = null; // 使用者的基準音高（對數）

        function setup() {
            const canvas = createCanvas(800, 400);
            canvas.parent('canvas-container');
            audioContext = getAudioContext();
            mic = new p5.AudioIn();
            mic.start(() => {
                pitch = ml5.pitchDetection('./model/', audioContext, mic.stream, () => {
                    select('#status').html('請開始發音');
                    getPitch();
                });
            });
        }

        function getPitch() {
            pitch.getPitch((err, frequency) => {
                if (frequency && frequency > 50 && frequency < 1000) {
                    select('#freq').html(floor(frequency));
                    
                    // 1. 將頻率轉為對數半音值
                    let logFreq = Math.log2(frequency);
                    
                    // 2. 自動校準：如果還沒有基準，就以第一聲為基準
                    if (userBaseLog === null) userBaseLog = logFreq;
                    // 緩慢跟蹤使用者的中音域 (Simple Smoothing)
                    userBaseLog = lerp(userBaseLog, logFreq, 0.01);

                    // 3. 映射到五度 (假設音域範圍為上下各 5 個半音)
                    let toneValue = map(logFreq, userBaseLog - 0.5, userBaseLog + 0.5, 1, 5);
                    points.push(toneValue);
                } else {
                    // 沒聲音時推入 null 保持曲線斷開
                    points.push(null);
                }

                // 保持陣列長度，實現滾動感
                if (points.length > maxPoints) {
                    points.shift();
                }
                getPitch();
            });
        }

        function draw() {
            background(30);
            drawGrid();
            
            noFill();
            strokeWeight(4);
            stroke(0, 255, 200);

            // 計算繪製起點，實現「左 8 右 2」的視覺感
            // 我們永遠畫到畫布的 80% (width * 0.8)
            let drawWidth = width * 0.8;
            let step = drawWidth / maxPoints;

            beginShape();
            for (let i = 0; i < points.length; i++) {
                if (points[i] !== null) {
                    let x = i * step;
                    let y = map(points[i], 1, 5, height - 50, 50);
                    vertex(x, y);
                } else {
                    endShape();
                    beginShape();
                }
            }
            endShape();

            // 繪製掃描線指示器
            stroke(255, 100, 100, 150);
            strokeWeight(1);
            line(points.length * step, 0, points.length * step, height);
        }

        function drawGrid() {
            stroke(60);
            strokeWeight(1);
            for (let i = 1; i <= 5; i++) {
                let y = map(i, 1, 5, height - 50, 50);
                line(0, y, width, y);
                fill(100);
                noStroke();
                textSize(16);
                text(i + " 度", width * 0.85, y + 5);
            }
            // 繪製 80% 邊界線
            stroke(100, 100, 255, 50);
            line(width * 0.8, 0, width * 0.8, height);
        }
    </script>
</body>
</html>
