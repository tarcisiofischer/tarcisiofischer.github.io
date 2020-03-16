---
layout: post
title: Paralelização da matriz de rigidez de elementos finitos em Python com joblib
---

Em linguagens como Matlab e C++ com [openmp](https://www.openmp.org/), temos a possibilidade de paralelizar
for-loops de maneira bastante simples, por exemplo, com o uso do [parfor](https://www.mathworks.com/help/parallel-computing/parfor.html)
ou do [pragma omp for](https://www.openmp.org/wp-content/uploads/openmp-examples-5.0.0.pdf), respectivamente.
Em Python, existem algumas maneiras de trabalhar com paralelização. Lembrando sempre
que não é uma tarefa tão trivial, por conta da [GIL](https://docs.python.org/2/glossary.html#term-global-interpreter-lock).
Métodos baseados em [multiprocessing](https://docs.python.org/2/library/multiprocessing.html)
são geralmente preferidos por esse motivo (Embora dependendo do contexto, o módulo
[threading](https://docs.python.org/2/library/threading.html) pode ser bastente útil). 
O uso de bibliotecas especializadas é uma boa alternativa, para problemas mais específicos.
Eu, particularmente, gosto bastante da [joblib](https://joblib.readthedocs.io/),
pela relativa facilidade em paralelizar pequenas porções de código ["embaraçosamente paralelizáveis"](https://en.wikipedia.org/wiki/Embarrassingly_parallel).

Na aproximação de solução de equações diferenciais parciais utilizando a técnica
de [elementos finitos](https://pt.wikipedia.org/wiki/M%C3%A9todo_dos_elementos_finitos),
a matriz de rigidez global é obtida em um procedimento conhecido como "assembly".
Basicamente, é uma operação de sobreposição das matrizes elementares.
Essa operação pode ser custosa dependendo do número de elementos (Quanto mais elementos, maior a matriz), mesmo
nos casos que ela só é calculada uma única vez. Normalmente, é utilizada uma estrutura
de dados de matriz esparsa para compor essa matriz global.
Nesse post, porém, farei as operações em matriz cheia apenas para dar maior ênfase
no processo de paralelização utilizando a joblib.


# Montagem da matriz de rigidez

O estudo de caso apresentado aqui será o da montagem da matriz de rigidez para
a discretização pelo método de elementos finitos. O procedimento
consiste em dividir uma matriz grande e esparsa em diversas matrizes menores,
computando cada uma delas individualmente e posteriormente fazendo a soma posicionada
na matriz global. A figura abaixo ilustra essa ideia. Cada bloco colorido é uma matriz
elementar, enquanto a sobreposição delas na matriz maior forma a matriz de rigidez global.

{:refdef: style="text-align: center;"}
![](/images/post6-img2.png)
{: refdef}

O código da função referente a isso, proveniente de um programa de elementos finitos,
foi extraído e é apresentado abaixo. Essa extração ajuda a focar apenas
no problema em discussão.

{% highlight python linenos %}
def run_serial():
    Ke = np.zeros(shape=(n_points, n_points))
    for e in elements:
        element_points = np.array([points[pid] for pid in e])
        Ke_local = build_local_Ke(element_points, omega)
        for k, p1 in enumerate(e):
            for l, p2 in enumerate(e):
                Ke[p1, p2] += Ke_local[k, l]
    return Ke
{% endhighlight %}

Assumindo que a função "build_local_Ke" é computacionalmente custosa, e que o número
de elementos é grande, torna-se interessante a paralelização dessa função.


# Paralelização com joblib

Partindo da [documentação oficial da joblib](https://joblib.readthedocs.io/en/latest/parallel.html),
encontramos um exemplo simples de uso da biblioteca. Fiz uma pequena modificação para
tentar elucidar a transformação de código. A ideia é visualizar o mapeamento e aplicar
a mesma técnica no nosso problema. Observe, nos códigos abaixo, como foram movimentados
a função sqrt, o for-loop, e a geração da lista de resultados. No caso da joblib, a função
Parallel() retornará uma lista de resultados.

{% highlight python linenos %}
def f():
    results = []
    for i in range(10):
        results.append(sqrt(i ** 2))
    return results
{% endhighlight %}

{% highlight python linenos %}
def g():
    def _g(x):
        return sqrt(x)
    return Parallel(n_jobs=2)(delayed(_g)(i ** 2) for i in range(10))
{% endhighlight %}

Para a montagem da matriz global de rigidez, nota-se uma leve dificuldade: Caso fosse feito uma simples
paralelização sobre o loop, haveria uma possível [condição de corrida](https://pt.wikipedia.org/wiki/Condi%C3%A7%C3%A3o_de_corrida)
sobre operações envolvendo a soma no vetor global. Existem várias formas de evitar tal problema. A
que vou escolher aqui é a de separar essa parcela de código para fora do loop. O
resultado final é, então:

{% highlight python linenos %}
from joblib import Parallel, delayed
def run_parallel():
    def _compute_element(e):
        element_points = np.array([points[pid] for pid in e])
        Ke_local = build_local_Ke(element_points, omega)
        return (e, Ke_local)
    Ke_local_list = Parallel(n_jobs=6)(delayed(_compute_element)(e) for e in elements)

    Ke = np.zeros(shape=(n_points, n_points))
    for (e, Ke_local) in Ke_local_list:
        for k, p1 in enumerate(e):
            for l, p2 in enumerate(e):
                Ke[p1, p2] += Ke_local[k, l]
    return Ke_local_list
{% endhighlight %}


# Resultados e conclusão

Uma breve comparação é apresentada abaixo. Foi computada as matrizes globais com
$50^2$, $100^2$, $150^2$, $200^2$ e $250^2$ elementos. Para o loop paralelo, foram
utilizadas 4 threads. Nota-se que, para poucos elementos, não há grandes ganhos e,
a medida que o número aumenta, o ganho começa a ser expressivo. O código para o plot
é muito semelhante ao que foi feito em posts anteriores, e é apresentado abaixo
apenas por completude.

{:refdef: style="text-align: center;"}
![](/images/post6-img1.png)
{: refdef}

{% highlight python linenos %}
data_dict = {}
data_dict['N'] = domain
data_dict['serial'] = serial_times
data_dict['parallel'] = parallel_times

from matplotlib import pyplot as plt
import pandas as pd
fig, ax = plt.subplots(1, 1)
df = pd.DataFrame(data_dict)

df.plot.bar(
    rot=0,
    x='N',
    sharex=True,
    ax=ax
)
plt.xlabel("N")
plt.ylabel("time [s]")
plt.show()
{% endhighlight %}

A análise feita aqui é bastante superficial, apenas para ilustrar o uso da biblioteca
em um caso simples. Claro que a joblib não é uma biblioteca que resolve todos os problemas
relacionados a paralelismo. Mas é uma biblioteca bastante interessante como alternativa
para esses casos "embarrassingly parallel" ou que sejam próximos disso e exijam
pouca modificação de código e cuidados em gerenciar memória ou outros compartilhamento
de recursos. Espero que dê algum insight para outros usos ;)

Até a próxima!
