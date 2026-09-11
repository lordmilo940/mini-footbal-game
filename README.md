<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mini Football Game - Stable Single File</title>
    <style>
        body {
            margin: 0;
            background-color: #1a1a1a;
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
            gap: 40px;
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 15px;
            background: linear-gradient(145deg, #2a2a2a, #111111);
            padding: 12px 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            border: 1px solid #444;
            user-select: none;
        }
        canvas {
            border: 4px solid #fff;
            border-radius: 40px;
            box-shadow: 0px 15px 35px rgba(0,0,0,0.7);
            background-color: #2e7d32;
        }
        .controls-hint {
            margin-top: 15px;
            font-size: 14px;
            color: #bbb;
            text-align: center;
            line-height: 1.6;
        }
    </style>
</head>
<body>

    <div id="scoreboard">
        <span style="color: #3498db;">ARG (Player): <span id="playerScore">0</span></span>
        <span style="color: #e74c3c;">BRA (AI): <span id="aiScore">0</span></span>
        <span style="color: #f1c40f;">Time: <span id="timer">60</span>s</span>
    </div>

    <canvas id="gameCanvas" width="800" height="500"></canvas>

    <div class="controls-hint">
        Use <b>W, A, S, D</b> or <b>Arrow Keys</b> to move.<br>
        <i>All fixes applied in one bulletproof file!</i>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        const playerScoreEl = document.getElementById("playerScore");
        const aiScoreEl = document.getElementById("aiScore");
        const timerEl = document.getElementById("timer");

        let playerScore = 0;
        let aiScore = 0;
        let timeLeft = 60;
        let gameActive = true;

        const fieldRadius = 40; 

        const ball = { x: 400, y: 250, vx: 0, vy: 0, radius: 12, friction: 0.98 };
        const player = { x: 200, y: 250, radius: 22, speed: 4.5, color: "#3498db", angle: 0 };
        const ai = { 
            x: 600, 
            y: 250, 
            radius: 22, 
            speed: 3.2, 
            color: "#e74c3c", 
            angle: Math.PI,
            reactionDelay: 0, 
            targetY: 250
        };

        const goalHeight = 150;
        const goals = {
            left: { x: 0, y: (canvas.height - goalHeight) / 2, width: 15, height: goalHeight },
            right: { x: canvas.width - 15, y: (canvas.height - goalHeight) / 2, width: 15, height: goalHeight }
        };

        const keys = {};
        window.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
        window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

        const countdown = setInterval(() => {
            if (timeLeft > 0 && gameActive) {
                timeLeft--;
                timerEl.innerText = timeLeft;
            } else if (timeLeft === 0) {
                gameActive = false;
                clearInterval(countdown);
                alert(`Match Over! Final Score -> Player: ${playerScore} | AI: ${aiScore}`);
            }
        }, 1000);

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

        function handleCollision(entity) {
            let dx = ball.x - entity.x;
            let dy = ball.y - entity.y;
            let distance = Math.sqrt(dx * dx + dy * dy);
            let minDistance = ball.radius + entity.radius;

            if (distance  limit && limit > 0) {
                let nx = dx / dist;
                let ny = dy / dist;
                entity.x = cx + nx * limit;
                
                if (entity === ball) {
                    let dotProduct = ball.vx * nx + ball.vy * ny;
                    ball.vx = (ball.vx - 2 * dotProduct * nx) * 0.8;
                    ball.vy = (ball.vy - 2 * dotProduct * ny) * 0.8;
                }
            }
        }

        function update() {
            if (!gameActive) return;

            let moveX = 0;
            let moveY = 0;
            if (keys["w"] || keys["arrowup"]) moveY -= 1;
            if (keys["s"] || keys["arrowdown"]) moveY += 1;
            if (keys["a"] || keys["arrowleft"]) moveX -= 1;
            if (keys["d"] || keys["arrowright"]) moveX += 1;

            if (moveX !== 0 || moveY !== 0) {
                player.x += moveX * player.speed;
                player.y += moveY * player.speed;
                player.angle = Math.atan2(moveY, moveX);
            }

            ai.reactionDelay--;
            if (ai.reactionDelay <= 0) {
                let errorFactor = (Math.random() - 0.5) * 60;
                ai.targetY = ball.y + errorFactor;
                ai.reactionDelay = 10; 
            }

            let aiMoveY = 0;
            let aiMoveX = 0;

            if (ai.y < ai.targetY - 15) aiMoveY = 1;
            else if (ai.y > ai.targetY + 15) aiMoveY = -1;

            if (ball.x > canvas.width * 0.4) {
                if (ai.x > ball.x + 10) aiMoveX = -1;
                else if (ai.x < ball.x - 10) aiMoveX = 1;
            } else {
                if (ai.x < 620) aiMoveX = 1;
                if (ai.x > 640) aiMoveX = -1;
            }

            ai.x += aiMoveX * ai.speed;
            ai.y += aiMoveY * ai.speed;
            ai.angle = Math.atan2(aiMoveY, aiMoveX);

            [player, ai, ball].forEach(e => {
                e.x = Math.max(e.radius, Math.min(canvas.width - e.radius, e.x));
                e.y = Math.max(e.radius, Math.min(canvas.height - e.radius, e.y));
            });

            ball.x += ball.vx;
            ball.y += ball.vy;
            ball.vx *= ball.friction;
            ball.vy *= ball.friction;

            if (Math.abs(ball.vx) < 0.1) ball.vx = 0;
            if (Math.abs(ball.vy) < 0.1) ball.vy = 0;

            if (ball.x < fieldRadius && ball.y < fieldRadius) handleCornerCollision(ball, fieldRadius, fieldRadius);
            if (ball.x > canvas.width - fieldRadius && ball.y < fieldRadius) handleCornerCollision(ball, canvas.width - fieldRadius, fieldRadius);
            if (ball.x < fieldRadius && ball.y > canvas.height - fieldRadius) handleCornerCollision(ball, fieldRadius, canvas.height - fieldRadius);
            if (ball.x > canvas.width - fieldRadius && ball.y > canvas.height - fieldRadius) handleCornerCollision(ball, canvas.width - fieldRadius, canvas.height - fieldRadius);

            [player, ai].forEach(p => {
                if (p.x < fieldRadius && p.y < fieldRadius) handleCornerCollision(p, fieldRadius, fieldRadius);
                if (p.x > canvas.width - fieldRadius && p.y < fieldRadius) handleCornerCollision(p, canvas.width - fieldRadius, fieldRadius);
                if (p.x < fieldRadius && p.y > canvas.height - fieldRadius) handleCornerCollision(p, fieldRadius, canvas.height - fieldRadius);
                if (p.x > canvas.width - fieldRadius && p.y > canvas.height - fieldRadius) handleCornerCollision(p, canvas.width - fieldRadius, canvas.height - fieldRadius);
            });

            if (ball.y - ball.radius <= 0) { ball.y = ball.radius; ball.vy = -ball.vy * 0.8; }// --- REBOTES Y GOLES CORREGIDOS ---

// Rebote en Techo y Piso básicos
if (ball.y - ball.radius <= 0) { 
    ball.y = ball.radius; 
    ball.vy = -ball.vy * 0.8; 
}
if (ball.y + ball.radius >= canvas.height) { 
    ball.y = canvas.height - ball.radius; 
    ball.vy = -ball.vy * 0.8; 
}

// Colisión de pared Izquierda (Arco de la IA)
if (ball.x - ball.radius <= 0) {
    // Verifica si la pelota está a la altura del arco izquierdo
    if (ball.y > goals.left.y && ball.y < goals.left.y + goals.left.height) {
        // Si cruzó la línea de meta por completo
        if (ball.x + ball.radius < 0) { 
            aiScore++; 
            aiScoreEl.innerText = aiScore; 
            resetPositions(); 
        }
    } else {
        // Si no es gol, rebota normalmente contra la pared exterior
        ball.x = ball.radius; 
        ball.vx = -ball.vx * 0.8; 
    }
}

// Colisión de pared Derecha (Arco del Jugador)
if (ball.x + ball.radius >= canvas.width) {
    // Verifica si la pelota está a la altura del arco derecho
    if (ball.y > goals.right.y && ball.y < goals.right.y + goals.right.height) {
        // Si cruzó la línea de meta por completo
        if (ball.x - ball.radius > canvas.width) { 
            playerScore++; 
            playerScoreEl.innerText = playerScore; 
            resetPositions(); 
        }
    } else {
        // Si no es gol, rebota normalmente contra la pared exterior
        ball.x = canvas.width - ball.radius; 
        ball.vx = -ball.vx * 0.8; 
    }
}
