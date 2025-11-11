# 🚌 Projeto Databricks - Pipeline SPTrans  
**Thiago Chagas e Isaque Freitas**

---

## 🧱 Arquitetura do Pipeline

| Camada | Notebook | Descrição |
|:--|:--|:--|
| 🟤 **Bronze** | `Notebook Bronze.py` | Coleta os dados brutos da API SPTrans e grava em JSON sob `/Volumes/main/labdat/bronze/sptrans/posicao/`. |
| ⚪ **Silver A** | `Notebook Silver A.py` | Lê os arquivos Bronze, realiza *flatten* das estruturas aninhadas e cria a tabela `main.labdat.silver_sptrans_posicao`. |
| ⚪ **Silver B** | `Notebook Silver B.py` | Enriquece os dados com coordenadas geográficas, gera *geohash* e tabela de dimensão `main.labdat.silver_sptrans_dim_geoloc`. |
| ⚪ **Silver C (KPIs)** | `Notebook Silver C.py` | Calcula KPIs operacionais — veículos ativos, *headway* médio e velocidade média — consolidando em `main.labdat.silver_sptrans_kpis`. |
| 🟡 **Gold A/B/C** | `Notebook Gold A.py / B.py / C.py` | Constrói dimensões (`dim_linha`, `dim_tempo`, `dim_geoloc`) e o fato operacional `main.labdat.gold_sptrans_fato_operacional`. |

---

## ⚙️ Detalhes Técnicos

- **Fonte de dados:** API pública [Olho Vivo - SPTrans](http://api.olhovivo.sptrans.com.br/v2.1)  
- **Armazenamento:** Unity Catalog (`main.labdat`) com tabelas Delta Lake  
- **Linguagem:** PySpark (Databricks Runtime 13.x)  
- **Camadas:** Bronze → Silver → Gold  
- **Particionamento:** por `date`  
- **Ingestão:** incremental via `MERGE INTO`  
- **Segurança:** tokens e segredos armazenados no `dbutils.secrets`  

---

## 🗂️ Estrutura de Pastas
LABDAT/
│
├── 📂 bronze/
│ ├── Notebook Bronze.py
│ └── /Volumes/main/labdat/bronze/sptrans/posicao/
│
├── 📂 silver/
│ ├── Notebook Silver A.py
│ ├── Notebook Silver B.py
│ ├── Notebook Silver C.py
│ ├── main.labdat.silver_sptrans_posicao
│ ├── main.labdat.silver_sptrans_posicao_geo
│ ├── main.labdat.silver_sptrans_dim_geoloc
│ └── main.labdat.silver_sptrans_kpis
│
└── 📂 gold/
├── Notebook Gold A.py
├── Notebook Gold B.py
├── Notebook Gold C.py
├── main.labdat.gold_sptrans_dim_tempo
├── main.labdat.gold_sptrans_dim_linha
├── main.labdat.gold_sptrans_dim_geoloc
└── main.labdat.gold_sptrans_fato_operacional



