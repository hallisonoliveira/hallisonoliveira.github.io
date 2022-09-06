---
layout: post
title: L - Princípio de Substituição de Liskov
img: solid/liskov_substitution.png
category: SOLID
series: solid
tags: [
    solid,
    uncle bob,
    software,
    development,
    liskov substitution,
    substituição de liskov
]
comments: true
hidden: true
---

Em inglês, *Liskov Substitution Principle*.

## Um pouco de história

Esse princípio vem do notável trabalho de [Bárbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov), cientista da computação, que foi pioneira em diversas contribuições para a ciência da computação e computação distribuída.

Barbara foi reconhecida em 2008 com o *[Turing Award](https://en.wikipedia.org/wiki/Turing_Award)*, a mais alta distinção em ciência da computação, dada à ela “por contribuições para fundamentos práticos e teóricos de linguagem de programação e design de sistemas, especialmente relacionados à abstração de dados, tolerância a falhas e computação distribuída”. [*traduzido de* A. M. Turing Award](https://amturing.acm.org/award_winners/liskov_1108679.cfm).

No vídeo abaixo, Bárbara fala para o A. M. Turing Award sobre a origem e o nome desse princípio:

{% include youtube.html id='-Z-17h3jG0A' %}

## O princípio

Basicamente, o Princípio de Substituição de Liskov fala sobre tipagem comportamental. Ou seja, quando temos uma classe principal e uma sub-classe que extende da principal (herança), a sub-classe deve se comportar da mesma forma que a classe principal, podendo substituí-la sem que a execução do sistema seja comprometida.

### Comportamento

Vejamos o seguinte exemplo, onde esse princípio é quebrado:

```kotlin
open class ValidadorDeTexto {
    open val limiteDeCaracteres = 5

    fun ehValido(texto: String): Boolean {
        return texto.length < limiteDeCaracteres
    }
}

fun main() {
    val validador: ValidadorDeTexto = ValidadorDeTexto()

    val nome = "Carlos"

    if (validador.ehValido(nome)) {
        println("Nome valido")
    } else {
        println("Nome invalido")
    }
}
```

```bash
Nome invalido
```

No código acima, temos a classe `ValidadorDeTexto`, que tem por objetivo validar o tamanho do texto através da função `ehValido(texto: String): Boolean` , que retorna `True` se o texto enviado tiver tamanho menor do que o definido na variável `limiteDeCaracteres`. Essa classe é uma `open class`, o que significa que podemos utilizá-la diretamente ou criar novas classes que estendem dela.

Nesse primeiro exemplo, fizemos uso da classe `ValidadorDeTexto` diretamente passando o texto `Carlos` para ser validado. O resultado é `Nome invalido`, já que o texto possui 6 caracteres e ultrapassa o limite de 5 caracteres definido pela variável `limiteDeCaracteres`.

Agora, imagine que temos um cenário onde precisamos criar um validador específico para nomes. Esse novo validador deve permitir nomes de até 10 caracteres.

Já que a classe `ValidadorDeTexto` é uma `open class`, podemos criar o novo validador herdando dela e aproveitar a implementação da função já existente. Ou seja, a classe `ValidadorDeNome`, que iremos criar, será uma sub-classe de `ValidadorDeTexto`.

Vejamos abaixo como ficaria:

```kotlin
open class ValidadorDeTexto {
    open val limiteDeCaracteres = 5

    fun ehValido(texto: String): Boolean {
        return texto.length < limiteDeCaracteres
    }
}

class ValidadorDeNome : ValidadorDeTexto() {
    override val limiteDeCaracteres = 10
}

fun main() {
    val validador = ValidadorDeNome()

    val nome = "Carlos"

    if (validador.ehValido(nome)) {
        println("Nome valido")
    } else {
        println("Nome invalido")
    }
}
```

```bash
Nome valido
```

Do ponto de vista de código, essa é uma solução válida e não gera problemas de compilação ou execução. Porém, o Princípio de Substituição de Liskov é quebrado pois, quando substituímos a classe `ValidadorDeTexto` pela sua sub-classe `ValidadorDeNome`, o comportamento do sistema é alterado. Ou seja, o mesmo texto que antes era inválido, passou a ser válido após a mudança.

Esse efeito é perigoso pelo fato de que a mudança na validação não é explícita e só é visível analisando a implementação, o que não pode ser aceito. Se esses validadores estivessem em uma biblioteca externa, é provável que não seja possível ter acesso à implementação. Assim, não seria possível avaliar o comportamento sem que o sistema seja executado ou através de testes unitários.

### _Exceptions_

Esse princípio não trata apenas de comportamentos e retornos. É importante tomar cuidado também com o lançamento de exceções pelas sub-classes, onde novas exceções não podem ser lançadas pelas funções das sub-classes, a não ser que essas exceções sejam subtipos das exceções já lançadas pelas funções da classe principal.

## Referências

- Arquitetura Limpa: O Guia do Artesão para Estrutura e Design de Software. Martin, Robert C ; traduzido por Samantha Batista. Rio de Janeiro. Alta Books, 2018.
