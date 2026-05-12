# 1. Landing Zone

A **Landing Zone** (ou Camada Raw) é o ponto de entrada dos dados na plataforma. Nesta etapa, o objetivo é provisionar a infraestrutura lógica no Unity Catalog para receber os arquivos exatamente no formato em que foram extraídos do sistema de origem, sem qualquer transformação.

---

# Preparação do Ambiente (Infraestrutura)

Para garantir o isolamento e a governança da Arquitetura Medalhão, os bancos de dados (Schemas) foram estruturados de forma individualizada para cada camada do Data Lakehouse.

Abaixo está o código SQL executado no Databricks para inicializar o ambiente. A cláusula `IF NOT EXISTS` garante a idempotência do processo, permitindo múltiplas execuções sem sobrescrever estruturas existentes.

```sql
-- Criação do Schema para a Landing Zone
CREATE SCHEMA IF NOT EXISTS workspace.landing
COMMENT 'Schema/Database para dados de entrada';

-- Criação do Volume para receber arquivos físicos (CSVs)
CREATE VOLUME IF NOT EXISTS workspace.landing.dados
COMMENT 'Volume para arquivos brutos na landing';

-- Criação dos Schemas das camadas de processamento (Medalhão)
CREATE SCHEMA IF NOT EXISTS workspace.bronze;

CREATE SCHEMA IF NOT EXISTS workspace.silver;

CREATE SCHEMA IF NOT EXISTS workspace.gold;
```

---

# Por que utilizar Volumes?

Ao invés de utilizar o DBFS tradicional, o projeto adota **Volumes do Unity Catalog** (`workspace.landing.dados`).

Volumes representam a abordagem moderna recomendada pela Databricks para armazenamento e governança de arquivos não-tabulares, como:

- CSV
- JSON
- Imagens
- Arquivos de texto

Essa estratégia permite aplicar as mesmas políticas de segurança, permissões e auditoria utilizadas nas tabelas SQL do ambiente Lakehouse.

---

# Ingestão de Dados (Processo)

Após a criação do volume `workspace.landing.dados`, a plataforma está pronta para receber os dados transacionais do sistema de E-commerce.

Nesta simulação, o processo de ingestão ocorre através de **upload manual** para o Volume da Landing Zone. Os arquivos representam uma extração do banco de dados relacional operacional da loja.

---

# Arquivos Esperados na Landing

Os seguintes arquivos devem estar presentes na Landing Zone antes da execução dos pipelines de processamento:

| Arquivo | Formato | Descrição |
|---|---|---|
| `categorias.csv` | CSV | Metadados e descrições das categorias de produtos |
| `clientes.csv` | CSV | Base cadastral de consumidores e status de conta |
| `produtos.csv` | CSV | Catálogo de produtos e respectivos preços |
| `vendas.csv` | CSV | Cabeçalho das transações financeiras (pedidos) |
| `itens_venda.csv` | CSV | Relação detalhada dos produtos vendidos em cada pedido |

---

# Próximos Passos

Com os arquivos armazenados de forma segura no Volume da Landing Zone, o pipeline segue para a próxima etapa da Arquitetura Medalhão.

Na **Camada Bronze**, o Apache Spark realizará:

- Leitura dos arquivos CSV
- Inferência e padronização de schema
- Conversão para o formato Delta Lake
- Inclusão de metadados de rastreabilidade

Continue para: **[Camada Bronze](02_bronze.md)**