# Análise de Engajamento e Métrica de Aprendizado de um Portal de Conteúdo

Projeto de análise de dados para definir e medir a *North Star Metric* de um portal de conteúdo educacional: **quantas pessoas realmente aprendem com o conteúdo a cada semana** (*Weekly Active Learners*, WAL).

> **Aviso sobre os dados:** este projeto foi anonimizado. Todos os números, temas, nomes e URLs de páginas exibidos nas imagens do dashboard são **fictícios**, gerados aleatoriamente. Por questões de confidencialidade, não é possível divulgar dados da empresa, e nenhum valor ou conteúdo aqui representa um portal real.

---

## Visão geral do dashboard

![Capa do dashboard](images/capa-dashboard.png)

---

## Design no Figma

O layout do dashboard foi prototipado no Figma antes da implementação no Power BI.

**[Ver o design no Figma](COLE-AQUI-O-LINK-DO-FIGMA)**

![Protótipo no Figma](images/figma-prototipo.png)

---

## Desafio

Aprendizado não pode ser medido diretamente. Foi preciso traduzi-lo em comportamentos observáveis usando apenas métricas já disponíveis no Google Analytics 4, sem desenvolvimento adicional.

**Definição adotada de *Weekly Active Learner*:** usuário que, dentro de uma semana, visualizou pelo menos 2 páginas de conteúdo e teve tempo médio de engajamento superior a 2 minutos.

**Learning Rate:** WAL ÷ usuários ativos, ou seja, o percentual de visitantes com comportamento compatível com aprendizado.

---

## Páginas do dashboard

> Os números, temas e URLs que aparecem nas imagens abaixo são fictícios.

### 1. Visão geral
![Visão geral](images/pagina-1-visao-geral.png)

KPIs principais, evolução semanal do WAL e do Learning Rate e usuários ativos.

### 2. Comportamento e conteúdo
![Comportamento e conteúdo](images/pagina-2-comportamento-conteudo.png)

Usuários novos vs. recorrentes, páginas por usuário e WAL por página.

### 3. Canais e aquisição
![Canais e aquisição](images/pagina-3-canais-aquisicao.png)

Origem dos usuários, tempo médio e taxa de engajamento por canal.

### 4. Temas e detalhe por página
![Temas e detalhe por página](images/pagina-4-temas-detalhe.png)

Engajamento por tema e tabela de auditoria com os indicadores de cada página e a indicação de quais atendem ao critério de WAL.

---

## O que foi feito

- Extração e consolidação de 8 semanas de dados de aquisição de tráfego e de páginas e telas.
- Modelagem no Power BI, com tabela de semanas, relacionamentos e medidas DAX: WAL, Learning Rate, usuários novos e recorrentes, páginas por usuário e por sessão e tempo médio de engajamento.
- Criação de coluna de tema a partir da estrutura das URLs, para analisar o engajamento por assunto.
- Dashboard em quatro páginas, com *slicers* por semana, tema e canal.
- Resumo executivo de uma página com conclusões e recomendações.

---

## Validação e qualidade dos dados

Todas as medidas foram recalculadas em **Python (pandas)** e comparadas com os resultados do Power BI. A validação revelou e corrigiu erros de modelagem, como:

- somas de percentuais no lugar de médias ponderadas;
- denominadores que ignoravam o filtro de semana;
- referência a uma coluna que não existia nos dados;
- medidas que misturavam tabelas sem relacionamento;
- decimais importados com a localidade errada, que faziam o critério do WAL aprovar todas as linhas.

Limitações documentadas: dupla contagem de usuários entre páginas e entre semanas, o que torna o WAL uma estimativa (*proxy*).

---

## Insights gerados

- Evolução semanal da métrica e do Learning Rate, com identificação de picos e análise de possíveis causas.
- Conteúdos e temas que mais engajam e canais que trazem usuários mais qualificados.
- Oportunidades de melhoria na navegação e na descoberta de conteúdo.

---

## Ferramentas

`Google Analytics 4` · `Power BI` · `DAX` · `Python (pandas)` · `Figma` · `Excel/CSV`

## Competências demonstradas

Definição de métricas de produto · Modelagem de dados · DAX avançado · Validação e auditoria de números · Design de dashboards · Visualização de dados · Comunicação executiva
