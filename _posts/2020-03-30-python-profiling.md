---
layout: post
title: Profiling do problema da cavidade implementado em Python
---

Profiling de código é a análise dinâmica destinada a medir propriedades do software, tais como
tempo e espaço. Ferramentas de profiling são necessárias para tomar decisões sobre como otimizar códigos,
pois provê
informações sobre a execução do sistema sendo analisado. Claro que os resultados crus da ferramenta 
não são suficientes para tomar decisões, pois deve-se levar em consideração o contexto e a forma como
foram obtidos. O objetivo desse post é ilustrar brevemente o uso do cProfile em Python para a análise
de performance de um simulador sobre o problema da cavidade.

{:refdef: style="text-align: center;"}
![Profiling do problema da cavidade implementado em Python](/images/2020-03-30-python-profiling/img001.png)
{: refdef}


# Visão geral do problema & implementação

Um problema clássico, ensinado em cursos de métodos computacionais em mecânica dos fluidos, é o problema
da cavidade (Lid driven cavity). Resolve-se as equação de
Navier-Stokes, assumindo um
fluido newtoniano e incompressível, em um domínio quadrado (embora existam versões que
resolvem o domínio tridimensional), cujas bordas laterais e inferior estão fechadas, enquanto a superior
contém uma "tampa" com velocidade constante na direção x, conforme ilustração abaixo.

{:refdef: style="text-align: center;"}
![Exemplo - Problema da Cavidade](/images/2020-03-30-python-profiling/img002.png)
{: refdef}

As equações relevantes são resumidas abaixo. As incógnitas são os campos de velocidade $u$, $v$ e o de pressão $P$, enquanto
a distribuição das outras propriedades são conhecidas e constantes, tanto no espaço quanto no tempo. A discretização
não é mostrada aqui, mas foi utilizado o método dos volumes finitos em uma malha staggered,
resultando em um sistema algébrico de equações não lineares, que é
resolvido de maneira acoplada, utilizando o método de Newton, em cada passo de tempo.

$$
\frac{\partial u}{\partial x} + 
\frac{\partial v}{\partial y} =
0
$$

$$
\frac{\partial (\rho u)}{\partial t} +
\frac{\partial (\rho u u)}{\partial x} + 
\frac{\partial (\rho v u)}{\partial y} =
\frac{\partial}{\partial x} \left( \mu \frac{\partial u}{\partial x} \right) +
\frac{\partial}{\partial y} \left( \mu \frac{\partial u}{\partial y} \right) -
\frac{\partial P}{\partial x}
$$

$$
\frac{\partial (\rho v)}{\partial t} +
\frac{\partial (\rho u v)}{\partial x} + 
\frac{\partial (\rho v v)}{\partial y} =
\frac{\partial}{\partial x} \left( \mu \frac{\partial v}{\partial x} \right) +
\frac{\partial}{\partial y} \left( \mu \frac{\partial v}{\partial y} \right) -
\frac{\partial P}{\partial y}
$$

A implementação, conforme ilustrado na imagem abaixo, está dividida em duas partes principais: O input dos
dados passa por um procedimento de geração de um grafo de propriedades (representação da malha a ser
simulada). A seguir, essas informações são passadas para o solver, que é dividido entre Time Stepper (que
cuida do transiente), Newton Solver, e a função resíduo (dados vetores $u$, $v$ e $P$, retorna um "erro"). O Newton
Solver engloba tanto a aproximação da jacobiana quanto a solução do sistema linear própriamente dito.

{:refdef: style="text-align: center;"}
![Visão geral do sistema](/images/2020-03-30-python-profiling/img003.png)
{: refdef}


# Python profiling com cProfile

Não é incomum vermos a análise por "tempo de parede" (wall clock time), ou seja, se mandassemos
executar o programa e ficassemos olhando o relógio em nossa parede, quanto tempo teria passado até que
a execução terminasse? Esse tipo de análise é relevante, por exemplo, para comparação de resultados "antes e
depois". Porém, pouco nos dizem sobre onde podemos melhorar. Uma alternativa seria a de instrumentar todo o
código, guardando os tempos de execução parciais (Já viram alguém colocando vários "time()" pelo código?).
Isso não só é tedioso como também é desnecessário, pois ferramentas de profiling fazem algo semelhante
com muito menos esforço.

Em Python, uma possível ferramenta é o cProfile. Ela é relativamente simples, e sua 
[documentação é bastante completa](https://docs.python.org/2/library/profile.html).
Por isso, deixo abaixo um exemplo de uso muito simples, pela linha de comando, apenas por completude, sem
maiores aprofundamentos sobre as opções que a ferramenta disponibiliza. No exemplo, é possível observar
que o tempo de execução total foi de 1 segundo, distribuido por funções no módulo example.py e outras
builtin. O nome das funções aparece entre parenteses, ao lado do nome de cada módulo python. Por padrão,
o resultado é ordenado por nome (não por tempo de execução).

{% highlight python linenos %}
def g(x):
    import time
    time.sleep(1)
    return x + 1

def f(x):
    return 2 * g(x)

print(f(10))
{% endhighlight %}

{% highlight python linenos %}
python -m cProfile example.py
{% endhighlight %}

{% highlight python linenos %}
         7 function calls in 1.000 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    1.000    1.000 example.py:1(<module>)
        1    0.000    0.000    0.999    0.999 example.py:1(g)
        1    0.000    0.000    0.999    0.999 example.py:6(f)
        1    0.000    0.000    1.000    1.000 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {built-in method builtins.print}
        1    0.999    0.999    0.999    0.999 {built-in method time.sleep}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
{% endhighlight %}

cProfile não é a única alternativa, e provavelmente não é a melhor. O que ela faz é basicamente inserir em
cada chamada de função um "observador", para produzir um relatório no final da execução contendo informações
de tempo de execução. Note que
isso significa que ela adiciona um overhead distribuido (e pior: não homogeneo) no sistema, que não existia
no software original.

Mesmo com essas considerações, o cProfile acaba sendo uma das ferramentas que eu mais uso, pelo menos nas etapas
iniciais da análise, e principalmente quando eu não tenho muito domínio do software que estou
analisando. No mínimo, não é raro ela me prover as "palavras-chave" pra conversar com os desenvolvedores,
o que me permite iniciar uma discussão de aprofundamento.


# Discussão, experimento e análise

O cuidado com o experimento é tão importante quanto (ou mais que) saber utilizar a ferramenta.
Para ilustrar essa questão, considere dois casos distintos:

- Malha grossa (ou pouco refinada) + tempo final grande (passo de tempo pequeno)
- Malha fina (ou refinada) + tempo final pequeno (passo de tempo grande)

Será que os dois experimentos
vão acusar os mesmos problemas de performance? Qual é mais relevante? Talvez uma malha fina com um
tempo de simulação também grande? Nem sempre. Se estivessemos no contexto em que o solver faz parte da
função objetivo de um processo de otimização, talvez uma malha mais grosseira fosse suficiente, por exemplo.

Claro que sempre vai ter quem argumente "Mas o melhor seria uma malha fina e um tempo de simulação grande
também, pois aí eu cubro todos os casos". Um possível argumento contra isso é que você deveria
observar as limitações e restrições de projeto, para tentar ser mais efetivo com o menor esforço possível.
Eu vejo isso como aquela velha história: Se você atirar com um canhão, provavelmente vai matar a mosca,
mas vai te custar muito mais.

Discussões filosóficas à parte, aqui vamos avaliar um caso com uma malha grossa, mas um tempo de simulação
suficientemente grande para atingir regime permanente. Mostro abaixo as primeiras 5 linhas de duas rodadas
do cProfile no meu código de exemplo. Na primeira, as chamadas estão ordenadas pelo cumtime
(Tempo acumulado na função + todas as subfunções chamadas a partir da mesma). Já a segunda está ordenada
pelo totime (Tempo total despendido em uma função, sem considerar a chamada das subfunções a partir dela).

{% highlight python linenos %}
         2882855 function calls (2872415 primitive calls) in 16.535 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
    830/1    0.016    0.000   16.537   16.537 {built-in method builtins.exec}
        1    0.000    0.000   16.536   16.536 run_solver.py:1(<module>)
        1    0.007    0.007   15.463   15.463 time_stepper.py:8(run_simulation)
       13    0.000    0.000   15.455    1.189 scipy_solver_wrapper.py:17(solve)
       13    0.000    0.000   15.450    1.188 minpack.py:48(fsolve)
{% endhighlight %}

Pela característica da implementação ser basicamente "linear" (run_solver chama time_stepper, que chama
o solver, que chama a minpack, ...), para esse caso específico, o primeiro resultado não nos diz muito,
se considerarmos apenas os cinco maiores "gargalos" ordenados pelo tempo acumulado. Olhando por essa
métrica, talvez possíveis "melhorias" seriam tentar engrossar ainda mais a malha, pra que cada solução do sistema
linear fosse mais rápido, ou aumentar o passo de tempo, para que seja consumido menos tempo no time stepper.

{% highlight python linenos %}
         2883005 function calls (2872565 primitive calls) in 17.222 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
       13    7.683    0.591   16.184    1.245 {built-in method scipy.optimize._minpack._hybrd}
    12293    7.034    0.001    8.510    0.001 numpy_residual_function.py:4(residual_function)
   319659    0.404    0.000    0.404    0.000 {built-in method numpy.zeros}
135835/135655    0.308    0.000    0.366    0.000 {built-in method numpy.core._multiarray_umath.implement_array_function}
     4863    0.288    0.000    0.288    0.000 {built-in method nt.stat}
730700/730259    0.244    0.000    0.370    0.000 {built-in method builtins.len}
   700753    0.126    0.000    0.126    0.000 staggered_grid.py:18(__len__)
{% endhighlight %}

Para o segundo conjunto de resultados, é possível notar que um dos maiores gargalos do sistema em relação a tempo total
dispendido é a função resíduo. É possível observar que ela é custosa por dois motivos: (1) cada
chamada dela é custosa por si só e (2) ela é chamada muitas (ncalls=12293) vezes. Com essa informação,
algumas alternativas para diminuir o tempo de execução são possíveis. Por exemplo, uma alternativa seria
de trabalhar para diminuir o tempo de execução da função resíduo, por exemplo, passando apenas ela para C++.

Outras sugestões poderiam ser apresentadas, e nem todas as alternativas envolvem mudança de código. Cabe agora
ao analista verificar qual abordagem seria interessante seguir, dada a facilidade de implementação, restrições
de projeto, perspectiva de ganho, conhecimentos prévios, etc.

Para finalizar, alguns colegas de trabalho comentaram alternativas ao cProfile, caso o leitor tenha interesse:
[py-spy](https://github.com/benfred/py-spy), [pyflame](https://pyflame.readthedocs.io/en/latest/),
[pyvmmonitor](https://www.pyvmmonitor.com/) e [pyinstrument](https://github.com/joerick/pyinstrument/).
Um detalhe importante de comentar é que, embora o objetivo dessas ferramentas seja o mesmo,
a forma como fazem isso é diferente e, portanto, os resultados podem ser diferentes. Por exemplo, o pyinstrument é
uma ferramenta de profiling por sampling (adquire dados de tempos em tempos), diferente do cProfile que
é deterministico (adquire dados para cada chamada).

Até a próxima!
