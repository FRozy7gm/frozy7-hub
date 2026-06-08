```html
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<title>FRozy7 HUB</title>

<style>
body {
 margin: 0;
 overflow: hidden;
 font-family: Arial, sans-serif;
 background: radial-gradient(circle, #2a0044, #000);
 color: white;
 user-select: none;
}

/*  ÜST BALIK */
#top {
 position: fixed;
 top: 0;
 width: 100%;
 text-align: center;
 padding: 10px;
 background: rgba(0, 0, 0, 0.7);
 font-size: 22px;
 font-weight: bold;
 z-index: 10;
}

/*  OYUN UI (SKOR, SÜRE, CAN, KALKAN) */
#ui {
 position: fixed;
 top: 50px;
 left: 10px;
 background: rgba(0, 0, 0, 0.6);
 padding: 12px;
 border-radius: 10px;
 z-index: 10;
 font-size: 16px;
 line-height: 1.5;
}

/*  MARKET BUTONU */
#toggleShopBtn {
 position: fixed;
 top: 50px;
 right: 10px;
 background: #00d4ff;
 color: black;
 border: none;
 padding: 10px 15px;
 font-weight: bold;
 border-radius: 8px;
 cursor: pointer;
 z-index: 11;
}

/*  MARKET PANEL */
#shopContainer {
 display: none;
 position: fixed;
 top: 100px;
 right: 10px;
 background: rgba(0, 0, 0, 0.9);
 padding: 15px;
 border-radius: 10px;
 border: 2px solid #00d4ff;
 min-width: 180px;
 z-index: 10;
}

#shopTitle {
 font-weight: bold;
 margin-bottom: 5px;
 text-align: center;
 color: #00d4ff;
}

#shopItems {
 margin-top: 10px;
 display: flex;
 flex-direction: column;
 gap: 8px;
}

.shop-btn {
 background: #00d4ff;
 color: black;
 border: none;
 padding: 8px;
 border-radius: 5px;
 cursor: pointer;
 font-weight: bold;
 text-align: center;
}

/*  HAREKET ETTRLEBLR ALT BAR */
#controlBar {
 position: fixed;
 bottom: 0;
 left: 0;
 width: 100%;
 height: 60px;
 background: rgba(255, 255, 255, 0.15);
 border-top: 2px solid #00d4ff;
 cursor: ew-resize;
 z-index: 10;
}

canvas { display: block; }
</style>
</head>

<body>

<div id="top"> FRozy7</div>

<div id="ui">
  Skor: <span id="gameScore">0</span><br>
  Süre: <span id="gameTime">0</span><br>
  Can: <span id="gameLives"></span><br>
  Kalkan: <span id="gameShield">YOK</span>
</div>

<button id="toggleShopBtn" onclick="toggleShop()"> Market Aç</button>

<div id="shopContainer">
 <div id="shopTitle">Market Skor: <span id="scoreText">0</span></div>
 <div id="shopItems">
  <button class="shop-btn" onclick="buyShield()"> Kalkan (30)</button>
  <button class="shop-btn" onclick="buyHeart()"> Can Yenile (40)</button>
  <button class="shop-btn" onclick="buyBoost()"> Skor x2 (50)</button>
 </div>
</div>

<div id="controlBar"></div>

<canvas id="game"></canvas>

<script>
const c = document.getElementById("game");
const ctx = c.getContext("2d");
const controlBar = document.getElementById("controlBar");

c.width = innerWidth;
c.height = innerHeight;

let score = 0, time = 0, lives = 3;
let isPaused = false; 
let isShopOpen = false;
let scoreMultiplier = 1;
let hasShield = false;
let gameOver = false;

// Artk çubuk yerine  karakteri var (Genilik ve yükseklik emoji boyutuna göre ayarland)
let paddle = { x: c.width / 2 - 25, y: c.height - 120, w: 50, h: 50 };
let objs = [];

const names = [
 { n: "EREF", c: "#fff" },
 { n: "MEHMET", c: "#0ff" },
 { n: "ANIMUS", c: "#f0f" },
 { n: "GÖKHAN", c: "#ff0" },
 { n: "BELNAY", c: "pink" },
 { n: "SELAHATTN", c: "#0f0" }
];

function spawn() {
 if (isPaused || isShopOpen || gameOver) return;
 
 let n;
 // FRozy7 yine nadir geliyor (%10 ans)
 if(Math.random() < 0.1) {
  n = { n: "FRozy7", c: "#00d4ff" };
  triggerFrozyEffect(); 
 } else {
  n = names[Math.floor(Math.random() * names.length)];
 }

 objs.push({
  x: Math.random() * (c.width - 70),
  y: -40,
  w: 70, // Yazlarn smas için genilik
  h: 40,
  sp: 3, // Hz biraz artrld, kaçmak daha elenceli olsun diye
  ...n
 });
}

setInterval(spawn, 800);
setInterval(() => {
 if (!isPaused && !isShopOpen && !gameOver) {
  time++;
  score += 1 * scoreMultiplier; // Hayatta kaldkça puan kazanrsn
 }
}, 1000);

// --- HAREKET SSTEM ---
let dragPaddle = false;

function movePaddle(clientX) {
 if (!isPaused && !isShopOpen && !gameOver) {
  paddle.x = clientX - paddle.w / 2;
  if (paddle.x < 0) paddle.x = 0;
  if (paddle.x + paddle.w > c.width) paddle.x = c.width - paddle.w;
 }
}

c.onmousedown = () => dragPaddle = true;
c.onmouseup = () => dragPaddle = false;
c.onmousemove = e => { if (dragPaddle) movePaddle(e.clientX); };

controlBar.onmousedown = () => dragPaddle = true;
window.onmouseup = () => dragPaddle = false;
controlBar.onmousemove = e => { if (dragPaddle) movePaddle(e.clientX); };

// Mobil Dokunmatik Kontroller
c.ontouchstart = () => dragPaddle = true;
c.ontouchend = () => dragPaddle = false;
c.ontouchmove = e => { if (dragPaddle) movePaddle(e.touches[0].clientX); };

controlBar.ontouchstart = () => dragPaddle = true;
controlBar.ontouchend = () => dragPaddle = false;
controlBar.ontouchmove = e => { if (dragPaddle) movePaddle(e.touches[0].clientX); };

// --- MARKET SSTEM ---
function toggleShop() {
 if (gameOver) return;
 const shop = document.getElementById("shopContainer");
 const btn = document.getElementById("toggleShopBtn");
 
 isShopOpen = !isShopOpen;
 
 if (isShopOpen) {
  shop.style.display = "block";
  btn.innerText = " Kapat";
 } else {
  shop.style.display = "none";
  btn.innerText = " Market Aç";
 }
}

function buyShield() {
 if (score >= 30 && !hasShield) {
  score -= 30;
  hasShield = true;
 }
}

function buyHeart() {
 if (score >= 40 && lives < 3) {
  score -= 40;
  lives++;
 }
}

function buyBoost() {
 if (score >= 50) {
  score -= 50;
  scoreMultiplier = 2;
  setTimeout(() => { scoreMultiplier = 1; }, 10000);
 }
}

function triggerFrozyEffect() {
 isPaused = true; 
 if (navigator.vibrate) { navigator.vibrate([200, 100, 200]); }
 setTimeout(() => { isPaused = false; }, 1200);
}

// --- ANA OYUN DÖNGÜSÜ ---
function loop() {
 ctx.clearRect(0, 0, c.width, c.height);

 //  MOAI KARAKTERN ÇZME
 ctx.font = "45px Arial";
 ctx.textAlign = "center";
 ctx.fillText("", paddle.x + paddle.w / 2, paddle.y + paddle.h - 5);

 // Eer aktifse kalkan efekti çiz
 if (hasShield) {
  ctx.strokeStyle = "#00d4ff";
  ctx.lineWidth = 4;
  ctx.beginPath();
  ctx.arc(paddle.x + paddle.w / 2, paddle.y + paddle.h / 2, 40, 0, Math.PI * 2);
  ctx.stroke();
 }

 // Objeleri güncelle ve kontrol et
 if (!isPaused && !isShopOpen && !gameOver) {
  for (let i = 0; i < objs.length; i++) {
   let o = objs[i];
   o.y += o.sp;

   //  SMLERE DEERSEK (ÇARPIMA)
   if (
    o.y + o.h >= paddle.y &&
    o.y <= paddle.y + paddle.h &&
    o.x < paddle.x + paddle.w &&
    o.x + o.w > paddle.x
   ) {
    // Kalkan varsa can gitmez, kalkan krlr
    if (hasShield) {
     hasShield = false;
    } else {
     lives--;
     if (lives <= 0) {
      gameOver = true;
     }
    }
    objs.splice(i, 1);
    i--;
    continue;
   }

   // Ekrandan çknca temizle (Çarpmadan kaçarsan puan gelir)
   if (o.y > c.height) {
    score += 5 * scoreMultiplier;
    objs.splice(i, 1);
    i--;
   }
  }
 }

 // sim kutularn çiz
 for (let i = 0; i < objs.length; i++) {
  let o = objs[i];
  ctx.fillStyle = o.c;
  ctx.fillRect(o.x, o.y, o.w, o.h);

  ctx.fillStyle = "#000";
  ctx.font = "bold 13px Arial";
  ctx.textAlign = "center";
  ctx.fillText(o.n, o.x + o.w / 2, o.y + o.h / 2 + 5);
 }

 // Arayüzü Güncelleme
 document.getElementById("gameScore").innerText = score;
 document.getElementById("scoreText").innerText = score;
 document.getElementById("gameTime").innerText = time;
 document.getElementById("gameShield").innerText = hasShield ? "AKTF " : "YOK";
 
 // Canlar kalbe dökme
 let heartStr = "";
 for(let l=0; l<lives; l++) heartStr += "";
 document.getElementById("gameLives").innerText = heartStr || "ÖLDÜN";

 // Oyun Bitti Ekran
 if (gameOver) {
  ctx.fillStyle = "rgba(0,0,0,0.8)";
  ctx.fillRect(0, 0, c.width, c.height);
  ctx.fillStyle = "red";
  ctx.font = "bold 40px Arial";
  ctx.textAlign = "center";
  ctx.fillText("OYUN BTT!", c.width / 2, c.height / 2 - 20);
  ctx.fillStyle = "white";
  ctx.font = "20px Arial";
  ctx.fillText("Yeniden balamak için sayfay yenile.", c.width / 2, c.height / 2 + 20);
  return;
 }

 requestAnimationFrame(loop);
}

loop();

window.onresize = () => {
 c.width = innerWidth;
 c.height = innerHeight;
};
</script>

</body>
</html>

```
