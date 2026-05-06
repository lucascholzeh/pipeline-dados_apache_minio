# Notebook 00 — Setup SQL Server

## Objetivo

Preparar o banco de dados **SeguroDB** no SQL Server para ser a fonte de dados do pipeline. Cria as tabelas e carrega os dados de exemplo a partir dos arquivos CSV em `data/`.

## O que faz

1. Conecta ao SQL Server via `pyodbc` no banco `master`
2. Cria o banco `SeguroDB` (se não existir)
3. Reconecta ao `SeguroDB` e cria as 11 tabelas com DDL
4. Lê cada CSV da pasta `data/` com `pandas`
5. Insere os dados em lote via `cursor.executemany()`
6. Valida a carga com `SELECT COUNT(*)` por tabela

## Tecnologias

- `pyodbc` — driver ODBC para SQL Server
- `pandas` — leitura dos CSVs e manipulação de dados
- `python-dotenv` — leitura das credenciais do `.env`

## Dependências

```
SQL Server container rodando (docker compose up -d)
ODBC Driver 18 for SQL Server instalado
Arquivos CSV em data/
```

## Resultado esperado

```
Tabela          Registros
-----------------------
apolice                20
carro                  20
cliente                20
endereco               20
estado                 27
marca                  10
modelo                 30
municipio              30
regiao                  5
sinistro               10
telefone               20
```

## Execução idempotente

O notebook verifica antes de inserir se a tabela já contém dados. Se sim, pula a tabela. Isso permite reexecutar o notebook sem duplicar dados.
