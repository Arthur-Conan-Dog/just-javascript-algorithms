# Two pointers

## Two pointers & Array

### [Two Sum II - Input array is sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

```js
var twoSum = function (numbers, target) {
  let left = 0,
    right = numbers.length - 1;
  while (left < right) {
    let sum = numbers[left] + numbers[right];
    if (sum === target) return [left + 1, right + 1];
    else if (sum < target) left++;
    else right--;
  }
};
```

### [Two Sum Less Than K](https://www.youtube.com/watch?v=2Uq7p7HE0TI)

Given an array nums of integers and integer K, return the maximum s such that there exists i < j with A[i] + A[j] < K. If no such pair exists, return -1.

1. 暴力求解。O(n^2)。2) 先排序，后左右向中间靠拢。

```js
function twoSumLessThanK(nums, k) {
  nums.sort((a, b) => a - b);
  let l = 0,
    r = nums.length - 1,
    max = -1;
  while (l < r) {
    let sum = nums[l] + nums[r];
    if (sum >= k) r--;
    else {
      max = Math.max(max, sum);
      l++;
    }
  }
  return max;
}
```

### [Move Zeroes](https://leetcode.com/problems/move-zeroes/)

```js
var moveZeroes = function (nums) {
  let n = nums.length;
  for (let i = 0, j = 0; i < n; i++) {
    if (nums[i] !== 0) {
      let temp = nums[i];
      nums[i] = nums[j];
      nums[j] = temp;
      j++;
    }
  }
};
```

### [Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/)

```js
var sortedSquares = function (nums) {
  let n = nums.length,
    l = 0,
    r = n - 1,
    res = [];
  for (let k = n - 1; k >= 0; k--) {
    if (Math.abs(nums[l]) > Math.abs(nums[r])) {
      res[k] = nums[l] * nums[l];
      l++;
    } else {
      res[k] = nums[r] * nums[r];
      r--;
    }
  }
  return res;
};
```

### [Sort Colors](https://leetcode.com/problems/sort-colors/)

just like the 3-way partitioning for quicksort.

```js
var sortColors = function (nums) {
  let l = 0,
    r = nums.length - 1,
    i = 0;
  while (i <= r) {
    if (nums[i] === 0) {
      let temp = nums[l];
      nums[l] = nums[i];
      nums[i] = temp;
      i++;
      l++;
    } else if (nums[i] === 2) {
      let temp = nums[r];
      nums[r] = nums[i];
      nums[i] = temp;
      r--;
    } else {
      i++;
    }
  }
};
```

### [Remove Duplicates from Sorted Array II](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/)

```js
var removeDuplicates = function (nums) {
  let i = 0;
  for (let n of nums) {
    if (i < 2 || n != nums[i - 2]) {
      nums[i++] = n;
    }
  }
  return i;
};
```

### 同理：[Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

```js
var removeDuplicates = function (nums) {
  let i = 0;
  for (let n of nums) {
    if (i < 1 || nums[i - 1] != n) {
      nums[i++] = n;
    }
  }
  return i;
};
```

### [Pour Water](https://github.com/grandyang/leetcode/issues/755)

常规分析 case 题目。

```js
function pourWater(heights, V, K) {
  let n = heights.length;
  while (V--) {
    let l = K,
      r = K;
    while (l >= 0 && heights[l - 1] <= heights[l]) l--;
    while (l < K && heights[l] === heights[l + 1]) l++;
    while (r < n && heights[r] >= heights[r + 1]) r++;
    while (r > K && heights[r] === heights[r - 1]) r--;
    if (heights[l] < heights[K]) {
      heights[l]++;
    } else {
      heights[r]++;
    }
  }
  return heights;
}
```


### [Valid Triangle Number](https://leetcode.com/problems/valid-triangle-number/)

```js
function triangleNumber(nums) {
  let res = 0, n = nums.length;
  nums.sort((a, b) => a - b);
  for (let k = n - 1; k >= 0; k--) {
    for (let i = 0, j = k - 1; i < j;) {
      if (nums[i] + nums[j] > nums[k]) {
        res += j - i;
        j--;
      } else {
        i++;
      }
    }
  }
  return res;
}
```

[ref](https://leetcode.com/problems/valid-triangle-number/discuss/128135/A-similar-O(n2)-solution-to-3-Sum)

## Two pointers & String

### [Reverse Vowels of a String](https://leetcode.com/problems/reverse-vowels-of-a-string/)

```js
var reverseVowels = function (s) {
  let vowels = new Set(["a", "e", "i", "o", "u"]),
    chars = s.split("");
  let l = 0,
    r = s.length - 1;
  while (l < r) {
    if (!vowels.has(chars[l].toLowerCase())) l++;
    else if (!vowels.has(chars[r].toLowerCase())) r--;
    else {
      let temp = chars[l];
      chars[l] = chars[r];
      chars[r] = temp;
      l++;
      r--;
    }
  }
  return chars.join("");
};
```

### [Long Pressed Name](https://leetcode.com/problems/long-pressed-name/)

cases:

1. when s1[i] === s2[j], need move both pointers;
2. when s2[j] === s2[j-1], only need move j;
3. when s1[i] !== s2[j] && s2[j] !== s2[j-1], which the latter condition means we could not skip this character for s2, and so it cannot be thought as a long pressed character, return false directly.
4. done all the checks on s2, check whether s1 also goes to the end.

```js
var isLongPressedName = function (name, typed) {
  if (name === typed) return true;
  let i = 0,
    m = name.length,
    n = typed.length;
  for (let j = 0; j < n; j++) {
    if (i < m && name[i] === typed[j]) {
      i++;
    } else if (typed[j] !== typed[j - 1]) {
      return false;
    }
  }
  return i === m;
};
```

## Rain Water

### [Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

choose all possibilities of (i, j) in a wise way:

- ai < aj we will check the next (i+1, j) (or move i to the right)

- ai >= aj we will check the next (i, j-1) (or move j to the left)

why?

当 a[i] < a[j] 时，比如：[3, 2, 4, 7, 6]，left bar = a[0] = 3，假设不移动 left bar，由于此时每一格能使用的最大高度为 3，当 right bar 左移时，矩形宽度会减少，即使 a[3] 大于 a[4]，得到的面积也不可能再超过当前 right bar = a[4] = 6 所围成的矩形面积，因而如果想得到潜在的最大面积，应左移 left bar。当 a[i] >= a[j] 时道理类似。

```js
var maxArea = function (height) {
  let i = 0,
    j = height.length - 1,
    max = 0;
  while (i < j) {
    max = Math.max(max, Math.min(height[i], height[j]) * (j - i));
    if (height[i] < height[j]) {
      i++;
    } else {
      j--;
    }
  }
  return max;
};
```

### [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

基于 DP 的思路，能否更简化？能否在遍历的过程中得到 leftMax & rightMax？

只要 rightMax[i] > leftMax[i]，则当前蓄水高度由 left bar 决定，那么应该移动 left，反之移动 right。移动的过程中更新 leftMax & rightMax，若当前区域低于 max，则表明可以蓄水。

```js
var trap = function (height) {
  let n = height.length,
    res = 0;
  let left = 0,
    right = n - 1,
    leftMax = 0,
    rightMax = 0;
  while (left < right) {
    if (height[left] < height[right]) {
      if (height[left] > leftMax) {
        leftMax = height[left];
      } else {
        res += leftMax - height[left];
      }
      left++;
    } else {
      if (height[right] > rightMax) {
        rightMax = height[right];
      } else {
        res += rightMax - height[right];
      }
      right--;
    }
  }
  return res;
};
```

## Sliding Window

max => outer loop

min => inner loop

count number of valid solutions => res += end - start + 1, see explanation in "Subarray Product Less Than K".

### Continuous Subarray

#### [Minimum Operations to Reduce X to Zero](https://leetcode.com/problems/minimum-operations-to-reduce-x-to-zero/)

等于求和为 sum - x 的 maximum window size。

```js
var minOperations = function (nums, x) {
  let n = nums.length,
    sum = nums.reduce((s, c) => s + c, 0);
  if (sum < x) return -1;
  if (sum === x) return n;
  let curr = 0,
    max = -1;
  for (let i = 0, j = 0; j < n; j++) {
    curr += nums[j];
    while (curr > sum - x) {
      curr -= nums[i];
      i++;
    }
    if (curr === sum - x) {
      max = Math.max(max, j - i + 1);
    }
  }
  return max === -1 ? max : n - max;
};
```

#### [Get Equal Substrings Within Budget](https://leetcode.com/problems/get-equal-substrings-within-budget/)

等于求 sum(s[i]-t[i], ..., s[j]-t[j]) <= maxCost 的 maximum window size。

```js
var equalSubstring = function (s, t, maxCost) {
  let cost = 0,
    max = 0;
  for (let i = 0, j = 0; j < s.length; j++) {
    cost += Math.abs(s.charCodeAt(j) - t.charCodeAt(j));
    while (cost > maxCost) {
      cost -= Math.abs(s.charCodeAt(i) - t.charCodeAt(i));
      i++;
    }
    max = Math.max(max, j - i + 1);
  }
  return max;
};
```

#### [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/)

maximum window of countOfZeros === k.

```js
var longestOnes = function (A, K) {
  let counts = 0,
    n = A.length,
    res = 0;
  for (let start = 0, end = 0; end < n; end++) {
    if (A[end] === 0) counts++;
    while (counts > K) {
      if (A[start] === 0) counts--;
      start++;
    }
    res = Math.max(res, end - start + 1);
  }
  return res;
};
```

#### [Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/)

```js
var numberOfSubarrays = function (nums, k) {
  let n = nums.length,
    res = 0,
    cntOnes = 0,
    prev = 0;
  for (let start = 0, end = 0; end < n; end++) {
    if (nums[end] % 2 === 1) {
      prev = 0;
      cntOnes++;
    }
    while (cntOnes === k) {
      prev++;
      if (nums[start] % 2 === 1) cntOnes--;
      start++;
    }
    res += prev;
  }
  return res;
};
```

optimize:

TODO: 何时可以直接操作 k 来替代 cntOnes？对比 at least 和 exactly。

```js
var numberOfSubarrays = function (nums, k) {
  let n = nums.length,
    res = 0,
    cnt = 0;
  for (let start = 0, end = 0; end < n; end++) {
    if (nums[end] % 2 === 1) {
      k--;
      cnt = 0;
    }
    while (k === 0) {
      cnt++;
      k += nums[start] % 2;
      start++;
    }
    res += cnt;
  }
  return res;
};
```

#### [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/)

求 count number。

why take `res += j - i + 1` as current loop number of current solution? => after while loop, current window is valid, and unique solutions in this window would be every sub-array inside this window that ends with nums[j], which is the length of the window. eg. in window [1,2,1,2,2] (k = 2), valid and won't cover previous window's solutions are: [1,2,1,2,2], [2,1,2,2], [1,2,2], [2,2], [2].

```js
var numSubarrayProductLessThanK = function (nums, k) {
  if (k === 0) return 0;
  let res = 0,
    prod = 1;
  for (let i = 0, j = 0; j < nums.length; j++) {
    prod *= nums[j];
    while (i <= j && prod >= k) {
      prod /= nums[i];
      i++;
    }
    res += j - i + 1;
  }
  return res;
};
```

#### [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/)

exactly(k) = atMost(k) - atMost(k - 1)

same like other 'at most' questions, but operations on map will cause TLE.

```js
var subarraysWithKDistinct = function (A, K) {
  const atMostK = (s, k) => {
    let res = 0,
      map = {};
    for (let i = 0, j = 0; j < s.length; j++) {
      map[s[j]] = (map[s[j]] || 0) + 1;
      while (Object.keys(map).length > k) {
        map[s[i]]--;
        if (map[s[i]] === 0) delete map[s[i]];
        i++;
      }
      res += j - i + 1;
    }
    return res;
  };
  return atMostK(A, K) - atMostK(A, K - 1);
};
```

optimize: use k directly.

```js
var subarraysWithKDistinct = function (A, K) {
  const atMostK = (s, k) => {
    let res = 0,
      map = {};
    for (let i = 0, j = 0; j < s.length; j++) {
      map[s[j]] = (map[s[j]] || 0) + 1;
      if (map[s[j]] === 1) k--; // 满足条件所需的 integer 少了一个
      while (k < 0) {
        map[s[i]]--;
        if (map[s[i]] === 0) k++; // 满足条件所需的 integer 多了一个
        i++;
      }
      res += j - i + 1;
    }
    return res;
  };
  return atMostK(A, K) - atMostK(A, K - 1);
};
```

#### [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/)

求满足 sum >= target 的 minimum window。

```js
var minSubArrayLen = function (target, nums) {
  let n = nums.length,
    sum = 0,
    min = n + 1;
  for (let i = 0, j = 0; j < n; j++) {
    sum += nums[j];
    while (sum >= target) {
      min = Math.min(min, j - i + 1);
      sum -= nums[i];
      i++;
    }
  }
  return min <= n ? min : 0;
};
```

#### [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

保持窗口 len = k，并找出窗口中的最大值。为了优化每次窗口长度合法时，遍历窗口内的数字找最大值的过程，可以使用双端队列。

```js
var maxSlidingWindow = function (nums, k) {
  let queue = [],
    res = [];
  for (let start = 0, end = 0; end < nums.length; end++) {
    while (queue.length && nums[end] > queue[queue.length - 1]) queue.pop();
    queue.push(nums[end]);
    while (end - start + 1 > k) {
      if (nums[start] === queue[0]) queue.shift();
      start++;
    }
    if (end - start + 1 === k) res.push(queue[0]);
  }
  return res;
};
```

#### [Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)

Minimum Size Subarray Sum 变体 => 数组中的数可以为负。此时不能再通过原数组进行滑动窗口。

```js
function shortestSubarray(A, K) {
  let n = A.length,
    dp = Array(n + 1).fill(0),
    d = [],
    min = n + 1;
  for (let i = 0; i < n; i++) dp[i + 1] = dp[i] + A[i];
  for (let i = 0; i <= n; i++) {
    while (d.length && dp[i] <= dp[d[d.length - 1]]) d.pop();
    while (d.length && dp[i] - dp[d[0]] >= K)
      min = Math.min(min, i - d.shift());
    d.push(i);
  }
  return min <= n ? min : -1;
}
```

#### [Constrained Subsequence Sum](https://leetcode.com/problems/constrained-subsequence-sum/)

dp[i] 表示只使用前 i 个元素能得到的最大值。则 dp[i] = max{ dp[i-1], dp[i-2], ..., dp[i-k] } + nums[i]

brute force: TLE

```js
var constrainedSubsetSum = function (nums, k) {
  let n = nums.length,
    dp = Array(n),
    res = -Infinity;
  for (let i = 0; i < n; i++) {
    let max = 0;
    for (let j = Math.max(i - k, 0); j < i; j++) {
      max = Math.max(max, dp[j]);
    }
    dp[i] = nums[i] + max;
    res = Math.max(res, dp[i]);
  }
  return res;
};
```

optimize: use dequeue to track the maximum

```js
var constrainedSubsetSum = function (nums, k) {
  let queue = [],
    res = nums[0],
    dp = nums.concat();
  for (let i = 0; i < dp.length; i++) {
    dp[i] += queue.length ? queue[0] : 0;
    res = Math.max(res, dp[i]);
    while (queue.length && dp[i] > queue[queue.length - 1]) queue.pop();
    if (queue.length && i >= k && queue[0] == dp[i - k]) queue.shift();
    if (dp[i] > 0) queue.push(dp[i]);
  }
  return res;
};
```

ref: https://leetcode.com/problems/constrained-subsequence-sum/discuss/605822/Java-Decreasing-Monotonic-Queue-Clean-code-O(N)

#### [Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)

brute force

```js
var longestSubarray = function (nums, limit) {
  let res = 0;
  for (
    let start = 0, end = 0, min = nums[0], max = nums[0];
    end < nums.length;
    end++
  ) {
    min = Math.min(min, nums[end]);
    max = Math.max(max, nums[end]);
    while (nums[end] - min > limit || max - nums[end] > limit) {
      if (nums[start] === min || nums[start] === max) {
        min = Infinity;
        max = 0;
        for (let i = start + 1; i <= end; i++) {
          min = Math.min(min, nums[i]);
          max = Math.max(max, nums[i]);
        }
      }
      start++;
    }
    res = Math.max(res, end - start + 1);
  }
  return res;
};
```

optimize: use dequeue（双端队列）

```js
var longestSubarray = function (nums, limit) {
  let maxd = [],
    mind = [],
    res = 0;
  for (let start = 0, end = 0; end < nums.length; end++) {
    while (maxd.length && nums[end] > maxd[maxd.length - 1]) maxd.pop();
    while (mind.length && nums[end] < mind[mind.length - 1]) mind.pop();
    maxd.push(nums[end]);
    mind.push(nums[end]);
    if (maxd[0] - mind[0] > limit) {
      if (maxd[0] == nums[start]) maxd.shift();
      if (mind[0] == nums[start]) mind.shift();
      start++;
    }
    res = Math.max(res, end - start + 1);
  }
  return res;
};
```

### Substring

主要是用 hashmap 记录字符，以及 counts 记录 distinct character numbers。

#### [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)

Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"

```js
var minWindow = function (s, t) {
  let map = {};
  for (let ch of t) map[ch] = (map[ch] || 0) + 1;
  let start = 0,
    end = 0,
    min = Infinity,
    idx = 0,
    counts = Object.keys(map).length;
  while (end < s.length) {
    if (map[s[end]] !== undefined) map[s[end]]--;
    if (map[s[end]] === 0) counts--;
    end++;
    while (counts === 0) {
      if (end - start < min) {
        min = end - start;
        idx = start;
      }
      if (map[s[start]] !== undefined) map[s[start]]++;
      if (map[s[start]] === 1) counts++;
      start++;
    }
  }
  return min === Infinity ? "" : s.substr(idx, min);
};
```

#### [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/)

```js
var findAnagrams = function (s, p) {
  if (p.length > s.length) return [];
  let map = {};
  for (let ch of p) map[ch] = (map[ch] || 0) + 1;
  let k = Object.keys(map).length,
    res = [];
  for (let i = 0, j = 0; j < s.length; j++) {
    if (map[s[j]] !== undefined) map[s[j]]--;
    if (map[s[j]] === 0) k--;
    while (k === 0) {
      if (j - i + 1 === p.length) res.push(i);
      if (map[s[i]] !== undefined) map[s[i]]++;
      if (map[s[i]] > 0) k++;
      i++;
    }
  }
  return res;
};
```

#### [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

```js
var lengthOfLongestSubstring = function (s) {
  let n = s.length,
    res = 0,
    map = {};
  for (let start = 0, end = 0; end < n; end++) {
    if (map[s[end]] !== undefined) {
      start = Math.max(start, map[s[end]] + 1);
    }
    res = Math.max(res, end - start + 1);
    map[s[end]] = end;
  }
  return res;
};
```

#### [Longest Substring with At Most Two Distinct Characters](https://github.com/grandyang/leetcode/issues/159)

```js
var totalFruit = function (s) {
  let max = 0,
    map = {};
  for (let start = 0, end = 0; end < s.length; end++) {
    map[s[end]] = (map[s[end]] || 0) + 1;
    while (Object.keys(map).length > 2) {
      map[s[start]]--;
      if (map[s[start]] === 0) delete map[s[start]];
      start++;
    }
    max = Math.max(max, end - start + 1);
  }
  return max;
};
```

eceba => ec | ce | eb | ba

```js
var totalFruit = function (s) {
  let a = 0,
    b = 0,
    countB = 0,
    curr = 0,
    max = 0;
  for (let el of s) {
    curr = el === a || el === b ? curr + 1 : countB + 1;
    countB = el === b ? countB + 1 : 1;
    if (el !== b) {
      a = b;
      b = el;
    }
    max = Math.max(max, curr);
  }
  return max;
};
```

same question: [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/)

#### [Longest Substring with At Most K Distinct Characters](https://github.com/grandyang/leetcode/issues/340)

```js
var lengthOfLongestSubstringKDistinct = function (s, k) {
  let max = 0,
    map = {};
  for (let i = 0, j = 0; j < s.length; j++) {
    map[s[j]] = (map[s[j]] || 0) + 1;
    while (Object.keys(map).length > k) {
      map[s[i]]--;
      if (map[s[i]] === 0) delete map[s[i]];
      i++;
    }
    max = Math.max(max, j - i + 1);
  }
  return max;
};
```

#### [Number of Substrings Containing All Three Characters](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/)

```js
var numberOfSubstrings = function (s) {
  let n = s.length,
    map = {},
    dp = Array(n).fill(0),
    res = 0;
  for (let start = 0, end = 0; end < n; end++) {
    map[s[end]] = (map[s[end]] || 0) + 1;
    let cnt = 0;
    while (Object.keys(map).length === 3) {
      cnt++;
      map[s[start]]--;
      if (map[s[start]] === 0) delete map[s[start]];
      start++;
    }
    dp[end] = (end > 0 ? dp[end - 1] : 0) + cnt;
    res += dp[end];
  }
  return res;
};
```

optimize:

a a a b b c c a b c
when all a, b, c > 0 for first time at end = 5 the n after while loop start will be at start = 3, we can simply add start = 3 to result because there would be three substrings from three a's.

```js
var numberOfSubstrings = function (s) {
  let n = s.length,
    cnt = [0, 0, 0],
    res = 0;
  for (let start = 0, end = 0; end < n; end++) {
    cnt[s.charCodeAt(end) - 97]++;
    while (cnt[0] > 0 && cnt[1] > 0 && cnt[2] > 0) {
      cnt[s.charCodeAt(start) - 97]--;
      start++;
    }
    res += start;
  }
  return res;
};
```

### Substring plus

判断条件不太好想

#### [Replace the Substring for Balanced String](https://leetcode.com/problems/replace-the-substring-for-balanced-string/)

窗口合法的条件为：外部各个字符计数小于等于 n / 4，则窗口代表的子串均需要进行字符替换，子串/窗口长度就是当前需要替换的次数。

不断移动左边界，刷新最小值记录。

特殊情况处理：已经平衡。

```js
var balancedString = function (s) {
  let n = s.length,
    map = { Q: 0, W: 0, E: 0, R: 0 },
    k = n / 4,
    res = n;
  for (let i = 0; i < n; i++) map[s[i]]++;
  for (let start = 0, end = 0; end < n; end++) {
    map[s[end]]--;
    while (
      start < n &&
      map["Q"] <= k &&
      map["W"] <= k &&
      map["E"] <= k &&
      map["R"] <= k
    ) {
      res = Math.min(res, end - start + 1);
      map[s[start]]++;
      start++;
    }
  }
  return res;
};
```

#### [Longest Substring with At Least K Repeating Characters](https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/)

https://leetcode.com/problems/longest-substring-with-at-least-k-repeating-characters/solution/

如何想到需要 currUnique & unique & counts？

```js
const getMaxUnique = (s) => {
  let map = {};
  for (let ch of s) {
    map[ch] = true;
  }
  return Object.keys(map).length;
};
var longestSubstring = function (s, k) {
  let map = {};
  let maxUnique = getMaxUnique(s),
    max = 0;
  for (let currUnique = 1; currUnique <= maxUnique; currUnique++) {
    map = {};
    let start = 0,
      end = 0,
      unique = 0,
      counts = 0;
    while (end < s.length) {
      if (unique <= currUnique) {
        let ch = s[end];
        if (map[ch] === undefined || map[ch] === 0) unique++;
        map[ch] = (map[ch] || 0) + 1;
        if (map[ch] === k) counts++;
        end++;
      } else {
        let ch = s[start];
        if (map[ch] === k) counts--;
        map[ch]--;
        if (map[ch] === 0) unique--;
        start++;
      }
      if (unique === currUnique && counts === unique) {
        max = Math.max(max, end - start);
      }
    }
  }
  return max;
};
```

#### [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)

The problem says that we can make at most k changes to the string (any character can be replaced with any other character). So, let's say there were no constraints like the k. Given a string convert it to a string with all same characters with minimal changes.

Given this, we can apply the at most k changes constraint and maintain a sliding window such that:

(length of substring - number of times of the maximum occurring character in the substring) <= k

Since we are only interested in the longest valid substring, our sliding windows need not shrink, even if a window may cover an invalid substring. We either grow the window by appending one char on the right, or shift the whole window to the right by one. And we only grow the window when the count of the new char exceeds the historical max count (from a previous window that covers a valid substring).

That is, we do not need the accurate max count of the current window; we only care if the max count exceeds the historical max count; and that can only happen because of the new char.

If replaceCount = (end - start + 1) - mostFreq is bigger than k, we got an invalid window, we should skip it until window is valid again, but only expands window size, never shrink (because even if we got a smaller window thats valid, it doesn't matter because we already found a window thats bigger and valid)

```js
var characterReplacement = function (s, k) {
  let n = s.length,
    map = {};
  let mostFreq = 0,
    max = 0;
  for (let start = 0, end = 0; end < n; end++) {
    map[s[end]] = (map[s[end]] || 0) + 1;
    mostFreq = Math.max(mostFreq, map[s[end]]);
    while (end - start + 1 - mostFreq > k) {
      map[s[start]]--;
      start++;
    }
    max = Math.max(max, end - start + 1);
  }
  return max;
};
```

#### [Partition Labels](https://leetcode.com/problems/partition-labels/)

O(n) 解法：记录每个字符最后一次出现的位置，从头遍历字符串，不断扩展当前访问过的字符能达到的最右端，当 i = right 时说明该区间已访问完，更新 left & right 访问下一个区间。

```js
var partitionLabels = function (S) {
  let map = {},
    res = [];
  for (let i = 0; i < S.length; i++) {
    map[S[i]] = i;
  }
  let left = 0,
    right = 0;
  for (let i = 0; i < S.length; i++) {
    right = Math.max(right, map[S[i]]);
    if (i === right) {
      res.push(right - left + 1);
      left = i + 1;
    }
  }
  return res;
};
```
