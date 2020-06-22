---
layout: post
title: Implementação de um simulador para as equações de Navier Stokes em Python - Parte 1
image: 2020-06-22/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-06-22/img001.png)
{: refdef}

Na semana passada, comecei essa série sobre a implementação do solver para as
equações de Navier Stokes em Python, onde mostrei brevemente a derivação matemática
do que seria implementado. Essa semana continuo, dando os highlights sobre as
partes relevantes relacionadas à implementação.


# Arquitetura do software

A visão geral do sistema pode ser observado na ilustração abaixo. O ponto de
entrada, após o setup, é o *TimeStepper*, responsável por orquestrar os passos
de tempo. A determinação dos campos de pressão e velocidades de cada tempo
depende da solução de um sistema não linear. O sistema não linear é resolvido
usando a biblioteca *PETSc*. Após a conclusão de todos os passos de tempo,
uma etapa de pós-processamento cuida dos plots, geração de relatórios, etc.

{:refdef: style="text-align: center;"}
![](/images/2020-06-22/img002.png)
{: refdef}


# Solução do sistema não linear

A parte mais relevante no sistema é a solução do sistema não linear. Para 
utilizar a biblioteca PETSc, as equações discretizadas foram colocadas em
forma residual $R(X) = 0$. Dessa forma, a função resíduo pode ser implementada
em qualquer linguagem de programação, desde que siga a interface *f(x, graph)*,
onde *x* é o vetor contendo as pressões e velocidades de cada volume de controle,
e *graph* é a estrutura de dados com informações de cada volume de controle (a malha).

Como de costume aqui no blog, a primeira versão da implementação foi feita puramente em
Python (com auxilio da biblioteca Numpy). O código completo é demasiado grande,
por isso, mostro apenas parte da equação da quantidade de movimento, após
conversão dos indices do vetor *x*, aplicação das condições de contorno e
preparo de variáveis "secundárias", como por exemplo as derivadas.

{% highlight python linenos %}
# Terms calculation for Navier Stokes X
transient_term = (rho * U_P - rho * U_P_old) * (dx * dy / dt)
advective_term = \
    rho * U_e * ((.5 - beta_U_e) * U_E + (.5 + beta_U_e) * U_P) * dy - \
    rho * U_w * ((.5 - beta_U_w) * U_P + (.5 + beta_U_w) * U_W) * dy + \
    rho * V_n * ((.5 - beta_V_n) * U_N + (.5 + beta_V_n) * U_P) * dx - \
    rho * V_s * ((.5 - beta_V_s) * U_P + (.5 + beta_V_s) * U_S) * dx
difusive_term = \
    mi * dU_e_dx * dy - \
    mi * dU_w_dx * dy + \
    mi * dU_n_dx * dx - \
    mi * dU_s_dx * dx
source_term = -(P_e - P_w) * dy

# Residual equation for Navier Stokes X
ii = 3 * (i + j) + 1
residual[ii] = transient_term + advective_term - difusive_term - source_term
{% endhighlight %}

A separação dos termos ajuda na compreensão da implementação,
e a numenclatura não é arbitrária: tenta-se sempre fazer um mapeamento em
relação às equações originais. Muita gente "não gosta" de usar letras
maiusculas no meio do nome de variáveis, a não ser no caso de *CamelCase*.
Porém, aqui, a numenclatura matemática (misturada com a sintaxe de Latex)
leva um mapeamento muito mais natural entre código e equações. Defendo
fortemente essa propriedade, pois facilita na hora de revisitar o código,
meses depois de sua implementação (com o detalhe que é muito difícil manter
código e implementação sincronizados... Mas a ideia é chegar o mais próximo
disso na medida do possível).

Aqui, é importante notar que a matriz Jacobiana (ou matriz tangente) não será
computada analiticamente. Ao invés disso, a PETSc possui uma implementação
genérica para aproximar a mesma por meio de diferenças finitas. Como sabe-se
que a matriz é esparsa a prióri, é possível prover uma estrutura de não-zeros,
a fim de acelerar o processo de computação dessa aproximação. A PETSc usa uma
técnica conhecida como matrix coloring para isso. A ideia geral é que, sendo
conhecida a estrutura de não-zeros da matriz, é possível computar algumas
derivadas com um incremento em mais de uma componente no vetor *x*, o que
diminui o número total de chamadas à função resíduo e, portanto, diminui o
tempo de execução.

A determinação da estrutura de não zeros foi feita puramente
em Python (com numpy), e é mostrada abaixo. Essa função é chamada apenas uma
vez, no início do algoritmo. Aqui, chamo a atenção para o uso interessante
das funções auxiliares *diags* e *kron*, na Scipy. A função *diags* cria uma
matriz diagonal, dado um tamanho e as posições onde espera-se valores. O
mais interessante, é que é a matriz resultante já é esparsa, o que é importante,
conforme o problema aumenta. A função *kron*, aqui, serve para expandir a matriz
anterior em uma maior, considerando todas as variáveis do problema. Dessa forma,
em apenas duas chamadas de função a estrutura de não zeros esparsa está pronta
para ser enviada para a PETSc.

{% highlight python linenos %}
def _calculate_jacobian_mask(nx, ny, dof):
    from scipy.sparse import diags, kron

    N = nx * ny

    j_structure = diags(
        np.ones(shape=(7,)),
        [-nx + 1, -nx, -1, 0, 1, +nx, +nx - 1],
        shape=(N, N),
        format='csr',
    )

    j_structure = kron(
        j_structure,
        np.ones(shape=(dof, dof)),
        format='csr',
    )

    return j_structure
{% endhighlight %}

Dado essas funções implementadas, é feita a configuração
do solver não linear pela PETSc, passando as informações provenientes nas
funções anteriores. As configurações utilizadas são mostradas no código abaixo.
O solver linear é o GMRes, e o não linear é método de Newton com Line-Search (LS).
Todo o processo é orquestrado pelo *TimeStepper*, e o critério de parada foi
dado por um "tempo final de simulação", definido pelo usuário. Para saber mais
sobre a PETSc, dê uma olhada no [site official](https://www.mcs.anl.gov/petsc/).

{% highlight python linenos %}
options.setValue('mat_fd_type', 'ds')
options.setValue('pc_type', 'ilu')
options.setValue('pc_factor_shift_type', 'NONZERO')
options.setValue('ksp_type', 'gmres')
options.setValue('snes_type', 'newtonls')
options.setValue('snes_linesearch_type', 'basic')
# (...)

# Creates the Jacobian matrix structure.
j_structure = _calculate_jacobian_mask(pressure_mesh.nx, pressure_mesh.ny, 3)
csr = (j_structure.indptr, j_structure.indices, j_structure.data)
self._petsc_jacobian = PETSc.Mat().createAIJWithArrays(j_structure.shape, csr)
self._petsc_jacobian.assemble(assembly=self._petsc_jacobian.AssemblyType.FINAL_ASSEMBLY)
self._comm = PETSc.COMM_WORLD
self._dm = PETSc.DMShell().create(comm=self._comm)
self._dm.setMatrix(self._petsc_jacobian)
# (...)

# Residual vector setup
self._r = PETSc.Vec().createSeq(residual_size)
self._snes = PETSc.SNES().create(comm=self._comm)
self._snes.setFunction(self.residual_function_for_petsc, self._r)
self._snes.setFromOptions()
self._snes.setUseFD(True)
self._snes.setTolerances(rtol=1e-4, atol=1e-4, stol=1e-4, max_it=50)
# (...)
{% endhighlight %}


# Resultados e discussão

{:refdef: style="text-align: center;"}
![](/images/2020-06-22/img003.png)
{: refdef}

O caso de teste foi o clássico problema da cavidade (*lid driven cavity problem*),
vastamente utilizado como benchmark na literatura. Os tempos de execução para
uma variação no tamanho da malha é ilustrado acima. A implementação atual é puramente
em python, salvo o uso de bibliotecas externas, que são implementadas em C, C++
ou FORTRAN (Scipy, Numpy, PETSc). Não foi feita comparação com nenhuma referência,
para saber o quão lento são esses valores, porém, são visivelmente inadimissíveis,
dadas malhas tão minusculas.

Como em posts anteriores, o objetivo
aqui é ter uma implementação inicial para comparar com 
algumas funções traduzidas para C++. Fazendo uma análise de performance, notou-se
que o grande gargalo desse programa era a função resíduo, que, além de lenta, é
chamada muitas vezes, por conta do solver não linear. Na segunda parte dessa
breve série (próximo post), comparo os resultados aqui obtidos com os referentes
à implementação parcialmente em C++.

Até a próxima!
