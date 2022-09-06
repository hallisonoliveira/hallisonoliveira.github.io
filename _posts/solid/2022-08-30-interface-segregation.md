---
layout: post
title: I - Princípio da Segregação de Interface
img: solid/interface_segregation.png
category: SOLID
series: solid
tags: [
    solid,
    uncle bob,
    software,
    development,
    interface segregation,
    segregação de interface
]
comments: true
hidden: true
---

Em inglês, *Interface Segregation Principle*.

# Interfaces

Antes de falarmos sobre esse princípio, é importante entendermos o que é interface:

> Uma interface é uma série de abstrações ou um contrato ao qual uma determinada classe, que implementa essa interface, deve seguir. Nas interfaces, são definidos os comportamentos, ou seja, as funções e atributos que compõem o contrato, mas nada é implementado de fato.

Assim sendo, considere seguinte interface:

```kotlin
interface Cachorro {
    val cor: String

    fun latir()
}
```

Perceba que nessa interface definimos duas coisas: um atributo `cor` e uma ação `latir()`. Porém, não foi definida uma cor e a função `latir` não possui implementação. Isso significa que esse é apenas o contrato obrigatório para todas as classes que implementam essa interface.

Assim, quanto implementamos uma classe, esse contrato deve ser seguido e as implementações deverão ser feitas:

```kotlin
class Beagle : Cachorro {
    override val cor: String = "Branco"

    override fun latir() {
        println("Latindo...")
    }
}
```

# O princípio

Entendido o uso de interfaces, vamos analisar o Princípio de Segregação de Interface. Basicamente, esse princípio fala que as classes clientes, ou seja, as classes que implementam interfaces, não deveriam implementar comportamentos que elas não precisam. Isso quer dizer que precisamos definir nossas interfaces de forma que o conjunto de funções e atributos nela definidos, seja coeso, faça sentido.

Vejamos a seguinte interface. Nela, estamos definindo um contrato a ser seguido por classes que representarão meios de transporte.

```kotlin
interface Transporte {

    val quantidadeDeRodas: Int

    fun ligarMotor()

    fun andar()
}
```

Vamos criar uma classe que implementa essa interface.

```kotlin
class Carro : Transporte {

    override val quantidadeDeRodas: Int = 4

    override fun ligarMotor() {
        println("Motor ligado.")
    }

    override fun andar() {
        println("Andando...")
    }
}
```

Até aqui, tudo bem! Estamos definindo uma classe `Carro` que consegue seguir por completo o contrato definido pela interface `Transporte`.

Agora, imagine que precisamos aumentar as opções de meio de transporte do nosso sistema, adicionando a opção Bicicleta:

```kotlin
class Bicicleta : Transporte {

    override val quantidadeDeRodas: Int = 2

    override fun ligarMotor() {
        // Não implementada. Bicicleta não tem motor.
    }

    override fun andar() {
        println("Andando...")
    }
}
```

No caso da classe `Bicicleta`, nem todo o contrato pôde ser seguido, pois bicicletas não possuem motor (considerando a bicicleta normal e não a elétrica), e portando, a função `ligarMotor()` não faz sentido para ela.

Sendo assim, temos o Princípio de Segregação de Interface sendo violado na classe `Bicicleta`, pois a interface `Transporte` está definindo um comportamento que a classe não precisa e/ou não pode ter.

Uma alternativa para esse caso, seria redefinir essa interface, melhorando nível de abstração de forma que o princípio não seja quebrado:

```kotlin
interface TransporteSemMotor {

    val quantidadeDeRodas: Int

    fun andar()
}

interface TransporteComMotor {

    val quantidadeDeRodas: Int

    fun ligarMotor()

    fun andar()
}

class Carro : TransporteComMotor {

    override val quantidadeDeRodas: Int = 4

    override fun ligarMotor() {
        println("Motor ligado.")
    }

    override fun andar() {
        println("Andando...")
    }
}

class Bicicleta : TransporteSemMotor {

    override val quantidadeDeRodas: Int = 4

    override fun andar() {
        println("Andando...")
    }
}
```

Agora, temos duas interfaces, uma para representar meios de transportes motorizados e outra para representar meios de transportes sem motor.

É possível perceber também que ambas as interfaces possuem o atributo `quantidadeDeRodas` e a função `andar()`. Isso para o nosso exemplo é aceitável, já que é um exemplo simples. Mas, para um projeto real, que precisa ser escalável e ter suas interfaces muito bem definidas, talvez essa solução não seja interessante e a revisão dessas abstrações deva se feita com mais cuidado.

# Conclusão

O Princípio de Segregação de Interface é um princípio simples e fácil de ser violado ao adicionar nas interfaces comportamentos e atributos além do necessário. Por outro lado, a tentativa de corrigir essa violação pode fazer com que as abstrações sejam muito específicas, gerando um número de interfaces maior do que o necessário. Isso traz a reflexão de que, quando vamos definir abstrações para os nossos projetos, precisamos analisar com cuidado para que não criemos interfaces muito genéricas e nem muito específicas.
