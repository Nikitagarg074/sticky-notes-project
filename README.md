<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>PinBoard — Sticky Notes</title>
  <link href="https://fonts.googleapis.com/css2?family=Kalam:wght@300;400;700&family=Playfair+Display:wght@700;900&display=swap" rel="stylesheet" />
  <style>
    :root {
      --bg: #e8ddd0;
      --bg2: #d9cfc0;
      --board: #c8b89a;
      --header-bg: rgba(232,221,208,0.85);
      --text: #2c1a0e;
      --subtext: #7a6050;
      --border: rgba(0,0,0,0.1);
      --shadow: rgba(0,0,0,0.18);
      --pin: #c0392b;
      --btn-bg: #2c1a0e;
      --btn-text: #f5ede0;
      --toggle-bg: #f5c842;
      --toggle-knob: #fff;
    }
    [data-theme="dark"] {
      --bg: #0f0e17;
      --bg2: #1a1828;
      --board: #161424;
      --header-bg: rgba(15,14,23,0.9);
      --text: #ede8f5;
      --subtext: #8b85a0;
      --border: rgba(255,255,255,0.07);
      --shadow: rgba(0,0,0,0.55);
      --pin: #ff6b6b;
      --btn-bg: #ede8f5;
      --btn-text: #0f0e17;
      --toggle-bg: #5c4fff;
      --toggle-knob: #fff;
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Kalam', cursive;
      background: var(--bg);
      color: var(--text);
      min-height: 100vh;
      transition: background 0.5s, color 0.4s;
      overflow-x: hidden;
    }

    /* Cork texture overlay */
    body::before {
      content: '';
      position: fixed;
      inset: 0;
      pointer-events: none;
      z-index: 0;
      opacity: 0.35;
      background-image:
        radial-gradient(circle at 20% 30%, rgba(180,140,80,0.15) 0%, transparent 60%),
        radial-gradient(circle at 80% 70%, rgba(100,70,40,0.1) 0%, transparent 50%),
        url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='300' height='300'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.75' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='300' height='300' filter='url(%23n)' opacity='0.12'/%3E%3C/svg%3E");
    }

    [data-theme="dark"] body::before {
      background-image:
        radial-gradient(circle at 20% 30%, rgba(80,60,160,0.12) 0%, transparent 60%),
        radial-gradient(circle at 80% 70%, rgba(60,40,120,0.1) 0%, transparent 50%);
    }

    /* ─── HEADER ─── */
    header {
      position: sticky;
      top: 0;
      z-index: 100;
      background: var(--header-bg);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      border-bottom: 1px solid var(--border);
      padding: 0 32px;
      height: 68px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      transition: background 0.5s, border-color 0.4s;
    }

    .logo {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    .logo-pin {
      font-size: 28px;
      transform: rotate(-15deg);
      display: inline-block;
      filter: drop-shadow(0 2px 4px rgba(0,0,0,0.2));
    }
    .logo h1 {
      font-family: 'Playfair Display', serif;
      font-size: 22px;
      font-weight: 900;
      color: var(--text);
      letter-spacing: -0.03em;
      transition: color 0.4s;
    }
    .logo span {
      font-family: 'Kalam', cursive;
      font-size: 11px;
      font-weight: 300;
      color: var(--subtext);
      display: block;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      margin-top: -4px;
    }

    .header-actions {
      display: flex;
      align-items: center;
      gap: 14px;
    }

    .note-count {
      font-size: 13px;
      color: var(--subtext);
      font-family: 'Kalam', cursive;
      transition: color 0.4s;
    }

    /* Dark mode toggle */
    .toggle-wrap {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .toggle-label {
      font-size: 13px;
      color: var(--subtext);
      font-family: 'Kalam', cursive;
      transition: color 0.4s;
    }
    .toggle {
      position: relative;
      width: 52px;
      height: 26px;
      cursor: pointer;
    }
    .toggle input { display: none; }
    .toggle-track {
      position: absolute;
      inset: 0;
      border-radius: 13px;
      background: var(--toggle-bg);
      transition: background 0.4s;
      box-shadow: inset 0 1px 3px rgba(0,0,0,0.2);
    }
    .toggle-thumb {
      position: absolute;
      top: 3px;
      left: 3px;
      width: 20px;
      height: 20px;
      border-radius: 50%;
      background: var(--toggle-knob);
      transition: transform 0.35s cubic-bezier(.4,0,.2,1), background 0.3s;
      box-shadow: 0 2px 6px rgba(0,0,0,0.25);
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 11px;
    }
    .toggle input:checked ~ .toggle-thumb {
      transform: translateX(26px);
    }
    .toggle-icon { line-height: 1; pointer-events: none; }

    /* Add note button */
    .add-btn {
      height: 38px;
      padding: 0 18px;
      border-radius: 20px;
      border: none;
      background: var(--btn-bg);
      color: var(--btn-text);
      font-family: 'Kalam', cursive;
      font-size: 15px;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 6px;
      transition: background 0.4s, color 0.4s, transform 0.15s, box-shadow 0.15s;
      box-shadow: 0 3px 10px rgba(0,0,0,0.2);
    }
    .add-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 6px 18px rgba(0,0,0,0.25);
    }
    .add-btn:active { transform: translateY(0); }
    .add-btn .plus { font-size: 20px; line-height: 1; }

    /* ─── BOARD ─── */
    .board-wrap {
      position: relative;
      z-index: 1;
      min-height: calc(100vh - 68px);
      padding: 48px 32px 80px;
    }

    /* ─── NOTES GRID ─── */
    .notes-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
      gap: 36px 28px;
      max-width: 1280px;
      margin: 0 auto;
    }

    /* ─── NOTE CARD ─── */
    .note-card {
      position: relative;
      border-radius: 3px;
      padding: 38px 16px 14px;
      min-height: 200px;
      display: flex;
      flex-direction: column;
      box-shadow:
        4px 6px 22px var(--shadow),
        0 1px 3px rgba(0,0,0,0.12),
        inset 0 0 0 1px rgba(255,255,255,0.18);
      transition: transform 0.25s cubic-bezier(.4,0,.2,1), box-shadow 0.25s;
      animation: noteIn 0.35s cubic-bezier(.4,0,.2,1) both;
    }
    @keyframes noteIn {
      from { opacity: 0; transform: scale(0.85) rotate(var(--rot)) translateY(20px); }
      to   { opacity: 1; transform: scale(1)    rotate(var(--rot)) translateY(0); }
    }
    .note-card:hover {
      transform: rotate(0deg) scale(1.03) translateY(-4px) !important;
      box-shadow: 8px 16px 40px var(--shadow), 0 2px 6px rgba(0,0,0,0.15);
      z-index: 10;
    }

    /* Pin */
    .note-pin {
      position: absolute;
      top: -10px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 22px;
      filter: drop-shadow(0 3px 5px rgba(0,0,0,0.3));
      z-index: 2;
      pointer-events: none;
      line-height: 1;
    }

    /* Fold corner */
    .note-card::after {
      content: '';
      position: absolute;
      bottom: 0;
      right: 0;
      width: 22px;
      height: 22px;
      background: linear-gradient(225deg, rgba(0,0,0,0.12) 50%, transparent 50%);
      border-radius: 0 0 3px 0;
    }

    .note-textarea {
      flex: 1;
      background: transparent;
      border: none;
      outline: none;
      resize: none;
      font-family: 'Kalam', cursive;
      font-size: 16px;
      line-height: 1.65;
      color: inherit;
      width: 100%;
      min-height: 120px;
      letter-spacing: 0.01em;
    }
    .note-textarea::placeholder { opacity: 0.4; }

    /* Ruled lines */
    .note-lines {
      position: absolute;
      inset: 52px 12px 44px;
      pointer-events: none;
      z-index: 0;
      overflow: hidden;
    }
    .note-lines::before {
      content: '';
      position: absolute;
      inset: 0;
      background-image: repeating-linear-gradient(
        to bottom,
        transparent,
        transparent 26px,
        rgba(0,0,0,0.07) 26px,
        rgba(0,0,0,0.07) 27px
      );
    }

    /* Note footer */
    .note-footer {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-top: 10px;
      position: relative;
      z-index: 2;
    }

    .color-dots {
      display: flex;
      gap: 5px;
    }
    .color-dot {
      width: 13px;
      height: 13px;
      border-radius: 50%;
      cursor: pointer;
      border: 2px solid transparent;
      transition: transform 0.15s, border-color 0.15s;
    }
    .color-dot:hover { transform: scale(1.25); }
    .color-dot.active { border-color: rgba(0,0,0,0.5); }
    [data-theme="dark"] .color-dot.active { border-color: rgba(255,255,255,0.7); }

    .delete-btn {
      width: 24px;
      height: 24px;
      border-radius: 50%;
      border: none;
      background: rgba(0,0,0,0.1);
      color: inherit;
      font-size: 13px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      opacity: 0;
      transition: opacity 0.2s, background 0.2s, transform 0.15s;
    }
    .note-card:hover .delete-btn { opacity: 0.6; }
    .delete-btn:hover { opacity: 1 !important; background: rgba(200,50,50,0.25); transform: scale(1.15); }

    .note-date {
      font-size: 11px;
      opacity: 0.45;
      letter-spacing: 0.04em;
      margin-top: 2px;
    }

    /* ─── EMPTY STATE ─── */
    .empty-state {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 50vh;
      gap: 14px;
      opacity: 0;
      transition: opacity 0.4s;
    }
    .empty-state.visible { opacity: 1; }
    .empty-state .emoji { font-size: 56px; }
    .empty-state p {
      font-size: 20px;
      color: var(--subtext);
      transition: color 0.4s;
    }
    .empty-state small {
      font-size: 14px;
      color: var(--subtext);
      opacity: 0.6;
    }

    /* ─── HERO STRIP ─── */
    .hero {
      max-width: 1280px;
      margin: 0 auto 40px;
      padding: 0 4px;
      display: flex;
      align-items: baseline;
      gap: 12px;
    }
    .hero h2 {
      font-family: 'Playfair Display', serif;
      font-size: clamp(28px, 4vw, 42px);
      font-weight: 900;
      color: var(--text);
      transition: color 0.4s;
    }
    .hero h2 em {
      font-style: italic;
      color: var(--subtext);
      font-size: 0.7em;
    }

    /* Scrollbar */
    ::-webkit-scrollbar { width: 6px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }

    /* ─── RESPONSIVE ─── */
    @media (max-width: 600px) {
      header { padding: 0 16px; }
      .board-wrap { padding: 32px 16px 60px; }
      .logo span { display: none; }
      .note-count { display: none; }
    }
  </style>
</head>
<body>

<header>
  <div class="logo">
    <span class="logo-pin">📌</span>
    <div>
      <h1>PinBoard</h1>
      <span>your digital corkboard</span>
    </div>
  </div>

  <div class="header-actions">
    <span class="note-count" id="noteCount">0 notes</span>

    <div class="toggle-wrap">
      <span class="toggle-label" id="modeLabel">Light</span>
      <label class="toggle" title="Toggle dark mode">
        <input type="checkbox" id="darkToggle" />
        <div class="toggle-track"></div>
        <div class="toggle-thumb">
          <span class="toggle-icon" id="toggleIcon">☀️</span>
        </div>
      </label>
    </div>

    <button class="add-btn" onclick="addNote()">
      <span class="plus">+</span> New Note
    </button>
  </div>
</header>

<main class="board-wrap">
  <div class="hero">
    <h2>My Board <em>— thoughts, ideas & reminders</em></h2>
  </div>

  <div class="notes-grid" id="notesGrid"></div>

  <div class="empty-state" id="emptyState">
    <div class="emoji">🗒️</div>
    <p>Your board is empty</p>
    <small>Click <strong>+ New Note</strong> to get started</small>
  </div>
</main>

<script>
  // ─── CONFIG ───────────────────────────────────────────────
  const COLORS = [
    { id:'lemon',  light:'#FFF59D', dark:'#F9A825', dot:'#FDD835' },
    { id:'mint',   light:'#C8E6C9', dark:'#2E7D32', dot:'#66BB6A' },
    { id:'peach',  light:'#FFCCBC', dark:'#BF360C', dot:'#FF7043' },
    { id:'lavender',light:'#E1BEE7',dark:'#6A1B9A', dot:'#AB47BC' },
    { id:'sky',    light:'#B3E5FC', dark:'#01579B', dot:'#29B6F6' },
    { id:'rose',   light:'#F8BBD9', dark:'#880E4F', dot:'#EC407A' },
  ];

  const ROTATIONS = [-3.5, -2, -1, 0, 1, 2, 3.5, -2.5, 2.5];
  const PINS = ['📌','📍','🔴','🟡','🟢','🔵'];
  const PLACEHOLDERS = [
    'Jot something down…',
    'What\'s on your mind?',
    'A reminder, perhaps?',
    'Great idea goes here…',
    'Don\'t forget this!',
    'Note to self…',
  ];

  let notes = JSON.parse(localStorage.getItem('pinboard-notes') || '[]');
  let isDark = localStorage.getItem('pinboard-dark') === 'true';
  let nextId = parseInt(localStorage.getItem('pinboard-nextid') || '1');

  // ─── INIT ──────────────────────────────────────────────────
  applyTheme();
  renderAll();

  if (notes.length === 0) {
    // seed demo notes
    const demos = [
      { text: 'Welcome to PinBoard! ✨\nYour thoughts, beautifully organized.' , colorIdx: 0 },
      { text: 'Toggle dark mode with the switch above 🌙', colorIdx: 4 },
      { text: 'Click anywhere on a note to edit it ✏️', colorIdx: 2 },
      { text: 'Use the color dots to change each note\'s color 🎨', colorIdx: 1 },
    ];
    demos.forEach(d => addNote(d.text, d.colorIdx));
  }

  // ─── DARK MODE ────────────────────────────────────────────
  document.getElementById('darkToggle').checked = isDark;
  document.getElementById('darkToggle').addEventListener('change', function() {
    isDark = this.checked;
    localStorage.setItem('pinboard-dark', isDark);
    applyTheme();
    updateToggleLabel();
    recolorNotes();
  });
  updateToggleLabel();

  function applyTheme() {
    document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');
  }

  function updateToggleLabel() {
    document.getElementById('modeLabel').textContent = isDark ? 'Dark' : 'Light';
    document.getElementById('toggleIcon').textContent = isDark ? '🌙' : '☀️';
  }

  // ─── NOTES ───────────────────────────────────────────────
  function addNote(text = '', colorIdx = Math.floor(Math.random() * COLORS.length)) {
    const note = {
      id: nextId++,
      text,
      colorIdx,
      rotation: ROTATIONS[Math.floor(Math.random() * ROTATIONS.length)],
      pin: PINS[Math.floor(Math.random() * PINS.length)],
      placeholder: PLACEHOLDERS[Math.floor(Math.random() * PLACEHOLDERS.length)],
      created: new Date().toLocaleDateString('en-US', { month:'short', day:'numeric' }),
    };
    notes.push(note);
    save();
    renderNote(note, true);
    updateCount();
    hideEmpty();
  }

  function deleteNote(id) {
    const el = document.getElementById('note-' + id);
    if (el) {
      el.style.transition = 'transform 0.3s, opacity 0.3s';
      el.style.transform = 'scale(0.8) rotate(10deg)';
      el.style.opacity = '0';
      setTimeout(() => el.remove(), 300);
    }
    notes = notes.filter(n => n.id !== id);
    save();
    updateCount();
    checkEmpty();
  }

  function updateNoteText(id, text) {
    const note = notes.find(n => n.id === id);
    if (note) { note.text = text; save(); }
  }

  function updateNoteColor(id, colorIdx) {
    const note = notes.find(n => n.id === id);
    if (note) {
      note.colorIdx = colorIdx;
      save();
      const el = document.getElementById('note-' + id);
      if (el) {
        const color = COLORS[colorIdx];
        el.style.background = isDark ? color.dark : color.light;
        el.style.color = isDark ? '#fff' : '#2c1a0e';
        // update active dot
        el.querySelectorAll('.color-dot').forEach((dot, i) => {
          dot.classList.toggle('active', i === colorIdx);
        });
      }
    }
  }

  function recolorNotes() {
    notes.forEach(note => {
      const el = document.getElementById('note-' + note.id);
      if (el) {
        const color = COLORS[note.colorIdx];
        el.style.background = isDark ? color.dark : color.light;
        el.style.color = isDark ? '#fff' : '#2c1a0e';
      }
    });
  }

  function renderNote(note, animate = false) {
    const grid = document.getElementById('notesGrid');
    const color = COLORS[note.colorIdx];
    const bg = isDark ? color.dark : color.light;
    const textColor = isDark ? '#fff' : '#2c1a0e';
    const rot = note.rotation;

    const card = document.createElement('div');
    card.className = 'note-card';
    card.id = 'note-' + note.id;
    card.style.cssText = `background:${bg};color:${textColor};--rot:${rot}deg;transform:rotate(${rot}deg);`;

    // Color dots HTML
    const dotsHtml = COLORS.map((c, i) => `
      <div class="color-dot ${i === note.colorIdx ? 'active' : ''}"
           style="background:${c.dot}"
           onclick="updateNoteColor(${note.id}, ${i})"
           title="${c.id}"></div>
    `).join('');

    card.innerHTML = `
      <div class="note-pin">${note.pin}</div>
      <div class="note-lines"></div>
      <textarea class="note-textarea"
                placeholder="${note.placeholder}"
                oninput="updateNoteText(${note.id}, this.value)"
      >${escHtml(note.text)}</textarea>
      <div class="note-footer">
        <div>
          <div class="color-dots">${dotsHtml}</div>
          <div class="note-date">${note.created}</div>
        </div>
        <button class="delete-btn" onclick="deleteNote(${note.id})" title="Delete note">✕</button>
      </div>
    `;

    if (animate) {
      card.style.animationName = 'noteIn';
      card.style.animationDuration = '0.35s';
    } else {
      card.style.animation = 'none';
    }

    grid.appendChild(card);
  }

  function renderAll() {
    const grid = document.getElementById('notesGrid');
    grid.innerHTML = '';
    notes.forEach(n => renderNote(n, false));
    updateCount();
    checkEmpty();
  }

  function updateCount() {
    const c = notes.length;
    document.getElementById('noteCount').textContent = c === 0 ? 'Empty board' : `${c} note${c !== 1 ? 's' : ''}`;
  }

  function checkEmpty() {
    const es = document.getElementById('emptyState');
    if (notes.length === 0) {
      es.classList.add('visible');
    } else {
      es.classList.remove('visible');
    }
  }

  function hideEmpty() {
    document.getElementById('emptyState').classList.remove('visible');
  }

  function save() {
    localStorage.setItem('pinboard-notes', JSON.stringify(notes));
    localStorage.setItem('pinboard-nextid', nextId);
  }

  function escHtml(str) {
    return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }
</script>
</body>
</html>
# sticky-notes-project
mainly use of taggle in the project
