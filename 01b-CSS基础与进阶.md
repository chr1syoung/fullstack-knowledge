# 一（续）、CSS 基础与进阶

---

## 1.2 CSS 基础

### 1.2.1 选择器

#### 知识点详解

**基础选择器：**

```css
/* 元素选择器 */
p { color: red; }

/* 类选择器 */
.container { width: 100%; }

/* ID 选择器 */
#header { background: blue; }

/* 通配符 */
* { box-sizing: border-box; }

/* 属性选择器 */
[type="text"] { border: 1px solid #ccc; }
[href^="https"] { color: green; }  /* 以 https 开头 */
[href$=".pdf"] { color: red; }     /* 以 .pdf 结尾 */
[class*="btn"] { cursor: pointer; } /* 包含 btn */
[lang|="en"] { font-family: serif; } /* en 或 en-US */
[class~="active"] { font-weight: bold; } /* 空格分隔的词 */
```

**组合选择器：**

```css
/* 后代选择器（空格） */
.parent .child { color: red; }

/* 子选择器（>） */
.parent > .child { color: blue; }

/* 相邻兄弟选择器（+） */
h1 + p { margin-top: 0; }

/* 通用兄弟选择器（~） */
h1 ~ p { color: gray; }

/* 多选择器（,） */
h1, h2, h3 { font-weight: bold; }
```

**伪类选择器：**

```css
/* 链接状态 */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }

/* 结构伪类 */
li:first-child { font-weight: bold; }
li:last-child { border-bottom: none; }
li:nth-child(2) { color: red; }
li:nth-child(odd) { background: #f5f5f5; }
li:nth-child(even) { background: white; }
li:nth-child(3n+1) { color: blue; }
li:nth-last-child(2) { color: green; }
p:first-of-type { font-size: 1.2em; }
p:last-of-type { margin-bottom: 0; }
p:only-child { text-align: center; }
p:only-of-type { font-style: italic; }

/* 表单伪类 */
input:focus { outline: 2px solid blue; }
input:disabled { opacity: 0.5; }
input:enabled { cursor: text; }
input:checked { accent-color: blue; }
input:required { border-color: red; }
input:optional { border-color: gray; }
input:valid { border-color: green; }
input:invalid { border-color: red; }
input:placeholder-shown { font-style: italic; }

/* 其他 */
:not(.active) { opacity: 0.5; }
:is(h1, h2, h3) { font-weight: bold; }
:where(h1, h2, h3) { font-weight: bold; }  /* 优先级为 0 */
:has(img) { border: 1px solid red; }
:empty { display: none; }
:root { --primary: blue; }
```

**伪元素选择器：**

```css
/* 内容前后 */
p::before {
    content: '→ ';
    color: red;
}

p::after {
    content: ' ←';
    color: blue;
}

/* 首字母/首行 */
p::first-letter {
    font-size: 2em;
    float: left;
}

p::first-line {
    font-weight: bold;
}

/* 选中文本 */
::selection {
    background: yellow;
    color: black;
}

/* 占位符 */
input::placeholder {
    color: #999;
    font-style: italic;
}

/* 滚动条 */
::-webkit-scrollbar {
    width: 8px;
}
::-webkit-scrollbar-track {
    background: #f1f1f1;
}
::-webkit-scrollbar-thumb {
    background: #888;
    border-radius: 4px;
}
```

### 1.2.2 优先级与层叠

**优先级计算：**

```
优先级从高到低：
1. !important（最高）
2. 内联样式（style=""）：1000
3. ID 选择器（#id）：100
4. 类选择器（.class）、属性选择器、伪类：10
5. 元素选择器、伪元素：1
6. 通配符（*）、组合符：0

计算示例：
#nav .item:hover a  →  100 + 10 + 10 + 1 = 121
.container > p      →  10 + 1 = 11
```

```css
/* !important 覆盖所有 */
.text { color: red !important; }

/* 相同优先级，后面的覆盖前面的 */
.text { color: red; }
.text { color: blue; }  /* 最终是蓝色 */

/* 继承的样式优先级最低 */
body { color: red; }
p { color: inherit; }  /* 继承 body 的颜色 */
```

### 1.2.3 盒模型

```css
/* 标准盒模型（默认） */
/* width = content width */
/* 实际占用宽度 = width + padding + border + margin */
.box {
    box-sizing: content-box;
    width: 200px;
    padding: 20px;
    border: 2px solid black;
    /* 实际宽度 = 200 + 40 + 4 = 244px */
}

/* IE 盒模型（border-box） */
/* width = content + padding + border */
/* 实际占用宽度 = width + margin */
.box {
    box-sizing: border-box;
    width: 200px;
    padding: 20px;
    border: 2px solid black;
    /* 实际宽度 = 200px（padding 和 border 包含在内） */
}

/* 全局设置 border-box（推荐） */
*, *::before, *::after {
    box-sizing: border-box;
}
```

**margin 合并（塌陷）：**

```css
/* 相邻兄弟元素的 margin 合并 */
.box1 { margin-bottom: 20px; }
.box2 { margin-top: 30px; }
/* 实际间距 = max(20, 30) = 30px */

/* 父子元素 margin 合并 */
.parent { margin-top: 20px; }
.child { margin-top: 30px; }
/* 父元素实际 margin-top = max(20, 30) = 30px */

/* 解决方案 */
.parent {
    overflow: hidden;  /* 创建 BFC */
    /* 或 */
    padding-top: 1px;
    /* 或 */
    border-top: 1px solid transparent;
}
```

#### 真实面试题

**题目：谈谈你对 CSS 盒模型的理解，以及 box-sizing 的作用。**

**满分答案：**

**盒模型两种模式：**

1. **标准盒模型（content-box）**
   - `width` = 内容宽度
   - 实际占用宽度 = width + padding + border + margin

2. **IE 盒模型（border-box）**
   - `width` = 内容 + padding + border
   - 实际占用宽度 = width + margin

**box-sizing 作用：**
- `content-box`（默认）：按标准盒模型计算
- `border-box`：按 IE 盒模型计算，更符合直觉

**最佳实践：**
```css
*, *::before, *::after {
    box-sizing: border-box;
}
```
全局设置 `border-box`，避免计算混乱。

**Margin 合并问题：**
- 相邻兄弟元素：取较大值
- 父子元素：子元素 margin 会"穿透"父元素
- 解决：父元素创建 BFC（overflow: hidden、padding、border 等）
```

### 1.2.4 Flexbox 布局

```css
/* 容器属性 */
.container {
    display: flex;
    
    /* 主轴方向 */
    flex-direction: row;           /* 默认，水平从左到右 */
    flex-direction: row-reverse;   /* 水平从右到左 */
    flex-direction: column;        /* 垂直从上到下 */
    flex-direction: column-reverse; /* 垂直从下到上 */
    
    /* 换行 */
    flex-wrap: nowrap;    /* 默认，不换行 */
    flex-wrap: wrap;      /* 换行 */
    flex-wrap: wrap-reverse; /* 反向换行 */
    
    /* 简写 */
    flex-flow: row wrap;
    
    /* 主轴对齐 */
    justify-content: flex-start;    /* 默认，左对齐 */
    justify-content: flex-end;      /* 右对齐 */
    justify-content: center;        /* 居中 */
    justify-content: space-between; /* 两端对齐 */
    justify-content: space-around;  /* 均匀分布（两端有间距） */
    justify-content: space-evenly;  /* 均匀分布（两端间距相等） */
    
    /* 交叉轴对齐 */
    align-items: stretch;     /* 默认，拉伸填满 */
    align-items: flex-start;  /* 顶部对齐 */
    align-items: flex-end;    /* 底部对齐 */
    align-items: center;      /* 居中 */
    align-items: baseline;    /* 基线对齐 */
    
    /* 多行对齐 */
    align-content: flex-start;
    align-content: flex-end;
    align-content: center;
    align-content: space-between;
    align-content: space-around;
    align-content: stretch;
    
    /* 间距（现代浏览器支持） */
    gap: 10px;
    row-gap: 10px;
    column-gap: 20px;
}

/* 子项属性 */
.item {
    /* 排列顺序 */
    order: 0;  /* 默认，数值越小越靠前 */
    
    /* 放大比例 */
    flex-grow: 0;  /* 默认，不放大 */
    flex-grow: 1;  /* 占据剩余空间 */
    
    /* 缩小比例 */
    flex-shrink: 1;  /* 默认，等比缩小 */
    flex-shrink: 0;  /* 不缩小 */
    
    /* 基础大小 */
    flex-basis: auto;  /* 默认，根据内容 */
    flex-basis: 200px;
    
    /* 简写 */
    flex: 1;           /* flex: 1 1 0% */
    flex: auto;        /* flex: 1 1 auto */
    flex: none;        /* flex: 0 0 auto */
    flex: 0 1 200px;   /* grow shrink basis */
    
    /* 单独对齐 */
    align-self: auto;
    align-self: flex-start;
    align-self: flex-end;
    align-self: center;
    align-self: stretch;
}
```

**Flexbox 常见布局：**

```css
/* 水平垂直居中 */
.center {
    display: flex;
    justify-content: center;
    align-items: center;
}

/* 导航栏 */
.nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

/* 圣杯布局 */
.layout {
    display: flex;
    min-height: 100vh;
    flex-direction: column;
}
.header, .footer { flex-shrink: 0; }
.main {
    flex: 1;
    display: flex;
}
.sidebar { width: 200px; flex-shrink: 0; }
.content { flex: 1; }

/* 等高列 */
.columns {
    display: flex;
    align-items: stretch;  /* 默认 */
}
```

### 1.2.5 Grid 布局

```css
/* 容器属性 */
.container {
    display: grid;
    
    /* 定义列 */
    grid-template-columns: 200px 1fr 200px;
    grid-template-columns: repeat(3, 1fr);
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    
    /* 定义行 */
    grid-template-rows: 100px auto 100px;
    grid-template-rows: repeat(3, 100px);
    
    /* 间距 */
    gap: 20px;
    row-gap: 10px;
    column-gap: 20px;
    
    /* 对齐 */
    justify-items: start | end | center | stretch;
    align-items: start | end | center | stretch;
    place-items: center;  /* 简写 */
    
    /* 整体对齐 */
    justify-content: start | end | center | space-between | space-around;
    align-content: start | end | center | space-between | space-around;
    
    /* 命名区域 */
    grid-template-areas:
        "header header header"
        "sidebar main main"
        "footer footer footer";
}

/* 子项属性 */
.item {
    /* 列位置 */
    grid-column: 1 / 3;      /* 从第1列到第3列 */
    grid-column: 1 / span 2; /* 从第1列跨2列 */
    grid-column: 1;           /* 第1列 */
    
    /* 行位置 */
    grid-row: 1 / 3;
    grid-row: 1 / span 2;
    
    /* 命名区域 */
    grid-area: header;
    
    /* 对齐 */
    justify-self: start | end | center | stretch;
    align-self: start | end | center | stretch;
    place-self: center;
}
```

**Grid 常见布局：**

```css
/* 圣杯布局 */
.layout {
    display: grid;
    grid-template-areas:
        "header header header"
        "sidebar main aside"
        "footer footer footer";
    grid-template-columns: 200px 1fr 200px;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }

/* 响应式卡片网格 */
.cards {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
}
```

### 1.2.6 响应式布局

```css
/* 媒体查询 */
/* 移动优先（推荐） */
.container {
    width: 100%;
    padding: 0 16px;
}

@media (min-width: 768px) {
    .container {
        max-width: 768px;
        margin: 0 auto;
    }
}

@media (min-width: 1024px) {
    .container {
        max-width: 1024px;
    }
}

@media (min-width: 1280px) {
    .container {
        max-width: 1280px;
    }
}

/* 常用断点 */
/* xs: < 576px */
/* sm: >= 576px */
/* md: >= 768px */
/* lg: >= 992px */
/* xl: >= 1200px */
/* xxl: >= 1400px */

/* 媒体查询类型 */
@media screen { }  /* 屏幕 */
@media print { }   /* 打印 */
@media (prefers-color-scheme: dark) { }  /* 暗色模式 */
@media (prefers-reduced-motion: reduce) { }  /* 减少动画 */
@media (orientation: landscape) { }  /* 横屏 */
@media (hover: hover) { }  /* 支持 hover */

/* 相对单位 */
/* rem：相对于根元素字体大小 */
/* em：相对于当前元素字体大小 */
/* vw：视口宽度的 1% */
/* vh：视口高度的 1% */
/* vmin：vw 和 vh 中较小的 */
/* vmax：vw 和 vh 中较大的 */
/* %：相对于父元素 */
/* ch：字符 0 的宽度 */
/* ex：字符 x 的高度 */
```

---

## 1.3 CSS 进阶

### 1.3.1 CSS 变量

```css
/* 定义变量 */
:root {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --font-size-base: 16px;
    --border-radius: 4px;
    --spacing-unit: 8px;
}

/* 使用变量 */
.button {
    background-color: var(--primary-color);
    font-size: var(--font-size-base);
    border-radius: var(--border-radius);
    padding: calc(var(--spacing-unit) * 1) calc(var(--spacing-unit) * 2);
}

/* 带默认值 */
.element {
    color: var(--text-color, black);  /* 如果 --text-color 未定义，使用 black */
}

/* 主题切换 */
[data-theme="dark"] {
    --primary-color: #4dabf7;
    --background: #1a1a1a;
    --text-color: #ffffff;
}

/* JavaScript 操作 CSS 变量 */
document.documentElement.style.setProperty('--primary-color', '#ff0000');
const value = getComputedStyle(document.documentElement).getPropertyValue('--primary-color');
```

### 1.3.2 动画与过渡

**transition（过渡）：**

```css
/* 基本语法 */
.element {
    transition: property duration timing-function delay;
    
    /* 单个属性 */
    transition: color 0.3s ease;
    
    /* 多个属性 */
    transition: color 0.3s ease, transform 0.5s ease-in-out;
    
    /* 所有属性 */
    transition: all 0.3s ease;
}

/* 时间函数 */
transition-timing-function: ease;        /* 默认，慢-快-慢 */
transition-timing-function: linear;      /* 匀速 */
transition-timing-function: ease-in;     /* 慢-快 */
transition-timing-function: ease-out;    /* 快-慢 */
transition-timing-function: ease-in-out; /* 慢-快-慢 */
transition-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1);
transition-timing-function: steps(4, end);  /* 步进 */

/* 示例：按钮悬停效果 */
.button {
    background: blue;
    transform: scale(1);
    transition: background 0.3s, transform 0.2s;
}
.button:hover {
    background: darkblue;
    transform: scale(1.05);
}
```

**animation（动画）：**

```css
/* 定义关键帧 */
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes slideIn {
    0% {
        transform: translateX(-100%);
        opacity: 0;
    }
    100% {
        transform: translateX(0);
        opacity: 1;
    }
}

@keyframes bounce {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-20px); }
}

/* 使用动画 */
.element {
    animation: fadeIn 1s ease forwards;
    
    /* 详细属性 */
    animation-name: fadeIn;
    animation-duration: 1s;
    animation-timing-function: ease;
    animation-delay: 0.5s;
    animation-iteration-count: 1;  /* 或 infinite */
    animation-direction: normal;   /* reverse, alternate, alternate-reverse */
    animation-fill-mode: forwards; /* none, backwards, both */
    animation-play-state: running; /* paused */
}

/* 多个动画 */
.element {
    animation: fadeIn 1s ease, slideIn 0.5s ease;
}

/* 性能优化：使用 transform 和 opacity */
/* 这两个属性不触发重排，只触发合成 */
@keyframes optimized {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(100px); opacity: 0; }
}
```

**transform（变换）：**

```css
.element {
    /* 位移 */
    transform: translate(50px, 100px);
    transform: translateX(50px);
    transform: translateY(100px);
    transform: translateZ(50px);  /* 3D */
    transform: translate3d(50px, 100px, 50px);
    
    /* 缩放 */
    transform: scale(1.5);
    transform: scaleX(1.5);
    transform: scaleY(0.5);
    transform: scale3d(1.5, 0.5, 1);
    
    /* 旋转 */
    transform: rotate(45deg);
    transform: rotateX(45deg);
    transform: rotateY(45deg);
    transform: rotateZ(45deg);
    
    /* 倾斜 */
    transform: skew(10deg, 20deg);
    transform: skewX(10deg);
    transform: skewY(20deg);
    
    /* 组合（从右到左执行） */
    transform: translate(50px, 50px) rotate(45deg) scale(1.5);
    
    /* 变换原点 */
    transform-origin: center;
    transform-origin: top left;
    transform-origin: 50% 50%;
    
    /* 3D 透视 */
    perspective: 1000px;
    transform-style: preserve-3d;
    backface-visibility: hidden;
}
```

### 1.3.3 层叠上下文与 z-index

```css
/* 创建层叠上下文的方式 */
.stacking-context {
    /* 1. position 不为 static 且 z-index 不为 auto */
    position: relative;
    z-index: 1;
    
    /* 2. opacity < 1 */
    opacity: 0.9;
    
    /* 3. transform 不为 none */
    transform: translateZ(0);
    
    /* 4. filter 不为 none */
    filter: blur(0);
    
    /* 5. will-change */
    will-change: transform;
    
    /* 6. isolation: isolate */
    isolation: isolate;
    
    /* 7. flex/grid 子项且 z-index 不为 auto */
    /* 8. mix-blend-mode 不为 normal */
}

/* z-index 只在同一层叠上下文中比较 */
.parent1 { position: relative; z-index: 1; }
.child1 { position: relative; z-index: 100; }

.parent2 { position: relative; z-index: 2; }
.child2 { position: relative; z-index: 1; }

/* child2 在 child1 上面，因为 parent2 的 z-index 更高 */
```

### 1.3.4 CSS 性能优化

```css
/* 1. 使用 will-change 提示浏览器 */
.animated {
    will-change: transform, opacity;
}
/* 注意：不要过度使用，会消耗内存 */

/* 2. 使用 contain 限制重排范围 */
.widget {
    contain: layout;  /* 内部布局变化不影响外部 */
    contain: paint;   /* 内部绘制不影响外部 */
    contain: strict;  /* 最严格的隔离 */
    contain: content; /* layout + paint */
}

/* 3. 使用 GPU 加速 */
.gpu-accelerated {
    transform: translateZ(0);
    /* 或 */
    will-change: transform;
}

/* 4. 避免触发重排的属性 */
/* 触发重排（Layout）：width, height, margin, padding, border, top, left, font-size... */
/* 触发重绘（Paint）：color, background, visibility, box-shadow... */
/* 只触发合成（Composite）：transform, opacity */

/* 5. 减少选择器复杂度 */
/* 不好 */
.nav ul li a:hover { }

/* 好 */
.nav-link:hover { }

/* 6. 避免使用 @import */
/* 不好 */
@import url('other.css');

/* 好：使用 <link> 标签并行加载 */
```

---

## 1.4 CSS 框架

### 1.4.1 Tailwind CSS

```html
<!-- 基本用法 -->
<div class="flex items-center justify-between p-4 bg-white shadow-md rounded-lg">
    <h1 class="text-2xl font-bold text-gray-800">标题</h1>
    <button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors">
        按钮
    </button>
</div>

<!-- 响应式 -->
<div class="w-full md:w-1/2 lg:w-1/3">
    <!-- 移动端全宽，平板半宽，桌面三分之一 -->
</div>

<!-- 暗色模式 -->
<div class="bg-white dark:bg-gray-800 text-black dark:text-white">
    内容
</div>

<!-- 自定义配置 -->
<!-- tailwind.config.js -->
module.exports = {
    theme: {
        extend: {
            colors: {
                primary: '#007bff',
                secondary: '#6c757d'
            },
            fontFamily: {
                sans: ['Inter', 'sans-serif']
            }
        }
    },
    plugins: []
};
```

#### 真实面试题

**题目1：Flexbox 和 Grid 有什么区别？如何选择？**

**满分答案：**

**Flexbox（一维布局）：**
- 适合单行或单列的布局
- 主要用于组件内部的对齐
- 更灵活，适合不确定数量的元素

**Grid（二维布局）：**
- 适合行列同时控制的布局
- 主要用于页面整体布局
- 更精确，适合复杂的网格布局

**选择建议：**

```css
/* 使用 Flexbox：导航栏、工具栏、卡片内部 */
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

/* 使用 Grid：页面布局、卡片网格 */
.page-layout {
    display: grid;
    grid-template-areas:
        "header"
        "main"
        "footer";
}

.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;
}
```

**可以结合使用：**

```css
/* Grid 做整体布局，Flexbox 做局部对齐 */
.layout {
    display: grid;
    grid-template-columns: 200px 1fr;
}

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}
```

---

**题目2：什么是 BFC？如何触发？有什么作用？**

**满分答案：**

**BFC（Block Formatting Context，块级格式化上下文）：**

BFC 是一个独立的渲染区域，内部元素的布局不会影响外部，外部也不会影响内部。

**触发 BFC 的方式：**

```css
/* 1. overflow 不为 visible */
.bfc { overflow: hidden; }

/* 2. display 为 flex/grid/inline-flex/inline-grid */
.bfc { display: flex; }

/* 3. position 为 absolute/fixed */
.bfc { position: absolute; }

/* 4. float 不为 none */
.bfc { float: left; }

/* 5. display: flow-root（专门用于创建 BFC，推荐） */
.bfc { display: flow-root; }
```

**BFC 的作用：**

1. **清除浮动**
```css
.clearfix {
    display: flow-root;  /* 创建 BFC，包含浮动子元素 */
}
```

2. **防止 margin 塌陷**
```css
.parent {
    display: flow-root;  /* 创建 BFC，防止与子元素 margin 合并 */
}
```

3. **防止文字环绕浮动元素**
```css
.text-block {
    overflow: hidden;  /* 创建 BFC，不与浮动元素重叠 */
}
```

4. **自适应两栏布局**
```css
.sidebar { float: left; width: 200px; }
.main { overflow: hidden; }  /* 创建 BFC，不与浮动元素重叠 */
```

---

## 1.9 CSS 主题色切换

#### 知识点详解

**方案一：CSS 变量（推荐）：**

```css
/* 定义 CSS 变量 */
:root {
    --primary-color: #1890ff;
    --bg-color: #ffffff;
    --text-color: #333333;
}

/* 暗色主题 */
[data-theme="dark"] {
    --primary-color: #40a9ff;
    --bg-color: #141414;
    --text-color: #ffffff;
}
```

```javascript
// 切换主题
function setTheme(theme) {
    document.documentElement.setAttribute('data-theme', theme);
    localStorage.setItem('theme', theme);
}

// 初始化
const savedTheme = localStorage.getItem('theme') || 'light';
setTheme(savedTheme);
```

**方案二：Vue 中的实现：**

```vue
<template>
  <div :class="['app', theme]">
    <button @click="toggleTheme">切换主题</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      theme: 'light'
    }
  },
  methods: {
    toggleTheme() {
      this.theme = this.theme === 'light' ? 'dark' : 'light';
      this.applyTheme();
    },
    applyTheme() {
      document.documentElement.setAttribute('data-theme', this.theme);
    }
  }
}
</script>

<style>
:root {
  --bg-color: #ffffff;
}

[data-theme="dark"] {
  --bg-color: #141414;
}

.app {
  background: var(--bg-color);
}
</style>
```

**方案三：React 中的实现：**

```jsx
// ThemeContext.js
import { createContext, useContext, useState, useEffect } from 'react';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
    const [theme, setTheme] = useState('light');
    
    useEffect(() => {
        const saved = localStorage.getItem('theme');
        if (saved) setTheme(saved);
    }, []);
    
    const toggleTheme = () => {
        const newTheme = theme === 'light' ? 'dark' : 'light';
        setTheme(newTheme);
        localStorage.setItem('theme', newTheme);
        document.documentElement.setAttribute('data-theme', newTheme);
    };
    
    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

export const useTheme = () => useContext(ThemeContext);
```

**系统偏好检测：**

```javascript
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)');
prefersDark.addEventListener('change', (e) => {
    setTheme(e.matches ? 'dark' : 'light');
});
```

#### 真实面试题

**题目：前端如何实现页面主题色切换？有哪些方案？**

**满分答案：**

**方案对比：**

| 方案 | 优点 | 缺点 |
|------|------|------|
| CSS 变量 | 性能好，易维护 | 需要浏览器支持 |
| CSS-in-JS | 组件化，动态强 | 运行时开销 |
| 动态加载 CSS | 解耦 | 加载延迟 |

**推荐：CSS 变量 + data-theme**

1. 定义 CSS 变量
2. 使用 data-theme 切换
3. localStorage 持久化
4. 监听系统偏好

---

## 1.10 CSS 水平垂直居中

#### 知识点详解

**一、Flexbox 方案（推荐）：**

```css
/* 水平垂直居中 */
.center-flex {
    display: flex;
    justify-content: center;
    align-items: center;
}

/* 关键代码 */
display: flex;
justify-content: center;  /* 水平居中 */
align-items: center;      /* 垂直居中 */
```

**二、Grid 方案：**

```css
/* 方式1 */
.center-grid {
    display: grid;
    place-items: center;  /* 简写，同时设置 justify-items 和 align-items */
}

/* 方式2 */
.center-grid2 {
    display: grid;
    justify-content: center;
    align-content: center;
}
```

**三、绝对定位 + transform：**

```css
.center-absolute {
    position: relative;  /* 父容器 */
}

.center-absolute > .child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

**四、绝对定位 + margin：**

```css
.center-margin {
    position: relative;
}

.center-margin > .child {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}
```

**五、绝对定位 + 固定宽高：**

```css
.center-fixed {
    position: relative;
}

.center-fixed > .child {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 200px;   /* 固定宽高 */
    height: 100px;
    margin-left: -100px;  /* 负 margin */
    margin-top: -50px;
}
```

**六、行内元素水平居中：**

```css
/* 行内/行内块元素 */
.center-inline {
    text-align: center;
}
```

**七、line-height + text-align（适用于单行文本）：**

```css
.center-text {
    text-align: center;
    line-height: 100px;  /* 等于 height */
}
```

**八、CSS 新特性 - 容器查询（Chrome 105+）：**

```css
/* 父容器 */
.card {
    container-type: inline-size;
}

/* 子元素使用容器查询 */
@container (min-width: 300px) {
    .inner {
        display: flex;
        justify-content: center;
        align-items: center;
    }
}
```

#### 真实面试题

**题目：CSS 中实现水平垂直居中的方式有哪些？**

**满分答案：**

| 方案 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| Flexbox | 任何场景 | 简洁强大，推荐 | 需兼容旧浏览器 |
| Grid | 任何场景 | 最简洁 | 旧浏览器兼容性 |
| 绝对定位 + transform | 已知/未知宽高 | 兼容性最好 | 需计算 |
| 绝对定位 + margin | 固定宽高 | 简单 | 需要知道宽高 |
| table-cell | 父容器定高 | 兼容性好 | 需设高度 |

**实际推荐：**
- **首选 Flexbox**：`display: flex; justify-content: center; align-items: center;`
- **次选 Grid**：`display: grid; place-items: center;`
- **绝对定位备选**：适用于弹窗、遮罩等特殊场景

---

## 1.11 AI 对话列表布局实战

#### 知识点详解

**布局需求分析：**

```
┌─────────────────────────────────────┐
│  AI Chat Layout                      │
├──────────┬──────────────────────────┤
│  头像     │  用户消息（右侧）         │
│  (左固定) │  自动换行                │
│           │                          │
│  (左固定) │  AI回复消息（右侧）       │
│           │  自动换行，支持代码块       │
└──────────┴──────────────────────────┘
```

**核心实现：**

```css
/* 外层容器 */
.chat-list {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

/* 消息行：flex 横向布局 */
.message-row {
    display: flex;
    align-items: flex-start;
    gap: 12px;
}

/* 头像固定宽度 */
.avatar {
    width: 36px;
    height: 36px;
    flex-shrink: 0;
    border-radius: 50%;
}

/* 消息内容：flex: 1 自适应，且自动换行 */
.message-content {
    flex: 1;
    /* 重要：允许内容换行 */
    min-width: 0;
    word-break: break-word;
    overflow-wrap: break-word;
}

/* 头像位置：左固定右自适应 */
.message-row.user {
    flex-direction: row;
}
.message-row.ai {
    flex-direction: row;
}

/* AI 消息靠左，用户消息靠右 */
.message-row.user .message-content {
    background: #e8f0fe;
    border-radius: 12px 12px 4px 12px;
}
.message-row.ai .message-content {
    background: #f1f3f4;
    border-radius: 12px 12px 12px 4px;
}

/* 代码块样式 */
.message-content pre {
    background: #1e1e1e;
    color: #d4d4d4;
    padding: 12px;
    border-radius: 8px;
    overflow-x: auto;
    margin: 8px 0;
}
```

**带头像区分的完整实现：**

```jsx
function ChatMessage({ message }) {
    const isUser = message.role === 'user';
    return (
        <div className={`message-row ${isUser ? 'user' : 'ai'}`}>
            {/* 头像：左固定 */}
            <div className="avatar">
                <img
                    src={isUser ? '/icons/user.png' : '/icons/ai.png'}
                    alt={isUser ? '用户' : 'AI'}
                />
            </div>

            {/* 消息内容：右自适应，自动换行 */}
            <div className="message-content">
                {message.isStreaming ? (
                    <StreamingText text={message.text} />
                ) : (
                    <div>{message.text}</div>
                )}
            </div>
        </div>
    );
}
```

**关键 CSS 技巧：**

```css
/* 1. 头像固定，内容自适应 */
.avatar {
    flex-shrink: 0;   /* 不被压缩 */
    width: 36px;
}
.message-content {
    flex: 1;          /* 占据剩余空间 */
}

/* 2. 长文本自动换行 */
.message-content {
    min-width: 0;     /* 关键！允许收缩到0再换行 */
    word-break: break-word;
}

/* 3. 代码块不溢出 */
pre {
    max-width: 100%;
    overflow-x: auto;  /* 水平滚动而非溢出 */
}
```

#### 真实面试题

**题目：实现一个类似 AI 聊天对话框的 CSS 布局，要求左侧头像固定，右侧内容自适应，且内容过长时自动换行**

**满分答案：**

**核心布局：Flexbox 横向布局**

1. **消息行用 Flex**：`display: flex; align-items: flex-start`
2. **头像固定**：`flex-shrink: 0; width: 36px`（不被压缩，宽度固定）
3. **内容自适应**：`flex: 1`（占据剩余空间）
4. **自动换行关键**：`min-width: 0` + `word-break: break-word`

**常见踩坑：**
- ❌ 不加 `min-width: 0`，内容不换行，直接撑破布局
- ❌ 不加 `flex-shrink: 0`，头像被压缩变形
- ✅ 代码块用 `overflow-x: auto` 而非直接溢出

---

## 1.12 CSS 无限旋转 Loading 动画

#### 知识点详解

**基础实现：**

```css
/* 1. 定义旋转动画 */
@keyframes spin {
    from { transform: rotate(0deg); }
    to   { transform: rotate(360deg); }
}

/* 2. 应用到元素 */
.loading-spinner {
    width: 32px;
    height: 32px;
    border: 3px solid #e0e0e0;       /* 底色 */
    border-top-color: #3b82f6;       /* 旋转部分颜色 */
    border-radius: 50%;               /* 圆形 */
    animation: spin 0.8s linear infinite;  /* 匀速无限旋转 */
}
```

**带渐变的高级 Loading：**

```css
/* 渐变 + 旋转 = 更现代的加载效果 */
.loading-advanced {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    background: conic-gradient(
        from 0deg,
        #3b82f6 0deg,
        #8b5cf6 90deg,
        #06b6d4 180deg,
        transparent 270deg
    );
    animation: spin 1.2s linear infinite;
    /* 配合 mask 形成缺口效果 */
    -webkit-mask: radial-gradient(
        farthest-side,
        transparent calc(100% - 4px),
        white calc(100% - 4px)
    );
    mask: radial-gradient(
        farthest-side,
        transparent calc(100% - 4px),
        white calc(100% - 4px)
    );
}

/* 打字机效果：AI 思考中 */
@keyframes thinking {
    0%, 100% { opacity: 0.3; }
    50%       { opacity: 1; }
}
.loading-dots span {
    display: inline-block;
    width: 8px;
    height: 8px;
    background: #666;
    border-radius: 50%;
    animation: thinking 1.2s ease-in-out infinite;
}
.loading-dots span:nth-child(2) { animation-delay: 0.2s; }
.loading-dots span:nth-child(3) { animation-delay: 0.4s; }
```

**配合 AI 思考状态的完整示例：**

```jsx
function AIThinkingIndicator() {
    return (
        <div className="ai-thinking">
            <div className="avatar">
                <img src="/icons/ai.png" alt="AI" />
            </div>
            <div className="thinking-bubbles">
                {/* 三点跳动效果 */}
                <div className="loading-dots">
                    <span></span>
                    <span></span>
                    <span></span>
                </div>
                {/* 旋转圆环（可选） */}
                <div className="loading-spinner"></div>
                <p className="thinking-text">AI 正在思考...</p>
            </div>
        </div>
    );
}
```

```css
.ai-thinking {
    display: flex;
    align-items: center;
    gap: 12px;
    animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
    from { opacity: 0; transform: translateY(8px); }
    to   { opacity: 1; transform: translateY(0); }
}
```

#### 真实面试题

**题目：CSS 中如何实现一个无限旋转的 Loading 动画，用于表示 AI 正在思考？**

**满分答案：**

**三步实现：**

1. **定义动画**：`@keyframes spin { from → to { transform: rotate(0→360deg) } }`
2. **绘制圆环**：`border-radius: 50%` + `border-top-color`（其余三边底色）
3. **应用动画**：`animation: spin 0.8s linear infinite`

**AI 思考状态的最佳实践：**

| 效果 | 场景 | CSS 技巧 |
|------|------|---------|
| 旋转圆环 | 长时间加载 | `linear` 匀速 |
| 三点跳动 | AI 实时思考 | `ease-in-out` + `nth-child` 错开 |
| 渐变圆环 | 现代化风格 | `conic-gradient` + `mask` 裁剪 |

**加分项：** 说明配合 `prefers-reduced-motion` 媒体查询做无障碍处理：
```css
@media (prefers-reduced-motion: reduce) {
    .loading-spinner { animation: none; }
}
```

---
