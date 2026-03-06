<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Snake</title>
  <style>
    :root { color-scheme: light dark; }
    body {
      margin: 0; display: grid; place-items: center; height: 100dvh;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
      background: #111; color: #eee;
    }
    .wrap { display: grid; gap: 12px; place-items: center; }
    canvas { background: #1b1b1b; border: 2px solid #333; image-rendering: pixelated; }
    .hud { display: flex; gap: 16px; align-items: center; font-size: 14px; color: #ccc; }
    .btn {
      padding: 6px 10px; border: 1px solid #444; border-radius: 6px;
      background: #222; color: #ddd; cursor: pointer; font-size: 14px;
    }
    .btn:hover { background: #2a2a2a; }
    .note { font-size: 12px; color: #aaa; }
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="game" width="400" height="400" aria-label="Snake game"></canvas>
    <div class="hud">
      <div>Score: <span id="score">0</span></div>
      <div>Best: <span id="best">0</span></div>
      <button class="btn" id="restart">Restart (R)</button>
      <button class="btn" id="pause">Pause (Space)</button>
    </div>
    <div class="note">Use arrow keys or WASD. Snake wraps around edges. Don’t hit yourself!</div>
  </div>

  <script>
    // Config
    const tileSize = 20;        // px per grid cell
    const tiles = 20;           // grid is 20x20 (400px canvas)
    const speedMs = 120;        // lower is faster

    // Canvas
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');

    // HUD
    const scoreEl = document.getElementById('score');
    const bestEl = document.getElementById('best');
    const pauseBtn = document.getElementById('pause');
    const restartBtn = document.getElementById('restart');

    // State
    let snake, dir, nextDir, food, score, best, timer, paused;

    function init() {
      snake = [{ x: 10, y: 10 }];
      dir = { x: 1, y: 0 };
      nextDir = { x: 1, y: 0 };
      score = 0;
      paused = false;
      placeFood();
      updateHUD();
      if (timer) clearInterval(timer);
      timer = setInterval(tick, speedMs);
      draw();
    }

    function updateHUD() {
      scoreEl.textContent = score;
      best = Math.max(score, Number(localStorage.getItem('snake_best') || 0));
      bestEl.textContent = best;
    }

    function placeFood() {
      while (true) {
        const f = { x: Math.floor(Math.random() * tiles), y: Math.floor(Math.random() * tiles) };
        if (!snake.some(s => s.x === f.x && s.y === f.y)) {
          food = f; return;
        }
      }
    }

    function tick() {
      if (paused) return;
      // Apply next direction but prevent 180° reversal
      if (!(nextDir.x === -dir.x && nextDir.y === -dir.y)) {
        dir = nextDir;
      }
      // New head with wrap-around
      const head = snake[0];
      const nx = (head.x + dir.x + tiles) % tiles;
      const ny = (head.y + dir.y + tiles) % tiles;
      const newHead = { x: nx, y: ny };

      // Self-collision
      if (snake.some(s => s.x === newHead.x && s.y === newHead.y)) {
        gameOver();
        return;
      }

      snake.unshift(newHead);

      // Eat
      if (newHead.x === food.x && newHead.y === food.y) {
        score++;
        localStorage.setItem('snake_best', String(Math.max(score, Number(localStorage.getItem('snake_best') || 0))));
        updateHUD();
        placeFood();
      } else {
        snake.pop();
      }

      draw();
    }

    function draw() {
      // Background
      ctx.fillStyle = '#121212';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Food
      ctx.fillStyle = '#e74c3c';
      drawCell(food.x, food.y);

      // Snake
      snake.forEach((seg, i) => {
        ctx.fillStyle = i === 0 ? '#2ecc71' : '#27ae60';
        drawCell(seg.x, seg.y);
      });
    }

    function drawCell(x, y) {
      ctx.fillRect(x * tileSize, y * tileSize, tileSize - 1, tileSize - 1);
    }

    function gameOver() {
      clearInterval(timer);
      timer = null;
      // Simple overlay
      ctx.fillStyle = 'rgba(0,0,0,0.6)';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 24px system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText('Game Over', canvas.width / 2, canvas.height / 2 - 10);
      ctx.font = '16px system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif';
      ctx.fillText(`Score: ${score}   Best: ${best}`, canvas.width / 2, canvas.height / 2 + 16);
      ctx.fillText('Press R to restart', canvas.width / 2, canvas.height / 2 + 38);
    }

    // Controls
    window.addEventListener('keydown', (e) => {
      const k = e.key.toLowerCase();
      if (['arrowup','w'].includes(k))      nextDir = { x: 0, y: -1 };
      else if (['arrowdown','s'].includes(k)) nextDir = { x: 0, y: 1 };
      else if (['arrowleft','a'].includes(k)) nextDir = { x: -1, y: 0 };
      else if (['arrowright','d'].includes(k)) nextDir = { x: 1, y: 0 };
      else if (k === ' ') togglePause();
      else if (k === 'r') init();
    });

    function togglePause() {
      paused = !paused;
      pauseBtn.textContent = paused ? 'Resume (Space)' : 'Pause (Space)';
    }

    pauseBtn.addEventListener('click', togglePause);
    restartBtn.addEventListener('click', init);

    // Start
    init();
  </script>
</body>
</html>
