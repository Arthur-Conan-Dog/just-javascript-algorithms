# Tree

Keynotes:

- undirected and connected acyclic graph

- special types: binary tree, BST, Trie.

(TODO: B-tree, B+ tree, red-black tree)

- recursion is a common approach

Routines:

- traversal: pre-order, in-order, post-order, level-ordered

Corner cases:

- empty tree
- single node
- two nodes
- very skewed tree(like a linked list)
- types: whether it is guaranteed to be valid

## Types

### [Binary Tree](https://en.wikipedia.org/wiki/Binary_tree#Properties_of_binary_trees)

#### pre-order: root, left, right

recursion:

```js
var preorderTraversal = function (root) {
  if (!root) return [];

  let res = [root.val];
  if (root.left) res = res.concat(preorderTraversal(root.left));
  if (root.right) res = res.concat(preorderTraversal(root.right));

  return res;
};
```

iteration: (use stack)

由于需要序列出栈为 [DLR]，所以需要 R 先入栈，即 [RLD]。

```js
var preorderTraversal = function (root) {
  if (!root) return [];

  let stack = [root],
    res = [];
  while (stack.length) {
    let node = stack.pop();
    res.push(node.val);
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }

  return res;
};
```

n-ary:

```js
// recursion
if (root.children && root.children.length) {
  root.children.forEach((node) => {
    res = res.concat(preorder(node));
  });
}

// iteration
if (node.children && node.children.length) {
  node.children.reverse().forEach((child) => stack.push(child));
}
```

#### in-order: left, root, right

recursion:

```js
var inorderTraversal = function (root) {
  if (!root) return [];

  let res = [];
  if (root.left) res = res.concat(inorderTraversal(root.left));
  res.push(root.val);
  if (root.right) res = res.concat(inorderTraversal(root.right));

  return res;
};
```

iteration:

```js
var inorderTraversal = function (root) {
  if (!root) return [];

  let stack = [],
    res = [];
  while (root || stack.length) {
    if (root) {
      stack.push(root);
      root = root.left;
    } else {
      root = stack.pop();
      res.push(root.val);
      root = root.right;
    }
  }

  return res;
};
```

Morris traversal: not using recursion or helper stack

对于每个节点，其前序节点是唯一的，可以在遍历的过程中建立两者间的关系。当 curr 不存在左子树时，情况比较简单，curr.val 可以被记录，并向右子树继续遍历。当 curr 存在左子树时，先从左子树中找到当前节点的前序节点 prev，并让 prev.right 指向 curr，再深入左子树。若前序节点已存在，则需要修正指针，避免留下环，并深入右子树。

```js
var inorderTraversal = function (root) {
  if (!root) return [];

  let res = [],
    curr = root,
    prev = null;

  while (curr !== null) {
    if (curr.left === null) {
      res.push(curr.val);
      curr = curr.right;
    } else {
      prev = curr.left;
      while (prev.right !== null && prev.right !== curr) prev = prev.right;

      if (prev.right === null) {
        // link curr to its ancestor
        prev.right = curr;
        curr = curr.left;
      }

      if (prev.right === curr) {
        // remove link set by previous step, recover tree
        prev.right = null;
        res.push(curr.val);
        curr = curr.right;
      }
    }
  }

  return res;
};
```

中序遍历有时需要用到前序节点的一些属性，此时需要设置一个 prev pointer，并在递归中不断更新指向。相关应用参见：BST validation etc.

#### post-order: left, right, root

recursion:

```js
var postorderTraversal = function (root) {
  if (!root) return [];

  let res = [];
  if (root.left) res = res.concat(postorderTraversal(root.left));
  if (root.right) res = res.concat(postorderTraversal(root.right));

  return [...res, root.val];
};
```

iteration:

思路一：出栈所需序列 [LRD] 反过来为 [DRL]，可以通过调整 pre-order 得到。

```js
var postorderTraversal = function (root) {
  if (!root) return [];

  let res = [],
    stack = [root];
  while (stack.length) {
    let node = stack.pop();
    res.push(node.val);

    if (node.left) stack.push(node.left);
    if (node.right) stack.push(node.right);
  }

  return res.reverse();
};
```

思路二 (optional): 出栈序列 [LRD]，则可以令入栈顺序为 [LRD]，每次 pop 出的元素值放至 res 最前端。跟上一种对比等于在过程中 reverse 结果数组（`res.unshift(node.val);` ）。

```js
var postorderTraversal = function (root) {
  if (!root) return [];

  let res = [],
    stack = [root];

  while (stack.length) {
    let node = stack.pop();
    res.unshift(node.val);

    if (node.left) stack.push(node.left);
    if (node.right) stack.push(node.right);
  }

  return res;
};
```

n-ary:

```js
// iteration
if (node.children && node.children.length) {
  node.children.forEach((child) => stack.push(child));
}
```

#### level-order

recursion:

```js
var levelOrder = function (root) {
  let res = [];
  helper(root, 0, res);
  return res;
};

function helper(node, depth, res) {
  if (!node) return;
  if (depth === res.length) res.push([]);

  res[depth].push(node.val);
  helper(node.left, depth + 1, res);
  helper(node.right, depth + 1, res);
}
```

iteration:

```js
var levelOrder = function (root) {
  if (!root) return [];

  let res = [],
    queue = [root];
  while (queue.length) {
    let curr = [],
      size = queue.length;
    while (size--) {
      let node = queue.shift();
      curr.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    res.push(curr);
  }

  return res;
};
```

#### recovery

[Preorder + Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

preorder 1[245](36)
inorder [425]1(36)

rootVal = preorder[0]
rootIdx = inorder.indexOf(val) => to reduce time, could record node's idx in advance.
leftLen = rootIdx - i1 = 3 - 0 = 3
left: preorder: p1 + 1(1), p1 + leftLen(0+3=3); inorder: i1, idx-1
right: preorder: p1 + leftLen + 1, p2; inorder: idx+1, i2

```js
var buildTree = function (preorder, inorder) {
  let map = {};
  inorder.forEach((node, idx) => (map[node] = idx));
  const helper = (p1, p2, i1, i2) => {
    if (p1 > p2 || i1 > i2) return null;
    let val = preorder[p1],
      idx = map[val],
      leftLen = idx - i1,
      root = new TreeNode(val);
    root.left = helper(p1 + 1, p1 + leftLen, i1, idx - 1);
    root.right = helper(p1 + leftLen + 1, p2, idx + 1, i2);
    return root;
  };
  return helper(0, preorder.length - 1, 0, inorder.length - 1);
};
```

OR, if we could manipulate preorder array directly:

```js
var buildTree = function (preorder, inorder) {
  let map = {};
  inorder.forEach((node, idx) => (map[node] = idx));
  const helper = (start, end) => {
    if (start > end) return null;
    let root = new TreeNode(preorder.shift());
    root.left = helper(start, map[root.val] - 1);
    root.right = helper(map[root.val] + 1, end);
    return root;
  };
  return helper(0, inorder.length - 1);
};
```

OR, use `idx` to track current root's position in preorder array.(**recommend**)

```js
var buildTree = function (preorder, inorder) {
  let map = {};
  inorder.forEach((node, idx) => (map[node] = idx));
  let idx = 0;
  const helper = (start, end) => {
    if (start > end) return null;
    let root = new TreeNode(preorder[idx++]);
    root.left = helper(start, map[root.val] - 1);
    root.right = helper(map[root.val] + 1, end);
    return root;
  };
  return helper(0, inorder.length - 1);
};
```

注意：上述解法均默认各节点的值唯一，如果不唯一需要改变 map 的记录方式 => map[node] = [], map[node].push(idx)。

[Postorder + Inorder](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

```js
var buildTree = function (inorder, postorder) {
  let map = {};
  inorder.forEach((node, idx) => (map[node] = idx));
  const helper = (p1, p2, i1, i2) => {
    if (p1 > p2 || i1 > i2) return null;
    let rootVal = postorder[p2],
      rootIdx = map[rootVal],
      len = rootIdx - i1,
      root = new TreeNode(rootVal);
    root.left = helper(p1, p1 + len - 1, i1, rootIdx - 1);
    root.right = helper(p1 + len, p2 - 1, rootIdx + 1, i2);
    return root;
  };
  return helper(0, postorder.length - 1, 0, inorder.length - 1);
};
```

OR, if we could manipulate postorder array: 注意递归顺序，需要先递归求出右子树。

```js
var buildTree = function (inorder, postorder) {
  let map = {};
  inorder.forEach((node, idx) => (map[node] = idx));
  const helper = (left, right) => {
    if (left > right) return null;
    let root = new TreeNode(postorder.pop());
    root.right = helper(map[root.val] + 1, right);
    root.left = helper(left, map[root.val] - 1);
    return root;
  };
  return helper(0, inorder.length - 1);
};
```

OR, use `idx` to track current root's position in postorder array. (**recommend**)

注意递归时应先递归右子树。

```js
var buildTree = function (inorder, postorder) {
  let map = {},
    idx = postorder.length - 1;
  inorder.forEach((node, idx) => (map[node] = idx));
  const helper = (left, right) => {
    if (left > right) return null;
    let root = new TreeNode(postorder[idx--]);
    root.right = helper(map[root.val] + 1, right);
    root.left = helper(left, map[root.val] - 1);
    return root;
  };
  return helper(0, inorder.length - 1);
};
```

[Preorder + Postorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

[idea](<https://leetcode.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/discuss/161268/C%2B%2BJavaPython-One-Pass-Real-O(N)>): take [1,2,3,null,4,5] as example, pre: [1,2,4,3,5], post: [4,2,5,3,1], res: [1,2,3,4,null,5]. 只要 pre 没有跟 post 产生交集，就一直递增 preIdx，并且填充节点，如果有交集则返回上层，并后移 postIdx。

recursion:

```js
var constructFromPrePost = function (pre, post) {
  let preIdx = 0,
    postIdx = 0;
  const helper = (pre, post) => {
    let root = new TreeNode(pre[preIdx++]);
    if (root.val !== post[postIdx]) {
      root.left = helper(pre, post);
    }
    if (root.val !== post[postIdx]) {
      root.right = helper(pre, post);
    }
    postIdx++;
    return root;
  };
  return helper(pre, post);
};
```

iteration:

```js
var constructFromPrePost = function (pre, post) {
  let queue = [new TreeNode(pre[0])];
  for (let i = 1, j = 0; i < pre.length; i++) {
    let node = new TreeNode(pre[i]);
    while (queue[queue.length - 1].val === post[j]) {
      queue.pop();
      j++;
    }
    if (queue[queue.length - 1].left === null) {
      queue[queue.length - 1].left = node;
    } else {
      queue[queue.length - 1].right = node;
    }
    queue.push(node);
  }
  return queue.shift();
};
```

### BST: Binary Search Tree

In-order traversal of a BST will give you all elements in order. When a question involves a BST, the interviewer is usually looking for a solution which runs faster than O(n).

#### [search](https://leetcode.com/problems/search-in-a-binary-search-tree/)

```js
var searchBST = function (root, val) {
  if (!root) return null;
  if (val < root.val) return searchBST(root.left, val);
  if (val > root.val) return searchBST(root.right, val);
  return root;
};
```

#### [insert](https://leetcode.com/problems/insert-into-a-binary-search-tree/)

It's guaranteed that val does not exist in the original BST.

```js
var insertIntoBST = function (root, val) {
  if (!root) return new TreeNode(val);
  if (val < root.val) root.left = insertIntoBST(root.left, val);
  else if (val > root.val) root.right = insertIntoBST(root.right, val);
  return root;
};
```

#### [delete](https://leetcode.com/problems/delete-node-in-a-bst/)

```js
var deleteNode = function (root, key) {
  if (!root) return null;
  if (key < root.val) root.left = deleteNode(root.left, key);
  else if (key > root.val) root.right = deleteNode(root.right, key);
  else {
    if (!root.left) return root.right; // return kept branches
    if (!root.right) return root.left; // return kept branches
    let node = root.right;
    while (node.left) node = node.left; // find min in root.right, which is the smallest element bigger than root
    node.left = root.left; // and remove root, replace it with this min node
    return root.right; // return kept branches
  }
};
```

#### [validation](https://leetcode.com/problems/validate-binary-search-tree/)

思路一：inorder 遍历，通过 prev 指针对比当前节点和前序节点值，并根据结果更新 res。=> 不会提前返回，即使找到不符合条件的节点也仍然会继续遍历整个树。

```js
var isValidBST = function (root) {
  let res = true,
    prev = null;
  function inorder(node) {
    if (!node) return;
    inorder(node.left);
    if (prev && prev.val >= node.val) res = false;
    prev = node;
    inorder(node.right);
  }
  inorder(root);
  return res;
};
```

思路二（改进）：约束当前子树不能超过的最小值和最大值 => 向左子树递归，则当前节点值为新的 max；向右子树递归，则当前节点值为新的 min。

```js
var isValidBST = function (root) {
  function helper(node, min, max) {
    if (!node) return true;
    if (node.val <= min.val || node.val >= max.val) return false;
    return helper(node.left, min, node) && helper(node.right, node, max);
  }
  return helper(root, Infinity, -Infinity);
};
```

### [Trie](https://leetcode.com/explore/learn/card/trie/)

Trie, also called prefix tree, is a special form of a Nary tree. Typically, a trie is used to store strings.

Applications: autocomplete, spell checker, etc.

Be familiar with implementing, from scratch, a Trie class and its add, remove and search methods.

How to represent a Trie?

- use array to store child nodes: fast to visit a child node; might be some waste of space.

- use hashmap to store child nodes: might be a little slower than using an array; saves some space; more flexible.

#### [implementation](https://leetcode.com/problems/implement-trie-prefix-tree/)

TrieNode 由两部分构成：children 节点集合 & 当前是否为一个有效的单词。这样做的好处是便于给节点增加其他属性，比如 Map Sum Pairs 中的 val。

```js
class TrieNode {
  constructor() {
    this.next = {};
    this.isEnd = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }

  /**
   * Inserts a word into the trie.
   * @param {string} word
   * @return {void}
   */
  insert(word) {
    let node = this.root;
    for (let ch of word) {
      node = node.next[ch] = node.next[ch] || new TrieNode();
    }
    node.isEnd = true;
  }

  traverse(word) {
    let node = this.root;
    for (let ch of word) {
      node = node.next[ch];
      if (!node) return null;
    }
    return node;
  }

  /**
   * Returns if the word is in the trie.
   * @param {string} word
   * @return {boolean}
   */
  search(word) {
    let node = this.traverse(word);
    return node !== null && node.isEnd === true;
  }

  /**
   * Returns if there is any word in the trie that starts with the given prefix.
   * @param {string} prefix
   * @return {boolean}
   */
  startsWith(prefix) {
    return this.traverse(prefix) !== null;
  }
}
```

在跟其他题目联动时常用的模板：

```js
class TrieNode {
  constructor() {
    this.next = {};
    this.word = null;
  }
}

function buildTrie(words) {
  let root = new TrieNode();
  for (let word of words) {
    let node = root;
    for (let ch of word) {
      node = node.next[ch] = node.next[ch] || new TrieNode();
    }
    node.word = word;
  }
  return root;
}
```

## Questions

递归的输入值 & 输出值需要明确。对于 BST，inorder 的递归、迭代写法需要很熟！

### Traversal

#### [Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

#### [N-ary Tree Preorder Traversal](https://leetcode.com/problems/n-ary-tree-preorder-traversal/)

#### [Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

#### [Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

#### [N-ary Tree Postorder Traversal](https://leetcode.com/problems/n-ary-tree-postorder-traversal/)

#### [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

#### [N-ary Tree Level Order Traversal](https://leetcode.com/problems/n-ary-tree-level-order-traversal/)

```js
// recursion
if (root.children && root.children.length) {
  root.children.forEach((child) => helper(child, depth + 1, res));
}
// iteration
if (node.children && node.children.length) {
  node.children.forEach((child) => queue.push(child));
}
```

#### [Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/)

return the bottom-up level order traversal of its nodes' values.

#### [Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)

recursion:

```js
var zigzagLevelOrder = function (root) {
  let res = [];
  helper(root, 0, res);
  return res;
};

function helper(root, depth, res) {
  if (!root) return;
  if (depth === res.length) res.push([]);

  if (depth % 2 === 0) {
    res[depth].push(root.val);
  } else {
    res[depth].unshift(root.val);
  }

  helper(root.left, depth + 1, res);
  helper(root.right, depth + 1, res);
}
```

iteration:

```js
var zigzagLevelOrder = function (root) {
  if (!root) return [];

  let queue = [root],
    res = [],
    ltr = true;
  while (queue.length) {
    let curr = [],
      size = queue.length;
    for (let i = 0; i < size; i++) {
      let node = queue.shift();
      curr.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    res.push(ltr ? curr : curr.reverse());
    ltr = !ltr;
  }

  return res;
};
```

#### [Check Completeness of a Binary Tree](https://leetcode.com/problems/check-completeness-of-a-binary-tree/)

层次遍历，不能出现 node => null => node 的序列。

```js
var isCompleteTree = function (root) {
  let q = [root];
  while (q.length > 0) {
    let node = q.shift();
    while (!node && q.length) {
      if (q.shift()) {
        return false;
      }
    }
    if (node) {
      q.push(node.left);
      q.push(node.right);
    }
  }
  return true;
};
```

#### [Maximum Width of Binary Tree](https://leetcode.com/problems/maximum-width-of-binary-tree/)

一开始想通过把所有子节点都放进下一层来算最大，但 too naive，[1,3,2,5] 这种情况会不满足。由这个 case 想到，需要记录每一层首个和末个节点的位置。

为什么不用 2\*pos & 2\*pos+1 这种正向 idx 来记录？会溢出。

```js
var widthOfBinaryTree = function (root) {
  let q = [[root, 0]],
    max = 0;
  while (q.length) {
    let size = q.length,
      l = q[0][1],
      r = q[size - 1][1];
    max = Math.max(max, r - l + 1);
    while (size--) {
      let [node, pos] = q.shift();
      if (node.left) q.push([node.left, 2 * pos - 1]);
      if (node.right) q.push([node.right, 2 * pos]);
    }
  }
  return max;
};
```

#### [Boundary of Binary Tree](https://github.com/grandyang/leetcode/issues/545)

```js
function boundaryOfBinaryTree(root) {
  if (!root) return [];
  let res = [root.val];
  leftBoundary(root.left, res);
  leaves(root.left, res);
  leaves(root.right, res);
  rightBoundary(root.right, res);
  return res;
}

function leaves(root, res) {
  if (!root) return;
  if (!root.left && !root.right) {
    res.push(root.val);
    return;
  }
  leaves(root.left, res);
  leaves(root.right, res);
}

function leftBoundary(root, res) {
  if (!root || (!root.left && !root.right)) return;
  res.push(root.val);
  if (!root.left) {
    leftBoundary(root.right, res);
  } else {
    leftBoundary(root.left, res);
  }
}

function rightBoundary(root, res) {
  if (!root || (!root.left && !root.right)) return;
  if (!root.right) {
    rightBoundary(root.left, res);
  } else {
    rightBoundary(root.right, res);
  }
  res.push(root.val);
}
```

### Recursion

可能的技巧：通过特殊返回值表示结果；若递归过程是求最值过程，最值需要在计算的同时被不断比较更新。

TODO：分类如 Subtree of Another Tree。以及这样写的时间复杂度。

#### [Cousins in Binary Tree](https://leetcode.com/problems/cousins-in-binary-tree/)

```js
var isCousins = function (root, x, y) {
  let [depthX, pX] = find(root, x, 0, null);
  let [depthY, pY] = find(root, y, 0, null);
  return depthX === depthY && pX !== pY;
};

function find(root, x, depth, parent) {
  if (!root) return null;
  if (root.val === x) return [depth, parent];
  return (
    find(root.left, x, depth + 1, root) || find(root.right, x, depth + 1, root)
  );
}
```

#### [Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)

```js
var maxDepth = function (root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
};
```

#### [Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/)

```js
var minDepth = function (root) {
  if (!root) return 0;
  if (!root.left) return minDepth(root.right) + 1;
  if (!root.right) return minDepth(root.left) + 1;
  return Math.min(minDepth(root.left), minDepth(root.right)) + 1;
};
```

#### [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)

需要递归计算出左右子树的深度，并返回给上层，同时上层需要递归结果来比较得到当前节点所对应的子树是否满足 balanced 条件。一种做法是通过“全局变量”，helper 在递归时不断修改这个变量；另一种做法是，通过特殊返回值规避“全局变量”。

由于只计算左右子树深度，递归结果只可能 >= 0，那么在递归的过程中一旦发现不符合条件的情况，可以通过返回特殊值 -1 表示当前结果不符合 balanced 条件，让递归提前结束。

```js
var isBalanced = function (root) {
  return helper(root) !== -1;
};

function helper(root) {
  if (!root) return 0;
  let left = helper(root.left),
    right = helper(root.right);
  if (left === -1 || right === -1 || Math.abs(left - right) > 1) return -1;

  return Math.max(left, right) + 1;
}
```

#### [Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)

recursion

```js
var isSymmetric = function (root) {
  return !root || check(root.left, root.right);
};

function check(left, right) {
  if (!left || !right) return left === right;
  if (left.val !== right.val) return false;
  return check(left.left, right.right) && check(left.right, right.left);
}
```

iteration

```js
var isSymmetric = function (root) {
  if (!root) return true;
  let queue = [root.left, root.right];
  while (queue.length) {
    let l = queue.shift(),
      r = queue.shift();
    if (!l && !r) continue;
    if (!l || !r || l.val !== r.val) return false;

    queue.push(l.left);
    queue.push(r.right);
    queue.push(l.right);
    queue.push(r.left);
  }

  return true;
};
```

#### [Binary Tree Longest Consecutive Sequence II](https://github.com/grandyang/leetcode/issues/549)

The path can be either increasing or decreasing, and the path can be in the child-Parent-child order.

```js
function longestConsecutive(root) {
  if (!root) return 0;
  let res = dfs(root, 1) + dfs(root, -1) + 1;
  return Math.max(
    res,
    longestConsecutive(root.left),
    longestConsecutive(root.right)
  );
}

const dfs = (root, diff) => {
  if (!root) return 0;
  let left = 0,
    right = 0;
  if (root.left && root.left.val === root.val + diff) {
    left = dfs(root.left, diff) + 1;
  }
  if (root.right && root.right.val === root.val + diff) {
    right = dfs(root.right, diff) + 1;
  }
  return Math.max(left, right);
};
```

#### [Binary Tree Longest Consecutive Sequence](https://github.com/grandyang/leetcode/issues/298)

比较容易想到的递归方式同上题，但会造成重复的递归计算。可以在递归时传递 parent 节点，帮助判断当前路径是否要延续。

```js
function longestConsecutive(root) {
  const dfs = (node, parent, res) => {
    if (!node) return res;
    res = parent && node.val === parent.val + 1 ? res + 1 : 1;
    return Math.max(res, dfs(node.left, node, res), dfs(node.right, node, res));
  };
  return dfs(root, null, 0);
}
```

#### [Path Sum](https://leetcode.com/problems/path-sum/)

return true if the tree has a root-to-leaf path such that adding up all the values along the path equals targetSum.

root-to-leaf path = sum => sum - root.val. corner case: empty tree, sum = 0, should return false.

```js
var hasPathSum = function (root, sum) {
  if (!root) return false;
  if (!root.left && !root.right) return sum == root.val;
  return (
    hasPathSum(root.left, sum - root.val) ||
    hasPathSum(root.right, sum - root.val)
  );
};
```

#### [Path Sum II](https://leetcode.com/problems/path-sum-ii/)

return all root-to-leaf paths where each path's sum equals targetSum.

在递归之路上，除当前 sum 外，还需要额外记录的信息：所有路径合法的集合 paths，以及当前路径 path。当最终抵达叶子结点时，判断当前路径是否为合法路径，若是则将当前路径 path 加入合法路径集合 paths。

```js
var pathSum = function (root, sum) {
  let paths = [];
  findPath(root, sum, paths, []);
  return paths;
};

function findPath(root, sum, paths, path) {
  if (!root) return;

  path.push(root.val);
  if (!root.left && !root.right && root.val === sum) {
    paths.push(path.concat());
  }
  findPath(root.left, sum - root.val, paths, path);
  findPath(root.right, sum - root.val, paths, path);
  path.pop();
}
```

#### [Path Sum III](https://leetcode.com/problems/path-sum-iii/)

Find the number of paths that sum to a given value. The path does not need to start or end at the root or a leaf, but it must travel only from parent nodes to child nodes.

需要一个辅助函数来帮助计算，将当前节点作为起始节点时满足条件的路经数。

```js
var pathSum = function (root, sum) {
  if (!root) return 0;
  return (
    pathSumFrom(root, sum) + pathSum(root.left, sum) + pathSum(root.right, sum)
  );
};

var pathSumFrom = function (root, sum) {
  if (!root) return 0;
  return (
    (root.val === sum ? 1 : 0) +
    pathSumFrom(root.left, sum - root.val) +
    pathSumFrom(root.right, sum - root.val)
  );
};
```

#### [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)

The diameter of a binary tree is the length of the longest path between any two nodes in a tree.

路径可能穿过根节点，可以分成两部分：左子树中的最长路径 + 右子树中的最长路径。以某节点为起始节点的最长路径可以通过递归得到。在递归过程中，更新 max diameter 的值：Math.max(l + r, max)。

```js
var diameterOfBinaryTree = function (root) {
  let max = 0;
  function helper(node) {
    if (!node) return 0;
    let l = helper(node.left),
      r = helper(node.right);
    max = Math.max(l + r, max);
    return 1 + Math.max(l, r);
  }
  helper(root);
  return max;
};
```

#### [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

return the maximum path sum of any path. Note that the path does not need to pass through the root.

```js
const MIN = -2147483648; // strange min value LT uses
var maxPathSum = function (root) {
  if (!root) return MIN;
  let max = MIN;
  function maxPathDown(node) {
    if (!node) return 0;
    let left = Math.max(0, maxPathDown(node.left)),
      right = Math.max(0, maxPathDown(node.right));
    max = Math.max(max, left + right + node.val);
    return Math.max(left, right) + node.val;
  }
  maxPathDown(root);
  return max;
};
```

复盘思考过程：想到结合之前做的递归题目（如 Path Sum III），先简化问题为：求单边树的 maxPathSum，然后分别处理，于是想当然地写出了：

```js
function maxPathSumFrom(root) {
  if (!root) return 0;
  let left = Math.max(0, maxPathSumFrom(root.left)), // => wrong recursion abstraction
    right = Math.max(0, maxPathSumFrom(root.right));
  return Math.max(left, right) + root.val;
}

var maxPathSum = function (root) {
  if (!root) return -2147483648;
  return Math.max(
    maxPathSumFrom(root),
    maxPathSum(root.left),
    maxPathSum(root.right)
  );
};
```

但输入 [-10,9,20,null,null,15,7]，expected = 42，output = 35。

原因是递归的返回其实只能代表一种情况：root 的值被用于当前路径中，且为路径末端。而 root 作为路径中的节点这种情况则被忽略了，即 left + right + val。这种情况其实可以在递归得到 root as end，go left / right 的过程中处理。

#### [Delete Nodes And Return Forest](https://leetcode.com/problems/delete-nodes-and-return-forest/)

应该判断当前节点 node 的子节点 left & right 的值在 to_delete 中，还是判断当前节点的值在其中？=> 当前节点。如果判断子节点，那么就需要处理将当前节点与子节点的节点进行连接的情况，复杂化解决方案。=> 如果判断当前节点，那么必然应先递归处理好子节点。

如果当前节点的值符合条件，我们应该做什么处理？=> 对于当前节点，需要设置为 null。对于该节点的子节点呢？

```js
var delNodes = function (root, to_delete) {
  let set = new Set(to_delete),
    res = [];
  function helper(node) {
    if (!node) return null;

    node.left = helper(node.left);
    node.right = helper(node.right);

    if (set.has(node.val)) {
      if (node.left) res.push(node.left);
      if (node.right) res.push(node.right);
      node = null;
    }
    return node;
  }

  root = helper(root);
  if (root) res.push(root);
  return res;
};
```

#### [Sum of Left Leaves](https://leetcode.com/problems/sum-of-left-leaves/)

不记录 parent 节点，如何找出左叶子？

```js
var sumOfLeftLeaves = function (root) {
  if (!root) return 0;
  if (root.left && !root.left.left && !root.left.right)
    return root.left.val + sumOfLeftLeaves(root.right);
  return sumOfLeftLeaves(root.left) + sumOfLeftLeaves(root.right);
};
```

#### [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/)

```js
var invertTree = function (root) {
  if (!root) return null;

  let left = invertTree(root.left);
  let right = invertTree(root.right);

  root.left = right;
  root.right = left;

  return root;
};
```

#### [Binary Tree Pruning](https://leetcode.com/problems/binary-tree-pruning/)

Return the same tree where every subtree (of the given tree) not containing a 1 has been removed.

```js
var pruneTree = function (root) {
  if (!root) return null;
  root.left = pruneTree(root.left);
  root.right = pruneTree(root.right);
  // if (!root.left && !root.right && root.val === 0) return null;
  // return root;
  return root.left || root.right || root.val ? root : null;
};
```

#### [Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/)

```js
var isSubtree = function (s, t) {
  if (!s) return false;
  return isSame(s, t) || isSubtree(s.left, t) || isSubtree(s.right, t);
};

function isSame(s, t) {
  if (!s && !t) return true;
  if (!s || !t) return false;
  return s.val === t.val && isSame(s.left, t.left) && isSame(s.right, t.right);
}
```

If assume m is the number of nodes in the 1st tree and n is the number of nodes in the 2nd tree, then:

Time complexity: O(m*n), worst case: for each node in the 1st tree, we need to check if isSame(Node s, Node t). Total m nodes, isSame(...) takes O(n) worst case

Space complexity: Basically you would try to compare the the target tree with each subtree in the source tree - the root for each subtree will be picked in a preorder manner (total of m root nodes). So the worst possibility is that you travel each subtree and find out the last node is different (total of n nodes compared). If the tree is balanced, your recursion would be the depth of each subtree, thus Height == logm. If the tree is skewed, then height would be m. O(height of 1str tree)(Or you can say: O(m) for worst case, O(logm) for average case)


#### [Merge Two Binary Trees](https://leetcode.com/problems/merge-two-binary-trees/)

```js
var mergeTrees = function (t1, t2) {
  if (!t1 && !t2) return null;
  if (!t1) return t2;
  if (!t2) return t1;

  return new TreeNode(
    t1.val + t2.val,
    mergeTrees(t1.left, t2.left),
    mergeTrees(t1.right, t2.right)
  );
};
```

#### [Sum Root to Leaf Numbers](https://leetcode.com/problems/sum-root-to-leaf-numbers/)

到叶子结点时累加该条路径对应的数值，中间节点则继续向下递归。能否直接通过递归 fn(root) 得到结果？不能，原因是每条路径的 sum 值需要到叶子节点才能知道，通过返回值记录路径比较麻烦。

```js
var sumNumbers = function (root) {
  if (!root) return 0;
  let res = 0;
  function dfs(root, sum) {
    if (!root.left && !root.right) res += sum * 10 + root.val;
    if (root.left) dfs(root.left, sum * 10 + root.val);
    if (root.right) dfs(root.right, sum * 10 + root.val);
  }
  dfs(root, 0);
  return res;
};
```

#### [Linked List in Binary Tree](https://leetcode.com/problems/linked-list-in-binary-tree/)

Time O(N \* min(L,H))
Space O(H)
where N = tree size, H = tree height, L = list length.

```js
function dfs(head, root) {
  if (head === null) return true;
  if (root === null) return false;
  return (
    head.val === root.val &&
    (dfs(head.next, root.left) || dfs(head.next, root.right))
  );
}

var isSubPath = function (head, root) {
  if (head === null) return true;
  if (root === null) return false;
  return (
    dfs(head, root) || isSubPath(head, root.left) || isSubPath(head, root.right)
  );
};
```

#### [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

通常情况：p q 一个在左子树，一个在右子树，那么从当前节点向左子树和右子树递归，当两边递归的返回值都存在时，说明当前节点就是祖先节点。

通常情况：p q 都在左子树或者都在右子树。如果目标节点在某一侧，另一侧应返回 null，加一个判断来处理这种情况。

特殊情况，p 或者 q 其中一个就是祖先节点。因为 p q 一定存在于树中，那么如果当前节点等于 p 或者 q 时，意味着祖先节点已经找到，不必再查找直接返回该节点即可。

```js
var lowestCommonAncestor = function (root, p, q) {
  if (!root || root === p || root === q) return root;

  let l = lowestCommonAncestor(root.left, p, q),
    r = lowestCommonAncestor(root.right, p, q);

  if (l && r) return root;
  return l ? l : r;
};
```

TODO: LCA of BT 后续系列

https://leetcode.com/problems/smallest-common-region/
https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-ii/
https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-iii/
https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-iv/

#### [House Robber III](https://leetcode.com/problems/house-robber-iii/)

如果选择当前节点，那么 left & right 必须被跳过，递归 left & right 的下层节点；如果不选择当前节点，则递归左右节点。

暴力递归：

```js
var rob = function (root) {
  if (!root) return 0;

  let sum = 0;
  if (root.left) sum += rob(root.left.left) + rob(root.left.right);
  if (root.right) sum += rob(root.right.left) + rob(root.right.right);

  return Math.max(root.val + sum, rob(root.left) + rob(root.right));
};
```

记录两种情况下各自的最大值，消除重复计算的递归：

```js
var rob = function (root) {
  function robAt(node) {
    if (!node) return [0, 0];
    let l = robAt(node.left),
      r = robAt(node.right);
    return [
      node.val + l[1] + r[1], // use current node and nodes below left & right
      Math.max(...l) + Math.max(...r), // skip current node, use nodes below(might include left & right)
    ];
  }

  return Math.max(...robAt(root));
};
```

### BST

#### [Maximum Sum BST in Binary Tree](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree/)

与 validate BST 不同，需要从下向上求解。null 节点返回值应如何处理可以让上一层的叶子节点满足 `left.isValid && right.isValid && left.max < node.val && right.min > node.val`？

```js
class NodeInfo {
  constructor(isValid, sum, min, max) {
    this.isValid = isValid;
    this.sum = sum;
    this.min = min;
    this.max = max;
  }
}

var maxSumBST = function (root) {
  let max = 0;
  const helper = (node) => {
    if (!node) return new NodeInfo(true, 0, Infinity, -Infinity);
    let left = helper(node.left),
      right = helper(node.right),
      isValid =
        left.isValid &&
        right.isValid &&
        left.max < node.val &&
        right.min > node.val,
      sum = left.sum + right.sum + node.val;
    if (isValid) {
      max = Math.max(max, sum);
    }
    return new NodeInfo(
      isValid,
      sum,
      Math.min(left.min, node.val),
      Math.max(right.max, node.val)
    );
  };
  helper(root);
  return max;
};
```

#### [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

```js
var lowestCommonAncestor = function (root, p, q) {
  while (
    (root.val > p.val && root.val > q.val) ||
    (root.val < p.val && root.val < q.val)
  ) {
    root = root.val > p.val ? root.left : root.right;
  }
  return root;
};
```

#### [Trim a Binary Search Tree](https://leetcode.com/problems/trim-a-binary-search-tree/)

```js
var trimBST = function (root, low, high) {
  if (!root) return null;

  if (root.val < low) return trimBST(root.right, low, high);
  if (root.val > high) return trimBST(root.left, low, high);

  root.left = trimBST(root.left, low, high);
  root.right = trimBST(root.right, low, high);

  return root;
};
```

#### [Range Sum of BST](https://leetcode.com/problems/range-sum-of-bst/)

```js
var rangeSumBST = function (root, low, high) {
  if (!root) return 0;
  if (root.val < low) {
    return rangeSumBST(root.right, low, high);
  }
  if (root.val > high) {
    return rangeSumBST(root.left, low, high);
  }
  return (
    root.val +
    rangeSumBST(root.left, low, high) +
    rangeSumBST(root.right, low, high)
  );
};
```

#### [Closest Binary Search Tree Value](https://github.com/grandyang/leetcode/issues/270)

```js
var closestValue = function (root, target) {
  let res = root.val;
  if (root.val > target && root.left) {
    let l = closestValue(root.left, target);
    if (Math.abs(l - target) < root.val - target) res = l;
  } else if (root.val < target && root.right) {
    let r = closestValue(root.right, target);
    if (Math.abs(r - target) < target - root.val) res = r;
  }
  return res;
};
```

#### [Construct Binary Search Tree from Preorder Traversal](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/)

All the values of preorder are unique. 如果有重复则无法唯一确定一棵树。比如：pre-order [2,2,1,2,4], inorder [1,2,2,2,4] 这种情况。

根据 BST 特性切分数组。

```js
var bstFromPreorder = function (preorder) {
  const helper = (start, end) => {
    if (start > end) return null;
    let root = new TreeNode(preorder[start]),
      idx = start + 1;
    while (preorder[idx] < root.val) idx++;
    root.left = helper(start + 1, idx - 1);
    root.right = helper(idx, end);
    return root;
  };
  return helper(0, preorder.length - 1);
};
```

#### [Minimum Absolute Difference in BST](https://leetcode.com/problems/minimum-absolute-difference-in-bst/)

找到任意两节点间最小差 => 需要遍历所有节点 => inorder

任意两节点 => BST 特性 => 只需要考虑 inorder 过程中的相邻节点

中序遍历如何记住前一节点？=> prev pointer

```js
var getMinimumDifference = function (root) {
  let prev = null,
    min = Infinity;
  function inorder(node) {
    if (!node) return;
    inorder(node.left);
    if (prev) {
      min = Math.min(min, Math.abs(prev.val - node.val));
    }
    prev = node;
    inorder(node.right);
  }
  inorder(root);
  return min;
};
```

#### [Binary Search Tree to Greater Sum Tree](https://leetcode.com/problems/binary-search-tree-to-greater-sum-tree/)

inorder ldr => rdl

```js
var bstToGst = function (root) {
  let prev = null;
  const rdl = (root) => {
    if (!root) return;
    rdl(root.right);
    if (prev) {
      root.val += prev.val;
    }
    prev = root;
    rdl(root.left);
  };
  rdl(root);
  return root;
};
```

or, iteration:

```js
var bstToGst = function (root) {
  let sum = 0,
    stack = [],
    node = root;
  while (node || stack.length) {
    while (node) {
      stack.push(node);
      node = node.right;
    }
    node = stack.pop();
    node.val += sum;
    sum = node.val;
    node = node.left;
  }
  return root;
};
```

#### [Increasing Order Search Tree](https://leetcode.com/problems/increasing-order-search-tree/)

思路一：递归返回当前子树被更新后的根节点 => 可能存在的问题：该子树有右子树，上层节点应被重新链接至右子树的最右端叶子结点上。

```js
var increasingBST = function (root) {
  if (!root) return null;
  let left = increasingBST(root.left);
  if (left) {
    let node = left;
    while (node.right) node = node.right;
    node.right = root;
    root.left = null;
  }
  root.right = increasingBST(root.right);
  return left ? left : root;
};
```

思路二：inorder 的过程中利用 prev 指针，更新关系。

```js
var increasingBST = function (root) {
  let node = root,
    prev = null;
  while (node.left) node = node.left;
  function increase(root) {
    if (!root) return;
    increase(root.left);
    if (prev) prev.right = root;
    root.left = null;
    prev = root;
    increase(root.right);
  }
  increase(root);
  return node;
};
```

#### [Find Mode in Binary Search Tree](https://leetcode.com/problems/find-mode-in-binary-search-tree/)

**caution!**

要找到树中所有最频繁出现的 val => 需要遍历整个树 => in-order。

```js
var findMode = function (root) {
  let currVal = null,
    currCount = 0,
    maxCount = 0,
    modeCount = 0;
  let modes = [];

  function handleValue(val) {
    if (val !== currVal) {
      currVal = val;
      currCount = 0;
    }
    currCount++;
    if (currCount > maxCount) {
      maxCount = currCount;
      modeCount = 1;
      modes[0] = val;
    } else if (currCount === maxCount) {
      modes[modeCount] = val;
      modeCount++;
    }
  }

  function inorder(node) {
    if (!node) return;
    inorder(node.left);
    handleValue(node.val);
    inorder(node.right);
  }

  inorder(root);

  return modes.slice(0, modeCount);
};
```

follow-up：如何不使用额外的空间（递归调用栈除外）？

遍历两遍，第一遍找到 maxCount，第二遍只录入 currCount = maxCount 的 val。

```js
var findMode = function (root) {
  let currVal = null,
    currCount = 0,
    maxCount = 0,
    modeCount = 0;
  let modes = null;

  function handleValue(val) {
    if (val !== currVal) {
      currVal = val;
      currCount = 0;
    }
    currCount++;
    if (currCount > maxCount) {
      maxCount = currCount;
      modeCount = 1;
    } else if (currCount === maxCount) {
      if (modes !== null) {
        modes[modeCount] = currVal;
      }
      modeCount++;
    }
  }

  function inorder(node) {
    if (!node) return;
    inorder(node.left);
    handleValue(node.val);
    inorder(node.right);
  }

  inorder(root);
  modes = [];
  modeCount = 0;
  currCount = 0;
  inorder(root);

  return modes;
};
```

follow-up：如何不使用额外的空间（包含递归调用栈）？

Morris traversal

```js
var findMode = function (root) {
  let currVal = null,
    currCount = 0,
    maxCount = 0,
    modeCount = 0;
  let modes = null;

  function handleValue(val) {
    if (val !== currVal) {
      currVal = val;
      currCount = 0;
    }
    currCount++;
    if (currCount > maxCount) {
      maxCount = currCount;
      modeCount = 1;
    } else if (currCount === maxCount) {
      if (modes !== null) {
        modes[modeCount] = currVal;
      }
      modeCount++;
    }
  }

  function inorder(node) {
    let curr = node,
      prev = null;
    while (curr !== null) {
      if (curr.left === null) {
        handleValue(curr.val);
        curr = curr.right;
      } else {
        prev = curr.left;
        while (prev.right !== null && prev.right !== curr) prev = prev.right;
        if (prev.right === null) {
          prev.right = curr;
          curr = curr.left;
        }
        if (prev.right === curr) {
          prev.right = null;
          handleValue(curr.val);
          curr = curr.right;
        }
      }
    }
  }

  inorder(root);
  modes = [];
  modeCount = 0;
  currCount = 0;
  inorder(root);

  return modes;
};
```

#### [Recover Binary Search Tree](https://leetcode.com/problems/recover-binary-search-tree/)

有两个节点进行了交换 => 从中序遍历的结果可以看出是哪两个节点 => 前一位的值大于后一位 => 需要 prev 指针标记前序节点

交换可能存在的情况 => 无非是邻近交换，或者间隔交换，即：[1,2,3,4] => [2,1,3,4] 或者 [4,2,3,1]

=> 还需要两个指针 first & second 标记需要交换的两个节点

```js
var recoverTree = function (root) {
  let prev = null,
    first = null,
    second = null;
  function inorder(node) {
    if (!node) return;
    inorder(node.left);
    if (prev && prev.val > node.val) {
      if (!first) first = prev;
      second = node;
    }
    prev = node;
    inorder(node.right);
  }

  inorder(root);
  let temp = first.val;
  first.val = second.val;
  second.val = temp;
};
```

#### [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

inorder by iteration => iterate to kth

```js
var kthSmallest = function (root, k) {
  let stack = [];

  while (true) {
    while (root !== null) {
      stack.push(root);
      root = root.left;
    }
    root = stack.pop();
    k--;
    if (!k) return root.val;
    root = root.right;
  }
};
```

#### [Inorder Successor in BST](https://github.com/grandyang/leetcode/issues/285)

1. use inorder + pre pointer

```js
var inorderSuccessor = function (root, p) {
  let pre = null,
    suc = null;
  const inorder = (node) => {
    if (!node) return null;
    inorder(node.left);
    if (pre === p) suc = node;
    pre = node;
    inorder(node.right);
  };
  inorder(root);
  return suc;
};
```

2. 利用 BST 特性：若 root.val <= p.val，则 successor 一定在右子树中，向右子树递归查找；若 root.val > p.val，则 successor 可能存在于左子树中，先向左子树递归，若左子树没找到，则说明当前节点就是 successor，若找到了则返回。

```js
var inorderSuccessor = function (root, p) {
  if (!root) return null;
  if (root.val <= p.val) {
    return inorderSuccessor(root.right, p);
  } else {
    let left = inorderSuccessor(root.left, p);
    return left ? left : root;
  }
};
```

#### [Inorder Successor in BST II](https://github.com/grandyang/leetcode/issues/510)

nodes have parent pointer, given a node in tree, find successor without visiting node.val:

```js
var inorderSuccessor = function (node) {
  if (!node) return null;
  if (node.right) {
    node = node.right;
    while (node && node.left) node = node.left;
    return node;
  }
  while (node) {
    if (!node.parent) return null;
    if (node === node.parent.left) return node.parent;
    node = node.parent; // is parent's right child, look up to parent's parent.
  }
  return node;
};
```

#### [Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)

二分

```js
var sortedArrayToBST = function (nums) {
  if (!nums) return;

  function helper(arr, lo, hi) {
    if (lo > hi) return null;
    let mid = Math.floor((lo + hi) / 2);
    let node = new TreeNode(arr[mid], null, null);
    node.left = helper(arr, lo, mid - 1);
    node.right = helper(arr, mid + 1, hi);
    return node;
  }

  return helper(nums, 0, nums.length - 1);
};
```

#### [Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)

快慢指针

```js
var sortedListToBST = function (head) {
  if (!head) return null;
  if (!head.next) return new TreeNode(head.val);

  let slow = head,
    fast = head,
    pre = null;
  while (fast && fast.next) {
    fast = fast.next.next;
    pre = slow;
    slow = slow.next;
  }

  pre.next = null;
  let node = new TreeNode(slow.val);
  node.left = sortedListToBST(head);
  node.right = sortedListToBST(slow.next);
  return node;
};
```

#### [Balance a Binary Search Tree](https://leetcode.com/problems/balance-a-binary-search-tree/)

convert to sorted array, and re-build. inorder + Convert Sorted Array to Binary Search Tree.


#### [Largest BST Subtree](https://github.com/grandyang/leetcode/issues/333)

```js
function largestBSTSubtree(root) {
  if (!root) return 0;
  if (isValid(root, -Infinity, Infinity)) return count(root);
  return Math.max(largestBSTSubtree(root.left), largestBSTSubtree(root.right));
}

function isValid(root, min, max) {
  if (!root) return true;
  if (root.val < min || root.val > max) return false;
  return isValid(root.left, min, root.val) && isValid(root.right, root.val, max);
}

function count(root) {
  if (!root) return 0;
  return count(root.left) + count(root.right) + 1;
}
```

optimize to O(n):

```js
function largestBSTSubtree(root) {
  let res = helper(root);
  return res.count;
}

function helper(root) {
  if (!root) return new Meta(0, Infinity, -Infinity);
  let left = helper(root.left), right = helper(root.right);
  if (root.val > left.max && root.val < right.min) {
    return new Meta(left.count + right.count + 1, left.min, right.max);
  } else {
    return new Meta(Math.max(left.count, right.count), -Infinity, Infinity);
  }
}

class Meta {
  constructor(count, min, max) {
    this.count = count;
    this.min = min;
    this.max = max;
  }
}
```

### Trie

#### [Map Sum Pairs](https://leetcode.com/problems/map-sum-pairs/)

稍微修改一下 TrieNode 的结构。

```js
class TrieNode {
  constructor() {
    this.next = {};
    this.isEnd = false;
    this.val = 0;
  }
}

class MapSum {
  constructor() {
    this.root = new TrieNode();
  }

  /**
   * @param {string} key
   * @param {number} val
   * @return {void}
   */
  insert(key, val) {
    let node = this.root;
    for (let ch of key) {
      node = node.next[ch] = node.next[ch] || new TrieNode();
    }
    node.isEnd = true;
    node.val = val;
  }

  traverse(key) {
    let node = this.root;
    for (let ch of key) {
      node = node.next[ch];
      if (!node) return null;
    }
    return node;
  }

  sumFrom(node) {
    return (
      node.val +
      Object.keys(node.next).reduce(
        (sum, key) => sum + this.sumFrom(node.next[key]),
        0
      )
    );
  }

  /**
   * @param {string} prefix
   * @return {number}
   */
  sum(prefix) {
    let node = this.traverse(prefix);
    if (node === null) return 0;
    return this.sumFrom(node);
  }
}
```

#### [Replace Words](https://leetcode.com/problems/replace-words/)

```js
class TrieNode {
  constructor() {
    this.next = {};
    this.isEnd = false;
  }
}

var replaceWords = function (dictionary, sentence) {
  let root = new TrieNode();
  dictionary.forEach((word) => {
    let node = root;
    for (let ch of word) {
      node = node.next[ch] = node.next[ch] || new TrieNode();
    }
    node.isEnd = true;
  });

  return sentence
    .split(" ")
    .map((word) => {
      let node = root,
        res = "";
      for (let ch of word) {
        node = node.next[ch];
        res += ch;
        if (!node) return word;
        if (node.isEnd) break;
      }
      return res;
    })
    .join(" ");
};
```

#### [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/)

```js
class TireNode {
  constructor() {
    this.next = {};
    this.isEnd = false;
  }
}

class WordDictionary {
  constructor() {
    this.root = new TireNode();
  }

  addWord(word) {
    let node = this.root;
    for (let ch of word) {
      if (!node.next[ch]) node.next[ch] = new TireNode();
      node = node.next[ch];
    }
    node.isEnd = true;
  }

  search(word) {
    return bfs(this.root, word);
  }
}

function bfs(root, word) {
  let node = root;
  for (let i = 0; i < word.length; i++) {
    let ch = word[i];
    if (ch === ".") {
      return Object.keys(node.next).reduce(
        (res, curr) => res || bfs(node.next[curr], word.slice(i + 1)),
        false
      );
    } else {
      if (!node.next[ch]) {
        return false;
      }
      node = node.next[ch];
    }
  }
  return node.isEnd;
}
```

#### [Word Search II](https://leetcode.com/problems/word-search-ii/)

```js
function dfs(board, i, j, node, res) {
  let ch = board[i][j],
    placeholder = "*";
  if (ch === placeholder || !node.next[ch]) return;
  node = node.next[ch];
  if (node.word) {
    res.push(node.word);
    node.word = null;
  }
  board[i][j] = placeholder;
  if (i > 0) dfs(board, i - 1, j, node, res);
  if (i < board.length - 1) dfs(board, i + 1, j, node, res);
  if (j > 0) dfs(board, i, j - 1, node, res);
  if (j < board[0].length - 1) dfs(board, i, j + 1, node, res);
  board[i][j] = ch;
}

class TrieNode {
  constructor() {
    this.next = {};
    // this.isEnd => this.word since we need to put whole word in the res array
    this.word = null;
  }
}

function buildTrie(words) {
  let root = new TrieNode();
  for (let word of words) {
    let node = root;
    for (let ch of word) {
      node = node.next[ch] = node.next[ch] || new TrieNode();
    }
    node.word = word;
  }
  return root;
}

var findWords = function (board, words) {
  let res = [],
    dic = buildTrie(words);
  for (let i = 0; i < board.length; i++) {
    for (let j = 0; j < board[0].length; j++) {
      dfs(board, i, j, dic, res);
    }
  }
  return res;
};
```

### Others

#### [All Possible Full Binary Trees](https://leetcode.com/problems/all-possible-full-binary-trees/)

```js
var allPossibleFBT = function (N) {
  if (N % 2 === 0) return [];
  if (N === 1) return [new TreeNode(0)];
  let res = [];
  for (let i = 1; i <= N - 2; i += 2) {
    // N - 2: save one for right node, one for root node.
    let lefts = allPossibleFBT(i),
      rights = allPossibleFBT(N - i - 1); // total(N) - left(i) - root(1)
    for (let l of lefts) {
      for (let r of rights) {
        res.push(new TreeNode(0, l, r));
      }
    }
  }
  return res;
};
```

类似题目见 DP：Unique Binary Search Trees II

#### [Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)

given a perfect binary tree.

recursion

```js
var connect = function (root) {
  if (!root || !root.left) return root;
  root.left.next = root.right;
  root.right.next = root.next ? root.next.left : null;
  connect(root.left);
  connect(root.right);
  return root;
};
```

iteration: 利用 .next 层次遍历

```js
var connect = function (root) {
  if (!root) return null;
  let node = root,
    next = null;
  while (node && node.left) {
    next = node;
    while (next) {
      next.left.next = next.right;
      if (next.next) next.right.next = next.next.left;
      next = next.next;
    }
    node = node.left;
  }
  return root;
};
```

#### [Populating Next Right Pointers in Each Node II](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/)

Given a binary tree.

需要先链接右子树。

```js
var connect = function (root) {
  if (!root) return root;
  if (root.right) root.right.next = getChild(root.next);
  if (root.left) root.left.next = root.right ? root.right : getChild(root.next);
  connect(root.right);
  connect(root.left);
  return root;
};

const getChild = (node) => {
  if (!node) return null;
  if (node.left) return node.left;
  if (node.right) return node.right;
  return getChild(node.next);
};
```

#### [Find Leaves of Binary Tree](https://github.com/grandyang/leetcode/issues/366)

```js
var findLeaves = function (root) {
  let res = [];
  while (root) {
    let curr = [];
    root = cut(root, curr);
    res.push(curr);
  }
  return res;
};

function cut(root, curr) {
  if (!root) return null;
  if (!root.left && !root.right) {
    curr.push(root.val);
    return null;
  }
  root.left = cut(root.left, curr);
  root.right = cut(root.right, curr);
  return root;
}
```

#### [Distribute Coins in Binary Tree](https://leetcode.com/problems/distribute-coins-in-binary-tree/)

[return the balance of coins.](https://leetcode.com/problems/distribute-coins-in-binary-tree/discuss/221939/C%2B%2B-with-picture-post-order-traversal)

```js
var distributeCoins = function (root) {
  let moves = 0;
  const dfs = (root) => {
    if (!root) return 0;
    let left = dfs(root.left),
      right = dfs(root.right);
    moves += Math.abs(left) + Math.abs(right);
    return root.val + left + right - 1; // save 1 for current root
  };
  dfs(root);
  return moves;
};
```

#### [Flip Equivalent Binary Trees](https://leetcode.com/problems/flip-equivalent-binary-trees/)

```js
var flipEquiv = function (root1, root2) {
  if (!root1 || !root2) return root1 === root2;
  return (
    root1.val === root2.val &&
    ((flipEquiv(root1.left, root2.left) &&
      flipEquiv(root1.right, root2.right)) ||
      (flipEquiv(root1.left, root2.right) &&
        flipEquiv(root1.right, root2.left)))
  );
};
```

#### [Smallest Subtree with all the Deepest Nodes](https://leetcode.com/problems/smallest-subtree-with-all-the-deepest-nodes/)

```js
var subtreeWithAllDeepest = function (root) {
  const dfs = (root) => {
    if (!root) return [0, null];
    let [lMax, lNode] = dfs(root.left),
      [rMax, rNode] = dfs(root.right);
    return [
      Math.max(lMax, rMax) + 1,
      lMax === rMax ? root : lMax > rMax ? lNode : rNode,
    ];
  };
  return dfs(root)[1];
};
```

#### [Construct Maximum Binary Tree](https://leetcode.com/problems/maximum-binary-tree/)

```js
var constructMaximumBinaryTree = function (nums) {
  function findMax(arr, left, right) {
    var max = left;
    for (let i = left; i < right; i++) if (arr[max] < arr[i]) max = i;
    return max;
  }

  function construct(arr, left, right) {
    if (left === right) return null;
    var max = findMax(arr, left, right);
    var root = new TreeNode(arr[max]);
    root.left = construct(arr, left, max);
    root.right = construct(arr, max + 1, right);
    return root;
  }

  return construct(nums, 0, nums.length);
};
```

类似题目：Construct Binary Search Tree from Preorder Traversal

#### [Find Bottom Left Tree Value](https://leetcode.com/problems/find-bottom-left-tree-value/)

iteration：BFS root => root.right => root.left，最后一个被遍历到的就是最下层的最左节点。

```js
var findBottomLeftValue = function (root) {
  let queue = [root];
  while (queue.length) {
    root = queue.shift();
    if (root.right) queue.push(root.right);
    if (root.left) queue.push(root.left);
  }
  return root.val;
};
```

recursion: level-order traversal 当当前深度首次超过最大深度时，该节点就是该层的最左侧节点。

```js
var findBottomLeftValue = function (root) {
  let maxDepth = 0,
    res = null;
  const helper = (node, depth) => {
    if (node !== null) {
      if (depth > maxDepth) {
        maxDepth = depth;
        res = node;
      }
      helper(node.left, depth + 1);
      helper(node.right, depth + 1);
    }
  };
  helper(root, 1);
  return res.val;
};
```

#### [Kill Process](https://github.com/grandyang/leetcode/issues/582)

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

#### [Convert Binary Search Tree to Sorted Doubly Linked List](https://github.com/grandyang/leetcode/issues/426)

use prev pointer + inorder traversal

```js
function treeToDoublyList(root) {
  if (!root) return null;
  let head = null,
    prev = null;
  const inorder = (root) => {
    if (!root) return;
    inorder(root.left);
    if (!head) head = root;
    if (prev) {
      root.left = prev;
      prev.right = root;
    }
    prev = root;
    inorder(root.right);
  };
  inorder(root);
  head.left = prev;
  prev.right = head;
  return head;
}
```

#### [Most Frequent Subtree Sum](https://leetcode.com/problems/most-frequent-subtree-sum/)

```js
var findFrequentTreeSum = function (root) {
  let map = {},
    max = 0;
  const postorder = (root) => {
    if (!root) return 0;
    let curr = root.val + postorder(root.left) + postorder(root.right);
    map[curr] = map[curr] === undefined ? 1 : map[curr] + 1;
    max = Math.max(max, map[curr]);
    return curr;
  };
  postorder(root);
  return Object.keys(map).filter((key) => map[key] === max);
};
```

#### [All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

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

#### [Split BST](https://github.com/grandyang/leetcode/issues/776)

res[0]: node.val <= v; res[1]: node.val > v

```js
function splitBST(root, v) {
  let res = [null, null];
  if (!root) return res;
  if (root.val <= v) {
    res = splitBST(root.right, v);
    root.right = res[0];
    res[0] = root;
  } else {
    res = splitBST(root.left, v);
    root.left = res[1];
    res[1] = root;
  }
  return res;
}
```

#### [Binary Tree Upside Down](https://github.com/grandyang/leetcode/issues/156)

```js
function upsideDownBinaryTree(root) {
  if (!root || !root.left) return root;
  let l = root.left,
    r = root.right,
    newRoot = upsideDownBinaryTree(l);
  l.left = r;
  l.right = root;
  root.left = null;
  root.right = null;
  return newRoot;
}
```

#### [Path Sum IV](https://github.com/grandyang/leetcode/issues/666)

return the sum of all paths from the root towards the leaves.

关键是如何根据 root 的信息得到 left & right 的对应信息 => 建立 [level, pos] => val 的映射关系。

```js
var pathSum = function (nums) {
  if (nums.length === 0) return 0;
  let res = 0,
    map = new Map();
  for (let num of nums) {
    map.set(Math.floor(num / 10), num % 10);
  }
  const helper = (node, curr) => {
    let level = Math.floor(node / 10),
      pos = node % 10;
    let left = (level + 1) * 10 + 2 * pos - 1,
      right = left + 1;
    curr += map.get(node);
    if (!map.has(left) && !map.has(right)) {
      res += curr;
      return;
    }
    if (map.has(left)) helper(left, curr);
    if (map.has(right)) helper(right, curr);
  };

  helper(Math.floor(nums[0] / 10), 0);
  return res;
};
```

#### [Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)

将左子树的右分枝不断并入右子树前。

```js
var flatten = function (root) {
  let curr = root;
  while (curr) {
    if (curr.left) {
      let node = curr.left;
      while (node.right) node = node.right;
      node.right = curr.right;
      curr.right = curr.left;
      curr.left = null;
    }
    curr = curr.right;
  }
};
```

#### [Add One Row to Tree](https://leetcode.com/problems/add-one-row-to-tree/)

由于题目定义根节点 d = 1，所以递归终止 d = 0 or d = 1，处理 >= 2 的情况。

利用特殊的递归参数区分情况。d = 0 或 d = 1 均表示该行需要插入节点，d = 0 表示原节点应被当作新节点的右节点，d = 1 表示原节点应被当作新节点的左节点，同时 d = 1 覆盖了在根节点前新增一行这种情况。

```js
var addOneRow = function (root, v, d) {
  if (d == 0 || d == 1) {
    var newRoot = new TreeNode(v);
    newRoot.left = d === 1 ? root : null;
    newRoot.right = d === 0 ? root : null;
    return newRoot;
  }
  if (root !== null && d >= 2) {
    root.left = addOneRow(root.left, v, d > 2 ? d - 1 : 1);
    root.right = addOneRow(root.right, v, d > 2 ? d - 1 : 0);
  }
  return root;
};
```

#### [Smallest String Starting From Leaf](https://leetcode.com/problems/smallest-string-starting-from-leaf/)

反向添加字符，即 ch + curr。

```js
var smallestFromLeaf = (root, curr = "") => {
  if (!root) return curr;
  let ch = String.fromCharCode(root.val + 97);
  if (!root.left) return smallestFromLeaf(root.right, ch + curr);
  if (!root.right) return smallestFromLeaf(root.left, ch + curr);

  let l = smallestFromLeaf(root.left, ch + curr),
    r = smallestFromLeaf(root.right, ch + curr);
  return l < r ? l : r;
};
```

#### [Flip Binary Tree To Match Preorder Traversal](https://leetcode.com/problems/flip-binary-tree-to-match-preorder-traversal/)

需要 helper 函数判断是否能通过 flip 构成给定的 preorder 序列。

```js
var flipMatchVoyage = function (root, v) {
  let res = [],
    idx = 0;
  const dfs = (root) => {
    if (!root) return true;
    if (root.val !== v[idx]) return false;
    idx++;
    if (root.left && root.left.val !== v[idx]) {
      res.push(root.val);
      return dfs(root.right) && dfs(root.left);
    }
    return dfs(root.left) && dfs(root.right);
  };
  return dfs(root) ? res : [-1];
};
```

#### [Longest Univalue Path](https://leetcode.com/problems/longest-univalue-path/)

[idea: same like Binary Tree Maximum Path Sum](https://leetcode.com/problems/longest-univalue-path/discuss/108136/JavaC%2B%2B-Clean-Code)

```js
var longestUnivaluePath = function (root) {
  let max = 0;
  const dfs = (root) => {
    if (!root) return 0;
    let l = root.left ? dfs(root.left) : 0,
      r = root.right ? dfs(root.right) : 0;
    l = root.left && root.left.val === root.val ? l + 1 : 0;
    r = root.right && root.right.val === root.val ? r + 1 : 0;
    max = Math.max(max, l + r);
    return Math.max(l, r);
  };
  dfs(root);
  return max;
};
```

#### [Construct String from Binary Tree](https://leetcode.com/problems/construct-string-from-binary-tree/)

```js
var tree2str = function (root) {
  if (root === null) return "";
  if (root.left === null && root.right === null) return root.val.toString();

  let left = tree2str(root.left),
    right = tree2str(root.right);
  if (root.left && root.right)
    return root.val + "(" + left + ")" + "(" + right + ")";
  if (root.left === null) return root.val + "()" + "(" + right + ")";
  if (root.right === null) return root.val + "(" + left + ")";
};
```

#### [Construct Binary Tree from String](https://github.com/grandyang/leetcode/issues/536)

主要是正则。

```js
function str2tree(s) {
  if (s === "") return null;
  let groups = /([0-9])+(\((.*)\))?\((.*)\)/g.exec(s);
  if (groups === null) {
    return new TreeNode(parseInt(s));
  }
  let root = new TreeNode(parseInt(groups[1]));
  if (groups[3] === undefined) {
    root.left = str2tree(groups[4]);
  } else {
    root.left = str2tree(groups[3]);
    root.right = str2tree(groups[4]);
  }
  return root;
}
```

#### [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

注意：节点值可能重复。按照 preorder traversal 序列化反序列化。

```js
/**
 * Encodes a tree to a single string.
 *
 * @param {TreeNode} root
 * @return {string}
 */
var serialize = function (root) {
  let res = [];
  const helper = (node) => {
    if (node) {
      res.push(node.val);
      helper(node.left);
      helper(node.right);
    } else {
      res.push("#");
    }
  };
  helper(root);
  return res.join(" ");
};

/**
 * Decodes your encoded data to tree.
 *
 * @param {string} data
 * @return {TreeNode}
 */
var deserialize = function (data) {
  let chars = data.split(" ");
  const helper = () => {
    let ch = chars.shift();
    if (ch === "#") return null;
    let node = new TreeNode(parseInt(ch));
    node.left = helper();
    node.right = helper();
    return node;
  };
  return helper();
};
```

类似题

[Serialize and Deserialize BST](https://leetcode.com/problems/serialize-and-deserialize-bst/): 若节点没有重复，可以省去 null => '#' 的映射，通过 Construct Binary Search Tree from preorder traversal 反序列化。若节点有重复则还是需要 null => '#' 的映射。

[Serialize and Deserialize N-ary Tree](https://github.com/grandyang/leetcode/issues/428): 记录 node.val 后记录 node.children.length。
