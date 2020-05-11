---
layout: post
title: Geração de malha bidimensional para o problema de Helmholtz
---

O próximo passo no processo de implementação da solução da
equação de Helmholtz, será a geração da malha e distribuição de propriedades do meio.
Considera-se, aqui, um domínio bidimensional, discretizado através de elementos
quadriláteros. Ainda
não estou me preocupando com performance, mas sim com a implementação em si.
Por isso, todo código é escrito em Python - Lembrando que linguagens como
C++ e FORTRAN possuem uma performance melhor, se comparadas à Python puro. O código dessa etapa
[já está disponível no meu GitHub](https://github.com/tarcisiofischer/helmholtz-solver).
Encorajo-os a dar uma conferida!

{:refdef: style="text-align: center;"}
![](/images/2020-05-11/img001.png)
{: refdef}


# Geração de malha em Python com Numpy

A representação da malha bidimensional será através de um conjunto
de propriedades físicas (dimensões e propriedades), geométricas (pontos) e
topológicas (conectividade). A palavra "propriedade" aqui, se refere aos campos
escalares conhecidos no meio ($\eta$ e $\mu$). Como esses dados estão fortemente
relacionados, decido por agrupa-los em uma estrutura "Mesh".

{% highlight python linenos %}
class Mesh:

    def __init__(self, nx, ny, points, connectivity_list, size_x, size_y):
        self.points = points
        self.connectivity_list = connectivity_list
        self.n_points = (nx + 1) * (ny + 1)
        self.nx = nx
        self.ny = ny
        self.size_x = size_x
        self.size_y = size_y
        self.points_in_elements = self._build_points_in_elements()
{% endhighlight %}

Enquanto essa classe cuida das informações da malha, a criação de uma
instância da mesma fica por conta de uma free-function "build_mesh". Essa função,
é responsável por definir a disposição dos pontos (nós) e a conectividade
dos mesmos na malha, conforme o código abaixo. A disposição de pontos será
uniforme, dados os tamanhos geométricos da malha. A lista de conectividade foi
gerada em puro Python (Que pode se tornar um problema de performance, a medida
que a malha aumenta).

{% highlight python linenos %}
def _build_connectivity_list(nx, ny):
    from_ij_to_flat_index = lambda i, j: (nx + 1) * i + j
    connectivity_list = [None] * (nx * ny)
    for i in range(ny):
        for j in range(nx):
            connectivity_list[i * nx + j] = [
                from_ij_to_flat_index(i, j),
                from_ij_to_flat_index(i, j + 1),
                from_ij_to_flat_index(i + 1, j + 1),
                from_ij_to_flat_index(i + 1, j),
            ]
    return np.array(connectivity_list)
{% endhighlight %}

Apenas por curiosidade, plotei os tempos de geração para alguns tamanhos de
malha em relação ao número de nós $N$. Percebe-se uma tendência exponencial no
tempo de execução (Fruto da complexidade $O(N^2)$ devido ao duplo aninhamento
de loops), o que mostra que, para $N$ suficientemente grande, o tempo
pode ser tornar um problema. Aqui, estou considerando $N = n_x = n_y$. Fica como
exercício aos leitores tentar melhorar esse tempo de execução, de alguma forma.

{:refdef: style="text-align: center;"}
![](/images/2020-05-11/img002.png)
{: refdef}


# Mapeamento dos campos escalares

Enquanto a geração da geométria e topologia da malha são feitas com o
procedimento explicado anteriormente, o mapeamento dos campos de propriedades 
escalares são feitos separadamente. Para manter as distribuições contínuas,
iremos definir funções escalares bidimensionais $mu(x, y)$ e $eta(x, y)$, ao invés de
computar os valores nodais. Como as funções que usaremos de exemplo terão
regiões com distribuição constante, é útil fazer um "builder" de funções desse
tipo.

{% highlight python linenos %}
class RegionFunctionBuilder():

    def build(self):
        def f(x, y):
            for region in self._region_list:
                if region.is_point_inside((x, y)):
                    return region.value

            return self._overall_value
        return f
{% endhighlight %}

O resultado final para um exemplo é mostrado na figura abaixo. O plot foi realizado
no Paraview, utilizando uma pequena variação do código que mostrei em um post
passado desse blog. A camada mais inferior (estrutura quadriculada), representa
a conectividade da malha. As duas camadas acima representam os campos contínuos de
propriedades, relacionados à $\mu$ e $\eta$. Como esses campos são contínuos,
eles oferecem maior liberdade para recuperar valores em qualquer posição. Os
próximos passos serão a montagem do sistema linear de equações e resolvê-lo.

{:refdef: style="text-align: center;"}
![](/images/2020-05-11/img003.png)
{: refdef}

A discussão sobre a implementação diretamente em Python, as vezes geram
dúvidas sobre a possibilidade de transformar o mesmo código para C++ ou FORTRAN,
a fim de diminuir o tempo de execução. Conforme as estruturas de dados utilizados
no Python ficam mais complexas, surgem dúvidas se é possível fazer tal transformação.
Apenas por curiosidade, mostro um exemplo de como fazer algo análogo em C++.
O código abaixo não será de fato utilizado no atual projeto, mas fica o
exemplo para quem tem curiosidade de como seria um possível caminho para
transformar o código Python em C++.

{% highlight cpp linenos %}
std::function<double(double, double)> RegionFunctionBuilder::build()
{
    auto f = [&](double x, double y) -> double
    {
        for (auto const& region : this->region_list) {
            if (region.is_point_inside(x, y)) {
                return region.value;
            }
        }
        return this->overall_value;
    };
    return f;
}
{% endhighlight %}

Agradeço ao Bruno Klahr pelas sugestões. Até a próxima!
