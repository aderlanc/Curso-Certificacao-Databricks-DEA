# 🧭 Trilha de Estudo — DEA (exame de maio/2026)

Ritmo sugerido: **6 semanas, ~1h por dia**, em 3 sprints de 2 semanas, seguindo o [método oficial de preparação com IA](03%20-%20Preparacao%20com%20IA.md) da Databricks:

| Fase do método | Sprint | Foco |
|---|---|---|
| Primeiro quarto: **diagnosticar** | Sprint 1 (semanas 1-2) | Diagnóstico de todos os domínios, depois base da plataforma |
| Metade do meio: **aprofundar + laboratórios** | Sprint 2 (semanas 3-4) e início do Sprint 3 | Domínios fracos primeiro, aulas, resumos e [laboratórios](04%20-%20Laboratorios%20Hands-on.md) |
| Último quarto: **simulados** | Final do Sprint 3 (semana 6) | Simulados completos e reparo dos erros |

**Data da prova:** ____/____/______. Com mais de 6 semanas, adicione sprints extras na fase do meio. Com menos, encurte a fase do meio, nunca o diagnóstico nem os laboratórios.

## Como usar cada etapa (o loop de 5 etapas)

1. **Orientar:** leia os objetivos no topo de `Resumos/Dx`.
2. **Diagnosticar:** faça `Questoes/Dx` **sem consultar** e registre no [diário](05%20-%20Diario%20de%20Estudo.md).
3. **Aprofundar:** assista às aulas do [mapa](01%20-%20Mapa%20Dominios%20x%20Aulas.md), leia o resumo e use o prompt de aprofundamento só nos objetivos que errou.
4. **Praticar:** faça os laboratórios do domínio e, no fim, os simulados.
5. **Reparar:** use o prompt de reparo em cada questão errada e anote em "Meus erros".

Em toda sessão com IA, cole o **prompt de abertura** e a **tabela de nomes renomeados** do [Passo 2 e 3](03%20-%20Preparacao%20com%20IA.md).

---

## Sprint 1 — Diagnóstico e base (semanas 1-2)

### Semana 1 — Preparar e diagnosticar tudo
- [ ] Passo 1 do método: agendar a prova, baixar o Exam Guide, criar conta na Free Edition e no Databricks Academy
- [ ] Rodar o prompt de nomes renomeados (Passo 3) e comparar com a tabela pronta
- [ ] Diagnóstico de **todos** os domínios: `Questoes/D1` a `D7` + 10 questões da IA por domínio (~1 domínio por dia)
- [ ] Preencher a tabela de diagnóstico do [diário](05%20-%20Diario%20de%20Estudo.md) e escolher os **2-3 domínios mais fracos**

### Semana 2 — Plataforma e Lakehouse (D1 6% + parte de D3, D6 e D7)
- [ ] Aulas `03 - Componentes Databricks` (arquitetura, compute, notebooks, magic commands, Git folders)
- [ ] Aula `04 - Introdução ao Unity Catalog`
- [ ] Aulas `05 - Databricks Lakehouse` (Delta Lake, managed x external, time travel, OPTIMIZE, Liquid Clustering, CTAS, constraints, clone, views)
- [ ] Resumo D1 · Laboratório **L8** (Liquid Clustering)

## Sprint 2 — Aprofundar os domínios de maior peso (semanas 3-4)

> Se o diagnóstico apontou D5, D6 ou D7 como fracos, traga a semana 5 para cá e empurre esta parte para depois.

### Semana 3 — Ingestão e transformação (D2 21% + D3 22%)
- [ ] Aulas `06 - ETL e Processamento Spark` (medalhão, JSON/CSV/TSV/XML, arrays, UDF)
- [ ] Aulas `07 - Spark Structured Streaming` (COPY INTO, Auto Loader)
- [ ] Resumos D2 e D3 · Laboratórios **L1**, **L2** e **L3**
- [ ] Refazer as questões D2 e D3 que errou no diagnóstico

### Semana 4 — Pipelines declarativos e Jobs (D3 + D4 16%)
- [ ] Aulas `08 - Pipelines Declarativos - DLT` (streaming table x MV, expectations, CDC, modos)
- [ ] Aulas `09 - Databricks Jobs` (tasks, dependências, triggers)
- [ ] Resumo D4 · Laboratórios **L4** e **L5**
- [ ] Refazer as questões D4 que errou no diagnóstico

## Sprint 3 — Completar e simular (semanas 5-6)

### Semana 5 — CI/CD, troubleshooting, SQL e governança (D5 10% + D6 10% + D7 15%)
- [ ] Resumo D5 (Bundles + CLI) · Laboratório **L6** ⚠️ *o curso não cobre, estude pelo resumo*
- [ ] Resumo D6 (Spark UI, OOM, run history) · Laboratório **L7** ⚠️
- [ ] Aulas `10 - Databricks SQL` e `11 - Governança de Dados`
- [ ] Resumo D7 (row filters, column masks, ABAC) · Laboratório **L9**
- [ ] Todos os 9 laboratórios concluídos? O que faltar vira prioridade

### Semana 6 — Simulados e reparo
- [ ] Aula `12 - Simulado Certificação`
- [ ] [Simulado 01](../Questoes/Simulados/Simulado%2001%20-%20Questoes.md) (45 questões, 90 min cronometrados) e [gabarito](../Questoes/Simulados/Simulado%2001%20-%20Gabarito.md)
- [ ] Um simulado gerado pela IA (prompt da etapa 4) · Simulado 02 (em breve)
- [ ] Prompt de reparo em todos os erros, até zerar "Meus erros"
- [ ] Reler o [Exam Guide oficial](https://www.databricks.com/sites/default/files/2026-10/Databricks-Certified-Data-Engineer-Associate-Exam-Guide-August-2026.pdf) (versão atual!) e o [mapa](01%20-%20Mapa%20Dominios%20x%20Aulas.md)

### ✅ Critério para ir à prova
Simulados não são uma medida calibrada de prontidão. Use como sinal:
- [ ] **Todos** os objetivos do Exam Guide estudados (mapa)
- [ ] **Todos** os laboratórios feitos sem consultar o passo a passo
- [ ] Simulados acima de **80%** como confirmação, não como único critério

---

## 🎯 Prioridade pelo peso

| Domínio | Peso | ~Questões (de 45) |
|---|---|---|
| 3. Data Transformation and Modeling | 22% | 10 |
| 2. Data Ingestion and Loading | 21% | 9-10 |
| 4. Working with Lakeflow Jobs | 16% | 7 |
| 7. Governance and Security | 15% | 7 |
| 5. Implementing CI/CD | 10% | 4-5 |
| 6. Troubleshooting, Monitoring, Optimization | 10% | 4-5 |
| 1. Databricks Intelligence Platform | 6% | 3 |

---

## 📝 Meus erros, termos e simulados

Use o [Diário de Estudo](05%20-%20Diario%20de%20Estudo.md).
