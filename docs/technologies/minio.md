# MinIO

## O que é

**MinIO** é um servidor de object storage de alto desempenho e código aberto, 100% compatível com a API do Amazon S3. Pode ser rodado on-premise (em servidores próprios) ou em containers Docker, substituindo o S3 da AWS em ambientes locais e privados.

## Versão utilizada

`minio/minio:RELEASE.2025-02-03T21-03-04Z`

## Por que MinIO?

| Razão | Explicação |
|---|---|
| Compatibilidade S3 | Qualquer biblioteca que usa boto3 ou S3A funciona sem alteração de código |
| Self-hosted | Não precisa de conta AWS — roda localmente em Docker |
| Console web | Interface gráfica para visualizar buckets e arquivos |
| Performance | Adequado para volumes de dados de laboratório e produção |
| Gratuito | Open-source, sem licença |

## Buckets utilizados

| Bucket | Propósito | Formato dos dados |
|---|---|---|
| `landing-zone` | Dados brutos extraídos do SQL Server | CSV |
| `bronze` | Dados convertidos para Delta Lake | Parquet + `_delta_log` |

## Acesso

### Console Web
- URL: [http://localhost:9021](http://localhost:9021)
- Usuário: `minioadmin`
- Senha: `minioadmin`

### API S3
- Endpoint: `http://localhost:9020`
- Porta diferente da 9021 (console) para separar as interfaces

## Uso com boto3 (Python)

```python
import boto3
from botocore.client import Config

s3 = boto3.client(
    's3',
    endpoint_url='http://localhost:9020',
    aws_access_key_id='minioadmin',
    aws_secret_access_key='minioadmin',
    config=Config(signature_version='s3v4'),
    region_name='us-east-1',
)

# Upload de arquivo
s3.put_object(Bucket='landing-zone', Key='marca.csv', Body=csv_bytes)

# Listar objetos
response = s3.list_objects_v2(Bucket='landing-zone')
for obj in response['Contents']:
    print(obj['Key'], obj['Size'])
```

## Uso com Spark (S3A)

O Spark acessa o MinIO via protocolo S3A — o mesmo usado para o Amazon S3 real. A única diferença é o `endpoint` apontando para `localhost:9020`:

```python
spark.read.csv('s3a://landing-zone/marca.csv')
df.write.format('delta').save('s3a://bronze/marca')
```

!!! tip "path.style.access"
    É obrigatório configurar `spark.hadoop.fs.s3a.path.style.access = true` ao usar MinIO. O acesso por subdomain (`bucket.localhost`) não funciona em localhost.

## Estrutura de paths no MinIO

```
s3a://landing-zone/
├── regiao.csv
├── estado.csv
└── ...

s3a://bronze/
├── marca/
│   ├── part-00000-xyz.parquet
│   └── _delta_log/
│       └── 00000000000000000000.json
└── ...
```
