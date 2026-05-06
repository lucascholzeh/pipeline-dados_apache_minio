# Pipeline de Dados: SQL Server + Apache Spark + Delta Lake + MinIO

Projeto desenvolvido para o curso de **Engenharia de Dados** — demonstra a construção de um pipeline completo de dados utilizando SQL Server como fonte, MinIO como object storage e Delta Lake como formato de armazenamento lakehouse.

---

## Visão Geral

```
┌─────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│   SQL Server    │────▶│    MinIO (S3)        │────▶│    MinIO (S3)        │
│   2022 Dev      │     │    landing-zone/      │     │    bronze/           │
│                 │     │    (CSVs)             │     │    (Delta Tables)    │
│   SeguroDB      │     │                      │     │                      │
│   11 tabelas    │     │   1 CSV por tabela   │     │   INSERT / UPDATE    │
│                 │     │                      │     │   DELETE / HISTORY   │
└─────────────────┘     └──────────────────────┘     └──────────────────────┘
     Notebook 00              Notebook 01                Notebooks 02/03
      (Setup)                  (Extração)               (Delta + DML)
```

### O que este projeto demonstra

- Extração de dados de banco relacional (SQL Server) com `pyodbc` e `pandas`
- Object Storage como repositório de dados intermediário (MinIO / S3-compatible)
- Conversão de CSV para **Delta Lake** com Apache Spark
- Transações **ACID** em data lakes
- Operações **DML** (INSERT, UPDATE, DELETE) em tabelas Delta
- **Versionamento** de dados com History e **Time Travel**
- Arquitetura Medalhão: **Landing Zone → Bronze**

---

## Tecnologias

| Tecnologia | Versão | Papel no projeto |
|---|---|---|
| Apache Spark (PySpark) | 3.5.3 | Motor de processamento distribuído |
| Delta Lake | 3.2.0 | Formato lakehouse com ACID e Time Travel |
| MinIO | 2025-02-03 | Object Storage compatível com S3 |
| SQL Server | 2022 Developer | Banco de dados relacional fonte |
| Docker Compose | v2+ | Orquestração dos containers |
| Python | 3.11 | Linguagem principal |
| pyodbc | 5.1+ | Conector ODBC para SQL Server |
| boto3 | 1.34+ | SDK AWS/S3 para upload ao MinIO |

---

## Pré-requisitos

- **Linux** (Ubuntu 24.04) ou **WSL2** no Windows 11
- **Docker** e **Docker Compose v2+**
- **Python 3.11** (PySpark 3.5.3 requer Python ≤ 3.12)
- **Java 11** (OpenJDK)
- **UV** (gerenciador de pacotes Python)
- **ODBC Driver 18 for SQL Server**

---

## Setup do Ambiente

### 1. Clonar o repositório e configurar variáveis de ambiente

```bash
git clone <url-do-repositorio>
cd pipeline-dados
cp .env.example .env
```

> O arquivo `.env` contém as credenciais do SQL Server e do MinIO. Os valores padrão do `.env.example` já funcionam para o ambiente local com Docker.

### 2. Subir os containers (SQL Server + MinIO)

```bash
docker compose up -d
```

Containers criados:

| Container | Imagem | Portas |
|---|---|---|
| sqlserver-2022 | mcr.microsoft.com/mssql/server:2022-latest | 1433 |
| minio | minio/minio:RELEASE.2025-02-03T21-03-04Z | 9020 (API), 9021 (Console) |

Credenciais padrão:

| Serviço | Usuário | Senha |
|---|---|---|
| SQL Server | sa | SqlServer@2022! |
| MinIO | minioadmin | minioadmin |

> Console MinIO: [http://localhost:9021](http://localhost:9021)

### 3. Instalar UV (se ainda não tiver)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
```

### 4. Criar ambiente virtual e instalar dependências

```bash
uv venv
source .venv/bin/activate
uv sync
```

### 5. Instalar ODBC Driver for SQL Server (Ubuntu/WSL)

```bash
sudo apt install -y unixodbc-dev
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
sudo curl https://packages.microsoft.com/config/ubuntu/24.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
sudo apt update
sudo ACCEPT_EULA=Y apt install -y msodbcsql18
```

Validar instalação:

```bash
odbcinst -q -d
# Deve retornar: [ODBC Driver 18 for SQL Server]
```

### 6. Instalar Java 11

```bash
sudo apt install -y openjdk-11-jdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

---

## Executando o Projeto

Execute os notebooks na ordem indicada. Antes de executar, selecione o ambiente virtual `.venv` como Kernel do Jupyter.

| # | Notebook | Descrição |
|---|---|---|
| 0 | `00_setup_sqlserver.ipynb` | Cria o banco `SeguroDB` e carrega as 11 tabelas com dados de exemplo |
| 1 | `01_sqlserver_to_minio_csv.ipynb` | Extrai todas as tabelas → CSV no MinIO (bucket `landing-zone`) |
| 2 | `02_csv_to_delta.ipynb` | Lê os CSVs com Spark e converte para Delta Lake (bucket `bronze`) |
| 3 | `03_dml_delta.ipynb` | Executa INSERT, UPDATE, DELETE; exibe HISTORY e TIME TRAVEL |

```bash
# Iniciar o JupyterLab
jupyter lab
```

---

## Estrutura do Projeto

```
pipeline-dados/
├── docker-compose.yml               # SQL Server 2022 + MinIO
├── .env.example                     # Template de variáveis de ambiente
├── .env                             # Variáveis locais (não versionado)
├── pyproject.toml                   # Dependências Python (UV)
├── .python-version                  # Python 3.11
├── README.md
├── data/                            # CSVs de dados de exemplo
│   ├── regiao.csv
│   ├── estado.csv
│   ├── municipio.csv
│   ├── marca.csv
│   ├── modelo.csv
│   ├── cliente.csv
│   ├── endereco.csv
│   ├── telefone.csv
│   ├── carro.csv
│   ├── apolice.csv
│   └── sinistro.csv
├── notebook/                        # Notebooks Jupyter
│   ├── 00_setup_sqlserver.ipynb     # Setup: CSV → SQL Server
│   ├── 01_sqlserver_to_minio_csv.ipynb  # Extração: SQL Server → MinIO (CSV)
│   ├── 02_csv_to_delta.ipynb        # Conversão: CSV → Delta Lake
│   └── 03_dml_delta.ipynb           # DML: INSERT, UPDATE, DELETE + History
└── docs/                            # Documentação MkDocs
    ├── mkdocs.yml
    └── docs/
```

---

## Schema do Banco de Dados (SeguroDB)

Banco relacional com domínio de **seguradora de veículos** — 11 tabelas:

```
regiao ──► estado ──► municipio ──► endereco ──► cliente
                                                    │
marca ──► modelo ──► carro ──► apolice ◄────────────┘
                                   │
                               sinistro
```

| Tabela | Registros | Descrição |
|---|---|---|
| regiao | 5 | Regiões do Brasil |
| estado | 27 | Estados + UF |
| municipio | 30 | Municípios |
| marca | 10 | Marcas de veículos |
| modelo | 30 | Modelos de veículos |
| cliente | 20 | Clientes da seguradora |
| endereco | 20 | Endereços dos clientes |
| telefone | 20 | Telefones dos clientes |
| carro | 20 | Veículos segurados |
| apolice | 20 | Apólices de seguro |
| sinistro | 10 | Sinistros registrados |

---

## Conceitos Demonstrados

### Delta Lake — Transações ACID

Diferente de arquivos Parquet simples, o Delta Lake mantém um `_delta_log` — um diretório de arquivos JSON que registra cada operação (commit). Isso garante:

- **Atomicidade**: a escrita é tudo ou nada
- **Consistência**: o schema é validado em cada escrita
- **Isolamento**: leituras não veem escritas parciais
- **Durabilidade**: os dados não se perdem após confirmação

### Time Travel

```python
# Ler a versão original (antes de qualquer DML)
df_v0 = spark.read.format("delta").option("versionAsOf", 0).load("s3a://bronze/marca")

# Ler por timestamp
df_ts = spark.read.format("delta").option("timestampAsOf", "2024-01-01").load("s3a://bronze/marca")
```

### Arquitetura Medalhão

| Camada | Bucket | Formato | Descrição |
|---|---|---|---|
| Landing Zone | `landing-zone` | CSV | Dados brutos extraídos da fonte |
| Bronze | `bronze` | Delta Lake | Dados brutos em formato analítico com ACID |

---

## Referências

- [Delta Lake Documentation](https://docs.delta.io)
- [PySpark Documentation](https://spark.apache.org/docs/3.5.3/api/python/)
- [MinIO Documentation](https://min.io/docs/minio/linux/index.html)
- [SQL Server 2022 Docker](https://hub.docker.com/_/microsoft-mssql-server)
- [Repositório de referência](https://github.com/jlsilva01/spark-delta-minio-sqlserver)
