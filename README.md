# 🚦 Mobilidade Urbana e Segurança Viária no Recife (2024)

Projeto prático da disciplina de Data Science (SENAC Pernambuco), desenvolvido pelo **Aluno 4** em equipe de 4 integrantes. O objetivo é integrar cinco fontes de dados abertos da Prefeitura do Recife para apoiar o planejamento da mobilidade urbana e da segurança viária.

## 📊 Fontes de Dados

| Ref. | Conjunto | Formato | Registros |
|------|----------|---------|-----------|
| S1 | Chamados de Sinistros de Trânsito | CSV | 404.975 |
| S2 | Registro das Infrações de Trânsito | CSV | 404.975 |
| S3 | Malha Cicloviária | GeoJSON | 79 segmentos (186,09 km) |
| S4 | Estações de Bikes | CSV | 90 estações |
| S5 | Equipamentos de Fiscalização | CSV | 87 equipamentos |
| S6 | Limites Territoriais (bairros) | GeoJSON | 94 polígonos |

**API utilizada:** CKAN — https://dados.recife.pe.gov.br/api/3/action/

## 🛠️ Tecnologias

- Python 3.13 (Google Colab)
- Pandas, GeoPandas, Shapely, PyProj
- Matplotlib, Seaborn, Plotly
- SQLite3 (modelo dimensional)
- Streamlit / HTML estático (dashboard)

## 📁 Estrutura do Projeto

```text
Projeto_Mobilidade_Recife/
├── .gitignore
├── README.md
├── requirements.txt
├── ETLs/
│   ├── ETL_01_Chamados.ipynb
│   ├── ETL_02_Infracoes.ipynb
│   ├── ETL_03_Malha_Cicloviaria.ipynb
│   ├── ETL_04_Estacoes_Bikes.ipynb
│   └── ETL_05_Equipamentos_Fiscalizacao.ipynb
├── Documentacao/
│   ├── Dicionario_de_Dados.md
│   ├── Modelo_de_Banco.png
│   └── Relatorio_Qualidade/
├── Dashboard/
│   └── dashboard_mobilidade_recife.html
└── Relatorio_Final.pdf
```

## ▶️ Instruções de Execução

### 1. ETLs (Notebooks)
Cada notebook é autocontido e deve ser executado **em ordem, do Bloco 1 ao último**.

- **Primeira execução:** todos os blocos são processados (download, limpeza, interseção espacial, carga).
- **Execuções seguintes:** o **Bloco de Restauração** (presente no topo de cada notebook) detecta o banco SQLite no Google Drive e carrega os dados em segundos, permitindo pular direto para as análises.

**Pré-requisitos:**
- Conta Google (para montar o Drive)
- Python 3.10+ com as bibliotecas listadas em `requirements.txt`

### 2. Banco de Dados
Os 5 bancos SQLite são gerados automaticamente durante a execução dos ETLs e salvos no Google Drive:

| ETL | Banco | Tabelas principais |
|-----|-------|--------------------|
| 1 | `mobilidade_recife.db` | `fato_chamados`, `dim_bairro`, `dim_tempo`, `dim_fonte` |
| 2 | `infracoes_recife.db` | `fato_infracoes`, `dim_infracao`, `dim_local`, `dim_tempo`, `dim_fonte` |
| 3 | `ciclovias_recife.db` | `fato_segmentos_cicloviarios`, `dim_bairro`, `dim_tipo`, `dim_fonte` |
| 4 | `estacoes_recife.db` | `fato_estacoes`, `dim_bairro`, `dim_fonte` |
| 5 | `etl5_equipamentos.db` | `dim_equipamento`, `agg_equip_bairro`, `agg_equip_tipo` |

### 3. Dashboard
Abra o arquivo `Dashboard/dashboard_mobilidade_recife.html` em qualquer navegador moderno (Chrome, Edge, Firefox). **Não requer servidor local** — é um dashboard HTML estático com Plotly via CDN.

Funcionalidades:
- 5 abas temáticas (uma por ETL)
- KPIs resumidos no topo
- Gráficos interativos (zoom, hover, download PNG)
- Gráfico de dispersão integrado (ETL 1 × ETL 5)

### 4. Relatório Final
Consulte o arquivo `Relatorio_Final.pdf` para a análise descritiva completa, incluindo método, 3 achados sustentados por dados, 2 recomendações de política pública e limitações.

## 🔒 Segurança e Privacidade

- Nenhuma credencial (senhas, tokens privados) está hardcoded neste repositório.
- A extração utiliza apenas a **API pública** do Portal de Dados Abertos do Recife.
- Os dados são públicos e anonimizados na origem.

## ⚠️ Aviso Analítico

Este projeto apresenta **análises descritivas**. Dados cadastrais ≠ dados operacionais. A presença de infraestrutura ou equipamentos **não implica relação causal** com a ocorrência de sinistros. Recomenda-se cautela na interpretação dos resultados.

**Professor:** Heuryk Wylk  
**Instituição:** SENAC Pernambuco  
**Data:** Novembro de 2026