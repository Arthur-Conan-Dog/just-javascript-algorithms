# Heap

maxPQ 和 minPQ 的使用场景？什么时候用前者什么时候用后者？

注意 heap[0] 使用与否时 index 怎么求。

## Questions

### [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

use a minPQ and continue delete min to keep its size at k.

```js
function kth(nums, k) {
  let minPQ = new PriorityQueue((a, b) => a - b);
  for (let num of nums) {
    minPQ.enqueue(num);
    if (minPQ.size() > k) {
      minPQ.dequeue();
    }
  }
  return minPQ.peek();
}
```

similar questions: Kth smallest element in an array => use a maxPQ and continue del max to keep its size at k.

```js
function kth(nums, k) {
  let maxPQ = new PriorityQueue((a, b) => b - a);
  for (let num of nums) {
    maxPQ.enqueue(num);
    if (maxPQ.size() > k) {
      maxPQ.dequeue();
    }
  }
  return maxPQ.peek();
}
```

### [Trapping Rain Water II](https://leetcode.com/problems/trapping-rain-water-ii/)

https://leetcode.com/problems/trapping-rain-water-ii/discuss/89495/How-to-get-the-solution-to-2-D-%22Trapping-Rain-Water%22-problem-from-1-D-case

Trapping Rain Water 1D 数组只有左右两个纬度，而这一题则是多个方向多个纬度，需要找的是一个“圈”。圈的边界最开始正是矩阵的四条边，那如何通过缩小这个圈来计算容水量呢？由于最小的高度决定了最大的容水量，所以我们需要找到圈的最小高度，用它与它的邻居节点的高度做对比，差值为可以蓄水的量。而找最小的这个过程可以通过使用 MinPQ 来优化。

```js
const directions = [
  [0, 1],
  [0, -1],
  [1, 0],
  [-1, 0],
];

var trapRainWater = function (heightMap) {
  let m = heightMap.length,
    n = heightMap[0].length;
  let pq = new PriorityQueue((a, b) => a[2] - b[2]);
  let visited = [...Array(m)].map(() => Array(n).fill(false));
  for (let i = 0; i < m; i++) {
    pq.enqueue([i, 0, heightMap[i][0]]);
    pq.enqueue([i, n - 1, heightMap[i][n - 1]]);
    visited[i][0] = visited[i][n - 1] = true;
  }
  for (let i = 1; i < n - 1; i++) {
    pq.enqueue([0, i, heightMap[0][i]]);
    pq.enqueue([m - 1, i, heightMap[m - 1][i]]);
    visited[0][i] = visited[m - 1][i] = true;
  }
  let res = 0;
  while (pq.size() > 0) {
    let node = pq.dequeue();
    for (let [dx, dy] of directions) {
      let x = node[0] + dx,
        y = node[1] + dy;
      if (x < 0 || x >= m || y < 0 || y >= n || visited[x][y]) continue;
      res += Math.max(0, node[2] - heightMap[x][y]);
      pq.enqueue([x, y, Math.max(node[2], heightMap[x][y])]);
      visited[x][y] = true;
    }
  }
  return res;
};
```

### [Closest Binary Search Tree Value II](https://github.com/grandyang/leetcode/issues/272)

```js
var closestValue = function (root, target, k) {
  let res = [],
    maxPQ = new PriorityQueue((a, b) => b[0] - a[0]);
  const dfs = (root) => {
    if (!root) return;
    dfs(root.left);
    maxPQ.enqueue([Math.abs(root.val - target), root.val]);
    if (maxPQ.size > k) {
      maxPQ.dequeue();
    }
    dfs(root.right);
  };
  dfs(root);
  while (maxPQ.size > 0) {
    res.push(maxPQ.dequeue());
  }
  return res;
};
```

### [Complete Binary Tree Inserter](https://leetcode.com/problems/complete-binary-tree-inserter/)

O(1) insertion

```js
class CBTInserter {
  constructor(root) {
    this.tree = [root];
    for (let i = 0; i < this.tree.length; i++) {
      if (this.tree[i].left) this.tree.push(this.tree[i].left);
      if (this.tree[i].right) this.tree.push(this.tree[i].right);
    }
  }
  /**
   * @param {number} v
   * @return {number}
   */
  insert(v) {
    let n = this.tree.length,
      node = new TreeNode(v);
    this.tree.push(node);
    let idx = Math.floor((n - 1) / 2);
    if (n % 2 === 1) {
      this.tree[idx].left = node;
    } else {
      this.tree[idx].right = node;
    }
    return this.tree[idx].val;
  }
  /**
   * @return {TreeNode}
   */
  get_root() {
    return this.tree[0];
  }
}
```
