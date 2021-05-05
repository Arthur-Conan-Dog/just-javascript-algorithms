# dfs

## general

### [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)

start from border => inside.

```js
var pacificAtlantic = function (matrix) {
  let res = [];
  if (!matrix || !matrix.length || !matrix[0].length) return res;

  let dir = [
    [0, 1],
    [0, -1],
    [1, 0],
    [-1, 0],
  ];
  let m = matrix.length,
    n = matrix[0].length;
  let p = Array(m)
      .fill(0)
      .map(() => Array(n).fill(false)),
    a = Array(m)
      .fill(0)
      .map(() => Array(n).fill(false));

  function dfs(visited, height, x, y) {
    if (
      x < 0 ||
      x >= m ||
      y < 0 ||
      y >= n ||
      visited[x][y] ||
      matrix[x][y] < height
    ) {
      return;
    }
    visited[x][y] = true;
    for (let [dx, dy] of dir) {
      dfs(visited, matrix[x][y], x + dx, y + dy);
    }
  }

  for (let i = 0; i < m; i++) {
    dfs(p, -Infinity, i, 0);
    dfs(a, -Infinity, i, n - 1);
  }
  for (let i = 0; i < n; i++) {
    dfs(p, -Infinity, 0, i);
    dfs(a, -Infinity, m - 1, i);
  }

  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (p[i][j] && a[i][j]) res.push([i, j]);
    }
  }

  return res;
};
```

### [Number of Distinct Island](https://github.com/grandyang/leetcode/issues/694)

An island is considered to be the same as another if and only if one island can be translated (and not rotated or reflected) to equal the other.

```js
function numberOfDistinctIslands(board) {
  let m = board.length,
    n = board[0].length,
    res = new Set();
  const directions = [
    [0, 1],
    [0, -1],
    [1, 0],
    [-1, 0],
  ];

  const dfs = (i, j, ix, iy, set) => {
    for (let [dx, dy] of directions) {
      let x = i + dx,
        y = j + dy;
      if (x < 0 || y < 0 || x >= m || y >= n || board[x][y] === 0) continue;
      board[x][y] = 0;
      set.add(`${x - ix}_${y - iy}`);
      dfs(x, y, ix, iy, set);
    }
  };
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (board[i][j] === 1) {
        let set = new Set();
        dfs(i, j, i, j, set);
        let rec = [...set].join('_');
        if (!res.has(rec)) {
          res.add(rec);
        }
      }
    }
  }
  return res.size;
}
```

## backtracking

The backtracking algorithms consists of the following steps:

1. Choose: Choose the potential candidate. => check all possibilities.

2. Constraint: Define a constraint that must be satisfied by the chosen candidate. => cut when impossible, continue if still possible.

3. Goal: We must define the goal that determines if have found the required solution and we must backtrack.

### [Subsets](https://leetcode.com/problems/subsets/)

```js
var subsets = function (nums) {
  let res = [];
  const dfs = (curr, start) => {
    res.push(curr.concat());
    for (let i = start; i < nums.length; i++) {
      curr.push(nums[i]);
      dfs(curr, i + 1);
      curr.pop();
    }
  };
  dfs([], 0);
  return res;
};
```

### [Subsets II](https://leetcode.com/problems/subsets-ii/)

contains duplicates.

```js
var subsetsWithDup = function (nums) {
  nums.sort((a, b) => a - b);
  let res = [];
  const dfs = (curr, start) => {
    res.push(curr.concat());
    for (let i = start; i < nums.length; i++) {
      if (i > start && nums[i] === nums[i - 1]) continue; // skip duplicates
      curr.push(nums[i]);
      dfs(curr, i + 1);
      curr.pop();
    }
  };
  dfs([], 0);
  return res;
};
```

### [Combinations](https://leetcode.com/problems/combinations/)

all possible combinations of k numbers out of the range [1, n].

```js
var combine = function (n, k) {
  let res = [];
  const dfs = (curr = [], start = 1) => {
    if (curr.length > k) return;
    if (curr.length === k) {
      res.push(curr.concat());
      return;
    }
    for (let i = start; i <= n; i++) {
      curr.push(i);
      dfs(i + 1, curr);
      curr.pop();
    }
  };
  dfs();
  return res;
};
```

### [Permutations](https://leetcode.com/problems/permutations/)

all numbers need to be used.

```js
var permute = function (nums) {
  let res = [];
  const dfs = (curr = new Set()) => {
    if (curr.size === nums.length) {
      res.push(curr.concat());
      return;
    }
    for (let num of nums) {
      if (curr.has(num)) continue; // skip chosen ones
      curr.add(num);
      dfs(curr);
      curr.delete(num);
    }
  };
  dfs();
  return res;
};
```

### [Permutations II](https://leetcode.com/problems/permutations-ii/)

might contain duplicates.

With inputs as [a1, a2, b1], we must make sure a1 goes before a2 to avoid duplicates: nums[i] === nums[i - 1] && !using[i - 1]) => skip.

```js
var permuteUnique = function (nums) {
  nums.sort((a, b) => a - b);
  let res = [],
    using = Array(nums.length).fill(false);
  const dfs = (curr = []) => {
    if (curr.length === nums.length) {
      res.push(curr.concat());
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (using[i] || (i > 0 && nums[i] === nums[i - 1] && !using[i - 1]))
        continue;
      using[i] = true;
      curr.push(nums[i]);
      dfs(curr);
      curr.pop();
      using[i] = false;
    }
  };
  dfs();
  return res;
};
```

### [Combination Sum III](https://leetcode.com/problems/combination-sum-iii/)

only numbers 1 through 9 are used;

each used at once;

all unique combinations.

```js
var combinationSum3 = function (k, n) {
  let res = [];
  const dfs = (remain = n, start = 1, curr = []) => {
    if (remain === 0 && curr.length === k) {
      res.push(curr.concat());
      return;
    }
    for (let i = start; i <= 9; i++) {
      curr.push(i);
      dfs(remain - i, i + 1, curr);
      curr.pop();
    }
  };
  dfs();
  return res;
};
```

### [Combination Sum](https://leetcode.com/problems/combination-sum/)

given distinct integers;

can reuse same element;

all unique combinations.

```js
var combinationSum = function (candidates, target) {
  let res = [];
  const dfs = (remain = target, start = 0, curr = []) => {
    if (remain < 0) return;
    if (remain === 0) {
      res.push(curr.concat());
      return;
    }
    for (let i = start; i < candidates.length; i++) {
      curr.push(candidates[i]);
      dfs(remain - candidates[i], i, curr); // not i + 1 because we can reuse same elements
      curr.pop();
    }
  };
  dfs();
  return res;
};
```

### [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

integers might have duplicates;

cannot reuse same elements;

all unique combinations.

=> sort and skip duplicates when going down from the current path.

```js
var combinationSum2 = function (candidates, target) {
  let res = [];
  candidates.sort((a, b) => a - b);
  const dfs = (remain = target, start = 0, curr = []) => {
    if (remain < 0) return;
    if (remain === 0) {
      res.push(curr.concat());
      return;
    }
    for (let i = start; i < candidates.length; i++) {
      if (i > start && candidates[i] === candidates[i - 1]) continue; // skip duplicates
      curr.push(candidates[i]);
      dfs(remain - candidates[i], i + 1, curr);
      curr.pop();
    }
  };
  dfs();
  return res;
};
```

### [Factor Combinations](https://github.com/grandyang/leetcode/issues/254)

return all possible combinations of given number's factors.

11 => []

12 => [[2,2,3], [2,6], [3,4]]

```js
function getFactors(n) {
  let res = [];
  const dfs = (remain, start = 2, curr = []) => {
    if (remain === 1) {
      if (curr.length > 1)
        // skip self
        res.push(curr.concat());
      return;
    }
    for (let i = start; i <= n; i++) {
      if (remain % i !== 0) continue;
      curr.push(i);
      dfs(remain / i, i, curr);
      curr.pop();
    }
  };
  dfs(n);
  return res;
}
```

### [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

```js
var partition = function (s) {
  let res = [];
  const isPalindrome = (start, end) => {
    while (start < end) {
      if (s[start] !== s[end]) return false;
      start++;
      end--;
    }
    return true;
  };
  const dfs = (start = 0, curr = []) => {
    if (start === s.length) {
      res.push(curr.concat());
      return;
    } 
    for (let end = start; end < s.length; end++) {
      if (isPalindrome(start, end)) {
        curr.push(s.slice(start, end + 1));
        dfs(end + 1, curr);
        curr.pop();
      }
    }
  };
  dfs();
  return res;
};
```

O(N*2^N) time complexity: for each character in the string we have 2 choices to create new palindrome substrings, one is to join with previous substring and another is to start a new palindrome substring. Thus in the worst case there are 2^N palindrome substrings. Each substring will take O(N) time to check if it's palindrome and O(N) time to generate substring from start to end indexes.

### [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)

```js
var generateParenthesis = function (n) {
  let res = [];
  const dfs = (s, left, right) => {
    if (right > left || left > n || right > n) return;
    if (left === n && right === n) {
      res.push(s);
      return;
    }
    dfs(s + "(", left + 1, right);
    dfs(s + ")", left, right + 1);
  };
  dfs("(", 1, 0);
  return res;
};
```

### [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

```js
var letterCombinations = function (digits) {
  if (digits.length === 0) return [];
  let map = {
    2: ["a", "b", "c"],
    3: ["d", "e", "f"],
    4: ["g", "h", "i"],
    5: ["j", "k", "l"],
    6: ["m", "n", "o"],
    7: ["p", "q", "r", "s"],
    8: ["t", "u", "v"],
    9: ["w", "x", "y", "z"],
  };
  let res = [];
  const dfs = (s, idx) => {
    if (idx === digits.length) {
      res.push(s);
      return;
    }
    for (let ch of map[digits[idx]]) {
      dfs(s + ch, idx + 1);
    }
  };
  dfs("", 0);
  return res;
};
```

### [Letter Case Permutation](https://leetcode.com/problems/letter-case-permutation/)

```js
var letterCasePermutation = function (S) {
  let res = [];
  const dfs = (pos, curr) => {
    if (pos === S.length) {
      res.push(curr);
      return;
    }
    dfs(pos + 1, curr + S[pos]);
    if (/[a-z]/.test(S[pos])) dfs(pos + 1, curr + S[pos].toUpperCase());
    if (/[A-Z]/.test(S[pos])) dfs(pos + 1, curr + S[pos].toLowerCase());
  };
  dfs(0, "");
  return res;
};
```

### [Generalized Abbreviation](https://github.com/grandyang/leetcode/issues/320)

对于每个 pos，可以选择不压缩当前字符，即把当前字符添加至目前 track 的字符串 curr 中；也可以选择压缩当前字符，即增加已压缩的字符计数。

```js
function generateAbbreviations(word) {
  let res = [];
  const dfs = (pos, cnt, curr) => {
    if (pos === word.length) {
      res.push(cnt > 0 ? curr + cnt : curr);
      return;
    }
    // (continue) abbreviate current character
    dfs(pos + 1, cnt + 1, curr);
    // keep current character
    dfs(pos + 1, 0, curr + (cnt > 0 ? cnt : "") + word[pos]);
  };
  dfs(0, 0, "");
  return res;
}
```

### [Word Search](https://leetcode.com/problems/word-search/)

```js
var exist = function (board, word) {
  let m = board.length,
    n = board[0].length,
    dirs = [
      [0, 1],
      [0, -1],
      [1, 0],
      [-1, 0],
    ];
  const dfs = (i, j, idx) => {
    if (board[i][j] !== word[idx]) return false;
    if (idx === word.length - 1) return true;

    board[i][j] = "*"; // flip board[i][j] to '*' as visited, so that we could save some space.
    for (let [dx, dy] of dirs) {
      let x = i + dx,
        y = j + dy;
      if (x >= 0 && x < m && y >= 0 && y < n) {
        if (dfs(x, y, idx + 1)) return true;
      }
    }
    board[i][j] = word[idx];
    return false;
  };
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (dfs(i, j, 0)) return true;
    }
  }
  return false;
};
```

### [Knight Probability in Chessboard](https://leetcode.com/problems/knight-probability-in-chessboard/)

```js
function knightProbability(n, k, r, c) {
  let factor = 1 / 8,
    map = new Map();
  const dfs = (i, j, steps) => {
    if (i < 0 || i >= n || j < 0 || j >= n) return 0;
    if (steps === k) return 1;
    if (map.has(i + "," + j + "," + steps))
      return map.get(i + "," + j + "," + steps);
    let curr =
      factor *
      (dfs(i - 2, j - 1, steps + 1) +
        dfs(i - 2, j + 1, steps + 1) +
        dfs(i - 1, j + 2, steps + 1) +
        dfs(i + 1, j + 2, steps + 1) +
        dfs(i + 2, j - 1, steps + 1) +
        dfs(i + 2, j + 1, steps + 1) +
        dfs(i - 1, j - 2, steps + 1) +
        dfs(i + 1, j - 2, steps + 1));

    map.set(i + "," + j + "," + steps, curr);
    return curr;
  };
  return dfs(r, c, 0);
}
```

### [N-Queens](https://leetcode.com/problems/n-queens/)

```js
var solveNQueens = function (n) {
  let board = [...Array(n)].map(() => Array(n).fill(".")),
    res = [];
  const dfs = (row) => {
    if (row === n) {
      res.push(board.map((row) => row.join("")));
      return;
    }
    for (let col = 0; col < n; col++) {
      if (canPlace(board, row, col)) {
        board[row][col] = "Q";
        dfs(row + 1);
        board[row][col] = ".";
      }
    }
  };
  dfs(0);
  return res;
};

function canPlace(board, row, col) {
  for (let i = 0; i < row; i++) {
    if (board[i][col] == "Q") return false;
  }
  for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
    if (board[i][j] == "Q") return false;
  }
  for (let i = row - 1, j = col + 1; i >= 0 && j < board.length; i--, j++) {
    if (board[i][j] == "Q") return false;
  }
  return true;
}
```

optimize space: using 1D array, only track col pos.

```js
var solveNQueens = function (n) {
  let res = [];
  dfs(res, n);
  return res;
};

function dfs(res, n, board = [], row = 0) {
  if (row === n) {
    res.push(
      board.map((col) => ".".repeat(col) + "Q" + ".".repeat(n - col - 1))
    );
    return;
  }
  for (let col = 0; col < n; col++) {
    if (
      !board.some(
        (prevCol, prevRow) =>
          prevCol === col ||
          prevCol === row + col - prevRow ||
          prevCol === col - row + prevRow
      )
    ) {
      board.push(col);
      dfs(res, n, board, row + 1);
      board.pop();
    }
  }
}
```

### [Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/)

```js
var restoreIpAddresses = function (s) {
  let res = [];
  if (s.length > 12) return res;
  const dfs = (idx, curr, count) => {
    if (count === 4 && idx === s.length) {
      res.push(curr);
      return;
    }
    for (let i = 1; i <= 3; i++) {
      if (idx + i > s.length) return;
      if (i > 1 && s[idx] === "0") return; // skip 0x & 0xx
      let sec = s.slice(idx, idx + i);
      if (Number(sec) > 255) return;
      dfs(idx + i, count === 0 ? sec : curr + "." + sec, count + 1);
    }
  };
  dfs(0, "", 0);
  return res;
};
```

### [24 Game](https://leetcode.com/problems/24-game/)

use epsilon: the reason to use epsilon is because when we get the result with recurring decimal. eg. [3,3,8,8], 8 / (3 - 8/3) = 23.999...

check zero for division.

all pairs with all operation permutations.

```js
var judgePoint24 = function (nums) {
  const eps = 0.001;
  let permutations = (a, b) => {
    let res = [a - b, b - a, a + b, a * b];
    if (Math.abs(a) > eps) res.push(b / a);
    if (Math.abs(b) > eps) res.push(a / b);
    return res;
  };
  let dfs = (curr) => {
    let n = curr.length;
    if (n === 1) return Math.abs(curr[0] - 24) < eps;
    for (let i = 0; i < n; i++) {
      for (let j = i + 1; j < n; j++) {
        let next = [];
        for (let k = 0; k < n; k++) {
          if (k !== i && k !== j) next.push(curr[k]);
        }
        for (let val of permutations(curr[i], curr[j])) {
          next.push(val);
          if (dfs(next)) return true;
          next.pop();
        }
      }
    }
    return false;
  };
  return dfs(nums);
};
```

### [The Maze](https://github.com/grandyang/leetcode/issues/490)

How to keep the ball rolling until it meets a wall?

```js
function hasPath(map, start, destination) {
  let seen = new Set(), m = map.length, n = map[0].length;
  const directions = [[0,1], [0,-1], [1,0], [-1,0]];
  const dfs = (i, j) => {
    if (i === destination[0] && j === destination[1]) return true;
    for (let [dx, dy] of directions) {
      let x = i, y = j;
      while ((x + dx) >= 0 && (x + dx) < m && (y + dy) >= 0 && (y + dy) < n && map[(x + dx)][(y + dy)] !== 1) {
        x = x + dx;
        y = y + dy;
      }
      if (i === x && j === y) continue;
      if (!seen.has(x + '_' + y)) {
        seen.add(x + '_' + y);
        if (dfs(x, y)) return true;
      }
    }
    return false;
  }
  return dfs(start[0], start[1]);
}
```

# bfs

## general

### [Minesweeper](https://leetcode.com/problems/minesweeper/)

```js
var updateBoard = function (board, click) {
  let [x, y] = click;
  if (board[x][y] === "M") {
    board[x][y] = "X";
    return board;
  }
  return bfs(board, click);
};

function bfs(board, click) {
  let directions = [
      [-1, -1],
      [-1, 0],
      [-1, 1],
      [0, -1],
      [0, 1],
      [1, -1],
      [1, 0],
      [1, 1],
    ],
    m = board.length,
    n = board[0].length;

  let queue = [click];
  while (queue.length) {
    let [x, y] = queue.shift(),
      counts = 0;
    for (let [dx, dy] of directions) {
      let i = dx + x,
        j = dy + y;
      if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] === "M") {
        counts++;
      }
    }

    if (counts > 0) {
      board[x][y] = counts.toString();
    } else {
      board[x][y] = "B";
      for (let [dx, dy] of directions) {
        let i = dx + x,
          j = dy + y;
        if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] === "E") {
          queue.push([i, j]);
          board[i][j] = "B";
        }
      }
    }
  }
  return board;
}
```

### [Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders/)

problems:

1. how to find the minimum steps? => BFS, 1 step each time

2. on what condition it is not possible? => all nodes traversed without hitting n * n.

3. how to calc square => i, j mapping? => rowIdx = number / n, colIdx = number % n combine with (rowIdx is even or odd)

```js
var snakesAndLadders = function(board) {
  let n = board.length, target = n * n, queue = [1], map = {1:0};
  while (queue.length) {
    let num = queue.shift();
    if (num === target) return map[num];
    for (let i = num + 1; i <= Math.min(num + 6, target); i++) {
      let [x, y] = toIdx(i, n),
          next = board[x][y] === -1 ? i : board[x][y];
      if (map[next] === undefined) {
        map[next] = map[num] + 1;
        queue.push(next);
      }
    }
  }
  return -1;
};

function toIdx(num, n) {
  let row = Math.floor((num - 1) / n),
      col = (num - 1) % n;
  col = (row % 2) === 0 ? col : n - 1 - col;
  row = n - 1 - row;
  return [row, col]
};
```

### [Kill Process](https://github.com/grandyang/leetcode/issues/582)

```js
function killProcess(pid, ppid, kill) {
  let res = [],
    map = {};
  for (let i = 0; i < pid.length; i++) {
    if (!map[ppid[i]]) map[ppid[i]] = [];
    map[ppid[i]].push(pid[i]);
  }
  let queue = [kill];
  while (queue.length) {
    let node = queue.shift();
    res.push(node);
    if (map[node]) {
      for (let child of map[node]) {
        queue.push(child);
      }
    }
  }
  return res;
}
```

#### [All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

Return a list of the values of all nodes that have a distance K from the target node.

构建邻接表，然后 BFS。

```js
var distanceK = function (root, target, K) {
  let adjs = build(null, root);
  let nodes = [target.val],
    seen = new Set(nodes);
  for (let i = 0; i < K; i++) {
    let curr = [];
    for (let node of nodes) {
      if (!adjs[node]) continue;
      for (let next of adjs[node]) {
        if (!seen.has(next)) {
          curr.push(next);
          seen.add(next);
        }
      }
    }
    nodes = curr;
  }
  return nodes;
};

function build(parent, child, adjs = []) {
  if (parent && child) {
    if (!adjs[parent.val]) adjs[parent.val] = [];
    if (!adjs[child.val]) adjs[child.val] = [];
    adjs[parent.val].push(child.val);
    adjs[child.val].push(parent.val); // child => parent 因为 BFS 时还需要从 target node 向上递归
  }
  if (child.left) build(child, child.left, adjs);
  if (child.right) build(child, child.right, adjs);
  return adjs;
}
```

recursion: 会修改 node，添加 parent 指针。

```js
var distanceK = function (root, target, K) {
  if (!root) return [];
  const node = findTarget(root, null, target);
  return findAllKApart(node, K);
};

function findTarget(root, parent, target) {
  if (!root) return null;
  root.parent = parent;
  if (root === target) {
    return root;
  }
  return (
    findTarget(root.left, root, target) || findTarget(root.right, root, target)
  );
}

function findAllKApart(root, k, res = [], seen = new Set()) {
  if (!root || seen.has(root.val)) return res;
  if (k == 0) {
    res.push(root.val);
    return res;
  }
  seen.add(root.val);
  findAllKApart(root.left, k - 1, res, seen);
  findAllKApart(root.right, k - 1, res, seen);
  findAllKApart(root.parent, k - 1, res, seen);
  return res;
}
```
