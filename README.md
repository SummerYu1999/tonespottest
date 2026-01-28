<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>五度標記法即時音高測試</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/addons/p5.sound.min.js"></script>
    <script src="https://unpkg.com/ml5@latest/dist/ml5.min.js"></script>
    <style>
        body { font-family: sans-serif; text-align: center; background: #f0f0f0; }
        #canvas-container { margin: 20px auto; border: 2px solid #333; width: 800px; background: white; }
        .info { color: #555; }
        .tone-guide { font-size: 24px; font-weight: bold; color: #2c3e50; }
    </style>
</head>
<body>
    <h1>五度標記法 - 即時音高追蹤</h1>
    <div class="tone-guide">
        5 (高) ——————<br>
        4 (中高) —————<br>
        3 (中) ——————<br>
        2 (中低) —————<br>
        1 (低) ——————
    </div>
    <div id="canvas-container"></div>
    <p class="info">狀態：<span id="status">正在加載模型...</span></p>
    <p>目前的頻率 (Hz): <span id="freq">0</span></p>

    <script>
        let audioContext;
        let pitch;
        let xPos = 0;
        let prevY = 0;

        function setup() {
            const canvas = createCanvas(800, 400);
            canvas.parent('canvas-container');
            background(255);
            drawGrid();
            
            audioContext = getAudioContext();
            mic = new p5.AudioIn();
            mic.start(startPitch);
        }

        function startPitch() {
            pitch = ml5.pitchDetection('./model/', audioContext, mic.stream, modelLoaded);
        }

        function modelLoaded() {
            select('#status').html('模型已就緒，請開始發音');
            getPitch();
        }

        function getPitch() {
            pitch.getPitch(function(err, frequency) {
                if (frequency) {
                    select('#freq').html(floor(frequency));
                    drawPitch(frequency);
                }
                getPitch();
            });
        }

        function drawGrid() {
            stroke(220);
            for (let i = 1; i <= 5; i++) {
                let y = map(i, 1, 5, height - 50, 50);
                line(0, y, width, y);
                noStroke();
                fill(150);
                text(i, 10, y - 5);
                stroke(220);
            }
        }

        function drawPitch(freq) {
            // 這裡簡單將 100Hz-500Hz 映射到畫布高度，後續需要校準
            let y = map(freq, 100, 500, height - 50, 50);
            y = constrain(y, 0, height);

            stroke(255, 0, 0);
            strokeWeight(3);
            if (xPos > 0) {
                line(xPos - 2, prevY, xPos, y);
            }

            prevY = y;
            xPos += 2;

            if (xPos > width) {
                xPos = 0;
                background(255);
                drawGrid();
            }
        }
    </script>
</body>
</html>
