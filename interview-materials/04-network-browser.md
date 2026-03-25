# 网络与浏览器

> 面试频率：⭐⭐⭐⭐⭐

---

## 一、HTTP 协议

### 1.1 HTTP 1.0 / 1.1 / 2.0 / 3.0 区别

| 版本 | 关键特性 |
|------|----------|
| **HTTP 1.0** | 短连接（每个请求一个 TCP） |
| **HTTP 1.1** | 持久连接 (keep-alive)、管道化 (pipelining)、缓存 |
| **HTTP 2.0** | 多路复用、header 压缩、Server Push、二进制分帧 |
| **HTTP 3.0** | QUIC 协议（UDP）、0-RTT、连接迁移 |

### 1.2 多路复用 vs 管线化

```
HTTP 1.1 管线化（问题：队头阻塞）：
GET /a → GET /b → GET /c → 响应1 → 响应2 → 响应3

HTTP 2.0 多路复用（并行）：
┌─ GET /a ──────────────────→ 响应A
├─ GET /b ─────────────→ 响应B
└─ GET /c ─→ 响应C
```

### 1.3 GET vs POST

| 特性 | GET | POST |
|------|-----|------|
| 用途 | 获取资源 | 创建资源 |
| 参数位置 | URL query | 请求体 |
| 缓存 | 可缓存 | 不可缓存 |
| 长度限制 | URL 长度（浏览器约 2KB） | 无限制 |
| 幂等 | ✅ 幂等 | ❌ 非幂等 |
| 安全性 | ⚠️ 参数暴露 | 相对安全 |

### 1.4 RESTful 规范

| 方法 | 含义 | 幂等 |
|------|------|------|
| GET | 获取资源 | ✅ |
| POST | 创建资源 | ❌ |
| PUT | 完整替换 | ✅ |
| PATCH | 部分更新 | ❌ |
| DELETE | 删除资源 | ✅ |

---

## 二、状态码

| 分类 | 范围 | 含义 |
|------|------|------|
| 1xx | 100-199 | 信息性 |
| 2xx | 200-299 | 成功 |
| 3xx | 300-399 | 重定向 |
| 4xx | 400-499 | 客户端错误 |
| 5xx | 500-599 | 服务器错误 |

### 常见状态码

| 状态码 | 含义 |
|--------|------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 301 | 永久重定向 |
| 302 | 临时重定向 |
| 304 | Not Modified（缓存） |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

---

## 三、Cookie / Session / Token

### 3.1 认证流程对比

**Cookie-Session**：
```
1. 用户登录 → 服务器创建 Session，sessionId 存入 Cookie
2. 后续请求 → Cookie 携带 sessionId → 服务器验证
3. 服务器从 Session 存储中获取用户信息
```

**Token (JWT)**：
```
1. 用户登录 → 服务器验证，返回 Token（包含用户信息）
2. 后续请求 → Header 携带 Token (Authorization: Bearer xxx)
3. 服务器验证 Token 签名（无需存储）
```

### 3.2 JWT 结构

```
Header.Payload.Signature

Header: {"alg": "HS256", "typ": "JWT"}
Payload: {"sub": "123", "exp": 1699999999}
Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### 3.3 对比

| 特性 | Cookie-Session | Token (JWT) |
|------|----------------|-------------|
| 存储 | 服务器 Session | 客户端 |
| 状态 | 有状态 | 无状态 |
| 扩展性 | 跨域难（需 sticky session） | 跨域易 |
| 安全性 | CSRF 攻击 | XSS 攻击 |
| 性能 | 需查表 | 验证签名即可 |

### 3.4 Token 过期处理

- **简单**：过期后跳转登录
- **Refresh Token**：access_token 短过期，refresh_token 换新 token

---

## 四、跨域解决方案

### 4.1 同源策略

> 协议 + 域名 + 端口 相同

### 4.2 解决方案

| 方案 | 原理 | 适用场景 |
|------|------|----------|
| **CORS** | 服务端设置 Access-Control-Allow-Origin | 前后端可控 |
| **JSONP** | 动态创建 script 标签 | GET 请求（老项目） |
| **postMessage** | 窗口间通信 | 多 iframe/窗口 |
| **nginx 反向代理** | 同域转发 | 生产环境 |
| **WebSocket** | 全双工通信 | 实时通信 |

### 4.3 CORS 详解

```js
// 简单请求
Access-Control-Allow-Origin: * 或具体域名

// 预检请求（复杂请求）
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400  // 预检结果缓存时间
```

---

## 五、浏览器缓存

### 5.1 缓存位置

```
请求 → 1. Service Worker (可编程)
     → 2. Memory Cache (内存)
     → 3. Disk Cache (磁盘)
     → 4. 网络请求
```

### 5.2 强缓存 vs 协商缓存

| 类型 | 原理 | 响应头 |
|------|------|--------|
| **强缓存** | 不请求，直接用 | Expires / Cache-Control |
| **协商缓存** | 询问服务器，能否用缓存 | Last-Modified / ETag |

### 5.3 缓存头详解

```http
# 强缓存
Cache-Control: max-age=3600        # 3600秒内有效
Cache-Control: no-cache            # 需协商缓存
Cache-Control: no-store            # 不缓存

# 协商缓存
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
If-Modified-Since: Wed, 21 Oct 2024 07:28:00 GMT

ETag: "abc123"
If-None-Match: "abc123"
```

### 5.4 缓存流程图

```
请求资源
    ↓
检查 Cache-Control
    ↓
┌─ 命中 (max-age 内) ──→ 直接用缓存
└─ 未命中
    ↓
发送请求，携带 If-Modified-Since / If-None-Match
    ↓
┌─ 304 Not Modified ──→ 使用缓存
└─ 200 OK ──────────────→ 下载新资源，更新缓存
```

---

## 六、DNS 解析

### 6.1 解析过程

```
1. 浏览器缓存
2. 本地 hosts 文件
3. 本地 DNS 解析器缓存
4. 根 DNS 服务器 (.com)
5. 顶级 DNS 服务器 (.example.com)
6. 权威 DNS 服务器 (example.com)
```

### 6.2 DNS 预解析

```html
<link rel="dns-prefetch" href="//static.example.com">
```

---

## 七、TCP 三次握手 / 四次挥手

### 7.1 三次握手

```
客户端                    服务器
  │                        │
  │──── SYN=1, seq=x ─────→│  客户端发送 SYN
  │                        │
  │←─ SYN=1, ACK=1, seq=y ─│  服务器确认并发送 SYN
  │    ack=x+1              │
  │                        │
  │──── ACK=1, seq=x+1 ───→│  客户端确认
  │      ack=y+1            │
  │                        │
         建立连接
```

**目的**：确认双方收发能力正常

### 7.2 四次挥手

```
客户端                    服务器
  │                        │
  │──── FIN=1, seq=u ─────→│  客户端要断开
  │                        │
  │←──── ACK=1, ack=u+1 ───│  服务器确认
  │    (可以继续发送数据)   │
  │                        │
  │←──── FIN=1, seq=v ─────│  服务器也要断开
  │                        │
  │──── ACK=1, ack=v+1 ───→│  客户端确认
  │                        │
         等待 2MSL 后关闭
```

---

## 八、HTTPS 原理

### 8.1 TLS 握手过程

```
1. Client Hello → 服务器支持的加密算法
2. Server Hello → 选择算法 + 证书
3. 客户端验证证书（CA 签名）
4. 客户端生成随机数，用公钥加密
5. 服务器用私钥解密
6. 双方用随机数生成对称密钥
7. 后续通信使用对称加密
```

### 8.2 证书验证

```
浏览器内置 CA 公钥
    ↓
验证服务器证书签名
    ↓
检查证书有效期、域名匹配
    ↓
建立安全连接
```

---

## 九、安全

### 9.1 XSS (跨站脚本)

| 类型 | 原理 |
|------|------|
| 存储型 | 恶意脚本存入数据库 |
| 反射型 | URL 参数注入 |
| DOM 型 | 前端代码注入 |

**防御**：
- 转义 HTML：`&lt;` `<`
- CSP 内容安全策略
- HttpOnly Cookie

### 9.2 CSRF (跨站请求伪造)

> 用户登录 A 网站，访问恶意网站 B，B 自动发起对 A 的请求

**防御**：
- CSRF Token
- 双重 Cookie 验证
- SameSite Cookie

---

> 📌 下一章：[手写代码题](./05-coding-handwrite.md)
