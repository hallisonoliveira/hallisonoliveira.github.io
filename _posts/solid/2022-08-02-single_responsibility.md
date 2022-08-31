---
layout: post
title: S - Princípio da Responsabilidade Única
img: solid/single_responsibility.png
category: SOLID
series: solid
tags: [
    solid,
    uncle bob,
    software,
    development,
    single responsibility,
    responsabilidade unica
]
comments: true
hidden: true
---

Em inglês, *Single Responsibility Principle*.

Esse é o princípio que mais causa confusão. Não pela complexidade do princípio em si, mas pelo nome, que pode levar à um entendimento diferente do que o princípio fala.

## A definição equivocada

A principal confusão aqui é que, pelo nome, pode-se dizer que métodos/classes devem fazer apenas uma coisa, e fazê-lo bem feito. Esse é um princípio muito comum e deve sempre ser lembrado durante um processo de refatoração, quando temos funções e classes muito grandes e com muitas responsabilidades. Nesses casos, é importante que tais funções/classes sejam melhor divididas de forma que sejam criadas pequenas unidades funcionais e testáveis. Tais unidades, sejam funções ou classes, devem fazer apenas uma coisa, e fazê-la bem feito. Novamente, esse princípio é importante e faz todo sentido, mas não é essa a definição correta para o Princípio da Responsabilidade Única, conforme o próprio Robert C. Martin fala no livro *Clean Architecture*:

> Não se engane, saiba que há um princípio com esse. Uma função deve fazer uma, e apenas uma, coisa. Usamos esse princípio quando refatoramos funções grandes em funções menores; usamos isso nos níveis mais baixos. Mas ele não é um dos princípios SOLID - não é SRP.
>
- Martin, Robert C. Clean Architecture.

## A definição correta

Ainda segundo Robert C. Martin, a melhor definição para esse princípio seria:

> Um módulo deve ser responsável por um, e apenas um, ator.
>
- Martin, Robert C. Clean Architecture.

Por **ator**, entende-se que podem ser um usuário, um grupo de usuários, *stake holders*, ou qualquer outra pessoa que tenha possibilidade de solicitar uma mudança no projeto.

Por **módulo**, entende-se qualquer conjunto coeso de funções e estruturas, que possam fazer parte de um mesmo escopo e com apenas um ator como responsável.

## Código

Para esse princípio, segui o exemplo apresentado no livro, trazendo para a linguagem Kotlin. Observe a seguinte classe:

```kotlin
class Employee {

    fun calculatePay() { }

    fun reportHours() { }

    fun save() { }
}
```

O Princípio da Responsabilidade Única é quebrado aqui pois temos em uma única classe, três funções pertencentes à contextos diferentes e, portanto, pertencentes à atores diferentes. Abaixo, a representação das funções e seus atores, conforme tratado no livro:

- `calculatePay()` → Departamento de contabilidade, subordinado ao CFO;
- `reportHours()` → Departamento de recursos humanos, subordinado ao COO;
- `save()` → Departamento de TI, administradores de base de dados, subordinados ao CTO.

A violação do princípio qui está no fato de que, caso qualquer um dos atores citados solicite alguma alteração na implementação de sua função, essa classe terá que mudar, podendo inclusive interferir no funcionamento das demais funções dos outros atores. Isso é um problema.

Uma maneira que poderia ser utilizada para resolver esse problema é ter classes específicas para cada contexto e portanto, para cada ator. Assim, qualquer alteração solicitada por qualquer ator, afetará apenas a classe que faz parte do seu contexto. A solução em código poderia ficar da seguinte forma:

```kotlin
fun main() {
    val employee = Employee(
    	salary = 1000.0,
        hours = 220
    )

    // Para o cálculo
    val paymentCalculator = PaymentCalculator()
    paymentCalculator.calculate(employee.salary)

    // Para o reporte de horas
    val hourReporter = HourReporter()
    hourReporter.report(employee.hours)

    // Para salvar o objeto Employee na base de dados
    val employeeSaver = EmployeeSaver()
    employeeSaver.save(employee)
}

// Classe que representa o Employee
data class Employee(
    val salary: Double,
    val hours: Int
)

// Classe responsável por efetuar os cálculos.
class PaymentCalculator() {
    fun calculate(salary: Double): Double { }
}

// Classe responsável por efetuar o reporte de horas
class HourReporter() {
    fun report(hours: Int) { }
}

// Classe responsável por salvar o Employee na base de dados
class EmployeeSaver() {
    fun save(employee: Employee) { }
}
```

No exemplo acima, temos a seguinte relação entre as classes e seus atores:

- `PaymentCalculator` → Departamento de contabilidade, subordinado ao CFO
- `HourReporter` → Departamento de recursos humanos, subordinado ao COO
- `EmployeeSaver` → Departamento de TI, administradores de base de dados, subordinados ao CTO

Perceba que, para cada contexto (ou ator), temos uma classe correspondente e nenhuma dessas classes possui referência para alguma outra. Ou seja, caso um ator solicite uma alteração na implementação da sua função, as classes dos outros atores não precisarão ser alteradas, mantendo os contextos coesos. Num caso real, a comunicação entre as classes poderia ser possível, desde que sejam tomados alguns cuidados. Mas para simplificar o exemplo, vamos considerar que isso não poderia acontecer.

A única classe que pode ser conhecida por um ou mais contextos é a classe `Employee` que é a representação de um empregado no contexto do projeto.

Outro detalhe importante a ser mencionado é o acréscimo de código ao projeto quando nos preocupamos com esse princípio, o que pode representar de início, mais complexidade. Porém, analisando mais a fundo, isso não é verdade, pois essa separação cria contextos específicos para cada ator. Além disso, cada função terá mais implementações auxiliares (funções privadas dentro das classes), o que justifica ainda mais a necessidade de separação em classes específicas.

## Conclusão

O Princípio da Responsabilidade Única é a base para a criação de classes com contexto definido e de uma forma que os limites entre as classes seja respeitado. Isso nos dá a possibilidade de reaproveitar essas classes em outros pontos do projeto. Além disso, classes coesas e divididas de acordo com suas responsabilidades contribuem para facilitar o desenvolvimento de testes unitários, outro tópico muito importante no desenvolvimento de software.

## Referências

- Arquitetura Limpa: O Guia do Artesão para Estrutura e Design de Software. Martin, Robert C ; traduzido por Samantha Batista. Rio de Janeiro. Alta Books, 2018.
