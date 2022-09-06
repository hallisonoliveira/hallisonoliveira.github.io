---
layout: post
title: D - Princípio da Inversão de Dependência
img: solid/dependency_inversion.png
category: SOLID
series: solid
tags: [
    solid,
    uncle bob,
    software,
    development,
    dependency inversion,
    inversão de dependencia
]
comments: true
hidden: true
---

Em inglês, *Dependency Inversion Principle*.

A ideia central desse princípio é a de que módulos não devem depender de implementações concretas, mas sim de abstrações. Esse é um dos conceitos centrais da arquitetura limpa, o qual  protege a camada de domínio da aplicação de possuir dependências à bibliotecas e/ou frameworks externos dos quais ela não deveria possuir.

Para entender melhor o princípio, vamos utilizar o exemplo de *Logger* novamente (já o utilizamos para demonstrar o <a href="{{ site.baseurl }}/posts/open_close_principle.html">princípio aberto/fechado</a>).

```kotlin
fun main() {
    val logger = TerminalLogger()

    logger.debug("Log de debug")
}

class TerminalLogger {
    fun debug(message: String) {
        println(message)
    }
}
```

Com esse exemplo simples, já é possível visualizar a violação do princípio. Perceba que a variável `logger` é do tipo `TerminalLogger()`, que é uma implementação concreta. Ou seja, nossa função `main()`, que nesse exemplo representa qualquer outra classe, está suscetível à mudanças que eventualmente possam acontecer na classe `TerminalLogger()`. Podemos demonstrar isso fazendo com que a classe `TerminalLogger()` passe a depender de uma outra classe. Nesse caso, a função `main()` teria que conhecer essa nova classe para que uma nova instância de `TerminalLogger` possa ser criada:

```kotlin
fun main() {
    val systemProperties = SystemProperties()
    val logger = TerminalLogger(systemProperties)

    logger.debug("Log de debug")
}

class TerminalLogger(
    private val systemProperties: SystemProperties
) {
    fun debug(message: String) {
        val systemName = systemProperties.getName()
        println("$systemName - $message")
    }
}

class SystemProperties() {
    fun getName(): String {
        return "Exemplo-DIP"
    }
}
```

Agora, temos a classe `SystemProperties` que é responsável por fornecer o nome do sistema através da função `getName(): String`. O problema aqui é que, quem depende diretamente dela  é a classe `TerminalLogger`. Porém, é a função `main()` quem deve instancia-la, fazendo com que a função conheça uma dependência de `TerminalLogger`, algo que está além da sua responsabilidade.

Vejamos como esse exemplo ficaria, eliminando a violação desse princípio. O primeiro passo, é criar uma abstração (interface), para os *Loggers* e fazer com que a classe `TerminalLogger` a implemente:

```kotlin
interface Logger {

    fun debug(message: String)
}

class TerminalLogger(
    private val systemProperties: SystemProperties
) : Logger {
    override fun debug(message: String) {
        val systemName = systemProperties.getName()
        println("$systemName - $message")
    }
}
```

Feito isso, precisamos de uma forma de criar uma nova instância da classe `TerminalLogger` sem que a função `main()` conheça suas dependências. Uma forma de fazer isso, é através de [injeção de dependência (que falaremos num outro post em breve)]([https://pt.wikipedia.org/wiki/Injeção_de_dependência](https://pt.wikipedia.org/wiki/Inje%C3%A7%C3%A3o_de_depend%C3%AAncia)), ou através de um [*design pattern*]([https://refactoring.guru/pt-br/design-patterns/abstract-factory](https://refactoring.guru/pt-br/design-patterns/abstract-factory)*)* chamado [*Abstract Factory (fábrica abstrata)*]([https://refactoring.guru/pt-br/design-patterns/abstract-factory](https://refactoring.guru/pt-br/design-patterns/abstract-factory)).

Esse *design pattern* especifica um formato que pode ser utilizado para criar e entregar as instâncias dos objetos que precisamos. Essa classe é responsável por conhecer as dependências necessárias para a criação das instâncias, evitando que essa responsabilidade fique nas classes onde não deveriam estar.

Vejamos nosso exemplo, aplicando a interface `Logger`, juntamente com o *design pattern Abstract Factory*:

```kotlin
fun main() {

    val loggerFactory = LoggerFactory()
    val logger = loggerFactory.createTerminalLogger()

    logger.debug("Log de debug")
}

interface Logger {

    fun debug(message: String)
}

class TerminalLogger(
    private val systemProperties: SystemProperties
) : Logger {

    override fun debug(message: String) {
        val systemName = systemProperties.getName()
        println("$systemName - $message")
    }
}

class SystemProperties() {

    fun getName(): String {
        return "Exemplo-DIP"
    }
}

class LoggerFactory() {

    private val systemProperties = SystemProperties()

    fun createTerminalLogger(): TerminalLogger {
        val terminalLogger = TerminalLogger(systemProperties)
        return terminalLogger
    }
}
```

Agora, temos a classe `LoggerFactory` que é responsável por criar e retornar uma instância de `TerminalLogger` através da função `createTerminalLogger()`. Assim, a dependência pela classe `SystemProperties` fica na nossa classe *factory* e não mais na função `main()`.

Essa solução pode deixar uma certa dúvida, já que a classe `LoggerFactory` depende de `SystemProperties`. Isso seria facilmente resolvido utilizando injeção de dependências, conforme citado anteriormente, mas como trata-se de um exemplo simples, essa solução é aceitável, pois apesar de a dependência ainda existir, ela está numa classe que faz mais sentido de estar. A classe `LoggerFactory` é responsável por criar a instância de `TerminalLogger` e, portanto, deve conhecer suas dependências, ao contrário da função `main()` que só precisa utilizar a classe `TerminalLogger`, não importando como ela funciona ou quais suas dependências.

## Conclusão

O Princípio da Inversão de Dependência é um princípio muito utilizado quando falamos de arquitetura limpa, que possui como premissa o uso de abstrações para redução do acoplamento entre as camadas. Podemos ver também que, mesmo que todas as abstrações sejam bem estruturadas, ainda terão casos onde os acoplamentos não serão removidos por completo, como a classe String do Java, citada por Robert C. Martin como exemplo aceitável de dependência direta à implementação.

## Referências

- Arquitetura Limpa: O Guia do Artesão para Estrutura e Design de Software. Martin, Robert C. ; traduzido por Samantha Batista. Rio de Janeiro. Alta Books, 2018.
