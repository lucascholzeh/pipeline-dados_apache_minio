# Notebook 03 — DML no Delta Lake

## Objetivo

Demonstrar as três operações DML (INSERT, UPDATE, DELETE) sobre tabelas Delta Lake no bucket `bronze`, com exibição do histórico de versões (HISTORY) e consulta a versões anteriores (TIME TRAVEL).

## Tabelas utilizadas

| Tabela | Operações |
|---|---|
| `marca` | INSERT (Tesla, BYD, GWM), UPDATE (Hyundai), DELETE (GWM) |
| `cliente` | INSERT (novo cliente), UPDATE (email), DELETE (novo cliente) |
| `apolice` | UPDATE (status → Expirada) |

## Operações por seção

### INSERT

**Tabela `marca` via Spark SQL:**
```sql
-- Cria um DataFrame com as novas linhas
SELECT 11 AS id_marca, 'Tesla' AS nome_marca, 'EUA' AS pais_origem
UNION ALL SELECT 12, 'BYD', 'China'
UNION ALL SELECT 13, 'GWM', 'China'
```
Depois escreve com `.mode('append')`.

**Tabela `cliente` via DeltaTable MERGE:**
```python
dt_cliente.alias('target').merge(
    novo_cliente.alias('source'),
    'target.id_cliente = source.id_cliente'
).whenNotMatchedInsertAll().execute()
```
O MERGE garante idempotência — não insere duplicatas.

### UPDATE

**Tabela `marca` via Spark SQL:**
```sql
UPDATE delta.`s3a://bronze/marca`
SET pais_origem = 'Coreia do Sul'
WHERE nome_marca = 'Hyundai'
```

**Tabela `apolice` via DeltaTable API:**
```python
dt_apolice.update(
    condition = F.col('data_fim') < F.current_date(),
    set       = {'status': F.lit('Expirada')}
)
```

**Tabela `cliente` via DeltaTable API:**
```python
dt_cliente.update(
    condition = F.col('id_cliente') == 1,
    set       = {'email': F.lit('ana.souza.novo@email.com')}
)
```

### DELETE

**Tabela `marca` via Spark SQL:**
```sql
DELETE FROM delta.`s3a://bronze/marca`
WHERE id_marca = 13
```

**Tabela `cliente` via DeltaTable API:**
```python
dt_cliente.delete(condition = F.col('id_cliente') == 21)
```

### HISTORY

```python
dt = DeltaTable.forPath(spark, 's3a://bronze/marca')
dt.history().select('version', 'timestamp', 'operation').show()
```

Resultado esperado para `marca`:

```
+-------+-------------------+---------+
|version|timestamp          |operation|
+-------+-------------------+---------+
|3      |2024-... 10:05:00  |DELETE   |
|2      |2024-... 10:04:00  |UPDATE   |
|1      |2024-... 10:03:00  |WRITE    |  ← INSERT (append)
|0      |2024-... 10:02:00  |WRITE    |  ← carga inicial
+-------+-------------------+---------+
```

### TIME TRAVEL

```python
# Versão 0 (antes de qualquer DML)
df_v0 = spark.read.format('delta').option('versionAsOf', 0).load('s3a://bronze/marca')

# Versão atual
df_atual = spark.read.format('delta').load('s3a://bronze/marca')
```

O Time Travel permite comparar o estado dos dados antes e depois das operações DML — útil para auditoria e recuperação de dados.

## Resumo das operações

```
╔══════════╦════════════╦═══════════════════════════════════════════╗
║ Operação ║  Tabela    ║  Descrição                                ║
╠══════════╬════════════╬═══════════════════════════════════════════╣
║ INSERT   ║ marca      ║ Novas marcas: Tesla, BYD, GWM (Spark SQL) ║
║ INSERT   ║ cliente    ║ Novo cliente id=21 (DeltaTable MERGE API) ║
║ UPDATE   ║ marca      ║ Hyundai → pais_origem corrigido (SQL)     ║
║ UPDATE   ║ apolice    ║ Status = 'Expirada' p/ vencidas (API)     ║
║ UPDATE   ║ cliente    ║ Email da Ana Souza atualizado (API)       ║
║ DELETE   ║ marca      ║ Removeu GWM (id=13) via Spark SQL         ║
║ DELETE   ║ cliente    ║ Removeu cliente id=21 via DeltaTable API  ║
╚══════════╩════════════╩═══════════════════════════════════════════╝
```
