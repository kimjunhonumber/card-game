# card-game<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>과일 카드 뒤집기 게임</title>
    <link href="https://fonts.googleapis.com/css2?family=Jua&display=swap" rel="stylesheet">
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/tone@14.7.58/build/Tone.js"></script>
    <style>
        body {
            font-family: 'Jua', sans-serif;
            background-color: #1a202c;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            color: #e2e8f0;
        }
        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
            padding: 20px;
            background-color: #2d3748;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            max-width: 600px;
            width: 95%;
        }
        .game-board {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 10px;
            perspective: 1000px;
        }
        .card {
            width: 100px;
            height: 120px;
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.5s;
            cursor: pointer;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            transition: transform 0.5s, opacity 0.5s;
        }
        .card.flipped {
            transform: rotateY(180deg);
        }
        .card.matched {
            opacity: 0;
            cursor: default;
        }
        .card-face {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 2.5rem;
            border-radius: 0.5rem;
            transition: background-color 0.3s;
        }
        .card-back {
            background-color: #4a5568;
            color: #cbd5e0;
        }
        .card-front {
            background-color: #a0aec0;
            color: #2d3748;
            transform: rotateY(180deg);
        }
        .card.matched .card-front {
            background-color: #48bb78;
        }
        .score-board {
            display: flex;
            gap: 20px;
            font-size: 1.25rem;
        }
        #timer {
            font-size: 1.5rem;
            font-weight: bold;
            color: #f6ad55;
        }
        #message {
            font-size: 1.5rem;
            height: 30px;
            text-align: center;
        }
        .modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-content {
            background-color: #2d3748;
            padding: 40px;
            border-radius: 1rem;
            text-align: center;
            color: #e2e8f0;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            animation: fadeIn 0.5s ease-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.9); }
            to { opacity: 1; transform: scale(1); }
        }
        .close-button {
            margin-top: 20px;
            background-color: #48bb78;
            color: white;
            padding: 10px 20px;
            border-radius: 0.5rem;
            cursor: pointer;
            font-weight: bold;
        }
        @media (max-width: 600px) {
            .game-board {
                grid-template-columns: repeat(4, 1fr);
                gap: 8px;
            }
            .card {
                width: 70px;
                height: 90px;
                font-size: 2rem;
            }
        }
        .difficulty-buttons button {
            background-color: #4a5568;
            color: #e2e8f0;
            padding: 8px 16px;
            border-radius: 0.5rem;
            transition: background-color 0.3s;
        }
        .difficulty-buttons button.active, .difficulty-buttons button:hover {
            background-color: #48bb78;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-3xl font-bold mb-4">과일 카드 뒤집기 게임</h1>
        <div class="flex flex-col items-center">
            <h2 class="text-lg mb-2">난이도 선택</h2>
            <div class="difficulty-buttons flex gap-2">
                <button id="easy-btn" data-time="60" class="active">쉬움</button>
                <button id="medium-btn" data-time="45">보통</button>
                <button id="hard-btn" data-time="30">어려움</button>
            </div>
        </div>
        <div class="score-board">
            <span>시도 횟수: <span id="moves-count">0</span></span>
            <span>짝 맞춤: <span id="matches-count">0</span> / 10</span>
        </div>
        <div id="timer">남은 시간: <span id="time-left">60</span>초</div>
        <div class="game-board" id="game-board"></div>
        <div id="message"></div>
        <button id="restart-button" class="mt-4 px-6 py-2 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-lg shadow-md transition-all">재시작</button>
    </div>

    <div id="win-modal" class="modal">
        <div class="modal-content">
            <h2 class="text-2xl mb-2">승리! 🎉</h2>
            <p id="final-message" class="text-lg"></p>
            <div id="high-score-display" class="mt-4"></div>
            <button id="modal-close" class="close-button">새 게임 시작</button>
        </div>
    </div>

    <div id="lose-modal" class="modal">
        <div class="modal-content">
            <h2 class="text-2xl mb-2">게임 오버! 😢</h2>
            <p class="text-lg">시간이 다 되었어요. 다음엔 더 빨리 도전해 보세요!</p>
            <button id="lose-modal-close" class="close-button">새 게임 시작</button>
        </div>
    </div>
    
    <div id="memorize-modal" class="modal">
        <div class="modal-content">
            <h2 class="text-2xl mb-2">기억하세요!</h2>
            <p id="memorize-timer" class="text-lg font-bold text-yellow-400"></p>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const gameBoard = document.getElementById('game-board');
            const movesCountSpan = document.getElementById('moves-count');
            const matchesCountSpan = document.getElementById('matches-count');
            const messageDiv = document.getElementById('message');
            const restartButton = document.getElementById('restart-button');
            const winModal = document.getElementById('win-modal');
            const loseModal = document.getElementById('lose-modal');
            const finalMessageP = document.getElementById('final-message');
            const modalCloseButton = document.getElementById('modal-close');
            const loseModalCloseButton = document.getElementById('lose-modal-close');
            const timeLeftSpan = document.getElementById('time-left');
            const memorizeModal = document.getElementById('memorize-modal');
            const memorizeTimerP = document.getElementById('memorize-timer');
            const difficultyButtons = document.querySelectorAll('.difficulty-buttons button');
            const highScoreDisplay = document.getElementById('high-score-display');
            
            // Tone.js를 이용한 사운드 효과
            const matchSynth = new Tone.Synth().toDestination();
            const wrongSynth = new Tone.NoiseSynth({
                noise: {
                    type: "pink"
                },
                envelope: {
                    attack: 0.005,
                    decay: 0.1,
                    sustain: 0.0,
                }
            }).toDestination();
            const flipSynth = new Tone.MembraneSynth().toDestination();

            // 과일 이모지 목록
            const fruitEmojis = ['🍇', '🍈', '🍉', '🍊', '🍋', '🍌', '🍍', '🥭', '🍎', '🍓'];
            let cards = [...fruitEmojis, ...fruitEmojis];
            let flippedCards = [];
            let matchedCards = [];
            let canFlip = false;
            let moves = 0;
            let matches = 0;
            let gameTimer;
            let memorizeTimer;
            let gameTimeLeft = 60;
            let memorizeTimeLeft = 5;

            // Load high score from local storage
            let highScore = JSON.parse(localStorage.getItem('cardMatchHighScore')) || { time: Infinity, moves: Infinity };

            function updateHighScoreDisplay() {
                if (highScore.time !== Infinity) {
                    highScoreDisplay.innerHTML = `<p class="font-bold">최고 기록: <span class="text-green-400">${highScore.time}초</span>, <span class="text-blue-400">${highScore.moves}회</span></p>`;
                } else {
                    highScoreDisplay.textContent = '아직 최고 기록이 없습니다.';
                }
            }

            // Google Sheets에 데이터를 저장하는 함수
            async function saveHighScoreToSheet(difficulty, moves, timeLeft) {
                // Apps Script 배포 시 생성된 URL을 여기에 붙여넣으세요.
                const url = '<YOUR_APPS_SCRIPT_URL_HERE>'; 
                
                // 전송할 데이터
                const data = {
                    difficulty: getDifficultyLabel(difficulty),
                    moves: moves,
                    timeLeft: timeLeft
                };

                try {
                    const response = await fetch(url, {
                        method: 'POST',
                        mode: 'no-cors', // Apps Script CORS 정책 때문에 필요
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify(data),
                    });
                    console.log('데이터가 Google Sheets에 성공적으로 전송되었습니다.');
                } catch (error) {
                    console.error('Google Sheets에 데이터를 저장하는 중 오류가 발생했습니다:', error);
                }
            }
            
            // 난이도에 해당하는 라벨을 반환하는 헬퍼 함수
            function getDifficultyLabel(time) {
                if (time === 60) return '쉬움';
                if (time === 45) return '보통';
                if (time === 30) return '어려움';
                return '알 수 없음';
            }


            function shuffle(array) {
                for (let i = array.length - 1; i > 0; i--) {
                    const j = Math.floor(Math.random() * (i + 1));
                    [array[i], array[j]] = [array[j], array[i]];
                }
            }

            function startMemorizePhase() {
                memorizeModal.style.display = 'flex';
                memorizeTimerP.textContent = `${memorizeTimeLeft}초`;
                
                // Show all cards for memorization
                const allCards = document.querySelectorAll('.card');
                allCards.forEach(card => card.classList.add('flipped'));

                memorizeTimer = setInterval(() => {
                    memorizeTimeLeft--;
                    memorizeTimerP.textContent = `${memorizeTimeLeft}초`;
                    if (memorizeTimeLeft <= 0) {
                        clearInterval(memorizeTimer);
                        memorizeModal.style.display = 'none';
                        allCards.forEach(card => card.classList.remove('flipped'));
                        startMainTimer();
                        canFlip = true;
                    }
                }, 1000);
            }

            function startMainTimer() {
                clearInterval(gameTimer);
                timeLeftSpan.textContent = gameTimeLeft;
                gameTimer = setInterval(() => {
                    gameTimeLeft--;
                    timeLeftSpan.textContent = gameTimeLeft;
                    if (gameTimeLeft <= 0) {
                        clearInterval(gameTimer);
                        showLoseModal();
                    }
                }, 1000);
            }

            function createBoard() {
                gameBoard.innerHTML = '';
                shuffle(cards);
                flippedCards = [];
                matchedCards = [];
                moves = 0;
                matches = 0;
                canFlip = false;
                movesCountSpan.textContent = moves;
                matchesCountSpan.textContent = matches;
                messageDiv.textContent = '';
                
                cards.forEach(cardValue => {
                    const card = document.createElement('div');
                    card.classList.add('card');
                    card.dataset.value = cardValue;
                    
                    const cardBack = document.createElement('div');
                    cardBack.classList.add('card-face', 'card-back');
                    cardBack.textContent = '❓';
                    
                    const cardFront = document.createElement('div');
                    cardFront.classList.add('card-face', 'card-front');
                    cardFront.textContent = cardValue;

                    card.appendChild(cardBack);
                    card.appendChild(cardFront);

                    card.addEventListener('click', flipCard);
                    gameBoard.appendChild(card);
                });
                memorizeTimeLeft = 5;
                startMemorizePhase();
                updateHighScoreDisplay();
            }

            function flipCard(event) {
                if (!canFlip || gameTimeLeft <= 0) return;
                const clickedCard = event.currentTarget;
                
                if (clickedCard.classList.contains('flipped') || matchedCards.includes(clickedCard)) {
                    return;
                }

                // Play flip sound
                flipSynth.triggerAttackRelease("C4", "8n");

                clickedCard.classList.add('flipped');
                flippedCards.push(clickedCard);
                
                if (flippedCards.length === 2) {
                    canFlip = false;
                    moves++;
                    movesCountSpan.textContent = moves;

                    const [card1, card2] = flippedCards;
                    if (card1.dataset.value === card2.dataset.value) {
                        messageDiv.textContent = '짝을 맞췄어요! 🎉';
                        matches++;
                        matchesCountSpan.textContent = matches;

                        // Play match sound
                        matchSynth.triggerAttackRelease("C5", "8n");
                        
                        // Add matched class to trigger fade-out animation and hide
                        card1.classList.add('matched');
                        card2.classList.add('matched');

                        setTimeout(() => {
                            messageDiv.textContent = '';
                            canFlip = true;
                            if (matches === cards.length / 2) {
                                clearInterval(gameTimer);
                                // Check for new high score
                                if (moves < highScore.moves || (moves === highScore.moves && gameTimeLeft > highScore.time)) {
                                     highScore.moves = moves;
                                     highScore.time = gameTimeLeft;
                                     localStorage.setItem('cardMatchHighScore', JSON.stringify(highScore));
                                }
                                showWinModal();
                            }
                        }, 1000);
                        flippedCards = [];
                    } else {
                        messageDiv.textContent = '다시 시도해 보세요!';
                        // Play wrong sound
                        wrongSynth.triggerAttackRelease("4n");

                        setTimeout(() => {
                            card1.classList.remove('flipped');
                            card2.classList.remove('flipped');
                            flippedCards = [];
                            messageDiv.textContent = '';
                            canFlip = true;
                        }, 1000);
                    }
                }
            }

            function showWinModal() {
                finalMessageP.textContent = `축하해요! ${moves}번의 시도만에 모든 짝을 맞췄습니다!`;
                updateHighScoreDisplay();
                winModal.style.display = 'flex';
                // 승리 시 Google Sheets에 데이터 저장
                saveHighScoreToSheet(gameTimeLeft, moves, gameTimeLeft);
            }

            function showLoseModal() {
                loseModal.style.display = 'flex';
            }

            modalCloseButton.addEventListener('click', () => {
                winModal.style.display = 'none';
                createBoard();
            });

            loseModalCloseButton.addEventListener('click', () => {
                loseModal.style.display = 'none';
                createBoard();
            });

            restartButton.addEventListener('click', () => {
                clearInterval(gameTimer);
                clearInterval(memorizeTimer);
                createBoard();
            });

            difficultyButtons.forEach(button => {
                button.addEventListener('click', () => {
                    difficultyButtons.forEach(btn => btn.classList.remove('active'));
                    button.classList.add('active');
                    gameTimeLeft = parseInt(button.dataset.time);
                    createBoard();
                });
            });

            // Initial board creation
            createBoard();
        });
    </script>
</body>
</html>
