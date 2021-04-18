# Dynamic Programming

典型问题，直接代入。没有遇到过的问题，可以先按照下面这样的思考过程一步步尝试解决。

https://claytonjwong.github.io/The-ART-of-Dynamic-Programming/

1. All possibilities are considered via recursion, top-down brute-force depth-first-search

2. Remember each sub-problem’s optimal solution via a DP memo

3. Turn the top-down solution upside-down to create the bottom-up solution

## Key Points

动态规划一般求最值，凡是求最值都需要穷举所有可能情况。

重叠子问题、最优子结构、状态转移方程就是动态规划三要素。

明确 base case -> 明确「状态」-> 明确「选择」 -> 优化：dp[] 记录子问题解 -> 优化：压缩 dp[] 只记录必要值。

```python
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```

递归算法的时间复杂度：子问题个数乘以解决一个子问题需要的时间。

递归实现一般是自顶向下，大问题分解成小问题，递归返回局部解；迭代实现一般是自底向上，从 base case 出发，推出最终解。

最优子结构：将问题划分成子问题，各个子问题之间相互独立，解不受其他子问题影响。

优化 dp 空间：画图，找出必要状态，投影。处理 base case。

egs. array => pre & curr; [][] => []; [N] => [N%30]

## [7 Steps to solve a DP problem](https://dev.to/nikolaotasevic/dynamic-programming--7-steps-to-solve-any-dp-interview-problem-3870)

1. How to recognize a DP problem

Whether this problem solution can be expressed as a function of solutions to similar smaller problems.

2. Identify problem variables

Express the problem in terms of the function parameters and see which of those parameters are changing.

A way to determine the number of changing parameters is to list examples of several sub-problems and compare the parameters.

3. Clearly express the recurrence relation

Assuming you have computed the sub-problems, how would you compute the main problem?

4. Identify the base cases

The reason a problem cannot be simplified further is that one of the parameters would become a value that is not possible given constraints of a problem.

5. Decide if you want to implement it iteratively or recursively

Carefully think about the trade-offs.

6. Add memoization

Remember that memoization is just a cache of the function results. In recursive solutions, adding memoization should feel straightforward.

7. Determine time complexity

## Questions

### [背包问题](https://www.kancloud.cn/kancloud/pack/70124)

有 n 种物品，物品 j 的重量为 wj，价格为 pj。假定所有物品的重量和价格都是非负的。背包所能承受的最大重量为 W。求背包能装进的最大价值。

限定每种物品只能选择 0 个或 1 个，则问题称为 0-1 背包问题。

如果不限定每种物品的数量，则问题称为无界背包问题。

如果限定物品 j 最多只能选择 bj 个，则问题称为有界背包问题。

各类复杂的背包问题总可以变换为简单的 0-1 背包问题进行求解。

总结：

可选物品是否能被多次选中？

- 不能 => 0-1 背包。

  - 物品选择有多维限制？（如除重量外，还限制最多只能取 m[i] 件物品）

    - 有 => 多维费用的 0-1 背包问题。加一个纬度。见 Ones and Zeroes。

- 能 => 完全背包

  - 物品的拿取顺序是否影响结果？

    - 不影响 => 一般完全背包问题。组合。

    - 影响 => 涉及顺序的完全背包问题。排列。

#### 1. 0-1 背包问题

状态变量：可选择的物品种类、可容纳的最大重量。

选择：放入物品，或者不放入。

dp 定义：dp[i][j] 可选择的物品种类为前 i 种，可容纳的最大重量为 j 时，能达到的最大价值。0 <= i <= n, 0 <= j <= W.

基础情况：dp[0][...] = 0，表示没有可选择的物品；dp[...][0] = 0，表示可容纳的重量为 0。

假设已知前 i - 1 种可选的物品种类对应所有可能存在的背包重量下，能达到的最大价值，即已知 dp[i-1][0...w]，如何推出 dp[i][0...w]？

- 无法用物品 i 替换背包中的物品，即 j < w[i]，则 `dp[i][j] = dp[i-1][j]`;

- 可以用物品 i 替换背包中的物品，即 j >= w[i]，则：

  - 若不放入物品 i，则 `dp[i][j] = dp[i-1][j]`;

  - 若放入物品 i，则 `dp[i][j] = dp[i-1][j-w[i]] + v[i]`;

  - 比较两种情况取较大值。

```js
function zeroOneKnapsack(W, N, weights, values) {
  const dp = [...Array(N + 1)].map(() => Array(W + 1).fill(0));
  for (let i = 1; i <= N; i++) {
    const w = weights[i - 1],
      v = values[i - 1];
    for (let j = 1; j < w; j++) {
      // 放不下当前物品
      dp[i][j] = dp[i - 1][j];
    }
    for (let j = w; j <= W; j++) {
      // 可以放下当前物品 选择放与不放中价值较大者
      dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - w] + v);
    }
  }

  return dp[N][W];
}
```

复杂度：O(N\*W) & O(N\*W)

优化空间：内层循环每一轮只与前一轮的 j 有关，可以只用一维数组来记录。但为了防止前一轮的 j 被抹去，需要倒序求解。即：

```js
for (let j = W; j >= w; j--) {
  dp[j] = Math.max(dp[j], dp[j - w] + v);
}
```

#### 2. 完全背包问题

状态变量：可选择的物品种类、可容纳的最大重量。

选择：不放入物品 i；放入 0 - k 个 物品 i。

dp 定义：dp[i][j] 可选择的物品种类为前 i 种，可容纳的最大重量为 j 时，能达到的最大价值。0 <= i <= n, 0 <= j <= W.

基础情况：dp[0][...] = 0，表示没有可选择的物品；dp[...][0] = 0，表示可容纳的重量为 0。

假设已知前 i - 1 种可选的物品种类对应所有可能存在的背包重量下，能达到的最大价值，即已知 dp[i-1][0...w]，如何推出 dp[i][0...w]？

- 不可能用物品 i 替换背包中的物品，即 j < w[i]，则 `dp[i][j] = dp[i-1][j]`;

- 可以用物品 i 替换背包中的物品，即 j >= w[i]，则：

  - 若不放入物品 i，则 `dp[i][j] = dp[i-1][j]`;

  - 若放入物品 i，则 `dp[i][j] = dp[i][j-w[i]] + v[i]`;

    - _Note: 由于物品 i 可以重复选择，所以需要依赖的子问题变成了：`dp[i][j-w[i]]`，即可能已选入物品 i，与 0-1 背包中的子问题：`dp[i-1][j-w[i]]`是不同的。_

  - 比较两种情况取较大值。

```js
function unboundedKnapsack(W, N, weights, values) {
  const dp = [...Array(N + 1)].map(() => Array(W + 1).fill(0));
  for (let i = 1; i <= N; i++) {
    const w = weights[i - 1],
      v = values[i - 1];
    for (let j = 0; j < w; j++) {
      dp[i][j] = dp[i - 1][j];
    }
    for (let j = w; j <= W; j++) {
      dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - w] + v);
    }
  }

  return dp[N][W];
}
```

复杂度：O(N\*W) & O(N\*W)

优化空间：内层循环每一轮只与前一轮的 j 有关，可以只用一维数组来记录。由于依赖于本轮之前计算得到的 dp[i][j-w]，需要顺序求解。即：

```js
for (let j = w; j <= W; j++) {
  dp[j] = Math.max(dp[j], dp[j - w] + v);
}
```

思考过程：

1. 减少物品数量。去掉 w[i] > W 的物品；对物品按照 w[i] 排序（计数排序），只留下相同 w[i] 下 v[i] 最大的物品。

2. 转换成 0-1 背包问题。将一种物品可被多次选择转化成多种物品每种只能选一次。比如，每种物品最多选 Math.floor(W / w[i]) 件 => 该种物品转换成 Math.floor(W / w[i]) 件费用和价值均相等的物品，不过这样对于时间复杂度没有改善。再比如：拆成 w[i] <= w[i]\*2^k <= W 的物品，对应可以拆成 log(W/w[i]) 件，将选 x 件第 i 种物品通过选了 1 件 x 个第 i 种物品的和来简介表示，较前者有所改善。

3. 转换成 0-1 背包问题。重新回顾 0-1 背包问题的 dp 含义：dp[i][j] 表示只使用前 i 种物品，容量为 j 的情况下，所能得到的最大价值。状态转移方程：`dp[i][j] = dp[i-1][j]` 表示完全不用第 i 种物品；`dp[i][j] = dp[i-1][j-w[i]] + v[i]` 则表示在考虑选用第 i 种物品时，需要依赖的子问题为：此前**完全未选择过**第 i 种物品，且能容量能允许再放入**一个**物品 i。那么当物品 i 可以被重复选择时，在考虑“选用第 i 种物品”这一策略时，依赖的子问题就变成了：可能选择过第 i 种物品，且容量能允许再放入一个物品 i，即 `dp[i][j] = dp[i][j-w[i]] + v[i]`。

#### [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

Given a non-empty array nums containing only positive integers, find if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.

0-1 背包变体。转换背包容量。

```js
var canPartition = function (nums) {
  let sum = nums.reduce((total, curr) => total + curr, 0);
  if (sum % 2 !== 0) return false;

  let half = sum / 2,
    dp = new Array(half + 1).fill(false);
  dp[0] = true;

  for (let num of nums) {
    for (let i = half; i >= num; i--) {
      dp[i] = dp[i] || dp[i - num];
    }
  }

  return dp[half];
};
```

#### [Target Sum](https://leetcode.com/problems/target-sum/description/)

Adding one of the symbols '+' and '-' before each integer in nums and then concatenate all the integers. Return the number of different expressions that you can build, which evaluates to target.

0-1 背包变体。转换背包容量。

sum(P) - sum(N) = target

sum(P) - sum(N) + sum(P) + sum(N) = target + sum

sum(P) \* 2 = target + sum

sum(P) = (target + sum) / 2

```js
var findTargetSumWays = function (nums, S) {
  let sum = nums.reduce((total, curr) => total + curr, 0);
  if (sum < S || (sum + S) % 2 === 1) return 0;

  let half = (sum + S) / 2,
    dp = new Array(half + 1).fill(0);
  dp[0] = 1;
  for (let num of nums) {
    for (let i = half; i >= num; i--) {
      dp[i] += dp[i - num];
    }
  }

  return dp[half];
};
```

dfs + memo

```js
var findTargetSumWays = function (nums, S) {
  let len = nums.length,
    memo = new Map();
  function dfs(level, target) {
    if (level === len) {
      return target === 0 ? 1 : 0;
    }
    if (memo.has(level) && memo.get(level).has(target))
      return memo.get(level).get(target);

    let cnt =
      dfs(level + 1, target - nums[level]) +
      dfs(level + 1, target + nums[level]);
    if (!memo.has(level)) memo.set(level, new Map());
    memo.get(level).set(target, cnt);
    return cnt;
  }

  return dfs(0, S);
};
```

#### [Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/)

Return the smallest possible weight of the left stone. If there are no stones left, return 0.

0-1 背包变体。将石头分成两波，两波总重越接近，得到的结果越小。dp[i] 表示总重为 i 时，能装入石头的最大值。

```js
var lastStoneWeightII = function (stones) {
  let n = stones.length,
    sum = stones.reduce((prev, curr) => prev + curr, 0),
    half = Math.floor(sum / 2);
  let dp = [...Array(half + 1)].fill(0);
  for (let stone of stones) {
    for (let i = half; i >= stone; i--) {
      dp[i] = Math.max(dp[i], dp[i - stone] + stone);
    }
  }
  return sum - dp[half] - dp[half]; // (sum - dp[half]) - dp[half], dp[half]: 总重限制为 half 时能允许装入的石头重量。
};
```

[ref](https://leetcode.com/problems/last-stone-weight-ii/discuss/294888/JavaC%2B%2BPython-Easy-Knapsacks-DP)

#### [Split Array With Same Average](https://leetcode.com/problems/split-array-with-same-average/)

Given an integer array nums, move each element of nums into one of the two arrays A and B such that A and B are non-empty, and average(A) == average(B). Return true if it is possible to achieve that and false otherwise.

sum(A) / count(A) = sum(B) / count(B); count(B) = n - count(A); sum(B) = total - sum(A)
=> sum(A) / count(A) = total / n

assume A is the smaller half, then 1 <= count(A) <= n / 2.

early pruning: since sum(A) must be an integer, and we have count(A) in [1, n / 2], we can check for all possible count(A), if there exists one that makes `(total \* count(A)) % n === 0`.

0-1 背包 => combination sum of all possible count(A). dp[i] 表示当可选前 i 个数时，能得到的所有组合的和的情况。

```js
var splitArraySameAverage = function (nums) {
  let total = nums.reduce((s, c) => s + c, 0),
    n = nums.length,
    m = Math.floor(n / 2);

  if (!isPossible(total, m, n)) return false;

  let dp = [...Array(m + 1)].map(() => new Set());
  dp[0].add(0);
  for (let num of nums) {
    for (let i = m; i >= 1; i--) {
      for (let t of dp[i - 1]) {
        dp[i].add(t + num);
      }
    }
  }
  for (let i = 1; i <= m; i++) {
    if ((total * i) % n === 0 && dp[i].has((total * i) / n)) return true;
  }
  return false;
};

function isPossible(total, m, n) {
  let flag = false;
  for (let i = 0; i < m && !flag; i++) {
    if ((total * i) % n === 0) flag = true;
  }
  return flag;
}
```

#### [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/description/)

Given an array of binary strings strs and two integers m and n. Return the size of the largest subset of strs such that there are at most m 0's and n 1's in the subset.

多维费用的 0-1 背包问题。费用加了一维，只需状态也加一维。`dp[i][j][k]` 表示只使用前 i 个字符串，zero 数量为 j 时，one 数量为 k 时，最多能加入解集的字符串个数。其中 j <= m, k <= n。下面的实现简化去掉了 i 层，简化思路见 0-1 背包。

```js
var findMaxForm = function (strs, m, n) {
  let dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));
  for (let s of strs) {
    let ones = 0,
      zeros = 0;
    for (let ch of s) ch === "0" ? zeros++ : ones++;
    for (let j = m; j >= zeros; j--) {
      for (let k = n; k >= ones; k--) {
        dp[j][k] = Math.max(dp[j][k], dp[j - zeros][k - ones] + 1);
      }
    }
  }
  return dp[m][n];
};
```

#### [Coin Change](https://leetcode.com/problems/coin-change/)

fewest number of coins that you need to make up that amount, the number of each kind of coin is unlimited.

1. 完全背包

由于 coin 面值最小为 1，所以使用的 coin 总数不可能超过 amount，可以将 amount + 1 作为约束值。base case：当 amount = 0 时，dp[0...N][0] = 0，但额外写循环比较麻烦，可以在 j = w 时直接处理，即 `if (j === w) dp[i][j] = 1;`。

```js
var coinChange = function (coins, amount) {
  if (amount === 0) return 0;
  let N = coins.length,
    W = amount;
  const dp = [...Array(N + 1)].map(() => Array(W + 1).fill(amount + 1));

  for (let i = 1; i <= N; i++) {
    const w = coins[i - 1],
      v = 1;
    // amount 不足以放入 coin => 沿用子问题：只使用前 i-1 个 coin 的结果
    for (let j = 1; j < w; j++) {
      dp[i][j] = dp[i - 1][j];
    }
    // 可以放入当前 coin，一个或多个
    for (let j = w; j <= W; j++) {
      // 刚好可以放入 1 个
      if (j === w) dp[i][j] = 1;
      else {
        dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - w] + v);
      }
    }
  }

  return dp[N][W] === amount + 1 ? -1 : dp[N][W];
};
```

二维数组会导致 LC 爆堆。优化空间：

```js
var coinChange = function (coins, amount) {
  const dp = Array(amount + 1).fill(amount + 1);
  dp[0] = 0;
  for (let i = 0; i < coins.length; i++) {
    for (let j = coins[i]; j <= amount; j++) {
      dp[j] = Math.min(dp[j], dp[j - coins[i]] + 1);
    }
  }
  return dp[amount] === amount + 1 ? -1 : dp[amount];
};
```

2. dfs + memo

```js
var coinChange = function (coins, amount) {
  let map = {};
  const dp = (n) => {
    if (map[n]) return map[n];
    if (n === 0) return 0;
    if (n < 0) return -1;
    let min = Infinity;
    for (let coin of coins) {
      let curr = dp(n - coin);
      if (curr === -1) continue;
      min = Math.min(min, curr + 1);
    }
    map[n] = min === Infinity ? -1 : min;
    return map[n];
  };

  return dp(amount);
};
```

#### [Coin Change 2](https://leetcode.com/problems/coin-change-2/)

compute the number of combinations that make up that amount.

完全背包问题。

```js
// dp[i][j] 表示只使用前 i 种硬币，拼出总值 j 的方法数。
// 若当前 j 不可能放入第 i 种硬币，则 dp[i][j] = dp[i - 1][j];
// 若当前 j 可以放入第 i 种硬币，可以选择放或者不放。
// 不放的话 dp[i][j] = dp[i - 1][j];
// 放的话 dp[i][j] = dp[i][j-w]
// dp[i][j] 等于这两种情况的总和
var change = function (amount, coins) {
  if (amount === 0) return 1;
  let N = coins.length,
    W = amount;
  const dp = [...Array(N + 1)].map(() => Array(W + 1).fill(0));

  for (let i = 1; i <= N; i++) {
    const w = coins[i - 1];
    for (let j = 1; j < w; j++) {
      dp[i][j] = dp[i - 1][j];
    }
    for (let j = w; j <= W; j++) {
      if (j === w) dp[i][j] = dp[i - 1][j] + 1;
      // 手动处理 base case 引起的值更新
      else {
        dp[i][j] = dp[i - 1][j] + dp[i][j - w]; // 1. not use current kind of coin; 2. use current kind of coin.
      }
    }
  }

  return dp[N][W];
};
```

优化空间

```js
var change = function (amount, coins) {
  let dp = new Array(amount + 1).fill(0);
  dp[0] = 1; // 数组从二维变成一维，base case 可以比较简单地赋值。当 amount = 0 时，只有一种拼凑方法。
  for (let coin of coins) {
    for (let j = coin; j <= amount; j++) {
      dp[j] += dp[j - coin];
    }
  }
  return dp[amount];
};
```

#### [Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/)

positive numbers, no duplicates, number of combinations that add up to a positive integer target.

涉及顺序的完全背包问题。排列问题。

对于 coin change 2 来说，`dp[i][j]` 表示只使用前 i 种硬币，拼出总值 j 的方法数，即在处理 `dp[i][j]` 时只考虑用与不用第 i 种硬币，不能考虑用与不用前 i - 1 种硬币，否则情况会重复。比如：[1,2,2] [2,1,2] [2,2,1] 对于 coin change 2 来说表示的是同一种情况。coins 作为外层循环，保证了 coin 的拿取顺序不会重复。

但对于本题来说，coin 使用顺序不同，即使总和相同，也是两种不同的情况，都要被计算在内，因而计算结果时无需再刻意保证 coins 的拿取顺序，而要保证每次拿取时都是从全部 coins 种去选择。

抛开背包问题，只考虑题目自身。选择：对于 coins 里的每种 coin，是否使用。状态变量：总和。

dp[i] 表示用 coins 拼出总和为 i 的排列数。dp[0] = 1。

已知 dp[0...i-1] 求 dp[i]。计算每种选择下的排列数，并相加，即：判断是否可能放入 coins[0] (i >= coins[0])，若不能数量不增加，若能则增加 `dp[i-coins[0]]` 种排列；是否可能放入 coins[1]...。

```js
var combinationSum4 = function (nums, target) {
  const dp = Array(target + 1).fill(0);
  dp[0] = 1;
  for (let i = 1; i <= target; i++) {
    for (let num of nums) {
      if (i >= num) dp[i] += dp[i - num];
    }
  }
  return dp[target];
};
```

从递归角度思考：https://leetcode.com/problems/combination-sum-iv/discuss/85036/1ms-Java-DP-Solution-with-Detailed-Explanation

#### [Minimum Cost For Tickets](https://leetcode.com/problems/minimum-cost-for-tickets/)

Return the minimum number of dollars you need to travel every day in the given list of days.

coin change 变体。

dp[i] 表示前 i 天所需的最小花费，1 <= i <= last day in days array。对于每一天，分情况考虑：1) 该天不用出行 => 沿用前一天的开销。2) 该天需要出行，在三种续签方式中选最小值。需要注意的是回溯的天数必须在合法范围内。

```js
var mincostTickets = function (days, costs) {
  let travels = new Set(days),
    last = days[days.length - 1];
  let dp = Array(last + 1).fill(0);
  dp[0] = 0;
  for (let i = 1; i <= last; i++) {
    if (!travels.has(i)) dp[i] = dp[i - 1];
    else {
      let day = dp[i - 1] + costs[0],
        week = dp[Math.max(0, i - 7)] + costs[1],
        month = dp[Math.max(0, i - 30)] + costs[2];
      dp[i] = Math.min(day, week, month);
    }
  }
  return dp[last];
};
```

但这种方式需要一个 set 的额外空间来帮助查找。另一种方式：dp[i] 表示 days[0...i] 天所需的最小开销，1 <= i <= days.length。对于每种续签方式，回溯找到 days 中距离这种方式最近的日期，再在三者中取最小值。回溯找到 days 中距离这种方式最近的日期 => 隐式规避了处理不出行的那些日子，即 `dp[i] = dp[i - 1];`。

```js
var mincostTickets = function (days, costs) {
  let n = days.length,
    dp = Array(n).fill(Infinity),
    passes = [1, 7, 30];
  dp[0] = Math.min(...costs); // base case: days[0] 第一天的最小开销应该是三种方式中花费最少的那种
  for (let i = 1; i <= n; i++) {
    let j = i;
    for (let k = 0; k < 3; k++) {
      while (j >= 0 && days[j] > days[i] - passes[k]) j--;
      dp[i] = Math.min(dp[i], dp[Math.max(0, j)] + costs[k]); // j 可能越界
    }
  }
  return dp[n - 1];
};
```

[Note:如何进一步压缩空间](https://leetcode.com/problems/minimum-cost-for-tickets/discuss/226659/Two-DP-solutions-with-pictures)

#### [Perfect Squares](https://leetcode.com/problems/perfect-squares/)

Given an integer n, return the least number of perfect square numbers that sum to n.

完全背包问题。可选择的“物品”根据条件为：j 从 1 开始，同时 `j * j <= i`。

```js
var numSquares = function (n) {
  const dp = Array(n + 1).fill(n + 1);
  dp[0] = 0;
  for (let i = 1; i <= n; i++) {
    for (let j = 1; j * j <= i; j++) {
      dp[i] = Math.min(dp[i], 1 + dp[i - j * j]);
    }
  }
  return dp[n];
};
```

可以先把可选择的“物品”先算出来，避免内层循环重复计算乘方。

```js
var numSquares = function (n) {
  let squares = [],
    dp = Array(n + 1).fill(n + 1);
  dp[0] = 0;
  let boundary = Math.sqrt(n);
  for (let i = 1; i <= boundary; i++) {
    let product = i * i;
    if (n === product) return 1;
    squares.push(product);
    dp[product] = 1;
  }
  for (let i = 2; i <= n; i++) {
    for (let square of squares) {
      if (square <= i) {
        dp[i] = Math.min(dp[i], dp[i - square] + 1);
      }
    }
  }
  return dp[n];
};
```

### Longest Increasing Subsequence

单个序列，求满足某条件的最长子序列长度 / 序列本身 / 数量。

#### [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

dp[i] 表示以 nums[i] 为结尾的最长递增子序列长度。

```js
var lengthOfLIS = function (nums) {
  let dp = Array(nums.length).fill(1),
    max = 1;
  for (let i = 0; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
    max = Math.max(max, dp[i]);
  }
  return max;
};
```

TODO: [nlogn 解法 binarySearch + patience sorting](https://github.com/CyC2018/CS-Notes/blob/master/notes/Leetcode%20%E9%A2%98%E8%A7%A3%20-%20%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92.md#1-%E6%9C%80%E9%95%BF%E9%80%92%E5%A2%9E%E5%AD%90%E5%BA%8F%E5%88%97)

#### [Largest Divisible Subset](https://leetcode.com/problems/largest-divisible-subset/)

Given a set of _distinct_ positive integers nums, return the largest subset answer such that every pair (answer[i], answer[j]) of elements in this subset satisfies: answer[i] % answer[j] == 0, or answer[j] % answer[i] == 0. Any valid solution is okay.

LIS 变体。求满足条件的最长子序列本身。

dp[i] 表示以 nums[i] 为结尾的子序列所对应的满足条件的最大值。条件从递增变为满足 nums[i] % nums[j] === 0。

如何保证当 nums[i] % nums[j] === 0 时，nums[i] 对 dp[j] 所代表的子序列中的每个元素仍能满足取余为 0？=> 保证序列是递增的。

如何记下满足条件的子序列？=> 记录最大值对应的子序列结尾 idx，并通过一个 prev 数组记录该 idx 对应的前序元素序号。

```js
var largestDivisibleSubset = function (nums) {
  nums.sort((a, b) => a - b);

  let n = nums.length,
    dp = Array(n).fill(1),
    prev = Array(n).fill(-1);
  let max = 1,
    maxIdx = 0;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] % nums[j] === 0 && dp[j] + 1 > dp[i]) {
        dp[i] = dp[j] + 1;
        prev[i] = j;
      }
    }
    if (dp[i] > max) {
      max = dp[i];
      maxIdx = i;
    }
  }

  let res = [];
  while (maxIdx >= 0) {
    res.push(nums[maxIdx]);
    maxIdx = prev[maxIdx];
  }
  return res;
};
```

#### [Number of Longest Increasing Subsequence](https://leetcode.com/problems/number-of-longest-increasing-subsequence/)

LIS 变体。求满足条件的最长子序列数量。

需要额外的 counts 数组来记录以 nums[i] 为结尾的最长子序列的数量。

当 nums[i] > nums[j] 时，需要考虑两种情况：1) lengths[j] + 1 == lengths[i] => 则 counts[i] 需要加上 counts[j]; 2) lengths[j] + 1 > lengths[i] => 说明能够构造出更长的子序列，counts[i] = counts[j]。

如果 lengths[i] 超过了当前记录的最大值，需要重置 max & res。如果等于 max，需要把当前的数量累加到 res 中。

```js
var findNumberOfLIS = function (nums) {
  if (nums.length <= 1) return nums.length;
  let lengths = Array(nums.length).fill(1),
    counts = Array(nums.length).fill(1),
    res = 1,
    max = 1;
  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        if (lengths[j] + 1 === lengths[i]) {
          counts[i] += counts[j];
        } else if (lengths[j] + 1 > lengths[i]) {
          lengths[i] = lengths[j] + 1;
          counts[i] = counts[j];
        }
      }
    }
    if (lengths[i] === max) {
      res += counts[i];
    } else if (lengths[i] > max) {
      res = counts[i];
      max = lengths[i];
    }
  }
  return res;
};
```

#### [Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain/)

Given a set of pairs, find the length longest chain which can be formed. In every pair, the first number is always smaller than the second number.

LIS 变体。满足条件改变。

dp[i] 表示只使用前 i 个 pair 所能得到的最长链。条件从递增变为满足 pairs[j][1] < pairs[i][0]。

```js
var findLongestChain = function (pairs) {
  if (!pairs) return 0;
  if (pairs.length === 0 || pairs.length === 1) return 1;
  pairs.sort((a, b) => a[0] - b[0]);
  let n = pairs.length,
    dp = Array(n).fill(1),
    max = 0;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (pairs[j][1] < pairs[i][0]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
    max = Math.max(dp[i], max);
  }
  return max;
};
```

贪心解法 TODO: 为什么可以用贪心？

```js
var findLongestChain = function (pairs) {
  if (!pairs) return 0;
  let curr = -Infinity,
    res = 0;
  pairs.sort((a, b) => a[1] - b[1]); // 注意这里是依据 pair 的第二个数进行排序，前一种解法是通过第一个数。
  for (let [x, y] of pairs) {
    if (curr < x) {
      curr = y;
      res++;
    }
  }
  return res;
};
```

#### [Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/)

Return the maximum number of envelopes you can Russian doll (i.e., put one inside the other). Note: You cannot rotate an envelope.

LIS 变体。满足条件改变。

```js
var maxEnvelopes = function (envelopes) {
  if (envelopes.length === 0) return 0;
  if (envelopes.length === 1) return 1;
  envelopes.sort((a, b) => a[0] - b[0]);
  let n = envelopes.length,
    dp = Array(n).fill(1),
    max = 1;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (
        envelopes[j][1] < envelopes[i][1] &&
        envelopes[j][0] < envelopes[i][0]
      ) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
    max = Math.max(max, dp[i]);
  }
  return max;
};
```

#### [Longest String Chain](https://leetcode.com/problems/longest-string-chain/)

Return the longest possible length of a word chain. word1 is a predecessor of word2 if and only if we can add exactly one letter anywhere in word1 to make it equal to word2.

LIS 变体。满足条件改变。

```js
var longestStrChain = function (words) {
  if (words.length <= 1) return words.length;
  words.sort((a, b) => a.length - b.length);
  let n = words.length,
    dp = Array(n).fill(1),
    max = 1;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (canAddOne(words[j], words[i])) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
    max = Math.max(max, dp[i]);
  }
  return max;
};

function canAddOne(a, b) {
  if (a.length + 1 !== b.length) return false;
  let i = 0,
    j = 0;
  while (i < a.length && j < b.length) {
    if (a[i] === b[j]) {
      i++;
      j++;
    } else {
      j++;
    }
  }
  return i === a.length;
}
```

#### [Delete and Earn](https://leetcode.com/problems/delete-and-earn/)

Start with 0 points. Return the maximum number of points you can earn by applying such operations.

dp[i] 表示使用 nums[0...i] 能得到的最高分值。则对于 dp[i]，可以考虑使用或者不使用 nums[j] != nums[i] - 1 的那些数字，不使用则保留 dp[i]，使用则有 dp[j] + nums[i]，在两者中取较大值。需要先排序来保证只对比 nums[j] 能覆盖 j 之前的子问题，并且不需要再考虑 nums[i] + 1 的情况。

```js
var deleteAndEarn = function (nums) {
  if (nums.length === 0) return 0;
  if (nums.length === 1) return nums[0];
  nums.sort((a, b) => a - b);
  let n = nums.length,
    dp = Array(n).fill(1),
    max = 0;
  dp[0] = nums[0];
  for (let i = 1; i < n; i++) {
    dp[i] = nums[i];
    for (let j = 0; j < i; j++) {
      if (nums[j] !== nums[i] - 1) {
        dp[i] = Math.max(dp[i], dp[j] + nums[i]);
      }
    }
    max = Math.max(max, dp[i]);
  }
  return max;
};
```

另一种思路：利用 constraints 约束的数字范围。dp[i] 表示使用数字 0 - i 得到的最高分数，则选择有：1) 不使用数字 i，使用 [0...i-1]；2) 使用数字 i，则能得到的值为 `dp[i-2] + counts[i] * i` 出现的次数。状态转移方程：`dp[i] = Math.max(dp[i-1], dp[i-2] + counts[i] * i)`。

```js
var deleteAndEarn = function (nums) {
  let n = 10001,
    counts = Array(n).fill(0);
  for (let num of nums) counts[num]++;
  let dp = Array(n).fill(0);
  dp[1] = counts[1];
  for (let i = 2; i < n; i++) {
    let points = counts[i] * i;
    dp[i] = Math.max(dp[i - 1], dp[i - 2] + points);
  }
  return dp[n - 1];
};
```

#### [Arithmetic Slices](https://leetcode.com/problems/arithmetic-slices/)

Given an integer array nums, return the number of arithmetic subarrays(a contiguous subsequence) of nums.

dp[i] 表示以 nums[i] 为结尾的满足条件的 **子数组** 数。由于连续，即 dp[i] 仅与 dp[i-1] 相关，可以省略至只用一个变量 dp 来维护。

以 [1,2,3,4] 为例。dp[2] = 1，代表序列 [1,2,3]；dp[3] = dp[2] + 1 => [1,2,3][4] + [2,3,4]。故 sum 需要累加 dp[i]。

```js
var numberOfArithmeticSlices = function (A) {
  var dp = 0,
    sum = 0;
  for (let i = 2; i < A.length; i++) {
    if (A[i] - A[i - 1] === A[i - 1] - A[i - 2]) {
      dp = dp + 1;
      sum += dp;
    } else dp = 0;
  }
  return sum;
};
```

#### [Arithmetic Slices II - Subsequence](https://leetcode.com/problems/arithmetic-slices-ii-subsequence/)

Given an integer array nums, return the number of all the arithmetic subsequences of nums.

`dp[i][diff]` 表示以 nums[i] 为结尾的，各元素间差值为 diff 的序列数。很自然地，长度 >= 2 才能构成序列。以 [2, 4, 6, 1, 8] 为例：

i = 2, j = 1 时，diff = nums[i] - nums[j] = 2。对于 `dp[i][diff]` 产生了一个新序列：`[nums[j], nums[i]]`，即 [4,6]。此外，还可能由 `dp[j][diff]` 附加上当前元素 nums[i] 产生更多子序列：即 dp[j] = {2:1} => 序列 [2,4] => [2,4,6]。得到最终 dp[i] = {2:2} => [4,6] & [2,4,6]。

由于 diff = nums[i] - nums[j] 的存在，`dp[j][diff]` 所代表的所有序列均成为了合法序列，可以加入 res 中。

```js
var numberOfArithmeticSlices = function (A) {
  let res = 0,
    dp = new Array(A.length);

  for (let i = 0; i < A.length; i++) {
    dp[i] = new Map();
    for (let j = 0; j < i; j++) {
      let diff = A[i] - A[j];
      let ci = dp[i].get(diff) || 0;
      let cj = dp[j].get(diff) || 0;
      res += cj;
      dp[i].set(diff, ci + cj + 1); // 1 for sequence [A[j], A[i]]
    }
  }

  return res;
};
```

[dp + hashmap](<https://leetcode.com/problems/arithmetic-slices-ii-subsequence/discuss/92822/Detailed-explanation-for-Java-O(n2)-solution>)

#### [Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/)

```js
var increasingTriplet = function (nums) {
  let n = nums.length,
    dp = Array(n).fill(1);
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
        if (dp[i] === 3) return true;
      }
    }
  }
  return false;
};
```

Follow up: O(n) time complexity and O(1) space complexity?

```js
var increasingTriplet = function (nums) {
  let n = nums.length,
    first = Infinity,
    sec = Infinity;
  for (let n of nums) {
    if (n <= first) first = n;
    else if (n <= sec) sec = n;
    else return true;
  }
  return false;
};
```

### Longest Common Subsequence

求 LCS 的长度 / 本身 / 个数。使两字符串变得一致的操作数。

Subsequence：字符相等和不相等两种情况均可以扩展 `dp[i-1][j-1]`; `dp[i-1][j]`, `dp[i][j-1]` => `dp[i][j]`。

Substring: 只有字符相等时才可以扩展 `dp[i-1][j-1]` => `dp[i][j]`。

#### [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

`dp[i][j]` 表示 S1 的前 i 个字符与 S2 的前 j 个字符间最长公共子序列的长度。

base case: `dp[0][j]` or `dp[i][0]` => 0。

状态：

- 若 `S1[i] === S2[j]`，表示可以纳入公共子序列进行计算，则 `dp[i][j] = dp[i-1][j-1] + 1`；

- 若 `S1[i] !== S2[j]`，则从 `dp[i-1][j]` & `dp[i][j-1]` 取较大值。

  - 为什么不直接在 `dp[i-1][j]`，`dp[i][j-1]`，`dp[i-1][j-1]` + 1 间取最大值？=> 因为 S1[i] 与 S2[j] 要么相等要么不想等，不可能既相等又不相等，`dp[i-1][j-1]` + 1 就表示 S1[i] 与 S2[j] 相等。这与背包问题在每一步即可以选择拿也可以选择不拿是不同的。

  - 为什么 S1[i] !== S2[j] 时，不在 `dp[i-1][j]`，`dp[i][j-1]`，`dp[i-1][j-1]` 里取最大值？=> 根据 dp 定义，`dp[i-1][j-1]` 只可能 <= `dp[i-1][j]`，后者比前者可能多纳入 lsc 一个字符，故 max 不可能取自 `dp[i-1][j-1]`。

与最长递增子序列相比，最长公共子序列有以下不同点：

- 最长递增子序列只针对单个序列；最长公共子序列针对的是两个序列。

- 在最长递增子序列中，dp[i] 表示以 S[i] 为结尾的最长递增子序列长度，子序列必须包含 S[i]；在最长公共子序列中，`dp[i][j]` 表示 S1 中前 i 个字符与 S2 中前 j 个字符的最长公共子序列长度，不一定包含 S1[i] 和 S2[j]。（因为当前的最大值可能来自于 `dp[i-1][j]` 或 `dp[i][j-1]`）

- 最长递增子序列中 dp[n] 不是最终解，因为以 Sn 为结尾的最长递增子序列不一定是整个序列最长递增子序列，需要遍历一遍 dp 数组找到最大者；而对于最长公共子序列，dp[n][m] 就是最终解。

- 注意 i 和 j 应用到 s 和 t 上时需要减 1。

```js
var longestCommonSubsequence = function (s, t) {
  let m = s.length,
    n = t.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === t[j - 1]) {
        dp[i][j] = 1 + dp[i - 1][j - 1];
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }

  return dp[m][n];
};
```

压缩空间：m x n => 2 x n

```js
var longestCommonSubsequence = function (s, t) {
  let m = s.length,
    n = t.length,
    dp = [...Array(2)].map(() => Array(n + 1).fill(0));

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === t[j - 1]) {
        dp[i % 2][j] = 1 + dp[(i - 1) % 2][j - 1];
      } else {
        dp[i % 2][j] = Math.max(dp[(i - 1) % 2][j], dp[i % 2][j - 1]);
      }
    }
  }

  return dp[m % 2][n];
};
```

#### [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)

```js
var longestPalindromeSubseq = function (s) {
  return longestCommonSubsequence(s, s.split("").reverse().join(""));
};
```

#### [Shortest Common Supersequence](https://leetcode.com/problems/shortest-common-supersequence/)

LCS 变体 + 求 LCS 本身。找到 LCS 后加上各自不同的字符。

```js
var shortestCommonSupersequence = function (s1, s2) {
  let lcs = longestCommonSequence(s1, s2);

  let res = "",
    i = 0,
    j = 0;
  for (let ch of lcs) {
    while (s1[i] !== ch) res += s1[i++];
    while (s2[j] !== ch) res += s2[j++];

    res += ch;
    i++;
    j++;
  }

  return res + s1.slice(i) + s2.slice(j);
};

var longestCommonSequence = function (s1, s2) {
  let m = s1.length,
    n = s2.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(""));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s1[i - 1] === s2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + s1[i - 1];
      } else {
        dp[i][j] =
          dp[i - 1][j].length > dp[i][j - 1].length
            ? dp[i - 1][j]
            : dp[i][j - 1];
      }
    }
  }
  return dp[m][n];
};
```

#### [Longest Common Substring](https://www.geeksforgeeks.org/longest-common-substring-dp-29/)

由于是 substring，为保证 i - 1 => i 的连续，只有当 s[i] === t[j] 时，才应该更新 dp[i][j] 的值，而最大值需要在迭代过程中不断更新。

```js
var longestCommonSubstring = function (s, t) {
  let m = s.length,
    n = t.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0)),
    max = 0;

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === t[j - 1]) {
        dp[i][j] = 1 + dp[i - 1][j - 1];
      }
      max = Math.max(max, dp[i][j]);
    }
  }

  return max;
};
```

#### [Longest Repeating Subsequence](https://www.geeksforgeeks.org/longest-repeating-subsequence/)

Find the LCS(str, str), where str is the input string and with the restriction that when both the characters are same, they shouldn’t be on the same index in the two strings.

```js
function longestRepeatingSubsequence(s) {
  let n = s.length,
    dp = [...Array(n + 1)].map(() => Array(n + 1).fill(0));
  for (let i = 1; i <= n; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === s[j - 1] && i !== j) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[n][n];
}
```

#### [Longest Repeating Substring](https://www.geeksforgeeks.org/longest-repeating-and-non-overlapping-substring/)

```js
function longestRepeatingSubsequence(s) {
  let n = s.length,
    dp = [...Array(n + 1)].map(() => Array(n + 1).fill("")),
    max = "";
  for (let i = 1; i <= n; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === s[j - 1] && i !== j) {
        dp[i][j] = dp[i - 1][j - 1] + s[i - 1];
      }
      max = dp[i][j].length > max.length ? dp[i][j] : max;
    }
  }
  return max;
}
```

#### [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/)

the number of distinct subsequences of s which equals t, s => t.

dp[i][j] 表示 s[0...i] 与 t[0...j] 间合法的子序列数。

base case

- '' => '' = 1
- 's' => '' = 1（空串是所有字符串子串）
- '' => 't' = 0（如果 s 为空串则不可能有子序列能无中生有出 t 字符串）

general case:

- s[i] !== t[j]，则 s[i] 对构成 subsequence 没有帮助，只能选择不使用 s[i]，向子问题靠拢：dp[i][j] = dp[i-1][j]；

- s[i] === t[j]，则有两种选择：1) 使用 s[i]，取 dp[i-1][j-1]，因为要保证 s[i] & t[j] 未被考虑过以免重复；2) 不使用 s[i]，取 dp[i-1][j]。序列数为两种情况的和。

为什么 s[i] === t[j]，不使用 s[i] 时不需要 dp[i-1][j]？因为是以 t 为目标去比对。

```js
var numDistinct = function (s, t) {
  let m = s.length,
    n = t.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) dp[i][0] = 1;
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === t[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
      } else {
        dp[i][j] = dp[i - 1][j];
      }
    }
  }
  return dp[m][n];
};
```

#### [Edit Distance](https://leetcode.com/problems/edit-distance/)

Given two strings word1 and word2, return the minimum number of operations required to convert word1 to word2. Can have 3 operations permitted on a word: insert, delete, replace a character.

base case: 另一方为空串，dp[i][0] / dp[0][i] 表示与空串之间的操作数，等于当前字符串长度。

general case：

- word1[i-1] === word2[j-1]，则表示不需要进行操作，沿用之前的值 dp[i][j] = dp[i-1][j-1]；

- word1[i-1] !== word2[j-1]：

  - replace word1[i-1] with word2[j-1]: dp[i][j] = dp[i-1][j-1] + 1

  - delete word1[i-1]: dp[i][j] = dp[i-1][j] + 1

  - add word2[j-1]: dp[i][j] = dp[i][j-1] + 1

  - find min in above

```js
var minDistance = function (word1, word2) {
  let m = word1.length,
    n = word2.length;
  let dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) {
    dp[i][0] = i;
  }
  for (let i = 0; i <= n; i++) {
    dp[0][i] = i;
  }
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = Math.min(dp[i - 1][j - 1], dp[i][j - 1], dp[i - 1][j]) + 1;
      }
    }
  }
  return dp[m][n];
};
```

压缩空间

```js
var minDistance = function (word1, word2) {
  let m = word1.length,
    n = word2.length;
  let dp = [...Array(2)].map(() => Array(n + 1).fill(0));
  for (let i = 1; i <= n; i++) {
    dp[0][i] = i;
  }
  for (let i = 1; i <= m; i++) {
    dp[i % 2][0] = i;
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i % 2][j] = dp[(i - 1) % 2][j - 1];
      } else {
        dp[i % 2][j] =
          Math.min(
            dp[(i - 1) % 2][j - 1],
            dp[i % 2][j - 1],
            dp[(i - 1) % 2][j]
          ) + 1;
      }
    }
  }
  return dp[m % 2][n];
};
```

#### [Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/)

min operations = len(s) + len(t) - 2 \* len(lcs)。

```js
var minDistance = function (word1, word2) {
  let m = word1.length,
    n = word2.length;
  let dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }

  return m + n - 2 * dp[m][n];
};
```

or directly:

```js
var minDistance = function (word1, word2) {
  let m = word1.length,
    n = word2.length;
  let dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    dp[i][0] = i;
  }
  for (let j = 1; j <= n; j++) {
    dp[0][j] = j;
  }
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        // no deletion
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        // delete either word1[i-1] or word2[j-1]
        dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + 1;
      }
    }
  }

  return dp[m][n];
};
```

#### [Minimum ASCII Delete Sum for Two Strings](https://leetcode.com/problems/minimum-ascii-delete-sum-for-two-strings/)

从求 longest common subsequence 转变成求 common subsequence with largest ascii code sum。

```js
var minimumDeleteSum = function (s1, s2) {
  let m = s1.length,
    n = s2.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s1[i - 1] === s2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + s1[i - 1].charCodeAt();
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return codeSum(s1) + codeSum(s2) - 2 * dp[m][n];
};

function codeSum(s) {
  return Array.from(s).reduce((sum, ch) => sum + ch.charCodeAt(), 0);
}
```

直接求 min sum

```js
var minimumDeleteSum = function (s1, s2) {
  let m = s1.length,
    n = s2.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    dp[i][0] = dp[i - 1][0] + s1.charCodeAt(i - 1);
  }
  for (let j = 1; j <= n; j++) {
    dp[0][j] = dp[0][j - 1] + s2.charCodeAt(j - 1);
  }
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s1[i - 1] === s2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = Math.min(
          dp[i - 1][j] + s1.charCodeAt(i - 1),
          dp[i][j - 1] + s2.charCodeAt(j - 1)
        );
      }
    }
  }
  return dp[m][n];
};
```

#### [Interleaving String](https://leetcode.com/problems/interleaving-string/)

DFS + memo: i, j, k for s1, s2, s3; memo for i, j combinations.

```js
var isInterleave = function (s1, s2, s3) {
  if (s1.length + s2.length !== s3.length) return false;
  let map = new Map();
  const key = (i, j) => i + "," + j;
  const dfs = (i, j, k) => {
    if (i === s1.length) return s2.slice(j) === s3.slice(k);
    if (j === s2.length) return s1.slice(i) === s3.slice(k);
    if (map.has(key(i, j))) return map.get(key(i, j));
    let res = false;
    if (
      (s1[i] === s3[k] && dfs(i + 1, j, k + 1)) ||
      (s2[j] === s3[k] && dfs(i, j + 1, k + 1))
    ) {
      res = true;
    }
    map.set(key(i, j), res);
    return res;
  };
  return dfs(0, 0, 0);
};
```

DP: i for s1, j for s2, i + j for s3.

```js
var isInterleave = function (s1, s2, s3) {
  if (s1.length + s2.length !== s3.length) return false;
  let m = s1.length,
    n = s2.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(false));
  for (let i = 0; i <= m; i++) {
    for (let j = 0; j <= n; j++) {
      if (i === 0 && j === 0) {
        dp[i][j] = true;
      } else if (i === 0) {
        dp[i][j] = dp[i][j - 1] && s2[j - 1] === s3[i + j - 1];
      } else if (j === 0) {
        dp[i][j] = dp[i - 1][j] && s1[i - 1] === s3[i + j - 1];
      } else {
        dp[i][j] =
          (dp[i - 1][j] && s1[i - 1] === s3[i + j - 1]) ||
          (dp[i][j - 1] && s2[j - 1] === s3[i + j - 1]);
      }
    }
  }
  return dp[m][n];
};
```

DP(optimize space): 只与 i - 1 和 j - 1 有关。

```js
var isInterleave = function (s1, s2, s3) {
  if (s1.length + s2.length !== s3.length) return false;
  let m = s1.length,
    n = s2.length,
    dp = Array(n + 1).fill(false);
  for (let i = 0; i <= m; i++) {
    for (let j = 0; j <= n; j++) {
      if (i === 0 && j === 0) {
        dp[j] = true;
      } else if (i === 0) {
        dp[j] = dp[j - 1] && s2[j - 1] === s3[i + j - 1];
      } else if (j === 0) {
        dp[j] = dp[j] && s1[i - 1] === s3[i + j - 1];
      } else {
        dp[j] =
          (dp[j] && s1[i - 1] === s3[i + j - 1]) ||
          (dp[j - 1] && s2[j - 1] === s3[i + j - 1]);
      }
    }
  }
  return dp[n];
};
```

### Palindrome String

#### [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)

Given a string s, return the longest palindromic substring in s.

dp[i][j] 表示 s(i, j) 是回文串，s(i, j) 为回文串的条件：

- i === j，比如 'a'；

- i + 1 = j (len = 2) 或 i + 2 = j (len = 3) 时，s[i] === s[j]。比如 'aa' & 'aba'；

- j - i >= 3 (len >= 4)，s[i] === s[j] 且 s(i+1, j-1) 为回文串。比如 'abba'。

由于 s(i, j) 依赖于 s(i+1, j-1)，循环时要先求 i + 1，j - 1，所以 i 需要递减，j 需要递增。又因为 i 表示 start，j 表示 end，j >= i，所以 j 的循环范围为 [i, n-1]。

https://leetcode.com/problems/longest-palindromic-substring/discuss/151144/Bottom-up-DP-Logical-Thinking

```js
var longestPalindrome = function (s) {
  let n = s.length,
    dp = [...Array(n)].map(() => Array(n).fill(false)),
    res = "";
  for (let i = n - 1; i >= 0; i--) {
    for (let j = i; j < n; j++) {
      dp[i][j] = s.charAt(i) === s.charAt(j) && (j - i < 3 || dp[i + 1][j - 1]);
      if (dp[i][j] && j - i + 1 > res.length) {
        res = s.substring(i, j + 1);
      }
    }
  }
  return res;
};
```

非 DP 解法：从每个字符出发，扩展直到不符合回文串的条件为止。

len = (end-1) - (start+1) - 1 = start - end - 1

end - 1 & start + 1: 因为 end & start 各超出了一个字符，idx 需要回移。

```js
var longestPalindrome = function (s) {
  if (s.length < 2) return s;

  let res = "";
  const expand = (start, end) => {
    while (start >= 0 && end < s.length && s[start] === s[end]) {
      start--;
      end++;
    }
    if (res.length < end - start - 1) {
      res = s.slice(start + 1, end);
    }
  };
  for (let i = 0; i < s.length; i++) {
    expand(i, i);
    expand(i, i + 1);
  }
  return res;
};
```

#### [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/)

求回文子串数量。

DP 解法

```js
var countSubstrings = function (s) {
  let n = s.length,
    dp = [...Array(n)].map(() => Array(n).fill(false)),
    res = 0;
  for (let i = n - 1; i >= 0; i--) {
    for (let j = i; j < n; j++) {
      dp[i][j] = s.charAt(i) === s.charAt(j) && (j - i < 3 || dp[i + 1][j - 1]);
      if (dp[i][j]) {
        res++;
      }
    }
  }
  return res;
};
```

非 DP 解法

```js
var countSubstrings = function (s) {
  if (s === null || s.length < 1) return 0;

  let count = 0;
  const expand = (start, end) => {
    while (start >= 0 && end < s.length && s[start] === s[end]) {
      count++;
      start--;
      end++;
    }
  };
  for (let i = 0; i < s.length; i++) {
    expand(i, i);
    expand(i, i + 1);
  }
  return count;
};
```

#### [Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/)

Given a string s, partition s such that every substring of the partition is a palindrome. Return the minimum cuts needed.

cuts[i] 表示从第一个字符开始，长度为 i 的子串所需要的最小分割数。

对于每个字符，分别进行以 s[i] 为中心的扩展（奇数长度），和以 s[i] & s[i+1] 为中心的扩展（偶数长度）。

```js
var minCut = function (s) {
  let n = s.length,
    cuts = [...Array(n + 1)].fill(0);
  for (let i = 0; i <= n; i++) cuts[i] = i - 1; // can at most cut i - 1 times for a string with length i.
  for (let center = 0; center < n; center++) {
    for (
      let gap = 0;
      center - gap >= 0 &&
      center + gap < n &&
      s[center - gap] === s[center + gap];
      gap++
    ) {
      // center + gap => new idx, len = idx + 1, cuts[len] = cuts[center + gap + 1]
      cuts[center + gap + 1] = Math.min(
        cuts[center + gap + 1],
        cuts[center - gap] + 1
      );
      // use previous cuts number of s[0...center - gap - 1] => cuts[center - gap - 1 + 1] + 1
      // + 1 for s[center - gap, center + gap]
    }
    for (
      let gap = 1;
      center - gap + 1 >= 0 &&
      center + gap < n &&
      s[center - gap + 1] === s[center + gap];
      gap++
    ) {
      // center - gap + 1, +1 for center itself
      cuts[center + gap + 1] = Math.min(
        cuts[center + gap + 1],
        cuts[center - gap + 1] + 1
      );
      // use previous cuts number of s[0...center - gap + 1 - 1] = s[0...center - gap] => cuts[center - gap + 1] + 1
      // + 1 for s[center - gap + 1, center + gap]
    }
  }
  return cuts[n];
};
```

### House Robber

#### [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

```js
var climbStairs = function (n) {
  let dp = [0, 1, 2];
  for (let i = 3; i <= n; i++) {
    dp[i] = dp[i - 2] + dp[i - 1];
  }
  return dp[n];
};
```

#### [House Robber](https://leetcode.com/problems/house-robber/)

Return the maximum amount of money you can rob. Cannot break into two adjacent houses on the same night.

```js
var rob = function (nums) {
  if (nums.length === 0) return 0;
  if (nums.length === 1) return nums[0];

  let n = nums.length;
  let dp = new Array(n).fill(0);
  dp[0] = nums[0];
  dp[1] = nums[0] > nums[1] ? nums[0] : nums[1];
  for (let i = 2; i <= n; i++) {
    dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
  }

  return dp[n - 1];
};
```

optimize space: 只与 i - 1 和 i - 2 有关。

```js
var rob = function (nums) {
  let pre1 = 0,
    pre2 = 0;
  for (let i = 0; i < nums.length; i++) {
    let curr = Math.max(pre1, pre2 + nums[i]);
    pre2 = pre1;
    pre1 = curr;
  }
  return pre1;
};
```

#### [House Robber II](https://leetcode.com/problems/house-robber-ii/)

All houses at this place are arranged in a circle.

say we have house 1 - 9, divide to two sub-problems: rob 1-8 & rob 2-9, since 1 & 9 are adjacent.

```js
var rob = function (nums) {
  if (nums.length === 1) return nums[0];
  let n = nums.length;
  const robFrom = (start, end) => {
    let pre1 = 0,
      pre2 = 0;
    for (let i = start; i < end; i++) {
      let curr = Math.max(pre1, pre2 + nums[i]);
      pre2 = pre1;
      pre1 = curr;
    }
    return pre1;
  };
  return Math.max(robFrom(0, n - 1), robFrom(1, n));
};
```

#### [House Robber III](https://leetcode.com/problems/house-robber-iii/)

Given the root of the binary tree.

```js
var rob = function (root) {
  function robIn(node) {
    if (!node) return [0, 0];

    var left = robIn(node.left);
    var right = robIn(node.right);

    return [
      Math.max(...left) + Math.max(...right), // skip current & (might) use next level.
      node.val + left[0] + right[0], // use current & skip next level.
    ];
  }

  return Math.max(...robIn(root));
};
```

### Sell Stock

#### [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

Choosing a single day to buy one stock and choosing a different day in the future to sell that stock. Return the maximum profit.

思路一：对于每个时间点，找到其后比它大的每一天并计算差值，比较得到最大。O(N^2)

思路二：计算差值，差值序列中和最大的子数组。O(N)

```js
var maxProfit = function (prices) {
  let max = 0,
    maxCurr = 0;
  for (let i = 1; i < prices.length; i++) {
    maxCurr += prices[i] - prices[i - 1];
    maxCurr = Math.max(0, maxCurr);
    max = Math.max(max, maxCurr);
  }

  return max;
};
```

Why sum of diff sub-array works:

Suppose we have original array: [a0, a1, a2, a3, a4, a5, a6], and its diff array [b0, b1, b2, b3, b4, b5, b6], where b[i] = 0, when i == 0, and b[i] = a[i] - a[i - 1], when i != 0.

Suppose if a2 and a6 are the points that give us the max difference (a2 < a6), then in our given array, we need to find the sum of sub array from b3 to b6: b3 = a3 - a2, ..., b6 = a6 - a5.

Adding all these, all the middle terms will cancel out except two: b3 + b4 + b5 + b6 = a6 - a2, and a6 - a2 is the required solution.

#### [Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

Find the maximum profit. Transaction times is not limited.

add all non-zero values in diff sub-array.

```js
var maxProfit = function (prices) {
  let max = 0;
  for (let i = 1; i < prices.length; i++) {
    max += Math.max(0, prices[i] - prices[i - 1]);
  }
  return max;
};
```

#### [Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions: cannot buy stock on the next day (i.e., cooldown one day).

state machine: buy => sell => cool down => buy

DP:

- `buy[i] = max{ buy[i-1], sell[i-2] - prices[i] }`

  - 选择不交易：沿用 buy[i-1]

  - 在这一天买入：sell[i-2] - prices[i] (sell[i-2] for one day cooldown)

- `sell[i] = max{ sell[i-1], buy[i-1] + prices[i] }`

  - 选择不交易：沿用 sell[i-1]

  - 在这一天卖出：buy[i-1] + prices[i]

- base case

  - for buy: since only need i & i-1: buy[-1] = -prices[0], buy[0] = -prices[0].

  - for sell: need i-2 and since nothing to sell without buy in first: sell[-2] = 0, sell[-1] = 0, sell[0] = 0.

```js
var maxProfit = function (prices) {
  if (!prices || prices.length === 0) return 0;

  let b1 = -prices[0],
    b0 = b1, // i-1, i
    s2 = 0,
    s1 = 0,
    s0 = 0; // i-2, i-1, i

  for (let i = 1; i < prices.length; i++) {
    b0 = Math.max(b1, s2 - prices[i]);
    s0 = Math.max(s1, b1 + prices[i]);
    b1 = b0;
    s2 = s1;
    s1 = s0;
  }

  return s0;
};
```

#### [Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

DP:

- buy[i] = max{ buy[i-1], sell[i-1] - prices[i] }

- sell[i] = max{ sell[i-1], buy[i-1] + prices[i] - fee }

  - 选择不交易：sell[i-1]: choose not to operate, rest 1 day

  - 在这一天卖出：buy[i-1] + prices[i] - fee

```js
var maxProfit = function (prices, fee) {
  if (!prices || prices.length === 0) return 0;

  let b1 = -prices[0],
    b0 = b1,
    s1 = 0,
    s0 = 0;

  for (let i = 1; i < prices.length; i++) {
    b0 = Math.max(b1, s1 - prices[i]);
    s0 = Math.max(s1, b1 + prices[i] - fee);
    b1 = b0;
    s1 = s0;
  }

  return s0;
};
```

_Note: could also pay transaction fee when buy in._

#### [Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)

You may complete at most 2 transactions.

```js
var maxProfit = function (prices) {
  let buy1 = Infinity,
    buy2 = Infinity,
    sell1 = 0,
    sell2 = 0;
  for (let p of prices) {
    buy1 = Math.min(buy1, p);
    sell1 = Math.max(sell1, p - buy1);
    buy2 = Math.min(buy2, p - sell1);
    sell2 = Math.max(sell2, p - buy2);
  }
  return sell2;
};
```

#### [Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

You may complete at most k transactions.

dp[k][i] 表示最多交易 k 次，在前 i 天得到的最大利润。

- base case: dp[0][i] = 0; dp[k][0] = 0

- 对于每一天：

  - 若第 i 天不交易，则沿用前一天的值

    - `dp[k][i-1]`

  - 若第 i 天交易，则需要在 j: 0...i-1 中选一天买入，并求所有可能情况中的最大值

    - `max{ prices[i] - prices[j] + dp[k-1][j] } }`, j: 0...i-1.

  - 两种情况中取较大值。

```js
var maxProfit = function (prices) {
  let dp = [...Array(3)].map(() => Array(prices.length).fill(0));
  for (let k = 1; k <= 2; k++) {
    for (let i = 1; i < prices.length; i++) {
      dp[k][i] = dp[k][i - 1];
      for (let j = 0; j < i; j++) {
        dp[k][i] = Math.max(dp[k][i], prices[i] - prices[j] + dp[k - 1][j]);
      }
    }
  }
  return dp[2][prices.length - 1];
};
```

优化：去掉对于 j: 0...i-1 的循环。设 max = dp[k-1][j] - prices[j]，max 对于每一轮 k 来说是唯一的，可以在遍历 i 的过程中同时求出。

```js
var maxProfit = function (k, prices) {
  let len = prices.length, dp = [...Array(k + 1)].map(() => Array(len).fill(0)));
  for (let i = 1; i <= k; i++) {
    let max = -prices[0]; // dp[i-1][0] - prices[0]
    for (let j = 1; j < len; j++) {
      max = Math.max(max, dp[i - 1][j] - prices[j]);
      dp[i][j] = Math.max(dp[i][j - 1], prices[j] + max);
    }
  }
  return dp[k][len - 1];
};
```

优化：如果 k >= prices.length / 2，则退化为一般的 Best Time to Buy and Sell Stock 问题。

```js
var maxProfit = function (k, prices) {
  let len = prices.length;
  if (k >= len / 2) {
    let max = 0;
    for (let i = 1; i < len; i++) {
      if (prices[i] > prices[i - 1]) {
        max += prices[i] - prices[i - 1];
      }
    }
    return max;
  }

  let dp = [...Array(k + 1)].map(() => Array(prices.length).fill(0));
  for (let i = 1; i <= k; i++) {
    let max = -prices[0];
    for (let j = 1; j < len; j++) {
      max = Math.max(max, dp[i - 1][j] - prices[j]);
      dp[i][j] = Math.max(dp[i][j - 1], prices[j] + max);
    }
  }
  return dp[k][len - 1];
};
```

### Matrix & 2D Array

#### [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

Given an integer array nums, find the contiguous subarray (len >= 1) which has the largest sum and return its sum.

dp[i] 表示子数组 nums[0]...nums[i] 能得到的最大值。对于 dp[i]，若 dp[i-1] > 0 则 nums[i] + dp[i-1]，若 dp[i-1] <= 0，则 dp[i] = nums[i]。

```js
var maxSubArray = function (nums) {
  let dp = Array(nums.length).fill(0),
    max = nums[0];
  dp[0] = nums[0];
  for (let i = 1; i < nums.length; i++) {
    dp[i] = (dp[i - 1] > 0 ? dp[i - 1] : 0) + nums[i];
    max = Math.max(max, dp[i]);
  }
  return max;
};
```

optimize: just like Best Time to Buy and Sell Stock

```js
var maxSubArray = function (nums) {
  let max = -Infinity,
    curr = 0;
  for (let i = 0; i < nums.length; i++) {
    curr += nums[i];
    curr = Math.max(0, curr);
    max = Math.max(max, curr);
  }
  return max;
};
```

#### [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)

Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right, which minimizes the sum of all numbers along its path. Can only move either down or right at any point in time.

dp

```js
var minPathSum = function (grid) {
  let m = grid.length,
    n = grid[0].length;
  let dp = [...Array(m)].map(() => Array(n).fill(Infinity));
  dp[0][0] = grid[0][0];
  for (let i = 1; i < m; i++) {
    dp[i][0] = dp[i - 1][0] + grid[i][0];
  }
  for (let j = 1; j < n; j++) {
    dp[0][j] = dp[0][j - 1] + grid[0][j];
  }
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
    }
  }

  return dp[m - 1][n - 1];
};
```

optimize space:

```js
var minPathSum = function (grid) {
  let m = grid.length,
    n = grid[0].length;
  let dp = grid[0].concat();
  for (let i = 1; i < m; i++) {
    dp[i] += dp[i - 1];
  }
  for (let i = 1; i < m; i++) {
    dp[0] += grid[i][0];
    for (let j = 1; j < n; j++) {
      dp[j] = Math.min(dp[j], dp[j - 1]) + grid[i][j];
    }
  }
  return dp[n - 1];
};
```

#### [Unique Paths](https://leetcode.com/problems/unique-paths/description/)

```js
var uniquePaths = function (m, n) {
  if (m === 1 || n === 1) return 1;

  let dp = new Array(n).fill(1);
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[j] += dp[j - 1];
    }
  }

  return dp[n - 1];
};
```

### Tree

#### [Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/)

[F(i, n) = G(i-1) \* G(n-i)](<https://leetcode.com/problems/unique-binary-search-trees/discuss/31666/DP-Solution-in-6-lines-with-explanation.-F(i-n)-G(i-1)-*-G(n-i)>)

dp[i] 表示 i 个节点所能构成的 BST 数量。

i = 0: dp[0] = 1;

i = 1: dp[1] = 1;

i >= 2: `dp[i] = sum { dp[j] * dp[i - 1 - j] }`, 0 <= j < i. (i - 1 to exclude root)

```js
var numTrees = function (n) {
  let dp = Array(n + 1).fill(0);
  dp[0] = dp[1] = 1;
  for (let i = 2; i <= n; i++) {
    for (let j = 0; j < i; j++) {
      dp[i] += dp[j] * dp[i - 1 - j];
    }
  }
  return dp[n];
};
```

#### [Unique Binary Search Trees II](https://leetcode.com/problems/unique-binary-search-trees-ii/)

divide & conquer

```js
var generateTrees = function (n) {
  const helper = (l, r, res = []) => {
    if (l > r) return [null];
    for (let i = l; i <= r; i++) {
      let lefts = helper(l, i - 1),
        rights = helper(i + 1, r);
      for (let left of lefts) {
        for (let right of rights) {
          res.push(new TreeNode(i, left, right));
        }
      }
    }
    return res;
  };
  return helper(1, n);
};
```

time complexity: T(n)=T(0)T(n-1)+T(1)T(n-2)+...+T(n-1)T(0) => catalan number => O(4^n)

### Others

#### [Wiggle Subsequence](https://leetcode.com/problems/wiggle-subsequence/)

dp: up[i] 表示只考虑前 i 个数，以正数差值结尾的最长子序列长度；down[i] 表示只考虑前 i 个数，以负数差值结尾的最长子序列长度。

若 nums[i] > nums[i-1]：

- 若 nums[i-1] 是 down[i-1] 的结尾，则 up[i] = down[i-1] + 1。

- 若 nums[i-1] 不是 down[i-1] 的结尾，设 down[i-1] 子序列结尾为 y。既然 nums[i-1] 不是 down[i-1] 的结尾，则必有 nums[i-1] > y，那么 nums[i] > nums[i-1] > y，up[i] = down[i-1] + 1 仍然成立。

若 nums[i] < nums[i-1]：

- 若 nums[i-1] 是 up[i-1] 的结尾，则 down[i] = up[i-1] + 1。

- 类似上述。

若 nums[i] === nums[i-1]：up & down 沿用 index = i-1 时的值。

```js
var wiggleMaxLength = function (nums) {
  if (nums.length < 2) return nums.length;
  let up = 1,
    down = 1;
  for (let i = 1; i < nums.length; i++) {
    if (nums[i] > nums[i - 1]) up = down + 1;
    else if (nums[i] < nums[i - 1]) down = up + 1;
  }
  return Math.max(up, down);
};
```

dfs + memo

```js
var wiggleMaxLength = function (nums) {
  if (nums.length === 0) return 0;

  let n = nums.length,
    positive = Array(n).fill(0),
    negative = Array(n).fill(0);
  const dfs = (i, flag) => {
    if (i === n - 1) return 1;
    if (flag === 1 && positive[i]) return positive[i];
    if (flag === -1 && negative[i]) return negative[i];

    let max = 0;
    for (let j = i + 1; j < n; j++) {
      if ((nums[j] - nums[i]) * flag > 0)
        max = Math.max(max, dfs(j, -flag) + 1);
    }
    if (flag === 1) positive[i] = max;
    if (flag === -1) negative[i] = max;
    return max;
  };

  let max = 1;
  for (let i = 0; i < n; i++) {
    max = Math.max(max, dfs(i, 1), dfs(i, -1));
  }

  return max;
};
```

#### [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/)

Given an integer array nums, find a contiguous non-empty subarray within the array that has the largest product, and return the product.

imax/imin stores the max/min product of subarray that ends with the current number A[i];

multiplied by a negative makes big number smaller, small number bigger, so we redefine the imax & imin by swapping them;

max/min product for the current number is either the current number itself, or the max/min by the previous number times the current one;

the newly computed max value is a candidate for our global result.

```js
var maxProduct = function (nums) {
  let max = nums[0];
  for (let i = 1, imax = max, imin = max; i < nums.length; i++) {
    if (nums[i] < 0) [imin, imax] = [imax, imin]; // swap for negative numbers
    imax = Math.max(nums[i], imax * nums[i]);
    imin = Math.min(nums[i], imin * nums[i]);
    max = Math.max(max, imax);
  }
  return max;
};
```

#### [Integer Break](https://leetcode.com/problems/integer-break/)

dp[i] 表示整数 i 对应的最大乘积。由于 i 是递增的，可以将其分成两部分，使得这两部分乘积最大，这两部分构成了子问题。

则对于 j: 1...i/2，有 `maxLeft = Math.max(j, dp[j])`, `maxRight = Math.max(i-j, dp[i-j])`, `dp[i] = Math.max(dp[i], maxLeft * maxRight)`

```js
var integerBreak = function (n) {
  let dp = new Array(n + 1).fill(0);
  dp[1] = 1;

  for (let i = 2; i <= n; i++) {
    for (let j = 1; j < i; j++) {
      let maxLeft = Math.max(j, dp[j]),
        maxRight = Math.max(i - j, dp[i - j]);
      dp[i] = Math.max(dp[i], maxLeft * maxRight);
    }
  }

  return dp[n];
};
```

#### [Non-negative Integers without Consecutive Ones](https://leetcode.com/problems/non-negative-integers-without-consecutive-ones/)

Given a positive integer n, find the number of non-negative integers less than or equal to n, whose binary representations do NOT contain consecutive ones.

zero[i]: the number of binary strings of length i which do not contain any two consecutive 1’s and which end in 0.

one[i]: the number of binary strings of length i which do not contain any two consecutive 1’s and which end in 1.

res -= one[i]: reverse 之后要消除掉大于 n 的那些情况。

```js
var findIntegers = function (num) {
  let s = num.toString(2).split("").reverse().join(""),
    n = s.length;
  let zero = Array(n).fill(0),
    one = Array(n).fill(0);
  zero[0] = one[0] = 1;
  for (let i = 1; i < n; i++) {
    zero[i] = zero[i - 1] + one[i - 1];
    one[i] = zero[i - 1];
  }
  let res = zero[n - 1] + one[n - 1];
  for (let i = n - 2; i >= 0; i--) {
    if (s.charAt(i) === "1" && s.charAt(i + 1) === "1") break;
    if (s.charAt(i) === "0" && s.charAt(i + 1) === "0") res -= one[i];
  }
  return res;
};
```

#### [Decode Ways](https://leetcode.com/problems/decode-ways/)

dp[i] 表示前 i 个字符可能的编码数。

- 认为当前字符对应一位数，则前一位必须不能是 0: dp[i] += dp[i-1];

- 认为当前字符对应两位数，则前一位为 1 或者前一位为 2 且当前位小于 6: dp[i] += dp[i - 2];

```js
var numDecodings = function (s) {
  if (!s || s.length === 0) return 0;

  const n = s.length,
    dp = Array(n + 1).fill(0);
  dp[0] = 1;
  dp[1] = s[0] === "0" ? 0 : 1;

  for (let i = 2; i <= n; i++) {
    if (s[i - 1] !== "0") dp[i] += dp[i - 1];
    if (s[i - 2] == "1" || (s[i - 2] == "2" && s[i - 1] <= 6))
      dp[i] += dp[i - 2];
  }

  return dp[n];
};
```

#### [Word Break](https://leetcode.com/problems/word-break/)

Given a string s and a dictionary of strings wordDict, return true if s can be segmented into a space-separated sequence of one or more dictionary words.

两种思路。

```js
var wordBreak = function (s, wordDict) {
  let n = s.length,
    dp = new Array(n + 1).fill(false),
    set = new Set(wordDict);
  dp[0] = true;
  for (let i = 1; i <= n; i++) {
    for (let j = 0; j < i; j++) {
      if (dp[j] && set.has(s.substring(j, i))) {
        dp[i] = true;
        break;
      }
    }
  }
  return dp[n];
};
```

完全背包。

```js
var wordBreak = function (s, wordDict) {
  let n = s.length,
    dp = Array(n + 1).fill(0);
  dp[0] = true;
  for (let i = 1; i <= n; i++) {
    for (let word of wordDict) {
      if (i >= word.length && s.slice(i - word.length, i) === word) {
        dp[i] = dp[i] || dp[i - word.length];
      }
    }
  }
  return dp[n];
};
```

#### [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

https://leetcode.com/problems/trapping-rain-water/solution/

1. Brute force: For each element in the array, we find the maximum level of water it can trap after the rain, which is equal to the minimum of maximum height of bars on both the sides minus its own height.

```js
function trap(heights) {
  let res = 0,
    n = heights.length;
  for (let i = 1; i < n - 1; i++) {
    let leftMax = heights[i],
      rightMax = heights[i];
    for (let j = i - 1; j >= 0; j--) {
      leftMax = Math.max(leftMax, heights[j]);
    }
    for (let j = i + 1; j < n; j++) {
      rightMax = Math.max(rightMax, heights[j]);
    }
    res += Math.min(leftMax, rightMax) - heights[i];
  }
  return res;
}
```

2. DP: since leftMax & rightMax is fixed for each i, we could populate them in advance.

```js
function trap(heights) {
  let res = 0,
    n = heights.length,
    lefts = Array(n).fill(0),
    rights = lefts.concat();
  lefts[0] = heights[0];
  for (let i = 1; i < n; i++) {
    lefts[i] = Math.max(lefts[i - 1], heights[i]);
  }
  rights[n - 1] = heights[n - 1];
  for (let i = n - 2; i >= 0; i--) {
    rights[i] = Math.max(rights[i + 1], heights[i]);
  }
  for (let i = 1; i < n - 1; i++) {
    res += Math.min(lefts[i], rights[i]) - heights[i];
  }
  return res;
}
```

3. two pointers: 见 two-pointers.md.

#### [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/)

'\*' Matches zero or more of the preceding element.

'.' 可以匹配任意单个字符，所以只需要依次比对每个字符即可。但由于 '\*' 的存在，需要每次考虑两位字符。

- 如果前两个字符不含 '\*'，只需比较第一个字符，然后递归 s.slice(1) & p.slice(1)；

- 如果存在 '\*'，考虑 '\*' 的匹配方式：

  - 'a*' 表示匹配零个字符，则去掉 'a*'，递归 s & p.slice(2)；

  - 'a\*' 表示匹配一到多个字符，则需要比较第一个字符是否匹配：

    - 若匹配：递归 s.slice(1) & p；

    - 若不匹配：则匹配结果为 false；

  - 上述几种情况取或。

```js
var isMatch = function (s, p) {
  if (p.length === 0) return s.length === 0;
  let firstMatch = s.length !== 0 && (p[0] === s[0] || p[0] === ".");
  if (p.length >= 2 && p[1] === "*") {
    return isMatch(s, p.slice(2)) || (firstMatch && isMatch(s.slice(1), p));
  } else {
    return firstMatch && isMatch(s.slice(1), p.slice(1));
  }
};
```

DP (top-down): 先将上面的 isMatch 的递归部分提取出来，改写成通过 i & j 来分别标识当前 s 和 p 被比对的位置。由于之前递归参数是字符串，且 slice 方法默认帮助处理了越界情况，改写成下标后需要手动处理下标越界的可能情况。若 s 已经结束，p 还没有结束，由于 '\*' 的存在可能仍然匹配，比如: '' & 'a\*'；若 s 未结束，而 p 已经结束，则不可能存在能匹配的情况。

```js
var isMatch = function (s, p) {
  let map = new Map();
  function helper(i, j) {
    if (map.has(i + "," + j)) return map.get(i + "," + j);
    if (i > s.length || j > p.length) return false;
    if (i === s.length && j === p.length) return true;
    if (i !== s.length && j === p.length) return false;

    let res = false;
    if (j + 1 < p.length && p[j + 1] === "*") {
      res =
        helper(i, j + 2) ||
        ((s[i] === p[j] || p[j] === ".") && helper(i + 1, j));
    } else {
      res = (s[i] === p[j] || p[j] === ".") && helper(i + 1, j + 1);
    }
    map.set(i + "," + j, res);
    return res;
  }
  return helper(0, 0);
};
```

DP (bottom up)

```js
var isMatch = function (s, p) {
  let m = s.length,
    n = p.length;
  let dp = [...Array(m + 1)].map(() => Array(n + 1).fill(false));
  dp[0][0] = true;

  for (let i = 2; i <= n; i++) {
    if (p[i - 1] === "*") dp[0][i] = dp[0][i - 2];
  }
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s[i - 1] === p[j - 1] || p[j - 1] === ".") {
        dp[i][j] = dp[i - 1][j - 1];
      } else if (p[j - 1] === "*") {
        if (s[i - 1] !== p[j - 2] && p[j - 2] !== ".") {
          dp[i][j] = dp[i][j - 2];
        } else {
          dp[i][j] = dp[i - 1][j] || dp[i][j - 2] || dp[i][j - 1];
        }
      }
    }
  }
  return dp[m][n];
};
```

#### [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/)

```js
var isMatch = function (s, p) {
  let m = s.length,
    n = p.length,
    dp = [...Array(m + 1)].map(() => Array(n + 1).fill(false));
  dp[0][0] = true;
  for (let j = 1; j <= n; j++) {
    if (p[j - 1] === "*") dp[0][j] = true;
    else break;
  }
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (p[j - 1] !== "*") {
        dp[i][j] =
          dp[i - 1][j - 1] && (s[i - 1] === p[j - 1] || p[j - 1] === "?");
      } else {
        dp[i][j] = dp[i - 1][j] || dp[i][j - 1];
      }
    }
  }
  return dp[m][n];
};
```

#### [Knight Dialer](https://leetcode.com/problems/knight-dialer/)

1. smart dp

pad:

1 2 3
4 5 6
7 8 9

- 0 #

dp[i][j] 表示号码长度为 i 时，以数字 j 为结尾的数量。注意 dp[1][5] = 0。

又因为 i 只与 i - 1 相关，可以只用一维数组，循环 n - 1 次即可。

```js
var knightDialer = function (n) {
  if (n === 1) return 10;

  let MOD = Math.pow(10, 9) + 7,
    dp = [1, 1, 1, 1, 1, 0, 1, 1, 1, 1];
  for (let i = 1; i < n; i++) {
    dp = [
      (dp[4] + dp[6]) % MOD, // 0 can be reached via 4 / 6
      (dp[6] + dp[8]) % MOD,
      (dp[7] + dp[9]) % MOD,
      (dp[4] + dp[8]) % MOD,
      (dp[3] + dp[9] + dp[0]) % MOD,
      dp[5],
      (dp[1] + dp[7] + dp[0]) % MOD,
      (dp[2] + dp[6]) % MOD,
      (dp[1] + dp[3]) % MOD,
      (dp[2] + dp[4]) % MOD,
    ];
  }
  return dp.reduce((s, c) => (s + c) % MOD, 0);
};
```

2. dfs + memo

```js
var knightDialer = function (n) {
  let pad = [
      [1, 2, 3],
      [4, 5, 6],
      [7, 8, 9],
      [-1, 0, -1],
    ],
    max = Math.pow(10, 9) + 7,
    map = new Map();
  let dfs = (i, j, steps) => {
    if (i < 0 || j < 0 || i >= 4 || j >= 3 || (i == 3 && j != 1)) return 0;
    if (steps === n) return 1;
    let key = [i, j, steps].join(".");
    if (map.has(key)) return map.get(key);
    let res =
      (dfs(i - 1, j - 2, steps + 1) % max) +
      (dfs(i - 2, j - 1, steps + 1) % max) +
      (dfs(i - 2, j + 1, steps + 1) % max) +
      (dfs(i - 1, j + 2, steps + 1) % max) +
      (dfs(i + 1, j + 2, steps + 1) % max) +
      (dfs(i + 2, j + 1, steps + 1) % max) +
      (dfs(i + 2, j - 1, steps + 1) % max) +
      (dfs(i + 1, j - 2, steps + 1) % max);
    map.set(key, res);
    return res;
  };

  let res = 0;
  for (let i = 0; i < 4; i++) {
    for (let j = 0; j < 3; j++) {
      if (pad[i][j] !== -1) {
        res = (res + dfs(i, j, 1)) % max;
      }
    }
  }
  return res;
};
```

#### [Integer Replacement](https://leetcode.com/problems/integer-replacement/)

Given a positive integer n, you can apply one of the following operations:

If n is even, replace n with n / 2.

If n is odd, replace n with either n + 1 or n - 1.

Return the minimum number of operations needed for n to become 1.

TODO: bit manipulation

```js
var integerReplacement = function (n) {
  let map = new Map();
  function helper(curr) {
    if (curr === 1) return 0;
    if (map.has(curr)) return map.get(curr);
    let res = 0;
    if (curr % 2 === 0) {
      res = 1 + helper(curr / 2);
    } else {
      res = 1 + Math.min(helper(curr - 1), helper(curr + 1));
    }
    map.set(curr, res);
    return res;
  }
  return helper(n);
};
```
