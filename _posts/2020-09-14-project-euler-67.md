---
layout: post
title: Project Euler 67 - Maximum path sum II
image: 2020-09-14/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-09-14/img001.png)
{: refdef}

Eu já devo ter comentado algumas vezes aqui no blog como eu gosto de fazer esses desafios de lógica, algoritmos e programação em geral. [Project Euler](https://projecteuler.net/) é um site que contém vários desses desafios, geralmente envolvendo algum problema matemático que você pode resolver com um mix de programação.

O [problema 67](https://projecteuler.net/index.php?section=problems&id=67), pede para você encontrar um caminho de números adjacentes, de cima pra baixo, em um triangulo de números que maximize a soma de todos os valores nesse caminho. Você só pode escolher um número por linha, e esse número deve ser adjacente ao que você escolheu anteriormente.

Por exemplo, no primeiro triângulo abaixo, o caminho com maior soma é o identificado em vermelho, ou seja, 3 + 7 + 4 + 9 = 23. Se você escolher qualquer outro caminho, a soma deve dar menor, como no exemplo do segundo triangulo, em azul: 3 + 4 + 6 + 3 = 16. Outro contra-exemplo é o caminho do terceiro triângulo, em verde: 3 + 7 + 6 + 9. Apesar de somar 25, esse caminho não é válido pois 7 e 6 não são adjacentes. Partindo do 7 você poderia escolher apenas o 2 ou o 4.

{:refdef: style="text-align: center;"}
![](/images/2020-09-14/example001.png)
{: refdef}

Porém, para complicar um pouco mais o problema, o triângulo que resolveremos não é o mostrado acima, mas sim um com mil linhas, provido pelo próprio Project Euler. A imagem reduzida do problema completo é mostrada abaixo, apenas para ter noção do tamanho do problema. Note que, como o próprio site sugere, tentar todas as rotas possíveis não é boa ideia, pois existem $2^99$ rotas e, mesmo que você conseguisse testar um trilhão de rotas por segundo, você demoraria vinte bilhões de anos para testar todas as possibilidades :)

{:refdef: style="text-align: center;"}
![](/images/2020-09-14/big_triangle.png)
{: refdef}

A primeira ideia que surge é a de pegar sempre o maior número possível, maximizando sempre as somas parciais no caminho até a base do triângulo. Nessa ideia, o primeiro valor do triângulo é (trivialmente) o único valor do topo e, a partir dele, escolhe-se o adjacente de maior valor. Esse tipo de algoritmo é chamado "algoritmo guloso" (greedy algorithm). A ideia geral é que sempre se escolhe um "caminho ótimo local", ignorando informações globais. Nesse caso, porém, esse algoritmo não funciona.

Para ilustrar o problema, considere os dois caminhos escolhidos no triângulo das imagens abaixo. O caminho em vermelho segue o algoritmo, criando a soma 9 + 9 + 1 + 1 + 1 = 21. O problema é que, por fazer escolhas aparentemente boas no início, o algoritmo logo perde a oportunidade de fazer um bom caminho enquanto, no caminho em azul da imagem, o início é aparentemente ruim, porém, a soma total é a melhor possível: 9 + 1 + 9 + 9 + 9 = 37.

{:refdef: style="text-align: center;"}
![](/images/2020-09-14/greedy.png)
{: refdef}

Uma possível solução para esse algoritmo tira proveito de uma técnica chamada Programação Dinâmica. Basicamente, memoriza-se soluções ótimas parciais mas, ao invés de simplesmente toma-las como solução, como no caso dos algoritmos gulosos, elas são usadas para decidir as melhores soluções posteriores.

No caso do problema dos triângulos, a ideia é, começando do topo, iterar linha por linha (de cima pra baixo), e ir salvando uma estrutura de dados que contém a maior soma possível até cada coluna da linha atual. Quando chegar na última linha, é necessário apenas iterar em todos as colunas da última linha, buscando o maior valor.

Para ficar mais claro, considere novamente o triângulo do primeiro exemplo:

{% highlight python linenos %}
   3
  7 4
 2 4 6
8 5 9 3
{% endhighlight %}

Começando da primeira linha, nossa estrutura de dados contém, trivialmente, o valor 3 apenas. A partir da segunda linha, para cada coluna, a maior soma possível até aquele momento é salva:

{% highlight python linenos %}
  3
(3+7=)10 (3+4=)7
{% endhighlight %}

A partir da terceira coluna as coisas ficam mais interessantes:

{% highlight python linenos %}
    3
  10  7
(10+2=)12  (10+4=)14  (7+6=)13
{% endhighlight %}

Observe o valor do meio. Haviam dois caminhos possíveis para chegar à ele: O caminho que somava 3 + 7 e o caminho que somava 3 + 4. Ambos já estão computados na segunda linha: 10 e 7. Como sabemos que essas são as possibilidades máximas até a segunda linha, escolhemos aquela de maior valor para somar com o valor 4, totalizando 14.

Note que essa abordagem é bem diferente do algoritmo guloso, pois ao invés de simplesmente ir somando os maiores valores que encontra pela frente, esse algoritmo considera as somas parciais que ele tem em memória até o momento. O algoritmo guloso possuia apenas a informação do maior valor que ele visualizou no caminho que ele tomou até o momento, mas não possuia os valores dos outros possíveis caminhos, e isso faz toda a diferença!

Além disso, diferente da abordagem que testa todas as possibilidades, esse algoritmo não refaz os caminhos do zero para descobrir uma nova soma. Ele aproveita os caminhos que já tomou para aproveitar sub-soluções e guardar apenas as de maior valor, descartando apenas aquelas que com certeza não vão gerar a maior soma possível.

Claro que a explicação do algoritmo ainda está incompleta. Seguindo a ideia apresentada, a estrutura de dados final gerada por essa abordagem é o seguinte:

{% highlight python linenos %}
       3
    10   7
  12  14  13
20  19  23  16
{% endhighlight %}

A última linha é a linha mais relevante. Para cada coluna, ela possui a maior soma possível de caminhos que termina naquele local. Ou seja, partindo do topo, a maior soma possível que termina na segunda coluna da esquerda pra direita é 19. O caminho é 3 + 7 + 4 + 5 = 19. Porém, estamos interessados no maior valor possível, independente de qual coluna termine. Dessa forma, iteramos em cada valor da última linha (ou seja, ignoramos todos os outros valores do triângulo de somas parciais), e encontramos 23.

Eu não vou entrar no mérito sobre o estudo da complexidade de algoritmo, pois gostaria de dedicar um post só sobre esse assunto. Porém, é acho relevante comentar que esse algoritmo é muito mais rápido do que aquele que passava por todas as possibilidades, ou seja, esse algoritmo termina em um tempo muito menor do que vinte bilhões de anos :)

Até a próxima!
