# Dataset Description — Cleaned Customer Features

## 1. Source and provenance

The analytic table `cleaned_customer_features.csv` (delivered as `cleaned_customer_features.csv.gz` in the repository to keep transfer size small) is derived from the public **Olist Brazilian E-Commerce Public Dataset** (Kaggle), which contains nine relational CSV tables: orders, customers, order items, payments, order reviews, products, sellers, geolocation and product-category translation. An upstream cleaning pipeline joined these tables at customer level and engineered delivery, satisfaction, discount and RFM-style features. No proprietary or personal data were collected by the authors.

| Raw table | Role in feature construction |
| :--- | :--- |
| `olist_orders_dataset.csv` | Purchase and delivery timestamps; delivery duration and delay. |
| `olist_order_reviews_dataset.csv` | Review score and textual-comment indicator. |
| `olist_customers_dataset.csv` | `customer_unique_id` used for customer-level aggregation. |
| `olist_order_items_dataset.csv` | Item count, price and freight value. |
| `olist_order_payments_dataset.csv` | Payment type, instalments, voucher share. |
| `olist_products_dataset.csv`, `product_category_name_translation.csv` | Product category labels (English). |
| `olist_sellers_dataset.csv`, `olist_geolocation_dataset.csv` | Geography and logistics context (not used as predictors). |

## 2. Sample size and structure

* Full cleaned table: **93,350 rows x 33 columns**; each row is one unique customer (`customer_unique_id`).
* Analytic cohort: **62,835 rows** satisfy `in_cohort = 1` (a complete 180-day observation window). The notebook restricts modelling to this cohort to avoid right-censoring bias.
* 30,515 customers fall outside the cohort and are retained in the delivered CSV for transparency but excluded from modelling.

## 3. Target definition

| Label | Meaning | Cohort distribution |
| :--- | :--- | :--- |
| `churn_180d` | 1 = customer made **no repeat purchase within 180 days** of the first order; 0 = at least one repeat purchase in that window. | 61,767 churned (98.30%); 1,068 retained (1.70%) |
| `churn_90d` | Same definition over a 90-day window (sensitivity label). | 98.5% churned in full table |
| `repeat_180d` | Exact complement of `churn_180d`. | 1,351 repeat customers in full table |

The extreme imbalance is a genuine property of the marketplace (most customers transact once), not a processing error. It is handled with class-weighted estimators, stratified splitting and imbalance-appropriate metrics; it is **not** corrected by synthetic resampling.

## 4. Missing-value profile (full 93,350-row table)

| Column | Missing count | Missing % | Treatment in the notebook |
| :--- | ---: | ---: | :--- |
| `days_to_next_order` | 91,551 | 98.08% | Excluded (outcome information / leakage). |
| `first_review_score` | 615 | 0.66% | Median imputation inside the pipeline inputs. |
| `first_is_bad_review` | 615 | 0.66% | Filled with 0 (no negative review recorded). |
| `desc_avg_review_score` | 603 | 0.65% | Excluded (descriptive field). |
| `first_total_payment` | 1 | <0.01% | Median imputation before log transform. |
| `first_max_installments` | 1 | <0.01% | Median imputation. |

All other 27 columns are complete.

## 5. Variable groups

1. **First-order features (predictors):** `first_delivery_duration_days[_w]`, `first_delivery_delay_days[_w]`, `first_is_delayed`, `first_review_score`, `first_has_review`, `first_is_bad_review`, `first_voucher_payment_share`, `first_has_voucher`, `first_freight_ratio`, `first_total_payment`, `first_item_count`, `first_max_installments`, `first_main_category`, `customer_state`.
2. **Cohort and outcome fields:** `in_cohort`, `has_full_window`, `churn_180d`, `churn_90d`, `repeat_180d`, `repeat_90d`, `days_to_next_order`, `ordered_before_first_delivery`.
3. **Descriptive lifetime fields (excluded from modelling):** `desc_order_count_total`, `desc_total_spent`, `desc_avg_review_score`, `desc_is_repeat_ever`.
4. **Identifiers / dates (excluded):** `customer_unique_id`, `first_order_id`, `first_purchase_date`, `first_delivered_date`, `customer_city`.

See `codebook/CODEBOOK.md` for the full variable-by-variable specification.

## 6. Known limitations

* Churn is inferred from purchase inactivity rather than explicitly declared, so "churned" customers may simply have a longer natural repurchase cycle.
* The 180-day window is applied uniformly; customers acquired near the dataset cutoff are excluded via `in_cohort` to mitigate censoring.
* Category and state levels with very small counts are collapsed to `other` before encoding to limit overfitting.
* Results describe one marketplace and one time period; external validity to other platforms is not established.
