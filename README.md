<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mini Football Game</title>
    <style>
        body {
            margin: 0;
            background-color: #222;
            color: white;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        #scoreboard {
            display: flex;
            gap: 30px;
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 10px;
            background: rgba(0,0,0,0.5);
            padding: 10px 20px;
            border-radius: 8px;
        }
        canvas {
            border: 4px solid #fff;
            box-shadow: 0px 10px 30px rgba(0,0,0,0.5);
            background-color: #388e3c;
        }
        .controls-hint {
            margin-top: 15px;
            font-size: 14px;
            color: #aaa;
        }
    </style>
</head>
<body>

    <div id="scoreboard">
        <span style="color: #3498db;">Player: <span id="playerScore">0</span></span>
        <span style="color: #e74c3c;">AI: <span id="aiScore">0</span></span>
        <span style="color: #f1c40f;">Time: <span id="timer">60</span>s</span>
    </div>

    <canvas id="gameCanvas" width="800" height="500"></canvas>

    <div class="controls-hint">Use <b>W, A, S, D</b> or <b>Arrow Keys</b> to move and push the ball into the net!</div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        // Score Elements
        const playerScoreEl = document.getElementById("playerScore");
        const aiScoreEl = document.getElementById("aiScore");
        const timerEl = document.getElementById("timer");

        // Game State Variables
        let playerScore = 0;
        let aiScore = 0;
        let timeLeft = 60;
        let gameActive = true;

        // Timer Interval
        const countdown = setInterval(() => {
            if (timeLeft > 0 && gameActive) {
                timeLeft--;
                timerEl.innerText = timeLeft;
            } else if (timeLeft === 0) {
                gameActive = false;
                clearInterval(countdown);
                alert(`Game Over! Final Score - Player: ${playerScore} | AI: ${aiScore}`);
            }
        }, 1000);

        // Entities Setup
        const ball = { x: 400, y: 250, vx: 0, vy: 0, radius: 12, speed: 0.98 };
        const player = { x: 200, y: 250, radius: 20, speed: 4, color: "#3498db" };
        const ai = { x: 600, y: 250, radius: 20, speed: 2.8, color: "#e74c3c" };

        const goalHeight = 140;
        const goals = {
            left: { x: 0, y: (canvas.height - goalHeight) / 2, width: 15, height: goalHeight },
            right: { x: canvas.width - 15, y: (canvas.height - goalHeight) / 2, width: 15, height: goalHeight }
        };

        // Keyboard Controls Tracker
        const keys = {};
        window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
        window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

        // Reset positions after a goal
        function resetPositions() {
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.vx = 0;
            ball.vy = 0;
            player.x = 200;
            player.y = canvas.height / 2;
            ai.x = 600;
            ai.y = canvas.height / 2;
        }

        // Circular Collision Resolution (Player/AI pushing the ball)
        function handleCollision(entity) {
            let dx = ball.x - entity.x;
            let dy = ball.y - entity.y;
            let distance = Math.sqrt(dx * dx + dy * dy);
            let minDistance = ball.radius + entity.radius;

            if (distance < minDistance) {
                // Prevent sticking by pushing the ball outwards physically
                let overlap = minDistance - distance;
                let nx = dx / distance;
                let ny = dy / distance;

                ball.x += nx * overlap;
                ball.y += ny * overlap;

                // Transfer velocity
                ball.vx = nx * 6;
                ball.vy = ny * 6;
            }
        }

        // Game Logic updates
        function update() {
            if (!gameActive) return;

            // --- 1. Player Movement ---
            if (keys["w"] || keys["arrowup"]) player.y -= player.speed;
            if (keys["s"] || keys["arrowdown"]) player.y += player.speed;
            if (keys["a"] || keys["arrowleft"]) player.x -= player.speed;
            if (keys["d"] || keys["arrowright"]) player.x += player.speed;

            // --- 2. Simple Enemy AI Engine ---
            // AI follows the ball's Y axis and approaches the ball horizontally
            if (ai.y < ball.y - 10) ai.y += ai.speed;
            else if (ai.y > ball.y + 10) ai.y -= ai.speed;
            
            if (ai.x < ball.x && ball.x > canvas.width / 2) ai.x += ai.speed;
            else if (ai.x > 600) ai.x -= ai.speed; // Returns to guard defensive side

            // --- 3. Boundary Collisions (Keep Entities on field) ---
            [player, ai].forEach(e => {
                e.x = Math.max(e.radius, Math.min(canvas.width - e.radius, e.x));
                e.y = Math.max(e.radius, Math.min(canvas.height - e.radius, e.y));
            });

            // --- 4. Ball Physics & Friction ---
            ball.x += ball.vx;
            ball.y += ball.vy;
            ball.vx *= ball.speed; // Friction slows it down gradually
            ball.vy *= ball.speed;

            // Ball bounce off top and bottom walls
            if (ball.y - ball.radius <= 0 || ball.y + ball.radius >= canvas.height) {
                ball.vy = -ball.vy;
            }
            // Ball bounce off side walls (excluding goal sectors)
            if (ball.x - ball.radius <= 0) {
                if (ball.y > goals.left.y && ball.y < goals.left.y + goals.left.height) {
                    // Left Goal Scored (AI scores)
                    aiScore++;
                    aiScoreEl.innerText = aiScore;
                    resetPositions();
                } else {
                    ball.vx = -ball.vx;
                    ball.x = ball.radius;
                }
            }
            if (ball.x + ball.radius >= canvas.width) {
                if (ball.y > goals.right.y && ball.y < goals.right.y + goals.right.height) {
                    // Right Goal Scored (Player scores)
                    playerScore++;
                    playerScoreEl.innerText = playerScore;
                    resetPositions();
                } else {
                    ball.vx = -ball.vx;
                    ball.x = canvas.width - ball.radius;
                }
            }

            // --- 5. Collision Detections ---
            handleCollision(player);
            handleCollision(ai);
        }

        // Render Field graphics
        function draw() {
            // Clear canvas & field grass color lines
            ctx.fillStyle = "#2e7d32";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Pitch Lines Drawings
            ctx.strokeStyle = "rgba(255, 255, 255, 0.4)";
            ctx.lineWidth = 4;
            
            // Center line & center circle
            ctx.beginPath();
            ctx.moveTo(canvas.width / 2, 0);
            ctx.lineTo(canvas.width / 2, canvas.height);
            ctx.stroke();

            ctx.beginPath();
            ctx.arc(canvas.width / 2, canvas.height / 2, 70, 0, Math.PI * 2);
            ctx.stroke();

            // Render Goals
            ctx.fillStyle = "rgba(255,255,255,0.8)";
            ctx.fillRect(0, goals.left.y, goals.left.width, goals.left.height);
            ctx.fillRect(goals.right.x, goals.right.y, goals.right.width, goals.right.height);

            // Draw Player 1
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.lineWidth = 2;
            ctx.strokeStyle = '#fff';
            ctx.stroke();

            // Draw AI Opponent
            ctx.fillStyle = ai.color;
            ctx.beginPath();
            ctx.arc(ai.x, ai.y, ai.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.stroke();

            // Draw Football Ball
            ctx.fillStyle = "#ffffff";
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.strokeStyle = '#111';
            ctx.stroke();
        }

        // Core Game Engine Loop
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Fire engine start
        gameLoop();
    </script>
</body>
</html>
