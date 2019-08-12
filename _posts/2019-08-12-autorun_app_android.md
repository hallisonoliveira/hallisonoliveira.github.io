---
layout: post
title: Executar um aplicativo automaticamente ao ligar o dispositivo
img: autorun_app_android.png
category: android
tags: [
    android,
    kotlin
]
comments: true
---

Apesar não parecer uma boa prática, as vezes surgem projetos onde precisamos que um aplicativo seja executado automaticamente logo após a inicialização do dispositivo.

Para fazer isso, basta alterar o **AndroidManifest.xml**, adicionando os seguintes trechos:
<script src="https://gist.github.com/hallisonoliveira/bfa675e64c028b72b885c8c66ad91321.js"></script>

**Atenção para os locais onde as linhas acima devem ser inseridas.**

A tag *\<uses-permission>* refere-se à permissão que o aplicativo precisa para receber o evento emitido pelo Android quando o processo de boot for concluído.

A tag *\<receiver>* declara um <a href="https://developer.android.com/reference/android/content/BroadcastReceiver.html">*BroadcastReceiver*</a> responsável por habilitar a aplicação para receber o evento de boot completo lançado pelo sistema.

Além do *AndroidManifest*, é necessário criar uma classe de serviço que será chamada quando o *BroadcastReceiver* registrado no *AndroidManifest* receber o evento de *boot* completo do sistema. É ela que se encarregará de chamar a *activity* inicial do aplicativo, iniciando-o.

<script src="https://gist.github.com/hallisonoliveira/824bfb87bedb23dfc51620a19c8b2b10.js"></script>

Essa classe deve extender de *BroadcastReceiver* e implementar o método *onReceive*. Nesse método, é verificado se a ação da *intent* recebida é do tipo *ACTION_BOOT_COMPLETED*. Se for, iniciamos a *activity* desejada para iniciar o aplicativo.

A forma de chamar a *activity* é comum, bastando declarar uma *intent* passando o contexto e a classe a ser chamada como parâmetro.

O único detalhe diferente nesse caso é a adição da *flag* <a href="https://developer.android.com/reference/android/content/Intent.html#FLAG_ACTIVITY_NEW_TASK">**Intent.FLAG_ACTIVITY_NEW_TASK**</a>, que indicará que essa *activity* se tornará o inicio de uma nova tarefa. Além disso, para o Android versão 9+ essa *flag* sempre é necessária quando a *activity* em questão não é chamada por outra *activity*.

Implementações realizadas, basta compilar e executar o aplicativo. Talvez seja necessário desinstala-lo para que alteração funcione.

*Imagem do post: <a href="https://pixabay.com/pt/users/raphaelsilva-4702998/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3383929">raphaelsilva</a> por <a href="https://pixabay.com/pt/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3383929">Pixabay</a>*

Obrigado e até +.
