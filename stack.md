# Stack

## 计算器 逆波兰表达式

### [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/)

```js
var calculate = function (s) {
  if (!s || s.length === 0) return 0;
  s = s.replace(/\s/g, "");
  let stack = [],
    num = 0,
    sign = "+";
  for (let i = 0; i < s.length; i++) {
    if (/\d/.test(s[i])) {
      num = num * 10 + parseInt(s[i]);
    }
    if (/\D/.test(s[i]) || i == s.length - 1) {
      if (sign === "-") stack.push(-num);
      else if (sign === "+") stack.push(num);
      else if (sign === "*") stack.push(stack.pop() * num);
      else if (sign === "/") stack.push(Math.trunc(stack.pop() / num));
      sign = s[i];
      num = 0;
    }
  }
  return stack.reduce((a, b) => a + b, 0);
};
```

### [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

```js
var evalRPN = function(tokens) {
    let stack = []
    
    for (let i = 0; i < tokens.length; i++) {
        let ch = tokens[i]
        if (ch == '+') {
            stack.push(stack.pop() + stack.pop())
        } else if (ch == '-') {
            stack.push(-stack.pop() + stack.pop())
        } else if (ch == '*') {
            stack.push(stack.pop() * stack.pop())
        } else if (ch == '/') {
            stack.push(Math.trunc(1 / stack.pop() * stack.pop()))
        } else {
            stack.push(Number.parseInt(ch, 10))
        }
    }

    return stack.pop()
};
```

## monotone stack 单调栈

either increasing or decreasing

### [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/)

1. 暴力：map 记录 nums2 数组的 el => idx 映射，对于 nums1 中的每个数，从其在 nums2 的 idx 后开始遍历，找到一个比它大的值。

```js
var nextGreaterElement = function (nums1, nums2) {
  let map = new Map();
  for (let i = 0; i < nums2.length; i++) {
    map.set(nums2[i], i);
  }
  let res = [];
  for (let n of nums1) {
    let i = map.get(n) + 1;
    while (i < nums2.length && nums2[i] <= n) i++;
    res.push(i === nums2.length ? -1 : nums2[i]);
  }
  return res;
};
```

2. 单调栈

```js
var nextGreaterElement = function (nums1, nums2) {
  let map = new Map(),
    stack = [];
  for (let n of nums2) {
    while (stack.length && stack[stack.length - 1] < n) {
      map.set(stack.pop(), n);
    }
    stack.push(n);
  }
  return nums1.map((n) => map.get(n) || -1);
};
```

### [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/)

对于 n <= i < 2n 的数字，只跟栈内元素比较，不入栈。

```js
var nextGreaterElements = function (nums) {
  let stack = [],
    n = nums.length,
    res = Array(n).fill(-1);
  for (let i = 0; i < 2 * n; i++) {
    let num = nums[i % n];
    while (stack.length && nums[stack[stack.length - 1]] < num) {
      res[stack.pop()] = num;
    }
    if (i < n) stack.push(i);
  }
  return res;
};
```

### [Next Greater Element III](https://leetcode.com/problems/next-greater-element-iii/)

根据前两题的思路，1) 先从右向左找到第一个 decrease 的数位 i，2) 然后再从右向左找到第一个比它大的数字 j，交换这两个数字的位置，3) 再 reverse decrease 数位右侧的所有数字。

由于 1) 保证了 decrease 数位右侧，从右向左是递增的，而 2) 又保证了交换后 i 左侧的数字仍然都比 i 大，从而保证了 reverse 后得到的序列最小。

```js
var nextGreaterElement = function (n) {
  let digits = n.toString().split("");

  let i = getFirstDesc(digits);
  if (i === -1) return -1;

  let j = getFirstLarger(digits, i);

  swap(digits, i, j);
  reverse(digits, i + 1, digits.length - 1);

  let res = parseInt(digits.join(""));
  return res > 2147483647 ? -1 : res;
};

function getFirstDesc(digits) {
  for (let i = digits.length - 2; i >= 0; i--) {
    if (digits[i] < digits[i + 1]) return i;
  }
  return -1;
}

function getFirstLarger(digits, i) {
  for (let j = digits.length - 1; j > i; j--) {
    if (digits[j] > digits[i]) return j;
  }
  return -1;
}

function swap(arr, i, j) {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}

function reverse(arr, i, j) {
  while (i < j) {
    swap(arr, i, j);
    i++;
    j--;
  }
}
```

### [Remove K Digits](https://leetcode.com/problems/remove-k-digits/)

1. 暴力：找到所有 count(remaining digits) = num.length - k，且代表的值最小的子序列。

```js
var removeKdigits = function (num, k) {
  let min = Infinity,
    remain = num.length - k;
  const dfs = (idx, curr) => {
    if (curr.length == remain) {
      min = Math.min(min, Number(curr));
      return;
    }
    if (idx >= num.length) return;
    dfs(idx + 1, curr + num[idx]);
    dfs(idx + 1, curr);
  };
  dfs(0, "");
  return min.toString();
};
```

2. 找到华点：每次 remove first peak digit，因为 remove 其他 digit 都不可能使得当前结果比其更小。O(nk)。可以通过单调栈简化找 peak digit 的过程。

```js
var removeKdigits = function (num, k) {
  let stack = [];
  for (let digit of num) {
    while (stack.length && stack[stack.length - 1] > digit && k) {
      stack.pop();
      k--;
    }
    stack.push(digit);
  }
  while (k--) stack.pop();
  let res = "",
    i = 0;
  while (i < stack.length && stack[i] === "0") i++;
  while (i < stack.length) res += stack[i++];
  return res === "" ? "0" : res;
};
```


### [Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/)

how many times will each sub-array minimum occur in the sum?

=> find distance to current number's prev less & next less element.

=> what if duplicate exists?

```js
var sumSubarrayMins = function(arr) {
  let n = arr.length, res = 0, mod = 1e9 + 7;
  for (let i = 0; i < n; i++) {
    let j = i - 1, k = i + 1;
    while (j >= 0 && arr[j] >= arr[i]) j--;
    while (k < n && arr[k] > arr[i]) k++;
    let prevLess = j >= 0 ? i - j : i + 1, nextLess = k < n ? k - i : n - i;
    res = (res + arr[i] * prevLess * nextLess) % mod;
  }
  return res;
};
```

optimize: use monotonic stack to track each number's prev less & next less.

```js
var sumSubarrayMins = function(arr) {
  let n = arr.length, left = [], right = [], prevLess = [], nextLess = [];
  for (let i = 0; i < n; i++) {
    while (left.length && left[left.length - 1][0] >= arr[i]) left.pop();
    prevLess[i] = left.length === 0 ? i + 1 : i - left[left.length-1][1];
    left.push([arr[i], i]);
  }
  for (let i = n - 1; i >= 0; i--) {
    while (right.length && right[right.length - 1][0] > arr[i]) right.pop();
    nextLess[i] = right.length === 0 ? n - i : right[right.length - 1][1] - i;
    right.push([arr[i], i]);
  }

  let res = 0, mod = 10 ** 9 + 7;
  for (let i = 0; i < n; i++) {
    res = (res + arr[i] * prevLess[i] * nextLess[i]) % mod;
  }
  return res;
};
```

[ref](https://leetcode.com/problems/sum-of-subarray-minimums/discuss/178876/stack-solution-with-very-detailed-explanation-step-by-step)

## others

### [Next Greater Node In Linked List](https://leetcode.com/problems/next-greater-node-in-linked-list/)

```js
var nextLargerNodes = function (head) {
  let nums = [],
    stack = [];
  while (head) {
    nums.push(head.val);
    head = head.next;
  }
  for (let i = 0; i < nums.length; i++) {
    while (stack.length && nums[stack[stack.length - 1]] < nums[i]) {
      nums[stack.pop()] = nums[i];
    }
    stack.push(i);
  }
  while (stack.length) nums[stack.pop()] = 0;
  return nums;
};
```

### [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/)

```js
var asteroidCollision = function (asteroids) {
  let res = [];
  for (let a of asteroids) {
    if (a < 0) {
      while (res.length && res[res.length - 1] > 0 && res[res.length - 1] < -a)
        res.pop();
      if (res.length && res[res.length - 1] === -a) {
        res.pop();
      } else if (res.length === 0 || res[res.length - 1] < 0) {
        res.push(a);
      }
    } else {
      res.push(a);
    }
  }
  return res;
};
```

### [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

```js
const mapLeft = {
  "{": 1,
  "[": 2,
  "(": 3,
};

const mapRight = {
  "}": 1,
  "]": 2,
  ")": 3,
};

const isValid = function (s) {
  let stack = [];

  for (let i = 0; i < s.length; i++) {
    if (mapLeft[s[i]]) {
      stack.push(s[i]);
    } else if (!stack.length || mapRight[s[i]] !== mapLeft[stack.pop()]) {
      return false;
    }
  }

  return !stack.length;
};
```

### [Exclusive Time of Functions](https://leetcode.com/problems/exclusive-time-of-functions/)

```js
var exclusiveTime = function (n, logs) {
  let res = Array(n).fill(0), stack = [], prevStart = 0;
  for (let log of logs) {
    let [id, flag, stamp] = log.split(":");
    id = Number(id);
    stamp = Number(stamp);
    if (flag === 'start') {
      if (stack.length) res[stack[stack.length - 1]] += stamp - prevStart;
      stack.push(id);
      prevStart = stamp;
    } else {
      res[stack.pop()] += stamp - prevStart + 1;
      prevStart = stamp + 1;
    }
  }
  return res;
};
```

### [Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/)

```js
var removeDuplicateLetters = function (s) {
  if (s.length <= 1) return s;
  let count = new Map(),
    used = new Set();
  for (let ch of s) count.set(ch, (count.get(ch) || 0) + 1);
  let res = [];
  for (let ch of s) {
    let code = ch.charCodeAt();
    count.set(ch, count.get(ch) - 1);
    if (!used.has(ch)) {
      while (
        res.length &&
        res[res.length - 1].charCodeAt() > code &&
        count.get(res[res.length - 1]) > 0
      ) {
        used.delete(res[res.length - 1]);
        res.pop();
      }
      res.push(ch);
    }
    used.add(ch);
  }
  return res.join("");
};
```
