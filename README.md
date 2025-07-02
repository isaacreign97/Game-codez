<!DOCTYPE html>
<html lang="en">
<head>
    <title>Battleship Game - Enhanced</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <style>
        :root { --cell: 40px; --cell-sm: 32px; }
        html, body { height: 100%; margin: 0; }
        body {
            font-family: Arial, sans-serif; margin:0; min-height:100vh;
            background: linear-gradient(160deg, #92d2fc 0%, #1e90ff 100%);
        }
        .main-bg {
            background: url('https://www.transparenttextures.com/patterns/waves.png'), linear-gradient(160deg, #92d2fc 0%, #1e90ff 100%);
            min-height:100vh;
            background-blend-mode: multiply;
        }
        .center { text-align: center; margin-top: 60px;}
        .menu-btn, .mode-btn, .size-btn, .back-btn, .restart-btn {
            padding: 16px 36px; margin: 18px 12px; border-radius: 10px;
            border: none; font-size: 22px; cursor: pointer;
            background: #3867d6; color: #fff; transition: 0.18s;
            box-shadow: 0 2px 12px #009aff33;
        }
        .menu-btn:hover, .mode-btn:hover, .size-btn:hover, .back-btn:hover, .restart-btn:hover { background: #2d98da; }
        .howtoplay-box { width: 520px; margin: 30px auto 0 auto; padding: 26px; background: #fff8; border-radius: 15px; box-shadow: 0 2px 16px #0002; text-align:left;}
        .game-area {
            display: flex;
            justify-content: center;
            align-items: flex-start;
            gap: 48px;
            margin-top: 24px;
        }
        .board {
            background: #eaf6ffee;
            padding: 20px 22px 18px 22px;
            border-radius: 16px;
            box-shadow: 0 2px 18px #1e90ff44;
            display: inline-block;
            min-width: unset;
        }
        .grid-wrap {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .labels {
            display: grid;
            grid-template-columns: var(--cell) repeat(var(--grid-size), var(--cell));
            width: fit-content;
        }
        .label-cell {
            width: var(--cell);
            height: var(--cell);
            text-align: center;
            line-height: var(--cell);
            font-weight: bold;
            font-size: 18px;
            color: #222a;
            user-select: none;
            box-sizing: border-box;
        }
        .row { display: flex; }
        .numbers {
            display: flex;
            flex-direction: column;
            margin-right: 0px;
        }
        .number-cell {
            width: var(--cell);
            height: var(--cell);
            text-align: center;
            line-height: var(--cell);
            font-weight: bold;
            font-size: 18px;
            color: #222a;
            user-select: none;
            box-sizing: border-box;
        }
        .grid {
            display: flex;
            flex-direction: column;
        }
        .cell {
            width: var(--cell); height: var(--cell);
            border: 1.5px solid #45aaf2;
            box-sizing: border-box;
            display: flex; align-items: center; justify-content: center;
            font-weight: bold; font-size: 18px; cursor: pointer;
            background: #c7ecee;
            position: relative;
            transition: background 0.15s, box-shadow 0.2s, border 0.1s;
            outline: none;
        }
        .cell:focus { outline: 2px solid #0984e3; }
        .cell.placement { background: #fff6b2 !important; border: 2.5px solid #ffb300 !important; }
        .cell.ship { background: #54a0ff !important; }
        .cell.ship.sunk { background: #c2c2c2 !important; animation: sunkAnim 0.5s; }
        @keyframes sunkAnim {
            from { box-shadow: 0 0 0 0 #ff7675; }
            to   { box-shadow: 0 0 18px 10px #ff767588; }
        }
        .cell.computer { cursor: not-allowed; }
        .cell.hit { background: #ff7979 !important; color: white;}
        .cell.miss { background: #b2bec3 !important; color: #fff; }
        .cell svg { pointer-events: none; }
        .cell.player-hit { background: #30336b !important;}
        .cell.targetable:not(.hit):not(.miss):hover { background: #81ecec !important; box-shadow: 0 0 0 3px #00cec988; }
        .cell.selected { border: 2.5px solid #0be881 !important; background: #a3e9e7 !important; }
        .cell.last-move { animation: lastmoveflash 0.8s; border: 2.5px solid #f0932b !important; }
        @keyframes lastmoveflash {
            from { box-shadow: 0 0 0 6px #fdcb6e77; }
            to   { box-shadow: none; }
        }
        .confetti { position:fixed; pointer-events:none; z-index:99; }
        .statbox {
            margin: 12px auto 0 auto;
            background: #fff8;
            border-radius: 10px;
            padding: 8px 24px 8px 24px;
            width: 350px;
            font-size: 16px;
            color: #333;
            box-shadow: 0 2px 12px #3867d633;
        }
        .ships-remaining {
            margin: 8px 0;
            font-size: 17px;
            color: #144b5d;
            font-weight: bold;
        }
        #salvoInfo { margin-top: 10px; color:#3867d6; font-size:15px;}
        #status { margin-top: 18px; font-weight: bold; min-height: 26px; color:#145;}
        .logo-wave {
            width: 110px; margin-bottom: 8px;
            filter: drop-shadow(0 2px 12px #5ed6ff88);
        }
        .placement-hint { color:#ffb300; font-weight:bold; margin:10px 0; }
        @media (max-width: 900px) {
            .game-area { flex-direction:column; gap: 18px; }
        }
        @media (max-width: 520px) {
            .howtoplay-box { width: 99vw; font-size: 14px; padding:8px;}
            .statbox { width:98vw;}
            .logo-wave { width: 60vw; }
        }
        @media (max-width: 500px) {
            :root { --cell: var(--cell-sm); }
        }
    </style>
</head>
<body class="main-bg">
<div id="mainMenu" class="center" aria-label="Main menu">
    <svg class="logo-wave" viewBox="0 0 60 16"><path fill="#1e90ff" d="M0 11c7 0 7-9 14-9s7 9 14 9 7-9 14-9 7 9 14 9v5H0z"/></svg>
    <h1 style="letter-spacing:2px;color:#0a3d62;">Battleship Game</h1>
    <button class="menu-btn" onclick="showModeMenu()" aria-label="New Game">New Game</button>
    <button class="menu-btn" onclick="showHowToPlay()" aria-label="How to Play">How to Play</button>
    <div class="statbox" id="statBox"></div>
</div>
<div id="howToPlay" class="center hidden" aria-label="How to Play Instructions">
    <div class="howtoplay-box">
        <h2>How to Play</h2>
        <b>Basic Gameplay:</b>
        <ul>
            <li>Sink all of your opponent's ships.</li>
            <li>Take turns firing by clicking cells on your target board.</li>
            <li>Red ✖ = hit, Gray ● = miss, Blue = your ships.</li>
        </ul>
        <b>Advanced (Salvo) Gameplay:</b>
        <ul>
            <li>Shots per turn = your remaining ships.</li>
            <li>Win by sinking all enemy ships.</li>
        </ul>
        <b>Ship Placement:</b>
        <ul>
            <li>Click to place your ships before the game starts. Use the rotate button to change orientation.</li>
            <li>Ships cannot touch each other, even diagonally.</li>
        </ul>
        <b>Accessibility:</b>
        <ul>
            <li>Use Tab to navigate between cells and Enter/Space to select.</li>
        </ul>
        <button class="back-btn" onclick="showMainMenu()">Back to Main Menu</button>
    </div>
</div>
<div id="modeMenu" class="center hidden" aria-label="Game mode selection">
    <svg class="logo-wave" viewBox="0 0 60 16"><path fill="#1e90ff" d="M0 11c7 0 7-9 14-9s7 9 14 9 7-9 14-9 7 9 14 9v5H0z"/></svg>
    <h2 style="color:#145;">Select Game Mode</h2>
    <button class="mode-btn" onclick="chooseMode('basic')" aria-label="Basic Mode">Basic Gameplay</button>
    <button class="mode-btn" onclick="chooseMode('salvo')" aria-label="Salvo Mode">Advanced Gameplay (Salvo)</button>
    <br><br>
    <button class="back-btn" onclick="showMainMenu()">Back to Main Menu</button>
</div>
<div id="difficultyMenu" class="center hidden" aria-label="Difficulty selection">
    <svg class="logo-wave" viewBox="0 0 60 16"><path fill="#1e90ff" d="M0 11c7 0 7-9 14-9s7 9 14 9 7-9 14-9 7 9 14 9v5H0z"/></svg>
    <h2 style="color:#145;">Choose Difficulty</h2>
    <button class="mode-btn" onclick="chooseDifficulty('easy')">Easy</button>
    <button class="mode-btn" onclick="chooseDifficulty('hard')">Hard</button>
    <br><br>
    <button class="back-btn" onclick="showDifficultyMenu()">Back</button>
</div>
<div id="sizeMenu" class="center hidden" aria-label="Board size selection">
    <svg class="logo-wave" viewBox="0 0 60 16"><path fill="#1e90ff" d="M0 11c7 0 7-9 14-9s7 9 14 9 7-9 14-9 7 9 14 9v5H0z"/></svg>
    <h2 style="color:#145;">Choose Board Size</h2>
    <button class="size-btn" onclick="chooseBoardSize(5)">5 x 5 Board</button>
    <button class="size-btn" onclick="chooseBoardSize(10)">10 x 10 Board</button>
    <br><br>
    <button class="back-btn" onclick="showDifficultyMenu()">Back</button>
</div>
<div id="placementUI" class="center hidden" aria-label="Ship placement interface">
    <svg class="logo-wave" viewBox="0 0 60 16"><path fill="#1e90ff" d="M0 11c7 0 7-9 14-9s7 9 14 9 7-9 14-9 7 9 14 9v5H0z"/></svg>
    <h2 style="color:#145;">Place Your Ships</h2>
    <button class="restart-btn" onclick="showMainMenu()">Main Menu</button>
    <button class="restart-btn" onclick="restartPlacement()">Restart Placement</button>
    <div class="placement-hint" id="placementHint"></div>
    <button class="mode-btn" style="font-size:18px;padding:10px 24px;" onclick="togglePlacementOrientation()" aria-label="Rotate Ship">Rotate Ship</button>
    <div style="display:flex;justify-content:center;">
        <div class="board" id="placementBoard"></div>
    </div>
</div>
<div id="gameUI" class="center hidden" aria-label="Game interface">
    <svg class="logo-wave" viewBox="0 0 60 16"><path fill="#1e90ff" d="M0 11c7 0 7-9 14-9s7 9 14 9 7-9 14-9 7 9 14 9v5H0z"/></svg>
    <h2 style="color:#145;">Battleship Game</h2>
    <button class="restart-btn" onclick="showMainMenu()">Main Menu</button>
    <button class="restart-btn" onclick="restartGame()">Restart Game</button>
    <div class="game-area">
        <div class="board">
            <div class="board-title">Your Target Board</div>
            <div class="ships-remaining" id="enemyShipsRemaining"></div>
            <div class="grid-wrap">
                <div class="labels" id="targetColLabels"></div>
                <div style="display:flex;">
                    <div class="numbers" id="targetRowLabels"></div>
                    <div class="grid" id="targetBoard"></div>
                </div>
            </div>
        </div>
        <div class="board">
            <div class="board-title">Your Ocean Board</div>
            <div class="ships-remaining" id="playerShipsRemaining"></div>
            <div class="grid-wrap">
                <div class="labels" id="oceanColLabels"></div>
                <div style="display:flex;">
                    <div class="numbers" id="oceanRowLabels"></div>
                    <div class="grid" id="oceanBoard"></div>
                </div>
            </div>
        </div>
    </div>
    <div id="status"></div>
    <div id="salvoInfo"></div>
    <div class="statbox" id="statBox2"></div>
</div>
<script>
/* ========== CONFETTI ANIMATION ========== */
function confetti() {
    for(let i=0;i<90;i++){
        let div=document.createElement('div');
        div.className='confetti';
        div.style.left=(Math.random()*100)+'vw';
        div.style.top='-10px';
        let sz=6+Math.random()*10;
        div.style.width=sz+'px';
        div.style.height=sz+'px';
        div.style.backgroundColor=`hsl(${Math.random()*360},95%,60%)`;
        div.style.opacity=Math.random()*0.7+0.3;
        div.style.borderRadius=Math.random()>0.5?'50%':'20%';
        div.style.position='fixed';
        div.style.animation=`fall ${1+Math.random()*1.4}s linear ${Math.random()}s 1`;
        div.style.pointerEvents='none';
        document.body.appendChild(div);
        div.addEventListener('animationend',()=>div.remove());
    }
}
let style=document.createElement('style');
style.innerHTML='@keyframes fall{to{transform:translateY(110vh) rotate(360deg);}}';document.head.appendChild(style);

/* ========== SIMPLE BEEP SOUNDS ========== */
function beep(type){
    try {
        let ctx=new(window.AudioContext||window.webkitAudioContext)();
        let o=ctx.createOscillator(),g=ctx.createGain();
        o.connect(g);g.connect(ctx.destination);
        g.gain.value=0.05;
        o.type='sine';
        if(type==='hit') o.frequency.value=220;
        else if(type==='miss') o.frequency.value=350;
        else if(type==='place') o.frequency.value=530;
        else o.frequency.value=880;
        o.start(0);o.stop(ctx.currentTime+0.18);
        setTimeout(()=>ctx.close(),250);
    }catch{}
}

/* ========== UI NAVIGATION & GLOBAL STATE ========== */
let selectedMode=null, selectedBoardSize=10, selectedDifficulty='hard';
function showMainMenu(){ hideAll(); updateStats(); document.getElementById('mainMenu').classList.remove('hidden'); }
function showModeMenu(){ hideAll(); document.getElementById('modeMenu').classList.remove('hidden'); }
function showDifficultyMenu(){ hideAll(); document.getElementById('difficultyMenu').classList.remove('hidden'); }
function showSizeMenu(){ hideAll(); document.getElementById('sizeMenu').classList.remove('hidden'); }
function showHowToPlay(){ hideAll(); document.getElementById('howToPlay').classList.remove('hidden'); }
function showGameUI(){ hideAll(); document.getElementById('gameUI').classList.remove('hidden'); }
function showPlacementUI(){ hideAll(); document.getElementById('placementUI').classList.remove('hidden'); }
function hideAll(){
    ['mainMenu','howToPlay','modeMenu','difficultyMenu','gameUI','sizeMenu','placementUI'].forEach(id=>{
        document.getElementById(id).classList.add('hidden');
    });
}

/* ========== GAME CONFIGS ========== */
let gridSize, ships, shipConfigs;
let targetBoard, oceanBoard, computerShips, playerShips, playerShipPositions, computerShipPositions;
let computerHits, playerHits, computerGuesses, gameOver;
let gameMode = "basic";
let gameDifficulty = 'hard';
let salvoShots = 5, salvoComputerShots = 5, playerTurnActive = true;
let salvoShotsFiredThisTurn = 0, availableShotsThisTurn = [];
let placementStep=0, placementState, placementOrientation=true;
let stats={played:0,won:0,lost:0};

function updateStats(){
    try{stats=JSON.parse(localStorage.battleStats)||stats;}catch{}
    document.getElementById('statBox').innerHTML =
      `<b>Stats:</b> Games: ${stats.played} | Wins: ${stats.won} | Losses: ${stats.lost}`;
    let e = document.getElementById('statBox2');
    if(e) e.innerHTML = `<b>Games:</b> ${stats.played} | <b>Wins:</b> ${stats.won} | <b>Losses:</b> ${stats.lost}`;
}
function saveStats(){
    try{localStorage.battleStats=JSON.stringify(stats);}catch{}
}

/* ========== SHIP SETTINGS & PLACEMENT ========== */
function setupConfigs() {
    gridSize = selectedBoardSize;
    if(gridSize===5){
        ships = 3;
        shipConfigs = [
            [3,true],
            [2,true],
            [2,false],
        ];
    } else {
        ships = 5;
        shipConfigs = [
            [5,true],
            [4,true],
            [3,true],
            [3,false],
            [2,true],
        ];
    }
    document.documentElement.style.setProperty('--grid-size', gridSize);
}
function restartPlacement(){
    placementStep=0;
    placementState=Array.from({length:gridSize},()=>Array(gridSize).fill(0));
    placementOrientation=true;
    showPlacementUI();
    drawPlacementBoard();
}
function chooseMode(mode) {
    selectedMode = mode;
    showDifficultyMenu();
}
function chooseDifficulty(diff) {
    selectedDifficulty = diff;
    showSizeMenu();
}
function chooseBoardSize(size) {
    selectedBoardSize = size;
    setupConfigs();
    restartPlacement();
}
function togglePlacementOrientation(){
    placementOrientation=!placementOrientation;
    drawPlacementBoard();
}
function canPlaceShipSafe(grid, x, y, length, isHorizontal) {
    // Must not touch other ships, even diagonally
    for(let i=0;i<length;i++){
        let nx=x+(isHorizontal?0:i), ny=y+(isHorizontal?i:0);
        if(nx<0||ny<0||nx>=gridSize||ny>=gridSize) return false;
        if(grid[nx][ny]!==0) return false;
        for(let dx=-1;dx<=1;dx++)
        for(let dy=-1;dy<=1;dy++){
            let cx=nx+dx,cy=ny+dy;
            if(cx>=0&&cy>=0&&cx<gridSize&&cy<gridSize)
                if(grid[cx][cy]!==0 && (dx!==0||dy!==0)) return false;
        }
    }
    return true;
}
function drawPlacementBoard(){
    let html='<div style="margin-bottom:8px;">';
    let step=placementStep, cfg=shipConfigs[step];
    document.getElementById('placementHint').innerHTML = 
        `Place ship #${step+1} (length: ${cfg[0]}, ${placementOrientation?'horizontal':'vertical'})`;
    html+='<div class="labels">';
    html+='<div class="label-cell"></div>';
    let alpha="ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    for(let i=0;i<gridSize;i++) html+=`<div class="label-cell">${alpha[i]}</div>`;
    html+='</div>';
    html+='<div style="display:flex;">';
    html+='<div class="numbers">';
    for(let i=0;i<gridSize;i++) html+=`<div class="number-cell">${i+1}</div>`;
    html+='</div>';
    html+='<div class="grid">';
    for(let x=0;x<gridSize;x++){
        html+='<div class="row">';
        for(let y=0;y<gridSize;y++){
            let cl="cell";
            let placed=placementState[x][y];
            // show as ship
            if(placed) cl+=" ship";
            // highlight for possible place
            if(canPlaceShipSafe(placementState,x,y,cfg[0],placementOrientation))
                cl+=" placement";
            html+=`<div class="${cl}" tabindex="0" aria-label="Cell ${alpha[y]}${x+1}" onclick="placePlayerShip(${x},${y})"></div>`;
        }
        html+='</div>';
    }
    html+='</div></div></div>';
    document.getElementById('placementBoard').innerHTML=html;
}
function placePlayerShip(x,y){
    let step=placementStep,cfg=shipConfigs[step];
    if(!canPlaceShipSafe(placementState,x,y,cfg[0],placementOrientation)) { beep('miss'); return; }
    for(let i=0;i<cfg[0];i++){
        let nx=x+(placementOrientation?0:i),ny=y+(placementOrientation?i:0);
        placementState[nx][ny]=step+1;
    }
    placementStep++;
    beep('place');
    if(placementStep>=shipConfigs.length){
        setTimeout(startGame, 120);
    }else{
        drawPlacementBoard();
    }
}

/* ========== COMPUTER SHIP PLACEMENT ========== */
function canPlaceShip(grid,x,y,len,isHor){
    for(let i=0;i<len;i++){
        let nx=x+(isHor?0:i),ny=y+(isHor?i:0);
        if(nx<0||ny<0||nx>=gridSize||ny>=gridSize) return false;
        if(grid[nx][ny]!==0) return false;
        for(let dx=-1;dx<=1;dx++)
        for(let dy=-1;dy<=1;dy++){
            let cx=nx+dx,cy=ny+dy;
            if(cx>=0&&cy>=0&&cx<gridSize&&cy<gridSize)
                if(grid[cx][cy]!==0 && (dx!==0||dy!==0)) return false;
        }
    }
    return true;
}
function placeShipsRandomly(grid) {
    let positions = [];
    for (let s = 0; s < shipConfigs.length; s++) {
        let [length, defaultHorizontal] = shipConfigs[s];
        let isHorizontal = Math.random()>0.5?true:false;
        let placed = false, tries=0;
        while (!placed && tries < 700) {
            let x = Math.floor(Math.random() * (gridSize - (isHorizontal ? 0 : length - 1)));
            let y = Math.floor(Math.random() * (gridSize - (isHorizontal ? length - 1 : 0)));
            if (canPlaceShip(grid, x, y, length, isHorizontal)) {
                let shipCells = [];
                for (let i = 0; i < length; i++) {
                    let nx = x + (isHorizontal ? 0 : i);
                    let ny = y + (isHorizontal ? i : 0);
                    grid[nx][ny] = s + 1;
                    shipCells.push([nx, ny]);
                }
                positions.push({cells: shipCells, sunk:false, length:length, id:s+1, isHorizontal});
                placed = true;
            }
            tries++;
        }
    }
    return positions;
}

/* ========== GAME START & SETUP ========== */
function startGame() {
    setupConfigs();
    gameMode = selectedMode;
    gameDifficulty = selectedDifficulty;
    targetBoard = Array.from({length:gridSize},()=>Array(gridSize).fill(0));
    oceanBoard = Array.from({length:gridSize},()=>Array(gridSize).fill(0));
    computerShips = Array.from({length:gridSize},()=>Array(gridSize).fill(0));
    playerShips = Array.from({length:gridSize},()=>Array(gridSize).fill(0));
    // Use player placementState for player ships
    playerShipPositions=[];
    for(let s=0;s<shipConfigs.length;s++){
        let cells=[];
        for(let x=0;x<gridSize;x++)
        for(let y=0;y<gridSize;y++)
            if(placementState[x][y]===s+1) { playerShips[x][y]=s+1; cells.push([x,y]); }
        playerShipPositions.push({cells:safeCopy(cells),sunk:false,length:shipConfigs[s][0],id:s+1,isHorizontal:shipConfigs[s][1]});
    }
    computerShipPositions = placeShipsRandomly(computerShips);
    computerHits=0; playerHits=0; computerGuesses=[];
    gameOver=false; playerTurnActive=true;
    salvoShots=ships; salvoComputerShots=ships;
    salvoShotsFiredThisTurn=0; availableShotsThisTurn=[];
    showGameUI(); drawBoards(); drawLabels();
    if(gameMode==="salvo") document.getElementById('salvoInfo').textContent = `Salvo Mode: You have ${salvoShots} shots this turn.`;
    else document.getElementById('salvoInfo').textContent="";
    document.getElementById('status').textContent = `Your turn! (${gameDifficulty.charAt(0).toUpperCase()+gameDifficulty.slice(1)} AI) Click a cell on the Target Board to fire.`;
    updateShipsRemaining();
    updateStats();
}
function safeCopy(arr){ return JSON.parse(JSON.stringify(arr)); }

/* ========== DRAW LABELS & SHIPS REMAINING ========== */
function drawLabels() {
    let alpha = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    let lbls = `<div class="label-cell"></div>`;
    for(let i=0;i<gridSize;i++) lbls += `<div class="label-cell">${alpha[i]}</div>`;
    document.getElementById('targetColLabels').innerHTML = lbls;
    document.getElementById('oceanColLabels').innerHTML = lbls;
    let nums = '';
    for(let i=0;i<gridSize;i++) nums += `<div class="number-cell">${i+1}</div>`;
    document.getElementById('targetRowLabels').innerHTML = nums;
    document.getElementById('oceanRowLabels').innerHTML = nums;
}
function updateShipsRemaining(){
    let remP = playerShipPositions.filter(ship=>!ship.sunk).map(ship=>ship.length);
    let remC = computerShipPositions.filter(ship=>!ship.sunk).map(ship=>ship.length);
    document.getElementById('playerShipsRemaining').innerHTML =
      `<span>Your ships left:</span> ${remP.length>0?remP.join(', '):'None'}`;
    document.getElementById('enemyShipsRemaining').innerHTML =
      `<span>Enemy ships left:</span> ${remC.length>0?remC.join(', '):'None'}`;
}

/* ========== DRAW GAME BOARDS ========== */
function shipSVG(sunk){
    return `<svg width="36" height="36" viewBox="0 0 36 36">
        <defs>
            <linearGradient id="shipGrad" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" style="stop-color:#5a7cb8;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#273c75;stop-opacity:1" />
            </linearGradient>
        </defs>
        <ellipse cx="18" cy="18" rx="16" ry="10" fill="${sunk?'#c2c2c2':'url(#shipGrad)'}" stroke="#192a56" stroke-width="2"/>
        <ellipse cx="18" cy="16" rx="13" ry="7" fill="#6c5ce7" opacity="0.8"/>
        <circle cx="12" cy="16" r="3" fill="#dff9fb" stroke="#192a56" stroke-width="1.5"/>
        <circle cx="24" cy="16" r="3" fill="#dff9fb" stroke="#192a56" stroke-width="1.5"/>
        <rect x="16" y="8" width="4" height="8" fill="#ff6b6b" stroke="#c0392b" stroke-width="1"/>
        <polygon points="18,6 20,8 16,8" fill="#ff6b6b"/>
    </svg>`;
}
function drawBoards(lastMoveTarget=null,lastMoveOcean=null) {
    // Target Board
    let alpha="ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    let targetHtml = '';
    for (let x = 0; x < gridSize; x++) {
        targetHtml += `<div class="row">`;
        for (let y = 0; y < gridSize; y++) {
            let cellClass = "cell targetable";
            let content = '';
            if (targetBoard[x][y] === 2) { cellClass+=" hit"; content='✖'; }
            else if (targetBoard[x][y] === -1) { cellClass+=" miss"; content='●'; }
            if(lastMoveTarget && lastMoveTarget[0]===x&&lastMoveTarget[1]===y) cellClass+=" last-move";
            targetHtml += `<div class="${cellClass}" tabindex="0" aria-label="Cell ${alpha[y]}${x+1}" onclick="playerFire(${x},${y})">${content}</div>`;
        }
        targetHtml += "</div>";
    }
    document.getElementById('targetBoard').innerHTML = targetHtml;
    // Ocean Board
    let oceanHtml = '';
    for (let x = 0; x < gridSize; x++) {
        oceanHtml += `<div class="row">`;
        for (let y = 0; y < gridSize; y++) {
            let cellClass = "cell computer";
            let display = '';
            let shipId = playerShips[x][y];
            let ship = shipId?playerShipPositions[shipId-1]:null;
            if (shipId) {
                cellClass += " ship";
                if(ship&&ship.sunk) cellClass+=" sunk";
                display += shipSVG(ship&&ship.sunk);
            }
            if (oceanBoard[x][y] === 2) { cellClass += " player-hit"; display = '✖'; }
            if (oceanBoard[x][y] === -1) { cellClass += " miss"; display = '●'; }
            if(lastMoveOcean && lastMoveOcean[0]===x&&lastMoveOcean[1]===y) cellClass+=" last-move";
            oceanHtml += `<div class="${cellClass}" tabindex="-1">${display}</div>`;
        }
        oceanHtml += "</div>";
    }
    document.getElementById('oceanBoard').innerHTML = oceanHtml;
    updateShipsRemaining();
}

/* ========== GET SHIP AT COORD ========== */
function getShipAt(posList, x, y) {
    for (let s of posList) {
        for (let c of s.cells) if (c[0]===x && c[1]===y) return s;
    }
    return null;
}

/* ========== PLAYER FIRE ========== */
function playerFire(x, y) {
    if (gameOver || !playerTurnActive) return;
    if (targetBoard[x][y] !== 0) return;
    if (gameMode === "basic") {
        singleShotPlayerFire(x, y);
    } else {
        salvoModePlayerFire(x, y);
    }
}
function singleShotPlayerFire(x, y) {
    let lastMove=null;
    if (computerShips[x][y] !== 0) {
        targetBoard[x][y] = 2;
        playerHits++;
        let ship = getShipAt(computerShipPositions, x, y);
        let sunkText = "";
        if (ship && !ship.sunk) {
            ship.sunk = ship.cells.every(c => targetBoard[c[0]][c[1]] === 2);
            if (ship.sunk) { sunkText = " (You sunk a ship!)"; beep('place'); confetti(); }
            else beep('hit');
        } else beep('hit');
        document.getElementById('status').textContent = "Hit!" + sunkText;
        if (playerHits === computerShipPositions.map(s=>s.length).reduce((a,b)=>a+b,0)) {
            document.getElementById('status').textContent = "You win! All enemy ships sunk!";
            stats.played++; stats.won++; saveStats();
            confetti();
            gameOver = true;
            drawBoards([x,y]);
            updateStats();
            return;
        }
    } else {
        targetBoard[x][y] = -1;
        document.getElementById('status').textContent = "Miss!";
        beep('miss');
    }
    drawBoards([x,y]);
    playerTurnActive = false;
    setTimeout(computerFire, 1100);
}

/* ========== SALVO MODE ========== */
function salvoModePlayerFire(x, y) {
    if (salvoShotsFiredThisTurn >= salvoShots) return;
    if (availableShotsThisTurn.some(coord => coord[0] === x && coord[1] === y)) return;
    availableShotsThisTurn.push([x, y]);
    let lastMove=null;
    if (computerShips[x][y] !== 0) {
        targetBoard[x][y] = 2;
        playerHits++;
        let ship = getShipAt(computerShipPositions, x, y);
        let sunkText = "";
        if (ship && !ship.sunk) {
            ship.sunk = ship.cells.every(c => targetBoard[c[0]][c[1]] === 2);
            if (ship.sunk) { sunkText = " (You sunk a ship!)"; beep('place'); confetti(); }
            else beep('hit');
        } else beep('hit');
        document.getElementById('status').textContent = `Hit! (${availableShotsThisTurn.length}/${salvoShots} shots fired)${sunkText}`;
    } else {
        targetBoard[x][y] = -1;
        document.getElementById('status').textContent = `Miss! (${availableShotsThisTurn.length}/${salvoShots} shots fired)`;
        beep('miss');
    }
    salvoShotsFiredThisTurn++;
    drawBoards([x,y]);
    if (playerHits === computerShipPositions.map(s=>s.length).reduce((a,b)=>a+b,0)) {
        document.getElementById('status').textContent = "You win! All enemy ships sunk!";
        stats.played++; stats.won++; saveStats(); confetti();
        gameOver = true;
        updateStats();
        return;
    }
    if (salvoShotsFiredThisTurn === salvoShots) {
        playerTurnActive = false;
        setTimeout(()=>{
            let shipsSunk = computerShipPositions.filter(ship => ship.cells.every(c=>targetBoard[c[0]][c[1]]===2)).length;
            salvoShots = ships - shipsSunk;
            if (salvoShots < 1) salvoShots = 1;
            document.getElementById('salvoInfo').textContent = `Salvo Mode: You have ${salvoShots} shots this turn.`;
            availableShotsThisTurn = [];
            salvoShotsFiredThisTurn = 0;
            computerFireSalvo();
        }, 900);
    }
}

/* ========== SMARTER COMPUTER AI ("HUNT AND TARGET") ========== */
let aiState = {};
function resetAIState(){
    aiState = {hunt:true,targets:[],dir:null,origin:null,missed:[],tried:[]};
}
function computerFire(){
    if(gameOver) return;
    resetAIState();
    let best = gameDifficulty==='easy'?computerRandomShot():computerSmartShot();
    let x=best[0],y=best[1];
    let hit=false, sunkText="";
    if (playerShips[x][y] !== 0) {
        oceanBoard[x][y] = 2;
        computerHits++;
        let ship = getShipAt(playerShipPositions, x, y);
        if (ship && !ship.sunk) {
            ship.sunk = ship.cells.every(c => oceanBoard[c[0]][c[1]] === 2);
            if (ship.sunk) { sunkText = " (Your ship was sunk!)"; beep('place'); confetti(); }
            else beep('hit');
        } else beep('hit');
        document.getElementById('status').textContent = `Computer hits your ship at ${String.fromCharCode(65+y)}${x+1}!${sunkText}`;
        if (computerHits === playerShipPositions.map(s=>s.length).reduce((a,b)=>a+b,0)) {
            document.getElementById('status').textContent = "Computer wins! All your ships sunk.";
            stats.played++; stats.lost++; saveStats(); confetti();
            gameOver = true; drawBoards(null,[x,y]);
            updateStats();
            return;
        }
        drawBoards(null,[x,y]);
        beep('hit');
    } else {
        oceanBoard[x][y] = -1;
        document.getElementById('status').textContent = `Computer misses at ${String.fromCharCode(65+y)}${x+1}! Your turn.`;
        drawBoards(null,[x,y]);
        beep('miss');
    }
    playerTurnActive = true;
}

function computerRandomShot(){
    let available=[];
    for(let x=0;x<gridSize;x++)
        for(let y=0;y<gridSize;y++)
            if(oceanBoard[x][y]===0) available.push([x,y]);
    return available[Math.floor(Math.random()*available.length)];
}

/* Hunt and target: after hit, tries adjacent cells; prefers finishing off partial hits. */
function computerSmartShot() {
    // Check for any unsunk ship cells with only one hit ("target mode")
    let hits = [];
    for(let x=0;x<gridSize;x++)
        for(let y=0;y<gridSize;y++)
            if(oceanBoard[x][y]===2) hits.push([x,y]);
    // Try finishing off hits with untested neighbors
    for(let i=0;i<hits.length;i++){
        let [x,y]=hits[i];
        let neighbors=[[x-1,y],[x+1,y],[x,y-1],[x,y+1]];
        for(let [nx,ny] of neighbors){
            if(nx>=0&&ny>=0&&nx<gridSize&&ny<gridSize&&oceanBoard[nx][ny]===0)
                return [nx,ny];
        }
    }
    // Otherwise random but avoid repeats
    let available=[];
    for(let x=0;x<gridSize;x++)
        for(let y=0;y<gridSize;y++)
            if(oceanBoard[x][y]===0) available.push([x,y]);
    return available[Math.floor(Math.random()*available.length)];
}

/* ========== COMPUTER FIRE SALVO MODE ========== */
function computerFireSalvo() {
    if (gameOver) return;
    let shots = salvoComputerShots;
    let shipsSunk = playerShipPositions.filter(ship => ship.cells.every(c=>oceanBoard[c[0]][c[1]]===2)).length;
    salvoComputerShots = ships - shipsSunk;
    if (salvoComputerShots < 1) salvoComputerShots = 1;

    let fired = 0, coords = [];
    // Target mode: prioritize adjacent to known hits, else random
    while (fired < salvoComputerShots) {
        let shot = gameDifficulty==='easy'?computerRandomShot():computerSmartShot();
        // don't shoot same cell twice
        if (coords.some(([x,y])=>x===shot[0]&&y===shot[1])) continue;
        coords.push(shot); fired++;
    }
    let hitCount = 0, sunkThisTurn = 0;
    for (let i = 0; i < coords.length; i++) {
        let x = coords[i][0], y = coords[i][1];
        if (playerShips[x][y] !== 0) {
            oceanBoard[x][y] = 2; computerHits++;
            let ship = getShipAt(playerShipPositions, x, y);
            if (ship && !ship.sunk) {
                ship.sunk = ship.cells.every(c => oceanBoard[c[0]][c[1]] === 2);
                if (ship.sunk) sunkThisTurn++;
            }
            hitCount++;
        } else {
            oceanBoard[x][y] = -1;
        }
    }
    document.getElementById('status').textContent = `Computer fires ${shots} shots and ${hitCount} hit(s)!${sunkThisTurn > 0 ? ' ('+sunkThisTurn+' ship(s) sunk!)' : ''}`;
    if (computerHits === playerShipPositions.map(s=>s.length).reduce((a,b)=>a+b,0)) {
        document.getElementById('status').textContent = "Computer wins! All your ships sunk.";
        stats.played++; stats.lost++; saveStats(); confetti();
        gameOver = true; drawBoards();
        updateStats();
        return;
    }
    shipsSunk = playerShipPositions.filter(ship => ship.cells.every(c=>oceanBoard[c[0]][c[1]]===2)).length;
    salvoComputerShots = ships - shipsSunk;
    if (salvoComputerShots < 1) salvoComputerShots = 1;
    drawBoards();
    playerTurnActive = true;
    document.getElementById('salvoInfo').textContent = `Salvo Mode: You have ${salvoShots} shots this turn.`;
}

/* ========== RESTART & INIT ========== */
function restartGame() {
    showPlacementUI();
    restartPlacement();
}
showMainMenu();
updateStats();
</script>
</body>
</html>
