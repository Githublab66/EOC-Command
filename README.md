<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>EOC Command - Game Intro</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }
    body {
      background-color: #05080d;
      color: #e2e8f0;
      font-family: "Courier New", Courier, monospace;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
    }
    canvas {
      border: 2px solid #1e293b;
      box-shadow: 0 0 30px rgba(239, 68, 68, 0.15);
      background-color: #0a0f18;
      max-width: 100%;
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="800" height="500"></canvas>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    // Game States: "INTRO" -> "PLAYING"
    let gameState = "INTRO";
    let lastTime = 0;

    // Intro Variables
    let radarAngle = 0;
    let blinkTimer = 0;
    let showPrompt = true;

    const briefingLines = [
      "PRIORITY ALERT // NATIONAL WEATHER & EMERGENCY NET",
      "--------------------------------------------------",
      "> SYSTEM: Hurricane 'Vanguard' upgraded to Category 4.",
      "> STATUS: Coastal evacuation routes at 85% capacity.",
      "> POWER: Sector 4 grid instability detected.",
      "> DIRECTIVE: Mayor has activated EOC Command.",
      " ",
      "Commander, the city is yours. Keep them alive."
    ];

    let currentLineIndex = 0;
    let currentCharIndex = 0;
    let typeTimer = 0;
    const typeSpeed = 0.025; // Seconds per character

    // Gameplay Placeholder Variables (Emergency Unit)
    const unit = {
      x: 150,
      y: 250,
      targetX: 150,
      targetY: 250,
      speed: 180
    };

    // Handle Keyboard & Click Inputs
    function startGame() {
      if (gameState === "INTRO") {
        // If text is still typing, finish it instantly on first press
        if (currentLineIndex < briefingLines.length) {
          currentLineIndex = briefingLines.length;
        } else {
          gameState = "PLAYING";
        }
      }
    }

    window.addEventListener("keydown", (e) => {
      if (e.key === "Enter" || e.key === " ") {
        startGame();
      }
    });

    canvas.addEventListener("click", (e) => {
      if (gameState === "INTRO") {
        startGame();
      } else if (gameState === "PLAYING") {
        // Point-and-click unit dispatch in gameplay state
        const rect = canvas.getBoundingClientRect();
        unit.targetX = e.clientX - rect.left;
        unit.targetY = e.clientY - rect.top;
      }
    });

    // Update Logic
    function update(dt) {
      if (gameState === "INTRO") {
        radarAngle += 1.5 * dt;
        blinkTimer += dt;
        if (blinkTimer >= 0.5) {
          showPrompt = !showPrompt;
          blinkTimer = 0;
        }

        // Typewriter effect
        if (currentLineIndex < briefingLines.length) {
          typeTimer += dt;
          if (typeTimer >= typeSpeed) {
            typeTimer = 0;
            currentCharIndex++;
            if (currentCharIndex > briefingLines[currentLineIndex].length) {
              currentLineIndex++;
              currentCharIndex = 0;
            }
          }
        }
      } else if (gameState === "PLAYING") {
        // Move response unit toward clicked target
        const dx = unit.targetX - unit.x;
        const dy = unit.targetY - unit.y;
        const distance = Math.hypot(dx, dy);

        if (distance > 2) {
          unit.x += (dx / distance) * unit.speed * dt;
          unit.y += (dy / distance) * unit.speed * dt;
        }
      }
    }

    // Draw Background Tactical Grid
    function drawGrid() {
      ctx.strokeStyle = "#111b2b";
      ctx.lineWidth = 1;
      for (let x = 0; x < canvas.width; x += 40) {
        ctx.beginPath();
        ctx.moveTo(x, 0);
        ctx.lineTo(x, canvas.height);
        ctx.stroke();
      }
      for (let y = 0; y < canvas.height; y += 40) {
        ctx.beginPath();
        ctx.moveTo(0, y);
        ctx.lineTo(canvas.width, y);
        ctx.stroke();
      }
    }

    // Render Intro Screen
    function drawIntro() {
      drawGrid();

      // Radar Graphic on Right Side
      const rx = 640;
      const ry = 230;
      const radius = 95;

      ctx.strokeStyle = "rgba(239, 68, 68, 0.35)";
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(rx, ry, radius, 0, Math.PI * 2);
      ctx.stroke();

      ctx.beginPath();
      ctx.arc(rx, ry, radius * 0.55, 0, Math.PI * 2);
      ctx.stroke();

      // Rotating Radar Sweep Line
      ctx.strokeStyle = "#ef4444";
      ctx.beginPath();
      ctx.moveTo(rx, ry);
      ctx.lineTo(rx + Math.cos(radarAngle) * radius, ry + Math.sin(radarAngle) * radius);
      ctx.stroke();

      // Title Header
      ctx.fillStyle = "#ef4444";
      ctx.font = "bold 34px Courier New";
      ctx.fillText("GRIDLOCK: EOC COMMAND", 50, 75);

      // Briefing Typewriter Text
      ctx.font = "15px Courier New";
      for (let i = 0; i <= currentLineIndex && i < briefingLines.length; i++) {
        const line =
          i === currentLineIndex
            ? briefingLines[i].substring(0, currentCharIndex)
            : briefingLines[i];

        ctx.fillStyle = i === 0 ? "#f87171" : i === 7 ? "#38bdf8" : "#cbd5e1";
        ctx.fillText(line, 50, 130 + i * 28);
      }

      // Blinking Action Prompt
      if (showPrompt) {
        ctx.fillStyle = "#facc15";
        ctx.font = "bold 18px Courier New";
        const promptText =
          currentLineIndex < briefingLines.length
            ? "[ CLICK OR PRESS ENTER TO SKIP BRIEFING ]"
            : "[ CLICK OR PRESS ENTER TO TAKE COMMAND ]";
        ctx.fillText(promptText, 50, 435);
      }
    }

    // Render Active Gameplay Screen
    function drawPlaying() {
      drawGrid();

      // Top HUD Bar
      ctx.fillStyle = "#0f172a";
      ctx.fillRect(0, 0, canvas.width, 45);
      ctx.fillStyle = "#38bdf8";
      ctx.font = "bold 16px Courier New";
      ctx.fillText("EOC ACTIVE // CLICK ANYWHERE ON MAP TO DISPATCH RESPONSE UNIT", 20, 28);

      // Target Waypoint Marker
      ctx.strokeStyle = "#facc15";
      ctx.beginPath();
      ctx.arc(unit.targetX, unit.targetY, 8, 0, Math.PI * 2);
      ctx.stroke();

      // Response Unit Icon
      ctx.fillStyle = "#ef4444";
      ctx.fillRect(unit.x - 12, unit.y - 12, 24, 24);
      ctx.fillStyle = "#ffffff";
      ctx.font = "bold 11px Courier New";
      ctx.fillText("UNIT 1", unit.x - 20, unit.y - 18);
    }

    // Main Game Loop
    function gameLoop(timestamp) {
      const dt = (timestamp - lastTime) / 1000;
      lastTime = timestamp;

      update(dt);

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      if (gameState === "INTRO") {
        drawIntro();
      } else {
        drawPlaying();
      }

      requestAnimationFrame(gameLoop);
    }

    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
# EOC-Command