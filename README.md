# 🤖 MzansiConnect Customer Onboarding Automation
### Project: PM-04-PS02 | UiPath RPA Process

---

## 📋 Project Overview

**MzansiConnect Customer Onboarding** is a UiPath RPA automation that processes a batch of South African customer records from a CSV input file. The bot validates each customer's data, simulates submission to a CRM system, and generates an exceptions report for any records that fail validation.

| Field | Details |
|-------|---------|
| **Project Name** | MzansiConnect_CustomerOnboarding |
| **Project Code** | PM-04-PS02 |
| **Author** | Shanon Lutchman |
| **Version** | 1.0.0 |
| **Studio Version** | UiPath Studio 26.0.191.0 |
| **Target Framework** | Windows (.NET) |
| **Expression Language** | VB.NET |
| **Process Type** | Unattended |
| **Created** | 2026 |

---

## 🎯 Business Purpose

This automation addresses the manual effort of onboarding new customers into the MzansiConnect CRM system. The bot reads customer data from a structured CSV file, applies data quality validation rules (SA ID Number and Phone Number checks), logs all activities, and outputs a skipped-records exception report — enabling fast, accurate and auditable customer onboarding at scale.

---

## 🗂️ Project Structure

```
PM_04_PS02/
│
├── Main.xaml                          # Main automation workflow (entry point)
│
├── Data/
│   ├── Input/
│   │   └── customer_data_south_africa.csv   # Input: Customer records to process
│   ├── Exceptions/
│   │   └── Skipped_Records.csv             # Output: Exception/skipped records report
│   └── Screenshots/                        # Screenshot folder (reserved for future use)
│
├── project.json                       # UiPath project configuration
└── README.md                          # This file
```

---

## ⚙️ Workflow Logic — Main.xaml

The automation follows this end-to-end process:

```
START
  │
  ├─► Log: Process started (timestamp)
  │
  ├─► Read CSV: customer_data_south_africa.csv → dt_CustomerData
  │
  ├─► Log: Records loaded (count)
  │
  ├─► Build Exceptions DataTable (CustomerID | Reason | Timestamp)
  │
  └─► For Each Row in dt_CustomerData
        │
        ├─► Load row values:
        │     CustomerID, Name, Surname, SA_ID_Number, Phone_Number
        │
        ├─► [TryCatch]
        │     │
        │     ├─► VALIDATE: SA ID Number
        │     │     └── If missing → Log Warning → Add to Exceptions → Skip
        │     │
        │     ├─► VALIDATE: Phone Number (must be exactly 10 digits)
        │     │     └── If invalid → Log Warning → Add to Exceptions → Skip
        │     │
        │     ├─► Log: Valid record (masked ID & Phone)
        │     ├─► Simulate: CRM Submission
        │     └─► Increment: intSuccessCount
        │
        └─► [Catch: Exception]
              └── Log Warning → Add to Exceptions → Increment intSkipCount

  ├─► Log: Summary (Success | Skipped | Total)
  ├─► Write Exceptions CSV → Skipped_Records.csv
  ├─► Log: Exceptions report saved
  └─► Log: Process ended (timestamp)

END
```

---

## 📥 Input File

**Path:** `Data\Input\customer_data_south_africa.csv`

| Column | Type | Description |
|--------|------|-------------|
| `CustomerID` | String | Unique customer identifier (e.g. C001) |
| `Name` | String | Customer first name |
| `Surname` | String | Customer last name |
| `SA_ID_Number` | String | 13-digit South African ID number |
| `Phone_Number` | String | SA phone number (must resolve to 10 digits) |
| `Email` | String | Customer email address |
| `Province` | String | South African province |
| `City` | String | City of residence |

> ⚠️ **Note:** The workflow currently processes `CustomerID`, `Name`, `Surname`, `SA_ID_Number`, and `Phone_Number`. The `Email`, `Province`, and `City` fields are available in the input but reserved for future CRM integration.

---

## 📤 Output File

**Path:** `Data\Exceptions\Skipped_Records.csv`

| Column | Description |
|--------|-------------|
| `CustomerID` | ID of the skipped customer |
| `Reason` | Reason for skipping (e.g. "Missing SA_ID_Number", "Invalid Phone Number", "System Exception: ...") |
| `Timestamp` | Date and time the record was skipped (yyyy-MM-dd HH:mm:ss) |

---

## ✅ Validation Rules

| Rule | Condition | Action on Failure |
|------|-----------|-------------------|
| SA ID Number present | `SA_ID_Number` must not be null or whitespace | Log warning → Add to exceptions → Skip row |
| Phone Number valid | Phone (digits only) must be exactly **10 digits** | Log warning → Add to exceptions → Skip row |

> 📌 Phone numbers are cleaned of all non-numeric characters before length validation (e.g. `083-456-7890` → `0834567890`).

---

## 🔒 Data Privacy

- SA ID Numbers are **masked** in logs — only the last 4 digits are shown (e.g. `*********3087`).
- Phone Numbers are **masked** in logs — only the last 4 digits are shown (e.g. `******6677`).
- Sensitive fields are excluded from robot logs via `Private:*` and `*password*` exclusion rules.

---

## 📊 Variables

| Variable | Type | Purpose |
|----------|------|---------|
| `dt_CustomerData` | DataTable | Holds all rows from the input CSV |
| `dt_Exceptions` | DataTable | Accumulates skipped/failed records |
| `strCustomerID` | String | Current row's customer ID |
| `strName` | String | Current row's first name |
| `strSurname` | String | Current row's surname |
| `strSA_ID` | String | Current row's SA ID number |
| `strPhone` | String | Current row's phone (digits only) |
| `intSuccessCount` | Int32 | Count of successfully processed records |
| `intSkipCount` | Int32 | Count of skipped/failed records |
| `strExceptionPath` | String | Path to the exceptions CSV output file |
| `strScreenshotFolder` | String | Path for screenshots (reserved) |

---

## 📦 Dependencies

| Package | Version |
|---------|---------|
| `UiPath.System.Activities` | 26.2.4 |
| `UiPath.Excel.Activities` | 3.5.0-preview |

---

## 🚀 How to Run

### Prerequisites
1. UiPath Studio **26.0+** installed
2. The input file `Data\Input\customer_data_south_africa.csv` must exist and be properly formatted
3. The `Data\Exceptions\` folder must exist (or will be created on first run)

### Steps
1. Open **UiPath Studio**
2. Open the project folder `PM_04_PS02`
3. Open `Main.xaml`
4. Click **▶ Run** (or press `F5`)
5. Monitor the **Output** panel for live logs
6. After completion, review `Data\Exceptions\Skipped_Records.csv` for skipped records

---

## 📝 Expected Output (Sample Run)

```
[Info]  MzansiConnect Customer Onboarding process started at 2026-06-17 14:00:00
[Info]  CSV loaded successfully. Total records loaded: 20
[Info]  Exceptions table created successfully.
[Info]  Processing row: C001
[Info]  Valid record. Processing Row C001 | ID: *********087 | Phone: ******4567
[Info]  Row C001 submitted to CRM successfully
...
[Warn]  Row C012 skipped: Phone number invalid (length: 0)
[Warn]  Row C016 skipped: SA_ID_Number is missing
...
[Info]  Process completed. Success: 18 | Skipped: 2 | Total: 20
[Info]  Exceptions report saved to Data\Exceptions\Skipped_Records.csv
[Info]  MzansiConnect Customer Onboarding process ended at 2026-06-17 14:00:05
```

---

## 🔧 Error Handling

| Scenario | Handling |
|----------|---------|
| Missing SA ID | Business validation → skip record |
| Invalid phone number | Business validation → skip record |
| Unexpected system error | `TryCatch` → log warning → add to exceptions → continue processing |

The automation uses a **row-level TryCatch** to ensure that a single bad record does not halt the entire batch — all other records continue to be processed.

---

## 🔮 Future Enhancements

- [ ] Integrate with live CRM API for actual submission
- [ ] Add SA ID Number format/checksum validation (Luhn algorithm)
- [ ] Email notification on process completion with summary stats
- [ ] Add screenshot capture on exception for audit trail
- [ ] Validate `Email` field format using regex
- [ ] Support for Excel input in addition to CSV

---

## 👤 Author

| Field | Value |
|-------|-------|
| **Name** | Shanon Lutchman |
| **Email** | shanonlutchman@gmail.com |
| **Organisation** | Alexander Forbes Group Services |
| **Project** | PM-04-PS02 |

---

*Generated for UiPath RPA Automation — MzansiConnect_CustomerOnboarding © 2026*
