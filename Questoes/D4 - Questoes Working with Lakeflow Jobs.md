# Questões — Domínio 4: Working with Lakeflow Jobs

> Questões **inéditas** escritas no estilo da prova, a partir dos objetivos do guia oficial (ago/2026). Não são questões vazadas.

---

### 1. Um fornecedor deposita arquivos numa external location em horários imprevisíveis. Eles devem ser processados logo que chegam, sem manter compute ligado à toa. Qual trigger usar?

A) Scheduled a cada 5 minutos
B) Continuous
C) File arrival
D) Table update

<details><summary>Resposta</summary>

**C.** File arrival dispara o job quando chegam arquivos novos no caminho monitorado. Scheduled de 5 em 5 minutos e continuous gastariam compute sem dados.
</details>

---

### 2. O job Gold do time de BI deve rodar sempre que a tabela `vendas.silver.pedidos`, mantida por outro time, for atualizada. Qual trigger usar?

A) Scheduled no horário em que a Silver costuma terminar
B) Table update monitorando `vendas.silver.pedidos`
C) File arrival na pasta da Silver
D) Manual

<details><summary>Resposta</summary>

**B.** Table update reage a mudanças em tabelas do Unity Catalog e desacopla os dois jobs.
</details>

---

### 3. Uma task de limpeza deve rodar **sempre** ao final do job, mesmo que tasks anteriores falhem. Qual condição *Run if* usar?

A) All succeeded
B) None failed
C) All done
D) At least one succeeded

<details><summary>Resposta</summary>

**C.** All done executa a task quando as dependências terminam, com qualquer resultado.
</details>

---

### 4. A task `validar` calcula quantas linhas novas chegaram. Se for zero, o job deve só notificar; caso contrário, deve processar a Silver. Como implementar?

A) Duas tasks em paralelo sem dependência
B) `validar` define um task value e uma task **If/else condition** compara `{{tasks.validar.values.qtd}} > 0` para escolher o ramo
C) Uma task For each sobre as linhas
D) Usar *Run if: At least one failed*

<details><summary>Resposta</summary>

**B.** Task values levam dados entre tasks, e a If/else condition desvia o fluxo conforme o valor.
</details>

---

### 5. Qual comando grava um valor que outra task do mesmo job poderá ler?

A) `dbutils.widgets.text("qtd", "10")`
B) `dbutils.jobs.taskValues.set(key="qtd", value=10)`
C) `spark.conf.set("qtd", 10)`
D) `dbutils.notebook.exit("10")` é a única forma

<details><summary>Resposta</summary>

**B.** Quem lê usa `dbutils.jobs.taskValues.get(taskKey=..., key="qtd")` ou `{{tasks.<task>.values.qtd}}`.
</details>

---

### 6. É preciso rodar o mesmo notebook para 30 filiais, no máximo 5 ao mesmo tempo, sem criar 30 tasks. Qual recurso usar?

A) Run Job task
B) For each task com a lista de filiais e concorrência 5
C) 30 jobs diferentes
D) Continuous trigger

<details><summary>Resposta</summary>

**B.** For each itera sobre uma lista (fixa ou vinda de task value) e controla quantas iterações rodam em paralelo.
</details>

---

### 7. Qual tipo de compute é o mais indicado para um job de produção agendado, considerando custo?

A) All-purpose cluster compartilhado com os analistas
B) Job cluster ou serverless para jobs
C) SQL warehouse classic para notebooks Python
D) Cluster single-node sempre ligado

<details><summary>Resposta</summary>

**B.** Job clusters existem só durante o run e têm DBU mais barato que all-purpose. Serverless elimina a configuração e o tempo de startup.
</details>

---

### 8. Uma task falha ocasionalmente por instabilidade momentânea de rede na API de origem. O que configurar para que o job se recupere sozinho?

A) Retries na task, com intervalo entre tentativas
B) Aumentar o número de workers
C) Trocar o trigger para continuous
D) Remover as dependências

<details><summary>Resposta</summary>

**A.** Retries reexecutam a task automaticamente após a falha.
</details>

---

### 9. Um job agendado de hora em hora às vezes leva mais de uma hora. O time não quer que dois runs rodem ao mesmo tempo, mas também não quer perder o run seguinte. O que configurar?

A) Max concurrent runs = 1 com **queueing** habilitado
B) Max concurrent runs = 10
C) Timeout de 30 minutos
D) Trigger continuous

<details><summary>Resposta</summary>

**A.** Com limite 1, um run extra seria pulado. Com queueing ele espera na fila e roda quando o anterior terminar.
</details>
