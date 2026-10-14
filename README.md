TEST12345678
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3x3 熄燈遊戲 - 步步成就版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .grid-cell {
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            aspect-ratio: 1 / 1;
        }
        .cell-on {
            background-color: #fbbf24; /* 黃色 */
            box-shadow: 0 0 20px rgba(251, 191, 36, 0.6), inset 0 0 10px rgba(255, 255, 255, 0.5);
        }
        .cell-off {
            background-color: #374151; /* 深灰色 */
            box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3);
        }
        .win-animation {
            animation: celebrate 0.5s ease-out infinite alternate;
        }
        @keyframes celebrate {
            from { transform: scale(1); }
            to { transform: scale(1.05); }
        }
        /* 成就彈窗動畫：每次點擊都快速彈起並淡入 */
        @keyframes popIn {
            0% { transform: translate(-50%, -20px) scale(0.9); opacity: 0; }
            50% { transform: translate(-50%, 5px) scale(1.05); opacity: 1; }
            100% { transform: translate(-50%, 0) scale(1); opacity: 1; }
        }
        .toast-pop {
            animation: popIn 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
    </style>
</head>
<body class="bg-gray-900 text-white min-h-screen flex flex-col items-center justify-start p-4 overflow-y-auto">

    <!-- 成就即時彈出通知區（定位在頂部中央） -->
    <div id="achievement-toast" class="fixed top-4 left-1/2 transform -translate-x-1/2 z-50 pointer-events-none hidden max-w-sm w-[90%]">
        <div id="toast-bg" class="bg-gradient-to-r from-amber-500 via-yellow-500 to-amber-600 text-slate-900 font-bold px-5 py-3 rounded-2xl shadow-2xl flex items-center space-x-3 border border-yellow-200 transition-all duration-300">
            <span id="toast-icon" class="text-3xl">🏆</span>
            <div class="flex-1 min-w-0">
                <div id="toast-category" class="text-[10px] uppercase tracking-wider text-amber-950 font-extrabold opacity-75">解鎖步數成就！</div>
                <div id="toast-title" class="text-base font-black truncate">成就名稱</div>
                <div id="toast-desc" class="text-[11px] font-medium text-amber-950/90 truncate">成就描述</div>
            </div>
        </div>
    </div>

    <div class="max-w-md w-full text-center space-y-5 my-auto pt-16 pb-12">
        <div>
            <h1 class="text-4xl font-extrabold mb-1 tracking-tight bg-gradient-to-r from-yellow-400 via-amber-400 to-orange-500 bg-clip-text text-transparent">
                熄燈挑戰賽
            </h1>
            <p class="text-gray-400 text-xs">每點一步，都有專屬的爆笑步數成就等你解鎖！</p>
        </div>

        <!-- 狀態資訊 -->
        <div class="flex justify-around text-lg font-medium bg-gray-800 p-4 rounded-xl shadow-lg border border-gray-700">
            <div>步數: <span id="move-count" class="text-yellow-400 font-mono font-extrabold">0</span></div>
            <div>勝場: <span id="win-count" class="text-green-400 font-mono font-bold">0</span></div>
            <div id="status-text" class="text-green-400 font-bold hidden animate-bounce">🎉 完美通關！</div>
        </div>

        <!-- 遊戲主體 -->
        <div id="game-board" class="grid grid-cols-3 gap-3 p-4 bg-gray-800 rounded-2xl shadow-2xl border border-gray-700">
            <!-- 由 JS 生成 9 個方塊 -->
        </div>

        <!-- 控制按鈕 -->
        <div class="flex gap-4">
            <button onclick="initGame(true)" class="flex-1 bg-gray-700 hover:bg-gray-600 py-3 rounded-lg font-bold transition-all active:scale-95 shadow-md text-sm">
                重設目前關卡
            </button>
            <button onclick="randomizeBoard()" class="flex-1 bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-500 hover:to-indigo-500 py-3 rounded-lg font-bold transition-all active:scale-95 shadow-lg text-sm">
                隨機新題目
            </button>
        </div>
        
        <!-- 成就面板 -->
        <div class="bg-gray-800/80 p-4 rounded-2xl border border-gray-700 text-left space-y-3">
            <div class="flex flex-col space-y-1">
                <div class="flex items-center justify-between text-sm font-bold text-gray-300">
                    <span>🏅 特殊成就收藏庫</span>
                    <span id="achievement-ratio" class="text-xs text-yellow-400 font-mono">0 / 16 已解鎖</span>
                </div>
                <!-- 成就進度條 -->
                <div class="w-full bg-gray-950 rounded-full h-2 overflow-hidden">
                    <div id="achievement-progress" class="bg-gradient-to-r from-yellow-500 to-amber-400 h-full transition-all duration-500" style="width: 0%"></div>
                </div>
            </div>

            <!-- 滾動的成就列表，防止畫面過長 -->
            <div id="achievement-list" class="grid grid-cols-2 gap-2 text-[11px] max-h-48 overflow-y-auto pr-1">
                <!-- 由 JS 動態生成 16 個成就標章 -->
            </div>
        </div>

        <div class="text-[10px] text-gray-500">
            規則：點擊一個方塊會同時切換它本身與上下左右方塊的狀態。
        </div>
    </div>

    <script>
        const boardElement = document.getElementById('game-board');
        const moveDisplay = document.getElementById('move-count');
        const winDisplay = document.getElementById('win-count');
        const statusText = document.getElementById('status-text');
        
        // 彈出通知相關元素
        const toastElement = document.getElementById('achievement-toast');
        const toastBg = document.getElementById('toast-bg');
        const toastIcon = document.getElementById('toast-icon');
        const toastCategory = document.getElementById('toast-category');
        const toastTitle = document.getElementById('toast-title');
        const toastDesc = document.getElementById('toast-desc');
        
        const achievementListContainer = document.getElementById('achievement-list');
        const achievementRatio = document.getElementById('achievement-ratio');
        const achievementProgress = document.getElementById('achievement-progress');
        
        let moves = 0;
        let totalWins = 0;
        let grid = []; // 3x3 二維陣列
        let initialGrid = []; // 用於重置目前關卡
        let hasWon = false;
        let toastTimeout = null;

        // 追蹤本局行為（用於特殊成就判定）
        let clickedCenterCount = 0;
        let clickedCornerCount = 0;
        let clickedEdgeCount = 0;
        let clickedCellsSet = new Set(); 
        let lastClickedCell = null; 
        let hasDoubleClicked = false; 
        let resetCount = 0; 
        let roundStartTime = null; 

        // 前 20 步精心設計的專屬成語/趣味稱號
        const stepMilestones = {
            1: { title: "一元復始 🚩", desc: "第一步！傳奇冒險的起點！", icon: "🌱" },
            2: { title: "雙喜臨門 👯", desc: "第二步！感覺有好事要發生了！", icon: "✨" },
            3: { title: "三陽開泰 ☀️", desc: "第三步！好運連連，燈光退散！", icon: "🔆" },
            4: { title: "四平八穩 🧱", desc: "第四步！沉穩的步伐，大師風範！", icon: "🧱" },
            5: { title: "五福臨門 🖐️", desc: "第五步！希望你有在認真思考！", icon: "🍀" },
            6: { title: "六六大順 🎲", desc: "第六步！順順利利，通關就在眼前！", icon: "🎲" },
            7: { title: "北斗七星 🌌", desc: "第七步！指引神祕的黑暗力量！", icon: "🌌" },
            8: { title: "才高八斗 🎓", desc: "第八步！智商正在高速運轉！", icon: "🧠" },
            9: { title: "九九歸一 🌀", desc: "第九步！所有的點擊終將化為黑暗！", icon: "🌀" },
            10: { title: "十全十美 💎", desc: "第十步！完美得像個專業解謎家！", icon: "💎" },
            11: { title: "一心一意 ❤️", desc: "第十一步！專注的人最帥氣！", icon: "❤️" },
            12: { title: "雙十二特賣 🛒", desc: "第十二步！不花一毛錢解鎖本步！", icon: "🛒" },
            13: { title: "十三太保 ⚔️", desc: "第十三步！無畏無懼地突破難關！", icon: "⚔️" },
            14: { title: "一生一世 🌸", desc: "第十四步！這輩子一定要把燈熄滅！", icon: "🌸" },
            15: { title: "月圓之夜 🌕", desc: "第十五步！月亮圓了，燈也該滅了！", icon: "🌕" },
            16: { title: "黃金十六 🌟", desc: "第十六步！散發著高手的神祕氣場！", icon: "🌟" },
            17: { title: "石破天驚 ⚡", desc: "第十七步！這一擊，驚天動地！", icon: "⚡" },
            18: { title: "青春十八 🎒", desc: "第十八步！充滿了無限可能！", icon: "🎒" },
            19: { title: "十九行詩 📜", desc: "第十九步！充滿詩情畫意的一點！", icon: "📜" },
            20: { title: "雙十國慶 🎈", desc: "第二十步！普天同慶，掌聲鼓勵！", icon: "🎈" }
        };

        // 超過 20 步後，無限動態生成的稱號
        function generateDynamicStepAchievement(step) {
            if (stepMilestones[step]) return stepMilestones[step];
            
            const prefixes = ["超能", "幻影", "極限", "史詩", "傳奇", "至尊", "宇宙", "皇家", "霸氣", "奇蹟", "終極", "神聖", "炫酷", "無敵", "精準", "量子", "光速"];
            const mid = ["點擊", "指法", "震盪", "射線", "旋風", "風暴", "火花", "魔力", "節奏", "光譜", "逆轉", "守護", "奇想", "探險", "脈衝", "音速"];
            const emojis = ["🔥", "⚡", "✨", "💫", "🎯", "🎮", "🌟", "👾", "🌀", "🔮", "🚀", "🛸", "💎", "🧩", "👑"];
            
            // 透過數學運算產生確定但隨步數變化的組合
            const pIdx = (step * 3) % prefixes.length;
            const mIdx = (step * 7) % mid.length;
            const eIdx = (step * 13) % emojis.length;
            
            return {
                title: `${prefixes[pIdx]}${mid[mIdx]} ${emojis[eIdx]}`,
                desc: `第 ${step} 步：依然堅持不懈，離勝利更近了！`,
                icon: "👟"
            };
        }

        // 定義 16 個特殊大成就清單（藏在下方，達成本局條件才解鎖）
        const achievements = [
            { id: 'first_move', title: '冒險的起點 🚀', desc: '踏出第一步！進行了第 1 次點擊', unlocked: false, check: (s) => s.moves === 1 },
            { id: 'five_moves', title: '漸入佳境 👍', desc: '在單局中累計點擊達到 5 次', unlocked: false, check: (s) => s.moves === 5 },
            { id: 'ten_moves', title: '深思熟慮 🧠', desc: '在單局中累計點擊達到 10 次', unlocked: false, check: (s) => s.moves === 10 },
            { id: 'thirty_moves', title: '馬拉松大師 🏃', desc: '不屈不撓！單局移動達到 30 步', unlocked: false, check: (s) => s.moves === 30 },
            
            { id: 'click_center', title: '直擊紅心 🎯', desc: '點擊了最正中央的黃金位置', unlocked: false, check: (s) => s.clickedCenterCount >= 1 },
            { id: 'click_corner', title: '角落生物 🐱', desc: '點擊了角落的方塊', unlocked: false, check: (s) => s.clickedCornerCount >= 1 },
            { id: 'click_edge', title: '邊緣守衛 🧱', desc: '點擊了十字側邊的邊緣方塊', unlocked: false, check: (s) => s.clickedEdgeCount >= 1 },
            { id: 'all_corners', title: '四角俱全 🔲', desc: '在同一局內把 4 個角落都點過一遍', unlocked: false, check: (s) => s.clickedFourCorners },

            { id: 'hesitate', title: '猶豫不決 ❓', desc: '連續兩次點擊同一個方塊（狀態回復）', unlocked: false, check: (s) => s.hasDoubleClicked },
            { id: 'carpet_search', title: '地毯式搜索 🗺️', desc: '在同一局內點過盤面上的所有 9 個方塊', unlocked: false, check: (s) => s.clickedAllCells },
            { id: 'time_travel', title: '時光倒流 ⏳', desc: '使用「重設目前關卡」功能來重新挑戰', unlocked: false, check: (s) => s.resetCount >= 1 },
            
            { id: 'perfect_win', title: '黃金步數 ⭐', desc: '在 6 步（含）之內完美解鎖通關！', unlocked: false, check: (s) => s.hasWon && s.moves <= 6 },
            { id: 'speed_runner', title: '光速解謎 ⚡', desc: '動腦超快！在第一步點擊後的 12 秒內通關', unlocked: false, check: (s) => s.hasWon && s.elapsedTime > 0 && s.elapsedTime <= 12000 },
            
            { id: 'win_game', title: '光芒終結者 🏆', desc: '成功熄滅所有燈光，通關 1 次！', unlocked: false, check: (s) => s.hasWon },
            { id: 'double_win', title: '梅開二度 ✌️', desc: '累計通關 2 次遊戲', unlocked: false, check: (s) => s.totalWins >= 2 },
            { id: 'legend_master', title: '熄燈大宗師 👑', desc: '實力非凡！累計通關達到 5 次', unlocked: false, check: (s) => s.totalWins >= 5 }
        ];

        // 初始化/重設目前關卡
        function initGame(isResetBtn = false) {
            moves = 0;
            hasWon = false;
            clickedCenterCount = 0;
            clickedCornerCount = 0;
            clickedEdgeCount = 0;
            clickedCellsSet.clear();
            lastClickedCell = null;
            hasDoubleClicked = false;
            roundStartTime = null;

            if (isResetBtn) {
                resetCount++;
            } else {
                resetCount = 0;
            }
            
            moveDisplay.innerText = moves;
            statusText.classList.add('hidden');
            
            if (initialGrid.length > 0) {
                grid = JSON.parse(JSON.stringify(initialGrid));
            } else {
                grid = [
                    [false, false, false],
                    [false, true, false],
                    [false, false, false]
                ];
            }
            
            renderBoard();
            renderAchievements();
            checkSpecialAchievements(); // 檢查重置狀況
        }

        // 隨機生成可解的題目
        function randomizeBoard() {
            moves = 0;
            hasWon = false;
            clickedCenterCount = 0;
            clickedCornerCount = 0;
            clickedEdgeCount = 0;
            clickedCellsSet.clear();
            lastClickedCell = null;
            hasDoubleClicked = false;
            resetCount = 0;
            roundStartTime = null;
            
            moveDisplay.innerText = moves;
            statusText.classList.add('hidden');
            
            grid = [
                [false, false, false],
                [false, false, false],
                [false, false, false]
            ];

            const clickCount = Math.floor(Math.random() * 6) + 5;
            for (let i = 0; i < clickCount; i++) {
                const r = Math.floor(Math.random() * 3);
                const c = Math.floor(Math.random() * 3);
                toggleLogic(r, c); 
            }
            
            if (checkWin()) {
                toggleLogic(1, 1);
            }
            
            initialGrid = JSON.parse(JSON.stringify(grid));
            
            renderBoard();
            renderAchievements();
        }

        // 純邏輯上的切換
        function toggleLogic(r, c) {
            const targets = [[r, c], [r-1, c], [r+1, c], [r, c-1], [r, c+1]];
            targets.forEach(([row, col]) => {
                if (row >= 0 && row < 3 && col >= 0 && col < 3) {
                    grid[row][col] = !grid[row][col];
                }
            });
        }

        // 玩家進行點擊
        function playerToggle(r, c) {
            if (hasWon) return;

            if (moves === 0) {
                roundStartTime = Date.now();
            }

            const cellKey = `${r}-${c}`;

            // 1. 位置統計
            if (r === 1 && c === 1) {
                clickedCenterCount++;
            } else if ((r === 0 || r === 2) && (c === 0 || c === 2)) {
                clickedCornerCount++;
            } else {
                clickedEdgeCount++;
            }

            clickedCellsSet.add(cellKey);

            if (lastClickedCell === cellKey) {
                hasDoubleClicked = true;
            }
            lastClickedCell = cellKey;

            // 翻轉燈光
            toggleLogic(r, c);
            moves++;
            moveDisplay.innerText = moves;

            if (checkWin()) {
                hasWon = true;
                totalWins++;
                winDisplay.innerText = totalWins;
                handleWin();
            }

            renderBoard();

            // 核心功能：每一步必觸發「步數成就」
            const stepAch = generateDynamicStepAchievement(moves);
            
            // 檢查是否有同步達成「特殊大成就」
            const newlyUnlockedSpecial = checkSpecialAchievements();

            // 顯示成就 Toast 彈窗
            showAchievementToast(stepAch, newlyUnlockedSpecial);
        }

        // 檢查並解鎖特殊成就，回傳剛解鎖的成就物件 (如果有的話)
        function checkSpecialAchievements() {
            const elapsedTime = (hasWon && roundStartTime) ? (Date.now() - roundStartTime) : 0;
            const clickedFourCorners = 
                clickedCellsSet.has('0-0') && 
                clickedCellsSet.has('0-2') && 
                clickedCellsSet.has('2-0') && 
                clickedCellsSet.has('2-2');

            const clickedAllCells = clickedCellsSet.size === 9;

            const state = {
                moves: moves,
                clickedCenterCount: clickedCenterCount,
                clickedCornerCount: clickedCornerCount,
                clickedEdgeCount: clickedEdgeCount,
                clickedFourCorners: clickedFourCorners,
                clickedAllCells: clickedAllCells,
                hasDoubleClicked: hasDoubleClicked,
                resetCount: resetCount,
                hasWon: hasWon,
                elapsedTime: elapsedTime,
                totalWins: totalWins
            };

            let newlyUnlocked = null;

            achievements.forEach(ach => {
                if (!ach.unlocked && ach.check(state)) {
                    ach.unlocked = true;
                    newlyUnlocked = ach; // 記錄剛達成的特殊成就
                }
            });

            renderAchievements();
            return newlyUnlocked;
        }

        // 顯示成就通知：每一步點擊時呼叫
        function showAchievementToast(stepAch, specialAch) {
            // 清除之前的淡出計時器
            if (toastTimeout) clearTimeout(toastTimeout);

            // 如果有達成特殊成就，顯示「金橘色大成就風格」，否則顯示「藍紫色步數風格」
            if (specialAch) {
                toastBg.className = "bg-gradient-to-r from-yellow-500 via-amber-500 to-orange-500 text-slate-900 font-bold px-5 py-3 rounded-2xl shadow-2xl flex items-center space-x-3 border border-yellow-200 transition-all duration-300";
                toastCategory.innerText = "🏆 解鎖特殊大成就！";
                toastTitle.innerText = specialAch.title;
                toastDesc.innerText = specialAch.desc;
                toastIcon.innerText = "👑";
            } else {
                toastBg.className = "bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500 text-white font-bold px-5 py-3 rounded-2xl shadow-2xl flex items-center space-x-3 border border-purple-300 transition-all duration-300";
                toastCategory.innerText = "👟 解鎖步數成就！";
                toastTitle.innerText = stepAch.title;
                toastDesc.innerText = stepAch.desc;
                toastIcon.innerText = stepAch.icon;
            }

            // 秀出彈窗並重啟 popIn 動畫
            toastElement.classList.remove('hidden');
            toastElement.classList.remove('toast-pop');
            void toastElement.offsetWidth; // 強制瀏覽器重繪
            toastElement.classList.add('toast-pop');

            // 1.8秒後如果沒有新點擊，自動淡出
            toastTimeout = setTimeout(() => {
                toastElement.classList.add('hidden');
            }, 1800);
        }

        // 繪製特殊成就收藏庫與進度條
        function renderAchievements() {
            achievementListContainer.innerHTML = '';
            let unlockedCount = 0;

            achievements.forEach(ach => {
                if (ach.unlocked) unlockedCount++;

                const card = document.createElement('div');
                card.className = `p-2 rounded-lg flex flex-col justify-between transition-all border ${
                    ach.unlocked 
                        ? 'bg-gradient-to-br from-amber-500/20 to-yellow-500/10 border-yellow-500/40 text-yellow-100 shadow-sm shadow-yellow-500/10' 
                        : 'bg-gray-900/40 border-gray-800 text-gray-500'
                }`;

                card.innerHTML = `
                    <div class="font-bold flex items-start justify-between gap-1">
                        <span class="truncate text-[10px]">${ach.title.split(' ')[0]} ${ach.unlocked ? ach.title.split(' ').slice(1).join(' ') : '未解鎖'}</span>
                        <span class="text-[10px] flex-shrink-0">${ach.unlocked ? '✨' : '🔒'}</span>
                    </div>
                    <p class="text-[9px] text-gray-400 mt-0.5 leading-tight">${ach.unlocked ? ach.desc : '點擊、探索或通關來解鎖'}</p>
                `;
                achievementListContainer.appendChild(card);
            });

            achievementRatio.innerText = `${unlockedCount} / ${achievements.length} 已解鎖`;
            const percentage = (unlockedCount / achievements.length) * 100;
            achievementProgress.style.width = `${percentage}%`;
        }

        // 檢查是否全熄滅
        function checkWin() {
            return grid.every(row => row.every(cell => cell === false));
        }

        // 勝利特效
        function handleWin() {
            statusText.classList.remove('hidden');
            const cells = document.querySelectorAll('.grid-cell');
            cells.forEach(cell => cell.classList.add('win-animation'));
        }

        // 繪製遊戲盤面
        function renderBoard() {
            boardElement.innerHTML = '';
            for (let r = 0; r < 3; r++) {
                for (let c = 0; c < 3; c++) {
                    const cell = document.createElement('div');
                    cell.className = `grid-cell rounded-lg cursor-pointer ${grid[r][c] ? 'cell-on' : 'cell-off'}`;
                    cell.onclick = () => playerToggle(r, c);
                    boardElement.appendChild(cell);
                }
            }
        }

        // 網頁載入時啟動
        window.onload = () => {
            randomizeBoard();
        };
    </script>
</body>
</html>
