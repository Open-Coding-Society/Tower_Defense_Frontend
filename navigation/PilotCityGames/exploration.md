---
layout: bootstrap
title: Exploration
description: Exploration
permalink: /exploration
Author: Darsh
---


<!-- Bootstrap CSS for styling -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

<script type="module">
import { pythonURI, fetchOptions } from '{{ site.baseurl }}/assets/js/api/config.js';

function showPopup(message) {
  const popup = document.createElement("div");
  popup.textContent = message;
  Object.assign(popup.style, {
    position: "fixed", top: "50%", left: "50%", transform: "translate(-50%, -50%)",
    backgroundColor: "rgba(0, 0, 0, 0.8)", color: "white", padding: "20px",
    borderRadius: "8px", zIndex: "1000", textAlign: "center", fontSize: "18px"
  });
  document.body.appendChild(popup);
  setTimeout(() => document.body.removeChild(popup), 1000); // Popup lasts 1 second
}

async function updatePoints(points) {
  try {
    const response = await fetch(`${pythonURI}/api/points`, {
      ...fetchOptions,
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ points })
    });
    const data = await response.json();
    if (response.ok) {
      showPopup("You gained 10 points!");
    }
  } catch (error) {
    console.error('Error updating points:', error);
  }
}

document.addEventListener('DOMContentLoaded', function () {
  // Ensure the game-container and joystick-container exist
  const infoContainer = document.getElementById('info-container');

  if (!infoContainer) {
    console.error('Required containers are missing in the DOM.');
    return;
  }

  // Remove the game container entirely
  const gameContainer = document.getElementById('game-container');
  if (gameContainer) {
    gameContainer.remove();
  }

  // Remove joystick container
  const joystickContainer = document.getElementById('joystick-container');
  if (joystickContainer) {
    joystickContainer.remove();
  }

  // Create the canvas for the game
  const canvas = document.createElement('canvas');
  infoContainer.appendChild(canvas);
  canvas.width = 2000;
  canvas.height = 600;
  const ctx = canvas.getContext('2d');

  const player = { x: 100, y: 100, size: 15, speed: 2, dx: 0, dy: 0 };
  let discovered = new Set();
  let points = 0; // Initialize points
  const organelles = [
    { name: "Nucleus", x: 400, y: 300, r: 30, desc: "Controls cell activities and contains DNA." },
    { name: "Chloroplast", x: 600, y: 150, r: 25, desc: "Performs photosynthesis." },
    { name: "Vacuole", x: 200, y: 450, r: 35, desc: "Stores nutrients and waste products." },
    { name: "Cell Wall", x: 700, y: 500, r: 20, desc: "Provides structural support." },
    { name: "Cell Membrane", x: 100, y: 300, r: 20, desc: "Regulates what enters and leaves the cell." },
    { name: "Cytoplasm", x: 350, y: 100, r: 20, desc: "Gel-like substance where organelles reside." },
    { name: "Mitochondrion", x: 500, y: 400, r: 25, desc: "Produces energy for the cell." },
    { name: "Ribosome", x: 250, y: 200, r: 15, desc: "Synthesizes proteins." },
    { name: "Golgi Apparatus", x: 450, y: 500, r: 20, desc: "Modifies and packages proteins." },
    { name: "Endoplasmic Reticulum", x: 150, y: 100, r: 20, desc: "Transports materials within the cell." }
  ];

  // UI Elements
  const progressSpan = document.createElement('span');
  const progressDiv = document.createElement('div');
  progressDiv.classList.add('mb-3');
  progressDiv.innerHTML = "<strong>Organelles Discovered:</strong> ";
  progressDiv.appendChild(progressSpan);
  infoContainer.appendChild(progressDiv);

  const pointsDiv = document.createElement('div'); // Points display
  pointsDiv.classList.add('mb-3');
  pointsDiv.innerHTML = `<strong>Points:</strong> <span id="points-counter">0</span>`;
  infoContainer.appendChild(pointsDiv);

  const infoBox = document.createElement('div');
  infoContainer.appendChild(infoBox);

  // Functions for the game
  function drawPlayer() {
    ctx.fillStyle = "#3e8e41";
    ctx.beginPath();
    ctx.arc(player.x, player.y, player.size, 0, Math.PI * 2);
    ctx.fill();
  }

  function drawOrganelles() {
    organelles.forEach(o => {
      ctx.beginPath();
      ctx.arc(o.x, o.y, o.r, 0, Math.PI * 2);
      ctx.fillStyle = discovered.has(o.name) ? '#ffe600' : '#7ec850';
      ctx.fill();
      ctx.stroke();
      ctx.fillStyle = '#000';
      ctx.fillText(o.name, o.x - o.r, o.y - o.r - 5);
    });
  }

  function detectCollisions() {
    organelles.forEach(o => {
      const dist = Math.hypot(player.x - o.x, player.y - o.y);
      if (dist < player.size + o.r && !discovered.has(o.name)) {
        discovered.add(o.name);
        points += 10; // Add 10 points for each interaction
        document.getElementById('points-counter').textContent = points; // Update points display
        progressSpan.textContent = discovered.size;
        infoBox.style.display = 'block';
        infoBox.innerHTML = `<strong>${o.name}</strong><br>${o.desc}`;
        updatePoints(10); // Call the API to update points
      }
    });
  }

  function updatePlayer() {
    player.x += player.dx;
    player.y += player.dy;
    player.x = Math.max(player.size, Math.min(canvas.width - player.size, player.x));
    player.y = Math.max(player.size, Math.min(canvas.height - player.size, player.y));
  }

  function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawOrganelles();
    drawPlayer();
    detectCollisions();
    updatePlayer();
    requestAnimationFrame(gameLoop);
  }

  function resetPlayer() {
    player.x = 100;
    player.y = 100;
    player.dx = 0;
    player.dy = 0;
  }

  // Ensure WASD controls are the only input method
  document.addEventListener('keydown', (event) => {
    switch (event.key) {
      case 'w':
        player.dy = -player.speed;
        break;
      case 'a':
        player.dx = -player.speed;
        break;
      case 's':
        player.dy = player.speed;
        break;
      case 'd':
        player.dx = player.speed;
        break;
    }
  });

  document.addEventListener('keyup', (event) => {
    switch (event.key) {
      case 'w':
      case 's':
        player.dy = 0;
        break;
      case 'a':
      case 'd':
        player.dx = 0;
        break;
    }
  });

  // Start the game loop
  gameLoop();
});
</script>

<!-- Bootstrap JS and NippleJS for the joystick -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/nipplejs/0.9.0/nipplejs.min.js"></script>
<script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.5.2/dist/umd/popper.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>

<!-- Container for game and UI -->
<div class="container">
  <div class="row">
    <div class="col-md-4" id="info-container" style="margin-left: 20px;">
      <!-- Progress and organelle info will be shown here -->
    </div>
  </div>
</div>
