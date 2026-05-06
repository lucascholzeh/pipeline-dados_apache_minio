# Delta Lake

## O que é

**Delta Lake** é uma camada de armazenamento open-source que adiciona transações ACID, versionamento e suporte a DML sobre arquivos Parquet em data lakes. Foi criado pela Databricks e hoje é mantido pela Linux Foundation.

## Versão utilizada

**delta-spark 3.2.0** — compatível com PySpark 3.5.x.

## Por que Delta Lake?

### O problema com Parquet puro

Arquivos Parquet são excelentes para leitura analítica, mas têm limitações sérias:

- **Sem UPDATE/DELETE** — só é possível reescrever o arquivo inteiro
- **Sem atomicidade** — se uma escrita falha no meio, os dados ficam corrompidos
- **Sem versionamento** — não há como saber o que mudou ou quando
- **Problemas de concorrência** — leituras simultâneas a escritas podem retornar dados inconsistentes

### Como o Delta Lake resolve

Delta Lake mantém um **transaction log** (`_delta_log`) — um diretório de arquivos JSON que registra cada operação:

```
bronze/marca/
├── part-00000-abc123.parquet   ← dados v0
├── part-00000-def456.parquet   ← dados v1 (após INSERT)
└── _delta_log/
    ├── 00000000000000000000.json   ← commit: WRITE (versão 0)
    ├── 00000000000000000001.json   ← commit: WRITE (versão 1: INSERT)
    ├── 00000000000000000002.json   ← commit: UPDATE (versão 2)
    └── 00000000000000000003.json   ← commit: DELETE (versão 3)
```

## Operações DML suportadas

### INSERT (append)

```python
df_novos.write.format('delta').mode('append').save('s3a://bronze/marca')
```

### INSERT via Spark SQL

```sql
INSERT INTO delta.`s3a://bronze/marca`
VALUES (11, 'Tesla', 'EUA')
```

### UPDATE via DeltaTable API

```python
from delta import DeltaTable

dt = DeltaTable.forPath(spark, 's3a://bronze/apolice')
dt.update(
    condition = F.col('data_fim') < F.current_date(),
    set       = {'status': F.lit('Expirada')}
)
```

### UPDATE via Spark SQL

```sql
UPDATE delta.`s3a://bronze/marca`
SET pais_origem = 'Coreia do Sul'
WHERE nome_marca = 'Hyundai'
```

### DELETE via DeltaTable API

```python
dt.delete(condition = F.col('id_marca') == 13)
```

### DELETE via Spark SQL

```sql
DELETE FROM delta.`s3a://bronze/marca`
WHERE id_marca = 13
```

### MERGE (Upsert)

```python
dt.alias('target').merge(
    df_source.alias('source'),
    'target.id_marca = source.id_marca'
).whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()
```

## History — Histórico de versões

```python
dt = DeltaTable.forPath(spark, 's3a://bronze/marca')
dt.history().select('version', 'timestamp', 'operation').show()
```

Saída típica:

```
+-------+-------------------+-----------+
|version|timestamp          |operation  |
+-------+-------------------+-----------+
|3      |2024-01-15 10:05:00|DELETE     |
|2      |2024-01-15 10:04:00|UPDATE     |
|1      |2024-01-15 10:03:00|WRITE      |
|0      |2024-01-15 10:02:00|WRITE      |
+-------+-------------------+-----------+
```

## Time Travel — Consulta a versões anteriores

```python
# Por número de versão
df_v0 = spark.read.format('delta').option('versionAsOf', 0).load('s3a://bronze/marca')

# Por timestamp
df_ts = spark.read.format('delta').option('timestampAsOf', '2024-01-15 10:02:00').load('s3a://bronze/marca')
```

## Garantias ACID

| Propriedade | Delta Lake |
|---|---|
| **A**tomicidade | Uma operação é tudo ou nada — nunca parcial |
| **C**onsistência | O schema é validado em cada escrita |
| **I**solamento | Leituras não veem escritas em andamento |
| **D**urabilidade | Dados confirmados no log não se perdem |
