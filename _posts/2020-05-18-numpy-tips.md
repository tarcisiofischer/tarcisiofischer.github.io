---
layout: post
title: 5 dicas sobre Numpy
---

Numpy é uma biblioteca para Python que possui estruturas de arrays e vetores, 
em conjunto de implementações eficientes de operações entre os mesmos. Ela faz parte
da stack de desenvolvimento científico Scipy. Nesse post, mostrarei 5 dicas básicas
que podem ser úteis no dia-a-dia, baseadas em questões e discussões que tive
recentemente com amigos e colegas.

{:refdef: style="text-align: center;"}
![](/images/2020-05-18/img001.png)
{: refdef}


# 1. Broadcasting

No contexto da Numpy, o broadcasting ocorre quando são feitas operações com
arrays de diferentes tamanhos. Entender o conceito é particularmente útil
para corrigir erros semelhantes ao abaixo.

{% highlight python linenos %}
ValueError: operands could not be broadcast together with shapes (...) (...)
{% endhighlight %}

Antes de tudo, considere o seguinte caso: Aplicamos a função soma sobre dois arrays
unidimensionais, conforme código abaixo. Quando fazemos isso, a operação
resultante é, na verdade, uma soma "ponto a ponto". Note que, nesse caso,
os dois vetores possuem o mesmo shape (3,).

{% highlight python linenos %}
>>> import numpy as np
>>> a = np.array([1,2,3])
>>> b = np.array([4,5,6])
>>> a + b
array([5, 7, 9])
{% endhighlight %}

Por outro lado, quando fazemos a mesma operação com dois arrays com shapes
diferentes, o broadcasting se torna mais interessante. Para simplificar, três
"regras" são aplicadas ao executar uma operação:

1) Compara-se as dimensões. Se não possuem o mesmo tamanho,
aumenta-se o menor com '1's à esquerda até ficarem iguais.

2) Compara-se cada magnitude dos shapes. As dimensões de tamanho '1' são
aumentadas para o valor do outro array.

3) Verifica-se cada valor no shape. Se algum deles não for igual, gera um erro.

Fica mais fácil visualizar com um exemplo:

{% highlight python linenos %}
>>> a = np.array([[ [[1,2]], [[3,4]] ]])
>>> a.shape
(1, 2, 1, 2)
>>> b = np.array([ [[10, 10], [20, 20]] ])
>>> b.shape
(1, 2, 2)
>>> c = a + b
{% endhighlight %}

Seguindo os passos anteriores, para compreender o resultado dessa operação,
primeiro extendemos o de menor tamanho (b), adicionando '1's à esquerda. Note
que isso não altera os valores do array b.

{% highlight python linenos %}
a.shape => (1, 2, 1, 2)
b.shape => (1, 1, 2, 2)
{% endhighlight %}

Aplicando o passo 2, as posições com valor '1' são extendidas com o tamanho do
maior shape. Dessa forma, temos uma homogeneização dos shapes.

{% highlight python linenos %}
a.shape => (1, 2, 2, 2)
b.shape => (1, 2, 2, 2)
{% endhighlight %}

Aplicando a verificação do passo 3, nenhum erro é gerado, e a operação pode ser
feita normalmente:

{% highlight python linenos %}
>>> c
array([[[[11, 12],
         [21, 22]],

        [[13, 14],
         [23, 24]]]])
>>> c.shape
(1, 2, 2, 2)
{% endhighlight %}

Obviamente, essa sequência de passos não altera os arrays 'a' e 'b'.

Conhecer sobre broadcasting pode ser bastante útil para entender e resolver
problemas, e até debugar códigos que estejam com algum comportamento estranho.
Sugiro procurar mais sobre o assunto, pois tem muito material gratuito na
internet, inclusive no manual oficial da Numpy!


# 2. Sintaxe de cópia

Em Python puro, uma forma rápida de fazer uma cópia de uma lista, é utilizando
a sintaxe abaixo. O problema, é que em Numpy essa mesma sintaxe não faz cópia.
Isso pode ser (e já acompanhei casos) uma fonte de bugs muito sutil. O código
abaixo, o qual compara a diferença entre uma
lista comum do Python e um array da Numpy, ilustra bem o problema.

{% highlight python linenos %}
>>> a = [1,2,3,4]
>>> b = a[:]
>>> b[0] = 99
>>> a
[1, 2, 3, 4]
>>> b
[99, 2, 3, 4]
>>> 
>>> 
>>> a = np.array([1,2,3,4])
>>> b = a[:]
>>> b[0] = 99
>>> a
array([99,  2,  3,  4])
>>> b
array([99,  2,  3,  4])
{% endhighlight %}

Para fazer efetivamente uma cópia de um array da Numpy, é possível utilizar
o método 'copy'. Essa não é a única forma de fazer isso, mas é a mais
explícita.

{% highlight python linenos %}
>>> a = np.array([1,2,3])
>>> b = a.copy()
>>> b[0] = 99
>>> a
array([1, 2, 3])
>>> b
array([99,  2,  3])
{% endhighlight %}


# 3. Índices negativos

Como já comentei em um ou outro post, é possível (não só na Numpy mas também
em listas do Python) utilizar valores negativos como índices, a fim de pegar
os últimos valores de um dado array. Isso é bastante útil, porém, pode também
ser fonte de bugs. Por exemplo, dentro de um for-loop onde índices são
computados dinamicamente, é possível pegar um valor do fim do array por engano,
como foi o caso em um dos primeiros posts sobre a solução da equação de difusão,
nesse blog ([dê uma conferida!]({% post_url 2019-02-13-resolvendo-a-equacao-da-difusao-em-python-parte-2 %}))

{% highlight python linenos %}
>>> a = np.array([1,2,3])
>>> a[0]
1
>>> a[-1]
3
>>> a[-3]
1
>>> a[-4]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: index -4 is out of bounds for axis 0 with size 3
>>> a[3]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: index 3 is out of bounds for axis 0 with size 3
{% endhighlight %}


# 4. Transposta de arrays 1D

Transpor matrizes é uma operação utilizada frequentemente, nos programas
que envolvem a aproximação de solução de equações diferenciais. Um detalhe pra
ficar atento é sobre a operação de transposta em arrays unidimensionais da
Numpy. Como mostra o código abaixo, a transposta de um array unidimensional
é igual ao próprio array, como se a operação "não tivesse sido efetuada".
Na verdade, o resultado é consistênte, pois a Numpy não tem como "adivinhar"
qual o shape que o usuário se refere. Uma das formas de contornar o problema,
é trabalhar sempre com arrays bidimensionais. Segue o código:

{% highlight python linenos %}
>>> a = np.array([1,2,3])
>>> a
array([1, 2, 3])
>>> a.T
array([1, 2, 3])
>>> 
>>> a = np.array([[1,2,3]])
>>> a
array([[1, 2, 3]])
>>> a.T
array([[1],
       [2],
       [3]])
{% endhighlight %}


# 5. Numexpr

Por fim, a última dica é pra quem não quer ter o trabalho de mover partes do
código pra C++, FORTRAN ou Rust (ou qualquer outra linguagem com melhor
performance pontualmente), mas ainda quer tentar extrair um pouco mais da
Numpy. A Numexpr resolve um problema que é a quantidade de sub-operações
geradas por grandes expressões na Numpy. Segue um exemplo:

{% highlight python linenos %}
>>> timeit(
>>>   "2 * np.sin(a) + a * np.cos(b) + 3 * b * b * a",
>>>   setup="import numpy as np; import numexpr as ne; a = np.arange(1e+6); b = np.arange(1e+6)",
>>>   number=10
>>> )
0.8443612269993537
>>> timeit(
>>>   "ne.evaluate('2 * sin(a) + a * cos(b) + 3 * b * b * a')",
>>>   setup="import numpy as np; import numexpr as ne; a = np.arange(1e+6); b = np.arange(1e+6)",
>>>   number=10
>>> )
0.463900212000226
{% endhighlight %}

Eu, particularmente, não costumo utilizar pois prefiro seguir outras abordagens...
Inclusive, pra quem tiver
curiosidade, é possível utilizar [Cython](https://cython.org/) como um meio-termo, pra evitar
ainda de mover o código pra outras linguagens.

Espero que essas dicas sejam uteis pra quem estiver iniciando com Python e Numpy.
Provavelmente outras pessoas podem sugerir várias
outras dicas. Neste caso, sugiro deixar nos comentários no LinkedIn,
Facebook, ou Twitter - Que são os principais locais que eu divulgo meus
posts :)

Agradeço ao Bruno Klahr pelas sugestões. Até a próxima!
