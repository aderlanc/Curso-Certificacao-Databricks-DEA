# Domínio 3 — Data Transformation and Modeling (22% · ~10 questões)

> É o domínio de maior peso. O curso cobre medalhão, arrays, UDFs, Delta e pipelines declarativos (`05`, `06`, `08`).
> Joins, agregações, deduplicação e parâmetros de tuning são completados por este resumo.

**Objetivos do guia:**
1. Limpeza: ler Bronze com PySpark/SQL, tratar nulos, padronizar tipos e gravar Silver.
2. Combinar DataFrames: inner, left, **broadcast**, múltiplas chaves, **cross join**, **union** e **union all**.
3. Manipular colunas, linhas e estrutura: adicionar, remover, dividir, renomear, filtrar, **explode**.
4. **Deduplicação** e **agregações** (count, `approx_count_distinct`, média, summary).
5. Parâmetros básicos de tuning: `spark.sql.shuffle.partitions`, `spark.default.parallelism`, memória de executor/driver, `spark.sql.autoBroadcastJoinThreshold`.
6. Objetos da camada Gold no UC: **materialized views, views, streaming tables e tabelas**.
7. Regras de **qualidade de dados** para Silver e Gold confiáveis.

---

## 1. Bronze → Silver: limpeza

```python
from pyspark.sql import functions as F

bronze = spark.read.table("vendas.bronze.pedidos")

silver = (bronze
    .withColumn("valor", F.col("valor").cast("decimal(10,2)"))          # padronizar tipo
    .withColumn("data_pedido", F.to_date("data_pedido", "yyyy-MM-dd"))
    .withColumn("email", F.lower(F.trim("email")))
    .fillna({"canal": "desconhecido"})                                   # nulo → valor padrão
    .dropna(subset=["pedido_id", "cliente_id"])                          # remove linhas sem chave
    .withColumn("uf", F.split("cidade_uf", "-").getItem(1))              # dividir coluna
    .withColumnRenamed("cli_nome", "cliente_nome")
    .drop("cidade_uf", "_rescued_data")
    .filter(F.col("valor") > 0))

silver.write.mode("overwrite").saveAsTable("vendas.silver.pedidos")
```

Equivalentes em SQL: `CAST`, `TRY_CAST` (retorna NULL em vez de erro), `COALESCE`, `NULLIF`, `TRIM`, `SPLIT`, `WHERE`.

## 2. Joins e unions

| Operação | PySpark | SQL | Observação |
|---|---|---|---|
| Inner | `a.join(b, "id")` | `JOIN ... ON` | Só linhas com correspondência |
| Left | `a.join(b, "id", "left")` | `LEFT JOIN` | Mantém tudo de `a` |
| Múltiplas chaves | `a.join(b, ["id", "data"])` | `ON a.id=b.id AND a.data=b.data` | Lista de colunas evita coluna duplicada |
| Left semi / anti | `"left_semi"`, `"left_anti"` | `LEFT SEMI JOIN`, `LEFT ANTI JOIN` | "existe em b" / "não existe em b" |
| Cross | `a.crossJoin(b)` | `CROSS JOIN` | Produto cartesiano: linhas(a) × linhas(b) |
| **Broadcast** | `a.join(F.broadcast(b), "id")` | `SELECT /*+ BROADCAST(b) */ ...` | Copia a tabela **pequena** para todos os executores e **evita shuffle** |

**Unions — pegadinha clássica:**
| | Remove duplicatas? | Casa colunas por |
|---|---|---|
| SQL `UNION` | **Sim** | Posição |
| SQL `UNION ALL` | Não | Posição |
| PySpark `df1.union(df2)` | **Não** (equivale a UNION ALL) | **Posição** |
| PySpark `df1.unionByName(df2, allowMissingColumns=True)` | Não | **Nome** |

> Para remover duplicatas no PySpark depois de `union`, encadeie `.distinct()`.

## 3. Explode e dados aninhados

```python
pedidos.select("pedido_id", F.explode("itens").alias("item")) \
       .select("pedido_id", "item.sku", "item.qtd")
```
- `explode` descarta linhas com array vazio/nulo; `explode_outer` mantém.
- Funções de ordem superior em SQL: `transform`, `filter`, `exists`, `aggregate` (aula `06/08`).

## 4. Deduplicação e agregações

```python
# Duplicatas exatas / por chave
df.distinct()
df.dropDuplicates(["pedido_id"])

# Ficar com o registro MAIS RECENTE por chave
from pyspark.sql.window import Window
w = Window.partitionBy("cliente_id").orderBy(F.col("atualizado_em").desc())
ultimo = df.withColumn("rn", F.row_number().over(w)).filter("rn = 1").drop("rn")

# Agregações
(df.groupBy("data_pedido")
   .agg(F.sum("valor").alias("receita"),
        F.count("*").alias("linhas"),
        F.countDistinct("pedido_id").alias("pedidos_unicos"),
        F.approx_count_distinct("cliente_id").alias("clientes_aprox"),  # rápido, com pequena margem de erro
        F.avg("valor").alias("ticket_medio")))

df.describe("valor").show()   # count, mean, stddev, min, max
df.summary().show()           # + percentis 25%, 50%, 75%
```

- `count("*")` conta linhas; `count("col")` ignora nulos; `countDistinct` é exato; `approx_count_distinct` usa HyperLogLog e é bem mais barato em grandes volumes.
- Em SQL, deduplicar na escrita: `MERGE INTO` com `WHEN NOT MATCHED THEN INSERT`.

## 5. Parâmetros de tuning

| Parâmetro | Padrão | Para que serve |
|---|---|---|
| `spark.sql.shuffle.partitions` | 200 (no Databricks com AQE, ajustado automaticamente) | Nº de partições depois de um shuffle (`groupBy`, `join`). Poucas → spill; muitas → tasks minúsculas |
| `spark.default.parallelism` | Nº total de cores | Nº de partições padrão para **RDDs** (não DataFrames/SQL) |
| `spark.sql.autoBroadcastJoinThreshold` | 10 MB | Tabelas menores que isso são broadcast automaticamente. `-1` desliga |
| `spark.executor.memory` / `spark.driver.memory` | Pelo tipo de nó | Memória dos processos. Definida na **configuração do cluster**, não em tempo de execução |
| `spark.sql.adaptive.enabled` | `true` | AQE: ajusta partições, joins e skew durante a execução |

```python
spark.conf.set("spark.sql.shuffle.partitions", 64)
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 50 * 1024 * 1024)
```
> Mudou um parâmetro? **Meça de novo** (tempo da query, Spark UI) para confirmar o ganho. Em compute serverless, a maioria desses parâmetros é gerenciada pela Databricks.

## 6. Objetos da camada Gold

| Objeto | Armazena dados? | Atualização | Quando usar |
|---|---|---|---|
| **View** | Não | Recalcula a cada consulta | Lógica leve, sempre atual, sem custo de armazenamento |
| **Materialized view** | Sim (pré-calculada) | `REFRESH` manual, agendada ou no pipeline; incremental quando possível | Agregações pesadas consultadas muitas vezes por BI |
| **Streaming table** | Sim | Processa só dados novos de uma fonte **append-only** | Ingestão incremental, Bronze/Silver |
| **Tabela Delta** | Sim | Você controla (INSERT, MERGE, overwrite) | Controle total da carga |

```sql
CREATE MATERIALIZED VIEW vendas.gold.receita_diaria
SCHEDULE EVERY 1 HOUR
AS SELECT data_pedido, SUM(valor) AS receita FROM vendas.silver.pedidos GROUP BY data_pedido;

REFRESH MATERIALIZED VIEW vendas.gold.receita_diaria;

CREATE OR REFRESH STREAMING TABLE vendas.bronze.eventos
AS SELECT * FROM STREAM read_files('/Volumes/vendas/raw/eventos/', format => 'json');
```

Lakeflow Spark Declarative Pipelines (ex-DLT): você **declara** as tabelas e o motor resolve a ordem (DAG), a infraestrutura, retries e checkpoints.
```python
from pyspark import pipelines as dp     # código antigo: import dlt

@dp.materialized_view
def receita_diaria():
    return spark.read.table("silver_pedidos").groupBy("data_pedido").agg(F.sum("valor"))
```

## 7. Qualidade de dados

**Expectations** (pipelines declarativos):
| Ação | SQL | Python | Efeito |
|---|---|---|---|
| Avisar (padrão) | `CONSTRAINT c EXPECT (cond)` | `@dp.expect` | Mantém a linha, registra métrica |
| Descartar | `... ON VIOLATION DROP ROW` | `@dp.expect_or_drop` | Remove a linha inválida |
| Falhar | `... ON VIOLATION FAIL UPDATE` | `@dp.expect_or_fail` | Para a atualização do fluxo |

Vários de uma vez: `@dp.expect_all({...})`, `@dp.expect_all_or_drop({...})`.

**Constraints Delta** (fora de pipelines): `NOT NULL` e `CHECK` fazem a **transação inteira falhar** se alguma linha violar (aula `05/8`).
```sql
ALTER TABLE vendas.silver.pedidos ADD CONSTRAINT valor_positivo CHECK (valor > 0);
```

Padrão **quarentena**: gravar linhas inválidas numa tabela separada para análise, em vez de perdê-las.

## 8. DDL/DML que sempre aparece

```sql
CREATE OR REPLACE TABLE t (id STRING, inicio DATE, nota FLOAT);   -- cria vazia, existindo ou não
CREATE TABLE IF NOT EXISTS t (...);                               -- só cria se não existir
CREATE TABLE t2 AS SELECT * FROM t;                               -- CTAS: schema vem da query
INSERT INTO t VALUES ('a1', '2026-01-01', 9.4);                   -- append
INSERT OVERWRITE t SELECT ...;                                    -- substitui os dados
MERGE INTO alvo a USING novos n ON a.id = n.id
  WHEN MATCHED THEN UPDATE SET *
  WHEN NOT MATCHED THEN INSERT *;                                 -- upsert
```

---

## ✅ Checklist do domínio
- [ ] Sei limpar nulos e converter tipos em PySpark e SQL
- [ ] Sei quando usar broadcast join e que ele evita shuffle
- [ ] Sei a diferença entre `UNION`, `UNION ALL`, `union()` e `unionByName()`
- [ ] Sei deduplicar por chave e pegar o registro mais recente
- [ ] Sei `countDistinct` x `approx_count_distinct`
- [ ] Sei o efeito de `shuffle.partitions` e `autoBroadcastJoinThreshold`
- [ ] Sei escolher entre view, materialized view, streaming table e tabela
- [ ] Sei as 3 ações das expectations

## 📚 Fontes oficiais
- [Joins no Databricks](https://learn.microsoft.com/azure/databricks/transform/join)
- [Materialized views](https://learn.microsoft.com/azure/databricks/views/materialized)
- [Streaming tables](https://learn.microsoft.com/azure/databricks/tables/streaming)
- [Expectations](https://learn.microsoft.com/azure/databricks/ldp/expectations)
- [MERGE INTO](https://learn.microsoft.com/azure/databricks/sql/language-manual/delta-merge-into)
