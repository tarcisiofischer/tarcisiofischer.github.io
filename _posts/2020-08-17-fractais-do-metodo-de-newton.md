---
layout: post
title: Fractais do método de Newton
image: 2020-08-17/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-08-17/img001.png)
{: refdef}

Quando eu fiz o post sobre a solução de sistemas não lineares usando a PETSc,
um amigo comentou sobre esses "fractais do método de Newton". Eu não conhecia
antes disso, e fiquei interessado em tentar.


# Método de Newton

Também conhecido como método de Newton-Raphson, é um método iterativo para
estimar as raizes de uma função arbitrária. A ideia geral consiste em, a partir
de um chute inicial, calcular a raiz da equação linear dada pela reta tangente 
que passa pelo pondo dado por esse chute. A partir disso, encontra-se uma nova
posição, e o processo se repete, até que a solução esteja próxima o suficiente
do desejado.

A interpretação geométrica geralmente é mostrada para ilustrar o
método (veja a imagem abaixo). Na imagem, $x_0$ é o chute inicial. A linha
vermelha representa a função linear dada pela reta tangente a partir do chute
inicial. $x_1$ é a "nova posição", onde esse processo se repete. $x_2$ e $x_3$
são os resultados das iterações seguintes. Nesse caso simples, com poucas iterações
encontramos uma aproximação suficientemente boa.

{:refdef: style="text-align: center;"}
![](/images/2020-08-17/newton_ex.png)
{: refdef}

Matematicamente, o método é representado por:

$$
x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}
$$

Onde $f'$ representa a derivada da função $f$ em relação à x ($\frac{df}{dx}$).
Porém, o método pode falhar em encontrar uma solução, em alguns casos, e, em
outros casos, pode levar um número muito elevado de iterações.
O método pode ser utilizado também com números complexos, tendo a mesma forma
matemática que a mostrada acima.

# Fractais de Newton

Como sabemos, algumas raizes podem conter números complexos (imaginários). Além
disso, é possível que uma função contenha mais de uma raiz. A ideia para criar
os fractais é, dada uma função qualquer, criar uma imagem bidimensional
onde, para cada pixel, um chute inicial para o método de Newton é associado.
Todos os chutes serão números complexos. Cada
*linha* da imagem representa uma variação da componente imaginária do chute inicial. Cada
*coluna* da imagem representa uma variação da componente real do chute inicial. Portanto,
cada par (linha, coluna) será um chute inicial diferente.

Por exemplo, se variarmos a componente imaginária entre $(0, 2)$ com 200 pixels, e
a componente real entre $(0, 1)$ em 100 pixels, teremos o pixel $[0, 0]$
tendo valor $x_{guess} = 0\frac{1}{100} + 0\frac{2}{200}i$, o pixel
$[0, 1]$ com valor $x_{guess} = 0\frac{1}{100} + 1\frac{2}{200}i$, o pixel
$[1, 0]$ com valor $x_{guess} = 1\frac{1}{100} + 0\frac{2}{200}i$, e assim por diante.

Para cada *solução* (raiz) da função, uma cor é associada. Por exemplo,
para a função $f(x) = (x - 1)(x - 2)$, temos duas raizes: $x_0=1 + 0i$ e $x_1=2 + 0i$.
podemos associar vermelho para a solução $x_0$ e azul para $x_1$.
Dessa forma, as cores
da imagem representam qual solução foi encontrada para o chute associado àquele
pixel.

Além disso, a intensidade do pixel (mais claro ou mais escuro) é
determinado pela quantidade de iterações necessárias para chegar naquela
solução (Dado um número máximo de iterações). Por exemplo, se com $x_{guess}=5 + 2i$
o método encontra a solução $x_0$ em 3 iterações, a cor do pixel na posição
relativa à esse chute será um vermelho pouco escuro. Além disso, se a solução
não for encontrada
dentro daquele limite de iterações, desenha-se um pixel em branco.

Essa descrição foi baseada nos trabalhos apresentados
[nesse link](https://www.krofchok.com/fractals/index.html) e
[nesse link](https://mathforum.org/advanced/robertd/newtons.html). Eles
inclusive possuem soluções bem legais, também ;)


# Implementação em Python

Ao invés de usar uma implementação pronta do método de Newton, resolvi fazer
a implementação em python puro. O loop principal é mostrado abaixo:

{% highlight python linenos %}
for (l, xi), (c, xr) in itertools.product(enumerate(imag_space), enumerate(real_space)):
    if (l * N_COLS + c) % 100 == 0:
        print(f"{(l * N_COLS + c)} / {N_ROWS * N_COLS}")
    result[l, c] = choose_color_for(*solve_f(complex(xr, xi)))
{% endhighlight %}

A última linha é a mais relevante. Para cada par (xr, xi) (que representam as
componentes real e imaginária do chute inicial), as soluções são aproximadas
pela função *solve_f*, que retorna a aproximação da solução e o número de
iterações necessárias. Essas informações são repassadas para a função
*choose_color_for*, que retorna um número que representa uma cor. Para facilitar
o processo, aproveitei os *colormaps* da *matplotlib*, de forma que eu só preciso
mapear as soluções conhecidas para um mesmo valor real. Para controlar a
intensidade, somo um valor pequeno multiplicado pelo número de iterações,
conforme código abaixo:

{% highlight python linenos %}
known_solutions = []
def choose_color_for(solution, n_iterations):
    if solution is None:
        return -1

    for color, known_solution in enumerate(known_solutions):
        if np.isclose(known_solution, solution, atol=1e-1):
            break
    else:
        known_solutions.append(solution)
        color = len(known_solutions) - 1
    return 5. * color + 1. * n_iterations
{% endhighlight %}

Para evitar a necessidade de ter que implementar uma derivada para cada função
de entrada, fiz uma aproximação por diferenças finitas, que é usada
dentro da implementação do método de Newton. Tolerâncias de convergência e
o número máximo de iterações são dados por constantes globais (poderiam também
ser parâmetros da função).

{% highlight python linenos %}
def df_dx(f, x, fx):
    h = 1e-6
    return (f(x + h) - fx) / h

def solve_f(x_guess):
    x = x_guess
    x_prev = x + 2e-4
    n_iterations = 0
    while abs(x_prev - x).real > CONVERGENCE_TOL and n_iterations < MAX_ITER:
        fx = f(x)
        x_prev = x
        x = x - fx / df_dx(f, x, fx)
        n_iterations += 1
    if n_iterations == MAX_ITER:
        return None, None
    return x, n_iterations
{% endhighlight %}


# Soluções

## Caso 1

$$f(x) = -x^3 + 9x^2 - 18x + 6$$

Resolução = 50x50px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case1-50x50-20it.png)
{: refdef}

Resolução = 150x150px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case1-150x150-20it.png)
{: refdef}

Resolução = 300x300px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case1-300x300-20it.png)
{: refdef}


## Caso 2

$$f(x) = x^5 - 1$$

Resolução = 300x300px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case2-300x300-20it.png)
{: refdef}

Resolução = 300x300px, max 40 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case2-300x300-40it.png)
{: refdef}

Resolução = 300x300px, max 80 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case2-300x300-80it.png)
{: refdef}


## Caso 3

$$f(x) = x^3 - x - 1$$

Resolução = 50x50px, max 40 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case3-50x50-40it.png)
{: refdef}

Resolução = 150x150px, max 40 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case3-150x150-40it.png)
{: refdef}

Resolução = 300x300px, max 40 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case3-300x300-40it.png)
{: refdef}


## Caso 4

$$f(x) = (x^2 - 11)^2 + (x - 7)^2$$

Resolução = 50x50px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case4-50x50-20it.png)
{: refdef}

Resolução = 150x150px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case4-150x150-20it.png)
{: refdef}

Resolução = 300x300px, max 20 iterações
{:refdef: style="text-align: center;"}
![](/images/2020-08-17/case4-300x300-20it.png)
{: refdef}

Até a próxima!
