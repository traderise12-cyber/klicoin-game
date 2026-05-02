<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Klicoin & ELO Pro</title>
    <style>
        * {
            touch-action: manipulation;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
            box-sizing: border-box;
        }

        body { 
            text-align: center; 
            font-family: 'Segoe UI', sans-serif; 
            background: #1a1a2e; 
            color: #fff; 
            margin: 0;
            padding: 20px;
        }

        .header { margin-bottom: 20px; }
        #elo-rank { font-size: 20px; color: #00d2ff; }
        #balance { font-size: 30px; color: #f1c40f; font-weight: bold; }

        #mainButton { 
            width: 150px; height: 150px; border-radius: 50%; border: 6px solid #16213e; 
            background: #0f3460; color: white; font-size: 20px; font-weight: bold;
            cursor: pointer; transition: 0.1s; box-shadow: 0 5px 15px rgba(0,0,0,0.3);
        }
        #mainButton:active { transform: scale(0.95); }

        .section {
            margin-top: 25px;
            padding: 15px;
            background: rgba(255,255,255,0.05);
            border-radius: 15px;
        }

        .upgrade { 
            padding: 12px; background: #533483; border: none; color: white; 
            border-radius: 8px; width: 100%; margin-bottom: 8px;
            display: flex; justify-content: space-between;
        }
        .upgrade:disabled { opacity: 0.3; }

        .settings-btn {
            background: #444; border: none; color: white; padding: 10px;
            border-radius: 5px; width: 100%; margin-top: 10px;
        }
        
        .reset-btn { background: #c0392b; }
    </style>
</head>
<body>

    <div class="header">
        <div id="elo-rank">ELO: 0</div>
        <div id="balance">0.00 Klicoins</div>
    </div>
    
    <button id="mainButton" onclick="clickAction()">КЛИК!</button>

    <div class="section">
        <h3>Магазин</h3>
        <button class="upgrade" id="btnUpg" onclick="buyUpg()">
            <span>Улучшить клик</span>
            <span id="costUpg">1.00 KC</span>
        </button>
        <button class="upgrade" id="btnSkin" onclick="buySkin()">
            <span>Скин игрока</span>
            <span>5.00 KC</span>
        </button>
    </div>

    <div class="section">
        <h3>Настройки</h3>
        <button class="settings-btn" onclick="saveGame()">Сохранить игру</button>
        <button class="settings-btn reset-btn" onclick="resetGame()">Сбросить всё</button>
    </div>

    <script>
        let klicoins = 0.0;
        let elo = 0;
        let multiplier = 1;
        let costUpg = 1.00;

        function updateUI() {
            document.getElementById('balance').innerText = klicoins.toFixed(2) + " Klicoins";
            document.getElementById('elo-rank').innerText = "ELO: " + elo;
            document.getElementById('btnUpg').disabled = klicoins < costUpg;
            document.getElementById('btnSkin').disabled = klicoins < 5.00;
        }

        function clickAction() {
            klicoins += (0.05 * multiplier);
            elo += 10;
            updateUI();
        }

        function buyUpg() {
            if (klicoins >= costUpg) {
                klicoins -= costUpg;
                multiplier += 1;
                costUpg *= 2;
                document.getElementById('costUpg').innerText = costUpg.toFixed(2) + " KC";
                updateUI();
                saveGame();
            }
        }

        function buySkin() {
            if (klicoins >= 5.00) {
                klicoins -= 5.00;
                document.getElementById('mainButton').style.background = "#e74c3c";
                updateUI();
            }
        }

        // --- ФУНКЦИИ КНОПОК НАСТРОЕК ---

        function saveGame() {
            const data = {
                kc: klicoins,
                elo: elo,
                mult: multiplier,
                cost: costUpg
            };
            localStorage.setItem('klicoinSave', JSON.stringify(data));
            alert("Игра сохранена!");
        }

        function loadGame() {
            const saved = localStorage.getItem('klicoinSave');
            if (saved) {
                const data = JSON.parse(saved);
                klicoins = data.kc;
                elo = data.elo;
                multiplier = data.mult;
                costUpg = data.cost;
                document.getElementById('costUpg').innerText = costUpg.toFixed(2) + " KC";
                updateUI();
            }
        }

        function resetGame() {
            if (confirm("Точно сбросить весь прогресс?")) {
                localStorage.removeItem('klicoinSave');
                location.reload();
            }
        }

        // Загрузка при старте
        loadGame();
        updateUI();
    </script>
</body>
</html>
