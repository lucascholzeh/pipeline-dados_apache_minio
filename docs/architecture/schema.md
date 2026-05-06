# Schema do Banco de Dados — SeguroDB

## Diagrama Entidade-Relacionamento

```
regiao (5)
  └── estado (27)
        └── municipio (30)
              └── endereco (20) ──────────────────────┐
                                                      │
marca (10)                                        cliente (20)
  └── modelo (30)                                     │
        └── carro (20)                                │
               └── apolice (20) ◄────────────────────┘
                       └── sinistro (10)

telefone (20) ──► cliente (20)
```

## Tabelas

### `regiao`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_regiao | INT (PK) | Identificador |
| nome_regiao | VARCHAR(50) | Norte, Sul, etc. |

### `estado`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_estado | INT (PK) | Identificador |
| nome_estado | VARCHAR(100) | Nome completo |
| uf | CHAR(2) | Sigla (SP, RJ, ...) |
| id_regiao | INT (FK) | Região |

### `municipio`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_municipio | INT (PK) | Identificador |
| nome_municipio | VARCHAR(150) | Nome da cidade |
| id_estado | INT (FK) | Estado |

### `marca`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_marca | INT (PK) | Identificador |
| nome_marca | VARCHAR(100) | Ex: Volkswagen |
| pais_origem | VARCHAR(50) | País fabricante |

### `modelo`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_modelo | INT (PK) | Identificador |
| nome_modelo | VARCHAR(100) | Ex: Gol, Civic |
| id_marca | INT (FK) | Marca |
| categoria | VARCHAR(50) | Hatch, Sedan, SUV, Picape |

### `cliente`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_cliente | INT (PK) | Identificador |
| nome | VARCHAR(150) | Nome completo |
| cpf | VARCHAR(14) | CPF (único) |
| data_nascimento | DATE | Data de nascimento |
| email | VARCHAR(200) | E-mail |
| sexo | CHAR(1) | M / F |

### `endereco`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_endereco | INT (PK) | Identificador |
| id_cliente | INT (FK) | Cliente |
| logradouro | VARCHAR(200) | Rua/Avenida |
| numero | VARCHAR(10) | Número |
| complemento | VARCHAR(100) | Apto, Casa, etc. |
| bairro | VARCHAR(100) | Bairro |
| cep | VARCHAR(10) | CEP |
| id_municipio | INT (FK) | Município |

### `telefone`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_telefone | INT (PK) | Identificador |
| id_cliente | INT (FK) | Cliente |
| ddd | CHAR(2) | DDD |
| numero | VARCHAR(15) | Número |
| tipo | VARCHAR(20) | Celular / Residencial |

### `carro`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_carro | INT (PK) | Identificador |
| id_modelo | INT (FK) | Modelo |
| placa | VARCHAR(8) | Placa (único) |
| chassi | VARCHAR(30) | Chassi (único) |
| ano_fabricacao | INT | Ano |
| cor | VARCHAR(30) | Cor |
| combustivel | VARCHAR(20) | Flex / Gasolina / Diesel |

### `apolice`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_apolice | INT (PK) | Identificador |
| id_cliente | INT (FK) | Cliente segurado |
| id_carro | INT (FK) | Veículo segurado |
| numero_apolice | VARCHAR(20) | Número único |
| data_inicio | DATE | Início da vigência |
| data_fim | DATE | Fim da vigência |
| valor_cobertura | DECIMAL(12,2) | Valor máximo coberto |
| valor_franquia | DECIMAL(10,2) | Franquia |
| status | VARCHAR(20) | Ativa / Expirada |

### `sinistro`
| Coluna | Tipo | Descrição |
|---|---|---|
| id_sinistro | INT (PK) | Identificador |
| id_apolice | INT (FK) | Apólice vinculada |
| data_ocorrencia | DATE | Data do evento |
| tipo_sinistro | VARCHAR(50) | Colisão, Roubo, Incêndio... |
| descricao | VARCHAR(500) | Descrição do evento |
| valor_prejuizo | DECIMAL(12,2) | Valor do dano |
| status | VARCHAR(30) | Em análise / Encerrado |
