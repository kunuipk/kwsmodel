<!doctype html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AI Pose Game</title>

<link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/pose@0.8/dist/teachablemachine-pose.min.js"></script>

<style>
:root{
    --bg:#0f172a;
    --card:#1e293b;
    --primary:#8b5cf6;
    --success:#22c55e;
    --text:#f1f5f9;
}
*{box-sizing:border-box;margin:0;padding:0}
body{
    font-family:'Sarabun',sans-serif;
    background:linear-gradient(135deg,var(--bg),#1e1b4b);
    color:var(--text);
}
.wrapper{
    max-width:900px;
    margin:auto;
    padding:1rem;
}
h1{text-align:center;margin-bottom:.5rem}
.desc{text-align:center;color:#94a3b8;margin-bottom:1rem}

.card{
    background:var(--card);
    border-radius:20px;
    padding:1rem;
    margin-bottom:1rem;
    box-shadow:0 10px 30px rgba(0,0,0,.3);
}
.center{display:flex;flex-direction:column;align-items:center}

button{
    background:linear-gradient(135deg,#8b5cf6,#ec4899);
    color:white;
    border:none;
    padding:.9rem 2.2rem;
    border-radius:999px;
    font-size:1.1rem;
    font-weight:600;
    cursor:pointer;
}
button:disabled{opacity:.5}

canvas{
    width:100%;
    max-width:360px;
    border-radius:16px;
    margin-top:1rem;
}

.score{
    font-size:1.5rem;
    font-weight:700;
    color:var(--success);
    margin:.5rem 0;
}

#label-container div{
    background:rgba(139,92,246,.15);
    padding:.6rem;
    border-radius:10px;
    margin-bottom:.4rem;
}
</style>
</head>

<body>
<div class="wrapper">
    <h1>🎮 เกมตรวจท่าทาง AI</h1>
    <p class="desc">ทำท่าทางให้ถูกต้อง ระบบจะให้คะแนนอัตโนมัติ</p>

    <div class="card center">
        <button id="startBtn" onclick="init()">เริ่มเกม</button>
        <div class="score">คะแนน: <span id="score">0</span></div>
        <canvas id="canvas"></canvas>
        <div id="status"></div>
    </div>

    <div class="card">
        <h3>ผลการตรวจจับ</h3>
        <div id="label-container"></div>
    </div>
</div>

<script>
const URL = "https://teachablemachine.withgoogle.com/models/1cfH5vGbG/";
let model, webcam, ctx, maxPredictions;
let score = 0;
let lastScoreTime = 0;

async function init(){
    document.getElementById("startBtn").disabled = true;
    setStatus("กำลังโหลดโมเดล...");

    model = await tmPose.load(URL+"model.json", URL+"metadata.json");
    maxPredictions = model.getTotalClasses();

    const size = 320;
    webcam = new tmPose.Webcam(size, size, true);
    await webcam.setup({facingMode:"user"});
    await webcam.play();
    window.requestAnimationFrame(loop);

    const canvas = document.getElementById("canvas");
    canvas.width = size;
    canvas.height = size;
    ctx = canvas.getContext("2d");

    const labelContainer = document.getElementById("label-container");
    labelContainer.innerHTML = "";
    for(let i=0;i<maxPredictions;i++){
        labelContainer.appendChild(document.createElement("div"));
    }

    setStatus("เริ่มทำท่าทางได้เลย!");
}

async function loop(){
    webcam.update();
    await predict();
    requestAnimationFrame(loop);
}

async function predict(){
    const {pose, posenetOutput} = await model.estimatePose(webcam.canvas);
    const prediction = await model.predict(posenetOutput);

    let best = prediction.reduce((a,b)=>a.probability>b.probability?a:b);
    let now = Date.now();

    prediction.forEach((p,i)=>{
        document.getElementById("label-container").children[i].innerHTML =
        `${p.className}: ${(p.probability*100).toFixed(1)}%`;
    });

    if(best.probability > 0.85 && now-lastScoreTime>1500){
        score += 10;
        document.getElementById("score").textContent = score;
        lastScoreTime = now;
        setStatus(`✔ ตรวจพบท่า: ${best.className} (+10 คะแนน)`);
    }

    drawPose(pose);
}

function drawPose(pose){
    ctx.drawImage(webcam.canvas,0,0);
    if(pose){
        tmPose.drawKeypoints(pose.keypoints,0.5,ctx);
        tmPose.drawSkeleton(pose.keypoints,0.5,ctx);
    }
}

function setStatus(msg){
    document.getElementById("status").textContent = msg;
}
</script>
</body>
</html>
