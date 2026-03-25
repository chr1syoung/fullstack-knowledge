# 场景题

> 面试频率：⭐⭐⭐⭐（高级/架构岗必考）

---

## 一、搜索框设计

### 1.1 需求分析

- 输入防抖（避免每次输入都请求）
- 搜索联想（自动补全）
- 历史搜索记录
- 缓存机制
- 关键词高亮

### 1.2 核心实现

```jsx
function Search() {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [history, setHistory] = useState([]);
  
  // 防抖处理
  const debouncedQuery = useDebounce(query, 300);
  
  // 搜索建议
  useEffect(() => {
    if (debouncedQuery) {
      fetchSuggestions(debouncedQuery).then(setSuggestions);
    }
  }, [debouncedQuery]);
  
  // 缓存实现（LRU）
  const cache = useRef(new LRUCache(100));
  
  const handleSearch = async (keyword) => {
    // 1. 检查缓存
    const cached = cache.current.get(keyword);
    if (cached) {
      setResults(cached);
      return;
    }
    
    // 2. 请求
    const data = await fetchResults(keyword);
    
    // 3. 存入缓存
    cache.current.set(keyword, data);
    
    // 4. 更新历史
    const newHistory = [keyword, ...history.filter(k => k !== keyword)].slice(0, 10);
    localStorage.setItem('search_history', JSON.stringify(newHistory));
  };
  
  // 关键词高亮
  const highlight = (text, keyword) => {
    if (!keyword) return text;
    const parts = text.split(new RegExp(`(${keyword})`, 'g'));
    return parts.map((part, i) => 
      part === keyword ? <mark key={i}>{part}</mark> : part
    );
  };
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <div className="suggestions">
        {suggestions.map(s => (
          <div onClick={() => handleSearch(s)}>
            {highlight(s, query)}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 1.3 追问点

| 问题 | 答案 |
|------|------|
| 如何处理高并发？ | CDN、限流、熔断 |
| 如何做前端缓存？ | LRU + localStorage |
| 联想服务怎么做？ |  Trie 树 / 前缀匹配 |

---

## 二、大文件上传

### 2.1 需求

- 支持 GB 级别文件
- 断点续传
- 上传进度
- 并行上传加速

### 2.2 核心实现

```js
class Uploader {
  constructor(file, options = {}) {
    this.file = file;
    this.chunkSize = options.chunkSize || 2 * 1024 * 1024; // 2MB
    this.threads = options.threads || 4;
    this.uploadedChunks = new Set(); // 已上传的分片
  }
  
  // 分片
  getChunks() {
    const chunks = [];
    let start = 0;
    while (start < this.file.size) {
      const end = Math.min(start + this.chunkSize, this.file.size);
      chunks.push({
        index: chunks.length,
        start,
        end,
        blob: this.file.slice(start, end)
      });
      start = end;
    }
    return chunks;
  }
  
  // 上传单个分片
  async uploadChunk(chunk) {
    const formData = new FormData();
    formData.append('chunk', chunk.blob);
    formData.append('index', chunk.index);
    formData.append('filename', this.file.name);
    
    return fetch('/api/upload', {
      method: 'POST',
      body: formData
    });
  }
  
  // 并行上传
  async upload(onProgress) {
    const chunks = this.getChunks();
    let uploaded = 0;
    
    // 从服务器获取已上传分片
    await this.getUploadedChunks();
    
    // 并行上传
    const promises = chunks
      .filter(c => !this.uploadedChunks.has(c.index))
      .map(chunk => this.uploadChunk(chunk));
    
    await Promise.all(promises);
    
    // 通知合并
    await this.mergeChunks();
  }
  
  // 断点续传：查询已上传
  async getUploadedChunks() {
    const res = await fetch(`/api/uploaded?filename=${this.file.name}`);
    const uploaded = await res.json();
    this.uploadedChunks = new Set(uploaded);
  }
  
  // 合并分片
  async mergeChunks() {
    return fetch('/api/merge', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        filename: this.file.name,
        total: this.getChunks().length
      })
    });
  }
}
```

### 2.3 追问点

| 问题 | 答案 |
|------|------|
| 切片大小多少合适？ | 1-10MB，太小请求多，太大失败重传成本高 |
| 如何保证顺序？ | 每个分片带 index，服务器按顺序合并 |
| 失败怎么办？ | 重试已失败的 chunk，不重复上传成功的 |

---

## 三、页面卡顿排查

### 3.1 排查工具

| 工具 | 用途 |
|------|------|
| Chrome DevTools Performance | 录制分析 |
| Performance API | 代码中埋点 |
| React DevTools | 组件渲染分析 |
| Lighthouse | 整体性能评分 |

### 3.2 排查步骤

```
1. 打开 Performance 面板
2. 录制页面加载过程
3. 查看 FPS 曲线（低于 60fps 有问题）
4. 查看 Main 线程火焰图
5. 定位长任务（Long Tasks）
6. 分析具体函数调用栈
```

### 3.3 常见原因与解决方案

| 原因 | 解决方案 |
|------|----------|
| 大量 JS 执行 | 代码分割、懒加载 |
| 频繁重渲染 | React.memo/useMemo |
| 频繁 DOM 操作 | 虚拟列表、减少 reflow |
| 大图片 | 压缩、懒加载、WebP |
| 内存泄漏 | 检查未清理的定时器/事件 |

### 3.4 内存泄漏检测

```js
// 记录堆快照对比
console.log(performance.memory);

// 监控内存变化
setInterval(() => {
  console.log(performance.memory.usedJSHeapSize);
}, 1000);

// Chrome DevTools:
// Memory → Take heap snapshot → 对比前后对象
```

---

## 四、Promise 重试机制

### 4.1 需求

- 失败后自动重试
- 可配置重试次数、间隔
- 指数退避

### 4.2 实现

```js
function retry(fn, options = {}) {
  const { 
    retries = 3, 
    delay = 1000, 
    backoff = 2,
    onRetry = () => {} 
  } = options;
  
  return new Promise(async (resolve, reject) => {
    let lastError;
    
    for (let i = 0; i <= retries; i++) {
      try {
        return resolve(await fn());
      } catch (e) {
        lastError = e;
        
        if (i < retries) {
          const waitTime = delay * Math.pow(backoff, i);
          onRetry(i + 1, waitTime, e);
          await new Promise(r => setTimeout(r, waitTime));
        }
      }
    }
    reject(lastError);
  });
}

// 使用
retry(() => fetch('/api'), {
  retries: 3,
  delay: 1000,
  backoff: 2,
  onRetry: (n, wait, e) => console.log(`第 ${n} 次重试，${wait}ms 后`)
});
```

---

## 五、EventBus 设计

### 5.1 需求

- 事件发布/订阅
- 支持 once、off
- 命名空间
- 类型推断（可选）

### 5.2 实现

```js
class EventBus {
  constructor() {
    this.events = new Map();
  }
  
  on(event, callback) {
    if (!this.events.has(event)) {
      this.events.set(event, new Set());
    }
    this.events.get(event).add(callback);
    
    // 返回取消订阅函数
    return () => this.off(event, callback);
  }
  
  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
  
  off(event, callback) {
    if (callback) {
      this.events.get(event)?.delete(callback);
    } else {
      this.events.delete(event);
    }
  }
  
  emit(event, ...args) {
    this.events.get(event)?.forEach(cb => cb(...args));
  }
  
  // 命名空间
  onNamespace(ns, event, callback) {
    return this.on(`${ns}:${event}`, callback);
  }
}

// TypeScript 版本
interface EventBus {
  on<T extends any[]>(event: string, cb: (...args: T) => void): () => void;
  emit(event: string, ...args: any[]): void;
}
```

---

## 六、设计一个 Loading 组件

### 6.1 需求

- 支持多种样式（spin/dots/pulse）
- 可自定义大小、颜色
- 全屏/内联模式

### 6.2 实现

```jsx
function Loading({ 
  type = 'spin', 
  size = 'medium', 
  fullscreen = false,
  color = '#333' 
}) {
  const sizeMap = { small: 16, medium: 24, large: 40 };
  const px = sizeMap[size] || size;
  
  const style = { 
    width: px, 
    height: px, 
    color,
    ...(fullscreen && { position: 'fixed', inset: 0 })
  };
  
  return (
    <div className={`loading ${fullscreen ? 'fullscreen' : ''}`}>
      {type === 'spin' && <Spin style={style} />}
      {type === 'dots' && <Dots style={style} />}
      {type === 'pulse' && <Pulse style={style} />}
    </div>
  );
}
```

---

## 七、无限滚动列表

### 7.1 核心思路

- 可视区域只渲染可见元素
- 监听滚动，动态计算渲染范围
- 使用 absolute 定位

### 7.2 实现

```jsx
function VirtualList({ 
  items, 
  itemHeight = 50, 
  containerHeight = 400 
}) {
  const [scrollTop, setScrollTop] = useState(0);
  
  // 计算可见范围
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(
    items.length - 1,
    Math.floor((scrollTop + containerHeight) / itemHeight) + 5
  );
  
  // 渲染的数据（前后各多渲染几个，防闪烁）
  const visibleItems = items.slice(
    Math.max(0, startIndex - 5),
    endIndex + 1
  );
  
  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };
  
  return (
    <div 
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: items.length * itemHeight, position: 'relative' }}>
        {visibleItems.map((item, i) => (
          <div
            key={item.id}
            style={{
              position: 'absolute',
              top: (startIndex - 5 + i) * itemHeight,
              height: itemHeight
            }}
          >
            {item.content}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 7.3 追问

| 问题 | 答案 |
|------|------|
| 性能优化？ | 虚拟滚动 + requestAnimationFrame |
| 动态高度？ | 预估高度 + 测量修正 |
| 库推荐？ | react-virtualized、react-window |

---

## 八、扫码登录设计

### 8.1 流程

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   电脑端    │     │   手机端    │     │   服务器    │
│  生成二维码 │────→│   扫码     │────→│  验证登录   │
│  (包含 token)│     │ (携带 token)│     │  返回 token │
│             │     │             │←────│             │
│  轮询状态   │────→│  确认       │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 8.2 核心实现

```js
// 电脑端：生成二维码
async function createQRCode() {
  const { token } = await fetch('/api/qr/token').then(r => r.json());
  return generateQRCode(`login://${token}`);
}

// 电脑端：轮询状态
function checkStatus(token) {
  return setInterval(async () => {
    const { status, userInfo } = await fetch(`/api/qr/status?token=${token}`);
    
    if (status === 'confirmed') {
      // 登录成功，获取 token
      localStorage.setItem('token', userInfo.token);
      clearInterval(checkStatus);
    } else if (status === 'expired') {
      // 二维码过期
      clearInterval(checkStatus);
    }
  }, 2000);
}

// 手机端：扫码
function scanQRCode(token) {
  // 调起手机相机扫码
  // 获得 token 后确认登录
  return fetch('/api/qr/confirm', {
    method: 'POST',
    body: JSON.stringify({ token, userId: currentUser.id })
  });
}
```

---

## 九、场景题回答技巧

### 9.1 回答框架

```
1. 需求澄清 - 先确认具体场景、用户量、数据规模
2. 方案设计 - 给出整体架构
3. 技术选型 - 为什么选这个技术
4. 核心实现 - 关键代码/思路
5. 扩展考虑 - 性能、安全、容错
```

### 9.2 加分项

- 考虑边界情况（失败、重试、超时）
- 提到监控和日志
- 想到前后端协作
- 有量化思维（预估 QPS、延迟）

---

> 📚 面试资料库整理完毕！祝面试顺利 🚀
