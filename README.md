# Retail Transaction Simulator

## Overview

This project is a Python-based system designed to simulate daily customer shopping behavior in a retail environment. It generates synthetic transaction data over a configurable period, driven by predefined customer personas. Each persona has unique shopping habits, including purchase frequency, preferred shopping times, and a specific basket profile with item probabilities.

The primary goal is to create a realistic, structured dataset that can be used for analytics, model training, or evaluating data processing pipelines. This project was developed to demonstrate skills in data modeling, behavior simulation, and probabilistic sampling.

-----

## Features

  * **Persona-Driven Simulation**: The entire simulation is based on customizable customer personas defined in an easy-to-edit YAML file.
  * **Object-Oriented Design**: The code is structured logically using `Simulator`, `Persona`, and `Customer` classes for clarity, maintainability, and scalability.
  * **Probabilistic Logic**: Customer shopping decisions and basket contents are determined by probabilistic sampling, creating realistic variations in the data.
  * **Temporal Variation (Bonus Feature)**: The simulation includes a "weekend effect" where the probability of purchasing certain items (e.g., snacks) increases on Saturdays and Sundays.
  * **Configurable Parameters**: Key simulation settings like the number of customers per persona, duration of the simulation (in days), and output file format can be easily adjusted within the script.
  * **Flexible Output Formats**: The generated transaction data can be saved in either **CSV** or the more efficient **Apache Parquet** format.
  * **Professional Logging**: The script provides clear, timestamped log messages to monitor the simulation's progress and identify any issues.

-----

## Project Structure

The project is organized into the following directories and files:

```
.
├── config/
│   ├── personas.yaml      # Input file for defining customer personas.
│   └── products.csv       # Input file containing the list of products and their prices.
|
├── output/
│   └── transactions.csv   # Generated transaction data (or transactions.parquet).
|
├── transaction_simulator.py  # The main Python script containing the simulation logic.
|
└── README.md                 # This file.
```

-----

## How to Run

### 1\. Prerequisites

Ensure you have Python 3.6 or higher installed. The script relies on the following Python libraries:

  * `pyyaml`
  * `pandas`
  * `pyarrow`
  * `fastparquet`

### 2\. Installation

Install the required libraries using pip:

```bash
pip install pyyaml pandas pyarrow fastparquet
```

### 3\. Configuration

Before running the simulation, you can customize the input data and simulation parameters.

**a. Define Personas:**
Modify the `config/personas.yaml` file to define the customer personas. Each persona requires a `name`, `frequency`, `preferred_time`, and a `basket_profile`.

*Example:*

```yaml
personas:
  - name: "Health-Conscious Parent"
    frequency: "weekly"
    preferred_time: ["9:00am–11:00am"]
    basket_profile:
      fruit:
        probability: 0.9
        quantity: [3, 5]
      vegetables:
        probability: 0.9
        quantity: [4, 6]
      milk:
        probability: 0.8
        quantity: [1, 2]
      cereal:
        probability: 0.6
        quantity: [1, 1]
```

**b. Set Product Prices:**
Update the `config/products.csv` file with the items that can appear in a customer's basket and their corresponding prices. The file must have `item_name` and `price` columns.

*Example:*

```csv
item_name,price
fruit,3.50
vegetables,2.75
milk,4.25
cereal,5.50
snacks,2.00
```

**c. Adjust Simulation Parameters:**
Open the main Python script (`transaction_simulator.py`) and modify the properties of the `SimConfig` class to control the simulation:

```python
class SimConfig:
    personas_path = "config/personas.yaml"
    products_path = "config/products.csv"
    output_path = "output/transactions"
    num_customers = 100   # Number of customers to simulate for EACH persona
    sim_days = 30         # Total number of days to run the simulation
    format = 'csv'        # Output format: 'csv' or 'parquet'
```

### 4\. Execute the Script

Run the simulator from your terminal:

```bash
python transaction_simulator.py
```

The script will log its progress to the console and, upon completion, save the transaction data in the `output/` directory.

-----

## Design Decisions

The simulation is built around three core classes to ensure a clear separation of concerns:

1.  **`Persona`**: A simple data class that holds the static shopping attributes defined in `personas.yaml`. It represents a *type* of customer.

2.  **`Customer`**: Represents an individual agent in the simulation. Each `Customer` is assigned a `Persona` and maintains its own state, most importantly the `last_shopping_day`. This stateful design allows the simulator to make decisions for each customer independently.

3.  **`Simulator`**: The main orchestrator. It is responsible for:

      * Loading all configuration and input data.
      * Initializing the population of `Customer` objects.
      * Running the main daily loop, iterating through each day of the simulation.
      * For each day, polling every customer to see if they should shop (`should_shop_today`). This logic is based on their persona's frequency and their last visit.
      * Generating a probabilistic shopping basket (`_generate_basket`) for customers who decide to shop.
      * Aggregating all transactions and saving them to the desired output format.

This object-oriented approach makes the system modular and easy to extend. For instance, adding new temporal effects (like holiday sales) or more complex customer behaviors would only require modifying specific methods without restructuring the entire application.

-----

## Output Data Format

The final output file (`transactions.csv` or `transactions.parquet`) will contain the following columns:

| Column           | Description                                                 | Data Type |
| ---------------- | ----------------------------------------------------------- | --------- |
| `transaction_id` | A unique identifier for each transaction (basket).          | Integer   |
| `date`           | The date of the transaction (YYYY-MM-DD).                   | String    |
| `timestamp`      | The full timestamp of the transaction (YYYY-MM-DD HH:MM:SS). | String    |
| `customer_id`    | A unique identifier for the customer.                       | String    |
| `customer_persona` | The name of the persona assigned to the customer.         | String    |
| `item_name`      | The name of the product purchased.                          | String    |
| `quantity`       | The number of units of the item purchased.                  | Integer   |
| `price_per_item` | The price of a single unit of the item.                     | Float     |
| `total_price`    | The total price for the item (`quantity` \* `price_per_item`). | Float     |
