# 3. Camada Silver (Cleansed Data)

A Camada Silver representa a zona central de higienização e padronização do Lakehouse. Nesta etapa, os dados provenientes da Camada Bronze passam por processos de limpeza, validação e enriquecimento, tornando-se uma fonte confiável para consumo analítico e modelagem dimensional.

O principal objetivo desta camada é construir uma **versão única da verdade** (*Single Source of Truth*), eliminando inconsistências estruturais e garantindo qualidade dos dados.

---

# 1. Leitura dos Dados e Padronização Estrutural

O processamento inicia com a leitura das tabelas Delta presentes no schema `bronze`.

A primeira transformação aplicada consiste na padronização semântica dos nomes das colunas. Todas as colunas são convertidas para letras maiúsculas (`UPPER`) para evitar problemas relacionados a *case-sensitivity* em consultas futuras.

```python
# Função auxiliar para padronização de nomenclatura
def _apply_name_rules(colname: str) -> str:
    """
    Regras de renome:
    Para este projeto, todas as colunas serão padronizadas em MAIÚSCULO.
    """
    return colname.upper()

# Aplicação dinâmica das regras
new_cols = [_apply_name_rules(c) for c in df.columns]

df = df.toDF(*new_cols)
```

---

# 2. Aplicação de Regras de Data Quality (DQ)

Após a padronização estrutural, os dados passam por um conjunto de validações de qualidade (*Data Quality Rules*).

As regras implementadas garantem:

- Integridade lógica
- Padronização
- Eliminação de inconsistências
- Confiabilidade analítica

---

## Regras Aplicadas

| Entidade | Regra de Qualidade |
|---|---|
| Todas | Remoção de registros duplicados com `dropDuplicates()` |
| Clientes | Padronização do campo `ESTADO` com `TRIM` e `UPPER` |
| Produtos | Exclusão de produtos com `PRECO` ou `ESTOQUE` negativos |
| Vendas | Remoção de pedidos com `VALOR_TOTAL` negativo |
| Itens de Venda | Garantia de `QUANTIDADE > 0` |

---

## Exemplo de tratamento aplicado

```python
# Remoção de registros duplicados
df = df.dropDuplicates(["ID_CLIENTE"])

# Padronização do estado
df = df.withColumn(
    "ESTADO",
    F.upper(F.trim(F.col("ESTADO")))
)
```

---

# 3. Gestão de Metadados e Auditoria

Além da limpeza e validação, a Camada Silver mantém o processo de rastreabilidade dos dados (*Data Lineage*).

Os metadados técnicos da camada Bronze são substituídos por novos marcadores específicos da Silver.

---

## Alterações de auditoria

| Ação | Objetivo |
|---|---|
| Remoção de `DATA_HORA_BRONZE` | Eliminar metadados transitórios |
| Remoção de `NOME_ARQUIVO` | Evitar redundância técnica |
| Inclusão de `NOME_ARQUIVO_ORIGEM` | Registrar origem lógica da tabela |
| Inclusão de `DATA_HORA_SILVER` | Registrar momento da higienização |

---

## Exemplo de implementação

```python
# Remoção segura de colunas técnicas
df = _safe_drop(df, ["DATA_HORA_BRONZE", "NOME_ARQUIVO"])

# Inclusão de novos metadados
df = df.withColumn(
    "NOME_ARQUIVO_ORIGEM",
    F.lit("bronze.clientes")
)

df = df.withColumn(
    "DATA_HORA_SILVER",
    F.current_timestamp()
)
```

---

# 4. Escrita no Formato Delta (Carga)

Após os tratamentos de qualidade e auditoria, os DataFrames são persistidos novamente no formato Delta Lake, agora no schema `workspace.silver`.

O pipeline utiliza o modo `overwrite`, garantindo idempotência e substituindo versões antigas pelos dados recém-processados.

```python
# Salvando como Managed Table na camada Silver
(
    df.write
        .format("delta")
        .mode("overwrite")
        .saveAsTable(dest_fqn)
)
```

---

# Benefícios da Camada Silver

A etapa Silver desempenha papel fundamental na confiabilidade do ambiente analítico.

## Principais ganhos

- Padronização estrutural
- Eliminação de inconsistências
- Governança dos dados
- Rastreabilidade completa
- Preparação para modelagem analítica
- Garantia de integridade lógica

Com isso, as tabelas tornam-se adequadas para construção de modelos dimensionais e consumo corporativo.

---

# Próximos Passos: Modelagem Dimensional

Após os processos de limpeza e validação, os dados seguem para a Camada Gold.

Nesta última etapa do pipeline, as entidades serão reorganizadas em um modelo dimensional (*Star Schema*), utilizando joins e métricas analíticas voltadas para:

- Dashboards
- KPIs
- Ferramentas BI
- Consultas OLAP
- Analytics avançado

Continue para: **[Camada Gold](03_gold.md)**