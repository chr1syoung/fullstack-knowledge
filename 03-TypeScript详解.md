# 三、TypeScript 详解

---

## 3.1 类型系统

### 3.1.1 基础类型

#### 知识点详解

**原始类型：**

```typescript
// 布尔值
let isDone: boolean = false;

// 数字
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;

// 字符串
let color: string = "blue";
let fullName: string = `Bob Bobbington`;
let sentence: string = `Hello, my name is ${fullName}.`;

// 空值
function warnUser(): void {
    console.log("This is my warning message");
}
let unusable: void = undefined;  // 只能赋值 undefined 或 null

// null 和 undefined
let u: undefined = undefined;
let n: null = null;

// 默认情况下 null 和 undefined 是所有类型的子类型
// 但启用 --strictNullChecks 后，它们只能赋值给自己和 void
```

**数组类型：**

```typescript
// 方式1：元素类型后面接 []
let list: number[] = [1, 2, 3];

// 方式2：数组泛型
let list: Array<number> = [1, 2, 3];

// 只读数组
let readonlyList: ReadonlyArray<number> = [1, 2, 3];
readonlyList[0] = 4;  // 错误
```

**元组（Tuple）：**

```typescript
// 固定长度和类型的数组
let x: [string, number];
x = ["hello", 10];  // OK
x = [10, "hello"];  // Error

// 访问越界元素
x[2] = "world";  // Error，越界访问

// 可选元素
let y: [string, number?, boolean?];
y = ["hello"];
y = ["hello", 1];
y = ["hello", 1, true];

// 带标签的元组（提高可读性）
type Range = [start: number, end: number];
let range: Range = [0, 10];
```

**枚举（Enum）：**

```typescript
// 数字枚举（默认从 0 开始）
enum Direction {
    Up,
    Down,
    Left,
    Right
}
console.log(Direction.Up);    // 0
console.log(Direction[0]);    // "Up"

// 手动设置值
enum Direction {
    Up = 1,
    Down,   // 2
    Left,   // 3
    Right   // 4
}

// 字符串枚举
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT"
}

// 异构枚举（混合）
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}

// 常量枚举（编译时内联，更高效）
const enum Direction {
    Up,
    Down,
    Left,
    Right
}
let directions = [Direction.Up, Direction.Down];
// 编译后：var directions = [0 /* Up */, 1 /* Down */];
```

**Any、Unknown、Never：**

```typescript
// Any：放弃类型检查
let anything: any = 4;
anything = "maybe a string";
anything = false;
anything.ifItExists();  // 编译通过，运行时可能报错

// Unknown：安全的 any
let uncertain: unknown = 4;
uncertain = "maybe a string";

// 使用前必须进行类型检查
if (typeof uncertain === "string") {
    console.log(uncertain.toUpperCase());  // OK
}
// uncertain.toUpperCase();  // Error

// 类型断言
(uncertain as string).toUpperCase();

// Never：永不存在的类型
// 1. 总是抛出异常的函数
function error(message: string): never {
    throw new Error(message);
}

// 2. 无限循环
function infiniteLoop(): never {
    while (true) {}
}

// 3. 类型收窄的穷尽检查
type Shape = "circle" | "square";

function getArea(shape: Shape): number {
    switch (shape) {
        case "circle":
            return Math.PI * 1 * 1;
        case "square":
            return 1 * 1;
        default:
            // 如果未来添加新类型，这里会报错
            const _exhaustiveCheck: never = shape;
            return _exhaustiveCheck;
    }
}
```

#### 真实面试题

**题目：`any`、`unknown`、`never` 有什么区别？何时使用？**

**满分答案：**

**any：**
- 放弃类型检查，允许任何操作
- 是所有类型的父类型，也是所有类型的子类型
- 会"传染"给其他变量

```typescript
let a: any = "hello";
let b: number = a;  // 不报错，但运行时可能出错
a.nonExistentMethod();  // 编译通过
```

**unknown：**
- 安全的 any，是所有类型的父类型，但不是子类型
- 使用前必须进行类型检查或断言

```typescript
let u: unknown = "hello";
let c: number = u;  // Error

if (typeof u === "string") {
    console.log(u.toUpperCase());  // OK
}
```

**never：**
- 表示永不存在的类型
- 是所有类型的子类型，没有类型是 never 的子类型

**使用场景：**

| 类型 | 使用场景 |
|------|----------|
| any | 迁移 JS 代码、第三方库无类型定义、快速原型 |
| unknown | 不确定类型但需要类型安全、输入验证 |
| never | 不可能的分支、穷尽检查、总是抛错的函数 |

---

### 3.1.2 接口与类型别名

#### 知识点详解

**接口（Interface）：**

```typescript
// 基本用法
interface User {
    id: number;
    name: string;
    age?: number;  // 可选属性
    readonly email: string;  // 只读属性
}

const user: User = {
    id: 1,
    name: "John",
    email: "john@example.com"
};

// 函数类型
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc = (src, sub) => {
    return src.search(sub) !== -1;
};

// 可索引类型
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray = ["Bob", "Fred"];

// 字典模式
interface NumberDictionary {
    [index: string]: number | string;
    length: number;    // ok
    name: string;      // ok
}

// 类类型
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date): void;
}

class Clock implements ClockInterface {
    currentTime: Date = new Date();
    setTime(d: Date) {
        this.currentTime = d;
    }
}

// 接口继承
interface Animal {
    name: string;
}

interface Dog extends Animal {
    breed: string;
}

// 多继承
interface Pet {
    play(): void;
}

interface WorkingDog extends Dog, Pet {
    work(): void;
}
```

**类型别名（Type）：**

```typescript
// 基本用法
type UserName = string;
type UserAge = number;

// 联合类型
type ID = string | number;

// 交叉类型
type Employee = User & {
    employeeId: string;
    department: string;
};

// 字面量类型
type Direction = "up" | "down" | "left" | "right";
type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;

// 对象类型
type Point = {
    x: number;
    y: number;
};

// 函数类型
type Callback = (data: string) => void;

// 映射类型
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

// 条件类型
type NonNullable<T> = T extends null | undefined ? never : T;
```

**Interface vs Type：**

```typescript
// 1. 接口可以重复声明，类型别名不行
interface User {
    name: string;
}
interface User {
    age: number;
}
// User 现在有两个属性

type Person = { name: string };
type Person = { age: number };  // Error: Duplicate identifier

// 2. 类型别名支持联合、交叉、元组、映射等
type Union = A | B;
type Intersection = A & B;
type Tuple = [string, number];
type Mapped = { [K in keyof T]: T[K] };

// 3. 接口更适合类实现
interface IUser {
    name: string;
}
class User implements IUser {
    name = "";
}

// 4. extends vs &
interface A { a: string }
interface B extends A { b: string }

type C = { a: string } & { b: string };
```

#### 真实面试题

**题目：`interface` 和 `type` 有什么区别？如何选择？**

**满分答案：**

**区别：**

| 特性 | Interface | Type |
|------|-----------|------|
| 重复声明 | ✓ 支持合并 | ✗ 不支持 |
| extends/implements | ✓ 原生支持 | 需使用 & 模拟 |
| 联合类型 | ✗ 不支持 | ✓ 支持 |
| 交叉类型 | ✗ 不支持 | ✓ 支持 |
| 元组 | ✗ 不支持 | ✓ 支持 |
| 映射类型 | ✗ 不支持 | ✓ 支持 |
| 条件类型 | ✗ 不支持 | ✓ 支持 |

**选择建议：**

1. **优先使用 interface**：
   - 定义对象结构
   - 类实现接口
   - 需要声明合并（如扩展第三方库类型）

2. **使用 type**：
   - 联合类型、交叉类型
   - 元组
   - 映射类型
   - 条件类型
   - 类型别名（简化复杂类型）

---

## 3.2 泛型

### 3.2.1 泛型基础

#### 知识点详解

**基本语法：**

```typescript
// 泛型函数
function identity<T>(arg: T): T {
    return arg;
}

let output1 = identity<string>("myString");
let output2 = identity("myString");  // 类型推断

// 泛型接口
interface GenericIdentityFn<T> {
    (arg: T): T;
}

let myIdentity: GenericIdentityFn<number> = identity;

// 泛型类
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = (x, y) => x + y;

// 泛型约束
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // OK
    return arg;
}

loggingIdentity("hello");     // OK
loggingIdentity([1, 2, 3]);   // OK
loggingIdentity({ length: 10, value: "hi" });  // OK
loggingIdentity(3);           // Error
```

**多个类型参数：**

```typescript
function swap<T, U>(tuple: [T, U]): [U, T] {
    return [tuple[1], tuple[0]];
}

swap([1, "hello"]);  // ["hello", 1]

// 约束对象属性
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

const obj = { a: 1, b: 2, c: 3 };
getProperty(obj, "a");  // OK
getProperty(obj, "z");  // Error
```

**泛型默认类型：**

```typescript
interface ApiResponse<T = any> {
    code: number;
    message: string;
    data: T;
}

const response: ApiResponse = {  // T 默认为 any
    code: 200,
    message: "success",
    data: {}
};

const userResponse: ApiResponse<{ name: string }> = {
    code: 200,
    message: "success",
    data: { name: "John" }
};
```

**泛型工具类型：**

```typescript
// Partial<T> - 所有属性可选
interface User {
    name: string;
    age: number;
}
type PartialUser = Partial<User>;
// { name?: string; age?: number; }

// Required<T> - 所有属性必需
type RequiredUser = Required<PartialUser>;
// { name: string; age: number; }

// Readonly<T> - 所有属性只读
type ReadonlyUser = Readonly<User>;
// { readonly name: string; readonly age: number; }

// Pick<T, K> - 选择部分属性
type UserName = Pick<User, "name">;
// { name: string; }

// Omit<T, K> - 排除部分属性
type UserWithoutAge = Omit<User, "age">;
// { name: string; }

// Record<K, T> - 构造对象类型
type UserMap = Record<string, User>;
// { [key: string]: User }

// Exclude<T, U> - 从联合类型中排除
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"

// Extract<T, U> - 提取联合类型中的类型
type T1 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"

// NonNullable<T> - 排除 null 和 undefined
type T2 = NonNullable<string | number | null | undefined>;  // string | number

// ReturnType<T> - 获取函数返回类型
function fn() { return { x: 10, y: 20 }; }
type FnReturnType = ReturnType<typeof fn>;  // { x: number; y: number; }

// Parameters<T> - 获取函数参数类型
type FnParams = Parameters<typeof fn>;  // []

// InstanceType<T> - 获取构造函数实例类型
class C { x = 0; y = 0; }
type CInstance = InstanceType<typeof C>;  // C
```

#### 真实面试题

**题目：实现一个 `DeepPartial<T>` 工具类型，使对象所有嵌套属性都变为可选。**

**满分答案：**

```typescript
// 基础版
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object 
        ? DeepPartial<T[P]> 
        : T[P];
};

// 完整版（处理数组、函数等特殊情况）
type DeepPartial<T> = T extends Function
    ? T
    : T extends Array<infer U>
    ? Array<DeepPartial<U>>
    : T extends object
    ? {
          [P in keyof T]?: DeepPartial<T[P]>;
      }
    : T;

// 测试
interface User {
    name: string;
    age: number;
    address: {
        city: string;
        street: string;
    };
    hobbies: string[];
}

const partialUser: DeepPartial<User> = {
    name: "John",
    address: {
        // city 和 street 都是可选的
    }
};
```

---

## 3.3 高级类型

### 3.3.1 类型守卫

#### 知识点详解

```typescript
// typeof 类型守卫
function printLength(str: string | number) {
    if (typeof str === "string") {
        console.log(str.length);  // str 被收窄为 string
    } else {
        console.log(str.toFixed(2));  // str 被收窄为 number
    }
}

// instanceof 类型守卫
class Dog {
    bark() {}
}
class Cat {
    meow() {}
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        animal.bark();
    } else {
        animal.meow();
    }
}

// in 操作符
interface Fish {
    swim: () => void;
}
interface Bird {
    fly: () => void;
}

function move(animal: Fish | Bird) {
    if ("swim" in animal) {
        animal.swim();
    } else {
        animal.fly();
    }
}

// 自定义类型守卫
interface Admin {
    name: string;
    permissions: string[];
}

interface User {
    name: string;
    email: string;
}

function isAdmin(user: Admin | User): user is Admin {
    return "permissions" in user;
}

function processUser(user: Admin | User) {
    if (isAdmin(user)) {
        console.log(user.permissions);  // user 是 Admin
    } else {
        console.log(user.email);  // user 是 User
    }
}

// 可辨识联合
type Shape =
    | { kind: "circle"; radius: number }
    | { kind: "square"; size: number }
    | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.size ** 2;
        case "rectangle":
            return shape.width * shape.height;
    }
}
```

### 3.3.2 条件类型

```typescript
// 基本语法
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// 分发条件类型
type ToArray<T> = T extends any ? T[] : never;
type StrArr = ToArray<string | number>;  // string[] | number[]

// infer 关键字
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

function fn(x: string): number {
    return parseInt(x);
}
type R = ReturnType<typeof fn>;  // number

// 提取 Promise 值类型
type Unpromise<T> = T extends Promise<infer U> ? U : T;
type Result = Unpromise<Promise<string>>;  // string

// 提取数组元素类型
type ElementType<T> = T extends (infer U)[] ? U : T;
type E = ElementType<string[]>;  // string

// 提取函数第一个参数类型
type FirstParam<T> = T extends (first: infer F, ...rest: any[]) => any
    ? F
    : never;
type P = FirstParam<(x: number, y: string) => void>;  // number
```

### 3.3.3 映射类型

```typescript
// 基本映射类型
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

type Optional<T> = {
    [P in keyof T]?: T[P];
};

// 修饰符
type Mutable<T> = {
    -readonly [P in keyof T]: T[P];  // 移除 readonly
};

type Required<T> = {
    [P in keyof T]-?: T[P];  // 移除可选
};

// Key 重映射（TS 4.1+）
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }

// 过滤属性
type OnlyStrings<T> = {
    [K in keyof T as T[K] extends string ? K : never]: T[K];
};

interface Mixed {
    name: string;
    age: number;
    active: boolean;
}

type StringProps = OnlyStrings<Mixed>;
// { name: string }
```

---

## 3.4 装饰器

### 3.4.1 类装饰器

```typescript
// 类装饰器
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}

@sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

// 类装饰器工厂
function classDecorator<T extends { new (...args: any[]): {} }>(
    constructor: T
) {
    return class extends constructor {
        newProperty = "new property";
        hello = "override";
    };
}

@classDecorator
class Greeter {
    property = "property";
    hello: string;
    constructor(m: string) {
        this.hello = m;
    }
}

console.log(new Greeter("world"));
// { property: 'property', hello: 'override', newProperty: 'new property' }
```

### 3.4.2 方法装饰器

```typescript
// 方法装饰器
function enumerable(value: boolean) {
    return function (
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
        descriptor.enumerable = value;
    };
}

class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}

// 方法装饰器 - 日志
function log(
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
) {
    const original = descriptor.value;

    descriptor.value = function (...args: any[]) {
        console.log(`Calling ${propertyKey} with arguments:`, args);
        const result = original.apply(this, args);
        console.log(`Result:`, result);
        return result;
    };

    return descriptor;
}

class Calculator {
    @log
    add(a: number, b: number) {
        return a + b;
    }
}
```

### 3.4.3 属性装饰器

```typescript
// 属性装饰器
function DefaultValue(value: any) {
    return function (target: any, propertyKey: string) {
        Object.defineProperty(target, propertyKey, {
            value,
            writable: true,
            enumerable: true,
            configurable: true,
        });
    };
}

class User {
    @DefaultValue("Guest")
    name: string;

    @DefaultValue(0)
    age: number;
}

const user = new User();
console.log(user.name);  // "Guest"
```

### 3.4.4 参数装饰器

```typescript
// 参数装饰器
function LogParameter(
    target: any,
    propertyKey: string,
    parameterIndex: number
) {
    console.log(`Parameter ${parameterIndex} of ${propertyKey}`);
}

class UserService {
    createUser(@LogParameter name: string, @LogParameter age: number) {
        // ...
    }
}
```

---

## 3.5 tsconfig.json 配置

```json
{
    "compilerOptions": {
        /* 基本选项 */
        "target": "ES2020",                    // 编译目标版本
        "module": "ESNext",                    // 模块系统
        "lib": ["ES2020", "DOM"],              // 包含的库声明文件
        "outDir": "./dist",                    // 输出目录
        "rootDir": "./src",                    // 源码目录
        
        /* 严格类型检查 */
        "strict": true,                        // 启用所有严格选项
        "noImplicitAny": true,                 // 禁止隐式 any
        "strictNullChecks": true,              // 严格空值检查
        "strictFunctionTypes": true,           // 严格函数类型
        "strictBindCallApply": true,           // 严格 bind/call/apply
        "strictPropertyInitialization": true,  // 严格属性初始化
        "noImplicitThis": true,                // 禁止隐式 this
        "alwaysStrict": true,                  // 始终严格模式
        
        /* 额外检查 */
        "noUnusedLocals": true,                // 未使用局部变量报错
        "noUnusedParameters": true,            // 未使用参数报错
        "noImplicitReturns": true,             // 每个分支都有返回值
        "noFallthroughCasesInSwitch": true,    // switch 穿透报错
        
        /* 模块解析 */
        "moduleResolution": "node",            // 模块解析策略
        "baseUrl": "./",                       // 基础路径
        "paths": {                             // 路径映射
            "@/*": ["src/*"]
        },
        "esModuleInterop": true,               // 允许 CommonJS 模块默认导入
        "allowSyntheticDefaultImports": true,  // 允许合成默认导入
        "resolveJsonModule": true,             // 解析 JSON 模块
        
        /* Source Map */
        "sourceMap": true,                     // 生成 sourceMap
        "declaration": true,                   // 生成 .d.ts
        "declarationMap": true,                // 生成声明文件的 sourceMap
        
        /* 其他 */
        "skipLibCheck": true,                  // 跳过库检查
        "forceConsistentCasingInFileNames": true  // 强制文件名大小写一致
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

---

## 3.6 TypeScript 实战案例

### 3.6.1 类型安全的 API 请求封装

```typescript
// 定义 API 响应类型
interface ApiResponse<T> {
    code: number;
    message: string;
    data: T;
}

// 定义 API 端点
interface ApiEndpoints {
    "/user": {
        GET: {
            response: User;
        };
        POST: {
            body: { name: string; email: string };
            response: User;
        };
    };
    "/posts": {
        GET: {
            query: { page?: number; limit?: number };
            response: Post[];
        };
    };
}

// 类型安全的请求函数
async function request<T extends keyof ApiEndpoints>(
    url: T,
    method: keyof ApiEndpoints[T],
    options?: {
        body?: ApiEndpoints[T][typeof method]["body"];
        query?: ApiEndpoints[T][typeof method]["query"];
    }
): Promise<ApiEndpoints[T][typeof method]["response"]> {
    const response = await fetch(url, {
        method,
        body: options?.body ? JSON.stringify(options.body) : undefined,
    });
    const json: ApiResponse<ApiEndpoints[T][typeof method]["response"]> =
        await response.json();
    return json.data;
}

// 使用
const user = await request("/user", "GET");
const newUser = await request("/user", "POST", {
    body: { name: "John", email: "john@example.com" },
});
```

### 3.6.2 类型安全的 EventEmitter

```typescript
type EventMap = {
    userCreated: { id: number; name: string };
    userDeleted: { id: number };
    message: string;
};

class TypedEventEmitter<T extends Record<string, any>> {
    private listeners: Partial<{ [K in keyof T]: ((data: T[K]) => void)[] }> =
        {};

    on<K extends keyof T>(event: K, listener: (data: T[K]) => void): void {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        this.listeners[event]!.push(listener);
    }

    emit<K extends keyof T>(event: K, data: T[K]): void {
        const eventListeners = this.listeners[event];
        if (eventListeners) {
            eventListeners.forEach((listener) => listener(data));
        }
    }

    off<K extends keyof T>(event: K, listener: (data: T[K]) => void): void {
        const eventListeners = this.listeners[event];
        if (eventListeners) {
            this.listeners[event] = eventListeners.filter(
                (l) => l !== listener
            );
        }
    }
}

// 使用
const emitter = new TypedEventEmitter<EventMap>();

emitter.on("userCreated", (user) => {
    console.log(`User created: ${user.name}`);
});

emitter.emit("userCreated", { id: 1, name: "John" });
emitter.emit("message", "Hello");
// emitter.emit("userCreated", { id: 1 });  // Error: missing 'name'
```
