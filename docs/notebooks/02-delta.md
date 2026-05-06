# Notebook 02 — CSV → Delta Lake

## Objetivo

Ler os arquivos CSV do bucket `landing-zone` com **Apache Spark** e convertê-los para o formato **Delta Lake** no bucket `bronze`. Esta é a transformação central do pipeline.

## O que faz

1. Inicializa uma SparkSession com suporte a Delta Lake e conectividade S3A (MinIO)
2. Cria o bucket `bronze` no MinIO (se não existir)
3. Lista os arquivos CSV disponíveis no `landing-zone`
4. Para cada CSV:
   - Lê com `spark.read.csv()` (inferência de schema automática)
   - Escreve em Delta Lake com `df.write.format('delta').save()`
   - Valida com `DeltaTable.isDeltaTable()`
5. Exibe schema e amostra de cada tabela Delta criada
6. Demonstra o transaction log (`_delta_log`) da tabela `apolice`

## Tecnologias

- `pyspark` — motor de processamento
- `delta-spark` — extensão Delta Lake para Spark
- `hadoop-aws` + `aws-java-sdk-bundle` — conectividade S3A/MinIO

## Configuração crítica da SparkSession

```python
builder = (
    SparkSession.builder
    .config('spark.sql.extensions', 'io.delta.sql.DeltaSparkSessionExtension')
    .config('spark.sql.catalog.spark_catalog', 'org.apache.spark.sql.delta.catalog.DeltaCatalog')
    .config('spark.hadoop.fs.s3a.endpoint', MINIO_ENDPOINT)
    .config('spark.hadoop.fs.s3a.path.style.access', 'true')
    .config('spark.jars.packages', 'io.delta:delta-spark_2.12:3.2.0,...')
)
```

!!! warning "Primeira execução"
    Na primeira execução, o Spark baixa os JARs do Maven (~200 MB). Isso pode levar alguns minutos dependendo da conexão. As execuções seguintes usam o cache local.

## Resultado esperado

Após execução, o bucket `bronze` terá 11 pastas Delta:

```
s3a://bronze/
├── regiao/      (_delta_log/ + parquet)
├── estado/      (_delta_log/ + parquet)
├── municipio/   (_delta_log/ + parquet)
├── marca/       (_delta_log/ + parquet)
├── modelo/      (_delta_log/ + parquet)
├── cliente/     (_delta_log/ + parquet)
├── endereco/    (_delta_log/ + parquet)
├── telefone/    (_delta_log/ + parquet)
├── carro/       (_delta_log/ + parquet)
├── apolice/     (_delta_log/ + parquet)
└── sinistro/    (_delta_log/ + parquet)
```

## Validação Delta

O método `DeltaTable.isDeltaTable(spark, path)` retorna `True` se o diretório é uma tabela Delta válida (tem `_delta_log`).

## History após a conversão

Cada tabela terá **versão 0** no histórico, com operação `WRITE`:

```
+-------+-----------+---------+
|version|operation  |timestamp|
+-------+-----------+---------+
|0      |WRITE      |2024-....|
+-------+-----------+---------+
```
