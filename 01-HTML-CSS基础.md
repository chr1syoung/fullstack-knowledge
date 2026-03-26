---

## 1.7 DOM和BOM的概念

### 题目
什么是DOM和BOM?

#### 知识点详解

**DOM（Document Object Model，文档对象模型）**

DOM 是 W3C 制定的标准，它将 HTML/XML 文档解析为树形结构（DOM 树），每个节点都是一个对象，开发者可以通过 JavaScript 操作这些对象来动态修改页面内容。

```javascript
// DOM 将文档解析为树形结构
document.getElementById('app');      // 通过 ID 获取元素
document.querySelector('.container');  // 通过选择器获取元素
document.createElement('div');         // 创建新元素节点
element.appendChild(child);             // 添加子节点
element.removeChild(child);            // 移除子节点
```

**BOM（Browser Object Model，浏览器对象模型）**

BOM 提供了操作浏览器窗口的 API，没有统一标准（各浏览器实现有差异），核心对象是 `window`。

```javascript
// BOM 核心对象
window;           // 顶级对象，表示浏览器窗口
window.location;  // 当前 URL 信息，可读写
window.navigator; // 浏览器信息（UA、系统等）
window.history;   // 浏览器历史记录
window.screen;    // 屏幕信息（分辨率、颜色深度等）
window.document;  // DOM 的入口，实际上 BOM 的一部分

// 常用 BOM 操作
window.location.href = 'https://example.com';  // 跳转页面
window.history.back();                           // 后退
window.history.forward();                        // 前进
window.navigator.userAgent;                     // 获取 UA 字符串
window.screen.width;                            // 屏幕宽度
window.open('https://example.com', '_blank');  // 打开新窗口
window.close();                                 // 关闭当前窗口
```

**两者关系：**

```
window（顶级对象，BOM 核心）
  ├── document（DOM 入口，包含在 BOM 中）
  ├── location（Navigator...）
  ├── history
  ├── screen
  └── frames[]
```

- BOM 包含 DOM，`window.document` 是 DOM 的入口
- `window` 是 BOM 的顶级对象
- DOM 是 W3C 标准，所有浏览器统一
- BOM 没有统一标准，各浏览器实现存在差异

#### 真实面试题

**题目：什么是DOM和BOM?**

**满分答案：**

DOM（文档对象模型）和BOM（浏览器对象模型）是浏览器提供的两套API，用于操作页面和浏览器本身。

**DOM（文档对象模型）：**
- 由W3C制定的标准化接口，将HTML/XML文档解析为树形结构
- 每个HTML元素、文本、注释都是树上的一个节点
- 开发者可以通过JavaScript操作DOM节点（增删改查）
- 常用API：`getElementById`、`querySelector`、`createElement`、`appendChild`等
- DOM操作会触发浏览器的重排和重绘

**BOM（浏览器对象模型）：**
- 提供操作浏览器窗口的API，没有统一标准（各浏览器实现存在差异）
- 核心对象是`window`，其他重要对象包括：
  - `location`：URL信息（href、protocol、hostname等）
  - `navigator`：浏览器信息（userAgent、platform等）
  - `history`：浏览历史记录（back、forward、pushState等）
  - `screen`：屏幕信息（width、height、colorDepth等）
- 可以控制浏览器行为：弹窗、跳转、前进后退等

**两者的关系：**
- BOM是大概念，DOM是其中的子集
- `window.document`是DOM的入口点
- `window`对象是BOM的顶级对象
- 简单记忆：BOM管浏览器，DOM管文档内容

---

## 1.8 从输入网址到页面显示的过程

### 题目
简单描述从输入网址到页面显示的过程

#### 知识点详解

**完整流程分为7个主要阶段：**

```
1. URL 解析
2. DNS 解析
3. TCP 三次握手
4. HTTP 请求 / TLS 握手（HTTPS）
5. 服务器处理并返回响应
6. 浏览器渲染（关键渲染路径）
7. 绘制与合成
```

**1. URL 解析：**
- 浏览器检查 URL 合法性
- 分解出协议（http/https）、域名、端口、路径、查询参数
- 若为搜索词而非 URL，则使用默认搜索引擎

**2. DNS 解析：**
```
example.com → DNS 服务器查询 → IP 地址（如 93.184.216.34）
```
- 浏览器缓存 → 系统缓存 → 路由器缓存 → ISP DNS 缓存 → 递归查询
- 可能涉及 DNS 预解析：`<link rel="dns-prefetch" href="//static.example.com">`

**3. TCP 三次握手：**
```
客户端 → SYN → 服务器
客户端 ← SYN+ACK ← 服务器
客户端 → ACK → 服务器
（连接建立完成）
```

**4. HTTP 请求：**
- 构建请求行（GET / HTTP/1.1）
- 添加请求头（Host、User-Agent、Cookie 等）
- 若是 HTTPS，还需要 TLS/SSL 握手

**5. 浏览器渲染（关键渲染路径）：**
```
HTML 解析 → DOM 树构建
                    ↓
              CSS 解析 → CSSOM 树构建
                    ↓
              DOM + CSSOM → 渲染树（Render Tree）
                    ↓
              布局（Layout/Reflow）
                    ↓
              绘制（Paint）
                    ↓
              合成（Composite）
```

- HTML 解析是**增量过程**，遇到 `<script>` 可能阻塞
- CSS 是**渲染阻塞资源**，必须完全下载解析后才能构建 CSSOM
- JS 会阻塞 HTML 解析（除非 defer/async）

**6. 关键渲染路径优化目标：**
- 减少关键资源数量（CSS/JS）
- 缩短关键资源加载时间
- 减少关键渲染路径长度

#### 真实面试题

**题目：简单描述从输入网址到页面显示的过程**

**满分答案：**

完整过程可分为7个主要阶段：

**第一步：URL解析**
- 浏览器解析用户输入的URL，识别协议、域名、端口、路径等
- 如果是搜索词而非URL，则使用默认搜索引擎

**第二步：DNS解析**
- 将域名解析为IP地址
- 查找顺序：浏览器缓存 → 系统缓存 → 路由器缓存 → ISP DNS → 递归查询
- 可用 `dns-prefetch` 预解析跨域域名

**第三步：TCP三次握手**
- 客户端发送SYN包
- 服务器返回SYN+ACK包
- 客户端发送ACK包，连接建立
- HTTPS还需要额外的TLS/SSL握手

**第四步：发送HTTP请求**
- 构建请求行和请求头
- 携带Cookie等认证信息
- 若是POST等方法，还包含请求体

**第五步：服务器处理并返回响应**
- 服务器根据路径处理请求
- 返回HTTP响应（状态码 + 响应头 + 响应体）

**第六步：浏览器渲染（核心流水线）**
1. **解析HTML → 构建DOM树**：从上到下解析HTML标签
2. **解析CSS → 构建CSSOM树**：处理所有CSS资源
3. **DOM + CSSOM → 生成渲染树**：只包含可见节点
4. **布局（Layout）**：计算每个节点的几何信息
5. **绘制（Paint）**：将节点绘制到多个图层
6. **合成（Composite）**：将各图层合成为最终画面

**第七步：绘制到屏幕**
- 浏览器将合成的帧数据发送给GPU
- GPU将内容绘制到屏幕上

**加分项：**
- 提及CSS加载会阻塞渲染（必须等CSSOM构建完才能生成渲染树）
- 提及script标签的defer/async属性影响渲染时机
- 提及关键渲染路径优化（内联关键CSS、延迟非关键CSS）

---

## 1.9 设备dpr

### 题目
一台设备的dpr，是否是可变的?

#### 知识点详解

**dpr（Device Pixel Ratio，设备像素比）**

```
dpr = 物理像素（设备屏幕实际像素点） / 逻辑像素（CSS 像素）
```

```javascript
// 获取设备像素比
console.log(window.devicePixelRatio);

// 示例
// iPhone 14: 逻辑像素 390×844，物理像素 1170×2532
// dpr = 1170 / 390 = 3

// iPhone 8: 逻辑像素 375×667，物理像素 750×1334
// dpr = 2
```

**物理 dpr（设备固定）：**

- 物理像素是屏幕制造商在生产时就确定的，不可改变
- 屏幕分辨率（如 1170×2532）是硬件固定参数

**逻辑 dpr（可被人为改变）：**

- 用户缩放（浏览器缩放，Ctrl+/Ctrl-）会改变 CSS 像素与物理像素的映射关系
- `window.devicePixelRatio` 的值会随缩放而改变
- 但用户通常不会主动缩放，默认情况下 dpr 对应物理 dpr

**CSS 中的影响：**

```css
/* 1 个 CSS 像素 = 1 * dpr 个物理像素 */

/* 高清屏适配：设计稿 750px 的图，在 dpr=2 屏上需要 1500px 物理像素 */
@media (-webkit-min-device-pixel-ratio: 2), (min-resolution: 192dpi) {
    .icon {
        background-image: url('icon@2x.png');  /* 1500px */
    }
}

/* dpr=3 时 */
@media (-webkit-min-device-pixel-ratio: 3), (min-resolution: 288dpi) {
    .icon {
        background-image: url('icon@3x.png');  /* 2250px */
    }
}

/* CSS zoom 会改变逻辑像素 */
.box {
    zoom: 2;  /* 放大 2 倍，物理像素占用增加 */
}

/* transform: scale() 不会改变布局，只改变渲染 */
.box {
    transform: scale(2);
}
```

**viewport meta 缩放：**

```html
<!-- initial-scale=0.5 使 CSS 像素变大（视觉缩小） -->
<!-- 此时 1 CSS 像素 = 0.5 逻辑像素，影响 dpr 感知 -->
<meta name="viewport" content="width=device-width, initial-scale=0.5">
```

#### 真实面试题

**题目：一台设备的dpr，是否是可变的?**

**满分答案：**

**结论：物理 dpr 固定不变，逻辑 dpr 可以通过缩放改变。**

**什么是 dpr：**
- dpr（Device Pixel Ratio）= 物理像素 / CSS 逻辑像素
- iPhone 14 的 dpr=3，意味着 1 个 CSS 像素占用 3×3=9 个物理像素点
- 物理像素是屏幕硬件决定的，屏幕出厂后物理分辨率就固定了

**物理 dpr 不可变：**
- 屏幕的物理像素点数量由显示器硬件决定
- 1920×1080 的显示器，无论如何 dpr=1（按 96dpi 标准）

**逻辑 dpr 可变：**
- 用户缩放（浏览器缩放 Ctrl+/Ctrl-）会改变 CSS 像素与物理像素的映射
- `window.devicePixelRatio` 的值会随缩放变化
- CSS `zoom` 属性、viewport `initial-scale` 都可以改变缩放比例
- `transform: scale()` 只改变渲染，不改变布局逻辑

**实际开发中的注意点：**
- 高清屏（dpr=2/3）需要提供 2x/3x 分辨率的图片
- 使用 `@media (-webkit-min-device-pixel-ratio: 2)` 做高清适配
- 可以用 `<img srcset="img@2x.jpg 2x, img@3x.jpg 3x">` 让浏览器自动选择
- 现代浏览器默认不缩放，用户一般不会主动缩放页面
- `window.devicePixelRatio` 在默认情况下反映的就是物理 dpr

---

## 1.10 前端图片格式选择

### 题目
前端该如何选择图片的格式?

#### 知识点详解

**各主流图片格式对比：**

| 格式 | 压缩方式 | 透明通道 | 动画支持 | 兼容性 | 典型用途 |
|------|---------|---------|---------|--------|---------|
| JPEG | 有损 | ❌ | ❌ | 100% | 照片、复杂图像 |
| PNG-8 | 无损 | ✅ | ❌ | 100% | 图标、Logo（≤256色） |
| PNG-24 | 无损 | ✅ | ❌ | 100% | Logo、需要透明的图 |
| WebP | 有损/无损 | ✅ | ✅ | 主流浏览器 | 现代应用首选 |
| AVIF | 高压缩 | ✅ | ✅ | 现代浏览器 | 最高压缩率场景 |
| SVG | 矢量 | ✅ | ✅ | 100% | 图标、Logo、矢量图 |
| GIF | 无损 | ✅ | ✅ | 100% | 简单动图（已不推荐） |

**JPEG：**
```html
<!-- 适合：照片、风景、人像等色彩丰富的图像 -->
<img src="photo.jpg" alt="照片" width="800" height="600">
```

**PNG：**
```html
<!-- 适合：需要透明背景的 Logo、图标、截图 -->
<!-- PNG-8：256色以内的小图，体积小 -->
<!-- PNG-24：色彩丰富或需要半透明的图 -->
<img src="logo.png" alt="Logo" width="200" height="80">
```

**WebP（推荐优先选择）：**
```html
<!-- 相比 JPEG 体积减少 25-35%，透明度支持 -->
<!-- 相比 PNG 无损压缩减少 26% -->
<picture>
    <source type="image/webp" srcset="image.webp">
    <img src="image.jpg" alt="降级方案">
</picture>

<!-- 直接使用 -->
<img src="photo.webp" alt="照片">
```

**AVIF（最新最强压缩）：**
```html
<!-- 压缩率比 WebP 再高 50%，但兼容性较差 -->
<picture>
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">
    <img src="image.jpg" alt="降级方案">
</picture>
```

**SVG（矢量图）：**
```html
<!-- 适合：图标、Logo、简单图形，可无限缩放 -->
<!-- 代码可直接内联，性能好 -->
<svg width="24" height="24" viewBox="0 0 24 24">
    <path d="M12 2L2 7l10 5 10-5-10-5z" fill="currentColor"/>
</svg>

<!-- 外部引用 -->
<img src="icon.svg" alt="图标" width="24" height="24">
```

**GIF（已基本被淘汰）：**
```html
<!-- 简单动画（抖动问题、颜色少） -->
<!-- 建议：用 CSS 动画或 WebP/AVIF 动图替代 -->
```

**现代图片选型策略：**

```html
<!-- 最佳实践：渐进增强 + 降级 -->
<picture>
    <!-- 最高压缩：现代浏览器 -->
    <source
        media="(min-width: 800px)"
        type="image/avif"
        srcset="large.avif">

    <!-- 中等压缩：主流浏览器 -->
    <source
        type="image/webp"
        srcset="large.webp">

    <!-- 兜底方案 -->
    <img src="large.jpg" alt="响应式图片" width="800" height="600">
</picture>

<!-- 移动端适配：srcset -->
<img
    srcset="small.jpg 500w, medium.jpg 800w, large.jpg 1200w"
    sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 33vw"
    src="medium.jpg"
    alt="适配不同屏幕">
```

#### 真实面试题

**题目：前端该如何选择图片的格式?**

**满分答案：**

图片格式的选择需要综合考虑**图像内容、兼容性要求、性能需求**三个维度：

**JPEG（适合照片类）：**
- 有损压缩，压缩率高，体积小
- 不支持透明，不支持动画
- 场景：产品照片、风景图、人物图等色彩丰富的照片

**PNG（适合需要透明或无损）：**
- 无损压缩，保留所有像素信息
- 支持透明（PNG-24 支持半透明）
- 场景：Logo、图标（需要透明背景）、截图、需要锐利边缘的设计图

**WebP（现代首选）：**
- Google 开发，同时支持有损和无损压缩
- 相比 JPEG 体积减少 25-35%，相比 PNG 无损减少 26%
- 支持透明，支持动画
- 场景：通用场景优先选择 WebP，性能收益明显

**AVIF（最高压缩率）：**
- 基于 AV1 视频编码，压缩率最高（比 WebP 再小约 50%）
- 支持透明和动画，但兼容性较差
- 场景：对性能极致追求、兼容性要求低的现代应用

**SVG（矢量图首选）：**
- 矢量格式，可无限缩放不失真
- 体积小（简单图形），支持透明和动画
- 场景：图标、Logo、简单几何图形、图表
- 注意：复杂照片不适合 SVG

**GIF（已不推荐）：**
- 支持动画但颜色少（256色），有抖动问题
- 建议：用 CSS 动画替代简单动图，用 WebP/AVIF 动图替代复杂动画

**实际开发中的最佳实践：**
```html
<!-- 使用 picture 元素实现渐进增强 -->
<picture>
    <source type="image/avif" srcset="image.avif">   <!-- 最新格式 -->
    <source type="image/webp" srcset="image.webp">     <!-- 主流格式 -->
    <img src="image.jpg" alt="兜底">                 <!-- 最强兼容 -->
</picture>
```

**选型总结：**
- 图标/Logo → SVG
- 照片/复杂图 → WebP（优选）或 JPEG（兼容优先）
- 追求极致性能 → AVIF + WebP 降级
- 简单动画 → CSS 动画或 WebP 动图，告别 GIF

---

## 1.11 跨页面通信

### 题目
前端跨页面通信，你知道哪些方法?

#### 知识点详解

**同源页面通信：**

**1. localStorage / sessionStorage + storage 事件：**
```javascript
// 页面 A：存储数据
localStorage.setItem('token', 'abc123');
sessionStorage.setItem('user', JSON.stringify({ name: '张三' }));

// 页面 B：监听变化（不同标签页之间通信）
window.addEventListener('storage', (e) => {
    console.log('key:', e.key);
    console.log('newValue:', e.newValue);
    console.log('oldValue:', e.oldValue);
    // e.key === null 时，清除所有（clear）
});
```
- `localStorage`：同源所有标签页共享，持久化
- `sessionStorage`：仅当前标签页，不共享
- `storage` 事件：其他标签页修改时触发，当前标签页修改不触发

**2. BroadcastChannel API：**
```javascript
// 创建频道
const channel = new BroadcastChannel('my_channel');

// 发送消息
channel.postMessage({ type: 'UPDATE', data: { count: 10 } });

// 接收消息
channel.addEventListener('message', (e) => {
    console.log('收到:', e.data);
});

// 关闭频道
channel.close();
```
- 支持任意同源页面（标签页、iframe、Worker）
- API 简洁，比 localStorage 事件更现代

**3. SharedWorker：**
```javascript
// shared-worker.js
const connections = new Set();

self.onconnect = (e) => {
    const port = e.ports[0];
    connections.add(port);

    port.onmessage = (e) => {
        // 广播给所有连接
        connections.forEach(p => p.postMessage(e.data));
    };

    port.start();
};

// 页面中使用
const worker = new SharedWorker('shared-worker.js');
const port = worker.port;
port.onmessage = (e) => console.log('收到:', e.data);
port.start();
port.postMessage('hello');
```

**4. Service Worker：**
```javascript
// service-worker.js
self.addEventListener('message', (e) => {
    // 广播消息
    self.clients.matchAll().then(clients => {
        clients.forEach(client => client.postMessage(e.data));
    });
});

// 页面通信
navigator.serviceWorker.controller?.postMessage({ type: 'SYNC' });
navigator.serviceWorker.addEventListener('message', (e) => {
    console.log(e.data);
});
```

**5. window.postMessage：**
```javascript
// 发送（可跨域！）
otherWindow.postMessage({ type: 'ping' }, 'https://target-origin.com');

// 接收
window.addEventListener('message', (e) => {
    // 始终验证 origin
    if (e.origin !== 'https://expected-origin.com') return;
    console.log('收到:', e.data);
});
```
- 主要用于 iframe、popup、新窗口之间的通信
- 可以跨域使用，但必须验证 `e.origin`

**跨域页面通信：**

**1. postMessage（跨域可用）：**
- 同源和跨域场景都可以用
- 关键点：始终验证消息来源的 origin

**2. 服务端中转：**
```javascript
// WebSocket（实时双向通信）
const ws = new WebSocket('wss://api.example.com/ws');
ws.onmessage = (e) => console.log(e.data);
ws.send('hello');

// 轮询（定时向服务端请求）
setInterval(() => {
    fetch('/api/poll').then(r => r.json()).then(data => {
        // 处理数据更新
    });
}, 5000);
```

**3. 第三方库：**
- 基于 BroadcastChannel 的封装（如 `Comlink`）
- 基于 SharedWorker 的封装
- 基于 postMessage + iframe 的封装

#### 真实面试题

**题目：前端跨页面通信，你知道哪些方法?**

**满分答案：**

前端跨页面通信分为**同源**和**跨域**两大类场景：

**同源页面通信：**

**1. localStorage + storage 事件**
- `localStorage` 在同源的所有标签页之间共享
- 一个标签页修改，其他标签页会触发 `storage` 事件
- 适合：状态同步（如登录状态、主题切换）
```javascript
// A页面
localStorage.setItem('theme', 'dark');
// B页面
window.addEventListener('storage', e => {
    if (e.key === 'theme') applyTheme(e.newValue);
});
```

**2. BroadcastChannel API（现代推荐）**
- 同源标签页、iframe、Worker 之间通信
- API 简洁，功能强大
```javascript
const ch = new BroadcastChannel('app_channel');
ch.postMessage({ action: 'refresh' });
ch.onmessage = e => console.log(e.data);
```

**3. SharedWorker**
- 多个页面共享同一个 Worker 实例
- 适合：高频数据同步、复杂状态管理

**4. window.postMessage**
- 适合：iframe 与父页面、popup 窗口之间的通信
- 可跨域使用，但必须验证 origin

**跨域页面通信：**

**1. postMessage（跨域可用）**
- 唯一原生支持跨域页面通信的方法
- 必须验证 `e.origin` 防止安全漏洞

**2. 服务端中转**
- WebSocket：实时双向通信（推荐）
- HTTP 轮询/SSE：单向或准实时

**3. 第三方库**
- 封装了以上方法，提供统一 API

**场景选型建议：**
| 场景 | 推荐方案 |
|------|---------|
| 简单状态同步 | localStorage + storage |
| 多标签实时通信 | BroadcastChannel |
| iframe 通信 | postMessage |
| 跨域实时通信 | WebSocket |
| 复杂状态管理 | SharedWorker |

---

## 1.12 DOM树的理解

### 题目
说说你对DOM树的理解

#### 知识点详解

**DOM 树的概念：**

浏览器将 HTML 文档解析为一棵以 `document` 为根节点的树形结构，每个 HTML 标签对应树上的一个元素节点。

```
document
└── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── header
        │   └── nav
        │       └── ul
        │           ├── li × 3
        └── main
            └── article
                ├── h1
                └── p
```

**Node 节点类型（nodeType）：**

| nodeType | 名称 | 说明 |
|----------|------|------|
| 1 | ELEMENT_NODE | 元素节点 |
| 3 | TEXT_NODE | 文本节点（包含空白） |
| 8 | COMMENT_NODE | 注释节点 |
| 9 | DOCUMENT_NODE | document 节点 |
| 10 | DOCUMENT_TYPE_NODE | DOCTYPE |
| 11 | DOCUMENT_FRAGMENT_NODE | 文档片段 |

```javascript
// 判断节点类型
element.nodeType === 1;  // 元素节点
textNode.nodeType === 3; // 文本节点
commentNode.nodeType === 8; // 注释节点
```

**DOM 操作 API：**

```javascript
// 创建节点
const div = document.createElement('div');
const text = document.createTextNode('Hello');
const fragment = document.createDocumentFragment();

// 插入节点
parent.appendChild(child);              // 末尾添加
parent.insertBefore(newNode, refNode);  // 插入到 refNode 前
parent.append(newNode1, newNode2);     // 末尾添加多个（现代 API）

// 删除节点
parent.removeChild(child);
element.remove();  // 直接删除（现代 API）

// 替换节点
parent.replaceChild(newChild, oldChild);
element.replaceWith(newElement);

// 查询节点
document.getElementById('id');
document.querySelector('.class');
document.querySelectorAll('.items');  // 返回 NodeList（静态）

// 属性操作
element.setAttribute('data-id', '123');
element.getAttribute('data-id');
element.removeAttribute('data-id');

// 类名操作
element.classList.add('active');
element.classList.remove('hidden');
element.classList.toggle('active');
```

**虚拟 DOM 与真实 DOM：**

```javascript
// 虚拟 DOM：用 JS 对象描述 DOM 结构
const vnode = {
    type: 'div',
    props: { class: 'container' },
    children: [
        { type: 'h1', props: {}, children: 'Title' },
        { type: 'p', props: { class: 'text' }, children: 'Content' }
    ]
};

// 真实 DOM：浏览器的 DOM 树结构
// 虚拟 DOM 是真实 DOM 的 JS 映射，React/Vue 用它实现 diff 算法
```

#### 真实面试题

**题目：说说你对DOM树的理解**

**满分答案：**

**DOM 树是什么：**

DOM（Document Object Model，文档对象模型）树是浏览器将 HTML 文档解析为的树形数据结构。每个 HTML 标签、文本内容、注释都是一个节点，所有节点按层级关系组织成一棵树，`document` 是整棵树的根节点。

**节点类型：**

DOM 树中包含多种类型的节点：

| 类型 | nodeType | 说明 |
|------|----------|------|
| 元素节点 | 1 | HTML 标签，如 `<div>`、`<p>` |
| 文本节点 | 3 | 标签内的文本内容，包括空白 |
| 注释节点 | 8 | HTML 注释 `<!-- -->` |
| Document | 9 | document 对象本身 |
| DocumentType | 10 | `<!DOCTYPE html>` |
| DocumentFragment | 11 | 文档片段（不渲染的容器） |

```javascript
// 示例：nodeType 判断
console.log(divEl.nodeType === Node.ELEMENT_NODE);  // true
console.log(text.nodeType === Node.TEXT_NODE);       // true
```

**DOM 操作：**

- **创建**：`createElement`、`createTextNode`、`createDocumentFragment`
- **插入**：`appendChild`、`insertBefore`、`append`（现代 API）
- **删除**：`removeChild`、`remove`（现代 API）
- **查询**：`getElementById`、`querySelector`、`querySelectorAll`
- **属性**：`getAttribute`、`setAttribute`、`classList`

**渲染阻塞关系：**
- HTML 解析 → DOM 树构建
- CSS 解析 → CSSOM 构建
- DOM + CSSOM → 渲染树
- 渲染树 → 布局 → 绘制

**虚拟 DOM：**

虚拟 DOM 是真实 DOM 的 JavaScript 对象表示，React/Vue 等框架用它来：
1. 减少对真实 DOM 的直接操作
2. 通过 diff 算法计算出最小更新
3. 批量更新 DOM，减少重排重绘

```javascript
// 虚拟 DOM 示例
const vdom = {
    type: 'div',
    props: { class: 'app' },
    children: [
        { type: 'h1', children: 'Hello' }
    ]
};
```

**理解要点：**
- DOM 是浏览器提供的接口，不是 JavaScript 语言的一部分
- 操作 DOM 会触发浏览器的重排和重绘，影响性能
- 虚拟 DOM 是对真实 DOM 的抽象，是框架层面的优化策略

---

## 1.13 行内元素、块级元素、空元素

### 题目
行内元素有哪些?块级元素有哪些?空(void)元素有哪些?

#### 知识点详解

**行内元素（Inline Elements）：**

```html
<!-- 常见行内元素 -->
<span>行内容器</span>
<a href="#">链接</a>
<em>强调（斜体）</em>
<strong>强调（加粗）</strong>
<i>斜体（无语义）</i>
<b>加粗（无语义）</b>
<u>下划线</u>
<s>删除线</s>
<code>代码片段</code>
<kbd>键盘按键</kbd>
<abbr title="缩写">HTML</abbr>
<sub>下标</sub>
<sup>上标</sup>
<small>小字</small>
<img src="..." alt="图片">  <!-- 可设宽高 -->
<input type="text">        <!-- 可设宽高 -->
<button>按钮</button>      <!-- 可设宽高 -->
<label>标签</label>
<select>下拉框</select>
<textarea>文本域</textarea>
<br>换行（虽可设宽高，但不是真正行内块）
```

**行内元素特性：**
- 不独占一行，多个行内元素可以在同一行
- 设置 `width`/`height` **无效**（img、input、button、select、textarea 例外）
- 设置上下 `margin`/`padding` 不推开同行的其他元素
- 只能包含文本或其他行内元素

**块级元素（Block-level Elements）：**

```html
<!-- 常见块级元素 -->
<div>通用块级容器</div>
<p>段落</p>
<h1>~<h6>标题</h6></h6></h1>
<ul>无序列表</ul>
<ol>有序列表</ol>
<li>列表项</li>
<dl>定义列表</dl>
<dt>定义项</dt>
<dd>定义描述</dd>
<table>表格</table>
<tr>表格行</tr>
<td>单元格</td>
<th>表头单元格</th>
<thead>表头区域</thead>
<tbody>表体区域</tbody>
<tfoot>表脚区域</tfoot>
<form>表单</form>
<fieldset>字段集</fieldset>
<address>地址</address>
<hr>分隔线
<blockquote>长引用</blockquote>
<pre>预格式化文本</pre>

<!-- HTML5 语义化块级元素 -->
<header>头部</header>
<nav>导航</nav>
<main>主体</main>
<article>文章</article>
<section>章节</section>
<aside>侧边栏</aside>
<footer>底部</footer>
<figure>独立内容块</figure>
<figcaption>figure 的标题</figcaption>
<details>详情折叠</details>
<summary>折叠标题</summary>
```

**块级元素特性：**
- 独占一行（父容器宽度）
- 可以设置 `width`、`height`、`margin`、`padding`
- 默认宽度 100%（不设宽度时）
- 可以包含块级元素和行内元素

**空元素（Void Elements / 自闭合元素）：**

```html
<!-- 空元素：没有闭合标签，不能包含子节点 -->
<br>          <!-- 换行 -->
<hr>          <!-- 水平线 -->
<img src="..." alt="...">  <!-- 图片 -->
<input type="text">        <!-- 输入框 -->
<meta charset="UTF-8">    <!-- 元数据 -->
<link rel="stylesheet" href="style.css">  <!-- 外部资源 -->
<area shape="rect" coords="0,0,100,100" href="#">  <!-- 图片热区 -->
<base href="/">           <!-- 基准 URL -->
<col span="2">           <!-- 表格列 -->
<embed src="video.mp4">  <!-- 嵌入内容 -->
<source src="video.mp4">  <!-- 多媒体源 -->
<track kind="subtitles" src="sub.vtt">  <!-- 字幕轨道 -->
<wbr>           <!-- 软换行 -->
```

**display 属性转换：**

```css
/* 行内 → 块级 */
span { display: block; }

/* 块级 → 行内 */
div { display: inline; }

/* 块级 → 行内块（兼具两者优点） */
span { display: inline-block; }
/* 特点：同行显示 + 可设宽高 */
```

#### 真实面试题

**题目：行内元素有哪些?块级元素有哪些?空(void)元素有哪些?**

**满分答案：**

**行内元素（Inline Elements）：**
- 常见：`span`、`a`、`em`、`strong`、`i`、`b`、`u`、`s`
- 文本相关：`code`、`kbd`、`abbr`、`sub`、`sup`、`small`
- 表单相关：`img`（可设宽高）、`input`（可设宽高）、`button`（可设宽高）、`label`、`select`、`textarea`（可设宽高）

**块级元素（Block-level Elements）：**
- 容器：`div`、`p`、`ul`、`ol`、`li`、`dl`、`dt`、`dd`
- 表格：`table`、`tr`、`td`、`th`、`thead`、`tbody`、`tfoot`
- 表单：`form`、`fieldset`
- 标题：`h1` ~ `h6`
- 其他：`header`、`nav`、`main`、`article`、`section`、`aside`、`footer`、`figure`、`details`、`address`、`blockquote`、`pre`

**空元素（Void Elements）：**
- 常用：`br`、`hr`、`img`、`input`、`meta`、`link`
- 媒体：`source`、`track`、`embed`
- 其他：`area`（热区）、`base`、`col`、`wbr`

**三者的关键区别：**

| 特性 | 行内元素 | 块级元素 | 空元素 |
|------|---------|---------|--------|
| `display` 默认值 | `inline` | `block` | 行内或特殊 |
| 独占一行 | ❌ | ✅ | ❌ |
| 可设宽高 | ❌（部分除外） | ✅ | ❌ |
| 上下 margin | ❌ 推开外部 | ✅ | ❌ |
| 上下 padding | 不推开邻居 | ✅ | — |
| 包含内容 | 文本/行内 | 任意 | 无内容 |

**最佳实践：**
- 布局用块级元素，文本样式用行内元素
- 用 CSS `display` 属性按需转换：`inline`、`block`、`inline-block`
- HTML5 中用语义化标签替代部分 `div`（header/nav/main/article/section/aside/footer）

---

## 1.14 HTML和CSS中的图片加载与渲染规则

### 题目
HTML和CSS中的图片加载与渲染规则是什么样的?

#### 知识点详解

**HTML `<img>` 的加载规则：**

```html
<!-- 遇到 src 属性立即发起请求 -->
<img src="large-image.jpg" alt="描述" width="800" height="600">

<!-- loading="lazy" 懒加载：进入视口才加载 -->
<img src="image.jpg" loading="lazy" alt="懒加载图片">

<!-- decoding="async" 异步解码，不阻塞主线程 -->
<img src="image.jpg" decoding="async" alt="异步解码">

<!-- fetchpriority 优先级（high/low/auto） -->
<img src="hero.jpg" fetchpriority="high" alt="首图高优先级">

<!-- width/height 防止布局抖动（CLS） -->
<img src="image.jpg" width="800" height="600" alt="固定尺寸">
```

**`<img>` 加载时机：**
1. 解析到 `<img>` 标签，立即下载 `src` 资源
2. 多个 `<img>` 并行下载，受浏览器并发连接数限制（通常 6 个/域名）
3. `loading="lazy"` 时，图片进入视口前不下载
4. CSS 中的背景图不受 `loading` 属性控制

**CSS 背景图的加载规则：**

```css
/* CSSOM 构建时不加载，只有渲染树生成后才加载 */
.hero {
    background-image: url('hero-bg.jpg');
    /* 条件：
       1. 该元素的 CSS 规则被匹配
       2. 该元素在渲染树中（可见）
       3. 样式计算完成
    */
}

/* 首
屏大图（通常不需要懒加载）*/
@media (min-width: 1200px) {
    .hero {
        background-image: url('hero-large.jpg');
    }
}
```

**关键区别：**

| 对比项 | `<img>` | `background-image` |
|--------|---------|-------------------|
| 加载时机 | HTML 解析时立即下载 | CSSOM 构建时不加载，渲染树生成后才加载 |
| 懒加载 | `loading="lazy"` | CSS 无法懒加载，需 JS/IntersectionObserver |
| 占位 | 占据空间（需设宽高或 aspect-ratio） | 不占文档流空间 |
| SEO | alt 属性可被索引 | 无法被搜索引擎索引 |
| 打印 | 可设置打印样式 | 需额外配置 `@media print` |
| CSS 动画 | — | 可用 `background-position` 做精灵图 |

**图片预加载：**

```html
<!-- 预加载关键图片 -->
<link rel="preload" as="image" href="hero.jpg" fetchpriority="high">

<!-- 预加载 WebP 版本 -->
<link rel="preload" as="image" href="hero.webp" type="image/webp">

<!-- DNS 预解析 -->
<link rel="dns-prefetch" href="//static.example.com">

<!-- 预连接 -->
<link rel="preconnect" href="https://static.example.com" crossorigin>
```

**IntersectionObserver 实现懒加载：**

```javascript
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const img = entry.target;
            img.src = img.dataset.src;  // 使用 data-src 存储真实 URL
            observer.unobserve(img);
        }
    });
}, { rootMargin: '200px' });  // 提前 200px 开始加载

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

**Base64 内联图片：**

```html
<!-- 增大 HTML 体积，但减少 HTTP 请求 -->
<img src="data:image/png;base64,iVBORw0KGgoAAAANS..." alt="图标">

<!-- 适用：小图标（< 2KB），减少请求数 -->
<!-- 不适用：大图，增大 CSS/HTML 体积，缓存失效时全部重新下载 -->
```

#### 真实面试题

**题目：HTML和CSS中的图片加载与渲染规则是什么样的?**

**满分答案：**

**HTML `<img>` 的加载规则：**
- 解析到 `<img>` 标签时，立即发起 `src` 请求（不等待页面渲染）
- 受浏览器并发连接数限制，同一域名通常最多 6 个并发请求
- `loading="lazy"` 可实现懒加载——图片进入视口前不下载
- `fetchpriority="high"` 标记高优先级图片，优先加载
- 设置 `width`/`height` 属性可防止布局抖动（CLS）

**CSS `background-image` 的加载规则：**
- CSS 解析阶段（CSSOM 构建）**不会加载**背景图
- 只有当元素的 CSS 规则被匹配，且该元素出现在渲染树中（可见）时，才会加载
- 如果元素被 `display: none` 隐藏，其中的背景图**不会加载**
- 如果元素不在视口内，浏览器可能延迟加载

**两者的核心区别：**
1. `<img>` 在 HTML 解析时就下载；CSS 背景图要等渲染树生成后才下载
2. `<img>` 占据文档流空间；背景图不占空间
3. `<img>` 可以通过 `alt` 被 SEO 索引；背景图无法被索引
4. `<img>` 可以直接用 `loading="lazy"`；背景图需要 IntersectionObserver 实现懒加载

**实际开发建议：**
- 首屏关键图片用 `<img>` + `fetchpriority="high"` 或 `preload`
- 非关键图片用 `loading="lazy"`
- 图标、装饰图用 CSS `background-image`，配合雪碧图减少请求
- 小图标可用 Base64 内联，减少请求但增加 CSS 体积
- 用 `<link rel="preload" as="image">` 预加载关键图片

---

## 1.15 title与h1、b与strong、i与em的区别

### 题目
title与h1的区别、b与strong的区别、i与em的区别?

#### 知识点详解

**title vs h1：**

| 对比项 | `<title>` | `<h1>` |
|--------|-----------|--------|
| 位置 | `<head>` 内，不可视元数据 | `<body>` 内，页面可见内容 |
| 页面显示 | 不显示在页面内容中 | 显示为页面主标题 |
| 显示位置 | 浏览器标签页、搜索结果、书签 | 页面内容区域 |
| SEO 影响 | 最重要的 SEO 元素之一 | 重要，影响页面语义和权重 |
| 数量 | 每个页面只能有一个 | 每个页面建议只有一个（最佳实践） |
| 字符限制 | 建议 50-60 字符（搜索结果截断） | 无限制 |

```html
<head>
    <title>2025年前端面试题库 - HTML/CSS/JavaScript满分答案</title>
</head>
<body>
    <h1>前端面试题库</h1>  <!-- 页面主标题 -->
</body>
```

**b vs strong：**

| 对比项 | `<b>`（bold） | `<strong>` |
|--------|--------------|-----------|
| 语义 | 视觉加粗，无语义 | 语义强调（重要性、严重性） |
| 默认样式 | 加粗 | 加粗（相同） |
| SEO | 无影响 | 可能获得更高权重 |
| 可访问性 | 无特殊处理 | 屏幕阅读器会加重语气 |
| 使用场景 | 产品名称、关键词（无强调含义） | 警告、重要信息、关键数据 |

```html
<!-- b：无语义，仅视觉加粗 -->
<p>欢迎使用 <b>微信</b> 支付</p>
<p>React 和 <b>Vue</b> 都是优秀框架</p>

<!-- strong：语义强调 -->
<p><strong>警告：</strong>此操作不可撤销！</p>
<p>限时优惠：<strong>￥99</strong>（原价 ￥199）</p>
```

**i vs em：**

| 对比项 | `<i>`（italic） | `<em>`（emphasis） |
|--------|----------------|-------------------|
| 语义 | 视觉斜体，无语义 | 语义强调，语气重读 |
| 默认样式 | 斜体 | 斜体（相同） |
| 可访问性 | 无特殊处理 | 屏幕阅读器会加重语气 |
| HTML5 语义 | 技术术语、外语词汇、文学作品名称 | 强调的内容 |

```html
<!-- i：无语义斜体 -->
<p>The word <i>alpha</i> comes from Greek.</p>
<p>书名：《<i>三国演义</i>》</p>
<p>计算机术语：<i>API</i>、<i>DOM</i></p>

<!-- em：语义强调 -->
<p>我 <em>真的</em> 喜欢这本书！</p>
<p>他 <em>没有</em> 来参加会议。</p>
```

**HTML5 中的变化：**
- `<b>` 和 `<i>` 在 HTML5 中被重新定义为有语义的角色：
  - `<b>`：用于"在不传达额外重要性的情况下，吸引读者注意的样式范围"
  - `<i>`：用于"技术术语、外语词汇、文学作品名称等"
- 但语义仍不如 `<strong>` 和 `<em>` 强烈，优先使用后者

#### 真实面试题

**题目：title与h1的区别、b与strong的区别、i与em的区别?**

**满分答案：**

**title vs h1：**

- `<title>` 位于 `<head>` 中，是元数据，不显示在页面内容中，出现在浏览器标签页标题、搜索引擎结果、书签中
- `<h1>` 位于 `<body>` 中，是可见的页面主标题，用户一眼就能看到
- 两者的核心区别是：**title 管页面在浏览器/搜索引擎中的展示，h1 管页面内容本身的标题**
- SEO 角度：`<title>` 是最重要的排名因素之一，`<h1>` 是页面语义的重要信号
- 每个页面应该有一个唯一的 `<title>`；`<h1>` 通常每页一个（最佳实践）

**b vs strong：**

- `<b>` 是无语义的视觉加粗（presentational），仅表示样式
- `<strong>` 是语义强调（semantic），表示内容的重要性、严重性
- 两者的默认样式都是加粗，但含义完全不同
- 屏幕阅读器会加重语气朗读 `<strong>` 内容
- 实际使用：有真正的重要/强调含义用 `<strong>`，仅需要粗体样式用 CSS `font-weight: bold` 或 `<b>`

**i vs em：**

- `<i>` 是无语义的视觉斜体，仅表示样式
- `<em>` 是语义强调，语气比普通更强（可以多层嵌套递进强调）
- 屏幕阅读器会加重语气朗读 `<em>` 内容
- HTML5 规范中 `<i>` 用于技术术语、外语词汇、书名等，但语义仍不如 `<em>`

**核心结论：**

每组区别的核心都是 **视觉表现 vs 语义含义**。现代前端开发应优先使用语义化标签 `<strong>` 和 `<em>`，只有在真正无语义需求时才用 `<b>` 和 `<i>`，或者直接用 CSS 控制样式。

---

## 1.16 script标签放置位置

### 题目
script标签为什么建议放在body标签的底部(defer、async)

#### 知识点详解

**为什么脚本放在底部（传统方案）：**

```html
<body>
    <div id="app"></div>

    <!-- 脚本放底部：确保 DOM 已解析完毕 -->
    <script src="app.js"></script>
</body>
```

**问题分析——脚本阻塞 DOM 解析：**

```
场景：script 放在 <head> 中（同步执行）

HTML 解析:  ████████████████░░░░░░░░░░░░░░░░░░░░░░
脚本执行:         ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓（假设耗时 2s）
DOM Ready:                                    ████

→ 白屏时间 = HTML 解析时间 + 脚本执行时间
→ 如果 app.js 中有 document.getElementById('app')，
  但此时 #app 还没解析到，会报错！
```

**脚本放在 body 底部的效果：**

```
场景：script 放在 body 底部

HTML 解析 + 渲染:  ████████████████████████████████
（#app 已加载到 DOM）

脚本执行:                                    ▓▓▓▓▓▓
DOM Ready:                                  ████

→ 脚本执行时，DOM 已完整构建
→ 可以安全操作任何 DOM 节点，不报错
→ 页面已有内容可见，不白屏
```

**defer 属性（现代推荐）：**

```html
<head>
    <!-- defer：并行下载，DOM 解析完成后按文档顺序执行 -->
    <script src="app.js" defer></script>
</head>
<body>
    <div id="app"></div>
</body>
```

```
HTML 解析:  ████████████████████████████████████
JS 下载:     ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
JS 执行:                                          ▓▓▓▓▓▓

→ HTML 解析不被阻塞，页面更快渲染
→ JS 在 DOM 完全就绪后执行，安全
→ 按文档顺序执行，保证依赖关系
```

**async 属性：**

```html
<head>
    <!-- async：并行下载，下载完立即执行，不保证顺序 -->
    <script src="analytics.js" async></script>
</head>
```

```
HTML 解析:  ████████████████████████████████████
JS 下载:     ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
JS 执行:              ▓▓▓▓▓（下载完即执行）

→ 不阻塞 HTML 解析
→ 但可能阻塞渲染（执行时）
→ 不保证执行顺序，不适合有依赖的脚本
→ 适合：独立脚本（统计、监控、广告 SDK）
```

**三种方式对比：**

| 方式 | 放置位置 | 加载 | 执行时机 | 执行顺序 | 适用场景 |
|------|---------|------|---------|---------|---------|
| 无属性 | body 底部 | 阻塞解析 | HTML 解析暂停时执行 | 文档顺序 | 需要 DOM 就绪 |
| defer | head | 并行下载 | DOM 解析完成后 | 按文档顺序 | 有依赖的脚本 |
| async | head | 并行下载 | 下载完成立即 | 不保证顺序 | 独立脚本 |

**module 类型默认 defer：**

```html
<!-- ES Module 默认行为同 defer -->
<script type="module" src="app.js"></script>

<!-- 可加 async 让其优先执行（但仍比 defer 早） -->
<script type="module" src="app.js" async></script>
```

#### 真实面试题

**题目：script标签为什么建议放在body标签的底部(defer、async)**

**满分答案：**

**核心原因：确保 DOM 解析完毕再执行 JS**

当浏览器解析到 `<script>`（无 defer/async）时，会暂停 HTML 解析，先下载并执行脚本。如果脚本放在 `<head>` 中，此时 DOM 还没构建完，脚本中的 `document.getElementById()` 等 DOM 操作会找不到元素而报错。

**三种解决方案：**

**1. 放在 body 底部（传统方案）**
```html
<body>
    <div id="app"></div>
    <script src="app.js"></script>
</body>
```
- HTML 完全解析后才执行脚本
- DOM 已就绪，不会报错
- 缺点：串行加载，延迟了 JS 的下载开始时间

**2. defer（现代推荐）**
```html
<head>
    <script src="app.js" defer></script>
</head>
```
- JS 与 HTML 并行下载
- 在 HTML 解析完成后、按文档顺序执行
- 兼具性能和不阻塞渲染的优点
- 适合有依赖关系的脚本（main.js → util.js → plugin.js）

**3. async（适合独立脚本）**
```html
<head>
    <script src="analytics.js" async></script>
</head>
```
- JS 与 HTML 并行下载
- 下载完立即执行，不等待 HTML 解析
- 不保证顺序，不适合有依赖的脚本
- 适合：统计脚本、第三方 SDK、广告脚本

**最佳实践：**
- 关键业务 JS（有 DOM 操作）：用 defer，放在 `<head>`
- 第三方独立脚本：用 async，放在 `<head>`
- 内联脚本：放在 `<body>` 底部（确保 DOM 就绪）
- 现代开发：`type="module"` 默认行为同 defer

---

## 1.17 SSG（静态站点生成）

### 题目
说说你对SSG的理解

#### 知识点详解

**SSG（Static Site Generation，静态站点生成）：**

SSG 在构建阶段（build time）生成完整的静态 HTML 文件，部署到 CDN 后用户直接访问预渲染好的页面。

```
构建阶段（Build）：
源码（Markdown/JSON/DB） → SSG 工具 → 静态 HTML 文件
  pages/index.md              →  dist/index.html
  pages/about.md              →  dist/about.html
  posts/hello.md              →  dist/posts/hello/index.html

访问阶段（Runtime）：
用户请求 → CDN → 直接返回预渲染的 HTML
→ 无需服务端计算
→ 首字节时间（TTFB）极短
```

**SSG vs SSR vs CSR：**

| 对比项 | SSG | SSR | CSR |
|--------|-----|-----|-----|
| 渲染时机 | 构建时 | 每次请求时 | 浏览器运行时 |
| 内容新鲜度 | 构建时生成 | 实时 | 实时 |
| 服务器负载 | 极低（静态文件） | 高（每次请求计算） | 极低（纯客户端） |
| 首屏性能 | 最快 | 快 | 慢（需等 JS 执行） |
| SEO | 完美 | 完美 | 需额外处理 |
| 适用内容 | 静态/低频更新 | 动态/高频更新 | 高度交互应用 |
| 重新部署 | 需要（全量/ISR） | 不需要 | 不需要 |

**SSG 实现方案：**

```javascript
// Next.js - getStaticProps（构建时生成）
export async function getStaticProps() {
    const posts = await fetchPosts();  // 从 CMS/DB 获取数据
    return {
        props: { posts },              // 传递给页面组件
        revalidate: 60                 // ISR：60 秒后重新生成
    };
}

// Nuxt.js - generate（Nuxt 3）
export default defineNuxtConfig({
    nitro: {
        prerender: {
            routes: ['/', '/about', '/blog']
        }
    }
});
```

**主流 SSG 框架：**

| 框架 | 特点 |
|------|------|
| Next.js | React 生态，支持 SSG + SSR + ISR |
| Nuxt.js | Vue 生态，Nuxt 3 默认开启 SSG |
| Astro | 多框架混合，可只发送最小 JS |
| Gatsby | React 生态，GraphQL 数据层 |
| Hugo | Go 语言，极快构建速度 |
| VitePress | Vue + Markdown，适合文档 |
| Docusaurus | React，适合技术文档 |

**ISR（增量静态再生）：**

```javascript
// Next.js ISR 示例
export async function getStaticProps() {
    const data = await fetchData();
    return {
        props: { data },
        revalidate: 300  // 5 分钟增量更新一次
    };
}
```
- 首次构建生成静态 HTML
- 后续请求：如果距上次生成超过 5 分钟，触发后台重新生成
- 用户始终拿到的是静态 HTML（快），同时保证数据相对新鲜

**SSG 的优势与劣势：**

**优势：**
- 首屏加载快（静态 HTML + CDN）
- SEO 友好（内容直接可爬取）
- 安全性高（无服务端逻辑，减少攻击面）
- 成本低（静态文件托管，无服务器费用）
- 性能稳定（无服务端负载波动）

**劣势：**
- 构建时间长（大站点可能需要几分钟）
- 内容更新需要重新构建部署
- 不适合完全动态内容（除非结合 ISR/SSR）

#### 真实面试题

**题目：说说你对SSG的理解**

**满分答案：**

**什么是 SSG：**

SSG（Static Site Generation，静态站点生成）是一种前端渲染策略，在**构建阶段**生成完整的静态 HTML 文件，然后部署到 CDN 或静态服务器。用户访问时，服务器直接返回预渲染好的 HTML，无需服务端计算。

**与 SSR/CSR 的对比：**

| 渲染方式 | SSG（静态生成） | SSR（服务端渲染） | CSR（客户端渲染） |
|---------|----------------|-----------------|-----------------|
| 渲染时机 | 构建时 | 每次请求 | 浏览器运行时 |
| 首屏速度 | 最快（静态 HTML） | 快 | 慢（等 JS 执行） |
| SEO | 完美 | 完美 | 需预渲染处理 |
| 服务器负载 | 极低 | 高 | 极低 |
| 内容更新 | 需重新构建 | 实时 | 实时 |

**适用场景：**

SSG 非常适合内容**不频繁变化**的站点：
- 博客、个人网站
- 文档站点（VitePress、Docusaurus）
- 营销页、产品页
- 企业官网
- 文档型知识库

**主流实现方案：**

- **Next.js**：`getStaticProps` + `revalidate`（支持 ISR）
- **Nuxt.js**：默认 SSG，支持 ISR
- **Astro**：只发送必要 JS，性能极佳
- **VitePress**：Vue 文档站首选
- **Gatsby**：React 生态，GraphQL 数据层

**SSG + ISR 的结合：**

传统 SSG 更新内容需要全站重新构建，ISR（增量静态再生）解决了这个问题：
- 数据有更新时，后台重新生成对应页面
- 用户访问时立即拿到新内容（快）
- 保证数据新鲜度（增量更新）

**优缺点总结：**

- ✅ 首屏快、SEO 好、安全性高、部署简单
- ❌ 构建时间长、全量更新需重新构建
- 💡 实际项目中通常 SSG + ISR 组合使用，兼顾性能和内容更新

---

## 1.18 HTML5及与HTML的区别

### 题目
什么是HTML5，以及和HTML的区别是什么?

#### 知识点详解

**HTML5 是什么：**

HTML5 是 HTML 的第 5 个主要版本，2014 年由 W3C 正式推荐。带来了大量新特性和 API，从根本上改变了 Web 应用开发方式。

**核心区别：**

| 对比项 | HTML4 / XHTML | HTML5 |
|--------|--------------|-------|
| DOCTYPE | `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "...">` | `<!DOCTYPE html>` |
| 字符编码 | `<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">` | `<meta charset="UTF-8">` |
| 语义 | 无语义标签，大量 `<div>` | 新增语义化标签 |
| 多媒体 | 依赖 Flash/插件 | 原生 `<video>`、`<audio>` |
| 绘图 | 无 | `<canvas>`、SVG |
| 表单 | 基础输入类型 | 新增 date/email/range/color 等 |
| 存储 | Cookie | localStorage/sessionStorage/WebSQL |
| 离线 | 不支持 | Service Worker/Manifest |
| 通信 | 无 | WebSocket |
| 线程 | 无 | Web Workers |
| API 丰富度 | 极少 | 极大（地理定位、拖放、文件系统等） |

**HTML5 新特性分类：**

**1. 语义化标签**
```html
<header>、<nav>、<main>、<article>、<section>、<aside>、<footer>
<figure>、<figcaption>、<time>、<mark>、<details>、<summary>
```

**2. 多媒体**
```html
<video src="video.mp4" controls autoplay muted></video>
<audio src="music.mp3" controls autoplay></audio>
```

**3. 绘图能力**
```html
<canvas id="draw" width="500" height="300"></canvas>

<!-- SVG 矢量图 -->
<svg width="100" height="100">
    <circle cx="50" cy="50" r="40" fill="red"/>
</svg>
```

**4. 表单增强**
```html
<input type="email">       <!-- 邮箱验证 -->
<input type="date">        <!-- 日期选择 -->
<input type="range">       <!-- 滑块 -->
<input type="color">       <!-- 颜色选择 -->
<input type="number">      <!-- 数字输入 -->
<input type="tel">         <!-- 电话号码 -->
<input type="url">         <!-- URL 验证 -->
<input type="search">      <!-- 搜索框 -->

<!-- 新属性 -->
<input placeholder="请输入">      <!-- 占位符 -->
<input required>                     <!-- 必填 -->
<input autofocus>                    <!-- 自动聚焦 -->
<input pattern="[A-Za-z]+">          <!-- 正则验证 -->
```

**5. 本地存储**
```javascript
localStorage.setItem('key', 'value');   // 持久化存储
sessionStorage.setItem('token', 'abc'); // 会话级存储
```

**6. 离线应用（PWA）**
```html
<!-- manifest.json 链接 -->
<link rel="manifest" href="/manifest.json">

<!-- Service Worker 注册 -->
<script>
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js');
}
</script>
```

**7. Web Workers（JS 多线程）**
```javascript
const worker = new Worker('worker.js');
worker.postMessage({ data: [1, 2, 3] });
worker.onmessage = e => console.log(e.data);
```

**8. WebSocket（双向通信）**
```javascript
const ws = new WebSocket('wss://example.com/socket');
ws.onopen = () => ws.send('hello');
ws.onmessage = e => console.log(e.data);
```

**9. 其他新 API**

- **Geolocation API**：`navigator.geolocation.getCurrentPosition()`
- **Drag and Drop API**：原生拖拽实现
- **Notification API**：系统通知
- **Fullscreen API**：全屏模式
- **IntersectionObserver**：懒加载、无限滚动
- **History API**：`history.pushState`、`history.replaceState`

**HTML5 移除的元素：**

纯表现类和已被新特性替代的元素被移除：
- `<font>`、`<center>`、`<big>`、`<strike>`、`<tt>`
- `<frame>`、`<frameset>`、`<noframes>`
- `<applet>`（被 `<object>` 替代）

#### 真实面试题

**题目：什么是HTML5，以及和HTML的区别是什么?**

**满分答案：**

**什么是 HTML5：**

HTML5 是 HTML（超文本标记语言）的第 5 个主要版本，2014 年由 W3C 正式推荐。相比 HTML4/XHTML，HTML5 不仅仅是标签的升级，而是**从标记语言到应用平台的重大演进**，引入了大量原生 API，让 Web 应用具备接近原生 App 的能力。

**与 HTML4 的核心区别：**

**1. DOCTYPE 和语法简化**
```html
<!-- HTML4：复杂声明 -->
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

<!-- HTML5：极简 -->
<!DOCTYPE html>
```

**2. 语义化标签**
```html
<!-- HTML4：大量无语义 div -->
<div class="header">
    <div class="nav">...</div>
</div>

<!-- HTML5：语义清晰 -->
<header><nav>...</nav></header>
```

**3. 多媒体原生支持**
```html
<!-- HTML4：需要 Flash 插件 -->
<object classid="clsid:...">...</object>

<!-- HTML5：原生标签 -->
<video src="video.mp4" controls></video>
<audio src="music.mp3" controls></audio>
```

**4. 表单能力增强**
```html
<!-- HTML4：只有 text/password/checkbox/radio -->
<!-- HTML5：email/date/range/color/tel/search/number/url -->
<input type="email" required placeholder="请输入邮箱">
<input type="date" min="2024-01-01">
```

**5. 本地存储**
```javascript
// HTML4：只有 Cookie（4KB，每次请求携带）
// HTML5：localStorage（5MB）、sessionStorage（5MB）
localStorage.setItem('user', JSON.stringify(user));
```

**6. 离线与通信**
- HTML5：Service Worker、Manifest（支持 PWA 离线应用）、WebSocket
- HTML4：不支持离线，无法原生实现实时通信

**7. 绘图能力**
- HTML5：`<canvas>` 2D/3D 绑图、SVG 矢量图
- HTML4：需要 Flash/第三方插件

**8. Web Workers**
- HTML5：JavaScript 多线程，后台执行耗时任务
- HTML4：不支持

**9. 被移除的元素**
- HTML5 移除了纯表现类标签：`<font>`、`<center>`、`<big>`、`<frame>`
- 强制语义化：用 CSS 控制样式，用语义标签表达含义

**总结：HTML5 的本质变化是——从内容标记语言，升级为包含语义、多媒体、离线、通信、绘图的完整应用开发平台。**

---

## 1.19 渐进增强与优雅降级

### 题目
什么是渐进增强和优雅降级?

#### 知识点详解

**渐进增强（Progressive Enhancement）：**

核心思路：先为所有用户提供最基础的功能体验，再逐步为支持更多特性的浏览器增强体验。

```
用户浏览器能力
    高 ─ ─ ─ ─ ─ ─ ─ ─ ┬─ 完整功能（最新 CSS/JS 特性）
                       │
    中 ─ ─ ─ ─ ─ ─ ─ ─ ┼─ 增强功能（部分新特性）
                       │
    低 ─ ─ ─ ─ ─ ─ ─ ─ ┴─ 基础功能（HTML + 基本 CSS）
```

```html
<!-- 渐进增强示例 -->
<!-- 所有浏览器：基本功能可用 -->
<button>提交</button>

<!-- 支持时：增强样式（CSS 特性检测） -->
<style>
    button {
        padding: 10px 20px;  /* 所有浏览器 */
    }

    @supports (backdrop-filter: blur(10px)) {
        /* 仅支持 backdrop-filter 的浏览器才应用 */
        button {
            backdrop-filter: blur(10px);
            background: rgba(0,0,0,0.5);
        }
    }

    @supports not (backdrop-filter: blur(10px)) {
        /* 不支持时：降级样式 */
        button {
            background: rgba(0,0,0,0.8);
        }
    }
</style>
```

**优雅降级（Graceful Degradation）：**

核心思路：先为最新浏览器构建完整功能，再为不支持新特性的旧浏览器提供降级方案。

```javascript
// 优雅降级示例：fetch API
// 新浏览器：使用 fetch
// 旧浏览器：降级使用 XMLHttpRequest

function fetchData(url) {
    if ('fetch' in window) {
        // 现代浏览器
        return fetch(url).then(r => r.json());
    } else {
        // 旧浏览器降级
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            xhr.open('GET', url);
            xhr.onload = () => resolve(JSON.parse(xhr.responseText));
            xhr.onerror = reject;
            xhr.send();
        });
    }
}
```

**CSS 中的优雅降级：**

```css
/* 先写基础样式（所有浏览器支持） */
.container {
    background: black;
    color: white;
}

/* 再增强（新浏览器支持时应用） */
@supports (backdrop-filter: blur(10px)) {
    .container {
        background: rgba(0, 0, 0, 0.5);
        backdrop-filter: blur(10px);
    }
}
```

**JavaScript 特性检测：**

```javascript
// 检测特性而非检测浏览器
if ('IntersectionObserver' in window) {
    // 使用 IntersectionObserver
} else {
    // 降级方案：滚动监听
}

// 检测 API 返回值
if (document.createElement('canvas').getContext('2d')) {
    // Canvas 可用
} else {
    // 显示占位图
}
```

**HTML 中的渐进增强：**

```html
<!-- 所有浏览器：链接可用 -->
<a href="/page">跳转</a>

<!-- 现代浏览器：增强体验 -->
<a href="/page" class="js-popup">跳转</a>

<!-- JS 处理后：SPA 行为（无刷新） -->
<!-- JS 不可用：正常跳转（降级） -->
```

**两者对比：**

| 对比项 | 渐进增强 | 优雅降级 |
|--------|---------|---------|
| 出发点 | 从低版本/基础出发向上构建 | 从高版本/完整出发向下兼容 |
| 关注对象 | 低版本浏览器体验 | 高版本新特性 |
| CSS 策略 | `@supports (feature)` | `@supports not (feature)` |
| JS 策略 | 特性检测存在才执行 | try-catch 捕获错误 |
| 开发顺序 | 先基础 → 再增强 | 先完整 → 再降级 |
| 用户体验 | 低版本也能用 | 高版本体验更好 |

**实际开发中的最佳实践：**

通常两者结合使用：
```css
/* 1. 基础样式（所有浏览器） */
.gradient-box {
    background: linear-gradient(red, blue); /* 旧浏览器会解析失败 */
}

/* 2. 安全兜底 */
.gradient-box {
    background: red; /* 降级色 */
    background: linear-gradient(red, blue); /* 现代浏览器 */
}

/* 3. 渐进增强：仅在支持时才额外处理 */
@supports (background: linear-gradient(red, blue)) {
    .gradient-box {
        border-radius: 8px;
    }
}
```

#### 真实面试题

**题目：什么是渐进增强和优雅降级?**

**满分答案：**

**渐进增强（Progressive Enhancement）：**

核心思想是**从底层构建向上增强**——先确保所有用户（无论浏览器新旧）都能获得可用的基础体验，再逐步为支持新特性的浏览器添加增强功能。

- 开发顺序：先做基础版本 → 再做增强版本
- 关注点：低版本浏览器
- 策略：用 `@supports` 检测新特性支持后，才应用增强样式
- 例子：先确保按钮在任何浏览器都能点击，再用 CSS 特性增强样式

**优雅降级（Graceful Degradation）：**

核心思想是**从顶层向下兼容**——先为最新浏览器构建完整功能，再为旧浏览器提供降级方案。

- 开发顺序：先做完整版本 → 再做降级版本
- 关注点：高版本浏览器
- 策略：先写高级语法，不支持时降级处理
- 例子：先使用 Canvas 绘图，不支持时用静态图替代

**两者的核心区别：**

| | 渐进增强 | 优雅降级 |
|--|---------|---------|
| 出发方向 | 低端 → 高端 | 高端 → 低端 |
| 关注对象 | 低版本浏览器体验 | 高版本新特性 |
| CSS 写法 | `@supports (feature)` | `@supports not (feature)` |
| JS 写法 | `if ('API' in window)` | `try { newAPI() } catch {}` |

**实际开发中的建议：**

两者常结合使用，核心原则是：
1. **不依赖用户浏览器能力**：始终提供兜底方案
2. **用特性检测而非浏览器检测**：`if ('fetch' in window)` 而非 `if (IE)`
3. **CSS 降级写法**：`background: #fff;`（降级）→ `background: linear-gradient(...)`（增强）

---

## 1.20 Node与Element的关系

### 题目
Node和Element是什么关系?

#### 知识点详解

**继承关系：**

```
Node（基类，DOM Level 1）
  └── CharacterData
        ├── Text（文本节点）
        └── Comment（注释节点）
  └── Document
        └── HTMLDocument
  └── DocumentFragment
  └── Element（DOM Level 1）
        └── HTMLElement
              ├── HTMLDivElement
              ├── HTMLSpanElement
              ├── HTMLInputElement
              └── ...（各元素类型）
```

**Node 的 12 种类型（nodeType）：**

| nodeType | 值 | 节点类型 | 继承自 |
|----------|----|---------|--------|
| ELEMENT_NODE | 1 | 元素节点 | Element |
| TEXT_NODE | 3 | 文本节点 | CharacterData |
| COMMENT_NODE | 8 | 注释节点 | CharacterData |
| DOCUMENT_NODE | 9 | document 节点 | Node |
| DOCUMENT_TYPE_NODE | 10 | DOCTYPE 节点 | Node |
| DOCUMENT_FRAGMENT_NODE | 11 | 文档片段 | Node |
| ATTRIBUTE_NODE | 2 | 属性节点（已废弃） | — |
| CDATA_SECTION_NODE | 4 | CDATA 段落 | — |
| ENTITY_REFERENCE_NODE | 5 | 实体引用（已废弃） | — |
| ENTITY_NODE | 6 | 实体（已废弃） | — |
| PROCESSING_INSTRUCTION_NODE | 7 | 处理指令 | — |
| NOTATION_NODE | 12 | 符号（已废弃） | — |

**Element 是 Node 的子集：**

- Node 是所有节点的基类（12 种类型）
- Element 是 Node 的子集（仅指元素节点，nodeType === 1）
- 元素节点是那些对应 HTML 标签的节点

**childNodes vs children：**

```javascript
const container = document.querySelector('.container');

console.log(container.childNodes);
// NodeList(5) [text, div, text, div, text]
// → 包含所有类型的节点：元素 + 文本 + 注释

console.log(container.children);
// HTMLCollection(2) [div, div]
// → 只包含元素节点（Element）
```

**firstChild vs firstElementChild：**

```javascript
<div class="container">
    <div>第一个子元素</div>
    <div>第二个子元素</div>
</div>

container.firstChild;
// → text（空白文本节点！）
// → 因为 <div> 和 <div> 之间的换行被解析为文本节点

container.firstElementChild;
// → <div>第一个子元素</div>
// → 第一个元素节点（跳过空白）

container.lastElementChild;
// → <div>第二个子元素</div>
```

**parentNode vs parentElement：**

```javascript
// 大多数// 大多数情况下两者相等
console.log(el.parentNode === el.parentElement);  // true（元素节点的父节点也是元素）

// 唯一区别：document 的 parentNode 是 #document（Node），parentElement 是 null
console.log(document.documentElement.parentNode);   // #document
console.log(document.documentElement.parentElement); // null

// 获取最近的父元素节点（跳过非 Element 节点）
function closestElement(el, selector) {
    let current = el;
    while (current) {
        if (current.matches(selector)) return current;
        current = current.parentElement;
    }
    return null;
}
```

**实际开发注意事项：**

```javascript
// ❌ 错误：firstChild 可能返回空白文本节点
const first = container.firstChild;
if (first.nodeType === 3) { /* 处理空白节点 */ }

// ✅ 正确：使用 firstElementChild 获取第一个元素
const first = container.firstElementChild;

// ❌ 错误：childNodes 包含文本节点
container.childNodes.length;  // 可能很大，包含空白文本

// ✅ 正确：children 只包含元素
container.children.length;    // 准确的子元素数量

// 遍历所有子元素
Array.from(el.children).forEach(child => {
    console.log(child.tagName);
});

// 过滤出元素节点
Array.from(el.childNodes).filter(node => node.nodeType === 1);
```

#### 真实面试题

**题目：Node和Element是什么关系?**

**满分答案：**

**继承关系：**

Node 是 DOM 树中所有节点的**基类**（基类型），Element 继承自 Node。简单说：**Element 是 Node 的一种（元素节点），但 Node 不一定是 Element。**

```
Node（基类，12 种类型）
  ├── Document
  ├── DocumentFragment
  ├── CharacterData（Text、Comment）
  └── Element ← HTML 元素的类型
        └── HTMLElement（HTMLDivElement、HTMLSpanElement...）
```

**Node 的 12 种类型中，常用的有：**

| nodeType | 值 | 节点类型 | 是否 Element |
|----------|----|---------|------------|
| ELEMENT_NODE | 1 | 元素节点 | ✅ |
| TEXT_NODE | 3 | 文本节点 | ❌ |
| COMMENT_NODE | 8 | 注释节点 | ❌ |
| DOCUMENT_NODE | 9 | document | ❌ |
| DOCUMENT_FRAGMENT_NODE | 11 | 文档片段 | ❌ |

**childNodes vs children 的区别：**

```javascript
// childNodes：NodeList，包含所有类型的子节点（元素 + 文本 + 注释）
container.childNodes;  // [text, div, text, div, text, comment, div, text]

// children：HTMLCollection，只包含子元素节点（Element）
container.children;    // [div, div, div]
```

**firstChild vs firstElementChild：**

```javascript
// firstChild：第一个子节点（可能是空白文本节点！）
el.firstChild;  // 换行符被解析为 #text 节点

// firstElementChild：第一个子元素节点（跳过空白）
el.firstElementChild;  // 直接返回第一个 HTML 元素
```

**parentNode vs parentElement：**

- 大多数情况下两者相等（父节点也是元素）
- 唯一区别：`document.documentElement.parentNode` 返回 `#document`，`parentElement` 返回 `null`

**开发建议：**

- 遍历子元素时，优先使用 `children` 和 `firstElementChild`
- 使用 `classList`、`querySelector` 等 Element 级别的方法
- 注意空白文本节点：HTML 标签之间的换行会被解析为 `#text` 节点
