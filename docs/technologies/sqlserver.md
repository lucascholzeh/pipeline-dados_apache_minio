# SQL Server

## O que é

**Microsoft SQL Server** é um sistema de gerenciamento de banco de dados relacional (RDBMS) desenvolvido pela Microsoft. É amplamente utilizado em ambientes corporativos e é a principal fonte de dados deste pipeline.

## Versão utilizada

**SQL Server 2022 Developer Edition** — gratuita para desenvolvimento, idêntica à Enterprise em funcionalidades.

## Por que SQL Server?

Este projeto utiliza SQL Server como fonte de dados para simular um cenário real de empresa, onde os dados operacionais estão armazenados em banco relacional e precisam ser exportados para um data lake para análise.

## Configuração Docker

```yaml
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      SA_PASSWORD: "SqlServer@2022!"
      ACCEPT_EULA: "Y"
      MSSQL_PID: "Developer"
    ports:
      - "1433:1433"
```

## Conexão com pyodbc

```python
import pyodbc

conn_str = (
    'DRIVER={ODBC Driver 18 for SQL Server};'
    'SERVER=localhost,1433;'
    'DATABASE=SeguroDB;'
    'UID=sa;PWD=SqlServer@2022!;'
    'TrustServerCertificate=yes;'
)

conn = pyodbc.connect(conn_str)
```

!!! note "TrustServerCertificate"
    Em ambiente Docker local, o SQL Server usa um certificado auto-assinado. O parâmetro `TrustServerCertificate=yes` desativa a validação do certificado — adequado apenas para desenvolvimento.

## Banco SeguroDB

O banco **SeguroDB** simula o sistema de uma seguradora de veículos:

- 11 tabelas relacionais com chaves estrangeiras
- Dados de exemplo cobrindo regiões, estados, municípios, clientes, veículos, apólices e sinistros
- Schema no namespace `dbo` (padrão SQL Server)

## Extração das tabelas

A extração usa `pandas.read_sql` sobre a conexão `pyodbc`, que retorna um DataFrame pronto para serialização em CSV:

```python
import pandas as pd

df = pd.read_sql('SELECT * FROM dbo.marca', conn)
df.to_csv('marca.csv', index=False)
```

## Listagem dinâmica de tabelas

```python
query = """
    SELECT TABLE_NAME
    FROM INFORMATION_SCHEMA.TABLES
    WHERE TABLE_TYPE = 'BASE TABLE'
    ORDER BY TABLE_NAME
"""
df_tabelas = pd.read_sql(query, conn)
tabelas = df_tabelas['TABLE_NAME'].tolist()
```

Isso garante que o Notebook 01 extraia **todas** as tabelas existentes automaticamente, sem precisar listar os nomes manualmente.
