# JavaScript 基础

> 面试频率：⭐⭐⭐⭐⭐

---

## 一、原型链

### 1.1 `prototype` vs `__proto__`

```js
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function() {
  console.log(`Hi, I'm ${this.name}`);
};

const p = new Person('Tom');

// prototype：函数属性，指向原型对象
// __proto__：对象属性，指向构造函数的 prototype
console.log(p.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
```

### 1.2 原型链图解

```
p (实例)
  ↓ __proto__
Person.prototype
  ↓ __proto__
Object.prototype
  ↓ __proto__
null
```

### 1.3 `instanceof` 原理

```js
function myInstanceof(obj, Constructor) {
  let proto = obj.__proto__;
  while (proto) {
    if (proto === Constructor.prototype) return true;
    proto = proto.__proto__;
  }
  return false;
}
```

---

## 二、闭包

### 2.1 什么是闭包

> **定义**：函数 + 引用到的外部变量 = 闭包

```js
function outer() {
  let count = 0;
  return function() {
    count++;
    console.log(count);
  };
}
const fn = outer(); // fn 形成闭包，引用了 count
fn(); // 1
fn(); // 2
```

### 2.2 闭包的应用场景

| 场景 | 例子 |
|------|------|
| 函数工厂 | `createAdd(n)` 返回加法函数 |
| 私有变量 | 模块化、科里化 |
| 定时器循环 | `for` 循环 + `setTimeout` (经典问题) |
| 防抖/节流 | 内部维护计时器变量 |

### 2.3 经典面试题：for 循环闭包

```js
// ❌ 错误：输出 5 5 5 5 5
for (var i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 100);
}

// ✅ 正确1：使用 let
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 100);
}

// ✅ 正确2：使用闭包
for (var i = 0; i < 5; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i);
}
```

### 2.4 内存泄漏

闭包可能导致内存泄漏的场景：
- 循环引用 DOM 元素
- 全局变量引用大对象
- 未清理的定时器/事件监听

---

## 三、异步编程

### 3.1 Promise 状态

```
pending (进行中)
  ↓ resolve()
fulfilled (已成功)
  ↓ reject()
rejected (已失败)
```

**状态一旦改变，不可逆**

```js
const p = new Promise((resolve, reject) => {
  resolve('ok'); // 只能触发一次
  reject('error');
  resolve('again');
});
p.then(console.log, console.error); // 'ok'
```

### 3.2 事件循环 (Event Loop)

```
┌─────────────────────────┐
│         Call Stack      │ ← 执行同步代码
└─────────────────────────┘
            ↓
┌─────────────────────────┐
│   MicroTask Queue       │ ← Promise.then, queueMicrotask
│   (先执行)               │
└─────────────────────────┘
            ↓
┌─────────────────────────┐
│   MacroTask Queue       │ ← setTimeout, setInterval, I/O
│   (后执行)               │
└─────────────────────────┘
```

**经典题目：输出顺序**

```js
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve().then(() => console.log(3));

console.log(4);

// 输出: 1 → 4 → 3 → 2
```

### 3.3 `async/await` 原理

`async` 函数返回值是 Promise，`await` 是 Promise 的语法糖：

```js
async function fn() {
  const res = await fetch('/api');
  return res.json();
}
// 等价于
function fn() {
  return fetch('/api').then(res => res.json());
}
```

### 3.4 经典面试题：Promise 链

```js
Promise.resolve(1)
  .then(x => x + 1)
  .then(x => {
    throw new Error('error');
  })
  .catch(e => 1) // 捕获后返回 1
  .then(x => console.log(x)) // 打印 1
  .then(x => console.log(x + 1)); // undefined (上一步没返回值)
```

---

## 四、this 绑定

### 4.1 绑定规则优先级

```
new 绑定 > 显式绑定(call/apply/bind) > 隐式绑定(obj.foo()) > 默认绑定(独立函数)
```

### 4.2 题目讲解

```js
var value = 100;
var obj = {
  value: 1,
  getValue: function() {
    console.log(this.value);
  }
};

obj.getValue(); // 1 (隐式绑定)

const fn = obj.getValue;
fn(); // 100 (默认绑定，指向 window/global)

const newFn = obj.getValue.bind({ value: 999 });
newFn(); // 999 (显式绑定)
```

### 4.3 箭头函数 this

> 箭头函数没有自己的 `this`，指向**定义时的外层作用域**

```js
function Timer() {
  this.time = 0;
  
  setInterval(() => {
    this.time++; // this 指向 Timer 实例
  }, 1000);
  
  setInterval(function() {
    this.time++; // this 指向 window
  }, 1000);
}
```

---

## 五、类型转换

### 5.1 面试必刷题

```js
// [] == ![] 
// ![] = false
// [] == false
// Number([]) = 0, Number(false) = 0
// true

// {} + [] 
// {} 被当作代码块，+[] = Number([]) = 0
// 0

// 1 + '1'
// '11'

// [] + {}
// Number([]) = 0, Number({}) = NaN
// '0[object Object]'

// '1' - 1
// Number('1') - 1 = 0
```

### 5.2 转换规则表

| 值 | Number() | String() | Boolean() |
|----|----------|----------|------------|
| `0` | 0 | '0' | false |
| `'0'` | 0 | '0' | true |
| `[]` | 0 | '' | true |
| `{}` | NaN | '[object Object]' | true |
| `null` | 0 | 'null' | false |
| `undefined` | NaN | 'undefined' | false |

---

## 六、ES6+

### 6.1 `let` vs `const` vs `var`

| 特性 | var | let/const |
|------|-----|-----------|
| 作用域 | 函数作用域 | 块级作用域 |
| 提升 | ✅ 有提升，值为 undefined | ✅ 有提升（暂时性死区） |
| 重复声明 | ✅ 允许 | ❌ 不允许 |
| 全局绑定 | 挂载 window | 不挂载 |

### 6.2 暂时性死区 (TDZ)

```js
console.log(a); // ReferenceError
let a = 1;

// 解释：let 在声明前不能访问
```

### 6.3 解构赋值

```js
// 数组解构
const [a, b, ...rest] = [1, 2, 3, 4]; // a=1, b=2, rest=[3,4]

// 对象解构
const { name, age = 18 } = { name: 'Tom' }; // name='Tom', age=18

// 函数参数解构
function fn({ x, y = 10 }) { return x + y; }
fn({ x: 5 }); // 15
```

### 6.4 Set / Map

```js
// Set：去重
const arr = [1, 2, 2, 3];
[...new Set(arr)]; // [1, 2, 3]

// Map：键可以是任意类型
const m = new Map();
m.set({}, 'value');
m.get({}); // undefined (对象是引用)
```

### 6.5 Proxy / Reflect

```js
const obj = { name: 'Tom' };
const proxy = new Proxy(obj, {
  get(target, key) {
    console.log('get:', key);
    return Reflect.get(target, key);
  },
  set(target, key, value) {
    console.log('set:', key, value);
    return Reflect.set(target, key, value);
  }
});
```

---

## 七、常见追问

1. **new 操作符做了什么？**
   - 创建新对象
   - 绑定原型
   - 绑定 this
   - 返回对象

2. **深拷贝怎么实现？**
   - 递归处理 + 解决循环引用 (Map 记录已拷贝对象)
   - `JSON.parse(JSON.stringify())` 缺点：无法拷贝函数、undefined、Symbol

3. **垃圾回收机制**
   - 标记清除、引用计数
   - 常见内存泄漏：全局变量、闭包、定时器、未清理的 DOM 引用

---

> 📌 下一章：[React 面试题](./02-react.md)
