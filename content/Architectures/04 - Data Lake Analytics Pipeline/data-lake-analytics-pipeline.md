# Data Lake + Analytics Pipeline

**Diagram file:** `data-lake-analytics-pipeline.drawio`
**Audience:** Technical (SAA-C03 exam prep)

## Flow Description

1. **Data Sources** (web/mobile apps, IoT devices, application logs) generate events continuously and send them to Kinesis Data Streams as producers.
2. **Kinesis Data Streams** ingests the real-time data and durably buffers it in shards for up to 24 hours (365 days with extended retention), fanning out to two independent consumer lanes.
3. **Top lane — real-time path:** Kinesis Data Analytics (Managed Service for Apache Flink / SQL) reads directly from the stream, runs windowed SQL/Flink transformations, and emits results to real-time apps and dashboards with sub-second to low-second latency.
4. **Bottom lane — batch/lake path:** Kinesis Data Firehose reads from the same stream, buffers by size/time, and delivers batched objects to an S3 raw data bucket without needing custom consumer code.
5. S3 event notifications trigger a **Lambda** function that converts the raw (JSON/CSV) objects into **Parquet** format, applying columnar compression and partitioning (e.g., by year/month/day) as it writes back to S3.
6. **AWS Glue** crawlers catalog the Parquet data (populating the Glue Data Catalog with schema/partition metadata), and Glue ETL jobs handle any further transformation/cleaning.
7. The catalog is shared by two downstream query engines: **Athena** for ad-hoc, serverless SQL directly against S3, and **Redshift** (via Redshift Spectrum or COPY) for structured, high-concurrency data warehouse workloads.
8. Both **Athena** and **Redshift** feed **QuickSight**, which builds the business dashboards end users consume.

## Service Table

| Service | Role in this architecture | Category |
|---|---|---|
| Kinesis Data Streams | Real-time, durable ingestion buffer; decouples producers from consumers; supports multiple parallel consumers | Analytics |
| Kinesis Data Analytics | Real-time SQL/Flink processing directly on the stream for low-latency insights | Analytics |
| Kinesis Data Firehose | Fully managed delivery of streaming data to S3 with buffering, batching, and format conversion — no custom consumer code | Analytics |
| S3 | Durable, cheap object storage for the raw and Parquet-converted data lake | Storage |
| Lambda | Serverless transform: converts raw JSON/CSV to partitioned Parquet | Compute |
| AWS Glue | Serverless ETL + Data Catalog (crawlers infer schema, catalog is the shared metastore) | Analytics |
| Athena | Serverless, pay-per-query SQL engine for ad-hoc analysis directly on S3 | Analytics |
| Redshift | Managed data warehouse for structured, high-concurrency BI workloads | Analytics |
| QuickSight | Serverless BI dashboards consuming both Athena and Redshift | Analytics |

## Key Design Decisions

**Kinesis Data Streams vs. Firehose vs. Data Analytics**
- **Data Streams** is the right choice when you need custom consumer logic, multiple independent consumers reading the same data, replay capability (up to 365 days), or sub-second processing latency. You manage shards/scaling and write your own consumer application (KCL, Lambda, or Flink).
- **Firehose** is the right choice when the destination is a *known* sink (S3, Redshift, OpenSearch, Splunk, HTTP endpoint) and you don't need custom processing beyond optional Lambda transforms. It's fully managed — no shard management, no consumer code — but it's a near-real-time (buffered) delivery mechanism, not a millisecond-latency stream.
- **Data Analytics** (Managed Service for Apache Flink) sits on top of a stream (or Firehose) when you need continuous SQL or Flink-based aggregation/windowing/anomaly detection *in-flight*, before data lands anywhere.
- **Exam pattern:** if the question says "process data in real time with SQL" → Data Analytics. If it says "load streaming data into S3/Redshift with minimal ops" → Firehose. If it says "multiple consumers need the same raw stream" or "need replay" → Data Streams.

**Why Parquet + partitioning reduces Athena cost**
- Athena charges per byte scanned. Parquet is columnar and compressed, so a query that only needs a few columns skips reading the rest of the row — this alone commonly cuts scanned bytes by 80-90% versus row-based JSON/CSV.
- Partitioning (e.g., by date) lets Athena use partition pruning to skip entire prefixes/objects that don't match the query's WHERE clause, instead of scanning the whole bucket.
- Combined, Parquet + partitioning is the standard exam answer for "how do you reduce Athena query cost/time" — along with keeping file sizes reasonably large (avoiding the small-files problem) via compaction.

**Glue Data Catalog's role**
- It's the central, Hive-compatible metastore: table definitions, schemas, and partition metadata that Athena, Redshift Spectrum, EMR, and Glue ETL jobs all read from.
- Glue Crawlers populate/update it automatically by scanning S3 data and inferring schema — removing the need to manually run `CREATE TABLE` / `ALTER TABLE ADD PARTITION` statements.
- Because it's shared across services, cataloging data once in Glue makes it queryable from Athena *and* Redshift Spectrum *and* EMR without duplicating metadata management.

**Athena vs. Redshift use cases**
- **Athena**: serverless, pay-per-query, ideal for ad-hoc/exploratory analysis, infrequent queries, or when you don't want to provision/manage a cluster. Query performance depends on data layout (partitioning, file format, file size).
- **Redshift**: provisioned (or serverless) data warehouse, ideal for repeated, complex joins, high-concurrency BI dashboards, and workloads needing consistent low-latency performance with tunable compute (RA3 nodes, concurrency scaling). Redshift Spectrum lets it query S3 data directly using the same Glue Data Catalog, blending warehouse and lake data in one query.
- **Exam pattern:** "infrequent ad-hoc queries, no ops overhead" → Athena. "Frequent, complex BI queries at scale, need consistent performance / concurrency" → Redshift.

**Common SAA-C03 exam traps**
- Confusing Firehose (near-real-time, managed delivery) with Data Streams (real-time, self-managed consumers, replay). Firehose *cannot* be read by multiple custom consumers the way a Kinesis stream can.
- Assuming Kinesis Data Analytics requires Firehose — it can read directly from a Kinesis Data Stream *or* a Firehose delivery stream.
- Forgetting that Firehose has a minimum buffering interval (as low as 0 seconds for some destinations, but typically object accumulates for 60s+ /5MB by default) — it is not millisecond-latency.
- Believing Athena requires a running cluster or that it charges a flat rate — it's serverless and billed per TB scanned, which is exactly why storage format/partitioning matters.
- Thinking Glue is only an ETL tool — the Data Catalog is a distinct, equally important component that many other AWS analytics services depend on.
- Assuming Redshift is required to query S3 — Redshift Spectrum and Athena can both query S3 directly; Redshift's advantage is for data already loaded into its own managed storage or for very high query concurrency.
- Overlooking Kinesis Data Streams' shard-based scaling — throughput (1MB/s or 1000 records/s write per shard) is a common quantitative exam question.

## Notes

- PNG export was skipped — no local draw.io CLI/desktop app was available in this environment to render the diagram. Open `data-lake-analytics-pipeline.drawio` directly in the draw.io app or desktop client to view/export it.
- Icon stencil names were verified directly against the draw.io `Sidebar-AWS4.js` source (kinesis_data_streams, kinesis_data_firehose, kinesis_data_analytics, s3, lambda, glue, athena, redshift, quicksight, sensor, client, traditional_server) rather than guessed.
