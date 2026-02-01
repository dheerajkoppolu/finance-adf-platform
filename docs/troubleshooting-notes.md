# Troubleshooting & Learnings – On-Prem to Azure Data Platform

This document captures real-world issues encountered while building an end-to-end
on-prem SQL Server to Azure Data Lake ingestion pipeline using Azure Data Factory.
Each issue includes symptoms, root cause analysis, resolution, and learnings.

---

## Issue 1: Unable to connect SSMS to SQL Server using instance name

### Context
- Environment: Local Windows 11 machine
- Tool: SQL Server Management Studio (SSMS)
- Goal: Connect to on-prem SQL Server instance

### Error / Symptom
Error 26 - Error Locating Server/Instance Specified


### Root Cause
- SQL Server Browser service was disabled
- TCP/IP protocol was not enabled for the SQL Server instance

### Resolution Steps
1. Open **SQL Server Configuration Manager**
2. Enable **TCP/IP** under:
SQL Server Network Configuration → Protocols for MSSQLSERVER

3. Restart SQL Server service
4. Reconnect using:
localhost


### Key Learning
> SQL Server connectivity issues are often protocol-related, not credential-related.

---

## Issue 2: SQL Server Alias configuration UI not accepting input

### Context
- Tool: SQL Server Configuration Manager
- Goal: Create a realistic server alias for on-prem SQL Server

### Error / Symptom
- Alias dialog opened but text fields were not editable

### Root Cause
- Known UI/permission glitch when Configuration Manager is not launched with elevated privileges

### Resolution
- Restarted system
- Relaunched SQL Server Configuration Manager as Administrator
- Alias creation succeeded after restart

### Key Learning
> Always run infra-level tools with admin privileges on Windows.

---

## Issue 3: ADF unable to connect to on-prem SQL Server (Error 26 / 1225)

### Context
- Tool: Azure Data Factory (V2)
- Source: On-prem SQL Server
- Integration: Self-Hosted Integration Runtime (SHIR)

### Error / Symptom
The remote computer refused the network connection
Error 1225


### Root Cause
- TCP/IP enabled on SQL Server
- But SQL Server service restart was pending
- Firewall rules not yet applied consistently

### Resolution Steps
1. Enabled TCP/IP in SQL Server Configuration Manager
2. Restarted SQL Server service
3. Restarted Self-Hosted Integration Runtime service
4. Retested connection from ADF

### Key Learning
> Configuration changes on SQL Server almost always require a service restart.

---

## Issue 4: Confusion around Self-Hosted Integration Runtime registration

### Context
- SHIR was previously installed for another project
- New ADF instance created for this project

### Problem
- Could not find "Register" option during SHIR setup

### Root Cause
- Existing SHIR was already registered with another Data Factory
- UI behavior differs for "New SHIR" vs "Linked SHIR"

### Resolution
- Reused existing SHIR
- Created **Linked Self-Hosted Integration Runtime** in new ADF
- Verified node status as **Connected**

### Key Learning
> SHIR can be shared across Data Factories using Linked IR – reinstall is not always required.

---

## Issue 5: ADF Linked Service for SQL Server asking for username & password

### Context
- Authentication type: Windows Authentication
- SQL Server login: DOMAIN\username

### Problem
- ADF still required username/password
- Connection failing with authentication errors

### Root Cause
- Windows authentication through ADF requires Kerberos delegation
- Not feasible in local / learning environment

### Resolution
- Switched to **SQL Authentication**
- Created SQL login with least-privilege access
- Updated ADF linked service to use SQL Auth
- Connection succeeded

### Key Learning
> SQL Authentication is simpler and more reliable for on-prem ingestion via SHIR unless Kerberos is properly configured.

---

## Issue 6: “Bad JSON escape sequence” error in ADF test connection

### Error
Bad JSON escape sequence (\d)


### Root Cause
- Backslash in Windows username (DOMAIN\user) was not escaped properly in JSON

### Resolution
- Avoided Windows authentication
- Used SQL Authentication instead

### Key Learning
> Many ADF errors are serialization-related, not connectivity-related.

---

## Final Outcome

- Successfully connected on-prem SQL Server to Azure Data Factory
- Data ingestion pipeline ready to load data into Bronze layer
- All infra issues documented for future reference and interviews

---

## Overall Learnings

- Most production issues are configuration and networking related
- Reading error messages carefully saves hours
- Always validate each layer independently (SQL → SHIR → ADF)
- Documentation is as important as implementation

### Issue: ADF SQL Linked Service fails with Login failed (Error 4060)

**Root Cause**
Login existed at SQL Server level but database user lacked permissions.

**Resolution**
1. Create database user for login
2. Grant db_datareader / required roles
3. Validate SQL login via SSMS before testing in ADF

**Key Learning**
Authentication ≠ Authorization in SQL Server.

