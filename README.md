<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Space Invaders</title>
<style>
    body {
        margin: 0;
        background: black;
        overflow: hidden;
        color: white;
        font-family: Arial, sans-serif;
        text-align: center;
    }

    canvas {
        display: block;
        margin: 0 auto;
        background: #000;
        border: 2px solid white;
    }

    #ui {
        position: absolute;
        top: 10px;
        left: 10px;
        font-size: 18px;
    }

    button {
        position: absolute;
        top: 10px;
        right: 10px;
        padding: 10px;
        cursor: pointer;
    }
</style>
</head>
<body>

<div id="ui">
    Score: <span id="score">0</span> |
    Lives: <span id="lives">3</span>
</div>

<button onclick="restartGame()">Restart</button>

<canvas id="game" width="800" height="600"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let scoreEl = document.getElementById("score");
let livesEl = document.getElementById("lives");

let player = {
    x: 370,
    y: 550,
    w: 60,
    h: 20,
    speed: 6
};

let bullets = [];
let enemies = [];
let enemyDir = 1;
let enemySpeed = 1;
let gameOver = false;

let score = 0;
let lives = 3;

// Create enemies
function createEnemies() {
    enemies = [];
    for (let r = 0; r < 5; r++) {
        for (let c = 0; c < 10; c++) {
            enemies.push({
                x: 60 + c * 60,
                y: 50 + r * 50,
                w: 40,
                h: 30,
                alive: true
            });
        }
    }
}
createEnemies();

let keys = {};

document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

document.addEventListener("keydown", e => {
    if (e.key === " ") shoot();
});

function shoot() {
    if (gameOver) return;
    bullets.push({
        x: player.x + player.w / 2,
        y: player.y,
        speed: 7
    });
}

function updatePlayer() {
    if (keys["ArrowLeft"] || keys["a"]) player.x -= player.speed;
    if (keys["ArrowRight"] || keys["d"]) player.x += player.speed;

    player.x = Math.max(0, Math.min(canvas.width - player.w, player.x));
}

function updateBullets() {
    bullets.forEach(b => b.y -= b.speed);
    bullets = bullets.filter(b => b.y > 0);
}

function updateEnemies() {
    let hitEdge = false;

    enemies.forEach(e => {
        if (!e.alive) return;
        e.x += enemySpeed * enemyDir;

        if (e.x + e.w > canvas.width || e.x < 0) {
            hitEdge = true;
        }
    });

    if (hitEdge) {
        enemyDir *= -1;
        enemies.forEach(e => e.y += 20);
    }
}

function collision(a, b) {
    return a.x < b.x + b.w &&
           a.x + 5 > b.x &&
           a.y < b.y + b.h &&
           a.y + 10 > b.y;
}

function checkCollisions() {
    bullets.forEach(b => {
        enemies.forEach(e => {
            if (e.alive && collision(b, e)) {
                e.alive = false;
                b.y = -100;
                score += 10;
            }
        });
    });

    enemies.forEach(e => {
        if (e.alive && e.y + e.h >= player.y) {
            lives--;
            e.alive = false;
        }
    });

    if (lives <= 0) gameOver = true;

    let allDead = enemies.every(e => !e.alive);
    if (allDead) {
        createEnemies();
        enemySpeed += 0.5;
    }
}

function drawPlayer() {
    ctx.fillStyle = "lime";
    ctx.fillRect(player.x, player.y, player.w, player.h);
}

function drawBullets() {
    ctx.fillStyle = "red";
    bullets.forEach(b => {
        ctx.fillRect(b.x, b.y, 4, 10);
    });
}

function drawEnemies() {
    ctx.fillStyle = "white";
    enemies.forEach(e => {
        if (e.alive) {
            ctx.fillRect(e.x, e.y, e.w, e.h);
        }
    });
}

function drawText() {
    if (gameOver) {
        ctx.fillStyle = "white";
        ctx.font = "40px Arial";
        ctx.fillText("GAME OVER", 280, 300);
    }
}

function restartGame() {
    score = 0;
    lives = 3;
    gameOver = false;
    enemySpeed = 1;
    bullets = [];
    createEnemies();
}

function loop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (!gameOver) {
        updatePlayer();
        updateBullets();
        updateEnemies();
        checkCollisions();
    }

    drawPlayer();
    drawBullets();
    drawEnemies();
    drawText();

    scoreEl.textContent = score;
    livesEl.textContent = lives;

    requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
