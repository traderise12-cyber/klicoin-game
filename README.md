<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ELO CLICKER: INVENTORY</title>
    <style>
        :root { --main: #00ffcc; --coin: #ffcc00; --bg: #0a0a0a; --card: #161616; --rare: #b026ff; }
        * { box-sizing: border-box; outline: none; -webkit-tap-highlight-color: transparent; }
        body { background: var(--bg); color: white; font-family: sans-serif; margin: 0; user-select: none; overflow: hidden; touch-action: none; display: flex; flex-direction: column; height: 100vh; }

        .screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; display: none; flex-direction: column; z-index: 100; background: var(--bg); }
        .active-screen { display: flex; }

        /* Верх */
        .game-top { flex: 0 0 45%; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; border-bottom: 2px solid #1a1a1a; background: radial-gradient(circle at center, #1a1a1a 0%, #0a0a0a 100%); }
        .profile-bar { position: absolute; top: 15px; left: 15px; display: flex; align-items: center; gap: 10px; }
        #u-avatar { width: 45px; height: 45px; border-radius: 50%; object-fit: cover; border: 2px solid var(--main); background: #222; }
        .user-info { display: flex; flex-direction: column; text-align: left; }
        #u-name { font-weight: bold; font-size: 0.9rem; background-clip: text; -webkit-background-clip: text; }
        #u-title { font-size: 0.6rem; color: var(--rare); font-weight: bold; text-transform: uppercase; }

        .settings-btn { position: absolute; top: 15px; right: 15px; background: none; border: none; color: white; font-size: 1.5rem; }
        #elo-display { font-size: 2.5rem; color: var(--main); font-weight: bold; margin: 0; }
        #coin-display { font-size: 1.1rem; color: var(--coin); font-weight: bold; }

        #main-btn { width: 120px; height: 120px; border-radius: 50%; border: 6px solid #111; background: var(--main); font-size: 1.3rem; font-weight: bold; transition: 0.1s; }
        #main-btn:active { transform: scale(0.9); }

        /* Меню */
        .bottom-area { flex: 1; display: flex; flex-direction: column; background: #0f0f0f; }
        .tabs { display: flex; background: #111; border-bottom: 1px solid #222; }
        .tab { flex: 1; padding: 15px; border: none; background: none; color: #444; font-weight: bold; font-size: 0.65rem; text-transform: uppercase; }
        .tab.active { color: var(--main); box-shadow: inset 0 -2px 0 var(--main); }
        .list { flex: 1; overflow-y: auto; padding: 15px; }
        .card { background: var(--card); margin-bottom: 10px; padding: 15px; border-radius: 12px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #222; }

        /* Админка */
        .ui-input { width: 100%; padding: 10px; margin: 5px 0; border-radius: 8px; border: 1px solid #333; background: #000; color: white; font-size: 0.85rem; }
        .btn-ui { background: var(--main); color: black; border: none; padding: 10px; border-radius: 8px; font-weight: bold; width: 100%; cursor: pointer; margin-top: 5px; }
        .skin-preview { width: 30px; height: 30px; border-radius: 50%; border: 1px solid #fff; }
    </style>
</head>
<body>

    <div id="game-screen" class="screen active-screen">
        <div class="game-top">
            <div class="profile-bar">
                <img id="u-avatar" src="https://placeholder.com">
                <div class="user-info">
                    <span id="u-name">Player</span>
                    <span id="u-title">НОВИЧОК</span>
                </div>
            </div>
            <button class="settings-btn" onclick="openSettings()">⚙️</button>
            <p id="elo-display">0</p>
            <p id="coin-display">0.00 CLC</p>
            <button id="main-btn" onclick="clickElo()">TAP!</button>
        </div>

        <div class="bottom-area">
            <div class="tabs">
                <button id="t-upg" class="tab active" onclick="setTab('upg')">Рынок</button>
                <button id="t-inv" class="tab" onclick="setTab('inv')">Инвентарь</button>
            </div>
            <div id="list-cont" class="list"></div>
        </div>
    </div>

    <!-- НАСТРОЙКИ -->
    <div id="settings-screen" class="screen" style="padding: 20px; overflow-y: auto;">
        <div style="background:#111; padding:20px; border-radius:15px; width:100%;">
            <h2 onclick="adminTaps()">Настройки</h2>
            <input type="text" id="edit-nick" class="ui-input" placeholder="Ник" onchange="gameData.nick=this.value; updateUI(); save()">
            
            <label style="font-size:0.7rem; color:orange; margin-top:10px; display:block">ПРОМОКОД</label>
            <div style="display:flex; gap:5px;"><input type="text" id="promo-in" class="ui-input"><button onclick="usePromo()" class="btn-ui" style="width:60px; margin:0">OK</button></div>

            <div id="admin-auth" style="display:none; margin-top:15px;">
                <input type="password" id="adm-pass" class="ui-input" placeholder="Ключ">
                <button onclick="unlockAdmin()" class="btn-ui">ВОЙТИ</button>
            </div>

            <div id="admin-panel" style="display:none; border-top:1px dashed orange; margin-top:20px; padding-top:10px;">
                <p style="color:orange; font-weight:bold; font-size:0.8rem;">ГЕНЕРАТОР ПРОМО</p>
                <input type="text" id="p-name" class="ui-input" placeholder="Название">
                <input type="number" id="p-elo" class="ui-input" placeholder="+ELO">
                <select id="p-skin" class="ui-input">
                    <option value="">Без скина</option>
                    <option value="#ff4444">Красный</option>
                    <option value="#ffcc00">Золотой</option>
                    <option value="linear-gradient(45deg, #00f, #f0f)">Неон</option>
                </select>
                <button onclick="createPromo()" class="btn-ui" style="background:orange">СОЗДАТЬ</button>
            </div>
            <button class="btn-ui" style="background:#333; color:white; margin-top:20px;" onclick="closeSettings()">ЗАКРЫТЬ</button>
        </div>
    </div>

    <script>
        let gameData = { 
            elo: 0, clicoins: 0, click: 10, nick: 'Player', title: 'НОВИЧОК', 
            avatar: 'https://placeholder.com', 
            inventory: [{name: 'Стандарт', color: '#00ffcc'}], 
            activeSkin: '#00ffcc', used: [] 
        };
        let promos = JSON.parse(localStorage.getItem('promo_v4')) || {};
        let aTaps = 0, currentTab = 'upg';

        window.onload = () => {
            const saved = localStorage.getItem('elo_inv_save');
            if(saved) gameData = JSON.parse(saved);
            updateUI(); setTab('upg');
        };

        function clickElo() { gameData.elo += gameData.click; gameData.clicoins += 0.05; updateUI(); save(); }

        function setTab(t) {
            currentTab = t;
            document.querySelectorAll('.tab').forEach(b => b.classList.remove('active'));
            document.getElementById('t-'+t).classList.add('active');
            renderList(t);
        }

        function renderList(t) {
            const cont = document.getElementById('list-cont');
            cont.innerHTML = '';
            if(t === 'upg') {
                cont.innerHTML = `<div class="card"><b>Сила клика</b><button onclick="buy()" class="btn-ui" style="width:80px">10 CLC</button></div>`;
            } else if(t === 'inv') {
                gameData.inventory.forEach((item, index) => {
                    let isActive = gameData.activeSkin === item.color;
                    cont.innerHTML += `
                        <div class="card">
                            <div style="display:flex; align-items:center; gap:10px;">
                                <div class="skin-preview" style="background:${item.color}"></div>
                                <b>${item.name}</b>
                            </div>
                            <button onclick="equipSkin(${index})" class="btn-ui" style="width:80px; background:${isActive?'#555':''}" ${isActive?'disabled':''}>
                                ${isActive ? 'Надет' : 'Надеть'}
                            </button>
                        </div>`;
                });
            }
        }

        function equipSkin(i) {
            gameData.activeSkin = gameData.inventory[i].color;
            updateUI(); save();
        }

        function buy() { if(gameData.clicoins >= 10) { gameData.clicoins -= 10; gameData.click += 20; updateUI(); save(); } }

        function createPromo() {
            const n = document.getElementById('p-name').value.toUpperCase();
            if(n) { promos[n] = { elo: parseInt(document.getElementById('p-elo').value)||0, skin: document.getElementById('p-skin').value }; localStorage.setItem('promo_v4', JSON.stringify(promos)); alert("Создан!"); }
        }

        function usePromo() {
            const c = document.getElementById('promo-in').value.toUpperCase();
            const p = promos[c];
            if(p && !gameData.used.includes(c)) {
                gameData.elo += p.elo;
                if(p.skin) {
                    let skinName = p.skin.includes('gradient') ? 'Неон' : 'Редкий';
                    gameData.inventory.push({name: skinName, color: p.skin});
                }
                gameData.used.push(c);
                updateUI(); save(); alert("Успешно!");
            } else { alert("Ошибка!"); }
        }

        function updateUI() {
            document.getElementById('elo-display').innerText = Math.floor(gameData.elo).toLocaleString();
            document.getElementById('coin-display').innerText = gameData.clicoins.toFixed(2) + " CLC";
            document.getElementById('u-name').innerText = gameData.nick;
            document.getElementById('main-btn').style.background = gameData.activeSkin;
            renderList(currentTab);
        }

        function save() { localStorage.setItem('elo_inv_save', JSON.stringify(gameData)); }
        function openSettings() { document.getElementById('settings-screen').classList.add('active-screen'); }
        function closeSettings() { document.getElementById('settings-screen').classList.remove('active-screen'); aTaps=0; document.getElementById('admin-auth').style.display='none'; document.getElementById('admin-panel').style.display='none'; }
        function adminTaps() { aTaps++; if(aTaps >= 5) document.getElementById('admin-auth').style.display='block'; }
        function unlockAdmin() { if(document.getElementById('adm-pass').value==='8989') document.getElementById('admin-panel').style.display='block'; }
    </script>
</body>
</html>
