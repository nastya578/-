<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ловец Яблок</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Arial', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
            padding: 20px;
        }
        
        .game-container {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            text-align: center;
            max-width: 500px;
            width: 100%;
        }
        
        h1 {
            color: #2c3e50;
            margin-bottom: 10px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
        }
        
        .stats {
            display: flex;
            justify-content: space-around;
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
            margin: 20px 0;
            font-size: 1.2em;
            font-weight: bold;
        }
        
        .stat-item {
            color: #2c3e50;
        }
        
        .stat-value {
            color: #e74c3c;
            font-size: 1.3em;
        }
        
        .game-area {
            position: relative;
            width: 100%;
            height: 400px;
            background: #ecf0f1;
            border-radius: 10px;
            overflow: hidden;
            border: 3px solid #bdc3c7;
            margin-bottom: 20px;
        }
        
        #basket {
            position: absolute;
            bottom: 10px;
            width: 100px;
            height: 50px;
            background: linear-gradient(45deg, #3498db, #2980b9);
            border-radius: 10px 10px 0 0;
            transition: all 0.1s;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }
        
        .apple {
            position: absolute;
            width: 40px;
            height: 40px;
            background: radial-gradient(circle at 30% 30%, #ff3838, #c70000);
            border-radius: 50%;
            box-shadow: inset -5px -5px 10px rgba(0, 0, 0, 0.3);
            animation: apple-glow 1.5s infinite alternate;
        }
        
        @keyframes apple-glow {
            from { box-shadow: 0 0 10px #ff3838; }
            to { box-shadow: 0 0 20px #ff3838; }
        }
        
        .controls {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 20px;
        }
        
        button {
            padding: 12px 30px;
            font-size: 1.1em;
            font-weight: bold;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
        }
        
        #startBtn {
            background: linear-gradient(45deg, #2ecc71, #27ae60);
            color: white;
        }
        
        #resetBtn {
            background: linear-gradient(45deg, #e74c3c, #c0392b);
            color: white;
        }
        
        button:hover {
            transform: translateY(-3px);
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
        }
        
        button:active {
            transform: translateY(1px);
        }
        
        .instructions {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
            margin-top: 20px;
            font-size: 0.95em;
            color: #2c3e50;
            line-height: 1.5;
        }
        
        .level-indicator {
            margin-top: 15px;
            font-weight: bold;
            color: #9b59b6;
        }
        
        @media (max-width: 600px) {
            .game-area {
                height: 300px;
            }
            
            .controls {
                flex-direction: column;
                gap: 10px;
            }
            
            button {
                width: 100%;
            }
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>🍎 Ловец Яблок</h1>
        
        <div class="stats">
            <div class="stat-item">
                Счёт: <span id="score" class="stat-value">0</span>
            </div>
            <div class="stat-item">
                Время: <span id="timer" class="stat-value">60</span>с
            </div>
            <div class="stat-item">
                Пропущено: <span id="missed" class="stat-value">0</span>
            </div>
        </div>
        
        <div class="game-area" id="gameArea">
            <div id="basket"></div>
        </div>
        
        <div class="level-indicator">
            Уровень: <span id="level">1</span>
        </div>
        
        <div class="controls">
            <button id="startBtn">Старт</button>
            <button id="resetBtn">Новая игра</button>
        </div>
        
        <div class="instructions">
            <p>🎯 <strong>Цель игры:</strong> Ловить падающие яблоки корзиной!</p>
            <p>🕹️ <strong>Управление:</strong> Используй мышку или двигай пальцем по экрану</p>
            <p>⚠️ <strong>Правила:</strong> Не пропусти больше 10 яблок. Каждые 20 очков повышается уровень!</p>
        </div>
    </div>

    <script>
        const gameArea = document.getElementById('gameArea');
        const basket = document.getElementById('basket');
        const scoreElement = document.getElementById('score');
        const timerElement = document.getElementById('timer');
        const missedElement = document.getElementById('missed');
        const levelElement = document.getElementById('level');
        const startBtn = document.getElementById('startBtn');
        const resetBtn = document.getElementById('resetBtn');
        
        let score = 0;
        let missed = 0;
        let timeLeft = 60;
        let gameActive = false;
        let gameTimer;
        let appleInterval;
        let level = 1;
        let basketSpeed = 1;
        
        // Начальная позиция корзины
        basket.style.left = (gameArea.offsetWidth / 2 - basket.offsetWidth / 2) + 'px';
        
        // Управление корзиной
        gameArea.addEventListener('mousemove', moveBasket);
        gameArea.addEventListener('touchmove', function(e) {
            e.preventDefault();
            const touch = e.touches[0];
            const rect = gameArea.getBoundingClientRect();
            moveBasket({
                clientX: touch.clientX,
                clientY: touch.clientY,
                target: gameArea
            });
        });
        
        function moveBasket(e) {
            if (!gameActive) return;
            
            const gameAreaRect = gameArea.getBoundingClientRect();
            let newX = e.clientX - gameAreaRect.left - basket.offsetWidth / 2;
            
            // Ограничение движения в пределах игровой области
            newX = Math.max(0, Math.min(newX, gameArea.offsetWidth - basket.offsetWidth));
            
            basket.style.left = newX + 'px';
        }
        
        // Создание яблок
        function createApple() {
            if (!gameActive) return;
            
            const apple = document.createElement('div');
            apple.className = 'apple';
            
            // Случайная позиция по X
            const maxX = gameArea.offsetWidth - 40;
            apple.style.left = Math.random() * maxX + 'px';
            apple.style.top = '-40px';
            
            gameArea.appendChild(apple);
            
            // Скорость падения в зависимости от уровня
            const fallSpeed = 2 + level * 0.5;
            
            // Анимация падения
            function fall() {
                if (!gameActive) {
                    apple.remove();
                    return;
                }
                
                const currentTop = parseInt(apple.style.top);
                const newTop = currentTop + fallSpeed;
                apple.style.top = newTop + 'px';
                
                // Проверка столкновения с корзиной
                const appleRect = apple.getBoundingClientRect();
                const basketRect = basket.getBoundingClientRect();
                
                if (appleRect.bottom >= basketRect.top &&
                    appleRect.top <= basketRect.bottom &&
                    appleRect.right >= basketRect.left &&
                    appleRect.left <= basketRect.right) {
                    
                    // Яблоко поймано
                    apple.remove();
                    score += level; // Больше очков на высоких уровнях
                    scoreElement.textContent = score;
                    
                    // Звуковой эффект (просто вибрация, если поддерживается)
                    if (navigator.vibrate) {
                        navigator.vibrate(50);
                    }
                    
                    // Проверка на повышение уровня
                    if (score >= level * 20) {
                        level++;
                        levelElement.textContent = level;
                        basketSpeed = 1 + level * 0.2;
                    }
                    
                    return;
                }
                
                // Если яблоко упало
                if (newTop > gameArea.offsetHeight) {
                    apple.remove();
                    missed++;
                    missedElement.textContent = missed;
                    
                    // Проверка на окончание игры
                    if (missed >= 10) {
                        endGame();
                        alert(`Игра окончена! Вы пропустили слишком много яблок!`);
                    }
                } else {
                    requestAnimationFrame(fall);
                }
            }
            
            requestAnimationFrame(fall);
        }
        
        // Таймер игры
        function updateTimer() {
            if (!gameActive) return;
            
            timeLeft--;
            timerElement.textContent = timeLeft;
            
            if (timeLeft <= 0) {
                endGame();
                alert(`Время вышло! Ваш счёт: ${score}`);
            }
        }
        
        // Начало игры
        function startGame() {
            if (gameActive) return;
            
            gameActive = true;
            score = 0;
            missed = 0;
            timeLeft = 60;
            level = 1;
            basketSpeed = 1;
            
            scoreElement.textContent = score;
            missedElement.textContent = missed;
            timerElement.textContent = timeLeft;
            levelElement.textContent = level;
            
            // Очистка предыдущих яблок
            document.querySelectorAll('.apple').forEach(apple => apple.remove());
            
            // Запуск таймеров
            gameTimer = setInterval(updateTimer, 1000);
            appleInterval = setInterval(createApple, 800);
            
            startBtn.textContent = 'Пауза';
            startBtn.style.background = 'linear-gradient(45deg, #f39c12, #e67e22)';
        }
        
        function pauseGame() {
            gameActive = false;
            clearInterval(gameTimer);
            clearInterval(appleInterval);
            startBtn.textContent = 'Продолжить';
            startBtn.style.background = 'linear-gradient(45deg, #3498db, #2980b9)';
        }
        
        function endGame() {
            gameActive = false;
            clearInterval(gameTimer);
            clearInterval(appleInterval);
            startBtn.textContent = 'Старт';
            startBtn.style.background = 'linear-gradient(45deg, #2ecc71, #27ae60)';
        }
        
        function resetGame() {
            endGame();
            startGame();
        }
        
        // Обработчики кнопок
        startBtn.addEventListener('click', function() {
            if (gameActive) {
                pauseGame();
            } else {
                startGame();
            }
        });
        
        resetBtn.addEventListener('click', resetGame);
        
        // Автоматическая адаптация при изменении размера окна
        window.addEventListener('resize', function() {
            const currentLeft = parseInt(basket.style.left);
            const maxLeft = gameArea.offsetWidth - basket.offsetWidth;
            basket.style.left = Math.min(currentLeft, maxLeft) + 'px';
        });
        
        // Начальная подсказка
        setTimeout(() => {
            if (!gameActive) {
                alert("Добро пожаловать в 'Ловец яблок'! Нажмите 'Старт' чтобы начать игру.");
            }
        }, 500);
    </script>
</body>
</html>
