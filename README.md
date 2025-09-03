<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Bomb Dodge - Single Player</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: linear-gradient(to bottom, #0a0a2a, #000000);
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      font-family: Arial, sans-serif;
      color: white;
    }

    canvas {
      background: transparent;
      display: block;
    }

    #scoreBoard {
      position: absolute;
      top: 15px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(255, 255, 255, 0.1);
      backdrop-filter: blur(8px);
      padding: 8px 20px;
      border-radius: 20px;
      font-size: 18px;
      display: flex;
      gap: 20px;
    }

    #gameOver {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
      display: none;
    }

    #gameOver h1 {
      font-size: 40px;
      color: #ff4444;
      text-shadow: 0 0 20px #ff0000;
    }

    #restartBtn {
      margin-top: 15px;
      padding: 10px 20px;
      font-size: 20px;
      border: none;
      border-radius: 12px;
      background: #00bcd4;
      color: white;
      cursor: pointer;
      box-shadow: 0 0 10px #00bcd4;
    }

    #restartBtn:hover {
      background: #0097a7;
    }
  </style>
</head>
<body>
  <div id="scoreBoard">Score: <span id="score">0</span></div>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <div id="gameOver">
    <h1>Game Over</h1>
    <button id="restartBtn">Restart</button>
  </div>

  <audio id="bgMusic" loop>
    <source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-10.mp3" type="audio/mpeg">
  </audio>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreDisplay = document.getElementById("score");
    const gameOverScreen = document.getElementById("gameOver");
    const restartBtn = document.getElementById("restartBtn");
    const music = document.getElementById("bgMusic");

    let player = { x: canvas.width / 2, y: canvas.height - 60, r: 20 };
    let bombs = [];
    let score = 0;
    let gameOver = false;
    let bombSpeed = 2;

    // 🎶 Start music when game starts
    function startMusic() {
      music.volume = 0.5;
      music.play().catch(() => {}); // Ignore autoplay block
    }

    // Reset game
    function resetGame() {
      bombs = [];
      score = 0;
      gameOver = false;
      bombSpeed = 2;
      player.x = canvas.width / 2;
      gameOverScreen.style.display = "none";
      startMusic();
      requestAnimationFrame(update);
    }

    // Create bombs
    function createBomb() {
      bombs.push({
        x: Math.random() * (canvas.width - 20) + 10,
        y: -20,
        r: 15
      });
    }

    // Update game
    function update() {
      if (gameOver) return;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Draw player
      ctx.beginPath();
      ctx.arc(player.x, player.y, player.r, 0, Math.PI * 2);
      ctx.fillStyle = "orange";
      ctx.shadowColor = "orange";
      ctx.shadowBlur = 20;
      ctx.fill();
      ctx.closePath();
      ctx.shadowBlur = 0;

      // Move & draw bombs
      bombs.forEach((bomb, i) => {
        bomb.y += bombSpeed;

        ctx.beginPath();
        ctx.arc(bomb.x, bomb.y, bomb.r, 0, Math.PI * 2);
        ctx.fillStyle = "black";
        ctx.fill();
        ctx.strokeStyle = "red";
        ctx.stroke();
        ctx.closePath();

        // Collision check
        let dx = player.x - bomb.x;
        let dy = player.y - bomb.y;
        let dist = Math.sqrt(dx * dx + dy * dy);

        if (dist < player.r + bomb.r) {
          gameOver = true;
          gameOverScreen.style.display = "block";
          music.pause();
        }

        // Remove bombs off screen & increase score
        if (bomb.y > canvas.height + bomb.r) {
          bombs.splice(i, 1);
          score++;
          scoreDisplay.textContent = score;

          if (score % 5 === 0) {
            bombSpeed += 0.5; // speed up with score
          }
        }
      });

      // Add bombs
      if (Math.random() < 0.02) {
        createBomb();
      }

      requestAnimationFrame(update);
    }

    // Drag to move player
    let dragging = false;
    canvas.addEventListener("touchstart", (e) => {
      let touch = e.touches[0];
      let dx = touch.clientX - canvas.getBoundingClientRect().left - player.x;
      let dy = touch.clientY - canvas.getBoundingClientRect().top - player.y;
      if (Math.sqrt(dx * dx + dy * dy) < player.r) {
        dragging = true;
      }
    });

    canvas.addEventListener("touchmove", (e) => {
      if (dragging) {
        let touch = e.touches[0];
        let rect = canvas.getBoundingClientRect();
        player.x = touch.clientX - rect.left;

        // Keep inside screen
        if (player.x < player.r) player.x = player.r;
        if (player.x > canvas.width - player.r) player.x = canvas.width - player.r;
      }
    });

    canvas.addEventListener("touchend", () => {
      dragging = false;
    });

    restartBtn.addEventListener("click", resetGame);

    resetGame();
  </script>
</body>
</html>
