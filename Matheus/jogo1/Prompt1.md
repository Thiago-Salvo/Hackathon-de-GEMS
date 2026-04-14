1. VISÃO GERAL
Crie um jogo completo em um único arquivo HTML chamado `ABYSS INVADERS - YELLOW SUBMARINE EDITION`.  
É um clone de Space Invaders ambientado no fundo do oceano abissal, com estética retro-bioluminescente
(CRT/terminal anos 80 + criaturas das profundezas associadas a alienígenas).  
Use Phaser 3 (CDN: `https://cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js`).  
O canvas deve ser 800 × 600 px.
---
2. ESTRUTURA DE CENAS
O jogo possui 4 cenas registradas nesta ordem:
```
[BootScene, MainMenu, GameScene, GameOver]
```
---
3. ASSETS EXTERNOS (carregados via `this.load.image`)
Criaturas inimigas (5 linhas, 1 tipo por linha):
```
creature1 → assets/abyssal_creatures_sprites/Creature_1.png
creature2 → assets/abyssal_creatures_sprites/Creature_2.png
creature3 → assets/abyssal_creatures_sprites/Creature_3.png
creature4 → assets/abyssal_creatures_sprites/Creature_4.png
creature5 → assets/abyssal_creatures_sprites/Creature_5.png
```
Decoração do fundo oceânico:
```
algae      → assets/coral_sprites/algae2.png
concha     → assets/coral_sprites/Concha.png
coral1     → assets/coral_sprites/Coral1.png
coral3     → assets/coral_sprites/Coral3.png   ← coral rosa
rock       → assets/coral_sprites/rock.png
chest      → assets/coral_sprites/chest.png
```
---
4. ASSETS GERADOS VIA CÓDIGO (Phaser Graphics)
Todos gerados com `this.add.graphics()` + `generateTexture()` na `BootScene`.
Use `if (this.textures.exists('player')) return;` para evitar duplicação.
Chave	Tamanho	Descrição
`heart`	24×24	Dois círculos `0xff3366` r=6 em (8,8) e (16,8) + triângulo (2,10)→(22,10)→(12,22)
`bgBubble`	16×16	Círculo stroke `0xaaffff` r=6 em (8,8), lineStyle 2px
`player`	40×30	Submarino amarelo `0xffff00`: corpo `fillRoundedRect(5,10,30,15,r=8)`, torre `fillRect(18,2,4,10)`, antena `fillRect(20,2,6,3)`, 3 janelas azuis `0x003366` em (12,17)(20,17)(28,17) r=2.5, hélice laranja `0xff9900` `fillRect(0,14,5,7)`
`bubble`	12×12	Círculo stroke `0x00ffcc` r=5 em (6,6), lineStyle 2px — projétil do jogador
`enemy_shot`	14×22	Orbe bioluminescente multicamadas (ver §4.1)
`coral`	60×40	`fillRoundedRect(0,0,60,40,r=10)` cor `0x00cc66` — barreira
`particle`	4×4	`fillCircle(2,2,2)` branco — partícula de explosão
4.1 — Textura `enemy_shot` (orbe abissal)
```javascript
g.clear();
// Cauda esfumaçada (acima do orbe — dá sentido descendente)
g.fillStyle(0x660099, 0.25); g.fillEllipse(7, 2, 5, 8);
g.fillStyle(0x9900cc, 0.40); g.fillEllipse(7, 5, 4, 6);
// Halo exterior
g.fillStyle(0xcc00ff, 0.30); g.fillCircle(7, 14, 6);
// Corpo principal
g.fillStyle(0xee44ff, 0.85); g.fillCircle(7, 14, 4);
// Núcleo brilhante
g.fillStyle(0xffffff, 1);    g.fillCircle(7, 14, 2);
// Brilho especular
g.fillStyle(0xffffff, 0.7);  g.fillCircle(6, 13, 1);
g.generateTexture('enemy_shot', 14, 22);
```
---
5. CSS / APRESENTAÇÃO VISUAL
```css
/* Fonte: importar do Google Fonts */
@import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap');

body {
  background: #000;
  display: flex; justify-content: center; align-items: center;
  height: 100vh; overflow: hidden;
}

#game-container {
  position: relative;
  box-shadow:
    0 0 0 2px #00ffcc,
    0 0 20px rgba(0,255,204,0.4),
    0 0 60px rgba(0,255,204,0.15);
  border-radius: 2px;
}

/* Scanlines CRT — NÃO cobre a areia (últimos 52px do canvas) */
#game-container::after {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 548px;           /* canvas 600px − areia 52px */
  background: repeating-linear-gradient(
    0deg, transparent, transparent 2px,
    rgba(0,0,0,0.08) 2px, rgba(0,0,0,0.08) 4px
  );
  pointer-events: none;
  z-index: 10;
}

/* Vinheta radial */
#game-container::before {
  content: '';
  position: absolute; inset: 0;
  background: radial-gradient(ellipse at center, transparent 60%, rgba(0,0,20,0.6) 100%);
  pointer-events: none; z-index: 11;
}
```
Fonte padrão para todos os textos do jogo: `'Share Tech Mono', 'Courier New', monospace`
---
6. CENA: BootScene
Carrega todos os assets externos
Gera todas as texturas procedurais
Transita para `MainMenu` no `create()`
---
7. CENA: MainMenu
Layout do painel central
Painel escuro `0x000d1a` com alpha 0.88, `fillRoundedRect(120,130,560,300,r=6)`
Borda dupla: linha `0x00ffcc` opacidade 0.9, interna opacidade 0.2
Cantos em bracket `┌ ┐ └ ┘` desenhados manualmente (tamanho 14px, lineStyle 2px)
Elementos de texto (fonte Share Tech Mono)
Elemento	Posição	Tamanho	Cor
Título `ABYSS INVADERS` (3 camadas glow)	centro, y=202	46px	`#00ffcc` — pisca alpha 1→0.85
Subtítulo `— YELLOW SUBMARINE EDITION —`	centro, y=250	13px	`#ffcc00`
Linha separadora dupla	x=160→640, y=270/271	—	`#00ffcc` alpha 0.5
`> PROFUNDIDADE: 7800m`	x=190, y=292	13px	`#4488aa`
`> CRIATURAS ABISSAIS DETECTADAS: 55`	x=190, y=312	13px	`#4488aa`
`> STATUS DO SUBMARINO: OPERACIONAL`	x=190, y=332	13px	`#4488aa`
`> PROTOCOLO: ELIMINAÇÃO TOTAL`	x=190, y=352	13px	`#4488aa`
Painel controles (fundo `0x001122`)	x=150,y=375, w=500, h=38	—	borda `0x003344`
Texto controles	centro, y=394	11px	`#006655`
`[ MERGULHAR ]` (pisca stepped)	centro, y=460	24px	`#00ffcc`
Fundo do menu (`_buildMenuOcean`)
Gradiente vertical `0x0a1a4a` → `0x000510` em 20 faixas
3 feixes de luz triangulares `0x5599ff` alpha 0.05
Areia: `fillRect(0,548,W,52)` cor `0xb89560` + elipses irregulares `0xd4b07a`
Partículas `bgBubble` subindo do fundo (emitter com `blendMode: 'ADD'`)
Criaturas decorativas (`_spawnMenuCreatures`)
3 imagens de criaturas com alpha baixo flutuando suavemente (tween y ±12px):
`creature1` em (130, 420) scale=0.35 alpha=0.25
`creature3` em (660, 390) scale=0.4 alpha=0.2
`creature5` em (400, 480) scale=0.3 alpha=0.15
Input
`SPACE` ou `pointerdown` → `fadeOut(400)` → `GameScene`
---
8. CENA: GameScene
8.1 Inicialização (`init`)
```javascript
this.score = 0; this.lives = 3; this.wave = 1;
```
8.2 Grupos de física
```javascript
this.playerBullets = this.physics.add.group();
this.enemyBullets  = this.physics.add.group();
this.enemies       = this.physics.add.group();
this.barriers      = this.physics.add.group();
```
8.3 Timer de tiro inimigo
```javascript
this.enemyFireTimer = this.time.addEvent({
  delay: 1000, callback: this.enemyFire,
  callbackScope: this, loop: true
});
```
8.4 Jogador
Sprite `player` em (400, 520), `setCollideWorldBounds(true)`, depth=5
Move com setas esquerda/direita a ±300 px/s
Dispara com SPACE ou clique (cooldown 400ms)
Projétil `bubble` com `velocityY = -450`
8.5 Colisões
```javascript
overlap(playerBullets, enemies,  hitEnemy)
overlap(enemyBullets,  player,   hitPlayer)
overlap([playerBullets, enemyBullets], barriers, hitBarrier)
```
8.6 SpawnWave
5 linhas × 11 colunas = 55 inimigos
Posição inicial: `x = 100 + col*55`, `y = 80 + row*60`
Criatura por linha: `creature1` a `creature5` (row 0→4)
`scoreValue = (5 - row) * 10` (linha do topo vale 50, base vale 10)
`setScale(0.5)`, depth=3
`enemySpeed = 30 + wave * 10`
`enemyFireTimer.delay = Math.max(200, 1000 - wave*100)`
Exibe `_showWaveAnnouncement()` — painel ciano centralizado que faz fade em 1.8s
8.7 Movimento dos inimigos
Todos se movem horizontalmente com a mesma velocityX
Ao atingir x<20 ou x>780: inverter direção + descer 15px cada um
`speedMult = 1 + (55 - count) / 55` (ficam mais rápidos conforme diminuem)
8.8 Tiro inimigo (`enemyFire`)
Seleciona um inimigo aleatório ativo
Cria projétil `enemy_shot` com `velocityY = 250 + wave*20`
8.9 Colisão hitEnemy
Destrói bullet + enemy
Explode 12 partículas (emitter `blendMode: ADD`)
Soma score, chama `updateHUD()`
Se `countActive() === 0`: `wave++`, delayedCall 1000ms → `spawnWave()`
8.10 Colisão hitPlayer
Destrói bullet, `lives--`
Anima o último coração (scale 1.6, alpha 0, 250ms) e remove
`cameras.main.shake(120, 0.012)`
Se `lives <= 0`: delayedCall 300ms → `GameOver { score, wave }`
8.11 Colisão hitBarrier
`barrier.health--`, `setAlpha(health/5)`
Se `health <= 0`: destroi
---
9. HUD (GameScene)
Painel superior
`fillRect(0,0,800,50)` cor `0x000d1a` alpha 0.92
Linha inferior `0x00ffcc` alpha 0.7 + inner glow alpha 0.15
3 separadores verticais em x=220, 440, 620 (alpha 0.25)
Cantos decorativos em bracket (8px)
Campos (todos em Share Tech Mono)
Campo	Label (9px `#005544`)	Valor	Posição valor	Cor valor
Pontos	`PONTOS` x=14,y=8	6 dígitos zero-fill	x=14,y=20 20px	`#00ffcc`
Onda	`ONDA` x=234,y=8	`W-01` formato	x=234,y=20 20px	`#ffcc00`
Vidas	`VIDAS` x=454,y=8	ícones heart	x=458+i*28, y=28	tint `0xff3366`
Profundidade	`PROF.` x=634,y=8	`3800m` decrescente	x=634,y=20 20px	`#4488aa`
Fórmula profundidade: `Math.max(100, 3800 - (wave-1) * 340)`
Painel inferior
`fillRect(0,548,800,52)` cor `0x000810` alpha 0.7
Linha superior `0x00ffcc` alpha 0.15
---
10. CENÁRIO OCEÂNICO (createOceanAmbience)
Chamar nesta ordem:
`createWaterGradient()`
`createSandFloor()`
`createParallaxBackground()`
`createBubbleSystem()`
`createBackgroundFish()`
`createDecorativeCorals()`
10.1 Gradiente da água
24 faixas horizontais interpolando `0x244fc7` → `0x041238`, depth=-30
Escurecimento inferior: `fillRect(0,430,800,170)` `0x020814` alpha 0.28
3 feixes de luz triangulares `0x9ec5ff` alpha 0.07, depth=-29
10.2 Areia (depth=-5)
`fillRect(0,548,800,52)` cor `0xc2a56c`
40 elipses `0xd7bb83` irregulares
180 pontinhos granulados `0xb9985e`/`0xe1c78f` alpha 0.45
10.3 Parallax de 3 camadas
Cada camada redesenhada a cada frame em `update()`:
Camada	Depth	Cor	Velocidade	baseY
bgFar	-25	`0x17389f`	-0.08 px/frame	470
bgMid	-18	`0x0f2d87`	-0.18 px/frame	520
bgNear	-8	`0x021a63`	-0.35 px/frame	565
Cada camada: colinas irregulares (`drawHillChain`) + silhuetas de algas (`drawSeaweedSilhouettes`).
drawHillChain: path fechado com pontos alternando `baseY` (vale) e `peakY=baseY-h` (pico).  
drawSeaweedSilhouettes: 3 hastes retangulares + 2 triângulos foliares por item.
O offset de cada camada usa `% 800` para loop contínuo. Dobrar a lista de colinas para cobrir 2× a largura.
10.4 Bolhas de fundo
```javascript
emitter: x={min:20,max:780}, y=590, speedY={min:-90,max:-40},
scale={start:0.15,end:0.6}, alpha={start:0,end:0.45},
frequency=220, blendMode='ADD'
```
10.5 Peixinhos de fundo
Timer a cada 2600ms: elipse azul escuro atravessa da esquerda (-30) para direita (850),
altura aleatória entre y=120 e y=420, duração 7000–11000ms.
10.6 Decoração do fundo (zonas)
Distribuição em 3 zonas com sistema de `depth` em -4 (fundos), -3 (médios), -2 (frente).
Assets não usam `seaweed_img`. Todos têm `setOrigin(0.5, 1)` + ângulo aleatório leve (±3°).
Itens com `swing:true` recebem tween de balanço (angle ±2.5°, 2000–3400ms).
Corais com depth=-2 recebem tween de bioluminescência (alpha 0.88↔1).
Linha de brilho na borda da areia: `lineStyle(2, 0x44aaff, 0.18)` em y=548.
Lista de elementos por zona:
Zona Esquerda (x 22–224):
```
rock   x=22  scale=0.12  depth=-4  swing=false
chest  x=72  scale=0.07  depth=-3  swing=false
algae  x=120 scale=0.13  depth=-7  swing=true
coral1 x=168 scale=0.58  depth=-1  swing=true
algae  x=222 scale=0.08  depth=-4  swing=true
```
Zona Central (x 268–516):
```
coral3 x=268 scale=0.35  depth=-3  swing=true
coral1 x=338 scale=0.85  depth=-3  swing=true
concha x=372 scale=0.30  depth=-2  swing=false
chest  x=520 scale=0.05  depth=-2  swing=false
concha x=428 scale=0.24  depth=-2  swing=false
coral3 x=462 scale=0.44  depth=-3  swing=true
```
Zona Direita (x 538–775):
```
coral1 x=538 scale=0.36  depth=-3  swing=true
rock   x=572 scale=0.18  depth=-4  swing=false
algae  x=614 scale=0.09  depth=-3  swing=true
coral3 x=648 scale=0.42  depth=-3  swing=true
concha x=672 scale=0.28  depth=-2  swing=false
rock   x=726 scale=0.15  depth=-4  swing=false
algae  x=736 scale=0.11  depth=-3  swing=true
coral1 x=775 scale=0.40  depth=-3  swing=true
```
y de todos os elementos: `FLOOR + N` onde `FLOOR = 548` e N varia entre 1 e 32 conforme o asset
(rochas ficam mais "enterradas": +20 a +32; corais e algas: +2 a +5).
10.7 Barreiras
4 corais posicionados em `x = 130 + i*180`, `y = 430`.  
`health = 5`, `setImmovable(true)`. A cada dano: `setAlpha(health/5)`.
---
11. CENA: GameOver
Layout
Fundo `0x000510` + brilho vermelho radial `0xff0000` alpha 0.04
Painel `0x000d1a` em (160,140,480×320), bordas duplas `0xff0044`
Cantos em bracket vermelhos
Elementos
Elemento	Posição	Tamanho	Cor
`! ! ! ALERTA CRÍTICO ! ! !`	centro y=168	11px	`#ff0044`
`FALHA NA MISSÃO` (glow + pisca)	centro y=220	38px	`#ff0044`
Linha separadora	x=200→600 y=252	—	`#ff0044` alpha 0.4
`PONTUAÇÃO FINAL:`	x=210 y=275	15px	`#884455`
valor score 6 dígitos	x=590 y=275	15px	`#ff6688`
`ONDAS SOBREVIVIDAS:`	x=210 y=309	15px	`#884455`
valor `wave-1`	x=590 y=309	15px	`#ff6688`
`CLASSIFICAÇÃO:`	x=210 y=343	15px	`#884455`
rank string	x=590 y=343	15px	`#ff6688`
Botão `[ RECOMEÇAR ]` (pisca)	centro y=400	20px	`#ff0044`
Sistema de ranking (`_getRank`)
```
score >= 3000 → 'CAPITÃO ABISSAL'
score >= 1500 → 'COMANDANTE'
score >= 700  → 'OPERADOR'
score >= 200  → 'RECRUTA'
default       → 'PEIXE MORTO'
```
Input
`pointerdown` ou `SPACE` → `GameScene`
---
12. PALETA DE CORES GLOBAL
Uso	Cor hex
Cor primária / UI principal	`#00ffcc` (ciano bioluminescente)
Score / destaques	`#ffcc00` (âmbar)
Profundidade / info secundária	`#4488aa` (azul aço)
Labels HUD	`#005544` (verde submarino)
Alerta / Game Over	`#ff0044` (vermelho crítico)
Fundo oceano topo	`#244fc7`
Fundo oceano base	`#041238`
Areia	`#c2a56c`
Projétil inimigo	`#ee44ff` + `#cc00ff` (violeta abissal)
---
13. ANIMAÇÕES E TWEENS RECORRENTES
Alvo	Tipo	Parâmetros
Título do menu	alpha pulse	1→0.85, 1800ms, sine, yoyo, repeat=-1
Botão `[ MERGULHAR ]`	alpha blink	1→0.3, 600ms, stepped, yoyo, repeat=-1
Botão `[ RECOMEÇAR ]`	alpha blink	1→0.4, 500ms, yoyo, repeat=-1
Título GameOver	alpha pulse	1→0.7, 700ms, yoyo, repeat=-1
Corais com swing	angle oscillate	±2.5°, 2000–3400ms, sine, yoyo, repeat=-1
Criaturas do menu	y float	y-12, 2800–4200ms, sine, yoyo, repeat=-1
Último coração	alpha pulse	1→0.7, 900ms, sine, yoyo, repeat=-1
Coração ao tomar dano	scale+alpha	scale=1.6, alpha=0, 250ms, Back.easeIn
Câmera ao tomar dano	shake	120ms, intensity=0.012
Transição menu→jogo	fadeOut/fadeIn	400ms
---
14. REGRAS DE IMPLEMENTAÇÃO
Tudo em um único `.html` — sem arquivos externos além do CDN do Phaser e Google Fonts.
Inicializar `enemyFireTimer` ANTES de `spawnWave()` para evitar erro de referência nula.
Inicializar todos os grupos de física ANTES de criar qualquer sprite que dependa deles.
Scanlines CSS devem usar `height: 548px` (não `inset: 0`), para não cobrir a areia.
Profundidade do jogador: depth=5. Inimigos: depth=3. Projéteis: depth=4. HUD: depth 20–22.
O parallax é redesenhado a cada frame (chamada em `update()`); não usar sprites estáticos.
A velocidade dos inimigos é recalculada após cada morte com o multiplicador de escassez.
`fadeIn(400)` na câmera ao iniciar `GameScene.create()`.
O `chest` (baú do tesouro) fica posicionado na zona central como elemento focal — não balança.
---
15. CONFIGURAÇÃO DO PHASER
```javascript
new Phaser.Game({
  type: Phaser.AUTO,
  width: 800,
  height: 600,
  parent: 'game-container',
  backgroundColor: '#000510',
  physics: { default: 'arcade' },
  scene: [BootScene, MainMenu, GameScene, GameOver]
});
```