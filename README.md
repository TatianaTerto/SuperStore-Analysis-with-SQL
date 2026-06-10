# 🛒 SuperStore: Rentabilidade e Logística com SQL
### *SuperStore: Profitability and Logistics with SQL*

> **PT** — Quando o desconto vira prejuízo? **EN** — When does a discount become a loss?

Projeto de análise de dados desenvolvido durante o bootcamp da [Laboratoria](https://www.laboratoria.la/), investigando padrões de rentabilidade e eficiência logística em uma rede varejista global com operações em 38 países.

*Data analysis project developed during the [Laboratoria](https://www.laboratoria.la/) bootcamp, investigating profitability and logistics efficiency patterns across a global retail chain operating in 38 countries.*

---

## ❓ Pergunta central / Central Question

**PT** — Descontos aumentam receita ou destroem margem? E qual modal de entrega realmente compensa o custo?

**EN** — Do discounts increase revenue or destroy margin? And which shipping mode actually justifies its cost?

---

## 📦 Sobre os dados / About the Data

**PT**

O dataset é composto por duas tabelas públicas da SuperStore, unidas via `LEFT JOIN` com chave composta no BigQuery:

- `orders` — informações por pedido: país, categoria, produto, desconto, receita e lucro
- `people` — informações logísticas: modal de entrega, tempo de envio e região

**EN**

The dataset consists of two public SuperStore tables, joined via `LEFT JOIN` with a composite key in BigQuery:

- `orders` — order-level data: country, category, product, discount, revenue, and profit
- `people` — logistics data: shipping mode, delivery time, and region

|                                         |          |
|-----------------------------------------|----------|
| **Linhas por tabela / Rows per table**  | 19.960   |
| **Países / Countries**                  | 38       |
| **Categorias / Categories**             | 3        |
| **Período / Period**                    | 2020–2024 |

---

## 🧹 Processo de limpeza / Data Cleaning Process

**PT** — A limpeza foi realizada em SQL no BigQuery. Cada decisão foi documentada com a query utilizada e a justificativa.

**EN** — Cleaning was performed in SQL on BigQuery. Each decision was documented with the query used and its rationale.

---

### 1. Valores nulos / Null Values

**PT** — 22 valores nulos identificados na coluna `Sales`. Após investigação, os registros correspondiam a devoluções parciais com `Profit` preenchido. Os nulos foram mantidos e documentados como limitação conhecida.

**EN** — 22 null values identified in the `Sales` column. After investigation, the records corresponded to partial returns with `Profit` filled in. Nulls were kept and documented as a known limitation.

---

### 2. Duplicatas / Duplicates

**PT** — 16 registros duplicados identificados por `order_id`. Após cruzamento com a tabela de logística, confirmou-se que eram entregas parciais de um mesmo pedido — cada linha representava um envio distinto. Registros mantidos.

**EN** — 16 duplicate records identified by `order_id`. After cross-referencing with the logistics table, they were confirmed as partial deliveries of the same order — each row represented a distinct shipment. Records kept.

---

### 3. Variáveis categóricas / Categorical Variables

**PT** — Verificação de consistência nas colunas `Category`, `Ship Mode`, `Segment` e `Region`. Nenhuma inconsistência de formatação ou grafia encontrada.

**EN** — Consistency check on `Category`, `Ship Mode`, `Segment`, and `Region` columns. No formatting or spelling inconsistencies found.

---

### 4. Outliers

**PT** — 113 pedidos identificados com desconto igual ou superior a 50%. A média de desconto nesse grupo era de 58%. Em vez de removê-los, esse grupo tornou-se o objeto central da análise de impacto de desconto.

**EN** — 113 orders identified with discounts of 50% or more. The average discount in this group was 58%. Rather than removing them, this group became the central object of the discount impact analysis.

---

### 5. JOIN entre tabelas / Table Join

**PT** — As duas tabelas foram unidas via `LEFT JOIN` usando chave composta `order_id + product_id`. Uma subquery foi necessária para evitar multiplicação de linhas antes do join. 31 registros extras foram identificados e documentados como limitação; nenhum foi removido.

**EN** — Both tables were joined via `LEFT JOIN` using the composite key `order_id + product_id`. A subquery was required to prevent row multiplication before the join. 31 extra records were identified and documented as a known limitation; none were removed.

---

## 🔍 Análise / Analysis

**PT** — Quatro perguntas guiaram a investigação:

**EN** — Four questions guided the investigation:

---

### 1. Qual categoria é mais rentável? / Which category is most profitable?

**PT** — Cálculo de margem bruta por categoria, isolando pedidos com e sem desconto alto.

**EN** — Gross margin calculated by category, isolating orders with and without heavy discounts.

| Categoria / Category | Margem / Margin | Lucro médio com desconto alto / Avg profit under heavy discount |
|----------------------|-----------------|------------------------------------------------------------------|
| Technology           | 13,78%          | -$123 por pedido / per order                                     |
| Office Supplies      | 11,54%          | -$61 por pedido / per order                                      |
| Furniture            | 6,92%           | -$189 por pedido / per order                                     |

**PT** — Technology lidera em margem geral, mas é a categoria com maior queda absoluta de lucro quando o desconto é alto. Furniture opera na margem mais baixa mesmo sem desconto.

**EN** — Technology leads in overall margin, but shows the steepest absolute profit drop under heavy discounts. Furniture operates at the lowest margin even without discounting.

---

### 2. Existe um ponto de virada no desconto? / Is there a discount tipping point?

**PT** — Segmentação dos pedidos em faixas de desconto para identificar onde o lucro médio cruza zero.

**EN** — Orders were segmented into discount bands to identify where average profit crosses zero.

| Faixa / Band | Lucro médio / Avg profit |
|--------------|--------------------------|
| 0%           | +$68                     |
| 1–10%        | +$47                     |
| 11–20%       | +$12                     |
| 21–30%       | -$38                     |
| 31%+         | -$117                    |

**PT** — 20% é o ponto exato de virada: acima desse valor, todos os pedidos geram prejuízo sem exceção. As 3.850 ordens nas faixas críticas (21%+) representam aproximadamente $294k de prejuízo evitável.

**EN** — 20% is the exact tipping point: above that threshold, every order generates a loss, without exception. The 3,850 orders in the critical bands (21%+) represent approximately $294k in avoidable losses.

---

### 3. Qual modal de entrega compensa o custo? / Which shipping mode justifies its cost?

**PT** — Cruzamento das duas tabelas para calcular custo total e lucro gerado por modal.

**EN** — Cross-table analysis to calculate total cost and profit generated per shipping mode.

| Modal / Mode   | Custo total / Total cost | Lucro gerado / Profit generated |
|----------------|--------------------------|----------------------------------|
| Same Day       | $45.000                  | $33.000                          |
| First Class    | $31.000                  | $29.000                          |
| Second Class   | $22.000                  | $24.000                          |
| Standard Class | $18.000                  | $67.000                          |

**PT** — Same Day é o único modal que gera menos lucro do que custa. Standard Class é o único sustentável em escala: menor custo e maior lucro absoluto.

**EN** — Same Day is the only mode that generates less profit than it costs. Standard Class is the only sustainable mode at scale: lowest cost and highest absolute profit.

---

### 4. Existem padrões geográficos e sazonais? / Are there geographic and seasonal patterns?

**PT** — Mapeamento de margem acumulada por país e análise de variação mensal de lucro.

**EN** — Cumulative margin mapped by country and monthly profit variation analyzed.

**PT** — 29 países acumulam margem negativa. Turquia e Nigéria concentram juntas $73k de prejuízo acumulado. Em novembro de 2024, o volume de pedidos apresentou pico consistente com comportamento de Black Friday.

**EN** — 29 countries show negative cumulative margins. Turkey and Nigeria together account for $73k in cumulative losses. In November 2024, order volume showed a peak consistent with Black Friday behavior.

---

## 💡 Principais achados / Key Findings

**PT**

1. **20% é o limite real do desconto** — qualquer pedido acima desse percentual gera prejuízo, independentemente de categoria ou região.
2. **Same Day custa mais do que gera** — $45k de custo contra $33k de lucro. Standard Class inverte essa equação.
3. **Technology tem a maior margem e o maior risco** — lidera em rentabilidade geral, mas é a mais sensível a desconto alto.
4. **29 países operam no prejuízo** — a expansão geográfica da SuperStore não se traduz em lucratividade uniforme.
5. **Black Friday existe nos dados** — pico de novembro identificado, mas sem dados suficientes para isolar o efeito de outras variáveis sazonais.

**EN**

1. **20% is the real discount ceiling** — any order above that percentage generates a loss, regardless of category or region.
2. **Same Day costs more than it earns** — $45k in cost against $33k in profit. Standard Class reverses that equation.
3. **Technology has the highest margin and the highest risk** — leads in overall profitability but is the most sensitive to heavy discounting.
4. **29 countries operate at a loss** — SuperStore's geographic expansion does not translate into uniform profitability.
5. **Black Friday exists in the data** — November peak identified, but insufficient data to isolate the effect from other seasonal variables.

---

## 🛠️ Ferramentas utilizadas / Tools Used

| Ferramenta / Tool   | Etapa / Stage                                                                                              |
|---------------------|------------------------------------------------------------------------------------------------------------|
| SQL + BigQuery      | Limpeza, JOIN, análises e agregações / Cleaning, JOIN, analysis, and aggregations                          |
| Looker Studio       | Dashboard interativo com 4 páginas / Interactive 4-page dashboard                                          |
| Google Sheets       | Organização intermediária de dados / Intermediate data organization                                        |
| Claude (Anthropic)  | Apoio na análise e documentação / Analysis and documentation support                                       |
| Gemini (Google)     | Apoio na análise e documentação / Analysis and documentation support                                       |

---

## 📁 Estrutura do repositório / Repository Structure

```
superstore-sql-analysis/
│
├── queries/
│   ├── 01_data_quality.sql
│   ├── 02_join_tables.sql
│   ├── 03_profitability.sql
│   ├── 04_discount_analysis.sql
│   ├── 05_logistics.sql
│   └── 06_geography_seasonality.sql
│
├── docs/
│   ├── analise_narrativa.docx
│   └── ficha_tecnica.pdf
│
└── README.md
```

---

## 🔗 Links

- 📊 [Dashboard no Looker Studio](https://datastudio.google.com/reporting/a7d94cb2-0069-48f8-926f-de3cf44d1f38) 
- 📄 [Ficha técnica / Technical Sheet](https://docs.google.com/document/d/1rOSxQKdCHq4ITvd_hOoDMyDPI8-gf6CCHmR21_1cMtk/edit?usp=sharing) 

---

## 👩‍💻 Sobre / About

**PT** — Projeto desenvolvido por **Tatiana Terto** durante o bootcamp de análise de dados da Laboratoria, com foco na interseção entre estratégia comercial e dados.

**EN** — Project developed by **Tatiana Terto** during the Laboratoria data analysis bootcamp, focused on the intersection between business strategy and data.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Tatiana%20Terto-blue)](https://www.linkedin.com/in/tatianaterto/) 📧 tatianaalmeida.s@outlook.com
