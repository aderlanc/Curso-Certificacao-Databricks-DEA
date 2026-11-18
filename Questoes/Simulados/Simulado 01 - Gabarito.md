# ✅ Simulado 01 — Gabarito comentado

> Corrija, anote o domínio de cada erro na tabela "Meus erros" da [trilha de estudo](../../00%20-%20Guia%20de%20Estudo/02%20-%20Trilha%20de%20Estudo.md) e revise o resumo correspondente em `Resumos/`.

## Gabarito rápido

| Questão | Resposta | Domínio | | Questão | Resposta | Domínio |
|---|---|---|---|---|---|---|
| **1** | C | D1 | | **24** | B | D3 |
| **2** | A | D1 | | **25** | A | D4 |
| **3** | B | D1 | | **26** | C | D4 |
| **4** | D | D2 | | **27** | B | D4 |
| **5** | C | D2 | | **28** | B | D4 |
| **6** | A | D2 | | **29** | B | D4 |
| **7** | C | D2 | | **30** | B | D4 |
| **8** | D | D2 | | **31** | B | D4 |
| **9** | B | D7 | | **32** | C | D5 |
| **10** | C | D2 | | **33** | C | D5 |
| **11** | C | D7 | | **34** | B | D5 |
| **12** | B | D2 | | **35** | B | D5 |
| **13** | C | D2 | | **36** | C | D5 |
| **14** | C | D3 | | **37** | A | D6 |
| **15** | D | D3 | | **38** | D | D6 |
| **16** | D | D3 | | **39** | B | D6 |
| **17** | B | D3 | | **40** | A | D6 |
| **18** | A | D3 | | **41** | C | D7 |
| **19** | D | D3 | | **42** | B | D7 |
| **20** | C | D3 | | **43** | C | D7 |
| **21** | A | D3 | | **44** | A | D7 |
| **22** | B | D3 | | **45** | D | D7 |
| **23** | D | D3 | | | | |

## Pontuação por domínio

| Domínio | Questões | Acertos |
|---|---|---|
| D1 — Databricks Intelligence Platform | 1, 2, 3 | ___ / 3 |
| D2 — Data Ingestion and Loading | 4, 5, 6, 7, 8, 10, 12, 13 | ___ / 8 |
| D3 — Data Transformation and Modeling | 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24 | ___ / 11 |
| D4 — Working with Lakeflow Jobs | 25, 26, 27, 28, 29, 30, 31 | ___ / 7 |
| D5 — Implementing CI/CD | 32, 33, 34, 35, 36 | ___ / 5 |
| D6 — Troubleshooting, Monitoring, Optimization | 37, 38, 39, 40 | ___ / 4 |
| D7 — Governance and Security | 9, 11, 41, 42, 43, 44, 45 | ___ / 7 |
| **Total** | 45 | ___ / 45 (meta ≥ 36) |

## Explicações

**1. C** (D1) — Notebooks, UI, scheduler e APIs ficam no **control plane** (conta da Databricks). Os clusters clássicos rodam na assinatura do cliente.

**2. A** (D1) — **Serverless para notebooks** inicia em segundos e não exige configurar cluster. Job cluster é para jobs, SQL warehouse não roda notebooks Python.

**3. B** (D1) — Pools mantêm **VMs ociosas prontas** e reduzem startup e autoscaling. Enquanto ociosas, você paga a VM da nuvem, mas não DBU.

**4. D** (D2) — `COPY_OPTIONS ('force' = 'true')` ignora o controle de idempotência e recarrega os arquivos.

**5. C** (D2) — Auto Loader é `readStream` + `format("cloudFiles")` + `cloudFiles.format`, com `cloudFiles.schemaLocation` para guardar o schema inferido.

**6. A** (D2) — Modo **`rescue`**: o schema não muda e o stream não falha; valores incompatíveis vão para `_rescued_data`.

**7. C** (D2) — Sem `inferColumnTypes`, o Auto Loader infere JSON e CSV como **STRING** (mais seguro contra mudanças de tipo).

**8. D** (D2) — Managed connectors do **Lakeflow Connect** para bancos fazem ingestão incremental gerenciada, com mínimo de manutenção.

**9. B** (D7) — **Lakehouse Federation** consulta a fonte no lugar, via connection e foreign catalog governados pelo UC, sem copiar dados.

**10. C** (D2) — Padrão recomendado: gravar a resposta bruta num **Volume** e deixar o **Auto Loader** carregar, com o notebook agendado por **Lakeflow Jobs**.

**11. C** (D7) — Credenciais ficam em **Databricks secrets** (`dbutils.secrets.get`), que também aparecem como `[REDACTED]` na saída.

**12. B** (D2) — Em coluna STRING com JSON, use **`:`** para navegar (`evento:pos.x`). A notação com ponto pura é para STRUCT.

**13. C** (D2) — **Auto Loader** é o padrão para ingestão incremental de grandes volumes (milhões de arquivos). **COPY INTO** atende cargas SQL de milhares de arquivos.

**14. C** (D3) — **`TRY_CAST`** retorna NULL quando a conversão falha. `CAST` faria a consulta falhar com ANSI mode.

**15. D** (D3) — **`left_anti`** devolve as linhas da esquerda **sem** correspondência. `left_semi` devolve as que **têm**.

**16. D** (D3) — Cross join é o produto cartesiano: 4 × 5 = **20**.

**17. B** (D3) — `UNION` em SQL **remove duplicatas**, então o 7 aparece **uma** vez. Com `UNION ALL` apareceria duas.

**18. A** (D3) — `-1` **desliga** o broadcast join automático. Broadcast explícito com `broadcast()` ou hint continua funcionando.

**19. D** (D3) — Depois de mudar um parâmetro, **meça de novo** (tempo e Spark UI) para confirmar o efeito, como pede o guia.

**20. C** (D3) — `summary()` inclui os percentis 25/50/75%. `describe()` mostra só count, mean, stddev, min e max.

**21. A** (D3) — **View** não guarda dados e reflete sempre a Silver. Bom para lógica leve sem custo de storage.

**22. B** (D3) — Leitura incremental de arquivos num pipeline = **streaming table** com `STREAM read_files(...)`.

**23. D** (D3) — `ON VIOLATION FAIL UPDATE` para o fluxo quando há violação. `DROP ROW` só descarta e o padrão só avisa.

**24. B** (D3) — **`MERGE INTO`** com `WHEN MATCHED THEN UPDATE` e `WHEN NOT MATCHED THEN INSERT` é o upsert do Delta.

**25. A** (D4) — Horário fixo e dados sempre prontos antes → **scheduled**, com o fuso correto.

**26. C** (D4) — **At least one failed** roda a task somente quando alguma dependência falha, ideal para alertas.

**27. B** (D4) — A **pipeline task** executa um pipeline declarativo existente. O código do pipeline não roda como notebook comum.

**28. B** (D4) — **For each** itera sobre a lista vinda do task value, com concorrência configurável.

**29. B** (D4) — Um **job cluster compartilhado** entre tasks evita subir um cluster por task.

**30. B** (D4) — A task Gold depende das duas, com **All succeeded** (o padrão).

**31. B** (D4) — `{{job.start_time.iso_date}}` é uma referência dinâmica de valor dos Lakeflow Jobs.

**32. C** (D5) — **Pull** pela UI do Git folder traz as mudanças remotas para a branch local.

**33. C** (D5) — Diferenças por ambiente ficam em **`targets`** (overrides de recursos ou de variáveis).

**34. B** (D5) — Ordem típica: **validate → deploy → run**.

**35. B** (D5) — Em CI/CD use **service principal com OAuth M2M**, com client id e secret guardados como secrets do CI.

**36. C** (D5) — Modo development prefixa recursos com **`[dev <usuário>]`** e **pausa schedules**.

**37. A** (D6) — Poucas tasks muito mais lentas e com muito mais shuffle read que a mediana = **data skew**.

**38. D** (D6) — A **matrix view** mostra cada task em cada execução, com cor do status e duração.

**39. B** (D6) — Predictive Optimization roda **OPTIMIZE, VACUUM e ANALYZE** automaticamente em **tabelas managed do UC**.

**40. A** (D6) — Conflito de bibliotecas se resolve **fixando versões compatíveis** e usando instalações no escopo do notebook.

**41. C** (D7) — Ler = `SELECT`; inserir, atualizar, excluir e fazer merge = **`MODIFY`**.

**42. B** (D7) — **`UNDROP TABLE`** recupera tabelas managed excluídas há até 7 dias.

**43. C** (D7) — **Row filter** controla quais **linhas** cada um vê; **column mask** controla o **valor** exibido de uma coluna.

**44. A** (D7) — **`is_account_group_member()`** checa grupos da conta, e é a função usada nas regras de filtro e máscara.

**45. D** (D7) — `SELECT` no schema é **herdado** por todas as tabelas atuais e futuras, o que fere o menor privilégio.
