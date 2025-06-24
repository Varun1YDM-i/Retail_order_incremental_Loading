
#  Real-Time Retail Orders Incremental Data Pipeline 

##  Project Overview

Imagine a fast-paced retail client managing hundreds of orders each day. Instead of repeatedly processing the same data, the business needs a solution that captures only new and updated records—a classic case of incremental data loading.

This project delivers a real-time data pipeline using Azure Data Factory (ADF) and Azure SQL Database, designed to ingest JSON-formatted order data from a REST API and perform intelligent upsert operations (insert and update), ensuring the final order table remains accurate and up to date.

---

## 🧠 What This Project Solves

> ✅ Incrementally Load only **new and updated orders** from an API  
> ✅ Load raw data to a **staging table** (TempOrders)  
> ✅ Use a **stored procedure with SQL MERGE** to upsert data  
> ✅ Keep the final OrderTable **clean, current, and client-ready**

---

## 🏗️ Architecture

### 🔹 Ingestion: Copy Data from API to Staging  
- Source: REST API (JSON) hosted on GitHub  
- Sink: `TempOrders` staging table in Azure SQL  
- **Pre-Copy Script**: `TRUNCATE TABLE TempOrders` before every run

### 🔹 Transformation: Upsert to Final Table  
- `SP_OnlyGetNewRecords` SQL stored procedure  
- Uses **MERGE** to:
  - INSERT new records (based on OrderID)
  - UPDATE existing records (based on newer UpdateDate)

---

## 🛠️ Azure Services Used
- Azure Data Factory (v2)
- Azure SQL Database

---

## ⚙️ How It Works

1. **ADF Pipeline** pulls full snapshot from REST API → loads to `TempOrders`
2. Stored procedure compares `TempOrders` with `OrderTable`
3. Only **new** or **updated** records are merged into `OrderTable`

---

## 📈 Example Scenario

- Initial load: Order IDs 1001, 1002, 1003 → inserted
- Next Load : Order IDs 1002 , 1004
- API updates Order 1002 and adds Order 1004
- Re-run pipeline:
  - ✅ Order 1002 → **updated**
  - ✅ Order 1004 → **inserted**

---

## ▶️ How to Run It

1. Set up Azure SQL Database & create:
   - `TempOrders` (staging table)
   - `OrderTable` (final table with PK)
   - Stored Procedure: `SP_OnlyGetNewRecords`

2. Build ADF pipeline:
   - **Copy Data Activity** from REST API → `TempOrders`
   - **Stored Procedure Activity** → call `SP_OnlyGetNewRecords`

3. Debug/Test:
   - First run → all data inserted
   - Modify source JSON (add/update orders) → re-run → upserted correctly

---

## 🧪 Verification SQL

```sql
SELECT * FROM dbo.TempOrders;
SELECT * FROM dbo.OrderTable;
