# Questões — Domínio 7: Governance and Security

> Questões **inéditas** escritas no estilo da prova, a partir dos objetivos do guia oficial (ago/2026). Não são questões vazadas.

---

### 1. Um engenheiro executa `DROP TABLE vendas.bronze.logs`. A tabela foi criada com `LOCATION 'abfss://raw@conta.dfs.core.windows.net/logs'`. O que acontece?

A) Metadados e arquivos são removidos
B) Apenas os metadados são removidos; os arquivos permanecem no storage
C) O comando falha, pois tabelas external não podem ser excluídas
D) Os arquivos são movidos para o local gerenciado do catálogo

<details><summary>Resposta</summary>

**B.** Tabela com `LOCATION` é **external**: o DROP remove só o registro no Unity Catalog. Em tabela managed, os dados também seriam removidos.
</details>

---

### 2. O time quer que uma tabela external passe a ter Predictive Optimization e ciclo de vida gerenciado pelo Unity Catalog, sem recriar a tabela manualmente. Qual comando usar?

A) `ALTER TABLE t SET MANAGED`
B) `CONVERT TO DELTA t`
C) `ALTER TABLE t SET TBLPROPERTIES ('managed' = 'true')`
D) `CREATE OR REPLACE TABLE t DEEP CLONE t`

<details><summary>Resposta</summary>

**A.** `SET MANAGED` converte external → managed. `CONVERT TO DELTA` transforma Parquet em Delta, que é outra coisa. A conversão pode ser desfeita com `UNSET MANAGED` em até 14 dias.
</details>

---

### 3. O grupo `analistas` precisa ler todas as tabelas do schema `vendas.gold`, inclusive as que forem criadas no futuro. Ele já tem `USE CATALOG` em `vendas`. Qual é o menor conjunto de comandos?

A) `GRANT SELECT ON TABLE` para cada tabela atual
B) `GRANT USE SCHEMA ON SCHEMA vendas.gold` e `GRANT SELECT ON SCHEMA vendas.gold`
C) `GRANT ALL PRIVILEGES ON CATALOG vendas`
D) `GRANT SELECT ON SCHEMA vendas.gold` apenas

<details><summary>Resposta</summary>

**B.** Sem `USE SCHEMA` o grupo não "entra" no schema. `SELECT` no schema é herdado por todas as tabelas, presentes e futuras. ALL PRIVILEGES daria acesso demais.
</details>

---

### 4. Um usuário tem `SELECT` na tabela `vendas.gold.kpis`, mas recebe erro de permissão ao consultá-la. Qual é a causa mais provável?

A) Falta `MODIFY` na tabela
B) Falta `USE CATALOG` em `vendas` e/ou `USE SCHEMA` em `vendas.gold`
C) A tabela é managed
D) O usuário precisa ser owner da tabela

<details><summary>Resposta</summary>

**B.** Ler uma tabela exige `USE CATALOG` + `USE SCHEMA` + `SELECT`.
</details>

---

### 5. Um job em produção precisa gravar em tabelas Silver sem depender da conta de nenhum funcionário. Qual identidade deve receber as permissões e ser usada como `run_as`?

A) O usuário do engenheiro líder
B) Um service principal
C) O grupo `account users`
D) O owner do metastore

<details><summary>Resposta</summary>

**B.** Service principals são identidades para automação: não saem da empresa nem têm senha pessoal, e recebem GRANT como usuários e grupos.
</details>

---

### 6. Apenas membros do grupo `finance_team` devem ver o número completo do cartão na coluna `cartao`. Os demais devem ver só os 4 últimos dígitos, consultando a **mesma tabela**. Qual é a solução?

A) Criar uma view separada para o financeiro
B) Criar uma função SQL com `is_account_group_member('finance_team')` e aplicá-la com `ALTER TABLE ... ALTER COLUMN cartao SET MASK`
C) Revogar `SELECT` de quem não é do financeiro
D) Criptografar a coluna com AES e dar a chave ao financeiro

<details><summary>Resposta</summary>

**B.** Column mask aplica a regra na própria tabela, por grupo. A view separada exigiria outro objeto, e revogar `SELECT` tiraria todo o acesso.
</details>

---

### 7. Vendedores devem ver apenas os pedidos da sua região, e gerentes devem ver todos, na mesma tabela `pedidos`. Qual recurso usar?

A) Column mask na coluna `regiao`
B) Row filter com uma função que recebe `regiao` e retorna `TRUE` para gerentes ou para a região do usuário
C) Particionar a tabela por região
D) Criar uma tabela por região

<details><summary>Resposta</summary>

**B.** Restringir **linhas** é row filter. Column mask altera **valores** de uma coluna, não esconde linhas.
</details>

---

### 8. A empresa tem mais de 500 tabelas com colunas de CPF, e novas tabelas são criadas toda semana. Compliance quer mascarar CPF para todos exceto o grupo `compliance`, sem configurar tabela por tabela. Qual abordagem atende?

A) Aplicar `SET MASK` em cada coluna de cada tabela
B) Marcar as colunas com uma governed tag (ex.: `pii = cpf`) e criar uma política ABAC no catálogo que aplica a máscara às colunas com essa tag
C) Criar dynamic views para todas as tabelas
D) Mover as tabelas para um catálogo sem permissão

<details><summary>Resposta</summary>

**B.** ABAC aplica filtros e máscaras de forma centralizada com base em tags, e cobre automaticamente tabelas novas no escopo da política.
</details>

---

### 9. Um analista consegue ver que a tabela `rh.gold.salarios` existe no Catalog Explorer, mas não consegue consultar os dados. Qual privilégio explica isso?

A) `SELECT`
B) `BROWSE`
C) `MODIFY`
D) `EXECUTE`

<details><summary>Resposta</summary>

**B.** `BROWSE` permite descobrir objetos e ver metadados, sem ler os dados.
</details>
