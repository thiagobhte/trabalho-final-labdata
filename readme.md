# 🚌 Projeto Databricks - Pipeline SPTrans

## 📘 Visão Geral
Este projeto implementa um pipeline **ETL completo em Databricks**, responsável por coletar, tratar e gerar indicadores (KPIs) de mobilidade urbana a partir dos dados da **API Olho Vivo SPTrans**.  

O fluxo foi desenhado em camadas **Bronze → Silver → Gold**, seguindo boas práticas de *Lakehouse* e o padrão **Delta Live Architecture** com **Unity Catalog**.

---

## 🧱 Arquitetura do Pipeline

| Camada | Notebook | Descrição |
|---------|-----------|-----------|
| 🟤 **Bronze** | `Notebook Bronze.py` | Coleta os dados brutos da API SPTrans e grava em JSON sob `/Volumes/main/labdat/bronze/sptrans/posicao/`. |
| ⚪ **Silver A** | `Notebook Silver A.py` | Lê os arquivos Bronze, realiza *flatten* das estruturas aninhadas e cria a tabela `main.labdat.silver_sptrans_posicao`. |
| ⚪ **Silver B** | `Notebook Silver B.py` | Enriquece os dados com coordenadas geográficas, gera *geohash* e tabela de dimensão `main.labdat.silver_sptrans_dim_geoloc`. |
| ⚪ **Silver C (KPIs)** | `Notebook Silver C.py` | Calcula KPIs operacionais: veículos ativos, headway médio e velocidade média, consolidando em `main.labdat.silver_sptrans_kpis`. |

---

## ⚙️ Detalhes Técnicos

### 🔹 Entrada
- **Fonte:** API REST pública [Olho Vivo - SPTrans](http://api.olhovivo.sptrans.com.br/v2.1)
- **Método:** GET `/Posicao`
- **Autenticação:** Token armazenado no **Databricks Secrets** (`scope: sptrans`, `key: token`)

### 🔹 Processamento
- **Linguagem:** PySpark (Databricks Runtime 13.x)
- **Framework:** Delta Lake
- **Particionamento:** por `date`
- **Esquema:** inferido e padronizado em cada camada
- **Atualização incremental:** via `MERGE INTO`

### 🔹 Saída
| Tabela | Descrição |
|---------|------------|
| `main.labdat.silver_sptrans_posicao` | Posição geográfica limpa e padronizada de cada veículo |
| `main.labdat.silver_sptrans_posicao_geo` | Versão enriquecida com `geohash` |
| `main.labdat.silver_sptrans_dim_geoloc` | Dimensão de geolocalização única |
| `main.labdat.silver_sptrans_kpis` | Indicadores de performance (headway, velocidade média, veículos ativos) |

---

## 📊 KPIs Calculados

| Indicador | Cálculo | Observações |
|------------|----------|-------------|
| **Veículos ativos por linha/hora** | `countDistinct(vehicle_id)` agrupado por `line_code`, `hour` | Mede operação em tempo real |
| **Headway médio (min)** | Média do intervalo entre posições consecutivas | Calculado via `Window` e `lag()` |
| **Velocidade média (km/h)** | Distância Haversine / tempo entre pontos consecutivos | Filtragem de outliers > 80 km/h |

---

## 📁 Estrutura de Pastas

