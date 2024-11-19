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

### Installation with compression support
You can install extras packages if you need compression support. Example of `LZ4` compression requirements installation:
```bash
pip install clickhouse-driver[lz4]
```
You also can specify multiple extras by using comma. Install `LZ4` and `ZSTD` requirements:
```bash
pip install clickhouse-driver[lz4,zstd]
```

### Installation with NumPy support
You can install additional packages (`NumPy` and `Pandas`) if you need NumPy support:
```bash
pip install clickhouse-driver[numpy]
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
