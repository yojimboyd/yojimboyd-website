<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>James Boyd — Terminal</title>

<!-- Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Barlow+Condensed:wght@600;800;900&family=Inter:wght@400;600;700&display=swap" rel="stylesheet">

<style>
:root{
  /* Perspective + vanishing point */
  --vp-x: 50vw;
  --vp-y: 42vh;
  --persp: 1300px;
  --tilt: 22deg;
  --depth: 880px;

  /* Midnight Runway palette */
  --tarmac:#020c1b;
  --terminal:#04142e;
  --sodium:#0a254d;
  --line:#123567;

  --sign-yellow:#00faff;
  --sign-amber:#00c6ff;
  --white:#e2f7ff;
  --muted:#7da3c7;

  --glass-line: rgba(0, 200, 255, .22);
}

*{ box-sizing:border-box; }
html,body{ height:100%; }
body{
  margin:0; color:var(--white);
  font-family:"Inter", system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
  background: transparent;         /* video shows through */
  overflow-x:hidden;
}

/* Fullscreen YouTube video background */
.bg-video{
  position: fixed;
  inset: 0;
  z-index: -2;                     /* behind everything */
  overflow: hidden;
  pointer-events: none;
}
.bg-video iframe{
  width: 100vw;
  height: 56.25vw;                 /* 16:9 fit by width */
  min-height: 100vh;
  min-width: 177.78vh;             /* 16:9 fit by height */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
  filter: brightness(.75) saturate(1.05); /* keep content legible */
}

/* Optional extra contrast (toggle opacity if needed) */
.video-overlay{
  position: fixed; inset:0; z-index:-1; pointer-events:none;
  background: linear-gradient(180deg, rgba(0,8,20,.45), rgba(0,8,20,.25));
}

/* Scene */
.scene{
  position:relative; z-index:2; min-height:100svh;
  perspective: var(--persp);
  perspective-origin: var(--vp-x) var(--vp-y);
}

/* Title as a signage header */
.header{
  position:relative; z-index:3;
  display:grid; place-items:center; margin: clamp(12px, 4vh, 28px) 0;
}
.wordmark{
  font-family:"Barlow Condensed", system-ui, sans-serif;
  font-weight:900;
  font-size: clamp(42px, 9vw, 120px);
  letter-spacing:.02em;
  background: linear-gradient(180deg, var(--sign-yellow), var(--sign-amber));
  -webkit-background-clip:text; background-clip:text; color:transparent;
  text-shadow: 0 0 18px #00e6ff40, 0 0 42px #00e6ff26;
  position:relative;
}
.wordmark::after{
  content:""; position:absolute; left:0; right:0; bottom:.08em; height:2px;
  background: linear-gradient(90deg, transparent, #00d8ff80 20%, transparent 50%, #00d8ff80 80%, transparent);
  filter: blur(.5px); opacity:.6; animation: tube 6s ease-in-out infinite;
}
@keyframes tube{ 0%,100%{opacity:.4} 50%{opacity:.9} }

/* ===== Background Lyric Board (fading, behind content) ===== */
.lyric-board{
  position:fixed;
  left:50%; top:40%;
  transform: translate(-50%,-50%);
  z-index:1; /* stay behind cards */
  pointer-events:none;
  text-align:center;
  width:min(92vw, 1200px);
  opacity:.12; /* faint */
  filter: blur(.1px);
}
.lyric-board .line{
  font-family: "Barlow Condensed", system-ui, sans-serif;
  font-weight:800;
  letter-spacing:.08em;
  text-transform:uppercase;
  color:#d7f3ff;
  font-size: clamp(18px, 2.6vw, 34px);
  opacity:0;
  animation: lyricFade 16s infinite;
}
.lyric-board .line:nth-child(2){ animation-delay: 4s; }
.lyric-board .line:nth-child(3){ animation-delay: 8s; }
.lyric-board .line:nth-child(4){ animation-delay: 12s; }
@keyframes lyricFade{
  0%,20%{opacity:0; transform: translateY(6px)}
  30%,60%{opacity:1; transform: translateY(0)}
  80%,100%{opacity:0; transform: translateY(-6px)}
}

/* ===== Perspective Cards (foreground) ===== */
.cards{
  position:relative; z-index:4; /* in front of lyric board */
  display:grid;
  grid-template-columns: repeat(auto-fit, minmax(420px, 1fr)); /* collapses exactly when needed */
  gap: 2.2rem;
  width:min(1300px, 92vw);
  margin: clamp(4px, 2vh, 16px) auto 0;
  transform-style: preserve-3d;
}
.card{
  position:relative;
  min-height: 430px;
  border-radius: 16px;
  overflow:hidden;
  background: linear-gradient(180deg, rgba(6,10,18,.85), rgba(10,16,28,.85));
  border:1px solid var(--glass-line);
  box-shadow: 0 46px 110px rgba(0,0,0,.65), inset 0 2px 0 rgba(255,255,255,.07);
  transition: transform .5s ease;
}
.card.left  { transform: rotateY(var(--tilt))  translateZ(calc(var(--depth) * -1)) translateX(-2.2vw); transform-origin:right center; }
.card.right { transform: rotateY(calc(var(--tilt) * -1)) translateZ(calc(var(--depth) * -1)) translateX( 2.2vw); transform-origin:left  center; }
/* keep angle on hover (no popout) */

/* Labels */
.card .label{
  position:absolute; left:0; right:0; top:0; padding:12px 16px;
  text-transform:uppercase; letter-spacing:.14em; font-weight:800; color:#05121f;
  background: linear-gradient(180deg, var(--sign-yellow), var(--sign-amber));
  border-bottom:1px solid #0002; box-shadow: inset 0 1px 0 #fff5;
}

/* Video */
.card.video .inside{ position:absolute; inset:56px 18px 18px; border-radius:14px; overflow:hidden; border:1px solid rgba(255,255,255,.16); background:#0008; }
.card.video iframe{ position:absolute; inset:0; width:100%; height:100%; border:0; }

/* Dates */
.card.dates .inside{ position:absolute; inset:56px 16px 16px; padding:14px; border-radius:14px; background: linear-gradient(180deg, #0e1422cc, #0a0f18cc); border:1px solid rgba(255,255,255,.16); }
.dates-list{ list-style:none; padding:0; margin:0; }
.dates-list li{ display:flex; justify-content:space-between; padding:12px 10px; border-bottom:1px dashed rgba(255,255,255,.18); font-size:1.05rem; }
.dates-list li:last-child{ border-bottom:none; }
.dates-list strong{ color:#eaf2ff; }
.dates-list span{ color:var(--muted); }

/* Listen (boarding-pass grid that fills the space) */
.card.listen .inside{
  position:absolute; inset:56px 16px 16px;
  display:grid; grid-template-columns: repeat(2, minmax(0,1fr)); gap:14px;
}
@media (max-width:760px){ .card.listen .inside{ grid-template-columns:1fr; } }
.pass{
  display:flex; align-items:center; justify-content:center; text-align:center;
  padding:18px 16px; border-radius:12px; text-decoration:none; font-weight:800;
  color:#0b0f14; background:#fff;
  border:1px solid #fff; box-shadow: 0 10px 28px rgba(0,0,0,.4);
  transition: transform .18s ease, filter .18s ease;
}
.pass:hover{ transform: translateY(-2px); filter: brightness(1.04); }
.pass.cyan{    background:linear-gradient(180deg,#e6fbff,#c9f5ff); }
.pass.amber{   background:linear-gradient(180deg,#dff7ff,#bfeeff); color:#002233; }
.pass.violet{  background:linear-gradient(180deg,#efe9ff,#ddccff); }
.pass.mint{    background:linear-gradient(180deg,#e8fff7,#caffee); }

/* Photos */
.card.photos .inside{
  position:absolute; inset:56px 16px 16px;
  display:grid; grid-template-columns:repeat(2,1fr); gap:12px;
}
.card.photos img{
  width:100%; height:160px; object-fit:cover; border-radius:10px; opacity:.95;
  filter:saturate(1.05) contrast(1.03);
}

/* Footer */
footer{ text-align:center; color:var(--muted); padding:56px 0 80px; }

@media (max-width: 980px){
  .cards{ grid-template-columns:1fr; }
  .card.left,.card.right{ transform:none; }
  .card{ min-height:360px; }
}

/* Reduced motion: hide video, show a static gradient fallback */
@media (prefers-reduced-motion: reduce){
  .bg-video{ display:none; }
  .video-overlay{ display:none; }
  body{
    background:
      radial-gradient(80vh 60vh at 50% 10%, #111a33 0%, transparent 60%),
      radial-gradient(100vh 50vh at 50% 100%, #0b1220 0%, transparent 60%),
      linear-gradient(var(--tarmac), var(--terminal) 40%, #070b12 100%);
  }
}
</style>
</head>
<body>

<!-- Fullscreen Background Video -->
<div class="bg-video">
  <iframe
    src="https://www.youtube.com/embed/7kYr5IXa4a0?autoplay=1&mute=1&loop=1&playlist=7kYr5IXa4a0&controls=0&showinfo=0&modestbranding=1&playsinline=1&rel=0"
    title="Background Video"
    frameborder="0"
    allow="autoplay; fullscreen"
    allowfullscreen
  ></iframe>
</div>
<div class="video-overlay" aria-hidden="true"></div>

<div class="scene">
  <!-- Title -->
  <div class="header">
    <div class="wordmark">James&nbsp;Boyd</div>
  </div>

  <!-- BACKGROUND LYRIC BOARD (fades, behind cards) -->
  <div class="lyric-board" id="lyric-board"></div>

  <!-- FOREGROUND PERSPECTIVE CARDS -->
  <section class="cards">
    <!-- Video -->
    <article class="card video left">
      <div class="label">Video</div>
      <div class="inside">
        <iframe
          src="https://www.youtube.com/embed/5g09jWyo_8k"
          title="James Boyd — Live"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
          allowfullscreen></iframe>
      </div>
    </article>

    <!-- Dates -->
    <article class="card dates right">
      <div class="label">Upcoming Dates</div>
      <div class="inside">
        <ul class="dates-list" id="dates-list"></ul>
      </div>
    </article>

    <!-- Listen -->
    <article class="card listen left">
      <div class="label">Listen</div>
      <div class="inside">
        <!-- Replace # with your URLs -->
        <a class="pass cyan"   href="#" target="_blank" rel="noopener">Spotify</a>
        <a class="pass amber"  href="#" target="_blank" rel="noopener">Apple Music</a>
        <a class="pass violet" href="#" target="_blank" rel="noopener">Amazon Music</a>
        <a class="pass mint"   href="#" target="_blank" rel="noopener">TIDAL</a>
      </div>
    </article>

    <!-- Photos -->
    <article class="card photos right">
      <div class="label">Photos</div>
      <div class="inside">
        <img src="https://images.unsplash.com/photo-1511379938547-c1f69419868d?q=80&w=1200&auto=format&fit=crop" alt="">
        <img src="https://images.unsplash.com/photo-1514516870926-20598945bdf7?q=80&w=1200&auto=format&fit=crop" alt="">
        <img src="https://images.unsplash.com/photo-1521334884684-d80222895322?q=80&w=1200&auto=format&fit=crop" alt="">
        <img src="https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?q=80&w=1200&auto=format&fit=crop" alt="">
      </div>
    </article>
  </section>
</div>

<footer>© <span id="year"></span> James Boyd</footer>

<!-- ===== Inline JSON (edit here) ===== -->
<script id="dates-data" type="application/json">
[
  { "date":"2025-09-10", "city":"London, UK" },
  { "date":"2025-09-18", "city":"Manchester, UK" },
  { "date":"2025-10-02", "city":"Dublin, IE" }
]
</script>

<!-- Lyric lines for the background board -->
<script id="lyrics-data" type="application/json">
[
  "faces through the trees",
  "they're here for me.",
  "shame on them",
  "no telling him",
  "walls were coming down",
  "stars on the ceiling",
  "darkness dreaming",
  "under a belt of suns",
  "sky has fallen in",
  "cast to the wind",
  "it's too late",
  "spit it out",
  "make a sound",
  "who do I owe this to?",
  "where's the line?",
  "running through?",
  "can't be mine...",
  "all mine to lose?",
  "cells divide",
  "into pieces",
  "all those days",
  "what's wrong with me?",
  "all the worry left"
]
</script>

<script>
/* Footer year */
document.getElementById('year').textContent = new Date().getFullYear();

/* Render dates from JSON */
(() => {
  const list = document.getElementById('dates-list');
  const raw = document.getElementById('dates-data');
  if (!list || !raw) return;
  let items = [];
  try { items = JSON.parse(raw.textContent.trim()); } catch(e){ items = []; }
  items.sort((a,b)=> new Date(a.date) - new Date(b.date));
  list.innerHTML = items.map(it=>{
    const d = new Date(it.date + 'T12:00:00');
    const pretty = d.toLocaleDateString(undefined,{year:'numeric',month:'short',day:'2-digit'});
    return `<li><strong>${pretty}</strong><span> — ${it.city||""}</span></li>`;
  }).join('');
})();

/* Build the background lyric board (fading) */
(() => {
  const mount = document.getElementById('lyric-board');
  const raw = document.getElementById('lyrics-data');
  if (!mount || !raw) return;
  let lines = [];
  try { lines = JSON.parse(raw.textContent.trim()); } catch(e){ lines = []; }
  if (!lines.length) return;

  // Shuffle a copy so order varies
  lines = lines.slice().sort(()=> Math.random() - 0.5);

  // Render up to 10 lines
  const count = Math.min(lines.length, 10);
  for (let i=0; i<count; i++){
    const div = document.createElement('div');
    div.className = 'line';
    div.textContent = lines[i].toUpperCase();
    div.style.animationDelay = `${i * 3.2}s`;  // stagger
    mount.appendChild(div);
  }
})();
</script>
</body>
</html>
