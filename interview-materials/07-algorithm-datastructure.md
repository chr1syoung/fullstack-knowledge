# 算法与数据结构

> 面试频率：⭐⭐⭐⭐ | 大厂必考

---

## 一、必刷题型汇总

| 题型 | 难度 | 必刷题 |
|------|------|--------|
| 二分查找 | ⭐⭐ | 704、35、34、153 |
| 双指针 | ⭐⭐ | 977、283、26、15 |
| 滑动窗口 | ⭐⭐ | 3、209、438 |
| BFS/DFS | ⭐⭐⭐ | 102、104、200、547 |
| 动态规划 | ⭐⭐⭐ | 70、322、198、1143 |
| 排序 | ⭐⭐ | 912、215、75 |
| 链表 | ⭐⭐ | 206、21、141、19 |
| 二叉树 | ⭐⭐ | 94、102、104、226 |
| 栈/队列 | ⭐⭐ | 20、232、225 |

---

## 二、数组

### 2.1 二分查找

```js
// 标准二分
function binarySearch(nums, target) {
  let left = 0, right = nums.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (nums[mid] === target) return mid;
    else if (nums[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}

// 查找左边界
function leftBound(nums, target) {
  let left = 0, right = nums.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (nums[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return left;  // left 是左边界
}

// 旋转数组查找
function search(nums, target) {
  let left = 0, right = nums.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (nums[mid] === target) return mid;
    
    // 左边有序
    if (nums[left] <= nums[mid]) {
      if (target >= nums[left] && target < nums[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    } 
    // 右边有序
    else {
      if (target > nums[mid] && target <= nums[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }
  return -1;
}
```

### 2.2 双指针

```js
// 有序数组平方（977）
function sortedSquares(nums) {
  const result = new Array(nums.length);
  let left = 0, right = nums.length - 1, pos = nums.length - 1;
  
  while (left <= right) {
    if (Math.abs(nums[left]) > Math.abs(nums[right])) {
      result[pos] = nums[left] * nums[left];
      left++;
    } else {
      result[pos] = nums[right] * nums[right];
      right--;
    }
    pos--;
  }
  return result;
}

// 移动零（283）
function moveZeroes(nums) {
  let slow = 0;
  for (let fast = 0; fast < nums.length; fast++) {
    if (nums[fast] !== 0) {
      [nums[slow], nums[fast]] = [nums[fast], nums[slow]];
      slow++;
    }
  }
}

// 三数之和（15）
function threeSum(nums) {
  const result = [];
  nums.sort((a, b) => a - b);
  
  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    
    let left = i + 1, right = nums.length - 1;
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;
        left++;
        right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }
  return result;
}
```

### 2.3 滑动窗口

```js
// 最小子数组（209）
function minSubArrayLen(target, nums) {
  let minLen = Infinity;
  let sum = 0;
  let left = 0;
  
  for (let right = 0; right < nums.length; right++) {
    sum += nums[right];
    
    while (sum >= target) {
      minLen = Math.min(minLen, right - left + 1);
      sum -= nums[left];
      left++;
    }
  }
  return minLen === Infinity ? 0 : minLen;
}

// 字符串排列（567）
function checkInclusion(s1, s2) {
  const need = new Map();
  const window = new Map();
  
  for (const c of s1) {
    need.set(c, (need.get(c) || 0) + 1);
  }
  
  let left = 0, right = 0;
  let valid = 0;
  
  while (right < s2.length) {
    const c = s2[right];
    right++;
    
    if (need.has(c)) {
      window.set(c, (window.get(c) || 0) + 1);
      if (window.get(c) === need.get(c)) valid++;
    }
    
    while (right - left >= s1.length) {
      if (valid === need.size) return true;
      
      const d = s2[left];
      left++;
      
      if (need.has(d)) {
        if (window.get(d) === need.get(d)) valid--;
        window.set(d, window.get(d) - 1);
      }
    }
  }
  return false;
}
```

---

## 三、链表

### 3.1 反转链表（206）

```js
function reverseList(head) {
  let prev = null;
  let curr = head;
  
  while (curr) {
    const next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }
  return prev;
}

// 递归版本
function reverseListRec(head) {
  if (!head || !head.next) return head;
  const newHead = reverseListRec(head.next);
  head.next.next = head;
  head.next = null;
  return newHead;
}
```

### 3.2 环形链表检测（141）

```js
function hasCycle(head) {
  let slow = head, fast = head;
  
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    
    if (slow === fast) return true;
  }
  return false;
}
```

### 3.3 删除倒数第 N 个节点（19）

```js
function removeNthFromEnd(head, n) {
  const dummy = new ListNode(0);
  dummy.next = head;
  
  let slow = dummy, fast = dummy;
  
  for (let i = 0; i <= n; i++) {
    fast = fast.next;
  }
  
  while (fast) {
    slow = slow.next;
    fast = fast.next;
  }
  
  slow.next = slow.next.next;
  return dummy.next;
}
```

---

## 四、二叉树

### 4.1 遍历

```js
// 前序遍历（递归）
function preorder(root, res = []) {
  if (!root) return res;
  res.push(root.val);
  preorder(root.left, res);
  preorder(root.right, res);
  return res;
}

// 中序遍历（递归）
function inorder(root, res = []) {
  if (!root) return res;
  inorder(root.left, res);
  res.push(root.val);
  inorder(root.right, res);
  return res;
}

// 后序遍历（递归）
function postorder(root, res = []) {
  if (!root) return res;
  postorder(root.left, res);
  postorder(root.right, res);
  res.push(root.val);
  return res;
}

// 层序遍历（BFS）
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];
  
  while (queue.length) {
    const level = [];
    const len = queue.length;
    
    for (let i = 0; i < len; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
```

### 4.2 最大深度（104）

```js
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

### 4.3 对称二叉树（101）

```js
function isSymmetric(root) {
  function check(left, right) {
    if (!left && !right) return true;
    if (!left || !right) return false;
    return left.val === right.val && 
           check(left.left, right.right) && 
           check(left.right, right.left);
  }
  return check(root.left, root.right);
}
```

---

## 五、动态规划

### 5.1 斐波那契 / 爬楼梯（70）

```js
function climb(n) {
  if (n <= 2) return n;
  
  let dp1 = 1, dp2 = 2;
  for (let i = 3; i <= n; i++) {
    const temp = dp1 + dp2;
    dp1 = dp2;
    dp2 = temp;
  }
  return dp2;
}
```

### 5.2 零钱兑换（322）

```js
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  
  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i && dp[i - coin] + 1 < dp[i]) {
        dp[i] = dp[i - coin] + 1;
      }
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

### 5.3 最长公共子序列（1143）

```js
function longestCommonSubsequence(text1, text2) {
  const m = text1.length, n = text2.length;
  const dp = new Array(m + 1).fill(0).map(() => new Array(n + 1).fill(0));
  
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[m][n];
}
```

---

## 六、排序

### 6.1 快速排序

```js
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (left >= right) return;
  
  const pivot = partition(arr, left, right);
  quickSort(arr, left, pivot - 1);
  quickSort(arr, pivot + 1, right);
  return arr;
}

function partition(arr, left, right) {
  const pivot = arr[right];
  let i = left - 1;
  
  for (let j = left; j < right; j++) {
    if (arr[j] <= pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  [arr[i + 1], arr[right]] = [arr[right], arr[i + 1]];
  return i + 1;
}
```

### 6.2 归并排序

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }
  return result.concat(left.slice(i)).concat(right.slice(j));
}
```

---

## 七、栈与队列

### 7.1 有效括号（20）

```js
function isValid(s) {
  const stack = [];
  const map = { ')': '(', ']': '[', '}': '{' };
  
  for (const c of s) {
    if (c in map) {
      if (stack.pop() !== map[c]) return false;
    } else {
      stack.push(c);
    }
  }
  return stack.length === 0;
}
```

### 7.2 用栈实现队列（232）

```js
class MyQueue {
  constructor() {
    this.stackIn = [];
    this.stackOut = [];
  }
  
  push(x) {
    this.stackIn.push(x);
  }
  
  pop() {
    if (this.stackOut.length === 0) {
      while (this.stackIn.length) {
        this.stackOut.push(this.stackIn.pop());
      }
    }
    return this.stackOut.pop();
  }
  
  peek() {
    if (this.stackOut.length === 0) {
      while (this.stackIn.length) {
        this.stackOut.push(this.stackIn.pop());
      }
    }
    return this.stackOut[this.stackOut.length - 1];
  }
  
  empty() {
    return this.stackIn.length === 0 && this.stackOut.length === 0;
  }
}
```

---

## 八、必刷资源推荐

| 资源 | 说明 |
|------|------|
| **LeetCode Hot 100** | 必刷清单 |
| **代码随想录** | 中文最优解详解 |
| **NeetCode 150** | 系统性刷题路线 |
| **剑指 Offer** | 面试高频题 |

---

> 📌 下一章：[场景题](./08-scenario-questions.md)
