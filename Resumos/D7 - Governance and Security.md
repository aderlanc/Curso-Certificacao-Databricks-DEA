# Domínio 7 — Governance and Security (15% · ~7 questões)

> O curso cobre managed x external (`05/2`), hierarquia do Unity Catalog (`04`) e GRANT/REVOKE (`11/1`).
> Conversão de tabelas, row filters, column masks e ABAC são estudados por este resumo.

**Objetivos do guia:**
1. Diferenciar tabelas **managed** e **external** no Unity Catalog e fazer operações básicas: criar, alterar, excluir e **converter** entre elas.
2. Configurar controle de acesso pela UI e por SQL (**GRANT, REVOKE, DENY**) para usuários, grupos e **service principals**, no nível certo da hierarquia.
3. Entender **column masking** e **row-level security** para restringir o que cada grupo vê.
4. Entender políticas **ABAC** do Unity Catalog para aplicar filtros e máscaras de forma centralizada.

---

## 1. Hierarquia de objetos

```
Metastore (1 por região)
└── Catalog                 → USE CATALOG
    └── Schema              → USE SCHEMA
        ├── Table / View / Materialized View / Streaming Table → SELECT, MODIFY
        ├── Volume          → READ VOLUME, WRITE VOLUME
        └── Function / Model → EXECUTE
Objetos de segurança externos: Storage Credential, External Location, Connection (Lakehouse Federation)
```

Nome de 3 níveis: `catalogo.schema.objeto`.

## 2. Managed x External

| | Managed | External |
|---|---|---|
| Onde ficam os arquivos | Local gerenciado pelo UC (definido no metastore, catálogo ou schema) | Caminho que você informa com `LOCATION` |
| Quem gerencia o ciclo de vida dos dados | Unity Catalog | Você |
| `DROP TABLE` | Remove metadados **e dados** (recuperável com `UNDROP TABLE` por 7 dias) | Remove **só os metadados**, os arquivos continuam no storage |
| Formato | Delta (ou Iceberg) | Delta, Parquet, CSV, JSON, etc. |
| Recursos automáticos | Predictive Optimization, `CLUSTER BY AUTO`, cache de metadados | Não |
| Pré-requisito | Nenhum | **Storage credential** + **external location** com permissão `CREATE EXTERNAL TABLE` |
| Recomendação Databricks | **Padrão** para quase tudo | Dados compartilhados com outras ferramentas fora do Databricks |

```sql
-- Managed
CREATE TABLE vendas.silver.clientes (id BIGINT, nome STRING);

-- External
CREATE TABLE vendas.bronze.logs (id BIGINT, msg STRING)
LOCATION 'abfss://raw@meustorage.dfs.core.windows.net/logs';

-- Converter EXTERNAL → MANAGED (os dados são copiados para o local gerenciado)
ALTER TABLE vendas.bronze.logs SET MANAGED;

-- Desfazer a conversão (só dentro de 14 dias após a conversão)
ALTER TABLE vendas.bronze.logs UNSET MANAGED;

-- Ver o tipo da tabela
DESCRIBE TABLE EXTENDED vendas.bronze.logs;   -- campo "Type": MANAGED ou EXTERNAL
```

> MANAGED → EXTERNAL não tem comando direto: recrie a tabela com `LOCATION` (por exemplo com `CREATE TABLE ... LOCATION ... AS SELECT` ou `DEEP CLONE`).

## 3. Permissões

```sql
GRANT USE CATALOG ON CATALOG vendas TO `analistas`;
GRANT USE SCHEMA  ON SCHEMA  vendas.gold TO `analistas`;
GRANT SELECT      ON SCHEMA  vendas.gold TO `analistas`;   -- vale para todas as tabelas atuais e futuras do schema
GRANT MODIFY      ON TABLE   vendas.silver.pedidos TO `etl-service-principal`;
REVOKE SELECT     ON TABLE   vendas.gold.salarios FROM `analistas`;
SHOW GRANTS ON SCHEMA vendas.gold;
ALTER TABLE vendas.gold.kpis OWNER TO `time-dados`;
```

Regras que caem na prova:
- Para ler uma tabela é preciso **as três**: `USE CATALOG` no catálogo + `USE SCHEMA` no schema + `SELECT` na tabela (ou no schema/catálogo, por herança).
- Privilégios **herdam para baixo**: `SELECT` no schema vale para todas as tabelas dele, inclusive as criadas depois.
- O **owner** do objeto pode tudo nele e pode conceder permissões. Admins do metastore e owners do catálogo controlam tudo abaixo.
- `MODIFY` = INSERT, UPDATE, DELETE, MERGE. `ALL PRIVILEGES` concede tudo (exceto ownership).
- `BROWSE` deixa ver os metadados (descobrir que a tabela existe) sem ler os dados.
- Conceda a **grupos**, não a pessoas. Para jobs e CI/CD use **service principals** (identidade de máquina) como `run_as`.
- **DENY**: nega explicitamente e prevalece sobre um GRANT. Vem do modelo de table ACLs do `hive_metastore`. No Unity Catalog, tudo que não foi concedido já é negado por padrão, então a forma usual de tirar acesso é `REVOKE`.

## 4. Row filters e column masks

São **funções SQL (UDFs)** ligadas à tabela. A função recebe valores da linha e decide o que o usuário vê, normalmente com `is_account_group_member()` ou `current_user()`.

```sql
-- Row filter: gerentes veem tudo, os demais só a região BR
CREATE FUNCTION vendas.seg.filtro_regiao(regiao STRING)
RETURN is_account_group_member('gerentes') OR regiao = 'BR';

ALTER TABLE vendas.gold.pedidos SET ROW FILTER vendas.seg.filtro_regiao ON (regiao);
ALTER TABLE vendas.gold.pedidos DROP ROW FILTER;

-- Column mask: só finance_team vê o cartão completo
CREATE FUNCTION vendas.seg.mascara_cartao(cartao STRING)
RETURN CASE WHEN is_account_group_member('finance_team') THEN cartao
            ELSE concat('****-****-****-', right(cartao, 4)) END;

ALTER TABLE vendas.gold.pagamentos ALTER COLUMN cartao SET MASK vendas.seg.mascara_cartao;
ALTER TABLE vendas.gold.pagamentos ALTER COLUMN cartao DROP MASK;
```

- A alternativa antiga é a **dynamic view** (view com `CASE WHEN is_account_group_member(...)` e `WHERE`). Funciona, mas o usuário precisa ler a view e não a tabela.
- `is_account_group_member()` checa grupos da **conta**. `is_member()` é a versão antiga, de grupos do workspace.

## 5. ABAC (Attribute-Based Access Control)

Problema: aplicar máscara coluna por coluna em centenas de tabelas não escala.
Solução: marcar dados com **governed tags** (ex.: `pii = cpf`) e criar uma **política** num catálogo ou schema que aplica filtro/máscara a **toda coluna ou tabela com aquela tag**, inclusive as criadas depois.

```sql
-- Sintaxe ilustrativa, confira na documentação
ALTER TABLE vendas.silver.clientes ALTER COLUMN cpf SET TAGS ('pii' = 'cpf');

CREATE POLICY mascarar_pii
ON SCHEMA vendas.silver
COLUMN MASK vendas.seg.mascara_texto
TO `account users` EXCEPT `compliance`
FOR TABLES
MATCH COLUMNS has_tag_value('pii', 'cpf') AS c
ON COLUMN c;
```

| | Row filter / column mask por tabela | ABAC |
|---|---|---|
| Onde se define | Em cada tabela/coluna | Uma política no catálogo ou schema |
| Como acha os dados | Nome da tabela e coluna | **Tags** |
| Tabelas novas | Precisa configurar de novo | Herdam automaticamente |
| Bom para | Poucos casos específicos | Governança em escala |

## 6. Outros recursos de governança (bom saber)

- **Lineage**: o UC captura automaticamente a linhagem de tabelas e colunas, visível no Catalog Explorer (aba *Lineage*) e em `system.access.table_lineage`.
- **Audit logs**: em `system.access.audit` ou entregues como **JSON** no storage do cliente.
- **Delta Sharing**: compartilha dados sem cópia. *Databricks-to-Databricks* (entre metastores UC) ou *aberto* (para qualquer cliente, via token). Objetos: share, recipient, provider.
- **Lakehouse Federation**: consulta bancos externos (SQL Server, PostgreSQL, Snowflake…) **sem ingerir**, via *connection* + *foreign catalog*.

---

## ✅ Checklist do domínio
- [ ] Sei o que o DROP faz em managed x external e o que é `UNDROP`
- [ ] Sei converter external → managed (`SET MANAGED`) e desfazer (`UNSET MANAGED`, 14 dias)
- [ ] Sei as 3 permissões mínimas para ler uma tabela e como a herança funciona
- [ ] Sei criar row filter e column mask com `is_account_group_member()`
- [ ] Sei explicar quando ABAC é melhor que máscara por tabela

## 📚 Fontes oficiais
- [Managed x external tables](https://learn.microsoft.com/azure/databricks/tables/)
- [Converter external para managed](https://learn.microsoft.com/azure/databricks/tables/convert-external-managed)
- [Privilégios do Unity Catalog](https://learn.microsoft.com/azure/databricks/data-governance/unity-catalog/manage-privileges/privileges)
- [Row filters e column masks](https://learn.microsoft.com/azure/databricks/tables/row-and-column-filters)
- [ABAC](https://learn.microsoft.com/azure/databricks/data-governance/unity-catalog/abac/)
