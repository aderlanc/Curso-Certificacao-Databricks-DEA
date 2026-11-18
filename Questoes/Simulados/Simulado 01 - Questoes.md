# 📝 Simulado 01 — Databricks Data Engineer Associate

> **45 questões · 90 minutos · sem consulta.** Marque suas respostas numa folha e só depois abra o [gabarito](Simulado%2001%20-%20Gabarito.md).
> Distribuição igual à do exame (guia de ago/2026): D1 3 · D2 8 · D3 11 · D4 7 · D5 5 · D6 4 · D7 7 (próxima da proporção do exame).
> Questões **inéditas**, escritas no estilo da prova. Diferentes das questões por domínio em `Questoes/`.
> Meta: **≥ 80% (36 acertos)** antes de agendar a prova. A Databricks não publica a nota de corte.

---

**1.** Um engenheiro está em dúvida sobre onde ficam os notebooks, o agendador de jobs e a UI do workspace. Em qual camada da arquitetura do Databricks eles ficam?

A) No storage da conta de nuvem do cliente
B) Dentro de cada cluster
C) No control plane, gerenciado pela Databricks
D) No compute plane clássico, na assinatura do cliente

---

**2.** Uma equipe pequena desenvolve notebooks durante o dia e quer evitar esperar minutos pelo startup do cluster, sem gerenciar configurações. Qual opção atende melhor?

A) Serverless compute para notebooks
B) All-purpose cluster com auto-termination desligado
C) SQL warehouse classic
D) Job cluster

---

**3.** Para que servem os instance pools no compute clássico?

A) Para dividir o custo de DBU entre equipes
B) Para manter VMs ociosas prontas e reduzir o tempo de startup e de autoscaling dos clusters
C) Para aplicar permissões do Unity Catalog
D) Para executar queries SQL sem cluster

---

**4.** Um `COPY INTO` precisa carregar de novo arquivos que já foram carregados, porque a tabela foi truncada. Qual opção usar?

A) Não é possível; é preciso renomear os arquivos
B) `FILEFORMAT = DELTA`
C) `FORMAT_OPTIONS ('mergeSchema' = 'true')`
D) `COPY_OPTIONS ('force' = 'true')`

---

**5.** Qual código inicia corretamente uma leitura com Auto Loader de arquivos CSV?

A) `spark.readStream.format("csv").option("autoLoader", "true").load(path)`
B) `spark.readStream.format("autoloader").load(path)`
C) `spark.readStream.format("cloudFiles").option("cloudFiles.format", "csv").option("cloudFiles.schemaLocation", sl).load(path)`
D) `spark.read.format("cloudFiles").option("cloudFiles.format", "csv").load(path)`

---

**6.** Uma fonte JSON envia às vezes o campo `idade` como texto (`"trinta"`). O time não quer que o stream falhe nem perder esses dados, e quer manter o schema fixo. Qual modo de schema evolution usar no Auto Loader?

A) `rescue`
B) `failOnNewColumns`
C) `addNewColumns`
D) `none`

---

**7.** Por que é recomendado definir `cloudFiles.inferColumnTypes = true` ao ler JSON e CSV com Auto Loader?

A) Porque ela desliga o `_rescued_data`
B) Porque sem ela o Auto Loader não cria checkpoint
C) Porque sem essa opção todas as colunas são inferidas como STRING
D) Porque ela ativa o modo file notification

---

**8.** A empresa precisa ingerir tabelas de um SQL Server on-premises para o Unity Catalog, de forma incremental (CDC), com o mínimo de código e infraestrutura para manter. Qual opção é a mais indicada?

A) Lakehouse Federation
B) Exportar CSVs e fazer upload pela UI
C) Notebook com JDBC lendo a tabela inteira todo dia
D) Managed connector do Lakeflow Connect para SQL Server

---

**9.** Um time precisa consultar ocasionalmente tabelas de um PostgreSQL externo, juntando com tabelas do Unity Catalog, **sem copiar** os dados para o lakehouse. Qual recurso usar?

A) COPY INTO
B) Lakehouse Federation (connection + foreign catalog)
C) Auto Loader
D) Delta Sharing aberto

---

**10.** Um notebook consome uma API REST que retorna JSON paginado e deve rodar a cada hora. Qual é a abordagem recomendada?

A) Usar COPY INTO diretamente na URL da API
B) Colar a resposta da API manualmente numa tabela
C) Gravar as respostas como arquivos JSON num Volume e carregá-las com Auto Loader, orquestrando com Lakeflow Jobs
D) Usar Lakehouse Federation apontando para a URL da API

---

**11.** Onde as credenciais de um banco acessado via JDBC devem ficar?

A) Em uma célula markdown
B) Num widget do notebook
C) Em Databricks secrets, lidas com `dbutils.secrets.get`
D) No nome da tabela de destino

---

**12.** Uma coluna `evento` do tipo STRING contém `{"tipo":"click","pos":{"x":10}}`. Qual consulta retorna o valor de `x`?

A) `SELECT explode(evento) FROM t`
B) `SELECT evento:pos.x FROM t`
C) `SELECT evento.pos.x FROM t`
D) `SELECT evento->pos->x FROM t`

---

**13.** Qual afirmação sobre Auto Loader e COPY INTO é verdadeira?

A) COPY INTO escala melhor para milhões de arquivos
B) COPY INTO exige Structured Streaming
C) Auto Loader é recomendado para ingestão incremental contínua de grandes volumes de arquivos; COPY INTO atende cargas SQL simples de milhares de arquivos
D) Auto Loader não suporta JSON

---

**14.** Uma tabela Bronze tem a coluna `valor` como STRING e alguns registros têm texto inválido. Na Silver, valores inválidos devem virar NULL **sem** a consulta falhar. Qual expressão SQL usar?

A) `COALESCE(valor)`
B) `CONVERT(valor)`
C) `TRY_CAST(valor AS DECIMAL(10,2))`
D) `CAST(valor AS DECIMAL(10,2))`

---

**15.** Qual código retorna os clientes que **não** têm nenhum pedido?

A) `clientes.crossJoin(pedidos)`
B) `clientes.join(pedidos, "cliente_id", "inner")`
C) `clientes.join(pedidos, "cliente_id", "left_semi")`
D) `clientes.join(pedidos, "cliente_id", "left_anti")`

---

**16.** `cores` tem 4 linhas e `tamanhos` tem 5 linhas. Quantas linhas retorna `cores.crossJoin(tamanhos)`?

A) 4
B) 5
C) 9
D) 20

---

**17.** Em SQL, `SELECT id FROM a UNION SELECT id FROM b`, onde `a` e `b` têm o id 7. Quantas vezes o 7 aparece no resultado?

A) Nenhuma
B) Uma
C) Duas
D) Depende da ordem das tabelas

---

**18.** Qual é o efeito de `spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)`?

A) O broadcast join automático é desligado
B) O AQE é desligado
C) As partições de shuffle passam a ser ilimitadas
D) Todo join vira broadcast

---

**19.** Um engenheiro mudou `spark.sql.shuffle.partitions` de 200 para 50 para uma tabela pequena. Qual é o próximo passo recomendado?

A) Executar VACUUM
B) Reiniciar o metastore
C) Aplicar o mesmo valor em todos os jobs da empresa
D) Medir de novo o tempo e as métricas no Spark UI para confirmar o ganho

---

**20.** Qual método do DataFrame mostra count, mean, stddev, min, max **e** os percentis 25%, 50% e 75%?

A) `df.count()`
B) `df.describe()`
C) `df.summary()`
D) `df.printSchema()`

---

**21.** Um time de BI quer uma camada Gold sempre atualizada, com uma lógica simples de filtro sobre a Silver, sem custo de armazenamento extra. Qual objeto usar?

A) View
B) Streaming table
C) Materialized view
D) Tabela com CTAS diário

---

**22.** Em um pipeline declarativo, a tabela `bronze_eventos` deve ler incrementalmente novos arquivos JSON de um Volume. Qual definição SQL está correta?

A) `CREATE MATERIALIZED VIEW bronze_eventos AS SELECT * FROM read_files('/Volumes/raw/eventos')`
B) `CREATE OR REFRESH STREAMING TABLE bronze_eventos AS SELECT * FROM STREAM read_files('/Volumes/raw/eventos', format => 'json')`
C) `CREATE VIEW bronze_eventos AS SELECT * FROM json.'/Volumes/raw/eventos'`
D) `COPY INTO bronze_eventos STREAM FROM '/Volumes/raw/eventos'`

---

**23.** Em um pipeline declarativo, um pedido sem `cliente_id` significa erro grave de origem, e o pipeline deve **parar** se isso acontecer. Qual expectation usar?

A) `EXPECT (cliente_id IS NOT NULL)`
B) `EXPECT (cliente_id IS NOT NULL) ON VIOLATION DROP ROW`
C) `EXPECT (cliente_id IS NOT NULL) ON VIOLATION WARN`
D) `EXPECT (cliente_id IS NOT NULL) ON VIOLATION FAIL UPDATE`

---

**24.** Qual comando faz upsert dos registros de `novos` na tabela `clientes`, atualizando quem já existe e inserindo quem não existe?

A) `INSERT OVERWRITE clientes SELECT * FROM novos`
B) `MERGE INTO clientes c USING novos n ON c.id = n.id WHEN MATCHED THEN UPDATE SET * WHEN NOT MATCHED THEN INSERT *`
C) `INSERT INTO clientes SELECT * FROM novos`
D) `UPDATE clientes SET * FROM novos`

---

**25.** Um job deve rodar todos os dias às 6h no horário de Brasília para o relatório das 7h. Os dados de origem estão sempre prontos à meia-noite. Qual trigger usar?

A) Scheduled com fuso horário `America/Sao_Paulo`
B) File arrival
C) Table update
D) Continuous

---

**26.** Uma task de notificação deve rodar **somente** se alguma das tasks anteriores falhar. Qual *Run if* usar?

A) None failed
B) All succeeded
C) At least one failed
D) All done

---

**27.** Qual tipo de task executa um Lakeflow Declarative Pipeline (ex-DLT) dentro de um job?

A) dbt task
B) Pipeline task
C) SQL alert task
D) Notebook task apontando para o código do pipeline

---

**28.** Em um job, a task `listar` grava um task value `arquivos` com uma lista de 50 nomes. A próxima etapa deve processar cada arquivo com o mesmo notebook, 10 por vez. Como configurar?

A) Run Job task chamada 50 vezes manualmente
B) For each task com inputs `{{tasks.listar.values.arquivos}}` e concorrência 10
C) If/else condition com 50 ramos
D) Continuous trigger

---

**29.** Um job tem 3 tasks que usam o mesmo tipo de cluster clássico. Como reduzir o tempo e o custo de startup entre elas?

A) Usar um all-purpose cluster para cada task
B) Usar um job cluster compartilhado entre as tasks do job
C) Aumentar o número de retries
D) Criar 3 jobs separados

---

**30.** Uma task Gold só deve rodar se a Silver **e** a dimensão de produtos terminarem com sucesso. As duas rodam em paralelo. Como configurar a task Gold?

A) Sem dependências
B) Dependendo das duas tasks, com *Run if: All succeeded*
C) Dependendo das duas tasks, com *Run if: At least one succeeded*
D) Dependendo só da Silver

---

**31.** Qual referência dinâmica insere a data de início do run em um parâmetro de task?

A) `${job.date}`
B) `{{job.start_time.iso_date}}`
C) `$(date)`
D) `{{now}}`

---

**32.** Qual é a forma correta de trazer as mudanças mais recentes da `main` remota para o seu Git folder antes de começar uma feature?

A) Clonar o repositório de novo em outra pasta
B) Fazer push com `--force`
C) Fazer **pull** pela UI do Git folder
D) Usar o Version History do notebook

---

**33.** Em um bundle, o job deve usar `cluster pequeno` em dev e `cluster grande` em prod. Onde essa diferença deve ser declarada?

A) No `.gitignore`
B) Em um widget do notebook
C) No mapeamento `targets`, sobrescrevendo a configuração/variável por ambiente
D) Em dois repositórios Git separados

---

**34.** Qual é a ordem típica de comandos para publicar e testar um bundle em dev?

A) `bundle destroy` → `bundle run`
B) `bundle validate` → `bundle deploy -t dev` → `bundle run -t dev <job>`
C) `bundle run` → `bundle validate` → `bundle deploy`
D) `bundle init` → `bundle destroy`

---

**35.** No pipeline de CI/CD (GitHub Actions), como a Databricks CLI deve se autenticar no workspace de produção?

A) Com usuário e senha escritos no `databricks.yml`
B) Com um service principal usando OAuth M2M (client id e secret guardados como secrets do CI)
C) Com o token pessoal do engenheiro mais antigo
D) Sem autenticação, pois o bundle é público

---

**36.** Qual afirmação sobre o modo `development` de um target de bundle é verdadeira?

A) Só pode ser usado com clusters all-purpose
B) Os schedules ficam ativos imediatamente
C) Os recursos recebem o prefixo `[dev <usuário>]` e os schedules ficam pausados
D) Ele impede o deploy de pipelines

---

**37.** No Spark UI, um stage de join mostra 400 tasks com mediana de 5 s, mas 3 tasks levam 20 minutos e leem 100 vezes mais dados de shuffle. Qual é o diagnóstico?

A) Data skew na chave do join
B) Cluster subdimensionado de forma uniforme
C) Muitos arquivos pequenos
D) Falta de Photon

---

**38.** Na aba Runs de um job, qual visão mostra o status e a duração de cada task em várias execuções lado a lado, ajudando a achar qual task ficou mais lenta?

A) Catalog Explorer
B) Event log
C) Query history
D) Matrix view

---

**39.** Qual afirmação sobre Predictive Optimization está correta?

A) Precisa ser agendada como um job
B) Executa automaticamente OPTIMIZE, VACUUM e ANALYZE em tabelas managed do Unity Catalog
C) Substitui o Unity Catalog
D) Funciona em tabelas external de qualquer formato

---

**40.** Depois de instalar com `%pip` uma versão nova de uma biblioteca, um notebook passa a falhar com `ImportError` em outra biblioteca que dependia da versão antiga. Qual é a melhor ação?

A) Fixar versões compatíveis das bibliotecas (ex.: `%pip install lib==x.y`) e manter as instalações no escopo do notebook
B) Trocar o trigger do job
C) Desligar o AQE
D) Aumentar a memória do driver

---

**41.** Um grupo precisa **inserir e atualizar** dados na tabela `vendas.silver.pedidos`, além de ler. Já tem `USE CATALOG` e `USE SCHEMA`. Quais privilégios conceder na tabela?

A) `CREATE TABLE`
B) `EXECUTE`
C) `SELECT` e `MODIFY`
D) Só `SELECT`

---

**42.** Uma tabela managed foi excluída por engano há 2 dias. Como recuperá-la?

A) `RESTORE TABLE t TO VERSION AS OF 0`
B) `UNDROP TABLE t`
C) `CREATE TABLE t LOCATION` no caminho antigo
D) Não é possível recuperar tabelas managed

---

**43.** Qual é a diferença entre um row filter e um column mask no Unity Catalog?

A) Column mask só funciona em views
B) Row filter exige tabela external
C) Row filter esconde linhas conforme quem consulta; column mask altera o valor exibido de uma coluna conforme quem consulta
D) Ambos removem colunas da tabela

---

**44.** Qual função é usada em row filters e column masks para verificar se quem consulta pertence a um grupo da conta?

A) `is_account_group_member('grupo')`
B) `current_database()`
C) `has_permission('grupo')`
D) `user_in_group()`

---

**45.** Um analista recebeu `GRANT SELECT ON TABLE financas.gold.dre`, mas não deve ter acesso a mais nada do schema. Um admin propõe `GRANT SELECT ON SCHEMA financas.gold` "para simplificar". Qual é o problema dessa proposta?

A) GRANT em schema não existe no Unity Catalog
B) Nenhum, é equivalente
C) Removeria o acesso à tabela `dre`
D) Daria SELECT em todas as tabelas atuais e futuras do schema, violando o menor privilégio
