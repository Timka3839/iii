<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Stickman's School - Alpha 0.1</title>
    <style>
        * {
            user-select: none;
            touch-action: manipulation;
        }
        body {
            margin: 0;
            min-height: 100vh;
            background: linear-gradient(135deg, #1a1f2e 0%, #0f1420 100%);
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Courier New', monospace;
        }
        .game {
            background: #0a0e17;
            border-radius: 32px;
            padding: 16px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.5);
            border: 1px solid #2d3e5f;
        }
        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 20px;
            box-shadow: 0 0 0 3px #2d3e5f;
            cursor: crosshair;
            touch-action: none;
            width: 100%;
            height: auto;
        }
        .info {
            display: flex;
            justify-content: space-between;
            margin-bottom: 12px;
            gap: 12px;
            flex-wrap: wrap;
        }
        .panel {
            background: rgba(0,0,0,0.6);
            backdrop-filter: blur(4px);
            padding: 6px 14px;
            border-radius: 40px;
            color: #ffd966;
            font-weight: bold;
            font-size: 1.1rem;
            border: 1px solid #ffd96644;
        }
        .controls {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 18px;
        }
        button {
            background: #2d3e5f;
            border: none;
            font-family: monospace;
            font-weight: bold;
            font-size: 1.2rem;
            color: white;
            padding: 10px 20px;
            border-radius: 60px;
            box-shadow: 0 4px 0 #0f1a2c;
            transition: 0.05s linear;
            touch-action: manipulation;
            cursor: pointer;
        }
        button:active {
            transform: translateY(2px);
            box-shadow: 0 1px 0 #0f1a2c;
        }
        .status {
            margin-top: 12px;
            text-align: center;
            color: #aaaaff;
            font-size: 0.85rem;
            background: #00000066;
            padding: 6px 12px;
            border-radius: 30px;
            backdrop-filter: blur(2px);
        }
        @media (max-width: 550px) {
            .panel { font-size: 0.8rem; padding: 4px 10px; }
            button { font-size: 1rem; padding: 8px 16px; }
        }
    </style>
</head>
<body>
<div class="game">
    <div class="info">
        <div class="panel">🎒 ТЕТРАДИ: <span id="scoreVal">0</span></div>
        <div class="panel">📏 ОШИБКИ: <span id="mistakesVal">0</span></div>
        <div class="panel">📖 СТИКМЕН</div>
    </div>
    <canvas id="gameCanvas" width="500" height="550" style="width:100%; height:auto; max-width:550px; aspect-ratio:500/550"></canvas>
    <div class="controls">
        <button id="restartBtn">🔄 ЗАНОВО</button>
    </div>
    <div class="status" id="statusMsg">👉 Ты в школе Стикмена. Собери 4 тетради.</div>
</div>

<script>
    (function(){
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreSpan = document.getElementById('scoreVal');
        const mistakesSpan = document.getElementById('mistakesVal');
        const statusDiv = document.getElementById('statusMsg');

        let cw = 500, ch = 550;
        canvas.width = cw;
        canvas.height = ch;

        // Игрок (прицел / крестик в центре)
        let player = { x: cw/2, y: ch/2, size: 10 };
        
        // Тетради
        let notebooks = [];
        let collected = 0;
        let mistakes = 0;
        let gameActive = true;
        
        // Дверь (кликабельная, не улетает)
        let door = { x: cw-80, y: ch-100, w: 50, h: 80 };
        
        // Стикмен (стоит в углу, иногда меняет выражение)
        let stickman = { x: 50, y: ch-100, size: 50, angry: false };
        
        // Печеньки / пельмени для поднятия духа
        let items = [];
        
        function initGame(){
            notebooks = [];
            collected = 0;
            mistakes = 0;
            gameActive = true;
            items = [];
            // генерируем 4 тетради в случайных местах (но не на игроке)
            for(let i=0;i<4;i++){
                let placed = false;
                while(!placed){
                    let rx = 60 + Math.random() * (cw-120);
                    let ry = 80 + Math.random() * (ch-180);
                    let distToPlayer = Math.hypot(rx - player.x, ry - player.y);
                    if(distToPlayer > 60){
                        notebooks.push({ x: rx, y: ry, w: 25, h: 30, collected: false });
                        placed = true;
                    }
                }
            }
            // пара пельменей для подкрепления
            for(let i=0;i<3;i++){
                items.push({ x: 80 + Math.random()*(cw-160), y: 100 + Math.random()*(ch-150), w: 20, h: 20, type: 'pelmen' });
            }
            updateUI();
            statusDiv.innerText = '📘 Найди 4 тетради и коснись двери!';
        }
        
        function updateUI(){
            scoreSpan.innerText = collected;
            mistakesSpan.innerText = mistakes;
            if(collected >= 4){
                statusDiv.innerText = '🚪 ТЕТРАДИ СОБРАНЫ! ИДИ К ДВЕРИ!';
            }
        }
        
        function checkCollisions(){
            if(!gameActive) return;
            // сбор тетрадей (пальцем / мышкой через canvas click)
            for(let i=0; i<notebooks.length; i++){
                let nb = notebooks[i];
                if(!nb.collected){
                    if(player.x > nb.x && player.x < nb.x+nb.w && player.y > nb.y && player.y < nb.y+nb.h){
                        nb.collected = true;
                        collected++;
                        updateUI();
                        statusDiv.innerText = '✅ Тетрадь найдена! Осталось ' + (4-collected);
                        if(collected === 4){
                            statusDiv.innerText = '🌟 Все тетради собраны! Беги к двери!';
                        }
                    }
                }
            }
            // сбор пельменей (лечат ошибки)
            for(let i=0;i<items.length;i++){
                let it = items[i];
                if(!it.collected && player.x > it.x && player.x < it.x+it.w && player.y > it.y && player.y < it.y+it.h){
                    it.collected = true;
                    if(mistakes > 0){
                        mistakes--;
                        updateUI();
                        statusDiv.innerText = '🥟 Пельмень! Ошибки уменьшены!';
                    } else {
                        statusDiv.innerText = '🥟 Пельмень съеден, но ошибок и не было.';
                    }
                }
            }
            // дверь: победа, если собраны тетради
            if(collected >= 4 && gameActive){
                if(player.x > door.x && player.x < door.x+door.w && player.y > door.y && player.y < door.y+door.h){
                    gameActive = false;
                    statusDiv.innerText = '🏆 ПОБЕДА! Ты выбрался из школы Стикмена! 🏆';
                    statusDiv.style.color = '#aaffaa';
                }
            }
        }
        
        // перемещение игрока (по касанию / мыши)
        function movePlayerTo(e){
            if(!gameActive) return;
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            let clientX, clientY;
            if(e.touches){
                clientX = e.touches[0].clientX;
                clientY = e.touches[0].clientY;
            } else {
                clientX = e.clientX;
                clientY = e.clientY;
            }
            let canvasX = (clientX - rect.left) * scaleX;
            let canvasY = (clientY - rect.top) * scaleY;
            if(canvasX >= 0 && canvasX <= canvas.width && canvasY >= 0 && canvasY <= canvas.height){
                player.x = canvasX;
                player.y = canvasY;
                checkCollisions();
            }
            e.preventDefault();
        }
        
        // отрисовка всего
        function draw(){
            ctx.clearRect(0,0,cw,ch);
            // стены школы
            ctx.fillStyle = '#1a2a3a';
            ctx.fillRect(0,0,cw,ch);
            ctx.fillStyle = '#3a4a5a';
            for(let i=0;i<10;i++){
                ctx.fillRect(i*60, ch-40, 30, 8);
            }
            // тетради
            for(let nb of notebooks){
                if(!nb.collected){
                    ctx.fillStyle = '#8B5A2B';
                    ctx.fillRect(nb.x, nb.y, nb.w, nb.h);
                    ctx.fillStyle = '#FFD966';
                    ctx.font = 'bold 18px monospace';
                    ctx.fillText('📘', nb.x+4, nb.y+22);
                }
            }
            // пельмени
            for(let it of items){
                if(!it.collected){
                    ctx.fillStyle = '#E8C39E';
                    ctx.beginPath();
                    ctx.ellipse(it.x+10, it.y+10, 10, 7, 0, 0, Math.PI*2);
                    ctx.fill();
                    ctx.fillStyle = '#A57C4C';
                    ctx.fillRect(it.x+6, it.y+8, 8, 4);
                }
            }
            // дверь
            ctx.fillStyle = '#6B4226';
            ctx.fillRect(door.x, door.y, door.w, door.h);
            ctx.fillStyle = '#D4AF37';
            ctx.fillRect(door.x+door.w-12, door.y+door.h/2-6, 8, 12);
            ctx.fillStyle = '#FFFFFF';
            ctx.font = 'bold 14px monospace';
            ctx.fillText('🚪', door.x+15, door.y+50);
            
            // Стикмен (безликий, но меняет цвет при ошибках)
            ctx.fillStyle = '#F0F0E0';
            ctx.fillRect(stickman.x, stickman.y, stickman.size, stickman.size);
            ctx.fillStyle = mistakes >= 2 ? '#FF8888' : '#444444';
            ctx.fillRect(stickman.x+10, stickman.y+12, 8, 8);
            ctx.fillRect(stickman.x+30, stickman.y+12, 8, 8);
            if(mistakes >= 3){
                ctx.fillStyle = '#FF0000';
                ctx.fillRect(stickman.x+15, stickman.y+32, 20, 6);
            }
            
            // крестик-прицел (игрок)
            ctx.beginPath();
            ctx.arc(player.x, player.y, 12, 0, Math.PI*2);
            ctx.strokeStyle = '#FFD966';
            ctx.lineWidth = 3;
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo(player.x-8, player.y);
            ctx.lineTo(player.x+8, player.y);
            ctx.moveTo(player.x, player.y-8);
            ctx.lineTo(player.x, player.y+8);
            ctx.stroke();
        }
        
        function restart(){
            initGame();
            gameActive = true;
            statusDiv.style.color = '#aaaaff';
            statusDiv.innerText = '👉 Игра перезапущена. Собери 4 тетради.';
            draw();
        }
        
        // события
        canvas.addEventListener('touchstart', movePlayerTo);
        canvas.addEventListener('mousedown', movePlayerTo);
        document.getElementById('restartBtn').addEventListener('click', () => { restart(); draw(); });
        
        initGame();
        setInterval(() => { if(gameActive) draw(); else draw(); }, 30);
        draw();
    })();
</script>
</body>
</html> # iii
crazy
