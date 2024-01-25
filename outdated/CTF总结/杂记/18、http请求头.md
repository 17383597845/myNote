1. **Remote Address:**  
    - `Remote Address` 是指当前网络连接的远程地址，通常表示客户端的 IP 地址。  
2. **X-Forwarded-For:**  
    - `X-Forwarded-For` 是一种常见的头字段，用于表示经过的代理服务器，其中第一个 IP 地址通常是客户端的原始 IP。  
3. **X-Real-IP:**  
    - `X-Real-IP` 有时包含客户端的真实 IP 地址，但它可以被伪造，不是绝对可信的来源。  
4. **X-Client-IP:**  
    - `X-Client-IP` 也可能包含客户端的真实 IP 地址。  
5. **X-Original-Forwarded-For:**  
    - `X-Original-Forwarded-For` 可能包含原始的 `X-Forwarded-For` 头字段信息，用于跟踪经过的代理。  
6. **X-Requester:**  
    - `X-Requester` 是自定义头字段，有时用于表示请求的发起者，可能包含 IP 地址信息。  
7. **Forwarded:**  
    - `Forwarded` 头字段提供了一个标准的方法来包含途中经过的代理信息，包括客户端 IP 地址。  
8. **Via:**  
    - `Via` 头字段用于标识请求经过的代理服务器，可以包含 IP 地址信息。  
9. **Client-Ip:**  
- `Client-Ip` 头字段也可能包含客户端的 IP 地址信息。  
1. **X-Originating-IP:**  
    - `X-Originating-IP` 有时用于表示请求的初始来源 IP。  
2. **X-Host:**  
    - `X-Host` 头字段用于标识主机信息，包括 IP 地址。  
3. **X-My-Forwarded-For:**  
    - `X-My-Forwarded-For` 是自定义头字段，有时被用于表示经过的代理的 IP 地址。  
4. **X-Forwarded:**  
    - `X-Forwarded` 是一种标准的头字段，包含了代理信息，包括 `for`、`proto` 和 `host` 等。  
5. **X-Client-IP:**  
    - `X-Client-IP` 可能包含客户端的真实 IP 地址。

```
Client-IP: 127.0.0.1
Forwarded-For-Ip: 127.0.0.1
Forwarded-For: 127.0.0.1
Forwarded-For: localhost
Forwarded: 127.0.0.1
Forwarded: localhost
True-Client-IP: 127.0.0.1
X-Client-IP: 127.0.0.1
X-Custom-IP-Authorization: 127.0.0.1
X-Forward-For: 127.0.0.1
X-Forward: 127.0.0.1
X-Forward: localhost
X-Forwarded-By: 127.0.0.1
X-Forwarded-By : localhost
X-Forwarded-For-original: 127.0.0.1
X-Forwarded-For-original: localhost
X-Forwarded-For: 127.0.0.1
X-Forwarded-For: localhost
X-Forwarded-Server: 127.0.0.1
X-Forwarded-server: localhost
X-Forwarded: 127.0.0.1
X-Forwarded: localhost
X-Forwarei-Host: 127.0.0.1
X-Forwared-Host: localhost
X-Host: 127.0.0.1
X-Host: localhost
X-HTTP-Host-override: 127.0.0.1
X-originating-IP: 127.0.0 .1
X-Real-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Remote-Addr: localhost
X-Remote-IP: 127.0.0.1

```


## 指定源域名

Referer: www.eeeee.com


## 代理
via:Clash.win



