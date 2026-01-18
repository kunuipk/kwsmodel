<!doctype html>
<html lang="th" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Teachable Machine Game</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
  <script src="/_sdk/element_sdk.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600;700&amp;display=swap" rel="stylesheet">
  <style>
    body {
      box-sizing: border-box;
      font-family: 'Kanit', sans-serif;
    }
    
    @keyframes pulse-glow {
      0%, 100% { box-shadow: 0 0 20px rgba(99, 102, 241, 0.5); }
      50% { box-shadow: 0 0 40px rgba(99, 102, 241, 0.8); }
    }
    
    @keyframes float {
      0%, 100% { transform: translateY(0px); }
      50% { transform: translateY(-10px); }
    }
    
    @keyframes confetti {
      0% { transform: translateY(0) rotate(0deg); opacity: 1; }
      100% { transform: translateY(100px) rotate(720deg); opacity: 0; }
    }
    
    .floating { animation: float 3s ease-in-out infinite; }
    .pulse-glow { animation: pulse-glow 2s ease-in-out infinite; }
    
    .gradient-bg {
      background: linear-gradient(135deg, #1e1b4b 0%, #312e81 50%, #4c1d95 100%);
    }
    
    .card-glass {
      background: rgba(255, 255, 255, 0.1);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(255, 255, 255, 0.2);
    }
    
    .btn-primary {
      background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
      transition: all 0.3s ease;
    }
    
    .btn-primary:hover {
      transform: scale(1.05);
      box-shadow: 0 10px 30px rgba(99, 102, 241, 0.4);
    }
    
    .score-badge {
      background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    }
    
    .timer-ring {
      stroke-dasharray: 283;
      stroke-dashoffset: 0;
      transition: stroke-dashoffset 1s linear;
    }
    
    #webcam-container {
      position: relative;
      overflow: hidden;
    }
    
    #webcam-container canvas {
      border-radius: 1rem;
    }
    
    .result-correct {
      animation: pulse-glow 0.5s ease-in-out;
      background: linear-gradient(135deg, #10b981 0%, #059669 100%) !important;
    }
    
    .result-wrong {
      background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%) !important;
    }
    
    .confetti-piece {
      position: absolute;
      width: 10px;
      height: 10px;
      animation: confetti 1s ease-out forwards;
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
  <script src="/_sdk/data_sdk.js" type="text/javascript"></script>
 </head>
 <body class="h-full gradient-bg text-white overflow-auto">
  <div id="app" class="h-full w-full"><!-- Page 1: Welcome -->
   <div id="page-welcome" class="h-full flex flex-col items-center justify-center p-6">
    <div class="text-center mb-8">
     <div class="text-8xl mb-4 floating">
      🎮
     </div>
     <h1 id="main-title" class="text-4xl md:text-5xl font-bold bg-gradient-to-r from-indigo-300 to-purple-300 bg-clip-text text-transparent mb-2">เกมยกมือ</h1>
     <p class="text-indigo-200 text-lg">ทดสอบความเร็วของคุณ!</p>
    </div>
    <div class="card-glass rounded-2xl p-8 w-full max-w-md"><label class="block text-indigo-200 mb-2 text-sm">กรอกชื่อของคุณ</label> <input type="text" id="player-name" placeholder="ใส่ชื่อที่นี่..." class="w-full px-4 py-3 rounded-xl bg-white/10 border border-white/20 text-white placeholder-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-400 mb-6"> <button id="btn-to-instructions" class="btn-primary w-full py-4 rounded-xl font-semibold text-lg"> ถัดไป → </button>
    </div>
   </div><!-- Page 2: Instructions -->
   <div id="page-instructions" class="h-full flex flex-col items-center justify-center p-6 hidden">
    <div class="text-center mb-8">
     <div class="text-6xl mb-4">
      📖
     </div>
     <h2 class="text-3xl font-bold mb-2">วิธีการเล่น</h2>
     <p id="player-greeting" class="text-indigo-200"></p>
    </div>
    <div class="card-glass rounded-2xl p-6 w-full max-w-md mb-8">
     <div class="space-y-4">
      <div class="flex items-start gap-4">
       <div class="w-10 h-10 rounded-full bg-indigo-500 flex items-center justify-center font-bold shrink-0">
        1
       </div>
       <p id="instruction-text-display" class="text-indigo-100 pt-2">ยกมือขึ้นเมื่อเห็นสัญญาณบนหน้าจอ</p>
      </div>
      <div class="flex items-start gap-4">
       <div class="w-10 h-10 rounded-full bg-indigo-500 flex items-center justify-center font-bold shrink-0">
        2
       </div>
       <p class="text-indigo-100 pt-2">กล้องจะตรวจจับท่าทางของคุณ</p>
      </div>
      <div class="flex items-start gap-4">
       <div class="w-10 h-10 rounded-full bg-indigo-500 flex items-center justify-center font-bold shrink-0">
        3
       </div>
       <p class="text-indigo-100 pt-2">ทำท่าให้ถูกต้องเพื่อรับคะแนน!</p>
      </div>
      <div class="flex items-start gap-4">
       <div class="w-10 h-10 rounded-full bg-purple-500 flex items-center justify-center font-bold shrink-0">
        🏆
       </div>
       <p class="text-indigo-100 pt-2">รวบรวมคะแนนให้ได้มากที่สุด!</p>
      </div>
     </div>
    </div><button id="btn-start-game" class="btn-primary px-12 py-4 rounded-xl font-semibold text-lg pulse-glow"> 🎯 เริ่มเล่น! </button>
   </div><!-- Page 3: Game -->
   <div id="page-game" class="h-full flex flex-col p-4 hidden"><!-- Header -->
    <div class="flex justify-between items-center mb-4">
     <div class="score-badge px-4 py-2 rounded-full"><span class="text-sm">คะแนน:</span> <span id="score" class="font-bold text-xl ml-1">0</span>
     </div>
     <div class="card-glass px-4 py-2 rounded-full"><span class="text-sm">รอบ:</span> <span id="round" class="font-bold ml-1">1</span> <span class="text-indigo-300">/5</span>
     </div>
    </div><!-- Timer -->
    <div class="flex justify-center mb-4">
     <div class="relative w-24 h-24">
      <svg class="w-24 h-24 transform -rotate-90"><circle cx="48" cy="48" r="45" stroke="rgba(255,255,255,0.2)" stroke-width="6" fill="none" /> <circle id="timer-circle" cx="48" cy="48" r="45" stroke="#6366f1" stroke-width="6" fill="none" class="timer-ring" />
      </svg>
      <div class="absolute inset-0 flex items-center justify-center"><span id="timer-text" class="text-2xl font-bold">5</span>
      </div>
     </div>
    </div><!-- Status Message -->
    <div id="game-status" class="text-center mb-4">
     <div class="card-glass inline-block px-6 py-3 rounded-full"><span id="status-text" class="text-lg">🎬 กำลังโหลดกล้อง...</span>
     </div>
    </div><!-- Webcam -->
    <div class="flex-1 flex justify-center items-center">
     <div id="webcam-container" class="card-glass p-2 rounded-2xl"><!-- Webcam canvas will be inserted here -->
     </div>
    </div><!-- Prediction Display -->
    <div id="prediction-display" class="mt-4 text-center">
     <div class="card-glass inline-block px-6 py-3 rounded-xl"><span class="text-indigo-200">ตรวจจับ: </span> <span id="prediction-text" class="font-bold">-</span>
     </div>
    </div>
   </div><!-- Page 4: Results -->
   <div id="page-results" class="h-full flex flex-col items-center justify-center p-6 hidden">
    <div class="text-center mb-8">
     <div class="text-8xl mb-4">
      🏆
     </div>
     <h2 class="text-3xl font-bold mb-2">จบเกม!</h2>
     <p id="result-name" class="text-indigo-200 text-lg"></p>
    </div>
    <div class="card-glass rounded-2xl p-8 w-full max-w-md text-center mb-8">
     <div class="text-6xl font-bold bg-gradient-to-r from-yellow-300 to-orange-300 bg-clip-text text-transparent mb-2"><span id="final-score">0</span>
     </div>
     <p class="text-indigo-200 mb-6">คะแนนรวม</p>
     <div class="grid grid-cols-2 gap-4">
      <div class="bg-green-500/20 rounded-xl p-4">
       <div id="correct-count" class="text-2xl font-bold text-green-400">
        0
       </div>
       <div class="text-sm text-green-300">
        ถูกต้อง ✓
       </div>
      </div>
      <div class="bg-red-500/20 rounded-xl p-4">
       <div id="wrong-count" class="text-2xl font-bold text-red-400">
        0
       </div>
       <div class="text-sm text-red-300">
        ผิดพลาด ✗
       </div>
      </div>
     </div>
     <div class="mt-6 pt-6 border-t border-white/10">
      <div id="result-message" class="text-xl font-semibold text-yellow-300"></div>
     </div>
    </div><button id="btn-play-again" class="btn-primary px-12 py-4 rounded-xl font-semibold text-lg"> 🔄 เล่นอีกครั้ง </button>
   </div>
  </div><!-- Audio elements using Web Audio API -->
  <script>
    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
    
    function playCorrectSound() {
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
    
    function playWrongSound() {
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(audioContext.destination);
      
      oscillator.type = 'sawtooth';
      oscillator.frequency.setValueAtTime(200, audioContext.currentTime);
      oscillator.frequency.exponentialRampToValueAtTime(100, audioContext.currentTime + 0.3);
      
      gainNode.gain.setValueAtTime(0.2, audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.3);
      
      oscillator.start(audioContext.currentTime);
      oscillator.stop(audioContext.currentTime + 0.3);
    }
    
    function playCountdownBeep() {
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(audioContext.destination);
      
      oscillator.frequency.setValueAtTime(440, audioContext.currentTime);
      gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);
      gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
      
      oscillator.start(audioContext.currentTime);
      oscillator.stop(audioContext.currentTime + 0.1);
    }
  </script>
  <script>
    // Default configuration
    const defaultConfig = {
      game_title: 'เกมยกมือ',
      instruction_text: 'ยกมือขึ้นเมื่อเห็นสัญญาณบนหน้าจอ',
      primary_color: '#6366f1',
      secondary_color: '#8b5cf6',
      background_color: '#1e1b4b',
      text_color: '#e0e7ff',
      accent_color: '#10b981'
    };

    // Game state
    let playerName = '';
    let score = 0;
    let correctCount = 0;
    let wrongCount = 0;
    let currentRound = 1;
    const totalRounds = 5;
    let gameTimer = null;
    let timeLeft = 5;
    let isDetecting = false;
    let currentPose = 'waiting'; // waiting, raise_hand
    
    // Teachable Machine
    const URL = "https://teachablemachine.withgoogle.com/models/Ie7MQxg3s/";
    let model, webcam, maxPredictions;

    // DOM Elements
    const pages = {
      welcome: document.getElementById('page-welcome'),
      instructions: document.getElementById('page-instructions'),
      game: document.getElementById('page-game'),
      results: document.getElementById('page-results')
    };

    // Initialize Element SDK
    if (window.elementSdk) {
      window.elementSdk.init({
        defaultConfig,
        onConfigChange: async (config) => {
          document.getElementById('main-title').textContent = config.game_title || defaultConfig.game_title;
          document.getElementById('instruction-text-display').textContent = config.instruction_text || defaultConfig.instruction_text;
        },
        mapToCapabilities: (config) => ({
          recolorables: [
            {
              get: () => config.primary_color || defaultConfig.primary_color,
              set: (value) => window.elementSdk.setConfig({ primary_color: value })
            }
          ],
          borderables: [],
          fontEditable: undefined,
          fontSizeable: undefined
        }),
        mapToEditPanelValues: (config) => new Map([
          ['game_title', config.game_title || defaultConfig.game_title],
          ['instruction_text', config.instruction_text || defaultConfig.instruction_text]
        ])
      });
    }

    // Page Navigation
    function showPage(pageName) {
      Object.values(pages).forEach(page => page.classList.add('hidden'));
      pages[pageName].classList.remove('hidden');
    }

    // Initialize Teachable Machine
    async function initTeachableMachine() {
      const modelURL = URL + "model.json";
      const metadataURL = URL + "metadata.json";
      
      try {
        model = await tmImage.load(modelURL, metadataURL);
        maxPredictions = model.getTotalClasses();
        
        const flip = true;
        webcam = new tmImage.Webcam(320, 240, flip);
        await webcam.setup();
        await webcam.play();
        
        document.getElementById('webcam-container').appendChild(webcam.canvas);
        
        window.requestAnimationFrame(loop);
        document.getElementById('status-text').textContent = '✋ ยกมือขึ้นเมื่อพร้อม!';
        
        startRound();
      } catch (error) {
        document.getElementById('status-text').textContent = '❌ ไม่สามารถเปิดกล้องได้';
        console.error('Error initializing Teachable Machine:', error);
      }
    }

    async function loop() {
      if (webcam) {
        webcam.update();
        await predict();
        window.requestAnimationFrame(loop);
      }
    }

    async function predict() {
      if (!model || !isDetecting) return;
      
      const prediction = await model.predict(webcam.canvas);
      
      let highestPrediction = { className: '', probability: 0 };
      for (let i = 0; i < maxPredictions; i++) {
        if (prediction[i].probability > highestPrediction.probability) {
          highestPrediction = prediction[i];
        }
      }
      
      const predictionText = document.getElementById('prediction-text');
      const confidence = (highestPrediction.probability * 100).toFixed(0);
      predictionText.textContent = `${highestPrediction.className} (${confidence}%)`;
      
      // Check for correct pose
      if (currentPose === 'raise_hand' && highestPrediction.probability > 0.8) {
        const className = highestPrediction.className.toLowerCase();
        if (className.includes('raise') || className.includes('hand') || className.includes('up') || className === 'class 1' || className === 'class 2') {
          handleCorrect();
        }
      }
    }

    function startRound() {
      if (currentRound > totalRounds) {
        endGame();
        return;
      }
      
      document.getElementById('round').textContent = currentRound;
      timeLeft = 5;
      isDetecting = false;
      currentPose = 'waiting';
      
      updateTimerDisplay();
      document.getElementById('status-text').textContent = '⏳ เตรียมตัว...';
      
      // Random delay before showing raise hand signal
      const delay = Math.random() * 2000 + 1000;
      
      setTimeout(() => {
        currentPose = 'raise_hand';
        isDetecting = true;
        document.getElementById('status-text').textContent = '✋ ยกมือขึ้น!';
        document.getElementById('game-status').querySelector('.card-glass').classList.add('bg-yellow-500/30');
        startTimer();
      }, delay);
    }

    function startTimer() {
      gameTimer = setInterval(() => {
        timeLeft--;
        playCountdownBeep();
        updateTimerDisplay();
        
        if (timeLeft <= 0) {
          clearInterval(gameTimer);
          handleWrong();
        }
      }, 1000);
    }

    function updateTimerDisplay() {
      document.getElementById('timer-text').textContent = timeLeft;
      const circle = document.getElementById('timer-circle');
      const offset = 283 - (283 * timeLeft / 5);
      circle.style.strokeDashoffset = offset;
    }

    function handleCorrect() {
      if (!isDetecting) return;
      
      clearInterval(gameTimer);
      isDetecting = false;
      
      score += 10 + timeLeft * 2;
      correctCount++;
      
      document.getElementById('score').textContent = score;
      document.getElementById('status-text').textContent = '✅ ถูกต้อง! +' + (10 + timeLeft * 2) + ' คะแนน';
      document.getElementById('game-status').querySelector('.card-glass').classList.remove('bg-yellow-500/30');
      document.getElementById('game-status').querySelector('.card-glass').classList.add('result-correct');
      
      playCorrectSound();
      
      setTimeout(() => {
        document.getElementById('game-status').querySelector('.card-glass').classList.remove('result-correct');
        currentRound++;
        startRound();
      }, 1500);
    }

    function handleWrong() {
      isDetecting = false;
      wrongCount++;
      
      document.getElementById('status-text').textContent = '❌ หมดเวลา! ลองใหม่';
      document.getElementById('game-status').querySelector('.card-glass').classList.remove('bg-yellow-500/30');
      document.getElementById('game-status').querySelector('.card-glass').classList.add('result-wrong');
      
      playWrongSound();
      
      setTimeout(() => {
        document.getElementById('game-status').querySelector('.card-glass').classList.remove('result-wrong');
        currentRound++;
        startRound();
      }, 1500);
    }

    function endGame() {
      if (webcam) {
        webcam.stop();
      }
      
      document.getElementById('result-name').textContent = `ผู้เล่น: ${playerName}`;
      document.getElementById('final-score').textContent = score;
      document.getElementById('correct-count').textContent = correctCount;
      document.getElementById('wrong-count').textContent = wrongCount;
      
      let message = '';
      if (correctCount === totalRounds) {
        message = '🌟 สุดยอด! คะแนนเต็ม!';
      } else if (correctCount >= totalRounds * 0.8) {
        message = '👏 เยี่ยมมาก!';
      } else if (correctCount >= totalRounds * 0.6) {
        message = '👍 ดีมาก!';
      } else if (correctCount >= totalRounds * 0.4) {
        message = '💪 พยายามต่อไป!';
      } else {
        message = '🎯 ฝึกฝนเพิ่มนะ!';
      }
      document.getElementById('result-message').textContent = message;
      
      showPage('results');
    }

    function resetGame() {
      score = 0;
      correctCount = 0;
      wrongCount = 0;
      currentRound = 1;
      timeLeft = 5;
      isDetecting = false;
      currentPose = 'waiting';
      
      document.getElementById('score').textContent = '0';
      document.getElementById('round').textContent = '1';
      
      const container = document.getElementById('webcam-container');
      while (container.firstChild) {
        container.removeChild(container.firstChild);
      }
    }

    // Event Listeners
    document.getElementById('btn-to-instructions').addEventListener('click', () => {
      playerName = document.getElementById('player-name').value.trim() || 'ผู้เล่น';
      document.getElementById('player-greeting').textContent = `สวัสดี ${playerName}! มาเริ่มกันเลย`;
      showPage('instructions');
    });

    document.getElementById('btn-start-game').addEventListener('click', () => {
      showPage('game');
      initTeachableMachine();
    });

    document.getElementById('btn-play-again').addEventListener('click', () => {
      resetGame();
      showPage('welcome');
    });

    // Allow Enter key to proceed from name input
    document.getElementById('player-name').addEventListener('keypress', (e) => {
      if (e.key === 'Enter') {
        document.getElementById('btn-to-instructions').click();
      }
    });
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9bfba57b51147336',t:'MTc2ODcxMzY2MS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
