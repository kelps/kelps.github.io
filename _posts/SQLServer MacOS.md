---
layout: post
title: Instalando e usando SQLServer no MacOS 
published: false
date: 2026-01-05T13:19:42.0000000-03:00
tags: [SQL, MacOS, VS Code]
---

Recentemente surgiu a necessidade de preparar um ambiente de desenvolvimento rodando em MacOS e estou aos poucos vendo o que é necessário para tornar viável e quais passos precisam ser seguidos. Vou tentar documentar aqui esses passos para poder segui-los no futuro novamente quando for necessário.

Este ambiente de desenvolvimento tem alguns requisitos relativamente simples: 
- Desenvolvimento de aplicações em .NET (web, console e bibliotecas nuget internas)
- Acesso a repositórios git no Azure Devops e GitHub
- Acesso a banco de dados SQL Server remoto e local (para execução de alguns dos testes integrados do código)

Este post irá tratar do acesso ao SQL Server, usando:
- Visual Studio Code, com a extensão (mssql)
- Instalação de servidor SQL Server local no mac (usando Docker), para execução de testes de integração sem depender de serviços externos 
<!--more-->

