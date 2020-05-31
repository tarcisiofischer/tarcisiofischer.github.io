---
layout: post
title: Movendo código de um solver em Python para C++
---

{:refdef: style="text-align: center;"}
![](/images/2020-06-01/img001.png)
{: refdef}

Esse é, possívelmente, o último post da sobre a solução da equação
de Helmholtz em elementos finitos. Para fechar, mostro os resultados
de mover partes do gargalo computacional da implementação de posts passados
para C++, utilizando a biblioteca Pybind11. O alvo não é o código inteiro,
apenas as funções que possuiam maior custo computacional.


# Análise de performance

Conforme post anterior, observou-se que o gargalo computacional está relacionado
principalmente com a transformação do sistema linear complexo, e com a construção
do sistema linear esparso (Matriz $K$ e vetor $f$). Deixo abaixo os resultados
anteriores, apenas para referência.

{% highlight python linenos %}
     2600681 function calls (2585757 primitive calls) in 11.122 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    2.534    2.534    2.534    2.534 complex_linear_system.py:5(_build_extended_A)
     6400    2.483    0.000    4.285    0.001 solver.py:28(_build_local_Ke)
     6400    0.952    0.000    1.968    0.000 solver.py:103(_build_local_f)
   173576    0.875    0.000    0.875    0.000 {built-in method numpy.array}
{% endhighlight %}

A ideia é mover pontualmente essas funções para C++. Outras abordagens poderiam
ser sugeridas, por exemplo, a paralelização da montagem da matriz, utilização
de Cython, ou mesmo FORTRAN. Acredito que todas são válidas, porém, a fim de
estabelecer um caminho único (e por preferência pessoal), escolho fazer a
tradução dos códigos para C++ e utilizar a biblioteca Pybind11 para fazer
a integração com Python.

A biblioteca já apareceu aqui no blog em outra ocasião. Basicamente, configuram-se
algumas funções em C++, em uma descrição de módulo, e posteriormente compila-se
tal módulo gerando uma biblioteca importável a partir do Python. O meu módulo
ficou da seguinte forma:

{% highlight python linenos %}
namespace py = pybind11;
PYBIND11_MODULE(_fwi_ls, m) {
    m.def("build_local_Ke", &fwi_ls::build_local_Ke);
    m.def("build_local_f", &fwi_ls::build_local_f);
    m.def("build_extended_A", &fwi_ls::build_extended_A);
    m.def("build_connectivity_list", &fwi_ls::build_connectivity_list);
}
{% endhighlight %}


# Tradução de código

Resta agora implementar cada uma dessas funções. Claro que não faremos isso do
zero. É muito mais fácil se basear nas implementações prontas, feitas em Python,
e apenas escolher o código equivalente em C++. Mostrarei apenas alguns trechos
de código, e como foram feitas as traduções para eles. Acredito que seja
suficiente para ter uma visão geral da abordagem.

Em primeiro lugar, utilizei a biblioteca Eigen para compatibilizar as estruturas
de arrays. Essa escolha não é arbitrária. Acontece que existe uma interoperabilidade
já implementada na Pybind11, então, os arrays da Numpy são automaticamente
convertidos para arrays da Eigen e, com suficiente cuidado, não existem cópias
envolvidas no processo. A tradução é mostrada abaixo. Eu reconheço que fica
muito mais verboso, mas não passa de um processo de tradução. Uma vez escolhida
a tradução que se quer, é só replicar por todo o código. Uma sugestão (que não
estou utilizando aqui) é o uso de `typedef`s em C++.


{% highlight python linenos %}
def _build_local_Ke(element_points, omega, mu, eta):
{% endhighlight %}

{% highlight c++ linenos %}
Eigen::Array<std::complex<double>, 4, 4> fwi_ls::build_local_Ke(
    Eigen::Ref<Eigen::Array<double, 4, 2> const> const& element_points,
    double omega,
    double mu,
    double eta
)
{% endhighlight %}

A inicialização de arrays pode ser feito como segue. Não há grandes comentários.
Novamente, C++ é mais verboso, porém, dada a escolha de tradução do código,
é só replicar pelo resto das funções.

{% highlight python linenos %}
integration_point = np.array(
    [
        [-1.0 / np.sqrt(3.0), -1.0 / np.sqrt(3.0)],
        [+1.0 / np.sqrt(3.0), -1.0 / np.sqrt(3.0)],
        [+1.0 / np.sqrt(3.0), +1.0 / np.sqrt(3.0)],
        [-1.0 / np.sqrt(3.0), +1.0 / np.sqrt(3.0)],
    ]
)
{% endhighlight %}

{% highlight c++ linenos %}
auto integration_point = Eigen::Array<double, 4, 2>();
integration_point <<
    -1. / sqrt(3.), -1. / sqrt(3.),
    +1. / sqrt(3.), -1. / sqrt(3.),
    +1. / sqrt(3.), +1. / sqrt(3.),
    -1. / sqrt(3.), +1. / sqrt(3.);
{% endhighlight %}

A Eigen possui duas estruturas principais: Uma que serve para trabalhar com
arrays, e outra para trabalhar com matrizes. A diferença é que as matrizes possuem
métodos específicos, tais como a computação do determinante. Pra ser sincero,
eu nunca havia utilizado essas funções antes, pois geralmente eu resolvo esses
problemas de outra forma, e acabo usando mais os Arrays, no meu dia-a-dia. Digo
isso porque não se se a minha tradução para o código abaixo não poderia ser
melhor, de alguma outra forma. Deixo isso bem explícito no código em C++,
por meio de um "TODO" (to-do).

{% highlight python linenos %}
J = np.dot(grad_N.T, element_points)
dJ = det(J)
{% endhighlight %}

{% highlight c++ linenos %}
// TODO: I don't know if the following operations are fast. Must check
// later.
auto J = (grad_N.transpose().matrix() * element_points.matrix()).eval();
auto dJ = J.determinant();
{% endhighlight %}

O resto do código está [disponível no GitHub](https://github.com/tarcisiofischer/helmholtz-solver). Acredito que esses exemplos são
suficientes para entender a ideia de "tradução" de código. No fim, são pequena
"receitas" que são usadas caso-a-caso.


# Resultados e discussão

Abaixo, mostro uma comparação do tempo de execução antes e depois da modificação.
Como eu já havia comentado, estou comparando minha implementação com ela mesma, o
que pode dar a entender que os novos tempos são excelentes. Porém, acredito que
se fosse fazer um comparativo com outras implementações, por exemplo, uma
baseada no projeto FEniCS, poderia-se observar que ainda há muito a ser feito.
A questão aqui foi apenas exemplificar o processo de desenvolvimento, com um
caso de uso simples, e não competir com outras bibliotecas.

{:refdef: style="text-align: center;"}
![](/images/2020-06-01/img002.png)
{: refdef}

Uma vantagem que vejo dessa abordagem, em relação a implementar tudo em C++
ou em FORTRAN de uma vez, é que em Python, pode-se modificar e
testar o código várias vezes sem ter que passar pelo processo de compilação,
linkagem e execução. Isso sem falar os tempos perdidos em debuging. O argumento é
que você ganha tempo no desenvolvimento de modelos e/ou procedimentos de
solução que não estejam ainda disponíveis em forma de bibliotecas de terceiros.

Por fim, convido o leitor a [dar uma olhada na implementação completa, no GitHub](https://github.com/tarcisiofischer/helmholtz-solver).
É possível que, em algum momento no futuro, eu volte a trabalhar um pouco mais
em outros aspectos desse projeto. Se você tiver interesse em algum ponto mais
específico, me procure nas redes sociais que ficarei feliz em conversar :)

Até a próxima!
