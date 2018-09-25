---
layout: blogger
title: AssemblyVersion command line regular csproj
published: false
tags: [MSBuild, .NET, Código, Integração Contínua, Automatização]
---
Com o lançamento do .NET Core foram introduzidas diversas novidades e melhorias na plataforma .NET. Uma dessas novidades é o novo formato de arquivos de projeto (.csproj) que melhora e simplifica substancialmente seu conteúdo, tornando-o mais fácil de entender e possível de editar manualmente sem perder todas as facilidades do arquivo <strong>project.json</strong> que estava sendo usando até então no início do projeto do ASP.NET Core.

Infelizmente oficialmente esse novo formato de arquivo de projeto (conhecido como projeto baseado em SDK) por enquanto só suporta projetos Class Library e Console, nas plataformas .NET Standard, .NET Core. Há planos de oferecer suporte oficial a outros tipos de projeto em próximas atualizações do Visual Studio (fala-se principalmente de UWP e projetos mobile usando Xamarin), mas nada concreto ainda.

Apesar disso, como esse tipo de projeto continua usando MSBuild, diversos desenvolvedores conhecidos na comunidade como Nick Carver (do StackOverflow) têm feito experimentos e até passaram a usar esses tipos de projeto para desenvolvimento não relacionado a .NET Core nem .NET Standard, para poder tirar proveito de algumas vantagens como a definição das variáveis de assembly direto no arquivo de projeto ao invés do AssemblyInfo.cs e a referência direta a pacotes Nuget usando tags PackageReference ao invés de arquivos packages.config. E é exatamente por causa disso que estou escrevendo este post.

Na empresa onde trabalho temos alguns produtos web que por conta de legado e retro-compatibilidade ainda não podemos migrar para ASP.NET Core. Acontece que depois que comecei a fazer testes com projetos .NET Standard eu vi um potencial enorme de personalização e simplificação de build nesses projetos com relação ao nosso pipeline atual.

Hoje nosso produto principal é composto por mais de 40 projetos class libraries e web agrupados em 5 solutions diferentes, cada uma em seu próprio repositório no visualstudio.com. Cada repositório tem sua definição de build que compila a solution a cada novo commit. No início desse processo de build é executado um script em powershell que eu fiz que atualiza o arquivo AssemblyInfo.cs com o número do build atual e adiciona uma informação no atributo AssemblyInformationalVersion com o id do commit para podermos rastrear melhor as versões aplicadas nos clientes. Há algumas coisas que nunca me deixaram satisfeito nesse arranjo:

- Esses valores só são alterados se rodar o script, o que hoje acontece apenas durante o processo de build
- O script obtém as informações necessárias lendo variáveis de contexto definidas na configuração do build
- O script efetivamente altera o arquivo AssemblyInfo.cs, que está no controle de versão, momentos antes de efetivamente compilar o projeto

Durante meus testes com projetos .NET Standard e Core eu imediatamente notei que não existe mais o arquivo AssemblyInfo.cs. Fiquei curioso para saber como eram definidas as informações de AssemblyVersion, AssemblyCopyright, AssemblyFileVersion, etc… nesses novos projetos e decidi olhar o fonte do arquivo .csproj (que nesse novo formato, pode ser editado direto no Visual Studio sem precisar descarregar o projeto). Para minha surpresa, um projeto novo em .NET Standard contém apenas 7 linhas de código, sendo 2 delas vazias e apenas uma está de fato definindo algo no código, que é a plataforma de compilação.

Para saber onde as informações de versão são armazenadas agora eu editei o projeto pela tela de propriedades do projeto e as novas propriedades foram adicionadas no arquivo.

Quando descobri isso, fiquei extremamente animado pois quaisquer propriedades definidas no arquivo de projeto podem ser alteradas passando parâmetros via linha de comando na compilação, ou seja, eu não precisaria mais rodar o meu script e alterar arquivos do controle de versão para mudar o número de versão do assembly. Outra vantagem de ter essa informação no arquivo de projeto é que eu poderia compor os valores de forma mais simples e dinâmica, sem depender de código externo, e garantindo que essas informações estariam disponíveis e atuais em qualquer compilação, não apenas nos builds gerados no servidor.

Criei então um arquivo com extensão .props para a solution, que conteria todas as informações compartilhadas que quero atualizar e meu único trabalho agora é importar esse arquivo em todos os projetos de cada solution. Este arquivo de definições ficou bem completo e muito melhor de manter e compor do que usando o script. Além disso passei a ser capaz de definir até algumas informações dinâmicas como o ano atual.

Estava tudo ótimo, até que descobri que não consigo ter um projeto web project convencional (ou WPF, ou qualquer outra coisa que não seja Class Library ou Console) usando o novo formato de arquivo. Sem o novo formato baseado em SDK, não consigo tirar proveito dessa nova facilidade de definir essas informações como variáveis do projeto e, consequentemente, não consigo uma forma única de manter essa informação de versão para todos os projetos da solution nem posso alterar essas informações via linha de comando. Voltei a estaca zero.

Mas agora já tinha ido longe demais para desistir, e continuei pesquisando. Foi assim que encontrei um post de 2014 de Lev Gimelfarb mostrando uma abordagem bem legal simplesmente usando recursos do MSBuild, sem requerer arquivos ou bibliotecas externas. A solução dele foca apenas em alterar o AssemblyVersion, mas a ideia estava lá e tudo que eu tive que fazer foi adaptar.

Usando o código dele como base, editei o meu arquivo .props. Meu novo arquivo props agora continha alguns blocos novos com condicionais para que sejam executados apenas em projetos que não sejam no novo formato SDK. O código que fiz usa as mesma variáveis que são usadas pelo projeto SDK. O que ele faz é:

- Remove o arquivo AssemblyInfo da lista de arquivos para compilar do projeto atual (.cs ou .vb)
- Cria uma cópia desse arquivo na pasta obj e adiciona na lista de compilação, assim o arquivo alterado não será o do controle de versão já que a pasta obj é ignorada e o original se mantém inalterado
- Substitui todos os valores dos atributos no novo AssemblyInfo pelos definidos nas variáveis do projeto. Se houver valor na variável para o qual o atributo não exista, este é incluído no arquivo.</li></ul><p>Agora, graças a esse truque, não precisarei mais do script que altera o número de versão. Basta eu passar as propriedades que eu quiser como parâmetro para o MSBuild durante a compilação que os assemblies gerados usaram esses novos valores. Se eu não usar nada, serão usados os valores padrão que estão definidos no arquivo de projeto. Nesse meu projeto, os valores padrão são definidos de tal forma que seja fácil de identificar a versão e origem do assembly (desenvolvedor ou assembly oficial do servidor). Além disso, a mesma regra será aplicada a todos os projetos da solution que importarem esse arquivo .props.
