# ClickHouse Driver (TCP)

## Overview
| **Description**                    | **Details**                                                                  |
|------------------------------------|------------------------------------------------------------------------------|
| **Github Repository**              | [GitHub - clickhouse-driver](https://github.com/mymarilyn/clickhouse-driver) |
| **ClickHouse Documentation Page**  | [ClickHouse Docs](https://clickhouse-driver.readthedocs.io/en/latest/)       |
| **Base ClickHouse Driver Version** | [0.2.9](https://pypi.org/project/clickhouse-driver/0.2.9/)                   |
| **Verified Python Version**        | Python 3.8                                                                   |

## Installation
`gcc`, `python` and `linux` headers are required to build the driver. Example for `python:alpine` docker image:
```bash
apk add gcc musl-dev
```
The package can be installed via `pip`:
```bash
pip install clickhouse-driver
```

## Client Construction
### 1. Passing parameters
```python
from clickhouse_driver import Client

client = Client(
    host="gateway-v2.bytehouse-cn-{REGION}.volces.com",
    port=19000,
    user="bytehouse",
    password="{API_KEY}",
    database="{DATABASE}",
    secure=True,
)
```
### 2. Passing connection string
```python
from clickhouse_driver import Client

uri = "clickhouse://bytehouse:{API_KEY}@gateway-v2.bytehouse-cn-{REGION}.volces.com:19000/{DATABASE}?secure=True"
client = Client.from_url(uri)
```

## Quickstart
```python
from clickhouse_driver import Client
from datetime import datetime

# Connect to ByteHouse
client = Client(
    host="gateway-v2.bytehouse-cn-{REGION}.volces.com",
    port=19000,
    user="bytehouse",
    password="{API_KEY}",
    database="{DATABASE}",
    secure=True,
)

# Create Table
create_table_query = """
CREATE TABLE IF NOT EXISTS example_table (
    id Int32,
    name String,
    created_at DateTime
)
ENGINE = CnchMergeTree()
ORDER BY id;
"""
client.execute(create_table_query)

# Insert Data
data = [
    (1, 'Alice', datetime(2024, 11, 20, 10, 30, 0)),
    (2, 'Bob', datetime(2024, 11, 20, 11, 0, 0)),
    (3, 'Charlie', datetime(2024, 11, 20, 12, 15, 0)),
]
client.execute("INSERT INTO example_table (id, name, created_at) VALUES", data)

# Query Data
rows = client.execute("SELECT * FROM example_table")
for row in rows:
    print(row)
```
## Data Types
| Type Name                        | Insert Type                                | Select Type                     |
|----------------------------------|--------------------------------------------|---------------------------------|
| [U]Int8/16/32/64/128/256         | int, long                                  | int                             |
| Float32/64                       | float, int, long                           | float                           |
| Date/Date32                      | date, datetime                             | date                            |
| DateTime                         | datetime, int, long                        | datetime                        |
| String/FixedString(N)            | str, bytes                                 | str, bytes                      |
| Enum8/16                         | Enum, int, long, str                       | str                             |
| Array(T)                         | list, tuple                                | list                            |
| Bool                             | bool                                       | bool                            |
| UUID                             | str, UUID                                  | str                             |
| Decimal                          | Decimal, float, int, long                  | Decimal                         |
| IPv4/IPv6                        | IPv4Address/IPv6Address, int, long, str    | IPv4Address/IPv6Address         |

## Specifying Server Settings
if setting is supported by `ByteHouse` server (e.g. `virtual_warehouse`) and missed in the open source driver package, it 
can be added by monkey patching. Serialization type of the setting value needs to be provided.
```python
from clickhouse_driver import Client
from clickhouse_driver.settings.available import settings as available_settings, SettingString

available_settings['virtual_warehouse'] = SettingString

client = Client(
    host="gateway-v2.bytehouse-cn-{REGION}.volces.com",
    port=19000,
    user="bytehouse",
    password="{API_KEY}",
    database="{DATABASE}",
    secure=True,
    settings={"virtual_warehouse": "vw-123456789-vw-name"},
)
```
## Specifying Query ID
```python
from clickhouse_driver import Client

# client = Client(...) # Initialize client

query_id = "bbd7dea3-eb63-4a21-b727-f55b420a7223"
client.execute("SELECT 1", query_id=query_id)
```
## Inserting data from Pandas dataframe
`'use_numpy': True` setting is required for `Numpy` / `Pandas` package support.
```python
from clickhouse_driver import Client
import pandas as pd

client = Client(
    host="gateway-v2.bytehouse-cn-{REGION}.volces.com",
    port=19000,
    user="bytehouse",
    password="{API_KEY}",
    database="{DATABASE}",
    secure=True,
    settings={"use_numpy": True},
)

client.execute(
   'CREATE TABLE test_df (x Int, y String) Engine = CnchMergeTree() ORDER BY tuple()'
)

df_insert = pd.DataFrame({
    'x': [1, 2],
    'y': ['apple', 'banana'],
}, dtype=object)

client.insert_dataframe('INSERT INTO test_df VALUES', df_insert)

df_select = client.query_dataframe('SELECT * FROM test_df')

print(df_select)

#    x       y
# 0  1   apple
# 1  2  banana
```

## Inserting data from CSV file
Let’s assume you have the following data in `CSV` file:
```csv
time,order,qty
2019-08-01 15:23:14,New order1,5
2019-08-05 09:14:45,New order2,3
2019-08-13 12:20:32,New order3,7
```
Data can be inserted into `ByteHouse` in the following way:
```python
from csv import DictReader
from datetime import datetime

from clickhouse_driver import Client

# client = Client(...) # Initialize client

def iter_csv(filename):
    converters = {
        'time': lambda x: datetime.strptime(x, '%Y-%m-%d %H:%M:%S'),
        'order': str,
        'qty': int,
    }
    with open(filename, 'r') as f:
        reader = DictReader(f)
        for line in reader:
            yield {k: (converters[k](v) if k in converters else v) for k, v in line.items()}


client.execute(
    'CREATE TABLE IF NOT EXISTS data_csv (time DateTime, order String, qty Int32) Engine = CnchMergeTree() ORDER BY tuple()'
)
client.execute('INSERT INTO data_csv VALUES', iter_csv('data.csv'))
```
## Retrieve data based on Column Names
`'with_column_types' = True` is required to retrieve data by column names.
```python
from clickhouse_driver import Client

# client = Client(...) # Initialize client

client.execute("""
CREATE TABLE IF NOT EXISTS test_table (
    int_column Int32,
    string_column String,
    array_column Array(Int32)
) ENGINE = CnchMergeTree()
ORDER BY int_column
""")

data_to_insert = [
    (1, 'First', [1, 2, 3]),
    (2, 'Second', [4, 5, 6]),
    (3, 'Third', [7, 8, 9])
]

client.execute(
    "INSERT INTO test_table (int_column, string_column, array_column) VALUES",
    data_to_insert
)

query = "SELECT * FROM test_table"
data, columns = client.execute(query, with_column_types=True)

# Use dictionary comprehension to create a dictionary with column names as keys
column_data = {col_name: [row[i] for row in data] for i, col_name in enumerate([col[0] for col in columns])}

# Access by column names
print("int_column:", column_data['int_column'])
print("string_column:", column_data['string_column'])
print("array_column:", column_data['array_column'])
```