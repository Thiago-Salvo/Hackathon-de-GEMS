"Crie um jogo completo em HTML, CSS e JavaScript mocênicamente TOTALMENTE INSPIRADO em Space Invaders, com temática brasileira, mecânicas modernas e código bem estruturado, legível e robusto.
obrigat´rio no MINIMO 500 linhas.
Objetivo do jogo
O jogador controla um personagem na parte inferior da tela e deve derrotar ondas de inimigos que descem gradualmente. O objetivo é sobreviver o máximo possível e alcançar a maior pontuação.
Tema brasileiro
Utilize elementos que remetam ao Brasil de forma criativa e estilizada:
Personagem principal: pode ser uma capivara.
Inimigos: mosquitos da dengue, figuras caricatas ou elementos urbanos.
Projéteis: objetos temáticos, como chinelo, latas ou sprays.
Cenário: cidade brasileira em estilo simples ou pixel art.
Mecânicas principais
Movimento horizontal suave do jogador (teclas A/D ou setas)
Disparo com tecla (espaço)
Sistema de cooldown para evitar disparos contínuos ilimitados
Inimigos se movem lateralmente e disparam uma particula que tira vida da capivara
Sistema de colisão confiável (bounding box)
Sistema de vidas (por exemplo, 3 vidas)
Game Over ao perder todas as vidas
Mecânicas adicionais (implementar pelo menos três)
Power-ups, como tiro duplo, escudo ou desaceleração do tempo
Inimigos com comportamentos variados (zig-zag, mais rápidos, que atiram)
Progressão de fases com aumento de dificuldade, mas já deve começar numa dificuldade razoável.
Um chefe (boss) com mais vida e padrões de ataque distintos.
Sistema de combo ou multiplicador de pontos
Barra de energia especial
Estrutura do código
Separar claramente HTML, CSS e JavaScript
Utilizar requestAnimationFrame para o loop principal
Organizar o código em classes ou objetos (por exemplo: Player, Enemy, Bullet, Game)
Evitar variáveis globais desnecessárias
Incluir comentários explicando as partes mais importantes
Interface
Tela inicial com opção de iniciar o jogo
Tela de Game Over exibindo a pontuação final
Interface durante o jogo mostrando:
vidas
pontuação
nível ou fase
Layout adaptável a diferentes tamanhos de tela
Requisitos técnicos
Evitar múltiplos loops ao reiniciar o jogo
Remover corretamente elementos fora da tela (balas, inimigos)
Prevenir vazamentos de memória
Garantir que a detecção de colisão não prejudique o desempenho
Utilizar deltaTime para manter consistência de movimento
Manter taxa de quadros estável
Extras (opcional)
Armazenamento de pontuação local com localStorage
Efeitos sonoros básicos (tiro, acerto, fim de jogo)
Animações simples
Entrega
Código completo pronto para execução no navegador
Pode ser um único arquivo ou arquivos separados (index.html, style.css, script.js)
Preferencialmente sem dependências externas
Observações finais
O jogo deve funcionar imediatamente ao abrir o arquivo HTML. Priorize clareza, organização e desempenho. Evite código redundante ou soluções improvisadas. Considere todo o fluxo de uso: iniciar jogo, jogar, perder e reiniciar.
"