# Finance Analytics in Excel & Power Query

This repository demonstrates end-to-end **finance analytics workflows** using Excel (PivotTables, formulas, Power Query) on a General Ledger (GL) dataset. The focus is on **business logic, correctness, and auditability**, not just visuals.

---

## Dataset Overview

**GL Columns**:

* GLID
* TxnDate
* AccountNumber
* AccountName
* Debit
* Credit
* Dept
* CostCenter
* Description
* Currency

Derived columns are added during analysis (Net Amount, FX-converted amounts, audit flags).

---

## Task 1: Create Trial Balance PivotTables

### Business Logic

A trial balance summarizes **debits and credits by account** to validate accounting integrity. A balanced trial balance requires **both P&L and Balance Sheet accounts**.

### What Was Done

* Created **Net Amount = Debit − Credit**
* Converted multi-currency amounts to INR before aggregation
* Built PivotTable:

  * Rows: AccountName
  * Values: Debit_INR, Credit_INR, Trial Balance Amount

### Key Insight

* The dataset contains **only P&L accounts** (Revenue, COGS, Expenses)
* Balance Sheet accounts are missing
* Therefore, **trial balance does NOT net to zero by design**

This limitation is explicitly documented.

---

## Task 2: Build Department-Level P&L Dashboards

### Business Logic

P&L reporting requires grouping GL accounts into logical categories:

* Revenue
* COGS
* Expenses

This enables calculation of **Gross Profit** and **Gross Margin** by department.

### Steps

* Created an `Account_Mapping` table (AccountName → P&L Category)
* Used `XLOOKUP` to assign P&L Category to GL rows
* Built PivotTable:

  * Rows: Department
  * Columns: P&L Category
  * Values: Net Amount (INR)

### Derived Metrics

* **Gross Profit = Revenue − COGS**
* **Gross Margin % = Gross Profit / Revenue**

### Output

* Flat summary table for charting
* Column chart of Gross Profit by Department

---

## Task 3: Test Multi-Currency Conversions

### Business Logic

Financial reporting requires a **single reporting currency**. Transactions occur in multiple currencies across years, requiring **date-aware FX conversion**.

### Helper Table

Created `FX_Table` with:

* Currency
* RateDate
* FX_Rate (to INR)

Rates vary by year (2023–2025).

### Conversion Logic

* Used `XLOOKUP` with multiple conditions:

  * Match Currency
  * Pick the **latest FX rate on or before TxnDate**

### Formula Pattern

```excel
=XLOOKUP(
  1,
  (FX_Table[Currency]=[@Currency])*
  (FX_Table[RateDate]<=[@TxnDate]),
  FX_Table[FX_Rate],
  ,
  -1
)
```

### Result

* Debit_INR and Credit_INR columns added
* All reporting and pivots rebuilt on INR values

---

## Task 4: Audit Journal Entries with Power Query

### Business Logic

Audit analytics focuses on identifying **high-risk or unusual journal entries**, not proving correctness.

### Audit Flags Created

* **IsWeekend** → Transactions posted on Saturday/Sunday
* **IsPeriodEnd** → Transactions posted on company-specific close dates
* **IsLargeAmount** → Amount exceeds a data-driven threshold
* **IsRoundAmount** → Absolute amount is a clean round number

### Threshold Logic

* Calculated statistical thresholds (not hardcoded)
* Used **absolute values** for consistency
* Ensured numeric data types before flag evaluation

### Outcome

* Each journal entry tagged with audit indicators
* Enables filtering and focused audit review

---

## Key Learnings

* Trial balance validation requires **complete double-entry data**
* P&L reporting depends on **correct account mapping**, not raw GL names
* FX conversion must be **date-aware**, not static
* Audit analysis highlights **risk patterns**, not accounting balance

---

## Tools Used

* Excel PivotTables & Charts
* XLOOKUP (multi-condition)
* Structured Tables
* Power Query (data profiling & audit flags)

---
## Repository Structure

- General-Ledger-RawData.xlsx  
  Original general ledger extract used as input.

- General-Ledger-Analysis.xlsx  
  Excel-based financial analysis including:
  - Trial Balance
  - Department-level P&L
  - Multi-currency conversion
  - Audit journal analytics

## Disclaimer

This project uses a **P&L-only GL extract**. Trial balance imbalance is expected and documented as a data scope limitation.
