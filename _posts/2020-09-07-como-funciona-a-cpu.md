---
layout: post
title: Como funciona o processador?
image: 2020-09-07/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/img001.png)
{: refdef}

Eu sempre fui apaixonado pela parte de construção de computadores "do zero". Tanto a parte teórica quanto a parte prática de arquitetura de computadores é uma área que tenho muita saudade, pois parei de mexer com isso desde a época de faculdade. Volta e meia volto a arriscar um projeto aqui e acolá, seja com ARM, Arduino ou mesmo com componentes eletrônicos crus. Porém, com o mestrado, trabalho e filha pequena em casa, o tempo é reduzido, e esses hobbies acabam ficando de lado. Esse post é uma forma de revisitar essa área e, no caminho, criar um conteúdo interessante aos curiosos.

Um processador é um componente eletrônico capaz de executar operações programáveis. Geralmente, ele depende de outros componentes para poder ser utilizado, tais como uma placa mãe, memórias RAM e alguma unidade de disco rígido.

Para explicar como funciona um processador, vou fazer um caminho "longo", porém, dando saltos largos. Nesse sentido, tudo inicia com operações simples: As operações "E", "OU" e "NOT". Graças à criação dos transistores, é possível construir dispositivos eletrônicos que, dado dois valores, é capaz de calcular essas operações. Para exemplificar o que isso faz, observe a tabela abaixo. Para cada combinação de entradas (verdadeiro ou falso), os dispositivos computam uma saída:

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/and_or_tables.png)
{: refdef}

Esses componentes podem ser inclusive comprados para fazer projetos caseiros. Na prática, em projetos eletrônicos, você descobre o que cada pino faz usando o manual do produto (o "Datasheet"), e ligando essas portas à alguma fonte de alimentação. A existência de passagem de corrente representa um "verdadeiro", e a ausência representa um "falso". Dada uma "entrada" de dados, o valor de saída pode ser observado em uma "porta de saída".

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/and_gate.png)
{: refdef}

A parte interessante aqui é que, com isso, é possível construir uma abstração independente de tecnologia. Podemos usar uma representação visual para representar essas tabelas, e construir componentes mais complexos sobre elas. Com alguma astúcia, conseguimos conectar uma sequência desses componentes simples para construir, por exemplo, um componente que soma dois números.

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/and_sum.png)
{: refdef}

Mas como somar números se nossas entradas são apenas a presença ou ausência de um sinal? Aqui que entra o "sistema binário" de representação dos números. Por exemplo, para representar uma soma números entre 0 e 7, precisamos de 3 "fios" (3 bits). 3 bits nos provê $2^3=8$ combinações possíveis de valores: "000" representa 0, "001" representa 1, "010" representa 2, "011" representa 3 e assim por diante, até "111" representa 7, totalizando 8 valores.

Uma soma é feita bit-a-bit: $0+0=0$ (com resto 0), $0+1=1$ (com resto 0), $1+0=1$ (com resto 0) e $1+1=1$ (com resto 1). Note que é preciso carregar esse "resto da soma", pois não é possível representar o número $2$ com apenas um bit.

Não quero me estender sobre isso, só estou detalhando esse esse exemplo inicial como curiosidade. A ideia geral é que é possível modularizar a soma em um componente único, e representa-la como uma "caixinha" (um módulo). O mesmo é feito para outras operações como subtração, divisão, multiplicação, bit-shift, AND, OR, complemento, incremento, decremento, etc.

Ignorando os detalhes de implementação, imagine agora um novo componente que, dado um conjunto de entradas, controla qual dessas operações será processada. Colocamos, então, todas as operações comentadas anteriormente em um único módulo, enquanto esse controlador atua escolhendo qual das operações será efetivamente executada. Essa unidade capaz de executar várias operações, dado um valor de controle, é chamada ALU (Arithmetic-Logic Unit / Unidade Lógica e Aritmética).

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/alu.png)
{: refdef}

Começa a nascer uma noção de "instrução". A instrução (de forma bem grosseira) é a sequencia de bits que escolhe qual operação a ALU vai executar. Isso já dá uma breve noção sobre arquitetura de conjunto de instruções (Instruction Set Architecture - ISA).

Apesar de termos um componente capaz de fazer operações e podermos controlar essas operações, ainda seria necessário ligar vários fios em sequencias específicas para produzir a saída que queremos. Dessa fpr,a. adicionamos um novo módulo no nosso sistema: Um "banco de registradores".

Essa é a unidade de memória mais rápida que o processador tem acesso. Em geral são apenas alguns poucos bytes de memória, separados em "variáveis". Aqui podemos colocar os valores que vão servir de entrada para a ALU. Dessa forma, ao invés de precisar colocar os valores exatos para somar duas quantidades, por exemplo, podemos agora pedir para "somar o registrador A com o B e colocar o resultado no registrador C".

Da mesma forma que temos um banco de registradores para facilitar a entrada de variáveis, também é possível construir uma outra porção de memória para guardar as operações que serão executadas na ALU. Basicamente, essa memória atua como uma "tabela" de operações que deve ser executada sequencialmente, uma após a outra. Nesse momento, começamos a ter uma noção de um processador real: Uma unidade que possui uma tabela de operações, e executa uma operação após a outra. Existem alguns detalhes para se tomar cuidado, por exemplo, a necessidade da existência de um contador específico, que é incrementado depois de cada operação ser executada. Esse contador diz qual das operações será executada na tabela de operações, e é chamado de "Program Counter" (PC).

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/pc_alu_reg.png)
{: refdef}

Mas nosso processador ainda não é capaz de fazer muita coisa. Para deixar ele mais poderoso, precisamos implementar uma operação específica capaz de alterar o Program Counter. Ou seja, no meio da execução, o próprio programa pode decidir qual é a próxima operação que vai realizar. Isso lembra algum conceito de programação? Sim! Começa a nascer a noção de um "GOTO" (e por consequência as operações "if", "while", "for"...). Nosso processador começa a ter um poder maior, sendo agora uma maquina de Turing completa (Mas isso é assunto pra outro post). Pra fazer isso, existe muito detalhe que eu vou abstrair aqui, por motivos de simplificação, mas a ideia geral é essa.

Por fim, é necessário um último módulo que atualiza as entradas de todos os módulos o mais rápido possível. É o chamado "cristal oscilador", ou "clock" do processador. Quanto mais rápido esse módulo oscilar, maior a frequência de operações que conseguimos computar (Na verdade, existe um limite de projeto, pois a velocidade com que os sinais se propagam não é instantâneo). Por isso uma das métricas pra comparar processadores é verificar a sua frequência - Embora isso não seja uma métrica tão boa, pois processadores diferentes implementam ALUs diferentes, com mais ou menos operações, além de poderem ter múltiplos processadores internos (multicores ou multithreads), e ainda podem ter arquiteturas diferentes, possibilitando outros tipos de otimizações a nível de pipeline.

Essa questão de pipeline é a próxima parte importante a se explicar. Como o nosso processador agora possui uma organização mais rigorosa, podemos começar a "subir o nível", e colocar essas operações em uma sequência estrutural. Basicamente o que quero dizer é que, de forma geral, a sequência de operações que os dados precisam trilhar para chegar ao fim da execução de uma única execução de um comando é: Carrega da memória a próxima operação (Instruction Fetch), Interpreta o que aquela operação significa em termos de operações disponíveis na ALU, carregando os registradores relacionados do banco de registradores (Instruction Decode), Executa a instrução (Execute), Salva/Atualiza os dados em memória (Memory Access), Atualiza os registradores (Write Back).

{:refdef: style="text-align: center;"}
![](/images/2020-09-07/classic_pipeline.png)
{: refdef}

Essa estrutura idealizada em processadores RISC não é de fato utilizada dessa forma na prática, mas já dá uma noção ao leitor de que essas coisas não precisam ser feitas necessariamente em sequência. Com alguma astúcia, enquanto uma instrução está sendo executada, outra já poderia começar a ser carregada da memória. Essa ideia de "paralelizar" as diferentes etapas desse processo é bem ilustrada no livro "Computer Organization and Design", do Patterson & Henessy.

O autor faz a comparação com o processo de lavar roupas: Você precisa colocar a roupa na maquina de lavar, depois colocar na maquina de secar, depois dobrar e finalmente guardar no guarda-roupas. Porém, se você possui pessoas o suficiente, não parece eficiente fazer essas operações uma após a outra: Enquanto uma pessoa guarda algumas roupas, outra pessoa já dobra e, ao mesmo tempo, outras roupas já estão secando.

Claro que existem VÁRIOS detalhes nesse processo. Por exemplo, o que acontece quando temos um "GOTO" (if, while, for...) no meio desse procedimento? E o que acontece quando um registrador precisa ser salvo no local onde uma operação necessitava ter lido aquele valor? E quando uma operação precisa de um valor que está na memória principal, ou seja, longe do domínio dos registradores dentro do processador? São questões que devem ser resolvidas com cuidado (E, claro, não vou mostrar aqui).

Essa é a visão geral e simplificada do que ocorre dentro de um processador teórico (baseado em um processador MIPS, geralmente usado para estudo). Existem muitas diferenças do que eu expliquei para um processador atual. Por exemplo, não entrei no mérito de explicar arquiteturas RISC (Reduced Instruction Set Computer) contra CISC (Complex Instruction Set Computer), nem como são implementadas operações de ponto flutuante, nem as operações SIMD (Single Instruction Multiple Data) e SSE (Streaming SIMD Extensions), ou pelo menos as operações com ponto flutuante em geral (embora eu tenha um post sobre a representação de ponto flutuante no blog!), nem na parte de hierarquia de memória, com os diferentes níveis de memória cache, as SPM (scratchpad memory), a memória principal RAM (Random Access Memory), e o disco. É um mundo a parte e, talvez, um dia eu explore melhor alguma dessas questões.

Até a próxima!

