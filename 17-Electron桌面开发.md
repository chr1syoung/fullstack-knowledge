# 十七、Electron 桌面应用开发

> 本章涵盖 Electron 从基础原理到打包部署的全流程知识点与面经。

---

## 17.1 Electron 基础与核心原理

### 17.1.1 Electron 概述与核心架构

#### 知识点详解

**Electron 是什么：**

Electron 是一个使用 Web 技术（HTML + CSS + JavaScript / React / Vue）构建跨平台桌面应用程序的开源框架，由 GitHub 于 2013 年创建（Atom 编辑器内部项目），现由 OpenJS Foundation 维护。

**代表应用：**

```
VS Code、GitHub Desktop、Slack、Discord、Notion、Figma（早期版本）、
Postman、Microsoft Teams、Zoom、Typora
```

**核心架构：**

```
┌─────────────────────────────────────────────────────┐
│                    Electron App                       │
├─────────────────────────────────────────────────────┤
│                                                      │
│              ┌──────────────────┐                   │
│              │   主进程 (Main)   │                   │
│              │  Node.js 环境     │                   │
│              │  窗口管理 / 原生API │                   │
│              └────────┬─────────┘                   │
│                       │                            │
│              IPC Bridge (Chromium / V8)             │
│                       │                            │
│              ┌────────┴─────────┐                   │
│              │  渲染进程 (Renderer) │               │
│              │  Chromium 环境     │                   │
│              │  页面 UI（React/Vue）│                │
│              └──────────────────┘                   │
│                                                      │
└─────────────────────────────────────────────────────┘

Electron 版本 = Chromium（渲染）+ Node.js（主进程）+ Native API（系统交互）
```

**主进程（Main Process）：**

```javascript
// main.js — 主进程入口
const { app, BrowserWindow, Menu, Tray, globalShortcut, dialog } = require('electron');
const path = require('path');

// 1. 应用生命周期事件
app.whenReady().then(() => {
    createWindow();
    createMenu();
    registerShortcuts();
});

app.on('window-all-closed', () => {
    // macOS 除外：关闭窗口后 App 仍然在 Dock 中运行
    if (process.platform !== 'darwin') {
        app.quit();
    }
});

app.on('activate', () => {
    // macOS：点击 Dock 图标时重新创建窗口
    if (BrowserWindow.getAllWindows().length === 0) {
        createWindow();
    }
});

function createWindow() {
    mainWindow = new BrowserWindow({
        width: 1200,
        height: 800,
        minWidth: 800,
        minHeight: 600,
        title: 'My Electron App',
        backgroundColor: '#ffffff',
        show: false,              // 先隐藏，等 ready-to-show 再显示（避免白屏）
        webPreferences: {
            nodeIntegration: false,   // 安全：禁用 Node.js 集成
            contextIsolation: true,    // 安全：上下文隔离
            sandbox: true,            // 安全：启用沙箱
            preload: path.join(__dirname, 'preload.js'),  // 预加载脚本
        },
    });

    // 监听页面加载完成再显示窗口（减少白屏）
    mainWindow.once('ready-to-show', () => {
        mainWindow.show();
    });

    // 加载页面
    if (process.env.NODE_ENV === 'development') {
        mainWindow.loadURL('http://localhost:3000');  // 开发环境
        mainWindow.webContents.openDevTools();         // 打开 DevTools
    } else {
        mainWindow.loadFile(path.join(__dirname, '../dist/index.html'));  // 生产环境
    }

    // 窗口关闭时触发（可用于清理或阻止关闭）
    mainWindow.on('close', (event) => {
        if (!app.isQuitting) {
            event.preventDefault();  // 阻止默认关闭行为
            mainWindow.hide();       // 隐藏窗口而非真正关闭
        }
    });
}
```

**渲染进程（Renderer Process）：**

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- CSP 安全策略 -->
    <meta http-equiv="Content-Security-Policy"
          content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';">
    <title>My Electron App</title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```

**Electron 渲染进程 ≠ 普通浏览器：**

| 特性 | 普通浏览器 | Electron 渲染进程 |
|------|----------|-----------------|
| Node.js 环境 | 无 | 默认无（需开启 nodeIntegration） |
| 文件系统访问 | 受限 | 通过 IPC 调用主进程 |
| 原生对话框 | 无 | 可通过 IPC 调用 |
| 系统通知 | 受限 | 完全支持 |
| 访问网络 | 受限（CORS） | 无 CORS 限制 |
| 调试工具 | 浏览器 DevTools | 内置 DevTools |

**Electron 项目结构：**

```
electron-app/
├── package.json
├── main.js              # 主进程入口
├── preload.js           # 预加载脚本（安全桥梁）
├── src/
│   ├── index.html
│   ├── index.js         # 渲染进程入口
│   ├── App.jsx         # React/Vue 组件
│   └── components/
├── build/               # 打包资源（图标等）
│   └── icon.png
└── dist/                # 生产构建产物
```

**Electron vs Tauri 对比：**

| 维度 | Electron | Tauri |
|------|---------|-------|
| 底层引擎 | Chromium + Node.js | Rust + WebView（系统原生） |
| 包体积 | 大（80-200MB） | 小（5-15MB） | 
| 内存占用 | 高 | 低 |
| 启动速度 | 慢（Chromium 全量加载） | 快 |
| 生态 | 成熟（VS Code 等） | 快速成长 |
| 语言 | JavaScript/TypeScript | Rust + Web |
| 原生能力 | 完整（Node.js 所有模块） | 完整（Rust 生态） |
| 安全 | 需手动配置 | Rust 更安全 |
| 适用场景 | 复杂 UI 应用（VS Code） | 轻量工具（替代 Web） |

#### 真实面试题

**题目：什么是 Electron？它的核心架构是什么？Electron 和普通浏览器有什么区别？**

**满分答案：**

**Electron 定义：**
Electron 是一个使用 Web 技术栈（HTML/CSS/JavaScript）构建跨平台桌面应用的框架。核心由 Chromium（渲染页面）+ Node.js（主进程）+ 自定义 API 组成。

**核心架构：**
- **主进程**：Node.js 环境，拥有操作系统完整权限，管理窗口、菜单、托盘、系统事件
- **渲染进程**：Chromium 环境，负责 UI 渲染，只能通过 IPC 与主进程通信
- **预加载脚本**：安全桥梁，将主进程 API 安全暴露给渲染进程

**Electron vs 普通浏览器：**

| 维度 | 普通浏览器 | Electron |
|------|----------|---------|
| Node.js | ❌ | ❌（默认） |
| 本地文件系统 | 受限 | 通过 IPC |
| 无 CORS 限制 | ❌ | ✅ |
| 原生对话框 | ❌ | ✅ |
| 完整操作系统权限 | ❌ | ✅ |

**Electron 为什么能开发桌面应用：**
Chromium 提供 Web 渲染能力，Node.js 提供系统能力，两者组合即可让 Web 代码运行在桌面环境中，并通过 Electron API 访问原生系统能力。

---

## 17.2 主进程与渲染进程通信（IPC）

### 17.2.1 IPC 通信机制详解

#### 知识点详解

**IPC 通信四种方式：**

```
渲染进程 → 主进程（单向）：ipcRenderer.send  → ipcMain.on
渲染进程 → 主进程（双向）：ipcRenderer.invoke → ipcMain.handle
主进程 → 渲染进程：webContents.send → ipcRenderer.on
渲染进程 → 主进程（同步）：ipcRenderer.sendSync → ipcMain.on（⚠️ 已废弃，不推荐）
```

**预加载脚本（Preload Script）— 安全通信桥梁：**

```javascript
// preload.js（预加载脚本）
const { contextBridge, ipcRenderer } = require('electron');

// 通过 contextBridge 安全暴露 API，不暴露 Node.js 给渲染进程
contextBridge.exposeInMainWorld('electronAPI', {
    // 单向通信：渲染进程 → 主进程
    send: (channel, data) => {
        // 白名单验证：只允许特定的 channel
        const validChannels = ['set-title', 'open-file', 'save-file'];
        if (validChannels.includes(channel)) {
            ipcRenderer.send(channel, data);
        }
    },

    // 双向通信：invoke/handle（Promise 方式，最推荐）
    invoke: async (channel, data) => {
        const validChannels = ['get-user-data', 'read-file', 'write-file'];
        if (validChannels.includes(channel)) {
            return await ipcRenderer.invoke(channel, data);
        }
    },

    // 主进程 → 渲染进程（监听）
    on: (channel, callback) => {
        const validChannels = ['update-available', 'download-progress'];
        if (validChannels.includes(channel)) {
            ipcRenderer.on(channel, (event, ...args) => callback(...args));
        }
    },

    // 移除监听（防止内存泄漏）
    removeListener: (channel) => {
        ipcRenderer.removeAllListeners(channel);
    },
});
```

**方式一：单向通信（渲染 → 主）**

```javascript
// 主进程 main.js
const { ipcMain } = require('electron');

ipcMain.on('set-title', (event, title) => {
    const win = BrowserWindow.fromWebContents(event.sender);
    win.setTitle(title);
});

// 渲染进程（React/Vue）
window.electronAPI.send('set-title', 'New Window Title');
```

**方式二：双向通信（invoke/handle，Promise，推荐）**

```javascript
// 主进程 main.js
const { ipcMain, dialog } = require('electron');

ipcMain.handle('dialog:openFile', async (event, options) => {
    const result = await dialog.showOpenDialog({
        properties: ['openFile'],
        filters: [
            { name: 'Images', extensions: ['jpg', 'png', 'gif'] },
            { name: 'All Files', extensions: ['*'] },
        ],
        ...options,
    });
    return result;  // 返回 Promise resolve 的值
});

ipcMain.handle('read-file', async (event, filePath) => {
    const fs = require('fs').promises;
    const content = await fs.readFile(filePath, 'utf-8');
    return content;
});

ipcMain.handle('write-file', async (event, filePath, content) => {
    const fs = require('fs').promises;
    await fs.writeFile(filePath, content, 'utf-8');
    return { success: true };
});
```

```javascript
// 渲染进程（调用方）
async function openAndReadFile() {
    // 打开文件对话框
    const result = await window.electronAPI.invoke('dialog:openFile');
    if (!result.canceled && result.filePaths.length > 0) {
        const filePath = result.filePaths[0];
        const content = await window.electronAPI.invoke('read-file', filePath);
        console.log('文件内容:', content);
        return content;
    }
}
```

**方式三：主进程 → 渲染进程（主动推送）**

```javascript
// 主进程 main.js
// 在某个时机主动推送消息给渲染进程
function sendUpdateNotification() {
    const mainWindow = BrowserWindow.getAllWindows()[0];
    if (mainWindow && !mainWindow.isDestroyed()) {
        mainWindow.webContents.send('update-available', {
            version: '2.0.0',
            releaseNotes: 'New features added',
        });
    }
}

// 渲染进程（监听方）
window.electronAPI.on('update-available', (data) => {
    console.log('有新版本:', data.version);
    showUpdateDialog(data);
});
```

**渲染进程多窗口通信：**

```javascript
// 主进程：创建多个窗口
const windows = {};

function createMainWindow() {
    windows.main = new BrowserWindow({ ... });
}

function createSettingsWindow() {
    windows.settings = new BrowserWindow({
        width: 600,
        height: 400,
        parent: windows.main,  // 模态子窗口
        modal: true,
        show: false,
    });
    windows.settings.once('ready-to-show', () => windows.settings.show());
}

// 渲染进程 → 渲染进程（通过主进程中转）
// main.js
ipcMain.on('main-to-settings', (event, data) => {
    windows.settings?.webContents.send('settings:update', data);
});
```

**IPC Channel 命名规范：**

```javascript
// 推荐：使用统一的 channel 命名规范
const IPC_CHANNELS = {
    // 文件操作
    FILE_READ: 'file:read',
    FILE_WRITE: 'file:write',
    FILE_OPEN_DIALOG: 'file:open-dialog',

    // 系统操作
    WINDOW_MINIMIZE: 'window:minimize',
    WINDOW_MAXIMIZE: 'window:maximize',
    WINDOW_CLOSE: 'window:close',
    WINDOW_SET_TITLE: 'window:set-title',

    // 更新通知
    UPDATE_AVAILABLE: 'update:available',
    UPDATE_DOWNLOAD_PROGRESS: 'update:progress',
    UPDATE_DOWNLOADED: 'update:downloaded',

    // 应用信息
    GET_APP_VERSION: 'app:get-version',
    GET_PLATFORM: 'app:get-platform',
};
```

#### 真实面试题

**题目：Electron 中主进程和渲染进程如何通信？什么是 contextBridge？为什么推荐使用它？**

**满分答案：**

**IPC 通信三种方式：**

| 方式 | 渲染进程调用 | 主进程处理 | 适用场景 |
|------|------------|----------|---------|
| `ipcRenderer.send` + `ipcMain.on` | `send(channel, data)` | `on(channel, handler)` | 单向通知，无需返回值 |
| `ipcRenderer.invoke` + `ipcMain.handle` | `await invoke(channel)` | `handle(channel, async handler)` | 需要返回值（Promise，推荐） |
| `webContents.send` | 主进程主动推送 | `on(channel, handler)` | 主进程主动通知渲染进程 |

**contextBridge 的核心作用：**

```javascript
// preload.js — contextBridge 在渲染进程的 window 和主进程之间建立隔离的安全桥梁
contextBridge.exposeInMainWorld('electronAPI', {
    readFile: (path) => ipcRenderer.invoke('file:read', path),
    // ...
});
```

**推荐原因：**
1. **安全**：渲染进程无法直接访问 `require`、`fs`、`process`，只能通过暴露的 API 间接访问
2. **隔离**：即使网站代码被 XSS 注入，攻击者也无法调用主进程能力
3. **兼容**：配合 `contextIsolation: true`，渲染进程和 preload 脚本的上下文完全隔离

**⚠️ 危险配置（禁止使用）：**
```javascript
// ❌ 绝对不要这样配置！
new BrowserWindow({
    nodeIntegration: true,    // 让渲染进程直接使用 require
    contextIsolation: false,  // 关闭上下文隔离
    sandbox: false,           // 关闭沙箱
});
// 攻击者执行 require('child_process').exec('rm -rf /') 即可控制系统
```

---

## 17.3 Electron 安全架构

### 17.3.1 安全配置与最佳实践

#### 知识点详解

**Electron 安全三大核心配置：**

```javascript
// ✅ 推荐的安全配置
const mainWindow = new BrowserWindow({
    webPreferences: {
        // 1. 上下文隔离：渲染进程和 preload 脚本上下文分离
        contextIsolation: true,

        // 2. 禁用 Node.js 集成：渲染进程无法直接 require
        nodeIntegration: false,

        // 3. 启用沙箱：渲染进程无法访问系统资源
        sandbox: true,

        // 4. 预加载脚本（安全通信的唯一通道）
        preload: path.join(__dirname, 'preload.js'),

        // 5. 禁用 webSecurity 仅在开发时使用
        webSecurity: true,

        // 6. 禁止加载远程内容（除非明确指定）
        allowRunningInsecureContent: false,
    },
});
```

**CSP（内容安全策略）：**

```html
<!-- index.html — 防止 XSS 和恶意脚本注入 -->
<meta http-equiv="Content-Security-Policy"
      content="
          default-src 'self';
          script-src 'self';
          style-src 'self' 'unsafe-inline';
          img-src 'self' data: https:;
          font-src 'self';
          connect-src 'self' https://api.example.com;
          frame-src 'none';
      ">
```

**安全的文件操作（永远不要相信用户输入）：**

```javascript
// 主进程：安全的文件操作
const path = require('path');
const { safeStorage } = require('electron');

ipcMain.handle('secure:save-token', async (event, { token }) => {
    // 1. 验证 token 格式（防止注入）
    if (typeof token !== 'string' || token.length > 1000) {
        throw new Error('Invalid token format');
    }

    // 2. 使用 safeStorage 加密敏感数据
    const encrypted = safeStorage.encryptString(token);
    const userDataPath = app.getPath('userData');
    const tokenPath = path.join(userDataPath, 'auth.token');

    // 3. 白名单路径验证（防止路径遍历攻击）
    const normalizedPath = path.normalize(tokenPath);
    if (!normalizedPath.startsWith(userDataPath)) {
        throw new Error('Path traversal detected');
    }

    await require('fs').promises.writeFile(encrypted, tokenPath);
    return { success: true };
});

// 路径遍历攻击防护
ipcMain.handle('file:read', async (event, userPath) => {
    const fs = require('fs').promises;
    const userDataPath = app.getPath('userData');

    // 解析路径并验证
    const requestedPath = path.normalize(userPath);
    const absolutePath = path.resolve(requestedPath);

    // 确保文件在允许的目录内
    if (!absolutePath.startsWith(userDataPath)) {
        throw new Error('Access denied: path outside user data directory');
    }

    return await fs.readFile(absolutePath, 'utf-8');
});
```

**安全会话管理：**

```javascript
const { session } = require('electron');

app.whenReady().then(() => {
    // 1. 配置安全 Cookie
    session.defaultSession.cookies.get({ name: 'session' }).then(cookies => {
        session.defaultSession.cookies.set({
            name: 'session',
            value: 'your-session-token',
            httpOnly: true,      // 禁止 JS 访问 Cookie
            secure: true,        // 仅 HTTPS
            sameSite: 'strict',  // CSRF 防护
        });
    });

    // 2. 配置 CORS（如果需要加载远程资源）
    session.defaultSession.webRequest.onHeadersReceived((details, callback) => {
        callback({
            responseHeaders: {
                ...details.responseHeaders,
                'Content-Security-Policy': [
                    "default-src 'self'; script-src 'self';"
                ],
            },
        });
    });

    // 3. 设置安全请求头
    session.defaultSession.webRequest.onBeforeSendHeaders((details, callback) => {
        details.requestHeaders['X-Requested-With'] = 'MyElectronApp';
        callback({ cancel: false, requestHeaders: details.requestHeaders });
    });

    // 4. 禁用自动下载未验证的文件
    session.defaultSession.on('will-download', (event, item, webContents) => {
        // 验证下载来源
        if (!isTrustedDownload(item.getURL())) {
            event.preventDefault();
        }
    });
});
```

**Electron 安全清单：**

| 安全措施 | 说明 | 优先级 |
|---------|------|--------|
| `nodeIntegration: false` | 禁用渲染进程 Node.js | 🔴 必须 |
| `contextIsolation: true` | 上下文隔离 | 🔴 必须 |
| `sandbox: true` | 启用渲染进程沙箱 | 🔴 必须 |
| CSP Meta 标签 | 内容安全策略 | 🔴 必须 |
| contextBridge | 安全的 IPC 通信 | 🟡 推荐 |
| safeStorage | 加密敏感数据 | 🟡 推荐 |
| 路径验证 | 防止路径遍历 | 🟡 推荐 |
| Session 安全配置 | Cookie / CORS | 🟡 推荐 |
| disableRemote | 禁用 remote 模块 | 🟡 推荐（已废弃） |

#### 真实面试题

**题目：Electron 有哪些主要安全风险？如何配置安全的 Electron 应用？**

**满分答案：**

**主要安全风险：**

1. **远程代码执行（RCE）**：`nodeIntegration: true` + 加载远程内容 → 攻击者可在页面执行任意系统命令
2. **XSS → 本地文件读写**：关闭 `contextIsolation` 后，XSS 可访问 `window.require('fs')`
3. **路径遍历**：用户输入的文件路径未经校验，可读取任意系统文件
4. **敏感数据泄露**：明文存储 Token、密码
5. **恶意下载**：应用自动下载并执行危险文件

**安全配置（必做）：**

```javascript
// ✅ 最低安全标准
new BrowserWindow({
    webPreferences: {
        nodeIntegration: false,   // 必须禁用
        contextIsolation: true,   // 必须启用
        sandbox: true,           // 必须启用
        preload: path.join(__dirname, 'preload.js'),
    },
});
```

**安全编码规范：**
- 永远使用 `ipcRenderer.invoke` 而非 `ipcRenderer.sendSync`
- 永远通过 contextBridge 暴露 API，不暴露 Node.js 全局对象
- 永远验证用户输入（文件路径、数据格式）
- 永远使用 `safeStorage` 存储敏感信息

---

## 17.4 Electron 打包与发布

### 17.4.1 electron-builder 完整配置

#### 知识点详解

**electron-builder 核心概念：**

```
源码（src/）
    ↓ npm run build（前端构建）
前端产物（dist/）
    ↓ electron-builder 打包
平台安装包（.exe / .dmg / .AppImage）
```

**package.json 完整配置：**

```json
{
  "name": "my-electron-app",
  "version": "1.0.0",
  "description": "My Electron Application",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "dev": "concurrently \"npm run dev:web\" \"wait-on http://localhost:3000 && electron .\"",
    "dev:web": "vite",
    "build": "vite build",
    "build:electron": "npm run build && electron-builder",
    "build:win": "npm run build && electron-builder --win",
    "build:mac": "npm run build && electron-builder --mac",
    "build:linux": "npm run build && electron-builder --linux",
    "build:all": "npm run build && electron-builder --win --mac --linux"
  },
  "build": {
    "appId": "com.mycompany.myapp",
    "productName": "MyElectronApp",
    "copyright": "Copyright © 2026 MyCompany",

    "directories": {
      "output": "release",
      "buildResources": "build"
    },

    "files": [
      "dist/**/*",
      "main.js",
      "preload.js",
      "!**/*.map",
      "!**/node_modules/*/{CHANGELOG.md,README.md,README,readme.md,readme}",
      "!**/node_modules/*/{test,__tests__,tests,powered-test,example,examples}",
      "!**/node_modules/.bin"
    ],

    "extraMetadata": {
      "main": "main.js"
    },

    "asar": true,
    "compression": "maximum",

    "win": {
      "target": [
        {
          "target": "nsis",
          "arch": ["x64"]
        },
        {
          "target": "portable",
          "arch": ["x64"]
        }
      ],
      "icon": "build/icon.ico",
      "artifactName": "${productName}-${version}-${arch}.${ext}",
      "requestedExecutionLevel": "asInvoker"
    },

    "nsis": {
      "oneClick": false,
      "perMachine": false,
      "allowToChangeInstallationDirectory": true,
      "createDesktopShortcut": true,
      "createStartMenuShortcut": true,
      "shortcutName": "MyElectronApp",
      "installerIcon": "build/icon.ico",
      "uninstallerIcon": "build/icon.ico",
      "installerHeaderIcon": "build/icon.ico",
      "deleteAppDataOnUninstall": false,
      "differentialPackage": false
    },

    "mac": {
      "category": "public.app-category.developer-tools",
      "target": [
        { "target": "dmg", "arch": ["x64", "arm64"] },
        { "target": "zip", "arch": ["x64", "arm64"] }
      ],
      "icon": "build/icon.icns",
      "hardenedRuntime": true,
      "gatekeeperAssess": false,
      "entitlements": "build/entitlements.mac.plist",
      "entitlementsInherit": "build/entitlements.mac.plist"
    },

    "linux": {
      "target": [
        { "target": "AppImage", "arch": ["x64"] },
        { "target": "deb", "arch": ["x64"] }
      ],
      "category": "Development",
      "icon": "build/icons"
    },

    "publish": {
      "provider": "generic",
      "url": "https://update.example.com/electron-app/"
    }
  }
}
```

**NSIS 安装程序配置（Windows）：**

```nsis
; build/installer.nsh（自定义 NSIS 脚本）
!macro customHeader
    !system "echo 'Custom header'"
!macroend

!macro customInstall
    ; 创建额外快捷方式
    CreateShortCut "$DESKTOP\MyApp.lnk" "$INSTDIR\MyApp.exe"
!macroend

!macro customUnInstall
    ; 卸载时清理配置
    RMDir /r "$APPDATA\MyApp"
!macroend
```

**macOS 权限配置（entitlements）：**

```xml
<!-- build/entitlements.mac.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- 允许网络访问 -->
    <key>com.apple.security.network.client</key>
    <true/>
    <!-- 允许读取文件 -->
    <key>com.apple.security.files.user-selected.read-only</key>
    <true/>
    <!-- 允许写入文件 -->
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
    <!-- App Sandbox（沙箱） -->
    <key>com.apple.security.app-sandbox</key>
    <true/>
</dict>
</plist>
```

**完整打包流程（Vite + Electron）：**

```bash
# 1. 安装依赖
npm install electron electron-builder --save-dev

# 2. 国内镜像（加速 Electron 下载）
npm config set electron_mirror https://npmmirror.com/mirrors/electron/

# 3. 构建前端（Vite/Webpack）
npm run build

# 4. electron-builder 打包
npm run build:win    # Windows
npm run build:mac    # macOS
npm run build:linux  # Linux

# 5. 查看产物
ls release/
# my-electron-app-1.0.0.exe（NSIS 安装包）
# my-electron-app-1.0.0.exe（portable 便携版）
```

**electron-builder 产物说明：**

| 产物 | 说明 | 使用场景 |
|------|------|---------|
| `.exe`（NSIS） | Windows 安装程序 | 分发下载、企业部署 |
| `.exe`（portable） | 单文件便携版 | U盘运行、无需安装 |
| `.dmg` | macOS 磁盘镜像 | macOS 分发 |
| `.AppImage` | Linux 便携包 | Linux 各发行版 |
| `.deb` | Debian/Ubuntu 安装包 | Linux 系统包管理 |

**asar 打包（源码保护）：**

```bash
# Electron 默认将源码打包为 asar 归档
# 查看 asar 内容
npx asar list app.asar

# 解压 asar（调试用）
npx asar extract app.asar app.asar.unpacked
npx asar extract-file app.asar some-file.dat
```

#### 真实面试题

**题目：Electron 应用如何打包？electron-builder 的核心配置有哪些？如何生成 Windows 安装包？**

**满分答案：**

**打包流程：**

```
1. npm run build（前端构建）→ dist/
2. electron-builder 读取 dist/ 和 main.js
3. 下载对应平台 Electron 二进制文件
4. 打包为 .exe / .dmg / .AppImage
```

**核心配置：**

| 配置项 | 说明 |
|-------|------|
| `appId` | 唯一应用标识（com.company.app） |
| `productName` | 用户看到的应用名称 |
| `files` | 需要打包的源码文件 |
| `win/mac/linux` | 各平台打包配置 |
| `nsis` | Windows 安装程序配置 |
| `publish` | 自动更新服务器地址 |
| `asar` | 是否将源码打包为 asar |

**Windows 安装包生成：**
```bash
npm install electron-builder --save-dev
npm run build && electron-builder --win
# 产物：release/ 目录下 .exe 文件
```

---

### 17.4.2 自动更新（electron-updater）

#### 知识点详解

**自动更新原理：**

```
┌──────────────────────────────────────────────────┐
│                  更新服务器                        │
│  https://update.example.com/releases/            │
│     ├── latest.yml                               │
│     ├── MyApp-2.0.0.exe                         │
│     └── MyApp-2.0.1.exe                         │
└──────────────────────┬───────────────────────────┘
                      │ HTTP 请求检查版本
                      ▼
┌──────────────────────────────────────────────────┐
│              Electron App                          │
│  electron-updater 检查 latest.yml                │
│  → 发现新版本 → 下载 → 重启安装                   │
└──────────────────────────────────────────────────┘
```

**latest.yml 配置（更新服务器）：**

```yaml
# 发布到 GitHub Releases 时自动生成（使用 electron-builder-github-releaser）
# 或手动创建
version: 2.0.1
files:
  - url: MyApp-2.0.1.exe
    sha512: <sha512-hash>
    size: 84235200
path: MyApp-2.0.1.exe
sha512: <sha512-hash>
releaseDate: '2026-04-02T00:00:00.000Z'
```

**主进程自动更新配置：**

```javascript
// main.js
const { autoUpdater } = require('electron-updater');
const log = require('electron-log');

// 配置日志
autoUpdater.logger = log;
autoUpdater.logger.transports.file.level = 'info';

// 自动更新配置
function setupAutoUpdater() {
    // 1. 检查更新（可在启动时或定期检查）
    autoUpdater.checkForUpdatesAndNotify().catch(err => {
        log.error('检查更新失败:', err);
    });

    // 2. 更新可用（发现新版本）
    autoUpdater.on('checking-for-update', () => {
        log.info('检查更新中...');
    });

    // 3. 发现更新（可以下载）
    autoUpdater.on('update-available', (info) => {
        log.info('发现新版本:', info.version);
        // 显示通知
        new Notification({
            title: '发现新版本',
            body: `版本 ${info.version} 可用，正在下载...`,
        }).show();
    });

    // 4. 下载进度
    autoUpdater.on('download-progress', (progressObj) => {
        const percent = Math.round(progressObj.percent);
        log.info(`下载进度: ${percent}%`);
        mainWindow?.webContents.send('update:progress', { percent });
    });

    // 5. 下载完成（准备安装）
    autoUpdater.on('update-downloaded', (info) => {
        log.info('下载完成:', info.version);
        // 询问用户是否立即重启安装
        new Notification({
            title: '更新就绪',
            body: '新版本已下载，重启应用以完成更新',
        }).show();

        // 立即安装并重启（可选）
        // autoUpdater.quitAndInstall();
    });

    // 6. 更新错误
    autoUpdater.on('error', (err) => {
        log.error('更新错误:', err);
    });

    // 7. 没有可用更新
    autoUpdater.on('update-not-available', () => {
        log.info('当前已是最新版本');
    });
}

// 在应用准备好时启用
app.whenReady().then(() => {
    setupAutoUpdater();
    createWindow();
});

// 让渲染进程触发更新检查（用户点击"检查更新"按钮时）
ipcMain.handle('app:check-update', async () => {
    return await autoUpdater.checkForUpdates();
});
```

**增量更新（差分包）：**

```json
// package.json build 配置
{
  "build": {
    "nsis": {
      "differentialPackage": true  // 启用差分包（仅下载变化部分）
    }
  }
}
```

**私域更新服务器（通用 HTTP 服务器）：**

```javascript
// 不依赖 GitHub，自己搭建更新服务器
const { autoUpdater } = require('electron-updater');

autoUpdater.updateConfigPath = path.join(__dirname, 'update-config.json');

autoUpdater.setFeedURL({
    provider: 'generic',
    url: 'https://update.mycompany.com/electron-app/releases/',
});
```

**手动更新（用户点击按钮触发）：**

```javascript
// 渲染进程
async function checkForUpdates() {
    const result = await window.electronAPI.invoke('app:check-update');
    if (!result) {
        alert('当前已是最新版本更新。');
    }
}

// 监听进度
window.electronAPI.on('update:progress', ({ percent }) => {
    updateProgressBar.value = percent;
});
```

---

## 17.5 Electron 性能优化

### 17.5.1 性能问题诊断与优化策略

#### 知识点详解

**Electron 性能问题主要来源：**

```
内存占用高：
- Chromium 渲染进程本身占用大量内存
- 每个 BrowserWindow 都会创建独立渲染进程
- 图片/视频等媒体资源未优化

启动速度慢：
- Chromium 完整启动需要时间
- 大量 npm 模块在 main.js 中同步加载
- 界面空白时间（用户看到白屏）

通信延迟：
- IPC 通信序列化开销
- 频繁的跨进程数据传递

CPU 占用高：
- 硬件加速（GPU 进程占用高）
- 动画和过渡效果未优化
- 频繁的 DOM 操作
```

**启动速度优化：**

```javascript
// main.js — 优化启动速度

// 1. 延迟加载不必要的模块（懒加载）
// ❌ 低效：模块在应用启动时全部加载
// const fs = require('fs');
// const crypto = require('crypto');
// const zlib = require('zlib');

// ✅ 高效：使用时再加载
function getFileSystem() {
    return require('fs').promises;
}

// 2. v8-compile-cache（缓存 V8 编译结果）
// 在 main.js 第一行添加
if (!process.env.ELECTRON_DISABLE_V8_COMPILE_CACHE) {
    require('v8-compile-cache');
}

// 3. 使用 asar 打包（加快文件读取）
// package.json 中配置
// "asar": true（默认）

// 4. 减少界面空白时间
const mainWindow = new BrowserWindow({
    show: false,  // 先隐藏
    backgroundColor: '#ffffff',
    titleBarStyle: 'default',
});

mainWindow.once('ready-to-show', () => {
    mainWindow.show();  // 等页面加载完成再显示
});

// 5. 延迟加载非核心窗口
function openSettings() {
    // 按需创建设置窗口，而不是一开始就创建
    if (!settingsWindow) {
        settingsWindow = new BrowserWindow({
            width: 600,
            height: 400,
            parent: mainWindow,
        });
        settingsWindow.loadURL(...);
    }
    settingsWindow.show();
}
```

**内存优化：**

```javascript
// 1. 监控内存使用
app.on('browser-window-created', (_, window) => {
    // 设置内存限制（防止单个窗口耗尽内存）
    window.webContents.on('crashed', (event) => {
        log.error('渲染进程崩溃');
    });
});

// 2. 定期清理无用资源
setInterval(() => {
    // 清理已关闭窗口的事件监听器
    // 强制垃圾回收（仅在开发调试时）
    if (process.env.NODE_ENV === 'development') {
        global.gc && global.gc();
    }
}, 60000);

// 3. 图片懒加载
// <img> 标签使用 loading="lazy" 或 Intersection Observer
// src/utils/lazyImage.js
function lazyLoadImages(container) {
    const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
            if (entry.isIntersecting) {
                const img = entry.target;
                img.src = img.dataset.src;  // data-src 存储真实 URL
                observer.unobserve(img);
            }
        });
    }, { rootMargin: '100px' });

    container.querySelectorAll('img[data-src]').forEach(img => {
        observer.observe(img);
    });
}

// 4. 使用 WebP 格式图片
// 在 HTML 中使用
// <picture>
//     <source srcset="image.webp" type="image/webp">
//     <img src="image.jpg" alt="">
// </picture>
```

**GPU 内存优化（硬件加速调优）：**

```javascript
// main.js
app.disableHardwareAcceleration();  // 如果应用不需要 GPU 加速，禁用以节省资源

// 或者只对特定窗口禁用
new BrowserWindow({
    webPreferences: {
        enableBlinkFeatures: 'GPU',  // 控制 GPU 特性
    },
});

// 限制 GPU 进程数量
app.commandLine.appendSwitch('gpu-process-limit', '1');
app.commandLine.appendSwitch('disable-gpu-sandbox');
```

**IPC 通信优化：**

```javascript
// preload.js — 批量 IPC 调用

// ❌ 低效：多次 IPC 调用
await window.electronAPI.invoke('get-user-name');
await window.electronAPI.invoke('get-user-email');
await window.electronAPI.invoke('get-user-avatar');

// ✅ 高效：一次 IPC 获取全部数据
ipcMain.handle('get-user-info', async () => {
    return {
        name: await getUserName(),
        email: await getUserEmail(),
        avatar: await getUserAvatar(),
    };
});

// 渲染进程一次调用
const userInfo = await window.electronAPI.invoke('get-user-info');
```

**Web Worker（避免主进程阻塞）：**

```javascript
// 渲染进程：使用 Worker 处理重计算
// worker.js
self.onmessage = function(e) {
    const result = heavyComputation(e.data);
    self.postMessage(result);
};

// renderer.js
const worker = new Worker('./worker.js');
worker.postMessage({ data: largeArray });
worker.onmessage = (e) => {
    console.log('计算结果:', e.data);
};

// 主进程：使用 Node.js Worker Threads
// main.js
const { Worker } = require('worker_threads');
const worker = new Worker('./compute-worker.js', {
    workerData: { input: largeData },
});
worker.on('message', (result) => {
    mainWindow.webContents.send('compute:result', result);
});
```

**DevTools 性能分析：**

```javascript
// 渲染进程 DevTools 性能分析
// Chrome DevTools → Performance 面板 → 录制用户操作 → 分析 FPS / JS 执行时间

// 主进程性能分析
// 使用 electron-devtools-installer 或 v8-profiler-node8
const v8Profiler = require('v8-profiler-node8');

// 在主进程中
v8Profiler.startProfiling('main-process');
// ... 执行操作 ...
const profile = v8Profiler.stopProfiling('main-process');
console.log(profile);
```

**Chrome DevTools 使用技巧：**

```
Electron DevTools 与 Chrome DevTools 使用方式完全相同：

1. 打开 DevTools
   - 开发环境：mainWindow.webContents.openDevTools()
   - 快捷键：Cmd/Ctrl + Option/Alt + I

2. Performance 面板：录制滚动、动画等操作，分析帧率
3. Memory 面板：Heap Snapshot 分析内存泄漏
4. Network 面板：查看资源加载情况
5. Lighthouse 面板：性能评分和优化建议
```

#### 真实面试题

**题目：Electron 应用常见的性能问题有哪些？如何优化启动速度和内存占用？**

**满分答案：**

**常见性能问题：**

| 问题 | 表现 | 原因 |
|------|------|------|
| 启动慢 | 白屏时间长 | 模块同步加载、Chromium 启动 |
| 内存占用高 | 多窗口时内存飙升 | Chromium 每个窗口独立进程 |
| CPU 占用高 | 风扇狂转 | 硬件加速、频繁动画 |
| 通信延迟 | 点击后响应慢 | IPC 通信开销 |
| 卡顿 | 滚动/动画掉帧 | 渲染进程阻塞 |

**启动速度优化（5 步）：**

```javascript
// 1. 延迟加载模块（最有效）
// ❌ const fs = require('fs');
// ✅ const getFS = () => require('fs');

// 2. v8-compile-cache 缓存 V8 编译结果
require('v8-compile-cache');

// 3. 先创建窗口但不立即显示
new BrowserWindow({ show: false });
window.once('ready-to-show', () => window.show());

// 4. 减少首次加载的资源体积（前端代码分割）
// 5. 使用 asar 打包
```

**内存优化（3 步）：**

```javascript
// 1. 图片优化：WebP 格式、懒加载
// 2. 监控内存：window.webContents.on('crashed')
// 3. 按需创建窗口（设置窗口延迟创建）
```

---

## 17.6 Electron CI/CD 自动化构建

### 17.6.1 GitHub Actions 多平台 CI 配置

#### 知识点详解

**Electron CI/CD 挑战：**

```
- macOS 应用需要 macOS 构建机器（Apple 硬件限制）
- Windows 应用需要 Windows 构建机器
- Linux 应用可在任意平台构建
- 代码签名（Code Signing）需要证书
```

**GitHub Actions 完整配置：**

```yaml
# .github/workflows/electron-ci.yml
name: Electron CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20.x'

jobs:
  # 1. 代码质量检查
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run tests
        run: npm test

  # 2. Windows 构建
  build-windows:
    needs: lint
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build frontend
        run: npm run build

      - name: Build Windows installer
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CSC_LINK: ${{ secrets.WINDOWS_CERTIFICATE }}
          CSC_KEY_PASSWORD: ${{ secrets.WINDOWS_CERTIFICATE_PASSWORD }}
        run: |
          npx electron-builder --win --config.win.certificateFile="$CSC_LINK"
          ls release/

      - name: Upload Windows installer
        uses: actions/upload-artifact@v4
        with:
          name: windows-installer
          path: release/*.exe
          retention-days: 30

  # 3. macOS 构建
  build-macos:
    needs: lint
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build frontend
        run: npm run build

      - name: Build macOS dmg
        env:
          CSC_LINK: ${{ secrets.MACOS_CERTIFICATE }}
          CSC_KEY_PASSWORD: ${{ secrets.MACOS_CERTIFICATE_PASSWORD }}
          APPLE_ID: ${{ secrets.APPLE_ID }}
          APPLE_APP_PASSWORD: ${{ secrets.APPLE_APP_PASSWORD }}
          APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
        run: |
          echo "$CSC_LINK" | base64 --decode > ./cert.p12
          npx electron-builder --mac --config.mac.certificate="$PWD/cert.p12"
          ls release/

      - name: Upload macOS dmg
        uses: actions/upload-artifact@v4
        with:
          name: macos-dmg
          path: release/*.dmg
          retention-days: 30

  # 4. Linux 构建
  build-linux:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build frontend
        run: npm run build

      - name: Build Linux AppImage
        run: npx electron-builder --linux

      - name: Upload Linux AppImage
        uses: actions/upload-artifact@v4
        with:
          name: linux-appimage
          path: release/*.AppImage
          retention-days: 30

  # 5. GitHub Releases 发布
  release:
    needs: [build-windows, build-macos, build-linux]
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts

      - name: Generate Release Notes
        run: |
          echo "## Changes in ${{ github.ref }}" > RELEASE_NOTES.md
          git log --oneline ${{ github.ref_name }}..HEAD >> RELEASE_NOTES.md

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          body_path: RELEASE_NOTES.md
          draft: false
          prerelease: ${{ contains(github.ref, 'beta') || contains(github.ref, 'alpha') }}
          files: |
            artifacts/windows-installer/*.exe
            artifacts/macos-dmg/*.dmg
            artifacts/linux-appimage/*.AppImage
          generate_release_notes: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**代码签名（Code Signing）：**

```yaml
# Windows 代码签名（使用证书 .p12 或 .pfx）
# secrets 设置：
#   WINDOWS_CERTIFICATE: base64 编码的证书文件
#   WINDOWS_CERTIFICATE_PASSWORD: 证书密码

# macOS 代码签名（需要 Apple Developer 证书）
# secrets 设置：
#   MACOS_CERTIFICATE: base64 编码的 .p12 文件
#   MACOS_CERTIFICATE_PASSWORD: 证书密码
#   APPLE_ID: Apple ID 邮箱
#   APPLE_APP_PASSWORD: 应用专用密码
#   APPLE_TEAM_ID: Team ID

# 证书转换为 base64（在本地 Mac 执行）
# base64 -i Certificates.p12 -o encoded.txt
# 将 encoded.txt 内容复制到 GitHub Secrets
```

**自动更新触发 CI：**

```yaml
# .github/workflows/release.yml
# 当创建 GitHub Tag 时自动构建并上传到更新服务器

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build all platforms
        run: |
          npm ci
          npm run build
          npx electron-builder --win --mac --linux

      - name: Upload to update server
        run: |
          # 上传到私有更新服务器
          curl -u ${{ secrets.UPDATE_SERVER_USER }}:${{ secrets.UPDATE_SERVER_PASS }} \
            -T release/*.exe \
            https://update.example.com/releases/
```

#### 真实面试题

**题目：如何配置 Electron 应用的 CI/CD？构建 macOS 应用有哪些特殊要求？**

**满分答案：**

**Electron CI/CD 配置要点：**

```yaml
# 关键步骤：
1. npm ci（依赖安装，带缓存）
2. npm run build（前端构建）
3. electron-builder（跨平台打包）
4. 代码签名（保证安全性）
5. 上传 artifact（保存构建产物）
```

**macOS 构建特殊要求：**

| 要求 | 说明 |
|------|------|
| macOS runner | 必须在 macOS 机器上构建（Apple 硬件限制） |
| Apple Developer 证书 | 需申请 Apple Developer 账号 |
| 代码签名 | 使用 p12 证书对 .app / .dmg 签名 |
|  notarization | 需上传到 Apple 进行公证（自动通过 `xcrun notarytool`） |

**Windows 代码签名流程：**
```bash
# 本地生成证书
openssl pkcs12 -export -in certificate.cer -inkey privateKey.key -out codesign.p12

# 上传到 GitHub Secrets（base64 编码）
base64 -i codesign.p12 -o encoded.txt

# GitHub Actions 自动使用证书签名
npx electron-builder --win --config.win.certificateFile="$CSC_LINK"
```

---

## 17.7 Electron 桌面特性与系统集成

### 17.7.1 窗口管理、系统托盘与原生能力

#### 知识点详解

**窗口管理：**

```javascript
// main.js

// 创建无边框自定义窗口
const framelessWindow = new BrowserWindow({
    frame: false,           // 隐藏原生标题栏
    transparent: true,      // 透明背景
    titleBarStyle: 'hidden', // macOS 隐藏标题栏但保留窗口控件
    trafficLightPosition: { x: 15, y: 15 },  // macOS 关闭按钮位置

    // 自定义窗口控件（渲染进程中通过 IPC 调用）
    webPreferences: {
        preload: path.join(__dirname, 'preload.js'),
    },
});

// 窗口控制（渲染进程触发）
ipcMain.handle('window:minimize', (event) => {
    BrowserWindow.fromWebContents(event.sender)?.minimize();
});

ipcMain.handle('window:maximize', (event) => {
    const win = BrowserWindow.fromWebContents(event.sender);
    if (win?.isMaximized()) {
        win.unmaximize();
    } else {
        win?.maximize();
    }
});

ipcMain.handle('window:close', (event) => {
    BrowserWindow.fromWebContents(event.sender)?.close();
});

// 监听最大化状态变化
win.on('maximize', () => {
    win.webContents.send('window:maximized', true);
});

win.on('unmaximize', () => {
    win.webContents.send('window:maximized', false);
});
```

**系统托盘（System Tray）：**

```javascript
// main.js
const { Tray, Menu, nativeImage } = require('electron');
let tray = null;

function createTray() {
    // 创建托盘图标
    const icon = nativeImage.createFromPath(
        path.join(__dirname, '../build/icon.png')
    );
    tray = new Tray(icon.resize({ width: 16, height: 16 }));

    const contextMenu = Menu.buildFromTemplate([
        {
            label: '打开应用',
            click: () => {
                mainWindow?.show();
                mainWindow?.focus();
            },
        },
        { type: 'separator' },
        {
            label: '设置',
            click: () => openSettingsWindow(),
        },
        { type: 'separator' },
        {
            label: '关于',
            click: () => showAboutDialog(),
        },
        {
            label: '退出',
            click: () => {
                app.isQuitting = true;
                app.quit();
            },
        },
    ]);

    tray.setToolTip('My Electron App');
    tray.setContextMenu(contextMenu);

    // 点击托盘图标：显示/隐藏主窗口
    tray.on('click', () => {
        if (mainWindow?.isVisible()) {
            mainWindow.hide();
        } else {
            mainWindow?.show();
            mainWindow?.focus();
        }
    });
}
```

**原生菜单栏：**

```javascript
// main.js
function createMenu() {
    const template = [
        {
            label: '文件',
            submenu: [
                {
                    label: '新建窗口',
                    accelerator: 'CmdOrCtrl+N',
                    click: () => createNewWindow(),
                },
                {
                    label: '打开文件',
                    accelerator: 'CmdOrCtrl+O',
                    click: () => openFileDialog(),
                },
                { type: 'separator' },
                {
                    label: '退出',
                    accelerator: 'CmdOrCtrl+Q',
                    click: () => app.quit(),
                },
            ],
        },
        {
            label: '编辑',
            submenu: [
                { role: 'undo' },
                { role: 'redo' },
                { type: 'separator' },
                { role: 'cut' },
                { role: 'copy' },
                { role: 'paste' },
                { role: 'selectAll' },
            ],
        },
        {
            label: '视图',
            submenu: [
                { role: 'reload' },
                { role: 'forceReload' },
                { role: 'toggleDevTools' },
                { type: 'separator' },
                { role: 'resetZoom' },
                { role: 'zoomIn' },
                { role: 'zoomOut' },
                { type: 'separator' },
                { role: 'togglefullscreen' },
            ],
        },
        {
            label: '窗口',
            submenu: [
                { role: 'minimize' },
                { role: 'zoom' },
                { type: 'separator' },
                {
                    label: '前置全部窗口',
                    role: 'front',
                },
            ],
        },
        {
            label: '帮助',
            submenu: [
                {
                    label: '关于',
                    click: () => showAboutDialog(),
                },
            ],
        },
    ];

    const menu = Menu.buildFromTemplate(template);
    Menu.setApplicationMenu(menu);
}
```

**全局快捷键：**

```javascript
// main.js
const { globalShortcut } = require('electron');

app.whenReady().then(() => {
    // 注册全局快捷键（即使应用不在前台也可响应）
    globalShortcut.register('CommandOrControl+Shift+D', () => {
        // 显示 DevTools
        const focusedWindow = BrowserWindow.getFocusedWindow();
        focusedWindow?.webContents.toggleDevTools();
    });

    globalShortcut.register('CommandOrControl+Shift+M', () => {
        // 切换窗口显示/隐藏
        if (mainWindow?.isVisible()) {
            mainWindow.hide();
        } else {
            mainWindow?.show();
            mainWindow?.focus();
        }
    });
});

app.on('will-quit', () => {
    // 取消注册所有全局快捷键
    globalShortcut.unregisterAll();
});
```

**系统通知（Notification）：**

```javascript
// main.js
const { Notification } = require('electron');

function showNotification(title, body) {
    if (Notification.isSupported()) {
        const notification = new Notification({
            title,
            body,
            silent: false,
            icon: path.join(__dirname, '../build/icon.png'),
            timeoutType: 'default',  // 'default' | 'never'
        });
        notification.show();
    }
}

// 渲染进程通过 IPC 调用
ipcMain.handle('notification:show', (_, { title, body }) => {
    showNotification(title, body);
});
```

**系统对话框：**

```javascript
// main.js
const { dialog } = require('electron');

ipcMain.handle('dialog:openFile', async () => {
    const result = await dialog.showOpenDialog({
        title: '选择文件',
        defaultPath: app.getPath('documents'),
        properties: ['openFile'],
        filters: [
            { name: 'Images', extensions: ['jpg', 'png', 'gif'] },
            { name: 'All Files', extensions: ['*'] },
        ],
    });
    return result;  // { canceled, filePaths }
});

ipcMain.handle('dialog:saveFile', async (_, defaultName) => {
    const result = await dialog.showSaveDialog({
        title: '保存文件',
        defaultPath: defaultName,
        filters: [
            { name: 'Text Files', extensions: ['txt'] },
            { name: 'JSON Files', extensions: ['json'] },
        ],
    });
    return result;  // { canceled, filePath }
});

ipcMain.handle('dialog:confirm', async (_, message) => {
    const result = await dialog.showMessageBox({
        type: 'question',
        buttons: ['是', '否', '取消'],
        defaultId: 0,
        title: '确认',
        message,
    });
    return result.response;  // 0=是, 1=否, 2=取消
});
```

#### 真实面试题

**题目：Electron 应用有哪些桌面特有能力？如何实现系统托盘、窗口管理和原生菜单栏？**

**满分答案：**

**桌面特有能力：**

| 能力 | API | 说明 |
|------|-----|------|
| 系统托盘 | `Tray` | 后台运行，托盘图标 |
| 原生菜单 | `Menu` | 应用菜单栏 |
| 全局快捷键 | `globalShortcut` | 应用外也可响应 |
| 系统通知 | `Notification` | 原生桌面通知 |
| 文件对话框 | `dialog` | 原生打开/保存对话框 |
| 拖拽文件 | `ondragover` | 原生文件拖拽 |
| 剪切板 | `clipboard` | 读写系统剪切板 |
| 系统信息 | `app.getPath` | 获取系统路径 |

**窗口管理：**
```javascript
// 最小化、最大化/还原、关闭
ipcMain.handle('window:minimize', ...)
ipcMain.handle('window:maximize', ...)

// 无边框窗口（自定义标题栏）
new BrowserWindow({ frame: false, ... })
```

**系统托盘：**
```javascript
const tray = new Tray(icon);
tray.setContextMenu(Menu.buildFromTemplate([...]));
tray.on('click', () => showWindow());
```

---

## 17.8 Electron 与 Tauri 对比选型

### 17.8.1 技术选型决策

#### 知识点详解

**Electron vs Tauri 完整对比：**

| 维度 | Electron | Tauri |
|------|---------|-------|
| 底层引擎 | Chromium + Node.js | Rust + 系统 WebView |
| 包体积（Windows） | 80-200 MB | 5-15 MB |
| 内存占用 | 较高 | 极低 |
| 启动速度 | 慢（2-5s） | 快（0.5-1s） |
| 开发体验 | 成熟（VS Code 等大型应用验证） | 快速成长 |
| 前端技术 | React/Vue/Angular 均可 | React/Vue/Angular 均可 |
| 原生能力 | 完整（Node.js 所有模块） | 通过 Rust FFI 扩展 |
| 安全模型 | 需手动配置 | Rust 安全特性内置 |
| 生态 | 成熟（npm 包 + 大量桌面框架） | 成长中（Rust 生态） |
| 代码签名 | 复杂（多平台） | 简单（Rust 工具链） |
| 自动更新 | electron-updater | Tauri updater |
| 适用场景 | VS Code、Slack 等大型应用 | 轻量工具、替代 Web |

**选型决策树：**

```
应用类型？
    │
    ├── 大型 IDE / 专业工具（VS Code 级别）
    │       → Electron（成熟生态、复杂 API 支持）
    │
    ├── 轻量工具 / 辅助软件（替代 Web）
    │       → Tauri（包体积小、启动快）
    │
    ├── 已有 Electron 项目
    │       → Electron（迁移成本高）
    │
    ├── 团队有 Rust 能力
    │       → Tauri（性能优势 + Rust 安全）
    │
    └── 需要 Node.js 原生模块（sharp / sqlite 等）
            → Electron（Node.js 生态无缝集成）
```

**Electron 项目迁移到 Tauri 评估：**

```javascript
// 评估清单
const migrationChecklist = [
    { item: 'Node.js 原生模块', status: 'hard', note: '需用 Rust 重写或通过 FFI 调用' },
    { item: 'Electron 特有 API', status: 'medium', note: 'Tray / Menu / Notification 均有对应 API' },
    { item: 'IPC 通信模式', status: 'easy', note: 'Tauri Command 类似 invoke/handle' },
    { item: '前端代码', status: 'easy', note: '直接复用，无需修改' },
    { item: '打包配置', status: 'medium', note: 'electron-builder → tauri.conf.json' },
];
```

#### 真实面试题

**题目：Electron 和 Tauri 有什么区别？什么场景下应该选择 Tauri 而不是 Electron？**

**满分答案：**

**核心区别：**

| 维度 | Electron | Tauri |
|------|---------|-------|
| 体积 | 80-200MB | 5-15MB（Rust 编译产物极小） |
| 内存 | 高 | 极低 |
| 启动 | 慢 | 快 |
| 底层 | Chromium | 系统 WebView |
| 扩展语言 | Node.js | Rust |
| 生态 | 成熟 | 成长中 |

**选择 Tauri 的场景：**
- 轻量工具应用（笔记、辅助工具）
- 对安装包体积敏感（< 20MB）
- 追求极快启动速度
- 团队有 Rust 能力

**选择 Electron 的场景：**
- VS Code 这类大型复杂应用
- 需要大量 Node.js 原生模块
- 已有 Electron 项目（迁移成本高）
- 需要最深度的系统集成能力

**实际案例：**
- Postman（Electron）：API 调试工具，需复杂网络和 Node.js 原生能力
- Obsidian（Tauri）：笔记应用，追求轻量和快速启动
- VS Code（Electron）：大型 IDE，生态成熟，功能复杂
CONTENT
echo "✅ Electron 内容写入完成"