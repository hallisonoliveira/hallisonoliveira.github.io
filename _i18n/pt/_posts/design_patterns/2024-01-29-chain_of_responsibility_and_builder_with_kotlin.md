---
layout: post
title: Chain Of Responsibility e Builder - Um exemplo prático em Kotlin
permalink: /posts/chain_of_responsibility_and_builder_with_kotlin.html
img: chain_of_responsibility_and_builder.png
category: Design Patterns
series_main: "chain_of_responsibility_and_builder"
tags: [
    kotlin,
    design pattern,
    padrões de projeto,
    chain of responsibility,
    cadeia de responsabilidade,
    builder
]
comments: true
---

# Os Design Patterns

## Chain Of Responsibility

O ***Chain of Responsibility*** (Cadeia de Responsabilidade) é um padrão de projeto que permite organizar objetos em uma sequência hierárquica, onde cada objeto na cadeia tem a capacidade de processar uma solicitação ou passá-la para o próximo nível da hierarquia. Esse padrão é útil quando múltiplos objetos podem lidar com uma solicitação, mas não se sabe antecipadamente qual deles será responsável por processá-la.

Para facilitar o entendimento, podemos ver esse padrão como uma corrente, onde cada elo dessa corrente possui uma referência para o próximo elo e um método para processar a solicitação. Essa solicitação percorre cada elo da corrente até que seja processada por um deles ou chegue ao final da cadeia sem ser tratada. Isso proporciona flexibilidade e desacopla o remetente da solicitação do seu destinatário, tornando o sistema mais modular e fácil de manter.

No exemplo apresentado aqui, cada elo tem uma função que indicará se ele deverá ser executado e uma função com a lógica a ser aplicada. Se o elo deve ser executado, a função com a lógica é chamada. Se o elo não puder ser executado, a execução da corrente continua e o próximo elo será acionado. Há também uma classe responsável por manter a instância de cada elo da corrente e por chamar todos os elos. Se um dos elos for executado, a classe da corrente interrompe a execução e retorna o resultado para quem à chamou.

Esse padrão é muito interessante e é frequentemente utilizado em situações em que há múltiplos manipuladores para um mesmo tipo de solicitação, como em sistemas de processamento de eventos, validações e/ou cálculos para diferentes tipos de dados ou em casos em que o comportamento do sistema pode ser estendido sem a necessidade de modificar o código existente, sendo um recurso muito útil já que precisamos ter sempre em mente os princípios <a href="{{ site.baseurl }}/posts/solid.html">SOLID</a>.

## Builder

O padrão ***Builder*** é um padrão de projeto voltado para a construção de objetos complexos. Ele é útil quando a construção de determinado objeto exige uma grande quantidade de informações ou envolve uma série de lógicas com diversas etapas. Assim, nesses casos, é possível abstrair toda a complexidade da criação do objeto para uma classe separada, responsável apenas pelo processo de criação.

No exemplo apresentado aqui, utilizamos esse padrão para a construção da classe que gerencia a cadeia de regras. Através do ***Builder***, adicionamos as regras à cadeia, com possibilidade de adicionar uma regra padrão, e ao final, recebemos a instância da classe da cadeia.

# Vamos ao código

Para exemplificar o uso dos padrões ***Chain Of Responsibility*** e ***Builder***, trago um sistema simples de cálculo de preço de venda de veículos.

Como entrada para esse sistema, fornecemos um determinado veículo com um tipo (carro ou pickup), um fabricante (Chevrolet ou Ford) e um preço de tabela. As regras de cálculo serão aplicadas ao preço de tabela e resultarão num preço final de venda, a ser definido via regras de cálculo.

Para a representação do veículo, temos o *data class* `Vehicle`, com o atributo `type` (*Enum*), que indica se o veículo é um carro (`CAR`), uma pickup (`PICKUP`) ou um ônibus (`BUS`) e o atributo `brand` indicando o fabricante, conforme a seguir:

```kotlin
data class Vehicle(
    val value: Double,
    val brand: Brand,
    val type: Type
)

enum class Type {
    CAR,
    PICKUP,
    BUS
}

enum class Brand {
    CHEVROLET,
    FORD
}
```

Cada combinação de tipo de veículo e fabricante terá uma regra específica para o cálculo de preço de venda. Usaremos o padrão ***Chain Of Responsibility*** para gerenciar a aplicação dessas regras. Assim, para cada par tipo + fabricante, teremos uma implementação da classe de regra.

As classes de regra devem seguir o formato definido pela classe abstrata `SellValueCalculationRule`, que pode ser vista abaixo:

```kotlin
abstract class SellValueCalculationRule {

    protected abstract fun shouldCalculate(
        vehicle: Vehicle,
    ): Boolean

    protected abstract fun doCalculation(
        vehicle: Vehicle,
    ): Double

    fun run(
        vehicle: Vehicle,
    ) = if (shouldCalculate(vehicle)) {
        doCalculation(vehicle)
    } else {
        null
    }
}
```

Essa classe possui três funções:

- `shouldCalculate` - Essa função recebe um objeto `Vehicle` como parâmetro e retorna um *Boolean*. É nela onde colocamos a lógica que define se a regra deve ser executada ou não. Se a regra deve ser executada, essa função deve retornar `true`, se não, ela deve retornar `false`. Perceba que ela está marcada como `protected`, o que indica que ela não pode ser acessada externamente.
- `doCalculation` -  Essa função recebe um objeto `Vehicle` como parâmetro e retorna um *Double*. É nela onde colocamos a implementação da lógica de cálculo da regra. Ou seja, é nessa função onde o cálculo do preço de venda será feito. Perceba que, assim como a função `shouldCalculate`, essa função também está marcada como `protected`.
- `run` - Essa função recebe um objeto `Vehicle` como parâmetro e retorna um *Double nullable*. Essa é a única função que poderá ser acessada externamente e que não terá sua implementação feita pelas classes de regras. Ela faz parte da estrutura da cadeia e é responsável por chamar a função `shouldExecute` para verificar se a regra deve ser executada. Se sim, a função `doCalculation` é chamada e seu resultado é retornado. Se a função `shouldExecute` retornar `false` indicando que a regra não deve ser executada, a função `run` retornará `null`, indicando para a classe `SellValueCalculationChain` que a próxima regra deve ser chamada.

A seguir, temos as implementações concretas das regras de cálculo de preço de venda para cada combinação de tipo e fabricante de veículo.

Para veículos do tipo `CAR` e fabricante `CHEVROLET`:

```kotlin
class ChevroletCarSellValueCalculationRule : SellValueCalculationRule() {

    override fun shouldCalculate(
        vehicle: Vehicle,
    ) = vehicle.type == Type.CAR && vehicle.brand == Brand.CHEVROLET

    override fun doCalculation(vehicle: Vehicle): Double {
        return vehicle.value + ((vehicle.value * CHEVROLET_CAR_INTEREST_VALUE) / 100)
    }

    companion object {
        private const val CHEVROLET_CAR_INTEREST_VALUE = 0.99
    }
}
```

Para veículos do tipo `PICKUP` e fabricante `CHEVROLET`: 

```kotlin
class ChevroletPickupSellValueCalculationRule : SellValueCalculationRule() {

    override fun shouldCalculate(
        vehicle: Vehicle,
    ) = vehicle.type == Type.PICKUP && vehicle.brand == Brand.CHEVROLET

    override fun doCalculation(vehicle: Vehicle): Double {
        return vehicle.value +
                ((vehicle.value * CHEVROLET_PICKUP_INTEREST_PERCENTAGE_VALUE) / 100) -
                CHEVROLET_PICKUP_DISCOUNT_MONETARY_VALUE
    }

    companion object {
        private const val CHEVROLET_PICKUP_INTEREST_PERCENTAGE_VALUE = 0.49
        private const val CHEVROLET_PICKUP_DISCOUNT_MONETARY_VALUE = 1500.00
    }
}
```

Para veículos do tipo `CAR` e fabricante `FORD`:

```kotlin
class FordCarSellValueCalculationRule : SellValueCalculationRule() {

    override fun shouldCalculate(
        vehicle: Vehicle,
    ) = vehicle.type == Type.CAR && vehicle.brand == Brand.FORD

    override fun doCalculation(
        vehicle: Vehicle,
    ) = vehicle.value + ((vehicle.value * CHEVROLET_CAR_INTEREST_VALUE) / 100)

    companion object {
        private const val CHEVROLET_CAR_INTEREST_VALUE = 0.49
    }
}
```

Para veículos do tipo `PICKUP` e fabricante `FORD`:

```kotlin
class FordPickupSellValueCalculationRule : SellValueCalculationRule() {

    override fun shouldCalculate(
        vehicle: Vehicle,
    ) = vehicle.type == Type.PICKUP && vehicle.brand == Brand.FORD

    override fun doCalculation(
        vehicle: Vehicle
    ) = vehicle.value +
            ((vehicle.value * FORD_PICKUP_INTEREST_PERCENTAGE_VALUE) / 100) -
            FORD_PICKUP_DISCOUNT_MONETARY_VALUE

    companion object {
        private const val FORD_PICKUP_INTEREST_PERCENTAGE_VALUE = 0.69
        private const val FORD_PICKUP_DISCOUNT_MONETARY_VALUE = 2000.00
    }
}
```

Por fim, vamos criar uma regra *default* para demonstrar seu funcionamento:

```kotlin
class DefaultDiscountCalculationRule : SellValueCalculationRule() {

    override fun shouldCalculate(
        vehicle: Vehicle,
    ) = true

    override fun doCalculation(
        vehicle: Vehicle
    ) = vehicle.value - ((vehicle.value * GENERAL_DISCOUNT_PERCENTAGE_VALUE) / 100)

    companion object {
        private const val GENERAL_DISCOUNT_PERCENTAGE_VALUE = 0.1
    }
}
```

Perceba que na função `shouldCalculate` de cada regra, é verificado o tipo do veículo e o fabricante, criando a pré condição para que a regra seja executada. Já na classe da regra *default*, essa função retorna `true` diretamente. Assim, garantimos que ela sempre vai executar. Devemos apenas garantir que essa regra seja sempre utilizada apenas como regra *default*. Se ela for adicionada com as demais regras, a execução da cadeia terá um comportamento inválido, já que ao passar por essa regra, ela sempre será executada, fazendo com que as demais regras - definidas após ela - sejam ignoradas.

Para todas as regras, temos o cálculo em si definidos na função `doCalculation`.

Por fim, podemos ver que a função `run` não está presente em nenhuma implementação concreta de regra.

Com as regras implementadas, podemos passar para a classe `SellValueCalculationChain`, responsável por gerenciar a cadeia de regras. Ela é a representação da corrente de regras e é dentro dela que as regras - os elos da corrente - serão armazenados.

Sua implementação pode ser vista a seguir:

```kotlin
class SellValueCalculationChain private constructor() {

    private val calculationRules: MutableList<SellValueCalculationRule> = mutableListOf()
    private var defaultCalculationRule: SellValueCalculationRule? = null

    fun addRule(rule: SellValueCalculationRule) {
        calculationRules.add(rule)
    }

    fun setDefaultRule(rule: SellValueCalculationRule) {
        defaultCalculationRule = rule
    }

    fun run(vehicle: Vehicle): Double {

        calculationRules.forEach { calculationRule ->
            val calculationResult = calculationRule.run(vehicle)
            calculationResult?.run { return this }
        }

        defaultCalculationRule?.run {
            val defaultCalculationResult = run(vehicle)
            defaultCalculationResult?.run { return this }
        }

        return vehicle.value
    }

    object Builder {

        private val rules: MutableList<SellValueCalculationRule> = mutableListOf()
        private var defaultRule: SellValueCalculationRule? = null

        fun add(rule: SellValueCalculationRule) = apply {
            rules.add(rule)
        }

        fun setDefault(rule: SellValueCalculationRule) = apply {
            defaultRule = rule
        }

        fun build(): SellValueCalculationChain {
            val chain = SellValueCalculationChain()
            rules.forEach(chain::addRule)
            defaultRule?.run(chain::setDefaultRule)
            return chain
        }
    }
}
```

Nessa classe temos diversos pontos interessantes a serem vistos:

Construtor privado para garantirmos que essa classe não será instanciada sem o uso do *Builder* (explicado a seguir).

Lista `calculationRules` que representa as regras pertencentes à cadeia, ou seja, os elos da corrente.

Função `addRule` que será utilizada para adicionar as regras à cadeia, uma por uma. Ao chamar a essa função, a regra passada por parâmetro será adicionada ao fim lista `calculationRules`.

Objeto `defaultCalculationRule` que representa a regra *default* da cadeia. Se nenhuma regra presente na lista `calculationRules` for executada, a regra *default* será executada, se estiver definida (≠ null).

Função `setDefaultRule` responsável por definir a regra *default* da cadeia. Ao chamar essa função, a regra passada por parâmetro será definida como valor do objeto `defaultCalculationRule`. Como podemos ter apenas uma regra *default*, se a função for chamada mais de uma vez, o valor do objeto será substituído.

Função `run`. Essa função recebe um objeto `Vehicle` e retorna um *Double* e é responsável por chamar todas as regras presentes na lista `calculationRules`, uma após a outra, na sequência em que foram adicionadas.

- Se alguma regra for executada (retornar um valor válido diferente de `null`), esse valor sera retornado pela função `run` imediatamente;
- Se nenhuma regra for executada, a regra *default* será chamada e seu resultado será retornado, se ela estiver definida;
- Se nenhuma regra da lista `calculationRules` for executada e não houver uma regra *default* definida, então apenas retornamos o valor de tabela do veículo recebido como parâmetro.

Dentro da classe `SellValueCalculationChain`, temos a classe *Builder*. Como o nome sugere, essa classe é a implementação do *design pattern Builder* e é ela a responsável por construir a classe `SellValueCalculationChain`. Nela também temos uma lista de regras (`rules`) e um objeto que representa a regra *default (`defaultRule`)*.

A lista `rules` é uma lista temporária que serve apenas para armazenar as regras durante o processo de criação da instância da classe da cadeia.

Assim como a lista `rules`, o objeto `defaultRule` também é temporário e serve apenas para armazenada a regra *default* - se usada - durante o processo de criação da classe da cadeia.

Temos as funções `add` para adicionar as regras e `setDefault` para definir a regra *default*.

A função principal aqui é a função `build`. Ela não recebe parâmetro algum e retorna uma instância da classe `SellValueCalculationChain`. É nela onde está toda a lógica para a criação da classe da cadeia. Quando chamada, ela cria uma instância da classe, adiciona todas as regras presentes na lista `rules` do *Builder* à instância da classe da cadeia, uma por uma e, em seguida, a função define a regra *default*, se ela estiver definida. Por fim, a função retorna a instância recém criada da classe `SellValueCalculationChain` com todas as regras adicionadas.

Abaixo, temos uma função `main()` para demonstrar o sistema de cálculo de preço de venda em funcionamento:

```kotlin
fun main() {
    println("Sistema de cálculo de preço de venda")

    /** Criação dos veículos para serem utilizados nos cálculos **/
    val chevroletCamaro = Vehicle(
        type = Type.CAR,
        brand = Brand.CHEVROLET,
        value = 150000.00
    )

    val chevroletSilverado = Vehicle(
        type = Type.PICKUP,
        brand = Brand.CHEVROLET,
        value = 190000.00
    )

    val fordMaverick = Vehicle(
        type = Type.CAR,
        brand = Brand.FORD,
        value = 155000.00
    )

    val fordRanger = Vehicle(
        type = Type.PICKUP,
        brand = Brand.FORD,
        value = 185000.00
    )

    val fordBus = Vehicle(
        type = Type.BUS,
        brand = Brand.FORD,
        value = 200000.00
    )

    /** Criação da cadeia de regras de cálculos **/
    val sellCalculationChain = SellValueCalculationChain.Builder
        .add(ChevroletCarSellValueCalculationRule())
        .add(ChevroletPickupSellValueCalculationRule())
        .add(FordCarSellValueCalculationRule())
        .add(FordPickupSellValueCalculationRule())
        .setDefault(DefaultDiscountCalculationRule())
        .build()

    /** Calculo de venda do Chevrolet Camaro **/
    val chevroletCamaroSellValue = sellCalculationChain.run(chevroletCamaro)
    println("Chevrolet Camaro\n\t- Preço de tabela: ${chevroletCamaro.value}\n\t- Preço de venda: ${chevroletCamaroSellValue}\n")
    /**
     * Chevrolet Camaro
     * 	- Preço de tabela: 150000.0
     * 	- Preço de venda: 151485.0
     */

    /** Calculo de venda do Chevrolet Silverado **/
    val chevroletSilveradoSellValue = sellCalculationChain.run(chevroletSilverado)
    println("Chevrolet Silverado\n\t- Preço de tabela: ${chevroletSilverado.value}\n\t- Preço de venda: ${chevroletSilveradoSellValue}\n")
    /**
     * Chevrolet Silverado
     * 	- Preço de tabela: 190000.0
     * 	- Preço de venda: 189431.0
     */

    /** Calculo de venda do Ford Maverick **/
    val fordMaverickSellValue = sellCalculationChain.run(fordMaverick)
    println("Ford Maverick\n\t- Preço de tabela: ${fordMaverick.value}\n\t- Preço de venda: ${fordMaverickSellValue}\n")
    /**
     * Ford Maverick
     * 	- Preço de tabela: 155000.0
     * 	- Preço de venda: 155759.5
     */

    /** Calculo de venda do Ford Ranger **/
    val fordRangerSellValue = sellCalculationChain.run(fordRanger)
    println("Ford Ranger\n\t- Preço de tabela: ${fordRanger.value}\n\t- Preço de venda: ${fordRangerSellValue}\n")
    /**
     * Ford Ranger
     * 	- Preço de tabela: 185000.0
     * 	- Preço de venda: 184276.5
     */

    /** Calculo de venda do onibus Ford **/
    val fordBusSellValue = sellCalculationChain.run(fordBus)
    println("Onibus Ford\n\t- Preço de tabela: ${fordBus.value}\n\t- Preço de venda: ${fordBusSellValue}\n")
    /**
     * Onibus Ford
     * 	- Preço de tabela: 200000.0
     * 	- Preço de venda: 199800.0
     */
}
```

Para demonstrar o funcionamento do sistema, primeiro criamos os objetos que representam os veículos, através do *data class* `Vehicle`.

Em seguida, criamos a instância da classe `SellValueCalculationChain` através do *Builder*, adicionando as classes de regras. Perceba que, para cada regra, chamamos a função `add`. Temos também a definição da regra *default* através da função `setDefault`.

```kotlin
val sellCalculationChain = SellValueCalculationChain.Builder
        .add(ChevroletCarSellValueCalculationRule())
        .add(ChevroletPickupSellValueCalculationRule())
        .add(FordCarSellValueCalculationRule())
        .add(FordPickupSellValueCalculationRule())
        .setDefault(DefaultDiscountCalculationRule())
        .build(
```

Com a instância da classe `SellValueCalculationChain` criada, podemos executar os cálculos de preço de venda para os veículos criados anteriormente. Para isso, basta chamar a função `run` passando veículo como parâmetro. O resultado será o preço de venda do veículo. Para informação, podemos comparar o preço de tabela com o preço de venda através dos comentários adicionados abaixo de cada chamada. 

O último veículo que teve seu preço de venda calculado é o objeto `fordBus`. Esse veículo é do tipo `BUS` e do fabricante `FORD`. Para essa combinação de tipo + fabricante, nós não criamos nenhuma classe de regra. Assim, o cálculo do preço de venda para esse veículo foi feito através da regra *default*. Se essa regra não estivesse definida, teríamos como retorno o preço de tabela do veículo.

Perceba que, em nenhum momento, nós vinculamos os veículos às regras de cálculo e nem informamos diretamente qual regra deve ser executada para determinado veículo. Ou seja, quem chama a função `run` da classe da cadeia de regras, não tem conhecimento sobre qual regra será executada, nem como essas regras foram implementadas. Assim, a função `main` não possui a responsabilidade de saber como efetuar o cálculo do preço de venda dos veículos, ela só precisa saber que precisa chamar a função `run` passando o veículo a ser calculado como parâmetro e esperar o resultado do tipo *Double*.

# Algumas considerações pessoais

Gostaria de citar aqui alguns pontos que eu considero importantes quanto à implementação desses padrões de projeto.

## Chain Of Responsibility

- A premissa principal desse padrão é a possibilidade de adicionar novos elos à cadeia sem que seja necessário alterar os elos já existentes ou alterar a estrutura da cadeia. Tendo isso em mente, conseguimos respeitar o <a href="{{ site.baseurl }}/posts/open_close_principle.html">Princípio Aberto/Fechado</a> do <a href="{{ site.baseurl }}/posts/solid.html">SOLID</a>;
- Sobre a implementação das classes dos elos da cadeia, é fundamental que o contrato definido para os elos (classe abstrata) seja bem desenhado, pois ele deve conseguir atender à todos os cenários que a cadeia vai ter. Porém, devemos sempre nos preocupar em não tentar torna-la “completa demais” e criar funções e variáveis além do realmente necessário. Não podemos esquecer do <a href="{{ site.baseurl }}/posts/interface_segregation.html">Princípio da Segregação de Interface</a>, o “I” do do <a href="{{ site.baseurl }}/posts/solid.html">SOLID</a>;
- Quanto aos elos da cadeia, é fundamental que toda a lógica referente àquele elo esteja dentro da classe de implementação do elo ou em classes auxiliares. Um elo da cadeia não pode depender de outro em termos de lógica;
- Ao implementar esse padrão de projeto, a estrutura não pode permitir que os dados que trafegam entre os elos da cadeia sejam alterados. Ou seja, os dados não podem ser alterados em um elo e repassados para outro elo. A partir do momento que um elo da cadeia é executado, a execução da cadeia deve ser interrompida;
- A responsabilidade da função `shouldCalculate` dos elos é única e exclusivamente decidir se o elo deve ser executado ou não. Dentro dessa função, devemos ter o mínimo de lógica possível para tomar essa decisão; 
- Não podemos esquecer dos testes unitários. Como temos classes pequenas e de escopo muito bem definido para os elos da cadeia, é possível testá-las tranquilamente;
- No projeto apresentado aqui, utilizei a classe `SellValueCalculationChain` como responsável pelo controle da chamada dos elos. Na definição desse padrão, essa classe não existe. Ao invés disso, cada elo é responsável por conhecer o próximo elo e chamá-lo quando necessário. Eu, pessoalmente, prefiro ter uma classe que faça esse controle. Assim, os elos deixam de ter a responsabilidade de conhecer e chamar o próximo elo, ficando apenas com a responsabilidade de “decidir” quando eles devem ser executados e com a lógica em si. Essa classe em conjunto com o uso do padrão *Builder* torna a criação da cadeia mais simples e a adição ou remoção de elos fica facilitada, inclusive em tempo de execução quando precisamos “ligar" ou 'desligar” algum elo com base em *feature flags* (ou *feature toggles*).

## Builder

- Esse padrão é muito útil quando precisamos criar objetos complexos e que dependem de várias informações externas, que não poderiam ser feitas apenas passando dados via construtor;
- É importante que a classe a ser construída através do *Builder* tenha seu construtor marcado como privado. Isso evita que a classe seja construída sem o uso do seu *Builder*, ajudando a eliminar possíveis inconsistências de dados ou comportamentos inadequados ao criar a instância da classe de forma errada/não recomendada.
- No exemplo apresentado aqui, eu defini a classe *Builder* como [`object`](https://kotlinlang.org/docs/object-declarations.html#object-declarations-overview), uma forma de criar *Singleton* em Kotlin. Como esse exemplo é simples, não precisamos de nenhuma classe auxiliar para que a instância da classe `SellValueCalculationChain` seja criada, então o *Builder* ser um *Singleton* faz total sentido. Se alguma classe fosse necessária, poderíamos definir o *Builder* como uma classe comum e passar a classe auxiliar como parâmetro de construtor do *Builder*.
    - Acredito que seja importante que as funções públicas do *Builder* sejam apenas voltadas para a criação da instância da classe da qual aquele *Builder* pertence. Se eventualmente o *Builder* precisar de alguma classe extra para que ele possa criar a instância, essa classe deve ser passada no construtor do *Builder,* nunca via função. Assim, evitamos confusões quanto ao uso do *Builder*. Em resumo, usamos o construtor do *Builder* para fornecer dados necessários para o *Builder* fazer o seu trabalho e as funções públicas são para fornecer dados ao *Builder* que serão adicionadas à instância da classe a ser criada.

# Projeto completo

Para ver esse projeto no [Github](https://github.com/hallisonoliveira/ChainOfResponsibilityAndBuilderWithKotlin) ou no [Kotlin Playground](https://pl.kotl.in/-Tp4Dfgp4)

# Referências

- [Refactoring Guru - Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)
- [Refactoring Guru - Builder](https://refactoring.guru/design-patterns/builder)