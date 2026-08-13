# UPI Payment Transactions — Fraud & Spending Analysis (Python / Pandas)

## Overview
This project analyzes 20,000 UPI transactions across 2,000 users and 400 merchants (Jan–Dec 2024), covering data cleaning, table merging, and business-question-driven fraud analysis. Using Python and Pandas, the data was cleaned, connected across three source tables, and queried to answer real fraud-risk questions — each paired with a data-driven insight and recommendation, following the same insight-first approach as my SQL retail analysis project.

## Data Structure
Three related source files: `transactions.csv`, `users.csv`, and `merchants.csv`, merged into a single working table (`merged_data`) using `user_id` (transactions ↔ users) and `receiver_id`/`merchant_id` (transactions ↔ merchants). A fourth file, `fraud_labels.csv`, was checked and found to duplicate columns already present in `transactions.csv`, so `transactions.csv` was used as the single source of truth for fraud-related fields rather than merging both.

## Data Cleaning Notes
- `transaction_velocity` (393 missing) → filled with the **mode**, since it's whole-number/count data
- `amount_deviation_score` (367 missing) → filled with the **median**, since the mean was skewed upward by outliers (mean ≈ 0.65 vs median ≈ 0.44)
- `time_since_last_txn_min` (2,335 missing, plus ~8,900 logically invalid negative values) → **dropped entirely**. Two hypotheses (a calculation-direction bug, and a "first transaction has no prior gap" explanation) were tested against the actual data and neither matched the observed pattern, pointing to intentional synthetic noise in the dataset rather than a fixable error.
- Several duplicate columns appeared after merging (e.g. `user_loyalty_score_x`/`_y`, `user_kyc_status`/`kyc_status`) — each was checked for identical values before dropping the duplicate, rather than assuming.
- Merchant-related fields (e.g. `merchant_city`) are intentionally left blank for P2P transactions, since those genuinely have no merchant — left unfilled rather than guessed, to avoid distorting the fraud analysis.

---

## Key Findings

### 1. Merchant Category — Fraud Rate Needs a Sample-Size Check
Fuel shows the highest fraud rate (5.6%), followed by Insurance and Grocery (~4.7%). However, Fuel has only 179 transactions — the smallest category by far — making its fraud rate statistically unreliable, since a small sample can swing significantly from just a few random cases. Grocery has a similar fraud rate (4.7%) but is backed by 2,513 transactions, making it a far more statistically trustworthy signal.
**Recommendation:** Prioritize fraud monitoring in the Grocery category, since its elevated fraud rate is supported by a large, reliable sample. Continue observing Fuel, but avoid major business decisions based on its current rate until more transaction data is collected.

### 2. City Tier — Tier 3 Shows a Genuine, Trustworthy Gap
Tier 3 shows a higher fraud rate (4.08%) compared to Tier 2 (3.59%) and Tier 1 (3.88%). Unlike the Fuel finding above, all three tiers have thousands of transactions (4,824–8,015), so this difference is unlikely to be random noise.
**Recommendation:** Monitor Tier 3 transactions more closely, without shifting all fraud-prevention focus away from Tier 1 and Tier 2. A possible contributing factor is lower digital literacy in smaller cities, which could make users more vulnerable to fraud tactics — worth investigating further.

### 3. Transaction Timing — Night Transactions Are Meaningfully Riskier
Night transactions show a fraud rate of 5.27%, compared to 3.12% for day transactions — roughly 1.6x higher, with a solid sample size on both sides (6,492 night vs 13,508 day transactions).
**Recommendation:** Introduce additional verification steps (e.g. OTP or biometric confirmation) for late-night transactions, and enable real-time alert notifications so users can react quickly if a transaction wasn't authorized by them.

### 4. Device Familiarity — The Strongest Fraud Signal in This Analysis
Transactions from new/unrecognized devices show a fraud rate of 12.3%, compared to just 3.2% for familiar devices — roughly 3.8x higher, the largest gap found in this entire analysis. With 1,296 new-device transactions (a reasonably solid sample on its own), this difference is unlikely to be random noise.
**Recommendation:** Transactions from new devices should trigger mandatory additional verification (PIN re-entry, OTP confirmation) before approval, along with an immediate high-priority alert. New-device checks should be prioritized as a core fraud-prevention rule, given this is the strongest signal found.

### 5. Transaction Type — A Weak Signal (and a Corrected Hypothesis)
Fraud rates across transaction types are surprisingly close together — ranging from 3.50% (EMI) to 3.99% (P2M), a much smaller spread than device type or timing. This contradicts the initial hypothesis that P2P transfers would carry higher fraud risk than P2M (merchant) payments — the data shows P2P (3.73%) and P2M (3.99%) are nearly identical.
**Recommendation:** Since transaction type shows minimal variation in fraud risk, it should not be treated as a primary fraud-monitoring factor. Resources should instead be concentrated on the stronger signals identified above — device familiarity and transaction timing.

### 6. Custom Risk Rule — Testing a Hypothesis Against Real Outcomes
A custom rule was built, flagging transactions as risky when they occurred on a new/unrecognized device **and** during night-time hours — the two strongest individual signals found above. This rule flagged 412 transactions (2.1% of the dataset), of which 59 were confirmed fraud — a 14.3% fraud rate within the flagged group, nearly 4x higher than the overall 3.8% average. However, the rule only captured 59 of 763 total fraud cases (7.7% coverage), confirming most fraud occurs through other patterns not captured by these two conditions alone.
**Recommendation:** This rule should not be used as a standalone fraud detection system, since it only catches a small portion of overall fraud. It should instead be used as one component of a broader system — automatically flagging new-device-plus-night transactions for priority review, while additional rules are developed around other signals (transaction velocity, amount deviation, IP mismatches) to improve overall coverage.

---

## Visualizations
- `screenshots/device_fraud_chart.png` — Fraud rate: new device vs familiar device
- `screenshots/timing_fraud_chart.png` — Fraud rate: night vs day transactions

## Python Skills Demonstrated
- Data cleaning: handling missing values (mode/median imputation), identifying and dropping unreliable columns based on evidence, not assumption
- Merging multiple related tables (`pd.merge`) with different join-key names across tables
- Identifying and resolving duplicate columns created by merges
- Groupby aggregation (`.mean()`, `.size()`) to compare fraud rates across categories
- Hypothesis testing against real data, including reporting when a hypothesis was proven wrong
- Building and evaluating a custom rule-based classifier (precision vs coverage trade-off)
- Data visualization with Matplotlib

## Files
- `UPI_Fraud_Analysis.ipynb` — full analysis: data loading, cleaning, merging, all 6 business questions, and charts
- `README.md` — this summary
- `screenshots/` — chart images referenced above
