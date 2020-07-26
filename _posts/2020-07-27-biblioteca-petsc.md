---
layout: post
title: Solução de sistemas não lineares utilizando a PETSc
image: 2020-07-27/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-07-27/img001.png)
{: refdef}


[PETSc (Portable, Extensible Toolkit for Scientific Computation)](https://www.mcs.anl.gov/petsc/) é uma biblioteca
escrita em C/FORTRAN, que pode ser usada em programas que envolvem a solução
de equações diferenciais parciais. [FEniCS](https://fenicsproject.org/)
, [Firedrake](https://www.firedrakeproject.org/) e [Petsc4FOAM](https://develop.openfoam.com/modules/external-solver)
(para [OpenFOAM](https://www.openfoam.org/)), são algumas das
bibliotecas que usam ou possuem alguma forma de integração com a PETSc. Nesse
post, mostro um passo-a-passo introdutório para a bibloteca.


# Problema de exemplo

Pra deixa a parte matemática trivial, o objetivo é encontrar $x$ e $y$
que satisfazem o sistema de equações não linear a seguir.

$$
\left\{\begin{matrix}
x^2 + y^2 = 20\\ 
x - y = -2
\end{matrix}\right.
$$

Graficamente, é fácil observar as possíveis soluções:

{:refdef: style="text-align: center;"}
![Exemplo - Representação do problema](/images/2020-07-27/problem_representation.png)
{: refdef}


# Configuração da PETSc

Como a PETSc é uma biblioteca escrita em C, a API dela pode ser um pouco
estranho pra quem não está acostumado. Existem bindings pra Python ([PETSc4Py](https://bitbucket.org/petsc/petsc4py/)),
mas nesse post vou fazer a configuração inteira em C puro. O código completo
[está no github](https://github.com/tarcisiofischer/simple_petsc_example),
mas acho que vale a pena passar por alguns pontos pra fazer comentários.

Por exemplo, as estruturas de matrizes e vetores da PETSc são todas baseadas
em ponteiros, com typedefs pra "esconder" a implementação real. Uma estrutura
de matriz (Mat) é, na realidade, um typedef pra um ponteiro pra estrutura
real _p_Mat:

{% highlight c++ linenos %}
typedef struct _p_Mat* Mat;
{% endhighlight %}

Todas as estruturas de dados possuem algum tipo de função auxiliar para fazer
sua inicialização.


## Inicialização das estruturas de dados

É necessário inicializar a PETSc com PetscInitialize ou 
PetscInitializeNoArguments. A inicialização deve ser feita apenas uma vez.
Nesse exemplo, estou utilizando o PetscInitializeNoArguments, pois fiz toda a
configuração manualmente. No fim do programa é necessário chamar PetscFinalize,
para que a PETSc limpe todas as suas estruturas internas.

Como a API da biblioteca é em C, e portanto não existem exceções, os
desenvolvedores usaram um padrão onde todas as chamadas de função retornam um
código de erro. Como estou usando C++, criei uma macro que verifica se a função
resultou em um erro e transforma em uma exceção, que me diz qual
linha aconteceu o erro. Isso pode ser útil pra identificar rapidamente a fonte
de problemas de implementação:

{% highlight c++ linenos %}
#define PETSC_CHECK(X) \
    { auto _err = X; if (_err) throw std::runtime_error("PETSc error at line " + std::to_string(__LINE__)); }
{% endhighlight %}


## Sistema não linear

A PETSc tem várias configurações de sistemas não lineares. Nesse exemplo simples,
estou usando o método de Newton com Line Search (NEWTONLS). Dentro desse método,
ainda é possível configurar como será feito o Line Search, por exemplo, por
backtracking. Nesse exemplo estou usando o método "basic", que na prática não
faz nenhum tipo de line search.

{% highlight c++ linenos %}
    SNES snes;
    PETSC_CHECK(SNESCreate(PETSC_COMM_WORLD, &snes));
    PETSC_CHECK(SNESSetType(snes, SNESNEWTONLS));

    SNESLineSearch linesearch;
    PETSC_CHECK(SNESGetLineSearch(snes, &linesearch));
    PETSC_CHECK(SNESLineSearchSetType(linesearch, SNESLINESEARCHBASIC));
    // ...
    PETSC_CHECK(SNESSetFunction(snes, r, residual_function, NULL));

{% endhighlight %}

E, para a solução pelo método de newton, é necessário prover
uma função que calcula a matriz jacobiana (matriz tangente). Também é
possível também utilizar a implementação por diferenças finitas que a PETSc
provê, mas não farei isso nesse post.

{% highlight c++ linenos %}
    Mat J;
    PETSC_CHECK(MatCreate(PETSC_COMM_WORLD, &J));
    PETSC_CHECK(MatSetSizes(J, PETSC_DECIDE, PETSC_DECIDE, 2, 2));
    PETSC_CHECK(MatSetFromOptions(J));
    PETSC_CHECK(MatSetUp(J));
    PETSC_CHECK(SNESSetJacobian(snes, J, J, jacobian_function, NULL));
{% endhighlight %}

{% highlight c++ linenos %}
PetscErrorCode jacobian_function(SNES snes, Vec x, Mat jac, Mat B, void *dummy)
{
    // ...
    PetscScalar A[4] = {
        2. * xx[0], 2. * xx[1],
        1.0, -1.0
    };
    // ...
}
{% endhighlight %}


## Sistema linear

Para cada iteração do método de Newton, é necessário resolver um sistema linear.
Resolvi utilizar uma fatoração LU. A PETSc possui solvers lineares baseados em
subespaços de Krylov - são os "KSP" (Krylov Subspace Methods, por exemplo, o
GMRES). Os métodos fora desse contexto são vistos como "pré-condicionadores",
e ficam na família "PC" (PreConditioners). Então, pra usar a faturação LU, é
necessário configurar o KSP como "KSPPREONLY", ou seja, será apenas utilizado
o PC. A configuração final fica da seguinte forma:

{% highlight c++ linenos %}
    KSP ksp;
    PETSC_CHECK(SNESGetKSP(snes, &ksp));
    PC pc;
    PETSC_CHECK(KSPGetPC(ksp, &pc));
    PETSC_CHECK(KSPSetType(ksp, KSPPREONLY));
    PETSC_CHECK(PCSetType(pc, PCLU));
    PETSC_CHECK(SNESSetFromOptions(snes));
{% endhighlight %}


# Solução, resultados e discussão

Finalmente, para resolver o problema, basta chamar a função SNESSolve. Nesse
exemplo, eu adicionei um monitor no processo de solução, pra salvar as soluções
intermediarias e, posteriormente, retornar elas para o Python e usar a matplotlib
para mostrar os "caminhos" que o método percorreu.

O resultado para 4 chutes iniciais é mostrado abaixo. Cada chute é representado
por uma cor, de forma que o conjunto de linhas de uma determinada cor forma
o caminho que o método percorreu para chegar na aproximação da solução.
Para os chutes escolhidos, as soluções foram satisfatórias, e a
PETSc foi capaz de encontrar as duas raizes.

{:refdef: style="text-align: center;"}
![Resultados](/images/2020-07-27/results.png)
{: refdef}

Claro que esse post é totalmente introdutório, e a PETSc é uma biblioteca
muito mais poderosa que está sendo subutilizada nesse exemplo. Para quem, tiver
interesse, sugiro olhar a documentação oficial, em especial os exemplos e
o manual em PDF para maiores informações.

Até a próxima!
