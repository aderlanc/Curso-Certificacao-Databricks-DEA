# Questões — Domínio 3: Data Transformation and Modeling

> Questões **inéditas** escritas no estilo da prova, a partir dos objetivos do guia oficial (ago/2026). Não são questões vazadas.

---

### 1. A tabela `faturamento` tem as colunas `fatura_id`, `cliente_id`, `data`, `valor`. Uma mesma fatura pode aparecer em várias linhas (uma por item). Qual código gera, por data, a receita total e o número de faturas únicas?

A) `df.groupBy("data").agg(F.sum("valor"), F.count("fatura_id"))`
B) `df.groupBy("data").agg(F.sum("valor"), F.countDistinct("fatura_id"))`
C) `df.groupBy("data").agg(F.sum("valor"), F.countDistinct("cliente_id"))`
D) `df.agg(F.sum("valor"), F.countDistinct("fatura_id"))`

<details><summary>Resposta</summary>

**B.** `count` contaria linhas (itens), não faturas. C conta clientes. D não agrupa por data.
</details>

---

### 2. Um engenheiro faz join de uma tabela de 2 TB de vendas com uma tabela de 5 MB de lojas. O Spark UI mostra um grande shuffle nas duas tabelas. Qual mudança reduz o shuffle?

A) `vendas.join(F.broadcast(lojas), "loja_id")`
B) `vendas.crossJoin(lojas)`
C) Aumentar `spark.sql.shuffle.partitions` para 2000
D) Converter `lojas` em view

<details><summary>Resposta</summary>

**A.** O broadcast envia a tabela pequena a todos os executores, e a grande não precisa ser redistribuída.
</details>

---

### 3. `df1` e `df2` têm as mesmas colunas, mas em **ordem diferente**. Qual operação combina as linhas corretamente?

A) `df1.union(df2)`
B) `df1.unionByName(df2)`
C) `df1.join(df2)`
D) `df1.crossJoin(df2)`

<details><summary>Resposta</summary>

**B.** `union` casa colunas por posição e misturaria os valores. `unionByName` casa pelo nome.
</details>

---

### 4. Qual afirmação sobre `df1.union(df2)` no PySpark está correta?

A) Remove duplicatas, como o `UNION` do SQL
B) Mantém duplicatas, equivalente a `UNION ALL`
C) Exige que os DataFrames tenham o mesmo número de linhas
D) Faz join pela primeira coluna

<details><summary>Resposta</summary>

**B.** Para remover duplicatas é preciso encadear `.distinct()`.
</details>

---

### 5. A tabela Bronze `clientes_raw` recebe várias versões do mesmo cliente, com a coluna `atualizado_em`. Como manter apenas a versão mais recente de cada `cliente_id` na Silver?

A) `df.dropDuplicates(["cliente_id"])`
B) `df.distinct()`
C) `row_number()` sobre uma janela particionada por `cliente_id` e ordenada por `atualizado_em` desc, filtrando `rn = 1`
D) `df.groupBy("cliente_id").count()`

<details><summary>Resposta</summary>

**C.** `dropDuplicates` mantém uma linha qualquer por chave, sem garantir que seja a mais recente.
</details>

---

### 6. O time precisa de uma estimativa rápida do número de usuários únicos em 10 bilhões de eventos, aceitando pequena margem de erro. Qual função usar?

A) `countDistinct("user_id")`
B) `approx_count_distinct("user_id")`
C) `count("user_id")`
D) `sum("user_id")`

<details><summary>Resposta</summary>

**B.** `approx_count_distinct` (HyperLogLog) é muito mais barato que a contagem exata em grandes volumes.
</details>

---

### 7. Uma agregação gera spill em disco porque cada partição pós-shuffle ficou com vários GB. Qual parâmetro ajustar?

A) Aumentar `spark.sql.shuffle.partitions`
B) Reduzir `spark.sql.shuffle.partitions`
C) Aumentar `spark.default.parallelism`
D) Definir `spark.sql.autoBroadcastJoinThreshold = -1`

<details><summary>Resposta</summary>

**A.** Mais partições pós-shuffle = partições menores, que cabem em memória. `default.parallelism` vale para RDDs.
</details>

---

### 8. Um dashboard de BI consulta várias vezes por hora uma agregação cara sobre a Silver. Os dados podem ter até 1 hora de atraso. Qual objeto Gold é o mais indicado?

A) Uma view comum
B) Uma materialized view com refresh agendado de hora em hora
C) Uma temporary view
D) Uma streaming table lendo da Silver com `STREAM`

<details><summary>Resposta</summary>

**B.** A materialized view guarda o resultado pré-calculado. A view comum recalcularia a agregação a cada consulta. Streaming table pede fonte append-only e não é ideal para agregações que mudam.
</details>

---

### 9. Em um pipeline declarativo, registros com `valor <= 0` devem ser removidos da tabela Silver, mas o pipeline deve continuar. Qual expectation usar?

A) `CONSTRAINT v EXPECT (valor > 0)`
B) `CONSTRAINT v EXPECT (valor > 0) ON VIOLATION DROP ROW`
C) `CONSTRAINT v EXPECT (valor > 0) ON VIOLATION FAIL UPDATE`
D) `ALTER TABLE ADD CONSTRAINT v CHECK (valor > 0)`

<details><summary>Resposta</summary>

**B.** A opção A só registra a violação e mantém a linha. C para o fluxo. D faria a escrita falhar.
</details>

---

### 10. Qual comando cria uma tabela Delta **vazia** com as colunas `id STRING, inicio DATE, nota FLOAT`, **independentemente** de já existir uma tabela com esse nome?

A) `CREATE TABLE IF NOT EXISTS t (id STRING, inicio DATE, nota FLOAT)`
B) `CREATE OR REPLACE TABLE t (id STRING, inicio DATE, nota FLOAT)`
C) `CREATE TABLE t AS SELECT id, inicio, nota`
D) `INSERT OVERWRITE t (id STRING, inicio DATE, nota FLOAT)`

<details><summary>Resposta</summary>

**B.** `IF NOT EXISTS` não faz nada se a tabela já existir, mantendo os dados antigos. `CREATE OR REPLACE` sempre termina com a tabela nova e vazia.
</details>

---

### 11. Uma tabela tem a coluna `itens ARRAY<STRUCT<sku STRING, qtd INT>>`. Alguns pedidos têm array vazio, e todos os pedidos precisam aparecer no resultado. Qual função usar?

A) `explode(itens)`
B) `explode_outer(itens)`
C) `flatten(itens)`
D) `array_distinct(itens)`

<details><summary>Resposta</summary>

**B.** `explode` descarta pedidos com array vazio ou nulo. `explode_outer` os mantém com valores nulos.
</details>
