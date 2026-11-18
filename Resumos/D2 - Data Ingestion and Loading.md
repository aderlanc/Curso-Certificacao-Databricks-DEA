# Domínio 2 — Data Ingestion and Loading (21% · ~9-10 questões)

> O curso cobre bem COPY INTO e Auto Loader (`07/1-3`) e a leitura de JSON/CSV/TSV/XML (`06/04-07`).
> Lakeflow Connect, JDBC/REST e a escolha entre os métodos são estudados por este resumo.

**Objetivos do guia:**
1. Padrões de ingestão: **batch**, **streaming**, **incremental**, upload de **arquivos locais** e **Lakeflow Connect**.
2. **COPY INTO** para carga incremental do storage (ADLS/S3/GCS) em tabelas do UC.
3. **Auto Loader** com schema enforcement e schema evolution, nos modos **directory listing** ou **file notification**.
4. Configurar **Lakeflow Connect** para ingerir de fontes corporativas em tabelas do UC.
5. **JDBC/ODBC** ou **REST** em notebooks para gravar no storage ou em tabelas, orquestrados por Jobs.
6. Escolher entre Auto Loader, Lakeflow Connect (standard e managed), partner connectors e outros, por volume, frequência, tipo de dado e governança.
7. Ingerir dados **semi-estruturados** e **não estruturados** (JSON, nested) em tabelas Delta.

---

## 1. Padrões de ingestão

| Padrão | Ideia | Ferramenta típica |
|---|---|---|
| **Batch completo** | Lê tudo de novo a cada execução | `spark.read`, `CREATE OR REPLACE TABLE ... AS SELECT` |
| **Incremental** | Só processa o que é novo desde a última vez | COPY INTO, Auto Loader com `availableNow`, Lakeflow Connect |
| **Streaming** | Processa continuamente, baixa latência | Auto Loader / Kafka com Structured Streaming |
| **Upload local** | Arquivo pequeno do seu computador | UI *Add data → Create or modify table* (vira tabela Delta managed) ou upload para um **Volume** |

## 2. COPY INTO

```sql
CREATE TABLE IF NOT EXISTS vendas.bronze.pedidos;   -- pode ser criada vazia, sem schema

COPY INTO vendas.bronze.pedidos
FROM '/Volumes/vendas/raw/landing/pedidos/'
FILEFORMAT = JSON
FORMAT_OPTIONS ('inferSchema' = 'true', 'mergeSchema' = 'true')
COPY_OPTIONS ('mergeSchema' = 'true');
```

- **Idempotente**: arquivos já carregados são ignorados em execuções seguintes. Para recarregar: `COPY_OPTIONS ('force' = 'true')`.
- Comando SQL, em **batch**. Bom para **milhares** de arquivos e cargas agendadas.
- Formatos: CSV, JSON, Avro, ORC, Parquet, text, binaryFile, XML.

## 3. Auto Loader (`cloudFiles`)

```python
(spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/Volumes/vendas/raw/_schemas/pedidos")  # onde guarda o schema inferido
    .option("cloudFiles.inferColumnTypes", "true")       # sem isso, JSON/CSV viram STRING
    .option("cloudFiles.schemaHints", "valor DOUBLE")    # força tipo de colunas específicas
    .load("/Volumes/vendas/raw/landing/pedidos/")
 .writeStream
    .option("checkpointLocation", "/Volumes/vendas/raw/_checkpoints/pedidos")  # garante exactly-once
    .trigger(availableNow=True)      # processa o que há de novo e PARA (incremental em batch)
    .toTable("vendas.bronze.pedidos"))
```

| `cloudFiles.schemaEvolutionMode` | Coluna nova aparece… |
|---|---|
| `addNewColumns` (**padrão**) | O stream **falha**, o schema é atualizado e, ao reiniciar, a coluna entra |
| `rescue` | Schema não muda; dados novos vão para a coluna **`_rescued_data`** |
| `failOnNewColumns` | O stream falha e não evolui até você mudar o schema |
| `none` | Ignora colunas novas (e `_rescued_data` só se configurada) |

- **`_rescued_data`**: coluna adicionada automaticamente com dados que não bateram com o schema (tipo errado, coluna a mais, caixa diferente).
- **Directory listing** (padrão): lista a pasta a cada micro-batch. Simples, sem permissões extras.
- **File notification** (`cloudFiles.useNotifications = true`, ou *managed file events*): usa fila/eventos da nuvem. Escala para **milhões** de arquivos por diretório.
- Triggers: `availableNow=True` (batch incremental), `processingTime="5 minutes"` (micro-batch periódico), sem trigger (padrão, ~500 ms).

### COPY INTO x Auto Loader
| | COPY INTO | Auto Loader |
|---|---|---|
| Interface | SQL | Python/SQL (`read_files` em streaming table) |
| Escala | Milhares de arquivos | Milhões de arquivos |
| Execução | Batch | Streaming ou batch incremental (`availableNow`) |
| Schema evolution | `mergeSchema` | Modos completos + `_rescued_data` |
| Recomendação Databricks | Cargas simples e pontuais | **Padrão** para ingestão incremental de arquivos |

Em SQL, dentro de pipelines declarativos:
```sql
CREATE OR REFRESH STREAMING TABLE bronze_pedidos AS
SELECT * FROM STREAM read_files('/Volumes/vendas/raw/landing/pedidos/', format => 'json');
```

## 4. Lakeflow Connect

Família de conectores de ingestão da Databricks, governados pelo Unity Catalog.

| Tipo | O que é | Exemplos |
|---|---|---|
| **Managed connectors** | Conector pronto e gerenciado: você configura pela UI/API, a Databricks cuida de incremental, schema e infraestrutura **serverless**. Cria um *ingestion pipeline* que grava **streaming tables** no UC | SaaS: Salesforce, Workday, ServiceNow, Google Analytics, SharePoint. Bancos: SQL Server, PostgreSQL, MySQL (via CDC) |
| **Standard connectors** | Conectores que você usa no seu código/pipeline, com mais controle | Auto Loader / cloud storage, Kafka, Kinesis, Event Hubs, Pub/Sub, upload de arquivos |

Como configurar um managed connector: criar uma **Connection** no UC com as credenciais → escolher objetos/tabelas da fonte → escolher catálogo/schema de destino → agendar o pipeline.

## 5. JDBC/ODBC e REST em notebooks

```python
# JDBC com leitura paralela e senha em secret
df = (spark.read.format("jdbc")
      .option("url", "jdbc:sqlserver://srv.database.windows.net:1433;database=erp")
      .option("dbtable", "dbo.pedidos")
      .option("user", dbutils.secrets.get("erp", "user"))
      .option("password", dbutils.secrets.get("erp", "password"))
      .option("partitionColumn", "id").option("lowerBound", 1)
      .option("upperBound", 10_000_000).option("numPartitions", 8)
      .load())
df.write.mode("append").saveAsTable("vendas.bronze.pedidos_erp")

# REST: gravar a resposta bruta num Volume e deixar o Auto Loader carregar
import requests, json, datetime
resp = requests.get("https://api.exemplo.com/v1/clientes", timeout=30)
caminho = f"/Volumes/vendas/raw/landing/api_clientes/{datetime.datetime.now():%Y%m%d%H%M%S}.json"
dbutils.fs.put(caminho, json.dumps(resp.json()))
```

- Sempre use **`dbutils.secrets`** para credenciais. Valores de secrets aparecem como `[REDACTED]` na saída.
- Orquestre com **Lakeflow Jobs** (agendamento, retries).
- **Lakehouse Federation** é diferente: consulta a fonte **no lugar**, sem copiar dados.

## 6. Qual método escolher?

| Cenário | Escolha |
|---|---|
| Arquivos chegando continuamente no storage, alto volume | **Auto Loader** (file notification se milhões de arquivos) |
| Carga SQL simples, agendada, poucos milhares de arquivos | **COPY INTO** |
| Dados de SaaS (Salesforce, Workday) ou banco com CDC, sem querer manter código | **Lakeflow Connect managed connector** |
| Kafka / Event Hubs | **Standard connector** com Structured Streaming |
| Banco sem conector managed, ou API própria | **JDBC / REST** em notebook + Job |
| Fonte coberta por ferramenta de parceiro (Fivetran etc.) | **Partner Connect** |
| Consultar sem copiar | **Lakehouse Federation** |
| CSV pequeno no seu notebook | **Upload pela UI** |

## 7. Semi-estruturados e não estruturados

```sql
-- JSON guardado como string: acessar campos com ":"
SELECT raw:cliente.nome, raw:itens[0].sku FROM bronze_eventos;

-- Converter string JSON em STRUCT
SELECT from_json(raw, schema_of_json('{"id":1,"nome":"a"}')) AS dados FROM bronze_eventos;

-- Tipo VARIANT: guarda JSON sem schema fixo, com leitura eficiente
SELECT parse_json(raw) AS v FROM bronze_eventos;

-- Arrays e structs
SELECT pedido_id, item.sku, item.qtd
FROM pedidos LATERAL VIEW explode(itens) t AS item;
```

- Não estruturados (PDF, imagens, áudio): guarde em **Volumes**. Leia com `format("binaryFile")` ou `read_files(..., format => 'binaryFile')`.
- `explode` remove linhas com array nulo ou vazio; `explode_outer` mantém (aula `06/08`).

---

## ✅ Checklist do domínio
- [ ] Sei que COPY INTO é idempotente e quando usar `force`
- [ ] Sei escrever um Auto Loader com `schemaLocation`, `checkpointLocation` e `availableNow`
- [ ] Sei os modos de schema evolution e o que é `_rescued_data`
- [ ] Sei diferenciar directory listing e file notification
- [ ] Sei diferenciar Lakeflow Connect managed x standard
- [ ] Sei escolher o método de ingestão para cada cenário

## 📚 Fontes oficiais
- [Ingestão no Databricks (visão geral)](https://learn.microsoft.com/azure/databricks/ingestion/)
- [COPY INTO](https://learn.microsoft.com/azure/databricks/ingestion/cloud-object-storage/copy-into/)
- [Auto Loader: schema evolution](https://learn.microsoft.com/azure/databricks/ingestion/cloud-object-storage/auto-loader/schema)
- [Lakeflow Connect](https://learn.microsoft.com/azure/databricks/ingestion/lakeflow-connect/)
- [JDBC](https://learn.microsoft.com/azure/databricks/connect/external-systems/jdbc)
