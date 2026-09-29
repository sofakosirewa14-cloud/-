<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Наш Котик</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #222; color: #fff; overflow: hidden; }
    
    /* Стартовый экран */
    #start-screen {
      position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
      background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%, #fecfef 100%);
      display: flex; flex-direction: column; justify-content: center; align-items: center;
      z-index: 100;
    }
    .start-btn {
      font-size: 80px; background: #fff; border: 4px solid #ff6b6b; border-radius: 50%;
      width: 140px; height: 140px; cursor: pointer; display: flex; justify-content: center;
      align-items: center; box-shadow: 0 10px 25px rgba(0,0,0,0.2); transition: transform 0.2s;
    }
    .start-btn:active { transform: scale(0.9); }
    .start-title { font-size: 28px; color: #d63031; margin-top: 20px; font-weight: bold; }

    /* Главный контейнер */
    #game-container {
      position: relative; width: 100vw; height: 100vh; display: flex; flex-direction: column;
    }

    /* Холст сцены */
    canvas { width: 100%; flex: 1; background: #87ceeb; display: block; }

    /* Панель управления (Верхняя и Боковые) */
    .top-bar {
      position: absolute; top: 10px; left: 10px; right: 10px;
      display: flex; justify-content: space-between; gap: 5px; z-index: 10;
    }
    .btn {
      background: rgba(255, 255, 255, 0.9); color: #333; border: none; padding: 8px 12px;
      border-radius: 12px; font-weight: bold; font-size: 14px; cursor: pointer;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1); display: flex; align-items: center; gap: 5px;
    }
    .btn:active { transform: scale(0.95); }

    .actions-menu {
      position: absolute; right: 10px; top: 60px; display: flex; flex-direction: column; gap: 10px; z-index: 10;
    }

    /* Оверлей Пианино / Инструментов */
    .instrument-bar {
      height: 120px; background: #333; display: flex; position: relative; border-top: 4px solid #555;
    }
    .key {
      flex: 1; background: white; border: 1px solid #ccc; border-radius: 0 0 5px 5px;
      margin: 0 2px; cursor: pointer; display: flex; align-items: flex-end; justify-content: center;
      padding-bottom: 10px; color: #333; font-weight: bold; font-size: 12px;
    }
    .key:active, .key.active { background: #ffeaa7; transform: translateY(2px); }

    /* Модальные окна (Гардероб, Конструктор Фона) */
    .modal {
      position: absolute; top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.7); display: none; flex-direction: column;
      justify-content: center; align-items: center; z-index: 20; padding: 20px;
    }
    .modal-content {
      background: #fff; color: #333; border-radius: 20px; padding: 20px;
      width: 90%; max-width: 400px; max-height: 80vh; overflow-y: auto; text-align: center;
    }
    .grid-options { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin: 15px 0; }
    .option-card {
      background: #f0f0f0; border-radius: 10px; padding: 10px; cursor: pointer; border: 2px solid transparent;
    }
    .option-card.selected { border-color: #ff6b6b; background: #ffe3e3; }

    /* Таймер танца */
    #dance-timer {
      position: absolute; top: 60px; left: 50%; transform: translateX(-50%);
      font-size: 24px; font-weight: bold; background: rgba(0,0,0,0.6); padding: 5px 15px;
      border-radius: 20px; display: none; z-index: 10;
    }
  </style>
</head>
<body>

  <!-- Стартовый экран -->
  <div id="start-screen">
    <div class="start-btn" id="start-btn">😺</div>
    <div class="start-title">Наш Котик</div>
  </div>

  <!-- Главный экран игры -->
  <div id="game-container">
    <div class="top-bar">
      <button class="btn" onclick="toggleModal('wardrobe-modal')">👗 Гардероб</button>
      <button class="btn" onclick="toggleModal('bg-modal')">🎨 Фон</button>
      <button class="btn" onclick="changeInstrument()">🎵 <span id="inst-label">Пианино</span></button>
      <button class="btn" id="mic-btn" onclick="toggleMic()">🎙️ Голос</button>
    </div>

    <div class="actions-menu">
      <button class="btn" onclick="startDance()">💃 Танец</button>
      <button class="btn" onclick="feedCat()">🐟 Еда</button>
      <button class="btn" onclick="getSelfGift()">🎁 Себе</button>
      <button class="btn" onclick="giveViewerGift()">💝 Зрителю</button>
    </div>

    <div id="dance-timer">15</div>

    <canvas id="scene"></canvas>

    <!-- Клавиатура музыкального инструмента -->
    <div class="instrument-bar" id="keyboard">
      <!-- Генерируется в JS -->
    </div>
  </div>

  <!-- Модалка Гардероба -->
  <div class="modal" id="wardrobe-modal">
    <div class="modal-content">
      <h3>Гардероб и Принты</h3>
      <p style="margin-top:10px; font-weight:bold;">Принты одежды:</p>
      <div class="grid-options">
        <div class="option-card" onclick="setPattern('none')">Без принта</div>
        <div class="option-card" onclick="setPattern('leopard')">🐆 Леопард</div>
        <div class="option-card" onclick="setPattern('cheetah')">🎷 Гепард</div>
        <div class="option-card" onclick="setPattern('snake')">🐍 Змея</div>
        <div class="option-card" onclick="setPattern('hearts')">❤️ Сердечки</div>
        <div class="option-card" onclick="setPattern('flowers')">🌸 Цветы</div>
        <div class="option-card" onclick="setPattern('zebra')">🦓 Зебра</div>
        <div class="option-card" onclick="setPattern('fish')">🐟 Рыбки</div>
      </div>
      <button class="btn" style="margin: 0 auto;" onclick="toggleModal('wardrobe-modal')">Закрыть</button>
    </div>
  </div>

  <!-- Модалка Конструктора Фона -->
  <div class="modal" id="bg-modal">
    <div class="modal-content">
      <h3>Создание фона</h3>
      <div class="grid-options">
        <div class="option-card" onclick="setPresetBG('home')">🏠 Дом</div>
        <div class="option-card" onclick="setPresetBG('street')">🌆 Улица</div>
        <div class="option-card" onclick="setPresetBG('park')">🌳 Парк</div>
        <div class="option-card" onclick="setPresetBG('club')">🪩 Клуб</div>
      </div>
      <p style="margin-top:10px; font-weight:bold;">Детали:</p>
      <div class="grid-options">
        <div class="option-card" onclick="toggleBGDetail('sun')">☀️ Солнце/Луна</div>
        <div class="option-card" onclick="toggleBGDetail('clouds')">☁️ Облака</div>
        <div class="option-card" onclick="toggleBGDetail('bench')">🪑 Лавочка</div>
        <div class="option-card" onclick="toggleBGDetail('lights')">💡 Фонари</div>
      </div>
      <button class="btn" style="margin: 0 auto;" onclick="toggleModal('bg-modal')">Закрыть</button>
    </div>
  </div>

<script>
  // === ЗВУКОВОЙ ДВИЖОК (Web Audio API) ===
  const AudioCtx = new (window.AudioContext || window.webkitAudioContext)();
  
  function playNote(freq, type = 'piano') {
    if (AudioCtx.state === 'suspended') AudioCtx.resume();
    const osc = AudioCtx.createOscillator();
    const gain = AudioCtx.createGain();
    osc.connect(gain);
    gain.connect(AudioCtx.destination);

    // Выбор инструмента
    if (type === 'piano') osc.type = 'triangle';
    else if (type === 'guitar') osc.type = 'sawtooth';
    else if (type === 'flute') osc.type = 'sine';
    else if (type === 'harp') osc.type = 'sine';
    else osc.type = 'square';

    osc.frequency.setValueAtTime(freq, AudioCtx.currentTime);
    gain.gain.setValueAtTime(0.5, AudioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, AudioCtx.currentTime + 1.2);

    osc.start();
    osc.stop(AudioCtx.currentTime + 1.2);

    // Котик мяукает под ноту
    catMeow(freq);
  }

  function catMeow(pitch) {
    const osc = AudioCtx.createOscillator();
    const gain = AudioCtx.createGain();
    osc.type = 'sine';
    osc.connect(gain);
    gain.connect(AudioCtx.destination);
    
    // Эмуляция интонации Мяу
    osc.frequency.setValueAtTime(pitch * 1.5, AudioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(pitch * 0.8, AudioCtx.currentTime + 0.3);
    
    gain.gain.setValueAtTime(0.1, AudioCtx.currentTime);
    gain.gain.linearRampToValueAtTime(0.001, AudioCtx.currentTime + 0.3);

    osc.start();
    osc.stop(AudioCtx.currentTime + 0.3);
  }

  // Списки продуктов, голосов, инструментов
  const foods = [
    '🍦 Мороженое', '🥧 Пирог', '🍲 Суп', '🍔 Бургер', '🍭 Леденец', '🍟 Картошка фри',
    '🍣 Суши', '🍕 Пицца', '🍺 Квас', '🍞 Хлеб', '🐟 Рыба', '🍕 Скат', '🐋 Манта',
    '🦈 Акула', '🐡 Рыба-фугу', '🐠 Рыба-петушок', '🐋 Синий кит', '🐳 Косатка',
    '🎣 Удильщик', '🦖 Динозавр', '🦁 Крылатка', '🦁 Рыба-лев', '🐆 Пантера',
    '🐆 Чёрный Ягуар', '🦐 Креветка', '🦀 Краб', ' кальмар', '🐙 Осьминог', '🧀 Сыр'
  ];

  const voices = ['Качок', 'Принцесса', 'Младенец', 'Старик', 'Вампир', 'Микки Маус', 'Хатсунэ Мику', 'Робот', 'Инопланетянин', 'Гном', 'Дракон', 'Эльф'];
  const instruments = ['Пианино', 'Флейта Ди Цзи', 'Барабаны', 'Арфа', 'Гитара', 'Балалайка', 'Гуцинь', 'Пипа'];
  let currentInstIndex = 0;

  // Игровое состояние Котика
  const cat = {
    x: 0, y: 0, scale: 1,
    pattern: 'none',
    isDancing: false,
    danceFrame: 0,
    currentAction: null, // 'eating', 'gift_self', 'gift_viewer', 'sad'
    actionItem: '',
    actionTimer: 0
  };

  // Конструктор фона
  const bgConfig = {
    type: 'home',
    details: { sun: true, clouds: true, bench: false, lights: false }
  };

  // Canvas
  const canvas = document.getElementById('scene');
  const ctx = canvas.getContext('2d');

  function resize() {
    canvas.width = canvas.parentElement.clientWidth;
    canvas.height = canvas.parentElement.clientHeight;
    cat.x = canvas.width / 2;
    cat.y = canvas.height / 2 + 50;
  }
  window.addEventListener('resize', resize);

  // Клавиатура
  const notes = [
    { name: 'До', freq: 261.63 }, { name: 'Ре', freq: 293.66 },
    { name: 'Ми', freq: 329.63 }, { name: 'Фа', freq: 349.23 },
    { name: 'Соль', freq: 392.00 }, { name: 'Ля', freq: 440.00 },
    { name: 'Си', freq: 493.88 }, { name: 'До2', freq: 523.25 }
  ];

  function renderKeyboard() {
    const kb = document.getElementById('keyboard');
    kb.innerHTML = '';
    notes.forEach(note => {
      const key = document.createElement('div');
      key.className = 'key';
      key.innerText = note.name;
      key.onclick = () => playNote(note.freq, getInstType());
      kb.appendChild(key);
    });
  }

  function getInstType() {
    const name = instruments[currentInstIndex];
    if (name.includes('Флейта')) return 'flute';
    if (name.includes('Гитара') || name.includes('Балалайка')) return 'guitar';
    if (name.includes('Арфа') || name.includes('Гуцинь')) return 'harp';
    return 'piano';
  }

  function changeInstrument() {
    currentInstIndex = (currentInstIndex + 1) % instruments.length;
    document.getElementById('inst-label').innerText = instruments[currentInstIndex];
  }

  // Вызов стартового экрана
  document.getElementById('start-btn').onclick = () => {
    document.getElementById('start-screen').style.display = 'none';
    AudioCtx.resume();
    resize();
    renderKeyboard();
    requestAnimationFrame(gameLoop);
  };

  // МОДАЛКИ
  function toggleModal(id) {
    const el = document.getElementById(id);
    el.style.display = el.style.display === 'flex' ? 'none' : 'flex';
  }

  function setPattern(p) { cat.pattern = p; }
  function setPresetBG(type) { bgConfig.type = type; }
  function toggleBGDetail(d) { bgConfig.details[d] = !bgConfig.details[d]; }

  // МИКРОФОН (Заглушка повторялки)
  let micActive = false;
  function toggleMic() {
    micActive = !micActive;
    const btn = document.getElementById('mic-btn');
    if (micActive) {
      btn.style.background = '#ff7675';
      const randomVoice = voices[Math.floor(Math.random() * voices.length)];
      alert('Микрофон включен! Голос повтора: ' + randomVoice);
    } else {
      btn.style.background = 'rgba(255, 255, 255, 0.9)';
    }
  }

  // ТАНЕЦ (15 сек)
  let danceInterval;
  function startDance() {
    if (cat.isDancing) return;
    cat.isDancing = true;
    let timeLeft = 15;
    const timerEl = document.getElementById('dance-timer');
    timerEl.style.display = 'block';
    timerEl.innerText = timeLeft;

    // Воспроизведение случайной танцевальной мелодии
    const melodyInterval = setInterval(() => {
      if (!cat.isDancing) return;
      playNote(200 + Math.random() * 400, getInstType());
    }, 300);

    danceInterval = setInterval(() => {
      timeLeft--;
      timerEl.innerText = timeLeft;
      if (timeLeft <= 0) {
        clearInterval(danceInterval);
        clearInterval(melodyInterval);
        cat.isDancing = false;
        timerEl.style.display = 'none';
      }
    }, 1000);
  }

  // ЕДА
  function feedCat() {
    cat.currentAction = 'eating';
    cat.actionItem = foods[Math.floor(Math.random() * foods.length)];
    cat.actionTimer = 180; // Фреймы
  }

  // ПОДАРКИ СЕБЕ
  function getSelfGift() {
    const items = ['💍 Кольцо', '💐 Цветы', '🎂 Торт', '🪳 Таракан Вася', '🚗 Машинка', '🏠 Дом'];
    cat.currentAction = 'gift_self';
    cat.actionItem = items[Math.floor(Math.random() * items.length)];
    cat.actionTimer = 200;
  }

  // ПОДАРКИ ЗРИТЕЛЮ
  let viewerGiftTimeout;
  function giveViewerGift() {
    cat.currentAction = 'gift_viewer';
    cat.actionItem = '🎁 Коробка с бантом';
    cat.actionTimer = 300; // 5 секунд на нажатие (60fps * 5)
  }

  // Клик по Canvas для взаимодействия с подарком
  canvas.onclick = (e) => {
    if (cat.currentAction === 'gift_viewer') {
      // Игрок успел нажать!
      cat.currentAction = 'gift_opened';
      const effects = ['💍 Кольцо исчезло со вспышкой!', '💐 Цветы выросли и улетели!', '🎂 Торт шлепнулся на котика!'];
      cat.actionItem = effects[Math.floor(Math.random() * effects.length)];
      if (cat.actionItem.includes('Торт')) {
        cat.currentAction = 'cake_splat';
      }
      cat.actionTimer = 150;
    }
  };

  // === ОТРИСОВКА (RENDER LOOP) ===
  function drawBackground() {
    // Базовый цвет
    if (bgConfig.type === 'home') ctx.fillStyle = '#ffeaa7';
    else if (bgConfig.type === 'street') ctx.fillStyle = '#74b9ff';
    else if (bgConfig.type === 'park') ctx.fillStyle = '#55efc4';
    else if (bgConfig.type === 'club') ctx.fillStyle = '#2d3436';
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Отрисовка деталей
    if (bgConfig.details.sun) {
      ctx.fillStyle = bgConfig.type === 'club' ? '#fdcb6e' : '#f1c40f';
      ctx.beginPath(); ctx.arc(canvas.width - 60, 60, 30, 0, Math.PI * 2); ctx.fill();
    }
    if (bgConfig.details.clouds) {
      ctx.fillStyle = 'rgba(255,255,255,0.8)';
      ctx.beginPath(); ctx.arc(100, 80, 20, 0, Math.PI * 2); ctx.fill();
      ctx.beginPath(); ctx.arc(120, 80, 25, 0, Math.PI * 2); ctx.fill();
    }
    // Пол / Дорожка
    ctx.fillStyle = bgConfig.type === 'club' ? '#0984e3' : '#dfe6e9';
    ctx.fillRect(0, canvas.height / 2 + 100, canvas.width, canvas.height / 2);
  }

  function drawCat() {
    ctx.save();
    ctx.translate(cat.x, cat.y);

    // Анимация танца
    if (cat.isDancing) {
      cat.danceFrame += 0.15;
      ctx.translate(Math.sin(cat.danceFrame) * 15, Math.abs(Math.cos(cat.danceFrame)) * -20);
      ctx.rotate(Math.sin(cat.danceFrame) * 0.1);
    }

    // Тело Котика (Трехцветный Chibi)
    // Уши
    ctx.fillStyle = '#d63031';
    ctx.beginPath(); ctx.moveTo(-35, -60); ctx.lineTo(-15, -90); ctx.lineTo(-5, -60); ctx.fill();
    ctx.beginPath(); ctx.moveTo(35, -60); ctx.lineTo(15, -90); ctx.lineTo(5, -60); ctx.fill();

    // Голова
    ctx.fillStyle = '#e17055';
    ctx.beginPath(); ctx.arc(0, -40, 45, 0, Math.PI * 2); ctx.fill();
    
    // Мордочка (Белое пятнышко)
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(0, -30, 25, 0, Math.PI * 2); ctx.fill();

    // Глаза
    ctx.fillStyle = '#2d3436';
    ctx.beginPath(); ctx.arc(-12, -42, 6, 0, Math.PI * 2); ctx.fill();
    ctx.beginPath(); ctx.arc(12, -42, 6, 0, Math.PI * 2); ctx.fill();

    // Носик и Рот
    ctx.fillStyle = '#ff7675';
    ctx.beginPath(); ctx.arc(0, -32, 3, 0, Math.PI * 2); ctx.fill();

    // Туловище + Принт Одежды
    ctx.fillStyle = '#e17055';
    ctx.beginPath(); ctx.ellipse(0, 25, 30, 40, 0, 0, Math.PI * 2); ctx.fill();

    if (cat.pattern !== 'none') {
      ctx.fillStyle = cat.pattern === 'hearts' ? '#ff4757' : (cat.pattern === 'zebra' ? '#000' : '#f1c40f');
      ctx.font = '12px Arial';
      const icon = { leopard: '🐆', cheetah: '🎷', snake: '🐍', hearts: '❤️', flowers: '🌸', zebra: '🦓', fish: '🐟' }[cat.pattern] || '✨';
      ctx.fillText(icon, -10, 25);
      ctx.fillText(icon, 5, 35);
    }

    // Лапки
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(-20, 60, 10, 0, Math.PI * 2); ctx.fill();
    ctx.beginPath(); ctx.arc(20, 60, 10, 0, Math.PI * 2); ctx.fill();

    // Анимация Действий / Еды / Подарков
    if (cat.actionTimer > 0) {
      cat.actionTimer--;
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 16px Arial';
      
      if (cat.currentAction === 'eating') {
        // Котик лапой заводит за спину и ест
        ctx.fillText('Ням-ням! ' + cat.actionItem, -50, -100);
      } else if (cat.currentAction === 'gift_self') {
        ctx.fillText('Подарок мне: ' + cat.actionItem, -60, -100);
      } else if (cat.currentAction === 'gift_viewer') {
        ctx.fillText('Тебе подарок! (Нажми!) ' + cat.actionItem, -80, -100);
        if (cat.actionTimer === 1) {
          // Игрок не успел нажать за 5 сек
          cat.currentAction = 'sad';
          cat.actionTimer = 120;
        }
      } else if (cat.currentAction === 'sad') {
        ctx.fillText('😿 Ты не взял подарок...', -70, -100);
      } else if (cat.currentAction === 'cake_splat') {
        ctx.fillText('🎂 БУМ! Торт на личике!', -70, -100);
        ctx.fillStyle = '#ffeaa7';
        ctx.beginPath(); ctx.arc(0, -40, 30, 0, Math.PI * 2); ctx.fill(); // Торт на лице
      } else {
        ctx.fillText(cat.actionItem, -60, -100);
      }
    }

    ctx.restore();
  }

  function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawBackground();
    drawCat();
    requestAnimationFrame(gameLoop);
  }
</script>
</body>
</html>
