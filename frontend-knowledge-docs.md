# 前端知识库文档规范

> AI 添加/修改内容时必须遵循的格式指南

**相关文档：**
- 面经处理完整流程 → 见 [`INTERVIEW-SOP.md`](./INTERVIEW-SOP.md)

---

## 一、项目结构

```
frontend-knowledge/
├── 前端开发知识库/           # 知识点详解 (14个文件)
│   ├── 01-HTML-CSS基础.md
│   ├── 02-JavaScript基础与进阶.md
│   ├── 03-TypeScript详解.md
│   ├── 04-Vue框架详解.md
│   ├── 05-React框架详解.md
│   ├── 06-Node.js详解.md
│   ├── 07-工程化与构建工具.md
│   ├── 08-计算机基础.md
│   ├── 09-性能优化.md
│   ├── 10-Web安全.md
│   └── 11-14-图形可视化移动端软技能新技术.md
│
├── interview-materials/      # 面经题库 (9个文件)
│   ├── 01-js-fundamentals.md
│   ├── 02-react.md
│   ├── 03-vue.md
│   ├── 04-network-browser.md
│   ├── 05-coding-handwrite.md
│   ├── 06-engineering-performance.md
│   ├── 07-algorithm-datastructure.md
│   └── 08-scenario-questions.md
│
└── 前端开发工程师知识点大全.md   # 总大纲
```

---

## 二、文档内容组织

### 每个技术领域的文件结构：

```
## X.X 主题名称

### X.X.X 知识点名称

#### 知识点详解
（概念解释 + 代码示例 + 原理分析）

#### 真实面试题
**题目N：**[具体问题]

**满分答案：**
[详细回答，包含：]
1. 核心要点分点说明
2. 代码示例（必要时）
3. 延伸思考或最佳实践
```

---

## 三、添加新内容的规则

### 1. 知识点讲解格式

```markdown
#### 知识点详解

**概念名称：**
[一句话概括核心定义]

**基本用法：**
```javascript
// 代码示例
const example = 'code here';
```

**原理分析：**
[为什么这样工作，关键点]

**常见误区：**
- 误区1：...
- 误区2：...
```

### 2. 面试题格式

```markdown
#### 真实面试题

**题目：Vue3 的 ref 和 reactive 有什么区别？**

**满分答案：**

**区别：**

1. **第一点区别**
   - 详细说明
   - 代码示例

2. **第二点区别**
   - 详细说明
   - 代码示例

**选择建议：**
- 推荐做法
- 适用场景

**延伸问题：**
- 相关问题提示
```

### 3. 内容比例

- **知识点讲解** : **面试题** = **6** : **4**
- 每个知识点至少配 1 道面试题
- 面试题要真实（来自实际面试场）

---

## 四、格式规范

### 4.1 标题层级

```markdown
## 一级标题（章） - 主题
### 二级标题（节）- 大知识点
#### 三级标题 - 小知识点
##### 四级标题 - 细节
```

### 4.2 代码块

- 使用 fenced code block（```）
- 标注语言类型：```javascript, ```vue, ```css
- 代码注释要清晰

### 4.3 列表

- 使用 `-` 而非 `*`
- 缩进 2 个空格表示层级
- 列表项用中文句号结尾

### 4.4 强调

- **粗体** 用于关键概念
- 代码用反引号 `code`

### 4.5 标点

- 使用中文标点（，。：；？！""）
- 英文术语两边保留空格

---

## 五、添加新内容的检查清单

- [ ] 先讲解知识点（概念 + 原理 + 示例）
- [ ] 再加面试题（题目 + 满分答案 + 要点）
- [ ] 代码示例完整可运行
- [ ] 与现有文档风格一致
- [ ] 检查是否有重复内容
- [ ] 更新对应的大纲文件

---

## 六、示例：添加 "Pinia 状态管理"

```markdown
## 4.X Pinia 状态管理

### 4.X.1 Pinia 基本使用

#### 知识点详解

**Pinia 是 Vue 3 的官方状态管理库**，用于集中管理组件状态。

**为什么选择 Pinia：**
- 更轻量（约 1KB）
- 支持 TypeScript
- 模块化设计
- 无需 mutations

**基本用法：**

```javascript
// stores/counter.js
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount: (state) => state.count * 2
  },
  actions: {
    increment() {
      this.count++;
    }
  }
});
```

**核心概念：**
- `state`：状态（相当于 data）
- `getters`：计算属性（相当于 computed）
- `actions`：方法（相当于 methods）

#### 真实面试题

**题目：Pinia 相比 Vuex 有哪些改进？**

**满分答案：**

**1. 极简设计**
- 去除了 mutations，需要异步操作直接调用 actions
- 不再需要 mapState、mapGetters 等辅助函数

**2. TypeScript 支持**
- 完美的类型推断
- 不需要手动类型声明

**3. 模块化**
- 每个 store 都是独立的模块
- 无需手动注册模块

**4. 无嵌套结构**
- 平铺式设计，避免深度嵌套

**代码对比：**

```javascript
// Vuex
this.$store.commit('increment');
this.$store.state.module.count;

// Pinia
store.increment();
store.count;
```

**选择建议：**
- 新项目直接用 Pinia
- Vue2 项目可用 Vuex 或 pinia-plugin-piniajs
```

---

## 七、注意事项

1. **不重复**：添加前先搜索，确认没有相同内容
2. **注明来源**：面试题可标注来自哪家公司/年份
3. **保持更新**：涉及版本特性要说明版本号
4. **实际代码**：示例代码要可运行，不要伪代码
5. **图文并茂**：复杂概念可添加流程图或表格

---
