---
layout: post
title: Python, Numpy e o artigo da Numpy na Nature
image: 2020-09-21/img001.png
---

{:refdef: style="text-align: center;"}
![](/images/2020-09-21/img001.png)
{: refdef}

Eu programo em Python faz aproximadamente 9 anos, e uso a Numpy acho que faz uns 6 anos. Tem sido um longo aprendizado, tanto sobre a biblioteca em si quanto sobre a comunidade e as ferramentas que orbitam ao redor dela. Semana passada, a equipe da Numpy [publicou um artigo na Nature](https://www.nature.com/articles/s41586-020-2649-2) (Acessível gratuitamente), trazendo um pouco sobre o histórico, visão geral sobre a biblioteca e sobre o "ecossistema científico" do Python. Embora eu já tenha comentado algumas [dicas sobre Numpy](https://tarcisiofischer.github.io/2020-05-18/numpy-tips) em um post passado, achei uma boa oportunidade de falar um pouco mais sobre Numpy e Python, de uma forma um pouco menos técnica.

Eu não consigo imaginar alguém trabalhando com desenvolvimento de software científico ou pesquisa utilizando Python e não utilizando esse ecossistema formado pela [Numpy](http://numpy.org/), [Scipy](scipy.org), [matplotlib](http://matplotlib.org/), e possívelmente usando os [scikits](https://www.scipy.org/scikits.html), ou outras bibliotecas. Isso por que os módulos disponíveis permitem trabalhar de forma fácil e eficiente com operações envolvendo vetores, matrizes, tensores e algebra linear, contendo tanto as estruturas de dados e operações básicas, quanto funções para soluções de [sistemas lineares e não lineares](https://docs.scipy.org/doc/scipy/reference/tutorial/linalg.html), [matrizes esparsas](https://docs.scipy.org/doc/scipy/reference/sparse.html), [otimização](https://docs.scipy.org/doc/scipy/reference/optimize.html), [estatística](https://docs.scipy.org/doc/scipy/reference/stats.html), [machine learning](https://scikit-learn.org/), entre outras.

Sobre [Python](https://www.python.org/), embora muita gente ainda prefira se manter no Matlab, Fortran, C++ ou outras linguagens, migrar para Python e utilizar a Numpy possui algumas vantagens. A linguagem é totalmente gratuita e open-source e possui uma vasta comunidade (não restrita à comunidade da Numpy). Isso é legal pois alguém de ciência de dados pode tirar dúvidas sobre Python com alguém que trabalha com desenvolvimento web, por exemplo. Apesar de linguagens como [Julia](https://julialang.org/) estarem em crescimento, ainda vejo o Python com Numpy como uma base sólida para "caminhar", com mais ferramentas, IDEs, debuggers, maior comunidade, conteúdo gratuito disponível e tempo de vida - Pelo menos até onde tenho conhecimento.

Voltando à Numpy e ao artigo, vejo que a publicação possui relevância por, além de dar o panorâma histórico e o resumo técnico, atualizar todo o contexto do que surgiu e está surgindo ao redor do projeto. O artigo causou [várias reações no twitter](https://twitter.com/numpy_team/status/1306268442450972674). Um ponto levantado foi sobre a motivação e relevância em publicar um novo artigo sobre algo que já existe faz tanto tempo (Afinal [já existe um artigo de 2011](https://ieeexplore.ieee.org/document/5725236)). Ao meu ver, o artigo anterior é bem diferente do atual, e funcionava mais como um tutorial introdutório pra biblioteca. Por outro lado, o novo artigo dá uma visão bem mais geral, o que, ao meu ver, é muito interessante principalmente para aqueles que estão iniciando ou acompanhando de longe o desenvolvimento do projeto.

Em uma pesquisa rápida na comunidade da [linguagem Julia](https://julialang.org/), encontrei [essa thread](https://discourse.julialang.org/t/a-review-article-about-numpy-was-published-in-nature/46739) de discussão entre os membros. Eu tenho tateado Julia "de longe", e estou com alguns projetos "engatilhados" para fazer uns testes. Para mim, um artigo análogo à esse que fizeram da Numpy, só que pra Julia, seria um prato cheio. É esse o tipo de coisa que me refiro ao galar que "é muito interessante para aqueles que estão acompanhando de longe o desenvolvimento do projeto". Eu tenho dúvidas sobre afirmações do tipo "Julia está pelo menos 15 anos à frente", mas, ainda assim, entendo que existe muita coisa boa e interessante crescendo ali. Falar sobre a linguagem em si daria um (ou mais) posts por si só.

Outro questionamento do pessoal no Twitter foi sobre ausência de representatividade feminina na lista de autores. Eu não faço parte da equipe, embora conheça pessoas que façam. Eu não quero entrar muito nesse assunto, mas [existem sim colaboradoras femininas](https://twitter.com/InessaPawson/status/1306923254813085700) e [estão ativamente trabalhando nessas questões](https://twitter.com/numpy_team/status/1306547886579167232).

É interessante acompanhar e observar a reação e percepção das pessoas em cima do projeto Numpy. Com certeza é um projeto que está ativo e avançando. Um exemplo disso foi a atualização recente do logo e do [site do projeto](https://numpy.org/). Sei que tem muita coisa boa acontecendo na parte de documentação, criação de tutoriais e conteúdo, e estou bastante curioso para ver os resultados de todo esse trabalho no futuro.

Até a próxima!

As imagens de capa foram extraídas do artigo: Harris, C.R., Millman, K.J., van der Walt, S.J. et al. Array programming with NumPy. Nature 585, 357–362 (2020). [https://doi.org/10.1038/s41586-020-2649-2](https://doi.org/10.1038/s41586-020-2649-2)
