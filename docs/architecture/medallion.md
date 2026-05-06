# Arquitetura Medalhão

## O que é a Arquitetura Medalhão?

A **Arquitetura Medalhão** (Medallion Architecture) é um padrão de design para organizar dados em um data lake em camadas progressivas de qualidade e processamento. O nome vem das camadas: **Bronze, Silver e Gold** — como medalhas olímpicas.

```
Fonte de Dados
     │
     ▼
  BRONZE  ←── Dados brutos, formato Delta Lake
     │
     ▼
  SILVER  ←── Dados limpos e validados
     │
     ▼
   GOLD   ←── Dados agregados, prontos para consumo
```

## Camadas implementadas neste projeto

Neste trabalho implementamos as duas primeiras camadas:

### Landing Zone (Pré-Bronze)

!!! info "Camada Landing Zone"
    Dados exatamente como saem da fonte, sem qualquer transformação.

- **Bucket MinIO**: `landing-zone`
- **Formato**: CSV (para banco relacional)
- **Conteúdo**: Uma arquivo `.csv` por tabela do SQL Server
- **Acesso**: Boto3 (upload direto, sem Spark)

**Por que CSV?**
> A recomendação do trabalho define CSV para fontes relacionais. É o formato de mais baixo esforço para extração — pandas/pyodbc exportam nativamente para CSV, e qualquer ferramenta consegue ler.

### Bronze

!!! success "Camada Bronze"
    Dados brutos convertidos para formato analítico com ACID e versionamento.

- **Bucket MinIO**: `bronze`
- **Formato**: Delta Lake (Parquet + `_delta_log`)
- **Conteúdo**: Uma pasta Delta por tabela
- **Acesso**: Apache Spark com delta-spark

**Por que Delta Lake em vez de Parquet puro?**
> Parquet puro não suporta UPDATE ou DELETE de forma eficiente — é necessário reescrever o arquivo inteiro. Delta Lake adiciona um transaction log que permite operações ACID granulares, versionamento automático e time travel.

## Estrutura no MinIO após execução completa

```
landing-zone/
├── regiao.csv
├── estado.csv
├── municipio.csv
├── marca.csv
├── modelo.csv
├── cliente.csv
├── endereco.csv
├── telefone.csv
├── carro.csv
├── apolice.csv
└── sinistro.csv

bronze/
├── regiao/
│   ├── part-00000-*.parquet
│   └── _delta_log/
│       ├── 00000000000000000000.json   ← versão 0: WRITE
│       └── ...
├── marca/
│   ├── part-00000-*.parquet
│   └── _delta_log/
│       ├── 00000000000000000000.json   ← versão 0: WRITE
│       ├── 00000000000000000001.json   ← versão 1: INSERT
│       ├── 00000000000000000002.json   ← versão 2: UPDATE
│       └── 00000000000000000003.json   ← versão 3: DELETE
└── ...
```

## Por que não implementamos Silver e Gold?

O escopo do trabalho cobre a extração e a camada Bronze com DML. As camadas Silver (limpeza/validação) e Gold (agregações de negócio) são etapas naturais de evolução do projeto, mas estão fora do requisito desta entrega.
