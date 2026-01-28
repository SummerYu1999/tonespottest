# tone-practice-traditional-chinese
There are five different tones in Chinese, and this is the tool to make sure if your tone right or not.
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>五度標記正音器 - 頻率增強版</title>
    <style>
        body { font-family: sans-serif; display: flex; height: 100vh; margin: 0; background: #1a1a1a; color: white; }
        #left { flex: 1; padding: 20px; display: flex; flex-direction: column; align-items: center; }
        #right { width: 300px; padding: 20px; background: #222; overflow-y: auto; font-size: 14px; }
        canvas { background: #000; width: 100%; height: 400px; border: 2px solid #444; border-radius: 10px; }
        .controls { margin-bottom: 15px; }
        .stat { font-size: 28px; font-family: monospace; color: #00ff00; margin-bottom: 10px; }
        button { padding: 12px 20px; font-size: 16px; cursor: pointer; background: #4a90e2; color: white; border: none; border-radius: 5px; }
        li { margin-bottom: 8px; border-bottom: 1px solid #333; padding-bottom: 5px; }
    </style>
</head>
<body>
    <div id="left">
        <div class="stat">音高：<span id="hzDisplay">0</span> Hz | 樓層：<span id="lvDisplay">-</span></div>
        <div class="controls">
            <button id="startBtn">開始偵測</button>
            <button id="caliBtn" style="background:#e67e22">校準(5秒：低到高)</button>
        </div>
        <canvas id="canvas"></canvas>
        <p style="color:#888">註：若線條不動，請確保說話聲音清晰。程式已鎖定頻率，不受音量影響。</p>
    </div>
    <div id="right">
        <h3>練習清單</h3>
        <ul id="list"></ul>
    </div>

<script>
let audioCtx, analyser, dataArray;
let minHz = 100, maxHz = 350, isCalib = false;
const canvas = document.getElementById('canvas'), ctx = canvas.getContext('2d');
let history = new Array(800).fill(null);

// 初始化單詞表
const words = ["ㄅ：八方(55)、爸爸(51)","ㄉ：單獨(55-35)","ㄓ：真正(55-55)","ㄕ：事實(51-51)","三聲：把、改、指(214)"];
words.forEach(w => { let li = document.createElement('li'); li.innerText = w; document.getElementById('list').appendChild(li); });

document.getElementById('startBtn').onclick = async () => {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const source = audioCtx.createMediaStreamSource(stream);
    analyser = audioCtx.createAnalyser();
    analyser.fftSize = 2048;
    source.connect(analyser);
    dataArray = new Float32Array(analyser.fftSize);
    loop();
};

document.getElementById('caliBtn').onclick = () => {
    isCalib = true; minHz = 1000; maxHz = 50;
    setTimeout(() => { isCalib = false; alert("校準完成！"); }, 5000);
};

function loop() {
    requestAnimationFrame(loop);
    analyser.getFloatTimeDomainData(dataArray);
    let pitch = findPitch(dataArray, audioCtx.sampleRate);

    if (pitch && pitch > 70 && pitch < 600) {
        document.getElementById('hzDisplay').innerText = Math.round(pitch);
        if (isCalib) {
            minHz = Math.min(minHz, pitch);
            maxHz = Math.max(maxHz, pitch);
        }
        let lv = 1 + 4 * (Math.log2(pitch/minHz) / Math.log2(maxHz/minHz));
        lv = Math.max(1, Math.min(5, lv));
        document.getElementById('lvDisplay').innerText = lv.toFixed(1);
        history.push(lv);
    } else {
        history.push(null);
    }
    if (history.length > canvas.width) history.shift();
    draw();
}

function findPitch(data, sampleRate) {
    let n = data.length, r = new Float32Array(n), sum = 0;
    for (let i = 0; i < n; i++) {
        for (let j = 0; j < n - i; j++) r[i] += data[j] * data[j + i];
    }
    let d = 0; while (r[d] > r[d+1]) d++;
    let maxV = -1, maxP = -1;
    for (let i = d; i < n; i++) { if (r[i] > maxV) { maxV = r[i]; maxP = i; } }
    return sampleRate / maxP;
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.strokeStyle = "#444";
    for(let i=1; i<=5; i++) {
        let y = canvas.height - (i-1) * (canvas.height/4);
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        ctx.fillStyle = "#888"; ctx.fillText(i+"樓", 5, y-5);
    }
    ctx.beginPath(); ctx.strokeStyle = "#00ff00"; ctx.lineWidth = 4;
    for(let i=0; i<history.length; i++) {
        if (history[i] === null) continue;
        let y = canvas.height - (history[i]-1) * (canvas.height/4);
        if (i===0) ctx.moveTo(i, y); else ctx.lineTo(i, y);
    }
    ctx.stroke();
}
</script>
</body>
</html>
</html>
