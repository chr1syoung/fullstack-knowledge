# 二、JavaScript 基础与进阶

---

## 2.1 基础语法

### 2.1.1 变量声明

#### 知识点详解

JavaScript 中有三种变量声明方式：`var`、`let`、`const`。

**`var` 的特点：**
```javascript
// 1. 函数作用域（不是块作用域）
function test() {
    if (true) {
        var x = 1;
    }
    console.log(x);  // 1，可以访问
}

// 2. 变量提升
console.log(a);  // undefined（不会报错）
var a = 1;
// 等价于：
// var a;
// console.log(a);
// a = 1;

// 3. 允许重复声明
var a = 1;
var a = 2;  // 不报错

// 4. 全局变量会成为 window 的属性
var globalVar = 1;
console.log(window.globalVar);  // 1
```

**`let` 的特点：**
```javascript
// 1. 块级作用域
function test() {
    if (true) {
        let x = 1;
    }
    console.log(x);  // ReferenceError: x is not defined
}

// 2. 暂时性死区（Temporal Dead Zone）
console.log(a);  // ReferenceError: Cannot access 'a' before initialization
let a = 1;

// 3. 不允许重复声明
let a = 1;
let a = 2;  // SyntaxError: Identifier 'a' has already been declared

// 4. 全局变量不会成为 window 的属性
let globalLet = 1;
console.log(window.globalLet);  // undefined
```

**`const` 的特点：**
```javascript
// 1. 声明时必须初始化
const a;  // SyntaxError: Missing initializer in const declaration
const a = 1;  // 正确

// 2. 块级作用域（同 let）

// 3. 不能重新赋值
const a = 1;
a = 2;  // TypeError: Assignment to constant variable

// 4. 但是对象和数组的内容可以修改
const obj = { name: 'John' };
obj.name = 'Jane';  // 允许，修改属性
obj.age = 20;       // 允许，添加属性
obj = {};           // 错误，重新赋值

const arr = [1, 2, 3];
arr.push(4);        // 允许
arr[0] = 0;         // 允许
arr = [];           // 错误

// 5. 冻结对象（完全不可变）
const frozenObj = Object.freeze({ name: 'John' });
frozenObj.name = 'Jane';  // 严格模式下报错，非严格模式静默失败
```

**对比总结：**

| 特性 | var | let | const |
|------|-----|-----|-------|
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 是（初始化为 undefined） | 否（TDZ） | 否（TDZ） |
| 重复声明 | 允许 | 不允许 | 不允许 |
| 重新赋值 | 允许 | 允许 | 不允许 |
| 全局对象属性 | 是 | 否 | 否 |
| 初始化 | 可选 | 可选 | 必须 |

#### 真实面试题

**题目1：下面的代码输出什么？为什么？**

```javascript
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
```

**满分答案：**

**输出：**
```
3
3
3
```

**原因分析：**

1. `var` 是函数作用域，整个循环中只有一个 `i` 变量
2. `setTimeout` 是异步的，回调函数会在循环结束后执行
3. 当回调执行时，`i` 的值已经是 3（循环结束条件）
4. 三个回调函数都引用同一个 `i`，所以都输出 3

**解决方案：**

```javascript
// 方案1：使用 let（推荐）
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
// 输出：0, 1, 2
// 原因：let 创建块级作用域，每次循环都创建新的 i

// 方案2：IIFE 创建闭包
for (var i = 0; i < 3; i++) {
    ((j) => {
        setTimeout(() => console.log(j), 100);
    })(i);
}

// 方案3：使用 setTimeout 的第三个参数
for (var i = 0; i < 3; i++) {
    setTimeout((j) => console.log(j), 100, i);
}
```

---

**题目2：什么是暂时性死区（TDZ）？**

**满分答案：**

**定义：**

暂时性死区（Temporal Dead Zone, TDZ）是指从代码块开始到变量声明语句被执行之间的区域，在这期间访问 `let` 或 `const` 声明的变量会抛出 `ReferenceError`。

**示例：**

```javascript
{
    // TDZ 开始
    console.log(myVar);  // ReferenceError: Cannot access 'myVar' before initialization
    
    // TDZ 结束
    let myVar = 'hello';  // 变量声明并初始化
    console.log(myVar);   // 'hello'
}
```

**与 var 的区别：**

```javascript
// var 有变量提升，初始化为 undefined
console.log(a);  // undefined
var a = 1;

// let/const 有 TDZ，声明前不可访问
console.log(b);  // ReferenceError
let b = 1;
```

**typeof 也受影响：**

```javascript
console.log(typeof x);  // ReferenceError (TDZ)
let x = 1;

console.log(typeof y);  // 'undefined'（y 未声明，但 typeof 安全）
```

**实际意义：**

TDZ 的存在让代码更加规范：
- 强制开发者在使用变量前先声明
- 避免因变量提升带来的难以发现的 bug
- 使代码行为更符合直觉

---

### 2.1.2 数据类型

#### 知识点详解

JavaScript 有 8 种数据类型，分为原始类型和引用类型。

**原始类型（Primitive Types）：**

```javascript
// 1. Number（数字）
let num = 42;
let float = 3.14;
let negative = -10;
let infinity = Infinity;
let notANumber = NaN;
console.log(typeof num);  // 'number'

// Number 特殊值
console.log(Number.MAX_SAFE_INTEGER);  // 9007199254740991
console.log(Number.MIN_SAFE_INTEGER);  // -9007199254740991
console.log(Number.isNaN(NaN));        // true
console.log(Number.isFinite(Infinity)); // false

// 2. String（字符串）
let str = 'hello';
let template = `The value is ${num}`;
console.log(typeof str);  // 'string'

// 3. Boolean（布尔值）
let isTrue = true;
let isFalse = false;
console.log(typeof isTrue);  // 'boolean'

// 4. Undefined（未定义）
let undef;
console.log(undef);         // undefined
console.log(typeof undef);  // 'undefined'

// 5. Null（空值）
let empty = null;
console.log(empty);         // null
console.log(typeof empty);  // 'object'（历史遗留 bug）

// 6. Symbol（符号，ES6 新增）
let sym = Symbol('description');
let sym2 = Symbol('description');
console.log(sym === sym2);  // false，每个 Symbol 都是唯一的

// 作为对象属性键
const obj = {
    [sym]: 'value'
};
console.log(obj[sym]);  // 'value'

// 7. BigInt（大整数，ES2020 新增）
let bigNum = 9007199254740993n;  // 后缀 n
let bigNum2 = BigInt(9007199254740993);
console.log(typeof bigNum);  // 'bigint'
```

**引用类型（Reference Types）：**

```javascript
// Object（对象）
const obj = {
    name: 'John',
    age: 30
};

// Array（数组）
const arr = [1, 2, 3];

// Function（函数）
function fn() {}
const arrowFn = () => {};

// Date、RegExp、Map、Set 等
const date = new Date();
const map = new Map();
const set = new Set();
```

**类型检测：**

```javascript
// 1. typeof 操作符
typeof 42              // 'number'
typeof 'hello'         // 'string'
typeof true            // 'boolean'
typeof undefined       // 'undefined'
typeof null            // 'object'（bug）
typeof {}              // 'object'
typeof []              // 'object'
typeof function(){}    // 'function'
typeof Symbol()        // 'symbol'
typeof 10n             // 'bigint'

// 2. instanceof 检测原型链
[] instanceof Array    // true
[] instanceof Object   // true

// 3. Object.prototype.toString.call()（最准确）
Object.prototype.toString.call(42)        // '[object Number]'
Object.prototype.toString.call('hello')   // '[object String]'
Object.prototype.toString.call(null)      // '[object Null]'
Object.prototype.toString.call(undefined) // '[object Undefined]'
Object.prototype.toString.call([])        // '[object Array]'
Object.prototype.toString.call({})        // '[object Object]'
Object.prototype.toString.call(new Date)  // '[object Date]'

// 4. Array.isArray()
Array.isArray([])      // true
Array.isArray({})      // false

// 5. Number.isNaN() vs isNaN()
Number.isNaN(NaN)      // true
Number.isNaN('hello')  // false
isNaN('hello')         // true（会先转换）
```

**类型转换：**

```javascript
// 转字符串
String(42)           // '42'
(42).toString()      // '42'
42 + ''              // '42'

// 转数字
Number('42')         // 42
parseInt('42px')     // 42
parseFloat('3.14')   // 3.14
+'42'                // 42
'42' - 0             // 42

// 转布尔值
Boolean(1)           // true
Boolean(0)           // false
!!1                  // true
!!0                  // false

// 假值（Falsy Values）
Boolean(false)       // false
Boolean(0)           // false
Boolean(-0)          // false
Boolean(0n)          // false
Boolean('')          // false
Boolean(null)        // false
Boolean(undefined)   // false
Boolean(NaN)         // false
```

#### 真实面试题

**题目1：`typeof null` 为什么返回 `'object'`？**

**满分答案：**

**历史原因：**

这是 JavaScript 的一个历史遗留 bug，自语言诞生之初就存在。

在 JavaScript 的早期实现中，值以 32 位存储：
- 低位 0-2 位表示类型标签（type tag）
- `000` 表示对象
- `1` 表示整数
- `010` 表示双精度浮点数
- `100` 表示字符串
- `110` 表示布尔值

`null` 在内部表示为机器码空指针（所有位都是 0），因此它的类型标签也是 `000`，导致 `typeof null` 返回 `'object'`。

**为什么没有修复：**

1. **兼容性**：现有代码可能依赖这个行为
2. **提案被拒绝**：曾有提案修复此 bug，但被 ECMAScript 委员会拒绝

**正确的 null 检测方式：**

```javascript
// 方法1：严格相等
value === null

// 方法2：同时检测 null 和 undefined
value == null

// 方法3：Object.prototype.toString
Object.prototype.toString.call(value) === '[object Null]'
```

---

**题目2：`==` 和 `===` 有什么区别？什么时候用哪个？**

**满分答案：**

**区别：**

- `===`（严格相等）：比较值和类型，不进行类型转换
- `==`（宽松相等）：比较前会进行类型转换

**类型转换规则（`==`）：**

```javascript
// 1. null 和 undefined 相等
null == undefined  // true

// 2. 数字和字符串：字符串转数字
'42' == 42  // true

// 3. 布尔值和其他：布尔值转数字
true == 1   // true
false == 0  // true

// 4. 对象和原始值：对象转原始值
[1] == 1    // true，[1].valueOf().toString() = '1'，然后转数字

// 5. NaN 不等于任何值
NaN == NaN  // false
```

**诡异案例：**

```javascript
[] == ![]    // true
// 解析：
// ![] = false
// [] == false
// [] 转原始值 ''，false 转数字 0
// '' == 0 → 0 == 0 → true

[] == false  // true
!![] == true // true

'' == 0      // true
'0' == 0     // true
'' == '0'    // false

null == 0    // false
null < 0     // false
null <= 0    // true
```

**使用建议：**

| 场景 | 推荐使用 |
|------|----------|
| 普通比较 | `===` |
| 检测 null 或 undefined | `== null`（同时检测两者） |
| 比较数字和字符串形式的数字 | 用 `===`，显式转换类型 |

```javascript
// 推荐
if (value === null || value === undefined) { }
// 或简写
if (value == null) { }

// 不推荐
if (value == 0) { }      // 可能传入 '0' 或 false
if (value == '') { }     // 可能传入 0 或 false

// 推荐
if (value === 0) { }
if (value === '') { }
```

---

### 2.1.3 闭包

#### 知识点详解

**定义：**

闭包是指有权访问另一个函数作用域中变量的函数。简单来说，闭包让一个函数能够"记住"它被创建时的环境。

**基本示例：**

```javascript
function outer() {
    const outerVar = 'I am from outer';
    
    function inner() {
        console.log(outerVar);  // 可以访问 outerVar
    }
    
    return inner;
}

const myClosure = outer();
myClosure();  // 'I am from outer'
```

**闭包的用途：**

1. **数据私有化**
```javascript
function createCounter() {
    let count = 0;  // 私有变量
    
    return {
        increment() {
            count++;
            return count;
        },
        decrement() {
            count--;
            return count;
        },
        getCount() {
            return count;
        }
    };
}

const counter = createCounter();
console.log(counter.increment());  // 1
console.log(counter.increment());  // 2
console.log(counter.getCount());   // 2
// 无法直接访问 count
```

2. **函数工厂**
```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

3. **模块模式**
```javascript
const module = (function() {
    let privateVar = 0;
    
    function privateMethod() {
        console.log('private');
    }
    
    return {
        publicVar: 'public',
        publicMethod() {
            privateMethod();
            privateVar++;
            return privateVar;
        }
    };
})();

console.log(module.publicVar);      // 'public'
console.log(module.publicMethod()); // 'private' then 1
// 无法访问 privateVar 和 privateMethod
```

4. **保持状态（如防抖、节流）**
```javascript
function debounce(fn, delay) {
    let timer = null;
    
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            fn.apply(this, args);
        }, delay);
    };
}

const debouncedFn = debounce(() => console.log('执行'), 300);
```

**闭包的注意事项：**

1. **内存占用**
```javascript
function createClosure() {
    const largeData = new Array(10000).fill('data');
    
    return function() {
        console.log(largeData.length);  // largeData 不会被释放
    };
}

// 解决：不需要时手动释放
function createClosure() {
    const largeData = new Array(10000).fill('data');
    const length = largeData.length;  // 只保存需要的值
    
    return function() {
        console.log(length);
    };
}
```

2. **循环中的闭包问题**
```javascript
// 问题
for (var i = 1; i <= 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, i * 1000);
}
// 输出：6, 6, 6, 6, 6

// 解决方案1：let
for (let i = 1; i <= 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, i * 1000);
}
// 输出：1, 2, 3, 4, 5

// 解决方案2：IIFE
for (var i = 1; i <= 5; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(j);
        }, j * 1000);
    })(i);
}
```

#### 真实面试题

**题目1：什么是闭包？闭包的使用场景有哪些？**

**满分答案：**

**定义：**

闭包是指函数能够访问其词法作用域，即使该函数在其词法作用域之外执行。简单来说，闭包是函数和其声明时所在的作用域的组合。

**核心原理：**

1. JavaScript 采用词法作用域（静态作用域）
2. 函数创建时会保存对外部变量的引用
3. 即使外部函数执行完毕，内部函数仍能访问这些变量

**使用场景：**

```javascript
// 1. 数据私有化
function createWallet(initialBalance) {
    let balance = initialBalance;
    
    return {
        deposit(amount) {
            balance += amount;
            return balance;
        },
        withdraw(amount) {
            if (amount > balance) throw new Error('余额不足');
            balance -= amount;
            return balance;
        },
        getBalance() {
            return balance;
        }
    };
}

// 2. 柯里化
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        return function(...moreArgs) {
            return curried.apply(this, args.concat(moreArgs));
        };
    };
}

// 3. 模块化
const counter = (function() {
    let count = 0;
    return {
        increment: () => ++count,
        decrement: () => --count,
        get: () => count
    };
})();

// 4. 回调和事件处理
function setupButton(buttonId, message) {
    const button = document.getElementById(buttonId);
    button.addEventListener('click', function() {
        alert(message);  // 闭包捕获 message
    });
}

// 5. 缓存/Memoization
function memoize(fn) {
    const cache = {};
    return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) return cache[key];
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}
```

---

**题目2：闭包会导致什么问题？如何解决？**

**满分答案：**

**问题1：内存泄漏**

闭包会阻止垃圾回收，因为被引用的变量无法释放。

```javascript
function problematic() {
    const largeData = new Array(1000000).fill('x');
    
    return function() {
        console.log('Hello');
        // largeData 被闭包引用，无法释放
    };
}

// 解决：只保留需要的数据
function fixed() {
    const largeData = new Array(1000000).fill('x');
    const needed = largeData.length;  // 只保留需要的值
    
    return function() {
        console.log(needed);
    };
}
```

**问题2：循环变量问题**

```javascript
// 问题代码
var buttons = document.querySelectorAll('button');
for (var i = 0; i < buttons.length; i++) {
    buttons[i].addEventListener('click', function() {
        console.log('Button ' + i + ' clicked');
    });
}
// 所有按钮点击都输出相同的 i

// 解决方案
for (let i = 0; i < buttons.length; i++) {
    buttons[i].addEventListener('click', function() {
        console.log('Button ' + i + ' clicked');
    });
}

// 或使用 IIFE
for (var i = 0; i < buttons.length; i++) {
    (function(index) {
        buttons[index].addEventListener('click', function() {
            console.log('Button ' + index + ' clicked');
        });
    })(i);
}
```

**问题3：意外的共享状态**

```javascript
// 问题代码
function createHandlers() {
    const handlers = [];
    
    for (var i = 0; i < 3; i++) {
        handlers.push(function() {
            console.log(i);
        });
    }
    
    return handlers;
}

const handlers = createHandlers();
handlers[0]();  // 3
handlers[1]();  // 3
handlers[2]();  // 3

// 解决：let 或 IIFE
function createHandlers() {
    const handlers = [];
    
    for (let i = 0; i < 3; i++) {
        handlers.push(function() {
            console.log(i);
        });
    }
    
    return handlers;
}
```

**最佳实践：**

1. 及时解除不需要的闭包引用
2. 使用 `let` 替代 `var` 避免循环问题
3. 只在必要时使用闭包
4. 注意闭包捕获的变量大小

#### 真实面试题

**题目：什么是闭包？它在实际开发中有什么应用场景？**

**满分答案：**

**定义：** 闭包是函数与其词法环境的组合，使函数能访问其外部作用域的变量，即使外部函数已执行完毕。

**核心应用场景：**

1. **数据私有化**（模块模式）
```javascript
function createCounter() {
    let count = 0;  // 私有变量，外部无法直接访问
    return {
        increment: () => ++count,
        decrement: () => --count,
        getCount: () => count
    };
}
const counter = createCounter();
counter.increment(); // 1
```

2. **函数工厂**
```javascript
function multiply(x) {
    return (y) => x * y;  // 记住 x
}
const double = multiply(2);
const triple = multiply(3);
double(5); // 10
```

3. **防抖/节流**（缓存 timer 变量）

4. **React Hooks**（useState 内部用闭包保存状态）

**注意事项：** 闭包会持有外部变量引用，可能导致内存泄漏，用完要及时解除引用。

---

### 2.1.4 原型与原型链

#### 知识点详解

**原型（Prototype）：**

每个 JavaScript 对象都有一个原型对象，对象从原型继承方法和属性。

```javascript
// 构造函数
function Person(name) {
    this.name = name;
}

// 在原型上添加方法
Person.prototype.sayHello = function() {
    console.log(`Hello, I'm ${this.name}`);
};

// 创建实例
const person1 = new Person('Alice');
const person2 = new Person('Bob');

person1.sayHello();  // "Hello, I'm Alice"
person2.sayHello();  // "Hello, I'm Bob"

// 原型关系
console.log(person1.__proto__ === Person.prototype);  // true
console.log(Person.prototype.constructor === Person); // true
console.log(person1 instanceof Person);               // true
```

**原型链：**

当访问对象的属性或方法时，JavaScript 会沿着原型链向上查找，直到找到或到达链的末端（null）。

```javascript
// 原型链示例
function Animal(name) {
    this.name = name;
}
Animal.prototype.eat = function() {
    console.log(`${this.name} is eating`);
};

function Dog(name, breed) {
    Animal.call(this, name);
    this.breed = breed;
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() {
    console.log(`${this.name} is barking`);
};

const dog = new Dog('Max', 'Golden Retriever');

// 原型链
console.log(dog.__proto__ === Dog.prototype);           // true
console.log(Dog.prototype.__proto__ === Animal.prototype); // true
console.log(Animal.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__);                // null

// 方法查找
dog.bark();     // 在 Dog.prototype 找到
dog.eat();      // 在 Animal.prototype 找到
dog.toString(); // 在 Object.prototype 找到
```

**原型链图示：**

```
dog
  └── __proto__ === Dog.prototype
        ├── bark()
        └── __proto__ === Animal.prototype
              ├── eat()
              └── __proto__ === Object.prototype
                    ├── toString()
                    ├── hasOwnProperty()
                    └── __proto__ === null
```

**属性查找规则：**

```javascript
const obj = { a: 1 };
obj.__proto__ = { b: 2 };
Object.prototype.c = 3;

console.log(obj.a);  // 1（自身属性）
console.log(obj.b);  // 2（原型属性）
console.log(obj.c);  // 3（原型链顶端）
console.log(obj.d);  // undefined（不存在）

// 检查属性
console.log('a' in obj);                    // true（包括继承）
console.log(obj.hasOwnProperty('a'));       // true（仅自身）
console.log(obj.hasOwnProperty('b'));       // false
console.log(Object.hasOwn(obj, 'a'));       // true（推荐方式）
```

**Class 语法（ES6）：**

```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    eat() {
        console.log(`${this.name} is eating`);
    }
    
    static isAnimal(obj) {
        return obj instanceof Animal;
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);
        this.breed = breed;
    }
    
    bark() {
        console.log(`${this.name} is barking`);
    }
}

const dog = new Dog('Max', 'Golden Retriever');
dog.eat();   // 继承的方法
dog.bark();  // 自身方法
Dog.isAnimal(dog);  // true（静态方法）
```

#### 真实面试题

**题目1：解释 JavaScript 的原型链是如何工作的？**

**满分答案：**

**基本概念：**

1. 每个对象都有一个内部属性 `[[Prototype]]`（通过 `__proto__` 或 `Object.getPrototypeOf()` 访问）
2. 当访问对象的属性时，如果对象本身没有该属性，JavaScript 会沿着原型链向上查找
3. 原型链的终点是 `null`

**查找过程：**

```javascript
const obj = { a: 1 };

// 访问 obj.a
// 1. 检查 obj 自身是否有 a → 有，返回 1

// 访问 obj.toString
// 1. 检查 obj 自身 → 没有
// 2. 检查 obj.__proto__（Object.prototype）→ 有，返回函数

// 访问 obj.xyz
// 1. 检查 obj 自身 → 没有
// 2. 检查 Object.prototype → 没有
// 3. 检查 Object.prototype.__proto__ → null
// 4. 返回 undefined
```

**原型链的形成：**

```javascript
function Foo() {}
const foo = new Foo();

// 原型链
foo.__proto__ === Foo.prototype
Foo.prototype.__proto__ === Object.prototype
Object.prototype.__proto__ === null

// 构造函数的原型链
Foo.__proto__ === Function.prototype
Function.prototype.__proto__ === Object.prototype
```

**图示：**

```
          null
           ↑
    Object.prototype
     ↑           ↑
Foo.prototype  Function.prototype
     ↑               ↑
    foo            Foo/Function/Object
```

**属性遮蔽（Shadowing）：**

```javascript
const parent = { name: 'parent' };
const child = Object.create(parent);
child.name = 'child';  // 遮蔽父类的 name

console.log(child.name);         // 'child'（自身属性）
console.log(parent.name);        // 'parent'
console.log(child.hasOwnProperty('name'));  // true
```

---

**题目2：如何实现继承？请说出几种方式及其优缺点。**

**满分答案：**

**方式1：原型链继承**

```javascript
function Animal(name) {
    this.name = name;
    this.colors = ['black', 'white'];
}
Animal.prototype.eat = function() {
    console.log('eating');
};

function Dog() {}
Dog.prototype = new Animal('dog');

const dog1 = new Dog();
const dog2 = new Dog();

dog1.colors.push('brown');
console.log(dog2.colors);  // ['black', 'white', 'brown']
```

**缺点：** 引用类型属性被所有实例共享

**方式2：借用构造函数**

```javascript
function Animal(name) {
    this.name = name;
    this.colors = ['black', 'white'];
}
Animal.prototype.eat = function() {
    console.log('eating');
};

function Dog(name) {
    Animal.call(this, name);
}

const dog1 = new Dog('dog1');
const dog2 = new Dog('dog2');

dog1.colors.push('brown');
console.log(dog2.colors);  // ['black', 'white']
```

**缺点：** 无法继承原型上的方法

**方式3：组合继承（原型链 + 借用构造函数）**

```javascript
function Animal(name) {
    this.name = name;
    this.colors = ['black', 'white'];
}
Animal.prototype.eat = function() {
    console.log('eating');
};

function Dog(name, breed) {
    Animal.call(this, name);  // 第二次调用 Animal
    this.breed = breed;
}
Dog.prototype = new Animal();  // 第一次调用 Animal
Dog.prototype.constructor = Dog;
```

**缺点：** 父构造函数被调用两次

**方式4：寄生组合式继承（最优）**

```javascript
function Animal(name) {
    this.name = name;
    this.colors = ['black', 'white'];
}
Animal.prototype.eat = function() {
    console.log('eating');
};

function Dog(name, breed) {
    Animal.call(this, name);
    this.breed = breed;
}

// 关键：创建一个以 Animal.prototype 为原型的对象
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() {
    console.log('barking');
};
```

**方式5：ES6 Class 继承**

```javascript
class Animal {
    constructor(name) {
        this.name = name;
        this.colors = ['black', 'white'];
    }
    
    eat() {
        console.log('eating');
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name);
        this.breed = breed;
    }
    
    bark() {
        console.log('barking');
    }
}
```

**总结：**

| 方式 | 引用类型共享问题 | 方法继承 | 调用父构造函数次数 |
|------|-----------------|----------|-------------------|
| 原型链继承 | 有问题 | ✓ | 1 次 |
| 借用构造函数 | 无问题 | ✗ | 1 次 |
| 组合继承 | 无问题 | ✓ | 2 次 |
| 寄生组合式 | 无问题 | ✓ | 1 次 |
| ES6 Class | 无问题 | ✓ | 1 次 |

#### 真实面试题

**题目：描述一下 JavaScript 的原型链机制。**

**满分答案：**

**核心概念：**
- 每个对象都有 `__proto__` 指向其构造函数的 `prototype`
- 访问属性时，先找自身，找不到就沿 `__proto__` 向上查找，直到 `null`

```javascript
function Dog(name) { this.name = name; }
Dog.prototype.bark = function() { console.log('woof'); };

const dog = new Dog('Rex');
// 查找链：dog → Dog.prototype → Object.prototype → null

dog.bark();          // 在 Dog.prototype 找到
dog.hasOwnProperty; // 在 Object.prototype 找到
dog.xxx;            // undefined（到 null 还没找到）
```

**原型链终点：** `Object.prototype.__proto__ === null`

**instanceof 原理：** 沿原型链查找，看右侧构造函数的 `prototype` 是否在链上

**ES6 Class 本质：** 语法糖，底层仍是原型链

---

## 2.2 数据结构与算法

### 2.2.1 数组方法详解

#### 知识点详解

**数组的创建：**

```javascript
// 字面量
const arr1 = [1, 2, 3];

// 构造函数
const arr2 = new Array(3);        // [empty × 3]
const arr3 = new Array(1, 2, 3);  // [1, 2, 3]

// Array.from()
const arr4 = Array.from('hello');  // ['h', 'e', 'l', 'l', 'o']
const arr5 = Array.from({ length: 5 }, (_, i) => i);  // [0, 1, 2, 3, 4]

// Array.of()
const arr6 = Array.of(1, 2, 3);   // [1, 2, 3]
const arr7 = Array.of(3);         // [3]（不是 [empty × 3]）

// fill()
const arr8 = new Array(5).fill(0);  // [0, 0, 0, 0, 0]
```

**修改数组的方法（原数组改变）：**

```javascript
const arr = [1, 2, 3, 4, 5];

// push/pop（尾部操作）
arr.push(6, 7);  // 返回新长度 7，arr 变为 [1,2,3,4,5,6,7]
arr.pop();       // 返回 7，arr 变为 [1,2,3,4,5,6]

// unshift/shift（头部操作）
arr.unshift(0);  // 返回新长度 7，arr 变为 [0,1,2,3,4,5,6]
arr.shift();     // 返回 0，arr 变为 [1,2,3,4,5,6]

// splice（任意位置增删改）
arr.splice(2, 1);        // 删除：从索引2开始删除1个，返回 [3]
arr.splice(2, 0, 'a');   // 插入：在索引2插入'a'
arr.splice(2, 1, 'b');   // 替换：删除索引2的1个，插入'b'

// sort（排序）
const nums = [3, 1, 4, 1, 5, 9];
nums.sort();  // [1, 1, 3, 4, 5, 9]（默认字符串排序）
nums.sort((a, b) => a - b);  // 数字升序
nums.sort((a, b) => b - a);  // 数字降序

// reverse（反转）
arr.reverse();  // 反转数组

// copyWithin（内部复制）
[1, 2, 3, 4, 5].copyWithin(0, 3);  // [4, 5, 3, 4, 5]

// fill（填充）
[1, 2, 3].fill(0, 1, 2);  // [1, 0, 3]，从索引1到2填充0
```

**不修改数组的方法（返回新数组或值）：**

```javascript
const arr = [1, 2, 3, 4, 5];

// slice（截取，返回新数组）
arr.slice(1, 3);     // [2, 3]
arr.slice(-2);       // [4, 5]
arr.slice();         // 浅拷贝

// concat（连接）
arr.concat([6, 7]);  // [1,2,3,4,5,6,7]
[...arr, 6, 7];      // 展开运算符

// join（转字符串）
arr.join('-');       // '1-2-3-4-5'

// indexOf/lastIndexOf
arr.indexOf(3);      // 2
arr.indexOf(6);      // -1

// includes（ES7）
arr.includes(3);     // true
[NaN].includes(NaN); // true（indexOf 不能正确检测 NaN）

// at（ES2022，支持负索引）
arr.at(0);           // 1
arr.at(-1);          // 5
```

**遍历方法：**

```javascript
const arr = [1, 2, 3, 4, 5];

// forEach（遍历，无返回值）
arr.forEach((item, index, array) => {
    console.log(item, index);
});

// map（映射，返回新数组）
const doubled = arr.map(x => x * 2);  // [2, 4, 6, 8, 10]

// filter（过滤，返回满足条件的元素）
const evens = arr.filter(x => x % 2 === 0);  // [2, 4]

// find/findIndex（查找第一个满足条件的元素）
const found = arr.find(x => x > 3);        // 4
const index = arr.findIndex(x => x > 3);   // 3

// findLast/findLastIndex（ES2023，从后查找）
const last = arr.findLast(x => x > 3);     // 5

// every/some（检测所有/任一）
const allPositive = arr.every(x => x > 0);  // true
const hasEven = arr.some(x => x % 2 === 0); // true

// reduce/reduceRight（累积计算）
const sum = arr.reduce((acc, cur) => acc + cur, 0);  // 15
const max = arr.reduce((a, b) => Math.max(a, b));    // 5
const obj = arr.reduce((acc, cur) => {
    acc[cur] = cur * 2;
    return acc;
}, {});  // {1: 2, 2: 4, 3: 6, 4: 8, 5: 10}

// flat/flatMap（扁平化）
[[1, 2], [3, 4]].flat();           // [1, 2, 3, 4]
[1, 2, 3].flatMap(x => [x, x*2]);  // [1, 2, 2, 4, 3, 6]
```

**数组去重：**

```javascript
const arr = [1, 2, 2, 3, 3, 3];

// Set
const unique = [...new Set(arr)];  // [1, 2, 3]

// filter
const unique = arr.filter((item, index) => arr.indexOf(item) === index);

// reduce
const unique = arr.reduce((acc, cur) => {
    if (!acc.includes(cur)) acc.push(cur);
    return acc;
}, []);
```

**数组扁平化：**

```javascript
const arr = [1, [2, [3, [4]]]];

// flat
arr.flat(Infinity);  // [1, 2, 3, 4]

// 递归
function flatten(arr) {
    return arr.reduce((acc, cur) => {
        return acc.concat(Array.isArray(cur) ? flatten(cur) : cur);
    }, []);
}

// 迭代
function flatten(arr) {
    while (arr.some(item => Array.isArray(item))) {
        arr = [].concat(...arr);
    }
    return arr;
}
```

#### 真实面试题

**题目1：实现数组扁平化方法 `flatten`**

**满分答案：**

```javascript
// 方法1：递归（DFS）
function flatten(arr) {
    const result = [];
    
    arr.forEach(item => {
        if (Array.isArray(item)) {
            result.push(...flatten(item));
        } else {
            result.push(item);
        }
    });
    
    return result;
}

// 方法2：reduce + 递归
function flatten(arr) {
    return arr.reduce((acc, cur) => {
        return acc.concat(Array.isArray(cur) ? flatten(cur) : cur);
    }, []);
}

// 方法3：指定深度版本
function flatten(arr, depth = 1) {
    if (depth === 0) return arr;
    
    return arr.reduce((acc, cur) => {
        return acc.concat(
            Array.isArray(cur) ? flatten(cur, depth - 1) : cur
        );
    }, []);
}

// 方法4：迭代（BFS）
function flatten(arr) {
    const result = [];
    const stack = [...arr];
    
    while (stack.length) {
        const item = stack.pop();
        if (Array.isArray(item)) {
            stack.push(...item);
        } else {
            result.unshift(item);
        }
    }
    
    return result;
}

// 方法5：Generator
function* flatten(arr) {
    for (const item of arr) {
        if (Array.isArray(item)) {
            yield* flatten(item);
        } else {
            yield item;
        }
    }
}

[...flatten([1, [2, [3, [4]]]])];  // [1, 2, 3, 4]
```

---

**题目2：实现 `Array.prototype.reduce`**

**满分答案：**

```javascript
Array.prototype.myReduce = function(callback, initialValue) {
    // 检查回调是否为函数
    if (typeof callback !== 'function') {
        throw new TypeError(callback + ' is not a function');
    }
    
    const arr = this;
    const length = arr.length;
    
    // 空数组且没有初始值时报错
    if (length === 0 && initialValue === undefined) {
        throw new TypeError('Reduce of empty array with no initial value');
    }
    
    let acc = initialValue;
    let startIndex = 0;
    
    // 如果没有初始值，使用第一个元素作为初始值
    if (acc === undefined) {
        // 跳过空槽
        while (startIndex < length && !(startIndex in arr)) {
            startIndex++;
        }
        if (startIndex >= length) {
            throw new TypeError('Reduce of empty array with no initial value');
        }
        acc = arr[startIndex];
        startIndex++;
    }
    
    // 遍历执行回调
    for (let i = startIndex; i < length; i++) {
        if (i in arr) {  // 跳过空槽
            acc = callback(acc, arr[i], i, arr);
        }
    }
    
    return acc;
};

// 测试
const arr = [1, 2, 3, 4];
console.log(arr.myReduce((a, b) => a + b, 0));  // 10
console.log(arr.myReduce((a, b) => a + b));     // 10
```

---

## 2.3 异步编程

### 2.3.1 Promise

#### 知识点详解

**Promise 基础：**

```javascript
// 创建 Promise
const promise = new Promise((resolve, reject) => {
    // 异步操作
    setTimeout(() => {
        const success = true;
        if (success) {
            resolve('成功的数据');
        } else {
            reject(new Error('失败的原因'));
        }
    }, 1000);
});

// 使用 Promise
promise
    .then(data => {
        console.log(data);
        return '处理后的数据';
    })
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.error(error);
    })
    .finally(() => {
        console.log('无论成功失败都执行');
    });
```

**Promise 状态：**

- `pending`：初始状态，既未成功也未失败
- `fulfilled`：操作成功完成
- `rejected`：操作失败

**状态一旦改变就不可逆：**

```javascript
const p = new Promise((resolve, reject) => {
    resolve('成功');
    reject('失败');  // 无效，状态已确定
});
```

**Promise 静态方法：**

```javascript
// Promise.resolve / Promise.reject
Promise.resolve('立即成功').then(console.log);  // '立即成功'
Promise.reject('立即失败').catch(console.error);

// Promise.all（全部成功才成功）
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.resolve(3);

Promise.all([p1, p2, p3])
    .then(results => console.log(results));  // [1, 2, 3]

// 任一失败就失败
Promise.all([p1, Promise.reject('error'), p3])
    .catch(err => console.log(err));  // 'error'

// Promise.allSettled（等待所有完成，无论成功失败）
Promise.allSettled([
    Promise.resolve(1),
    Promise.reject('error'),
    Promise.resolve(3)
]).then(results => {
    console.log(results);
    // [
    //   { status: 'fulfilled', value: 1 },
    //   { status: 'rejected', reason: 'error' },
    //   { status: 'fulfilled', value: 3 }
    // ]
});

// Promise.race（返回最先完成的结果）
Promise.race([
    new Promise(resolve => setTimeout(() => resolve('慢'), 200)),
    new Promise(resolve => setTimeout(() => resolve('快'), 100))
]).then(result => console.log(result));  // '快'

// Promise.any（任一成功就成功，全部失败才失败）
Promise.any([
    Promise.reject('error1'),
    Promise.resolve('success'),
    Promise.reject('error2')
]).then(result => console.log(result));  // 'success'

Promise.any([
    Promise.reject('error1'),
    Promise.reject('error2')
]).catch(err => {
    console.log(err);  // AggregateError: All promises were rejected
});
```

**Promise 链式调用：**

```javascript
fetch('/api/user/1')
    .then(response => response.json())
    .then(user => fetch(`/api/posts/${user.id}`))
    .then(response => response.json())
    .then(posts => console.log(posts))
    .catch(error => console.error('请求失败:', error));

// 返回值会自动包装成 Promise
Promise.resolve(1)
    .then(x => x + 1)         // 返回 2，自动包装成 Promise.resolve(2)
    .then(x => Promise.resolve(x + 1))  // 返回 Promise
    .then(x => { throw new Error('错误'); })  // 抛出错误
    .catch(err => console.log(err.message))   // 捕获错误
    .then(() => '继续执行')   // catch 后可以继续 then
    .then(console.log);       // '继续执行'
```

#### 真实面试题

**题目1：实现 `Promise.all`**

**满分答案：**

```javascript
Promise.myAll = function(promises) {
    return new Promise((resolve, reject) => {
        // 可迭代对象转数组
        const iterable = Array.from(promises);
        const length = iterable.length;
        
        // 空数组直接解决
        if (length === 0) {
            resolve([]);
            return;
        }
        
        const results = new Array(length);
        let completed = 0;
        
        iterable.forEach((promise, index) => {
            // 确保是 Promise
            Promise.resolve(promise)
                .then(value => {
                    results[index] = value;
                    completed++;
                    
                    if (completed === length) {
                        resolve(results);
                    }
                })
                .catch(reject);  // 任一失败立即拒绝
        });
    });
};

// 测试
Promise.myAll([
    Promise.resolve(1),
    Promise.resolve(2),
    Promise.resolve(3)
]).then(console.log);  // [1, 2, 3]
```

---

**题目2：实现 Promise 的并发限制**

**满分答案：**

```javascript
class PromisePool {
    constructor(maxConcurrent) {
        this.maxConcurrent = maxConcurrent;
        this.running = 0;
        this.queue = [];
    }
    
    async run(task) {
        if (this.running >= this.maxConcurrent) {
            await new Promise(resolve => this.queue.push(resolve));
        }
        
        this.running++;
        
        try {
            return await task();
        } finally {
            this.running--;
            if (this.queue.length) {
                this.queue.shift()();
            }
        }
    }
}

// 使用示例
const pool = new PromisePool(3);  // 最多 3 个并发

const tasks = Array.from({ length: 10 }, (_, i) => 
    () => new Promise(resolve => {
        console.log(`任务 ${i} 开始`);
        setTimeout(() => {
            console.log(`任务 ${i} 完成`);
            resolve(i);
        }, 1000);
    })
);

// 方式2：更简洁的实现
async function limitConcurrency(tasks, maxConcurrent) {
    const results = [];
    const executing = new Set();
    
    for (const task of tasks) {
        const promise = Promise.resolve().then(() => task());
        results.push(promise);
        executing.add(promise);
        
        const cleanup = () => executing.delete(promise);
        promise.then(cleanup, cleanup);
        
        if (executing.size >= maxConcurrent) {
            await Promise.race(executing);
        }
    }
    
    return Promise.all(results);
}

// 测试
const tasks = Array.from({ length: 10 }, (_, i) => 
    () => new Promise(resolve => {
        console.log(`任务 ${i} 开始`);
        setTimeout(() => {
            console.log(`任务 ${i} 完成`);
            resolve(i);
        }, Math.random() * 1000);
    })
);

limitConcurrency(tasks, 3).then(console.log);
```

---

### 2.3.2 async/await

#### 知识点详解

**基本语法：**

```javascript
async function fetchData() {
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('请求失败:', error);
        throw error;
    }
}
```

**async/await 与 Promise 的关系：**

```
async/await 是 Promise 的语法糖
  ↓
async 函数自动返回 Promise
  ↓
await 等待 Promise resolve/reject
  ↓
本质还是基于 Promise 的事件循环机制
```

```javascript
// async 函数自动返回 Promise
async function test() {
    return 123;
}
// 等价于
function test() {
    return Promise.resolve(123);
}

// await 等待 Promise
async function demo() {
    const result = await Promise.resolve('hello');
    console.log(result);  // 'hello'
    // 等价于：
    // Promise.resolve('hello').then(res => console.log(res));
}
```

**async/await 的优势：**

```javascript
// Promise 链式调用（容易回调地狱）
fetch('/api/user')
    .then(res => res.json())
    .then(user => fetch(`/api/posts/${user.id}`))
    .then(res => res.json())
    .then(posts => console.log(posts))
    .catch(err => console.error(err));

// async/await（同步写法，更易读）
async function getUserPosts() {
    try {
        const userRes = await fetch('/api/user');
        const user = await userRes.json();
        const postsRes = await fetch(`/api/posts/${user.id}`);
        const posts = await postsRes.json();
        console.log(posts);
    } catch (err) {
        console.error(err);
    }
}
```

**错误处理：**

```javascript
// 方式1：try-catch
async function withTryCatch() {
    try {
        const data = await fetchData();
        return data;
    } catch (error) {
        console.error('请求失败:', error);
        return null;
    }
}

// 方式2：Promise.catch
async function withCatch() {
    return await fetchData().catch(err => {
        console.error(err);
        return null;
    });
}

// 方式3：统一错误处理
async function withErrorCheck() {
    const [error, data] = await fetchData()
        .then(data => [null, data])
        .catch(error => [error, null]);
    
    if (error) {
        console.error(error);
        return null;
    }
    return data;
}
```

**并行执行优化：**

```javascript
// 串行（慢）
async function serial() {
    const a = await getA();  // 等 1s
    const b = await getB();  // 等 1s
    const c = await getC();  // 等 1s
    // 总计 3s
}

// 并行（快）
async function parallel() {
    const [a, b, c] = await Promise.all([getA(), getB(), getC()]);
    // 总计 1s
}

// Promise.all 的 async/await 写法
async function parallelAll() {
    const results = await Promise.all([
        fetch('/api/users').then(r => r.json()),
        fetch('/api/posts').then(r => r.json()),
        fetch('/api/comments').then(r => r.json())
    ]);
    return results;
}

// Promise.allSettled（全部执行，不短路）
async function allSettled() {
    const results = await Promise.allSettled([
        fetch('/api/1'),
        fetch('/api/2'),
        fetch('/api/3')
    ]);
    
    results.forEach((result, i) => {
        if (result.status === 'fulfilled') {
            console.log(`API ${i+1} 成功:`, result.value);
        } else {
            console.log(`API ${i+1} 失败:`, result.reason);
        }
    });
}
```

**面试常考点：**

```javascript
// Q: async 函数返回值是什么？
async function fn() { return 1; }
fn() instanceof Promise  // true

// Q: await 后面跟非 Promise 会怎样？
async function test() {
    const result = await 123;  // 自动转 Promise.resolve(123)
    console.log(result);  // 123
}

// Q: await 在 forEach 中不会并行等待？
async function wrong() {
    [1, 2, 3].forEach(async (n) => {
        await fetch(`/api/${n}`);  // forEach 不会等待 async 回调
    });
    console.log('立即执行');  // 不会等所有请求完成
}

// 正确做法：使用 for...of
async function correct() {
    for (const n of [1, 2, 3]) {
        await fetch(`/api/${n}`);  // 串行等待
    }
    console.log('所有请求完成');
}

// 或者用 map + Promise.all
async function best() {
    await Promise.all([1, 2, 3].map(n => fetch(`/api/${n}`)));
    console.log('所有请求完成');
}
```

#### 真实面试题

**题目：Promise 有哪几种状态？async/await 和它是什么关系？**

**满分答案：**

**Promise 三种状态：**
- `pending`：初始状态
- `fulfilled`：操作成功完成
- `rejected`：操作失败

**状态特点：**
- 状态一旦改变就不可逆
- 只能从 pending → fulfilled 或 pending → rejected

**async/await 与 Promise 的关系：**
1. `async` 函数自动返回 Promise
2. `await` 等待 Promise resolve/reject
3. `await` 本质是基于 Promise 的微任务机制
4. `async/await` 是 Promise 的语法糖，让异步代码更像同步写法

---


---

### 2.x.x 通过设置失效时间清除本地存储的数据 [重要]

#### 知识点详解

localStorage 本身不支持设置过期时间，需要自行封装或使用额外策略来实现过期机制。

**核心思路：** 存储时附带时间戳，读取时判断是否过期，过期则删除。

**1. 封装 localStorage 工具函数：**

```javascript
const storage = {
  // 设置带过期时间的值
  set(key, value, expireSeconds) {
    const data = {
      value,
      expire: expireSeconds ? Date.now() + expireSeconds * 1000 : null
    };
    localStorage.setItem(key, JSON.stringify(data));
  },

  // 获取值，过期自动删除
  get(key) {
    const raw = localStorage.getItem(key);
    if (!raw) return null;

    try {
      const { value, expire } = JSON.parse(raw);
      // 过期则删除
      if (expire && Date.now() > expire) {
        localStorage.removeItem(key);
        return null;
      }
      return value;
    } catch {
      return null;
    }
  },

  // 删除
  remove(key) {
    localStorage.removeItem(key);
  },

  // 清除所有（可选：只清过期）
  clear() {
    localStorage.clear();
  }
};

// 使用
storage.set('token', 'abc123', 3600); // 1小时过期
const token = storage.get('token');
```

**2. 封装 sessionStorage 版本：**

```javascript
const sessionStorageWithExpire = {
  set(key, value, expireSeconds) {
    const data = {
      value,
      expire: expireSeconds ? Date.now() + expireSeconds * 1000 : null
    };
    sessionStorage.setItem(key, JSON.stringify(data));
  },
  get(key) {
    const raw = sessionStorage.getItem(key);
    if (!raw) return null;
    try {
      const { value, expire } = JSON.parse(raw);
      if (expire && Date.now() > expire) {
        sessionStorage.removeItem(key);
        return null;
      }
      return value;
    } catch {
      return null;
    }
  }
};
```

**3. 扩展原生 localStorage 类的写法：**

```javascript
// 给 localStorage 添加过期方法
Storage.prototype.setExpire = function(key, value, seconds) {
  this.setItem(key, JSON.stringify({
    v: value,
    e: Date.now() + seconds * 1000
  }));
};

Storage.prototype.getExpire = function(key) {
  const raw = this.getItem(key);
  if (!raw) return null;
  const { v, e } = JSON.parse(raw);
  if (Date.now() > e) {
    this.removeItem(key);
    return null;
  }
  return v;
};

// 使用
localStorage.setExpire('token', 'xyz', 7200);
const token = localStorage.getExpire('token');
```

**4. 过期数据的自动清理（懒检查 / 主动遍历）：**

```javascript
// 每次使用前检查所有 key（可封装到 get 时）
function autoCleanup(storage) {
  const keys = Object.keys(storage);
  keys.forEach(key => {
    try {
      const { e } = JSON.parse(storage.getItem(key));
      if (e && Date.now() > e) {
        storage.removeItem(key);
        console.log(`已清理过期数据: ${key}`);
      }
    } catch { /* 忽略非过期格式的数据 */ }
  });
}

// 可定时清理
setInterval(() => autoCleanup(localStorage), 60000); // 每分钟检查一次
```

**5. localStorage vs sessionStorage vs Cookie 对比：**

| 特性 | localStorage | sessionStorage | Cookie |
|------|-------------|----------------|--------|
| 大小 | ~5-10MB | ~5-10MB | ~4KB |
| 过期 | 永不过期（需手动清理） | 关闭标签页清除 | 可设置 expires/max-age |
| 随请求发送 | ❌ | ❌ | ✅（自动带在 HTTP header） |
| 适用场景 | 持久存储偏好设置 | 临时会话数据 | 认证 token（配合服务端） |

**常见面试题：**

> **Q：localStorage 存满了怎么办？**
> - 捕获 `QuotaExceededError` 异常
- 清理旧数据或使用 indexedDB 作为替代

#### 面试加分项
- 了解 IndexedDB 的使用场景（大容量结构化数据）
- 了解 Cookie、localStorage、sessionStorage、IndexedDB 的完整对比

---

### 2.x.x 如何判断 DOM 元素是否在可视区域 [重要]

#### 知识点详解

判断元素是否在可视区域内有多种方法，从原生 API 到自定义计算：

**1. getBoundingClientRect()（最常用）：**

```javascript
function isInViewport(el) {
  const rect = el.getBoundingClientRect();
  return (
    rect.top >= 0 &&
    rect.left >= 0 &&
    rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
    rect.right <= (window.innerWidth || document.documentElement.clientWidth)
  );
}

// 使用
const box = document.querySelector('.target');
console.log(isInViewport(box)); // true / false
```

**2. 判断元素是否部分可见（更实用）：**

```javascript
function isPartiallyInViewport(el) {
  const rect = el.getBoundingClientRect();
  const windowHeight = window.innerHeight || document.documentElement.clientHeight;
  const windowWidth = window.innerWidth || document.documentElement.clientWidth;

  const vertInView = (rect.top <= windowHeight) && ((rect.top + rect.height) >= 0);
  const horInView = (rect.left <= windowWidth) && ((rect.left + rect.width) >= 0);

  return vertInView && horInView;
}
```

**3. IntersectionObserver API（推荐，性能最优）：**

```javascript
// 现代浏览器推荐方式，不阻塞主线程
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('元素进入可视区域', entry.target);
      // 执行加载、动画等操作
      // observer.unobserve(entry.target); // 只触发一次
    }
  });
}, {
  threshold: 0.1,      // 10% 可见时触发
  rootMargin: '0px',   // 可视区域偏移
});

// 观察多个元素
document.querySelectorAll('.lazy-image').forEach(img => {
  observer.observe(img);
});

// 断开观察
// observer.disconnect();
```

**4. scroll 事件监听（传统方式，不推荐频繁使用）：**

```javascript
function checkVisible(el) {
  const rect = el.getBoundingClientRect();
  return (
    rect.top < window.innerHeight &&
    rect.bottom > 0
  );
}

window.addEventListener('scroll', () => {
  document.querySelectorAll('.item').forEach(item => {
    if (checkVisible(item)) {
      item.classList.add('visible');
    }
  });
}, { passive: true }); // passive: true 提升滚动性能
```

**5. Element.scrollIntoView()（滚动到可视区域）：**

```javascript
// 滚动到可视区域
el.scrollIntoView();              // 默认顶部对齐
el.scrollIntoView({ behavior: 'smooth' }); // 平滑滚动
el.scrollIntoView({ block: 'center' });    // 元素在视口中央
```

**懒加载图片的完整实现：**

```javascript
// 图片懒加载
const imageObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      const src = img.dataset.src;
      img.src = src;
      img.removeAttribute('data-src');
      observer.unobserve(img);
    }
  });
}, { rootMargin: '50px' }); // 提前 50px 开始加载

document.querySelectorAll('img[data-src]').forEach(img => {
  imageObserver.observe(img);
});
```

**常见面试题：**

> **Q：getBoundingClientRect 和 IntersectionObserver 哪个更好？**
> - `getBoundingClientRect`：简单直接，但需要手动计算，在 scroll 事件中频繁调用会影响性能
> - `IntersectionObserver`：异步执行，不阻塞主线程，专门为"元素是否可见"场景优化，现代浏览器优先使用

#### 面试加分项
- 了解 `rootMargin` 的作用（提前/延后触发边界）
- 了解 IntersectionObserver 在广告曝光统计、无限滚动等场景的应用


---

### 2.x.x 什么是领域模型（Domain Model）[必会]

#### 知识点详解

领域模型（Domain Model）是一种对业务问题域的概念抽象，通过面向对象的方式描述现实世界中的实体及其关系。

**核心概念：**

```
现实世界          抽象为代码
─────────────────────────────────────
实体（Entity）   →  class / 对象
属性（Property） →  属性/字段
行为（Behavior） →  方法/函数
关系（Relation） →  引用/关联
```

**举例：电商系统的领域模型**

```javascript
// 领域模型：商品
class Product {
  constructor({ id, name, price, stock }) {
    this.id = id;
    this.name = name;
    this.price = price;
    this.stock = stock;
  }

  // 领域行为：检查库存
  isAvailable(quantity = 1) {
    return this.stock >= quantity;
  }

  // 领域行为：扣减库存
  deductStock(quantity) {
    if (!this.isAvailable(quantity)) {
      throw new Error(`库存不足，当前库存: ${this.stock}`);
    }
    this.stock -= quantity;
  }
}

// 领域模型：购物车项
class CartItem {
  constructor(product, quantity) {
    if (!product.isAvailable(quantity)) {
      throw new Error('商品库存不足');
    }
    this.product = product;
    this.quantity = quantity;
  }

  getSubtotal() {
    return this.product.price * this.quantity;
  }
}

// 领域模型：购物车（聚合根）
class Cart {
  constructor() {
    this.items = [];
  }

  addItem(product, quantity) {
    const existing = this.items.find(i => i.product.id === product.id);
    if (existing) {
      existing.quantity += quantity;
    } else {
      this.items.push(new CartItem(product, quantity));
    }
  }

  getTotal() {
    return this.items.reduce((sum, item) => sum + item.getSubtotal(), 0);
  }

  checkout() {
    // 领域行为：结账时校验库存并扣减
    this.items.forEach(item => {
      item.product.deductStock(item.quantity);
    });
    this.items = [];
  }
}

// 使用
const cart = new Cart();
const product = new Product({ id: 1, name: 'iPhone', price: 6999, stock: 10 });
cart.addItem(product, 2);
console.log('总价:', cart.getTotal()); // 13998
```

**为什么需要领域模型？**

| 没有领域模型 | 有领域模型 |
|-------------|-----------|
| 业务逻辑散落在各层（Controller/Service/Util） | 业务逻辑内聚在领域对象中 |
| 修改业务逻辑需要改多处代码 | 改一处即可 |
| 难以发现业务规则 | 规则显式表达在模型中 |
| 代码难以复用和测试 | 模型可独立测试 |

**领域模型 vs 贫血模型 vs 充血模型：**

| 模型类型 | 特点 | 适用场景 |
|---------|------|---------|
| **贫血模型** | 只有 getter/setter，无业务逻辑 | 简单 CRUD，数据传输对象 |
| **充血模型** | 属性 + 业务逻辑方法 | 复杂业务逻辑 |
| **DDD 领域模型** | 聚合根、值对象、领域事件 | 复杂业务，团队协作 |

**DDD（Domain-Driven Design）关键概念：**

```javascript
// 值对象（不可变，整个替换）
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }
  add(other) {
    return new Money(this.amount + other.amount, this.currency);
  }
}

// 实体（有唯一ID，可变）
class Order extends Entity {
  // ...
}

// 领域事件
class OrderPlaced extends DomainEvent {
  constructor(orderId, total) {
    this.name = 'OrderPlaced';
    this.orderId = orderId;
    this.total = total;
    this.occurredOn = new Date();
  }
}
```

**常见面试题：**

> **Q：领域模型和数据库模型有什么区别？**
> - 数据库模型（ER Model）：面向存储，关注表结构、字段类型、索引
> - 领域模型（Domain Model）：面向业务，关注实体、行为、规则
> - 领域模型最终会持久化到数据库，但两者关注的维度不同

> **Q：什么时候用领域模型？**
> 业务逻辑复杂时（如电商、金融、ERP 等），代码需要清晰表达业务规则时推荐使用。

#### 面试加分项
- 了解 DDD（Domain-Driven Design，领域驱动设计）核心概念
- 了解 CQRS（Command Query Responsibility Segregation，命令查询职责分离）
- 了解聚合根（Aggregate Root）、限界上下文（Bounded Context）


---

### 2.x.x 在 window 上挂载变量的风险 [重要]

#### 知识点详解

在 window 对象上直接挂载变量（全局变量）是前端开发中常见但不推荐的做法，存在多种风险。

**常见写法（不推荐）：**

```javascript
// 直接挂载
window.userInfo = { name: 'Tom' };
window.API_BASE = 'https://api.example.com';
window.utils = { formatDate, debounce, throttle };

// 全局变量（隐式挂载）
userInfo = { name: 'Tom' }; // 未声明，等价于 window.userInfo
```

**风险一：命名冲突**

```javascript
// 库A
window.utils = { debounce: () => {}, throttle: () => {} };

// 库B
window.utils = { deepClone: () => {}, randomId: () => {} };

// 库B 会覆盖库A的 utils，后者功能丢失！
console.log(window.utils.debounce); // undefined
```

**风险二：内存泄漏**

```javascript
// 常见泄漏场景：
// 1. 闭包引用
function createLeak() {
  const largeData = new Array(1000000);
  window.handler = () => largeData; // 闭包中的大对象被 window 持有
}

// 2. 事件监听未清理
window.scrollHandler = () => handleScroll();
window.addEventListener('scroll', window.scrollHandler);
// 组件卸载时忘记 removeEventListener，导致监听器无法回收
window.removeEventListener('scroll', window.scrollHandler); // 被遗漏

// 3. 定时器未清理
window.timerId = setInterval(() => fetchData(), 1000);
// 页面关闭前没有 clearInterval
```

**风险三：变量污染（第三方 SDK 冲突）**

```javascript
// 很多第三方库会占用 window 上的特定名称
// 如：jQuery → window.$

// 如果你的代码也用了 $ 做变量
window.$ = document.querySelectorAll; // 会导致 jQuery 失效
```

**风险四：安全风险**

```javascript
// 在 window 上暴露敏感数据
window.userToken = 'abc123';      // ❌ 可被任意第三方脚本读取
window.adminMode = true;          // ❌ 可被任意脚本修改

// 如果页面引入了恶意广告/统计脚本，它们可以直接访问这些数据
```

**风险五：SSR / Node.js 兼容性问题**

```javascript
// 在 React SSR 环境中
// window is not defined ❌ 报错
// 因为 Node.js 没有 window 对象

// 如果代码中直接访问 window
const width = window.innerWidth;

// 在 SSR 时会报错
// 解决方案：添加判断
if (typeof window !== 'undefined') {
  // 只在浏览器环境执行
  console.log(window.innerWidth);
}
```

**推荐做法：**

```javascript
// 1. 使用 IIFE 或 ES Module 封装
const Utils = (function() {
  function debounce(fn, delay) { /* ... */ }
  return { debounce };
})();

// 2. 使用命名空间对象（单一全局变量）
const MyApp = {
  utils: { debounce, throttle },
  store: createStore(),
  api: { baseUrl: '...', fetch }
};

// 3. 使用 Symbol 避免冲突
const _privateData = Symbol('private');
class MyClass {
  constructor() {
    this[_privateData] = {};
  }
}

// 4. IIFE + 导出
(function(global) {
  const Utils = { /* ... */ };
  global.Utils = Utils; // 只暴露一个全局变量
})(window);

// 5. React/Vue 生态中
// React: 使用 Context + useContext
// Vue: 使用 Vuex/Pinia / provide-inject
// Angular: 使用 Service 单例（providedIn: 'root'）
```

**SPA 框架的全局状态管理：**

```javascript
// React: Context
const AppContext = createContext();
<AppContext.Provider value={{ user, theme }}>
  <App />
</AppContext.Provider>

// Vue: Pinia
// stores/user.js
export const useUserStore = defineStore('user', {
  state: () => ({ name: '', token: '' }),
  actions: { login() { /* ... */ } }
});

// Redux
const store = createStore(rootReducer);
<Provider store={store}><App /></Provider>
```

**常见面试题：**

> **Q：全局变量和 window 变量的区别？**
> - `var` 声明的全局变量会自动成为 window 的属性
> - `let/const` 声明的全局变量不会成为 window 的属性
> - 未声明的隐式全局变量（`a = 1`）会成为 window 属性

> **Q：如何避免全局变量污染？**
> 1. 始终使用 `use strict` 或 ES Module
> 2. 使用 IIFE 包裹模块代码
> 3. 统一用命名空间对象管理模块
> 4. 使用 Webpack/Rollup 等打包工具自动处理作用域

#### 面试加分项
- 了解 `Object.freeze()` 冻结暴露的对象防止外部修改
- 了解 CSP（Content Security Policy）限制外部脚本访问
- 了解 Shadow DOM 隔离样式和变量

---

## 2.4 大对象深度对比

#### 知识点详解

**深拷贝 vs 深对比：**

- 深拷贝：创建一个对象的完整副本
- 深对比：判断两个对象是否"相等"（结构完全相同）

**实现思路：**

```javascript
// 基础版深对比
function deepCompare(obj1, obj2) {
    // 1. 引用相等直接返回
    if (obj1 === obj2) return true;
    
    // 2. 类型检查
    if (typeof obj1 !== 'object' || typeof obj2 !== 'object' || 
        obj1 === null || obj2 === null) {
        return obj1 === obj2;
    }
    
    // 3. 获取键数组
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);
    
    // 4. 键数量不同，直接返回 false
    if (keys1.length !== keys2.length) return false;
    
    // 5. 递归比较每个属性
    for (const key of keys1) {
        if (!keys2.includes(key)) return false;
        if (!deepCompare(obj1[key], obj2[key])) return false;
    }
    
    return true;
}
```

**处理边界情况：**

```javascript
function deepCompare(obj1, obj2, visited = new WeakMap()) {
    // 引用相等
    if (obj1 === obj2) return true;
    
    // 处理循环引用
    if (visited.has(obj1)) return visited.get(obj1) === obj2;
    
    // null 检查
    if (obj1 === null || obj2 === null) return obj1 === obj2;
    
    // 原始类型
    if (typeof obj1 !== 'object' || typeof obj2 !== 'object') {
        return obj1 === obj2;
    }
    
    // 日期对象
    if (obj1 instanceof Date && obj2 instanceof Date) {
        return obj1.getTime() === obj2.getTime();
    }
    
    // 正则对象
    if (obj1 instanceof RegExp && obj2 instanceof RegExp) {
        return obj1.toString() === obj2.toString();
    }
    
    // 数组
    if (Array.isArray(obj1) !== Array.isArray(obj2)) return false;
    
    // 记录访问
    visited.set(obj1, obj2);
    
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);
    
    if (keys1.length !== keys2.length) return false;
    
    for (const key of keys1) {
        if (!deepCompare(obj1[key], obj2[key], visited)) return false;
    }
    
    return true;
}
```

**性能优化 - 快速比较：**

```javascript
// 使用 JSON 快速比较（仅适用于可序列化的对象）
function fastDeepCompare(obj1, obj2) {
    return JSON.stringify(obj1) === JSON.stringify(obj2);
}
```

#### 真实面试题

**题目：如何实现一个高效的大对象深度对比函数？**

**满分答案：**

1. **处理基本类型**：null、undefined、string、number、boolean
2. **处理循环引用**：使用 WeakMap 记录已访问对象
3. **处理特殊对象**：Date、RegExp、Function
4. **处理数组**：数组和普通对象分开处理
5. **性能优化**：
   - 引用相等直接返回 true
   - 键数量不同直接返回 false
   - 使用 WeakMap 避免内存泄漏

**实际应用建议：**
- 业务代码推荐使用 lodash 的 `_.isEqual`
- React 中使用 `fast-deep-equal` 包
- 超大对象考虑用 hash 或结构共享优化

---

## 2.5 DocumentFragment API

---

## 2.6 浏览器事件循环（Event Loop）

#### 知识点详解

**什么是事件循环？**

事件循环是浏览器的执行机制，负责协调 JavaScript 单线程执行顺序。

```
┌─────────────────────┐
│    Call Stack       │  ← 执行代码
│    (调用栈)         │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│    Web APIs         │  ← setTimeout, DOM, fetch 等
│    (浏览器提供)      │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│    Task Queue        │  ← 宏任务队列
│    (宏任务队列)       │     setTimeout, setInterval, I/O
└─────────┬───────────┘
          │ 事件循环
┌─────────────────────┐
│    Microtask Queue  │  ← 微任务队列
│    (微任务队列)       │     Promise.then, MutationObserver
└─────────────────────┘
```

**宏任务 vs 微任务：**

```javascript
// 宏任务（Macrotask）
setTimeout(() => console.log('setTimeout'), 0);
setInterval(() => console.log('setInterval'), 0);
I/O 操作
UI 渲染
requestAnimationFrame
```

```javascript
// 微任务（Microtask）
Promise.then(() => console.log('Promise'));
MutationObserver
queueMicrotask()
async/await（await 后的代码）
```

**执行顺序：**

```javascript
console.log('1');  // 同步，立即执行

setTimeout(() => console.log('2'), 0);  // 宏任务

Promise.resolve().then(() => console.log('3'));  // 微任务

console.log('4');  // 同步

// 输出顺序：1 → 4 → 3 → 2
// 同步 → 微任务 → 宏任务
```

**经典面试题：**

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

new Promise((resolve) => {
    console.log('3');
    resolve();
}).then(() => console.log('4'));

console.log('5');

// 答案：1 → 3 → 5 → 4 → 2
```

**async/await 中的微任务：**

```javascript
async function async1() {
    console.log('1');  // 同步
    await async2();
    console.log('2');  // 微任务（await 后面的代码）
}

async function async2() {
    console.log('3');  // 同步
}

console.log('4');

setTimeout(() => console.log('5'), 0);

async1();

// 输出：4 → 1 → 3 → 2 → 5
```

**为什么微任务先于宏任务？**

1. 微任务在当前调用栈执行完毕后立即执行
2. 微任务全部执行完毕后，才会开始下一轮事件循环
3. 微任务用于处理需要尽快执行的任务

#### 真实面试题

**题目：解释浏览器的事件循环机制，宏任务和微任务有什么区别？**

**满分答案：**

**事件循环过程：**

1. 执行同步代码（属于当前任务）
2. 执行所有微任务（Promise 回调、queueMicrotask）
3. 执行一个宏任务（setTimeout、setInterval、I/O）
4. 重复以上步骤

**宏任务 vs 微任务：**

| 类型 | 示例 | 执行时机 |
|------|------|---------|
| 宏任务 | setTimeout, setInterval, I/O, UI 渲染 | 下一轮事件循环 |
| 微任务 | Promise.then, queueMicrotask, MutationObserver | 当前任务执行完毕后，下一轮宏任务之前 |

**实际应用：**
- 需要尽快执行 → 用 Promise.resolve().then()
- 需要延迟执行 → 用 setTimeout

#### 真实面试题（补充）

**题目：浏览器中的事件循环（Event Loop）是如何工作的？**

**满分答案：**

**核心流程：**

```
┌─────────────────────────────────────┐
│ 1. 执行同步代码（主线程）             │
│    ↓                               │
│ 2. 执行所有微任务（Promise.then/MutationObserver/queueMicrotask）│
│    ↓ 微任务队列空                    │
│ 3. 执行一个宏任务（setTimeout/setInterval/I/O/UI渲染） │
│    ↓ 循环回步骤 1                    │
└─────────────────────────────────────┘
```

**关键规则：**

1. **微任务优于宏任务**：每次执行完微任务，必须清空整个微任务队列，才会开始下一个宏任务
2. **微任务可嵌套**：微任务中可以继续添加微任务（立即清空）
3. **UI 渲染在宏任务之后**：通常在每帧结束后触发（约 16.7ms）

**经典题目：执行顺序**

```javascript
console.log('1');           // 同步

setTimeout(() => console.log('2'), 0);  // 宏任务

Promise.resolve().then(() => console.log('3'));  // 微任务

queueMicrotask(() => console.log('4'));  // 微任务

console.log('5');           // 同步

// 输出顺序：1 → 5 → 3 → 4 → 2
```

**async/await 本质：**

```javascript
async function test() {
    console.log('a');
    await console.log('b');
    console.log('c');  // await 后的代码 = 微任务
}

test();
console.log('d');
// 输出：a → b → d → c
// 因为 await console.log('b') 后面的代码被包装成微任务
```

**AI 前端应用：**

- SSE 流式数据 → 微任务批量更新，减少重渲染
- `requestIdleCallback` → 在宏任务之间执行非紧急任务
- React Scheduler → 自己实现任务优先级调度（模拟事件循环）

---

## 2.7 防抖与节流（面试重点）

#### 知识点详解

**防抖（Debounce）：**

```javascript
function debounce(fn, delay = 300, immediate = false) {
    let timer = null;
    
    return function(...args) {
        const context = this;
        
        // 立即执行模式
        if (immediate && !timer) {
            fn.apply(context, args);
        }
        
        // 清除之前的定时器
        clearTimeout(timer);
        
        // 设置新的定时器
        timer = setTimeout(() => {
            if (!immediate) {
                fn.apply(context, args);
            }
            timer = null;
        }, delay);
    };
}

// 使用示例
const debouncedSearch = debounce((query) => {
    console.log('搜索:', query);
    fetchResults(query);
}, 500);

input.addEventListener('input', (e) => {
    debouncedSearch(e.target.value);
});
```

**节流（Throttle）：**

```javascript
function throttle(fn, interval = 300) {
    let lastTime = 0;
    let timer = null;
    
    return function(...args) {
        const context = this;
        const now = Date.now();
        
        if (now - lastTime >= interval) {
            // 间隔时间到了，直接执行
            fn.apply(context, args);
            lastTime = now;
        } else {
            // 还在间隔内，用 setTimeout 保证最后执行一次
            clearTimeout(timer);
            timer = setTimeout(() => {
                fn.apply(context, args);
                lastTime = Date.now();
            }, interval - (now - lastTime));
        }
    };
}

// 使用示例
const throttledScroll = throttle(() => {
    console.log('滚动位置:', window.scrollY);
}, 200);

window.addEventListener('scroll', throttledScroll);
```

**防抖 vs 节流：**

| 特性 | 防抖 | 节流 |
|------|------|------|
| 执行时机 | 最后一次触发后 delay 毫秒 | 每隔 interval 毫秒执行一次 |
| 适用场景 | 搜索框输入、窗口调整 | 滚动、按钮点击、拖拽 |
| 关键词 | "停止输入后" | "每间隔" |

**经典面试题：实现带取消的防抖：**

```javascript
function debounce(fn, delay) {
    let timer = null;
    
    const debounced = function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            fn.apply(this, args);
        }, delay);
    };
    
    // 取消功能
    debounced.cancel = function() {
        clearTimeout(timer);
        timer = null;
    };
    
    // 立即执行版本
    debounced.flush = function(...args) {
        if (timer) {
            clearTimeout(timer);
            fn.apply(this, args);
        }
    };
    
    return debounced;
}

// 使用
const debouncedFn = debounce(saveData, 1000);

// 取消
button.addEventListener('click', () => debouncedFn.cancel());

// 立即执行
button.addEventListener('click', () => debouncedFn.flush());
```

#### 真实面试题

**题目：手写代码：实现一个基础的防抖函数（debounce）**

**满分答案：**

```javascript
function debounce(fn, delay) {
    let timer = null;
    
    return function(...args) {
        // 清除之前的定时器
        clearTimeout(timer);
        
        // 设置新的定时器
        timer = setTimeout(() => {
            // 在定时器到期后执行
            fn.apply(this, args);
        }, delay);
    };
}

// 测试
const debouncedFn = debounce(() => {
    console.log('防抖执行');
}, 300);

window.addEventListener('resize', debouncedFn);
// 连续 resize 时，只会执行一次
```

**进阶：支持立即执行版本：**

```javascript
function debounce(fn, delay, immediate = false) {
    let timer = null;
    
    return function(...args) {
        if (timer) clearTimeout(timer);
        
        if (immediate && !timer) {
            fn.apply(this, args);
        }
        
        timer = setTimeout(() => {
            if (!immediate) fn.apply(this, args);
            timer = null;
        }, delay);
    };
}
```

---

#### 真实面试题（补充）

**题目：简单实现一个防抖函数，并说明它在 AI 搜索建议场景下的作用**

**满分答案：**

**手写防抖：**

```javascript
function debounce(fn, delay = 300) {
    let timer = null;
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => {
            fn.apply(this, args);
        }, delay);
    };
}

// 带立即执行的版本
function debounceImmediate(fn, delay = 300) {
    let timer = null;
    let immediate = true;
    return function(...args) {
        if (immediate) {
            fn.apply(this, args);
            immediate = false;
        }
        clearTimeout(timer);
        timer = setTimeout(() => immediate = true, delay);
    };
}
```

**AI 搜索建议场景的作用：**

```
用户输入 "React"：
用户: R → R-e → R-e-a → R-e-a-c → R-e-a-c-t

不用防抖：每个字触发一次请求 → 5次
用防抖(300ms)：等待用户停顿时触发 → 1-2次

节省：60-80% 的请求量
```

**结合 AI 搜索的完整实现：**

```javascript
function SearchWithAISuggestion() {
    const [query, setQuery] = useState('');
    const [suggestions, setSuggestions] = useState([]);

    // 防抖：用户停止输入 300ms 后才触发搜索
    const debouncedSearch = useMemo(
        () => debounce(async (q) => {
            if (!q.trim()) {
                setSuggestions([]);
                return;
            }
            const results = await fetchAISuggestions(q);
            setSuggestions(results);
        }, 300),
        []
    );

    return (
        <div>
            <input
                value={query}
                onChange={e => {
                    setQuery(e.target.value);
                    debouncedSearch(e.target.value);
                }}
                placeholder="搜索..."
            />
            {suggestions.map(s => <SuggestionItem key={s.id} {...s} />)}
        </div>
    );
}
```

**核心原理：** 防抖确保"最后一次"操作生效，在 AI 搜索场景下节省 API 调用。

---

**题目：Promise.all 和 Promise.allSettled 的区别，在并行调用多个 AI 模型时选哪个？**

**满分答案：**

**区别：**

| 方法 | 行为 | 一个失败 | 全部成功 |
|------|------|---------|---------|
| `Promise.all` | 全部 resolved 才成功 | 立即 reject | ✅ |
| `Promise.allSettled` | 等所有完成 | 不影响其他 | 全部 resolved/rejected 都返回 |

```javascript
// Promise.all：一个失败，全部白干
const results = await Promise.all([
    callGPT4(),
    callClaude(),
    callGemini()
]);
// GPT4 成功，Claude 失败 → 整体 reject，GPT4 结果丢失

// Promise.allSettled：全部跑完，各论各的
const results = await Promise.allSettled([
    callGPT4(),
    callClaude(),
    callGemini()
]);
// 每个结果都带 status: 'fulfilled' | 'rejected'
results.forEach((r, i) => {
    if (r.status === 'fulfilled') {
        console.log(`模型${i}成功:`, r.value);
    } else {
        console.log(`模型${i}失败:`, r.reason);
    }
});
```

**AI 多模型调用选哪个：**
- **选 `Promise.allSettled`（推荐）**：部分模型失败不影响其他结果，全部拿到后再统一处理
- **选 `Promise.all`**：只在所有模型都必须成功时才用（如全部成功才给用户展示）



## 2.8 数组根据对象属性去重

#### 知识点详解

**方法一：Set + Map（最简洁）：**

```javascript
// 使用 Map 基于对象属性去重
function uniqueByProperty(arr, key) {
    const seen = new Map();
    return arr.filter(item => {
        const value = item[key];
        return !seen.has(value) && seen.set(value, true);
    });
}

// 示例
const users = [
    { id: 1, name: '张三' },
    { id: 2, name: '李四' },
    { id: 1, name: '王五' },  // id 重复，会被过滤
    { id: 3, name: '赵六' }
];

const uniqueUsers = uniqueByProperty(users, 'id');
// [{ id: 1, name: '张三' }, { id: 2, name: '李四' }, { id: 3, name: '赵六' }]
```

**方法二：reduce + includes：**

```javascript
function uniqueByReduce(arr, key) {
    return arr.reduce((result, item) => {
        const exists = result.some(i => i[key] === item[key]);
        if (!exists) result.push(item);
        return result;
    }, []);
}
```

**方法三：Object key-value（适合数字 ID）：**

```javascript
function uniqueByObject(arr, key) {
    const obj = {};
    arr.forEach(item => {
        obj[item[key]] = item;
    });
    return Object.values(obj);
}
```

**方法四：多属性去重：**

```javascript
function uniqueByMultipleKeys(arr, ...keys) {
    const seen = new Set();
    
    return arr.filter(item => {
        const key = keys.map(k => item[k]).join('-');
        if (seen.has(key)) return false;
        seen.add(key);
        return true;
    });
}

// 示例：同时根据 id 和 name 去重
const items = [
    { id: 1, name: 'A' },
    { id: 1, name: 'A' },  // 完全重复
    { id: 1, name: 'B' },  // 部分重复，保留
    { id: 2, name: 'A' }
];

uniqueByMultipleKeys(items, 'id', 'name');
// [{ id: 1, name: 'A' }, { id: 1, name: 'B' }, { id: 2, name: 'A' }]
```

**方法五：lodash 的 uniqBy（生产环境推荐）：**

```javascript
import { uniqBy } from 'lodash';

const unique = uniqBy(arr, 'id');
```

#### 真实面试题

**题目：数组中有多个对象，如何根据对象的某个属性进行去重？**

**满分答案：**

```javascript
// 方式一：Map 去重（推荐）
function uniqueById(arr, key = 'id') {
    return [...new Map(arr.map(item => [item[key], item])).values()];
}

// 方式二：reduce 去重
function uniqueByReduce(arr, key) {
    return arr.reduce((acc, cur) => {
        if (!acc.some(item => item[key] === cur[key])) {
            acc.push(cur);
        }
        return acc;
    }, []);
}

// 方式三：对象键值对
function uniqueByObject(arr, key) {
    const result = {};
    arr.forEach(item => {
        result[item[key]] = item;
    });
    return Object.values(result);
}

// 测试
const data = [
    { id: 1, name: 'A' },
    { id: 2, name: 'B' },
    { id: 1, name: 'C' }
];

console.log(uniqueById(data));  // [{ id: 1, name: 'A' }, { id: 2, name: 'B' }]
```

**注意：**
- Map 方式保留最后一个重复项的值
- 如果需要保留第一个，使用 reduce 或 filter

#### 知识点详解

**什么是 DocumentFragment？**

DocumentFragment 是"文档片段"，是一个轻量级的文档对象。

```javascript
// 创建
const fragment = document.createDocumentFragment();
```

**特点：**
1. 不属于真实 DOM 树
2. 在其中添加子元素不会触发回流
3. 一次性将子元素添加到真实 DOM

**使用场景：**

**1. 批量插入 DOM（性能优化）：**

```javascript
// ❌ 每次添加触发回流
const list = document.getElementById('list');
for (let i = 0; i < 1000; i++) {
    const item = document.createElement('div');
    item.textContent = `Item ${i}`;
    list.appendChild(item);  // 触发 1000 次回流
}

// ✅ 使用 DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const item = document.createElement('div');
    item.textContent = `Item ${i}`;
    fragment.appendChild(item);  // 不触发回流
}
list.appendChild(fragment);  // 只触发 1 次回流
```

**2. 模板渲染：**

```javascript
const template = document.createElement('template');
template.innerHTML = `
    <div class="card">
        <h3></h3>
        <p></p>
    </div>
`;

function createCard(title, content) {
    const clone = template.content.cloneNode(true);
    clone.querySelector('h3').textContent = title;
    clone.querySelector('p').textContent = content;
    return clone;
}
```

**与 innerHTML 对比：**

```javascript
// innerHTML（会解析 HTML，可能有安全问题）
container.innerHTML = '<div>content</div>';

// DocumentFragment（更安全，只支持 DOM 节点）
const fragment = document.createDocumentFragment();
const div = document.createElement('div');
div.textContent = 'content';
fragment.appendChild(div);
container.appendChild(fragment);
```

#### 真实面试题

**题目：DocumentFragment 有什么特点？适用于哪些场景？**

**满分答案：**

**特点：**
1. 虚拟文档节点，不属于真实 DOM
2. 在其中操作子节点不触发回流
3. appendChild 会移动节点而非复制

**适用场景：**
1. **批量 DOM 插入**：减少回流次数
2. **模板渲染**：使用 template 标签
3. **节点移动**：高效移动大量节点

**性能优势：**
- 减少重排（reflow）
- 减少重绘（repaint）
- 适合大数据量渲染

---


## 2.9 深拷贝实现

#### 知识点详解

**浅拷贝 vs 深拷贝：**

```javascript
// 浅拷贝：只复制第一层，引用类型仍共享
const obj = { a: 1, b: { c: 2 } };
const shallow = { ...obj };
shallow.b.c = 99;
console.log(obj.b.c); // 99（被修改了！）

// 深拷贝：完全独立的副本
const deep = deepClone(obj);
deep.b.c = 99;
console.log(obj.b.c); // 2（不受影响）
```

**方法一：JSON 序列化（简单但有局限）**

```javascript
const deepClone = (obj) => JSON.parse(JSON.stringify(obj));

// 局限：
// ❌ 函数、undefined、Symbol 会丢失
// ❌ Date 变成字符串
// ❌ RegExp 变成 {}
// ❌ 循环引用会报错
```

**方法二：递归实现（完整版）**

```javascript
function deepClone(obj, map = new WeakMap()) {
    // 处理基本类型和 null
    if (obj === null || typeof obj !== 'object') return obj;

    // 处理特殊类型
    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);
    if (obj instanceof Map) {
        const clonedMap = new Map();
        obj.forEach((v, k) => clonedMap.set(deepClone(k, map), deepClone(v, map)));
        return clonedMap;
    }
    if (obj instanceof Set) {
        const clonedSet = new Set();
        obj.forEach(v => clonedSet.add(deepClone(v, map)));
        return clonedSet;
    }

    // 处理循环引用
    if (map.has(obj)) return map.get(obj);

    // 创建同类型的空对象（保留原型链）
    const cloned = Array.isArray(obj) ? [] : Object.create(Object.getPrototypeOf(obj));
    map.set(obj, cloned);

    // 递归复制所有属性（包括 Symbol）
    [...Object.keys(obj), ...Object.getOwnPropertySymbols(obj)].forEach(key => {
        cloned[key] = deepClone(obj[key], map);
    });

    return cloned;
}

// 测试
const original = {
    num: 1,
    str: 'hello',
    arr: [1, 2, 3],
    nested: { a: { b: 1 } },
    date: new Date(),
    reg: /test/gi,
    fn: () => 'function',
    sym: Symbol('test'),
    map: new Map([['key', 'value']]),
    set: new Set([1, 2, 3])
};
// 循环引用
original.self = original;

const cloned = deepClone(original);
cloned.nested.a.b = 99;
console.log(original.nested.a.b); // 1（不受影响）
console.log(cloned.self === cloned); // true（循环引用正确处理）
```

**方法三：structuredClone（现代浏览器原生）**

```javascript
// 浏览器原生深拷贝，支持循环引用、Date、Map、Set 等
const cloned = structuredClone(obj);

// 局限：不支持函数、Symbol、DOM 节点
```

**性能对比：**

| 方法 | 速度 | 支持函数 | 支持循环引用 | 支持 Date/RegExp |
|------|------|---------|------------|-----------------|
| JSON | 快 | ❌ | ❌ | ❌ |
| 递归 | 中 | ✅ | ✅ | ✅ |
| structuredClone | 快 | ❌ | ✅ | ✅ |

#### 真实面试题

**题目：手写实现一个深拷贝函数。**

**满分答案：**

关键点：
1. **基本类型直接返回**
2. **特殊对象单独处理**（Date、RegExp、Map、Set）
3. **WeakMap 处理循环引用**（避免无限递归）
4. **保留原型链**（`Object.create(Object.getPrototypeOf(obj))`）
5. **处理 Symbol 键**

#### 真实面试题（补充）

**题目：手写一个简单的深拷贝函数，需要考虑循环引用。**

**满分答案：**

**完整实现（含循环引用处理）：**

```typescript
function deepClone<T>(obj: T, map = new WeakMap()): T {
    // 基本类型和 null
    if (obj === null || typeof obj !== 'object') return obj;

    // 特殊对象类型
    if (obj instanceof Date) return new Date(obj) as T;
    if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags) as T;
    if (obj instanceof Map) {
        const cloned = new Map() as T;
        obj.forEach((val, key) => cloned.set(deepClone(key, map), deepClone(val, map)));
        return cloned;
    }
    if (obj instanceof Set) {
        const cloned = new Set() as T;
        obj.forEach(val => cloned.add(deepClone(val, map)));
        return cloned;
    }

    // 处理循环引用：检查是否已拷贝过
    if (map.has(obj as object)) return map.get(obj as object) as T;

    // 克隆对象（保留原型链）
    const cloned = Object.create(Object.getPrototypeOf(obj)) as T;
    map.set(obj as object, cloned);

    // 递归拷贝所有键（包括 Symbol）
    Reflect.ownKeys(obj).forEach(key => {
        cloned[key] = deepClone((obj as Record<string | symbol, unknown>)[key], map);
    });

    return cloned;
}

// ============ 测试 ============
// 循环引用测试
const original: Record<string, unknown> = { name: 'test' };
original.self = original; // 循环引用
const cloned = deepClone(original);
console.log(cloned.self === cloned); // true（循环引用正确处理！）

// 普通对象
const obj = { a: 1, b: { c: 2 }, d: new Date() };
const copy = deepClone(obj);
console.log(copy.b === obj.b); // false（深拷贝）
console.log(copy.d instanceof Date); // true
```

**关键点：**
- `WeakMap` 记录已访问对象，防止无限递归
- 特殊类型（Date/RegExp/Map/Set）单独处理
- `Reflect.ownKeys` 包含 Symbol 键
- `Object.create(Object.getPrototypeOf(obj))` 保留原型链

---

## 2.10 数组去重方法汇总

#### 知识点详解

```javascript
const arr = [1, 2, 2, 3, 3, 3, 'a', 'a', null, null, NaN, NaN];

// 方法1：Set（最简洁，推荐）
const unique1 = [...new Set(arr)];
// [1, 2, 3, 'a', null, NaN]  ✅ NaN 也能去重

// 方法2：filter + indexOf
const unique2 = arr.filter((item, index) => arr.indexOf(item) === index);
// ❌ NaN 无法去重（indexOf 找不到 NaN）

// 方法3：reduce
const unique3 = arr.reduce((acc, cur) => {
    if (!acc.includes(cur)) acc.push(cur);
    return acc;
}, []);
// ✅ includes 能识别 NaN

// 方法4：Map（可处理对象去重）
const unique4 = [...new Map(arr.map(item => [item, item])).values()];

// 方法5：对象数组按属性去重
const users = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 1, name: 'Alice duplicate' }
];
const uniqueUsers = [...new Map(users.map(u => [u.id, u])).values()];
// [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]
```

#### 真实面试题

**题目：如何实现数组去重？请列举几种方法。**

**满分答案：**

1. **Set（推荐）**：`[...new Set(arr)]`，简洁，能处理 NaN
2. **filter + indexOf**：语义清晰，但 NaN 无法去重
3. **reduce + includes**：灵活，能处理 NaN
4. **Map**：适合对象数组按 key 去重

**面试加分：** 说明 `indexOf` 用 `===` 比较，无法识别 NaN；而 `Set` 和 `includes` 用 SameValueZero 算法，能识别 NaN。

---

## 2.11 数组转树结构（任务拆解）

#### 知识点详解

**题目：把扁平任务列表转为嵌套树**

```javascript
/**
 * 数组转树结构
 * @param {Array} list - 扁平的任务列表
 * @param {string} idKey - ID 字段名，默认 'id'
 * @param {string} parentKey - 父 ID 字段名，默认 'parentId'
 * @param {string} rootValue - 根节点的父 ID 值，默认 null
 * @returns {Array} 树结构
 */
function arrayToTree(list, idKey = 'id', parentKey = 'parentId', rootValue = null) {
    // 1. 建立 ID → 节点的 Map（O(n) 查找）
    const map = new Map();
    for (const item of list) {
        map.set(item[idKey], { ...item, children: [] });
    }

    // 2. 构建树
    const tree = [];
    for (const item of list) {
        const node = map.get(item[idKey]);
        const parentId = item[parentKey];

        if (parentId === rootValue || parentId === null || parentId === undefined) {
            // 根节点
            tree.push(node);
        } else {
            // 找到父节点，挂接到 children
            const parent = map.get(parentId);
            if (parent) {
                parent.children.push(node);
            }
        }
    }

    return tree;
}

/**
 * 带排序版本的数组转树
 */
function arrayToTreeSorted(list, idKey = 'id', parentKey = 'parentId', orderKey = 'order') {
    const map = new Map();
    for (const item of list) {
        map.set(item[idKey], { ...item, children: [] });
    }

    const tree = [];
    for (const item of list) {
        const node = map.get(item[idKey]);
        const parentId = item[parentKey];

        if (parentId === null || parentId === undefined) {
            tree.push(node);
        } else {
            const parent = map.get(parentId);
            if (parent) {
                parent.children.push(node);
            }
        }
    }

    // 对每个节点的 children 按 order 排序
    const sortNodes = (nodes) => {
        nodes.sort((a, b) => (a[orderKey] || 0) - (b[orderKey] || 0));
        nodes.forEach(node => sortNodes(node.children));
    };

    sortNodes(tree);
    return tree;
}

/**
 * 树转数组（逆操作）
 */
function treeToArray(tree, childrenKey = 'children') {
    const result = [];
    const traverse = (nodes) => {
        for (const node of nodes) {
            const { [childrenKey]: children, ...rest } = node;
            result.push(rest);
            if (children?.length) {
                traverse(children);
            }
        }
    };
    traverse(tree);
    return result;
}

// ============ 测试用例 ============
const tasks = [
    { id: 1, name: 'AI Agent 开发', parentId: null, order: 1 },
    { id: 2, name: 'Prompt 设计', parentId: 1, order: 1 },
    { id: 3, name: '模型选型', parentId: 1, order: 2 },
    { id: 4, name: '前端界面', parentId: 1, order: 3 },
    { id: 5, name: '上下文管理', parentId: 2, order: 1 },
    { id: 6, name: 'few-shot 示例', parentId: 2, order: 2 },
    { id: 7, name: 'React 实现', parentId: 4, order: 1 },
    { id: 8, name: '流式渲染', parentId: 4, order: 2 }
];

console.log(JSON.stringify(arrayToTreeSorted(tasks), null, 2));
// 输出：
// [
//   {
//     "id": 1, "name": "AI Agent 开发", "children": [
//       { "id": 2, "name": "Prompt 设计", "children": [
//           { "id": 5, "name": "上下文管理", "children": [] },
//           { "id": 6, "name": "few-shot 示例", "children": [] }
//       ]},
//       { "id": 3, "name": "模型选型", "children": [] },
//       { "id": 4, "name": "前端界面", "children": [
//           { "id": 7, "name": "React 实现", "children": [] },
//           { "id": 8, "name": "流式渲染", "children": [] }
//       ]}
//     ]
//   }
// ]
```

**时间复杂度分析：**

| 方法 | 时间复杂度 | 空间复杂度 |
|------|-----------|-----------|
| 双重循环 O(n²) | ❌ 嵌套遍历 | O(1) |
| Map 优化 O(n) | ✅ 一次遍历建 Map，一次遍历建树 | O(n) |

#### 真实面试题

**题目：AI Agent 的任务拆解通常是树状的，请写一个函数将扁平的任务列表转为嵌套树**

**满分答案：**

**Map + 两次遍历（最优解）：**

1. **第一次遍历**：将所有节点存入 `Map<id, node>`，O(n)
2. **第二次遍历**：根据 `parentId` 挂接到父节点的 `children`，O(n)
3. **最终返回根节点集合**，O(n)

**关键点：**
- 用 `Map` 做 O(1) 查找，避免双重循环 O(n²)
- `parentId === null` 或 `undefined` 是根节点
- `children` 数组初始化为空，避免 `undefined.children` 报错
- 可扩展：支持按 `order` 字段排序

---

## 2.12 不冒泡的事件

#### 知识点详解

有些事件天生不会冒泡，理解它们对于事件委托和性能优化至关重要。

**完整列表：**

```javascript
// 焦点事件（可捕获但不冒泡）
focus        // 对应冒泡版本：focusin
blur         // 对应冒泡版本：focusout

// 鼠标进入/离开（不冒泡，且不进入子元素）
mouseenter   // 对应冒泡版本：mouseover
mouseleave   // 对应冒泡版本：mouseout

// 资源加载/错误
load
unload
abort
error

// 视图事件
resize       // 部分浏览器不冒泡
scroll      // 部分浏览器不冒泡

// 指针事件（Pointer Events API）
pointerenter  // 对应冒泡版本：pointerover
pointerleave  // 对应冒泡版本：pointerout

// 媒体事件
playing
paused
ended
waiting
canplay
loadedmetadata
// ... 更多媒体相关事件

// 其他
DOMNodeInsertedIntoDocument  // 已废弃
DOMNodeRemovedFromDocument   // 已废弃
beforeinput
```

**焦点事件捕获 vs 冒泡：**

```javascript
// focus/blur 不冒泡，但可以在捕获阶段监听
element.addEventListener('focus', handler, true);   // 捕获阶段
element.addEventListener('focus', handler, false); // 无效，不会冒泡

// 现代推荐：使用 focusin/focusout（冒泡版本）
element.addEventListener('focusin', handler);  // 会冒泡
```

**mouseenter vs mouseover 关键区别：**

```javascript
// mouseover：进入元素或其任意子元素都会触发（冒泡）
// mouseenter：仅进入元素本身时触发，进入子元素不触发（不冒泡）

const parent = document.getElementById('parent');
const child = parent.querySelector('#child');

// mouseover：从父元素进入子元素
//   → 父元素触发 mouseout
//   → 子元素触发 mouseover（冒泡到父元素）
//   → 父元素又触发 mouseover！

// mouseenter/mouseleave：完全不会冒泡
// 从父元素进入子元素：
//   → 父元素触发 mouseleave（离开父元素）
//   → 子元素触发 mouseenter（进入子元素）
//   → 父元素不再触发任何事件
```

#### 真实面试题

**题目：不会冒泡的事件有哪些？**

**满分答案：**

**核心概念：**
不冒泡的事件是指事件触发后不会沿 DOM 树向上传播到祖先元素。

**完整分类列表：**

| 类别 | 事件 | 对应的冒泡版本 |
|------|------|----------------|
| 焦点事件 | `focus` / `blur` | `focusin` / `focusout` |
| 鼠标进入/离开 | `mouseenter` / `mouseleave` | `mouseover` / `mouseout` |
| 资源加载 | `load` / `unload` / `abort` / `error` | — |
| 视图事件 | `resize` / `scroll` | — |
| 指针事件 | `pointerenter` / `pointerleave` | `pointerover` / `pointerout` |
| 媒体事件 | `playing` / `paused` / `ended` 等 | — |

**为什么这些事件设计为不冒泡？**

1. **focus/blur**：焦点切换是元素自身行为，不需要通知祖先元素；使用 `focusin/focusout` 需要冒泡时才用
2. **mouseenter/mouseleave**：设计初衷就是避免 `mouseover/mouseout` 频繁触发子元素的干扰（进入子元素时父元素反复触发）
3. **load/error**：加载状态是叶子节点的职责，不需要祖先参与
4. **resize**：窗口大小变化只需要在窗口层面处理一次
5. **scroll**：滚动事件如果冒泡，每个祖先都会收到，性能开销大
6. **媒体事件**：播放状态属于媒体元素自身，祖先不需要感知

**实际应用注意事项：**
```javascript
// ❌ 不能对 blur 使用事件委托
document.addEventListener('blur', (e) => {
    // e.target 是失焦的元素，但事件不会冒泡到这里！
}, true); // 只能在捕获阶段监听

// ✅ 正确做法：使用 focusout（冒泡版本）
document.addEventListener('focusout', (e) => {
    // 可以冒泡到 document
});

// ✅ 下拉菜单使用 mouseenter/mouseleave 避免子元素干扰
menu.addEventListener('mouseenter', showMenu);
menu.addEventListener('mouseleave', hideMenu);
```

---

## 2.13 mouseEnter 和 mouseOver 的区别

#### 知识点详解

`mouseenter` 和 `mouseover`（以及 `mouseleave` 和 `mouseout`）看似相似，实际行为有本质区别。

**核心区别总结：**

| 特性 | mouseenter | mouseover |
|------|-----------|-----------|
| 是否冒泡 | 不冒泡 | 冒泡 |
| 进入子元素时 | 不触发 | 触发（父元素也会触发 mouseout→mouseover） |
| 事件委托 | 不支持 | 支持 |
| 性能 | 更好（触发次数少） | 较差（频繁触发） |

**代码演示 — mouseover（冒泡 + 子元素干扰）：**

```javascript
const parent = document.getElementById('parent');
const child = parent.querySelector('#child');

parent.addEventListener('mouseover', (e) => {
    console.log('mouseover:', e.target.id);
});

// 鼠标从 parent 外部 → parent（触发1次）
// 鼠标从 parent → child（触发多次）：
//   1. parent 触发 mouseout（离开 parent）
//   2. child 触发 mouseover（进入 child，冒泡到 parent）
//   3. parent 再次触发 mouseover！
//   → 父元素的 mouseover 被触发了两次！
```

**代码演示 — mouseenter（简单直接）：**

```javascript
parent.addEventListener('mouseenter', (e) => {
    console.log('mouseenter:', e.target.id);
});

// 鼠标从 parent 外部 → parent（触发1次）
// 鼠标从 parent → child：
//   → parent 不触发任何事件
//   → child 触发 mouseenter
// ✅ 父元素完全不受子元素影响
```

**mouseleave vs mouseout 区别同理：**

```javascript
// 鼠标从 child → parent（鼠标回到父元素）
// mouseout：
//   1. child 触发 mouseout（离开 child）
//   2. parent 触发 mouseover（冒泡）

// mouseleave：
//   1. child 触发 mouseleave
//   ✅ parent 不触发任何事件
```

**实际使用场景：**

```javascript
// 场景1：下拉菜单/悬浮菜单（推荐 mouseenter/mouseleave）
const dropdown = document.querySelector('.dropdown');
const menu = dropdown.querySelector('.menu');

// ✅ 推荐：mouseenter + mouseleave
dropdown.addEventListener('mouseenter', () => menu.style.display = 'block');
dropdown.addEventListener('mouseleave', () => menu.style.display = 'none');

// ❌ 不推荐：mouseover/mouseout → 鼠标移到菜单项时会反复触发
//   导致菜单闪烁/抖动

// 场景2：元素高亮效果（鼠标悬停高亮）
item.addEventListener('mouseenter', () => item.classList.add('hover'));
item.addEventListener('mouseleave', () => item.classList.remove('hover'));

// 场景3：需要事件委托时只能用 mouseover/mouseout
// （mouseenter 不冒泡，无法委托）
container.addEventListener('mouseover', (e) => {
    if (e.target.matches('.item')) {
        handleItemHover(e.target);
    }
});
```

**性能对比（子元素多时差异明显）：**

```javascript
// 有20个子元素的容器
// 使用 mouseover：鼠标从父元素进入任一子元素，会触发多次事件
// 使用 mouseenter：每个子元素只触发一次进入事件

// 结论：mouseenter 触发次数 ≈ 子元素数量
//       mouseover 触发次数 ≈ 子元素数量 × 穿越次数
```

#### 真实面试题

**题目：mouseEnter 和 mouseOver 有什么区别？**

**满分答案：**

**核心区别（两点）：**

1. **冒泡 vs 不冒泡**：`mouseover` 会冒泡，`mouseenter` 不会冒泡
2. **子元素干扰**：
   - `mouseover`：鼠标从父元素进入子元素时，父元素也会触发 `mouseover`（因为冒泡）
   - `mouseenter`：仅在进入绑定元素本身时触发，进入子元素不会触发父元素的 `mouseenter`

**代码对比：**

```javascript
// mouseover（进入子元素时父元素会重复触发）
parent.addEventListener('mouseover', () => console.log('over'));
// 父→子：输出 "over"（从父离开）→ "over"（进入子，冒泡到父）

// mouseenter（完全不受子元素影响）
parent.addEventListener('mouseenter', () => console.log('enter'));
// 父→子：只触发子元素的 enter，父元素不触发任何事件
```

**使用场景建议：**

| 场景 | 推荐事件 |
|------|---------|
| 下拉菜单、悬浮卡片 | `mouseenter` / `mouseleave` ✅ |
| 元素高亮/样式变化 | `mouseenter` / `mouseleave` ✅ |
| 需要事件委托（给动态子元素） | `mouseover` / `mouseout` |
| 拖拽进入/离开检测 | `dragenter` / `dragleave`（类似机制） |

**结论**：除需要事件委托外，优先使用 `mouseenter/mouseleave`，行为更可预测，性能也更好。

#### 真实面试题

**题目：请简述一下浏览器中的事件循环机制，特别是微任务和宏任务的执行顺序。**

**满分答案：**

**执行顺序核心规则：**

1. 执行同步代码（调用栈）
2. 清空所有微任务队列
3. 执行一个宏任务
4. 重复 2-3

**微任务（Microtask）：**
- Promise.then / catch / finally
- MutationObserver
- queueMicrotask()
- async 函数 await 后的代码

**宏任务（Macrotask）：**
- setTimeout / setInterval
- I/O 操作
- requestAnimationFrame
- UI 渲染

**完整执行流程：**

```javascript
console.log('1 - 同步');

setTimeout(() => console.log('4 - setTimeout'), 0);

Promise.resolve()
  .then(() => console.log('2 - Promise.then'))
  .then(() => console.log('3 - Promise.then链'));

queueMicrotask(() => console.log('2.5 - queueMicrotask'));

async function test() {
  console.log('async 开始');
  await Promise.resolve();
  console.log('async await 之后 -> 微任务');
}
test();

console.log('1 - 同步结束');

// 输出顺序：
// 1 - 同步
// async 开始
// 1 - 同步结束
// 2 - Promise.then
// 2.5 - queueMicrotask
// 3 - Promise.then链
// async await 之后 -> 微任务
// 4 - setTimeout
```

**关键点：**
- 微任务优先级高于宏任务，同一批微任务按入队顺序执行
- async 函数中，await 相当于暂停，等待 Promise resolve 后将后续代码加入微任务队列
- 微任务执行完后会触发"微任务检查点"，清空队列后再执行下一个宏任务
- 每次宏任务执行前都会重复 2-3 步骤

---

## 2.14 MessageChannel

#### 知识点详解

`MessageChannel` 是 Web API 中用于在同一源的不同 browsing context（如 iframe、worker、主线程）之间建立通信通道的机制。

**基本 API：**

```javascript
// 创建通道，产生两个互通的端口
const channel = new MessageChannel();
const port1 = channel.port1;
const port2 = channel.port2;

// 端口1发，端口2收
port1.onmessage = (event) => console.log('收到:', event.data);
port2.postMessage('Hello from port2');

// 端口2发，端口1收（反向通道）
port2.onmessage = (event) => console.log('收到:', event.data);
port1.postMessage('Hello from port1');

// 注意：必须显式打开端口才能通信
port1.start();
port2.start();

// 不再使用时关闭
port1.close();
```

**核心特点 — 宏任务级别的异步：**

```javascript
// MessageChannel 的 onmessage 是宏任务（不像 Promise.then 是微任务）
const channel = new MessageChannel();
let order = [];

queueMicrotask(() => order.push('microtask'));
channel.port1.onmessage = () => order.push('MessageChannel');

queueMicrotask(() => order.push('microtask2'));
channel.port2.postMessage('ping');

// 执行顺序：microtask → microtask2 → MessageChannel
// 验证：
Promise.resolve().then(() => order.push('Promise微任务'));
setTimeout(() => order.push('setTimeout宏任务'), 0);
// 结果顺序：Promise微任务 → setTimeout宏任务 → MessageChannel
```

**使用场景一：Web Worker 通信（并发）：**

```javascript
// 主线程
const worker = new Worker('worker.js');
worker.postMessage({ type: 'start', data: heavyComputation() });

// worker.js
self.onmessage = (e) => {
    const result = compute(e.data.data);
    self.postMessage({ type: 'result', result });
};

// 或者使用 MessageChannel 让 Worker 直接与另一个 Worker 通信
```

**使用场景二：大任务分片（避免阻塞主线程）：**

```javascript
function scheduleWork(callback, slots) {
    let startTime = performance.now();
    let currentSlot = 0;

    const channel = new MessageChannel();
    channel.port1.onmessage = (e) => {
        if (currentSlot < slots && performance.now() - startTime < 16) {
            // 帧还有时间，继续执行
            currentSlot++;
            callback(currentSlot);
            channel.port2.postMessage(null);
        } else {
            // 时间耗尽，让出主线程
            startTime = performance.now();
            currentSlot = 0;
            requestAnimationFrame(() => {
                channel.port2.postMessage(null);
            });
        }
    };
    channel.port2.postMessage(null);
    return () => channel.port1.close();
}
```

**使用场景三：Vue3 的调度器（scheduler）：**

```javascript
// Vue3 源码简化版
const queue = [];
let isFlushing = false;
const p = Promise.resolve();

function queueJob(job) {
    if (!queue.includes(job)) {
        queue.push(job);
        if (!isFlushing) {
            isFlushing = true;
            // 使用 MessageChannel 将 flush 放到微任务之后、下一帧之前
            nextTick().then(() => {
                flushing = true;
                let job;
                while ((job = queue.shift())) {
                    job();
                }
                flushing = false;
            });
        }
    }
}

// nextTick 的实现优先使用 MessageChannel（性能优于 setTimeout(0)）
let timerFunc;
if (typeof MessageChannel !== 'undefined') {
    const channel = new MessageChannel();
    const port = channel.port1;
    channel.port2.postMessage(null);
    port.onmessage = () => flushJobs; // 宏任务
    timerFunc = () => port.postMessage(null);
} else {
    timerFunc = () => setTimeout(flushJobs, 0);
}
```

**使用场景四：React 的调度器：**

```javascript
// React 早期版本的调度器中曾使用 MessageChannel
// 现代 React 使用 scheduler 包，内部实现类似：

// Scheduler 优先使用 MessageChannel，回退到 setTimeout
const channel = new MessageChannel();
const port = channel.port1;
channel.port2.postMessage(null);

// port.onmessage 会在下一帧前被调用
// 比 setTimeout(0) 更早执行，但仍然是异步的
```

#### 真实面试题

**题目：MessageChannel 是什么？有什么使用场景？**

**满分答案：**

**概念：**
`MessageChannel` 创建两个互通的端口（`port1` 和 `port2`），可以从一个端口发送消息，另一个端口异步接收。

```javascript
const channel = new MessageChannel();
const { port1, port2 } = channel;

port1.onmessage = (e) => console.log('port1收到:', e.data);
port2.postMessage('Hello');

// 注意：需要手动 start() 才正式开始通信（在新版浏览器中 onmessage 已自动 start）
```

**与 postMessage 的区别：**

| 特性 | postMessage | MessageChannel |
|------|------------|----------------|
| 通信方式 | 直接指定目标 window/worker | 创建端口对，端口间互通 |
| 适用场景 | 不同源通信（iframe/worker） | 同一源内的高效通信 |
| 性能 | 需要结构化克隆 | 同上，但更轻量 |
| 灵活性 | 需要目标引用 | 解耦，端口可传递给任意地方 |

**四个典型使用场景：**

**1. Web Worker 通信**
```javascript
const worker = new Worker('task.js');
worker.postMessage({ type: 'compute', data: largeArray });
worker.onmessage = (e) => console.log(e.data.result);
```

**2. 大任务分片（时间切片）**
```javascript
// 将大数据处理分散到多帧执行，避免主线程卡顿
function chunkProcess(data, processItem) {
    const channel = new MessageChannel();
    let i = 0;
    const work = () => {
        let batch = 0;
        while (i < data.length && batch < 1000) {
            processItem(data[i++]);
            batch++;
        }
        if (i < data.length) {
            channel.port2.postMessage(null); // 触发下一帧
        } else {
            channel.port1.close();
        }
    };
    channel.port2.onmessage = work;
    channel.port1.onmessage = work;
    channel.port2.postMessage(null);
}
```

**3. Vue3 调度器（scheduler）**
```javascript
// Vue3 用 MessageChannel 实现 nextTick 的异步队列调度
// 确保同一 tick 内的多次响应式更新合并为一次 DOM 更新
// 比 setTimeout(0) 更早执行，比 Promise.then() 延迟更可控
```

**4. React 调度器**
```javascript
// React Scheduler 中使用 MessageChannel 作为任务调度器
// 实现类似 requestIdleCallback 的功能（该 API 浏览器支持不完整）
// 将任务分散到浏览器空闲时间执行
```

**关键结论**：`MessageChannel` 的核心价值在于宏任务级别的异步调度，是 Vue3 和 React 等框架实现高性能响应式和任务调度的基础设施。

---

## 2.15 async/await 实现原理

#### 知识点详解

`async/await` 是 ES2017 引入的异步编程语法，本质是 Generator 函数 + 自动执行器的语法糖。

**async 函数的转换过程：**

```javascript
// 原始代码
async function fetchData() {
    const res = await fetch('/api/data');
    return res.json();
}

// babel 转换后大致等价于：
function fetchData() {
    return _asyncToGenerator(function* () {
        const res = yield fetch('/api/data');
        return res.json();
    });
}
```

**自动执行器 `_asyncToGenerator`：**

```javascript
function _asyncToGenerator(generatorFn) {
    return function(...args) {
        const generator = generatorFn.apply(this, args);

        function handle(result) {
            // result 是 { done, value }
            if (result.done) {
                // Generator 执行完毕，将返回值作为 Promise resolve 值
                return Promise.resolve(result.value);
            }

            // result.value 是 yield 后面表达式的值（通常是 Promise）
            return Promise.resolve(result.value)
                .then(
                    (resolvedValue) => handle(generator.next(resolvedValue)),
                    (reason) => handle(generator.throw(reason))
                );
        }

        return handle(generator.next());
    };
}
```

**手动模拟 async 函数行为：**

```javascript
// 手写一个 async 函数的等价实现
function myAsync(fn) {
    return function(...args) {
        const gen = fn.apply(this, args);

        return new Promise((resolve, reject) => {
            function step(key, arg) {
                let result;
                try {
                    result = gen[key](arg);
                } catch (err) {
                    return reject(err);
                }

                const { done, value } = result;
                if (done) {
                    return resolve(value);
                } else {
                    return Promise.resolve(value).then(
                        (val) => step('next', val),
                        (err) => step('throw', err)
                    );
                }
            }

            step('next');
        });
    };
}

// 使用
const fetchData = myAsync(function* () {
    const res = yield Promise.resolve({ json: () => 'data' });
    return res.json();
});

fetchData().then(console.log); // 'data'
```

**await 的细节：**

```javascript
// 1. await 后面不是 Promise 会自动包装
async function test() {
    const result = await 123;
    // 等价于 await Promise.resolve(123)
    console.log(result); // 123
}

// 2. await Promise.reject() 会抛出错误
async function test2() {
    try {
        await Promise.reject('error');
    } catch (e) {
        console.log(e); // 'error'
    }
}

// 3. 错误处理的 try/catch 对应 Promise.catch
async function test3() {
    try {
        await fetch('/api');
    } catch (err) {
        // 等价于 Promise.catch()
        console.error(err);
    }
}

// 4. 错误会阻止后续代码执行
async function test4() {
    await step1(); // 失败
    await step2(); // 不会执行
    await step3(); // 不会执行
}

// 5. 顶层 await（ES2022 / 模块中）
// await outside_async function.js
const data = await fetch('/api/config').then(r => r.json());
export default data;
```

**async 函数的返回值：**

```javascript
async function fn() { return 1; }
fn() instanceof Promise; // true，始终返回 Promise
```

#### 真实面试题

**题目：async、await 的实现原理是什么？**

**满分答案：**

**核心原理：**
`async/await` 是 Generator 函数 + 自动执行器的语法糖。编译器将 `async` 函数转为 Generator，将 `await` 转为 `yield`，再用自动执行器自动调用 `next()` 推进 Generator。

**转换过程：**

```javascript
// 原始代码
async function example() {
    const a = await step1();
    const b = await step2(a);
    return b;
}

// Babel/编译器转换后等价于：
function example() {
    return _asyncToGenerator(function* () {
        const a = yield step1();
        const b = yield step2(a);
        return b;
    })();
}
```

**自动执行器核心代码：**

```javascript
function _asyncToGenerator(generatorFn) {
    return function(...args) {
        const gen = generatorFn.apply(this, args);

        function step(method, arg) {
            let result;
            try {
                result = gen[method](arg);
            } catch (err) {
                return Promise.reject(err);
            }

            const { done, value } = result;
            if (done) {
                return Promise.resolve(value);
            }

            // 关键：用 Promise.then 等待 yield 的值，然后继续 next
            return Promise.resolve(value).then(
                (val) => step('next', val),    // 成功，继续
                (err) => step('throw', err)     // 失败，抛出
            );
        }

        return step('next');
    };
}
```

**错误处理机制：**

```javascript
// async 函数中的 try/catch 对应 Promise.catch
async function safe() {
    try {
        await riskyOperation();
    } catch (err) {
        // 等价于 .catch()
        console.error('出错:', err.message);
    }
    // try/catch 之后代码继续执行
}

// 等价于：
function safe() {
    return riskyOperation()
        .catch((err) => console.error('出错:', err.message));
}
```

**await 的特殊规则：**

```javascript
// 1. await 非 Promise 自动包装
await 123;          // 等价于 await Promise.resolve(123)

// 2. await undefined 会永久等待（错误）
// await undefined;  // ❌ 永远不会 resolve

// 3. 顶层 await（模块中可用）
// const config = await fetch('/config').then(r => r.json());
```

**Generator 与 async/await 的关系：**

| 特性 | Generator | async/await |
|------|-----------|-------------|
| 关键字 | `function*` + `yield` | `async function` + `await` |
| 返回值 | Iterator | Promise |
| 执行方式 | 手动调用 `.next()` | 自动执行 |
| 暂停恢复 | 同步代码中可暂停 | 只能在 async 函数中暂停 |
| 错误处理 | try/catch | try/catch |
| 语法简洁度 | 较复杂 | 更简洁 |

---

## 2.16 Proxy 监听嵌套对象

#### 知识点详解

`Proxy` 本身只能拦截第一层属性操作，嵌套对象需要递归代理才能监听。

**问题演示 — 默认只能拦截第一层：**

```javascript
const obj = { person: { name: 'Alice', age: 30 } };
const proxy = new Proxy(obj, {
    get(target, key, receiver) {
        console.log(`get: ${String(key)}`);
        return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
        console.log(`set: ${String(key)} = ${value}`);
        return Reflect.set(target, key, value, receiver);
    }
});

proxy.person.name = 'Bob'; // ❌ 不会触发 set！person 本身没变
proxy.person = { name: 'Bob' }; // ✅ 触发 set（第一层）
proxy.person.age = 31; // ❌ 不触发（嵌套对象的属性修改）
```

**解决方案 — 惰性递归代理（访问时才代理）：**

```javascript
function reactive(obj) {
    if (obj === null || typeof obj !== 'object') {
        return obj; // 基本类型直接返回，不代理
    }

    const proxyMap = new WeakMap(); // 缓存已代理的对象

    function createReactive(target) {
        // 避免重复代理
        if (proxyMap.has(target)) {
            return proxyMap.get(target);
        }

        const proxy = new Proxy(target, {
            get(target, key, receiver) {
                const value = Reflect.get(target, key, receiver);
                // 递归代理：访问嵌套对象时自动代理
                if (value !== null && typeof value === 'object') {
                    return createReactive(value);
                }
                return value;
            },
            set(target, key, value, receiver) {
                const oldValue = target[key];
                const result = Reflect.set(target, key, value, receiver);
                if (result && oldValue !== value) {
                    console.log(`更新: ${String(key)} = ${JSON.stringify(value)}`);
                    // 触发更新通知
                    trigger(target, key);
                }
                return result;
            }
        });

        proxyMap.set(target, proxy);
        return proxy;
    }

    return createReactive(obj);
}

// 使用
const state = reactive({
    user: {
        profile: {
            name: 'Alice',
            settings: { theme: 'dark' }
        }
    }
});

state.user.profile.name = 'Bob'; // ✅ 触发 set
state.user.profile.settings.theme = 'light'; // ✅ 触发 set
```

**Vue3 reactive 的惰性代理策略：**

```javascript
// Vue3 源码简化版
const proxyMap = new WeakMap();

function reactive(obj) {
    // 如果已被代理，直接返回
    if (proxyMap.has(obj)) {
        return proxyMap.get(obj);
    }

    const proxy = new Proxy(obj, {
        get(target, key) {
            const res = Reflect.get(target, key);
            // 依赖收集（track）
            track(target, key);
            // 惰性代理：递归包装嵌套对象
            if (isObject(res)) {
                return reactive(res);
            }
            return res;
        },
        set(target, key, newValue) {
            const oldValue = target[key];
            const result = Reflect.set(target, key, newValue);
            if (result && oldValue !== newValue) {
                // 触发更新（trigger）
                trigger(target, key, newValue, oldValue);
            }
            return result;
        },
        deleteProperty(target, key) {
            const oldValue = target[key];
            const result = Reflect.deleteProperty(target, key);
            if (result) {
                trigger(target, key, undefined, oldValue);
            }
            return result;
        }
    });

    proxyMap.set(obj, proxy);
    return proxy;
}
```

**关键设计决策 — 惰性代理 vs 预代理：**

```javascript
// 预代理：创建时递归代理所有嵌套对象
// 缺点：引用未使用的深层对象也会被代理，浪费内存

// 惰性代理（Vue3 方案）：访问时才递归代理
// 优点：按需代理，内存效率高
// 缺点：首次访问有微小开销

// 示例：惰性代理的好处
const bigData = { deeply: { nested: { value: 123 } } };
const proxy = reactive(bigData);
// 此时 deep.nested.value 还没被代理
const x = proxy.only.read; // 只有这个路径被代理
// 节省了大量不必要的代理
```

#### 真实面试题

**题目：Proxy 能够监听到对象中的对象的引用吗？**

**满分答案：**

**直接回答：不能。** `Proxy` 默认只能拦截第一层属性操作，嵌套对象的属性修改不会触发陷阱。

**原因分析：**

```javascript
const obj = { outer: { inner: { value: 1 } } };
const proxy = new Proxy(obj, {
    set(target, key, value) {
        console.log(`set ${key}`); // 只有这里触发
        return Reflect.set(target, key, value);
    }
});

proxy.outer = { inner: { value: 2 } }; // ✅ 触发（outer 引用变了）
proxy.outer.inner.value = 3;           // ❌ 不触发
// 因为 proxy.outer 获取到的是原始的 { inner: { value: 1 } }
// 没有被 Proxy 包装，无法拦截
```

**解决方案：递归代理（惰性代理）：**

```javascript
const proxyMap = new WeakMap();

function reactive(obj) {
    if (proxyMap.has(obj)) return proxyMap.get(obj);

    const proxy = new Proxy(obj, {
        get(target, key, receiver) {
            const value = Reflect.get(target, key, receiver);
            // 惰性递归：访问嵌套对象时才代理
            if (value !== null && typeof value === 'object') {
                return reactive(value);
            }
            return value;
        },
        set(target, key, value, receiver) {
            const result = Reflect.set(target, key, value, receiver);
            if (result) {
                console.log(`触发更新: ${String(key)} = ${value}`);
                // 通知依赖
            }
            return result;
        }
    });

    proxyMap.set(obj, proxy);
    return proxy;
}

// 测试
const state = reactive({ user: { address: { city: '北京' } } });
state.user = { address: { city: '上海' } }; // ✅ 触发
state.user.address.city = '深圳';          // ✅ 触发（递归代理生效）
```

**Vue3 reactive 的惰性代理策略：**

Vue3 的 `reactive()` 正是采用这种惰性代理：
1. 用 `WeakMap` 缓存已代理对象，避免重复代理（处理循环引用）
2. `get` 时递归代理嵌套对象
3. 首次访问深层属性时才会代理对应路径
4. 不访问的属性不会被代理，节省内存

---

## 2.17 解构赋值对象

#### 知识点详解

数组解构要求被解构对象是可迭代的，对象没有 `Symbol.iterator`，因此不能直接用数组解构。

**问题根源 — 为什么 `var [a, b] = { a: 1, b: 2 }` 失败？**

```javascript
// 数组解构本质是调用 Symbol.iterator
const [a, b] = [1, 2]; // ✅ 数组有 iterator

const obj = { a: 1, b: 2 };
const iterator = obj[Symbol.iterator];
console.log(iterator); // undefined ❌ 对象没有 iterator

// 所以会报错：
// TypeError: {(intermediate value)(intermediate value)} is not iterable
```

**解决方案一：正确使用对象解构（推荐）：**

```javascript
// ✅ 对象解构
const { a, b } = { a: 1, b: 2 };
console.log(a, b); // 1 2

// ✅ 重命名
const { a: first, b: second } = { a: 1, b: 2 };

// ✅ 嵌套解构
const { user: { name, age } } = { user: { name: 'Alice', age: 30 } };
```

**解决方案二：先转为数组再解构：**

```javascript
// Object.values — 获取所有值（按插入顺序）
const [a, b] = Object.values({ a: 1, b: 2 });
console.log(a, b); // 1 2

// Object.entries — 获取键值对数组
const [first, second] = Object.entries({ a: 1, b: 2 });
const [key1, val1] = first;
const [key2, val2] = second;

// 只需要值时用 Object.values 更简洁
```

**解决方案三：给对象添加迭代器（不推荐但有趣）：**

```javascript
// 风险：不推荐修改 Object.prototype，会影响所有对象
Object.prototype[Symbol.iterator] = function() {
    const keys = Object.keys(this);
    let index = 0;
    return {
        next: () => ({
            done: index >= keys.length,
            value: this[keys[index++]]
        })
    };
};

// 现在可以了（非常不推荐这样做）
// const [a, b] = { a: 1, b: 2 }; // ❌ 不要这样做
```

**解决方案四：自定义迭代器（优雅方案）：**

```javascript
// 给特定对象添加迭代器
function createIterableObj(obj) {
    const entries = Object.entries(obj);
    let index = 0;

    return {
        [Symbol.iterator]() {
            return {
                next() {
                    if (index < entries.length) {
                        return { done: false, value: entries[index++][1] };
                    }
                    return { done: true };
                }
            };
        }
    };
}

const iterObj = createIterableObj({ a: 1, b: 2 });
const [a, b] = iterObj;
console.log(a, b); // 1 2
```

**常见错误场景：**

```javascript
// ❌ 常见错误：数组解构对象
const [name, age] = { name: 'Alice', age: 30 };
// TypeError: {(intermediate value)(intermediate value)} is not iterable

// ✅ 正确：对象解构
const { name, age } = { name: 'Alice', age: 30 };

// ✅ 正确：先转值
const [name, age] = Object.values({ name: 'Alice', age: 30 });
console.log(name, age); // 'Alice' 30
```

#### 真实面试题

**题目：如何让 `var [a, b] = { a: 1, b: 2 }` 解构赋值成功？**

**满分答案：**

**报错原因：**
数组解构依赖 `Symbol.iterator` 协议，要求被解构对象必须是一个可迭代对象。普通对象没有 `Symbol.iterator`，所以会抛出 `TypeError: {(intermediate value)} is not iterable`。

**解决方案：**

**方案一：使用对象解构（推荐，正确用法）**

```javascript
// ✅ 对象解构是正解
var { a, b } = { a: 1, b: 2 };
console.log(a, b); // 1 2
```

**方案二：先转数组再解构**

```javascript
// Object.values — 提取所有值
var [a, b] = Object.values({ a: 1, b: 2 });
console.log(a, b); // 1 2

// Object.entries — 提取键值对（如果需要键名）
var [[k1, a], [k2, b]] = Object.entries({ a: 1, b: 2 });
console.log(a, b); // 1 2
```

**方案三：给 Object.prototype 添加迭代器（强烈不推荐）**

```javascript
// 风险：修改 Object.prototype 会影响所有普通对象
// 迭代 for...of 时会产生意外行为
Object.prototype[Symbol.iterator] = function() {
    const keys = Object.keys(this);
    let i = 0;
    return {
        next: () => ({
            done: i >= keys.length,
            value: this[keys[i++]]
        })
    };
};

// 现在这样做可以了（但强烈不推荐）
var [a, b] = { a: 1, b: 2 };
console.log(a, b); // 1 2
```

**方案四：自定义迭代器类（安全的做法）**

```javascript
function IterableObj(obj) {
    this.obj = obj;
}
IterableObj.prototype[Symbol.iterator] = function() {
    const keys = Object.keys(this.obj);
    let i = 0;
    return {
        next: () => ({
            done: i >= keys.length,
            value: this.obj[keys[i++]]
        })
    };
};

const [a, b] = new IterableObj({ a: 1, b: 2 });
console.log(a, b); // 1 2
```

**结论**：如果是业务需求，直接用对象解构 `{ a, b }`；如果必须用数组解构，先 `Object.values()` 转一下。永远不要改 `Object.prototype`。

---

## 2.18 作用域链

#### 知识点详解

作用域链是 JavaScript 查找变量的机制。当访问变量时，先在当前作用域查找，找不到则沿外层作用域逐级向上查找，直到全局作用域。

**作用域链的查找过程：**

```javascript
const globalVar = 'global';

function outer() {
    const outerVar = 'outer';

    function inner() {
        const innerVar = 'inner';

        console.log(innerVar);  // ✅ 就近找到 innerVar
        console.log(outerVar);  // ✅ 沿作用域链找到 outerVar
        console.log(globalVar); // ✅ 找到全局变量
        console.log(nonExistent); // ❌ 沿链找完到 null，返回 ReferenceError
    }

    inner();
}

outer();
```

**作用域链的形成：**

```javascript
// 每个执行上下文有一个 [[Scope]] 属性（函数创建时确定）
function outer() {
    const a = 1;
    function inner() {
        console.log(a); // inner 的 [[Scope]] = [outer的AO, global]
    }
    return inner;
}

// 即使 outer 执行完毕，inner 函数依然持有 outer 的作用域引用
const fn = outer();
fn(); // 1，闭包机制
```

**词法作用域（静态作用域）vs 动态作用域：**

```javascript
// JavaScript 使用词法作用域（定义时确定，不是调用时）
const x = 10;
function f() {
    console.log(x); // 打印定义处能看到的 x = 10
}
function g() {
    const x = 20;
    f(); // 10（不是 20！）因为 f 定义时就已经决定了引用的是外层的 x
}
g();

// 动态作用域（少数语言如 Bash、Perl 某些场景）相反：
// 打印调用栈上最近定义的 x → 20
```

**作用域链与闭包的关系：**

```javascript
// 闭包 = 函数 + 其定义时的作用域链
function makeAdder(x) {
    // x 在这里被创建，作用域链包含 x
    return function(y) {
        return x + y; // x 从闭包中捕获
    };
}

const add5 = makeAdder(5);
const add10 = makeAdder(10);

add5(2);  // 7，x = 5 来自 add5 的闭包
add10(2); // 12，x = 10 来自 add10 的闭包
// 两个闭包各自持有独立的 x
```

**作用域链性能考量：**

```javascript
// 嵌套越深，查找越慢
function level1() {
    const l1 = 1;
    function level2() {
        const l2 = 2;
        function level3() {
            const l3 = 3;
            function level4() {
                console.log(l1); // 查找链：l4 → l3 → l2 → l1
                console.log(l3); // 查找链：l4 → l3
            }
            level4();
        }
        level3();
    }
    level2();
}

// 优化：频繁访问的外层变量缓存到局部变量
function optimized() {
    const config = getGlobalConfig(); // 跨作用域查找

    function handler() {
        const cfg = config; // 一次性跨作用域查找
        for (let i = 0; i < 1000; i++) {
            cfg.method(); // 用局部变量访问，比每次跨作用域查找快
        }
    }
}
```

#### 真实面试题

**题目：什么是作用域链？**

**满分答案：**

**定义：**
作用域链是 JavaScript 引擎用于解析变量引用的机制。当访问一个变量时，引擎从当前执行作用域开始，沿着 `[[Scope]]` 链接逐级向上查找，直到全局作用域，找到则返回，找不到则报 `ReferenceError`。

**查找机制示意：**

```javascript
const global = '🌍';

function A() {
    const a = '📦';

    function B() {
        const b = '📦';

        function C() {
            console.log(a); // C → B的AO → A的AO → global
            console.log(global); // C → global
        }
        C();
    }
    B();
}
A();
```

**词法作用域 vs 动态作用域：**

JavaScript 采用**词法作用域（静态作用域）**，作用域在函数**定义**时确定，不受调用位置影响：

```javascript
const x = '全局';
function foo() { console.log(x); }

function bar() {
    const x = '局部';
    foo(); // 打印 '全局'，不是 '局部'
}
bar();
```

而动态作用域（如 Shell 的 `$0`）在**调用时**确定引用。

**与闭包的关系：**
闭包的本质就是函数携带了定义时的作用域链，使函数在定义作用域之外执行时仍能访问外层变量。

**性能注意：**
- 嵌套越深，变量查找越慢
- 频繁访问的外层变量可缓存到局部变量

---

## 2.19 bind/call/apply 区别与实现

#### 知识点详解

`bind`、`call`、`apply` 都用于改变函数执行时的 `this` 指向，区别在于调用时机和传参方式。

**三者对比：**

| 方法 | 调用方式 | 传参形式 | 返回值 |
|------|---------|---------|--------|
| `call` | 立即执行 `fn.call(this, a, b)` | 逐个参数 | 函数返回值 |
| `apply` | 立即执行 `fn.apply(this, [a, b])` | 数组参数 | 函数返回值 |
| `bind` | 返回新函数 `const f = fn.bind(this, a)` | 逐个参数（可分次） | 新函数（稍后调用） |

**基本使用：**

```javascript
const person = { name: 'Alice' };

function greet(age, city) {
    console.log(`I am ${this.name}, ${age} years old, from ${city}`);
}

greet.call(person, 25, '北京');   // 立即执行
greet.apply(person, [25, '北京']);  // 立即执行，数组传参
greet.bind(person, 25, '北京')(); // 返回新函数，需要手动调用
greet.bind(person)(25, '北京');    // 分次传参
```

**手写实现 — 完整版 bind：**

```javascript
Function.prototype.myBind = function(thisArg, ...bindArgs) {
    if (typeof this !== 'function') {
        throw new TypeError('must be called on a function');
    }

    const originalFn = this;

    function boundFn(...callArgs) {
        // 关键点：使用 new 调用时，this 指向实例而非 thisArg
        const newThis = this instanceof boundFn ? this : thisArg;
        return originalFn.apply(newThis, [...bindArgs, ...callArgs]);
    }

    // 保持原型链
    boundFn.prototype = Object.create(originalFn.prototype);

    return boundFn;
};

// 测试
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.toString = function() {
    return `${this.x},${this.y}`;
};

const YAxisPoint = Point.myBind(null, 0);
const p = new YAxisPoint(5);
console.log(p.toString()); // '0,5'
console.log(p instanceof Point); // true
```

#### 真实面试题

**题目：bind、call、apply 有什么区别？如何实现一个 bind？**

**满分答案：**

**三者对比表：**

| 特性 | call | apply | bind |
|------|------|-------|------|
| 传参形式 | `fn.call(obj, a, b, c)` | `fn.apply(obj, [a, b, c])` | `fn.bind(obj, a, b)(c)` |
| 执行时机 | 立即执行 | 立即执行 | 返回新函数（延迟执行） |
| 返回值 | 函数返回值 | 函数返回值 | 新函数 |
| 绑定 this | ✅ | ✅ | ✅ |
| 预设部分参数 | ❌ | ❌ | ✅（柯里化） |

**bind 完整实现（考虑 new）：**

```javascript
Function.prototype.myBind = function(thisArg, ...bindArgs) {
    const fn = this;

    function bound(...callArgs) {
        // 关键：new 调用时，this 指向实例，应使用实例的 this
        const newThis = this instanceof bound ? this : thisArg;
        return fn.apply(newThis, [...bindArgs, ...callArgs]);
    }

    // 保证 instanceof 和 prototype 正确
    bound.prototype = Object.create(fn.prototype);

    return bound;
};
```

**使用场景：**

```javascript
// call：需要立即执行且参数数量明确
Array.prototype.slice.call({ 0: 'a', 1: 'b', length: 2 }); // ['a', 'b']

// apply：参数是数组或不确定数量
Math.max.apply(null, [3, 1, 4, 1, 5]); // 5
Math.min.apply(null, [3, 1, 4, 1, 5]); // 1

// bind：预设参数、事件处理、函数柯里化
const log = console.log.bind(console); // 固定 this
const fetchData = api.get.bind(api, '/users'); // 预设参数
```

---

## 2.20 CommonJS 和 ES Module 区别

#### 知识点详解

CommonJS（`require`/`module.exports`）和 ES Module（`import`/`export`）是 JavaScript 的两种模块系统。

**核心差异：**

| 特性 | CommonJS (CJS) | ES Module (ESM) |
|------|---------------|----------------|
| 加载方式 | 运行时（动态） | 编译时（静态） |
| 值拷贝 | 浅拷贝，输出值副本 | 引用绑定，动态同步 |
| this 指向 | 当前模块对象 | undefined |
| 导入位置 | 任意（条件导入） | 顶层（静态） |
| 循环引用 | 可能拿到不完整值 | 支持但需小心 |
| tree-shaking | 不支持 | 支持 |
| 浏览器原生支持 | 不支持 | 支持 |

**值拷贝 vs 引用绑定：**

```javascript
// CommonJS — 值拷贝
// counter.js
let count = 0;
module.exports = { count, increment };

// main.js
const { count, increment } = require('./counter');
console.log(count); // 0
increment();
console.log(count); // 0（副本，未变！）❌
// 如果要同步，需要导出函数
```

```javascript
// ES Module — 引用绑定
// counter.mjs
export let count = 0;
export function increment() { count++; }

// main.mjs
import { count, increment } from './counter.mjs';
console.log(count); // 0
increment();
console.log(count); // 1（实时同步）✅
```

**导入位置限制：**

```javascript
// CommonJS — 可以条件导入
if (process.env.NODE_ENV === 'development') {
    const logger = require('./dev-logger');
} else {
    const logger = require('./prod-logger');
}

// ES Module — 必须顶层导入（编译时确定）
// ❌ 不允许
// if (condition) import { a } from './a';
// ✅ 正确
import { a } from './a';
if (condition) { /* 使用 a */ }
```

#### 真实面试题

**题目：common.js 和 es6 中模块引入的区别？**

**满分答案：**

**核心差异（6个维度）：**

| 维度 | CommonJS (require) | ES Module (import) |
|------|-------------------|-------------------|
| 加载方式 | **运行时动态加载** | **编译时静态分析** |
| 值传递 | **值拷贝**（输出副本，模块内部修改不同步） | **引用绑定**（实时同步，模块内部修改外部可见） |
| this 指向 | `module.exports` 对象 | `undefined` |
| 导入位置 | 可在条件语句中（`if(process.env...`） | 必须顶层（静态） |
| 循环引用 | 可能有 undefined 值 | 尽量避免，依赖声明顺序重要 |
| tree-shaking | 不支持（难以静态分析） | 支持（基于 import/export） |

**代码对比：**

```javascript
// CommonJS
let count = 0;
module.exports = { count };
count = 1; // 外部不感知

// ES Module
export let count = 0;
count = 1; // 外部实时感知（read-only binding）
```

**互操作注意事项：**

```javascript
// ESM 导入 CJS（只能默认导入）
import React from 'react'; // React 是 CJS

// CJS 不能直接 require ESM（用 await import()）
const { default: ESM } = await import('./esm.mjs');
```

**实际选型建议**：
- 库/框架：**ESM**（支持 tree-shaking，浏览器原生）
- Node.js 后端：Node 14+ 同时支持两者
- 打包项目：通常 ESM 入口更好，rollup/webpack 会做兼容处理

---

## 2.21 Vue3 响应式设计原理

#### 知识点详解

Vue3 的响应式系统基于 `Proxy`，相比 Vue2 的 `Object.defineProperty` 有根本性优势。

**核心 API — reactive 和 ref：**

```javascript
// reactive — 深度代理对象
const state = reactive({
    user: { name: 'Alice', age: 30 }
});
state.user.name = 'Bob'; // ✅ 触发响应式更新

// ref — 代理基本类型（通过 .value）
const count = ref(0);
count.value++; // ✅ 触发更新
// 模板中自动解包 ref，不需要 .value
```

**Proxy 代理实现：**

```javascript
function reactive(obj) {
    const proxyMap = new WeakMap();

    function createReactive(obj) {
        if (proxyMap.has(obj)) return proxyMap.get(obj);

        const proxy = new Proxy(obj, {
            get(target, key, receiver) {
                const value = Reflect.get(target, key, receiver);
                // 依赖收集
                track(target, key);
                // 惰性递归代理
                if (value !== null && typeof value === 'object') {
                    return createReactive(value);
                }
                return value;
            },
            set(target, key, newValue, receiver) {
                const oldValue = target[key];
                const result = Reflect.set(target, key, newValue, receiver);
                if (result && oldValue !== newValue) {
                    // 触发更新
                    trigger(target, key, newValue, oldValue);
                }
                return result;
            },
            deleteProperty(target, key) {
                const oldValue = target[key];
                const result = Reflect.deleteProperty(target, key);
                if (result) {
                    trigger(target, key, undefined, oldValue);
                }
                return result;
            }
        });

        proxyMap.set(obj, proxy);
        return proxy;
    }

    return createReactive(obj);
}
```

**依赖收集（track）和触发更新（trigger）：**

```javascript
// 正在执行的副作用函数
let activeEffect = null;

function track(target, key) {
    if (activeEffect) {
        // 建立 target.key → effect 的映射
        const depsMap = getOrCreateDepMap(target);
        const deps = getOrCreateSet(depsMap, key);
        deps.add(activeEffect);
        activeEffect.deps.push(deps);
    }
}

function trigger(target, key, newValue, oldValue) {
    // 找到所有依赖 target.key 的 effect，重新执行
    const deps = getDeps(target, key);
    deps.forEach(effect => effect.run());
}
```

**与 Vue2 Object.defineProperty 对比：**

| 特性 | Vue2 (defineProperty) | Vue3 (Proxy) |
|------|----------------------|--------------|
| 新增属性 | ❌ 需要 Vue.set | ✅ 直接支持 |
| 删除属性 | ❌ 需要 Vue.delete | ✅ 支持 deleteProperty |
| 数组下标 | ⚠️ 需特殊处理 | ✅ 直接支持 |
| 性能 | 递归 defineProperty 较重 | 按需代理，更高效 |
| 浏览器 API | 全部支持 | IE 不支持 |

#### 真实面试题

**题目：说说 Vue3 中的响应式设计原理**

**满分答案：**

**核心原理 — 基于 Proxy：**

Vue3 使用 `Proxy` 代理整个对象，拦截 `get`（依赖收集）和 `set`（触发更新），实现数据变化驱动视图更新。

**响应式流程：**

```
数据读取（get）
    ↓
track(target, key) → 收集当前 effect 到 Dep
    ↓
数据修改（set）
    ↓
trigger(target, key) → 通知 Dep 中的所有 effect 重新执行
    ↓
scheduler 批量调度 → 合并多次更新，异步更新 DOM
```

**关键代码实现：**

```javascript
// reactive — 代理对象
const state = reactive({
    user: { name: 'Alice', age: 30 }
});

// 等价于：
const state = new Proxy({
    user: { name: 'Alice', age: 30 }
}, {
    get(target, key) {
        track(target, key);         // 依赖收集
        const val = Reflect.get(target, key);
        return isObject(val) ? reactive(val) : val; // 惰性递归
    },
    set(target, key, value) {
        const result = Reflect.set(target, key, value);
        trigger(target, key);        // 触发更新
        return result;
    }
});

// ref — 代理基本类型
function ref(value) {
    return {
        get value() {
            track(target, 'value');
            return value;
        },
        set value(newVal) {
            value = newVal;
            trigger(target, 'value');
        }
    };
}
```

**与 Vue2 的核心区别：**

| 能力 | Vue2 | Vue3 |
|------|------|------|
| 新增属性响应式 | 需 `Vue.set` | ✅ 直接支持 |
| 删除属性响应式 | 需 `Vue.delete` | ✅ 直接支持 |
| 数组索引响应式 | ⚠️ 有限支持 | ✅ 完全支持 |
| 惰性代理 | 不支持 | ✅ 惰性递归 |
| 性能 | defineProperty 递归开销大 | 按需代理，更高效 |
| IE 兼容 | ✅ 支持 | ❌ 不支持 |

**reactive vs ref：**

```javascript
// reactive 用于对象（深度响应）
const state = reactive({ count: 0 });
state.count++;

// ref 用于基本类型（包装为 .value 对象）
const count = ref(0);
count.value++;

// 模板中 ref 自动解包，不需要 .value
```

---

## 2.22 script 标签放在 header 和 body 底部

#### 知识点详解

`script` 标签的位置直接影响页面渲染行为和 DOM 可用性。

**head 中的行为（阻塞渲染）：**

```html
<!DOCTYPE html>
<html>
<head>
    <!-- ❌ 脚本立即下载并执行，阻塞 HTML 解析 -->
    <!-- 此时 DOM 还没构建，document.body === null -->
    <script>
        console.log(document.body); // null ❌
        console.log(document.getElementById('app')); // null ❌
    </script>
</head>
<body>
    <div id="app">Hello</div>
</body>
</html>
```

**body 底部的行为（DOM 已就绪）：**

```html
<body>
    <div id="app">Hello</div>
    <script>
        // ✅ DOM 已构建完毕，可以安全操作
        console.log(document.getElementById('app')); // <div>Hello</div>
    </script>
</body>
</html>
```

**现代最佳实践 — 使用 defer（head 中推荐）：**

```html
<head>
    <!-- ✅ defer：不阻塞解析，DOM 解析完后执行 -->
    <!-- 多个 defer 脚本按顺序执行 -->
    <script defer src="vendor.js"></script>
    <script defer src="app.js"></script>
</head>
<body>
    <div id="app"></div>
    <!-- ✅ DOMContentLoaded 之前脚本已执行 -->
</body>
```

**async vs defer：**

```html
<script async src="analytics.js"></script>
<!-- async：下载完立即执行，不阻塞解析，但也不保证顺序 -->
<!-- 适合独立脚本（统计、广告等），不依赖 DOM -->

<script defer src="app.js"></script>
<!-- defer：下载不阻塞解析，DOM 解析完后按顺序执行 -->
<!-- 适合需要 DOM 的业务脚本 -->
```

**type="module" 默认等于 defer：**

```html
<!-- type="module" 始终默认 defer，不阻塞 -->
<script type="module" src="module.js"></script>
<!-- 可以用 async 改为下载完立即执行 -->
<script type="module" src="module.js" async></script>
```

#### 真实面试题

**题目：script 标签放在 header 里和放在 body 底部里有什么区别？**

**满分答案：**

**从 JS 执行角度的核心区别：**

| 位置 | HTML 解析 | DOM 是否就绪 | DOM 操作 | 推荐度 |
|------|-----------|------------|---------|--------|
| `head`（同步） | 被阻塞 | ❌ 未构建 | ❌ 不能操作 | ❌ 避免 |
| `head` + `defer` | 不阻塞 | ✅ 解析完后 | ✅ 可以操作 | ✅ 推荐 |
| `head` + `type="module"` | 不阻塞（默认 defer） | ✅ 解析完后 | ✅ 可以操作 | ✅ 推荐 |
| `body` 底部 | 完全解析后 | ✅ 就绪 | ✅ 可以操作 | ✅ 可接受 |

**具体行为：**

```html
<!-- head 中同步脚本（不推荐） -->
<head>
    <script>
        console.log(document.body); // null ❌
        // DOM 还没构建，不能操作 DOM
    </script>
</head>

<!-- body 底部（可接受） -->
<body>
    <div id="app">Hello</div>
    <script>
        console.log(document.getElementById('app')); // ✅ <div>
        // DOM 已完整构建
    </script>
</body>
</html>
```

**现代最佳实践：**

```html
<!-- 推荐：defer 在 head 中 -->
<head>
    <script defer src="bundle.js"></script>
    <!-- defer 特点：下载不阻塞解析，DOM 解析完后按顺序执行 -->
</head>

<!-- 独立脚本可用 async -->
<head>
    <script async src="analytics.js"></script>
    <!-- async：下载完立即执行，不保证顺序 -->
</head>
```

**结论**：同步脚本放 `body` 底部是历史做法，`defer` 是现代标准推荐方案。

---

## 2.23 Vue 中 age 值变化问题分析

#### 知识点详解

Vue 响应式失效是常见陷阱，主要原因是对响应式对象的操作方式不正确。

**陷阱一：直接修改非响应式变量（data 外）：**

```javascript
export default {
    data() {
        return {
            age: 25 // ✅ 在 data 中声明，是响应式的
        };
    },
    mounted() {
        // ✅ 正确方式
        this.age = 30;
        // 或用 this.$set
        this.$set(this, 'age', 30);

        // ❌ 常见错误：试图通过解构破坏响应式
        const { age } = this; // age 是原始值，非响应式代理
        age = 30; // ❌ 不会触发视图更新
    }
};
```

**陷阱二：连续多次修改基本类型（ref）：**

```javascript
import { ref } from 'vue';

// ✅ 正确方式：ref 包装的值要通过 .value 修改
const age = ref(25);
age.value++;
age.value++;
age.value++; // 最终 28

// ❌ 错误做法：ref 不加 .value
age++; // ❌ 无效

// ❌ 在同一个同步代码块中多次修改
age.value = 25;
age.value = 26;
age.value = 27;
// ⚠️ Vue3 会合并这些更新为一次，视图只更新到 27
```

**陷阱三：修改数组通过索引（Vue2）：**

```javascript
// Vue2 中数组通过索引赋值不是响应式的
this.items[0] = newValue;  // ❌ 不触发更新
this.items.length = 0;     // ❌ 不触发更新

// ✅ 正确做法
this.$set(this.items, 0, newValue); // Vue2
this.items.splice(0, 1, newValue);   // ✅ splice 是响应式的
// Vue3 中直接通过索引赋值是响应式的 ✅
```

#### 真实面试题

**题目：下面代码中，点击"+3"按钮后，age 的值是什么？**

```javascript
// Vue3 Composition API 示例
import { ref } from 'vue';
export default {
    setup() {
        const age = ref(25); // ✅ 正确：ref 包装
        const increment = () => age.value++;
        return { age, increment };
    }
    // template: <button @click="age++; age++; age++;">+3</button>
};
```

**满分答案：**

**关键分析：**

1. **是否正确使用响应式 API？**
   - 基本类型 → 是否有 `ref()`
   - 对象类型 → 是否有 `reactive()` 或 `ref()`

2. **ref 是否正确使用 `.value`？**
   - `<script setup>` 中模板自动解包，模板里不需要 `.value`
   - `<script>` 中必须用 `.value`

3. **Vue2 vs Vue3 行为差异？**
   - Vue2 中数组索引赋值不响应
   - Vue3 中数组索引赋值是响应的

**分析：**

```javascript
// 模板中连续修改 ref
<button @click="age++; age++; age++;">+3</button>

// Vue3 行为：微任务批处理
// 同步执行 age.value++ 三次（内存值为 28）
// 然后异步合并为一次 DOM 更新
// 最终视图显示 age = 28
```

**响应式陷阱常见原因：**

| 陷阱 | 错误写法 | 正确写法 |
|------|---------|---------|
| 基本类型不用 ref | `let age = 25;` | `const age = ref(25);` |
| 解构丢失响应式 | `const { age } = state;` | 直接用 `state.age` |
| 数组索引（Vue2） | `arr[i] = val` | `arr.splice(i, 1, val)` |
| 箭头函数代替普通函数 | `methods: { add: () => {} }` | `methods: { add() {} }` |

**结论**：正确使用 `ref` 时，点击 "+3" 后 `age.value` = 28，视图显示 28。

---

## 2.24 Vue created 和 mounted 时间差

#### 知识点详解

`created` 和 `mounted` 是 Vue 组件生命周期钩子，两者的主要区别在于 DOM 是否可用。

**执行时机：**

```javascript
// Vue 生命周期简化流程：
// beforeCreate → created → (编译模板) → beforeMount → mounted → ...
//                                                        ↑
//                                                  created 到这里的时间差
```

**created 阶段：**
- 组件实例已创建
- data、computed、methods、watch 已设置
- **DOM 不可用**（模板未编译/挂载）
- 适合：数据初始化、网络请求（SSR 中也可用）

**mounted 阶段：**
- 组件模板已挂载到真实 DOM
- `$el`、`$refs` 已可用
- **DOM 已就绪**
- 适合：DOM 操作、第三方库初始化、滚动/动画

**两者之间发生了什么：**

```javascript
// created 之后 → mounted 之前
// 1. 如果是运行时编译（没有 vue-loader 的 compile）：
//    将模板编译成 render 函数
//    （使用 template 选项时需要编译，render 函数不需要）

// 2. 创建 VDOM（虚拟 DOM）

// 3. patch（将 VDOM 渲染到真实 DOM）

// 4. 触发子组件的 mounted 钩子
//    父组件 mounted 等待所有同步子组件 mounted 完成
//    （异步组件不等待）

// 5. 执行 $nextTick 中的回调
```

**时间差的影响因素：**

```javascript
// 因素1：模板复杂度
// 模板越复杂，编译时间越长，created → mounted 时间差越大
// 使用 render 函数或 .vue 文件（预编译）可避免运行时编译

// 因素2：子组件数量
// 父组件的 mounted 需要等待同步子组件全部 mounted 后才触发
// 异步子组件不阻塞父组件的 mounted

// 因素3：DOM 操作复杂度
// 大量 DOM 操作会延长 patch 时间

// 因素4：浏览器性能
// CPU/GPU 性能越好，时间差越小
```

**代码示例：**

```javascript
export default {
    data() {
        return { items: [] };
    },
    created() {
        // ✅ 可以发送请求
        fetch('/api/data').then(r => r.json()).then(data => {
            this.items = data;
        });

        // ❌ 不能操作 DOM
        console.log(this.$refs.box); // undefined
    },
    mounted() {
        // ✅ DOM 已就绪，可以操作
        console.log(this.$refs.box); // <div ref="box"></div>
        this.initChart(); // 初始化图表库
        this.scrollToTop(); // 滚动到顶部
    }
};
```

#### 真实面试题

**题目：Vue 中，created 和 mounted 两个钩子之间调用时间差值受什么影响？**

**满分答案：**

**两者之间的执行流程：**

```
created ✅
    ↓
模板编译（运行时编译时）
    ↓
创建虚拟 DOM
    ↓
子组件 created → mounted（同步子组件）
    ↓
patch（VDOM → 真实 DOM）
    ↓
mounted ✅
```

**影响时间差的因素：**

| 因素 | 影响说明 |
|------|---------|
| **模板复杂度** | 运行时编译（`template` 选项）耗时；预编译（`render` 函数）跳过此步 |
| **组件树深度** | 深度越深，子孙组件的 `created/mounted` 越多，父组件 `mounted` 越晚触发 |
| **DOM 操作复杂度** | patch 阶段渲染的 DOM 越多，时间越长 |
| **异步组件数量** | 异步组件不阻塞 `mounted`；同步组件全部完成后才触发父 `mounted` |
| **浏览器性能** | CPU/GPU 越快，编译 + patch 越快 |

**最佳实践：**

```javascript
// ✅ 数据请求 → 放 created（更早发起）
created() {
    this.fetchUser();
    this.initLogic();
}

// ✅ 需要 DOM 的操作 → 必须放 mounted
mounted() {
    this.$refs.chart.init();
    new ThirdPartyLib(this.$el);
    document.addEventListener('scroll', this.onScroll);
}

// ✅ 两者都可以的（轻量初始化）
created() {
    this.timer = setInterval(...); // 定时器两边都可以
}
```

**Vue3 组合式 API 对应关系：**

```javascript
// created → setup()（setup 在 created 之前执行）
// mounted  → onMounted()

import { onMounted } from 'vue';

setup() {
    // 等价于 created
    console.log('setup (类似 created)');
    onMounted(() => {
        // 等价于 mounted
        console.log('mounted');
    });
}
```


---

## 2.25 带并发限制的 Promise 调度器

### 知识点详解

**使用场景：**
- 多 AI 模型并发调用（同时请求 GPT/Claude/Gemini）
- 控制 API 请求并发数，防止接口限流
- 批量任务调度（如批量处理用户消息）

**核心思路：**

```
任务队列：[ task1, task2, task3, task4, task5 ]
                    ↑
                同时执行 N 个
running = 2
limit = 2
```

**实现原理：**

```typescript
class Scheduler {
    private running = 0;
    private queue: Array<() => Promise<unknown>> = [];

    constructor(private limit: number) {}

    add<T>(fn: () => Promise<T>): Promise<T> {
        return new Promise<T>((resolve, reject) => {
            this.queue.push(() => fn().then(resolve).catch(reject));
            this.drain();
        });
    }

    private drain(): void {
        while (this.running < this.limit && this.queue.length > 0) {
            const task = this.queue.shift()!;
            this.running++;
            task().finally(() => {
                this.running--;
                this.drain(); // 完成后继续执行下一个
            });
        }
    }
}
```

### 真实面试题

**题目：手写代码：实现一个带并发限制的 Promise 调度器。**

**满分答案：**

```typescript
class Scheduler {
    private running = 0;
    private queue: Array<() => Promise<unknown>> = [];

    constructor(private limit: number) {}

    add<T>(fn: () => Promise<T>): Promise<T> {
        return new Promise<T>((resolve, reject) => {
            this.queue.push(() => fn().then(resolve).catch(reject));
            this.drain();
        });
    }

    private drain(): void {
        while (this.running < this.limit && this.queue.length > 0) {
            const task = this.queue.shift()!;
            this.running++;
            task().finally(() => {
                this.running--;
                this.drain(); // 递归触发下一个
            });
        }
    }
}

// ============ 测试用例 ============
const sleep = (ms: number) => new Promise(r => setTimeout(r, ms));

const scheduler = new Scheduler(2);

(async () => {
    const start = Date.now();

    scheduler.add(() => sleep(1000).then(() => 1));
    scheduler.add(() => sleep(800).then(() => 2));
    scheduler.add(() => sleep(600).then(() => 3));
    scheduler.add(() => sleep(400).then(() => 4));

    const results = await Promise.all([
        scheduler.add(() => sleep(1000).then(() => 'A')),
        scheduler.add(() => sleep(800).then(() => 'B')),
        scheduler.add(() => sleep(600).then(() => 'C')),
        scheduler.add(() => sleep(400).then(() => 'D')),
    ]);

    console.log('耗时:', Date.now() - start, 'ms'); // ~1600ms（2并发）
    console.log('结果:', results); // ['A','B','C','D']
})();
/*
执行流程：
t=0:   A、B 同时开始
t=800: B 结束 → C 开始
t=1000: A 结束 → D 开始
t=1400: C 结束
t=1600: D 结束
总耗时 ≈ 1600ms（而非 4000ms 串行）
*/
```

**AI 场景实战（多模型并发调用）：**

```typescript
const scheduler = new Scheduler(3); // 最多3个并发

const results = await Promise.allSettled([
    scheduler.add(() => callOpenAI(message)),
    scheduler.add(() => callClaude(message)),
    scheduler.add(() => callGemini(message)),
    scheduler.add(() => callDeepSeek(message)),  // 排队等待
]);

results.forEach((r, i) => {
    if (r.status === 'fulfilled') showModelResponse(i, r.value);
    else showError(i, r.reason);
});
```

**扩展：添加任务优先级**

```typescript
add<T>(fn: () => Promise<T>, priority: number = 0): Promise<T> {
    return new Promise<T>((resolve, reject) => {
        // 优先级高的插入队列前面
        if (priority > 0) {
            this.queue.unshift(() => fn().then(resolve).catch(reject));
        } else {
            this.queue.push(() => fn().then(resolve).catch(reject));
        }
        this.drain();
    });
}
```

---

---

## 2.25 Promise.all vs Promise.allSettled + 深拷贝(循环引用) + 事件循环 + 并发限制调度器

#### 知识点详解

**Promise.all vs Promise.allSettled：**

```javascript
// Promise.all：全部成功才成功，任一失败立即失败
Promise.all([p1, p2, p3])
    .then(results => console.log(results))  // [r1, r2, r3]
    .catch(error => console.error(error));  // 第一个失败的错误

// Promise.allSettled：等待所有完成，无论成功失败
Promise.allSettled([p1, p2, p3])
    .then(results => {
        // [
        //   { status: 'fulfilled', value: r1 },
        //   { status: 'rejected', reason: err },
        //   { status: 'fulfilled', value: r3 }
        // ]
    });
```

| 特性 | Promise.all | Promise.allSettled |
|------|-------------|-------------------|
| 成功条件 | 所有Promise成功 | 所有Promise完成（无论结果） |
| 失败行为 | 任一失败立即reject | 不会reject，返回所有结果 |
| 返回值 | 成功值的数组 | 包含status/value/reason的对象数组 |
| 使用场景 | 需要所有成功才能继续 | 需要知道每个Promise的结果 |

**深拷贝（考虑循环引用）：**

```javascript
function deepClone(obj, map = new WeakMap()) {
    // 基本类型直接返回
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
    
    // 处理循环引用
    if (map.has(obj)) {
        return map.get(obj);
    }
    
    // 处理日期
    if (obj instanceof Date) {
        return new Date(obj.getTime());
    }
    
    // 处理正则
    if (obj instanceof RegExp) {
        return new RegExp(obj);
    }
    
    // 处理Map
    if (obj instanceof Map) {
        const clone = new Map();
        map.set(obj, clone);
        obj.forEach((value, key) => {
            clone.set(deepClone(key, map), deepClone(value, map));
        });
        return clone;
    }
    
    // 处理Set
    if (obj instanceof Set) {
        const clone = new Set();
        map.set(obj, clone);
        obj.forEach(value => {
            clone.add(deepClone(value, map));
        });
        return clone;
    }
    
    // 处理数组
    if (Array.isArray(obj)) {
        const clone = [];
        map.set(obj, clone);
        for (let i = 0; i < obj.length; i++) {
            clone[i] = deepClone(obj[i], map);
        }
        return clone;
    }
    
    // 处理对象
    const clone = Object.create(Object.getPrototypeOf(obj));
    map.set(obj, clone);
    
    for (const key of Object.keys(obj)) {
        clone[key] = deepClone(obj[key], map);
    }
    
    // 拷贝Symbol类型的key
    const symbols = Object.getOwnPropertySymbols(obj);
    for (const sym of symbols) {
        clone[sym] = deepClone(obj[sym], map);
    }
    
    return clone;
}

// 测试用例
const obj = {
    name: 'test',
    date: new Date(),
    map: new Map([['key', 'value']]),
    set: new Set([1, 2, 3]),
    regexp: /test/gi,
    func: function() { return this.name; },  // 函数通常不拷贝
    [Symbol('sym')]: 'symbol value'
};
obj.circular = obj;  // 循环引用

const cloned = deepClone(obj);
console.log(cloned.circular === cloned);  // true，循环引用正确
console.log(cloned.date !== obj.date);    // true，日期是新对象
console.log(cloned.map !== obj.map);      // true，Map是新对象
```

**浏览器事件循环（Event Loop）：**

```
┌─────────────────────────────────────────────────────────────┐
│                        调用栈 (Call Stack)                    │
│  - 同步代码执行                                              │
│  - 函数调用入栈出栈                                          │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                      任务队列 (Task Queues)                   │
├─────────────────────────────┬───────────────────────────────┤
│      微任务队列              │       宏任务队列               │
│  (Microtask Queue)          │   (Macrotask Queue)           │
│                             │                               │
│  - Promise.then()           │  - setTimeout                 │
│  - Promise.catch()          │  - setInterval                │
│  - Promise.finally()        │  - setImmediate (Node)        │
│  - queueMicrotask()         │  - I/O操作                     │
│  - MutationObserver         │  - UI渲染                      │
│  - process.nextTick (Node)  │  - requestAnimationFrame      │
└─────────────────────────────┴───────────────────────────────┘

执行顺序：
1. 执行同步代码（调用栈）
2. 执行所有微任务（清空微任务队列）
3. 执行一个宏任务
4. 重复步骤2-3
```

```javascript
console.log('1');  // 同步

setTimeout(() => {
    console.log('2');  // 宏任务
    Promise.resolve().then(() => {
        console.log('3');  // 微任务（在宏任务之后）
    });
}, 0);

Promise.resolve().then(() => {
    console.log('4');  // 微任务
    setTimeout(() => {
        console.log('5');  // 宏任务（在微任务之后）
    }, 0);
});

console.log('6');  // 同步

// 输出顺序：1 6 4 2 3 5
```

**带并发限制的Promise调度器：**

```javascript
class PromiseScheduler {
    constructor(concurrency) {
        this.concurrency = concurrency;  // 最大并发数
        this.running = 0;                // 当前运行数
        this.queue = [];                 // 等待队列
    }

    // 添加任务
    add(promiseFactory) {
        return new Promise((resolve, reject) => {
            this.queue.push({
                promiseFactory,
                resolve,
                reject
            });
            this.run();
        });
    }

    // 执行队列
    run() {
        // 如果当前运行数小于并发限制，且有等待任务
        while (this.running < this.concurrency && this.queue.length > 0) {
            const { promiseFactory, resolve, reject } = this.queue.shift();
            this.running++;
            
            promiseFactory()
                .then(resolve)
                .catch(reject)
                .finally(() => {
                    this.running--;
                    this.run();  // 继续执行队列中的下一个
                });
        }
    }
}

// 使用示例
const scheduler = new PromiseScheduler(3);  // 最多3个并发

const urls = ['url1', 'url2', 'url3', 'url4', 'url5'];

const promises = urls.map(url => 
    scheduler.add(() => fetch(url).then(r => r.json()))
);

Promise.all(promises).then(results => {
    console.log('所有请求完成:', results);
});

// 更简洁的实现（使用async/await）
async function asyncPool(concurrency, iterable, iteratorFn) {
    const ret = [];      // 存储所有Promise
    const executing = []; // 存储正在执行的Promise
    
    for (const item of iterable) {
        const p = Promise.resolve().then(() => iteratorFn(item));
        ret.push(p);
        
        if (iterable.length >= concurrency) {
            const e = p.then(() => {
                // 当这个Promise完成时，从executing中移除
                executing.splice(executing.indexOf(e), 1);
            });
            executing.push(e);
            
            // 如果正在执行的数量达到并发限制，等待其中一个完成
            if (executing.length >= concurrency) {
                await Promise.race(executing);
            }
        }
    }
    
    return Promise.all(ret);
}

// 使用asyncPool
asyncPool(3, urls, url => fetch(url))
    .then(results => console.log(results));
```

#### 真实面试题

**题目1：Promise.all 和 Promise.allSettled 的区别是什么？**

**满分答案：**

| 特性 | Promise.all | Promise.allSettled |
|------|-------------|-------------------|
| 成功条件 | 所有Promise成功 | 所有Promise完成 |
| 失败行为 | 任一失败立即reject | 不会reject |
| 返回值 | 成功值数组 | 包含status/value/reason的对象数组 |
| 使用场景 | 需要所有成功才能继续 | 需要知道每个Promise的结果 |

```javascript
// Promise.all：任一失败整体失败
Promise.all([p1, p2, p3]).catch(err => handleError(err));

// Promise.allSettled：获取所有结果
Promise.allSettled([p1, p2, p3]).then(results => {
    results.forEach(r => {
        if (r.status === 'fulfilled') {
            console.log('成功:', r.value);
        } else {
            console.log('失败:', r.reason);
        }
    });
});
```

---

**题目2：请手写一个简单的深拷贝函数，需要考虑循环引用。**

**满分答案：**

```javascript
function deepClone(obj, map = new WeakMap()) {
    if (obj === null || typeof obj !== 'object') return obj;
    
    // 处理循环引用
    if (map.has(obj)) return map.get(obj);
    
    // 处理特殊类型
    if (obj instanceof Date) return new Date(obj.getTime());
    if (obj instanceof RegExp) return new RegExp(obj);
    if (obj instanceof Map) {
        const clone = new Map();
        map.set(obj, clone);
        obj.forEach((v, k) => clone.set(deepClone(k, map), deepClone(v, map)));
        return clone;
    }
    if (obj instanceof Set) {
        const clone = new Set();
        map.set(obj, clone);
        obj.forEach(v => clone.add(deepClone(v, map)));
        return clone;
    }
    
    // 处理数组和对象
    const clone = Array.isArray(obj) ? [] : Object.create(Object.getPrototypeOf(obj));
    map.set(obj, clone);
    
    for (const key of [...Object.keys(obj), ...Object.getOwnPropertySymbols(obj)]) {
        clone[key] = deepClone(obj[key], map);
    }
    
    return clone;
}
```

**关键点：**
1. 使用WeakMap记录已拷贝的对象，处理循环引用
2. 特殊处理Date、RegExp、Map、Set
3. 拷贝Symbol类型的key

---

**题目3：浏览器中的事件循环(EventLoop)是如何工作的？**

**满分答案：**

**执行顺序：**
1. 执行同步代码（调用栈）
2. 执行所有微任务（Promise.then、queueMicrotask）
3. 执行一个宏任务（setTimeout、setInterval、I/O）
4. 重复步骤2-3

**微任务 vs 宏任务：**
- 微任务：Promise.then/catch/finally、queueMicrotask、MutationObserver
- 宏任务：setTimeout、setInterval、setImmediate、I/O、UI渲染

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// 输出：1 4 3 2
```

---

**题目4：手写代码：实现一个带并发限制的Promise调度器。**

**满分答案：**

```javascript
class PromiseScheduler {
    constructor(concurrency) {
        this.concurrency = concurrency;
        this.running = 0;
        this.queue = [];
    }

    add(promiseFactory) {
        return new Promise((resolve, reject) => {
            this.queue.push({ promiseFactory, resolve, reject });
            this.run();
        });
    }

    run() {
        while (this.running < this.concurrency && this.queue.length > 0) {
            const { promiseFactory, resolve, reject } = this.queue.shift();
            this.running++;
            promiseFactory()
                .then(resolve)
                .catch(reject)
                .finally(() => {
                    this.running--;
                    this.run();
                });
        }
    }
}

// 使用
const scheduler = new PromiseScheduler(3);
urls.forEach(url => 
    scheduler.add(() => fetch(url))
**核心原理：**

```typescript
Promise.all = function <T>(promises: Promise<T>[]): Promise<T[]> {
  const results: T[] = new Array(promises.length);
  let completed = 0;

  return new Promise((resolve, reject) => {
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        (value) => {
          results[i] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        },
        (reason) => reject(reason)  // 快速失败
      );
    });
  });
};
```

**关键细节：**
1. Promise.resolve(p) 包装：支持普通值和 Promise 混合
2. results[i] = value：保持结果顺序（即使先完成也放对位置）
3. reject：任一失败立即 reject（快速失败）
4. 返回新 Promise，状态跟随所有 Promise
5. 空数组边界：直接 resolve([])

---

## 2.26 手写 Promise.all

#### 知识点详解

**核心原理：**

```typescript
Promise.all = function <T>(promises: Promise<T>[]): Promise<T[]> {
  const results: T[] = new Array(promises.length);
  let completed = 0;

  return new Promise((resolve, reject) => {
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        (value) => {
          results[i] = value;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        },
        (reason) => reject(reason)  // 快速失败
      );
    });
  });
};
```

**关键细节：**
1. Promise.resolve(p) 包装：支持普通值和 Promise 混合
2. results[i] = value：保持结果顺序（即使先完成也放对位置）
3. reject：任一失败立即 reject（快速失败）
4. 返回新 Promise，状态跟随所有 Promise
5. 空数组边界：直接 resolve([])

#### 真实面试题

**题目：请手写一个简单的 Promise.all 实现。**

**满分答案：**

```typescript
function promiseAll<T>(promises: Promise<T>[]): Promise<T[]> {
  const results: T[] = new Array(promises.length);
  let completed = 0;

  return new Promise((resolve, reject) => {
    // 边界：空数组
    if (promises.length === 0) {
      resolve([]);
      return;
    }

    promises.forEach((p, index) => {
      // Promise.resolve 兼容普通值
      Promise.resolve(p).then(
        (value) => {
          results[index] = value as T;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        },
        (error) => reject(error)  // 快速失败
      );
    });
  });
}

// ============ 测试用例 ============
(async () => {
  const r1 = await promiseAll([
    Promise.resolve(1),
    Promise.resolve(2),
    Promise.resolve(3),
  ]);
  console.log(r1); // [1, 2, 3]

  // 顺序保持测试
  const r2 = await promiseAll([
    new Promise(r => setTimeout(() => r(1), 100)),
    new Promise(r => setTimeout(() => r(2), 10)),
    new Promise(r => setTimeout(() => r(3), 50)),
  ]);
  console.log(r2); // [1, 2, 3] 顺序保持

  // 错误处理
  try {
    await promiseAll([
      Promise.resolve(1),
      Promise.reject(new Error('oops')),
      Promise.resolve(3),
    ]);
  } catch (e) {
    console.log('捕获错误:', e); // Error: oops
  }
})();
```

**面试加分点：**
- 提到"空数组边界情况"
- 提到"Promise.resolve 包装原始值"
- 提到"Promise.allSettled 用于不快速失败的场景"
- 保持结果顺序（即使 P2 比 P1 先完成）
);
```

**核心思路：**
- 维护一个任务队列
- 控制同时运行的任务数不超过并发限制
- 一个任务完成后，自动从队列中取出下一个执行


## 2.27 Promise.all vs Promise.allSettled

#### 知识点详解

| 特性 | Promise.all | Promise.allSettled |
|------|-------------|-------------------|
| 成功条件 | 所有成功 | 所有完成 |
| 失败行为 | 立即 reject | 等待全部 |
| 返回值 | 结果数组 | 状态对象数组 |

```typescript
// 多模型对比 → allSettled
const results = await Promise.allSettled([
    callOpenAI(prompt),
    callClaude(prompt),
    callGemini(prompt)
]);
results.forEach((r, i) => {
    if (r.status === 'fulfilled') useResponse(i, r.value);
    else showError(i, r.reason);
});
```

#### 真实面试题

**题目：Promise.all 和 Promise.allSettled 的区别？AI 模型调用选哪个？**

**满分答案：**

**区别：**

| 特性 | Promise.all | Promise.allSettled |
|------|-------------|-------------------|
| 成功条件 | 全部成功 | 全部完成 |
| 失败行为 | 立即 reject | 等待全部完成 |
| 返回值 | 结果数组 | `{status, value/reason}` 数组 |

**AI 模型调用选择：**

**选 Promise.allSettled：**
- 多模型对比展示
- 单个失败不影响其他
- 需要知道每个模型结果

```typescript
const results = await Promise.allSettled([
    callOpenAI(message),
    callClaude(message)
]);
results.forEach((r, i) => {
    if (r.status === 'fulfilled') showModelResponse(i, r.value);
    else showModelError(i);
});
```

**选 Promise.all：**
- 数据聚合分析（需全部成功）
- 任一失败则整体失败

---


---

## 2.28 带并发限制的 Promise 调度器

### 知识点详解

**核心思想：**
- 同时最多执行 N 个异步任务
- 新任务排队等待，空闲时自动执行
- 任务完成后自动补充执行

```typescript
class PromiseScheduler {
    private running = 0;
    private queue: Array<() => void> = [];

    constructor(private limit: number) {}

    async add<T>(fn: () => Promise<T>): Promise<T> {
        return new Promise((resolve, reject) => {
            this.queue.push(() => fn().then(resolve).catch(reject));
            this.process();
        });
    }

    private process(): void {
        while (this.running < this.limit && this.queue.length > 0) {
            const task = this.queue.shift()!;
            this.running++;
            task().finally(() => {
                this.running--;
                this.process(); // 继续处理下一个
            });
        }
    }
}

// 使用示例
const scheduler = new PromiseScheduler(3); // 最多3个并发

const results = await Promise.all([
    scheduler.add(() => callAPI('/model-1')),
    scheduler.add(() => callAPI('/model-2')),
    scheduler.add(() => callAPI('/model-3')),
    scheduler.add(() => callAPI('/model-4')), // 排队等待
    scheduler.add(() => callAPI('/model-5')),
]);
```

#### 真实面试题

**题目：手写代码：实现一个带并发限制的 Promise 调度器。**

**满分答案：**

**核心思路：**
1. 维护一个"正在执行"计数器
2. 任务包装成 Promise 加入队列
3. 每完成一个，从队列取下一个继续执行

**完整实现：**

```typescript
class Scheduler {
    private running = 0;
    private queue: Array<() => Promise<unknown>> = [];

    constructor(private limit: number) {}

    add<T>(fn: () => Promise<T>): Promise<T> {
        return new Promise((resolve, reject) => {
            this.queue.push(() => fn().then(resolve).catch(reject));
            this.drain();
        });
    }

    private drain(): void {
        while (this.running < this.limit && this.queue.length > 0) {
            const task = this.queue.shift()!;
            this.running++;
            task().finally(() => {
                this.running--;
                this.drain(); // 递归：继续执行下一个
            });
        }
    }
}

// ============ 测试用例 ============
const scheduler = new Scheduler(2); // 最多2个并发

async function simulateRequest(id: number, ms: number) {
    console.log(`[${id}] 开始`);
    await new Promise(r => setTimeout(r, ms));
    console.log(`[${id}] 完成`);
    return id;
}

(async () => {
    const results = await Promise.all([
        scheduler.add(() => simulateRequest(1, 1000)),
        scheduler.add(() => simulateRequest(2, 800)),
        scheduler.add(() => simulateRequest(3, 600)), // 等待前面完成
        scheduler.add(() => simulateRequest(4, 400)), // 等待前面完成
    ]);
    console.log('全部完成:', results);
})();
/*
执行顺序：
[1] 开始
[2] 开始
[1] 完成 → [3] 开始
[2] 完成 → [4] 开始
[3] 完成
[4] 完成
全部完成: [1, 2, 3, 4]
*/
```

**AI 场景应用：**
- 多 AI 模型并发调用（如同时查询 GPT/Claude/Gemini）
- 批量发送 AI 请求时控制并发，防止接口限流
- 限制同时进行的 Embedding 请求数量

---



---

## 2.29 手写防抖（Debounce）函数

#### 知识点详解

**手写防抖（基础版）：**

```typescript
function debounce<T extends (...args: unknown[]) => unknown>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timer: ReturnType<typeof setTimeout> | null = null;

  return function (this: unknown, ...args: Parameters<T>) {
    if (timer !== null) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}

// 测试
const handleSearch = (query: string) => {
  console.log('搜索:', query);
};

const debouncedSearch = debounce(handleSearch, 300);

// AI 搜索建议场景
input.addEventListener('input', (e) => {
  debouncedSearch((e.target as HTMLInputElement).value);
});
```

**进阶版（立即执行 + 取消）：**

```typescript
interface DebouncedFn<T extends unknown[]> {
  (this: unknown, ...args: T): void;
  cancel(): void;
  flush(): void;
}

function debounce<T extends unknown[]>(
  fn: (...args: T) => unknown,
  delay: number,
  immediate = false
): DebouncedFn<T> {
  let timer: ReturnType<typeof setTimeout> | null = null;

  const debounced = function (this: unknown, ...args: T) {
    if (timer !== null) clearTimeout(timer);

    if (immediate && !timer) {
      fn.apply(this, args);
    }

    timer = setTimeout(() => {
      if (!immediate) fn.apply(this, args);
      timer = null;
    }, delay);
  };

  debounced.cancel = () => {
    if (timer !== null) {
      clearTimeout(timer);
      timer = null;
    }
  };

  debounced.flush = () => {
    if (timer !== null) {
      clearTimeout(timer);
      timer = null;
      fn.apply(debounced, args_ref);
    }
  };

  let args_ref: T;
  return debounced as DebouncedFn<T>;
}
```

**节流（Throttle）：**

```typescript
function throttle<T extends unknown[]>(
  fn: (...args: T) => unknown,
  delay: number
): DebouncedFn<T> {
  let last = 0;

  return function (this: unknown, ...args: T) {
    const now = Date.now();
    if (now - last >= delay) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

#### 真实面试题

**题目：手写题：请实现一个简单的防抖（Debounce）函数，并说明它在 AI 搜索建议场景下的应用。**

**满分答案：**

```typescript
function debounce<T extends (...args: unknown[]) => unknown>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timer: ReturnType<typeof setTimeout> | null = null;

  return function (...args: Parameters<T>) {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => {
      fn(...args);
      timer = null;
    }, delay);
  };
}

// AI 搜索建议场景：
// 用户输入"前端框架哪个好" -> 触发6次input事件
// 防抖后：只发送最后一次（300ms无新输入后）
const onInput = debounce((value: string) => {
  fetchAICompletions(value);  // 只发1次请求
}, 300);

input.addEventListener('input', (e) => onInput(e.target.value));
```

**面试加分点：**
- 提到"AI 搜索用防抖节省 Token，每次请求都花钱"
- 提到"cancel() 取消最后一次，用在组件卸载时"
- 提到"immediate=true 立即执行第一次，用户感觉更快"
---
---
