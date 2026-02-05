<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>全方位發音部位調教器 (1-18 全收錄)</title>
    <style>
        body { font-family: sans-serif; background: #f4f7f6; display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .main-layout { display: flex; gap: 20px; max-width: 1500px; width: 100%; justify-content: center; }
        
        .map-container { 
            position: relative; 
            background: white; 
            padding: 10px; 
            border-radius: 10px; 
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            user-select: none;
            height: fit-content;
        }
        #target-img { max-width: 650px; height: auto; display: block; }

        .draggable-dot {
            position: absolute;
            width: 22px;
            height: 22px;
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
            box-shadow: 0 2px 5px rgba(0,0,0,0.3);
        }
        .draggable-dot.active { background: #2ed573; scale: 1.2; box-shadow: 0 0 15px #2ed573; }

        .control-panel { width: 600px; display: flex; flex-direction: column; gap: 15px; }
        .editor-section { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        
        .coord-list { max-height: 500px; overflow-y: auto; margin-bottom: 15px; border: 1px solid #eee; padding: 10px; border-radius: 5px; }
        .coord-item { display: flex; align-items: center; gap: 8px; margin-bottom: 5px; padding: 5px; border-bottom: 1px solid #f9f9f9; }
        .coord-item label { width: 140px; font-weight: bold; font-size: 13px; color: #2f3542; }
        .coord-item input { width: 55px; padding: 3px; border: 1px solid #ccc; border-radius: 4px; text-align: center; }

        .code-output { 
            background: #2f3542; color: #ced6e0; padding: 15px; border-radius: 5px; 
            font-family: 'Courier New', monospace; font-size: 11px; height: 250px; overflow-y: auto; white-space: pre;
        }
        
        h2 { color: #2f3542; margin-bottom: 10px; }
        h3 { margin: 10px 0; font-size: 16px; border-left: 4px solid #3498db; padding-left: 8px; }
        button { padding: 8px 15px; cursor: pointer; border-radius: 5px; border: none; background: #3498db; color: white; transition: 0.3s; font-weight: bold; }
        button:hover { background: #2980b9; }
    </style>
</head>
<body>

    <h2>🎨 全方位發音部位視覺調教器</h2>
    <p style="color: #666; margin-bottom: 20px;">包含 1~18 號解剖部位，請拖移圓點或微調數值以校準座標。</p>

    <div class="main-layout">
        <div class="map-container" id="map-box">
            <img id="target-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png">
            <div id="dots-layer"></div>
        </div>

        <div class="control-panel">
            <div class="editor-section">
                <h3>🔢 部位座標微調 (百分比 %)</h3>
                <div id="coord-list" class="coord-list"></div>

                <h3>📋 生成 MasterDictionary 配置</h3>
                <div id="code-box" class="code-output"></div>
                
                <div style="margin-top: 15px; display: flex; gap: 10px;">
                    <button onclick="copyToClipboard()" style="background: #2ed573; flex: 1;">複製代碼</button>
                    <button onclick="window.location.reload()" style="background: #747d8c;">重置位置</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 匯入你提供的 1~18 完整部位
        // 像素值 (如 277) 自動轉換為百分比 (27.7) 供初步定位
        let points = [
            { id: 1, name: "上唇", x: 9.1, y: 40.3 },
            { id: 1.1, name: "下唇", x: 4.3, y: 72.0 },
            { id: 2, name: "上齒", x: 14.7, y: 44.6 },
            { id: 2.1, name: "下齒", x: 9.9, y: 66.1 },
            { id: 3, name: "上齒齦", x: 20.0, y: 42.0 },
            { id: 4, name: "齒齦後", x: 27.7, y: 40.6 },
            { id: 5, name: "齒齦後部", x: 32.5, y: 39.2 },
            { id: 6, name: "硬腭前部", x: 38.3, y: 36.3 },
            { id: 7, name: "硬腭", x: 51.7, y: 37.4 },
            { id: 8, name: "軟腭", x: 64.9, y: 39.5 },
            { id: 9, name: "小舌", x: 72.3, y: 48.9 },
            { id: 10, name: "咽腔壁", x: 84.9, y: 67.0 },
            { id: 11, name: "聲門", x: 85.5, y: 91.2 },
            { id: 12, name: "會厭", x: 74.5, y: 76.9 },
            { id: 13, name: "舌根", x: 68.1, y: 68.9 },
            { id: 14, name: "舌面後", x: 55.3, y: 56.6 },
            { id: 15, name: "舌面前", x: 34.5, y: 55.2 },
            { id: 16, name: "舌葉", x: 16.9, y: 56.6 },
            { id: 17, name: "舌尖", x: 12.3, y: 61.1 },
            { id: 18, name: "舌尖下部", x: 19.3, y: 64.6 }
        ];

        const dotsLayer = document.getElementById('dots-layer');
        const coordList = document.getElementById('coord-list');
        const codeBox = document.getElementById('code-box');
        const img = document.getElementById('target-img');

        function init() { renderDots(); renderInputs(); updateCode(); }

        function renderDots() {
            dotsLayer.innerHTML = '';
            points.forEach((p, index) => {
                const dot = document.createElement('div');
                dot.className = 'draggable-dot';
                dot.id = `dot-${index}`;
                dot.style.left = p.x + '%';
                dot.style.top = p.y + '%';
                dot.innerText = p.id;
                dot.onmousedown = (e) => startDrag(e, index, dot);
                dotsLayer.appendChild(dot);
            });
        }

        function renderInputs() {
            coordList.innerHTML = '';
            points.forEach((p, index) => {
                const item = document.createElement('div');
                item.className = 'coord-item';
                item.innerHTML = `
                    <label>${p.id}. ${p.name}</label>
                    X: <input type="number" step="0.1" value="${p.x}" oninput="syncInput(${index}, 'x', this.value)">
                    Y: <input type="number" step="0.1" value="${p.y}" oninput="syncInput(${index}, 'y', this.value)">
                `;
                coordList.appendChild(item);
            });
        }

        function startDrag(e, index, dotElement) {
            dotElement.classList.add('active');
            const rect = img.getBoundingClientRect();
            function move(e) {
                let x = Math.max(0, Math.min(100, ((e.clientX - rect.left) / rect.width * 100)));
                let y = Math.max(0, Math.min(100, ((e.clientY - rect.top) / rect.height * 100)));
                points[index].x = parseFloat(x.toFixed(1));
                points[index].y = parseFloat(y.toFixed(1));
                dotElement.style.left = points[index].x + '%';
                dotElement.style.top = points[index].y + '%';
                const inputs = coordList.querySelectorAll('.coord-item')[index].querySelectorAll('input');
                inputs[0].value = points[index].x;
                inputs[1].value = points[index].y;
                updateCode();
            }
            function stop() {
                document.removeEventListener('mousemove', move);
                document.removeEventListener('mouseup', stop);
                dotElement.classList.remove('active');
            }
            document.addEventListener('mousemove', move);
            document.addEventListener('mouseup', stop);
        }

        window.syncInput = function(index, axis, value) {
            const val = parseFloat(value) || 0;
            points[index][axis] = val;
            const dot = document.getElementById(`dot-${index}`);
            if (axis === 'x') dot.style.left = val + '%';
            else dot.style.top = val + '%';
            updateCode();
        };

        function updateCode() {
            let code = "const PronounceMap = {\n";
            points.forEach(p => {
                code += `    "${p.id}": { name: "${p.name}", x: ${p.x}, y: ${p.y} },\n`;
            });
            code += "};\n\n";
            code += "// 使用範例：ㄅ的部位為 PronounceMap['1']";
            codeBox.innerText = code;
        }

        window.copyToClipboard = function() {
            navigator.clipboard.writeText(codeBox.innerText);
            alert("部位配置已複製！");
        }

        img.onload = init;
        if(img.complete) init();
    </script>
</body>
</html>
