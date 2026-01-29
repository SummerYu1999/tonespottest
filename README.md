<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8" />
<title>五度聲調即時回饋</title>
<style>
  body { font-family: sans-serif; background:#111; color:#eee; }
  canvas { background:#000; display:block; margin:10px auto; }
  button { font-size:16px; padding:6px 12px; }
</style>
</head>
<body>
<h3>校正：請用正常速度唸「一二三四五」</h3>
<button id="start">開始</button>
<canvas id="cv" width="800" height="300"></canvas>

<script>
const canvas = document.getElementById("cv");
const ctx = canvas.getContext("2d");
const W = canvas.width, H = canvas.height;

let audioCtx, analyser, data;
let f0Min=null, f0Max=null;
let buffer=[], timeBuf=[];
const WINDOW_SEC = 2.0;
const ANCHOR = 0.8;
let startTime = null;
let emaPrev = null;
const ALPHA = 0.3;

// ---- 簡化 YIN（可即時跑）----
function yinPitch(buf, sr) {
  const N = buf.length;
  let minTau = sr/500, maxTau = sr/80;
  let bestTau = -1, bestVal = 1e9;
  for (let tau=minTau; tau<maxTau; tau++) {
    let sum=0;
    for (let i=0;i<N-tau;i++){
      let d=buf[i]-buf[i+tau];
      sum+=d*d;
    }
    if (sum<bestVal){ bestVal=sum; bestTau=tau; }
  }
  return bestTau>0 ? sr/bestTau : null;
}

function smooth(v){
  if (emaPrev==null) emaPrev=v;
  emaPrev = ALPHA*v + (1-ALPHA)*emaPrev;
  return emaPrev;
}

function draw(){
  ctx.clearRect(0,0,W,H);
  ctx.strokeStyle="#0f0";
  ctx.beginPath();
  for(let i=0;i<buffer.length;i++){
    const t = timeBuf[i];
    const x = W*(ANCHOR + (t - (performance.now()/1000))/WINDOW_SEC);
    const y = H*(1 - (buffer[i]-1)/4);
    if (i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
  }
  ctx.stroke();
}

async function start(){
  audioCtx = new AudioContext();
  const stream = await navigator.mediaDevices.getUserMedia({audio:true});
  const src = audioCtx.createMediaStreamSource(stream);
  analyser = audioCtx.createAnalyser();
  analyser.fftSize = 2048;
  data = new Float32Array(analyser.fftSize);
  src.connect(analyser);
  startTime = performance.now()/1000;
  loop();
}

function loop(){
  analyser.getFloatTimeDomainData(data);
  const f0 = yinPitch(data, audioCtx.sampleRate);
  if (f0){
    // 校正階段
    if (f0Min==null){
      f0Min=f0; f0Max=f0;
    } else {
      f0Min = Math.min(f0Min, f0);
      f0Max = Math.max(f0Max, f0);
    }
    if (f0Max > f0Min){
      let tone = 1 + 4*(f0 - f0Min)/(f0Max - f0Min);
      tone = Math.max(1, Math.min(5, tone));
      tone = smooth(tone);
      const now = performance.now()/1000;
      buffer.push(tone);
      timeBuf.push(now);
      // 移除過舊資料
      while (timeBuf[0] < now - WINDOW_SEC){
        buffer.shift(); timeBuf.shift();
      }
      draw();
    }
  }
  requestAnimationFrame(loop);
}

document.getElementById("start").onclick = start;
</script>
</body>
</html>
