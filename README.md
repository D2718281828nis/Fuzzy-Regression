# Fuzzy Time Series Forecasting - README

## 📋 Overview

This notebook demonstrates **Fuzzy Time Series (FTS) forecasting** using historical enrollment data (1971-1992). The implementation follows the methodology and compares FTS predictions against:
- **Linear Regression**
- **Simple Linear Interpolation**

### Goal
Forecast the 1992 year value using data from 1971-1991, then evaluate accuracy using **Mean Absolute Percentage Error (MAPE)**.

---

## How It Works: 6-Step Fuzzy Time Series Algorithm

### Step 1: Define the Universe of Discourse (U)
```python
D_min = np.min(training_data)
D_max = np.max(training_data)
D1 = 0.1 * D_min  # Buffer from paper
D2 = 0.1 * D_max
U = [D_min - D1, D_max + D2]  # Extended range
```
- Establishes the complete range of possible values
- Adds 10% buffer on both ends to accommodate future variations

### Step 2: Partition U into Equal Intervals
```python
n_intervals = 5
interval_length = (U_max - U_min) / n_intervals
```
- Divides the universe into `n` equal-length intervals
- Each interval represents a linguistic term (e.g., "Low", "Medium", "High")

### Step 3: Define Triangular Fuzzy Membership Functions
```python
def membership_degree(value, fuzzy_set_params):
    # Triangular membership: rises linearly to peak, then falls
    if left_base < value <= center:
        return (value - left_base) / (center - left_base)
    elif center < value < right_base:
        return (right_base - value) / (right_base - center)
```
- Each fuzzy set (A₁, A₂, ..., Aₙ) has a **triangular membership function**
- A crisp value can belong to multiple sets with varying degrees (0 to 1)
- Enables smooth transitions between linguistic categories

### Step 4: Fuzzify Training Data
```python
def fuzzify_data_point(data_point, fuzzy_sets, intervals):
    # Assign each crisp value to the fuzzy set with highest membership
    return best_fuzzy_set_name  # e.g., "A3"
```
- Converts numerical time series → sequence of fuzzy labels
- Example: `15603 → A3`, `18150 → A4`

### Step 5: Build Fuzzy Logical Relationship Groups (FLRGs)
```python
# From sequence: [A1, A1, A2, A2, A2, A3, ...]
# Extract relationships: A1→A1, A1→A2, A2→A2, A2→A3, ...
relationships = {
    'A1': ['A1', 'A2'],
    'A2': ['A2', 'A2', 'A2', 'A3', ...],
    'A3': ['A3', 'A3', 'A3', 'A2', 'A4'],
    ...
}
```
- Captures temporal patterns: *"If last year was A₃, what states followed historically?"*
- Forms the **knowledge base** for forecasting

### Step 6: Defuzzify & Forecast
Three inference rules handle different relationship scenarios:

| Rule | Condition | Forecast Calculation |
|------|-----------|---------------------|
| **Rule 1** | No historical relationship | Use midpoint of current fuzzy set |
| **Rule 2** | One-to-one relationship | Use midpoint of the single next state |
| **Rule 3** | One-to-many relationship | Average midpoints of all possible next states |

```python
# Example: Current state = A4, historical next states = [A4, A4, A4]
forecast = average([midpoint_A4, midpoint_A4, midpoint_A4]) = midpoint_A4
```

---

## Results Comparison (1992 Forecast)

| Method | Forecast | Actual | MAPE |
|--------|----------|--------|------|
| **Fuzzy Time Series** | 18,414.34 | 18,876 | **2.45%** |
| Linear Regression | 18,687.22 | 18,876 | **1.00%** ✅ |
| Simple Interpolation | 19,346.00 | 18,876 | **2.49%** |

> In this example, Linear Regression performed best. However, FTS excels when:
> - Data has **non-linear patterns** or **linguistic trends**
> - Relationships are **qualitative** rather than strictly numerical
> - Uncertainty and vagueness are inherent in the domain

---

## Fuzzy Logic for Regression: Key Concepts

### What is Fuzzy Regression?
Unlike classical regression that fits a precise mathematical function (e.g., `y = mx + b`), **fuzzy regression**:
- Models relationships using **fuzzy sets** and **linguistic rules**
- Handles **imprecise, noisy, or subjective data**
- Outputs **fuzzy predictions** (ranges with membership degrees) rather than point estimates

### Why Use Fuzzy Logic for Time Series?

| Advantage | Explanation |
|-----------|-------------|
| **Handles Uncertainty** | Real-world data often has measurement error, missing values, or inherent vagueness |
| **Linguistic Interpretability** | Rules like *"If enrollment was High last year, it tends to stay High"* are human-readable |
| **Non-linear Modeling** | No assumption of linearity; captures complex temporal dependencies |
| **Robust to Outliers** | Membership functions smooth extreme values rather than overfitting them |

### FTS vs. Classical Regression

| Feature | Fuzzy Time Series | Linear Regression |
|---------|------------------|-------------------|
| **Model Type** | Rule-based, linguistic | Parametric, mathematical |
| **Assumptions** | Minimal; data-driven patterns | Linearity, homoscedasticity, independence |
| **Output** | Crisp value via defuzzification | Direct point prediction |
| **Interpretability** | High (explicit rules) | Medium (coefficients only) |
| **Best For** | Qualitative trends, uncertain data | Clear linear relationships, precise data |

---

## References

1. Tsaur, R.-C. (2012). A fuzzy time series-Markov chain model with an application to forecast the exchange rate between the Taiwan and US dollar. *International Journal of Innovative Computing, Information and Control*, 8(11(B)), 4931–4942.
2. Song, Q., & Chissom, B. S. (1993). Fuzzy time series and its models. *Fuzzy Sets and Systems*, 54(3), 269-277.
3. Zadeh, L. A. (1965). Fuzzy sets. *Information and Control*, 8(3), 338-353.


> **Based on**: Tsaur, R.-C. (2012). *A fuzzy time series-Markov chain model with an application to forecast the exchange rate between the Taiwan and US dollar*. International Journal of Innovative Computing, Information and Control.
