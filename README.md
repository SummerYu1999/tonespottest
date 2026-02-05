<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>注音點位拖移調教器</title>
    <style>
        body { font-family: sans-serif; background: #f4f7f6; display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .main-layout { display: flex; gap: 20px; max-width: 1200px; }
        
        .map-container { 
            position: relative; 
            background: white; 
            padding: 10px; 
            border-radius: 10px; 
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            user-select: none; /* 防止拖移時選中文字 */
        }
        #target-img { max-width: 600px; height: auto; display: block; }

        /* 可拖移的點 */
        .draggable-dot {
            position: absolute;
            width: 18px;
            height: 18px;
            background: #ff4757;
            border: 2px solid white;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            cursor: move;
            z-index: 100;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 10px;
            font-weight: bold;
        }
        .draggable-dot:hover { background: #ff6b81; transform: translate(-50%, -50%) scale(1.2); }
        .draggable-dot.active { background: #2ed573; box-shadow: 0 0 15px #2ed573; }

        .side-panel { width: 450px; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .code-output { 
            background: #2f3542; color: #ced6e0; padding: 15px; border-radius: 5px; 
            font-family: 'Courier New', monospace; font-size: 13px; height: 500px; overflow-y: auto; white-space: pre;
        }
        h3 { margin-top: 0; color: #2f3542; }
        .hint { font-size: 0.9rem; color: #747d8c; margin-bottom: 10px; }
    </style>
</head>
<body>

    <h2>🎨 視覺化點位拖移調教器</h2>
    <p class="hint">直接用滑鼠「按住並拖移」紅點，右側的程式碼會即時更新座標。</p>

    <div class="main-layout">
        <div class="map-container" id="map-box">
            <img id="target-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png">
            <div id="dots-layer"></div>
        </div>

        <div class="side-panel">
            <h3>📋 即時 MasterDictionary 代碼</h3>
            <div id="code-box" class="code-output"></div>
            <button onclick="copyToClipboard()" style="margin-top:10px; padding:8px 15px; cursor:pointer;">複製全部字典代碼</button>
        </div>
    </div>

    <script>
        // 初始化數據 (基於你提供的座標)
        let points = [
            { id: "ㄅㄆㄇ", label: "1上", x: 9.1, y: 40.3 },
            { id: "ㄈ", label: "2上", x: 14.7, y: 44.6 },
            { id: "ㄉㄊㄋㄌ", label: "4", x: 27.7, y: 40.6 },
            { id: "ㄓㄔㄕㄖ", label: "6", x: 38.3, y: 36.3 },
            { id: "ㄐㄑㄒ", label: "7", x: 51.7, y: 37.4 },
            { id: "ㄍㄎㄏ", label: "8", x: 64.9, y: 39.5 },
            { id: "ㄦ", label: "17", x: 12.3, y: 61.1 }
        ];

        const dotsLayer = document.getElementById('dots-layer');
        const codeBox = document.getElementById('code-box');
        const img = document.getElementById('target-img');
        let activeDot = null;

        // 渲染點位
        function render() {
            dotsLayer.innerHTML = '';
            points.forEach((p, index) => {
                const dot = document.createElement('div');
                dot.className = 'draggable-dot';
                dot.style.left = p.x + '%';
                dot.style.top = p.y + '%';
                dot.innerText = p.label;
                
                dot.onmousedown = (e) => startDrag(e, index, dot);
                dotsLayer.appendChild(dot);
            });
            updateCode();
        }

        function startDrag(e, index, dotElement) {
            activeDot = { index, element: dotElement };
            dotElement.classList.add('active');
            
            document.onmousemove = (e) => {
                const rect = img.getBoundingClientRect();
                let x = ((e.clientX - rect.left) / rect.width * 100);
                let y = ((e.clientY - rect.top) / rect.height * 100);

                // 限制範圍在圖片內
                x = Math.max(0, Math.min(100, x));
                y = Math.max(0, Math.min(100, y));

                points[index].x = parseFloat(x.toFixed(1));
                points[index].y = parseFloat(y.toFixed(1));
                
                dotElement.style.left = points[index].x + '%';
                dotElement.style.top = points[index].y + '%';
                updateCode();
            };

            document.onmouseup = () => {
                document.onmousemove = null;
                dotElement.classList.remove('active');
            };
        }

        function updateCode() {
            let code = "const MasterDictionary = {\n";
            points.forEach(p => {
                code += `    "${p.id}": { pos: {x: ${p.x}, y: ${p.y}}, loc: "部位${p.label}" },\n`;
            });
            code += "};";
            codeBox.innerText = code;
        }

        function copyToClipboard() {
            navigator.clipboard.writeText(codeBox.innerText);
            alert("代碼已複製！");
        }

        // 確保圖片載入後再渲染
        img.onload = render;
        if(img.complete) render();

    </script>
</body>
</html>
