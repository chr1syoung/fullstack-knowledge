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

#### 知识点详解

**什么是 SSE？**

SSE 是一种服务器向浏览器推送数据的技术，是 HTTP 服务器推送的轻量级方案。

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

