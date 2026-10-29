# 🎬 Netflix: volume de produção vs. qualidade percebida

Análise do catálogo da Netflix com **Python** (tratamento) e **Power BI** (dashboard de 4 páginas), para responder a uma pergunta de negócio:

> **A Netflix concentra seu catálogo nos gêneros e países que realmente performam bem com o público, ou existe um descompasso entre volume de produção e qualidade percebida?**

![Capa do dashboard](docs/img/capa.png)

---

## 🔎 Principais resultados

| # | Achado | Números |
|---|---|---|
| 1 | Os gêneros mais produzidos não são os mais bem avaliados | Drama: 2.827 títulos e nota 6,62 (7º lugar). Comédia: 6,39. Thriller: 6,37. História e guerra passam de 7,0 com poucos títulos |
| 2 | O maior produtor fica na média; os melhores estão em outros países | EUA: 2.163 títulos e nota 6,53. Coreia do Sul: 197 títulos e nota 7,23, a maior entre países com 10+ títulos |
| 3 | Séries são mais bem avaliadas que filmes | Nota média 6,98 contra 6,25. Alta performance: 32% das séries contra 18% dos filmes |
| 4 | O catálogo cresceu e a nota média caiu | Anos 2000: 345 títulos, nota 6,85. Anos 2010: 3.092 títulos, nota 6,58. Anos 2020: 1.735 títulos, nota 6,29 |

Nota média geral da base: **6,51** (IMDb, 0 a 10).

> As conclusões completas, com a interpretação e as ressalvas de cada uma, estão em [`README_DETALHADO.md`](README_DETALHADO.md).

---

## 📊 O dashboard

| Página | O que responde |
|---|---|
| **1. Visão geral** | Qual o tamanho e a composição do catálogo? (títulos, filmes vs séries, evolução por ano e década, mapa por país) |
| **2. Gêneros** | Quais gêneros têm volume alto e quais têm nota alta? (dispersão volume × nota, rankings) |
| **3. Países** | Quais países produzem mais e quais produzem melhor? (dispersão, Top 10 por nota, tabela com índice de eficiência) |
| **4. Storytelling** | Qual a história que os dados contam e o que recomendar? (4 insights, recomendação e limitações) |

![Página 1](docs/img/pagina1.png)
![Página 2](docs/img/pagina2.png)
![Página 3](docs/img/pagina3.png)
![Página 4](docs/img/pagina4.png)

---

## 🛠️ Tecnologias

- **Python** (pandas, ast) para limpeza e preparação dos dados
- **Power BI** (modelagem, DAX, tema personalizado em JSON) para o dashboard
- **Dataset:** [Netflix TV Shows and Movies](https://www.kaggle.com/datasets/victorsoeiro/netflix-tv-shows-and-movies), de Victor Soeiro (Kaggle), com dados de catálogo, notas do IMDb e do TMDb (coleta de 2022)

---

## 🗂️ Estrutura do repositório

```
.
├── README.md                  ← este arquivo
├── README_DETALHADO.md        ← passo a passo, medidas DAX, problemas resolvidos e conclusões
├── dados/
│   ├── titles.csv             ← dados originais (Kaggle)
│   ├── netflix_tratado.csv    ← tabela fato (1 linha por título)
│   ├── netflix_generos.csv    ← dimensão de gêneros (1 linha por gênero de cada título)
│   └── netflix_paises.csv     ← dimensão de países (1 linha por país de cada título)
├── scripts/
│   └── tratamento.py          ← script de tratamento
├── dashboard/
│   └── netflix.pbix           ← arquivo do Power BI
├── tema/
│   ├── tema_netflix_powerbi.json         ← tema escuro
│   ├── tema_netflix_claro_powerbi.json   ← tema claro
│   └── coluna_nome_pais.dax.txt          ← coluna DAX com nomes dos países
└── docs/img/                  ← prints do dashboard
```

---

## ▶️ Como reproduzir

1. Baixe o dataset no Kaggle e coloque `titles.csv` na pasta `dados/`
2. Instale as dependências: `pip install pandas`
3. Rode o tratamento: `python scripts/tratamento.py` (gera os 3 CSVs tratados)
4. Abra `dashboard/netflix.pbix` no Power BI Desktop e atualize as fontes de dados para a sua pasta local
5. Para aplicar o visual: **Exibição > Temas > Procurar temas** e escolha um dos arquivos da pasta `tema/`

> Ao importar os CSVs no Power BI com configuração regional em português, confira o tipo das colunas numéricas (veja a seção de problemas resolvidos no `README_DETALHADO.md`).

---

## ⚠️ Limitações

- A nota do IMDb mede a **avaliação do público**, não audiência nem retorno financeiro. A base não tem custo de produção nem visualizações
- Os dados são de **2022**, e os anos 2020 estão incompletos
- Cerca de **8% dos títulos não têm nota** e ficaram fora das médias
- Países com poucos títulos têm médias instáveis, por isso a análise por país usa o mínimo de 10 títulos
- Análise **descritiva**: correlação não é causalidade

---

## 🤝 Créditos

- **Design e montagem do dashboard:** Gabriela
- **Planejamento da análise, código Python, medidas DAX, tema e textos:** desenvolvidos com o **Claude** (Anthropic), assistente de IA, em colaboração com a autora

---

## 👩‍💻 Autora

**Gabriela** · Estudante de Ciência da Computação (CEUB)

[LinkedIn](https://www.linkedin.com/in/SEU-PERFIL) · [Portfólio](https://SEU-PORTFOLIO.lovable.app)
