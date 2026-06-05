<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>🐍 GitHub Snake Game – Eat Contributions! 🎯</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }

        body {
            background: linear-gradient(145deg, #0a0f1e 0%, #0c1222 100%);
            font-family: 'Segoe UI', 'Fira Code', 'Inter', monospace;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 2rem;
        }

        /* main card */
        .game-container {
            max-width: 1100px;
            width: 100%;
            background: rgba(10, 20, 30, 0.65);
            backdrop-filter: blur(2px);
            border-radius: 3rem;
            padding: 1.5rem;
            box-shadow: 0 25px 45px rgba(0, 0, 0, 0.5), inset 0 1px 1px rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(72, 187, 255, 0.2);
        }

        /* header area with profile vibe */
        .snake-header {
            display: flex;
            justify-content: space-between;
            align-items: baseline;
            flex-wrap: wrap;
            margin-bottom: 1.2rem;
            padding-bottom: 0.8rem;
            border-bottom: 1px dashed #2a3a55;
        }

        .title-snake {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .title-snake h2 {
            font-size: 1.7rem;
            background: linear-gradient(135deg, #b3f0ff, #7aa2f7);
            background-clip: text;
            -webkit-background-clip: text;
            color: transparent;
            letter-spacing: -0.5px;
        }

        .score-board {
            background: #07111e;
            padding: 0.3rem 1rem;
            border-radius: 60px;
            font-weight: bold;
            font-size: 1.3rem;
            color: #f0db4f;
            box-shadow: inset 0 0 4px #00a6ff40, 0 4px 8px rgba(0,0,0,0.3);
            font-family: monospace;
        }

        .reset-btn {
            background: #2a3a55;
            border: none;
            padding: 6px 18px;
            border-radius: 40px;
            font-weight: bold;
            color: white;
            cursor: pointer;
            transition: 0.2s;
            font-size: 0.9rem;
            box-shadow: 0 2px 8px black;
        }

        .reset-btn:hover {
            background: #4c7a9a;
            transform: scale(1.02);
            box-shadow: 0 0 10px #3b82f6;
        }

        /* contribution graph + game canvas area */
        .game-arena {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 1rem;
        }

        canvas {
            background-color: #0b1120;
            border-radius: 28px;
            box-shadow: 0 20px 35px -10px black, inset 0 0 0 2px rgba(70, 130, 200, 0.2);
            width: 100%;
            height: auto;
            cursor: pointer;
        }

        .legend-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 0.8rem;
            margin-top: 0.5rem;
            font-size: 0.75rem;
            color: #90a0c0;
        }

        .controls {
            display: flex;
            gap: 20px;
            background: #0f172ad9;
            padding: 8px 16px;
            border-radius: 40px;
        }

        .controls span {
            background: #1e2a3a;
            padding: 4px 12px;
            border-radius: 36px;
            font-weight: bold;
            font-family: monospace;
            letter-spacing: 1px;
        }

        .status {
            background: #00000066;
            padding: 6px 14px;
            border-radius: 28px;
            font-weight: bold;
            backdrop-filter: blur(4px);
        }

        .footer-note {
            margin-top: 1.2rem;
            text-align: center;
            font-size: 0.75rem;
            color: #6f85aa;
        }

        @media (max-width: 680px) {
            .game-container {
                padding: 0.9rem;
            }
            .controls span {
                font-size: 0.7rem;
                padding: 2px 8px;
            }
        }
    </style>
</head>
<body>
<div class="game-container">
    <div class="snake-header">
        <div class="title-snake">
            <h2>🐍 SNAKE × CONTRIBUTION GRID</h2>
            <div class="score-board">🍎 SCORE: <span id="scoreValue">0</span></div>
        </div>
        <button class="reset-btn" id="resetGameBtn">🔄 NEW GAME</button>
    </div>

    <div class="game-arena">
        <canvas id="snakeCanvas" width="800" height="480" style="width:100%; height:auto; max-width:800px; aspect-ratio:800/480"></canvas>
        <div class="legend-controls">
            <div class="controls">
                <span>⬆️ W / ↑</span>
                <span>⬇️ S / ↓</span>
                <span>⬅️ A / ←</span>
                <span>➡️ D / →</span>
            </div>
            <div class="status" id="gameStatusText">🎮 Use ARROWS or WASD</div>
        </div>
    </div>
    <div class="footer-note">
        💡 The snake eats <span style="color:#39ff14;">GREEN contribution cells</span> → each eaten cell adds +1 score & disappears!<br>
        🧱 Walls & self-collision end the game. Press NEW GAME to restart. Inspired by GitHub contributions heatmap!
    </div>
</div>

<script>
    (function(){
        // ---------- CONFIGURATION ----------
        const canvas = document.getElementById('snakeCanvas');
        const ctx = canvas.getContext('2d');

        // Grid dimensions: 40 columns x 24 rows -> each cell 20x20px (canvas 800x480)
        const COLS = 40;
        const ROWS = 24;
        const CELL_SIZE = 20;   // 800/40 =20, 480/24=20

        // Snake initial setup
        let snake = [
            {x: 20, y: 12},
            {x: 19, y: 12},
            {x: 18, y: 12},
            {x: 17, y: 12}
        ];
        let direction = 'RIGHT';    // current moving direction
        let nextDirection = 'RIGHT';
        let score = 0;
        let gameLoopInterval = null;
        let gameActive = true;
        
        // Contribution grid: simulates GitHub-style activity cells (green intensity)
        // We'll create an array [ROWS][COLS] of boolean (true = active "food" cell / contribution that can be eaten)
        // But to make it look like contribution graph, we generate random but "heatmap" style + some weekly pattern
        let contributions = Array(ROWS).fill().map(() => Array(COLS).fill(false));
        
        // To avoid too many food cells, we'll keep density ~35% but visually looks like contributions grid.
        // Each eaten contribution becomes false -> disappears.
        
        // ---- Helper: generate nice pseudo contribution pattern (like GitHub but randomized, green tiles) ----
        function generateContributions() {
            // reset contributions matrix
            for(let i=0; i<ROWS; i++) {
                for(let j=0; j<COLS; j++) {
                    // create organic look: higher probability in certain columns (like weekdays pattern)
                    // and some random streaks
                    let colPattern = (j % 7);  // 0-6 days
                    let baseProb = 0.28;
                    // add more density in middle columns? just for aesthetics
                    if(colPattern === 1 || colPattern === 3 || colPattern === 5) baseProb += 0.08;
                    if(j > 8 && j < 32) baseProb += 0.05;
                    // random factor
                    let rand = Math.random();
                    let isActive = rand < baseProb;
                    // also ensure edges have slight less to avoid immediate edge collision but fine
                    if(i === 0 || i === ROWS-1 || j === 0 || j === COLS-1) {
                        if(Math.random() > 0.5) isActive = false;
                    }
                    contributions[i][j] = isActive;
                }
            }
            // Ensure we have at least 15-20 food pieces for fun
            let foodCount = contributions.flat().filter(v=>v===true).length;
            if(foodCount < 18) {
                for(let attempt=0; attempt<40; attempt++) {
                    let rx = Math.floor(Math.random()*COLS);
                    let ry = Math.floor(Math.random()*ROWS);
                    if(!contributions[ry][rx]) contributions[ry][rx] = true;
                }
            }
            // Also make sure NO food spawns exactly on snake initial head/body positions
            for(let segment of snake) {
                if(contributions[segment.y] && contributions[segment.y][segment.x] === true) {
                    contributions[segment.y][segment.x] = false;
                }
            }
        }
        
        // --- drawing functions: cell styles like GitHub contribution tiles ---
        function drawContributionCell(x, y, active) {
            const cellX = x * CELL_SIZE;
            const cellY = y * CELL_SIZE;
            if(active) {
                // gradient green: fresh contribution - "edible"!
                const intensity = 0.7 + Math.sin(x * 0.5 + y) * 0.2;
                const greenBase = 80 + Math.floor(120 * intensity);
                ctx.fillStyle = `rgb(30, ${greenBase}, 40)`;
                ctx.fillRect(cellX, cellY, CELL_SIZE-0.5, CELL_SIZE-0.5);
                ctx.shadowBlur = 0;
                // add highlight dot
                ctx.fillStyle = '#a0ffa0';
                ctx.fillRect(cellX+2, cellY+2, 3, 3);
                ctx.fillStyle = '#50ff70';
                ctx.fillRect(cellX+4, cellY+4, 2, 2);
            } else {
                // empty contribution cell (dark, muted)
                ctx.fillStyle = '#16212e';
                ctx.fillRect(cellX, cellY, CELL_SIZE-0.5, CELL_SIZE-0.5);
                ctx.fillStyle = '#1d2c3f';
                ctx.fillRect(cellX+1, cellY+1, CELL_SIZE-2, CELL_SIZE-2);
            }
            // subtle grid border
            ctx.strokeStyle = '#2a3e55';
            ctx.lineWidth = 0.4;
            ctx.strokeRect(cellX, cellY, CELL_SIZE, CELL_SIZE);
        }
        
        function drawSnake() {
            for(let i=0; i<snake.length; i++) {
                const seg = snake[i];
                const xPos = seg.x * CELL_SIZE;
                const yPos = seg.y * CELL_SIZE;
                const isHead = i === 0;
                if(isHead) {
                    // snake head with glowing eyes
                    ctx.fillStyle = '#f5c542';
                    ctx.shadowBlur = 6;
                    ctx.shadowColor = '#ffb347';
                    ctx.fillRect(xPos+1, yPos+1, CELL_SIZE-2, CELL_SIZE-2);
                    ctx.fillStyle = '#000000';
                    ctx.fillRect(xPos+12, yPos+5, 3, 3);
                    ctx.fillRect(xPos+5, yPos+5, 3, 3);
                    ctx.fillStyle = 'white';
                    ctx.fillRect(xPos+12.5, yPos+5.5, 1.5, 1.5);
                    ctx.fillRect(xPos+5.5, yPos+5.5, 1.5, 1.5);
                } else {
                    ctx.fillStyle = '#4fb07f';
                    ctx.shadowBlur = 2;
                    ctx.fillRect(xPos+1, yPos+1, CELL_SIZE-2, CELL_SIZE-2);
                    ctx.fillStyle = '#2b7a4b';
                    ctx.fillRect(xPos+3, yPos+3, CELL_SIZE-6, CELL_SIZE-6);
                }
            }
            ctx.shadowBlur = 0;
        }
        
        function drawGame() {
            if(!ctx) return;
            // draw background and contribution grid
            for(let row=0; row<ROWS; row++) {
                for(let col=0; col<COLS; col++) {
                    drawContributionCell(col, row, contributions[row][col]);
                }
            }
            drawSnake();
            
            // game over overlay if not active
            if(!gameActive) {
                ctx.font = "bold 26px 'Segoe UI', 'Fira Code'";
                ctx.shadowBlur = 0;
                ctx.fillStyle = "#ffcc88";
                ctx.shadowColor = "black";
                ctx.fillText("💀 GAME OVER 💀", canvas.width/2-130, canvas.height/2-30);
                ctx.font = "16px monospace";
                ctx.fillStyle = "#bbddff";
                ctx.fillText("click 'NEW GAME' to restart", canvas.width/2-125, canvas.height/2+25);
            } else if(contributions.flat().every(v => v === false)) {
                // no more contributions left -> you WIN!
                ctx.font = "bold 28px 'Segoe UI'";
                ctx.fillStyle = "#f3ffbd";
                ctx.shadowBlur = 0;
                ctx.fillText("🏆 PERFECT CLEAN! YOU WIN 🏆", canvas.width/2-190, canvas.height/2);
                gameActive = false;
                if(gameLoopInterval) clearInterval(gameLoopInterval);
                gameLoopInterval = null;
                document.getElementById('gameStatusText').innerHTML = "✨ VICTORY! No contributions left ✨";
            }
        }
        
        // ---- CORE SNAKE MOVEMENT + EATING LOGIC: eats contributions ----
        function moveSnake() {
            if(!gameActive) return false;
            
            // commit direction
            direction = nextDirection;
            
            // compute new head
            let newHead = {...snake[0]};
            switch(direction) {
                case 'RIGHT': newHead.x += 1; break;
                case 'LEFT':  newHead.x -= 1; break;
                case 'UP':    newHead.y -= 1; break;
                case 'DOWN':  newHead.y += 1; break;
                default: break;
            }
            
            // check if new head collides with walls
            if(newHead.x < 0 || newHead.x >= COLS || newHead.y < 0 || newHead.y >= ROWS) {
                gameActive = false;
                stopGameLoop();
                document.getElementById('gameStatusText').innerHTML = "💥 WALL COLLISION! GAME OVER 💥";
                drawGame();
                return false;
            }
            
            // check if there is a contribution (food) at new head position
            let ateFood = false;
            if(contributions[newHead.y] && contributions[newHead.y][newHead.x] === true) {
                ateFood = true;
                // Eat the contribution: remove tile
                contributions[newHead.y][newHead.x] = false;
                score++;
                document.getElementById('scoreValue').innerText = score;
            }
            
            // perform snake movement
            if(ateFood) {
                // add new head, keep tail (grow)
                snake.unshift(newHead);
                // no pop -> length increases
            } else {
                // normal move: insert new head, remove tail
                snake.unshift(newHead);
                snake.pop();
            }
            
            // check self-collision (after movement, head can't overlap with body)
            const headPos = snake[0];
            for(let i=1; i<snake.length; i++) {
                if(snake[i].x === headPos.x && snake[i].y === headPos.y) {
                    gameActive = false;
                    stopGameLoop();
                    document.getElementById('gameStatusText').innerHTML = "🐍 SELF BITE! GAME OVER 🐍";
                    drawGame();
                    return false;
                }
            }
            
            // if all contributions are eaten, win condition will be shown on draw
            const anyFoodLeft = contributions.some(row => row.some(cell => cell === true));
            if(!anyFoodLeft) {
                gameActive = false;
                stopGameLoop();
                document.getElementById('gameStatusText').innerHTML = "🎉 VICTORY! You ate all contributions! 🎉";
                drawGame();
                return false;
            }
            
            return true;
        }
        
        function stepGame() {
            if(!gameActive) {
                return;
            }
            moveSnake();
            drawGame();
        }
        
        function startGameLoop() {
            if(gameLoopInterval) clearInterval(gameLoopInterval);
            gameLoopInterval = setInterval(() => {
                stepGame();
            }, 130);
        }
        
        function stopGameLoop() {
            if(gameLoopInterval) {
                clearInterval(gameLoopInterval);
                gameLoopInterval = null;
            }
        }
        
        // ---- reset everything (new game) ----
        function resetGame() {
            stopGameLoop();
            // reset snake
            snake = [
                {x: 20, y: 12},
                {x: 19, y: 12},
                {x: 18, y: 12},
                {x: 17, y: 12}
            ];
            direction = 'RIGHT';
            nextDirection = 'RIGHT';
            score = 0;
            gameActive = true;
            document.getElementById('scoreValue').innerText = "0";
            document.getElementById('gameStatusText').innerHTML = "🎮 Use ARROWS or WASD";
            // regenerate contributions (new fresh grid)
            generateContributions();
            // ensure no food on snake segments
            for(let seg of snake) {
                if(contributions[seg.y] && contributions[seg.y][seg.x] === true) {
                    contributions[seg.y][seg.x] = false;
                }
            }
            // also guarantee enough food remains after clearing snake positions
            let foodLeft = contributions.flat().filter(v=>v===true).length;
            if(foodLeft < 15) {
                for(let i=0; i<25; i++) {
                    let rx = Math.floor(Math.random()*COLS);
                    let ry = Math.floor(Math.random()*ROWS);
                    let occupied = snake.some(s => s.x===rx && s.y===ry);
                    if(!occupied && !contributions[ry][rx]) contributions[ry][rx] = true;
                }
            }
            drawGame();
            startGameLoop();
        }
        
        // ---- handle keyboard input ----
        function handleKey(e) {
            if(!gameActive) return;
            const key = e.key;
            // prevent default for arrow keys and WASD to avoid scrolling
            if (key === 'ArrowUp' || key === 'ArrowDown' || key === 'ArrowLeft' || key === 'ArrowRight' ||
                key === 'w' || key === 'W' || key === 's' || key === 'S' || key === 'a' || key === 'A' || key === 'd' || key === 'D') {
                e.preventDefault();
            }
            
            let newDir = null;
            if (key === 'ArrowRight' || key === 'd' || key === 'D') newDir = 'RIGHT';
            else if (key === 'ArrowLeft' || key === 'a' || key === 'A') newDir = 'LEFT';
            else if (key === 'ArrowUp' || key === 'w' || key === 'W') newDir = 'UP';
            else if (key === 'ArrowDown' || key === 's' || key === 'S') newDir = 'DOWN';
            
            if (newDir) {
                // prevent 180-degree reverse
                if ((newDir === 'RIGHT' && direction === 'LEFT') ||
                    (newDir === 'LEFT' && direction === 'RIGHT') ||
                    (newDir === 'UP' && direction === 'DOWN') ||
                    (newDir === 'DOWN' && direction === 'UP')) {
                    return;
                }
                nextDirection = newDir;
            }
        }
        
        // ---- ensure canvas and event binding, also resize awareness ----
        function init() {
            // set canvas actual dimensions
            canvas.width = COLS * CELL_SIZE;
            canvas.height = ROWS * CELL_SIZE;
            generateContributions();
            // clean snake start collisions
            for(let seg of snake) {
                if(contributions[seg.y] && contributions[seg.y][seg.x] === true) contributions[seg.y][seg.x] = false;
            }
            drawGame();
            startGameLoop();
            window.addEventListener('keydown', handleKey);
            document.getElementById('resetGameBtn').addEventListener('click', () => {
                resetGame();
            });
            // optional: touch / mobile simulated via basic alert? but we keep desktop friendly.
            // add 'Restart' also works.
        }
        
        // call init when page loads
        window.addEventListener('load', init);
    })();
</script>
</body>
</html>
