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
    gameContainer.appendChild(img);

    const enemy = {
      el: img,
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
      lastTimestamp: null
    };

    enemy.segmentDistance = Math.hypot(
      enemy.end.x - enemy.start.x,
      enemy.end.y - enemy.start.y
    );

    enemies.push(enemy);
    requestAnimationFrame(ts => moveEnemy(enemy, ts));
  }

  function moveEnemy(enemy, timestamp) {
    if (!enemy.lastTimestamp) enemy.lastTimestamp = timestamp;
    const dt = (timestamp - enemy.lastTimestamp) / 1000;
    enemy.lastTimestamp = timestamp;

    enemy.progress += (enemy.speed * dt) / enemy.segmentDistance;

    if (enemy.progress >= 1) {
      enemy.currentIndex++;
      if (enemy.currentIndex >= pathPoints.length - 1) {
        enemy.el.remove();
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

    requestAnimationFrame(ts => moveEnemy(enemy, ts));
  }

  // 🎯 Example usage: spawn multiple types
  spawnEnemy({
    speed: 50,
    imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png', // giant
    size: 80,
    health: 300
  });

  spawnEnemy({
    speed: 100,
    imageSrc: 'https://i.postimg.cc/4NxrrzmL/image-2025-05-16-101734632.png', // HOGGG RIDA
    size: 80,
    health: 100
  });

  // 🕒 Optional: spawn every few seconds (for testing)
  setInterval(() => {
    spawnEnemy({
      speed: 75,
      imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png',
      size: 70,
      health: 200
    });
  }, 4000);
</script>