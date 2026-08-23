<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>一年级数学平面对垒游戏</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=ZCOOL+KuaiLe&display=swap" rel="stylesheet">
    
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #1a202c; /* Dark background to make game pop */
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'ZCOOL KuaiLe', cursive, sans-serif;
            overflow: hidden;
            touch-action: none; /* Prevent pull-to-refresh on mobile */
        }

        /* Enforce 16:9 aspect ratio */
        #game-wrapper {
            position: relative;
            width: 100vw;
            height: 56.25vw; /* 100 * 9 / 16 */
            max-height: 100vh;
            max-width: 177.78vh; /* 100 * 16 / 9 */
            background-color: white;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
            overflow: hidden;
            border-radius: 12px;
        }

        /* Common Screen Styles */
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: none;
            flex-direction: column;
        }
        .screen.active {
            display: flex;
        }

        /* Animations */
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.05); }
        }
        .animate-pulse-fast { animation: pulse 0.5s infinite; }
        
        @keyframes popIn {
            0% { transform: scale(0.5); opacity: 0; }
            80% { transform: scale(1.1); opacity: 1; }
            100% { transform: scale(1); opacity: 1; }
        }
        .pop-in { animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards; }

        @keyframes floatUp {
            0% { transform: translateY(20px); opacity: 0; }
            10% { transform: translateY(0); opacity: 1; }
            90% { transform: translateY(-50px); opacity: 1; }
            100% { transform: translateY(-70px); opacity: 0; }
        }
        .float-msg {
            position: absolute;
            font-size: 3rem;
            font-weight: bold;
            pointer-events: none;
            z-index: 50;
            animation: floatUp 1.5s ease-out forwards;
            text-shadow: 2px 2px 0 #fff, -2px -2px 0 #fff, 2px -2px 0 #fff, -2px 2px 0 #fff, 0 4px 6px rgba(0,0,0,0.3);
        }

        /* Freeze overlay */
        .freeze-overlay {
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(173, 216, 230, 0.6);
            backdrop-filter: blur(4px);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 40;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.2s;
        }
        .frozen .freeze-overlay {
            opacity: 1;
            pointer-events: all;
        }
        
        /* Shape Buttons */
        .shape-btn {
            transition: all 0.1s;
            filter: drop-shadow(0 4px 6px rgba(0,0,0,0.2));
        }
        .shape-btn:active {
            transform: scale(0.95) translateY(4px);
            filter: drop-shadow(0 1px 2px rgba(0,0,0,0.2));
        }
        
        /* Custom Scrollbar for Setup */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #f1f1f1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
    </style>
</head>
<body>

    <div id="game-wrapper">
        
        <!-- Screen 1: Setup -->
        <div id="screen-setup" class="screen active bg-indigo-50 p-6">
            <h1 class="text-4xl text-center text-indigo-800 font-bold mb-6 drop-shadow-md">✨ 欢乐平面图形大对垒 ✨</h1>
            
            <div class="flex-1 flex flex-col md:flex-row gap-6 bg-white p-6 rounded-2xl shadow-inner overflow-hidden">
                <!-- Total Students -->
                <div class="w-full md:w-1/3 flex flex-col items-center justify-center border-b md:border-b-0 md:border-r border-gray-200 pb-4 md:pb-0">
                    <label class="text-2xl text-gray-700 mb-4">每组总人数:</label>
                    <input type="number" id="input-total" value="38" min="2" max="100" class="w-32 text-center text-4xl p-2 border-4 border-indigo-200 rounded-xl focus:outline-none focus:border-indigo-500 font-bold text-indigo-700">
                    <p class="text-gray-400 mt-2 text-sm">红组和蓝组人数相同</p>
                </div>
                
                <!-- Absent Selection -->
                <div class="w-full md:w-2/3 flex gap-4 h-full">
                    <div class="flex-1 flex flex-col">
                        <h3 class="text-xl text-red-600 font-bold text-center mb-2">红组缺席号码</h3>
                        <div class="flex-1 overflow-y-auto bg-red-50 rounded-lg p-2 border-2 border-red-100" id="red-absent-container">
                            <!-- Populated by JS -->
                        </div>
                    </div>
                    <div class="flex-1 flex flex-col">
                        <h3 class="text-xl text-blue-600 font-bold text-center mb-2">蓝组缺席号码</h3>
                        <div class="flex-1 overflow-y-auto bg-blue-50 rounded-lg p-2 border-2 border-blue-100" id="blue-absent-container">
                            <!-- Populated by JS -->
                        </div>
                    </div>
                </div>
            </div>
            
            <div class="text-center mt-6">
                <button id="btn-save-setup" class="bg-gradient-to-r from-green-400 to-green-600 text-white text-3xl py-4 px-12 rounded-full font-bold shadow-[0_6px_0_#166534] active:shadow-[0_0px_0_#166534] active:translate-y-[6px] hover:from-green-500 hover:to-green-700 transition-all">
                    准备开始！
                </button>
            </div>
        </div>

        <!-- Screen 2: Drawing Lots -->
        <div id="screen-draw" class="screen bg-gradient-to-b from-purple-600 to-indigo-900 flex justify-center items-center">
            <h2 class="text-5xl text-white font-bold absolute top-10 drop-shadow-lg">抽取对垒选手</h2>
            
            <div class="flex w-4/5 justify-between items-center h-3/5">
                <!-- Red Draw Box -->
                <div class="w-[45%] h-full bg-white rounded-3xl border-8 border-red-500 flex flex-col items-center justify-center relative shadow-[0_0_30px_rgba(239,68,68,0.5)]">
                    <div class="absolute -top-6 bg-red-500 text-white px-8 py-2 rounded-full text-2xl font-bold shadow-md">红组</div>
                    <div id="draw-red-num" class="text-[8rem] font-bold text-red-600 leading-none">?</div>
                </div>
                
                <div class="text-6xl text-yellow-300 font-bold italic animate-pulse-fast">VS</div>
                
                <!-- Blue Draw Box -->
                <div class="w-[45%] h-full bg-white rounded-3xl border-8 border-blue-500 flex flex-col items-center justify-center relative shadow-[0_0_30px_rgba(59,130,246,0.5)]">
                    <div class="absolute -top-6 bg-blue-500 text-white px-8 py-2 rounded-full text-2xl font-bold shadow-md">蓝组</div>
                    <div id="draw-blue-num" class="text-[8rem] font-bold text-blue-600 leading-none">?</div>
                </div>
            </div>
            
            <button id="btn-draw-action" class="absolute bottom-10 bg-yellow-400 text-red-800 text-4xl py-4 px-16 rounded-full font-bold shadow-[0_8px_0_#b45309] active:shadow-[0_0px_0_#b45309] active:translate-y-[8px] hover:bg-yellow-300 transition-all">
                开始抽签
            </button>
        </div>

        <!-- Screen 3: Waiting for Players to step up -->
        <div id="screen-wait" class="screen bg-gray-900 bg-opacity-90 flex justify-center items-center z-50">
            <div class="bg-white p-12 rounded-3xl flex flex-col items-center shadow-[0_0_50px_rgba(255,255,255,0.2)] text-center max-w-3xl">
                <h2 class="text-5xl font-bold text-gray-800 mb-8">请选手上台！</h2>
                <div class="flex items-center justify-center gap-8 text-6xl font-bold mb-12">
                    <span id="wait-red" class="text-red-500">红 XX</span>
                    <span class="text-gray-400 text-4xl">VS</span>
                    <span id="wait-blue" class="text-blue-500">蓝 XX</span>
                </div>
                <button id="btn-start-match" class="bg-gradient-to-r from-red-500 via-yellow-500 to-blue-500 text-white text-5xl py-6 px-20 rounded-full font-bold shadow-[0_10px_0_#4b5563] active:shadow-[0_0px_0_#4b5563] active:translate-y-[10px] hover:brightness-110 transition-all animate-pulse-fast">
                    开始比赛 !
                </button>
            </div>
        </div>

        <!-- Screen 4: The Game Board -->
        <div id="screen-game" class="screen flex-row relative bg-gray-200">
            <!-- Header (Scores & Timer & Question Info) -->
            <div class="absolute top-0 left-0 w-full h-24 bg-white shadow-md z-30 flex justify-between items-center px-8 rounded-b-3xl">
                <!-- Red Score -->
                <div class="flex items-center gap-4">
                    <div class="bg-red-500 text-white w-16 h-16 rounded-full flex items-center justify-center text-3xl font-bold border-4 border-red-200 shadow-inner">红</div>
                    <div class="flex flex-col">
                        <span class="text-gray-500 text-sm font-bold uppercase tracking-wider">Score</span>
                        <span id="score-red" class="text-4xl font-bold text-red-600 leading-none">0</span>
                    </div>
                </div>
                
                <!-- Center Info -->
                <div class="flex flex-col items-center">
                    <div class="text-xl text-gray-600 font-bold bg-gray-100 px-6 py-1 rounded-full mb-1 border-2 border-gray-200 shadow-inner">
                        第 <span id="q-current" class="text-indigo-600 text-2xl">1</span> / 3 题
                    </div>
                    <div class="relative w-24 h-24 -mt-4 bg-white rounded-full flex items-center justify-center border-8 border-indigo-100 shadow-lg z-40">
                        <span id="timer-display" class="text-5xl font-bold text-indigo-600">10</span>
                    </div>
                </div>

                <!-- Blue Score -->
                <div class="flex items-center gap-4 text-right flex-row-reverse">
                    <div class="bg-blue-500 text-white w-16 h-16 rounded-full flex items-center justify-center text-3xl font-bold border-4 border-blue-200 shadow-inner">蓝</div>
                    <div class="flex flex-col items-end">
                        <span class="text-gray-500 text-sm font-bold uppercase tracking-wider">Score</span>
                        <span id="score-blue" class="text-4xl font-bold text-blue-600 leading-none">0</span>
                    </div>
                </div>
            </div>

            <!-- Central Target Instruction -->
            <div class="absolute top-28 left-1/2 transform -translate-x-1/2 z-30 bg-yellow-300 px-8 py-3 rounded-full border-4 border-yellow-500 shadow-lg flex items-center gap-4">
                <span class="text-3xl text-yellow-900 font-bold">请找出：</span>
                <span id="target-shape-name" class="text-5xl text-red-600 font-black drop-shadow-sm pop-in">圆形</span>
            </div>

            <!-- Left: Red Team Area -->
            <div id="area-red" class="w-1/2 h-full bg-red-50 pt-48 pb-10 px-10 relative flex items-center justify-center border-r-4 border-gray-300">
                <div class="absolute top-32 left-10 text-2xl font-bold text-red-400 bg-red-100 px-4 py-1 rounded-full">选手: <span id="player-red-num">1</span></div>
                
                <!-- Shape Grid -->
                <div id="grid-red" class="grid grid-cols-2 gap-8 w-full max-w-[400px]">
                    <!-- Buttons injected by JS -->
                </div>

                <!-- Freeze Overlay -->
                <div class="freeze-overlay flex-col text-blue-800">
                    <svg class="w-24 h-24 mb-4 animate-spin" style="animation-duration: 3s;" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 2a1 1 0 011 1v1.326l.732.732a1 1 0 01-1.414 1.414l-.024-.024H8.706l-.024.024a1 1 0 01-1.414-1.414l.732-.732V3a1 1 0 011-1zm-3.707 5.293a1 1 0 011.414 0L8 7.586v2.12l-2.732 2.733a1 1 0 11-1.414-1.415l1.024-1.024H3a1 1 0 110-2h1.878l-1.024-1.024a1 1 0 010-1.414zM10 12a1 1 0 011 1v1.268l1.707-1.707a1 1 0 011.414 1.414l-2.414 2.414A1 1 0 0110 17a1 1 0 01-.707-.293l-2.414-2.414a1 1 0 111.414-1.414L10 14.268V13a1 1 0 011-1zm3.707-4.707a1 1 0 010 1.414l-1.024 1.024H17a1 1 0 110 2h-1.878l1.024 1.024a1 1 0 01-1.414 1.414l-2.732-2.732V10.41l.293-.293a1 1 0 011.414-1.414z" clip-rule="evenodd"></path></svg>
                    <span class="text-4xl font-black drop-shadow-md">被冰冻了! 3s</span>
                </div>
                
                <div id="popup-area-red" class="absolute inset-0 pointer-events-none flex items-center justify-center"></div>
            </div>

            <!-- Right: Blue Team Area -->
            <div id="area-blue" class="w-1/2 h-full bg-blue-50 pt-48 pb-10 px-10 relative flex items-center justify-center">
                <div class="absolute top-32 right-10 text-2xl font-bold text-blue-400 bg-blue-100 px-4 py-1 rounded-full">选手: <span id="player-blue-num">1</span></div>
                
                <!-- Shape Grid -->
                <div id="grid-blue" class="grid grid-cols-2 gap-8 w-full max-w-[400px]">
                    <!-- Buttons injected by JS -->
                </div>

                <!-- Freeze Overlay -->
                <div class="freeze-overlay flex-col text-blue-800">
                     <svg class="w-24 h-24 mb-4 animate-spin" style="animation-duration: 3s;" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 2a1 1 0 011 1v1.326l.732.732a1 1 0 01-1.414 1.414l-.024-.024H8.706l-.024.024a1 1 0 01-1.414-1.414l.732-.732V3a1 1 0 011-1zm-3.707 5.293a1 1 0 011.414 0L8 7.586v2.12l-2.732 2.733a1 1 0 11-1.414-1.415l1.024-1.024H3a1 1 0 110-2h1.878l-1.024-1.024a1 1 0 010-1.414zM10 12a1 1 0 011 1v1.268l1.707-1.707a1 1 0 011.414 1.414l-2.414 2.414A1 1 0 0110 17a1 1 0 01-.707-.293l-2.414-2.414a1 1 0 111.414-1.414L10 14.268V13a1 1 0 011-1zm3.707-4.707a1 1 0 010 1.414l-1.024 1.024H17a1 1 0 110 2h-1.878l1.024 1.024a1 1 0 01-1.414 1.414l-2.732-2.732V10.41l.293-.293a1 1 0 011.414-1.414z" clip-rule="evenodd"></path></svg>
                    <span class="text-4xl font-black drop-shadow-md">被冰冻了! 3s</span>
                </div>
                
                <div id="popup-area-blue" class="absolute inset-0 pointer-events-none flex items-center justify-center"></div>
            </div>
            
            <!-- Midline decorative -->
            <div class="absolute top-24 bottom-0 left-1/2 w-1 bg-gray-300 transform -translate-x-1/2 z-10 opacity-50 border-dashed border-l-4 border-gray-300 bg-transparent"></div>
        </div>

        <!-- Screen 5: Match Over Summary -->
        <div id="screen-summary" class="screen bg-indigo-900 bg-opacity-95 flex flex-col justify-center items-center z-50">
            <h1 class="text-6xl text-yellow-300 font-bold mb-12 drop-shadow-[0_5px_5px_rgba(0,0,0,0.8)] pop-in">本轮比赛结束！</h1>
            
            <div class="flex gap-16 mb-16 w-3/4 justify-center">
                <div class="bg-red-500 p-8 rounded-3xl flex flex-col items-center shadow-[0_0_40px_rgba(239,68,68,0.6)] w-64 transform -rotate-3 border-8 border-white">
                    <span class="text-white text-3xl font-bold mb-4">红组总分</span>
                    <span id="summary-score-red" class="text-7xl font-black text-white">0</span>
                </div>
                
                <div class="bg-blue-500 p-8 rounded-3xl flex flex-col items-center shadow-[0_0_40px_rgba(59,130,246,0.6)] w-64 transform rotate-3 border-8 border-white">
                    <span class="text-white text-3xl font-bold mb-4">蓝组总分</span>
                    <span id="summary-score-blue" class="text-7xl font-black text-white">0</span>
                </div>
            </div>
            
            <button id="btn-next-round" class="bg-gradient-to-b from-green-400 to-green-600 text-white text-4xl py-5 px-16 rounded-full font-bold shadow-[0_8px_0_#166534] active:shadow-[0_0px_0_#166534] active:translate-y-[8px] hover:brightness-110 transition-all">
                抽取下一组选手
            </button>
        </div>

    </div>

    <script>
        // Game State Variables
        let totalStudents = 38;
        let absentRed = new Set();
        let absentBlue = new Set();
        let availableRed = [];
        let availableBlue = [];
        
        let currentRed = null;
        let currentBlue = null;
        
        let scoreRed = 0;
        let scoreBlue = 0;
        
        let questionCount = 0;
        const MAX_QUESTIONS = 3;
        
        let timeLeft = 10;
        let timerInterval = null;
        let drawInterval = null;
        
        let questionAnswered = false;
        let currentTargetShape = null;

        // Audio synths (initialized on user interaction)
        let bgmSynth = null;
        let bgmPart = null;
        let sfxDrawTick = null;
        let sfxDrawWin = null;
        let sfxCorrect = null;
        let sfxWrong = null;
        let sfxWin = null;
        let sfxFreeze = null;
        let audioInitialized = false;

        // Shape Data Definitions (Using inline SVGs for crisp rendering)
        const SHAPES = [
            { id: 'circle', name: '圆形', color: '#ef4444', 
              svg: '<svg viewBox="0 0 100 100" class="w-full h-full drop-shadow-md"><circle cx="50" cy="50" r="45" fill="currentColor"/></svg>' },
            { id: 'square', name: '正方形', color: '#3b82f6', 
              svg: '<svg viewBox="0 0 100 100" class="w-full h-full drop-shadow-md"><rect x="10" y="10" width="80" height="80" rx="8" fill="currentColor"/></svg>' },
            { id: 'triangle', name: '三角形', color: '#eab308', 
              svg: '<svg viewBox="0 0 100 100" class="w-full h-full drop-shadow-md"><polygon points="50,10 90,90 10,90" fill="currentColor" stroke-linejoin="round" stroke-width="5" stroke="currentColor"/></svg>' },
            { id: 'rectangle', name: '长方形', color: '#22c55e', 
              svg: '<svg viewBox="0 0 100 100" class="w-full h-full drop-shadow-md"><rect x="10" y="25" width="80" height="50" rx="6" fill="currentColor"/></svg>' }
        ];

        // DOM Elements
        const screens = {
            setup: document.getElementById('screen-setup'),
            draw: document.getElementById('screen-draw'),
            wait: document.getElementById('screen-wait'),
            game: document.getElementById('screen-game'),
            summary: document.getElementById('screen-summary')
        };

        function initAudio() {
            if (audioInitialized) return;
            
            // Start Audio Context
            Tone.start();

            // Background Music (Cheerful Arpeggio Loop)
            bgmSynth = new Tone.PolySynth(Tone.Synth, {
                oscillator: { type: "triangle" },
                envelope: { attack: 0.1, decay: 0.2, sustain: 0.2, release: 1 }
            }).toDestination();
            bgmSynth.volume.value = -15;

            const chords = [
                ['C4', 'E4', 'G4'], ['F4', 'A4', 'C5'], ['G4', 'B4', 'D5'], ['C4', 'E4', 'G4']
            ];
            let chordIdx = 0;
            bgmPart = new Tone.Loop(time => {
                bgmSynth.triggerAttackRelease(chords[chordIdx], "8n", time);
                chordIdx = (chordIdx + 1) % chords.length;
            }, "4n");
            Tone.Transport.bpm.value = 120;

            // SFX setup
            sfxDrawTick = new Tone.MembraneSynth().toDestination();
            sfxDrawTick.volume.value = -10;
            
            sfxDrawWin = new Tone.Synth({
                oscillator: { type: 'sine' }, envelope: { attack: 0.05, decay: 0.2, sustain: 0.5, release: 2 }
            }).toDestination();
            
            sfxCorrect = new Tone.PolySynth(Tone.Synth).toDestination();
            sfxCorrect.volume.value = -5;
            
            sfxWrong = new Tone.FMSynth({
                modulationIndex: 12.22,
                envelope: { attack: 0.01, decay: 0.2 },
                modulation: { type: "square" },
                modulationEnvelope: { attack: 0.2, decay: 0.01 }
            }).toDestination();
            sfxWrong.volume.value = -5;

            sfxWin = new Tone.PolySynth(Tone.Synth).toDestination();
            
            // Noise synth for freeze effect
            sfxFreeze = new Tone.NoiseSynth({
                noise: { type: 'white' }, envelope: { attack: 0.05, decay: 0.5, sustain: 0, release: 0 }
            }).toDestination();
            sfxFreeze.volume.value = -10;

            audioInitialized = true;
        }

        function playBGM() {
            if (bgmPart && Tone.Transport.state !== 'started') {
                Tone.Transport.start();
                bgmPart.start(0);
            }
        }
        function stopBGM() {
            if (bgmPart) {
                bgmPart.stop();
                Tone.Transport.stop();
            }
        }

        function renderAbsentCheckboxes() {
            totalStudents = parseInt(document.getElementById('input-total').value) || 38;
            const redContainer = document.getElementById('red-absent-container');
            const blueContainer = document.getElementById('blue-absent-container');
            
            redContainer.innerHTML = '';
            blueContainer.innerHTML = '';
            
            // Grid for checkboxes
            const gridClass = "grid grid-cols-5 md:grid-cols-7 gap-2";
            const redGrid = document.createElement('div'); redGrid.className = gridClass;
            const blueGrid = document.createElement('div'); blueGrid.className = gridClass;

            for (let i = 1; i <= totalStudents; i++) {
                // Red
                const redLabel = document.createElement('label');
                redLabel.className = "flex items-center gap-1 cursor-pointer bg-white rounded border border-red-200 p-1 hover:bg-red-100 transition";
                redLabel.innerHTML = `<input type="checkbox" class="w-4 h-4 accent-red-500" value="${i}" onchange="toggleAbsent('red', ${i}, this.checked)"> <span class="text-sm font-bold text-gray-700">${i}</span>`;
                if(absentRed.has(i)) redLabel.querySelector('input').checked = true;
                redGrid.appendChild(redLabel);

                // Blue
                const blueLabel = document.createElement('label');
                blueLabel.className = "flex items-center gap-1 cursor-pointer bg-white rounded border border-blue-200 p-1 hover:bg-blue-100 transition";
                blueLabel.innerHTML = `<input type="checkbox" class="w-4 h-4 accent-blue-500" value="${i}" onchange="toggleAbsent('blue', ${i}, this.checked)"> <span class="text-sm font-bold text-gray-700">${i}</span>`;
                if(absentBlue.has(i)) blueLabel.querySelector('input').checked = true;
                blueGrid.appendChild(blueLabel);
            }
            
            redContainer.appendChild(redGrid);
            blueContainer.appendChild(blueGrid);
        }

        window.toggleAbsent = (team, num, isChecked) => {
            const set = team === 'red' ? absentRed : absentBlue;
            if (isChecked) set.add(num);
            else set.delete(num);
        };

        document.getElementById('input-total').addEventListener('change', () => {
            // Clear sets if total reduces below selected numbers, but simple re-render is fine for now
            renderAbsentCheckboxes();
        });

        // Initialize setup screen
        renderAbsentCheckboxes();

        function switchScreen(screenId) {
            Object.values(screens).forEach(s => s.classList.remove('active'));
            screens[screenId].classList.add('active');
        }

        document.getElementById('btn-save-setup').addEventListener('click', async () => {
            await initAudio();
            playBGM();
            
            // Build available arrays
            availableRed = [];
            availableBlue = [];
            for (let i = 1; i <= totalStudents; i++) {
                if (!absentRed.has(i)) availableRed.push(i);
                if (!absentBlue.has(i)) availableBlue.push(i);
            }

            if (availableRed.length === 0 || availableBlue.length === 0) {
                alert("人数不足，请减少缺席人数！");
                return;
            }
            startDrawingLots();
        });

        function startDrawingLots() {
            switchScreen('draw');
            
            if (availableRed.length === 0 || availableBlue.length === 0) {
                 // Game over, everyone has played.
                 alert("所有选手都已经上场过了！游戏结束！");
                 switchScreen('setup'); // Back to start
                 return;
            }

            document.getElementById('draw-red-num').textContent = '?';
            document.getElementById('draw-blue-num').textContent = '?';
            const btn = document.getElementById('btn-draw-action');
            btn.style.display = 'block';
            btn.textContent = '开始抽签';
            
            btn.onclick = () => {
                btn.style.display = 'none';
                animateDrawing();
            };
        }

        function animateDrawing() {
            let ticks = 0;
            const maxTicks = 30;
            
            drawInterval = setInterval(() => {
                // Cycle visual numbers
                document.getElementById('draw-red-num').textContent = availableRed[Math.floor(Math.random() * availableRed.length)];
                document.getElementById('draw-blue-num').textContent = availableBlue[Math.floor(Math.random() * availableBlue.length)];
                
                if (audioInitialized) sfxDrawTick.triggerAttackRelease("C2", "16n");

                ticks++;
                if (ticks >= maxTicks) {
                    clearInterval(drawInterval);
                    finalizeDrawing();
                }
            }, 100);
        }

        function finalizeDrawing() {
            // Pick real numbers
            const redIdx = Math.floor(Math.random() * availableRed.length);
            const blueIdx = Math.floor(Math.random() * availableBlue.length);
            
            currentRed = availableRed[redIdx];
            currentBlue = availableBlue[blueIdx];
            
            // Remove from pool
            availableRed.splice(redIdx, 1);
            availableBlue.splice(blueIdx, 1);

            document.getElementById('draw-red-num').textContent = currentRed;
            document.getElementById('draw-blue-num').textContent = currentBlue;
            
            if (audioInitialized) {
                sfxDrawWin.triggerAttackRelease("C5", "8n");
                setTimeout(() => sfxDrawWin.triggerAttackRelease("E5", "8n"), 150);
                setTimeout(() => sfxDrawWin.triggerAttackRelease("G5", "4n"), 300);
            }

            // Show waiting screen after delay
            setTimeout(() => {
                document.getElementById('wait-red').textContent = `红 ${currentRed} 号`;
                document.getElementById('wait-blue').textContent = `蓝 ${currentBlue} 号`;
                document.getElementById('player-red-num').textContent = currentRed;
                document.getElementById('player-blue-num').textContent = currentBlue;
                switchScreen('wait');
            }, 2000);
        }

        document.getElementById('btn-start-match').addEventListener('click', () => {
            questionCount = 0;
            updateScoresUI();
            switchScreen('game');
            nextQuestion();
        });

        function updateScoresUI() {
            document.getElementById('score-red').textContent = scoreRed;
            document.getElementById('score-blue').textContent = scoreBlue;
        }

        function shuffle(array) {
            let currentIndex = array.length,  randomIndex;
            while (currentIndex > 0) {
                randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;
                [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
            }
            return array;
        }

        function nextQuestion() {
            if (questionCount >= MAX_QUESTIONS) {
                endMatch();
                return;
            }

            questionCount++;
            document.getElementById('q-current').textContent = questionCount;
            questionAnswered = false;
            
            // Clear freezes
            document.getElementById('area-red').classList.remove('frozen');
            document.getElementById('area-blue').classList.remove('frozen');

            // Select Target Shape
            currentTargetShape = SHAPES[Math.floor(Math.random() * SHAPES.length)];
            
            // Re-trigger animation by cloning and replacing
            const targetEl = document.getElementById('target-shape-name');
            const newTargetEl = targetEl.cloneNode(true);
            newTargetEl.textContent = currentTargetShape.name;
            targetEl.parentNode.replaceChild(newTargetEl, targetEl);

            // Generate buttons for both sides
            renderButtons('red');
            renderButtons('blue');

            startTimer();
        }

        function renderButtons(team) {
            const grid = document.getElementById(`grid-${team}`);
            grid.innerHTML = '';
            
            // Ensure 4 unique shapes (we have exactly 4 in SHAPES array)
            const options = shuffle([...SHAPES]);
            
            options.forEach(shape => {
                const btn = document.createElement('button');
                // Use team specific styling slightly, but keep shapes colorful
                btn.className = `shape-btn bg-white rounded-2xl p-4 w-full aspect-square border-b-8 shadow-sm flex items-center justify-center text-[${shape.color}]`;
                btn.style.borderColor = shape.color;
                btn.innerHTML = shape.svg;
                
                // Interaction
                btn.onclick = () => handleAnswer(team, shape.id, btn);
                // Touch support
                btn.addEventListener('touchstart', (e) => {
                    e.preventDefault(); // Prevent double firing on some devices
                    handleAnswer(team, shape.id, btn);
                }, {passive: false});

                grid.appendChild(btn);
            });
        }

        function startTimer() {
            clearInterval(timerInterval);
            timeLeft = 10;
            document.getElementById('timer-display').textContent = timeLeft;
            
            timerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('timer-display').textContent = timeLeft;
                
                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    // Time up logic - nobody gets points, next question
                    setTimeout(nextQuestion, 1000); 
                }
            }, 1000);
        }

        function handleAnswer(team, selectedId, btnElement) {
            if (questionAnswered) return; // Block if already answered correctly
            const teamArea = document.getElementById(`area-${team}`);
            if (teamArea.classList.contains('frozen')) return; // Block if frozen

            const isCorrect = selectedId === currentTargetShape.id;

            if (isCorrect) {
                // Correct Answer
                questionAnswered = true;
                clearInterval(timerInterval);
                
                // Calculate score (timeLeft * 10)
                const points = timeLeft * 10;
                if (team === 'red') scoreRed += points;
                else scoreBlue += points;
                
                updateScoresUI();
                
                // Visual & Audio Feedback
                btnElement.classList.add('bg-green-100');
                showPopup(team, `+${points} 分!`, 'text-green-500');
                if (audioInitialized) sfxCorrect.triggerAttackRelease(["C5", "E5", "G5"], "8n");
                
                // Next question after delay
                setTimeout(nextQuestion, 1500);

            } else {
                // Wrong Answer -> Freeze
                if (audioInitialized) {
                    sfxWrong.triggerAttackRelease("C2", "8n");
                    sfxFreeze.triggerAttackRelease("8n");
                }
                btnElement.classList.add('bg-red-100', 'opacity-50');
                showPopup(team, "答错了!", 'text-red-500');
                
                teamArea.classList.add('frozen');
                
                // Unfreeze after 3s
                setTimeout(() => {
                    teamArea.classList.remove('frozen');
                    btnElement.classList.remove('bg-red-100', 'opacity-50');
                }, 3000);
            }
        }

        function showPopup(team, text, colorClass) {
            const area = document.getElementById(`popup-area-${team}`);
            const msg = document.createElement('div');
            msg.className = `float-msg ${colorClass}`;
            msg.textContent = text;
            
            // Randomize position slightly within the container
            const x = Math.random() * 40 - 20; // -20 to 20 px offset
            msg.style.transform = `translateX(${x}px)`;
            
            area.appendChild(msg);
            setTimeout(() => {
                if(area.contains(msg)) area.removeChild(msg);
            }, 1500);
        }

        function endMatch() {
            clearInterval(timerInterval);
            stopBGM();
            switchScreen('summary');
            
            document.getElementById('summary-score-red').textContent = scoreRed;
            document.getElementById('summary-score-blue').textContent = scoreBlue;

            if (audioInitialized) {
                sfxWin.triggerAttackRelease(["C4", "E4", "G4", "C5"], "2n");
            }

            // Confetti
            const duration = 3000;
            const end = Date.now() + duration;

            (function frame() {
                confetti({
                    particleCount: 5,
                    angle: 60,
                    spread: 55,
                    origin: { x: 0, y: 0.8 },
                    colors: ['#ef4444', '#facc15']
                });
                confetti({
                    particleCount: 5,
                    angle: 120,
                    spread: 55,
                    origin: { x: 1, y: 0.8 },
                    colors: ['#3b82f6', '#facc15']
                });

                if (Date.now() < end) {
                    requestAnimationFrame(frame);
                }
            }());
        }

        document.getElementById('btn-next-round').addEventListener('click', () => {
            playBGM();
            startDrawingLots();
        });

    </script>
</body>
</html>
