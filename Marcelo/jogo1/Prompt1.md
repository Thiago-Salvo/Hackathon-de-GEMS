"BugStar, enviarei aqui um prompt de um jogo que deve ser jogado através de sua interface, ele é baseado no Flappy Bird, mas com algumas alterações que você precisa implementar.
Nome do Jogo: Flappy Bird 2077
Contexto:
O ambiente é Cyberpunk 
FlappyBird está em um período que o planeta vive um caos ambiental e o oxigênio é escasso
 Espalhado pelo mapa, existe uma nuvem tóxica que exige a reposição do oxigênio consumido pelo personagem em seu traje anti toxina.
Luzes de Neon roxas e azuis estão cintilando pela paisagem, jutamente de prédios em ruína que marcam uma civilização em declínio
 Seres humanos andam ao fundo do mapa com alterações biotecnológicas, braços, pernas, olhos biônicos, os cérebros de alguns deles foram transplantados em corpos mecânicos, como forma de sobreviver ao caos. 
Roteiro:
O espaço ocupado pelo personagem é baseado em um plano cartesiano
   De maneira horizontal, cada obstáculo que aparece ocupa um valor crescente na reta dos números reais
   Simule todos os itens e o espaço no personagem na reta dos numeros reais
   Se o oxigenio estiver atras do Flappy Bird, nao será possivel pegar o oxigenio
   Se o combustivel estiver atras do Flappy Bird, nao será possivel pegar o combustivel
   Se nao for possivel pegar o oxigenio, não exiba que é possivel pegar ele
   Se nao for possivel pegar o combustivel, nao exiba que é possível pegar ele
   O personagem deve se mover horizontalmente
   O sentido do movimento se dá da esquerda para a direita
   O sentido do movimento não pode ser da direita para a esquerda
   Não dê dicas sobre o que acontecerá se o jogador Pular ou Aguardar
   O Flappy Bird deve ser menor que a abertura do cano
O Flappy Bird necessita realizar pulos que permitam passar entre canos com espaços abertos.
O Flappy Bird passará por esse espaço em aberto.
Os espaços em aberto podem estar em diferentes alturas de cada cano 
O objetivo do Flappy Bird é chegar em sua nave espacial, sendo utilizada para ir embora da atmosfera planetária tóxica. 
A nave possui algumas limitações e precisa de combustível, sendo colhido durante o processo de passagem pelos espaços abertos dos canos. 
No total, devem ser coletadas 12 unidades de combustível.
Recordando o ambiente, o oxigenio é escasso, o Flappy Bird também precisa coletar oxigenio pelo caminho até chegar em sua nave são e salvo.
Objetivo:
0 - O jogo possui movimento linear, supondo que o personagem ocupe um espaço numa reta de numeros reais, ele nao pode voltar a um valor menor que o atual ocupado.
1 - O jogador deve digitar ""Pular"" ou ""P"" para subir. Caso queira realizar um mergulho, deve digitar ""A"" ou ""Aguardar""
2 -Se não pular, o pássaro perde altitude constantemente.
3 - Colidir com um cano, teto ou com o chão resulta em Game Over imediato.
4 - Aparecem, aleatoriamente, unidades de combustiveis pelo mapa ao decorrer do jogo.
5 - A entrada do Flappy Bird em sua nave ocorre apenas após a coleta dos 12 combustíveis
6 - Ao terminar de coletar o combustível, entre na nave espacial e saia do planeta
7 - Toque um som de missão cumprida quando chegar até à nave, semelhante ao Mario Bros quando completa uma missão
8 - Continue exibindo o cano mesmo quando o Flappy Bird estiver na mesma posição que ele
9 - Painel de Status Exigido: Após cada ação, exiba:
   Pontuação: [X] canos ultrapassados.
   Oxigênio: [X]% restante.
   Próximo obstáculo: Distância e tipo (Cano, Combustive ou Aro de Oxigênio).
   
Modelo:
   Cada ação deve ser representada graficamente
   As imagens devem seguir um modelo de pixel art
   A temática das imagens é em 8-bit
"