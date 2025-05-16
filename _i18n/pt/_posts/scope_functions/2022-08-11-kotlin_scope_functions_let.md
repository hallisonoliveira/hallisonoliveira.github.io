---
layout: post
title: Let
img: kotlin_scope_functions/let.png
category: Kotlin
series: "kotlin_scope_functions"
tags: [
    kotlin,
    scope functions,
    let
]
comments: true
hidden: true
---

Aproveitando o exemplo apresentado na documentação do Kotlin, um uso comum do `let` é o apresentado abaixo:

```kotlin
class Person(
    val name: String,
    var age: Int,
    var city: String
) {
    fun moveTo(city: String) {
        this.city = city
    }

    fun incrementAge() {
        this.age++
    }

    override fun toString(): String {
        return "${this.name}, ${this.age}, ${this.city}"
    }
}

fun main() {
    Person("Alice", 20, "Amsterdam").let {
        println(it.toString()) // Alice, 20, Amsterdam
        it.moveTo("London")
        it.incrementAge()
        println(it.toString()) // Alice, 21, London
    }
}
```

Além do acesso direto aos atributos e funções do objeto, a função ``let`` também pode retornar um valor como resultado da execução do bloco. Nesse caso, a última linha do bloco poderá ser retornada como resultado.
O próximo exemplo demonstra também a possibilidade de nomear o objeto dentro do bloco, removendo a ambiguidade do termo ``it``, que pode causar confusão:

```kotlin
class Person(
    val name: String,
    var age: Int,
    var city: String
) {
    override fun toString(): String {
        return "${this.name}, ${this.age}, ${this.city}"
    }
}

fun main() {
    val personString: String = Person("Pedro", 18, "São Paulo").let { person ->
        // O resultado da última linha do bloco é retornado com resultado do bloco
        // Pode-se acessar os atributos através do nome atribuido para o objeto dentro do bloco
        person.toString()
    }

    println(personString) // Pedro, 18, São Paulo
}
```

Por fim, um uso muito útil do ``let`` é a execução do bloco apenas para objetos não-nulos. Para isso, basta utilizar o [*safe call operator* (operador de chamada segura)](https://kotlinlang.org/docs/null-safety.html#safe-calls) da seguinte forma:

```kotlin
class Person(
    val name: String,
    var age: Int,
    var city: String
) {

    fun incrementAge() {
        this.age++
    }
}

fun main() {
    // declaramos o objeto person como null
    val person: Person? = null

    // Erro, pois o objeto person é nulo nesse ponto
    // person.incrementAge()

    person?.let {
        // Sem problemas, pois garantimos que o objeto person não é nulo com o uso do ?.let{ }
        it.incrementAge()
    }
}
```

## Referências

[Kotlin Scope Functions](https://kotlinlang.org/docs/scope-functions.html). Acesso em 09 de Agosto de 2022.
