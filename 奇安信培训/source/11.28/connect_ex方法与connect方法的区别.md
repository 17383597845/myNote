`sock.connect_ex()` 与 `sock.connect()` 都是用于建立TCP连接的方法，但它们在处理连接失败时的返回值上有一些区别。

1. **`sock.connect_ex()`：**
    - 如果连接成功，返回 0。
    - 如果连接失败，返回错误码，而不是引发异常。这使得你可以通过检查返回值来判断连接是否成功，而无需处理异常。
```python
result = sock.connect_ex((target_ip, port))
if result == 0:
    print("连接成功！")
else:
    print("连接失败，错误码:", result)
```
2. **`sock.connect()`：**
    - 如果连接成功，方法返回 `None`。
    - 如果连接失败，会引发 `socket.error` 异常。你需要使用 `try` 和 `except` 语句来捕获异常，并处理连接问题。
```python
try:
    sock.connect((target_ip, port))
    print("连接成功！")
except socket.error as e:
    print("连接失败，错误信息:", e)
```
在实际应用中，你可以根据需要选择使用其中一个方法。如果你更倾向于使用错误码进行处理，可以选择 `sock.connect_ex()`。如果你更倾向于使用异常处理机制，可以选择 `sock.connect()`