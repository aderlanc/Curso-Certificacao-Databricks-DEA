# 📚 Curso de Preparação - Databricks Data Engineer Associate (DEA)

Material de estudo para a certificação **Databricks Certified Data Engineer Associate**, com foco em ambientes **Azure Databricks**.

---

## 🎯 Objetivo

Este repositório contém documentação, exemplos práticos e anotações organizadas por tópico para preparação completa para o exame de certificação DEA.

A certificação valida a capacidade de utilizar a **Databricks Data Intelligence Platform** para realizar tarefas de engenharia de dados, incluindo compreensão da arquitectura, execução de ETL com Apache Spark SQL e PySpark, e orquestração de workloads com Databricks Workflows.

---

## 📋 Conteúdo Programático

| Pasta | Tópico | Descrição |
|-------|--------|-----------|
| `00` | **Guia de Estudo** | Trilha, mapa domínio x aula, método de preparação com IA, laboratórios e diário (comece aqui) |
| `01` | Introdução ao Curso | Boas vindas e introdução |
| `02` | Criando Conta Free na Azure |Criando conta Azure com bonus de $200 |
| `03` | Componentes Databricks | Arquitectura, clusters, workspaces |
| `04` | Introdução ao Unity Catalog | Governança, metastore, namespaces |
| `05` | Databricks Lakehouse | Delta Lake, arquitectura medallion |
| `06` | ETL e Processamento Spark | Transformações, DataFrames, SQL |
| `07` | Spark Structured Streaming | Streaming, triggers, checkpoints |
| `08` | Pipelines Declarativos - DLT | Delta Live Tables, expectations |
| `09` | Databricks Jobs | Orquestração, triggers, task values |
| `10` | Databricks SQL | Warehouses, dashboards, alertas |
| `11` | Governança de Dados | Permissões, GRANT/REVOKE, security |
| `12` | Simulado Certificação | Simulado de Perguntas Certificação |
| `13` | Condideração Final| Encerramento do treinamento |
| `Resumos/` | Resumos por domínio | Um resumo por domínio do exame, incluindo o que o curso não cobre |
| `Questoes/` | Questões de prática | Questões inéditas no estilo da prova, com gabarito comentado |
---

## 🎓 Sobre o Exame

| Aspecto | Detalhe |
|---------|---------|
| **Tipo** | Proctored (supervisionado) |
| **Questões** | 45 questões pontuadas |
| **Duração** | 90 minutos |
| **Formato** | Múltipla escolha |
| **Taxa de Registo** | $200 USD |
| **Idiomas** | English, 日本語, Português BR, 한국어 |
| **Modalidade** | Online ou centro de testes |
| **Pré-requisitos** | Python e SQL nível básico |
| **Experiência Recomendada** | 6+ meses hands-on |
| **Validade** | 2 anos |
| **Recertificação** | Refazer exame na versão actual |

---

## 📊 Distribuição de Tópicos no Exame (versão de 04/05/2026, guia de ago/2026)

| Domínio | Peso |
|---------|------|
| 1. Databricks Intelligence Platform | 6% |
| 2. Data Ingestion and Loading | 21% |
| 3. Data Transformation and Modeling | 22% |
| 4. Working with Lakeflow Jobs | 16% |
| 5. Implementing CI/CD | 10% |
| 6. Troubleshooting, Monitoring, and Optimization | 10% |
| 7. Governance and Security | 15% |

```
Data Transformation and Modeling       ██████████████████████  22%
Data Ingestion and Loading             █████████████████████   21%
Working with Lakeflow Jobs             ████████████████        16%
Governance and Security                ███████████████         15%
Implementing CI/CD                     ██████████              10%
Troubleshooting, Monitoring, Optim.    ██████████              10%
Databricks Intelligence Platform       ██████                   6%
```

> O exame tem 45 questões pontuadas e pode ter até 10 não pontuadas, em 90 minutos.
> Veja em [`00 - Guia de Estudo/01 - Mapa Dominios x Aulas.md`](00%20-%20Guia%20de%20Estudo/01%20-%20Mapa%20Dominios%20x%20Aulas.md) o que o curso cobre em cada domínio.

---

## 🛠️ Estrutura do Material

Cada pasta contém:

- **Documentação em Markdown** — Resumos focados em pontos de certificação
- **Notebooks** — Exemplos práticos para execução no Databricks
- **Cenários de Exame** — Questões típicas e como responder

---

## 📖 Como Usar

O estudo segue o **método oficial da Databricks para preparação com IA** ("AI Prep Guide: Any Databricks Certification"), adaptado à DEA:

1. **Prepare o terreno e a IA** — [`03 - Preparacao com IA`](00%20-%20Guia%20de%20Estudo/03%20-%20Preparacao%20com%20IA.md): prompt de abertura e tabela de nomes renomeados (DLT → Lakeflow Spark Declarative Pipelines etc.)
2. **Diagnostique antes de estudar** — faça as questões de todos os domínios em `Questoes/` e registre no [`05 - Diario de Estudo`](00%20-%20Guia%20de%20Estudo/05%20-%20Diario%20de%20Estudo.md)
3. **Aprofunde os domínios fracos** — aulas (pelo [mapa](00%20-%20Guia%20de%20Estudo/01%20-%20Mapa%20Dominios%20x%20Aulas.md)) + `Resumos/`
4. **Pratique no workspace** — os 9 [`04 - Laboratorios Hands-on`](00%20-%20Guia%20de%20Estudo/04%20-%20Laboratorios%20Hands-on.md) (só ler não aprova)
5. **Simule e repare** — `Questoes/Simulados/` e o prompt de reparo em cada erro

O calendário de 6 semanas está na [`02 - Trilha de Estudo`](00%20-%20Guia%20de%20Estudo/02%20-%20Trilha%20de%20Estudo.md).

> ⚠️ IA erra sobre produtos Databricks. Confirme tudo no Exam Guide e em [docs.databricks.com](https://docs.databricks.com/): resposta sem link da documentação é palpite.

---

## 📝 Passos para Preparação (Recomendado pela Databricks)

1. ✅ Revisar o [Exam Guide oficial](https://www.databricks.com/sites/default/files/2026-10/Databricks-Certified-Data-Engineer-Associate-Exam-Guide-August-2026.pdf)
2. ✅ Completar treino relacionado
3. ✅ Registar para o exame
4. ✅ Verificar [requisitos técnicos](https://kryterion.my.site.com/support/s/article/Online-Testing-Requirements?language=en_US) para exame online
5. ✅ Executar [system check](https://www.kryterion.com/systemcheck/)
6. ✅ Identificar gaps e estudar
7. ✅ Fazer o exame!

> **Nota:** O código no exame é fornecido em SQL quando possível. Nos demais casos, será em Python.

---

## ✅ Checklist de Preparação

O checklist detalhado, semana a semana, está em [`00 - Guia de Estudo/02 - Trilha de Estudo.md`](00%20-%20Guia%20de%20Estudo/02%20-%20Trilha%20de%20Estudo.md).

### 1. Databricks Intelligence Platform (6%)
- [ ] Componentes: arquitetura, Delta Lake, Unity Catalog
- [ ] Escolher o compute certo (serverless, classic, job cluster, SQL warehouse) e seus custos

### 2. Data Ingestion and Loading (21%)
- [ ] COPY INTO e Auto Loader (schema evolution, directory listing x file notification)
- [ ] Lakeflow Connect (conectores standard e managed), JDBC/REST
- [ ] Dados semi-estruturados (JSON, nested, arrays)

### 3. Data Transformation and Modeling (22%)
- [ ] Limpeza Bronze → Silver, joins (broadcast, cross, union), deduplicação, agregações
- [ ] Parâmetros de tuning do Spark
- [ ] Objetos Gold: views, materialized views, streaming tables; data quality

### 4. Working with Lakeflow Jobs (16%)
- [ ] Tasks e dependências no DAG, retries, If/else, For each
- [ ] Triggers: scheduled, file arrival, table update

### 5. Implementing CI/CD (10%)
- [ ] Git folders: branch, commit, push, pull request
- [ ] Declarative Automation Bundles (ex-DABs): variáveis, targets, deploy
- [ ] Databricks CLI: `bundle validate / deploy / run`

### 6. Troubleshooting, Monitoring, and Optimization (10%)
- [ ] Run history, matrix view, repair run
- [ ] Spark UI: skew, shuffle, spill
- [ ] Liquid Clustering e Predictive Optimization
- [ ] Falhas de cluster, conflitos de bibliotecas, OOM

### 7. Governance and Security (15%)
- [ ] Managed x External (incluindo conversão)
- [ ] GRANT / REVOKE / DENY para usuários, grupos e service principals
- [ ] Row filters, column masks e ABAC

---

## 📚 Recursos Oficiais

- [Página da Certificação](https://www.databricks.com/learn/certification/data-engineer-associate)
- [Exam Guide (PDF)](https://www.databricks.com/sites/default/files/2026-10/Databricks-Certified-Data-Engineer-Associate-Exam-Guide-August-2026.pdf)
- [Registo para Exame](http://webassessor.com/databricks)
- [Databricks Academy](https://www.databricks.com/learn/training/login)
- [Documentação oficial Databricks](https://docs.databricks.com/) (fonte principal para validar respostas da IA)
- [Databricks Free Edition](https://www.databricks.com/learn/free-edition) (workspace gratuito para os laboratórios)
- [Documentação Azure Databricks](https://learn.microsoft.com/pt-pt/azure/databricks/)
- [FAQ da Certificação](https://www.databricks.com/learn/certification/faq)

---

## 📁 Pasta Arquivos

Contém ficheiros de dados utilizados nos notebooks práticos (CSV, JSON, Parquet) para simulação de cenários reais de ingestão e transformação.

---

## ⚠️ Notas Importantes

- **Conteúdo não pontuado:** O exame pode incluir questões para fins estatísticos que não afetam a nota
- **Materiais permitidos:** Nenhum (closed book)
- **Código:** SQL é preferido, Python quando necessário

---

**Boa sorte na certificação!** 🚀