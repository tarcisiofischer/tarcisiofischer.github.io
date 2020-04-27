---
layout: post
title: Discretização da equação de Helmholtz em elementos finitos
---

Usarei esse projeto para,
em posts futuros, começar a explorar diversos aspectos computacionais
relacionados à implementação do mesmo. Já aviso que sacrifiquei o rigor matemático
para relaxar um pouco o post, na expectativa de torná-lo mais legível àqueles
com pouca familiaridade no tema. A ideia é que, quem conhece o método, pode
simplesmente passar o olho e ter uma visão geral do que vai ser implementado
em posts futuros, enquanto que, aqueles que estão aprendendo ou desconhecem,
podem fazer a leitura sem se preocupar com detalhes.

{:refdef: style="text-align: center;"}
![](/images/2020-04-27/img001.png)
{: refdef}


# Forma fraca

É necessário iniciar de algum lugar. Aqui, vou pular toda a motivação do
uso da equação, a derivação matemática dos aspectos físicos, os balanços
conservativos, a modelagem constitutiva, as hipóteses de simplificação, as
transformações de domínio temporal, etc. E vou assumir que queremos simplesmente
resolver a equação diferencial abaixo. Caso o leitor não tenha familiaridade
com a nomenclatura matemática, sugiro procurar sobre algebra vetorial e tensorial.

$$
\frac{\omega^{2}}{c^{2}(\mathbf{x})} p(\mathbf{x}, \omega)
- i \omega \eta(\mathbf{x}) p(\mathbf{x}, \omega)
+ \nabla^{2} p(\mathbf{x}, \omega)
+ s(\mathbf{x}, \omega) = 0 \quad \forall \mathbf{x} \in \Omega
$$

Queremos encontrar um campo escalar $p(\mathbf{x}, \omega)$ que, dados campos
escalares conhecidos de $\eta$, $c$ e $s$, e valor conhecido de $\omega$, satisfaça a
equação acima em todo o domínio de interesse $\Omega$. Ao invés de tentar resolver analiticamente, vamos fazer isso
buscando uma aproximação da solução, utilizando o método dos elementos finitos.

{:refdef: style="text-align: center;"}
![](/images/2020-04-27/img002.png)
{: refdef}

Se a equação possui valor zero para todo $\mathbf{x}$ no domínio de interesse, então, se
multiplicarmos a mesma por uma função arbitrária $h$, continuará zero.
Essa função $h$ é normalmente chamada de "função de teste" ou "função peso".
Como isso é verdade para qualquer parte do domínio, será também para
o domínio como um todo, portanto, podemos integrar, obtendo a equação abaixo.
Essa explicação poderia ser bem melhor trabalhada, utilizando argumentações
relacionadas ao método dos resíduos ponderados, e tendo um cuidado maior ao
explicar as propriedades que a função $h$ deve carregar. Porém, a titulo de
simplificar a leitura do leitor, sigo sem maiores rigores. Daqui em diante,
leia "para qualquer $h$" como "para qualquer $h$ contendo as propriedades
necessárias".

$$
\int_{\Omega} \left({\frac{\omega^2}{c^2\left( \mathbf{x}\right)}}
p\left(  \mathbf{x},\omega\right)  -i\omega\eta\left(  \mathbf{x}\right)
p\left(  \mathbf{x},\omega\right)  +\nabla^{2}p\left(  \mathbf{x}%
,\omega\right)  +s\left(  \mathbf{x},\omega\right)  \right)  h\ d\Omega = 0 \quad \forall h
$$

A partir daí, trabalhamos 
para aparecerem as condições de contorno e para remover a dependência do laplaciano.
Para pular o rigor matemático, tomemos como verdade a seguinte relação:

$$
\int_{\Omega}\nabla^{2}p\ h\ d\Omega=\int_{\Gamma}\left(  \nabla ph\right)
\ \cdot\mathbf{n}d\Gamma-\int_{\Omega}\nabla p\cdot\nabla h\ d\Omega
$$

Onde $\Gamma$ representa o domínio no contorno da região de interesse.
Por hipótese, considerarei que a integral no contorno é sempre nula, na solução
deste problema em específico (isso não é verdade em geral). Segue, então,
a forma fraca do problema:

$$
\int_{\Omega}\frac{\omega^{2}}{c^{2}\left(  \mathbf{x}\right)  }p \left(\mathbf{x}, \omega\right)h\ d\Omega
- \int_{\Omega} i \omega \eta \left(\mathbf{x}\right)  p\left(  \mathbf{x},\omega\right)  h\ d\Omega
- \int_{\Omega} \nabla p \cdot \nabla h\ d\Omega
+ \int_{\Omega} s \left(\mathbf{x}, \omega \right) h\ d\Omega
= 0 \quad \forall h
$$


# Separação de variáveis

A partir desse ponto, buscamos um campo escalar $p$ que satisfaça a equação
em sua forma fraca. A priori, parece que pioramos o problema, pois, além de termos uma equação
diferencial, temos também integrações no domínio. Essa etapa ataca essa questão,
montando um sistema de equações linear. Porém, introduz um novo problema,
que é a de determinar uma função $N$, como veremos a seguir.

Separamos o campo escalar $p$ e a função de teste $h$ em duas partes:
$N$, contendo apenas as variações, com magnitude unitária, e $P$, que provê as
magnitudes. Sendo assim, podemos reescrever esses campos no formato abaixo.

$$
p(\mathbf{x}, \omega) = N(\mathbf{x}) P(\omega)
$$

$$
\nabla p(\mathbf{x}, \omega) = \nabla N(\mathbf{x}) P(\omega) = B(\mathbf{x}) P(\omega)
$$

Fazemos o mesmo para a função arbitrária $h$. Note que essa escolha poderia ser
feita de outro modo, visto que não existe necessariamente relação entre $h$ e
$p$.

$$
h(\mathbf{x}, \omega) = N(\mathbf{x}) H(\omega)
$$

Para encurtar o post, mostro diretamente a equação resultante.
Aqui, o detalhe importante é que $P$
e $H$ podem ser extraídos da integral, agora que independem de $\mathbf{x}$ e,
como assumimos que conhecemos a função $N$ e seu gradiente $B$, não há mais
incógnitas dentro das integrais.

$$
\Big{\{} \omega^{2} \int_{\Omega}\mu(x) N(x) N(x) \ d\Omega P(\omega) +\\
-i \omega\int_{\Omega} \eta(x) N(x) N(x) \ d \Omega P(\omega) +\\
- \int_{\Omega} B(x) B(x) \ d \Omega P(\omega) +\\
\int_{\Omega} s(x, \omega) N(x) \ d \Omega \Big{\}} H(\omega) = 0
$$

Ademais, se a equação acima é válida para
qualquer $H$, então assume-se que a parte entre chaves é necessariamente igual
a zero, e a única incógnita da equação agora, é a função $P$. Coletando termos
e reescrevendo a equação, ficamos com:

$$
\underbrace{\Big{\{} \omega^{2} \int_{\Omega}\mu(x) N(x) N(x) \ d\Omega -i
\omega\int_{\Omega} \eta(x) N(x) N(x) \ d \Omega
- \int_{\Omega} B(x) B(x) \ d
\Omega\Big{\}}}_{K} P(\omega) +\\
 = \underbrace{-\int_{\Omega} s(x, \omega) N(x) \ d \Omega}_{f}
$$


# Funções de forma

Para que a equação anterior se torne um sistema linear de equações, precisamos
ser capazes de computar $K$ e $f$, e discretizar $P$.

Teóricamente, a função $N$ poderia ser qualquer função (dado um conjunto
de propriedades, tais como ser contínua e diferenciável). Porém, a fim de
facilitar encontrar a solução do problema, escolhe-se uma função que pode ser
repartida em funções menores. Cada função independe de todas as outras,
mas a composição aditiva delas provê a "forma" da solução final. Não posso deixar de
mostrar a imagem que é um exemplo clássico, para o caso unidimensional. A
situação é análoga para casos em outras dimensões.

{:refdef: style="text-align: center;"}
![](/images/2020-04-27/img003.png)
{: refdef}

Para o projeto, escolho um domínio bidimensional, elementos quadriláteros, e
o uso de funções lineares para representar a forma como a solução varia dentro
de cada elemento (chamados "elementos quadriláteros lineares"). É possível 
escrever a função $N$ da seguinte maneira:

$$
N(x, y) = \sum_{e} N_e(x, y)
$$

Como cada elemento pode ser computado individualmente, fazemos uma transformação
espacial para escrever, em um domínio padrão $(r, s)$, uma equação genérica,
que compõe o mesmo em um somatório de equações lineares para cada nó.

$$
N_e(r, s) = \sum_0^3 N_i(r, s)
$$

Onde

$$
N_{0}(r, s) = \frac{(1 - r) (1 - s)}{4}\\
N_{1}(r, s) = \frac{(1 + r) (1 - s)}{4}\\
N_{2}(r, s) = \frac{(1 + r) (1 + s)}{4}\\
N_{3}(r, s) = \frac{(1 - r) (1 + s)}{4}%
$$

Enquanto seus gradientes ($\nabla N = B$) são facilmente computados como:

$$
\nabla N_{0} =
\begin{bmatrix}
\frac{-(1. - s)}{4}\\
\frac{-(1. - r)}{4}%
\end{bmatrix}
\\
\nabla N_{1} =
\begin{bmatrix}
\frac{+(1. - s)}{4}\\
\frac{-(1. + r)}{4}%
\end{bmatrix}
\\
\nabla N_{2} =
\begin{bmatrix}
\frac{+(1. + s)}{4}\\
\frac{+(1. + r)}{4}%
\end{bmatrix}
\\
\nabla N_{3} =
\begin{bmatrix}
\frac{-(1. + s)}{4}\\
\frac{+(1. - r)}{4}%
\end{bmatrix}
$$

Com isso, é possível determinar a matriz $K$ como uma composição de várias matrizes menores.
Tal operação é conhecida como "assembly" (montagem) da matriz "global" $K$,
a partir de matrizes "elementares" $K_e$. Matematicamente (E note a troca de
da sintaxe tensorial para uma sintaxe matricial) denota-se:

$$
K = \bigcup_{e} K_{e} =
\bigcup_{e} \Big{\{} \omega^{2} \int_{\Omega_{e}} \mu_{e} N_{e}^{T} N_{e} \ d\Omega
- i \omega\int_{\Omega_{e}} \eta_{e} N_{e}^{T} N_{e} \ d \Omega
- \int_{\Omega_{e}} B_{e}^{T} B_{e} \ d
\Omega\Big{\}}
$$

$$
f = \bigcup_{e} f_{e} = \bigcup_{e} \Big{\{} \int_{\Omega_{e}} N_{e}^{T} N_{e}
\ d \Omega S_{e}(\omega) \Big{\}}
$$

Enquanto isso, $P$ se torna um vetor de valores. Cada valor corresponde à
magnitude da função em cada um dos nós. Aumentar o número de elementos (e portanto
o número de nós), é uma forma de melhorar a aproximação da solução do problema.

Novamente, não posso deixar de mostrar outra imagem clássica, dessa vez
relacionada à explicação do assembly da matriz global $K$, abaixo. Na imagem, as
componentes de cada elemento são posicionadas na matriz global numa composição
aditiva de termos. Aproveito para chamar atenção, para a esparsidade da matriz
global. Fato que auxilia na solução de grandes sistemas lineares.

{:refdef: style="text-align: center;"}
![](/images/2020-04-27/img004.png)
{: refdef}


# Integração nos elementos

Por fim, existem várias formas de computar as integrais (agora por elemento).
Por exemplo, poderiam-se computa-las analiticamente, visto que agora todos
os termos são conhecidos, para esse caso. Isso não é sempre verdade, visto
que dependendo da escolha do elemento isso pode ser muito difícil ou até
impossível.

De qualquer forma, é bastante comum utilizar uma quadratura de 
Gauss (Quadratura Gaussiana), como alternativa à integral analitica.
Inclusive várias fontes, sejam livros clássicos
ou páginas da web, trazem os parâmetros ou mesmo o procedimento completo.

$$
\int_{a}^{b} f(k) \ dk \approx\sum_{i=1}^{n} w_{i} f_{i}
$$

$$
K_{e} = \omega\sum_{i=0}^{3} w_{i} \mu_{i} N^{T} N \ dJ\\
-i \omega\sum_{i=0}^{3} w_{i} \eta_{i} N^{T} N \ dJ\\
-\sum_{i=0}^{3} w_{i} B^{T} B \ dJ
$$

Onde, para esse caso, $w_i = 1\ \forall i$ (isso não é sempre dessa forma,
depende, por exemplo, da ordem da quadratura), e $dJ = \text{det}(\nabla^T N \cdot X)$. Por fim,

$$
f_{e} = -\sum_{i=0}^{3} N_{e}^{T} N S_{e} \ dJ
$$

Salvo alguns detalhes, acredito que, a partir daqui, poderiamos seguir com a
implementação, em Python. Infelizmente, esse post ficou demasiado extenso, por
isso, vou parar por aqui e deixar-la pra uma próxima. O mais importante foi
criar essa visão geral do que será resolvido. Eu compreendo que ficaram vários
detalhes para trás, então, sugestões ou críticas ao post são bem vindas!
Agradeço aos colegas de laboratório, Matheus Wenceslau, Jan-Michel e Paulo
Bastos, pelas sugestões.

Até a próxima!
