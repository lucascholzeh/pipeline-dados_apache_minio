# Como Executar o Projeto

## Pré-requisitos

Antes de começar, instale:

| Ferramenta | Versão mínima | Link |
|---|---|---|
| Docker + Docker Compose | v2+ | [docs.docker.com](https://docs.docker.com) |
| Python | 3.11 | [python.org](https://python.org) |
| Java (OpenJDK) | 11 | `sudo apt install openjdk-11-jdk` |
| UV (gestor Python) | latest | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| ODBC Driver 18 | 18 | Instruções abaixo |

## 1. Clonar o repositório

```bash
git clone <url-do-repositorio>
cd pipeline-dados
```

## 2. Subir os containers

```bash
docker compose up -d
```

Verifique se os containers estão saudáveis:

```bash
docker compose ps
```

Saída esperada:

```
NAME             STATUS          PORTS
sqlserver-2022   running         0.0.0.0:1433->1433/tcp
minio            running         0.0.0.0:9020->9000/tcp, 0.0.0.0:9021->9021/tcp
```

## 3. Configurar o ambiente

```bash
cp .env.example .env
```

## 4. Instalar UV (se ainda não tiver)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env   # ativa o UV no shell atual
```

## 5. Criar ambiente virtual e instalar dependências

```bash
uv venv
source .venv/bin/activate
uv sync
```

## 6. Instalar ODBC Driver (Ubuntu/WSL)

```bash
sudo apt install -y unixodbc-dev

curl https://packages.microsoft.com/keys/microsoft.asc \
  | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc

sudo curl https://packages.microsoft.com/config/ubuntu/24.04/prod.list \
  | sudo tee /etc/apt/sources.list.d/mssql-release.list

sudo apt update
sudo ACCEPT_EULA=Y apt install -y msodbcsql18
```

Validar:

```bash
odbcinst -q -d
# Deve retornar: [ODBC Driver 18 for SQL Server]
```

## 7. Configurar Java

```bash
sudo apt install -y openjdk-11-jdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
```

Validar:

```bash
java -version
# openjdk version "11.0.x"
```

## 8. Iniciar o JupyterLab

```bash
jupyter lab
```

Acesse: [http://localhost:8888](http://localhost:8888)

## 9. Executar os notebooks em ordem

!!! danger "Ordem obrigatória"
    Execute os notebooks **estritamente na ordem** indicada. Cada um depende do anterior.

| Ordem | Notebook | Tempo estimado |
|---|---|---|
| 1 | `notebook/00_setup_sqlserver.ipynb` | 1 min |
| 2 | `notebook/01_sqlserver_to_minio_csv.ipynb` | 1 min |
| 3 | `notebook/02_csv_to_delta.ipynb` | 3-5 min (download JARs na 1ª vez) |
| 4 | `notebook/03_dml_delta.ipynb` | 2-3 min |

### Selecionar o kernel correto

No JupyterLab, antes de executar qualquer notebook:

1. Clique no nome do kernel (canto superior direito)
2. Selecione **Python 3 (.venv)**

## 10. Verificar resultados

### Console MinIO
- Acesse: [http://localhost:9021](http://localhost:9021)
- Login: `minioadmin` / `minioadmin`
- Verifique os buckets `landing-zone` e `bronze`

## Solução de problemas comuns

### Spark não consegue conectar ao MinIO

```
java.net.ConnectException: Connection refused (localhost:9020)
```

**Causa**: Container MinIO não está rodando.  
**Solução**: `docker compose up -d minio`

### ODBC Driver não encontrado

```
pyodbc.Error: ('01000', "[01000] [unixODBC][Driver Manager]...")
```

**Causa**: ODBC Driver 18 não instalado.  
**Solução**: Seguir o passo 6 acima.

### JARs Delta/Hadoop não baixam

```
org.apache.spark.SparkException: Failed to get main class...
```

**Causa**: Sem acesso à internet ou Maven indisponível.  
**Solução**: Verificar conexão e aguardar — o download pode demorar alguns minutos.

### SQL Server não aceita conexão

```
pyodbc.OperationalError: Login timeout expired
```

**Causa**: SQL Server ainda iniciando (pode demorar 30-60s).  
**Solução**: Aguardar e tentar novamente.
