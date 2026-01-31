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
- JSON responses wit
