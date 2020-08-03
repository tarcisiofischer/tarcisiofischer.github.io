---
layout: post
title: Metaprogramação estática em C++ - Um estudo básico na Eigen
image: 2020-08-03/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-08-03/img001.png)
{: refdef}


Metaprogramação estática utilizando templates em C++ é geralmente motivo de
medo para muitos programadores. Templates podem ser muito complexos em geral,
mas existem casos simples e, se você tem interesse em aprender essa ferramenta,
sugiro começar exatamente por esses. Por outro lado, Eigen é uma biblioteca
escrita em C++ que permite trabalhar com arrays e com matrizes e vetores (com operações
relacionadas à algebra linear), e faz uso intensivo de templates.
Nesse post faço uma breve revisão sobre templates e mostro um rápido exemplo
de uso dentro da Eigen, como estudo de caso para entender um pouco mais sobre
o tema.

# Uma breve revisão sobre templates em C++

A grosso modo, templates em C++ são uma forma de você não repetir código para
diversos tipos diferentes. Acredito que o exemplo mais simples seja o seguinte:
Imagine que você defina uma função de soma para dois inteiros. Tempos depois,
você precisa usar a mesma função para dois doubles. Sem templates, você é obrigado
a repetir o código da seguinte forma:

{% highlight c++ linenos %}
int sum(int a, int b) { return a + b; }
double sum(double a, double b) { return a + b; }
{% endhighlight %}

Enquanto que, com template, você pode definir a função apenas uma vez, para um
tipo genérico de nome "T". Ou seja, você pode substituir "T" por "int", por
"float", por "double", etc.

{% highlight c++ linenos %}
template<typename T>
T sum(T a, T b) { return a + b; }
{% endhighlight %}

Além de funções, templates também podem ser utilizados em classes/structs em C++. Nesse
contexto, a ideia é que pode-se gerar classes com as mesmas funcionalidades para
vários tipos diferentes. O exemplo mais clássico é o de uma lista de tipo
genérico. Se não fosse feito com templates, teria-se que implementar uma classe
lista para cada tipo que se quisesse usa-la, copiando e colando praticamente toda
a sua implementação e mudando os tipos nos locais pertinentes.

{% highlight c++ linenos %}
template<typename T, int N>
class MyList
{
public:
    int size() const { return N; }

    // ... (Rest of the implementation goes here) ...

private:
    T data[N];
};

int main()
{
    auto l1 = MyList<int, 5>();
    auto l2 = MyList<double, 8>();

    return 0;
}
{% endhighlight %}

No exemplo, T vai assumir "int" para a variável l1 e "double" para l2. Dessa
forma, l1 representa uma lista com 5 membros do tipo int, enquanto l2 representa
uma lista com 8 membros do tipo double.

Um detalhe adicional é que, se necessário,
é possível especializar um template para um tipo em particular. Dessa forma,
implementa-se de forma diferente a classe ou função para aquele tipo particular.
No exemplo abaixo, a classe "is_int" possui apenas um membro estático: Um
booleano de nome "result". Para os tipos em geral, esse booleano assume o valor
falso. Em específico para o tipo "int", cria-se uma especialização para mudar
esse valor para true. Portanto, o resultado ao executar esse código será "0, 0, 1",
ou seja, os dois primeiros tipos não são "int", mas o terceiro é.

{% highlight c++ linenos %}
#include <iostream>
  
template<typename T>
struct is_int
{
    static const bool result = false;
};

template<>
struct is_int<int>
{
    static const bool result = true;
};

int main()
{
    std::cout
        << is_int<double>::result << ", "
        << is_int<bool>::result << ", "
        << is_int<int>::result << std::endl;

    return 0;
}
{% endhighlight %}

A especialização de templates da um poder muito grande para a ferramenta, pois
além de ser utilizada como uma decisão (como no exemplo acima), pode também ser
usada para gerar uma estrutura de repetição, por meio de uma recursão. Um exemplo
clássico, encontrado tanto em livros quanto pela internet, é o da implementação
de fatorial em tempo de compilação (As vezes o pessoal também mostra o exemplo da soma
dos números da sequencia Fibonacci, que é muito semelhante):

{% highlight c++ linenos %}
#include <iostream>
  
template<int N>
struct factorial
{
    static const int result = N * factorial<N - 1>::result;
};

template<>
struct factorial<1>
{
    static const int result = 1;
};

int main()
{
    std::cout << factorial<5>::result << std::endl;
    return 0;
}
{% endhighlight %}

Mas pra que essa complicação inteira? Que vantagem isso tem sobre uma simples
implementação de fatorial comum? A questão aqui é que todo esse código será 
resolvido em tempo de compilação. Isso significa que, pra gerar o código
para a linha "std::cout << factorial<5>::result << std::endl;", o compilador
terá que descobrir qual o valor de "factorial<5>::result". Dessa forma, em
tempo de execução, não será necessário calcular o fatorial de 5. Será
simplesmente escrito 120 na tela.

Apenas para deixar o exemplo completo, é
comum mostrar a parcela do código gerado (assembly), apontando que o valor
120 foi, de fato, computado pelo compilador. O código final possui desempenho
(de tempo de execução) muito melhor, do que se tivesse implementado o fatorial
da forma convencional.

{% highlight c++ linenos %}
main:
.LFB1522:
        .cfi_startproc
        endbr64
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        movl    $120, %esi   <=======================================
        leaq    _ZSt4cout(%rip), %rdi
        call    _ZNSolsEi@PLT
{% endhighlight %}


# Estudo de caso: Alocação de arrays na Eigen

Já comentei sobre a biblioteca Eigen em vários posts desse blog. Tenho usado
ela bastante em projetos pessoais que precisem de uma estrutura de vetores com
boa performance e, eventualmente, funções auxiliares ligadas à algebra linear.
A Eigen é uma biblioteca header-only, de forma que não é necessário fazer nenhum
tipo de linkagem para utiliza-la. Por isso, ela faz uso intenso de
metaprogramação estática por meio de templates em C++.

Apenas para mostrar brevemente um estudo de caso de uso de templates em C++,
resolvi "garimpar" no código fonte da Eigen, a fim de descobrir como ela faz
para determinar como será a alocação dos membros dos arrays. Pelo que vi, a
solução envolve verificar se o usuário determinou o número de linhas e colunas
do array em tempo de compilação e, com essa informação, decidir como será feita
a alocação desses dados.

Apenas relembrando, de forma geral, assume-se que alocações dinâmicas (na heap)
são muito mais lentas que as alocações da stack. Em outras palavras, fazer
"new" é mais lento do que simplesmente definir variáveis locais. Por isso,
fazer essa distinção é importante.

Para criar um Array na Eigen, é necessário prover uma série de parametros de
template, como ilustrado no código abaixo. Os dois parâmetros importantes aqui
são o _Rows e o _Cols, que representam, respectivamente, o número de linhas e
de colunas. Caso o programador não saiba esses valores em tempo de compilação,
utiliza-se o valor "Dynamic".

{% highlight c++ linenos %}
template<typename _Scalar, int _Rows, int _Cols, int _Options, int _MaxRows, int _MaxCols>
class Eigen::Array< _Scalar, _Rows, _Cols, _Options, _MaxRows, _MaxCols >
{% endhighlight %}

Exemplo de uso:

{% highlight c++ linenos %}
Eigen::Array<float, Dynamic, 1> a1; // Array de N linhas e 1 coluna de floats
Eigan::Array<double, 3, 3> a2; // Array de 3 linhas e 3 colunas de doubles
{% endhighlight %}

Finalmente chegamos à parte interessante. Para decidir como os dados serão
alocados, a Eigen possui especializações de template para cada
situação específica. Caso se saiba tanto _Rows quanto _Cols, a forma geral do
template assume a implementação abaixo, incentivando a alocação dos dados na
pilha:

{% highlight c++ linenos %}
template<typename T, int Size, int _Rows, int _Cols, int _Options> class DenseStorage
{
    internal::plain_array<T,Size,_Options> m_data;
{% endhighlight %}

Onde (dadas as devidas simplificações), plain_array é o seguinte:

{% highlight c++ linenos %}
struct plain_array
{
  T array[Size];
{% endhighlight %}

Por outro lado, para a especialização onde tanto _Rows quanto _Cols é "Dynamic",
a estrutura de dados muda para um ponteiro cru, que será alocado dinamicamente,
ou seja, em tempo de execução:

{% highlight c++ linenos %}
template<typename T, int _Options> class DenseStorage<T, Dynamic, Dynamic, Dynamic, _Options>
{
    T *m_data;
    Index m_rows;
    Index m_cols;
{% endhighlight %}

Esse é um caso prático do uso de templates em C++, que mostra como aquelas funcionalidades
aparentemente sem uso tem, de fato, grande importância prática no dia-a-dia.
Por outro lado, a não ser que você esteja implementando uma biblioteca muito
genérica como é o caso da Eigen, talvez seja difícil você utilizar templates
dessa forma. De qualquer modo, reforço a importância de entender as ferramentas
que estamos usando para, se necessário, debugar problemas, implementar novas
features ou extrair o máximo de performance.

Até a próxima!
