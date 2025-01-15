# Scenarios in Maudlin

A **scenario** in Maudlin represents a variation on a theme that you want to explore. Scenarios allow you to systematically test different configurations, feature modifications, or model architectures to achieve better results, such as improving performance metrics like AUC.

## Use Case Example:

In the Bank Marketing case study, the `duration` field is highly predictive. To enhance the AUC score, we can use scenarios to explore variations of the model configuration and feature engineering.

## Setting Up a Scenario

1. **Choose a Base Training Run**

   - Use the **interactive history viewer** to set a given training run as the base for your scenario.
   - Alternatively, edit the `run_metadata.json` file in the training directory (e.g., `~/src/_data/maudlin/trainings/bank-csv2`) to set the current run ID to your chosen base run ID.

2. **Build a Training Scenario**

   - Run the following command to initiate the scenario creation process:
     ```bash
     mdln build-training-scenario
     ```
   - This command opens a `VI` editor with the most recent unit configuration file.

3. **Edit the Configuration**

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

For each scenario:

- A new model is created.
- A full training run is performed based on the edited configuration.
- Normal outputs, such as logs, metrics, and visualizations, are generated.

## Optional: Hyperparameter Optimization

When creating a scenario, you can enable hyperparameter optimization. This step determines an ideal set of hyperparameters for the scenario, ensuring better training performance. These fields can also be edited later if needed.

## Benefits of Using Scenarios

- Enables systematic exploration of different configurations.
- Facilitates iterative improvement of model performance.
- Allows for side-by-side comparison of results from multiple variations.

By leveraging scenarios, Maudlin provides a powerful mechanism for refining your models and discovering the best-performing configurations for your specific use case.

