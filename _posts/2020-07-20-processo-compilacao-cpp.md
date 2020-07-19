---
layout: post
title: Processo de compilação e linkagem em C++
image: 2020-07-20/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/img001.png)
{: refdef}


Compilar é basicamente **traduzir** um texto que está escrito em uma linguagem
(nesse caso a **linguagem C++**) para outra linguagem: a **linguagem de máquina**,
para que o computador possa entender e executar posteriormente.

Entender o processo de compilação pode ser algo complexo. Tentei resumir
nesse post de forma pragmática, dividindo em apenas duas partes: A
**compilação** e a **linkagem**.

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/compiling.png)
{: refdef}
[https://xkcd.com/303/](https://xkcd.com/303/)

Na etapa de compilação, cada arquivo com extensão *.cpp* é pré-processado, e
combinado com vários arquivos *.h* (ou *.hh* ou *.hpp*, chamados **Header files**),
pra formar unidades de compilação (**compilation units**). Nessa etapa, o compilador
resolve as macros ("#define", "#include", etc...).

Inclusive, você sabia que o comando **#include "a.h"** vai basicamente copiar
todo o conteúdo do arquivo *a.h* e colar no local do include? É basicamente
um *find-and-replace* ;)

Cada "unidade de compilação" é, então, processada pelo compilador que gera um
arquivo de saída que chamamos de arquivo "objeto". São os arquivos de extensão
*.obj* (no Windows) ou *.o* (no Linux).

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/cpp_plus_h.png)
{: refdef}

Na etapa de linkagem, o compilador pega todos esses arquivos objeto e junta eles
pra formar um arquivo executável (*.exe*, de "Executable" ou *.dll*,
de "Dynamic Linked Library", por exemplo). Nesse momento,
ele monta uma tabela com todos os simbolos que ele encontrar no programa.
Por exemplo, pra cada declaração de função deve existir pelo menos uma definição
daquela função em algum lugar.

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/obj_to_exe.png)
{: refdef}

É também comum combinar vários arquivos objetos em um arquivo *.lib* (no Windows) ou *.a* (no Linux).
Essa é uma forma comum de compartilhar bibliotecas
**estáticas** (**static linked libraries**).

Na verdade, pra ser completamente
correto, arquivos **.lib** no Windows também são usados como passo intermediário
para bibliotecas dinâmicas - Dynamic Libraries ou Shared Libraries, mas não
vou entrar em detalhes sobre isso nesse post...

É isso! Claro que eu simplifiquei tudo ao extremo aqui,
pra você poder ter a visão geral da coisa... Nas próximas sessões, vou mostrar
dois exemplos práticos. O primeiro é utilizando a Eigen, que é uma biblioteca
"header-only", ou seja, **não é necessário linkar com a biblioteca**. A segunda é
com a biblioteca Intel MKL, que eu [já usei ela aqui pelo blog](https://tarcisiofischer.github.io/2020-03-09/resolvendo-a-equacao-da-difusao-em-python-parte-5).
Nessa, vou usar a PARDISO como **exemplo de linkagem**.


# Exemplo 1: Header-only library

O código abaixo (baseado [nos exemplos da documentação da Eigen](https://eigen.tuxfamily.org/dox/group__TutorialArrayClass.html)) cria uma matriz
2x2, populando alguns dados e, depois, escreve ela na tela do terminal.

{% highlight C++ linenos %}
#include <Eigen/Dense>
#include <iostream>

int main()
{
    Eigen::ArrayXXf m(2, 2);
    m << 1.0, 2.0,
        3.0, 4.0;
    std::cout << m << std::endl;
}
{% endhighlight %}

Abrindo um novo projeto no Visual Studio, copiando o código e compilando sem
fazer nenhuma configuração, gera o erro a seguir:

{% highlight text linenos %}
E1696   cannot open source file "Eigen/Dense"
{% endhighlight %}

Por não encontrar esse arquivo, o compilador começa a gerar vários outros erros,
que acabam não fazendo sentido para o leitor (vide abaixo). Isso acontece por
que o compilador tenta continuar a geração de código, mesmo depois do primeiro
erro. O que ocorre é que, sem as declarações e definições presentes no arquivo
"Eigen/Dense", muitos outros nomes e tipos são desconhecidos e, portanto, os
erros vão se acumulando...

{% highlight text linenos %}
E1696   cannot open source file "Eigen/Dense"
E0276   name followed by '::' must be a class or namespace name
E0065   expected a ';'
E0020   identifier "m" is undefined
{% endhighlight %}

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/sweating_meme.gif)
{: refdef}

Para corrigir esse problema, precisamos simplesmente adicionar o caminho
onde da pasta "Eigen", nos "Include Directories", ou seja, os diretórios que
serão incluidos no processo de compilação. Pode paracer muito básico, mas eu
sei que muita gente ainda tem problema com esse tipo de coisa. Por isso,
inclusive, deixo abaixo a configuração para esse projeto de exemplo. No 
Visual Studio, essa tela fica em "Debug> (project_name) Debug Properties", e
na janela de propriedades, "Configuration Properties > C/C++ > General", conforme
print abaixo.

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/visual_studio_include.png)
{: refdef}


# Exemplo 2: Linkagem estática

Para o segundo exemplo, usei o mesmo código do [post passado](https://tarcisiofischer.github.io/2020-03-09/resolvendo-a-equacao-da-difusao-em-python-parte-5), onde usei a PARDISO
como solver linear para resolver o problema de difusão bidimensional. Copiando
e colando o código no Visual Studio, sem fazer mais nenhuma configuração, resulta
no seguinte erro:

{% highlight text linenos %}
C1083   Cannot open include file: 'mkl_pardiso.h': No such file or directory
{% endhighlight %}

Como já vimos, esse não é um erro de linkagem, mas de compilação! Pra resolver,
foi só configurar o "Include Directories" para o diretório dos header files
onde instalei a MKL. Tentando a compilação novamente, recebo o seguinte erro:

{% highlight text linenos %}
LNK2019 unresolved external symbol PARDISO referenced in function "void __cdecl pardiso_solve(double const *,int const *,int const *,double const *,double *,int)" (?pardiso_solve@@YAXPEBNPEBH10PEANH@Z)
{% endhighlight %}

Eu sei que o erro é feio, mas antes de "não ler" a mensagem, perceba pelo menos
o código dela: Começa com "LNK2019". LNK vem de "Linker", que já me dá o indício
de que é um erro de linkagem. A mensagem, claro, também vai nesse sentido e diz
que "não foi possível resolver o simbolo externo PARDISO, usado na função ...",
seguido do [name-mangling](https://en.wikipedia.org/wiki/Name_mangling) da função
(Não vou entrar em detalhes sobre isso).

Bom, pra resolver é só adicionar a biblioteca da mkl (arquivo *.lib*) nas
dependencias do Visual Studio:

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/linker_add_dependencies.png)
{: refdef}

Depois disso é só recompilar e... ... Erro? Opa!?

{% highlight text linenos %}
LNK1104 cannot open file 'mkl_rt.lib'
{% endhighlight %}

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/thinking_meme.png)
{: refdef}

O problema é que, apesar de termos dito que existe uma biblioteca que é necessário
linkar (mkl_rt.lib), não falamos ONDE ela está. O compilador vai procurar em algumas
pastas de sistema, mas, no meu caso, esse arquivo está numa pasta específica
onde instalei a MKL. Então, pra configurar, preciso avisar o Visual Studio:

{:refdef: style="text-align: center;"}
![](/images/2020-07-20/linker_add_directories.png)
{: refdef}

Feito! Agora consigo compilar o programa sem problemas e gerar um executável! :)


# Conclusão e discussão

Perceba a diferença no estilo dos erros de compilação e linkagem.
Um erro de compilação
aparece até mesmo quando você está tentando compilar um arquivo, individualmente.
Quando o erro só acontece quando você está tentando compilar o programa como
um todo, pra gerar um executável, muito provavelmente você está tendo um erro de
linkagem. E é importante saber isso pra saber que ação você vai tomar a seguir.

Claro que existem outros problemas que podem ocorrer, por exemplo, no momento
de executar o programa (os mais comuns são "segmentation fault", "could not find
dll", e a aplicação simplesmente "sumindo" sem mensagem). Mas vou parar o post
por aqui, quem sabe no futuro volto a comentar sobre outras possibilidades.

Além disso, ferramentas como CMake podem auxiliar em todo esse processo,
automatizando as configurações do processo de compilação. E gerenciadores de
pacote, como o Conda ou o Conan, ajudam a manter compatibilidade entre as
bibliotecas.

Até a próxima!
