# 四、Vue 框架详解

---

## 4.0 数据驱动视图

#### 知识点详解

**数据驱动视图的概念：**

"数据驱动视图"是指页面的渲染由数据状态决定，而不是直接操作 DOM。开发者只关注数据变化，框架/库自动处理视图更新。

**核心要素：**

1. **状态（State）**
   - 描述 UI 当前状况的数据
   - 状态变化 → 视图自动更新

2. **响应式系统（Reactivity）**
   - 监听数据变化
   - 数据变化时自动触发更新

3. **虚拟 DOM / 渲染函数**
   - 状态 → 描述 UI 的数据结构
   - 数据变化 → 重新生成 UI 描述
   - 对比差异 → 最小化 DOM 操作

4. **单向数据流**
   - 数据从顶层向底层传递
   - 用户交互 → 触发 action → 更新 state → 触发渲染

**Vue 的数据驱动实现：**

```javascript
// Vue 3 响应式原理
const reactive = (target) => {
    return new Proxy(target, {
        get(target, key, receiver) {
            track(target, key); // 收集依赖
            return Reflect.get(target, key, receiver);
        },
        set(target, key, value, receiver) {
            trigger(target, key); // 触发更新
            return Reflect.set(target, key, value, receiver);
        }
    });
};
```

#### 真实面试题

**题目：如何理解"数据驱动视图"？Vue 和 React 分别是如何实现的？**

**满分答案：**

**数据驱动视图的理解：**
- 开发者只关心数据变化
- 框架自动处理 DOM 更新
- 分离数据和视图关注点

**Vue 的实现：**
1. 使用 Proxy/defineProperty 劫持数据 getter/setter
2. 数据变化时，通过 Dep 收集的依赖触发更新
3. 组件 render 函数执行，生成虚拟 DOM
4. 渲染真实 DOM

**React 的实现：**
1. useState/setState 修改状态
2. 触发组件重新渲染
3. 生成新的虚拟 DOM 树
4. Diff 算法对比新旧虚拟 DOM
5. 最小化更新真实 DOM

**两者区别：**
- Vue：数据劫持 + 细粒度更新
- React：组件级重新渲染 + Diff 合并

---

## 4.1 Vue 3 组合式 API

### 4.1.1 setup 与响应式基础

#### 知识点详解

**setup 函数：**

```javascript
import { ref, reactive, computed, watch, watchEffect, onMounted } from 'vue';

export default {
    props: {
        title: String
    },
    setup(props, context) {
        // context 包含 attrs, slots, emit, expose
        const { attrs, slots, emit, expose } = context;
        
        // 响应式数据
        const count = ref(0);
        const state = reactive({ name: 'Vue', version: 3 });
        
        // 计算属性
        const doubleCount = computed(() => count.value * 2);
        
        // 方法
        function increment() {
            count.value++;
        }
        
        // 生命周期
        onMounted(() => {
            console.log('组件已挂载');
        });
        
        // 暴露给模板
        return {
            count,
            state,
            doubleCount,
            increment
        };
    }
};
```

**`<script setup>` 语法糖（推荐）：**

```vue
<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue';

// 直接声明，自动暴露给模板
const count = ref(0);
const state = reactive({ name: 'Vue' });

const doubleCount = computed(() => count.value * 2);

function increment() {
    count.value++;
}

onMounted(() => {
    console.log('组件已挂载');
});

// defineProps 和 defineEmits 是编译器宏，无需导入
const props = defineProps({
    title: String,
    count: {
        type: Number,
        default: 0
    }
});

const emit = defineEmits(['update:count', 'change']);

// 暴露给父组件（通过 ref）
defineExpose({
    count,
    increment
});
</script>
```

### 4.1.2 ref 与 reactive

**ref：**

```javascript
import { ref, isRef, unref } from 'vue';

// 基本类型
const count = ref(0);
console.log(count.value);  // 0
count.value++;

// 对象类型（内部使用 reactive）
const user = ref({ name: 'John', age: 30 });
user.value.name = 'Jane';  // 响应式

// 模板中自动解包（不需要 .value）
// <template>{{ count }}</template>

// 检测
isRef(count);  // true
unref(count);  // 0（等同于 isRef(count) ? count.value : count）

// DOM 引用
const el = ref(null);
// <div ref="el"></div>
onMounted(() => {
    console.log(el.value);  // DOM 元素
});
```

**reactive：**

```javascript
import { reactive, isReactive, toRaw, markRaw } from 'vue';

// 对象
const state = reactive({
    count: 0,
    user: {
        name: 'John',
        age: 30
    }
});

state.count++;
state.user.name = 'Jane';

// 注意：不能解构（会失去响应性）
const { count } = state;  // count 不是响应式的

// 使用 toRefs 保持响应性
import { toRefs } from 'vue';
const { count, user } = toRefs(state);
count.value++;  // 仍然响应式

// 数组
const list = reactive([1, 2, 3]);
list.push(4);
list[0] = 0;

// 检测
isReactive(state);  // true

// 获取原始对象
const raw = toRaw(state);

// 标记为不可响应
const nonReactive = markRaw({ name: 'raw' });
```

**ref vs reactive：**

| 特性 | ref | reactive |
|------|-----|----------|
| 适用类型 | 任意类型 | 对象/数组 |
| 访问方式 | `.value` | 直接访问 |
| 解构 | 需要 toRefs | 需要 toRefs |
| 模板中 | 自动解包 | 直接使用 |
| 替换整个值 | 支持 | 不支持 |

### 4.1.3 computed 与 watch

**computed：**

```javascript
import { ref, computed } from 'vue';

const firstName = ref('John');
const lastName = ref('Doe');

// 只读计算属性
const fullName = computed(() => `${firstName.value} ${lastName.value}`);

// 可写计算属性
const fullNameWritable = computed({
    get() {
        return `${firstName.value} ${lastName.value}`;
    },
    set(newValue) {
        const parts = newValue.split(' ');
        firstName.value = parts[0];
        lastName.value = parts[1];
    }
});

fullNameWritable.value = 'Jane Smith';
console.log(firstName.value);  // 'Jane'
```

**watch：**

```javascript
import { ref, reactive, watch } from 'vue';

const count = ref(0);
const state = reactive({ name: 'John' });

// 监听 ref
watch(count, (newValue, oldValue) => {
    console.log(`count: ${oldValue} -> ${newValue}`);
});

// 监听 reactive 的属性（需要 getter）
watch(
    () => state.name,
    (newValue, oldValue) => {
        console.log(`name: ${oldValue} -> ${newValue}`);
    }
);

// 监听多个源
watch([count, () => state.name], ([newCount, newName], [oldCount, oldName]) => {
    console.log('count or name changed');
});

// 选项
watch(count, (newValue) => {
    console.log(newValue);
}, {
    immediate: true,   // 立即执行
    deep: true,        // 深度监听
    flush: 'post'      // 在 DOM 更新后执行
});

// 停止监听
const stop = watch(count, () => {});
stop();  // 停止监听
```

**watchEffect：**

```javascript
import { ref, watchEffect } from 'vue';

const count = ref(0);
const name = ref('John');

// 自动追踪依赖
const stop = watchEffect(() => {
    console.log(`count: ${count.value}, name: ${name.value}`);
    // 自动追踪 count 和 name
});

// 清理副作用
watchEffect((onCleanup) => {
    const timer = setInterval(() => {
        console.log(count.value);
    }, 1000);
    
    onCleanup(() => {
        clearInterval(timer);
    });
});

// 停止
stop();
```

**watch vs watchEffect：**

| 特性 | watch | watchEffect |
|------|-------|-------------|
| 依赖追踪 | 显式指定 | 自动追踪 |
| 初始执行 | 默认不执行 | 立即执行 |
| 旧值访问 | 支持 | 不支持 |
| 懒执行 | 支持 | 不支持 |

### 4.1.4 生命周期

```javascript
import {
    onBeforeMount,
    onMounted,
    onBeforeUpdate,
    onUpdated,
    onBeforeUnmount,
    onUnmounted,
    onErrorCaptured,
    onActivated,
    onDeactivated
} from 'vue';

export default {
    setup() {
        // 对应 beforeCreate 和 created（setup 本身就是）
        
        onBeforeMount(() => {
            console.log('挂载前：DOM 还未创建');
        });
        
        onMounted(() => {
            console.log('挂载后：DOM 已创建，可以操作 DOM');
        });
        
        onBeforeUpdate(() => {
            console.log('更新前：数据已变化，DOM 还未更新');
        });
        
        onUpdated(() => {
            console.log('更新后：DOM 已更新');
        });
        
        onBeforeUnmount(() => {
            console.log('卸载前：组件还在，可以清理');
        });
        
        onUnmounted(() => {
            console.log('卸载后：组件已销毁');
        });
        
        // 错误捕获
        onErrorCaptured((error, instance, info) => {
            console.error('捕获到错误:', error);
            return false;  // 阻止错误继续传播
        });
        
        // keep-alive 相关
        onActivated(() => {
            console.log('组件被激活');
        });
        
        onDeactivated(() => {
            console.log('组件被停用');
        });
    }
};
```

**Vue 3 vs Vue 2 生命周期对比：**

| Vue 2 | Vue 3 (Options API) | Vue 3 (Composition API) |
|-------|---------------------|-------------------------|
| beforeCreate | beforeCreate | setup() |
| created | created | setup() |
| beforeMount | beforeMount | onBeforeMount |
| mounted | mounted | onMounted |
| beforeUpdate | beforeUpdate | onBeforeUpdate |
| updated | updated | onUpdated |
| beforeDestroy | beforeUnmount | onBeforeUnmount |
| destroyed | unmounted | onUnmounted |

#### 真实面试题

**题目1：Vue 3 的 `ref` 和 `reactive` 有什么区别？如何选择？**

**满分答案：**

**区别：**

1. **适用类型**
   - `ref`：适用于任何类型（基本类型、对象、数组）
   - `reactive`：只适用于对象类型（对象、数组、Map、Set）

2. **访问方式**
   - `ref`：在 JS 中需要 `.value`，在模板中自动解包
   - `reactive`：直接访问属性

3. **解构行为**
   - `ref`：解构后仍然是 ref，保持响应性
   - `reactive`：解构后失去响应性，需要 `toRefs`

4. **替换整个值**
   - `ref`：可以替换整个值（`count.value = newValue`）
   - `reactive`：不能替换整个对象（会失去响应性）

```javascript
// ref 可以替换整个值
const user = ref({ name: 'John' });
user.value = { name: 'Jane' };  // OK，仍然响应式

// reactive 不能替换整个对象
const state = reactive({ name: 'John' });
state = { name: 'Jane' };  // 错误！失去响应性

// 正确做法
Object.assign(state, { name: 'Jane' });
```

**选择建议：**

```javascript
// 推荐使用 ref（更统一）
const count = ref(0);
const user = ref({ name: 'John' });
const list = ref([1, 2, 3]);

// reactive 适合管理一组相关状态
const formState = reactive({
    username: '',
    password: '',
    rememberMe: false
});
```

---

**题目2：Vue 3 的响应式原理是什么？与 Vue 2 有什么区别？**

**满分答案：**

**Vue 2 响应式原理（Object.defineProperty）：**

```javascript
function defineReactive(obj, key, val) {
    const dep = new Dep();
    
    Object.defineProperty(obj, key, {
        get() {
            dep.depend();  // 收集依赖
            return val;
        },
        set(newVal) {
            if (newVal === val) return;
            val = newVal;
            dep.notify();  // 通知更新
        }
    });
}
```

**Vue 2 的局限性：**
- 无法检测对象属性的添加和删除（需要 `Vue.set`）
- 无法检测数组索引和长度变化（需要重写数组方法）
- 需要递归遍历所有属性，初始化性能差

**Vue 3 响应式原理（Proxy）：**

```javascript
function reactive(target) {
    return new Proxy(target, {
        get(target, key, receiver) {
            track(target, key);  // 收集依赖
            const result = Reflect.get(target, key, receiver);
            // 深层响应式
            if (typeof result === 'object' && result !== null) {
                return reactive(result);
            }
            return result;
        },
        set(target, key, value, receiver) {
            const result = Reflect.set(target, key, value, receiver);
            trigger(target, key);  // 触发更新
            return result;
        },
        deleteProperty(target, key) {
            const result = Reflect.deleteProperty(target, key);
            trigger(target, key);
            return result;
        }
    });
}
```

**Vue 3 的优势：**
- 可以检测属性添加和删除
- 可以检测数组索引和长度变化
- 懒代理（访问时才代理），性能更好
- 支持 Map、Set、WeakMap、WeakSet

**对比总结：**

| 特性 | Vue 2 | Vue 3 |
|------|-------|-------|
| 实现方式 | Object.defineProperty | Proxy |
| 属性添加 | 需要 Vue.set | 自动检测 |
| 属性删除 | 需要 Vue.delete | 自动检测 |
| 数组索引 | 不支持 | 支持 |
| 数组长度 | 不支持 | 支持 |
| 初始化性能 | 递归遍历，较慢 | 懒代理，较快 |
| 浏览器兼容 | IE9+ | 不支持 IE |

---

## 4.2 Vue Router

### 4.2.1 路由配置

```javascript
import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router';

const routes = [
    {
        path: '/',
        name: 'Home',
        component: () => import('./views/Home.vue'),  // 懒加载
        meta: {
            title: '首页',
            requiresAuth: false
        }
    },
    {
        path: '/user/:id',  // 动态路由
        name: 'User',
        component: () => import('./views/User.vue'),
        props: true  // 将路由参数作为 props 传递
    },
    {
        path: '/admin',
        component: () => import('./views/Admin.vue'),
        meta: { requiresAuth: true },
        children: [  // 嵌套路由
            {
                path: '',
                name: 'AdminDashboard',
                component: () => import('./views/AdminDashboard.vue')
            },
            {
                path: 'users',
                name: 'AdminUsers',
                component: () => import('./views/AdminUsers.vue')
            }
        ]
    },
    {
        path: '/redirect',
        redirect: '/'  // 重定向
    },
    {
        path: '/:pathMatch(.*)*',  // 404 页面
        name: 'NotFound',
        component: () => import('./views/NotFound.vue')
    }
];

const router = createRouter({
    history: createWebHistory(),  // HTML5 History 模式
    // history: createWebHashHistory(),  // Hash 模式
    routes,
    scrollBehavior(to, from, savedPosition) {
        if (savedPosition) {
            return savedPosition;
        }
        return { top: 0 };
    }
});

export default router;
```

### 4.2.2 路由守卫

```javascript
// 全局前置守卫
router.beforeEach((to, from, next) => {
    // 检查是否需要认证
    if (to.meta.requiresAuth && !isAuthenticated()) {
        next({ name: 'Login', query: { redirect: to.fullPath } });
    } else {
        next();
    }
});

// 全局后置守卫
router.afterEach((to, from) => {
    document.title = to.meta.title || '默认标题';
});

// 路由独享守卫
const routes = [
    {
        path: '/admin',
        component: Admin,
        beforeEnter: (to, from, next) => {
            if (!isAdmin()) {
                next('/');
            } else {
                next();
            }
        }
    }
];

// 组件内守卫
export default {
    beforeRouteEnter(to, from, next) {
        // 进入路由前，此时组件实例还未创建
        next(vm => {
            // 通过 vm 访问组件实例
        });
    },
    beforeRouteUpdate(to, from, next) {
        // 路由参数变化时（如 /user/1 -> /user/2）
        next();
    },
    beforeRouteLeave(to, from, next) {
        // 离开路由前
        if (this.hasUnsavedChanges) {
            const answer = window.confirm('确定要离开吗？');
            if (answer) {
                next();
            } else {
                next(false);
            }
        } else {
            next();
        }
    }
};
```

### 4.2.3 编程式导航

```javascript
import { useRouter, useRoute } from 'vue-router';

export default {
    setup() {
        const router = useRouter();
        const route = useRoute();
        
        // 获取路由参数
        console.log(route.params.id);
        console.log(route.query.page);
        console.log(route.meta.title);
        
        // 导航
        function navigate() {
            // 字符串路径
            router.push('/user/1');
            
            // 对象
            router.push({ path: '/user/1' });
            
            // 命名路由
            router.push({ name: 'User', params: { id: 1 } });
            
            // 带查询参数
            router.push({ path: '/search', query: { q: 'vue' } });
            
            // 替换当前历史记录
            router.replace('/user/1');
            
            // 前进/后退
            router.go(1);
            router.go(-1);
            router.back();
            router.forward();
        }
        
        return { navigate };
    }
};
```

---

## 4.3 Pinia 状态管理

### 4.3.1 基础用法

```javascript
// stores/counter.js
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';

// 选项式 Store
export const useCounterStore = defineStore('counter', {
    state: () => ({
        count: 0,
        name: 'Eduardo'
    }),
    getters: {
        doubleCount: (state) => state.count * 2,
        doubleCountPlusOne() {
            return this.doubleCount + 1;
        }
    },
    actions: {
        increment() {
            this.count++;
        },
        async fetchUser(userId) {
            const user = await fetch(`/api/user/${userId}`);
            this.name = user.name;
        }
    }
});

// 组合式 Store（推荐）
export const useCounterStore = defineStore('counter', () => {
    const count = ref(0);
    const name = ref('Eduardo');
    
    const doubleCount = computed(() => count.value * 2);
    
    function increment() {
        count.value++;
    }
    
    async function fetchUser(userId) {
        const user = await fetch(`/api/user/${userId}`);
        name.value = user.name;
    }
    
    return { count, name, doubleCount, increment, fetchUser };
});
```

**在组件中使用：**

```vue
<script setup>
import { storeToRefs } from 'pinia';
import { useCounterStore } from '@/stores/counter';

const store = useCounterStore();

// 解构（保持响应性）
const { count, name, doubleCount } = storeToRefs(store);

// 方法可以直接解构
const { increment, fetchUser } = store;

// 修改状态
store.count++;
store.$patch({ count: 10, name: 'John' });
store.$patch((state) => {
    state.count++;
    state.name = 'John';
});

// 重置状态
store.$reset();

// 订阅状态变化
store.$subscribe((mutation, state) => {
    console.log(mutation.type);  // 'direct' | 'patch object' | 'patch function'
    console.log(state);
});

// 订阅 action
store.$onAction(({ name, args, after, onError }) => {
    console.log(`Action ${name} called with`, args);
    after((result) => {
        console.log(`Action ${name} finished with`, result);
    });
    onError((error) => {
        console.error(`Action ${name} failed with`, error);
    });
});
</script>
```

---

## 4.4 Vue 组件通信

### 4.4.1 父子组件通信

```vue
<!-- 父组件 -->
<template>
    <ChildComponent
        :title="title"
        :count="count"
        @update:count="count = $event"
        @change="handleChange"
    />
</template>

<script setup>
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const title = ref('Hello');
const count = ref(0);

function handleChange(value) {
    console.log('子组件触发了 change 事件:', value);
}
</script>

<!-- 子组件 -->
<template>
    <div>
        <h1>{{ title }}</h1>
        <p>{{ count }}</p>
        <button @click="increment">+1</button>
    </div>
</template>

<script setup>
const props = defineProps({
    title: {
        type: String,
        required: true
    },
    count: {
        type: Number,
        default: 0
    }
});

const emit = defineEmits(['update:count', 'change']);

function increment() {
    emit('update:count', props.count + 1);
    emit('change', props.count + 1);
}
</script>
```

### 4.4.2 provide/inject（跨级通信）

```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from 'vue';

const theme = ref('dark');
const user = ref({ name: 'John' });

// 提供响应式数据
provide('theme', theme);
provide('user', user);

// 提供方法
provide('updateTheme', (newTheme) => {
    theme.value = newTheme;
});
</script>

<!-- 后代组件（任意层级） -->
<script setup>
import { inject } from 'vue';

const theme = inject('theme', 'light');  // 第二个参数是默认值
const user = inject('user');
const updateTheme = inject('updateTheme');

function toggleTheme() {
    updateTheme(theme.value === 'dark' ? 'light' : 'dark');
}
</script>
```

### 4.4.3 插槽

```vue
<!-- 父组件 -->
<template>
    <BaseCard>
        <!-- 默认插槽 -->
        <p>默认内容</p>
        
        <!-- 具名插槽 -->
        <template #header>
            <h1>卡片标题</h1>
        </template>
        
        <!-- 作用域插槽 -->
        <template #footer="{ user }">
            <p>{{ user.name }}</p>
        </template>
    </BaseCard>
</template>

<!-- BaseCard 组件 -->
<template>
    <div class="card">
        <div class="card-header">
            <slot name="header">默认标题</slot>
        </div>
        
        <div class="card-body">
            <slot>默认内容</slot>
        </div>
        
        <div class="card-footer">
            <slot name="footer" :user="currentUser">
                默认底部
            </slot>
        </div>
    </div>
</template>

<script setup>
const currentUser = { name: 'John', age: 30 };
</script>
```

---

## 4.5 Vue 进阶特性

### 4.5.1 自定义指令

```javascript
// 全局注册
app.directive('focus', {
    mounted(el) {
        el.focus();
    }
});

// 局部注册（在 <script setup> 中）
const vFocus = {
    mounted(el) {
        el.focus();
    }
};

// 完整钩子
const vMyDirective = {
    created(el, binding, vnode, prevVnode) {},
    beforeMount(el, binding, vnode, prevVnode) {},
    mounted(el, binding, vnode, prevVnode) {},
    beforeUpdate(el, binding, vnode, prevVnode) {},
    updated(el, binding, vnode, prevVnode) {},
    beforeUnmount(el, binding, vnode, prevVnode) {},
    unmounted(el, binding, vnode, prevVnode) {}
};

// binding 对象
// binding.value - 指令的值
// binding.oldValue - 上一个值
// binding.arg - 指令参数
// binding.modifiers - 修饰符对象

// 使用示例
const vPermission = {
    mounted(el, binding) {
        const { value } = binding;
        const userPermissions = store.getters.permissions;
        
        if (!userPermissions.includes(value)) {
            el.parentNode?.removeChild(el);
        }
    }
};

// 模板中使用
// <button v-permission="'admin'">管理员按钮</button>
```

### 4.5.2 Teleport

```vue
<template>
    <div>
        <button @click="showModal = true">打开弹窗</button>
        
        <!-- 将弹窗渲染到 body 下 -->
        <Teleport to="body">
            <div v-if="showModal" class="modal">
                <h2>弹窗标题</h2>
                <p>弹窗内容</p>
                <button @click="showModal = false">关闭</button>
            </div>
        </Teleport>
    </div>
</template>

<script setup>
import { ref } from 'vue';
const showModal = ref(false);
</script>
```

### 4.5.3 Suspense（异步组件）

```vue
<template>
    <Suspense>
        <!-- 异步组件 -->
        <template #default>
            <AsyncComponent />
        </template>
        
        <!-- 加载中的占位内容 -->
        <template #fallback>
            <div>加载中...</div>
        </template>
    </Suspense>
</template>

<script setup>
import { defineAsyncComponent } from 'vue';

const AsyncComponent = defineAsyncComponent({
    loader: () => import('./AsyncComponent.vue'),
    loadingComponent: LoadingComponent,
    errorComponent: ErrorComponent,
    delay: 200,
    timeout: 3000
});
</script>
```

### 4.5.4 渲染函数

```javascript
import { h, ref } from 'vue';

export default {
    setup() {
        const count = ref(0);
        
        return () => h('div', { class: 'container' }, [
            h('h1', '标题'),
            h('p', `计数: ${count.value}`),
            h('button', {
                onClick: () => count.value++
            }, '点击')
        ]);
    }
};

// 使用 JSX（需要配置）
export default {
    setup() {
        const count = ref(0);
        
        return () => (
            <div class="container">
                <h1>标题</h1>
                <p>计数: {count.value}</p>
                <button onClick={() => count.value++}>点击</button>
            </div>
        );
    }
};
```

---

## 4.6 Vue 性能优化

### 4.6.1 组件优化

```vue
<!-- 1. v-once：只渲染一次 -->
<template>
    <div v-once>{{ staticContent }}</div>
</template>

<!-- 2. v-memo：记忆化渲染 -->
<template>
    <div v-for="item in list" :key="item.id" v-memo="[item.id, item.selected]">
        {{ item.name }}
    </div>
</template>

<!-- 3. 使用 shallowRef/shallowReactive 减少深层响应 -->
<script setup>
import { shallowRef, shallowReactive } from 'vue';

const bigData = shallowRef({ /* 大量数据 */ });
const state = shallowReactive({ /* 浅层响应 */ });
</script>

<!-- 4. 使用 defineAsyncComponent 懒加载 -->
<script setup>
import { defineAsyncComponent } from 'vue';

const HeavyComponent = defineAsyncComponent(() =>
    import('./HeavyComponent.vue')
);
</script>

<!-- 5. keep-alive 缓存组件 -->
<template>
    <keep-alive :include="['ComponentA', 'ComponentB']" :max="10">
        <component :is="currentComponent" />
    </keep-alive>
</template>
```

#### 真实面试题

**题目：Vue 3 中如何实现组件间通信？请列举所有方式。**

**满分答案：**

**1. Props / Emits（父子通信）**
```vue
<!-- 父 → 子：props -->
<Child :data="parentData" />

<!-- 子 → 父：emits -->
<Child @update="handleUpdate" />
```

**2. v-model（双向绑定）**
```vue
<!-- 父组件 -->
<Child v-model:title="title" v-model:count="count" />

<!-- 子组件 -->
<script setup>
defineProps(['title', 'count']);
const emit = defineEmits(['update:title', 'update:count']);
</script>
```

**3. provide/inject（跨级通信）**
```javascript
// 祖先
provide('key', value);

// 后代
const value = inject('key');
```

**4. Pinia/Vuex（全局状态管理）**
```javascript
const store = useCounterStore();
store.increment();
```

**5. $refs（父访问子）**
```vue
<Child ref="childRef" />
<script setup>
const childRef = ref(null);
childRef.value.someMethod();
</script>
```

**6. EventBus（任意组件，Vue 3 推荐使用 mitt）**
```javascript
import mitt from 'mitt';
const emitter = mitt();

// 发送
emitter.emit('event', data);

// 接收
emitter.on('event', (data) => {});
```

**7. 路由参数（路由组件间）**
```javascript
router.push({ name: 'User', params: { id: 1 } });
const route = useRoute();
route.params.id;
```

**8. localStorage/sessionStorage（持久化）**
```javascript
localStorage.setItem('key', JSON.stringify(data));
const data = JSON.parse(localStorage.getItem('key'));
```

**选择建议：**

| 场景 | 推荐方式 |
|------|----------|
| 父子组件 | Props + Emits |
| 双向绑定 | v-model |
| 跨级组件 | provide/inject |
| 全局状态 | Pinia |
| 兄弟组件 | Pinia 或 mitt |
| 路由组件 | 路由参数 |
