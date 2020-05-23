---
layout: post
title: Montagem e solução do sistema linear para a equação de Helmholtz
---

Nesse post, descrevo as etapas de montagem e a solução do sistema linear
gerado a partir da discretização da equação de Helmholtz em Elementos Finitos.
Pra quem não acompanhou, essa é a continuação de outros posts, onde eu fiz
a explicação matemática do que seria implementado e fiz a geração da malha
bidimensional. Lembrando que você pode acompanhar o código completo
[pelo Github](https://github.com/tarcisiofischer/helmholtz-solver).

{:refdef: style="text-align: center;"}
![](/images/2020-05-25/img001.png)
{: refdef}


# Juntando as peças

Muita coisa já foi implementada em outros posts. Como explicado
no [post sobre a solução de sistemas lineares com componentes complexas]({% post_url 2020-04-20-sistema-linear-complexo %}),
é possível fazer uma transformação em um sistema linear para que todas as
suas componentes sejam reais. Aproveitei essa solução pra implementar o
módulo 'complex_linear_system', que implementa tanto a transformação para
números reais quanto a volta do resultado para os complexos.

Para o solver linear, resolvi utilizar a [PyPARDISO](https://github.com/haasad/PyPardisoProject).
A interface dela pro uso básico é simples, e é mostrado abaixo. O resultado
final em termos de tempo de execução foram satisfatórios para um primeiro momento,
embora eu não os mostre nesse momento.

{% highlight python linenos %}
def _solve_linear_system(A, b):
    return pypardiso.spsolve(A.tocsc(), b)
{% endhighlight %}

Resta implementar a montagem da matriz de rigidez e vetor global. A
implementação segue a "receita de bolo" descrita em qualquer livro de elementos
finitos (E descrito no primeiro post dessa série). A matriz de rigidez global
é separada em matrizes elementares (que são densas). A computação de cada
matriz elementar é computada de forma independente das outras (Oportunidade
obvia de paralelismo). Para fazer o mapeamento das matrizes elementares para
a matriz global, usei a conectividade disponível na malha. A implementação
completa está disponível no GitHub.

Por fim, usei aquela técnica de inversão de dependência pra mover o pós
processamento para callbacks. Dessa forma, o plot dos resultados fica por
responsabilidade do módulo plotting, que possui uma função auxiliar
"get_callbacks" apenas para facilitar esse processo. O resultado final mara
uma malha de $80 \times 80$ e apenas 1 termo fonte é mostrado na figura abaixo.
Existe uma camada em volta do domínio que faz um amortecimento da onda, enquanto
dentro do domínio existem duas regiões distintas de propriedades do meio.

{:refdef: style="text-align: center;"}
![](/images/2020-05-25/img002.png)
{: refdef}


# Análise de performance

A implementação, atualmente, está totalmente em Python, com uso da Numpy para
acelerar algumas partes e da PyPardiso para solucionar o sistema
linear. A figura abaixo mostra os tempos de execução para malhas de
$N \times N$ com $N=40$ até $N=80$, com passo 10. A priori, não temos uma
implementação para comparar. Mas podemos fazer uma auto-análise para saber se
conseguimos melhorar a performance.

{:refdef: style="text-align: center;"}
![](/images/2020-05-25/img003.png)
{: refdef}

Seguirei a abordagem mostrada em um post anterior desse blog, sobre análise de
performance. A ferramenta
cProfile da uma ideia do que pode ser trabalhado para melhorar performance.
Os resultados são mostrados abaixo. A sugestão, a partir daqui, será seguir
movendo os gargalos computacionais para C++, com bindings para Python utilizando
a Pybind11, como já foi feito em posts passados.

{% highlight python linenos %}
     2600681 function calls (2585757 primitive calls) in 11.122 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    2.534    2.534    2.534    2.534 complex_linear_system.py:5(_build_extended_A)
     6400    2.483    0.000    4.285    0.001 solver.py:28(_build_local_Ke)
     6400    0.952    0.000    1.968    0.000 solver.py:103(_build_local_f)
   173576    0.875    0.000    0.875    0.000 {built-in method numpy.array}
{% endhighlight %}


# Discussão e conclusão

Apesar de já termos alguns resultados preliminares, ainda há muito o que fazer
nesse projeto. Melhorias incluem o input para melhorar o setup dos casos,
efetivamente melhorar a performance, movendo partes do código para C++, criar
mais opções por linha de comando ou GUI, adicionar testes e documentações,
tirar proveito de paralelismo, etc.

Python pode não produzir códigos eficientes, em termos de tempo. Porém, nos
ajuda a ter algum resultado rápido, pois seu ciclo de desenvolvimento permite
isso: Não existe etapa de compilação nem linkagem, a instalação de bibliotecas
é fácil, com as ferramentas certas, e as IDEs disponíveis gratuitamente são
suficientemente poderosas. Por isso, eu ainda sou defensor de utilizar Python,
principalmente nas etapas mais iniciais do projeto. Além disso, eu nem estou
levando em considerações poderosas bibliotecas como FEniCS, fiPy, devito, etc,
que podem deixar esse processo de desenvolvimento ainda mais rápido.

Sugestões e críticas são bem vindas para esse projeto. Se quiser baixar para
testar/modificar, fique a vontade e, caso tenha interesse, me chama pra uma
conversa pra gente discutir sobre isso :)

Até a próxima!
