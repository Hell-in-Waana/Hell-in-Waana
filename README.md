<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Hell in Waana</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #111;
            font-family: Arial, sans-serif;
            touch-action: none;
        }
        #gameCanvas {
            width: 100%;
            max-width: 800px;
            height: 100%;
            max-height: 600px;
            border: 3px solid #fff;
            background: linear-gradient(#222, #333);
            image-rendering: pixelated;
            pointer-events: none;
        }
        #loginScreen, #menu, #instructions, #statusMenu, #gameOver {
            position: absolute;
            background: rgba(0, 0, 0, 0.9);
            color: #fff;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            width: 80%;
            max-width: 400px;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            display: none;
            font-size: 14px;
            max-height: 80vh;
            overflow-y: auto;
        }
        button {
            display: block;
            margin: 10px auto;
            padding: 10px 20px;
            font-size: 18px;
            background: #444;
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            touch-action: manipulation;
        }
        button:hover {
            background: #666;
        }
        select, input {
            margin: 10px;
            padding: 10px;
            font-size: 16px;
        }
        #joystick {
            position: fixed;
            bottom: 20px;
            left: 20px;
            width: 100px;
            height: 100px;
            background: rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            touch-action: none;
            display: none;
        }
        #joystickKnob {
            width: 30px;
            height: 30px;
            background: linear-gradient(#fff, #ccc);
            border-radius: 50%;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }
        #actionButtons {
            position: fixed;
            bottom: 20px;
            right: 20px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            display: none;
            z-index: 10;
        }
        .actionButton {
            padding: 10px 20px;
            font-size: 14px;
            background: #28a745;
            color: #fff;
            border: none;
            border-radius: 5px;
            touch-action: manipulation;
        }
        #menuButton {
            background: #dc3545;
        }
        #status {
            position: fixed;
            top: 10px;
            left: 10px;
            color: #fff;
            font-size: 14px;
            background: rgba(0, 0, 0, 0.7);
            padding: 5px 10px;
            border-radius: 5px;
        }
        #notification {
            position: fixed;
            top: 50px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 0, 0, 0.7);
            color: #fff;
            padding: 10px 20px;
            border-radius: 5px;
            display: none;
        }
    </style>
</head>
<body>
    <div id="loginScreen">
        <h2>Hell in Waana</h2>
        <input type="password" id="password" placeholder="Enter Password">
        <button onclick="checkPassword()">Login</button>
    </div>
    <canvas id="gameCanvas"></canvas>
    <div id="menu">
        <h2>Hell in Waana</h2>
        <p style="font-size: 10px;">Developed by Bazza</p>
        <select id="difficulty">
            <option value="Hard">Hard</option>
            <option value="Medium" selected>Medium</option>
            <option value="Lazy">Lazy</option>
        </select>
        <button onclick="startGame()">Start</button>
        <button onclick="showInstructions()">Instructions</button>
    </div>
    <div id="instructions">
        <h2>Instructions</h2>
        <p>Process samples in a laboratory to earn money:</p>
        <ul style="text-align: left;">
            <li>Move with WASD keys or onscreen joystick.</li>
            <li>Samples (RD, BH, OPF, TLO) spawn in receival area (top-left grid).</li>
            <li><b>Pick Up</b> a sample (green, unscanned, wet) from receival grid or ground.</li>
            <li><b>Interact</b> with scanner (green) to scan it (cyan).</li>
            <li><b>Interact</b> with oven (red) while holding a scanned, wet sample to dry (TLO: 60s, OPF: 90s, BH: 120s, RD: 150s).</li>
            <li>Dry, unscanned samples appear in trolley (bottom-right grid).</li>
            <li>Pick up, scan, and <b>Interact</b> with crusher (yellow) while holding a dry sample.</li>
            <li>After 5 crushed samples, a tray (orange, unread) spawns near crusher.</li>
            <li>Pick up unread trays and <b>Interact</b> with magazine (bottom-right, 3 slots).</li>
            <li>Trays fuse (15min), then become read and appear next to magazine.</li>
            <li><b>Interact</b> with bottom computer while holding a read tray to report and earn $1.</li>
            <li><b>Drop</b> places samples or trays on ground.</li>
            <li><b>Game Over</b>: If receival or trolley grid fills up.</li>
            <li><b>Interact</b> with top computer to view statuses, timers, and crusher progress (X/5).</li>
            <li>Stand in YouTube area (purple) for 30min to earn $10,000 (resets if you leave).</li>
            <li>Difficulty (Hard/Medium/Lazy) sets sample spawn rate: Hard (fast), Lazy (slow).</li>
        </ul>
        <button onclick="backToMenu()">Back</button>
    </div>
    <div id="statusMenu">
        <h2>Status</h2>
        <div id="statusContent"></div>
        <button onclick="closeStatusMenu()">Close</button>
    </div>
    <div id="gameOver">
        <h2>Game Over</h2>
        <p id="gameOverMessage"></p>
        <button onclick="restartGame()">Restart</button>
    </div>
    <div id="joystick">
        <div id="joystickKnob"></div>
    </div>
    <div id="actionButtons">
        <button class="actionButton" id="pickupButton">Pick Up</button>
        <button class="actionButton" id="dropButton">Drop</button>
        <button class="actionButton" id="interactButton">Interact</button>
        <button class="actionButton" id="menuButton">Menu</button>
    </div>
    <div id="status"></div>
    <div id="notification"></div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const loginScreen = document.getElementById('loginScreen');
        const menu = document.getElementById('menu');
        const instructions = document.getElementById('instructions');
        const statusMenu = document.getElementById('statusMenu');
        const statusContent = document.getElementById('statusContent');
        const gameOverScreen = document.getElementById('gameOver');
        const gameOverMessage = document.getElementById('gameOverMessage');
        const joystick = document.getElementById('joystick');
        const joystickKnob = document.getElementById('joystickKnob');
        const actionButtons = document.getElementById('actionButtons');
        const status = document.getElementById('status');
        const notification = document.getElementById('notification');

        // Canvas setup
        canvas.width = 800;
        canvas.height = 600;
        ctx.imageSmoothingEnabled = false;

        // Game state
        let gameState = 'login';
        let player = {
            x: 400,
            y: 300,
            size: 30,
            speed: 3,
            holding: null,
            score: 0
        };
        let samples = [];
        let receivalSamples = [];
        let ovenSamples = [];
        let trolleySamples = [];
        let trays = [];
        let magazine = Array(3).fill(null);
        const sampleTypes = ['RD', 'BH', 'OPF', 'TLO'];
        let currentTrayNumber = 101;
        let crusherCount = 0;
        let lastSampleTime = 0;
        let youtubeTimerStart = 0;
        let difficulty = 'Medium';

        // Drying times (in milliseconds)
        const dryingTimes = {
            TLO: 60 * 1000,
            OPF: 90 * 1000,
            BH: 120 * 1000,
            RD: 150 * 1000
        };

        // Laboratory objects
        const receivalArea = {
            x: 50,
            y: 50,
            width: 120,
            height: 120,
            grid: Array(9).fill(null),
            computer: { x: 260, y: 110, width: 60, height: 60 }
        };
        const scanner = { x: 260, y: 190, width: 60, height: 60 };
        const oven = { x: 600, y: 50, width: 100, height: 100 };
        const trolley = {
            x: 600,
            y: 300,
            width: 120,
            height: 120,
            grid: Array(9).fill(null)
        };
        const crusher = { x: 50, y: 450, width: 75, height: 75 };
        const magazineObj = { x: 600, y: 500, width: 130, height: 50 };
        const computer = { x: 400, y: 500, width: 60, height: 60 };
        const youtubeArea = { x: 50, y: 300, width: 50, height: 50 };

        // Joystick controls
        let joystickActive = false;
        let joystickCenter = { x: 50, y: 50 };
        let joystickTouchId = null;
        let moveDirection = { x: 0, y: 0 };

        joystick.addEventListener('touchstart', (e) => {
            e.preventDefault();
            const touch = e.touches[0];
            joystickTouchId = touch.identifier;
            joystickActive = true;
            updateJoystick(touch);
        });

        joystick.addEventListener('touchmove', (e) => {
            e.preventDefault();
            for (let touch of e.touches) {
                if (touch.identifier === joystickTouchId) {
                    updateJoystick(touch);
                    break;
                }
            }
        });

        joystick.addEventListener('touchend', (e) => {
            e.preventDefault();
            for (let touch of e.changedTouches) {
                if (touch.identifier === joystickTouchId) {
                    joystickActive = false;
                    moveDirection = { x: 0, y: 0 };
                    joystickKnob.style.transform = `translate(-50%, -50%)`;
                    break;
                }
            }
        });

        function updateJoystick(touch) {
            const rect = joystick.getBoundingClientRect();
            const x = touch.clientX - rect.left - joystickCenter.x;
            const y = touch.clientY - rect.top - joystickCenter.y;
            const distance = Math.sqrt(x * x + y * y);
            const maxDistance = 30;
            let dx = x;
            let dy = y;
            if (distance > maxDistance) {
                dx = (x / distance) * maxDistance;
                dy = (y / distance) * maxDistance;
            }
            joystickKnob.style.transform = `translate(${dx - 15}px, ${dy - 15}px)`;
            moveDirection = {
                x: (dx / maxDistance) * player.speed,
                y: (dy / maxDistance) * player.speed
            };
        }

        // Keyboard controls
        const keys = { w: false, a: false, s: false, d: false };
        window.addEventListener('keydown', (e) => {
            if (gameState !== 'playing') return;
            if (e.key === 'w') keys.w = true;
            if (e.key === 'a') keys.a = true;
            if (e.key === 's') keys.s = true;
            if (e.key === 'd') keys.d = true;
        });

        window.addEventListener('keyup', (e) => {
            if (e.key === 'w') keys.w = false;
            if (e.key === 'a') keys.a = false;
            if (e.key === 's') keys.s = false;
            if (e.key === 'd') keys.d = false;
        });

        // Button event handling
        const pickupButton = document.getElementById('pickupButton');
        const dropButton = document.getElementById('dropButton');
        const interactButton = document.getElementById('interactButton');
        const menuButton = document.getElementById('menuButton');

        function addButtonListeners(button, handler) {
            button.addEventListener('click', (e) => {
                e.preventDefault();
                console.log(`Button ${button.textContent} clicked`);
                handler();
            });
            button.addEventListener('touchstart', (e) => {
                e.preventDefault();
                console.log(`Button ${button.textContent} touched`);
                handler();
            });
        }

        addButtonListeners(pickupButton, pickup);
        addButtonListeners(dropButton, drop);
        addButtonListeners(interactButton, interact);
        addButtonListeners(menuButton, showMenu);

        // Dynamic canvas sizing
        function resizeCanvas() {
            const scale = Math.min(window.innerWidth / 800, window.innerHeight / 600);
            canvas.style.width = `${800 * scale}px`;
            canvas.style.height = `${600 * scale}px`;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // Password protection
        const correctPassword = "INTERTEK";
        function checkPassword() {
            const input = document.getElementById('password').value;
            if (input === correctPassword) {
                loginScreen.style.display = 'none';
                menu.style.display = 'block';
                gameState = 'menu';
            } else {
                alert('Incorrect password!');
            }
        }

        // Menu functions
        function startGame() {
            difficulty = document.getElementById('difficulty').value;
            gameState = 'playing';
            menu.style.display = 'none';
            joystick.style.display = 'block';
            actionButtons.style.display = 'flex';
            lastSampleTime = performance.now();
            spawnSample();
            updateStatus();
        }

        function showInstructions() {
            menu.style.display = 'none';
            instructions.style.display = 'block';
        }

        function backToMenu() {
            instructions.style.display = 'none';
            statusMenu.style.display = 'none';
            menu.style.display = 'block';
            gameState = 'menu';
        }

        function showStatusMenu() {
            gameState = 'status';
            statusMenu.style.display = 'block';
            joystick.style.display = 'none';
            actionButtons.style.display = 'none';
            let content = '<h3>Samples</h3>';
            receivalSamples.forEach(s => content += `<p>Sample ${s.type}: ${s.scanStatus}, ${s.wetStatus} (Receival)</p>`);
            ovenSamples.forEach(s => content += `<p>Sample ${s.type}: ${s.scanStatus}, ${s.wetStatus} (Oven, ${Math.ceil((s.dryTimer - performance.now()) / 1000)}s left)</p>`);
            trolleySamples.forEach(s => content += `<p>Sample ${s.type}: ${s.scanStatus}, ${s.wetStatus} (Trolley)</p>`);
            samples.forEach(s => content += `<p>Sample ${s.type}: ${s.scanStatus}, ${s.wetStatus} (Dropped)</p>`);
            content += '<h3>Trays</h3>';
            trays.forEach(t => content += `<p>Tray ${t.id}: ${t.status} (Ground)</p>`);
            magazine.forEach((t, i) => {
                if (t) content += `<p>Tray ${t.id}: ${t.status} (Magazine Slot ${i + 1}, ${Math.ceil((t.fuseTimer - performance.now()) / 1000)}s left)</p>`;
            });
            content += `<p>Crusher: ${crusherCount}/5 dry samples processed</p>`;
            statusContent.innerHTML = content || '<p>No samples or trays.</p>';
        }

        function closeStatusMenu() {
            gameState = 'playing';
            statusMenu.style.display = 'none';
            joystick.style.display = 'block';
            actionButtons.style.display = 'flex';
        }

        function endGame(message) {
            gameState = 'gameOver';
            gameOverMessage.textContent = message;
            gameOverScreen.style.display = 'block';
            joystick.style.display = 'none';
            actionButtons.style.display = 'none';
        }

        function restartGame() {
            gameState = 'menu';
            gameOverScreen.style.display = 'none';
            menu.style.display = 'block';
            player = { x: 400, y: 300, size: 30, speed: 3, holding: null, score: 0 };
            samples = [];
            receivalSamples = [];
            ovenSamples = [];
            trolleySamples = [];
            trays = [];
            magazine = Array(3).fill(null);
            currentTrayNumber = 101;
            crusherCount = 0;
            receivalArea.grid = Array(9).fill(null);
            trolley.grid = Array(9).fill(null);
            lastSampleTime = 0;
            youtubeTimerStart = 0;
            updateStatus();
        }

        function showMenu() {
            console.log('Menu button pressed');
            gameState = 'menu';
            menu.style.display = 'block';
            joystick.style.display = 'none';
            actionButtons.style.display = 'none';
        }

        // Notifications
        function showNotification(message) {
            notification.textContent = message;
            notification.style.display = 'block';
            setTimeout(() => notification.style.display = 'none', 3000);
        }

        // Game logic
        function spawnSample() {
            const emptySlots = receivalArea.grid.reduce((acc, val, idx) => (val === null ? [...acc, idx] : acc), []);
            if (emptySlots.length === 0) {
                endGame('Receival grid full!');
                return;
            }
            const slot = emptySlots[Math.floor(Math.random() * emptySlots.length)];
            const sample = {
                type: sampleTypes[Math.floor(Math.random() * sampleTypes.length)],
                scanStatus: 'unscanned',
                wetStatus: 'wet',
                slot: slot
            };
            receivalArea.grid[slot] = sample;
            receivalSamples.push(sample);
            showNotification(`Sample ${sample.type} spawned`);
        }

        function pickup() {
            console.log('Pickup button pressed', { playerX: player.x, playerY: player.y, holding: player.holding });
            if (player.holding) {
                console.log('Already holding an item');
                return;
            }

            for (let sample of receivalSamples) {
                const slot = sample.slot;
                const gridX = receivalArea.x + (slot % 3) * 40 + 20;
                const gridY = receivalArea.y + Math.floor(slot / 3) * 40 + 20;
                const distance = Math.hypot(player.x - gridX, player.y - gridY);
                console.log('Checking receival sample', { type: sample.type, slot, gridX, gridY, distance });
                if (distance < 40) {
                    player.holding = sample;
                    receivalArea.grid[slot] = null;
                    receivalSamples = receivalSamples.filter(s => s !== sample);
                    updateStatus();
                    console.log('Picked up sample', sample);
                    return;
                }
            }

            for (let sample of trolleySamples) {
                const slot = sample.slot;
                const gridX = trolley.x + (slot % 3) * 40 + 20;
                const gridY = trolley.y + Math.floor(slot / 3) * 40 + 20;
                const distance = Math.hypot(player.x - gridX, player.y - gridY);
                console.log('Checking trolley sample', { type: sample.type, slot, gridX, gridY, distance });
                if (distance < 40) {
                    player.holding = sample;
                    trolley.grid[slot] = null;
                    trolleySamples = trolleySamples.filter(s => s !== sample);
                    updateStatus();
                    console.log('Picked up sample', sample);
                    return;
                }
            }

            for (let sample of samples) {
                const distance = Math.hypot(player.x - sample.x, player.y - sample.y);
                console.log('Checking dropped sample', { type: sample.type, x: sample.x, y: sample.y, distance });
                if (distance < 40) {
                    player.holding = sample;
                    samples = samples.filter(s => s !== sample);
                    updateStatus();
                    console.log('Picked up sample', sample);
                    return;
                }
            }

            for (let tray of trays) {
                const distance = Math.hypot(player.x - tray.x, player.y - tray.y);
                console.log('Checking tray', { id: tray.id, x: tray.x, y: tray.y, distance });
                if (distance < 40) {
                    player.holding = tray;
                    trays = trays.filter(t => t !== tray);
                    updateStatus();
                    console.log('Picked up tray', tray);
                    return;
                }
            }
            console.log('No item picked up');
        }

        function drop() {
            console.log('Drop button pressed', { holding: player.holding });
            if (!player.holding) {
                console.log('Nothing to drop');
                return;
            }

            const dropped = player.holding;
            player.holding = null;

            if ('type' in dropped) {
                dropped.x = player.x;
                dropped.y = player.y;
                samples.push(dropped);
                console.log('Dropped sample on ground', dropped);
            } else {
                dropped.x = player.x;
                dropped.y = player.y;
                trays.push(dropped);
                console.log('Dropped tray on ground', dropped);
            }
            updateStatus();
        }

        function interact() {
            console.log('Interact button pressed', { playerX: player.x, playerY: player.y, holding: player.holding });

            // Show status menu only for top computer
            if (player.x >= receivalArea.computer.x && player.x <= receivalArea.computer.x + receivalArea.computer.width &&
                player.y >= receivalArea.computer.y && player.y <= receivalArea.computer.y + receivalArea.computer.height) {
                showStatusMenu();
                console.log('Showing status menu');
                return;
            }

            // Scan sample
            if (player.holding && 'type' in player.holding && player.holding.scanStatus === 'unscanned' &&
                player.x >= scanner.x && player.x <= scanner.x + scanner.width &&
                player.y >= scanner.y && player.y <= scanner.y + scanner.height) {
                player.holding.scanStatus = 'scanned';
                updateStatus();
                console.log('Scanned sample', player.holding);
                return;
            }

            // Place sample in oven
            if (player.holding && 'type' in player.holding && player.holding.scanStatus === 'scanned' && player.holding.wetStatus === 'wet' &&
                player.x >= oven.x && player.x <= oven.x + oven.width &&
                player.y >= oven.y && player.y <= oven.y + oven.height) {
                player.holding.wetStatus = 'drying';
                player.holding.dryTimer = performance.now() + dryingTimes[player.holding.type];
                player.holding.x = oven.x + 5 + Math.random() * 90;
                player.holding.y = oven.y + 5 + Math.random() * 90;
                ovenSamples.push(player.holding);
                player.holding = null;
                updateStatus();
                console.log('Placed sample in oven via interact');
                return;
            }

            // Place sample in crusher
            if (player.holding && 'type' in player.holding && player.holding.scanStatus === 'scanned' && player.holding.wetStatus === 'dry' &&
                player.x >= crusher.x && player.x <= crusher.x + crusher.width &&
                player.y >= crusher.y && player.y <= crusher.y + crusher.height) {
                crusherCount++;
                if (crusherCount >= 5) {
                    trays.push({
                        id: currentTrayNumber.toString(),
                        status: 'unread',
                        x: crusher.x + 60,
                        y: crusher.y
                    });
                    currentTrayNumber++;
                    crusherCount = 0;
                    showNotification(`Tray ${currentTrayNumber - 1} spawned`);
                    console.log('Tray spawned', { id: currentTrayNumber - 1 });
                }
                player.holding = null;
                updateStatus();
                console.log('Placed sample in crusher via interact');
                return;
            }

            // Place tray in magazine
            if (player.holding && 'id' in player.holding && player.holding.status === 'unread' &&
                player.x >= magazineObj.x && player.x <= magazineObj.x + magazineObj.width &&
                player.y >= magazineObj.y && player.y <= magazineObj.y + magazineObj.height) {
                const emptySlot = magazine.findIndex(slot => slot === null);
                if (emptySlot !== -1) {
                    player.holding.status = 'fusing';
                    player.holding.fuseTimer = performance.now() + 900 * 1000; // 15 minutes
                    magazine[emptySlot] = player.holding;
                    player.holding = null;
                    updateStatus();
                    console.log('Placed tray in magazine via interact');
                    return;
                }
                console.log('Magazine full');
                return;
            }

            // Report tray at bottom computer
            if (player.holding && 'id' in player.holding && player.holding.status === 'read' &&
                player.x >= computer.x && player.x <= computer.x + computer.width &&
                player.y >= computer.y && player.y <= computer.y + computer.height) {
                player.score += 1;
                player.holding = null;
                showNotification(`Tray reported! Score: $${player.score}`);
                updateStatus();
                console.log('Reported tray');
                return;
            }
            console.log('No interaction performed');
        }

        function updateStatus() {
            status.innerHTML = `Score: $${player.score}<br>Holding: ${player.holding ? ('type' in player.holding ? `Sample ${player.holding.type} (${player.holding.scanStatus}, ${player.holding.wetStatus})` : `Tray ${player.holding.id} (${player.holding.status})`) : 'None'}`;
        }

        function update() {
            if (gameState !== 'playing') return;

            // Keyboard movement
            let keyboardDirection = { x: 0, y: 0 };
            if (keys.w) keyboardDirection.y -= player.speed;
            if (keys.s) keyboardDirection.y += player.speed;
            if (keys.a) keyboardDirection.x -= player.speed;
            if (keys.d) keyboardDirection.x += player.speed;

            // Combine joystick and keyboard input
            let newX = player.x + (moveDirection.x + keyboardDirection.x);
            let newY = player.y + (moveDirection.y + keyboardDirection.y);
            if (newX >= 0 && newX <= canvas.width - player.size && newY >= 0 && newY <= canvas.height - player.size) {
                player.x = newX;
                player.y = newY;
            }

            const inYoutubeArea = player.x >= youtubeArea.x && player.x <= youtubeArea.x + youtubeArea.width &&
                                  player.y >= youtubeArea.y && player.y <= youtubeArea.y + youtubeArea.height;

            // YouTube timer
            const now = performance.now();
            if (inYoutubeArea) {
                if (youtubeTimerStart === 0) youtubeTimerStart = now;
                if (now - youtubeTimerStart >= 1800 * 1000) { // 30 minutes
                    player.score += 10000;
                    youtubeTimerStart = 0;
                    showNotification('YouTube bonus: $10,000 earned!');
                    updateStatus();
                }
            } else {
                youtubeTimerStart = 0;
            }

            // Sample spawning
            if (!inYoutubeArea) {
                let spawnInterval;
                if (difficulty === 'Hard') spawnInterval = (Math.random() * 30 + 15) * 1000; // 15–45s
                else if (difficulty === 'Medium') spawnInterval = (Math.random() * 120 + 30) * 1000; // 30–150s
                else spawnInterval = (Math.random() * 150 + 150) * 1000; // 150–300s
                if (now - lastSampleTime > spawnInterval) {
                    spawnSample();
                    lastSampleTime = now;
                }
            }

            const speedMultiplier = inYoutubeArea ? 0.5 : 1;

            ovenSamples = ovenSamples.filter(sample => {
                if (now > sample.dryTimer * speedMultiplier) {
                    const emptySlots = trolley.grid.reduce((acc, val, idx) => (val === null ? [...acc, idx] : acc), []);
                    if (emptySlots.length === 0) {
                        endGame('Trolley grid full!');
                        return false;
                    }
                    const slot = emptySlots[Math.floor(Math.random() * emptySlots.length)];
                    sample.wetStatus = 'dry';
                    sample.scanStatus = 'unscanned';
                    sample.slot = slot;
                    trolley.grid[slot] = sample;
                    trolleySamples.push(sample);
                    showNotification(`Sample ${sample.type} dried`);
                    return false;
                }
                return true;
            });

            magazine = magazine.map(tray => {
                if (tray && tray.status === 'fusing' && now > tray.fuseTimer * speedMultiplier) {
                    tray.status = 'read';
                    tray.x = magazineObj.x + 140;
                    tray.y = magazineObj.y;
                    trays.push(tray);
                    showNotification(`Tray ${tray.id} ready`);
                    return null;
                }
                return tray;
            });
        }

        function render() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = 'gray';
            ctx.fillRect(receivalArea.x, receivalArea.y, receivalArea.width, receivalArea.height);
            ctx.fillStyle = '#333';
            ctx.fillRect(receivalArea.x + 5, receivalArea.y + 5, receivalArea.width - 10, receivalArea.height - 10);
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 2;
            for (let i = 0; i < 3; i++) {
                for (let j = 0; j < 3; j++) {
                    ctx.strokeRect(receivalArea.x + j * 40, receivalArea.y + i * 40, 40, 40);
                    const sample = receivalArea.grid[i * 3 + j];
                    if (sample) {
                        ctx.fillStyle = sample.scanStatus === 'unscanned' ? '#0f0' : '#0ff';
                        ctx.beginPath();
                        ctx.arc(receivalArea.x + j * 40 + 20, receivalArea.y + i * 40 + 20, 10, 0, Math.PI * 2);
                        ctx.fill();
                        ctx.fillStyle = '#fff';
                        ctx.font = 'bold 12px Arial';
                        ctx.textAlign = 'center';
                        ctx.fillText(sample.type, receivalArea.x + j * 40 + 20, receivalArea.y + i * 40 + 25);
                    }
                }
            }
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('Receival', receivalArea.x + receivalArea.width / 2, receivalArea.y - 10);

            ctx.fillStyle = '#005';
            ctx.fillRect(receivalArea.computer.x, receivalArea.computer.y, 60, 40);
            ctx.fillStyle = '#00f';
            ctx.fillRect(receivalArea.computer.x + 4, receivalArea.computer.y + 4, 52, 32);
            ctx.fillStyle = '#88f';
            ctx.fillRect(receivalArea.computer.x + 8, receivalArea.computer.y + 8, 44, 24);
            ctx.fillStyle = '#005';
            ctx.fillRect(receivalArea.computer.x + 24, receivalArea.computer.y + 40, 12, 12);
            ctx.beginPath();
            ctx.moveTo(receivalArea.computer.x + 30, receivalArea.computer.y + 52);
            ctx.lineTo(receivalArea.computer.x + 20, receivalArea.computer.y + 60);
            ctx.lineTo(receivalArea.computer.x + 40, receivalArea.computer.y + 60);
            ctx.fillStyle = '#003';
            ctx.fill();
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Computer', receivalArea.computer.x + 30, receivalArea.computer.y - 10);

            ctx.fillStyle = '#080';
            ctx.fillRect(scanner.x, scanner.y, 60, 60);
            ctx.fillStyle = '#0f0';
            ctx.fillRect(scanner.x + 4, scanner.y + 4, 52, 52);
            ctx.fillStyle = '#8f8';
            ctx.fillRect(scanner.x + 8, scanner.y + 20, 44, 8);
            ctx.beginPath();
            ctx.arc(scanner.x + 50, scanner.y + 10, 5, 0, Math.PI * 2);
            ctx.fillStyle = '#f00';
            ctx.fill();
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Scanner', scanner.x + 30, scanner.y - 10);

            ctx.fillStyle = '#800';
            ctx.fillRect(oven.x, oven.y, oven.width, oven.height);
            ctx.fillStyle = '#f00';
            ctx.fillRect(oven.x + 5, oven.y + 5, 90, 90);
            ctx.fillStyle = '#f88';
            ctx.fillRect(oven.x + 10, oven.y + 10, 80, 80);
            ctx.beginPath();
            ctx.arc(oven.x + 80, oven.y + 20, 5, 0, Math.PI * 2);
            ctx.fillStyle = '#ff0';
            ctx.fill();
            ctx.fillStyle = '#f44';
            ctx.fillRect(oven.x + 15, oven.y + 15, 70, 70);
            ctx.beginPath();
            ctx.moveTo(oven.x + 10, oven.y + 90);
            ctx.lineTo(oven.x + 20, oven.y + 80);
            ctx.lineTo(oven.x + 30, oven.y + 90);
            ctx.fillStyle = '#600';
            ctx.fill();
            for (let sample of ovenSamples) {
                ctx.fillStyle = '#0ff';
                ctx.beginPath();
                ctx.arc(sample.x, sample.y, 8, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 10px Arial';
                ctx.fillText(sample.type, sample.x, sample.y + 13);
            }
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Oven', oven.x + 50, oven.y - 10);

            ctx.fillStyle = '#444';
            ctx.fillRect(trolley.x, trolley.y, trolley.width, trolley.height);
            ctx.fillStyle = '#222';
            ctx.fillRect(trolley.x + 5, trolley.y + 5, trolley.width - 10, trolley.height - 10);
            ctx.strokeStyle = '#fff';
            for (let i = 0; i < 3; i++) {
                for (let j = 0; j < 3; j++) {
                    ctx.strokeRect(trolley.x + j * 40, trolley.y + i * 40, 40, 40);
                    const sample = trolley.grid[i * 3 + j];
                    if (sample) {
                        ctx.fillStyle = '#0ff';
                        ctx.beginPath();
                        ctx.arc(trolley.x + j * 40 + 20, trolley.y + i * 40 + 20, 10, 0, Math.PI * 2);
                        ctx.fill();
                        ctx.fillStyle = '#fff';
                        ctx.font = 'bold 12px Arial';
                        ctx.fillText(sample.type, trolley.x + j * 40 + 20, trolley.y + i * 40 + 25);
                    }
                }
            }
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Trolley', trolley.x + trolley.width / 2, trolley.y - 10);

            ctx.fillStyle = '#880';
            ctx.fillRect(crusher.x, crusher.y, 75, 75);
            ctx.fillStyle = '#ff0';
            ctx.fillRect(crusher.x + 5, crusher.y + 5, 65, 65);
            ctx.beginPath();
            ctx.moveTo(crusher.x + 10, crusher.y + 5);
            ctx.lineTo(crusher.x + 35, crusher.y + 20);
            ctx.lineTo(crusher.x + 60, crusher.y + 5);
            ctx.fillStyle = '#cc0';
            ctx.fill();
            ctx.fillStyle = '#444';
            ctx.fillRect(crusher.x + 25, crusher.y + 50, 25, 15);
            ctx.beginPath();
            ctx.arc(crusher.x + 60, crusher.y + 60, 5, 0, Math.PI * 2);
            ctx.fillStyle = '#f00';
            ctx.fill();
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Crusher', crusher.x + 37.5, crusher.y - 10);

            ctx.fillStyle = '#666';
            ctx.fillRect(magazineObj.x, magazineObj.y, magazineObj.width, magazineObj.height);
            ctx.strokeStyle = '#fff';
            for (let i = 0; i < 3; i++) {
                ctx.strokeRect(magazineObj.x + i * 40, magazineObj.y, 40, 40);
                if (magazine[i]) {
                    ctx.fillStyle = magazine[i].status === 'fusing' ? '#ffa500' : '#00f';
                    ctx.fillRect(magazineObj.x + i * 40 + 5, magazineObj.y + 5, 30, 30);
                    ctx.fillStyle = '#fff';
                    ctx.font = 'bold 12px Arial';
                    ctx.fillText(magazine[i].id, magazineObj.x + i * 40 + 20, magazineObj.y + 25);
                }
            }
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Magazine', magazineObj.x + magazineObj.width / 2, magazineObj.y - 10);

            ctx.fillStyle = '#005';
            ctx.fillRect(computer.x, computer.y, 60, 40);
            ctx.fillStyle = '#00f';
            ctx.fillRect(computer.x + 4, computer.y + 4, 52, 32);
            ctx.fillStyle = '#88f';
            ctx.fillRect(computer.x + 8, computer.y + 8, 44, 24);
            ctx.fillStyle = '#005';
            ctx.fillRect(computer.x + 24, computer.y + 40, 12, 12);
            ctx.beginPath();
            ctx.moveTo(computer.x + 30, computer.y + 52);
            ctx.lineTo(computer.x + 20, computer.y + 60);
            ctx.lineTo(computer.x + 40, computer.y + 60);
            ctx.fillStyle = '#003';
            ctx.fill();
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 14px Arial';
            ctx.fillText('Computer', computer.x + 30, computer.y - 10);

            ctx.fillStyle = '#80f';
            ctx.fillRect(youtubeArea.x, youtubeArea.y, youtubeArea.width, youtubeArea.height);
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 12px Arial';
            ctx.fillText('YouTube', youtubeArea.x + youtubeArea.width / 2, youtubeArea.y - 10);

            for (let tray of trays) {
                ctx.fillStyle = '#ffa500';
                ctx.fillRect(tray.x, tray.y, 30, 15);
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 12px Arial';
                ctx.fillText(tray.id, tray.x + 15, tray.y + 25);
            }

            for (let sample of samples) {
                ctx.fillStyle = sample.scanStatus === 'unscanned' ? '#0f0' : '#0ff';
                ctx.beginPath();
                ctx.arc(sample.x, sample.y, 10, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 12px Arial';
                ctx.fillText(sample.type, sample.x, sample.y + 15);
            }

            ctx.fillStyle = '#fff';
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.size / 2, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = '#ccc';
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.size / 4, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 12px Arial';
            ctx.fillText('Player', player.x, player.y - player.size / 2 - 10);

            if (player.holding) {
                ctx.fillStyle = 'type' in player.holding ? (player.holding.scanStatus === 'unscanned' ? '#0f0' : '#0ff') : '#ffa500';
                ctx.beginPath();
                ctx.arc(player.x + 20, player.y, 10, 0, Math.PI * 2);
                ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 12px Arial';
                ctx.fillText('type' in player.holding ? player.holding.type : player.holding.id, player.x + 20, player.y + 15);
            }
        }

        function gameLoop(timestamp) {
            try {
                update();
                render();
            } catch (e) {
                console.error('Game loop error:', e);
                alert('Game error: ' + e.message + '. Please reload.');
            }
            requestAnimationFrame(gameLoop);
        }

        window.onload = () => {
            if (!canvas.getContext) {
                alert('Your browser does not support HTML5 Canvas. Please use a modern browser like Chrome or Safari.');
                return;
            }
            loginScreen.style.display = 'block';
            gameLoop();
        };
    </script>
</body>
</html>
