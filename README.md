<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音五度標記偵測工具 v3.5</title>
    <style>
        body { font-family: "PingFang TC", "Microsoft JhengHei", sans-serif; background: #121212; color: #eee; margin: 0; display: flex; flex-direction: column; height: 100vh; }
        .container { display: flex; flex: 1; overflow: hidden; }
        #left { flex: 1; padding: 20px; display: flex; flex-direction: column; }
        #right { width: 320px; background: #1e1e1e; padding: 20px; border-left: 1px solid #333; overflow-y: auto; }
        h1 { color: #4a90e2; font-size: 22px; margin: 0 0 10px 0; }
        .description { font-size: 13px; color: #bbb; margin-bottom: 15px; border-left: 3px solid #4a90e2; padding-left: 10px; }
        .status-box { background: #2a2a2a; padding: 12px; border-radius: 8px; margin-bottom: 15px; display: flex; justify-content: space-around; }
        .val { color: #00ff00; font-family: monospace; font-size: 22px; }
        canvas { background: #000; width: 100%; height: 380px; border-radius: 10px; border: 1px solid #444; }
        .btns { margin-bottom: 15px; display: flex; gap: 10px; }
        button { flex: 1; padding: 12px; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; color: white; transition: 0.2s; }
        #startBtn { background: #4a90e2; }
        #caliBtn { background: #f39c12; }
        button:disabled { background: #444; opacity: 0.5; }
        .tips { font-size: 14px; line-height: 1.6; color: #ccc; }
        .sentence-box { background: #333; padding: 10px; border-radius: 5px; color: #f39c12; font-weight: bold; margin: 10px 0; text-align: center; }
    </style>
</head>
<body>

<div class="container">
    <div id="left">
        <h1>五度標記法即時正音工具 v3.5</h1>
        <div class="description">
            透過即時音高偵測，將您的聲音對應至傳統語音學的「五度標記法」樓層。<br>
            <span style="font-size: 11px;">Visualizing Mandarin tones on a 5-level pitch scale.</span>
        </div>

        <div class="btns">
            <button id="startBtn">1. 環境音學習 (Stay Quiet)</button>
            <button id="caliBtn" disabled>2. 校準音域 (Read Sentence)</button>
        </div>

        <div class="status-box">
            <div>Pitch: <span id="hzDisplay" class="val">0</span> Hz</div>
            <div>Level: <span id="lvDisplay" class="val">--</span></div>
        </div>

        <canvas id="canvas"></canvas>
    </div>

    <div id="right">
        <h3>校準例句 Calibration</h3>
        <p class="tips">校準時請自然地讀出這句話，包含高低起伏：</p>
        <div class="sentence-box">「他拔起把柄。」<br>(Tā bá qǐ bà bǐng)</div>
        <hr style="border:0; border-top:1px solid #444;">
        <h3>練習建議 Tips</h3>
        <div class="tips">
            <b>● 一聲 (55):</b> 維持在高位 5 樓。<br>
            <b>● 二聲 (35):</b> 3 樓滑向 5 樓。<br>
            <b>● 三聲 (214):</b> 壓到 1 樓再微勾。<br>
            <b>● 四聲 (51):</b> 5 樓垂直重摔到 1 樓。<br>
            <b>● 輕聲:</b> 短促且根據前字變動。
        </div>
    </div>
</div>

<script>
let audioCtx, analyser, dataArray;
let minHz = 100, maxHz = 350; // 初始預設值
let noiseFloor = 0.015;
let isLearningNoise = false, isCalibrating = false;
let tempMin = 1000, tempMax = 50;

const canvas = document.getElementById('canvas'), ctx = canvas.getContext('2d');
let history = new Array(800).fill(null);

document.getElementById('startBtn').onclick = async () => {
    try {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const stream = await navigator.mediaDevices.getUserMedia({ audio: { echoCancellation: true, noiseSuppression: true } });
        const source = audioCtx.createMediaStreamSource(stream);
        analyser = audioCtx.createAnalyser();
        analyser.fftSize = 2048;
        source.connect(analyser);
        dataArray = new Float32Array(analyser.fftSize);
        
        document.getElementById('startBtn').disabled = true;
        isLearningNoise = true;
        alert("【步驟 1】請保持安靜 2 秒，系統正在過濾背景雜音...");
        
        setTimeout(() => {
            let sum = 0;
            analyser.getFloatTimeDomainData(dataArray);
            for(let v of dataArray) sum += v*v;
            noiseFloor = Math.sqrt(sum/dataArray.length) * 2.5; 
            isLearningNoise = false;
            document.getElementById('caliBtn').disabled = false;
            alert("環境音學習完成！現在可以進行校準或練習。");
        }, 2000);
        loop();
    } catch (e) { alert("麥克風存取失敗！"); }
};

document.getElementById('caliBtn').onclick = () => {
    isCalibrating = true;
    tempMin = 600; tempMax = 60; // 重置校準範圍
    alert("【步驟 2】請點擊確定後，自然讀出右側例句：\n「他拔起把柄。」");
    setTimeout(() => {
        isCalibrating = false;
        if (tempMax > tempMin + 30) {
            minHz = tempMin; maxHz = tempMax;
            alert(`校準成功！\n底音: ${Math.round(minHz)}Hz / 高音: ${Math.round(maxHz)}Hz`);
        } else {
            alert("校準失敗，讀音起伏不夠明顯，請再試一次。");
        }
    }, 6000); // 給予 6 秒唸完例句
};

function findPitch(data, sampleRate) {
    let sum = 0;
    for (let v of data) sum += v*v;
    if (Math.sqrt(sum/data.length) < noiseFloor) return null;

    let n = data.length, r = new Float32Array(n);
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n - i; j++) r[i] += data[j] * data[j + i];
    }
    let d = 0; while (r[d] > r[d+1]) d++;
    let maxV = -1, maxP = -1;
    for (let i = d; i < n; i++) { if (r[i] > maxV) { maxV = r[i]; maxP = i; } }
    let p = sampleRate / maxP;
    return (p > 70 && p < 600) ? p : null;
}

function loop() {
    if(!isLearningNoise && analyser) {
        analyser.getFloatTimeDomainData(dataArray);
        let p = findPitch(dataArray, audioCtx.sampleRate);
        if (p) {
            document.getElementById('hzDisplay').innerText = Math.round(p);
            if(isCalibrating) {
                tempMin = Math.min(tempMin, p);
                tempMax = Math.max(tempMax, p);
            }
            // 樓層計算邏輯
            let lv = 1 + 4 * (p - minHz) / (maxHz - minHz);
            lv = Math.max(1, Math.min(5, lv));
            document.getElementById('lvDisplay').innerText = lv.toFixed(1);
            history.push(lv);
        } else { history.push(null); }
    }
    if (history.length > canvas.width) history.shift();
    draw();
    requestAnimationFrame(loop);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const margin = 30; // 增加邊距防止 5 樓消失
    const drawHeight = canvas.height - margin * 2;

    for(let i=1; i<=5; i++) {
        let y = canvas.height - margin - (i-1) * (drawHeight/4);
        ctx.strokeStyle = "#333"; ctx.lineWidth = 1;
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        ctx.fillStyle = "#888"; ctx.font = "14px Arial";
        ctx.fillText(i + " 樓", 10, y - 5);
    }

    ctx.beginPath(); ctx.strokeStyle = "#00ff00"; ctx.lineWidth = 4; ctx.lineCap = "round";
    let first = true;
    for(let i=0; i<history.length; i++) {
        if (history[i] === null) { first = true; continue; }
        let y = canvas.height - margin - (history[i]-1) * (drawHeight/4);
        if (first) { ctx.moveTo(i, y); first = false; } else { ctx.lineTo(i, y); }
    }
    ctx.stroke();
}
</script>
</body>
</html>
