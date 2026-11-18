# Domínio 1 — Databricks Intelligence Platform (6% · ~3 questões)

> Bem coberto pelo curso: `03/1-2` (arquitetura e clusters), `04` (Unity Catalog), `05/1-2` (Delta Lake), `10/2` (SQL warehouses).

**Objetivos do guia:**
1. Entender os componentes principais: arquitetura, **Delta Lake** e **Unity Catalog**.
2. Entender os serviços de **compute**: características, limitações e modelo de custo, e escolher o melhor para cada workload.

---

## 1. Arquitetura

| Plano | O que tem | Onde roda |
|---|---|---|
| **Control plane** | UI do workspace, notebooks, jobs scheduler, Unity Catalog, APIs | Conta da Databricks |
| **Compute plane clássico** | Clusters (VMs) que processam os dados | **Sua** assinatura de nuvem |
| **Compute plane serverless** | Compute gerenciado (notebooks, jobs, pipelines, SQL warehouses serverless) | Conta da Databricks, isolado por workspace |
| **Storage** | Dados das tabelas (ADLS, S3, GCS) | Sua conta de nuvem |

**Data Intelligence Platform** = lakehouse (Delta Lake + Unity Catalog) + IA aplicada à plataforma (assistente, otimizações automáticas como Predictive Optimization, busca semântica no catálogo).

## 2. Delta Lake em uma frase por recurso

- Arquivos **Parquet** + **transaction log** (`_delta_log/`) = transações **ACID**.
- **Time travel** (`VERSION AS OF`, `TIMESTAMP AS OF`, `RESTORE`), **schema enforcement** e **schema evolution**.
- Manutenção: `OPTIMIZE`, `VACUUM` (retenção padrão de 7 dias), Liquid Clustering.

## 3. Unity Catalog em uma frase por recurso

Governança única para dados e IA: namespace de 3 níveis, permissões SQL, linhagem, auditoria, descoberta e compartilhamento (detalhes no D7).

## 4. Compute: qual escolher?

| Compute | Para quê | Custo / limitação |
|---|---|---|
| **All-purpose (interativo)** | Desenvolvimento em notebooks, vários usuários | DBU mais caro; fica ligado até o auto-termination |
| **Job compute** | Jobs agendados em produção | DBU mais barato; criado e destruído a cada run |
| **Serverless (notebooks, jobs, pipelines)** | Começar rápido sem configurar nada | Sem gerenciar VMs; poucas configs Spark ajustáveis; paga só o uso |
| **SQL warehouse serverless** | SQL/BI com muitos usuários, startup em segundos | Recomendado para BI |
| **SQL warehouse pro / classic** | SQL na sua rede; classic sem recursos mais novos | Startup de minutos |
| **Instance pools** | Reduzir tempo de startup de clusters clássicos com VMs ociosas prontas | Paga a VM da nuvem enquanto ociosa, sem DBU |

Conceitos que aparecem:
- **Access mode**: *Standard* (antes "shared", vários usuários com isolamento) e *Dedicated* (antes "single user"). Ambos funcionam com Unity Catalog.
- **Autoscaling** (min/max workers) e **auto-termination** (desliga após X min ocioso).
- **Photon**: motor vetorizado em C++ que acelera SQL e DataFrames.
- **Cluster policies**: limitam o que usuários podem configurar (tipo de nó, tamanho), controlando custo.
- **Databricks Runtime (DBR)**: versão do Spark + bibliotecas. *ML* inclui bibliotecas de machine learning. Use versões **LTS** em produção.
- **DBU** (Databricks Unit) é a unidade de cobrança e varia pelo tipo de compute.

> 🎯 Analistas fazendo SQL ad hoc com picos de concorrência e startup rápido → **SQL warehouse serverless**. Job noturno em produção → **job compute** ou **serverless jobs**. Desenvolvimento em notebook → **all-purpose** ou **serverless notebook**.

---

## ✅ Checklist do domínio
- [ ] Sei o que fica no control plane e no compute plane (clássico e serverless)
- [ ] Sei os pilares do Delta Lake e do Unity Catalog
- [ ] Sei escolher entre all-purpose, job compute, serverless e SQL warehouse
- [ ] Sei para que servem pools, policies, Photon e auto-termination

## 📚 Fontes oficiais
- [Arquitetura do Azure Databricks](https://learn.microsoft.com/azure/databricks/getting-started/overview)
- [Tipos de compute](https://learn.microsoft.com/azure/databricks/compute/)
- [Serverless compute](https://learn.microsoft.com/azure/databricks/compute/serverless/)
