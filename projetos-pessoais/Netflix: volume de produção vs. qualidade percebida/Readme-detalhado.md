# Documentação detalhada: Netflix, volume vs. qualidade

Este documento explica o projeto do começo ao fim: a pergunta, os dados, cada etapa do tratamento em Python, a modelagem e as medidas no Power BI, os problemas encontrados no caminho, o design do dashboard e as conclusões com as respectivas ressalvas.

O resumo do projeto está no [`README.md`](README.md).

## Sumário

1. [Contexto e pergunta de negócio](#1-contexto-e-pergunta-de-negócio)
2. [Os dados](#2-os-dados)
3. [Tratamento em Python](#3-tratamento-em-python)
4. [Modelagem no Power BI](#4-modelagem-no-power-bi)
5. [Medidas e colunas DAX](#5-medidas-e-colunas-dax)
6. [Validação dos números](#6-validação-dos-números)
7. [Problemas encontrados e como foram resolvidos](#7-problemas-encontrados-e-como-foram-resolvidos)
8. [Design do dashboard](#8-design-do-dashboard)
9. [Resultados e conclusões](#9-resultados-e-conclusões)
10. [Limitações](#10-limitações)
11. [Próximos passos](#11-próximos-passos)
12. [Créditos e uso de IA](#12-créditos-e-uso-de-ia)

---

## 1. Contexto e pergunta de negócio

Catálogos de streaming crescem rápido, e uma plataforma precisa decidir **onde investir**: em mais volume nos gêneros e países que já domina, ou em áreas menores com melhor recepção do público.

A pergunta do projeto:

> A Netflix concentra seu catálogo nos gêneros e países que realmente performam bem com o público, ou existe um descompasso entre volume de produção e qualidade percebida?

Para responder, o projeto cruza duas informações de cada título: **quanto existe** (volume) e **como o público avalia** (nota do IMDb).

## 2. Os dados

**Fonte:** [Netflix TV Shows and Movies](https://www.kaggle.com/datasets/victorsoeiro/netflix-tv-shows-and-movies), de Victor Soeiro no Kaggle. Os dados de catálogo foram coletados em 2022 via JustWatch, com notas do IMDb e do TMDb.

O dataset tem dois arquivos. Este projeto usa apenas o `titles.csv` (5.850 títulos, 15 colunas). O `credits.csv` (elenco e direção) não foi usado.

### Dicionário de colunas

**Tabela fato: `netflix_tratado.csv`** (1 linha = 1 título)

| Coluna | Descrição |
|---|---|
| `id` | Identificador único do título. É a chave de ligação com as outras tabelas |
| `title` | Nome do filme ou série |
| `type` | `MOVIE` (filme) ou `SHOW` (série) |
| `description` | Sinopse |
| `release_year` | Ano de lançamento |
| `age_certification` | Classificação indicativa (nulos preenchidos com "Não informado") |
| `runtime` | Duração em minutos (filme: duração total; série: duração média do episódio) |
| `genres` | Gêneros concatenados em texto, só para referência |
| `production_countries` | Países concatenados em texto, só para referência |
| `seasons` | Número de temporadas (só existe para séries) |
| `imdb_id` | Código do título no IMDb |
| `imdb_score` | Nota do IMDb (0 a 10) |
| `imdb_votes` | Quantidade de votos que geraram a nota |
| `tmdb_popularity` | Índice de popularidade do TMDb (não é nota) |
| `tmdb_score` | Nota do TMDb (0 a 10) |
| `decada` | **Criada:** década de lançamento (2010, 2020...) |
| `tem_nota_imdb` | **Criada:** indica se o título tem nota do IMDb |
| `alta_performance` | **Criada:** nota ≥ 7 e votos acima da mediana |

**Dimensões explodidas**

| Arquivo | Colunas | Observação |
|---|---|---|
| `netflix_generos.csv` | `id`, `title`, `genero` | Um título com 3 gêneros gera 3 linhas |
| `netflix_paises.csv` | `id`, `title`, `pais` | Um título com 2 países gera 2 linhas. Países em código ISO (`US`, `GB`...) |

## 3. Tratamento em Python

Script: [`scripts/tratamento.py`](scripts/tratamento.py). O que cada etapa faz e por quê:

| # | Etapa | Por quê |
|---|---|---|
| 1 | Carregar o `titles.csv` | Ponto de partida: 5.850 linhas |
| 2 | Remover duplicatas por `id` | Garante 1 linha por título |
| 3 | Remover o título com nome nulo (1 caso) | Linha sem identificação não serve para análise |
| 4 | Converter `genres` e `production_countries` de texto para lista, com `ast.literal_eval` | No CSV, as listas estão gravadas como texto (`"['drama', 'crime']"`). Sem converter, não é possível separar os itens |
| 5 | Preencher `age_certification` nula com "Não informado" | Evita perder títulos em filtros por classificação |
| 6 | Criar `decada` | Permite agrupar a evolução do catálogo |
| 7 | Criar `tem_nota_imdb` | Marca quais títulos podem entrar em médias de nota, sem inventar valores para os que não têm |
| 8 | Criar `alta_performance` | Nota ≥ 7 **e** votos ≥ mediana (2.233,5). Evita que um título com nota 9 e 4 votos pareça sucesso |
| 9 | Explodir `genres` com `.explode()` | Uma linha por gênero, para a contagem por gênero ficar correta |
| 10 | Explodir `production_countries` com `.explode()` | Mesma lógica, por país |
| 11 | Exportar os 3 CSVs | Fato + 2 dimensões, prontos para o Power BI |

Resultado: **5.849 títulos** (3.743 filmes e 2.106 séries), com 15.088 linhas na tabela de gêneros.

**Decisões de método que vale destacar:**

- **Mediana, não média, no corte de votos.** A mediana não é distorcida por poucos títulos com centenas de milhares de votos.
- **Nulos de nota não foram preenchidos.** Inventar uma nota distorceria todas as médias. Em vez disso, a flag `tem_nota_imdb` deixa claro quem entra no cálculo.
- **`seasons` nulo em filmes é esperado**, não é erro. Médias de temporadas filtram só séries.

## 4. Modelagem no Power BI

O modelo tem a tabela fato no centro e duas dimensões ligadas pela coluna `id`:

```
netflix_generos (*) ──id──> (1) netflix_tratado (1) <──id── (*) netflix_paises
```

**Ponto de atenção: a direção do filtro.** Como as dimensões foram explodidas (um título aparece várias vezes), elas ficam do lado "muitos" do relacionamento, o oposto de um modelo estrela tradicional. Por padrão, o Power BI só propaga filtros do lado "um" para o lado "muitos", ou seja, de `netflix_tratado` para as dimensões. A análise precisa do caminho inverso: filtrar por `genero` ou `pais` e ver o efeito nas notas da tabela fato.

Solução: **direção do filtro cruzado = Ambos** nos dois relacionamentos. Aqui isso é seguro, porque há só uma dimensão em cada ponta e não existe caminho ambíguo entre elas.

As medidas ficam em uma tabela própria chamada `Medidas`.

## 5. Medidas e colunas DAX

### Página 1: visão geral

```dax
Total_Titulos = COUNTROWS(netflix_tratado)

Total_Filmes =
CALCULATE(COUNTROWS(netflix_tratado), netflix_tratado[type] = "MOVIE")

Total_Series =
CALCULATE(COUNTROWS(netflix_tratado), netflix_tratado[type] = "SHOW")

% Alta_Performance =
DIVIDE(
    CALCULATE(COUNTROWS(netflix_tratado), netflix_tratado[alta_performance] = "True"),
    COUNTROWS(netflix_tratado)
)

Media_Temporadas =
CALCULATE(AVERAGE(netflix_tratado[seasons]), netflix_tratado[type] = "SHOW")
```

`Media_Temporadas` filtra só séries porque filmes não têm temporada. Sem o filtro, a média seria distorcida.

### Página 2: gêneros

```dax
Nota_Media_Genero =
CALCULATE(
    AVERAGE(netflix_tratado[imdb_score]),
    netflix_tratado[tem_nota_imdb] = "True"
)

Qtd_Titulos_Genero =
CALCULATE(
    DISTINCTCOUNT(netflix_generos[id]),
    ALLEXCEPT(netflix_generos, netflix_generos[genero]),
    netflix_tratado[tem_nota_imdb] = "True"
)
```

`Qtd_Titulos_Genero` conta só títulos **com nota**, para o número ao lado da média representar exatamente quantos títulos entraram no cálculo. `DISTINCTCOUNT` evita contar o mesmo título duas vezes.

### Página 3: países

```dax
Nota_Media_Pais =
CALCULATE(
    AVERAGE(netflix_tratado[imdb_score]),
    netflix_tratado[tem_nota_imdb] = "True"
)

Qtd_Titulos_Pais =
CALCULATE(
    DISTINCTCOUNT(netflix_paises[id]),
    ALLEXCEPT(netflix_paises, netflix_paises[pais]),
    netflix_tratado[tem_nota_imdb] = "True"
)

Indice_Eficiencia_Pais =
DIVIDE([Nota_Media_Pais], DIVIDE([Qtd_Titulos_Pais], 100))
```

A coluna calculada `Nome_Pais` traduz os códigos para nomes em português (`SWITCH` com 109 países). O código completo está em [`tema/coluna_nome_pais.dax.txt`](tema/coluna_nome_pais.dax.txt).

**Sobre o `Indice_Eficiencia_Pais`:** a divisão pela quantidade faz o índice ser dominado pelo denominador. Países com 1 título e nota alta têm índice altíssimo, sem significado estatístico. Por isso ele aparece só como coluna de apoio na tabela, e a análise principal usa nota média, com o filtro de mínimo de 10 títulos.

### Página 4: cartões e gráficos de apoio

```dax
Nota_Drama =
CALCULATE([Nota_Media_Genero], netflix_generos[genero] = "drama")

Nota_Coreia =
CALCULATE([Nota_Media_Pais], netflix_paises[pais] = "KR")

Nota_Series_vs_Filmes =
VAR Series =
    CALCULATE(AVERAGE(netflix_tratado[imdb_score]), netflix_tratado[type] = "SHOW")
VAR Filmes =
    CALCULATE(AVERAGE(netflix_tratado[imdb_score]), netflix_tratado[type] = "MOVIE")
RETURN
    FORMAT(Series, "0.00") & " vs " & FORMAT(Filmes, "0.00")

Nota_Decadas =
VAR Nota2000 =
    CALCULATE(AVERAGE(netflix_tratado[imdb_score]), netflix_tratado[decada] = 2000)
VAR Nota2020 =
    CALCULATE(AVERAGE(netflix_tratado[imdb_score]), netflix_tratado[decada] = 2020)
RETURN
    FORMAT(Nota2000, "0.00") & " → " & FORMAT(Nota2020, "0.00")

Nota_Media_Titulos =
CALCULATE(
    AVERAGE(netflix_tratado[imdb_score]),
    netflix_tratado[tem_nota_imdb] = "True"
)

Qtd_Com_Nota =
CALCULATE(COUNTROWS(netflix_tratado), netflix_tratado[tem_nota_imdb] = "True")

Linha_Media_Nota =
CALCULATE(AVERAGE(netflix_tratado[imdb_score]), ALL(netflix_tratado))
```

Colunas calculadas de apoio:

```dax
Tipo_PT = IF(netflix_tratado[type] = "SHOW", "Séries", "Filmes")

Destaque = IF(netflix_generos[genero] = "drama", "Drama", "Outros gêneros")
```

`Destaque` alimenta a legenda do gráfico de dispersão para pintar o drama de vermelho e os demais de cinza. `Linha_Media_Nota` alimenta a linha de referência (6,51) do mesmo gráfico.

## 6. Validação dos números

Todas as medidas foram **conferidas contra um cálculo independente em Python** antes de entrarem no dashboard. Exemplos:

| Medida | Power BI | Python |
|---|---|---|
| Total de títulos | 5.849 | 5.849 |
| Filmes / Séries | 3.743 / 2.106 | 3.743 / 2.106 |
| % Alta performance | 21,15% | 21,15% |
| Média de temporadas (séries) | 2,16 | 2,16 |
| Nota média, gênero `history` | 7,13 | 7,13 |
| Nota média, gênero `drama` (2.827 títulos) | 6,62 | 6,62 |
| Nota média, país `KR` (197 títulos) | 7,23 | 7,23 |
| Índice de eficiência, `AE` | 30,00 | 30,00 |

Os 19 gêneros e os países da tabela foram conferidos linha a linha.

## 7. Problemas encontrados e como foram resolvidos

Esta seção documenta os erros reais do processo. Cada um é um bug comum em projetos de BI.

| # | Sintoma | Causa | Solução |
|---|---|---|---|
| 1 | `Media_Temporadas` = 21,63 em vez de 2,16 (10× maior) | Ao importar o CSV com configuração regional em português, o ponto decimal (`2.0`) foi lido como separador de milhar (`20`) | Alterar o tipo da coluna no Power Query **usando localidade** "Inglês (Estados Unidos)" |
| 2 | Erro em `CALCULATE(..., coluna = TRUE)` | O Python grava booleanos como texto (`True`/`False`) e o Power BI os importou como texto | Comparar com texto: `= "True"` |
| 3 | `Nota_Media_Genero` = 6,51 para **todos** os gêneros | O filtro de `genero` não chegava à tabela fato (relacionamento só no sentido "um para muitos") | Mudar a direção do filtro cruzado para **Ambos** |
| 4 | `Qtd_Titulos_Genero` maior que o esperado | A medida contava também títulos sem nota, enquanto a média só considerava títulos com nota | Adicionar o filtro `tem_nota_imdb = "True"` à contagem |
| 5 | `Indice_Eficiencia_Pais` não batia com a tabela | A fórmula dividia pela contagem total de títulos, não pela medida corrigida `Qtd_Titulos_Pais` | Referenciar as medidas existentes entre colchetes em vez de recalcular |
| 6 | Erro de comparação texto × inteiro em `Nota_Decadas` | A coluna `decada` foi importada como texto | Converter a coluna para número inteiro no Power Query (ou comparar com `"2000"`) |

**Lição geral:** duas medidas que aparecem lado a lado precisam compartilhar o mesmo critério de filtro. Referenciar medidas prontas (`[Nome_Medida]`) evita inconsistências.

**Ponto de atenção conhecido:** um título tem o país escrito por extenso (`Lebanon`) em vez do código (`LB`). Isso faz o Líbano aparecer em duas linhas na tabela de países. O ideal é padronizar no tratamento, trocando `Lebanon` por `LB` antes de exportar.

## 8. Design do dashboard

### Identidade visual

Paleta inspirada na marca da plataforma, com um tema personalizado em JSON, disponível nos modos escuro e claro (pasta `tema/`).

| Uso | Cor |
|---|---|
| Fundo da página (escuro) | `#141414` |
| Fundo dos visuais (escuro) | `#1F1F1F` |
| Destaque principal | `#E50914` |
| Vinho / coral (apoio ao vermelho) | `#B20710` / `#F6575F` |
| Valores bons | `#46D369` |
| Neutro ou comparação | `#5B8DB8` |
| Atenção | `#F5A623` |
| Texto secundário | `#8C8C8C` |
| Texto principal | `#F5F5F1` |

Princípios aplicados:

- No máximo 3 cores por visual
- **Vermelho só no que importa:** o elemento que o gráfico quer destacar
- Eixos que não começam em zero recebem aviso na legenda, para a diferença visual não parecer maior do que é
- Mesmo menu de navegação, com ícones, nas 4 páginas
- Mesma cor para a mesma categoria em todas as páginas (filmes em vermelho e séries em azul-acinzentado)

### Visuais por página

| Página | Visuais |
|---|---|
| 1. Visão geral | 5 cartões (total, filmes, séries, % alta performance, média de temporadas), linha por ano de lançamento, colunas por década, mapa por país |
| 2. Gêneros | Dispersão volume × nota (com linha da média e quadrantes), barras de nota, barras de volume |
| 3. Países | Dispersão volume × nota, Top 10 por nota, tabela de detalhe (filtro de mínimo de 10 títulos) |
| 4. Storytelling | 4 cartões de insight com mini-gráficos, recomendação e limitações |

## 9. Resultados e conclusões

Nota média geral da base: **6,51**.

### 1. Os gêneros mais produzidos não são os mais bem avaliados

| Gênero | Títulos com nota | Nota média |
|---|---|---|
| História | 254 | 7,13 |
| Guerra | 155 | 7,07 |
| Documentário | 859 | 7,01 |
| Drama | 2.827 | 6,62 |
| Comédia | 2.218 | 6,39 |
| Thriller | 1.180 | 6,37 |
| Terror | 366 | 6,01 |

**Interpretação:** o drama é o gênero com mais títulos, mas está em 7º lugar em nota. Comédia e thriller, também de alto volume, ficam abaixo da média geral. Já história e guerra têm pouco volume e notas acima de 7,0, com cerca de 46% dos títulos classificados como alta performance, contra 29% do drama. O catálogo se concentra onde há mais produção, não onde há mais qualidade percebida.

### 2. O maior produtor fica na média; os melhores estão em outros países

| País | Títulos com nota | Nota média |
|---|---|---|
| Coreia do Sul | 197 | 7,23 |
| Japão | 259 | 6,97 |
| Reino Unido | 378 | 6,80 |
| Estados Unidos | 2.163 | 6,53 |

**Interpretação:** os EUA produzem cerca de 11 vezes mais títulos que a Coreia do Sul, mas ficam praticamente na média geral. Países com produção menor (Coreia do Sul, Japão e Reino Unido) entregam notas consideravelmente maiores. Os países considerados nessa leitura têm no mínimo 10 títulos com nota, para evitar médias instáveis.

### 3. Séries são mais bem avaliadas que filmes

Séries: nota média **6,98** e **32,3%** de alta performance. Filmes: **6,25** e **17,8%**.

**Interpretação:** séries superam filmes nas duas medidas. **Ressalva:** pode haver viés de seleção, já que séries com nota no IMDb tendem a ser as mais populares, o que pode inflar a média do grupo.

### 4. O catálogo cresceu e a nota média caiu

| Década | Títulos com nota | Nota média |
|---|---|---|
| 2000 | 345 | 6,85 |
| 2010 | 3.092 | 6,58 |
| 2020 | 1.735 | 6,29 |

**Interpretação:** à medida que o volume cresce, a nota média diminui. **Ressalvas:** os anos 2020 estão incompletos na coleta (2022), e títulos mais recentes tendem a ter menos votos. O resultado indica uma associação, não prova que mais volume cause nota menor.

### Recomendação

Priorizar **séries** e **gêneros de nicho com nota alta** (história, guerra, documentário), e ampliar parcerias com mercados que entregam qualidade consistente, como **Coreia do Sul, Japão e Reino Unido**, em vez de apostar em volume nos gêneros mais saturados (comédia, thriller, terror).

## 10. Limitações

- **A nota mede avaliação do público, não resultado de negócio.** Não há dados de audiência, custo de produção ou receita. A recomendação é qualitativa, não um cálculo de retorno
- **Dados de 2022.** O panorama do catálogo mudou desde então, e os anos 2020 estão incompletos
- **Cerca de 8% dos títulos não têm nota** e ficaram fora das médias
- **Viés de seleção nas notas:** títulos com nota no IMDb não são uma amostra aleatória do catálogo
- **Amostras pequenas:** países e gêneros com poucos títulos têm médias instáveis (por isso o corte de 10 títulos por país)
- **Análise descritiva:** os resultados mostram associações, não causas

## 11. Próximos passos

- Padronizar `Lebanon` → `LB` no tratamento
- Incluir o `credits.csv` para analisar elenco e direção por nota
- Comparar com um catálogo mais recente, para ver se os padrões se mantêm
- Testar a diferença de notas entre grupos com um teste estatístico, para saber se é significativa
- Publicar o relatório no Power BI Service com o link no portfólio

## 12. Créditos e uso de IA

Este projeto foi desenvolvido **em colaboração com o Claude**, assistente de IA da Anthropic.

| Parte | Responsável |
|---|---|
| Design e montagem do dashboard no Power BI | Gabriela |
| Planejamento da análise e da estrutura de páginas | Claude, com decisões da autora |
| Script de tratamento em Python | Claude |
| Medidas e colunas DAX | Claude, validadas pela autora contra os cálculos em Python |
| Tema em JSON e paleta de cores | Claude, com escolhas da autora |
| Depuração dos problemas de importação e de modelo | Claude e autora, em conjunto |
| Textos e conclusões | Claude, revisados pela autora |

**Dataset:** Victor Soeiro, [Netflix TV Shows and Movies](https://www.kaggle.com/datasets/victorsoeiro/netflix-tv-shows-and-movies), no Kaggle. Os dados de notas pertencem ao IMDb e ao TMDb. Consulte os termos de uso do dataset no Kaggle antes de redistribuí-lo.
