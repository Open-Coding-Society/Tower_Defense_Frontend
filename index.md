---
layout: post
title: BarrierOps Tower Defense
description: Build and Upgrade your towers to defend your king!
Author: Lars, Darsh, Pradyun
---


<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Tower Defense Game</title>
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
      background-color: limegreen;
      border-radius: 50%;
      position: absolute;
      z-index: 5;
    }
  </style>
</head>
<body>
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

    pathPoints.forEach(point => {
      const dot = document.createElement("div");
      dot.className = "path-point";
      dot.style.left = `${point.x}px`;
      dot.style.top = `${point.y}px`;
      gameContainer.appendChild(dot);
    });

    function spawnEnemy(speed = 300) { // speed in pixels per second
  const enemy = document.createElement("div");
  enemy.className = "enemy";
  gameContainer.appendChild(enemy);

  let currentIndex = 0;
  let start = pathPoints[currentIndex];
  let end = pathPoints[currentIndex + 1];

  let progress = 0;
  let lastTimestamp = null;

  const segmentDistance = Math.hypot(end.x - start.x, end.y - start.y);

  function moveEnemy(timestamp) {
    if (!lastTimestamp) lastTimestamp = timestamp;
    const deltaTime = (timestamp - lastTimestamp) / 1000; // seconds
    lastTimestamp = timestamp;

    progress += (speed * deltaTime) / segmentDistance;

    if (progress >= 1) {
      currentIndex++;
      if (currentIndex >= pathPoints.length - 1) {
        enemy.remove(); // reached end
        return;
      }

      start = pathPoints[currentIndex];
      end = pathPoints[currentIndex + 1];
      progress = 0;
    }

    const x = start.x + (end.x - start.x) * progress;
    const y = start.y + (end.y - start.y) * progress;
    enemy.style.left = `${x}px`;
    enemy.style.top = `${y}px`;

    requestAnimationFrame(moveEnemy);
  }

  requestAnimationFrame(moveEnemy);
}
spawnEnemy();  // <-- call it after defining the function
  </script>
</body>
</html>