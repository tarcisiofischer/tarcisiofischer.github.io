---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 2
---

No [último post](https://tarcisiofischer.github.io/2019-10-02/resolvendo-a-equacao-da-difusao-em-python-parte-1),
discretizei a equação de difusão utilizando diferenças finitas, comentei sobre
o sistema linear gerado e comecei a compor as condições iniciais $f_o^{x,y}$.
Nesse post, dando continuidade ao projeto, vou compor a matriz $A$ do sistema
linear, considerando suas propriedades, e visualiza-la.

# Compreensão da matriz $A$

Novamente, essa seção é apenas para contextualizar a parte matemática do
problema. Se o leitor estiver desconfortável, pode pular diretamente para a
parte da programação e, acredito, a leitura será ainda coerente. Porém, sinto-me
obrigado a manter essa seção apenas por completude.

Para relembrar o que era a matriz $A$, mostro novamente as dependencias de um
elemento do nosso domínio discretizado abaixo. O quadradinho amarelo depende
dos quadradinhos em verde, adjacentes, por meio da equação

$$
[\Delta s^2 + 4 c \Delta t]f^{x,y}
- c \Delta t f^{x+1, y}
- c \Delta t f^{x-1, y}
- c \Delta t f^{x, y+1}
- c \Delta t f^{x, y-1}
=
\Delta s^2 f_o^{x, y} + \Delta s^2 \Delta t s^{x, y}
$$

{:refdef: style="text-align: center;"}
![Exemplo - dependências](/images/example002.png)
{: refdef}

Nessa equação, $f^{x,y}$, $f^{x+1,y}$, $f^{x-1,y}$, $f^{x,y+1}$ e $f^{x,y-1}$
são incógnitas. E essa equação se repete para cada elemento da imagem acima,
ou seja,

$$
\begin{bmatrix}
\Delta s^2 + 4 c \Delta t & c \Delta t & 0 & ...\\ 
c \Delta t & \Delta s^2 + 4 c & c \Delta t & ...\\ 
0 & c \Delta t & \Delta s^2 + 4 c & ...\\ 
0 & 0 & c \Delta t & \ddots 
\end{bmatrix}

\begin{bmatrix}
f^{0,0}\\ 
f^{1,0}\\ 
f^{2,0}\\ 
\vdots
\end{bmatrix}
=
\begin{bmatrix}
\Delta s^2 f_o^{0,0} + \Delta s^2 \Delta t \ s^{0, 0}\\ 
\Delta s^2 f_o^{1,0} + \Delta s^2 \Delta t \ s^{1, 0}\\ 
\Delta s^2 f_o^{2,0} + \Delta s^2 \Delta t \ s^{2, 0}\\ 
\vdots
\end{bmatrix}
$$

E assim por diante...

Se isolarmos os termos que multiplicam as incógnitas, podemos representar esse
conjunto de equações por um produto matriz-vetor na forma $Ax = b$, onde $A$
é a matriz de coeficientes que multiplicam as incógnitas, $x$ é um vetor de
incógnitas e $b$ um vetor contendo a parcela conhecida do sistema, como mostrado
na equação abaixo:

$$
\begin{bmatrix}
\Delta s^2 + 4 c \Delta t & c \Delta t & 0 & ...\\ 
c \Delta t & \Delta s^2 + 4 c & c \Delta t & ...\\ 
0 & c \Delta t & \Delta s^2 + 4 c & ...\\ 
0 & 0 & c \Delta t & \ddots 
\end{bmatrix}

\begin{bmatrix}
f^{0,0}\\ 
f^{1,0}\\ 
f^{2,0}\\ 
\vdots
\end{bmatrix}
$$

É importante notar que, para os elementos de borda, não é possível determinar
o valor de $f^{x,y}$. Por exemplo, não temos valor para $f^{-1,0}$ ou de
$f^{0,-1}$. Para evitar esse problema, admitimos que esses elementos existem,
mas possuem valor sempre zero (Ou seja, admitimos que esses valores são
conhecidos e, portanto, não entram na matriz $A$).

# Composição da matriz $A$ em Python

Tendo como referência a explicação anterior, a composição da matriz $A$ pode
ser facilmente implementada em código Python, com alguns cuidados. Em primeiro
lugar, nos concentramos nos coeficientes da diagonal principal:

{% highlight python linenos %}
import numpy as np

A = np.zeros(shape=(N * N, N * N))
for i in range(N):
    for j in range(N):
        A[i * N + j, i * N + j] = ds**2 + 4 * c * dt
{% endhighlight %}

Perceba o calculo dos indices da matriz. Como cada linha da matriz A se refere
à dependência entre o elemento (i, j) = A[i * N + j] e todos os outros elementos
(x, y) do nosso problema, essa conversão é necessária. A imagem abaixo ilustra
essa conversão de indices. Na imagem, a linha 1 da matriz A (à direita) se
refere à dependencia do elemento 1 na matriz do domínio (à esquerda) e todos
os outros elementos (de 0 a 8).

{:refdef: style="text-align: center;"}
![Exemplo - dependência entre elementos](/images/post2-img1.png)
{: refdef}

Seguindo o algorimo, precisamos popular os coeficientes dos elementos adjacentes
na matriz A. Isso é análogo ao que já fizemos para a diagnal principal, porém,
somando ou decrementando os indices para cada caso. Obtemos, então, o seguinte
código:

{% highlight python linenos %}
A = np.zeros(shape=(N * N, N * N))
for i in range(N):
    for j in range(N):
        A[i * N + j, i * N + j] = ds**2 + 4 * c * dt

        A[i * N + j, (i - 1) * N + j] = -c * dt
        A[i * N + j, (i + 1) * N + j] = -c * dt
        A[i * N + j, i * N + (j - 1)] = -c * dt
        A[i * N + j, i * N + (j + 1)] = -c * dt
{% endhighlight %}

Porém, ao tentar executar o código acima, obtemos

{% highlight python linenos %}
IndexError: index 9 is out of bounds for axis 1 with size 9
{% endhighlight %}

O que acontece é que estamos tentando acessar um indice fora do vetor A. Sendo
ainda mais específico, isso acontece por que, ao iterar nos elementos de borda,
não temos informação dos elementos adjacentes, pois eles não existem. Por 
exemplo, para o elemento 2 na imagem acima, não existem elementos acima nem
à direita. Existem, porém, elementos à esquerda (1) e abaixo (5).

Dessa forma, precisamos definir as **condições de contorno** do nosso problema.
Para simplificar, vamos derminar que no contorno, existem elementos "fantasma"
cujos valores são conhecidos, e sempre nulos (ou seja, iguais a zero). Em termos
de código, basicamente não preenchemos informações sobre esses elementos:

{% highlight python linenos %}
A = np.zeros(shape=(N * N, N * N))
for i in range(N):
    for j in range(N):
        A[i * N + j, i * N + j] = ds**2 + 4 * c * dt

        if i - 1 >= 0:
            A[i * N + j, (i - 1) * N + j] = -c * dt
        if i + 1 < N:
            A[i * N + j, (i + 1) * N + j] = -c * dt
        if j - 1 >= 0:
            A[i * N + j, i * N + (j - 1)] = -c * dt
        if j + 1 < N:
            A[i * N + j, i * N + (j + 1)] = -c * dt
{% endhighlight %}

O resultado final é mostrado abaixo, para fins ilustrativos. As cores diferentes
representam os valores diferentes entre a dependencia dos elementos. Esse
resultado específico é para um domínio de tamanho $3 \times 3$.

{:refdef: style="text-align: center;"}
![Exemplo - matriz A para um problema 3x3](/images/post2-img2.png)
{: refdef}

# Complexidade espacial do problema

Eu compreendo que não é usual se preocupar com memória nos dias de hoje, pois,
em geral, temos maquinas com muitos Gb de memória ram, e, para algumas
aplicações nunca chegamos a ter esse tipo de problema. Considere, porém, o
caso em que eu construir a matriz $A$ para o caso onde N = 10000, ou seja,
a matriz tem um total de $N^4$ elementos. Ao tentar executar nosso programa
batemos no seguinte problema

{% highlight python linenos %}
MemoryError: Unable to allocate array with shape
    (100000000, 100000000) and data type float64
{% endhighlight %}

De fato, para construir tal matriz precisariamos de $10000^4 \cdot 8$ bytes,
ou 80 petabytes. Por outro lado, se observarmos a matriz $A$, vemos que a
grande maioria dos elementos são zeros, ou seja, a matriz é esparsa (Ou, nesse
caso específico, ela é uma matriz esparsa banda). Dessa forma, podemos utilizar
uma estrutura de dados para tirar proveito dessa propriedade, nesse caso, as
estruturas disponíveis na [scipy.sparse](https://docs.scipy.org/doc/scipy/reference/sparse.html).

Existem várias formas de fazer isso. Nesse post, vou utilizar a função auxiliar
[diags](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.diags.html),
pois me parece simples o suficiente. 
