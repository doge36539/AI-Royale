<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Clash Royale: Full Arena</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Lilita+One&family=Roboto:wght@700&display=swap');

        :root {
            --grass-light: #6bc234;
            --grass-dark: #62b32e;
            --river: #3aa6e8;
            --bridge: #d98e36;
            --bg-color: #1a1a1a;
        }

        body {
            background-color: var(--bg-color);
            margin: 0;
            padding: 0;
            height: 100vh;
            width: 100vw;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            font-family: 'Roboto', sans-serif;
            user-select: none;
        }

        /* Main Container: Stacked Vertically */
        #app-container {
            width: 100%;
            height: 100%;
            max-width: 500px; /* Phone width limit */
            display: flex;
            flex-direction: column;
            background: #000;
            box-shadow: 0 0 40px rgba(0,0,0,0.8);
        }

        /* 1. The Arena (Top) */
        #arena-wrapper {
            flex: 1; /* Takes up remaining space */
            position: relative;
            background: var(--grass-light);
            overflow: hidden;
            cursor: crosshair;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            /* Pixel art style scaling */
            image-rendering: pixelated; 
        }

        /* 2. The UI Panel (Bottom - Extended Area) */
        #ui-panel {
            height: 220px; /* Fixed height for controls */
            background: #2b2b2b;
            border-top: 4px solid #111;
            display: flex;
            flex-direction: column;
            padding: 10px;
            box-sizing: border-box;
            position: relative;
            z-index: 10;
        }

        /* Elixir Bar */
        #elixir-wrapper {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
            color: white;
            font-family: 'Lilita One', sans-serif;
        }
        
        #elixir-bar-bg {
            flex: 1;
            height: 20px;
            background: #111;
            border-radius: 10px;
            margin-left: 10px;
            position: relative;
            overflow: hidden;
            border: 2px solid #444;
        }

        #elixir-fill {
            height: 100%;
            width: 0%;
            background: linear-gradient(to bottom, #d536d8, #b026b3);
            transition: width 0.1s linear;
        }

        #elixir-count {
            position: absolute;
            right: 10px;
            top: 50%;
            transform: translateY(-50%);
            font-size: 14px;
            z-index: 2;
            text-shadow: 1px 1px 0 #000;
        }

        /* Card Hand */
        #hand {
            display: flex;
            justify-content: space-between;
            height: 100%;
            gap: 8px;
        }

        .card-slot {
            flex: 1;
            background: #3a3a3a;
            border-radius: 8px;
            position: relative;
        }

        .card {
            width: 100%;
            height: 100%;
            background: #fff;
            border-radius: 6px;
            border-bottom: 4px solid #999;
            cursor: pointer;
            transition: transform 0.1s, border-color 0.1s;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            position: relative;
            box-sizing: border-box;
        }

        .card.selected {
            transform: translateY(-15px);
            border: 3px solid #ffff00;
            box-shadow: 0 0 15px #ffff00;
            z-index: 100;
        }

        .card.loading {
            transform: scale(0);
            transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        .card-cost {
            position: absolute;
            top: -5px; left: -5px;
            width: 20px; height: 20px;
            background: #fff;
            border: 2px solid #d536d8;
            color: #d536d8;
            border-radius: 50%;
            font-size: 12px;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .card-icon { font-size: 32px; }
        .card-name { font-size: 10px; font-weight: bold; margin-top: 4px; text-align: center; }

        /* Next Card Preview */
        #next-card-wrapper {
            position: absolute;
            top: -40px;
            left: 10px;
            background: #222;
            padding: 5px;
            border-radius: 6px;
            border: 2px solid #444;
            color: #888;
            font-size: 10px;
            text-align: center;
        }
        #next-card-icon { font-size: 16px; display: block; margin-top: 2px; }

        /* Popups */
        .floater {
            position: absolute;
            font-weight: bold;
            pointer-events: none;
            animation: floatUp 1s forwards;
            text-shadow: 1px 1px 0 #000;
        }
        
        @keyframes floatUp {
            0% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(-50px); opacity: 0; }
        }

        #game-over {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            display: none;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            color: white;
            z-index: 999;
        }
    </style>
</head>
<body>

<div id="app-container">
    <div id="arena-wrapper">
        <canvas id="gameCanvas"></canvas>
        <div id="game-over">
            <h1 id="winner-msg">GAME OVER</h1>
            <button onclick="location.reload()" style="padding:10px 20px; font-size:18px; cursor:pointer;">Play Again</button>
        </div>
    </div>

    <div id="ui-panel">
        <div id="next-card-wrapper">
            NEXT
            <span id="next-card-icon">?</span>
        </div>

        <div id="elixir-wrapper">
            <span style="color:#d536d8">ELIXIR</span>
            <div id="elixir-bar-bg">
                <div id="elixir-fill"></div>
                <div id="elixir-count">5</div>
            </div>
        </div>

        <div id="hand">
            <div class="card-slot" id="slot-0"></div>
            <div class="card-slot" id="slot-1"></div>
            <div class="card-slot" id="slot-2"></div>
            <div class="card-slot" id="slot-3"></div>
        </div>
    </div>
</div>

<script>
    /**
     * CLASH ROYALE: EXTENDED UI EDITION
     * - Fixed "undefined textures" by using robust drawing paths.
     * - UI is separate (bottom div) so it never covers the game.
     * - Elixir is 2x faster.
     * - 0.3s Delay added to card cycling.
     */

    const CONFIG = {
        FPS: 60,
        ELIXIR_MAX: 10,
        ELIXIR_SPEED: 0.8, // Seconds per 1 elixir (Faster than normal 2.8s)
        CYCLE_DELAY: 300, // ms
        ARENA_W: 720,
        ARENA_H: 1280, // High res internal logic
        BRIDGE_Y: 640
    };

    const TEAMS = { BLUE: 0, RED: 1 };

    // --- CARDS DATABASE ---
    const CARDS = [
        { name: 'Knight', emoji: '⚔️', cost: 3, hp: 1400, dmg: 150, speed: 3, range: 60, type: 'troop', count: 1, hitSpeed: 60 },
        { name: 'Archers', emoji: '🏹', cost: 3, hp: 300, dmg: 90, speed: 4, range: 300, type: 'troop', count: 2, projectile: true, hitSpeed: 50 },
        { name: 'Giant', emoji: '🧔', cost: 5, hp: 3500, dmg: 200, speed: 2, range: 80, type: 'troop', count: 1, target: 'building', hitSpeed: 90 },
        { name: 'Pekka', emoji: '🤖', cost: 4, hp: 1100, dmg: 500, speed: 5, range: 60, type: 'troop', count: 1, hitSpeed: 80 },
        { name: 'Skeletons', emoji: '💀', cost: 1, hp: 60, dmg: 60, speed: 5, range: 40, type: 'troop', count: 3, hitSpeed: 40 },
        { name: 'Bomber', emoji: '💣', cost: 2, hp: 250, dmg: 200, speed: 4, range: 250, type: 'troop', count: 1, area: 100, projectile: true, hitSpeed: 70 },
        { name: 'Fireball', emoji: '🔥', cost: 4, dmg: 500, type: 'spell', radius: 200 },
        { name: 'Zap', emoji: '⚡', cost: 2, dmg: 160, type: 'spell', radius: 150 }
    ];

    // --- GAME STATE ---
    const game = {
        canvas: document.getElementById('gameCanvas'),
        ctx: document.getElementById('gameCanvas').getContext('2d'),
        
        width: CONFIG.ARENA_W,
        height: CONFIG.ARENA_H,
        
        towers: [],
        units: [],
        projectiles: [],
        effects: [], // Explosions
        
        elixir: 5,
        elixirTimer: 0,
        
        deck: [],
        hand: [null, null, null, null],
        nextCard: null,
        
        selectedIndex: -1,
        isGameOver: false
    };

    // --- CLASSES ---
    class Entity {
        constructor(x, y, team) {
            this.x = x;
            this.y = y;
            this.team = team;
            this.dead = false;
            this.maxHp = 100;
            this.hp = 100;
            this.radius = 30;
        }
        draw(ctx) {
            // Shadow
            ctx.fillStyle = 'rgba(0,0,0,0.3)';
            ctx.beginPath(); ctx.ellipse(this.x, this.y + 5, this.radius, this.radius/2, 0, 0, Math.PI*2); ctx.fill();
        }
    }

    class Tower extends Entity {
        constructor(x, y, team, isKing) {
            super(x, y, team);
            this.isKing = isKing;
            this.active = !isKing;
            this.radius = isKing ? 70 : 60;
            this.range = isKing ? 450 : 500;
            this.maxHp = isKing ? 4000 : 2500;
            this.hp = this.maxHp;
            this.dmg = 100;
            this.cooldown = 0;
        }
        update() {
            if (this.isKing && !this.active) {
                if (this.hp < this.maxHp) this.active = true; // Activate on dmg
            }
            if (!this.active) return;

            if (this.cooldown > 0) this.cooldown--;
            else {
                const target = getNearestEnemy(this, this.range);
                if (target) {
                    game.projectiles.push(new Projectile(this, target, this.dmg));
                    this.cooldown = 50;
                }
            }
        }
        draw(ctx) {
            super.draw(ctx);
            // Draw Tower Base
            ctx.fillStyle = this.team === TEAMS.BLUE ? '#3366cc' : '#cc3333';
            ctx.fillRect(this.x - this.radius/2, this.y - this.radius, this.radius, this.radius);
            
            // Emoji overlay
            ctx.font = `${this.radius}px Arial`;
            ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillStyle = '#fff';
            ctx.fillText(this.isKing ? '👑' : '🏰', this.x, this.y - this.radius/2);

            // HP Bar
            drawHpBar(ctx, this.x, this.y - this.radius - 20, this.radius, this.hp, this.maxHp, this.team);
        }
    }

    class Unit extends Entity {
        constructor(x, y, team, card) {
            super(x, y, team);
            this.stats = card;
            this.hp = card.hp;
            this.maxHp = card.hp;
            this.radius = 25;
            this.cooldown = 0;
            this.target = null;
        }
        update() {
            if (this.cooldown > 0) this.cooldown--;

            // Find Target
            this.target = getBestTarget(this);

            if (this.target) {
                const dist = Math.hypot(this.target.x - this.x, this.target.y - this.y);
                const reach = this.stats.range + this.target.radius;

                if (dist <= reach) {
                    // Attack
                    if (this.cooldown <= 0) {
                        if (this.stats.projectile) {
                            game.projectiles.push(new Projectile(this, this.target, this.stats.dmg));
                        } else {
                            this.target.hp -= this.stats.dmg;
                            createFloater(this.target.x, this.target.y, `-${this.stats.dmg}`);
                            if(this.target.hp <= 0) this.target.hp = 0; // Dead handled in loop
                        }
                        this.cooldown = this.stats.hitSpeed;
                    }
                } else {
                    // Move
                    const angle = Math.atan2(this.target.y - this.y, this.target.x - this.x);
                    
                    // Simple Bridge Pathing
                    let moveAngle = angle;
                    const onRiver = (this.y > 550 && this.y < 730); // River area logic coords
                    if (onRiver) {
                        // Funnel to X=180 or X=540 (Bridges)
                        const bridgeX = Math.abs(this.x - 180) < Math.abs(this.x - 540) ? 180 : 540;
                        if (Math.abs(this.x - bridgeX) > 20) {
                            moveAngle = Math.atan2(CONFIG.BRIDGE_Y - this.y, bridgeX - this.x);
                        }
                    }

                    this.x += Math.cos(moveAngle) * this.stats.speed;
                    this.y += Math.sin(moveAngle) * this.stats.speed;
                }
            } else {
                // Default move forward if no targets
                this.y += (this.team === TEAMS.BLUE ? -1 : 1) * this.stats.speed;
            }
        }
        draw(ctx) {
            super.draw(ctx);
            // Draw Unit
            ctx.font = '40px Arial';
            ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            ctx.fillText(this.stats.emoji, this.x, this.y);
            
            drawHpBar(ctx, this.x, this.y - 40, 40, this.hp, this.maxHp, this.team);
        }
    }

    class Projectile {
        constructor(owner, target, dmg) {
            this.x = owner.x;
            this.y = owner.y - 20;
            this.target = target;
            this.dmg = dmg;
            this.speed = 15;
            this.dead = false;
        }
        update() {
            const angle = Math.atan2(this.target.y - this.y, this.target.x - this.x);
            this.x += Math.cos(angle) * this.speed;
            this.y += Math.sin(angle) * this.speed;

            if (Math.hypot(this.x - this.target.x, this.y - this.target.y) < 30) {
                this.target.hp -= this.dmg;
                createFloater(this.target.x, this.target.y, `-${this.dmg}`);
                if (this.target.isKing) this.target.active = true;
                this.dead = true;
            }
        }
        draw(ctx) {
            ctx.beginPath();
            ctx.arc(this.x, this.y, 8, 0, Math.PI*2);
            ctx.fillStyle = 'orange';
            ctx.fill();
        }
    }

    // --- LOGIC FUNCTIONS ---

    function init() {
        // Resize Canvas to internal resolution
        game.canvas.width = CONFIG.ARENA_W;
        game.canvas.height = CONFIG.ARENA_H;

        // Towers
        spawnTower(360, 150, TEAMS.RED, true);
        spawnTower(180, 350, TEAMS.RED, false);
        spawnTower(540, 350, TEAMS.RED, false);

        spawnTower(360, 1130, TEAMS.BLUE, true);
        spawnTower(180, 930, TEAMS.BLUE, false);
        spawnTower(540, 930, TEAMS.BLUE, false);

        // Deck Init
        let allCards = JSON.parse(JSON.stringify(CARDS)); // Deep copy
        let indexes = [0,1,2,3,4,5,6,7];
        indexes.sort(() => Math.random() - 0.5);
        
        game.deck = indexes.map(i => allCards[i]);
        
        // Fill Hand
        for(let i=0; i<4; i++) game.hand[i] = game.deck.shift();
        game.nextCard = game.deck.shift();

        renderUI();

        // Mouse/Touch Events
        game.canvas.addEventListener('mousedown', handleInput);
        game.canvas.addEventListener('touchstart', (e) => {
            e.preventDefault();
            handleInput(e.touches[0]);
        }, {passive: false});

        // AI Spawner
        setInterval(() => {
            if (game.isGameOver) return;
            const enemyCard = CARDS[Math.floor(Math.random() * 5)]; // Only troops
            spawnUnit(enemyCard, 180 + Math.random()*360, 200, TEAMS.RED);
        }, 2500);

        loop();
    }

    function spawnTower(x, y, team, isKing) {
        game.towers.push(new Tower(x, y, team, isKing));
    }

    function spawnUnit(card, x, y, team) {
        for(let i=0; i<card.count; i++) {
            const ox = (Math.random()-0.5)*50;
            const oy = (Math.random()-0.5)*50;
            game.units.push(new Unit(x+ox, y+oy, team, card));
        }
    }

    function handleInput(e) {
        if (game.selectedIndex === -1 || game.isGameOver) return;

        // Get Coordinates adjusted for resizing
        const rect = game.canvas.getBoundingClientRect();
        const scaleX = game.canvas.width / rect.width;
        const scaleY = game.canvas.height / rect.height;

        const clientX = e.clientX;
        const clientY = e.clientY;

        const x = (clientX - rect.left) * scaleX;
        const y = (clientY - rect.top) * scaleY;

        // Valid Placement Check (Blue Side Only roughly)
        // Red side is y < 640. Blue side is y > 640.
        // Spell can go anywhere.
        const card = game.hand[game.selectedIndex];
        
        if (card.type !== 'spell' && y < CONFIG.BRIDGE_Y) {
            createFloater(x, y, "Invalid!", "red");
            return;
        }

        if (game.elixir >= card.cost) {
            // Deploy
            game.elixir -= card.cost;
            if (card.type === 'spell') {
                castSpell(card, x, y, TEAMS.BLUE);
            } else {
                spawnUnit(card, x, y, TEAMS.BLUE);
            }

            // Cycle Logic with 0.3s Delay
            cycleCard(game.selectedIndex);
            
            game.selectedIndex = -1;
            renderUI();
        } else {
            createFloater(x, y, "No Elixir!", "red");
        }
    }

    function cycleCard(slotIndex) {
        const usedCard = game.hand[slotIndex];
        
        // 1. Mark slot as loading visually
        const slotDiv = document.getElementById(`slot-${slotIndex}`);
        slotDiv.innerHTML = ''; // Clear it immediately
        
        // 2. Logic: Push used card to bottom of deck
        game.deck.push(usedCard);

        // 3. Wait 0.3s before filling from Next
        setTimeout(() => {
            game.hand[slotIndex] = game.nextCard;
            game.nextCard = game.deck.shift();
            renderUI(); // Re-render to show new card
        }, CONFIG.CYCLE_DELAY);
    }

    function castSpell(card, x, y, team) {
        // Visual
        game.effects.push({x, y, r: card.radius, life: 20});
        // Logic
        [...game.units, ...game.towers].forEach(e => {
            if (e.team !== team && Math.hypot(e.x - x, e.y - y) < card.radius) {
                e.hp -= card.dmg;
                createFloater(e.x, e.y, `-${card.dmg}`);
                if(e.isKing) e.active = true;
            }
        });
    }

    function getNearestEnemy(me, range) {
        const enemies = [...game.units, ...game.towers].filter(e => e.team !== me.team && !e.dead);
        let nearest = null;
        let minDist = range;
        
        for(let e of enemies) {
            const d = Math.hypot(e.x - me.x, e.y - me.y) - e.radius;
            if (d < minDist) {
                minDist = d;
                nearest = e;
            }
        }
        return nearest;
    }

    function getBestTarget(me) {
        const enemies = [...game.units, ...game.towers].filter(e => e.team !== me.team && !e.dead);
        
        // Filter by preference
        let candidates = enemies;
        if (me.stats.target === 'building') {
            candidates = enemies.filter(e => e instanceof Tower);
        }

        let nearest = null;
        let minDist = 9999;

        for (let e of candidates) {
            // Don't target inactive King unless it's the only building left
            if (e instanceof Tower && e.isKing && !e.active) {
                // simple check: are there other towers?
                const otherTowers = candidates.filter(t => t instanceof Tower && !t.isKing);
                if (otherTowers.length > 0) continue;
            }

            const d = Math.hypot(e.x - me.x, e.y - me.y);
            if (d < minDist) {
                minDist = d;
                nearest = e;
            }
        }
        return nearest;
    }

    // --- RENDER & UI ---

    function createFloater(x, y, text, color='#fff') {
        const div = document.createElement('div');
        div.className = 'floater';
        div.innerText = text;
        div.style.color = color;
        // Position relative to screen
        const rect = game.canvas.getBoundingClientRect();
        const sx = rect.left + (x / CONFIG.ARENA_W) * rect.width;
        const sy = rect.top + (y / CONFIG.ARENA_H) * rect.height;
        div.style.left = sx + 'px';
        div.style.top = sy + 'px';
        document.body.appendChild(div);
        setTimeout(() => div.remove(), 1000);
    }

    function drawHpBar(ctx, x, y, w, hp, max, team) {
        if (hp >= max) return;
        ctx.fillStyle = '#000';
        ctx.fillRect(x - w/2, y, w, 6);
        ctx.fillStyle = team === TEAMS.BLUE ? '#4caf50' : '#f44336';
        ctx.fillRect(x - w/2 + 1, y+1, (w-2) * (hp/max), 4);
    }

    function renderUI() {
        // Next Card
        document.getElementById('next-card-icon').innerText = game.nextCard ? game.nextCard.emoji : '';

        // Hand
        game.hand.forEach((card, i) => {
            const slot = document.getElementById(`slot-${i}`);
            slot.onclick = () => {
                if (card && game.elixir >= card.cost) {
                    game.selectedIndex = i;
                    renderUI(); // Re-render to show selection highlight
                }
            };
            
            // If slot is empty (waiting for cycle), don't draw
            if (!card) {
                // Keeping it empty implies loading
                return; 
            }
            
            // If we already have a card element, just update class
            // To be safe and simple, we rebuild the inner HTML
            let html = `
                <div class="card ${i === game.selectedIndex ? 'selected' : ''}" ${i === game.selectedIndex ? '' : 'style="transform: scale(1)"'}>
                    <div class="card-cost">${card.cost}</div>
                    <div class="card-icon">${card.emoji}</div>
                    <div class="card-name">${card.name}</div>
                </div>
            `;
            // Add loading animation class if it was just spawned (optional, but simplified here)
            slot.innerHTML = html;
            
            // Gray out if no elixir
            if (game.elixir < card.cost) {
                slot.firstElementChild.style.filter = "grayscale(1)";
                slot.firstElementChild.style.opacity = "0.7";
            }
        });
        
        // Elixir Bar
        const pct = (game.elixir / CONFIG.ELIXIR_MAX) * 100;
        document.getElementById('elixir-fill').style.width = pct + '%';
        document.getElementById('elixir-count').innerText = Math.floor(game.elixir);
    }

    function loop() {
        if (game.isGameOver) return;
        
        // 1. Update Elixir
        if (game.elixir < CONFIG.ELIXIR_MAX) {
            game.elixir += (1 / (CONFIG.FPS * CONFIG.ELIXIR_SPEED));
            if(game.elixir > CONFIG.ELIXIR_MAX) game.elixir = CONFIG.ELIXIR_MAX;
            
            // Only re-render UI every 10 frames to save perf, or if integer changes
            if (Math.floor(game.elixir) !== parseInt(document.getElementById('elixir-count').innerText)) {
                renderUI();
            } else {
                 // Smooth fill update
                 const pct = (game.elixir / CONFIG.ELIXIR_MAX) * 100;
                 document.getElementById('elixir-fill').style.width = pct + '%';
            }
        }

        // 2. Logic
        game.towers.forEach(t => t.update());
        game.units.forEach(u => u.update());
        game.projectiles.forEach(p => p.update());

        // Cleanup Dead
        game.units = game.units.filter(u => u.hp > 0);
        game.towers.forEach(t => {
            if (t.hp <= 0) {
                t.hp = 0;
                t.dead = true;
                // Force activate King if princess dies
                if (!t.isKing) {
                    game.towers.find(k => k.team === t.team && k.isKing).active = true;
                }
            }
        });
        game.projectiles = game.projectiles.filter(p => !p.dead);

        // Check Win
        const blueKing = game.towers.find(t => t.team === TEAMS.BLUE && t.isKing);
        const redKing = game.towers.find(t => t.team === TEAMS.RED && t.isKing);
        
        if (blueKing.dead || redKing.dead) {
            game.isGameOver = true;
            document.getElementById('game-over').style.display = 'flex';
            document.getElementById('winner-msg').innerText = blueKing.dead ? "RED WINS!" : "BLUE WINS!";
            return;
        }

        // 3. Draw
        const ctx = game.ctx;
        
        // Background
        ctx.fillStyle = '#6bc234';
        ctx.fillRect(0, 0, CONFIG.ARENA_W, CONFIG.ARENA_H);
        
        // River
        ctx.fillStyle = '#3aa6e8';
        ctx.fillRect(0, 600, CONFIG.ARENA_W, 80);
        
        // Bridges
        ctx.fillStyle = '#d98e36';
        ctx.fillRect(140, 590, 80, 100);
        ctx.fillRect(500, 590, 80, 100);
        
        // Entities (Sorted by Y for occlusion)
        const allEntities = [...game.towers, ...game.units].sort((a,b) => a.y - b.y);
        allEntities.forEach(e => {
            if (!e.dead || e instanceof Tower) e.draw(ctx); // Draw towers even if dead (ruins)
        });

        // Projectiles
        game.projectiles.forEach(p => p.draw(ctx));

        // Effects
        game.effects = game.effects.filter(e => e.life > 0);
        game.effects.forEach(e => {
            ctx.fillStyle = `rgba(255, 165, 0, ${e.life/20})`;
            ctx.beginPath(); ctx.arc(e.x, e.y, e.r, 0, Math.PI*2); ctx.fill();
            e.life--;
        });

        requestAnimationFrame(loop);
    }

    // Start
    window.onload = init;

</script>
</body>
</html>
