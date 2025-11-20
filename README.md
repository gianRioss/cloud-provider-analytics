# Análisis de Proveedores de Nube  
**ETL Batch + Streaming (PySpark) → Parquet (Bronze/Silver/Gold) → Serving en AstraDB (Cassandra)**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/<ID_DE_TU_NOTEBOOK>)  
![Python](https://img.shields.io/badge/Python-3.10+-blue)
![PySpark](https://img.shields.io/badge/PySpark-3.5.x-orange)
![Cassandra](https://img.shields.io/badge/Cassandra-AstraDB-4B0082)
![Architecture](https://img.shields.io/badge/Architecture-Lambda-00aa88)
![Status](https://img.shields.io/badge/Status-Ready-green)

Proyecto de ingeniería de datos para un **cloud provider**: ingesta, limpieza, conformado y publicación de datos para **FinOps, Soporte y Producto (GenAI)**.  
- **Batch**: maestros (orgs, users, resources), billing mensual, NPS.  
- **Streaming**: eventos de uso (`usage_events_stream/*.jsonl`) con *watermark*, *dedup* e idempotencia.  
- **Storage intermedio**: Parquet particionado por fecha/servicio.  
- **Gold marts** servidos en **AstraDB (Cassandra)**, listos para BI.  
- **Detección de anomalías**: p99 / z-score / MAD.  

---

## 🧭 Diagrama (Lambda)

mermaid
flowchart LR
  A[Landing (CSV/JSONL)] --> B[Bronze (Parquet)]
  subgraph Batch
    A1[customers_orgs.csv]
    A2[users.csv]
    A3[resources.csv]
    A4[support_tickets.csv]
    A5[marketing_touches.csv]
    A6[nps_surveys.csv]
    A7[billing_monthly.csv]
  end
  subgraph Streaming
    S1[usage_events_stream/*.jsonl]
  end
  A1 --> B
  A2 --> B
  A3 --> B
  A4 --> B
  A5 --> B
  A6 --> B
  A7 --> B
  S1 -->|Structured Streaming (schema v1/v2, watermark, dedup)| B
  B --> C[Silver (Conformado + Enriquecido)]
  C --> D[Gold (Marts)]
  D --> E[(AstraDB / Cassandra)]

Estructura
.
├── CQL/
│   ├── ddl_keyspace_tables.cql       # Keyspace + tablas Gold en Cassandra
│   └── demo_queries.cql              # 5 consultas demo (CQL)
├── Docs/
│   ├── arquitectura.png              # (opcional) diagrama/export desde Mermaid
│   └── diccionario_campos.md         # (opcional)
├── Evidencias/
│   ├── streaming_ok.png
│   ├── astra_tablas.png
│   └── consultas_demo.png
├── Notebook.ipynb                    # pipeline completo (Colab)
├── env.sample                        # variables ejemplo (sin secretos)
├── .gitignore
└── README.md

  E --> F[Dashboards / CQL / BI]

