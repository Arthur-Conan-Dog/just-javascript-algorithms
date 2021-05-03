# Array

## Intervals

a & b 无交集 => a.end < b.start || a. start > b.end;

so, a & b 有交集（对上面的逻辑取反） => a.end >= b.start && a.start <= b.end.

### [Meeting Rooms](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/252.%20Meeting%20Rooms.md)

determine whether two intervals have overlapping regions.

```js
function meetingRooms(intervals) {
  if (!intervals || intervals.length <= 1) return true;
  intervals.sort((a, b) => a[0] - b[0]);
  for (let i = 1; i < intervals.length; i++) {
    if (intervals[i - 1][1] > intervals[i][0]) return false;
  }
  return true;
}
```

### [Merge Intervals](https://leetcode.com/problems/merge-intervals/)

as the title says, merge given intervals.

use a buffer interval to compare, if buffer has overlap with current interval, merge them and continue; if has no overlap, push buffer into res, and reset current interval as buffer.

```js
var merge = function (intervals) {
  let res = [];
  intervals.sort((a, b) => a[0] - b[0]);
  let buffer = intervals[0];
  for (let interval of intervals) {
    if (interval[0] <= buffer[1]) {
      buffer[1] = Math.max(buffer[1], interval[1]);
    } else {
      res.push(buffer);
      buffer = interval;
    }
  }
  res.push(buffer);
  return res;
};
```

### [Meeting Rooms II](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/253.%20Meeting%20Rooms%20II.md)

find the minimum number of conference rooms required.

sort intervals according to the start time.

1. if currentMeeting.start < ongoingMeetingThatEndsEarliest.end, we need one more room;
2. if currentMeeting.start >= ongoingMeetingThatEndsEarliest.end, we could use that room, remove the that meeting from ongoing list, and add current meeting into it.
3. how to find the meeting that ends earliest in the ongoing list? we could use a minPQ.
4. at last, the size of the ongoing list would be the number of rooms we need.

```js
function minMeetingRooms(intervals) {
  if (!intervals) return 0;
  if (intervals.length <= 1) return intervals.length;
  intervals.sort((a, b) => a[0] - b[0]);
  let pq = new PriorityQueue((a, b) => a[1] - b[1]);
  let res = 1;
  pq.enqueue(intervals[0]);
  for (let i = 1; i < intervals.length; i++) {
    if (intervals[i][0] >= pq.peek()[1]) {
      pq.dequeue();
    }
    pq.enqueue(intervals[i]);
    res = Math.max(res, pq.size());
  }
  return res;
}
```

### [Insert Interval](https://leetcode.com/problems/insert-interval/)

insert a new interval into the intervals (merge if necessary).

目标区域的范围为 start 至 end，因为 intervals 已经有序，可以想象拖动要与其匹配的区间为滑块在下面自左向右滑动。最多需要考虑与其匹配的滑块大小大于/小于目标区域这两种情况。故：

1. 当 interval.end < start 时，没有交集；
2. 当 interval.start > end 时，没有交集；
3. 当 interval.end >= start && interval.start <= end 时，可以合并。前一个条件由于是从左向右拖动，当 interval.end < start 的检查不满足之后会自然存在，所以实际实现时可以省略。

```js
var insert = function (intervals, newInterval) {
  let res = [],
    i = 0,
    start = 0,
    end = 1,
    n = intervals.length;
  while (i < n && intervals[i][end] < newInterval[start]) {
    res.push(intervals[i++]);
  }
  while (i < n && intervals[i][start] <= newInterval[end]) {
    newInterval[start] = Math.min(newInterval[start], intervals[i][start]);
    newInterval[end] = Math.max(newInterval[end], intervals[i][end]);
    i++;
  }
  res.push([newInterval[start], newInterval[end]]);
  while (i < n) res.push(intervals[i++]);
  return res;
};
```

### [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/)

given two lists of closed intervals, each list of intervals is pairwise _disjoint_ and in _sorted_ order, return the intersection of these two interval lists.

a & b 有交集 => a.end >= b.start && a.start <= b.end。

设置两个 index，一个标记 a 当前遍历到的位置，另一个标记 b。若当前 index 对应的两个时间段有交集，则取交集部分加入 res。

何时移动 index？不论当前对比的两个时间段是否有交集，本轮遍历都应排除 end 更早的时间段。

```js
var intervalIntersection = function (first, sec) {
  let i = 0,
    j = 0,
    res = [];
  while (i < first.length && j < sec.length) {
    if (first[i][0] <= sec[j][1] && sec[j][0] <= first[i][1]) {
      res.push([
        Math.max(first[i][0], sec[j][0]),
        Math.min(first[i][1], sec[j][1]),
      ]);
    }
    if (first[i][1] <= sec[j][1]) i++;
    else j++;
  }
  return res;
};
```

### [Meeting Scheduler](https://poby.medium.com/leetcode-1229-meeting-scheduler-b9bed7c40a75)

given slots1 and slots2 of two people, return the earliest time slot that works for both of them and is of duration.

暴力解法：对比所有区间，第一个符合条件即返回。

```js
function minAvailableDuration(slots1, slots2, duration) {
  let m = slots1.length,
    n = slots2.length;
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      let [l1, r1] = slots1[i];
      let [l2, r2] = slots2[j];
      if (l2 > r1 || l1 > r2) continue;
      let start = Math.max(l1, l2),
        end = Math.min(r1, r2);
      if (end - start >= duration) return [start, start + duration];
    }
  }
  return [];
}
```

延用上一题思路：

```js
function minAvailableDuration(slots1, slots2, duration) {
  let m = slots1.length,
    n = slots2.length,
    i = 0,
    j = 0;
  while (i < m && j < n) {
    if (slots1[i][0] <= slots2[j][1] && slots1[i][1] >= slots2[j][0]) {
      let start = Math.max(slots1[i][0], slots2[j][0]),
        end = Math.min(slots1[i][1], slots2[j][1]);
      if (end - start >= duration) return [start, start + duration];
    }
    if (slots1[i][1] <= slots2[j][1]) i++;
    else j++;
  }
  return [];
}
```

### [Teemo Attacking](https://leetcode.com/problems/teemo-attacking/)

假设每次 attack 都被敌人完整接收，如果发现有区间 times[i] - times[i-1] 小于 duration，则减去之前多加上的部分。

```js
var findPoisonedDuration = function (timeSeries, duration) {
  if (!timeSeries || !timeSeries.length || duration == 0) return 0;
  let n = timeSeries.length,
    res = n * duration;
  for (let i = 1; i < timeSeries.length; i++) {
    res -= Math.max(0, duration - (timeSeries[i] - timeSeries[i - 1]));
  }
  return res;
};
```

### [Missing Ranges](https://github.com/grandyang/leetcode/issues/163)

given a sorted integer array, return its missing ranges by string in a particular format.

对比当前 num 和 lower，如果 num > lower 则说明有跳过的区间，需要加入 res；如果 num = upper，说明需要检查的区间已经全部检查完毕了，不需要再继续判断；更新下次迭代期望的 lower 值。对比最后的 lower 和 upper，处理可能存在的剩余区间。

```js
var missingRanges = function (nums, lower, upper) {
  let res = [];
  for (let num of nums) {
    if (num > lower) {
      res.push(num - 1 === lower ? lower.toString() : `${lower}->${num - 1}`);
    }
    if (num === upper) {
      return res;
    }
    lower = num + 1;
  }
  if (lower <= upper)
    res.push(lower === upper ? lower.toString() : `${lower}->${upper}`);
  return res;
};
```

## Math

### [Majority Element](https://leetcode.com/problems/majority-element/)

Given an array nums of size n, return the majority element.

```js
var majorityElement = function (nums) {
  let major = 0,
    count = 0,
    n = nums.length;
  for (let i = 0; i < n; i++) {
    if (count == 0) {
      major = nums[i];
      count = 1;
    } else if (major == nums[i]) {
      count++;
    } else {
      count--;
    }
  }
  return major;
};
```

### [Majority Element II](https://leetcode.com/problems/majority-element-ii/)

all elements appear more than ⌊ n/3 ⌋ times. => can at most have two candidates.

```js
var majorityElement = function (nums) {
  if (nums.length === 0) return [];
  if (nums.length === 1) return nums.concat();

  let count1 = 0,
    count2 = 0,
    m1 = 0,
    m2 = 1;
  for (let num of nums) {
    if (num === m1) count1++;
    else if (num === m2) count2++;
    else if (count1 === 0) {
      m1 = num;
      count1++;
    } else if (count2 === 0) {
      m2 = num;
      count2++;
    } else {
      count1--;
      count2--;
    }
  }
  let res = [],
    feq = Math.floor(nums.length / 3);
  if (nums.reduce((s, c) => s + Number(c === m1), 0) > feq) res.push(m1);
  if (nums.reduce((s, c) => s + Number(c === m2), 0) > feq) res.push(m2);
  return res;
};
```

Note: the value of m1 & m2 given at initialization doesn't matter, but they should be different from each other.

### [Find Pivot Index](https://leetcode.com/problems/find-pivot-index/)

Given an array of integers nums, calculate the leftmost pivot index of this array. The pivot index: separate the array to two half that sums equally.

```js
var pivotIndex = function (nums) {
  let sum = nums.reduce((s, c) => s + c, 0),
    l = 0,
    r = sum;
  for (let i = 0; i < nums.length; i++) {
    r -= nums[i];
    if (l === r) return i;
    l += nums[i];
  }
  return -1;
};
```

### [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)

Given an integer array, return the answer such that answer[i] is equal to the product of all the elements of nums except nums[i]. solve it without division and in O(n).

从左到右：res[i] = product of nums[0]...nums[i-1]；再从右到左: res[i] \*= product of nums[i+1]...nums[end]。

```js
var productExceptSelf = function (nums) {
  let res = [],
    left = 1;
  for (let i = 0; i < nums.length; i++) {
    res.push(left);
    left *= nums[i];
  }
  let right = 1;
  for (let i = nums.length - 1; i >= 0; i--) {
    res[i] *= right;
    right *= nums[i];
  }
  return res;
};
```

### [Ugly Number II](https://leetcode.com/problems/ugly-number-ii/)

An ugly number is a positive integer whose prime factors are limited to 2, 3, and 5. Given an integer n, return the nth ugly number. Note that 1 is always the first ugly number.

```js
var nthUglyNumber = function (n) {
  let ugly = [1],
    f2 = 2,
    f3 = 3,
    f5 = 5,
    idx2 = 0,
    idx3 = 0,
    idx5 = 0;
  for (let i = 1; i < n; i++) {
    let min = Math.min(f2, f3, f5);
    ugly[i] = min;
    if (f2 === min) f2 = 2 * ugly[++idx2];
    if (f3 === min) f3 = 3 * ugly[++idx3];
    if (f5 === min) f5 = 5 * ugly[++idx5];
  }
  return ugly[n - 1];
};
```

### Prefix Sum

i = 0: nums[0], sum[0] = nums[0]

i = 1: nums[1], sum[1] = nums[0] + nums[1]

i = 2: nums[2], sum[2] = nums[0] + nums[1] + nums[2]

... => sum[2] - sum[0] = nums[1] + nums[2], sum[2] - sum[1] = nums[2]

sub-array sum 问题，通常需要借助 hashmap 记录符合条件的子数组数量。

求 total number。求最大 / 最小 => 更新 hashmap 的时机。

#### [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

return the total number of continuous subarrays whose sum equals to k. nums[i] could be negative.

assume sum[0...i] - sum[0...j] = sum(j...i] = k, the number of subarrays that sum equals to sum[0...i] - k could be added to the result. how to track the number of such subarray? use hashmap.

```js
var subarraySum = function (nums, k) {
  let res = 0,
    map = {},
    sum = 0;
  map[0] = 1;
  for (let num of nums) {
    sum += num;
    res += map[sum - k] || 0;
    map[sum] = (map[sum] || 0) + 1;
  }
  return res;
};
```

#### [Longest Well-Performing Interval](https://leetcode.com/problems/longest-well-performing-interval/)

若 curr > 0，表明直到目前为止，sum[0, i] 都是正数，max 取当前长度 i - 0 + 1 = i + 1；

若 curr <= 0，表明当前和不再满足要求，需要找到 i 最小的 sum[i,j] = 1。

=> why 1？因为题目要求为 tiring day 数量严格大于 non-tiring days，在满足这个条件的情况下要使 subarray 最长，最极限的情况就是前者比后者刚好大 1。且 curr 是连续的，记录的又是每个取值首次出现的位置，如果要找 curr - 1 不存在，那么 curr - 2 / 3 / 4... 一定不会存在，反之如果 curr - 2 / 3 / 4... 存在，那么 curr - 1 肯定出现在其之前的位置，能够使得 subarray 长度更长。

```js
var longestWPI = function (hours) {
  let n = hours.length,
    max = 0,
    map = new Map(),
    curr = 0;
  for (let i = 0; i < n; i++) {
    curr += hours[i] > 8 ? 1 : -1;
    if (curr > 0) {
      max = i + 1;
    } else {
      if (map.has(curr - 1)) {
        max = Math.max(max, i - map.get(curr - 1));
      }
      if (!map.has(curr)) {
        map.set(curr, i);
      }
    }
  }
  return max;
};
```

#### [Minimum Operations to Reduce X to Zero](https://leetcode.com/problems/minimum-operations-to-reduce-x-to-zero/)

minimum operations => maximum subarray that sum = sum(nums) - x

```js
var minOperations = function (nums, x) {
  let n = nums.length,
    sum = nums.reduce((s, c) => s + c, 0);
  if (sum === x) return n;
  let target = sum - x,
    map = new Map(),
    max = -1,
    curr = 0;
  map.set(0, -1);
  for (let i = 0; i < n; i++) {
    curr += nums[i];
    if (map.has(curr - target)) {
      max = Math.max(max, i - map.get(curr - target));
    }
    // i - map.get(prev) 越小 max越大 要让 map.get(prev) 越小 则同样值要取 right most 位置
    // 所以永远 map.set(curr, i)
    map.set(curr, i);
  }
  return max === -1 ? max : n - max;
};
```

用 sliding window 更好，节省空间。等于求全是正数的 Sliding Window Maximum。

#### [Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)

假设 sum[0, j] - sum[0, i] = sub-array sum[i+1, j] = x \* k

=> 令 sum[0, j] = m _ k + mod1, sum[0, i] = n _ k + mod2

=> sum[0, j] - sum[0, i] = (m - n) _ k + mod1 - mod2 = x _ k

=> 则必有 mod1 = mod2

又：sum[0, i] = m \* k + mod1

=> sum[0, i + 1] % k = (m \* k + mod1 + nums[i+1]) % k = (mod1 + nums[i+1]) % k

=> sum += nums[i]; sum %= k;

```js
var checkSubarraySum = function (nums, k) {
  let map = new Map(),
    sum = 0;
  map.set(0, -1);
  for (let i = 0; i < nums.length; i++) {
    sum = (sum + nums[i]) % k;
    if (map.has(sum)) {
      if (i - map.get(sum) > 1) return true;
    } else {
      // 让 i - map.get(sum) 最大，只记录 first occurrence
      map.set(sum, i);
    }
  }
  return false;
};
```

#### [Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/)

注意与上一题 Continuous Subarray Sum 区别，本题数组中元素可为负，需要特别处理：if (sum < 0) sum += K。

```js
var subarraysDivByK = function (A, K) {
  let map = new Map(),
    sum = 0,
    res = 0;
  map.set(0, 1);
  for (let a of A) {
    sum = (sum + a) % K;
    if (sum < 0) sum += K;
    res += map.get(sum) || 0;
    map.set(sum, (map.get(sum) || 0) + 1);
  }
  return res;
};
```

#### [Make Sum Divisible by P](https://leetcode.com/problems/make-sum-divisible-by-p/)

(sum[0, n] - sum[i, j]) % p = 0

=> sum[0, n] % p = sum[i, j] % p

=> 找到满足此条件的最短子数组

target = mod = sum[0, n] % p

=> sum[i, j] % p = (sum[0, j] - sum[0, i]) % p = target

=> check whether already seen (sum[0, j] - target) % p

```js
var minSubarray = function (nums, p) {
  let mod = 0;
  for (let num of nums) {
    mod = (mod + num) % p;
    // need fix if nums have negative number
    // if (mod < 0) mod += p;
  }
  if (mod === 0) return 0;

  let n = nums.length,
    curr = 0,
    min = n,
    map = new Map();
  map.set(0, -1);
  for (let i = 0; i < n; i++) {
    curr = (curr + nums[i]) % p;
    // need fix if nums have negative number
    // if (curr < 0) curr += p;
    let prev = (curr - mod + p) % p; // curr - mod might be negative, need fix by adding p.
    if (map.has(prev)) min = Math.min(min, i - map.get(prev));
    map.set(curr, i);
  }
  return min < n ? min : -1;
};
```

Note: [why (curr - mod + p) % p instead of (curr - mod) % p directly?](https://leetcode.com/problems/make-sum-divisible-by-p/discuss/854895/How-to-approach-this-kind-of-problem-mind-map) => Why in java solutions we have to do a lookup for (cur - target + p)%p but for python solutions we do lookup only for (cur - target)%p

#### [Remove Zero Sum Consecutive Nodes from Linked List](https://leetcode.com/problems/remove-zero-sum-consecutive-nodes-from-linked-list/)

list = [1, 2, -3, 3, 1]
pref = [1, 3, 0, 3, 4]
idx = [0, 1, 2, 3, 4]

=> [3,1]

```js
var removeZeroSumSublists = function (head) {
  let sum = 0,
    map = new Map(),
    dummy = new ListNode(0);
  dummy.next = head;
  for (let i = dummy; i !== null; i = i.next) {
    sum += i.val;
    map.set(sum, i);
  }
  sum = 0;
  for (let i = dummy; i !== null; i = i.next) {
    sum += i.val;
    i.next = map.get(sum).next;
  }
  return dummy.next;
};
```

## Flip Array Elements

### [Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/)

Given an array nums of n integers where nums[i] is in the range [1, n], return an array of all the integers in the range [1, n] that do not appear in nums.

将 nums[i] 作为 idx 去访问数组元素。由于 1 <= nums[i] <= n，所以作为 idx 时要减 1。将访问到的数置为负，没有被访问到的说明其 idx + 1 不存在于数组中。

```js
var findDisappearedNumbers = function (nums) {
  let res = [],
    n = nums.length;
  for (let i = 0; i < nums.length; i++) {
    if (nums[Math.abs(nums[i]) - 1] > 0) {
      nums[Math.abs(nums[i]) - 1] *= -1;
    }
  }
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] > 0) res.push(i + 1);
    else nums[i] *= -1;
  }
  return res;
};
```

### [Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/)

some elements appear twice and others appear once.

```js
var findDuplicates = function (nums) {
  let res = [];
  for (let i = 0; i < nums.length; i++) {
    let index = Math.abs(nums[i]) - 1; // nums[i] might have already been set to negative at this point
    if (nums[index] < 0) {
      res.push(Math.abs(nums[i]));
    }
    nums[index] = -nums[index];
  }
  return res;
};
```

### [First Missing Positive](https://leetcode.com/problems/first-missing-positive/)

```js
var firstMissingPositive = function (nums) {
  let set = new Set(nums),
    res = 1;
  while (res <= set.size) {
    if (!set.has(res)) return res;
    res++;
  }
  return res;
};
```

Follow up: O(n) time and uses constant extra space?

for the positive number, swap until it is placed at the right position: nums[nums[i]-1] = nums[i]; skip negative numbers since they won't affect the results. iterate again to find the first matched one.

```js
var firstMissingPositive = function (nums) {
  let n = nums.length;
  for (let i = 0; i < n; i++) {
    while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
      let idx = nums[i];
      nums[i] = nums[idx - 1];
      nums[idx - 1] = idx;
    }
  }
  for (let i = 0; i < n; i++) {
    if (nums[i] != i + 1) return i + 1;
  }
  return n + 1;
};
```

## Matrix

### [Rotate Image](https://leetcode.com/problems/rotate-image/)

clockwise rotate: swap according to left diagonal and reverse.

123 => 147 => 741
456 => 258 => 852
789 => 369 => 963

```js
var rotate = function (matrix) {
  let n = matrix.length;
  for (let i = 0; i < n; i++) {
    for (let j = i + 1; j < n; j++) {
      let temp = matrix[i][j];
      matrix[i][j] = matrix[j][i];
      matrix[j][i] = temp;
    }
    matrix[i].reverse();
  }
};
```

### [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)

螺旋矩阵。考验设计 case 来验证正确性。

```js
var spiralOrder = function (matrix) {
  let m = matrix.length,
    n = matrix[0].length;
  let i = 0,
    j = 0,
    rowStart = 0,
    rowEnd = m,
    colStart = 0,
    colEnd = n;
  let res = [];
  while (i >= rowStart && i < rowEnd && j >= colStart && j < colEnd) {
    for (i = rowStart, j = colStart; j < colEnd; j++) {
      res.push(matrix[i][j]);
    }
    rowStart++;
    for (i = rowStart, j = colEnd - 1; i < rowEnd; i++) {
      res.push(matrix[i][j]);
    }
    colEnd--;
    for (i = rowEnd - 1, j = colEnd - 1; i >= rowStart && j >= colStart; j--) {
      res.push(matrix[i][j]);
    }
    rowEnd--;
    for (i = rowEnd - 1, j = colStart; i >= rowStart && j < colEnd; i--) {
      res.push(matrix[i][j]);
    }
    colStart++;
    i = rowStart;
    j = colStart;
  }
  return res;
};
```

### [Reshape the Matrix](https://leetcode.com/problems/reshape-the-matrix/)

m x n => r x c

```js
var matrixReshape = function (nums, r, c) {
  if (nums.length * nums[0].length !== r * c) return nums;

  let res = [],
    prevC = nums[0].length;
  for (let i = 0; i < r; i++) {
    res.push([]);
    for (let j = 0; j < c; j++) {
      let x = Math.floor((i * c + j) / prevC);
      let y = (i * c + j) % prevC;
      res[i][j] = nums[x][y]; // prev position
    }
  }

  return res;
};
```

### [Pascal's Triangle](https://leetcode.com/problems/pascals-triangle/)

```js
var generate = function (numRows) {
  let res = [[1]];
  if (numRows === 1) return res;
  for (let i = 1; i < numRows; i++) {
    let curr = [1];
    for (let r = i - 1, c = 0; c < i - 1; c++) {
      curr.push(res[r][c] + res[r][c + 1]);
    }
    curr.push(1);
    res.push(curr);
  }
  return res;
};
```

### [Pascal's Triangle II](https://leetcode.com/problems/pascals-triangle-ii/)

return the rowIndex-th (0-indexed) row of the Pascal's triangle. use only O(rowIndex) extra space.

for row index = 0 / 1, return [1], [1,1] directly;

start from row index >= 2, loop from i = 3 to row index + 1 times, calculate each row on the same array.

```js
var getRow = function (rowIndex) {
  let res = Array(rowIndex + 1).fill(1),
    prev = 1;
  for (let i = 3; i <= rowIndex + 1; i++) {
    for (let j = 1; j < i - 1; j++) {
      let temp = res[j];
      res[j] += prev;
      prev = temp;
    }
  }
  return res;
};
```

### [Image Overlap](https://leetcode.com/problems/image-overlap/)

For each pair of 1's you can make from all the 1's in A and B:

=> record the vector that points from pointA to pointB;

=> keep track of how many points correspond to the same vector;

=> choose the max one.

```js
var largestOverlap = function (A, B) {
  let n = A.length,
    la = [],
    lb = [];
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      if (A[i][j] === 1) la.push([i, j]);
      if (B[i][j] === 1) lb.push([i, j]);
    }
  }
  let map = new Map(),
    max = 0;
  for (let [x1, y1] of la) {
    for (let [x2, y2] of lb) {
      let s = x2 - x1 + "_" + (y2 - y1);
      map.set(s, (map.get(s) || 0) + 1);
      max = Math.max(max, map.get(s));
    }
  }
  return max;
};
```

### [Valid Sudoku](https://leetcode.com/problems/valid-sudoku/)

block index: (i / 3, j / 3)

```js
var isValidSudoku = function (board) {
  let set = new Set();

  for (let i = 0; i < 9; i++) {
    for (let j = 0; j < 9; j++) {
      let ch = board[i][j];
      if (ch !== ".") {
        if (
          set.has(ch + "in row" + i) ||
          set.has(ch + "in col" + j) ||
          set.has(ch + "in block" + Math.floor(i / 3) + "-" + Math.floor(j / 3))
        )
          return false;

        set.add(ch + "in row" + i);
        set.add(ch + "in col" + j);
        set.add(ch + "in block" + Math.floor(i / 3) + "-" + Math.floor(j / 3));
      }
    }
  }

  return true;
};
```

### [Flatten 2D Vector](https://github.com/grandyang/leetcode/issues/251)

for next() method: we need to check hasNext() first, so it's naturally we could place the logic of correcting invariants in hasNext() method.

how to correct invariants? how to move to next row and skip empty rows?

```js
class Vector2D {
  constructor(vector) {
    this.i = 0;
    this.j = 0;
    this.v = vector;
  }

  hasNext() {
    while (this.i < this.v.length && this.j === this.v[this.i].length) {
      this.i++;
      this.j = 0;
    }
    return this.i < this.v.length;
  }

  next() {
    if (!this.hasNext()) return null;
    return this.v[this.i][this.j++];
  }
}
```

## Reverse Procedure

### [Reveal Cards In Increasing Order](https://leetcode.com/problems/reveal-cards-in-increasing-order/)

given a list of numbers in unsorted order, return a sequence that could reorder it by below steps: 1. Take the top card out of the deck, gather it; 2. If there are still cards in the deck, put the next top card to the bottom. 3. repeat 1 & 2 until cards are all gathered.

so, given the sorted target list, we could generate the required sequence by reversing above steps: 1. If there are still cards in the res, put the bottom card to the top. 2. set last element of target as the new top of the res sequence.

```js
var deckRevealedIncreasing = function (deck) {
  let target = deck.concat(),
    res = [];
  target.sort((a, b) => a - b);
  while (target.length) {
    if (res.length > 0) res.unshift(res.pop());
    res.unshift(target.pop());
  }
  return res;
};
```

### [Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/)

sort people first by height desc, then by k asc, and insert into res by k value one by one.

```js
var reconstructQueue = function (people) {
  people.sort(([h1, k1], [h2, k2]) => (h1 != h2 ? h2 - h1 : k1 - k2));
  let res = [];
  for (let p of people) {
    res.splice(p[1], 0, p);
  }
  return res;
};
```

## Others

### [Maximum Swap](https://leetcode.com/problems/maximum-swap/)

O(n^2)

```js
function maximumSwap(num) {
  let arr = num
    .toString()
    .split("")
    .map((ch) => Number.parseInt(ch));

  for (let i = 0; i < arr.length; i++) {
    let j = i,
      max = i;
    while (j < arr.length) {
      if (arr[max] <= arr[j]) max = j;
      j++;
    }
    if (max !== i && arr[i] !== arr[max]) {
      let temp = arr[i];
      arr[i] = arr[max];
      arr[max] = temp;
      return parseInt(arr.join(""));
    }
  }
  return num;
}
```

O(n): record last pos in num for each digit

```js
function maximumSwap(num) {
  let arr = num
      .toString()
      .split("")
      .map((ch) => parseInt(ch)),
    buckets = Array(10).fill(0);
  for (let i = 0; i < arr.length; i++) {
    buckets[arr[i]] = i; // record last pos of this digit in num
  }

  for (let i = 0; i < arr.length; i++) {
    for (let k = 9; k > arr[i]; k--) {
      if (buckets[k] > i) {
        let tmp = arr[i];
        arr[i] = arr[buckets[k]];
        arr[buckets[k]] = tmp;
        return parseInt(arr.join(""));
      }
    }
  }

  return num;
}
```

### [Rotate Array](https://leetcode.com/problems/rotate-array/)

Given an array, rotate the array to the right by k steps.

when we rotate the array k times, k elements from the back end of the array come to the front and the rest of the elements from the front shift backwards. so we firstly reverse all the elements of the array. Then, reversing the first k elements followed by reversing the rest n-k elements gives us the required result.

```js
var rotate = function (nums, k) {
  k %= nums.length;
  reverse(nums, 0, nums.length - 1);
  reverse(nums, 0, k - 1);
  reverse(nums, k, nums.length - 1);
};

function reverse(nums, start, end) {
  while (start < end) {
    var temp = nums[start];
    nums[start++] = nums[end];
    nums[end--] = temp;
  }
}
```

### [Wiggle Sort](https://github.com/grandyang/leetcode/issues/280)

Given an unsorted array nums, reorder it in-place such that nums[0] <= nums[1] >= nums[2] <= nums[3]...

解法一：O(nlogn) 先排序，排序后一定有 a < b < c < d < e，此时只需每两个一组交换位置就能满足要求，即 a < c > b < e > d etc.

```js
function wiggleSort(nums) {
  nums.sort((a, b) => a - b);
  for (let i = 2; i < nums.length; i += 2) {
    let temp = nums[i];
    nums[i] = nums[i - 1];
    nums[i - 1] = temp;
  }
}
```

解法二：O(n) 当 i 为奇数时，应满足 nums[i] >= nums[i - 1]；当 i 为偶数时，应满足 nums[i] <= nums[i - 1]。

```js
function wiggleSort(nums) {
  for (let i = 1; i < nums.length; i++) {
    if (
      (i % 2 === 1 && nums[i] < nums[i - 1]) ||
      (i % 2 === 0 && nums[i] > nums[i - 1])
    ) {
      let temp = nums[i];
      nums[i] = nums[i - 1];
      nums[i - 1] = temp;
    }
  }
}
```

### [Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/)

Given an integer array nums, reorder it such that nums[0] < nums[1] > nums[2] < nums[3]...

Wiggle Sort solution won't work for [5,5,5,4,4,4,4].

O(nlogn) + O(n) sort and split to two groups: A & B, all elements in A is smaller than elements in B, then place them into the right place.

```js
var wiggleSort = function (nums) {
  nums.sort((a, b) => a - b);
  let n = nums.length,
    mid = (n + 1) >> 1;
  let A = nums.slice(0, mid),
    B = nums.slice(mid);

  for (let i = 0; i < n; i++) {
    if (i % 2 === 0) {
      nums[i] = A.pop();
    } else {
      nums[i] = B.pop();
    }
  }
};
```

### [Range Addition](https://github.com/grandyang/leetcode/issues/370)

For each update operation, do you really need to update all elements between i and j?

从 start 开始每个元素 + num，为了修正 end 后的数值，从 end + 1 开始每个元素 - num。

```js
function getModifiedArray(length, updates) {
  let res = Array(length).fill(0);
  for (let [start, end, num] of updates) {
    res[start] += num;
    if (end + 1 < length) res[end + 1] -= num;
  }
  for (let i = 1; i < length; i++) {
    res[i] += res[i - 1];
  }
  return res;
}
```

### [Range Addition II](https://github.com/grandyang/leetcode/issues/598)

只有 [min(i), min(j)] 范围内的格子们每次都得到了更新，数量为 min(i) \* min(j)。

```js
function maxCount(m, n, ops) {
  for (let [i, j] of ops) {
    m = Math.min(m, i);
    n = Math.min(n, j);
  }
  return m * n;
}
```

### [Next Permutation](https://leetcode.com/problems/next-permutation/)

same as Next Greater Element III.

```js
var nextPermutation = function (nums) {
  let i = nums.length - 2;
  while (i >= 0 && nums[i + 1] <= nums[i]) i--;
  if (i >= 0) {
    let j = nums.length - 1;
    while (j >= 0 && nums[j] <= nums[i]) j--;
    swap(nums, i, j);
  }
  let s = i + 1,
    e = nums.length - 1;
  while (s < e) {
    swap(nums, s, e);
    s++;
    e--;
  }
};

function swap(nums, i, j) {
  let temp = nums[i];
  nums[i] = nums[j];
  nums[j] = temp;
}
```
