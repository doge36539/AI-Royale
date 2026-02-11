<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Clash Royale 9:16</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Lilita+One&family=Roboto:wght@700&display=swap');

        :root {
            --grass: #6bc234;
            --river: #3aa6e8;
            --bridge: #d98e36;
            --ui-bg: rgba(0, 0, 0, 0.6);
        }

        body {
            margin: 0;
            padding: 0;
            background-color: #111;
            height: 100vh;
            width: 100vw;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            font-family: 'Roboto', sans-serif;
            user-select: none;
        }

        /* 9:16 Aspect Ratio Container */
        #game-container {
            position: relative;
            height: 100vh;
            max-height: 100vh;
            aspect-ratio: 9/16;
            background: var(--grass);
            box-shadow: 0 0 50px black;
            overflow: hidden;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }

        /* UI Overlay - Bottom Middle */
        #ui-layer {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 180px;
            pointer-events: none; /* Let clicks pass through empty spaces */
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            align-items: center;
            padding-bottom: 10px;
            box-sizing: border-box;
            background: linear-gradient(to top, rgba(0,0,0,0.8) 0%, transparent 100%);
        }

        /* Elixir Bar */
        #elixir-wrapper {
            width: 90%;
            height: 25px;
            background: #222;
            border: 2px solid #555;
            border-radius: 15px;
            margin-bottom: 8px;
            position: relative;
            overflow: hidden;
            pointer-events: auto;
        }

        #elixir-fill {
            height: 100%;
            width: 50%;
            background: linear-gradient(to bottom, #d536d8, #b026b3);
            transition: width 0.1s linear;
        }

        #elixir-text {
            position: absolute;
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-family: 'Lilita One', cursive;
            text-shadow: 1px 1px 0 #000;
            z-index: 2;
        }

        /* Hand */
        #hand {
            display: flex;
            gap: 10px;
            pointer-events: auto;
            align-items: flex-end;
        }

        .card {
            width: 70px;
            height: 90px;
            background: #fff;
            border-radius: 6px;
            border-bottom: 4px solid #888;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            position: relative;
            cursor: pointer;
            transition: transform 0.1s;
            box-shadow: 0 4px 6px rgba(0,0,0,0.5);
        }

        .card.selected {
            transform: translateY(-20px);
            border: 3px solid #ffff00;
            box-shadow: 0 0 20px #ffff00;
            z-index: 10;
        }

        .card-emoji {
            font-size: 40px;
            line-height: 40px;
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
            align-items: center;
            justify-content: center;
        }

        /* Next Card Bubble */
        #next-card {
            position: absolute;
            bottom: 120px;
            left: 10px;
            width: 40px; height: 50px;
            background: #333;
            border: 1px solid #555;
            border-radius: 4px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: #888;
            font-size: 10px;
            pointer-events: auto;
        }
        #next-emoji { font-size: 20px; margin-top: 2px; }

        /* Popups */
        .floater {
            position: absolute;
            font-weight: bold;
            pointer-events: none;
            animation: floatUp 0.8s forwards;
            font-size: 24px;
            text-shadow: 2px 2px 0 #000;
        }
        @keyframes floatUp { to { transform: translateY(-60px); opacity: 0; } }

        #game-over {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 100;
            font-family: 'Lilita One', cursive;
        }
    </style>
</head>
<body>

<div id="game-container">
    <canvas id="gameCanvas"></canvas>

    <div id="ui-layer">
        <div id="next-card">
            NEXT
            <div id="next-emoji">?</div>
        </div>

        <div id="elixir-wrapper">
            <div id="elixir-fill"></div>
            <div id="elixir-text">5</div>
        </div>

        <div id="hand">
            </div>
    </div>

    <div id="game-over">
        <h1 id="winner-text" style="font-size: 40px; margin-bottom: 20px;">GAME OVER</h1>
        <button onclick="location.reload()" style="padding: 15px 30px; font-size: 20px; cursor: pointer; background:#3aa6e8; border:none; color:white; border-radius:8px;">PLAY AGAIN</button>
    </div>
</div>

<script>
// --- CONFIG ---
const CONFIG = {
    FPS: 60,
    ELIXIR_SPEED: 0.5, // Super fast elixir
    ARENA_W: 720,
    ARENA_H: 1280, // 9:16 Internal Resolution
    BRIDGE_Y: 640
};

const TEAMS = { BLUE: 0, RED: 1 };

const CARDS = [
    { name: 'Knight', emoji: '⚔️', cost: 3, hp: 1400, dmg: 150, speed: 4, range: 60, type: 'troop', count: 1, hitSpeed: 60 },
    { name: 'Archers', emoji: '🏹', cost: 3, hp: 300, dmg: 100, speed: 5, range: 350, type: 'troop', count: 2, projectile: true, hitSpeed: 50 },
    { name: 'Giant', emoji: '🧔', cost: 5, hp: 3500, dmg: 250, speed: 2, range: 80, type: 'troop', count: 1, target: 'building', hitSpeed: 100 },
    { name: 'Pekka', emoji: '🤖', cost: 4, hp: 1200, dmg: 600, speed: 6, range: 60, type: 'troop', count: 1, hitSpeed: 90 },
    { name: 'Skeletons', emoji: '💀', cost: 1, hp: 80, dmg: 80, speed: 6, range: 40, type: 'troop', count: 3, hitSpeed: 30 },
    { name: 'Wiz', emoji: '🧙‍♂️', cost: 5, hp: 600, dmg: 200, speed: 4, range: 300, type: 'troop', count: 1, projectile: true, area: 100, hitSpeed: 80 },
    { name: 'Fireball', emoji: '🔥', cost: 4, dmg: 600, type: 'spell', radius: 250 },
    { name: 'Arrows', emoji: '➵', cost: 3, dmg: 300, type: 'spell', radius: 300 }
];

// --- GAME STATE ---
const game = {
    canvas: document.getElementById('gameCanvas'),
    ctx: document.getElementById('gameCanvas').getContext('2d'),
    width: CONFIG.ARENA_W,
    height: CONFIG.ARENA_H,
    
    entities: [], // Towers and Units
    projectiles: [],
    
    elixir: 5,
    deck: [],
    hand: [],
    nextCard: null,
    
    selectedIdx: -1,
    gameOver: false
};

// --- CLASSES ---

class Entity {
    constructor(x, y, team) {
        this.x = x; this.y = y; this.team = team;
        this.hp = 100; this.maxHp = 100;
        this.dead = false;
        this.radius = 30;
    }
}

class Tower extends Entity {
    constructor(x, y, team, isKing) {
        super(x, y, team);
        this.isKing = isKing;
        this.active = !isKing; // King inactive
        this.maxHp = isKing ? 4000 : 2500;
        this.hp = this.maxHp;
        this.radius = isKing ? 70 : 60;
        this.range = isKing ? 450 : 500;
        this.dmg = 120;
        this.cooldown = 0;
        this.emoji = isKing ? '👑' : '🏰';
    }
    
    update() {
        if(this.dead) return;
        
        // King Activate Logic
        if(this.isKing && !this.active) {
            if(this.hp < this.maxHp) this.active = true; // Damaged
            // Check princess towers
            const princess = game.entities.filter(e => e instanceof Tower && e.team === this.team && !e.isKing && !e.dead);
            if(princess.length < 2) this.active = true;
        }

        if(!this.active) return;

        if(this.cooldown > 0) this.cooldown--;
        else {
            const target = getNearestEnemy(this, this.range);
            if(target) {
                game.projectiles.push(new Projectile(this, target, this.dmg));
                this.cooldown = 40;
            }
        }
    }

    draw(ctx) {
        if(this.dead) return; // Disappear if dead

        // Base
        ctx.fillStyle = this.team === TEAMS.BLUE ? '#2196f3' : '#f44336';
        ctx.fillRect(this.x - this.radius, this.y - this.radius, this.radius*2, this.radius*2);
        
        // Emoji
        ctx.font = `${this.radius*1.5}px Arial`;
        ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        ctx.fillText(this.emoji, this.x, this.y);

        // HP Bar
        drawHp(ctx, this.x, this.y - this.radius - 20, 80, this.hp, this.maxHp, this.team);
    }
}

class Unit extends Entity {
    constructor(x, y, team, card) {
        super(x, y, team);
        this.stats = card;
        this.hp = card.hp; this.maxHp = card.hp;
        this.radius = 25;
        this.cooldown = 0;
    }
    
    update() {
        if(this.dead) return;
        if(this.cooldown > 0) this.cooldown--;

        let target = getBestTarget(this);

        if(target) {
            const dist = Math.hypot(target.x - this.x, target.y - this.y);
            const reach = this.stats.range + target.radius;

            if(dist <= reach) {
                // Attack
                if(this.cooldown <= 0) {
                    if(this.stats.projectile) {
                        game.projectiles.push(new Projectile(this, target, this.stats.dmg));
                    } else {
                        target.hp -= this.stats.dmg;
                        addFloat(target.x, target.y, `-${this.stats.dmg}`);
                        if(target.isKing) target.active = true;
                    }
                    this.cooldown = this.stats.hitSpeed;
                }
            } else {
                // Move
                const ang = Math.atan2(target.y - this.y, target.x - this.x);
                let moveAng = ang;

                // Bridge Logic
                const onRiver = (this.y > 550 && this.y < 730);
                if(onRiver) {
                    const bridgeX = Math.abs(this.x - 180) < Math.abs(this.x - 540) ? 180 : 540;
                    if(Math.abs(this.x - bridgeX) > 20) {
                        moveAng = Math.atan2(CONFIG.BRIDGE_Y - this.y, bridgeX - this.x);
                    }
                }

                this.x += Math.cos(moveAng) * this.stats.speed;
                this.y += Math.sin(moveAng) * this.stats.speed;
            }
        } else {
            // March forward if no targets
            this.y += (this.team === TEAMS.BLUE ? -1 : 1) * this.stats.speed;
        }

        if(this.hp <= 0) this.dead = true;
    }

    draw(ctx) {
        if(this.dead) return;
        // Shadow
        ctx.fillStyle = 'rgba(0,0,0,0.4)';
        ctx.beginPath(); ctx.ellipse(this.x, this.y+10, this.radius, 10, 0, 0, Math.PI*2); ctx.fill();

        // Emoji (Texture)
        ctx.font = '50px Arial';
        ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        ctx.fillText(this.stats.emoji, this.x, this.y);

        drawHp(ctx, this.x, this.y - 40, 50, this.hp, this.maxHp, this.team);
    }
}

class Projectile {
    constructor(owner, target, dmg) {
        this.x = owner.x; this.y = owner.y - 20;
        this.target = target;
        this.dmg = dmg;
        this.speed = 18;
        this.dead = false;
    }
    update() {
        const ang = Math.atan2(this.target.y - this.y, this.target.x - this.x);
        this.x += Math.cos(ang) * this.speed;
        this.y += Math.sin(ang) * this.speed;

        if(Math.hypot(this.x - this.target.x, this.y - this.target.y) < 30) {
            this.target.hp -= this.dmg;
            addFloat(this.target.x, this.target.y, `-${this.dmg}`);
            if(this.target.isKing) this.target.active = true;
            this.dead = true;
        }
    }
    draw(ctx) {
        // Big Visible Projectile
        ctx.beginPath();
        ctx.arc(this.x, this.y, 15, 0, Math.PI*2);
        ctx.fillStyle = '#ffeb3b'; // Bright Yellow
        ctx.fill();
        ctx.strokeStyle = '#f57f17';
        ctx.lineWidth = 3;
        ctx.stroke();
    }
}

// --- LOGIC ---

function init() {
    game.canvas.width = CONFIG.ARENA_W;
    game.canvas.height = CONFIG.ARENA_H;

    // Towers
    game.entities.push(new Tower(360, 150, TEAMS.RED, true));
    game.entities.push(new Tower(180, 350, TEAMS.RED, false));
    game.entities.push(new Tower(540, 350, TEAMS.RED, false));

    game.entities.push(new Tower(360, 1130, TEAMS.BLUE, true));
    game.entities.push(new Tower(180, 930, TEAMS.BLUE, false));
    game.entities.push(new Tower(540, 930, TEAMS.BLUE, false));

    // Cards
    let deckIdx = [0,1,2,3,4,5,6,7].sort(()=>Math.random()-0.5);
    game.deck = deckIdx.map(i => CARDS[i]);
    game.hand = game.deck.splice(0, 4);
    game.nextCard = game.deck.shift();

    renderUI();
    
    // Inputs
    game.canvas.addEventListener('mousedown', handleInput);
    game.canvas.addEventListener('touchstart', (e)=>{e.preventDefault(); handleInput(e.touches[0])}, {passive:false});

    // Enemy AI
    setInterval(() => {
        if(game.gameOver) return;
        const card = CARDS[Math.floor(Math.random()*6)];
        spawnUnit(card, 180 + Math.random()*360, 200, TEAMS.RED);
    }, 2500);

    loop();
}

function spawnUnit(card, x, y, team) {
    for(let i=0; i<card.count; i++) {
        const ox = (Math.random()-0.5)*40;
        game.entities.push(new Unit(x+ox, y, team, card));
    }
}

function getNearestEnemy(me, range) {
    // Filter dead
    let enemies = game.entities.filter(e => e.team !== me.team && !e.dead);
    let close = null;
    let min = range;
    for(let e of enemies) {
        let d = Math.hypot(e.x - me.x, e.y - me.y) - e.radius;
        if(d < min) { min = d; close = e; }
    }
    return close;
}

function getBestTarget(me) {
    let enemies = game.entities.filter(e => e.team !== me.team && !e.dead);
    if(me.stats.target === 'building') enemies = enemies.filter(e => e instanceof Tower);

    let close = null;
    let min = 9999;
    for(let e of enemies) {
        // Skip inactive King unless forced
        if(e instanceof Tower && e.isKing && !e.active) {
            if(enemies.filter(t => t instanceof Tower && !t.isKing).length > 0) continue;
        }
        let d = Math.hypot(e.x - me.x, e.y - me.y);
        if(d < min) { min = d; close = e; }
    }
    return close;
}

function handleInput(e) {
    if(game.selectedIdx === -1 || game.gameOver) return;

    const rect = game.canvas.getBoundingClientRect();
    const scaleX = CONFIG.ARENA_W / rect.width;
    const scaleY = CONFIG.ARENA_H / rect.height;
    
    const x = (e.clientX - rect.left) * scaleX;
    const y = (e.clientY - rect.top) * scaleY;

    // Check bounds (roughly blue side or spell)
    const card = game.hand[game.selectedIdx];
    if(card.type !== 'spell' && y < CONFIG.BRIDGE_Y) {
        addFloat(x, y, "INVALID", "red");
        return;
    }

    if(game.elixir >= card.cost) {
        game.elixir -= card.cost;
        
        if(card.type === 'spell') {
            // AoE Damage
            game.entities.forEach(e => {
                if(e.team !== TEAMS.BLUE && !e.dead && Math.hypot(e.x - x, e.y - y) < card.radius) {
                    e.hp -= card.dmg;
                    addFloat(e.x, e.y, `-${card.dmg}`);
                    if(e.isKing) e.active = true;
                }
            });
            // Visual Effect
            const fx = {x, y, r: 1, maxR: card.radius, life: 20};
            game.projectiles.push(fx); // Hack to draw using proj loop or just add fx array
            fx.draw = (ctx) => {
                ctx.fillStyle = 'rgba(255,100,0,0.5)';
                ctx.beginPath(); ctx.arc(fx.x, fx.y, fx.maxR, 0, Math.PI*2); ctx.fill();
            }
        } else {
            spawnUnit(card, x, y, TEAMS.BLUE);
        }

        // Cycle
        const used = game.hand[game.selectedIdx];
        game.deck.push(used);
        game.hand[game.selectedIdx] = null; // Empty slot
        
        const slot = game.selectedIdx;
        renderUI();
        
        setTimeout(() => {
            game.hand[slot] = game.nextCard;
            game.nextCard = game.deck.shift();
            renderUI();
        }, 300);

        game.selectedIdx = -1;
        renderUI();
    } else {
        addFloat(x, y, "NO ELIXIR", "red");
    }
}

// --- RENDER ---

function drawHp(ctx, x, y, w, hp, max, team) {
    if(hp >= max) return;
    ctx.fillStyle = '#000';
    ctx.fillRect(x - w/2, y, w, 8);
    ctx.fillStyle = team === TEAMS.BLUE ? '#4caf50' : '#f44336';
    ctx.fillRect(x - w/2+1, y+1, (w-2)*(hp/max), 6);
}

function addFloat(x, y, text, col='#fff') {
    const div = document.createElement('div');
    div.className = 'floater';
    div.innerText = text;
    div.style.color = col;
    
    // Convert game coord to screen coord
    const rect = game.canvas.getBoundingClientRect();
    const sx = rect.left + (x/CONFIG.ARENA_W)*rect.width;
    const sy = rect.top + (y/CONFIG.ARENA_H)*rect.height;
    
    div.style.left = sx + 'px';
    div.style.top = sy + 'px';
    document.body.appendChild(div);
    setTimeout(()=>div.remove(), 800);
}

function renderUI() {
    document.getElementById('next-emoji').innerText = game.nextCard.emoji;
    document.getElementById('elixir-fill').style.width = (game.elixir/10)*100 + '%';
    document.getElementById('elixir-text').innerText = Math.floor(game.elixir);

    const container = document.getElementById('hand');
    container.innerHTML = '';
    
    game.hand.forEach((card, i) => {
        const div = document.createElement('div');
        div.className = `card ${i===game.selectedIdx ? 'selected' : ''}`;
        
        if(!card) {
            div.style.opacity = 0; // Placeholder for cycle delay
        } else {
            div.innerHTML = `
                <div class="card-cost">${card.cost}</div>
                <div class="card-emoji">${card.emoji}</div>
            `;
            if(game.elixir < card.cost) {
                div.style.filter = "grayscale(1)";
                div.style.opacity = 0.6;
            }
            div.onclick = (e) => {
                e.stopPropagation();
                if(game.elixir >= card.cost) {
                    game.selectedIdx = i;
                    renderUI();
                }
            }
        }
        container.appendChild(div);
    });
}

function loop() {
    if(game.gameOver) return;

    // Logic
    if(game.elixir < 10) {
        game.elixir += 1 / (60 * CONFIG.ELIXIR_SPEED);
        if(game.elixir > 10) game.elixir = 10;
        if(Math.floor(game.elixir) !== parseInt(document.getElementById('elixir-text').innerText)) renderUI();
        document.getElementById('elixir-fill').style.width = (game.elixir/10)*100 + '%';
    }

    game.entities.forEach(e => e.update());
    game.projectiles.forEach(p => p.update());
    
    // Cleanup Dead
    game.entities = game.entities.filter(e => e.hp > 0);
    game.projectiles = game.projectiles.filter(p => !p.dead);

    // Win Check
    const blueK = game.entities.find(e => e instanceof Tower && e.team === TEAMS.BLUE && e.isKing);
    const redK = game.entities.find(e => e instanceof Tower && e.team === TEAMS.RED && e.isKing);
    
    if(!blueK || !redK) {
        game.gameOver = true;
        document.getElementById('game-over').style.display = 'flex';
        document.getElementById('winner-text').innerText = blueK ? "BLUE WINS!" : "RED WINS!";
    }

    // Draw
    const ctx = game.ctx;
    ctx.fillStyle = '#6bc234';
    ctx.fillRect(0,0,CONFIG.ARENA_W, CONFIG.ARENA_H);

    // River & Bridge
    ctx.fillStyle = '#3aa6e8';
    ctx.fillRect(0, 600, CONFIG.ARENA_W, 80);
    ctx.fillStyle = '#d98e36';
    ctx.fillRect(140, 590, 80, 100);
    ctx.fillRect(500, 590, 80, 100);

    // Sort by Y for depth
    game.entities.sort((a,b) => a.y - b.y);
    game.entities.forEach(e => e.draw(ctx));

    // Projectiles
    game.projectiles.forEach(p => {
        if(p.draw) p.draw(ctx); // Special FX
        else p.draw(ctx); // Standard Class
    });

    requestAnimationFrame(loop);
}

init();

</script>
</body>
</html>
