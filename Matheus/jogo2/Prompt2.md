1. VISÃO GERAL
Crie um jogo completo em um único arquivo HTML chamado `FLAPPY MASS`.
É uma variante de Flappy Bird com mecânica central de mudança de massa (shrink),
estética neo-brutalista (fundo industrial escuro, tipografia bold, sombras duras,
paleta amarela/ciano/vermelho) e progressão de fases por score.
Use Phaser 3 (CDN: `https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.min.js`).
Canvas: 420 × 700 px, pai `#game-wrap`.
---
2. CONSTANTES GLOBAIS
```javascript
const WIDTH              = 420;
const HEIGHT             = 700;
const PIPE_WIDTH         = 72;
const CAP_H              = 20;         // altura do capitel (boca do pipe)
const BASE_PIPE_SPEED    = 210;        // px/s base
const PIPE_SPACING_PX    = 300;        // distância horizontal entre pares
const HOLD_THRESHOLD     = 180;        // ms para distinguir tap de hold
const SHRINK_MIN_COOLDOWN= 500;        // cooldown mínimo após uso brevíssimo
const SHRINK_DURATION    = 1800;       // duração máxima do shrink em ms
const SHRINK_COOLDOWN    = 3000;       // cooldown máximo (uso total)
const FLOOR_Y            = HEIGHT - 68;
const MIN_PIPE_H         = 60;
```
---
3. ESTADOS DO JOGO
```javascript
const GAME_STATE = { READY: 'ready', PLAYING: 'playing', DEAD: 'dead' };
```
---
4. CSS / APRESENTAÇÃO
```css
:root {
  --bg: #141414; --panel: #0b0b0b; --ink: #f4f4f4;
  --accent: #f7df1e; --accent2: #39d5ff; --danger: #ff4d4d;
}

html, body {
  margin: 0; width: 100%; height: 100%; overflow: hidden;
  background: radial-gradient(circle at top, #2b2b2b 0%, #171717 45%, #0c0c0c 100%);
  font-family: "Courier New", monospace;
  display: flex; align-items: center; justify-content: center;
}

#game-wrap {
  position: relative;
  box-shadow: 18px 18px 0 #000;   /* sombra dura neo-brutalista */
  border: 4px solid #fff;
  background: #111;
}

canvas { display: block; }
```
---
5. ASSETS GERADOS VIA CÓDIGO
Todos gerados em `generatePlayerTextures(scene)` com `scene.make.graphics({ add: false })`.
Pipes, mísseis e todos os elementos de UI são `scene.add.rectangle()` — sem texturas.
Chave	Tamanho	Descrição
`player-big`	34×34	`fillRoundedRect(0,0,34,34,r=6)` amarelo `0xf7df1e` + stroke branco 4px
`player-small`	18×18	`fillRoundedRect(0,0,18,18,r=5)` ciano `0x39d5ff` + stroke branco 3px
---
6. ESTRUTURA DE FUNÇÕES (uma única cena Phaser)
O jogo usa uma única cena com as funções globais:
```
preload()             → vazia
create()              → orquestra toda inicialização
update()              → loop principal
generatePlayerTextures()
createBackground()
createPlayer()
createGroups()
createHUD()
createCollisions()
setupInput()
startRun()
spawnPipePair()
createPipe()
calcSpawnDelay()
updatePipeCleanup()
updatePipeCollision()
updatePlayerAngle()
updatePlayerTrail()
updateDangerIndicator()
updateParallax()
updatePhaseVisuals()
updatePhaseDecor()
applyPhase()
updateHUD()
jump()
activateShrink()
deactivateShrink()
spawnMissileWithWarning()
launchMissile()
createGateMarker()
renderReadyMessage()
showStartMenu()
hideStartMenu()
showDeathMenu()
hideDeathMenu()
triggerGameOver()
restartScene()
```
---
7. INICIALIZAÇÃO (`create`)
Ordem obrigatória:
```javascript
1. Inicializar variáveis de estado no contexto `this`:
   this.lastClickTime = 0
   this.shrinkLocked = false
   this.shiftKey = addKey(SHIFT)
   this.state = GAME_STATE.READY
   this.score = 0
   this.best = Number(localStorage.getItem('mass_shift_best') || 0)
   this.holdTimer = null
   this.shrinkStartTime = 0
   this.tunnelWarningActive = false
   this.shrinkActive = false
   this.shrinkCooldownActive = false
   this.pipeSpeed = BASE_PIPE_SPEED
   this.currentPhase = -1
   this.pipePairsPassed = 0
   this.lastPipeY = HEIGHT / 2
   this.segmentType = 'normal'
   this.segmentTimer = -1
   this.playerTrail = []
   this.decalLines = []

2. generatePlayerTextures(scene)
3. createBackground(scene)
4. createGroups(scene)
5. createPlayer(scene)
6. createHUD(scene)
7. createCollisions(scene)
8. setupInput(scene)
9. Criar barra de energia (energyFrame, shrinkBarBg, shrinkBarFill) — ver §11
10. Criar timers (missileSpawner, pipeSpawner, scoreTimer) — todos paused:true
11. renderReadyMessage(scene)
```
---
8. BACKGROUND (`createBackground`)
Camadas (3 tileSprites sobrepostas, sem textura inicial)
Gerar cada textura com `scene.make.graphics({ add: false })`, depois aplicar:
bg-back (420×700):
`fillRect(0,0,W,H)` cor `0x2a2a2a`
45 edifícios: `fillRect(x, H-h, 34, h)` cor `0x262626`, x = (i83)%W, h = 90+(i13)%180
Grid horizontal: linhas `0x333333` a cada 70px
bg-mid (420×700):
Fundo transparente
10 edifícios maiores: `fillRect(x, H-h, 52, h)` cor `0x3a3a3a`, stroke `0xffffff` alpha 0.25
x = i70+20, h = 130+(i29)%160
bg-front (420×700):
Chão: `fillRect(0, FLOOR_Y, W, H-FLOOR_Y)` cor `0x1f1f1f`
Faixas alternadas a cada 24px: brancas (`0xffffff` alpha 0.18) e cinzas (`0x2a2a2a`)
Parallax (atualizado em `updateParallax` a cada frame)
```javascript
bgBack.tilePositionX  += speed/60 * 0.18
bgMid.tilePositionX   += speed/60 * 0.45
bgFront.tilePositionX += speed/60 * 1.0
// speed = pipeSpeed quando PLAYING, 1.4 quando READY
```
Estrelas decorativas
20 círculos brancos (r=1–2, alpha 0.15–0.5, depth=5) com tween de piscar:
`alpha 0.1→0.6`, duração 800–2000ms, yoyo, repeat=-1.
Efeitos globais (depth alto)
`scene.vignette` — rectangle preto 420×700, alpha=0, depth=30 (sobe com perigo)
`scene.flash` — rectangle branco 420×700, alpha=0, depth=31 (flash no pulo)
`scene.scanlines` — graphics com linhas brancas alpha 0.04 a cada 4px, depth=29
---
9. PLAYER (`createPlayer`)
```javascript
scene.player = scene.physics.add.sprite(110, HEIGHT*0.45, 'player-big')
  .setDepth(20)
  .body.setAllowGravity(false)   // inicia parado
  .body.setSize(28, 28)
  .body.setOffset(3, 3)

// Propriedades custom:
player.baseGravity   = 1250
player.shrinkGravity = 900
player.jumpForceBig  = -355
player.jumpForceSmall = -285
```
Decoradores do player:
```javascript
scene.playerGlow    = add.circle(x, y, 26, 0xf7df1e, 0.12).setDepth(19)
scene.playerOutline = add.rectangle(x, y, 40, 40).setStrokeStyle(2, 0xffffff, 0.28).setDepth(18)
scene.playerDanger  = add.rectangle(x, y, 44, 44).setStrokeStyle(3, 0xff4d4d, 0).setDepth(21)
```
Todos reposicionados a cada frame em `updatePlayerAngle()`.
Trail do player (`updatePlayerTrail`)
A cada frame durante PLAYING: cria círculo na posição atual do player.
Normal: r=7, cor `0xf7df1e`, alpha=0.35
Shrink: r=4, cor `0x39d5ff`, alpha=0.35
Tween: move -10 a -25px em X, fade alpha→0, scale→0.2, 300ms
Máximo 15 trails simultâneos
Ângulo do player (`updatePlayerAngle`)
```javascript
player.angle = Clamp(body.velocity.y * 0.045, -28, 80)
playerGlow.radius = shrinkActive ? 18 : 24
playerOutline.width = playerOutline.height = shrinkActive ? 28 : 40
```
---
10. PIPES (sistema sem physics body)
CRÍTICO: Pipes são `scene.add.rectangle()` sem `physics.add.existing()`.
A colisão é feita por AABB manual em `updatePipeCollision()`.
A movimentação é feita manualmente em `updatePipeCleanup()`.
Criar pipe (`createPipe`)
```javascript
// Pipe principal
const pipe = scene.add.rectangle(x, y, w, h, 0x1a1a1a)
  .setStrokeStyle(2, 0xffffff, 0.35).setDepth(15)
pipe.passScored = false
pipe.isBottom = isBottom
pipe.scored = false
pipe.speedX = speed
scene.pipes.add(pipe)   // grupo simples (não physics)

// Cap (boca do gap) — sempre do lado do gap
const capY = isBottom ? y - h/2 - CAP_H/2 : y + h/2 + CAP_H/2
const cap = scene.add.rectangle(x, capY, w+16, CAP_H, 0x2c2c2c)
  .setStrokeStyle(3, 0xffffff, 0.9).setDepth(16)
cap.passScored = true
cap.scored = false
cap.speedX = speed
scene.pipes.add(cap)
```
Spawn de par (`spawnPipePair`)
Segmentos: a cada chamada, `segmentTimer--`. Quando chega a 0, sorteia novo tipo:
```javascript
const segs = ['normal', 'wave', 'tight', 'shrinkTunnel']
segmentTimer = Between(5, 9)
```
Gap por tipo:
Tipo	Gap (px)	Variação Y
`normal`	185 - t*55 (min 130)	±65
`wave`	158	sin(time*0.0025)*75
`tight`	125	±30
`shrinkTunnel`	105	±20
t = `Math.min(1, score/1200)` — fator de progressão 0→1.
Velocidade dinâmica:
```javascript
const speed = Math.round(165 + Math.pow(t, 1.6) * 140)  // 165→305 px/s
```
Delay do spawner:
```javascript
calcSpawnDelay(speed) = Math.round((PIPE_SPACING_PX / speed) * 1000)
```
Posição Y:
```javascript
let centerY = lastPipeY + variation
centerY = Clamp(centerY, lastPipeY-90, lastPipeY+90)  // MAX_STEP = 90
centerY = Clamp(centerY, MIN_PIPE_H + gap/2, FLOOR_Y - MIN_PIPE_H - gap/2)

topH        = centerY - gap/2
bottomStartY = centerY + gap/2
bottomH     = FLOOR_Y - bottomStartY
// Abort se topH < 60 ou bottomH < 60
```
Spawnar em `x = WIDTH + PIPE_WIDTH/2 + 10`.
Aviso de shrinkTunnel: ao selecionar o segmento, exibir `shrinkText` com
`'PREPARE-SE: ENCOLHE!'`, tween de flash 5× (alpha 1↔0.3, 280ms each).
Movimentação e limpeza (`updatePipeCleanup`)
```javascript
// A cada frame:
pipe.x -= (pipe.speedX || pipeSpeed) / 60

// Pontuação ao cruzar o player:
if (!pipe.passScored && !pipe.scored && pipe.x + pipe.width/2 < player.x) {
  pipe.scored = true
  score += shrinkActive ? 5 : 12
  updateHUD()
}

if (pipe.x < -150) pipe.destroy()
```
Colisão AABB manual (`updatePipeCollision`)
```javascript
const hw = shrinkActive ? 7 : 11   // half-width da hitbox
const hh = shrinkActive ? 7 : 11   // half-height da hitbox

// Para cada pipe ativo:
if (pRight > pipeLeft && pLeft < pipeRight &&
    pBottom > pipeTop && pTop < pipeBottom) {
  triggerGameOver(scene)
}
```
---
11. BARRA DE ENERGIA (shrink)
Três objetos criados no `create()` após a HUD:
```javascript
// Moldura
scene.energyFrame = add.rectangle(W/2, H-60, 132, 14, 0x111111, 0.65)
  .setStrokeStyle(1, 0xffffff, 0.18).setDepth(50)

// Fundo da barra
scene.shrinkBarBg = add.rectangle(W/2, H-60, 120, 8, 0x222222, 0.85).setDepth(51)

// Preenchimento (origin 0, 0.5 — cresce da esquerda)
scene.shrinkBarFill = add.rectangle(W/2 - 60, H-60, 120, 8, 0x39d5ff, 1)
  .setOrigin(0, 0.5).setDepth(52)
```
Durante shrink ativo:
```javascript
usedMs   = now - shrinkStartTime
remaining = 1 - (usedMs / SHRINK_DURATION)
shrinkBarFill.width = Math.max(0, remaining * 120)
if (usedMs >= SHRINK_DURATION) → deactivateShrink(scene, force=true)
```
Cores da barra:
Ativa: `0x00ff88`
Em recarga: `0xff0055` (tween de width 0→120 na duração do cooldown)
---
12. MECÂNICA DE SHRINK
Ativar (`activateShrink`)
Condições: `PLAYING && !shrinkActive && !shrinkCooldownActive`
```javascript
shrinkActive = true
shrinkStartTime = now

player.setTexture('player-small')
player.body.setSize(18, 18).setOffset(2, 2)
player.body.setGravityY(shrinkGravity)          // 900
player.body.setVelocityY(body.velocity.y * 0.4) // amortecer queda
cameras.main.shake(80, 0.003)
shrinkText.setText('SHRINK ACTIVE').setAlpha(1)
```
Desativar (`deactivateShrink`)
Parâmetro `force=false`: ignora `shrinkLocked` se false
Cooldown proporcional ao tempo de uso real:
```javascript
const usedMs  = now - shrinkStartTime
const ratio   = Math.min(1, usedMs / SHRINK_DURATION)
const cooldown = Math.round(SHRINK_MIN_COOLDOWN + ratio * (SHRINK_COOLDOWN - SHRINK_MIN_COOLDOWN))
// Faixa: 500ms (uso brevíssimo) → 3000ms (uso total)
```
```javascript
shrinkActive = false
shrinkLocked = false
player.setTexture('player-big')
player.body.setSize(28, 28).setOffset(3, 3)
player.body.setGravityY(baseGravity)   // 1250
shrinkCooldownActive = true

// Tween de recarga da barra:
tween(shrinkBarFill, { width: 120 }, cooldown, 'Linear')
  .onComplete → shrinkCooldownActive = false, shrinkText.setAlpha(0)
  + tween playerOutline alpha 0↔1 × 2 (feedback de recarga completa)
```
---
13. SISTEMA DE INPUT
Mouse / Touch (`setupInput`)
```
pointerdown:
  → DEAD   → restartScene()
  → READY  → startRun()
  → PLAYING:
      delta = now - lastClickTime
      lastClickTime = now

      SE delta < 300ms (clique duplo):
        cancela holdTimer
        → activateShrink (shrinkLocked = true)
        return

      SE não: inicia holdTimer (180ms)
        → se expirar sem pointerup: activateShrink (shrinkLocked = false)

pointerup:
  SE holdTimer ainda ativo:
    cancela holdTimer → jump()
  SENÃO SE shrinkActive && !shrinkLocked:
    → deactivateShrink()
```
Teclado
`SPACE down`: jump() (também inicia run / reinicia)
`SHIFT down`: activateShrink (shrinkLocked = true)
Pulo (`jump`)
```javascript
vy = shrinkActive ? jumpForceSmall (-285) : jumpForceBig (-355)
player.body.setVelocityY(vy)

// Burst visual (3 círculos):
for 0..2:
  circle branco r=5, alpha=0.4
  tween → x-=10..30, alpha→0, scale→0.2, 200ms

// Squash:
tween player → scaleX: shrinkActive?0.67:1.08, scaleY: shrinkActive?0.75:0.9, 85ms yoyo

// Flash:
flash.alpha = 0.05 → tween alpha→0, 120ms
```
---
14. SISTEMA DE FASES VISUAIS
Score define a fase. `applyPhase()` só executa se `currentPhase !== phaseId`.
Score	Fase	BG Color	Accent
0–149	0	`0x161616`	`0xf7df1e` (amarelo)
150–399	1	`0x001a33`	`0x39d5ff` (ciano)
400–799	2	`0x2b0000`	`0xff4d4d` (vermelho)
800+	3	`0x051a05`	`0x39ff14` (verde neon)
Ao trocar de fase:
```javascript
cameras.main.flash(500, 255,255,255, alpha=0.3)
cameras.main.setBackgroundColor(bgColor)
bgBack/bgMid/bgFront.setTint(bgColor)
playerGlow.fillColor = accentColor
tween(scoreText, { scale: 1.3 }, 300ms, yoyo)
```
Decor de fase (`updatePhaseDecor`): a cada frame, 5% de chance de spawnar
um retângulo branco (20–60px × 2px, alpha 0.08) que desloca com os pipes.
---
15. MÍSSEIS
Aparecem quando `score > 150`, spawner a cada 3000ms.
Aviso (`spawnMissileWithWarning`)
```javascript
// Triângulo vermelho piscando na borda direita
const warning = add.triangle(WIDTH-30, targetY, 0,20, 20,10, 0,0, 0xff4d4d)
tween → alpha 0↔1, 150ms, repeat=5
onComplete → launchMissile()
```
Lançamento (`launchMissile`)
```javascript
const missile = add.rectangle(WIDTH+50, y, 45, 20, 0xff4d4d)
  .setStrokeStyle(2, 0xffffff).setDepth(25)
physics.add.existing(missile)
missiles.add(missile)
missile.body.setAllowGravity(false)
missile.body.setVelocityX(-(pipeSpeed * 3))

// Trail: 1 rect principal + 6 peças em cascata
missile.trail = add.rectangle(x+25, y, 18, 8, 0xf7df1e, 0.45).setDepth(24)
for i 0..5:
  trailPieces[i] = add.rectangle(x+25+i*10, y, 16-i*2, 8-i, 0xffd54a, 0.40-i*0.05).setDepth(23)
```
Trail reposicionado em `updatePipeCleanup` a cada frame.
Colisão: `physics.add.overlap(player, missiles, triggerGameOver)`.
---
16. GATE MARKER
Criado a cada par de pipes:
```javascript
const zone = add.rectangle(x+2, centerY, 12, gap-8, 0xffffff, 0.05)
  .setStrokeStyle(2, 0xffffff, 0.18).setDepth(12)
phaseMarks.add(zone)
// Move junto com os pipes em updatePipeCleanup
```
---
17. TIMERS
```javascript
// Spawner de pipes (delay dinâmico)
pipeSpawner = time.addEvent({
  delay: calcSpawnDelay(BASE_PIPE_SPEED),
  callback: spawnPipePair, loop: true, paused: true
})

// Score (a cada 100ms)
scoreTimer = time.addEvent({
  delay: 100, loop: true, paused: true,
  callback: () => {
    score += shrinkActive ? 0.6 : 1.35
    updateHUD()
  }
})

// Mísseis (a cada 3s, condicionado a score > 150)
missileSpawner = time.addEvent({
  delay: 3000, loop: true, paused: true,
  callback: () => { if (score > 150) spawnMissileWithWarning() }
})
```
`startRun()`: despausa os três timers, `player.body.setAllowGravity(true)`,
`setGravityY(baseGravity)`, chama `spawnPipePair()` imediatamente.
---
18. HUD
Todos os textos usam `fontFamily: 'Arial'` (não Courier — o Courier é só no CSS body).
Painel superior
```javascript
hudTop = add.rectangle(W/2, 36, W-24, 52, 0x000000, 0.28)
  .setStrokeStyle(2, 0xffffff, 0.10).setDepth(40)
hudTopGlow = add.rectangle(W/2, 12, W-40, 2, 0xffffff, 0.10).setDepth(41)
```
Campos dinâmicos
Campo	Posição	Tamanho	Cor inicial
`scoreText` `'DEPTH: 0m'`	x=18, y=18	24px bold	`#ffffff` (muda por fase)
`bestText` `'BEST: Xm'`	x=W-18, y=20, origin(1,0)	16px bold	`#bdbdbd`
`shrinkText` (contextual)	centro, y=H-108	16px bold	`#39d5ff`
`hintText` fixo	centro, y=H-94	14px	`#6f6f6f`
Cores do score por fase:
Fase 0: `#f7df1e` · Fase 1: `#39d5ff` · Fase 2: `#ff4d4d` · Fase 3: `#39ff14`
Cor do shrinkText:
Normal: `#39d5ff` · Em recarga: `#ff4d4d`
---
19. MENU INICIAL (neo-brutalista)
Todos os elementos criados em `createHUD()` com `setVisible(false)`.
Exibidos com fade+easing em `showStartMenu()`.
Estrutura visual
```
startShadow     → rect 310×300, cor 0xf7df1e, offset +6/+8 (sombra dura)
startPanel      → rect 310×300, cor 0x101010, stroke branco 4px
startTopBar     → rect 310×24, cor 0x39d5ff (barra superior)
startTopLabel   → texto 'BOOT SEQUENCE' 12px preto, centrado na barra
startTitleGlow  → texto 'FLAPPY\nMASS' 34px ciano, alpha pulsante (glow)
startTitle      → texto 'FLAPPY\nMASS' 34px branco, stroke preto 6px
startSubtitle   → texto 'SOBREVIVA AOS TÚNEIS' 14px #bdbdbd
startJumpBox    → rect 118×62, cor 0xf7df1e, stroke preto (instrução pulo)
startShrinkBox  → rect 118×62, cor 0x39d5ff, stroke preto (instrução shrink)
startJumpText   → 'BARRA\nPULO' 18px bold preto
startShrinkText → 'BARRA 2x\nENCOLHER' 18px bold preto
startButtonShadow → rect 220×44, ciano, offset +6/+6
startButton     → rect 220×44, preto, stroke branco 3px
startButtonText → 'Clique para iniciar' 18px bold branco
startDecoLeft   → rect 18×18, amarelo (canto sup. esq.)
startDecoRight  → rect 18×18, ciano (canto inf. dir.)
```
Animações do menu
```javascript
// Sombra oscila horizontalmente
tween([startShadow, startButtonShadow], { x: '+=3' }, 220ms, yoyo, repeat=-1

// Glow do título pulsa
tween(startTitleGlow, { alpha: 0.08→0.22 }, 900ms, yoyo, repeat=-1

// Decorações giram
tween([startDecoLeft, startDecoRight], { angle: 180 }, 1400ms, repeat=-1

// Botão pulsa de escala
tween(startButton, { scaleX: 1.03, scaleY: 1.03 }, 700ms, yoyo, repeat=-1
```
---
20. MENU DE GAME OVER
Mesma lógica de `showStartMenu` mas com paleta vermelha e dados de score.
Estrutura visual
```
deathShadow     → rect 300×250, ciano (sombra dura)
deathPanel      → rect 300×250, 0x111111, stroke branco 4px
deathTopBar     → rect 300×24, 0xff0055 (barra vermelha)
deathTopLabel   → 'FALHA NO SISTEMA' 12px preto
deathTitleGlow  → 'NÚCLEO\nCOLAPSADO' 29px ciano (glow)
deathTitle      → 'NÚCLEO\nCOLAPSADO' 28px branco, stroke preto 6px
deathScoreBox   → rect 110×58, 0xf7df1e (caixa amarela)
deathBestBox    → rect 110×58, 0x39d5ff (caixa ciano)
deathScoreText  → 'FINAL\n{score}' 18px bold preto
deathBestText   → 'BEST\n{best}' 18px bold preto
deathRestartShadow → rect 210×42, ciano
deathRestartBtn    → rect 210×42, preto, stroke branco
deathRestartText   → 'Clique para reiniciar' 18px bold branco
deathDecoLeft   → rect 18×18, ciano
deathDecoRight  → rect 18×18, vermelho
```
Animações: idênticas ao menu inicial.
---
21. GAME OVER (`triggerGameOver`)
Guard: `if (state === DEAD) return`
```javascript
state = DEAD
pipeSpawner.paused = true
scoreTimer.paused = true
missileSpawner.paused = true

finalScore = Math.floor(score)
if (finalScore > best) → best = finalScore, localStorage.setItem(...)

// 10 fragmentos explosivos radiais:
for i 0..9:
  angle = (i/10) * PI * 2
  dist  = Between(40, 120)
  frag  = rect(player.x, player.y, 4..10, 4..10, color) depth=50
  tween → x+cos(a)*dist, y+sin(a)*dist, alpha→0, scale→0.1, 500ms, Quad.easeOut

physics.pause()
cameras.main.shake(300, 0.005 + pipeSpeed/BASE * 0.008)
showDeathMenu(scene, finalScore)
```
Reinício: `pointerdown` ou `SPACE` quando `DEAD` → `scene.restart()`
---
22. INDICADOR DE PERIGO (`updateDangerIndicator`)
```javascript
// Para cada pipe ativo (não caps):
dx = |pipe.x - player.x| - (pipe.width + 34) / 2
dy = |pipe.y - player.y| - (pipe.height + 34) / 2
nearest = min(nearest, max(dx, dy))

alpha = nearest < 90 ? Clamp((90-nearest)/90, 0, 0.9) : 0
playerDanger.setStrokeStyle(3, 0xff4d4d, alpha)
vignette.setAlpha(alpha * 0.18)
```
---
23. PALETA GLOBAL
Elemento	Cor
Background neutro	`#161616`
Jogador normal	`#f7df1e` (amarelo)
Jogador shrink	`0x39d5ff` (ciano)
Pipes	`0x1a1a1a` + stroke `0xffffff` 0.35
Caps dos pipes	`0x2c2c2c` + stroke `0xffffff` 0.9
Mísseis	`0xff4d4d`
Trail míssil	`0xf7df1e` / `0xffd54a`
Perigo / morte	`0xff0044` / `0xff0055`
Sombra neo-brutalista	`#000000` sólido, offset 18px
Borda painéis	`#ffffff` 4px sólido
---
24. REGRAS DE IMPLEMENTAÇÃO
Pipes não usam physics.add.existing() — colisão é AABB puro em `updatePipeCollision`.
Mísseis usam physics — `physics.add.existing()` + `overlap()` com o player.
`shrinkLocked` previne `deactivateShrink` ao soltar o mouse quando ativado por duplo-clique.
Cooldown proporcional: shrink curto = cooldown curto; duração total = cooldown máximo.
Segmento `shrinkTunnel`: aviso é emitido no momento do sorteio, não na chegada do pipe.
Score via timer (a cada 100ms) + bônus por pipe cruzado. Shrink reduz ambos.
`pipeSpawner.delay` é atualizado dinamicamente se a diferença for > 40ms.
`state` é a única fonte de verdade — todo update condiciona a ele.
Todos os menus são criados em `createHUD()` e gerenciados por `show/hide` com tweens.
RecordPessoal salvo em `localStorage` com chave `'mass_shift_best'`.
---
25. CONFIGURAÇÃO DO PHASER
```javascript
new Phaser.Game({
  type: Phaser.AUTO,
  parent: 'game-wrap',
  width: WIDTH,          // 420
  height: HEIGHT,        // 700
  backgroundColor: '#161616',
  physics: {
    default: 'arcade',
    arcade: { gravity: { y: 0 }, debug: false }
  },
  scene: { preload, create, update }
})
´´´
