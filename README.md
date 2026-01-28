<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音五度標記正音儀 - 純原生版</title>
    <style>
        body { font-family: "PingFang TC", "Microsoft JhengHei", sans-serif; background: #0a0a0a; color: #fff; margin: 0; display: flex; height: 100vh; overflow: hidden; }
        #main { flex: 1; display: flex; flex-direction: column; padding: 20px; }
        #side { width: 320px; background: #161616; padding: 25px; border-left: 1px solid #333; overflow-y: auto; }
        
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
        .title { font-size: 22px; color: #4a90e2; font-weight: bold; }
        
        .stat-card { background: #222; border-radius: 12px; padding: 15px; display: flex; justify-content: space-around; margin-bottom: 15px; border: 1px solid #444; }
        .stat-item { text-align: center; }
        .stat-label { font-size: 12px; color: #888; display: block; }
        .stat-value { font-size: 28px; font-family: 'Courier New', monospace; color: #00ff00; font-weight: bold; }

        canvas { background: #000; flex: 1; border-radius: 15px; border: 2px solid #333; }
        
        .controls { display: flex; gap: 12px; margin-bottom: 15px; }
        button { flex: 1; padding: 14px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; font-size: 16px; transition: 0.3s; color: white; }
        #startBtn { background: #4a90e2; }
        #caliBtn { background: #f39c12; }
        button:disabled { background: #444; opacity: 0.5; cursor: not-allowed; }

        .sentence-box { background: #2a2a2a; padding: 20px; border-radius: 10px; border-left: 5px solid #f39c12; margin-bottom: 25px; text-align: center; }
        .sentence-text { font-size: 28px; color: #f39c12; font-weight: bold; letter-spacing: 5px; margin-bottom: 5px; }
        .zhuyin-text { font-size: 16px; color: #aaa; }

        h3 { color: #4a90e2; border-bottom: 1px solid #444; padding-bottom: 8px; margin-top: 0; }
        .tone-guide { font-size: 14px; line-height: 2; color: #ccc; }
        .tone-guide b { color: #00ff00; }
    </style>
</head>
<body>

<div id="main">
    <div class="header">
        <div class="title">注音五度標記正音儀</div>
        <div class="zhuyin-text">原生偵測引擎 v9.0</div>
    </div>

    <div class="controls">
        <button id="startBtn">1. 開啟偵測器</button>
        <button id="caliBtn" disabled>2. 唸例句校準</button>
    </div>

    <div class="stat-card">
        <div class="stat-item"><span class="stat-label">頻率 PITCH</span><span id="hzVal" class="stat-value">---</span> Hz</div>
        <div class="stat-item"><span class="stat-label">五度樓層 LEVEL</span><span id="lvVal" class="stat-value">--</span></div>
    </div>

    <canvas id="canvas"></canvas>
</div>

<div id="side">
    <h3>練習例句</h3>
    <div class="sentence-box">
        <div class="sentence-text">他拔起把柄</div>
        <div class="zhuyin-text">ㄊㄚ ㄅㄚˊ ㄑㄧˇ ㄅㄚˇ ㄅㄧㄥˇ</div>
    </div>

    <h3>聲調觀察指南</h3>
    <div class="tone-guide">
        <b>● 一聲 (55):</b> 高平線，維持在 5 樓。<br>
        <b>● 二聲 (35):</b> 斜升線，3 樓衝到 5 樓。<br>
        <b>● 三聲 (214):</b> 降升線，壓到 1 樓再勾起。<br>
        <b>● 四聲 (51):</b> 垂直線，由 5 樓急速墜落。<br>
    </div>
</div>

<script>
let audioCtx, analyser, dataArray;
let minHz = 100, maxHz = 350;
let isCalibrating = false;
let tempMin = 1000, tempMax = 50;
let history = [];

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

document.getElementById('startBtn').onclick = async () => {
    try {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)({ latencyHint: 'interactive' });
        const stream = await navigator.mediaDevices.getUserMedia({ 
            audio: { echoCancellation: false, noiseSuppression: true, autoGainControl: true } 
        });
        const source = audioCtx.createMediaStreamSource(stream);
        analyser = audioCtx.createAnalyser();
        analyser.fftSize = 2048; 
        source.connect(analyser);
        dataArray = new Float32Array(analyser.fftSize);

        document.getElementById('startBtn').disabled = true;
        document.getElementById('startBtn').innerText = "偵測中...";
        document.getElementById('caliBtn').disabled = false;
        
        history = new Array(Math.floor(canvas.offsetWidth)).fill(null);
        loop();
    } catch (e) {
        alert("麥克風存取失敗！");
    }
};

document.getElementById('caliBtn').onclick = () => {
    isCalibrating = true;
    tempMin = 500; tempMax = 80;
    alert("請自然讀出：他拔起把柄");
    setTimeout(() => {
        isCalibrating = false;
        if(tempMax > tempMin + 30) {
            minHz = tempMin; maxHz = tempMax;
        }
    }, 5000);
};

// YIN 核心演算 - 確保即時回饋
function getYinPitch(buffer, sampleRate) {
    let size = buffer.length / 2;
    let yinBuffer = new Float32Array(size);
    for (let t = 1; t < size; t++) {
        for (let i = 0; i < size; i++) {
            let delta = buffer[i] - buffer[i + t];
            yinBuffer[t] += delta * delta;
        }
    }
    yinBuffer[0] = 1;
    let runningSum = 0;
    for (let t = 1; t < size; t++) {
        runningSum += yinBuffer[t];
        yinBuffer[t] *= t / runningSum;
    }
    let tau = -1;
    for (let t = 1; t < size; t++) {
        if (yinBuffer[t] < 0.15) {
            tau = t;
            break;
        }
    }
    return tau === -1 ? null : sampleRate / tau;
}

function loop() {
    analyser.getFloatTimeDomainData(dataArray);
    let rms = 0;
    for (let i = 0; i < dataArray.length; i++) rms += dataArray[i] * dataArray[i];
    rms = Math.sqrt(rms / dataArray.length);

    if (rms > 0.015) {
        let p = getYinPitch(dataArray, audioCtx.sampleRate);
        if (p && p > 70 && p < 600) {
            document.getElementById('hzVal').innerText = Math.round(p);
            if(isCalibrating) {
                tempMin = Math.min(tempMin, p);
                tempMax = Math.max(tempMax, p);
            }
            let lv = 1 + 4 * (p - minHz) / (maxHz - minHz);
            lv = Math.max(1, Math.min(5, lv));
            document.getElementById('lvVal').innerText = lv.toFixed(1);
            history.push(lv);
        } else { history.push(null); }
    } else { history.push(null); }

    if (history.length > canvas.offsetWidth) history.shift();
    
    // 繪圖
    if (canvas.width !== canvas.offsetWidth) canvas.width = canvas.offsetWidth;
    if (canvas.height !== canvas.offsetHeight) canvas.height = canvas.offsetHeight;
    ctx.fillStyle = "#000";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    const m = 50, h = canvas.height - m * 2;
    for(let i=1; i<=5; i++) {
        let y = canvas.height - m - (i-1)*(h/4);
        ctx.strokeStyle = "#333";
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        ctx.fillStyle = "#666"; ctx.fillText(i + "F", 10, y - 5);
    }
    ctx.beginPath();
    ctx.strokeStyle = "#00ff00";
    ctx.lineWidth = 4;
    let drawing = false;
    for(let i=0; i<history.length; i++) {
        if (history[i] === null) { drawing = false; continue; }
        let y = canvas.height - m - (history[i]-1)*(h/4);
        if (!drawing) { ctx.moveTo(i, y); drawing = true; }
        else { ctx.lineTo(i, y); }
    }
    ctx.stroke();
    requestAnimationFrame(loop);
}
</script>
</body>
</html>
