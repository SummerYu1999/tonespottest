<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>注音發音部位座標校準器</title>
    <style>
        body { font-family: "Microsoft JhengHei", sans-serif; background: #f4f7f6; display: flex; flex-direction: column; align-items: center; padding: 40px; }
        .wrapper { display: flex; gap: 30px; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        
        .image-container { position: relative; border: 1px solid #ddd; line-height: 0; }
        #target-image { max-width: 500px; height: auto; cursor: crosshair; }

        .dot {
            position: absolute;
            width: 14px;
            height: 14px;
            background: #3498db;
            border: 2px solid white;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            box-shadow: 0 0 8px rgba(52, 152, 219, 0.8);
            pointer-events: none;
        }

        .data-panel { width: 350px; }
        .code-box { 
            background: #2c3e50; color: #ecf0f1; padding: 15px; border-radius: 8px; 
            font-family: monospace; font-size: 14px; line-height: 1.6; white-space: pre-wrap;
            margin-top: 10px;
        }
        h3 { margin-top: 0; color: #2c3e50; }
        .hint { color: #7f8c8d; font-size: 0.9rem; margin-bottom: 15px; }
    </style>
</head>
<body>

    <h2>📍 發音部位座標校準工具</h2>
    <p class="hint">請點擊圖片中的數字編號，右側會自動生成可用於 MasterDictionary 的程式碼。</p>

    <div class="wrapper">
        <div class="image-container" id="container">
            <img id="target-image" src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png" alt="Articulation Map">
            <div id="marker"></div>
        </div>

        <div class="data-panel">
            <h3>📋 點擊結果</h3>
            <div id="display-text">請點擊圖片獲取座標...</div>
            <div id="code-output" class="code-box">// 點擊後顯示程式碼</div>
            <button onclick="copyCode()" style="margin-top:10px; cursor:pointer; padding:5px 10px;">複製程式碼</button>
        </div>
    </div>

    <script>
        const container = document.getElementById('container');
        const img = document.getElementById('target-image');
        const marker = document.getElementById('marker');
        const codeOutput = document.getElementById('code-output');
        const displayText = document.getElementById('display-text');

        img.addEventListener('click', function(e) {
            const rect = img.getBoundingClientRect();
            
            // 計算百分比座標
            const x = ((e.clientX - rect.left) / rect.width * 100).toFixed(1);
            const y = ((e.clientY - rect.top) / rect.height * 100).toFixed(1);

            // 放置標記點
            marker.innerHTML = `<div class="dot" style="left: ${x}%; top: ${y}%;"></div>`;

            // 顯示文字資訊
            displayText.innerHTML = `<strong>最後點擊位置：</strong> X: ${x}%, Y: ${y}%`;

            // 生成字典格式程式碼
            const codeSnippet = `pos: {x: ${x}, y: ${y}}`;
            codeOutput.innerText = codeSnippet;
        });

        function copyCode() {
            const text = codeOutput.innerText;
            navigator.clipboard.writeText(text).then(() => alert('已複製到剪貼簿！'));
        }
    </script>

</body>
</html>
