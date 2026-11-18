# Domínio 4 — Working with Lakeflow Jobs (16% · ~7 questões)

> O curso cobre tasks, dependências, Run if e schedule (`09/1-4`).
> If/else, For each, triggers de file arrival e table update, e a escolha do trigger são completados por este resumo.
> Nome atual: **Lakeflow Jobs** (antes: Databricks Workflows / Jobs).

**Objetivos do guia:**
1. **Control flow**: retries e tasks condicionais (**If/else** e **For each**).
2. Configurar tasks comuns (**notebook, SQL query, dashboard, pipeline**) e suas dependências no **DAG**.
3. Agendar com os triggers **scheduled**, **file arrival** e **table update**.
4. Escolher entre trigger por **tempo** e trigger por **dado**, conforme a disponibilidade dos dados e as dependências.

---

## 1. Anatomia de um job

```
Job
├── Tasks (cada uma com task_key, tipo, compute e dependências)
├── Trigger (schedule, file arrival, table update, continuous ou manual)
├── Parâmetros do job  → {{job.parameters.data_ref}}
├── Notificações       → início, sucesso, falha, duração, backlog
└── Permissões e run_as (usuário ou service principal)
```

### Tipos de task
| Tipo | Uso |
|---|---|
| **Notebook** | Notebook do workspace ou de um repositório Git |
| **Python script / Python wheel / JAR** | Código empacotado |
| **SQL** | Query, arquivo `.sql`, **alert** ou **dashboard legado**, rodando em SQL warehouse |
| **Dashboard** | Atualiza um dashboard AI/BI |
| **Pipeline** | Executa um Lakeflow Declarative Pipeline (ex-DLT) |
| **dbt** | Projeto dbt |
| **Run Job** | Dispara outro job (modularização) |
| **If/else condition** | Desvio condicional |
| **For each** | Repete uma task para cada item de uma lista |

### Compute para jobs
| Opção | Característica |
|---|---|
| **Serverless jobs** | Sem configurar cluster, inicia rápido, Databricks otimiza |
| **Job cluster** | Criado para o run e encerrado no fim. DBU mais barato que all-purpose. Pode ser **compartilhado entre tasks** do mesmo job |
| **All-purpose cluster** | Interativo, mais caro. Evite em produção |
| **SQL warehouse** | Para tasks SQL e dashboard |

## 2. Dependências e "Run if"

Cada task lista as tasks de que depende (`depends_on`). Sem dependência = roda em paralelo.

| Run if | A task roda quando as dependências… |
|---|---|
| **All succeeded** (padrão) | Todas deram certo |
| **At least one succeeded** | Pelo menos uma deu certo |
| **None failed** | Nenhuma falhou (pode haver *skipped*) |
| **All done** | Terminaram, com qualquer status (bom para limpeza/log) |
| **At least one failed** | Pelo menos uma falhou (bom para notificação/rollback) |
| **All failed** | Todas falharam |

## 3. Control flow

### Retries e timeout (por task)
- **Retries**: nº máximo de tentativas, intervalo mínimo entre elas e se deve repetir em caso de timeout.
- **Timeout**: cancela a task se passar do tempo.
- Em tasks de streaming contínuo, prefira retries ilimitados.

### Task values: passando dados entre tasks
```python
# Task "validar"
qtd = spark.table("vendas.bronze.pedidos").count()
dbutils.jobs.taskValues.set(key="qtd_linhas", value=qtd)

# Task seguinte (notebook)
qtd = dbutils.jobs.taskValues.get(taskKey="validar", key="qtd_linhas", default=0)
```
Em campos da UI/YAML: `{{tasks.validar.values.qtd_linhas}}`.
Outras referências dinâmicas: `{{job.run_id}}`, `{{job.start_time.iso_date}}`, `{{job.parameters.<nome>}}`.

### If/else condition
```
validar ──► [If/else: {{tasks.validar.values.qtd_linhas}} > 0]
                 ├── true  ──► processar_silver
                 └── false ──► notificar_sem_dados
```
- Operadores: `==`, `!=`, `>`, `>=`, `<`, `<=`.
- Compara task values, parâmetros do job ou valores dinâmicos.
- A task dependente escolhe o ramo (`true` ou `false`). O ramo não seguido fica como **Excluded**.

### For each
```
listar_lojas (seta task value "lojas" = ["SP","RJ","MG"])
   └── For each: inputs = {{tasks.listar_lojas.values.lojas}}, concorrência = 3
          └── task aninhada: processar_loja (parâmetro loja = {{input}})
```
- Recebe uma lista JSON fixa ou um task value.
- **Concorrência** define quantas iterações rodam em paralelo.
- Cada iteração pode ser reparada individualmente no *repair run*.

## 4. Triggers

| Trigger | Dispara quando | Detalhes |
|---|---|---|
| **Scheduled** | Num horário | Sintaxe simples ou **cron Quartz** (`0 0 6 * * ?` = todo dia às 6h), com fuso horário; pode ser pausado |
| **File arrival** | Chegam **arquivos novos** num caminho | Monitora uma **external location** ou **Volume** do UC. Opções: tempo mínimo entre disparos, esperar X segundos após a última mudança |
| **Table update** | Uma ou mais **tabelas do UC são atualizadas** | Pode exigir que *qualquer* ou *todas* as tabelas monitoradas mudem; também tem tempo mínimo e espera |
| **Continuous** | Assim que o run anterior termina | Para jobs que devem estar sempre rodando (streaming) |
| **Manual / API** | Alguém clica *Run now* ou chama a API/CLI | — |

### Tempo x dado: como escolher

| Situação | Trigger |
|---|---|
| Relatório precisa estar pronto às 7h, os dados chegam à meia-noite | **Scheduled** |
| Fornecedor envia arquivos em horários imprevisíveis e o processamento deve ser rápido, sem compute ocioso | **File arrival** |
| Job Gold deve rodar sempre que a Silver (de outro time/pipeline) for atualizada | **Table update** |
| Agendar "de 5 em 5 minutos para ver se chegou algo" | Evite: desperdiça compute. Use **file arrival** |
| Latência mínima, fluxo contínuo de eventos | **Continuous** (ou pipeline contínuo) |

> 🎯 Trigger por dado (file arrival / table update) **desacopla** jobs: o consumidor não precisa saber o horário do produtor.

## 5. Outras configurações que caem

- **Max concurrent runs**: limita execuções simultâneas do mesmo job (padrão 1). Com **queueing**, runs extras esperam na fila em vez de serem pulados.
- **Parâmetros do job** são passados para todas as tasks e podem ser sobrescritos em *Run now with different parameters*.
- **Notificações**: e-mail, Slack, Teams, PagerDuty, webhook.
- **Repair run**: reexecuta só tasks com falha e suas dependentes (ver D6).
- Jobs podem ser definidos pela UI, pela API/CLI ou por **bundles** (ver D5).

---

## ✅ Checklist do domínio
- [ ] Sei os tipos de task e qual compute usar para cada um
- [ ] Sei as opções de *Run if* e um caso de uso de *All done* e *At least one failed*
- [ ] Sei passar valores entre tasks com `dbutils.jobs.taskValues`
- [ ] Sei montar um If/else e um For each
- [ ] Sei os 3 triggers do guia e quando usar cada um
- [ ] Sei por que job cluster é mais barato que all-purpose

## 📚 Fontes oficiais
- [Lakeflow Jobs](https://learn.microsoft.com/azure/databricks/jobs/)
- [Control flow: If/else e Run if](https://learn.microsoft.com/azure/databricks/jobs/conditional-tasks)
- [For each](https://learn.microsoft.com/azure/databricks/jobs/for-each)
- [Triggers](https://learn.microsoft.com/azure/databricks/jobs/triggers)
- [Task values](https://learn.microsoft.com/azure/databricks/jobs/task-values)
