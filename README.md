<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Jevil Final Fight</title>
  <style>
    body {
      font-family: 'Courier New', monospace;
      background-color: black;
      color: white;
      text-align: center;
      margin: 0;
      padding: 0;
      overflow: hidden;
    }

    .game-container {
      padding: 20px;
    }

    .battlefield {
      position: relative;
      width: 600px;
      height: 400px;
      margin: 0 auto 20px;
      border: 2px solid white;
      background-color: #111;
      overflow: hidden;
    }

    #jevil {
      position: absolute;
      width: 80px;
      transition: top 0.3s, left 0.3s;
    }

    #soul {
      position: absolute;
      width: 20px;
      height: 20px;
      background-color: red;
      border-radius: 50%;
      z-index: 2;
    }

    .bullet {
      position: absolute;
      width: 10px;
      height: 10px;
      background-color: yellow;
      border-radius: 50%;
      z-index: 1;
    }

    button {
      padding: 10px 20px;
      font-size: 18px;
      background-color: purple;
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      margin-top: 10px;
    }

    progress {
      width: 80%;
      height: 20px;
      margin: 5px 0;
    }

    #log {
      margin-top: 15px;
      font-style: italic;
    }
  </style>
</head>
<body>
  <audio id="music" src="https://vgmsite.com/soundtracks/deltarune-ost/lpqlxdedwn/36.%20THE%20WORLD%20REVOLVING.mp3" autoplay loop></audio>

  <div class="game-container">
    <h1>💥 Final Jevil Fight</h1>
    <div class="battlefield" id="battlefield">
      <img src="https://static.wikia.nocookie.net/deltarune/images/f/f6/Jevil_Battle.gif" id="jevil" />
      <div id="soul"></div>
    </div>

    <div>
      <p>Your HP: <span id="player-hp">100</span>/100</p>
      <progress id="player-hp-bar" value="100" max="100"></progress>
    </div>

    <div>
      <p>Jevil HP: <span id="jevil-hp">100</span>/100</p>
      <progress id="jevil-hp-bar" value="100" max="100"></progress>
    </div>

    <button id="attackBtn">Attack</button>
    <div id="log"></div>
  </div>

  <script>
    const battlefield = document.getElementById('battlefield');
    const jevil = document.getElementById('jevil');
    const soul = document.getElementById('soul');
    const attackBtn = document.getElementById('attackBtn');
    const log = document.getElementById('log');

    let soulX = 300, soulY = 200;
    const soulSpeed = 5;
    let keys = {};
    let playerHP = 100;
    let jevilHP = 100;
    let bulletSpeed = 2;

    document.addEventListener('keydown', e => keys[e.key] = true);
    document.addEventListener('keyup', e => keys[e.key] = false);

    function moveSoul() {
      if (keys['ArrowLeft']) soulX -= soulSpeed;
      if (keys['ArrowRight']) soulX += soulSpeed;
      if (keys['ArrowUp']) soulY -= soulSpeed;
      if (keys['ArrowDown']) soulY += soulSpeed;

      soulX = Math.max(0, Math.min(battlefield.clientWidth - 20, soulX));
      soulY = Math.max(0, Math.min(battlefield.clientHeight - 20, soulY));

      soul.style.left = `${soulX}px`;
      soul.style.top = `${soulY}px`;
    }

    function moveJevil() {
      const maxX = battlefield.clientWidth - jevil.clientWidth;
      const maxY = battlefield.clientHeight - jevil.clientHeight;
      jevil.style.left = `${Math.random() * maxX}px`;
      jevil.style.top = `${Math.random() * maxY}px`;
    }

    function spawnBullet() {
      const bullet = document.createElement('div');
      bullet.classList.add('bullet');

      const side = Math.floor(Math.random() * 4);
      let x, y, dx = 0, dy = 0;

      if (side === 0) { x = Math.random() * battlefield.clientWidth; y = 0; dy = bulletSpeed; }
      else if (side === 1) { x = battlefield.clientWidth; y = Math.random() * battlefield.clientHeight; dx = -bulletSpeed; }
      else if (side === 2) { x = Math.random() * battlefield.clientWidth; y = battlefield.clientHeight; dy = -bulletSpeed; }
      else { x = 0; y = Math.random() * battlefield.clientHeight; dx = bulletSpeed; }

      bullet.style.left = `${x}px`;
      bullet.style.top = `${y}px`;
      battlefield.appendChild(bullet);

      const interval = setInterval(() => {
        x += dx;
        y += dy;
        bullet.style.left = `${x}px`;
        bullet.style.top = `${y}px`;

        if (x < -10 || x > battlefield.clientWidth + 10 || y < -10 || y > battlefield.clientHeight + 10) {
          clearInterval(interval);
          bullet.remove();
        }

        const bx = bullet.getBoundingClientRect();
        const sx = soul.getBoundingClientRect();
        if (bx.left < sx.right && bx.right > sx.left && bx.top < sx.bottom && bx.bottom > sx.top) {
          takeDamage(10);
          clearInterval(interval);
          bullet.remove();
        }
      }, 20);
    }

    function takeDamage(amount) {
      if (playerHP <= 0) return;
      playerHP = Math.max(0, playerHP - amount);
      document.getElementById('player-hp-bar').value = playerHP;
      document.getElementById('player-hp').textContent = playerHP;
      log.textContent = `💥 Jevil hit you! (-${amount} HP)`;
      if (playerHP <= 0) {
        log.textContent = "💀 You lost to Jevil. Chaos reigns.";
        clearInterval(gameLoop);
        clearInterval(bulletSpawner);
        attackBtn.disabled = true;
      }
    }

    function dealDamage() {
      if (jevilHP <= 0) return;
      const dmg = Math.floor(Math.random() * 15) + 5;
      jevilHP = Math.max(0, jevilHP - dmg);
      document.getElementById('jevil-hp-bar').value = jevilHP;
      document.getElementById('jevil-hp').textContent = jevilHP;
      log.textContent = `🗡️ You attacked Jevil! (-${dmg} HP)`;

      if (jevilHP <= 0) {
        log.textContent = "✨ You defeated Jevil! Peace returns.";
        clearInterval(gameLoop);
        clearInterval(bulletSpawner);
        attackBtn.disabled = true;
        jevil.style.opacity = 0;
        return;
      }

      // Phases
      if (jevilHP < 70 && bulletSpeed === 2) {
        bulletSpeed = 3;
        log.textContent += " 😈 Jevil is getting serious!";
      } else if (jevilHP < 40 && bulletSpeed === 3) {
        bulletSpeed = 4;
        log.textContent += " 🔥 Final chaos phase!";
      }
    }

    attackBtn.addEventListener('click', dealDamage);

    const gameLoop = setInterval(() => {
      moveSoul();
    }, 20);

    const bulletSpawner = setInterval(() => {
      spawnBullet();
      moveJevil();
    }, 600);
  </script>
</body>
</html>

