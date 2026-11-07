# 📖 Dicionário de Dados — Mobilidade Urbana Recife 2024

Este documento descreve a estrutura dos bancos de dados SQLite gerados pelos 5 pipelines ETL do projeto. O modelo adotado é o **Dimensional (Star Schema)**, composto por Tabelas Fato (métricas e eventos) e Tabelas Dimensão (atributos descritivos).

> **Nota Técnica (Geoespacial):** O SQLite não possui suporte nativo a tipos de dados espaciais (como o PostGIS). Para garantir a compatibilidade e a portabilidade dos dados, as geometrias (pontos e linhas) foram convertidas para o formato padrão **WKT (Well-Known Text)** e armazenadas como texto.

---

## 1. ETL 1: Chamados de Sinistros (`mobilidade_recife.db`)

### Tabela Fato: `fato_chamados`
Armazena os registros individuais dos chamados de sinistros de trânsito.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `protocolo` | TEXT | Identificador único (chave primária natural) do chamado. |
| `id_tempo` | INTEGER | Chave estrangeira para `dim_tempo`. |
| `id_bairro` | INTEGER | Chave estrangeira para `dim_bairro`. |
| `data_evento` | TEXT | Data do ocorrido (YYYY-MM-DD). |
| `hora` | TEXT | Hora exata do ocorrido (HH:MM:SS). |
| `faixa_horaria` | TEXT | Faixa de hora agrupada (ex: "00h-05h"). |
| `mes_num` | INTEGER | Número do mês (1-12). |
| `mes` | TEXT | Nome do mês (ex: "Janeiro"). |
| `dia_semana` | TEXT | Dia da semana (ex: "Segunda-feira"). |
| `bairro` | TEXT | Nome do bairro onde ocorreu o sinistro. |
| `tipo` | TEXT | Tipo de sinistro (ex: "Colisão", "Atropelamento"). |
| `vitimas` | INTEGER | Quantidade total de vítimas no evento. |
| `vitimasfatais`| INTEGER | Quantidade de vítimas fatais no evento. |
| `resource_id` | TEXT | ID do recurso na API CKAN de origem. |

### Tabelas Dimensão
- **`dim_tempo`:** (`id_tempo`, `data_evento`, `mes_num`, `mes`, `dia_semana`, `faixa_horaria`)
- **`dim_bairro`:** (`id_bairro`, `Nome_Bairro_Oficial`)
- **`dim_fonte`:** (`id_fonte`, `fonte`, `conjunto`, `data_extracao`, `formato`)

---

## 2. ETL 2: Infrações de Trânsito (`infracoes_recife.db`)

### Tabela Fato: `fato_infracoes`
Armazena os registros de infrações de trânsito, enriquecidos com dados de tempo e local.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id_tempo` | INTEGER | Chave estrangeira para `dim_tempo`. |
| `id_infracao` | INTEGER | Chave estrangeira para `dim_infracao`. |
| `id_local` | INTEGER | Chave estrangeira para `dim_local`. |
| `data_infracao` | TIMESTAMP | Data e hora completa da infração. |
| `hora_infracao` | TEXT | Hora extraída da infração. |
| `faixa_horaria` | TEXT | Faixa de hora agrupada. |
| `nome_mes` | TEXT | Nome do mês formatado. |
| `dia_semana` | TEXT | Dia da semana. |
| `infracao` | INTEGER | Código numérico da infração. |
| `descricao_infracao`| TEXT | Descrição textual da infração cometida. |
| `amparo_legal` | TEXT | Artigo da lei que ampara a infração. |
| `local_cometimento` | TEXT | Endereço ou local exato do cometimento. |
| `bairro` | TEXT | Bairro onde a infração foi registrada. |
| `agente_equipamento`| TEXT | Tipo de agente ou equipamento que autuou. |

### Tabelas Dimensão
- **`dim_tempo`:** (`id_tempo`, `data_infracao`, `mes_num`, `nome_mes`, `dia_semana_num`, `dia_semana`, `faixa_horaria`)
- **`dim_infracao`:** (`id_infracao`, `infracao`, `descricao_infracao`, `amparo_legal`)
- **`dim_local`:** (`id_local`, `bairro`)
- **`dim_fonte`:** (`id_fonte`, `fonte`, `conjunto`, `ano`, `resource_id`, `data_extracao`, `formato`)

---

## 3. ETL 3: Malha Cicloviária (`ciclovias_recife.db`)

### Tabela Fato: `fato_segmentos_cicloviarios`
Armazena os segmentos da malha cicloviária recortados pelos limites dos bairros.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id_segmento` | INTEGER | Identificador único do segmento recortado. |
| `id_bairro` | INTEGER | Chave estrangeira para `dim_bairro`. |
| `id_tipo` | INTEGER | Chave estrangeira para `dim_tipo`. |
| `id_fonte` | INTEGER | Chave estrangeira para `dim_fonte`. |
| `Tipo_Limpo` | TEXT | Tipo específico (ex: "ciclovia", "ciclorrota"). |
| `Categoria` | TEXT | Categoria geral ("Permanente" ou "Operacional"). |
| `Nome_Bairro_Oficial`| TEXT | Nome do bairro onde o segmento está localizado. |
| `comprimento_km` | REAL | Comprimento do segmento em quilômetros. |
| `geometry_wkt` | TEXT | Geometria da linha no formato WKT (EPSG:31985 original). |

### Tabelas Dimensão
- **`dim_bairro`:** (`id_bairro`, `Nome_Bairro_Oficial`, `area_km2`)
- **`dim_tipo`:** (`id_tipo`, `Tipo_Limpo`, `Categoria`)
- **`dim_fonte`:** (`id_fonte`, `fonte`, `conjunto`, `data_extracao`, `formato`)

---

## 4. ETL 4: Estações de Bikes (`estacoes_recife.db`)

### Tabela Fato: `fato_estacoes`
Armazena as estações de bicicletas e sua relação de proximidade com a malha permanente.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id_estacao` | INTEGER | Identificador único da estação. |
| `id_bairro` | INTEGER | Chave estrangeira para `dim_bairro`. |
| `id_fonte` | INTEGER | Chave estrangeira para `dim_fonte`. |
| `codigo` | TEXT | Código original da estação (tratado como texto). |
| `capacidade` | INTEGER | Capacidade total de bicicletas da estação. |
| `latitude` | REAL | Latitude original (WGS84). |
| `longitude` | REAL | Longitude original (WGS84). |
| `Nome_Bairro_Oficial`| TEXT | Nome do bairro de localização. |
| `distancia_m` | REAL | Distância em metros até a malha permanente mais próxima. |
| `proximidade_300m`| INTEGER/BOOL | Flag (1/0) indicando se está a ≤ 300m da malha. |
| `geometry_wkt` | TEXT | Geometria do ponto no formato WKT. |

### Tabelas Dimensão
- **`dim_bairro`:** (`id_bairro`, `Nome_Bairro_Oficial`, `area_km2`)
- **`dim_fonte`:** (`id_fonte`, `fonte`, `conjunto`, `data_extracao`, `formato`)

---

## 5. ETL 5: Equipamentos de Fiscalização (`etl5_equipamentos.db`)

### Tabela Principal: `dim_equipamento`
Neste ETL, a tabela de detalhe armazena os 87 equipamentos cadastrados.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id_equipamento` | TEXT | Identificador único do equipamento. |
| `tipo_equipamento` | TEXT | Tipo do equipamento (ex: "Fixo"). |
| `local_instalacao` | TEXT | Endereço ou referência de instalação. |
| `latitude` | REAL | Latitude do equipamento. |
| `longitude` | REAL | Longitude do equipamento. |
| `coord_valida` | INTEGER/BOOL | Flag de validação da bounding box do Recife. |
| `bairro` | TEXT | Bairro onde o equipamento está instalado. |
| `geometry_wkt` | TEXT | Geometria do ponto no formato WKT. |

### Tabelas Agregadas (Views de Negócio)
- **`agg_equip_bairro`:** (`bairro`, `quantidade_equipamentos`) - Total de equipamentos por bairro.
- **`agg_equip_tipo`:** (`tipo_equipamento`, `quantidade`) - Total de equipamentos por tipo.