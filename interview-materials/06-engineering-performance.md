# 工程化与性能

> 面试频率：⭐⭐⭐⭐

---

## 一、Webpack

### 1.1 核心概念

| 概念 | 说明 |
|------|------|
| **Entry** | 入口文件 |
| **Output** | 输出配置 |
| **Loader** | 处理非 JS 文件 |
| **Plugin** | 扩展功能 |
| **Chunk** | 代码块 |
| **Module** | 模块 |

### 1.2 Loader vs Plugin

| 类型 | 作用 | 例子 |
|------|------|------|
| Loader | 转换特定类型文件 | babel-loader, css-loader, file-loader |
| Plugin | 打包优化、资源管理 | HtmlWebpackPlugin, MiniCssExtractPlugin, DefinePlugin |

### 1.3 构建流程

```
1. 初始化参数 (webpack.config.js)
2. 开始编译 (run)
3. 编译模块 (build module)
   - 读取文件内容
   - 调用 Loader 转换
   - 解析依赖
4. 输出模块 (seal)
   - 合并 chunks
   - 生成 chunk assets
5. 输出文件 (emit)
```

### 1.4 代码分割 (Code Splitting)

```js
// 方式1: 动态 import
import('./module').then(module => {
  module.default();
});

// 方式2: React.lazy
const Component = React.lazy(() => import('./Component'));

// 方式3: webpack 配置
optimization: {
  splitChunks: {
    chunks: 'all',  // all/async/initial
    cacheGroups: {
      vendors: {
        test: /[\\/]node_modules[\\/]/,
        priority: -10
      },
      default: {
        minChunks: 2,
        priority: -20
      }
    }
  }
}
```

### 1.5 Tree Shaking

> 移除未使用的代码（ES Module）

```js
// webpack.config.js
module.exports = {
  mode: 'production',  // 自动开启
  optimization: {
    usedExports: true,
    sideEffects: false  // 标记无副作用
  }
};
```

**注意**：
- 只对 ES Module 有效
- `sideEffects: false` 告诉 webpack 可以安全删除
- 常见误区：`import './style.css'` 可能被 tree shake 掉（需配置 `sideEffects`）

---

## 二、Vite

### 2.1 原理

| 特性 | Webpack | Vite |
|------|---------|------|
| 开发模式 | 打包启动 | ESM 按需加载 |
| 热更新 | 重新打包 | ESM + ES Build |
| 生产模式 | 打包 | Rollup 打包 |

### 2.2 Vite HMR 原理

```
1. ESM 加载模块
2. 浏览器请求文件
3. Vite 拦截请求，转换后返回
4. 模块变化时，通知相关模块更新
5. 浏览器重新请求更新的模块
```

### 2.3 Vite 配置示例

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': '/src'
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': 'http://localhost:8080'
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom']
        }
      }
    }
  }
});
```

---

## 三、性能优化

### 3.1 Core Web Vitals

| 指标 | 全称 | 含义 | 优秀标准 |
|------|------|------|----------|
| **LCP** | Largest Contentful Paint | 最大内容绘制 | < 2.5s |
| **FID** | First Input Delay | 首次输入延迟 | < 100ms |
| **CLS** | Cumulative Layout Shift | 累计布局偏移 | < 0.1 |

### 3.2 首屏加载优化

| 方案 | 说明 |
|------|------|
| **Code Splitting** | 路由懒加载 |
| **CDN** | 静态资源放 CDN |
| **Gzip/Brotli** | 压缩资源 |
| **SSR/SSG** | 服务端渲染 |
| **预加载** | `<link rel="preload">` |
| **骨架屏** | loading 占位 |

### 3.3 懒加载

```js
// 图片懒加载
<img loading="lazy" src="image.jpg" />

// 路由懒加载
const Home = () => import('./Home');
const About = () => import('./About');

// 组件懒加载
const Modal = React.lazy(() => import('./Modal'));
```

### 3.4 React 性能优化

```jsx
// 1. React.memo
const Button = React.memo(({ onClick }) => <button onClick={onClick}>click</button>);

// 2. useMemo / useCallback
const memoizedValue = useMemo(() => compute(a, b), [a, b]);
const memoizedFn = useCallback(() => doSomething(a), [a]);

// 3. 虚拟列表
import { FixedSizeList } from 'react-window';
```

### 3.5 Vue 性能优化

```js
// 1. v-show vs v-if
// 频繁切换用 v-show

// 2. computed 缓存
computed: {
  cachedValue() {
    return this.a + this.b; // 依赖不变不重算
  }
}

// 3. 虚拟滚动
import VueVirtualScroller from 'vue-virtual-scroller';
```

---

## 四、安全

### 4.1 XSS 防御

```js
// 转义 HTML
function escapeHTML(str) {
  return str.replace(
    /[&<>"']/g,
    tag => ({
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#39;'
    }[tag])
  );
}

// CSP 配置
// <meta http-equiv="Content-Security-Policy" 
//       content="default-src 'self'; script-src 'self' 'unsafe-inline'">
```

### 4.2 CSRF 防御

```js
// 1. CSRF Token
// 服务器生成 token，表单中携带
// <input type="hidden" name="csrf_token" value="xxx">

// 2. SameSite Cookie
Set-Cookie: session=xxx; SameSite=Strict
// Strict: 完全禁止跨域发送
// Lax: 导航请求允许（GET 链接、预加载、GET 表单）
// None: 允许跨域（需 Secure）
```

---

## 五、构建工具对比

| 特性 | Webpack | Vite | Rollup | esbuild |
|------|---------|------|--------|---------|
| 定位 | 通用 | 开发+生产 | 库打包 | 极速打包 |
| 速度 | 慢 | 快 | 中 | 极快 |
| HMR | 重打包 | 原生 ESM | 插件 | 原生 |
| 生态 | 丰富 | 增长中 | 库作者首选 | 新兴 |

---

## 六、常见工程问题

### 6.1 如何排查性能问题

1. **Chrome DevTools Performance** - 录制分析
2. **Lighthouse** - 性能评分
3. **React DevTools** - 组件渲染分析
4. **webpack-bundle-analyzer** - 打包体积分析

### 6.2 包体积过大怎么办

1. `webpack-bundle-analyzer` 分析
2. 按需加载（路由、组件）
3. Tree Shaking
4. 压缩 (Terser)
5. CDN
6. 删除无用依赖

### 6.3 CI/CD 优化

- 缓存 node_modules
- 并行构建
- 增量构建
- Docker 多阶段构建

---

> 📌 下一章：[算法与数据结构](./07-algorithm-datastructure.md)
