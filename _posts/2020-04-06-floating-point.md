---
layout: post
title: De volta ao básico: Aritmética de ponto flutuante
---

Compreender as ferramentas que usamos é muito importante pra tomar decisões. Sempre que eu falo sobre
ponto flutuante (float/double), algumas pessoas reclamam que esse assunto é muito básico, ou que "todo
mundo sabe isso". Mesmo que isso seja verdade (O que não me parece ser), até mesmo quem sabe
exatamente o que está fazendo as vezes cai em alguma "pegadinha". Minha sugestão é que não subestime
o conhecimento sobre esse assunto, por mais básico que ele possa parecer.

Esse post é dividido em 3 partes: Na primeira, mostro alguns resultados que não são normalmente esperados.
Na segunda,  explico o mínimo sobre a representação de ponto flutuante, na esperança de fazer um
"nivelamento" do grupo de leitores. Por fim, na última parte, mostro algumas orientações básicas sobre
a comparação de valores.

{:refdef: style="text-align: center;"}
![](/images/2020-04-06-floating-point/img001.png)
{: refdef}

# Resultados curiosos

Os casos que vou mostrar, estão na linguagem Python apenas por que é a linguagem que tenho usado mais
nesse meu blog. Mas teóricamente é possível reproduzir em qualquer linguagem, desde que a mesma utilize
do mesmo padrão para representação de números em ponto flutuante. Acredito que, até o final do artigo,
o leitor consiga tirar as próprias conclusões do por que esses resultados acontecem. Fico, porém,
disponível caso alguém tenha alguma dúvida adicional. Além disso, existem vários outros casos que conheço,
mas que vou omitir, pelo motivo de que são mais raros de aparecerem no dia-a-dia (pelo menos no meu), e
devem existir mais $N$ outros que eu desconheço. Sinta-se livre para comentar comigo casos que você
conheça, pra eu adicionar no meu "banco de casos"...

## Caso 1: 

{% highlight python linenos %}
>>> print(0.1 + 0.2)
0.30000000000000004
{% endhighlight %}

## Caso 2: Somatório impreciso

{% highlight python linenos %}
>>> r = 0.0
>>> for i in range(100):
...   r += 0.01
...
>>> print(r)
1.0000000000000007
{% endhighlight %}

## Caso 3: 

{% highlight python linenos %}
>>> a = 1e+100
>>> b = -1e+100
>>> c = 0.01
>>> print(a + (b + c))
0.0
>>> (a + b) + c
0.01
{% endhighlight %}

## Caso 4: 

A função round(number, ndigits) retorna o valor arredondado mais próximo.
Se estiver exatamente no meio do caminho, arredonda para cima.
Por exemplo, round(1.4, 0) == 0.0, round(1.5, 0) == 1.0 e round(1.6, 0) == 1.0. Porém,

{% highlight python linenos %}
>>> round(2.665, 2) # Esse resultado é esperado
2.67
>>> round(2.675, 2) # Esse resultado não é esperado... Deveria ser 2.68!?
2.67
>>> round(2.685, 2) # Esse resultado é esperado
2.69
{% endhighlight %}


# Representação de números reais

O conjunto dos números reais é infinito. Infelizmente, a memória do nosso computador é finita. Mais que
isso, a memória destinada às variáveis de um programa é muito inferior ao tamanho total de memória
disponível no mesmo. Portanto, representar números reais no computador implica em perca de informação.

Uma forma de guardar esses valores seria por meio de um ponto "fixo", ou seja, escolhe-se uma posição fixa
para as casas decimais. Isso é basicamente equivalente à guardar os valores como um inteiro e ter um fator
de escala fixo. Por exemplo, a quantidade $1.237$ é a equivalente à quantidade $12370 * S$, com $S = \frac{1}{10000}$.
A desvantagem, porém, é que existe um limite de valores representáveis.

A alternativa é a aritmética de ponto flutuante, que acabamos escolhendo não por escolha racional, mas
por ser a que a maioria das linguagens deixa como disponível de forma padronizada e, por consequência,
acabamos "aprendendo", aceitando e utilizando. Quando nos damos conta que não sabemos explicar o motivo
pelos quais as contas não estão batendo, primeiro culpamos o compilador, depois os erros de truncamento
do método numérico, ou a imprecisão nas medições, ou o hardware, etc. Não que esses não possam ser fontes
de erro. Minha argumentação é apenas que raramente duvidamos da própria forma como guardamos os dados.

A representação segue um padrão chamado IEEE 754. De forma geral, ela segue o padrão semelhante ao que
usamos em notação científica, ou seja

$$
m \cdot B^e
$$

onde $m$ é a mantissa, $B$ é a base (aqui é usado a base 2, para o sistema binário) e $e$ o expoente. Em
memória, os tipos float e double diferem na quantidade de bits para cada parcela, e usam-se offsets para
evitar representações repetidas (por exemplo, $0.01 = 0.01 \cdot 10^1 = 1.0 \cdot 10^{-1}$. Dessa forma,
a representação final fica conforme representação abaixo:

{:refdef: style="text-align: center;"}
![Representação ponto flutuante](/images/2020-04-06-floating-point/img002.png)
{: refdef}

Deixo abaixo dois exemplos para a representação do valor $0.5$ em formatos float e double. Não se preocupe
se tudo ainda está muito abstrato. A ideia geral é entender que existe um formato padronizado e rigoroso
de representação dos valores, que é seguido. Algumas implicações dessa escolha são os resultados obtidos
na primeira parte desse post: Nem todos os números são representáveis (por exemplo, 0.3 não é
representável, o que explica o primeiro caso), e as operações entre valores de ponto flutuante podem carregar
esses pequenos erros de representação.

Além disso, quanto maior o expoente, maior será o "passo" entre dois valores representáveis. A imagem
abaixo ilustra essa característica de representação. A distância entre dois números representados utilizando
ponto flutuante vai ser maior, a medida que o próprio valor aumenta. Por exemplo, o próximo valor real
representável depois do valor 0.00003051759267691522836685180664062500 é 0.00003051759631489403545856475830078125,
enquanto o próximo valor representável depois do valor 32768.01562500 é 32768.01953125.

{:refdef: style="text-align: center;"}
![Distância entre valores](/images/2020-04-06-floating-point/img003.png)
{: refdef}

Dessa forma, se tentarmos somar dois números cujas magnitudes estão muito distantes, como no exemplo do
caso 3, a representação da resposta é exatamente igual à representação da quantidade de maior magnitude.
Ou seja, $b + c = b$, naquela ocasião, quando forçamos a ordem das operações utilizando os parenteses.

Não vou entrar em outros detalhes tais como a representação do infinito, do NAN (Not A Number), dos valores
denormais, do formalismo entre as operações de ponto flutuante, etc., pois não quero transformar esse post
em um manual. O objetivo dessa breve seção é, como comentei, apenas apresentar um pouco mais
formalmente para amigos e colegas, principalmente estudantes, que não tiveram a sorte de serem devidamente
introduzidos sobre o tema. Espero que esse resumo tenha valor para essas pessoas.


# Comparação de valores

Por fim, um último tópico que gostaria de dissertar sobre é a questão de comparação. Como vimos, dependendo
da sequência de operações, os valores esperados podem não ser representáveis, quando utilizamos aritmética
de ponto flutuante. Por conta disso, jamais devemos comparar dois números absolutamente, conforme exemplo
abaixo:

{% highlight python linenos %}
a = f(x)
b = g(y)
if a == b:
    h(a, b)
{% endhighlight %}

O uso da palavra "jamais" é colocado com cuidado, pois de fato existem casos que queremos fazer
isso, porém, não é o caso geral. Por exemplo, não é raro vermos a verificação por 0.0 para evitar
divisão por zero, por exemplo:

{% highlight python linenos %}
if a == 0.0:
  a = 1e-32
c = b / a
{% endhighlight %}

Mesmo em casos como esse, deve-se tomar cuidado, pois números muito pequenos por exemplo $10^-50$ podem
gerar resultados absurdamente altos para $c$, o que pode ser ruim, mesmo que a divisão por zero tenha
sido evitada...

Uma forma mais segura (e bastante comum) para comparar valores de ponto flutuante é verificar se dois
valores são aproximadamente iguais, dada uma tolerância, ou seja

{% highlight python linenos %}
a = f(x)
b = g(y)
if abs(a - b) < tolerance:
    h(a, b)
{% endhighlight %}

Porém, até essa forma pode ser ruim quando temos valores muito próximos de 0, pois podemos cair exatamente
no caso em que todas as diferenças entre a e b são menores que a tolerância. Para contornar esse problema,
adicionamos um erro relativo, dada uma referência. Por exemplo, a implementação da numpy isclose usa
como referência o valor de b, ou seja:

{% highlight python linenos %}
abs(a - b) <= (atol + rtol * abs(b))
{% endhighlight %}

Por outro lado, a função isclose builtin do Python, usa como referência o maior valor absoluto entre a
e b:

{% highlight python linenos %}
abs(a-b) <= max(rel_tol * max(abs(a), abs(b)), abs_tol)
{% endhighlight %}

Claro que ninguém vai escrever esse código toda vez que quiser comparar dois valores. A ideia é utilizar
alguma dessas funções prontas, seja da numpy, builtin do Python ou mesmo de uma implementação própria.
mas o mais importante é que se evite comparações de absoluta equivalencia, ou seja, a == b, pois geralmente
os valores não serão exatamente iguais, e a condicional vai tratar um caso possívelmente tão específico e
raro, que talvez nem precisasse de tratamento...

Para maiores informações sobre aritmética de ponto flutuante, sugiro as seguintes leituras:
[esse artigo](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html) (Que é denso, porém, muito bom), [esse blog](https://randomascii.wordpress.com/),
[esse site](https://floating-point-gui.de/), [esse vídeo](https://www.youtube.com/watch?v=PZRI1IfStY0) e, por que não, [minha apresentação](https://github.com/tarcisiofischer/examples-and-demos/blob/master/Floating%20Point%20-%20Full%20presentation.pdf) ;)

Até a próxima!
