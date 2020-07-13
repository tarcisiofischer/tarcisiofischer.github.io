---
layout: post
title: De volta ao básico - Ponteiros em C++
image: 2020-07-13/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/img001.png)
{: refdef}


Esse vai ser um post diferente dos outros. Apesar do tema ser muito básico, 
tenho a impressão que muita gente tem dúvidas sobre ele. Ponteiros está
praticamente no topo da lista de tópicos que eu ouço as pessoas terem
dificuldade.

Mesmo com muita referência pela internet, acho que esse post é
relevante. Vou tentar ser o mais didático possível, pra eu poder usar esse
post como referência a próxima vez que alguém me perguntar sobre ponteiros.
Então, se você é uma dessas pessoas, espero que esse post te ajude um pouco
a colocar uma luz nesse tema!


# O que são ponteiros?

Ponteiro é um número. Esse número representa um endereço. **Fim.**

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/end.gif)
{: refdef}

Brincadeiras à parte, vamos continuar nesse raciocínio...

Sabe quando você está em uma rua, e precisa encontrar uma casa? Você olha o
endereço dessa casa para encontra-la. A casa possui um número. Com esse número,
você pode encontrar a tal da casa e entrar... A ideia é a mesma para ponteiros.

Imagine que a memória é uma grande rua. Cada conjunto de 8 bits é uma "casa".
Cada casa possui um número. Esse número é o endereço.

Mas eu sei: Isso é muito abstrato. O que tem a ver casas e ruas com programação
e ponteiros? Se as coisas são tão simples por que não consigo vê-las? Bom...
Você pode.

**Vamos pra parte prática**. Na próxima imagem, deixei um código em C++ onde declaro,
na linha 3, uma variável chamada "var" e defino o valor "42" à ela.
Na linha 4, declaro a variável chamada "pointer", e nela coloco o endereço de
"var". Quando usado antes do nome de uma variável, o simbolo "&" representa
o "&ndereço" daquela variável (Sacou? "**&**ndereço" Hein?).

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/img006.gif)
{: refdef}

Coloquei todo o código no Visual Studio, executei e inspecionei os valores.
Na aba superior, é possível ver toda a memória do programa. Observei qual o
endereço na variável "pointer" e procurei esse local na memória e... ta-dã!

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/img002.png)
{: refdef}

Mas pera... 2a? Não devia ser 42? Bom... Acontece que 2a é 42 em hexadecimal.
Essa aba de "memória" do Visual Studio mostra os valores em hexadecimal. O
valor 2a aparece no primeiro byte e não no último por causa do [endianess](https://pt.wikipedia.org/wiki/Extremidade_(ordena%C3%A7%C3%A3o)), mas
não vou entrar nesse detalhe agora.

Último conceito importante: "Derreferenciamento" (Derreferencing). O nome é
feio. Mas a ideia é simples. Para recuperar o valor contido no endereço da
variável "pointer", é necessário colocar uma estrela (*) antes do nome da
variável, como no código abaixo. Essa operação tem o nome de derreferenciamento.

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/img003.png)
{: refdef}

Não há muito mais o que dizer. Isso é o básico sobre ponteiros. Mas quero
aproveitar esse post pra entrar em algumas outras questões!


# Memória, new, delete...

Como eu comentei, a memória (RAM) do computador é como se fosse uma grande rua,
cheia de endereços. Em cada endereço, temos um valor. Se isso é verdade, e
um ponteiro pode apontar para qualquer lugar na memória, então posso
acessar memória de outros programas e roubar senhas e dados? **Posso hackear
programas assim?** Não é bem assim...

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/img004.png)
{: refdef}

O sistema operacional (Windows, Linux, MacOS...) não deixa isso
acontecer. O sistema operacional controla quais posições na memória você pode
acessar. Em particular, você tem acesso à duas partes importantes: A "stack" e
a "heap". Estou sendo bem simplista aqui e ignorando MUITO detalhe.

A stack (pilha) é uma porção de memória usada para guardar variáveis temporárias.
Como assim "temporárias"? Bem. No primeiro exemplo, "var" e "pointer" são
duas variáveis temporárias. O "tempo de vida" delas inicia quando o programa
entra na função "main" e termina quando sai da função "main".

A heap é uma porção de memória usada para guardar variáveis dinâmicamente.
Como assim "dinâmicamente"? Aqui entra o "new" e o "delete". Dinâmicamente
significa que você escolhe quando a variável vai ser criada (com new/malloc), e
quando ela vai ser destruida (com delete/free).

Mas assim... Se você está usando C++ como
ferramenta, eu sugiro não se importar muito com new, delete, malloc, free, e
gerenciamento de memória nesse nível. Talvez uma hora você precise, mas não
é comum. Por exemplo, eu trabalho com desenvolvimento de software, e usamos
C++ (e outras linguagens de baixo nível) em alguns aplicativos de simulação
computacional. Faz um bom tempo que não uso um "new" e menos ainda um "malloc",
em C++.


# Smart pointers e estruturas de dados

Estamos em 2020. C++ evoluiu - E já faz tempo. "Novas" versões de C++ possuem
smart pointers (em especial "std::unique\_ptr" e "std::shared\_ptr").

{:refdef: style="text-align: center;"}
![](/images/2020-07-13/img005.png)
{: refdef}
[https://xkcd.com/371/](https://xkcd.com/371/)

Esses "ponteiros espertos" se "auto-gerenciam". Isso significa que eles vão
cuidar dos "new"s e dos "delete"s. Tudo que você precisa fazer é aprender a
sintaxe e usar.

unique_ptr é um "ponteiro único", ou seja, existe apenas uma instância daquele
ponteiro no seu programa. Não existem duas variáveis que possuem aquele mesmo
ponteiro único. Apenas uma variável pode conter um ponteiro único.

shared_ptr é um "ponteiro compartilhado", no sentido de que mais de uma variável
pode compartilhar aquele ponteiro. Duas ou mais variáveis podem apontar para
uma mesma região de memória.

Não se desespere. É pra ser simples. Se você procurar material sobre smart
pointers na internet, tenho certeza que vai encontrar o que precisa e já
conseguir usar.

Por último, gostaria de comentar do caso específico do uso de matrizes e vetores.
No contexto de software de simulação computacional e métodos numéricos, é bastante
comum precisar de vetores, matrizes, matrizes esparsas, e outras estruturas de
dados que são maiores que simples escalares.

Eu sugiro evitar alocar e desalocar esses tipos de estruturas usando ponteiros.
Ao invés disso, verifique as bibliotecas específicas para trabalhar com matrizes
e vetores, como a Eigen, xTensor, Blaze, Armadillo, etc. Isso por que, além
de gerenciar a memória, essas bibliotecas possuem outras
funções que muito provavelmente você vai precisar, por exemplo, produto
interno, operações aritméticas ponto a ponto, etc.


# Pra onde ir a partir daqui?

Pra fazer esse post, eu dei uma procurada no Google sobre posts e textos que
falassem sobre ponteiros. Segue uma lista de links que achei pertinente, para
os estudantes que querem se aprofundar um pouco mais. Espero que eu tenha de
alguma forma ajudado quem ainda tem dúvida sobre esse tema. Se você não entendeu,
continue tentando. Eventualmente vai se tornar simples!

[Essa resposta do Stack Overflow](https://pt.stackoverflow.com/a/266768)

[Esse post da Microsoft sobre ponteiros crus](https://docs.microsoft.com/pt-br/cpp/cpp/raw-pointers?view=vs-2019)

[Esse post da Microsoft sobre smart pointers](https://docs.microsoft.com/pt-br/cpp/cpp/smart-pointers-modern-cpp?view=vs-2019)

[Essa página sobre stack e heap](https://gribblelab.org/CBootCamp/7_Memory_Stack_vs_Heap.html)

[Esse vídeo do Bjarn, criador da linguagem](https://www.youtube.com/watch?v=86xWVb4XIyE)

Até a próxima!
