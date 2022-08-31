---
layout: post
title: O - Princípio Aberto/Fechado
img: solid/open_close.png
category: SOLID
series: solid
tags: [
    solid,
    uncle bob,
    software,
    development,
    open close,
    aberto fechado
]
comments: true
hidden: true
---
Em inglês, *Open Closed Principle*.

A definição desse princípio, segundo seu criador Bertrand Meyer, é a seguinte:

> Um artefato de software deve ser aberto para extensão, mas fechado para modificação.
>

Ok, mas o que isso significa na prática?

Decompondo um pouco essa definição, podemos dividi-la em duas partes:

- Aberto para extensão significa que um módulo pode ser alterado para receber novos atributos às suas estruturas de dados existentes ou então, novas funções podem ser adicionadas.
- Fechado para modificação significa que esse módulo específico já está bem definido e não há mais alteração alguma para fazer nele. Isso significa que ele está estável o suficiente para que outros módulos possam utilizá-lo.

Tais definições parecem simples de entender. Porém, quando se trata de desenvolvimento de software, o cenário pode ser um tanto complexo, já que raramente um software é desenvolvido, concluído e não receberá mais modificações.

No *post* em seu blog sobre esse princípio, Robert C. Martin cita uma nova definição:

> Você deve ser capaz de estender o comportamento de um sistema sem a necessidade de modificar tal sistema.
>

Ou seja, durante o processo de desenvolvimento, devemos pensar nos módulos, classes, funções, etc, de uma forma que seja possível adicionar novos comportamentos à eles sem que o código já existente seja modificado. Isso traz extensão (aberto para extensão), mas mantém o que já existe inalterado, evitando que *bugs* ou novos comportamentos não mapeados sejam introduzidos às funcionalidades já existentes (fechado para modificação).

## Estudo de caso: Logger

Imagine que temos uma aplicação qualquer que possui a necessidade de registrar alguns *logs* de seu funcionamento. De início, esses *logs* serão apresentados apenas no terminal onde o sistema está sendo executado. O código poderia ficar semelhante ao apresentado a seguir:

```kotlin
fun main() {
    val logger = Logger()

    logger.debug("Log de debug")
}

class Logger {
    fun debug(message: String) {
        println(message)
    }
}
```

```bash
Log de debug
```

Agora, precisamos que a classe `Logger` registre os *logs* em arquivo também. Para isso, será necessário adicionar um parâmetro novo à implementação da classe e também será necessário fazer com que a função `debug(message: String)` envie o *log* para o arquivo. A implementação poderia ficar assim:

```kotlin
fun main() {
    val logger = Logger()

    logger.debug("Log de debug")
}

class Logger {

    private val fileLogger = FileLogger()

    fun debug(message: String) {
        fileLogger.writeLogToFile(message)
        println(message)
    }
}

class FileLogger {
    private val fileName = "/var/log/example.log"
    private val logFile = File(fileName)

    fun writeLogToFile(message: String) {
        logFile.bufferedWriter().use { out ->
            out.write(message)
        }
    }
}
```

Perceba que foi necessário criar a classe `FileLogger` para fazer a gravação dos *logs* em arquivo e alterar a classe ``Logger`` já existente para fazer uso da nova classe.

Foi preciso também alterar a função `debug(message: String)` para que ela chame a função `writeLogToFile(message: String)` da classe `FileLogger` e assim, efetuar a gravação da mensagem no arquivo.

Nesse exemplo, pode-se ver claramente a quebra do Princípio Aberto/Fechado, pois foi necessário alterar implementações já existentes, como foi o caso da classe `Logger`, para que uma nova funcionalidade fosse adicionada. Ou seja, ela está aberta para extensão, mas não está fechada para alteração.

Vejamos agora esse mesmo exemplo sem que esse princípio seja violado:

```kotlin
fun main() {
    val logger: Logger = TerminalLogger()

    logger.debug("Log de debug")
}

interface Logger {
    fun debug(message: String)
}

class TerminalLogger : Logger {
    override fun debug(message: String) {
        println(message)
    }
}
```

Agora, temos a interface `Logger` que serve de contrato para qualquer tipo de *logger* que seja necessário no projeto. Assim, se precisarmos alterar a forma de registrar os *logs*, não será necessário alterar nenhuma implementação já existente, desde que os novos *loggers* sigam esse contrato.

Para exibir os logs no terminal, criamos a classe `TerminalLogger`, que implementa a interface `Logger`, que será responsável por enviar as mensagens de *log* para o terminal.

Para ilustrar o cenário onde seria necessário adicionar uma nova funcionalidade ao sistema, vamos trocar o *log* no terminal para *log* em arquivo, adicionando um novo *logger*.

```kotlin
fun main() {
    // Trocamos TerminalLogger por FileLogger
    val logger: Logger = FileLogger()

    logger.debug("Log de debug")
}

interface Logger {
    fun debug(message: String)
}

// Não utilizado nesse exemplo
class TerminalLogger : Logger {
    override fun debug(message: String) {
        println(message)
    }
}

class FileLogger : Logger {
    private val fileName = "/var/log/example.log"
    private val logFile = File(fileName)

    override fun debug(message: String) {
        logFile.bufferedWriter().use { out ->
            out.write(message)
        }
    }
}
```

Perceba que a classe `FileLogger` também implementa a interface `Logger`, ou seja, ela também segue o mesmo contrato utilizado para a classe `TerminalLogger`. Isso quer dizer que, mesmo trocando `TerminalLogger` por `FileLogger` na função `main()`, o uso permanece o mesmo. Além disso, mantive a classe `TerminalLogger` no exemplo, mesmo ela não sendo utilizada, para mostrar que ela não precisou ser alterada.

Agora, e se fosse necessário que os logs fossem registrados tanto no terminal, quanto em arquivo? Vejamos:

```kotlin
fun main() {
    val logger: Logger = TerminalAndFileLogger()

    logger.debug("Log de debug")
}

interface Logger {
    fun debug(message: String)
}

class TerminalLogger : Logger {
    override fun debug(message: String) {
        println(message)
    }
}

class FileLogger : Logger {
    private val fileName = "/var/log/example.log"
    private val logFile = File(fileName)

    override fun debug(message: String) {
        logFile.bufferedWriter().use { out ->
            out.write(message)
        }
    }
}

class TerminalAndFileLogger : Logger {
    private val terminalLogger = TerminalLogger()
    private val fileLogger = FileLogger()

    override fun debug(message: String) {
        terminalLogger.debug(message)
        fileLogger.debug(message)
    }
}
```

Temos agora a classe `TerminalAndFileLogger` que, como o próprio nome mostra, será responsável por chamar `TerminalLogger` e `FileLogger` e ela também implementa a interface `Logger`, fazendo com que ela siga o mesmo contrato das demais.

Essa forma de implementação fica interessante pois o uso de todos os *loggers* permanece o mesmo. Assim, a forma de chamá-la na função `main()` não mudou em relação às outras. Além disso, mesmo adicionando uma nova forma de registro de *logs* no sistema, terminal e arquivo juntos, o código dos demais loggers não foi alterado, mantendo eles fechados para alterações.

## Conclusão

Esse princípio é muito interessante e importante para o desenvolvimento de um sistema, pois tendo ele em mente, acabamos por estruturar o projeto de uma forma que facilite futuras implementações, reduzindo o esforço para adicionar uma nova funcionalidade ou até mesmo, alterar as funcionalidades já existentes. Além disso, tivemos oportunidade de aplicar herança e polimorfismo, dois conceitos muito importantes da Orientação à Objetos (logo teremos *posts* sobre eles).

## Referências

- Arquitetura Limpa: O Guia do Artesão para Estrutura e Design de Software. Martin, Robert C. ; traduzido por Samantha Batista. Rio de Janeiro. Alta Books, 2018.
- [The Open Closed Principle. Martin, Robert C. 12 de maio de 2014](http://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html)
