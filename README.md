// 2D Shooter Game with Levels, Lives, and Sound Effects
const canvas = document.createElement("canvas");
const ctx = canvas.getContext("2d");
document.body.appendChild(canvas);
canvas.width = 800;
canvas.height = 600;

let player = { x: 400, y: 550, width: 40, height: 20, color: "blue", lives: 3 };
let bullets = [];
let enemies = [];
let keys = {};
let score = 0;
let level = 1;

const shootSound = new Audio('https://www.soundjay.com/button/beep-07.wav');
const hitSound = new Audio('https://www.soundjay.com/button/beep-05.wav');
const loseLifeSound = new Audio('https://www.soundjay.com/button/beep-10.wav');

function shootBullet() {
  bullets.push({ x: player.x + player.width / 2 - 2.5, y: player.y, width: 5, height: 10, color: "red" });
  shootSound.currentTime = 0;
  shootSound.play();
}

function spawnEnemies() {
  enemies = [];
  for (let i = 0; i < level * 5; i++) {
    enemies.push({
      x: Math.random() * (canvas.width - 40),
      y: Math.random() * 100,
      width: 40,
      height: 20,
      color: "green",
      speed: 1 + level * 0.2
    });
  }
}

window.addEventListener("keydown", function(e) {
  keys[e.key] = true;
  if (e.key === " ") shootBullet();
});

window.addEventListener("keyup", function(e) {
  keys[e.key] = false;
});

function update() {
  if (keys["ArrowLeft"] && player.x > 0) player.x -= 5;
  if (keys["ArrowRight"] && player.x < canvas.width - player.width) player.x += 5;

  bullets = bullets.filter(b => b.y > 0);
  for (let b of bullets) b.y -= 7;

  for (let e of enemies) e.y += e.speed;

  for (let i = bullets.length - 1; i >= 0; i--) {
    for (let j = enemies.length - 1; j >= 0; j--) {
      const b = bullets[i];
      const e = enemies[j];
      if (b.x < e.x + e.width && b.x + b.width > e.x && b.y < e.y + e.height && b.y + b.height > e.y) {
        bullets.splice(i, 1);
        enemies.splice(j, 1);
        hitSound.currentTime = 0;
        hitSound.play();
        score += 100;
        break;
      }
    }
  }

  for (let i = enemies.length - 1; i >= 0; i--) {
    if (enemies[i].y > canvas.height) {
      enemies.splice(i, 1);
      player.lives--;
      loseLifeSound.currentTime = 0;
      loseLifeSound.play();
      if (player.lives <= 0) {
        alert("Game Over! Score: " + score);
        document.location.reload();
      }
    }
  }

  if (enemies.length === 0) {
    level++;
    spawnEnemies();
  }
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.fillStyle = player.color;
  ctx.fillRect(player.x, player.y, player.width, player.height);

  for (let b of bullets) {
    ctx.fillStyle = b.color;
    ctx.fillRect(b.x, b.y, b.width, b.height);
  }

  for (let e of enemies) {
    ctx.fillStyle = e.color;
    ctx.fillRect(e.x, e.y, e.width, e.height);
  }

  ctx.fillStyle = "black";
  ctx.font = "20px Arial";
  ctx.fillText("Score: " + score, 10, 20);
  ctx.fillText("Lives: " + player.lives, 10, 45);
  ctx.fillText("Level: " + level, 10, 70);
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

spawnEnemies();
loop();
