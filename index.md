---
layout: post
title: BarrierOps Tower Defense
description: Build and Upgrade your towers to defend your king!
Author: Lars, Darsh, Pradyun
---

<style>
  body {
    margin: 0;
    background-color: #000;
  }

  #gameContainer {
    width: 1000px;
    height: 600px;
    margin: auto;
    background-image: url('https://i.postimg.cc/FzCm3vpj/Screenshot-2025-05-13-at-10-04-00-AM.png');
    background-size: cover;
    background-position: center;
    border: 2px solid white;
    position: relative;
  }

  .path-point {
    width: 10px;
    height: 10px;
    background-color: red;
    position: absolute;
    border-radius: 50%;
    z-index: 10;
  }

  .enemy {
    position: absolute;
    z-index: 5;
    pointer-events: none;
  }
</style>

<div id="userHealthBarContainer" style="width: 400px; height: 28px; margin: 16px auto 8px auto; position: relative; background: rgba(0,0,0,0.7); border-radius: 8px; border: 2px solid #fff; display: flex; align-items: center; justify-content: center;">
  <div id="userHealthBarBg" style="position: absolute; left: 0; top: 0; width: 100%; height: 100%; background: #333; border-radius: 8px;"></div>
  <div id="userHealthBar" style="position: absolute; left: 0; top: 0; height: 100%; background: linear-gradient(90deg, #4caf50, #a5d6a7); border-radius: 8px;"></div>
  <span id="userHealthText" style="position: relative; color: #fff; font-weight: bold; font-size: 18px; z-index: 2;">5000 / 5000</span>
</div>
<div id="gameContainer"></div>

<script>
  const pathPoints = [
    { x: 100, y: 245 },
    { x: 500, y: 245 },
    { x: 500, y: 100 },
    { x: 328, y: 100 },
    { x: 328, y: 475 },
    { x: 145, y: 475 },
    { x: 145, y: 350 },
    { x: 625, y: 350 },
    { x: 625, y: 193 },
    { x: 745, y: 193 },
    { x: 745, y: 442 },
    { x: 433, y: 442 },
    { x: 433, y: 510 },
  ];

  const gameContainer = document.getElementById("gameContainer");

  // 🧪 Debug path points — comment out to hide
  pathPoints.forEach(point => {
    const dot = document.createElement("div");
    dot.className = "path-point";
    dot.style.left = `${point.x}px`;
    dot.style.top = `${point.y}px`;
    gameContainer.appendChild(dot);
  });

  const enemies = [];

  function spawnEnemy({ imageSrc, speed, size, health }) {
    const img = document.createElement("img");
    img.src = imageSrc;
    img.className = "enemy";
    img.style.width = `${size}px`;
    img.style.height = `${size}px`;

    // Create health bar container
    const healthBarContainer = document.createElement("div");
    healthBarContainer.style.position = "absolute";
    healthBarContainer.style.width = `${size}px`;
    healthBarContainer.style.height = "8px";
    healthBarContainer.style.top = "0px";
    healthBarContainer.style.left = "0px";
    healthBarContainer.style.pointerEvents = "none";
    healthBarContainer.style.zIndex = "20";
    // Health bar background
    healthBarContainer.style.background = "rgba(0,0,0,0.5)";
    healthBarContainer.style.borderRadius = "4px";
    // Create health bar fill
    const healthBar = document.createElement("div");
    healthBar.style.height = "100%";
    healthBar.style.width = "100%";
    healthBar.style.background = "linear-gradient(90deg, #4caf50, #a5d6a7)";
    healthBar.style.borderRadius = "4px";
    healthBarContainer.appendChild(healthBar);

    gameContainer.appendChild(img);
    gameContainer.appendChild(healthBarContainer);

    const enemy = {
      el: img,
      healthBarContainer,
      healthBar,
      x: pathPoints[0].x,
      y: pathPoints[0].y,
      speed,
      size,
      health,
      maxHealth: health,
      currentIndex: 0,
      progress: 0,
      segmentDistance: null,
      start: pathPoints[0],
      end: pathPoints[1],
      lastTimestamp: null,
      // For future: method to take damage
      takeDamage(amount) {
        this.health = Math.max(0, this.health - amount);
        this.updateHealthBar();
        if (this.health === 0) {
          this.el.remove();
          this.healthBarContainer.remove();
          enemies.splice(enemies.indexOf(this), 1);
        }
      },
      updateHealthBar() {
        const percent = Math.max(0, this.health / this.maxHealth);
        this.healthBar.style.width = `${percent * 100}%`;
        if (percent > 0.5) {
          this.healthBar.style.background = "linear-gradient(90deg, #4caf50, #a5d6a7)";
        } else if (percent > 0.2) {
          this.healthBar.style.background = "linear-gradient(90deg, #ffc107, #ffe082)";
        } else {
          this.healthBar.style.background = "linear-gradient(90deg, #f44336, #ff8a65)";
        }
      }
    };

    enemy.segmentDistance = Math.hypot(
      enemy.end.x - enemy.start.x,
      enemy.end.y - enemy.start.y
    );

    enemy.updateHealthBar();

    enemies.push(enemy);
    requestAnimationFrame(ts => moveEnemy(enemy, ts));
  }

  // User health bar logic
  let userHealth = 5000;
  const userMaxHealth = 5000;
  const userHealthBar = document.getElementById("userHealthBar");
  const userHealthText = document.getElementById("userHealthText");
  const userHealthBarContainer = document.getElementById("userHealthBarContainer");

  function updateUserHealthBar() {
    const percent = Math.max(0, userHealth / userMaxHealth);
    userHealthBar.style.width = `${percent * 100}%`;
    if (percent > 0.5) {
      userHealthBar.style.background = "linear-gradient(90deg, #4caf50, #a5d6a7)";
    } else if (percent > 0.2) {
      userHealthBar.style.background = "linear-gradient(90deg, #ffc107, #ffe082)";
    } else {
      userHealthBar.style.background = "linear-gradient(90deg, #f44336, #ff8a65)";
    }
    userHealthText.textContent = `${userHealth} / ${userMaxHealth}`;

    // Game over logic
    if (userHealth === 0 && !document.getElementById("gameOverPopup")) {
      showGameOverPopup();
    }
  }

  function showGameOverPopup() {
    // Create overlay
    const overlay = document.createElement("div");
    overlay.id = "gameOverPopup";
    overlay.style.position = "fixed";
    overlay.style.left = "0";
    overlay.style.top = "0";
    overlay.style.width = "100vw";
    overlay.style.height = "100vh";
    overlay.style.background = "rgba(0,0,0,0.85)";
    overlay.style.display = "flex";
    overlay.style.flexDirection = "column";
    overlay.style.alignItems = "center";
    overlay.style.justifyContent = "center";
    overlay.style.zIndex = "9999";

    // Game Over text
    const text = document.createElement("div");
    text.textContent = "Game Over";
    text.style.color = "#fff";
    text.style.fontSize = "48px";
    text.style.fontWeight = "bold";
    text.style.marginBottom = "24px";
    overlay.appendChild(text);

    document.body.appendChild(overlay);

    // After 3 seconds, show restart button
    setTimeout(() => {
      const restartBtn = document.createElement("button");
      restartBtn.textContent = "Start Over";
      restartBtn.style.fontSize = "24px";
      restartBtn.style.padding = "12px 32px";
      restartBtn.style.borderRadius = "8px";
      restartBtn.style.border = "none";
      restartBtn.style.background = "#4caf50";
      restartBtn.style.color = "#fff";
      restartBtn.style.cursor = "pointer";
      restartBtn.style.fontWeight = "bold";
      restartBtn.style.opacity = "0";
      restartBtn.style.transition = "opacity 1s";
      // Ensure button is above overlay and doesn't cover health bar
      restartBtn.style.zIndex = "10000";
      // Fix: force repaint to ensure transition works and preserve background color
      restartBtn.style.background = "#4caf50";
      restartBtn.onclick = () => {
        // Remove all enemies
        enemies.forEach(e => {
          e.el.remove();
          if (e.healthBarContainer) e.healthBarContainer.remove();
        });
        enemies.length = 0;
        // Reset user health
        userHealth = userMaxHealth;
        updateUserHealthBar();
        // Remove popup
        overlay.remove();
        // Spawn a giant instantly again
        spawnEnemy({
          speed: 25,
          imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
          size: 80,
          health: 300
        });
      };
      overlay.appendChild(restartBtn);
      // Fade in the button
      setTimeout(() => {
        // Re-apply background color in case browser resets it on opacity change
        restartBtn.style.background = "#4caf50";
        restartBtn.style.opacity = "1";
      }, 50);
    }, 3000);
  }

  function moveEnemy(enemy, timestamp) {
    if (!enemy.lastTimestamp) enemy.lastTimestamp = timestamp;
    const dt = (timestamp - enemy.lastTimestamp) / 1000;
    enemy.lastTimestamp = timestamp;

    enemy.progress += (enemy.speed * dt) / enemy.segmentDistance;

    if (enemy.progress >= 1) {
      enemy.currentIndex++;
      if (enemy.currentIndex >= pathPoints.length - 1) {
        // Damage user health based on enemy type
        if (enemy.size === 80 && enemy.health === 300 && enemy.speed === 25) {
          // Giant
          userHealth = Math.max(0, userHealth - 300);
        } else if (enemy.size === 60 && enemy.health === 100 && enemy.speed === 100) {
          // Hog Rider
          userHealth = Math.max(0, userHealth - 500);
        }
        updateUserHealthBar();
        enemy.el.remove();
        if (enemy.healthBarContainer) enemy.healthBarContainer.remove(); // Remove health bar
        enemies.splice(enemies.indexOf(enemy), 1);
        return;
      }
      enemy.start = pathPoints[enemy.currentIndex];
      enemy.end = pathPoints[enemy.currentIndex + 1];
      enemy.segmentDistance = Math.hypot(
        enemy.end.x - enemy.start.x,
        enemy.end.y - enemy.start.y
      );
      enemy.progress = 0;
    }

    const x = enemy.start.x + (enemy.end.x - enemy.start.x) * enemy.progress;
    const y = enemy.start.y + (enemy.end.y - enemy.start.y) * enemy.progress;
    enemy.x = x;
    enemy.y = y;

    // Center the enemy image
    enemy.el.style.left = `${x - enemy.size / 2}px`;
    enemy.el.style.top = `${y - enemy.size / 2}px`;

    // Position health bar above the enemy
    enemy.healthBarContainer.style.left = `${x - enemy.size / 2}px`;
    enemy.healthBarContainer.style.top = `${y - enemy.size / 2 - 12}px`;

    requestAnimationFrame(ts => moveEnemy(enemy, ts));
  }

  // 🎯 Spawn a giant instantly on game start
  spawnEnemy({
    speed: 25,
    imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png', // giant
    size: 80,
    health: 300
  });

  // 🎯 Spawn giants every 10 seconds
  setInterval(() => {
    spawnEnemy({
      speed: 25,
      imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png', // giant
      size: 80,
      health: 300
    });
  }, 10000);

  // 🎯 Spawn hog riders every 15 seconds
  setInterval(() => {
    spawnEnemy({
      speed: 100,
      imageSrc: 'https://i.postimg.cc/4NxrrzmL/image-2025-05-16-101734632.png', // HOGGG RIDA
      size: 60,
      health: 100
    });
  }, 15000);
</script>