<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音五度標記正音儀 - 台灣繁體版</title>
    <style>
        body { font-family: "PingFang TC", "Microsoft JhengHei", "Segoe UI", sans-serif; background: #121212; color: #eee; margin: 0; display: flex; flex-direction: column; height: 100vh; overflow: hidden; }
        
        header { background: #1e1e1e; padding: 12px 25px; border-bottom: 2px solid #4a90e2; display: flex; justify-content: space-between; align-items: center; }
        h1 { margin: 0; font-size: 20px; color: #4a90e2; letter-spacing: 1px; }

        .main-container { display: flex; flex: 1; }

        /* 左側偵測器鑲嵌區 */
        #left-panel { flex: 1; background: #000; position: relative; border-right: 1px solid #333; }
        iframe { width: 100%; height: 100%; border: none; background: #000; }

        /* 右側教學說明欄 */
        #right-panel { width: 340px; background: #181818; padding: 25px; overflow-y: auto; box-shadow: -5px 0 15px rgba(0,0,0,0.5); }
        
        .box { background: #252525; padding: 18px; border-radius: 10px; margin-bottom: 25px; border-left: 5px solid #f39c12; }
        .sentence { font-size: 26px; color: #f39c12; text-align: center; font-weight: bold; margin: 15px 0; letter-spacing: 4px; }
        .zhuyin { font-size: 14px; color: #aaa; text-align: center; margin-top: -10px; margin-bottom: 10px; }
        
        h3 { color: #4a90e2; font-size: 18px; margin-top: 0; border-bottom: 1px solid #333; padding-bottom: 8px; }
        .tone-item { margin-bottom: 15px; font-size: 15px; line-height: 1.6; }
        .tone-item b { color: #00ff00; font-size: 16px; }
        
        .btn-link { display: inline-block; background: #4a90e2; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-weight: bold; margin-top: 15px; transition: 0.3s; }
        .btn-link:hover { background: #357abd; }
        
        .instruction { font-size: 13px; color: #999; line-height: 1.6; }
    </style>
</head>
<body>

<header>
    <h1>注音五度標記正音儀 <small style="color:#666; font-size:11px;">Taiwan Edition</small></h1>
    <div style="font-size: 12px; color: #888;">專業音高追蹤技術</div>
</header>

<div class="main-container">
    <div id="left-panel">
        <iframe 
            src="https://bideyuanli.com/pp" 
            allow="microphone" 
            title="Professional Pitch Processor">
        </iframe>
    </div>

    <div id="right-panel">
        <h3>1. 暖身校準句</h3>
        <div class="box">
            <p class="instruction">請點擊左側「開始」後，自然讀出：</p>
            <div class="sentence">他拔起把柄</div>
            <div class="zhuyin">ㄊㄚ ㄅㄚˊ ㄑㄧˇ ㄅㄚˇ ㄅㄧㄥˇ</div>
            <p class="instruction" style="text-align:center;">(涵蓋一、二、三聲變化)</p>
        </div>

        <h3>2. 五度聲調對照表</h3>
        
        <div class="tone-item">
            <b>● 一聲 (55)</b>：<br>高位平走，曲線應維持在上方頂端。
        </div>
        <div class="tone-item">
            <b>● 二聲 (35)</b>：<br>由中段向高位滑升，呈現上揚曲線。
        </div>
        <div class="tone-item">
            <b>● 三聲 (214)</b>：<br>最關鍵！聲音要先壓低到地板再稍微勾起。
        </div>
        <div class="tone-item">
            <b>● 四聲 (51)</b>：<br>由最高位急速墜落，呈現陡降線。
        </div>

        <div style="margin-top: 30px; padding: 15px; background: #222; border-radius: 8px;">
            <h3 style="font-size:15px; color:#e74c3c;">🆘 故障排除</h3>
            <p class="instruction">
                若左側畫面顯示「拒絕連線」或一片漆黑，是因為部分瀏覽器基於安全性禁止網頁嵌套。請改用以下方式：
            </p>
            <a href="https://bideyuanli.com/pp" target="_blank" class="btn-link">直接開啟偵測器視窗</a>
            <p class="instruction" style="margin-top:10px;">
                開啟後與本頁面「並排顯示」即可對照練習。
            </p>
        </div>
    </div>
</div>

</body>
</html>
