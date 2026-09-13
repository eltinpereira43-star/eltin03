      <div id="player"></div>

            <div id="ground"></div>

            <div id="gameOver">
                <h1>GAME OVER</h1>
                <p>Pontuação: <span id="finalScore">0</span></p>
                <button id="restartButton">Jogar novamente</button>
            </div>

        </div>

        <p class="instructions">
            Pressione <strong>ESPAÇO</strong> ou toque na tela para pular
        </p>

    </div>

    <script src="script.js"></script>
</body>
</html>
2. style.css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    background: #111;
    color: white;
    font-family: Arial, sans-serif;

    min-height: 100vh;

    display: flex;
    justify-content: center;
    align-items: center;

    overflow: hidden;
}

.game-container {
    width: 100%;
    max-width: 1000px;
    padding: 20px;
}

.hud {
    display: flex;
    justify-content: space-between;

    font-size: 20px;
    font-weight: bold;

    margin-bottom: 10px;
}

#game {
    position: relative;

    width: 100%;
    height: 400px;

    background: linear-gradient(
        to bottom,
        #65c7ff,
        #dff6ff
    );

    border: 4px solid white;
    border-radius: 15px;

    overflow: hidden;
}

#ground {
    position: absolute;

    bottom: 0;
    left: 0;

    width: 100%;
    height: 50px;

    background: #333;
}

/* PERSONAGEM */

#player {
    position: absolute;

    left: 100px;
    bottom: 50px;

    width: 45px;
    height: 45px;

    background: #ff4757;

    border-radius: 8px;

    z-index: 5;
}

/* OLHOS DO PERSONAGEM */

#player::before {
    content: "";

    position: absolute;

    top: 10px;
    left: 9px;

    width: 7px;
    height: 7px;

    background: white;

    border-radius: 50%;
}

#player::after {
    content: "";

    position: absolute;

    top: 10px;
    right: 9px;

    width: 7px;
    height: 7px;

    background: white;

    border-radius: 50%;
}

/* OBSTÁCULOS */

.obstacle {
    position: absolute;

    bottom: 50px;

    width: 35px;
    height: 55px;

    background: #2f3542;

    border-radius: 5px;

    z-index: 4;
}

/* GAME OVER */

#gameOver {
    position: absolute;

    inset: 0;

    background: rgba(0, 0, 0, 0.75);

    display: none;

    flex-direction: column;
    justify-content: center;
    align-items: center;

    text-align: center;

    z-index: 10;
}

#gameOver h1 {
    font-size: 45px;

    margin-bottom: 10px;
}

#gameOver p {
    font-size: 20px;

    margin-bottom: 20px;
}

button {
    border: none;

    padding: 12px 25px;

    background: #ff4757;
    color: white;

    font-size: 18px;
    font-weight: bold;

    border-radius: 8px;

    cursor: pointer;

    transition: 0.2s;
}

button:hover {
    transform: scale(1.05);
    background: #ff6b81;
}

.instructions {
    text-align: center;

    margin-top: 15px;

    color: #aaa;
}

/* CELULAR */

@media (max-width: 600px) {

    #game {
        height: 300px;
    }

    #player {
        left: 60px;

        width: 40px;
        height: 40px;
    }

    .hud {
        font-size: 16px;
    }

    #gameOver h1 {
        font-size: 35px;
    }

    .instructions {
        font-size: 14px;
    }
}
3. script.js
const game = document.getElementById("game");
const player = document.getElementById("player");

const scoreText = document.getElementById("score");
const highScoreText = document.getElementById("highScore");

const gameOverScreen = document.getElementById("gameOver");
const finalScore = document.getElementById("finalScore");

const restartButton = document.getElementById("restartButton");

let playerY = 0;
let velocityY = 0;

let gravity = 0.8;

let jumping = false;
let gameRunning = true;

let score = 0;
let highScore = localStorage.getItem("jumpHighScore") || 0;

let gameSpeed = 5;

let obstacles = [];

let lastObstacleTime = 0;
let obstacleInterval = 1500;

highScoreText.textContent = highScore;


// ============================
// PULO
// ============================

function jump() {

    if (!gameRunning) return;

    if (!jumping) {

        velocityY = 14;

        jumping = true;
    }
}


// ============================
// CONTROLES
// ============================

document.addEventListener("keydown", function(event) {

    if (
        event.code === "Space" ||
        event.code === "ArrowUp"
    ) {

        event.preventDefault();

        jump();
    }

});


// CONTROLE PARA CELULAR

game.addEventListener("touchstart", function(event) {

    event.preventDefault();

    jump();

});


// ============================
// CRIAR OBSTÁCULO
// ============================

function createObstacle() {

    if (!gameRunning) return;

    const obstacle = document.createElement("div");

    obstacle.classList.add("obstacle");

    const gameWidth = game.offsetWidth;

    obstacle.style.left = gameWidth + "px";

    game.appendChild(obstacle);

    obstacles.push({
        element: obstacle,
        x: gameWidth
    });
}


// ============================
// COLISÃO
// ============================

function checkCollision(playerRect, obstacleRect) {

    return (

        playerRect.left < obstacleRect.right &&

        playerRect.right > obstacleRect.left &&

        playerRect.top < obstacleRect.bottom &&

        playerRect.bottom > obstacleRect.top

    );

}


// ============================
// GAME OVER
// ============================

function gameOver() {

    gameRunning = false;

    finalScore.textContent = score;

    gameOverScreen.style.display = "flex";

    if (score > highScore) {

        highScore = score;

        localStorage.setItem(
            "jumpHighScore",
            highScore
        );

        highScoreText.textContent = highScore;
    }
}


// ============================
// ATUALIZAR JOGADOR
// ============================

function updatePlayer() {

    velocityY -= gravity;

    playerY += velocityY;

    if (playerY <= 0) {

        playerY = 0;

        velocityY = 0;

        jumping = false;
    }

    player.style.bottom =
        (50 + playerY) + "px";
}


// ============================
// ATUALIZAR OBSTÁCULOS
// ============================

function updateObstacles() {

    const playerRect =
        player.getBoundingClientRect();

    for (let i = obstacles.length - 1; i >= 0; i--) {

        const obstacle = obstacles[i];

        obstacle.x -= gameSpeed;

        obstacle.element.style.left =
            obstacle.x + "px";

        const obstacleRect =
            obstacle.element.getBoundingClientRect();


        // COLISÃO

        if (
            checkCollision(
                playerRect,
                obstacleRect
            )
        ) {

            gameOver();

            return;
        }


        // REMOVE OBSTÁCULO

        if (
            obstacle.x <
            -obstacle.element.offsetWidth
        ) {

            obstacle.element.remove();

            obstacles.splice(i, 1);
        }
    }
}


// ============================
// AUMENTAR DIFICULDADE
// ============================

function increaseDifficulty() {

    gameSpeed += 0.002;

    if (obstacleInterval > 700) {

        obstacleInterval -= 0.05;
    }
}


// ============================
// LOOP DO JOGO
// ============================

let lastTime = 0;

function gameLoop(timestamp) {

    if (!gameRunning) return;

    const deltaTime =
        timestamp - lastTime;

    lastTime = timestamp;


    updatePlayer();

    updateObstacles();

    increaseDifficulty();


    // CRIAR OBSTÁCULOS

    if (
        timestamp - lastObstacleTime >
        obstacleInterval
    ) {

        createObstacle();

        lastObstacleTime = timestamp;
    }


    // PONTUAÇÃO

    score += 0.01;

    scoreText.textContent =
        Math.floor(score);


    requestAnimationFrame(gameLoop);
}


// ============================
// REINICIAR
// ============================

function restartGame() {

    obstacles.forEach(obstacle => {

        obstacle.element.remove();

    });

    obstacles = [];

    playerY = 0;

    velocityY = 0;

    jumping = false;

    score = 0;

    gameSpeed = 5;

    obstacleInterval = 1500;

    gameRunning = true;

    gameOverScreen.style.display = "none";

    lastObstacleTime = performance.now();

    requestAnimationFrame(gameLoop);
}


restartButton.addEventListener(
    "click",
    restartGame
);


// ============================
// INICIAR
// ============================

requestAnimationFrame(gameLoop);
