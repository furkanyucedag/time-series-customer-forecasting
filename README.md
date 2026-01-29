# time-series-customer-forecasting

# Time Series Customer Forecasting (Holt-Winters)

This project forecasts **future daily customer counts** by transforming customer-level transaction data (email + order date) into **segment-level time series** and applying **Triple Exponential Smoothing (Holt-Winters)** with yearly seasonality.

## Problem
Given transactional order data containing `customer_email` and `OrderDate`, we want to:
1) Define customer segments based on purchase recurrence (business-driven churn window),
2) Build daily time series per segment (unique customer counts per day),
3) Forecast future customer dynamics (focus: **NewBuyer** daily counts).

## Approach

### 1) Label Engineering (Customer-level)
For each customer, compute the day difference between consecutive orders and assign labels:

- **NewBuyer**: first observed purchase for a customer
- **drop**: customer purchased again within **<= 365 days**
- **ChurnNewBuyer**: customer purchased again after **> 365 days**

> Note: The 365-day window is a business-defined churn/recency boundary and can be adjusted.

### 2) Segment-level Daily Time Series
Aggregate transactions into a daily time series:

`(label, day) -> unique_customer_count`

This produces a table similar to:

| label | order_day | customer_count |
|------|-----------|----------------|
| NewBuyer | 2026-01-01 | 120 |
| drop | 2026-01-01 | 310 |
| ChurnNewBuyer | 2026-01-01 | 15 |

### 3) Robust Preprocessing (No Explicit Event Calendar)
Promotional or operational events (e.g., SMS campaigns) can cause spikes/dips.
When an explicit event calendar is not available, we apply **robust smoothing** on the target series:

- Rolling median smoothing (window=7 days)

This helps the forecasting model remain stable in the presence of **unobserved event effects**.

### 4) Forecasting Model
We use **Holt-Winters (Additive)**:

- Trend: additive
- Seasonality: additive
- Seasonal period: **365** (yearly seasonality; suitable for datasets with 2+ years of daily history)

## Project Structure
