---
layout: post
title: Resolvendo a equação de difusão em Python - Parte 5
---

Nesse último post planejado da série, gostaria de agradecer todos os feedbacks
positivos que tenho recebido, incluindo dicas, sugestões e agradecimentos.
Claro que, apesar de ser o último post dessa série, pretendo
ainda continuar o blog com outros temas. Inclusive, é possível que eu aproveite
esse mesmo projeto para explicar outros tópicos. Se você, leitor, tiver alguma
sugestão fique livre pra me procurar no LinkedIn, Telegram ou qualquer rede
social :)

Nesse post, vamos trabalhar com a integração Python/C++ dentro de um contexto
bastante específico: A troca do solver linear. Na parte 3 dessa série, decidi
ficar com o GMRes por sua performance, apesar de ser um solver iterativo. Por
outro lado, um solver bastante conhecido na academia e até na industria, pelo
menos dentro dos meus circulos de contato, é o [MKL Pardiso](https://software.intel.com/en-us/articles/intel-mkl-pardiso) (Parallel Direct
Solver), que atualmente pode ser [baixado gratuitamente direto do site da Intel](https://software.intel.com/en-us/mkl/choose-download).

Porém, o PARDISO é disponibilizado por meio de uma DLL dentro do pacote da MKL.
A ideia desse post é utilizar a PyBind11 para criar uma ponte entre a chamada
do PARDISO e o código em Python.

{:refdef: style="text-align: center;"}
![](/images/pybind11.png)
{: refdef}


# PARDISO - Parallel Direct Solver

Para ser sincero, eu usei muito pouco esse solver nos meus projetos,
então sou um mero usuário iniciante na ferramenta (Inclusive aproveitei esse
post pra dar uma olhada nela). O código abaixo é baseado
nos exemplos da documentação do próprio PARDISO, com pequenas modificações
para facilitar a integração posterior com Python. Note que para o exemplo, eu
estou ignorando qualquer erro gerado pelo procedimento.

O PARDISO trabalha com a matriz $A$ esparsa. Por isso, existem três parâmetros
a serem passados, que compoem a matriz em CSR. Para maiores detalhes sobre a
configuração do PARDISO, sugiro a leitura do [manual oficial no site da Intel](https://software.intel.com/en-us/mkl/documentation/get-started).
O objetivo dessa seção não é aprender a utilizar o PARDISO, mas sim apenas
introduzir a noção de que "existe uma função em C/C++ dentro de um módulo que
gostariamos de acessar via Python".

Caso o leitor não esteja acostumado
com os detalhes da linguagem ou mesmo da biblioteca (Eu mesmo
usei o PARDISO em apenas um projeto antes desse, e já faz alguns anos), não
se preocupe - Visualize apenas o "big picture": Existe uma função em C++ que
resolve um sistema linear $Ax = b$ e gostariamos de utiliza-la no Python. Como
podemos fazer isso será o tema da próxima seção.

{% highlight c++ linenos %}
#include <pardiso.hpp>

#include <math.h>

#include <mkl_pardiso.h>
#include <mkl_types.h>

/* Internal solver memory pointer pt, */
/* 32-bit: int pt[64]; 64-bit: long int pt[64] */
/* or void *pt[64] should be OK on both architectures */
void *pt[64];
int iparm[64];

void pardiso_setup()
{
    for (int i = 0; i < 64; i++) {
        pt[i] = 0;
    }

    for (int i = 0; i < 64; i++) {
        iparm[i] = 0;
    }
    iparm[0] = 1;         /* No solver default */
    iparm[1] = 2;         /* Fill-in reordering from METIS */
    iparm[3] = 0;         /* No iterative-direct algorithm */
    iparm[4] = 0;         /* No user fill-in reducing permutation */
    iparm[5] = 0;         /* Write solution into x */
    iparm[6] = 0;         /* Not in use */
    iparm[7] = 2;         /* Max numbers of iterative refinement steps */
    iparm[8] = 0;         /* Not in use */
    iparm[9] = 13;        /* Perturb the pivot elements with 1E-13 */
    iparm[10] = 1;        /* Use nonsymmetric permutation and scaling MPS */
    iparm[11] = 0;        /* Conjugate transposed/transpose solve */
    iparm[12] = 1;        /* Maximum weighted matching algorithm is switched-on (default for non-symmetric) */
    iparm[13] = 0;        /* Output: Number of perturbed pivots */
    iparm[14] = 0;        /* Not in use */
    iparm[15] = 0;        /* Not in use */
    iparm[16] = 0;        /* Not in use */
    iparm[17] = -1;       /* Output: Number of nonzeros in the factor LU */
    iparm[18] = -1;       /* Output: Mflops for LU factorization */
    iparm[19] = 0;        /* Output: Numbers of CG Iterations */
    iparm[34] = 1;        /* 0 -> Fortran indexing ; 1 -> C-Style indexing */
}

void pardiso_solve(double const* A, int const* iA, int const* jA, double const* b, double* x, int n)
{
    auto _unused_d = double(NAN);
    auto _unused_i = 0;
    auto maxfct = 1; // Maximum number of numerical factorizations
    auto mnum = 1; // Which factorization to use
    auto msglvl = 0; // Print statistical information
    auto error = 0; // Initialize error flag
    auto nrhs = 1; // Number of right-hand side
    auto mtype = 11;
    auto phase = 0;

    phase = 11;
    PARDISO(pt, &maxfct, &mnum, &mtype, &phase, &n, A, iA, jA, &_unused_i, &nrhs, iparm, &msglvl, &_unused_d, &_unused_d, &error);
    phase = 22;
    PARDISO(pt, &maxfct, &mnum, &mtype, &phase, &n, A, iA, jA, &_unused_i, &nrhs, iparm, &msglvl, &_unused_d, &_unused_d, &error);
    phase = 33;
    PARDISO(pt, &maxfct, &mnum, &mtype, &phase, &n, A, iA, jA, &_unused_i, &nrhs, iparm, &msglvl, const_cast<double*>(b), x, &error);
    phase = -1;
    PARDISO(pt, &maxfct, &mnum, &mtype, &phase, &n, &_unused_d, iA, jA, &_unused_i, &nrhs, iparm, &msglvl, &_unused_d, &_unused_d, &error);
}
{% endhighlight %}


# Integrando Python e C++ com a Pybind11

Na verdade, existem muitas possibilidades de como fazer a integração e chamada
de funções de C++ em Python. A primeira delas, e talvez mais óbvia, seria
procurar no Github ou na internet pacotes que já fazem esse trabalho pra você.
Por exemplo, para o presente problema, eu poderia simplesmente procurar algo
como "calling PARDISO from Python" no Google e verificar se alguém já não tinha
feito.

Outra possibilidade é utilizando a biblioteca [CTypes](https://docs.python.org/3/library/ctypes.html), que possibilita
a chamada de funções de dentro de uma DLL em Python. Outra possibilidade ainda
seria utilizando a própria API de [Python](https://docs.python.org/3/extending/extending.html),
que utiliza diretamente códigos em C. Talvez alguém sugerisse [Cython](https://cython.org/) como uma
alternativa não tão low-level. Já vi pessoas preferirem a [Boost Python](https://www.boost.org/doc/libs/1_70_0/libs/python/doc/html/index.html),
biblioteca que inclusive eu utilizei por muito tempo.

Cada uma dessas alternativas (E devem existir outras) possuem suas
vantagens e desvantagens. Não vou entrar no mérito de comparar cada uma delas
individualmente. Minha escolha pela
PyBind11 vai ser motivada por um conjunto de fatores pessoais e práticos, dos
quais listo quatro que me vêm à cabeça:

1) É a biblioteca que eu mais tenho usado para o propósito de integrar C++
com Python, pelo menos em projetos pessoais;

2) Possui tanto uma utilização
de alto nível quanto, caso necessário, baixo nível (É fácil "abrir o capô" e
"mexer no motor");

3) Header-Only library (Embora algumas pessoas colocariam isso como um ponto
negativo heheh...)

4) Integração fácil com a ndarray da Numpy

Vou deixar a configuração do ambiente para compilação na mão do leitor, pois
não quero transformar esse post em um tutorial. Eu já fiz algumas [palestras sobre a Pybind11](https://github.com/tarcisiofischer/examples-and-demos/blob/master/Integrando%20Python%20e%20C%2B%2B%20com%20pybind11.pdf),
e os slides estão disponíveis publicamente. Além disso, as
[documentações da Pybind11](https://pybind11.readthedocs.io/en/stable/) são bastante boas, e, acredito, já cumprem esse
propósito. Para o escopo desse post, utilizei a Pybind11 na versão 2.3, embora
já exista a 2.4. Estou linkando com o Python 3.6 para fazer os testes.

{% highlight c++ linenos %}
#include <pybind11/pybind11.h>
#include <pybind11/numpy.h>
#include <pardiso.hpp>

namespace py = pybind11;

PYBIND11_MODULE(pypardiso, m)
{
    m.def("setup", &pardiso_setup);
    m.def("solve", [](
        py::array_t<int> const& rows_A,
        py::array_t<int> const& cols_A,
        py::array_t<double> const& A,
        py::array_t<double> const& py_b
    )
    {
        auto n = int(py_b.size());
        auto x = py::array_t<double>({size_t(n)});
        pardiso_solve(A.data(), rows_A.data(), cols_A.data(), py_b.data(), x.mutable_data(), n);
        return x;
    });
}
{% endhighlight %}

Um comentário rápido, que não tem nada a ver com o post: Perceba como número
de linhas de código é uma métrica horrível para determinar tempo de
desenvolvimento. O número de linhas de código do bloco anterior é bastante
pequeno, mas a quantidade de tecnologia específica exige um nível de conhecimento
muito maior, se comparado a velocidade de digitação do programador.

Voltando ao tema, o código acima é compilado e linkado com o Python e a MKL,
juntamente com a API que construí anteriormente, produzindo uma biblioteca
dinâmica cuja extensão é renomeada para .pyd, e esse artefato pode ser importado
a partir do Python. Basicamente, a parte difícil está pronta :)

Perceba a preocupação com a passagem dos tipos na lambda function do solve:
O tipo [py::array](https://pybind11.readthedocs.io/en/stable/advanced/pycpp/numpy.html) disponível na PyBind11 possui os métodos data() e
mutable_data(), que possibilitam o acesso direto dos valores em memória dos
numpy arrays sem precisar fazer a cópia dos dados, e garantindo constness,
quando requerido. Embora isso seja muito bom, na minha opinião ainda é bastante
low-level. Duas alternativas interessantes são as bibliotecas [XTensor](https://github.com/xtensor-stack/xtensor) e [Eigen](https://eigen.tuxfamily.org/),
que eu só não estou utilizando nesse post para não adicionar ainda mais
dependências no exemplo.


# Utilizando a biblioteca do Python

O artefato gerado na sessão anterior é um arquivo pyd que pode ser importado
diretamente do Python. Para utilizar no nosso projeto da equação da difusão,
basta trocar a chamada do nosso solver linear, que anteriormente era feita
para o GMRes:

{% highlight python linenos %}
def solve_linear_system(A, b):
    x, _ = linalg.gmres(A, b)
    return x
{% endhighlight %}

E agora pode ser feita diretamente pelo nosso binding:

{% highlight python linenos %}
import pypardiso
def solve_linear_system(A, b):
    return pypardiso.solve(A.indptr, A.indices, A.data, b)
{% endhighlight %}

A única modificação adicional, que é algo inclusive que é pertinente fazer
para a utilização do GMRes, é que é necessário transformar a matriz A em
esparsa. O código relevante a ser modificado é:

{% highlight python linenos %}
def solve_diffusion_equation(N, c, ds, dt, final_time, initial_conditions):
    A = build_A(N, c, ds, dt)
    A_sparse = csr_matrix(A)
    # ...
{% endhighlight %}

Claro que, a rigor (E como eu já tinha comentado), deveriamos gerar a matriz
esparsa dentro da função build_A. Porém, a titulo de experimento apenas da
integração Python com C++, o exemplo acima é suficiente.

Por fim, é necessário fazer a configuração (setup) do PARDISO, chamando a
função no início do script:

{% highlight python linenos %}
pypardiso.setup()

N = 80
ics = np.zeros(shape=(N, N,))
ics[N//4:N - N//4, N//4:N - N//4] = 10.0
result = solve_diffusion_equation(
    N,
    c=0.1,
    ds=0.1,
    dt=0.1,
    final_time=2.0,
    initial_conditions=ics
)
{% endhighlight %}



# Conclusão e discussão

Note que eu deixei um pouco de lado a parte puramente mecânica de todo o
processo (Não mostrei que botão que eu aperto para gerar o módulo Python a
partir dos códigos acima). Porém, gostaria que o leitor levasse em consideração
o processo de solução do problema como um todo: Queriamos testar o PARDISO,
que a prióri pode não estar disponível em Python; Fizemos uma breve análise
de alternativas de como fazer isso e escolhemos uma; Criamos um pequeno código
sem validações em tempo de execução, nem nos preocupamos em entender todas
as configurações; Geramos um protótipo final e funcional que, agora, pode ser
estudado com mais cuidado, se necessário.

A integração de Python com C++ nem sempre é motivada puramente por performance,
que acredito ser a motivação que as pessoas mais me comentam. A integração com
código inacessível do Python, geralmente em C/C++ ou mesmo outras linguagens
como Fortran também tem seu valor, como foi o caso desse post. Seja pra fazer
um teste rápido (protótipo), ou mesmo para uma integração completa que vai pro
software final. 

Se você ficou interessado em saber mais sobre o uso da Pybind11, sugiro a
leitura da parte básica da [documentação oficial](https://pybind11.readthedocs.io/en/stable/basics.html).
Para dúvidas mais específicas, pode me procurar nas redes sociais que, na medida
do possível, eu respondo ;) Até a próxima!
