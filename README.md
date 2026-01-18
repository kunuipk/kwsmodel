<!doctype html>
<html lang="th">
<head>
<meta charset="UTF-8">
<title>AI Image Team Game</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

<style>
body{
  margin:0;
  font-family:sans-serif;
  background:#101820;
  color:white;
  text-align:center;
}
button{
  font-size:18px;
  padding:10px 20px;
  border:none;
  border-radius:10px;
  background:#00c853;
  color:black;
}
canvas{
  width:90vw;
  max-width:320px;
  border-radius:15px;
  border:3px solid #fff;
}
.card{
  margin:10px auto;
  padding:10px;
  width:90%;
  max-width:400px;
  background:#1f2933;
  border-radius:15px;
}
.score{
  font-size:22px;
}
.target{
  color:#ff5252;
  font-size:20px;
}
.timer{
  font-size:24px;
  color:#ffeb3b;
}
</style>
</head>

<body>

<h2>🎮 AI แข่งเป็นทีม</h2>
<button onclick="startGame()">▶ เริ่มเกม</button>

<div class="card timer">⏱ เวลา: <span id="time">60</span></div>
<div class="card">👥 ทีมปัจจุบัน: <b id="team">ทีม A</b></div>
<div class="card">🎯 เป้าหมาย: <div class="target" id="target">-</div></div>

<div id="webcam-container"></div>

<div class="card score">
ทีม A: <span id="scoreA">0</span> |
ทีม B: <span id="scoreB">0</span>
</div>

<div id="result" class="card"></div>

<audio id="sound-correct" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
<audio id="sound-end" src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg"></audio>

<script>
const URL = "https://teachablemachine.withgoogle.com/models/XXXXXXXXX/";

let model, webcam, maxPredictions;
let targetClass = "";
let team = "A";
let scoreA = 0, scoreB = 0;
let time = 60;
let canScore = true;
let timerInterval;

async function startGame(){
  document.getElementById("result").innerHTML = "";
  time = 60;
  scoreA = scoreB = 0;
  team = "A";
  updateUI();

  const modelURL = URL + "model.json";
  const metadataURL = URL + "metadata.json";
  model = await tmImage.load(modelURL, metadataURL);
  maxPredictions = model.getTotalClasses();

  webcam = new tmImage.Webcam(300,300,true);
  await webcam.setup();
  await webcam.play();
  document.getElementById("webcam-container").innerHTML="";
  document.getElementById("webcam-container").appendChild(webcam.canvas);

  newTarget();
  window.requestAnimationFrame(loop);

  timerInterval = setInterval(countdown,1000);
}

function countdown(){
  time--;
  document.getElementById("time").innerText=time;
  if(time===30){
    team="B"; updateUI();
  }
  if(time<=0){
    endGame();
  }
}

function endGame(){
  clearInterval(timerInterval);
  document.getElementById("sound-end").play();

  let winner="เสมอ";
  if(scoreA>scoreB) winner="🏆 ทีม A ชนะ!";
  if(scoreB>scoreA) winner="🏆 ทีม B ชนะ!";

  document.getElementById("result").innerHTML=
    `📊 ตารางคะแนน<br>
     ทีม A: ${scoreA}<br>
     ทีม B: ${scoreB}<br><br>${winner}`;
}

function newTarget(){
  const labels = model.getClassLabels();
  targetClass = labels[Math.floor(Math.random()*labels.length)];
  document.getElementById("target").innerText=targetClass;
  canScore=true;
}

function updateUI(){
  document.getElementById("team").innerText="ทีม "+team;
  document.getElementById("scoreA").innerText=scoreA;
  document.getElementById("scoreB").innerText=scoreB;
}

async function loop(){
  webcam.update();
  const prediction = await model.predict(webcam.canvas);
  for(const p of prediction){
    if(p.className===targetClass && p.probability>0.8 && canScore && time>0){
      if(team==="A") scoreA++; else scoreB++;
      document.getElementById("sound-correct").play();
      updateUI();
      canScore=false;
      setTimeout(newTarget,1200);
    }
  }
  if(time>0) requestAnimationFrame(loop);
}
</script>

</body>
</html>
