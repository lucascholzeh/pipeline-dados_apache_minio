# Pipeline de Dados: SQL Server → MinIO → Delta Lake

Bem-vindo à documentação do projeto de **Engenharia de Dados** desenvolvido para a disciplina de Pipeline de Dados.

## O que este projeto faz

Este projeto implementa um pipeline de dados completo seguindo a **Arquitetura Medalhão**, partindo de um banco de dados relacional (SQL Server) e chegando a tabelas Delta Lake armazenadas em um object storage compatível com S3 (MinIO).

```
SQL Server (SeguroDB)
        │
        │  pyodbc + pandas
        ▼
MinIO — landing-zone/  (CSV)
        │
        │  Apache Spark + Delta Lake
        ▼
MinIO — bronze/  (Delta Tables)
        │
        │  INSERT / UPDATE / DELETE
        │  HISTORY / TIME TRAVEL
        ▼
     Análise de Dados
```

## Funcionalidades

| Feature | Descrição |
|---|---|
| Extração SQL Server | Leitura de todas as tabelas via `pyodbc` |
| Upload para MinIO | Armazenamento CSV no bucket `landing-zone` |
| Conversão Delta Lake | Spark lê CSV e escreve formato Delta no bucket `bronze` |
| Transações ACID | Garantia de consistência nas operações |
| DML completo | INSERT, UPDATE e DELETE em tabelas Delta |
| Histórico de versões | Cada operação gera uma nova versão registrada |
| Time Travel | Consulta a qualquer versão histórica dos dados |

## Início Rápido

```bash
# 1. Subir containers
docker compose up -d

# 2. Configurar ambiente Python
cp .env.example .env
uv venv && source .venv/bin/activate
uv sync

# 3. Executar notebooks em ordem:
#    00_setup_sqlserver.ipynb
#    01_sqlserver_to_minio_csv.ipynb
#    02_csv_to_delta.ipynb
#    03_dml_delta.ipynb
```

## Tecnologias Utilizadas

<div class="grid cards" markdown>

- :material-lightning-bolt: **Apache Spark 3.5.3** — Motor de processamento distribuído
- :material-database: **Delta Lake 3.2.0** — Formato lakehouse com ACID e Time Travel
- :material-server: **MinIO** — Object Storage compatível com S3
- :material-microsoft: **SQL Server 2022** — Banco de dados relacional fonte
- :material-docker: **Docker Compose** — Orquestração dos containers
- :material-language-python: **Python 3.11** — Linguagem principal do projeto

</div>
