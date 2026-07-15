<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>도파민 왕국: 확률의 군주</title>
    <!-- 구체적인 스타일링을 위해 구글 폰트 적용 -->
    <link href="https://fonts.googleapis.com/css2?family=Gothic+A1:wght@400;700&family=Playfair+Display:wght@700&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary-color: #f1c40f; /* 금색 */
            --secondary-color: #34495e; /* 깊은 파란색 */
            --bg-color-study: url('king_study.jpg'); /* 여기에 집무실 이미지 파일명 입력 */
            --bg-color-corridor: url('palace_corridor.jpg'); /* 여기에 복도 이미지 파일명 입력 */
            --font-family-body: 'Gothic+A1', sans-serif;
            --font-family-title: 'Playfair Display', serif;
        }

        body {
            font-family: var(--font-family-body);
            background-color: #333;
            color: white;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden; /* 배경 이미지를 위해 스크롤 방지 */
            background-size: cover;
            background-position: center;
            transition: background-image 0.5s ease;
        }

        /* 게임 컨테이너 */
        #game-container {
            width: 80%;
            max-width: 800px;
            background-color: rgba(0, 0, 0, 0.8);
            border: 5px solid var(--primary-color);
            border-radius: 15px;
            padding: 30px;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            position: relative;
        }

        /* 제목 스타일 */
        h1 {
            font-family: var(--font-family-title);
            color: var(--primary-color);
            margin-bottom: 20px;
        }

        /* 국가 스탯 스타일 */
        #status-bar {
            display: flex;
            justify-content: space-around;
            margin-bottom: 30px;
            border-bottom: 2px solid var(--primary-color);
            padding-bottom: 15px;
        }

        .stat {
            font-size: 1.1em;
        }

        .stat-value {
            font-weight: bold;
            color: var(--primary-color);
        }

        /* 사건/결과 스타일 */
        #event-container {
            margin-bottom: 30px;
            min-height: 150px;
            border: 2px dashed rgba(255, 255, 255, 0.5);
            border-radius: 10px;
            padding: 20px;
            background-color: rgba(255, 255, 255, 0.1);
        }

        #event-text, #result-text {
            font-size: 1.2em;
            margin-bottom: 15px;
        }

        #result-text {
            color: #ccc;
        }

        /* 버튼 스타일 */
        button {
            background-color: var(--secondary-color);
            color: var(--primary-color);
            border: 2px solid var(--primary-color);
            border-radius: 5px;
            padding: 10px 20px;
            font-size: 1.1em;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.1s;
            margin: 0 10px;
        }

        button:hover {
            background-color: var(--primary-color);
            color: var(--secondary-color);
        }

        button:active {
            transform: translateY(2px);
        }

        button:disabled {
            background-color: #555;
            color: #888;
            border-color: #888;
            cursor: not-allowed;
        }

        /* 애니메이션 효과 (주사위) */
        .dice {
            display: inline-block;
            font-size: 3em;
            margin: 10px;
            animation: none;
        }

        @keyframes rollDice {
            0% { transform: rotate(0deg); }
            25% { transform: rotate(180deg); }
            50% { transform: rotate(360deg); }
            75% { transform: rotate(540deg); }
            100% { transform: rotate(720deg); }
        }

        .rolling {
            animation: rollDice 0.5s ease-out;
        }

    </style>
</head>
<body>

    <div id="game-container">
        <h1>도파민 왕국: 확률의 군주</h1>

        <!-- 국가 상태 바 -->
        <div id="status-bar">
            <div class="stat">연도: <span id="year" class="stat-value">1</span></div>
            <div class="stat">금화: <span id="gold" class="stat-value">1000</span>g</div>
            <div class="stat">인구: <span id="population" class="stat-value">10000</span></div>
            <div class="stat">만족도: <span id="happiness" class="stat-value">80</span>%</div>
            <div class="stat">군사력: <span id="military" class="stat-value">100</span></div>
        </div>

        <!-- 사건/도박 선택 섹션 -->
        <div id="event-container">
            <p id="event-text">전하, 국정을 결정하실 시간입니다.</p>
            <p id="result-text"></p>
            <div id="visual-effect">
                <!-- 결과 시각화 (예: 주사위)가 들어갈 자리 -->
            </div>
        </div>

        <!-- 행동 버튼 -->
        <div id="action-buttons">
            <button id="choice-1-btn" onclick="makeChoice(0)">외교 협상 (성공 60%)</button>
            <button id="choice-2-btn" onclick="makeChoice(1)">농업 혁명 (성공 55%)</button>
            <button id="start-btn" onclick="startGame()">새로 시작</button>
        </div>

    </div>

    <script>
        // 초기 국가 스탯
        let gameState = {
            year: 1,
            gold: 1000,
            population: 10000,
            happiness: 80,
            military: 100,
            gameId: 0
        };

        const MAX_YEAR = 10;
        const choices = [
            {
                name: "외교 협상",
                description: "이웃 나라와 무역 협정을 맺어 경제를 활성화시킵니다.",
                successProbability: 0.6,
                onSuccess: (state) => {
                    state.gold += 500;
                    state.happiness += 10;
                    return "협상이 대성공을 거두어 금화 500g과 국민들의 만족도가 10% 상승했습니다!";
                },
                onFailure: (state) => {
                    state.gold -= 200;
                    state.military -= 10;
                    return "협상이 실패하여 금화 200g을 낭비하고 군사력이 10 하락했습니다.";
                }
            },
            {
                name: "농업 혁명",
                description: "새로운 농법을 도입하여 식량을 증산하고 인구를 늘립니다.",
                successProbability: 0.55,
                onSuccess: (state) => {
                    state.population += 2000;
                    state.happiness += 5;
                    return "농업 혁명이 성공하여 인구가 2000명 증가하고 만족도가 5% 상승했습니다!";
                },
                onFailure: (state) => {
                    state.gold -= 100;
                    state.happiness -= 10;
                    return "혁명 시도가 실패하여 흉작이 발생했습니다. 금화 100g과 만족도가 10% 하락했습니다.";
                }
            }
            // 더 많은 리얼한 선택지를 여기에 추가할 수 있습니다.
        ];

        function updateStatus() {
            document.getElementById('year').textContent = gameState.year;
            document.getElementById('gold').textContent = gameState.gold;
            document.getElementById('population').textContent = gameState.population;
            document.getElementById('happiness').textContent = gameState.happiness;
            document.getElementById('military').textContent = gameState.military;

            // 배경 이미지 업데이트 (예시: 집무실로 설정)
            document.body.style.backgroundImage = var(--bg-color-study);

            checkGameOver();
        }

        function startGame() {
            gameState = {
                year: 1,
                gold: 1000,
                population: 10000,
                happiness: 80,
                military: 100,
                gameId: gameState.gameId + 1
            };
            document.getElementById('start-btn').style.display = 'none';
            document.getElementById('choice-1-btn').style.display = 'inline';
            document.getElementById('choice-2-btn').style.display = 'inline';
            document.getElementById('choice-1-btn').disabled = false;
            document.getElementById('choice-2-btn').disabled = false;
            updateStatus();
            nextTurn();
        }

        function nextTurn() {
            if (gameState.year > MAX_YEAR) {
                return; // 게임 종료
            }
            // 매 턴 랜덤하게 두 가지 선택지를 제시
            gameState.year++;
            updateStatus();
            document.getElementById('event-text').textContent = "전하, 올해의 국정을 결정해주십시오.";
            document.getElementById('result-text').textContent = "";
            document.getElementById('choice-1-btn').style.display = 'inline';
            document.getElementById('choice-2-btn').style.display = 'inline';
            document.getElementById('visual-effect').innerHTML = ''; // 이전 효과 제거
        }

        function makeChoice(choiceIndex) {
            const choice = choices[choiceIndex];
            document.getElementById('choice-1-btn').disabled = true;
            document.getElementById('choice-2-btn').disabled = true;

            // 시각적 효과 (예: 주사위) 추가
            document.getElementById('visual-effect').innerHTML = '<div class="dice">🎲</div>';
            const dice = document.querySelector('.dice');
            dice.classList.add('rolling');

            // 확률 계산 (0~1 사이의 랜덤 값)
            const randomValue = Math.random();

            // 0.5초 후에 결과 공개
            setTimeout(() => {
                dice.classList.remove('rolling');
                let resultMessage = "";

                if (randomValue <= choice.successProbability) {
                    resultMessage = choice.onSuccess(gameState);
                    dice.innerHTML = "🎉"; // 승리 아이콘
                    // (성공 시 배경 이미지 변경 - 정원)
                    document.body.style.backgroundImage = var(--bg-color-garden); // 실제 이미지 파일명으로 변경 필요
                } else {
                    resultMessage = choice.onFailure(gameState);
                    dice.innerHTML = "😭"; // 패배 아이콘
                    // (패배 시 배경 이미지 변경 - 지하 감옥)
                    document.body.style.backgroundImage = var(--bg-color-dungeon); // 실제 이미지 파일명으로 변경 필요
                }

                document.getElementById('result-text').textContent = resultMessage;
                nextTurn();

            }, 500); // 0.5초 애니메이션 시간
        }

        function checkGameOver() {
            let gameOver = false;
            let gameOverReason = "";

            if (gameState.gold <= 0) {
                gameOver = true;
                gameOverReason = "국고가 바닥나 나라가 파산했습니다.";
            } else if (gameState.happiness <= 20) {
                gameOver = true;
                gameOverReason = "국민들의 폭동으로 왕좌에서 쫓겨났습니다.";
            } else if (gameState.military <= 0) {
                gameOver = true;
                gameOverReason = "이웃 나라의 침략으로 나라가 멸망했습니다.";
            } else if (gameState.year > MAX_YEAR) {
                gameOver = true;
                gameOverReason = "통치 10주년을 맞이했습니다! 당신의 통치는 역사에 어떻게 기록될까요?";
            }

            if (gameOver) {
                document.getElementById('event-text').textContent = "게임 오버!";
                document.getElementById('result-text').textContent = `${gameOverReason} 최종 스탯을 확인하십시오.`;
                document.getElementById('start-btn').style.display = 'inline';
                document.getElementById('choice-1-btn').disabled = true;
                document.getElementById('choice-2-btn').disabled = true;
            }
        }

        // 초기 시작
        // startGame(); // 나중에 실제 이미지 경로 설정 후 주석 해제

    </style>
</body>
</html># gambling
