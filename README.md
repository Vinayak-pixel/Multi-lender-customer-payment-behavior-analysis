
# 🏦 Cross-NBFC Payment Behaviour Analysis — Targeted Collection Strategy

> **Identifying borrowers who honour repayments at other NBFCs while defaulting on our NBFC loans, enabling precision-targeted collections to reduce GNPA.**

---

## 📌 Problem Statement

In the NBFC lending ecosystem, a borrower may hold loans from multiple financial institutions simultaneously. A critical risk pattern exists where a customer **consistently pays EMIs to competing NBFCs on time, while deliberately delaying or skipping payments to our company **. Left undetected, this behaviour inflates the company's Gross Non-Performing Assets (GNPA) and misrepresents the true credit risk of the borrower.

This project was designed to **surface exactly these customers** from a large, unstructured credit bureau dataset and direct targeted collection efforts at them — protecting the company's loan book and improving recovery rates.

---

## 🎯 Objective

- Parse and structure raw Equifax credit bureau scrub data (≈1.3 crore records)
- Identify customers with active loans at **both our NBFC (ON US) and external NBFCs (OFF US)**
- Detect customers showing **worse repayment behaviour with MCSL vs. other lenders**
- Build a **DPD cross-matrix** to segment and prioritise collection targets
- Drive targeted collection interventions to **normalise delinquent accounts and reduce GNPA**

---

## 🗂️ Dataset

| Attribute | Detail |
|---|---|
| **Source** | Equifax Credit Bureau (Scrub Data) |
| **Scope** | Entire parent company portfolio |
| **Size** | ~1.3 crore (13 million) records |
| **Format** | Unstructured, pipe-delimited (`\|`) flat file |
| **Key Columns** | Reference No (PK), Customer Name, Loan Amount, Sector, DPD |

---

## 🔧 Methodology

### Step 1 — Data Parsing & Structuring
- Ingested the raw pipe-delimited (`|`) flat file using **Python (Pandas)**
- Converted the unstructured format into a clean, analysis-ready structured DataFrame
- Retained only the columns required for analysis:
  `Reference No`, `Customer Name`, `Customer Details`, `Loan Amount`, `Sector`, `DPD`

### Step 2 — Data Cleaning
- Removed records with missing values in critical fields: `DPD`, `Reference No`
- Enforced correct data types — numerical columns cast to numeric, categorical columns standardised as strings
- Handled encoding issues arising from large flat-file ingestion

### Step 3 — Sector Classification
Applied a conditional rule to classify every loan record:

```python
df['Loan_Type'] = df['Sector'].apply(
    lambda x: 'ON US' if x in mcsl_identifiers else 'OFF US'
)
```

- **ON US** → Loan is with our company
- **OFF US** → Loan is with any other external NBFC

### Step 4 — Multi-Lender Customer Identification
- Grouped records by `Reference No` (primary key)
- Filtered customers who had entries in **both** `ON US` and `OFF US` categories — these are the multi-lender borrowers

Two sub-segments were created:

| Segment | Description |
|---|---|
| **Segment A** | 1 ON US loan + 1 OFF US loan |
| **Segment B** | 1 ON US loan + 2 or more OFF US loans |

For **Segment B**, the OFF US record with the **highest DPD (worst repayment)** was retained as the representative OFF US behaviour, ensuring a conservative and fair comparison.

### Step 5 — DPD Bucket Cross-Matrix
DPD values were bucketed into standard collection categories:

`0/STD` | `1–30` | `30–60` | `60–90` | `90+`

A **pivot matrix** was built with:
- **Rows** → OFF US DPD bucket (their repayment at other NBFCs)
- **Columns** → ON US DPD bucket (their repayment at our NBFC)

```
              ┌─────────────────── ON US DPD ───────────────────┐
              │  0/STD   1-30   30-60   60-90    90+            │
OFF US  0/STD │    3       2      5       9     2542  ◀ HIGH RISK│
        1-30  │    0       0      0       1      203            │
        30-60 │    1       1      0       0      109            │
        60-90 │    0       1      0       0      114            │
        90+   │    5       3      1       1     2975            │
              └───────────────────────────────────────────────────┘
```

> 🔴 **Red-shaded cells** (low OFF US DPD, high ON US DPD) = customers paying others on time but defaulting with our NBFC → **Primary collection targets**

---

## 📊 Key Insight — The Matrix Logic

| Cell Interpretation | Action |
|---|---|
| **OFF US low DPD + ON US high DPD** 🔴 | Customer *can* pay but *chooses not to* pay our NBFC → Immediate targeted follow-up |
| **OFF US high DPD + ON US high DPD** 🟠 | Genuinely stressed borrower → Restructuring / NPA provisioning |
| **OFF US low DPD + ON US low DPD** 🟢 | Healthy borrower → Monitor only |

---

## ✅ Outcome

- Successfully isolated **high-priority delinquent accounts** where wilful default was strongly indicated
- Targeted collection campaigns were launched on the red-zone customer segment
- Achieved **normalisation of multiple high-DPD accounts** — moving them from NPA back to standard
- Contributed to a **measurable reduction in the company's GNPA**
- The matrix framework was adopted as a **repeatable monthly monitoring tool** within the BI team

---

## 🛠️ Tech Stack

| Tool | Usage |
|---|---|
| **Python** | End-to-end pipeline |
| **Pandas** | Data parsing, cleaning, transformation, groupby logic |
| **NumPy** | Numerical operations |
| **Jupyter Notebook** | Development & analysis environment |
| **Excel** | Stakeholder reporting & matrix visualisation |

---





## ⚠️ Data Privacy Note

This project was executed on proprietary credit bureau scrub data under strict data governance protocols. No raw customer data, PII or confidential financial records are included in this repository. All data references are for illustrative and documentation purposes only.

---


=======


