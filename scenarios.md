# Scenarios in Maudlin

A **scenario** in Maudlin represents a variation on a theme that you want to explore. Scenarios allow you to systematically test different configurations, feature modifications, or model architectures to achieve better results, such as improving performance metrics like AUC.

## Use Case Example:

In the Bank Marketing case study, the `duration` field is highly predictive. To enhance the AUC score, we can use scenarios to explore variations of the model configuration and feature engineering.

## Setting Up a Scenario

1. **Choose a Base Training Run**

   - Use the **interactive history viewer** to set a given training run as the base for your scenario.
   - Alternatively, edit the `run_metadata.json` file in the training directory (e.g., `~/src/_data/maudlin/trainings/bank-csv2`) to set the current run ID to your chosen base run ID.

2. **Build a Training Scenario**
   Run the following command to open the unit configuration file in an editor:
   ```bash
   mdln build-training-scenario
   ```

   - Modify the configuration in the editor.
   - Upon exiting, the system prompts you to:
     - Provide a **comment** describing the changes.
     - Decide whether to **optimize** before training with the modified configuration.
   - The changes are saved in `batch_config_changes.txt` under the unit's training directory.

3. **Edit the Configuration**
   To edit a scenario that hasn't been processed yet:
   ```bash
   mdln edit-training-scenario
   ```
   This lists unprocessed scenarios, allowing you to select one, edit its changes, and update the comment or optimization flag.

   - In the editor, make changes to reflect what you want to test in your scenario. For example, you could:
     - Modify hyperparameters.
     - Add or remove features.
     - Change model architecture.
   - You will also be prompted to provide a comment describing the changes made for the scenario.
   - Save and exit the editor to finalize the configuration for the first scenario.

4. **Create Additional Scenarios**

   - Run `mdln build-training-scenario` again to create and edit another configuration for a different scenario.
   - Repeat this process for as many variations as you want to test.

## Listing and Editing Scenarios

- **List Scenarios**:
  Use the following command to list all available scenarios:
  ```bash
  mdln list-training-scenario
  ```
- **Edit Scenarios**:
  Use the following command to choose a scenario to edit:
  ```bash
  mdln edit-training-scenario
  ```
  The selected scenario's configuration will be merged with the current config, allowing you to make additional changes.

## Running the Scenarios
  ```bash
  mdln build-training-scenario
  mdln train
  ```

`build-training-scenario` creates a file, `batch_config_changes.txt`. When `mdln train` is run, if `batch_config_changes.txt` exists, the code processes each scenario in sequence:
1. **Applies Configuration Changes**:
   - Each scenario's `sed_commands` are applied to the unit configuration file.
2. **Optimization**:
   - If the `optimize` flag is set, an optimization which narrows down the best hyperparameters for your scenario is run.
3. **Executes Training**:
   - Trains the model with the updated configuration.
4. **Saves Metadata**:
   - Stores the scenario's metadata, including the comment and changes applied, in the training run directory.

For each scenario:

- A new model is created.
- A full training run is performed based on the edited configuration.
- Normal outputs, such as logs, metrics, and visualizations, are generated.

## Data Structure

A file called `batch_config_changes.txt` stores each scenario as a JSON object with the following structure:

```json
{
    "comment": "Brief description of the configuration changes.",
    "sed_commands": [
        "sed-command-1",
        "sed-command-2"
    ],
    "optimize": true
}
```

- **`comment`**: Describes the changes made.
- **`sed_commands`**: Text-based search-and-replace instructions to modify the configuration.
- **`optimize`**: Whether to optimize the configuration before training (`true` or `false`).

## Optional: Hyperparameter Optimization

When creating a scenario, you can enable hyperparameter optimization. This step determines an ideal set of hyperparameters for the scenario, ensuring better training performance. These fields can also be edited later if needed.

## Benefits of Using Scenarios

- Enables systematic exploration of different configurations.
- Facilitates iterative improvement of model performance.
- Allows for side-by-side comparison of results from multiple variations.

By leveraging scenarios, Maudlin provides a powerful mechanism for refining your models and discovering the best-performing configurations for your specific use case.
