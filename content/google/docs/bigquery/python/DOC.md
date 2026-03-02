---
name: bigquery
description: "Google Cloud BigQuery Python client library for data warehouse queries and analytics"
metadata:
  languages: "python"
  versions: "3.38.0"
  updated-on: "2026-03-02"
  source: maintainer
  tags: "google,bigquery,data-warehouse,sql,analytics"
---

# Google Cloud BigQuery - Python Client Library

## Golden Rule

**ALWAYS use `google-cloud-bigquery` version 3.38.0 or higher.**

This is the official, maintained Google Cloud client library for BigQuery. Do NOT use deprecated packages like `bigquery` (without google-cloud prefix) or any unofficial libraries.

**Installation:**
```bash
pip install google-cloud-bigquery
```

**IMPORTANT:** BigQuery does NOT support API keys for authentication. You MUST use OAuth 2.0 credentials via service accounts or Application Default Credentials (ADC).

## Installation

### Install the Package

```bash
pip install google-cloud-bigquery
```

### Install with Optional Dependencies

```bash
# With pandas support
pip install google-cloud-bigquery[pandas]

# With all optional dependencies
pip install google-cloud-bigquery[all]
```

### Authentication Setup

BigQuery uses Application Default Credentials (ADC). Set the environment variable to point to your service account key:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
```

### Environment Variables

Create a `.env` file:

```bash
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account-key.json
PROJECT_ID=your-gcp-project-id
DATASET_ID=your_dataset_id
```

Load environment variables in your application:

```python
import os
from dotenv import load_dotenv

load_dotenv()

project_id = os.getenv('PROJECT_ID')
dataset_id = os.getenv('DATASET_ID')
```

## Initialization

### Basic Client Initialization

```python
from google.cloud import bigquery

# Automatically uses GOOGLE_APPLICATION_CREDENTIALS
client = bigquery.Client()
```

### Explicit Configuration

```python
from google.cloud import bigquery

client = bigquery.Client(
    project='your-gcp-project-id',
    credentials=credentials
)
```

### Using Service Account Key File

```python
from google.cloud import bigquery

client = bigquery.Client.from_service_account_json(
    '/path/to/service-account-key.json'
)
```

### With Project ID

```python
from google.cloud import bigquery

client = bigquery.Client(project='your-gcp-project-id')
```

### With Location

```python
from google.cloud import bigquery

client = bigquery.Client(
    project='your-gcp-project-id',
    location='US'  # or 'EU', 'asia-northeast1', etc.
)
```

## Querying Data

### Basic Query

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state, year, number
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
    LIMIT 100
"""

query_job = client.query(query)
results = query_job.result()

for row in results:
    print(f"{row.name}: {row.number}")
```

### Query with query_and_wait()

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, SUM(number) as total
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
    GROUP BY name
    ORDER BY total DESC
    LIMIT 10
"""

rows = client.query_and_wait(query)

for row in rows:
    print(f"{row.name}: {row.total}")
```

### Query to DataFrame

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state, year, number
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
    LIMIT 1000
"""

df = client.query(query).to_dataframe()
print(df.head())
```

### Parameterized Queries

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state, year, number
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state IN UNNEST(@states)
      AND year >= @year
    ORDER BY number DESC
    LIMIT @limit
"""

job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ArrayQueryParameter("states", "STRING", ["WA", "WI", "WV", "WY"]),
        bigquery.ScalarQueryParameter("year", "INT64", 2000),
        bigquery.ScalarQueryParameter("limit", "INT64", 20),
    ]
)

query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(row)
```

### Query with Struct Parameters

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT @struct_value.name as name, @struct_value.age as age
"""

job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.StructQueryParameter(
            "struct_value",
            bigquery.ScalarQueryParameter("name", "STRING", "Tom"),
            bigquery.ScalarQueryParameter("age", "INT64", 30),
        )
    ]
)

query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(f"{row.name}: {row.age}")
```

### Query with Array of Structs

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT country.name, country.capital_city
    FROM UNNEST(@countries) as country
"""

job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ArrayQueryParameter(
            "countries",
            bigquery.StructQueryParameterType(
                bigquery.ScalarQueryParameterType("name", "STRING"),
                bigquery.ScalarQueryParameterType("capital_city", "STRING"),
            ),
            [
                {"name": "France", "capital_city": "Paris"},
                {"name": "Germany", "capital_city": "Berlin"},
            ]
        )
    ]
)

query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(f"{row.name}: {row.capital_city}")
```

### Query with Timestamp Parameters

```python
from google.cloud import bigquery
from datetime import datetime

client = bigquery.Client()

query = """
    SELECT @timestamp_value as timestamp_col
"""

job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ScalarQueryParameter(
            "timestamp_value",
            "TIMESTAMP",
            datetime(2024, 1, 1, 0, 0, 0)
        )
    ]
)

query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(row.timestamp_col)
```

### Dry Run Query (Check Bytes Processed)

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
"""

job_config = bigquery.QueryJobConfig(dry_run=True, use_query_cache=False)
query_job = client.query(query, job_config=job_config)

print(f"This query will process {query_job.total_bytes_processed} bytes.")
```

### Query Job with Manual Job Control

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, COUNT(*) as count
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    GROUP BY name
    ORDER BY count DESC
    LIMIT 10
"""

query_job = client.query(query)
print(f"Job {query_job.job_id} started.")

results = query_job.result()

print("Rows:")
for row in results:
    print(row)
```

### Polling Query Job Status

```python
from google.cloud import bigquery
import time

client = bigquery.Client()

query = "SELECT 1 as value"
query_job = client.query(query)

print(f"Job {query_job.job_id} started.")

while query_job.state != 'DONE':
    time.sleep(1)
    query_job.reload()
    print(f"Job state: {query_job.state}")

print(f"Job completed. Processed {query_job.total_bytes_processed} bytes.")
```

## Datasets

### Create Dataset

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_new_dataset"
dataset = bigquery.Dataset(dataset_id)
dataset = client.create_dataset(dataset)

print(f"Created dataset {dataset.dataset_id}")
```

### Create Dataset with Options

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_new_dataset"
dataset = bigquery.Dataset(dataset_id)

dataset.location = "US"
dataset.description = "My dataset description"
dataset.default_table_expiration_ms = 3600000  # 1 hour

dataset = client.create_dataset(dataset)
print(f"Created dataset {dataset.dataset_id}")
```

### Get Dataset

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_dataset"
dataset = client.get_dataset(dataset_id)

print(f"Dataset {dataset.dataset_id}")
print(f"Description: {dataset.description}")
print(f"Location: {dataset.location}")
```

### List Datasets

```python
from google.cloud import bigquery

client = bigquery.Client()

datasets = list(client.list_datasets())

if datasets:
    print("Datasets:")
    for dataset in datasets:
        print(f"  {dataset.dataset_id}")
else:
    print("No datasets found.")
```

### Update Dataset

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_dataset"
dataset = client.get_dataset(dataset_id)

dataset.description = "Updated description"
dataset = client.update_dataset(dataset, ["description"])

print(f"Updated dataset {dataset.dataset_id}")
```

### Delete Dataset

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_dataset"
client.delete_dataset(dataset_id, delete_contents=True, not_found_ok=True)

print(f"Deleted dataset {dataset_id}")
```

## Tables

### Create Table

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

schema = [
    bigquery.SchemaField("name", "STRING", mode="REQUIRED"),
    bigquery.SchemaField("age", "INTEGER", mode="NULLABLE"),
    bigquery.SchemaField("email", "STRING", mode="REQUIRED"),
    bigquery.SchemaField("created_at", "TIMESTAMP", mode="REQUIRED"),
]

table = bigquery.Table(table_id, schema=schema)
table = client.create_table(table)

print(f"Created table {table.table_id}")
```

### Create Table with Nested Schema

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_nested_table"

schema = [
    bigquery.SchemaField("id", "INTEGER", mode="REQUIRED"),
    bigquery.SchemaField(
        "address",
        "RECORD",
        mode="NULLABLE",
        fields=[
            bigquery.SchemaField("street", "STRING", mode="NULLABLE"),
            bigquery.SchemaField("city", "STRING", mode="NULLABLE"),
            bigquery.SchemaField("zip", "STRING", mode="NULLABLE"),
        ],
    ),
    bigquery.SchemaField("tags", "STRING", mode="REPEATED"),
]

table = bigquery.Table(table_id, schema=schema)
table = client.create_table(table)

print(f"Created table {table.table_id}")
```

### Get Table Metadata

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
table = client.get_table(table_id)

print(f"Table {table.table_id}")
print(f"Schema: {table.schema}")
print(f"Num rows: {table.num_rows}")
print(f"Num bytes: {table.num_bytes}")
```

### List Tables

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_dataset"
tables = list(client.list_tables(dataset_id))

if tables:
    print("Tables:")
    for table in tables:
        print(f"  {table.table_id}")
else:
    print("No tables found.")
```

### Delete Table

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
client.delete_table(table_id, not_found_ok=True)

print(f"Deleted table {table_id}")
```

### Update Table Schema

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
table = client.get_table(table_id)

original_schema = table.schema
new_schema = original_schema[:]
new_schema.append(bigquery.SchemaField("new_field", "STRING", mode="NULLABLE"))

table.schema = new_schema
table = client.update_table(table, ["schema"])

print(f"Updated table {table.table_id}")
```

## Inserting Data

### Streaming Insert (Single Row)

```python
from google.cloud import bigquery
from datetime import datetime

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

rows_to_insert = [
    {"name": "Tom", "age": 30, "email": "tom@example.com", "created_at": datetime.now().isoformat()}
]

errors = client.insert_rows_json(table_id, rows_to_insert)

if errors == []:
    print("New rows have been added.")
else:
    print(f"Encountered errors while inserting rows: {errors}")
```

### Streaming Insert (Multiple Rows)

```python
from google.cloud import bigquery
from datetime import datetime

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

rows_to_insert = [
    {"name": "Tom", "age": 30, "email": "tom@example.com", "created_at": datetime.now().isoformat()},
    {"name": "Jane", "age": 32, "email": "jane@example.com", "created_at": datetime.now().isoformat()},
    {"name": "Bob", "age": 28, "email": "bob@example.com", "created_at": datetime.now().isoformat()},
]

errors = client.insert_rows_json(table_id, rows_to_insert)

if errors == []:
    print(f"{len(rows_to_insert)} rows have been added.")
else:
    print(f"Encountered errors while inserting rows: {errors}")
```

### Streaming Insert with Row Objects

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
table = client.get_table(table_id)

rows_to_insert = [
    ("Tom", 30, "tom@example.com"),
    ("Jane", 32, "jane@example.com"),
]

errors = client.insert_rows(table, rows_to_insert)

if errors == []:
    print("New rows have been added.")
else:
    print(f"Encountered errors while inserting rows: {errors}")
```

### Streaming Insert with Insert IDs (Deduplication)

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

rows_to_insert = [
    {"name": "Tom", "age": 30},
    {"name": "Jane", "age": 32},
]

row_ids = ["unique-id-1", "unique-id-2"]

errors = client.insert_rows_json(table_id, rows_to_insert, row_ids=row_ids)

if errors == []:
    print("New rows have been added with insert IDs.")
else:
    print(f"Encountered errors while inserting rows: {errors}")
```

### Streaming Insert with Nested Data

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_nested_table"

rows_to_insert = [
    {
        "id": 1,
        "address": {
            "street": "123 Main St",
            "city": "Austin",
            "zip": "78701"
        },
        "tags": ["important", "customer"]
    }
]

errors = client.insert_rows_json(table_id, rows_to_insert)

if errors == []:
    print("New rows have been added.")
else:
    print(f"Encountered errors while inserting rows: {errors}")
```

## Loading Data

### Load from Cloud Storage (CSV)

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
uri = "gs://my-bucket/data.csv"

job_config = bigquery.LoadJobConfig(
    schema=[
        bigquery.SchemaField("name", "STRING"),
        bigquery.SchemaField("age", "INTEGER"),
        bigquery.SchemaField("email", "STRING"),
    ],
    skip_leading_rows=1,
    source_format=bigquery.SourceFormat.CSV,
)

load_job = client.load_table_from_uri(uri, table_id, job_config=job_config)
load_job.result()

print(f"Loaded {load_job.output_rows} rows.")
```

### Load from Cloud Storage (JSON)

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
uri = "gs://my-bucket/data.json"

job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,
    autodetect=True,
)

load_job = client.load_table_from_uri(uri, table_id, job_config=job_config)
load_job.result()

print(f"Loaded {load_job.output_rows} rows.")
```

### Load from Local File

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
filename = "./data.csv"

job_config = bigquery.LoadJobConfig(
    schema=[
        bigquery.SchemaField("name", "STRING"),
        bigquery.SchemaField("age", "INTEGER"),
    ],
    skip_leading_rows=1,
    source_format=bigquery.SourceFormat.CSV,
)

with open(filename, "rb") as source_file:
    load_job = client.load_table_from_file(source_file, table_id, job_config=job_config)

load_job.result()
print(f"Loaded {load_job.output_rows} rows.")
```

### Load with Write Disposition

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
uri = "gs://my-bucket/data.csv"

job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.CSV,
    skip_leading_rows=1,
    autodetect=True,
    write_disposition=bigquery.WriteDisposition.WRITE_TRUNCATE,  # WRITE_APPEND, WRITE_EMPTY
)

load_job = client.load_table_from_uri(uri, table_id, job_config=job_config)
load_job.result()

print(f"Loaded {load_job.output_rows} rows.")
```

### Load Parquet from Cloud Storage

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
uri = "gs://my-bucket/data.parquet"

job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.PARQUET,
)

load_job = client.load_table_from_uri(uri, table_id, job_config=job_config)
load_job.result()

print(f"Loaded {load_job.output_rows} rows.")
```

### Load from DataFrame

```python
from google.cloud import bigquery
import pandas as pd

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

df = pd.DataFrame({
    "name": ["Tom", "Jane", "Bob"],
    "age": [30, 32, 28],
    "email": ["tom@example.com", "jane@example.com", "bob@example.com"]
})

job_config = bigquery.LoadJobConfig(
    schema=[
        bigquery.SchemaField("name", "STRING"),
        bigquery.SchemaField("age", "INTEGER"),
        bigquery.SchemaField("email", "STRING"),
    ]
)

load_job = client.load_table_from_dataframe(df, table_id, job_config=job_config)
load_job.result()

print(f"Loaded {load_job.output_rows} rows.")
```

### Load Multiple Files from Cloud Storage

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
uris = [
    "gs://my-bucket/data1.csv",
    "gs://my-bucket/data2.csv",
    "gs://my-bucket/data3.csv",
]

job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.CSV,
    skip_leading_rows=1,
    autodetect=True,
)

load_job = client.load_table_from_uri(uris, table_id, job_config=job_config)
load_job.result()

print(f"Loaded {load_job.output_rows} rows.")
```

## Exporting Data

### Export to Cloud Storage (CSV)

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
destination_uri = "gs://my-bucket/export.csv"

extract_job = client.extract_table(table_id, destination_uri)
extract_job.result()

print(f"Exported {table_id} to {destination_uri}")
```

### Export to Cloud Storage with Options

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
destination_uri = "gs://my-bucket/export-*.csv"

job_config = bigquery.ExtractJobConfig(
    compression=bigquery.Compression.GZIP,
    destination_format=bigquery.DestinationFormat.CSV,
    print_header=True,
)

extract_job = client.extract_table(table_id, destination_uri, job_config=job_config)
extract_job.result()

print(f"Exported {table_id} to {destination_uri}")
```

### Export to Cloud Storage (JSON)

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
destination_uri = "gs://my-bucket/export-*.json"

job_config = bigquery.ExtractJobConfig(
    destination_format=bigquery.DestinationFormat.NEWLINE_DELIMITED_JSON
)

extract_job = client.extract_table(table_id, destination_uri, job_config=job_config)
extract_job.result()

print(f"Exported {table_id} to {destination_uri}")
```

### Export to Cloud Storage (Avro)

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
destination_uri = "gs://my-bucket/export-*.avro"

job_config = bigquery.ExtractJobConfig(
    destination_format=bigquery.DestinationFormat.AVRO
)

extract_job = client.extract_table(table_id, destination_uri, job_config=job_config)
extract_job.result()

print(f"Exported {table_id} to {destination_uri}")
```

### Export Query Results to Cloud Storage

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state, year
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
    LIMIT 1000
"""

destination_table_id = f"{client.project}.my_dataset.temp_table"

job_config = bigquery.QueryJobConfig(
    destination=destination_table_id,
    write_disposition=bigquery.WriteDisposition.WRITE_TRUNCATE,
)

query_job = client.query(query, job_config=job_config)
query_job.result()

destination_uri = "gs://my-bucket/query-results-*.csv"

extract_job = client.extract_table(destination_table_id, destination_uri)
extract_job.result()

print(f"Exported query results to {destination_uri}")
```

## Advanced Querying

### Query with Destination Table

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, SUM(number) as total
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    WHERE state = 'TX'
    GROUP BY name
"""

table_id = f"{client.project}.my_dataset.results_table"

job_config = bigquery.QueryJobConfig(
    destination=table_id,
    write_disposition=bigquery.WriteDisposition.WRITE_TRUNCATE,
)

query_job = client.query(query, job_config=job_config)
results = query_job.result()

print(f"Query results saved to {table_id}. {results.total_rows} rows.")
```

### Query with Caching Disabled

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    LIMIT 10
"""

job_config = bigquery.QueryJobConfig(use_query_cache=False)
query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(row)
```

### Query with Legacy SQL

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state
    FROM [bigquery-public-data:usa_names.usa_1910_2013]
    WHERE state = 'TX'
    LIMIT 10
"""

job_config = bigquery.QueryJobConfig(use_legacy_sql=True)
query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(row)
```

### Query with Maximum Billing Tier

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, COUNT(*) as count
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    GROUP BY name
"""

job_config = bigquery.QueryJobConfig(maximum_billing_tier=1)
query_job = client.query(query, job_config=job_config)
results = query_job.result()

print(f"Processed {query_job.total_bytes_processed} bytes.")
```

### Query with Maximum Bytes Billed

```python
from google.cloud import bigquery

client = bigquery.Client()

query = """
    SELECT name, state
    FROM `bigquery-public-data.usa_names.usa_1910_2013`
    LIMIT 10
"""

job_config = bigquery.QueryJobConfig(maximum_bytes_billed=1000000)
query_job = client.query(query, job_config=job_config)
results = query_job.result()

for row in results:
    print(row)
```

### Create Clustered Table

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_clustered_table"

schema = [
    bigquery.SchemaField("name", "STRING"),
    bigquery.SchemaField("state", "STRING"),
    bigquery.SchemaField("year", "INTEGER"),
    bigquery.SchemaField("number", "INTEGER"),
]

table = bigquery.Table(table_id, schema=schema)
table.clustering_fields = ["state", "year"]

table = client.create_table(table)
print(f"Created clustered table {table.table_id}")
```

### Create Partitioned Table

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_partitioned_table"

schema = [
    bigquery.SchemaField("name", "STRING"),
    bigquery.SchemaField("created_date", "DATE"),
    bigquery.SchemaField("value", "INTEGER"),
]

table = bigquery.Table(table_id, schema=schema)
table.time_partitioning = bigquery.TimePartitioning(
    type_=bigquery.TimePartitioningType.DAY,
    field="created_date",
)

table = client.create_table(table)
print(f"Created partitioned table {table.table_id}")
```

### Create Partitioned and Clustered Table

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_optimized_table"

schema = [
    bigquery.SchemaField("transaction_id", "STRING"),
    bigquery.SchemaField("transaction_date", "DATE"),
    bigquery.SchemaField("customer_id", "STRING"),
    bigquery.SchemaField("amount", "FLOAT"),
]

table = bigquery.Table(table_id, schema=schema)
table.time_partitioning = bigquery.TimePartitioning(
    type_=bigquery.TimePartitioningType.DAY,
    field="transaction_date",
)
table.clustering_fields = ["customer_id"]

table = client.create_table(table)
print(f"Created partitioned and clustered table {table.table_id}")
```

## Jobs

### List Jobs

```python
from google.cloud import bigquery

client = bigquery.Client()

jobs = list(client.list_jobs(max_results=10))

print("Jobs:")
for job in jobs:
    print(f"{job.job_id} - {job.state}")
```

### List Jobs with Filter

```python
from google.cloud import bigquery

client = bigquery.Client()

jobs = list(client.list_jobs(
    max_results=10,
    state_filter="DONE"
))

print("Completed jobs:")
for job in jobs:
    print(f"{job.job_id} - {job.job_type}")
```

### Get Job Details

```python
from google.cloud import bigquery

client = bigquery.Client()

job = client.get_job("my-job-id")

print(f"Job {job.job_id}")
print(f"State: {job.state}")
print(f"Created: {job.created}")
print(f"Started: {job.started}")
print(f"Ended: {job.ended}")

if hasattr(job, 'total_bytes_processed'):
    print(f"Bytes processed: {job.total_bytes_processed}")
```

### Cancel Job

```python
from google.cloud import bigquery

client = bigquery.Client()

job = client.get_job("my-job-id")
job.cancel()

print(f"Job {job.job_id} cancelled.")
```

## Copying Tables

### Copy Table

```python
from google.cloud import bigquery

client = bigquery.Client()

source_table_id = f"{client.project}.my_dataset.source_table"
dest_table_id = f"{client.project}.my_dataset.dest_table"

job = client.copy_table(source_table_id, dest_table_id)
job.result()

print(f"Table copied to {dest_table_id}")
```

### Copy Table with Write Disposition

```python
from google.cloud import bigquery

client = bigquery.Client()

source_table_id = f"{client.project}.my_dataset.source_table"
dest_table_id = f"{client.project}.my_dataset.dest_table"

job_config = bigquery.CopyJobConfig(
    write_disposition=bigquery.WriteDisposition.WRITE_TRUNCATE
)

job = client.copy_table(source_table_id, dest_table_id, job_config=job_config)
job.result()

print(f"Table copied to {dest_table_id}")
```

### Copy Multiple Tables

```python
from google.cloud import bigquery

client = bigquery.Client()

source_tables = [
    f"{client.project}.my_dataset.table1",
    f"{client.project}.my_dataset.table2",
    f"{client.project}.my_dataset.table3",
]
dest_table_id = f"{client.project}.my_dataset.merged_table"

job = client.copy_table(source_tables, dest_table_id)
job.result()

print(f"Tables merged into {dest_table_id}")
```

## Row-Level Operations

### Get Table Rows

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

rows = client.list_rows(table_id, max_results=10)

for row in rows:
    print(row)
```

### Get Rows with Selected Fields

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
table = client.get_table(table_id)

selected_fields = [
    table.schema[0],  # name
    table.schema[1],  # age
]

rows = client.list_rows(table_id, selected_fields=selected_fields, max_results=10)

for row in rows:
    print(row)
```

### Get Rows with Pagination

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

page_size = 100
pages = client.list_rows(table_id, max_results=page_size)

for page in pages.pages:
    for row in page:
        print(row)
```

### Convert Rows to DataFrame

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

rows = client.list_rows(table_id, max_results=1000)
df = rows.to_dataframe()

print(df.head())
```

## IAM and Access Control

### Get Dataset IAM Policy

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_dataset"
dataset = client.get_dataset(dataset_id)

policy = client.get_iam_policy(dataset)

print("IAM Policy:")
for binding in policy.bindings:
    print(f"Role: {binding['role']}")
    print(f"Members: {binding['members']}")
```

### Set Dataset IAM Policy

```python
from google.cloud import bigquery

client = bigquery.Client()

dataset_id = f"{client.project}.my_dataset"
dataset = client.get_dataset(dataset_id)

policy = client.get_iam_policy(dataset)

policy.bindings.append({
    "role": "roles/bigquery.dataViewer",
    "members": {"user:example@example.com"}
})

policy = client.set_iam_policy(dataset, policy)
print("IAM policy updated.")
```

## Error Handling

### Comprehensive Error Handling

```python
from google.cloud import bigquery
from google.api_core import exceptions

client = bigquery.Client()

query = "SELECT * FROM invalid_table"

try:
    query_job = client.query(query)
    results = query_job.result()
except exceptions.NotFound as e:
    print(f"Table not found: {e}")
except exceptions.BadRequest as e:
    print(f"Invalid query: {e}")
except exceptions.Forbidden as e:
    print(f"Permission denied: {e}")
except exceptions.GoogleAPIError as e:
    print(f"API error: {e}")
```

### Handle Streaming Insert Errors

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"

rows_to_insert = [
    {"name": "Tom", "age": 30},
    {"name": "Jane", "age": "invalid"},  # Invalid type
]

errors = client.insert_rows_json(table_id, rows_to_insert)

if errors:
    print("Errors encountered:")
    for error in errors:
        print(f"Row index: {error['index']}")
        print(f"Errors: {error['errors']}")
else:
    print("All rows inserted successfully.")
```

### Handle Load Job Errors

```python
from google.cloud import bigquery

client = bigquery.Client()

table_id = f"{client.project}.my_dataset.my_table"
uri = "gs://my-bucket/data.csv"

job_config = bigquery.LoadJobConfig(
    source_format=bigquery.SourceFormat.CSV,
    skip_leading_rows=1,
    autodetect=True,
)

load_job = client.load_table_from_uri(uri, table_id, job_config=job_config)

try:
    load_job.result()
    print(f"Loaded {load_job.output_rows} rows.")
except Exception as e:
    print(f"Load job failed: {e}")

    if load_job.errors:
        print("Errors:")
        for error in load_job.errors:
            print(error)
```
