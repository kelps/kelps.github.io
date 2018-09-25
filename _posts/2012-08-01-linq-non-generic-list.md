---
layout: blogger
title: Usando LINQ com listas não genéricas
date: 2012-08-01T10:45:00.0010000-03:00
permalink: /:year/:month/:slug:output_ext
tags: [Dicas, LINQ]
---
Hoje, mais uma vez eu me deparei com um “problema” relativamente comum: Usar LINQ em uma lista, coleção ou enumerável não generic (ocorre bastante quando se lida com código legado ou de aplicações mais antigas). Isso não é possível nativamente pois o LINQ trabalha apenas com generics por causa de inferencia de tipos.

Para não ter que colocar loopings no código apenas para migrar de não generico para genérico há 2 Extension methods bem simples que podemos usar:


1. ```Cast<type>()``` – converte a coleção inteira para um IEnumerable<type> do tipo informado. Praticamente não há impacto de performance pois internamente ele simplesmente faz um cast explicito para o tipo solicitado e devolve o objeto. O único problema deste método é que se houver algum objeto na coleção sendo convertida que não seja do tipo solicitado, ocorrerá uma exceção quando seu código chegar neste ítem.
2. ```OfType<type>()``` – converte a coleção para um IEnumerable<type> do tipo informado, mas contendo apenas com os ítens que são do tipo solicitado. Este é mais seguro para quando você estiver usando uma coleção que pode conter objetos de tipos diferentes mas apenas um tipo lhe interessa. Há um pequeno impacto de performance pois antes de retornar cada ítem este é testado para ver se é do tipo correto e se não for, os seguintes são testados até encontrar um que seja ou acabarem os ítens.


Ou seja, para listas de objetos em código legado não generic mas onde o tipo é conhecido e garantido, a primeria opção é melhor. Para outros casos, use a segunda. Abaixo tem um pequeno exemplo de uso desse código:

```csharp
var col = new Collection();
col.Add("text");
col.Add("text2");
col.Add(1);
col.Add("text4");

var keysCast = col.Cast<string>();
var keysOfType = col.OfType<string>();

var resultadoT = (from k in keysOfType
                  select k).ToArray();

var resultadoC = (from k in keysCast
                  select k).ToArray();

```
Neste exemplo, o LINQ que popula resultadoT funcionará corretamente e resultadoT conterá 3 ítens, mas o LINQ de resultadoC vai gerar uma exceção ao tentar fazer cast explicito do inteiro 1 para string. Se a linha que acrescenta 1 na collection col for comentada, as duas queries LINQ funcionarão.
