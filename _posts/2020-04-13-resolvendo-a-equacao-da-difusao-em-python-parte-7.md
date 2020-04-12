---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 7
---

Estou reabrindo a série pra adicionar mais um tópico que ficou pra trás.
O objetivo desse post é mostrar como evoluir o projeto para suportar malhas
maiores, e resolver a equação de difusão em Python em uma malha
1000 x 1000, ou seja, 1 milhão de elementos. Para ter noção do tamanho do
problema, é importante lembrar que o tamanho da matriz para a solução do sistema
linear tinha tamanho $(N \cdot N)^2$ elementos, ou seja, nesse caso, seria
necessário guardar em memória um total de $1000000^2$ valores. Claro que não
faremos isso - Aproveitaremos a esparsidade da matriz para resolver tal problema.

{:refdef: style="text-align: center;"}
![](/images/2020-04-13/img001.png)
{: refdef}


# Esparsidade da matriz $A$

Quando dizemos
que uma matriz é esparsa, queremos dizer que a maior parte de seus elementos
possui valor nulo. Como comentei em outros posts, o problema de difusão discretizado com diferenças
finitas gera um sistema linear ($Ax = b$) cuja matriz $A$ é esparsa (banda).
Nos posts anteriores, apesar de ter esse conhecimento, mantive a matriz cheia,
ou seja, ignorei essa esparsidade, para focar em outras questões. 
A imagem abaixo ilustra essa característica: As cores em roxo claro representam
os zeros, enquanto as outras cores representam valores na matriz.

{:refdef: style="text-align: center;"}
![](/images/2020-04-13/img002.png)
{: refdef}

Dado que a maior parte dos valores são zeros, não precisamos guardar em memória todos
os elementos da matriz, mas apenas os elementos cujo valor são não nulos.
Existem várias possíveis representações para guardar elementos de uma matriz de
forma esparsa. Três formatos comuns, que inclusive estão disponíveis na scipy,
são o formato COO (coordinates), CSR (Compressed Sparse Rows) e CSC (Compressed
Sparse Columns). Cada um deles possui vantagens e desvantagens.
Por exemplo, o formato COO possui a vantagem de ser simples de construir,
pois precisamos apenas saber as coordenadas e o valor não nulo presente nessa
coordenada. Por outro lado, nem todo solver recebe esse tipo de formato como
entrada. A biblioteca SciPy possui conversores, que produzem
outros formatos a partir do formato COO.

Apenas para fins comparativos, se considerarmos
a matriz densa, o número de valores para guardar em memória seria de $1000000^2$.
Considerando que cada valor ocupa 8 bytes de memória, precisariamos de aproximadamente
7450Gb apenas para guardar os elementos, enquanto que, tirando proveito da
esparsidade do caso, precisaremos de aproximadamente 40Mb.

Uma forma fácil de descobrir o consumo de memória utilizando a numpy é por meio
do atributo nbytes, disponível em qualquer ndarray:

{% highlight python linenos %}
>>> A = np.zeros(shape=(1000*1000, 1000*1000))
>>> A.nbytes
8000000000000 # bytes
>>> A.nbytes / 1024. / 1024. / 1024. # gigabytes (Gb)
7450.580596923828
{% endhighlight %}

Uma discussão a parte, seria a de trabalhar para evitar guardar a matriz A,
por exemplo, por meio de um algoritmo [TDMA](https://en.wikipedia.org/wiki/Tridiagonal_matrix_algorithm) ou semelhantes. Porém,
nesse post, eu quis apenas mostrar o uso das estruturas de matrizes esparsas 
da SciPy, e como uma modificação simples em um código existente torna o mesmo
capaz de resolver uma gama consideravelmente maior de problemas. As estruturas
de matrizes esparsas são ainda mais relevantes na solução de sistemas
lineares que não são tridiagonais ou banda.


# Modificação do projeto e solução

Abaixo, mostro o código de construção da matriz A. Copiei a estrutura do primeiro post
da série, e fiz pequenas alterações para construir a matriz
esparsa, ao invés da matriz densa, utilizando COO matrix. Infelizmente, desconheço
uma forma mais simples do que essa de guardar individualmente cada valor em conjunto
com sua posição, antes de criar a estrutura de dados. Uma melhoria adicional seria a de paralelizar
essa montagem, visto que, apesar de acontecer apenas no início do programa, ela
chega a tomar um tempo considerável de processamento, a medida que o tamanho
da malha aumenta (um profiling do código poderia validar minha hipótese ;).

{% highlight python linenos %}
def build_A(N, c, ds, dt):
    to_flat_index = lambda x, y: x * N + y

    coo_i = []
    coo_j = []
    coo_data = []
    def set_A(x, y, value):
        coo_i.append(x)
        coo_j.append(y)
        coo_data.append(value)

    for i in range(N):
        for j in range(N):
            x = to_flat_index(i, j)
            y = to_flat_index(i, j)
            set_A(x, y, ds**2 + 4 * c * dt)
            if i - 1 >= 0:
                y = to_flat_index(i - 1, j)
                set_A(x, y, -c * dt)
            if i + 1 < N:
                y = to_flat_index(i + 1, j)
                set_A(x, y, -c * dt)
            if j - 1 >= 0:
                y = to_flat_index(i, j - 1)
                set_A(x, y, -c * dt)
            if j + 1 < N:
                y = to_flat_index(i, j + 1)
                set_A(x, y, -c * dt)

    A = coo_matrix((coo_data, (coo_i, coo_j)), shape=(N * N, N * N))
    return A.tocsc()
{% endhighlight %}

Para deixar o resultado um pouco mais interessante do que um quadrado, como
nos posts anteriores, fiz a leitura de uma imagem em PNG e executei o algoritmo
utilizando a mesma como condição inicial. Isso não faz diferença no tempo de
execução ou na solução do problema, mas o resultado fica um pouco mais legal :)
Infelizmente, o arquivo gif sem compressão ficou muito grande, então eu cortei
o mesmo só pra ter uma noção e salvei o resultado no último tempo em png
estático abaixo:

{:refdef: style="text-align: center;"}
![](/images/2020-04-13/img003.gif)
{: refdef}

{:refdef: style="text-align: center;"}
![](/images/2020-04-13/img004.png)
{: refdef}

Até a próxima!
