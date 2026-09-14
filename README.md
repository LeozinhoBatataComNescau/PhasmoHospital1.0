<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PhasmoHospital: Toon Asylum</title>
    <link href="https://fonts.googleapis.com/css2?family=Fredoka+One&family=Mali:wght@600;700&display=swap" rel="stylesheet">
    <style>
        * { box-sizing: border-box; user-select: none; }
        body { 
            margin: 0; 
            overflow: hidden; 
            background: #0d0714; 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            height: 100vh; 
            font-family: 'Fredoka One', 'Mali', cursive, sans-serif; 
            color: #ffffff; 
        }
        #game-wrapper {
            position: relative;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8), 0 0 40px rgba(168, 85, 247, 0.4);
            border-radius: 28px;
            overflow: hidden;
            border: 8px solid #a855f7;
            background: #1a0c2e;
        }
        canvas { 
            display: block; 
            background: #180928; 
            /* Smooth vectors instead of pixelated rendering */
            image-rendering: auto; 
        }
        
        .overlay { 
            position: absolute; 
            top: 0; left: 0; 
            width: 100%; height: 100%; 
            display: flex; 
            flex-direction: column; 
            justify-content: center; 
            align-items: center; 
            background: rgba(22, 10, 38, 0.94); 
            z-index: 20; 
            backdrop-filter: blur(10px);
            transition: all 0.3s ease;
        }
        .hidden { display: none !important; opacity: 0; pointer-events: none; }
        
        .logo-title { 
            font-size: 58px; 
            color: #ff3366; 
            text-shadow: 4px 4px 0px #1a052e, -3px -3px 0px #ff99c8, 0 8px 15px rgba(255, 51, 102, 0.4); 
            margin: 0 0 8px 0; 
            letter-spacing: 3px; 
            transform: rotate(-2deg);
        }
        .subtitle { 
            font-size: 20px; 
            color: #c084fc; 
            margin-bottom: 30px; 
            font-family: 'Mali', cursive;
            font-weight: 700;
            text-shadow: 0 2px 8px rgba(0,0,0,0.5);
        }
        
        button { 
            background: linear-gradient(180deg, #ff4d8d 0%, #d91b5c 100%); 
            border: 4px solid #ff99c8; 
            border-radius: 50px;
            color: white; 
            padding: 16px 36px; 
            font-family: inherit; 
            font-size: 22px; 
            cursor: pointer; 
            margin: 10px; 
            transition: transform 0.15s ease, box-shadow 0.15s ease, background 0.2s; 
            width: 400px; 
            text-transform: uppercase; 
            letter-spacing: 1px;
            box-shadow: 0 8px 0 #8a0b38, 0 12px 25px rgba(0,0,0,0.5);
            text-shadow: 2px 2px 0px #4d0019;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }
        button:hover { 
            transform: translateY(-4px) scale(1.02); 
            background: linear-gradient(180deg, #ff669d 0%, #ed2b6c 100%); 
            box-shadow: 0 12px 0 #8a0b38, 0 16px 30px rgba(255, 77, 141, 0.5); 
        }
        button:active {
            transform: translateY(4px);
            box-shadow: 0 4px 0 #8a0b38;
        }

        #ui-ingame { 
            position: absolute; 
            top: 20px; left: 20px; 
            pointer-events: none; 
            z-index: 10; 
            width: 960px; 
        }
        .stats-row { display: flex; justify-content: space-between; align-items: flex-start; }
        
        .card-ui {
            background: rgba(30, 15, 52, 0.9);
            border: 4px solid #a855f7;
            border-radius: 20px;
            padding: 12px 22px;
            box-shadow: 0 8px 0 rgba(0,0,0,0.5), inset 0 2px 4px rgba(255,255,255,0.2);
            backdrop-filter: blur(6px);
        }

        .hp-bar { color: #ff477e; font-size: 24px; text-shadow: 2px 2px 0 #1a052e; display: flex; align-items: center; gap: 8px; }
        .essence-box { color: #ffc107; font-size: 20px; text-shadow: 2px 2px 0 #1a052e; margin-top: 4px; display: flex; align-items: center; gap: 8px; }
        
        .bar-bg { 
            width: 240px; height: 22px; 
            background: #120624; 
            border: 3px solid #38bdf8; 
            border-radius: 14px; 
            margin-top: 6px; 
            overflow: hidden; 
            box-shadow: inset 0 2px 6px rgba(0,0,0,0.8);
        }
        #crucifix-bar { 
            width: 0%; height: 100%; 
            background: linear-gradient(90deg, #38bdf8, #818cf8, #f0f9ff); 
            box-shadow: 0 0 12px #38bdf8; 
            transition: width 0.1s linear; 
        }
        
        .upgrade-container { 
            background: #1e0e38; 
            padding: 30px; 
            border-radius: 28px; 
            border: 5px solid #a855f7; 
            box-shadow: 0 12px 0 #0d041c; 
            display: flex;
            flex-direction: column;
            gap: 14px;
        }

        .btn-secondary {
            background: linear-gradient(180deg, #64748b 0%, #334155 100%);
            border-color: #94a3b8;
            box-shadow: 0 8px 0 #1e293b;
            text-shadow: 2px 2px 0 #0f172a;
        }
        .btn-secondary:hover {
            background: linear-gradient(180deg, #94a3b8 0%, #475569 100%);
            box-shadow: 0 12px 0 #1e293b;
        }
    </style>
</head>
<body>

    <div id="game-wrapper">
        <div id="ui-ingame" class="hidden">
            <div class="stats-row">
                <div class="card-ui">
                    <div class="hp-bar">❤️ FÉ: <span id="hp-val">10</span>/<span id="hp-max-val">10</span></div>
                    <div class="essence-box">⭐ ESSÊNCIA: <span class="essence-count">0</span></div>
                </div>
                <div class="card-ui" style="text-align: right;">
                    <div style="font-size: 15px; letter-spacing: 1px; color: #38bdf8; text-shadow: 1px 1px #000;">✨ BÊNÇÃO [ESPAÇO]</div>
                    <div class="bar-bg"><div id="crucifix-bar"></div></div>
                </div>
            </div>
        </div>

        <div id="menu-screen" class="overlay">
            <h1 class="logo-title">PHASMOHOSPITAL</h1>
            <div class="subtitle">O hospital expandiu sua escuridão. Traga a luz!</div>
            <button onclick="startGame()">🎮 ADENTRAR O ASILO</button>
            <button onclick="showScreen('upgrades-screen')" class="btn-secondary">🔮 SANTUÁRIO DE BÊNÇÃOS</button>
        </div>

        <div id="death-screen" class="overlay hidden">
            <h1 class="logo-title" style="color: #ff3366; text-shadow: 4px 4px 0px #000;">POSSUÍDO!</h1>
            <p class="subtitle" style="color: #e2e8f0;">Sua luz divina se apagou no asilo.</p>
            <button onclick="startGame()">🔄 TENTAR NOVAMENTE</button>
            <button onclick="showScreen('menu-screen')" class="btn-secondary">🏠 VOLTAR AO MENU</button>
        </div>

        <div id="upgrades-screen" class="overlay hidden">
            <h1 class="logo-title" style="color: #c084fc;">BÊNÇÃOS E PODER</h1>
            <div class="essence-box" style="margin-bottom: 25px; font-size: 28px;">⭐ ESSÊNCIA: <span class="essence-count">0</span></div>
            <div class="upgrade-container">
                <button onclick="buyUpgrade('hp')" id="btn-up-hp"></button>
                <button onclick="buyUpgrade('speed')" id="btn-up-speed"></button>
                <button onclick="buyUpgrade('cooldown')" id="btn-up-cooldown"></button>
            </div>
            <button onclick="showScreen('menu-screen')" class="btn-secondary" style="margin-top: 25px; width: 220px;">VOLTAR</button>
        </div>

        <canvas id="gameCanvas"></canvas>
    </div>

<script>
/** @type {HTMLCanvasElement} */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

canvas.width = 1000; 
canvas.height = 700;

// REDUZIDA A VELOCIDADE DOS PERSONAGENS
// Velocidade inicial do jogador reduzida de 5.0/4.0 para 2.6
let stats = { totalEssence: 0, maxHp: 10, moveSpeed: 2.6, cooldownMod: 1, upgradesBought: { hp: 0, speed: 0, cooldown: 0 } };
const BASE_COSTS = { hp: 50, speed: 40, cooldown: 120 };
const GRAVITY = 0.45; // Física mais leve e suave

const FLOORS = [160, 320, 480, 640]; 

const BASE_COOLDOWN = 12 * 60;
let gameState = "MENU", score = 0, sessionEssence = 0, crucifixCooldown = 0, shake = 0, spawnTimer = 0;
const keys = {};

window.onkeydown = (e) => {
    let key = e.key.toLowerCase();
    if(key === 'arrowleft') key = 'a';
    if(key === 'arrowright') key = 'd';
    if(key === 'arrowup') key = 'w';
    if(key === 'arrowdown') key = 's';
    keys[key] = true;
};

window.onkeyup = (e) => {
    let key = e.key.toLowerCase();
    if(key === 'arrowleft') key = 'a';
    if(key === 'arrowright') key = 'd';
    if(key === 'arrowup') key = 'w';
    if(key === 'arrowdown') key = 's';
    keys[key] = false;
};

window.onmousedown = () => { if(gameState === "PLAYING" && player.ritualLock <= 0) player.shoot(); };

// --- DESENHO DIGITAL CARTUNESCO (VECTOR CARTOON DRAWING FUNCTIONS) ---

function drawCartoonyWindow(x, y, w, h) {
    ctx.save();
    ctx.fillStyle = "#ffb703";
    ctx.shadowBlur = 20; ctx.shadowColor = "#ffb703";
    ctx.beginPath();
    ctx.roundRect(x, y, w, h, 12);
    ctx.fill();
    ctx.shadowBlur = 0;

    // Moldura
    ctx.strokeStyle = "#4a154b";
    ctx.lineWidth = 5;
    ctx.stroke();
    
    // Grades da janela
    ctx.beginPath();
    ctx.moveTo(x + w/2, y); ctx.lineTo(x + w/2, y + h);
    ctx.moveTo(x, y + h/2); ctx.lineTo(x + w, y + h/2);
    ctx.stroke();
    ctx.restore();
}

function drawCartoonyProps(p) {
    ctx.save();
    if(p.type === "STRETCHER") {
        // Maca em Desenho Digital
        ctx.fillStyle = "#cbd5e1";
        ctx.beginPath(); ctx.roundRect(p.x, p.y - 18, 65, 10, 5); ctx.fill();
        ctx.strokeStyle = "#1e293b"; ctx.lineWidth = 3; ctx.stroke();

        // Almofada/Travesseiro
        ctx.fillStyle = "#ff4d8d";
        ctx.beginPath(); ctx.roundRect(p.x + 4, p.y - 23, 16, 7, 3); ctx.fill(); ctx.stroke();

        // Rodinhas e pernas
        ctx.strokeStyle = "#475569"; ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(p.x + 10, p.y - 8); ctx.lineTo(p.x + 10, p.y);
        ctx.moveTo(p.x + 55, p.y - 8); ctx.lineTo(p.x + 55, p.y);
        ctx.stroke();

        ctx.fillStyle = "#0f172a";
        ctx.beginPath(); ctx.arc(p.x + 10, p.y, 4, 0, Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(p.x + 55, p.y, 4, 0, Math.PI*2); ctx.fill();
    } else {
        // Suporte de Soro Digital
        ctx.strokeStyle = "#94a3b8"; ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(p.x, p.y - 50); ctx.lineTo(p.x, p.y);
        ctx.moveTo(p.x - 10, p.y - 46); ctx.lineTo(p.x + 10, p.y - 46);
        ctx.stroke();

        // Bolsa de soro sorridente
        ctx.fillStyle = "#38bdf8";
        ctx.beginPath(); ctx.roundRect(p.x - 8, p.y - 42, 16, 22, 6); ctx.fill();
        ctx.strokeStyle = "#0284c7"; ctx.lineWidth = 2.5; ctx.stroke();

        // Carinha fofa na bolsa
        ctx.fillStyle = "#0c4a6e";
        ctx.beginPath(); ctx.arc(p.x - 3, p.y - 32, 1.5, 0, Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(p.x + 3, p.y - 32, 1.5, 0, Math.PI*2); ctx.fill();
    }
    ctx.restore();
}

const props = []; 

function initHospital() {
    props.length = 0;
    FLOORS.forEach(fy => {
        for(let x = 90; x < 900; x += 220) {
            if(Math.random() > 0.35) props.push({x: x + Math.random()*20, y: fy, type: Math.random() > 0.5 ? "STRETCHER" : "IV_STAND"});
        }
    });
}

function drawHospital() {
    // Fundo estilizado com gradiente de ilustração digital
    let bgGrad = ctx.createLinearGradient(0, 0, 0, canvas.height);
    bgGrad.addColorStop(0, "#1f0938");
    bgGrad.addColorStop(0.5, "#2a1047");
    bgGrad.addColorStop(1, "#18062b");
    ctx.fillStyle = bgGrad;
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Janelas iluminadas ao fundo
    drawCartoonyWindow(120, 50, 50, 70);
    drawCartoonyWindow(820, 50, 50, 70);
    drawCartoonyWindow(480, 210, 50, 70);
    drawCartoonyWindow(150, 370, 50, 70);
    drawCartoonyWindow(800, 370, 50, 70);

    // Desenho dos Andares (Plataformas estilo Cartoon Digital)
    FLOORS.forEach((fy) => {
        // Sombra da plataforma
        ctx.fillStyle = "rgba(10, 3, 20, 0.6)";
        ctx.fillRect(0, fy + 18, 1000, 15);

        // Corpo principal da plataforma
        let floorGrad = ctx.createLinearGradient(0, fy, 0, fy + 18);
        floorGrad.addColorStop(0, "#a855f7");
        floorGrad.addColorStop(0.3, "#7e22ce");
        floorGrad.addColorStop(1, "#581c87");
        ctx.fillStyle = floorGrad;
        
        ctx.beginPath();
        ctx.roundRect(0, fy, 1000, 18, 4);
        ctx.fill();

        // Borda superior brilhante
        ctx.fillStyle = "#c084fc";
        ctx.fillRect(0, fy, 1000, 4);

        // Contorno cartunesco espesso
        ctx.strokeStyle = "#2e1065";
        ctx.lineWidth = 3.5;
        ctx.strokeRect(0, fy, 1000, 18);

        // Luminárias fofas de teto
        for(let lx = 180; lx < 1000; lx += 320) {
            let sky = fy - 120;
            // Brilho suave
            let radial = ctx.createRadialGradient(lx, sky + 15, 5, lx, sky + 15, 70);
            radial.addColorStop(0, "rgba(255, 230, 120, 0.35)");
            radial.addColorStop(1, "rgba(255, 230, 120, 0)");
            ctx.fillStyle = radial;
            ctx.beginPath(); ctx.arc(lx, sky + 15, 70, 0, Math.PI * 2); ctx.fill();

            // Cordinha e Lâmpada Cartoon
            ctx.strokeStyle = "#4c1d95"; ctx.lineWidth = 3;
            ctx.beginPath(); ctx.moveTo(lx, sky - 20); ctx.lineTo(lx, sky + 5); ctx.stroke();

            ctx.fillStyle = "#ffde59";
            ctx.beginPath(); ctx.arc(lx, sky + 10, 10, 0, Math.PI*2); ctx.fill();
            ctx.strokeStyle = "#b45309"; ctx.lineWidth = 2.5; ctx.stroke();
        }
    });

    // Adereços de Fundo
    props.forEach(p => drawCartoonyProps(p));
}

// --- CLASSE DE PARTÍCULAS EM DESENHO DIGITAL ---
class Particle {
    constructor(x, y, color) {
        this.x = x; this.y = y; this.color = color;
        this.vx = (Math.random() - 0.5) * 8;
        this.vy = (Math.random() - 0.5) * 8;
        this.gravity = 0.18; this.life = 1.0;
        this.size = Math.random() * 8 + 5;
    }
    update() { this.x += this.vx; this.y += this.vy; this.vy += this.gravity; this.life -= 0.03; }
    draw() { 
        ctx.save();
        ctx.globalAlpha = Math.max(0, this.life); 
        ctx.fillStyle = this.color; 
        ctx.strokeStyle = "#10002b";
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI*2);
        ctx.fill();
        ctx.stroke();
        ctx.restore();
    }
}

// --- PERSONAGEM EM ILUSTRAÇÃO DIGITAL (VECTOR CARTOON PLAYER) ---
class Player {
    constructor() { this.reset(); }
    reset() {
        this.x = 480; 
        this.y = 80;
        this.vx = 0; this.vy = 0;
        this.w = 46; this.h = 58; this.dir = 1;
        this.ritualLock = 0; this.hp = stats.maxHp; this.invul = 0;
        this.animTimer = 0;
    }
    update() {
        this.animTimer += 0.15;
        if (this.invul > 0) this.invul--;
        if (this.ritualLock > 0) { 
            this.ritualLock--; 
            this.vx = 0; 
            if(this.ritualLock === 40) purgeAll(); 
            return; 
        }
        
        // Movimento com velocidade reduzida e aceleração suave
        if (keys.a) { this.vx = -stats.moveSpeed; this.dir = -1; }
        else if (keys.d) { this.vx = stats.moveSpeed; this.dir = 1; }
        else { this.vx *= 0.72; }

        this.x += this.vx; 
        this.y += this.vy; 
        this.vy += GRAVITY;

        let grounded = false;
        let prevY = this.y - this.vy;

        FLOORS.forEach((fy, i) => {
            const isLast = (i === FLOORS.length - 1);
            // Verifica se o jogador estava acima da plataforma na frame anterior e pousou nela nesta frame
            if (prevY + this.h <= fy + 6 && this.y + this.h >= fy && this.vy >= 0) {
                if (keys.s && !isLast) {
                    // Permite atravessar a plataforma para baixo ao segurar S
                    this.y += 6;
                } else { 
                    this.y = fy - this.h; 
                    this.vy = 0; 
                    grounded = true; 
                }
            }
        });

        if (this.y + this.h >= canvas.height - 18) { 
            this.y = canvas.height - 18 - this.h; 
            this.vy = 0; 
            grounded = true; 
        }

        // Pulo com força ajustada para subir facilmente entre andares
        if (keys.w && grounded) {
            this.vy = -13.5;
        }
        this.x = Math.max(10, Math.min(canvas.width - this.w - 10, this.x));
    }
    draw() {
        if(this.invul % 4 > 2) return;
        
        ctx.save();
        ctx.translate(this.x + this.w/2, this.y + this.h/2);
        
        // Efeito de bobbing/respiração ao andar
        let bounce = Math.abs(this.vx) > 0.2 ? Math.sin(this.animTimer * 2) * 3 : Math.sin(this.animTimer) * 1.5;
        ctx.translate(0, bounce);

        if(this.dir === -1) ctx.scale(-1, 1);

        // --- DESENHO DO PADRE/HERÓI CARTUNESCO ---
        // Sombra nos pés
        ctx.fillStyle = "rgba(0,0,0,0.3)";
        ctx.beginPath(); ctx.ellipse(0, this.h/2 - 2, 18, 6, 0, 0, Math.PI*2); ctx.fill();

        // Manto/Roupa Roxa com Contorno Espesso
        ctx.fillStyle = "#3b0764";
        ctx.strokeStyle = "#120324";
        ctx.lineWidth = 3.5;
        ctx.beginPath();
        ctx.roundRect(-18, -10, 36, 36, [10, 10, 4, 4]);
        ctx.fill(); ctx.stroke();

        // Cachecol / Gola Branca
        ctx.fillStyle = "#f8fafc";
        ctx.beginPath();
        ctx.roundRect(-10, -12, 20, 8, 4);
        ctx.fill(); ctx.stroke();

        // Cabeça
        ctx.fillStyle = "#ffdbac";
        ctx.beginPath();
        ctx.arc(0, -22, 16, 0, Math.PI * 2);
        ctx.fill(); ctx.stroke();

        // Cabelo Estilizado
        ctx.fillStyle = "#451a03";
        ctx.beginPath();
        ctx.arc(0, -26, 16, Math.PI, Math.PI * 2);
        ctx.fill(); ctx.stroke();

        // Olhos Expressivos estilo Desenho
        ctx.fillStyle = "#0f172a";
        ctx.beginPath(); ctx.arc(4, -22, 3.5, 0, Math.PI*2); ctx.fill(); // Olho direito
        ctx.beginPath(); ctx.arc(-4, -22, 3.5, 0, Math.PI*2); ctx.fill(); // Olho esquerdo

        // Brilho nos olhos
        ctx.fillStyle = "#ffffff";
        ctx.beginPath(); ctx.arc(5, -23, 1.2, 0, Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(-3, -23, 1.2, 0, Math.PI*2); ctx.fill();

        // Mão segurando a Maçã Divina
        ctx.fillStyle = "#ffdbac";
        ctx.beginPath(); ctx.arc(16, -2, 6, 0, Math.PI*2); ctx.fill(); ctx.stroke();
        
        // Mini Maçã na Mão
        ctx.fillStyle = "#ff2a6d";
        ctx.beginPath(); ctx.arc(18, -4, 5, 0, Math.PI*2); ctx.fill();

        ctx.restore();

        // RITUAL DO CRUCIFIXO (DESENHO DIGITAL)
        if(this.ritualLock > 0) {
            ctx.save();
            ctx.shadowBlur = 25; ctx.shadowColor = "#38bdf8";
            ctx.strokeStyle = "#38bdf8"; ctx.lineWidth = 6;
            
            // Mandala/Feixes Divinos Ilustrados
            for(let i=0; i<8; i++) {
                ctx.beginPath(); 
                ctx.moveTo(this.x + this.w/2, this.y + this.h/2);
                let ang = i * (Math.PI / 4) + (this.ritualLock * 0.1);
                ctx.lineTo(this.x + this.w/2 + Math.cos(ang)*300, this.y + this.h/2 + Math.sin(ang)*300);
                ctx.stroke();
            }

            // Cruz Dourada Ilustrada flutuando
            let crossX = this.x + this.w/2 + (this.dir * 50);
            let crossY = this.y - 45;
            
            ctx.fillStyle = "#ffbe0b";
            ctx.strokeStyle = "#78350f";
            ctx.lineWidth = 4;
            
            // Vertical
            ctx.beginPath(); ctx.roundRect(crossX - 8, crossY - 25, 16, 50, 6); ctx.fill(); ctx.stroke();
            // Horizontal
            ctx.beginPath(); ctx.roundRect(crossX - 22, crossY - 15, 44, 16, 6); ctx.fill(); ctx.stroke();

            ctx.restore();
        }
    }
    shoot() { 
        apples.push(new CartoonyApple(this.x + this.w/2, this.y + 10, this.dir * 7.5)); 
    }
}

// --- MAÇÃ PROJÉTIL EM DESENHO DIGITAL ---
class CartoonyApple {
    constructor(x, y, vx) {
        this.x = x; this.y = y; this.vx = vx;
        this.angle = 0; this.rotSpeed = 0.2;
        this.radius = 12;
    }
    update() {
        this.x += this.vx;
        this.angle += this.rotSpeed;
    }
    draw() {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.rotate(this.angle);

        // Corpo da Maçã Cartoon
        ctx.fillStyle = "#ff2a6d";
        ctx.strokeStyle = "#800020";
        ctx.lineWidth = 3;
        ctx.beginPath();
        ctx.arc(0, 0, this.radius, 0, Math.PI * 2);
        ctx.fill(); ctx.stroke();

        // Brilho da Maçã
        ctx.fillStyle = "rgba(255,255,255,0.6)";
        ctx.beginPath(); ctx.arc(-4, -4, 4, 0, Math.PI*2); ctx.fill();

        // Cabinho e Folha
        ctx.strokeStyle = "#451a03"; ctx.lineWidth = 2.5;
        ctx.beginPath(); ctx.moveTo(0, -this.radius); ctx.lineTo(2, -this.radius - 5); ctx.stroke();
        
        ctx.fillStyle = "#10b981";
        ctx.beginPath(); ctx.ellipse(4, -this.radius - 4, 4, 2, Math.PI/4, 0, Math.PI*2); ctx.fill();

        ctx.restore();
    }
}

// --- ENFERMEIRA FANTASMA EM ILUSTRAÇÃO DIGITAL (VECTOR CARTOON ENEMY) ---
class Nurse {
    constructor() {
        this.w = 44; this.h = 56;
        this.floorY = FLOORS[Math.floor(Math.random() * FLOORS.length)];
        this.y = this.floorY - this.h;
        this.patrolDir = Math.random() > 0.5 ? 1 : -1;
        this.x = this.patrolDir === 1 ? -80 : 1050; 
        
        // REDUZIDA A VELOCIDADE DOS INIMIGOS
        // Velocidade reduzida de 2.0+ para 1.1 + escassos aumentos
        this.speed = 1.1 + (score * 0.03);
        this.hp = 2; this.hitTimer = 0; this.purging = 0;
        this.floatTimer = Math.random() * 100;
    }
    update() {
        this.floatTimer += 0.08;
        if(this.purging > 0) { this.purging--; return; }
        
        this.x += this.patrolDir * this.speed;
        if (this.x < -100) this.patrolDir = 1; 
        if (this.x > 1080) this.patrolDir = -1;
        if(this.hitTimer > 0) this.hitTimer--;
    }
    draw() {
        ctx.save();
        let floatY = Math.sin(this.floatTimer) * 6;
        ctx.translate(this.x + this.w/2, this.y + this.h/2 + floatY);

        if(this.patrolDir === -1) ctx.scale(-1, 1);

        let alpha = this.purging > 0 ? (this.purging / 40) : 0.9;
        ctx.globalAlpha = alpha;

        // Corpo Fantasma / Vestido de Enfermeira Cartoon
        ctx.fillStyle = this.hitTimer > 0 ? "#ffffff" : "#c084fc";
        ctx.strokeStyle = "#2e1065";
        ctx.lineWidth = 3.5;

        // Vestido de Fantasma Flutuante
        ctx.beginPath();
        ctx.moveTo(-16, -10);
        ctx.lineTo(16, -10);
        ctx.lineTo(20, 22);
        // Cauda ondulada de fantasma
        ctx.quadraticCurveTo(10, 28, 0, 22);
        ctx.quadraticCurveTo(-10, 28, -20, 22);
        ctx.closePath();
        ctx.fill(); ctx.stroke();

        // Cabeça Fantagórica
        ctx.fillStyle = this.hitTimer > 0 ? "#ffffff" : "#e9d5ff";
        ctx.beginPath(); ctx.arc(0, -20, 15, 0, Math.PI * 2); ctx.fill(); ctx.stroke();

        // Chapéu de Enfermeira com Cruz Vermelha
        ctx.fillStyle = "#ffffff";
        ctx.beginPath(); ctx.roundRect(-10, -34, 20, 10, 3); ctx.fill(); ctx.stroke();
        
        ctx.fillStyle = "#ff2a6d";
        ctx.fillRect(-2, -32, 4, 6);
        ctx.fillRect(-4, -30, 8, 2);

        // Olhos Brilhantes Assustadores / Fofos
        ctx.fillStyle = "#ff0055";
        ctx.beginPath(); ctx.arc(5, -20, 3, 0, Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(-5, -20, 3, 0, Math.PI*2); ctx.fill();

        ctx.restore();
    }
}

const player = new Player();
let enemies = [], apples = [], particles = [];

function purgeAll() {
    shake = 30;
    enemies.forEach(e => { 
        if(e.x > -50 && e.x < 1050) { 
            e.purging = 40; 
            score++; 
            sessionEssence += 10; 
        } 
    });
}

function updateUpgradeUI() {
    document.querySelectorAll('.essence-count').forEach(el => el.innerText = stats.totalEssence);
    const costHp = BASE_COSTS.hp + (stats.upgradesBought.hp * 80);
    const costSpeed = BASE_COSTS.speed + (stats.upgradesBought.speed * 60);
    const costCD = BASE_COSTS.cooldown + (stats.upgradesBought.cooldown * 200);
    document.getElementById('btn-up-hp').innerText = `❤️ RESILIÊNCIA (+2 HP) | CUSTO: ${costHp}`;
    document.getElementById('btn-up-speed').innerText = `⚡ PASSOS LUMINOSOS (+0.4) | CUSTO: ${costSpeed}`;
    document.getElementById('btn-up-cooldown').innerText = `✨ RECARGA DIVINA | CUSTO: ${costCD}`;
}

function buyUpgrade(type) {
    let cost = (type === 'hp') ? BASE_COSTS.hp + (stats.upgradesBought.hp * 80) :
               (type === 'speed') ? BASE_COSTS.speed + (stats.upgradesBought.speed * 60) :
               BASE_COSTS.cooldown + (stats.upgradesBought.cooldown * 200);
    if(stats.totalEssence >= cost) {
        stats.totalEssence -= cost;
        stats.upgradesBought[type]++;
        if(type === 'hp') stats.maxHp += 2;
        if(type === 'speed') stats.moveSpeed += 0.4;
        if(type === 'cooldown') stats.cooldownMod *= 0.8;
        updateUpgradeUI();
    }
}

function showScreen(id) {
    document.querySelectorAll('.overlay').forEach(s => s.classList.add('hidden'));
    document.getElementById(id).classList.remove('hidden');
    updateUpgradeUI();
}

function startGame() {
    score = 0; sessionEssence = 0; enemies = []; apples = []; particles = [];
    player.reset(); 
    initHospital();
    document.getElementById('ui-ingame').classList.remove('hidden');
    document.querySelectorAll('.overlay').forEach(s => s.classList.add('hidden'));
    gameState = "PLAYING";
}

function gameOver() {
    gameState = "DEATH"; 
    stats.totalEssence += sessionEssence; 
    document.getElementById('ui-ingame').classList.add('hidden');
    showScreen('death-screen');
}

function gameLoop() {
    ctx.save();
    if(shake > 0) { 
        ctx.translate((Math.random()-0.5)*shake, (Math.random()-0.5)*shake); 
        shake *= 0.88; 
    }
    
    drawHospital();

    if (gameState === "PLAYING") {
        // Spawns compassados para acompanhar o ritmo mais tranquilo do jogo
        if(++spawnTimer > 90 && enemies.length < 10) { 
            enemies.push(new Nurse()); 
            spawnTimer = 0; 
        }

        // Atualização dos projéteis (Maçãs)
        apples = apples.filter(a => {
            a.update();
            a.draw();
            
            let hit = false;
            enemies.forEach(e => {
                if(e.purging <= 0 && Math.abs(a.x - (e.x + e.w/2)) < 35 && Math.abs(a.y - (e.y + e.h/2)) < 35) {
                    hit = true; e.hp--; e.hitTimer = 8;
                    for(let i=0; i<10; i++) particles.push(new Particle(a.x, a.y, "#ff2a6d"));
                    shake = 6;
                }
            });
            return !hit && a.x > -50 && a.x < 1050;
        });

        // Atualização dos inimigos (Enfermeiras)
        enemies = enemies.filter(e => {
            e.update(); 
            e.draw();
            
            if(e.purging === 1) return false;
            if(e.hp <= 0 && e.purging <= 0) { 
                score++; sessionEssence += 5; 
                for(let i=0; i<16; i++) particles.push(new Particle(e.x + e.w/2, e.y + e.h/2, "#ffc107"));
                return false; 
            }
            
            // Colisão com o Jogador
            if(player.invul <= 0 && e.purging <= 0 && 
               Math.abs((player.x + player.w/2) - (e.x + e.w/2)) < 32 && 
               Math.abs((player.y + player.h/2) - (e.y + e.h/2)) < 36) {
                player.hp--; player.invul = 55; shake = 18;
                if(player.hp <= 0) gameOver();
            }
            return true;
        });

        particles = particles.filter(p => { p.update(); p.draw(); return p.life > 0; });
        
        player.update(); 
        player.draw();

        // Recarga da Bênção / Crucifixo
        if(crucifixCooldown > 0) crucifixCooldown--;
        else if(keys[' ']) { 
            crucifixCooldown = BASE_COOLDOWN * stats.cooldownMod; 
            player.ritualLock = 90; 
        }

        document.getElementById('hp-val').innerText = player.hp;
        document.getElementById('hp-max-val').innerText = stats.maxHp;
        document.querySelector('.essence-count').innerText = sessionEssence;
        document.getElementById('crucifix-bar').style.width = Math.min(100, (1 - crucifixCooldown/(BASE_COOLDOWN * stats.cooldownMod))*100) + "%";
    }

    ctx.restore();
    requestAnimationFrame(gameLoop);
}

// Iniciar o loop de renderização do jogo
gameLoop();
</script>
</body>
</html>
