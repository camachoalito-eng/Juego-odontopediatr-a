<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
<title>Plaque Run: Sonrisa en Movimiento</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@500;600;700&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root{
    --enamel:#EAF6F6;
    --enamel-deep:#CFEFEA;
    --dentin:#0E3B43;
    --dentin-soft:#155059;
    --plaque:#C7D94B;
    --plaque-dark:#9CB22E;
    --reveal:#E8487C;
    --reveal-soft:#FBD3E2;
    --mint:#2FBF9F;
    --gold:#F4B740;
    --danger:#D94F30;
    --gum:#F6C9D6;
    --gum-deep:#E89BB4;
    --font-display:'Fredoka', sans-serif;
    --font-body:'Inter', sans-serif;
  }

  *{ box-sizing:border-box; -webkit-tap-highlight-color:transparent; }
  html,body{ height:100%; margin:0; overscroll-behavior:none; }
  body{
    background:var(--dentin);
    font-family:var(--font-body);
    display:flex;
    align-items:center;
    justify-content:center;
    min-height:100svh;
    color:var(--dentin);
    user-select:none;
  }

  #app{ width:100%; height:100svh; display:flex; align-items:center; justify-content:center; }

  #game{
    position:relative;
    width:min(480px, 100vw);
    height:min(900px, 100svh);
    overflow:hidden;
    background:linear-gradient(180deg, var(--enamel) 0%, var(--enamel-deep) 78%, var(--gum) 100%);
    box-shadow:0 0 60px rgba(0,0,0,0.45);
    touch-action:none;
  }
  @media (min-width:481px){
    #game{ border-radius:28px; border:6px solid var(--dentin); }
  }

  /* ---------- Backdrop ---------- */
  #backdrop{ position:absolute; inset:0; pointer-events:none; }
  .lane-line{
    position:absolute; top:0; bottom:10%; width:2px;
    background:repeating-linear-gradient(180deg, rgba(14,59,67,0.16) 0 18px, transparent 18px 36px);
  }
  .lane-line.l1{ left:33.333%; }
  .lane-line.l2{ left:66.666%; }
  #gumline{
    position:absolute; left:0; right:0; bottom:0; height:10%;
    background:linear-gradient(180deg, var(--gum) 0%, var(--gum-deep) 100%);
    box-shadow:0 -2px 0 rgba(14,59,67,0.12) inset;
  }
  #gumline::before{
    content:""; position:absolute; top:-6px; left:0; right:0; height:6px;
    background:repeating-linear-gradient(90deg, var(--enamel) 0 14px, transparent 14px 28px);
    opacity:.55;
  }

  /* ---------- HUD ---------- */
  #hud{
    position:absolute; top:0; left:0; right:0; z-index:30;
    display:flex; align-items:center; justify-content:space-between;
    padding:14px 16px; pointer-events:none;
  }
  #lives{ display:flex; gap:4px; font-size:22px; filter:drop-shadow(0 2px 2px rgba(0,0,0,.15)); }
  #lives .tooth{ transition:opacity .2s, transform .2s; }
  #lives .tooth.lost{ opacity:.25; transform:scale(.85) rotate(-8deg); }
  #scoreWrap{
    font-family:var(--font-display); font-weight:600; color:var(--dentin-soft);
    background:rgba(255,255,255,.55); padding:6px 14px; border-radius:999px;
    font-size:18px; letter-spacing:.5px;
  }

  #sugarBanner{
    position:absolute; top:54px; left:50%; transform:translateX(-50%) translateY(-10px);
    background:var(--danger); color:#fff; font-family:var(--font-display); font-weight:600;
    font-size:13px; padding:5px 12px; border-radius:999px; opacity:0; pointer-events:none;
    transition:opacity .25s, transform .25s; z-index:30; white-space:nowrap;
  }
  #sugarBanner.show{ opacity:1; transform:translateX(-50%) translateY(0); }
  #game.shake { animation: shakeAnim .3s; }
  @keyframes shakeAnim{
    0%,100%{ transform:translateX(0); }
    20%{ transform:translateX(-8px); }
    40%{ transform:translateX(7px); }
    60%{ transform:translateX(-5px); }
    80%{ transform:translateX(4px); }
  }

  #game.tincion-flash::after{
    content:""; position:absolute; inset:0; background:radial-gradient(circle at 50% 30%, var(--reveal-soft), transparent 70%);
    opacity:.85; animation:flashFade .5s ease-out forwards; pointer-events:none; z-index:25;
  }
  @keyframes flashFade{ from{opacity:.85;} to{opacity:0;} }

  /* ---------- Entities ---------- */
  /* NOTE: el contenedor .entity SOLO se usa para posicionar (lo mueve el JS con
     transform: translateY). Toda animación decorativa (wobble, pulso, pop, floaty)
     vive en el hijo .entity-visual para que ambos `transform` no se peleen entre sí. */
  #entities{ position:absolute; inset:0; z-index:10; pointer-events:none; }
  .entity{
    position:absolute; top:0; left:0;
    will-change:transform;
  }
  .entity-visual{
    width:100%; height:100%;
    display:flex; align-items:center; justify-content:center;
    font-size:30px; line-height:1;
    filter:drop-shadow(0 4px 4px rgba(0,0,0,.18));
  }
  .entity-visual.powerup{
    font-size:26px;
    animation:floaty 1.4s ease-in-out infinite;
  }
  @keyframes floaty{ 0%,100%{ margin-top:0;} 50%{ margin-top:-6px;} }

  .entity-visual.pop{ animation:popOut .22s ease-out forwards; }
  @keyframes popOut{ to{ transform:scale(1.6); opacity:0; } }

  .entity-visual.type-biofilm{
    border-radius:46% 54% 58% 42% / 50% 45% 55% 50%;
    background:radial-gradient(circle at 35% 30%, #DCE96B, var(--plaque) 60%, var(--plaque-dark) 100%);
    box-shadow:0 0 0 2px rgba(156,178,46,.35) inset;
    animation:blobPulse 1.6s ease-in-out infinite;
  }
  .entity-visual.type-biofilm.revealed{
    background:radial-gradient(circle at 35% 30%, #FF9CC0, var(--reveal) 60%, #B82E5C 100%);
    box-shadow:0 0 0 3px rgba(232,72,124,.45) inset, 0 0 14px rgba(232,72,124,.6);
    animation:revealPulse .5s ease-in-out infinite;
  }
  @keyframes blobPulse{ 0%,100%{ transform:scale(1);} 50%{ transform:scale(1.08);} }
  @keyframes revealPulse{ 0%,100%{ transform:scale(1);} 50%{ transform:scale(1.16);} }

  .entity-visual.type-sarro{
    filter:sepia(.6) saturate(1.4) brightness(.85) drop-shadow(0 4px 4px rgba(0,0,0,.25));
    animation:wobble 2s ease-in-out infinite;
  }
  @keyframes wobble{ 0%,100%{ transform:rotate(-4deg);} 50%{ transform:rotate(4deg);} }

  .entity-visual.type-tincion{ filter:drop-shadow(0 0 8px rgba(232,72,124,.65)); }
  .entity-visual.type-cepillo{ filter:drop-shadow(0 0 8px rgba(47,191,159,.65)); }
  .entity-visual.type-hilo{ filter:drop-shadow(0 0 8px rgba(47,191,159,.65)); }
  .entity-visual.type-dentista{ filter:drop-shadow(0 0 9px rgba(244,183,64,.75)); }

  /* ---------- Player ---------- */
  #player{
    position:absolute; top:76%; width:60px; height:60px; margin-left:-30px;
    display:flex; align-items:center; justify-content:center; font-size:38px;
    transition:left .14s ease-out;
    z-index:20; transform:rotate(0deg); pointer-events:none;
    animation:run .5s ease-in-out infinite;
    filter:drop-shadow(0 6px 6px rgba(0,0,0,.25));
  }
  @keyframes run{ 0%,100%{ transform:translateY(0) rotate(-4deg);} 50%{ transform:translateY(-6px) rotate(4deg);} }
  #player.shield::before{
    content:""; position:absolute; inset:-10px; border-radius:50%;
    border:3px solid var(--gold); box-shadow:0 0 16px rgba(244,183,64,.8);
    animation:shieldSpin 1.2s linear infinite;
  }
  @keyframes shieldSpin{ from{ transform:rotate(0);} to{ transform:rotate(360deg);} }

  /* ---------- Toast ---------- */
  #toast{
    position:absolute; left:50%; bottom:13%; transform:translate(-50%, 14px);
    max-width:88%; background:var(--dentin); color:#fff; font-size:13.5px; line-height:1.35;
    padding:10px 16px; border-radius:14px; opacity:0; pointer-events:none;
    transition:opacity .3s, transform .3s; z-index:35; text-align:center;
    font-weight:500;
  }
  #toast.show{ opacity:1; transform:translate(-50%, 0); }
  #toast.combo{ background:var(--gold); color:var(--dentin); font-family:var(--font-display); font-weight:600; font-size:16px; }
  #toast.good{ background:var(--mint); color:var(--dentin); font-weight:700; }
  #toast.bad{ background:var(--danger); color:#fff; font-weight:700; }

  /* ---------- Overlays ---------- */
  .overlay{
    position:absolute; inset:0; z-index:50;
    background:linear-gradient(180deg, rgba(14,59,67,.96), rgba(21,80,89,.96));
    color:var(--enamel);
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    text-align:center; padding:32px 26px; overflow-y:auto;
  }
  .overlay.hidden{ display:none; }

  .ink-swirl{
    width:160px; height:160px; margin-bottom:6px; opacity:.95;
  }

  .title{
    font-family:var(--font-display); font-weight:700; font-size:clamp(28px,7vw,38px);
    margin:6px 0 4px; letter-spacing:.5px; color:var(--enamel);
  }
  .subtitle{ font-size:14px; color:var(--reveal-soft); margin-bottom:22px; max-width:320px; }

  .legend{
    display:grid; grid-template-columns:1fr 1fr; gap:8px 18px; margin:6px 0 26px;
    width:100%; max-width:340px;
  }
  .legend h4{ font-family:var(--font-display); font-size:13px; color:var(--gold); margin:0 0 6px; text-align:left; letter-spacing:.4px;}
  .legend-col{ text-align:left; }
  .legend-item{ display:flex; align-items:center; gap:8px; font-size:12.5px; margin-bottom:5px; color:var(--enamel); }
  .legend-item span.ic{ font-size:17px; width:22px; text-align:center; }

  .btn{
    font-family:var(--font-display); font-weight:600; font-size:18px;
    background:var(--reveal); color:#fff; border:none; padding:14px 36px;
    border-radius:999px; cursor:pointer; box-shadow:0 8px 0 #B82E5C, 0 10px 18px rgba(0,0,0,.3);
    transition:transform .12s, box-shadow .12s;
  }
  .btn:active{ transform:translateY(6px); box-shadow:0 2px 0 #B82E5C, 0 4px 10px rgba(0,0,0,.25); }
  .btn:focus-visible{ outline:3px solid var(--gold); outline-offset:3px; }
  .hint{ margin-top:16px; font-size:12px; color:rgba(234,246,246,.65); }

  .btn-secondary{
    font-family:var(--font-body); font-weight:600; font-size:13.5px;
    background:transparent; color:var(--enamel); border:1.5px solid rgba(234,246,246,.45);
    padding:9px 20px; border-radius:999px; cursor:pointer; margin-top:14px;
    transition:background .15s, border-color .15s;
  }
  .btn-secondary:hover{ background:rgba(255,255,255,.08); border-color:var(--mint); }
  .btn-secondary:focus-visible{ outline:3px solid var(--gold); outline-offset:3px; }

  /* ---------- Glossary ---------- */
  .gloss-list{ width:100%; max-width:380px; margin:4px 0 8px; }
  .gloss-section-title{
    font-family:var(--font-display); font-weight:600; font-size:13px; color:var(--gold);
    text-align:left; margin:18px 0 8px; letter-spacing:.4px;
  }
  .gloss-item{ display:flex; gap:12px; align-items:flex-start; text-align:left; margin-bottom:13px; }
  .gloss-icon{
    flex:0 0 auto; width:42px; height:42px; border-radius:14px;
    background:rgba(255,255,255,.12); display:flex; align-items:center; justify-content:center;
    font-size:24px;
  }
  .gloss-icon.blob-icon{
    background:radial-gradient(circle at 35% 30%, #DCE96B, var(--plaque) 60%, var(--plaque-dark) 100%);
    border-radius:46% 54% 58% 42% / 50% 45% 55% 50%;
  }
  .gloss-icon.sarro-icon{ filter:sepia(.6) saturate(1.4) brightness(.85); }
  .gloss-text h5{ margin:0 0 2px; font-family:var(--font-display); font-weight:600; font-size:14px; color:var(--enamel); }
  .gloss-text p{ margin:0; font-size:12px; line-height:1.42; color:rgba(234,246,246,.85); }

  .go-headline{ font-family:var(--font-display); font-weight:700; font-size:26px; color:var(--gold); margin-bottom:4px; }
  .go-score{ font-family:var(--font-display); font-size:42px; font-weight:700; margin-bottom:18px; }
  .stat-row{ display:flex; justify-content:space-between; width:100%; max-width:340px; font-size:13px; padding:6px 0; border-bottom:1px solid rgba(255,255,255,.12); }
  .stat-row b{ color:var(--mint); }
  .recap{ width:100%; max-width:360px; margin:18px 0 10px; text-align:left; }
  .recap-item{ display:flex; gap:10px; font-size:12.5px; margin-bottom:10px; line-height:1.4; color:var(--enamel); }
  .recap-item span.ic{ font-size:18px; flex:0 0 auto; }

  /* ---------- Touch zones (invisible) ---------- */
  #touchZones{ position:absolute; inset:0; z-index:5; touch-action:none; }

  @media (prefers-reduced-motion: reduce){
    #player, .entity-visual.powerup, .entity-visual.type-biofilm, .entity-visual.type-sarro, #player.shield::before { animation:none !important; }
  }
</style>
</head>
<body>
<div id="app">
  <div id="game">
    <div id="backdrop">
      <div class="lane-line l1"></div>
      <div class="lane-line l2"></div>
      <div id="gumline"></div>
    </div>

    <div id="entities"></div>
    <div id="player">🪥</div>

    <div id="hud">
      <div id="lives">
        <span class="tooth" data-i="0">🦷</span><span class="tooth" data-i="1">🦷</span><span class="tooth" data-i="2">🦷</span>
      </div>
      <div id="scoreWrap">0 pts</div>
    </div>
    <div id="sugarBanner">⚡ Subidón de azúcar</div>
    <div id="toast"></div>
    <div id="touchZones"></div>

    <!-- START SCREEN -->
    <div id="startScreen" class="overlay">
      <svg class="ink-swirl" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <radialGradient id="swirlGrad" cx="40%" cy="35%" r="70%">
            <stop offset="0%" stop-color="#FF9CC0"/>
            <stop offset="55%" stop-color="#E8487C"/>
            <stop offset="100%" stop-color="#9C2C53"/>
          </radialGradient>
        </defs>
        <path fill="url(#swirlGrad)" d="M100,18 C140,12 176,42 178,84 C180,124 158,150 128,166 C100,180 62,176 42,150 C22,124 16,86 36,58 C50,38 70,24 100,18 Z"/>
        <text x="100" y="116" font-size="64" text-anchor="middle">🪥</text>
      </svg>
      <div class="title">Plaque Run</div>
      <div class="subtitle">Sonrisa en Movimiento — esquiva el biofilm, usa tus herramientas y descubre para qué sirve la tinción y la profilaxis.</div>

      <div class="legend">
        <div class="legend-col">
          <h4>EVITA</h4>
          <div class="legend-item"><span class="ic">🟢</span> Biofilm</div>
          <div class="legend-item"><span class="ic">🍔</span> Restos de comida</div>
          <div class="legend-item"><span class="ic">🦠</span> Bacterias</div>
          <div class="legend-item"><span class="ic">🍬</span> Dulces (¡resta vida y acelera todo!)</div>
          <div class="legend-item"><span class="ic">🪨</span> Placa dental (¡cuidado, pega fuerte!)</div>
        </div>
        <div class="legend-col">
          <h4>RECOGE</h4>
          <div class="legend-item"><span class="ic">💧</span> Tinción reveladora</div>
          <div class="legend-item"><span class="ic">🪥</span> Cepillo dental</div>
          <div class="legend-item"><span class="ic">🧵</span> Hilo dental</div>
          <div class="legend-item"><span class="ic">🦷</span> Dentista (profilaxis)</div>
        </div>
      </div>

      <button class="btn" id="startBtn">Empezar</button>
      <button class="btn-secondary" id="glossaryBtnStart">📖 ¿Qué significa cada ícono?</button>
      <div class="hint">Desliza o usa ← → para cambiar de carril</div>
    </div>

    <!-- GAME OVER SCREEN -->
    <div id="gameOverScreen" class="overlay hidden">
      <div class="go-headline" id="goHeadline">¡La placa dental te alcanzó!</div>
      <div class="go-score" id="goScore">0</div>

      <div class="stat-row"><span>Biofilm evitado</span><b id="statBiofilm">0</b></div>
      <div class="stat-row"><span>Dulces comidos</span><b id="statCandy">0</b></div>
      <div class="stat-row"><span>Rutina usada (cepillo+hilo)</span><b id="statRoutine">0</b></div>
      <div class="stat-row"><span>Visitas al dentista</span><b id="statDentist">0</b></div>
      <div class="stat-row"><span>Rutinas completas 🎉</span><b id="statCombo">0</b></div>

      <div class="recap">
        <div class="recap-item"><span class="ic">💧</span><span>El biofilm es casi invisible a simple vista: la tinción reveladora lo tiñe para mostrarte justo dónde te falta limpiar.</span></div>
        <div class="recap-item"><span class="ic">🍬</span><span>El azúcar alimenta a las bacterias de tu boca y acelera la formación de biofilm.</span></div>
        <div class="recap-item"><span class="ic">🪥</span><span>Cepillo e hilo dental son tu rutina diaria de defensa, pero no siempre llegan a todos lados.</span></div>
        <div class="recap-item"><span class="ic">🦷</span><span>La profilaxis profesional elimina la placa dental endurecida y biofilme que tu rutina diaria no puede quitar.</span></div>
      </div>

      <button class="btn" id="retryBtn">Jugar de nuevo</button>
      <button class="btn-secondary" id="glossaryBtnGO">📖 Ver glosario de íconos</button>
    </div>

    <!-- GLOSSARY SCREEN -->
    <div id="glossaryScreen" class="overlay hidden">
      <div class="title" style="font-size:26px;">Glosario de íconos</div>
      <div class="subtitle">Cada elemento del juego representa algo real de tu salud bucal.</div>

      <div class="gloss-list">
        <div class="gloss-section-title">🚫 EVITA — dañan tus dientes</div>

        <div class="gloss-item">
          <div class="gloss-icon blob-icon"></div>
          <div class="gloss-text"><h5>Biofilm</h5><p>El biofilme dental que se forma todo el día sobre tus dientes. Contiene bacterias y es casi invisible a simple vista.</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon">🍔</div>
          <div class="gloss-text"><h5>Restos de comida</h5><p>Partículas que quedan atrapadas entre los dientes y alimentan a las bacterias si no se remueven.</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon">🦠</div>
          <div class="gloss-text"><h5>Bacterias</h5><p>Los microorganismos que viven dentro del biofilme y producen ácidos que desgastan el diente.</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon">🍬</div>
          <div class="gloss-text"><h5>Dulces</h5><p>El azúcar es el alimento favorito de las bacterias. Comerlos te quita una vida y acelera la formación de biofilme dental.</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon sarro-icon">🪨</div>
          <div class="gloss-text"><h5>Placa dental</h5><p>Biofilme que no se quitó a tiempo y se endureció (calcificó). Ya no sale con el cepillo: solo el dentista puede quitarlo.</p></div>
        </div>

        <div class="gloss-section-title">✅ RECOGE — tus herramientas</div>

        <div class="gloss-item">
          <div class="gloss-icon">💧</div>
          <div class="gloss-text"><h5>Tinción reveladora</h5><p>Un líquido o pastilla que tiñe el biofilm para que veas exactamente dónde no estás cepillando bien. ¡Nunca te hace daño, solo te ayuda!</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon">🪥</div>
          <div class="gloss-text"><h5>Cepillo dental</h5><p>Tu primera línea de defensa diaria: elimina el biofilm de la superficie de los dientes.</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon">🧵</div>
          <div class="gloss-text"><h5>Hilo dental</h5><p>Limpia los espacios entre los dientes, justo donde las cerdas del cepillo no llegan.</p></div>
        </div>
        <div class="gloss-item">
          <div class="gloss-icon">🦷</div>
          <div class="gloss-text"><h5>Dentista (profilaxis)</h5><p>La limpieza profesional que elimina la placa dental endurecida y biofilme acumulado que tu rutina diaria no puede quitar sola.</p></div>
        </div>
      </div>

      <button class="btn" id="glossaryBackBtn">Volver</button>
    </div>
  </div>
</div>

<script>
(function(){
  "use strict";

  const gameEl = document.getElementById('game');
  const entitiesEl = document.getElementById('entities');
  const playerEl = document.getElementById('player');
  const scoreWrap = document.getElementById('scoreWrap');
  const livesEl = document.getElementById('lives');
  const toastEl = document.getElementById('toast');
  const sugarBanner = document.getElementById('sugarBanner');
  const startScreen = document.getElementById('startScreen');
  const gameOverScreen = document.getElementById('gameOverScreen');
  const startBtn = document.getElementById('startBtn');
  const retryBtn = document.getElementById('retryBtn');
  const touchZones = document.getElementById('touchZones');
  const glossaryScreen = document.getElementById('glossaryScreen');
  const glossaryBtnStart = document.getElementById('glossaryBtnStart');
  const glossaryBtnGO = document.getElementById('glossaryBtnGO');
  const glossaryBackBtn = document.getElementById('glossaryBackBtn');

  const LANE_COUNT = 3;
  const PLAYER_SIZE = 60;

  const EMOJI = {
    food:'🍔', bacteria:'🦠', candy:'🍬', sarro:'🪨',
    tincion:'💧', cepillo:'🪥', hilo:'🧵', dentista:'🦷'
  };
  const SIZE = {
    biofilm:52, food:50, bacteria:46, candy:42, sarro:72,
    tincion:46, cepillo:48, hilo:48, dentista:54
  };
  const OBSTACLE_WEIGHTS = { biofilm:30, food:24, bacteria:20, candy:16, sarro:10 };
  const POWERUP_WEIGHTS  = { tincion:18, cepillo:30, hilo:30, dentista:22 };

  const FACTS = {
    tincion:"💡 El biofilm es casi invisible: la tinción reveladora lo tiñe para que puedas verlo y cepillarlo bien.",
    candy:"💡 El azúcar alimenta a las bacterias de tu boca y acelera la formación de biofilme.",
    rutina:"💡 Cepillarte y usar hilo dental a diario es tu rutina de defensa, ¡pero no llega a todos lados!",
    dentista:"💡 La profilaxis profesional elimina la placa dental endurecida y biofilme que tu rutina diaria no puede quitar."
  };

  let entities = [];
  let entityId = 0;
  let player = { lane: 1, invulnerable:false };
  let lives = 3;
  let score = 0;
  let running = false;
  let lastTime = null;
  let spawnTimer = 0;
  let laneLastObstacle = {0:-9999,1:-9999,2:-9999};
  let tincionActive = false, tincionTimer = null;
  let shieldTimer = null;
  let sugarRushDeadline = 0, sugarRushActive = false;
  let toastTimer = null;
  let factsShown = {};
  let comboTimestamps = { cepillo:null, hilo:null, dentista:null };
  let stats = { biofilmAvoided:0, candyEaten:0, routineUses:0, dentistUses:0, combos:0 };

  function getDims(){
    return { w: gameEl.clientWidth, h: gameEl.clientHeight };
  }
  function laneCenterX(lane, w){ return (lane + 0.5) * (w / LANE_COUNT); }

  function weightedPick(weights){
    const entries = Object.entries(weights);
    const total = entries.reduce((s,[,v])=>s+v,0);
    let r = Math.random()*total;
    for(const [k,v] of entries){ if(r<v) return k; r-=v; }
    return entries[0][0];
  }

  function pickObstacleLane(now, interval){
    const recentWindow = Math.max(380, interval*0.95);
    const recent = [0,1,2].filter(l => now - laneLastObstacle[l] < recentWindow);
    if(recent.length >= 2){
      const free = [0,1,2].find(l => !recent.includes(l));
      if(free !== undefined) return free;
    }
    return Math.floor(Math.random()*3);
  }

  function updateScoreUI(){
    scoreWrap.textContent = Math.floor(score) + ' pts';
  }
  function updateLivesUI(){
    const teeth = livesEl.querySelectorAll('.tooth');
    teeth.forEach((t,i)=>{ t.classList.toggle('lost', i >= lives); });
  }

  function showToast(text, variant){
    clearTimeout(toastTimer);
    toastEl.textContent = text;
    toastEl.className = 'toast show' + (variant ? ' '+variant : '');
    toastTimer = setTimeout(()=>{ toastEl.classList.remove('show'); }, 3500);
  }
  function showFactOnce(key){
    if(!factsShown[key]){ factsShown[key] = true; showToast(FACTS[key]); }
  }

  function shakeScreen(){
    gameEl.classList.remove('shake'); void gameEl.offsetWidth;
    gameEl.classList.add('shake');
    setTimeout(()=>gameEl.classList.remove('shake'), 300);
  }

  function loseLife(n){
    if(player.invulnerable) return;
    lives = Math.max(0, lives - n);
    updateLivesUI();
    shakeScreen();
    if(lives <= 0) gameOver();
  }

  function activateShield(ms){
    player.invulnerable = true;
    playerEl.classList.add('shield');
    clearTimeout(shieldTimer);
    shieldTimer = setTimeout(()=>{
      player.invulnerable = false;
      playerEl.classList.remove('shield');
    }, ms);
  }

  function activateTincion(){
    tincionActive = true;
    entities.forEach(e=>{ if(e.type==='biofilm' && e.visual) e.visual.classList.add('revealed'); });
    gameEl.classList.remove('tincion-flash'); void gameEl.offsetWidth;
    gameEl.classList.add('tincion-flash');
    clearTimeout(tincionTimer);
    tincionTimer = setTimeout(()=>{
      tincionActive = false;
      entities.forEach(e=>{ if(e.visual) e.visual.classList.remove('revealed'); });
    }, 4000);
  }

  function activateSugarRush(){
    sugarRushDeadline = performance.now() + 4000;
    if(!sugarRushActive){
      sugarRushActive = true;
      sugarBanner.classList.add('show');
    }
  }

  function removeFromArray(e){
    const idx = entities.indexOf(e);
    if(idx > -1) entities.splice(idx,1);
  }

  function clearByLaneAndTypes(lane, types){
    let count = 0;
    entities.filter(e => e.category==='obstacle' && e.lane===lane && types.includes(e.type) && !e.resolved)
      .forEach(e=>{
        e.resolved = true;
        score += 15;
        if(e.visual) e.visual.classList.add('pop');
        if(e.el){ const el = e.el; setTimeout(()=>el.remove(),220); }
        removeFromArray(e);
        count++;
      });
    return count;
  }

  function clearAllSarro(){
    entities.filter(e => e.category==='obstacle' && e.type==='sarro' && !e.resolved)
      .forEach(e=>{
        e.resolved = true;
        score += 30;
        if(e.visual) e.visual.classList.add('pop');
        if(e.el){ const el = e.el; setTimeout(()=>el.remove(),220); }
        removeFromArray(e);
      });
  }

  function adjacentLaneFor(lane){
    if(lane === 1) return Math.random() < 0.5 ? 0 : 2;
    return 1;
  }

  function recordComboUse(type){
    comboTimestamps[type] = performance.now();
    const vals = Object.values(comboTimestamps).filter(v => v != null);
    if(vals.length === 3 && (Math.max(...vals) - Math.min(...vals)) <= 12000){
      score += 500;
      stats.combos++;
      showToast('🎉 ¡Rutina Completa! +500', 'combo');
      comboTimestamps = { cepillo:null, hilo:null, dentista:null };
    }
  }

  function applyObstacleEffect(e){
    switch(e.type){
      case 'biofilm':
        if(tincionActive){ score += 25; }
        else { loseLife(1); }
        break;
      case 'food': loseLife(1); break;
      case 'bacteria': loseLife(1); break;
      case 'candy':
        stats.candyEaten++;
        loseLife(1);
        activateSugarRush();
        showToast('🍬 ¡Azúcar! -1 vida y todo se acelera', 'bad');
        showFactOnce('candy');
        break;
      case 'sarro':
        if(player.invulnerable){ score += 50; }
        else { loseLife(2); }
        break;
    }
  }

  function applyPowerupEffect(e){
    switch(e.type){
      case 'tincion':
        activateTincion();
        score += 30;
        showToast('✨ ¡Tinción aplicada! Biofilm revelado +30 pts', 'good');
        showFactOnce('tincion');
        break;
      case 'cepillo':
        score += 10;
        clearByLaneAndTypes(e.lane, ['biofilm','food']);
        stats.routineUses++;
        showFactOnce('rutina');
        recordComboUse('cepillo');
        break;
      case 'hilo':
        score += 10;
        clearByLaneAndTypes(e.lane, ['biofilm','food']);
        clearByLaneAndTypes(adjacentLaneFor(e.lane), ['biofilm','food']);
        stats.routineUses++;
        showFactOnce('rutina');
        recordComboUse('hilo');
        break;
      case 'dentista':
        score += 100;
        activateShield(4000);
        clearAllSarro();
        lives = Math.min(3, lives+1);
        updateLivesUI();
        stats.dentistUses++;
        showFactOnce('dentista');
        recordComboUse('dentista');
        break;
    }
  }

  // ---------- Spawning ----------
  // Estructura: <div class="entity">  (solo posición, la mueve el JS)
  //               <div class="entity-visual type-X">  (emoji + animaciones CSS)
  // Así el `translateY` de JS nunca compite con el `rotate`/`scale` del CSS.
  function spawnEntity(category, type, lane, now){
    const { w } = getDims();
    const size = SIZE[type] || 50;

    const el = document.createElement('div');
    el.className = 'entity';
    el.style.width = size + 'px';
    el.style.height = size + 'px';
    el.style.left = (laneCenterX(lane, w) - size/2) + 'px';
    const startY = -size;
    el.style.transform = 'translateY(' + startY + 'px)';

    const visual = document.createElement('div');
    visual.className = 'entity-visual ' + category + ' type-' + type;
    if(type !== 'biofilm') visual.textContent = EMOJI[type];
    el.appendChild(visual);

    entitiesEl.appendChild(el);
    if(type === 'biofilm' && tincionActive) visual.classList.add('revealed');

    const speedFactor = type === 'sarro' ? 0.65 : (category === 'powerup' ? 0.82 : 1);
    const entity = { id: ++entityId, category, type, lane, y:startY, size, el, visual, resolved:false, speedFactor };
    entities.push(entity);
    if(category === 'obstacle') laneLastObstacle[lane] = now;
  }

  function doSpawn(now){
    const isPowerup = Math.random() < 0.27;
    if(isPowerup){
      const type = weightedPick(POWERUP_WEIGHTS);
      const lane = Math.floor(Math.random()*3);
      spawnEntity('powerup', type, lane, now);
    } else {
      const type = weightedPick(OBSTACLE_WEIGHTS);
      const interval = Math.max(420, 1100 - score*0.15);
      const lane = pickObstacleLane(now, interval);
      spawnEntity('obstacle', type, lane, now);
    }
  }

  function resolveCollision(e){
    e.resolved = true;
    if(e.visual) e.visual.classList.add('pop');
    if(e.el){ const el = e.el; setTimeout(()=>el.remove(), 220); }
    if(e.category === 'obstacle') applyObstacleEffect(e);
    else applyPowerupEffect(e);
  }

  function update(dt, now){
    const { w, h } = getDims();

    const speedMult = (now < sugarRushDeadline) ? 1.4 : 1;
    if(sugarRushActive && now >= sugarRushDeadline){
      sugarRushActive = false;
      sugarBanner.classList.remove('show');
    }
    const baseSpeed = Math.min(620, 220 + score*0.04);
    const speed = baseSpeed * speedMult;

    score += dt * 12;
    updateScoreUI();

    spawnTimer += dt*1000;
    const interval = Math.max(420, 1100 - score*0.15);
    if(spawnTimer > interval){ spawnTimer = 0; doSpawn(now); }

    playerEl.style.left = laneCenterX(player.lane, w) + 'px';

    const playerTop = h*0.76;
    const playerBottom = playerTop + PLAYER_SIZE;

    for(let i = entities.length-1; i >= 0; i--){
      const e = entities[i];
      e.y += speed * e.speedFactor * dt;
      if(e.el) e.el.style.transform = 'translateY(' + e.y + 'px)';

      if(e.resolved) continue;

      const eBottom = e.y + e.size;
      const overlap = eBottom > playerTop && e.y < playerBottom;
      if(overlap && e.lane === player.lane){
        resolveCollision(e);
        entities.splice(i,1);
        continue;
      }
      if(e.y > h){
        if(e.category === 'obstacle' && e.type === 'biofilm') stats.biofilmAvoided++;
        if(e.el) e.el.remove();
        entities.splice(i,1);
      }
    }
  }

  function loop(ts){
    if(!running) return;
    if(lastTime === null) lastTime = ts;
    const dt = Math.min(0.05, (ts - lastTime)/1000);
    lastTime = ts;
    update(dt, ts);
    requestAnimationFrame(loop);
  }

  function setLane(delta){
    if(!running) return;
    player.lane = Math.max(0, Math.min(2, player.lane + delta));
  }

  // ---------- Controls ----------
  document.addEventListener('keydown', (ev)=>{
    if(!running) return;
    if(ev.key === 'ArrowLeft' || ev.key.toLowerCase() === 'a') setLane(-1);
    if(ev.key === 'ArrowRight' || ev.key.toLowerCase() === 'd') setLane(1);
  });

  let touchStartX = null;
  touchZones.addEventListener('pointerdown', (ev)=>{ touchStartX = ev.clientX; });
  touchZones.addEventListener('pointerup', (ev)=>{
    if(touchStartX === null || !running) return;
    const dx = ev.clientX - touchStartX;
    if(Math.abs(dx) > 28){
      setLane(dx > 0 ? 1 : -1);
    } else {
      const { w } = getDims();
      setLane(ev.clientX < w/2 ? -1 : 1);
    }
    touchStartX = null;
  });
  touchZones.addEventListener('pointercancel', ()=>{ touchStartX = null; });

  // ---------- Game flow ----------
  function resetState(){
    entities.forEach(e=>{ if(e.el) e.el.remove(); });
    entities = [];
    entityId = 0;
    player = { lane: 1, invulnerable:false };
    lives = 3;
    score = 0;
    spawnTimer = 0;
    laneLastObstacle = {0:-9999,1:-9999,2:-9999};
    tincionActive = false; clearTimeout(tincionTimer);
    clearTimeout(shieldTimer); playerEl.classList.remove('shield');
    sugarRushDeadline = 0; sugarRushActive = false; sugarBanner.classList.remove('show');
    factsShown = {};
    comboTimestamps = { cepillo:null, hilo:null, dentista:null };
    stats = { biofilmAvoided:0, candyEaten:0, routineUses:0, dentistUses:0, combos:0 };
    updateLivesUI();
    updateScoreUI();
    toastEl.classList.remove('show');
    lastTime = null;
  }

  function startGame(){
    resetState();
    startScreen.classList.add('hidden');
    gameOverScreen.classList.add('hidden');
    running = true;
    requestAnimationFrame(loop);
  }

  function gameOver(){
    running = false;
    document.getElementById('goScore').textContent = Math.floor(score);
    document.getElementById('statBiofilm').textContent = stats.biofilmAvoided;
    document.getElementById('statCandy').textContent = stats.candyEaten;
    document.getElementById('statRoutine').textContent = stats.routineUses;
    document.getElementById('statDentist').textContent = stats.dentistUses;
    document.getElementById('statCombo').textContent = stats.combos;
    document.getElementById('goHeadline').textContent =
      stats.combos > 0 ? '¡Buena rutina, pero el biofilm ganó!' : '¡El biofilm se acumuló!';
    gameOverScreen.classList.remove('hidden');
  }

  startBtn.addEventListener('click', startGame);
  retryBtn.addEventListener('click', startGame);

  let glossaryReturnScreen = null;
  function showGlossary(fromEl){
    glossaryReturnScreen = fromEl;
    fromEl.classList.add('hidden');
    glossaryScreen.classList.remove('hidden');
    glossaryScreen.scrollTop = 0;
  }
  function hideGlossary(){
    glossaryScreen.classList.add('hidden');
    if(glossaryReturnScreen) glossaryReturnScreen.classList.remove('hidden');
  }
  glossaryBtnStart.addEventListener('click', ()=> showGlossary(startScreen));
  glossaryBtnGO.addEventListener('click', ()=> showGlossary(gameOverScreen));
  glossaryBackBtn.addEventListener('click', hideGlossary);

  updateLivesUI();
  updateScoreUI();
})();
</script>
</body>
</html>
