---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 1
---

Muita gente com conhecimento em outras linguagens de programação me pedem ajuda pra iniciar com
Python. Em geral são pessoas com experiência em outras linguagens de programação como C, C++,
Matlab ou Fortran (Sim… Fortran…). Estou planejando alguns materiais nesse sentido pra ajudar o
pessoal, e essa série de posts vai ser um desses.

A ideia aqui é resolver uma equação simples, apenas como incentivo pra mostrar algumas
funcionalidades de [Python](https://www.python.org/) e a
["stack científica"](https://www.scipy.org/about.html). Então, nessa série de posts vou abordar um
pouco sobre vários temas que eu ache relevante e/ou que me perguntam ou já me perguntaram. Por isso,
caso você, leitor, tenha alguma dúvida, fique livre pra comentar ou me procurar pelas redes sociais
que, dependendo do tema, eu posso gerar conteúdo direcionado. Sinta-se livre também pra me apontar
erros que, afinal, somos todos humanos.

# O problema

A equação que vamos resolver é

$$
\dot{f} = c \nabla^2 f + s
$$

E o resultado final pro nosso programa (No final da série de posts) será algo parecido com isso,
por exemplo:

{:refdef: style="text-align: center;"}
![Exemplo - Resultado final esperado](/images/example001.gif)
{: refdef}

Se você não tem familiaridade com esse tipo de equação, não se desespere – Alias, pode até pular
toda a parte matemática, que está aqui apenas para formalizar o problema antes de sair programando.
De qualquer forma, daqui a pouco vamos nos desvincular da parte matemática e entrar na
programação...

Se, por outro lado, você tem familiaridade, por favor, faça vista grossa das simplificações que eu
vou fazer aqui – por exemplo, na discretização do domínio e na aplicação das condições de contorno
:)

O problema é, então, que queremos encontrar uma função do tempo e do espaço, ou seja, $f(X, t)$ que
satisfaça a equação diferencial acima, dado um termo fonte $s(X, t)$.  Utilizaremos um domínio
bidimensional, admitindo um espaçamento igual em $x$ e em $y$ (que chamarei de $\Delta s$). Dessa
forma, temos que $X = X(x, y)$, e utilizaremos a discretização em diferenças finitas, de onde
obtemos:

$$
\frac{f^{x,y} - f_o^{x,y}}{\Delta t} = c \Bigg{[}
    \frac{f^{x+1,y} - 2 f^{x,y} + f^{x-1,y}}{\Delta s^2} +
    \frac{f^{x,y+1} - 2 f^{x,y} + f^{x,y+1}}{\Delta s^2}
\Bigg{]} + s^{x,y}
$$

Isolando os termos, ficamos com um sistema de equações lineares - Uma equação para cada $f^{x,y}$:

$$
[\Delta s^2 + 4 c \Delta t]f^{x,y}
- c \Delta t f^{x+1, y}
- c \Delta t f^{x-1, y}
- c \Delta t f^{x, y+1}
- c \Delta t f^{x, y-1}
=
\Delta s^2 f_o^{x, y} + \Delta s^2 \Delta t s^{x, y}
$$

Phew… Estamos quase lá. Basicamente, para descobrir o valor de cada $f^{x,y}$, dependemos dos
valores adjacentes, como no exemplo da imagem abaixo. O quadradinho em amarelo depende dos valores
quadradinhos em verde (Ou azul?):

{:refdef: style="text-align: center;"}
![Exemplo - dependências](/images/example002.png)
{: refdef}

Mas os quadradinhos em verde também possuem valores desconhecidos. Isso acaba gerando uma "cadeia"
de equações (Uma pra cada quadradinho). Ou seja, um sistema linear ($Ax = b$) com $N \times N$
equações, onde $N$ é a dimensão do domínio inteiro, $A$ é a matriz de coeficientes, $x$ são as
incógnitas ($f^{x,y}$ para cada $x$ e $y$ no domínio) e $b$ é um vetor conhecido, que depende das
informações relacionadas ao tempo anterior e ao termo fonte.

# Abordagem de solução

Usaremos, a princípio, a linguagem Python para resolver o problema, muito embora eu esteja com
vontade de possivelmente mostrar integração entre Python e C++ no intuito de ganhar performance.
Comentem aqui embaixo se tiver interesse em fazer isso.

A principio, essa série terá 4 posts:

1. Introdução e composição da condição inicial   (Este post)
2. Composição da matriz $A$
3. Solução do sistema linear
4. Visualização de resultados

Em cada um desses posts, me aprofundarei um pouco em algum conceito específico sobre Python,
bibliotecas ou programação em geral.

# Composição da condição inicial

Pois bem, podemos finalmente começar a programar. Começo aqui com a condição inicial $f_o^{x, y}$.
Criaremos, por enquanto, um quadrado no plano bidimensional:

{% highlight python linenos %}
import numpy as np

N = 20
fo = np.zeros(shape=(N, N,))
for i in range(N):
    for j in range(N):
        if i >= 5 and i < 15 and j >= 5 and j < 15:
            fo[i, j] = 10.0
{% endhighlight %}

Esse é o nosso primeiro uso da biblioteca numpy. Por hora, estamos utilizando apenas a estrutura de
dados [ndarray](https://numpy.org/devdocs/reference/arrays.ndarray.html) (N-dimension array). A
função auxiliar [np.zeros](https://docs.scipy.org/doc/numpy/reference/generated/numpy.zeros.html)
constrói um array de dimensão dada pelo parâmetro shape, nesse caso, $(N, N)$, ou seja, uma matriz
quadrada cheia de zeros. Por padrão, o tipo de dado do array de zeros é float.

Para quem não é acostumado, o for-loop do python itera sempre em uma sequência. Nesse caso, a
sequência gerada pelo [range](https://docs.python.org/3/library/functions.html#func-range) constrói
todos os valores de 0 até N (Não incluindo o próprio N). Iteramos, assim, em todo o domínio,
checando com o if se estamos dentro de um quadrado nos limites entre as coordenadas $(5, 5)$ e
$(15, 15)$.

Na verdade, o fato é que existe uma forma muito mais compacta (e eficiente) de preencher a matriz
da numpy, utilizando [slicing](https://docs.scipy.org/doc/numpy/reference/arrays.indexing.html):

{% highlight python linenos %}
fo = np.zeros(shape=(N, N,))
fo[5:15, 5:15] = 10.0
{% endhighlight %}

O pessoal que programa em Fortran talvez fique bastante familiarizado com a notação. Cuidado apenas
que os arrays da numpy (Assim como o resto do mundo moderno) parte os indices do 0 e não do 1. Essa
notação (utilizando slicing) não só é mais compacta, como é mais eficiente, e você pode testar isso
utilizando a função [timeit](https://docs.python.org/2/library/timeit.html) do Python (Tomei a
liberdade de adicionar uma segunda versão em Python puro utilizando os ranges sem ifs, antes que
alguém reclame que a comparação inicial era injusta):

{% highlight python linenos %}
import timeit

code1 = """
N = 20
fo = np.zeros(shape=(N, N,))
for i in range(N):
    for j in range(N):
        if i >= 5 and i < 15 and j >= 5 and j < 15:
            fo[i, j] = 10.0
"""

code2 = """
N = 20
fo = np.zeros(shape=(N, N,))
for i in range(5, 15):
    for j in range(5, 15):
        fo[i, j] = 10.0
"""

code3 = """
N = 20
fo = np.zeros(shape=(N, N,))
fo[5:15, 5:15] = 10.0
"""

print(timeit.timeit(
        code1,
        setup='import numpy as np',
        number=100000
    )
)
print(
    timeit.timeit(
        code2,
        setup='import numpy as np',
        number=100000
    )
)
print(
    timeit.timeit(
        code3,
        setup='import numpy as np',
        number=100000
    )
)
{% endhighlight %}

Os resultados são aproximadamente 7.7s, 2.7s e 0.4s (respectivamente, no meu PC). E um comentário
adicional é que da pra fazer MUITA coisa com a numpy e sua notação. Porém, na minha experiência, é
fácil você começar a escrever código muito performático e **ilegível**, portanto, cuidado com o uso
da biblioteca…

Eu poderia escrever mais uns 10 parágrafos sobre esse pequeno trecho de código, mas vamos seguir
pra evitar de transformar essa série de posts em um livro… Se tem algo aqui que você gostaria que
eu comentasse mais ou falasse melhor, deixa um comentário aqui embaixo!

Finalmente, para visualizar nossa matriz, existem algumas alternativas. A mais óbvia, mas que eu
não posso deixar de comentar, é com a função print, do Python, que vai trazer uma representação do
array em texto. Porém, para casos com muitos elementos (Por exemplo esse, que é uma matriz
20 x 20), eu vou sugerir uma alternativa utilizando a biblioteca [matplotlib](https://matplotlib.org/).

A biblioteca possui uma extensa variedade de funções auxiliares para plotar gráficos e imagens.
Dessa forma, o código abaixo gera uma visualização que, ao meu ver, é bem mais interessante que um
simples print:

{% highlight python linenos %}
from matplotlib import pyplot as plt
plt.imshow(fo)
plt.show()
{% endhighlight %}

{:refdef: style="text-align: center;"}
![Exemplo - plot com matplotlib](/images/example003.png)
{: refdef}

E nós vamos utilizar essa mesma abordagem pra visualizar nossa matriz $A$ no próximo post! Mas por
hora vou ficando por aqui. Por favor, se você gostou desse post e quer ver o resto da série me
deixa um feedback pra eu saber se vale a pena continuar fazendo. Se tem algum tema em específico
que você queira ler sobre, pode comentar também (Exemplos: paralelização, integração com C++,
interfaces gráficas, testes automatizados, algum algoritmo numérico específico, etc).

Até a próxima!

