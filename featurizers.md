# Featurizers in Maudlin

## Overview

Featurizers in Maudlin are preprocessing functions written in Python, applied sequentially to transform raw data before input and target extraction. They handle tasks like dropping columns, merging data, and encoding categories. Defined in the YAML configuration, their order determines execution, enabling chained transformations for refined feature engineering. Flexible and customizable, featurizers streamline data preparation while ensuring consistency and reproducibility.

---

## Purpose of Featurizers

They do anything you want to do to the set of features coming from your input CSV, and in the order you specify. This includes.

1. Data Transformation: Modify raw data into features suitable for modeling.
2. Column Management: Merge, split, rename, or drop columns as needed.
3. Feature Engineering: Create new features, such as moving averages or categorical encodings.
4. Data Cleaning: Handle missing values, outliers, or inconsistent formats.
5. Aggregation: Compute statistics like sums, means, or counts over grouped data.
6. Sequencing Operations: Apply transformations in a defined order for complex workflows.
7. Consistency and Reproducibility: Ensure preprocessing steps are standardized across runs.
8. Customization: Support domain-specific logic through custom Python functions.

---

## Configuration in YAML

Featurizers are defined in the unit configuration file (e.g., `UNIT-NAME.config.yaml`) under the `data.features` section.

### Example YAML Configuration

```yaml
data:
  columns:
    csv:
      - 'date'
      - 'product_id'
      - 'quantity_sold'
      - 'price_per_unit'
      - 'total_revenue'
  features:
    - name: day_of_the_week
    - name: holiday_indicator
    - name: moving_average
      params:
        periods: [3, 7]
        data_field_name: 'quantity_sold'
```

### Key Elements:

- **name**: Identifier for the featurizer.
- **params**: Parameters to customize behavior, e.g., window size, fields, etc.

---

## Writing Featurizer Functions

Featurizer functions are defined in `savvato-maudlin-lib`. Eventually we will make them user-definable, but since we're the only user at the moment, that's where they are at.  Each function must adhere to the following signature:

### Example Featurizer Function

```python
from keras.saving import register_keras_serializable

@register_keras_serializable(package="CustomPackage")
def apply(data, periods, data_field_name, include_differences=False):
    import pandas as pd
    
    for period in periods:
        data[f'moving_avg_{period}'] = data[data_field_name].rolling(window=period).mean()
        
        if include_differences:
            data[f'moving_avg_diff_{period}'] = data[data_field_name] - data[f'moving_avg_{period}']
    return data
```

### Function Components:

1. **Parameters**:
   - **data**: Input dataframe.
   - **periods**: List of time periods for calculations.
   - **data\_field\_name**: Column name for the input data.
   - **include\_differences**: Optional flag to compute differences.
2. **Processing**:
   - Calculate rolling averages.
   - Optionally compute differences between actual and average values.
3. **Return**:
   - Modified dataframe with new features added.

---

## Common Featurizers

1. **Rolling Averages**:

   - Purpose: Smooth time-series data.
   - Example: Compute averages for 3-day and 7-day windows.

2. **Lags**:

   - Purpose: Introduce past values as predictors.
   - Example: Shift values by 1, 3, or 7 periods.

3. **One-Hot Encoding**:

   - Purpose: Convert categorical variables into binary indicators.

4. **Normalization and Scaling**:

   - Purpose: Normalize numeric features between 0 and 1 or scale to standard deviation.

5. **Date Features**:

   - Purpose: Extract day, month, or day-of-week features from date columns.

---

## Featurizer Loading

Featurizers are dynamically loaded by their names specified in the YAML file. The Maudlin framework applies them sequentially during preprocessing.

### YAML Reference:

```yaml
features:
  - name: moving_average
    params:
      periods: [3, 5]
      data_field_name: 'sales'
  - name: lag_features
    params:
      lags: [1, 3]
      data_field_name: 'sales'
```

---

## Debugging and Validation

- Ensure functions are stateless and operate only on input data.
- Log intermediate outputs to verify feature creation.
- Validate outputs by visualizing features using histograms, line plots, or scatter plots.

### Example Debug Function:

```python
print(data[['date', 'quantity_sold', 'moving_avg_3', 'moving_avg_7']].head(10))
```

---

## Best Practices

1. Use descriptive names for features.
2. Document assumptions and parameters in the code.
3. Test featurizers independently before integrating them into workflows.
4. Apply feature selection to reduce dimensionality.
5. Consider computational efficiency for large datasets.

---

## Conclusion

Featurizers are a powerful tool in Maudlin to transform raw data into features that enhance model performance. By leveraging YAML configurations and reusable Python functions, Maudlin ensures flexibility and scalability in feature engineering. Proper validation and testing of featurizers can significantly impact model accuracy and robustness.

