---
layout: post
title: Solução de sistemas lineares com componentes imaginárias
---

Estou pra começar uma nova série de posts, que vai envolver a solução da equação
de Helmholtz, com um termo adicional de "amortecimento", conforme equação abaixo.
Esse termo acaba adicionando uma componente complexa (número imaginário) no
sistema linear discretizado. Por isso, antes de começar a nova série de posts,
vou deixar essa parte resolvida, pra quando precisar, não ter que parar pra
explicar :)

$$
\frac{\omega^{2}}{c^{2}(\mathbf{x})} p(\mathbf{x}, \omega)
- i \omega \eta(\mathbf{x}) p(\mathbf{x}, \omega)
+ \nabla^{2} p(\mathbf{x}, \omega)
+ s(\mathbf{x}, \omega) = 0
$$

Um dos meus temas preferidos pra esses posts desse blog são aqueles relacionados
à solução de sistemas lineares. Esse vai ser mais um nesse sentido.
Nem todos os solvers são preparados para lidar com números complexos. Nesse
post, mostro como contornar esse problema de maneira
de simples implementação. A técnica pode ser replicada para qualquer sistema
linear com componentes complexas, por isso, vale a leitura!

{:refdef: style="text-align: center;"}
![](/images/2020-04-20/img001.png)
{: refdef}


# Derivação matemática

A forma como será resolvido o problema não é nova. Alias, apesar de nunca ter
visto em aulas de pós graduação, não duvído que seja vista em alguma disciplina
mais específica. A premissa aqui é que possuimos um solver de sistemas lineares
($Ax = b$) capaz de resolver problemas onde todos os componentes são reais,
tanto na matriz de coeficientes $A$ quanto no vetor $b$, e, por consequência,
no vetor solução $x$.

Dado, então, um sistema $Ax = b$, cuja matriz $A$ e o vetor $b$ possuem
componentes imaginárias, então eles podem ser escritos separando cada uma
das partes, obtendo-se $A = A^r + i A^i$ e $b = b^r + i b^i$. Dessa forma,
reescrevemos o sistema linear como

$$
(A_{n}^{r} + iA_{n}^{c}) (x_{n}^{r} + ix_{n}^{c}) = b_{n}^{r} + ib_{n}^{c}%
$$

$$
A_{n}^{r} x_{n}^{r} + A_{n}^{r} ix_{n}^{c} + iA_{n}^{c} x_{n}^{r} + iA_{n}^{c}
ix_{n}^{c} = b_{n}^{r} + ib_{n}^{c}%
$$

$$
(A_{n}^{r} x_{n}^{r} - A_{n}^{c} x_{n}^{c}) + i (A_{n}^{r} x_{n}^{c} +
A_{n}^{c} x_{n}^{r}) = b_{n}^{r} + ib_{n}^{c}%
$$

Dado que cada uma das partes não é capaz de gerar um novo número imaginário,
então separamos em duas equações distintas, ou seja,

$$
\begin{matrix}
A_{n}^{r} x_{n}^{r} - A_{n}^{c} x_{n}^{c} = b_{n}^{r}\\
A_{n}^{c} x_{n}^{r} + A_{n}^{r} x_{n}^{c} = b_{n}^{c}\\
\end{matrix}
$$

Ou, em notação matricial,

$$
\begin{bmatrix}
A^{r} & -A^{c}\\
A^{c} & A^{r}%
\end{bmatrix}
x =
\begin{bmatrix}
b^{r}\\
b^{c}%
\end{bmatrix}
$$

Tal operação, ao mesmo tempo que provê
a flexibilidade de resolver problemas com números imaginários, quadriplica o
tamanho do problema original. Uma curiosidade
adicional é que podemos permutar as linhas e colunas para produzir sistemas
equivalentes.


# Implementação e resultados

Em Python, com ajuda da numpy, é possível montar o novo sistema linear com ajuda
de slices, como mostrado no código abaixo. Os arrays da numpy conseguem guardar
valores complexos, possuindo os atributos auxiliares ".imag" e ".real" para 
capturar cada uma das parcelas. O objeto np.s_ guarda a informação
de um conjunto de índices para um vetor ou matriz (o slice). Como sabemos que a matriz
nova (chamada "big_A") tem a propriedade de ser quatro vezes maior do que a
original, guardamos os índices para melhorar a leitura do código em questão.

{% highlight python linenos %}
def build_real_system(A, b):
    big_A = np.zeros(shape=(2*len(A), 2*len(A)))
    A0 = np.s_[0:len(A)]
    A1 = np.s_[len(A):2*len(A)]
    big_A[A0, A0] = A.real
    big_A[A1, A0] = A.imag
    big_A[A0, A1] = -A.imag
    big_A[A1, A1] = A.real
    
    big_b = np.zeros(shape=(2*len(b)))
    big_b[0:len(b)] = b.real
    big_b[len(b):2*len(b)] = b.imag
    
    return big_A, big_b
{% endhighlight %}

Agora, é só guardar o novo sistema linear e utilizar qualquer solver. Por exemplo:

{% highlight python linenos %}
N = 10

A = np.array([
    [1.0, 0.0, 0.0, 2.+5j],
    [0.0, 1.0+2.j, 0.0, 0.0],
    [0.0, 3.j, 1.0, 0.0],
    [0.0, 0.0, 0.0, 1.0],
])
b = np.array([2.0, 3.0, 4.0, 5.0])

big_A, big_b = build_real_system(A, b)

r = solve(big_A, big_b)

solution = np.zeros(shape=(len(b),), dtype=np.complex)
solution.real = r[:len(b)]
solution.imag = r[len(b):]

print(np.allclose(np.dot(A, solution) - b, 0.0))
{% endhighlight %}

Por hora é isso. Todo esse conteúdo ficará mais evidente quando eu começar a
resolver um exemplo real. Para quem tiver interesse, isso deve acontecer em
algum momento nas próximas semanas.

Até a próxima!
