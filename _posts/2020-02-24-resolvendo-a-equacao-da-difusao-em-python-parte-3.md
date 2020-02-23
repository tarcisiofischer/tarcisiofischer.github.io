---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 3
---

Chegamos ao terceiro post da série. Até agora, ignorando o termo fonte,
temos todos os termos para montar e resolver o sistema linear. Nesse post,
nos concentraremos nessa, que é uma das partes que eu mais gosto nesse processo!
No próximo post da série, finalmente colocaremos todo o processo em um loop para
resolver a parcela transiente, e faremos a visualização dos resultados.

Conforme pedidos, vou extender essa série para 5 posts (Ao invés de 4). No
quinto, trabalharemos um pouco na integração de Python com C++, ou seja,
chamaremos uma função implementada em C++ a partir do Python. Aguardem ;)

# Solução do sistema linear

Existem algumas formas de se resolver um sistema linear. Se você não possui
nenhum conhecimento nessa área, possivelmente vai pensar em fazer a seguinte
sequência de operações:

$$
Ax = b
$$

$$
A^{-1}Ax = A^{-1}b
$$

$$
x = A^{-1}b
$$

Portanto, para resolver o problema, precisamos "simplesmente" inverter a matriz
$A$ e multiplicar pelo vetor $b$. De fato, isso funciona, conforme pode ser
observado no código a seguir:

{% highlight python linenos %}
import numpy as np
from matplotlib import pyplot as plt

ds = 0.1
dt = 0.1

c = 0.1

N = 20

to_flat_index = lambda x, y: x * N + y

A = np.zeros(shape=(N * N, N * N))
for i in range(N):
    for j in range(N):
        x = to_flat_index(i, j)

        y = to_flat_index(i, j)
        A[x, y] = ds**2 + 4 * c * dt
        if i - 1 >= 0:
            y = to_flat_index(i - 1, j)
            A[x, y] = -c * dt
        if i + 1 < N:
            y = to_flat_index(i + 1, j)
            A[x, y] = -c * dt
        if j - 1 >= 0:
            y = to_flat_index(i, j - 1)
            A[x, y] = -c * dt
        if j + 1 < N:
            y = to_flat_index(i, j + 1)
            A[x, y] = -c * dt

fo = np.zeros(shape=(N, N,))
fo[5:15, 5:15] = 10.0
b = ds ** 2. * fo.reshape((N * N,))

def solve_2d(A, b):
    x = np.dot(np.linalg.inv(A), b)
    return x.reshape((N, N))

result = solve_2d(A, b)
plt.imshow(result)
plt.show()
{% endhighlight %}

{:refdef: style="text-align: center;"}
![Solução do sistema linear](/images/post3-img1.png)
{: refdef}

O procedimento de inversão de uma matriz, porém, é bastante caro
computacionalmente. Dessa forma, é comum optarmos por outros métodos que
solucionam ou aproximam a solução do sistema linear, seja por métodos iterativos
ou por métodos diretos.

Antes que alguém comente, é verdade que, para casos
simples como o do presente post, é possível inverter a matriz uma vez e
guarda-la, de forma que a solução do sistema fica reduzido a um produto
matriz-vetor. Porém, vou considerar aqui que a inversão faz parte do processo de
solução, sendo **sempre** necessário calcular a inversão novamente. Isso em
geral vai ser o caso, em problemas do dia-a-dia, portanto, é uma hipótese
relevante de ser admitida.

Meu objetivo aqui não é dar uma aula sobre métodos de solução de sistemas
lineares (Muito embora seria muito legal). Ao invés disso, vou apenas
utilizar e comparar o tempo de execução de alguns solvers lineares disponíveis
na SciPy. Vale mencionar que existem outras bibliotecas com solvers lineares.
Uma que me sinto obrigado a comentar é a [PETSc](https://www.mcs.anl.gov/petsc/).
Mas vamos nos manter no "feijão com arroz" do desenvolvedor numérico, por
enquanto, utilizando as funções da SciPy.

Abaixo, deixo a parte relevante do script Python que compara a solução do
sistema linear utilizando (na ordem):
(a) [Inversão da matriz](https://en.wikipedia.org/wiki/Invertible_matrix#Methods_of_matrix_inversion) $A$,
(b) [Fatoração LU](https://en.wikipedia.org/wiki/LU_decomposition),
(c) Fatoração LU sparsa,
(d) [Fatoração ILU](https://en.wikipedia.org/wiki/Incomplete_LU_factorization),
(e) [GMRES](https://en.wikipedia.org/wiki/Generalized_minimal_residual_method).

{% highlight python linenos %}
def solve_2d_inv_A():
    x = np.dot(np.linalg.inv(A), b)
    return x.reshape((N, N))

def solve_2d_linalg_solve():
    x = np.linalg.solve(A, b)
    return x.reshape((N, N))

def solve_2d_linalg_spsolve():
    x = linalg.spsolve(sp_A_csr, b)
    return x.reshape((N, N))

def solve_2d_linalg_ilu():
    invA = linalg.spilu(sp_A_csc)
    x = invA.solve(b)
    return x.reshape((N, N))

def solve_2d_linalg_gmres():
    x, _ = linalg.gmres(sp_A_csr, b)
    return x.reshape((N, N))
{% endhighlight %}

Com o detalhe que os métodos esparsos necessitam da matriz $A$ convertida para
esparsa. Tentei "documentar" isso no código por meio do nome das variáveis.
Dessa forma, sp_A_csr é a versão esparsa (sp) de $A$ em CSR, e sp_A_csc a
versão esparsa em CSC.

# Breve comparação entre os diferentes métodos

Para fazer a comparação, estou utilizando o [pandas](https://pandas.pydata.org/).
Montei uma pequena estrutura de dicionários pra guardar os dados de cada caso.
Aqui, não vou observar o erro (imprecisão dos solvers) na solução final, e não
vou utilizar pré-condicionadores.

A ideia geral é simplesmente executar cada um dos métodos, variando o tamanho
da matriz com $N=10$ até $N=40$, e guardar o tempo de execução para resolver
o sistema linear 1000 vezes (Utilizando [timeit](https://docs.python.org/2/library/timeit.html)).

O programa inteiro é bem simplório, e, claro, não é algo para ser utilizado em
casos reais. Mas como exercício de visualização e aprendizado, eu particularmente
acho bastante válido. Sugiro que copie esse código, cole no seu editor preferido
e teste, modifique e faça experimentos. Existem vários conceitos legais nesse
pequeno snippet, que vão além da questão da solução do sistema linear: listas,
dicionários cujos valores são funções, matplotlib em conjunto com pandas para
gerar o plot de barras.

Cuidado, porém, com a questão de uso de variáveis globais. Gerei esse snippet
rapidamente e decidi que as funções build_A e build_b atualizariam variáveis
globais. Essa prática não é legal por vários motivos. Um bastante óbvio é que,
dessa forma, a paralelização do snippet não seria tão trivial. Deixo de
exercício ao leitor melhorar o código e resolver esses problemas ;)

{% highlight python linenos %}
solver_names = [
    'inv_A',
    'dense_LU',
    'sparse_LU',
    'sparse_ILU',
    'GMRes'
]
solver_name_to_function = {
    'inv_A': solve_2d_inv_A,
    'dense_LU': solve_2d_linalg_solve,
    'sparse_LU': solve_2d_linalg_spsolve,
    'sparse_ILU': solve_2d_linalg_ilu,
    'GMRes': solve_2d_linalg_gmres,
}
data_dict = {s: {} for s in solver_names}
sizes = [10, 15, 20, 25, 30, 35, 40]

def case_takes_too_long(s, N):
    '''
    Helper function to identify those cases that takes too long to run,
    so that we can skip them.
    '''
    return (
        s == 'inv_A' and N > 25 or
        s == 'dense_LU' and N > 30 or
        False
    )

for N in sizes:
    build_A() # Build A matrix and update global variables
    build_b() # Build b vector and update global variables
    for s in solver_names:
        print(f'Running {s} with N={N}...', end='')
        if case_takes_too_long(s, N):
            print(' (skipped)')
            continue
        f = solver_name_to_function[s]
        data_dict[s][N] = timeit.timeit(f, number=1000,)
        print(' Done.')

fig, ax = plt.subplots(len(data_dict), 1)
df = pd.DataFrame(data_dict)

print("RESULTS")
print(df)

df.plot.bar(
    rot=0,
    subplots=True,
    sharex=True,
    title=[''] * len(data_dict),
    ax=ax
)
plt.xlabel("N")
[x.set_ylabel('time [s]') for x in ax]
plt.show()
{% endhighlight %}

Atenção para essa questão de converter a matriz densa para esparsa: No
último post, comentei que a construção da matriz esparsa precisava de um cuidado
adicional e, agora, estou falando que é possível construir uma matriz esparsa
a partir de uma densa. Perceba que, caso a matriz seja suficientemente grande,
você não vai nem conseguir gera-la densa para posterior conversão. Então, em
casos muito grandes, não existiria essa opção de fazer a conversão direta para
matriz esparsa.

# Resultados e conclusão

Foram, então, gerados dois resultados relevantes com a execução do código acima:
(a) O print com os tempos absolutos e (b) O plot comparativo para cada método.
Os dois contém a mesma informação, mas com apresentação diferente:

{% highlight python linenos %}
        inv_A   dense_LU  sparse_LU  sparse_ILU     GMRes
10   0.402125   0.205415   0.374370    0.369797  1.273823
15   2.672651   0.916614   0.721103    0.752904  1.398468
20  10.229078   3.763351   1.211724    1.346811  1.421030
25  40.561536  14.481547   2.017594    2.254834  1.514201
30        NaN  38.811441   2.959303    3.436828  1.646307
35        NaN        NaN   3.828993    4.631033  1.825023
40        NaN        NaN   5.206256    6.056734  1.865046
{% endhighlight %}

{:refdef: style="text-align: center;"}
![Resultados](/images/time_comparison.png)
{: refdef}

O tempo computacional relacionado aos métodos diretos cresce muito rapidamente,
o que não é novidade para quem já estudou sobre, mas é muito legal ver na
prática, ao invés de simplesmente acreditar na palavra do professor. Mesmo sem
pré-condicionador, o GMRes se mostra uma excelente alternativa (Lembrando que
eu não comparei o erro da solução para cada método). A fatoração LU incompleta
se mostra uma alternativa razoável, e talvez mais conhecida para alguns
estudantes.

Termino por aqui nossa breve discussão sobre a solução de sistemas
lineares em Python com Numpy/Scipy. No próximo post, utilizaremos todo nosso
trabalho para finalizar o loop transiente, e gerar imagens utilizando a
matplotlib. Peço pro pessoal que está acompanhando que me dê um toque nas
redes sociais caso tenham sugestões de temas futuros ou comentários.

Até a próxima!
