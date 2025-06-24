# data_Analylitics_pipeline
A fully-orchestrated Spark + Airflow pipeline that ingests open municipal data every night, builds a star-schema warehouse, and surfaces ‘What-if’ mobility &amp; weather insights through SQL and lightweight ML.
```mermaid
flowchart TD
  subgraph Kafka Topics
    profiles((user_profiles))
    posts((user_posts))
    follow((user_follows))
  end

  SparkSS["Spark 3.5 Structured Streaming<br/>⇢ micro-batch 10 s"] 
  SparkSS -->|upserts Iceberg V2<br/>s3a://sociallake/bronze| Iceberg[(Apache Iceberg<br/>on MinIO/S3)]
  SparkSS -->|sends edges| Neo4j[(Neo4j 5 • Aura or Docker)]
  SparkSS -->|push metrics| ClickHouse[(ClickHouse for low-latency API)]

  Airflow["Airflow 2.8 DAGs<br/>daily compaction + model retrain"] --> SparkSS

  Trino["Trino 448"] --> Iceberg
  Superset --> Trino
  RealtimeAPI["FastAPI - /graph/reco"] --> Neo4j
  RealtimeAPI --> ClickHouse
