# Apache Spark

## O que é

**Apache Spark** é um motor de processamento de dados distribuído de código aberto, projetado para processar grandes volumes de dados de forma rápida e eficiente. É o padrão de mercado para engenharia de dados em escala.

## Versão utilizada

**PySpark 3.5.3** — versão LTS com suporte ao Delta Lake 3.2.0.

## Por que Spark neste projeto?

| Razão | Explicação |
|---|---|
| Integração Delta Lake | Spark é o motor nativo do Delta Lake — a API `delta-spark` é desenvolvida em cima do Spark |
| Leitura S3/MinIO | Spark lê e escreve diretamente no MinIO via protocolo S3A sem etapas intermediárias |
| Spark SQL | Permite executar INSERT, UPDATE, DELETE com sintaxe SQL padrão sobre tabelas Delta |
| Escalabilidade | O mesmo código funciona localmente e em clusters de produção (AWS EMR, Databricks, etc.) |

## Configuração S3A para MinIO

O Spark se conecta ao MinIO via protocolo **S3A** (S3-compatible). A configuração é feita diretamente no `SparkSession.builder`:

```python
builder = (
    SparkSession.builder
    .config('spark.hadoop.fs.s3a.endpoint', 'http://localhost:9020')
    .config('spark.hadoop.fs.s3a.access.key', 'minioadmin')
    .config('spark.hadoop.fs.s3a.secret.key', 'minioadmin')
    .config('spark.hadoop.fs.s3a.path.style.access', 'true')        # MinIO exige path-style
    .config('spark.hadoop.fs.s3a.impl', 'org.apache.hadoop.fs.s3a.S3AFileSystem')
    .config('spark.hadoop.fs.s3a.connection.ssl.enabled', 'false')  # HTTP local
)
```

!!! warning "path.style.access"
    MinIO (e a maioria dos S3-compatíveis self-hosted) **exige** `path.style.access = true`. Sem isso, as requisições vão para `bucket.localhost:9020` (subdomain-style), que não funciona em ambiente local.

## JARs necessários

Os JARs são baixados automaticamente pelo Maven na primeira execução:

```python
.config('spark.jars.packages',
    'io.delta:delta-spark_2.12:3.2.0,'           # Delta Lake
    'org.apache.hadoop:hadoop-aws:3.3.4,'         # S3A connector
    'com.amazonaws:aws-java-sdk-bundle:1.12.262'  # AWS SDK
)
```

## Modos de Escrita Delta

| Modo | Comportamento |
|---|---|
| `overwrite` | Substitui todos os dados existentes |
| `append` | Adiciona novos dados sem apagar os existentes |
| `merge` | Upsert — insere se não existe, atualiza se existe |

## Exemplo de leitura e escrita

```python
# Lê CSV do landing-zone
df = spark.read.option('header', 'true').option('inferSchema', 'true').csv('s3a://landing-zone/marca.csv')

# Escreve como Delta no bronze
df.write.format('delta').mode('overwrite').save('s3a://bronze/marca')

# Lê Delta do bronze
df_delta = spark.read.format('delta').load('s3a://bronze/marca')
```
