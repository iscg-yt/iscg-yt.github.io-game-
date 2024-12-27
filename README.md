# iscg-yt.github.io-game-
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
    #score, #highScore {
      position: absolute;
      color: black;
      font-size: 24px;
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
  </style>
</head>
<body>
  <div id="score">Score: 0</div>
  <div id="highScore">High Score: 0</div>
  <canvas id="gameCanvas"></canvas>
  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    document.getElementById("highScore").innerText = "High Score: " + highScore;
    const paddle = {
      width: 100,
      height: 50,
      x: canvas.width / 2 - 50,
      y: canvas.height - 30,
      color: "white",
      speed: 15
    };
    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      radius: 10,
      dx: 5,
      dy: 5,
      color: "red"
    };
    const bounceSound = new Audio("https://www.fesliyanstudios.com/play-mp3/387");
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
      ball.x += ball.dx;
      ball.y += ball.dy;
      // 邊界反彈
      if (ball.x + ball.radius > canvas.width || ball.x - ball.radius < 0) {
        ball.dx *= -1;
        bounceSound.play();
      }
      if (ball.y - ball.radius < 0) {
        ball.dy *= -1;
        bounceSound.play();
      }
      // 碰到底部
      if (ball.y + ball.radius > canvas.height) {
        if (score > highScore) {
          highScore = score;
          localStorage.setItem("highScore", highScore);
          document.getElementById("highScore").innerText = "High Score: " + highScore;
        }
        alert("遊戲結束！最終得分：" + score);
        document.location.reload();
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
        bounceSound.play();
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
    function gameLoop() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawPaddle();
      drawBall();
      updateBall();
      requestAnimationFrame(gameLoop);
    }
    // 初始化
    gameLoop();
    canvas.addEventListener("mousemove", (e) => updatePaddle(e));
    canvas.addEventListener("touchmove", (e) => updatePaddle(e));
  </script>
</body>
</html>
