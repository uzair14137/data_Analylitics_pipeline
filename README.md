# data_Analylitics_pipeline
A fully-orchestrated Spark + Airflow pipeline that ingests open municipal data every night, builds a star-schema warehouse, and surfaces ‘What-if’ mobility &amp; weather insights through SQL and lightweight ML.
```mermaid
flowchart TD
  %% ===========  Streaming Sources =============
  subgraph "Kafka Topics"
    profiles((user_profiles))
    posts((user_posts))
    follows((user_follows))
  end

  %% ===========  Core Pipeline =============
  SparkSS["Spark 3.5 • Structured Streaming\nExactly-once (Iceberg v2)"]

  profiles & posts & follows -->|batch 10 s| SparkSS
  SparkSS -->|"upserts\nParquet + Manifests"| MinIO[(MinIO\ns3a://sociallake)]



  Airflow["Airflow 2.8\nDAGs"] -->|schedule & retry| SparkSS

  %% ===========  Serving & BI =============
  Trino["Trino 448"] --> MinIO
  Superset["Apache Superset\n(BI dashboards)"] --> Trino
 
```
  %% ===========  Styles (no commas!) ===========
  classDef spark fill:#FECACA stroke:#B91C1C color:#111
  classDef graph fill:#C7D2FE stroke:#4338CA color:#111
  classDef airflow fill:#FDE68A stroke:#D97706 color:#111
  classDef trino fill:#A5F3FC stroke:#0891B2 color:#111
  classDef viz fill:#D8B4FE stroke:#7C3AED color:#111
  classDef api fill:#BFDBFE stroke:#2563EB color:#111

  class SparkSS spark
  class Janus graph
  class Airflow airflow
  class Trino trino
  class ClickHouse trino
  class Superset viz
  class API api
  class MinIO trino
