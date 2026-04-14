# GAME FORGE — PANDA COSMIC JOURNEY

Crie um jogo completo em um único arquivo `.html` usando exclusivamente HTML5 Canvas, CSS3 e JavaScript ES6+ vanilla. Zero dependências externas. Abre e joga direto no navegador. Canvas lógico 480×640px, responsivo via `canvas.style`.

---

## Conceito

Sidescroller infinito estilo **Flappy Bird**. O jogador controla **Ping**, um panda místico que voa por 4 biomas progressivos — Floresta Encantada (pts 0–19), Templo nas Nuvens (20–44), Estratosfera Aurora (45–74) e Cosmos Profundo (75+) — cada um com paleta, parallax de 3 camadas, obstáculos e atmosfera distintos. Transições entre biomas são crossfades de 2.5s com mensagem narrativa sem interromper o jogo.

---

## Física

```js
GRAVITY = 0.38 | FLAP_FORCE = -7.5 | MAX_FALL_SPEED = 10
PIPE_SPEED = 2.8 (+0.15/10pts, máx 6.5) | PIPE_GAP = 155px (-3/10pts, mín 105)
PIPE_INTERVAL = 1600ms (-50/10pts, mín 950) | PANDA_X = width*0.22
HITBOX = raio circular 14px | delta = Math.min((t - last) / 16.67, 3)
```

Rotação: −25° ao flap → +70° em queda livre, lerp fator 0.15. Controles: Espaço/clique/tap = flap · Shift/duplo-clique/double-tap = Cosmic Burst.

---

## Mecânica Especial — Cosmic Burst

Passar por 5 obstáculos enche a barra. Ao ativar: Ping vai em tween easeOutQuad para `height/2` em 350ms, fica invulnerável por 1.2s com aura pulsante, emite 60 partículas e flash branco. Barra volta a zero. Indicador de 5 segmentos no HUD.

---

## Personagem — Ping

Desenhado 100% no Canvas: corpo elipse branca 34×30px, cabeça círculo raio 18px, manchas pretas nos olhos, orelhas semi-círculo preto/rosa, focinho bege. Três expressões: normal, flap (olhos semicerrados), game over (olhos em X). Patas animam em ciclo de 150ms (Frame A/B). Trail de 2 partículas/frame nas patas traseiras, burst de 8 ao flap, cor varia por bioma.

---

## Biomas

| Bioma | Pts | Obstáculo | Destaque visual |
|---|---|---|---|
| Floresta Encantada | 0–19 | Bambus com nós e folhas procedurais | Fireflies, flores, tons verdes |
| Templo nas Nuvens | 20–44 | Colunas de pedra dourada com glow | Sol se pondo, templos flutuantes |
| Estratosfera Aurora | 45–74 | Prismas de cristal de gelo | Aurora boreal animada com sin(time) |
| Cosmos Profundo | 75+ | Meteoros + planetas (pts ≥ 90) | Nebulosa procedural, 400+ estrelas |

Evento especial em 100pts: câmera lenta por 3s ("VELOCIDADE CÓSMICA").

---

## Pontuação, HUD e Telas

+1 ponto por obstáculo passado × multiplicador do bioma (×1/×2/×3/×5). High score em `localStorage` chave `pcj_highscore`. HUD no Canvas: score (topo esq), best (topo dir), nome do bioma (topo centro), barra Cosmic Burst (base centro). Texto flutuante "+N" sobe e faz fade a cada ponto.

Telas: **menu** com Ping flutuando animado · **game over** com painel translúcido, stats e cooldown de 800ms antes de aceitar input · **transição de bioma** com overlay colorido sobre o jogo em execução.

---

## Efeitos e Áudio

Screen shake (morte: intensity 14 · Burst: intensity 5), flashes (vermelho na morte, branco no Burst), 25 partículas de explosão na morte, vinheta permanente nas bordas. Áudio via Web Audio API (AudioContext lazy no primeiro input): flap, ponto, morte, Cosmic Burst, transição de bioma e jingle de recorde — todos sintetizados com osciladores e envelope ADSR.

---

## Implementação

Object pooling obrigatório para obstáculos e partículas. Código em 19 seções comentadas (Config → State → Audio → Biomes → Draw → Particles → Effects → Physics → Obstacles → Burst → Background → HUD → Input → Update → Render → Loop → Init). Prioridade: física → obstáculos → biomas → Ping → Burst → game feel → áudio → telas.