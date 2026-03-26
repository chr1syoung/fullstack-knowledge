# 六、Node.js 详解

---

## 6.1 Node.js 基础

### 6.1.1 事件循环

#### 知识点详解

Node.js 的事件循环基于 libuv 库，与浏览器的事件循环有所不同。

**Node.js 事件循环阶段：**

```
   ┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← I/O 错误回调
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← 内部使用
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  ← 获取新的 I/O 事件
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  ← setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  ← socket.on('close', ...)
   └───────────────────────────┘
```

**各阶段说明：**

1. **timers**：执行 `setTimeout` 和 `setInterval` 的回调
2. **pending callbacks**：执行上一轮循环中延迟的 I/O 回调
3. **idle, prepare**：内部使用
4. **poll**：获取新的 I/O 事件，执行 I/O 回调
5. **check**：执行 `setImmediate` 的回调
6. **close callbacks**：执行关闭事件的回调

**微任务（在每个阶段之间执行）：**

```javascript
// process.nextTick（优先级最高）
process.nextTick(() => {
    console.log('nextTick');
});

// Promise.then（微任务）
Promise.resolve().then(() => {
    console.log('Promise');
});

// 执行顺序
console.log('start');

setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));

process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));

console.log('end');

// 输出：
// start
// end
// nextTick
// Promise
// setTimeout 或 setImmediate（顺序不确定）
```

**setTimeout vs setImmediate：**

```javascript
// 在 I/O 回调中，setImmediate 总是先于 setTimeout
const fs = require('fs');

fs.readFile(__filename, () => {
    setTimeout(() => console.log('setTimeout'), 0);
    setImmediate(() => console.log('setImmediate'));
});
// 输出：
// setImmediate
// setTimeout
```

### 6.1.2 模块系统

**CommonJS（CJS）：**

```javascript
// 导出
// math.js
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

module.exports = { add, subtract };
// 或
exports.add = add;
exports.subtract = subtract;

// 导入
const { add, subtract } = require('./math');
const math = require('./math');

// 特点
// - 同步加载
// - 运行时解析
// - 可以动态 require
const moduleName = 'math';
const module = require(`./${moduleName}`);
```

**ES Modules（ESM）：**

```javascript
// 导出
// math.mjs 或 package.json 中设置 "type": "module"
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export default function multiply(a, b) { return a * b; }

// 导入
import { add, subtract } from './math.js';
import multiply from './math.js';
import * as math from './math.js';

// 动态导入
const { add } = await import('./math.js');

// 特点
// - 异步加载
// - 编译时解析（静态分析）
// - 支持 Tree Shaking
```

**CJS vs ESM：**

| 特性 | CommonJS | ES Modules |
|------|----------|------------|
| 加载方式 | 同步 | 异步 |
| 解析时机 | 运行时 | 编译时 |
| 动态导入 | 支持 | 需要 import() |
| Tree Shaking | 不支持 | 支持 |
| 循环依赖 | 部分支持 | 更好支持 |
| 顶层 await | 不支持 | 支持 |

### 6.1.3 文件系统

```javascript
const fs = require('fs');
const fsPromises = require('fs/promises');
const path = require('path');

// 同步读取
const data = fs.readFileSync('file.txt', 'utf8');

// 异步回调
fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data);
});

// Promise 方式（推荐）
async function readFile() {
    const data = await fsPromises.readFile('file.txt', 'utf8');
    return data;
}

// 写入文件
await fsPromises.writeFile('output.txt', 'Hello World', 'utf8');

// 追加内容
await fsPromises.appendFile('log.txt', 'New log entry\n');

// 读取目录
const files = await fsPromises.readdir('./src');

// 创建目录
await fsPromises.mkdir('./new-dir', { recursive: true });

// 删除文件
await fsPromises.unlink('file.txt');

// 删除目录
await fsPromises.rmdir('./dir', { recursive: true });

// 文件信息
const stats = await fsPromises.stat('file.txt');
console.log(stats.size);
console.log(stats.isFile());
console.log(stats.isDirectory());

// 路径操作
path.join('/foo', 'bar', 'baz');  // '/foo/bar/baz'
path.resolve('foo', 'bar');       // 绝对路径
path.dirname('/foo/bar/baz.txt'); // '/foo/bar'
path.basename('/foo/bar/baz.txt'); // 'baz.txt'
path.extname('/foo/bar/baz.txt'); // '.txt'
path.parse('/foo/bar/baz.txt');   // { root, dir, base, ext, name }

// __dirname 和 __filename（CJS）
console.log(__dirname);   // 当前文件目录
console.log(__filename);  // 当前文件路径

// ESM 中的等价方式
import { fileURLToPath } from 'url';
import { dirname } from 'path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

### 6.1.4 Stream

```javascript
const fs = require('fs');
const { Transform } = require('stream');

// 读取流
const readStream = fs.createReadStream('large-file.txt', {
    encoding: 'utf8',
    highWaterMark: 64 * 1024  // 64KB 块大小
});

readStream.on('data', (chunk) => {
    console.log(`Received ${chunk.length} bytes`);
});

readStream.on('end', () => {
    console.log('读取完成');
});

readStream.on('error', (err) => {
    console.error(err);
});

// 写入流
const writeStream = fs.createWriteStream('output.txt');
writeStream.write('Hello ');
writeStream.write('World');
writeStream.end();

// 管道（pipe）
const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');
readStream.pipe(writeStream);

// 转换流
const { Transform } = require('stream');

const upperCaseTransform = new Transform({
    transform(chunk, encoding, callback) {
        this.push(chunk.toString().toUpperCase());
        callback();
    }
});

// 链式管道
fs.createReadStream('input.txt')
    .pipe(upperCaseTransform)
    .pipe(fs.createWriteStream('output.txt'));

// 使用 pipeline（推荐，自动处理错误）
const { pipeline } = require('stream/promises');

await pipeline(
    fs.createReadStream('input.txt'),
    upperCaseTransform,
    fs.createWriteStream('output.txt')
);
```

---

## 6.2 Express 框架

### 6.2.1 基础用法

```javascript
const express = require('express');
const app = express();

// 中间件
app.use(express.json());                    // 解析 JSON 请求体
app.use(express.urlencoded({ extended: true }));  // 解析 URL 编码
app.use(express.static('public'));          // 静态文件

// 路由
app.get('/', (req, res) => {
    res.send('Hello World');
});

app.get('/user/:id', (req, res) => {
    const { id } = req.params;
    const { page, limit } = req.query;
    res.json({ id, page, limit });
});

app.post('/user', (req, res) => {
    const { name, email } = req.body;
    // 创建用户...
    res.status(201).json({ message: '创建成功', user: { name, email } });
});

app.put('/user/:id', (req, res) => {
    // 更新用户...
    res.json({ message: '更新成功' });
});

app.delete('/user/:id', (req, res) => {
    // 删除用户...
    res.status(204).send();
});

// 启动服务器
app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### 6.2.2 中间件

```javascript
// 自定义中间件
function logger(req, res, next) {
    console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
    next();
}

function auth(req, res, next) {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: '未授权' });
    }
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        res.status(401).json({ error: 'Token 无效' });
    }
}

// 错误处理中间件（4个参数）
function errorHandler(err, req, res, next) {
    console.error(err.stack);
    
    if (err.name === 'ValidationError') {
        return res.status(400).json({ error: err.message });
    }
    
    res.status(500).json({ error: '服务器内部错误' });
}

// 使用中间件
app.use(logger);
app.use('/api', auth);  // 只对 /api 路径生效
app.use(errorHandler);  // 错误处理放在最后

// 路由级中间件
const router = express.Router();

router.use(auth);  // 路由级别的中间件

router.get('/profile', (req, res) => {
    res.json(req.user);
});

app.use('/api/user', router);
```

### 6.2.3 路由组织

```javascript
// routes/user.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');
const { auth, validate } = require('../middleware');

router.get('/', userController.getAll);
router.get('/:id', userController.getById);
router.post('/', auth, validate('createUser'), userController.create);
router.put('/:id', auth, userController.update);
router.delete('/:id', auth, userController.delete);

module.exports = router;

// app.js
const userRouter = require('./routes/user');
app.use('/api/users', userRouter);
```

---

## 6.3 数据库操作

### 6.3.1 MongoDB + Mongoose

```javascript
const mongoose = require('mongoose');

// 连接数据库
mongoose.connect('mongodb://localhost:27017/mydb', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

// 定义 Schema
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, '姓名是必填项'],
        trim: true,
        maxlength: [50, '姓名不能超过50个字符']
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        match: [/^\S+@\S+\.\S+$/, '邮箱格式不正确']
    },
    age: {
        type: Number,
        min: [0, '年龄不能为负数'],
        max: [150, '年龄不合理']
    },
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    createdAt: {
        type: Date,
        default: Date.now
    }
}, {
    timestamps: true  // 自动添加 createdAt 和 updatedAt
});

// 虚拟属性
userSchema.virtual('fullName').get(function() {
    return `${this.firstName} ${this.lastName}`;
});

// 实例方法
userSchema.methods.comparePassword = async function(password) {
    return bcrypt.compare(password, this.password);
};

// 静态方法
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email });
};

// 中间件（钩子）
userSchema.pre('save', async function(next) {
    if (this.isModified('password')) {
        this.password = await bcrypt.hash(this.password, 10);
    }
    next();
});

// 创建模型
const User = mongoose.model('User', userSchema);

// CRUD 操作
// 创建
const user = await User.create({ name: 'John', email: 'john@example.com' });
const user = new User({ name: 'John' });
await user.save();

// 查询
const users = await User.find({ role: 'admin' });
const user = await User.findById(id);
const user = await User.findOne({ email: 'john@example.com' });

// 查询条件
const users = await User.find({
    age: { $gte: 18, $lte: 60 },
    role: { $in: ['admin', 'moderator'] },
    name: { $regex: /john/i }
})
.select('name email')  // 选择字段
.sort({ createdAt: -1 })  // 排序
.skip(10)  // 跳过
.limit(10)  // 限制
.populate('posts');  // 关联查询

// 更新
await User.findByIdAndUpdate(id, { name: 'Jane' }, { new: true });
await User.updateMany({ role: 'user' }, { $set: { active: true } });

// 删除
await User.findByIdAndDelete(id);
await User.deleteMany({ active: false });

// 聚合
const result = await User.aggregate([
    { $match: { role: 'user' } },
    { $group: { _id: '$city', count: { $sum: 1 } } },
    { $sort: { count: -1 } },
    { $limit: 10 }
]);
```

### 6.3.2 MySQL + Sequelize

```javascript
const { Sequelize, DataTypes, Op } = require('sequelize');

// 连接数据库
const sequelize = new Sequelize('database', 'username', 'password', {
    host: 'localhost',
    dialect: 'mysql',
    pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000
    }
});

// 定义模型
const User = sequelize.define('User', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    name: {
        type: DataTypes.STRING(50),
        allowNull: false
    },
    email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: {
            isEmail: true
        }
    },
    age: {
        type: DataTypes.INTEGER,
        validate: {
            min: 0,
            max: 150
        }
    },
    role: {
        type: DataTypes.ENUM('user', 'admin'),
        defaultValue: 'user'
    }
}, {
    tableName: 'users',
    timestamps: true
});

// 关联
const Post = sequelize.define('Post', {
    title: DataTypes.STRING,
    content: DataTypes.TEXT
});

User.hasMany(Post, { foreignKey: 'userId' });
Post.belongsTo(User, { foreignKey: 'userId' });

// 同步数据库
await sequelize.sync({ alter: true });

// CRUD
// 创建
const user = await User.create({ name: 'John', email: 'john@example.com' });

// 查询
const users = await User.findAll({
    where: {
        age: { [Op.gte]: 18 },
        role: 'user'
    },
    attributes: ['id', 'name', 'email'],
    order: [['createdAt', 'DESC']],
    limit: 10,
    offset: 0,
    include: [{
        model: Post,
        attributes: ['id', 'title']
    }]
});

const user = await User.findByPk(1);
const user = await User.findOne({ where: { email: 'john@example.com' } });

// 更新
await User.update({ name: 'Jane' }, { where: { id: 1 } });
user.name = 'Jane';
await user.save();

// 删除
await User.destroy({ where: { id: 1 } });

// 事务
const t = await sequelize.transaction();
try {
    const user = await User.create({ name: 'John' }, { transaction: t });
    const post = await Post.create({ title: 'Hello', userId: user.id }, { transaction: t });
    await t.commit();
} catch (error) {
    await t.rollback();
    throw error;
}
```

---

## 6.4 API 设计

### 6.4.1 RESTful API

```javascript
// RESTful 设计原则
// GET    /users          - 获取用户列表
// GET    /users/:id      - 获取单个用户
// POST   /users          - 创建用户
// PUT    /users/:id      - 完整更新用户
// PATCH  /users/:id      - 部分更新用户
// DELETE /users/:id      - 删除用户

// 嵌套资源
// GET    /users/:id/posts     - 获取用户的文章列表
// POST   /users/:id/posts     - 为用户创建文章

// 版本控制
// /api/v1/users
// /api/v2/users

// 分页
// GET /users?page=1&limit=10
// GET /users?offset=0&limit=10
// GET /users?cursor=xxx&limit=10

// 过滤、排序、搜索
// GET /users?role=admin&sort=-createdAt&q=john

// 响应格式
{
    "code": 200,
    "message": "success",
    "data": {
        "users": [...],
        "pagination": {
            "total": 100,
            "page": 1,
            "limit": 10,
            "pages": 10
        }
    }
}

// 错误响应
{
    "code": 400,
    "message": "Validation Error",
    "errors": [
        { "field": "email", "message": "邮箱格式不正确" }
    ]
}
```

### 6.4.2 JWT 鉴权

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// 生成 Token
function generateTokens(userId) {
    const accessToken = jwt.sign(
        { userId, type: 'access' },
        process.env.JWT_SECRET,
        { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
        { userId, type: 'refresh' },
        process.env.JWT_REFRESH_SECRET,
        { expiresIn: '7d' }
    );
    
    return { accessToken, refreshToken };
}

// 验证 Token
function verifyToken(token, secret) {
    try {
        return jwt.verify(token, secret);
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            throw new Error('Token 已过期');
        }
        throw new Error('Token 无效');
    }
}

// 登录接口
app.post('/api/auth/login', async (req, res) => {
    const { email, password } = req.body;
    
    const user = await User.findOne({ email });
    if (!user) {
        return res.status(401).json({ error: '用户不存在' });
    }
    
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
        return res.status(401).json({ error: '密码错误' });
    }
    
    const { accessToken, refreshToken } = generateTokens(user.id);
    
    // 将 refreshToken 存入数据库
    await user.update({ refreshToken });
    
    res.json({
        accessToken,
        refreshToken,
        user: { id: user.id, name: user.name }
    });
});

// 刷新 Token
app.post('/api/auth/refresh', async (req, res) => {
    const { refreshToken } = req.body;
    
    const decoded = verifyToken(refreshToken, process.env.JWT_REFRESH_SECRET);
    const user = await User.findByPk(decoded.userId);
    
    if (!user || user.refreshToken !== refreshToken) {
        return res.status(401).json({ error: 'Refresh Token 无效' });
    }
    
    const tokens = generateTokens(user.id);
    await user.update({ refreshToken: tokens.refreshToken });
    
    res.json(tokens);
});

// 认证中间件
function authenticate(req, res, next) {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: '未提供 Token' });
    }
    
    const decoded = verifyToken(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
}
```

---

## 6.5 Node.js 性能优化

### 6.5.1 集群模式

```javascript
const cluster = require('cluster');
const os = require('os');
const express = require('express');

if (cluster.isMaster) {
    const numCPUs = os.cpus().length;
    
    console.log(`Master ${process.pid} is running`);
    
    // 创建工作进程
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    cluster.on('exit', (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died`);
        cluster.fork();  // 重启工作进程
    });
} else {
    const app = express();
    
    app.get('/', (req, res) => {
        res.send(`Hello from worker ${process.pid}`);
    });
    
    app.listen(3000, () => {
        console.log(`Worker ${process.pid} started`);
    });
}
```

### 6.5.2 缓存策略

```javascript
const redis = require('ioredis');
const redis = new Redis({
    host: 'localhost',
    port: 6379
});

// 缓存中间件
function cache(duration) {
    return async (req, res, next) => {
        const key = `cache:${req.url}`;
        
        const cached = await redis.get(key);
        if (cached) {
            return res.json(JSON.parse(cached));
        }
        
        // 拦截 res.json
        const originalJson = res.json.bind(res);
        res.json = async (data) => {
            await redis.setex(key, duration, JSON.stringify(data));
            return originalJson(data);
        };
        
        next();
    };
}

// 使用
app.get('/api/users', cache(60), async (req, res) => {
    const users = await User.findAll();
    res.json(users);
});

// 手动缓存
async function getUserById(id) {
    const cacheKey = `user:${id}`;
    
    // 先查缓存
    const cached = await redis.get(cacheKey);
    if (cached) {
        return JSON.parse(cached);
    }
    
    // 查数据库
    const user = await User.findByPk(id);
    
    // 写入缓存
    await redis.setex(cacheKey, 3600, JSON.stringify(user));
    
    return user;
}

// 缓存失效
async function updateUser(id, data) {
    await User.update(data, { where: { id } });
    await redis.del(`user:${id}`);  // 删除缓存
}
```

#### 真实面试题

**题目：Node.js 如何处理高并发？**

**满分答案：**

**Node.js 处理高并发的机制：**

1. **单线程 + 事件循环**
   - Node.js 使用单线程处理请求
   - 通过事件循环实现非阻塞 I/O
   - 不需要为每个请求创建线程，节省内存

2. **非阻塞 I/O**
   ```javascript
   // 阻塞（不好）
   const data = fs.readFileSync('file.txt');
   
   // 非阻塞（好）
   fs.readFile('file.txt', (err, data) => {
       // 处理数据
   });
   ```

3. **集群模式（Cluster）**
   - 利用多核 CPU
   - 每个 CPU 核心运行一个 Node.js 进程
   - 主进程负责负载均衡

4. **Worker Threads（CPU 密集型任务）**
   ```javascript
   const { Worker, isMainThread, parentPort } = require('worker_threads');
   
   if (isMainThread) {
       const worker = new Worker(__filename);
       worker.on('message', (result) => {
           console.log('Result:', result);
       });
       worker.postMessage({ data: 'heavy computation' });
   } else {
       parentPort.on('message', ({ data }) => {
           // 执行 CPU 密集型任务
           const result = heavyComputation(data);
           parentPort.postMessage(result);
       });
   }
   ```

5. **连接池**
   ```javascript
   // 数据库连接池
   const pool = mysql.createPool({
       connectionLimit: 10,
       host: 'localhost',
       database: 'mydb'
   });
   ```

6. **缓存**
   - Redis 缓存热点数据
   - 内存缓存（LRU Cache）
   - HTTP 缓存

7. **限流**
   ```javascript
   const rateLimit = require('express-rate-limit');
   
   const limiter = rateLimit({
       windowMs: 15 * 60 * 1000,  // 15 分钟
       max: 100  // 最多 100 次请求
   });
   
   app.use('/api/', limiter);
   ```

8. **负载均衡**
   - Nginx 反向代理
   - 多个 Node.js 实例
   - 轮询、最少连接等策略

---

### 6.x.x Node.js 实现命令行工具：统计目录代码行数 [重要]

#### 知识点详解

使用 Node.js 原生 API（无第三方依赖）实现一个代码行数统计 CLI：

**基础实现：**

```javascript
#!/usr/bin/env node
// 文件顶部加 shebang：#!/usr/bin/env node，让系统识别为可执行脚本

const fs = require('fs');
const path = require('path');

// 统计单个文件
function countFile(filePath, options = {}) {
  const { extensions = ['.js', '.jsx', '.ts', '.tsx', '.vue'] } = options;

  // 过滤文件类型
  const ext = path.extname(filePath).toLowerCase();
  if (!extensions.includes(ext)) return { lines: 0, code: 0, comment: 0, blank: 0 };

  const content = fs.readFileSync(filePath, 'utf-8');
  const lines = content.split('\n');

  let codeLines = 0;
  let commentLines = 0;
  let blankLines = 0;

  let inBlockComment = false;

  lines.forEach(line => {
    const trimmed = line.trim();

    // 空行
    if (trimmed === '') {
      blankLines++;
      return;
    }

    // 块注释开始
    if (trimmed.startsWith('/*')) inBlockComment = true;

    // 行注释
    if (trimmed.startsWith('//') || trimmed.startsWith('#')) {
      commentLines++;
    } else if (inBlockComment) {
      commentLines++;
      if (trimmed.endsWith('*/')) inBlockComment = false;
    } else {
      codeLines++;
    }
  });

  return { lines: lines.length, code: codeLines, comment: commentLines, blank: blankLines };
}

// 递归统计目录
function countDir(dirPath, options = {}) {
  const { exclude = ['node_modules', '.git', 'dist', 'build'] } = options;

  let total = { files: 0, lines: 0, code: 0, comment: 0, blank: 0 };

  function walk(currentPath) {
    const items = fs.readdirSync(currentPath);

    items.forEach(item => {
      const fullPath = path.join(currentPath, item);
      const stat = fs.statSync(fullPath);

      // 排除目录
      if (exclude.includes(item)) return;

      if (stat.isDirectory()) {
        walk(fullPath);
      } else if (stat.isFile()) {
        const result = countFile(fullPath, options);
        total.files++;
        total.lines += result.lines;
        total.code += result.code;
        total.comment += result.comment;
        total.blanks += result.blank;
      }
    });
  }

  walk(dirPath);
  return total;
}

// CLI 参数解析
function parseArgs() {
  const args = process.argv.slice(2);
  const options = {
    extensions: [],
    exclude: ['node_modules', '.git', 'dist', 'build'],
    target: process.cwd()
  };

  for (let i = 0; i < args.length; i++) {
    const arg = args[i];
    if (arg === '-e' || arg === '--ext') {
      options.extensions = args[++i].split(',').map(e => e.startsWith('.') ? e : '.' + e);
    } else if (arg === '-x' || arg === '--exclude') {
      options.exclude = args[++i].split(',');
    } else if (!arg.startsWith('-')) {
      options.target = path.resolve(arg);
    }
  }

  return options;
}

// 打印结果
function printReport(total, target) {
  console.log('\n📊 代码统计报告');
  console.log('─'.repeat(45));
  console.log(`  目录: ${target}`);
  console.log(`  文件数: ${total.files}`);
  console.log(`  总行数: ${total.lines}`);
  console.log(`  代码行: ${total.code}`);
  console.log(`  注释行: ${total.comment}`);
  console.log(`  空  行: ${total.blank || 0}`);
  console.log('─'.repeat(45));
}

// 主程序
const options = parseArgs();
const result = countDir(options.target, options);
printReport(result, options.target);
```

**进阶：文件详细列表：**

```javascript
// 显示每个文件的统计（文件大小排序）
function countDirWithFiles(dirPath, options = {}) {
  const { exclude = ['node_modules', '.git', 'dist', 'build'] } = options;
  const fileResults = [];

  function walk(currentPath) {
    const items = fs.readdirSync(currentPath);
    items.forEach(item => {
      const fullPath = path.join(currentPath, item);
      if (exclude.includes(item)) return;

      const stat = fs.statSync(fullPath);
      if (stat.isDirectory()) {
        walk(fullPath);
      } else {
        const ext = path.extname(fullPath).toLowerCase();
        if (options.extensions?.length && !options.extensions.includes(ext)) return;

        const result = countFile(fullPath, options);
        result.path = path.relative(dirPath, fullPath);
        fileResults.push(result);
      }
    });
  }

  walk(dirPath);
  return fileResults;
}
```

**使用方式：**

```bash
# 直接运行（需先 chmod +x count-lines.js）
chmod +x count-lines.js

# 统计当前目录
./count-lines.js

# 统计指定目录
./count-lines.js ./src

# 统计多种文件类型
./count-lines.js -e js,jsx,ts,tsx,vue

# 排除特定目录
./count-lines.js -x vendor,temp,.tmp

# 完整命令
./count-lines.js ./project --ext js,jsx,ts --exclude node_modules,dist
```

**package.json 注册命令：**

```json
{
  "bin": {
    "cloc": "./bin/count-lines.js"
  },
  "scripts": {
    "cloc": "node bin/count-lines.js"
  }
}
```

```bash
# 全局安装后可直接使用
npm install -g ./cloc-tool
cloc ./src --ext js,ts
```

**常见面试题：**

> **Q：fs.readFileSync vs fs.readFile vs fs.createReadStream 的区别？**
> - `readFileSync`：同步读取，整个文件读入内存，简单但会阻塞
> - `readFile`：异步读取，不阻塞，但大文件会占内存
> - `createReadStream`：流式读取，适合大文件（GB级别），边读边处理

> **Q：为什么用 shebang `#!/usr/bin/env node` 而不是 `#!/usr/bin/node`？**
> `env` 会自动在 PATH 中查找 node 的位置，兼容性更好（不同系统 node 路径可能不同）。

#### 面试加分项
- 了解 Node.js 流（Stream）处理大文件
- 了解 commander.js / yargs 等 CLI 参数解析库
- 了解进度条实现（`readline` + `ansi-escapes`）

---

## 6.6 SSE（Server-Sent Events）

```
浏览器 ──── HTTP 请求 ────→ 服务器
        ←── 单个 HTTP 连接 ←──
        ←── data: 消息1\n\n ───
        ←── data: 消息2\n\n ───
        ←── event: done\n\n ─────
```

**与 WebSocket 的区别：**

| 特性 | SSE | WebSocket |
|------|-----|-----------|
| 方向 | 单向（服务端→客户端） | 双向 |
| 协议 | HTTP | ws:// / wss:// |
| 自动重连 | ✅ 内置 | ❌ 需手动实现 |
| 二进制数据 | ❌ 仅文本 | ✅ 支持 |
| 兼容性 | 需要 polyfill | 现代浏览器都支持 |
| 简单性 | 简单 | 较复杂 |

**服务端实现（Node.js）：**

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    // 设置 SSE headers
    res.writeHead(200, {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
        'Access-Control-Allow-Origin': '*'
    });
    
    // 定时发送消息
    const interval = setInterval(() => {
        const data = {
            time: new Date().toISOString(),
            message: '服务器时间'
        };
        
        // 格式：data: 内容\n\n
        res.write(`data: ${JSON.stringify(data)}\n\n`);
    }, 1000);
    
    // 客户端断开时清理
    req.on('close', () => {
        clearInterval(interval);
        console.log('客户端断开');
    });
});

server.listen(3000, () => {
    console.log('SSE 服务运行在 http://localhost:3000');
});
```

**客户端接收（浏览器）：**

```javascript
// 使用 EventSource API
const eventSource = new EventSource('http://localhost:3000/events');

// 监听默认消息
eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('收到消息:', data);
};

// 监听自定义事件
eventSource.addEventListener('customEvent', (event) => {
    const data = JSON.parse(event.data);
    console.log('自定义事件:', data);
});

// 错误处理
eventSource.onerror = (error) => {
    console.error('SSE 错误:', error);
    // EventSource 会自动重连
};

// 关闭连接
eventSource.close();
```

**SSE 在 AI 对话场景的优势：**

```javascript
// AI 对话流式响应示例
async function createAIStream() {
    const response = await fetch('/api/ai/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: '你好' })
    });
    
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    
    const eventSource = new EventSource('/api/ai/stream');
    
    eventSource.onmessage = (event) => {
        const chunk = event.data;  // AI 返回的文字片段
        appendToChat(chunk);        // 实时显示
    };
}
```

**为什么 AI 对话适合用 SSE：**

1. **单向流式输出**：AI 主要从服务端推送文本片段到前端
2. **自动重连**：网络波动时自动重连，保证对话连续性
3. **简单轻量**：相比 WebSocket 协议更简单
4. **HTTP 兼容**：无需特殊协议，普通 HTTP 端口即可
5. **EventSource 简洁**：原生 API，无需手动解析流数据

**与 Fetch + ReadableStream 对比：**

```javascript
// 现代方案：Fetch + ReadableStream
const response = await fetch('/api/ai/chat', {
    method: 'POST',
    body: JSON.stringify({ message: '你好' })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    const chunk = decoder.decode(value);
    appendToChat(chunk);
}
```

| 方案 | 优点 | 缺点 |
|------|------|------|
| SSE | 简单、自动重连、HTTP 兼容 | 单向、仅文本 |
| Fetch + Stream | 双向支持、可处理二进制 | 需手动处理重连 |

#### 真实面试题

**题目：什么是 SSE？它在 AI 对话场景中有什么优势？**

**满分答案：**

**SSE（Server-Sent Events）：**
服务端向浏览器推送数据的轻量级方案，通过单个 HTTP 连接持续发送数据。

**AI 对话场景优势：**
1. **单向流式输出**：AI 输出是服务端不断推送文本片段
2. **自动重连**：网络断开后自动重连，用户无感知
3. **简单轻量**：基于 HTTP，无需 WebSocket 那样复杂的握手协议
4. **HTTP 兼容**：只需开放普通 HTTP 端口，防火墙/代理更易通过
5. **API 简洁**：EventSource 比手动处理流更简单

**适用场景：**
- AI 对话流式输出
- 实时通知推送
- 股票/比赛实时数据
- 日志实时查看

---

## 6.7 CommonJS和ES Module在Node中的区别

### 6.7.1 CommonJS和ES Module在Node中的区别

#### 知识点详解

在Node.js环境中，CommonJS（CJS）和ES Module（ESM）是两种主要的模块系统。

**Node.js中的启用方式：**

```javascript
// CommonJS（默认）
// 文件扩展名：.js 或 .cjs
const fs = require('fs');
module.exports = { foo: 'bar' };

// ES Module
// 方式1：package.json 中设置 "type": "module"
// 方式2：文件扩展名使用 .mjs
import fs from 'fs';
export const foo = 'bar';
```

**关键区别对比：**

| 特性 | CommonJS (CJS) | ES Module (ESM) |
|------|----------------|-----------------|
| 加载方式 | 同步（require） | 异步（import） |
| 解析时机 | 运行时 | 编译时 |
| 文件扩展名 | 可省略 | 必须指定 |
| __dirname/__filename | 存在 | 不存在（用import.meta.url） |
| 模块导出 | 值拷贝 | 值的引用 |
| 动态导入 | 支持 | 支持（import()函数） |
| 顶层await | 不支持 | 支持 |
| Tree Shaking | 不支持 | 支持 |

**互操作规则：**

```javascript
// ESM 可以导入 CJS（默认导入）
import cjsModule from './common.js';  // ✅ 可以

// CJS 不能静态导入 ESM
const esmModule = require('./esm.mjs');  // ❌ 报错

// CJS 可以通过动态 import() 加载 ESM
async function loadESM() {
    const esmModule = await import('./esm.mjs');  // ✅ 可以
}
```

**ESM中的__dirname替代方案：**

```javascript
// ESM 中没有 __dirname 和 __filename
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

#### 真实面试题

**题目：common.js和es6中模块引入的区别?**

**满分答案：**

在Node.js环境下，CommonJS和ES Module有以下关键差异：

**1. 加载机制不同**
- CJS的`require`是同步加载，适合服务端环境
- ESM的`import`是异步加载，编译时解析

**2. 文件扩展名要求**
- CJS可以省略扩展名，Node会自动查找(.js→.json→.node→index.js)
- ESM必须指定完整路径，包括扩展名

**3. 特殊变量差异**
- CJS有`__dirname`和`__filename`
- ESM中不存在，需通过`import.meta.url`转换获取

**4. 导出机制**
- CJS是值的拷贝，导出后修改原值不影响导入方
- ESM是值的引用，支持实时绑定

**5. 互操作规则**
- ESM可以静态导入CJS模块
- CJS不能静态导入ESM，只能通过动态`import()`函数加载

---

## 6.8 ES Module为什么必须加文件扩展名

### 6.8.1 ES Module为什么必须加文件扩展名

#### 知识点详解

**ESM规范要求完全指定URL：**

ES Module规范源自浏览器环境，浏览器加载模块时必须写完整路径：

```javascript
// 浏览器中必须这样写
import { foo } from './foo.js';  // ✅ 必须带.js
import { foo } from './foo';     // ❌ 浏览器无法解析
```

**Node.js遵循ESM规范：**

```javascript
// ESM 中必须带扩展名
import { foo } from './foo.js';  // ✅
import { foo } from './foo';     // ❌ Error: Cannot find module

// 对比 CJS 可以省略
const { foo } = require('./foo');  // ✅ Node自动补全
```

**CJS的查找算法 vs ESM的严格模式：**

```javascript
// CJS require查找顺序：
// 1. ./foo.js
// 2. ./foo.json
// 3. ./foo.node
// 4. ./foo/index.js

// ESM import要求：
// 必须精确指定 './foo.js'，不自动补全
```

**设计原因：**

1. **与浏览器保持一致**：ESM设计初衷是统一浏览器和Node.js的模块系统
2. **明确意图**：开发者明确知道导入的是什么类型的文件
3. **提升解析性能**：不需要文件系统查询，直接定位文件

**路径别名配置（package.json exports）：**

```json
{
  "exports": {
    ".": "./index.js",
    "./utils": "./src/utils/index.js",
    "./config/*": "./config/*.js"
  }
}
```

```javascript
// 使用别名
import { helper } from 'my-package/utils';
import dbConfig from 'my-package/config/database';
```

#### 真实面试题

**题目：为什么Node在使用ES Module时必须加上文件扩展名?**

**满分答案：**

**1. ESM规范要求**
ES Module规范要求模块路径必须是完整的URL，浏览器环境必须写完整路径（如`./foo.js`），Node.js遵循这一规范。

**2. 与CJS查找算法对比**
- CJS的`require`有自动查找算法：`.js` → `.json` → `.node` → `目录/index.js`
- ESM不支持自动补全扩展名，必须显式指定

**3. 设计原因**
- **浏览器一致性**：ESM设计目标是统一浏览器和Node.js模块系统
- **明确意图**：显式写扩展名让代码意图更清晰
- **性能优化**：省去文件系统查询步骤，直接定位文件

**4. 解决方案**
可通过`package.json`的`exports`字段配置模块路径别名，简化导入路径。

---

## 6.9 浏览器和Node事件循环区别

### 6.9.1 浏览器和Node事件循环区别

#### 知识点详解

**共同点：**
- 都基于事件驱动架构
- 都有宏任务（Macrotask）和微任务（Microtask）队列
- 微任务优先级高于宏任务

**核心区别：**

| 特性 | 浏览器 | Node.js |
|------|--------|---------|
| 宏任务来源 | DOM事件、定时器、网络请求 | 文件I/O、定时器、setImmediate |
| 执行模型 | 两个队列（宏任务+微任务） | 6个阶段循环 |
| 微任务执行时机 | 每个宏任务后 | 每个阶段结束后 |
| 特有API | requestAnimationFrame | setImmediate、process.nextTick |

**浏览器事件循环：**

```
┌─────────────────────────────────────────┐
│              宏任务队列                  │
│  (setTimeout, DOM事件, AJAX, I/O)       │
└─────────────┬───────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────┐
│              微任务队列                  │
│  (Promise.then, MutationObserver)       │
└─────────────────────────────────────────┘
```

**Node.js事件循环（6个阶段）：**

```
   ┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout/setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  ← 延迟的I/O回调
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  ← 内部使用
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  ← 获取I/O事件
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  ← setImmediate
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  ← 关闭事件回调
   └───────────────────────────┘
```

**微任务执行时机差异：**

```javascript
// 浏览器：每个宏任务后执行微任务
setTimeout(() => {
    console.log('timer1');
    Promise.resolve().then(() => console.log('promise1'));
}, 0);
setTimeout(() => console.log('timer2'), 0);
// 浏览器输出: timer1 → promise1 → timer2

// Node < 11：当前阶段的所有任务完成后才执行微任务
// Node 11+：与浏览器行为一致，每个任务后执行微任务
```

**Node特有API：**

```javascript
// process.nextTick - 优先级最高的微任务
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
setImmediate(() => console.log('setImmediate'));
setTimeout(() => console.log('setTimeout'), 0);
// 输出顺序: nextTick → promise → setTimeout/setImmediate
```

**Node 11+的变化：**

```javascript
// Node 11之前：timer阶段全部执行完才执行微任务
// Node 11+：每个timer回调后立即执行微任务（与浏览器一致）

setTimeout(() => {
    console.log('timer1');
    Promise.resolve().then(() => console.log('promise1'));
}, 0);

setTimeout(() => console.log('timer2'), 0);

// Node < 11: timer1 → timer2 → promise1
// Node 11+:   timer1 → promise1 → timer2
```

#### 真实面试题

**题目：浏览器和Node中的事件循环有什么区别?**

**满分答案：**

**核心区别表格：**

| 特性 | 浏览器 | Node.js |
|------|--------|---------|
| 宏任务来源 | DOM事件、定时器、网络请求 | 文件I/O、定时器、setImmediate |
| 执行模型 | 两个队列（宏+微） | 6个阶段循环 |
| 微任务执行时机 | 每个宏任务后 | 每个阶段结束后 |
| 特有API | requestAnimationFrame | setImmediate、process.nextTick |

**Node的6个阶段：**
1. **timers**：执行setTimeout/setInterval回调
2. **pending callbacks**：执行延迟的I/O回调
3. **idle/prepare**：内部使用
4. **poll**：获取I/O事件，执行I/O回调
5. **check**：执行setImmediate回调
6. **close callbacks**：执行关闭事件回调

**Node 11+的变化：**
修改了微任务执行时机，与浏览器行为更接近——在每个回调后立即执行微任务，而不是等当前阶段全部完成。

**Node特有机制：**
- `process.nextTick`优先级高于Promise微任务
- `setImmediate`只在check阶段执行
- `setTimeout`和`setImmediate`执行顺序取决于执行上下文

---

## 6.10 Node性能监控与优化

### 6.10.1 Node性能监控与优化

#### 知识点详解

**监控工具链：**

```javascript
// 1. 内置性能API
const process = require('process');
const perfHooks = require('perf_hooks');

// 内存使用
console.log(process.memoryUsage());
// {
//   rss: 4935680,      // 常驻内存
//   heapTotal: 1826816, // 堆内存总量
//   heapUsed: 650472,   // 已使用堆内存
//   external: 49879     // 外部内存
// }

// CPU使用
console.log(process.cpuUsage());

// Performance API
const { performance } = perfHooks;
const start = performance.now();
// ... 执行代码
const duration = performance.now() - start;
```

**APM工具：**

```javascript
// New Relic / Datadog / 阿里云ARMS
// 自动监控：响应时间、错误率、吞吐量、依赖调用

// 日志工具
const pino = require('pino');
const logger = pino({
    level: 'info',
    transport: {
        target: 'pino-pretty'
    }
});
logger.info({ userId: 123 }, '用户登录');
```

**进程管理（PM2）：**

```bash
# 启动集群模式
pm2 start app.js -i max  # 使用所有CPU核心

# 常用命令
pm2 logs      # 查看日志
pm2 monit     # 实时监控
pm2 reload all # 零停机重启
```

**性能分析工具：**

```bash
# Chrome DevTools 调试
node --inspect app.js
node --inspect-brk app.js  # 在第一行断点

# clinic.js 性能分析
clinic doctor -- node app.js    # 诊断问题
clinic bubbleprof -- node app.js # 异步流程分析
clinic flame -- node app.js     # 生成火焰图
```

**优化策略：**

```javascript
// 1. 集群模式利用多核
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
    os.cpus().forEach(() => cluster.fork());
} else {
    require('./app');
}

// 2. 内存泄漏排查
const heapdump = require('heapdump');
heapdump.writeSnapshot('./heap-' + Date.now() + '.heapsnapshot');

// 3. 流处理大文件
const fs = require('fs');
fs.createReadStream('large-file.txt')
    .pipe(fs.createWriteStream('output.txt'));

// 4. 缓存策略
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 });

async function getData(key) {
    let data = cache.get(key);
    if (!data) {
        data = await fetchFromDB(key);
        cache.set(key, data);
    }
    return data;
}

// 5. 数据库连接池
const mysql = require('mysql2');
const pool = mysql.createPool({
    connectionLimit: 10,
    host: 'localhost',
    database: 'mydb'
});

// 6. 限流
const rateLimit = require('express-rate-limit');
app.use(rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100
}));
```

#### 真实面试题

**题目：Node性能如何进行监控以及优化?**

**满分答案：**

**监控工具链：**
1. **内置工具**：`process.memoryUsage()`（堆内存）、`process.cpuUsage()`、`performance API`
2. **APM工具**：New Relic、Datadog、阿里云ARMS（自动监控响应时间、错误率、吞吐量）
3. **日志工具**：winston、pino（结构化日志，便于分析）
4. **进程管理**：PM2（集群模式、自动重启、日志聚合）
5. **性能分析**：Node `--inspect` + Chrome DevTools、clinic.js（火焰图、诊断报告）

**优化策略分类：**
1. **多核利用**：cluster模块或PM2集群模式
2. **内存优化**：排查内存泄漏（heapdump + Chrome DevTools）、流处理大文件
3. **I/O优化**：异步非阻塞、数据库连接池、慢查询优化
4. **缓存策略**：Redis缓存、内存缓存（LRU）
5. **稳定性**：限流（rate limiting）、熔断机制

---

## 6.11 分页功能设计

### 6.11.1 分页功能设计

#### 知识点详解

**两种分页方案对比：**

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| Offset/Limit | 简单、可跳页 | 深分页性能差 | 数据量小、需要跳页 |
| Cursor/游标 | 性能好、无深分页问题 | 不能跳页 | 数据量大、瀑布流 |

**Offset/Limit方案：**

```javascript
// API设计
// GET /api/items?page=1&pageSize=20&sort=createdAt&order=desc

// 后端实现
async function getItems(page = 1, pageSize = 20) {
    const offset = (page - 1) * pageSize;
    
    const [items, total] = await Promise.all([
        db.query('SELECT * FROM items ORDER BY created_at DESC LIMIT ? OFFSET ?', 
            [pageSize, offset]),
        db.query('SELECT COUNT(*) as total FROM items')
    ]);
    
    return {
        list: items,
        total: total[0].total,
        page,
        pageSize,
        totalPages: Math.ceil(total[0].total / pageSize)
    };
}
```

**深分页优化：**

```javascript
// 问题：OFFSET 1000000 会扫描100万行
// 优化1：使用覆盖索引 + 子查询
async function getItemsOptimized(lastId, pageSize = 20) {
    // 基于最后一条记录的id查询
    const items = await db.query(
        'SELECT * FROM items WHERE id > ? ORDER BY id LIMIT ?',
        [lastId, pageSize]
    );
    return { list: items, nextCursor: items[items.length - 1]?.id };
}

// 优化2：延迟关联
SELECT * FROM items 
INNER JOIN (
    SELECT id FROM items ORDER BY created_at LIMIT 100000, 20
) AS tmp USING(id);
```

**Cursor/游标方案：**

```javascript
// API设计
// GET /api/items?cursor=xxx&limit=20

async function getItemsByCursor(cursor, limit = 20) {
    const where = cursor ? 'WHERE created_at < ?' : '';
    const params = cursor ? [decodeCursor(cursor), limit] : [limit];
    
    const items = await db.query(
        `SELECT * FROM items ${where} ORDER BY created_at DESC LIMIT ?`,
        params
    );
    
    return {
        list: items,
        nextCursor: items.length === limit 
            ? encodeCursor(items[items.length - 1].created_at) 
            : null,
        hasMore: items.length === limit
    };
}

function encodeCursor(value) {
    return Buffer.from(value).toString('base64');
}
```

**前端分页组件设计：**

```javascript
// URL同步（便于分享和刷新）
function Pagination({ page, totalPages, onChange }) {
    useEffect(() => {
        const params = new URLSearchParams(location.search);
        params.set('page', page);
        navigate(`?${params.toString()}`);
    }, [page]);
    
    return (
        <div className="pagination">
            <button disabled={page <= 1} onClick={() => onChange(page - 1)}>上一页</button>
            <span>{page} / {totalPages}</span>
            <button disabled={page >= totalPages} onClick={() => onChange(page + 1)}>下一页</button>
        </div>
    );
}

// 滚动加载（IntersectionObserver）
function InfiniteScroll({ onLoadMore, hasMore }) {
    const observerRef = useRef();
    
    useEffect(() => {
        const observer = new IntersectionObserver(entries => {
            if (entries[0].isIntersecting && hasMore) {
                onLoadMore();
            }
        });
        observer.observe(observerRef.current);
        return () => observer.disconnect();
    }, [hasMore]);
    
    return <div ref={observerRef}>加载中...</div>;
}
```

#### 真实面试题

**题目：如果让你来设计一个分页功能，你会怎么设计?前后端如何交互?**

**满分答案：**

**两种分页方案：**

| 方案 | 适用场景 | 特点 |
|------|----------|------|
| Offset/Limit | 数据量小、需要跳页 | 简单，但深分页性能差 |
| Cursor/游标 | 数据量大、瀑布流 | 性能好，但不能跳页 |

**API设计：**
```
GET /api/items?page=1&pageSize=20&sort=createdAt&order=desc
```
返回结构：`{list, total, page, pageSize, totalPages}`

**深分页优化：**
- 使用cursor-based方案（基于最后一条记录的id）
- 使用子查询/覆盖索引避免全表扫描

**前端设计：**
- 分页组件：当前页/总页数/页码按钮/跳页输入
- URL同步：query参数同步，支持刷新和分享
- 滚动加载：IntersectionObserver实现无限滚动
- 缓存策略：当前页数据缓存、total缓存

---

## 6.12 文件上传实现

### 6.12.1 文件上传实现

#### 知识点详解

**前端上传方案：**

```javascript
// 1. 基础上传
const input = document.querySelector('input[type="file"]');
input.addEventListener('change', async (e) => {
    const file = e.target.files[0];
    const formData = new FormData();
    formData.append('file', file);
    
    await fetch('/api/upload', {
        method: 'POST',
        body: formData
    });
});

// 2. 拖拽上传
const dropZone = document.getElementById('drop-zone');
dropZone.addEventListener('dragover', (e) => {
    e.preventDefault();
    dropZone.classList.add('drag-over');
});
dropZone.addEventListener('drop', (e) => {
    e.preventDefault();
    const files = e.dataTransfer.files;
    uploadFiles(files);
});
```

**大文件分片上传：**

```javascript
// 分片上传
async function uploadLargeFile(file) {
    const chunkSize = 1024 * 1024; // 1MB
    const chunks = Math.ceil(file.size / chunkSize);
    const fileHash = await calculateHash(file); // MD5/SHA-256
    
    for (let i = 0; i < chunks; i++) {
        const start = i * chunkSize;
        const end = Math.min(start + chunkSize, file.size);
        const chunk = file.slice(start, end);
        
        const formData = new FormData();
        formData.append('chunk', chunk);
        formData.append('hash', fileHash);
        formData.append('index', i);
        formData.append('total', chunks);
        
        await fetch('/api/upload/chunk', {
            method: 'POST',
            body: formData
        });
    }
    
    // 通知服务端合并
    await fetch('/api/upload/merge', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ hash: fileHash, total: chunks, filename: file.name })
    });
}

// 计算文件hash
async function calculateHash(file) {
    const buffer = await file.arrayBuffer();
    const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
    return Array.from(new Uint8Array(hashBuffer))
        .map(b => b.toString(16).padStart(2, '0'))
        .join('');
}
```

**秒传和断点续传：**

```javascript
// 秒传：计算hash后查询是否已存在
async function quickUpload(file) {
    const hash = await calculateHash(file);
    
    // 查询文件是否已存在
    const res = await fetch(`/api/upload/check?hash=${hash}`);
    const { exists, url } = await res.json();
    
    if (exists) {
        console.log('秒传成功');
        return url; // 直接返回已有文件的URL
    }
    
    // 不存在则正常上传
    return uploadLargeFile(file);
}

// 断点续传：查询已上传的分片
async function resumeUpload(file) {
    const hash = await calculateHash(file);
    const chunkSize = 1024 * 1024;
    
    // 获取已上传的分片索引
    const res = await fetch(`/api/upload/progress?hash=${hash}`);
    const { uploadedChunks } = await res.json();
    
    const chunks = Math.ceil(file.size / chunkSize);
    for (let i = 0; i < chunks; i++) {
        if (uploadedChunks.includes(i)) {
            console.log(`分片${i}已存在，跳过`);
            continue;
        }
        // 上传未上传的分片
        await uploadChunk(file, i, chunkSize, hash);
    }
}
```

**上传进度：**

```javascript
// XMLHttpRequest进度
const xhr = new XMLHttpRequest();
xhr.upload.onprogress = (e) => {
    if (e.lengthComputable) {
        const percent = (e.loaded / e.total) * 100;
        console.log(`上传进度: ${percent}%`);
    }
};
xhr.open('POST', '/api/upload');
xhr.send(formData);

// Axios进度
await axios.post('/api/upload', formData, {
    onUploadProgress: (progressEvent) => {
        const percent = (progressEvent.loaded / progressEvent.total) * 100;
        console.log(`上传进度: ${percent}%`);
    }
});
```

**后端处理（multer）：**

```javascript
const multer = require('multer');
const upload = multer({ 
    dest: 'uploads/',
    limits: { fileSize: 100 * 1024 * 1024 }, // 100MB限制
    fileFilter: (req, file, cb) => {
        // 限制文件类型
        if (file.mimetype.startsWith('image/')) {
            cb(null, true);
        } else {
            cb(new Error('只允许上传图片'));
        }
    }
});

// 单文件上传
app.post('/api/upload', upload.single('file'), (req, res) => {
    res.json({ url: `/uploads/${req.file.filename}` });
});

// 分片合并
const fs = require('fs');
const path = require('path');

app.post('/api/upload/merge', async (req, res) => {
    const { hash, total, filename } = req.body;
    const chunkDir = path.join('uploads', hash);
    const filePath = path.join('uploads', filename);
    
    // 按顺序合并分片
    const writeStream = fs.createWriteStream(filePath);
    for (let i = 0; i < total; i++) {
        const chunkPath = path.join(chunkDir, `${i}`);
        const chunk = fs.readFileSync(chunkPath);
        writeStream.write(chunk);
        fs.unlinkSync(chunkPath); // 删除分片
    }
    writeStream.end();
    fs.rmdirSync(chunkDir);
    
    res.json({ url: `/uploads/${filename}` });
});
```

#### 真实面试题

**题目：如何实现文件上传?说说你的思路**

**满分答案：**

**前端方案：**
1. **基础上传**：`input[type=file]` + `FormData`
2. **拖拽上传**：监听`dragover`/`drop`事件
3. **大文件分片**：将文件切分为多个chunk，并行上传，最后合并
4. **秒传**：计算文件hash → 服务端查询是否已存在
5. **断点续传**：记录已上传的chunk，中断后从断点继续
6. **上传进度**：`XMLHttpRequest.upload.onprogress`或axios的`onUploadProgress`

**后端处理：**
1. **multer**：处理`multipart/form-data`
2. **分片合并**：使用流合并分片文件
3. **限制**：文件大小、文件类型校验

**分片上传流程：**
1. 前端计算文件hash
2. 将文件切分为固定大小的chunk
3. 并行/串行上传各分片
4. 上传完成后通知服务端合并
5. 服务端按顺序合并分片，返回文件URL

---

## 6.13 JWT鉴权机制

### 6.13.1 JWT鉴权机制

#### 知识点详解

**JWT结构：**

```
Header.Payload.Signature

// Header（Base64）
{
    "alg": "HS256",
    "typ": "JWT"
}

// Payload（Base64）
{
    "userId": "123",
    "role": "admin",
    "exp": 1699999999  // 过期时间
}

// Signature（签名，防止篡改）
HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret
)
```

**完整鉴权流程：**

```javascript
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// 1. 登录生成JWT
app.post('/api/login', async (req, res) => {
    const { email, password } = req.body;
    
    // 验证用户
    const user = await User.findOne({ email });
    const valid = await bcrypt.compare(password, user.password);
    
    if (!valid) {
        return res.status(401).json({ error: '密码错误' });
    }
    
    // 生成Token
    const accessToken = jwt.sign(
        { userId: user.id, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: '15m' }  // 15分钟有效期
    );
    
    const refreshToken = jwt.sign(
        { userId: user.id, type: 'refresh' },
        process.env.JWT_REFRESH_SECRET,
        { expiresIn: '7d' }  // 7天有效期
    );
    
    // 存储refreshToken（数据库或Redis）
    await user.update({ refreshToken });
    
    res.json({ accessToken, refreshToken });
});

// 2. 客户端存储
// localStorage.setItem('token', accessToken);
// 或 cookie（httpOnly更安全）

// 3. 请求携带Token
// Authorization: Bearer xxx.yyy.zzz

// 4. 服务端验证
function authMiddleware(req, res, next) {
    const authHeader = req.headers.authorization;
    if (!authHeader?.startsWith('Bearer ')) {
        return res.status(401).json({ error: '未提供Token' });
    }
    
    const token = authHeader.substring(7);
    
    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.userId = decoded.userId;
        req.userRole = decoded.role;
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ error: 'Token已过期', code: 'TOKEN_EXPIRED' });
        }
        res.status(401).json({ error: 'Token无效' });
    }
}

// 5. 刷新Token
app.post('/api/refresh', async (req, res) => {
    const { refreshToken } = req.body;
    
    try {
        const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
        const user = await User.findByPk(decoded.userId);
        
        // 验证refreshToken是否匹配
        if (user.refreshToken !== refreshToken) {
            return res.status(401).json({ error: 'Refresh Token无效' });
        }
        
        // 生成新的accessToken
        const newAccessToken = jwt.sign(
            { userId: user.id, role: user.role },
            process.env.JWT_SECRET,
            { expiresIn: '15m' }
        );
        
        res.json({ accessToken: newAccessToken });
    } catch (error) {
        res.status(401).json({ error: 'Refresh Token无效' });
    }
});
```

**JWT优缺点：**

| 优点 | 缺点 |
|------|------|
| 无状态（不需要服务端存储session） | 不能主动失效（除非维护黑名单） |
| 适合分布式系统 | Token体积比session id大 |
| 跨域友好 | Payload可解码，不能放敏感信息 |
| 减少数据库查询 | 需要处理Token刷新机制 |

**安全实践：**

```javascript
// 1. 设置较短的过期时间 + refresh token
// 2. 使用HTTPS传输
// 3. 不在JWT中放敏感信息（密码、手机号等）
// 4. 使用httpOnly cookie存储（防XSS）
// 5. 实现Token黑名单（登出时失效）

// Token黑名单（Redis实现）
const redis = require('redis');
const client = redis.createClient();

async function logout(token) {
    const decoded = jwt.decode(token);
    const ttl = decoded.exp - Math.floor(Date.now() / 1000);
    await client.setEx(`blacklist:${token}`, ttl, '1');
}

async function isBlacklisted(token) {
    return await client.get(`blacklist:${token}`);
}
```

**与Session方案对比：**

| 特性 | JWT | Session |
|------|-----|---------|
| 存储位置 | 客户端 | 服务端（内存/数据库/Redis） |
| 状态管理 | 无状态 | 有状态 |
| 扩展性 | 适合分布式 | 需要共享session |
| 安全性 | Payload可解码 | sessionId不可伪造 |
| 主动失效 | 需黑名单 | 直接删除 |
| 性能 | 无服务端查询 | 可能需要DB查询 |

#### 真实面试题

**题目：如何实现JWT鉴权机制?说说你的思路**

**满分答案：**

**JWT结构：**
- Header（算法+类型，Base64编码）
- Payload（用户信息+过期时间，Base64编码）
- Signature（签名，防止篡改）

**完整鉴权流程：**
1. **登录**：用户提交账号密码 → 服务端验证 → 生成JWT返回
2. **存储**：客户端存储JWT（localStorage或httpOnly cookie）
3. **请求**：每次请求携带`Authorization: Bearer xxx`
4. **验证**：服务端验证签名和过期时间

**优缺点：**
- **优点**：无状态、适合分布式、减少数据库查询
- **缺点**：不能主动失效（需黑名单）、Token体积较大

**安全实践：**
1. 设置较短过期时间（15分钟）+ refresh token（7天）
2. 使用HTTPS传输
3. 不在JWT中放敏感信息（密码、手机号等）
4. 使用httpOnly cookie存储（防止XSS攻击）
5. 实现Token黑名单机制（登出时失效）

**与Session方案对比：**
- JWT：无状态、分布式友好、需主动失效机制
- Session：有状态、易失效、需session存储

---

## 6.14 中间件概念与封装

### 6.14.1 中间件概念与封装

#### 知识点详解

**中间件概念：**

中间件是接收请求(req)、响应(res)、下一个中间件(next)的函数，形成处理管道（洋葱模型）。

```
请求 → 中间件A → 中间件B → 路由处理 → 中间件B → 中间件A → 响应
           next()           执行       next()返回
```

**Express中间件：**

```javascript
// 普通中间件
function logger(req, res, next) {
    console.log(`${req.method} ${req.url} - ${new Date()}`);
    next();  // 必须调用next()继续执行
}

// 异步中间件（注意错误处理）
async function asyncMiddleware(req, res, next) {
    try {
        const data = await fetchData();
        req.data = data;
        next();
    } catch (error) {
        next(error);  // 传递错误给错误处理中间件
    }
}

// 条件中间件
app.use('/api', authMiddleware);  // 只对/api路径生效

// 多个中间件（按顺序执行）
app.get('/user', 
    validateInput,
    fetchUser,
    sendResponse
);
```

**洋葱模型图解：**

```javascript
// 中间件按顺序形成洋葱模型
app.use(async (req, res, next) => {
    console.log('1. 进入请求处理');
    await next();
    console.log('5. 响应前的处理');
});

app.use(async (req, res, next) => {
    console.log('2. 第二层中间件');
    await next();
    console.log('4. 返回第二层');
});

app.get('/', (req, res) => {
    console.log('3. 路由处理');
    res.send('Hello');
});

// 输出顺序：1 → 2 → 3 → 4 → 5
```

**封装中间件：**

```javascript
// 1. 基本封装：返回中间件函数
function createAuthMiddleware(options) {
    return (req, res, next) => {
        const { required = true, roles = [] } = options;
        
        const token = req.headers.authorization;
        if (!token && required) {
            return res.status(401).json({ error: '未授权' });
        }
        
        if (roles.length && !roles.includes(req.userRole)) {
            return res.status(403).json({ error: '权限不足' });
        }
        
        next();
    };
}

// 使用
app.get('/admin', createAuthMiddleware({ roles: ['admin'] }), (req, res) => {
    res.send('Admin Panel');
});

// 2. 带配置的中间件
function validate(schema) {
    return (req, res, next) => {
        const { error, value } = schema.validate(req.body);
        if (error) {
            return res.status(400).json({ error: error.details[0].message });
        }
        req.validatedBody = value;
        next();
    };
}

// 3. 错误处理中间件（4个参数）
function errorHandler(err, req, res, next) {
    console.error(err.stack);
    
    if (err.name === 'ValidationError') {
        return res.status(400).json({ error: err.message });
    }
    
    if (err.name === 'UnauthorizedError') {
        return res.status(401).json({ error: '认证失败' });
    }
    
    res.status(500).json({ error: '服务器内部错误' });
}
```

**4种中间件类型：**

```javascript
// 1. 应用级中间件（绑定到app实例）
app.use(AuthMiddleware);           // 全局
app.get('/path', AuthMiddleware);  // 路由级

// 2. 路由级中间件（绑定到Router）
const router = express.Router();
router.use(rateLimiter);
router.get('/users', userController.list);

// 3. 错误处理中间件（必须有4个参数）
app.use((err, req, res, next) => {
    // 错误处理逻辑
});

// 4. 第三方中间件
app.use(cookieParser());           // 解析cookie
app.use(cors({ origin: '*' }));    // 跨域
app.use(morgan('combined'));        // HTTP日志
app.use(helmet());                 // 安全头
app.use(compress());               // 压缩
```

#### 真实面试题

**题目：说说对中间件概念的理解，如何封装Node中间件?**

**满分答案：**

**中间件概念：**
中间件是形成处理管道的函数，接收请求(req)、响应(res)、下一个中间件(next)，形成洋葱模型。

**洋葱模型：**
```
请求 → 中1 → 中2 → 路由 → 中2 → 中1 → 响应
```

**封装中间件步骤：**
1. 定义返回中间件的函数
2. 可传入配置参数
3. 处理错误（try/catch或next(error)）

**4种中间件类型：**
1. **应用级**：app.use()、app.get()等
2. **路由级**：router.use()
3. **错误处理**：4个参数(err, req, res, next)
4. **第三方**：cookie-parser、cors、morgan等

---

## 6.15 Node文件查找策略

### 6.15.1 Node文件查找策略

#### 知识点详解

**require查找算法（4级优先级）：**

```javascript
// Node.js的require()按以下顺序查找模块：

// 1. 核心模块（fs, path, http, crypto等）
const fs = require('fs');           // 直接返回
const path = require('path');       // 直接返回

// 2. 文件模块（以/、./、../开头）
require('./foo');    // 精确路径 → .js → .json → .node → 目录/index.js
require('../utils');
require('/absolute/path');

// 3. node_modules（从当前目录向上逐级查找）
// 当前目录/node_modules → 上级/node_modules → ... → 根目录/node_modules
require('express');  // 查找 ./node_modules/express → ../node_modules/express → ...

// 4. 路径映射（package.json的exports/imports）
// "exports": { ".": "./index.js", "./utils": "./lib/utils.js" }
```

**查找顺序详解：**

```javascript
// 对于 require('./foo')：
// 1. 精确文件：./foo.js
// 2. 精确文件：./foo.json（JSON.parse）
// 3. 精确文件：./foo.node（C++ addon）
// 4. 目录索引：./foo/package.json → main字段
// 5. 目录索引：./foo/index.js
```

**模块缓存机制：**

```javascript
// 同一模块只执行一次，第二次require直接返回缓存
console.log(require.cache);  // 查看缓存

// 手动清除缓存
const modulePath = require.resolve('./myModule');
delete require.cache[modulePath];

// 热更新示例
function requireFresh(modulePath) {
    delete require.cache[require.resolve(modulePath)];
    return require(modulePath);
}
```

**package.json exports字段：**

```javascript
// package.json
{
    "name": "my-package",
    "exports": {
        ".": "./dist/index.js",
        "./utils": "./dist/utils/index.js",
        "./config": {
            "import": "./dist/config/index.mjs",
            "require": "./dist/config/index.cjs"
        },
        "./package.json": "./package.json"
    }
}

// 使用
const pkg = require('my-package');           // ./dist/index.js
const utils = require('my-package/utils');     // ./dist/utils/index.js
```

**完整查找流程图：**

```
require('module_name')
       ↓
┌──────────────────┐
│  核心模块?        │ → yes → 直接返回
└────────┬─────────┘
         ↓ no
┌──────────────────┐
│  文件模块(/./../)?│ → yes → 按.js/.json/.node/index顺序查找
└────────┬─────────┘
         ↓ no
┌──────────────────┐
│  node_modules?   │ → yes → 从当前目录向上逐级查找
└────────┬─────────┘
         ↓ no
┌──────────────────┐
│  exports映射?     │ → yes → 使用exports配置的路径
└────────┬─────────┘
         ↓ no
      Error: Cannot find module
```

#### 真实面试题

**题目：说说Node中文件查找的优先级以及Require方法的文件查找策略?**

**满分答案：**

**4级查找优先级：**
1. **核心模块**（fs、path、http等）→ 直接返回缓存
2. **文件模块**（以/、./、../开头）→ 精确路径 → 补全.js/.json/.node → 目录/index.js
3. **node_modules** → 从当前目录向上逐级查找node_modules文件夹
4. **exports映射** → package.json的exports字段配置路径

**node_modules向上查找机制：**
```
当前目录/node_modules → 上级/node_modules → ... → 根目录/node_modules
```

**模块缓存机制：**
- require过的模块缓存在`Module._cache`中
- 同一模块只执行一次
- 可通过`delete require.cache[require.resolve('./module')]`清除缓存实现热更新

---

## 6.16 Node事件循环机制

### 6.16.1 Node事件循环机制

#### 知识点详解

**Node事件循环基于libuv，6个阶段循环执行：**

```
   ┌───────────────────────────────────────────┐
┌─>│                timers                     │  ← setTimeout/setInterval回调
│  └────────────────────┬──────────────────────┘
│  ┌────────────────────┴──────────────────────┐
│  │           pending callbacks                │  ← 上一轮延迟的I/O回调
│  └────────────────────┬──────────────────────┘
│  ┌────────────────────┴──────────────────────┐
│  │            idle, prepare                   │  ← 内部使用
│  └────────────────────┬──────────────────────┘
│  ┌────────────────────┴──────────────────────┐
│  │               poll                         │  ← 获取I/O事件，执行I/O回调
│  │        （可能阻塞等待新I/O）               │    可能执行check阶段
│  └────────────────────┬──────────────────────┘
│  ┌────────────────────┴──────────────────────┐
│  │                check                      │  ← setImmediate回调
│  └────────────────────┬──────────────────────┘
│  ┌────────────────────┴──────────────────────┐
└──┤            close callbacks                 │  ← 关闭事件回调
   └───────────────────────────────────────────┘

   每个阶段后执行微任务：
   nextTick队列（优先级最高） → Promise微任务队列
```

**各阶段详解：**

```javascript
// 1. timers阶段
// 执行setTimeout和setInterval的回调
setTimeout(() => console.log('timer'), 1000);

// 2. pending callbacks阶段
// 执行延迟的I/O错误回调

// 3. idle, prepare阶段
// 内部使用，开发者不接触

// 4. poll阶段（关键）
// - 获取新的I/O事件，执行I/O回调
// - 如果队列为空，可能在此阶段阻塞等待
// - 如果有setImmediate待执行，会立即进入check阶段
const fs = require('fs');
fs.readFile(__filename, () => {
    console.log('poll callback');
});

// 5. check阶段
// 执行setImmediate的回调
setImmediate(() => console.log('immediate'));

// 6. close callbacks阶段
// 执行close事件回调
socket.on('close', () => console.log('closed'));
```

**微任务执行时机：**

```javascript
// nextTick队列（优先级最高）
process.nextTick(() => console.log('nextTick1'));
process.nextTick(() => console.log('nextTick2'));

// Promise微任务
Promise.resolve().then(() => console.log('promise'));

// 每个阶段结束后：
// 1. 清空所有nextTick队列
// 2. 清空所有Promise微任务队列
// 然后进入下一个阶段

// 执行顺序
console.log('script start');
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));

// 输出：
// script start
// nextTick
// promise
// setTimeout 或 setImmediate（取决于进入事件循环的时机）
```

**setTimeout vs setImmediate顺序问题：**

```javascript
// 问题：这两个谁先执行？
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));

// 在主模块（不在I/O回调中）：
// 顺序不确定，取决于系统性能
// 因为主模块结束后，timers阶段可能已过，需要等下一个循环

// 在I/O回调中：
const fs = require('fs');
fs.readFile(__filename, () => {
    setTimeout(() => console.log('setTimeout'), 0);
    setImmediate(() => console.log('setImmediate'));
});
// 输出：setImmediate → setTimeout
// 因为poll阶段完成后立即进入check阶段
```

**poll阶段的特殊行为：**

```javascript
// poll阶段的行为：
// 1. 队列非空 → 依次执行所有回调，直到队列为空或达到上限
// 2. 队列为空 → 
//    a. 如果有setImmediate，进入check阶段
//    b. 如果没有，阻塞等待poll队列新事件

// 示例：poll队列为空时
setImmediate(() => {
    console.log('immediate');  // 进入check阶段
});

setTimeout(() => {
    console.log('timer');  // 进入timers阶段
}, 100);

// 输出：immediate → timer
// 因为poll阶段发现setImmediate，立即进入check
```

#### 真实面试题

**题目：说说对Node中的事件循环机制理解?**

**满分答案：**

**6个阶段详解：**
1. **timers**：执行setTimeout/setInterval回调
2. **pending callbacks**：执行上一轮延迟的I/O回调
3. **idle/prepare**：内部使用
4. **poll**：获取I/O事件，执行I/O回调，可能阻塞等待
5. **check**：执行setImmediate回调
6. **close callbacks**：执行关闭事件回调

**微任务执行时机：**
- 每个阶段结束后清空微任务队列
- nextTick队列优先级高于Promise微任务
- Node 11+后，微任务在每个回调后立即执行（与浏览器一致）

**setTimeout vs setImmediate顺序：**
- 主模块中顺序不确定（取决于timers阶段是否已过）
- I/O回调中setImmediate一定先执行（因为poll完成后进入check阶段）

**poll阶段特殊情况：**
- 队列为空且有setImmediate待执行时，立即进入check阶段
- 没有setImmediate时会阻塞等待新I/O事件

---

## 6.17 Node EventEmitter

### 6.17.1 Node EventEmitter

#### 知识点详解

**EventEmitter是Node事件机制的核心，实现发布-订阅模式：**

```javascript
const { EventEmitter } = require('events');

// 创建事件发射器
class MyEmitter extends EventEmitter {
    constructor() {
        super();
    }
}

const myEmitter = new MyEmitter();
```

**核心API：**

```javascript
// on / addListener - 注册监听器
myEmitter.on('event', (arg1, arg2) => {
    console.log('事件触发', arg1, arg2);
});

// once - 一次性监听（触发一次后自动移除）
myEmitter.once('connect', () => {
    console.log('只触发一次');
});

// emit - 触发事件
myEmitter.emit('event', '参数1', '参数2');
myEmitter.emit('connect');

// off / removeListener - 移除监听器
function handler() {
    console.log('handler');
}
myEmitter.on('data', handler);
myEmitter.off('data', handler);      // 推荐
myEmitter.removeListener('data', handler);  // 等效

// removeAllListeners - 移除所有监听器
myEmitter.removeAllListeners('data');
myEmitter.removeAllListeners();  // 移除所有事件的所有监听器

// listeners - 获取监听器列表
console.log(myEmitter.listeners('event'));

// listenerCount - 获取监听器数量
console.log(myEmitter.listenerCount('event'));

// eventNames - 获取所有事件名
console.log(myEmitter.eventNames());

// setMaxListeners - 设置最大监听器数
myEmitter.setMaxListeners(20);
```

**EventEmitter特点：**

```javascript
// 1. 同步执行
myEmitter.on('sync', () => console.log('1'));
myEmitter.on('sync', () => console.log('2'));
myEmitter.emit('sync');
// 输出：1 → 2（同步执行）

// 2. 按注册顺序执行
// 监听器按注册顺序执行

// 3. 默认最大10个监听器（超过警告内存泄漏）
myEmitter.on('event', () => {});
myEmitter.on('event', () => {});
// ...
// console.log(myEmitter.getMaxListeners()); // 10
// 触发警告：MaxListenersExceededWarning

// 4. error事件（未监听会抛出错误）
myEmitter.emit('error', new Error('错误'));
// 如果没有error监听器，会抛出未捕获的错误
```

**手写EventEmitter实现：**

```javascript
class EventEmitter {
    constructor() {
        this.events = {};  // 事件名 → 回调数组
        this.maxListeners = 10;
    }
    
    // 注册监听器
    on(eventName, listener) {
        if (!this.events[eventName]) {
            this.events[eventName] = [];
        }
        this.events[eventName].push(listener);
        
        // 内存泄漏警告
        if (this.events[eventName].length > this.maxListeners) {
            console.warn(`Warning: possible EventEmitter memory leak detected. ${eventName}`);
        }
        
        return this;  // 支持链式调用
    }
    
    // 一次性监听
    once(eventName, listener) {
        const wrapper = (...args) => {
            this.off(eventName, wrapper);  // 触发后移除
            listener(...args);
        };
        this.on(eventName, wrapper);
        return this;
    }
    
    // 触发事件
    emit(eventName, ...args) {
        const listeners = this.events[eventName] || [];
        listeners.forEach(listener => listener(...args));
        return listeners.length > 0;
    }
    
    // 移除监听器
    off(eventName, listener) {
        if (!this.events[eventName]) return this;
        this.events[eventName] = this.events[eventName]
            .filter(l => l !== listener);
        return this;
    }
    
    // 移除所有监听器
    removeAllListeners(eventName) {
        if (eventName) {
            delete this.events[eventName];
        } else {
            this.events = {};
        }
        return this;
    }
}
```

**继承EventEmitter：**

```javascript
// 常见使用场景：自定义事件类
class Server extends EventEmitter {
    constructor(port) {
        super();
        this.port = port;
    }
    
    start() {
        console.log(`Server starting on port ${this.port}`);
        this.emit('start', this.port);
    }
    
    stop() {
        console.log('Server stopped');
        this.emit('stop');
    }
}

const server = new Server(3000);
server.on('start', (port) => console.log(`Server is running on ${port}`));
server.on('error', (err) => console.error('Server error:', err));
server.start();
```

#### 真实面试题

**题目：说说Node中的EventEmitter?如何实现一个EventEmitter?**

**满分答案：**

**核心API：**
- `on/addListener`：注册监听
- `emit`：触发事件
- `once`：一次性监听
- `off/removeListener`：移除监听
- `removeAllListeners`：移除所有监听

**特点：**
1. 同步执行（emit时立即调用所有监听器）
2. 同一事件可注册多个监听器（按注册顺序执行）
3. 默认最大10个监听器（超过警告内存泄漏）
4. 未捕获的error事件会抛出错误

**实现原理：**
维护一个`events`对象（事件名→回调数组），`on`时push到数组，`emit`时forEach调用，`off`时splice移除。

**使用方式：**
继承`EventEmitter`实现自定义事件类，如HTTP服务器、文件监听器等。

---

## 6.18 Node Stream

### 6.18.1 Node Stream

#### 知识点详解

**Stream是处理流式数据的抽象接口，4种类型：**

```javascript
const { 
    Readable,    // 可读流
    Writable,    // 可写流
    Duplex,      // 双工流
    Transform    // 转换流
} = require('stream');
```

**Readable（可读流）：**

```javascript
const fs = require('fs');

// 方式1：流动模式（自动读取）
const readStream = fs.createReadStream('large-file.txt', {
    encoding: 'utf8',
    highWaterMark: 64 * 1024  // 64KB，默认64KB
});

readStream.on('data', (chunk) => {
    console.log(`Received ${chunk.length} bytes`);
});

readStream.on('end', () => console.log('读取完成'));
readStream.on('error', (err) => console.error(err));

// 方式2：暂停模式（手动读取）
readStream.pause();
readStream.on('readable', () => {
    let chunk;
    while ((chunk = readStream.read()) !== null) {
        console.log(`Read: ${chunk.length} bytes`);
    }
});
readStream.resume();

// HTTP请求是可读流
const http = require('http');
http.get('http://example.com', (res) => {
    res.on('data', (chunk) => console.log(chunk));
});
```

**Writable（可写流）：**

```javascript
const fs = require('fs');

const writeStream = fs.createWriteStream('output.txt');

// 写入数据
writeStream.write('Hello ', 'utf8', () => console.log('chunk1 done'));
writeStream.write('World', 'utf8', () => console.log('chunk2 done'));
writeStream.end('!', () => console.log('write complete'));

// 事件
writeStream.on('finish', () => console.log('All data written'));
writeStream.on('error', (err) => console.error(err));

// drain事件（背压处理）
const readable = fs.createReadStream('big-file.txt');
const writable = fs.createWriteStream('output.txt');

readable.on('data', (chunk) => {
    const canContinue = writable.write(chunk);
    if (!canContinue) {
        readable.pause();  // 暂停读取
        writable.taint('drain', () => readable.resume());  // 等待排干后继续
    }
});
```

**Duplex（双工流）：**

```javascript
// 既可读又可写的流
// 例如：TCP socket、WebSocket
const { Duplex } = require('stream');

const duplex = new Duplex({
    read(size) {
        this.push('data from read');
        this.push(null);  // 结束读取
    },
    write(chunk, encoding, callback) {
        console.log('write:', chunk.toString());
        callback();  // 告诉写入完成
    }
});
```

**Transform（转换流）：**

```javascript
// 转换流：读取时转换数据后输出
const { Transform } = require('stream');
const fs = require('fs');
const zlib = require('zlib');

// 示例1：大写转换
const upperCase = new Transform({
    transform(chunk, encoding, callback) {
        this.push(chunk.toString().toUpperCase());
        callback();
    }
});

// 示例2：文件压缩
const gzip = zlib.createGzip();
const gunzip = zlib.createGunzip();

// 管道链
fs.createReadStream('input.txt')
    .pipe(upperCase)
    .pipe(gzip)
    .pipe(fs.createWriteStream('output.txt.gz'));

// 解压
fs.createReadStream('output.txt.gz')
    .pipe(gunzip)
    .pipe(fs.createWriteStream('output-decompressed.txt'));
```

**管道机制（pipe）：**

```javascript
const fs = require('fs');

// 基本用法
readStream.pipe(writeStream);

// 链式管道
input
    .pipe(transform1)
    .pipe(transform2)
    .pipe(output);

// 使用 pipeline（推荐，自动处理错误）
const { pipeline } = require('stream/promises');

await pipeline(
    fs.createReadStream('input.txt'),
    upperCaseTransform,
    fs.createWriteStream('output.txt')
);
// pipeline会在任意步骤出错时自动销毁所有流
```

**背压（Backpressure）：**

```javascript
// 问题：写入速度 < 读取速度，导致内存堆积
// 解决：背压机制

// 低效写法
readable.on('data', (chunk) => {
    writable.write(chunk);  // 可能返回false，表示缓冲区满
});

// 高效写法（手动背压）
readable.on('data', (chunk) => {
    const canContinue = writable.write(chunk);
    if (!canContinue) {
        readable.pause();
        writable.once('drain', () => readable.resume());
    }
});

// 或者用 pipe/pipeline（自动处理背压）
readable.pipe(writable);  // 自动管理
```

**Stream应用场景：**

```javascript
// 1. 大文件读写（不会一次性加载到内存）
const fs = require('fs');
fs.createReadStream('movie.mp4').pipe(fs.createWriteStream('copy.mp4'));

// 2. HTTP请求/响应流
app.get('/download', (req, res) => {
    const stream = fs.createReadStream('large-file.zip');
    res.setHeader('Content-Disposition', 'attachment');
    stream.pipe(res);  // 边读边发，不占用大量内存
});

// 3. 文件压缩解压
const zlib = require('zlib');
fs.createReadStream('file.txt')
    .pipe(zlib.createGzip())
    .pipe(fs.createWriteStream('file.txt.gz'));

// 4. 日志处理
const { createReadStream } = require('fs');
const { createWriteStream } = require('fs');
const { Transform } = require('stream');

class JSONParser extends Transform {
    constructor() {
        super({ objectMode: true });
    }
    _transform(chunk, encoding, callback) {
        try {
            const obj = JSON.parse(chunk.toString());
            this.push(obj);
            callback();
        } catch (e) {
            callback(e);
        }
    }
}

// 5. HTTP代理
const http = require('http');
http.createServer((req, res) => {
    const options = { hostname: 'target.com', path: req.url };
    req.pipe(http.request(options, (response) => {
        response.pipe(res);
    }));
});
```

#### 真实面试题

**题目：说说对Node中的Stream的理解?应用场景?**

**满分答案：**

**4种Stream类型：**
- **Readable**：可读流（fs.createReadStream、HTTP request）
- **Writable**：可写流（fs.createWriteStream、HTTP response）
- **Duplex**：双工流（TCP socket、WebSocket）
- **Transform**：转换流（zlib、crypto）

**Stream优势：**
- **内存效率**：不需要将整个文件加载到内存
- **时间效率**：数据到达即处理，不需要等待全部加载

**pipe管道机制：**
- `readable.pipe(writable)`自动管理数据流
- 自动处理背压（当写入速度跟不上读取速度时暂停读取）

**背压概念：**
- 当write()返回false时，表示缓冲区满，需要暂停读取
- 等drain事件后再恢复读取

**典型应用场景：**
1. 大文件读写（边读边写，不占内存）
2. 文件压缩解压（gzip/gunzip）
3. HTTP代理转发
4. 日志处理（实时解析、分析）
5. 实时数据处理（AI流式输出）

---

## 6.19 Node Buffer

### 6.19.1 Node Buffer

#### 知识点详解

**Buffer是Node处理二进制数据的类，表示固定长度的字节序列：**

```javascript
// Buffer在V8堆外分配内存（不归GC管理）
// 用于处理文件I/O、网络传输等二进制数据
```

**创建Buffer：**

```javascript
// 方式1：从字符串/数组创建
const buf1 = Buffer.from('Hello', 'utf8');
const buf2 = Buffer.from([0x68, 0x65, 0x6c, 0x6c, 0x6f]);

// 方式2：分配指定大小（不清零，可能包含旧数据）
const buf3 = Buffer.allocUnsafe(10);  // 更快，但不安全

// 方式3：分配指定大小（清零，更安全）
const buf4 = Buffer.alloc(10);

// allocUnsafe安全风险
const bufUnsafe = Buffer.allocUnsafe(10);
console.log(bufUnsafe.toString());  // 可能输出乱码/旧数据
```

**Buffer与ArrayBuffer关系：**

```javascript
// Buffer是Uint8Array的子类
const buf = Buffer.from('Hello');
console.log(buf instanceof Uint8Array);  // true

// 共享底层ArrayBuffer
const arrayBuffer = buf.buffer;
console.log(arrayBuffer);  // ArrayBuffer { byteLength: 5 }

// 创建指定大小的Buffer
const buf2 = Buffer.alloc(10);
console.log(buf2.buffer.byteLength);  // 10
```

**常用操作：**

```javascript
const buf = Buffer.from('Hello World', 'utf8');

// 长度和索引
console.log(buf.length);       // 11
console.log(buf[0]);          // 72 (H的ASCII)

// 读写字符串
console.log(buf.toString('utf8'));      // Hello World
console.log(buf.toString('hex'));        // 48656c6c6f20576f726c64
console.log(buf.toString('base64'));     // SGVsbG8gV29ybGQ=

// 切片（共享内存）
const slice = buf.subarray(0, 5);
slice[0] = 0x41;  // 'A'
console.log(buf.toString());  // Aello World

// 拼接（注意：不用+运算符）
const buf1 = Buffer.from('Hello');
const buf2 = Buffer.from(' World');
const combined = Buffer.concat([buf1, buf2]);
console.log(combined.toString());  // Hello World

// 比较
const bufA = Buffer.from('abc');
const bufB = Buffer.from('abd');
console.log(bufA.compare(bufB));  // -1 (A < B)

// 填充
const bufFill = Buffer.alloc(10);
bufFill.fill(0);  // 用0填充

// 拷贝
const bufSrc = Buffer.from('Hello');
const bufDest = Buffer.alloc(5);
bufSrc.copy(bufDest);
```

**Buffer安全问题：**

```javascript
// 问题：Buffer.allocUnsafe可能泄露敏感数据
// 解决1：使用Buffer.alloc（更安全但稍慢）
const safe = Buffer.alloc(10);
safe.fill(0);  // 确保清零

// 解决2：用完后清零
const buf = Buffer.allocUnsafe(1024);
try {
    // 使用buf
} finally {
    buf.fill(0);  // 清除敏感数据
}
```

**Buffer应用场景：**

```javascript
// 1. 文件I/O
const fs = require('fs');
const data = fs.readFileSync('image.png');
// fs模块默认返回Buffer

// 2. 网络传输（TCP/UDP）
const net = require('net');
const server = net.createServer((socket) => {
    socket.on('data', (buf) => {
        console.log('Received:', buf);
    });
    socket.write(Buffer.from('Hello client'));
});

// 3. 加密/哈希
const crypto = require('crypto');
const hash = crypto.createHash('sha256');
hash.update('password');
console.log(hash.digest('hex'));

// 4. 图片处理
const sharp = require('sharp');
const imageBuffer = fs.readFileSync('input.png');
const resized = await sharp(imageBuffer)
    .resize(200, 200)
    .toBuffer();

// 5. 编码转换
const buf = Buffer.from('你好', 'utf8');
console.log(buf.toString('base64'));     // 5L2g5aW9
console.log(Buffer.from('5L2g5aW9', 'base64').toString('utf8'));  // 你好

// 6. WebSocket二进制数据
const WebSocket = require('ws');
const ws = new WebSocket('ws://localhost:8080');
ws.on('message', (buf) => {
    const int32 = buf.readInt32BE(0);  // 从Buffer读取二进制数据
    console.log(int32);
});
```

#### 真实面试题

**题目：说说对Node中的Buffer的理解?应用场景?**

**满分答案：**

**Buffer本质：**
- Buffer是Node.js用于处理二进制数据的类
- 在V8堆外分配内存（不受GC常规管理）
- 是Uint8Array的子类，共享底层ArrayBuffer

**创建方式及安全注意：**
- `Buffer.from()`：从字符串或数组创建
- `Buffer.alloc(size)`：分配清零（安全）
- `Buffer.allocUnsafe(size)`：分配不清零（更快，但可能有旧数据残留）
- 敏感数据场景必须用`Buffer.alloc()`或事后`fill(0)`

**与ArrayBuffer的关系：**
Buffer是Uint8Array的子类，共享底层ArrayBuffer，可以相互转换。

**5个应用场景：**
1. **文件I/O**：fs模块默认返回Buffer
2. **网络传输**：TCP/UDP数据处理
3. **加密/哈希**：crypto模块使用Buffer
4. **图片处理**：sharp等图像库
5. **编码转换**：base64/hex/utf8互转

**Buffer拼接注意：**
- 必须用`Buffer.concat()`，不能用`+`运算符
- `+`运算符会把Buffer转成字符串，丢失二进制数据

---

