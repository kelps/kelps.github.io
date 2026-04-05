---
layout: post
title: Independência da Nuvem - parte 1: Rede
published: false
date: 2026-04-05T13:19:42.0000000-03:00
tags: [Synology, NAS, Tailscale, Plex, Sonarr, Radarr, Audiobookshelf, PI-Hole, Immich, Unifi, Enpass, cloud]
---

Este é primeiro de uma série de posts que estou preparando para descrever de forma detalhada como estou fazendo para reduzir minha dependência de diversos serviços online, de forma segura, prática e agradável de usar. Nestes posts vou falar sobre os serviços que estou usando e detalhar como foram configurados e as decisões que levaram às escolhas que foram feitas.

Neste primeiro post vou falar um pouco sobre:

# Rede

<!--more-->

Aproximadamente 12 anos atrás eu comecei a ficar incomodado com as limitações de configurações de rede disponíveis quando usamos exclusivamente os roteadores disponibilizados pelos provedores de acesso. Também estava relativamente frustrado por ter precisado reconfigurar minha rede 2 vezes em menos de 1 ano por conta de problemas com o roteador da Vivo, e quando foi instalado o novo roteador, eu nem pude usar a mesma senha de rede que usava antes, pois ele tinha restrições de formato, tamanho e quantidade de caracteres que podiam ser usados na senha.

Foi quando decidi começar a usar roteador próprio com firmware open source DD-WRT. A partir deste momento, o roteador da operadora passou a ser apenas uma ponte para o sinal de internet e a configuração de rede ficava no meu roteador interno, com wifi, firewalls e outras pequenas configurações de segurança sempre constantes, mesmo se eu trocar provedor de internet.

Mas foi durante a pandemia que houve o grande avanço na qualidade da minha rede. Como passei a trabalhar 100% remoto, em home office, a estabilidade e robustez da minha rede passou a ser um fator muito mais importante. Decidi que havia chegado a hora de aposentar o roteador antigo e passar a usar equipamentos novos de alto desempenho. Comprei o meu roteador Ubiquiti Unifi UDM e de lá pra cá, aos poucos fui expandindo a rede conforme a necessidade, incluíndo access points para expandir a cobertura de wifi e mini switches POE para extender a rede cabeada a outros locais da casa e permitir a fácil instalação de câmeras de segurança. 

## Segregração de acesso

Um dos recursos que eu agora poderia usar para configurar minha rede era o conceito de segregação de rede. Basicamente, por esse equipamento ser mais sério e criado inicialmente para ambientes corporativos, eu tenho acesso a recursos mais avançados e posso configurar redes virtuais (vlans) separadas dentro na mesma infra física, o que me permite definir regras e restrições de acesso por tipo de equipamento, além de separar lógicamente os tipos de tráfego na rede, o que era bem mais complexo (quando possível) no roteador anterior.

Eu decidi separar minha rede em 3 vlans:

- principal: nessa vlan ficam todos equipamentos de uso pessoal, como desktops, laptops, tablets, smartphones, smartwatches. Esta vlan consegue acessar livremente todos equipamentos e serviços nas outras 2 vlans.
- IOT (internet of things): esta vlan é para outros tipos de equipamentos, como TVs, interruptores/tomadas inteligentes, câmeras de segurança. Enfim, qualquer dispositivo que você precise de acesso a internet, mas que você não quer que necessariamente tenha acesso aos outros dispositivos/serviços na rede. Qualquer tipo de eletrodoméstico "inteligente" que você desejar usar, inclusive assistentes pessoais como Alexa devem ficar nesta vlan.
- visitantes/convidados: essa vlan é usada exclusivamente para dar acesso a internet para convidados/visitantes, mais uma vez, sem expor os outros serviçoes/equipamentos na sua rede.

Cada uma dessa vlans acima tem um intervalo de ips próprio na rede e tem também sua própria rede wifi. Os equipamentos Unifi são capazes de disponibilizar diversas redes wifi ao mesmo tempo, inclusive em bandas diferentes (2,4Ghz, 5Ghz, wifi6, wifi7...). A rede de convidados eu deixo normalmente desligada, e ativo apenas quando há visitas em casa. Para facilitar o acesso, eu gero um QR Code quando ativo a rede para que a pessoa acesse com facilidade. (em um dos próximos posts vou mostrar como faço isso).

## Privacidade

A segregação das redes feita acima já ajuda bastante em questões de privacidade, pois as regras de firewall criadas impedem que equipamentos nas redes IOT vasculhem a rede e descubram meus outros dispositivos e enviem informações de telemetria sobre eles para os fabricantes, mas isso por si só não é mais suficiente. Por conta disso, adotei um novo equipamento na rede: um servidor PI-Hole.

PI-Hole é um servidor DNS open source que foi criado com o objetivo de permitir a fácil criação e manutenção de listas de bloqueio de DNS, para bloquear facilmente sites de telemetria, propagandas, inseguros (conhecidos por hospedar virus/malware), pornográficos... 

Ele funciona se comportando como o servidor de DNS principal da sua rede. Todas solicitações de DNS realizadas são feitas para ele. Se ela estiver em uma das listas de bloqueio que foram importadas no aplicativo, ele retorna um IP inválido para o serviço, assim ele não consegue acessar o conteúdo indesejado. Se a url solicitada não estiver na lista de bloqueio, a soliticação é repassada adiante para servidores de DNS externos oficiais que você configurar nele (como o Google ou da Cloudflare, por exemplo).

Além disso, ele permite a criação de endereços DNS privados na sua rede, o que será muito útil mais adiante para tornar mais amigável o acesso aos serviços que serão instalados.

Decidi usar um hardware separado para o PI-Hole pois para ter o resultado desejado, ele deve estar disponível na rede o mais rápido possível. Se utilizasse alguma máquina virtual, ele só estaria disponível depois do servidor da máquina virtual ter inicializado completamente.

### DNS Hijack

Agora, após configurar o PI-Hole como DNS primário da minha rede no UDM, e sem DNS secundário, todos equipamentos de rede deveriam passar a usar ele para resolver urls. Mas infelizmente isso não é mais verdade. Hoje temos diversos dispositivos "inteligentes" que colocamos na nossa rede, que burlam os protocolos padrões e não respeitam as definições do roteador. Para resolver isso, fiz uma última configuração no UDM, chamada DNS Hijack.

Basicamente, essa configuração diz para o roteador que qualquer soliticação de comunicação nas portas de rede usadas pelo protocolo de DNS deve ser encaminhada para o ip do servidor PI-Hole na rede. Isso ajuda a fechar ainda mais o cerco. Ou seja, agora, mesmo dispositivos "mal comportados" vão acabar passando pelo servidor de DNS oficial da rede.

Depois de todas essas configurações e com 4 listas de bloqueio ativas (contendo mais de 400mil endereços bloqueados), Eu descobri que 30% das soliticações de DNS feitas pelos equipamentos na minha rede são bloqueados. Isso é assustador e mostra o quanto estamos vulneráveis. Absolutamente TODOS dispositivos da minha rede já tiveram alguma solicitação bloqueada. 



