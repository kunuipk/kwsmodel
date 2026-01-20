<!doctype html>
<html lang="th" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Training By Super Wai</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/pose@0.8/dist/teachablemachine-pose.min.js"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&amp;display=swap" rel="stylesheet">
  <style>
        body {
            box-sizing: border-box;
            font-family: 'Prompt', sans-serif;
        }
        
        .gradient-bg {
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
        }
        
        .glass-card {
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .neon-glow {
            box-shadow: 0 0 20px rgba(0, 212, 255, 0.3), 0 0 40px rgba(0, 212, 255, 0.1);
        }
        
        .pulse-ring {
            animation: pulse-ring 2s cubic-bezier(0.455, 0.03, 0.515, 0.955) infinite;
        }
        
        @keyframes pulse-ring {
            0% { transform: scale(0.8); opacity: 0.8; }
            50% { transform: scale(1); opacity: 0.4; }
            100% { transform: scale(0.8); opacity: 0.8; }
        }
        
        .float-animation {
            animation: float 3s ease-in-out infinite;
        }
        
        @keyframes float {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-10px); }
        }
        
        .shimmer {
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.1), transparent);
            background-size: 200% 100%;
            animation: shimmer 2s infinite;
        }
        
        @keyframes shimmer {
            0% { background-position: -200% 0; }
            100% { background-position: 200% 0; }
        }
        
        .success-pulse {
            animation: success-pulse 0.5s ease-out;
        }
        
        @keyframes success-pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
        
        .prediction-bar {
            transition: width 0.3s ease-out;
        }
        
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            animation: blink 1s infinite;
        }
        
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
    </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full gradient-bg text-white overflow-auto">
  <div id="app-container" class="min-h-full flex flex-col"><!-- Header -->
   <header class="pt-8 pb-4 px-4">
    <div class="flex flex-col items-center">
     <div class="float-animation mb-4"><img id="logo-img" src="https://drive.google.com/uc?export=view&amp;id=16-zNTS1ZRaf-WAmYAhXNzbzkXVuw2rV-" alt="Logo" class="max-w-[140px] h-auto drop-shadow-2xl" loading="lazy" onerror="this.style.display='none'; document.getElementById('fallback-logo').style.display='flex';">
      <div id="fallback-logo" class="hidden w-[120px] h-[120px] rounded-full bg-gradient-to-br from-cyan-400 to-blue-600 items-center justify-center">
       <svg class="w-16 h-16 text-white" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.75 17L9 20l-1 1h8l-1-1-.75-3M3 13h18M5 17h14a2 2 0 002-2V5a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
       </svg>
      </div>
     </div>
     <h1 id="system-title" class="text-2xl md:text-3xl font-bold bg-gradient-to-r from-cyan-400 via-blue-400 to-purple-500 bg-clip-text text-transparent text-center">AI Training By Super Wai</h1>
     <p class="text-gray-400 text-sm mt-2">ระบบตรวจสอบท่าทางด้วย AI</p>
    </div>
   </header><!-- Main Content -->
   <main class="flex-1 px-4 py-6">
    <div class="max-w-4xl mx-auto"><!-- Status Indicator -->
     <div class="glass-card rounded-2xl p-4 mb-6">
      <div class="flex items-center justify-between">
       <div class="flex items-center gap-3">
        <div id="status-dot" class="status-dot bg-gray-500"></div><span id="status-text" class="text-gray-300 font-medium">พร้อมเริ่มต้น</span>
       </div>
       <div id="sound-indicator" class="flex items-center gap-2 text-sm text-gray-400">
        <svg class="w-4 h-4" fill="currentColor" viewbox="0 0 24 24"><path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z" />
        </svg><span>เสียงแจ้งเตือน: เปิด</span>
       </div>
      </div>
     </div><!-- Webcam Container -->
     <div class="glass-card rounded-3xl p-6 mb-6 neon-glow">
      <div class="relative flex justify-center"><!-- Decorative Ring -->
       <div id="pulse-container" class="absolute inset-0 flex items-center justify-center pointer-events-none opacity-0 transition-opacity duration-500">
        <div class="pulse-ring w-[340px] h-[340px] rounded-full border-2 border-cyan-400/30"></div>
       </div><!-- Canvas Container -->
       <div class="relative">
        <div id="canvas-wrapper" class="relative rounded-2xl overflow-hidden bg-gray-900/50">
         <canvas id="canvas" class="block"></canvas><!-- Placeholder when not started -->
         <div id="placeholder" class="absolute inset-0 flex flex-col items-center justify-center bg-gray-900/80">
          <svg class="w-20 h-20 text-cyan-400 mb-4" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M15 10l4.553-2.276A1 1 0 0121 8.618v6.764a1 1 0 01-1.447.894L15 14M5 18h8a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z" />
          </svg>
          <p class="text-gray-400 text-center">กดปุ่ม Start เพื่อเริ่มต้น<br>
           การตรวจสอบท่าทาง</p>
         </div>
        </div><!-- Success Overlay -->
        <div id="success-overlay" class="absolute inset-0 rounded-2xl border-4 border-green-400 opacity-0 pointer-events-none transition-opacity duration-300"></div>
       </div>
      </div>
     </div><!-- Control Buttons -->
     <div class="flex justify-center gap-4 mb-6"><button id="start-btn" onclick="init()" class="group relative px-8 py-4 bg-gradient-to-r from-cyan-500 to-blue-600 rounded-xl font-semibold text-lg shadow-lg shadow-cyan-500/30 hover:shadow-cyan-500/50 transition-all duration-300 hover:scale-105 active:scale-95"> <span class="flex items-center gap-2">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z" /> <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg> Start </span>
       <div class="absolute inset-0 rounded-xl shimmer"></div></button> <button id="stop-btn" onclick="stopWebcam()" disabled class="px-8 py-4 bg-gradient-to-r from-red-500 to-pink-600 rounded-xl font-semibold text-lg shadow-lg shadow-red-500/30 hover:shadow-red-500/50 transition-all duration-300 hover:scale-105 active:scale-95 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:scale-100"> <span class="flex items-center gap-2">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /> <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 10a1 1 0 011-1h4a1 1 0 011 1v4a1 1 0 01-1 1h-4a1 1 0 01-1-1v-4z" />
        </svg> Stop </span> </button>
     </div><!-- Predictions Container -->
     <div class="glass-card rounded-2xl p-6">
      <h2 class="text-lg font-semibold mb-4 flex items-center gap-2">
       <svg class="w-5 h-5 text-cyan-400" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
       </svg> ผลการตรวจสอบ</h2>
      <div id="label-container" class="space-y-3">
       <div class="text-gray-500 text-center py-8">
        <svg class="w-12 h-12 mx-auto mb-3 text-gray-600" fill="none" stroke="currentColor" viewbox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
        </svg>
        <p>รอเริ่มการตรวจสอบ...</p>
       </div>
      </div>
     </div>
    </div>
   </main><!-- Footer -->
   <footer class="py-6 px-4">
    <p id="footer-text" class="text-center text-gray-500 text-sm">© 2026 Dev by Krunuipk</p>
   </footer>
  </div>
  <script>
        // Configuration
        const defaultConfig = {
            system_title: 'AI Training By Super Wai',
            footer_text: '© 2026 Dev by Krunuipk',
            background_color: '#1a1a2e',
            accent_color: '#00d4ff',
            text_color: '#ffffff'
        };

        let config = { ...defaultConfig };

        // Model URL - Change this to your model path
        const URL = "./my_model/";
        let model, webcam, ctx, labelContainer, maxPredictions;
        let isRunning = false;
        let animationId = null;
        let audioContext = null;
        let lastCorrectPose = false;

        // Initialize Audio Context
        function initAudio() {
            if (!audioContext) {
                audioContext = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        // Play success sound
        function playSuccessSound() {
            initAudio();
            
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            oscillator.frequency.setValueAtTime(523.25, audioContext.currentTime); // C5
            oscillator.frequency.setValueAtTime(659.25, audioContext.currentTime + 0.1); // E5
            oscillator.frequency.setValueAtTime(783.99, audioContext.currentTime + 0.2); // G5
            
            gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.4);
            
            oscillator.start(audioContext.currentTime);
            oscillator.stop(audioContext.currentTime + 0.4);
        }

        // Show success effect
        function showSuccessEffect() {
            const overlay = document.getElementById('success-overlay');
            const canvasWrapper = document.getElementById('canvas-wrapper');
            
            overlay.style.opacity = '1';
            canvasWrapper.classList.add('success-pulse');
            
            setTimeout(() => {
                overlay.style.opacity = '0';
                canvasWrapper.classList.remove('success-pulse');
            }, 500);
        }

        // Initialize webcam and model
        async function init() {
            try {
                const startBtn = document.getElementById('start-btn');
                const stopBtn = document.getElementById('stop-btn');
                const statusDot = document.getElementById('status-dot');
                const statusText = document.getElementById('status-text');
                const placeholder = document.getElementById('placeholder');
                const pulseContainer = document.getElementById('pulse-container');

                // Update UI - Loading state
                startBtn.disabled = true;
                startBtn.innerHTML = `
                    <span class="flex items-center gap-2">
                        <svg class="w-6 h-6 animate-spin" fill="none" viewBox="0 0 24 24">
                            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                        </svg>
                        กำลังโหลด...
                    </span>
                `;
                statusDot.className = 'status-dot bg-yellow-500';
                statusText.textContent = 'กำลังโหลดโมเดล...';

                const modelURL = URL + "model.json";
                const metadataURL = URL + "metadata.json";

                // Load model
                model = await tmPose.load(modelURL, metadataURL);
                maxPredictions = model.getTotalClasses();

                // Setup webcam
                const size = 300;
                const flip = true;
                webcam = new tmPose.Webcam(size, size, flip);
                await webcam.setup();
                await webcam.play();
                
                isRunning = true;
                animationId = window.requestAnimationFrame(loop);

                // Setup canvas
                const canvas = document.getElementById("canvas");
                canvas.width = size;
                canvas.height = size;
                ctx = canvas.getContext("2d");

                // Setup prediction labels
                labelContainer = document.getElementById("label-container");
                labelContainer.innerHTML = '';
                
                for (let i = 0; i < maxPredictions; i++) {
                    const predDiv = document.createElement('div');
                    predDiv.className = 'prediction-item';
                    predDiv.innerHTML = `
                        <div class="flex justify-between items-center mb-1">
                            <span class="class-name text-gray-300 font-medium">Class ${i + 1}</span>
                            <span class="probability text-cyan-400 font-semibold">0%</span>
                        </div>
                        <div class="w-full h-2 bg-gray-700 rounded-full overflow-hidden">
                            <div class="prediction-bar h-full bg-gradient-to-r from-cyan-400 to-blue-500 rounded-full" style="width: 0%"></div>
                        </div>
                    `;
                    labelContainer.appendChild(predDiv);
                }

                // Update UI - Running state
                placeholder.style.display = 'none';
                pulseContainer.style.opacity = '1';
                startBtn.disabled = true;
                startBtn.innerHTML = `
                    <span class="flex items-center gap-2">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z"></path>
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                        </svg>
                        Start
                    </span>
                    <div class="absolute inset-0 rounded-xl shimmer"></div>
                `;
                stopBtn.disabled = false;
                statusDot.className = 'status-dot bg-green-500';
                statusText.textContent = 'กำลังทำงาน';

                // Initialize audio
                initAudio();

            } catch (error) {
                console.error('Error initializing:', error);
                
                const startBtn = document.getElementById('start-btn');
                const statusDot = document.getElementById('status-dot');
                const statusText = document.getElementById('status-text');
                
                startBtn.disabled = false;
                startBtn.innerHTML = `
                    <span class="flex items-center gap-2">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z"></path>
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                        </svg>
                        Start
                    </span>
                    <div class="absolute inset-0 rounded-xl shimmer"></div>
                `;
                statusDot.className = 'status-dot bg-red-500';
                statusText.textContent = 'เกิดข้อผิดพลาด - กรุณาตรวจสอบโมเดลและลองใหม่';
            }
        }

        // Stop webcam
        function stopWebcam() {
            isRunning = false;
            
            if (animationId) {
                window.cancelAnimationFrame(animationId);
                animationId = null;
            }
            
            if (webcam) {
                webcam.stop();
                webcam = null;
            }

            const startBtn = document.getElementById('start-btn');
            const stopBtn = document.getElementById('stop-btn');
            const statusDot = document.getElementById('status-dot');
            const statusText = document.getElementById('status-text');
            const placeholder = document.getElementById('placeholder');
            const pulseContainer = document.getElementById('pulse-container');
            const canvas = document.getElementById('canvas');

            // Clear canvas
            if (ctx) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
            }

            // Update UI
            placeholder.style.display = 'flex';
            pulseContainer.style.opacity = '0';
            startBtn.disabled = false;
            stopBtn.disabled = true;
            statusDot.className = 'status-dot bg-gray-500';
            statusText.textContent = 'หยุดทำงาน';

            // Reset predictions
            labelContainer = document.getElementById("label-container");
            labelContainer.innerHTML = `
                <div class="text-gray-500 text-center py-8">
                    <svg class="w-12 h-12 mx-auto mb-3 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"></path>
                    </svg>
                    <p>รอเริ่มการตรวจสอบ...</p>
                </div>
            `;
        }

        // Animation loop
        async function loop(timestamp) {
            if (!isRunning) return;
            
            webcam.update();
            await predict();
            animationId = window.requestAnimationFrame(loop);
        }

        // Predict pose
        async function predict() {
            const { pose, posenetOutput } = await model.estimatePose(webcam.canvas);
            const prediction = await model.predict(posenetOutput);

            let highestPrediction = { probability: 0, className: '' };

            for (let i = 0; i < maxPredictions; i++) {
                const probability = prediction[i].probability;
                const className = prediction[i].className;
                const predItem = labelContainer.children[i];
                
                if (predItem) {
                    const classNameEl = predItem.querySelector('.class-name');
                    const probabilityEl = predItem.querySelector('.probability');
                    const barEl = predItem.querySelector('.prediction-bar');
                    
                    classNameEl.textContent = className;
                    probabilityEl.textContent = (probability * 100).toFixed(1) + '%';
                    barEl.style.width = (probability * 100) + '%';
                    
                    // Highlight high confidence predictions
                    if (probability > 0.8) {
                        barEl.className = 'prediction-bar h-full bg-gradient-to-r from-green-400 to-emerald-500 rounded-full';
                    } else if (probability > 0.5) {
                        barEl.className = 'prediction-bar h-full bg-gradient-to-r from-cyan-400 to-blue-500 rounded-full';
                    } else {
                        barEl.className = 'prediction-bar h-full bg-gradient-to-r from-gray-500 to-gray-600 rounded-full';
                    }
                }

                if (probability > highestPrediction.probability) {
                    highestPrediction = { probability, className };
                }
            }

            // Check for correct pose (high confidence) and play sound
            const isCorrectPose = highestPrediction.probability > 0.85;
            
            if (isCorrectPose && !lastCorrectPose) {
                playSuccessSound();
                showSuccessEffect();
            }
            
            lastCorrectPose = isCorrectPose;

            // Draw pose
            drawPose(pose);
        }

        // Draw pose on canvas
        function drawPose(pose) {
            if (webcam && webcam.canvas) {
                ctx.drawImage(webcam.canvas, 0, 0);
                if (pose) {
                    const minPartConfidence = 0.5;
                    tmPose.drawKeypoints(pose.keypoints, minPartConfidence, ctx);
                    tmPose.drawSkeleton(pose.keypoints, minPartConfidence, ctx);
                }
            }
        }

        // Element SDK Integration
        async function onConfigChange(newConfig) {
            config = { ...defaultConfig, ...newConfig };
            
            document.getElementById('system-title').textContent = config.system_title || defaultConfig.system_title;
            document.getElementById('footer-text').textContent = config.footer_text || defaultConfig.footer_text;
        }

        function mapToCapabilities(config) {
            return {
                recolorables: [],
                borderables: [],
                fontEditable: undefined,
                fontSizeable: undefined
            };
        }

        function mapToEditPanelValues(config) {
            return new Map([
                ['system_title', config.system_title || defaultConfig.system_title],
                ['footer_text', config.footer_text || defaultConfig.footer_text]
            ]);
        }

        // Initialize Element SDK
        if (window.elementSdk) {
            window.elementSdk.init({
                defaultConfig,
                onConfigChange,
                mapToCapabilities,
                mapToEditPanelValues
            });
        }
    </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9c0a068b847645b9',t:'MTc2ODg2NDQzNy4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
