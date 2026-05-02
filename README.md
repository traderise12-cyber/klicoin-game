<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Klicoin Global Admin</title>
    <style>
        * { touch-action: manipulation; -webkit-tap-highlight-color: transparent; user-select: none; box-sizing: border-box; }
        body { text-align: center; font-family: sans-serif; background: #0d0d1a; color: #fff; margin: 0; padding: 20px; }
        
        .header { margin-bottom: 20px; }
        #elo-rank { font-size: 20px; color: #00d2ff; }
        #balance { font-size: 30px; color: #f1c40f; font-weight: bold; }

        #mainButton { 
            width: 150px; height: 150px; border-radius: 50%; border: 6px solid #16213e; 
            background: #1e3c72; color: white; font-size: 20px; font-weight: bold;
            cursor: pointer; transition: 0.1s; box-shadow: 0 0 20px rgba(0,0,0,0.5);
        }
        #mainButton:active { transform: scale(0.92); }

        .section { margin-top: 15px; padding: 15px; background: rgba(255,255,255,0.05); border-radius: 15px; border: 1px solid rgba(255,255,255,0.1); }
        
        .btn { padding: 12px; border: none; color: white; border-radius: 8px; width: 100%; margin-bottom: 8px; font-weight: bold; cursor: pointer; }
        .btn-green { background: #27ae60; }
        .btn-blue { background: #3498db; }
        
        input, select { width: 100%; padding: 10px; border-radius: 8px; border: none; margin-bottom: 8px; background: #2c3e50; color: white; text-align: center; }
        
        .admin-panel { display: none; background: #1a0000; border: 2px solid #ff4757; text-align: left; }
        label { font-size: 12px; color: #aaa; margin-left: 5px; }
    </style>
</head>
<body>

    <div class="header">
        <div id="elo-rank">ELO: 0</div>
        <div id="balance">0.00 KC</div>
    </div>
    
    <button id="mainButton" onclick="clickAction()">ЖМИ!</button>

    <div class="section">
        <h3>Активация кода</h3>
        <input type="text" id="promoInput" placeholder="Введите код">
        <button class="btn btn-green" onclick="usePromo()">Активировать</button>
    </div>

    <button class="btn" style="background:#333; opacity:0.6;" onclick="openAdmin()">Админ-панель</button>

    <div id="adminArea" class="section admin-panel">
        <h3 style="text-align:center">КОНСТРУКТОР КОДОВ</h3>
        
        <label>Название кода:</label>
        <input type="text" id="newPromoName" placeholder="Напр: FREE100">
        
        <label>Тип награды:</label>
        <select id="newPromoType">
            <option value="money">Валюта (KC)</option>
            <option value="elo">Рейтинг (ELO)</option>
        </select>
        
        <label>Количество награды:</label>
        <input type="number" id="newPromoVal" placeholder="Сумма">
        
        <label>Лимит активаций (сколько человек введет):</label>
        <input type="number" id="newPromoLimit" placeholder="Напр: 50" value="1">
        
        <button class="btn btn-blue" onclick="createPromo()">Создать промокод</button>
        <button class="btn" style="background:#c0392b" onclick="closeAdmin()">Закрыть</button>
    </div>

    <script>
        let klicoins = 0.0;
        let elo = 0;
        let customPromos = {}; 
        let userActivated = []; // Коды, которые ввел именно этот пользователь

        function updateUI() {
            document.getElementById('balance').innerText = klicoins.toFixed(2) + " KC";
            document.getElementById('elo-rank').innerText = "ELO: " + elo;
        }

        function clickAction() {
            klicoins += 0.05;
            elo += 10;
            updateUI();
        }

        function createPromo() {
            let name = document.getElementById('newPromoName').value.toUpperCase().trim();
            let type = document.getElementById('newPromoType').value;
            let val = parseFloat(document.getElementById('newPromoVal').value) || 0;
            let limit = parseInt(document.getElementById('newPromoLimit').value) || 1;

            if (!name) return alert("Введите название!");

            customPromos[name] = { 
                type: type, 
                value: val, 
                limit: limit, 
                uses: 0 // Сколько раз код уже использовали всего
            };
            
            alert(`Код ${name} создан! Лимит: ${limit} раз.`);
            saveGame();
            document.getElementById('newPromoName').value = "";
        }

        function usePromo() {
            let input = document.getElementById('promoInput');
            let code = input.value.toUpperCase().trim();

            if (userActivated.includes(code)) {
                alert("Вы уже использовали этот код!");
                return;
            }

            if (customPromos[code]) {
                let p = customPromos[code];

                // Проверка общего лимита активаций
                if (p.uses >= p.limit) {
                    alert("Лимит активаций этого кода исчерпан!");
                    return;
                }

                // Выдача награды
                if (p.type === "money") klicoins += p.value;
                if (p.type === "elo") elo += p.value;

                p.uses += 1; // Увеличиваем общий счетчик использований
                userActivated.push(code); // Запоминаем, что этот юзер его ввел

                alert(`Успех! Награда получена. (Использовано ${p.uses}/${p.limit})`);
                updateUI();
                saveGame();
            } else {
                alert("Код не найден");
            }
            input.value = "";
        }

        function openAdmin() {
            if (prompt("Пароль:") === "777") document.getElementById('adminArea').style.display = "block";
        }
        function closeAdmin() { document.getElementById('adminArea').style.display = "none"; }

        function saveGame() {
            const data = { kc: klicoins, elo: elo, cp: customPromos, ua: userActivated };
            localStorage.setItem('klicoin_v5_save', JSON.stringify(data));
        }

        function loadGame() {
            const saved = localStorage.getItem('klicoin_v5_save');
            if (saved) {
                const d = JSON.parse(saved);
                klicoins = d.kc || 0;
                elo = d.elo || 0;
                customPromos = d.cp || {};
                userActivated = d.ua || [];
                updateUI();
            }
        }

        loadGame();
    </script>
</body>
</html>
