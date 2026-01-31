# Data Model Design

This document describes the data model used in the Finance & Payments Data Platform. The model is designed to reflect real-world payment processing systems while supporting analytics and reporting needs.

---

## 1. OLTP Data Model (On-Premise)

### Payments Table

Purpose:
- Stores individual payment transactions initiated by customers.

Columns:
- payment_id (primary key)
- account_id (foreign key)
- amount
- currency
- status (PENDING, SUCCESS, FAILED)
- payment_method
- created_at
- updated_at

Design Considerations:
- Status updates are common and must be tracked.
- updated_at is used for incremental ingestion.
- Duplicate payment attempts may occur due to retries.

---

### Accounts Table

Purpose:
- Stores customer account information linked to payments.

Columns:
- account_id (primary key)
- customer_id
- country
- account_type
- created_at

Design Considerations:
- Accounts are relatively stable compared to payments.
- Used for joins during enrichment and analytics.

---

## 2. Analytical Data Model (Gold Layer)

### Fact_Payments

Purpose:
- Central fact table for payment analytics.

Columns:
- payment_id
- account_id
- amount
- currency
- amount_converted
- status
- payment_method
- payment_date

---

### Dim_Account

Purpose:
- Dimension table for account-related attributes.

Columns:
- account_id
- customer_id
- country
- account_type

---

## Why This Model Works for Finance

- Separates transactional and analytical concerns.
- Supports incremental ingestion and late-arriving data.
- Enables reconciliation and auditability.
- Scales as transaction volume increases.
