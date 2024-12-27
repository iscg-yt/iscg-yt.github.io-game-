<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>接球遊戲</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: #f0f0f0;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      height: 100vh;
      padding: 20px;
    }
    canvas {
      display: block;
      background: #282c34;
      margin-top: 30px;
    }
    #score, #highScore {
      position: absolute;
      color: black;
      font-size: 20px;
      font-family: Arial, sans-serif;
    }
    #score {
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
    }
    #highScore {
      top: 40px;
      left: 50%;
      transform: translateX(-50%);
    }
    #controls {
      position: absolute;
      top: 70px;
      left: 50%;
      transform: translateX(-50%);
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
  <div id="score">分數：0</div>
  <div id="highScore">最高分：0</div>
  <div id="controls">
    <button id="toggleSound">開關聲音</button>
  </div>
  <div id="gameOver">
    <p>遊戲結束！</p>
    <p>你的分數：<span id="finalScore">0</span></p>
    <button onclick="restartGame()">重新開始</button>
  </div>
  <canvas id="gameCanvas"></canvas>
  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    canvas.width = window.innerWidth - 40; // 左右各留 20px 邊距
    canvas.height = window.innerHeight - 100; // 調整高度，增加底部空間
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    let isGameOver = false; // 用於判斷遊戲狀態
    document.getElementById("highScore").innerText = "最高分：" + highScore;
    let soundEnabled = true;
    const bounceSound = new Audio("https://www.fesliyanstudios.com/play-mp3/387");
    const paddle = {
      width: 150,
      height: 30, // 增加高度
      x: canvas.width / 2 - 75,
      y: canvas.height - 60, // 提高盤子位置
      color: "white",
      speed: 15
    };
    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      radius: 10,
      dx: 4,
      dy: 4,
      color: "red"
    };
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
        document.getElementById("score").innerText = "分數：" + score;
        if (soundEnabled) bounceSound.play();
        // 每 5 分增加難度
        if (score % 5 === 0) {
          ball.dx *= 1.1;
          ball.dy *= 1.1;
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
        document.getElementById("highScore").innerText = "最高分：" + highScore;
      }
      document.getElementById("finalScore").innerText = score;
      document.getElementById("gameOver").style.display = "block";
    }
    function restartGame() {
      score = 0;
      isGameOver = false;
      document.getElementById("score").innerText = "分數：0";
      ball.x = canvas.width / 2;
      ball.y = canvas.height / 2;
      ball.dx = 5;
      ball.dy = 5;
      document.getElementById("gameOver").style.display = "none";
      gameLoop();
    }
document.getElementById("toggleSound").addEventListener("click", () => {
      soundEnabled = !soundEnabled;
      document.getElementById("toggleSound").innerText = soundEnabled
        ? "聲音：開"
        : "聲音：關";
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
