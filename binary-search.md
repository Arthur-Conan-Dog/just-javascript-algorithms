# Binary Search

实现前需要想好区间形式是 [] 还是 [)。

## Implementation

### 基本结构

元素已有序且不重复，找到目标元素（或重复但返回任意位置即可）。

```js
function binarySearch(arr, target) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else if (arr[mid] > target) {
      right = mid - 1;
    }
  }
  return -1;
}
```

闭区间 [left, right] 决定了 while 的终止条件，以及 arr[mid] != target 时边界如何扩展。

_Note: `left + Math.floor((right - left) / 2)` 防止溢出。_

### 寻找左侧边界的二分搜索

元素已有序且存在重复，找到目标元素 LTR 第一次出现的位置。

```js
function binarySearchLeftBound(arr, target) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (arr[mid] === target) {
      right = mid - 1;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else if (arr[mid] > target) {
      right = mid - 1;
    }
  }

  if (left > arr.length - 1 || arr[left] !== target) {
    return -1;
  }
  return left;
}
```

当 mid === target 时，收缩区间右侧，向目标元素可能的左边界靠拢，最终返回标志区间左侧的 left。

由于要找左边界，mid === target 时不能马上 return，所以在 while 结束之后需要再判断一次当前 arr[left] 是否等于 target。而在使用 arr[left] 之前，需要判断 left 是否已经超出数组边界。（left <= right, right = [0, arr.length - 1], 即 while 终止时 left 可能等于 arr.length）

返回值的特殊含义：可以代表 arr 中小于 target 的元素有多少个。

简化：

```js
function leftBound(arr, target) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }

  return left < arr.length && arr[left] === target ? left : -1;
}
```

### 寻找右侧边界的二分搜索

元素已有序且存在重复，找到目标元素 RTL 首次出现的位置。

```js
function rightBound(arr, target) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (arr[mid] === target) {
      left = mid + 1;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else if (arr[mid] > target) {
      right = mid - 1;
    }
  }

  if (right < 0 || arr[right] !== target) {
    return -1;
  }
  return right;
}
```

实现类似寻找左边界。当 mid === target 时，收缩区间左侧，向目标元素可能的右边界靠拢，最终返回标志区间右侧的 right。

由于要返回 right，所以需要对 right 的最终值进行检查。

### 前闭后开的实现

更简洁，少一次 l = r 时的对比，且退出循环时 l & r 值相等。

[ref](https://github.com/yangshun/lago/blob/master/src/algorithms/binarySearch.ts)

## Questions

将什么值作为左右边界？收缩区间的条件又是什么？=> 有的题会比较不直观，不容易看出来。

### [Valid Perfect Square](https://leetcode.com/problems/valid-perfect-square/)

Given a positive integer num, write a function which returns true if num is a perfect square else false.

```js
var isPerfectSquare = function (num) {
  let l = 1,
    r = num;
  while (l <= r) {
    let mid = l + Math.floor((r - l) / 2),
      prod = mid * mid;
    if (prod === num) return true;
    if (prod < num) {
      l = mid + 1;
    } else {
      r = mid - 1;
    }
  }
  return false;
};
```

思考：[] 更合适还是 [) 更合适？

```js
var isPerfectSquare = function (num) {
  let l = 1,
    r = num;
  while (l < r) {
    let mid = l + Math.floor((r - l) / 2),
      prod = mid * mid;
    if (prod === num) return true;
    else if (prod < num) {
      l = mid + 1;
    } else {
      r = mid;
    }
  }
  return l * l === num; // [1,1)
};
```

### [Sqrt(x)](https://leetcode.com/problems/sqrtx/)

Given a non-negative integer x, compute and return the square root of x, truncate decimal part, only return the integer part.

```js
var mySqrt = function (x) {
  let lo = 1,
    hi = x;
  while (lo <= hi) {
    let mid = lo + Math.floor((hi - lo) / 2),
      prod = mid * mid;
    if (prod === x) return mid;
    if (prod < x) {
      lo = mid + 1;
    } else {
      hi = mid - 1;
    }
  }
  return x < lo * lo ? lo - 1 : lo;
};
```

```js
var mySqrt = function (x) {
  let l = 1,
    r = x;
  while (l < r) {
    let mid = l + Math.floor((r - l) / 2),
      prod = mid * mid;
    if (prod === x) return mid;
    else if (prod < x) {
      l = mid + 1;
    } else {
      r = mid;
    }
  }
  return l * l <= x ? l : l - 1;
};
```

### [First Bad Version](https://leetcode.com/problems/first-bad-version/)

找到满足 isBadVersion(n) === true 子序列的最左侧边界。

```js
var solution = function (isBadVersion) {
  /**
   * @param {integer} n Total versions
   * @return {integer} The first bad version
   */
  return function (n) {
    let l = 1,
      r = n;
    while (l <= r) {
      let mid = l + Math.floor((r - l) / 2);
      if (isBadVersion(mid)) {
        r = mid - 1;
      } else {
        l = mid + 1;
      }
    }
    return l;
  };
};
```

### [Missing Element in Sorted Array](https://leetcode.com/problems/missing-element-in-sorted-array/)

Given a sorted list of n-1 integers and these integers are in the range of 1 to n. There are no duplicates in list. One of the integers is missing in the list. Find the missing integer.

找到满足 arr[index] !== index + 1 子序列的最左侧边界。

```js
function findMissingElement(arr) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (arr[mid] - 1 !== mid) {
      right = mid - 1;
    } else {
      left = mid + 1;
    }
  }

  return left + 1;
}
```

### [Peak Index in a Mountain Array](https://leetcode.com/problems/peak-index-in-a-mountain-array/)

guaranteed exists.

找到满足 arr[mid] > arr[mid + 1] 区间的最左侧。

```js
var peakIndexInMountainArray = function (arr) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (arr[mid] < arr[mid + 1]) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return left;
};
```

Note: 最右侧也可以，题目没要求。

### [Find Peak Element](https://leetcode.com/problems/find-peak-element/)

同上题。

找到满足 arr[mid] > arr[mid + 1] 区间的最左侧。

```js
var findPeakElement = function (arr) {
  let l = 0,
    r = arr.length;
  while (l < r) {
    let mid = Math.floor((r + l) / 2),
      next = mid + 1 < arr.length ? arr[mid + 1] : -Infinity;
    if (arr[mid] < next) {
      l = mid + 1; // [l, mid] is going up
    } else {
      // [mid, r) is going down
      r = mid;
    }
  }
  return l;
};
```

### [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

通过 leftBound 和 rightBound 分别找到左右边界。=> 优化：第二次搜索找右边界时，范围区间是 [left, nums.length - 1]。

```js
var searchRange = function (arr, target) {
  let left = 0,
    right = arr.length - 1,
    res = [];
  while (left <= right) {
    let mid = Math.floor((right + left) / 2);
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  res[0] = left < arr.length && arr[left] === target ? left : -1;

  right = arr.length - 1;
  while (left <= right) {
    let mid = Math.floor((right + left) / 2);
    if (arr[mid] > target) {
      right = mid - 1;
    } else {
      left = mid + 1;
    }
  }
  res[1] = right >= 0 && arr[right] === target ? right : -1;

  return res;
};
```

### [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

思路一：通过判断 nums[mid] < nums[0] && nums[mid] < nums[nums.length - 1] 来缩小范围。

```js
var findMin = function (nums) {
  let left = 0,
    right = nums.length - 1;
  while (left <= right) {
    let mid = left + Math.floor((right - left) / 2);
    if (nums[mid] < nums[0] && nums[mid] < nums[nums.length - 1]) {
      right = mid - 1;
    } else {
      left = mid + 1;
    }
  }

  return left < nums.length
    ? nums[left]
    : nums[0] < nums[nums.length - 1]
    ? nums[0]
    : nums[nums.length - 1];
};
```

对于 [4,5,1,2,3] 的理想情况：如果 mid 在 [target, length - 1) 内，则符合条件，right 会不断向左靠近。如果 mid 在 [left, target) 内，则一定不符合条件，left => target 靠近。最终 left 与 right 重合，符合条件 => right - 1，while 终止，返回 left。

对于 [1,2,3,4] 的情况：在未到数组边界前，mid 一定不满足值小于最左端元素且小于最右端元素的条件。故，left 会持续增加至与 right 相等。此时还会再进行一次循环。left 再次偏移 = nums.length，越界。

对于 [4,3,2,1] 的情况：与上同。

如何解决 edge case：打补丁。

思路二 (recommend)：通过判断 nums[mid] > nums[r] 来缩小范围。

```js
var findMin = function (nums) {
  let l = 0,
    r = nums.length - 1;
  while (l < r) {
    const m = Math.floor((l + r) / 2);
    if (nums[m] > nums[r]) l = m + 1;
    else r = m;
  }
  return nums[l];
};
```

由于判断条件为 nums[m] > nums[r]，故 r 必须在 [0, length - 1] 范围内。

对于 [4,5,1,2,3] 的理想情况：如果 mid 在 [left, min) 内，按照题目定义，这个区间内的数值一定都大于 nums[right]，left => target 靠近。如果 mid 在 [min, right) 内，则不符合条件，right = mid。while 的终止条件是 l < r，所以即使 right = min 也并没有问题，left 会继续移动至 min 的位置。最终 left = right，跳出循环。

对于 [1,2,3,4] 的情况：mid 一定不满足值大于最右端元素的条件。故，right 会持续减少至与 left 相等。

对于 [4,3,2,1] 的情况：mid 一直满足值大于最右端元素的条件。故，left 会持续增加至与 right = length - 1 相等。

### [Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/)

基于上一题改动：可能存在重复元素。

设计用例：

偏转
[2,2,3,1]
[2,3,1,1]
[2,3,3,1]
[2,3,1,2]
[2,2,1,2]
[2,2,2,1,2]

正序
[1,1,2,2,3,3]

逆序
[3,3,2,2,1,1]

思路：消除重复元素。当重复只出现在一端时，I 的代码已经可以解决问题，因为相等时会向左侧收缩区间。如果同一个数字重复出现在了两端，I 的代码会导致过度收缩，比如：[2,2,2,1,2] 会收缩成 [2,2,2]，忽略了 1。我们只需要对这种情况特殊处理就可以了。即，当发现 nums[mid] = nums[r] 时只向左收缩一位，去掉重复。

```js
var findMin = function (nums) {
  let l = 0,
    r = nums.length - 1;
  while (l < r) {
    const m = Math.floor((l + r) / 2);
    if (nums[m] === nums[r]) r--;
    else if (nums[m] > nums[r]) l = m + 1;
    else r = m;
  }
  return nums[l];
};
```

### [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)

pivot 将数组分为两个局部有序的子数组 [0, pivot) & [pivot, nums.length - 1]。解题最关键的点是：1. 如何判断 mid 在哪个区间，以及 2. mid 和 target 是否处于同一区间。若为同一区间，则退化为二分搜索的一般情况；若为不同区间，则根据位置向 target 所在的区间收缩。

思路一：先确定 mid 所处的区间，再判断 target 所在的区间，再判断 mid 和 target 的相对位置（以 mid 处于 [0, pivot) 中为例）

1. 如何判断 mid 落在了这个区间？ => 若 mid 处于 [0, pivot) 中，则一定有 nums[mid] > nums[r]。

2. mid 和 target 是否处于同一区间？ => 若 target 也处于 [0, pivot) 中，则一定也有 target > nums[r]。此时再判断 mid 与 target 的相对关系：若 target < [mid]，则需要收缩右侧区间；若 [mid] < target，则需要收缩左侧区间。若 target 处于 [pivot, nums.length - 1] 中，说明当前位置是这样的：0 ... mid ... pivot ... target ... nums.length - 1，需要收缩左侧区间。

```js
var search = function (nums, target) {
  let l = 0,
    r = nums.length - 1;

  while (l <= r) {
    let m = l + Math.floor((r - l) / 2);
    if (nums[m] === target) return m;
    if (nums[m] > nums[r]) {
      if (target > nums[r] && nums[m] > target) {
        r = m - 1;
      } else {
        l = m + 1;
      }
    } else {
      if (target <= nums[r] && target > nums[m]) {
        l = m + 1;
      } else {
        r = m - 1;
      }
    }
  }

  return -1;
};
```

思路二：我们已经知道，当 [mid] & target 处于同一区间时，会退化成一般的二分搜索，那么当它们不属于同一区间时，能不能通过某种手段将这种特殊情况也转化成一般的二分搜索？

nums[m] < nums[0] === target < nums[0] 表示 [m] 和 target 处于同一区间。

当不属于同一区间时，若 target < nums[0]，则 target 一定在 [pivot, nums.length - 1] 中，而 [mid] 一定在 [0, pivot) 中，此时可以将 [mid] 的值看作负无穷，这样就回归到了一般的二分搜索中。

```js
var search = function (nums, target) {
  let l = 0,
    r = nums.length;

  while (l < r) {
    let m = Math.floor((l + r) / 2);
    let n =
      nums[m] < nums[0] === target < nums[0]
        ? nums[m]
        : target < nums[0]
        ? -Infinity
        : Infinity;

    if (n < target) {
      l = m + 1;
    } else if (n > target) {
      r = m;
    } else {
      return m;
    }
  }

  return -1;
};
```

注：nums[m] < nums[0] === target < nums[0] 的反面并不是 nums[m] > nums[0] === target > nums[0]，而是 nums[m] > nums[nums.length-1] === target > nums[nums.length-1]。因为 nums[m] > nums[0] === target > nums[0] 所指代的区间会略过 nums[0]，若 target 为第一个元素时就会出错。

### [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)

数组中元素可重复。

若重复出现在内部，如 [2,3,3,0,1]，则对之前的算法没什么影响。会影响的是当重复出现在两端，如 [1,3,1,1]，这样原算法所做的位置判断可能就会出错了。如何让情况回归到上一题？ => 消除重复。

思路一 v2:

```js
var search = function (nums, target) {
  let l = 0,
    r = nums.length - 1;

  while (l <= r) {
    let m = l + Math.floor((r - l) / 2);
    if (nums[m] === target) return m;
    if (nums[m] === nums[r]) {
      r--;
    } else if (nums[m] > nums[r]) {
      if (target > nums[r] && nums[m] > target) {
        r = m - 1;
      } else {
        l = m + 1;
      }
    } else {
      if (target <= nums[r] && target > nums[m]) {
        l = m + 1;
      } else {
        r = m - 1;
      }
    }
  }

  return -1;
};
```

思路二 v2：

对比思路一进行的改动要麻烦很多。

```js
var search = function (nums, target) {
  let l = 0,
    r = nums.length - 1;
  while (l <= r) {
    let m = Math.floor((l + r) / 2);
    // 补丁二：由于 r 的初始值改成了 nums.length - 1，while 条件对应改成了 l <= r，则存在 l === r 且 target === nums[l] 的情况，补丁一的 if 条件可能成立，导致边界越过 target，所以需要预先判断并返回。
    if (nums[m] === target) return m;
    // 补丁一：选择一端（右端）消除重复。由于要取 nums[r] 的值，所以 r 的初始值改成了 nums.length - 1 以防越界。
    if (nums[m] === nums[r]) {
      while (nums[m] === nums[r]) r--;
      continue;
    }
    let n =
      nums[m] < nums[0] === target < nums[0]
        ? nums[m]
        : target < nums[0]
        ? -Infinity
        : Infinity;

    if (n < target) {
      l = m + 1;
    } else if (n > target) {
      r = m - 1;
    } else {
      return m;
    }
  }

  return -1;
};
```

### [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/)

```js
var searchMatrix = function (matrix, target) {
  if (!matrix.length || !matrix[0].length) return false;
  let m = matrix.length;
  let n = matrix[0].length;
  let l = 0,
    r = m * n - 1;
  while (l < r) {
    let mid = Math.floor((l + r) / 2);
    if (matrix[Math.floor(mid / n)][mid % n] < target) l = mid + 1;
    else r = mid;
  }
  return matrix[Math.floor(l / n)][l % n] == target;
};
```

延伸题：[Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/)

row, left to right increasing; col, top to bottom, increasing.

```js
var searchMatrix = function (matrix, target) {
  if (!matrix.length || !matrix[0].length) return false;

  let i = 0,
    j = matrix[0].length - 1;

  while (i < matrix.length && j >= 0) {
    if (matrix[i][j] == target) return true;
    else if (matrix[i][j] < target) i++;
    else j--;
  }

  return false;
};
```

### [Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/)

n x n, each of the rows and columns are sorted in ascending order

O(N^2) => O(N\*logN)

用 matrix 首尾元素的值作为左右边界。

```js
var kthSmallest = function (matrix, k) {
  let n = matrix.length;
  let l = matrix[0][0],
    r = matrix[n - 1][n - 1];
  let mid,
    count = 0;

  while (l < r) {
    mid = l + Math.floor((r - l) / 2);
    count = 0;

    matrix.forEach((row) => {
      let j = 0;
      while (j < n && row[j] <= mid) j++;
      count += j;
    });

    if (count < k) l = mid + 1;
    else r = mid;
  }

  return l;
};
```

### [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/)

every element appears exactly twice except for one.

range: [0, n/2], condition: nums[mid*2] !== nums[mid*2+1].

```js
var singleNonDuplicate = function (nums) {
  let l = 0,
    r = Math.floor(nums.length / 2);
  while (l < r) {
    let mid = Math.floor((l + r) / 2);
    if (nums[mid * 2] === nums[mid * 2 + 1]) {
      l = mid + 1;
    } else {
      r = mid;
    }
  }
  return nums[l * 2];
};
```

### [Sum of Mutated Array Closest to Target](https://leetcode.com/problems/sum-of-mutated-array-closest-to-target/)

bestValue 取值范围: [0, maxInTheArray]。原因：即使 bestValue > maxInTheArray，按照题意 sum 整个数组得到的值也跟 bestValue = maxInTheArray 时相等。注意到这个区间是一个有序的递增区间，那么就可以使用二分搜索，找到满足条件 calc(arr) === target 的最小值左边界。

calc 的逻辑按题意：大于当前选取值的，取选取值；小于当前选取值的，取该值，最终得到 special sum。

```js
function calc(value) {
  return function (prev, curr) {
    return (curr > value ? value : curr) + prev;
  };
}

var findBestValue = function (arr, target) {
  let l = 0,
    r = 0;
  for (let i = 0; i < arr.length; i++) r = r > arr[i] ? r : arr[i];

  while (l <= r) {
    let mid = l + Math.floor((r - l) / 2);
    let sum = arr.reduce(calc(mid), 0);
    if (sum < target) l = mid + 1;
    else r = mid - 1;
  }

  let a = arr.reduce(calc(l), 0);
  let b = arr.reduce(calc(l - 1), 0);

  return Math.abs(a - target) < Math.abs(b - target) ? l : l - 1;
};
```

### [3Sum](https://leetcode.com/problems/3sum/)

dfs => TLE

```js
var threeSum = function (nums) {
  nums.sort((a, b) => a - b);
  let res = [];
  const dfs = (idx, curr, sum) => {
    if (sum === 0 && curr.length === 3) {
      res.push(curr.concat());
      return;
    }
    if (curr.length === 3) return;
    for (let i = idx; i < nums.length; i++) {
      curr.push(nums[i]);
      dfs(i + 1, curr, sum + nums[i]);
      curr.pop();
      while (i < nums.length - 1 && nums[i] == nums[i + 1]) {
        // skip duplicates
        i++;
      }
    }
  };
  dfs(0, [], 0);
  return res;
};
```

binary search

```js
var threeSum = function (nums) {
  nums.sort((a, b) => a - b);
  let res = [];
  for (let i = 0; i < nums.length - 2 && nums[i] <= 0; i++) {
    if (i === 0 || (i > 0 && nums[i] !== nums[i - 1])) {
      let lo = i + 1,
        hi = nums.length - 1,
        sum = -nums[i];
      while (lo < hi) {
        if (nums[lo] + nums[hi] === sum) {
          res.push([nums[i], nums[lo], nums[hi]]);
          while (lo < hi && nums[lo] === nums[lo + 1]) lo++;
          while (lo < hi && nums[hi] === nums[hi + 1]) hi++;
          lo++;
          hi--;
        } else if (nums[lo] + nums[hi] > sum) {
          hi--;
        } else {
          lo++;
        }
      }
    }
  }
  return res;
};
```
