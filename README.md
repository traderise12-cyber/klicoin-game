<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ELO CLICKER: TELEGRAM EDITION</title>
    <!-- Подключение API Telegram -->
    <script src="https://telegram.org"></script>
    <style>
        :root { --main: #00ffcc; --coin: #ffcc00; --bg: #0a0a0a; --card: #1a1a1a; --tg-blue: #24A1DE; }
        * { box-sizing: border-box; outline: none; -webkit-tap-highlight-color: transparent; }
        
        body { 
            background: var(--bg); color: white; font-family: sans-serif; 
            margin: 0; user-select: none; overflow: hidden; touch-action: none;
            display: flex; flex-direction: column; height: 100vh;
        }

        .screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; z-index: 100; background: var(--bg); }
        .active-screen { display: flex; }

        /* Верхняя зона */
        .game-top {
            flex: 0 0 45%; display: flex; flex-direction: column; align-items: center; justify-content: center;
            position: relative; border-bottom: 2px solid #1a1a1a;
        }
        
        .profile-bar { position: absolute; top: 15px; left: 15px; display: flex; align-items: center; gap: 10px; }
        #u-avatar { width: 45px; height: 45px; border-radius: 50%; object-fit: cover; border: 2px solid var(--main); background: #222; }
        
        .user-info { display: flex; flex-direction: column; text-align: left; }
        #u-name { font-weight: bold; font-size: 0.9rem; background-clip: text; -webkit-background-clip: text; }
        #u-id { font-size: 0.7rem; color: #777; }
        #u-title { font-size: 0.6rem; color: var(--main); font-weight: bold; text-transform: uppercase; }

        .settings-btn { position: absolute; top: 15px; right: 15px; background: none; border: none; color: white; font-size: 1.5rem; }

        .currencies { text-align: center; margin-bottom: 10px; }
        #elo-display { font-size: 2.8rem; color: var(--main); font-weight: bold; margin: 0; }
        #coin-display { font-size: 1.2rem; color: var(--coin); font-weight: bold; margin: 0; }

        #main-btn { 
            width: 140px; height: 140px; border-radius: 50%; border: 8px solid #1a1a1a; 
            background: var(--main); font-size: 1.3rem; font-weight: bold; transition: 0.1s;
        }
        #main-btn:active { transform: scale(0.9); }

        /* Нижнее меню */
        .bottom-area { flex: 1; display: flex; flex-direction: column; background: #111; }
        .tabs { display: flex; background: #151515; }
        .tab { flex: 1; padding: 15px; border: none; background: none; color: #555; font-weight: bold; font-size: 0.75rem; }
        .tab.active { color: var(--main); border-bottom: 2px solid var(--main); }
        .list { flex: 1; overflow-y: auto; padding: 15px; }
        
        .card { background: var(--card); margin-bottom: 10px; padding: 15px; border-radius: 12px; display: flex; justify-content: space-between; align-items: center; }
        .price-tag { background: var(--tg-blue); color: white; border: none; padding: 8px 12px; border-radius: 8px; font-weight: bold; }

        /* Настройки */
        .ui-input { width: 100%; padding: 12px; margin: 5px 0; border-radius: 8px; border: 1px solid #333; background: #000; color: white; font-size: 0.85rem; }
        .country-list { height: 150px; overflow-y: auto; border: 1px solid #222; border-radius: 8px; margin-top: 5px; }
        .country-item { padding: 10px; border-bottom: 1px solid #222; font-size: 0.8rem; display: flex; justify-content: space-between; }
    </style>
</head>
<body>

    <!-- ЭКРАН ВХОДА -->
    <div id="auth-screen" class="screen active-screen" style="justify-content: center; align-items: center;">
        <h1 style="color: var(--main); margin-bottom: 30px;">ELO CLICKER</h1>
        <button class="auth-btn" style="width:260px; padding:15px; border-radius:12px; border:none; font-weight:bold;" onclick="login('tg')">ВОЙТИ ЧЕРЕЗ TELEGRAM</button>
    </div>

    <!-- ИГРА -->
    <div id="game-screen" class="screen">
        <div class="game-top">
            <div class="profile-bar">
                <img id="u-avatar" src="https://placeholder.com">
                <div class="user-info">
                    <span id="u-name">Player</span>
                    <span id="u-id">#0000</span>
                    <span id="u-title">НОВИЧОК</span>
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
                <button id="t-don" class="tab" onclick="setTab('don')">ЗВЕЗДЫ ⭐</button>
                <button id="t-st" class="tab" onclick="setTab('st')">СТАТЫ</button>
            </div>
            <div id="list-cont" class="list"></div>
        </div>
    </div>

    <!-- НАСТРОЙКИ -->
    <div id="settings-screen" class="screen" style="padding: 20px; overflow-y: auto;">
        <div style="background:#111; padding:20px; border-radius:15px; width:100%; margin-top: 30px;">
            <h2 onclick="adminTaps()" id="st-title">Настройки</h2>
            <label style="font-size:0.7rem; color:#555">Ваш ник:</label>
            <input type="text" id="edit-nick" class="ui-input" onchange="gameData.nick=this.value; updateUI(); save()">
            
            <label style="font-size:0.7rem; color:#555; margin-top:10px; display:block;">Регион (валюта):</label>
            <input type="text" id="search-reg" class="ui-input" placeholder="Поиск страны..." oninput="filterReg()">
            <div class="country-list" id="reg-list"></div>

            <!-- АДМИНКА -->
            <div id="admin-box" style="display:none; margin-top:15px; border-top:1px dashed #333; padding-top:10px;">
                <input type="password" id="adm-key" class="ui-input" placeholder="Ключ (8989)">
                <button onclick="checkAdmin()" style="background:var(--main); border:none; padding:10px; width:100%; border-radius:8px; font-weight:bold;">ВОЙТИ В АДМИНКУ</button>
                <div id="admin-tools" style="display:none; margin-top:10px;">
                    <input type="text" class="ui-input" placeholder="Кастомный Титул" onchange="gameData.title=this.value; updateUI(); save()">
                    <input type="text" class="ui-input" placeholder="ID (напр. #ADMIN)" onchange="gameData.id=this.value; updateUI(); save()">
                    <button onclick="gameData.elo+=1000000; updateUI(); save()" style="background:orange; width:100%; padding:10px; border:none; border-radius:8px; margin-top:5px;">+1M ELO</button>
                </div>
            </div>

            <button class="buy-btn" style="width:100%; padding:15px; background:#333; color:white; border:none; border-radius:10px; margin-top:20px;" onclick="closeSettings()">ЗАКРЫТЬ</button>
        </div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();

        let gameData = { 
            elo: 0, clicoins: 0, clickElo: 10, clickCoins: 0.05,
            nick: 'Игрок', id: '#0000', title: 'НОВИЧОК', avatar: 'https://placeholder.com',
            region: 'Russia', totalClicks: 0
        };

        const countries = [
            { id: 'Russia', n: 'Россия', s: '₽', r: 95 },
            { id: 'USA', n: 'USA', s: '$', r: 1 },
            { id: 'Ukraine', n: 'Украина', s: '₴', r: 41 },
            { id: 'Europe', n: 'Европа', s: '€', r: 0.92 }
        ];

        function login() {
            let saved = localStorage.getItem('tg_elo_save');
            if(saved) gameData = JSON.parse(saved);
            document.getElementById('auth-screen').classList.remove('active-screen');
            document.getElementById('game-screen').classList.add('active-screen');
            updateUI();
        }

        function clickElo() {
            gameData.elo += gameData.clickElo;
            gameData.clicoins += gameData.clickCoins;
            gameData.totalClicks++;
            updateUI(); save();
        }

        function updateUI() {
            document.getElementById('elo-display').innerText = Math.floor(gameData.elo).toLocaleString();
            document.getElementById('coin-display').innerText = gameData.clicoins.toFixed(2) + " CLC";
            document.getElementById('u-name').innerText = gameData.nick;
            document.getElementById('u-id').innerText = gameData.id;
            document.getElementById('u-title').innerText = gameData.title;
            renderList(document.querySelector('.tab.active').id.replace('t-', ''));
        }

        function renderList(t) {
            const cont = document.getElementById('list-cont');
            cont.innerHTML = '';
            const reg = countries.find(c => c.id === gameData.region);

            if(t === 'upg') {
                cont.innerHTML = `<div class="card"><div><b>Клик +10 ELO</b><br><small>Цена: 5.00 CLC</small></div><button class="price-tag" style="background:var(--coin); color:black;" onclick="buyUpg()">КУПИТЬ</button></div>`;
            } else if(t === 'don') {
                const packs = [{clc: 100, star: 50}, {clc: 500, star: 200}, {clc: 2000, star: 750}];
                packs.forEach(p => {
                    cont.innerHTML += `<div class="card">
                        <div><b>${p.clc} CLC</b><br><small>За Telegram Stars</small></div>
                        <button class="price-tag" onclick="payStars(${p.star}, ${p.clc})">⭐ ${p.star}</button>
                    </div>`;
                });
            } else {
                cont.innerHTML = `<div class="card">Всего кликов: <b>${gameData.totalClicks}</b></div>`;
            }
        }

        function payStars(stars, reward) {
            tg.showConfirm(`Купить ${reward} CLC за ⭐ ${stars}?`, (ok) => {
                if(ok) {
                    tg.showAlert("Счет сформирован! Оплатите его в боте.");
                    // Имитация зачисления для теста
                    gameData.clicoins += reward; updateUI(); save();
                }
            });
        }

        function buyUpg() {
            if(gameData.clicoins >= 5) { gameData.clicoins -= 5; gameData.clickElo += 10; updateUI(); save(); }
        }

        function openSettings() { document.getElementById('settings-screen').classList.add('active-screen'); filterReg(); }
        function closeSettings() { document.getElementById('settings-screen').classList.remove('active-screen'); }
        
        function filterReg() {
            const s = document.getElementById('search-reg').value.toLowerCase();
            const list = document.getElementById('reg-list');
            list.innerHTML = '';
            countries.filter(c => c.n.toLowerCase().includes(s)).forEach(c => {
                let div = document.createElement('div');
                div.className = 'country-item';
                div.innerHTML = `<span>${c.n}</span><b>${c.s}</b>`;
                div.onclick = () => { gameData.region = c.id; updateUI(); save(); };
                list.appendChild(div);
            });
        }

        let aTaps = 0;
        function adminTaps() { aTaps++; if(aTaps >= 5) document.getElementById('admin-box').style.display='block'; }
        function checkAdmin() { if(document.getElementById('adm-key').value === '8989') document.getElementById('admin-tools').style.display='block'; }
        function save() { localStorage.setItem('tg_elo_save', JSON.stringify(gameData)); }
        function setTab(t) { document.querySelectorAll('.tab').forEach(b => b.classList.remove('active')); document.getElementById('t-'+t).classList.add('active'); renderList(t); }
    </script>
</body>
</html>
