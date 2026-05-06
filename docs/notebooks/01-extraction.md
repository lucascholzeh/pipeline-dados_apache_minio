# Notebook 01 — SQL Server → MinIO (CSV)

## Objetivo

Extrair todas as tabelas do **SeguroDB** e salvar como arquivos CSV no bucket `landing-zone` do MinIO. Esta é a primeira etapa do pipeline — a **extração** (E do ETL).

## O que faz

1. Conecta ao SQL Server e lista todas as tabelas via `INFORMATION_SCHEMA`
2. Cria o bucket `landing-zone` no MinIO (se não existir)
3. Para cada tabela:
   - Executa `SELECT * FROM dbo.<tabela>`
   - Converte o resultado para DataFrame pandas
   - Serializa para CSV em memória (`io.BytesIO`)
   - Faz upload para `s3a://landing-zone/<tabela>.csv` via boto3

## Tecnologias

- `pyodbc` — conexão ao SQL Server
- `pandas` — manipulação dos dados
- `boto3` — SDK S3 para upload ao MinIO
- `python-dotenv` — leitura de credenciais

## Por que não usar Spark neste notebook?

Para a extração de um banco relacional com poucos registros, o uso de Spark seria desnecessariamente complexo. O Spark não possui conector nativo para SQL Server via ODBC — precisaria do JDBC — o que adiciona complexidade de configuração. O `pyodbc` com pandas é mais simples e igualmente eficiente para este volume de dados.

## Resultado esperado

```
[landing-zone/regiao.csv]     5 linhas  (0.1 KB)
[landing-zone/estado.csv]    27 linhas  (0.5 KB)
[landing-zone/municipio.csv] 30 linhas  (0.6 KB)
[landing-zone/marca.csv]     10 linhas  (0.3 KB)
[landing-zone/modelo.csv]    30 linhas  (0.7 KB)
[landing-zone/cliente.csv]   20 linhas  (1.1 KB)
[landing-zone/endereco.csv]  20 linhas  (1.2 KB)
[landing-zone/telefone.csv]  20 linhas  (0.7 KB)
[landing-zone/carro.csv]     20 linhas  (1.0 KB)
[landing-zone/apolice.csv]   20 linhas  (1.3 KB)
[landing-zone/sinistro.csv]  10 linhas  (0.9 KB)
```

## Verificação no Console MinIO

Após executar este notebook, acesse [http://localhost:9021](http://localhost:9021) → bucket `landing-zone` e confirme que os 11 arquivos CSV estão presentes.
