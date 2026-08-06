## # Energy Sector Data Analytics: Customer Time Series Multi-Factor Regression

## Overview
This repository showcases a production-grade SQL analytics engine designed for **Regional Utility – Northwest Division**. The primary script, `REGIONAL_NW_TOU_EXTRACT.sql`, performs advanced reconciliation between legacy CIS (Customer Information System) billing documents and high-frequency AMI (Advanced Metering Infrastructure) interval data.

This project demonstrates the ability to handle massive utility datasets within a **Cloud Data Warehouse (Amazon Redshift)** environment, ensuring billing accuracy for Time-of-Use (TOU) customers.

## Key Technical Features

### 1. Complex Legacy Billing System Integration (SAP/IS-U)
The script integrates several core SAP/IS-U billing and installation tables, implementing rigorous normalization and performance filters:

* **ERCH / ERCHC:** Extracts billing document headers and ensures only non-reversed, fully invoiced documents are processed.
* **EVER / EANL / EANLH:** Navigates the complex relationship between Contracts, Installations, and Rate Categories (Tariftyp) to ensure data temporal integrity.
* **ETTIFN:** Aggregates granular billing operands like `HISTKWH` and `MAXKW`.

### 2. Custom TOU & SDTR Calendar Logic
A significant portion of the logic is dedicated to a dynamic Time-of-Use calendar that distinguishes between:

* **Seasonal Windows:** Automated switching between Winter (Nov-Mar) and Summer (Apr-Oct) peak windows.
* **Peak/Off-Peak Definitions:** Precise hour-of-day filtering (e.g., 6 AM–9 AM and 6 PM–9 PM for Winter On-Peak).
* **SDTR Support:** Specialized logic for Seasonal Demand Time-of-Use Rider (SDTR) windows (3 PM–5 PM, June-Sept).
* **Holiday Awareness:** Joins with corporate calendars to correctly reclassify weekend and holiday peaks as off-peak.

### 3. AMI Data Scaling & Validation
* **Interval-to-Bill Reconciliation:** The engine joins 15-minute or 60-minute interval reads (`utl_local`) with UTC-to-EST timezone conversion.
* **Precision Scaling:** Includes a calculated `interval_scale_to_billed` factor to validate if AMI data accurately accounts for the total billed kWh, identifying potential data gaps or meter multiplier issues.

## Performance Optimization
* **Early Filter Push-Down:** Uses Common Table Expressions (CTEs) to apply `VKONT` (Account) and `MANDT` (Client) filters at the earliest possible stage, significantly reducing I/O.
* **Temporal Joins:** Implements range-based joins (`valid_from` / `valid_to`) to accurately map rate codes to specific billing periods.

## Use Case: Business Intelligence
This script is instrumental for:

* **Rate Impact Analysis:** Determining how customer bills would change under different TOU structures.
* **Revenue Protection:** Identifying discrepancies between meter reads and billed amounts.
* **Load Research:** Analyzing peak demand patterns for grid stability planning.

---

**Environment:** Amazon Redshift / PostgreSQL
