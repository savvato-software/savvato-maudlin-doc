# Documentation: `mdln train`

## Overview

The `mdln train` command in Maudlin is used to train a neural network model using the specified configuration and dataset. It handles the full training pipeline, including feature engineering, model creation, and evaluation.

---

## Command Syntax

```bash
mdln train
```

### Description

This command:

1. Loads the configuration specified in the active Maudlin unit.
2. Processes the input data as defined in the YAML configuration.
3. Constructs and exercises the neural network model to get predictions
4. Generates outputs, logs, and visualizations in the results directory.

---

## Prerequisites

- Ensure that a Maudlin unit has been created using:
  ```bash
  mdln new <unit-name> --training_csv_path=./path/to/train.csv --prediction_csv_path=./path/to/predict.csv
  ```
- Verify the configuration file (e.g., `UNIT-NAME.config.yaml`) contains at least initial, if not time-tested settings for:
  - Data preprocessing steps.
  - Model architecture.
  - Training parameters.

---

## Configuration Settings

The `mdln train` command uses the configuration YAML file to define training parameters.

### Example Configuration

```yaml
model_architecture:
  - layer_type: LSTM
    units: 64
    return_sequences: True
  - layer_type: Dense
    units: 1
    activation: 'linear'
optimizer: 'adam'
loss_function: 'mse'
metrics: ['mae', 'accuracy']
epochs: 50
batch_size: 32
pre_training:
  diagrams:
  - oversampling
  - correlation_matrix
  - boxplot
  oversampling:
    calculate_long_running_diagrams: false
    console_logging: true
    enabled: true
    k_neighbors: 3
    method: adasyn
    n_jobs: -1
    random_state: 42
    run_pca_before: false
    sampling_strategy: 0.8
training:
  adaptive_learning_rate:
    patience: 5
    factor: 0.85
    min-lr: 1e-6
post_training:
  diagrams:
  - confusion_matrix
  - precision_recall_curve
  - roc_curve

```

---

## Custom Functions

You can influence the data being trained on, and implement custom handling of the results once the training is done, by using unit-specific workflow functions. In your unit, by default at `MAUDLIN_DATA_DIR/functions`, you have the following:

- `input_function`: Prepares and processes input data for training.
- `target_function`: Defines target values and labels for supervised learning.
- `pre_training_function`: Executes preprocessing steps before training starts, such as normalization or augmentation.
- `post_training_function`: Processes outputs and generates reports after training.
- `pre_prediction_function` (*): Prepares data before making predictions (not used during training).
- `post_prediction_function` (*): Processes outputs after predictions are made (not used during training).

(*) Functions marked with an asterisk are specific to prediction workflows and are not invoked during training.

These functions allow granular customization to align data preparation and post-processing with specific project needs.

---

## Outputs

Upon completion, `mdln train` generates the following outputs:

1. Model files, weights, and architecture.
2. Training logs including loss and accuracy metrics.
3. Pre- and post-training visualizations:
   - Confusion matrix
   - Precision-recall curve
   - ROC curve
4. Feature importance analysis using SHAP.
5. Diagnostic plots, if enabled.

---

## Example Usage

```bash
mdln use bank_marketing_unit
mdln train
```

This example uses the `bank_marketing_unit` configuration and trains a model based on its settings.

---

## Error Handling

If errors occur, check:

1. YAML configuration syntax.
2. Paths to CSV files in the configuration.
3. Compatibility of data columns with defined features.
4. Logs generated in the output directory for debugging.

---

## Additional Notes

- Use `mdln list` to verify available units.
- Activate a unit with `mdln use <unit-name>` before training.
- Customize pre- and post-training steps by defining additional Python functions in the `functions/` directory.

---

## Related Commands

- `mdln new` - Create a new Maudlin unit.
- `mdln predict` - Perform predictions using a trained model.
- `mdln list` - List all available units.
- `mdln use` - Select a unit for training or prediction.

---

## Conclusion

The `mdln train` command simplifies the end-to-end training process in Maudlin, leveraging YAML configurations for flexible customization. It supports advanced features such as SMOTE, adaptive learning rates, and post-training visualizations for comprehensive model evaluation.

