---
layout: post
title: Instalando e usando SQLServer no MacOS 
published: true
date: 2026-01-05T13:28:07.0000000-03:00
tags: [SQL, MacOS, VS Code]
---

Recentemente surgiu a necessidade de preparar um ambiente de desenvolvimento rodando em MacOS e estou aos poucos vendo o que é necessário para tornar viável e quais passos precisam ser seguidos. Vou tentar documentar aqui esses passos para poder segui-los no futuro novamente quando for necessário.

Este ambiente de desenvolvimento tem alguns requisitos relativamente simples: 
- Desenvolvimento de aplicações em .NET (web, console e bibliotecas nuget internas)
- Acesso a repositórios git no Azure Devops e GitHub
- Acesso a banco de dados SQL Server remoto e local (para execução de alguns dos testes integrados do código)

Este post irá tratar do acesso ao SQL Server, usando:
- Instalação de servidor SQL Server local no mac (usando Docker), para execução de testes de integração sem depender de serviços externos 
- Visual Studio Code, com a extensão (mssql)

<!--more-->

## Instalação


Não há suporte nativo do SQL Server para MacOS, mas já a algum tempo existe suporte para Linux, e é bem fácil de rodar usando Docker. Portanto, o primeiro passo é justamente instalar o Docker. No meu caso, instalei o Docker Desktop para MacOS (isso está sendo feito em um MacBook Pro com processador M4 e 16GB RAM, mas funciona em qualquer MacBook com Apple Silicon). 

Como já estamos em 2026 e a versão 2025 do SQL Server foi lançada em novembro de 2025, minha primeira tentativa foi com ele, mas sempre que eu tentava recebia mensagens de erro e o servidor não inicializava. Olhando os logs e pesquisando online, descobri que o SQL Server 2025 requer um processador com instruções AVX, e o recurso de emulação Rosetta 2 usado pelo Docker não emula instruções AVX.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=MinhaSenhaSuperSegura@2026' -p 1433:1433 --name sql-server-2025 --platform linux/amd64 -v /Users/kelps/sql/mssql:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2025-latest
```

Decidi então tentar usar a imagem do SQL Server 2022, pois vi relatos de que essa versão funciona. Após ajustar o comando para usar a imagem do SQL Server 2022, consegui subir o servidor com sucesso!

Com este comando, os arquivos .mdf e .ldf dos databases que serão criados neste servidor SQL ficarão gravados no diretório ```/Users/kelps/sql/mssql``` do MacOS. Assim, os dados não são perdidos quando o servidor é reiniciado.

```bash
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=MinhaSenhaSuperSegura@2026' -p 1433:1433 --name sql-server-2022 --platform linux/amd64 -v /Users/kelps/sql/mssql:/var/opt/mssql -d mcr.microsoft.com/mssql/server:2022-latest
```

## Conexão

A outra parte para tornar possível/viável trabalhar com SQL Server a partir de um computador rodando MacOS é poder conectar no banco de dados para executar queries e fazer análises, tanto no servidor "local" recém criado, como nos servidores de dados da empresa. Mas como o SQL Server Management Studio só está disponível para Windows, a solução passa a ser o uso do VS Code com a extensão SQL Server, da própria Microsoft, que o deixa muito próximo do SSMS.

Essa extensão foi criada quando o Azure Data Studio foi descontinuado pela Microsoft, justamente para que desenvolvedores que trabalham usando Linux ou MacOS e usavam o Azure Data Studio (que era multi-plataforma), pudessem continuar trabalhando com seus projetos em SQL Server.

![Extensão SQL Server no VS Code](/img/posts/2026/vs-code-sql-server-extensao.png)

O acessar a extensão pela barra lateral do VS Code, aparece uma opção para incluir uma nova conexão. Basta preencher os dados da conexão SQL, passando o usuário "sa" e a senha que foi usada para criar o container Docker e a conexão é realizada como esperado. Também é possível conectar em servidores externos da mesma forma, inclusive utilizando autenticação federada do Microsoft Entra, se for necessário.

![SQL Server no VS Code](/img/posts/2026/vs-code-sql-server.png)

Agora é torcer para sair algum update da imagem Docker do SQL Server 2025 que funcione corretamente em Apple Silicon ou que Rosetta passe a ter suporte às instruções AVX em breve, ou que essa imagem do 2022 não seja descontinuada por um bom tempo!