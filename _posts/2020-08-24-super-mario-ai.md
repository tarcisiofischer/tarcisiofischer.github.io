---
layout: post
title: Um protótipo de IA que joga Super Mario
image: 2020-08-24/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-08-24/img001.png)
{: refdef}

Pouco tempo depois de terminar minha graduação, um amigo me chamou para um projeto
que envolvia a criação de uma inteligência artificial para jogar o clássico
do Super Nintendo: Super Mario World. O projeto **nunca foi concluído**, mas os
resultados preliminares ficaram razoavelmente legais. Esse post mostra um pouco
sobre o processo, ou pelo menos o que eu consigo lembrar de memória ;)


# Descrição do problema

Existem algumas formas de se abordar o problema. Uma que eu lembro que pensamos
foi a de discretizar o tempo de jogo e, para cada timestep, associar um
conjunto de botões a serem apertados no controle do console. Ao meu ver, isso
significa que o jogo se tornaria uma caixa preta, onde o input seria toda a
sequência de comandos e o output alguma métrica de quão bom a IA se comportou.
A figura abaixo ilustra essa ideia. A partir do valor de saída dessa função-jogo,
estimar-se-ia de alguma forma uma evolução para a sequencia de comandos, e se
tentaria novamente, na esperança do valor de saída decrementar (Tal qual qualquer
método de otimização).

{:refdef: style="text-align: center;"}
![](/images/2020-08-24/ideia001.png)
{: refdef}

Existiam alguns problemas nessa abordagem. O principal deles é que não fazia
sentido com o tema de pesquisa que meu colega se propunha a iniciar no mestrado
dele, sobre Inteligência Artificial orientada a programação de Agentes (AOP).
Dessa forma, essa ideia logo foi abandonada e seguimos por um caminho
completamente diferente.

Nossa nova ideia consistiu em que o agente (o personagem Mário) possuiria uma
visão própria das coisas que estavam acontecendo em sua volta e, a partir de
crenças, escolhas, etc, decidiria o que seria feito. Essa abordagem é comum
em Programação Orientada a Agentes. Dessa forma, precisavamos de informações
sobre o ambiente pra começar a seguir por esse caminho. Foi onde eu comecei a
atuar de fato no projeto.


# Arquitetura do software

Postas as regras, fui atrás de uma forma de criar uma arquitetura para
começarmos a trabalhar. Em primeiro lugar, precisavamos de um vídeo game para
a IA atuar. Ao invés de comprar um console real, optamos por usar um emulador
chamado Snes9x, escrito em C/C++. A partir de um fork do repositório publico
deles, [fiz um hack para criar um controle virtual](https://github.com/tarcisiofischer/snes9x/commit/3b40cb39e92513a8b8ff59c725a60436b341ca82))
baseado em arquivo (para fazer um "Inter-Process Communication"), e para roubar
cópias de tela, que serviria posteriormente como input cru da visão
computacional.

{:refdef: style="text-align: center;"}
![](/images/2020-08-24/arquitetura.png)
{: refdef}

A ideia seria que o software seria dividido em 3 partes: A primeira, seria o
emulador em si, de onde poderiamos apenas atuar sobre e pegar alguns dados.
A segunda, seria responsável pela visão e atuação sobre o emulador. Finalmente,
a terceira seria responsável pela inteligência artificial, decidindo o que
seria feito, dadas as informações disponíveis.

Com essa arquitetura, poderiamos inclusive dividir as tarefas de forma bem
isolada. Inclusive, se me lembro bem, as partes eram independentes de linguagem.
Tanto é, que do meu lado, eu trabalharia com C++ e Python, para os hacks no
emulador e a parte da de visão e atuação, enquanto a IA ficaria implementada
em JAVA, por conta da possibilidade de uso da biblioteca JADE.


# "Inteligência" Artificial

Em um primeiro momento, como não tinhamos a IA de fato implementada, o que foi
feito foi a implementação da visão e atuadores, que geravam comandos para o
controle virtual, no emulador.
[O código do protótipo está no GitHub](https://github.com/tarcisiofischer/SM_AIPlayer/blob/master/src/python/sm_ai/agent_vision.py),
e o que é feito, basicamente, é um loop eterno que pega a screenshot do jogo
mais recente e encontra as posições da cabeça do Mário e de blocos que impedem
a passagem do jogador.

A partir dessas informações, a IA se perguntava "Estou próximo de um bloco
que impede minha passagem?" - Se sim, era enviado o comando de pulo. Se não,
o personagem apenas continuava caminhando para frente. Essa implementação simples
já era capaz de jogar uma fase fácil do jogo (Também criada por nós, apenas
para teste). Esse resultado é mostrado na próxima seção desse post. A
implementação disso usando programação orientada a agentes é trivial, e
[pode ser vista aqui](https://github.com/tarcisiofischer/SM_AIPlayer/blob/master/src/java/sm_ai/player_agent.asl).
(Claro, [a parte de controle ainda exige uma implementação adicional](https://github.com/tarcisiofischer/SM_AIPlayer/blob/master/src/java/sm_ai/FileBasedController.java))

A visão computacional foi implementada usando um algoritmo de votação, com uma
ideia mais ou menos parecida com a do algoritmo de Hough, usado para encontrar
linhas e circulos (nas implementações clássicas). A implementação em Python
puro era extremamente lenta para ser usada em tempo real, então, optamos na
época por usar Numba para acelerar o código. Além disso, aproveitamos para
implementar uma pequena tela adicional que mostrava a visão da IA, inclusive
identificando com circulos as posições dos artefatos em questão.


# Resultados preliminares

Chegamos a criar uma fase de exemplo, muito simples e sem inimigos, onde o
único objetivo era correr até o final do level, pulando por cima dos blocos.
O resultado foi filmado e é mostrado abaixo. A esquerda, a visão diretamente
do jogo em tempo real. A direita, a visão do robô, com as camadas de visão
computacional referentes à posição do jogador e dos blocos.

{:refdef: style="text-align: center;"}
![](/images/2020-08-24/game_example.gif)
{: refdef}

O resultado é muito simples, e qualquer um com um pouco mais de experiência
sabe que o que foi feito não é nenhuma "rocket science". Mesmo assim, achei
legal documentar esse estudo de caso para não se perder no tempo :) Agradeço
ao Thiago Gelaim pelas discussões e risadas que tiramos desse projeto
incompleto.

Até a próxima!

<div>Icons made by <a href="https://www.flaticon.com/authors/freepik" title="Freepik">Freepik</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a></div>
