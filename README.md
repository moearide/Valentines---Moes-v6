<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Will you be my Valentine?</title>
  <style>
    :root { --bg1:#ffe6f2; --bg2:#fff6fb; --text:#1f1f1f; }
    * { box-sizing: border-box; }

    body{
      margin:0;
      min-height:100vh;
      display:flex;
      align-items:center;
      justify-content:center;
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      color:var(--text);
      background: radial-gradient(circle at 20% 20%, var(--bg1), var(--bg2));
      overflow:hidden;
      -webkit-tap-highlight-color: transparent;
      touch-action: manipulation;
    }

    .card{
      width:min(720px, 92vw);
      background: rgba(255,255,255,.78);
      border: 1px solid rgba(0,0,0,.06);
      border-radius: 22px;
      padding: 28px 22px;
      box-shadow: 0 18px 60px rgba(0,0,0,.08);
      text-align:center;
      position:relative;
      z-index:2;
      backdrop-filter: blur(10px);
    }

    h1{
      margin: 4px 0 10px;
      font-size: clamp(26px, 5.2vw, 40px);
      line-height: 1.1;
    }
    .sub{ margin: 0 0 18px; opacity: .8; font-size: 16px; }

    /* Run area */
    #btnArea{
      position: relative;
      width: min(560px, 92vw);
      height: 260px;
      margin: 18px auto 6px;
      border-radius: 18px;
      border: 1px dashed rgba(0,0,0,.12);
      background: rgba(255,255,255,.62);
      overflow:hidden;
      touch-action: none; /* allows us to read touchmove precisely */
    }

    button{
      position:absolute;
      border:0;
      border-radius: 16px;
      padding: 14px 22px;
      font-size: 18px;
      cursor:pointer;
      user-select:none;
      box-shadow: 0 10px 22px rgba(0,0,0,.10);
      transition: transform .22s ease, left .22s ease, top .22s ease, opacity .22s ease;
    }

    #yesBtn{
      background:#ff4da6;
      color:white;
      left: 42%;
      top: 55%;
      transform: translate(-50%,-50%) scale(1);
      z-index:2;
    }

    #noBtn{
      background:white;
      color:#222;
      left: 66%;
      top: 56%;
      transform: translate(-50%,-50%) scale(1);
      border:1px solid rgba(0,0,0,.08);
      z-index:1;
    }

    #message{
      margin: 12px 0 0;
      font-size: 18px;
      min-height: 26px;
    }

    /* Confetti canvas */
    #confetti{
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      pointer-events:none;
      z-index: 9999;
      display:none;
    }

    /* Floating hearts */
    .hearts{
      position: fixed;
      inset: 0;
      pointer-events:none;
      overflow:hidden;
      z-index:0;
    }
    .heart{
      position:absolute;
      bottom:-40px;
      font-size: 18px;
      opacity:.9;
      animation: floatUp linear forwards;
      filter: drop-shadow(0 8px 10px rgba(0,0,0,.10));
    }
    @keyframes floatUp{
      0%   { transform: translateY(0) translateX(0) rotate(0deg); opacity: 0; }
      10%  { opacity: .9; }
      100% { transform: translateY(-120vh) translateX(var(--drift)) rotate(var(--rot)); opacity: 0; }
    }
  </style>
</head>

<body>
  <div class="hearts" id="hearts"></div>
  <canvas id="confetti"></canvas>

  <div class="card">
    <h1>Will you be my Valentine? 💘</h1>
    <p class="sub">Be honest… but choose wisely 😌</p>

    <div id="btnArea">
      <button id="yesBtn">Yes 💖</button>
      <button id="noBtn">No 🙈</button>
    </div>

    <p id="message">“No” seems a bit shy…</p>
  </div>

<script>
(() => {
  let noClicks = 0;

  const btnArea = document.getElementById("btnArea");
  const noBtn   = document.getElementById("noBtn");
  const yesBtn  = document.getElementById("yesBtn");
  const message = document.getElementById("message");
  const heartsLayer = document.getElementById("hearts");

  const clamp = (v, min, max) => Math.max(min, Math.min(max, v));

  // --- Floating hearts ---
  const heartEmojis = ["💖","💘","💝","💕","💗","💓","💞"];
  function spawnHeart() {
    const el = document.createElement("div");
    el.className = "heart";
    el.textContent = heartEmojis[Math.floor(Math.random()*heartEmojis.length)];
    el.style.left = Math.random() * 100 + "vw";
    el.style.fontSize = (14 + Math.random()*22).toFixed(0) + "px";
    el.style.setProperty("--drift", (Math.random()*140 - 70).toFixed(0) + "px");
    el.style.setProperty("--rot", (Math.random()*80 - 40).toFixed(0) + "deg");
    const dur = 5 + Math.random()*5;
    el.style.animationDuration = dur + "s";
    heartsLayer.appendChild(el);
    setTimeout(() => el.remove(), (dur + 0.2) * 1000);
  }
  setInterval(spawnHeart, 480);
  for (let i=0;i<10;i++) setTimeout(spawnHeart, i*160);

  // --- Text prompts ---
  function setAreYouSureText() {
    if (noClicks === 1) message.textContent = "Are you sure? 🥺";
    else if (noClicks === 2) message.textContent = "Really sure?? 😭";
    else if (noClicks === 3) message.textContent = "Last chance… are you sure? 💔";
    else if (noClicks >= 4) message.textContent = "Okay fine… only YES is left 😌";
  }

  // --- YES scaling (base) + magnet offset ---
  let yesBaseScale = 1;
  let yesOffX = 0, yesOffY = 0;

  function applyYesTransform() {
    yesBtn.style.transform = `translate(calc(-50% + ${yesOffX}px), calc(-50% + ${yesOffY}px)) scale(${yesBaseScale})`;
  }

  function growYes() {
    yesBaseScale = 1 + noClicks * 0.28;
    applyYesTransform();
  }

  // --- No runs away + shrinks ---
  function moveNoRunAway() {
    const area = btnArea.getBoundingClientRect();
    const btnW = noBtn.offsetWidth;
    const btnH = noBtn.offsetHeight;

    const padding = 12;
    const maxX = area.width  - btnW - padding;
    const maxY = area.height - btnH - padding;

    const x = clamp(Math.random() * maxX + padding, padding, maxX);
    const y = clamp(Math.random() * maxY + padding, padding, maxY);

    noBtn.style.left = `${x}px`;
    noBtn.style.top  = `${y}px`;

    const noScale = Math.max(0.18, 1 - noClicks * 0.22);
    const tilt = (Math.random() * 14 - 7).toFixed(1);
    noBtn.style.transform = `translate(0,0) scale(${noScale}) rotate(${tilt}deg)`;
  }

  // Make "No" run away on touchstart (mobile-first)
  noBtn.addEventListener("touchstart", (e) => {
    e.preventDefault();      // stops the tap from selecting/clicking
    moveNoRunAway();
  }, { passive:false });

  // Still handle click (if someone manages)
  noBtn.addEventListener("click", (e) => {
    e.preventDefault();
    noClicks++;

    setAreYouSureText();
    growYes();

    if (noClicks >= 4) {
      noBtn.style.opacity = "0";
      noBtn.style.pointerEvents = "none";
      setTimeout(() => noBtn.remove(), 220);
      return;
    }
    moveNoRunAway();
  });

  // --- YES magnet effect (touch + pointer) ---
  // On phones, user drags finger in btnArea, yes follows.
  const magnetStrength = 0.16;

  function magnetYesTo(clientX, clientY) {
    const yesRect = yesBtn.getBoundingClientRect();
    const yesCenterX = yesRect.left + yesRect.width / 2;
    const yesCenterY = yesRect.top + yesRect.height / 2;

    const dx = clientX - yesCenterX;
    const dy = clientY - yesCenterY;

    const targetX = yesOffX + dx * magnetStrength;
    const targetY = yesOffY + dy * magnetStrength;

    // Cap offset so it stays within the area nicely
    const cap = 85 + noClicks * 10;
    yesOffX = clamp(targetX, -cap, cap);
    yesOffY = clamp(targetY, -cap, cap);

    applyYesTransform();
  }

  // Touch tracking (primary)
  btnArea.addEventListener("touchmove", (e) => {
    const t = e.touches[0];
    magnetYesTo(t.clientX, t.clientY);
  }, { passive:true });

  // Also support pointer devices (optional)
  btnArea.addEventListener("pointermove", (e) => {
    // only if it's not touch (pointerType can be 'touch','mouse','pen')
    if (e.pointerType && e.pointerType !== "mouse") return;
    magnetYesTo(e.clientX, e.clientY);
  });

  // --- Confetti ---
  const canvas = document.getElementById("confetti");
  const ctx = canvas.getContext("2d");
  let pieces = [];
  let running = false;

  function resize() {
    canvas.width = innerWidth * devicePixelRatio;
    canvas.height = innerHeight * devicePixelRatio;
    ctx.setTransform(devicePixelRatio, 0, 0, devicePixelRatio, 0, 0);
  }
  addEventListener("resize", resize);

  function startConfetti() {
    canvas.style.display = "block";
    resize();

    pieces = Array.from({ length: 240 }, () => ({
      x: Math.random() * innerWidth,
      y: -20 - Math.random() * 320,
      w: 6 + Math.random() * 7,
      h: 4 + Math.random() * 6,
      vx: -2 + Math.random() * 4,
      vy: 2 + Math.random() * 5,
      r: Math.random() * Math.PI,
      vr: -0.25 + Math.random() * 0.5,
      life: 190 + Math.random() * 100
    }));

    running = true;
    requestAnimationFrame(tick);
    setTimeout(() => {
      running = false;
      setTimeout(() => (canvas.style.display = "none"), 350);
    }, 3000);
  }

  function tick() {
    if (!running) return;
    ctx.clearRect(0, 0, innerWidth, innerHeight);

    pieces.forEach(p => {
      p.x += p.vx; p.y += p.vy; p.r += p.vr; p.life -= 1;
      ctx.save();
      ctx.translate(p.x, p.y);
      ctx.rotate(p.r);
      ctx.fillRect(-p.w/2, -p.h/2, p.w, p.h);
      ctx.restore();
    });

    pieces = pieces.filter(p => p.life > 0 && p.y < innerHeight + 60);
    if (pieces.length) requestAnimationFrame(tick);
  }

  // --- Typewriter + vibration ---
  function vibrate(ms) { if (navigator.vibrate) navigator.vibrate(ms); }

  function typeText(text, speed=45) {
    message.textContent = "";
    let i = 0;
    const timer = setInterval(() => {
      message.textContent += text[i++];
      if (i >= text.length) clearInterval(timer);
    }, speed);
  }

  yesBtn.addEventListener("click", () => {
    startConfetti();
    vibrate(60);
    setTimeout(() => vibrate(40), 120);
    typeText("I knew you’d say yes ❤️", 45);
  });

  // Initial setup
  moveNoRunAway();
  applyYesTransform();

})();
</script>
</body>
</html>

