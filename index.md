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

    .tower {
      position: absolute;
      width: 50px;
      height: 50px;
      z-index: 15;
    }

    .tower-label {
      position: absolute;
      font-size: 12px;
      font-weight: bold;
      color: yellow;
      text-shadow: 1px 1px 2px #000;
      z-index: 16;
    }

    .upgrade-btn {
      position: absolute;
      font-size: 10px;
      background: #222;
      color: #0f0;
      border: 1px solid #0f0;
      border-radius: 4px;
      padding: 2px 4px;
      cursor: pointer;
      z-index: 17;
    }

    .enemy {
      position: absolute;
      z-index: 5;
      pointer-events: none;
    }

  #pointsDisplay {
    position: absolute;
    left: 16px;
    top: 16px;
    min-width: 0;
    width: auto;
    height: 48px;
    background: rgba(0,0,0,0.7);
    border: 2px solid #0074D9; /* border matches blue text */
    border-radius: 24px;
    display: flex;
    align-items: center;
    justify-content: center; /* center horizontally */
    font-size: 28px;
    font-weight: bold;
    color: #0074D9; /* blue text */
    z-index: 10003;
    box-shadow: 0 2px 8px #0008;
    padding: 0 18px;
    pointer-events: none;
    text-shadow: 1px 1px 2px #000, 0 0 2px #000; /* black shadow for contrast */
  }
  #pointsAmount {
    min-width: 2ch;
    display: inline-block;
    text-align: right;
    margin-left: 6px;
  }
</style>

<!-- Points Display -->
<div id="pointsDisplay">Points: <span id="pointsAmount">0</span></div>
<!-- Coin Display -->
<div id="coinDisplay" style="min-width:0;width:auto;height:48px;background:rgba(255,215,0,0.15);border:2px solid #ffd700;border-radius:24px;display:flex;align-items:center;justify-content:center;font-size:28px;font-weight:bold;color:#ffd700;z-index:10002;box-shadow:0 2px 8px #0008;pointer-events:none;padding:0 18px;">
  <span id="coinAmount" style="min-width:2ch;text-align:right;display:inline-block;flex:1 1 auto;margin-left:6px;">0</span>
</div>

<div id="userHealthBarContainer" style="width: 400px; height: 28px; margin: 16px auto 8px auto; position: relative; background: rgba(0,0,0,0.7); border-radius: 8px; border: 2px solid #fff; display: flex; align-items: center; justify-content: center;">
  <div id="userHealthBarBg" style="position: absolute; left: 0; top: 0; width: 100%; height: 100%; background: #333; border-radius: 8px;"></div>
  <div id="userHealthBar" style="position: absolute; left: 0; top: 0; height: 100%; background: linear-gradient(90deg, #4caf50, #a5d6a7); border-radius: 8px;"></div>
  <span id="userHealthText" style="position: relative; color: #fff; font-weight: bold; font-size: 18px; z-index: 2;">5000 / 5000</span>
</div>
<div id="gameContainer"></div>

<script type="module">
  // Import config values
  import { pythonURI, fetchOptions } from '{{ site.baseurl }}/assets/js/api/config.js';

  // --- OOP Refactor ---

  // --- Utility ---
  function lerp(a, b, t) { return a + (b - a) * t; }
  function dist(x1, y1, x2, y2) { return Math.hypot(x2 - x1, y2 - y1); }

  // --- Constants ---
  const AIR_TROOPS = ['Baby Dragon', 'Minion Horde', 'Balloon', 'Lava Hound'];
  const TOWER_ATTACK_INTERVAL = 500;
  const INFERNO_RAMP = [6, 12, 24, 48, 96]; // Increased scaling for inferno tower
  const ARCHER_DAMAGE = 18; // Reduced archer tower damage
  const BOMB_DAMAGE = 18;
  const BOMB_RADIUS = 50;
  const ARCHER_ARROW_IMG = 'https://i.postimg.cc/gjznhbcv/image-2025-05-21-114040090.png';
  const WIZARD_FIREBALL_IMG = 'https://i.postimg.cc/TwGw8vDZ/image-2025-05-21-114249663.png';
  const INFERNO_BEAM_IMG = 'https://i.postimg.cc/cLc8rtQv/image-2025-05-21-224441846.png';
  const LIGHTNING_BOLT_IMG = 'https://i.postimg.cc/4dY56Q5L/image-2025-05-21-224712958.png';
  const BOMB_PROJECTILE_IMG = 'https://i.postimg.cc/L6qPCWkV/download-removebg-preview.png';
  const MAGIC_BEAM_IMG = 'https://i.postimg.cc/hGVgS6Ng/image-2025-05-21-225231613.png';
  const HUNTER_CANNONBALL_IMG = 'https://i.postimg.cc/c1tDmc2t/image-2025-05-29-095416276.png';
  const FREEZE_ICE_IMG = 'https://i.postimg.cc/k4vW1BQZ/image-2025-05-29-100856983.png';
  const LIGHTNING_OBELISK_IMG = 'https://i.postimg.cc/4dYmty8V/image-2025-05-29-101044073.png';

  // --- Path Points ---
  const pathPoints = [
    { x: 100, y: 245 }, { x: 500, y: 245 }, { x: 500, y: 100 }, { x: 328, y: 100 },
    { x: 328, y: 475 }, { x: 145, y: 475 }, { x: 145, y: 350 }, { x: 625, y: 350 },
    { x: 625, y: 193 }, { x: 745, y: 193 }, { x: 745, y: 442 }, { x: 433, y: 442 },
    { x: 433, y: 510 },
  ];

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

  // --- Points API Integration ---
  class Points {
    constructor({ pythonURI, fetchOptions }) {
      this.pythonURI = pythonURI;
      this.fetchOptions = fetchOptions;
      this.points = 0;
      this.pointsDisplay = document.getElementById('pointsAmount');
      this.init();
    }

    async init() {
      await this.fetchPoints();
      this.startAutoIncrement();
    }

    async fetchPoints() {
      try {
        const response = await fetch(`${this.pythonURI}/api/points`, {
          ...this.fetchOptions,
          method: 'GET',
        });
        if (!response.ok) throw new Error('Failed to fetch points');
        const data = await response.json();
        this.points = data.points?.points || 0;
        this.updateDisplay();
      } catch (error) {
        console.error('Error fetching points:', error);
      }
    }

    async addPoints(amount) {
      try {
        // Try to update (PUT), if not exists then POST
        const response = await fetch(`${this.pythonURI}/api/points`, {
          ...this.fetchOptions,
          method: 'PUT',
          headers: { ...this.fetchOptions.headers, 'Content-Type': 'application/json' },
          body: JSON.stringify({ points: this.points + amount })
        });
        let data;
        if (response.ok) {
          data = await response.json();
        } else if (response.status === 404) {
          // No entry, create new
          const postResp = await fetch(`${this.pythonURI}/api/points`, {
            ...this.fetchOptions,
            method: 'POST',
            headers: { ...this.fetchOptions.headers, 'Content-Type': 'application/json' },
            body: JSON.stringify({ points: amount })
          });
          data = await postResp.json();
        } else {
          data = await response.json();
          throw new Error(data.message || 'Failed to add points');
        }
        this.points = data.points?.points || this.points + amount;
        this.updateDisplay();
      } catch (error) {
        console.error('Error adding points:', error);
      }
    }

    updateDisplay() {
      if (this.pointsDisplay) {
        this.pointsDisplay.textContent = ' ' + this.points;
      }
    }

    startAutoIncrement() {
      setInterval(() => {
        this.addPoints(5);
      }, 20000);
    }
  }

  // --- Usage Example ---
  window.PointsAPI = new Points({ pythonURI, fetchOptions });

  // --- Classes ---
  class Enemy {
    constructor(game, config) {
      this.game = game;
      this.x = (config.customStart ? config.customStart.x : pathPoints[0].x);
      this.y = (config.customStart ? config.customStart.y : pathPoints[0].y);
      this.speed = config.speed;
      this.size = config.size;
      this.health = config.health;
      this.maxHealth = config.health;
      this.coinReward = config.coinReward || 10;
      this.isRoyalGhost = !!config.isRoyalGhost;
      this.troopName = config.troopName || null;
      this.currentIndex = config._witchPathIndex || 0;
      this.progress = 0;
      this.start = config.customStart || pathPoints[this.currentIndex];
      this.end = pathPoints[this.currentIndex + 1];
      this.segmentDistance = Math.hypot(this.end.x - this.start.x, this.end.y - this.start.y);
      this.lastTimestamp = null;
      this.alive = true;

      // DOM
      this.el = document.createElement("img");
      this.el.src = config.imageSrc;
      this.el.className = "enemy";
      this.el.style.width = `${this.size}px`;
      this.el.style.height = `${this.size}px`;
      if (this.isRoyalGhost) {
        this.el.style.opacity = "0.15"; // More invisible by default
        this.el.dataset.royalGhost = "1";
      }
      this.healthBarContainer = document.createElement("div");
      this.healthBarContainer.style.position = "absolute";
      this.healthBarContainer.style.width = `${this.size}px`;
      this.healthBarContainer.style.height = "8px";
      this.healthBarContainer.style.top = "0px";
      this.healthBarContainer.style.left = "0px";
      this.healthBarContainer.style.pointerEvents = "none";
      this.healthBarContainer.style.zIndex = "20";
      this.healthBarContainer.style.background = "rgba(0,0,0,0.5)";
      this.healthBarContainer.style.borderRadius = "4px";
      this.healthBar = document.createElement("div");
      this.healthBar.style.height = "100%";
      this.healthBar.style.width = "100%";
      this.healthBar.style.background = "linear-gradient(90deg, #4caf50, #a5d6a7)";
      this.healthBar.style.borderRadius = "4px";
      this.healthBarContainer.appendChild(this.healthBar);

      this.game.gameContainer.appendChild(this.el);
      this.game.gameContainer.appendChild(this.healthBarContainer);

      this.updateHealthBar();
      this.game.enemies.push(this);
      requestAnimationFrame(ts => this.move(ts));
    }
    takeDamage(amount) {
      this.health = Math.max(0, this.health - amount);
      this.updateHealthBar();
      if (this.health === 0) {
        this.alive = false;
        this.el.remove();
        this.healthBarContainer.remove();
        this.game.enemies.splice(this.game.enemies.indexOf(this), 1);
        if (this.coinReward) this.game.addCoins(this.coinReward);
      }
    }
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
    move(timestamp) {
      if (!this.alive) return;
      if (!this.lastTimestamp) this.lastTimestamp = timestamp;
      const dt = (timestamp - this.lastTimestamp) / 1000;
      this.lastTimestamp = timestamp;
      this.progress += (this.speed * dt) / this.segmentDistance;

      // Royal Ghost visibility and attackability
      if (this.isRoyalGhost) {
        let revealed = false;
        for (const tower of this.game.placedTowers) {
          const dx = tower.x - this.x;
          const dy = tower.y - this.y;
          const revealRadius = tower.radius * 0.75;
          const d = Math.sqrt(dx * dx + dy * dy);
          if (d < revealRadius) {
            revealed = true;
            break;
          }
        }
        if (revealed) {
          // Instantly fully visible, no blue glow, no opacity ramp
          this.el.style.opacity = "1";
          this.el.style.filter = "";
          this.el.style.outline = "";
          this.canBeTargeted = true;
        } else {
          this.el.style.opacity = "0.15";
          this.el.style.filter = "";
          this.el.style.outline = "";
          this.canBeTargeted = false;
        }
      }

      if (this.progress >= 1) {
        this.currentIndex++;
        if (this.currentIndex < pathPoints.length - 1) {
          this.start = pathPoints[this.currentIndex];
          this.end = pathPoints[this.currentIndex + 1];
          this.segmentDistance = Math.hypot(this.end.x - this.start.x, this.end.y - this.start.y);
          this.progress = 0;
        } else {
          // Damage user health based on enemy type
          this.game.damageUserHealth(this);
          this.el.remove();
          this.healthBarContainer.remove();
          this.alive = false;
          this.game.enemies.splice(this.game.enemies.indexOf(this), 1);
          return;
        }
      }
      this.x = lerp(this.start.x, this.end.x, this.progress);
      this.y = lerp(this.start.y, this.end.y, this.progress);
      this.el.style.left = `${this.x - this.size / 2}px`;
      this.el.style.top = `${this.y - this.size / 2}px`;
      this.healthBarContainer.style.left = `${this.x - this.size / 2}px`;
      this.healthBarContainer.style.top = `${this.y - this.size / 2 - 12}px`;
      requestAnimationFrame(ts => this.move(ts));
    }
  }

  class Tower {
    constructor(game, config, x, y) {
      this.game = game;
      this.name = config.name;
      this.imageSrc = config.imageSrc;
      this.radius = config.radius;
      this.cost = config.cost;
      this.x = x;
      this.y = y;
      this.lastShot = 0;
      this.state = {};
    }
  }

  class Projectile {
    constructor(game, {fromX, fromY, toX, toY, imgSrc, speed, onHit}) {
      this.game = game;
      this.img = document.createElement('img');
      this.img.src = imgSrc;
      this.img.style.position = 'absolute';
      this.img.style.left = fromX + 'px';
      this.img.style.top = fromY + 'px';
      this.img.style.width = '40px';
      this.img.style.height = '40px';
      this.img.style.zIndex = 20;
      this.img.className = 'projectile';
      this.game.gameContainer.appendChild(this.img);
      this.fromX = fromX;
      this.fromY = fromY;
      this.toX = toX;
      this.toY = toY;
      this.speed = speed;
      this.onHit = onHit;
      this.dx = toX - fromX;
      this.dy = toY - fromY;
      this.dist = Math.sqrt(this.dx * this.dx + this.dy * this.dy);
      this.duration = this.dist / speed * 1000;
      this.start = null;
      requestAnimationFrame(ts => this.animate(ts));
    }
    animate(ts) {
      if (!this.start) this.start = ts;
      const elapsed = ts - this.start;
      const t = Math.min(1, elapsed / this.duration);
      const x = lerp(this.fromX, this.toX, t);
      const y = lerp(this.fromY, this.toY, t);
      this.img.style.left = x + 'px';
      this.img.style.top = y + 'px';
      if (t < 1) {
        requestAnimationFrame(ts => this.animate(ts));
      } else {
        this.img.remove();
        if (this.onHit) this.onHit();
      }
    }
  }

  class Game {
    constructor() {
      // DOM
      this.gameContainer = document.getElementById("gameContainer");
      this.coinAmountEl = document.getElementById("coinAmount");
      this.userHealthBar = document.getElementById("userHealthBar");
      this.userHealthText = document.getElementById("userHealthText");
      this.userHealthBarContainer = document.getElementById("userHealthBarContainer");

      // State
      this.coins = 500;
      this.userHealth = 5000;
      this.userMaxHealth = 5000;
      this.enemies = [];
      this.placedTowers = [];
      this.towerAttackState = new Map();
      this.enemySpawnTimeouts = [];
      this.skeletonSpawnCount = 1;
      this.minionSpawnCount = 1;
      this.globalSkeletonCount = 1;
      this.globalMinionCount = 1;
      this.baseIntervals = {
        giant: 10000, hog: 15000, skeleton: 18000, babyDragon: 22000, minion: 20000,
        balloon: 30000, bandit: 17000, pekka: 40000, witch: 25000, ram: 28000, lava: 45000, ghost: 35000
      };
      this.currentIntervals = {...this.baseIntervals};
      this.MIN_SPAWN_INTERVALS = {
        giant: 2000, hog: 2000, skeleton: 2000, babyDragon: 2500, minion: 2000,
        balloon: 3000, bandit: 2000, pekka: 5000, witch: 4000, ram: 2000, lava: 6000, ghost: 2000
      };

      // UI
      this.updateCoinDisplay();
      this.updateUserHealthBar();
      this.showTowerMenu();
      this.setupDragDrop();
      this.startEnemySpawns();
      this.spawnEnemy({ // spawn a giant instantly
        speed: 25,
        imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
        size: 80,
        health: 300,
        coinReward: 120,
        troopName: 'Giant'
      });
      this.towerAttackLoop();
    }

    updateCoinDisplay() {
      this.coinAmountEl.textContent = ' ' + this.coins.toLocaleString();
      // No need to set minWidth, flex and padding handle expansion
    }
    addCoins(amount) {
      this.coins += amount;
      this.updateCoinDisplay();
    }
    spendCoins(amount) {
      this.coins -= amount;
      this.updateCoinDisplay();
    }
    updateUserHealthBar() {
      const percent = Math.max(0, this.userHealth / this.userMaxHealth);
      this.userHealthBar.style.width = `${percent * 100}%`;
      if (percent > 0.5) {
        this.userHealthBar.style.background = "linear-gradient(90deg, #4caf50, #a5d6a7)";
      } else if (percent > 0.2) {
        this.userHealthBar.style.background = "linear-gradient(90deg, #ffc107, #ffe082)";
      } else {
        this.userHealthBar.style.background = "linear-gradient(90deg, #f44336, #ff8a65)";
      }
      this.userHealthText.textContent = `${this.userHealth} / ${this.userMaxHealth}`;
      if (this.userHealth === 0 && !document.getElementById("gameOverPopup")) {
        this.showGameOverPopup();
      }
    }
    damageUserHealth(enemy) {
      // Use same logic as before
      if (enemy.size === 80 && enemy.maxHealth === 300 && enemy.speed === 25) {
        this.userHealth = Math.max(0, this.userHealth - 300);
      } else if (enemy.size === 60 && enemy.maxHealth === 100 && enemy.speed === 100) {
        this.userHealth = Math.max(0, this.userHealth - 500);
      } else if (enemy.size === 40 && enemy.maxHealth === 30 && enemy.speed === 60) {
        this.userHealth = Math.max(0, this.userHealth - 100);
      } else if (enemy.size === 55 && enemy.maxHealth === 120 && enemy.speed === 50) {
        this.userHealth = Math.max(0, this.userHealth - 350);
      } else if (enemy.size === 35 && enemy.maxHealth === 25 && enemy.speed === 90) {
        this.userHealth = Math.max(0, this.userHealth - 120);
      } else if (enemy.size === 65 && enemy.maxHealth === 150 && enemy.speed === 30) {
        this.userHealth = Math.max(0, this.userHealth - 800);
      } else if (enemy.size === 50 && enemy.maxHealth === 80 && enemy.speed === 120) {
        this.userHealth = Math.max(0, this.userHealth - 200);
      } else if (enemy.size === 90 && enemy.maxHealth === 500 && enemy.speed === 15) {
        this.userHealth = Math.max(0, this.userHealth - 1000);
      } else if (enemy.size === 60 && enemy.maxHealth === 150 && enemy.speed === 35) {
        this.userHealth = Math.max(0, this.userHealth - 250);
      } else if (enemy.size === 65 && enemy.maxHealth === 180 && enemy.speed === 80) {
        this.userHealth = Math.max(0, this.userHealth - 400);
      } else if (enemy.size === 100 && enemy.maxHealth === 600 && enemy.speed === 20) {
        this.userHealth = Math.max(0, this.userHealth - 700);
      } else if (enemy.size === 70 && enemy.maxHealth === 200 && enemy.speed === 70) {
        this.userHealth = Math.max(0, this.userHealth - 300);
      }
      this.updateUserHealthBar();
    }
    showGameOverPopup() {
      // ...existing code for popup, but replace all global references with this.*
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
      const text = document.createElement("div");
      text.textContent = "Game Over";
      text.style.color = "#fff";
      text.style.fontSize = "48px";
      text.style.fontWeight = "bold";
      text.style.marginBottom = "24px";
      overlay.appendChild(text);
      document.body.appendChild(overlay);
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
        restartBtn.style.zIndex = "10000";
        restartBtn.onclick = () => {
          // Remove all enemies and clear the array
          this.enemies.forEach(e => {
            e.alive = false;
            e.el.remove();
            if (e.healthBarContainer) e.healthBarContainer.remove();
          });
          this.enemies.length = 0;
          // Remove all towers and their radii
          document.querySelectorAll('.tower').forEach(el => el.remove());
          document.querySelectorAll('.tower-radius').forEach(el => el.remove());
          this.placedTowers.length = 0;
          // Reset user health
          this.userHealth = this.userMaxHealth;
          this.updateUserHealthBar();
          overlay.remove();
          // --- Reset enemy spawns and intervals ---
          this.clearEnemySpawns();
          this.resetSpawnState();
          // Reset spawn intervals and minion/skeleton counts
          this.currentIntervals = {...this.baseIntervals};
          this.skeletonSpawnCount = 1;
          this.minionSpawnCount = 1;
          this.globalSkeletonCount = 1;
          this.globalMinionCount = 1;
          // --- End any pending skeleton/minion group spawns ---
          if (this._skeletonGroupTimeout) {
            clearTimeout(this._skeletonGroupTimeout);
            this._skeletonGroupTimeout = null;
          }
          if (this._minionGroupTimeout) {
            clearTimeout(this._minionGroupTimeout);
            this._minionGroupTimeout = null;
          }
          // --- Replace spawnGroup to allow cancelation ---
          this.spawnGroup = (count, spawnFn, delay = 150, groupType = null) => {
            let spawned = 0;
            const spawnNext = () => {
              if (spawned < count) {
                spawnFn(spawned);
                spawned++;
                if (groupType === "skeleton") {
                  this._skeletonGroupTimeout = setTimeout(spawnNext, delay);
                } else if (groupType === "minion") {
                  this._minionGroupTimeout = setTimeout(spawnNext, delay);
                } else {
                  setTimeout(spawnNext, delay);
                }
              }
            };
            spawnNext();
          };
          // Start enemy spawns again
          this.startEnemySpawns();
          this.spawnEnemy({
            speed: 25,
            imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
            size: 80,
            health: 300,
            troopName: 'Giant'
          });
          this.coins = 500;
          this.updateCoinDisplay();
          // Reset spawn state again for safety
          this.resetSpawnState();
        };
        overlay.appendChild(restartBtn);
        setTimeout(() => {
          restartBtn.style.background = "#4caf50";
          restartBtn.style.opacity = "1";
        }, 50);
      }, 3000);
    }
    clearEnemySpawns() {
      this.enemySpawnTimeouts.forEach(id => clearTimeout(id));
      this.enemySpawnTimeouts = [];
      // --- Also clear any pending skeleton/minion group spawns ---
      if (this._skeletonGroupTimeout) {
        clearTimeout(this._skeletonGroupTimeout);
        this._skeletonGroupTimeout = null;
      }
      if (this._minionGroupTimeout) {
        clearTimeout(this._minionGroupTimeout);
        this._minionGroupTimeout = null;
      }
    }
    resetSpawnState() {
      this.skeletonSpawnCount = 1;
      this.minionSpawnCount = 1;
      this.globalSkeletonCount = 1;
      this.globalMinionCount = 1;
      this.currentIntervals = {...this.baseIntervals};
    }
    spawnEnemy(config) {
      return new Enemy(this, config);
    }
    startEnemySpawns() {
      this.clearEnemySpawns();
      this.resetSpawnState();
      // --- Replace spawnGroup to allow cancelation ---
      this.spawnGroup = (count, spawnFn, delay = 150, groupType = null) => {
        let spawned = 0;
        const spawnNext = () => {
          if (spawned < count) {
            spawnFn(spawned);
            spawned++;
            if (groupType === "skeleton") {
              this._skeletonGroupTimeout = setTimeout(spawnNext, delay);
            } else if (groupType === "minion") {
              this._minionGroupTimeout = setTimeout(spawnNext, delay);
            } else {
              setTimeout(spawnNext, delay);
            }
          }
        };
        spawnNext();
      };
      const spawnLoop = (spawnFn, intervalKey, onSpawn) => {
        let interval = this.currentIntervals[intervalKey];
        const loop = () => {
          if (intervalKey === "skeleton") {
            this.spawnGroup(this.globalSkeletonCount, () => {
              this.spawnEnemy({
                speed: 60,
                imageSrc: 'https://i.postimg.cc/J7Z3cnmp/image-2025-05-16-103314524.png',
                size: 40,
                health: 30,
                coinReward: 8,
                troopName: 'Skeleton Army'
              });
            }, 120, "skeleton");
            if (this.globalSkeletonCount < 64) {
              this.globalSkeletonCount = Math.min(this.globalSkeletonCount * 2, 64);
            }
          } else if (intervalKey === "minion") {
            this.spawnGroup(this.globalMinionCount, () => {
              this.spawnEnemy({
                speed: 90,
                imageSrc: 'https://i.postimg.cc/PxZD43GC/image-2025-05-16-103616663.png',
                size: 35,
                health: 25,
                coinReward: 6,
                troopName: 'Minion Horde'
              });
            }, 100, "minion");
            if (this.globalMinionCount < 5) this.globalMinionCount += 1;
          } else {
            if (onSpawn) onSpawn();
            if (spawnFn) spawnFn();
          }
          interval = Math.max(interval * 0.97, this.MIN_SPAWN_INTERVALS[intervalKey] || 2000);
          this.currentIntervals[intervalKey] = interval;
          this.enemySpawnTimeouts.push(setTimeout(loop, interval));
        };
        this.enemySpawnTimeouts.push(setTimeout(loop, interval));
      };
      // ...existing code for all spawnLoop calls (giant, hog, skeleton, etc), but use this.spawnEnemy
      spawnLoop(() => this.spawnEnemy({
        speed: 25,
        imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
        size: 80,
        health: 300,
        coinReward: 120,
        troopName: 'Giant'
      }), "giant");
      spawnLoop(() => this.spawnEnemy({
        speed: 100,
        imageSrc: 'https://i.postimg.cc/4NxrrzmL/image-2025-05-16-101734632.png',
        size: 60,
        health: 100,
        coinReward: 80,
        troopName: 'Hog Rider'
      }), "hog");
      spawnLoop(null, "skeleton");
      spawnLoop(() => this.spawnEnemy({
        speed: 50,
        imageSrc: 'https://i.postimg.cc/zfyKLD0S/image-2025-05-16-103451709.png',
        size: 55,
        health: 120,
        coinReward: 90,
        troopName: 'Baby Dragon'
      }), "babyDragon");
      spawnLoop(null, "minion");
      spawnLoop(() => this.spawnEnemy({
        speed: 30,
        imageSrc: 'https://i.postimg.cc/28TZ2Lt7/image-2025-05-16-103738864.png',
        size: 65,
        health: 150,
        coinReward: 150,
        troopName: 'Balloon'
      }), "balloon");
      spawnLoop(() => this.spawnEnemy({
        speed: 120,
        imageSrc: 'https://i.postimg.cc/j2cLgBMS/image-2025-05-16-103917837.png',
        size: 50,
        health: 80,
        coinReward: 60,
        troopName: 'Bandit'
      }), "bandit");
      spawnLoop(() => this.spawnEnemy({
        speed: 15,
        imageSrc: 'https://i.postimg.cc/FsnKYyq8/image-2025-05-16-104224204.png',
        size: 90,
        health: 500,
        coinReward: 300,
        troopName: 'P.E.K.K.A'
      }), "pekka");
      // Witch with skeleton spawn logic
      spawnLoop(() => {
        let witchEnemyObj = this.spawnEnemy({
          speed: 35,
          imageSrc: 'https://i.postimg.cc/YqVprpLw/image-2025-05-16-104350767.png',
          size: 60,
          health: 150,
          coinReward: 100,
          troopName: 'Witch'
        });
        let witchSkeletonInterval;
        const spawnWitchSkeletons = () => {
          if (!witchEnemyObj || !witchEnemyObj.alive) return;
          let minDist = Infinity, closestIndex = 0;
          for (let i = 0; i < pathPoints.length - 1; i++) {
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
          const wx = witchEnemyObj.x, wy = witchEnemyObj.y;
          const offsets = [
            { dx: 30, dy: 0 }, { dx: -30, dy: 0 }, { dx: 0, dy: 30 }, { dx: 0, dy: -30 }
          ];
          offsets.forEach(offset => {
            this.spawnEnemy({
              imageSrc: 'https://i.postimg.cc/J7Z3cnmp/image-2025-05-16-103314524.png',
              speed: 60,
              size: 40,
              health: 30,
              coinReward: 8,
              customStart: { x: wx + offset.dx, y: wy + offset.dy },
              troopName: 'Skeleton Army',
              _witchPathIndex: closestIndex
            });
          });
          witchSkeletonInterval = setTimeout(spawnWitchSkeletons, 10000);
          this.enemySpawnTimeouts.push(witchSkeletonInterval);
        };
        spawnWitchSkeletons();
      }, "witch");
      spawnLoop(() => this.spawnEnemy({
        speed: 80,
        imageSrc: 'https://i.postimg.cc/YCYwtQw1/image-2025-05-16-104501378.png',
        size: 65,
        health: 180,
        coinReward: 110,
        troopName: 'Ram Rider'
      }), "ram");
      spawnLoop(() => this.spawnEnemy({
        speed: 20,
        imageSrc: 'https://i.postimg.cc/0QH4YNsk/image-2025-05-16-104632815.png',
        size: 100,
        health: 600,
        coinReward: 250,
        troopName: 'Lava Hound'
      }), "lava");
      spawnLoop(() => this.spawnEnemy({
        speed: 70,
        imageSrc: 'https://i.postimg.cc/CxL6QDxh/image-2025-05-16-104742624.png',
        size: 70,
        health: 200,
        coinReward: 90,
        isRoyalGhost: true,
        troopName: 'Royal Ghost'
      }), "ghost");
    }
    isValidTowerPlacement(x, y, radius, towerName = null) {
      // Prevent placing tower center on the path (within 10px of any path segment)
      const onPathThreshold = 10;
      for (let i = 0; i < pathPoints.length - 1; i++) {
        const p1 = pathPoints[i], p2 = pathPoints[i + 1];
        const dx = p2.x - p1.x, dy = p2.y - p1.y, len2 = dx * dx + dy * dy;
        let t = 0;
        if (len2 > 0) {
          t = ((x - p1.x) * dx + (y - p1.y) * dy) / len2;
          t = Math.max(0, Math.min(1, t));
        }
        const projX = p1.x + t * dx, projY = p1.y + t * dy;
        const d = Math.hypot(x - projX, y - projY);
        if (d < onPathThreshold) return false;
      }
      // Rage Beacon: only check for overlap with other Rage Beacons
      if (towerName === 'Rage Beacon') {
        for (const tower of this.placedTowers) {
          if (tower.name === 'Rage Beacon') {
            const distTowers = Math.hypot(x - tower.x, y - tower.y);
            if (distTowers < (radius + tower.radius - 20)) return false;
          }
        }
        return true;
      }
      // For all other towers: only check for overlap with other towers' centers (not radii), except Rage Beacon
      for (const tower of this.placedTowers) {
        if (tower.name === 'Rage Beacon') continue;
        // Only prevent placing if the centers are too close (e.g., 50px apart)
        const distTowers = Math.hypot(x - tower.x, y - tower.y);
        if (distTowers < 50) return false;
      }
      return true;
    }
    renderTowers() {
  // Clear existing towers and radius visuals
  document.querySelectorAll('.tower').forEach(el => el.remove());
  document.querySelectorAll('.tower-radius').forEach(el => el.remove());
  document.querySelectorAll('.tower-level').forEach(el => el.remove());
  document.querySelectorAll('.upgrade-btn').forEach(el => el.remove());

  this.placedTowers.forEach(tower => {
    // Create and display radius circle
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
    this.gameContainer.appendChild(radiusDiv);

    // Create tower image
    const img = document.createElement('img');
    img.src = tower.imageSrc;
    img.className = 'tower';
    img.style.position = 'absolute';
    img.style.left = `${tower.x - 25}px`;
    img.style.top = `${tower.y - 25}px`;
    img.style.width = '50px';
    img.style.height = '50px';
    img.title = tower.name;
    img.style.cursor = 'pointer';
    img.style.zIndex = 15;
    this.gameContainer.appendChild(img);

    // Level label
    const levelLabel = document.createElement('div');
    levelLabel.className = 'tower-level';
    levelLabel.textContent = `Lv ${tower.level || 0}`;
    levelLabel.style.position = 'absolute';
    levelLabel.style.left = `${tower.x - 12}px`;
    levelLabel.style.top = `${tower.y + 28}px`;
    levelLabel.style.color = '#fff';
    levelLabel.style.fontSize = '14px';
    levelLabel.style.fontWeight = 'bold';
    levelLabel.style.zIndex = 20;
    this.gameContainer.appendChild(levelLabel);

    // Upgrade button
    img.onclick = () => {
      const existing = document.getElementById('upgradeBtn');
      if (existing) {
        existing.remove();
        return;
      }

      const upgradeBtn = document.createElement('button');
      upgradeBtn.id = 'upgradeBtn';
      upgradeBtn.className = 'upgrade-btn';
      upgradeBtn.textContent = 'Upgrade';
      upgradeBtn.style.position = 'absolute';
      upgradeBtn.style.left = `${tower.x - 30}px`;
      upgradeBtn.style.top = `${tower.y + 45}px`;
      upgradeBtn.style.zIndex = 1000;
      upgradeBtn.style.padding = '4px 8px';
      upgradeBtn.style.fontSize = '12px';
      upgradeBtn.style.cursor = 'pointer';
      upgradeBtn.style.borderRadius = '5px';
      upgradeBtn.style.border = 'none';
      upgradeBtn.style.background = '#28a745';
      upgradeBtn.style.color = '#fff';

     upgradeBtn.onclick = (e) => {
  try {
    e.stopPropagation();
    if (tower.level < 5 && this.coins >= tower.cost) {
      this.spendCoins(tower.cost); // ⬅️ using `this` instead of `game`
      upgradeTower(tower, this);   // ⬅️ pass `this` as game
      this.renderTowers();
    } else {
      alert("Not enough coins or max level reached.");
    }
    upgradeBtn.remove();
  } catch (err) {
    console.error("Upgrade error:", err);
  }
};

      this.gameContainer.appendChild(upgradeBtn);
    };
  });
}

    showTowerMenu() {
      let menu = document.getElementById('towerMenu');
      if (menu) menu.remove();
      menu = document.createElement('div');
      menu.id = 'towerMenu';
      menu.style.position = 'absolute';
      menu.style.left = (this.gameContainer.offsetLeft + this.gameContainer.offsetWidth / 2 - 320) + 'px';
      menu.style.top = (this.gameContainer.offsetTop + this.gameContainer.offsetHeight + 16) + 'px';
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
    setupDragDrop() {
      this.gameContainer.ondragover = e => { e.preventDefault(); };
      this.gameContainer.ondrop = e => {
        e.preventDefault();
        const rect = this.gameContainer.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;
        const idx = e.dataTransfer.getData('towerIdx');
        const tower = towerData[idx];
        if (!tower) return;
        if (!this.isValidTowerPlacement(x, y, tower.radius, tower.name)) {
          alert('Cannot place tower here! Too close to path.');
          return;
        }
        if (this.coins < tower.cost) {
          alert('Not enough coins! (' + tower.cost + ' needed)');
          return;
        }
        this.spendCoins(tower.cost);
        this.placedTowers.push(new Tower(this, tower, x, y));
        this.renderTowers();
      };
    }
    upgrade() {
        if (coins < this.upgradeCost || this.level >= 5) return;
        coins -= this.upgradeCost;
        coinCount.textContent = coins;
        this.level++;
        this.label.textContent = `Lvl ${this.level}`;
        this.applyUpgrade();
      }

      applyUpgrade() {
        if (this.level === 1) this.attackSpeed /= 3;
        else if (this.level === 2) this.radius *= 3;
        else if (this.level === 3) this.multiShot = true;
        else if (this.level === 4) this.attackSpeed /= 1.5;
        else if (this.level === 5) {
          this.activateAbility();
          this.upgradeBtn.remove();
        }
      }
      activateAbility(tower) {
  if (tower.name !== 'Archer Tower' || tower.level < 5) return;

  // Temporarily enhance tower for 5 seconds
  const originalLastShot = tower.lastShot || 0;
  const enhancedDamage = ARCHER_DAMAGE * 10;
  const originalFireRate = 700;

  const abilityDuration = 5000;
  const startTime = performance.now();

  const shootLoop = () => {
    const now = performance.now();
    if (now - (tower.lastShot || 0) >= TOWER_ATTACK_INTERVAL / 5) {
      this.enemies.forEach(enemy => {
        if (!enemy.alive) return;
        const dx = tower.x - enemy.x;
        const dy = tower.y - enemy.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < tower.radius) {
          new Projectile(this, {
            fromX: tower.x,
            fromY: tower.y,
            toX: enemy.x,
            toY: enemy.y,
            imgSrc: ARCHER_ARROW_IMG,
            speed: 700,
            onHit: () => enemy.takeDamage(enhancedDamage)
          });
        }
      });
      tower.lastShot = now;
    }
    if (now - startTime < abilityDuration) {
      requestAnimationFrame(shootLoop);
    }
  };

  shootLoop();
}
    towerAttackLoop() {
      this.placedTowers.forEach(tower => {
        if (tower.name === 'Inferno Tower') {
          // --- Continuous Laser Logic ---
          let nearest = null, minDist = Infinity;
          this.enemies.forEach(enemy => {
            if (!enemy.alive) return;
            const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
            const dist = Math.sqrt(dx*dx + dy*dy);
            if (dist < tower.radius && dist < minDist) {
              nearest = enemy; minDist = dist;
            }
          });
          if (nearest) {
            let state = this.towerAttackState.get(tower) || { target: null, ramp: 0, lastTime: 0, laserEl: null, lastDamage: 0 };
            if (state.target !== nearest) {
              // New target, reset ramp and create laser
              state.target = nearest;
              state.ramp = 0;
              state.lastTime = performance.now();
              state.lastDamage = 0;
              if (state.laserEl) state.laserEl.remove();
              // Create laser DOM element
              const laser = document.createElement('div');
              laser.className = 'inferno-laser';
              laser.style.position = 'absolute';
              laser.style.pointerEvents = 'none';
              laser.style.zIndex = 30;
              laser.style.background = 'linear-gradient(90deg, #ff9800, #ffeb3b, #ff9800)';
              laser.style.boxShadow = '0 0 16px 4px #ff9800cc';
              laser.style.height = '6px';
              this.gameContainer.appendChild(laser);
              state.laserEl = laser;
            }
            // Update laser position and angle
            const dx = nearest.x - tower.x, dy = nearest.y - tower.y;
            const dist = Math.sqrt(dx*dx + dy*dy);
            const angle = Math.atan2(dy, dx) * 180 / Math.PI;
            state.laserEl.style.left = `${tower.x}px`;
            state.laserEl.style.top = `${tower.y - 3}px`;
            state.laserEl.style.width = `${dist}px`;
            state.laserEl.style.transform = `rotate(${angle}deg)`;
            state.laserEl.style.opacity = '1';
            // Damage ramping every 100ms
            const now = performance.now();
            if (!state.lastDamage || now - state.lastDamage > 100) {
              // Ramp up every 600ms
              if (now - state.lastTime > 600 && state.ramp < INFERNO_RAMP.length - 1) {
                state.ramp++;
                state.lastTime = now;
              }
              nearest.takeDamage(INFERNO_RAMP[state.ramp]);
              state.lastDamage = now;
            }
            this.towerAttackState.set(tower, state);
          } else {
            // No target, remove laser if exists
            let state = this.towerAttackState.get(tower);
            if (state && state.laserEl) {
              state.laserEl.style.opacity = '0';
              setTimeout(() => { if (state.laserEl) state.laserEl.remove(); }, 200);
            }
            this.towerAttackState.delete(tower);
          }
        } else if (tower.name === 'Archer Tower') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 700) {
            let nearest = null, minDist = Infinity;
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius && dist < minDist) {
                nearest = enemy; minDist = dist;
              }
            });
            if (nearest) {
              new Projectile(this, {
                fromX: tower.x, fromY: tower.y, toX: nearest.x, toY: nearest.y,
                imgSrc: ARCHER_ARROW_IMG, speed: 600,
                onHit: () => nearest.takeDamage(ARCHER_DAMAGE)
              });
              tower.lastShot = performance.now();
            }
          }
        } else if (tower.name === 'Bomb Tower') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 1200) {
            let nearest = null, minDist = Infinity;
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              if (AIR_TROOPS.includes(enemy.troopName)) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius && dist < minDist) {
                nearest = enemy; minDist = dist;
              }
            });
            if (nearest) {
              new Projectile(this, {
                fromX: tower.x, fromY: tower.y, toX: nearest.x, toY: nearest.y,
                imgSrc: BOMB_PROJECTILE_IMG, speed: 400,
                onHit: () => {
                  this.enemies.forEach(enemy2 => {
                    if (!enemy2.alive) return;
                    if (AIR_TROOPS.includes(enemy2.troopName)) return;
                    const dx = nearest.x - enemy2.x, dy = nearest.y - enemy2.y;
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
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 1200) {
            let nearest = null, minDist = Infinity;
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x;
              const dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius && dist < minDist) {
                nearest = enemy; minDist = dist;
              }
            });
            if (nearest) {
              new Projectile(this, {
                fromX: tower.x, fromY: tower.y, toX: nearest.x, toY: nearest.y,
                imgSrc: WIZARD_FIREBALL_IMG, speed: 400,
                onHit: () => nearest.takeDamage(60)
              });
              tower.lastShot = performance.now();
            }
          }
        } else if (tower.name === 'Magic Tower') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 1000) {
            let nearest = null, minDist = Infinity;
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius && dist < minDist) {
                nearest = enemy; minDist = dist;
              }
            });
            if (nearest) {
              const angle = Math.atan2(nearest.y - tower.y, nearest.x - tower.x);
              let pierced = 0;
              this.enemies.forEach(enemy => {
                if (!enemy.alive) return;
                const dx = enemy.x - tower.x, dy = enemy.y - tower.y;
                const dist = Math.sqrt(dx*dx + dy*dy);
                const enemyAngle = Math.atan2(dy, dx);
                const angleDiff = Math.abs(enemyAngle - angle);
                if (dist < tower.radius && angleDiff < 0.25 && pierced < 4) {
                  new Projectile(this, {
                    fromX: tower.x, fromY: tower.y, toX: enemy.x, toY: enemy.y,
                    imgSrc: MAGIC_BEAM_IMG, speed: 700,
                    onHit: () => enemy.takeDamage(30)
                  });
                  pierced++;
                }
              });
              tower.lastShot = performance.now();
            }
          }
        } else if (tower.name === 'Freeze Tower') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 2000) {
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius) {
                new Projectile(this, {
                  fromX: tower.x, fromY: tower.y, toX: enemy.x, toY: enemy.y,
                  imgSrc: FREEZE_ICE_IMG, speed: 500,
                  onHit: () => {
                    enemy.frozenUntil = performance.now() + 2000;
                    if (enemy._origSpeed === undefined) enemy._origSpeed = enemy.speed;
                    enemy.speed = enemy._origSpeed * 0.6;
                    setTimeout(() => {
                      // Only restore if not re-frozen
                      if (performance.now() > enemy.frozenUntil) {
                        enemy.speed = enemy._origSpeed;
                      }
                    }, 2000);
                  }
                });
              }
            });
            tower.lastShot = performance.now();
          }
        } else if (tower.name === 'Hunter Nest') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 1200) {
            let nearest = null, minDist = Infinity;
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius && dist < minDist) {
                nearest = enemy; minDist = dist;
              }
            });
            if (nearest) {
              const angle = Math.atan2(nearest.y - tower.y, nearest.x - tower.x);
              this.enemies.forEach(enemy => {
                if (!enemy.alive) return;
                const dx = enemy.x - tower.x, dy = enemy.y - tower.y;
                const dist = Math.sqrt(dx*dx + dy*dy);
                const enemyAngle = Math.atan2(dy, dx);
                const angleDiff = Math.abs(enemyAngle - angle);
                if (dist < tower.radius && angleDiff < 0.5) {
                  new Projectile(this, {
                    fromX: tower.x, fromY: tower.y, toX: enemy.x, toY: enemy.y,
                    imgSrc: HUNTER_CANNONBALL_IMG, speed: 600,
                    onHit: () => enemy.takeDamage(20)
                  });
                }
              });
              tower.lastShot = performance.now();
            }
          }
        } else if (tower.name === 'Lightning Obelisk') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 1500) {
            let targets = [];
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius) {
                targets.push({enemy, dist});
              }
            });
            targets.sort((a, b) => a.dist - b.dist);
            targets = targets.slice(0, 4);
            targets.forEach(t => {
              new Projectile(this, {
                fromX: tower.x, fromY: tower.y, toX: t.enemy.x, toY: t.enemy.y,
                imgSrc: LIGHTNING_BOLT_IMG, speed: 900,
                onHit: () => t.enemy.takeDamage(35)
              });
            });
            tower.lastShot = performance.now();
          }
        } else if (tower.name === 'Tesla Coil') {
          let lastShot = tower.lastShot || 0;
          if (performance.now() - lastShot > 900) {
            let nearest = null, minDist = Infinity;
            this.enemies.forEach(enemy => {
              if (!enemy.alive) return;
              const dx = tower.x - enemy.x, dy = tower.y - enemy.y;
              const dist = Math.sqrt(dx*dx + dy*dy);
              if (dist < tower.radius && dist < minDist) {
                nearest = enemy; minDist = dist;
              }
            });
            if (nearest) {
              new Projectile(this, {
                fromX: tower.x, fromY: tower.y, toX: nearest.x, toY: nearest.y,
                imgSrc: LIGHTNING_BOLT_IMG, speed: 800,
                onHit: () => nearest.takeDamage(40)
              });
              tower.lastShot = performance.now();
            }
          }
        } else if (tower.name === 'Rage Beacon') {
          // Support: increase attack speed of nearby towers
          this.placedTowers.forEach(otherTower => {
            if (otherTower === tower) return;
            const dx = tower.x - otherTower.x, dy = tower.y - otherTower.y;
            const dist = Math.sqrt(dx*dx + dy*dy);
            if (dist < tower.radius) {
              otherTower.rageUntil = performance.now() + 1000;
            }
          });
        }
      });
      setTimeout(() => this.towerAttackLoop(), TOWER_ATTACK_INTERVAL);
    }
  }
 function upgradeTower(tower, gameInstance) {
  if (tower.level >= 5) return;

  tower.level++;

  if (tower.name === 'Archer Tower') {
    switch (tower.level) {
      case 1:
        tower.fireRate = 700 / 3;
        break;
      case 2:
        tower.radius *= 3;
        break;
      case 3:
        tower.pierce = true;
        break;
      case 4:
        tower.multiShot = true;
        break;
      case 5:
        tower.abilityReady = true;
        break;
    }
  }

  gameInstance?.renderTowers?.(); // safe call
}

  // --- Start Game ---
  window.BarrierOpsGame = new Game();

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

      // Position points display at top left of map
      const pointsDisplay = document.getElementById('pointsDisplay');
      pointsDisplay.style.position = 'absolute';
      pointsDisplay.style.left = '16px';
      pointsDisplay.style.top = '16px';
      gameContainer.appendChild(pointsDisplay);
    }
    positionCoinDisplay();
    window.addEventListener('resize', positionCoinDisplay);
  });
</script>