# Arquitetura do Lakehouse

Esta seção detalha o design lógico e a implementação técnica da plataforma de dados, fundamentada na **Arquitetura Medalhão** e no paradigma de **Lakehouse**.

## Fluxo de Dados Conceitual

O processamento foi desenhado para garantir que o dado bruto, uma vez ingerido, passe por processos de limpeza e enriquecimento até estar pronto para o consumo analítico.



### 1. Ingestão e Landing (Unity Catalog)
Os dados brutos chegam via arquivos CSV e são armazenados em **Volumes do Unity Catalog**. Esta escolha permite o gerenciamento de arquivos não estruturados com o mesmo nível de segurança das tabelas SQL.

### 2. Camada Bronze (Histórico Bruto)
Nesta fase, os arquivos são convertidos para o formato **Delta Lake**.
* **Objetivo:** Manter a fidelidade total aos dados de origem.
* **Metadados:** São incluídas colunas de auditoria para rastrear o arquivo de origem e o timestamp da carga.

### 3. Camada Silver (Verdade Única)
Aqui aplicamos as regras de **Data Quality** observadas nos notebooks de processamento:
* **Padronização:** Conversão de nomes de colunas e strings para CAIXA ALTA.
* **Saneamento:** Remoção de duplicatas e filtragem de registros inconsistentes (ex: vendas com valores nulos ou negativos).
* **Estrutura:** Tabelas normalizadas e prontas para relacionamentos.

### 4. Camada Gold (Modelo Dimensional)
A última etapa transforma os dados em um **Star Schema (Kimball)**, focado em performance de consulta:
* **Tabelas de Dimensão:** `dim_clientes`, `dim_produtos` e `dim_tempo`.
* **Tabela de Fato:** `fato_vendas`, consolidando as métricas de negócio.

## Decisões de Design

* **Transações ACID:** O uso do Delta Lake foi essencial para garantir que falhas durante o processamento de uma camada não corrompam os dados das camadas subsequentes.
* **Governança Unificada:** A utilização do Unity Catalog centraliza a gestão de permissões, eliminando a necessidade de gerenciar caminhos de diretórios complexos (DBFS).