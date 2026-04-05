---
layout: post
title: Independência da Nuvem 
published: false
date: 2026-04-05T13:19:42.0000000-03:00
tags: [Synology, NAS, Tailscale, Plex, Sonarr, Radarr, Audiobookshelf, PI-Hole, Immich, Unifi, Enpass]
---

## TLDR
Este é o primeiro post de uma série onde vou entrar em detalhes sobre como estou montando uma infra de equipamentos e serviços para me tornar um pouco mais independente de serviços pagos na núvem e melhorar o controle e acesso aos meus dados e conteúdos pessoais, sem abrir mão de privacidade nem comodidade.

Neste primeiro post vou falar um pouco sobre as motivações e conceitos básicos. Cada serviço específico que vou usar terá seu próprio post com mais detalhes técnicos.

<!--more-->

Por anos eu mantive meus dados nos serviços de núvem que todos nós ficamos habituados, e fui criando meu padrões de uso. Mas com o tempo, mudanças nas prioridades e, principalmente, na confiabilidade das empresas responsáveis por esses serviços, comecei a ver que estamos todos ficando extremamente dependentes de serviços específicos, e as empresas por trás desses serviços têm tirado vantagem disso e vem aumentando os custos e/ou reduzindo as funcionalidades desses serviços para que façamos assinaturas cada vez mais custosas para manter o mesmo padrão de comodidade.



30 anos atrás, quando comecei a trabalhar e ter meu próprio dinheiro, lembro de ir com bastante frequencia lojas de discos e, posteriormente de CDs, para comprar os álbuns das bandas que eu gostava. Chegava em casa com o disco debaixo do braço, colocava no toca discos da sala e podia ouvir o quanto eu quisesse. Era dono do conteúdo que comprei. Adovava colecionar box de DVDs dos filmes e séries que eu gostava, tanto para poder rever a qualquer momento (vale lembrar que serviços de streaming nem existiam ainda), como para poder passar horas a mais vendo os extras, comentários, erros de gravação e curiosidades. Tinha tantos boxes de DVD, que nem cabiam mais no hack da minha TV. Também tinha dezenas de discos dos jogos de Xbox que eu jogava, comprados muitas vezes em pré-venda nos sites de ecommerce. E podia emprestar os jogos, que foi como conheci alguns que não tinha jogado ainda, antes de decidir comprar a minha cópia (Assassin`s Creed, por exemplo). Tinhamos controle e autonomia.

Hoje, está tudo online. Tudo instantâneo, Tudo "ao vivo". Parece muito mais prático e cômodo, e por muito tempo foi assim que me senti. Não precisava mais ficar encontrando espaço físico em casa para guardar meus discos, CDs e DVDs. Computadores ficaram mais poderosos e armazenamento foi aos poucos ficando mais barato. Internet ficou mais rápida e confiável. Passamos a usar streaming para música ao invés de ouvir nossos CDs ou até mesmo os arquivos mp3 extraídos desses CDs. Ficamos cansados da TV por assinatura e passamos a contratar planos de streaming pois tínhamos acesso ao um acervo enorme de filmes e séries a qualquer momento, sem comerciais, sem espera, sem o risco de chegar na locadora e já estar alugado para outra pessoa. E por muito tempo isso pareceu ser bom. Tudo rápido, sem espera, sem atrasos, sem precisar sair de casa.

Paramos de usar pen-drives e HDs externos para transportar nossos arquivos e começamos a deixar cópias de tudo na núvem. Tínhamos Dropbox, aí apareceram Google Drive e Skydrive (que precisou mudar de nome para OneDrive), e depois iCloud. Passamos a guardar mais e mais nossos documentos e fotos nesses serviços, pois assim conseguíamos transitar entre computadores (e eventualmente smartphones) sem precisar fazer backups intermináveis que só serviam para bagunçar cada vez mais nossos diretórios de documentos. Eu particularmente acabei entrando de cabeça no OneDrive pois participei de tantos eventos da Microsoft que consegui resgatar alguns códigos que davam espaço extra na minha conta. E esse espaço extra não era uma assinatura ou um especial de alguns meses, era espaço extra vitalício!

Mas as empresas começaram a aumentar os valores, justificando que era por conta dos novos recursos (que na maioria das vezes, não pedimos). Começaram a forçar o uso dessas plataformas e tornar cada vez mais difícil não usar elas. Você instala o SO no computador e já tem que fazer login com usa conta online e deixar seus arquivos na núvem. É como se o computador não fosse mais útil em modo offline.

Recentemente percebi que cansei disso. Sinto falta dos extras nos DVDs. Sinto falta de ter controle sobre o conteúdo que me interessa. Mas principalmente, sinto falta de ter controle sobre meus dados e minha privacidade. E para piorar, os serviços todos de streaming que começamos a assinar para fugir da TV por assinatura e seus comerciais, agora também têm comerciais, a não ser que você pague ainda mais caro por eles. Cansei de dar play na minha playlist de música e ouvir propagandas que não me interessam. Passei anos organizando minha playlist com as músicas que gosto e não quero ser interrompido por um comercial, muitas vezes tocando uma música que não tem absolutamente nada a ver com meu gosto musical.

Cerca de 8 anos atrás, comprei um servidor NAS da Synology (DS718+) durante uma viagem a NY. Não soube como usar bem ele por muito tempo e ele ficou praticamente como repositório de dados e servidor de Plex para meus backups de filmes de DVDs por pouco mais de 7 anos. Mas agora, decidi pegar todos meus dados que hoje estão online de alguma forma e trazer para dentro de casa. Decidi organizar tudo de uma forma eficiente, estruturada e que sejá prática no dia a dia. Basicamente, decidi montar minha núvem particular.