# Company frequently asked questions

## Array & String

### [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)

```js
var reverseWords = function (s) {
  return s
    .trim()
    .split(" ")
    .filter((w) => w !== "")
    .reverse()
    .join(" ");
};
```

Follow up: Could you solve it in-place with O(1) extra space? => not gonna happen in js, since string cannot be modified directly in js.

先 reverse 整个字符串，再逐个单词 reverse 回来。

```js
var reverseWords = function (s) {
  if (s.length < 1) return s;
  let arr = reverse(s.split(""), 0, s.length - 1),
    idx = 0; // imagine given s is a char array
  for (let i = 0; i < s.length; i++) {
    if (arr[i] !== " ") {
      if (idx !== 0) arr[idx++] = " ";
      let j = i;
      while (j < s.length && arr[j] !== " ") arr[idx++] = arr[j++];
      reverse(arr, idx - (j - i), idx - 1);
      i = j;
    }
  }
  return arr.slice(0, idx).join("");
};

function reverse(s, i, j) {
  while (i < j) {
    let temp = s[i];
    s[i] = s[j];
    s[j] = temp;
    i++;
    j--;
  }
  return s;
}
```

### [Reverse Words in a String II](https://github.com/grandyang/leetcode/issues/186)

[Reverse Words in a String III](https://leetcode.com/problems/reverse-words-in-a-string-iii/)

不用处理首尾、及中间多余的空格。

```js
var reverseWords = function (s) {
  if (s.length < 1) return s;
  let arr = s.split(""),
    idx = 0;
  for (let i = 0; i <= arr.length; i++) {
    if (i === arr.length || arr[i] === " ") {
      reverse(arr, idx, i - 1);
      idx = i + 1;
    }
  }
  return reverse(arr, 0, arr.length - 1).join("");
};
```

### [Two Sum](https://leetcode.com/problems/two-sum/)

```js
var twoSum = function (nums, target) {
  let map = {};
  for (let i = 0; i < nums.length; i++) {
    let res = target - nums[i];
    if (res in map) {
      return [map[res], i];
    }
    map[nums[i]] = i;
  }
  return null;
};
```

### [Word Search](https://leetcode.com/problems/word-search/)

see in dfs.

### [First Missing Positive](https://leetcode.com/problems/first-missing-positive/submissions/)

see in array.

### [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)

see in array.

### [Longest Substring with At Most K Distinct Characters](https://github.com/grandyang/leetcode/issues/340)

see in two pointers - sliding window.

### [Remove Comments](https://leetcode.com/problems/remove-comments/)

see in string.

### [String Compression](https://leetcode.com/problems/string-compression/)

```js
var compress = function (chars) {
  let res = 0,
    idx = 0;
  while (idx < chars.length) {
    let ch = chars[idx],
      count = 0;
    while (idx < chars.length && chars[idx] === ch) {
      idx++;
      count++;
    }
    chars[res++] = ch;
    if (count !== 1) {
      count
        .toString()
        .split("")
        .forEach((c) => (chars[res++] = c));
    }
  }
  return res;
};
```

### [Valid Tic-Tac-Toe State](https://leetcode.com/problems/valid-tic-tac-toe-state/)

if x != o, and o + 1 != x, it is invalid;
if exists one line of x = 3, but o != x - 1, it's invalid;
if exists one line of o = 3, but x != o, it's invalid;

```js
var validTicTacToe = function (board) {
  let rows = [0, 0, 0],
    cols = [0, 0, 0],
    diag = 0,
    revDiag = 0,
    x = 0,
    o = 0;
  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      let count = 0;
      if (board[i][j] === "X") {
        count = 1;
        x++;
      } else if (board[i][j] === "O") {
        count = -1;
        o++;
      }
      rows[i] += count;
      cols[j] += count;
      if (i === j) diag += count;
      if (i === 2 - j) revDiag += count;
    }
  }
  if (x !== o && x - 1 !== o) return false;
  if (
    (rows.some((r) => r === 3) ||
      cols.some((c) => c === 3) ||
      diag === 3 ||
      revDiag === 3) &&
    o !== x - 1
  )
    return false;
  if (
    (rows.some((r) => r === -3) ||
      cols.some((c) => c === -3) ||
      diag === -3 ||
      revDiag === -3) &&
    x !== o
  )
    return false;
  return true;
};
```

### [String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)

remove leading spaces;
check first sign, skip if is - or +;
skip leading 0;
iterate ch until meet a ch is not 0-9;
for each char, multiple 10;
if sign is +, and res is bigger than 2^31 - 1, return 2^31 - 1;
if sign is -, and res is bigger than 2^31, return -2^31;
return res with sign.

```js
var myAtoi = function (s) {
  let i = 0,
    res = 0,
    positive = true;
  while (s[i] === " ") i++;
  if (s[i] === "-") positive = false;
  if (s[i] === "-" || s[i] === "+") i++;
  while (s[i] === "0") i++;
  while (/[0-9]/.test(s[i])) {
    res = res * 10 + Number(s[i++]);
    if (positive && res > 2147483647) return 2147483647;
    if (!positive && res > 2147483648) return -2147483648;
  }
  return positive ? res : -res;
};
```

### [Rectangle Overlap](https://leetcode.com/problems/rectangle-overlap/)

```js
function isLine(x1, y1, x2, y2) {
  return x1 == x2 || y1 == y2;
}

var isRectangleOverlap = function (rec1, rec2) {
  if (isLine(...rec1) || isLine(...rec2)) return false;
  return !(
    rec1[2] <= rec2[0] ||
    rec1[3] <= rec2[1] ||
    rec1[0] >= rec2[2] ||
    rec1[1] >= rec2[3]
  );
};
```

### [Pancake Sorting](https://leetcode.com/problems/pancake-sorting/)

place elements from end of the array, loop from n => 1
Find the index i of the number x.
Reverse i + 1 numbers, so that the x will be at A[0]
Reverse x numbers, so that x will be at A[x - 1].
Repeat this process until all numbers are in the right place.

```js
var pancakeSort = function (arr) {
  let res = [];
  for (let i = arr.length; i > 0; i--) {
    let j = 0;
    while (arr[j] !== i) j++;
    reverse(arr, j);
    res.push(j + 1);
    reverse(arr, i - 1);
    res.push(i);
  }
  return res;
};

function reverse(arr, pos) {
  for (let i = 0, j = pos; i < j; i++, j--) {
    let t = arr[i];
    arr[i] = arr[j];
    arr[j] = t;
  }
}
```

## Graph

### [Find the Celebrity](https://github.com/grandyang/leetcode/issues/277)

```js
function findCelebrity(n, knows) {
  let ans = 0;
  for (let i = 0; i < n.length; i++) {
    if (knows(ans, i)) i = ans;
  }
  for (let i = 0; i < ans; i++) {
    if (knows(ans, i) || !knows(i, ans)) return -1;
  }
  for (let i = ans + 1; i < n.length; i++) {
    if (!knows(i, ans)) return -1;
  }
  return ans;
}
```

## Design

### [Design LRU Cache](https://leetcode.com/problems/lru-cache/)

1. 利用 JS Map 的特性，map.keys() 的遍历顺序跟 key 的添加顺序相同。

```js
class LRUCache {
  constructor(capacity) {
    this.cache = new Map();
    this.capacity = capacity;
  }

  get(key) {
    if (!this.cache.has(key)) return -1;
    let v = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, v);
    return v;
  }

  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    this.cache.set(key, value);
    if (this.cache.size > this.capacity) {
      this.cache.delete(this.cache.keys().next().value);
    }
  }
}
```

2. map + doubly linked list，map 记录 key 到 node 的映射，node 可以 O(1) 删除 / 增加。

```js
class DoublyLinkedListNode {
  constructor(key, value, pre = null, next = null) {
    this.key = key;
    this.value = value;
    this.pre = pre;
    this.next = next;
  }
}

class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map();
    this.head = new DoublyLinkedListNode();
    this.tail = new DoublyLinkedListNode();
    this.head.next = this.tail;
    this.tail.pre = this.head;
  }

  get(key) {
    if (this.map.has(key)) {
      let node = this.map.get(key);
      this.remove(node);
      this.add(node);
      return node.value;
    }
    return -1;
  }

  put(key, value) {
    if (this.map.has(key)) {
      this.remove(this.map.get(key));
    }
    let node = new DoublyLinkedListNode(key, value);
    this.add(node);
    this.map.set(key, node);
    if (this.map.size > this.capacity) {
      let node = this.head.next;
      this.remove(node);
      this.map.delete(node.key);
    }
  }

  remove(node) {
    let pre = node.pre,
      next = node.next;
    pre.next = next;
    next.pre = pre;
  }

  add(node) {
    let pre = this.tail.pre;
    pre.next = node;
    this.tail.pre = node;
    node.pre = pre;
    node.next = this.tail;
  }
}
```

### [Design Tic-Tac-Toe](https://github.com/grandyang/leetcode/issues/348)

1. n x n 矩阵模拟
2. 节省空间：rows[i] 代表玩家在 i 行累计的分数，cols[i] 代表玩家在 j 列累计的分数，diag & revDiag 分别代表两条对角线上的分数。

```js
class TicTacToe {
  constructor(n) {
    this.rows = Array(n).fill(0);
    this.cols = Array(n).fill(0);
    this.diag = 0;
    this.revDiag = 0;
    this.n = n;
  }

  move(row, col, player) {
    let count = player === 1 ? 1 : -1;
    this.rows[row] += count;
    this.cols[col] += count;
    if (row === col) this.diag += count;
    if (row === n - col - 1) this.revDiag += count;
    if (
      Math.abs(this.rows[row]) === this.n ||
      Math.abs(this.cols[row]) === this.n ||
      Math.abs(this.diag) === this.n ||
      Math.abs(this.revDiag) === this.n
    ) {
      return player;
    }
    return 0;
  }
}
```

### [Min Stack](https://leetcode.com/problems/min-stack/)

```js
class MinStack {
  constructor() {
    this.stack = [];
    this.min = [Infinity];
  }
  push(val) {
    this.stack.push(val);
    this.min.push(
      val < this.min[this.min.length - 1] ? val : this.min[this.min.length - 1]
    );
  }
  pop() {
    this.stack.pop();
    this.min.pop();
  }
  top() {
    return this.stack[this.stack.length - 1];
  }
  getMin() {
    return this.min[this.min.length - 1];
  }
}
```

### [Design HashMap](https://leetcode.com/problems/design-hashmap/)

```js
const factor = 10000;
function hash(value) {
  return value % factor;
}

class MyHashMap {
  constructor() {
    this.hash = Array(factor);
  }

  put(key, val) {
    let group = hash(key);
    if (!this.hash[group]) this.hash[group] = [];
    let idx = this.hash[group].findIndex((el) => el.key === key);
    if (idx === -1) {
      this.hash[group].push({ key, val });
    } else {
      this.hash[group][idx].val = val;
    }
  }

  get(key) {
    let group = hash(key);
    if (!this.hash[group]) return -1;

    let el = this.hash[group].find((el) => el.key === key);
    return el ? el.val : -1;
  }

  remove(key) {
    let group = hash(key);
    if (!this.hash[group]) return;
    let idx = this.hash[group].findIndex((el) => el.key === key);
    if (idx !== -1) {
      this.hash[group].splice(idx, 1);
    }
  }
}
```

### [Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/)

```js
class MyStack {
  constructor() {
    this.queue = [];
  }
  push(x) {
    this.queue.push(x);
  }
  pop() {
    let size = this.queue.length;
    for (let i = 1; i < size; i++) {
      this.queue.push(this.queue.shift());
    }
    return this.queue.shift();
  }
  empty() {
    return this.queue.length < 1;
  }
  top() {
    return this.queue[this.queue.length - 1];
  }
}
```

### [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)

```js
class MyQueue {
  constructor() {
    this.stack = [];
  }
  push(x) {
    let tempStack = [];
    while (this.stack.length > 0) {
      tempStack.push(this.stack.pop());
    }
    tempStack.push(x);
    while (tempStack.length > 0) {
      this.stack.push(tempStack.pop());
    }
  }
  pop() {
    return this.stack.pop();
  }
  peek() {
    return this.stack[this.stack.length - 1];
  }
  empty() {
    return this.stack.length === 0;
  }
}
```

### [Design Hit Counter](https://www.geeksforgeeks.org/design-a-hit-counter/)

1. using queue, remove elements not in the period.

```js
class HitCounter {
  constructor() {
    this.queue = [];
  }
  hit(timestamp) {
    this.queue.push(timestamp);
  }
  getHits(timestamp) {
    while (this.queue.length && timestamp - this.queue[0] >= 300) {
      this.queue.shift();
    }
    return this.queue.length;
  }
}
```

2. Follow up: what if the data comes in unordered and several hits carry the same timestamp?

use two arrays, one to record last timestamp of each time unit, and the hits for that unit.

```js
class HitCounter {
  constructor() {
    this.times = [];
    this.hits = [];
  }
  hit(timestamp) {
    let idx = timestamp % 300;
    if (this.times[idx] !== timestamp) {
      this.times[idx] = timestamp;
      this.hits[idx] = 1;
    } else {
      this.hits[idx]++;
    }
  }
  getHits(timestamp) {
    let res = 0;
    for (let i = 0; i < this.times.length; i++) {
      if (timestamp - this.times[i] < 300) {
        res += this.hits[i];
      }
    }
    return res;
  }
}
```

3. Follow up: How to handle concurrent requests? => lock; too many traffic? => distribute writes and reads by sum up all results on different machines.

### [Implement Magic Dictionary](https://leetcode.com/problems/implement-magic-dictionary/)

```js
class MagicDictionary {
  constructor() {
    this.dic = new Set();
  }
  buildDict(words) {
    words.forEach((word) => this.dic.add(word));
  }
  search(word) {
    for (let i = 0; i < word.length; i++) {
      for (let code = 97; code < 123; code++) {
        let c = String.fromCharCode(code);
        if (
          word[i] != c &&
          this.dic.has(word.slice(0, i) + c + word.slice(i + 1))
        ) {
          return true;
        }
      }
    }
    return false;
  }
}
```

### [My Calendar I](https://leetcode.com/problems/my-calendar-i/)

O(N): linear search

```js
class MyCalendar {
  constructor() {
    this.intervals = [];
  }

  book(start, end) {
    let i = 0;
    while (
      this.intervals.length &&
      i < this.intervals.length &&
      start >= this.intervals[i][1]
    )
      i++;
    if (
      this.intervals.length &&
      i < this.intervals.length &&
      end > this.intervals[i][0]
    )
      return false;
    this.intervals.splice(i, 0, [start, end]);
    return true;
  }
}
```

O(N): binary search

```js
class MyCalendar {
  constructor() {
    this.intervals = [];
  }

  book(start, end) {
    let l = 0,
      r = this.intervals.length;
    while (l < r) {
      let mid = Math.floor((l + r) / 2),
        [s, e] = this.intervals[mid];
      if (!(start >= e || end <= s)) {
        return false;
      } else if (e <= start) {
        l = mid + 1;
      } else {
        r = mid;
      }
    }
    this.intervals.splice(l, 0, [start, end]);
    return true;
  }
}
```

O(logN): binary-tree

```js
class Event {
  constructor(start, end, left = null, right = null) {
    this.start = start;
    this.end = end;
    this.left = left;
    this.right = right;
  }
}

function insert(root, node) {
  if (node.start >= root.end) {
    if (!root.right) {
      root.right = node;
      return true;
    }
    return insert(root.right, node);
  } else if (node.end <= root.start) {
    if (!root.left) {
      root.left = node;
      return true;
    }
    return insert(root.left, node);
  } else {
    return false;
  }
}

class MyCalendar {
  constructor() {
    this.root = null;
  }

  book(start, end) {
    let event = new Event(start, end);
    if (this.root === null) {
      this.root = event;
      return true;
    }
    return insert(this.root, event);
  }
}
```

### [Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/)

```js
class BSTIterator {
  constructor(root) {
    this.stack = [];
    while (root !== null) {
      this.stack.push(root);
      root = root.left;
    }
  }

  next() {
    let node = this.stack.pop(),
      next = node.right;
    while (next) {
      this.stack.push(next);
      next = next.left;
    }
    return node.val;
  }

  hasNext() {
    return this.stack.length > 0;
  }
}
```

### [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/)

```js
class RandomizedSet {
  constructor() {
    this.map = new Map();
    this.nums = [];
  }
  insert(x) {
    if (this.map.has(x)) return false;
    this.map.set(x, this.nums.length);
    this.nums.push(x);
    return true;
  }
  remove(x) {
    if (!this.map.has(x)) return false;
    if (this.nums.length > 1) {
      let pos = this.map.get(x),
        end = this.nums[this.nums.length - 1];
      this.nums[pos] = end;
      this.map.set(end, pos);
    }
    this.map.delete(x);
    this.nums.pop();
    return true;
  }
  getRandom() {
    return this.nums[Math.floor(Math.random() * this.nums.length)];
  }
}
```

### [Insert Delete GetRandom O(1) - Duplicates allowed](https://leetcode.com/problems/insert-delete-getrandom-o1-duplicates-allowed/)

```js
class RandomizedCollection {
  constructor() {
    this.map = new Map();
    this.nums = [];
  }
  insert(x) {
    let contains = this.map.has(x);
    if (!contains) this.map.set(x, new Set());
    this.map.get(x).add(this.nums.length);
    this.nums.push(x);
    return !contains;
  }
  remove(x) {
    if (!this.map.has(x)) return false;
    let pos = [...this.map.get(x)][0];
    this.map.get(x).delete(pos);
    if (pos < this.nums.length - 1) {
      let end = this.nums[this.nums.length - 1];
      this.nums[pos] = end;
      this.map.get(end).delete(this.nums.length - 1);
      this.map.get(end).add(pos);
    }
    this.nums.pop();
    if (this.map.get(x).size === 0) this.map.delete(x);
    return true;
  }
  getRandom() {
    return this.nums[Math.floor(Math.random() * this.nums.length)];
  }
}
```

## Others

### [Day of the Week](https://leetcode.com/problems/day-of-the-week/)

In general, you can start the list of days with a known date. For example, if today when writing the solution is Day = 28, Month = 3, and Year = 2021 and you know the day name is "Sunday" then +1 day must be "Monday" and likewise -1 day must be "Saturday".

```js
const months = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
const days = [
  "Sunday",
  "Monday",
  "Tuesday",
  "Wednesday",
  "Thursday",
  "Friday",
  "Saturday",
];

function dayOfTheWeek(day, month, year) {
  let today = daysFromStart(28, 3, 2021),
    target = daysFromStart(day, month, year);
  return days[(((target - today) % 7) + 7) % 7];
}

function daysFromStart(day, month, year) {
  let sum = 0;
  for (let i = 1971; i < year; i++) {
    sum += 365;
    if (isLeapYear(i)) {
      sum++;
    }
  }
  for (let i = 1; i < month; i++) {
    sum += months[i - 1];
    if (i == 2 && isLeapYear(year)) {
      sum++;
    }
  }
  sum += day - 1;
  return sum;
}

function isLeapYear(year) {
  return (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
}
```

### [Battleships in a Board](https://leetcode.com/problems/battleships-in-a-board/)

```js
var countBattleships = function (board) {
  let m = board.length,
    n = board[0].length,
    res = 0;
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (board[i][j] === ".") continue;
      if (i > 0 && board[i - 1][j] === "X") continue;
      if (j > 0 && board[i][j - 1] === "X") continue;
      res++;
    }
  }
  return res;
};
```

