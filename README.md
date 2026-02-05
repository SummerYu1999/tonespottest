Python 3.14.2 (tags/v3.14.2:df79316, Dec  5 2025, 17:18:21) [MSC v.1944 64 bit (AMD64)] on win32
Enter "help" below or click "Help" above for more information.
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>發音部位全座標測試</title>
    <style>
        body { font-family: sans-serif; background: #f0f0f0; display: flex; flex-direction: column; align-items: center; padding: 20px; }
        .map-wrapper { position: relative; display: inline-block; background: white; padding: 10px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.2); }
        #target-img { max-width: 600px; height: auto; display: block; }
        
        /* 測試藍點樣式 */
        .test-dot {
            position: absolute;
            width: 12px;
            height: 12px;
            background: #007bff;
            border: 2px solid white;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 9px;
            font-weight: bold;
            box-shadow: 0 0 5px rgba(0,0,0,0.5);
        }
        .label-text {
            position: absolute;
            white-space: nowrap;
            font-size: 12px;
            color: #0056b3;
            transform: translate(10px, -10px);
            background: rgba(255,255,255,0.8);
            padding: 2px 4px;
            border-radius: 3px;
        }
    </style>
</head>
<body>

    <h2>📍 發音部位座標對齊測試</h2>
    <p>藍點應準確覆蓋在圖片的數字編號上</p>

    <div class="map-wrapper" id="container">
        <img id="target-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png">
        </div>

    <script>
        // 這是你整理的數據
        // 注意：如果數據是像素(如 277)，我這裡暫時假設圖片原始寬度為 1000 來換算百分比
        // 如果點位偏移嚴重，表示需要重新用校準器點擊獲取百分比
        const testData = [
            { id: "1上", name: "上唇", x: 9.1, y: 40.3 },
            { id: "1下", name: "下唇", x: 4.3, y: 72.0 },
            { id: "2上", name: "上齒", x: 14.7, y: 44.6 },
            { id: "2下", name: "下齒", x: 9.9, y: 66.1 },
            { id: "4", name: "齒齦", x: 27.7, y: 40.6 }, // 自動轉為百分比
            { id: "5", name: "齒齦後", x: 32.5, y: 39.2 },
            { id: "6", name: "硬腭前", x: 38.3, y: 36.3 },
            { id: "7", name: "硬腭", x: 51.7, y: 37.4 },
            { id: "8", name: "軟腭", x: 64.9, y: 39.5 },
            { id: "9", name: "小舌", x: 72.3, y: 48.9 },
            { id: "10", name: "咽腔壁", x: 84.9, y: 67.0 },
            { id: "11", name: "聲門", x: 85.5, y: 91.2 },
            { id: "12", name: "會厭", x: 74.5, y: 76.9 },
            { id: "13", name: "舌根", x: 68.1, y: 68.9 },
            { id: "14", name: "舌面後", x: 55.3, y: 56.6 },
            { id: "15", name: "舌面前", x: 34.5, y: 55.2 },
            { id: "16", name: "舌葉", x: 16.9, y: 56.6 },
            { id: "17", name: "舌尖", x: 12.3, y: 61.1 },
            { id: "18", name: "舌尖下", x: 19.3, y: 64.6 }
        ];

        const container = document.getElementById('container');

        testData.forEach(point => {
            const dot = document.createElement('div');
            dot.className = 'test-dot';
            dot.style.left = point.x + '%';
            dot.style.top = point.y + '%';
            dot.innerText = point.id;

            const label = document.createElement('span');
            label.className = 'label-text';
            label.innerText = point.name;
            dot.appendChild(label);

            container.appendChild(dot);
        });
    </script>
</body>
</html>
