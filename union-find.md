# Union Find

[key points](<https://leetcode.com/discuss/general-discussion/1072418/Disjoint-Set-Union-(DSU)Union-Find-A-Complete-Guide>):

- determine use which info as parent id

- path compression: optimize find()

- union by rank: optimize union()

common questions:

- how many sets?(count)

- set detail?

## [Implementation](<https://leetcode.com/discuss/general-discussion/1072418/Disjoint-Set-Union-(DSU)Union-Find-A-Complete-Guide>)

```js
// 1. initialize uf with size & parent id.
let n = grid.length,
  uf = Array(n * n * 4),
  count = n * n * 4;
for (let i = 0; i < n * n * 4; i++) uf[i] = i;
// 2. implement union & find logic.
function union(x, y) {
  let px = find(x),
    py = find(y);
  if (px !== py) {
    uf[px] = py;
    count--;
  }
}
function find(x) {
  if (uf[x] !== x) {
    uf[x] = find(uf[x]); // path compression
  }
  return uf[x];
}
// 3. call union according to questions.
```

注意：

- uf size 有时与所给的数据结构大小并不一定完全对应，或是初始化时不一定需要全部添加 parent id，参考 number of islands。

- uf count 如果想封装得更好，会牺牲一些空间。

- uf 不一定通过数组来 track parent id，也可以用 map，看哪种更方便。

## Questions

### [Number of Islands](https://leetcode.com/problems/number-of-islands/)

使用 template：

```js
function numIslands(grid) {
  if (grid.length == 0) return 0;

  let m = grid.length,
    n = grid[0].length,
    uf = Array(m * n),
    count = 0;
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (grid[i][j] === "1") {
        uf[i * n + j] = i * n + j;
        count++;
      }
    }
  }
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (grid[i][j] == "1") {
        if (i > 0 && grid[i - 1][j] == "1") union(i * n + j, (i - 1) * n + j);
        if (j > 0 && grid[i][j - 1] == "1") union(i * n + j, i * n + (j - 1));
      }
    }
  }

  function union(x, y) {
    let px = find(x),
      py = find(y);
    if (px !== py) {
      uf[px] = py;
      count--;
    }
  }

  function find(x) {
    if (uf[x] !== x) {
      uf[x] = find(uf[x]);
    }
    return uf[x];
  }
  return count;
}
```

OR，封装：

```js
function numIslands(grid) {
  if (grid.length == 0) return 0;

  let m = grid.length,
    n = grid[0].length;
  let uf = new UnionFind(m * n);
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (grid[i][j] == "1") {
        uf.parents[i * n + j] = i * n + j;
        if (i > 0 && grid[i - 1][j] == "1")
          uf.union(i * n + j, (i - 1) * n + j); // union current & top
        if (j > 0 && grid[i][j - 1] == "1")
          uf.union(i * n + j, i * n + (j - 1)); // union current & left
      }
    }
  }
  return uf.size();
}

class UnionFind {
  constructor(size) {
    this.parents = Array(size).fill(-1);
  }

  find(x) {
    if (this.parents[x] !== x) {
      this.parents[x] = this.find(this.parents[x]);
    }
    return this.parents[x];
  }

  union(x, y) {
    let px = this.find(x);
    let py = this.find(y);
    if (px !== py) {
      this.parents[px] = py;
    }
  }

  size() {
    let set = new Set();
    for (let i = 0; i < this.parents.length; i++) {
      if (this.parents[i] != -1) set.add(this.find(i));
    }
    return set.size;
  }
}
```

_Note: why only union on top and left directions? What about right and bottom? => checking right for (i, j) is equivalent to checking left for (i, j + 1)._

### [Regions Cut By Slashes](https://leetcode.com/problems/regions-cut-by-slashes/)

把每一格想像成对角线分割出的四个小三角形，编号按顺时针依次为 0,1,2,3（这样编号的好处是便于我们找 idx 间的对应关系）。

```js
function regionsBySlashes(grid) {
  let n = grid.length,
    uf = Array(n * n * 4),
    count = n * n * 4;
  for (let i = 0; i < n * n * 4; i++) uf[i] = i;
  function union(x, y) {
    let px = find(x),
      py = find(y);
    if (px !== py) {
      uf[px] = py;
      count--;
    }
  }
  function find(x) {
    if (uf[x] !== x) {
      uf[x] = find(uf[x]);
    }
    return uf[x];
  }
  function g(i, j, k) {
    return (i * n + j) * 4 + k;
  }
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      if (i > 0) union(g(i - 1, j, 2), g(i, j, 0));
      if (j > 0) union(g(i, j - 1, 1), g(i, j, 3));
      if (grid[i][j] !== "/") {
        union(g(i, j, 0), g(i, j, 1));
        union(g(i, j, 2), g(i, j, 3));
      }
      if (grid[i][j] !== "\\") {
        union(g(i, j, 0), g(i, j, 3));
        union(g(i, j, 1), g(i, j, 2));
      }
    }
  }
  return count;
}
```

另一种思路是，扩充矩阵为 3n \* 3n 的大小，来表示斜线和反斜线，然后此问题就会变成一般的 number of islands 问题。

```js
function dfs(g, i, j) {
  if (i >= 0 && j >= 0 && i < g.length && j < g.length && g[i][j] == 0) {
    g[i][j] = 1;
    dfs(g, i - 1, j);
    dfs(g, i + 1, j);
    dfs(g, i, j - 1);
    dfs(g, i, j + 1);
  }
}

var regionsBySlashes = function (grid) {
  let n = grid.length,
    g = [...Array(n * 3)].map(() => Array(n * 3).fill(0));
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      if (grid[i][j] === "/")
        g[i * 3][j * 3 + 2] = g[i * 3 + 1][j * 3 + 1] = g[i * 3 + 2][j * 3] = 1;
      if (grid[i][j] === "\\")
        g[i * 3][j * 3] = g[i * 3 + 1][j * 3 + 1] = g[i * 3 + 2][j * 3 + 2] = 1;
    }
  }
  let res = 0;
  for (let i = 0; i < g.length; i++) {
    for (let j = 0; j < g.length; j++) {
      if (g[i][j] === 0) {
        dfs(g, i, j);
        res++;
      }
    }
  }
  return res;
};
```

### [Number of Provinces](https://leetcode.com/problems/number-of-provinces/)

```js
var findCircleNum = function (isConnected) {
  let n = isConnected.length,
    uf = [...Array(n + 1)].map((_, idx) => idx),
    count = n;
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      if (isConnected[i][j]) {
        union(i + 1, j + 1);
      }
    }
  }
  function union(x, y) {
    let px = find(x),
      py = find(y);
    if (px !== py) {
      uf[px] = py;
      count--;
    }
  }
  function find(x) {
    if (uf[x] !== x) {
      uf[x] = find(uf[x]);
    }
    return uf[x];
  }
  return count;
};
```

### [Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/)

parent id => char to charCode。

```js
var equationsPossible = function (equations) {
  function union(x, y) {
    let px = find(x),
      py = find(y);
    if (px !== py) {
      uf[px] = py;
    }
  }
  function find(x) {
    if (uf[x] !== x) {
      uf[x] = find(uf[x]);
    }
    return uf[x];
  }
  let uf = [...Array(26)].map((_, idx) => idx);
  for (let eq of equations) {
    let x = eq[0],
      y = eq[3],
      isEqual = eq[1] === "=";
    if (isEqual) {
      union(x.charCodeAt() - 97, y.charCodeAt() - 97);
    }
  }
  for (let eq of equations) {
    let x = eq[0],
      y = eq[3],
      notEqual = eq[1] === "!";
    if (notEqual) {
      let px = find(x.charCodeAt() - 97),
        py = find(y.charCodeAt() - 97);
      if (px === py) return false;
    }
  }
  return true;
};
```

### [Accounts Merge](https://leetcode.com/problems/accounts-merge/)

see one record as a group, use idx as group id. use a map to track email => id mappings.

when we find an existing email, union two groups.

for each email, find the group it belongs to, and merge it into that group.

```js
var accountsMerge = function (accounts) {
  // function union & find...
  let uf = [...Array(accounts.length)].map((_, idx) => idx),
    emailToGroupId = new Map(); // email:string => groupId: number
  for (let i = 0; i < accounts.length; i++) {
    for (let j = 1; j < accounts[i].length; j++) {
      if (!emailToGroupId.has(accounts[i][j])) {
        emailToGroupId.set(accounts[i][j], i);
      } else {
        union(emailToGroupId.get(accounts[i][j]), i);
      }
    }
  }

  let groups = new Map(); // groupId:number => emails: string[]
  for (let [email, id] of emailToGroupId) {
    let root = find(id);
    if (groups.has(root)) {
      groups.get(root).push(email);
    } else {
      groups.set(root, [email]);
    }
  }

  let res = [];
  for (let [id, emails] of groups) {
    emails.sort();
    emails.unshift(accounts[id][0]);
    res.push(emails);
  }

  return res;
};
```

### [Sentence Similarity II](https://github.com/grandyang/leetcode/issues/737)

use map to track node => parent relationship between strings instead of array index.

```js
function areSentencesSimilar(s1, s2, pairs) {
  if (s1.length !== s2.length) return false;
  function find(x) {
    if (!uf.has(x)) uf.set(x, x);
    if (uf.get(x) === x) return x;
    uf.set(x, find(uf.get(x)));
    return uf.get(x);
  }
  let uf = new Map();
  for (let [x, y] of pairs) {
    uf.set(find(x), find(y));
  }
  for (let i = 0; i < s1.length; i++) {
    if (find(s1[i]) !== find(s2[i])) {
      return false;
    }
  }
  return true;
}
```

### [Evaluate Division](https://leetcode.com/problems/evaluate-division/)

a / b = factor, set b as parent of a, and record the factor.

root: node => its top parent; dist: node => factor to its top parent.

given a / b = v, a / rootA = factorA, b / rootB = factorB, calculate rootA / rootB:

rootA / rootB = (a / factorA) \* (factorB / b) = (a \* factorA) / (b \* factorB) = v \* (factorB / factorA) => dist.set(r1, (dist.get(to) \* values[i]) / dist.get(from));

given a => factorA \* rootA, b => factorB \* rootB, how to calculate a / b?

if rootA = rootB, then a / b = (factorA \* root) / (factorB \* root) = fA / fB => dist.get(from) / dist.get(to);

update dist while doing path compression in find(): dist.set(s, dist.get(s) \* dist.get(lastP));

```js
var calcEquation = function (equations, values, queries) {
  let root = new Map(),
    dist = new Map();
  for (let i = 0; i < equations.length; i++) {
    let [from, to] = equations[i];
    let r1 = find(from, root, dist),
      r2 = find(to, root, dist);
    root.set(r1, r2);
    dist.set(r1, (dist.get(to) * values[i]) / dist.get(from));
  }
  let res = [];
  for (let [from, to] of queries) {
    if (!root.has(from) || !root.has(to)) {
      res.push(-1.0);
      continue;
    }
    let r1 = find(from, root, dist),
      r2 = find(to, root, dist);
    if (r1 !== r2) {
      res.push(-1.0);
      continue;
    }
    res.push(dist.get(from) / dist.get(to));
  }
  return res;
};

function find(node, root, dist) {
  if (!root.has(node)) {
    root.set(node, node);
    dist.set(node, 1.0);
    return node;
  }
  if (root.get(node) === node) return node;
  let lastP = root.get(node),
    p = find(lastP, root, dist);
  root.set(node, p);
  dist.set(node, dist.get(node) * dist.get(lastP));
  return p;
}
```

### [Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/)

Connected stones can be reduced to 1 stone,
the maximum stones can be removed = stones number - islands number.
so just count the number of "islands".

```js
var removeStones = (stones) => {
  function find(x) {
    if (x !== uf[x]) {
      uf[x] = find(uf[x]);
    }
    return uf[x];
  }
  function union(x, y) {
    uf[find(x)] = find(y);
  }
  let uf = [...Array(20000)].map((_, idx) => idx);
  for (let stone of stones) {
    union(stone[0], stone[1] + 10000);
  }
  let groups = new Set();
  for (let stone of stones) {
    groups.add(find(stone[0]));
  }
  return stones.length - groups.size;
};
```
