---
layout: post
title: Kotlin Scope Functions
permalink: /posts/kotlin_scope_functions.html
img: kotlin_scope_functions/kotlin_scope_functions.png
category: Kotlin
series_main: "kotlin_scope_functions"
tags: [
    kotlin,
    scope functions
]
comments: true
---

A biblioteca padrão do Kotlin possui algumas funções com o propósito de criar blocos de código dentro do contexto de um determinado objeto. Esses blocos temporários de código são criados utilizando [expressões lambda](https://kotlinlang.org/docs/lambdas.html) e permitem que os atributos do objeto em questão sejam acessados sem a necessidade de utilizar o nome deles.

Essas funções são chamadas de *Scope Functions* (ou Funções de Escopo), e atualmente há 5 delas: `let`, `run`, `with`, `apply`  e `also`.

Essencialmente, elas possuem o mesmo funcionamento. A diferença está na forma como o objeto utilizado se torna disponível dentro do bloco de código e qual é o resultado da execução de todo o bloco.

> 💡 Todos os exemplos apresentados podem ser executados na página de *playground* do Kotlin, em [https://play.kotlinlang.org/](https://play.kotlinlang.org/)

A medida que os artigos forem sendo publicados, eles serão listados abaixo:
