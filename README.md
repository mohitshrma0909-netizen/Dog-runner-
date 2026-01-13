<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Dog Subway Runner</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: #000;
  }
  canvas {
    display: block;
    background: linear-gradient(#333, #111);
  }
</style>
</head>
<body>

<canvas id="game"></canvas>

<script>
/* ===== CANVAS ===== */
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

/* ===== LANES ===== */
const lanes = [
  canvas.width / 2 - 160,
  canvas.width / 2 - 40,
  canvas.width / 2 + 80
];

/* ===== PLAYER (DOG) ===== */
const dog = {
  lane: 1,
  x: lanes[1],
  y: canvas.height - 180,
  w: 70,
  h: 70,
  dy: 0,
  jump: false,
  slide: false
};

let gravity = 0.7;
let speed = 8;
let score = 0;
let obstacles = [];

/* ===== OBSTACLES ===== */
function spawnObstacle() {
  obstacles.push({
    lane: Math.floor(Math.random() * 3),
    x: canvas.width,
    y: canvas.height - 160,
    w: 60,
    h: 60
  });
}
setInterval(spawnObstacle, 1400);

/* ===== DRAW DOG ===== */
function drawDog() {
  ctx.fillStyle = "orange"; // dog color
  let height = dog.slide ? 35 : dog.h;
  ctx.fillRect(dog.x, dog.y + (dog.slide ? 35 : 0), dog.w, height);
}

/* ===== DRAW OBSTACLE ===== */
function drawObstacle(obs) {
  ctx.fillStyle = "red";
  ctx.fillRect(lanes[obs.lane], obs.y, obs.w, obs.h);
}

/* ===== CONTROLS ===== */
let startX = 0, startY = 0;

document.addEventListener("touchstart", e => {
  startX = e.touches[0].clientX;
  startY = e.touches[0].clientY;
});

document.addEventListener("touchend", e => {
  let dx = e.changedTouches[0].clientX - startX;
  let dy = e.changedTouches[0].clientY - startY;

  if (Math.abs(dx) > Math.abs(dy)) {
    if (dx > 50 && dog.lane < 2) dog.lane++;
    if (dx < -50 && dog.lane > 0) dog.lane--;
  } else {
    if (dy < -50 && !dog.jump) {
      dog.dy = -16;
      dog.jump = true;
    }
    if (dy > 50 && !dog.slide) {
      dog.slide = true;
      setTimeout(() => dog.slide = false, 600);
    }
  }
});

document.addEventListener("keydown", e => {
  if (e.key === "ArrowLeft" && dog.lane > 0) dog.lane--;
  if (e.key === "ArrowRight" && dog.lane < 2) dog.lane++;
  if (e.key === " " && !dog.jump) {
    dog.dy = -16;
    dog.jump = true;
  }
});

/* ===== GAME LOOP ===== */
function update() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Road
  ctx.fillStyle = "#555";
  ctx.fillRect(0, canvas.height - 110, canvas.width, 110);

  // Gravity
  dog.dy += gravity;
  dog.y += dog.dy;

  if (dog.y > canvas.height - 180) {
    dog.y = canvas.height - 180;
    dog.dy = 0;
    dog.jump = false;
  }

  dog.x = lanes[dog.lane];
  drawDog();

  // Obstacles
  for (let i = 0; i < obstacles.length; i++) {
    obstacles[i].x -= speed;
    drawObstacle(obstacles[i]);

    // Collision
    if (
      dog.x < lanes[obstacles[i].lane] + obstacles[i].w &&
      dog.x + dog.w > lanes[obstacles[i].lane] &&
      dog.y < obstacles[i].y + obstacles[i].h &&
      dog.y + dog.h > obstacles[i].y
    ) {
      alert("🐕 GAME OVER\nScore: " + score);
      location.reload();
    }
  }

  // Score
  score++;
  ctx.fillStyle = "white";
  ctx.font = "24px Arial";
  ctx.fillText("Score: " + score, 20, 40);

  requestAnimationFrame(update);
}

update();
</script>

</body>
</html>
