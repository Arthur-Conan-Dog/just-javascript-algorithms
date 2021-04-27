# Linked List

Keynotes:

- linear data structure
- types: singly / doubly
- implementations: structure & operations(traversal, insertion, deletion etc.)
- complexity of operations
- two pointers technique
- dummy node technique(use as head/tail)
- check all things !== null before .! eg. node.next

Routines:

- traversal list
- reverse linked list
- find middle node using two pointers
- merge two lists

Corner cases:

- single node
- two nodes
- has cycle

## Implementation

Node

```js
class LinkedListNode {
  constructor(val) {
    this.val = val;
    this.next = null;
  }
}
```

## Two pointers technique

Two scenarios:

1. Two pointers starts at different position: eg. one starts at the beginning while another starts at the end;

2. Two pointers are moved at different speed: one is faster while another one might be slower.

Tips:

1. Always examine if the node is null before you call the next field.

2. Carefully define the end conditions of your loop.

Complexity Analysis:

Analyze how many times we will run our loop. Take 'detect cycle' as an example:

1. If there is no cycle => O(N/2) due to fast pointer

2. Has cycle => O(N + K), where K is the length of the cycle. Since K <= N, so in total it's O(2N) => O(N)

## Questions

### Basic Operations

#### [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists)

iteration

```js
var mergeTwoLists = function (l1, l2) {
  if (!l1) return l2;
  if (!l2) return l1;
  let dummy = new ListNode(),
    node = dummy;
  while (l1 && l2) {
    if (l1.val <= l2.val) {
      node.next = l1;
      l1 = l1.next;
    } else {
      node.next = l2;
      l2 = l2.next;
    }
    node = node.next;
  }
  node.next = l1 ? l1 : l2;
  return dummy.next;
};
```

recursion

```js
var mergeTwoLists = function (l1, l2) {
  if (!l1) return l2;
  if (!l2) return l1;

  if (l1.val < l2.val) {
    l1.next = mergeTwoLists(l1.next, l2);
    return l1;
  } else {
    l2.next = mergeTwoLists(l1, l2.next);
    return l2;
  }
};
```

#### [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)

iteration

```js
var reverseList = function (head) {
  let prev = null,
    curr = head;
  while (curr !== null) {
    let next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
  }
  return prev;
};
```

recursion

```js
var reverseList = function (node) {
  if (node === null) return null;
  if (node.next === null) return node;
  let next = node.next;
  node.next = null;
  let head = reverseList(next);
  next.next = node;
  return head;
};
```

#### [Remove Linked List Elements](https://leetcode.com/problems/remove-linked-list-elements/)

dummy node。

```js
var removeElements = function (head, val) {
  if (!head) return null;
  let dummy = new ListNode(),
    node = dummy;
  dummy.next = head;
  while (node && node.next) {
    if (node.next.val === val) {
      node.next = node.next.next;
    } else {
      node = node.next;
    }
  }
  return dummy.next;
};
```

#### [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

双指针 + 反转链表。

```js
var isPalindrome = function (head) {
  let fast = head,
    slow = head,
    rev = null;
  while (fast && fast.next) {
    fast = fast.next.next;
    let temp = slow.next;
    slow.next = rev;
    rev = slow;
    slow = temp;
  }

  let tail = slow,
    res = true;
  slow = fast ? slow.next : slow;
  while (slow) {
    res = res && slow.val === rev.val;
    slow = slow.next;
    let temp = rev.next;
    rev.next = tail;
    tail = rev;
    rev = temp;
  }
  return res;
};
```

#### [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle)

check whether has cycle.

快慢指针。

```js
var hasCycle = function (head) {
  if (!head) return false;
  let slow = head,
    fast = slow;
  while (fast.next && fast.next.next && slow.next) {
    fast = fast.next.next;
    slow = slow.next;

    if (fast === slow) return true;
  }
  return false;
};
```

#### [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii)

return the node where the cycle begins.

快慢指针。

```js
var detectCycle = function (head) {
  if (!head) return null;
  let fast = head,
    slow = fast;
  while (fast && fast.next) {
    fast = fast.next.next;
    slow = slow.next;
    if (fast === slow) {
      fast = head;
      while (fast !== slow) {
        fast = fast.next;
        slow = slow.next;
      }
      return slow;
    }
  }
  return null;
};
```

x1: head to cycle begin node;

x2: cycle begin node to slow & fast meet node;

x3: remaining cycle(slow & fast meet node to cycle begin node).

fast: x1 + x2 + x3 + x2

slow: x1 + x2

2 \* slow = fast => x1 = x3

#### [Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists)

数学思想。

```js
var getIntersectionNode = function (headA, headB) {
  let a = headA,
    b = headB;
  while (a !== b) {
    a = a ? a.next : headB;
    b = b ? b.next : headA;
  }
  return a;
};
```

a + c & b + c => a + c + b = a + b + c

#### [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

remove the nth node from the **end** of the list.

快慢指针。length - n.

```js
var removeNthFromEnd = function (head, n) {
  if (!head) return null;
  let dummy = new ListNode(),
    fast = dummy,
    slow = dummy;
  dummy.next = head;
  for (let i = 0; i < n; i++) {
    fast = fast.next;
  }
  while (fast.next) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next;
  return dummy.next;
};
```

#### [Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list)

倒腾指针。

```js
var oddEvenList = function (head) {
  if (head === null || head.next === null) return head;
  let odd = head,
    even = head.next,
    eHead = even;
  while (even && even.next) {
    odd.next = even.next;
    even.next = odd.next.next;
    odd = odd.next;
    even = even.next;
  }
  odd.next = eHead;
  return head;
};
```

#### [Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)

corner cases：single node; two nodes; triple nodes.

递归

```js
var swapPairs = function (head) {
  if (!head || !head.next) return head;

  let node = head.next;
  head.next = swapPairs(head.next.next);
  node.next = head;

  return node;
};
```

迭代 (dummy node)

```js
var swapPairs = function (head) {
  if (!head) return null;
  if (!head.next) return head;

  let dummy = new ListNode(),
    node = dummy;
  dummy.next = head;
  while (node.next && node.next.next) {
    let tail = node.next.next.next,
      p1 = node.next,
      p2 = node.next.next;
    node.next = p2;
    p2.next = p1;
    p1.next = tail;

    node = node.next.next;
  }

  return dummy.next;
};
```

#### [Partition List](https://leetcode.com/problems/partition-list/)

two lists, dummy node。

```js
var partition = function (head, x) {
  if (!head) return null;

  let h1 = new ListNode(),
    l1 = h1,
    h2 = new ListNode(),
    l2 = h2;

  while (head) {
    if (head.val < x) {
      l1.next = head;
      l1 = l1.next;
    } else {
      l2.next = head;
      l2 = l2.next;
    }
    head = head.next;
  }

  l2.next = null;
  l1.next = h2.next;
  return h1.next;
};
```

### Variants

#### [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

链表找环变体。快慢指针。index 作为 next pointer。

```js
var findDuplicate = function (nums) {
  let slow = nums[0],
    fast = nums[nums[0]];
  while (slow !== fast) {
    slow = nums[slow];
    fast = nums[nums[fast]];
  }

  slow = 0;
  while (slow !== fast) {
    slow = nums[slow];
    fast = nums[fast];
  }
  return slow;
};
```

#### [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

average length of lists is n.

1. merge lists one by one

```js
function mergeKLists(lists) {
  let res = null;
  for (let l of lists) {
    res = merge(res, l); // merge basic operation
  }
  return res;
}
```

time complexity: O(n), O(2n), O(3n) ... O(kn) => O(k(1+k)n/2) => O(k^2*n)

space complexity: O(1)

2. divide and conquer

```js
var mergeKLists = function (lists) {
  return partition(lists, 0, lists.length - 1);
};

function partition(lists, start, end) {
  if (start === end) return lists[start];
  if (start > end) return null;

  let mid = Math.floor((start + end) / 2),
    left = partition(lists, start, mid),
    right = partition(lists, mid + 1, end);
  return merge(left, right);
}
```

time complexity: for the recursion tree, it has logk levels, for each level, it needs O(k\*n) => O(k\*n\*logk)

space complexity: O(logk) for recursion stack space.

3. merge node of lists one by one

=> how to quickly find the current min in these lists?

=> priority queue

```js
function mergeKLists(lists) {
  let pq = new PriorityQueue((a, b) => a.val - b.val);
  for (let l of lists) {
    if (l) pq.enqueue(l);
  }
  let dummy = new ListNode(), node = dummy;
  while (pq.size()) {
    let currMin = pq.dequeue();
    node.next = currMin;
    node = node.next;
    if (currMin.next) pq.enqueue(currMin.next);
  }
  return dummy.next;
}
```

time complexity: O(k*n) nodes, O(logk) for each enqueue & dequeue operation => O(k\*n\*logk)

space complexity: O(k) for PQ.

#### [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

Swap Nodes in Pairs 变体 + 基础反转操作。递归 & 迭代。

iteration

```js
var reverseKGroup = function (head, k) {
  let dummy = new ListNode();
  dummy.next = head;
  let slow = dummy,
    fast = slow;

  while (fast) {
    for (let i = 0; i < k && fast; i++) {
      fast = fast.next;
    }
    if (!fast) break;
    let fn = fast.next,
      sn = slow.next;
    fast.next = null;
    slow.next = reverse(sn);
    sn.next = fn;
    slow = sn;
    fast = slow;
  }

  return dummy.next;
};
```

recursion

```js
var reverseKGroup = function (head, k) {
  if (!head) return null;

  let tail = head;
  for (let i = 1; i < k; i++) {
    tail = tail.next;
    if (!tail) return head;
  }

  let next = tail.next;
  tail.next = null;
  reverse(head);
  head.next = reverseKGroup(next, k);

  return tail;
};
```

#### [Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii)

Given the head of a singly linked list and two integers left and right where left <= right, reverse the nodes of the list from position left to position right, and return the reversed list.

反转链表变体。

```js
var reverseBetween = function (head, m, n) {
  let dummy = new ListNode();
  dummy.next = head;

  let slow = dummy,
    fast = dummy;
  for (let i = 0; i < n; i++) {
    fast = fast.next;
    if (i < m - 1) slow = slow.next;
  }

  let fn = fast.next,
    sn = slow.next;
  fast.next = null;
  slow.next = reverse(sn);
  sn.next = fn;

  return dummy.next;
};
```

#### [Reorder List](https://leetcode.com/problems/reorder-list/)

快慢指针中点 + 反转链表 + 合并链表。

```js
var reorderList = function (head) {
  if (!head || !head.next) return;

  let mid = head,
    fast = head;
  while (fast.next && fast.next.next) {
    fast = fast.next.next;
    mid = mid.next;
  }

  mid.next = reverse(mid.next);

  let node = head;
  while (node !== mid) {
    let midNext = mid.next,
      nodeNext = node.next;
    mid.next = midNext.next;
    node.next = midNext;
    midNext.next = nodeNext;
    node = nodeNext;
  }
};
```

#### [Sort List](https://leetcode.com/problems/sort-list/)

快慢指针找中点 + 二分法递归排序 + 合并链表。

```js
var sortList = function (head) {
  if (!head || !head.next) return head;

  let prev = null,
    slow = head,
    fast = head;
  while (fast && fast.next) {
    prev = slow;
    slow = slow.next;
    fast = fast.next.next;
  }

  prev.next = null;
  return merge(sortList(head), sortList(slow));
};
```

#### [Insert into a Cyclic Sorted List](https://github.com/grandyang/leetcode/issues/708)

一般情况：1 -> 3 -> 4, val = 2;
特殊情况：empty list; 4 -> 1 -> 3, val = 0 / val = 5.

```js
function insert(head, insertVal) {
  if (!head) {
    head = new ListNode(insertVal, null);
    head.next = head;
    return head;
  }
  let node = head;
  while (true) {
    if (node.val <= insertVal && insertVal <= node.next.val) break;
    if (
      node.val > node.next.val &&
      (node.val <= insertVal || insertVal <= node.next.val)
    )
      break;
    node = node.next;
  }
  node.next = new ListNode(insertVal, node.next);
  return head;
}
```

#### [Insertion Sort List](https://leetcode.com/problems/insertion-sort-list/)

新起一个链表，不断从所给链表中摘取节点插入新链表中。

```js
var insertionSortList = function (head) {
  let dummy = new ListNode(),
    prev = dummy;
  while (head !== null) {
    let temp = head.next;
    if (prev.val >= head.val) prev = dummy;
    while (prev.next && prev.next.val < head.val) prev = prev.next;
    head.next = prev.next;
    prev.next = head;
    head = temp;
  }
  return dummy.next;
};
```

#### [Swapping Nodes in a Linked List](https://leetcode.com/problems/swapping-nodes-in-a-linked-list/)

swap kth node from start and kth node from end.

```js
var swapNodes = function (head, k) {
  let k1 = null,
    k2 = null;
  for (let node = head; node !== null; node = node.next) {
    if (k2) k2 = k2.next;
    if (--k === 0) {
      k1 = node;
      k2 = head;
    }
  }
  let temp = k1.val;
  k1.val = k2.val;
  k2.val = temp;
  return head;
};
```

#### [Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/submissions/)

delete **all** nodes that have duplicate numbers. 注意 move next pointer 时的截断条件。

```js
var deleteDuplicates = function (head) {
  let dummy = new ListNode(0, head),
    node = dummy;

  while (node.next && node.next.next) {
    if (node.next.val === node.next.next.val) {
      let nn = node.next.next;
      while (nn.next && nn.next.val === node.next.val) nn = nn.next;
      node.next = nn.next;
    } else {
      node = node.next;
    }
  }

  return dummy.next;
};
```

### Others

#### [Add Two Numbers](https://leetcode.com/problems/add-two-numbers)

The digits are stored in reverse order.

加法运算，进位。

```js
var addTwoNumbers = function (l1, l2) {
  let dummy = new ListNode(),
    node = dummy,
    carry = 0;
  while (l1 || l2 || carry) {
    let sum = 0;
    if (l1) {
      sum += l1.val;
      l1 = l1.next;
    }
    if (l2) {
      sum += l2.val;
      l2 = l2.next;
    }
    sum += carry;
    carry = Math.floor(sum / 10);
    sum %= 10;
    node.next = new ListNode(sum);
    node = node.next;
  }
  return dummy.next;
};
```

#### [Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii/)

题目不允许反转链表。

借助栈反向构建链表。

```js
var addTwoNumbers = function (l1, l2) {
  let s1 = [],
    s2 = [];
  while (l1 !== null) {
    s1.push(l1.val);
    l1 = l1.next;
  }
  while (l2 !== null) {
    s2.push(l2.val);
    l2 = l2.next;
  }
  let dummy = null,
    carry = 0;
  while (s1.length || s2.length || carry != 0) {
    let a = s1.length ? s1.pop() : 0,
      b = s2.length ? s2.pop() : 0,
      c = a + b + carry,
      node = new ListNode(c % 10);
    node.next = dummy;
    dummy = node;
    carry = Math.floor(c / 10);
  }
  return dummy;
};
```

#### [Plus One Linked List](https://github.com/grandyang/leetcode/issues/369)

递归 / 反转链表。

```js
function plusOne(head) {
  if (!head) return head;

  const helper = (node) => {
    if (!node) return 1;
    let carry = helper(node.next);
    let sum = node.val + carry;
    node.val = sum % 10;
    return Math.floor(sum / 10);
  };

  let carry = helper(head);
  if (carry == 1) {
    let node = new ListNode(1);
    node.next = head;
    return node;
  }
  return head;
}
```

#### [Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list)

双向链表插入操作。迭代：每次向上挪一层。

```js
var flatten = function (head) {
  if (!head) return null;
  let node = head;
  while (node) {
    if (!node.child) node = node.next;
    else {
      let child = node.child;
      while (child.next) child = child.next;
      child.next = node.next;
      if (node.next) node.next.prev = child;
      node.next = node.child;
      node.child.prev = node;
      node.child = null;
    }
  }
  return head;
};
```

#### [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer)

1. HashMap O(N) & O(N)

iteration

```js
var copyRandomList = function (head) {
  if (!head) return null;
  let node = head,
    map = new Map();
  while (node) {
    map.set(node, new Node(node.val));
    node = node.next;
  }
  node = head;
  while (node) {
    map.get(node).next = map.get(node.next) || null;
    map.get(node).random = map.get(node.random) || null;
    node = node.next;
  }
  return map.get(head);
};
```

recursion

```js
var copyRandomList = function (head) {
  let map = new Map();
  function copy(node) {
    if (!node) return null;
    if (map.has(node)) return map.get(node);
    let n = new ListNode(node.val);
    map.set(node, n);
    n.next = copy(node.next);
    n.random = copy(node.random);
    return n;
  }
  return copy(head);
};
```

2. 1st iteration, link copy node right behind the original node; 2nd iteration, copy random pointers; 3rd iteration, extract copy nodes and recover original list. O(N) & O(1)

```js
var copyRandomList = function (head) {
  if (!head) return null;

  let node = head;
  while (node) {
    let temp = node.next;
    node.next = new Node(node.val);
    node.next.next = temp;
    node = temp;
  }

  node = head;
  while (node) {
    if (node.random) {
      node.next.random = node.random.next;
    }
    node = node.next.next;
  }

  node = head;
  let dummy = new Node(0),
    copy = dummy;
  while (node) {
    let temp = node.next.next;

    copy.next = node.next;
    copy = node.next;

    node.next = temp;
    node = temp;
  }

  return dummy.next;
};
```

#### [Rotate List](https://leetcode.com/problems/rotate-list/)

找到从左向右数对应的序号，将链表首尾相接，同时移动首尾到指定位置，最后再恢复。

```js
var rotateRight = function (head, k) {
  if (!head) return null;
  let tail = head,
    len = 1;
  while (tail.next) {
    tail = tail.next;
    len++;
  }
  tail.next = head;
  for (let i = len - (k % len); i > 0; i--) {
    tail = tail.next;
    head = head.next;
  }
  tail.next = null;
  return head;
};
```

#### [Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/)

快慢指针找中点。

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

  pre.next = null; // truncate list into three parts [-10, -3] & [0](slow) & [5, 9]
  let node = new TreeNode(slow.val); // slow is at the mid position
  node.left = sortedListToBST(head);
  node.right = sortedListToBST(slow.next);

  return node;
};
```

#### [Merge In Between Linked Lists](https://leetcode.com/problems/merge-in-between-linked-lists/)

```js
var mergeInBetween = function (list1, a, b, list2) {
  let l = null,
    r = null;
  for (let i = list1, idx = 0; i !== null; i = i.next, idx++) {
    if (idx === a - 1) l = i;
    if (idx === b) {
      r = i.next;
      break;
    }
  }
  l.next = list2;
  while (list2.next) list2 = list2.next;
  list2.next = r;
  return list1;
};
```
