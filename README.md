<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>注音五度標記偵測器 v4.0 - 即時優化版</title>
    <style>
        body { font-family: sans-serif; background: #121212; color: #eee; margin: 0; display: flex; height: 100vh; overflow: hidden; }
        #left { flex: 1; padding: 20px; display: flex; flex-direction: column; }
        #right { width: 300px; background: #1e1e1e; padding: 20px; border-left: 1px solid #333; }
        h1 { color: #4a90e2; font-size: 20px; margin: 0 0 10px 0; }
        canvas { background: #000; width: 100%; flex: 1; border-radius: 8px; border: 1px solid #444; }
        .stat-bar { background: #2a2a2a; padding: 10px; border-radius: 8px; margin-bottom: 10px; display: flex; justify-content: space-around; color: #00ff00; font-family: monospace; }
        .btns { display: flex; gap: 10px; margin-bottom: 10px; }
        button { flex: 1; padding: 10px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; background: #4a90e2; color: white; }
        button:disabled { background: #444; }
        .sentence { background: #333; padding: 10px; border-radius: 5px; color: #f39c12; text-align: center; font-weight: bold; }
    </style>
</head>
<body>

<div id="left">
    <h1>五度標記即時正音工具 v4.0</h1>
    <div class="btns">
        <button id="startBtn">1. 開始偵測</button>
        <button id="caliBtn" style="background:#f39c12" disabled>2. 唸例句校準</button>
    </div>
    <div class="stat-bar">
        <span>Hz: <span id="hzDisplay">0</span></span>
        <span>Level: <span id="lvDisplay">--</span></span>
    </div>
    <canvas id="canvas"></canvas>
</div>

<div id="right">
    <h3>校準例句</h3>
    <div class="sentence">「他拔起把柄。」</div>
    <p style="font-size: 13px; color: #888; line-height: 1.6;">
        <b>一聲:</b> 5-5 高平<br>
        <b>二聲:</b> 3-5 上揚<br>
        <b>三聲:</b> 2-1-4 轉折<br>
        <b>四聲:</b> 5-1 下墜
    </p>
</div>

<script>
let audioCtx, analyser, dataArray;
let minHz = 100, maxHz = 350;
let isCalibrating = false;
let tempMin = 500, tempMax = 80;

const canvas = document.getElementById('canvas'), ctx = canvas.getContext('2d');
let history = new Array(600).fill(null);

document.getElementById('startBtn').onclick = async () => {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    const stream = await navigator.mediaDevices.getUserMedia({ audio: { echoCancellation: true, noiseSuppression: true } });
    const source = audioCtx.createMediaStreamSource(stream);
    
    // 使用內建的頻率分析器，fftSize 設小一點可以降低延遲
    analyser = audioCtx.createAnalyser();
    analyser.fftSize = 2048;
    source.connect(analyser);
    dataArray = new Uint8Array(analyser.frequencyBinCount);
    
    document.getElementById('startBtn').disabled = true;
    document.getElementById('caliBtn').disabled = false;
    render();
};

document.getElementById('caliBtn').onclick = () => {
    isCalibrating = true;
    tempMin = 500; tempMax = 80;
    alert("請自然讀出：『他拔起把柄』");
    setTimeout(() => {
        isCalibrating = false;
        if(tempMax > tempMin + 20) {
            minHz = tempMin; maxHz = tempMax;
        }
    }, 5000);
};

// 使用快速頻率偵測法 (Pitch Detection using ByteFrequencyData)
function getPitch() {
    analyser.getByteFrequencyData(dataArray);
    let maxVal = -1, maxIndex = -1;
    // 限制在說話頻率範圍 (約 80Hz - 600Hz)
    let lower = Math.floor(80 / (audioCtx.sampleRate / analyser.fftSize));
    let upper = Math.ceil(600 / (audioCtx.sampleRate / analyser.fftSize));
    
    for (let i = lower; i < upper; i++) {
        if (dataArray[i] > maxVal) {
            maxVal = dataArray[i];
            maxIndex = i;
        }
    }
    
    if (maxVal < 50) return null; // 噪音過濾
    return maxIndex * (audioCtx.sampleRate / analyser.fftSize);
}

function render() {
    let pitch = getPitch();
    if (pitch) {
        document.getElementById('hzDisplay').innerText = Math.round(pitch);
        if(isCalibrating) {
            tempMin = Math.min(tempMin, pitch);
            tempMax = Math.max(tempMax, pitch);
        }
        let lv = 1 + 4 * (pitch - minHz) / (maxHz - minHz);
        lv = Math.max(1, Math.min(5, lv));
        document.getElementById('lvDisplay').innerText = lv.toFixed(1);
        history.push(lv);
    } else {
        history.push(null);
    }
    
    if (history.length > canvas.width) history.shift();
    draw();
    requestAnimationFrame(render);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const m = 40;
    const h = canvas.height - m * 2;
    
    // 樓層線
    ctx.strokeStyle = "#333";
    for(let i=1; i<=5; i++) {
        let y = canvas.height - m - (i-1)*(h/4);
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        ctx.fillStyle = "#666"; ctx.fillText(i+"F", 10, y-5);
    }
    
    // 聲調曲線
    ctx.beginPath();
    ctx.strokeStyle = "#00ff00";
    ctx.lineWidth = 3;
    let first = true;
    for(let i=0; i<history.length; i++) {
        if (history[i] === null) { first = true; continue; }
        let y = canvas.height - m - (history[i]-1)*(h/4);
        if (first) { ctx.moveTo(i, y); first = false; }
        else { ctx.lineTo(i, y); }
    }
    ctx.stroke();
}
</script>
</body>
</html>
