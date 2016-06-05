---
layout: post
title:  "Leia as mensagens de erros"
date:   2016-06-04
author: Ulisses Almeida
categories: aprendendo-a-programar
lang: pt-BR
excerpt: "Se existe uma dica que eu daria para alguém que está iniciando no maravilhoso mundo da programação seria: aprenda a ler as mensagens de erros."
image: /assets/leiaasmensagensderro.png

---

![errorsarecoming](/assets/leiaasmensagensderro.png)

Se existe uma dica que eu daria para alguém que está iniciando no maravilhoso mundo da programação seria: aprenda a ler as mensagens de erros.

No post [“Prepare-se, os erros estão vindo”][errorsarecoming]{:target="blank"}, eu escrevi sobre os tipos de erros que podem acontecer quando você for escrever um programa. Se você talvez não sabe do que estou falando, clique no link dê uma lida e então volte aqui para ler o resto desse post. :)

Me lembro quando eu era um jovem aprendendo a programar, querendo fazer coisas legais e deixar meus colegas de trabalho orgulhosos. Porém eu tinha uma dificuldade que me travava e fazia perder horas: erros. As mensagens que os erros apresentavam me frustavam e me deixavam muitas vezes confuso sem saber qual próximos passo tomar. Se essas mensagens te deixam assim, espero que esse texto consiga te ajudar a melhorar a relação com esses seres tão presentes no nosso cotidiano da programação.

As mensagens de erro poderiam ser perfeitas se elas conseguissem dizer o que você precisa fazer para dar certo, principalmente para iniciantes. Mas acredito que isso não acontece, porque é muito difícil programar isso. Tratar todos casos que podem dar errado e exibir uma mensagem que ajude a pessoa resolver é uma tarefa árdua. Se você programa um pouco, sabe do que eu estou falando. Outra dificuldade é que as pessoas que escrevem mensagens erros elas muitas vezes não conhecem todo público que vai tentar programar. Para agilizar o processo, elas escrevem para pessoas programadoras experientes que nem elas.

Para seguir a vida programando não tem jeito, você precisa aprender lidar com elas para ser produtivo. A seguir algumas dicas que seriam muito valiosas se alguém me dissesse quando eu era um jovem gafanhoto no mundo da programação, então espero que te ajude:

* **Identifique o tipo de erro**, você conhece os tipos de erros e então quando ele acontecer identifique-o. Quando for erro de sintaxe por exemplo, você verá algum texto semelhante a “syntax error”. Para erros de semântica provavelmente vai ter algo com “type error" ou “no method error”. Para mensagens que contém “file not found”, “access not allowed”, “network”, “could not reach DNS”, enfim se tem algo assim possivelmente está faltando alguma coisa faltando no seu ambiente para ser possível fazer a operação.

* **Busque mais dicas na mensagem de erro**, agora que você identificou tipo do erro, possivelmente na própria mensagem de erro possuem dicas para a solução. Respire, normalmente os erros lhe informam a linha e o arquivo em que o erro aconteceu.

* **Olhe o stack trace**, stéquitreice? O stacktrace é a lista de arquivos, funções, objetos, e métodos que foram executadas até chegar ao erro. Use-a para identificar que parte do seu código deu erro. As vezes o erro não foi exatamente no seu código, pode ter sido em alguma biblioteca que você esteja usando.

![evilerror1](/assets/evilerror1.png)

![evilerror2](/assets/evilerror2.png)

Ler as mensagens de erros é o passo fundamental para a solução. O segundo passo é analisar mensagem. Após analisar você vai conseguir entender. Entendendo o erro, provavelmente você já vai conseguir pensar em várias soluções. Pratique isso, que você vai perceber que sua capacidade de resolver problemas sozinho vai aumentar bastante.

[errorsarecoming]: http://ulissesalmeida.github.io/aprendendo-a-programar/2016/05/08/prepare-se-os-erros-estao-vindo.html
[lightbot]: https://lightbot.com
