<!doctype html>
<html lang="th">
<head>
<meta charset="UTF-8">
<title>Teachable Machine Image Model</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

<style>
  body { font-family: sans-serif; text-align: center; }
  canvas { border: 2px solid #333; border-radius: 10px; }
</style>
</head>

<body>

<h2>Teachable Machine Image Model</h2>
<button onclick="init()">🎥 Start Camera</button>

<div id="webcam-container"></div>
<div id="label-container"></div>

<script>
/* 🔴 ใส่ URL โมเดลของคุณตรงนี้ */
const URL = "https://teachablemachine.withgoogle.com/models/XXXXXXXXX/";

let model, webcam, labelContainer, maxPredictions;

async function init() {
    const modelURL = URL + "model.json";
    const metadataURL = URL + "metadata.json";

    model = await tmImage.load(modelURL, metadataURL);
    maxPredictions = model.getTotalClasses();

    const flip = true;
    webcam = new tmImage.Webcam(300, 300, flip);

    await webcam.setup(); // ขอสิทธิ์กล้อง
    await webcam.play();
    window.requestAnimationFrame(loop);

    document.getElementById("webcam-container").appendChild(webcam.canvas);

    labelContainer = document.getElementById("label-container");
    labelContainer.innerHTML = "";
    for (let i = 0; i < maxPredictions; i++) {
        labelContainer.appendChild(document.createElement("div"));
    }
}

async function loop() {
    webcam.update();
    await predict();
    window.requestAnimationFrame(loop);
}

async function predict() {
    const prediction = await model.predict(webcam.canvas);
    for (let i = 0; i < maxPredictions; i++) {
        labelContainer.childNodes[i].innerHTML =
            prediction[i].className + " : " +
            (prediction[i].probability * 100).toFixed(1) + "%";
    }
}
</script>

</body>
</html>
