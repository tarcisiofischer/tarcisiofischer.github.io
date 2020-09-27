---
layout: post
title: Breve noção sobre análise de algoritmos
image: 2020-09-28/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-09-28/img001.png)
{: refdef}

Desempenho computacional é um assunto recorrente aqui no blog. Entre as motivações para [mover código Python para C++](https://tarcisiofischer.github.io/2020-06-01/movendo-codigo-de-um-solver-em-python-para-cpp), por exemplo, é o ganho de desempenho. Além disso, umas semanas atrás, eu [mostrei a solução de um problema proposto em um desses sites de desafios de programação](https://tarcisiofischer.github.io/2020-09-14/project-euler-67), e comentei que queria dedicar um post só sobre complexidade de algoritmos. Vou evitar detalhes teóricos nesse post, mas quero explicar um pouco sobre esse tipo de análise.

Em primeiro lugar, um jeito comum de se estimar quão rápido é um algoritmo, é medindo o "tempo de parede". Ou seja, olha-se para um relógio que horas eram quando o algoritmo começou a executar e, quando termina, decrementa-se o horário final com o inicial, tendo uma ideia do tempo de execução. Claro que, na prática, o programador não olha de fato para um relógio em uma parede, mas usa uma função que retorna o tempo. Algo assim:

{% highlight python linenos %}
t = time()
run()
print(time() - t)
{% endhighlight %}

Quero deixar claro que essa análise é totalmente válida. De fato, eu até usei ela em um [post no passado](https://tarcisiofischer.github.io/2020-02-24/resolvendo-a-equacao-da-difusao-em-python-parte-3), aqui no blog, e [dei algumas alternativas](https://tarcisiofischer.github.io/2020-03-30/python-profiling) também interessantes. Porém, esse tipo de análise geralmente possui uma dependência muito forte em relação à tecnologia onde o experimento está acontecendo. Em outras palavras, o mesmo teste em dois PCs diferentes pode resultar em tempos diferentes, dependendo da frequência do computador, por exemplo.

Gosto muito do exemplo proposto por [Cormen, no livro de introdução à algoritmos](https://www.amazon.com.br/Introduction-Algorithms-3e-ISE-Cormen/dp/0262533057): Suponha que precisamos ordenar 10 milhões de números. Imagine que são dados dois computadores: computador A e computador B. O computador A executa 10 bilhões de instruções por segundo, e o computador B executa 10 milhões de instruções por segundo (A é 1000 vezes mais rápido que B). O programador A escreve um algoritmo chamado "insertion sort" no computador A, enquanto o programador B utiliza um algoritmo chamado "merge sort", no computador B. Não se preocupe com o entendimento exato do que esses algoritmos fazem. Por enquanto, o importante é saber que o computador B termina de executar em menos de 20 minutos, o computador A termina apenas depois de mais de 5 horas de execução.

O resultado pode parecer surpreendente, mas a questão aqui é que o algoritmo "[merge sort](https://pt.wikipedia.org/wiki/Merge_sort)" implementado pelo programador B executa em tempo proporcional a $n\ \text{lg}\ n$ instruções, enquanto o "[insertion sort](https://pt.wikipedia.org/wiki/Insertion_sort)" escrito pelo programador A executa em $n^2$ instruções, onde $n$ é a quantidade de números para ordenar. Não é trivial fazer essa análise, por isso não estou dando detalhes nesse post em como cheguei nesses números. Para esses algoritmos mais "clássicos", claro que existe tudo documentado, como é o caso do insertion sort e do merge sort.

A ideia geral é determinar quais entradas podem variar de tamanho e, a partir delas, contar quantas instruções são executadas para cada uma delas. A grande questão é que, diferente da primeira análise, não estamos comparando exatamente o tempo de execução, mas sim, o número de instruções que precisamos executar. É claro que se diminuirmos o tempo médio despendido a cada instrução executada, vamos conseguir executar mais coisas em menos tempo. Por outro lado, diminuir o número de instruções também é uma forma de diminuir o tempo despendido.

$$
\text{tempo total} = \frac{\text{segundos}}{\text{instruções}} \times \text{instruções}
$$

De forma grosseira, essa é a ideia em analisar a complexidade dos algoritmos: Poder entender e decidir opções que diminuam o número de instruções necessárias para resolver determinado problema.

Porém, ainda existe um problema: Mesmo que duas pessoas implementem o mesmo algoritmo (Por exemplo, duas pessoas implementem o "merge sort"), dificilmente vão fazer isso usando exatamente o mesmo número de instruções. Ou seja, um programador A pode conseguir escrever um código para "merge sort" que termina em $a_1 \cdot n\ \text{lg}\ n + a_2$ instruções enquanto um programador B pode conseguir $b_1 \cdot n\ \text{lg}\ n + b_2$ instruções.

De qualquer forma, sabemos que a "ordem de grandeza" de ambas implementações será em torno de $n\ \text{lg}\ n$ instruções. Dessa forma, usamos uma notação chamada "notação Big-O" para extrair apenas essa informação de "alto nível" sobre a implementação, ignorando os detalhes de como o algoritmo foi implementado. No exemplo do "merge sort" contra "insertion sort", o primeiro é $O(n^2)$ e o segundo $O(n\ \text{lg}\ n)$.

A análise de complexidade dos algoritmos é uma boa ferramenta para ajudar a identificar quais partes do código vão começar a ficar lentas a medida que o problema aumenta. Mas note que não existe bala de prata: Ainda que você identifique que uma parte do código está lenta, pode não existir algoritmo melhor do que aquele que já se está usando. A ideia de passar código de Python para C++ é, essencialmente, diminuir as constantes multiplicativas presentes na análise de complexidade (Assim como comprar um computador mais rápido). Em outras palavras, passar código de Python para C++ é apenas uma forma de diminuir o número de segundos despendido em cada "instrução" (Pois a execução de uma instrução em Python é muito mais lento do que a execução de uma instrução escrita em C++).

Claro que isso foi só uma brevíssima introdução ao tema. Como comentei no início do post, tentei não entrar em detalhes técnicos nessa breve introdução, e fui até um pouco raso nas explicações, na esperança de dar uma noção geral ao leitor. Deixo o formalismo e aprofundamento no assunto para outros posts.

Até a próxima!
