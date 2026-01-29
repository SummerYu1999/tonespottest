// ... (前面的 setup 與變數保持不變)

function draw() {
    background(15, 23, 42);
    drawGrid();
    
    // 1. 先畫出「目標路徑導引」
    drawStandardGuide(); 
    
    if (pitch && isCalibrated) {
        pitch.getPitch((err, freq) => {
            if (freq && freq > 60) {
                let logFreq = Math.log2(freq);
                // 這裡的映射範圍可依據你的回饋微調
                let val = map(logFreq, userBaseLog - 0.4, userBaseLog + 0.4, 1, 5);
                userStream.push(val);
                if (userStream.length > 50) userStream.shift();
                isRecording = true;
            } else {
                if (isRecording) { 
                    calculateScore(); 
                    isRecording = false;
                    // 停止發音時不立即清空，讓使用者看清楚剛才的曲線
                }
            }
        });
    }

    // 2. 繪製使用者即時曲線 (紅線)
    noFill();
    stroke(244, 63, 94);
    strokeWeight(5);
    beginShape();
    userStream.forEach((v, i) => {
        let x = map(i, 0, 50, 0, width * 0.8); // 配合左 8 右 2
        let y = map(v, 0.5, 5.5, height, 0);
        vertex(x, y);
    });
    endShape();
}

// 核心改動：繪製半透明虛線路徑
function drawStandardGuide() {
    let model = TONE_MODELS[currentTarget];
    
    // 繪製半透明的容錯區塊 (視覺引導)
    noFill();
    stroke(56, 189, 248, 40); // 淺藍色半透明
    strokeWeight(40); // 較寬的感應區
    drawPath(model);

    // 繪製中心虛線 (理想核心)
    stroke(56, 189, 248, 150); // 較深藍色
    strokeWeight(2);
    setLineDash([10, 10]); // 開啟虛線模式
    drawPath(model);
    setLineDash([]); // 關閉虛線模式，避免影響其他繪圖
}

function drawPath(dataArray) {
    beginShape();
    dataArray.forEach((v, i) => {
        let x = map(i, 0, dataArray.length - 1, 0, width * 0.8);
        let y = map(v, 0.5, 5.5, height, 0);
        vertex(x, y);
    });
    endShape();
}

// 輔助函式：讓 p5.js 支援虛線
function setLineDash(list) {
    drawingContext.setLineDash(list);
}
