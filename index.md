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

  .tower {
    position: absolute;
    z-index: 15;
  }

  .tower-radius {
    position: absolute;
    z-index: 10;
    pointer-events: none;
  }
</style>

<!-- Coin Display -->
<div id="coinDisplay" style="min-width: 120px; height: 48px; background: rgba(255,215,0,0.15); border: 2px solid #ffd700; border-radius: 24px; display: flex; align-items: center; justify-content: center; font-size: 28px; font-weight: bold; color: #ffd700; z-index: 10002; box-shadow: 0 2px 8px #0008; pointer-events: none;">
  <span id="coinAmount" style="min-width: 60px; text-align: center; display: inline-block; flex: 1 1 auto;">0</span>
</div>

<div id="userHealthBarContainer" style="width: 400px; height: 28px; margin: 16px auto 8px auto; position: relative; background: rgba(0,0,0,0.7); border-radius: 8px; border: 2px solid #fff; display: flex; align-items: center; justify-content: center;">
  <div id="userHealthBarBg" style="position: absolute; left: 0; top: 0; width: 100%; height: 100%; background: #333; border-radius: 8px;"></div>
  <div id="userHealthBar" style="position: absolute; left: 0; top: 0; height: 100%; background: linear-gradient(90deg, #4caf50, #a5d6a7); border-radius: 8px;"></div>
  <span id="userHealthText" style="position: relative; color: #fff; font-weight: bold; font-size: 18px; z-index: 2;">5000 / 5000</span>
</div>
<div id="gameContainer"></div>

<script>
  // --- Coin System ---
  let coins = 500; // Starting coins
  const coinAmountEl = document.getElementById("coinAmount");
  function updateCoinDisplay() {
    coinAmountEl.textContent = coins.toLocaleString();
    // Auto-expand for large numbers
    coinAmountEl.parentElement.style.minWidth = (80 + Math.max(0, (coinAmountEl.textContent.length - 4) * 16)) + "px";
  }
  updateCoinDisplay();

  function addCoins(amount) {
    coins += amount;
    updateCoinDisplay();
  }
  function spendCoins(amount) {
    coins -= amount;
    updateCoinDisplay();
  }

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
 // pathPoints.forEach(point => {
 //   const dot = document.createElement("div");
 //   dot.className = "path-point";
 //   dot.style.left = `${point.x}px`;
 //   dot.style.top = `${point.y}px`;
 //   gameContainer.appendChild(dot);
 // });

  const enemies = [];

  // --- Royal Ghost Reveal Radius ---
  const ROYAL_GHOST_REVEAL_RADIUS = 100;

  // List of air troop names for bomb tower targeting
  const AIR_TROOPS = [
    'Baby Dragon', 'Minion Horde', 'Balloon', 'Lava Hound'
  ];

  function spawnEnemy({ imageSrc, speed, size, health, customStart, coinReward, isRoyalGhost, troopName }) {
    const img = document.createElement("img");
    img.src = imageSrc;
    img.className = "enemy";
    img.style.width = `${size}px`;
    img.style.height = `${size}px`;
    if (isRoyalGhost) {
      img.style.opacity = "0.2";
      img.dataset.royalGhost = "1";
    }

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

    // Use custom start if provided (for witch skeletons)
    const startPoint = customStart ? customStart : pathPoints[0];
    const enemy = {
      el: img,
      healthBarContainer,
      healthBar,
      x: startPoint.x,
      y: startPoint.y,
      speed,
      size,
      health,
      maxHealth: health,
      currentIndex: 0,
      progress: 0,
      segmentDistance: null,
      start: startPoint,
      end: pathPoints[1],
      lastTimestamp: null,
      alive: true,
      coinReward: coinReward || 10, // Default if not specified
      isRoyalGhost: !!isRoyalGhost,
      troopName: troopName || null,
      takeDamage(amount) {
        this.health = Math.max(0, this.health - amount);
        this.updateHealthBar();
        if (this.health === 0) {
          this.alive = false;
          this.el.remove();
          this.healthBarContainer.remove();
          enemies.splice(enemies.indexOf(this), 1);
          if (this.coinReward) addCoins(this.coinReward);
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
  let enemySpawnTimeouts = [];
  let skeletonSpawnCount = 1; // Start with 1 skeleton per spawn
  let minionSpawnCount = 1;   // Start with 1 minion per spawn

  // Track skeleton and minion counts globally for proper reset
  let globalSkeletonCount = 1;
  let globalMinionCount = 1;

  // Store base intervals for each troop type so we can reset them
  const baseIntervals = {
    giant: 10000,
    hog: 15000,
    skeleton: 18000,
    babyDragon: 22000,
    minion: 20000,
    balloon: 30000,
    bandit: 17000,
    pekka: 40000,
    witch: 25000,
    ram: 28000,
    lava: 45000,
    ghost: 35000
  };

  let currentIntervals = { ...baseIntervals };

  function clearEnemySpawns() {
    enemySpawnTimeouts.forEach(id => clearTimeout(id));
    enemySpawnTimeouts = [];
  }

  function resetSpawnState() {
    // Reset spawn counts and intervals
    skeletonSpawnCount = 1;
    minionSpawnCount = 1;
    globalSkeletonCount = 1;
    globalMinionCount = 1;
    currentIntervals = { ...baseIntervals };
  }

  // --- Spawn Interval Minimum Cap ---
  const MIN_SPAWN_INTERVALS = {
    giant: 2000,
    hog: 2000,
    skeleton: 2000,
    babyDragon: 2500,
    minion: 2000,
    balloon: 3000,
    bandit: 2000,
    pekka: 5000,
    witch: 4000,
    ram: 2000,
    lava: 6000,
    ghost: 2000
  };

  function startEnemySpawns() {
    clearEnemySpawns();
    resetSpawnState();

    // Helper for dynamic spawn
    function spawnLoop(spawnFn, intervalKey, onSpawn) {
      let interval = currentIntervals[intervalKey];
      function loop() {
        if (intervalKey === "skeleton") {
          spawnGroup(globalSkeletonCount, () => {
            spawnEnemy({
              speed: 60,
              imageSrc: 'https://i.postimg.cc/J7Z3cnmp/image-2025-05-16-103314524.png',
              size: 40,
              health: 30,
              coinReward: 8,
              troopName: 'Skeleton Army'
            });
          }, 120);
          if (globalSkeletonCount < 64) {
            globalSkeletonCount = Math.min(globalSkeletonCount * 2, 64);
          }
        } else if (intervalKey === "minion") {
          spawnGroup(globalMinionCount, () => {
            spawnEnemy({
              speed: 90,
              imageSrc: 'https://i.postimg.cc/PxZD43GC/image-2025-05-16-103616663.png',
              size: 35,
              health: 25,
              coinReward: 6,
              troopName: 'Minion Horde'
            });
          }, 100);
          if (globalMinionCount < 5) globalMinionCount += 1;
        } else {
          if (onSpawn) onSpawn();
          spawnFn();
        }
        interval = Math.max(interval * 0.97, MIN_SPAWN_INTERVALS[intervalKey] || 2000);
        currentIntervals[intervalKey] = interval;
        enemySpawnTimeouts.push(setTimeout(loop, interval));
      }
      enemySpawnTimeouts.push(setTimeout(loop, interval));
    }

    // Helper to spawn a group with a delay between each
    function spawnGroup(count, spawnFn, delay = 150) {
      let spawned = 0;
      function spawnNext() {
        if (spawned < count) {
          spawnFn(spawned);
          spawned++;
          setTimeout(spawnNext, delay);
        }
      }
      spawnNext();
    }

    // Giant
    spawnLoop(() => {
      spawnEnemy({
        speed: 25,
        imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
        size: 80,
        health: 300,
        coinReward: 120,
        troopName: 'Giant'
      });
    }, "giant");

    // Hog Rider
    spawnLoop(() => {
      spawnEnemy({
        speed: 100,
        imageSrc: 'https://i.postimg.cc/4NxrrzmL/image-2025-05-16-101734632.png',
        size: 60,
        health: 100,
        coinReward: 80,
        troopName: 'Hog Rider'
      });
    }, "hog");

    // Skeleton Army (handled in spawnLoop above)
    spawnLoop(null, "skeleton");

    // Baby Dragon
    spawnLoop(() => {
      spawnEnemy({
        speed: 50,
        imageSrc: 'https://i.postimg.cc/zfyKLD0S/image-2025-05-16-103451709.png',
        size: 55,
        health: 120,
        coinReward: 90,
        troopName: 'Baby Dragon'
      });
    }, "babyDragon");

    // Minion Horde (handled in spawnLoop above)
    spawnLoop(null, "minion");

    // Balloon
    spawnLoop(() => {
      spawnEnemy({
        speed: 30,
        imageSrc: 'https://i.postimg.cc/28TZ2Lt7/image-2025-05-16-103738864.png',
        size: 65,
        health: 150,
        coinReward: 150,
        troopName: 'Balloon'
      });
    }, "balloon");

    // Bandit
    spawnLoop(() => {
      spawnEnemy({
        speed: 120,
        imageSrc: 'https://i.postimg.cc/j2cLgBMS/image-2025-05-16-103917837.png',
        size: 50,
        health: 80,
        coinReward: 60,
        troopName: 'Bandit'
      });
    }, "bandit");

    // P.E.K.K.A
    spawnLoop(() => {
      spawnEnemy({
        speed: 15,
        imageSrc: 'https://i.postimg.cc/FsnKYyq8/image-2025-05-16-104224204.png',
        size: 90,
        health: 500,
        coinReward: 300,
        troopName: 'P.E.K.K.A'
      });
    }, "pekka");

    // Witch
    spawnLoop(() => {
      // Spawn the witch
      const witchEnemyConfig = {
        speed: 35,
        imageSrc: 'https://i.postimg.cc/YqVprpLw/image-2025-05-16-104350767.png',
        size: 60,
        health: 150,
        coinReward: 100,
        troopName: 'Witch'
      };
      // Spawn the witch and get a reference to the enemy object
      let witchEnemyObj = null;
      function spawnWitchAndTrack() {
        const img = document.createElement("img");
        img.src = witchEnemyConfig.imageSrc;
        img.className = "enemy";
        img.style.width = `${witchEnemyConfig.size}px`;
        img.style.height = `${witchEnemyConfig.size}px`;

        const healthBarContainer = document.createElement("div");
        healthBarContainer.style.position = "absolute";
        healthBarContainer.style.width = `${witchEnemyConfig.size}px`;
        healthBarContainer.style.height = "8px";
        healthBarContainer.style.top = "0px";
        healthBarContainer.style.left = "0px";
        healthBarContainer.style.pointerEvents = "none";
        healthBarContainer.style.zIndex = "20";
        healthBarContainer.style.background = "rgba(0,0,0,0.5)";
        healthBarContainer.style.borderRadius = "4px";
        const healthBar = document.createElement("div");
        healthBar.style.height = "100%";
        healthBar.style.width = "100%";
        healthBar.style.background = "linear-gradient(90deg, #4caf50, #a5d6a7)";
        healthBar.style.borderRadius = "4px";
        healthBarContainer.appendChild(healthBar);

        gameContainer.appendChild(img);
        gameContainer.appendChild(healthBarContainer);

        const startPoint = pathPoints[0];
        const enemy = {
          el: img,
          healthBarContainer,
          healthBar,
          x: startPoint.x,
          y: startPoint.y,
          speed: witchEnemyConfig.speed,
          size: witchEnemyConfig.size,
          health: witchEnemyConfig.health,
          maxHealth: witchEnemyConfig.health,
          currentIndex: 0,
          progress: 0,
          segmentDistance: null,
          start: startPoint,
          end: pathPoints[1],
          lastTimestamp: null,
          alive: true,
          coinReward: witchEnemyConfig.coinReward,
          troopName: witchEnemyConfig.troopName,
          takeDamage(amount) {
            this.health = Math.max(0, this.health - amount);
            this.updateHealthBar();
            if (this.health === 0) {
              this.alive = false;
              this.el.remove();
              this.healthBarContainer.remove();
              enemies.splice(enemies.indexOf(this), 1);
              if (this.coinReward) addCoins(this.coinReward);
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
        return enemy;
      }

      witchEnemyObj = spawnWitchAndTrack();

      // Set up repeated skeleton spawns every 10 seconds for this witch
      let witchSkeletonInterval;
      function spawnWitchSkeletons() {
        // Only spawn skeletons if the witch is still alive and on the field
        if (!witchEnemyObj || !witchEnemyObj.alive) return;
        // Find the closest path segment the witch is currently on
        let minDist = Infinity;
        let closestIndex = 0;
        for (let i = 0; i < pathPoints.length - 1; i++) {
          // Project witch position onto the segment
          const sx = pathPoints[i].x, sy = pathPoints[i].y;
          const ex = pathPoints[i + 1].x, ey = pathPoints[i + 1].y;
          const dx = ex - sx, dy = ey - sy;
          const len2 = dx * dx + dy * dy;
          let t = 0;
          if (len2 > 0) {
            t = ((witchEnemyObj.x - sx) * dx + (witchEnemyObj.y - sy) * dy) / len2;
            t = Math.max(0, Math.min(1, t));
          }
          const projX = sx + t * dx;
          const projY = sy + t * dy;
          const dist = Math.hypot(witchEnemyObj.x - projX, witchEnemyObj.y - projY);
          if (dist < minDist) {
            minDist = dist;
            closestIndex = i;
          }
        }
        // Spawn skeletons around the witch's current position, and set their path index to match the witch's
        const wx = witchEnemyObj.x;
        const wy = witchEnemyObj.y;
        const offsets = [
          { dx: 30, dy: 0 },
          { dx: -30, dy: 0 },
          { dx: 0, dy: 30 },
          { dx: 0, dy: -30 }
        ];
        offsets.forEach(offset => {
          // Start at witch's position + offset, and set path following from the witch's current segment
          const start = { x: wx + offset.dx, y: wy + offset.dy };
          const end = pathPoints[closestIndex + 1] || pathPoints[pathPoints.length - 1];
          const skeleton = {
            imageSrc: 'https://i.postimg.cc/J7Z3cnmp/image-2025-05-16-103314524.png',
            speed: 60,
            size: 40,
            health: 30,
            coinReward: 8,
            customStart: start,
            troopName: 'Skeleton Army',
            // Pass custom path index info
            _witchPathIndex: closestIndex
          };
          // Custom spawnEnemy to set the correct path index
          const img = document.createElement("img");
          img.src = skeleton.imageSrc;
          img.className = "enemy";
          img.style.width = `${skeleton.size}px`;
          img.style.height = `${skeleton.size}px`;

          const healthBarContainer = document.createElement("div");
          healthBarContainer.style.position = "absolute";
          healthBarContainer.style.width = `${skeleton.size}px`;
          healthBarContainer.style.height = "8px";
          healthBarContainer.style.top = "0px";
          healthBarContainer.style.left = "0px";
          healthBarContainer.style.pointerEvents = "none";
          healthBarContainer.style.zIndex = "20";
          healthBarContainer.style.background = "rgba(0,0,0,0.5)";
          healthBarContainer.style.borderRadius = "4px";
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
            x: start.x,
            y: start.y,
            speed: skeleton.speed,
            size: skeleton.size,
            health: skeleton.health,
            maxHealth: skeleton.health,
            currentIndex: closestIndex,
            progress: 0,
            segmentDistance: Math.hypot(end.x - start.x, end.y - start.y),
            start: start,
            end: end,
            lastTimestamp: null,
            alive: true,
            coinReward: skeleton.coinReward,
            troopName: skeleton.troopName,
            takeDamage(amount) {
              this.health = Math.max(0, this.health - amount);
              this.updateHealthBar();
              if (this.health === 0) {
                this.alive = false;
                this.el.remove();
                this.healthBarContainer.remove();
                enemies.splice(enemies.indexOf(this), 1);
                if (this.coinReward) addCoins(this.coinReward);
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

          enemy.updateHealthBar();

          enemies.push(enemy);
          requestAnimationFrame(ts => moveEnemy(enemy, ts));
        });
        // Schedule next skeleton spawn for this witch
        witchSkeletonInterval = setTimeout(spawnWitchSkeletons, 10000);
        enemySpawnTimeouts.push(witchSkeletonInterval);
      }
      // Start the repeated skeleton spawn for this witch
      spawnWitchSkeletons();
    }, "witch");

    // Ram Rider
    spawnLoop(() => {
      spawnEnemy({
        speed: 80,
        imageSrc: 'https://i.postimg.cc/YCYwtQw1/image-2025-05-16-104501378.png',
        size: 65,
        health: 180,
        coinReward: 110,
        troopName: 'Ram Rider'
      });
    }, "ram");

    // Lava Hound
    spawnLoop(() => {
      spawnEnemy({
        speed: 20,
        imageSrc: 'https://i.postimg.cc/0QH4YNsk/image-2025-05-16-104632815.png',
        size: 100,
        health: 600,
        coinReward: 250,
        troopName: 'Lava Hound'
      });
    }, "lava");

    // Royal Ghost
    spawnLoop(() => {
      spawnEnemy({
        speed: 70,
        imageSrc: 'https://i.postimg.cc/CxL6QDxh/image-2025-05-16-104742624.png',
        size: 70,
        health: 200,
        coinReward: 90,
        isRoyalGhost: true,
        troopName: 'Royal Ghost'
      });
    }, "ghost");
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
          e.alive = false; // Mark as not alive to stop animation
          e.el.remove();
          if (e.healthBarContainer) e.healthBarContainer.remove();
        });
        enemies.length = 0; // This ensures no invisible enemies remain
        // Remove all towers and their radii
        document.querySelectorAll('.tower').forEach(el => el.remove());
        document.querySelectorAll('.tower-radius').forEach(el => el.remove());
        placedTowers.length = 0; // Clear placed towers array
        // Reset user health
        userHealth = userMaxHealth;
        updateUserHealthBar();
        // Remove popup
        overlay.remove();
        // Clear all enemy spawn timeouts
        clearEnemySpawns();
        // Reset spawn state (intervals and counts)
        resetSpawnState();
        // Start enemy spawns again
        startEnemySpawns();
        // Spawn a giant instantly again
        spawnEnemy({
          speed: 25,
          imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
          size: 80,
          health: 300,
          troopName: 'Giant'
        });
        // Reset coins
        coins = 500;
        updateCoinDisplay();
        // Reset spawn state (intervals and counts)
        resetSpawnState();
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
    if (!enemy.alive) return;
    if (!enemy.lastTimestamp) enemy.lastTimestamp = timestamp;
    const dt = (timestamp - enemy.lastTimestamp) / 1000;
    enemy.lastTimestamp = timestamp;

    enemy.progress += (enemy.speed * dt) / enemy.segmentDistance;

    // --- Royal Ghost Visibility Logic ---
    if (enemy.isRoyalGhost) {
      let revealed = false;
      for (const tower of placedTowers) {
        const dx = tower.x - enemy.x;
        const dy = tower.y - enemy.y;
        if (Math.sqrt(dx * dx + dy * dy) < ROYAL_GHOST_REVEAL_RADIUS) {
          revealed = true;
          break;
        }
      }
      enemy.el.style.opacity = revealed ? "1" : "0.2";
    }

    if (enemy.progress >= 1) {
      enemy.currentIndex++;
      if (typeof enemy.currentIndex !== "undefined" && enemy.currentIndex < pathPoints.length - 1) {
        enemy.start = pathPoints[enemy.currentIndex];
        enemy.end = pathPoints[enemy.currentIndex + 1];
        enemy.segmentDistance = Math.hypot(
          enemy.end.x - enemy.start.x,
          enemy.end.y - enemy.start.y
        );
        enemy.progress = 0;
      } else if (enemy.currentIndex >= pathPoints.length - 1) {
        // Damage user health based on enemy type
        if (enemy.size === 80 && enemy.maxHealth === 300 && enemy.speed === 25) {
          userHealth = Math.max(0, userHealth - 300);
        } else if (enemy.size === 60 && enemy.maxHealth === 100 && enemy.speed === 100) {
          userHealth = Math.max(0, userHealth - 500);
        } else if (enemy.size === 40 && enemy.maxHealth === 30 && enemy.speed === 60) {
          userHealth = Math.max(0, userHealth - 100);
        } else if (enemy.size === 55 && enemy.maxHealth === 120 && enemy.speed === 50) {
          userHealth = Math.max(0, userHealth - 350);
        } else if (enemy.size === 35 && enemy.maxHealth === 25 && enemy.speed === 90) {
          userHealth = Math.max(0, userHealth - 120);
        } else if (enemy.size === 65 && enemy.maxHealth === 150 && enemy.speed === 30) {
          userHealth = Math.max(0, userHealth - 800);
        } else if (enemy.size === 50 && enemy.maxHealth === 80 && enemy.speed === 120) {
          userHealth = Math.max(0, userHealth - 200);
        } else if (enemy.size === 90 && enemy.maxHealth === 500 && enemy.speed === 15) {
          userHealth = Math.max(0, userHealth - 1000);
        } else if (enemy.size === 60 && enemy.maxHealth === 150 && enemy.speed === 35) {
          userHealth = Math.max(0, userHealth - 250);
        } else if (enemy.size === 65 && enemy.maxHealth === 180 && enemy.speed === 80) {
          userHealth = Math.max(0, userHealth - 400);
        } else if (enemy.size === 100 && enemy.maxHealth === 600 && enemy.speed === 20) {
          userHealth = Math.max(0, userHealth - 700);
        } else if (enemy.size === 70 && enemy.maxHealth === 200 && enemy.speed === 70) {
          userHealth = Math.max(0, userHealth - 300);
        }
        updateUserHealthBar(); // <-- Ensure health bar updates here!
        enemy.el.remove();
        if (enemy.healthBarContainer) enemy.healthBarContainer.remove();
        enemy.alive = false;
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
    health: 300,
    coinReward: 120,
    troopName: 'Giant'
  });

  // --- Tower Data ---
  const towerData = [
    { name: 'Archer Tower', imageSrc: 'https://i.postimg.cc/TPZstVyP/image-2025-05-20-095201612.png', radius: 120, cost: 120 },
    { name: 'Wizard Tower', imageSrc: 'https://i.postimg.cc/sx8GWg6b/image-2025-05-20-095324671.png', radius: 100, cost: 220 },
    { name: 'Inferno Tower', imageSrc: 'https://i.postimg.cc/Y9vWCF8Q/image-2025-05-20-095600055.png', radius: 80, cost: 350 },
    { name: 'Tesla Coil', imageSrc: 'https://i.postimg.cc/ZKtdJCNy/image-2025-05-20-095705631.png', radius: 110, cost: 200 },
    { name: 'Bomb Tower', imageSrc: 'https://i.postimg.cc/JhnypXfS/image-2025-05-20-095832932.png', radius: 90, cost: 180 },
    { name: 'Magic Tower', imageSrc: 'https://i.postimg.cc/sDmDbX4z/image-2025-05-20-100137225.png', radius: 105, cost: 260 },
    { name: 'Freeze Tower', imageSrc: 'https://i.postimg.cc/jqQsJ59f/image-2025-05-20-100226131.png', radius: 95, cost: 210 },
    { name: 'Hunter Nest', imageSrc: 'https://i.postimg.cc/P5chk8Lg/image-2025-05-20-100344794.png', radius: 100, cost: 170 },
    { name: 'Lightning Obelisk', imageSrc: 'https://i.postimg.cc/br3cVGcF/image-2025-05-20-100433723.png', radius: 115, cost: 320 },
    { name: 'Rage Beacon', imageSrc: 'https://i.postimg.cc/PfKgj89S/image-2025-05-20-100521354.png', radius: 130, cost: 400 },
  ];

  const placedTowers = [];

  function isValidTowerPlacement(x, y, radius) {
    // Prevent placement if too close to any path segment (buffer 40px)
    const buffer = 40;
    for (let i = 0; i < pathPoints.length - 1; i++) {
      const p1 = pathPoints[i];
      const p2 = pathPoints[i + 1];
      // Project (x, y) onto the segment p1-p2
      const dx = p2.x - p1.x;
      const dy = p2.y - p1.y;
      const len2 = dx * dx + dy * dy;
      let t = 0;
      if (len2 > 0) {
        t = ((x - p1.x) * dx + (y - p1.y) * dy) / len2;
        t = Math.max(0, Math.min(1, t));
      }
      const projX = p1.x + t * dx;
      const projY = p1.y + t * dy;
      const dist = Math.hypot(x - projX, y - projY);
      if (dist < buffer) return false;
    }
    return true;
  }

  function renderTowers() {
    // Remove all previous tower images and radii
    document.querySelectorAll('.tower').forEach(el => el.remove());
    document.querySelectorAll('.tower-radius').forEach(el => el.remove());
    placedTowers.forEach(tower => {
      // Draw attack radius
      const radiusDiv = document.createElement('div');
      radiusDiv.className = 'tower-radius';
      radiusDiv.style.position = 'absolute';
      radiusDiv.style.left = `${tower.x - tower.radius}px`;
      radiusDiv.style.top = `${tower.y - tower.radius}px`;
      radiusDiv.style.width = `${tower.radius * 2}px`;
      radiusDiv.style.height = `${tower.radius * 2}px`;
      radiusDiv.style.borderRadius = '50%';
      radiusDiv.style.background = 'rgba(0, 200, 255, 0.15)';
      radiusDiv.style.border = '2px dashed #00bcd4';
      radiusDiv.style.pointerEvents = 'none';
      radiusDiv.style.zIndex = 10;
      gameContainer.appendChild(radiusDiv);
      // Draw tower image
      const img = document.createElement('img');
      img.src = tower.imageSrc;
      img.className = 'tower';
      img.style.position = 'absolute';
      img.style.left = `${tower.x - 25}px`;
      img.style.top = `${tower.y - 25}px`;
      img.style.width = '50px';
      img.style.height = '50px';
      img.title = tower.name;
      img.style.zIndex = 15;
      gameContainer.appendChild(img);
    });
  }

  // --- Tower Menu Centered Below Map ---
  function showTowerMenu() {
    let menu = document.getElementById('towerMenu');
    if (menu) menu.remove();
    menu = document.createElement('div');
    menu.id = 'towerMenu';
    menu.style.position = 'absolute';
    menu.style.left = (gameContainer.offsetLeft + gameContainer.offsetWidth / 2 - 320) + 'px';
    menu.style.top = (gameContainer.offsetTop + gameContainer.offsetHeight + 16) + 'px';
    menu.style.width = '640px';
    menu.style.background = 'rgba(30,30,30,0.95)';
    menu.style.border = '2px solid #fff';
    menu.style.borderRadius = '10px';
    menu.style.padding = '12px';
    menu.style.zIndex = 10001;
    menu.style.display = 'flex';
    menu.style.flexDirection = 'column';
    menu.style.gap = '8px';
    menu.innerHTML = '<b style="color:#fff;">Drag Towers</b>';
    const spriteRow = document.createElement('div');
    spriteRow.style.display = 'flex';
    spriteRow.style.gap = '18px';
    spriteRow.style.justifyContent = 'center';
    towerData.forEach((tower, idx) => {
      const spriteCol = document.createElement('div');
      spriteCol.style.display = 'flex';
      spriteCol.style.flexDirection = 'column';
      spriteCol.style.alignItems = 'center';
      const sprite = document.createElement('img');
      sprite.src = tower.imageSrc;
      sprite.title = tower.name + " (" + tower.cost + " coins)";
      sprite.style.width = '48px';
      sprite.style.height = '48px';
      sprite.style.cursor = 'grab';
      sprite.draggable = true;
      sprite.ondragstart = e => {
        e.dataTransfer.setData('towerIdx', idx);
      };
      // Show cost below sprite
      const costLabel = document.createElement('span');
      costLabel.textContent = tower.cost + "🪙";
      costLabel.style.color = "#ffd700";
      costLabel.style.fontWeight = "bold";
      costLabel.style.fontSize = "15px";
      costLabel.style.marginTop = "2px";
      spriteCol.appendChild(sprite);
      spriteCol.appendChild(costLabel);
      spriteRow.appendChild(spriteCol);
    });
    menu.appendChild(spriteRow);
    document.body.appendChild(menu);
  }

  showTowerMenu();

  gameContainer.ondragover = e => {
    e.preventDefault();
  };
  gameContainer.ondrop = e => {
    e.preventDefault();
    const rect = gameContainer.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    const idx = e.dataTransfer.getData('towerIdx');
    const tower = towerData[idx];
    if (!tower) return;
    if (!isValidTowerPlacement(x, y, tower.radius)) {
      alert('Cannot place tower here! Too close to path.');
      return;
    }
    if (coins < tower.cost) {
      alert('Not enough coins! (' + tower.cost + ' needed)');
      return;
    }
    spendCoins(tower.cost);
    placedTowers.push({ ...tower, x, y });
    renderTowers();
  };

  // --- Tower Attack Logic ---
  const TOWER_ATTACK_INTERVAL = 500; // ms
  const INFERNO_RAMP = [4, 6, 8, 12, 18]; // Slower and less aggressive ramp-up
  const ARCHER_DAMAGE = 30;
  const BOMB_DAMAGE = 18; // Reduced damage
  const BOMB_RADIUS = 50;

  const ARCHER_ARROW_IMG = 'https://i.postimg.cc/gjznhbcv/image-2025-05-21-114040090.png';
  const WIZARD_FIREBALL_IMG = 'https://i.postimg.cc/TwGw8vDZ/image-2025-05-21-114249663.png';
  const BOMB_PROJECTILE_IMG = 'https://i.postimg.cc/L6qPCWkV/download-removebg-preview.png';

  const towerAttackState = new Map(); // For inferno ramping

  function spawnProjectile({fromX, fromY, toX, toY, imgSrc, speed, onHit}) {
    const img = document.createElement('img');
    img.src = imgSrc;
    img.style.position = 'absolute';
    img.style.left = fromX + 'px';
    img.style.top = fromY + 'px';
    img.style.width = '40px'; // Increased size
    img.style.height = '40px'; // Increased size
    img.style.zIndex = 20;
    img.className = 'projectile';
    gameContainer.appendChild(img);
    const dx = toX - fromX;
    const dy = toY - fromY;
    const dist = Math.sqrt(dx*dx + dy*dy);
    const duration = dist / speed * 1000;
    let start = null;
    function animate(ts) {
      if (!start) start = ts;
      const elapsed = ts - start;
      const t = Math.min(1, elapsed / duration);
      const x = fromX + dx * t;
      const y = fromY + dy * t;
      img.style.left = x + 'px';
      img.style.top = y + 'px';
      if (t < 1) {
        requestAnimationFrame(animate);
      } else {
        img.remove();
        if (onHit) onHit();
      }
    }
    requestAnimationFrame(animate);
  }

  function towerAttackLoop() {
    placedTowers.forEach(tower => {
      if (tower.name === 'Inferno Tower') {
        // Find nearest enemy in range
        let nearest = null, minDist = Infinity;
        enemies.forEach(enemy => {
          if (!enemy.alive) return;
          const dx = tower.x - enemy.x;
          const dy = tower.y - enemy.y;
          const dist = Math.sqrt(dx*dx + dy*dy);
          if (dist < tower.radius && dist < minDist) {
            nearest = enemy;
            minDist = dist;
          }
        });
        if (nearest) {
          // Ramp up damage if same target, else reset
          let state = towerAttackState.get(tower) || { target: null, ramp: 0, lastTime: 0 };
          if (state.target === nearest) {
            // Only ramp up every 2 attack intervals (slower ramp)
            if (!state.lastTime || performance.now() - state.lastTime > TOWER_ATTACK_INTERVAL * 2) {
              state.ramp = Math.min(state.ramp + 1, INFERNO_RAMP.length - 1);
              state.lastTime = performance.now();
            }
          } else {
            state.target = nearest;
            state.ramp = 0;
            state.lastTime = performance.now();
          }
          nearest.takeDamage(INFERNO_RAMP[state.ramp]);
          towerAttackState.set(tower, state);
        } else {
          towerAttackState.delete(tower);
        }
      } else if (tower.name === 'Archer Tower') {
        // Shoot at nearest enemy in range
        let lastShot = tower.lastShot || 0;
        if (performance.now() - lastShot > 700) {
          let nearest = null, minDist = Infinity;
          enemies.forEach(enemy => {
            if (!enemy.alive) return;
            const dx = tower.x - enemy.x;
            const dy = tower.y - enemy.y;
            const dist = Math.sqrt(dx*dx + dy*dy);
            if (dist < tower.radius && dist < minDist) {
              nearest = enemy;
              minDist = dist;
            }
          });
          if (nearest) {
            // Arrow projectile
            spawnProjectile({
              fromX: tower.x,
              fromY: tower.y,
              toX: nearest.x,
              toY: nearest.y,
              imgSrc: ARCHER_ARROW_IMG,
              speed: 600,
              onHit: () => nearest.takeDamage(ARCHER_DAMAGE)
            });
            tower.lastShot = performance.now();
          }
        }
      } else if (tower.name === 'Bomb Tower') {
        // Bomb Tower: splash damage, does NOT attack air troops
        let lastShot = tower.lastShot || 0;
        if (performance.now() - lastShot > 1200) {
          let nearest = null, minDist = Infinity;
          enemies.forEach(enemy => {
            if (!enemy.alive) return;
            // Only target ground troops
            if (AIR_TROOPS.includes(enemy.troopName)) return;
            const dx = tower.x - enemy.x;
            const dy = tower.y - enemy.y;
            const dist = Math.sqrt(dx*dx + dy*dy);
            if (dist < tower.radius && dist < minDist) {
              nearest = enemy;
              minDist = dist;
            }
          });
          if (nearest) {
            // Bomb projectile
            spawnProjectile({
              fromX: tower.x,
              fromY: tower.y,
              toX: nearest.x,
              toY: nearest.y,
              imgSrc: BOMB_PROJECTILE_IMG,
              speed: 400,
              onHit: () => {
                enemies.forEach(enemy2 => {
                  if (!enemy2.alive) return;
                  if (AIR_TROOPS.includes(enemy2.troopName)) return;
                  const dx = nearest.x - enemy2.x;
                  const dy = nearest.y - enemy2.y;
                  if (Math.sqrt(dx*dx + dy*dy) < BOMB_RADIUS) {
                    enemy2.takeDamage(BOMB_DAMAGE);
                  }
                });
              }
            });
            tower.lastShot = performance.now();
          }
        }
      } else if (tower.name === 'Wizard Tower') {
        // Wizard Tower: fireball projectile
        let lastShot = tower.lastShot || 0;
        if (performance.now() - lastShot > 1200) {
          let nearest = null, minDist = Infinity;
          enemies.forEach(enemy => {
            if (!enemy.alive) return;
            const dx = tower.x - enemy.x;
            const dy = tower.y - enemy.y;
            const dist = Math.sqrt(dx*dx + dy*dy);
            if (dist < tower.radius && dist < minDist) {
              nearest = enemy;
              minDist = dist;
            }
          });
          if (nearest) {
            spawnProjectile({
              fromX: tower.x,
              fromY: tower.y,
              toX: nearest.x,
              toY: nearest.y,
              imgSrc: WIZARD_FIREBALL_IMG,
              speed: 400,
              onHit: () => nearest.takeDamage(60)
            });
            tower.lastShot = performance.now();
          }
        }
      }
    });
    setTimeout(towerAttackLoop, TOWER_ATTACK_INTERVAL);
  }
  towerAttackLoop();

  window.addEventListener('DOMContentLoaded', () => {
    // Move coin display to be relative to the gameContainer (top right of map)
    const coinDisplay = document.getElementById('coinDisplay');
    const gameContainer = document.getElementById('gameContainer');
    function positionCoinDisplay() {
      const rect = gameContainer.getBoundingClientRect();
      coinDisplay.style.position = 'absolute';
      coinDisplay.style.left = (gameContainer.offsetWidth - coinDisplay.offsetWidth - 24) + 'px';
      coinDisplay.style.top = '24px';
      coinDisplay.style.right = '';
      coinDisplay.style.pointerEvents = 'none';
      gameContainer.appendChild(coinDisplay);
    }
    positionCoinDisplay();
    window.addEventListener('resize', positionCoinDisplay);
  });
</script>