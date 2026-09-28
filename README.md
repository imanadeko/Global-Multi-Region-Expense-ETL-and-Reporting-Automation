# Global Multi-Region Expense ETL & Reporting Automation


An automated end-to-end Alteryx ETL pipeline designed to ingest, cleanse, reshape, and consolidate multi-regional financial expense data and organizational hierarchy structures. The workflow automates the normalization of wide-format monthly financial records across North America and South America, sanitizes messy organizational attributes across dynamic multi-sheet workbooks, and orchestrates sequential multi-tab Excel reporting.



## Project Overview

Financial reporting across global operating units frequently faces fragmentation due to disparate Excel spreadsheets, inconsistent cell formatting, merged title headers, wide-format date arrangements, and dirty organizational data. 

This project delivers a production-grade Alteryx workflow (`.yxmd`) and package (`.yxzp`) that:
- **Ingests heterogeneous regional files** without relying on manual cell selections.
- **Normalizes 36 months of horizontal financial records** (2014–2016) into structured time-series observations.
- **Dynamically reads multi-tab manager records** across global territories.
- **Cleans messy numeric/string fields**.
- **Sequences dual-sheet Excel outputs** (`Summary by Country` and `Detail`) within a single target file using Alteryx's `Block Until Done` tool to prevent write collisions.

---

## Business Problem & Requirements

### Key Objectives
1. **Multi-File Ingestion**: Read North America (`NA-1.xlsx`), South America (`SA-1.xlsx`), and Management hierarchy (`Managers-1.xlsx`) by selecting entire sheets rather than fixed cell ranges.
2. **Cleansing & Filtering**:
   - Bypass title blocks, metadata stamps, and blank lines (table data begins at row 9).
   - Eliminate null rows, spacer columns, and hidden cell artifacts (`"Hidden Value - Don't Show!!!"`).
3. **Data Reshaping (Wide-to-Long)**:
   - Pivot horizontal monthly expense dates (`2014-01-01` through `2016-12-01`) into vertical records.
4. **Dynamic Multi-Sheet Ingestion**:
   - Extract the sheet list from `Managers-1.xlsx` and read all available regions (`North America`, `South America`, `Europe`) dynamically.
   - Cleanse whitespace and remove non-numeric characters from team size metrics.
5. **Relational Join & Aggregation**:
   - Join expense time-series with management data using `Country` as the primary key, resulting in exactly **216 detailed records** across **5 core variables**.
   - Aggregate total expenses by country and manager.
6. **Orchestrated Excel Delivery**:
   - Sequentially publish both the high-level summary and granular details into separate sheets of a unified `Output.xlsx` workbook without file access conflicts.
7. **Step-Count Efficiency**:
   - Execute the entire end-to-end transformation within a strict **28-tool limit**.

---

## Pipeline Architecture

<img width="1920" height="1080" alt="Screenshot 2026-09-24 221318" src="https://github.com/user-attachments/assets/3cb8c324-619a-4735-96fd-f79957d7c1e6" />


---

## Data Engineering Methodology

1. Ingestion & Header Handling

Raw sheets (NA-1.xlsx, SA-1.xlsx) contain metadata across rows 1–8. Set ImportLine = 9 to map row 9 directly as field headers (Country and 36 monthly date columns), skipping the blank lines and header notes.

2. Cleansing & Filtering

Use the Select tool to drop artifact columns (F2, Date>>>, F40), the Data Cleansing tool to strip leading and trailing whitespace, and the Filter tool on !IsNull([2014-01-01]) to discard empty spacer rows.

3. Wide-to-Long Normalization

The Transpose tool keeps Country as the key field and collapses the 36 date columns into normalized Name (date) and Value (expense) rows. A Union tool then stacks the NA and SA streams into a single 216-row dataset.

4. Manager Ingestion & String Sanitization

DbFileInput and Dynamic Input read all worksheets from Managers-1.xlsx. The Data Cleansing tool removes irregular whitespace from country names and strips text from Team Size strings (e.g., "135 EE", "team of 15") to convert them to clean integers.

5. Relational Join & Reconciliation

A Join tool matches records on Country, casting Date (Date), Expense (Double), and Team Size (Int32). All 216 expense records match cleanly (0 left unjoined), leaving 4 right unjoined European managers who lack expense data.

6. Aggregation & Multi-Tab Writing

A Block Until Done tool prevents Excel write-lock errors on Output.xlsx. The first stream summarizes expenses by Country, Manager, and Team Size into the Summary by Country tab. Once that lock releases, the second stream writes the 216 granular records into the Detail tab.

---



---



```text
======================================================================
PIPELINE EXECUTION & RECONCILIATION SUMMARY
======================================================================
[+] Input Records Read:
    - North America Expenses  : 108 records (3 countries x 36 months)
    - South America Expenses  : 108 records (3 countries x 36 months)
    - Dynamic Manager Records : 10 records across 3 regions
[+] Cleaned Joined Records    : 216 records (100% match)
[+] Dropped Left Records      : 0 (No orphan expense entries)
[+] Unmatched Right Records   : 4 (European countries without expense data)
[+] Summary Tab Generated     : 6 countries (Brazil, Canada, Chile, Colombia, Mexico, US)
[+] Detail Tab Generated      : 216 rows x 5 attributes
======================================================================
```
