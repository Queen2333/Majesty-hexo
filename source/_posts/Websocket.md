---
title: Websocket/HTTP/TCP/IP
featured_image: ./images/wolverine.jpg
---

#### Websocket基础

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

#### Http

##### HTTP 基本概念

HTTP 是什么： 应用层协议，基于 TCP/IP，用于客户端和服务器之间通信。
无状态： 每次请求是独立的，服务器不会主动记住客户端状态。

##### HTTP 的请求方法：

GET：获取资源。
POST：提交数据。
PUT：更新资源。
DELETE：删除资源。
OPTIONS、HEAD、PATCH 等。

##### HTTP 头部：

请求头：Accept、Authorization、Content-Type 等。
响应头：Content-Type、Cache-Control、Set-Cookie 等。
常用的 Content-Type： application/json, text/html, multipart/form-data。

##### 长连接与短连接：

HTTP/1.0 默认短连接，HTTP/1.1 支持长连接，通过 Connection: keep-alive 实现。
HTTP 和 HTTPS 的区别：
HTTPS 是 HTTP + SSL/TLS 加密，数据传输更安全。

##### 浏览器输入一个 url 发生的事情

    1.根据域名到DNS中找到IP
    2.根据IP建立TCP连接(三次握手)
    3.连接建立成功发起http请求
    4.服务器响应http请求
    5.浏览器解析HTML代码并请求html中的静态资源（js,css）
    6.关闭TCP连接（四次挥手）
    7.浏览器渲染页面

##### http 缓存

缓存过程：

浏览器第一次加载资源，服务器返回200，浏览器将资源文件从服务器上请求下载下来，并把response header及该请求的返回时间(要与Cache-Control和Expires对比)一并缓存；

下一次加载资源时，先比较当前时间和上一次返回200时的时间差，如果没有超过Cache-Control设置的max-age，则没有过期，命中强缓存，不发请求直接从本地缓存读取该文件（如果浏览器不支持HTTP1.1，则用Expires判断是否过期）；

如果时间过期，服务器则查看header里的If-None-Match和If-Modified-Since ；

服务器优先根据Etag的值判断被请求的文件有没有做修改，Etag值一致则没有修改，命中协商缓存，返回304；如果不一致则有改动，直接返回新的资源文件带上新的Etag值并返回 200；

如果服务器收到的请求没有Etag值，则将If-Modified-Since和被请求文件的最后修改时间做比对，一致则命中协商缓存，返回304；不一致则返回新的last-modified和文件并返回 200；

使用协商缓存主要是为了进一步降低数据传输量，如果数据没有变，就不必要再传一遍

强缓存

    强缓存主要是采用响应头中的Cache-Control和Expires两个字段进行控制的

    Cache-Control 值理解:
    max-age 指定设置缓存最大的有效时间(单位为s)
    public 指定响应会被缓存，并且在多用户间共享
    private 响应只作为私有的缓存，不能在用户间共享
    no-cache 指定不缓存响应，表明资源不进行缓存
    no-store 绝对禁止缓存

    Expires 理解:
    缓存过期时间，用来指定资源到期的时间，是服务器端的具体的时间点, 需要和Last-modified结合使用

    Last-modified 理解
    服务器端文件的最后修改时间，需要和cache-control共同使用，是检查服务器端资源是否更新的一种方式

    ETag 理解
    根据实体内容生成一段hash字符串，标识资源的状态，由服务端产生。浏览器会将这串字符串传回服务器，验证资源是否已经修改

协商缓存(304)

    协商缓存是指当强缓存机制如果检测到缓存失效，就需要进行服务器再验证

##### HTTP/1.1, HTTP/2, HTTP/3 对比

HTTP/1.1：

支持长连接。
队头阻塞（请求需按顺序完成）。

HTTP/2：

二进制分帧，性能更高。
多路复用（同一连接可并行处理多个请求）。
压缩头部，减少数据传输。

HTTP/3：

基于 QUIC 协议（UDP）。
更快的连接建立，更少的延迟。

Cookie、Session 与 Token

Cookie： 客户端存储数据，用于记录状态。
Session： 服务端存储数据，通常依赖于 Cookie 的 SessionID。
Token： 基于 JWT（JSON Web Token）实现，用户认证后发放，客户端携带 Token 通信。

##### 常见问题总结

1.TCP 为什么需要三次握手？

防止客户端重复的连接请求影响服务端。

2.为什么四次挥手比三次握手多一步？

TCP 是全双工协议，需要双方确认发送和接收的关闭。

3.HTTP 的 OPTIONS 方法有什么用？

用于预检跨域请求，或者查询服务器支持的方法。

4.HTTPS 如何保证安全？

通过 TLS 加密，使用对称加密传输数据，非对称加密交换密钥。

5.HTTP 和 WebSocket 的区别？

HTTP 是一次请求-响应模型，WebSocket 是全双工通信协议。

6.常见的网络错误：

DNS 解析失败、连接超时、拒绝连接。

##### 常见状态码

1. 信息响应 (1xx)：请求已被接收并且处理正在进行中

100 Continue：服务器已经接收到请求头，可以继续发送请求体（如文件上传）。
101 Switching Protocols：服务器已同意切换协议，例如从 HTTP 切换到 WebSocket。

2. 成功响应 (2xx)：客户端请求已成功处理

200 OK：请求成功，服务器返回预期的结果。
201 Created：资源已成功创建（常用于 POST 或 PUT 请求）。
204 No Content：请求成功，但没有内容返回（常用于 DELETE 请求）。

3. 重定向 (3xx)：客户端需要采取进一步的操作才能完成请求

301 Moved Permanently：资源已永久移动到新位置，客户端应使用新 URL。
302 Found：资源临时移动到新位置，客户端仍可使用旧 URL。
304 Not Modified：资源未修改，客户端可使用缓存的版本（常见于条件请求）。

4. 客户端错误 (4xx)：客户端提交的请求有问题

400 Bad Request：请求无效，服务器无法理解请求。
401 Unauthorized：未授权，客户端需要提供身份验证信息。
403 Forbidden：禁止访问，客户端没有权限。
404 Not Found：请求的资源不存在。
405 Method Not Allowed：请求方法不被允许（如用 POST 访问只支持 GET 的资源）。
408 Request Timeout：客户端请求超时，服务器未收到完整请求。
409 Conflict：请求冲突，通常与资源状态相关（如同时更新同一资源）。
413 Payload Too Large：请求体过大，服务器无法处理。
429 Too Many Requests：客户端发送了过多请求（通常用于限流）。

5. 服务器错误 (5xx)：服务器在处理请求时发生错误

500 Internal Server Error：服务器内部错误，无法完成请求。
501 Not Implemented：服务器不支持请求的方法。
502 Bad Gateway：网关或代理服务器接收到无效响应。
503 Service Unavailable：服务器暂时不可用（如过载或维护）。
504 Gateway Timeout：网关或代理服务器未及时收到响应。
505 HTTP Version Not Supported：服务器不支持请求的 HTTP 协议版本。

##### 常见面试问题

1. 401 和 403 的区别是什么？

401：客户端未认证，缺少或提供了无效的凭据。
403：客户端已认证，但没有权限访问资源。

2. 302 和 301 的区别是什么？

301：永久重定向，资源位置已永久更改。
302：临时重定向，资源位置可能会改变。

3. 如何处理 404 错误？

确认 URL 是否正确。
检查资源是否存在。
提供友好的 404 页面。

4. 什么情况下会出现 500 错误？

代码运行时异常。
数据库连接失败。
配置错误。

---

####  TCP/IP 协议

##### TCP/IP 基本概念

##### 分层模型：

应用层（HTTP、SMTP 等）。
传输层（TCP、UDP）。
网络层（IP）。
数据链路层（ARP、MAC 等）。

##### IP 地址与端口：

IP 用于标识设备地址。
端口用于区分同一设备上的不同服务。

##### TCP 常见面试题

三次握手：

1️⃣ 客户端发送 SYN。
2️⃣ 服务器回复 SYN-ACK。
3️⃣ 客户端发送 ACK 确认连接。

四次挥手：

1️⃣ 客户端发送 FIN。
2️⃣ 服务器回复 ACK。
3️⃣ 服务器发送 FIN。
4️⃣ 客户端回复 ACK。

流量控制和拥塞控制：

流量控制：使用窗口机制（TCP sliding window）避免发送方超过接收方处理能力。
拥塞控制：包括慢启动、拥塞避免、快重传、快恢复。

TCP 特点：

可靠：数据有序传输，丢包时重传。
面向连接：需要三次握手建立连接。

UDP（User Datagram Protocol，用户数据报协议） 常见面试题

是面向无连接 的传输层协议，和 TCP 一样基于 IP 协议，但与 TCP 不同，它更加轻量级，适用于对速度要求较高但对可靠性要求不高的场景

特点：

无连接：不需要握手。
不可靠：可能丢包或乱序。
轻量级：没有复杂的握手和确认机制，传输延迟低。适合实时应用（如视频流、DNS）。
数据报传输：UDP 将数据分为一个个数据报，每个数据报是独立的，具有完整性。不会对数据进行分段或重组。
面向消息：UDP 是基于消息的，每个数据报保留了消息边界。



