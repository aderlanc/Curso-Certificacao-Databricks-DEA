# 🧪 Laboratórios Hands-on — mínimo prático para a DEA

> Passo 5 do [método de preparação com IA](03%20-%20Preparacao%20com%20IA.md): a prova cobra habilidade prática.
> Cada laboratório cabe em **até 30 minutos** na [Databricks Free Edition](https://www.databricks.com/learn/free-edition) (ou trial / Azure Databricks) e está ligado a objetivos do Exam Guide de ago/2026.
> Laboratório que você não conseguir terminar = próximo alvo de estudo. Registre no [diário](05%20-%20Diario%20de%20Estudo.md).

**Antes de começar:** crie um catálogo/schema de estudo e um volume para os arquivos.

```sql
CREATE SCHEMA IF NOT EXISTS workspace.dea_lab;      -- na Free Edition o catálogo padrão é "workspace"
CREATE VOLUME IF NOT EXISTS workspace.dea_lab.landing;
```

> ℹ️ A Free Edition só tem **serverless**. Onde um laboratório precisar de cluster clássico (Spark UI completa, logs de driver, init scripts), isso está indicado; nesses casos use um trial ou o Azure Databricks do curso.
> Confirme sintaxes e limites em [docs.databricks.com](https://docs.databricks.com/) antes de concluir que algo "não funciona".

---

## Resumo

| # | Laboratório | Domínio | Status |
|---|---|---|---|
| L1 | Auto Loader com schema evolution | D2 | ⬜ |
| L2 | COPY INTO idempotente | D2 | ⬜ |
| L3 | Bronze → Silver com joins, deduplicação e agregações | D3 | ⬜ |
| L4 | Lakeflow Spark Declarative Pipeline com expectations e AUTO CDC | D3 | ⬜ |
| L5 | Lakeflow Job com dependências, If/else, For each e file arrival | D4 | ⬜ |
| L6 | Git folder + Declarative Automation Bundle pela CLI | D5 | ⬜ |
| L7 | Run history, repair run e leitura do Spark UI / query profile | D6 | ⬜ |
| L8 | Liquid Clustering e manutenção de tabelas | D6 | ⬜ |
| L9 | GRANT/REVOKE, row filter e column mask | D7 | ⬜ |

Troque ⬜ por ✅ quando conseguir fazer **sem consultar o passo a passo**.

---

## L1 — Auto Loader com schema evolution (D2)

**Objetivo do guia:** *Auto Loader com schema enforcement e schema evolution, nos modos directory listing ou file notification.*

1. Faça upload de um JSON `pedidos_1.json` (campos `id`, `cliente`, `valor`) para `/Volumes/workspace/dea_lab/landing/pedidos/`.
2. Rode o stream com `trigger(availableNow=True)`:
   ```python
   (spark.readStream.format("cloudFiles")
      .option("cloudFiles.format", "json")
      .option("cloudFiles.schemaLocation", "/Volumes/workspace/dea_lab/landing/_schemas/pedidos")
      .load("/Volumes/workspace/dea_lab/landing/pedidos/")
    .writeStream
      .option("checkpointLocation", "/Volumes/workspace/dea_lab/landing/_chk/pedidos")
      .trigger(availableNow=True)
      .toTable("workspace.dea_lab.bronze_pedidos"))
   ```
3. Envie `pedidos_2.json` com uma coluna nova (`canal`) e rode de novo. Observe a falha e rode uma terceira vez.

**Pronto quando:** você explica por que o stream falhou, onde o schema novo foi salvo, o que é `_rescued_data` e como `cloudFiles.schemaEvolutionMode` muda o comportamento (`addNewColumns`, `rescue`, `failOnNewColumns`, `none`).

## L2 — COPY INTO idempotente (D2)

**Objetivo do guia:** *COPY INTO para carga incremental do storage em tabelas do UC.*

1. `CREATE TABLE IF NOT EXISTS workspace.dea_lab.bronze_clientes;` (sem schema).
2. Rode `COPY INTO ... FROM '/Volumes/.../clientes/' FILEFORMAT = CSV FORMAT_OPTIONS ('header'='true','inferSchema'='true') COPY_OPTIONS ('mergeSchema'='true')` **duas vezes**.
3. Adicione um CSV novo e rode de novo. Depois teste `COPY_OPTIONS ('force'='true')`.

**Pronto quando:** você mostra com `SELECT count(*)` que a segunda execução não duplicou nada, e sabe dizer quando escolher COPY INTO, Auto Loader ou Lakeflow Connect.

## L3 — Bronze → Silver com joins, deduplicação e agregações (D3)

**Objetivos do guia:** *limpeza Bronze → Silver; joins (inner, left, broadcast, cross, union/union all); deduplicação e agregações; parâmetros de tuning.*

1. A partir de `bronze_pedidos`, trate nulos, converta tipos (`cast`) e deduplique por `id` (`dropDuplicates` em PySpark **e** `ROW_NUMBER() OVER (PARTITION BY id ORDER BY ...)` em SQL).
2. Faça join com `bronze_clientes` usando `/*+ BROADCAST(c) */` e confira o plano com `EXPLAIN` (procure `BroadcastHashJoin`).
3. Compare `UNION` x `UNION ALL` e um `CROSS JOIN` pequeno.
4. Agregue com `count`, `approx_count_distinct`, `avg` e rode `df.summary()`.
5. Veja o valor de `spark.sql.shuffle.partitions` e `spark.sql.autoBroadcastJoinThreshold` (`SET spark.sql.shuffle.partitions;`).

**Pronto quando:** você escreve a tabela Silver nas duas linguagens e explica quando o broadcast é bom e quando é perigoso.

## L4 — Lakeflow Spark Declarative Pipeline (D3)

**Objetivos do guia:** *objetos Gold (materialized views, views, streaming tables); data quality nas camadas Silver/Gold.*

1. Crie um pipeline (ETL pipeline) com: uma **streaming table** Bronze lendo o volume com Auto Loader, uma Silver com três expectations (`EXPECT ... ON VIOLATION DROP ROW`, `FAIL UPDATE` e só warn) e uma **materialized view** Gold agregada.
2. Adicione um fluxo `AUTO CDC INTO` (SCD tipo 1) a partir de um arquivo de mudanças.
3. Rode em modo **triggered**, veja o grafo e as métricas de qualidade.

**Pronto quando:** você sabe dizer a diferença entre streaming table, materialized view e view, e o efeito de cada ação de expectation.

## L5 — Lakeflow Job completo (D4)

**Objetivos do guia:** *control flow (retries, If/else, For each); tasks e dependências; schedules e triggers (scheduled, file arrival, table update).*

1. Job com 3 tasks: `ingest` (notebook) → `quality` (notebook que define `dbutils.jobs.taskValues.set("linhas", n)`) → `if_ok` (**If/else condition** usando `{{tasks.quality.values.linhas}} > 0`).
2. No ramo verdadeiro, uma task **For each** que roda um notebook para cada valor de `["norte","sul","leste"]`.
3. Configure **retries** em `ingest` e troque o trigger para **File arrival** no volume. Envie um arquivo e veja o job disparar.

**Pronto quando:** você monta o DAG sem consultar e explica a diferença entre trigger por tempo (schedule) e por dado (file arrival, table update).

## L6 — Git folder + Declarative Automation Bundle (D5)

**Objetivos do guia:** *Git folders (branch, commit, push, PR); variáveis e overrides por ambiente; deploy de bundles; Databricks CLI.*

1. Crie um **Git folder** ligado a um repositório seu no GitHub, crie uma branch, altere um notebook, faça commit e push e abra um pull request.
2. No computador (ou no terminal web do workspace), com a Databricks CLI nova instalada e autenticada:
   ```bash
   databricks bundle init            # template default-python
   databricks bundle validate -t dev
   databricks bundle deploy  -t dev
   databricks bundle run     -t dev <nome_do_job>
   ```
3. Em `databricks.yml`, crie uma variável (`variables: catalogo: default: dev_catalog`) e um override no target `prod`.

**Pronto quando:** você explica o que é `targets`, `mode: development` (prefixo e pausa de schedules) e a ordem validate → deploy → run.

## L7 — Monitoramento e troubleshooting (D6)

**Objetivos do guia:** *tendência no run history; saúde do pipeline na UI de Jobs; gargalos no Spark UI (skew, shuffle, spill); falhas de cluster, bibliotecas e OOM.*

1. Rode o job do L5 várias vezes e veja a **matrix view** e a duração de cada run.
2. Force uma falha numa task (ex.: `raise Exception`), corrija e use **Repair run** para rodar só o que falhou.
3. Rode um `groupBy` com uma chave muito concentrada e abra o **query profile** (serverless) ou o **Spark UI** (cluster clássico, fora da Free Edition): encontre stages, tasks lentas, shuffle read/write e spill.

**Pronto quando:** você identifica skew (uma task muito mais lenta que a mediana), shuffle e spill num print, e sabe a primeira ação para cada um.

## L8 — Liquid Clustering e manutenção (D6)

**Objetivo do guia:** *Liquid Clustering e Predictive Optimization.*

1. `CREATE TABLE workspace.dea_lab.vendas_lc CLUSTER BY (data_venda) AS SELECT ...`
2. `ALTER TABLE ... CLUSTER BY (cliente_id);` depois `OPTIMIZE ...;` e `DESCRIBE DETAIL ...` (veja `clusteringColumns`).
3. Teste `CLUSTER BY AUTO` e verifique se Predictive Optimization está habilitado no schema/catálogo (`DESCRIBE ... EXTENDED` ou Catalog Explorer).

**Pronto quando:** você explica por que Liquid Clustering substitui partitioning + ZORDER em tabelas novas e o que Predictive Optimization faz sozinho.

## L9 — Governança (D7)

**Objetivos do guia:** *managed x external e conversão; GRANT/REVOKE/DENY; column masking e row-level security; ABAC.*

1. Dê e retire permissões: `GRANT USE CATALOG`, `USE SCHEMA` e `SELECT` a um grupo; `REVOKE`; confira com `SHOW GRANTS ON TABLE ...`.
2. Crie um row filter e um column mask:
   ```sql
   CREATE FUNCTION workspace.dea_lab.so_sul(regiao STRING)
     RETURN is_account_group_member('admins') OR regiao = 'sul';
   ALTER TABLE workspace.dea_lab.silver_pedidos SET ROW FILTER workspace.dea_lab.so_sul ON (regiao);

   CREATE FUNCTION workspace.dea_lab.mascara_email(email STRING)
     RETURN CASE WHEN is_account_group_member('admins') THEN email ELSE '***' END;
   ALTER TABLE workspace.dea_lab.silver_pedidos ALTER COLUMN email SET MASK workspace.dea_lab.mascara_email;
   ```
3. Leia na documentação como uma política **ABAC** aplica o mesmo filtro/máscara por **tag** em vez de tabela por tabela (verifique a disponibilidade no seu workspace).
4. Na teoria (ou no Azure do curso): crie uma tabela external e veja na documentação como convertê-la para managed.

**Pronto quando:** você explica a diferença entre row filter, column mask e ABAC, e qual privilégio falta quando alguém recebe `SELECT` mas não consegue ler a tabela (`USE CATALOG`/`USE SCHEMA`).
