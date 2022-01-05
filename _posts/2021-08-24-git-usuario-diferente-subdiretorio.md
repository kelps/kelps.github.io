---
layout: post
title: Git com múltiplas configurações no mesmo computador, organizado por subdiretórios
published: true
date: 2021-08-24T09:00:00.0000000-03:00
tags: [Controle de Versão, Git]
---

Tendo em vista que grande parte do desenvolvimento de software atual padronizou o controle de versão no Git, um novo problema que passou a ocorrer é o fato de que muitas vezes queremos ter configurações diferentes para situações diferentes. Ex.: Queremos usar um nome e e-mail para commits nos nossos trabalhos pessoais ou repositórios Open Source e outro para nossos commits nos repositórios da empresa. Há também situações onde podemos trabalhar como freelancer para múltiplos clientes e precisamos usar as plataformas desses clientes, e pode ser necessário usar logins diferentes para cada caso ou mesmo outras configurações do git.
<!--more-->

É possível fazer isso personalizando as configurações de cada repositório, mas fazendo assim corre-se o risco de cometer erros de configuração ou mesmo de esquecer de personalizar ao baixar o repositório (ou se for necessário baixá-lo novamente).

Lendo alguns artigos falando sobre configuração dos padrões do git, descobri que é possível realizar includes normais e condicionais de arquivos nas configurações. Com esse recurso, é possível ter um conjunto de configurações globais e outros conjuntos que se aplicam apenas a repositórios criados dentro de algum subdiretório, por exemplo.

Supondo que tenho no meu computador a seguinte estrutura de pastas:

- d:\repositorio
	- cliente-1
	- cliente-2
	- github

Nesta estrutura, desejo baixar todos os repositórios de projetos de cada cliente em sua própria pasta e tudo que for github onde eu fizer contribuições pessoais, nas pasta github. Quero que todos os commits nos repositórios do github utilizem o email que uso no github e os projetos de cada cliente utilizem os e-mails que das credenciais que cada cliente me forneceu.

- 1- Para fazer isso, meu primeiro passo foi criar uma pasta onde vou gravar minhas configurações personalizadas. Escolhi criar esta pasta no meu OneDrive pessoal, para que essas configurações fiquem guardadas na nuvem e sejam replicadas nos meus outros computadores ou quando eu formatar este computador possa recuperar as configurações com facilidade. Criei esta pasta em **d:\onedrive\git**
- 2- Nesta pasta, criei um arquivo texto chamado **"gitconfig"** (assim mesmo, sem extensão).
- 3- Criei também um arquivo para cada subdiretório onde desejo/preciso ter configurações diferentes: **"gitconfig-cliente-1"**, **"gitconfig-cliente-2"**, **"gitconfig-global"**.
- 4- Editei este arquivo **"gitconfig"** criado no passo 2 para que tenha o conteúdo abaixo:

```
[include]
	path = d:/onedrive/git/gitconfig-global

[includeIf "gitdir:d:/repositorio/cliente-1/"]
	path = d:/onedrive/git/gitconfig-cliente-1

[includeIf "gitdir:d:/repositorio/cliente-2/"]
	path = d:/onedrive/git/gitconfig-cliente-2
```

- 5- Editei o arquivo "gitconfig-global" para ficar assim:

```
[user]
	name = Kelps Sousa Alux
	email = meu-email-global@meu-dominio.com.br
[fetch]
	prune = true
[init]
	defaultbranch = main
```

- 6- Editei os arquivo de cliente trocando as configurações que fossem necessárias. No caso do cliente 1, troquei o e-mail de commit pelo e-mail das credenciais fornecidas pelo cliente:

```
[user]
	email = kelps.alux@cliente-1.com.br
```

- 7- No caso do cliente 2, troquei o e-mail de commit pelo e-mail das credenciais fornecidas pelo cliente e a definição de branch padrão também, pois este cliente tem um padrão próprio:

```
[user]
	name = Kelps Sousa Alux (Freelancer)
	email = kelps.alux@cliente-2.com.br
[init]
	defaultbranch = principal
```

- 8- Editei o arquivo de configurações padrão do git do meu usuário no computador para importar o **"gitconfig"** criado acima. Este arquivo fica em **"%homepath%\\.gitconfig"**. Se não existir o arquivo **".gitconfig"** na pasta **"%homepath%"** do seu computador (Windows), pode criar ele, mas isso provavelmente significa que o Git não está instalado corretamente para o usuário atual ou que não foi usado ainda. Neste arquivo eu apenas acrescentei no final as informações abaixo:

```
[include]
	path = d:/onedrive/git/gitconfig
```

Após seguidos esses passos, o arquivo de configuração padrão **"%homepath%\\.gitconfig"** estará importando as configurações do **"gitconfig"** que se encontra na pasta que criei no meu OneDrive. Este por usa vez, importa as configurações do arquivo **"gitconfig-global"** e logo na sequencia faz 2 importações condicionais das configurações específicas de cada cliente. A condição que criei para as importações condicionais foi com base no diretório onde cada repositório de cliente se encontra.

Para testar se a configuração ficou correta, basta abrir um prompt de comando no diretório de qualquer repositório e executar o comando ```git config -l```. Todas as instruções de include serão exibidas, mas os conteúdos das configurações condicionais são apareceram se executar o comando em repositórios dentro dos subdiretórios dos clientes. Se executar o comando no subdiretório do cliente 1, a lista de configurações vai mostrar meu e-mail sendo definido 2 vezes, a primeira para o da configuração global e a segunda para o que está na configuração do cliente 1.
Quaisquer configurações que possam ser realizadas nesses arquivos de configuração do git, podem ser feitas assim.

Após realizar essas configurações no meu computador, não importa mais se faço commits executando comando de terminal no repositório ou usando ferramentas como o próprio Visual Studio ou Visual Studio Code. E sempre que eu clonar novos repositório de cada cliente, basta que o faça dentro do respectivo subdiretório.
