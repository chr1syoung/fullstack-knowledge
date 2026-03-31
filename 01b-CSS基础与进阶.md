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

## 1.2.3 CSS 主题色切换

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

## 1.2.4 CSS 水平垂直居中

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

## 1.2.5 AI 对话列表布局实战

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

## 1.2.6 CSS 无限旋转 Loading 动画

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

## 1.2.7 animation/transition/transform 的区别

#### 知识点详解

**transform（变换）：**
- 定义元素的变换效果（旋转、缩放、倾斜、位移）
- 不会触发回流（reflow），只影响合成层（composite layer）
- 由 GPU 加速，性能优异
- 常见函数：`translate()`、`rotate()`、`scale()`、`skew()`

```css
/* 位移 */
transform: translate(50px, 100px);

/* 旋转 */
transform: rotate(45deg);

/* 缩放 */
transform: scale(1.5);

/* 组合 */
transform: translateX(50px) rotate(45deg) scale(1.2);
```

**transition（过渡）：**
- 定义属性变化的过渡效果
- 需要触发条件（如 `:hover`、`:active`、JS 修改属性）
- 只能从状态 A 过渡到状态 B（两个状态之间）
- 无法循环播放

```css
/* 基础语法 */
.element {
    background: blue;
    transition: background 0.3s ease, transform 0.5s ease-in-out;
}
.element:hover {
    background: red;
    transform: scale(1.1);
}
```

**animation（动画）：**
- 定义复杂动画，使用 `@keyframes` 定义多个关键帧
- 可以循环播放（`infinite`）
- 可以控制播放状态（暂停/播放）
- 不需要触发条件，可以自动播放

```css
/* 定义关键帧 */
@keyframes slideIn {
    0% { transform: translateX(-100%); opacity: 0; }
    50% { transform: translateX(10%); opacity: 0.5; }
    100% { transform: translateX(0); opacity: 1; }
}

/* 应用动画 */
.element {
    animation: slideIn 1s ease-out infinite;
}
```

**三者对比：**

| 特性 | transform | transition | animation |
|------|-----------|------------|-----------|
| 功能定位 | 定义变换 | 属性过渡 | 复杂动画 |
| 触发条件 | 无（即时生效） | 需要触发 | 自动播放 |
| 状态数量 | 单状态 | 两个状态 | 多关键帧 |
| 循环播放 | 不支持 | 不支持 | 支持 |
| 性能 | GPU 加速 | 部分 GPU 加速 | GPU 加速 |
| 适用场景 | 静态变换 | 简单交互 | 复杂动画 |

#### 真实面试题

**题目：css中的animation, transition, transform有什么区别?**

**满分答案：**

**1. 功能定位不同：**

- **transform**：定义元素的变换效果（旋转、缩放、倾斜、位移），是静态的变换声明
- **transition**：定义属性变化的过渡效果，用于平滑过渡两个状态之间的变化
- **animation**：定义复杂动画，使用 `@keyframes` 可以定义多个关键帧，实现复杂动画序列

**2. 触发机制不同：**

- **transform**：无需触发，即时生效
- **transition**：需要触发条件（hover、active、JS 修改属性等）
- **animation**：可自动播放，也可通过 JS 控制

**3. 性能特点：**

- **transform**：不会触发回流，只影响合成层，GPU 加速，性能最优
- **transition**：触发重绘或合成，性能较好
- **animation**：同样使用 GPU 加速，但复杂动画可能消耗更多资源

**4. 使用场景：**

```css
/* transform：静态变换 */
.icon { transform: rotate(45deg); }

/* transition：简单交互 */
.button {
    background: blue;
    transition: background 0.3s;
}
.button:hover { background: red; }

/* animation：复杂动画 */
@keyframes bounce {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-20px); }
}
.loader { animation: bounce 1s infinite; }
```

**总结：**
- 需要**静态变换**用 transform
- 需要**简单过渡**用 transition
- 需要**复杂动画**用 animation

---

## 1.2.8 移动端样式适配

#### 知识点详解

**Viewport Meta 标签：**

```html
<!-- 基础设置 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- 完整设置 -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

**适配方案对比：**

**1. rem 适配（传统方案）：**

```javascript
// lib-flexible 原理
function setRem() {
    const docEl = document.documentElement;
    const clientWidth = docEl.clientWidth;
    // 以 iPhone 6/7/8 的 375px 为基准，1rem = 100px
    docEl.style.fontSize = 100 * (clientWidth / 375) + 'px';
}
window.addEventListener('resize', setRem);
```

```css
/* 使用 rem */
.container {
    width: 3.75rem;  /* 375px */
    padding: 0.16rem; /* 16px */
}
```

**2. vw/vh 适配（现代推荐）：**

```css
/* 1vw = 视口宽度的 1% */
.container {
    width: 100vw;
    padding: 4.27vw;  /* 16px/375px ≈ 4.27vw */
}

/* 使用 calc 配合设计稿 */
.element {
    width: calc(100vw - 32px);
    font-size: clamp(14px, 3.73vw, 18px); /* 最小14px，最大18px */
}
```

**3. 媒体查询（断点适配）：**

```css
/* 移动优先 */
.container { padding: 16px; }

@media (min-width: 768px) {
    .container { padding: 24px; max-width: 720px; }
}

@media (min-width: 1024px) {
    .container { padding: 32px; max-width: 960px; }
}
```

**4. Flexible 布局（Flexbox/Grid）：**

```css
/* Flexbox 自适应 */
.card-list {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
}
.card {
    flex: 1 1 300px;  /* 弹性伸缩 */
}

/* Grid 自适应 */
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 16px;
}
```

**1px 边框解决方案：**

```css
/* 方案1：transform scale */
.border-1px {
    position: relative;
}
.border-1px::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 1px;
    background: #ccc;
    transform: scaleY(0.5);
    transform-origin: 0 0;
}

/* 方案2：使用 box-shadow */
.border-shadow {
    box-shadow: inset 0 -0.5px 0 0 #ccc;
}
```

**安全区域适配（刘海屏）：**

```css
/* 适配 iPhone X 等刘海屏 */
.safe-area {
    padding-top: env(safe-area-inset-top);
    padding-bottom: env(safe-area-inset-bottom);
    padding-left: env(safe-area-inset-left);
    padding-right: env(safe-area-inset-right);
}
```

#### 真实面试题

**题目：怎么做移动端的样式适配?**

**满分答案：**

**1. Viewport 设置：**

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**2. 主流适配方案对比：**

| 方案 | 原理 | 优点 | 缺点 |
|------|------|------|------|
| rem | 根据根元素 font-size 计算 | 兼容性好 | 需要 JS 计算 |
| vw/vh | 视口百分比 | 原生支持，无需 JS | 兼容性稍差（IE不支持） |
| 媒体查询 | 断点适配 | 精确控制 | 代码量大 |
| flex/grid | 弹性布局 | 最灵活 | 需要配合其他方案 |

**3. 现代推荐方案：**

```css
/* vw + clamp() 方案 */
.element {
    padding: clamp(12px, 3.2vw, 20px);
    font-size: clamp(14px, 3.73vw, 18px);
}
```

**4. 1px 问题解决方案：**

```css
/* 伪元素 + transform scale */
.border::after {
    content: '';
    position: absolute;
    width: 200%;
    height: 200%;
    border: 1px solid #ccc;
    transform: scale(0.5);
    transform-origin: 0 0;
}
```

**5. 安全区域适配：**

```css
.header {
    padding-top: env(safe-area-inset-top);
}
```

---

## 1.2.9 inline-block 节点间隔

#### 知识点详解

**问题原因：**

HTML 标签之间的空白字符（换行、空格、Tab）会被浏览器渲染为一个空格（约 4px）。

```html
<!-- 以下代码会产生间隔 -->
<div class="parent">
    <span class="item">A</span>
    <span class="item">B</span>
    <span class="item">C</span>
</div>
<!-- A B C 之间会有空格间隔 -->
```

**解决方案：**

**方案1：父元素 font-size: 0（推荐）**

```css
.parent {
    font-size: 0;  /* 消除空白字符 */
}
.item {
    display: inline-block;
    font-size: 16px;  /* 子元素恢复字体大小 */
}
```

**方案2：标签写在同一行（不推荐）**

```html
<!-- 消除换行 -->
<div class="parent">
    <span class="item">A</span><span class="item">B</span><span class="item">C</span>
</div>
```

**方案3：使用负 margin**

```css
.item {
    display: inline-block;
    margin-right: -4px;  /* 抵消空白字符 */
}
```

**方案4：使用 float 替代**

```css
.item {
    float: left;
}
.parent::after {
    content: '';
    display: block;
    clear: both;
}
```

**方案5：使用 flex 布局（现代推荐）**

```css
.parent {
    display: flex;
    gap: 0;  /* 或设置需要的间距 */
}
```

**方案6：使用 grid 布局**

```css
.parent {
    display: grid;
    grid-auto-flow: column;
    gap: 0;
}
```

#### 真实面试题

**题目：相邻的两个inline-block节点为什么会出现间隔，该如何解决?**

**满分答案：**

**根本原因：**

HTML 源码中的换行符、空格、Tab 等空白字符，在渲染时会被合并为一个空格（约 4px），导致 inline-block 元素之间产生间隔。

**5种解决方案：**

```css
/* 方案1：父元素 font-size: 0（最常用） */
.parent { font-size: 0; }
.item { display: inline-block; font-size: 16px; }

/* 方案2：负 margin */
.item { display: inline-block; margin-right: -4px; }

/* 方案3：HTML 标签不换行（不推荐，可读性差） */
<span>A</span><span>B</span>

/* 方案4：float 替代 */
.item { float: left; }

/* 方案5：flex 布局（现代推荐） */
.parent { display: flex; }
```

**现代替代方案：**

现代开发推荐直接使用 `flex` 或 `grid` 布局替代 `inline-block`，从根本上避免间隔问题：

```css
/* Flexbox 方案 */
.nav {
    display: flex;
    gap: 16px;  /* 精确控制间距 */
}

/* Grid 方案 */
.grid {
    display: grid;
    grid-auto-flow: column;
    gap: 16px;
}
```

---

## 1.2.10 Grid 网格布局

#### 知识点详解

**CSS Grid 核心概念：**

CSS Grid 是二维布局系统，可以同时控制行和列，适合复杂的页面整体布局。

**核心术语：**

- **Grid Container**：网格容器（`display: grid`）
- **Grid Item**：网格项（容器的直接子元素）
- **Grid Line**：网格线（分隔网格的线）
- **Grid Track**：网格轨道（行或列）
- **Grid Cell**：网格单元（行和列交叉的最小单元）
- **Grid Area**：网格区域（一个或多个单元格组成的矩形区域）

**常用属性：**

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
    
    /* 间距 */
    gap: 20px;
    row-gap: 10px;
    column-gap: 20px;
    
    /* 对齐 */
    justify-items: center;  /* 水平对齐 */
    align-items: center;    /* 垂直对齐 */
    place-items: center;    /* 简写 */
    
    /* 命名区域 */
    grid-template-areas:
        "header header header"
        "sidebar main main"
        "footer footer footer";
}

/* 子项属性 */
.item {
    /* 跨越行列 */
    grid-column: 1 / 3;      /* 从第1列到第3列 */
    grid-column: 1 / span 2; /* 从第1列跨2列 */
    
    grid-row: 1 / 3;
    
    /* 使用命名区域 */
    grid-area: header;
    
    /* 单独对齐 */
    justify-self: end;
    align-self: center;
}
```

**特殊单位与函数：**

```css
.container {
    /* fr 单位：剩余空间的分数 */
    grid-template-columns: 1fr 2fr 1fr;  /* 1:2:1 比例 */
    
    /* minmax()：设置最小和最大值 */
    grid-template-columns: minmax(200px, 1fr) 1fr;
    
    /* repeat()：重复模式 */
    grid-template-columns: repeat(4, 1fr);
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    
    /* auto-fill vs auto-fit */
    /* auto-fill：尽可能多填充轨道，可能产生空轨道 */
    /* auto-fit：拉伸现有轨道填满容器 */
}
```

**Grid vs Flexbox：**

| 特性 | Grid | Flexbox |
|------|------|---------|
| 维度 | 二维（行+列） | 一维（行或列） |
| 适用场景 | 页面整体布局 | 组件内部对齐 |
| 内容驱动 | 否（先定义网格） | 是（根据内容自适应） |
| 重叠控制 | 支持（z-index） | 有限支持 |

#### 真实面试题

**题目：grid网格布局是什么?**

**满分答案：**

**定义：**

CSS Grid 是一个二维布局系统，可以同时控制行和列的布局，适合复杂的页面整体布局。

**核心概念：**

1. **Grid Container**：声明 `display: grid` 的元素
2. **Grid Item**：网格容器的直接子元素
3. **Grid Line**：分隔网格的线
4. **Grid Track**：行或列的网格轨道
5. **Grid Cell**：最小的网格单元
6. **Grid Area**：一个或多个单元格组成的区域

**常用属性示例：**

```css
/* 基础网格 */
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: auto 1fr auto;
    gap: 20px;
}

/* 圣杯布局 */
.layout {
    display: grid;
    grid-template-areas:
        "header header header"
        "sidebar main aside"
        "footer footer footer";
    grid-template-columns: 200px 1fr 200px;
    min-height: 100vh;
}
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
```

**与 Flexbox 的区别：**

- **Grid**：二维布局，适合页面整体结构（如 header/sidebar/main/footer）
- **Flexbox**：一维布局，适合组件内部对齐（如导航栏、按钮组）

**实际布局示例：**

```css
/* 响应式卡片网格 */
.cards {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 24px;
}
```

---

## 1.2.11 CSS3 新增特性

#### 知识点详解

**CSS3 新增特性分类：**

**1. 选择器增强：**

```css
/* 结构伪类 */
:nth-child(n)          /* 第n个子元素 */
:nth-of-type(n)        /* 第n个特定类型的子元素 */
:not(selector)         /* 否定选择器 */
:has(selector)         /* 父选择器（:has） */

/* 属性选择器增强 */
[href^="https"]        /* 以 https 开头 */
[href$=".pdf"]         /* 以 .pdf 结尾 */
[class*="btn"]         /* 包含 btn */

/* 其他 */
:is(h1, h2, h3)        /* 匹配任一 */
:where(h1, h2, h3)     /* 匹配任一，优先级为0 */
```

**2. 布局模块：**

```css
/* Flexbox */
display: flex;

/* Grid */
display: grid;

/* 多列布局 */
column-count: 3;
column-gap: 20px;
```

**3. 视觉效果：**

```css
/* 圆角 */
border-radius: 8px;

/* 阴影 */
box-shadow: 0 2px 8px rgba(0,0,0,0.1);
text-shadow: 1px 1px 2px black;

/* 渐变 */
background: linear-gradient(to right, red, blue);
background: radial-gradient(circle, red, blue);

/* 透明度 */
opacity: 0.5;
background: rgba(255,0,0,0.5);
```

**4. 变换与动画：**

```css
/* Transform */
transform: translate(50px, 100px);
transform: rotate(45deg);
transform: scale(1.5);

/* Transition */
transition: all 0.3s ease;

/* Animation */
@keyframes slide {
    from { transform: translateX(0); }
    to { transform: translateX(100px); }
}
animation: slide 1s ease infinite;
```

**5. 背景增强：**

```css
/* 背景大小 */
background-size: cover;
background-size: contain;

/* 背景原点 */
background-origin: content-box;

/* 背景裁剪 */
background-clip: text;  /* 文字裁剪背景 */
```

**6. 文字效果：**

```css
/* 文字溢出 */
text-overflow: ellipsis;

/* 自动换行 */
word-wrap: break-word;
overflow-wrap: break-word;

/* 自定义字体 */
@font-face {
    font-family: 'MyFont';
    src: url('font.woff2') format('woff2');
}
```

**7. CSS 变量（自定义属性）：**

```css
:root {
    --primary-color: #007bff;
    --spacing: 16px;
}
.button {
    background: var(--primary-color);
    padding: var(--spacing);
}
```

**8. 容器查询（新特性）：**

```css
/* 根据容器大小而非视口 */
.card {
    container-type: inline-size;
}
@container (min-width: 400px) {
    .card-content { display: flex; }
}
```

**9. 逻辑属性：**

```css
/* 逻辑方向，适配不同书写模式 */
margin-inline: 20px;   /* 逻辑水平方向 */
margin-block: 10px;    /* 逻辑垂直方向 */
padding-inline-start: 16px;
border-inline: 1px solid;
```

#### 真实面试题

**题目：CSS3新增了哪些特性?**

**满分答案：**

**CSS3 主要新增特性（按类别）：**

| 类别 | 重要特性 | 面试频率 |
|------|---------|---------|
| 选择器 | :nth-child、:not、:has、属性选择器 | ⭐⭐⭐⭐⭐ |
| 布局 | flexbox、grid、multi-column | ⭐⭐⭐⭐⭐ |
| 视觉效果 | border-radius、box-shadow、gradient、opacity | ⭐⭐⭐⭐⭐ |
| 变换动画 | transform、transition、animation | ⭐⭐⭐⭐⭐ |
| 背景 | background-size、background-clip | ⭐⭐⭐⭐ |
| 文字 | text-overflow、@font-face | ⭐⭐⭐⭐ |
| 变量 | CSS Variables（var） | ⭐⭐⭐⭐⭐ |
| 容器查询 | @container | ⭐⭐⭐ |
| 逻辑属性 | margin-inline、padding-block | ⭐⭐⭐ |

**高频考点代码示例：**

```css
/* 选择器 */
li:nth-child(odd) { background: #f5f5f5; }
button:not(:disabled) { cursor: pointer; }

/* Flexbox 居中 */
.center { display: flex; justify-content: center; align-items: center; }

/* CSS 变量 */
:root { --primary: #1890ff; }
.btn { background: var(--primary); }

/* 渐变背景 */
.gradient { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
```

---

## 1.2.12 CSS3 实现动画

#### 知识点详解

**CSS3 动画两种方式：**

**1. Transition（过渡）：**

```css
/* 基础语法 */
.element {
    transition: property duration timing-function delay;
}

/* 示例 */
.button {
    background: blue;
    transform: scale(1);
    transition: background 0.3s ease, transform 0.2s ease-out;
}
.button:hover {
    background: darkblue;
    transform: scale(1.05);
}
```

**2. Animation（动画）：**

```css
/* 定义关键帧 */
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

/* 应用动画 */
.element {
    animation: slideIn 1s ease-out forwards;
}

/* 详细属性 */
.animation-detailed {
    animation-name: slideIn;
    animation-duration: 1s;
    animation-timing-function: ease;
    animation-delay: 0.5s;
    animation-iteration-count: infinite;  /* 或具体数字 */
    animation-direction: normal;          /* reverse, alternate */
    animation-fill-mode: forwards;        /* 保持结束状态 */
    animation-play-state: running;        /* paused */
}
```

**Animation 属性详解：**

| 属性 | 说明 | 常用值 |
|------|------|--------|
| name | 动画名称 | @keyframes 定义的名称 |
| duration | 持续时间 | 1s, 500ms |
| timing-function | 时间函数 | ease, linear, ease-in-out |
| delay | 延迟 | 0s, 1s |
| iteration-count | 播放次数 | 1, infinite |
| direction | 播放方向 | normal, reverse, alternate |
| fill-mode | 填充模式 | forwards, backwards, both |
| play-state | 播放状态 | running, paused |

**性能优化：**

```css
/* 只动画 transform 和 opacity（GPU 加速） */
@keyframes optimized {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(100px); opacity: 0; }
}

/* 避免动画触发回流 */
/* ❌ 不好：触发重排 */
@keyframes bad {
    from { width: 100px; }
    to { width: 200px; }
}

/* ✅ 好：只触发合成 */
@keyframes good {
    from { transform: scaleX(1); }
    to { transform: scaleX(2); }
}

/* will-change 预声明（谨慎使用） */
.animated {
    will-change: transform, opacity;
}
```

#### 真实面试题

**题目：怎么使用CSS3实现动画?**

**满分答案：**

**CSS3 实现动画的两种方式：**

**1. Transition（过渡）：**

适用于简单的属性变化，需要触发条件（如 hover）。

```css
.button {
    background: blue;
    transition: background 0.3s ease;
}
.button:hover {
    background: red;
}
```

**2. Animation（动画）：**

适用于复杂动画，使用 `@keyframes` 定义关键帧。

```css
/* 定义关键帧 */
@keyframes slideIn {
    0% { transform: translateX(-100%); opacity: 0; }
    100% { transform: translateX(0); opacity: 1; }
}

/* 应用动画 */
.element {
    animation: slideIn 1s ease-out forwards;
}
```

**完整代码示例：**

```css
/* 加载动画 */
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}

.loading {
    width: 40px;
    height: 40px;
    border: 3px solid #e0e0e0;
    border-top-color: #3b82f6;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
}
```

**性能优化要点：**

1. **只动画 `transform` 和 `opacity`**：这两个属性不触发回流，由 GPU 加速
2. **避免动画 `width`、`height`、`top`、`left`**：会触发重排，性能差
3. **使用 `will-change` 预声明**：提示浏览器优化，但不要过度使用

---

## 1.2.13 回流与重绘

#### 知识点详解

**回流（Reflow / Relayout）：**

当元素的几何属性（大小、位置）发生变化，浏览器需要重新计算元素的位置和大小，并重新构建渲染树。

**触发回流的操作：**

```css
/* 几何属性变化 */
width, height
padding, margin, border
display: none / block
position, float, clear
top, left, right, bottom
font-size, font-family
overflow（改变时）
```

```javascript
// DOM 操作
el.style.width = '100px';           // 回流
el.className = 'new-class';         // 回流
el.appendChild(newEl);              // 回流
window.scrollTo(0, 100);            // 回流
window.resize;                      // 回流
```

**重绘（Repaint）：**

当元素的外观属性（颜色、背景等）发生变化，但不影响布局时，浏览器只需重新绘制元素。

**触发重绘的操作：**

```css
/* 外观属性变化 */
color
background-color, background-image
visibility
box-shadow
outline
border-radius（某些情况）
```

**回流与重绘的关系：**

- **回流一定触发重绘**：布局变化后必须重新绘制
- **重绘不一定触发回流**：只改颜色不影响布局

**性能优化策略：**

```javascript
// ❌ 不好：多次触发回流
const el = document.getElementById('box');
el.style.width = '100px';
el.style.height = '100px';
el.style.margin = '10px';

// ✅ 好：批量修改，只触发一次回流
const el = document.getElementById('box');
el.style.cssText = 'width: 100px; height: 100px; margin: 10px;';

// ✅ 更好：使用 class
document.getElementById('box').className = 'new-style';
```

```javascript
// ❌ 不好：强制同步布局（读取后立即写入）
const height = box.offsetHeight;  // 读取（触发回流）
box.style.height = height + 10 + 'px';  // 写入（再次触发）

// ✅ 好：先读取，再写入
const height = box.offsetHeight;
// ... 其他读取操作
box.style.height = height + 10 + 'px';  // 批量写入
```

```css
/* 使用 transform 代替位置变化 */
/* ❌ 触发回流 */
.box { position: relative; left: 100px; }

/* ✅ 只触发合成 */
.box { transform: translateX(100px); }

/* 使用 requestAnimationFrame */
function animate() {
    requestAnimationFrame(() => {
        element.style.transform = `translateX(${x}px)`;
    });
}
```

#### 真实面试题

**题目：怎么理解回流跟重绘?什么场景下会触发?**

**满分答案：**

**定义：**

- **回流（Reflow）**：元素几何属性（大小、位置）变化，浏览器重新计算布局并重建渲染树
- **重绘（Repaint）**：元素外观属性（颜色、背景）变化，浏览器重新绘制元素

**关系：**

- 回流 **一定** 触发重绘
- 重绘 **不一定** 触发回流

**触发场景：**

**回流触发场景：**
- 增删 DOM 元素
- 改变几何属性：width、height、padding、margin、border、position、display
- 窗口 resize
- 字体变化
- 读取布局属性：offsetHeight、scrollTop、getComputedStyle

**重绘触发场景：**
- color、background、visibility、box-shadow、outline 变化

**性能优化策略：**

```javascript
// 1. 批量修改样式（使用 cssText 或 class）
// ❌ 不好
el.style.width = '100px';
el.style.height = '100px';

// ✅ 好
el.className = 'new-style';

// 2. 使用 transform 代替位置变化
// ❌ 触发回流
el.style.left = '100px';

// ✅ 只触发合成
el.style.transform = 'translateX(100px)';

// 3. 离线操作 DOM（文档碎片）
const fragment = document.createDocumentFragment();
// ... 在 fragment 上操作
parent.appendChild(fragment);

// 4. 使用 will-change 预声明
.animated { will-change: transform; }

// 5. 避免强制同步布局
// ❌ 交替读写
const h = el.offsetHeight;  // 读
el.style.height = h + 10 + 'px';  // 写
const w = el.offsetWidth;  // 又读 - 强制回流！

// ✅ 先读后写
const h = el.offsetHeight;
const w = el.offsetWidth;
el.style.height = h + 10 + 'px';  // 批量写
el.style.width = w + 10 + 'px';
```

---

## 1.2.14 响应式设计

#### 知识点详解

**响应式设计定义：**

同一页面在不同设备（手机、平板、桌面）和屏幕尺寸下都能有良好的展示效果。

**三大核心原理：**

1. **流式布局**：使用相对单位（%、vw、rem）而非固定像素
2. **媒体查询**：根据设备特性应用不同样式
3. **弹性图片/媒体**：图片和视频自适应容器

**实现技术栈：**

```css
/* 1. Viewport 设置 */
<meta name="viewport" content="width=device-width, initial-scale=1.0">

/* 2. 媒体查询 */
/* 移动优先 */
.container { padding: 16px; }

@media (min-width: 768px) {
    .container { max-width: 720px; margin: 0 auto; }
}

@media (min-width: 1024px) {
    .container { max-width: 960px; }
}

/* 3. 相对单位 */
.element {
    width: 100%;
    font-size: clamp(14px, 3.73vw, 18px);
}

/* 4. 弹性图片 */
img {
    max-width: 100%;
    height: auto;
}
```

**容器查询（新规范）：**

```css
/* 根据容器大小而非视口 */
.card-container {
    container-type: inline-size;
}

@container (min-width: 400px) {
    .card {
        display: grid;
        grid-template-columns: 1fr 2fr;
    }
}
```

**常用断点（Bootstrap 标准）：**

| 名称 | 宽度 | 设备 |
|------|------|------|
| xs | < 576px | 超小屏 |
| sm | ≥ 576px | 手机横屏 |
| md | ≥ 768px | 平板 |
| lg | ≥ 992px | 桌面 |
| xl | ≥ 1200px | 大桌面 |
| xxl | ≥ 1400px | 超大桌面 |

**移动优先 vs 桌面优先：**

```css
/* 移动优先（推荐） */
/* 从小屏幕开始，逐步为大屏幕添加样式 */
.container { width: 100%; }
@media (min-width: 768px) { /* 平板及以上 */ }

/* 桌面优先（不推荐） */
/* 从大屏幕开始，逐步为小屏幕添加样式 */
.container { width: 960px; }
@media (max-width: 768px) { /* 平板及以下 */ }
```

#### 真实面试题

**题目：什么是响应式设计?响应式设计的基本原理是什么?如何进行实现?**

**满分答案：**

**定义：**

响应式设计是指同一页面在不同设备和屏幕尺寸下都能有良好展示效果的设计理念。

**三大核心原理：**

1. **流式布局**：使用相对单位（%、vw、rem）代替固定像素
2. **媒体查询**：通过 `@media` 断点针对不同屏幕应用不同样式
3. **弹性图片/媒体**：图片和视频自动适应容器宽度

**实现技术栈：**

```css
/* 1. Viewport 设置 */
<meta name="viewport" content="width=device-width, initial-scale=1.0">

/* 2. 媒体查询（移动优先策略） */
.container { padding: 16px; }

@media (min-width: 768px) {
    .container { max-width: 720px; }
}

@media (min-width: 1024px) {
    .container { max-width: 960px; }
}

/* 3. Flexbox/Grid 弹性布局 */
.cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 16px;
}

/* 4. clamp() 响应式字体 */
h1 { font-size: clamp(1.5rem, 5vw, 3rem); }
```

**常用断点设计：**

| 设备 | 断点 |
|------|------|
| 手机 | < 576px |
| 平板 | 576px ~ 768px |
| 桌面 | 768px ~ 1200px |
| 大屏 | > 1200px |

---

## 1.2.15 CSS 提高页面性能

#### 知识点详解

**CSS 性能优化按阶段分类：**

**1. 传输阶段：**

```css
/* 压缩 CSS（构建工具处理） */

/* 避免 @import（串行加载，阻塞渲染） */
/* ❌ 不好 */
@import url('other.css');

/* ✅ 好：使用 <link> 并行加载 */
<!-- <link rel="stylesheet" href="a.css"> -->
<!-- <link rel="stylesheet" href="b.css"> -->
```

**2. 解析阶段：**

```css
/* 减少选择器复杂度 */
/* ❌ 不好 */
.nav ul li a:hover .item span { color: red; }

/* ✅ 好 */
.nav-link:hover span { color: red; }

/* 避免通用选择器性能开销 */
* { box-sizing: border-box; }  /* 这个是必要的，但避免通配符复杂选择器 */
```

**3. 渲染阶段：**

```css
/* 关键 CSS 内联首屏 */
/* 将首屏渲染所需的 CSS 内联到 <head>，其余异步加载 */

/* 减少重排重绘 */
.box {
    /* ❌ 不好：触发重排 */
    /* left: 100px; */
    
    /* ✅ 好：只触发合成 */
    transform: translateX(100px);
    opacity: 0.5;
}

/* 避免昂贵的渲染属性 */
.element {
    /* box-shadow、filter、blur 在大元素上性能差 */
    filter: blur(10px);  /* 谨慎使用 */
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);  /* 适度使用 */
}
```

**4. 动画阶段：**

```css
/* 硬件加速 */
.gpu {
    will-change: transform;
    transform: translateZ(0);  /* 触发 GPU 加速 */
}

/* 只动画 transform 和 opacity */
@keyframes bad {
    from { width: 0; }
    to { width: 200px; }  /* ❌ 触发回流 */
}

@keyframes good {
    from { transform: scaleX(0); opacity: 0; }
    to { transform: scaleX(1); opacity: 1; }  /* ✅ 只触发合成 */
}

/* 5. 使用 contain 限制渲染范围 */
.widget {
    contain: layout paint;  /* 内部变化不影响外部 */
}

/* 6. 减少请求数 */
.icon { background-image: url('sprite.png'); }

/* 7. font-display 控制字体加载策略 */
@font-face {
    font-family: 'MyFont';
    src: url('font.woff2') format('woff2');
    font-display: swap;  /* 避免字体阻塞渲染 */
}
```

#### 真实面试题

**题目：如何使用CSS提高页面性能?**

**满分答案：**

**CSS 性能优化按阶段分类：**

| 阶段 | 优化策略 | 方法 |
|------|---------|------|
| 传输 | 压缩 CSS、避免 @import | 构建工具、link 并行加载 |
| 解析 | 简化选择器 | 避免过深嵌套、避免通配符 |
| 渲染 | 关键 CSS 内联、减少重排重绘 | 首屏内联、transform 代替位置 |
| 动画 | GPU 加速、只动画 transform/opacity | will-change、transform |
| 请求 | CSS Sprites、字体优化 | 图片合并、font-display: swap |

**实际应用代码：**

```css
/* 1. 减少 @import，改用 <link> */
<link rel="stylesheet" href="base.css">
<link rel="stylesheet" href="components.css">

/* 2. 简化选择器 */
.nav-link { color: #333; }           /* ✅ 好 */
.nav ul li a.link { color: #333; }  /* ❌ 差 */

/* 3. 动画性能优化 */
@keyframes slide {
    from { transform: translateX(0); opacity: 1; }
    to { transform: translateX(100px); opacity: 0; }
}
.animated { will-change: transform; }

/* 4. contain 限制渲染范围 */
.list-item { contain: content; }
```

---

## 1.2.16 文本溢出省略

#### 知识点详解

**单行文本省略：**

```css
.single-line {
    overflow: hidden;           /* 隐藏溢出内容 */
    text-overflow: ellipsis;    /* 显示省略号 */
    white-space: nowrap;        /* 不换行 */
}
```

**多行文本省略（CSS3 -webkit-line-clamp）：**

```css
.multi-line {
    display: -webkit-box;           /* 使用弹性盒子 */
    -webkit-line-clamp: 3;          /* 限制行数 */
    -webkit-box-orient: vertical;   /* 垂直排列 */
    overflow: hidden;               /* 隐藏溢出 */
}
```

**现代方案（使用 line-clamp 简写）：**

```css
/* 标准属性，兼容性越来越好 */
.modern-multi-line {
    line-clamp: 3;        /* 限制3行 */
    overflow: hidden;
}
```

**JS 方案（兼容性考虑）：**

```javascript
function truncateText(element, maxLines) {
    const lineHeight = parseInt(getComputedStyle(element).lineHeight);
    const maxHeight = lineHeight * maxLines;
    
    while (element.scrollHeight > maxHeight) {
        element.textContent = element.textContent.slice(0, -1);
    }
}
```

**注意事项：**

- `-webkit-line-clamp` 需要配合 `display: -webkit-box` 使用
- 主流浏览器（Chrome、Firefox、Safari、Edge）已支持
- 移动端 Safari（iOS Safari 14.5+）已支持

#### 真实面试题

**题目：如何实现单行/多行文本溢出的省略样式?**

**满分答案：**

**单行省略（最常用）：**

```css
.text-ellipsis {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}
```

**多行省略（CSS3）：**

```css
.text-ellipsis-multi {
    display: -webkit-box;
    -webkit-line-clamp: 3;        /* 限制3行 */
    -webkit-box-orient: vertical;
    overflow: hidden;
}
```

**兼容性说明：**

- 单行省略：所有浏览器都支持
- 多行省略：`line-clamp` 是标准属性，主流浏览器已支持
- `-webkit-line-clamp` 是 WebKit 前缀，Safari/iOS 需要

**实际应用场景：**

```css
/* 标题省略 */
.card-title {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

/* 评论摘要省略 */
.comment-text {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
```

---

## 1.2.17 CSS 视差滚动效果

#### 知识点详解

**视差滚动原理：**

背景层移动速度慢，前景层移动速度快，造成视觉上的深度感。

**方案1：background-attachment: fixed（经典方案）：**

```css
.parallax-fixed {
    background-image: url('bg.jpg');
    background-attachment: fixed;  /* 背景固定，内容滚动 */
    background-size: cover;
    background-position: center;
    min-height: 100vh;
}
```

**方案2：transform: translateZ() + perspective（3D变换方案）：**

```css
.parallax-container {
    height: 100vh;
    overflow-x: hidden;
    overflow-y: scroll;
    perspective: 10px;  /* 透视深度 */
}

.parallax-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    transform: translateZ(-10px) scale(2);  /* 向后移动并放大 */
    z-index: -1;
}
```

**方案3：CSS Scroll-driven Animations（新规范）：**

```css
/* 利用滚动位置驱动动画，无需 JS */
@keyframes parallax-bg {
    from { transform: translateY(0); }
    to { transform: translateY(50%); }
}

.parallax-element {
    animation: parallax-bg linear;
    animation-timeline: scroll();
    /* 或 */
    animation-timeline: scroll(root block);
}
```

**JS 辅助方案（更灵活）：**

```javascript
// 监听滚动事件实现视差
const parallaxElements = document.querySelectorAll('.parallax');

window.addEventListener('scroll', () => {
    const scrollY = window.scrollY;
    parallaxElements.forEach(el => {
        const speed = el.dataset.speed || 0.5;
        el.style.transform = `translateY(${scrollY * speed}px)`;
    });
});
```

**性能注意事项：**

| 方案 | 性能 | 兼容性 | 适用场景 |
|------|------|--------|---------|
| background-attachment: fixed | 差（移动端） | 好 | 桌面端简单场景 |
| transform + perspective | 好（GPU） | 好 | 复杂视差效果 |
| CSS scroll-driven | 优 | 新规范 | 现代浏览器 |
| JS 监听 scroll | 中等 | 好 | 灵活控制 |

#### 真实面试题

**题目：如何使用CSS完成视差滚动效果?**

**满分答案：**

**三种 CSS 方案：**

**1. background-attachment: fixed（最简单）：**

```css
.hero {
    background-image: url('mountains.jpg');
    background-attachment: fixed;  /* 背景固定 */
    background-size: cover;
    height: 100vh;
}
```

**2. transform: translateZ() + perspective（推荐）：**

```css
.scene {
    perspective: 1px;
    overflow-x: hidden;
    overflow-y: scroll;
}

.layer-bg {
    transform: translateZ(-1px) scale(2);  /* 背景慢速移动 */
}

.layer-fg {
    transform: translateZ(0);  /* 前景正常速度 */
}
```

**3. CSS Scroll-driven Animations（新规范）：**

```css
@keyframes move {
    from { transform: translateY(0); }
    to { transform: translateY(100px); }
}
.element {
    animation: move linear;
    animation-timeline: scroll();
}
```

**选择建议：**

- **简单场景**：用 `background-attachment: fixed`
- **复杂效果**：用 `transform + perspective`（GPU 加速）
- **现代浏览器**：用 CSS scroll-driven animations

---

## 1.2.18 CSS 画三角形

#### 知识点详解

**原理：**

利用 border 的拼接特性。当元素的 width 和 height 为 0 时，四条 border 会从顶点向四个方向延伸，形成四个三角形。

**等腰三角形：**

```css
/* 上箭头三角形 */
.triangle-up {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 100px solid #333;
}

/* 下箭头三角形 */
.triangle-down {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-top: 100px solid #333;
}
```

**直角三角形：**

```css
/* 左下角三角形 */
.triangle-right-bottom {
    width: 0;
    height: 0;
    border-top: 50px solid transparent;
    border-left: 100px solid #333;
}

/* 右上角三角形 */
.triangle-right-top {
    width: 0;
    height: 0;
    border-bottom: 50px solid transparent;
    border-left: 100px solid #333;
}
```

**全方向三角形：**

```css
/* 上 */
border: 50px solid transparent;
border-bottom-color: #333;  /* 只保留底边 */

/* 下 */
border-bottom: none;
border-top: 50px solid #333;

/* 左 */
border-right: none;
border-left: 50px solid #333;

/* 右 */
border-left: none;
border-right: 50px solid #333;
```

**现代方案：clip-path（更灵活）：**

```css
/* clip-path polygon() 画任意三角形 */
.triangle-clip {
    width: 100px;
    height: 100px;
    background: #333;
    clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
    /* 参数：左上顶点、右下顶点、左下顶点 */
}

/* 等边三角形 */
.equilateral {
    width: 100px;
    height: 86.6px;  /* √3/2 * 边长 */
    background: #333;
    clip-path: polygon(50% 0%, 100% 100%, 0% 100%);
}
```

**带边框的三角形（CSS2方案）：**

```css
/* 使用伪元素实现边框效果 */
.triangle-with-border {
    position: relative;
    width: 0;
    height: 0;
}

.triangle-with-border::before {
    content: '';
    position: absolute;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 100px solid #666;  /* 外层边框色 */
}

.triangle-with-border::after {
    content: '';
    position: absolute;
    top: 4px;
    left: 50px;
    transform: translateX(-100%);
    border-left: 46px solid transparent;
    border-right: 46px solid transparent;
    border-bottom: 92px solid #fff;  /* 内层填充色 */
}
```

#### 真实面试题

**题目：怎么使用CSS画一个三角形**

**满分答案：**

**核心原理：**

将元素的 width 和 height 设为 0，利用 border 的拼接特性，将不同方向的 border 设为 transparent，有颜色的 border 就会呈现为三角形。

**等腰三角形代码：**

```css
/* 上箭头 */
.triangle-up {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-bottom: 100px solid #333;
}

/* 下箭头 */
.triangle-down {
    width: 0;
    height: 0;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-top: 100px solid #333;
}
```

**直角三角形代码：**

```css
.triangle-right-bottom {
    width: 0;
    height: 0;
    border-top: 50px solid transparent;
    border-left: 100px solid #333;
}
```

**现代方案（clip-path）：**

```css
/* 更灵活，支持任意形状 */
.triangle {
    width: 100px;
    height: 100px;
    background: #333;
    clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}
```

**实际应用场景：**

```css
/* 下拉框小箭头 */
.select::after {
    content: '';
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translateY(-50%);
    width: 0;
    height: 0;
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
    border-top: 5px solid #666;
}
```

---

## 1.2.19 CSS 工程化

#### 知识点详解

**CSS 工程化的核心问题：**

1. **全局作用域**：类名容易冲突
2. **无变量/逻辑**：无法复用和抽象
3. **重复代码**：相似样式需要重复写
4. **无法模块化**：难以大型项目维护

**主流解决方案：**

**1. 预处理器（Sass/Less/Stylus）：**

```scss
// Sass 示例
$primary-color: #007bff;
$spacing: 16px;

@mixin flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}

.card {
    background: $primary-color;
    padding: $spacing;
    
    &:hover {
        transform: scale(1.02);
    }
    
    .title {
        font-size: 18px;
    }
}
```

**2. PostCSS（插件生态）：**

```javascript
// postcss.config.js 示例
module.exports = {
    plugins: [
        'autoprefixer',    // 自动添加前缀
        'cssnano',          // 压缩 CSS
        'postcss-preset-env' // 现代 CSS 转换
    ]
};
```

**3. CSS Modules（局部作用域）：**

```css
/* Button.module.css */
.button { padding: 12px; }
.primary { background: blue; }
```

```jsx
// React 组件
import styles from './Button.module.css';
<button className={`${styles.button} ${styles.primary}`}>按钮</button>
/* 编译后类名：Button_button__hash */
```

**4. CSS-in-JS（组件化）：**

```jsx
// styled-components
import styled from 'styled-components';

const Button = styled.button`
    background: ${props => props.primary ? 'blue' : 'gray'};
    padding: 12px 24px;
    border-radius: 4px;
`;
```

```jsx
// emotion
import { css } from '@emotion/react';

const buttonStyle = css`
    background: blue;
    padding: 12px;
`;
<button css={buttonStyle}>按钮</button>
```

**5. 原子化 CSS（Tailwind CSS）：**

```html
<!-- 使用工具类组合样式 -->
<button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    按钮
</button>
```

**6. 设计系统/组件库：**

```css
/* CSS 自定义属性（设计令牌） */
:root {
    --color-primary: #1890ff;
    --spacing-base: 8px;
    --border-radius: 4px;
}

.button {
    background: var(--color-primary);
    padding: calc(var(--spacing-base) * 2);
    border-radius: var(--border-radius);
}
```

**技术选型建议：**

| 方案 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| Sass/Less | 中小型项目 | 上手简单、功能强大 | 仍全局作用域 |
| CSS Modules | React 项目 | 局部作用域、零冲突 | 类名不直观 |
| CSS-in-JS | React/组件库 | 组件化、动态样式强 | 运行时开销 |
| Tailwind | 快速开发 | 开发效率高、一致性好 | 需学习成本 |
| PostCSS | 任何项目 | 插件生态、可组合 | 配置复杂 |

#### 真实面试题

**题目：说说对CSS工程化的理解**

**满分答案：**

**核心问题：**

CSS 本身存在三大不足：
1. **无作用域**：全局类名容易冲突
2. **无变量/逻辑**：无法复用、难以维护
3. **无模块化**：大型项目难以管理

**主流解决方案对比：**

| 方案 | 原理 | 代表技术 | 适用场景 |
|------|------|---------|---------|
| 预处理器 | CSS 超集，编译为 CSS | Sass、Less、Stylus | 中小型项目 |
| PostCSS | CSS 转换插件生态 | autoprefixer、CSSnano | 任何项目 |
| CSS Modules | 构建时生成唯一类名 | CSS Modules | React 项目 |
| CSS-in-JS | JS 中写 CSS | styled-components | React 组件库 |
| 原子化 CSS | 工具类组合 | Tailwind CSS | 快速开发 |

**实际项目推荐：**

```css
/* 大型项目：CSS Modules + Sass */
.button { @include flex-center; }
.button--primary { background: var(--primary); }

/* 组件库：CSS-in-JS */
/* styled-components 或 emotion */

/* 快速开发：Tailwind CSS */
/* utility classes, JIT mode */
```

---

## 1.2.20 BFC 触发与应用

#### 知识点详解

**BFC 定义：**

BFC（Block Formatting Context，块级格式化上下文）是一个独立的渲染区域，内部子元素的布局不会影响外部，外部也不会影响内部。

**触发 BFC 的条件（满足任一即可）：**

```css
/* 1. overflow 不为 visible */
.bfc { overflow: hidden; }
.bfc { overflow: auto; }
.bfc { overflow: scroll; }

/* 2. display 为 flow-root（推荐，语义最明确） */
.bfc { display: flow-root; }

/* 3. display 为 flex/grid */
.bfc { display: flex; }
.bfc { display: grid; }

/* 4. position 为 absolute/fixed */
.bfc { position: absolute; }
.bfc { position: fixed; }

/* 5. float 不为 none */
.bfc { float: left; }
```

**BFC 三大应用场景：**

**场景1：清除浮动（防止父元素塌陷）**

```css
/* ❌ 不创建 BFC：父元素高度塌陷 */
.parent { border: 1px solid red; }
.child { float: left; }

/* ✅ 创建 BFC：父元素包含浮动元素 */
.parent {
    display: flow-root;  /* 推荐 */
    /* 或 */
    overflow: hidden;
    /* 或 */
    float: left; width: 100%;
}
```

**场景2：防止 margin 合并**

```css
/* ❌ 不创建 BFC：父子 margin 合并 */
.parent { margin-top: 20px; }
.child { margin-top: 30px; }  /* 合并为 30px */

/* ✅ 创建 BFC：阻止 margin 合并 */
.parent {
    display: flow-root;  /* 语义最明确 */
    /* 或 */
    overflow: hidden;
}
.child { margin-top: 30px; }  /* 不合并 */
```

**场景3：自适应两栏布局（左侧浮动 + 右侧 BFC）**

```css
.container { overflow: hidden; }  /* 创建 BFC */

.sidebar {
    float: left;
    width: 200px;
}

.main {
    overflow: hidden;  /* 创建 BFC，不与浮动重叠 */
}
```

**BFC 布局规则：**

- BFC 内部的 Box 从顶部开始垂直排列
- 同一个 BFC 内相邻 Box 的 margin 会合并
- BFC 内部不会影响外部布局
- BFC 的区域与外部浮动元素不重叠

#### 真实面试题

**题目：怎么触发BFC，BFC有什么应用场景?**

**满分答案：**

**什么是 BFC：**

BFC（Block Formatting Context，块级格式化上下文）是一个独立的渲染区域，内部元素的布局不会影响外部。

**触发条件（5种）：**

```css
/* 1. overflow 不为 visible（hidden/auto/scroll） */
.box { overflow: hidden; }

/* 2. display: flow-root（推荐，语义最明确） */
.box { display: flow-root; }

/* 3. display: flex / grid / inline-flex / inline-grid */
.box { display: flex; }

/* 4. position: absolute / fixed */
.box { position: absolute; }

/* 5. float 不为 none */
.box { float: left; }
```

**三大应用场景：**

**1. 清除浮动（防止父元素高度塌陷）：**

```css
.clearfix {
    display: flow-root;  /* 推荐方式 */
}
```

**2. 防止 margin 合并：**

```css
.parent {
    display: flow-root;  /* 阻止父子 margin 合并 */
}
.child {
    margin-top: 30px;
}
```

**3. 自适应两栏布局：**

```css
.sidebar {
    float: left;
    width: 200px;
}

.main {
    overflow: hidden;  /* 创建 BFC，不与浮动重叠 */
}
```

---
