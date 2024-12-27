<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>接球遊戲</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: #f0f0f0;
    }
    canvas {
      display: block;
      background: #282c34;
    }
    #score, #highScore, #controls {
      position: absolute;
      color: black;
      font-size: 20px;
      font-family: Arial, sans-serif;
    }
    #score {
      top: 10px;
      right: 10px;
    }
    #highScore {
      top: 10px;
      left: 10px;
    }
    #controls {
      top: 50px;
      left: 10px;
    }
    #gameOver {
      display: none;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: black;
      text-align: center;
      font-size: 24px;
      font-family: Arial, sans-serif;
    }
    #gameOver button {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 18px;
    }
  </style>
</head>
<body>
  <div id="score">Score: 0</div>
  <div id="highScore">High Score: 0</div>
  <div id="controls">
    <button id="toggleSound">Toggle Sound</button>
  </div>
  <div id="gameOver">
    <p>Game Over!</p>
    <p>Your Score: <span id="finalScore">0</span></p>
    <button onclick="restartGame()">Restart</button>
  </div>
  <canvas id="gameCanvas"></canvas>
  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    let isGameOver = false; // 用於判斷遊戲狀態
    document.getElementById("highScore").innerText = "High Score: " + highScore;
    let soundEnabled = true;
    const bounceSound = new Audio("https://www.fesliyanstudios.com/play-mp3/387")
    const paddle = {
      width: 150,
      height: 30, // 增加高度
      x: canvas.width / 2 - 75,
      y: canvas.height - 60, // 提高盤子位置
      color: "white",
      speed: 15
    };
    const initialBallSpeed = 5; // 初始速度值
    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      radius: 10,
      dx: initialBallSpeed,
      dy: initialBallSpeed,
      color: "red"
    };
    function resetBall() {
      ball.x = canvas.width / 2;
      ball.y = canvas.height / 2;
      ball.dx = initialBallSpeed;
      ball.dy = initialBallSpeed;
    }
    function drawPaddle() {
      ctx.fillStyle = paddle.color;
      ctx.fillRect(paddle.x, paddle.y, paddle.width, paddle.height);
    }
    function drawBall() {
      ctx.beginPath();
      ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
      ctx.fillStyle = ball.color;
      ctx.fill();
      ctx.closePath();
    }
    function updateBall() {
      if (isGameOver) return; // 如果遊戲結束，不再更新球的位置
      ball.x += ball.dx;
      ball.y += ball.dy;
      // 邊界反彈
      if (ball.x + ball.radius > canvas.width || ball.x - ball.radius < 0) {
        ball.dx *= -1;
        if (soundEnabled) bounceSound.play();
      }
      if (ball.y - ball.radius < 0) {
        ball.dy *= -1;
        if (soundEnabled) bounceSound.play();
      }
      // 碰到底部
      if (ball.y + ball.radius > canvas.height) {
        endGame();
      }
      // 碰到盤子
      if (
        ball.y + ball.radius > paddle.y &&
        ball.x > paddle.x &&
        ball.x < paddle.x + paddle.width
      ) {
        ball.dy *= -1;
        score++;
        document.getElementById("score").innerText = "Score: " + score;
        if (soundEnabled) bounceSound.play();
        // 每 5 分增加難度
        if (score % 5 === 0) {
          ball.dx *= 1.2;
          ball.dy *= 1.2;
        }
      }
    }
    function updatePaddle(e) {
      const touchX = e.touches ? e.touches[0].clientX : e.clientX;
      paddle.x = Math.max(0, Math.min(touchX - paddle.width / 2, canvas.width - paddle.width));
    }
    function endGame() {
      isGameOver = true; // 設定遊戲結束狀態
      if (score > highScore) {
        highScore = score;
        localStorage.setItem("highScore", highScore);
        document.getElementById("highScore").innerText = "High Score: " + highScore;
      }
      document.getElementById("finalScore").innerText = score;
      document.getElementById("gameOver").style.display = "block";
    }
    function restartGame() {
      score = 0;
      isGameOver = false;
      document.getElementById("score").innerText = "Score: 0";
      resetBall(); // 重置球的位置與速度
  document.getElementById("gameOver").style.display = "none";
      gameLoop();
    }
document.getElementById("toggleSound").addEventListener("click", () => {
      soundEnabled = !soundEnabled;
      document.getElementById("toggleSound").innerText = soundEnabled
        ? "Sound: ON"
        : "Sound: OFF";
    });
    function gameLoop() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawPaddle();
      drawBall();
      updateBall();
      requestAnimationFrame(gameLoop);
    }
    canvas.addEventListener("mousemove", (e) => updatePaddle(e));
    canvas.addEventListener("touchmove", (e) => updatePaddle(e));
    gameLoop();
  </script>
</body>
</html>
