---
layout: post
title: Automatização de testes numéricos com Pytest
image: 2020-07-06/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-07-06/img001.png)
{: refdef}


Ao fazer alterações e melhorias de performance em um software (e eu nem estou
restringindo à software numérico), garantir que ele continua funcionando é
essencial. Verificação formal é um caminho muito complicado e, geralmente,
não é o caminho que se toma, pelo menos na industria. Testar o software manualmente
é tedioso, além de ser sujeito à erros e enviesamentos por parte do testador.

Apesar de não garantir a corretude do software, a automatização de testes de
software é uma alternativa de baixo custo. A ideia é ter um conjunto de códigos
que, quando executado, verifica alguns aspectos da implementação, comparando
resultados e saídas dos algoritmos com valores pré-determinados.


# Pytest

No contexto da linguagem Python, a biblioteca [Pytest](https://docs.pytest.org/)
auxilia na criação e execução de testes automatizados. É uma biblioteca de
propósito geral, fácil e prática de utilizar tanto em pequenos quanto grandes
projetos. Abaixo um exemplo de teste automatizado, apenas para dar uma noção
ao leitor.

{% highlight python linenos %}
@pytest.mark.parametrize(
    ('ply_name', 'expected_number_of_faces', 'expected_number_of_points'),
    [
        ('cube', 6, 24),
        ('sphere', 840, 422),
    ],
)
def testReadPlyCube(ply_name, expected_number_of_faces, expected_number_of_points):
    expected_cube_points = np.loadtxt(
        getTestFile("expected_" + ply_name + "_points.txt")
    )
    polyhedron = readPly(getTestFile(ply_name + ".ply"))

    assert len(polyhedron.getFaces()) == expected_number_of_faces
    assert len(polyhedron.getPoints()) == expected_number_of_points
    assert np.allclose(polyhedron.getPoints(), expected_cube_points)
{% endhighlight %}

Existe muito conteúdo sobre a biblioteca na internet, então, não vou alongar
esse post nesse sentido. Sugiro a leitura da [documentação oficial](https://docs.pytest.org/) e, caso o
leitor tenha interesse, o [livro do Bruno Oliveira](https://www.amazon.com.br/pytest-Quick-Start-Guide-maintainable-ebook/dp/B07GZJD473), um dos desenvolvedores da
biblioteca.

A biblioteca também permite a criação de extensões (plugins).
Essas extensões possuem funções adicionais que preparam e provêem facilidades
para testes de domínio específico. Por exemplo, pytest-qt é uma extensão que
possui funções para testar interfaces gráficas escritas em Qt. Então, é possível
simular cliques, verificar conteúdo de janelas, etc. Uma lista de plugins para
o pytest está disponível em [http://plugincompat.herokuapp.com/](http://plugincompat.herokuapp.com/)
e na [documentação oficial](https://docs.pytest.org/en/2.7.3/plugins_index/index.html).


# Teste numérico com Pytest

No contexto do desenvolvimento de software de simulação, é necessário um cuidado
na criação dos testes. Como eu [já mencionei aqui no blog](https://tarcisiofischer.github.io/2020-04-06/floating-point), o uso de variáveis do
tipo ponto flutuante (double ou float, por exemplo), acaba gerando a demanda por
verificações não absolutas de valores. Isso por que, como visto,
é difícil (se não impossível) garantir que os erros decorrentes das operações
resultem em exatamente o número que o programador espera. Isso sem considerar
os erros de truncamento decorrentes de aproximações dos métodos numéricos.

Dessa forma, sugere-se evitar comparações absolutas nos testes numéricos que
utilizam de tais representações. Ao invés disso, existem funções auxiliares para
esses testes. No contexto da biblioteca pytest, pode-se utilizar a função approx,
como no exemplo abaixo. Sendo totalmente sincero, eu acho a sintaxe dela um
pouco esquisita: Eu esperaria passar os dois valores para a função. Mas detalhes
à parte, a [documentação oficial](https://docs.pytest.org/en/stable/reference.html#pytest-approx) traz vários exemplos de uso.

{% highlight python linenos %}
_arora_example_f = lambda x: 2 - 4 * x[0] + np.e ** x[0]
_arora_example_minimizer = 1.386511
_arora_example_minimum = _arora_example_f(np.array([_arora_example_minimizer]))

def test_equally_spaced_line_search():
    x0 = 0.0
    d = np.array([+1.0])

    alpha = equal_interval_search(
        _arora_example_f,
        d,
        x0,
        DEFAULT_STRATEGY_FUNCTIONS,
        tol=1e-6
    )
    minimum = _arora_example_f(alpha * d)

    assert pytest.approx(minimum) == _arora_example_minimum
{% endhighlight %}

Por outro lado, não é incomum, para softwares de aproximação de soluções para
equações diferenciais parciais, como àqueles ligados à simulação computacional,
necessitarmos verificar e analisar perfis em forma de curvas, e não valores
individuais. Por exemplo, no caso do problema da cavidade,
[resolvido aqui no blog nas últimas semanas](https://tarcisiofischer.github.io/2020-06-15/navier-stokes-disc-fvm),
uma forma de validar seus resultados é verificar as propriedades na
linha média vertical e horizontal da malha, e comparar com resultados clássicos
na literatura. Porém, esse procedimento normalmente é feito manualmente.

Para auxiliar nesse processo, o plugin [pytest-regressions](https://pytest-regressions.readthedocs.io/en/latest/)
possui funções na sua API de [numerical regressions](https://pytest-regressions.readthedocs.io/en/latest/api.html#num-regression)
que permite comparar curvas inteiras ao mesmo tempo, inclusive gerando arquivos
em CSV que podem ser visualizados em outras ferramentas, como a matplotlib,
ou mesmo o Excel. Eu, particularmente, utilizo
bastante esse plugin - inclusive parcitipei de uma parte do desenvolvimento
em versões mais antigas. Abaixo um exemplo extraído de outro projeto, apenas
para ilustrar seu uso em casos reais:

{% highlight python linenos %}
def test_forward_case_small_mesh(num_regression):
    case = build_forward_case(Source("my_source", (30.0, 30.0), value=1.0))
    P = forward_solver.solve_2d_hellmholtz(case).real

    num_regression.check({
        "P_horizontal_middle": P[:, P.shape[1] // 2],
        "P_vertical_middle": P[P.shape[0] // 2, :],
    })
{% endhighlight %}

Como comentado inicialmente, a produção e automatização de testes não garantem
a corretude do código. Porém, como validações formais são complicadas, e em
geral, não práticas (salvo casos específicos), acaba sendo uma alternativa
utilizada em diversos projetos de software. Espero que esse post tenha instigado
o leitor a pesquisar mais sobre o tema :)

Até a próxima!
