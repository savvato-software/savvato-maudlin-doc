# Documentation: Optimize Command in Maudlin

The `optimize` command in Maudlin leverages **Optuna** for hyperparameter optimization, enabling efficient tuning of machine learning models. This command performs automated model training and evaluation over multiple trials, applying advanced techniques such as adaptive learning rates and early stopping mechanisms. Below, we provide an in-depth guide for its usage and inner workings.

---

## Overview

The `optimize` command allows users to optimize a model's hyperparameters by:

1. Defining and running multiple optimization trials.
2. Configuring hyperparameter suggestions via Optuna.
3. Supporting pre-existing or new model configurations.
4. Utilizing adaptive learning rates and early stopping strategies.
5. Evaluating trial performance using custom metrics.
6. Storing trial results, best configurations, and logs for analysis.

---

## Usage

```bash
mdln optimize --trials <number_of_trials> [options]
```

### Arguments

- `--trials` (`-t`):
  Specifies the number of trials to run during optimization. Each trial corresponds to a unique set of hyperparameters.

- `--use-existing-model` (`-m`):
  Continue a previous optimization run.

- `--use-last-best-trials` (`-lbt`):
  Start a new optimization, constraining the hyperparameters using the best trials from the previous run.

### Example

Run an optimization with 50 trials:

```bash
mdln optimize --trials 50
```

Run an optimization using the last best trials:

```bash
mdln optimize --trials 20 --use-last-best-trials
```

---

## Workflow

1. **Configuration Parsing:**
   - The current unit's configuration is loaded using `get_current_unit_config`.
   - Runtime parameters (`use_existing_model`, `use_last_best_trials`) are added dynamically.

2. **Optimization Directory Initialization:**
   - A new directory is created for the optimization run.
   - Configurations and metadata are copied and updated.

3. **Data Preparation:**
   - Training, testing, and validation datasets are loaded and preprocessed via the `DataPreparationManager`.

4. **Pre-Optimization:**
   - Preprocessing steps, such as feature engineering, are executed.

5. **Objective Function:**
   - Defines the logic for evaluating hyperparameters using metrics such as `MAE`, `accuracy`, or `AUC`.
   - Implements feature masking if enabled.
   - Includes adaptive learning rate and early stopping mechanisms via callbacks.

6. **Optimization Execution:**
   - The optimization process is orchestrated using Optuna, which generates and evaluates trial configurations.
   - Each trial produces:
     - Hyperparameters.
     - Model architecture.
     - Performance metrics.

7. **Post-Optimization:**
   - Results, including best trial configurations, are saved in YAML format.
   - Logs are printed for review.

---

## Features

### Hyperparameter Suggestions

The following hyperparameters are optimized during each trial:

- Number of layers (`n_layers`)
- Units in the initial layer (`initial_layer_units`)
- Dropout rate (`dropout`)
- Learning rate (`lr`)
- Optimizer (`optimizer`)
- Activation function (`activation`)
- Batch size (`batch_size`)
- Optional: Feature masking and batch normalization

### Callbacks

- **Adaptive Learning Rate:**
  Dynamically adjusts the learning rate based on performance metrics.

- **Early Stopping:**
  Terminates trials early if performance improvement stagnates, using parameters like patience, delta, and top-N trial tracking.

### Metrics

Supports multiple evaluation metrics, including:

- `MAE` (Mean Absolute Error)
- `AUC` (Area Under the Curve)
- `accuracy`

---

## Output

### Directory Structure

- **Optimization Results:**
  - Saved under `<data_directory>/optimizations/<unit_name>/run_<run_id>`
  - Includes:
    - `config.yaml`: Updated configuration for the run.
    - `best_trials.yaml`: Details of the best trials.

### Logs

- Trial-specific logs and configurations are displayed during runtime.
- Summary of best trials, including parameters and metrics, is printed upon completion.

### Trial Evaluation

Each trial evaluates:
- Model architecture
- Feature subsets (if masking is enabled)
- Performance metrics

---

## Example Output

```
Trial Number: 3 | 47 trials remaining
Trial Params: {'n_layers': 4, 'dropout': 0.2, 'lr': 0.001, 'activation': 'relu'}

Best Params: {'n_layers': 4, 'dropout': 0.2, 'lr': 0.001, 'activation': 'relu'}
Finished at 2025-01-19 14:45:32
```

---

## Advanced Configuration

### Runtime Configuration

The runtime section of the configuration file (`config.yaml`) manages dynamic parameters:

```yaml
runtime:
  run_id: 12
  parent_run_id: 11
  use_existing_model: false
  use_last_best_trials: true
  base_optimization_run_id: 10
```

### Trial Constraints

Use `use_last_best_trials` to constrain trials to the top configurations from a previous optimization run.

---

## Best Practices

1. Use `--use-existing-model` to fine-tune a previously trained model.
2. Limit trials using `--trials` to save computation time during initial exploration.
3. Analyze `best_trials.yaml` for insights into hyperparameter influence on performance.

---

## Troubleshooting

1. **Config File Not Found:**
   Ensure the correct unit is set up and its configuration file exists in the expected directory.

2. **Metric Not Supported:**
   Verify that the primary metric in `config.yaml` matches supported metrics.

3. **Trial Errors:**
   Check for dataset integrity and configuration mismatches (e.g., unsupported layer types).

---

## References

- [Optuna Documentation](https://optuna.org)


