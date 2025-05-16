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
    width: 20px;
    height: 20px;
    border-radius: 50%;
    position: absolute;
    z-index: 5;
  }
</style>

<div id="gameContainer"></div>

<script>
  const pathPoints = [
    // in order; spawn at x100 y245
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

  pathPoints.forEach(point => { //comment this out to remove pathway points
    const dot = document.createElement("div");
    dot.className = "path-point";
    dot.style.left = `${point.x}px`;
    dot.style.top = `${point.y}px`;
    gameContainer.appendChild(dot);
  });

  spawnEnemy({
    speed: 50,
    imageSrc: 'https://i.postimg.cc/G2qWD0nP/image-2025-05-14-110409784.png', // giant
    size: 80
  });

  function spawnEnemy({ speed = 100, imageSrc = '', size = 20 }) {
    const enemy = document.createElement("img");
    enemy.src = imageSrc;
    enemy.className = "enemy";
    enemy.style.width = `${size}px`;
    enemy.style.height = `${size}px`;
    enemy.style.position = "absolute";
    enemy.style.zIndex = 5;
    gameContainer.appendChild(enemy);

    let currentIndex = 0;
    let start = pathPoints[currentIndex];
    let end = pathPoints[currentIndex + 1];
    let progress = 0;
    let lastTimestamp = null;
    let segmentDistance = Math.hypot(end.x - start.x, end.y - start.y);

    function moveEnemy(timestamp) {
      if (!lastTimestamp) lastTimestamp = timestamp;
      const deltaTime = (timestamp - lastTimestamp) / 1000;
      lastTimestamp = timestamp;

      progress += (speed * deltaTime) / segmentDistance;

      if (progress >= 1) {
        currentIndex++;
        if (currentIndex >= pathPoints.length - 1) {
          enemy.remove();
          return;
        }
        start = pathPoints[currentIndex];
        end = pathPoints[currentIndex + 1];
        progress = 0;
        segmentDistance = Math.hypot(end.x - start.x, end.y - start.y);
      }

      const x = start.x + (end.x - start.x) * progress;
      const y = start.y + (end.y - start.y) * progress;

      // 🧠 Offset by half the size to center the image on the path
      enemy.style.left = `${x - size / 2}px`;
      enemy.style.top = `${y - size / 2}px`;

      requestAnimationFrame(moveEnemy);
    }

    requestAnimationFrame(moveEnemy);
  }
  spawnEnemy();  // <-- call it after defining the function
</script>