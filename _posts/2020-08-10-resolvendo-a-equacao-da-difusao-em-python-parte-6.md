---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 6
image: 2020-08-10/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-08-10/img001.png)
{: refdef}

Lá em meados de fevereiro até abril, eu fiz uma série de posts sobre a
implementação da solução da equação de difusão usando diferenças finitas,
usando Python. Na época, dividi os posts em
[parte 1](https://tarcisiofischer.github.io/2019-02-10/resolvendo-a-equacao-da-difusao-em-python-parte-1),
[parte 2](https://tarcisiofischer.github.io/2019-02-13/resolvendo-a-equacao-da-difusao-em-python-parte-2),
[parte 3](https://tarcisiofischer.github.io/2020-02-24/resolvendo-a-equacao-da-difusao-em-python-parte-3),
[parte 4](https://tarcisiofischer.github.io/2020-03-02/resolvendo-a-equacao-da-difusao-em-python-parte-4),
[parte 5](https://tarcisiofischer.github.io/2020-03-09/resolvendo-a-equacao-da-difusao-em-python-parte-5)
e, no último post, ao invés de escrever "parte 6", escrevi
"[parte 7](https://tarcisiofischer.github.io/2020-04-13/resolvendo-a-equacao-da-difusao-em-python-parte-7)".
Resolvi esperar e criar uma parte 6 quando surgisse alguma ideia legal. E esse momento
finalmente chegou :) Aproveito pra lembrar que eu tambem
[fiz um breve vídeo sobre o assunto, no Youtube](https://www.youtube.com/watch?v=BFM0HRDjfnw).

A ideia foi fazer uma pequena mudança no modelo de
forma a aceitar um termo fonte e um coeficiente de difusão que muda
de acordo com a posição. Matematicamente, a equação torna-se a seguinte:

$$
\dot{f} = c(X) \nabla^2 f + s(X)
$$

O objetivo disso é resolver um caso onde a propriedade gerada pelo termo fonte
difunde por um meio pré-determinado. Minha ideia foi baseada naqueles processos
de injeção de plástico, onde a partir
de alguns pontos de injeção, o material escoa pra dentro de um molde. Claro
que essa foi apenas a inspiração: Esse caso **nada tem a ver** com uma simulação
real de injeção de plástico.

As mudanças de implementação em relação ao código daquela série foram mínimas.
Apenas tive que carregar os dados
para o coeficiente de difusão $c$, e o termo fonte, $s$, e atualizar a matriz
de coeficientes do sistema linear e o vetor b. A parte relevante é mostrada
abaixo, mas [o código completo está no GitHub](https://gist.github.com/tarcisiofischer/66f5839558eda8462cdaee81fbd8f7c6).

{% highlight python linenos %}
for i in range(n_rows):
    for j in range(n_cols):
        add_connectivity((i, j), (i, j), ds**2 + 4 * c[i, j] * dt)

        if j + 1 < n_cols:
            add_connectivity((i, j), (i, j + 1), -c[i, j] * dt)
        if j - 1 > 0:
            add_connectivity((i, j), (i, j - 1), -c[i, j] * dt)
        if i + 1 < n_rows:
            add_connectivity((i, j), (i + 1, j), -c[i, j] * dt)
        if i - 1 > 0:
            add_connectivity((i, j), (i - 1, j), -c[i, j] * dt)
{% endhighlight %}


{% highlight python linenos %}
b = ds ** 2 * previous_state.flatten() + sources.flatten()
result = pypardiso.spsolve(A.tocsr(), b).reshape(initial_conditions.shape)
{% endhighlight %}

As posições dos termos fonte são lidos de um arquivo de imagem, nesse caso,
foi utilizada a imagem abaixo:

{:refdef: style="text-align: center;"}
![](/images/2020-08-10/s.png)
{: refdef}

E a distribuição do coeficiente de difusão é lida da imagem a seguir.

{:refdef: style="text-align: center;"}
![](/images/2020-08-10/c.png)
{: refdef}

O resultado final é mostrado no gif abaixo, e é uma pequena homenagem:

{:refdef: style="text-align: center;"}
![](/images/2020-08-10/feliz_dia_dos_pais.gif)
{: refdef}

Feliz dia dos pais!
