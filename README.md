<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <title>Geometry Dash Demo</title>
  <style>
    body { margin:0; background:#111; display:flex; align-items:center; justify-content:center; height:100vh; }
    canvas { background:#222; border:4px solid #333; }
    #info { position:fixed; top:10px; left:10px; color:#fff; font-family:Arial; }
  </style>
</head>
<body>
  <div id="info">Score: <span id="score">0</span></div>
  <canvas id="game" width="800" height="300"></canvas>
  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');

    const player = { x:50, y:220, w:30, h:30, vy:0, gravity:0.8, jump:-14, grounded:false };
    const obstacles = [];
    let frame = 0, score = 0, speed = 3, gameOver = false;

    function spawnObstacle() {
      const h = 20 + Math.random()*60;
      obstacles.push({ x: canvas.width, y: canvas.height - h - 10, w:20 + Math.random()*30, h: h });
    }

    function reset() {
      obstacles.length = 0; frame = 0; score = 0; speed = 3; gameOver = false;
      player.y = 220; player.vy = 0;
      document.getElementById('score').textContent = score;
    }

    function update() {
      if (gameOver) return;
      frame++;
      // physics
      player.vy += player.gravity;
      player.y += player.vy;
      if (player.y + player.h >= canvas.height - 10) {
        player.y = canvas.height - 10 - player.h;
        player.vy = 0;
        player.grounded = true;
      } else player.grounded = false;

      // spawn
      if (frame % 90 === 0) spawnObstacle();
      // move obstacles
      for (let i = obstacles.length-1; i >= 0; i--) {
        obstacles[i].x -= speed;
        if (obstacles[i].x + obstacles[i].w < 0) { obstacles.splice(i,1); score++; document.getElementById('score').textContent = score; }
      }

      // collision
      for (const ob of obstacles) {
        if (player.x < ob.x + ob.w && player.x + player.w > ob.x &&
            player.y < ob.y + ob.h && player.y + player.h > ob.y) {
          gameOver = true;
        }
      }

      // increase difficulty
      if (frame % 600 === 0) speed += 0.5;
    }

    function draw() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      // ground
      ctx.fillStyle = '#444'; ctx.fillRect(0, canvas.height-10, canvas.width, 10);
      // player
      ctx.fillStyle = '#ffcc00'; ctx.fillRect(player.x, player.y, player.w, player.h);
      // obstacles
      ctx.fillStyle = '#e74c3c';
      for (const ob of obstacles) ctx.fillRect(ob.x, ob.y, ob.w, ob.h);
      if (gameOver) {
        ctx.fillStyle = 'rgba(0,0,0,0.6)'; ctx.fillRect(0,0,canvas.width,canvas.height);
        ctx.fillStyle = '#fff'; ctx.font = '28px Arial'; ctx.fillText('Game Over - Nhấn R để chơi lại', 180, 160);
      }
    }

    function loop() { update(); draw(); requestAnimationFrame(loop); }
    loop();

    // controls
    window.addEventListener('keydown', e => {
      if ((e.code === 'Space' || e.code === 'ArrowUp') && !gameOver) {
        if (player.grounded) player.vy = player.jump;
      }
      if (e.key.toLowerCase() === 'r' && gameOver) reset();
    });
    canvas.addEventListener('click', () => { if (!gameOver && player.grounded) player.vy = player.jump; });

  </script>
</body>
</html>
