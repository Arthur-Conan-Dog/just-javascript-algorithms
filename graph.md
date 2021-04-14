# Graph

keynotes:

- various graph representations

- graph search algorithms, their time and space complexities

- special types: bipartite graph, union find, etc.

representations:

- Adjacency matrix. v x v, `adj[u][v] = true` u -> v. `adj[u][v]` 也可存储权重。
- Adjacency list. `adj[u] = [v1, v2, v3,...]`
- list of edges. `[[u, v], [v, w]]` u -> v, w -> w.

Graph algorithms:

- Common - Breadth-first Search, Depth-first Search
- Uncommon - Topological Sort, Dijkstra's algorithm
- Rare - Bellman-Ford algorithm, Floyd-Warshall algorithm, Prim's algorithm, Kruskal's algorithm

Corner cases:

- empty graph
- graph with one or two nodes
- graph with multiple components
- graph with cycles
  - self loop
  - negative

## 基本概念

connected / disconnected：所有节点之间都有通路 / 存在个别节点间没有通路

cyclic / acyclic：有 / 无环

directed / undirected：有向 / 无向

degree：度，节点关联边的数目。有向图中又存在：in-degree & out-degree。

DAG：有向无环图 directed acyclic graph

bipartite graph: 二分图。满足所有节点分成两个集合，使得每条边的两端分别来自这两个集合。

tree：相连的无环图; forest：彼此完全不存在路径相连的一堆树

spanning tree：生成树 => 包含原图所有节点的一棵树，不唯一。

## 基本问题与基本算法

个别算法的演变过程参见 [graph-test.js](./graph-test.js)

### 常见表示方法间相互转换

adjacency matrix => adjacency list

```js
function adjMatrixToAdjList(matrix) {
  let n = matrix.length,
    adj = Array.from(Array(n)).map(() => []);
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      if (matrix[i][j]) {
        adj[i].push(j);
      }
    }
  }
  return adj;
}
```

edge list => adjacency list

```js
function edgeListToAdjList(N, edges) {
  let graph = [...Array(N)].map(() => []);
  for (let [v, w] of edges) {
    graph[v].push(w);
    // for undirected graph
    graph[w].push(v);
  }
  return graph;
}
```

adjacency list => adjacency matrix

```js
function adjListToAdjMatrix(adj) {
  let n = adj.length,
    matrix = Array.from(Array(n)).map(() => Array.from(Array(n)).map(() => 0));
  for (let i = 0; i < n; i++) {
    for (let j of adj[i]) {
      matrix[i][j] = 1;
    }
  }
  return matrix;
}
```

adjacency list => edge list

```js
function adjListToEdgeList(adj) {
  let edges = [];
  for (let i = 0; i < adj.length; i++) {
    for (let next of adj[i]) {
      // assume we don't have self loops
      if (i < next) {
        edges.push([i, next]);
      }
    }
  }
  return edges;
}
```

### 基本遍历算法

DFS

```js
function dfsTemplate(adj) {
  let n = adj.length,
    marked = Array.from(Array(n)).map(() => false),
    count = 0;
  function dfs(node) {
    marked[node] = true;
    count++; // do something
    for (let next of adj[node]) {
      if (!marked[next]) dfs(next);
    }
  }

  for (let v = 0; v < adj.length; v++) {
    if (!marked[v]) {
      dfs(v);
    }
  }

  return count;
}
```

- time: O(V+E) => since every vertex and edge will be visited in the worst case.

- space:O(V) => additional space for marked array.

BFS

```js
function bfsTemplate(adj) {
  let n = adj.length,
    marked = Array.from(Array(n)).map(() => false),
    count = 0;
  function bfs(node) {
    let queue = [node];
    marked[node] = true;
    count++; // do something
    while (queue.length) {
      let curr = queue.shift();
      for (let next of adj[curr]) {
        if (!marked[next]) {
          marked[next] = true;
          count++;
          queue.push(next);
        }
      }
    }
  }

  for (let v = 0; v < adj.length; v++) {
    if (!marked[v]) {
      bfs(v);
    }
  }

  return count;
}
```

- time: O(V+E) => since every vertex and edge will be visited in the worst case.

- space:O(V) => additional space for queue.

### 二分图 bipartite graph

```js
function isBipartiteV2(graph) {
  let n = graph.length,
    colors = Array(n).fill(0);

  function marked(node) {
    return colors[node] !== 0;
  }

  function dfs(node, color) {
    if (marked(node)) return colors[node] === color;

    colors[node] = color;
    for (let adj of graph[node]) {
      if (!dfs(adj, -color)) return false;
    }
    return true;
  }

  for (let node = 0; node < n; node++) {
    if (!marked(node) && !dfs(node, 1)) return false;
  }
  return true;
}
```

### 检测环 cycle detection

有向图检测环

```js
function hasCycleForDGV2(graph) {
  let n = graph.length,
    marked = new Set(),
    onPath = new Set();

  function dfs(v) {
    if (onPath.has(v)) return true;
    if (marked.has(v)) return false;

    onPath.add(v);
    for (let next of graph[v]) {
      if (!marked.has(next) && dfs(next)) return true;
    }

    onPath.delete(v);
    marked.add(v);
    return false;
  }

  for (let node = 0; node < n; node++) {
    if (!marked.has(node) && dfs(node)) return true;
  }

  return false;
}
```

有向图检测并输出环

```js
function findCycleForDGV3(graph) {
  let n = graph.length,
    marked = new Set(),
    onPath = new Set(),
    cycle = null;

  function dfs(node, path) {
    if (onPath.has(node)) {
      cycle = [...path, node];
      return false;
    }
    if (marked.has(node)) return true;

    path.push(node);
    onPath.add(node);
    for (let next of graph[node]) {
      if (!marked.has(next) && !dfs(next, path)) return false;
    }
    path.pop(node);
    onPath.delete(node);
    marked.add(node);
    return true;
  }

  for (let node = 0; node < n; node++) {
    if (!marked.has(node) && !dfs(node, [])) return cycle;
  }

  return cycle;
}
```

无向图检测环：由于无向图中两节点间会记录两条边，遍历时需要记录当前节点前序节点的信息（pre node）。

```js
function hasCycleForUGV3(adj) {
  let seen = new Set();

  function dfs(node, pre) {
    seen.add(node);
    for (let next of adj[node]) {
      if (!seen.has(next)) {
        if (dfs(next, node)) {
          return true;
        }
      } else if (next !== pre) {
        return true;
      }
    }
    return false;
  }

  for (let node = 0; node < adj.length; node++) {
    if (!seen.has(node) && dfs(node, -1)) return true;
  }

  return false;
}
```

或者使用 Union Find。需要注意的是得按照 edgesList 而不是 adjList 来遍历，否则节点会被重复访问。

```js
function hasCycleForUGV4(adj) {
  function find(x) {
    if (uf[x] !== x) {
      uf[x] = find(uf[x]);
    }
    return uf[x];
  }
  let edges = adjListToEdgeList(adj),
    uf = [...Array(adj.length)].map((_,idx) => idx);
  for (let [u, v] of edges) {
    let pu = find(u),
      pv = find(v);
    if (pu === pv) return true; // check before union
    uf[v] = u;
  }
  return false;
}
```

另一种 Algo4 提供的思路：

```js
function hasCycleForUGV2(adj) {
  let n = adj.length,
    marked = Array.from(Array(n)).map(() => false);

  function dfs(source, target) {
    marked[source] = true;
    for (let next of adj[source]) {
      // 当前 1 -> 2，排除 2 -> 1，存在 next -> 1，证明有环。
      if (
        (marked[next] && next !== target) ||
        (!marked[next] && dfs(next, source))
      )
        return true;
    }
    return false;
  }

  for (let node = 0; node < adj.length; node++) {
    if (!marked[node] && dfs(node, node)) return true;
  }

  return false;
}
```

### 拓扑排序 topological sort

Given a digraph, put the vertices in order such that all its directed edges point from a vertex earlier in the order to a vertex later in the order (or report that doing so is not possible).

only if the graph is a DAG => determine whether a digraph is a DAG => detect cycle for digraph

DFS

```js
function topologicalSort(graph) {
  let n = graph.length,
    marked = new Set(),
    onPath = new Set(),
    res = [];

  function dfs(node) {
    if (onPath.has(node)) return false;
    if (marked.has(node)) return true;

    onPath.add(node);
    for (let adj of graph[node]) {
      if (!marked.has(adj) && !dfs(adj)) return false;
    }
    onPath.delete(node);
    marked.add(node);
    res.push(node);
    return true;
  }

  for (let node = 0; node < n; node++) {
    if (!marked.has(node) && !dfs(node)) return [];
  }
  return res.reverse(); // 🚨!
}
```

BFS: calc in-degrees, and release node when its in-degree goes to zero.

```js
function topologicalSortBFS(graph) {
  let n = graph.length,
    degrees = Array(n).fill(0);
  for (let node of graph) {
    for (let next of node) {
      degrees[next]++;
    }
  }
  let res = [], queue = [];
  for (let i = 0; i < n; i++) {
    if (degrees[i] === 0) queue.push(i);
  }
  while (queue.length) {
    let node = queue.shift();
    n--;
    res.push(node);
    for (let next of graph[node]) {
      degrees[next]--;
      if (degrees[next] === 0) queue.push(next);
    }
  }
  return n === 0 ? res : [];
}
```

extra: when given graph is ensured to be a DAG

```js
function topologicalSort(graph) {
  let n = graph.length,
    seen = new Set(),
    res = [];

  function dfs(node) {
    for (let adj of graph[node]) {
      if (!seen.has(adj)) {
        dfs(adj);
      }
    }
    seen.add(node);
    res.unshift(node); // or we could reverse edge directions while building graph, then we could simply use res.push(node).
  }

  for (let node = 0; node < n; node++) {
    if (!seen.has(node)) {
      dfs(node);
    }
  }
  return res;
}
```

### 最短路径 edge-weighted shortest paths

edge-weighted digraph: type of adj[v] = List<DirectedEdge>, DirectedEdge: [source, target, weight]

negative cycle: a directed cycle whose total weight (sum of the weights of its edges) is negative.

1. [Floyd-Warshall](https://en.wikipedia.org/wiki/Floyd%E2%80%93Warshall_algorithm)

思路：暴力求解。遍历 i => k => j，若 dis[i][k] + dis[k][j] < dis[i][j] 则更新 dis[i][j]。初始化 dis[i][i] = 0，dis[u][v] = w，其余为 Infinity。

特点：

- 可以求任意两个结点之间的最短路

- 边权可正可负，但不能存在负环

复杂度：

- 时间：O(V^3)

- 空间：O(V^2)

2. [Dijkstra](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm#Pseudocode)

思路：贪心。dist[source] = 0，其他初始化为正无穷。选出目前距离 dist 最近的节点，并以该节点为跳板去计算 source 到该节点的邻接节点的距离。如果 dist[node] + distFromNodeToNext < dist[next]，更新 dist[next] 的值。

由于每一轮需要找最小，为了优化查找可以使用 minPQ。在比较并且需要更新 dist[next] 后，如果 pq 中此时已有 next 节点，更新其对应的 priority（dist） 值；如果不存在则加入 pq。

特点：

- 只能求单源最短路径

- 边权只能为正

- 贪心思想

复杂度：

- 时间：O(ElogV) => worst case, since each edge is visited once, and the find minimum takes logV due to the heap implementation.

- 空间：O(V) => extra space for vertices in minPQ.

3. [Bellman-Ford](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm#Algorithm)

思路：relax 每条边，循环 V 次。初始化 dis[source] = 0。

特点：

- 不能存在负环

复杂度：

- 时间：O(EV)

- 空间：O(V)

改良版：Queue-based Bellman-Ford => 需要再次 relax 的不会是所有节点，只会是 dist 在上一轮被更改了的那些节点。我们可以通过 queue 来追踪这些节点。

### 暂未归类

连通性 & 寻径 connectivity & finding paths

single-source connectivity 判断两个节点 s, v 是否相连 / s, v 两节点间是否存在通路 => single-source paths 找到  s, v 间任一路径

single-source shortest paths 找到 s, v 间最短路径

find connected components

TODO: 最小生成树

## Questions

### bipartiteness

#### [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)

```js
function isBipartite(graph) {
  let n = graph.length,
    colors = Array(n).fill(0);

  function marked(node) {
    return colors[node] !== 0;
  }

  function dfs(node, color) {
    if (marked(node)) return colors[node] === color;

    colors[node] = color;
    for (let adj of graph[node]) {
      if (!dfs(adj, -color)) return false;
    }
    return true;
  }

  for (let node = 0; node < n; node++) {
    if (!marked(node) && !dfs(node, 1)) return false;
  }
  return true;
}
```

#### [Possible Bipartition](https://leetcode.com/problems/possible-bipartition/)

```js
var possibleBipartition = function (N, dislikes) {
  let graph = [...Array(N + 1)].map(() => []);
  for (let [v, w] of dislikes) {
    graph[v].push(w);
    graph[w].push(v);
  }

  let colors = [...Array(N + 1)].fill(0);
  function dfs(node, color) {
    if (colors[node] !== 0) return colors[node] === color;
    colors[node] = color;
    for (let adj of graph[node]) {
      if (!dfs(adj, -color)) return false;
    }
    return true;
  }

  for (let node = 1; node <= N; node++) {
    if (colors[node] === 0 && !dfs(node, 1)) return false;
  }
  return true;
};
```

### topological sort

#### [Course Schedule](https://leetcode.com/problems/course-schedule/)

表示方式转换 + 检测环，如果有环则不存在拓扑排序，则不能完成所有课程。

```js
var canFinish = function (numCourses, prerequisites) {
  let graph = [...Array(numCourses)].map((n) => []);
  for (let [node, prereq] of prerequisites) {
    graph[prereq].push(node);
  }

  let n = graph.length,
    marked = new Set(),
    onPath = new Set();

  function dfs(v) {
    if (onPath.has(v)) return false;
    if (marked.has(v)) return true;

    onPath.add(v);
    for (let next of graph[v]) {
      if (!marked.has(next) && !dfs(next)) return false;
    }

    onPath.delete(v);
    marked.add(v);
    return true;
  }

  for (let node = 0; node < n; node++) {
    if (!marked.has(node) && !dfs(node)) return false;
  }

  return true;
};
```

#### [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

表示方式转换 + 拓扑排序

```js
var findOrder = function (numCourses, prerequisites) {
  let graph = [...Array(numCourses)].map((n) => []);
  for (let [node, prereq] of prerequisites) {
    graph[node].push(prereq); // inverse edge direction to skip reverse res in the end.
  }
  let n = graph.length,
    marked = new Set(),
    onPath = new Set(),
    res = [];

  function dfs(node) {
    if (onPath.has(node)) return false;
    if (marked.has(node)) return true;

    onPath.add(node);
    for (let adj of graph[node]) {
      if (!marked.has(adj) && !dfs(adj)) return false;
    }
    onPath.delete(node);
    marked.add(node);
    res.push(node);
    return true;
  }

  for (let node = 0; node < n; node++) {
    if (!marked.has(node) && !dfs(node)) return [];
  }
  return res;
};
```

bfs 解法

```js
var findOrder = function (numCourses, prerequisites) {
  let queue = [],
    res = [];
  let graph = new Map(),
    degrees = Array(numCourses).fill(0);

  for (let [next, pre] of prerequisites) {
    if (graph.has(pre)) {
      graph.get(pre).push(next);
    } else {
      graph.set(pre, [next]);
    }
    degrees[next]++;
  }

  for (let i = 0; i < degrees.length; i++) {
    if (degrees[i] === 0) {
      queue.push(i);
    }
  }

  while (queue.length > 0) {
    let node = queue.shift();
    if (graph.has(node)) {
      for (let adj of graph.get(node)) {
        degrees[adj]--;
        if (degrees[adj] === 0) {
          queue.push(adj);
        }
      }
    }
    res.push(node);
  }

  return res.length === numCourses ? res : [];
};
```

#### [Sequence Reconstruction](https://github.com/grandyang/leetcode/issues/444)

拓扑排序。每次能从 queue 中能取出的节点，即入度为 0 的节点是唯一的，若不满足则说明存在构建的多种可能。

```js
function sequenceReconstruction(org, seq) {
  let n = org.length,
    adj = [...Array(n + 1)].map(() => new Set()),
    degrees = Array(n + 1).fill(0);
  for (let s of seq) {
    for (let i = 1; i < s.length; i++) {
      let u = s[i - 1],
        v = s[i];
      if (!adj[u].has(v)) {
        adj[u].add(v);
        degrees[v]++;
      }
    }
  }
  let queue = [];
  for (let i = 1; i <= n; i++) {
    if (degrees[i] === 0) {
      queue.push(i);
    }
  }
  while (queue.length) {
    let node = queue.shift();
    for (let next of adj[node]) {
      degrees[next]--;
      if (degrees[next] === 0) {
        queue.push(next);
      }
    }
    if (queue.length > 1) return false;
  }
  return true;
}
```

### shortest path

#### [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)

Floyd-Warshall

```js
var findTheCity = function (n, edges, distanceThreshold) {
  let dis = [...Array(n)].map(() => Array(n).fill(Infinity));
  for (let i = 0; i < n; i++) dis[i][i] = 0
  for (let [u, v, w] of edges) {
    dis[v][u] = dis[u][v] = w;
  }
  for (let k = 0; k < n; k++) {
    for (let i = 0; i < n; i++) {
      for (let j = 0; j < n; j++) {
        dis[i][j] = Math.min(dis[i][j], dis[i][k] + dis[k][j]);
      }
    }
  }

  let res = 0,
    min = n;
  for (let i = 0; i < n; ++i) {
    let curr = 0;
    for (let j = 0; j < n; ++j) {
      if (dis[i][j] <= distanceThreshold) curr++;
    }
    if (curr <= min) {
      min = curr;
      res = i;
    }
  }
  return res;
};
```

#### [Network Delay Time](https://leetcode.com/problems/network-delay-time/)

Floyd

```js
function networkDelayTime(times, N, K) {
  let dist = [...Array(N + 1)].map(() => Array(N + 1).fill(Infinity));
  for (let i = 1; i <= N; i++) dist[i][i] = 0;
  for (let [u, v, w] of times) dist[u][v] = w;

  for (let k = 1; k <= N; k++) {
    for (let i = 1; i <= N; i++) {
      for (let j = 1; j <= N; j++) {
        dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
      }
    }
  }

  let max = -1;
  for (let i = 1; i <= N; i++) {
    max = Math.max(max, dist[K][i]);
  }
  return max === Infinity ? -1 : max;
}
```

Dijkstra

```js
function networkDelayTime(times, N, K) {
  let adj = [...Array(N + 1)].map(() => []),
    dist = Array(N + 1).fill(Infinity);
  for (let [u, v, w] of times) {
    adj[u].push([v, w]);
  }

  // [node, priority(dist from source to current node)]
  const pq = new PriorityQueue((a, b) => a[1] - b[1]);
  pq.enqueue([K, 0]);
  dist[K] = 0;

  while (pq.size() > 0) {
    let node = pq.dequeue()[0];
    for (let [next, w] of adj[node]) {
      if (dist[next] > dist[node] + w) {
        dist[next] = dist[node] + w;
        if (pq.contains(next)) {
          pq.change(next, dist[next]);
        } else {
          pq.enqueue([next, dist[next]]);
        }
      }
    }
  }
  let max = 0;
  for (let i = 1; i <= N; i++) max = Math.max(max, dist[i]);
  return max === Infinity ? -1 : max;
}
```

Bellman-Ford

```js
var networkDelayTime = function (times, N, K) {
  let time = Array(N + 1).fill(Infinity);
  time[K] = 0;

  for (let i = 0; i < N; i++) {
    for (const [u, v, w] of times) {
      if (time[u] === Infinity) continue;
      if (time[v] > time[u] + w) {
        time[v] = time[u] + w;
      }
    }
  }

  let res = 0;
  for (let i = 1; i <= N; i++) {
    if (time[i] === Infinity) return -1;
    res = Math.max(res, time[i]);
  }
  return res;
};
```

#### [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

Shortest path with exactly k edges in a directed and weighted graph.

```js
var findCheapestPrice = function (n, flights, src, dst, K) {
  let graph = [...Array(n + 1)].map(() => []),
    dist = Array(n + 1).fill(Infinity);
  for (let [u, v, w] of flights) {
    graph[u].push([v, w]);
  }

  const pq = new PriorityQueue((a, b) => a[0] - b[0]);
  pq.enqueue([0, src, K + 1]);
  dist[src] = 0;

  while (pq.size() > 0) {
    let [price, node, stops] = pq.dequeue();
    if (node === dst) return price;
    if (stops > 0) {
      for (let [v, w] of graph[node]) {
        pq.enqueue([price + w, v, stops - 1]);
      }
    }
  }

  return -1;
};
```

### DFS

#### [All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/)

given a DAG, find all possible paths from node 0 to node n - 1.

```js
var allPathsSourceTarget = function (graph) {
  let n = graph.length, paths = [];
  function dfs(node, path) {
    if (node === n - 1) {
      paths.push(path.concat());
      return;
    }
    for (let adj of graph[node]) {
      path.push(adj);
      dfs(adj, path);
      path.pop(adj);
    }
  }
  dfs(0, [0]);
  return paths;
};
```

#### [Number of Islands](https://leetcode.com/problems/number-of-islands/)

```js
var numIslands = function (grid) {
  if (!grid.length || !grid[0].length) return 0;

  let rows = grid.length,
    cols = grid[0].length,
    directions = [
      [0, 1],
      [0, -1],
      [1, 0],
      [-1, 0],
    ];

  const dfs = (i, j) => {
    if (i < 0 || i >= rows || j < 0 || j >= cols) return;
    if (grid[i][j] === "0") return;

    grid[i][j] = "0";
    for (let [dx, dy] of directions) {
      dfs(i + dx, j + dy);
    }
  };

  let res = 0;
  for (let i = 0; i < rows; i++)
    for (let j = 0; j < cols; j++)
      if (grid[i][j] === "1") {
        dfs(i, j);
        res++;
      }

  return res;
};
```

#### [Max Area of Island](https://leetcode.com/problems/max-area-of-island/)

```js
var maxAreaOfIsland = function (grid) {
  if (!grid.length || !grid[0].length) return 0;

  let rows = grid.length,
    cols = grid[0].length,
    directions = [
      [0, 1],
      [0, -1],
      [1, 0],
      [-1, 0],
    ];

  const dfs = (i, j) => {
    if (i < 0 || i >= rows || j < 0 || j >= cols) return 0;
    if (grid[i][j] === 0) return 0;

    grid[i][j] = 0;
    let res = 1;
    for (let [dx, dy] of directions) {
      res += dfs(i + dx, j + dy);
    }
    return res;
  };

  let max = 0;
  for (let i = 0; i < rows; i++)
    for (let j = 0; j < cols; j++)
      if (grid[i][j] === 1) {
        max = Math.max(max, dfs(i, j));
      }

  return max;
};
```

#### [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

Start DFS from nodes at boundary

```js
var solve = function (board) {
  if (!board.length || !board[0].length) return;

  let rows = board.length,
    cols = board[0].length,
    directions = [
      [0, 1],
      [0, -1],
      [1, 0],
      [-1, 0],
    ];

  const dfs = (i, j) => {
    if (i < 0 || i >= rows || j < 0 || j >= cols) return;
    if (board[i][j] !== "O") return;

    board[i][j] = "#";
    for (let [dx, dy] of directions) {
      dfs(i + dx, j + dy);
    }
  };

  for (let i = 0; i < rows; i++) {
    dfs(i, 0);
    dfs(i, cols - 1);
  }
  for (let i = 1; i < cols - 1; i++) {
    dfs(0, i);
    dfs(rows - 1, i);
  }
  for (let i = 0; i < rows; i++)
    for (let j = 0; j < cols; j++)
      if (board[i][j] === "O") board[i][j] = "X";
      else if (board[i][j] === "#") board[i][j] = "O";
};
```

#### [Number of Closed Islands](https://leetcode.com/problems/number-of-closed-islands/)

DFS from each unvisited node; connected components

```js
var closedIsland = function (grid) {
  if (!grid.length || !grid[0].length) return 0;

  let rows = grid.length,
    cols = grid[0].length,
    directions = [
      [0, 1],
      [0, -1],
      [1, 0],
      [-1, 0],
    ];

  const dfs = (i, j) => {
    if (i < 0 || i >= rows || j < 0 || j >= cols) return;
    if (grid[i][j] !== 0) return;

    grid[i][j] = -1;
    for (let [dx, dy] of directions) {
      dfs(i + dx, j + dy);
    }
  };

  for (let i = 0; i < rows; i++) {
    dfs(i, 0);
    dfs(i, cols - 1);
  }
  for (let i = 1; i < cols - 1; i++) {
    dfs(0, i);
    dfs(rows - 1, i);
  }

  let res = 0;
  for (let i = 1; i < rows - 1; i++)
    for (let j = 1; j < cols - 1; j++)
      if (grid[i][j] === 0) {
        dfs(i, j);
        res++;
      }

  return res;
};
```

#### [Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/)

single connected component

```js
var canVisitAllRooms = function (rooms) {
  const marked = new Set();
  const dfs = (node) => {
    marked.add(node);
    for (let next of rooms[node]) {
      if (!marked.has(next)) dfs(next);
    }
  };

  dfs(0);
  for (let i = 0; i < rooms.length; i++) {
    if (!marked.has(i)) return false;
  }
  return true;
};
```

#### [Redundant Connection](https://leetcode.com/problems/redundant-connection/)

cycle find; TODO: union find

build graph progressively, before adding edge [u,v], check if there already existed a path from u <=> v.

```js
var findRedundantConnection = function (edges) {
  const graph = [...Array(edges.length + 1)].map(() => []);

  function hasCycle(source, target, marked = new Set()) {
    if (source === target) return true;

    marked.add(source);
    for (let next of graph[source]) {
      if (!marked.has(next) && hasCycle(next, target, marked)) {
        return true;
      }
    }
    return false;
  }

  for (let edge of edges) {
    const [u, v] = edge;
    if (hasCycle(u, v)) return [u, v];
    graph[u].push(v);
    graph[v].push(u);
  }
};
```

#### [TODO: Redundant Connection II](https://leetcode.com/problems/redundant-connection-ii/)

```js
```

#### [Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/)

status[node] = 0: node has not been visited yet
status[node] = 1: node is on the seeing path
status[node] = 2: node has been seen

```js
var eventualSafeNodes = function (graph) {
  let res = [],
    n = graph.length,
    status = Array(n).fill(0);

  const dfs = (node) => {
    if (status[node] !== 0) return status[node] === 2; // if status[node] === 1, means it is in a cycle and it cannot be included in the result.
    status[node] = 1;
    for (let next of graph[node]) {
      if (!dfs(next)) return false;
    }
    status[node] = 2;
    return true;
  };

  for (let i = 0; i < n; i++) {
    if (dfs(i)) res.push(i);
  }
  return res;
};
```

#### [Time Needed to Inform All Employees](https://leetcode.com/problems/time-needed-to-inform-all-employees/)

```js
var numOfMinutes = function (n, headID, manager, informTime) {
  let graph = [...Array(n)].map(() => []);
  manager.forEach((u, v) => {
    if (u !== -1) graph[u].push(v);
  });

  function dfs(node, max = 0) {
    for (let adj of graph[node]) {
      max = Math.max(max, dfs(adj));
    }
    return max + informTime[node];
  }

  return dfs(headID);
};
```

#### [Evaluate Division](https://leetcode.com/problems/evaluate-division/)

1. build a directed weighted graph by adjacency list. if a / b = 5, then a => b = 5, b => a = 0.2;

2. for each query, try to find a path in the graph that can link the nodes in the query. eg. a / b = 2, b / c = 3, then a / c => (a / b) * (b / c) => 2 * 3 = 6;

3. if value is null, means there's no such path; if has value, update the adjacency list by adding two new edges to avoid duplicate computation.

```js
var calcEquation = (equations, values, queries) => {
  let res = [], adjs = buildGraph(equations, values);
  for (let [from, to] of queries) {
    let val = dfs(adjs, from, to);
    res.push(val ? val : -1.0);
    if (val) {
      adjs.get(from).set(to, val);
      adjs.get(to).set(from, 1 / val);
    }
  }
  return res;
};

function buildGraph(equations, values) {
  let adjs = new Map();
  for (let i = 0; i < equations.length; i++) {
    let [from, to] = equations[i], val = values[i];
    if (!adjs.has(from)) adjs.set(from, new Map());
    if (!adjs.has(to)) adjs.set(to, new Map());
    adjs.get(from).set(to, val);
    adjs.get(to).set(from, 1 / val);
  }
  return adjs;
}

function dfs(adjs, from, to, product = 1, seen = new Set()) {
  if (!adjs.has(from)) return null;
  seen.add(from);
  let neighbors = [...adjs.get(from).keys()];
  for (let next of neighbors) {
    let current = product * adjs.get(from).get(next);
    if (next === to) return current;
    if (!seen.has(next)) {
      let value = dfs(adjs, next, to, current, seen);
      if (value) return value;
    }
  }
  return null;
};
```

### BFS

#### [01 Matrix](https://leetcode.com/problems/01-matrix/)

```js
var updateMatrix = function (matrix) {
  let rows = matrix.length,
    cols = matrix[0].length,
    directions = [
      [0, 1],
      [1, 0],
      [0, -1],
      [-1, 0],
    ];

  let res = [...Array(rows)].map(() => Array(cols).fill(0));
  const bfs = (i, j) => {
    let queue = [[i, j]],
      depth = 1;
    while (queue.length) {
      let size = queue.length;
      while (size--) {
        let [row, col] = queue.shift();
        for (let [dx, dy] of directions) {
          let x = row + dx,
            y = col + dy;
          if (x < 0 || x >= rows || y < 0 || y >= cols) continue;
          if (matrix[x][y] === 0) return depth;
          queue.push([x, y]);
        }
      }
      depth++;
    }
    return depth;
  };

  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (matrix[i][j] === 1) res[i][j] = bfs(i, j);
    }
  }
  return res;
};
```

#### [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

```js
var orangesRotting = function (grid) {
  let rows = grid.length,
    cols = grid[0].length,
    directions = [
      [0, 1],
      [1, 0],
      [0, -1],
      [-1, 0],
    ];

  let queue = [],
    freshes = 0;
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (grid[i][j] === 2) queue.push([i, j]);
      else if (grid[i][j] === 1) freshes++;
    }
  }

  let minutes = 0;
  while (queue.length > 0 && freshes > 0) {
    minutes++;
    let size = queue.length;
    while (size--) {
      let [i, j] = queue.shift();
      for (let [dx, dy] of directions) {
        let x = i + dx,
          y = j + dy;
        if (x < 0 || x >= rows || y < 0 || y >= cols) continue;
        if (grid[x][y] !== 1) continue;
        freshes--;
        grid[x][y] = 2;
        queue.push([x, y]);
      }
    }
  }

  return freshes === 0 ? minutes : -1;
};
```

#### [As Far from Land as Possible](https://leetcode.com/problems/as-far-from-land-as-possible/)

```js
var maxDistance = function (grid) {
  let rows = grid.length,
    cols = grid[0].length,
    directions = [
      [0, 1],
      [1, 0],
      [0, -1],
      [-1, 0],
    ];

  let queue = [];
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (grid[i][j] === 1) queue.push([i, j]);
    }
  }

  if (queue.length === 0 || queue.length === rows * cols) return -1;

  let dis = [...Array(rows)].map(() => Array(cols).fill(0)),
    max = 0;
  while (queue.length > 0) {
    let size = queue.length;
    while (size--) {
      let [i, j] = queue.shift();
      for (let [dx, dy] of directions) {
        let x = i + dx,
          y = j + dy;
        if (x < 0 || x >= rows || y < 0 || y >= cols) continue;
        if (grid[x][y] === 1) continue;
        dis[x][y] = Math.max(
          dis[i][j],
          dis[i][j] + Math.abs(i - x) + Math.abs(j - y)
        );
        max = Math.max(max, dis[x][y]);
        grid[x][y] = 1;
        queue.push([x, y]);
      }
    }
  }

  return max;
};
```

### Others

#### [Find the Town Judge](https://leetcode.com/problems/find-the-town-judge/)

一开始用了两个数组 outs & ins 分别代表出度和入度，其实可以简化到只用一个数组。

```js
var findJudge = function (N, trust) {
  let degrees = Array(N + 1).fill(0);
  for (let [v, w] of trust) {
    degrees[v]--;
    degrees[w]++;
  }

  for (let i = 1; i <= N; i++) {
    if (degrees[i] === N - 1) return i;
  }
  return -1;
};
```

#### [Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/)

How many MHTs can a graph have at most? => 2 即最长路径上的节点数为偶数，中间节点有两个的情况。

思路：不断拿掉最外层叶子，等于从可能存在的最长路径两端向内部收缩，最终留下的就是最长路径上的中间节点。

```js
var findMinHeightTrees = function (n, edges) {
  if (n === 1) return [0];

  let adj = edgeListToAdjList(n, edges),
    leaves = new Set();
  for (let i = 0; i < adj.length; i++) if (adj[i].size === 1) leaves.add(i);

  while (n > 2) {
    n -= leaves.size;
    let temp = new Set();
    for (let leaf of leaves) {
      let next = [...adj[leaf]][0];
      adj[next].delete(leaf);
      if (adj[next].size === 1) temp.add(next);
    }
    leaves = temp;
  }

  return [...leaves];
};
```
