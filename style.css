// Game variables
let gameActive = false;
let score = 0;
let coinsCollected = 0;
let level = 'easy';
let selectedCar = 'red';
let controlMethod = 'keyboard';
let playerX = 50;
let roadLeft = 25;
let roadWidth = 50;
let aiCars = [];
let coins = [];
let obstacles = [];
let gameSpeed = 5;
let spawnRate = 120;
let frameCount = 0;
let recognition;
let handGestureInterval;
let lastGesture = '';
let hands = null;
let camera = null;
let gestureDetected = 'none';

// DOM elements
const startScreen = document.getElementById('startScreen');
const levelScreen = document.getElementById('levelScreen');
const carSelectionScreen = document.getElementById('carSelectionScreen');
const controlSelectionScreen = document.getElementById('controlSelectionScreen');
const gameScreen = document.getElementById('gameScreen');
const gameOverScreen = document.getElementById('gameOverScreen');
const road = document.getElementById('road');
const playerCar = document.getElementById('playerCar');
const scoreDisplay = document.getElementById('score');
const coinsDisplay = document.getElementById('coins');
const levelDisplay = document.getElementById('levelDisplay');
const cameraFeed = document.getElementById('cameraFeed');
const voiceStatus = document.getElementById('voiceStatus');
const gestureStatus = document.getElementById('gestureStatus');
const gestureDebug = document.getElementById('gestureDebug');
const finalScoreDisplay = document.getElementById('finalScore');
const bg1 = document.getElementById('bg1');
const bg2 = document.getElementById('bg2');
const bgMusic = document.getElementById('bgMusic');

// Initialize background patterns
bg1.style.backgroundImage = 'radial-gradient(circle, rgba(255,215,0,0.2) 1px, transparent 1px)';
bg1.style.backgroundSize = '30px 30px';
bg2.style.backgroundImage = 'linear-gradient(45deg, rgba(255,255,255,0.05) 25%, transparent 25%, transparent 75%, rgba(255,255,255,0.05) 75%, rgba(255,255,255,0.05)), linear-gradient(-45deg, rgba(255,255,255,0.05) 25%, transparent 25%, transparent 75%, rgba(255,255,255,0.05) 75%, rgba(255,255,255,0.05))';
bg2.style.backgroundSize = '60px 60px';

// Event listeners for buttons
document.getElementById('startBtn').addEventListener('click', () => {
    startScreen.style.display = 'none';
    levelScreen.style.display = 'flex';
});

document.querySelectorAll('.level-btn').forEach(btn => {
    btn.addEventListener('click', () => {
        level = btn.dataset.level;
        levelScreen.style.display = 'none';
        carSelectionScreen.style.display = 'flex';
    });
});

document.querySelectorAll('.car-btn').forEach(btn => {
    btn.addEventListener('click', () => {
        selectedCar = btn.dataset.car;
        carSelectionScreen.style.display = 'none';
        controlSelectionScreen.style.display = 'flex';
    });
});

document.querySelectorAll('.control-btn').forEach(btn => {
    btn.addEventListener('click', () => {
        controlMethod = btn.dataset.control;
        controlSelectionScreen.style.display = 'none';
        startGame();
    });
});

document.getElementById('restartBtn').addEventListener('click', () => {
    gameOverScreen.style.display = 'none';
    resetGame();
    startScreen.style.display = 'flex';
});

// Initialize game
function startGame() {
    gameScreen.style.display = 'block';
    gameActive = true;
    score = 0;
    coinsCollected = 0;
    frameCount = 0;
    aiCars = [];
    coins = [];
    obstacles = [];

    // Play background music
    bgMusic.volume = 0.3;
    bgMusic.play().catch(e => console.log("Autoplay prevented:", e));

    // Set level parameters
    switch(level) {
        case 'easy':
            gameSpeed = 5;
            spawnRate = 120;
            break;
        case 'medium':
            gameSpeed = 7;
            spawnRate = 90;
            break;
        case 'hard':
            gameSpeed = 10;
            spawnRate = 60;
            break;
    }

    levelDisplay.textContent = level.charAt(0).toUpperCase() + level.slice(1);

    // Set player car
    playerCar.style.backgroundImage = `url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 60"><rect x="10" y="20" width="80" height="30" rx="5" fill="<span class="math-inline">\{selectedCar \=\=\= 'red' ? '%23e74c3c' \: selectedCar \=\=\= 'blue' ? '%233498db' \: '%232ecc71'\}"/\><rect x\="20" y\="10" width\="60" height\="10" rx\="5" fill\="</span>{selectedCar === 'red' ? '%23e74c3c' : selectedCar === 'blue' ? '%233498db' : '%232ecc71'}"/><circle cx="25" cy="50" r="10" fill="%232c3e50"/><circle cx="75" cy="50" r="10" fill="%232c3e50"/></svg>')`;

    // Set up controls based on selection
    setupControls();

    // Start game loop
    requestAnimationFrame(gameLoop);
}

function setupControls() {
    // Remove previous event listeners
    document.removeEventListener('keydown', handleKeyDown);
    if (recognition) recognition.stop();
    if (handGestureInterval) clearInterval(handGestureInterval);
    if (hands) hands.close();
    if (camera) camera.stop();

    // Hide/show control UI elements
    cameraFeed.style.display = 'none';
    voiceStatus.style.display = 'none';
    gestureStatus.style.display = 'none';
    gestureDebug.style.display = 'none';

    switch(controlMethod) {
        case 'keyboard':
            document.addEventListener('keydown', handleKeyDown);
            break;
        case 'gesture':
            cameraFeed.style.display = 'block';
            gestureStatus.style.display = 'block';
            gestureDebug.style.display = 'block';
            initHandGestureControl();
            break;
        case 'voice':
            voiceStatus.style.display = 'block';
            initVoiceControl();
            break;
    }
}

function handleKeyDown(e) {
    if (!gameActive) return;

    const roadElement = document.getElementById('road');
    const roadRect = roadElement.getBoundingClientRect();
    const playerWidth = playerCar.offsetWidth;

    if (e.key === 'ArrowLeft') {
        playerX = Math.max(roadRect.left + playerWidth/2, playerX - 20);
        playerCar.style.left = `${playerX}px`;
    } else if (e.key === 'ArrowRight') {
        playerX = Math.min(roadRect.right - playerWidth/2, playerX + 20);
        playerCar.style.left = `${playerX}px`;
    }
}

function initHandGestureControl() {
    // Initialize MediaPipe Hands with higher confidence thresholds
    hands = new Hands({
        locateFile: (file) => {
            return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;
        }
    });

    hands.setOptions({
        maxNumHands: 1,
        modelComplexity: 1,
        minDetectionConfidence: 0.7,  // Increased from 0.5
        minTrackingConfidence: 0.7    // Increased from 0.5
    });

    hands.onResults((results) => {
        if (!gameActive) return;

        // Process hand landmarks
        if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
            const landmarks = results.multiHandLandmarks[0];
            const wrist = landmarks[0]; // Wrist base
            const thumbTip = landmarks[4]; // Thumb tip
            const indexTip = landmarks[8]; // Index finger tip

            // Calculate hand direction based on finger positions relative to wrist
            const handDirection = indexTip.x - wrist.x;

            // Adjust these thresholds as needed
            const leftThreshold = -0.15;  // More negative means more left movement required
            const rightThreshold = 0.15;  // More positive means more right movement required

            // Debug information
            gestureDebug.innerHTML = `
                Hand Direction: ${handDirection.toFixed(2)}<br>
                Wrist X: ${wrist.x.toFixed(2)}<br>
                Index X: ${indexTip.x.toFixed(2)}<br>
                Thumb X: ${thumbTip.x.toFixed(2)}
            `;

            if (handDirection < leftThreshold) {
                gestureDetected = 'left';
                gestureStatus.textContent = "Gesture: Moving Left";
            } else if (handDirection > rightThreshold) {
                gestureDetected = 'right';
                gestureStatus.textContent = "Gesture: Moving Right";
            } else {
                gestureDetected = 'none';
                gestureStatus.textContent = "Gesture: Center";
            }
        } else {
            gestureDetected = 'none';
            gestureStatus.textContent = "Gesture: No hand detected";
            gestureDebug.textContent = "No hand landmarks detected";
        }
    });

    // Start camera with higher resolution
    camera = new Camera(cameraFeed, {
        onFrame: async () => {
            if (!gameActive) return;
            await hands.send({ image: cameraFeed });
        },
        width: 320,  // Increased from 200
        height: 240  // Increased from 150
    });
    camera.start();

    // Update player position based on gestures - faster response
    handGestureInterval = setInterval(() => {
        if (!gameActive) return;

        const roadElement = document.getElementById('road');
        const roadRect = roadElement.getBoundingClientRect();
        const playerWidth = playerCar.offsetWidth;

        if (gestureDetected === 'left') {
            playerX = Math.max(roadRect.left + playerWidth/2, playerX - 25);  // Increased from 15
            playerCar.style.left = `${playerX}px`;
        } else if (gestureDetected === 'right') {
            playerX = Math.min(roadRect.right - playerWidth/2, playerX + 25); // Increased from 15
            playerCar.style.left = `${playerX}px`;
        }
    }, 50);  // Reduced interval from 100ms to 50ms for faster response
}

function initVoiceControl() {
    // Check if browser supports speech recognition
    if ('webkitSpeechRecognition' in window) {
        recognition = new webkitSpeechRecognition();
        recognition.continuous = true;
        recognition.interimResults = true;

        recognition.onstart = function() {
            voiceStatus.textContent = "Voice: Listening...";
        };

        recognition.onerror = function(event) {
            voiceStatus.textContent = "Voice: Error occurred";
        };

        recognition.onend = function() {
            if (gameActive) {
                voiceStatus.textContent = "Voice: Listening...";
                recognition.start();
            }
        };

        recognition.onresult = function(event) {
            const transcript = Array.from(event.results)
                .map(result => result[0])
                .map(result => result.transcript)
                .join('');

            if (transcript.includes('left') || transcript.includes('move left')) {
                movePlayer('left');
            } else if (transcript.includes('right') || transcript.includes('move right')) {
                movePlayer('right');
            }
        };

        recognition.start();
    } else {
        voiceStatus.textContent = "Voice: Not supported";
        // Fall back to keyboard controls
        controlMethod = 'keyboard';
        document.addEventListener('keydown', handleKeyDown);
    }
}

function movePlayer(direction) {
    if (!gameActive) return;

    const roadElement = document.getElementById('road');
    const roadRect = roadElement.getBoundingClientRect();
    const playerWidth = playerCar.offsetWidth;

    if (direction === 'left') {
        playerX = Math.max(roadRect.left + playerWidth/2, playerX - 20);
    } else if (direction === 'right') {
        playerX = Math.min(roadRect.right - playerWidth/2, playerX + 20);
    }

    playerCar.style.left = `${playerX}px`;
}

function gameLoop() {
    if (!gameActive) return;

    frameCount++;

    // Spawn AI cars, coins, and obstacles
    if (frameCount % spawnRate === 0) {
        spawnAICar();
    }

    if (frameCount % (spawnRate * 2) === 0) {
        spawnCoin();
    }

    if (frameCount % (spawnRate * 3) === 0 && level !== 'easy') {
        spawnObstacle();
    }

    // Move AI cars
    moveAICars();

    // Move coins
    moveCoins();

    // Move obstacles
    moveObstacles();

    // Check collisions
    checkCollisions();

    // Update score
    score += 1;
    scoreDisplay.textContent = score;

    // Increase difficulty over time
    if (frameCount % 500 === 0) {
        gameSpeed += 0.5;
        if (spawnRate > 30) spawnRate -= 5;
    }

    requestAnimationFrame(gameLoop);
}

function spawnAICar() {
    const roadRect = road.getBoundingClientRect();
    const laneWidth = roadRect.width / 3;
    const lane = Math.floor(Math.random() * 3);
    const x = roadRect.left + (lane * laneWidth) + (laneWidth / 2) - 25;

    const aiCar = document.createElement('div');
    aiCar.className = 'aiCar';
    const carColors = ['#e74c3c', '#3498db', '#f1c40f', '#9b59b6', '#1abc9c'];
    const randomColor = carColors[Math.floor(Math.random() * carColors.length)];
    aiCar.style.backgroundImage = `url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 60"><rect x="10" y="20" width="80" height="30" rx="5" fill="<span class="math-inline">\{encodeURIComponent\(randomColor\)\}"/\><rect x\="20" y\="10" width\="60" height\="10" rx\="5" fill\="</span>{encodeURIComponent(randomColor)}"/><circle cx="25" cy="50" r="10" fill="%232c3e50"/><circle cx="75" cy="50" r="10" fill="%232c3e50"/></svg>')`;
    aiCar.style.left = `${x}px`;
    aiCar.style.top = '-100px';
    gameScreen.appendChild(aiCar);

    aiCars.push({
        element: aiCar,
        x: x,
        y: -100,
        speed: gameSpeed + Math.random() * 2
    });
}

function spawnCoin() {
    const roadRect = road.getBoundingClientRect();
    const x = roadRect.left + Math.random() * (roadRect.width - 30);

    const coin = document.createElement('div');
    coin.className = 'coin';
    coin.style.left = `${x}px`;
    coin.style.top = '-30px';
    gameScreen.appendChild(coin);

    coins.push({
        element: coin,
        x: x,
        y: -30,
        speed: gameSpeed
    });
}

function spawnObstacle() {
    const roadRect = road.getBoundingClientRect();
    const x = roadRect.left + Math.random() * (roadRect.width - 60);

    const obstacle = document.createElement('div');
    obstacle.className = 'obstacle';
    obstacle.style.left = `${x}px`;
    obstacle.style.top = '-60px';
    gameScreen.appendChild(obstacle);

    obstacles.push({
        element: obstacle,
        x: x,
        y: -60,
        speed: gameSpeed
    });
}

function moveAICars() {
    for (let i = aiCars.length - 1; i >= 0; i--) {
        const car = aiCars[i];
        car.y += car.speed;
        car.element.style.top = `${car.y}px`;

        // Remove if off screen
        if (car.y > window.innerHeight) {
            gameScreen.removeChild(car.element);
            aiCars.splice(i, 1);
        }
    }
}

function moveCoins() {
    for (let i = coins.length - 1; i >= 0; i--) {
        const coin = coins[i];
        coin.y += coin.speed;
        coin.element.style.top = `${coin.y}px`;

        // Remove if off screen
        if (coin.y > window.innerHeight) {
            gameScreen.removeChild(coin.element);
            coins.splice(i, 1);
        }
    }
}

function moveObstacles() {
    for (let i = obstacles.length - 1; i >= 0; i--) {
        const obstacle = obstacles[i];
        obstacle.y += obstacle.speed;
        obstacle.element.style.top = `${obstacle.y}px`;

        // Remove if off screen
        if (obstacle.y > window.innerHeight) {
            gameScreen.removeChild(obstacle.element);
            obstacles.splice(i, 1);
        }
    }
}

function checkCollisions() {
    const playerRect = playerCar.getBoundingClientRect();

    // Check collision with AI cars
    for (let i = 0; i < aiCars.length; i++) {
        const carRect = aiCars[i].element.getBoundingClientRect();
        if (isColliding(playerRect, carRect)) {
            gameOver();
            return;
        }
    }

    // Check collision with obstacles
    for (let i = 0; i < obstacles.length; i++) {
        const obstacleRect = obstacles[i].element.getBoundingClientRect();
        if (isColliding(playerRect, obstacleRect)) {
            gameOver();
            return;
        }
    }

    // Check collision with coins
    for (let i = coins.length - 1; i >= 0; i--) {
        const coinRect = coins[i].element.getBoundingClientRect();
        if (isColliding(playerRect, coinRect)) {
            coinsCollected++;
            coinsDisplay.textContent = coinsCollected;
            score += 50;
            scoreDisplay.textContent = score;
            gameScreen.removeChild(coins[i].element);
            coins.splice(i, 1);
        }
    }
}

function isColliding(rect1, rect2) {
    return !(
        rect1.right < rect2.left || 
        rect1.left > rect2.right || 
        rect1.bottom < rect2.top || 
        rect1.top > rect2.bottom
    );
}

function gameOver() {
    gameActive = false;
    finalScoreDisplay.textContent = `Your Score: ${score}`;
    gameOverScreen.style.display = 'flex';

    // Stop background music
    bgMusic.pause();
    bgMusic.currentTime = 0;

    // Stop voice recognition if active
    if (recognition) recognition.stop();

    // Stop camera if active
    if (cameraFeed.srcObject) {
        cameraFeed.srcObject.getTracks().forEach(track => track.stop());
    }

    // Clear gesture interval if active
    if (handGestureInterval) clearInterval(handGestureInterval);

    // Stop MediaPipe hands
    if (hands) hands.close();
    if (camera) camera.stop();
}

function resetGame() {
    // Remove all AI cars, coins, and obstacles
    aiCars.forEach(car => gameScreen.removeChild(car.element));
    coins.forEach(coin => gameScreen.removeChild(coin.element));
    obstacles.forEach(obstacle => gameScreen.removeChild(obstacle.element));

    aiCars = [];
    coins = [];
    obstacles = [];

    // Reset player position
    playerX = window.innerWidth / 2;
    playerCar.style.left = `${playerX}px`;

    // Hide game screen
    gameScreen.style.display = 'none';
}

// Initialize player position
window.addEventListener('load', () => {
    playerX = window.innerWidth / 2;
    playerCar.style.left = `${playerX}px`;
});