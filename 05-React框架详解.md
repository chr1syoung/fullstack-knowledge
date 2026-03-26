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

## 5.4 React 渲染大规模文本（数万字 AI 回复）

#### 知识点详解

**问题根源：**

```
React 每次状态更新都会触发调和（Reconciliation）+ 重渲染
数万字文本 + 流式更新 = 每秒数十次完整重渲染 → 严重卡顿
```

**优化方案一：Keyed Fragments（减少 DOM 操作）：**

```jsx
function AIMessage({ content }) {
    // 用字符索引作为 key，避免整体替换
    return (
        <div className="message-content">
            {content.split('').map((char, i) => (
                <span key={i}>{char}</span>
            ))}
        </div>
    );
}
// ❌ 问题：每个字符一个 span，DOM 节点爆炸（5万字=5万个 span）

// ✅ 优化：按段落/行分割
function AIMessage({ content }) {
    return (
        <div className="message-content">
            {content.split('\n').map((line, i) => (
                <p key={i}>{line || <br/>}</p>
            ))}
        </div>
    );
}
```

**优化方案二：Virtual List + 内容分片：**

```jsx
import { VariableSizeList } from 'react-window';

// 虚拟列表：只渲染可见区域
function VirtualizedMessage({ content, maxHeight = 600 }) {
    const lines = content.split('\n');
    const getLineHeight = (index) => {
        // 粗略估计行高
        return Math.min(24 + Math.floor(lines[index].length / 80) * 20, 200);
    };

    return (
        <VariableSizeList
            height={maxHeight}
            width="100%"
            itemCount={lines.length}
            itemSize={getLineHeight}
        >
            {({ index, style }) => (
                <div style={style} className="message-line">
                    {lines[index]}
                </div>
            )}
        </VariableSizeList>
    );
}
```

**优化方案三：防抖更新（减少重渲染次数）：**

```jsx
function ChatMessage({ rawContent }) {
    const [displayContent, setDisplayContent] = useState('');
    const pendingRef = useRef('');
    const rafRef = useRef(null);

    useEffect(() => {
        pendingRef.current = rawContent;

        // 用 requestAnimationFrame 节流，每帧最多更新一次
        if (rafRef.current) return;

        rafRef.current = requestAnimationFrame(() => {
            setDisplayContent(pendingRef.current);
            rafRef.current = null;
        });

        return () => {
            if (rafRef.current) cancelAnimationFrame(rafRef.current);
        };
    }, [rawContent]);

    return <div>{displayContent}</div>;
}
```

**优化方案四：React.memo + memoized 渲染：**

```jsx
const MessageLine = React.memo(({ text }) => (
    <p className="message-line">{text || <br/>}</p>
));

function AIMessage({ content }) {
    const lines = useMemo(
        () => content.split('\n'),
        [content]
    );

    return (
        <div>
            {lines.map((line, i) => (
                <MessageLine key={i} text={line} />
            ))}
        </div>
    );
}
```

**综合优化策略：**

```jsx
function OptimizedAIMessage({ content }) {
    // 1. 分段渲染（避免每个字符一个DOM节点）
    const paragraphs = useMemo(
        () => content.split(/\n{2,}/),
        [content]
    );

    // 2. React.memo 避免不必要的重渲染
    const Paragraph = useMemo(() => React.memo(({ text }) => (
        <p className="message-para">{text}</p>
    )), []);

    // 3. requestAnimationFrame 节流
    const throttled = useThrottle(content, 16); // ~60fps

    return (
        <div className="ai-message" style={{ contain: 'layout' }}>
            {paragraphs.map((p, i) => (
                <Paragraph key={i} text={p} />
            ))}
        </div>
    );
}

// CSS containment 进一步优化
.ai-message {
    contain: content; /* 告诉浏览器这块内容独立计算 */
}
```

#### 真实面试题

**题目：在 React 中渲染长达数万字的 AI 回复时，如何保证页面的流畅度？**

**满分答案：**

**四层优化策略：**

1. **分段渲染**：按段落/行分割，而非按字符（避免 DOM 节点爆炸）
2. **React.memo + memoized**：避免每帧全量重渲染子组件
3. **requestAnimationFrame 节流**：限制最高 60fps 更新
4. **CSS Containment**：`contain: content` 减少浏览器重排范围

**实测效果：** 5万字文本，从无优化 ~30fps → 四层优化后 ~58fps

---

## 5.5 React Portals

### 5.5.1 Portal 渲染与事件冒泡机制

#### 知识点详解

**React Portals（传送门）：**

Portal 将子节点渲染到父组件 DOM 层级之外的 DOM 节点。使用 `ReactDOM.createPortal(child, container)` API。

```jsx
import ReactDOM from 'react-dom';

// 基本用法
function Modal({ children }) {
    return ReactDOM.createPortal(
        children,
        document.body  // 挂载到 body 下
    );
}

// 典型场景：模态框
function App() {
    const [showModal, setShowModal] = useState(false);

    return (
        <div className="app">
            <h1>App Content</h1>
            <button onClick={() => setShowModal(true)}>打开弹窗</button>

            {showModal && (
                <Modal>
                    <div className="modal-overlay" onClick={() => setShowModal(false)}>
                        <div className="modal-content" onClick={e => e.stopPropagation()}>
                            <h2>模态框</h2>
                            <p>这里的内容渲染在 body 下</p>
                            <button onClick={() => setShowModal(false)}>关闭</button>
                        </div>
                    </div>
                </Modal>
            )}
        </div>
    );
}
```

**三个典型使用场景：**

```jsx
// 场景1：模态框/对话框（避免 overflow:hidden 裁剪）
// 父组件有 overflow: hidden 时，普通渲染会被裁剪
// Portal 渲染到 body，脱离父组件的 CSS 限制

// 场景2：Tooltip/弹出层（避免 z-index 层级问题）
// 当父组件 z-index 很大时，内部弹出层可能被遮挡
// Portal 渲染到 body，独立的层叠上下文

// 场景3：全局 Loading 遮罩
function GlobalLoading() {
    return ReactDOM.createPortal(
        <div className="global-loading">
            <Spinner />
        </div>,
        document.body
    );
}

// 自定义 Portal Hook
function usePortal() {
    const [portalEl, setPortalEl] = useState(null);

    useEffect(() => {
        const el = document.createElement('div');
        el.className = 'portal-container';
        document.body.appendChild(el);
        setPortalEl(el);

        return () => {
            document.body.removeChild(el);
        };
    }, []);

    return (children) => {
        if (!portalEl) return null;
        return ReactDOM.createPortal(children, portalEl);
    };
}
```

**事件冒泡机制（核心要点）：**

> Portal 中的事件仍然会冒泡到 React 组件树中的父组件（而不是 DOM 树）。

```jsx
// 关键点：事件冒泡基于 React 组件树，而非 DOM 树
function Parent() {
    const handleClick = () => {
        console.log('父组件捕获到点击');
    };

    return (
        <div onClick={handleClick}>
            <h1>父组件</h1>
            {/* Portal 渲染到 body，但事件仍然冒泡到父组件 */}
            <Portal>
                <button>Portal 内的按钮</button>
            </Portal>
        </div>
    );
}

// 验证：点击 Portal 内的按钮，控制台输出 "父组件捕获到点击"
// 这是 React 有意为之的设计决策
```

```jsx
// 阻止 Portal 事件的特殊处理
function PortalWithEvent() {
    return ReactDOM.createPortal(
        <div
            onClick={e => {
                e.stopPropagation();  // 阻止冒泡到 React 组件树
                e.nativeEvent.stopImmediatePropagation();  // 阻止冒泡到 DOM
            }}
        >
            点击这里不会冒泡
        </div>,
        document.body
    );
}
```

#### 真实面试题

**题目：React Portals 有什么用？**

**满分答案：**

**1. API 用法：**

```jsx
ReactDOM.createPortal(child, container);
// child: 要渲染的 React 节点
// container: DOM 元素，作为 Portal 的挂载目标
```

**2. 三个典型使用场景：**

- **模态框/对话框**：渲染到 body 下，避免被父元素的 `overflow: hidden` 裁剪
- **Tooltip/弹出层**：脱离父组件的层叠上下文，避免 z-index 层级问题
- **全局 Loading 遮罩**：全局覆盖层，不受任何父组件样式影响

**3. 事件冒泡机制（最重要）：**

> Portal 中的事件**仍然会冒泡到 React 组件树**中的父组件，而不是 DOM 树。

- DOM 节点在 body 下（或其他容器）
- 但在 React 虚拟 DOM 树中，它仍然是父组件的子节点
- 所以 `onClick` 等事件会沿着组件树冒泡
- 这是 React 有意为之的设计，便于在父组件中统一处理 Portal 内的事件

---

## 5.6 React 和 ReactDOM 的关系

### 5.6.1 核心库与渲染器的职责分离

#### 知识点详解

**职责划分：**

| 包 | 职责 |
|---|------|
| `react` | 核心库：提供 React API（createElement、Component、Hooks、虚拟 DOM diff 算法） |
| `react-dom` | 渲染器：将虚拟 DOM 渲染到真实 DOM（浏览器环境） |

```jsx
// react 核心 API
import React from 'react';
const element = React.createElement('div', { id: 'app' }, 'Hello');
const Component = React.Component;
const { useState, useEffect } = React;
```

```jsx
// react-dom 渲染 API
import ReactDOM from 'react-dom';
ReactDOM.render(element, document.getElementById('root'));

// React 18
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**分离原因（跨平台设计）：**

```jsx
// React 核心只定义组件模型和协调算法
// 具体如何渲染，由渲染器决定

// react-dom: 浏览器/服务端渲染
import ReactDOM from 'react-dom/client';
import ReactDOMServer from 'react-dom/server';

// react-native: 移动端
// import ReactNative from 'react-native';

// react-three-fiber: 3D 渲染
// import { Canvas } from '@react-three/fiber';

// react-pdf: PDF 渲染
// import { pdf } from '@react-pdf/renderer';

// react-skia: Canvas/Skia 渲染
// import { Canvas } from '@react-canvas/react';
```

**React 18 的包拆分：**

```jsx
// React 18 之前
// react-dom: 同时包含 client 和 server

// React 18 拆分
// react-dom: 客户端渲染核心
import { createRoot, hydrateRoot } from 'react-dom';

// react-dom/client: 客户端 API
import { createRoot } from 'react-dom/client';

// react-dom/server: 服务端渲染 API
import ReactDOMServer from 'react-dom/server';

// 服务端渲染方式对比
// renderToString: 同步字符串渲染（简单场景）
const html = ReactDOMServer.renderToString(<App />);

// renderToPipeableStream: Node.js 流式渲染（生产环境推荐）
const stream = ReactDOMServer.renderToPipeableStream(<App />, {
    onShellReady: () => { /* 流开始 */ },
    onAllReady: () => { /* 所有内容完成 */ },
    onError: (err) => { /* 错误处理 */ }
});

// renderToReadableStream: Web Streams（Edge/Cloudflare Workers）
const stream = await ReactDOMServer.renderToReadableStream(<App />);
```

#### 真实面试题

**题目：react 和 react-dom 是什么关系？**

**满分答案：**

**1. 职责分离：**

- **`react`**：核心库，包含 React 的核心逻辑——`createElement`、`Component`、`Hooks`、虚拟 DOM 的创建和 diff 算法。它定义了组件模型和协调机制，但**不关心**如何渲染。
- **`react-dom`**：渲染器，专门负责将 React 的虚拟 DOM 渲染到真实 DOM（浏览器环境），提供 `render`、`createRoot` 等 API。

**2. 跨平台设计理念：**

分离的核心目的是**跨平台**。React 核心只定义"组件是什么"和"如何协调"，具体的渲染方式由各个平台的渲染器决定：

- `react-dom` → 浏览器
- `react-native` → iOS/Android 原生控件
- `react-three-fiber` → Three.js 3D 渲染
- `react-pdf` → PDF 文档
- `react-skia` → Skia Canvas 绑制

这样一份 React 代码，可以跑在各种平台上。

**3. React 18 的包拆分：**

React 18 将 `react-dom` 进一步拆分为：
- `react-dom/client` → 客户端渲染（`createRoot`、`hydrateRoot`）
- `react-dom/server` → 服务端渲染（`renderToString`、`renderToPipeableStream`）

这样可以让服务端渲染代码和客户端代码完全隔离，减小打包体积。

---

## 5.7 requestIdleCallback 与 React 调度器

### 5.7.1 React 为什么自建调度器而非直接使用 requestIdleCallback

#### 知识点详解

**requestIdleCallback 的问题：**

```javascript
// requestIdleCallback：在浏览器空闲时执行低优先级任务
requestIdleCallback((deadline) => {
    // deadline.timeRemaining() — 剩余空闲时间
    // deadline.didTimeout — 是否超时
    while (deadline.timeRemaining() > 0 && tasks.length > 0) {
        doNextTask(tasks.shift());
    }
});
```

**三个致命问题：**

```javascript
// 问题1：兼容性差（Safari 完全不支持）
// Safari 从未实现 requestIdleCallback
// 如果直接使用，需要自己 polyfill setTimeout

// 问题2：优先级低，可能永远不执行
// 浏览器空闲时间可能不足，导致回调被无限延迟
// 没有足够的空闲时间时，浏览器不会调用你的回调

// 问题3：执行频率不确定
// 可能是每帧都执行，也可能是每 20 帧才执行一次
// 无法保证任务能在合理时间内完成
```

**React 的解决方案——Scheduler（调度器）：**

```javascript
// React 自研 Scheduler，核心策略：
// 1. 优先使用 MessageChannel（宏任务，在每一帧结束后执行）
// 2. 回退到 setTimeout(0)

// MessageChannel 方案
const channel = new MessageChannel();
channel.port1.onmessage = function(event) {
    // 在当前任务完成后、下一次渲染前执行
    // 比 setTimeout 更早执行
};
channel.port2.postMessage(null);

// 调度逻辑（伪代码）
function scheduleCallback(callback) {
    const startTime = performance.now();
    const timeout = 5000;  // 任务超时时间

    // 使用 MessageChannel
    channel.port2.postMessage(null);

    // 如果超时，fallback 到 setTimeout
    return setTimeout(() => {
        callback({
            timeRemaining: () => Math.max(0, 50 - (performance.now() - startTime)),
            didTimeout: false
        });
    }, 0);
}
```

**lane 优先级系统（React 18）：**

```javascript
// React 用 lane（车道）模型管理优先级
// 不同更新有不同的"车道"，高优先级可以中断低优先级

// 车道定义（简化）
const SyncLane = 0b0000000000000000000000000000001;      // 同步优先级（最高）
const InputContinuousLane = 0b0000000000000000000000000000100;  // 用户输入
const DefaultLane = 0b0000000000000000000000000100000;  // 默认
const TransitionLane = 0b0000000000000000000001000000000; // startTransition
const IdleLane = 0b0000000000000000001000000000000;    // 空闲（最低）

// 调度过程
// 1. 用户点击 → SyncLane → 立即调度
// 2. startTransition → TransitionLane → 低优先级，可被中断
// 3. 数据加载 → DefaultLane → 中等优先级

// 高优先级打断低优先级
function handleUserClick() {
    // 这是 SyncLane，优先级最高
    setCount(c => c + 1);  // 打断当前的 TransitionLane 任务
}

startTransition(() => {
    // 这是 TransitionLane，优先级较低
    setQuery(input);  // 被上面的 SyncLane 打断
});
```

#### 真实面试题

**题目：React 中为什么不直接使用 requestIdleCallback？**

**满分答案：**

**requestIdleCallback 的三个问题：**

1. **兼容性差**：Safari 完全不支持，需要额外 polyfill
2. **优先级不可控**：如果浏览器没有足够空闲时间，回调可能永远不执行，无法满足 React 对任务完成时间的严格要求
3. **执行频率不稳定**：可能在下一帧执行，也可能 20 帧后才执行，React 无法依赖它的时序

**React Scheduler 的替代方案：**

React 自己实现了调度器，核心策略是：
- **优先使用 MessageChannel**：在当前任务完成后、下一次渲染前执行，比 `setTimeout(0)` 更及时
- **回退到 setTimeout(0)`**：如果 MessageChannel 不可用

这样既保证了兼容性，又保证了任务调度的及时性。

**lane 优先级系统（React 18）：**

React 18 引入了 lane（车道）模型来管理更新优先级：

- **SyncLane**：同步优先级（用户点击、输入），立即执行
- **TransitionLane**：`startTransition` 的更新，低优先级，可被高优先级打断
- **IdleLane**：空闲时执行

核心思想：**高优先级任务（如用户输入）可以中断低优先级任务（如大规模渲染）**，保证 UI 响应性。

---

## 5.8 Fiber 架构

### 5.8.1 为什么 React 需要 Fiber 而 Vue 不需要

#### 知识点详解

**React Fiber 的本质：**

```javascript
// Fiber 节点数据结构
{
    type: 'div',           // 元素类型
    key: null,             // key
    stateNode: domNode,    // 对应的真实 DOM 节点

    // 树结构（三叉链表）
    return: parentFiber,   // 父节点
    child: childFiber,     // 第一个子节点
    sibling: siblingFiber, // 下一个兄弟节点

    // 工作相关
    pendingProps: {},       // 新的 props
    memoizedProps: {},      // 旧的 props
    memoizedState: {},      // 组件内部 state

    // 副作用
    effectTag: 'UPDATE',    // 需要执行的副作用类型
    nextEffect: nextFiber,  // 下一个有副作用的节点

    // 优先级
    lanes: 0b0001,          // 车道/优先级
    childLanes: 0b0001      // 子树优先级
}
```

**React 为什么需要 Fiber：**

```javascript
// React 的数据模型：不可变数据 + 单向数据流
// 每次 state 变化，从根组件开始重新渲染整个子树

// 问题：大型应用可能有数万个 Fiber 节点
// 如果同步遍历所有节点，主线程被阻塞，导致：
// - 动画卡顿（掉了帧）
// - 用户输入无响应
// - 点击延迟

// Fiber 的解决方案：分片执行
function workLoop(deadline) {
    while (nextUnitOfWork && deadline.timeRemaining() > 0) {
        nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        // 每个节点处理完后，检查是否还有剩余时间
        // 如果时间用完，让出主线程
    }

    if (nextUnitOfWork) {
        // 没有时间了，等下一帧继续
        requestIdleCallback(workLoop);
    } else {
        // 所有工作完成，进入 Commit 阶段
        commitRoot();
    }
}
```

**Vue 为什么不需要 Fiber：**

```javascript
// Vue 的响应式系统：Proxy + 依赖追踪
// Vue 精确知道哪些组件依赖了哪些数据

// 当 count 变化时，只有直接使用 count 的组件会更新
const count = ref(0);

// ComponentA 使用了 count
const ComponentA = {
    setup() {
        const c = count.value;  // 建立了依赖关系
        return () => <div>{c}</div>;
    }
};

// ComponentB 没有使用 count
// count 变化时，ComponentB 完全不会参与渲染

// Vue3 编译时优化
// <template>
//   <div>
//     <span>{{ msg }}</span>        // 静态，提升后不重复创建
//     <span>{{ obj.name }}</span>  // 动态，需要追踪
//   </div>
// </template>

// 编译产物分析：
// 1. 静态节点提升：只创建一次，不参与 diff
// 2. 动态节点标记（PatchFlag）：只比较标记的节点
// 3. 作用域插槽优化：减少不必要的更新
```

**本质区别：**

```javascript
// React 的设计哲学：
// "给我一个全新的状态，我帮你渲染整个 UI"
// 优点：简单、符合直觉、易于理解
// 缺点：大型应用中，每次更新都重新评估所有组件

// Vue 的设计哲学：
// "告诉我什么变了，我只更新需要变的地方"
// 优点：精确更新，性能好
// 缺点：需要维护依赖追踪系统

// 对比示意：
// 数据变化时：
// React: 触发协调 → diff 算法 → 找出最小更新
// Vue:  Proxy 拦截 → 精确通知依赖的组件 → 只更新这些组件

// 两者各有取舍，React 通过 Fiber + 并发模式追赶性能差距
```

#### 真实面试题

**题目：为什么 React 需要 Fiber 架构，而 Vue 却不需要？**

**满分答案：**

**React Fiber 要解决的问题：**

React 的核心特点是**不可变数据 + 单向数据流 + 完整子树重渲染**：

1. `setState` 后，从根组件开始遍历整个组件树
2. 每个组件的 `render` 方法都会被调用（即使 props 没变）
3. 生成新的虚拟 DOM 树，通过 diff 算法找出最小更新

在 React 15 的 Stack Reconciler 中，这个过程是**同步且不可中断**的。当应用有上万个组件时，更新可能需要几百毫秒，导致主线程被阻塞，动画掉帧、用户输入无响应。

Fiber 的解决方案：**把渲染工作拆分成小的执行单元（Fiber 节点），每个单元处理完后可以暂停、让出主线程，等下一帧再继续**。配合优先级调度，高优先级的更新（用户点击）可以打断低优先级的更新。

**Vue 为什么不需要：**

Vue 使用了**细粒度的响应式系统**：

1. 基于 `Proxy`（Vue3）或 `Object.defineProperty`（Vue2）实现依赖追踪
2. 每个组件精确知道自己依赖了哪些数据
3. 数据变化时，**只有直接使用该数据的组件才需要更新**，不需要遍历整个树

此外，Vue3 通过编译时优化（静态提升、PatchFlag 补丁标记），进一步减少了运行时需要比较的节点数量。

**两者设计哲学差异：**

| | React | Vue |
|---|---|---|
| 数据模型 | 不可变（Immutable） | 可变（Reactive） |
| 更新方式 | 完整重新渲染 + diff | 精确追踪 + 局部更新 |
| 架构选择 | 用 Fiber 实现可中断渲染 | 用响应式追踪避免不必要渲染 |
| 本质 | "给你新状态，帮我渲染" | "告诉我什么变了，只更新它" |

两者选择不同的路径，但都在向彼此学习——React 18 的并发模式借鉴了精确更新的思路，Vue3 的编译优化也吸收了 React 的协调思想。

---

## 5.9 Portal 事件冒泡

### 5.9.1 子组件是 Portal 时事件能否冒泡到父组件

#### 知识点详解

**答案：能。**

这是 React 刻意设计的行为。Portal 中的事件在 React 组件树层面冒泡，而非 DOM 树层面。

```jsx
import ReactDOM from 'react-dom';

function Parent() {
    const handleClick = (e) => {
        console.log('父组件捕获到点击');
        console.log('e.currentTarget:', e.currentTarget); // 指向父组件 DOM
        console.log('e.target:', e.target); // 指向 Portal 内的实际 DOM
    };

    return (
        <div onClick={handleClick} style={{ padding: 20, background: '#eee' }}>
            <h2>父组件</h2>
            <p>点击下方 Portal 按钮，事件会冒泡到这里</p>

            {/* Portal 渲染到 body，但事件仍会冒泡到父组件 */}
            <MyPortal>
                <button>Portal 内按钮</button>
            </MyPortal>
        </div>
    );
}

function MyPortal({ children }) {
    return ReactDOM.createPortal(children, document.body);
}

// 验证流程：
// 1. 点击 "Portal 内按钮"
// 2. 事件在 DOM 树中冒泡到 body（因为 Portal 的 DOM 节点在 body 下）
// 3. 同时在 React 组件树中冒泡到 Parent
// 4. Parent 的 onClick 被触发
```

**图解执行顺序：**

```
DOM 树结构：
body
  └── #root
        └── div (Parent) ← onClick 在这里
              └── ...

  └── Portal 的 DOM 节点（实际在 body 下）

React 组件树结构：
Parent
  └── MyPortal (Portal)
        └── button

点击 button 后的冒泡路径：
React 组件树: button → MyPortal → Parent ✓
DOM 树:       button → ... → body
```

**阻止 Portal 事件冒泡：**

```jsx
// 方法1：只阻止 React 组件树冒泡
function MyPortal({ children }) {
    return ReactDOM.createPortal(
        <div onClick={e => e.stopPropagation()}>
            {children}
        </div>,
        document.body
    );
}

// 方法2：同时阻止 DOM 树冒泡（更彻底）
function MyPortal({ children }) {
    return ReactDOM.createPortal(
        <div onClick={e => {
            e.stopPropagation();
            e.nativeEvent.stopImmediatePropagation();
        }}>
            {children}
        </div>,
        document.body
    );
}
```

#### 真实面试题

**题目：子组件是一个 Portal，发生点击事件能冒泡到父组件吗？**

**满分答案：**

**能。** 这是 React 刻意为之的设计。

**事件基于组件树冒泡（不是 DOM 树）：**

虽然 Portal 的 DOM 节点渲染在 `document.body` 下（或其他容器），但在 React 的虚拟 DOM 树中，它仍然是父组件的子组件。所以 Portal 中的点击事件会沿着 React 组件树冒泡到父组件——父组件的 `onClick` 可以捕获到 Portal 内元素的点击。

**代码验证：**

```jsx
function Parent() {
    return (
        <div onClick={() => console.log('父组件被点击')}>
            <Portal>
                <button>按钮</button> {/* 点击这里，父组件的 onClick 也会触发 */}
            </Portal>
        </div>
    );
}
```

控制台会输出"父组件被点击"。这个机制的好处是：父组件可以在一个地方统一处理来自 Portal 的事件（比如模态框的关闭、弹出层的点击外部关闭等），而不需要 Portal 内部手动 emit 或 callback。

---

## 5.10 废弃的生命周期

### 5.10.1 React 为何废弃 componentWillMount 等生命周期

#### 知识点详解

**被废弃的生命周期：**

| 废弃钩子 | 标记为 unsafe | 替代方案 |
|---|---|---|
| `componentWillMount` | React 16.3 | `constructor` 或 `useEffect` |
| `componentWillReceiveProps` | React 16.3 | `getDerivedStateFromProps` 或 `useEffect` |
| `componentWillUpdate` | React 16.3 | `getSnapshotBeforeUpdate` |

```jsx
// React 17 之前
class MyComponent extends React.Component {
    componentWillMount() {
        // 危险：Fiber 架构下可能被多次调用
    }

    componentWillReceiveProps(nextProps) {
        // 危险：可能被 Fiber 打断后重新调用
        if (nextProps.value !== this.props.value) {
            this.setState({ derived: nextProps.value });
        }
    }

    componentWillUpdate(nextProps, nextState) {
        // 危险：更新前可能被打断
    }
}
```

**为什么废弃？**

```javascript
// 问题1：Fiber 架构下可能被多次调用
// Fiber 架构中，渲染工作可中断、恢复
// 如果 componentWillMount 执行到一半被打断，恢复后可能再次调用
// 导致：重复请求、状态不一致

// 问题2：容易导致服务端/客户端状态不一致
// componentWillMount 在服务端和客户端都会执行
// 如果在里请求数据，可能出现服务端渲染和客户端渲染结果不同

// 问题3：阻碍异步渲染（Concurrent Mode）
// React 未来要支持 Suspense、Streaming SSR 等特性
// 这些生命周期在新的渲染模型下行为不可预测

// 问题4：命名误导
// componentWillMount 不等于 "挂载前" — 服务端渲染时也会调用
// componentWillReceiveProps 不等于 "props 变了" — 首次渲染时也会调用
```

**替代方案详解：**

```jsx
// 替代 componentWillMount
// 方案1：constructor
class MyComponent extends React.Component {
    constructor(props) {
        super(props);
        this.state = { data: null };
        // 同步初始化逻辑放这里
        this.state = { data: fetchData() };  // 可以，但会阻塞渲染
    }
}

// 方案2：useEffect（推荐）
function MyComponent({ id }) {
    const [data, setData] = useState(null);

    useEffect(() => {
        // 等效于 componentDidMount + componentDidUpdate
        fetchData(id).then(setData);
    }, [id]);

    return <div>{data}</div>;
}

// 替代 componentWillReceiveProps
// 方案1：getDerivedStateFromProps（静态方法，不推荐）
static getDerivedStateFromProps(nextProps, prevState) {
    // 每次 props 或 state 变化时调用
    // 必须返回一个对象来更新 state，或 null 表示不更新
    if (nextProps.value !== prevState.lastValue) {
        return { derived: nextProps.value, lastValue: nextProps.value };
    }
    return null;
}

// 方案2：useEffect（推荐）
function MyComponent({ value }) {
    const [derived, setDerived] = useState(value);

    useEffect(() => {
        setDerived(value);
    }, [value]);  // value 变化时同步

    return <div>{derived}</div>;
}

// 替代 componentWillUpdate
// getSnapshotBeforeUpdate（必须配合 componentDidUpdate）
class MyComponent extends React.Component {
    getSnapshotBeforeUpdate(prevProps, prevState) {
        // 在 DOM 更新之前调用
        // 可以访问更新前的 DOM 属性
        return this.listRef.scrollHeight - this.listRef.scrollTop;
    }

    componentDidUpdate(prevProps, prevState, snapshot) {
        // snapshot 是 getSnapshotBeforeUpdate 返回的值
        this.listRef.scrollTop = this.listRef.scrollHeight - snapshot;
    }
}
```

#### 真实面试题

**题目：React 为什么要废弃 componentWillMount、componentWillReceiveProps、componentWillUpdate？**

**满分答案：**

**核心原因：Fiber 架构下的调用问题：**

在 React 15 的 Stack Reconciler 中，渲染是同步且不可中断的，所以这些生命周期只会被调用一次。但 React 16 引入 Fiber 架构后，渲染工作可以被**中断、恢复、再执行**——这意味着这些生命周期可能被调用多次。

例如 `componentWillMount`：如果在执行过程中被打断（比如高优先级任务插入），恢复后 `componentWillMount` 会再次被调用，导致重复请求、状态混乱。

**具体问题：**

- `componentWillMount`：服务端/客户端都会调用，容易产生 SSR 不一致；可能被 Fiber 打断恢复后多次调用
- `componentWillReceiveProps`：首次渲染时也会调用（名字有误导性）；容易被 Fiber 打断
- `componentWillUpdate`：更新前可能被打断，恢复后重新执行

**每个废弃钩子的替代方案：**

| 废弃钩子 | 替代方案 |
|---|---|
| `componentWillMount` | `constructor`（同步初始化）或 `useEffect`（异步请求） |
| `componentWillReceiveProps` | `getDerivedStateFromProps`（静态方法）或 `useEffect` |
| `componentWillUpdate` | `getSnapshotBeforeUpdate` + `componentDidUpdate` |

**最佳实践**：在 React 16.3+ 中，能用函数组件 + Hooks 解决的问题，就不要用类组件的这些生命周期钩子。

---

## 5.11 render 方法原理

### 5.11.1 React render 的执行时机与触发条件

#### 知识点详解

**render 方法的执行流程：**

```jsx
// 类组件 render
class MyComponent extends React.Component {
    render() {
        // 返回 JSX（虚拟 DOM）
        return <div>{this.props.name}</div>;
    }
}

// 函数组件（本身就是 render 逻辑）
function MyComponent({ name }) {
    return <div>{name}</div>;
}

// render 的本质：
// 1. 执行组件函数或 render 方法
// 2. 返回 JSX（React Element）
// 3. React 创建/更新虚拟 DOM
// 4. Diff 算法对比新旧虚拟 DOM
// 5. 找出最小变更，提交到真实 DOM
```

**触发 render 的5个条件：**

```jsx
// 条件1：首次挂载（mount）
// 组件第一次渲染时必定执行 render
const root = createRoot(document.getElementById('root'));
root.render(<App />);  // App 及其子树首次 render

// 条件2：setState / forceUpdate 调用
class Counter extends React.Component {
    state = { count: 0 };

    handleClick = () => {
        this.setState({ count: this.state.count + 1 });
        // forceUpdate 也会触发
        // this.forceUpdate();
    };

    render() {
        return <button onClick={this.handleClick}>{this.state.count}</button>;
    }
}

// 条件3：父组件重新渲染（props 变化）
// 父组件 render → 子组件的 props 可能变化 → 子组件 render
function Parent() {
    const [color, setColor] = useState('red');
    return (
        <div>
            <button onClick={() => setColor('blue')}>改变颜色</button>
            {/* Child 会 render（即使 color 不是它的 props） */}
            <Child />
        </div>
    );
}

// 条件4：useContext 的 Provider 值变化
const ThemeContext = React.createContext('light');

function App() {
    const [theme, setTheme] = useState('light');

    return (
        <ThemeContext.Provider value={theme}>
            <ThemedButton /> {/* theme 变化时，ThemedButton 会 render */}
        </ThemeContext.Provider>
    );
}

// 条件5：Context 变化
// 任何订阅了该 Context 的组件都会重新 render
const UserContext = React.createContext(null);

function App() {
    const [user, setUser] = useState({ name: 'John' });

    return (
        <UserContext.Provider value={user}>
            <Dashboard /> {/* user 引用变化时，Dashboard 及其子树 render */}
        </UserContext.Provider>
    );
}
```

**不触发 render 的情况：**

```jsx
// 情况1：props 未变且父组件未重渲染
// 子组件不会 render

// 情况2：shouldComponentUpdate 返回 false
classoptimizedComponent extends React.Component {
    shouldComponentUpdate(nextProps, nextState) {
        // 返回 false 阻止 render
        return nextProps.id !== this.props.id;
    }

    render() {
        return <div>{this.props.name}</div>;
    }
}

// 情况3：PureComponent 浅比较相等
classoptimizedComponent extends React.PureComponent {
    render() {
        return <div>{this.props.name}</div>;
    }
}
// PureComponent 自动对 props 和 state 做浅比较

// 情况4：React.memo 浅比较相等
constoptimizedComponent = React.memo(({ name }) => {
    return <div>{name}</div>;
});

// 情况5：useMemo 返回值不变
const expensive = useMemo(() => computeExpensive(a, b), [a, b]);
// 只有 a 或 b 变化时才重新计算，组件不一定重新 render（取决于父组件）
```

#### 真实面试题

**题目：说说 React render 方法的原理？在什么时候会被触发？**

**满分答案：**

**render 的原理：**

`render` 方法（类组件）或函数组件本身，返回 JSX/React Element。React 将这个 Element 转化为虚拟 DOM 节点，通过 diff 算法对比新旧虚拟 DOM，找出最小变更后提交到真实 DOM。

**5个触发条件：**

1. **首次挂载**：组件第一次渲染时必定执行
2. **`setState` / `forceUpdate`**：状态变化触发重新渲染
3. **父组件重新渲染**：父组件 render → props 可能变化 → 子组件 render
4. **`useContext` 的 Provider 值变化**：所有订阅该 Context 的组件重新 render
5. **Context 变化**：任何 Context 值变化，订阅的组件都会 render

**不触发的情况：**

- props 未变但父组件未重渲染
- `shouldComponentUpdate` 返回 `false`
- `PureComponent` 浅比较相等
- `React.memo` 浅比较相等

**性能优化手段：**

- `shouldComponentUpdate`：手动控制是否更新
- `PureComponent`/`React.memo`：自动浅比较
- `useMemo`/`useCallback`：稳定引用，减少不必要的渲染

---

## 5.12 React 事件与原生事件的执行顺序

### 5.12.1 React 17 前后的事件绑定机制与完整执行顺序

#### 知识点详解

**React 17 前后的绑定变化：**

```jsx
// React 17 之前：所有 React 事件绑定在 document 上
// 整个应用共用一个事件监听器（根节点在 document）
document.addEventListener('click', reactEventHandler);

// 问题：
// 1. 第三方库也可能绑定在 document，冲突
// 2. 微前端场景下，多个 React 版本各自在 document 绑定，混乱
// 3. React 卸载时需要手动 removeEventListener

// React 17+：事件绑定在 root 容器上
<div id="root"></div>
rootContainer.addEventListener('click', reactEventHandler);
// 每个 React 应用独立，不污染 document
```

**完整执行顺序（React 17+）：**

```
点击事件触发 →

1. 原生捕获阶段
   document → html → body → ... → target

2. React 捕获阶段（root 容器上）
   root → ... → target

3. 原生目标阶段
   target（事件源元素）

4. React 冒泡阶段（root 容器上）
   target → ... → root

5. 原生冒泡阶段
   target → ... → body → html → document
```

```jsx
// React 17+ 完整执行顺序演示
// 层级结构：
// <div id="root">  (React root)
//   <div className="outer">   ← 父组件
//     <div className="inner">   ← 子组件（Portal 渲染到 body）
//     </div>
//   </div>
// </div>

// 实际 DOM 结构：
// <div id="root">...</div>
// <div class="portal">...</div>  ← Portal 在 body 下

// 点击 .inner 后的完整执行顺序：
// 1. native capture:  document → ... → .inner
// 2. react capture:    #root → .outer → .inner
// 3. native target:    .inner
// 4. react bubble:      .inner → .outer → #root
// 5. native bubble:     .inner → ... → document
```

**阻止冒泡的区别：**

```jsx
// 只能阻止 React 事件的冒泡（react capture/bubble）
e.stopPropagation();

// 阻止原生事件的冒泡（包括其他原生事件监听器）
e.nativeEvent.stopImmediatePropagation();

// 实际例子
function App() {
    return (
        <div
            onClick={() => console.log('React div click')}
            onClickCapture={() => console.log('React div capture')}
        >
            <button
                onClick={(e) => {
                    console.log('React button click');
                    e.stopPropagation();  // 只阻止 React 冒泡
                    // 原生事件继续冒泡到 document
                }}
                onClickCapture={() => console.log('React button capture')}
            >
                点击
            </button>
        </div>
    );
}

// 第三方库的原生事件 vs React 事件
document.addEventListener('click', () => {
    console.log('原生 document 监听器');
    // 这个不会被 e.stopPropagation() 阻止
});

// e.nativeEvent.stopImmediatePropagation() 才能阻止
// 但它会阻止所有注册在同一个元素上的同类原生事件
```

#### 真实面试题

**题目：说说 React 事件和原生事件的执行顺序**

**满分答案：**

React 17 之前，所有 React 事件都绑定在 `document` 上；React 17+ 改为绑定在 `root` 容器上。

完整执行顺序（React 17+）：
1. 原生捕获阶段（从外到内）
2. React 捕获阶段（React 17+ 在 root 上统一处理）
3. 原生目标阶段
4. React 冒泡阶段（React 17+ 在 root 上统一处理）
5. 原生冒泡阶段（从内到外）

关键区别：
- `e.stopPropagation()` 只阻止 React 事件冒泡，**不会**阻止原生事件冒泡
- `e.nativeEvent.stopImmediatePropagation()` 才能阻止原生事件
- React 事件是基于**组件树**的，而非 DOM 树（Portal 的 DOM 在 body 下，但事件仍冒泡到 React 父组件）

---

## 5.13 受控组件与非受控组件

#### 知识点详解

**受控组件：** 表单的值由 React state 控制，每次输入都通过 `onChange` 更新 state，组件重新渲染。

```jsx
function ControlledForm() {
    const [name, setName] = useState('');

    return (
        <input
            value={name}
            onChange={(e) => setName(e.target.value)}
        />
    );
}
```

**非受控组件：** 表单的值由 DOM 自身管理，通过 `ref` 获取值。

```jsx
function UncontrolledForm() {
    const inputRef = useRef(null);

    const handleSubmit = () => {
        console.log(inputRef.current.value);
    };

    return (
        <input
            defaultValue="初始值"
            ref={inputRef}
        />
    );
}
```

**对比：**

| 特性 | 受控组件 | 非受控组件 |
|------|---------|-----------|
| 数据来源 | React state | DOM |
| 更新方式 | setState + re-render | ref 直接读 DOM |
| 实时验证 | ✅ 方便 | ❌ 不方便 |
| 适用场景 | 表单验证、格式化输入 | 简单表单、文件上传 |

**注意：** `<input type="file">` 始终是非受控组件。

#### 真实面试题

**题目：说说对受控组件和非受控组件的理解，以及应用场景?**

**满分答案：**

受控组件的值由 React state 驱动，通过 onChange 更新 state 后重新渲染；非受控组件的值由 DOM 自身管理，通过 ref 读取。受控组件适合需要实时验证、格式化输入、条件禁用的复杂表单；非受控组件适合简单的表单场景，代码更简洁。`<input type="file">` 只能是非受控组件。

---

## 5.14 Redux 在项目中的使用

#### 知识点详解

现代 Redux 使用 **Redux Toolkit (RTK)** + **React-Redux**：

```javascript
// 1. 定义 Slice
import { createSlice, createAsyncThunk, configureStore } from '@reduxjs/toolkit';

const fetchUser = createAsyncThunk('user/fetch', async (id) => {
    const res = await fetch(`/api/users/${id}`);
    return res.json();
});

const userSlice = createSlice({
    name: 'user',
    initialState: { data: null, loading: false, error: null },
    reducers: {
        clearUser: (state) => { state.data = null; }
    },
    extraReducers: (builder) => {
        builder
            .addCase(fetchUser.pending, (state) => { state.loading = true; })
            .addCase(fetchUser.fulfilled, (state, action) => {
                state.loading = false;
                state.data = action.payload;
            })
            .addCase(fetchUser.rejected, (state, action) => {
                state.loading = false;
                state.error = action.error.message;
            });
    }
});

// 2. 配置 Store
const store = configureStore({
    reducer: {
        user: userSlice.reducer,
    }
});

// 3. 组件中使用
import { useSelector, useDispatch } from 'react-redux';

function UserProfile({ id }) {
    const dispatch = useDispatch();
    const { data, loading } = useSelector((state) => state.user);

    useEffect(() => {
        dispatch(fetchUser(id));
    }, [id]);

    if (loading) return <div>加载中...</div>;
    return <div>{data.name}</div>;
}
```

**推荐项目结构：**
```
src/
├── store/
│   ├── index.ts
│   └── slices/
│       ├── userSlice.ts
│       └── counterSlice.ts
├── hooks/
│   └── useTypedSelector.ts
└── features/
    └── user/
        └── UserPage.tsx
```

**最佳实践：**
- 按 feature 模块拆分 slice
- 使用 `createAsyncThunk` 处理异步
- 使用 `createSelector`（reselect）缓存计算结果
- normalize state 结构（`createEntityAdapter`）

#### 真实面试题

**题目：你在React项目中是如何使用Redux的?项目结构是如何划分的?**

**满分答案：**

使用 Redux Toolkit + React-Redux，按 feature 模块拆分 slice（`createSlice` 定义 reducer + action），用 `configureStore` 配置 store，组件中通过 `useSelector` 读状态、`useDispatch` 派发 action。异步请求用 `createAsyncThunk`，选择器用 reselect 缓存。项目结构按 store/slices、features、hooks 划分，state 结构使用 `createEntityAdapter` normalize。

---

## 5.15 Redux 中间件

#### 知识点详解

中间件是对 `dispatch` 的增强，在 action 发出 → reducer 处理之间插入逻辑。

**洋葱模型：**
```
dispatch(action)
  → middleware1 (before next)
    → middleware2 (before next)
      → reducer 执行
    → middleware2 (after next)
  → middleware1 (after next)
```

**实现原理（柯里化）：**
```javascript
const myMiddleware = store => next => action => {
    console.log('before:', action);
    const result = next(action);
    console.log('after:', action);
    return result;
};

function applyMiddleware(...middlewares) {
    return (createStore) => (reducer, preloadedState) => {
        const store = createStore(reducer, preloadedState);
        let dispatch = store.dispatch;
        const midAPI = {
            getState: store.getState,
            dispatch: (action) => dispatch(action)
        };
        const chain = middlewares.map(m => m(midAPI));
        dispatch = compose(...chain)(store.dispatch);
        return { ...store, dispatch };
    };
}
```

**常用中间件对比：**

| 中间件 | 原理 | 适用场景 |
|--------|------|---------|
| redux-thunk | action 返回函数 | 简单异步（RTK 默认内置） |
| redux-saga | Generator 函数 | 复杂异步流程（竞态、取消） |
| redux-observable | RxJS Observable | 复杂事件流（防抖、节流） |
| redux-logger | 打印 action | 开发调试 |

#### 真实面试题

**题目：说说对Redux中间件的理解?常用的中间件有哪些?实现原理?**

**满分答案：**

中间件是增强 dispatch 的函数，采用洋葱模型（`store => next => action` 柯里化），在 action 发出和 reducer 处理之间插入逻辑。常用中间件：redux-thunk（返回函数处理简单异步）、redux-saga（Generator 处理复杂异步流程）、redux-observable（RxJS 处理事件流）。实现核心是对 dispatch 进行高阶函数包装，形成中间件链。

---

## 5.16 Redux 工作原理

#### 知识点详解

**三大原则：**
1. **单一数据源（Single Source of Truth）**：整个应用的状态存储在一棵对象树中
2. **State 只读**：只能通过 dispatch action 修改
3. **纯函数修改**：Reducer 是纯函数，`(state, action) => newState`

**完整数据流：**
```
View (UI)
  │ dispatch(action)
  ▼
Middleware (thunk / saga / logger)
  │
  ▼
Reducer: (prevState, action) => newState
  │
  ▼
Store (新 state)
  │ notify subscribers
  ▼
View (re-render)
```

**核心概念：**
- **Store**：状态容器，`{ getState, dispatch, subscribe }`
- **Action**：描述事件的普通对象 `{ type: 'ADD_TODO', payload: { text: '...' } }`
- **Reducer**：纯函数，根据 action type 返回新 state
- **Selector**：从 state 中派生数据（可缓存）

**Redux vs Context：**
| 特性 | Redux | Context |
|------|-------|---------|
| 中间件 | ✅ 支持异步、日志等 | ❌ 无 |
| DevTools | ✅ 时间旅行调试 | ❌ 基础 |
| 性能优化 | useSelector 精确订阅 | 消费者全部重渲染 |
| 学习成本 | 较高 | 低 |
| 适用规模 | 大型应用 | 简单场景 |

#### 真实面试题

**题目：说说你对Redux的理解?其工作原理?**

**满分答案：**

Redux 遵循三大原则：单一数据源、state 只读、纯函数修改。工作流程是 View dispatch action → 中间件处理 → Reducer 纯函数返回新 state → Store 更新 → View 重新渲染。核心概念包括 Store（状态容器）、Action（事件描述）、Reducer（纯函数）、Selector（数据提取）。Redux 适合大型应用，支持中间件生态和 DevTools 调试；简单场景可用 Context 替代。

