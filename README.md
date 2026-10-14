<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>學生作品範例：3D萌寵小狗空間 (Dog Simulator 3D)</title>
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
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
        }
        #canvas-container {
            width: 100%;
            height: 100vh;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        /* 隱藏滾動條 */
        ::-webkit-scrollbar {
            display: none;
        }
        .ui-element {
            z-index: 10;
            pointer-events: auto;
        }
        /* 體驗流暢過渡 */
        .transition-all {
            transition: all 0.3s ease;
        }
    </style>
</head>
<body class="bg-slate-900 text-white">

    <!-- 遊戲加載畫面 -->
    <div id="loader" class="fixed inset-0 bg-slate-950 flex flex-col items-center justify-center z-50 transition-opacity duration-500">
        <div class="text-center">
            <i class="fas fa-dog text-6xl text-amber-500 animate-bounce mb-4"></i>
            <h1 class="text-3xl font-extrabold tracking-wider text-amber-400 mb-2">3D 超寫實萌寵樂園</h1>
            <p class="text-slate-400 text-sm mb-6">正在打造您的 3A 級小狗專屬世界...</p>
            <div class="w-64 bg-slate-800 rounded-full h-2.5 overflow-hidden mx-auto">
                <div id="loading-bar" class="bg-amber-500 h-full w-0 transition-all duration-300"></div>
            </div>
            <p id="loading-status" class="text-xs text-slate-500 mt-2">載入 3D 著色器...</p>
        </div>
    </div>

    <!-- 遊戲主畫面 UI -->
    <div id="game-ui" class="absolute inset-0 pointer-events-none flex flex-col justify-between p-4 md:p-6 z-10">
        
        <!-- 頂部資訊欄 -->
        <div class="flex justify-between items-start w-full">
            <!-- 左上角：小狗狀態 -->
            <div class="ui-element bg-slate-900/80 backdrop-blur-md rounded-2xl p-4 border border-slate-700/50 shadow-2xl max-w-xs">
                <div class="flex items-center gap-3 mb-3">
                    <div class="w-12 h-12 rounded-full bg-amber-500 border-2 border-amber-400 flex items-center justify-center text-2xl shadow-inner overflow-hidden">
                        🐶
                    </div>
                    <div>
                        <h2 class="font-bold text-lg text-amber-300 flex items-center gap-2">
                            <span id="dog-name-display">巴弟 (Buddy)</span>
                            <button onclick="renameDog()" class="text-xs text-slate-400 hover:text-white transition-colors">
                                <i class="fas fa-pen"></i>
                            </button>
                        </h2>
                        <span id="dog-state-tag" class="text-xs bg-green-500/20 text-green-400 px-2 py-0.5 rounded-full font-medium">悠閒散步中</span>
                    </div>
                </div>
                
                <!-- 狀態條 -->
                <div class="space-y-2 text-xs">
                    <div>
                        <div class="flex justify-between mb-1 text-slate-300">
                            <span><i class="fas fa-heart text-red-500 mr-1"></i> 心情值</span>
                            <span id="stat-joy-val">80%</span>
                        </div>
                        <div class="w-full bg-slate-700 rounded-full h-1.5 overflow-hidden">
                            <div id="stat-joy-bar" class="bg-gradient-to-r from-red-500 to-pink-500 h-full transition-all duration-300" style="width: 80%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between mb-1 text-slate-300">
                            <span><i class="fas fa-meat text-amber-600 mr-1"></i> 飽食度</span>
                            <span id="stat-hunger-val">70%</span>
                        </div>
                        <div class="w-full bg-slate-700 rounded-full h-1.5 overflow-hidden">
                            <div id="stat-hunger-bar" class="bg-gradient-to-r from-amber-600 to-yellow-500 h-full transition-all duration-300" style="width: 70%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between mb-1 text-slate-300">
                            <span><i class="fas fa-tint text-blue-400 mr-1"></i> 水分值</span>
                            <span id="stat-thirst-val">60%</span>
                        </div>
                        <div class="w-full bg-slate-700 rounded-full h-1.5 overflow-hidden">
                            <div id="stat-thirst-bar" class="bg-gradient-to-r from-blue-500 to-cyan-400 h-full transition-all duration-300" style="width: 60%"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 右上角：設定與視角控制 -->
            <div class="ui-element flex flex-col gap-2">
                <!-- 視角切換 -->
                <div class="bg-slate-900/80 backdrop-blur-md rounded-xl p-2 border border-slate-700/50 shadow-lg flex flex-col gap-1">
                    <button onclick="setCameraMode('orbit')" id="cam-orbit-btn" class="p-2.5 rounded-lg bg-amber-500 text-white text-sm font-medium flex items-center gap-2 transition-all">
                        <i class="fas fa-video w-5 text-center"></i> 自由上帝視角
                    </button>
                    <button onclick="setCameraMode('follow')" id="cam-follow-btn" class="p-2.5 rounded-lg text-slate-300 hover:bg-slate-800 text-sm font-medium flex items-center gap-2 transition-all">
                        <i class="fas fa-running w-5 text-center"></i> 追隨小狗視角
                    </button>
                    <button onclick="setCameraMode('dog')" id="cam-dog-btn" class="p-2.5 rounded-lg text-slate-300 hover:bg-slate-800 text-sm font-medium flex items-center gap-2 transition-all">
                        <i class="fas fa-eye w-5 text-center"></i> 第一人稱狗眼
                    </button>
                </div>

                <!-- 自訂小狗外觀 -->
                <button onclick="toggleCustomizer()" class="bg-slate-900/80 backdrop-blur-md hover:bg-slate-800 rounded-xl p-3 border border-slate-700/50 shadow-lg flex items-center justify-between text-sm font-medium text-amber-400 transition-all">
                    <span><i class="fas fa-palette mr-2"></i> 自訂小狗造型</span>
                    <i class="fas fa-chevron-right text-xs"></i>
                </button>
            </div>
        </div>

        <!-- 畫面中央：操作手勢指示（僅在無操作時提示） -->
        <div id="gesture-hint" class="self-center text-center opacity-70 mb-12 pointer-events-none transition-opacity duration-1000">
            <p class="text-sm bg-black/40 px-4 py-2 rounded-full backdrop-blur-sm"><i class="fas fa-mouse-pointer mr-1 text-amber-400 animate-pulse"></i> 拖拽滑鼠可旋轉視野 / 滾輪可縮放</p>
        </div>

        <!-- 底部控制欄 -->
        <div class="flex flex-col gap-4 w-full">
            <!-- 互動動作快捷按鈕 -->
            <div class="ui-element flex justify-center gap-3 overflow-x-auto py-2">
                <button onclick="triggerAction('feed')" class="flex flex-col items-center gap-1 bg-amber-600 hover:bg-amber-500 text-white px-5 py-3 rounded-2xl shadow-xl border border-amber-400/30 min-w-[80px] transform active:scale-95 transition-all">
                    <i class="fas fa-bone text-2xl"></i>
                    <span class="text-xs font-bold">餵食骨頭</span>
                </button>
                <button onclick="triggerAction('water')" class="flex flex-col items-center gap-1 bg-blue-600 hover:bg-blue-500 text-white px-5 py-3 rounded-2xl shadow-xl border border-blue-400/30 min-w-[80px] transform active:scale-95 transition-all">
                    <i class="fas fa-faucet-drip text-2xl"></i>
                    <span class="text-xs font-bold">給予清水</span>
                </button>
                <button onclick="triggerAction('throw_ball')" class="flex flex-col items-center gap-1 bg-green-600 hover:bg-green-500 text-white px-5 py-3 rounded-2xl shadow-xl border border-green-400/30 min-w-[80px] transform active:scale-95 transition-all">
                    <i class="fas fa-baseball-ball text-2xl animate-spin"></i>
                    <span class="text-xs font-bold">投擲小球</span>
                </button>
                <button onclick="triggerAction('throw_frisbee')" class="flex flex-col items-center gap-1 bg-purple-600 hover:bg-purple-500 text-white px-5 py-3 rounded-2xl shadow-xl border border-purple-400/30 min-w-[80px] transform active:scale-95 transition-all">
                    <i class="fas fa-compact-disc text-2xl"></i>
                    <span class="text-xs font-bold">發射飛盤</span>
                </button>
                <button onclick="triggerAction('whistle')" class="flex flex-col items-center gap-1 bg-red-600 hover:bg-red-500 text-white px-5 py-3 rounded-2xl shadow-xl border border-red-400/30 min-w-[80px] transform active:scale-95 transition-all">
                    <i class="fas fa-volume-up text-2xl"></i>
                    <span class="text-xs font-bold">吹口哨召回</span>
                </button>
                <button onclick="triggerAction('pet')" class="flex flex-col items-center gap-1 bg-pink-600 hover:bg-pink-500 text-white px-5 py-3 rounded-2xl shadow-xl border border-pink-400/30 min-w-[80px] transform active:scale-95 transition-all">
                    <i class="fas fa-hand-holding-heart text-2xl"></i>
                    <span class="text-xs font-bold">摸摸頭</span>
                </button>
            </div>

            <!-- 底端免責/提示資訊 -->
            <div class="text-center text-xs text-slate-400 font-mono">
                Three.js 3A 級渲染引擎 &middot; 自訂物理動畫核心 &middot; 按下上方動作與小狗互動！
            </div>
        </div>
    </div>

    <!-- 自訂外觀面板 (滑出式) -->
    <div id="customizer-panel" class="ui-element fixed right-0 top-0 bottom-0 w-80 bg-slate-900/95 backdrop-blur-lg border-l border-slate-700/50 shadow-2xl z-20 p-6 flex flex-col justify-between transform translate-x-full transition-transform duration-300 ease-out">
        <div class="space-y-6">
            <div class="flex justify-between items-center border-b border-slate-800 pb-4">
                <h3 class="text-xl font-extrabold text-amber-400"><i class="fas fa-dog mr-2"></i>小狗外觀設定</h3>
                <button onclick="toggleCustomizer()" class="text-slate-400 hover:text-white"><i class="fas fa-times text-lg"></i></button>
            </div>

            <!-- 品種選取 (更換材質配色) -->
            <div class="space-y-3">
                <label class="text-sm font-semibold text-slate-300 block">1. 選擇毛色品種</label>
                <div class="grid grid-cols-2 gap-2">
                    <button onclick="changeDogBreed('shiba')" id="breed-shiba" class="p-3 rounded-xl border-2 border-amber-500 bg-amber-500/10 text-left transition-all">
                        <div class="font-bold text-sm text-amber-400">秋田/柴犬</div>
                        <div class="text-xs text-slate-400 mt-1">經典橘白相間</div>
                    </button>
                    <button onclick="changeDogBreed('golden')" id="breed-golden" class="p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750">
                        <div class="font-bold text-sm text-yellow-400">黃金獵犬</div>
                        <div class="text-xs text-slate-400 mt-1">溫暖亮麗金黃</div>
                    </button>
                    <button onclick="changeDogBreed('husky')" id="breed-husky" class="p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750">
                        <div class="font-bold text-sm text-slate-300">哈士奇</div>
                        <div class="text-xs text-slate-400 mt-1">酷炫灰黑白斑</div>
                    </button>
                    <button onclick="changeDogBreed('dalmatian')" id="breed-dalmatian" class="p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750">
                        <div class="font-bold text-sm text-white">大麥町</div>
                        <div class="text-xs text-slate-400 mt-1">逗趣黑白斑點</div>
                    </button>
                </div>
            </div>

            <!-- 配飾選取 (戴帽子) -->
            <div class="space-y-3">
                <label class="text-sm font-semibold text-slate-300 block">2. 寵物潮流配飾</label>
                <div class="grid grid-cols-3 gap-2 text-center text-xs">
                    <button onclick="changeAccessory('none')" id="acc-none" class="p-2.5 rounded-lg border-2 border-amber-500 bg-amber-500/10 font-bold">
                        無配飾
                    </button>
                    <button onclick="changeAccessory('party')" id="acc-party" class="p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-705">
                        🎉 派對帽
                    </button>
                    <button onclick="changeAccessory('crown')" id="acc-crown" class="p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-705">
                        👑 皇冠
                    </button>
                </div>
            </div>

            <!-- 草原環境更換 -->
            <div class="space-y-3">
                <label class="text-sm font-semibold text-slate-300 block">3. 改變世界時間</label>
                <div class="grid grid-cols-3 gap-2 text-center text-xs">
                    <button onclick="changeTimeOfDay('day')" id="time-day" class="p-2.5 rounded-lg border border-amber-500/50 bg-amber-500/10 text-amber-400 font-bold">
                        ☀️ 白天
                    </button>
                    <button onclick="changeTimeOfDay('sunset')" id="time-sunset" class="p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700">
                        🌅 夕陽
                    </button>
                    <button onclick="changeTimeOfDay('night')" id="time-night" class="p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700">
                        🌌 夜晚
                    </button>
                </div>
            </div>
        </div>

        <div class="border-t border-slate-800 pt-4">
            <button onclick="toggleCustomizer()" class="w-full py-3 bg-amber-500 hover:bg-amber-400 text-slate-950 font-extrabold rounded-xl transition-colors shadow-lg shadow-amber-500/20">
                套用並返回
            </button>
        </div>
    </div>

    <!-- 自訂狗名彈窗 -->
    <div id="rename-modal" class="ui-element fixed inset-0 bg-black/60 backdrop-blur-sm hidden items-center justify-center z-50">
        <div class="bg-slate-900 border border-slate-700 rounded-2xl p-6 w-80 shadow-2xl">
            <h3 class="text-lg font-bold text-amber-400 mb-4"><i class="fas fa-tags mr-2"></i>為您的小狗命名</h3>
            <input type="text" id="dog-name-input" class="w-full bg-slate-800 border border-slate-700 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-amber-500 mb-4" placeholder="輸入名字..." maxLength="10">
            <div class="flex gap-2 justify-end">
                <button onclick="closeRenameModal()" class="px-4 py-2 text-sm text-slate-400 hover:text-white">取消</button>
                <button onclick="saveDogName()" class="px-5 py-2 text-sm bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold rounded-lg transition-colors">確認修改</button>
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
        let currentToy = null; // 當前投擲的玩具（球或飛盤）
        let activeBones = [];  // 草地上的骨頭
        let activeWaterBowls = []; // 草地上的水盆
        let particles = [];    // 粒子系統

        // 遊戲狀態與數值
        const gameState = {
            dogName: "巴弟 (Buddy)",
            breed: "shiba", // shiba, golden, husky, dalmatian
            accessory: "none", // none, party, crown
            timeOfDay: "day", // day, sunset, night
            cameraMode: "orbit", // orbit, follow, dog
            dogState: "idle", // idle, walking, running, eating, drinking, sitting, fetching, begging
            stats: {
                joy: 80,
                hunger: 70,
                thirst: 60
            },
            // 小狗物理/運動數據
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

        // --- 初始化 Entry Point ---
        window.onload = function() {
            init3D();
            setupUIProgress();
            animate();
            
            // 監聽視窗縮放
            window.addEventListener('resize', onWindowResize, false);
            
            // 定時消耗小狗狀態
            setInterval(() => {
                gameState.stats.hunger = Math.max(0, gameState.stats.hunger - 1);
                gameState.stats.thirst = Math.max(0, gameState.stats.thirst - 1.5);
                gameState.stats.joy = Math.max(0, gameState.stats.joy - 0.8);
                updateStatsUI();
            }, 3000);
        };

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
                if (progress === 90) status.innerText = '正在設定物理反饋骨骼動畫...';
                if (progress >= 100) {
                    clearInterval(interval);
                    status.innerText = '啟動萌寵空間！';
                    setTimeout(() => {
                        loader.classList.add('opacity-0');
                        setTimeout(() => loader.style.display = 'none', 500);
                    }, 500);
                }
            }, 150);
        }

        // --- 3D 場景初始化 ---
        function init3D() {
            const container = document.getElementById('canvas-container');
            
            // 創建場景
            scene = new THREE.Scene();
            scene.fog = new THREE.FogExp2(0x87ceeb, 0.015);

            // 創建相機 (FOV, Aspect, Near, Far)
            camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 1000);
            resetCamera();

            // 創建渲染器
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            renderer.toneMapping = THREE.ACESFilmicToneMapping;
            renderer.toneMappingExposure = 1.0;
            container.appendChild(renderer.domElement);

            // 設置環境光源
            hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.6);
            hemiLight.position.set(0, 200, 0);
            scene.add(hemiLight);

            // 設置太陽直射光 (投射陰影)
            sunLight = new THREE.DirectionalLight(0xffffff, 1.2);
            sunLight.position.set(30, 40, 20);
            sunLight.castShadow = true;
            sunLight.shadow.camera.top = 25;
            sunLight.shadow.camera.bottom = -25;
            sunLight.shadow.camera.left = -25;
            sunLight.shadow.camera.right = 25;
            sunLight.shadow.camera.near = 0.1;
            sunLight.shadow.camera.far = 150;
            sunLight.shadow.mapSize.width = 2048;
            sunLight.shadow.mapSize.height = 2048;
            sunLight.shadow.bias = -0.0005;
            scene.add(sunLight);

            // 輔助環境光
            let ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
            scene.add(ambientLight);

            // 建立 3D 天空球
            createSky();

            // 建立 3D 綠色大草原 (含微地形與山丘)
            createTerrain();

            // 裝飾場景 (添加樹木、花草與圍籬)
            decorateEnvironment();

            // 核心：動態建立 3D 小狗模型
            createDogModel();

            // 建立滑鼠/觸控互動控制 (Camera Orbiting)
            setupInteractions();
        }

        // --- 天空建造 ---
        function createSky() {
            const skyGeo = new THREE.SphereGeometry(150, 32, 15);
            const skyMat = new THREE.MeshBasicMaterial({
                color: colors.sky.day,
                side: THREE.BackSide
            });
            skyMesh = new THREE.Mesh(skyGeo, skyMat);
            scene.add(skyMesh);
        }

        // --- 地形大草原建造 ---
        function createTerrain() {
            const size = 150;
            // 這裡使用 PlaneGeometry 創建一個大型平面
            const floorGeo = new THREE.PlaneGeometry(size, size, 40, 40);
            
            // 為平面加入一些「起伏不平」的高度，模擬真實草原山丘
            const pos = floorGeo.attributes.position;
            for (let i = 0; i < pos.count; i++) {
                const x = pos.getX(i);
                const y = pos.getY(i);
                // 離中心越遠起伏越大，中心保持平坦
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
                flatShading: true // 低多邊形風格更顯精緻
            });

            floor = new THREE.Mesh(floorGeo, floorMat);
            floor.rotation.x = -Math.PI / 2;
            floor.receiveShadow = true;
            scene.add(floor);
        }

        // --- 場景豐富度：裝飾樹木、小山與石頭 ---
        function decorateEnvironment() {
            // 樹木產生器
            const createTree = (x, z, scale) => {
                const treeGroup = new THREE.Group();
                treeGroup.position.set(x, getTerrainHeight(x, z), z);
                treeGroup.scale.set(scale, scale, scale);

                // 樹幹
                const trunkGeo = new THREE.CylinderGeometry(0.2, 0.4, 2, 5);
                const trunkMat = new THREE.MeshStandardMaterial({ color: 0x5c4033, roughness: 0.9 });
                const trunk = new THREE.Mesh(trunkGeo, trunkMat);
                trunk.position.y = 1;
                trunk.castShadow = true;
                trunk.receiveShadow = true;
                treeGroup.add(trunk);

                // 樹冠 (低多邊形雲朵狀)
                const leavesGeo = new THREE.DodecahedronGeometry(1.2, 1);
                const leavesMat = new THREE.MeshStandardMaterial({ color: 0x2e8b57, roughness: 0.6, flatShading: true });
                const leaves = new THREE.Mesh(leavesGeo, leavesMat);
                leaves.position.y = 2.2;
                leaves.castShadow = true;
                treeGroup.add(leaves);

                scene.add(treeGroup);
            };

            // 石頭產生器
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

            // 撒播樹木與石頭 (避開中央核心區域 15 單位內)
            for (let i = 0; i < 45; i++) {
                let r = 18 + Math.random() * 45;
                let theta = Math.random() * Math.PI * 2;
                let tx = Math.cos(theta) * r;
                let tz = Math.sin(theta) * r;
                createTree(tx, tz, 1.2 + Math.random() * 0.8);
            }

            for (let i = 0; i < 30; i++) {
                let r = 10 + Math.random() * 30;
                let theta = Math.random() * Math.PI * 2;
                let rx = Math.cos(theta) * r;
                let rz = Math.sin(theta) * r;
                createRock(rx, rz, 0.8 + Math.random() * 1.2);
            }
        }

        // 計算特定 X, Z 的地面高度 (匹配地形山丘)
        function getTerrainHeight(x, z) {
            const dist = Math.sqrt(x*x + z*z);
            if (dist <= 15) return 0; // 核心平坦活動區
            return Math.sin(x * 0.1) * Math.cos(z * 0.1) * 1.5 + Math.sin(x * 0.03) * 3.0;
        }

        // --- 核心：3D 組合式小狗模型建立 ---
        // 精雕細琢的低多邊形小狗，具備可控肢體：頭、身、四肢、耳朵、尾巴
        function createDogModel() {
            dog = new THREE.Group();
            dog.position.set(0, 0, 0);
            dog.castShadow = true;

            // 主要配色設定
            const breedColor = colors[gameState.breed];
            const primaryMat = new THREE.MeshStandardMaterial({ color: breedColor.primary, roughness: 0.8, flatShading: true });
            const secondaryMat = new THREE.MeshStandardMaterial({ color: breedColor.secondary, roughness: 0.8, flatShading: true });
            const blackMat = new THREE.MeshStandardMaterial({ color: breedColor.nose, roughness: 0.5 });
            const pinkMat = new THREE.MeshStandardMaterial({ color: 0xffb6c1, roughness: 0.6 });

            // 1. 身體 (Body)
            const bodyGeo = new THREE.BoxGeometry(0.9, 0.9, 1.6);
            const body = new THREE.Mesh(bodyGeo, primaryMat);
            body.position.y = 0.8;
            body.castShadow = true;
            body.receiveShadow = true;
            dog.add(body);
            dog.body = body; // 註冊引用

            // 肚子白色飾條
            const bellyGeo = new THREE.BoxGeometry(0.92, 0.5, 1.0);
            const belly = new THREE.Mesh(bellyGeo, secondaryMat);
            belly.position.set(0, -0.22, 0.1);
            body.add(belly);

            // 2. 頭部 (Head)
            const headGroup = new THREE.Group();
            headGroup.position.set(0, 0.6, 0.8); // 相對於身體的位置
            
            const headGeo = new THREE.BoxGeometry(0.8, 0.8, 0.8);
            const head = new THREE.Mesh(headGeo, primaryMat);
            head.castShadow = true;
            headGroup.add(head);
            headGroup.mainHead = head;

            // 臉部白毛 (柴犬特色)
            const faceWhiteGeo = new THREE.BoxGeometry(0.82, 0.4, 0.4);
            const faceWhite = new THREE.Mesh(faceWhiteGeo, secondaryMat);
            faceWhite.position.set(0, -0.2, 0.21);
            head.add(faceWhite);

            // 吻部/口鼻 (Muzzle)
            const muzzleGeo = new THREE.BoxGeometry(0.4, 0.3, 0.4);
            const muzzle = new THREE.Mesh(muzzleGeo, secondaryMat);
            muzzle.position.set(0, -0.1, 0.5);
            head.add(muzzle);

            // 鼻子
            const noseGeo = new THREE.BoxGeometry(0.18, 0.12, 0.12);
            const nose = new THREE.Mesh(noseGeo, blackMat);
            nose.position.set(0, 0.1, 0.22);
            muzzle.add(nose);

            // 眼睛
            const eyeGeo = new THREE.BoxGeometry(0.1, 0.1, 0.1);
            const leftEye = new THREE.Mesh(eyeGeo, blackMat);
            leftEye.position.set(0.28, 0.15, 0.38);
            const rightEye = leftEye.clone();
            rightEye.position.x = -0.28;
            head.add(leftEye);
            head.add(rightEye);

            // 眉毛 (白斑點)
            const browGeo = new THREE.BoxGeometry(0.15, 0.08, 0.05);
            const leftBrow = new THREE.Mesh(browGeo, secondaryMat);
            leftBrow.position.set(0.2, 0.32, 0.39);
            const rightBrow = leftBrow.clone();
            rightBrow.position.x = -0.2;
            head.add(leftBrow);
            head.add(rightBrow);

            // 舌頭 (預設隱藏，吃飯或哈哈笑時伸出)
            const tongueGeo = new THREE.BoxGeometry(0.16, 0.05, 0.25);
            const tongue = new THREE.Mesh(tongueGeo, pinkMat);
            tongue.position.set(0, -0.15, 0.35);
            tongue.rotation.x = 0.1;
            head.add(tongue);
            dog.tongue = tongue;
            tongue.visible = false;

            // 耳朵 (左與右)
            const earGeo = new THREE.ConeGeometry(0.25, 0.5, 4);
            earGeo.rotateX(Math.PI/2); // 調整圓錐方向
            
            const leftEar = new THREE.Mesh(earGeo, primaryMat);
            leftEar.position.set(0.32, 0.45, -0.1);
            leftEar.rotation.set(-0.1, 0, -0.2);
            leftEar.castShadow = true;
            
            // 耳朵內側粉色
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
            dog.head = headGroup; // 註冊引用

            // 3. 四肢 (Legs: LF, RF, LB, RB)
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

            // 前肢
            dog.legLF = createLeg(0.32, -0.3, 0.5);
            dog.legRF = createLeg(-0.32, -0.3, 0.5);
            // 後肢
            dog.legLB = createLeg(0.32, -0.3, -0.5);
            dog.legRB = createLeg(-0.32, -0.3, -0.5);

            body.add(dog.legLF);
            body.add(dog.legRF);
            body.add(dog.legLB);
            body.add(dog.legRB);

            // 4. 尾巴 (Tail)
            const tailGroup = new THREE.Group();
            tailGroup.position.set(0, 0.3, -0.85); // 屁股後方
            
            const tailGeo = new THREE.BoxGeometry(0.18, 0.18, 0.7);
            tailGeo.translate(0, 0, -0.3); // 調整旋轉中心到根部
            const tail = new THREE.Mesh(tailGeo, primaryMat);
            tail.castShadow = true;
            tail.rotation.x = 0.6; // 往上翹
            tailGroup.add(tail);

            // 尾巴尖白毛
            const tailTipGeo = new THREE.BoxGeometry(0.2, 0.2, 0.2);
            const tailTip = new THREE.Mesh(tailTipGeo, secondaryMat);
            tailTip.position.set(0, 0, -0.65);
            tail.add(tailTip);

            body.add(tailGroup);
            dog.tail = tailGroup; // 註冊引用

            // 5. 潮流飾物容器 (掛載在頭頂)
            dog.accessoryGroup = new THREE.Group();
            dog.accessoryGroup.position.set(0, 0.5, 0); // 頭頂上方
            head.add(dog.accessoryGroup);

            scene.add(dog);

            // 初步套用飾件
            updateDogAccessory();
        }

        // 當切換品種時，實時更新材質顏色
        function updateDogBreedColor() {
            if (!dog) return;
            const breedColor = colors[gameState.breed];
            
            // 遍歷所有 Mesh 更換對應顏色
            dog.traverse((child) => {
                if (child.isMesh) {
                    const colVal = child.material.color.getHex();
                    
                    // 根據原本顏色的分類匹配更換
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

        // --- 潮流配飾 3D 渲染 ---
        function updateDogAccessory() {
            if (!dog || !dog.accessoryGroup) return;

            // 清空舊配件
            while(dog.accessoryGroup.children.length > 0) { 
                dog.accessoryGroup.remove(dog.accessoryGroup.children[0]); 
            }

            if (gameState.accessory === 'none') return;

            if (gameState.accessory === 'party') {
                // 派對帽 (彩色圓錐)
                const hatGeo = new THREE.ConeGeometry(0.2, 0.5, 8);
                const hatMat = new THREE.MeshStandardMaterial({ color: 0xff00ff, roughness: 0.4 });
                const hat = new THREE.Mesh(hatGeo, hatMat);
                hat.rotation.x = -0.1;
                hat.position.set(0, 0.1, -0.05);
                hat.castShadow = true;

                // 頂部小球
                const pomGeo = new THREE.SphereGeometry(0.06, 8, 8);
                const pomMat = new THREE.MeshStandardMaterial({ color: 0xffff00 });
                const pom = new THREE.Mesh(pomGeo, pomMat);
                pom.position.y = 0.28;
                hat.add(pom);

                dog.accessoryGroup.add(hat);
            } 
            else if (gameState.accessory === 'crown') {
                // 黃金皇冠
                const crownGroup = new THREE.Group();
                crownGroup.position.set(0, 0.05, 0);

                const ringGeo = new THREE.CylinderGeometry(0.22, 0.22, 0.12, 12, 1, true);
                const goldMat = new THREE.MeshStandardMaterial({ color: 0xffd700, metalness: 0.9, roughness: 0.1 });
                const ring = new THREE.Mesh(ringGeo, goldMat);
                crownGroup.add(ring);

                // 皇冠尖角
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

        // --- 物理粒子特效系統 ---
        function createHeartParticle(pos) {
            const group = new THREE.Group();
            group.position.copy(pos);
            
            // 用幾何體拼一個心形，或者用簡單的紅球代替
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
                life: 1.0 // 壽命 (秒)
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

        // --- 動態相機操作控制 (支援拖拽旋轉、滾輪縮放) ---
        let isUserInteracting = false;
        let onPointerDownMouseX = 0, onPointerDownMouseY = 0;
        let lon = 0, onPointerDownLon = 0;
        let lat = 0, onPointerDownLat = 0;
        let cameraRadius = 10;

        function setupInteractions() {
            const dom = renderer.domElement;

            dom.addEventListener('pointerdown', (e) => {
                isUserInteracting = true;
                onPointerDownMouseX = e.clientX;
                onPointerDownMouseY = e.clientY;
                onPointerDownLon = lon;
                onPointerDownLat = lat;
                
                // 隱藏操作提示
                document.getElementById('gesture-hint').style.opacity = 0;
            });

            dom.addEventListener('pointermove', (e) => {
                if (isUserInteracting === true) {
                    lon = (onPointerDownMouseX - e.clientX) * 0.2 + onPointerDownLon;
                    lat = (e.clientY - onPointerDownMouseY) * 0.2 + onPointerDownLat;
                    lat = Math.max(-85, Math.min(85, lat)); // 限制仰角俯角
                }
            });

            document.addEventListener('pointerup', () => {
                isUserInteracting = false;
            });

            // 支援滑鼠滾輪縮放視角距離
            dom.addEventListener('wheel', (e) => {
                cameraRadius += e.deltaY * 0.005;
                cameraRadius = Math.max(4, Math.min(25, cameraRadius));
            });

            // 觸控拖拽 (行動裝置優化)
            dom.addEventListener('touchmove', (e) => {
                if (e.touches.length === 1 && isUserInteracting) {
                    const touch = e.touches[0];
                    lon = (onPointerDownMouseX - touch.clientX) * 0.25 + onPointerDownLon;
                    lat = (touch.clientY - onPointerDownMouseY) * 0.25 + onPointerDownLat;
                    lat = Math.max(-85, Math.min(85, lat));
                }
            }, { passive: true });
        }

        function resetCamera() {
            cameraRadius = 10;
            lon = 45;
            lat = 25;
        }

        // --- 遊戲邏輯與物理動畫循環 (Tick Loop) ---
        function animate() {
            requestAnimationFrame(animate);

            const delta = clock.getDelta();
            gameState.animTimer += delta;

            // 1. 小狗 AI & 位移邏輯
            updateDogAI(delta);

            // 2. 小狗肢體動畫 (Walk, Run, Wag tail, Beg)
            animateDogLimbs(delta);

            // 3. 玩具物理飛行與旋轉
            updateToysAndSupplies(delta);

            // 4. 粒子更新
            updateParticles(delta);

            // 5. 天空球自轉與光源微調
            if (skyMesh) skyMesh.rotation.y += 0.005 * delta;

            // 6. 相機跟隨與控制模式
            updateCameraPosition();

            // 執行渲染
            renderer.render(scene, camera);
        }

        // --- 小狗自主行為 AI 決策 ---
        function updateDogAI(delta) {
            if (!dog) return;

            // 判定目前是否需要撿玩具
            if (currentToy && gameState.dogState !== 'eating' && gameState.dogState !== 'drinking') {
                gameState.dogState = 'fetching';
                gameState.dogTargetPos.copy(currentToy.position);
            } 
            // 判定是否有骨頭
            else if (activeBones.length > 0 && gameState.dogState !== 'fetching' && gameState.dogState !== 'eating' && gameState.dogState !== 'drinking') {
                gameState.dogState = 'fetching';
                gameState.dogTargetPos.copy(activeBones[0].position);
            }
            // 判定是否有水盆
            else if (activeWaterBowls.length > 0 && gameState.dogState !== 'fetching' && gameState.dogState !== 'eating' && gameState.dogState !== 'drinking') {
                gameState.dogState = 'fetching';
                gameState.dogTargetPos.copy(activeWaterBowls[0].position);
            }
            // 沒有外在目標時，隨機漫步
            else if (gameState.dogState === 'idle' && Math.random() < 0.005) {
                gameState.dogState = 'walking';
                gameState.dogTargetPos.set(
                    (Math.random() - 0.5) * 20,
                    0,
                    (Math.random() - 0.5) * 20
                );
            }

            // 更新移動與旋轉物理
            let distance = gameState.dogPos.distanceTo(gameState.dogTargetPos);

            if (distance > 0.4) {
                // 轉向目標
                let targetAngle = Math.atan2(
                    gameState.dogTargetPos.x - gameState.dogPos.x,
                    gameState.dogTargetPos.z - gameState.dogPos.z
                );
                
                // 平滑旋轉過渡 (Slerp-like angle)
                let diff = targetAngle - gameState.dogRotationY;
                while (diff < -Math.PI) diff += Math.PI * 2;
                while (diff > Math.PI) diff -= Math.PI * 2;
                gameState.dogRotationY += diff * 0.1;
                dog.rotation.y = gameState.dogRotationY;

                // 奔跑與散步速度區分
                let maxSpeed = (gameState.dogState === 'fetching') ? 6.5 : 2.5;
                gameState.speed = THREE.MathUtils.lerp(gameState.speed, maxSpeed, 0.1);

                // 計算位移
                let moveStep = gameState.speed * delta;
                let dir = new THREE.Vector3().subVectors(gameState.dogTargetPos, gameState.dogPos).normalize();
                gameState.dogPos.addScaledVector(dir, moveStep);
                dog.position.copy(gameState.dogPos);

                // 產生踏地塵土特效
                if (Math.random() < 0.15) {
                    createDustParticle(gameState.dogPos);
                }

                // 標籤狀態
                document.getElementById('dog-state-tag').innerText = (gameState.dogState === 'fetching') ? "興奮奔跑中 🏃‍♂️" : "快樂漫步中 🐕";
                document.getElementById('dog-state-tag').className = "text-xs bg-amber-500/20 text-amber-300 px-2 py-0.5 rounded-full font-medium";
            } else {
                // 到達目標後的動作判定
                if (gameState.dogState === 'fetching') {
                    // 到達玩具/食物位置
                    if (currentToy) {
                        // 撿起玩具
                        gameState.dogState = 'begging'; // 搖尾巴慶祝
                        scene.remove(currentToy);
                        currentToy = null;
                        
                        // 心情大增
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
                
                if (gameState.dogState === 'idle') {
                    document.getElementById('dog-state-tag').innerText = "悠閒休息中 💤";
                    document.getElementById('dog-state-tag').className = "text-xs bg-green-500/20 text-green-400 px-2 py-0.5 rounded-full font-medium";
                } else if (gameState.dogState === 'eating') {
                    document.getElementById('dog-state-tag').innerText = "津津有味啃骨頭 🦴";
                    document.getElementById('dog-state-tag').className = "text-xs bg-pink-500/20 text-pink-400 px-2 py-0.5 rounded-full font-medium";
                } else if (gameState.dogState === 'drinking') {
                    document.getElementById('dog-state-tag').innerText = "大口喝水中 💧";
                    document.getElementById('dog-state-tag').className = "text-xs bg-blue-500/20 text-blue-400 px-2 py-0.5 rounded-full font-medium";
                } else if (gameState.dogState === 'begging') {
                    document.getElementById('dog-state-tag').innerText = "拿到玩具！討摸摸中 ✨";
                    document.getElementById('dog-state-tag').className = "text-xs bg-yellow-500/20 text-yellow-300 px-2 py-0.5 rounded-full font-medium";
                    
                    // 討摸摸一段時間後恢復
                    if (Math.random() < 0.005) gameState.dogState = 'idle';
                }
            }
        }

        // --- 肢體與尾巴物理動畫 (Procedural Animation) ---
        function animateDogLimbs(delta) {
            if (!dog) return;

            const t = gameState.animTimer;
            const speedFactor = gameState.speed;

            // 1. 舌頭露出狀態
            if (gameState.dogState === 'eating' || gameState.dogState === 'drinking' || gameState.dogState === 'begging') {
                dog.tongue.visible = true;
                dog.tongue.scale.y = 1 + Math.sin(t * 12) * 0.15;
            } else {
                dog.tongue.visible = false;
            }

            // 2. 尾巴搖擺 (依心情與狀態決定頻率與幅度)
            let tailWagSpeed = 8;
            let tailWagAmp = 0.2;

            if (gameState.dogState === 'fetching' || gameState.dogState === 'begging') {
                tailWagSpeed = 22; // 超興奮快搖
                tailWagAmp = 0.6;
            } else if (gameState.dogState === 'idle') {
                tailWagSpeed = 3;  // 慵懶慢搖
                tailWagAmp = 0.1;
            }

            dog.tail.rotation.y = Math.sin(t * tailWagSpeed) * tailWagAmp;

            // 3. 四肢奔跑/步行交替動畫 (行走與跑步的雙足相位差)
            if (speedFactor > 0.1) {
                // 行進間：前後肢成十字對角交叉擺動
                const walkCycle = t * speedFactor * 2.0;

                dog.legLF.rotation.x = Math.sin(walkCycle) * 0.6;
                dog.legRB.rotation.x = Math.sin(walkCycle) * 0.6;

                dog.legRF.rotation.x = -Math.sin(walkCycle) * 0.6;
                dog.legLB.rotation.x = -Math.sin(walkCycle) * 0.6;

                // 身體微小上下顛簸與前後擺動，增加軟Ｑ的物理感
                dog.body.position.y = 0.8 + Math.abs(Math.sin(walkCycle * 2)) * 0.15;
                dog.body.rotation.z = Math.sin(walkCycle) * 0.05;
                dog.head.rotation.x = -Math.abs(Math.cos(walkCycle)) * 0.1;
            } else {
                // 靜止或特殊動作
                if (gameState.dogState === 'begging') {
                    // 討摸摸：前肢抬起、後半身坐下
                    dog.legLF.rotation.x = -Math.PI / 3;
                    dog.legRF.rotation.x = -Math.PI / 3;
                    dog.legLB.rotation.x = -Math.PI / 2;
                    dog.legRB.rotation.x = -Math.PI / 2;
                    dog.body.position.y = 0.5 + Math.sin(t * 6) * 0.05;
                    dog.body.rotation.x = -Math.PI / 6;
                    dog.head.rotation.x = 0.2 + Math.sin(t * 4) * 0.05;
                } else if (gameState.dogState === 'eating' || gameState.dogState === 'drinking') {
                    // 啃咬/喝水：頭部向下快速擺動
                    dog.legLF.rotation.x = 0;
                    dog.legRF.rotation.x = 0;
                    dog.body.position.y = 0.7;
                    dog.body.rotation.x = 0.1;
                    dog.head.rotation.x = 0.6 + Math.abs(Math.sin(t * 10)) * 0.2;
                } else {
                    // 完全靜止站立
                    dog.legLF.rotation.x = 0;
                    dog.legRF.rotation.x = 0;
                    dog.legLB.rotation.x = 0;
                    dog.legRB.rotation.x = 0;
                    dog.body.position.y = 0.8;
                    dog.body.rotation.x = 0;
                    dog.body.rotation.z = 0;
                    dog.head.rotation.x = Math.sin(t * 1.5) * 0.04; // 呼吸微動
                }
            }

            // 耳朵隨風微動
            dog.leftEar.rotation.z = -0.2 + Math.sin(t * 5) * 0.03;
            dog.rightEar.rotation.z = 0.2 - Math.sin(t * 5) * 0.03;
        }

        // --- 玩具與消耗品物理拋物線更新 ---
        function updateToysAndSupplies(delta) {
            if (currentToy) {
                // 如果玩具還在空中飛行 (Y 座標 > 0)
                if (currentToy.position.y > 0.15) {
                    currentToy.velocity.y -= 9.8 * delta; // 重力
                    currentToy.position.addScaledVector(currentToy.velocity, delta);
                    
                    // 旋轉玩具
                    currentToy.rotation.x += 5 * delta;
                    currentToy.rotation.y += 5 * delta;
                } else {
                    currentToy.position.y = 0.15; // 落在地面上
                }
            }
        }

        // --- 粒子特效渲染刷新 ---
        function updateParticles(delta) {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= delta;
                
                // 套用速度
                p.mesh.position.addScaledVector(p.velocity, delta);
                
                if (p.type === 'heart') {
                    // 愛心一邊上升一邊放大然後淡出
                    p.mesh.scale.addScalar(delta * 0.8);
                } else if (p.type === 'dust') {
                    // 灰塵慢慢縮小
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

        // --- 相機跟隨與控制切換 ---
        function updateCameraPosition() {
            if (!dog) return;

            // 將極座標 (lon, lat) 轉換為三維笛卡爾直角座標
            const phi = THREE.MathUtils.degToRad(90 - lat);
            const theta = THREE.MathUtils.degToRad(lon);

            if (gameState.cameraMode === 'orbit') {
                // 上帝自由視角：圍繞世界原點(小狗當前位置附近)旋轉
                const targetX = dog.position.x + cameraRadius * Math.sin(phi) * Math.sin(theta);
                const targetY = dog.position.y + cameraRadius * Math.cos(phi) + 1.0;
                const targetZ = dog.position.z + cameraRadius * Math.sin(phi) * Math.cos(theta);

                camera.position.set(targetX, targetY, targetZ);
                camera.lookAt(dog.position.x, dog.position.y + 0.5, dog.position.z);
            } 
            else if (gameState.cameraMode === 'follow') {
                // 追隨視角：相機鎖定在小狗正後方
                const dogRot = dog.rotation.y;
                const offsetDistance = 6;
                const offsetHeight = 3;

                // 計算小狗正後方座標
                const targetCamX = dog.position.x - Math.sin(dogRot) * offsetDistance;
                const targetCamZ = dog.position.z - Math.cos(dogRot) * offsetDistance;
                const targetCamY = dog.position.y + offsetHeight;

                camera.position.lerp(new THREE.Vector3(targetCamX, targetCamY, targetCamZ), 0.08);
                camera.lookAt(dog.position.x, dog.position.y + 0.6, dog.position.z);
            } 
            else if (gameState.cameraMode === 'dog') {
                // 狗眼第一人稱：相機放置在小狗眼睛位置
                const headWorldPos = new THREE.Vector3();
                dog.head.mainHead.getWorldPosition(headWorldPos);

                camera.position.copy(headWorldPos);
                // 稍微向前看
                const lookDir = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
                const lookTarget = new THREE.Vector3().addVectors(headWorldPos, lookDir);
                camera.lookAt(lookTarget);
            }
        }

        // --- 互動功能與動作處發 ---
        function triggerAction(action) {
            if (!dog) return;

            // 口哨召回 (強行將目標點設為玩家位置，此處默認原點)
            if (action === 'whistle') {
                gameState.dogTargetPos.set(0, 0, 0);
                gameState.dogState = 'fetching';
                playBark();
                createCelebrationHearts(1);
                
                // 心情微升
                gameState.stats.joy = Math.min(100, gameState.stats.joy + 5);
                updateStatsUI();
                return;
            }

            // 投擲小球
            if (action === 'throw_ball') {
                if (currentToy) scene.remove(currentToy);
                
                const ballGeo = new THREE.SphereGeometry(0.2, 16, 16);
                const ballMat = new THREE.MeshStandardMaterial({ color: 0x39ff14, roughness: 0.3 });
                const ball = new THREE.Mesh(ballGeo, ballMat);
                
                // 從相機位置朝小狗前方扔出
                ball.position.set(dog.position.x + (Math.random() - 0.5) * 5, 5, dog.position.z + 8 + Math.random() * 5);
                ball.velocity = new THREE.Vector3(0, 2, 0); // 給予一點向上的初速
                ball.castShadow = true;
                
                scene.add(ball);
                currentToy = ball;
                return;
            }

            // 發射飛盤
            if (action === 'throw_frisbee') {
                if (currentToy) scene.remove(currentToy);

                const frisbeeGeo = new THREE.CylinderGeometry(0.4, 0.4, 0.06, 16);
                const frisbeeMat = new THREE.MeshStandardMaterial({ color: 0xbf00ff, roughness: 0.5 });
                const frisbee = new THREE.Mesh(frisbeeGeo, frisbeeMat);

                frisbee.position.set(dog.position.x + (Math.random() - 0.5) * 8, 4, dog.position.z + 10 + Math.random() * 4);
                frisbee.velocity = new THREE.Vector3(0, 1.2, 0);
                frisbee.castShadow = true;

                scene.add(frisbee);
                currentToy = frisbee;
                return;
            }

            // 餵食骨頭 (直接降落在小狗前方)
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

                // 計算落點 (小狗前方 3 單位)
                const dir = new THREE.Vector3(0, 0, 1).applyAxisAngle(new THREE.Vector3(0, 1, 0), dog.rotation.y);
                const targetPos = new THREE.Vector3().addVectors(dog.position, dir.multiplyScalar(3));
                targetPos.y = 0.1;

                boneGroup.position.copy(targetPos);
                scene.add(boneGroup);
                activeBones.push(boneGroup);
                return;
            }

            // 餵水 (生成藍色水盆)
            if (action === 'water') {
                const bowlGroup = new THREE.Group();
                const bowlMat = new THREE.MeshStandardMaterial({ color: 0xff4444, roughness: 0.5 });
                
                const outerGeo = new THREE.CylinderGeometry(0.4, 0.3, 0.25, 12);
                const outer = new THREE.Mesh(outerGeo, bowlMat);
                bowlGroup.add(outer);

                // 水面
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

            // 摸摸頭 (直接愛心爆發)
            if (action === 'pet') {
                gameState.stats.joy = Math.min(100, gameState.stats.joy + 15);
                updateStatsUI();
                createCelebrationHearts(5);
                playBark();
            }
        }

        // --- 簡易網頁音效生成器 (不依賴外部音檔，純 Web Audio API 實作寫實狗吠) ---
        function playBark() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                
                // 雙聲道重疊創造厚實感
                for(let i=0; i<2; i++) {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    
                    osc.type = 'triangle';
                    // 狗吠的特色：由高頻快速下降至低頻 (Pitch Sweep)
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
                console.log("AudioContext not supported or blocked");
            }
        }

        // --- 介面 UI 更新功能 ---
        function updateStatsUI() {
            document.getElementById('stat-joy-val').innerText = Math.round(gameState.stats.joy) + '%';
            document.getElementById('stat-joy-bar').style.width = gameState.stats.joy + '%';
            
            document.getElementById('stat-hunger-val').innerText = Math.round(gameState.stats.hunger) + '%';
            document.getElementById('stat-hunger-bar').style.width = gameState.stats.hunger + '%';
            
            document.getElementById('stat-thirst-val').innerText = Math.round(gameState.stats.thirst) + '%';
            document.getElementById('stat-thirst-bar').style.width = gameState.stats.thirst + '%';
        }

        function setCameraMode(mode) {
            gameState.cameraMode = mode;
            
            // 重置按鈕樣式
            document.getElementById('cam-orbit-btn').className = "p-2.5 rounded-lg text-slate-300 hover:bg-slate-800 text-sm font-medium flex items-center gap-2 transition-all";
            document.getElementById('cam-follow-btn').className = "p-2.5 rounded-lg text-slate-300 hover:bg-slate-800 text-sm font-medium flex items-center gap-2 transition-all";
            document.getElementById('cam-dog-btn').className = "p-2.5 rounded-lg text-slate-300 hover:bg-slate-800 text-sm font-medium flex items-center gap-2 transition-all";

            if (mode === 'orbit') {
                document.getElementById('cam-orbit-btn').className = "p-2.5 rounded-lg bg-amber-500 text-white text-sm font-medium flex items-center gap-2 transition-all";
                resetCamera();
            } else if (mode === 'follow') {
                document.getElementById('cam-follow-btn').className = "p-2.5 rounded-lg bg-amber-500 text-white text-sm font-medium flex items-center gap-2 transition-all";
            } else if (mode === 'dog') {
                document.getElementById('cam-dog-btn').className = "p-2.5 rounded-lg bg-amber-500 text-white text-sm font-medium flex items-center gap-2 transition-all";
            }
        }

        function toggleCustomizer() {
            const panel = document.getElementById('customizer-panel');
            panel.classList.toggle('translate-x-full');
        }

        function changeDogBreed(breed) {
            // 重置所有選取外觀
            document.getElementById('breed-shiba').className = "p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750";
            document.getElementById('breed-golden').className = "p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750";
            document.getElementById('breed-husky').className = "p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750";
            document.getElementById('breed-dalmatian').className = "p-3 rounded-xl border-2 border-transparent bg-slate-800 text-left transition-all hover:bg-slate-750";

            document.getElementById('breed-' + breed).className = "p-3 rounded-xl border-2 border-amber-500 bg-amber-500/10 text-left transition-all";
            
            gameState.breed = breed;
            updateDogBreedColor();
            createCelebrationHearts(3);
        }

        function changeAccessory(acc) {
            document.getElementById('acc-none').className = "p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-750";
            document.getElementById('acc-party').className = "p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-750";
            document.getElementById('acc-crown').className = "p-2.5 rounded-lg border-2 border-transparent bg-slate-800 text-slate-300 hover:bg-slate-750";

            document.getElementById('acc-' + acc).className = "p-2.5 rounded-lg border-2 border-amber-500 bg-amber-500/10 text-amber-400 font-bold";
            
            gameState.accessory = acc;
            updateDogAccessory();
            createCelebrationHearts(2);
        }

        function changeTimeOfDay(time) {
            document.getElementById('time-day').className = "p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700";
            document.getElementById('time-sunset').className = "p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700";
            document.getElementById('time-night').className = "p-2.5 rounded-lg border border-slate-700 bg-slate-800 text-slate-300 hover:bg-slate-700";

            document.getElementById('time-' + time).className = "p-2.5 rounded-lg border border-amber-500/50 bg-amber-500/10 text-amber-400 font-bold";

            gameState.timeOfDay = time;

            // 改變大自然色調、霧氣與光照
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
                sunLight.color.setHex(0xffaa44); // 暖橘光
                sunLight.intensity = 1.0;
                hemiLight.color.setHex(0xffaa66);
                renderer.setClearColor(colors.sky.sunset);
            } 
            else if (time === 'night') {
                scene.fog.color.setHex(colors.sky.night);
                skyMesh.material.color.setHex(colors.sky.night);
                floor.material.color.setHex(colors.grass.night);
                sunLight.color.setHex(0x5588ff); // 冷月光
                sunLight.intensity = 0.3;
                hemiLight.color.setHex(0x112244);
                renderer.setClearColor(colors.sky.night);
            }
        }

        // 命名模態框控制
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
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
    </script>
</body>
</html>
