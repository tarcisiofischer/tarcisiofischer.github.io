---
layout: post
title: Estudo de caso - Cuidados ao guardar closures em objetos
---

Estive desenvolvendo um pequeno jogo, sidescroller 2D, chamado King vs Pigs. No jogo, os porquinhos roubaram o
tesouro do personagem principal, que invade o castelo dos porcos em busca de recuperar suas joias. O jogo é
feito totalmente em C++, com auxilio da biblioteca SDL. Ele ainda está em desenvolvimento, mas espero eventualmente
disponibilizar uma versão para download.

{:refdef: style="text-align: center;"}
![](/images/2021-03-30/ssat2.gif)
{: refdef}

O jogo conta com animações baseadas em pixel art com auxilio de spritesheets. São basicamente grandes imagens
com vários frames, e um subconjunto desses frames completa um estado do personagem. Por exemplo, na imagem abaixo,
existem os frames do porquinho parado (`IDLE`), correndo (`RUNNING`), morrendo (`DYING`), entre outros. Todos as
imagens do jogo foram feitas pelo [Pixel Frog](https://twitter.com/_PixelFrog).

{:refdef: style="text-align: center;"}
![](/images/2021-03-30/pig80x80.png)
{: refdef}

Para fazer o handling das animações dos personagens do jogo, criei uma classe `Animation`, que possui informações em
memória do spritesheet, dos frames, e do tempo de cada frame (Geralmente 100ms). Cada personagem no
jogo possui um mapa, associando um estado (`IDLE`, `RUNNING`, `DYING`...) à uma `Animation`. O controle de qual
animação está ativa depende do estado do personagem, e ações que acontecem na camada de modelo do jogo
afetam a camada de apresentação por meio desses estados.

Existem, porém, estados que podem depender de acontecimentos na camada de animação para acontecer.
Por exemplo, ao tomar dano, o porco deve mudar seu estado para `TAKING_DAMAGE`, e quando essa animação
terminar de acontecer, seu estado deve ser revertido novamente para `IDLE` ou para `DYIED`, dependendo
de seus pontos de vida.

Para evitar que exista essa quebra de dependências entre as camadas de modelo e apresentação, criei uma
callback na Animation para que, ao terminar, o modelo possa registrar uma função para decidir como proceder.
O código é o seguinte:

{% highlight c++ linenos %}
void Animation::set_on_finish_animation_callback(std::function<void()> const& f)
{
   this->on_finish_animation = f;
}

void Animation::run(...)
{
// ...
        if ((this->state % this->frames.size()) == 0) {
            this->state = 0;
            if (this->on_finish_animation) {
                (*this->on_finish_animation)();
            }
        }
// ...
}
{% endhighlight %}

De forma que o código do modelo do porco fica:

{% highlight c++ linenos %}
void Pig::connect_callbacks()
{
    this->animations.at(TAKING_DAMAGE_ANIMATION).set_on_finish_animation_callback([this]() {
        this->is_taking_damage = false;
        this->life -= 1;
        if (this->life <= 0) {
            this->is_dying = true;
        }
    });
// ...
}
{% endhighlight %}

E, por fim, o código de controle do cenário, na parte que adiciona os personagens ativos na tela é algo assim:

{% highlight c++ linenos %}
auto pigs = std::vector<Pig>();
int n_pigs = 10;
for (int i = 0; i < n_pigs; ++i) {
    pigs.push_back(Pig(/*...*/));
}
{% endhighlight %}

E o resultado dessa versão ficava como no gif abaixo. Note que alguns porcos ficavam presos nos estados tomando
dano ou morrendo, o que não é esperado. Depois de debugar um pouco, descobri o motivo que, para alguns, pode
ser até obvio.

{:refdef: style="text-align: center;"}
![](/images/2021-03-30/bug_pigs.gif)
{: refdef}

O método `push_back` do `std::vector` adiciona um novo objeto ao vetor, aumentando o vetor caso necessário. Como
a cada adição o vetor aumenta, é necessário copiar os porcos existentes no vetor anterior para um novo local
na heap. Ao acontecer isso, o mapa de animações dos porcos é copiado e, com ele, cada animação faz uma cópia
da `std::function` referente à callback `on_finish_animation`.

Mas, como foi atribuido uma closure à uma std::function, o estado da mesma fica erroneamente referenciando
um ponteiro (`this`) para um porco que potencialmente não existe mais. A partir daí, perde-se o controle do
que exatamente vai acontecer quando essas callbacks forem chamadas.

Claro que existem alternativas para contornar o problema. Poderia modificar o vetor para um vetor de ponteiros
para porcos, e alocar dinamicamente cada um dos porcos, de forma a garantir que o ponteiro deles não mudassem
quando o vetor precisasse realocar os seus objetos. Outra solução seria evitar a lista de captura nas
funções lambda registradas nas callbacks.

Porém, eu preferi uma terceira opção, que foi de reescrever o construtor de cópia do porco, de forma a
atualizar as callbacks para cada nova cópia. Dessa forma, eu não preciso alterar nenhum outro código
que utilize os meus objetos, e também fico livre para capturar qualquer outra variável se eu precisar.
O resultado final pode ser visto no gif baixo. 

{:refdef: style="text-align: center;"}
![](/images/2021-03-30/fix_pigs.gif)
{: refdef}

Devido à essa escolha, foi apenas necessário tomar um cuidado com as
[regras para os construtores de cópia](https://en.cppreference.com/w/cpp/language/rule_of_three).
De qualquer forma, essa é a solução que decidi seguir dentro do escopo desse projeto.
