# Codebook — Independent Variable Handbook

Analytic unit: one unique customer (`customer_unique_id`). Target: `churn_180d`.
Conventions: `bin` = binary 0/1; `int` = integer; `float` = continuous; `str` = categorical string; `datetime` = date-time string.
"P" = used as a predictor in the notebook; "X" = excluded for leakage/identification reasons; "T" = target/label.

## 1. Identifiers, time and cohort fields

| Variable | Type | Business meaning | Encoding / coding rule | Preprocessing | Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `customer_unique_id` | str | Persistent anonymised customer key | 32-char hex; no ordering | Dropped | X |
| `first_order_id` | str | Identifier of the customer's first order | 32-char hex | Dropped | X |
| `first_purchase_date` | datetime | Timestamp of first purchase | ISO `YYYY-MM-DD HH:MM:SS` | Dropped | X |
| `first_delivered_date` | datetime | Delivery timestamp of first order | ISO datetime | Dropped | X |
| `in_cohort` | bin | Customer has a complete 180-day observation window | 1 = complete, 0 = censored | Filter `== 1` | X |
| `has_full_window` | bin | Customer has a complete generic observation window | 1/0 | Filtered via `in_cohort` | X |

## 2. First-order delivery features

| Variable | Type | Business meaning | Encoding / coding rule | Preprocessing | Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `first_delivery_duration_days` | float | Actual days from purchase to delivery | Continuous, days | Excluded; winsorised variant used | X |
| `first_delivery_duration_days_w` | float | Winsorised delivery duration (1st–99th pct) | Continuous, days | StandardScaler in pipeline | P |
| `first_delivery_delay_days` | float | Actual minus estimated delivery days (>0 = late) | Continuous, days | Excluded; winsorised variant used | X |
| `first_delivery_delay_days_w` | float | Winsorised delay (1st–99th pct) | Continuous, days | StandardScaler in pipeline | P |
| `first_is_delayed` | bin | First order arrived after the promised date | 1 = late, 0 = on time | Passthrough (numeric) | P |
| `ordered_before_first_delivery` | bin | Customer reordered before first delivery occurred | 1/0 | Excluded: post-outcome information | X |

## 3. Satisfaction and review features

| Variable | Type | Business meaning | Encoding / coding rule | Preprocessing | Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `first_review_score` | float | Star rating of the first order | Ordinal 1–5 | Median imputation, StandardScaler | P |
| `first_has_review` | bin | Customer submitted any review | 1 = yes, 0 = no | Passthrough (numeric) | P |
| `first_is_bad_review` | float | First review was 1 or 2 stars | 1 = negative, 0 = otherwise | Missing filled with 0; StandardScaler | P |
| `desc_avg_review_score` | float | Mean rating across all orders | Continuous 1–5 | Excluded: lifetime/descriptive | X |

## 4. Price, discount and product features

| Variable | Type | Business meaning | Encoding / coding rule | Preprocessing | Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `first_total_payment` | float | Amount paid on the first order (BRL) | Continuous, BRL R$ | Median impute, `log1p`, StandardScaler | P (as `log_total_payment`) |
| `first_voucher_payment_share` | float | Share of payment covered by voucher | Ratio 0–1 | StandardScaler in pipeline | P |
| `first_has_voucher` | bin | A voucher was used on the first order | 1 = yes, 0 = no | Passthrough (numeric) | P |
| `first_freight_ratio` | float | Freight / (price + freight) | Ratio 0–1 | StandardScaler in pipeline | P |
| `first_item_count` | int | Number of items in the first order | Count >= 1 | StandardScaler in pipeline | P |
| `first_max_installments` | float | Maximum instalments chosen on first order | Count 0–24 | Median impute, StandardScaler | P |
| `first_main_category` | str | Dominant English product category of first order | Nominal; top 10 levels retained | Rare levels -> `other`; OneHotEncoder(handle_unknown='ignore') | P |
| `customer_state` | str | Brazilian state of primary residence | Nominal 2-letter code | Top 10 levels retained; rest -> `other`; OneHotEncoder | P |
| `customer_city` | str | City of primary residence | Nominal, high cardinality | Dropped (too many levels) | X |

## 5. Behavioural and outcome fields (excluded from predictors)

| Variable | Type | Business meaning | Encoding / coding rule | Role |
| :--- | :--- | :--- | :--- | :--- |
| `churn_180d` | bin | No repeat purchase within 180 days | 1 = churn, 0 = retained | T |
| `churn_90d` | bin | No repeat purchase within 90 days | 1 = churn, 0 = retained | X (sensitivity label) |
| `repeat_180d` | bin | At least one repeat purchase within 180 days | Complement of `churn_180d` | X |
| `repeat_90d` | bin | At least one repeat within 90 days | Complement of `churn_90d` | X |
| `days_to_next_order` | float | Days from first order to next order | Continuous; 98.1% missing | X (leakage) |
| `desc_order_count_total` | int | Lifetime number of orders | Count >= 1 | X (leakage) |
| `desc_total_spent` | float | Lifetime expenditure (BRL) | Continuous | X (leakage) |
| `desc_is_repeat_ever` | bin | Ever repeated at any time | 1/0 | X (leakage) |

## 6. Derived variables created in the notebook

| Derived name | Formula | Purpose |
| :--- | :--- | :--- |
| `log_total_payment` | `log1p(first_total_payment)` | Compress right-skewed monetary values. |
| `first_main_category` (recoded) | Top-10 frequencies else `other` | Limit one-hot dimensionality. |
| `customer_state` (recoded) | Top-10 frequencies else `other` | Limit one-hot dimensionality. |
| Model matrix `X` | 12 numeric + 2 nominal encoded fields | Input to both pipelines. |

## 7. Preprocessing contract

All imputation is performed before the split only for median constants derived from descriptive statistics of the full table; scaling and one-hot encoding are fitted **inside** each scikit-learn `Pipeline` on the training fold only, preventing leakage into held-out data. Numerical columns are standardised with `StandardScaler`; categorical columns use `OneHotEncoder(handle_unknown='ignore')`. Both classifiers use `class_weight='balanced'` (or `balanced_subsample`) to account for the 98.3% churn base rate.
