# Bank Transaction Analytics

**Power BI dashboard** analyzing transaction health, fraud exposure, and fee revenue for a fictional UK neobank, built for Risk, Finance, Product, and Compliance teams.

### 2. Problem to Be Solved

Build a decision-making dashboard to:

- Monitor transaction health (Total Transaction, Trnsaction Value, Completion, Failed rates)
- Locate fraud exposure by segment, merchant category, customer, and time
- Detect fee billing errors (under-collection and overcharging)
- Check whether declared risk ratings match actual fraud outcomes

**Key questions**

- How have transaction volume and value trended month-on-month?
- Which customer segments drive the highest volumes and values?
- What share of transactions is flagged as fraud, and how does it vary by merchant category?
- Does the declared `risk_flag` align with actual fraud events?
- Are fees applied consistently, or is there leakage and overcharging?
- Which channels, devices, and transaction types fail most?
- Which individual customers generate outsized risk?

## Table of Contents

- [I. Introduction](#i-introduction)
- [II. Design Thinking](#ii-design-thinking)
- [III. Visualization](#iii-visualization)
- [IV. Insight and Recommendation](#iv-insight-and-recommendation)
- [V. Recommendations](#v-recommendations)
- [Tech Stack](#tech-stack)

## I. Introduction

### 1. Dataset

~1,500 digital transactions from 20 customers of a UK neobank, January to May 2026 (H1 2026 brief). Star schema: one fact table, three dimension tables

```mermaid
erDiagram
    dim_customer ||--o{ fact_transactions : customer_id
    dim_transaction_type ||--o{ fact_transactions : transaction_type_id
    dim_merchant_category ||--o{ fact_transactions : merchant_category_id
    dim_date ||--o{ fact_transactions : transaction_date

    dim_customer {
        int customer_id PK
        string customer_name
        string age_band
        string region
        string customer_segment
        boolean kyc_verified
        date account_open_date
    }
    dim_transaction_type {
        int transaction_type_id PK
        string type_name
        string channel
        boolean is_domestic
        decimal typical_fee_gbp
    }
    dim_merchant_category {
        int merchant_category_id PK
        string category_name
        string sector
        string risk_flag
    }
    fact_transactions {
        int transaction_id PK
        int customer_id FK
        int transaction_type_id FK
        int merchant_category_id FK
        date transaction_date
        datetime transaction_datetime
        decimal amount_gbp
        decimal fee_charged_gbp
        string transaction_status
        boolean is_flagged_fraud
        decimal fx_rate_used
        string device_type
        string failed_reason
    }
```

#### fact_transactions

| Column | Type | Description |
|---|---|---|
| transaction_id | INTEGER | Unique transaction identifier |
| customer_id, transaction_type_id, merchant_category_id | INTEGER | Foreign keys to the dimension tables |
| transaction_date, transaction_datetime | DATE, DATETIME | When the transaction occurred |
| amount_gbp | DECIMAL | Transaction value in GBP |
| fee_charged_gbp | DECIMAL | Fee actually charged (may differ from the typical fee if waived or in error) |
| transaction_status | STRING | Completed, Declined, Pending, Reversed |
| is_flagged_fraud | BOOLEAN | TRUE if the fraud engine flagged the transaction |
| fx_rate_used | DECIMAL | FX rate for international transactions; NULL for domestic |
| device_type | STRING | iOS, Android, Web, N/A |
| failed_reason | STRING | Reason for Declined or Reversed; NULL otherwise |

#### Dimension tables

| Table | Rows | Key columns |
|---|---|---|
| dim_customer | 20 | customer_segment (Starter, Standard, Premium, Business), region, age_band, kyc_verified, account_open_date |
| dim_transaction_type | 15 | type_name, channel (Mobile App, Web Browser, ATM Network, Automated), is_domestic, typical_fee_gbp |
| dim_merchant_category | 18 | category_name, sector, risk_flag (Low, Medium, High) |

### 2. Problem to Be Solved

Build a decision-making dashboard to:

- Monitor transaction health (Total Transaction, Trnsaction Value, Completion, Failed rates)
- Locate fraud exposure by segment, merchant category, customer, and time
- Detect fee billing errors (under-collection and overcharging)
- Check whether declared risk ratings match actual fraud outcomes

**Key questions**

- How have transaction volume and value trended month-on-month?
- Which customer segments drive the highest volumes and values?
- What share of transactions is flagged as fraud, and how does it vary by merchant category?
- Does the declared `risk_flag` align with actual fraud events?
- Are fees applied consistently, or is there leakage and overcharging?
- Which channels, devices, and transaction types fail most?
- Which individual customers generate outsized risk?

## II. Design Thinking

### Step 1: Empathize

Four stakeholder groups: **Risk** (where is fraud concentrated), **Finance** (is fee revenue correct), **Product** (where do transactions fail), **Compliance** (is KYC working).

### Step 2: Define POV

North star metrics

| Area | Metrics |
|---|---|
| Transaction health | Total Transaction, Transaction Value, Completion Rate, Failed Rate (Declined + Reversed) |
| Fraud and risk | Fraud Flag Rate |
| Revenue health | Fee Revenue, Revenue Leakage |

### Step 3: Ideate

Five report pages: Executive Overview (bank pulse), Risk (where fraud sits), Finance (fee revenue and leakage), Product (channels, devices, failures), Customer Investigation (drill-through to one customer).

### Step 4: Prototype and Review

Chart types matched to each question: line charts for trends, bar charts for mutually exclusive categories, matrix heatmaps for two-dimensional combinations (day × time), tables for customer-level investigation. Rates are shown with transaction counts (n) because the sample is small.

## III. Visualization

### Executive Overview

*What does the bank look like?*

- How have transaction count and value trended month-on-month?
- What is the status mix (Completed, Declined, Pending, Reversed)?
- Which channels carry the most transaction value?
- Which UK regions generate the most transaction activity, and where are fraud events concentrated?
- Which segments drive the highest values?
- Do unverified (non-KYC) customers show disproportionate rates of fraud-flagged transactions?

<img width="1423" height="732" alt="image" src="https://github.com/user-attachments/assets/736fcfab-5504-42ef-acb3-b2e93647bd84" />

**Insights**

- Transaction value increased 48.4% versus April, driven by higher activity among Standard clients.
- Monthly transaction count is stable between 279 and 322, and the status mix is consistent across months.
- Premium drives the highest transaction value (63% of the total).
- The Automated channel leads with 42.83% of total transaction value, followed by Mobile App at 35.5%.
- London accounts for 60% of all fraud flags.
- Only 4 customers are currently unverified. Results are not statistically significant, and a 0% fraud rate among non-KYC customers does not mean low risk. Compliance must still remediate all 4 non-KYC accounts.

### Risk

*Where does fraud concentrate?*

- What is the Fraud Rate Trend ?
- Fraud Rate across customer_segment ?
- What percentage of all transactions were flagged as fraudulent, and how does this vary by merchant category?
- Fraud Rate through device_type ?
- Is there a day-of-week or time-of-day pattern in high-risk transactions?
- Do fraud-flagged transactions correlate with non-KYC-verified customers?

<img width="1250" height="707" alt="image" src="https://github.com/user-attachments/assets/da7ea52a-c01f-43ea-b072-b0cfbf0f0d6a" />

**Insights**

- Fraud rate is steady at about 20% (300 flagged transactions), with a dip in April (about 13%) and a rebound in May (about 23%).
- Premium has the highest fraud rate (40%) and also 63% of value, so it carries the largest exposure.
- Nine merchant categories sit at about 29-31% fraud rate. Gambling is no worse than Fuel or Electronics.
- Declared `risk_flag` labels are misaligned with actual fraud outcomes: High 19.9%, Medium 22.3%, Low 19.0%.
- Half of flagged transactions come from Unknown device.
- There is no strong day or time pattern. Only Monday morning (38%) and Friday evening (36%) stand out, and small cell sizes make even these tentative.

### Fee Revenue and Leakage 

*Are we collecting the right fees?*

- What is the fee revenue trend ?
- Which transaction types contribute Fee revenue the most ?
- Are there transaction type or merchant categories where actual fees charged deviate from the typical fee - suggesting waivers or errors?
- Which customer segments generate the highest fee revenue per transaction?
- Which transaction type contribute the most ?(Domestic or  International ?)

<img width="1243" height="742" alt="image" src="https://github.com/user-attachments/assets/b50647ce-2539-4d36-8ea1-f8a54103d05e" />

**Insights**

- Fee errors run in both directions and cancel out. About £587 is under-collected (78% of the £750 expected fees) while a similar amount is overcharged, so net revenue (£750.17 against £750.00 expected) hides the errors.
- Transfer - International has the largest fee variance (-£400) and contributes disproportionately to fee leakage. ATM Withdrawals (-£88) and Crypto Purchases (-£37) are also under-collected.
- Business customers generate the highest fee revenue per transaction (£0.90 - £1.20).
### Product and Operation

*Where do transactions fail?*

- Which channel fails most?
- Which transaction types fail most?
- Do international transactions fail more than domestic?
- How does device usage differ by segment and Which devices see the most failures?
- Are channel failures Declines or Reversals?

<img width="1247" height="675" alt="image" src="https://github.com/user-attachments/assets/fd985429-01ea-4941-baec-7de3680bec37" />

**Insights**

- The failure rate is about 60% across the board, consistent across every channel (58% to 63%) and failure type. For a real bank this level would be unacceptable.
- Card Refund, Loan Repayment and Standing Order fail most (about 75%). Direct Debit, Salary Credit, Savings Pot Transfer and Transfer - Outbound sit lowest at 50%.
- Failures split about 60% Declined and 40% Reversed in every channel, so no channel leans toward one failure type.
- Business customers use mainly Unknown device (75%) and Web (25%), while Starter splits evenly between iOS and Android.

### Customer Investigation

Drill-through page: review one customer's transactions, flags, fees, and status in every dimension.

<img width="1255" height="692" alt="image" src="https://github.com/user-attachments/assets/11e5fe45-22e5-4f73-b48a-cddf6f78ac7b" />

**Insights**

- Rajan Mehta (Premium) generates the most risk. The nine largest flagged transactions all belong to this customer, are all Reversed, and repeat the same amounts (£17,273 six times, £16,818 three times).
- These nine transactions total about £154K, roughly 13% of the £1.19M fraud exposure, from a single customer.

## IV. Insight and Recommendation

## Insights and Recommendations

| Insight | Recommendation |
|---|---|
| London accounts for 60% of fraud flags (225 flagged transactions). The nine largest flagged transactions belong to one Premium customer, Rajan Mehta, with repeated amounts. | Investigate the London fraud cluster, starting with Rajan Mehta, the highest-exposure customer. |
| Fee errors run in both directions and cancel out. About £587 is under-collected (78% of the £750 expected fees) while a similar amount is overcharged, so net revenue hides the errors. | Recover the £587 in under-collected fees and refund overcharges. Start with International Transfers (-£400). |
| Transfer - International has the largest fee variance (-£400). ATM Withdrawals and Crypto Purchases are also consistently under-collected. | Review fee configuration for Transfer - International, ATM Withdrawals and Crypto Purchases. |
| Failure rate is about 60% across every channel and failure type, highest for Card Refund, Loan Repayment and Standing Order (about 75%). | Investigate the root cause of the platform-wide failure rate, starting with those three types. Analyse `failed_reason` (insufficient funds, fraud rules or system errors) and review the shared transaction processing flow. |
| Declared `risk_flag` labels do not match actual fraud outcomes: High 19.9%, Medium 22.3%, Low 19.0%. Nine merchant categories sit at about 29-31% fraud rate. | Re-tier merchant risk ratings using observed fraud rates. |
| Only 4 customers are unverified, and their 0% fraud rate is not statistically significant. | Remediate all 4 non-KYC accounts, regardless of the 0% observed fraud rate. |
| Business customers generate the highest fee revenue per transaction (£0.90 - £1.20). | Prioritise Business customers for premium products. |

## Tech Stack

- Power BI (data model, report pages, drill-through)
- DAX (all measures)
- Star schema with an Extended Calendar table

## Repository Structure

```
.
├── README.md
├── zephyr_bank_dashboard.pbix
├── data/
│   ├── fact_transactions.csv
│   ├── dim_customer.csv
│   ├── dim_transaction_type.csv
│   └── dim_merchant_category.csv
└── images/
    ├── overview.png
    ├── risk.png
    ├── finance.png
    ├── product.png
    └── customer.png
```
