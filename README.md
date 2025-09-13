<!doctype html>

<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Convite-Jogo — Anthony Theodoro</title>
  <style>
    /* ====== Config editável (troque as informações abaixo) ====== */
    :root{
      --nome: "Anthony Theodoro";
      --idade: "7"; /* número sem aspas */
      --data: "20/09/2025";
      --hora: "18:00";
      --local: "Rua das Flores, 123";
      --bg: #7ec0ee; /* céu */
    }html,body{height:100%;margin:0;font-family:Inter, system-ui, Arial}
body{display:flex;align-items:center;justify-content:center;background:linear-gradient(#7ec0ee,#9fd8ff);}
.frame{width:900px;max-width:96vw;background:#0f1720;border-radius:12px;box-shadow:0 10px 30px rgba(2,6,23,.6);overflow:hidden;color:#fff}
header{padding:12px 18px;display:flex;align-items:center;gap:12px;background:linear-gradient(90deg,#0b1220,#122032);}
header h1{font-size:18px;margin:0}
header p{margin:0;opacity:.8;font-size:13px}
.stage{position:relative;height:520px;background:var(--bg);}
canvas{display:block;background:transparent}
.ui{position:absolute;left:12px;top:12px;background:rgba(0,0,0,.35);padding:8px 10px;border-radius:8px;font-size:14px}
.overlay{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none}
.panel{background:rgba(3,7,18,.9);border:2px solid rgba(255,255,255,.06);padding:18px;border-radius:10px;min-width:320px;max-width:90%;color:#fff;text-align:center}
.panel h2{margin:0 0 8px 0}
.btn{display:inline-block;margin-top:10px;padding:8px 12px;border-radius:8px;background:#ffdd57;color:#000;font-weight:700;text-decoration:none}
.small{font-size:13px;opacity:.9}
footer{padding:10px 14px;background:#071021;text-align:center;font-size:13px;color:#9fb3d1}
/* mobile hint */
@media (max-width:520px){.stage{height:420px}}

  </style>
</head>
<body>
  <div class="frame">
    <header>
      <div style="width:60px;height:60px;border-radius:8px;background:linear-gradient(90deg,#ffd27a,#ff7a7a);display:flex;align-items:center;justify-content:center;font-weight:800;color:#0b1020">🍄</div>
      <div>
        <h1>Convite-jogo — <span id="nomeTitulo">Anthony Theodoro</span></h1>
        <p id="subtitulo">Toque blocos para revelar os detalhes da festa 🎉</p>
      </div>
    </header><div class="stage">
  <canvas id="game" width="900" height="520"></canvas>
  <div class="ui">Use ← → para andar • Espaço para pular • Enter para reiniciar</div>
  <div class="overlay" id="overlay" style="display:none;pointer-events:auto">
    <div class="panel" id="panel">
      <h2 id="panelTitle">Parabéns</h2>
      <div id="panelText" class="small">Você desbloqueou informações!</div>
      <a id="panelBtn" class="btn" href="#" onclick="closePanel();return false;">Continuar</a>
    </div>
  </div>
</div>

<footer>
  Convite interativo — personalize as informações no topo do arquivo e hospede em GitHub Pages / Netlify.
</footer>

  </div><script>
/* ====== Dados do convite (edite aqui se quiser usar outro texto) ====== */
const INV = {
  name: 'Anthony Theodoro',
  age: '7',
  date: '20/09/2025',
  time: '18:00',
  place: 'Rua das Flores, 123'
};

// atualiza título visível
document.getElementById('nomeTitulo').innerText = INV.name;

/* ====== Motor simples de plataforma em canvas ====== */
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;

let keys = {};
addEventListener('keydown',e=>{keys[e.code]=true; if(e.code==='Enter') init();});
addEventListener('keyup',e=>{keys[e.code]=false});

// mundo básico - blocos, chão e "blocos de informação"
const TILE = 40;
const gravity = 0.9;

let player, platforms, infoBlocks, cameraX;

function init(){
  cameraX = 0;
  player = {x:80,y: H-140, w:28, h:36, vx:0, vy:0, onGround:false};

  // plataformas: array de rect {x,y,w,h}
  platforms = [];
  // chão contínuo
  for(let i=0;i<40;i++) platforms.push({x:i*TILE,y:H-80,w:TILE,h:80});

  // plataformas flutuantes
  platforms.push({x:320,y:H-200,w:160,h:20});
  platforms.push({x:620,y:H-260,w:160,h:20});
  platforms.push({x:980,y:H-200,w:160,h:20});
  platforms.push({x:1280,y:H-240,w:160,h:20});

  // infoBlocks são blocos "marcados" que mostram mensagens quando tocados
  infoBlocks = [
    {x:360,y:H-240,w:40,h:40,msg:`📅 Data: ${INV.date}` , revealed:false},
    {x:660,y:H-300,w:40,h:40,msg:`🕒 Hora: ${INV.time}`   , revealed:false},
    {x:1020,y:H-240,w:40,h:40,msg:`📍 Local: ${INV.place}` , revealed:false},
    {x:1400,y:H-300,w:48,h:48,msg:`🎂 Festa: ${INV.name} — ${INV.age} anos` , revealed:false}
  ];

  // pequenos itens colecionáveis (moedas)
  coins = [{x:200,y:H-200,got:false},{x:240,y:H-200,got:false},{x:280,y:H-200,got:false},{x:720,y:H-320,got:false},{x:1040,y:H-280,got:false}];

  score = 0;
}

function rectsOverlap(a,b){
  return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
}

function update(){
  // movimento horizontal
  if(keys['ArrowLeft']) player.vx = -4.2;
  else if(keys['ArrowRight']) player.vx = 4.2;
  else player.vx = 0;

  // pulo
  if((keys['Space']||keys['KeyW']||keys['ArrowUp']) && player.onGround){ player.vy = -14; player.onGround=false; }

  // física
  player.vy += gravity;
  player.x += player.vx;
  player.y += player.vy;

  // colisão com plataformas
  player.onGround = false;
  for(let p of platforms){
    if(rectsOverlap(player,p)){
      // simples correção de posição: checar de cima
      if(player.vy > 0 && player.y + player.h - player.vy <= p.y){
        player.y = p.y - player.h; player.vy = 0; player.onGround = true;
      } else if(player.vy < 0 && player.y >= p.y + p.h - 1){
        player.y = p.y + p.h; player.vy = 0;
      } else {
        // colisão lateral — empurra para fora
        if(player.x < p.x) player.x = p.x - player.w; else player.x = p.x + p.w;
      }
    }
  }

  // limites verticais
  if(player.y > H + 200) { // caiu fora
    showPanel('Ops! Você caiu. Pressione Enter para tentar de novo.','Reiniciar');
  }

  // checar colisão com infoBlocks
  for(let b of infoBlocks){
    if(!b.revealed && rectsOverlap(player,b)){
      b.revealed = true;
      showPanel(b.msg,'Fechar');
    }
  }

  // coins
  for(let c of coins){ if(!c.got){ if(player.x+player.w>c.x && player.x<c.x+24 && player.y+player.h>c.y && player.y<c.y+24){ c.got=true; score++; if(score===coins.length) showPanel(`🎉 Você pegou todas as moedas! Fase final: ${INV.name} te espera na festa.`,`OK`); } } }

  // câmera segue o jogador
  cameraX = Math.max(0, player.x - 160);
}

function draw(){
  // fundo
  ctx.clearRect(0,0,W,H);

  // nuvens simples
  for(let i=0;i<6;i++){
    const cx = (i*300 - (cameraX*0.2 % 900)) % 900;
    ctx.fillStyle = 'rgba(255,255,255,0.85)';
    ctx.beginPath(); ctx.ellipse(cx+80,80,40,26,0,0,Math.PI*2); ctx.fill();
  }

  // desenha plataformas
  for(let p of platforms){
    ctx.fillStyle = '#6c4f2b';
    ctx.fillRect(p.x - cameraX, p.y, p.w, p.h);
    // detalhes do topo
    ctx.fillStyle = '#85a04a';
    ctx.fillRect(p.x - cameraX, p.y - 6, p.w, 6);
  }

  // desenha info blocks
  for(let b of infoBlocks){
    ctx.fillStyle = b.revealed ? '#ffd27a' : '#e8b34a';
    ctx.fillRect(b.x - cameraX, b.y, b.w, b.h);
    // ponto de interrogação
    ctx.fillStyle = '#2a1a0e';
    ctx.font = '20px Arial'; ctx.textAlign='center'; ctx.textBaseline='middle';
    ctx.fillText('?', b.x - cameraX + b.w/2, b.y + b.h/2 + 2);
  }

  // desenha coins
  for(let c of coins){ if(!c.got){ ctx.beginPath(); ctx.arc(c.x - cameraX + 12, c.y + 12, 10,0,Math.PI*2); ctx.fillStyle='gold'; ctx.fill(); ctx.fillStyle='#000'; ctx.font='10px Arial'; ctx.fillText('¢', c.x - cameraX + 12, c.y + 14); } }

  // desenha jogador (simples sprite)
  ctx.fillStyle = '#ff4545';
  ctx.fillRect(player.x - cameraX, player.y, player.w, player.h);
  // olho
  ctx.fillStyle='#fff'; ctx.fillRect(player.x - cameraX + 6, player.y + 8, 6,6);
  ctx.fillStyle='#000'; ctx.fillRect(player.x - cameraX + 8, player.y + 10, 2,2);

  // HUD
  ctx.fillStyle='rgba(0,0,0,0.35)'; ctx.fillRect(12,12,120,36);
  ctx.fillStyle='#fff'; ctx.font='14px Arial'; ctx.textAlign='left'; ctx.fillText(`Moedas: ${score}`,20,34);
}

let lastTime = 0;
function loop(ts){
  const dt = ts - lastTime; lastTime = ts;
  update(); draw();
  requestAnimationFrame(loop);
}

// painel modal
function showPanel(text,btn){
  const ov = document.getElementById('overlay');
  const t = document.getElementById('panelText');
  const title = document.getElementById('panelTitle');
  const btnEl = document.getElementById('panelBtn');
  title.innerText = `🎮 Convite — ${INV.name}`;
  t.innerHTML = text.replace(/\n/g,'<br>');
  btnEl.innerText = btn || 'Fechar';
  ov.style.display = 'flex';
}
function closePanel(){ document.getElementById('overlay').style.display='none'; }

// start
init(); requestAnimationFrame(loop);

// orientações de edição rápida no console (opcional)
console.log('Convite-jogo carregado. Para editar os dados, altere a constante INV no topo do arquivo.');
</script></body>
</html>
