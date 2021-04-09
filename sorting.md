# [Sorting](https://en.wikipedia.org/wiki/Sorting_algorithm#Comparison_of_algorithms)

general key points: stable / in-place / complexity

🚨 注意：用 JS array 默认的 sort 方法排序结果会很奇怪，一定要给明确的 comparator。

## Simple sorts

### insertion sort

Taking elements from the list one by one and inserting them in their correct position into a new sorted list.

#### key points

- stable

- in-place

- relatively efficient for small lists and mostly sorted lists

- the new list and the remaining elements can share the array's space, but insertion is expensive, requiring shifting all following elements over by one

- insertion sort's advantage is that it only scans as many elements as it needs in order to place the k + 1st element, while selection sort must scan all remaining elements to find the k + 1st element.

#### complexity

- best: already sorted O(N)

- worst: O(N^2)

- average: O(N^2)

#### implementation

```js
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    for (let j = i; j > 0 && arr[j - 1] > arr[j]; j--) {
      let temp = arr[j];
      arr[j] = arr[j - 1];
      arr[j - 1] = temp;
    }
  }
}
```

### selection sort

Finds the minimum value, swaps it with the value in the first position, and repeats these steps for the remainder of the list.

#### key points

- stable

- in-place

- inefficient on large lists, and generally performs worse than the similar insertion sort

- One thing which distinguishes selection sort from other sorting algorithms is that it makes the minimum possible number of swaps, n − 1 in the worst case

#### complexity

- best: O(N^2)

- worst: already sorted O(N^2)

- average: O(N^2)

#### implementation

```js
function selectionSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    let min = i;
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] < arr[min]) min = j;
    }
    let temp = arr[min];
    arr[min] = arr[i];
    arr[i] = temp;
  }
}
```

### bubble sort

repeatedly steps through the list, compares adjacent elements and swaps them if they are in the wrong order.

#### key points

- stable

- in-place

#### complexity

- best(after optimization): already sorted O(N)

- worst: O(N^2)

- average: O(N^2)

#### implementation

```js
function bubbleSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        let temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
  }
}
```

optimization: 当整个已有序时跳出循环。

```js
function bubbleSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    let swapped = false;
    for (let j = 0; j < arr.length - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        swapped = true;
      }
    }
    if (!swapped) return;
  }
}
```

## Efficient sorts

### merge sort

recursively split the array in half util it's left with individual items, then compare and merge them back.

[3,2,1,4] => [3,2] & [1,4] => [3] [2] [1] [4] => [2,3] & [1,4] => [1,2,3,4]

#### key points

- stable

- divide and conquer

- often the best choice for sorting a linked list

- top down & bottom up

#### complexity

- best: O(N\*logN)

- worst: O(N\*logN)

- average: O(N\*logN)

- space: O(N) for copy original out and merge

- proof: see Algorithms page 272

#### implementation

1. top down (without extra tension about space)

```js
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = (arr.length / 2) | 0;

  const left = arr.slice(0, mid);
  const right = arr.slice(mid);

  return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right) {
  let res = [],
    l = 0,
    r = 0;

  while (l < left.length || r < right.length) {
    if (l >= left.length) res.push(right[r++]);
    else if (r >= right.length) res.push(left[l++]);
    else if (left[l] <= right[r]) res.push(left[l++]);
    else res.push(right[r++]);
  }

  return res;
}
```

2. top down version: use lo & hi indices to indicate positions; use an extra copy array to copy original out, and merge back.

```js
function mergeSort(nums) {
  return sort(nums, 0, nums.length - 1, []);
}

function sort(nums, lo, hi, copy) {
  if (lo >= hi) return nums;
  let mid = lo + Math.floor((hi - lo) / 2);
  sort(nums, lo, mid, copy);
  sort(nums, mid + 1, hi, copy);
  merge(nums, lo, mid, hi, copy);
  return nums;
}

function merge(nums, lo, mid, hi, copy) {
  let l = lo,
    r = mid + 1;
  for (let i = lo; i <= hi; i++) {
    copy[i] = nums[i];
  }
  for (let i = lo; i <= hi; i++) {
    if (l > mid) nums[i] = copy[r++];
    else if (r > hi) nums[i] = copy[l++];
    else if (copy[l] <= copy[r]) nums[i] = copy[l++];
    else nums[i] = copy[r++];
  }
}
```

3. bottom up version

```js
function mergeSort(arr) {
  const n = arr.length,
    alt = [];
  for (let sz = 1; sz < n; sz *= 2) {
    for (let lo = 0; lo < n - sz; lo += 2 * sz) {
      merge(arr, lo, lo + sz - 1, Math.min(lo + 2 * sz - 1, n - 1), alt);
    }
  }
  return arr;
}

// same as top down version
function merge(arr, lo, mid, hi, alt) {}
```

### quicksort

Select a 'pivot' element from the array and partition the other elements into two sub-arrays, according to whether they are less than or greater than the pivot.

#### key points

- not stable

- divide and conquer

- in-place

- for merge sort, we do the two recursive calls _before_ working on the whole array, while for quicksort, we do the two recursive calls _after_ working on the whole array

- may need to shuffle before sorting for preserving randomness

#### complexity

- best: O(N\*logN)

- worst: O(N^2)

  - sub-lists returned by the partitioning routine is of size n − 1 and this happens repeatedly in every partition.

- average: O(N\*logN)

#### implementation

1. standard

```js
function quickSort(arr) {
  sort(arr, 0, arr.length - 1);
  return arr;
}

function sort(arr, lo, hi) {
  if (hi <= lo) return;

  const pivot = partition(arr, lo, hi);
  sort(arr, lo, pivot - 1);
  sort(arr, pivot + 1, hi);
}

function partition(a, lo, hi) {
  let p = a[hi],
    i = lo;
  for (let j = lo; j <= hi; j++) {
    if (a[j] < p) exch(a, j, i++);
  }
  exch(a, i, hi);
  return i;
}

function exch(a, i, j) {
  let temp = a[i];
  a[i] = a[j];
  a[j] = temp;
}
```

2. with 3-way partitioning

split array into 3 parts: less than, equals to, and greater than a certain pivot. The equal-to partition doesn't need further sorting because all its elements are already equal.

far more efficient for arrays with large numbers of duplicate items.

```js
function quickSort(arr) {
  sort(arr, 0, arr.length - 1);
  return arr;
}

function sort(a, lo, hi) {
  if (lo >= hi) return;

  // a[lo..lt-1] < v = a[lt..gt] < a[gt+1..hi]
  let lt = lo,
    i = lo + 1,
    gt = hi,
    v = a[lo];
  while (i <= gt) {
    if (a[i] < v) exch(a, lt++, i++);
    else if (a[i] > v) exch(a, gt--, i); // exchanged [gt] might still greater than v, so i stays for the next comparison.
    else i++;
  }

  sort(a, lo, lt - 1);
  sort(a, gt + 1, hi);
}

function exch(a, i, j) {
  let temp = a[i];
  a[i] = a[j];
  a[j] = temp;
}
```

### heapsort

The Heapsort algorithm involves preparing the list by first turning it into a max heap. The algorithm then repeatedly swaps the first value of the list with the last value, decreasing the range of values considered in the heap operation by one, and sifting the new first value into its position in the heap.

#### key points

- not stable

- in-place

#### complexity

- best: O(N\*logN)

- worst: O(N\*logN)

- average: O(N\*logN)

#### implementation

##### binary heap

N: size of current heap
k = 0, not using.
parent: Math.floor(k/2)
left child: 2*k
right child: 2*k + 1
operations: swim(go-up) & sink(go-down)

swim: child node's val is larger than the parent node's val

```js
while (k > 1 && less(k / 2, k)) {
  exch(k / 2, k);
  k = k / 2;
}
```

sink: parent's val is less than one of the children's val

```js
while (2 * k <= N) {
  let j = 2 * k;
  if (j < N && less(j, j + 1)) j = j + 1; // find the larger one in children
  if (!less(k, j)) break; // compare the larger one with parent
  exch(k, j);
  k = j;
}
```

##### binary heap based priority queue

A priority queue could be either a max heap or a min heap.

operations: `enqueue` & `dequeue`.

enqueue

```js
N++;
pq[N] = v;
swim(N);
```

dequeue

```js
let peek = pq[TOP];
exch(TOP, N);
pq[N] = null;
N--;
sink(TOP);
return peek;
```

full implementation: check on [priority-queue-test.js](./priority-queue-test.js)

##### heapsort

use original array to build a max heap. arr[0] needs to be handled carefully.

```js
function heapsort(arr) {
  let n = arr.length;
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    sink(arr, i, n);
  } // => build up max heap
  for (let i = n - 1; i > 0; i--) {
    // => sort
    exch(arr, 0, i); // => move largest to end of the array
    sink(arr, 0, i); // => find next largest
  }
  return arr;
}

function sink(arr, i, n) {
  while (2 * i + 1 < n) {
    let max = 2 * i + 1;
    if (max < n - 1 && arr[max] < arr[max + 1]) max = max + 1;
    if (arr[i] >= arr[max]) break;
    exch(arr, i, max);
    i = max;
  }
}
```

### shellsort

The method starts by sorting pairs of elements far apart from each other, then progressively reducing the gap between elements to be compared.

#### key points

- not stable

- in-place

- shellsort is a variant of insertion sort that is more efficient for larger lists

- The running time is heavily dependent on the gap sequence it uses.

#### complexity

- best: O(N\*logN)

- worst & average: depends

#### implementation

```js
function shellsort(arr) {
  const n = arr.length;
  for (let gap = Math.floor(n / 2); gap > 0; gap = Math.floor(gap / 2)) {
    for (let i = gap; i < n; i++) {
      for (let j = i; j >= gap && arr[j] < arr[j - gap]; j -= gap) {
        exch(arr, j, j - gap);
      }
    }
  }
  return arr;
}
```

## Other sorts

### bucket sort

## Questions

### [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

堆排序 / 快速排序。

1. 维护 size = k 的最小堆 / 堆排序

1) 最小堆

```js
var findKthLargest = function (arr, k) {
  let minHeap = new MinHeap();

  for (let i = 0; i < arr.length; i++) {
    minHeap.insert(arr[i]);
    if (minHeap.N > k) {
      minHeap.delMin();
    }
  }

  return minHeap.min();
};
```

2. 堆排序 在 n - k 处提前返回

```js
function heapsort(arr, k) {
  let n = arr.length;
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    sink(arr, i, n);
  }
  // i > 0 => i >= n - k
  for (let i = n - 1; i >= n - k; i--) {
    exch(arr, 0, i);
    sink(arr, 0, i);
  }
  return arr[n - k];
}

var findKthLargest = function (arr, k) {
  return heapsort(arr, k);
};
```

2. 快排

1) 普通快排

2) 快排 => 快速选择 找到 k-indexed 元素

```js
var findKthLargest = function(nums, k) {
    return quickSelect(nums, 0, nums.length-1, nums.length-k);
};

function quickSelect(nums, lo, hi, k) {
  if (lo === hi) return nums[lo];

  let pivot = partition(nums, lo, hi);
  if (pivot === k) return nums[k];
  else if (pivot > k) return quickSelect(nums, lo, pivot - 1, k);
  else return quickSelect(nums, pivot + 1, hi, k);
}

function partition(arr, lo, hi) {
  let pivot = lo;
  for (let i = lo; i < hi; i++) {
    if (arr[i] < arr[hi]) {
      exch(arr, pivot, i);
      pivot++;
    }
  }
  exch(arr, pivot, hi);
  return pivot;
}

function exch(arr, left, right) {
  let temp = arr[left];
  arr[left] = arr[right];
  arr[right] = temp;
}
```

### [Minimum Moves to Equal Array Elements II](https://leetcode.com/problems/minimum-moves-to-equal-array-elements-ii/)

quick select 找中位数。对于 even length 的数组来说，选择 mid 左右的随便哪个都可以，结果是一样的。

```js
var minMoves2 = function(nums) {
  let median = quickSelect(nums, 0, nums.length-1, Math.floor(nums.length/2));
  return nums.reduce((s, c) => s + Math.abs(c - median), 0);
};
```

### [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

time complexity must be better than O(n log n) => HashMap + 桶排序。

map: num => feq;
buckets: index as feq, [index] => elements.

naturally sorted by index.

```js
var topKFrequent = function (nums, k) {
  let map = new Map();
  for (let i = 0; i < nums.length; i++) {
    map.set(nums[i], (map.get(nums[i]) || 0) + 1);
  }

  let buckets = [];
  for (let [num, feq] of map) {
    if (!buckets[feq]) buckets[feq] = [];
    buckets[feq].push(num);
  }

  let topK = [];
  for (let i = buckets.length - 1; i >= 0 && topK.length < k; i--) {
    if (buckets[i]) {
      topK = topK.concat(buckets[i]);
    }
  }

  return topK;
};
```

### [Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/)

同上。

```js
var frequencySort = function (s) {
  let map = new Map();
  for (let i = 0; i < s.length; i++) {
    map.set(s[i], map.has(s[i]) ? map.get(s[i]) + 1 : 1);
  }

  let buckets = [];
  for (let [ch, feq] of map) {
    if (!buckets[feq]) buckets[feq] = [];
    buckets[feq].push(ch.repeat(feq));
  }

  let res = "";
  for (let i = buckets.length - 1; i >= 0; i--) {
    if (buckets[i]) res += buckets[i].join("");
  }

  return res;
};
```

### [Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/)

If two words have the same frequency, then the word with the lower alphabetical order comes first.

=> HashMap + 桶排序，需要对 HashMap 的 key 按照字典序排序；HashMap + 最小堆。

```js
var topKFrequent = function (words, k) {
  let map = new Map();
  words.forEach((word) => map.set(word, 1 + (map.get(word) || 0)));
  return [...map.entries()]
    .sort((a, b) => (a[1] != b[1] ? b[1] - a[1] : a[0].localeCompare(b[0])))
    .map((a) => a[0])
    .slice(0, k);
};
```
