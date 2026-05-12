# Dicionário de Dados

Este dicionário descreve as entidades do ecossistema de e-commerce processadas no Data Lake. A estrutura documenta a evolução dos dados desde a sua ingestão bruta na camada **Bronze** até a modelagem dimensional final na camada **Gold**, garantindo a linhagem e a qualidade da informação através do Delta Lake.

---

# Estrutura das Tabelas: Camada Bronze (Estado Bruto)

Nesta fase, as tabelas refletem o esquema original das fontes de dados, acrescidas de metadados de auditoria para rastreabilidade.

---

## 1. Tabela: `categorias`

Agrupamento lógico dos produtos.

| Campo | Tipo | Descrição |
|---|---|---|
| id_categoria | Integer | Identificador único da categoria |
| nome_categoria | String | Nome descritivo da categoria |
| descricao | String | Detalhes sobre os produtos da categoria |

---

## 2. Tabela: `produtos`

Itens comercializados e níveis de inventário.

| Campo | Tipo | Descrição |
|---|---|---|
| id_produto | Integer | Identificador único do produto |
| nome | String | Nome comercial do produto |
| id_categoria | Integer | Chave estrangeira para categorias |
| preco | Decimal | Valor unitário de venda |
| estoque | Integer | Quantidade disponível |

---

## 3. Tabela: `clientes`

Informações de usuários cadastrados.

| Campo | Tipo | Descrição |
|---|---|---|
| id_cliente | Integer | Identificador único do cliente |
| nome | String | Nome completo |
| estado | String | UF de residência |
| status_conta | String | Situação do cadastro (ex: Ativo) |

---

## 4. Tabela: `vendas`

Registros transacionais dos pedidos.

| Campo | Tipo | Descrição |
|---|---|---|
| id_venda | Integer | Identificador único da venda |
| id_cliente | Integer | Cliente responsável pela compra |
| data_venda | Date | Data da venda |
| valor_total | Decimal | Valor total do pedido |

---

## 5. Tabela: `itens_venda`

Itens pertencentes aos pedidos.

| Campo | Tipo | Descrição |
|---|---|---|
| id_item | Integer | Identificador único do item |
| id_venda | Integer | Chave estrangeira para vendas |
| id_produto | Integer | Chave estrangeira para produtos |
| quantidade | Integer | Quantidade vendida |
| preco_unitario | Decimal | Valor unitário do item |

---

# Estrutura das Tabelas: Camada Gold (Star Schema)

A camada Gold reorganiza os dados em um modelo dimensional (Fato e Dimensões) para otimizar a performance analítica e facilitar o consumo por ferramentas de BI.

---

# Tabelas de Dimensão

## dim_clientes

Contém os atributos cadastrais higienizados e padronizados (caixa alta).

| Campo | Tipo | Descrição |
|---|---|---|
| ID_CLIENTE | Integer | Chave primária do cliente |
| NOME | String | Nome completo padronizado |
| ESTADO | String | UF de residência normalizada |
| STATUS_CONTA | String | Situação atual do cadastro |

---

## dim_produtos

Dimensão desnormalizada que une as informações de produtos e categorias.

| Campo | Tipo | Descrição |
|---|---|---|
| ID_PRODUTO | Integer | Identificador único do produto |
| NOME | String | Nome comercial |
| PRECO | Double | Preço validado e higienizado |
| NOME_CATEGORIA | String | Categoria vinculada |
| DESCRICAO | String | Descritivo da categoria |

---

## dim_tempo

Dimensão temporal derivada para análises de sazonalidade.

| Campo | Tipo | Descrição |
|---|---|---|
| DATA_VENDA | Date | Chave primária da dimensão tempo |
| ANO | Integer | Ano da venda |
| MES | Integer | Mês (1-12) |
| TRIMESTRE | Integer | Trimestre do ano |
| DIA_SEMANA | String | Nome do dia da semana |

---

# Tabela de Fato

## fato_vendas

Consolida as transações e métricas de negócio.

| Campo | Tipo | Descrição |
|---|---|---|
| ID_ITEM | Integer | Chave primária do fato |
| ID_VENDA | Integer | Identificador do pedido |
| ID_CLIENTE | Integer | Chave estrangeira para `dim_clientes` |
| ID_PRODUTO | Integer | Chave estrangeira para `dim_produtos` |
| DATA_VENDA | Date | Chave estrangeira para `dim_tempo` |
| QUANTIDADE | Integer | Volume vendido |
| VALOR_TOTAL_ITEM | Double | Métrica calculada (Quantidade * Preço Unitário) |

---

# Notas de Implementação

- **Persistência**: Os dados são gerenciados via **Unity Catalog** no Databricks, utilizando **Volumes** para a Landing Zone e tabelas gerenciadas para as camadas subsequentes.
- **Versionamento**: O uso do Delta Lake permite o histórico de versões através do recurso de **Time Travel**, garantindo auditoria completa das alterações.
- **Integridade**: As transformações garantem que dados inconsistentes (ex: preços negativos ou quantidades nulas) sejam tratados antes de atingirem a camada Gold.

---

# Próximos Passos

Para compreender como estas tabelas são processadas através das camadas do Data Lakehouse, consulte a página detalhada da **[Arquitetura do Projeto](arquitetura.md)**.