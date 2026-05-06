# Tabelas Gerenciadas vs Não Gerenciadas

## Conceito

No Apache Spark com Delta Lake, há dois tipos de tabelas quanto à gestão dos dados:

| Aspecto | Gerenciada (Managed) | Não Gerenciada (External/Unmanaged) |
|---|---|---|
| **Dados** | Controlados pelo Spark (warehouse) | Armazenados em path externo definido pelo usuário |
| **Metadados** | Catalog do Spark | Catalog do Spark (apenas referência) |
| **DROP TABLE** | Remove dados e metadados | Remove apenas os metadados; **dados permanecem** |
| **Localização** | `/user/hive/warehouse/` (padrão) | Qualquer path (S3, HDFS, local...) |
| **Criação** | `CREATE TABLE nome (...)` | `CREATE TABLE nome USING delta LOCATION 's3a://...'` |

---

## O que usamos neste projeto

Neste projeto usamos **tabelas não gerenciadas** (external tables), pois especificamos o path de armazenamento explicitamente:

```python
# Tabela NÃO GERENCIADA — path explícito no MinIO
df.write.format('delta').mode('overwrite').save('s3a://bronze/marca')

# Leitura também por path
spark.read.format('delta').load('s3a://bronze/marca')
```

Isso significa que mesmo que o contexto Spark/catalog seja destruído, **os dados permanecem no MinIO** e podem ser acessados novamente informando o path.

---

## Diferenças práticas observadas

### No Laboratório 1 (referência do professor)

O repositório de referência (`jlsilva01/spark-delta-minio-sqlserver`) também usa tabelas não gerenciadas com paths S3A — exatamente o mesmo padrão deste projeto.

As tabelas são acessadas sempre via `DeltaTable.forPath(spark, path)` ou `spark.read.format('delta').load(path)`, não pelo nome no catalog.

### Tabela gerenciada — como seria

```python
# Registrar no catalog como tabela gerenciada
spark.sql("""
    CREATE TABLE IF NOT EXISTS marca
    USING delta
    AS SELECT * FROM delta.`s3a://bronze/marca`
""")

# Depois pode acessar pelo nome
spark.sql("SELECT * FROM marca")
spark.sql("DESCRIBE HISTORY marca")
```

Com tabela gerenciada, o Spark Catalog guarda os metadados e você pode usar o nome diretamente no Spark SQL sem informar o path. A desvantagem é que os dados ficam no warehouse local do Spark — não no MinIO.

### Tabela externa (unmanaged) com catalog

```python
# Registrar tabela externa no catalog
spark.sql("""
    CREATE TABLE IF NOT EXISTS marca_ext
    USING delta
    LOCATION 's3a://bronze/marca'
""")

# Agora pode usar o nome (mas dados ficam no MinIO)
spark.sql("SELECT * FROM marca_ext")
```

---

## Resumo da comparação

```
┌──────────────────────────────────────────────────────────────────┐
│                    TABELAS NO DELTA LAKE                         │
├──────────────────────┬───────────────────────────────────────────┤
│   GERENCIADA         │   NÃO GERENCIADA (EXTERNAL)               │
│                      │                                           │
│ ✓ Nome simples no SQL│ ✓ Dados em qualquer storage               │
│ ✓ Catalog integrado  │ ✓ Dados sobrevivem sem catalog            │
│ ✗ Dados no warehouse │ ✓ Ideal para S3/MinIO                     │
│ ✗ DROP apaga dados   │ ✓ DROP não apaga dados                    │
│                      │ ✗ Precisa do path completo                │
├──────────────────────┴───────────────────────────────────────────┤
│   ESTE PROJETO usa: TABELAS NÃO GERENCIADAS                      │
│   Motivo: dados armazenados no MinIO (object storage externo)    │
└──────────────────────────────────────────────────────────────────┘
```

## Quando usar cada uma

**Use tabela gerenciada quando:**
- Trabalha com Databricks Unity Catalog ou Hive Metastore centralizado
- Quer simplificar queries SQL sem informar paths
- Os dados podem ficar no warehouse padrão do cluster

**Use tabela não gerenciada quando:**
- Os dados ficam em object storage externo (S3, MinIO, ADLS)
- Precisa que os dados sobrevivam independente do cluster Spark
- Compartilha o mesmo storage entre múltiplas aplicações (este projeto)
