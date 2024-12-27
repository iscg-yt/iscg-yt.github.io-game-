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
      background: linear-gradient(to top, green, white);
    }
    canvas {
      display: block;
      margin: 0 auto;
      border: 5px solid black; /* 黑框描邊 */
    }
    #score, #highScore, #controls {
      position: absolute;
      font-size: 18px;
      color: black;
      font-family: Arial, sans-serif;
      background: white;
      padding: 5px;
      border-radius: 5px;
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
      top: 40px;
      right: 10px;
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
    canvas.width = window.innerWidth - 20;
    canvas.height = window.innerHeight - 20;
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    let isGameOver = false;
    document.getElementById("highScore").innerText = "最高分：" + highScore;
    let soundEnabled = true;
    const wallSound = new Audio("https://www.soundjay.com/button/beep-07.wav");
    const paddleSound = new Audio("https://www.soundjay.com/glass/glass-break-1.wav");
    const paddle = {
      width: 150,
      height: 30,
      x: canvas.width / 2 - 75,
      y: canvas.height - 60,
      color: "black",
      speed: 10
    };
    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      radius: 10,
      dx: 5,
      dy: 5,
      color: "red",
      defaultSpeed: 5
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
      if (isGameOver) return;
      ball.x += ball.dx;
      ball.y += ball.dy;
      // 邊界反彈
      if (ball.x + ball.radius > canvas.width || ball.x - ball.radius < 0) {
        ball.dx *= -1;
        if (soundEnabled) wallSound.play();
      }
      if (ball.y - ball.radius < 0) {
        ball.dy *= -1;
        if (soundEnabled) wallSound.play();
      }
      // 碰到底部
      if (ball.y + ball.radius > canvas.height) {
        endGame();
      }
      // 碰到盤子
      if (
        ball.y + ball.radius >= paddle.y &&
        ball.x > paddle.x &&
        ball.x < paddle.x + paddle.width &&
        ball.dy > 0
      ) {
        ball.dy *= -1;
        ball.y = paddle.y - ball.radius; // 防止卡住
        score++;
        document.getElementById("score").innerText = "分數：" + score;
        if (soundEnabled) paddleSound.play();
        // 每 5 分增加速度
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
      isGameOver = true;
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
      // 重置球的速度
      ball.x = canvas.width / 2;
      ball.y = canvas.height / 2;
      ball.dx = ball.defaultSpeed;
      ball.dy = ball.defaultSpeed;
      document.getElementById("gameOver").style.display = "none";
      gameLoop();
    }
    document.getElementById("toggleSound").addEventListener("click", () => {
      soundEnabled = !soundEnabled;
      document.getElementById("toggleSound").innerText = soundEnabled ? "聲音：開" : "聲音：關";
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
