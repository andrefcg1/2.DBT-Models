# DBT Models for Player Transaction Analysis

This project contains three DBT models that analyze player transaction data based on the provided CSV files.

## Data Sources

The models use three seed tables (CSV files in the `data/` directory):
- `affiliates_extended.csv` - Contains affiliate information with codes and origins
- `players_extended.csv` - Contains player information including KYC status and country
- `transactions_extended.csv` - Contains transaction data with amounts and types

These CSV files are loaded as seeds using `dbt seed` command.

## Models

### Model 1: Daily Player Transactions (`model_1_daily_player_transactions.sql`)
**Purpose**: Creates one row per player per day with separate columns for deposits and withdrawals.

**Key Features**:
- Aggregates transactions by player and date
- Deposits are shown as positive values
- Withdrawals are shown as negative values (as requested)
- Orders results by player ID and transaction date

**Output Columns**:
- `player_id`: Player identifier
- `transaction_date`: Date of transactions
- `deposits`: Total deposits for the day
- `withdrawals`: Total withdrawals for the day (negative values)

### Model 2: KYC Discord Deposits by Country (`model_2_kyc_discord_deposits.sql`)
**Purpose**: Calculates sum and count of deposits per country for KYC-approved players who came from Discord affiliates.

**Key Features**:
- Filters for players with `is_kyc_approved = true`
- Filters for players with affiliate origin = 'Discord'
- Aggregates deposit amounts and counts by country
- Orders results by total deposits (descending)

**Output Columns**:
- `country_code`: Player's country code
- `total_deposits_by_country`: Sum of all deposits for the country
- `total_deposit_count_by_country`: Count of all deposits for the country

### Model 3: Top Three Deposits per Player (`model_3_top_three_deposits.sql`)
**Purpose**: Creates one row per player showing their three largest deposit amounts.

**Key Features**:
- Uses window functions to rank deposits by amount
- Only includes deposit transactions
- Shows the three largest deposits in separate columns
- Orders results by player ID

**Output Columns**:
- `player_id`: Player identifier
- `largest_deposit`: Player's highest deposit amount
- `second_largest_deposit`: Player's second highest deposit amount
- `third_largest_deposit`: Player's third highest deposit amount

## Usage

To run these models, you'll need to:

1. Install DBT with DuckDB adapter: `pip install dbt-core dbt-duckdb`
2. Install packages: `dbt deps` (installs dbt_utils for additional tests)
3. Run `dbt seed` to load the CSV files from the `data/` directory into DuckDB
4. Run `dbt run` to execute all models
5. Use `dbt test` to run data quality tests

The CSV files are automatically loaded as seed tables when you run `dbt seed`, making them available for the models to reference. DuckDB will create a local `analytics.duckdb` file to store the data.

## Data Quality Tests

The project includes comprehensive data quality tests:

### **Seed Tests:**
- **Unique constraints** on all ID columns
- **Not null constraints** on primary keys

### **Model Tests:**
- **Model 1**: Unique combination of player_id + transaction_date, deposits ≥ 0, withdrawals ≤ 0
- **Model 2**: Unique country codes, positive deposit amounts and counts
- **Model 3**: Unique player IDs, largest deposit ≥ second largest ≥ third largest

### **Custom Business Logic Tests:**
- `test_withdrawals_are_negative.sql` - Ensures withdrawals are negative
- `test_deposits_are_positive.sql` - Ensures deposits are positive  
- `test_deposit_order.sql` - Validates deposit amounts are properly ordered
- `test_kyc_discord_filtering.sql` - Verifies KYC/Discord filtering logic

Run tests with: `dbt test`

## File Structure

```
├── dbt_project.yml          # DBT project configuration
├── profiles.yml             # DBT profiles configuration
├── packages.yml             # DBT packages dependencies
├── data/                    # Seed files (CSV data)
│   ├── affiliates_extended.csv
│   ├── players_extended.csv
│   └── transactions_extended.csv
├── models/
│   ├── schema.yml           # Model documentation and tests
│   ├── model_1_daily_player_transactions.sql
│   ├── model_2_kyc_discord_deposits.sql
│   └── model_3_top_three_deposits.sql
└── tests/                   # Custom data quality tests
    ├── test_withdrawals_are_negative.sql
    ├── test_deposits_are_positive.sql
    ├── test_deposit_order.sql
    └── test_kyc_discord_filtering.sql
```
