<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Наш Котик: Музыкальная Студия</title>
  <!-- Tone.js для качественного звука и создания музыки -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
    body { font-family: 'Comic Sans MS', 'Chalkboard SE', cursive, sans-serif; background: #2c2c54; color: #fff; overflow: hidden; }
    
    #game-container { position: relative; width: 100vw; height: 100vh; display: flex; flex-direction: column; }
    canvas { width: 100%; flex: 1; display: block; background: #f7f1e3; }

    /* Верхнее меню */
    .top-bar {
      position: absolute; top: 12px; left: 12px; right: 12px;
      display: flex; justify-content: space-between; gap: 8px; z-index: 10;
    }
    .btn {
      background: #ffffff; color: #2d3436; border: 3px solid #ffb142; border-radius: 16px;
      padding: 8px 14px; font-weight: bold; font-size: 14px; cursor: pointer;
      box-shadow: 0 4px 10px rgba(0,0,0,0.15); transition: all 0.15s ease;
    }
    .btn:active { transform: scale(0.92); }

    .side-menu {
      position: absolute; right: 12px; top: 70px; display: flex; flex-direction: column; gap: 10px; z-index: 10;
    }

    /* Модуль Инструментов и Клавиатуры */
    .bottom-panel {
      background: #341f97; border-top: 5px solid #ffb142; padding: 10px;
      display: flex; flex-direction: column; gap: 8px;
    }
    .inst-selector { display: flex; justify-content: center; gap: 6px; overflow-x: auto; padding-bottom: 4px; }
    .inst-btn {
      background: #5f27cd; color: #fff; border: 2px solid #ff9ff3; padding: 6px 12px;
      border-radius: 12px; font-size: 12px; cursor: pointer; white-space: nowrap;
    }
    .inst-btn.active { background: #ff9ff3; color: #2d3436; font-weight: bold; }

    .keys-container { display: flex; height: 90px; gap: 4px; width: 100%; }
    .key {
      flex: 1; background: #ffffff; border-radius: 0 0 8px 8px; border: 2px solid #dcdde1;
      display: flex; align-items: flex-end; justify-content: center; padding-bottom: 8px;
      color: #2f3542; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 0 #cbd5e1;
    }
    .key:active, .key.active { background: #feca57; transform: translateY(3px); box-shadow: 0 1px 0 #cbd5e1; }

    /* Окна Модалок */
    .modal {
      position: absolute; top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.65); display: none; justify-content: center; align-items: center; z-index: 20;
    }
    .modal-content {
      background: #fff; color: #2d3436; border-radius: 24px; padding: 20px;
      width: 90%; max-width: 440px; max-height: 85vh; overflow-y: auto; text-align: center;
      border: 4px solid #ffb142;
    }
    .grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin: 15px 0; }
    .card { background: #f1f2f6; border-radius: 12px; padding: 10px; cursor: pointer; border: 2px solid transparent; font-size: 13px; }
    .card.selected { border-color: #ff4757; background: #ff788e22; }

    /* Секвенсор (Сам себе композитор) */
    .sequencer-grid { display: grid; grid-template-columns: repeat(8, 1fr); gap: 4px; margin: 15px 0; }
    .seq-step { height: 35px; background: #dfe4ea; border-radius: 6px; cursor: pointer; }
    .seq-step.active { background: #ff4757; }
    .seq-step.playing { border: 2px solid #1e90ff; }
  </style>
</head>
<body>

  <div id="game-container">
    <div class="top-bar">
      <button class="btn" onclick="openModal('wardrobe-modal')">👗 Одежда</button>
      <button class="btn" onclick="openModal('studio-modal')">🎼 Сочини тему</button>
      <button class="btn" onclick="toggleDance()">💃 Танец</button>
    </div>

    <div class="side-menu">
      <button class="btn" onclick="feedCat()">🍕 Еда</button>
      <button class="btn" onclick="giveGift()">🎁 Подарок</button>
    </div>

    <canvas id="stage"></canvas>

    <!-- Панель управления музыкой -->
    <div class="bottom-panel">
      <div class="inst-selector">
        <button class="inst-btn active" onclick="setInstrument('piano', this)">🎹 Пианино</button>
        <button class="inst-btn" onclick="setInstrument('guitar', this)">🎸 Гитара</button>
        <button class="inst-btn" onclick="setInstrument('harp', this)">🎵 Арфа</button>
        <button class="inst-btn" onclick="setInstrument('flute', this)">🪈 Флейта</button>
      </div>
      <div class="keys-container" id="keyboard"></div>
    </div>
  </div>

  <!-- Гардероб -->
  <div class="modal" id="wardrobe-modal">
    <div class="modal-content">
      <h3>Гардероб Котика</h3>
      <p style="margin-top:10px; font-weight:bold;">Наряды:</p>
      <div class="grid">
        <div class="card selected" onclick="setOutfit('tuxedo', this)">🤵 Смокинг</div>
        <div class="card" onclick="setOutfit('vest', this)">🥼 Жилетка</div>
        <div class="card" onclick="setOutfit('dress', this)">👗 Платьице</div>
        <div class="card" onclick="setOutfit('rock', this)">🎸 Рокер</div>
      </div>
      <p style="font-weight:bold;">Принты / Узоры:</p>
      <div class="grid">
        <div class="card selected" onclick="setPattern('none', this)">Без принта</div>
        <div class="card" onclick="setPattern('leopard', this)">🐆 Леопард</div>
        <div class="card" onclick="setPattern('cheetah', this)">🎷 Гепард</div>
        <div class="card" onclick="setPattern('snake', this)">🐍 Змея</div>
        <div class="card" onclick="setPattern('hearts', this)">❤️ Сердечки</div>
        <div class="card" onclick="setPattern('flowers', this)">🌸 Цветы</div>
        <div class="card" onclick="setPattern('zebra', this)">🦓 Зебра</div>
      </div>
      <button class="btn" onclick="closeModal('wardrobe-modal')">Готово</button>
    </div>
  </div>

  <!-- Студия Своей Музыки -->
  <div class="modal" id="studio-modal">
    <div class="modal-content">
      <h3>Студия Композитора</h3>
      <p style="font-size: 12px; margin-bottom: 10px;">Создай свою мелодию! Нажимай на ячейки:</p>
      <div class="sequencer-grid" id="sequencer"></div>
      <div style="display:flex; justify-content:center; gap:10px;">
        <button class="btn" onclick="playCustomMelody()">▶️ Запустить</button>
        <button class="btn" onclick="stopCustomMelody()">⏹️ Стоп</button>
      </div>
      <br>
      <button class="btn" onclick="closeModal('studio-modal')">Закрыть</button>
    </div>
  </div>

<script>
  // ИНИЦИАЛИЗАЦИЯ Tone.js
  let currentSynth;
  const synths = {
    piano: new Tone.PolySynth(Tone.Synth, { oscillator: { type: "triangle" } }).toDestination(),
    guitar: new Tone.PolySynth(Tone.Synth, { oscillator: { type: "sawtooth" }, envelope: { attack: 0.01, decay: 0.3, sustain: 0.2, release: 0.8 } }).toDestination(),
    harp: new Tone.PolySynth(Tone.Synth, { oscillator: { type: "sine" }, envelope: { attack: 0.05, decay: 1, sustain: 0.4, release: 1.2 } }).toDestination(),
    flute: new Tone.PolySynth(Tone.Synth, { oscillator: { type: "sine" }, envelope: { attack: 0.1, decay: 0.2, sustain: 0.8, release: 0.3 } }).toDestination()
  };
  currentSynth = synths.piano;

  // СОСТОЯНИЕ ИГРЫ
  const cat = {
    x: 0, y: 0,
    outfit: 'tuxedo',
    pattern: 'none',
    isDancing: false,
    danceTime: 0,
    action: null, // 'eating', 'gift'
    actionText: '',
    actionTimer: 0
  };

  const scale = ["C4", "D4", "E4", "F4", "G4", "A4", "B4", "C5"];

  // Построение Клавиатуры
  const kbEl = document.getElementById('keyboard');
  scale.forEach((note) => {
    const k = document.createElement('div');
    k.className = 'key';
    k.innerText = note;
    k.onmousedown = k.ontouchstart = (e) => {
      e.preventDefault();
      playNote(note);
      k.classList.add('active');
    };
    k.onmouseup = k.ontouchend = () => k.classList.remove('active');
    kbEl.appendChild(k);
  });

  function playNote(note) {
    Tone.start();
    currentSynth.triggerAttackRelease(note, "8n");
    catBounce();
  }

  function setInstrument(type, btn) {
    document.querySelectorAll('.inst-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    currentSynth = synths[type];
  }

  // СЕКВЕНСОР (Создание своей музыки)
  const seqGrid = document.getElementById('sequencer');
  const matrix = Array(4).fill().map(() => Array(8).fill(false)); // 4 ноты x 8 шагов

  for (let r = 0; r < 4; r++) {
    for (let c = 0; c < 8; c++) {
      const cell = document.createElement('div');
      cell.className = 'seq-step';
      cell.onclick = () => {
        matrix[r][c] = !matrix[r][c];
        cell.classList.toggle('active', matrix[r][c]);
      };
      seqGrid.appendChild(cell);
    }
  }

  let seqLoop;
  function playCustomMelody() {
    Tone.start();
    let step = 0;
    if (seqLoop) seqLoop.dispose();
    
    seqLoop = new Tone.Loop((time) => {
      for (let r = 0; r < 4; r++) {
        if (matrix[r][step]) {
          currentSynth.triggerAttackRelease(scale[r * 2], "8n", time);
        }
      }
      Tone.Draw.schedule(() => {
        cat.danceTime += 0.5;
      }, time);
      step = (step + 1) % 8;
    }, "8n").start(0);

    Tone.Transport.start();
    cat.isDancing = true;
  }

  function stopCustomMelody() {
    Tone.Transport.stop();
    cat.isDancing = false;
  }

  // canvas РЕНДЕРИНГ КОТИКА (В точном стиле с артов)
  const canvas = document.getElementById('stage');
  const ctx = canvas.getContext('2d');

  function resize() {
    canvas.width = canvas.parentElement.clientWidth;
    canvas.height = canvas.parentElement.clientHeight;
    cat.x = canvas.width / 2;
    cat.y = canvas.height / 2 + 30;
  }
  window.addEventListener('resize', resize);
  resize();

  let bounce = 0;
  function catBounce() { bounce = 12; }

  function drawCat() {
    ctx.save();
    
    let danceOffset = 0;
    let rotation = 0;
    if (cat.isDancing) {
      cat.danceTime += 0.08;
      danceOffset = Math.sin(cat.danceTime * 5) * 15;
      rotation = Math.sin(cat.danceTime * 3) * 0.08;
    }

    if (bounce > 0) bounce -= 1;

    ctx.translate(cat.x, cat.y - bounce + danceOffset);
    ctx.rotate(rotation);

    // 1. Хвост
    ctx.strokeStyle = '#3d1e6d'; ctx.lineWidth = 14; ctx.lineCap = 'round';
    ctx.beginPath(); ctx.moveTo(25, 40); ctx.quadraticCurveTo(60, 20, 50, -20); ctx.stroke();

    // 2. Ушки
    // Левое ушко (Трёхцветное)
    ctx.fillStyle = '#e17055';
    ctx.beginPath(); ctx.moveTo(-45, -70); ctx.lineTo(-20, -115); ctx.lineTo(-5, -70); ctx.fill();
    ctx.fillStyle = '#ff7675';
    ctx.beginPath(); ctx.moveTo(-38, -73); ctx.lineTo(-20, -103); ctx.lineTo(-10, -73); ctx.fill();

    // Правое ушко
    ctx.fillStyle = '#2d3436';
    ctx.beginPath(); ctx.moveTo(45, -70); ctx.lineTo(20, -115); ctx.lineTo(5, -70); ctx.fill();
    ctx.fillStyle = '#ff7675';
    ctx.beginPath(); ctx.moveTo(38, -73); ctx.lineTo(20, -103); ctx.lineTo(10, -73); ctx.fill();

    // 3. Голова (Овальная Chibi)
    ctx.fillStyle = '#e17055'; // Рыжеватый
    ctx.beginPath(); ctx.arc(0, -50, 55, 0, Math.PI * 2); ctx.fill();

    // Бело-тёмные пятна на мордочке
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.ellipse(-15, -40, 25, 20, 0, 0, Math.PI * 2); ctx.fill();
    ctx.beginPath(); ctx.ellipse(15, -40, 25, 20, 0, 0, Math.PI * 2); ctx.fill();

    ctx.fillStyle = '#2d3436'; // Тёмное пятно над глазами
    ctx.beginPath(); ctx.ellipse(-25, -65, 18, 12, -0.2, 0, Math.PI * 2); ctx.fill();

    // 4. Глаза (Большие выразительные)
    ctx.fillStyle = '#2d3436';
    ctx.beginPath(); ctx.arc(-18, -52, 9, 0, Math.PI * 2); ctx.fill();
    ctx.beginPath(); ctx.arc(18, -52, 9, 0, Math.PI * 2); ctx.fill();

    // Блики в глазах
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(-21, -55, 3, 0, Math.PI * 2); ctx.fill();
    ctx.beginPath(); ctx.arc(15, -55, 3, 0, Math.PI * 2); ctx.fill();

    // Носик и ротик
    ctx.fillStyle = '#ff7675';
    ctx.beginPath(); ctx.arc(0, -42, 4, 0, Math.PI * 2); ctx.fill();

    // 5. Тело и Одежда
    ctx.fillStyle = '#ffffff'; // Белая грудка
    ctx.beginPath(); ctx.ellipse(0, 25, 32, 42, 0, 0, Math.PI * 2); ctx.fill();

    if (cat.outfit === 'tuxedo') {
      // Смокинг с бабочкой
      ctx.fillStyle = '#2d3436';
      ctx.beginPath(); ctx.ellipse(-20, 25, 18, 38, -0.2, 0, Math.PI * 2); ctx.fill();
      ctx.beginPath(); ctx.ellipse(20, 25, 18, 38, 0.2, 0, Math.PI * 2); ctx.fill();
      // Бабочка
      ctx.fillStyle = '#d63031';
      ctx.beginPath(); ctx.arc(0, -8, 5, 0, Math.PI * 2); ctx.fill();
      ctx.beginPath(); ctx.moveTo(0, -8); ctx.lineTo(-12, -14); ctx.lineTo(-12, -2); ctx.fill();
      ctx.beginPath(); ctx.moveTo(0, -8); ctx.lineTo(12, -14); ctx.lineTo(12, -2); ctx.fill();
    } else if (cat.outfit === 'dress') {
      ctx.fillStyle = '#ff7675';
      ctx.beginPath(); ctx.moveTo(-15, 0); ctx.lineTo(15, 0); ctx.lineTo(35, 55); ctx.lineTo(-35, 55); ctx.fill();
    }

    // Принты на одежде
    if (cat.pattern !== 'none') {
      ctx.fillStyle = '#000000';
      ctx.font = '12px sans-serif';
      const icons = { leopard: '🐆', cheetah: '🎷', snake: '🐍', hearts: '❤️', flowers: '🌸', zebra: '🦓' };
      ctx.fillText(icons[cat.pattern] || '', -10, 25);
      ctx.fillText(icons[cat.pattern] || '', 5, 40);
    }

    // Лапки
    ctx.fillStyle = '#ffffff';
    ctx.beginPath(); ctx.arc(-22, 65, 12, 0, Math.PI * 2); ctx.fill();
    ctx.beginPath(); ctx.arc(22, 65, 12, 0, Math.PI * 2); ctx.fill();

    // Текст событий
    if (cat.actionTimer > 0) {
      cat.actionTimer--;
      ctx.fillStyle = '#2d3436';
      ctx.font = 'bold 18px "Comic Sans MS"';
      ctx.fillText(cat.actionText, -60, -125);
    }

    ctx.restore();
  }

  function loop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Фон комнаты
    ctx.fillStyle = '#f7f1e3'; ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = '#dcdde1'; ctx.fillRect(0, canvas.height/2 + 80, canvas.width, canvas.height/2);

    drawCat();
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);

  // ВЗАИМОДЕЙСТВИЯ
  function openModal(id) { document.getElementById(id).style.display = 'flex'; }
  function closeModal(id) { document.getElementById(id).style.display = 'none'; }

  function setOutfit(type, el) {
    cat.outfit = type;
    el.parentElement.querySelectorAll('.card').forEach(c => c.classList.remove('selected'));
    el.classList.add('selected');
  }

  function setPattern(type, el) {
    cat.pattern = type;
    el.parentElement.querySelectorAll('.card').forEach(c => c.classList.remove('selected'));
    el.classList.add('selected');
  }

  function toggleDance() {
    cat.isDancing = !cat.isDancing;
    if (cat.isDancing) {
      Tone.start();
      // Включаем танцевальную фоновую дорожку
      const loop = new Tone.Pattern((time, note) => {
        synths.piano.triggerAttackRelease(note, "10n", time);
      }, ["C4", "E4", "G4", "B4", "A4", "F4"], "upPattern");
      loop.interval = "8n";
      loop.start(0);
      Tone.Transport.start();
    } else {
      Tone.Transport.stop();
    }
  }

  function feedCat() {
    const list = ['🍕 Пицца', '🍣 Суши', '🍦 Мороженое', '🦈 Акула', '🍔 Бургер', '🐟 Рыба-петушок'];
    cat.actionText = 'Ням! ' + list[Math.floor(Math.random() * list.length)];
    cat.actionTimer = 120;
    Tone.start();
    synths.piano.triggerAttackRelease("G5", "8n");
  }

  function giveGift() {
    const gifts = ['🎁 Коробка с бантом', '💐 Цветы', '🎂 Торт', '🪳 Таракан Вася'];
    cat.actionText = 'Тебе: ' + gifts[Math.floor(Math.random() * gifts.length)];
    cat.actionTimer = 120;
    Tone.start();
    synths.harp.triggerAttackRelease("C5", "8n");
  }
</script>
</body>
</html>
