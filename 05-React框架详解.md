# 五、React 框架详解

---

## 5.1 React 基础

### 5.1.1 JSX 语法

#### 知识点详解

JSX 是 JavaScript 的语法扩展，允许在 JavaScript 中编写类似 HTML 的代码。

```jsx
// 基本 JSX
const element = <h1>Hello, World!</h1>;

// JSX 表达式
const name = 'John';
const element = <h1>Hello, {name}!</h1>;

// JSX 属性
const element = <img src={user.avatarUrl} alt={user.name} />;

// 条件渲染
const element = (
    <div>
        {isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
        {showMessage && <p>消息内容</p>}
    </div>
);

// 列表渲染
const items = ['Apple', 'Banana', 'Cherry'];
const list = (
    <ul>
        {items.map((item, index) => (
            <li key={index}>{item}</li>
        ))}
    </ul>
);

// 样式
const style = { color: 'red', fontSize: '16px' };
const element = <div style={style}>样式</div>;
const element = <div style={{ color: 'red' }}>内联样式</div>;

// 类名
const element = <div className="container active">类名</div>;

// 事件处理
const element = (
    <button onClick={(e) => console.log(e)}>
        点击
    </button>
);

// Fragment（避免多余的 DOM 节点）
const element = (
    <>
        <h1>标题</h1>
        <p>段落</p>
    </>
);

// 或者
const element = (
    <React.Fragment>
        <h1>标题</h1>
        <p>段落</p>
    </React.Fragment>
);
```

**JSX 编译原理：**

```jsx
// JSX
const element = <h1 className="greeting">Hello</h1>;

// 编译后（React 17 之前）
const element = React.createElement(
    'h1',
    { className: 'greeting' },
    'Hello'
);

// React 17+ 新 JSX 转换
import { jsx as _jsx } from 'react/jsx-runtime';
const element = _jsx('h1', { className: 'greeting', children: 'Hello' });
```

### 5.1.2 函数组件与类组件

**函数组件（推荐）：**

```jsx
// 基本函数组件
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}

// 箭头函数
const Welcome = ({ name, age = 18 }) => {
    return (
        <div>
            <h1>Hello, {name}</h1>
            <p>Age: {age}</p>
        </div>
    );
};

// 带 TypeScript
interface WelcomeProps {
    name: string;
    age?: number;
}

const Welcome: React.FC<WelcomeProps> = ({ name, age = 18 }) => {
    return <h1>Hello, {name}, {age}</h1>;
};
```

**类组件：**

```jsx
class Welcome extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        };
        // 绑定 this
        this.handleClick = this.handleClick.bind(this);
    }
    
    handleClick() {
        this.setState({ count: this.state.count + 1 });
    }
    
    // 或使用箭头函数（自动绑定 this）
    handleClick = () => {
        this.setState(prevState => ({
            count: prevState.count + 1
        }));
    };
    
    componentDidMount() {
        console.log('组件已挂载');
    }
    
    componentDidUpdate(prevProps, prevState) {
        if (prevState.count !== this.state.count) {
            console.log('count 变化了');
        }
    }
    
    componentWillUnmount() {
        console.log('组件将卸载');
    }
    
    render() {
        return (
            <div>
                <p>Count: {this.state.count}</p>
                <button onClick={this.handleClick}>+1</button>
            </div>
        );
    }
}
```

### 5.1.3 Hooks 详解

**useState：**

```jsx
import { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    const [user, setUser] = useState({ name: 'John', age: 30 });
    
    // 更新基本类型
    const increment = () => setCount(count + 1);
    const incrementFn = () => setCount(prev => prev + 1);  // 函数式更新
    
    // 更新对象（必须创建新对象）
    const updateName = () => setUser({ ...user, name: 'Jane' });
    
    // 惰性初始化（避免每次渲染都执行）
    const [state] = useState(() => {
        return expensiveComputation();
    });
    
    return (
        <div>
            <p>{count}</p>
            <button onClick={increment}>+1</button>
        </div>
    );
}
```

**useEffect：**

```jsx
import { useEffect, useState } from 'react';

function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    
    // 1. 无依赖数组：每次渲染后执行
    useEffect(() => {
        console.log('每次渲染后执行');
    });
    
    // 2. 空依赖数组：只在挂载时执行
    useEffect(() => {
        console.log('只在挂载时执行');
        
        return () => {
            console.log('卸载时执行（清理函数）');
        };
    }, []);
    
    // 3. 有依赖：依赖变化时执行
    useEffect(() => {
        let cancelled = false;
        
        async function fetchUser() {
            const data = await fetch(`/api/user/${userId}`).then(r => r.json());
            if (!cancelled) {
                setUser(data);
            }
        }
        
        fetchUser();
        
        return () => {
            cancelled = true;  // 清理：防止组件卸载后设置状态
        };
    }, [userId]);
    
    // 4. 订阅/取消订阅
    useEffect(() => {
        const subscription = someObservable.subscribe(handler);
        
        return () => {
            subscription.unsubscribe();
        };
    }, []);
    
    // 5. 事件监听
    useEffect(() => {
        const handleResize = () => {
            console.log(window.innerWidth);
        };
        
        window.addEventListener('resize', handleResize);
        
        return () => {
            window.removeEventListener('resize', handleResize);
        };
    }, []);
    
    return <div>{user?.name}</div>;
}
```

**useContext：**

```jsx
import { createContext, useContext, useState } from 'react';

// 创建 Context
const ThemeContext = createContext('light');
const UserContext = createContext(null);

// Provider
function App() {
    const [theme, setTheme] = useState('light');
    const [user, setUser] = useState({ name: 'John' });
    
    return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
            <UserContext.Provider value={{ user, setUser }}>
                <MainContent />
            </UserContext.Provider>
        </ThemeContext.Provider>
    );
}

// Consumer
function ThemedButton() {
    const { theme, setTheme } = useContext(ThemeContext);
    
    return (
        <button
            style={{ background: theme === 'dark' ? '#333' : '#fff' }}
            onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
        >
            Toggle Theme
        </button>
    );
}
```

**useReducer：**

```jsx
import { useReducer } from 'react';

// 定义 reducer
function counterReducer(state, action) {
    switch (action.type) {
        case 'increment':
            return { count: state.count + 1 };
        case 'decrement':
            return { count: state.count - 1 };
        case 'reset':
            return { count: 0 };
        case 'set':
            return { count: action.payload };
        default:
            throw new Error(`Unknown action: ${action.type}`);
    }
}

function Counter() {
    const [state, dispatch] = useReducer(counterReducer, { count: 0 });
    
    return (
        <div>
            <p>Count: {state.count}</p>
            <button onClick={() => dispatch({ type: 'increment' })}>+</button>
            <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
            <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
            <button onClick={() => dispatch({ type: 'set', payload: 10 })}>Set 10</button>
        </div>
    );
}
```

**useRef：**

```jsx
import { useRef, useEffect } from 'react';

function TextInput() {
    // 1. DOM 引用
    const inputRef = useRef(null);
    
    useEffect(() => {
        inputRef.current.focus();
    }, []);
    
    // 2. 保存可变值（不触发重渲染）
    const countRef = useRef(0);
    const timerRef = useRef(null);
    
    function startTimer() {
        timerRef.current = setInterval(() => {
            countRef.current++;
            console.log(countRef.current);
        }, 1000);
    }
    
    function stopTimer() {
        clearInterval(timerRef.current);
    }
    
    // 3. 保存上一次的值
    const prevCountRef = useRef(0);
    const [count, setCount] = useState(0);
    
    useEffect(() => {
        prevCountRef.current = count;
    });
    
    const prevCount = prevCountRef.current;
    
    return (
        <div>
            <input ref={inputRef} />
            <p>Current: {count}, Previous: {prevCount}</p>
        </div>
    );
}
```

**useMemo 和 useCallback：**

```jsx
import { useMemo, useCallback, useState } from 'react';

function ExpensiveComponent({ items, onItemClick }) {
    // useMemo：缓存计算结果
    const sortedItems = useMemo(() => {
        console.log('排序中...');
        return [...items].sort((a, b) => a.name.localeCompare(b.name));
    }, [items]);  // 只有 items 变化时才重新计算
    
    // useCallback：缓存函数引用
    const handleClick = useCallback((id) => {
        onItemClick(id);
    }, [onItemClick]);  // 只有 onItemClick 变化时才创建新函数
    
    return (
        <ul>
            {sortedItems.map(item => (
                <li key={item.id} onClick={() => handleClick(item.id)}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
}

// React.memo：避免不必要的重渲染
const MemoizedComponent = React.memo(function MyComponent({ value }) {
    return <div>{value}</div>;
});

// 自定义比较函数
const MemoizedComponent = React.memo(
    function MyComponent({ user }) {
        return <div>{user.name}</div>;
    },
    (prevProps, nextProps) => {
        return prevProps.user.id === nextProps.user.id;
    }
);
```

**自定义 Hook：**

```jsx
// useLocalStorage
function useLocalStorage(key, initialValue) {
    const [storedValue, setStoredValue] = useState(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            return initialValue;
        }
    });
    
    const setValue = (value) => {
        try {
            const valueToStore = value instanceof Function
                ? value(storedValue)
                : value;
            setStoredValue(valueToStore);
            window.localStorage.setItem(key, JSON.stringify(valueToStore));
        } catch (error) {
            console.error(error);
        }
    };
    
    return [storedValue, setValue];
}

// useFetch
function useFetch(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        let cancelled = false;
        
        setLoading(true);
        setError(null);
        
        fetch(url)
            .then(res => res.json())
            .then(data => {
                if (!cancelled) {
                    setData(data);
                    setLoading(false);
                }
            })
            .catch(err => {
                if (!cancelled) {
                    setError(err);
                    setLoading(false);
                }
            });
        
        return () => {
            cancelled = true;
        };
    }, [url]);
    
    return { data, loading, error };
}

// useDebounce
function useDebounce(value, delay) {
    const [debouncedValue, setDebouncedValue] = useState(value);
    
    useEffect(() => {
        const timer = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);
        
        return () => clearTimeout(timer);
    }, [value, delay]);
    
    return debouncedValue;
}

// 使用
function SearchComponent() {
    const [query, setQuery] = useState('');
    const debouncedQuery = useDebounce(query, 300);
    const { data, loading } = useFetch(`/api/search?q=${debouncedQuery}`);
    
    return (
        <div>
            <input
                value={query}
                onChange={e => setQuery(e.target.value)}
                placeholder="搜索..."
            />
            {loading ? <p>加载中...</p> : <Results data={data} />}
        </div>
    );
}
```

#### 真实面试题

**题目1：`useEffect` 的依赖数组有什么作用？如何正确使用？**

**满分答案：**

**依赖数组的作用：**

控制 `useEffect` 的执行时机：

1. **不传依赖数组**：每次渲染后都执行
2. **空数组 `[]`**：只在挂载时执行一次
3. **有依赖 `[dep1, dep2]`**：依赖变化时执行

**常见错误：**

```jsx
// 错误1：遗漏依赖
function Component({ userId }) {
    const [user, setUser] = useState(null);
    
    useEffect(() => {
        fetchUser(userId).then(setUser);
    }, []);  // 错误：遗漏了 userId
    // 当 userId 变化时，不会重新请求
    
    // 正确
    useEffect(() => {
        fetchUser(userId).then(setUser);
    }, [userId]);
}

// 错误2：对象/函数作为依赖（每次渲染都是新引用）
function Component() {
    const options = { timeout: 1000 };  // 每次渲染都是新对象
    
    useEffect(() => {
        doSomething(options);
    }, [options]);  // 每次渲染都会执行
    
    // 正确：将对象移到 effect 内部
    useEffect(() => {
        const options = { timeout: 1000 };
        doSomething(options);
    }, []);
    
    // 或使用 useMemo
    const options = useMemo(() => ({ timeout: 1000 }), []);
}

// 错误3：在 effect 中使用 setState 但不处理清理
function Component() {
    const [data, setData] = useState(null);
    
    useEffect(() => {
        fetch('/api/data')
            .then(r => r.json())
            .then(setData);  // 组件卸载后可能仍然执行
    }, []);
    
    // 正确：添加清理
    useEffect(() => {
        let cancelled = false;
        
        fetch('/api/data')
            .then(r => r.json())
            .then(data => {
                if (!cancelled) setData(data);
            });
        
        return () => { cancelled = true; };
    }, []);
}
```

**最佳实践：**

1. 使用 ESLint 的 `exhaustive-deps` 规则检查依赖
2. 将函数移到 effect 内部或使用 `useCallback`
3. 将对象移到 effect 内部或使用 `useMemo`
4. 始终添加清理函数处理异步操作

---

**题目2：`useMemo` 和 `useCallback` 有什么区别？什么时候使用？**

**满分答案：**

**区别：**

| Hook | 缓存内容 | 返回值 |
|------|----------|--------|
| `useMemo` | 计算结果（值） | 缓存的值 |
| `useCallback` | 函数引用 | 缓存的函数 |

```jsx
// useMemo：缓存计算结果
const sortedList = useMemo(() => {
    return [...list].sort();
}, [list]);

// useCallback：缓存函数
const handleClick = useCallback(() => {
    doSomething(id);
}, [id]);

// useCallback 等价于
const handleClick = useMemo(() => {
    return () => doSomething(id);
}, [id]);
```

**何时使用：**

**使用 `useMemo`：**
- 计算开销大的操作（排序、过滤大数组）
- 需要稳定引用的对象（作为其他 Hook 的依赖）

**使用 `useCallback`：**
- 传递给子组件的回调函数（配合 `React.memo`）
- 作为其他 Hook 的依赖

**不要过度使用：**

```jsx
// 不必要的 useMemo（简单计算）
const doubled = useMemo(() => count * 2, [count]);  // 不必要
const doubled = count * 2;  // 更好

// 不必要的 useCallback（组件不使用 memo）
const handleClick = useCallback(() => {
    setCount(c => c + 1);
}, []);  // 如果子组件没有 memo，没有意义
```

---

## 5.2 React 进阶

### 5.2.1 虚拟 DOM 与 Diff 算法

#### 知识点详解

**虚拟 DOM（Virtual DOM）：**

虚拟 DOM 是真实 DOM 的 JavaScript 对象表示。

```javascript
// 真实 DOM
<div class="container">
    <h1>Hello</h1>
    <p>World</p>
</div>

// 虚拟 DOM
{
    type: 'div',
    props: { className: 'container' },
    children: [
        {
            type: 'h1',
            props: {},
            children: ['Hello']
        },
        {
            type: 'p',
            props: {},
            children: ['World']
        }
    ]
}
```

**为什么需要虚拟 DOM：**

1. 直接操作 DOM 性能差（DOM 操作触发重排重绘）
2. 虚拟 DOM 在内存中操作，批量更新
3. 跨平台（React Native、SSR）

**Diff 算法（协调算法）：**

React 的 Diff 算法基于三个假设：

1. **不同类型的元素产生不同的树**
2. **通过 key 标识稳定的元素**
3. **同层比较，不跨层**

```jsx
// 1. 不同类型：直接替换
// 旧
<div>
    <Counter />
</div>

// 新
<span>
    <Counter />
</span>
// 结果：销毁旧树，创建新树

// 2. 相同类型：更新属性
// 旧
<div className="before" title="stuff" />
// 新
<div className="after" title="stuff" />
// 结果：只更新 className

// 3. key 的作用
// 旧
<ul>
    <li key="1">First</li>
    <li key="2">Second</li>
</ul>

// 新
<ul>
    <li key="2">Second</li>
    <li key="1">First</li>
</ul>
// 结果：移动元素，不重新创建
```

**key 的重要性：**

```jsx
// 错误：使用 index 作为 key
{items.map((item, index) => (
    <Item key={index} data={item} />
))}
// 问题：删除中间元素时，后续元素的 key 变化，导致不必要的重渲染

// 正确：使用稳定的唯一 ID
{items.map(item => (
    <Item key={item.id} data={item} />
))}
```

### 5.2.2 React Router 和原生路由区别

#### 知识点详解

**原生路由：**

```javascript
// 浏览器原生 History API
history.pushState({}, '', '/home');
window.addEventListener('popstate', () => {
    // 监听浏览器前进/后退
});
```

**React Router：**

```jsx
import { BrowserRouter, Routes, Route, Link, useNavigate } from 'react-router-dom';

function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
            </Routes>
        </BrowserRouter>
    );
}

function About() {
    const navigate = useNavigate();
    return <button onClick={() => navigate('/')}>返回</button>;
}
```

**核心区别：**

| 特性 | 原生路由 | React Router |
|------|---------|--------------|
| 监听方式 | popstate 事件 | 内部封装 |
| URL 变化 | pushState | Link 组件 |
| 路由匹配 | 手动实现 | 自动匹配 |
| 嵌套路由 | 手动处理 | 组件化支持 |
| 传参 | 手动解析 | params/query |

**React Router 核心原理：**

```javascript
// 简化版 Router 实现
function createBrowserHistory() {
    const listeners = [];
    
    return {
        push(path) {
            window.history.pushState({}, '', path);
            listeners.forEach(fn => fn());
        },
        listen(fn) {
            listeners.push(fn);
        }
    };
}

// 路由匹配
function matchRoute(routes, pathname) {
    return routes.find(route => route.path === pathname);
}
```

#### 真实面试题

**题目：React Router 和原生浏览器路由有什么区别？**

**满分答案：**

1. **封装程度**：React Router 封装了原生 History API，更易用
2. **声明式**：使用 JSX 声明路由配置
3. **嵌套路由**：原生需要手动处理，React Router 原生支持
4. **动态路由**：React Router 支持路由参数和查询参数

**React Router 优势：**
- 声明式路由定义
- 嵌套路由支持
- 路由守卫/拦截器
- 与 React 组件深度集成

---

### 5.2.3 React Router v6

```jsx
import {
    BrowserRouter,
    Routes,
    Route,
    Navigate,
    Link,
    NavLink,
    Outlet,
    useNavigate,
    useParams,
    useLocation,
    useSearchParams
} from 'react-router-dom';

// 路由配置
function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
                
                {/* 嵌套路由 */}
                <Route path="/admin" element={<AdminLayout />}>
                    <Route index element={<AdminDashboard />} />
                    <Route path="users" element={<AdminUsers />} />
                    <Route path="settings" element={<AdminSettings />} />
                </Route>
                
                {/* 动态路由 */}
                <Route path="/user/:id" element={<UserProfile />} />
                
                {/* 重定向 */}
                <Route path="/old-path" element={<Navigate to="/new-path" replace />} />
                
                {/* 404 */}
                <Route path="*" element={<NotFound />} />
            </Routes>
        </BrowserRouter>
    );
}

// 嵌套路由布局
function AdminLayout() {
    return (
        <div>
            <nav>
                <NavLink to="/admin">Dashboard</NavLink>
                <NavLink to="/admin/users">Users</NavLink>
            </nav>
            <Outlet />  {/* 子路由渲染位置 */}
        </div>
    );
}

// 使用路由 Hooks
function UserProfile() {
    const { id } = useParams();
    const navigate = useNavigate();
    const location = useLocation();
    const [searchParams, setSearchParams] = useSearchParams();
    
    const page = searchParams.get('page') || '1';
    
    function goBack() {
        navigate(-1);
    }
    
    function goToUser(userId) {
        navigate(`/user/${userId}`, {
            state: { from: location.pathname }
        });
    }
    
    return <div>User {id}</div>;
}

// 路由守卫（受保护路由）
function ProtectedRoute({ children }) {
    const isAuthenticated = useAuth();
    const location = useLocation();
    
    if (!isAuthenticated) {
        return <Navigate to="/login" state={{ from: location }} replace />;
    }
    
    return children;
}

// 使用
<Route
    path="/admin"
    element={
        <ProtectedRoute>
            <AdminLayout />
        </ProtectedRoute>
    }
/>
```

### 5.2.4 Redux Toolkit

```javascript
// store/counterSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 异步 action
export const fetchUser = createAsyncThunk(
    'user/fetchUser',
    async (userId, { rejectWithValue }) => {
        try {
            const response = await fetch(`/api/user/${userId}`);
            return await response.json();
        } catch (error) {
            return rejectWithValue(error.message);
        }
    }
);

// Slice
const counterSlice = createSlice({
    name: 'counter',
    initialState: {
        value: 0,
        status: 'idle',
        error: null
    },
    reducers: {
        increment(state) {
            state.value++;  // Immer 允许直接修改
        },
        decrement(state) {
            state.value--;
        },
        incrementByAmount(state, action) {
            state.value += action.payload;
        }
    },
    extraReducers: (builder) => {
        builder
            .addCase(fetchUser.pending, (state) => {
                state.status = 'loading';
            })
            .addCase(fetchUser.fulfilled, (state, action) => {
                state.status = 'succeeded';
                state.user = action.payload;
            })
            .addCase(fetchUser.rejected, (state, action) => {
                state.status = 'failed';
                state.error = action.payload;
            });
    }
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
    reducer: {
        counter: counterReducer
    }
});

// 在组件中使用
import { useSelector, useDispatch } from 'react-redux';
import { increment, fetchUser } from './counterSlice';

function Counter() {
    const count = useSelector(state => state.counter.value);
    const dispatch = useDispatch();
    
    return (
        <div>
            <p>{count}</p>
            <button onClick={() => dispatch(increment())}>+</button>
            <button onClick={() => dispatch(fetchUser(1))}>Fetch User</button>
        </div>
    );
}
```

### 5.2.5 性能优化

```jsx
// 1. React.memo
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
    return <div>{data.name}</div>;
});

// 2. useMemo 缓存计算
function FilteredList({ items, filter }) {
    const filteredItems = useMemo(() => {
        return items.filter(item => item.name.includes(filter));
    }, [items, filter]);
    
    return <ul>{filteredItems.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
}

// 3. useCallback 稳定函数引用
function Parent() {
    const [count, setCount] = useState(0);
    
    const handleClick = useCallback(() => {
        console.log('clicked');
    }, []);  // 不依赖 count，所以不会重新创建
    
    return (
        <div>
            <p>{count}</p>
            <button onClick={() => setCount(c => c + 1)}>+</button>
            <Child onClick={handleClick} />
        </div>
    );
}

// 4. 代码分割
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <LazyComponent />
        </Suspense>
    );
}

// 5. 虚拟列表（react-window）
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
    const Row = ({ index, style }) => (
        <div style={style}>{items[index].name}</div>
    );
    
    return (
        <FixedSizeList
            height={600}
            itemCount={items.length}
            itemSize={50}
            width="100%"
        >
            {Row}
        </FixedSizeList>
    );
}

// 6. 避免不必要的状态
// 不好：将可以从 props 计算的值放入 state
function BadComponent({ items }) {
    const [filteredItems, setFilteredItems] = useState(items);
    
    useEffect(() => {
        setFilteredItems(items.filter(item => item.active));
    }, [items]);
    
    return <ul>{filteredItems.map(...)}</ul>;
}

// 好：直接计算
function GoodComponent({ items }) {
    const filteredItems = items.filter(item => item.active);
    return <ul>{filteredItems.map(...)}</ul>;
}
```

### 5.2.6 React 18 新特性

```jsx
// 1. Concurrent Mode（并发模式）
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);

// 2. Automatic Batching（自动批处理）
// React 18 之前：只在事件处理中批处理
// React 18：所有更新都自动批处理

function handleClick() {
    setCount(c => c + 1);  // 不会立即重渲染
    setFlag(f => !f);      // 不会立即重渲染
    // 只触发一次重渲染
}

// 3. Transitions（过渡）
import { useTransition, startTransition } from 'react';

function SearchComponent() {
    const [query, setQuery] = useState('');
    const [results, setResults] = useState([]);
    const [isPending, startTransition] = useTransition();
    
    function handleChange(e) {
        setQuery(e.target.value);  // 紧急更新
        
        startTransition(() => {
            setResults(search(e.target.value));  // 非紧急更新
        });
    }
    
    return (
        <div>
            <input value={query} onChange={handleChange} />
            {isPending ? <p>Loading...</p> : <Results data={results} />}
        </div>
    );
}

// 4. useDeferredValue
import { useDeferredValue } from 'react';

function SearchResults({ query }) {
    const deferredQuery = useDeferredValue(query);
    
    // 使用 deferredQuery 进行搜索，不阻塞输入
    const results = useMemo(() => search(deferredQuery), [deferredQuery]);
    
    return <ul>{results.map(r => <li key={r.id}>{r.name}</li>)}</ul>;
}

// 5. useId（生成唯一 ID）
import { useId } from 'react';

function FormField({ label }) {
    const id = useId();
    
    return (
        <div>
            <label htmlFor={id}>{label}</label>
            <input id={id} />
        </div>
    );
}
```

#### 真实面试题

**题目1：React 中如何避免不必要的重渲染？**

**满分答案：**

**1. React.memo（组件级别）**

```jsx
const Child = React.memo(({ value, onClick }) => {
    console.log('Child rendered');
    return <button onClick={onClick}>{value}</button>;
});

function Parent() {
    const [count, setCount] = useState(0);
    const [text, setText] = useState('');
    
    // 使用 useCallback 稳定函数引用
    const handleClick = useCallback(() => {
        console.log('clicked');
    }, []);
    
    return (
        <div>
            <input value={text} onChange={e => setText(e.target.value)} />
            {/* text 变化时，Child 不会重渲染 */}
            <Child value={count} onClick={handleClick} />
        </div>
    );
}
```

**2. useMemo（值级别）**

```jsx
function ExpensiveList({ items, filter }) {
    const filteredItems = useMemo(() => {
        return items.filter(item => item.name.includes(filter));
    }, [items, filter]);
    
    return <ul>{filteredItems.map(...)}</ul>;
}
```

**3. 状态下移（State Colocation）**

```jsx
// 不好：状态在父组件，导致兄弟组件重渲染
function Parent() {
    const [inputValue, setInputValue] = useState('');
    
    return (
        <div>
            <Input value={inputValue} onChange={setInputValue} />
            <ExpensiveComponent />  {/* 每次输入都重渲染 */}
        </div>
    );
}

// 好：状态下移到需要的组件
function Parent() {
    return (
        <div>
            <InputWithState />  {/* 状态在这里 */}
            <ExpensiveComponent />  {/* 不受影响 */}
        </div>
    );
}
```

**4. 内容提升（Content Lifting）**

```jsx
// 不好
function Parent() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <button onClick={() => setCount(c => c + 1)}>{count}</button>
            <ExpensiveChild />  {/* 每次 count 变化都重渲染 */}
        </div>
    );
}

// 好：将 ExpensiveChild 作为 children 传入
function Counter({ children }) {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <button onClick={() => setCount(c => c + 1)}>{count}</button>
            {children}  {/* children 是稳定的引用 */}
        </div>
    );
}

function Parent() {
    return (
        <Counter>
            <ExpensiveChild />  {/* 不会重渲染 */}
        </Counter>
    );
}
```

---

**题目2：解释 React Fiber 架构**

**满分答案：**

**什么是 Fiber：**

Fiber 是 React 16 引入的新协调引擎，是对 React 核心算法的重写。

**为什么需要 Fiber：**

React 15 的 Stack Reconciler 是同步的，一旦开始就无法中断，可能导致：
- 长时间占用主线程
- 动画卡顿
- 用户输入响应延迟

**Fiber 的核心思想：**

1. **可中断的渲染**：将渲染工作分成小单元（Fiber 节点），可以暂停和恢复
2. **优先级调度**：不同更新有不同优先级（用户输入 > 动画 > 数据加载）
3. **双缓冲**：current 树和 work-in-progress 树

**Fiber 节点结构：**

```javascript
{
    type: 'div',           // 元素类型
    key: null,             // key
    stateNode: domNode,    // 对应的 DOM 节点
    
    // 树结构
    return: parentFiber,   // 父节点
    child: childFiber,     // 第一个子节点
    sibling: siblingFiber, // 下一个兄弟节点
    
    // 工作
    pendingProps: {},       // 新 props
    memoizedProps: {},      // 旧 props
    memoizedState: {},      // 旧 state
    
    // 副作用
    effectTag: 'UPDATE',   // 需要执行的操作
    nextEffect: nextFiber  // 下一个有副作用的节点
}
```

**渲染阶段：**

1. **Render 阶段（可中断）**：
   - 遍历 Fiber 树，找出需要更新的节点
   - 构建 work-in-progress 树
   - 可以被高优先级任务中断

2. **Commit 阶段（不可中断）**：
   - 将变更应用到真实 DOM
   - 执行生命周期和 Hooks
   - 必须同步完成

**优先级调度：**

```javascript
// 优先级从高到低
ImmediatePriority = 1;    // 同步，立即执行
UserBlockingPriority = 2; // 用户交互（点击、输入）
NormalPriority = 3;       // 普通更新
LowPriority = 4;          // 低优先级
IdlePriority = 5;         // 空闲时执行
```

---

### 5.x.x React 路由变化监听 [必会]

#### 知识点详解

在 React 中监听路由变化有多种方式，取决于使用的路由库：

**1. React Router v6（推荐）**

```jsx
// 方式1：使用 useLocation + useEffect 监听
import { useNavigate, useLocation } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();
  const location = useLocation();

  // 监听路由变化
  useEffect(() => {
    console.log('路由变化:', location.pathname, location.search);
  }, [location]);

  return <div>...</div>;
}

// 方式2：使用 useLocation 直接订阅
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();

  useEffect(() => {
    // 每次路由切换都会触发
    window.scrollTo(0, 0);
    console.log('路由切换到:', location.pathname);
  }, [location]);

  return <Routes>...</Routes>;
}
```

**2. React Router v4/v5**

```jsx
// 方式1：使用 withRouter HOC
import { withRouter } from 'react-router-dom';

class MyComponent extends React.Component {
  componentDidUpdate(prevProps) {
    if (this.props.location !== prevProps.location) {
      console.log('路由变化:', this.props.location.pathname);
    }
  }
  render() { return <div>...</div>; }
}

export default withRouter(MyComponent);

// 方式2：使用 useLocation Hook（v5.1+）
import { useLocation } from 'react-router-dom';

function MyComponent() {
  const location = useLocation();
  useEffect(() => {
    console.log('路由变化到:', location.pathname);
  }, [location]);
}
```

**3. 全局路由监听（不限于组件）**

```jsx
// 使用 history 对象手动监听
import { createBrowserHistory } from 'history';

const history = createBrowserHistory();

history.listen(({ location, action }) => {
  console.log('路由变化:', location.pathname, action);
  // action: 'PUSH' | 'POP' | 'REPLACE'
});
```

**常见面试题：**

> **Q：为什么 useEffect 依赖数组要放 location？**
> 因为 location 是路由状态对象，路由切换时 location 引用会变化（即使 pathname 相同），所以需要监听 location 的变化来触发回调。

#### 面试加分项
- 了解 React Router 的实现原理（基于 history 库）
- 理解 v6 版本相对于 v4/v5 的改进（Routes 组件消歧义、useNavigate 替代 Redirect）

---

## 5.3 React 状态管理：Zustand 与 Redux

#### 知识点详解

**Redux 工作原理：**

```
Action → Dispatch → Reducer → New State → Subscribe → UI Update
```

```javascript
// Redux 核心概念
// 1. Action（动作）
const increment = { type: 'INCREMENT', payload: 1 };

// 2. Reducer（处理器）
const counterReducer = (state = 0, action) => {
    switch (action.type) {
        case 'INCREMENT':
            return state + action.payload;
        case 'DECREMENT':
            return state - action.payload;
        default:
            return state;
    }
};

// 3. Store（仓库）
import { createStore } from 'redux';
const store = createStore(counterReducer);

// 4. Subscribe（订阅）
store.subscribe(() => console.log(store.getState()));

// 5. Dispatch（分发）
store.dispatch(increment);
```

**Zustand（轻量级状态管理）：**

```javascript
// 创建 store
import { create } from 'zustand';

const useStore = create((set, get) => ({
    // 状态
    count: 0,
    user: null,
    messages: [],
    
    // 方法（类似 reducer）
    increment: () => set(state => ({ count: state.count + 1 })),
    decrement: () => set(state => ({ count: state.count - 1 })),
    setUser: (user) => set({ user }),
    addMessage: (msg) => set(state => ({ 
        messages: [...state.messages, msg] 
    })),
    
    // 异步
    fetchUser: async (id) => {
        const res = await fetch(`/api/users/${id}`);
        const user = await res.json();
        set({ user });
    },
    
    // 计算属性
    getCountDouble: () => get().count * 2
}));

// 组件中使用
function Counter() {
    const { count, increment } = useStore();
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={increment}>+</button>
        </div>
    );
}
```

**AI 工作流中的状态设计：**

```javascript
// AI 对话状态管理
import { create } from 'zustand';

const useAIStore = create((set, get) => ({
    // 状态
    messages: [],           // 对话消息
    isGenerating: false,   // 是否正在生成
    currentSession: null,  // 当前会话
    abortController: null, // 用于取消请求
    
    // 添加用户消息
    addUserMessage: (content) => set(state => ({
        messages: [...state.messages, {
            id: Date.now(),
            role: 'user',
            content,
            timestamp: new Date()
        }]
    })),
    
    // 添加 AI 消息（流式）
    addAIMessage: (content) => set(state => {
        const lastMsg = state.messages[state.messages.length - 1];
        if (lastMsg?.role === 'assistant') {
            // 追加内容
            const messages = [...state.messages];
            messages[messages.length - 1] = {
                ...lastMsg,
                content: lastMsg.content + content
            };
            return { messages };
        }
        // 新建消息
        return { messages: [...state.messages, {
            id: Date.now(),
            role: 'assistant',
            content,
            timestamp: new Date()
        }]};
    }),
    
    // 开始生成
    startGenerating: (controller) => set({ 
        isGenerating: true,
        abortController: controller 
    }),
    
    // 结束生成
    finishGenerating: () => set({ 
        isGenerating: false,
        abortController: null 
    }),
    
    // 取消生成
    cancelGenerating: () => {
        const { abortController } = get();
        if (abortController) {
            abortController.abort();
            get().finishGenerating();
        }
    },
    
    // 清空对话
    clearMessages: () => set({ messages: [] })
}));

// 使用
function Chat() {
    const { messages, addUserMessage, isGenerating, cancelGenerating } = useAIStore();
    
    const sendMessage = async (content) => {
        addUserMessage(content);
        
        const controller = new AbortController();
        startGenerating(controller);
        
        try {
            const response = await fetch('/api/chat', {
                method: 'POST',
                body: JSON.stringify({ messages: [...messages, { role: 'user', content }] }),
                signal: controller.signal
            });
            
            const reader = response.body.getReader();
            const decoder = new TextDecoder();
            
            while (true) {
                const { done, value } = await reader.read();
                if (done) break;
                
                const chunk = decoder.decode(value);
                addAIMessage(chunk);
            }
        } finally {
            finishGenerating();
        }
    };
    
    return (
        <div>
            {messages.map(m => (
                <div key={m.id}>{m.role}: {m.content}</div>
            ))}
            {isGenerating && <button onClick={cancelGenerating}>取消</button>}
        </div>
    );
}
```

#### 真实面试题

**题目：状态管理库（如 Zustand 或 Redux）在复杂 AI 工作流中如何设计？**

**满分答案：**

**AI 场景特点：**
1. 流式响应（需要增量更新状态）
2. 取消能力（用户中断生成）
3. 多会话管理
4. 消息状态（loading、error、success）

**Zustand 优势：**
1. 轻量、简洁
2. 无 Provider 包裹
3. 支持异步
4. 适合 AI 这种需要快速迭代的场景

**Redux 优势：**
1. 成熟稳定
2. DevTools 强大
3. 中间件生态

**推荐：**
- 中小项目 → Zustand
- 大型复杂项目 → Redux Toolkit

---
