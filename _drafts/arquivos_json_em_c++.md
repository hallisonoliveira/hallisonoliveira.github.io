---
layout: post
title: Leitura e gravação de arquivos JSON em C++
img: json_cpp_boost.png
category: C++
tags: [
    C++,
    JSON,
    boost
]
comments: true
---

Arquivos JSON são extremamente simples de serem entendidos e utilizados. Essa simplicidade fez com que o JSON se tornasse uma alternativa mais simples e leve para o uso do XML.

Existem várias bibliotecas para se trabalhar com JSON nas mais variadas linguagens. No [site oficial](https://www.json.org/json-pt.html) (em português), é possível ver uma lista com algumas delas.

Esse artigo explica como fazer a leitura e gravação de arquivos JSON em C++ utilizando o **Boost**.

O [Boost](https://www.boost.org/) é uma variada coleção de bibliotecas que estendem o C++, deixando-o ainda mais poderoso. Para a leitura e gravação de arquivos JSON, podemos utilizar a biblioteca **Property Tree** presente no Boost. Essa biblioteca fornece uma estrutura de dados que armazena uma árvore de valores no formato de chave e valor, onde os valores podem ser textos, números ou até mesmo uma lista ou nova árvore.

Lembrando que a Property Tree pode ser utilizada também para leitura e gravação de arquivos XML e INI.

Abaixo, um exemplo de como é possível ler um arquivo JSON com o Boost::Property_Tree.

## Leitura

Para esse exemplo, vamos utilizar o seguinte arquivo JSON:

<script src="https://gist.github.com/hallisonoliveira/b6f4c64022474c441460b1b69836ab5b.js"></script>

Esse arquivo de exemplo foi extraído da [StarWars API](https://swapi.co/). Uma API muito legal que possui várias as informações dos filmes da saga como planetas, espaço naves, veículos e personagens, até o episódio 7 (The Force Awakens), tudo em formato JSON. Para quem é fã da saga, vale a pena dar uma olhada.

No gist abaixo, está o código completo para realizar a leitura do arquivo JSON.

<script src="https://gist.github.com/hallisonoliveira/a0c6a548885c7df459d0003954e6c1d2.js"></script>

### Explicando por partes

Primeiro, criamos um objeto do tipo **ptree** que receberá o conteúdo do arquivo para que possamos manipula-lo.

~~~cpp
boost::property_tree::ptree luke_ptree;
~~~

Precisamos importar a biblioteca **boost/property_tree/ptree.hpp** para isso.

Criado o objeto, adicionamos o conteúdo do arquivo à ele utilizando o método [read_json](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/json_parser/read_json_idp699443552.html) presente na biblioteca **boost/property_tree/json_parser.hpp**.

~~~cpp
boost::property_tree::read_json("luke_skywalker.json", luke_ptree);
~~~

Esse método precisa de dois parâmetros. O primeiro é uma String referente ao nome do arquivo JSON a ser lido. O segundo parâmetro é o objeto *ptree* que receberá o conteúdo. Nesse processo, o conteúdo é traduzido para o formato de chave/valor.

Com isso, já podemos acessar o conteúdo do arquivo através do *ptree*. Basta utilizar o método *get* especificando o tipo de dado a ser lido. No código de exemplo, fazemos a leitura de uma *String* e um inteiro.

~~~cpp
luke_ptree.get<std::string>("name")
~~~

~~~cpp
luke_ptree.get<int>("height")
~~~

Os dois parâmetros acima (*name* e *height*) estão no primeiro nível do JSON e por isso, podemos acessa-los diretamente.

Para listar os itens de uma lista, podemos utilizar o [Foreach](https://theboostcpplibraries.com/boost.foreach) do *Boost* presente na bibliteca **boost/foreach.hpp**, conforme abaixo:

~~~cpp
BOOST_FOREACH(boost::property_tree::ptree::value_type &films_node, luke_ptree.get_child("films")) {
    std::cout << films_node.second.get<std::string>("") << std::endl;
}
~~~

O *foreach* do *Boost* possui um funcionamento bem simples. Ele utiliza dois parâmetros. O primeiro é a variavel ou referência que receberá o item da lista na iteração. O segundo parâmetro é a lista a ser iterada.

É importante ressaltar que o tipo da variável/referencia que irá receber o valor da lista deve ser do mesmo tipo do item da lista. A partir do **C++11**, podemos utilizar o [auto](https://en.cppreference.com/w/cpp/language/auto) para declaração de variáveis. Assim, podemos utiliza-lo dentro do *foreach* e não precisamos nos preocupar com o tipo.

Outro ponto é que no C++ podemos trabalhar com os endereços de memória dos objetos. Conforme citado acima, o *foreach* também aceita o uso de referência, basta colocar o **&** antes do nome do objeto. O código abaixo demonstra como seria o uso do *auto* declarando uma referência de objeto.

~~~cpp
BOOST_FOREACH(auto &films_node, luke_ptree.get_child("films")) {
    std::cout << films_node.second.get<std::string>("") << std::endl;
}
~~~

Para acesso ao item da lista, precisamos utilizar o *second* do objeto. Ele é utilizado para acessar o subnível do item. Dentro do subnível, basta utilizar o *get* da mesma forma que utilizamos anteriormente, sempre passando o tipo de valor a ser lido. Chamamos o *get* passando uma *String* vazia como parâmetro. Devemos fazer isso pois nesse caso, o *second* nos retorna um *std::pair("", ptree)*.

## Gravação

Para exemplificar a gravação, vamos adicionar mais algumas informações no arquivo JSON utilizado acima. As informações que vamos adicionar são referentes à irmâ do *Luke Skywalker*, a princesa *Leia Organa*. Vamos utilizar algumas informações também disponíveis na [StarWars API](https://swapi.co/). O JSON completo disponível no site é o apresentado abaixo:

<script src="https://gist.github.com/hallisonoliveira/efa651bd9b4e8185e797e1a82377cfd9.js"></script>

Vamos adicionar um novo nó no JSON utilizado no exemplo acima de leitura do arquivo. Esse nó terá as seguintes informações:

* Nome (name)
* Altura (height)
* Cor dos cabelos (hair_color)
* Cor dos olhos (eye_color)
* Lista de filmes (films)

Primeiro, criamos o objeto *ptree* que será a raiz desse novo item.

~~~cpp
boost::property_tree::ptree leia_ptree;
~~~

Em seguida, adicionamos as informações de nome, altura, cor dos cabelos e cor dos olhos. Esses itens podem ser adicionados diretamente, já que serão atributos simples dentro do JSON e ficarão direto na raiz do novo elemento.

~~~cpp
leia_ptree.put("name", "Leia Organa");
leia_ptree.put("height", 150);
leia_ptree.put("hair_color", "brown");
leia_ptree.put("eye_color", "brown");
~~~

Observe que adicionamos dois tipos de objetos acima. Os atributos nome, cor dos cabelos e cor dos olhos são do tipo *String* e o atributo altura é do tipo *int*.

Para a adição da lista de filmes, é necessário primeiro criar um [vetor](http://www.cplusplus.com/reference/vector/vector/) de *String* para facilitar o mapeamento dos elementos que serão adicionados à lista do JSON.

~~~cpp
std::vector<std::string> films;
~~~

Adicionamos os elementos utilizando o método [push_back](http://www.cplusplus.com/reference/vector/vector/push_back/) do *vector*.

~~~cpp
films.push_back("https://swapi.co/api/films/1/");
films.push_back("https://swapi.co/api/films/2/");
films.push_back("https://swapi.co/api/films/3/");
films.push_back("https://swapi.co/api/films/6/");
films.push_back("https://swapi.co/api/films/7/");
~~~

Agora, criamos o objeto *ptree* referente à lista de filmes.

~~~cpp
boost::property_tree::ptree films_ptree;
~~~

Com o uso do [Foreach](https://theboostcpplibraries.com/boost.foreach) do *Boost*, percorremos o *vector* de filmes, adicionando-os no *ptree*.

~~~cpp
BOOST_FOREACH(auto film, films) {
    boost::property_tree::ptree film_ptree;
    film_ptree.put("", film);

    films_ptree.push_back(std::make_pair("", film_ptree));
}
~~~

Criado o *ptree* de filmes, basta adiciona-lo ao *ptree* principal, passando a chave "films" como identificador desse novo nó da árvore.

~~~cpp
leia_ptree.add_child("films", films_ptree);
~~~

Agora, adicionamos essa nova informação ao arquivo JSON utilizado anteriormente. Para isso, precisamos abrir o arquivo e atribuir seu conteúdo à um objeto *ptree*, da mesma forma que fizemos na seção de leitura de arquivos.

~~~cpp
boost::property_tree::ptree luke_ptree;
boost::property_tree::read_json("luke_skywalker.json", luke_ptree);
~~~

No conteúdo do arquivo, adicionamos o novo nó da árvore de atributos. Por se tratar das informações da irmã do *Luke Skywalker*, vamos chamar esse novo nó de *sister*.

~~~cpp
luke_ptree.put_child("sister", leia_ptree);
~~~


*Background da imagem do post: <a href="https://pixabay.com/pt/users/JAKO5D-2733400/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1627322">JAKO5D</a> por <a href="https://pixabay.com/pt/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1627322">Pixabay</a>*