"**Crie um jogo 2D para navegador chamado *Cyber Dash*, inspirado no estilo de gameplay de Flappy Bird, porém com identidade própria, ambientado em um universo cyberpunk neon, usando apenas HTML, CSS e JavaScript. O jogo deve ser totalmente funcional no navegador, responsivo e com experiência fluida tanto em desktop quanto em dispositivos móveis.**

---

# CYBER DASH — Prompt Oficial de Desenvolvimento

## Objetivo geral

Desenvolver um arcade viciante, dinâmico e rejogável, com progressão de dificuldade, efeitos visuais intensos e sistema de sobrevivência baseado em habilidade e estratégia.

O jogador controla um drone/nave cibernética que atravessa corredores neon perigosos em alta velocidade, desviando de obstáculos futuristas enquanto sobrevive o máximo possível.

---

## Direção visual e temática

O jogo deve ter estética **cyberpunk futurista neon**, incluindo:

* cidades futuristas iluminadas ao fundo;
* prédios holográficos e luzes urbanas animadas;
* paleta neon: roxo, azul elétrico, rosa magenta, ciano;
* brilhos e reflexos digitais;
* HUD holográfica;
* efeitos glitch leves;
* partículas luminosas;
* design futurista tecnológico.

### Personagem:

O personagem principal deve ser:

* um drone futurista;
  ou
* nave compacta neon;
  ou
* pássaro robótico cibernético.

---

## 1. Tela inicial (Start Screen)

A tela inicial deve conter:

* título estilizado: **CYBER DASH**
* animação neon pulsante no logo;
* botão “PLAY” iluminado;
* instruções:

  * Clique / toque / espaço = subir
  * Shift = dash
  * P = pause
* melhor pontuação salva localmente;
* fundo animado cyberpunk com paralaxe.

---

## 2. Mecânica principal

* Movimento horizontal automático contínuo.
* Jogador sobe ao clicar, tocar ou pressionar espaço.
* Gravidade constante puxa para baixo.
* Física suave e responsiva.
* Movimento deve parecer fluido e moderno.

---

## 3. Sistema de HP (vida)

* Exibir barra holográfica de HP.
* Colisões causam dano.
* Não morrer instantaneamente ao tocar obstáculos.
* Obstáculos diferentes causam danos variados.
* Ao zerar HP → tela Game Over.

Exemplo:

* barreira comum = -1 HP
* laser intenso = -2 HP
* drone hostil = -3 HP

---

## 4. Dash (mecânica especial)

Implementar dash com:

* ativação via Shift;
* impulso rápido para frente;
* invulnerabilidade curta;
* cooldown visível;
* efeito visual de rastro neon;
* brilho energético durante dash.

---

## 5. Obstáculos dinâmicos

Obstáculos devem incluir:

### Tipos:

* pilares neon móveis;
* lasers verticais;
* drones inimigos;
* portões mecânicos que abrem/fecham;
* barreiras industriais móveis.

### Comportamentos:

* movimento horizontal constante;
* alguns sobem/descem;
* alguns alternam posição;
* alguns aparecem aleatoriamente.

---

## 6. Progressão de dificuldade

A dificuldade deve aumentar gradualmente:

* obstáculos mais rápidos;
* espaços menores;
* padrões mais complexos;
* spawn mais frequente.

### Escalada visual:

Quanto maior a pontuação:

* cenário fica mais intenso;
* luzes mais fortes;
* fundo mais caótico.

---

## 7. Sistema de pontuação

Ganhar pontos por:

* sobreviver ao tempo;
* ultrapassar obstáculos;
* sequências perfeitas;
* usar dash eficientemente.

### bônus:

* combo sem dano;
* passagem central perfeita;
* dash evasivo preciso.

---

## 8. Power-ups

Itens coletáveis ocasionais:

### Tipos:

* cura HP;
* escudo temporário;
* slow motion;
* dash instantâneo;
* multiplicador de pontos.

Todos devem:

* brilhar neon;
* flutuar com animação;
* ter efeito visual chamativo.

---

## 9. Feedback visual

Adicionar:

* tremor de câmera ao sofrer dano;
* flash ao colidir;
* tela piscando em perigo crítico;
* partículas ao pontuar;
* glitch leve em momentos intensos.

---

## 10. Sons

Adicionar efeitos sonoros para:

* colisão;
* dash;
* pontuação;
* coleta de power-up;
* game over.

Estilo:
sons eletrônicos futuristas.

---

## 11. Pause

Botão ou tecla pause:

* congelar jogo completamente;
* congelar obstáculos;
* congelar animações;
* exibir menu:

  * continuar;
  * reiniciar;
  * voltar menu.

---

## 12. Game Over

Tela final deve mostrar:

* mensagem “SYSTEM FAILURE”;
* pontuação final;
* recorde;
* botão “PLAY AGAIN”;
* botão “MAIN MENU”.

Visual:
efeito glitch neon futurista.

---

## 13. Interface responsiva

Compatível com:

* desktop;
* mobile;
* tablets.

### Mobile:

Adicionar botões touch:

* subir;
* dash;
* pause.

---

## 14. Extras desejáveis

### Fundo animado:

* paralaxe multilayer;
* prédios neon se movendo;
* drones ao fundo.

### Conquistas locais:

* sobreviver 30 segundos;
* atingir 100 pontos;
* vencer sem dano por 20 segundos.

### Extras premium:

* ranking local;
* skins desbloqueáveis;
* modo endless hardcore.

---

## 15. Estrutura do código

Separar em:

* `index.html`
* `style.css`
* `script.js`

Requisitos:

* código modular;
* bem comentado;
* fácil expansão futura.

---

## Entrega esperada

O resultado deve ser:

* totalmente jogável no navegador;
* visualmente impactante;
* altamente viciante;
* com identidade própria forte.

---

## Critério de qualidade

**Cyber Dash deve parecer um arcade cyberpunk premium**, com:

* visual neon marcante;
* gameplay rápido;
* sensação moderna;
* efeitos satisfatórios;
* alta rejogabilidade.

---

## Importante

O jogo NÃO deve parecer apenas um clone simples de Flappy Bird.
Ele deve transmitir identidade original, personalidade própria e acabamento profissional."