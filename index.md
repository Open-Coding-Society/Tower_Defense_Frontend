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

  // Store interval IDs for clearing on restart
  let enemySpawnIntervals = [];

  function startEnemySpawns() {
    // Clear any existing intervals first
    enemySpawnIntervals.forEach(id => clearInterval(id));
    enemySpawnIntervals = [];

    // Giant
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 25,
        imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
        size: 80,
        health: 300
      });
    }, 10000));
    // Hog Rider
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 100,
        imageSrc: 'https://i.postimg.cc/4NxrrzmL/image-2025-05-16-101734632.png',
        size: 60,
        health: 100
      });
    }, 15000));
    // Skeleton Army
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 60,
        imageSrc: 'https://i.postimg.cc/J7Z3cnmp/image-2025-05-16-103314524.png',
        size: 40,
        health: 30
      });
    }, 18000));
    // Baby Dragon
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 50,
        imageSrc: 'https://i.postimg.cc/zfyKLD0S/image-2025-05-16-103451709.png',
        size: 55,
        health: 120
      });
    }, 22000));
    // Minion Horde
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 90,
        imageSrc: 'https://i.postimg.cc/PxZD43GC/image-2025-05-16-103616663.png',
        size: 35,
        health: 25
      });
    }, 20000));
    // Balloon
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 30,
        imageSrc: 'https://i.postimg.cc/28TZ2Lt7/image-2025-05-16-103738864.png',
        size: 65,
        health: 150
      });
    }, 30000));
    // Bandit
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 120,
        imageSrc: 'https://i.postimg.cc/j2cLgBMS/image-2025-05-16-103917837.png',
        size: 50,
        health: 80
      });
    }, 17000));
    // P.E.K.K.A
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 15,
        imageSrc: 'https://i.postimg.cc/FsnKYyq8/image-2025-05-16-104224204.png',
        size: 90,
        health: 500
      });
    }, 40000));
    // Witch
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 35,
        imageSrc: 'https://i.postimg.cc/YqVprpLw/image-2025-05-16-104350767.png',
        size: 60,
        health: 150
      });
    }, 25000));
    // Ram Rider
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 80,
        imageSrc: 'https://i.postimg.cc/YCYwtQw1/image-2025-05-16-104501378.png',
        size: 65,
        health: 180
      });
    }, 28000));
    // Lava Hound
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 20,
        imageSrc: 'https://i.postimg.cc/0QH4YNsk/image-2025-05-16-104632815.png',
        size: 100,
        health: 600
      });
    }, 45000));
    // Royal Ghost
    enemySpawnIntervals.push(setInterval(() => {
      spawnEnemy({
        speed: 70,
        imageSrc: 'https://i.postimg.cc/CxL6QDxh/image-2025-05-16-104742624.png',
        size: 70,
        health: 200
      });
    }, 35000));
  }

  // Start all enemy spawns on game load
  startEnemySpawns();

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
        // Remove all enemies and clear the array
        enemies.forEach(e => {
          e.el.remove();
          if (e.healthBarContainer) e.healthBarContainer.remove();
        });
        enemies.length = 0; // This ensures no invisible enemies remain
        // Reset user health
        userHealth = userMaxHealth;
        updateUserHealthBar();
        // Remove popup
        overlay.remove();
        // Clear all enemy spawn intervals
        enemySpawnIntervals.forEach(id => clearInterval(id));
        enemySpawnIntervals = [];
        // Start enemy spawns again
        startEnemySpawns();
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
        } else if (enemy.size === 40 && enemy.health === 30 && enemy.speed === 60) {
          // Skeleton Army (swarm, low HP)
          userHealth = Math.max(0, userHealth - 100);
        } else if (enemy.size === 55 && enemy.health === 120 && enemy.speed === 50) {
          // Baby Dragon (flying AoE)
          userHealth = Math.max(0, userHealth - 350);
        } else if (enemy.size === 35 && enemy.health === 25 && enemy.speed === 90) {
          // Minion Horde (air swarm)
          userHealth = Math.max(0, userHealth - 120);
        } else if (enemy.size === 65 && enemy.health === 150 && enemy.speed === 30) {
          // Balloon (flying nuke)
          userHealth = Math.max(0, userHealth - 800);
        } else if (enemy.size === 50 && enemy.health === 80 && enemy.speed === 120) {
          // Bandit (dashing unit)
          userHealth = Math.max(0, userHealth - 200);
        } else if (enemy.size === 90 && enemy.health === 500 && enemy.speed === 15) {
          // P.E.K.K.A (boss tank)
          userHealth = Math.max(0, userHealth - 1000);
        } else if (enemy.size === 60 && enemy.health === 150 && enemy.speed === 35) {
          // Witch (spawner)
          userHealth = Math.max(0, userHealth - 250);
        } else if (enemy.size === 65 && enemy.health === 180 && enemy.speed === 80) {
          // Ram Rider (hybrid speed unit)
          userHealth = Math.max(0, userHealth - 400);
        } else if (enemy.size === 100 && enemy.health === 600 && enemy.speed === 20) {
          // Lava Hound (air tank)
          userHealth = Math.max(0, userHealth - 700);
        } else if (enemy.size === 70 && enemy.health === 200 && enemy.speed === 70) {
          // Royal Ghost (stealth)
          userHealth = Math.max(0, userHealth - 300);
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
</script>