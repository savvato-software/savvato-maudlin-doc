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

## Writing a Featurizer in Maudlin

Featurizers in Maudlin are Python functions used to preprocess and transform datasets before input and target extraction. They are defined in the **`savvato-maudlin-lib`** library, serving as a temporary location for development and iteration. Featurizers operate sequentially, following the order specified in the YAML configuration, enabling complex transformations through chaining.

---

### **Key Requirements for Featurizers**

1. **Function Signature:**  
   Each featurizer must define an `apply()` function that accepts a dataset and any required parameters.  

2. **Input and Output:**  
   - **Input:** A Pandas DataFrame containing the dataset.  
   - **Output:** The transformed DataFrame returned for further processing.  

3. **Serialization Support:**  
   Use `@register_keras_serializable` to ensure compatibility with Keras models and persistence across training and prediction.  

4. **Parameterization:**  
   All configurable options should be passed through the YAML file, avoiding hardcoded values.  

---

### **Example Featurizer: Target Encoding**

#### **Python Implementation**  
```python
import pandas as pd
from keras.saving import register_keras_serializable

@register_keras_serializable(package="CustomPackage")
def apply(data, columns, target_column='y', smoothing=10.0):
    """
    Target Encoding Featurizer.

    Args:
        data (pd.DataFrame): Input dataset.
        columns (list): Categorical columns to encode.
        target_column (str): Name of the target column.
        smoothing (float): Smoothing factor to handle low-frequency categories.

    Returns:
        pd.DataFrame: Transformed dataset with encoded features.
    """
    # Compute global mean
    global_mean = data[target_column].mean()

    # Apply target encoding to each specified column
    for column in columns:
        stats = data.groupby(column)[target_column].agg(['mean', 'count'])
        stats['smooth'] = (
            (stats['mean'] * stats['count'] + global_mean * smoothing) /
            (stats['count'] + smoothing)
        )
        data[column] = data[column].map(stats['smooth']).fillna(global_mean)

    return data
```

#### **YAML Configuration**  
```yaml
features:
  - name: target_encode
    params:
      columns: [job, marital, education]
      target_column: 'y'
      smoothing: 10.0
```

#### **What It Does:**  
- Computes smoothed mean target values for specified categorical columns.  
- Reduces dimensionality while preserving relationships with the target variable.  
- Handles low-frequency categories using smoothing to prevent overfitting.  

---

### **Featurizer Workflow**

1. **Define the Function:** Write the preprocessing logic using Pandas or NumPy.  
2. **Decorate for Serialization:** Use `@register_keras_serializable` to ensure compatibility with the Keras pipeline.  
3. **Parameterize Inputs:** Accept dynamic parameters like columns and smoothing factors via YAML.  
4. **Return Transformed Data:** Ensure the function outputs a modified DataFrame with the desired features.  
5. **Test Independently:** Validate the featurizer on sample datasets before integrating it into the Maudlin pipeline.  

---

### **Additional Examples**

#### **1. Winsorization (Outlier Handling)**  

**Python Implementation**  
```python
import numpy as np

def apply(data, columns, lower_percentile=0.01, upper_percentile=0.99):
    for col in columns:
        lower = data[col].quantile(lower_percentile)
        upper = data[col].quantile(upper_percentile)
        data[col] = np.clip(data[col], lower, upper)
    return data
```

**YAML Configuration**  
```yaml
- name: winsorize
  params:
    columns: [balance]
    lower_percentile: 0.01
    upper_percentile: 0.99
```

---

#### **2. Interaction Terms (Feature Multiplication)**  

**Python Implementation**  
```python
def apply(data, columns, method='multiply'):
    col1, col2 = columns
    if method == 'multiply':
        data[f"{col1}_{col2}_interaction"] = data[col1] * data[col2]
    return data
```

**YAML Configuration**  
```yaml
- name: interaction_term
  params:
    columns: [age, balance]
    method: multiply
```

---

### **Best Practices for Writing Featurizers**  

- **Modular Design:** Focus each featurizer on a single, well-defined transformation.  
- **Chaining Support:** Ensure compatibility with prior and subsequent transformations by assuming data may already be altered.  
- **Parameter Flexibility:** Avoid hardcoding parameters; rely on YAML for configuration.  
- **Error Handling:** Include checks for missing or invalid columns to prevent runtime errors.  
- **Testing and Debugging:** Test on sample data and log intermediate outputs for debugging complex chains.  
- **Reproducibility:** Use consistent names and column outputs to simplify debugging and reuse.  

---

# Common Featurizers  

## 1. Bin Numeric  
- **Purpose**: Group continuous numeric values into discrete bins or intervals.  
- **Example**: Categorize ages into bins like 0–18, 19–35, 36–50, and 51+.  

## 2. Drop Column  
- **Purpose**: Remove unnecessary or redundant columns from the dataset.  
- **Example**: Drop columns like `customer_id` or `timestamp`.  

## 3. Frequency Encode  
- **Purpose**: Replace categorical values with their frequency counts.  
- **Example**: Replace city names with the count of occurrences for each city.  

## 4. Target Encode  
- **Purpose**: Replace categorical values with the mean of the target variable for each category.  
- **Example**: Encode product categories based on average sales.  

## 5. Interaction Term  
- **Purpose**: Create features that combine two or more variables to capture relationships.  
- **Example**: Multiply `price` and `quantity_sold` to generate `revenue`.  

## 6. Min-Max Scale  
- **Purpose**: Scale numeric features to a specified range, usually [0, 1].  
- **Example**: Normalize income values between 0 and 1.  

## 7. Moving Average  
- **Purpose**: Smooth time-series data by averaging values over a sliding window.  
- **Example**: Calculate 7-day and 30-day moving averages for sales data.  

## 8. One-Hot Encode  
- **Purpose**: Convert categorical variables into binary indicator columns.  
- **Example**: Encode gender into two columns: `male` and `female`.  

## 9. Winsorize  
- **Purpose**: Limit extreme values (outliers) by capping them at specified percentiles.  
- **Example**: Cap incomes above the 95th percentile and below the 5th percentile.  

---

## Featurizer Loading

Featurizers are Python functions dynamically loaded by their names specified in the YAML file. The Maudlin framework applies them sequentially during preprocessing.

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

