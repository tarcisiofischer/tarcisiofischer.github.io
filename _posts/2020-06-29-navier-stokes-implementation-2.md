---
layout: post
title: Implementação de um simulador para as equações de Navier Stokes em Python - Parte 2
image: 2020-06-29/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-06-29/img001.png)
{: refdef}

Como comentado no post anterior, seguimos buscando melhorar o desempenho da
implementação em python puro, traduzindo os gargalos na implementação em Python
para C++. Sendo mais específico, esse post explora alguns detalhes na tradução
da função resíduo implementada em Python para C++.


# Tradução do código para C++

Como a implementação completa da função resíduo é demasiadamente grande, deixo
aqui apenas alguns trechos de código em C++, apenas para ilustrar como foi o
processo de tradução. A integração com C++ foi feita utilizando a biblioteca
Pybind11, e a biblioteca para matrizes e vetores foi a Eigen, como é de costume
aqui no blog.

{% highlight c++ linenos %}
// Extract useful variables from python objects
auto residual_size = residual.size();
auto pressure_mesh_size = py::len(graph.attr("pressure_mesh"));
auto pressure_mesh_nx = graph.attr("pressure_mesh").attr("nx").cast<int>();
auto pressure_mesh_ny = graph.attr("pressure_mesh").attr("ny").cast<int>();

// ...

    for (int i = 0; i < pressure_mesh_size; ++i) {
        // ...
        // Unknowns
        auto U_w = is_left_boundary ? 0.0 : U(i_U_w);
        auto U_e = is_right_boundary ? 0.0 : U(i_U_e);
        auto V_n = is_top_boundary ? 0.0 : V(i_V_n);
        auto V_s = is_bottom_boundary ? 0.0 : V(i_V_s);

        // Conservation of Mass
        auto ii = 3 * i;
        residual(ii) = (U_e * dy - U_w * dy) + (V_n * dx - V_s * dx);
    }

// ...

    for (int i = 0; i < ns_x_mesh_size; ++i) {
        // ...
        // Navier Stokes X
        auto transient_term = (rho * U_P - rho * U_P_old) * (dx * dy / dt);
        auto advective_term = \
            rho * U_e * ((.5 - beta_U_e) * U_E + (.5 + beta_U_e) * U_P) * dy - \
            rho * U_w * ((.5 - beta_U_w) * U_P + (.5 + beta_U_w) * U_W) * dy + \
            rho * V_n * ((.5 - beta_V_n) * U_N + (.5 + beta_V_n) * U_P) * dx - \
            rho * V_s * ((.5 - beta_V_s) * U_P + (.5 + beta_V_s) * U_S) * dx;
        auto difusive_term = \
            mi * dU_e_dx * dy - \
            mi * dU_w_dx * dy + \
            mi * dU_n_dx * dx - \
            mi * dU_s_dx * dx;
        auto source_term = -(P_e - P_w) * dy;
        
        auto ii = 3 * (i + j) + 1;
        residual(ii) = transient_term + advective_term - difusive_term - source_term;
    }

// ...
{% endhighlight %}

Como as operações são efetuadas ponto-a-ponto, e não foi utilizada vetorização
com Numpy, a tradução fica simples para a maior parte do
código desse caso. Além disso, como foi utilizado a Eigen, a conversão dos arrays
da Numpy para C++ é automático, de forma que a assinatura final da função fica
da seguinte forma, em C++:

{% highlight c++ linenos %}
using EigenArray1d = Eigen::Array<double, Eigen::Dynamic, 1>;

EigenArray1d residual_function(
    Eigen::Ref<EigenArray1d const> const& X,
    py::object graph
);

PYBIND11_PLUGIN(_residual_function) {
    py::module m("_residual_function");
    m.def("residual_function", &residual_function);
    return m.ptr();
}
{% endhighlight %}

Para manter compatibilidade com ambas implementações (Python e C++), foi feito
um wrapper para receber a função residuo como parametro de controle, de forma
que é possível executar o programa tanto com a função resíduo em C++ quanto
em Python. Isso é bastante útil para testar o código, validar e mensurar os
ganhos de desempenho. O único detalhe é que a interface da função resíduo
na PETSc é levemente diferente daquela que implementei. Isso é resolvido
exatamente nesse wrapper, que possui um método para fazer essa ponte, como
mostrado no código a seguir.

{% highlight python linenos %}
class PetscSolverWrapper:
    def __init__(self, residual_f):
        self._active_graph = None
        self._residual_f = residual_f
        self._first_run = True
        self._setup_options()

    def residual_function_for_petsc(self, snes, x, f):
        '''
        Wrapper over our `residual_f` so that it's in a way expected by PETSc.
        '''
        x = x[:]  # transforms `PETSc.Vec` into `numpy.ndarray`
        f[:] = self._residual_f(x, self._active_graph)
        f.assemble()
{% endhighlight %}


# Resultados e discussão

A ilustração abaixo mostra a comparação do tempo de execução entre o código
implementado completamente em Python, e movendo um dos gargalos para C++. Os
tempos se referem à execução do problema da cavidade até atingir regime 
permanente, variando a malha de 8x8 até 64x64 volumes de controle. Foram
escolhidos poucos volumes apenas pelo fato de que a implementação em puro
Python era muito lenta. Inclusive, foi omitido o tempo da malha de 64x64 em Python,
pois era muito grande em comparação com os outros.

Não foi feita comparação com implementações externas. De qualquer forma, a ideia
aqui foi simplesmente mostrar que não é necessário traduzir todo o código para
uma linguagem de baixo nível para haver um ganho substancial de desempenho.

{:refdef: style="text-align: center;"}
![](/images/2020-06-29/img002.png)
{: refdef}

Etapas de configuração e de pós processamento tendem a ter pouca relevância, a
medida que o tamanho do problema aumenta. Por isso, dado as vantagens de utilizar
linguagens interpretadas como Python, em relação à disponibilidade de bibliotecas,
e velocidade de implementação, pode ser bastante interessante fazer esse tipo
de abordagem incremental evolutiva sobre o código, a fim de tentar balancear
um ambiente de entregas parciais e contínuas, melhorando o software aos poucos.

Pretendo utilizar esse projeto ainda no futuro em posts individuais para explorar outros aspectos
computacionais, mas termino por aqui essa pequena série de posts. Ainda é possível
melhorar muito, por exemplo, caminhando para um paralelismo em uma decomposição
espacial do domínio, modificando o método numérico, apenas para comentar alguns
dos possíveis projetos futuros ;)

Até a próxima!
