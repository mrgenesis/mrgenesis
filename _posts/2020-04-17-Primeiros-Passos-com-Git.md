---
layout: post
title:  Primeiros Passos com Git
categories: [Git]
excerpt: Entenda conceitos básicos sobre Git e para que ele serve.
---

Git é um **controlador de versões** bastante conhecido no mundo da tecnologia. Neste artigo veremos mais detalhes a seu respeito e como iniciar com Git.

  

## O que é GIT?

  

Antes de responder esta pergunta, é preciso entender **o que é um VCS**. Em resumo significa Version Control System. É o nome dado aos Sistemas de Controle de Versão de arquivos.

  

Basicamente, o VCS (Version Control System) serve para realizar o controle das mudanças que ocorrem no processo de desenvolvimento de um projeto – o próprio nome sugere isso. Embora seja muito comum usá-lo em desenvolvimento de softwares, o seu uso pode ir muito além disso; pode ser usado em qualquer tipo de projeto. *Note que o objetivo do VCS é controlar as mudanças do projeto*. Com base nisso é que poderíamos usá-lo em projetos de qualquer natureza. Neste contexto, nós poderíamos realizar as seguintes coisas:

  

* Registrar as etapas das mudanças no projeto

* Criar pontos específicos de restauração

* Desfazer alterações

  
  

Reflita um pouco nas funcionalidades a cima e, pergunte-se: Será que um arquiteto poderia usar um VCS para gerenciar seus projetos de arquitetura? Se você entendeu o objetivo de um VCS, com certeza sua resposta será um "sim".

  

Bem, eu nunca vi, nem ouvi falar de alguém que use um VCS num projeto de arquitetura. Isso é uma coisa que me veio à mente quando eu estava pensando no que escreveria para explicar para você o que esta ferramenta faz. Mas, talvez, o que impeça que um arquiteto tenha usado esta ferramenta fantástica é que, como ela nasceu por meio dos [Homens do terminal](https://en.wikipedia.org/wiki/Revision_Control_System), e a maneira de usá-la seja *pouco convencional* do ponto de vista do usuário comum, isso tenha causado pouco ou nenhum interesse por parte de outros públicos, que não seja da área de tecnologia. Falo isso porque ainda existem desenvolvedores que sabem nada ou bem pouco sobre sistemas de controle de versão. Imagina um arquiteto!

  

Então, sobre sua utilidade, é correto afirmar que é bem amplo.

*Solte sua imaginação!*

  

## GIT

  

Falando agora, mais especificamente, sobre o mais conhecido VCS, vamos explicar de forma bem objetiva o que é git. Inicialmente criado para desenvolver o Kernel do Linux, hoje é o sistema mais usado entre os desenvolvedores. GIT é multiplataforma, disponível para Windows, Linux e MAC OS, e já conta com interfaces gráficas (nunca usei), que com certeza vai possibilitar maior adesão de outros públicos. Vou fazer uma série de artigo com objetivo de explicar como funciona esta ferramenta, e, quem sabe, como funciona a interface gráfica do GIT.

  

## Como começar a usar o GIT

  

Se você estiver usando Linux é bem provável que você já o tenho instalado, visto que vem como padrão em muita das distribuições. Mas se não tiver instalado um simples comando no terminal resolve isso fácil, fácil. Por exemplo: `sudo yum install git`.


> Claro que cada distribuição tem seu gerenciador de pacotes, mas você
> pode adequar o comando de acordo com a sua distribuição.

  

Para Windows ou MAC também é fácil: acesse o site oficial que você terá o [GIT em poucos passos](https://git-scm.com/downloads). Basta baixar e executar com Next, Next... Finish. Não precisa mexer em nada, okay? Instale do jeito padrão que vocês serão felizes para sempre!

  

  

## Primeiros comandos no GIT

  

Agora que você tem o GIT instalado em seu computador que tal dar os primeiros comandos e ver como isso funciona na prática?

  

A primeira coisa a fazer é configurar seus dados de identificação: Nome e Email. Desta forma será possível gravar suas informações no histórico de versão do projeto. Então localize o Git Bash e execute-o. Será aberto uma janela preta muito semelhante ao CMD do Windows. Digite o seguinte comando nele:

  

``` bash

~/aprendendo-git$ git config –global user.name "Seu nome completo"

~/aprendendo-git$ git config –global user.email seu@email.com

```

  

Pronto! Você acaba de fazer a configuração.

*Note: não haverá nenhuma mensagem*

  

Como nós falamos sobre a possibilidade de um arquiteto usar o GiT em seus projetos, então nada de `mkdir`, `touch`, `vi`. Esquece tudo isso! Este post é para usuários comuns também. Para manter a coerência vou explicar de uma forma simples para todos os níveis. Vou falar do ponto de vista do Windows visto que é mais abrangente. Se você está usando Linux poderá fazer do jeito mais apropriado.

  

Primeiramente crie uma pasta em um lugar de sua escolha e dê o nome de "aprendendo-git"

  

Agora abra a pasta, e crie um arquivo dentro com nome `test.txt`.

  

  

  

Note que agora você tem um projeto – Uau!!! É um projeto bem inútil, mas... você tem um projeto. Como fazer para controlá-lo com GIT? Abra o terminal clicando com direito dentro da pasta e selecione a opção *Git Bash Here* no menu de contexto.

  

  

Você vai perceber que abrirá uma tela preta, a mesma usada para configurar os dados de usuário. Mas agora estamos dentro da pasta do projeto. Agora escreva `git init` e dê enter.

  

Vai ver que será iniciado um repositório vazio, e uma pasta com nome .git será criada. Isto significa que o GIT já está monitorando seu projeto, mas para que isso ocorra de forma efetiva, escreva no terminal o seguinte: `git add .` (**tem um ponto no final**). Depois escreva: `git commit -m "primeiro commit."`

{% blockquote %}

Pasta e arquivos que começam com "." (ponto) são ocultos. É preciso habilitar a visualização em opções.

{% endblockquote %}

  

Se você percebeu que cada `commit` trata-se de comentário que especifica as mudanças no projeto, você realmente está certo. Veremos este conceito de forma detalhada no futuro, mas a ideia é controlar de forma organizada a ponto de você saber o que foi feito em cada mudança. Se você escrever `git status`, vai ver que não há nada para **commitar**. Mas, e se adicionarmos um novo arquivo ao projeto, e fizermos alguma mudança no arquivo `test.txt`? Vamos ver?

``` bash

~/aprendendo-git$ git status

On branch master

nothing to commit, working tree clean

```

``` bash

~/aprendendo-git$ git status

On branch master

Changes not staged for commit:

(use "git add <file>..." to update what will be committed)

(use "git checkout -- <file>..." to discard changes in working directory)

  

modified: test.txt

  

Untracked files:

(use "git add <file>..." to include in what will be committed)

  

TEST__2__.txt

  

no changes added to commit (use "git add" and/or "git commit -a")

```

  

Observe que o GIT captou todas as mudanças ocorridas no projeto. Note que o que foi modificado, foi apontado como **modified** (modificado) e o arquivo novo `TEST__2__.txt` foi apontado como **Untracked** (não rastreado). Notou também que isso virou um ciclo? Você sempre vai adicionar um novo arquivo e **commitar** ele. Também, sempre vai adicionar mudanças e **commitar** elas.

  
  

Estes são casos perfeitos, mas também existem os recursos de, caso haja algo errado com a modificação, você poderá voltar ao estado original. Neste exemplo basta remover o arquivo `TEST__2__.txt` e limpar o conteúdo do arquivo `test.txt`. Mas, e num projeto de dezenas arquivos? Como você vai saber exatamente o que foi alterado? Ou, mesmo num projeto de arquitetura, com AutoCAD, será que dá para saber exatamente o que foi modificado? É claro que não! É para isso que existe o GIT: Se o resultado da modificação foi boa, escreva: `git add .`, depois: `git commit -m 'sua mensagem da modificação'`, e seja feliz. Mas............. e se tudo que você mudou no projeto te levou para perdição e agora você vai ter que recomeçar seu projeto porque fez uma coisa terrível que causou a perda de uma semana de trabalho? Isto seria o seguinte: ***Oh! E agora quem poderá me defender? _ Eu! O Chapolin colorado!***

Então você poderá fazer o seguinte: `git checkout <nome do arquivo com extensão>`, para enviar as mudanças para o lixo. Com o GIT você será sempre feliz!

  

``` bash

~/aprendendo-git$ git checkout test.txt

~/aprendendo-git$ git status

On branch master

Untracked files:

(use "git add <file>..." to include in what will be committed)

  

TEST__2__.txt

  

nothing added to commit but untracked files present (use "git add" to track)

```

O que achou do post? Comenta aí sua dúvida ou sua crítica.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI0NDE0NzMwMCwtNjM0MDA4MDAzXX0=
-->