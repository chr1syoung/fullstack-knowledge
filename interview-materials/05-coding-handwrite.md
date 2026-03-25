# 手写代码题

> 面试频率：⭐⭐⭐⭐⭐ | 区分候选人的关键

---

## 一、防抖与节流

### 1.1 debounce（防抖）

> 连续触发只执行最后一次

```js
function debounce(fn, delay) {
  let timer = null;
  return function(...args) {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}

// 使用
const handleSearch = debounce((value) => {
  console.log('搜索:', value);
}, 300);
```

### 1.2 throttle（节流）

> 固定时间段内只执行一次

```js
function throttle(fn, interval) {
  let lastTime = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}

// 使用
const handleScroll = throttle(() => {
  console.log('滚动');
}, 200);
```

---

## 二、Promise 实现

### 2.1 Promise.all

```js
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises)) {
      return reject(new TypeError('参数必须是数组'));
    }
    
    const results = [];
    let completed = 0;
    
    if (promises.length === 0) return resolve(results);
    
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        val => {
          results[i] = val;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        },
        err => reject(err)
      );
    });
  });
}
```

### 2.2 Promise.race

```js
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach(p => {
      Promise.resolve(p).then(resolve, reject);
    });
  });
}
```

### 2.3 Promise.allSettled

```js
function promiseAllSettled(promises) {
  return new Promise(resolve => {
    const results = [];
    let completed = 0;
    
    if (promises.length === 0) return resolve(results);
    
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        val => {
          results[i] = { status: 'fulfilled', value: val };
        },
        err => {
          results[i] = { status: 'rejected', reason: err };
        }
      ).finally(() => {
        completed++;
        if (completed === promises.length) {
          resolve(results);
        }
      });
    });
  });
}
```

### 2.4 符合 Promise/A+ 规范的 Promise（简版）

```js
class MyPromise {
  constructor(executor) {
    this.status = 'pending';
    this.value = undefined;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];
    
    const resolve = (value) => {
      if (this.status !== 'pending') return;
      if (value instanceof MyPromise) {
        return value.then(resolve, reject);
      }
      this.status = 'fulfilled';
      this.value = value;
      this.onFulfilledCallbacks.forEach(cb => cb());
    };
    
    const reject = (reason) => {
      if (this.status !== 'pending') return;
      this.status = 'rejected';
      this.value = reason;
      this.onRejectedCallbacks.forEach(cb => cb());
    };
    
    try {
      executor(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }
  
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const wrap = (fn) => {
        try {
          const result = fn(this.value);
          resolve(result);
        } catch (e) {
          reject(e);
        }
      };
      
      if (this.status === 'fulfilled') {
        setTimeout(() => wrap(onFulfilled), 0);
      } else if (this.status === 'rejected') {
        setTimeout(() => wrap(onRejected), 0);
      } else {
        this.onFulfilledCallbacks.push(() => wrap(onFulfilled));
        this.onRejectedCallbacks.push(() => wrap(onRejected));
      }
    });
  }
  
  catch(onRejected) {
    return this.then(null, onRejected);
  }
  
  finally(cb) {
    return this.then(
      val => MyPromise.resolve(cb()).then(() => val),
      err => MyPromise.resolve(cb()).then(() => throw err)
    );
  }
}
```

---

## 三、原型链与继承

### 3.1 new 操作符

```js
function myNew(constructor, ...args) {
  // 1. 创建新对象
  const obj = {};
  
  // 2. 绑定原型
  obj.__proto__ = constructor.prototype;
  
  // 3. 绑定 this 并执行
  const result = constructor.apply(obj, args);
  
  // 4. 返回对象
  return result instanceof Object ? result : obj;
}

// 使用
function Person(name) {
  this.name = name;
}
const p = myNew(Person, 'Tom');
```

### 3.2 instanceof

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

### 3.3 继承（ES6）

```js
class Parent {
  constructor(name) {
    this.name = name;
  }
  sayHi() {
    console.log(`Hi, I'm ${this.name}`);
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
}
```

### 3.4 继承（ES5）

```js
function Parent(name) {
  this.name = name;
}
Parent.prototype.sayHi = function() {
  console.log(`Hi, I'm ${this.name}`);
};

function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}

// 原型链继承
Child.prototype = new Parent();
Child.prototype.constructor = Child;
```

---

## 四、函数柯里化

### 4.1 基础柯里化

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...args2) {
      return curried.apply(this, args.concat(args2));
    };
  };
}

// 使用
const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
curriedAdd(1)(2)(3); // 6
curriedAdd(1, 2)(3); // 6
```

---

## 五、对象操作

### 5.1 浅拷贝

```js
function shallowClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  
  const target = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      target[key] = obj[key];
    }
  }
  return target;
}
```

### 5.2 深拷贝

```js
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj);
  
  const target = Array.isArray(obj) ? [] : {};
  hash.set(obj, target);
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      target[key] = deepClone(obj[key], hash);
    }
  }
  return target;
}

// 简化版（无法处理循环引用）
function deepCloneSimple(obj) {
  return JSON.parse(JSON.stringify(obj));
}
```

### 5.3 Object.assign polyfill

```js
function assign(target, ...sources) {
  if (target === null || target === undefined) {
    throw new TypeError('Cannot convert undefined or null to object');
  }
  
  const to = Object(target);
  
  for (const source of sources) {
    if (source !== null && source !== undefined) {
      for (const key in source) {
        if (source.hasOwnProperty(key)) {
          to[key] = source[key];
        }
      }
    }
  }
  return to;
}
```

---

## 六、数组操作

### 6.1 数组去重

```js
// 方法1: Set
[...new Set(arr)];

// 方法2: filter + indexOf
arr.filter((item, i) => arr.indexOf(item) === i);

// 方法3: Map
function unique(arr) {
  const map = new Map();
  return arr.filter(item => !map.has(item) && map.set(item, true));
}
```

### 6.2 数组扁平化

```js
// 方法1: flat
[1, [2, [3]]].flat(Infinity);

// 方法2: 递归
function flatten(arr) {
  let result = [];
  for (const item of arr) {
    if (Array.isArray(item)) {
      result = result.concat(flatten(item));
    } else {
      result.push(item);
    }
  }
  return result;
}

// 方法3: reduce
function flattenReduce(arr) {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? flattenReduce(cur) : cur);
  }, []);
}
```

### 6.3 数组乱序（Fisher-Yates）

```js
function shuffle(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}
```

---

## 七、EventEmitter（发布订阅）

```js
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  off(event, callback) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(cb => cb !== callback);
  }
  
  emit(event, ...args) {
    if (!this.events[event]) return;
    this.events[event].forEach(cb => cb(...args));
  }
  
  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }
}

// 使用
const emitter = new EventEmitter();
emitter.on('click', (data) => console.log('clicked', data));
emitter.emit('click', { x: 1 });
```

---

## 八、call / apply / bind

### 8.1 call

```js
Function.prototype.myCall = function(context, ...args) {
  const ctx = context || window;
  const fn = Symbol('fn');
  ctx[fn] = this;
  const result = ctx[fn](...args);
  delete ctx[fn];
  return result;
};
```

### 8.2 apply

```js
Function.prototype.myApply = function(context, args = []) {
  const ctx = context || window;
  const fn = Symbol('fn');
  ctx[fn] = this;
  const result = ctx[fn](...args);
  delete ctx[fn];
  return result;
};
```

### 8.3 bind

```js
Function.prototype.myBind = function(context, ...args) {
  const fn = this;
  return function(...args2) {
    return fn.apply(context, [...args, ...args2]);
  };
};
```

---

## 九、AJAX / fetch 封装

### 9.1 AJAX

```js
function ajax(options) {
  return new Promise((resolve, reject) => {
    const { url, method = 'GET', data = null, headers = {} } = options;
    
    const xhr = new XMLHttpRequest();
    xhr.open(method, url, true);
    
    Object.entries(headers).forEach(([k, v]) => {
      xhr.setRequestHeader(k, v);
    });
    
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(xhr.statusText));
      }
    };
    
    xhr.onerror = () => reject(new Error('Network Error'));
    
    xhr.send(data);
  });
}
```

### 9.2 手写 fetch 简化版

```js
function myFetch(url, options = {}) {
  const { method = 'GET', headers = {}, body = null } = options;
  
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open(method, url, true);
    
    Object.entries(headers).forEach(([k, v]) => {
      xhr.setRequestHeader(k, v);
    });
    
    xhr.onload = () => {
      const response = {
        ok: xhr.status >= 200 && xhr.status < 300,
        status: xhr.status,
        statusText: xhr.statusText,
        text: () => Promise.resolve(xhr.responseText),
        json: () => Promise.resolve(JSON.parse(xhr.responseText))
      };
      if (response.ok) {
        resolve(response);
      } else {
        reject(response);
      }
    };
    
    xhr.onerror = () => reject(new Error('Fetch error'));
    
    xhr.send(body);
  });
}
```

---

## 十、LRU 缓存

```js
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1;
    
    // 移到末尾（最近使用）
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // 删除最旧的（第一个）
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}
```

---

> 📌 下一章：[工程化与性能](./06-engineering-performance.md)
