# GAME FORGE — RED PANDA INVADERS
## Prompt de Geração de Jogo Completo

> **Entrega:** Um único arquivo `.html` funcional — sem bibliotecas externas, sem imagens externas, sem fontes externas. Abre e joga diretamente no Chrome/Firefox.

---

## 1. RESTRIÇÕES ABSOLUTAS

| Regra | Valor |
|---|---|
| Tecnologia | HTML5 Canvas + CSS + JavaScript vanilla puro |
| Dependências externas | **ZERO** |
| Arquivos de saída | **1 único arquivo `.html`** |
| Placeholders / TODOs | **PROIBIDOS** — tudo implementado |

---

## 2. ARQUITETURA TÉCNICA

### 2.1 Canvas e Escala
- Tamanho lógico fixo: **480 × 620 px**
- Centralizado na página via CSS; escalonado para preencher a janela mantendo proporção
- Fundo da página: `#000000`

### 2.2 Game Loop
```
requestAnimationFrame → mede deltaTime → normaliza velocidade
```
- **Tudo que se move multiplica sua velocidade por `deltaTime`** — nunca velocidade bruta por frame
- `deltaTime` tem cap máximo para evitar saltos quando a aba fica em background

### 2.3 Máquina de Estados (5 estados, exaustivos)
```
MENU → PLAYING → PHASE_CLEAR → PLAYING (próxima fase)
                             → GAME_OVER
                             → VICTORY (após fase 4)
```
- Lógica de jogo **somente em PLAYING**
- HUD de jogo **somente em PLAYING e PHASE_CLEAR**
- Nenhum estado pode executar código de outro estado

### 2.4 Inicialização de Fase
- Função: `initPhase(n)` — recebe número da fase (1–4)
- Reseta: posições, velocidades, timers, barreiras, power-ups, arrays de projéteis e partículas
- **Nunca chamada durante PLAYING** — apenas ao entrar nele
- Ao (re)iniciar jogo: score=0, vidas=3, fase=1, todos os arrays limpos, HYPE=0

### 2.5 Organização do Código (seções comentadas, nessa ordem)
```
Config global → Estado do jogo → Utilitários → Audio → Partículas →
Entidades visuais (funções de desenho) → Lógica de inimigos → Lógica do boss →
Lógica de fases → Colisões → Input → HUD e telas → Update principal →
Render principal → Init e Loop
```

---

## 3. JOGADORA — RUBI

### 3.1 Movimento e Posição
- Controles: `←` `→` / `A` `D`
- Travada nas bordas do canvas (0 a 480px)
- Altura vertical **fixa**, próxima ao fundo, acima do HUD inferior

### 3.2 Tiro
- Disparo: `ESPAÇO` ou clique/toque
- Cooldown: **320ms**
- **Máx 1 projétil na tela por vez** — input ignorado se projétil em voo

### 3.3 Vidas e Dano
- Começa com **3 vidas**
- Ao ser atingida: −1 vida + **2s de invulnerabilidade** (pisca rápido)
- Durante invulnerabilidade: projéteis inimigos passam através
- 0 vidas → estado `GAME_OVER`

### 3.4 Visual (Canvas 2D puro, sem imagens)

| Parte | Descrição |
|---|---|
| Corpo | Arredondado, vermelho-escuro `#AA2200` |
| Cabeça | Levemente maior, posicionada acima e à frente |
| Orelhas | Triangulares pontudas, interior creme |
| Olhos | Grandes, expressivos, brilho branco |
| Focinho | Bege claro |
| Cauda | Longa, listrada, **oscila em sine wave contínua** |
| Braços | Dois curtos, estendidos, segurando bambu verde |
| Estado de dano | Olhos = traços fechados, boca levemente aberta |

**Paleta:** `#AA2200` (dominante) · `#F5DEB3` (creme) · `#111111` (preto) · `#228B22` (verde)

### 3.5 Projétil de Rubi
- Retângulo vertical fino, verde-escuro
- Marcação de nó no meio (traço transversal mais escuro)
- Sobe em linha reta até colisão ou topo do canvas

---

## 4. SISTEMA DE INIMIGOS

### 4.1 Entrada na Tela
- Formação começa **acima do canvas** (Y negativo, todos invisíveis)
- Desce verticalmente até a posição inicial (abaixo do HUD)
- **Sem tiros durante a descida**
- Velocidade de entrada varia por fase:

| Fase | Tempo de entrada |
|---|---|
| 1 | ~3.0s |
| 2 | ~2.2s |
| 3 | ~1.6s |
| 4 (boss) | ~2.5s |

### 4.2 Estrutura do Grid
- Grid rígido com **âncora única** (canto superior esquerdo)
- Posição de cada inimigo = âncora + (coluna × cellW, linha × cellH)
- Mover o grid = mover só a âncora
- Cada inimigo armazena: `coluna`, `linha`, `vivo/morto`, `tipo`, `HP`, `timerAnimação` (offset por coluna para dessincronizar)

### 4.3 Movimento Lateral (após entrada)
```
A cada frame:
1. Calcula posição FUTURA dos inimigos vivos mais à esquerda e mais à direita
2. SE posição futura direita > (480 - 8px):
   → inverte direção
   → âncora.y += 18px  ← aplicado NO MESMO FRAME, antes do movimento
3. SE posição futura esquerda < 8px: idem
4. Aplica movimento horizontal
5. SE qualquer inimigo vivo atingiu a altura de Rubi → GAME_OVER
```
- Velocidade lateral base (por fase) + **incremento fixo por inimigo morto** (tensão clássica)

### 4.4 Tiro dos Inimigos
- Timer global por fase (intervalo fixo)
- Ao disparar: escolhe coluna aleatória com ≥1 inimigo vivo → atira do inimigo com maior Y nessa coluna
- **Limite absoluto de 4 projéteis inimigos simultâneos** — se limite atingido, nenhum projétil criado naquele ciclo

### 4.5 Detecção de Fim de Fase
```
Final de cada frame de update (somente em PLAYING):
  SE inimigos vivos == 0:
    estado = PHASE_CLEAR
    timer interno = 0  ← conta com deltaTime, NÃO usa setTimeout
    
  SE timer > 2.5s:
    SE fase < 4: initPhase(fase + 1), estado = PLAYING
    SE fase == 4: estado = VICTORY
```
- **Transição sempre automática** — nunca espera input do jogador

---

## 5. FASES

### FASE 1 — Sala da Tadi
> Sala de aula universitária bagunçada. Tom cômico. Paleta quente e iluminada.

| Parâmetro | Valor |
|---|---|
| Grid | 9 colunas × 3 linhas = **27 inimigos** |
| Vel. lateral base | Baixa |
| Timer de tiro | 1600ms |

**Inimigos — Linhas 1–2 (Cadeiras com Bug):**
- Cadeira universitária de frente: assento arredondado marrom-médio, encosto marrom-escuro, patas nos lados
- Bug: a cada 90 frames → 6 frames de aberração cromática (contorno vermelho deslocado direita, azul esquerda)
- Oscilação leve lateral em sine wave (offset por coluna)
- Vale **10 pts**

**Inimigos — Linha 3 (Mesas com Bug):**
- Mesa escolar: tampo retangular marrom, patas em L, monitor miniatura sobre o tampo
- Monitor: moldura cinza, tela azul royal, texto pixelado "X_X" e "ERRO" em branco, pisca azul↔azul-escuro
- Vale **20 pts**

**Projétil:** bolinhas cyan 6px com glow suave, linha reta

**Cenário:**
- Paredes creme envelhecido
- Quadro negro (terço superior) com marcas de giz simuladas (linhas finas brancas baixa opacidade)
- Janelas laterais (moldura branca, interior azul claro)
- Piso cinza claro com grid de rejunte

**Power-up exclusivo — Cadeira Voadora** (spawn ao matar 14 inimigos = 50%):
- Visual: cadeira miniatura caricata em amarelo dourado brilhante, cai do topo
- Efeito: projétil horizontal com trajetória levemente parabólica, elimina todos os inimigos tocados
- A cadeira gira levemente durante o voo

---

### FASE 2 — Campus da USP
> Gramado aberto, tarde. Paleta azul e verde.

| Parâmetro | Valor |
|---|---|
| Grid | 9 colunas × 4 linhas = **36 inimigos** |
| Vel. lateral base | Moderada |
| Timer de tiro | 1200ms |

**Inimigos — Linhas 1–2 (Pombos):**
- Corpo oval cinza-médio com brilho, cabeça circular, bico laranja triangular, olhos pretos+brilho
- Asas = dois triângulos nas laterais alternando posição alta/baixa (animação de batida)
- Vale **15 pts**

**Inimigos — Linhas 3–4 (Ratos):**
- Corpo oval cinza-escuro, cabeça redonda, orelhas grandes rosa, olhos vermelhos com glow
- Dois dentes brancos projetados para baixo, cauda curvada, postura de estudante
- Vale **25 pts**

**Projétil:** bolinhas brancas 7px com contorno cinza claro, linha reta

**Cenário:**
- Gradiente de céu azul claro (topo) → tom quente (horizonte)
- Gramado verde-médio (quarto inferior)
- 3 silhuetas de prédios universitários cinza-azulado com janelas em grid
- 2 árvores estilizadas (tronco marrom + copa arredondada verde-escuro)

**Power-up exclusivo — Bomba de Metano** (spawn a 50% kills):
- Visual: bolha verde brilhante, caveira pixelada dentro, partículas saindo, pulsação lenta
- Efeito: identifica a linha com mais inimigos vivos → elimina todos dessa linha
- Visual: onda verde semitransparente se expande de Rubi em ambas as direções

---

### FASE 3 — Bambuzal
> Floresta densa de bambu à noite. Clima sombrio e opressivo. Paleta verde-escura com roxo.

| Parâmetro | Valor |
|---|---|
| Grid | 10 colunas × 4 linhas = **40 inimigos** |
| Vel. lateral base | Alta |
| Timer de tiro | 900ms |
| Drop power-up global | **35%** (aumentado pela dificuldade) |

**Inimigos — Linhas 1–2 (Pandas Malignos):**
- Mesma estrutura de Rubi, paleta invertida: corpo preto, manchas cinza-escuro, olhos vermelhos glow intenso
- Sobrancelhas em diagonal (raiva), boca firme
- **Vibração aleatória de 1–2px** em todas as direções a cada frame (efeito de possessão)
- Vale **20 pts**

**Inimigos — Linhas 3–4 (Pandas com Armadura de Bambu):**
- Base igual ao Panda Maligno + tiras de bambu verde sobre torso e braços (retângulos verdes com nós)
- **2 HP:** 1º acerto → armadura some visualmente, aparência fragilizada; 2º acerto → morte
- Enquanto armados: atiram orbes maiores (10px) e mais rápidas
- Vale **35 pts**

**Projétil:** orbes roxas 7px com glow (armados: 10px, mais rápidas)

**Cenário:**
- Fundo verde muito escuro, quase preto
- Colmos de bambu verticais distribuídos: retângulos verde-escuros com linhas horizontais (nós)
- Folhas de bambu no topo dos colmos: formas longas, finas e curvadas
- Névoa baixa no terço inferior: verde-escuríssimo, baixa opacidade

**Sem power-up exclusivo nesta fase.**

---

### FASE 4 — Sala do Digi (BOSS)
> Espaço digital sombrio. Um único chefão colossal.

#### O Sintetizado — Visual
| Elemento | Descrição |
|---|---|
| Tamanho | ~35% da largura do canvas |
| Corpo | Retangular arredondado, azul royal `#0033AA`, borda interna glow ciano `#00FFFF` |
| Olhos | 2 hexágonos grandes ciano, shadowBlur pulsante |
| Boca | Retangular larga, dentes irregulares em azul-escuro + contorno ciano |
| Superfície | Linhas de circuito ciano baixa opacidade (padrão ortogonal assimétrico) |
| Braços | 2 largos com garras triplas, saem dos lados inferiores |
| Pulsação | Escala oscila ±2% em ciclo de 2s (respira/processa) |
| **HP** | **60** |

**Dano progressivo:**
| HP | Estado visual |
|---|---|
| < 45 | Rachaduras vermelhas no corpo |
| < 30 | Corpo roxo-escuro + olhos mais vermelhos |
| < 15 | Pisca vermelho/roxo alternado + rachaduras multiplicadas |

**Barra de HP:** larga, logo abaixo do HUD · começa azul `#0044FF` → vermelho nos últimos 30% · label "SINTETIZADO" centralizado

#### Comportamento — 3 Fases de Ataque

| Fase | HP | Movimento | Ataque | Limite proj. |
|---|---|---|---|---|
| 1 | 60–41 | Lateral lento (terço superior) | 5 proj. retos aleatórios a cada 1.2s | 6 |
| 2 | 40–21 | Mais rápido | Leque de 3 proj. a cada 1s + laser de varredura a cada 3s | 8 |
| 3 | 20–1 | Máximo | Leque de 5 proj. a cada 0.8s + todos os ataques + varredura a cada 2s + proj. 25% mais rápidos | 10 |

**Laser de Varredura (fases 2 e 3):**
1. Linha horizontal amarela pulsante aparece por 1s como aviso (Rubi deve se mover)
2. Laser dispara por 0.6s → Rubi na linha = perde vida

**Limite de projéteis é absoluto:** se atingido, nenhum novo projétil criado.

#### As 3 Gemas
Spawnam individualmente em X aleatório na metade inferior do canvas. Flutuam em sine wave. Duram 7s (piscam nos últimos 2s).

| Gema | Cor | Aparece quando boss atinge |
|---|---|---|
| 1 | Vermelha `#FF2200` | 45 HP |
| 2 | Verde `#00FF44` | 30 HP |
| 3 | Azul `#2244FF` | 15 HP |

**Coletar as 3 → PANDA GIGANTE (permanente até fim da fase):**
- Escala visual 1.8×
- Aura vermelha pulsante
- Cooldown de tiro reduzido para 180ms
- Bambu causa **2 de dano** ao boss por acerto
- Mensagem "PANDA GIGANTE!" centralizada por 2s

**Cenário:**
- Fundo preto absoluto
- Grid de linhas finas azul-escuríssimo `#001133` por todo o canvas
- ~60 pontos brancos 1–2px espalhados, alguns piscam aleatoriamente
- Glow azul-ciano suave nas bordas do canvas

---

## 6. POWER-UPS GLOBAIS

- **Drop:** 28% por inimigo morto, dividido igualmente entre os 5 tipos
- **Comportamento:** caem em linha reta da posição do inimigo morto
- **Aviso:** piscam nos últimos 2s antes de sair do canvas
- **Na fase 3:** drop aumentado para 35%

| Nome | Visual | Efeito |
|---|---|---|
| **Bambu Grande** | Bambu grosso, contorno dourado | Próximos 5 tiros atravessam a coluna inteira (não param no 1º inimigo) |
| **Estrelas Ninja** | Shuriken dourada em rotação | Próximas 4 salvas disparam 3 bambus em leque (central reto + ±20°) |
| **Pergaminho dos Clones** | Pergaminho bege com detalhes vermelhos | Cria 1 clone translúcido que espelha todos os inputs; desaparece ao ser atingido sem custar vida |
| **Peças de Computador** | Chip verde com LED piscando | Escudo semicircular ciano absorve 1 projétil (dura 1 acerto ou 30s) |
| **Papagaio** | Papagaio verde, bico laranja | 3 papagaios orbitam Rubi; cada um absorve 1 projétil e desaparece |

---

## 7. MODO HYPE

### Acúmulo
- Contador de kills consecutivas **sem tomar dano**
- Tomar dano → contador vai a **zero imediatamente**
- **12 kills** → medidor cheio, pisca "SHIFT!"

### Ativação (`SHIFT` com medidor cheio)
- Duração: **2.5s**
- Efeito: **feixe vertical contínuo** (Rubi → topo do canvas)
  - Não são projéteis — é um **raio persistente** que verifica colisão a cada frame
  - Elimina todos os inimigos e projéteis inimigos que tocar
  - Rubi fica **imóvel** durante o feixe
- Visual do feixe: centro branco brilhante + bordas amarelo/laranja + faíscas laterais contínuas + tint amarelo sutil em toda a tela

### Pós-HYPE
- Timer vai a zero
- Cooldown de **4s** (barra cinza com "AGUARDE")
- Sem acumular kills durante cooldown

### HUD do HYPE
- Barra horizontal com **12 segmentos** amarelos abaixo do score
- Cheia: pulsa com glow + "SHIFT!" piscando ao lado
- Cooldown: cinza + "AGUARDE"

---

## 8. BARREIRAS

- **4 barreiras** entre Rubi e inimigos, espaçamento uniforme horizontal
- Compostas por segmentos individuais em grid
- Cada segmento: **2 HP**
  - 1º acerto → cor escurece + rachadura diagonal
  - 2º acerto → segmento **removido permanentemente**
- Bloqueiam projéteis **nos dois sentidos** (Rubi e inimigos)
- **Restauradas completamente** ao iniciar cada nova fase

| Fase | Cor/Material |
|---|---|
| 1–2 | Madeira (marrom) |
| 3 | Bambu (verde) |
| 4 | Metal ciano |

---

## 9. PONTUAÇÃO E HIGH SCORE

| Inimigo | Pontos |
|---|---|
| Cadeira com Bug | 10 |
| Mesa com Bug | 20 |
| Pombo | 15 |
| Rato | 25 |
| Panda Maligno | 20 |
| Panda com Armadura | 35 |

- High score salvo em `localStorage` com chave `rpi_highscore`
- Salvo sempre que score atual superar durante a partida
- "NOVO RECORDE!" exibido em dourado pulsante no Game Over e Victory

---

## 10. HUD

**Faixa semi-transparente escura no topo (sempre visível em PLAYING):**
```
[SCORE: XXXXX]  [BEST: XXXXX]  [FASE X / 4]  [♥ ♥ ♥]
```

**Abaixo da faixa:**
```
[■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■]  ← Barra HYPE 12 segmentos
```

**Fase 4 — abaixo do HYPE:**
```
[SINTETIZADO ████████████░░░░]  ← Barra HP boss
```

**Canto inferior esquerdo:**
- Ícone + contador do power-up temporário ativo

---

## 11. TELAS

### Menu
- Fundo preto + partículas caindo lentamente
- Título "RED PANDA INVADERS" em vermelho bold com sombra forte, centralizado no topo
- Rubi em tamanho maior, animada (cauda oscilando, piscada ocasional)
- High score em amarelo
- "ESPAÇO / CLIQUE para começar" piscando

### Phase Clear
- Overlay escuro semitransparente
- "FASE N CONCLUÍDA!" em verde brilhante
- Nome da próxima fase em branco
- Score parcial
- **Timer 2.5s automático** → próxima fase (sem input)

### Game Over
- Overlay preto 80%
- "GAME OVER" em vermelho grande
- Score da partida
- Se recorde: "NOVO RECORDE!" dourado pulsante
- Delay obrigatório **1.5s** antes de mostrar "ESPAÇO / CLIQUE para recomeçar"

### Victory
- Chuva de partículas coloridas
- "VITÓRIA!" dourado grande
- "O Sintetizado foi derrotado!" branco
- Score final ciano
- Opção de recomeçar após 2s

---

## 12. EFEITOS VISUAIS

| Efeito | Trigger | Descrição |
|---|---|---|
| Partículas de morte | Inimigo morto | 8–10 quadrados na cor do inimigo, explodem e fazem fade em ~25 frames |
| Texto flutuante | Kill / power-up | "+Npts" sobe e faz fade em 0.7s; nome do power-up sobe sobre Rubi |
| Screen shake | Rubi toma dano | Offsets aleatórios decrescentes por 12 frames |
| Screen shake intenso | Boss morto | Shake por 20 frames |
| Flash vermelho | Rubi toma dano | Overlay vermelho suave, 5 frames |
| Flash amarelo | HYPE ativado | Overlay amarelo suave, 4 frames |
| Flash branco | Gema coletada | Overlay branco suave, 3 frames |
| Idle inimigos | Sempre | Animação sutil com offset por coluna (assíncrono/orgânico) |

---

## 13. AUDIO (Web Audio API, zero arquivos externos)

- `AudioContext` criado **lazy** — somente após primeiro input do usuário
- Osciladores com envelope de amplitude (ataque + decaimento suaves)
- Verificar se AudioContext existe antes de qualquer uso

| Som | Característica |
|---|---|
| Tiro de bambu | Curto, agudo |
| Morte de inimigo | Breve, médio |
| Dano recebido | Grave, mais longo |
| Coleta de power-up | Arpejo ascendente, positivo |
| Ativação do HYPE | Sweep de frequência dramático |
| Projétil do boss | Grave, ameaçador |
| Morte do boss | Impacto grave, decaimento longo |
| Transição de fase | Arpejo ascendente de 3 notas |

---

## 14. REGRAS DE ROBUSTEZ (INVIOLÁVEIS)

### Fim de fase
```
- Verificação "todos mortos": 1× por frame, no final do update, SOMENTE em PLAYING
- Timer usa deltaTime do game loop — NUNCA setTimeout
- Detecção → muda estado imediatamente → timer começa
- Timer > 2.5s → initPhase(n+1) automático
```

### Borda do grid
```
- Checar posição FUTURA (projetada) antes de aplicar qualquer movimento
- Violação detectada → inverter direção + descer 18px NO MESMO FRAME
- Só então aplicar movimento horizontal
```

### Limite de projéteis
```
- Antes de criar projétil: contar quantos do tipo estão ativos
- Limite atingido → NÃO criar, sem fila, sem exceção
- Inimigos comuns: máx 4 | Boss fase 1: máx 6 | Boss fase 2: máx 8 | Boss fase 3: máx 10
```

### Reset de jogo
```
Ao iniciar ou recomeçar: score=0, vidas=3, fase=1
Limpar: projéteis[], partículas[], power-ups[], clones, shields, orbitais
Resetar: HYPE=0, cooldowns, timers
```

### Limpeza de arrays
```
A cada frame: filtrar projéteis[], partículas[], power-ups[] removendo inativos
Impede acúmulo indefinido de objetos
```

---

## 15. PRIORIDADE DE IMPLEMENTAÇÃO

Implementar nesta ordem — nunca avançar sem o anterior funcionar:

1. **Loop + estados + transições automáticas** (timer deltaTime, sem setTimeout)
2. **Grid entrando do topo, movendo lateral, atirando (limite 4), avançando de fase automaticamente**
3. **Colisões corretas em todos os pares:**
   - Projétil de Rubi × inimigo
   - Projétil inimigo × Rubi
   - Projétil × barreira (ambos os sentidos)
   - Rubi × power-up
4. **4 fases com visuais distintos e configurações corretas**
5. **Modo HYPE completo** (feixe, counter, cooldown)
6. **Power-ups com efeitos corretos**
7. **Arte expressiva com identidade visual forte por fase**