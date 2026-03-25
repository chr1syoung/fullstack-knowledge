# React 面试题

> 面试频率：⭐⭐⭐⭐⭐

---

## 一、组件基础

### 1.1 函数组件 vs 类组件

| 特性 | 函数组件 | 类组件 |
|------|----------|--------|
| 语法 | 函数 | ES6 class |
| 状态 | useState | this.state |
| 生命周期 | useEffect | componentDidMount 等 |
| this | 不需要 | 需要 bind |
| 性能 | 稍快（无类的开销） | 稍慢 |
| 推荐 | ✅ 官方推荐 | ⚠️ 不推荐新代码使用 |

```jsx
// 函数组件
function Welcome({ name }) {
  return <h1>Hello, {name}</h1>;
}

// 类组件
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### 1.2 受控组件 vs 非受控组件

| 类型 | 特点 | 例子 |
|------|------|------|
| 受控组件 | 状态由 React 控制，value 受 state 驱动 | `<input value={text} onChange={...} />` |
| 非受控组件 | 状态由 DOM 自身管理 | `<input ref={ref} />` |

```jsx
// 受控组件
const [value, setValue] = useState('');
<input value={value} onChange={e => setValue(e.target.value)} />

// 非受控组件
const ref = useRef();
<input ref={ref} defaultValue="initial" />
<button onClick={() => console.log(ref.current.value)}>获取</button>
```

---

## 二、状态管理

### 2.1 useState vs useReducer

| 场景 | 推荐 | 原因 |
|------|------|------|
| 简单状态（1-2个） | useState | 代码简洁 |
| 复杂状态（多个相关） | useReducer | 逻辑集中，易于测试 |

```jsx
// useState
const [count, setCount] = useState(0);

// useReducer
const [state, dispatch] = useReducer((state, action) => {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    default: return state;
  }
}, { count: 0 });
```

### 2.2 Context 使用场景

> 适用于：主题、用户信息、语言等**全局共享**的状态

```jsx
const ThemeContext = React.createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const theme = useContext(ThemeContext); // 'dark'
  return <div className={theme}>...</div>;
}
```

**⚠️ 注意点**：Context 会导致所有消费组件重渲染，使用时注意拆分

### 2.3 Redux 原理

```
Action → Dispatch → Reducer → Store → View
```

```js
// action
const add = (n) => ({ type: 'ADD', payload: n });

// reducer
const counter = (state = 0, action) => {
  switch (action.type) {
    case 'ADD': return state + action.payload;
    default: return state;
  }
};

// store
import { createStore } from 'redux';
const store = createStore(counter);

// view
store.dispatch(add(1));
store.getState();
```

---

## 三、生命周期与 Hooks

### 3.1 useEffect 详解

```jsx
useEffect(() => {
  // 1. 组件挂载后执行
  console.log('mount');
  
  return () => {
    // 2. 组件卸载前执行（清理函数）
    console.log('cleanup');
  };
}, [deps]); // 依赖数组
```

| deps | 行为 |
|------|------|
| `[]` | 仅挂载时执行一次（类似 componentDidMount） |
| `[count]` | count 变化时执行 |
| 不传 | 每次渲染都执行（⚠️ 可能导致无限循环） |

### 3.2 useLayoutEffect vs useEffect

| 特性 | useEffect | useLayoutEffect |
|------|-----------|-----------------|
| 执行时机 | 渲染后，浏览器 paint 前 | 渲染前，同步执行 |
| 适用场景 | 数据获取、订阅 | DOM 测量、调整 |
| 阻塞渲染 | 否 | 是 |

```jsx
// DOM 测量用 useLayoutEffect
useLayoutEffect(() => {
  const rect = ref.current.getBoundingClientRect();
  console.log(rect.height);
}, []);
```

### 3.3 useRef 用法

```jsx
// 1. 存储不需要触发渲染的值
const count = useRef(0);

// 2. 获取 DOM 引用
const inputRef = useRef(null);
<input ref={inputRef} />;
<button onClick={() => inputRef.current.focus()}>聚焦</button>;

// 3. 保存上一次的 value
const prevCount = useRef();
useEffect(() => {
  prevCount.current = count;
}, [count]);
```

---

## 四、渲染优化

### 4.1 React.memo

> 缓存组件渲染结果，props 不变时不重渲染

```jsx
const Button = React.memo(({ onClick, children }) => {
  console.log('Button render');
  return <button onClick={onClick}>{children}</button>;
});
```

### 4.2 useMemo vs useCallback

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
const memoizedCallback = useCallback(() => doSomething(a), [a]);
```

| Hook | 作用 | 适用场景 |
|------|------|----------|
| useMemo | 缓存计算结果 | 复杂计算、对象/数组字面量 |
| useCallback | 缓存函数 | 传给子组件的回调（防重渲染） |

### 4.3 虚拟 DOM 与 Diff 算法

**React 核心**：维护虚拟 DOM 树，变化时高效更新真实 DOM

**Diff 策略**：
1. 同层比较，不跨层级
2. 类型不同 → 销毁重建
3. key 相同且类型相同 → 复用节点

### 4.4 key 的作用

> key 帮助 React 识别哪些元素变化了

```jsx
// ❌ 错误：使用 index 作为 key
items.map((item, i) => <Item key={i} {...item} />)

// ✅ 正确：使用稳定唯一 ID
items.map(item => <Item key={item.id} {...item} />)
```

---

## 五、原理深入

### 5.1 Fiber 架构

> React 16 引入，目标是**可中断的异步渲染**

**核心概念**：
- Fiber = 工作单元（Unit of Work）
- 双缓冲：current tree ↔ workInProgress tree
- 优先级调度：高优先级任务可以中断低优先级

### 5.2 Concurrent Mode

```jsx
<React.concurrent mode>
  <App />
</React.concurrent mode>
```

开启后：
- `useTransition`: 标记非紧急更新
- `useDeferredValue`: 延迟非关键渲染

```jsx
const [query, setQuery] = useState('');
const [deferredQuery] = useDeferredValue(query);

// 高优先级（用户输入）
<input value={query} onChange={e => setQuery(e.target.value)} />

// 低优先级（搜索结果）
{deferredQuery && <SearchResults query={deferredQuery} />}
```

### 5.3 Reconciliation（协调）

> 虚拟 DOM diff 过程

**React 调度顺序**：
1. **Render 阶段**：生成 Fiber 树，可中断
2. **Commit 阶段**：同步更新 DOM，不可中断

---

## 六、自定义 Hook

### 6.1 经典例子：useDebounce

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// 使用
const debouncedSearch = useDebounce(search, 300);
useEffect(() => {
  if (debouncedSearch) fetchData(debouncedSearch);
}, [debouncedSearch]);
```

### 6.2 闭包陷阱

```jsx
// ❌ 常见错误：定时器读到旧的 state
const [count, setCount] = useState(0);
useEffect(() => {
  const timer = setInterval(() => {
    console.log(count); // 永远是 0
  }, 1000);
  return () => clearInterval(timer);
}, []); // 空依赖，只执行一次

// ✅ 正确做法：使用 ref 或函数式更新
const [count, setCount] = useState(0);
useEffect(() => {
  const timer = setInterval(() => {
    setCount(c => c + 1); // 使用函数式更新
  }, 1000);
  return () => clearInterval(timer);
}, []);
```

---

## 七、常见追问

1. **React 18 更新了什么？**
   - Automatic Batching（自动批处理）
   - Concurrent Features（并发模式）
   - 新 Hook: `useId`、`useTransition`、`useDeferredValue`
   - Suspense 改进

2. **setState 是同步还是异步？**
   - 在 React 事件中：异步（批处理）
   - 在 setTimeout / 原生事件中：同步
   - 使用 `flushSync` 强制同步

3. **如何设计一个好的组件？**
   - 单一职责
   -  Props 最小化
   -  避免不必要的重渲染
   -  合理拆分（容器组件 vs 展示组件）

---

> 📌 下一章：[Vue 面试题](./03-vue.md)
