---
layout: post
title: Independência da Nuvem - parte 1 - Rede
published: true
date: 2026-04-05T13:28:07.0000000-03:00
tags: [PI-Hole, Unifi, cloud]
---

Este é primeiro de uma série de posts que estou preparando para descrever de forma detalhada como estou fazendo para reduzir minha dependência de diversos serviços online, de forma segura, prática e agradável de usar. Nestes posts vou falar sobre os serviços que estou usando e detalhar como foram configurados e as decisões que levaram às escolhas que foram feitas.

Neste primeiro post vou falar um pouco sobre:

# Rede

<!--more-->

Aproximadamente 12 anos atrás eu comecei a ficar incomodado com as limitações de configurações de rede disponíveis quando usamos exclusivamente os roteadores disponibilizados pelos provedores de internet. Também estava relativamente frustrado por ter precisado reconfigurar minha rede 2 vezes em menos de 1 ano por conta de problemas com roteadores da NET e da Vivo, e quando foi instalado o novo roteador, eu nem pude usar a mesma senha de rede que usava antes, por conta de restrições de formato, tamanho ou quantidade de caracteres que podiam ser usados na senha (e minha senha era longa ou complexa demais 🤷).

Foi quando decidi começar a usar roteador próprio com firmware open source OpenWRT. A partir deste momento, o roteador da operadora passou a ser apenas um intermediário para o acesso à internet e a configuração da minha rede de fato passou a ficar no meu roteador, com wifi, firewalls e outras pequenas configurações de segurança sempre constantes, mesmo se trocar o provedor de internet. (importante sempre desligar o wifi do roteador do provedor quando usado assim)

Mas foi durante a pandemia que houve o grande avanço na qualidade da minha rede. Como passei a trabalhar 100% remoto, em home office, a estabilidade e robustez da minha rede passou a ser um fator muito mais importante. Decidi que havia chegado a hora de aposentar o roteador antigo e passar a usar equipamentos novos de alto desempenho. Comprei o meu atual roteador [Ubiquiti Unifi UDM](https://techspecs.ui.com/unifi/cloud-gateways/udm) e de lá pra cá, aos poucos, fui expandindo a rede conforme a necessidade, incluíndo access points para expandir a cobertura de wifi e mini switches POE para extender a rede cabeada a outros locais da casa e permitir uma eventual instalação de câmeras de segurança. 

![Unifi](/img/posts/2026/unifi.png)

## Segregração de acesso

Um dos recursos que passei a usar foi o conceito de segregação de rede. Basicamente, por esse equipamento ser mais sério e criado inicialmente para ambientes de pequenas/médias empresas, eu passei a ter acesso a recursos mais avançados, podendo configurar redes virtuais (vlans) separadas dentro na mesma infra física, o que me permite definir regras e restrições de acesso por tipo de equipamento, além de separar logicamente os tipos de tráfego na rede, o que era bem mais complexo (quando possível) no roteador anterior.

Eu decidi separar minha rede em 3 vlans:

- principal: nessa vlan ficam todos os meus equipamentos de uso pessoal, como desktop, laptops, tablets, smartphones, smartwatches. Esta vlan consegue acessar livremente todos equipamentos e serviços nas outras 2 vlans.
- IOT (internet of things): esta vlan é para outros tipos de equipamentos, como TVs, interruptores/tomadas inteligentes, câmeras de segurança. Enfim, qualquer dispositivo que você precise de acesso a internet, mas que você não quer que necessariamente tenha acesso aos outros dispositivos/serviços na rede. Qualquer tipo de eletrodoméstico "inteligente" que desejar usar, inclusive assistentes pessoais como Alexa devem ficar nesta vlan.
- visitantes/convidados: essa vlan é usada exclusivamente para dar acesso a internet para convidados/visitantes, mais uma vez, sem expor os outros serviçoes/equipamentos na sua rede.

Cada uma dessa vlans acima tem um intervalo de ips próprio na rede e tem também sua própria rede wifi. Os equipamentos Unifi são capazes de disponibilizar diversas redes wifi ao mesmo tempo, inclusive em bandas diferentes (2,4Ghz, 5Ghz, wifi6, wifi7...). A rede de convidados eu deixo normalmente desligada, e ativo apenas quando há visitas em casa. Para facilitar o acesso, eu gero um QR Code quando ativo a rede para que a pessoa acesse com facilidade. (em um dos próximos posts vou mostrar como faço isso).

## Privacidade

A segregação das redes feita acima já ajuda bastante em questões de privacidade, pois as regras de firewall criadas impedem que equipamentos nas redes IOT vasculhem a rede e descubram meus outros dispositivos e enviem informações de telemetria sobre eles para os fabricantes, mas isso por si só não é mais suficiente. Por conta disso, adotei um novo equipamento na rede: um servidor PI-Hole.

PI-Hole é um servidor DNS open source que foi criado com o objetivo de permitir a fácil criação e manutenção de listas de bloqueio de DNS, para bloquear facilmente sites de telemetria, propagandas, inseguros (conhecidos por hospedar virus/malware), pornográficos, etc...

![pi-hole](/img/posts/2026/pi-hole.png)

Ele funciona se comportando como o servidor de DNS principal da sua rede. Todas solicitações de DNS realizadas são feitas para ele. Se ela estiver em uma das listas de bloqueio que foram importadas no aplicativo, ele retorna um IP inválido para o serviço, assim ele não consegue acessar o conteúdo indesejado. Se a url solicitada não estiver na lista de bloqueio, a soliticação é repassada adiante para servidores de DNS externos oficiais que você configura (como o Google ou da Cloudflare, por exemplo).

Além disso, ele permite a criação de endereços DNS privados na sua rede, o que será muito útil mais adiante para tornar mais amigável o acesso aos serviços que serão instalados na minha rede.

Decidi usar um hardware separado para o PI-Hole pois para ter o resultado desejado, ele deve estar disponível na rede o mais rápido possível. Se utilizasse alguma máquina virtual, ele só estaria disponível depois do servidor da máquina virtual ter inicializado completamente.

### DNS Hijack

Agora, após configurar o PI-Hole como DNS primário da minha rede no UDM, e sem DNS secundário, todos equipamentos de rede deveriam passar a usar ele para resolver urls. Mas infelizmente isso não é mais verdade. Hoje temos diversos dispositivos "inteligentes" que colocamos na nossa rede, que burlam os protocolos padrões e não respeitam as definições do roteador. Para resolver isso, fiz uma última configuração no UDM, conhecida como DNS Hijack.

![DNS Hijack no Unifi](/img/posts/2026/unifi-dns-hijack.png)

Basicamente, essa configuração diz para o roteador que qualquer soliticação de comunicação nas portas de rede usadas pelo protocolo de DNS deve ser encaminhada para o ip do servidor PI-Hole na rede. Isso ajuda a fechar ainda mais o cerco. Ou seja, agora, mesmo dispositivos "mal comportados", que tentam usar servidores de DNS fixos definidos no seu código, vão acabar passando pelo servidor de DNS oficial da rede, sem nem saber.

Depois de todas essas configurações e com 3 listas de bloqueio ativas no PI-Hole (contendo mais de 740mil endereços bloqueados neste momento), Eu descobri que mais de 40% das soliticações de DNS feitas pelos equipamentos na minha rede estão sendo bloqueados. Isso é assustador e mostra o quanto estamos vulneráveis e o quanto não sabemos o que de fato nossos dispositivos podem estar fazendo. Absolutamente TODOS dispositivos da minha rede têm solicitações bloqueadas diariamente. 

![Lista de bloquei que estou usando](/img/posts/2026/dns-block-lists.png)

## Viagem

E para fechar o pacote, temos situação de viagens. Este tópico eu vou apenas iniciar aqui, pois para ficar 100% completo ainda depende de algo que vou detalhar em um dos próximos posts, mas já podemos começar.

A idéia é a seguinte: Quando viajamos, precisamos entrar no wifi do hotel no nosso destino, e muitas vezes precisamos fazer alguns malabarismos a cada nova conexão (entrar em página de login, digitar usuário e senha fornecido pela recepção, ....). E temos que fazer isso em todos dispositivos (laptop, smartphone, ...)

E muitas vezes, queremos acessar conteúdo na TV do hotel usando nossos acessos, mas não é seguro fazer login nas plataformas em um dispositivo que não nos pertence, então a solução que uso hoje é um mini roteador de viagem. 

Para isso, eu comprei um [GL-iNet Beryl AX](https://www.gl-inet.com/products/gl-mt3000/). Ele é um roteador bem pequeno (do tamanho de uma carteira), mas extremamente funcional e atende perfeitamente as necessidades de viagem. Com ele, eu chego no hotel, mas ao invés de conectar cada um dos meus dispositivos no hotel, eu ligo o roteador, conecto nele e depois configura ele para servir como repetidor da rede do hotel. 

Se o hotel só tiver wifi, é possível configurar ele como repetidor de wifi. Se precisar entrar em alguma página no wifi do hotel para fazer login, basta fazer isso no primeiro acesso. Se o hotel tiver rede cabeada, melhor ainda. E em último caso, é possível usar ele também com internet via USB do telefone.

Agora meus dispositivos não conectam mais diretamente na rede do hotel, e consequentemente não ficarão expostos na rede, pois estão em uma rede isolada apenas minha. E para finalizar meu conforto/comodidade em viagens, posso usar um Amazon Fire Stick ou Google Chromecast ou algum aparelho semelhante, conectar ele na TV do hotel e acessar meus conteúdos, meus canais no Youtube, minhas playlists de música, meus serviços de streaming, sem se preocupar em vazar os meus dados de login em um dispositivo que não me pertence e sem expor os meus dispositivos diretamente na rede do hotel.

## VPN

O último passo é ter uma VPN. Os roteadores Unifi permitem criar vários tipos de VPN, usando OpenVPN, WireGuard, ou o padrão exclusivo deles chamado Teleport (que é uma implementação própria usando o protocolo do WireGuard). As desvantagem de usar OpenVPN ou WireGuard é que é necessário um IP fixo ou de alguma forma de saber o IP atual de casa de forma dinâmica, sem contar que também precisa configurar desvios de portas (port forward) no roteador da operadora, o que nem sempre é trivial ou as vezes pode nem ser possível. 

Já o padrão Teleport não precisa de nada disso. Ele requer que você tenha uma conta na ui.com (site da Ubiquiti/Unifi), mas depois disso, basta instalar o aplicativo Wifiman da Ubiquiti no seu dispositivo (smartphone android/ios ou computador windows/mac) e conectar na mesma conta, que você será capaz de conectar na sua VPN de qualquer lugar, desde que a porta de VPN usada pelo Teleport esteja aberta na rede onde você estiver. 

E com isso, temos uma infra bem robusta, de manutenção extremamente baixa e altamente confiável

Em um próximo post vou falar sobre a evoluir ainda mais esta rede, mas isso depende de outros serviços que não foram abordados ainda.

