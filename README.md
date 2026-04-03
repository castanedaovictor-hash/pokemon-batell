<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pokémon Battle - Arceus Divine Edition</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
        
        :root {
            --hp-green: #48d0b0;
            --hp-yellow: #f8d030;
            --hp-red: #f85838;
            --arceus-gold: #f3e2a9;
        }

        body {
            font-family: 'Press Start 2P', cursive;
            background-color: #0f172a;
            color: #444;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        #game-container {
            width: 480px;
            height: 320px; 
            background: #000;
            position: relative;
            border: 8px solid #1e293b;
            border-radius: 12px;
            box-shadow: 0 0 40px rgba(0,0,0,0.9);
            transform: scale(1.5); 
            transform-origin: center;
            overflow: hidden;
        }

        .screen {
            width: 100%;
            height: 100%;
            display: none;
            flex-direction: column;
            background: white;
            position: relative;
        }

        .active { display: flex; }

        .battle-scene {
            height: 70%;
            background: linear-gradient(180deg, #60a5fa 0%, #93c5fd 50%, #f1f5f9 100%);
            position: relative;
            overflow: hidden;
        }

        .battle-scene.arceus-bg {
            background: linear-gradient(180deg, #1e293b 0%, #475569 100%);
        }

        .ui-panel {
            height: 30%;
            background: #2d3748;
            border-top: 4px solid #1a202c;
            display: grid;
            grid-template-columns: 1.2fr 1fr;
            padding: 4px;
            gap: 4px;
        }

        .status-box {
            position: absolute;
            background: rgba(255, 255, 255, 0.9);
            border: 2px solid #2d3748;
            border-radius: 4px;
            padding: 4px 6px;
            width: 150px;
            box-shadow: 2px 2px 5px rgba(0,0,0,0.2);
            z-index: 20;
        }

        .enemy-status { top: 12px; left: 12px; border-bottom-right-radius: 15px; }
        .player-status { bottom: 12px; right: 12px; border-top-left-radius: 15px; }

        .hp-bar-outer {
            width: 100%;
            height: 6px;
            background: #cbd5e1;
            border: 1px solid #475569;
            border-radius: 3px;
            margin-top: 3px;
            overflow: hidden;
        }

        .hp-bar-inner {
            height: 100%;
            width: 100%;
            transition: width 0.4s ease-out;
        }

        .arceus-aura {
            position: absolute;
            width: 160px; /* Un poco más pequeña */
            height: 160px;
            background: radial-gradient(circle, rgba(255,255,200,0.6) 0%, rgba(255,255,255,0) 70%);
            border-radius: 50%;
            animation: pulseAura 3s infinite alternate;
            z-index: 1;
        }

        @keyframes pulseAura {
            0% { transform: scale(0.8); opacity: 0.3; }
            100% { transform: scale(1.1); opacity: 0.6; }
        }

        .arceus-float {
            animation: floatArceus 4s infinite ease-in-out;
            z-index: 5;
            filter: drop-shadow(0 0 10px rgba(255,215,0,0.5));
            max-width: 100px; /* Limitar tamaño visual */
        }

        @keyframes floatArceus {
            0%, 100% { transform: translateY(0) rotate(0deg); }
            50% { transform: translateY(-10px) rotate(1deg); }
        }

        .sprite {
            position: absolute;
            width: 90px;
            height: 90px;
            object-fit: contain;
            z-index: 10;
            filter: drop-shadow(2px 4px 6px rgba(0,0,0,0.2));
        }

        .enemy-sprite { top: 10px; right: 50px; }
        .player-sprite { bottom: 0px; left: 35px; transform: scale(1.4); }

        .platform {
            position: absolute;
            background: radial-gradient(ellipse at center, rgba(0,0,0,0.3) 0%, rgba(0,0,0,0) 70%);
            border-radius: 50%;
            z-index: 0;
        }

        .dialog-box {
            background: #fff;
            border: 2px solid #4a5568;
            border-radius: 6px;
            padding: 8px;
            font-size: 8px;
            line-height: 1.3;
            color: #2d3748;
        }

        .command-box {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 2px;
        }

        .btn-cmd {
            background: #edf2f7;
            border: 1px solid #cbd5e1;
            border-radius: 4px;
            font-family: 'Press Start 2P', cursive;
            font-size: 6px;
            padding: 6px;
            cursor: pointer;
            transition: all 0.1s;
            color: #4a5568;
        }

        .btn-cmd:hover:not(:disabled) { 
            background: #4299e1; 
            color: white;
            transform: translateY(-1px);
        }

        .shake { animation: shake 0.3s cubic-bezier(.36,.07,.19,.97) both; }
        @keyframes shake {
            10%, 90% { transform: translate3d(-1px, 0, 0); }
            20%, 80% { transform: translate3d(2px, 0, 0); }
            30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
            40%, 60% { transform: translate3d(4px, 0, 0); }
        }

        .starter-card {
            border: 1px solid #e2e8f0;
            padding: 6px;
            text-align: center;
            cursor: pointer;
            background: white;
            border-radius: 8px;
            transition: transform 0.2s;
        }

        .starter-card:hover { 
            transform: translateY(-3px);
            border-color: #3182ce;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }

        .star-icon { color: #f59e0b; margin-right: 2px; }
    </style>
</head>
<body>

<div id="game-container">
    
    <!-- Inicio -->
    <div id="screen-start" class="screen active items-center justify-center p-6 text-center" style="background: radial-gradient(circle, #ffffff 0%, #e2e8f0 100%);">
        <div class="arceus-aura"></div>
        <img src="https://play.pokemonshowdown.com/sprites/ani/arceus.gif" class="w-28 mb-4 arceus-float relative" alt="Arceus">
        <div class="bg-white/80 backdrop-blur border-2 border-yellow-200 p-4 rounded-2xl mb-4 shadow-xl w-full min-h-[60px] flex items-center justify-center relative z-10">
            <p id="arceus-quote" class="text-[6px] leading-relaxed italic text-gray-800">"El universo nace de la voluntad..."</p>
        </div>
        <button onclick="startGame()" class="relative z-10 text-[8px] bg-gradient-to-r from-yellow-400 to-yellow-600 text-white px-8 py-4 rounded-full hover:scale-105 transition-all border-b-4 border-yellow-800 active:translate-y-1 active:border-0 shadow-lg">CONECTAR CON EL ORIGEN</button>
    </div>

    <!-- Selección y Tienda -->
    <div id="screen-select" class="screen flex-col bg-gray-50">
        <div class="flex shadow-md">
            <button id="tab-pkmn" class="flex-1 p-3 text-[7px] bg-blue-500 text-white font-bold" onclick="switchTab('pkmn')">EQUIPO</button>
            <button id="tab-shop" class="flex-1 p-3 text-[7px] bg-gray-200" onclick="switchTab('shop')">TIENDA</button>
        </div>
        <div id="wallet-display" class="bg-blue-900 text-white p-2 text-[6px] text-center">CRÉDITOS: $0</div>
        
        <div id="view-pkmn" class="flex-grow overflow-hidden flex flex-col">
            <div class="starter-grid grid grid-cols-3 gap-2 p-3 overflow-y-auto" id="starter-list"></div>
        </div>

        <div id="view-shop" class="hidden flex-grow overflow-y-auto p-4">
            <div class="space-y-2" id="shop-items-container"></div>
        </div>
    </div>

    <!-- Batalla -->
    <div id="screen-battle" class="screen">
        <div id="battle-scene-box" class="battle-scene">
            <div class="status-box enemy-status">
                <div class="flex justify-between text-[6px] font-bold text-gray-700">
                    <span id="enemy-name">---</span>
                    <span id="enemy-lv">Lv--</span>
                </div>
                <div class="hp-bar-outer">
                    <div id="enemy-hp-bar" class="hp-bar-inner bg-[#48d0b0]"></div>
                </div>
            </div>
            <div class="platform w-28 h-10 top-[110px] right-10"></div>
            <img id="enemy-sprite" class="sprite enemy-sprite" src="">

            <div class="status-box player-status">
                <div class="flex justify-between text-[6px] font-bold text-gray-700">
                    <span id="player-name">---</span>
                    <span id="player-lv">Lv--</span>
                </div>
                <div class="hp-bar-outer">
                    <div id="player-hp-bar" class="hp-bar-inner bg-[#48d0b0]"></div>
                </div>
                <div class="flex justify-between text-[5px] mt-1 text-gray-500">
                    <span id="player-xp-text">XP: 0/100</span>
                    <span id="player-hp-text">--/ --</span>
                </div>
            </div>
            <div class="platform w-32 h-12 bottom-6 left-6"></div>
            <img id="player-sprite" class="sprite player-sprite" src="">
        </div>

        <div class="ui-panel">
            <div class="dialog-box flex items-center justify-center text-center" id="battle-text">
                ¿Qué hará el entrenador?
            </div>
            
            <div class="command-box" id="menu-main">
                <button class="btn-cmd" onclick="showMoves()">LUCHA</button>
                <button class="btn-cmd" onclick="showBag()">MOCHILA</button>
                <button class="btn-cmd" onclick="cancelBattle()">EQUIPO</button>
                <button class="btn-cmd" onclick="runAway()">HUIDA</button>
            </div>

            <div class="command-box hidden" id="menu-moves">
                <button class="btn-cmd" id="m0" onclick="handleMove(0)">---</button>
                <button class="btn-cmd" id="m1" onclick="handleMove(1)">---</button>
                <button class="btn-cmd" id="m2" onclick="handleMove(2)">---</button>
                <button class="btn-cmd" id="m3" onclick="handleMove(3)">---</button>
                <button class="btn-cmd bg-red-100" onclick="hideSubMenus()">ATRÁS</button>
            </div>

            <div class="command-box hidden" id="menu-bag">
                <button class="btn-cmd" id="b0" onclick="useItem('potion40')">BAYA 40%</button>
                <button class="btn-cmd" id="b1" onclick="useItem('potion60')">POC. 60%</button>
                <button class="btn-cmd" id="b2" onclick="useItem('potion100')">POC. TOTAL</button>
                <button class="btn-cmd bg-red-100" onclick="hideSubMenus()">ATRÁS</button>
            </div>
        </div>
    </div>

</div>

<script>
    // --- SISTEMA DE AUDIO (BGM & SFX) ---
    const AudioEngine = {
        ctx: null,
        mainGain: null,
        musicOsc: null,
        currentBGM: null,
        
        init() {
            if (!this.ctx) {
                this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                this.mainGain = this.ctx.createGain();
                this.mainGain.gain.value = 0.05;
                this.mainGain.connect(this.ctx.destination);
            }
        },

        // SFX de corto tiempo
        play(freq, type = 'sine', duration = 0.1, vol = 0.1) {
            this.init();
            const osc = this.ctx.createOscillator();
            const gain = this.ctx.createGain();
            osc.type = type;
            osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
            gain.gain.setValueAtTime(vol, this.ctx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + duration);
            osc.connect(gain);
            gain.connect(this.ctx.destination);
            osc.start();
            osc.stop(this.ctx.currentTime + duration);
        },

        // Motor de música (Síntesis de melodías simples)
        playBGM(type) {
            this.init();
            if (this.currentBGM === type) return;
            this.stopBGM();
            this.currentBGM = type;

            const notes = type === 'lobby' 
                ? [261.63, 329.63, 392.00, 523.25] // Do Mi Sol Do (Tranquilo)
                : [110.00, 123.47, 130.81, 146.83]; // La Si Do Re (Bajo tenso)

            let step = 0;
            this.musicInterval = setInterval(() => {
                if(type === 'lobby') {
                    // Melodía tipo GameBoy Lobby
                    this.play(notes[step % notes.length], 'triangle', 0.4, 0.03);
                    if(step % 4 === 0) this.play(notes[0]/2, 'sine', 0.2, 0.02);
                } else {
                    // Melodía Batalla Rápida
                    this.play(notes[step % notes.length] * (step % 2 + 1), 'square', 0.1, 0.02);
                    if(step % 2 === 0) this.play(55, 'sine', 0.1, 0.04);
                }
                step++;
            }, type === 'lobby' ? 600 : 200);
        },

        stopBGM() {
            if(this.musicInterval) clearInterval(this.musicInterval);
            this.currentBGM = null;
        },

        playHit() { this.play(150, 'square', 0.1, 0.05); this.play(80, 'sine', 0.2, 0.1); },
        playSelect() { this.play(800, 'sine', 0.05, 0.05); },
        playHeal() {
            this.play(400, 'sine', 0.1, 0.05);
            setTimeout(() => this.play(600, 'sine', 0.1, 0.05), 100);
            setTimeout(() => this.play(800, 'sine', 0.1, 0.05), 200);
        },
        playLevelUp() {
            [523, 659, 783, 1046].forEach((f, i) => {
                setTimeout(() => this.play(f, 'square', 0.2, 0.05), i * 150);
            });
        }
    };

    const quoteGenerator = {
        subjects: ["Tu determinación", "El alma de tu Pokémon", "Cada paso en tu viaje", "Tu voluntad", "La fuerza de tus lazos", "Tu espíritu de lucha", "La luz de tu corazón", "Tu valentía", "La sombra de tu duda", "Tu fe en el camino"],
        verbs: ["es capaz de", "forjará", "superará", "iluminará", "guiará", "desbloqueará", "trascenderá", "protegerá", "transformará", "aniquilará"],
        predicates: ["el destino de todo el mundo.", "cualquier obstáculo en tu camino.", "el secreto de la megaevolución.", "un futuro lleno de gloria.", "los límites del universo.", "la oscuridad de cualquier batalla.", "un poder nunca antes visto.", "la paz entre humanos y Pokémon.", "el velo de la realidad conocida.", "la esencia de la creación misma."],
        extra: ["¡Sigue adelante!", "¡No te rindas!", "¡El mundo te observa!", "¡Cree en tu poder!", "¡La victoria es tuya!", "¡Arceus te guía!", "¡Siente la energía del cosmos!"]
    };

    function generateArceusQuote() {
        const s = quoteGenerator.subjects[Math.floor(Math.random() * quoteGenerator.subjects.length)];
        const v = quoteGenerator.verbs[Math.floor(Math.random() * quoteGenerator.verbs.length)];
        const p = quoteGenerator.predicates[Math.floor(Math.random() * quoteGenerator.predicates.length)];
        const e = quoteGenerator.extra[Math.floor(Math.random() * quoteGenerator.extra.length)];
        return `${s} ${v} ${p} ${e}`;
    }

    const MAX_LEVEL = 100;
    const EVO_LEVEL_1 = 16, EVO_LEVEL_2 = 32;
    const MOVE_POOL = {
        planta: ['Placaje', 'Látigo Cepa', 'Hoja Afilada', 'Rayo Solar', 'Drenadoras', 'Gigadrenado', 'Tormenta Hojas', 'Sintesis'],
        fuego: ['Arañazo', 'Ascuas', 'Lanzallamas', 'Garra Dragón', 'Fuego Fatuo', 'Llamarada', 'Envite Ígneo', 'Sofoco'],
        agua: ['Placaje', 'Pistola Agua', 'Rayo Burbuja', 'Hidrobomba', 'Giro Rápido', 'Rayo Hielo', 'Surf', 'Escaldar']
    };

    const POKEMON_DATA = {
        1: { name: 'Bulbasaur', next: 2, type: 'planta' }, 2: { name: 'Ivysaur', next: 3, type: 'planta' }, 3: { name: 'Venusaur', type: 'planta' },
        4: { name: 'Charmander', next: 5, type: 'fuego' }, 5: { name: 'Charmeleon', next: 6, type: 'fuego' }, 6: { name: 'Charizard', type: 'fuego' },
        7: { name: 'Squirtle', next: 8, type: 'agua' }, 8: { name: 'Wartortle', next: 9, type: 'agua' }, 9: { name: 'Blastoise', type: 'agua' },
        152: { name: 'Chikorita', next: 153, type: 'planta' }, 153: { name: 'Bayleef', next: 154, type: 'planta' }, 154: { name: 'Meganium', type: 'planta' },
        155: { name: 'Cyndaquil', next: 156, type: 'fuego' }, 156: { name: 'Quilava', next: 157, type: 'fuego' }, 157: { name: 'Typhlosion', type: 'fuego' },
        158: { name: 'Totodile', next: 159, type: 'agua' }, 159: { name: 'Croconaw', next: 160, type: 'agua' }, 160: { name: 'Feraligatr', type: 'agua' },
        252: { name: 'Treecko', next: 253, type: 'planta' }, 253: { name: 'Grovyle', next: 254, type: 'planta' }, 254: { name: 'Sceptile', type: 'planta' },
        255: { name: 'Torchic', next: 256, type: 'fuego' }, 256: { name: 'Combusken', next: 257, type: 'fuego' }, 257: { name: 'Blaziken', type: 'fuego' },
        258: { name: 'Mudkip', next: 259, type: 'agua' }, 259: { name: 'Marshtomp', next: 260, type: 'agua' }, 260: { name: 'Swampert', type: 'agua' },
        387: { name: 'Turtwig', next: 388, type: 'planta' }, 388: { name: 'Grotle', next: 389, type: 'planta' }, 389: { name: 'Torterra', type: 'planta' },
        390: { name: 'Chimchar', next: 391, type: 'fuego' }, 391: { name: 'Monferno', next: 392, type: 'fuego' }, 392: { name: 'Infernape', type: 'fuego' },
        393: { name: 'Piplup', next: 394, type: 'agua' }, 394: { name: 'Prinplup', next: 395, type: 'agua' }, 395: { name: 'Empoleon', type: 'agua' },
        495: { name: 'Snivy', next: 496, type: 'planta' }, 496: { name: 'Servine', next: 497, type: 'planta' }, 497: { name: 'Serperior', type: 'planta' },
        498: { name: 'Tepig', next: 499, type: 'fuego' }, 499: { name: 'Pignite', next: 500, type: 'fuego' }, 500: { name: 'Emboar', type: 'fuego' },
        501: { name: 'Oshawott', next: 502, type: 'agua' }, 502: { name: 'Dewott', next: 503, type: 'agua' }, 503: { name: 'Samurott', type: 'agua' },
        650: { name: 'Chespin', next: 651, type: 'planta' }, 651: { name: 'Quilladin', next: 652, type: 'planta' }, 652: { name: 'Chesnaught', type: 'planta' },
        653: { name: 'Fennekin', next: 654, type: 'fuego' }, 654: { name: 'Braixen', next: 655, type: 'fuego' }, 655: { name: 'Delphox', type: 'fuego' },
        656: { name: 'Froakie', next: 657, type: 'agua' }, 657: { name: 'Frogadier', next: 658, type: 'agua' }, 658: { name: 'Greninja', type: 'agua' },
        722: { name: 'Rowlet', next: 723, type: 'planta' }, 723: { name: 'Dartrix', next: 724, type: 'planta' }, 724: { name: 'Decidueye', type: 'planta' },
        725: { name: 'Litten', next: 726, type: 'fuego' }, 726: { name: 'Torracat', next: 727, type: 'fuego' }, 727: { name: 'Incineroar', type: 'fuego' },
        728: { name: 'Popplio', next: 729, type: 'agua' }, 729: { name: 'Brionne', next: 730, type: 'agua' }, 730: { name: 'Primarina', type: 'agua' },
        810: { name: 'Grookey', next: 811, type: 'planta' }, 811: { name: 'Thwackey', next: 812, type: 'planta' }, 812: { name: 'Rillaboom', type: 'planta' },
        813: { name: 'Scorbunny', next: 814, type: 'fuego' }, 814: { name: 'Raboot', next: 815, type: 'fuego' }, 815: { name: 'Cinderace', type: 'fuego' },
        816: { name: 'Sobble', next: 817, type: 'agua' }, 817: { name: 'Drizzile', next: 818, type: 'agua' }, 818: { name: 'Inteleon', type: 'agua' },
        906: { name: 'Sprigatito', next: 907, type: 'planta' }, 907: { name: 'Floragato', next: 908, type: 'planta' }, 908: { name: 'Meowscarada', type: 'planta' },
        909: { name: 'Fuecoco', next: 910, type: 'fuego' }, 910: { name: 'Crocalor', next: 911, type: 'fuego' }, 911: { name: 'Skeledirge', type: 'fuego' },
        912: { name: 'Quaxly', next: 913, type: 'agua' }, 913: { name: 'Quaxwell', next: 914, type: 'agua' }, 914: { name: 'Quaquaval', type: 'agua' }
    };

    const ALL_STARTER_IDS = [1, 4, 7, 152, 155, 158, 252, 255, 258, 387, 390, 393, 495, 498, 501, 650, 653, 656, 722, 725, 728, 810, 813, 816, 906, 909, 912];

    let gameState = { 
        money: 500, 
        inventory: { potion40: 2, potion60: 1, potion100: 0 }, 
        ownedPokemon: {},
        hasGoldAmulet: false
    };
    let player = null, enemy = null, turnBusy = false, isArceusBattle = false;

    function sleep(ms) { return new Promise(resolve => setTimeout(resolve, ms)); }

    function get3DSprite(id, back = false) {
        const name = POKEMON_DATA[id].name.toLowerCase();
        return `https://play.pokemonshowdown.com/sprites/ani${back ? '-back' : ''}/${name}.gif`;
    }

    function showScreen(id) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        
        if(id === 'screen-start') {
            document.getElementById('arceus-quote').innerText = `"${generateArceusQuote()}"`;
            AudioEngine.stopBGM();
        }
        if(id === 'screen-select') { 
            updateWallet(); 
            initStarterList(); 
            renderShop(); 
            AudioEngine.playBGM('lobby');
        }
        if(id === 'screen-battle') {
            AudioEngine.playBGM('battle');
        }
    }

    function startGame() {
        AudioEngine.playSelect();
        showScreen('screen-select');
    }

    function switchTab(tab) {
        AudioEngine.playSelect();
        document.getElementById('view-pkmn').classList.toggle('hidden', tab !== 'pkmn');
        document.getElementById('view-shop').classList.toggle('hidden', tab !== 'shop');
        document.getElementById('tab-pkmn').className = tab === 'pkmn' ? 'flex-1 p-3 text-[7px] bg-blue-500 text-white font-bold' : 'flex-1 p-3 text-[7px] bg-gray-200';
        document.getElementById('tab-shop').className = tab === 'shop' ? 'flex-1 p-3 text-[7px] bg-blue-500 text-white font-bold' : 'flex-1 p-3 text-[7px] bg-gray-200';
    }

    function updateWallet() { document.getElementById('wallet-display').innerText = `CRÉDITOS: $${gameState.money}`; }

    function renderShop() {
        const container = document.getElementById('shop-items-container');
        const items = [
            { id: 'potion40', name: 'BAYA MILAGROSA 40%', price: 100 },
            { id: 'potion60', name: 'POCIÓN 60%', price: 200 },
            { id: 'potion100', name: 'POCIÓN TOTAL', price: 800 },
            { id: 'amulet', name: 'AMULETO ORO (X2 $$$)', price: 5000 }
        ];
        container.innerHTML = items.map(item => `
            <div class="flex justify-between items-center bg-white p-3 border border-gray-200 rounded-lg shadow-sm" onclick="buyItem('${item.id}')">
                <span class="text-[6px] font-bold">${item.name}</span>
                <span class="text-[6px] bg-green-100 text-green-700 px-2 py-1 rounded">$${item.price}</span>
            </div>
        `).join('');
    }

    function buyItem(id) {
        const prices = { potion40: 100, potion60: 200, potion100: 800, amulet: 5000 };
        if(id === 'amulet' && gameState.hasGoldAmulet) return;
        if(gameState.money >= prices[id]) {
            AudioEngine.playHeal();
            gameState.money -= prices[id];
            if(id === 'amulet') gameState.hasGoldAmulet = true;
            else gameState.inventory[id]++;
            updateWallet();
            renderShop();
        }
    }

    function initStarterList() {
        const list = document.getElementById('starter-list');
        list.innerHTML = "";
        ALL_STARTER_IDS.forEach(id => {
            const data = POKEMON_DATA[id], owned = gameState.ownedPokemon[id];
            const level = owned ? owned.level : 5, currentId = owned ? owned.currentId : id;
            const star = (owned && owned.hasStar) ? '<span class="star-icon">★</span>' : '';
            const card = document.createElement('div');
            card.className = 'starter-card';
            card.innerHTML = `
                <div class="text-[4px] bg-blue-600 text-white rounded px-1 mb-1 inline-block">Nv ${level}</div>
                <img src="https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${currentId}.png" class="mx-auto w-10 h-10">
                <div class="font-bold text-gray-800 text-[5px] truncate">${star}${data.name.toUpperCase()}</div>
            `;
            card.onclick = () => selectStarter(id);
            list.appendChild(card);
        });
    }

    function selectStarter(baseId) {
        AudioEngine.playSelect();
        const data = POKEMON_DATA[baseId];
        if (gameState.ownedPokemon[baseId]) player = gameState.ownedPokemon[baseId];
        else {
            player = { baseId, currentId: baseId, name: data.name, level: 5, hp: 50, maxHp: 50, xp: 0, maxXp: 100, type: data.type, moves: [], hasStar: false };
            gameState.ownedPokemon[baseId] = player;
            updateMoveset();
        }
        startBattle();
    }

    function updateMoveset() {
        const pool = MOVE_POOL[player.type] || MOVE_POOL.planta;
        player.moves = pool.slice(0, Math.min(Math.floor(player.level / 4) + 1, 4));
    }

    async function startBattle() {
        isArceusBattle = (player.level >= 20 && Math.random() < 0.15); 
        document.getElementById('battle-scene-box').classList.toggle('arceus-bg', isArceusBattle);

        if(isArceusBattle) {
            enemy = { id: 493, name: 'Arceus', level: '∞', hp: 5000, maxHp: 5000 };
            updateBattleUI();
            showScreen('screen-battle');
            writeMsg("¡EL ORIGEN TE RECLAMA! ¡ARCEUS APARECIÓ!");
        } else {
            const wildIds = [10, 16, 19, 21, 23, 25, 41, 74, 129, 92, 133, 147, 448, 445, 150, 248, 376, 635, 149];
            const rid = wildIds[Math.floor(Math.random() * wildIds.length)];
            const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${rid}`);
            const data = await res.json();
            const enemyLvl = Math.max(2, player.level + (Math.floor(Math.random() * 5) - 2));
            enemy = { id: rid, name: data.name, level: enemyLvl, hp: 25 + (enemyLvl * 6), maxHp: 25 + (enemyLvl * 6) };
            updateBattleUI();
            showScreen('screen-battle');
            writeMsg(`¡Un ${enemy.name.toUpperCase()} salvaje apareció!`);
        }
        hideSubMenus();
    }

    function cancelBattle() {
        AudioEngine.playSelect();
        showScreen('screen-select');
    }

    function runAway() {
        AudioEngine.playSelect();
        writeMsg("¡Escapaste sano y salvo!");
        setTimeout(() => showScreen('screen-select'), 1000);
    }

    function updateBattleUI() {
        const star = player.hasStar ? '★' : '';
        document.getElementById('player-name').innerText = star + player.name.toUpperCase();
        document.getElementById('player-lv').innerText = `Lv${player.level}`;
        document.getElementById('player-sprite').src = get3DSprite(player.currentId, true);
        document.getElementById('player-xp-text').innerText = `XP: ${player.xp}/${player.maxXp}`;
        document.getElementById('enemy-name').innerText = enemy.name.toUpperCase();
        document.getElementById('enemy-lv').innerText = `Lv${enemy.level}`;
        
        const enemyImg = document.getElementById('enemy-sprite');
        enemyImg.src = `https://play.pokemonshowdown.com/sprites/ani/${enemy.name.toLowerCase()}.gif`;
        enemyImg.onerror = function() {
            this.src = `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${enemy.id}.png`;
        };
        updateHpUI();
    }

    function updateHpUI() {
        const pPct = (player.hp / player.maxHp) * 100, ePct = (enemy.hp / enemy.maxHp) * 100;
        const pBar = document.getElementById('player-hp-bar'), eBar = document.getElementById('enemy-hp-bar');
        pBar.style.width = Math.max(0, pPct) + '%';
        pBar.style.backgroundColor = pPct < 20 ? 'var(--hp-red)' : pPct < 50 ? 'var(--hp-yellow)' : 'var(--hp-green)';
        eBar.style.width = Math.max(0, ePct) + '%';
        eBar.style.backgroundColor = ePct < 20 ? 'var(--hp-red)' : ePct < 50 ? 'var(--hp-yellow)' : 'var(--hp-green)';
        document.getElementById('player-hp-text').innerText = `${Math.floor(Math.max(0, player.hp))}/${player.maxHp}`;
    }

    function writeMsg(txt) { document.getElementById('battle-text').innerText = txt; }

    function showMoves() {
        if(turnBusy) return;
        AudioEngine.playSelect();
        document.getElementById('menu-main').classList.add('hidden');
        document.getElementById('menu-moves').classList.remove('hidden');
        for(let i = 0; i < 4; i++) {
            const btn = document.getElementById(`m${i}`);
            if (player.moves[i]) { btn.innerText = player.moves[i].toUpperCase(); btn.classList.remove('hidden'); }
            else { btn.classList.add('hidden'); }
        }
    }

    function showBag() {
        if(turnBusy) return;
        AudioEngine.playSelect();
        document.getElementById('menu-main').classList.add('hidden');
        document.getElementById('menu-bag').classList.remove('hidden');
        document.getElementById('b0').innerText = `BAYA 40% x${gameState.inventory.potion40}`;
        document.getElementById('b1').innerText = `POC. 60% x${gameState.inventory.potion60}`;
        document.getElementById('b2').innerText = `POC. 100% x${gameState.inventory.potion100}`;
    }

    function hideSubMenus() {
        AudioEngine.playSelect();
        document.getElementById('menu-main').classList.remove('hidden');
        document.getElementById('menu-moves').classList.add('hidden');
        document.getElementById('menu-bag').classList.add('hidden');
    }

    async function handleMove(i) {
        if(turnBusy) return; turnBusy = true; 
        document.getElementById('menu-moves').classList.add('hidden'); 
        writeMsg(`¡${player.name.toUpperCase()} usó ${player.moves[i].toUpperCase()}!`);
        await sleep(600);
        
        AudioEngine.playHit();
        let dmg = (8 + Math.floor(player.level * 2));
        if(isArceusBattle) dmg = Math.floor(dmg / 2); 
        enemy.hp -= dmg;
        document.getElementById('enemy-sprite').classList.add('shake');
        updateHpUI();
        await sleep(400);
        document.getElementById('enemy-sprite').classList.remove('shake');
        if(enemy.hp <= 0) return win();
        enemyTurn();
    }

    async function useItem(itemId) {
        if(gameState.inventory[itemId] <= 0) return;
        turnBusy = true; 
        document.getElementById('menu-bag').classList.add('hidden');
        AudioEngine.playHeal();
        gameState.inventory[itemId]--;
        const percent = itemId === 'potion40' ? 0.4 : itemId === 'potion60' ? 0.6 : 1;
        player.hp = Math.min(player.maxHp, player.hp + Math.floor(player.maxHp * percent));
        writeMsg(`¡Se usó el objeto curativo!`);
        updateHpUI();
        await sleep(1000); 
        enemyTurn();
    }

    async function enemyTurn() {
        writeMsg(`${enemy.name.toUpperCase()} contraataca.`);
        await sleep(800);
        
        AudioEngine.playHit();
        let damage;
        if(isArceusBattle) {
            const rand = Math.random();
            if(rand < 0.4) { damage = player.maxHp * 0.05; writeMsg("Sentencia: Golpe leve"); }
            else if(rand < 0.7) { damage = player.maxHp * 0.40; writeMsg("Sentencia: Golpe grave"); }
            else { damage = player.maxHp * 0.95; writeMsg("Sentencia: ¡EL FIN ESTÁ CERCA!"); }
        } else {
            damage = (5 + Math.floor(enemy.level * 1.2));
        }

        player.hp -= damage;
        document.getElementById('player-sprite').classList.add('shake');
        updateHpUI();
        await sleep(400);
        document.getElementById('player-sprite').classList.remove('shake');
        if(player.hp <= 0) return lose();
        writeMsg(`¿Qué harás ahora?`);
        document.getElementById('menu-main').classList.remove('hidden');
        turnBusy = false;
    }

    async function win() {
        writeMsg(`¡${enemy.name.toUpperCase()} se debilitó!`); await sleep(1000);
        
        let reward = (100 + (enemy.level === '∞' ? 10000 : enemy.level * 25));
        if(gameState.hasGoldAmulet) reward *= 2;
        gameState.money += reward;

        if(isArceusBattle) {
            AudioEngine.playLevelUp();
            player.hasStar = true;
            writeMsg("¡HAS DERROTADO AL DIOS! Recibes la Estrella.");
            await sleep(2000);
        }

        player.xp += (50 + (enemy.level === '∞' ? 0 : enemy.level * 40));
        if(player.xp >= player.maxXp && player.level < MAX_LEVEL) await levelUp();
        
        showScreen('screen-select'); turnBusy = false;
    }

    async function lose() {
        writeMsg("¡Tu Pokémon se ha debilitado!");
        await sleep(2000);
        player.hp = player.maxHp;
        showScreen('screen-select');
        turnBusy = false;
    }

    async function levelUp() {
        AudioEngine.playLevelUp();
        player.level++; player.xp -= player.maxXp; player.maxXp = Math.floor(player.maxXp * 1.3);
        player.maxHp += 10; player.hp = player.maxHp;
        writeMsg(`¡Nv. ${player.level} alcanzado!`); await sleep(1200);
        if(player.level % 4 === 0) { updateMoveset(); writeMsg(`¡Nuevo ataque aprendido!`); await sleep(1200); }
        if(player.level === EVO_LEVEL_1 || player.level === EVO_LEVEL_2) {
            const nextEvo = POKEMON_DATA[player.currentId]?.next;
            if(nextEvo) await evolve(nextEvo);
        }
    }

    async function evolve(nextId) {
        writeMsg(`¿Eh? ¡${player.name.toUpperCase()} está evolucionando!`);
        await sleep(2000);
        player.currentId = nextId;
        player.name = POKEMON_DATA[nextId].name;
        writeMsg(`¡Felicidades! Evolucionó en ${player.name.toUpperCase()}`);
        await sleep(1500);
    }

    window.onload = () => {
        // Inicializar audio con interacción
        document.body.addEventListener('click', () => AudioEngine.init(), { once: true });
    };

</script>
</body>
</html>
