---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 4
---

Finalmente, temos todos os "pedaços" de código para completar nosso "solver"
da equação de difusão. Nesse post, quero aproveitar para fazer uma breve
discussão sobre como adicionar plots dos resultados. No
próximo post da série, vamos trabalhar com a integração Python/C++, fechando a
série de posts sobre o assunto.

# Juntando as partes

Em primeiro lugar, deixo aqui o código final, já com a solução do sistema
transiente (Que, acredito, é trivial o suficiente para entender sem grandes
comentários). Básicamente, dada uma condição inicial, existe um loop que itera
no tempo e vai resolvendo o sistema até um dado tempo final. Esse código não
possui saída de plots ou resultados.

{% highlight python linenos %}
import numpy as np
from scipy.sparse import linalg
from scipy.sparse import csr_matrix, csc_matrix

def build_A(N, c, ds, dt):
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

    return A


def build_b(N, ds, fo):
    return ds ** 2. * fo.reshape((N * N,))


def solve_linear_system(A, b):
    x, _ = linalg.gmres(A, b)
    return x


def solve_diffusion_equation(N, c, ds, dt, final_time, initial_conditions):
    A = build_A(N, c, ds, dt)
    fo = np.zeros(shape=(N, N))
    fo[:] = initial_conditions
    time = 0.0
    while time <= final_time:
        b = build_b(N, ds, fo)
        x = solve_linear_system(A, b)

        result = x.reshape((N, N))
        fo[:] = result
        time += dt

N = 40
ics = np.zeros(shape=(N, N,))
ics[N//4:N - N//4, N//4:N - N//4] = 10.0
solve_diffusion_equation(
    N,
    c=0.1,
    ds=0.1,
    dt=0.1,
    final_time=2.0,
    initial_conditions=ics
)
{% endhighlight %}


# Plots estáticos com matplotlib

O objetivo aqui é principalmente estudar a separaça da visualização de dados da
questão de solução do modelo. Ao invés de fazer todo o
processo de simulação para só depois mostrar os
resultados, vamos mostrar a medida que a solução é produzida.

Um jeito simples de fazer isso, é por meio de uma flag no código, que controla
se o plot será mostrado. O código seria algo assim:

{% highlight python linenos %}
def solve_diffusion_equation(N, c, ds, dt, final_time, initial_conditions, plot_results=True):
    A = build_A(N, c, ds, dt)
    fo = np.zeros(shape=(N, N))
    fo[:] = initial_conditions
    time = 0.0
    while time <= final_time:
        b = build_b(N, ds, fo)
        x = solve_linear_system(A, b)

        result = x.reshape((N, N))
        fo[:] = result
        time += dt

        if plot_results:
            from matplotlib import pyplot as plt
            plt.imshow(result, vmin=0.0, vmax=10.0)
            plt.show()
{% endhighlight %}

Existem pelo menos dois motivos simples pelos quais isso é ruim:

**(1) Adiciona código
específico de uso de uma biblioteca de visualização (No nosso caso, matplotlib)
na função que resolve o sistema.**

Isso é ruim pois, se você quiser passar esse código para alguém que nem quer
ver os resultados, essa pessoa vai precisar de qualquer forma baixar a
matplotlib ou, no mínimo, "caçar" os locais e remover código espalhado.
Nesse exemplo simples, isso não fica tão evidente, e talvez algumas pessoas
nem se incomodem tanto. Mas a medida que o software cresce, isso pode vir a ser
um problema (Até pra manter o código).

**(2) Cada nova funcionalidade adicionaria mais flags.**

Por exemplo, para salvar
opcionalmente os resultados, teria-se que adicionar uma nova flag save_results,
ou algo do tipo.

Eu poderia continuar listando outros problemas: Difícil manter esse código, caso
ele seja compartilhado em uma equipe (Pois a tendência é que todos comecem
a mexer no mesmo código); Difícil leitura entre código de solver e código de
plot; Difícil de alterar a biblioteca de plot, pois teria que procurar todos
os locais espalhados onde a matplotlib está sendo usada; entre outros...

Ao invés disso, uma sugestão para esse caso é [injetar](https://en.wikipedia.org/wiki/Dependency_injection)
uma função que vai fazer
o plot via parâmetro, de forma que tenhamos uma forma mais genérica de processar
os dados:

{% highlight python linenos %}
def solve_diffusion_equation(N, c, ds, dt, final_time, initial_conditions, results_callbacks=None):
    if results_callbacks is None:
        results_callbacks = []

    A = build_A(N, c, ds, dt)
    fo = np.zeros(shape=(N, N))
    fo[:] = initial_conditions
    time = 0.0
    while time <= final_time:
        b = build_b(N, ds, fo)
        x = solve_linear_system(A, b)

        result = x.reshape((N, N))
        fo[:] = result
        time += dt

        for callback in results_callbacks:
            callback(results)
{% endhighlight %}

Perceba que removemos a dependência da matplotlib no código acima. Claro que,
da forma como está, o algoritmo não vai mais plotar nada. E de qualquer forma
teremos que implementar a função de plot em outro lugar e passar como parâmetro,
como mostrado abaixo. Mas ganhamos a possibilidade de injetar
qualquer outra função para lidar com os resultados. Podemos salvar os resultados
em arquivo, adicionar plots, ou fazer outras operações sem ter que mexer
na função "solve" diretamente. Vamos testar possibilidades:

{% highlight python linenos %}
def plot_results(result):
    from matplotlib import pyplot as plt
    plt.imshow(result, vmin=0.0, vmax=10.0)
    plt.show()

def plot_results_at_center(result):
    from matplotlib import pyplot as plt
    center_results = result[:, 20]
    plt.plot(np.arange(len(center_results)), center_results)
    plt.show()

N = 40
ics = np.zeros(shape=(N, N,))
ics[N//4:N - N//4, N//4:N - N//4] = 10.0
solve_diffusion_equation(
    N,
    c=0.1,
    ds=0.1,
    dt=0.1,
    final_time=2.0,
    initial_conditions=ics,
    results_callbacks=[plot_results, plot_results_at_center]
)
{% endhighlight %}

{:refdef: style="text-align: center;"}
![](/images/post4-img1.png)
{: refdef}

{:refdef: style="text-align: center;"}
![](/images/post4-img2.png)
{: refdef}

Essa alternativa de implementação é bastante simples, mas também tem lá suas
desvantagens. Uma delas é que programadores iniciantes geralmente demoram um
pouco para compreender o fluxo de execução das coisas, pois o comportamento
foi separado do local onde ele é de fato utilizado. Outra é que, dependendo
da situação, a callback começa a precisar de variáveis específicas do algoritmo
principal, e deve-se tomar um cuidado criterioso para não expor muitos dos
detalhes de implementação, se não a complexidade cognitiva para compreensão
do código pode começar a explodir.

Uma última (apesar de poder listar outras),
é que deve-se tomar muito cuidado para **não alterar as variáveis** no momento
que a callback é chamada. Isso pode trazer bugs estranhos na simulação, por
exemplo, imagine que ao invés de plotar, a callback somasse uma constante
a todos os valores de resultado. Isso resultaria em um caos, para quem estivesse
debugando.


# Plots dinâmicos com a matplotlib

Como os plots na alternativa anterior são estáticos, é necessário ficar fechando
o plot para ver o próximo resultado. Para evitar isso, é possível implementar
uma callback que mostra dinâmicamente a medida que os resultados são calculados.

A modelagem dessa nova função lembra um algoritmo [produtor-consumidor](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem):
O solver
vai produzindo resultados e adicionando em uma fila para que o plotter vá
consumindo as imagens dessa fila e mostrando na tela. Para conseguir esse
efeito, precisamos compartilhar memória entre as duas threads: A que produz
os dados e a que consome os mesmos.

Aqui, fazemos uma solução super-simplificada apenas para ilustração, mas que
funciona satisfatóriamente. A variável compartilhada "frame_list" é compartilhada
como uma variável local no escopo da função "animated_plot". Essa variável é
atualizada a cada vez que "plot_results" é chamada, adicionando uma nova imagem e
colocando a thread atual para dormir por um curto período de tempo (Na verdade,
eu gostaria de chamar o event-loop diretamente, mas não encontrei a função na
matplotlib). Finalmente, quando tiver a chance, a matplotlib chama a função
"update_f" em um intervalo mínimo de 200ms, sobreescrevendo a imagem atual.

Um detalhe, porém, é que ao final da simulação, é possível que nem todas as
imagens tenham sido consumidas. Para resolver esse problema, no final do
programa forçamos plt.show() a bloquear para que o programa não termine. O
resultado final pode ser acompanhado abaixo:

{% highlight python linenos %}
from matplotlib import pyplot as plt
from matplotlib.animation import FuncAnimation
def animated_plot(vmin, vmax):
    fig = plt.figure()
    ln = plt.imshow(ics, vmin=vmin, vmax=vmax)

    frame_list = []
    def update_f(frame):
        nonlocal frame_list
        if len(frame_list) > 0:
            ln.set_data(frame_list.pop(0))
        return (ln,)
    ani = FuncAnimation(fig, update_f, interval=200)
    plt.show(block=False)

    def plot_results(result): 
        nonlocal frame_list
        frame_list.append(result.copy())
        plt.pause(0.001) # Give a chance to plot

    return plot_results
{% endhighlight %}

{% highlight python linenos %}
solve_diffusion_equation(
    N,
    c=0.1,
    ds=0.1,
    dt=0.1,
    final_time=2.0,
    initial_conditions=ics,
    results_callbacks=[animated_plot(0.0, 10.0),]
)

{:refdef: style="text-align: center;"}
![](/images/post4-img3.gif)
{: refdef}

# Avoid exiting before all images are shown
from matplotlib import pyplot as plt
plt.show(block=True)
{% endhighlight %}

Fica como exercício ao leitor tentar fazer o plot dos valores centrais
dinâmicamente ;) Outros exercícios legais, que ficam como sugestão para quem
está usando esses posts para estudar: Adicionar uma callback para mostrar
na tela quantos porcento do processo já foi concluído; Adicionar uma callback
para salvar os resultados parciais em arquivos texto; Uma callback para
inspecionar os valores de uma região e, caso algum deles for menor que um
limite mínimo (Ex.: 8.0), mostrar um "WARNING".

Acredito que esse tenha sido o post mais curto da série, mas ainda assim, eu
sei que vai ter grande valor para algumas pessoas que tinham me perguntado como
fazer um jeito de fazer os plots não ficarem sempre expalhados pelo código
de simulação. Claro que essa pode não ser a melhor solução para todos os casos,
e talvez não seja nem mesmo a melhor para o presente caso, dependendo dos
inputs. Sintam-se livres para me procurar e comentar sobre pelas redes sociais
:)

Até a próxima!
