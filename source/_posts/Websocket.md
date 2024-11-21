---
title: Websocket
featured_image: ./images/falcon.jpeg
---

#### 基础

##### 协议概述：

    WebSocket 是一种标准化协议，RFC 6455 定义。
    与 HTTP 一样，基于 TCP 连接，但不同于 HTTP 的请求-响应模型。
    支持客户端和服务端实时双向通信。

##### 工作流程：

    1.握手阶段：WebSocket 使用 HTTP/1.1 协议进行握手，连接成功后升级为 WebSocket 协议。

    2.客户端发送 HTTP 请求头中的 Upgrade: websocket 和 Connection: Upgrade。

    3.数据传输阶段：握手成功后，客户端和服务端使用 WebSocket 协议进行数据帧传输。

---

#### WebSocket 数据帧

##### WebSocket 数据传输单位是帧：

    控制帧：如连接关闭、Ping/Pong。
    数据帧：可传输文本或二进制数据。

##### 数据帧的结构：

    FIN：标志是否是消息的最后一帧。
    Opcode：帧类型（如文本、二进制、Ping、Pong 等）。
    Mask：客户端到服务端的数据必须使用掩码。
    Payload：负载数据，文本通常是 UTF-8 格式。

---

#### 与 HTTP 的关系和区别

##### 相同点

    握手阶段通过 HTTP/1.1 进行。
    都基于 TCP 协议。

##### 区别

    HTTP 是无状态的请求-响应模式，WebSocket 是长连接的全双工通信。
    WebSocket 连接建立后没有 HTTP 请求头的开销，通信效率更高。

---

#### 优势

    实时性：客户端和服务端可随时推送数据，无需轮询。
    节省带宽：与 HTTP 长轮询相比，WebSocket 的头部数据较小，通信更高效。
    低延迟：减少了重复建立连接的时间开销。

---

#### 应用场景

    实时聊天（如 IM 应用、在线客服）。
    实时通知（如交易提醒、社交媒体提醒）。
    实时数据更新（如股票行情、体育比分）。
    在线游戏。
    实时协作（如多人协作编辑文档）。

---

#### 常用操作

##### 握手请求（客户端）

    ```
        GET /chat HTTP/1.1
        Host: example.com
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
        Sec-WebSocket-Version: 13

    ```

##### 握手响应（服务端）

    ```
        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
    ```

##### 发送和接收消息（客户端/服务端）

    使用二进制或文本帧传输消息。
    服务端可以响应客户端的 Ping 消息（发送 Pong）

---

#### WebSocket 的实现

##### 浏览器端

    ```js
        const ws = new WebSocket('ws://example.com/socket');
        ws.onopen = () => console.log('Connected');
        ws.onmessage = (event) => console.log('Message:', event.data);
        ws.onclose = () => console.log('Disconnected');
        ws.onerror = (error) => console.error('Error:', error);
    ```

##### 服务端

    使用语言框架：

    Node.js：ws 库。
    Python：websockets 或 Socket.IO。
    Java：Spring WebSocket

---

#### WebSocket 与安全

##### WSS（WebSocket Secure）

    使用 TLS 加密的 WebSocket，类似 HTTPS。
    URL 以 wss:// 开头。

##### 防范风险

    跨站脚本攻击 (XSS)：验证数据来源。
    跨站 WebSocket 劫持：通过 Origin 或 Token 验证客户端。

---

#### WebSocket 与其他技术的比较

HTTP长轮询                   半双工     数据更新频率低      简单实现
WebSocket	                全双工	    实时性要求高	    高效、低延迟
Server-Sent Events (SSE)	单向推送	服务端频繁推送数据	简单实现，基于 HTTP

---

#### WebSocket 的扩展

##### 子协议（Subprotocols）

    应用层协议在 WebSocket 之上定义，例如 STOMP、MQTT。
    握手时可通过 Sec-WebSocket-Protocol 指定。

##### 负载均衡

    WebSocket 长连接需要 Sticky Session（会话粘滞）。
    通过 Nginx 等代理服务器支持。

##### 连接管理

    心跳检测（Ping/Pong）。
    断线重连策略。

##### 消息序列化

    JSON、Protobuf、MessagePack 等。

#### 调试与监控

    使用浏览器开发者工具检查 WebSocket 消息。
    使用工具如 Wireshark 或 Fiddler 监控数据帧。

---

#### 常见问题

##### 客户端无法与服务端成功建立 WebSocket 连接，可能报错 WebSocket connection failed 或 Error during WebSocket handshake

    1.原因：

    1>服务端未正确配置 WebSocket 协议支持。
    2>防火墙或代理服务器阻止了 WebSocket 协议。
    3>客户端和服务端的 URL 或端口配置错误。

    2.解决方法：

    1>检查服务端代码：确保服务端监听了正确的 WebSocket 请求路径。
    2>确认 URL 和端口：检查客户端使用的 ws:// 或 wss:// 是否正确。
    3>代理或防火墙配置：
        配置代理服务器（如 Nginx）以支持 WebSocket 协议:

        ```nginx
            location /ws/ {
                proxy_pass http://localhost:8080;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
            }
        ```
    4>调试握手过程：使用浏览器开发者工具查看 WebSocket 握手请求和响应。

##### WebSocket 连接意外断开，可能报错 WebSocket is closed with code 1006

    1.原因：

    1>网络不稳定或超时。
    2>服务器因资源限制主动断开连接。
    3>客户端或服务器缺少心跳检测机制。
    
    2.解决方法：

    1>实现心跳检测：
        客户端定期发送 Ping 帧，服务端返回 Pong 帧。

        ```js
            const ws = new WebSocket('ws://example.com/socket');
            let heartbeatInterval;

            ws.onopen = () => {
                heartbeatInterval = setInterval(() => {
                    ws.send(JSON.stringify({ type: 'ping' }));
                }, 30000); // 每 30 秒发送一次心跳
            };

            ws.onclose = () => clearInterval(heartbeatInterval);

        ```

    2>断线重连机制
        当连接断开时尝试重新连接

        ```js
            function connect() {
                const ws = new WebSocket('ws://example.com/socket');
                ws.onclose = () => {
                    console.log('Disconnected. Reconnecting...');
                    setTimeout(connect, 5000); // 5 秒后重连
                };
            }
            connect();
        ```

    3>优化服务端连接管理
        确保服务端没有因超出连接数限制而断开连接。
        使用负载均衡分散连接负担。

##### 客户端或服务端未收到预期的消息

    1.原因：

    1>连接断开时消息未发送或丢失。
    2>消息顺序错乱或未处理。
    
    2.解决方法：

    1>确认消息可靠性：在消息中加入序列号或唯一标识，服务端确认处理成功后再继续。

        客户端示例

        ```js
            let messageCounter = 0; // 序列号
            const ws = new WebSocket('ws://example.com/socket');

            ws.onopen = () => {
                const message = { id: ++messageCounter, content: 'Hello, Server!' };
                ws.send(JSON.stringify(message));
            };

            ws.onmessage = (event) => {
                const data = JSON.parse(event.data);
                console.log('Received:', data);
            };

        ```
        服务端示例 (Node.js)

        ```js
            const WebSocket = require('ws');
            const wss = new WebSocket.Server({ port: 8080 });

            wss.on('connection', (ws) => {
                ws.on('message', (message) => {
                    const data = JSON.parse(message);
                    console.log('Received message with ID:', data.id);
                    ws.send(JSON.stringify({ id: data.id, status: 'received' }));
                });
            });
        ```

    2>补偿机制：当检测到消息丢失时，重新发送丢失的消息。

        客户端示例

        ```js
            let messageQueue = [];
            let messageCounter = 0;

            const ws = new WebSocket('ws://example.com/socket');

            ws.onopen = () => {
                sendWithRetry({ content: 'Hello, Server!' });
            };

            ws.onmessage = (event) => {
                const data = JSON.parse(event.data);
                if (data.status === 'received') {
                    // 确认消息已接收，从队列中移除
                    messageQueue = messageQueue.filter((msg) => msg.id !== data.id);
                }
            };

            function sendWithRetry(message) {
                const msg = { id: ++messageCounter, ...message };
                messageQueue.push(msg);
                ws.send(JSON.stringify(msg));

                // 如果 3 秒未确认，重新发送
                setTimeout(() => {
                    if (messageQueue.find((msg) => msg.id === msg.id)) {
                        console.log('Retrying message:', msg.id);
                        ws.send(JSON.stringify(msg));
                    }
                }, 3000);
            }

        ```

        服务端处理：防止重复消息

        ```js
            const processedMessages = new Set();

            wss.on('connection', (ws) => {
                ws.on('message', (message) => {
                    const data = JSON.parse(message);
                    if (!processedMessages.has(data.id)) {
                        processedMessages.add(data.id);
                        console.log('Processing message:', data);
                        ws.send(JSON.stringify({ id: data.id, status: 'received' }));
                    } else {
                        console.log('Duplicate message ignored:', data.id);
                    }
                });
            });
        ```
    3>队列处理：在客户端和服务端实现消息队列，确保消息按序到达并处理。

        服务端示例 (Redis Pub/Sub)

        ```js
            const WebSocket = require('ws');
            const Redis = require('ioredis');

            const pub = new Redis();
            const sub = new Redis();

            const wss = new WebSocket.Server({ port: 8080 });

            sub.subscribe('chat');
            sub.on('message', (channel, message) => {
                wss.clients.forEach((client) => {
                    if (client.readyState === WebSocket.OPEN) {
                        client.send(message);
                    }
                });
            });

            wss.on('connection', (ws) => {
                ws.on('message', (message) => {
                    pub.publish('chat', message);
                });
            });

        ```

        适配断线重连（防止丢失未发送的消息）

        ```js
            let messageQueue = [];
            let ws;

            function connect() {
                ws = new WebSocket('ws://example.com/socket');

                ws.onopen = () => {
                    console.log('Connected');
                    // 重发未发送的消息
                    while (messageQueue.length > 0) {
                        const msg = messageQueue.shift();
                        ws.send(JSON.stringify(msg));
                    }
                };

                ws.onmessage = (event) => {
                    const data = JSON.parse(event.data);
                    console.log('Received:', data);
                };

                ws.onclose = () => {
                    console.log('Disconnected. Reconnecting...');
                    setTimeout(connect, 5000); // 5 秒后尝试重连
                };
            }

            function sendMessage(message) {
                const msg = { id: ++messageCounter, ...message };
                if (ws.readyState === WebSocket.OPEN) {
                    ws.send(JSON.stringify(msg));
                } else {
                    // 如果连接未建立，将消息存入队列
                    messageQueue.push(msg);
                }
            }

            connect();

        ```

##### 数据可能在传输过程中被窃听或篡改

    1.原因：使用未加密的 WebSocket (ws://) 进行通信。

    2.解决方法：

    1>使用加密连接 (WSS)：配置服务端支持 TLS，并使用 wss://。
    2>验证来源：通过 Origin 头部校验 WebSocket 请求来源。

        ```js
            // Node.js 示例
            const WebSocket = require('ws');
            const wss = new WebSocket.Server({ port: 8080 });

            wss.on('connection', (ws, req) => {
                const origin = req.headers.origin;
                if (origin !== 'https://example.com') {
                    ws.close();
                    console.error('Unauthorized origin:', origin);
                }
            });

        ```
    3>身份验证：在连接握手阶段通过 URL 参数或头部传递 Token 验证身份

        ```js
            const ws = new WebSocket('wss://example.com/socket?token=YOUR_TOKEN');
        ```

##### 多个客户端连接导致服务器性能下降

    1.原因：

        1>单服务器处理的连接数有限。
        2>未释放无效或过期的连接。

    2.解决方法：

        1>连接管理：定期清理长时间未活动的连接。
        2>水平扩展：使用负载均衡器（如 Nginx）分发连接。部署多台 WebSocket 服务实例。
        3>消息广播优化：对需要广播的消息，使用发布/订阅模式或消息中间件（如 Redis Pub/Sub）。

##### 某些老版本浏览器或特殊环境下不支持 WebSocket

    1.原因：

        WebSocket 是较新的协议，旧设备可能不支持。

    2.解决方法：

        1>降级为轮询： 使用 Socket.IO 等库，可以自动降级为 HTTP 长轮询。
        
        ```js
            const io = require('socket.io')(server);
            io.on('connection', (socket) => {
                console.log('Client connected');
                socket.on('message', (msg) => console.log(msg));
            });
        ```
        2>检测支持情况

            在客户端运行环境中检测 WebSocket 支持

            ```js
                if ('WebSocket' in window) {
                    console.log('WebSocket supported');
                } else {
                    console.log('WebSocket not supported');
                }

            ```

---