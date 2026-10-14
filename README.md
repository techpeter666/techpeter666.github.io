<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>3D 萌寵小狗空間 (Dog Simulator 3D)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Font for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #0f172a;
        }
        #canvas-container {
            width: 100%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        /* 隱藏滾動條但保持滾動能力 */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        .ui-element {
            z-index: 10;
            pointer-events: auto;
        }
        /* 安全區域邊距適配 (iPhone 瀏海與底部條) */
        .safe-top {
            padding-top: max(0.75rem, env(safe-area-inset-top));
        }
        .safe-bottom {
            padding-bottom: max(0.75rem, env(safe-area-inset-bottom));
        }
        .safe-left {
            padding-left: max(0.75rem, env(safe-area-inset-left));
        }
        .safe-right {
            padding-right: max(0.75rem, env(safe-area-inset-right));
        }
        /* 觸控彈性效果 */
        .active-bounce:active {
            transform: scale(0.94);
        }
    </style>
</head>
<body class="bg-slate-900 text-white w-screen h-screen overflow-hidden">

    <div id="loader" class="fixed inset-0 bg-slate-950 flex flex-col items-center justify-center z-50 transition-opacity duration-500 p-4">
        <div class="text-center max-w-xs sm:max-w-md">
            <i class="fas fa-dog text-5xl sm:text-6xl text-amber-500 animate-bounce mb-3 sm:mb-4"></i>
            <h1 class="text-2xl sm:text-3xl font-extrabold tracking-wider text-amber-400 mb-2">3D 超寫實萌寵樂園</h1>
            <p class="text-slate-400 text-xs sm:text-sm mb-6">正在打造您的 3A 級小狗專屬世界...</p>
            <div class="w-56 sm:w-64 bg-slate-800 rounded-full h-2.5 overflow-hidden mx-auto">
                <div id="loading-bar" class="bg-amber-500 h-full w-0 transition-all duration-300"></div>
            </div>
            <p id="loading-status" class="text-xs text-slate-500 mt-2">載入 3D 著色器與響應式環境...</p>
        </div>
    </div>

    <div id="game-ui" class="absolute inset-0 pointer-events-none flex flex-col justify-between safe-top safe-bottom safe-left safe-right p-2 sm:p-4 md:p-6 z-10 transition-opacity duration-300">
        
        <!-- 頂部資訊列 (支援小螢幕自動折疊與彈性收縮) -->
        <div class="flex justify-between items-start w-full gap-2">
            <!-- 左上角：小狗狀態儀表板 (可折疊設計，避免遮擋中央畫面) -->
            <div id="dog-status-card" class="ui-element bg-slate-900/85 backdrop-blur-md rounded-2xl border border-slate-700/60 shadow-2xl p-2 sm:p-3.5 w-auto max-w-[165px] xs:max-w-[200px] sm:max-w-xs transition-all duration-300">
                <div class="flex items-center justify-between gap-1.5 sm:gap-2 min-w-0">
                    <div class="flex items-center gap-1.5 sm:gap-2.5 cursor-pointer min-w-0 flex-1" onclick="toggleStatusExpand()">
                        <div class="w-8 h-8 sm:w-11 sm:h-11 rounded-full bg-amber-500 border-2 border-amber-400 flex items-center justify-center text-base sm:text-2xl shadow-inner shrink-0">
                            🐶
                        </div>
                        <div class="min-w-0 flex-1">
                            <div class="flex items-center gap-1 min-w-0">
                                <span id="dog-name-display" class="font-bold text-[11px] sm:text-base text-amber-300 truncate block max-w-[65px] xs:max-w-[85px] sm:max-w-[120px]">巴弟 (Buddy)</span>
                                <span id="dog-age-badge" class="bg-amber-500/30 text-amber-300 border border-amber-400/50 text-[9px] sm:text-xs px-1.5 py-0.5 rounded-full font-bold shrink-0">3 歲</span>
                                <button onclick="event.stopPropagation(); renameDog()" class="text-[9px] sm:text-xs text-slate-400 hover:text-white p-0.5 shrink-0" title="修改名字">
                                    <i class="fas fa-pen"></i>
                                </button>
                            </div>
                            <span id="dog-state-tag" class="block text-[9px] sm:text-xs bg-green-500/20 text-green-400 px-1 sm:px-1.5 py-0.5 rounded-full font-medium truncate max-w-[100px] sm:max-w-[120px] mt-0.5">悠閒散步中</span>
                        </div>
                    </div>
                    <!-- 折疊狀態條切換按鈕 -->
                    <button onclick="toggleStatusExpand()" id="status-toggle-btn" class="text-slate-400 hover:text-white p-0.5 text-xs sm:hidden shrink-0" title="展開/收合數值">
                        <i class="fas fa-chevron-down transition-transform duration-300" id="status-chevron"></i>
                    </button>
                </div>
                
                <!-- 狀態進度條區塊 (在手機小螢幕上支援折疊以釋放畫面視野) -->
                <div id="stats-container" class="mt-2.5 sm:mt-3 space-y-1.5 sm:space-y-2 text-[11px] sm:text-xs hidden sm:block">
                    <div>
                        <div class="flex justify-between mb-0.5 text-slate-300">
                            <span><i class="fas fa-heart text-red-500 mr-1"></i>心情</span>
                            <span id="stat-joy-val" class="font-mono">80%</span>
                        </div>
                        <div class="w-full bg-slate-700/80 rounded-full h-1.5 overflow-hidden">
                            <div id="stat-joy-bar" class="bg-gradient-to-r from-red-500 to-pink-500 h-full transition-all duration-300" style="width: 80%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between mb-0.5 text-slate-300">
                            <span><i class="fas fa-meat text-amber-600 mr-1"></i>飽食</span>
                            <span id="stat-hunger-val" class="font-mono">70%</span>
                        </div>
                        <div class="w-full bg-slate-700/80 rounded-full h-1.5 overflow-hidden">
                            <div id="stat-hunger-bar" class="bg-gradient-to-r from-amber-600 to-yellow-500 h-full transition-all duration-300" style="width: 70%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between mb-0.5 text-slate-300">
                            <span><i class="fas fa-tint text-blue-400 mr-1"></i>水分</span>
                            <span id="stat-thirst-val" class="font-mono">60%</span>
                        </div>
                        <div class="w-full bg-slate-700/80 rounded-full h-1.5 overflow-hidden">
                            <div id="stat-thirst-bar" class="bg-gradient-to-r from-blue-500 to-cyan-400 h-full transition-all duration-300" style="width: 60%"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 右上角：視角控制、自訂外觀、沉浸模式按鈕組 -->
            <div class="ui-element flex flex-col items-end gap-1.5 sm:gap-2">
                <!-- 第一排快速按鈕：沉浸模式與外觀自訂 -->
                <div class="flex items-center gap-1.5 sm:gap-2">
                    <!-- 全螢幕/沉浸檢視切換 (隱藏所有 UI 專注觀看狗狗) -->
                    <button onclick="toggleZenMode()" id="zen-btn" class="bg-slate-900/85 backdrop-blur-md hover:bg-slate-800 text-slate-300 hover:text-white rounded-xl p-2 sm:px-3 sm:py-2 border border-slate-700/60 shadow-lg text-xs sm:text-sm font-medium flex items-center gap-1.5 active-bounce transition-all" title="切換沉浸純淨視野">
                        <i class="fas fa-eye"></i>
                        <span class="hidden md:inline">簡潔視野</span>
                    </button>

                    <!-- 自訂小狗外觀按鈕 -->
                    <button onclick="toggleCustomizer()" class="bg-slate-900/85 backdrop-blur-md hover:bg-slate-800 rounded-xl p-2 sm:px-3.5 sm:py-2 border border-slate-700/60 shadow-lg flex items-center gap-1.5 text-xs sm:text-sm font-medium text-amber-400 active-bounce transition-all" title="自訂小狗造型">
                        <i class="fas fa-palette"></i>
                        <span class="hidden sm:inline">造型與環境</span>
                    </button>
                </div>

                <!-- 視角切換器 (手機版為水平圖示膠囊，桌面版為垂直完整列表) -->
                <div class="bg-slate-900/85 backdrop-blur-md rounded-xl p-1 sm:p-1.5 border border-slate-700/60 shadow-lg flex sm:flex-col gap-1">
                    <button onclick="setCameraMode('orbit')" id="cam-orbit-btn" class="p-2 sm:px-3 sm:py-2 rounded-lg bg-amber-500 text-white text-xs sm:text-sm font-medium flex items-center gap-2 active-bounce transition-all" title="自由上帝視角">
                        <i class="fas fa-video w-4 text-center"></i>
                        <span class="hidden md:inline">上帝視角</span>
                    </button>
                    <button onclick="setCameraMode('follow')" id="cam-follow-btn" class="p-2 sm:px-3 sm:py-2 rounded-lg text-slate-300 hover:bg-slate-800 text-xs sm:text-sm font-medium flex items-center gap-2 active-bounce transition-all" title="追隨小狗視角">
                        <i class="fas fa-running w-4 text-center"></i>
                        <span class="hidden md:inline">追隨視角</span>
                    </button>
                    <button onclick="setCameraMode('dog')" id="cam-dog-btn" class="p-2 sm:px-3 sm:py-2 rounded-lg text-slate-300 hover:bg-slate-800 text-xs sm:text-sm font-medium flex items-center gap-2 active-bounce transition-all" title="第一人稱狗眼視角">
                        <i class="fas fa-glasses w-4 text-center"></i>
                        <span class="hidden md:inline">狗狗視角</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- 畫面中央：操作手勢指示 (支援適應行動裝置與桌面) -->
        <div id="gesture-hint" class="self-center text-center opacity-80 mb-6 pointer-events-none transition-opacity duration-700 max-w-[85vw]">
            <p id="gesture-hint-text" class="text-xs sm:text-sm bg-black/60 text-slate-200 px-3 py-1.5 sm:px-4 sm:py-2 rounded-full backdrop-blur-md border border-white/10 shadow-lg">
                <i class="fas fa-hand-pointer mr-1.5 text-amber-400 animate-pulse"></i> 單指旋轉視角 / 雙指捏合縮放
            </p>
        </div>

        <!-- 慶祝成長歲數提升彈窗橫幅 (達 95% 觸發) -->
        <div id="ageup-toast" class="ui-element fixed top-16 sm:top-20 left-1/2 -translate-x-1/2 bg-gradient-to-r from-amber-500 via-rose-500 to-amber-500 text-white font-extrabold px-5 py-3 rounded-2xl shadow-2xl border-2 border-white/50 flex items-center gap-3 z-50 pointer-events-none transform scale-0 opacity-0 transition-all duration-500 max-w-[90vw]">
            <div class="text-3xl sm:text-4xl animate-bounce shrink-0">🎂</div>
            <div>
                <div class="text-xs sm:text-sm font-bold text-amber-200 flex items-center gap-1.5">
                    <span>✨ 細心照護！小狗成長蛻變！</span>
                    <span id="ageup-toast-age" class="bg-white/20 px-2 py-0.5 rounded-full text-[11px] font-black text-white">4 歲</span>
                </div>
                <div class="text-[11px] sm:text-xs text-white/95 mt-0.5" id="ageup-toast-desc">
                    心情、飽食與水分皆達 95% 以上！小狗長大 1 歲，體型茁壯長大！
                </div>
            </div>
        </div>

        <!-- 力量曲線與距離控制浮層 HUD -->
        <div id="throw-power-hud" class="ui-element fixed bottom-24 sm:bottom-28 left-1/2 -translate-x-1/2 bg-slate-900/95 backdrop-blur-xl border border-amber-500/50 rounded-2xl p-3.5 sm:p-5 shadow-2xl z-40 w-[94vw] max-w-sm hidden flex-col gap-3 transition-all">
            <!-- 標題與關閉 -->
            <div class="flex justify-between items-center border-b border-slate-700/60 pb-2">
                <div class="flex items-center gap-2 text-amber-400 font-bold text-xs sm:text-sm">
                    <span id="throw-hud-icon" class="text-base sm:text-lg">⚾</span>
                    <span id="throw-hud-title">投擲小球力度控制</span>
                </div>
                <button onclick="closeThrowAimHUD()" class="text-slate-400 hover:text-white p-1 text-xs">
                    <i class="fas fa-times"></i>
                </button>
            </div>

            <!-- 動態力量拋物曲線視覺化 SVG -->
            <div class="w-full bg-slate-950/70 rounded-xl p-2.5 border border-slate-800 relative flex flex-col items-center">
                <svg id="power-curve-svg" viewBox="0 0 280 80" class="w-full h-20 overflow-visible">
                    <defs>
                        <linearGradient id="curveGrad" x1="0%" y1="0%" x2="100%" y2="0%">
                            <stop offset="0%" stop-color="#38bdf8" />
                            <stop offset="50%" stop-color="#fbbf24" />
                            <stop offset="100%" stop-color="#f43f5e" />
                        </linearGradient>
                    </defs>
                    <!-- 地平基準線 -->
                    <line x1="20" y1="72" x2="260" y2="72" stroke="#334155" stroke-width="1.5" stroke-dasharray="3,3" />
                    <!-- 動態拋物線路徑 -->
                    <path id="svg-trajectory-path" d="M 25 70 Q 140 10 255 70" fill="none" stroke="url(#curveGrad)" stroke-width="3.5" stroke-linecap="round" />
                    <!-- 發射起點標記 -->
                    <circle cx="25" cy="70" r="4" fill="#38bdf8" />
                    <!-- 最高點標記 -->
                    <circle id="svg-apex-point" cx="140" cy="15" r="3.5" fill="#fbbf24" />
                    <!-- 預計落點標記 (脈衝光點) -->
                    <circle id="svg-target-point" cx="255" cy="70" r="5" fill="#f43f5e" />
                    <text x="25" y="62" font-size="9" fill="#94a3b8" text-anchor="middle">起點</text>
                    <text id="svg-target-text" x="255" y="62" font-size="9" fill="#fda4af" text-anchor="middle">7m</text>
                </svg>
                <div class="flex justify-between w-full text-[10px] sm:text-xs text-slate-400 mt-1 px-1">
                    <span>近距輕拋 (4m)</span>
                    <span id="power-level-label" class="text-amber-400 font-bold">標準投擲 (7m)</span>
                    <span>活力遠投 (10m)</span>
                </div>
            </div>

            <!-- 力量調整滑桿 (4m ~ 10m) -->
            <div class="w-full flex items-center gap-3">
                <i class="fas fa-feather-alt text-xs text-slate-400"></i>
                <input type="range" id="throw-power-slider" min="4" max="10" step="0.5" value="7" oninput="onPowerSliderChange(this.value)" class="w-full h-2 bg-slate-700 rounded-lg appearance-none cursor-pointer accent-amber-500">
                <i class="fas fa-bolt text-xs text-amber-400"></i>
            </div>

            <!-- 操作確認按鈕 -->
            <div class="flex gap-2">
                <button onclick="closeThrowAimHUD()" class="w-1/3 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold rounded-xl text-xs transition-colors active-bounce">
                    取消
                </button>
                <button onclick="confirmLaunchToy()" class="w-2/3 py-2 bg-gradient-to-r from-amber-500 to-orange-500 hover:from-amber-400 hover:to-orange-400 text-slate-950 font-extrabold rounded-xl text-xs sm:text-sm shadow-lg shadow-orange-500/20 transition-all active-bounce flex items-center justify-center gap-1.5">
                    <i class="fas fa-paper-plane"></i> 鎖定距離並投擲！
                </button>
            </div>
        </div>

        <!-- 底部控制欄 (滾動滑動條 + 自適應小螢幕) -->
        <div class="flex flex-col gap-1.5 sm:gap-3 w-full">
            <!-- 互動動作快捷按鈕區 (行動端流暢水平滾動，避免遮蔽) -->
            <div class="ui-element flex justify-start sm:justify-center items-center gap-2 sm:gap-3 overflow-x-auto no-scrollbar py-1 px-1 mask-linear">
                <button onclick="triggerAction('feed')" class="flex flex-col items-center gap-1 bg-amber-600/90 hover:bg-amber-500 text-white px-3.5 py-2 sm:px-5 sm:py-3 rounded-xl sm:rounded-2xl shadow-lg border border-amber-400/30 min-w-[70px] sm:min-w-[84px] active-bounce transition-all shrink-0">
                    <i class="fas fa-bone text-lg sm:text-2xl"></i>
                    <span class="text-[11px] sm:text-xs font-bold whitespace-nowrap">餵食骨頭</span>
                </button>
                <button onclick="triggerAction('water')" class="flex flex-col items-center gap-1 bg-blue-600/90 hover:bg-blue-500 text-white px-3.5 py-2 sm:px-5 sm:py-3 rounded-xl sm:rounded-2xl shadow-lg border border-blue-400/30 min-w-[70px] sm:min-w-[84px] active-bounce transition-all shrink-0">
                    <i class="fas fa-faucet-drip text-lg sm:text-2xl"></i>
                    <span class="text-[11px] sm:text-xs font-bold whitespace-nowrap">給予清水</span>
                </button>
                <button onclick="triggerAction('throw_ball')" class="flex flex-col items-center gap-1 bg-emerald-600/90 hover:bg-emerald-500 text-white px-3.5 py-2 sm:px-5 sm:py-3 rounded-xl sm:rounded-2xl shadow-lg border border-emerald-400/30 min-w-[70px] sm:min-w-[84px] active-bounce transition-all shrink-0">
                    <i class="fas fa-baseball-ball text-lg sm:text-2xl"></i>
                    <span class="text-[11px] sm:text-xs font-bold whitespace-nowrap">投擲小球</span>
                </button>
                <button onclick="triggerAction('throw_frisbee')" class="flex flex-col items-center gap-1 bg-purple-600/90 hover:bg-purple-500 text-white px-3.5 py-2 sm:px-5 sm:py-3 rounded-xl sm:rounded-2xl shadow-lg border border-purple-400/30 min-w-[70px] sm:min-w-[84px] active-bounce transition-all shrink-0">
                    <i class="fas fa-compact-disc text-lg sm:text-2xl"></i>
                    <span class="text-[11px] sm:text-xs font-bold whitespace-nowrap">發射飛盤</span>
                </button>
                <button onclick="triggerAction('whistle')" class="flex flex-col items-center gap-1 bg-rose-600/90 hover:bg-rose-500 text-white px-3.5 py-2 sm:px-5 sm:py-3 rounded-xl sm:rounded-2xl shadow-lg border border-rose-400/30 min-w-[70px] sm:min-w-[84px] active-bounce transition-all shrink-0">
                    <i class="fas fa-volume-up text-lg sm:text-2xl"></i>
                    <span class="text-[11px] sm:text-xs font-bold whitespace-nowrap">口哨召回</span>
                </button>
                <button onclick="triggerAction('pet')" class="flex flex-col items-center gap-1 bg-pink-600/90 hover:bg-pink-500 text-white px-3.5 py-2 sm:px-5 sm:py-3 rounded-xl sm:rounded-2xl shadow-lg border border-pink-400/30 min-w-[70px] sm:min-w-[84px] active-bounce transition-all shrink-0">
                    <i class="fas fa-hand-holding-heart text-lg sm:text-2xl"></i>
                    <span class="text-[11px] sm:text-xs font-bold whitespace-nowrap">摸摸頭</span>
                </button>
            </div>

            <!-- 底端提示 (手機螢幕縮減文字高度以避免遮擋) -->
            <div class="hidden sm:block text-center text-[11px] text-slate-400 font-mono">
                Three.js 3D 渲染 &middot; 行動手勢縮放 &middot; 隨時點擊按鈕與小狗同樂！
            </div>
        </div>
    </div>

    <!-- 沉浸模式下的單獨喚醒按鈕 (當 UI 被完全隱藏時顯示在螢幕角落) -->
    <button onclick="toggleZenMode()" id="zen-restore-btn" class="ui-element fixed top-4 right-4 z-20 bg-slate-900/80 backdrop-blur-md text-amber-400 border border-slate-700/60 rounded-full w-10 h-10 hidden items-center justify-center shadow-2xl active-bounce">
        <i class="fas fa-eye-slash"></i>
    </button>

    <!-- 自訂外觀與環境抽屜面板 (自適應寬度與捲動區域) -->
    <div id="customizer-panel" class="ui-element fixed right-0 top-0 bottom-0 w-full sm:w-80 md:w-96 max-w-full bg-slate-900/95 backdrop-blur-xl border-l border-slate-700/60 shadow-2xl z-30 p-4 sm:p-6 flex flex-col justify-between transform translate-x-full transition-transform duration-300 ease-out safe-top safe-bottom safe-right">
        <div class="overflow-y-auto no-scrollbar space-y-5 pr-1">
            <div class="flex justify-between items-center border-b border-slate-800 pb-3">
                <h3 class="text-lg sm:text-xl font-extrabold text-amber-400 flex items-center gap-2">
                    <i class="fas fa-dog"></i> 小狗外觀與環境
                </h3>
                <button onclick="toggleCustomizer()" class="text-slate-400 hover:text-white p-2 text-base active-bounce">
                    <i class="fas fa-times"></i>
                </button>
            </div>

            <!-- 1. 品種選取 (更換材質配色) -->
            <div class="space-y-2">
                <label class="text-xs sm:text-sm font-semibold text-slate-300 block">1. 選擇毛色品種</label>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="changeDogBreed('shiba')" id="breed-shiba" class="p-2.5 sm:p-3 rounded-xl border-2 border-amber-500 bg-amber-500/10 text-left transition-all active-bounce">
                        <div class="font-bold text-xs sm:text-sm text-amber-400">秋田 / 柴犬</div>
                        <div class="text-[10px] sm:text-xs text-slate-400">經典橘白相間</div>
                    </button>
                    <button onclick="changeDogBreed('golden')" id="breed-golden" class="p-2.5 sm:p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-700 active-bounce">
                        <div class="font-bold text-xs sm:text-sm text-yellow-400">黃金獵犬</div>
                        <div class="text-[10px] sm:text-xs text-slate-400">溫暖陽光金黃</div>
                    </button>
                    <button onclick="changeDogBreed('husky')" id="breed-husky" class="p-2.5 sm:p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-700 active-bounce">
                        <div class="font-bold text-xs sm:text-sm text-slate-300">哈士奇</div>
                        <div class="text-[10px] sm:text-xs text-slate-400">酷炫灰黑面具</div>
                    </button>
                    <button onclick="changeDogBreed('dalmatian')" id="breed-dalmatian" class="p-2.5 sm:p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-700 active-bounce">
                        <div class="font-bold text-xs sm:text-sm text-white">大麥町</div>
                        <div class="text-[10px] sm:text-xs text-slate-400">逗趣黑白斑點</div>
                    </button>
                </div>
            </div>

            <!-- 2. 配飾選取 (戴帽子/皇冠) -->
            <div class="space-y-2">
                <label class="text-xs sm:text-sm font-semibold text-slate-300 block">2. 潮流專屬配飾</label>
                <div class="grid grid-cols-3 gap-2 text-center text-xs">
                    <button onclick="changeAccessory('none')" id="acc-none" class="p-2 sm:p-2.5 rounded-lg border-2 border-amber-500 bg-amber-500/10 font-bold active-bounce">
                        無配飾
                    </button>
                    <button onclick="changeAccessory('party')" id="acc-party" class="p-2 sm:p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-700 active-bounce">
                        🎉 派對帽
                    </button>
                    <button onclick="changeAccessory('crown')" id="acc-crown" class="p-2 sm:p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-700 active-bounce">
                        👑 皇冠
                    </button>
                </div>
            </div>

            <!-- 3. 草原時間與光影更換 -->
            <div class="space-y-2">
                <label class="text-xs sm:text-sm font-semibold text-slate-300 block">3. 改變世界時間</label>
                <div class="grid grid-cols-3 gap-2 text-center text-xs">
                    <button onclick="changeTimeOfDay('day')" id="time-day" class="p-2 sm:p-2.5 rounded-lg border border-amber-500/50 bg-amber-500/10 text-amber-400 font-bold active-bounce">
                        ☀️ 白天
                    </button>
                    <button onclick="changeTimeOfDay('sunset')" id="time-sunset" class="p-2 sm:p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700 active-bounce">
                        🌅 夕陽
                    </button>
                    <button onclick="changeTimeOfDay('night')" id="time-night" class="p-2 sm:p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700 active-bounce">
                        🌌 夜晚
                    </button>
                </div>
            </div>
        </div>

        <div class="border-t border-slate-800 pt-3 mt-3">
            <button onclick="toggleCustomizer()" class="w-full py-2.5 sm:py-3 bg-amber-500 hover:bg-amber-400 text-slate-950 font-extrabold rounded-xl transition-colors shadow-lg shadow-amber-500/20 active-bounce text-sm sm:text-base">
                完成並返回樂園
            </button>
        </div>
    </div>

    <!-- 自訂狗名彈窗 (響應式居中對話盒) -->
    <div id="rename-modal" class="ui-element fixed inset-0 bg-black/60 backdrop-blur-sm hidden items-center justify-center z-50 p-4">
        <div class="bg-slate-900 border border-slate-700 rounded-2xl p-5 sm:p-6 w-full max-w-xs shadow-2xl">
            <h3 class="text-base sm:text-lg font-bold text-amber-400 mb-3"><i class="fas fa-tags mr-2"></i>為您的小狗命名</h3>
            <input type="text" id="dog-name-input" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-3.5 py-2.5 text-sm sm:text-base text-white focus:outline-none focus:border-amber-500 mb-4" placeholder="輸入名字..." maxLength="12">
            <div class="flex gap-2 justify-end">
                <button onclick="closeRenameModal()" class="px-3.5 py-2 text-xs sm:text-sm text-slate-400 hover:text-white">取消</button>
                <button onclick="saveDogName()" class="px-4 py-2 text-xs sm:text-sm bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold rounded-lg transition-colors active-bounce">確認修改</button>
            </div>
        </div>
    </div>

    <!-- 3D 渲染容器 -->
    <div id="canvas-container"></div>

    <script>
        // --- 遊戲全域狀態與配置 ---
        let scene, camera, renderer;
        let clock = new THREE.Clock();
        
        // 核心元素
        let dog; // 小狗主體 Group
        let floor; // 草原
        let sunLight, hemiLight; // 光源
        let skyMesh; // 天空球

        // 互動狀態
        let currentToy = null;
        let activeBones = [];
        let activeWaterBowls = [];
        let particles = [];

        // 投擲距離與力量預覽相關全域變數
        let isAimingToy = false;
        let aimingToyType = 'ball'; // 'ball' 或 'frisbee'
        let currentThrowDistance = 7; // 預設 7 公尺 (範圍 4m - 10m)
        let groundTargetMarker = null; // 3D 草原落點光圈
        let trajectoryArcMesh = null;  // 3D 空中拋物虛線

        // 介面折疊與沉浸模式狀態
        let isStatusExpanded = false;
        let isZenMode = false;
        let isAgeUpProcessing = false;

        // 遊戲狀態與數值
        const gameState = {
            dogName: "巴弟 (Buddy)",
            age: 3, // 從 3 歲開始
            baseScale: 1.0,
            targetScale: 1.0,
            breed: "shiba",
            accessory: "none",
            timeOfDay: "day",
            cameraMode: "orbit",
            dogState: "idle",
            stats: {
                joy: 80,
                hunger: 70,
                thirst: 60
            },
            dogPos: new THREE.Vector3(0, 0, 0),
            dogTargetPos: new THREE.Vector3(0, 0, 0),
            dogRotationY: 0,
            speed: 0,
            animTimer: 0
        };

        // 顏色彩現表
        const colors = {
            shiba: { primary: 0xd47a32, secondary: 0xffffff, nose: 0x111111 },
            golden: { primary: 0xe3a857, secondary: 0xffe599, nose: 0x1a1105 },
            husky: { primary: 0x4a4a4a, secondary: 0xf5f5f5, nose: 0x050505 },
            dalmatian: { primary: 0xeeeeee, secondary: 0x222222, nose: 0x000000 },
            grass: { day: 0x5a9e32, sunset: 0x8a5a22, night: 0x1a3a12 },
            sky: { day: 0x87ceeb, sunset: 0xfd5e53, night: 0x0b1d3a }
        };

        window.onload = function() {
            init3D();
            setupUIProgress();
            setupResponsiveHints();
            animate();
            
            // 監聽視窗旋轉與大小變動
            window.addEventListener('resize', onWindowResize, false);
            window.addEventListener('orientationchange', onWindowResize, false);
            
            // 定時小幅消耗小狗屬性
            setInterval(() => {
                gameState.stats.hunger = Math.max(0, gameState.stats.hunger - 1);
                gameState.stats.thirst = Math.max(0, gameState.stats.thirst - 1.5);
                gameState.stats.joy = Math.max(0, gameState.stats.joy - 0.8);
                updateStatsUI();
            }, 3000);
        };

        function setupResponsiveHints() {
            const isTouch = ('ontouchstart' in window) || (navigator.maxTouchPoints > 0);
            const hintElem = document.getElementById('gesture-hint-text');
            if (isTouch) {
                hintElem.innerHTML = `<i class="fas fa-hand-pointer mr-1.5 text-amber-400 animate-pulse"></i> 單指旋轉視角 / 雙指捏合縮放`;
            } else {
                hintElem.innerHTML = `<i class="fas fa-mouse-pointer mr-1.5 text-amber-400 animate-pulse"></i> 拖曳滑鼠旋轉視角 / 滾輪縮放`;
            }
        }

        function setupUIProgress() {
            const bar = document.getElementById('loading-bar');
            const status = document.getElementById('loading-status');
            const loader = document.getElementById('loader');
            
            let progress = 0;
            const interval = setInterval(() => {
                progress += 10;
                bar.style.width = progress + '%';
                if (progress === 30) status.innerText = '正在生成 3D 精細小狗模型...';
                if (progress === 60) status.innerText = '正在渲染高動態光源與草原環境...';
                if (progress === 90) status.innerText = '適配行動觸控手勢與響應式介面...';
                if (progress >= 100) {
                    clearInterval(interval);
                    status.innerText = '啟動萌寵空間！';
                    setTimeout(() => {
                        loader.classList.add('opacity-0');
                        setTimeout(() => loader.style.display = 'none', 500);
                    }, 400);
                }
            }, 120);
        }

        function init3D() {
            const container = document.getElementById('canvas-container');
            
            scene = new THREE.Scene();
            scene.fog = new THREE.FogExp2(0x87ceeb, 0.015);

            camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 1000);
            resetCamera();

            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            renderer.toneMappingExposure = 1.0;
            container.appendChild(renderer.domElement);

            hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.6);
            hemiLight.position.set(0, 200, 0);
            scene.add(hemiLight);

            sunLight = new THREE.DirectionalLight(0xffffff, 1.2);
            sunLight.position.set(30, 40, 20);
            sunLight.castShadow = true;
            sunLight.shadow.camera.top = 25;
            sunLight.shadow.camera.bottom = -25;
            sunLight.shadow.camera.left = -25;
            sunLight.shadow.camera.right = 25;
            sunLight.shadow.camera.near = 0.1;
            sunLight.shadow.camera.far = 150;
            sunLight.shadow.mapSize.width = 1024;
            sunLight.shadow.mapSize.height = 1024;
            sunLight.shadow.bias = -0.0005;
            scene.add(sunLight);

            const ambientLight = new THREE.AmbientLight(0xffffff, 0.25);
            scene.add(ambientLight);

            createSky();
            createTerrain();
            decorateEnvironment();
            createDogModel();
            createTrajectoryVisualizers();
            setupInteractions();
        }

        // 建立 3D 場景內的瞄準拋物軌跡線與地面落點標記光圈
        function createTrajectoryVisualizers() {
            // 1. 地面落點目標光圈
            const markerGroup = new THREE.Group();
            
            const ringGeo = new THREE.RingGeometry(0.5, 0.7, 32);
            ringGeo.rotateX(-Math.PI / 2);
            const ringMat = new THREE.MeshBasicMaterial({ 
                color: 0x38bdf8, 
                side: THREE.DoubleSide, 
                transparent: true, 
                opacity: 0.85 
            });
            const ringMesh = new THREE.Mesh(ringGeo, ringMat);
            markerGroup.add(ringMesh);

            const innerDotGeo = new THREE.CircleGeometry(0.18, 16);
            innerDotGeo.rotateX(-Math.PI / 2);
            const innerDotMat = new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide, transparent: true, opacity: 0.9 });
            const innerDot = new THREE.Mesh(innerDotGeo, innerDotMat);
            innerDot.position.y = 0.01;
            markerGroup.add(innerDot);

            markerGroup.position.set(0, 0.05, 0);
            markerGroup.visible = false;
            scene.add(markerGroup);
            groundTargetMarker = markerGroup;

            // 2. 空中拋物線軌跡 (動態 Line)
            const arcMat = new THREE.LineDashedMaterial({
                color: 0xfbbf24,
                linewidth: 2,
                dashSize: 0.4,
                gapSize: 0.2,
                transparent: true,
                opacity: 0.8
            });
            const arcGeo = new THREE.BufferGeometry();
            const initialPoints = new Float32Array(30 * 3);
            arcGeo.setAttribute('position', new THREE.BufferAttribute(initialPoints, 3));
            
            trajectoryArcMesh = new THREE.Line(arcGeo, arcMat);
            trajectoryArcMesh.computeLineDistances();
            trajectoryArcMesh.visible = false;
            scene.add(trajectoryArcMesh);
        }

        function createSky() {
            const skyGeo = new THREE.SphereGeometry(150, 32, 15);
            const skyMat = new THREE.MeshBasicMaterial({
                color: colors.sky.day,
                side: THREE.BackSide
            });
            skyMesh = new THREE.Mesh(skyGeo, skyMat);
            scene.add(skyMesh);
        }

        function createTerrain() {
            const size = 150;
            const floorGeo = new THREE.PlaneGeometry(size, size, 36, 36);
            const pos = floorGeo.attributes.position;
            for (let i = 0; i < pos.count; i++) {
                const x = pos.getX(i);
                const y = pos.getY(i);
                const distFromCenter = Math.sqrt(x*x + y*y);
                if (distFromCenter > 15) {
                    const z = Math.sin(x * 0.1) * Math.cos(y * 0.1) * 1.5 + Math.sin(x * 0.03) * 3.0;
                    pos.setZ(i, z);
                }
            }
            floorGeo.computeVertexNormals();

            const floorMat = new THREE.MeshStandardMaterial({
                color: colors.grass.day,
                roughness: 0.9,
                metalness: 0.1,
                flatShading: true
            });

            floor = new THREE.Mesh(floorGeo, floorMat);
            floor.rotation.x = -Math.PI / 2;
            floor.receiveShadow = true;
            scene.add(floor);
        }

        function decorateEnvironment() {
            const createTree = (x, z, scale) => {
                const treeGroup = new THREE.Group();
                treeGroup.position.set(x, getTerrainHeight(x, z), z);
                treeGroup.scale.set(scale, scale, scale);

                const trunkGeo = new THREE.CylinderGeometry(0.2, 0.4, 2, 5);
                const trunkMat = new THREE.MeshStandardMaterial({ color: 0x5c4033, roughness: 0.9 });
                const trunk = new THREE.Mesh(trunkGeo, trunkMat);
                trunk.position.y = 1;
                trunk.castShadow = true;
                trunk.receiveShadow = true;
                treeGroup.add(trunk);

                const leavesGeo = new THREE.DodecahedronGeometry(1.2, 1);
                const leavesMat = new THREE.MeshStandardMaterial({ color: 0x2e8b57, roughness: 0.6, flatShading: true });
                const leaves = new THREE.Mesh(leavesGeo, leavesMat);
                leaves.position.y = 2.2;
                leaves.castShadow = true;
                treeGroup.add(leaves);

                scene.add(treeGroup);
            };

            const createRock = (x, z, scale) => {
                const rockGeo = new THREE.DodecahedronGeometry(0.6, 0);
                const rockMat = new THREE.MeshStandardMaterial({ color: 0x888888, roughness: 0.9, flatShading: true });
                const rock = new THREE.Mesh(rockGeo, rockMat);
                rock.position.set(x, getTerrainHeight(x, z) + 0.1, z);
                rock.scale.set(scale, scale * 0.6, scale);
                rock.rotation.set(Math.random(), Math.random(), Math.random());
                rock.castShadow = true;
                scene.add(rock);
            };

            for (let i = 0; i < 35; i++) {
                let r = 18 + Math.random() * 45;
                let theta = Math.random() * Math.PI * 2;
                createTree(Math.cos(theta) * r, Math.sin(theta) * r, 1.2 + Math.random() * 0.8);
            }

            for (let i = 0; i < 25; i++) {
                let r = 12 + Math.random() * 32;
                let theta = Math.random() * Math.PI * 2;
                createRock(Math.cos(theta) * r, Math.sin(theta) * r, 0.8 + Math.random() * 1.0);
            }
        }

        function getTerrainHeight(x, z) {
            const dist = Math.sqrt(x*x + z*z);
            if (dist <= 15) return 0;
            return Math.sin(x * 0.1) * Math.cos(z * 0.1) * 1.5 + Math.sin(x * 0.03) * 3.0;
        }

        function createDogModel() {
            dog = new THREE.Group();
            dog.position.set(0, 0, 0);
            dog.castShadow = true;

            const breedColor = colors[gameState.breed];
            const primaryMat = new THREE.MeshStandardMaterial({ color: breedColor.primary, roughness: 0.8, flatShading: true });
            const secondaryMat = new THREE.MeshStandardMaterial({ color: breedColor.secondary, roughness: 0.8, flatShading: true });
            const blackMat = new THREE.MeshStandardMaterial({ color: breedColor.nose, roughness: 0.5 });
            const pinkMat = new THREE.MeshStandardMaterial({ color: 0xffb6c1, roughness: 0.6 });

            // 身體
            const bodyGeo = new THREE.BoxGeometry(0.9, 0.9, 1.6);
            const body = new THREE.Mesh(bodyGeo, primaryMat);
            body.position.y = 0.8;
            body.castShadow = true;
            body.receiveShadow = true;
            dog.add(body);
            dog.body = body;

            // 腹部
            const bellyGeo = new THREE.BoxGeometry(0.92, 0.5, 1.0);
            const belly = new THREE.Mesh(bellyGeo, secondaryMat);
            belly.position.set(0, -0.22, 0.1);
            body.add(belly);

            // 頭部
            const headGroup = new THREE.Group();
            headGroup.position.set(0, 0.6, 0.8);
            
            const headGeo = new THREE.BoxGeometry(0.8, 0.8, 0.8);
            const head = new THREE.Mesh(headGeo, primaryMat);
            head.castShadow = true;
            headGroup.add(head);
            headGroup.mainHead = head;

            const faceWhiteGeo = new THREE.BoxGeometry(0.82, 0.4, 0.4);
            const faceWhite = new THREE.Mesh(faceWhiteGeo, secondaryMat);
            faceWhite.position.set(0, -0.2, 0.21);
            head.add(faceWhite);

            const muzzleGeo = new THREE.BoxGeometry(0.4, 0.3, 0.4);
            const muzzle = new THREE.Mesh(muzzleGeo, secondaryMat);
            muzzle.position.set(0, -0.1, 0.5);
            head.add(muzzle);

            const noseGeo = new THREE.BoxGeometry(0.18, 0.12, 0.12);
            const nose = new THREE.Mesh(noseGeo, blackMat);
            nose.position.set(0, 0.1, 0.22);
            muzzle.add(nose);

            const eyeGeo = new THREE.BoxGeometry(0.1, 0.1, 0.1);
            const leftEye = new THREE.Mesh(eyeGeo, blackMat);
            leftEye.position.set(0.28, 0.15, 0.38);
            const rightEye = leftEye.clone();
            rightEye.position.x = -0.28;
            head.add(leftEye);
            head.add(rightEye);

            const browGeo = new THREE.BoxGeometry(0.15, 0.08, 0.05);
            const leftBrow = new THREE.Mesh(browGeo, secondaryMat);
            leftBrow.position.set(0.2, 0.32, 0.39);
            const rightBrow = leftBrow.clone();
            rightBrow.position.x = -0.2;
            head.add(leftBrow);
            head.add(rightBrow);

            const tongueGeo = new THREE.BoxGeometry(0.16, 0.05, 0.25);
            const tongue = new THREE.Mesh(tongueGeo, pinkMat);
            tongue.position.set(0, -0.15, 0.35);
            tongue.rotation.x = 0.1;
            head.add(tongue);
            dog.tongue = tongue;
            tongue.visible = false;

            // 耳朵
            const earGeo = new THREE.ConeGeometry(0.25, 0.5, 4);
            earGeo.rotateX(Math.PI/2);
            
            const leftEar = new THREE.Mesh(earGeo, primaryMat);
            leftEar.position.set(0.32, 0.45, -0.1);
            leftEar.rotation.set(-0.1, 0, -0.2);
            leftEar.castShadow = true;
            
            const innerEarGeo = new THREE.ConeGeometry(0.15, 0.35, 4);
            innerEarGeo.rotateX(Math.PI/2);
            const leftInnerEar = new THREE.Mesh(innerEarGeo, pinkMat);
            leftInnerEar.position.set(0, 0.01, 0.05);
            leftEar.add(leftInnerEar);

            const rightEar = leftEar.clone();
            rightEar.position.x = -0.32;
            rightEar.rotation.z = 0.2;

            head.add(leftEar);
            head.add(rightEar);
            dog.leftEar = leftEar;
            dog.rightEar = rightEar;

            body.add(headGroup);
            dog.head = headGroup;

            const legGeo = new THREE.BoxGeometry(0.24, 0.6, 0.24);
            const pawGeo = new THREE.BoxGeometry(0.28, 0.12, 0.32);

            const createLeg = (x, y, z) => {
                const legGroup = new THREE.Group();
                legGroup.position.set(x, y, z);

                const upperLeg = new THREE.Mesh(legGeo, primaryMat);
                upperLeg.castShadow = true;
                upperLeg.position.y = -0.2;
                legGroup.add(upperLeg);

                const paw = new THREE.Mesh(pawGeo, secondaryMat);
                paw.position.set(0, -0.5, 0.04);
                paw.castShadow = true;
                legGroup.add(paw);

                return legGroup;
            };

            dog.legLF = createLeg(0.32, -0.3, 0.5);
            dog.legRF = createLeg(-0.32, -0.3, 0.5);
            dog.legLB = createLeg(0.32, -0.3, -0.5);
            dog.legRB = createLeg(-0.32, -0.3, -0.5);

            body.add(dog.legLF);
            body.add(dog.legRF);
            body.add(dog.legLB);
            body.add(dog.legRB);

            // 尾巴
            const tailGroup = new THREE.Group();
            tailGroup.position.set(0, 0.3, -0.85);
            
            const tailGeo = new THREE.BoxGeometry(0.18, 0.18, 0.7);
            tailGeo.translate(0, 0, -0.3);
            const tail = new THREE.Mesh(tailGeo, primaryMat);
            tail.castShadow = true;
            tail.rotation.x = 0.6;
            tailGroup.add(tail);

            const tailTipGeo = new THREE.BoxGeometry(0.2, 0.2, 0.2);
            const tailTip = new THREE.Mesh(tailTipGeo, secondaryMat);
            tailTip.position.set(0, 0, -0.65);
            tail.add(tailTip);

            body.add(tailGroup);
            dog.tail = tailGroup;

            // 潮流飾品
            dog.accessoryGroup = new THREE.Group();
            dog.accessoryGroup.position.set(0, 0.5, 0);
            head.add(dog.accessoryGroup);

            scene.add(dog);
            updateDogAccessory();
        }

        function updateDogBreedColor() {
            if (!dog) return;
            const breedColor = colors[gameState.breed];
            
            dog.traverse((child) => {
                if (child.isMesh) {
                    const colVal = child.material.color.getHex();
                    if (colVal === 0xd47a32 || colVal === 0xe3a857 || colVal === 0x4a4a4a || colVal === 0xeeeeee) {
                        child.material.color.setHex(breedColor.primary);
                    } else if (colVal === 0xffffff || colVal === 0xffe599 || colVal === 0xf5f5f5 || colVal === 0x222222) {
                        child.material.color.setHex(breedColor.secondary);
                    } else if (colVal === 0x111111 || colVal === 0x1a1105 || colVal === 0x050505 || colVal === 0x000000) {
                        child.material.color.setHex(breedColor.nose);
                    }
                }
            });
        }

        function updateDogAccessory() {
            if (!dog || !dog.accessoryGroup) return;

            while(dog.accessoryGroup.children.length > 0) { 
                dog.accessoryGroup.remove(dog.accessoryGroup.children[0]); 
            }

            if (gameState.accessory === 'none') return;

            if (gameState.accessory === 'party') {
                const hatGeo = new THREE.ConeGeometry(0.2, 0.5, 8);
                const hatMat = new THREE.MeshStandardMaterial({ color: 0xff00ff, roughness: 0.4 });
                const hat = new THREE.Mesh(hatGeo, hatMat);
                hat.rotation.x = -0.1;
                hat.position.set(0, 0.1, -0.05);
                hat.castShadow = true;

                const pomGeo = new THREE.SphereGeometry(0.06, 8, 8);
                const pomMat = new THREE.MeshStandardMaterial({ color: 0xffff00 });
                const pom = new THREE.Mesh(pomGeo, pomMat);
                pom.position.y = 0.28;
                hat.add(pom);

                dog.accessoryGroup.add(hat);
            } 
            else if (gameState.accessory === 'crown') {
                const crownGroup = new THREE.Group();
                crownGroup.position.set(0, 0.05, 0);

                const ringGeo = new THREE.CylinderGeometry(0.22, 0.22, 0.12, 12, 1, true);
                const goldMat = new THREE.MeshStandardMaterial({ color: 0xffd700, metalness: 0.9, roughness: 0.1 });
                const ring = new THREE.Mesh(ringGeo, goldMat);
                crownGroup.add(ring);

                for (let i = 0; i < 5; i++) {
                    const spikeGeo = new THREE.ConeGeometry(0.06, 0.15, 4);
                    const spike = new THREE.Mesh(spikeGeo, goldMat);
                    const angle = (i / 5) * Math.PI * 2;
                    spike.position.set(Math.cos(angle) * 0.22, 0.1, Math.sin(angle) * 0.22);
                    spike.rotation.y = -angle;
                    spike.rotation.x = 0.2;
                    crownGroup.add(spike);
                }

                dog.accessoryGroup.add(crownGroup);
            }
        }

        function createHeartParticle(pos) {
            const group = new THREE.Group();
            group.position.copy(pos);
            
            const leftGeo = new THREE.SphereGeometry(0.12, 8, 8);
            const redMat = new THREE.MeshBasicMaterial({ color: 0xff3366 });
            
            const left = new THREE.Mesh(leftGeo, redMat);
            left.position.x = -0.08;
            const right = left.clone();
            right.position.x = 0.08;
            
            const bottomGeo = new THREE.ConeGeometry(0.18, 0.3, 8);
            bottomGeo.rotateX(Math.PI);
            const bottom = new THREE.Mesh(bottomGeo, redMat);
            bottom.position.y = -0.12;

            group.add(left);
            group.add(right);
            group.add(bottom);
            group.scale.set(0.1, 0.1, 0.1);

            scene.add(group);
            particles.push({
                mesh: group,
                type: 'heart',
                velocity: new THREE.Vector3((Math.random() - 0.5) * 0.5, 1.2 + Math.random() * 0.5, (Math.random() - 0.5) * 0.5),
                life: 1.0
            });
        }

        function createDustParticle(pos) {
            const geo = new THREE.DodecahedronGeometry(0.1, 0);
            const mat = new THREE.MeshBasicMaterial({ color: 0xcccccc, transparent: true, opacity: 0.6 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.copy(pos);
            mesh.position.y += 0.1;

            scene.add(mesh);
            particles.push({
                mesh: mesh,
                type: 'dust',
                velocity: new THREE.Vector3((Math.random() - 0.5) * 0.8, 0.5 + Math.random() * 0.4, (Math.random() - 0.5) * 0.8),
                life: 0.5
            });
        }

        let isUserInteracting = false;
        let onPointerDownMouseX = 0, onPointerDownMouseY = 0;
        let lon = 0, onPointerDownLon = 0;
        let lat = 0, onPointerDownLat = 0;
        let cameraRadius = 10;
        let initialPinchDistance = null;
        let initialPinchRadius = 10;

        function setupInteractions() {
            const dom = renderer.domElement;

            // 滑鼠與單指指標按下
            dom.addEventListener('pointerdown', (e) => {
                if (e.pointerType === 'touch' && !e.isPrimary) return;
                isUserInteracting = true;
                onPointerDownMouseX = e.clientX;
                onPointerDownMouseY = e.clientY;
                onPointerDownLon = lon;
                onPointerDownLat = lat;
                
                const hint = document.getElementById('gesture-hint');
                if (hint) hint.style.opacity = 0;
            });

            dom.addEventListener('pointermove', (e) => {
                if (isUserInteracting && (e.pointerType !== 'touch' || e.isPrimary)) {
                    lon = (onPointerDownMouseX - e.clientX) * 0.2 + onPointerDownLon;
                    lat = (e.clientY - onPointerDownMouseY) * 0.2 + onPointerDownLat;
                    lat = Math.max(-85, Math.min(85, lat));
                }
            });

            document.addEventListener('pointerup', () => {
                isUserInteracting = false;
            });

            // 滑鼠滾輪縮放
            dom.addEventListener('wheel', (e) => {
                cameraRadius += e.deltaY * 0.005;
                cameraRadius = Math.max(4, Math.min(25, cameraRadius));
            }, { passive: true });

            // 行動裝置專屬：雙指捏合縮放 (Pinch to Zoom)
            dom.addEventListener('touchstart', (e) => {
                if (e.touches.length === 2) {
                    isUserInteracting = false; // 暫停旋轉轉移為縮放
                    const dx = e.touches[0].clientX - e.touches[1].clientX;
                    const dy = e.touches[0].clientY - e.touches[1].clientY;
                    initialPinchDistance = Math.hypot(dx, dy);
                    initialPinchRadius = cameraRadius;
                }
            }, { passive: true });

            dom.addEventListener('touchmove', (e) => {
                if (e.touches.length === 2 && initialPinchDistance) {
                    const dx = e.touches[0].clientX - e.touches[1].clientX;
                    const dy = e.touches[0].clientY - e.touches[1].clientY;
                    const currentDist = Math.hypot(dx, dy);
                    const factor = initialPinchDistance / currentDist;
                    cameraRadius = Math.max(4, Math.min(25, initialPinchRadius * factor));
                }
            }, { passive: true });

            dom.addEventListener('touchend', (e) => {
                if (e.touches.length < 2) {
                    initialPinchDistance = null;
                }
            });
        }

        function resetCamera() {
            cameraRadius = 10;
            lon = 45;
            lat = 25;
        }

        function animate() {
            requestAnimationFrame(animate);

            const delta = clock.getDelta();
            gameState.animTimer += delta;

            // 平滑放大體型 (隨歲數線性與彈性長大)
            if (dog) {
                const currentS = dog.scale.x;
                const nextS = THREE.MathUtils.lerp(currentS, gameState.targetScale, delta * 2.5);
                dog.scale.set(nextS, nextS, nextS);
            }

            // 瞄準時讓地面目標圈輕微旋轉與縮放脈衝
            if (isAimingToy && groundTargetMarker && groundTargetMarker.visible) {
                groundTargetMarker.rotation.y += 1.5 * delta;
                const pulse = 1.0 + Math.sin(gameState.animTimer * 6) * 0.08;
                groundTargetMarker.scale.set(pulse, 1, pulse);
            }

            updateDogAI(delta);
            animateDogLimbs(delta);
            updateToysAndSupplies(delta);
            updateParticles(delta);

            if (skyMesh) skyMesh.rotation.y += 0.005 * delta;
            updateCameraPosition();
            renderer.render(scene, camera);
        }

        function updateDogAI(delta) {
            if (!dog) return;

            if (currentToy && gameState.dogState !== 'eating' && gameState.dogState !== 'drinking') {
                gameState.dogState = 'fetching';
                gameState.dogTargetPos.copy(currentToy.position);
            } 
            else if (activeBones.length > 0 && gameState.dogState !== 'fetching' && gameState.dogState !== 'eating' && gameState.dogState !== 'drinking') {
                gameState.dogState = 'fetching';
                gameState.dogTargetPos.copy(activeBones[0].position);
            }
            else if (activeWaterBowls.length > 0 && gameState.dogState !== 'fetching' && gameState.dogState !== 'eating' && gameState.dogState !== 'drinking') {
                gameState.dogState = 'fetching';
                gameState.dogTargetPos.copy(activeWaterBowls[0].position);
            }
            else if (gameState.dogState === 'idle' && Math.random() < 0.005) {
                gameState.dogState = 'walking';
                gameState.dogTargetPos.set(
                    (Math.random() - 0.5) * 20,
                    0,
                    (Math.random() - 0.5) * 20
                );
            }

            let distance = gameState.dogPos.distanceTo(gameState.dogTargetPos);

            if (distance > 0.4) {
                let targetAngle = Math.atan2(
                    gameState.dogTargetPos.x - gameState.dogPos.x,
                    gameState.dogTargetPos.z - gameState.dogPos.z
                );
                
                let diff = targetAngle - gameState.dogRotationY;
                while (diff < -Math.PI) diff += Math.PI * 2;
                while (diff > Math.PI) diff -= Math.PI * 2;
                gameState.dogRotationY += diff * 0.1;
                dog.rotation.y = gameState.dogRotationY;

                let maxSpeed = (gameState.dogState === 'fetching') ? 6.5 : 2.5;
                gameState.speed = THREE.MathUtils.lerp(gameState.speed, maxSpeed, 0.1);

                let moveStep = gameState.speed * delta;
                let dir = new THREE.Vector3().subVectors(gameState.dogTargetPos, gameState.dogPos).normalize();
                gameState.dogPos.addScaledVector(dir, moveStep);
                dog.position.copy(gameState.dogPos);

                if (Math.random() < 0.15) {
                    createDustParticle(gameState.dogPos);
                }

                const tag = document.getElementById('dog-state-tag');
                tag.innerText = (gameState.dogState === 'fetching') ? "興奮奔跑 🏃" : "快樂漫步 🐕";
                tag.className = "inline-block text-[10px] sm:text-xs bg-amber-500/20 text-amber-300 px-1.5 sm:px-2 py-0.5 rounded-full font-medium truncate max-w-[120px]";
            } else {
                if (gameState.dogState === 'fetching') {
                    if (currentToy) {
                        gameState.dogState = 'begging';
                        scene.remove(currentToy);
                        currentToy = null;
                        gameState.stats.joy = Math.min(100, gameState.stats.joy + 20);
                        createCelebrationHearts(3);
                        playBark();
                    } 
                    else if (activeBones.length > 0) {
                        gameState.dogState = 'eating';
                        let bone = activeBones.shift();
                        setTimeout(() => {
                            scene.remove(bone);
                            gameState.stats.hunger = Math.min(100, gameState.stats.hunger + 30);
                            gameState.stats.joy = Math.min(100, gameState.stats.joy + 10);
                            updateStatsUI();
                            gameState.dogState = 'idle';
                            createCelebrationHearts(4);
                        }, 2000);
                    }
                    else if (activeWaterBowls.length > 0) {
                        gameState.dogState = 'drinking';
                        let bowl = activeWaterBowls.shift();
                        setTimeout(() => {
                            scene.remove(bowl);
                            gameState.stats.thirst = Math.min(100, gameState.stats.thirst + 40);
                            updateStatsUI();
                            gameState.dogState = 'idle';
                            createCelebrationHearts(3);
                        }, 2000);
                    }
                } else if (gameState.dogState === 'walking') {
                    gameState.dogState = 'idle';
                }

                gameState.speed = THREE.MathUtils.lerp(gameState.speed, 0, 0.2);
                
                const tag = document.getElementById('dog-state-tag');
                if (gameState.dogState === 'idle') {
                    tag.innerText = "悠閒放鬆 💤";
                    tag.className = "inline-block text-[10px] sm:text-xs bg-green-500/20 text-green-400 px-1.5 sm:px-2 py-0.5 rounded-full font-medium truncate max-w-[120px]";
                } else if (gameState.dogState === 'eating') {
                    tag.innerText = "啃骨頭中 🦴";
                    tag.className = "inline-block text-[10px] sm:text-xs bg-pink-500/20 text-pink-400 px-1.5 sm:px-2 py-0.5 rounded-full font-medium truncate max-w-[120px]";
                } else if (gameState.dogState === 'drinking') {
                    tag.innerText = "大口喝水 💧";
                    tag.className = "inline-block text-[10px] sm:text-xs bg-blue-500/20 text-blue-400 px-1.5 sm:px-2 py-0.5 rounded-full font-medium truncate max-w-[120px]";
                } else if (gameState.dogState === 'begging') {
                    tag.innerText = "討摸摸 ✨";
                    tag.className = "inline-block text-[10px] sm:text-xs bg-yellow-500/20 text-yellow-300 px-1.5 sm:px-2 py-0.5 rounded-full font-medium truncate max-w-[120px]";
                    if (Math.random() < 0.005) gameState.dogState = 'idle';
                }
            }
        }

        function animateDogLimbs(delta) {
            if (!dog) return;

            const t = gameState.animTimer;
            const speedFactor = gameState.speed;

            if (gameState.dogState === 'eating' || gameState.dogState === 'drinking' || gameState.dogState === 'begging') {
                dog.tongue.visible = true;
                dog.tongue.scale.y = 1 + Math.sin(t * 12) * 0.15;
            } else {
                dog.tongue.visible = false;
            }

            let tailWagSpeed = 8;
            let tailWagAmp = 0.2;

            if (gameState.dogState === 'fetching' || gameState.dogState === 'begging') {
                tailWagSpeed = 22;
                tailWagAmp = 0.6;
            } else if (gameState.dogState === 'idle') {
                tailWagSpeed = 3;
                tailWagAmp = 0.1;
            }

            dog.tail.rotation.y = Math.sin(t * tailWagSpeed) * tailWagAmp;

            if (speedFactor > 0.1) {
                const walkCycle = t * speedFactor * 2.0;

                dog.legLF.rotation.x = Math.sin(walkCycle) * 0.6;
                dog.legRB.rotation.x = Math.sin(walkCycle) * 0.6;
                dog.legRF.rotation.x = -Math.sin(walkCycle) * 0.6;
                dog.legLB.rotation.x = -Math.sin(walkCycle) * 0.6;

                dog.body.position.y = 0.8 + Math.abs(Math.sin(walkCycle * 2)) * 0.15;
                dog.body.rotation.z = Math.sin(walkCycle) * 0.05;
                dog.head.rotation.x = -Math.abs(Math.cos(walkCycle)) * 0.1;
            } else {
                if (gameState.dogState === 'begging') {
                    dog.legLF.rotation.x = -Math.PI / 3;
                    dog.legRF.rotation.x = -Math.PI / 3;
                    dog.legLB.rotation.x = -Math.PI / 2;
                    dog.legRB.rotation.x = -Math.PI / 2;
                    dog.body.position.y = 0.5 + Math.sin(t * 6) * 0.05;
                    dog.body.rotation.x = -Math.PI / 6;
                    dog.head.rotation.x = 0.2 + Math.sin(t * 4) * 0.05;
                } else if (gameState.dogState === 'eating' || gameState.dogState === 'drinking') {
                    dog.legLF.rotation.x = 0;
                    dog.legRF.rotation.x = 0;
                    dog.body.position.y = 0.7;
                    dog.body.rotation.x = 0.1;
                    dog.head.rotation.x = 0.6 + Math.abs(Math.sin(t * 10)) * 0.2;
                } else {
                    dog.legLF.rotation.x = 0;
                    dog.legRF.rotation.x = 0;
                    dog.legLB.rotation.x = 0;
                    dog.legRB.rotation.x = 0;
                    dog.body.position.y = 0.8;
                    dog.body.rotation.x = 0;
                    dog.body.rotation.z = 0;
                    dog.head.rotation.x = Math.sin(t * 1.5) * 0.04;
                }
            }

            dog.leftEar.rotation.z = -0.2 + Math.sin(t * 5) * 0.03;
            dog.rightEar.rotation.z = 0.2 - Math.sin(t * 5) * 0.03;
        }

        function updateToysAndSupplies(delta) {
            if (currentToy) {
                if (currentToy.position.y > 0.15) {
                    // 套用重力 (小球標準重力 9.8，飛盤具有空氣動力學滑翔 4.8)
                    const grav = (currentToy.toyType === 'frisbee') ? 4.8 : 9.8;
                    currentToy.velocity.y -= grav * delta;
                    currentToy.position.addScaledVector(currentToy.velocity, delta);

                    if (currentToy.toyType === 'frisbee') {
                        currentToy.rotation.y += 15 * delta;
                        currentToy.rotation.z = Math.sin(gameState.animTimer * 8) * 0.1;
                    } else {
                        currentToy.rotation.x += 6 * delta;
                        currentToy.rotation.z += 6 * delta;
                    }
                } else {
                    currentToy.position.y = 0.15;
                    // 小球觸地彈跳物理 (Bounce)
                    if (currentToy.toyType === 'ball' && currentToy.bounceCount < 2 && Math.abs(currentToy.velocity.y) > 1.2) {
                        currentToy.bounceCount++;
                        currentToy.velocity.y = -currentToy.velocity.y * 0.45;
                        currentToy.velocity.x *= 0.6;
                        currentToy.velocity.z *= 0.6;
                        createDustParticle(currentToy.position);
                    } else {
                        currentToy.velocity.set(0, 0, 0);
                    }
                }
            }
        }

        function updateParticles(delta) {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= delta;
                p.mesh.position.addScaledVector(p.velocity, delta);
                
                if (p.type === 'heart') {
                    p.mesh.scale.addScalar(delta * 0.8);
                } else if (p.type === 'dust') {
                    p.mesh.scale.subScalar(delta * 0.4);
                }

                if (p.life <= 0 || p.mesh.scale.x <= 0) {
                    scene.remove(p.mesh);
                    particles.splice(i, 1);
                }
            }
        }

        function createCelebrationHearts(count) {
            for (let i = 0; i < count; i++) {
                setTimeout(() => {
                    if (dog) {
                        const headPos = new THREE.Vector3();
                        dog.head.getWorldPosition(headPos);
                        createHeartParticle(headPos);
                    }
                }, i * 200);
            }
        }

        function updateCameraPosition() {
            if (!dog) return;

            const phi = THREE.MathUtils.degToRad(90 - lat);
            const theta = THREE.MathUtils.degToRad(lon);

            if (gameState.cameraMode === 'orbit') {
                const targetX = dog.position.x + cameraRadius * Math.sin(phi) * Math.sin(theta);
                const targetY = dog.position.y + cameraRadius * Math.cos(phi) + 1.0;
                const targetZ = dog.position.z + cameraRadius * Math.sin(phi) * Math.cos(theta);

                camera.position.set(targetX, targetY, targetZ);
                camera.lookAt(dog.position.x, dog.position.y + 0.5, dog.position.z);
            } 
            else if (gameState.cameraMode === 'follow') {
                const dogRot = dog.rotation.y;
                const offsetDistance = Math.max(5, cameraRadius * 0.6);
                const offsetHeight = offsetDistance * 0.5;

                const targetCamX = dog.position.x - Math.sin(dogRot) * offsetDistance;
                const targetCamZ = dog.position.z - Math.cos(dogRot) * offsetDistance;
                const targetCamY = dog.position.y + offsetHeight;

                camera.position.lerp(new THREE.Vector3(targetCamX, targetCamY, targetCamZ), 0.08);
                camera.lookAt(dog.position.x, dog.position.y + 0.6, dog.position.z);
            } 
            else if (gameState.cameraMode === 'dog') {
                const headWorldPos = new THREE.Vector3();
                dog.head.mainHead.getWorldPosition(headWorldPos);

                camera.position.copy(headWorldPos);
                const lookDir = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
                const lookTarget = new THREE.Vector3().addVectors(headWorldPos, lookDir);
                camera.lookAt(lookTarget);
            }
        }

        function triggerAction(action) {
            if (!dog) return;

            if (action === 'throw_ball') {
                openThrowAimHUD('ball');
                return;
            }

            if (action === 'throw_frisbee') {
                openThrowAimHUD('frisbee');
                return;
            }

            if (action === 'whistle') {
                gameState.dogTargetPos.set(0, 0, 0);
                gameState.dogState = 'fetching';
                playBark();
                createCelebrationHearts(1);
                gameState.stats.joy = Math.min(100, gameState.stats.joy + 5);
                updateStatsUI();
                return;
            }

            if (action === 'feed') {
                const boneGroup = new THREE.Group();
                const boneMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.8 });
                
                const centerGeo = new THREE.CylinderGeometry(0.08, 0.08, 0.6, 8);
                centerGeo.rotateZ(Math.PI/2);
                const center = new THREE.Mesh(centerGeo, boneMat);
                boneGroup.add(center);

                const jointGeo = new THREE.SphereGeometry(0.12, 8, 8);
                const j1 = new THREE.Mesh(jointGeo, boneMat); j1.position.set(-0.3, 0.08, 0.08); boneGroup.add(j1);
                const j2 = j1.clone(); j2.position.set(-0.3, -0.08, -0.08); boneGroup.add(j2);
                const j3 = j1.clone(); j3.position.x = 0.3; boneGroup.add(j3);
                const j4 = j2.clone(); j4.position.x = 0.3; boneGroup.add(j4);

                const dir = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
                const targetPos = new THREE.Vector3().addVectors(dog.position, dir.multiplyScalar(3));
                targetPos.y = 0.1;

                boneGroup.position.copy(targetPos);
                scene.add(boneGroup);
                activeBones.push(boneGroup);
                return;
            }

            if (action === 'water') {
                const bowlGroup = new THREE.Group();
                const bowlMat = new THREE.MeshStandardMaterial({ color: 0xff4444, roughness: 0.5 });
                
                const outerGeo = new THREE.CylinderGeometry(0.4, 0.3, 0.25, 12);
                const outer = new THREE.Mesh(outerGeo, bowlMat);
                bowlGroup.add(outer);

                const waterGeo = new THREE.CylinderGeometry(0.36, 0.36, 0.02, 12);
                const waterMat = new THREE.MeshStandardMaterial({ color: 0x3399ff, roughness: 0.1, metalness: 0.8 });
                const water = new THREE.Mesh(waterGeo, waterMat);
                water.position.y = 0.08;
                bowlGroup.add(water);

                const dir = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
                const targetPos = new THREE.Vector3().addVectors(dog.position, dir.multiplyScalar(2.5));
                targetPos.y = 0.12;

                bowlGroup.position.copy(targetPos);
                scene.add(bowlGroup);
                activeWaterBowls.push(bowlGroup);
                return;
            }

            if (action === 'pet') {
                gameState.stats.joy = Math.min(100, gameState.stats.joy + 15);
                updateStatsUI();
                createCelebrationHearts(5);
                playBark();
            }
        }

        // 打開力量曲線控制介面
        function openThrowAimHUD(type) {
            isAimingToy = true;
            aimingToyType = type;

            const hud = document.getElementById('throw-power-hud');
            const title = document.getElementById('throw-hud-title');
            const icon = document.getElementById('throw-hud-icon');

            if (type === 'ball') {
                title.innerText = "投擲小球力度控制";
                icon.innerText = "⚾";
            } else {
                title.innerText = "發射飛盤力度控制";
                icon.innerText = "🥏";
            }

            hud.classList.remove('hidden');
            hud.classList.add('flex');

            const slider = document.getElementById('throw-power-slider');
            if (slider) slider.value = currentThrowDistance;

            updateTrajectoryAndHUD(currentThrowDistance);
        }

        // 關閉力量曲線控制介面並隱藏 3D 瞄準輔助標記
        function closeThrowAimHUD() {
            isAimingToy = false;
            const hud = document.getElementById('throw-power-hud');
            if (hud) {
                hud.classList.add('hidden');
                hud.classList.remove('flex');
            }
            if (groundTargetMarker) groundTargetMarker.visible = false;
            if (trajectoryArcMesh) trajectoryArcMesh.visible = false;
        }

        // 滑桿拖動時更新當前投擲距離與軌跡
        function onPowerSliderChange(val) {
            currentThrowDistance = parseFloat(val);
            updateTrajectoryAndHUD(currentThrowDistance);
        }

        // 計算並更新 2D SVG 力量曲線與 3D 世界預覽
        function updateTrajectoryAndHUD(dist) {
            // 1. 更新文字標籤
            const label = document.getElementById('power-level-label');
            const targetText = document.getElementById('svg-target-text');
            if (dist <= 5.5) {
                label.innerText = `近距輕拋 (${dist}m)`;
                label.className = "text-sky-400 font-bold";
            } else if (dist <= 8) {
                label.innerText = `標準投擲 (${dist}m)`;
                label.className = "text-amber-400 font-bold";
            } else {
                label.innerText = `活力遠投 (${dist}m)`;
                label.className = "text-rose-400 font-bold";
            }
            if (targetText) targetText.innerText = `${dist}m`;

            // 2. 更新 SVG 拋物線幾何 (範圍 4m - 10m 對應 SVG X 軸 90 到 260)
            const progress = (dist - 4) / (10 - 4);
            const targetX = 90 + progress * 170;
            const apexY = 55 - progress * 45; // 力量越大拋物頂點越高
            const apexX = (25 + targetX) / 2;

            const pathElem = document.getElementById('svg-trajectory-path');
            const apexCircle = document.getElementById('svg-apex-point');
            const targetCircle = document.getElementById('svg-target-point');

            if (pathElem) {
                pathElem.setAttribute('d', `M 25 70 Q ${apexX} ${apexY} ${targetX} 70`);
            }
            if (apexCircle) {
                apexCircle.setAttribute('cx', apexX);
                apexCircle.setAttribute('cy', apexY);
            }
            if (targetCircle) {
                targetCircle.setAttribute('cx', targetX);
            }
            if (targetText) {
                targetText.setAttribute('x', targetX);
            }

            // 3. 更新 3D 世界落點標記與立體弧線
            if (!dog) return;

            // 計算投擲朝向：從小狗背後向前投出，或沿小狗正面延伸
            const dogForward = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
            const landingPos = new THREE.Vector3().addVectors(dog.position, dogForward.clone().multiplyScalar(dist));
            landingPos.y = 0.05;

            if (groundTargetMarker) {
                groundTargetMarker.position.copy(landingPos);
                groundTargetMarker.visible = true;
            }

            // 計算 3D 拋物曲線點
            if (trajectoryArcMesh) {
                const startPos = new THREE.Vector3(dog.position.x, 1.2, dog.position.z);
                const apexHeight = 2.5 + progress * 3.5;
                const midPos = new THREE.Vector3().addVectors(startPos, landingPos).multiplyScalar(0.5);
                midPos.y = apexHeight;

                const curve = new THREE.QuadraticBezierCurve3(startPos, midPos, landingPos);
                const points = curve.getPoints(29);
                const posAttr = trajectoryArcMesh.geometry.attributes.position;
                for (let i = 0; i < points.length; i++) {
                    posAttr.setXYZ(i, points[i].x, points[i].y, points[i].z);
                }
                posAttr.needsUpdate = true;
                trajectoryArcMesh.computeLineDistances();
                trajectoryArcMesh.visible = true;
            }
        }

        // 確認發射小球或飛盤
        function confirmLaunchToy() {
            if (!dog) return;

            const type = aimingToyType;
            const dist = currentThrowDistance;
            closeThrowAimHUD();

            if (currentToy) {
                scene.remove(currentToy);
                currentToy = null;
            }

            // 計算目標落點座標
            const dogForward = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
            const targetLanding = new THREE.Vector3().addVectors(dog.position, dogForward.clone().multiplyScalar(dist));
            targetLanding.y = 0.15;

            // 起始位置 (小狗稍後方上空)
            const startPos = new THREE.Vector3(
                dog.position.x - dogForward.x * 2,
                2.5,
                dog.position.z - dogForward.z * 2
            );

            // 計算物理拋物線初速度向量
            const flightTime = 1.0 + dist * 0.045; // 飛行秒數
            const grav = (type === 'frisbee') ? 4.8 : 9.8;
            
            const vx = (targetLanding.x - startPos.x) / flightTime;
            const vz = (targetLanding.z - startPos.z) / flightTime;
            const vy = (targetLanding.y - startPos.y + 0.5 * grav * flightTime * flightTime) / flightTime;

            if (type === 'ball') {
                const ballGeo = new THREE.SphereGeometry(0.22, 16, 16);
                const ballMat = new THREE.MeshStandardMaterial({ color: 0x39ff14, roughness: 0.3 });
                const ball = new THREE.Mesh(ballGeo, ballMat);
                ball.position.copy(startPos);
                ball.velocity = new THREE.Vector3(vx, vy, vz);
                ball.castShadow = true;
                ball.toyType = 'ball';
                ball.bounceCount = 0;

                scene.add(ball);
                currentToy = ball;
            } else {
                const frisbeeGeo = new THREE.CylinderGeometry(0.42, 0.42, 0.06, 16);
                const frisbeeMat = new THREE.MeshStandardMaterial({ color: 0xbf00ff, roughness: 0.5 });
                const frisbee = new THREE.Mesh(frisbeeGeo, frisbeeMat);
                frisbee.position.copy(startPos);
                frisbee.velocity = new THREE.Vector3(vx, vy, vz);
                frisbee.castShadow = true;
                frisbee.toyType = 'frisbee';

                scene.add(frisbee);
                currentToy = frisbee;
            }

            // 觸發音效
            playWhooshSound();
        }

        // 發射破空微音效
        function playWhooshSound() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(200, audioCtx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(600, audioCtx.currentTime + 0.15);
                gain.gain.setValueAtTime(0.25, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.25);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.26);
            } catch (e) {
                console.log("Audio not supported");
            }
        }

        // 檢查是否三大指數同時達到 95% 以上並觸發長大歲數機制
        function checkAgeUp() {
            if (isAgeUpProcessing) return;

            // 判斷三個數值是否皆達到 95% 以上
            if (gameState.stats.joy >= 95 && gameState.stats.hunger >= 95 && gameState.stats.thirst >= 95) {
                isAgeUpProcessing = true;
                gameState.age += 1;
                
                // 體型依歲數逐步擴大：以 3 歲為基準 (1.0)，每長 1 歲體積增加 12%
                gameState.targetScale = 1.0 + (gameState.age - 3) * 0.12;

                // 更新年齡徽章與彈窗內容
                const ageBadge = document.getElementById('dog-age-badge');
                if (ageBadge) ageBadge.innerText = `${gameState.age} 歲`;

                const toastAge = document.getElementById('ageup-toast-age');
                if (toastAge) toastAge.innerText = `${gameState.age} 歲`;

                const toastDesc = document.getElementById('ageup-toast-desc');
                if (toastDesc) {
                    toastDesc.innerText = `指數皆達 95% 以上！${gameState.dogName} 成長至 ${gameState.age} 歲，體型大幅成長！`;
                }

                // 播放升級專屬慶祝音效與特效
                playLevelUpFanfare();
                createCelebrationHearts(12);

                // 顯示成長橫幅
                const toast = document.getElementById('ageup-toast');
                if (toast) {
                    toast.classList.remove('scale-0', 'opacity-0');
                    toast.classList.add('scale-100', 'opacity-100');
                }

                // 進入討摸摸/歡呼慶祝狀態
                const oldState = gameState.dogState;
                gameState.dogState = 'begging';

                // 成長消耗：稍微扣除部分數值至 75%，開啟下一階段的照護養成循環
                setTimeout(() => {
                    gameState.stats.joy = 75;
                    gameState.stats.hunger = 75;
                    gameState.stats.thirst = 75;
                    updateStatsUI();

                    if (toast) {
                        toast.classList.remove('scale-100', 'opacity-100');
                        toast.classList.add('scale-0', 'opacity-0');
                    }
                    gameState.dogState = 'idle';
                    isAgeUpProcessing = false;
                }, 3800);
            }
        }

        function playLevelUpFanfare() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const notes = [329.63, 440.00, 554.37, 659.25, 880.00]; // E4, A4, C#5, E5, A5 升級號角和弦
                notes.forEach((freq, idx) => {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, audioCtx.currentTime + idx * 0.12);
                    
                    gain.gain.setValueAtTime(0.3, audioCtx.currentTime + idx * 0.12);
                    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + idx * 0.12 + 0.35);
                    
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    
                    osc.start(audioCtx.currentTime + idx * 0.12);
                    osc.stop(audioCtx.currentTime + idx * 0.12 + 0.36);
                });
            } catch (e) {
                console.log("Audio fanfare error", e);
            }
        }

        function playBark() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                for(let i=0; i<2; i++) {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(320 + (i*50), audioCtx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(100, audioCtx.currentTime + 0.15);
                    
                    gain.gain.setValueAtTime(0.4, audioCtx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.2);
                    
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    
                    osc.start();
                    osc.stop(audioCtx.currentTime + 0.22);
                }
            } catch (e) {
                console.log("AudioContext blocked or unavailable");
            }
        }

        function updateStatsUI() {
            document.getElementById('stat-joy-val').innerText = Math.round(gameState.stats.joy) + '%';
            document.getElementById('stat-joy-bar').style.width = gameState.stats.joy + '%';
            
            document.getElementById('stat-hunger-val').innerText = Math.round(gameState.stats.hunger) + '%';
            document.getElementById('stat-hunger-bar').style.width = gameState.stats.hunger + '%';
            
            document.getElementById('stat-thirst-val').innerText = Math.round(gameState.stats.thirst) + '%';
            document.getElementById('stat-thirst-bar').style.width = gameState.stats.thirst + '%';

            // 每次數值變化時檢驗是否達成達標 95% 成長條件
            checkAgeUp();
        }

        // 行動端折疊/展開小狗狀態面板
        function toggleStatusExpand() {
            const stats = document.getElementById('stats-container');
            const chevron = document.getElementById('status-chevron');
            isStatusExpanded = !isStatusExpanded;
            
            if (isStatusExpanded) {
                stats.classList.remove('hidden');
                if (chevron) chevron.classList.add('rotate-180');
            } else {
                // 僅在小螢幕時隱藏
                if (window.innerWidth < 640) {
                    stats.classList.add('hidden');
                }
                if (chevron) chevron.classList.remove('rotate-180');
            }
        }

        // 沉浸視野模式 (Zen mode)：隱藏所有浮層 UI
        function toggleZenMode() {
            isZenMode = !isZenMode;
            const gameUI = document.getElementById('game-ui');
            const restoreBtn = document.getElementById('zen-restore-btn');
            
            if (isZenMode) {
                gameUI.classList.add('opacity-0', 'pointer-events-none');
                restoreBtn.classList.remove('hidden');
                restoreBtn.classList.add('flex');
            } else {
                gameUI.classList.remove('opacity-0');
                restoreBtn.classList.add('hidden');
                restoreBtn.classList.remove('flex');
            }
        }

        function setCameraMode(mode) {
            gameState.cameraMode = mode;
            
            const orbitBtn = document.getElementById('cam-orbit-btn');
            const followBtn = document.getElementById('cam-follow-btn');
            const dogBtn = document.getElementById('cam-dog-btn');

            const inactiveClasses = "p-2 sm:px-3 sm:py-2 rounded-lg text-slate-300 hover:bg-slate-800 text-xs sm:text-sm font-medium flex items-center gap-2 active-bounce transition-all";
            const activeClasses = "p-2 sm:px-3 sm:py-2 rounded-lg bg-amber-500 text-white text-xs sm:text-sm font-medium flex items-center gap-2 active-bounce transition-all";

            orbitBtn.className = inactiveClasses;
            followBtn.className = inactiveClasses;
            dogBtn.className = inactiveClasses;

            if (mode === 'orbit') {
                orbitBtn.className = activeClasses;
                resetCamera();
            } else if (mode === 'follow') {
                followBtn.className = activeClasses;
            } else if (mode === 'dog') {
                dogBtn.className = activeClasses;
            }
        }

        function toggleCustomizer() {
            const panel = document.getElementById('customizer-panel');
            panel.classList.toggle('translate-x-full');
        }

        function changeDogBreed(breed) {
            ['shiba', 'golden', 'husky', 'dalmatian'].forEach(b => {
                const el = document.getElementById('breed-' + b);
                if (el) el.className = "p-2.5 sm:p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-700 active-bounce";
            });

            document.getElementById('breed-' + breed).className = "p-2.5 sm:p-3 rounded-xl border-2 border-amber-500 bg-amber-500/10 text-left transition-all active-bounce";
            gameState.breed = breed;
            updateDogBreedColor();
            createCelebrationHearts(3);
        }

        function changeAccessory(acc) {
            ['none', 'party', 'crown'].forEach(a => {
                const el = document.getElementById('acc-' + a);
                if (el) el.className = "p-2 sm:p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-700 active-bounce";
            });

            document.getElementById('acc-' + acc).className = "p-2 sm:p-2.5 rounded-lg border-2 border-amber-500 bg-amber-500/10 text-amber-400 font-bold active-bounce";
            gameState.accessory = acc;
            updateDogAccessory();
            createCelebrationHearts(2);
        }

        function changeTimeOfDay(time) {
            ['day', 'sunset', 'night'].forEach(t => {
                const el = document.getElementById('time-' + t);
                if (el) el.className = "p-2 sm:p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700 active-bounce";
            });

            document.getElementById('time-' + time).className = "p-2 sm:p-2.5 rounded-lg border border-amber-500/50 bg-amber-500/10 text-amber-400 font-bold active-bounce";
            gameState.timeOfDay = time;

            if (time === 'day') {
                scene.fog.color.setHex(colors.sky.day);
                skyMesh.material.color.setHex(colors.sky.day);
                floor.material.color.setHex(colors.grass.day);
                sunLight.color.setHex(0xffffff);
                sunLight.intensity = 1.2;
                hemiLight.color.setHex(0xffffff);
                renderer.setClearColor(colors.sky.day);
            } 
            else if (time === 'sunset') {
                scene.fog.color.setHex(colors.sky.sunset);
                skyMesh.material.color.setHex(colors.sky.sunset);
                floor.material.color.setHex(colors.grass.sunset);
                sunLight.color.setHex(0xffaa44);
                sunLight.intensity = 1.0;
                hemiLight.color.setHex(0xffaa66);
                renderer.setClearColor(colors.sky.sunset);
            } 
            else if (time === 'night') {
                scene.fog.color.setHex(colors.sky.night);
                skyMesh.material.color.setHex(colors.sky.night);
                floor.material.color.setHex(colors.grass.night);
                sunLight.color.setHex(0x5588ff);
                sunLight.intensity = 0.3;
                hemiLight.color.setHex(0x112244);
                renderer.setClearColor(colors.sky.night);
            }
        }

        function renameDog() {
            document.getElementById('dog-name-input').value = gameState.dogName;
            document.getElementById('rename-modal').style.display = 'flex';
        }

        function closeRenameModal() {
            document.getElementById('rename-modal').style.display = 'none';
        }

        function saveDogName() {
            const name = document.getElementById('dog-name-input').value.trim();
            if (name) {
                gameState.dogName = name;
                document.getElementById('dog-name-display').innerText = name;
            }
            closeRenameModal();
        }

        function onWindowResize() {
            if (!camera || !renderer) return;
            const width = window.innerWidth;
            const height = window.innerHeight;
            
            camera.aspect = width / height;
            camera.updateProjectionMatrix();
            renderer.setSize(width, height);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

            // 自動適配狀態條面板
            const stats = document.getElementById('stats-container');
            if (width >= 640) {
                stats.classList.remove('hidden');
            } else if (!isStatusExpanded) {
                stats.classList.add('hidden');
            }
        }
    </script>
</body>
</html>
