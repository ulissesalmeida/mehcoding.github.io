---
layout: post
title:  "Prepare-se, os erros estão vindo"
date:   2016-05-08
author: Ulisses Almeida
categories: aprendendo-a-programar
lang: pt-BR
---

Quando somos iniciantes na programação é difícil lidar com os erros que brotam em nossa tela. Eles chegam quando não estamos esperando, não são gentis, na maioria das vezes são em inglês, tudo para de funcionar e ainda lhe acusam de maneira que parece que você fez tudo errado.

Os erros são tão cruéis que qualquer um da sala consegue ver que você está com problemas. Pois muitas vezes quando o erro acontece, todo resultado da sua tela é trocado por um monte de texto escrito em letras grandes e vermelhas. Só restava aplicar a fonte "Gothic" e elas se tornariam a obra prima da maldade.

“A pessoa que inventou as essas mensagens de erros tem um coração peludo”. Eu acredito você deve testar pensando isso agora, mas acredito não é por pura maldade. Duvido que tenha um ser humano escondido e rindo diabolicamente toda vez que alguém cometeu um erro ao escrever um programa.

Do jeito que eu escrevi parece o fim do mundo ou algo muito difícil, mas não é. Tivemos uma grande evolução na barreira de entrada para programar. Hoje está muito melhor do que há 10 anos atrás e será muito melhor nos próximos 10 anos.

No meu pouco tempo de vida pude observar que para fazer um jogo 3D há alguns anos atrás parecia ser algo para poucos e grandes equipes que estudaram a vida inteira para isso. Porém hoje, você consegue baixar ferramenta [Unity3D][unity3D]{:target="blank"} e fazer algo funcionar em pouco tempo.

Hoje existem programas que ajudam iniciantes aprenderem a programar e possuem uma interface bem amigável. Ao invés de escrever instruções com palavras por exemplo, você arrasta bloquinhos de instruções para fazer alguma coisa acontecer. Entre esses programas também existem [alguns jogos][lightbot]{:target="blank"} que ajudam ensinar lógica de programação.

Nesses jogos o erro humano de digitar uma letra errada é totalmente reduzido. Se você fizer um conjunto de instruções errados, no máximo que vai acontecer é sua personagem explodir e ser substituída rapidamente por uma nova em folha.

Porém o trabalho profissional de hoje não chegou a esse nível. Se você quiser programar e produzir trabalhos mais significativos, você terá que enfrentar linhas e mais linhas de texto. Terá que enfrentar mensagens de erros que não parecem fazer o menor sentido se fossem lidas para pessoas que nasceram 18XX. Acho que não fazem sentido nem para pessoas nascidas hoje.

Então vou tentar classificar aqui os erros que podem acontecer com você na jornada de desenvolvimento de software:

* **Erros de sintaxe**, você escreveu algo que a linguagem que você está usando não permite, você pode ter esquecido algum ponto e vírgula, fechar algum parêntese, digitar um espaço.

* **Erros de semântica**, você escreveu corretamente usando de maneira imprópria seu programa. Por exemplo, você escreveu corretamente associação de um valor numérico para um lugar que esperava um texto ou tentou chamar uma instrução que não existe.

* **Erros de execução**, você escreveu algo que vai ser nem sempre possível de ser executado. Por exemplo, um comando que tentou ler uma arquivo que não existe no seu HD, ler os tweets quando não tem acesso a internet ou precisou de memória quando a máquina não tinha mais.

* **Erros de lógica**, programa até parece funcionar até você ver que o resultado não foi o esperado. Todas instruções são executadas pela máquina corretamente. O erro foi que você mandou as instruções erradas. Imagina que você queria escrever um programa que somasse 2 mais 2, porém você escreveu 2 menos 2, você esperava 4 mas resultou em 0.

* **Erros de definição**, o programa parece funcionar, você julga que ele está correto. Porém o seu usuário diz que não era isso que ele esperava. Por exemplo, o usuário diz que quer X, quem analisou o requisito entendeu Y, explicaram para o programador Z. Programador fez Z, mas o usuário do programa queria X.

**Observação**: *Na teoria da computação que você vai encontrar em livros ou na falcudade, os erros são classificados de maneira diferente. Muitas vezes estão associados as fases de execução de um programa. Por exemplo, a definição mais comum são esses tipos: erros de compilação, erros de execução e erros de lógica. Eu acho isso muito confuso pois nem todas linguagens de programação possuem as mesmas fases. Os erros que acontecem na fase de uma linguagem nem sempre acontecem nas mesma fase da outra. Por exemplo, uma linguagem de tipagem dinâmica tem erros de tipagem durante a execução, enquanto uma linguagem de tipagem estática possui erro de tipagem na fase de compilação. Portanto prefiro minha classificação, pois eu classifico os erros não importando a fase que eles acontecem.*

Da próxima vez que encontrar um erro ao escrever um programa, tente identificar o tipo dele. Com isso você será uma pessoa mais assertiva em comunicar e resolver o problema.

[unity3D]: https://unity3d.com
[lightbot]: https://lightbot.com
