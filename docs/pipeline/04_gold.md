# 4. Camada Gold (Modelo Dimensional)

A Camada Gold representa o estágio final e refinado do Lakehouse. Nesta etapa, os dados relacionais e tratados provenientes da Camada Silver são reorganizados em um modelo dimensional (*Star Schema*), seguindo os princípios propostos por Ralph Kimball.

O objetivo principal desta camada é disponibilizar estruturas altamente otimizadas para:

- Ferramentas de Business Intelligence (BI)
- Dashboards analíticos
- Consultas OLAP
- Indicadores estratégicos
- Modelos de Machine Learning

---

# 1. Garantia de Idempotência (Limpeza)

Para garantir que o pipeline possa ser executado múltiplas vezes sem problemas de duplicidade ou conflito estrutural, o processo inicia removendo as tabelas finais caso elas já existam.

```sql
DROP TABLE IF EXISTS gold.dim_clientes;

DROP TABLE IF EXISTS gold.dim_produtos;

DROP TABLE IF EXISTS gold.dim_tempo;

DROP TABLE IF EXISTS gold.fato_vendas;
```

---

# 2. Criação das Tabelas de Dimensão

As tabelas de dimensão fornecem o contexto analítico para os dados transacionais.

Elas respondem perguntas como:

- Quem comprou?
- O que foi vendido?
- Quando ocorreu a venda?

---

## Dimensão Clientes (`dim_clientes`)

A dimensão de clientes concentra os atributos cadastrais relevantes para análise, preservando os metadados de auditoria herdados da Camada Silver.

### Principais atributos

| Campo | Descrição |
|---|---|
| `ID_CLIENTE` | Identificador do cliente |
| `NOME` | Nome padronizado |
| `ESTADO` | Unidade federativa |
| `STATUS_CONTA` | Situação cadastral |

---

## Dimensão Produtos (`dim_produtos`)

A dimensão de produtos é construída utilizando o conceito de desnormalização.

Para melhorar a performance analítica e evitar múltiplos joins em dashboards, é realizado um `LEFT JOIN` entre produtos e categorias, criando uma única dimensão enriquecida.

```python
# Desnormalizando Produtos e Categorias
df_dim_produtos = df_produtos.join(
    df_categorias,
    "ID_CATEGORIA",
    "left"
).select(
    "ID_PRODUTO",
    "NOME",
    "PRECO",
    "NOME_CATEGORIA",
    "DESCRICAO"
)

df_dim_produtos.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable("gold.dim_produtos")
```

### Benefícios da desnormalização

- Melhor desempenho em ferramentas BI
- Menor complexidade nas consultas
- Simplificação da modelagem analítica
- Redução de joins em dashboards

---

## Dimensão Tempo (`dim_tempo`)

A dimensão temporal é derivada a partir das datas únicas presentes nas vendas.

Com PySpark, novos atributos analíticos são gerados dinamicamente para permitir análises temporais avançadas.

```python
# Derivação de atributos temporais
df_dim_tempo = df_vendas.select("DATA_VENDA").distinct() \
    .withColumn("ANO", F.year("DATA_VENDA")) \
    .withColumn("MES", F.month("DATA_VENDA")) \
    .withColumn("TRIMESTRE", F.quarter("DATA_VENDA")) \
    .withColumn("DIA_SEMANA", F.date_format("DATA_VENDA", "EEEE"))
```

### Atributos derivados

| Campo | Objetivo Analítico |
|---|---|
| `ANO` | Comparações anuais |
| `MES` | Sazonalidade mensal |
| `TRIMESTRE` | Indicadores trimestrais |
| `DIA_SEMANA` | Análise de comportamento temporal |

Essa estrutura permite análises como:

- Crescimento Year-over-Year (YoY)
- Comparativos mensais
- Tendências sazonais
- Identificação de períodos de maior venda

---

# 3. Criação da Tabela Fato (`fato_vendas`)

A tabela fato representa o núcleo quantitativo do modelo dimensional.

Ela registra o evento de negócio principal do projeto:

> Um cliente comprou um produto em uma determinada data.

A construção da fato é realizada através de um `INNER JOIN` entre:

- Cabeçalho das vendas (`vendas`)
- Itens detalhados (`itens_venda`)

Além das chaves estrangeiras, também é calculada uma métrica analítica fundamental:

- `VALOR_TOTAL_ITEM`

```python
# Junção de Fato e Cálculo de Métrica
df_fato_vendas = df_itens_venda.join(
    df_vendas,
    "ID_VENDA",
    "inner"
).select(
    "ID_ITEM",
    "ID_VENDA",
    "ID_CLIENTE",
    "ID_PRODUTO",
    "DATA_VENDA",
    "QUANTIDADE",
    "PRECO_UNITARIO",
    (
        F.col("QUANTIDADE") * 
        F.col("PRECO_UNITARIO")
    ).alias("VALOR_TOTAL_ITEM")
)

df_fato_vendas.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable("gold.fato_vendas")
```

---

# Estrutura Final do Modelo Dimensional

O modelo final é composto por:

| Tipo | Tabela |
|---|---|
| Dimensão | `gold.dim_clientes` |
| Dimensão | `gold.dim_produtos` |
| Dimensão | `gold.dim_tempo` |
| Fato | `gold.fato_vendas` |

Essa arquitetura segue o padrão Star Schema:

- Dimensões fornecem contexto
- Fato concentra métricas quantitativas
- Consultas analíticas tornam-se mais rápidas e intuitivas

---

# Conclusão do Pipeline

Com a conclusão da Camada Gold, o ciclo completo da Arquitetura Medalhão foi implementado com sucesso.

Ao longo do pipeline, os dados evoluíram progressivamente:

| Camada | Objetivo |
|---|---|
| Landing | Recepção dos arquivos brutos |
| Bronze | Persistência e auditoria dos dados |
| Silver | Limpeza, padronização e qualidade |
| Gold | Modelagem analítica dimensional |

A solução construída demonstra conceitos modernos de engenharia de dados utilizando:

- Databricks
- Unity Catalog
- Apache Spark
- Delta Lake
- Arquitetura Medalhão
- Modelagem Dimensional

Além disso, o pipeline garante:

- Governança dos dados
- Rastreabilidade completa
- Versionamento via Delta Lake
- Escalabilidade analítica
- Facilidade de integração com ferramentas BI

O resultado final é um ambiente analítico robusto, pronto para alimentar dashboards estratégicos, análises avançadas e futuras aplicações de Machine Learning.