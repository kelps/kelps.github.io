---
layout: post
title: Trocando firmware do Sonoff Mini para Tasmota
published: false
date: 2022-01-20T13:28:07.0000000-03:00
tags: [Automação Residencial,Sonoff,DIY,Tasmota]
---
## TLDR

Passo a passo para substituir o firmware do Sonoff Mini por Tasmota


update OTA Sonoff Mini 3.5.0
- segurar botão até led ficar piscando constantemente
- conectar na rede wifi gerada com nome começado com ITEAD (12345678)
- acessar 10.10.7.1 e colocar o wifi e senha a usar
- voltar para a rede e descobrir o ip
- ver configuração: enviar um POST para http://ip-do-sonoff:8081/zeroconfig/info com o payload {"data":{}}
- habilitar OTA: enviar um POST para http://ip-do-sonoff:8081/zeroconfig/ota_config com o payload {"data":{}}
- desabilitar o firewall
- usar aplicativo sonoff para realizar o update do firmware (tasmota-lite-bin)
- habilitar o firewall

configurando tasmota
- configurar a rede wifi
- em configurações, outros, colocar o template de dispositivo
- definir um nome e friendly name no modo belkin
- no MQTT, colocar o topico como sonoff_mini_000 (trocar o número por um sequencial)



<!--more-->

## Motivação

Como a maioria dos dispositivos inteligentes da atualidade, os relês da Sonoff oferecem uma plataforma própria que permite que sejam operados remotamente via aplicativo. Mas levando em consideração os constantes problemas de falhas de segurança e privacidade que vêm ocorrendo recentemente, acaba sendo uma opção mais segura usar firmwares open source, cujo código pode ser conferido para garantir que não há brechas de segurança. Também tem a vantagem que esses firmwares normalmente não acessam a internet diretamente, então isso por si só já é uma camada extra de proteção/privacidade.

Não depender da internet para funcionar, significa também que é possível operar esses dispositivos localmente mesmo que a internet esteja indisponível no momento, o que torna sua utilidade maior e integração mais natural. Ninguém quer ficar sem poder ascender ou apagar uma luz porque a internet caiu.

Com o firmware alternativo, podemos controlar os aparelhos diretamente pela rede ou usando outros dispositivos inteligente na casa (como Alexa, Google Home, Samsung Smarthings, Home Assistante...). Eu escolhi usar os Sonoffs Mini pois é possível conectá-los a interruptores para que possam ser acionados fisicamente, além de poderem ser acionados via software. Assim posso ter interruptores na casa que não dependem de nenhuma maneira nem mesmo da rede wifi local para funcionar.

O Firmware alternativo Tasmota também provê algumas funcionalidades extras que não estão disponíveis no padrão do Sonoff, mas para o meu uso o mais importante mesmo é que posso usá-los com maior segurança tanto de forma física como via rede ou por meio de assistentes.

Um bom exemplo de uso que tenho é essa [automação do filtro da piscina que fiz algum tempo atrás](2020-12-09-automacao-residencial-comandos-eletricos).

