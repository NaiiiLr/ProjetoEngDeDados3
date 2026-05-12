# 2. Camada Bronze (Raw Data)

A Camada Bronze representa a primeira fase de persistência estruturada do Lakehouse. O principal objetivo desta etapa é extrair os arquivos brutos (CSV) armazenados na Landing Zone, convertê-los para o formato de alta performance **Delta Lake** e adicionar metadados de auditoria, preservando total fidelidade aos dados de origem.

Nesta camada, não são aplicadas regras de negócio, filtros de qualidade ou transformações analíticas.

---

# 1. Leitura dos Dados (Extração)

O pipeline inicia explorando o Volume `workspace.landing.dados`.

Utilizando PySpark, os cinco arquivos CSV provenientes do sistema transacional são carregados em DataFrames Spark. Durante a leitura, são ativadas:

- Inferência automática de schema (`inferSchema`)
- Identificação do cabeçalho (`header`)

```python
caminho_landing = '/Volumes/workspace/landing/dados'

# Leitura dos ficheiros CSV para DataFrames Spark
df_categorias  = spark.read.option("inferSchema", "true") \
    .option("header", "true") \
    .csv(f"{caminho_landing}/categorias.csv")

df_clientes = spark.read.option("inferSchema", "true") \
    .option("header", "true") \
    .csv(f"{caminho_landing}/clientes.csv")

df_itens_venda = spark.read.option("inferSchema", "true") \
    .option("header", "true") \
    .csv(f"{caminho_landing}/itens_venda.csv")

df_produtos = spark.read.option("inferSchema", "true") \
    .option("header", "true") \
    .csv(f"{caminho_landing}/produtos.csv")

df_vendas = spark.read.option("inferSchema", "true") \
    .option("header", "true") \
    .csv(f"{caminho_landing}/vendas.csv")
```

---

# 2. Injeção de Metadados (Auditoria e Linhagem)

Para garantir rastreabilidade e governança dos dados (*Data Lineage*), os DataFrames são enriquecidos com colunas técnicas de auditoria antes da persistência.

## Metadados adicionados

| Coluna | Descrição |
|---|---|
| `data_hora_bronze` | Timestamp exato em que o registro foi processado |
| `nome_arquivo` | Nome físico do arquivo de origem |

```python
from pyspark.sql.functions import current_timestamp, lit

# Adição das colunas de rastreabilidade
df_clientes = df_clientes \
    .withColumn("data_hora_bronze", current_timestamp()) \
    .withColumn("nome_arquivo", lit("clientes.csv"))

# O mesmo processo é repetido para os restantes DataFrames
```

---

# 3. Escrita no Formato Delta (Carga)

Após a leitura e enriquecimento dos metadados, os DataFrames são persistidos como **Managed Tables** no schema `workspace.bronze`, utilizando o formato Delta Lake.

O Delta Lake adiciona benefícios importantes já na camada bruta:

- Compressão otimizada via Parquet
- Versionamento dos dados
- Suporte a Time Travel
- Melhor performance de leitura
- Controle transacional ACID

```python
# Escrita dos dados como Tabelas Delta no schema Bronze
df_clientes.write \
    .format('delta') \
    .mode("overwrite") \
    .saveAsTable("bronze.clientes")
```

---

# 4. Validação da Camada

Após a execução do pipeline PySpark, comandos SQL nativos do Databricks são utilizados para validar a estrutura criada na Camada Bronze.

## Validações realizadas

| Comando | Objetivo |
|---|---|
| `SHOW TABLES IN bronze` | Confirmar a criação das tabelas |
| `DESCRIBE EXTENDED bronze.clientes` | Verificar propriedades técnicas e schema |

A validação confirma:

- Tipo da tabela (`MANAGED`)
- Formato de armazenamento (`delta`)
- Schema inferido
- Presença dos metadados técnicos de auditoria

---

# Próximos Passos

Com os dados devidamente persistidos no formato Delta Lake, o pipeline avança para a Camada Silver.

Nesta próxima etapa serão aplicados:

- Tratamentos de qualidade
- Limpeza de inconsistências
- Padronização de dados
- Regras de negócio
- Validações analíticas

Continue para: **[Camada Silver](03_silver.md)**