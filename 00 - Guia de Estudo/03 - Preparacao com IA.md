# 🤖 Preparação com IA — método oficial da Databricks aplicado à DEA

> Base: **"AI Prep Guide: Any Databricks Certification"** (guia oficial da Databricks sobre como usar um chatbot como tutor).
> Aqui o método foi adaptado à **Data Engineer Associate**, ao **Exam Guide de agosto/2026** e aos arquivos deste repositório.
>
> ⚠️ A própria Databricks avisa: seguir o guia **não garante aprovação**, e a IA erra sobre produtos Databricks.
> Toda afirmação importante deve ser confirmada no [Exam Guide](https://www.databricks.com/sites/default/files/2026-10/Databricks-Certified-Data-Engineer-Associate-Exam-Guide-August-2026.pdf) e em [docs.databricks.com](https://docs.databricks.com/).

Arquivos relacionados: [`01 - Mapa`](01%20-%20Mapa%20Dominios%20x%20Aulas.md) · [`02 - Trilha`](02%20-%20Trilha%20de%20Estudo.md) · [`04 - Laboratórios`](04%20-%20Laboratorios%20Hands-on.md) · [`05 - Diário de Estudo`](05%20-%20Diario%20de%20Estudo.md)

---

## Passo 1 — Preparar o terreno (uma vez)

- [ ] **Agendar a prova** em [webassessor.com/databricks](https://webassessor.com/databricks). Uma data real é o que organiza os sprints do Passo 6.
- [ ] **Baixar o Exam Guide** da [página da certificação](https://www.databricks.com/learn/certification/data-engineer-associate). Conferir de novo **2 semanas antes da prova**, pois a Databricks atualiza sem aviso.
- [ ] **Escolher uma IA** (Claude, ChatGPT, Gemini, Copilot, Genie Code no workspace). O plano gratuito basta.
- [ ] **Ter um workspace**: [Free Edition](https://www.databricks.com/learn/free-edition) ou trial (Azure Databricks também serve). Ler não basta para passar.
- [ ] **Cadastrar-se no [Databricks Academy](https://www.databricks.com/learn/training/login)**: o learning path "Data Engineer Associate" é gratuito.
- [ ] **Abrir o diário de estudo**: use [`05 - Diario de Estudo.md`](05%20-%20Diario%20de%20Estudo.md) deste repositório.

---

## Passo 2 — Prompt de abertura (cole em TODA sessão, junto com o PDF do Exam Guide)

```text
Estou me preparando para a certificação Databricks Certified Data Engineer Associate,
descrita no Exam Guide em anexo (versão de agosto/2026, exame de 04/05/2026).
Responda em português do Brasil, mas mantenha os nomes de produtos, comandos e opções
em inglês, como aparecem na prova.

Regras:
1. Use apenas nomes e comportamentos ATUAIS dos produtos Databricks. Antes de começar,
   leia a tabela de nomes renomeados que vou colar abaixo e use sempre o nome atual.
2. Não invente objetivos que não estão no Exam Guide.
3. Baseie tudo em fontes oficiais (docs.databricks.com e Databricks Academy), não em blogs,
   tutoriais de terceiros ou fóruns.
4. Se puder navegar na web, pesquise em docs.databricks.com e cite a URL da página da
   documentação ao lado de cada afirmação importante.
5. Se não conseguir confirmar algo na documentação oficial, ou não puder navegar, diga isso
   explicitamente em vez de adivinhar, e me diga onde na documentação devo verificar.
6. Se não puder ler PDFs, me avise que eu colo a seção "Sections and Objectives" como texto.

[COLE AQUI A TABELA DE NOMES RENOMEADOS DO PASSO 3]
```

---

## Passo 3 — Armadilhas de "produto renomeado" (faça uma vez, cole sempre)

A IA foi treinada antes das mudanças recentes da plataforma e usa nomes antigos com confiança.
Rode este prompt **uma vez** e compare com a tabela abaixo:

```text
Liste todo nome de produto Databricks do Exam Guide em anexo que foi renomeado nos últimos
2 anos, ou sobre o qual você tem dúvida. Para cada um: o(s) nome(s) antigo(s) que você poderia
usar por engano, o nome atual e uma frase sobre o que faz. Se não tiver certeza se o nome mudou,
diga isso. Eu vou verificar em docs.databricks.com.
```

**Tabela já levantada para a DEA** (cole no prompt de abertura). Os itens marcados 🔎 ainda precisam ser confirmados por você na documentação.

| Nome antigo (IA e curso usam) | Nome atual (prova usa) | O que é |
|---|---|---|
| Delta Live Tables (DLT), Lakeflow Declarative Pipelines | **Lakeflow Spark Declarative Pipelines** | Pipelines declarativos com streaming tables, materialized views e expectations |
| `APPLY CHANGES INTO` / `apply_changes()` | **`AUTO CDC INTO`** / `create_auto_cdc_flow()` | CDC (SCD tipo 1 e 2) dentro dos pipelines. A sintaxe antiga ainda funciona |
| Databricks Workflows, Databricks Jobs | **Lakeflow Jobs** | Orquestração de tasks em DAG, triggers, retries, If/else, For each |
| Databricks Asset Bundles (DABs) | **Declarative Automation Bundles** | Projeto em YAML (jobs, pipelines, variáveis, targets) implantado pela CLI |
| Databricks Repos | **Git folders** | Pastas Git no workspace: branch, commit, push, pull |
| `dbx`, CLI legada em Python (`databricks-cli` v0.1x) | **Databricks CLI** nova (v0.2xx, comando `databricks bundle ...`) | CLI oficial atual, usada para bundles |
| Partner Connect / conectores de terceiros como única opção | **Lakeflow Connect** (conectores *standard* e *managed*) | Ingestão de bancos e SaaS para tabelas do UC |
| SQL endpoints | **SQL warehouses** | Compute para SQL e dashboards |
| Lakehouse Platform | **Data Intelligence Platform** | Nome da plataforma usado no Domínio 1 |
| DBFS mounts (`/mnt/...`), DBFS root para dados | **Unity Catalog volumes** e **external locations** | Acesso governado a arquivos. Mounts e DBFS root são padrão legado |
| `hive_metastore` como padrão | **Unity Catalog** (`catalog.schema.tabela`) | Governança. O Hive metastore é legado |
| Partitioning + `ZORDER` como recomendação padrão | **Liquid Clustering** (`CLUSTER BY`, `CLUSTER BY AUTO`) | Layout de dados recomendado para tabelas novas |
| `OPTIMIZE`/`VACUUM` agendados à mão | **Predictive Optimization** | Manutenção automática em tabelas managed do UC |
| Databricks Assistant 🔎 | **Genie Code** 🔎 | Assistente de IA dentro do workspace (o guia oficial cita "Genie Code") |
| Dashboards legados / Databricks SQL dashboards 🔎 | **AI/BI Dashboards** 🔎 | Dashboards que podem ser tasks de um Job |

> Se a IA usar um nome da coluna da esquerda, corrija na hora: "use o nome atual da tabela".

---

## Passo 4 — O loop de 5 etapas, um domínio por vez

Para cada domínio do exame (D1 a D7), siga as etapas na ordem. O repositório já tem material para as etapas 1, 2 e 4. A IA entra para aprofundar e para gerar questões novas.

| Etapa | O que fazer | Material do repositório | Prompt para a IA |
|---|---|---|---|
| **1. Orientar** | Entender cada objetivo em uma frase | Lista de objetivos no topo de `Resumos/Dx` | *"Faça um resumo em linguagem simples dos objetivos da seção [DOMÍNIO] do Exam Guide que enviei. Uma frase por objetivo."* |
| **2. Diagnosticar** | Fazer questões **antes** de estudar o domínio | `Questoes/Dx` (sem abrir as respostas) | *"Me faça 10 questões de múltipla escolha sobre [DOMÍNIO], no estilo de uma prova de certificação Databricks real. Não mostre as respostas até eu responder."* |
| **3. Aprofundar** | Estudar só os objetivos que você errou | `Resumos/Dx` + aulas indicadas no [mapa](01%20-%20Mapa%20Dominios%20x%20Aulas.md) | Prompt de aprofundamento abaixo |
| **4. Praticar** | Simulado completo, cronometrado | [`Questoes/Simulados/`](../Questoes/Simulados/) | *"Gere um simulado completo com o número de questões pontuadas do Exam Guide (45), distribuídas pelos domínios nas proporções do guia. Não revele as respostas até eu enviar todas."* |
| **5. Reparar** | Corrigir o raciocínio de cada erro | Tabela "Meus erros" do [diário](05%20-%20Diario%20de%20Estudo.md) | Prompt de reparo abaixo |

**Prompt de aprofundamento (etapa 3)** — copie o objetivo **literalmente** do Exam Guide:

```text
Me ensine "[OBJETIVO QUE ERREI, COPIADO LITERALMENTE DO EXAM GUIDE]", usando como fonte apenas
a documentação oficial (docs.databricks.com) e o Databricks Academy, não blogs ou fóruns.
Se puder navegar, cite a URL exata da página ao lado de cada afirmação; se não conseguir
verificar algo na documentação oficial, diga isso em vez de adivinhar. Inclua: conceito central,
como funciona no Databricks, quando usar e quando usar a alternativa, erros comuns e um exemplo
de código que eu possa rodar na Free Edition.
```

**Prompt de reparo (etapa 5)**:

```text
Estas são as questões que errei: [COLE A QUESTÃO, A MINHA RESPOSTA E A CORRETA].
Para cada uma: explique o meu equívoco, explique por que a resposta certa está certa e me dê
uma questão nova que teste o mesmo conceito de outro jeito.
```

### Ordem dos domínios no loop

A ordem segue a [trilha](02%20-%20Trilha%20de%20Estudo.md), mas na etapa de diagnóstico faça **todos** os domínios primeiro (Sprint 1) e só depois aprofunde, começando pelos 2 ou 3 mais fracos, ponderados pelo peso:

| Domínio | Peso | Resumo | Questões |
|---|---|---|---|
| D3 Data Transformation and Modeling | 22% | [Resumo](../Resumos/D3%20-%20Data%20Transformation%20and%20Modeling.md) | [Questões](../Questoes/D3%20-%20Questoes%20Data%20Transformation%20and%20Modeling.md) |
| D2 Data Ingestion and Loading | 21% | [Resumo](../Resumos/D2%20-%20Data%20Ingestion%20and%20Loading.md) | [Questões](../Questoes/D2%20-%20Questoes%20Data%20Ingestion%20and%20Loading.md) |
| D4 Working with Lakeflow Jobs | 16% | [Resumo](../Resumos/D4%20-%20Working%20with%20Lakeflow%20Jobs.md) | [Questões](../Questoes/D4%20-%20Questoes%20Working%20with%20Lakeflow%20Jobs.md) |
| D7 Governance and Security | 15% | [Resumo](../Resumos/D7%20-%20Governance%20and%20Security.md) | [Questões](../Questoes/D7%20-%20Questoes%20Governance%20and%20Security.md) |
| D5 Implementing CI/CD | 10% | [Resumo](../Resumos/D5%20-%20Implementing%20CI-CD.md) | [Questões](../Questoes/D5%20-%20Questoes%20CI-CD.md) |
| D6 Troubleshooting, Monitoring, Optimization | 10% | [Resumo](../Resumos/D6%20-%20Troubleshooting%20Monitoring%20Optimization.md) | [Questões](../Questoes/D6%20-%20Questoes%20Troubleshooting%20Monitoring%20Optimization.md) |
| D1 Databricks Intelligence Platform | 6% | [Resumo](../Resumos/D1%20-%20Databricks%20Intelligence%20Platform.md) | [Questões](../Questoes/D1%20-%20Questoes%20Databricks%20Intelligence%20Platform.md) |

---

## Passo 5 — Mínimo hands-on (o passo mais importante)

A prova cobra habilidade prática, não só memória. A lista de laboratórios desta certificação já está pronta em [`04 - Laboratorios Hands-on.md`](04%20-%20Laboratorios%20Hands-on.md): 9 tarefas de até 30 minutos cada, mapeadas aos objetivos do Exam Guide.

Para conferir se falta algo, rode também:

```text
Com base no Exam Guide que enviei, liste de 6 a 10 tarefas hands-on que eu deveria executar com
fluência antes da prova. Para cada uma: uma frase de descrição e o objetivo do exame a que se
refere. Cada tarefa deve caber em menos de 30 minutos na Databricks Free Edition ou num trial.
```

Todo laboratório que você **não** conseguir concluir vira o próximo alvo de estudo.

---

## Passo 6 — Ritmo

Trabalhe de trás para frente a partir da data da prova, em sprints de 2 semanas. A [trilha](02%20-%20Trilha%20de%20Estudo.md) já está organizada assim:

| Fase | O que fazer |
|---|---|
| **Primeiro quarto** do tempo | Diagnosticar **todos** os domínios. Identificar os 2 ou 3 mais fracos |
| **Metade do meio** | Aprofundar os objetivos fracos. Fazer os laboratórios |
| **Último quarto** | Simulados completos para achar pontos fracos e ganhar resistência |

> Simulados (inclusive os deste repositório e os gerados por IA) **não são uma medida calibrada** de prontidão.
> O sinal real é: **todos os objetivos do Exam Guide cobertos + todos os laboratórios feitos com segurança.**

---

## Sem tempo para tudo? As 5 prioridades

1. **Abra toda sessão** com o Exam Guide + a tabela de nomes renomeados.
2. **Diagnostique antes de estudar**: não gaste tempo no que já sabe.
3. **Rode todo trecho de código** e confirme detalhes em docs.databricks.com. Peça a URL da documentação: **sem link, a IA está adivinhando.**
4. **Confira temas-chave com uma segunda IA**: se elas discordarem, vá à documentação.
5. **Faça os laboratórios**: só ler não aprova em nenhuma certificação Databricks.
