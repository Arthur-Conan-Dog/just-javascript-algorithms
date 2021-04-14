# String

## Common used methods

- get substring with start index & end index: slice(startIndex, endIndex) => s[startIndex, endIndex)

- get substring with start index & length: substr(startIndex, length)

- a.localeCompare(b) 为按字母表升序

- charCodeAt(); String.fromCharCode();

## Common operations

### 判断是否是回文串

```js
function isPalindrome(word) {
  let i = 0,
    j = word.length - 1;
  while (i < j) {
    if (word[i++] !== word[j--]) return false;
  }
  return true;
}
```

### 翻转字符串

```js
function reverse(word) {
  let res = "";
  for (let i = word.length - 1; i >= 0; i--) res += word[i];
  return res;
}
```

### 判断 t 是否是 s 的 subsequence

```js
function isSubsequence(s, t) {
  let i = 0, j = 0;
  while (i < s.length && j < t.length) {
    if (s[i] === t[j]) j++;
    i++;
  }
  return j === t.length;
}
```

### 求所有指定长度的子序列 (combination of length n)

```js
function allSubsequencesOfK(s, k) {
  let res = [];
  const dfs = (idx, curr) => {
    if (curr.length === k) {
      res.push(curr);
      return;
    }
    if (idx >= s.length) return;
    dfs(idx + 1, curr + s[idx]);
    dfs(idx + 1, curr);
  };
  dfs(0, "");
  return res;
}
```

## 正则

+ 表示 >= 1，* 表示 >= 0，? 表示 0 个或 1 个，[] 表示字符集，() 表示分组。

RegExp.prototype.exec() 返回参数：

- res[0] => 匹配到的字符串

- res[1...n] => 分组 1 - n

## Questions

### [Implement strStr()](https://leetcode.com/problems/implement-strstr/)

判断 t 是否是 s 的子串

```js
var strStr = function(s, t) {
  for (let i = 0;; i++) {
    for (let j = 0;; j++) {
      if (j === t.length) return i;
      if (i + j === s.length) return -1;
      if (t[j] !== s[i+j]) break;
    }
  }
}
```

### [Compare Version Numbers](https://leetcode.com/problems/compare-version-numbers/)

O(Max(M, N)) O(M+N)

```js
function compareVersion(a, b) {
  let ca = a.split('.').map(Number), cb = b.split('.').map(Number);
  let n = Math.max(ca.length, cb.length);
  for (let i = 0; i < n; i++) {
    let na = i < ca.length ? ca[i] : 0;
    let nb = i < cb.length ? cb[i] : 0;
    if (na > nb) return 1;
    else if (na < nb) return -1;
  }
  return 0;
}
```

O(Max(M, N)) O(1)

```js
function compareVersion(a, b) {
  let i = 0, j = 0, vi = 0, vj = 0;
  while (i < a.length || j < b.length) {
    let pair1 = getNextChunk(a, i);
    let pair2 = getNextChunk(b, j);
    vi = pair1[0], i = pair1[1];
    vj = pair2[0], j = pair2[1];
    if (vi > vj) return 1;
    else if (vi < vj) return -1;
  }
  return 0;
}

function getNextChunk(s, currentPos) {
  if (currentPos >= s.length) return [0, s.length];
  let i = currentPos, count = 0;
  while (i < s.length && s[i] !== '.') {
    i++;
    count++;
  }
  let sub = s.substr(currentPos, count);
  return [Number(sub), i + 1];
}
```

### [Palindrome Pairs](https://leetcode.com/problems/palindrome-pairs/)

a list of **unique** words

考验设计 & 分析不同情况的能力。

cases:
case 1: s1 = '', s2 is palindrome => s1 + s2 & s2 + s1
case 2: s1, s2 = reverse(s1) => s1 + s2 & s2 + s1
case 3: s1[0:cut] is palindrome, s2 = reverse(s1[cut+1:]) => s2 + s1
case 4: s1[cut+1:] is palindrome, s2 = reverse(s1[0:cut]) => s1 + s2

```js
var palindromePairs = function (words) {
  let res = [];
  if (words === null || words.length === 0) return res;

  let map = new Map();
  for (let i = 0; i < words.length; i++) {
    map.set(words[i], i);
  }

  // case 1
  if (map.has("")) {
    let idx = map.get("");
    for (let i = 0; i < words.length; i++) {
      if (isPalindrome(words[i]) && i !== idx) {
        res.push([idx, i]);
        res.push([i, idx]);
      }
    }
  }

  // case 2
  for (let i = 0; i < words.length; i++) {
    let temp = reverse(words[i]);
    if (map.has(temp) && map.get(temp) !== i) {
      res.push([i, map.get(temp)]);
    }
  }

  for (let i = 0; i < words.length; i++) {
    let curr = words[i];
    for (let cut = 1; cut < curr.length; cut++) {
      let left = curr.slice(0, cut),
        right = curr.slice(cut);
      // case 3
      if (isPalindrome(left)) {
        let temp = reverse(right);
        if (map.has(temp) && map.get(temp) !== i) {
          res.push([map.get(temp), i]);
        }
      }
      // case 4
      if (isPalindrome(right)) {
        let temp = reverse(left);
        if (map.has(temp) && map.get(temp) !== i) {
          res.push([i, map.get(temp)]);
        }
      }
    }
  }

  return res;
};
```

### [Remove Comments](https://leetcode.com/problems/remove-comments/)

考验设计 & 分析不同情况的能力。

corner cases:

- `['aa//bb','cc'] => ['aa', 'cc']`
- `['aa/*bb*///cc'] => ['aa']`
- `['aa/*bb', 'cc*/dd'] => ['aadd']`

```js
var removeComments = function (source) {
  let res = [],
    inBlock = false,
    newLine = "";
  for (let line of source) {
    for (let i = 0; i < line.length; i++) {
      if (inBlock) {
        if (line[i] == "*" && i + 1 < line.length && line[i + 1] == "/") {
          inBlock = false;
          i++;
        }
      } else {
        if (line[i] == "/" && i + 1 < line.length && line[i + 1] == "/") {
          break;
        } else if (
          line[i] == "/" &&
          i + 1 < line.length &&
          line[i + 1] == "*"
        ) {
          inBlock = true;
          i++;
        } else {
          newLine += line[i];
        }
      }
    }
    if (!inBlock && newLine.length > 0) {
      res.push(newLine);
      newLine = "";
    }
  }
  return res;
};
```

### [Simplify Path](https://leetcode.com/problems/simplify-path/)

```js
var simplifyPath = function (path) {
  let stack = [], paths = path.split('/').filter(p => p.length > 0);
  for (let dic of paths) {
    if (dic !== '.' && dic !== '..') stack.push(dic);
    else if (dic === '..') stack.pop();
  }
  return '/' + stack.join('/');
};
```

### [Find Common Characters](https://leetcode.com/problems/find-common-characters/)

数字、英文字母间转换

'a' => 97; 'A' => 65;

两字母间不能直接做减法。

```js
var commonChars = function (A) {
  let res = [],
    map = Array(26).fill(10001);
  for (let a of A) {
    let curr = Array(26).fill(0);
    for (let ch of a) curr[ch.charCodeAt() - 97]++;
    for (let i = 0; i < 26; i++) map[i] = Math.min(map[i], curr[i]);
  }
  for (let i = 0; i < 26; i++) {
    for (let j = 0; j < map[i]; j++) {
      res.push(String.fromCharCode(i + 97));
    }
  }
  return res;
};
```

### [Solve the Equation](https://leetcode.com/problems/solve-the-equation/)

表达式解析

[一个有用的正则：/(?=[+-])/](https://stackoverflow.com/questions/4416425/how-to-split-string-with-some-separator-but-without-removing-that-separator-in-j)

```js
var solveEquation = function (equation) {
  let [left, right] = equation.split("=");
  let [f1, i1] = evaluate(left),
    [f2, i2] = evaluate(right);
  f1 -= f2;
  i1 = i2 - i1;
  if (f1 === 0 && i1 === 0) return "Infinite solutions";
  if (f1 === 0) return "No solution";
  return "x=" + i1 / f1;
};

function evaluate(s) {
  let tokens = s.split(/(?=[+-])/),
    res = [0, 0];
  for (let token of tokens) {
    if (token === "+x" || token === "x") res[0]++;
    else if (token === "-x") res[0]--;
    else if (token.includes("x")) {
      res[0] += parseInt(token.slice(0, token.indexOf("x")));
    } else {
      res[1] += parseInt(token);
    }
  }
  return res;
}
```
