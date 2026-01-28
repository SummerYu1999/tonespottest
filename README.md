<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>注音五度標記偵測練習工具</title>
    <style>
        body { font-family: "Microsoft JhengHei", sans-serif; display: flex; height: 100vh; margin: 0; background: #1a1a1a; color: #e0e0e0; }
        #left { flex: 1; padding: 30px; display: flex; flex-direction: column; border-right: 1px solid #333; }
        #right { width: 320px; padding: 30px; background: #252525; overflow-y: auto; }
        
        .header { margin-bottom: 20px; }
        h1 { color: #4a90e2; margin: 0 0 10px 0; font-size: 24px; }
        .intro { font-size: 14px; line-height: 1.6; color: #aaa; margin-bottom: 20px; }
        
        canvas { background: #000; width: 100%; height: 400px; border-radius: 12px; border: 1px solid #444; }
        .stat-bar { display: flex; gap: 20px; margin-bottom: 15px; font-family: monospace; font-size: 18px; color: #00ff00; }
        
        .btn-group { display: flex; gap: 10px; margin-bottom: 20px; }
        button { padding: 12px 20px; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; transition: 0.3s; }
        #startBtn { background: #4a90e2; color: white; }
        #caliBtn { background: #e67e22; color: white; }
        button:disabled { background: #555; cursor: not-allowed; }
        
        .tone-guide { background: #333; padding: 15px; border-radius: 8px; font-size: 13px; }
        .tone-guide b { color: #4a90e2; }
    </style>
</head>
<body>

<div id="left">
    <div class="header">
        <h1>五度標記法即時正音工具</h1>
        <p class="intro">
            <b>Welcome!</b> This tool visualizes your Chinese tones (1st to 4th + Neutral) in real-time. 
            透過視覺化的「五度標記」曲線，讓你一眼看出聲調是否精準到位。
        </p>
    </div>

    <div class="stat-bar">
        <span>Frequency: <span id="hzDisplay">--</span> Hz</span>
        <span>Level: <span id="lvDisplay">--</span></span>
    </div>

    <div class="btn-group">
        <button id="startBtn">Step 1: 開啟麥克風 & 學習雜音</button>
        <button id="caliBtn" disabled>Step 2: 校準個人音域</button>
    </div>

    <canvas id="canvas"></canvas>
</div>

<div id="right">
    <h3>練習小秘訣 (Tips)</h3>
    <ul style="padding-left: 20px; line-height: 1.8;">
        <li><b>一聲 (55):</b> 高且平，像心電圖停止。</li>
        <li><b>二聲 (35):</b> 向上滑翔，從中間往頂部衝。</li>
        <li><b>三聲 (214):</b> 最難！必須先「摔到地板」再勾起來。</li>
        <li><b>四聲 (51):</b> 像懸崖跳水，要乾脆地墜落。</li>
    </ul>
    
    <div class="tone-guide">
        <p><b>💡 注意事項：</b><br>
        1. 點擊按鈕 1 後，請保持安靜 2 秒，系統會自動濾除環境噪音。<br>
        2. 若曲線不動，請確認麥克風權限並稍微大聲一點。</p>
    </div>
</div>

<script>
let audioCtx, analyser, dataArray;
let minHz = 100, maxHz = 350;
let noiseFloor = 0.02;
let isLearningNoise = false;
let isCalibrating = false;
let tempMin = 1000, tempMax = 50;

const canvas = document.getElementById('canvas'), ctx = canvas.getContext('2d');
let history = new Array(800).fill(null);

document.getElementById('startBtn').onclick = async () => {
    try {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const stream = await navigator.mediaDevices.getUserMedia({ 
            audio: { echoCancellation: true, noiseSuppression: true } 
        });
        const source = audioCtx.createMediaStreamSource(stream);
        analyser = audioCtx.createAnalyser();
        analyser.fftSize = 2048;
        source.connect(analyser);
        dataArray = new Float32Array(analyser.fftSize);
        
        document.getElementById('startBtn').disabled = true;
        document.getElementById('startBtn').innerText = "偵測中...";
        
        // 學習環境音
        isLearningNoise = true;
        let samples = [];
        console.log("Learning noise...");
        
        let checkNoise = setInterval(() => {
            analyser.getFloatTimeDomainData(dataArray);
            let rms = 0;
            for (let i = 0; i < dataArray.length; i++) rms += dataArray[i] * dataArray[i];
            samples.push(Math.sqrt(rms / dataArray.length));
        }, 100);

        setTimeout(() => {
            clearInterval(checkNoise);
            let avg = samples.reduce((a, b) => a + b) / samples.length;
            noiseFloor = avg * 2.0; // 稍微提高門檻避免誤觸
            isLearningNoise = false;
            document.getElementById('caliBtn').disabled = false;
            alert("✨ 環境音學習完成！現在可以開始說話，或者點擊 Step 2 校準音域。");
        }, 2000);

        loop();
    } catch (e) {
        alert("無法開啟麥克風，請檢查權限設定。");
    }
};

document.getElementById('caliBtn').onclick = () => {
    isCalibrating = true;
    tempMin = 1000; tempMax = 50;
    alert("請在 5 秒內，發出「啊——」並由低音滑向最高音...");
    setTimeout(() => {
        isCalibrating = false;
        if (tempMax > tempMin) {
            minHz = tempMin; maxHz = tempMax;
            alert(`校準成功！您的音域範圍：${Math.round(minHz)} - ${Math.round(maxHz)} Hz`);
        }
    }, 5000);
};

function findPitch(data, sampleRate) {
    let n = data.length;
    let rms = 0;
    for (let i = 0; i < n; i++) rms += data[i] * data[i];
    rms = Math.sqrt(rms / n);

    if (rms < noiseFloor) return null;

    let r = new Float32Array(n);
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n - i; j++) r[i] += data[j] * data[j + i];
    }
    let d = 0; while (r[d] > r[d+1]) d++;
    let maxV = -1, maxP = -1;
    for (let i = d; i < n; i++) {
        if (r[i] > maxV) { maxV = r[i]; maxP = i; }
    }
    let pitch = sampleRate / maxP;
    return (pitch > 70 && pitch < 450) ? pitch : null;
}

function loop() {
    if(!isLearningNoise && analyser) {
        analyser.getFloatTimeDomainData(dataArray);
        let pitch = findPitch(dataArray, audioCtx.sampleRate);

        if (pitch) {
            document.getElementById('hzDisplay').innerText = Math.round(pitch);
            if(isCalibrating) {
                tempMin = Math.min(tempMin, pitch);
                tempMax = Math.max(tempMax, pitch);
            }
            let lv = 1 + 4 * ((pitch - minHz) / (maxHz - minHz));
            lv = Math.max(1, Math.min(5, lv));
            document.getElementById('lvDisplay').innerText = lv.toFixed(1);
            history.push(lv);
        } else {
            history.push(null);
        }
    }
    if (history.length > canvas.width) history.shift();
    draw();
    requestAnimationFrame(loop);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.strokeStyle = "#333";
    for(let i=1; i<=5; i++) {
        let y = canvas.height - (i-1) * (canvas.height/4) - 20;
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        ctx.fillStyle = "#666";
        ctx.fillText(i + " 樓", 10, y - 5);
    }
    
    ctx.beginPath();
    ctx.strokeStyle = "#00ff00";
    ctx.lineWidth = 4;
    ctx.lineCap = "round";
    let first = true;
    for(let i=0; i<history.length; i++) {
        if (history[i] === null) { first = true; continue; }
        let y = canvas.height - (history[i]-1) * (canvas.height/4) - 20;
        if (first) { ctx.moveTo(i, y); first = false; }
        else { ctx.lineTo(i, y); }
    }
    ctx.stroke();
}
</script>

</body>
</html>
