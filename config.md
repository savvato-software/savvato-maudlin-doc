# Maudlin Configuration Documentation

This document provides detailed explanations of the configuration properties used in Maudlin. Each section describes the purpose and usage of specific settings.

---

## Batch Settings

### `batch_size`
- **Description:** The number of samples processed in a single forward/backward pass.
- **Example:** `8`

---

## Class Weights

### `classes`
- **Description:** Defines the class labels, their names, and associated weights for handling class imbalance.
- **Structure:**
  ```yaml
  classes:
  - label: <int>  # Integer label for the class
    name: <string>  # Name of the class
    weight: <float>  # Weight assigned to the class
  ```
- **Example:**
  ```yaml
  classes:
  - label: 0
    name: no_event
    weight: 1
  - label: 1
    name: event
    weight: 1.95
  ```

---

## Data Configuration

### `data`
- **Description:** Defines input data files, column configurations, and feature engineering steps.
- **Structure:**
  ```yaml
  data:
    training_file: <path_to_training_file>
    prediction_file: <path_to_prediction_file>
    columns:
      csv:  # Original columns from the CSV
      - <list_of_columns>
      final:  # Columns used after feature engineering
      - <list_of_columns>
    features:
      - name: <feature_name>
        params:
          <feature-specific_parameters>
  ```

### Feature Engineering Steps

#### `frequency_encode`
- Encodes categorical variables based on the frequency of values.
- **Parameters:**
  - `columns`: List of columns to encode.
- **Example:**
  ```yaml
  - name: frequency_encode
    params:
      columns:
      - poutcome
      - contact
  ```

#### `target_encode`
- Encodes categorical variables based on the mean target value.
- **Parameters:**
  - `columns`: List of columns to encode.
  - `smoothing`: Smoothing factor to control weight between prior and target.
  - `target_column`: Column used for the target variable.
- **Example:**
  ```yaml
  - name: target_encode
    params:
      columns:
      - job
      - marital
      smoothing: 12.0
      target_column: y
  ```

#### `drop_column`
- Removes specified columns from the dataset.
- **Parameters:**
  - `columns`: List of columns to drop.
- **Example:**
  ```yaml
  - name: drop_column
    params:
      columns:
      - poutcome
  ```

#### `interaction_term`
- Creates interaction terms between specified columns.
- **Parameters:**
  - `columns`: List of columns to interact.
  - `method`: Method of interaction (e.g., `multiply`).
- **Example:**
  ```yaml
  - name: interaction_term
    params:
      columns:
      - job
      - education
      method: multiply
  ```

#### `min_max_scale`
- Scales numerical features to a specified range (0 to 1).
- **Parameters:**
  - `columns`: List of columns to scale.
- **Example:**
  ```yaml
  - name: min_max_scale
    params:
      columns:
      - age
      - balance
  ```

#### `bin_numeric`
- Bins a numerical column into discrete intervals.
- **Parameters:**
  - `column`: Column to bin.
  - `bins`: List of bin edges.
  - `labels`: Labels for each bin.
- **Example:**
  ```yaml
  - name: bin_numeric
    params:
      column: duration
      bins:
      - 0
      - 60
      - 180
      labels:
      - 1
      - 2
  ```

#### `winsorize`
- Limits extreme values in a numerical column.
- **Parameters:**
  - `columns`: List of columns to winsorize.
  - `lower_percentile`: Lower bound for winsorization.
  - `upper_percentile`: Upper bound for winsorization.
- **Example:**
  ```yaml
  - name: winsorize
    params:
      columns:
      - balance
      lower_percentile: 0.01
      upper_percentile: 0.99
  ```

---

## Model Configuration

### `model_architecture`
- **Description:** Defines the architecture of the model.
- **Structure:**
  ```yaml
  model_architecture:
  - layer_type: <layer_type>
    units: <number_of_units>
    activation: <activation_function>
    rate: <dropout_rate>  # Optional for Dropout layers
  ```
- **Example:**
  ```yaml
  model_architecture:
  - layer_type: Dense
    units: 50
    activation: relu
  - layer_type: Dropout
    rate: 0.1
  - layer_type: Dense
    units: 1
    activation: sigmoid
  ```

### `optimizer`
- **Description:** Optimization algorithm used during training.
- **Example:** `adam`

### `loss_function`
- **Description:** Loss function to minimize during training.
- **Example:** `binary_crossentropy`

### `metrics`
- **Description:** Metrics to evaluate the model’s performance.
- **Example:**
  ```yaml
  metrics:
  - auc
  - accuracy
  ```

### `epochs`
- **Description:** Number of training iterations over the entire dataset.
- **Example:** `250`

### `learning_rate`
- **Description:** Initial learning rate for training.
- **Example:** `0.000875`

---

## Optimization

### `optimization`
- **Description:** Settings for hyperparameter tuning and optimization.
- **Structure:**
  ```yaml
  optimization:
    n_trials: <number_of_trials>
    optimizer: <list_of_optimizers>
    learning_rate:
      min: <min_lr>
      max: <max_lr>
    n_layers:
      min: <min_layers>
      max: <max_layers>
  ```
- **Example:**
  ```yaml
  optimization:
    n_trials: 200
    optimizer:
    - adam
    learning_rate:
      min: 1e-7
      max: 0.001
    n_layers:
      min: 2
      max: 3
  ```

---

## Pre- and Post-Training

### `pre_training`
- **Description:** Settings and visualizations applied before training.
- **Example:**
  ```yaml
  pre_training:
    diagrams:
    - correlation_matrix
    oversampling:
      enabled: true
      method: adasyn
    ```

### `post_training`
- **Description:** Outputs generated after training.
- **Example:**
  ```yaml
  post_training:
    diagrams:
    - confusion_matrix
    - roc_curve
    ```

---

## Prediction

### `prediction`
- **Description:** Settings for making predictions.
- **Structure:**
  ```yaml
  prediction:
    threshold: <threshold>
    perturbation:
      enabled: <true_or_false>
      features:
      - name: <feature_name>
        type: <feature_type>
  ```
- **Example:**
  ```yaml
  prediction:
    threshold: 0.5
    perturbation:
      enabled: true
      features:
      - name: age
        type: int
        range: [-10, 20]
      - name: housing
        type: binary
  ```

---
