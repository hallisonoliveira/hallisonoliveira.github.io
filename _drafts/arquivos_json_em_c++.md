---
layout: post
title: Leitura e gravação de arquivos JSON em C++
img: 
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

### Explicando por partes:

Primeiro, criamos um objeto do tipo **ptree** que receberá o conteúdo do arquivo para que possamos manipula-lo.

> boost::property_tree::ptree luke_ptree;

Precisamos importar a biblioteca **boost/property_tree/ptree.hpp** para isso.

Criado o objeto, adicionamos o conteúdo do arquivo à ele utilizando o método [read_json](https://www.boost.org/doc/libs/1_65_1/doc/html/boost/property_tree/json_parser/read_json_idp699443552.html) presente na biblioteca **boost/property_tree/json_parser.hpp**.

> boost::property_tree::read_json("luke_skywalker.json", luke_ptree);

Esse método precisa de dois parâmetros. O primeiro é uma String referente ao nome do arquivo JSON a ser lido. O segundo parâmetro é o objeto *ptree* que receberá o conteúdo. Nesse processo, o conteúdo é traduzido para o formato de chave/valor.

Com isso, já podemos acessar o conteúdo do arquivo através do *ptree*. Basta utilizar o método *get* especificando o tipo de dado a ser lido. No código de exemplo, fazemos a leitura de uma *String* e um inteiro.

> luke_ptree.get<std::string>("name")
> luke_ptree.get<int>("height")

Os dois parâmetros acima (*name* e *height*) estão no primeiro nível do JSON e por isso, podemos acessa-los diretamente.

Para listar os itens de uma lista, podemos utilizar o *Foreach* do *Boost* presente na bibliteca **boost/foreach.hpp**, conforme abaixo:

> BOOST_FOREACH(boost::property_tree::ptree::value_type &starships_node, luke_ptree.get_child("starships")) {
>   std::cout << starships_node.second.get<std::string>("") << std::endl;
> }


/*
void read_json_from_file()
{
    boost::filesystem::path file_path("/home/holiveira/Desenvolvimento/sensors_100.json");

    while (true)
    {
        const std::locale &loc = std::locale();
        cout << loc.name() << endl;

        boost::property_tree::ptree pt;

        try
        {
        boost::property_tree::read_json(file_path.string(), pt);
        } catch (...) {
           cout << "XXXXXXXXXXXXXXXXXXXXX" << endl;
        }

        //boost::property_tree::ptree listNode;
        try
        {
            BOOST_AUTO(listNode,  pt.get_child("sensors"));
            std::pair<boost::property_tree::ptree::const_assoc_iterator, boost::property_tree::ptree::const_assoc_iterator> bounds = listNode.equal_range("");

            boost::property_tree::ptree sensor_pt(bounds.first->second);

            cout << "ID: " << sensor_pt.get<int>("id") << endl;

            BOOST_FOREACH(boost::property_tree::ptree::value_type &faixas_node, sensor_pt.get_child("lanes") )
            {
                boost::property_tree::ptree &faixas_pt = faixas_node.second;

                cout << "ID: " << faixas_pt.get<int>("id") << endl;
            }

            cout << "Leitura OK" << endl;
        } catch (...) {
            cout << "Erro ao ler conteudo" << endl;
        }

        sleep(0.5);
    }
}
*/
========================================================================================================
/*
boost::filesystem::path lr_camera_file("/home/holiveira/Desenvolvimento/cameras_lr.json");
    boost::filesystem::path camera("/home/holiveira/Desenvolvimento/cameras.json");

    boost::property_tree::ptree lr_camera;
    boost::property_tree::ptree cameras_closer;

    boost::property_tree::read_json(lr_camera_file.string(), lr_camera);
    boost::property_tree::read_json(camera.string(), cameras_closer);


    cout << "iniciando" << endl;
    BOOST_FOREACH(boost::property_tree::ptree::value_type &lr_binds_node, lr_camera.get_child("camera.binds"))
    {
        boost::property_tree::ptree &lr_bind_pt = lr_binds_node.second;
        // Percorre a lista de cameras estreitas
        cout << "1" << endl;
        BOOST_FOREACH(boost::property_tree::ptree::value_type &closer_node, cameras_closer.get_child("cameras.closer"))
        {
            boost::property_tree::ptree &closer_pt = closer_node.second;

            cout << "ID " << closer_pt.get<std::string>("id") << endl;

            cout << "1" << endl;


            if (boost::iequals(lr_camera.get<std::string>("camera.id"), closer_pt.get<std::string>("id")))
            {
                BOOST_FOREACH(boost::property_tree::ptree::value_type &binds_node, closer_pt.get_child("binds"))
                {
                    boost::property_tree::ptree &bind_pt = binds_node.second;
                    if (boost::iequals(lr_bind_pt.get<std::string>("name"), bind_pt.get<std::string>("name")))
                    {
                        cout << "Adicionando..." << endl;
                        cameras_closer.push_back(std::make_pair("crop", lr_bind_pt.get_child("crop")));
                    }
                }
            }
        }
    }
*/

============================================================================================

void create_json_from_files_of_folder()
{
    using namespace boost::filesystem;
    using namespace std;

    namespace pt = boost::property_tree;

    path p("/home/holiveira/Desenvolvimento/distropk/issues/dcs/certs/");

    std::stringstream result;

    if(exists(p))
    {
        vector<path> files_vector;

        copy(directory_iterator(p), directory_iterator(), back_inserter(files_vector));

        pt::ptree certs_tree;

        pt::ptree files_tree;
        for (vector<path>::const_iterator it(files_vector.begin()), it_end(files_vector.end()); it != it_end; ++it)
        {
            path file(*it);

            boost::posix_time::ptime last_modified_time = boost::posix_time::from_time_t(last_write_time(file));
            std::string last_modified = boost::lexical_cast<std::string>(last_modified_time.date());

            std::string filename = file.filename().c_str();
            int size_file = file_size(file);

            cout << "Filename: " << filename << endl;
            cout << "Size: " << size_file << endl;
            cout << "Last Modified: " << last_modified << endl;

            pt::ptree file_tree;
            file_tree.put("filename", filename);
            file_tree.put("size", size_file);
            file_tree.put("last_modified", last_modified);

            files_tree.push_back(std::make_pair("", file_tree));
        }
        certs_tree.push_back(std::make_pair("files", files_tree));

        pt::write_json(result, certs_tree, false);

        cout << result.str() << endl;
    }
    else
    {
        cout << "Diretorio nao encontrado." << endl;
    }

}