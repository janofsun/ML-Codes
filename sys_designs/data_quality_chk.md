### Data Quality Check System

#### 1. Key Objectives for Data Quality System
The system should:
- Ensure data consistency and correctness across different sources.
- Detect missing, incomplete, or inaccurate data.
- Automate the process of data validation and reporting.
- Provide actionable insights when issues are detected.

#### 2. Components of the System
<span style="background-color:rgb(212, 104, 138,0.5)">a. Data Ingestion Layer</span>
- This layer handles the input of data from various sources (e.g., databases, files, APIs).
- At this stage, basic schema checks (e.g., data type validation, column existence) should be performed.

<span style="background-color:rgb(212, 104, 138,0.5)">b. Data Validation Rules</span>
- Schema Checks: Verify that incoming data adheres to the defined schema (e.g., correct data types, required columns).
- Null/Empty Checks: Ensure required fields are not empty.
- Range/Threshold Checks: Validate if numerical values fall within expected ranges.
- Uniqueness Checks: Ensure certain fields (like primary keys) are unique.
- Referential Integrity: Validate relationships between tables (e.g., foreign keys).
- Business Rule Checks: Implement custom business logic to ensure the data makes sense within the domain context (e.g., "order date should be before shipping date").

<span style="background-color:rgb(212, 104, 138,0.5)">c. Metrics and Reporting Layer</span>
- Generate reports and metrics such as the number of invalid records, missing data, or outliers.
- Use dashboards for real-time monitoring of data quality.
- Alerts should be triggered when data quality thresholds are breached.

<span style="background-color:rgb(212, 104, 138,0.5)">d. Remediation and Handling Invalid Data</span>
- Invalid data should be logged and flagged for manual or automated correction.
- Provide options for automatically fixing known issues (e.g., filling missing values with defaults, trimming whitespaces).
- Invalid rows can be sent to a "quarantine" area for review and further analysis.

<span style="background-color:rgb(212, 104, 138,0.5)">e. Integration with Data Pipelines</span>
- The system should integrate with ETL (Extract, Transform, Load) or ELT pipelines to ensure data quality is checked at every step.
- Implement checkpoints in the data pipeline to ensure any invalid data does not proceed downstream without correction.

<span style="background-color:rgb(212, 104, 138,0.5)">f. Audit and Logging</span>
- Every data quality check should be logged for audit purposes, providing traceability of changes, detected issues, and corrective actions.
- Maintain a history of changes in data quality over time to spot trends or recurring issues.

#### 3. System Architecture Diagram

+------------------+    +------------------+    +--------------------+    +-------------------+
|   Data Sources    | -> | Data Ingestion    | -> | Data Quality Checks | -> | Data Warehouse /   |
|  (Databases, APIs,|    |   (ETL/ELT)      |    |   (Schema, Null,    |    | Data Lake / BI     |
|  Files, etc.)     |    |                  |    |   Range, Business)  |    | Tool               |
+------------------+    +------------------+    +--------------------+    +-------------------+
                                                   |        |
                                                   V        V
                                             +--------------------+
                                             |  Invalid Data Logs  |
                                             |    & Correction     |
                                             +--------------------+


#### 4. Data Quality Check Template

Step 1: Define Data Schema

```json
    {
    "columns": [
        {
        "name": "user_id",
        "type": "integer",
        "nullable": false,
        "unique": true
        },
        {
        "name": "order_date",
        "type": "datetime",
        "nullable": false
        },
        {
        "name": "order_amount",
        "type": "float",
        "nullable": false,
        "range": {
            "min": 0.01,
            "max": 10000
        }
        }
    ]
    }

```

Step 2: Data Quality Check Logic (Python Example)

```python
import pandas as pd

# Load data (example)
data = pd.read_csv('data.csv')

# Define data quality rules
data_quality_rules = {
    'user_id': {'type': 'int', 'nullable': False, 'unique': True},
    'order_date': {'type': 'datetime', 'nullable': False},
    'order_amount': {'type': 'float', 'nullable': False, 'min': 0.01, 'max': 10000}
}

# Function to perform data quality checks
def data_quality_check(df, rules):
    errors = []
    
    # Check each column
    for col, rule in rules.items():
        if rule['nullable'] == False:
            null_count = df[col].isnull().sum()
            if null_count > 0:
                errors.append(f"{col} has {null_count} null values")
        
        if 'type' in rule:
            if not pd.api.types.is_numeric_dtype(df[col]) and rule['type'] == 'int':
                errors.append(f"{col} is not of type {rule['type']}")
        
        if 'min' in rule and 'max' in rule:
            invalid_range = df[(df[col] < rule['min']) | (df[col] > rule['max'])].shape[0]
            if invalid_range > 0:
                errors.append(f"{col} has {invalid_range} values out of range")

        if 'unique' in rule and rule['unique']:
            if df[col].duplicated().any():
                errors.append(f"{col} contains duplicates")
    
    return errors

# Perform the data quality checks
errors = data_quality_check(data, data_quality_rules)
if errors:
    for error in errors:
        print(f"Data Quality Error: {error}")
else:
    print("All checks passed.")


```

Step 3: Handling Invalid Data

```python
# Log invalid data for correction
invalid_data = data[data['order_amount'] > 10000]
invalid_data.to_csv('invalid_data_log.csv')

# Optional: Automatically fix certain issues
# e.g., fill missing values with defaults or drop invalid rows
data['order_amount'].fillna(0, inplace=True)
data.dropna(inplace=True)  # Drop rows with missing data

```

Step 4: Integrating with Data Pipeline
In an ETL pipeline (e.g., Airflow, Spark), these checks can be integrated as a part of the pipeline with checkpoints for validation, logging, and alerting.

#### 5. Monitoring and Alerts
Use tools like Prometheus, Grafana, or Airflow to set up alerting mechanisms for failed data quality checks. If the error rate exceeds a threshold, send alerts via email or Slack.

#### 6. Dashboard and Reporting
Create a dashboard that tracks data quality metrics over time, including:
Percentage of missing values.
Number of out-of-range values.
Number of invalid or quarantined rows.