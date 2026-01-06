<!doctype html>
<html lang="th">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Pose Recognition</title>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/pose@0.8/dist/teachablemachine-pose.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600;700&amp;display=swap" rel="stylesheet">
  <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-color: #f1f5f9;
            --primary-color: #8b5cf6;
            --secondary-color: #ec4899;
        }

        body {
            box-sizing: border-box;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        html, body {
            height: 100%;
            width: 100%;
        }

        body {
            font-family: 'Sarabun', sans-serif;
            background: linear-gradient(135deg, var(--bg-color) 0%, #1e1b4b 100%);
            color: var(--text-color);
            overflow-x: hidden;
        }

        .app-wrapper {
            width: 100%;
            height: 100%;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 2rem 1rem;
        }

        .header {
            text-align: center;
            margin-bottom: 2rem;
            animation: fadeInDown 0.8s ease-out;
        }

        .app-title {
            font-size: 2.5rem;
            font-weight: 700;
            background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            margin-bottom: 0.5rem;
        }

        .description {
            font-size: 1rem;
            color: #94a3b8;
            font-weight: 300;
        }

        .main-content {
            width: 100%;
            max-width: 900px;
            display: flex;
            flex-direction: column;
            gap: 2rem;
            animation: fadeInUp 0.8s ease-out 0.2s both;
        }

        .video-section {
            background: var(--card-bg);
            border-radius: 20px;
            padding: 2rem;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 1.5rem;
        }

        .canvas-container {
            position: relative;
            border-radius: 16px;
            overflow: hidden;
            box-shadow: 0 8px 32px rgba(139, 92, 246, 0.2);
            background: #000;
        }

        #canvas {
            display: block;
            max-width: 100%;
            height: auto;
        }

        .start-btn {
            background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
            color: white;
            border: none;
            padding: 1rem 3rem;
            font-size: 1.125rem;
            font-weight: 600;
            border-radius: 50px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(139, 92, 246, 0.4);
            font-family: 'Sarabun', sans-serif;
        }

        .start-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(139, 92, 246, 0.6);
        }

        .start-btn:active {
            transform: translateY(0);
        }

        .start-btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }

        .predictions-section {
            background: var(--card-bg);
            border-radius: 20px;
            padding: 2rem;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.3);
        }

        .predictions-title {
            font-size: 1.5rem;
            font-weight: 600;
            margin-bottom: 1.5rem;
            color: var(--text-color);
        }

        #label-container {
            display: flex;
            flex-direction: column;
            gap: 1rem;
        }

        .prediction-item {
            background: rgba(139, 92, 246, 0.1);
            padding: 1rem;
            border-radius: 12px;
            border-left: 4px solid var(--primary-color);
            font-size: 1rem;
            transition: all 0.3s ease;
        }

        .prediction-item:hover {
            background: rgba(139, 92, 246, 0.2);
            transform: translateX(5px);
        }

        .loading-spinner {
            display: none;
            width: 40px;
            height: 40px;
            border: 4px solid rgba(139, 92, 246, 0.3);
            border-top-color: var(--primary-color);
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }

        .loading-spinner.active {
            display: block;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        @keyframes fadeInDown {
            from {
                opacity: 0;
                transform: translateY(-20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @media (max-width: 768px) {
            .app-title {
                font-size: 2rem;
            }

            .video-section, .predictions-section {
                padding: 1.5rem;
            }

            .start-btn {
                padding: 0.875rem 2rem;
                font-size: 1rem;
            }
        }
    </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
  <script src="https://cdn.tailwindcss.com" type="text/javascript"></script>
 </head>
 <body>
  <div class="app-wrapper">
   <div class="header">
    <h1 class="app-title" id="app-title">ระบบจำแนกท่าทาง AI</h1>
    <p class="description" id="description">ใช้ปัญญาประดิษฐ์ตรวจจับและจำแนกท่าทางของคุณแบบเรียลไทม์</p>
   </div>
   <div class="main-content">
    <div class="video-section"><button class="start-btn" id="start-btn" onclick="init()"> <span id="start-btn-text">เริ่มใช้งาน</span> </button>
     <div class="loading-spinner" id="loading-spinner"></div>
     <div class="canvas-container">
      <canvas id="canvas"></canvas>
     </div>
    </div>
    <div class="predictions-section">
     <h2 class="predictions-title">ผลการจำแนกท่าทาง</h2>
     <div id="label-container"></div>
    </div>
   </div>
  </div>
  <script type="text/javascript">
        const URL = "./my_model/";
        let model, webcam, ctx, labelContainer, maxPredictions;
        let isInitialized = false;

        const defaultConfig = {
            app_title: "ระบบจำแนกท่าทาง AI",
            start_button_text: "เริ่มใช้งาน",
            description_text: "ใช้ปัญญาประดิษฐ์ตรวจจับและจำแนกท่าทางของคุณแบบเรียลไทม์",
            background_color: "#0f172a",
            card_background_color: "#1e293b",
            text_color: "#f1f5f9",
            primary_action_color: "#8b5cf6",
            secondary_action_color: "#ec4899"
        };

        async function onConfigChange(config) {
            const appTitle = config.app_title || defaultConfig.app_title;
            const startButtonText = config.start_button_text || defaultConfig.start_button_text;
            const descriptionText = config.description_text || defaultConfig.description_text;
            const backgroundColor = config.background_color || defaultConfig.background_color;
            const cardBg = config.card_background_color || defaultConfig.card_background_color;
            const textColor = config.text_color || defaultConfig.text_color;
            const primaryColor = config.primary_action_color || defaultConfig.primary_action_color;
            const secondaryColor = config.secondary_action_color || defaultConfig.secondary_action_color;

            document.getElementById('app-title').textContent = appTitle;
            document.getElementById('start-btn-text').textContent = startButtonText;
            document.getElementById('description').textContent = descriptionText;

            document.documentElement.style.setProperty('--bg-color', backgroundColor);
            document.documentElement.style.setProperty('--card-bg', cardBg);
            document.documentElement.style.setProperty('--text-color', textColor);
            document.documentElement.style.setProperty('--primary-color', primaryColor);
            document.documentElement.style.setProperty('--secondary-color', secondaryColor);
        }

        if (window.elementSdk) {
            window.elementSdk.init({
                defaultConfig: defaultConfig,
                onConfigChange: onConfigChange,
                mapToCapabilities: (config) => ({
                    recolorables: [
                        {
                            get: () => config.background_color || defaultConfig.background_color,
                            set: (value) => {
                                config.background_color = value;
                                window.elementSdk.setConfig({ background_color: value });
                            }
                        },
                        {
                            get: () => config.card_background_color || defaultConfig.card_background_color,
                            set: (value) => {
                                config.card_background_color = value;
                                window.elementSdk.setConfig({ card_background_color: value });
                            }
                        },
                        {
                            get: () => config.text_color || defaultConfig.text_color,
                            set: (value) => {
                                config.text_color = value;
                                window.elementSdk.setConfig({ text_color: value });
                            }
                        },
                        {
                            get: () => config.primary_action_color || defaultConfig.primary_action_color,
                            set: (value) => {
                                config.primary_action_color = value;
                                window.elementSdk.setConfig({ primary_action_color: value });
                            }
                        },
                        {
                            get: () => config.secondary_action_color || defaultConfig.secondary_action_color,
                            set: (value) => {
                                config.secondary_action_color = value;
                                window.elementSdk.setConfig({ secondary_action_color: value });
                            }
                        }
                    ],
                    borderables: [],
                    fontEditable: undefined,
                    fontSizeable: undefined
                }),
                mapToEditPanelValues: (config) => new Map([
                    ["app_title", config.app_title || defaultConfig.app_title],
                    ["start_button_text", config.start_button_text || defaultConfig.start_button_text],
                    ["description_text", config.description_text || defaultConfig.description_text]
                ])
            });
        }

        async function init() {
            if (isInitialized) return;

            const startBtn = document.getElementById('start-btn');
            const spinner = document.getElementById('loading-spinner');
            
            startBtn.disabled = true;
            spinner.classList.add('active');

            try {
                const modelURL = URL + "model.json";
                const metadataURL = URL + "metadata.json";

                model = await tmPose.load(modelURL, metadataURL);
                maxPredictions = model.getTotalClasses();

                const size = 400;
                const flip = true;
                webcam = new tmPose.Webcam(size, size, flip);
                await webcam.setup();
                await webcam.play();
                window.requestAnimationFrame(loop);

                const canvas = document.getElementById("canvas");
                canvas.width = size;
                canvas.height = size;
                ctx = canvas.getContext("2d");
                
                labelContainer = document.getElementById("label-container");
                labelContainer.innerHTML = '';
                
                for (let i = 0; i < maxPredictions; i++) {
                    const div = document.createElement("div");
                    div.className = "prediction-item";
                    labelContainer.appendChild(div);
                }

                isInitialized = true;
                startBtn.textContent = "กำลังทำงาน...";
            } catch (error) {
                const errorDiv = document.createElement("div");
                errorDiv.className = "prediction-item";
                errorDiv.style.borderLeftColor = "#ef4444";
                errorDiv.textContent = "ไม่สามารถโหลดโมเดลได้ กรุณาตรวจสอบว่าไฟล์โมเดลอยู่ในตำแหน่งที่ถูกต้อง";
                document.getElementById("label-container").appendChild(errorDiv);
                startBtn.disabled = false;
            } finally {
                spinner.classList.remove('active');
            }
        }

        async function loop(timestamp) {
            webcam.update();
            await predict();
            window.requestAnimationFrame(loop);
        }

        async function predict() {
            const { pose, posenetOutput } = await model.estimatePose(webcam.canvas);
            const prediction = await model.predict(posenetOutput);

            for (let i = 0; i < maxPredictions; i++) {
                const probability = (prediction[i].probability * 100).toFixed(1);
                const classPrediction = `${prediction[i].className}: ${probability}%`;
                labelContainer.childNodes[i].innerHTML = classPrediction;
            }

            drawPose(pose);
        }

        function drawPose(pose) {
            if (webcam.canvas) {
                ctx.drawImage(webcam.canvas, 0, 0);
                if (pose) {
                    const minPartConfidence = 0.5;
                    tmPose.drawKeypoints(pose.keypoints, minPartConfidence, ctx);
                    tmPose.drawSkeleton(pose.keypoints, minPartConfidence, ctx);
                }
            }
        }
    </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9b993a4b208999a6',t:'MTc2NzY4MTY2Ni4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
