# Bronze Layer Design – Finance Payments Data Platform

## 1. Purpose of the Bronze Layer

The Bronze layer represents the **raw ingestion zone** for all source systems.
Its primary purpose is to land data from on-prem systems into Azure Data Lake
**without applying business logic**, ensuring full traceability, auditability,
and reprocess capability.

In this project, the Bronze layer is designed to ingest data from an on-prem
SQL Server–based payment processing system using Azure Data Factory (ADF)
with a Self-Hosted Integration Runtime (SHIR).

---

## 2. Design Principles

The Bronze layer follows these non-negotiable principles:

- **No transformations**
- **No deduplication**
- **No deletes or updates**
- **Append-only ingestion**
- **Re-runnable pipelines**
- **Full audit trace to source**

Any data correction, deduplication, or enrichment happens in **Silver or Gold layers**, never in Bronze.

---

## 3. Source Systems

| Source | Type | Location |
|------|----|---------|
| FinancePaymentsOLTP | SQL Server | On-Prem (Windows 11) |

### Source Tables
- `Accounts`
- `Payments`
- `PaymentEvents`

---

## 4. Storage Layout (Azure Data Lake Gen2)

Root container:
Root container:
/bronze


Folder structure:
bronze/
├── accounts/
│ └── load_type=full/
│ └── load_date=YYYY-MM-DD/
│ └── accounts_YYYYMMDDHHMMSS.parquet
│
├── payments/
│ ├── load_type=full/
│ │ └── load_date=YYYY-MM-DD/
│ │ └── payments_YYYYMMDDHHMMSS.parquet
│ │
│ └── load_type=incremental/
│ └── load_date=YYYY-MM-DD/
│ └── payments_YYYYMMDDHHMMSS.parquet
│
└── payment_events/
└── load_type=incremental/
└── load_date=YYYY-MM-DD/
└── payment_events_YYYYMMDDHHMMSS.parquet


---

## 5. Partitioning Strategy

### Partition Key: `load_date`

**Why load_date and not event time?**

- Represents ingestion time, not business time
- Simplifies re-runs and backfills
- Allows clear visibility into daily ingestion
- Avoids late-arriving data complexity in Bronze

Event-time partitioning is deferred to **Silver layer**.

---

## 6. File Format

**Parquet**

### Rationale:
- Columnar format
- Cost-efficient storage
- Faster downstream analytics
- Industry standard for lakehouse architectures

---

## 7. Ingestion Strategy by Table

### 7.1 Accounts
- Load type: **Full**
- Frequency: Daily
- Reason:
  - Reference data
  - Low volume
  - Rare changes

---

### 7.2 Payments
- Load types:
  - **Full (initial)**
  - **Incremental (ongoing)**
- Incremental logic:
  - Watermark based on `initiated_at`
- Updates allowed in source
- Bronze retains all versions (no overwrite)

---

### 7.3 PaymentEvents
- Load type: **Incremental only**
- Nature: Append-only
- Incremental logic:
  - Watermark based on `event_time`
- Late-arriving events allowed

---

## 8. Technical Metadata Columns (Added in Bronze)

Each Bronze dataset includes the following technical columns:

| Column | Purpose |
|-----|--------|
| `_ingestion_time` | Time when data landed in Bronze |
| `_source_system` | Source identifier (`FinancePaymentsOLTP`) |
| `_pipeline_run_id` | ADF pipeline execution ID |
| `_load_type` | full / incremental |

These columns support:
- Auditing
- Debugging
- Replay
- Incident investigation

---

## 9. Error Handling Strategy

- Pipelines **fail fast**
- Partial loads are allowed (append-only)
- Failed runs do NOT delete existing Bronze data
- Errors are captured via:
  - ADF activity logs
  - Pipeline run metadata

---

## 10. Reprocessing Strategy

Reprocessing is achieved by:
- Re-running pipelines for specific `load_date`
- Writing to a new timestamped file
- Downstream layers decide which data is authoritative

No destructive operations are performed in Bronze.

---

## 11. Why Bronze Exists (Interview Summary)

The Bronze layer exists to ensure:

- Financial auditability
- Reconciliation accuracy
- Late-arriving data handling
- Safe reprocessing
- Regulatory compliance readiness

---

## 12. Key Takeaways

- Bronze is a **data landing zone**, not a reporting layer
- Correctness > cleanliness at this stage
- Finance systems require immutable raw data
- Design favors traceability over convenience
