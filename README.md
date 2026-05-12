![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=Databricks&logoColor=white)
![Apache Spark](https://img.shields.io/badge/Apache%20Spark-F68A1E?style=for-the-badge&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-00ADD8?style=for-the-badge&logo=delta&logoColor=white)
![MkDocs](https://img.shields.io/badge/MkDocs-Material-526EE5?style=for-the-badge)

# Data Platform: E-commerce Lakehouse no Databricks

Este projeto implementa um pipeline de Engenharia de Dados completo para uma operação de e-commerce, construído inteiramente na nuvem utilizando o **Databricks**. O objetivo é demonstrar a implementação ponta a ponta da **Arquitetura Medalhão** (Landing, Bronze, Silver, Gold), governança com Unity Catalog e modelagem dimensional (Kimball) utilizando Apache Spark e Delta Lake.

## Documentação Completa

Toda a arquitetura, modelagem de dados, diagramas ER e a explicação técnica detalhada das transformações foram documentadas e publicadas utilizando o MkDocs.

**[Acesse a Documentação Pública do Projeto Aqui](https://naiiilr.github.io/ProjetoEngDeDados3/)**

---

## Arquitetura de Dados

A infraestrutura é baseada no paradigma de Lakehouse, utilizando processamento distribuído na nuvem e armazenamento otimizado.

```text
┌──────────────┐    ┌──────────────┐    ┌───────────────┐    ┌────────────────┐    ┌─────────────────┐
│ Fontes OLTP  │ ──>│ Landing Zone │ ──>│ Camada Bronze │ ──>│ Camada Silver  │ ──>│  Camada Gold    │
│              │    │              │    │               │    │                │    │                 │
│ Arquivos CSV │    │ Unity Catalog│    │ Tabelas Delta │    │ Tabelas Delta  │    │  Tabelas Delta  │
│ (/data)      │    │ (Volumes)    │    │ (Histórico)   │    │ (Data Quality) │    │  (Star Schema)  │
└──────────────┘    └──────────────┘    └───────────────┘    └────────────────┘    └─────────────────┘
  Upload Manual        Notebook 01         Notebook 02          Notebook 03           Notebook 04
```

## Etapas do Pipeline

O processamento é dividido em estágios principais para garantir a integridade, rastreabilidade e performance analítica dos dados:

* Ingestão (Landing Zone): Leitura de dados brutos de e-commerce (CSV) armazenados em Volumes do Databricks Unity Catalog.

* Camada Bronze: Conversão dos dados brutos para o formato Delta Lake, adicionando colunas de metadados e auditoria (data de ingestão, arquivo de origem) garantindo o histórico imutável.

* Camada Silver: Limpeza e padronização dos dados (Data Quality). Aplicação de regras de negócio, como remoção de duplicatas, tratamento de nulos, padronização de strings para maiúsculo e validação de valores (ex: preços não negativos).

* Camada Gold: Criação de um modelo dimensional (Star Schema) contendo as Tabelas Dimensão (Clientes, Produtos, Tempo) e a Tabela Fato (Vendas), otimizadas para consumo de ferramentas de BI.

## Pré-requisitos (Ambiente Local)
Como o processamento ocorre no Databricks, o ambiente local serve exclusivamente para versionamento de código e geração da documentação estática.

* Python 3.11+

* UV (Gerenciador de pacotes ultra-rápido) — [instalação](https://github.com/astral-sh/uv)

* Conta no Databricks

## Setup e Execução

### 1. Configurar o Ambiente Local (Documentação)

Clone o repositório e instale as dependências locais utilizando o uv:

```bash
uv venv
source .venv/bin/activate
uv sync
```

Para visualizar a documentação técnica localmente:

```bash
uv run mkdocs serve
```

### 2. Execução no Databricks
1. No seu Workspace do Databricks, utilize o menu **Workspace > Repos** para clonar este repositório do GitHub.
2. Acesse o **Catalog**, crie um Schema chamado `landing_dados` e um Volume chamado `arquivos_csv`.
3. Faça o upload manual dos 5 arquivos localizados na pasta `data/` deste repositório para o Volume recém-criado.
4. Execute os notebooks na seguinte ordem lógica:

| # | Notebook | Descrição |
| :--- | :--- | :--- |
| 1 | `001_-_Atividade_Pratica_-_Lakehouse_-_Preparando_ambiente-69fa4f0f2d7b1.dbc` | Cria os databases e carrega os CSVs simulando a extração bruta. |
| 2 | `002_-_Atividade_Pratica_-_Lakehouse_-_Bronze-69fa4f0f35b55.dbc` | Ingestão append-only em formato Delta com metadados de auditoria. |
| 3 | `003_-_Atividade_Pratica_-_Lakehouse_-_Silver-69fa4f0f39c3c.dbc` | Aplicação de Data Quality, tipagem e padronização (Caixa Alta). |
| 4 | `004_-_Atividade_Pratica_-_Lakehouse_-_Gold-69fa4f0f6afff.dbc` | Modelagem Kimball (Star Schema) com Fato Vendas e Dimensões. |
| 5 | `005_-_Atividade_Pratica_-_Lakehouse_-_Destruindo_ambiente-69fa4f0f1bc85.dbc` | Limpeza (DROP) de todos os bancos de dados criados. |

## Estrutura do Projeto

```text
arquitetura-databricks/
├── data/                                # Amostra de dados brutos do e-commerce (CSV)
│   ├── categorias.csv
│   ├── clientes.csv
│   ├── itens_venda.csv
│   ├── produtos.csv
│   └── vendas.csv
├── docs/                                # Markdown da documentação técnica
├── notebooks/                           # Artefactos do pipeline Lakehouse (.dbc)
│   ├── 001_-_Atividade_Pratica_-_Lakehouse_-_Preparando_ambiente-69fa4f0f2d7b1.dbc
│   ├── 002_-_Atividade_Pratica_-_Lakehouse_-_Bronze-69fa4f0f35b55.dbc
│   ├── 003_-_Atividade_Pratica_-_Lakehouse_-_Silver-69fa4f0f39c3c.dbc
│   ├── 004_-_Atividade_Pratica_-_Lakehouse_-_Gold-69fa4f0f6afff.dbc
│   └── 005_-_Atividade_Pratica_-_Lakehouse_-_Destruindo_ambiente-69fa4f0f1bc85.dbc
├── .python-version                      # Versão do Python (3.11)
├── mkdocs.yml                           # Configuração do site de documentação
├── pyproject.toml                       # Gestão de metadados e dependências locais
├── README.md                            # Apresentação e guia do projeto
└── uv.lock                              # Lockfile das dependências
```

## Tecnologias e Conceitos Demonstrados

- **Databricks & Apache Spark:** Motor principal de processamento distribuído via PySpark.
- **Delta Lake:** Formato open-source garantindo transações ACID, Time Travel e alta performance.
- **Unity Catalog / DBFS:** Governança e gestão de arquivos/volumes na nuvem.
- **Engenharia de Dados:** Aplicação rigorosa da Arquitetura Medalhão (Landing, Bronze, Silver, Gold).
- **Modelagem de Dados:** Criação de Data Warehouse Kimball (Star Schema) adaptado para o ecossistema de Big Data.
- **Documentação como Código (DaC):** MkDocs e Markdown integrados ao repositório para geração de site estático.

---

## Links e Referências

- [Databricks - Documentação Oficial](https://docs.databricks.com/pt/index.html)
- [Arquitetura Medalhão (Medallion Architecture)](https://www.databricks.com/br/glossary/medallion-architecture)
- [Apache Spark (PySpark) - API Reference](https://spark.apache.org/docs/latest/api/python/)
- [Delta Lake - Guia Oficial](https://docs.delta.io/latest/index.html)
- [MkDocs Material - Documentação do Tema](https://squidfunk.github.io/mkdocs-material/)
- [UV - Gerenciador de Pacotes Python](https://docs.astral.sh/uv/)
