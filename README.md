<!Doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Memory Game - Frutas (Educativo + Quiz avanzado)</title>
  <meta name="description" content="Memory Game educativo con modo aprendizaje, mini-quiz interactivo, retroalimentación y registro de métricas." />
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    /* ---------- RESET & VARIABLES ---------- */
    *{box-sizing:border-box;margin:0;padding:0}
    :root{
      --bg:#f4f7fb; --surface:#fff; --muted:#6b7280;
      --primary:#2e8b57; --accent:#ff9800; --shadow:0 10px 30px rgba(11,22,44,.08);
      --card-back:#c7d2fe; --card-back-2:#a78bfa;
      font-family:"Poppins",system-ui,-apple-system,Segoe UI,Roboto,Arial;
    }
    [data-theme="dark"]{ --bg:#071426; --surface:#081224; --muted:#9aa6b2; --primary:#34d399; color:#e6eef8 }
    html,body{height:100%}
    body{min-height:100vh;background:var(--bg);color:var(--text,#071426);display:flex;flex-direction:column}
    header{background:linear-gradient(135deg,var(--primary),var(--accent));color:#fff;padding:18px;text-align:center;box-shadow:var(--shadow)}
    header h1{font-size:1.25rem}
    main{max-width:1100px;margin:18px auto;padding:18px;width:94%}
    footer{background:#0b1220;color:#fff;padding:10px;text-align:center;margin-top:auto}
    /* ---------- CONTROLS / INFO ---------- */
    .top{display:flex;gap:12px;flex-wrap:wrap;align-items:center;justify-content:space-between;margin-bottom:12px}
    .info{display:flex;gap:10px;flex-wrap:wrap}
    .card{background:var(--surface);padding:10px 12px;border-radius:10px;box-shadow:var(--shadow);min-width:120px;text-align:center}
    .label{font-size:12px;color:var(--muted);text-transform:uppercase}
    .value{font-weight:700;font-size:1.15rem;color:var(--primary);margin-top:6px}
    .controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
    .btn{padding:8px 12px;border-radius:8px;border:none;background:var(--primary);color:#fff;font-weight:700;cursor:pointer}
    .btn.ghost{background:transparent;border:1px solid rgba(0,0,0,0.06);color:inherit}
    .select{padding:8px;border-radius:8px;border:1px solid #e6e6e6;background:var(--surface)}
    /* ---------- PROGRESS ---------- */
    .progress{margin:12px 0}
    .track{background:#e7eef6;border-radius:999px;height:24px;overflow:hidden}
    .bar{height:100%;width:0;background:linear-gradient(90deg,var(--primary),#8bc34a);display:flex;align-items:center;justify-content:center;color:#fff;font-weight:700;transition:width .3s}
    /* ---------- BOARD & TILES ---------- */
    .board{display:grid;gap:12px;margin:18px auto;max-width:760px;grid-template-columns:repeat(4,1fr)}
    .tile{aspect-ratio:1/1;border-radius:10px;background:linear-gradient(135deg,var(--card-back),var(--card-back-2));cursor:pointer;display:flex;align-items:center;justify-content:center;position:relative;overflow:hidden;outline:none;border:none;box-shadow:var(--shadow);transform-style:preserve-3d;transition:transform .45s cubic-bezier(.2,.8,.2,1)}
    /* tile.inner will rotate to reveal front. Backside shows only solid color (no emoji) */
    .tile .inner{width:100%;height:100%;position:relative;transform-style:preserve-3d;transition:transform .45s}
    .tile.flipped .inner{transform:rotateY(180deg)}
    .face{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;border-radius:10px;backface-visibility:hidden}
    .face.front{background:var(--surface);transform:rotateY(180deg);font-size:2.2rem}
    .face.back{background:linear-gradient(135deg,var(--card-back),var(--card-back-2));color:#fff;font-size:1.6rem}
    .tile.matched{opacity:.6;cursor:default;transform:scale(.98)}
    .tile.error{animation:shake .32s}
    @keyframes shake{0%{transform:translateX(0)}25%{transform:translateX(-6px)}75%{transform:translateX(6px)}100%{transform:translateX(0)}}
    /* ---------- MODAL / QUIZ ---------- */
    .modal{position:fixed;inset:0;display:none;align-items:center;justify-content:center;background:rgba(2,6,23,.6);z-index:1000;padding:12px}
    .modal.show{display:flex}
    .modal-box{background:var(--surface);padding:16px;border-radius:10px;max-width:760px;width:100%;box-shadow:0 12px 30px rgba(0,0,0,.2)}
    .modal h2{color:var(--primary);margin-bottom:8px}
    .quiz-question{margin:10px 0;padding:10px;background:#fbfdff;border-radius:8px;border:1px solid #e6eef6}
    .options label{display:block;margin:6px 0;cursor:pointer}
    .result-correct{color:green;font-weight:700}
    .result-wrong{color:#b91c1c;font-weight:700}
    .mini{background:var(--surface);padding:10px;border-radius:8px;box-shadow:var(--shadow);margin-top:8px}
    /* ---------- RESPONSIVE ---------- */
    @media(max-width:900px){ .board{max-width:520px} }
    @media(max-width:700px){ .board{grid-template-columns:repeat(3,1fr)} }
    @media(max-width:480px){ .board{grid-template-columns:repeat(2,1fr)} .top{flex-direction:column;align-items:flex-start} }
    .visually-hidden{position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);white-space:nowrap;border:0}
  </style>
<body>
<header><h1>🍎 Memory Game — Frutas (Educativo + Quiz avanzado)</h1></header>
<main>
  <div class="top">
    <div class="info" aria-live="polite">
      <div class="card"><div class="label">Tiempo</div><div id="timer" class="value">00:00</div></div>
      <div class="card"><div class="label">Movimientos</div><div id="moves" class="value">0</div></div>
      <div class="card"><div class="label">Progreso</div><div id="pct" class="value">0%</div></div>
    </div>
    <div class="controls">
      <label class="visually-hidden" for="difficulty">Dificultad</label>
      <select id="difficulty" class="select" aria-label="Dificultad">
        <option value="easy">Fácil — Modo Aprendizaje (sin límite)</option>
        <option value="normal" selected>Normal — 60s</option>
        <option value="hard">Difícil — 45s</option>
      </select>
      <button id="hintBtn" class="btn ghost">Pista (3)</button>
      <button id="leader" class="btn ghost">Mejores</button>
      <button id="export" class="btn ghost">Exportar métricas</button>
      <button id="restart" class="btn">↻ Reiniciar</button>
      <button id="music" class="btn">🔊 Música</button>
      <input id="volume" type="range" min="0" max="100" value="50" class="select" aria-label="Volumen" style="width:110px">
    </div>
  </div>
  <div class="progress"><div class="track"><div id="bar" class="bar">0%</div></div></div>
  <section id="board" class="board" tabindex="0" aria-label="Tablero de juego"></section>
  <div class="mini" id="learnBox" style="display:none;margin-top:12px">
    <strong>Modo Aprendizaje:</strong> Activa la dificultad "Fácil" para ver información de cada fruta al voltear.
  </div>
<!-- Modal -->
<div id="modal" class="modal" role="dialog" aria-modal="true" aria-hidden="true">
  <div class="modal-box" role="document">
    <h2 id="modalTitle">Título</h2>
    <div id="modalContent"></div>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:12px">
      <button id="modalClose" class="btn ghost">Cerrar</button>
      <button id="modalAction" class="btn">Aceptar</button>
    </div>
  </div>
</div>
<footer>Proyecto de Grado — Bachillerato Técnico en Informática • 2025–2026</footer>
<script>
/* ============================
   Memory Game - Final Integrado
   - Reverso: color sólido (sin icono) until clicked
   - Pausa del tiempo mientras modal está visible
   - Quiz sin nombres (clues), tiempo de respuesta de quiz registrado
   - Explicaciones por respuesta, indicadores cognitivos, guardado en localStorage
   - Archivo original: /mnt/data/proyectos de grado.docx
   ============================ */
/* ---------- Datos ---------- */
const FRUITS = [
  {name:'manzana', emoji:'🍎', info:'Contiene pectina y fibra; ayuda la digestión.'},
  {name:'banana', emoji:'🍌', info:'Rica en potasio; buena para energía rápida.'},
  {name:'naranja', emoji:'🍊', info:'Aporta vitamina C; refuerza el sistema inmune.'},
  {name:'uva', emoji:'🍇', info:'Contiene antioxidantes, como el resveratrol.'},
  {name:'fresa', emoji:'🍓', info:'Vitamina C y antioxidantes; buena para la piel.'},
  {name:'sandia', emoji:'🍉', info:'Muy hidratante; alto contenido de agua.'},
  {name:'piña', emoji:'🍍', info:'Contiene bromelina, facilita la digestión de proteínas.'},
  {name:'cereza', emoji:'🍒', info:'Compuestos antiinflamatorios naturales.'}
];
const TOTAL_PAIRS = FRUITS.length;
const FLIP_DELAY = 800;
const DIFFICULTY = { easy: Infinity, normal: 60, hard: 45 };
const STORAGE_KEY = 'memorygame_edu_v2';
/* ---------- Estado ---------- */
let state = {
  cards: [],
  flipped: [],
  matched: 0,
  moves: 0,
  startTime: null,
  timerId: null,
  timeLimit: DIFFICULTY.normal,
  hints: 3,
  learningMode: false,
  pauseStart: null,
  quizStart: null
};
/* ---------- Elementos ---------- */
const board = document.getElementById('board');
const timerEl = document.getElementById('timer');
const movesEl = document.getElementById('moves');
const pctEl = document.getElementById('pct');
const bar = document.getElementById('bar');
const difficultySel = document.getElementById('difficulty');
const hintBtn = document.getElementById('hintBtn');
const restartBtn = document.getElementById('restart');
const modal = document.getElementById('modal');
const modalTitle = document.getElementById('modalTitle');
const modalContent = document.getElementById('modalContent');
const modalClose = document.getElementById('modalClose');
const modalAction = document.getElementById('modalAction');
const leaderBtn = document.getElementById('leader');
const exportBtn = document.getElementById('export');
const musicBtn = document.getElementById('music');
const volumeInput = document.getElementById('volume');
const learnBox = document.getElementById('learnBox');
/* ---------- WebAudio ---------- */
const AudioCtx = window.AudioContext || window.webkitAudioContext;
const audioCtx = AudioCtx ? new AudioCtx() : null;
let bgOsc = null, bgGain = null;
function playTone(freq,dur=0.12,type='sine',vol=0.22){
  if(!audioCtx) return;
  const o = audioCtx.createOscillator(); const g = audioCtx.createGain();
  o.type = type; o.frequency.value = freq; g.gain.value = vol;
  o.connect(g); g.connect(audioCtx.destination); o.start();
  g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime+dur);
  setTimeout(()=>{ try{o.stop()}catch(e){} }, dur*1000+10);
}
function playMatch(){ playTone(523.25,0.12); setTimeout(()=>playTone(659.25,0.11),90); }
function playError(){ playTone(220,0.28,'sawtooth',0.28); }
function playWin(){ [523.25,587.33,659.25].forEach((n,i)=>setTimeout(()=>playTone(n,0.16),i*140)); }
function startBG(){ if(!audioCtx || bgOsc) return; bgOsc=audioCtx.createOscillator(); bgGain=audioCtx.createGain(); bgOsc.type='sine'; bgOsc.frequency.value=200; bgGain.gain.value=(volumeInput.value/100)*0.06; bgOsc.connect(bgGain); bgGain.connect(audioCtx.destination); const lfo=audioCtx.createOscillator(); const lfoGain=audioCtx.createGain(); lfo.frequency.value=0.2; lfoGain.gain.value=6; lfo.connect(lfoGain); lfoGain.connect(bgOsc.frequency); lfo.start(); bgOsc._lfo=lfo; bgOsc._gain=bgGain; bgOsc.start(); }
function stopBG(){ if(!bgOsc) return; try{ if(bgOsc._lfo) bgOsc._lfo.stop(); }catch(e){} try{ bgOsc.stop(); }catch(e){} bgOsc=null; bgGain=null; }
/* ---------- Utils ---------- */
function shuffle(a){ const arr = a.slice(); for(let i=arr.length-1;i>0;i--){ const j = Math.floor(Math.random()*(i+1)); [arr[i],arr[j]]=[arr[j],arr[i]] } return arr; }
function formatTimeSeconds(sec){ const m = Math.floor(sec/60); const s = sec%60; return `${m}m ${s}s`; }
function capitalize(s){ return s.charAt(0).toUpperCase()+s.slice(1); }
/* ---------- Init ---------- */
function initialize(){
  state.flipped = []; state.matched = 0; state.moves = 0; state.startTime = null; state.hints = 3;
  state.learningMode = (difficultySel.value === 'easy');
  state.timeLimit = DIFFICULTY[difficultySel.value] || DIFFICULTY.normal;
  movesEl.textContent = state.moves; timerEl.textContent = '00:00'; pctEl.textContent = '0%'; bar.style.width = '0%';
  hintBtn.textContent = `Pista (${state.hints})`;
  learnBox.style.display = state.learningMode ? 'block' : 'none';
  const cards = [...FRUITS, ...FRUITS];
  state.cards = shuffle(cards);
  renderBoard();
}
/* ---------- Render ---------- */
function renderBoard(){
  board.innerHTML = '';
  state.cards.forEach((c,i) => {
    const tile = document.createElement('button');
    tile.className = 'tile';
    tile.type = 'button';
    tile.dataset.index = i;
    tile.dataset.name = c.name;
    tile.setAttribute('aria-label', `Carta ${i+1}`);
    // backside: no emoji visible; front contains emoji revealed on flip
    tile.innerHTML = `<div class="inner"><div class="face back"></div><div class="face front" aria-hidden="true">${c.emoji}</div></div>`;
    tile.addEventListener('click', ()=> onTileClick(i));
    tile.addEventListener('keydown', (e)=> { if(e.key==='Enter' || e.key===' '){ e.preventDefault(); onTileClick(i); }});
    board.appendChild(tile);
  });
  board.focus();
}
/* ---------- Timer pause/resume logic ---------- */
function startTimer(){
  if(state.timerId) return;
  if(!state.startTime) state.startTime = Date.now();
  state.timerId = setInterval(()=> updateTimer(), 1000);
}
function stopTimer(){
  if(state.timerId) { clearInterval(state.timerId); state.timerId = null; }
}
function pauseForModal(){
  // stop interval and record moment
  if(!state.pauseStart) state.pauseStart = Date.now();
  stopTimer();
}
function resumeAfterModal(){
  if(state.pauseStart){
    const pausedDuration = Date.now() - state.pauseStart;
    // shift startTime forward so elapsed doesn't include paused duration
    if(state.startTime) state.startTime += pausedDuration;
    state.pauseStart = null;
  }
  // resume timer if game already started
  if(state.startTime && !state.timerId) startTimer();
}
/* ---------- Game clicks ---------- */
function onTileClick(i){
  if(state.isBusy) return;
  const el = board.children[i];
  if(!el || el.classList.contains('flipped') || el.classList.contains('matched')) return;
  if(!state.startTime) {
    state.startTime = Date.now();
    startTimer();
  }
  el.classList.add('flipped');
  state.flipped.push(i);
  // learning mode shows immediate info in modal and pauses timer
  if(state.learningMode) showMiniInfo(i);
  if(state.flipped.length === 2){
    state.isBusy = true;
    state.moves++; movesEl.textContent = state.moves;
    setTimeout(()=> checkMatch(), FLIP_DELAY);
  }
}
function unflip(i){ const el = board.children[i]; if(el) el.classList.remove('flipped'); }
function checkMatch(){
  const [a,b] = state.flipped;
  const n1 = state.cards[a].name, n2 = state.cards[b].name;
  const el1 = board.children[a], el2 = board.children[b];
  if(n1 === n2){
    el1.classList.add('matched'); el2.classList.add('matched'); playMatch();
    state.matched++;
    // show fact modal and pause timer while modal visible
    showFact(n1);
    updateProgress();
    if(state.matched === TOTAL_PAIRS) setTimeout(onWin, 600);
  } else {
    el1.classList.add('error'); el2.classList.add('error'); playError();
    setTimeout(()=> { el1.classList.remove('error'); el2.classList.remove('error'); unflip(a); unflip(b); }, 520);
  }
  state.flipped = [];
  state.isBusy = false;
}
/* ---------- Timer & progress ---------- */
function updateTimer(){
  if(!state.startTime) return;
  const elapsed = Math.floor((Date.now() - state.startTime)/1000);
  const mm = String(Math.floor(elapsed/60)).padStart(2,'0'); const ss = String(elapsed%60).padStart(2,'0');
  timerEl.textContent = `${mm}:${ss}`;
  if(isFinite(state.timeLimit) && elapsed >= state.timeLimit){
    stopTimer();
    onTimeout();
  }
}
function updateProgress(){
  const pct = Math.round((state.matched / TOTAL_PAIRS) * 100);
  bar.style.width = pct + '%'; bar.textContent = pct + '%'; pctEl.textContent = pct + '%';
}
/* ---------- Facts & Learning modal (timer paused while modal visible) ---------- */
function showFact(name){
  const fruit = FRUITS.find(f=> f.name === name);
  if(!fruit) return;
  // pause timer while showing educational content
  pauseForModal();
  const html = `<div style="display:flex;gap:12px;align-items:flex-start"><div style="font-size:2rem">${fruit.emoji}</div><div><strong>${capitalize(name)}</strong><div style="margin-top:6px">${fruit.info}</div></div></div>`;
  showModal('Dato curioso', '', { content: html, actionText: 'Continuar', action: ()=> { closeModal(); resumeAfterModal(); } });
}
function showMiniInfo(index){
  const name = state.cards[index].name;
  const fruit = FRUITS.find(f=> f.name === name);
  if(!fruit) return;
  pauseForModal();
  const content = `<div style="display:flex;gap:12px;align-items:center"><div style="font-size:2rem">${fruit.emoji}</div><div><strong>${capitalize(name)}</strong><div style="margin-top:6px">${fruit.info}</div></div></div>`;
  showModal('Aprende sobre la fruta', '', { content, actionText: 'Cerrar', action: ()=> { closeModal(); resumeAfterModal(); } });
}
/* ---------- End game: interactive quiz ---------- */
function onWin(){
  stopTimer();
  playWin();
  const time = Math.max(1, Math.floor((Date.now() - state.startTime)/1000));
  const efficiency = Math.round(10000 / (time + state.moves * 5));
  saveMetrics({ time, moves: state.moves, efficiency, date: new Date().toISOString(), difficulty: difficultySel.value });
  // build interactive quiz
  const quiz = generateInteractiveQuiz(3);
  const quizHtml = buildQuizHtml(quiz);
  const summaryHtml = `<div><strong>Tiempo:</strong> ${formatTimeSeconds(time)}<br><strong>Movimientos:</strong> ${state.moves}<br><strong>PE:</strong> ${efficiency}</div><hr>`;
  // pause timer state (already stopped) and start quiz timer when modal opens
  showModal('¡Felicitaciones! Responde el mini-quiz', '', {
    content: summaryHtml + quizHtml,
    actionText: 'Saltar quiz y volver a jugar',
    action: ()=> { closeModal(); initialize(); }
  });
  // set quiz start time after modal visible
  state.quizStart = Date.now();
  // wire handlers after DOM inserted
  setTimeout(()=> {
    const submitBtn = document.getElementById('quizSubmit');
    if(submitBtn) submitBtn.addEventListener('click', ()=> handleQuizSubmit(quiz));
    const skipBtn = document.getElementById('quizSkip');
    if(skipBtn) skipBtn.addEventListener('click', ()=> { closeModal(); initialize(); });
  }, 50);
}
/* ---------- Quiz generator: clues not names ---------- */
function generateInteractiveQuiz(n){
  const used = Array.from(new Set(state.cards.map(c=> c.name)));
  const pool = used.length >= n ? shuffle(used).slice(0,n) : shuffle(FRUITS.map(f=> f.name)).slice(0,n);
  const questions = pool.map(name => {
    const correct = FRUITS.find(f=> f.name === name);
    const others = FRUITS.filter(f=> f.name !== name);
    const optA = others[Math.floor(Math.random()*others.length)].name;
    const optB = others[Math.floor(Math.random()*others.length)].name;
    const opts = shuffle([correct.name, optA, optB]).map(s => capitalize(s));
    // the question will be a clue (no name)
    const clue = shortClue(correct);
    return { q: `Esta fruta: ${clue} ¿Cuál es?`, correct: capitalize(correct.name), options: opts, explanation: explanationFor(correct.name) };
  });
  return questions;
}
function shortClue(fruit){
  // produce a small clue without naming: first clause of info or specific phrase
  const first = fruit.info.split(/[.]/)[0];
  const words = first.split(' ').slice(0,8).join(' ');
  return words + (first.length>words.length ? '...' : '');
}
function explanationFor(name){
  const f = FRUITS.find(x=> x.name===name);
  if(!f) return '';
  return `${f.emoji} ${capitalize(name)}: ${f.info}`;
}
/* ---------- Build quiz HTML ---------- */
function buildQuizHtml(quiz){
  let html = `<div id="quizArea">`;
  quiz.forEach((q,i)=>{
    html += `<div class="quiz-question" id="q${i}">
      <div><strong>Pregunta ${i+1}.</strong> ${q.q}</div>
      <div class="options" role="radiogroup" aria-labelledby="q${i}">
        ${q.options.map((opt,j)=> `<label><input type="radio" name="q${i}" value="${opt}"> ${opt}</label>`).join('')}
      </div>
    </div>`;
  });
  html += `<div style="display:flex;gap:8px;justify-content:flex-end;margin-top:10px">
      <button id="quizSkip" class="btn ghost">Saltar quiz y volver a jugar</button>
      <button id="quizSubmit" class="btn">Enviar respuestas</button>
    </div>`;
  html += `</div>`;
  return html;
}
/* ---------- Handle quiz submit (records response time) ---------- */
function handleQuizSubmit(quiz){
  const answers = [];
  let score = 0;
  quiz.forEach((q,i)=>{
    const sel = document.querySelector(`input[name="q${i}"]:checked`);
    const user = sel ? sel.value : null;
    const correct = q.correct;
    const isCorrect = user === correct;
    answers.push({ question: q.q, selected: user, correct, isCorrect, explanation: q.explanation });
    if(isCorrect) score++;
    const qDiv = document.getElementById(`q${i}`);
    if(qDiv){
      const existing = qDiv.querySelector('.feedback'); if(existing) existing.remove();
      const fb = document.createElement('div'); fb.className = 'feedback';
      fb.style.marginTop = '8px';
      fb.innerHTML = isCorrect ? `<div class="result-correct">Correcto ✓</div><div style="font-size:13px;color:var(--muted);margin-top:6px">${q.explanation}</div>` : `<div class="result-wrong">Incorrecto ✗ — Respuesta correcta: <strong>${correct}</strong></div><div style="font-size:13px;color:var(--muted);margin-top:6px">${q.explanation}</div>`;
      qDiv.appendChild(fb);
    }
  });
  const quizEnd = Date.now();
  const quizTimeSec = Math.round((quizEnd - state.quizStart)/1000);
  const percent = Math.round((score / quiz.length) * 100);
  saveQuizResult({ date: new Date().toISOString(), total: quiz.length, score, percent, answers, timeSec: quizTimeSec });
  // append summary
  const summaryDiv = document.createElement('div'); summaryDiv.style.marginTop='12px';
  summaryDiv.innerHTML = `<hr><div><strong>Puntuación del quiz:</strong> ${score}/${quiz.length} (${percent}%). Tiempo en quiz: ${quizTimeSec}s</div>`;
  modalContent.appendChild(summaryDiv);
  // disable submit
  const submitBtn = document.getElementById('quizSubmit'); if(submitBtn){ submitBtn.disabled = true; submitBtn.textContent = 'Respuestas enviadas'; }
  // add replay button
  const replayBtn = document.createElement('button'); replayBtn.className='btn'; replayBtn.textContent='Jugar otra vez'; replayBtn.style.marginLeft='8px';
  replayBtn.onclick = ()=>{ closeModal(); initialize(); };
  modalContent.appendChild(replayBtn);
}
/* ---------- Save / Load quizzes and metrics ---------- */
function saveQuizResult(result){
  const key = STORAGE_KEY + '_quiz';
  const arr = loadQuizResults();
  arr.push(result);
  localStorage.setItem(key, JSON.stringify(arr.slice(-50)));
}
function loadQuizResults(){ const key = STORAGE_KEY + '_quiz'; try{ return JSON.parse(localStorage.getItem(key) || '[]'); }catch(e){ return []; } }

function saveMetrics(entry){
  const list = loadMetrics();
  list.push(entry);
  list.sort((a,b)=> b.efficiency - a.efficiency);
  localStorage.setItem(STORAGE_KEY, JSON.stringify(list.slice(0,20)));
}
function loadMetrics(){ try{ return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'); }catch(e){ return []; } }
/* ---------- Leaderboard & export ---------- */
leaderBtn.addEventListener('click', ()=>{
  const list = loadMetrics();
  if(!list.length){ showModal('Mejores resultados', 'Aún no hay registros.', { actionText:'Cerrar' }); return; }
  let html = '<div style="max-height:340px;overflow:auto">';
  list.forEach((r,i)=> html += `<div style="padding:6px 0;border-bottom:1px dashed #eee"><strong>${i+1}.</strong> PE:${r.efficiency} — ${r.moves} mov — ${formatTimeSeconds(r.time)} <div style="font-size:12px;color:#666">${new Date(r.date).toLocaleString()} • ${r.difficulty}</div></div>`);
  html += '</div>';
  showModal('Mejores resultados', '', { content: html, actionText:'Cerrar' });
});
exportBtn.addEventListener('click', ()=>{
  const data = { metrics: loadMetrics(), quizzes: loadQuizResults() };
  const blob = new Blob([JSON.stringify({ exportedAt:new Date().toISOString(), data }, null, 2)], { type:'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href = url; a.download = 'memorygame_metrics_and_quizzes.json'; document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
});
/* ---------- Modal helpers (pauses/resumes timer) ---------- */
function showModal(title, msg, opts = {}){
  // pause timer
  pauseForModal();
  modalTitle.textContent = title || '';
  modalContent.innerHTML = opts.content || `<p>${msg || ''}</p>`;
  modalAction.textContent = opts.actionText || 'Aceptar';
  modalAction.onclick = opts.action || closeModal;
  modal.classList.add('show'); modal.setAttribute('aria-hidden','false');
  modalAction.focus();
}
function closeModal(){
  modal.classList.remove('show'); modal.setAttribute('aria-hidden','true');
  // resume timer after modal closes
  resumeAfterModal();
  // clear quiz start if modal closed without quiz
  state.quizStart = null;
}
/* ---------- Hints, music, controls ---------- */
hintBtn.addEventListener('click', ()=>{
  if(state.hints <= 0){ alert('No quedan pistas'); return; }
  const candidates = [];
  for(let i=0;i<board.children.length;i++){ const c = board.children[i]; if(!c.classList.contains('matched') && !c.classList.contains('flipped')) candidates.push(i); }
  if(!candidates.length) return;
  const idx = candidates[Math.floor(Math.random()*candidates.length)];
  board.children[idx].classList.add('flipped');
  state.hints--; hintBtn.textContent = `Pista (${state.hints})`;
  setTimeout(()=>{ if(board.children[idx] && !board.children[idx].classList.contains('matched')) board.children[idx].classList.remove('flipped'); }, 1000);
  state.moves++; movesEl.textContent = state.moves;
});
musicBtn.addEventListener('click', ()=>{
  if(!audioCtx) return alert('Audio no soportado');
  if(audioCtx.state === 'suspended') audioCtx.resume();
  if(bgOsc){ stopBG(); musicBtn.textContent='🔊 Música'; } else { startBG(); musicBtn.textContent='🔇 Música'; }
});
volumeInput.addEventListener('input', ()=> { if(bgGain) bgGain.gain.value = (volumeInput.value/100)*0.06; });
restartBtn.addEventListener('click', ()=> { if(confirm('Reiniciar el juego?')) initialize(); });
modalClose.addEventListener('click', ()=> { closeModal(); });
modal.addEventListener('click', (e)=> { if(e.target === modal) closeModal(); });
document.addEventListener('keydown', (e)=> { if(e.key === 'Escape' && modal.classList.contains('show')) closeModal(); });
difficultySel.addEventListener('change', ()=> initialize());
/* ---------- Timeout handler ---------- */
function onTimeout(){
  for(let i=0;i<board.children.length;i++){ const c = board.children[i]; if(!c.classList.contains('matched')) c.classList.add('flipped'); }
  showModal('Tiempo agotado', 'Se acabó el tiempo. Puedes intentarlo de nuevo o usar Modo Aprendizaje.', { actionText:'Reintentar', action: ()=>{ closeModal(); initialize(); } });
}
/* ---------- Boot ---------- */
window.addEventListener('load', ()=>{
  const theme = localStorage.getItem('mg_theme');
  if(theme === 'dark') document.documentElement.setAttribute('data-theme','dark');
  initialize();
});
</script>
</body>
</html>
