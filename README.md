<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Klicoin & ELO Clicker</title>
    <style>
        /* Защита от зума и выделения */
        * { 
            touch-action: manipulation; 
            -webkit-tap-highlight-color: transparent; 
            user-select: none; 
            box-sizing: border-box; 
        }

        body { 
            text-align: center; 
            font-family: 'Segoe UI', sans-serif; 
            background: #0d0d1a; 
            color: #fff; 
            margin: 0; 
            padding: 20px; 
            overflow-x: hidden;
        }

        .header { margin: 20px 0; }
        #elo-rank { font-size: 22px; color: #00d2ff; text-shadow: 0 0 10px #00d2ff; }
        #balance { font-size: 34px; color: #f1c40f; font-weight: bold; margin-top: 5px; }

        /* Главная кнопка */
        #mainButton { 
            width: 180px; height: 180px; border-radius: 50%; border: 8px solid #16213e; 
            background: radial-gradient(circle, #1e3c72, #2a5298); color: white; font-size: 24px;
            font-weight: bold; cursor: pointer; transition: 0.1s; 
            box-shadow: 0 0 25px rgba(0,210,255,0.3);
            margin: 15px 0; outline: none;
        }
        #mainButton:active { transform: scale(0.92); }

        .section { 
            margin-top: 20px; 
            padding: 15px; 
            background: rgba(255,255,255,0.07); 
            border-radius: 15px; 
            border: 1px solid rgba(255,255,255,0.1); 
        }

        h3 { margin: 0 0 15px 0; font-size: 18px; color: #a29bfe; text-transform: uppercase; letter-spacing: 1px; }

        .btn { 
            padding: 14px; border: none; color: white; border-radius: 10px; 
            width: 100%; margin-bottom: 10px; font-size: 16px; font-weight: bold; cursor: pointer;
        }
        
        .btn-star { background: linear-gradient(90deg, #ffcc00, #ff9500); color: #000; }
        .btn-upgrade { background: #533483; display: flex; justify-content: space-between; align-items: center; }
        .btn-upgrade:disabled { opacity: 0.3; filter: grayscale(1); }
        
        input { 
            width: 100%; padding: 12px; border-radius: 8px; border: 2px solid #34495e; 
            margin-bottom: 10px; background: #1a1a2e; color: white; text-align: center; font-size: 16px; 
        }
        
        .admin-panel { display: none; border: 2px solid #e74c3c; background: #2c0000; padding: 15px; margin-top: 15px; }
        .price { color: #f1c40f; }
        .admin-btn-open { background: #333; margin-top: 20px; opacity: 0.5; font-size: 12px; padding: 8px; }
    </style>
</head>
<body>

    <div class="header">
        <div id="elo-rank">ELO: 0</div>
        <div id="balance">0.00 KC</div>
    </div>
    
    <button id="mainButton" onclick="clickAction()">ЖМИ!</button>

    <!-- Донат секция -->
    <div class="section">
        <h3>Донат (Stars ⭐️)</h3>
        <button class="btn btn-star" onclick="buyStars(5, 500)">500 KC — 5 ⭐️</button>
        <button class="btn btn-star" onclick="buyStars(10, 1500)">1500 KC — 10 ⭐️</button>
    </div>

    <!-- Магазин -->
    <div class="section">
        <h3>Улучшения</h3>
        <button class="btn btn-upgrade" id="btnUpg" onclick="buyUpg()">
            <span>Сила клика (x2)</span>
            <span class="price" id="costUpg">1.00 KC</span>
        </button>
    </div>

    <!-- Промокоды -->
    <div class="section">
        <h3>Промокод</h3>
        <input type="text" id="promoInput" placeholder="Введите код...">
        <button class="btn" style="background: #27ae60;" onclick="usePromo()">Активировать</button>
    </div>

    <!-- Админка -->
    <button class="btn admin-btn-open" onclick="openAdmin()">Админ-панель</button>

    <div id="adminArea" class="admin-panel section">
        <h3>АДМИН-ЦЕНТР</h3>
        <button class="btn" onclick="addMoney(1000)">+1000 KC</button>
        <button class="btn" onclick="addElo(5000)">+5000 ELO</button>
        <button class="btn" style="background: #c0392b;" onclick="resetGame()">Полный сброс</button>
        <button class="btn" onclick="closeAdmin()">Закрыть</button>
    </div>

    <script>
        // Данные игры
        let klicoins = 0.0;
        let elo = 0;
        let multiplier = 1;
        let costUpg = 1.00;
        let usedPromos = []; 

        const adminPass = "777"; 
        const myTelegram = "h9xer"; // Твой ник установлен

        function updateUI() {
            document.getElementById('balance').innerText = klicoins.toFixed(2) + " KC";
            document.getElementById('elo-rank').innerText = "ELO: " + elo;
            document.getElementById('btnUpg').disabled = klicoins < costUpg;
        }

        // Клик
        function clickAction() {
            klicoins += (0.05 * multiplier);
            elo += 10;
            updateUI();
        }

        // Покупка улучшения
        function buyUpg() {
            if (klicoins >= costUpg) {
                klicoins -= costUpg;
                multiplier *= 2;
                costUpg *= 2.5;
                updateUI();
                saveGame();
            }
        }

        // Функция доната через ТГ
        function buyStars(stars, amount) {
            let message = `Привет! Хочу купить ${amount} KC за ${stars} звёзд. Мой игровой ID: ${Math.floor(Math.random() * 999999)}`;
            let url = `https://t.me{myTelegram}?text=${encodeURIComponent(message)}`;
            
            if (confirm(`Перейти в Telegram к @${myTelegram} для покупки за ${stars} ⭐️?`)) {
                window.location.href = url;
            }
        }

        // Промокоды (Одноразовые)
        function usePromo() {
            let input = document.getElementById('promoInput');
            let code = input.value.toUpperCase().trim();
            
            if (usedPromos.includes(code)) {
                alert("Этот код уже был активирован!");
                return;
            }

            if (code === "START") {
                klicoins += 5.0;
                alert("Успех! +5 KC");
                usedPromos.push(code);
            } else if (code === "H9XER") {
                elo += 5000;
                alert("Секретный код активирован! +5000 ELO");
                usedPromos.push(code);
            } else {
                alert("Код неверный или не существует");
            }
            
            input.value = "";
            updateUI();
            saveGame();
        }

        // Админ-функции
        function openAdmin() {
            if (prompt("Введите пароль разработчика:") === adminPass) {
                document.getElementById('adminArea').style.display = "block";
            }
        }
        function closeAdmin() { document.getElementById('adminArea').style.display = "none"; }
        function addMoney(n) { klicoins += n; updateUI(); saveGame(); }
        function addElo(n) { elo += n; updateUI(); saveGame(); }

        // Сохранение и загрузка
        function saveGame() {
            const data = { 
                kc: klicoins, 
                elo: elo, 
                m: multiplier, 
                c: costUpg, 
                used: usedPromos 
            };
            localStorage.setItem('klicoin_master_save', JSON.stringify(data));
        }

        function loadGame() {
            const saved = localStorage.getItem('klicoin_master_save');
            if (saved) {
                const d = JSON.parse(saved);
                klicoins = d.kc || 0;
                elo = d.elo || 0;
                multiplier = d.m || 1;
                costUpg = d.c || 1.0;
                usedPromos = d.used || [];
                document.getElementById('costUpg').innerText = costUpg.toFixed(2) + " KC";
                updateUI();
            }
        }

        function resetGame() {
            if (confirm("ВНИМАНИЕ! Это полностью удалит ваш прогресс. Продолжить?")) {
                localStorage.clear();
                location.reload();
            }
        }

        // Инициализация
        loadGame();
        updateUI();
        // Автосохранение каждые 20 секунд
        setInterval(saveGame, 20000);

    </script>
</body>
</html>
