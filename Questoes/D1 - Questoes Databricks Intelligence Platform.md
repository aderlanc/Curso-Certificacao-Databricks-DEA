# Questões — Domínio 1: Databricks Intelligence Platform

> Questões **inéditas** escritas no estilo da prova, a partir dos objetivos do guia oficial (ago/2026). Não são questões vazadas.

---

### 1. Analistas executam consultas SQL ad hoc com picos de concorrência ao longo do dia. Eles querem startup rápido e o gestor quer controlar custo. Qual compute usar?

A) All-purpose cluster grande sempre ligado
B) SQL warehouse serverless com autoscaling
C) Job cluster
D) SQL warehouse classic sem auto-stop

<details><summary>Resposta</summary>

**B.** Serverless inicia em segundos, escala com a concorrência e para quando ocioso.
</details>

---

### 2. Em um workspace com compute clássico, onde rodam as VMs que processam os dados?

A) No control plane da Databricks
B) Na assinatura de nuvem do cliente (compute plane clássico)
C) No navegador do usuário
D) No metastore do Unity Catalog

<details><summary>Resposta</summary>

**B.** O control plane (UI, scheduler, APIs) fica na conta da Databricks. No compute serverless, as VMs ficam na conta da Databricks.
</details>

---

### 3. Uma equipe precisa de rollback rápido após cargas com erro, trilha de auditoria e uma única fonte de dados para BI e IA. Qual combinação da plataforma atende?

A) Arquivos CSV no storage com controle por pasta
B) Delta Lake (ACID e time travel) com Unity Catalog (governança, auditoria e linhagem)
C) Tabelas temporárias em clusters all-purpose
D) Parquet sem transaction log

<details><summary>Resposta</summary>

**B.** Delta Lake dá transações ACID e time travel/RESTORE para rollback, e o Unity Catalog centraliza permissões, auditoria e linhagem.
</details>

---

### 4. Qual é a principal vantagem de custo do job compute em relação ao all-purpose para cargas agendadas?

A) Não cobra DBU
B) DBU mais barato e o cluster existe só durante o run
C) Roda sempre com Photon desligado
D) Não exige storage

<details><summary>Resposta</summary>

**B.** Job compute é criado para o run e encerrado ao fim, com tarifa de DBU menor que a de all-purpose.
</details>
