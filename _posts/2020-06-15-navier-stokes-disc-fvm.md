---
layout: post
title: Discretização das equações de Navier Stokes pelo Método dos Volumes Finitos
image: 2020-06-15/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-06-15/img001.png)
{: refdef}

O método dos volumes finitos (MVF) é vastamente utilizado na solução de problemas
envolvendo dinâmica dos fluidos. Nas próximas semanas, mostrarei alguns
highlights de implementação em Python. Seguindo a sugestão de alguns amigos
e colegas, serei mais breve nesse primeiro post, sobre a derivação matemática,
afinal, são passos clássicos que se encontra em livros e artigos fácilmente.


# Breve derivação matemática

Para um domínio bidimensional, considera-se a equação da conservação da massa
e quantidade de movimento nos eixos $x$ e $y$. Considerando
o fluido incompressível, e fazendo as devidas simplificações no problema,
podemos chegar nas equações abaixo. Dessa forma, três incógnitas
são levadas em consideração: Pressão ($P$), velocidade $x$ ($u$) e velocidade
$y$ ($v$).

$$
\frac{\partial \rho u}{\partial x}
+
\frac{\partial \rho v}{\partial y}
=
0
$$

$$
\frac{\partial \rho u}{\partial t}
+
\frac{\partial \rho u u}{\partial x}
+
\frac{\partial \rho v u}{\partial y}
=
\frac{\partial}{\partial x} \Bigg{(} \mu \frac{\partial u}{\partial x} \Bigg{)} +
\frac{\partial}{\partial y} \Bigg{(} \mu \frac{\partial y}{\partial x} \Bigg{)} -
\frac{\partial P}{\partial x}
$$

$$
\frac{\partial \rho u}{\partial t}
+
\frac{\partial \rho u v}{\partial x}
+
\frac{\partial \rho v v}{\partial y}
=
\frac{\partial}{\partial x} \Bigg{(} \mu \frac{\partial v}{\partial x} \Bigg{)} +
\frac{\partial}{\partial y} \Bigg{(} \mu \frac{\partial v}{\partial y} \Bigg{)} -
\frac{\partial P}{\partial y}
$$

Após o processo clássico de discretização em volumes finitos, surgem as equações
discretizadas, mostradas abaixo (equação de Navier-Stokes em Y não é mostrada
pois é analoga à de X). A malha
considerada é cartesiana quadrilátera com o mesmo número de pontos em $x$ e
em $y$. É utilizada uma malha com arranjo desencontrado (malha staggered),
na distribuição das incógnitas. A imagem abaixo ilustra essa distribuição,
onde os centros dos volumes de controle de pressão e das velocidades não se
encontram na mesma posição.

{:refdef: style="text-align: center;"}
![](/images/2020-06-15/img002.png)
{: refdef}

Para os termos advectivos, decidi por usar
a interpolação UDS (Upstream Differencing Scheme), utilizando $\beta = \pm 0.5$,
na formulação abaixo. Se $\beta = 0$, teriamos a formulação CDS. Os subescritos $P$ se referem
às incógnitas do volume de controle corrente, $e$ (east) o volume à direita,
$w$ (west) o volume à esquerda, $s$ (south) e $n$ north baixo e cima,
respectivamente, onde as letras maiúsculas denotam valores nos centros dos
volumes de controle, enquanto as minúsculas nas faces.

$$
u_e \Delta y  - u_w \Delta y
+
v_n \Delta x  - v_s \Delta x
=
0
$$

$$
(\rho u_P - \rho^o u_P^o) \frac{\Delta x \Delta y}{\Delta t}
+ \\
\rho u_e \Bigg{[} (0.5 - \beta_{u_e})u_E + (0.5 - \beta_{u_e})u_P \Bigg{]} \Delta y - \\
\rho u_w \Bigg{[} (0.5 - \beta_{u_w})u_P + (0.5 - \beta_{u_w})u_W \Bigg{]} \Delta y + \\
\rho v_n \Bigg{[} (0.5 - \beta_{v_n})u_N + (0.5 - \beta_{v_n})u_P \Bigg{]} \Delta x - \\
\rho v_s \Bigg{[} (0.5 - \beta_{v_s})u_P + (0.5 - \beta_{v_s})u_S \Bigg{]} \Delta x = \\
-(P_e - P_w) \Delta y + \\
\mu \frac{u_E - u_P}{\Delta x} \Delta y - \mu \frac{u_P - u_W}{\Delta x} \Delta y \\
\mu \frac{u_N - u_P}{\Delta y} \Delta x - \mu \frac{u_P - u_S}{\Delta y} \Delta x
$$

# Solução do sistema

Para resolver as não linearidades, o método de Newton multivariado foi
utilizado. Não estou utilizando os métodos clássicos, como SIMPLE ou PISO, por
exemplo. Vejo poucas pessoas utilizarem Newton para resolver
as equações de Navier Stokes. Um dos motivos é que, entre as iterações, pode-se produzir
campos de pressão e velocidade que talvez não conservem massa ou quantidade de movimento.
A principal motivação para utilizar o método de Newton aqui é a sua facilidade
de implementação, visto a disponibilidade de solvers não lineares.

As equações foram colocadas em sua forma residual
($R(x) = 0)$ e, em um processo iterativo, as velocidades e pressões são
aproximadas resolvendo o sistema linear $J\Delta x = F(x)$, onde $J$ é a matriz
jacobiana (Também conhecida como matriz tangente), enquanto $\Delta x$ é o
incremento das incógnitas u, v e P que, juntas, compõem o vetor de solução x.

Como o problema é transiente, um gerenciador de passos de tempo (time stepper)
foi implementado. Seu propósito é de preparar o sistema não linear, resolve-lo e
avançar o passo de tempo até que se atinja algum critério de parada (Regime
permanente, por exemplo).

Os sistema lineares gerados na solução do problema não linear são esparsos. A estrutura
de não zeros é mostrada na imagem abaixo. A malha utilizada para gerar a imagem
tinha $10 \times 10$ elementos. Como existem 3 graus de liberdade, o tamanho
total da matriz, considerando os zeros, é de $(10 \cdot 10 \cdot 3)^2$ elementos.
A parcela em roxo representa os zeros, enquanto a parcela em amarelo representa
os não-zeros. Para trabalhar com matrizes esparsas muito grandes, estruturas de
dados específicas devem ser utilizadas.

{:refdef: style="text-align: center;"}
![](/images/2020-06-15/img003.png)
{: refdef}

Nas próximas semanas, abordarei os aspectos de implementação da solução do
problema, utilizando abordagens semelhantes àquelas mostradas em posts anteriores.
Esse projeto também estará disponível no GitHub, pra quem quiser baixar e
fazer experimentos. Espero que sirva de exemplo para quem está começando! :)

Agradeço ao Arthur Soprana pelas sugestões, para esse post. Até a próxima!
