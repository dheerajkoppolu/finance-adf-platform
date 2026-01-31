# On-Prem SQL Server Infrastructure Setup

## Overview

This document describes the setup of an on-prem SQL Server environment used
as a transactional source system for a finance and payments data engineering
platform built using Azure Data Factory (ADF).

The goal of this setup is to simulate **real-world enterprise infrastructure**
rather than relying on cloud-managed or simplified local configurations.

---

## Why On-Prem SQL Server?

In many real-world organizations, core financial and payment systems still
operate on-premises due to:
- Regulatory constraints
- Legacy systems
- Latency requirements
- Data residency policies

This project intentionally uses an on-prem SQL Server to replicate challenges
such as:
- Network protocol configuration
- Client connectivity issues
- Infrastructure abstraction
- ADF Self-Hosted Integration Runtime readiness

---

## Environment Details

- Operating System: Windows 11
- Database Engine: SQL Server (Local installation)
- Client Tool: SQL Server Management Studio (SSMS)

---

## Logical Server Abstraction Using SQL Alias

Instead of directly connecting to the physical server using `localhost`,
a **logical server alias** was created to simulate an enterprise-style
server naming convention.

### Alias Configuration

- Alias Name: `PAYMENTS-ONPREM-DEV`
- Physical Server: `localhost`
- Network Protocol: `TCP/IP`
- Port: Dynamically assigned

### Why Use a SQL Alias?

Using a logical server alias provides:
- Decoupling of client connections from physical server names
- Environment portability (DEV / QA / PROD)
- Easier infrastructure changes without modifying client configurations
- A realistic enterprise connectivity pattern

All client connections in this project use the alias: PAYMENTS-ONPREM-DEV

---

## Issue Encountered During Setup

While attempting to connect to SQL Server using the alias, the following error
was encountered:
The remote computer refused the network connection
(TCP Provider, Error: 1225)

### Root Cause Analysis

- Local connections using `localhost` succeeded due to **Shared Memory**
- Alias-based connections use **TCP/IP**
- TCP/IP was **disabled by default** in the SQL Server network configuration
- As a result, SQL Server was not listening for TCP connections

This is a common issue in local SQL Server installations but becomes critical
when simulating on-prem enterprise environments.

---

## Resolution Steps

The issue was resolved through the following steps:

1. Enabled `TCP/IP` under:
SQL Server Network Configuration
→ Protocols for MSSQLSERVER

2. Restarted the SQL Server service to apply network changes

3. Reconnected using the logical alias: PAYMENTS-ONPREM-DEV

After these steps, the SQL Server successfully accepted TCP-based connections.

---

## Final Outcome

- SQL Server is accessible using a logical enterprise-style alias
- TCP/IP connectivity is enabled and validated
- The environment is ready for integration with:
- Azure Data Factory
- Self-Hosted Integration Runtime (SHIR)
- Production-like data pipelines

---

## Relevance to Azure Data Factory

Azure Data Factory requires TCP/IP connectivity when interacting with
on-premises data sources via SHIR.

This setup ensures:
- Realistic on-prem connectivity behavior
- Early detection of networking and protocol issues
- Production-aligned infrastructure design

---

## Key Learnings

- Local SQL Server defaults are not production-ready
- Alias-based abstraction is critical in enterprise environments
- Network protocol configuration is a common real-world failure point
- Infrastructure setup must be validated before pipeline development

---

## Next Steps

With the infrastructure layer complete, the next phase of the project focuses on:
- Creating the `FinancePaymentsOLTP` database
- Designing transactional payment schemas
- Building Azure Data Factory pipelines on top of this on-prem source
