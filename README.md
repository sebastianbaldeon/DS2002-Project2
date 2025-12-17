# DS-2002 Project 2: Northwind Data Lakehouse

**Sebastian Baldeon** | University of Virginia | DS-2002 Fall 2025

---

## Project Summary

Built a dimensional Data Lakehouse for Northwind Traders order management using PySpark Structured Streaming. Integrates data from MongoDB Atlas (NoSQL), MySQL (relational), and JSON files (streaming) through Bronze → Silver → Gold medallion architecture.

**Final Output:** 58 order records enriched with 5 dimension tables, ready for business analytics.

---

## Quick Facts

| Aspect | Implementation |
|--------|----------------|
| **Streaming** | 3 JSON batches processed incrementally (19, 19, 20 records) |
| **Data Sources** | MongoDB Atlas + MySQL + JSON Files (3 types ✓) |
| **Architecture** | Bronze (raw) → Silver (enriched) → Gold (analytics) |
| **Storage** | Parquet format (all layers) |
| **Dimensions** | 5 tables: date, customers, products, employees, shippers |
| **Business Queries** | 5 analytics queries demonstrating value |

---

## Key Design Decisions & Process

### 1. Why Medallion Architecture?

**Challenge:** Needed to integrate both static reference data and streaming transaction data.

**Solution:** 
- **Bronze** = Raw streaming ingestion with metadata tracking
- **Silver** = Enrichment by joining streams with static dimensions  
- **Gold** = Final denormalized fact table for fast queries

**Trade-off:** Initially tried Delta Lake for ACID guarantees, but it's not in local PySpark. Switched to Parquet - still fast for analytics, just no ACID.

---

### 2. Multi-Source Integration Strategy

**Original situation:** All data in MySQL Northwind database.

**What I did:**
- Exported customers & products → MongoDB Atlas (satisfies NoSQL requirement)
- Kept employees & shippers in MySQL (satisfies relational requirement)  
- Created 3 JSON batches from order_details (satisfies file/streaming requirement)

**Why?** This mirrors real-world architecture where product catalogs often use NoSQL while HR data stays in relational databases.

---

### 3. Streaming Implementation (AutoLoader)

**Challenge:** Requirements say "use Spark AutoLoader" but that's Databricks-specific.

**My approach:**
```python
.readStream.format("json").option("maxFilesPerTrigger", 1)
```

**Why this works:** `maxFilesPerTrigger: 1` processes one file per trigger = 3 intervals for 3 files. This is the local PySpark equivalent of AutoLoader. TA confirmed this approach was acceptable.

**Verified:** Console showed "Stream has processed 3 batches" ✓

---

### 4. Data Quality Issue & Fix

**Problem:** First streaming attempt showed all NULL values in Bronze layer.

**Root cause:** JSON files were in array format `[{...}, {...}]` but Spark streaming needs newline-delimited format (one object per line).

**Solution:** Added preprocessing cell to convert format. After conversion, all 58 records loaded correctly with real data.

**Lesson learned:** Always validate data format before streaming!

---

### 5. Silver Layer: Stream-to-Static Joins

**How it works:**
- Bronze streaming data has foreign keys (customer_id, product_id, etc.)
- Silver layer joins with static dimension tables (customers, products, employees, shippers)
- Result: Streaming facts enriched with descriptive attributes in real-time

**Why left joins:** Preserves all fact records even if dimension lookup fails (data quality monitoring).

---

## Requirements Checklist

✅ **Date dimension** - 365 dates with year/month/quarter  
✅ **3+ dimensions** - 4 dimensions (customers, products, employees, shippers)  
✅ **1+ fact table** - fact_orders with order management metrics  
✅ **Relational source** - MySQL (employees, shippers)  
✅ **NoSQL source** - MongoDB Atlas (customers, products)  
✅ **File source** - JSON streaming files  
✅ **Static + streaming** - Dimensions (batch) + Facts (stream)  
✅ **AutoLoader pattern** - maxFilesPerTrigger: 1 for 3 intervals  
✅ **Bronze/Silver/Gold** - Full medallion architecture  
✅ **Stream-static joins** - Silver layer enrichment  
✅ **Business value** - 5 analytics queries  

---

## Business Results

**5 Analytics Queries Delivered:**

1. **Sales by Country** → USA: $68,137 total
2. **Employee Performance** → Nancy Freehafer: top performer ($22,255)
3. **Product Analysis** → Top 10 products identified  
4. **Monthly Trends** → March 2006 peak ($32,609)
5. **Customer Intelligence** → Company BB: top customer ($15,432)

**Value:** Enables data-driven decisions on employee recognition, inventory planning, and customer retention.

---

## Project Files

```
DS2002-Project2/
├── DS2002_Project2_Northwind_Lakehouse.ipynb    # Main notebook (all code)
├── streaming_data/                               # 3 JSON batch files
├── config.py                                     # Credentials (not in Git)
└── README.md                                     # This file
```

---

## How to Run

**1. Create `config.py`:**
```python
MYSQL_USER = "your_username"
MYSQL_PASSWORD = "your_password"
MONGODB_USER = "your_mongo_user"
MONGODB_PASSWORD = "your_mongo_password"
```

**2. Update paths in notebook:**
- MongoDB cluster info (if different)
- MySQL JDBC JAR path

**3. Run notebook:**
- Kernel → Restart & Run All
- Expected runtime: 2-3 minutes
- All cells should execute successfully

---

## Key Learnings

**Technical:**
- Structured Streaming requires newline-delimited JSON (not arrays)
- Local PySpark can simulate cloud architectures effectively
- Parquet works great for read-heavy analytics

**Process:**
- Always validate format compatibility before streaming
- Multi-source integration needs careful planning
- Security matters: never commit credentials to Git

**Architecture:**
- Medallion pattern naturally separates ingestion/transformation/aggregation
- Stream-to-static joins enable real-time enrichment
- Star schema simplifies business queries

---

## Challenges Solved

| Problem | Solution |
|---------|----------|
| All NULL values in Bronze | Converted JSON to newline-delimited format |
| Delta Lake unavailable | Switched to Parquet (works for analytics) |
| AutoLoader not in local Spark | Used maxFilesPerTrigger equivalent |
| Credential security | Used config.py excluded from Git |

---

**End Result:** Production-ready data lakehouse demonstrating multi-source integration, streaming processing, and dimensional modeling for business intelligence.

---

**Author:** Sebastian Baldeon  
**GitHub:** https://github.com/sebastianbaldeon/DS2002-Project2
