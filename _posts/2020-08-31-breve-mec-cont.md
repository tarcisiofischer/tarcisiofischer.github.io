---
layout: post
title: Uma breve introdução sobre mecânica do contínuo
image: 2020-08-31/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-08-31/img001.png)
{: refdef}

Eu sou cientista da computação por formação. Na grade curricular do meu curso, na UFSC, não tinhamos nenhuma disciplina de física. Tinhamos cálculo I, II e calculo numérico, e também geometria analítica e algebra linear. Digo isso porque quando iniciei meu mestrado em engenharia mecânica, eu tinha pouca noção sobre cálculo multivariável, por exemplo. E isso, alinhado ao fato de eu não ser engenheiro mecânico, dificultou um pouco meu caminho, principalmente no decorrer das disciplinas.

A questão é que, na época, eu não sabia o quanto cálculo vetorial era importante. Cálculo vetorial e cálculo multivariável envolvem uma grande parte da modelagem de problemas físicos, em especial em problemas relacionados à mecânica do contínuo, seja mecânica dos sólidos ou dos fluidos. Isso por que conseguimos modelar muitos problemas do mundo real em um espaço tridimensional e aplicar modelos físicos para tirar conclusões que nos ajudam a solucionar problemas.

Dentro desse contexto, eu teria um longo caminho para explicar toda a parafernalha teórica necessária para chegar aos conceitos que quero descrever nesse texto. Porém, a ideia aqui é apenas introduzir algumas ideias para aqueles que, assim como eu, não compreendem como exatamente as coisas se encaixam e se aplicam em problemas de modelagem mecânica. Por isso, ao invés de seguir o caminho clássico, sigo um bem mais simples (mesmo assim complexo), até para tentar fazer caber tudo em um único post. Por outro lado, necessito que o leitor esteja no mínimo familiarizado com os conceitos de vetores e derivadas, pelo menos. Mas admito que esses são relativamente comuns em disciplinas dos primeiros anos de graduação de ciências exatas em geral.


## Campos escalares e vetoriais

Em primeiro lugar, é necessário pelo introduzir o conceito de "campo escalar". Campos escalares são funções que mapeiam os pontos no espaço em valores escalares. Matematicamente, é uma $f(X) = y$ onde $X$ é um vetor e $y$ um escalar (um número).

Por exemplo, imagine o fundo de uma panela de metal que aquece em cima de um fogão ligado. Ora, se a panela estava fria quando colocamos sobre o fogão, é fácil imaginar que ela está aquecendo a medida que o tempo passa. Dessa forma, se tirarmos uma "foto" do fundo da panela em um dado instante de tempo, e observarmos a temperatura de cada posição do fundo da panela, constatariamos que ele é mais quente no centro e, a medida que avançamos para as extremidades, ficaria gradativamente mais frio.  A imagem abaixo ilustra essa ideia: Olhando o fundo da panela de cima, ela é basicamente um circulo. A cada ponto desse circulo, associamos um valor de temperatura.

{:refdef: style="text-align: center;"}
![](/images/2020-08-31/fundo-panela-2d.png)
{: refdef}

Outro aspecto importante nessa questão (E que me deixava um pouco desconfortável no início dos meus estudos), é que precisamos acreditar que existe uma função matemática que define o campo de temperatura do fundo da panela. Ou seja, ao invés de eu ter uma tabela enorme, onde, para cada ponto no espaço eu associo um número, que representa uma temperatura, eu escrevo uma "regrinha", que associa os valores de $(x, y, z)$ à um valor de temperatura. Por exemplo, $f(x, y, z) = -(x^2 + y^2) + 100$. Acredito que, para alguns, esse conceito de poder-se associar a temperatura do fundo da panela à uma função pode parecer "óbvio".

Convido o leitor a parar um momento para pensar sobre isso. Especialmente se você estiver iniciando seus estudos nessa área. Pense como você, apenas observando algo na natureza, poderia criar uma função matematica que descreve o campo escalar de temperatura daquilo que se está observando, por exemplo. Será que é sempre uma função contínua? E se os materiais do objeto sendo observado forem diferentes em uma parte do mesmo? E se o tempo entrar na história, é fácil determinar tanto a variação espacial quanto temporal usando uma função escalar?

Essa ideia de associar valores à cada ponto no espaço aparece o tempo inteiro nos problemas de modelagem física. Campos de pressão e  campos de temperatura são apenas dois exemplos possíveis. Além disso, essa ideia é posteriormente expandida para imaginar campos vetoriais, ou seja, para cada ponto no espaço associamos um vetor. Pode parecer estranho em um primeiro momento, mas um exemplo bastante claro disso é o de um campo de velocidades. A velocidade é uma grandeza vetorial. Portanto, podemos ter um vetor diferente (que representa uma velocidade) para cada posição no espaço. Como exemplo, deixo abaixo uma imagem de um campo de velocidades do vento passando sobre uma vaca. Infelizmente eu não sei quem fez esse experimento, para dar os créditos. Imagine que cada seta em cima da superfície da vaca representa qual a velocidade do vento passando sobre a vaca naquele ponto. Isso é o campo vetorial de velocidade do vento sobre a superfície da vaca.

{:refdef: style="text-align: center;"}
![](/images/2020-08-31/vaca-3d.png)
{: refdef}


## Da física clássica à mecânica do contínuo

Tendo em vista os conceitos explicados anteriormente, convido agora o leitor a lembrar de uma fórmula clássica do ensino médio:

$$
f = ma
$$

Ou seja, o somatório de forças é igual à massa vezes aceleração. Até onde me lembro, aprendemos isso e aplicamos em vários problemas de quadradinhos caindo sobre triangulos, ou quadradinhos sendo puxados por linhas amarradas bem em seus centros. O pricípio fundamental da mecânica é muito útil na modelagem de sistemas com "corpos rígidos". De forma não matemática, um corpo rígido é um corpo que pode se movimentar, mas não pode se deformar. Por outro lado, o que você imagina que acontece se você segura uma extremidade de uma borracha e aperta em outra? Ou se você der um soco na água dentro de um balde?

{:refdef: style="text-align: center;"}
![](/images/2020-08-31/soco-agua.jpg)
{: refdef}

Acredito que tenha sido Euler que tenha apresentado formalmente a ideia de usar a segunda lei de Newton em uma parcela de um corpo. Para compreender isso, imagine dividir aquela borracha em pequenos pedaços (volumes). Os pedaços não podem ser tão pequenos quanto os átomos, nem tão grandes quanto a própria borracha. São pedaços grandes o suficiente para que você ainda consiga ver que aquilo é parte de uma borracha, mas pequenos o suficiente para que você perceba que, aplicando uma força sobre aquele pedaço, seja perceptível a ação que ele faz sobre os pedaços vizinhos.

{:refdef: style="text-align: center;"}
![](/images/2020-08-31/batatoide.png)
{: refdef}

Além disso, precisamos agora considerar que as propriedades (massa, aceleração, etc) podem ser diferentes em cada pedaço do domínio. Vendo cada pedaço com um volume definido, então, como massa é o produto de volume e densidade, podemos escrever toda a parcela à esquerda da segunda lei de Newton como

$$
ma = \frac{d}{dt} \int_V \rho u dV
$$

Admito que a partir daqui as coisas ficam um pouco mais difíceis. Mas vou tentar simplificar ao máximo, pelo menos para dar uma visão mais geral. Em primeiro lugar, a aceleração foi reescrita como a derivada da velocidade no tempo, ou seja, $a=du/dt$. Além disso, a massa foi reescrita como uma integral de densidade ($\rho$) no volume. Isso significa que a densidade é um campo escalar. Para cada recorte de volume do nosso domínio, podemos ter uma densidade e uma velocidade diferentes. Ou seja, na nossa borracha, poderiamos ter partes dela mais ou menos densas. Se o domínio fosse um fluido, o ar por exemplo, ele poderia estar mais ou menos comprimido, gerando densidades diferentes. É interessante parar pra pensar nas possibilidades de modelagem com esse tipo de abordagem.

Além disso, a parte à direita da equação pode ser decomposta em "forças externas" sendo aplicadas à superfície do volume sendo considerado (por exemplo, alguém empurrando a borracha), e "forças de corpo" sendo aplicadas à toda a extensão da borracha (por exemplo, a gravidade). Na equação abaixo, a integral em $V$ representa a integração no volume e a integral em $S$ representa em relação à superfície externa. Precisamos integrar por que, como comentado anteriormente, a mecânica clássica traz a lei apenas para corpos rígidos e, portanto, deve ser considerada por toda a extensão da borracha do nosso exemplo.

$$
f = \int_V f dV + \int_S t dS
$$

Admito que eu demorei um pouco para entender essa parte da equação. Mas dou uma dica: Se você decompor as forças que podem estar sendo aplicadas em um corpo, ou elas estão sendo aplicadas na superfície do corpo ($\int_S t dS$) ou elas estão sendo aplicadas no corpo, como no exemplo da gravidade ($\int_V f dV$). Digo isso no sentido de que não existe outra forma de aplicar uma força à um corpo. A equação completa fica

$$
\int_V f dV + \int_S t_n dS = \frac{d}{dt} \int_V \rho u dV
$$

Onde $t_n$ é uma função relacionada à forças externas e internas, do volume considerado. Em 1838, Cauchy mostrou que essa função qualquer $t_n$ pode ser decomposta em uma função qualquer $T$ aplicada à componente normal $n$ da superfície de qualquer volume interno, escrevendo

$$
t_n = T \cdot n
$$

Isso pode não parecer ter grande valor, mas a grande implicação disso é que, agora, podemos tirar proveito de algumas propriedades (veremos a seguir). Essa nova quantidade é chamada "tensor de tensões de Cauchy", e é aqui que entra toda a modelagem "pesada", chamada "modelagem constitutiva", que inclusive separa as disciplinas de mecânica dos sólidos e dos fluidos. As famosas conservações de quantidade de movimento linear das equações de Navier-Stokes, por exemplo, são nada mais do que equações derivadas a partir desse ponto, dada uma modelagem constitutiva específica para alguns fluidos. Por outro lado, se considerarmos uma barra de metal, partimos desse mesmo ponto, porém, estudamos uma modelagem constitutiva para elasticidade ou elasto-plasticidade. As possibilidades são muitas, e essa é a beleza da coisa.

Apenas por completude, continuo o trabalho mais um pouco, para mostrar que, na verdade, essas equações independem do domínio de integração. Isso é relevante pois explica por que não vemos muitos modelos em sua forma integral. Por exemplo, normalmente não existem integrais nas equações de Navier-Stokes. 

Basicamente, tudo começa com a seguinte indagação: "É uma pena que os domínios das integrais são diferentes: 2 são no volume mas o outro é na superfície..." Graças a um teorema clássico chamado "teorema da divergência" (Ou teorema de Gauss), isso pode ser contornado. O teorema diz, genéricamente, que

$$
\int_S F \cdot n dS = \int_V (\frac{\partial F}{\partial x} + \frac{\partial F}{\partial y} + \frac{\partial F}{\partial z}) \ dV
$$

Ou seja, se você integrar a componente normal de uma função $F$ qualquer em uma superfície qualquer, o resultado dessa integração é igual à você calcular a integral do somatório das derivadas parciais dessa mesma função. Não fiz nada além de descrever em texto o que está escrito matematicamente acima. Claro que não vou explicar a prova desse teorema nesse momento (porém, se o leitor tiver interesse, é muito fácil encontrar pela internet). Peço que o leitor assuma que esse teorema seja verdade, para que possamos continuar no raciocínio principal. Além disso, para facilitar a digitação, vou usar uma notação chamada "operador de divergente", que é definido assim:

$$
\nabla \cdot F = \frac{\partial F}{\partial x} + \frac{\partial F}{\partial y} + \frac{\partial F}{\partial z}
$$

Finalmente podemos escrever que

$$
\int_V \Big{(} f + \nabla \cdot T - \frac{d}{dt} \rho u \Big{)} \ dV = 0
$$

Se essa equação é verdade ao mesmo tempo para o domínio inteiro e também para uma parcela do domínio, então ela independe da integral em questão. Em outras palavras, se a integral de uma função qualquer é zero independente da parcela de domínio que estamos nos referindo, então ela deve ser zero no domínio inteiro. Ou seja, podemos escrever simplesmente

$$
\rho f + \nabla \cdot T - \frac{d}{dt} \rho u = 0
$$

Vou parar por aqui pois sei que o post ficou demasiado grande. Sei que deixei de lado vários formalismos matemáticos (Por exemplo, chamei de "função" alguns tensores de maior ordem, e nem  expliquei o que são tensores). Não mostrei as equações de conservação da massa, não mostrei as definições de integrais de campos vetoriais, nem a prova da existência do tensor de tensões de Cauchy. Faltaria ainda mostrar o teorema de Reynolds para deixar a última equação em uma forma até mais conhecida pelas pessoas. Isso sem nem comentar toda a parte termodinâmica, que é um mundo a parte e mereceria uma série de posts só pra ela.

Espero que, mesmo assim, esse texto possa ser mais uma fonte para outros estudantes que, assim como eu, não se sentiam confortáveis no início dos estudos.

Até a próxima!


Algumas imagens utilizadas nesse post foram retiradas de:
<a href='https://www.freepik.com/vectors/background'>Background vector created by freepik - www.freepik.com</a> <a href='https://www.freepik.com/vectors/school'>School vector created by freepik - www.freepik.com</a> <a href="https://pixabay.com/pt/users/MisterEmmEss-30669/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=103583">M S</a> por <a href="https://pixabay.com/pt/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=103583">Pixabay</a>
