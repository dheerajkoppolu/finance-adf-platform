# Data Sources and Characteristics

This document describes the real-world data sources used in the Finance & Payments Data Platform and the challenges associated with each source.

---

## 1. On-Premise Payment Transactions Database

Source Type:
- On-Premise SQL Server (simulated locally)

Description:
- This database represents the core payment processing system used by a finance company.
- It stores all customer payments, transaction statuses, and account mappings.

Key Tables:
- payments
- accounts

Data Characteristics:
- High volume of inserts throughout the day
- Frequent updates to payment status (PENDING → SUCCESS / FAILED)
- Duplicate transactions due to retries
- Late-arriving transactions

Real-World Challenges:
- Incremental data ingestion using watermark columns
- Handling updates and duplicates safely
- Ensuring idempotent pipeline execution
- Network connectivity failures (on-prem to cloud)

---

## 2. External Financial APIs

Source Type:
- REST APIs

Examples:
- Foreign Exchange (FX) Rates API
- Payment Gateway Status API

Data Characteristics:
- JSON responses with nested structures
- Paginated responses
- Rate limits (HTTP 429 errors)
- Occasional downtime or missing data

Real-World Challenges:
- API throttling and retry logic
- Handling partial API responses
- Schema drift in API payloads
- Ensuring data completeness

---

## 3. Settlement and Reconciliation Files

Source Type:
- CSV files dropped by external partners

Description:
- Daily settlement files used for payment reconciliation.

Data Characteristics:
- Files may arrive late or be missing
- Schema changes without prior notice
- Duplicate or missing payment records
- Incorrect or inconsistent file naming

Real-World Challenges:
- File arrival detection and validation
- Handling late and reprocessed files
- Schema validation before ingestion
- Reconciliation against transaction data

---

## Why These Sources Are Realistic

These data sources reflect real finance and payments systems where data reliability, traceability, reconciliation, and error handling are critical. Designing pipelines around these challenges ensures the platform behaves like a real production system rather than a demo.
