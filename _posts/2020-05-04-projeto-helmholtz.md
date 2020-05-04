---
layout: post
title: Da equação ao código - Projeto de implementação da solução para a equação de Helmholtz
---

Da mesma forma que a série de posts anterior, o solver para a equação de
Helmholtz em elementos finitos será dividido em partes.
Para acompanhar o desenvolvimento nas próximas semanas, vou disponibilizar todo
o [código fonte no github](https://github.com/tarcisiofischer/helmholtz-solver),
o que significa que você pode baixar, rodar, testar e modificar o código à
vontade, na sua máquina.

Em posts passados, já explorei um pouco o uso de
bibliotecas para a solução de sistemas lineares 
([veja aqui](https://tarcisiofischer.github.io/2020-02-24/resolvendo-a-equacao-da-difusao-em-python-parte-3)),
e já comentei sobre pós processamento com a Matplotlib
([veja aqui](https://tarcisiofischer.github.io/2020-03-02/resolvendo-a-equacao-da-difusao-em-python-parte-4))
e com o Paraview
([veja aqui](https://tarcisiofischer.github.io/2020-03-23/visualizacao-de-resultados-com-paraview)).
Além disso, já temos um post sobre paralelização da montagem da matriz de rigidez
([veja aqui](https://tarcisiofischer.github.io/2020-03-16/paralelizacao-da-matriz-de-rigidez)),
embora talvez eu revisite esse tema. Dessa forma, esse post vai subir um pouco
o nível de abstração e começar a falar de módulos, pacotes, organização e
reaproveitamento de código.

{:refdef: style="text-align: center;"}
![](/images/2020-05-04/img001.png)
{: refdef}


# Modularização e organização de código

Modularização, em linhas gerais, trata da separação do software em diferentes
partes. Ela pode ser feita em diversas granularidades: Funções separam porções
individuais de rotinas sem estado, classes separam conjuntos de métodos (funções
relacionadas a um objeto) e atributos de porções dependentes de estado, módulos
separam funções e classes, pacotes separam vários módulos,
bibliotecas separam um conjunto de módulos e pacotes. A palavra "estado" aqui
é usada em relação à modificação dos membros de um objeto.

Essa separação e nomenclatura
são bem características no contexto da linguagem Python.
Apesar de eu ter adotado Python como linguagem principal desse blog, a princípio,
as ideias gerais desse post são aplicáveis a "qualquer" linguagem de programação.

{:refdef: style="text-align: center;"}
![](/images/2020-05-04/img002.png)
{: refdef}

As vantagens de organizar o código vão além do simples fato de possibilitar
reutilização. Uma das vantagens que, na minha opinião, é muito importante é a
da legibilidade, pois diminui o esforço cognitivo de compreensão do que está
sendo feito. Isso está muito ligado à conceitos de manutenibilidade. Um exemplo
rápido sobre isso é apresentado abaixo. Os dois códigos fazem a mesma coisa.
A função extract\_complex\_solution possivelmente nunca vai ser utilizada em
outro lugar no projeto. Mas a leitura do segundo código é mais fluida que do
primeiro.

{% highlight python linenos %}
big_x = solve(Ke, f)
P = np.zeros(shape=(len(big_x) // 2,), dtype=np.complex)
P.real = big_x[0:len(P)]
P.imag = big_x[len(P):2*len(P)]
{% endhighlight %}

{% highlight python linenos %}
P = extract_complex_solution(solve(Ke, f))
{% endhighlight %}

Outra vantagem é a possibilidade de testar individualmente as funções,
módulos e pacotes. O uso de ferramentas de automação de testes é um assunto a
parte que vai ser alvo de algum post futuro. Testes automatizados
são importantes pois possibilitam um crescimento saudável do software. Nessa
linha, temos as frentes que defendem o desenvolvimento orientado à testes (Test
Driven Development - TDD) e suas variações, como ATDD, BDD, etc.

Uma desvantagem que existe, em relação à separação de código em classes e
funções, é que pode existir um custo computacional associado. Em geral, o
argumento contra isso incluem: (1) em geral, as vantagens são mais relevantes;
(2) se for realmente crítico, existem formas de contornar
o problema; (3) o custo,
em geral, é ínfimo perante ao tempo de execução total do sistema.


# Implementação e discussão

Voltando ao problema original, proponho começar pelo corpo principal do programa.
Note que, a priori, nenhuma das funções foi ainda implementada. Inclusive,
a forma como elas estão dispostas pode mudar, a medida que o código evolui,
mas a ideia geral do corpo do programa está presente.
Ademais, mostro a organização inicial do projeto na imagem abaixo, para
referência. Lembrando que o projeto
[está disponível no GitHub](https://github.com/tarcisiofischer/helmholtz-solver).

{:refdef: style="text-align: center;"}
![](/images/2020-05-04/img003.png)
{: refdef}

Escolhi essa organização de forma orgânica, simplesmente utilizando
de conhecimentos passados, e imaginando como seria a solução final. Essa parte é
bastante subjetiva - Conheço pessoas que preferem já esboçar uma hierarquia
de classes desde o início, outras que preferem iniciar pelos testes, seguindo
as ideias de TDD, e outras que preferem "utilizar apenas o mínimo possível"
, fazendo um grande programa de um arquivo, colocando todas as variáveis,
cálculos e comentários. Cada uma dessas formas possui suas vantagens e
desvantagens.

Na verdade, não é raro eu iniciar um projeto com essa última abordagem,
fazendo um grande arquivo, quando eu tenho pouca ou nenhuma noção de como o
sistema vai ficar. Primeiro escrevo o programa inteiro e, a medida que vou
reconhecendo padrões, extraio funções, que eventualmente migram para
módulos, e assim vai...

**main.py**
{% highlight python linenos %}
from solver import solve_2d_hellmholtz
from mesh import build_mesh
import plotting

if __name__ == "__main__":
    mesh = build_mesh()
    solve_2d_hellmholtz(
        mesh,
        omega=300.0,
        callbacks=plotting.get_callbacks()
    )
{% endhighlight %}

**solver.py**
{% highlight python linenos %}
from callbacks import dispatch_callbacks
from complex_linear_system import convert_complex_linear_system_to_real,\
    extract_complex_solution


def solve_2d_hellmholtz(
    mesh,
    omega,
    callbacks=None,
):
    if callbacks is None:
        callbacks = {}

    Ke, f = _prepare_linear_system(mesh, omega)
    Ke, f = convert_complex_linear_system_to_real(Ke, f)
    P = extract_complex_solution(_solve_linear_system(Ke, f))

    dispatch_callbacks(callbacks, 'on_after_solve_2d_hellmholtz', P=P, mesh=mesh)


def _prepare_linear_system(mesh, omega):
    # TODO: Implement
    return None, None


def _solve_linear_system(Ke, f):
    # TODO: Implement
    return None
{% endhighlight %}

A ideia agora é popular aos poucos os módulos e funções. Ainda não existem pacotes
no projeto (E talvez nem tenha). Isso é um ponto importante: Não é só por que
algo existe que deve sempre ser utilizado. Esse projeto ainda está
em um nível demasiado simples, para existir a necessidade de adicionar pacotes,
classes, etc. Essas coisas vêm com a evolução gradual do software.

Por hora, esse código é uma forma bastante complexa de fazer nada. Por mais que
isso seja verdade, é interessante notar a quantidade de conceito e carga teórica
que já existe ali. É um trabalho de introduzir conceitos e intenções.
Mesmo o código não fazendo nada, é (ou deveria ser) compreensível o que
ele se propõe a fazer. Obviamente, essa não é uma característica intrínseca da
linguagem Python. Inclusive, em C++, por exemplo, esse código não seria muito
diferente. Deixo a função main traduzida abaixo, apenas para fomentar essa
argumentação.

**main.cpp**
{% highlight cpp linenos %}
#include <solve_2d_hellmholtz.h>
#include <build_mesh.h>
#include <plotting.h>

int main()
{
    auto mesh = build_mesh();
    auto omega = 300.0;
    solve_2d_hellmholtz(mesh, omega, plotting::get_callbacks());

    return 0;
}
{% endhighlight %}

Vou parando por aqui, e deixo a implementação própriamente dita como assunto
para as próximas semanas. Agradeço ao Tarcísio Crocomo pelas sugestões.

Até a próxima!
