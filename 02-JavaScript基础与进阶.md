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

## 2.19 大对象深度对比

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

## 2.20 DocumentFragment API

---

## 2.21 浏览器事件循环（Event Loop）

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

---

## 2.22 防抖与节流（面试重点）

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

## 2.23 数组根据对象属性去重

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

