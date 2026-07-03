const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const scoreEl = document.getElementById("score");
const highEl = document.getElementById("high");

const overlay = document.getElementById("overlay");
const overlayTitle = document.getElementById("overlayTitle");
const overlayText = document.getElementById("overlayText");
const startBtn = document.getElementById("startBtn");

const W = canvas.width;
const H = canvas.height;

const keys = new Set();

const state = {
  running: false,
  lastTime: 0,
  score: 0,
  high: Number(localStorage.getItem("ad_high") || 0),

  // difficulty scaling
  spawnTimer: 0,
  spawnEvery: 0.65, // seconds
  baseSpeed: 220,   // px/s
};

highEl.textContent = state.high;

const player = {
  w: 46,
  h: 18,
  x: W / 2,
  y: H - 70,
  speed: 380, // px/s
};

let asteroids = [];

function resetGame() {
  state.score = 0;
  state.spawnTimer = 0;
  state.spawnEvery = 0.65;
  state.baseSpeed = 220;

  player.x = W / 2;
  player.y = H - 70;

  asteroids = [];
  scoreEl.textContent = "0";
}

function startGame() {
  resetGame();
  state.running = true;
  state.lastTime = performance.now();
  hideOverlay();
  requestAnimationFrame(loop);
}

function gameOver() {
  state.running = false;

  if (state.score > state.high) {
    state.high = state.score;
    localStorage.setItem("ad_high", String(state.high));
    highEl.textContent = String(state.high);
  }

  overlayTitle.textContent = "Game Over";
  overlayText.textContent = `Score: ${state.score}. Press Space to try again.`;
  startBtn.textContent = "Restart";
  showOverlay();
}

function showOverlay() { overlay.classList.remove("hidden"); }
function hideOverlay() { overlay.classList.add("hidden"); }

function clamp(v, min, max) { return Math.max(min, Math.min(max, v)); }

function rectsOverlap(a, b) {
  return (
    a.x - a.w / 2 < b.x + b.w / 2 &&
    a.x + a.w / 2 > b.x - b.w / 2 &&
    a.y - a.h / 2 < b.y + b.h / 2 &&
    a.y + a.h / 2 > b.y - b.h / 2
  );
}

function spawnAsteroid() {
  const size = 18 + Math.random() * 34; // 18..52
  const x = size / 2 + Math.random() * (W - size);
  const speed = state.baseSpeed + Math.random() * 220;

  asteroids.push({
    x,
    y: -size,
    w: size,
    h: size,
    vy: speed,
    rot: Math.random() * Math.PI * 2,
    vr: (Math.random() * 2 - 1) * 2.2,
  });
}

function update(dt) {
  // input
  const left = keys.has("ArrowLeft") || keys.has("a") || keys.has("A");
  const right = keys.has("ArrowRight") || keys.has("d") || keys.has("D");

  if (left) player.x -= player.speed * dt;
  if (right) player.x += player.speed * dt;
  player.x = clamp(player.x, player.w / 2, W - player.w / 2);

  // spawn
  state.spawnTimer += dt;
  if (state.spawnTimer >= state.spawnEvery) {
    state.spawnTimer = 0;
    spawnAsteroid();

    // slowly increase difficulty
    state.spawnEvery = Math.max(0.28, state.spawnEvery * 0.992);
    state.baseSpeed = Math.min(620, state.baseSpeed + 2.5);
  }

  // move asteroids
  for (const a of asteroids) {
    a.y += a.vy * dt;
    a.rot += a.vr * dt;
  }

  // remove off-screen
  asteroids = asteroids.filter(a => a.y < H + a.h);

  // scoring: survive time (60 points/sec)
  state.score += Math.floor(dt * 60);
  scoreEl.textContent = String(state.score);

  // collisions
  const p = { x: player.x, y: player.y, w: player.w, h: player.h };
  for (const a of asteroids) {
    if (rectsOverlap(p, a)) {
      gameOver();
      break;
    }
  }
}

function drawStarfield() {
  // simple faux starfield (deterministic based on score)
  ctx.save();
  ctx.fillStyle = "rgba(255,255,255,0.7)";
  const t = state.score * 0.002;
  for (let i = 0; i < 80; i++) {
    const x = (i * 67.3) % W;
    const y = ((i * 131.7) + (t * 120)) % H;
    const r = (i % 3 === 0) ? 1.4 : 1.0;
    ctx.globalAlpha = (i % 7 === 0) ? 0.8 : 0.35;
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI * 2);
    ctx.fill();
  }
  ctx.restore();
}

function draw() {
  // clear
  ctx.clearRect(0, 0, W, H);

  // background
  const g = ctx.createLinearGradient(0, 0, 0, H);
  g.addColorStop(0, "rgba(30, 50, 120, 0.35)");
  g.addColorStop(1, "rgba(10, 12, 24, 0.35)");
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, W, H);

  drawStarfield();

  // player
  ctx.save();
  ctx.translate(player.x, player.y);
  ctx.fillStyle = "rgba(122, 162, 255, 0.95)";
  ctx.strokeStyle = "rgba(255,255,255,0.35)";
  ctx.lineWidth = 2;

  // small ship shape
  ctx.beginPath();
  ctx.moveTo(0, -18);
  ctx.lineTo(22, 14);
  ctx.lineTo(10, 10);
  ctx.lineTo(-10, 10);
  ctx.lineTo(-22, 14);
  ctx.closePath();
  ctx.fill();
  ctx.stroke();
  ctx.restore();

  // asteroids
  for (const a of asteroids) {
    ctx.save();
    ctx.translate(a.x, a.y);
    ctx.rotate(a.rot);

    ctx.fillStyle = "rgba(255, 210, 140, 0.92)";
    ctx.strokeStyle = "rgba(0,0,0,0.25)";
    ctx.lineWidth = 3;

    // jagged-ish rock
    const r = a.w / 2;
    ctx.beginPath();
    for (let i = 0; i < 10; i++) {
      const ang = (i / 10) * Math.PI * 2;
      const jitter = 0.72 + Math.random() * 0.35; // tiny randomness per frame is ok for "rocky" look
      const rr = r * jitter;
      const px = Math.cos(ang) * rr;
      const py = Math.sin(ang) * rr;
      if (i === 0) ctx.moveTo(px, py);
      else ctx.lineTo(px, py);
    }
    ctx.closePath();
    ctx.fill();
    ctx.stroke();

    ctx.restore();
  }

  // bottom vignette
  const vg = ctx.createLinearGradient(0, H - 180, 0, H);
  vg.addColorStop(0, "rgba(0,0,0,0)");
  vg.addColorStop(1, "rgba(0,0,0,0.35)");
  ctx.fillStyle = vg;
  ctx.fillRect(0, H - 180, W, 180);
}

function loop(now) {
  if (!state.running) return;

  const dt = Math.min(0.033, (now - state.lastTime) / 1000);
  state.lastTime = now;

  update(dt);
  draw();

  if (state.running) requestAnimationFrame(loop);
}

// input handlers
window.addEventListener("keydown", (e) => {
  keys.add(e.key);

  if (e.key === " " || e.key === "Spacebar") {
    e.preventDefault();
    if (!state.running) startGame();
  }
});

window.addEventListener("keyup", (e) => {
  keys.delete(e.key);
});

startBtn.addEventListener("click", () => {
  if (!state.running) startGame();
});

// initial overlay
overlayTitle.textContent = "Asteroid Dodger";
overlayText.textContent = "Avoid the asteroids. Press Space to start.";
startBtn.textContent = "Start";
showOverlay();
