"**Objetivo:** Criar um jogo 2D completo, autocontido e pronto para rodar no navegador chamado **`Alien Data Swarm`**, inspirado na progressão e ritmo de *Space Invaders*, mas com identidade cyberpunk/digital própria, mecânicas modernas e suporte nativo a desktop e mobile.

---

#### TEMA & ESTÉTICA
- **Narrativa:** Vírus alienígenas digitais invadiram a World Wide Web. O jogador controla um `Firewall Móvel` armado no ciberespaço.
- **Cenário:** Grade digital infinita com perspectiva isométrica sutil, partículas de código hexadecimal/binário flutuando lentamente, paleta cyberpunk (preto profundo, ciano elétrico, magenta neon, verde terminal, branco glitch).
- **Elementos Visuais:** Tudo gerado proceduralmente via Canvas API (nenhuma imagem externa). Inimigos com formas geométricas/criptográficas, projéteis com trail neon, efeitos de scanline e distorção digital sutil.

---

#### MECÂNICAS DE JOGO
1. **Formação Inimiga:** Grid de vírus que se move lateralmente e desce progressivamente. Velocidade e frequência de tiro aumentam conforme o score.
2. **Tiro do Jogador:** Projéteis verticais. Colisão precisa com inimigos e bordas.
3. **Ataque Especial:** Carrega automaticamente (barra de energia). Quando ativado, limpa uma faixa horizontal, causa dano em área e **remove corrupção da tela**.
4. **Mecânica de Corrupção:** 
   - Quando um inimigo toca o fundo ou é destruído sem ser atingido por tiros especiais, deixa um `bloco de corrupção` (glitch visual + sobreposição semi-transparente).
   - Acúmulo > 40% da área visível reduz a cadência de tiro do jogador em 30% e distorce a HUD.
   - Pode ser limpo pelo ataque especial ou destruindo inimigos `Nó de Corrupção` (destaque magenta).
5. **Estados do Jogo:** Menu Inicial → Jogando → Pausa → Game Over → Reinício.
6. **Pontuação & High Score:** Score por destruição, bônus por limpar corrupção, persistência via `localStorage`.

---

#### CONTROLES
- **Desktop:** 
  - `←` / `→` : Mover horizontalmente
  - `Espaço` : Atirar
  - `Shift` : Ataque Especial (quando carregado)
- **Mobile:**
  - Toque/arrastar nas metades esquerda/direita da tela : Mover
  - Toque rápido em qualquer lugar : Atirar
  - Duplo toque ou swipe para cima : Ataque Especial
- **Regras de Input:** Prevenir `scroll`, `zoom`, `contextmenu`. Usar `touch-action: none` e mapeamento por zonas. Suporte a múltiplos toques.

---

#### 🛠️ REQUISITOS TÉCNICOS
- **Tecnologia:** HTML5 `<canvas>`, CSS3, JavaScript ES6+ puro. Zero dependências.
- **Renderização:** `requestAnimationFrame`, delta time para física estável em 60 FPS.
- **Responsividade:** Canvas escala proporcionalmente à viewport, mantendo aspect ratio 4:3 ou 16:9 (definir um e ajustar com letterboxing/pillarboxing CSS). Recalcula posições no `resize`.
- **Performance:** 
  - Object pooling para projéteis, partículas e inimigos.
  - Colisão via AABB otimizada.
  - Limpeza de arrays/entidades inativas.
- **Áudio:** Web Audio API para sons procedurais (tiro, explosão, especial, corrupção, game over). Sem arquivos externos.
- **Acessibilidade/UX:** HUD clara, feedback visual de dano/carga, instruções na tela inicial, botão de reinício visível.

---

#### ESTRUTURA DO CÓDIGO ESPERADA
```html
<!DOCTYPE html>
<html lang=""pt-BR"">
<head>
  <meta charset=""UTF-8"">
  <meta name=""viewport"" content=""width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"">
  <title>Alien Data Swarm</title>
  <style>/* CSS completo: reset, canvas fullscreen, UI overlay, glitch effects, responsividade */</style>
</head>
<body>
  <canvas id=""gameCanvas""></canvas>
  <div id=""ui-layer"">/* HUD, menus, score, barra de especial, mensagens */</div>
  <script>
    /* JS modularizado via IIFE ou classes:
       - Config & Constants
       - InputManager (keyboard + touch)
       - AudioManager (Web Audio procedural)
       - Entity (Player, Enemy, Bullet, Corruption, Particle)
       - Game (state machine, loop, spawn, collision, rendering)
       - UI (DOM sync, localStorage)
       - init() & boot sequence
    */
  </script>
</body>
</html>
```

---

#### CRITÉRIOS DE ACEITAÇÃO
1. Arquivo `.html` único que roda ao abrir no navegador.
2. 60 FPS estáveis em desktop e ≥30 FPS em mobile médio.
3. Controles funcionam simultaneamente em teclado e toque.
4. Corrupção visual e mecânica implementada conforme especificação.
5. Código comentado, sem variáveis globais soltas, com tratamento de erros básico.
6. Nenhuma requisição externa (fontes, imagens, áudios, CDN).
7. Testado em Chrome, Firefox e Safari mobile.

**Instrução final para a IA/Desenvolvedor:** 
Gere o código completo em um único bloco, pronto para copiar/colar. Inclua comentários explicando a arquitetura, onde ajustar balanceamento e como estender o jogo. Priorize legibilidade, performance e experiência fluida em telas pequenas. Não omita sistemas críticos (loop, input, colisão, estados, responsividade)."