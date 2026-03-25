# Vue 面试题

> 面试频率：⭐⭐⭐⭐（Vue2 / Vue3 都要掌握）

---

## 一、响应式原理

### 1.1 Vue2 响应式原理

> 核心：`Object.defineProperty` 劫持 getter/setter

```js
function defineReactive(obj, key, val) {
  // 递归处理嵌套对象
  if (typeof val === 'object') {
    new Observer(val);
  }
  
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get() {
      console.log(`读取 ${key}: ${val}`);
      return val;
    },
    set(newVal) {
      console.log(`设置 ${key}: ${newVal}`);
      if (newVal !== val) {
        val = newVal;
        // 通知更新
      }
    }
  });
}
```

**缺点**：
- 无法监听新增/删除属性（需用 `Vue.set` / `Vue.delete`）
- 数组下标无法响应（重写了 7 个数组方法）
- 深度监听性能开销大

### 1.2 Vue3 响应式原理

> 核心：`Proxy` 代理

```js
const reactive = (target) => {
  return new Proxy(target, {
    get(target, key, receiver) {
      console.log(`读取 ${key}`);
      // 依赖收集
      track(target, key);
      return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
      console.log(`设置 ${key}: ${value}`);
      // 触发更新
      trigger(target, key);
      return Reflect.set(target, key, value, receiver);
    },
    deleteProperty(target, key) {
      trigger(target, key);
      return Reflect.deleteProperty(target, key);
    }
  });
};
```

**优点**：
- 直接监听对象变化（新增/删除自动响应）
- 数组下标可以响应
- 性能更好

---

## 二、ref vs reactive

### 2.1 对比

| 特性 | ref | reactive |
|------|-----|----------|
| 适用类型 | 任意类型 | 对象/数组 |
| 访问方式 | `.value` | 直接访问 |
| 响应式 | ✅ | ✅ |
| 解构 | 失去响应式（需 toRef） | 失去响应式（需 toRefs） |

```js
// ref - 适合原始类型
const count = ref(0);
count.value++;

// reactive - 适合对象
const state = reactive({ name: 'Tom', age: 18 });
state.age = 20;
```

### 2.2 转换函数

```js
import { ref, reactive, toRefs, toRef } from 'vue';

// ref → reactive
const r = reactive(ref(0)); // 自动展开

// reactive → ref
const { name } = toRefs(state); // 解构保持响应式
const name = toRef(state, 'name'); // 单独转换
```

---

## 三、computed vs watch

### 3.1 computed

> 计算属性，缓存结果，依赖变化时重新计算

```js
const firstName = ref('John');
const lastName = ref('Doe');

const fullName = computed(() => {
  return firstName.value + ' ' + lastName.value;
});
```

**特点**：
- 默认懒执行
- 缓存结果，依赖不变不重算
- 必须有返回值

### 3.2 watch

> 监听数据变化，执行副作用

```js
// 监听单个
watch(count, (newVal, oldVal) => {
  console.log(`count 变化: ${oldVal} → ${newVal}`);
});

// 监听多个
watch([count, name], ([newCount, newName]) => {
  console.log('变化了');
});

// 深度监听
watch(obj, (newVal) => {
  console.log('obj 变化了');
}, { deep: true });

// 立即执行
watch(source, cb, { immediate: true });
```

### 3.3 对比

| 特性 | computed | watch |
|------|----------|-------|
| 用途 | 计算衍生数据 | 执行副作用 |
| 缓存 | ✅ 依赖不变不重算 | ❌ 每次变化都执行 |
| 返回值 | 必须有 | 不需要 |
| 适用 | 模板中的复杂表达式 | 异步/复杂副作用 |

---

## 四、组件通信

### 4.1 通信方式

| 方式 | 适用场景 | 语法 |
|------|----------|------|
| props / emit | 父子通信 | `<Child :msg="msg" @change="handle" />` |
| `provide / inject` | 祖孙通信 | `provide('key', value)` / `inject('key')` |
| `$attrs` | 透传 props | `$attrs` / `v-bind="$attrs"` |
| `$refs` | 父访问子 | `ref="child"` → `$refs.child` |
| EventBus | 任意组件 | `new Vue()` / `mitt` |
| Vuex / Pinia | 全局状态 | `store.state` |

### 4.2 provide / inject

```js
// 父组件
provide('theme', reactive({ color: 'blue' }));

// 子组件
const theme = inject('theme');
```

---

## 五、指令

### 5.1 v-if vs v-show

| 特性 | v-if | v-show |
|------|------|--------|
| 原理 | 条件渲染（DOM 删除/创建） | CSS 显示/隐藏 |
| 开销 | 高（切换时销毁重建） | 低（始终渲染，切换成本低） |
| 适用 | 很少切换 | 频繁切换 |
| v-else | ✅ 支持 | ❌ 不支持 |

### 5.2 自定义指令

```js
// 全局指令
Vue.directive('focus', {
  inserted(el) {
    el.focus();
  }
});

// 组件内指令
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus();
      }
    }
  }
}
```

---

## 六、Vue2 vs Vue3 区别

| 特性 | Vue2 | Vue3 |
|------|------|------|
| 响应式 | Object.defineProperty | Proxy |
| API 风格 | Options API | Composition API (推荐) |
| 生命周期 | created/mounted... | setup() + onMounted() |
| 多根节点 | ❌ | ✅ |
| Teleport | ❌ | ✅ (`<Teleport>`) |
| Fragments | ❌ | ✅ |
| 性能 | - | 提升约 30-50% |

### 3.1 Composition API

```js
import { ref, computed, onMounted } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const doubled = computed(() => count.value * 2);
    
    const increment = () => count.value++;
    
    onMounted(() => {
      console.log('mounted');
    });
    
    return { count, doubled, increment };
  }
}
```

---

## 七、原理深入

### 7.1 nextTick 原理

> DOM 更新是异步的，`nextTick` 等待更新完成后执行回调

```js
// 源码简析
function nextTick(cb) {
  return new Promise(resolve => {
    queueMicrotask(() => {
      cb();
      resolve();
    });
  });
}
```

**使用场景**：
```js
this.msg = '更新了';
this.$nextTick(() => {
  // DOM 已更新
  console.log(this.$refs.input.value);
});
```

### 7.2 虚拟 DOM 与 patch

```
Template → Render Function → VNode Tree
                    ↓
              Patch (Diff)
                    ↓
               Real DOM
```

**patch 过程**：
1. sameVNode ? **更新属性** : **替换节点**
2. children diff：双端比较 → 快速匹配 → 遍历比较

---

## 八、常见追问

1. **Vue3 性能提升原因？**
   - Proxy 响应式（无需递归）
   - Block/静态提升
   - PatchFlags（标记动态节点）
   - Treeshaking 支持

2. **Vue 组件为什么 data 必须是函数？**
   - 组件可能被复用多次，如果是对象，所有实例共享同一份数据

3. **Vue 的单向数据流？**
   - props 只能从父到子流动
   - 子组件不能直接修改 props，需通过 emit 通知父组件

---

> 📌 下一章：[网络与浏览器](./04-network-browser.md)
