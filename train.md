# Documentation: `mdln train`

## Overview

The `mdln train` command is a central feature in Maudlin, designed to streamline the training of neural network models. By leveraging a unit-specific YAML configuration, it orchestrates:

1. **Data preprocessing and feature engineering**: Defined in the configuration.
2. **Model training**: Executes training using the specified architecture and parameters.
3. **Result generation**: Produces visualizations, logs, and outputs in a unique, run-specific directory.

This command ensures reproducibility, flexibility, and ease of experimentation through structured workflows and granular configuration options.

---

## Command Syntax

```bash
mdln train [options]
```

### Options

| Option                  | Description                                                              |
|-------------------------|--------------------------------------------------------------------------|
| `-e`, `--epochs`        | Number of epochs for this training run (overrides YAML setting).         |
| `-r`, `--run-id`        | Specify a training run ID for incremental training or debugging.         |
| `-m`, `--use-existing-model` | Use a pre-trained model from a previous run instead of initializing a new one. |

---

## Workflow and Functionality

### Step-by-Step Execution

1. **Unit-Specific Configuration**:  
   - The active unit's configuration (`UNIT-NAME.config.yaml`) defines data paths, model settings, training parameters, and feature engineering.
   - You must activate the desired unit using:
     ```bash
     mdln use <unit-name>
     ```

2. **Directory Initialization**:  
   - A new directory is created for the training run under:
     ```
     MAUDLIN_DATA_DIR/trainings/<unit-name>/run_<run-id>/
     ```
   - The YAML configuration is copied into this directory and updated dynamically with:
     - `run_id` and `parent_run_id`.
     - Any specified runtime parameters, such as `--epochs`.

3. **Preprocessing and Data Preparation**:  
   - Input and target functions are executed to preprocess data and prepare labels.
   - Feature engineering steps (e.g., normalization, SMOTE) are applied, if defined in the YAML file.

4. **Model Training**:  
   - The model is constructed as per the `model_architecture` section in the configuration.
   - Training is executed with callbacks, including:
     - **Adaptive Learning Rate**: Adjusts learning rates dynamically.
     - **Metric Tracking**: Tracks the best metrics and saves corresponding model weights.
     - **TensorBoard**: Logs training data for visualization.

5. **Post-Training**:  
   - Custom post-training functions process results and generate reports.
   - Visualizations (e.g., confusion matrix, ROC curve) are saved in the run directory.

---

## Batch Scenario Workflow

### Overview

The **batch scenario workflow** allows you to define, modify, and process multiple training scenarios in sequence. See [Scenarios](./scenarios.md) for details. 

---

## Key Features and Parameters

### **1. Epochs (`-e` or `--epochs`)**
Overrides the default number of epochs specified in the YAML configuration for this specific run. This is reflected in the copied configuration for the run.

**Example**:
```bash
mdln train -e 100
```

---

### **2. Use Existing Model (`-m`)**
If specified, uses the model from the previous training run (as recorded in the unit metadata) to continue training. This is useful for incremental learning or fine-tuning.

**Example**:
```bash
mdln train -m
```

---

### **3. Batch Scenarios**
By leveraging `mdln build-training-scenario` and `batch_config_changes.txt`, you can iterate through multiple configurations efficiently.

---

## Outputs

Upon completion, `mdln train` generates the following outputs in the run-specific directory:

1. **Model Files**:
   - Saved weights and architecture for the trained model.
2. **Logs**:
   - Loss and accuracy metrics, along with TensorBoard logs.
3. **Visualizations**:
   - Diagnostic plots and charts for performance analysis.
4. **Metadata**:
   - YAML file reflecting the runtime configuration.
   - Metadata about the training run (e.g., run ID, parent run ID).

---

## Example Usage

```bash
# Activate a unit
mdln use bank_marketing_unit

# Train with default settings
mdln train

# Train for 100 epochs
mdln train -e 100

# Continue training from a previous run
mdln train -m

# Process batch scenarios
mdln train
```

---

## Related Commands

- **`mdln new`**: Create a new unit.
- **`mdln list`**: List available units.
- **`mdln use`**: Activate a unit for training or prediction.
- **`mdln predict`**: Perform predictions using a trained model.
- **`mdln build-training-scenario`**: Create a new scenario with configuration changes.
- **`mdln list-training-scenarios`**: List all unprocessed and processed scenarios.
- **`mdln edit-training-scenario`**: Edit an unprocessed scenario.

---

## Conclusion

The `mdln train` command is a versatile and robust tool for neural network training. Combined with the **batch scenario workflow**, it enables efficient experimentation, optimization, and model evaluation for complex projects.

