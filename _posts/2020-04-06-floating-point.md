---
layout: post
title: De volta ao básico - Aritmética de ponto flutuante
---

Sempre que eu falo sobre
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

Os casos que vou mostrar, estão escritos na linguagem Python apenas por que é a linguagem que tenho usado mais
nesse meu blog. Mas é possível reproduzir em qualquer linguagem, desde que a mesma utilize
do mesmo padrão para representação de números em ponto flutuante. Existem vários outros casos que conheço,
mas que vou omitir, pelo motivo de que são mais raros de aparecerem no dia-a-dia (pelo menos no meu), e
devem existir mais $N$ outros que eu desconheço. Sinta-se livre para comentar comigo casos que você
conheça, pra eu adicionar no meu "banco de casos"...

## Caso 1

{% highlight python linenos %}
>>> print(0.1 + 0.2)
0.30000000000000004
{% endhighlight %}

## Caso 2

{% highlight python linenos %}
>>> print(0.9 + 0.1)
1.0
>>> r = 0.0
>>> for i in range(100):
...   r += 0.01
...
>>> print(r)
1.0000000000000007
{% endhighlight %}

## Caso 3

{% highlight python linenos %}
>>> a = 1e+100
>>> b = -1e+100
>>> c = 0.01
>>> print(a + (b + c))
0.0
>>> print((a + b) + c)
0.01
{% endhighlight %}

## Caso 4

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
disponível no mesmo. Portanto, representar números reais no computador implica em perda de informação.

Uma forma de guardar esses valores é por meio de um ponto "fixo", ou seja, escolhe-se uma posição fixa
para as casas decimais. Isso é equivalente a guardar os valores como se fossem números inteiros, e ter um fator
de escala fixo. Por exemplo, a quantidade $1.237$ é a equivalente à quantidade $12370 \cdot S$, com $S = \frac{1}{10000}$.
A desvantagem, porém, é que a faixa de valores fica muito restrita, ou seja, dificilmente você vai conseguir 
representar valores muito pequenos e muito grandes ao mesmo tempo.

A alternativa é a aritmética de ponto flutuante, que acabamos escolhendo (nós, desenvolvedores de software)
não por escolha racional, mas
por ser a que a maioria das linguagens deixa disponível por padrão e, por consequência,
acabamos aceitando e utilizando. Quando nos damos conta que não sabemos explicar o motivo
pelos quais as contas não estão batendo, primeiro culpamos o compilador, depois os erros de truncamento
do método numérico, ou a imprecisão nas medições, ou o hardware, etc. Não que esses não possam ser fontes
de erro, mas raramente duvidamos da própria representação.

A representação segue um padrão chamado IEEE 754. De forma geral, ela se assemelha ao que
usamos em notação científica, ou seja, $m \cdot B^e$, onde $m$ é a mantissa, $B$ é a base (aqui é usado a
base 2, para o sistema binário) e $e$ o expoente. Em
memória, os tipos float e double diferem na quantidade de bits (32 e 64, respectivamente).
Usam-se offsets para
evitar representações repetidas (por exemplo, evitar que seja possível representar
$0.01$ como $0.01 \cdot 10^1$ e como $1.0 \cdot 10^{-1}$). Dessa forma,
a representação final fica conforme apresentado abaixo:

{:refdef: style="text-align: center;"}
![Representação ponto flutuante](/images/2020-04-06-floating-point/img002.png)
{: refdef}

Deixo abaixo dois exemplos para o valor $0.5$, em binário, em formatos float e double. Não se preocupe
se tudo ainda está muito abstrato. A ideia geral é entender que existe um formato padronizado 
de representação dos valores, e que existem algumas implicações dessa escolha, exemplificadas por meio dos
resultados obtidos na primeira parte desse post. Por exemplo, agora sabemos que nem todos os números são
representáveis de forma exata (por exemplo, $0.3$ não é, o que explica o primeiro caso), e as operações
entre valores de ponto flutuante podem carregar esses pequenos erros de representação.

{:refdef: style="text-align: center;"}
![Representação ponto flutuante](/images/2020-04-06-floating-point/img004.png)
{: refdef}

{:refdef: style="text-align: center;"}
![Representação ponto flutuante](/images/2020-04-06-floating-point/img005.png)
{: refdef}

Além disso, quanto maior o expoente, maior será o "passo" entre dois valores representáveis. A imagem
abaixo ilustra essa característica. A distância entre dois números representados utilizando
ponto flutuante vai ser maior, a medida que o próprio valor aumenta. Por exemplo, o próximo valor real
representável depois do valor $0.00003051759267691522836685180664062500$ é $0.00003051759631489403545856475830078125$,
enquanto o próximo valor representável depois do valor $32768.01562500$ é $32768.01953125$.

{:refdef: style="text-align: center;"}
![Distância entre valores](/images/2020-04-06-floating-point/img003.png)
{: refdef}

Dessa forma, se tentarmos somar dois números cujas magnitudes estão muito distantes, como no exemplo do
caso 3, a representação da resposta é exatamente igual à representação da quantidade de maior magnitude.
Ou seja, $b + c = b$, naquela ocasião, quando forçamos a ordem das operações utilizando os parênteses.

Não entrarei na explicação sobre outros detalhes tais como a representação do infinito, do NAN (Not A Number), dos valores
denormais, do formalismo entre as operações envolvendo ponto flutuante, etc., pois não quero transformar esse post
em um manual (Vide as sugestões de leitura no final desse post).
O objetivo dessa breve seção é, como comentei, apenas apresentar um pouco mais
formalmente para amigos e colegas, principalmente estudantes, que não tiveram a sorte de serem devidamente
introduzidos sobre o tema.


# Comparação de valores

Por conta dos possíveis problemas decorrentes da execução de operações envolvendo ponto flutuante, por
exemplo a falta de associatividade, jamais devemos comparar dois números absolutamente, conforme exemplo
abaixo:

{% highlight python linenos %}
a = f(x)
b = g(y)
if a == b:
    h(a, b)
{% endhighlight %}

O uso da palavra "jamais" é colocado com cuidado, pois de fato existem casos que queremos fazer
isso. Por exemplo, não é raro vermos a verificação por 0.0 para evitar
divisão por zero, por exemplo:

{% highlight python linenos %}
if a == 0.0:
  a = 1e-50
c = b / a
{% endhighlight %}

Mesmo em casos como esse, deve-se tomar cuidado, pois números muito pequenos por exemplo $10^{-50}$ podem
gerar resultados absurdamente altos para $c$, o que pode ser ruim, mesmo que a divisão por zero tenha
sido evitada...

Uma forma mais segura (e bastante comum) para comparar valores de ponto flutuante é verificar a diferença
absoluta entre dois valores é próxima de zero, dada uma tolerância:

{% highlight python linenos %}
a = f(x)
b = g(y)
if abs(a - b) < tolerance:
    h(a, b)
{% endhighlight %}

Porém, até essa forma pode ser ruim quando temos valores a e b muito próximos de 0. Para contornar esse problema,
adicionamos um erro relativo, dada uma referência. Por exemplo, a implementação da numpy isclose usa
como referência o valor de b:

{% highlight python linenos %}
abs(a - b) <= (atol + rtol * abs(b))
{% endhighlight %}

Por outro lado, a função isclose builtin do Python, usa como referência o maior valor absoluto entre a
e b:

{% highlight python linenos %}
abs(a-b) <= max(rel_tol * max(abs(a), abs(b)), abs_tol)
{% endhighlight %}

Claro que ninguém vai escrever esse código toda vez que quiser comparar dois valores. A ideia é utilizar
alguma função pronta - Seja da numpy, builtin do Python ou mesmo de uma implementação própria.
O mais importante é que se evite comparações de absoluta equivalência, ou seja, a == b, pois geralmente
os valores não serão exatamente iguais, e a condicional vai tratar um caso possívelmente tão específico e
raro, que talvez nem precisasse de tratamento...

Para maiores informações sobre aritmética de ponto flutuante, sugiro as seguintes referências:
[esse vídeo](https://www.youtube.com/watch?v=PZRI1IfStY0),
[esse artigo](https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html),
[esse blog](https://randomascii.wordpress.com/) e
[esse site](https://floating-point-gui.de/).

Até a próxima!
