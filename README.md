# tone-practice-traditional-chinese
There are five different tones in Chinese, and this is the tool to make sure that your tone is right or not.
這個工具可以幫助你知曉你在發音5個聲調(輕聲、一聲、二聲、三聲、四聲)時是否使用正確音調。

<script>
let audioCtx, analyser, dataArray;
let minHz = 100, maxHz = 350;
let noiseFloor = 0.02; // 預設噪聲門檻
let isCalibrating = false;
let isLearningNoise = false;

const canvas = document.getElementById('canvas'), ctx = canvas.getContext('2d');
let history = new Array(800).fill(null);

document.getElementById('startBtn').onclick = async () => {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    // 關鍵：開啟瀏覽器內建的降噪與回音消除
    const stream = await navigator.mediaDevices.getUserMedia({ 
        audio: {
            echoCancellation: true,
            noiseSuppression: true,
            autoGainControl: true
        } 
    });
    const source = audioCtx.createMediaStreamSource(stream);
    analyser = audioCtx.createAnalyser();
    analyser.fftSize = 2048;
    source.connect(analyser);
    dataArray = new Float32Array(analyser.fftSize);
    
    // 步驟 1: 先學習環境音
    learnNoise(); 
    loop();
    document.getElementById('startBtn').disabled = true;
};

// 學習環境音的功能
function learnNoise() {
    isLearningNoise = true;
    let samples = [];
    alert("請保持安靜 2 秒，讓程式學習環境雜音...");
    
    let checkNoise = setInterval(() => {
        analyser.getFloatTimeDomainData(dataArray);
        let rms = 0;
        for (let i = 0; i < dataArray.length; i++) rms += dataArray[i] * dataArray[i];
        samples.push(Math.sqrt(rms / dataArray.length));
    }, 100);

    setTimeout(() => {
        clearInterval(checkNoise);
        // 設定噪聲底限為平均值的 1.5 倍
        let avg = samples.reduce((a, b) => a + b) / samples.length;
        noiseFloor = avg * 1.5; 
        isLearningNoise = false;
        alert("環境音學習完成！現在可以開始說話了。");
    }, 2000);
}

// 核心偵測邏輯
function findPitch(data, sampleRate) {
    let n = data.length;
    let rms = 0;
    for (let i = 0; i < n; i++) rms += data[i] * data[i];
    rms = Math.sqrt(rms / n);

    // 如果聲音強度低於噪聲底限，直接判定為安靜，不計算 Hz
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
    
    // 只保留人類說話的合理頻率 (70Hz - 450Hz)
    return (pitch > 70 && pitch < 450) ? pitch : null;
}

function loop() {
    if(!isLearningNoise) {
        analyser.getFloatTimeDomainData(dataArray);
        let pitch = findPitch(dataArray, audioCtx.sampleRate);

        if (pitch) {
            document.getElementById('hzDisplay').innerText = Math.round(pitch);
            let range = maxHz - minHz;
            let lv = 1 + 4 * ((pitch - minHz) / range);
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

// 繪圖函數 (保持不變)
function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.strokeStyle = "#444";
    for(let i=1; i<=5; i++) {
        let y = canvas.height - (i-1) * (canvas.height/4) - 10;
        ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
    }
    ctx.beginPath(); ctx.strokeStyle = "#00ff00"; ctx.lineWidth = 4;
    let first = true;
    for(let i=0; i<history.length; i++) {
        if (history[i] === null) { first = true; continue; }
        let y = canvas.height - (history[i]-1) * (canvas.height/4) - 10;
        if (first) { ctx.moveTo(i, y); first = false; } else { ctx.lineTo(i, y); }
    }
    ctx.stroke();
}
</script>
