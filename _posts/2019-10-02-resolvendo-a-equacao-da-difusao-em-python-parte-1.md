---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 1
---

Muita gente com conhecimento em outras linguagens de programação me pedem ajuda pra iniciar com Python. Em geral são pessoas com experiência em outras linguagens de programação como C, C++, Matlab ou Fortran (Sim… Fortran…). Estou planejando alguns materiais nesse sentido pra ajudar o pessoal, e essa série de posts vai ser um desses.

A ideia aqui é resolver uma equação simples, apenas como incentivo pra mostrar algumas funcionalidades de Python e a “stack científica“. Então, nessa série de posts vou abordar um pouco sobre vários temas que eu ache relevante e/ou que me perguntam ou ja me perguntaram. Por isso, caso você, leitor, tenha alguma dúvida fique livre pra comentar ou me procurar pelas redes sociais que, dependendo do tema, eu posso gerar conteúdo direcionado. Sinta-se livre também pra me apontar erros que, afinal, somos todos humanos.

# O problema

A equação que vamos resolver é

$$
\dot{f} = c \nabla^2 f + s
$$

E o resultado final pro nosso programa (No final da série de posts) será algo parecido com isso, por exemplo:

{:refdef: style="text-align: center;"}
![Exemplo - Resultado final esperado](/images/example001.gif)
{: refdef}

Se você não tem familiaridade com esse tipo de equação, não se desespere – Alias, pode até pular toda a parte matemática, que está aqui apenas para formalizar o problema antes de sair programando. De qualquer forma, daqui a pouco vamos nos desvincular da parte matemática e entrar na programação…

Se você, por outro lado, tem familiaridade, por favor, faça vista grossa das simplificações que eu vou fazer aqui – por exemplo, na discretização do domínio e na aplicação das condições de contorno :)

O problema é, então, que queremos encontrar uma função do tempo e do espaço,
ou seja, $f(X, t)$ que satisfaça a equação diferencial acima, dado um termo fonte $s(X, t)$. 
Utilizaremos um domínio bidimensional, admitindo um espaçamento igual em $x$ e em $y$ (que chamarei de $\Delta s$).
Dessa forma, temos que $X = X(x, y)$, e utilizaremos a discretização em diferenças finitas, de onde obtemos:

$$
\frac{f^{x,y} - f_o^{x,y}}{\Delta t} = c \Bigg{[}
    \frac{f^{x+1,y} - 2 f^{x,y} + f^{x-1,y}}{\Delta s^2} +
    \frac{f^{x,y+1} - 2 f^{x,y} + f^{x,y+1}}{\Delta s^2}
\Bigg{]} + s^{x,y}
$$

Isolando os termos, ficamos com um sistema de equações lineares – Uma equação para cada $f^{x,y}$:

$$
[\Delta s^2 + 4 c \Delta t]f^{x,y}
- c \Delta t f^{x+1, y}
- c \Delta t f^{x-1, y}
- c \Delta t f^{x, y+1}
- c \Delta t f^{x, y-1}
=
\Delta s^2 f_o^{x, y} + \Delta s^2 \Delta t s^{x, y}
$$

Phew… Estamos quase lá. Basicamente, para descobrir o valor de cada $f^{x,y}$, dependemos dos valores adjacentes, como no exemplo da imagem abaixo. O quadradinho em amarelo depende dos valores quadradinhos em verde (Ou azul?):

{:refdef: style="text-align: center;"}
![Exemplo - dependências](/images/example002.gif)
{: refdef}

Mas os quadradinhos em verde também possuem valores desconhecidos.
Isso acaba gerando uma “cadeia” de equações (Uma pra cada quadradinho).
Ou seja, um sistema linear ($Ax = b$) com $NxN$ equações, onde $N$ é a dimensão
do domínio inteiro, A é a matriz de coeficientes, x são as incógnitas ($f^{x,y}$
para cada x e y no domínio) e b é um vetor conhecido, que depende das
informações relacionadas ao tempo anterior e ao termo fonte.

# Abordagem de solução

{% highlight python linenos %}
import numpy as np

N = 20

fo = np.zeros(shape=(N, N,))
for i in range(N):
    for j in range(N):
        if i >= 5 and i < 15 and j >= 5 and j < 15:
            fo[i, j] = 10.0
{% endhighlight %}

