Nós analisamos visualmente a média dos filmes do TMDB 5000, e percebemos que seu comportamento é parecido com uma distribuição normal, com exceção do lado esquerdo do gráfico. 

![histograma plotando as médias dos filmes no tmdb 5000. no eixo x, temos as notas de 0 a 9. porém, a distribuição parece populada apenas de 1 a 9. no eixo y, temos a densidade do conjunto, que vai de 0 a 0,7.](https://s3.amazonaws.com/caelum-online-public/1112+-+data-science-testes-estatisticos/Transcri%C3%A7%C3%A3o/Imagens/histogramav5.png)

Agora, vamos comparar esse conjunto com os dados do MovieLens, procurando saber se esse comportamento se repete. Primeiramente, carregaremos o arquivo  `ratings.csv` e o importaremos, atribuindo essa leitura a uma variável `notas`. Feito isso, exibiremos os `5` primeiros registros desse *dataset*:

```
notas = pd.read_csv("ratings.csv")
notas.head()
```

Temos aqui um formato diferente, com a identificação do usuário (`userId`), a identificação do filme (`movieId`) e a nota que foi dada para cada filme (`rating`). 

|   | userId | movieId | rating | timestamp |
|---|--------|---------|--------|-----------|
| 0 | 1      | 1       | 4.0    | 964982703 |
| 1 | 1      | 3       | 4.0    | 964981247 |
| 2 | 1      | 6       | 4.0    | 964982224 |
| 3 | 1      | 47      | 5.0    | 964983815 |
| 4 | 1      | 50      | 5.0    | 964982931 |

Ou seja, não temos as médias de cada filme, então teremos que calculá-la. Para isso, agruparemos as notas por filme (`notas.groupby("movieId")`) e tiraremos a média apenas do campo `rating` (o único em que essa medida faz sentido), atribuindo o retorno a uma variável `nota_media_por_filme`. Então, exibiremos os `5` primeiros registros: 

```
nota_media_por_filme = notas.groupby("movieId").mean()["rating"]
nota_media_por_filme.head()
```

| movieId |                |
|---------|----------------|
| 1       | 3.920930       |
| 2       | 3.431818       |
| 3       | 3.259615       |
| 4       | 2.357143       |
| 5       | 3.071429       |

Em seguida, geraremos a primeira visualização dessas médias. Para isso, repetiremos o processo que fizemos para as médias do TMDB 5000, acrescentando `.values` à variável `nota_media_por_filme` de modo a pegar somente os valores, e não os *ids* de cada filme: 

```
ax = sns.distplot(nota_media_por_filme.values)
ax.set(xlabel='Nota média', ylabel='Densidade')
ax.set_title('Média de votos em filmes no MovieLens')
```

Dessa vez, temos um gráfico com notas de `0` a `5`. Novamente, ainda que não tenhamos filmes com média `0`, temos alguns cuja média é `5`. Portanto, é de se esperar que existam alguns filmes com poucos votos.

![histograma plotando as médias dos filmes no Movie Lens. no eixo x, temos notas de 0 até 5. no eixo y, temos a densidade, que vai de 0 até 1. a maior parte das médias parece se concentrar entre 2,5 e 4,5](https://s3.amazonaws.com/caelum-online-public/1112+-+data-science-testes-estatisticos/Transcri%C3%A7%C3%A3o/Imagens/histograma2.png)

Com `notas.groupby("movieId").count()`, contaremos quantos votos cada um dos filmes possui nesse conjunto, atribuindo o resultado a uma variável `quantidade_de_votos_por_filme`. Com ela, faremos uma `query()` que selecionará apenas os filmes com `10` ou mais votos.


```
quantidade_de_votos_por_filme = notas.groupby("movieId").count()
quantidade_de_votos_por_filme.query("rating >= 10")
```

Quero, então, saber quais são os filmes que compõem esse conjunto. Para isso, utilizaremos o `index` (que se refere a `movieId`), já que esse campo é um índice e não uma coluna determinada. Criaremos, com essa `query()`, uma variável `filmes_com_pelo_menos_10_votos`. 

```
quantidade_de_votos_por_filme = notas.groupby("movieId").count()
filmes_com_pelo_menos_10_votos = quantidade_de_votos_por_filme.query("rating >= 10").index
```

O formato devolvido por essa função é `Int64Index`. Podemos extrair somente os valores, criando um *array*, com `filmes_com_pelo_menos_10_votos.values`.

A variável `nota_media_por_filme`, que criamos anteriormente, é uma série que contém as médias e os *ids* dos filmes no conjunto. Agora, queremos somente os valores cujo índice está contido em `filmes_com_pelo_menos_10_votos`.

Para isso, usaremos o indexador `.loc[]`. Como parâmetro, ele pode receber um índice ou até mesmo um array, que é exatamente o que temos em `filmes_com_pelo_menos_10_votos`! Atribuiremos a *series* retornada pelo `.loc[]` a uma nova variável, de nome bastante extenso mas igualmente explicativo: 

```
nota_media_dos_filmes_com_pelo_menos_10_votos = nota_media_por_filme.loc[filmes_com_pelo_menos_10_votos.values]
nota_media_dos_filmes_com_pelo_menos_10_votos.head()
```

| movieId |          |
|---------|----------|
| 1       | 3.920930 |
| 2       | 3.431818 |
| 3       | 3.259615 |
| 5       | 3.071429 |
| 6       | 3.946078 |

Agora, podemos gerar novamente um histograma, agora com dados mais significantes:

```
ax = sns.distplot(nota_media_dos_filmes_com_pelo_menos_10_votos)
ax.set(xlabel='Nota média', ylabel='Densidade')
ax.set_title('Média de votos em filmes no MovieLens')
```

Com isso, teremos uma visualização que se assemelha mais à do TMDB 5000, sem os extremos da esquerda (`0.5`) e da direita (`5`). 

![mesmo histograma plotado anteriormente, dessa vez removendo as médias 0 e 5 do conjunto, o que torna a distribuição mais uniforme e semelhante a uma normal](https://s3.amazonaws.com/caelum-online-public/1112+-+data-science-testes-estatisticos/Transcri%C3%A7%C3%A3o/Imagens/histograma2v2.png)

Se traçarmos uma "mediana imaginária" por volta do  `3,5`, teremos uma curva um pouco mais abaulada à esquerda, e um pouco mais inclinada à direita - da mesma forma que no histograma do TMBD 5000. Claro, podem existir outros fatores que tornam a distribuição daquele conjunto mais "espalhada". 

Vamos gerar também o *boxplot* dos dados do MovieLens:

```
ax = sns.boxplot(x=nota_media_dos_filmes_com_pelo_menos_10_votos.values)
ax.set(xlabel='Nota  média do filme')
ax.set_title('Distribuição de nota média dos filmes do MovieLens')
```

Repare que essa nova visualização também se assemelha àquela do TMDB, com os quartis à esquerda (`25%` e `50%`) mais densos que os da direita. 

![boxplot das médias dos filmes no MovieLens. os valores no eixo x, que representam as médias, vão de 1,5 até 4,5. porém, as plotagens parecem excedê-los, tanto para a esquerda quanto para a direita, mesmo que em poucos décimos. a mediana está próxima de 3,5, e a maioria dos dados (50%) se concentra entre aproximadamente 3,1 e 3,8](https://s3.amazonaws.com/caelum-online-public/1112+-+data-science-testes-estatisticos/Transcri%C3%A7%C3%A3o/Imagens/boxplot2.png)

Uma análise "visual" dessas distribuições está nos indicando que as pessoas se comportam de maneira similar. A seguir, continuaremos explorando esses dados.