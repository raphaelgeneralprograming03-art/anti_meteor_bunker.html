
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PLANETARY_DEFENSE_MATRIX_v4 - Anti-Meteor & Asteroid Simulation</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }
        body {
            background-color: #03060a;
            color: #ffaa00;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 10px;
        }
        .container {
            width: 100%;
            max-width: 1100px;
            background: #080f1a;
            border: 2px solid #ffaa00;
            box-shadow: 0 0 30px rgba(255, 170, 0, 0.25);
            border-radius: 8px;
            overflow: hidden;
        }
        .header {
            background: #040810;
            padding: 14px;
            text-align: center;
            border-bottom: 2px solid #ffaa00;
            font-weight: bold;
            font-size: 1.1rem;
            letter-spacing: 2px;
            color: #ffaa00;
            text-shadow: 0 0 10px rgba(255, 170, 0, 0.6);
        }
        .hud-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
            gap: 8px;
            padding: 10px;
            background: #060c16;
            border-bottom: 1px solid #1a2a40;
        }
        .hud-card {
            background: #03070f;
            border: 1px solid #ffaa0044;
            padding: 8px;
            border-radius: 4px;
        }
        .hud-title {
            font-size: 0.68rem;
            color: #8aa0be;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        .hud-value {
            font-size: 0.95rem;
            font-weight: bold;
            color: #ffffff;
            margin-top: 3px;
        }
        .hud-value.alert { color: #ff2255; text-shadow: 0 0 6px #ff2255; }
        .hud-value.active { color: #00d2ff; text-shadow: 0 0 6px #00d2ff; }
        .hud-value.amber { color: #ffaa00; text-shadow: 0 0 6px #ffaa00; }

        canvas {
            display: block;
            width: 100%;
            height: 500px;
            background-color: #020408;
            cursor: crosshair;
        }

        .controls {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            padding: 12px;
            background: #050a12;
            justify-content: center;
            align-items: center;
            border-top: 1px solid #1a2a40;
        }

        button {
            background: #0f1c2e;
            color: #ffaa00;
            border: 1px solid #ffaa00;
            padding: 9px 15px;
            font-family: inherit;
            font-size: 0.80rem;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
            transition: all 0.2s ease;
            text-shadow: 0 0 4px #ffaa00;
        }

        button:hover {
            background: #ffaa00;
            color: #03060a;
            box-shadow: 0 0 15px #ffaa00;
        }

        button.btn-cyan {
            color: #00d2ff;
            border-color: #00d2ff;
            text-shadow: 0 0 4px #00d2ff;
        }
        button.btn-cyan:hover {
            background: #00d2ff;
            color: #000;
            box-shadow: 0 0 15px #00d2ff;
        }

        button.btn-danger {
            color: #ff2255;
            border-color: #ff2255;
            text-shadow: 0 0 4px #ff2255;
        }
        button.btn-danger:hover {
            background: #ff2255;
            color: #000;
            box-shadow: 0 0 15px #ff2255;
        }

        .terminal {
            background: #020407;
            padding: 8px 12px;
            font-family: 'Courier New', Courier, monospace;
            font-size: 0.75rem;
            color: #ffaa00aa;
            height: 65px;
            overflow-y: auto;
            border-top: 1px solid #1a2a40;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        ORBITAL DEFENSE MATRIX v4 // ANTI-METEOR & CELESTIAL THREAT CONTROL
    </div>

    <!-- HUD PAINEL DE MONITORAMENTO -->
    <div class="hud-grid">
        <div class="hud-card">
            <div class="hud-title">AMEAÇA DETECTADA</div>
            <div class="hud-value amber" id="hud-threat-type">NENHUMA</div>
        </div>
        <div class="hud-card">
            <div class="hud-title">DISTÂNCIA DA TERRA</div>
            <div class="hud-value" id="hud-distance">0 km</div>
        </div>
        <div class="hud-card">
            <div class="hud-title">VELOCIDADE ORBITAL</div>
            <div class="hud-value" id="hud-speed">0.0 km/s</div>
        </div>
        <div class="hud-card">
            <div class="hud-title">ENERGIA DE IMPACTO</div>
            <div class="hud-value" id="hud-energy">0 MT</div>
        </div>
        <div class="hud-card">
            <div class="hud-title">CANHÃO LASER ORBITAL</div>
            <div class="hud-value active" id="hud-laser-status">PRONTO</div>
        </div>
        <div class="hud-card">
            <div class="hud-title">ESCUDO DO BUNKER</div>
            <div class="hud-value active" id="hud-shield">100%</div>
        </div>
    </div>

    <!-- TELA DA SIMULAÇÃO (CANVAS 2D) -->
    <canvas id="simCanvas" width="1050" height="500"></canvas>

    <!-- CONTROLES -->
    <div class="controls">
        <button id="btn-toggle">⏸️ PAUSAR SIMULAÇÃO</button>
        <button id="btn-reset">🔄 REINICIAR SISTEMA</button>
        <button id="btn-laser" class="btn-cyan">⚡ ATIVAR LASER ORBITAL DE GRAVIDADE</button>
        <button id="btn-spawn-meteor" class="btn-danger">☄️ LANÇAR METEORO (RÁPIDO)</button>
        <button id="btn-spawn-asteroid" class="btn-danger">🪨 LANÇAR ASTEROIDE (DENSE)</button>
        <button id="btn-spawn-comet" class="btn-danger">🌟 LANÇAR COMETA (GELADO)</button>
    </div>

    <!-- TERMINAL DE REGISTRO -->
    <div class="terminal" id="terminal-log">
        [SROS MATRIX v4] Varredura de radar orbital ativa. Terreno isolado carregado via satélite.
    </div>
</div>

<script>
    const canvas = document.getElementById('simCanvas');
    const ctx = canvas.getContext('2d');

    // Estados da Simulação
    let isRunning = true;
    let laserActive = false;
    let bunkerShield = 100;

    // Ameaça Atual
    let threat = null; // { type, x, y, vx, vy, radius, hp, maxHp, speed, mass }

    // Interceptadores e Efeitos
    let interceptors = [];
    let explosions = [];
    let particles = [];
    let laserTarget = null;

    // Coordenadas
    const GROUND_Y = 360;
    const BUNKER_X = 260;
    const BUNKER_Y = 410;

    function log(msg) {
        const term = document.getElementById('terminal-log');
        term.innerHTML = `> ${msg}<br>` + term.innerHTML;
    }

    function updateHUD() {
        if (threat && threat.hp > 0) {
            document.getElementById('hud-threat-type').innerText = threat.type.toUpperCase();
            document.getElementById('hud-threat-type').className = "hud-value alert";

            const dist = Math.max(0, Math.round((GROUND_Y - threat.y) * 45));
            document.getElementById('hud-distance').innerText = `${dist} km`;

            document.getElementById('hud-speed').innerText = `${threat.speed.toFixed(1)} km/s`;

            const energy = Math.round((threat.mass * threat.speed * threat.speed) / 100);
            document.getElementById('hud-energy').innerText = `${energy} Megatons`;
        } else {
            document.getElementById('hud-threat-type').innerText = "NENHUMA";
            document.getElementById('hud-threat-type').className = "hud-value amber";
            document.getElementById('hud-distance').innerText = "0 km";
            document.getElementById('hud-speed').innerText = "0.0 km/s";
            document.getElementById('hud-energy').innerText = "0 MT";
        }

        const laserElem = document.getElementById('hud-laser-status');
        if (laserActive) {
            laserElem.innerText = "DISPARANDO (100% CAP)";
            laserElem.className = "hud-value active";
        } else {
            laserElem.innerText = "PRONTO";
            laserElem.className = "hud-value amber";
        }

        const shieldElem = document.getElementById('hud-shield');
        shieldElem.innerText = `${Math.round(bunkerShield)}%`;
        shieldElem.className = bunkerShield > 50 ? "hud-value active" : "hud-value alert";
    }

    // Criar Ameaças Espaciais
    function spawnThreat(type) {
        let radius = 20;
        let hp = 100;
        let speed = 4.0;
        let mass = 50;

        if (type === 'Meteoro') {
            radius = 16; hp = 80; speed = 6.5; mass = 30;
        } else if (type === 'Asteroide') {
            radius = 28; hp = 220; speed = 3.2; mass = 120;
        } else if (type === 'Cometa') {
            radius = 22; hp = 110; speed = 7.8; mass = 60;
        }

        const startX = 800 + Math.random() * 150;
        const startY = -30;
        const targetX = BUNKER_X + (Math.random() - 0.5) * 80;
        const targetY = GROUND_Y;

        const dx = targetX - startX;
        const dy = targetY - startY;
        const dist = Math.hypot(dx, dy);

        threat = {
            type: type,
            x: startX,
            y: startY,
            vx: (dx / dist) * speed,
            vy: (dy / dist) * speed,
            radius: radius,
            hp: hp,
            maxHp: hp,
            speed: speed,
            mass: mass,
            angle: 0
        };

        log(`⚠️ ALERTA SATÉLITE: ${type.toUpperCase()} detectado em rota de colisão planetária!`);
    }

    // Eventos
    document.getElementById('btn-toggle').addEventListener('click', (e) => {
        isRunning = !isRunning;
        e.target.innerText = isRunning ? "⏸️ PAUSAR SIMULAÇÃO" : "▶️ INICIAR SIMULAÇÃO";
    });

    document.getElementById('btn-reset').addEventListener('click', () => {
        threat = null;
        interceptors = [];
        explosions = [];
        particles = [];
        bunkerShield = 100;
        laserActive = false;
        updateHUD();
        log("Sistema reiniciado. Todos os radares orbitais recalibrados.");
    });

    document.getElementById('btn-laser').addEventListener('click', () => {
        laserActive = !laserActive;
        log(laserActive ? "⚡ CANHÃO LASER ORBITAL FOCADO E ATIVADO!" : "Canhão Laser desativado.");
    });

    document.getElementById('btn-spawn-meteor').addEventListener('click', () => spawnThreat('Meteoro'));
    document.getElementById('btn-spawn-asteroid').addEventListener('click', () => spawnThreat('Asteroide'));
    document.getElementById('btn-spawn-comet').addEventListener('click', () => spawnThreat('Cometa'));

    // Clique na tela para disparar Interceptador Cinético
    canvas.addEventListener('click', (e) => {
        const rect = canvas.getBoundingClientRect();
        const clickX = e.clientX - rect.left;
        const clickY = e.clientY - rect.top;

        if (clickY < GROUND_Y) {
            const startX = BUNKER_X + 35;
            const startY = GROUND_Y - 15;
            const dx = clickX - startX;
            const dy = clickY - startY;
            const dist = Math.hypot(dx, dy);
            const speed = 9.0;

            interceptors.push({
                x: startX,
                y: startY,
                vx: (dx / dist) * speed,
                vy: (dy / dist) * speed,
                targetX: clickX,
                targetY: clickY
            });

            log(`🚀 Impactador Cinético disparado para as coordenadas (${Math.round(clickX)}, ${Math.round(clickY)}).`);
        }
    });

    // Atualização da Física
    function update() {
        if (!isRunning) return;

        // Atualizar Ameaça
        if (threat && threat.hp > 0) {
            threat.x += threat.vx;
            threat.y += threat.vy;
            threat.angle += 0.03;

            // Rastro de poeira/fogo
            if (Math.random() < 0.6) {
                particles.push({
                    x: threat.x - threat.vx * 2 + (Math.random() - 0.5) * 10,
                    y: threat.y - threat.vy * 2 + (Math.random() - 0.5) * 10,
                    radius: Math.random() * 4 + 1,
                    alpha: 1.0,
                    color: threat.type === 'Cometa' ? '#00d2ff' : '#ffaa00'
                });
            }

            // Dano por Laser
            if (laserActive) {
                threat.hp -= 1.8;
                if (Math.random() < 0.4) {
                    particles.push({
                        x: threat.x + (Math.random() - 0.5) * threat.radius,
                        y: threat.y + (Math.random() - 0.5) * threat.radius,
                        radius: Math.random() * 3,
                        alpha: 1.0,
                        color: '#ffffff'
                    });
                }
                if (threat.hp <= 0) {
                    triggerExplosion(threat.x, threat.y, threat.radius * 2, true);
                    log(`💥 ${threat.type.toUpperCase()} VAPORIZADO COM SUCESSO PELO LASER ORBITAL!`);
                    threat = null;
                }
            }

            // Checar impacto no solo/bunker
            if (threat && threat.y >= GROUND_Y - 10) {
                triggerExplosion(threat.x, GROUND_Y, threat.radius * 3, false);
                let damage = Math.round(threat.mass * 0.6);
                bunkerShield = Math.max(0, bunkerShield - damage);
                log(`⚠️ IMPACTO CATASTRÓFICO! ${threat.type.toUpperCase()} atingiu a zona do Bunker (-${damage}% Escudo).`);
                threat = null;
            }
        }

        // Atualizar Interceptadores Cinéticos
        for (let i = interceptors.length - 1; i >= 0; i--) {
            let m = interceptors[i];
            m.x += m.vx;
            m.y += m.vy;

            // Colisão com Ameaça
            if (threat && threat.hp > 0) {
                const dist = Math.hypot(m.x - threat.x, m.y - threat.y);
                if (dist < threat.radius + 10) {
                    threat.hp -= 45;
                    interceptors.splice(i, 1);
                    triggerExplosion(m.x, m.y, 25, true);
                    log(`🎯 IMPACTADOR CINÉTICO Atingiu o ${threat.type}! Dano infligido.`);
                    if (threat.hp <= 0) {
                        log(`💥 ${threat.type.toUpperCase()} DESTRUÍDO NO ESPAÇO/ATMOSFERA!`);
                        threat = null;
                    }
                    continue;
                }
            }

            // Fim da Trajetória
            if (Math.hypot(m.x - m.targetX, m.y - m.targetY) < 10 || m.y < 0) {
                interceptors.splice(i, 1);
            }
        }

        // Atualizar Partículas
        particles.forEach((p, idx) => {
            p.alpha -= 0.02;
            if (p.alpha <= 0) particles.splice(idx, 1);
        });

        // Atualizar Explosões
        explosions.forEach((exp, idx) => {
            exp.radius += 1.5;
            exp.alpha -= 0.02;
            if (exp.alpha <= 0) explosions.splice(idx, 1);
        });

        updateHUD();
    }

    function triggerExplosion(x, y, maxR, isSpace) {
        explosions.push({ x: x, y: y, radius: 5, maxRadius: maxR, alpha: 1.0, isSpace: isSpace });
    }

    // Desenho na Tela (Aparência estilo Satélite Orbital)
    function draw() {
        // Fundo Espaço Profundo
        ctx.fillStyle = '#02050a';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Renderizar estrelas distantes
        ctx.fillStyle = '#ffffff';
        for (let i = 0; i < 30; i++) {
            let sx = (i * 37) % canvas.width;
            let sy = (i * 23) % (GROUND_Y - 50);
            ctx.fillRect(sx, sy, 1.5, 1.5);
        }

        // Grade de Rastreamento de Satélite
        ctx.strokeStyle = '#ffaa0015';
        ctx.lineWidth = 1;
        for (let x = 0; x < canvas.width; x += 60) {
            ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, canvas.height); ctx.stroke();
        }
        for (let y = 0; y < canvas.height; y += 60) {
            ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
        }

        // TEXTURA DE SUPERFÍCIE DE SATÉLITE (TERRENO PLANO / CRATERAS DE OBSERVAÇÃO)
        ctx.fillStyle = '#0b1626';
        ctx.fillRect(0, GROUND_Y, canvas.width, canvas.height - GROUND_Y);

        // Crateras decorativas no terreno estilo mapa de satélite
        ctx.strokeStyle = '#182b45';
        ctx.lineWidth = 2;
        ctx.beginPath(); ctx.arc(100, GROUND_Y + 50, 35, 0, Math.PI*2); ctx.stroke();
        ctx.beginPath(); ctx.arc(600, GROUND_Y + 70, 45, 0, Math.PI*2); ctx.stroke();
        ctx.beginPath(); ctx.arc(900, GROUND_Y + 40, 25, 0, Math.PI*2); ctx.stroke();

        // Linha de Atmosfera/Horizonte
        ctx.strokeStyle = '#00d2ff88';
        ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(0, GROUND_Y); ctx.lineTo(canvas.width, GROUND_Y); ctx.stroke();

        // CANHÃO LASER ORBITAL (SATELLITE ARRAY)
        ctx.fillStyle = '#0f243a';
        ctx.fillRect(50, 20, 100, 25);
        ctx.strokeStyle = '#00d2ff';
        ctx.strokeRect(50, 20, 100, 25);
        ctx.fillStyle = '#00d2ff';
        ctx.font = '10px Segoe UI';
        ctx.fillText("SATÉLITE DEFESA", 58, 36);

        // Desenhar Feixe Laser se ativo e houver alvo
        if (laserActive && threat) {
            ctx.save();
            ctx.strokeStyle = '#00d2ff';
            ctx.lineWidth = 4;
            ctx.shadowBlur = 15;
            ctx.shadowColor = '#00d2ff';
            ctx.beginPath();
            ctx.moveTo(100, 45);
            ctx.lineTo(threat.x, threat.y);
            ctx.stroke();
            ctx.restore();
        }

        // ESTRUTURA DO BUNKER SUBTERRÂNEO
        ctx.fillStyle = '#102033';
        ctx.fillRect(BUNKER_X - 50, BUNKER_Y - 20, 100, 70);
        ctx.strokeStyle = '#ffaa00';
        ctx.strokeRect(BUNKER_X - 50, BUNKER_Y - 20, 100, 70);
        ctx.fillStyle = '#ffaa00';
        ctx.fillText("BUNKER NÚCLEO", BUNKER_X - 42, BUNKER_Y + 15);

        // Silo do Interceptador
        ctx.fillStyle = '#ffaa00';
        ctx.fillRect(BUNKER_X + 30, GROUND_Y - 8, 15, 8);

        // ESCUDO DE ENERGIA DO BUNKER
        if (bunkerShield > 0) {
            ctx.save();
            ctx.strokeStyle = bunkerShield > 40 ? 'rgba(0, 210, 255, 0.4)' : 'rgba(255, 34, 85, 0.5)';
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.arc(BUNKER_X, BUNKER_Y, 80, Math.PI, 0);
            ctx.stroke();
            ctx.restore();
        }

        // DESENHAR CORPOS CELESTES (AMEAÇA)
        if (threat && threat.hp > 0) {
            ctx.save();
            ctx.translate(threat.x, threat.y);
            ctx.rotate(threat.angle);

            if (threat.type === 'Meteoro') {
                ctx.fillStyle = '#ff5500';
                ctx.beginPath(); ctx.arc(0, 0, threat.radius, 0, Math.PI * 2); ctx.fill();
            } else if (threat.type === 'Asteroide') {
                ctx.fillStyle = '#6e7f96';
                ctx.beginPath();
                ctx.moveTo(-threat.radius, -threat.radius/2);
                ctx.lineTo(0, -threat.radius);
                ctx.lineTo(threat.radius, -threat.radius/2);
                ctx.lineTo(threat.radius/2, threat.radius);
                ctx.lineTo(-threat.radius/2, threat.radius);
                ctx.closePath();
                ctx.fill();
                ctx.strokeStyle = '#ffaa00';
                ctx.stroke();
            } else if (threat.type === 'Cometa') {
                ctx.fillStyle = '#88f0ff';
                ctx.beginPath(); ctx.arc(0, 0, threat.radius, 0, Math.PI * 2); ctx.fill();
            }

            ctx.restore();

            // Barra de HP da Ameaça
            ctx.fillStyle = '#111';
            ctx.fillRect(threat.x - 20, threat.y - threat.radius - 12, 40, 5);
            ctx.fillStyle = '#00d2ff';
            ctx.fillRect(threat.x - 20, threat.y - threat.radius - 12, (threat.hp / threat.maxHp) * 40, 5);
        }

        // Desenhar Interceptadores Cinéticos
        interceptors.forEach(m => {
            ctx.strokeStyle = '#ffaa00';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(m.x - m.vx*2, m.y - m.vy*2);
            ctx.lineTo(m.x, m.y);
            ctx.stroke();

            ctx.fillStyle = '#ffffff';
            ctx.beginPath(); ctx.arc(m.x, m.y, 3, 0, Math.PI*2); ctx.fill();
        });

        // Desenhar Partículas
        particles.forEach(p => {
            ctx.fillStyle = p.color;
            ctx.globalAlpha = p.alpha;
            ctx.beginPath(); ctx.arc(p.x, p.y, p.radius, 0, Math.PI*2); ctx.fill();
            ctx.globalAlpha = 1.0;
        });

        // Desenhar Explosões
        explosions.forEach(exp => {
            ctx.save();
            ctx.beginPath();
            ctx.arc(exp.x, exp.y, exp.radius, 0, Math.PI * 2);
            ctx.fillStyle = exp.isSpace ? `rgba(0, 210, 255, ${exp.alpha})` : `rgba(255, 170, 0, ${exp.alpha})`;
            ctx.fill();
            ctx.restore();
        });
    }

    // Loop
    function loop() {
        update();
        draw();
        requestAnimationFrame(loop);
    }

    loop();
</script>
</body>
</html>
