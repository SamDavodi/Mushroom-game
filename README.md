💥# Mushroom-game 
I wrote this game for the activity. Simple mushroom game 

<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mushroom Network</title>
<style>
* { margin:0; padding:0; box-sizing:border-box; }
body { background:#000; overflow:hidden; font-family:monospace; }
canvas { display:block; }
#ui {
  position:fixed; top:0; left:0; right:0;
  display:flex; justify-content:space-between;
  padding:10px 20px;
  color:#00eeff; font-size:14px; font-weight:bold;
  background:rgba(0,0,20,0.8);
  text-shadow:0 0 8px #00eeff;
  z-index:5;
}
.overlay {
  position:fixed; inset:0;
  display:flex; flex-direction:column;
  align-items:center; justify-content:center;
  background:rgba(0,0,20,0.9);
  color:#00eeff; gap:18px; z-index:10;
}
.overlay h1 { font-size:36px; text-shadow:0 0 20px #00eeff; letter-spacing:4px; }
.overlay p { color:#88ccff; font-size:13px; line-height:2; text-align:center; }
.btn {
  padding:12px 36px; background:transparent;
  border:2px solid #00eeff; color:#00eeff;
  font-family:monospace; font-size:14px; font-weight:bold;
  letter-spacing:2px; cursor:pointer;
  box-shadow:0 0 15px rgba(0,200,255,0.3);
}
.btn:hover { background:rgba(0,200,255,0.15); }
.hidden { display:none !important; }
#mctrl {
  position:fixed; bottom:15px; left:50%;
  transform:translateX(-50%);
  display:flex; gap:12px; z-index:5;
}
.cb {
  width:55px; height:55px;
  background:rgba(0,200,255,0.15);
  border:2px solid #00eeff; color:#00eeff;
  font-size:20px; border-radius:10px;
  cursor:pointer; display:flex;
  align-items:center; justify-content:center;
  user-select:none;
}
.cb:active { background:rgba(0,200,255,0.35); }
@media(min-width:600px){ #mctrl{display:none} }
</style>
</head>
<body>
<div id="ui">
  <span>امتیاز: <b id="sc">0</b></span>
  <span>مرحله: <b id="lv">1</b></span>
  <span id="hp">♥♥❤️♥</span>
</div>
<canvas id="c"></canvas>
<div id="mctrl">
  <div class="cb" id="bl">◀</div>
  <div class="cb" id="bj">▲</div>
  <div class="cb" id="br">▶</div>
</div>
<div class="overlay" id="s0">
  <div style="font-size:70px">🍄</div>
  <h1>MUSHROOM NET</h1>
  <p>← → برای حرکت<br>SPACE برای پریدن<br>روی دشمنان بپر!</p>
  <button class="btn" onclick="startGame()">شروع بازی</button>
</div>
<div class="overlay hidden" id="s1">
  <h1>GAME OVER</h1>
  <div style="font-size:32px;color:#fff" id="fs">0</div>
  <button class="btn" onclick="startGame()">دوباره</button>
</div>
<div class="overlay hidden" id="s2">
  <div style="font-size:60px">🏆</div>
  <h1>مرحله بعد!</h1>
  <button class="btn" onclick="nextLv()">ادامه</button>
</div>

<script>
const cv = document.getElementById('c');
const cx = cv.getContext('2d');
cv.width = window.innerWidth;
cv.height = window.innerHeight;
window.onresize = () => { cv.width=innerWidth; cv.height=innerHeight; if(gs==='play') init(); };

const W = () => cv.width, H = () => cv.height;
let gs='menu', score=0, lives=3, lv=1;
let plats=[], ens=[], coins=[], parts=[], goal=null;
let raf;

const K={};
window.onkeydown = e => K[e.code]=1;
window.onkeyup = e => K[e.code]=0;

// Mobile
const bl=document.getElementById('bl');
const br=document.getElementById('br');
const bj=document.getElementById('bj');
bl.ontouchstart=()=>K['ArrowLeft']=1; bl.ontouchend=()=>K['ArrowLeft']=0;
br.ontouchstart=()=>K['ArrowRight']=1; br.ontouchend=()=>K['ArrowRight']=0;
bj.ontouchstart=()=>K['Space']=1; bj.ontouchend=()=>K['Space']=0;

const P = {
  x:80, y:200, w:32, h:36,
  vx:0, vy:0, onG:false, jc:0, inv:0, face:1,
  trail:[]
};

const G=0.55, JF=-13, SP=5;
let jp=false;

function init(){
  const w=W(), h=H();
  plats=[{x:0,y:h-35,w:w,h:35}];
  const configs=[
    [{x:60,y:h-160,w:130,h:15},{x:240,y:h-240,w:130,h:15},{x:430,y:h-185,w:130,h:15},{x:620,y:h-280,w:130,h:15},{x:w-220,y:h-320,w:160,h:15}],
    [{x:50,y:h-180,w:100,h:15},{x:210,y:h-270,w:100,h:15},{x:370,y:h-210,w:100,h:15},{x:530,y:h-310,w:100,h:15},{x:690,y:h-250,w:100,h:15},{x:w-200,y:h-340,w:150,h:15}],

[{x:40,y:h-200,w:85,h:15},{x:185,y:h-290,w:85,h:15},{x:330,y:h-230,w:85,h:15},{x:475,y:h-340,w:85,h:15},{x:620,y:h-270,w:85,h:15},{x:765,y:h-370,w:85,h:15},{x:w-190,y:h-360,w:150,h:15}]
  ];
  const li=Math.min(lv-1,2);
  plats.push(...configs[li]);

  const ec=2+lv*2;
  ens=[];
  for(let i=0;i<ec;i++){
    ens.push({x:180+i*Math.floor((w-250)/ec),y:h-35-28,w:28,h:28,vx:(i%2?1:-1)*(1+lv*0.3),alive:true});
  }

  coins=[];
  const cc=4+lv*3;
  for(let i=0;i<cc;i++){
    coins.push({x:120+i*Math.floor((w-180)/cc),y:h-90-Math.random()*180,r:9,got:false,b:Math.random()*6.28});
  }

  const lp=plats[plats.length-1];
  goal={x:lp.x+lp.w/2-15,y:lp.y-50,w:30,h:50};

  P.x=80; P.y=h-120; P.vx=0; P.vy=0; P.onG=false; P.jc=0; P.inv=0; P.trail=[];
  parts=[];

  document.getElementById('sc').textContent=score;
  document.getElementById('lv').textContent=lv;
  setHP();
}

function setHP(){
  let s='';
  for(let i=0;i<3;i++) s+=i<lives?'<span style="color:#ff4466">♥❤️</span>':'<span style="opacity:.2">♥</span>';
  document.getElementById('hp').innerHTML=s;
}

function hit(a,b){
  return a.x<b.x+b.w&&a.x+a.w>b.x&&a.y<b.y+b.h&&a.y+a.h>b.y;
}

function loseLive(){
  lives--;setHP();
  spawnP(P.x+P.w/2,P.y+P.h/2,'#ff4466',14);
  if(lives<=0){
    gs='over';
    document.getElementById('fs').textContent='امتیاز: '+score;
    document.getElementById('s1').classList.remove('hidden');
  } else {
    P.x=80;P.y=H()-120;P.vx=0;P.vy=0;P.inv=100;
  }
}

function spawnP(x,y,col,n){
  for(let i=0;i<n;i++){
    const a=Math.random()*6.28,s=2+Math.random()*4;
    parts.push({x,y,vx:Math.cos(a)*s,vy:Math.sin(a)*s-2,life:1,d:0.03+Math.random()*0.03,sz:3+Math.random()*4,col});
  }
}

function update(){
  P.vx=0;
  if(K['ArrowLeft']){P.vx=-SP;P.face=-1;}
  if(K['ArrowRight']){P.vx=SP;P.face=1;}
  const wj=K['Space']||K['ArrowUp'];
  if(wj&&!jp&&P.jc<2){P.vy=JF;P.jc++;jp=true;spawnP(P.x+P.w/2,P.y+P.h,'#00eeff',5);}
  if(!wj)jp=false;

  P.trail.push({x:P.x+P.w/2,y:P.y+P.h/2});
  if(P.trail.length>10)P.trail.shift();

  P.vy+=G; P.x+=P.vx; P.y+=P.vy;
  P.onG=false;

  for(const pl of plats){
    if(hit(P,pl)){
      const ox=Math.min(P.x+P.w,pl.x+pl.w)-Math.max(P.x,pl.x);
      const oy=Math.min(P.y+P.h,pl.y+pl.h)-Math.max(P.y,pl.y);
      if(oy<ox){
        if(P.y+P.h/2<pl.y+pl.h/2){P.y=pl.y-P.h;P.vy=0;P.onG=true;P.jc=0;}
        else{P.y=pl.y+pl.h;P.vy=0;}
      } else {
        if(P.x+P.w/2<pl.x+pl.w/2)P.x=pl.x-P.w;
        else P.x=pl.x+pl.w;
        P.vx=0;
      }
    }
  }

  if(P.x<0)P.x=0;
  if(P.x+P.w>W())P.x=W()-P.w;
  if(P.y>H()+80)loseLive();

  for(const e of ens){
    if(!e.alive)continue;
    e.x+=e.vx;
    if(e.x<=0||e.x+e.w>=W())e.vx*=-1;
    for(const pl of plats){
      if(hit(e,pl)){
        const ox=Math.min(e.x+e.w,pl.x+pl.w)-Math.max(e.x,pl.x);
        const oy=Math.min(e.y+e.h,pl.y+pl.h)-Math.max(e.y,pl.y);
        if(ox<oy)e.vx*=-1;
      }
    }
    if(P.inv<=0&&hit(P,e)){
      if(P.vy>0&&P.y+P.h<e.y+e.h*0.55){
        e.alive=false;P.vy=JF*0.65;
        score+=100*lv;
        document.getElementById('sc').textContent=score;
        spawnP(e.x+e.w/2,e.y,'#ff6600',12);
      } else {
        loseLive();P.inv=90;
      }
    }
  }
  if(P.inv>0)P.inv--;

  for(const c of coins){
    if(c.got)continue;
    c.b+=0.06;
    if(Math.hypot(P.x+P.w/2-c.x,P.y+P.h/2-c.y)<P.w/2+c.r){
      c.got=true;score+=10*lv;
      document.getElementById('sc').textContent=score;
      spawnP(c.x,c.y,'#ffdd00',7);
    }
  }

  if(goal&&hit(P,goal)){
    gs='win';
    document.getElementById('s2').classList.remove('hidden');
  }

  for(const p of parts){p.x+=p.vx;p.y+=p.vy;p.vy+=0.12;p.life-=p.d;}
  parts=parts.filter(p=>p.life>0);
}

function draw(){
  cx.fillStyle='#000015';
  cx.fillRect(0,0,W(),H());

  // Grid
  const t=Date.now()*0.001;
  cx.strokeStyle=rgba(0,180,255,${0.06+Math.sin(t)*0.02});
  cx.lineWidth=0.5;
  for(let x=0;x<W();x+=40){cx.beginPath();cx.moveTo(x,0);cx.lineTo(x,H());cx.stroke();}
  for(let y=0;y<H();y+=40){cx.beginPath();cx.moveTo(0,y);cx.lineTo(W(),y);cx.stroke();}

// Network lines
  const nodes=[
    [W()*0.05,H()*0.15],[W()*0.9,H()*0.2],[W()*0.08,H()*0.75],
    [W()*0.85,H()*0.8],[W()*0.5,H()*0.1]
  ];
  cx.strokeStyle=rgba(0,100,255,0.08);cx.lineWidth=0.8;
  for(let i=0;i<nodes.length;i++)for(let j=i+1;j<nodes.length;j++){
    cx.beginPath();cx.moveTo(nodes[i][0],nodes[i][1]);cx.lineTo(nodes[j][0],nodes[j][1]);cx.stroke();
  }
  for(const n of nodes){
    cx.beginPath();cx.arc(n[0],n[1],3,0,6.28);
    cx.fillStyle='#00eeff';cx.shadowColor='#00eeff';cx.shadowBlur=8;cx.fill();cx.shadowBlur=0;
  }

  if(gs!=='play')return;

  // Platforms
  for(const pl of plats){
    cx.shadowColor='#00aaff';cx.shadowBlur=8;
    const g=cx.createLinearGradient(pl.x,pl.y,pl.x,pl.y+pl.h);
    g.addColorStop(0,'rgba(0,160,255,0.9)');g.addColorStop(1,'rgba(0,40,160,0.8)');
    cx.fillStyle=g;cx.fillRect(pl.x,pl.y,pl.w,pl.h);
    cx.fillStyle='rgba(120,220,255,0.5)';cx.fillRect(pl.x,pl.y,pl.w,3);
    cx.shadowBlur=0;
  }

  // Goal flag
  if(goal){
    const gp=Math.sin(t*3)*4;
    cx.strokeStyle='#00ff88';cx.lineWidth=3;cx.shadowColor='#00ff88';cx.shadowBlur=12;
    cx.beginPath();cx.moveTo(goal.x+15,goal.y+goal.h);cx.lineTo(goal.x+15,goal.y-10);cx.stroke();
    cx.fillStyle=rgba(0,255,136,${0.6+Math.sin(t*4)*0.3});
    cx.beginPath();cx.moveTo(goal.x+15,goal.y-10);cx.lineTo(goal.x+40,goal.y+5);cx.lineTo(goal.x+15,goal.y+18);cx.fill();
    cx.shadowBlur=0;
  }

  // Coins
  for(const c of coins){
    if(c.got)continue;
    const by=Math.sin(c.b)*4;
    cx.shadowColor='#ffcc00';cx.shadowBlur=12;
    cx.fillStyle='#ffcc00';
    cx.beginPath();cx.arc(c.x,c.y+by,c.r,0,6.28);cx.fill();
    cx.fillStyle='rgba(0,0,0,0.4)';cx.font=${c.r}px serif;
    cx.textAlign='center';cx.textBaseline='middle';
    cx.fillText('★',c.x,c.y+by);cx.shadowBlur=0;
  }

  // Enemies
  for(const e of ens){
    if(!e.alive)continue;
    const ex=e.x+e.w/2,ey=e.y+e.h/2;
    cx.shadowColor='#ff0055';cx.shadowBlur=14;
    cx.fillStyle='#ff1166';
    cx.beginPath();cx.arc(ex,ey-4,13,0,6.28);cx.fill();
    cx.fillStyle='rgba(255,255,255,0.7)';
    cx.beginPath();cx.arc(ex-4,ey-8,3,0,6.28);cx.fill();
    cx.beginPath();cx.arc(ex+4,ey-5,2.5,0,6.28);cx.fill();
    cx.fillStyle='#fff';
    cx.beginPath();cx.arc(ex-4,ey,4,0,6.28);cx.fill();
    cx.beginPath();cx.arc(ex+4,ey,4,0,6.28);cx.fill();
    cx.fillStyle='#111';
    cx.beginPath();cx.arc(ex-3+(e.vx>0?1:-1),ey,2,0,6.28);cx.fill();
    cx.beginPath();cx.arc(ex+5+(e.vx>0?1:-1),ey,2,0,6.28);cx.fill();
    cx.fillStyle='#cc0044';
    cx.beginPath();cx.ellipse(ex-6,e.y+e.h-2,6,4,0,0,6.28);cx.fill();
    cx.beginPath();cx.ellipse(ex+6,e.y+e.h-2,6,4,0,0,6.28);cx.fill();
    cx.shadowBlur=0;
  }

  // Player trail
  for(let i=0;i<P.trail.length;i++){
    const tr=P.trail[i],al=(i/P.trail.length)*0.35;
    cx.beginPath();cx.arc(tr.x,tr.y,5*(i/P.trail.length),0,6.28);
    cx.fillStyle=rgba(0,200,255,${al});cx.fill();
  }

  // Player
  if(P.inv>0&&Math.floor(P.inv/6)%2===0){}
  else {
    const px=P.x+P.w/2;
    cx.shadowColor='#00eeff';cx.shadowBlur=16;
    // Cap
    cx.fillStyle='#0033ee';
    cx.beginPath();cx.ellipse(px,P.y+10,18,12,0,Math.PI,0);cx.fill();
    cx.fillStyle='rgba(0,220,255,0.8)';
    cx.beginPath();cx.arc(px-6,P.y+7,3.5,0,6.28);cx.fill();
    cx.beginPath();cx.arc(px+6,P.y+7,3.5,0,6.28);cx.fill();
    cx.beginPath();cx.arc(px,P.y+4,2.5,0,6.28);cx.fill();
    // Body
    cx.fillStyle='#00aaee';
    cx.fillRect(P.x+7,P.y+18,P.w-14,13);
    // Eyes
    const fx=px+P.face*3;
    cx.fillStyle='#fff';
    cx.beginPath();cx.arc(fx-4,P.y+23,3.5,0,6.28);cx.fill();
    cx.beginPath();cx.arc(fx+4,P.y+23,3.5,0,6.28);cx.fill();
    cx.fillStyle='#001';
    cx.beginPath();cx.arc(fx-3+P.face,P.y+23,1.8,0,6.28);cx.fill();
    cx.beginPath();cx.arc(fx+5+P.face,P.y+23,1.8,0,6.28);cx.fill();
    // Legs
    cx.fillStyle='#0077cc';
    cx.fillRect(P.x+5,P.y+31,9,7);
    cx.fillRect(P.x+18,P.y+31,9,7);
    cx.shadowBlur=0;
  }

// Particles
  for(const p of parts){
    cx.globalAlpha=p.life;
    cx.shadowColor=p.col;cx.shadowBlur=6;
    cx.fillStyle=p.col;
    cx.beginPath();cx.arc(p.x,p.y,p.sz*p.life,0,6.28);cx.fill();
    cx.shadowBlur=0;
  }
  cx.globalAlpha=1;
}

function loop(){
  raf=requestAnimationFrame(loop);
  if(gs==='play')update();
  draw();
}

function startGame(){
  score=0;lives=3;lv=1;
  document.getElementById('s0').classList.add('hidden');
  document.getElementById('s1').classList.add('hidden');
  document.getElementById('s2').classList.add('hidden');
  gs='play';init();
}

function nextLv(){
  lv=lv>=3?1:lv+1;
  document.getElementById('s2').classList.add('hidden');
  gs='play';init();
}

loop();
</script>
</body>
</html>
