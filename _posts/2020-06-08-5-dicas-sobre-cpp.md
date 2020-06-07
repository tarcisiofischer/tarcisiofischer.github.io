---
layout: post
title: 5 dicas sobre C++
image: 2020-06-08/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-06-08/img001.png)
{: refdef}

Apesar de eu usar majoritariamente Python nos meus posts, sempre gostei muito da
linguagem C++. Não que outras linguagens sejam piores, mas C++ ainda é uma das
linguagens que me sinto mais confortável, pela forma como as coisas podem ser
feitas, no sentido de poder "abrir o capô" e "mexer no motor" quando necessário.
Falo de aspectos como gerenciamento de memória e programação de baixo nível.
Nesse post, mostro 5 dicas práticas para quem está começando com C++.


# 1. Prefira C++ ao invés de C

Eu não quero começar uma "flame war". Estou tentando ser bastante prático nessa
lista. Preparei 3 argumentos sobre por que utilizar C++ ao invés de C:

1. **Referências**: É meu argumento preferido, por isso o primeiro.
Basicamente, são uma forma elegante de mandar dados para funções
sem copia-los. Melhor ainda quando usado em conjunto com *const*. A alternativa,
em C, é mandar um ponteiro pro dado. Porém, isso gera uma complexidade
desnecessária pelo código.

2. **Exceptions**: São uma forma bem coerente de criar fluxos em situações
inesperadas. Em C, uma forma comum de simular exceções é retornando um
"código de erro", que é, então,
verificado para cada chamada. Você acaba perdendo a possibilidade de retornar
algum valor significativo, e acaba tendo um monte de código extra pra tratar
cada código de erro.

3. **Classes**: Programação Orientada a Objetos talvez seja "overrated". Mesmo
assim, decidi colocar esse ponto aqui nessa lista, pois existem bastantes
situações onde classes e objetos podem realmente ajudar na organização de um
sistema. É possível "simular" métodos em C, fazendo funções
cujo primeiro parâmetro é um ponteiro para uma estrutura conhecida. Porém, ter
a feature direto na linguagem com certeza deixa o código mais limpo.


# 2. Use as versões mais atuais da linguagem

C++ é uma linguagem que evoluiu durante os anos. Existem versões da linguagem,
que introduzem funcionalidades bastantes úteis. Seguem alguns exemplos, a partir
de C++11:

**Smart pointers**

Ao invés de ter que gerenciar memória com *new* e *delete*, os *smart pointers*
fazem esse gerenciamento, dependendo do contexto de uso, por meio de *unique_ptr*
e *shared_ptr*, por exemplo.

**Inferência de tipo**

A palavra-chave *auto* pode ser usada e, a partir do valor à direita,
o compilador consegue inferir o tipo do dado:

{% highlight c++ linenos %}
auto J = (grad_N.transpose().matrix() * element_points.matrix()).eval();
{% endhighlight %}

**Lambda functions**

Assim como em Python, é possível declarar uma pequena função, local, e atribui-la
a uma variável:

{% highlight c++ linenos %}
auto from_ij_to_flat_index = [&nx](int i, int j) -> int {
    return (nx + 1) * i + j;
};
{% endhighlight %}

**Várias estruturas úteis na biblioteca padrão**

Tais como *std::optional*, *std::variant*, *std::any*, entre outras,
que facilitam e dão mais semântica ao código.


# 3. Integração com Python com Pybind11

Já comentei várias vezes nesse blog sobre as vantagens de fazer esse tipo de
integração. Python possui várias vantagens e, combinada com C++ para suprir
o déficit de performance, é uma das formas de tirar o melhor dos dois mundos.
Uma característica que acredito nunca ter comentado, é que o oposto também
é possível: Chamar código Python a partir do C++, inclusive para utilizar
bibliotecas Python externas de dentro do C++.

Não que essa seja a única nem a melhor forma de programar. Porém, na minha
opinião, é uma forma conveniente. Algumas alternativas incluem utilizar a
linguagem Julia (julia-lang), misturar Rust com Python ou mesmo FORTRAN moderno.
Não existe resposta única, e cada linguagem possui vantagens
e desvantagens. O intuito de mistura-las é tentar obter o melhor de cada mundo,
tentando minimizar o trabalho para tal.


# 4. Eigen para matrizes e vetores

Arrays crus estão longe de ser boa ideia, para guardar as matrizes e vetores,
na solução de problemas que envolve mexer com os mesmos. Isso por que são
estruturas muito "cruas", que acabam dificultando mais do que ajudando.
*std::vector* e *std::array* até são alternativas válidas, porém, não possuem
muitas funções prontas para operações entre matrizes ou entre vetores. Além
disso, podem ser lentas em algumas ocasiões.

A Eigen é uma biblioteca que possui estruturas de matrizes e vetores, e
implementa operações entre eles de forma eficiente. A biblioteca possui até
mesmo solvers lineares e, no pior dos casos, te deixa acessar os ponteiros
crus das estruturas internas (o que é bom, por exemplo, para interfacear
com solvers em C ou FORTRAN).

É claro que existem alternativas. Eigen não é a única biblioteca que possui
tais estruturas. Porém, é uma biblioteca aparentemente bastante madura e
utilizada pela comunidade, o que traz certa robustez. Além disso, sua
documentação é bastante extensa, e é relativamente simples de usar. Uma última
vantagem é que possui fácil interoperabilidade com numpy array por meio da
Pybind11.


# 5. Use boas ferramentas

As IDEs (Integrated Development Environment) são programas que incluem um
conjunto de funcionalidades que auxiliam no desenvolvimento em determinada
linguagem. Para C++, no escopo do sistema operacional Windows, temos o
Visual Studio, da Microsoft. Na minha opinião, é, de longe, a melhor IDE
para C++, para esse sistema operacional. E o melhor de tudo: atualmente pode
ser baixado gratuitamente na versão "Visual Studio Community".

Outra sugestão é sobre os compiladores. Além do compilador da Microsoft,
existem o compilador da Intel, o GCC (GNU C Compiled) e o Clang, por exemplo.
Cada um desses compiladores possui algumas características únicas. É legal
acompanhar essas diferentes implementações, para tentar tirar proveito de
suas características. Por exemplo, o CLANG é bastante conhecido por
suas excelentes mensagens de erro de compilação.

Essas foram apenas algumas breves dicas. Para mais informações, sugiro também
o [excelente material](https://github.com/cppbrasil/material-de-aprendizado) do
pessoal do grupo ["C/C++ Brasil" no Telegram](https://t.me/cppbrazil).

Até a próxima!
