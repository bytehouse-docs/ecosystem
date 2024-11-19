# Troubleshooting Guide for Python Clients

## 1. clickhouse_driver.errors.NetworkError: Code: 210. Connection reset by peer
By default, `clickhouse-driver` uses non-secure `TCP` connection, which would throw this error. We need to add 
`secure=True` to the client constructor to use secure connection.

```python
client = Client(
    ### other parameters
    secure=True,
)
```
Some other error messages that might indicate the same issue:
- `clickhouse_driver.errors.NetworkError: Code: 210. Connection reset by peer`
- `Error on socket shutdown: [Errno 57] Socket is not connected`
- `ConnectionResetError: [Errno 54] Connection reset by peer`

## 2. clickhouse_driver.errors.UnexpectedPacketFromServerError: Code: 102. Unexpected packet from server
This error is thrown when wrong port number is used in the client constructor. For TCP, the port is `19000` and for 
HTTP, the port is `8123`.

Some other error messages that might indicate the same issue:
- `clickhouse_driver.errors.UnexpectedPacketFromServerError: Code: 102. Unexpected packet from server`
- `expected Hello or Exception, got Unknown packet`

## 3. receive pw openapiauth hello error: fullUsername should be in the format 'accountID::fullUsername' OR 'accountName::fullUsername'
This error is thrown when the username is not correct. For API key authentication, the username should always be `bytehouse`.