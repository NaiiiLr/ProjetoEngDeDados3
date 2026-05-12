# Bem-vindo ao E-commerce Lakehouse 

Bem-vindo à documentação oficial do projeto **E-commerce Lakehouse**. 

Este projeto foi desenvolvido como uma demonstração prática e ponta a ponta de um pipeline moderno de **Engenharia de Dados na Nuvem**, focado em processar, tratar e modelar dados transacionais de um e-commerce simulado.

---

## Objetivo do Projeto

O principal objetivo desta plataforma é transformar dados brutos de transações de vendas em informações analíticas confiáveis, prontas para serem consumidas por times de Business Intelligence (BI) e Ciência de Dados.

Para garantir a escalabilidade, governança e confiabilidade dos dados, toda a infraestrutura foi construída utilizando o paradigma de **Lakehouse**, unindo a flexibilidade dos Data Lakes com a robustez e transações ACID dos Data Warehouses tradicionais.

---

## Stack Tecnológico

A plataforma foi inteiramente desenvolvida utilizando tecnologias líderes de mercado no ecossistema de Big Data:

* **Databricks:** Ambiente unificado de dados e IA, responsável pela orquestração e computação em nuvem.
* **Apache Spark (PySpark):** Motor de processamento distribuído utilizado para as transformações de dados em larga escala.
* **Delta Lake:** Formato de armazenamento open-source que traz confiabilidade aos data lakes (suporte a transações ACID, versionamento de dados e Time Travel).
* **Unity Catalog:** Solução de governança unificada do Databricks, utilizada para o gerenciamento de *Schemas* e *Volumes* (armazenamento de arquivos brutos).
* **MkDocs Material:** Framework de documentação estática (Documentação como Código).

---

## Visão Geral da Arquitetura

O pipeline de dados segue rigorosamente a **Arquitetura Medalhão** (Medallion Architecture), que organiza os dados em camadas lógicas, melhorando progressivamente a estrutura e a qualidade do dado à medida que ele flui pela plataforma:

1. **Landing Zone (Volumes):** Recebe os arquivos brutos (`.csv`) diretamente dos sistemas de origem.
2. **Camada Bronze:** Os dados são convertidos para o formato Delta Lake e recebem colunas de auditoria (rastreabilidade), mantendo o histórico exato do estado bruto.
3. **Camada Silver:** A camada de "Data Quality". Aqui, os dados são higienizados, padronizados (ex: caixa alta), deduplicados e validados contra regras de negócio (ex: impossibilidade de preços negativos).
4. **Camada Gold:** A camada de negócios. Os dados são modelados utilizando as técnicas de Ralph Kimball (Star Schema), criando Fatos e Dimensões altamente otimizadas para consultas analíticas e painéis visuais.

---

## Como navegar nesta documentação?

Utilize o menu de navegação lateral para explorar os detalhes técnicos da plataforma:

* Em **Arquitetura**, você encontrará o desenho conceitual e técnico do pipeline.
* Em **Dicionário de Dados**, estão listadas as definições das tabelas finais (Camada Gold) prontas para o consumo.
* Na seção **Pipeline Medalhão**, você pode acompanhar a explicação técnica detalhada, passo a passo, do que o código PySpark realiza em cada uma das camadas do nosso Lakehouse.