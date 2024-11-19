# ClickHouse Connect (HTTP)

## Overview
| **Description**                     | **Details**                                                                     |
|-------------------------------------|---------------------------------------------------------------------------------|
| **Github Repository**               | [GitHub - clickhouse-connect](https://github.com/ClickHouse/clickhouse-connect) |
| **ClickHouse Documentation Page**   | [ClickHouse Docs](https://clickhouse.com/docs/en/integrations/python)           |
| **Base ClickHouse Connect Version** | [0.8.6](https://pypi.org/project/clickhouse-connect/0.8.6/)                     |
| **Verified Python Version**         | Python 3.8                                                                      |

## Installation
The package can be installed via `pip`:
```bash
pip install clickhouse-connect
```
Optionally, you can install `cython` package to build and enable the C/Cython optimizations
```bash
pip install cython
```

## Client Construction
### 1. Passing parameters
```python
import clickhouse_connect

client = clickhouse_connect.get_client(
    host="gateway-v2.bytehouse-cn-{REGION}.volces.com",
    port=8123,
    username="bytehouse",
    password="{API_KEY}",
    database="{DATABASE}",
    secure=True,
)
```
### 2. Passing connection string
TODO