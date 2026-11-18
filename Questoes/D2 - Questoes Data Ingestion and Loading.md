# Questões — Domínio 2: Data Ingestion and Loading

> Questões **inéditas** escritas no estilo da prova, a partir dos objetivos do guia oficial (ago/2026). Não são questões vazadas.

---

### 1. Um job executa o mesmo comando `COPY INTO` todo dia sobre a mesma pasta. Ontem 10 arquivos foram carregados e hoje chegaram 3 novos. O que acontece na execução de hoje?

A) Os 13 arquivos são carregados de novo, gerando duplicatas
B) Somente os 3 arquivos novos são carregados
C) O comando falha porque a tabela já tem dados
D) A tabela é sobrescrita com os 3 arquivos novos

<details><summary>Resposta</summary>

**B.** COPY INTO é idempotente e registra os arquivos já carregados. Para forçar recarga use `COPY_OPTIONS ('force' = 'true')`.
</details>

---

### 2. Um stream do Auto Loader com configurações padrão encontra uma coluna nova nos arquivos JSON. Qual é o comportamento?

A) A coluna é ignorada silenciosamente
B) O stream falha, o schema é atualizado no `schemaLocation` e, ao reiniciar, a coluna nova passa a ser incluída
C) Os dados novos vão para `_rescued_data` e o schema nunca muda
D) A tabela de destino é recriada

<details><summary>Resposta</summary>

**B.** O modo padrão é `addNewColumns`. Em produção, configure o job com retries para que o reinício seja automático.
</details>

---

### 3. Qual é o nome da coluna onde o Auto Loader guarda dados que não correspondem ao schema esperado?

A) `_corrupt_record`
B) `_rescued_data`
C) `_metadata`
D) `_unparsed`

<details><summary>Resposta</summary>

**B.** `_rescued_data` recebe valores com tipo divergente, colunas a mais e diferenças de maiúsculas/minúsculas.
</details>

---

### 4. Um diretório recebe milhões de arquivos por dia e a listagem de diretório está deixando o Auto Loader lento. Qual mudança resolve?

A) Trocar para COPY INTO
B) Usar o modo file notification (`cloudFiles.useNotifications = true` ou file events)
C) Aumentar `spark.sql.shuffle.partitions`
D) Remover o `schemaLocation`

<details><summary>Resposta</summary>

**B.** File notification usa eventos da nuvem em vez de listar o diretório, e escala para milhões de arquivos. COPY INTO é indicado para volumes menores.
</details>

---

### 5. O time quer processar, uma vez por hora, todos os arquivos novos que chegaram desde a última execução e desligar o compute em seguida. Qual trigger usar no `writeStream` do Auto Loader?

A) `.trigger(processingTime="1 hour")` em um cluster sempre ligado
B) `.trigger(availableNow=True)` em um job agendado de hora em hora
C) `.trigger(continuous="1 hour")`
D) Nenhum trigger

<details><summary>Resposta</summary>

**B.** `availableNow` processa tudo o que está pendente e encerra a query, ideal para ingestão incremental agendada. `processingTime` mantém o stream rodando.
</details>

---

### 6. A empresa precisa ingerir diariamente dados do Salesforce para o Unity Catalog, de forma incremental, sem escrever e manter código de integração. Qual opção é a mais indicada?

A) Notebook com chamadas REST à API do Salesforce
B) Exportar CSVs manualmente e usar COPY INTO
C) Um managed connector do Lakeflow Connect
D) Lakehouse Federation

<details><summary>Resposta</summary>

**C.** Managed connectors do Lakeflow Connect cuidam de autenticação, carga incremental e evolução de schema, gravando em tabelas do UC com compute serverless. Federation consulta sem ingerir.
</details>

---

### 7. Qual configuração do Auto Loader é **obrigatória** para inferir e acompanhar a evolução do schema entre execuções?

A) `cloudFiles.schemaLocation`
B) `cloudFiles.maxFilesPerTrigger`
C) `cloudFiles.useNotifications`
D) `cloudFiles.includeExistingFiles`

<details><summary>Resposta</summary>

**A.** O `schemaLocation` guarda o schema inferido e suas versões. Normalmente aponta para um caminho junto do checkpoint.
</details>

---

### 8. Um engenheiro lê uma tabela de 500 milhões de linhas do SQL Server via JDBC e a leitura usa uma única task. Como paralelizar?

A) Aumentar o número de workers
B) Definir `partitionColumn`, `lowerBound`, `upperBound` e `numPartitions`
C) Usar `.repartition(8)` depois do `load()`
D) Trocar para `format("csv")`

<details><summary>Resposta</summary>

**B.** Sem essas opções o Spark abre só uma conexão. `repartition` depois do `load()` não paraleliza a leitura em si.
</details>

---

### 9. Uma coluna `payload` contém strings JSON como `{"cliente":{"nome":"Ana"}}`. Qual consulta SQL retorna o nome do cliente sem definir schema?

A) `SELECT payload.cliente.nome FROM t`
B) `SELECT payload:cliente.nome FROM t`
C) `SELECT payload['cliente']['nome'] FROM t`
D) `SELECT get_json(payload, nome) FROM t`

<details><summary>Resposta</summary>

**B.** A sintaxe com `:` extrai campos de strings JSON. A notação com ponto puro (A) é para colunas STRUCT.
</details>

---

### 10. Um analista precisa carregar uma única vez um CSV de 5 MB que está no seu notebook pessoal para uma tabela no Unity Catalog. Qual é o caminho mais simples?

A) Configurar Auto Loader com file notification
B) Criar um managed connector
C) Usar a UI *Add data → Create or modify table* (upload de arquivo)
D) Escrever um job JDBC

<details><summary>Resposta</summary>

**C.** O upload pela UI cria uma tabela Delta managed diretamente, sem pipeline.
</details>
