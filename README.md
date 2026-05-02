<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ELO CLICKER</title>
    <!-- Подключаем API Telegram -->
    <script src="https://telegram.org"></script>
    <style>
        :root { --main: #00ffcc; --coin: #ffcc00; --bg: #0a0a0a; --card: #1a1a1a; }
        * { box-sizing: border-box; outline: none; -webkit-tap-highlight-color: transparent; }
        
        body { 
            background: var(--bg); color: white; font-family: sans-serif; 
            margin: 0; user-select: none; overflow: hidden; touch-action: none;
            display: flex; flex-direction: column; height: 100vh;
        }

        .screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; z-index: 100; background: var(--bg); }
        .active-screen { display: flex; }

        /* Верх */
        .game-top {
            flex: 0 0 45%; display: flex; flex-direction: column; align-items: center; justify-content: center;
            position: relative; border-bottom: 2px solid #1a1a1a;
        }
        .profile-bar { position: absolute; top: 15px; left: 15px; display: flex; align-items: center; gap: 10px; }
        #u-avatar { width: 40px; height: 40px; border-radius: 50%; border: 2px solid var(--main); }
        .currencies { text-align: center; margin-bottom: 10px; }
        #elo-display { font-size: 2.5rem; color: var(--main); font-weight: bold; margin: 0; }
        #coin-display { font-size: 1.1rem; color: var(--coin); font-weight: bold; margin: 0; }

        #main-btn { 
            width: 130px; height: 130px; border-radius: 50%; border: 6px solid #1a1a1a; 
            background: var(--main); font-size: 1.2rem; font-weight: bold; color: #000;
            box-shadow: 0 0 20px rgba(0,255,204,0.2);
        }
        #main-btn:active { transform: scale(0.95); }

        /* Низ */
        .bottom-area { flex: 1; display: flex; flex-direction: column; background: #111; }
        .tabs { display: flex; background: #151515; }
        .tab { flex: 1; padding: 15px; border: none; background: none; color: #555; font-weight: bold; font-size: 0.75rem; }
        .tab.active { color: var(--main); border-bottom: 2px solid var(--main); }
        .list { flex: 1; overflow-y: auto; padding: 15px; }
        .card { background: var(--card); margin-bottom: 10px; padding: 15px; border-radius: 12px; display: flex; justify-content: space-between; align-items: center; }
        
        .buy-btn { background: var(--main); color: black; border: none; padding: 8px 12px; border-radius: 8px; font-weight: bold; font-size: 0.8rem; }
        .settings-btn { position: absolute; top: 15px; right: 15px; background: none; border: none; color: white; font-size: 1.2rem; }
    </style>
</head>
<body>

    <!-- ЭКРАН ЗАГРУЗКИ -->
    <div id="loading-screen" class="screen active-screen" style="justify-content: center; align-items: center;">
        <h1 style="color: var(--main)">ЗАГРУЗКА...</h1>
    </div>

    <!-- ИГРА -->
    <div id="game-screen" class="screen">
        <div class="game-top">
            <div class="profile-bar">
                <img id="u-avatar" src="https://placeholder.com">
                <div style="font-size: 0.8rem;">
                    <b id="u-name">Player</b><br>
                    <span id="u-title" style="color:var(--main); font-size:0.6rem;">НОВИЧОК</span>
                </div>
            </div>
            <button class="settings-btn" onclick="openSettings()">⚙️</button>
            <div class="currencies">
                <p id="elo-display">0</p>
                <p id="coin-display">0.00 CLC</p>
            </div>
            <button id="main-btn" onclick="clickElo()">TAP!</button>
        </div>

        <div class="bottom-area">
            <div class="tabs">
                <button id="t-upg" class="tab active" onclick="setTab('upg')">УЛУЧШЕНИЯ</button>
                <button id="t-don" class="tab" onclick="setTab('don')">⭐ МАГАЗИН</button>
            </div>
            <div id="list-cont" class="list"></div>
        </div>
    </div>

    <!-- НАСТРОЙКИ -->
    <div id="settings-screen" class="screen" style="padding: 20px; justify-content: center; align-items: center;">
        <div style="background:#111; padding:20px; border-radius:15px; width:100%;">
            <h2>Настройки</h2>
            <p>Версия 1.0.1 (Stable)</p>
            <button class="buy-btn" style="width:100%; padding:15px;" onclick="closeSettings()">ЗАКРЫТЬ</button>
            <button class="buy-btn" style="width:100%; padding:15px; background: red; color: white; margin-top: 10px;" onclick="resetGame()">СБРОС ДАННЫХ</button>
        </div>
    </div>

    <script>
        // Инициализация Telegram
        const tg = window.Telegram ? window.Telegram.WebApp : null;
        if (tg) {
            tg.ready();
            tg.expand();
        }

        let gameData = { 
            elo: 0, clicoins: 0, clickElo: 10, clickCoins: 0.05,
            nick: 'Игрок', title: 'НОВИЧОК'
        };

        // Запуск
        window.onload = () => {
            setTimeout(() => {
                const saved = localStorage.getItem('elo_save_final');
                if (saved) gameData = JSON.parse(saved);
                
                // Берем имя из Telegram если есть
                if (tg && tg.initDataUnsafe && tg.initDataUnsafe.user) {
                    gameData.nick = tg.initDataUnsafe.user.first_name;
                }

                document.getElementById('loading-screen').classList.remove('active-screen');
                document.getElementById('game-screen').classList.add('active-screen');
                updateUI();
            }, 1000);
        };

        function clickElo() {
            gameData.elo += gameData.clickElo;
            gameData.clicoins += gameData.clickCoins;
            updateUI();
            save();
        }

        function updateUI() {
            document.getElementById('elo-display').innerText = Math.floor(gameData.elo).toLocaleString();
            document.getElementById('coin-display').innerText = gameData.clicoins.toFixed(2) + " CLC";
            document.getElementById('u-name').innerText = gameData.nick;
            renderList(document.querySelector('.tab.active').id.replace('t-', ''));
        }

        function renderList(t) {
            const cont = document.getElementById('list-cont');
            cont.innerHTML = '';
            if (t === 'upg') {
                cont.innerHTML = `<div class="card">
                    <div><b>Клик +10 ELO</b><br><small>Цена: 2.00 CLC</small></div>
                    <button class="buy-btn" onclick="buyUpg()">КУПИТЬ</button>
                </div>`;
            } else {
                cont.innerHTML = `<div class="card">
                    <div><b>100 Clicoins</b><br><small>⭐ 50 Stars</small></div>
                    <button class="buy-btn" onclick="buyStars(50, 100)">КУПИТЬ</button>
                </div>`;
            }
        }

        function buyUpg() {
            if (gameData.clicoins >= 2) {
                gameData.clicoins -= 2;
                gameData.clickElo += 10;
                updateUI(); save();
            } else {
                if (tg) tg.showAlert("Недостаточно CLC!"); else alert("Недостаточно CLC!");
            }
        }

        function buyStars(stars, coins) {
            if (tg) {
                tg.showConfirm(`Купить ${coins} CLC за ⭐ ${stars}?`, (ok) => {
                    if (ok) {
                        gameData.clicoins += coins;
                        updateUI(); save();
                        tg.showAlert("Успешно куплено!");
                    }
                });
            } else {
                alert("Эта функция работает только в Telegram!");
            }
        }

        function save() { localStorage.setItem('elo_save_final', JSON.stringify(gameData)); }
        function setTab(t) {
            document.querySelectorAll('.tab').forEach(b => b.classList.remove('active'));
            document.getElementById('t-' + t).classList.add('active');
            renderList(t);
        }
        function openSettings() { document.getElementById('settings-screen').classList.add('active-screen'); }
        function closeSettings() { document.getElementById('settings-screen').classList.remove('active-screen'); }
        function resetGame() { localStorage.clear(); location.reload(); }
    </script>
</body>
</html>
