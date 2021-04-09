# Math

注意：

- 各种边界情况。如存在负数、0不能做除数，做加法和乘法时容易溢出等。

- 边界值。JS 中 2^31 - 1 = 2147483647.

公式：

- 计算组合数 C(n,r)=n!/(r!(n−r)!)

## 进制转换

### [Excel Sheet Column Number](https://leetcode.com/problems/excel-sheet-column-number/)

```js
var titleToNumber = function (s) {
  let res = 0;
  for (let ch of s) {
    res *= 26;
    res += ch.charCodeAt() - 64;
  }
  return res;
};
```

### [Excel Sheet Column Title](https://leetcode.com/problems/excel-sheet-column-title/)

原本 i 进制每一位取值范围为 [0, i-1]。

这题稍微特殊，数位取值在：[1, 26]，需要先减 1，让取余在 [0,25] 内，转换字符时再把 1 加回来。

```js
var convertToTitle = function (n) {
  let res = [];
  while (n > 0) {
    n--; // tricky part
    res.push(String.fromCharCode(65 + (n % 26)));
    n = Math.floor(n / 26);
  }
  return res.reverse().join("");
};
```

### [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/)

```js
var hammingWeight = function (n) {
  let res = 0;
  while (n > 0) {
    res += n % 2;
    n = Math.floor(n / 2);
  }
  return res;
};
```

### [Reverse Bits](https://leetcode.com/problems/reverse-bits/)

eg. 6 => 110 取余依次得到的永远是最右端下一个数位，即 0 - 1 - 1。array.push 刚好翻转了原序列。

```js
var reverseBits = function (n) {
  let bits = [];
  while (n > 0) {
    bits.push(n % 2);
    n = Math.floor(n / 2);
  }
  while (bits.length < 32) bits.push(0);
  return bits.reduce((s, c) => s * 2 + c, 0);
};
```

### [Reverse Integer](https://leetcode.com/problems/reverse-integer/)

```js
var reverse = function (x) {
  let res = 0,
    positive = x > 0;
  x = Math.abs(x);
  while (x > 0) {
    res = res * 10 + (x % 10);
    x = Math.floor(x / 10);
  }
  return res > 2147483647 ? 0 : positive ? res : -res;
};
```

### [Roman to Integer](https://leetcode.com/problems/roman-to-integer/)

```js
var romanToInt = function(s) {
    let map = {
        'I': 1,
        'V': 5,
        'X': 10,
        'L': 50,
        'C': 100,
        'D': 500,
        'M': 1000
    }
    let res = 0
    for (let i = 0; i < s.length; i++) {
        if (s[i] === 'I' && i < s.length - 1 && (s[i+1] === 'V' || s[i+1] === 'X')) {
            res -= 1
        } else if (s[i] === 'X' && i < s.length - 1 && (s[i+1] === 'L' || s[i+1] === 'C')) {
            res -= 10
        } else if (s[i] === 'C' && i < s.length - 1 && (s[i+1] === 'D' || s[i+1] === 'M')) {
            res -= 100
        } else {
            res += map[s[i]]
        }
    }
    return res
};
```

## 位运算

### [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/)

```js
var getSum = function (a, b) {
  return b == 0 ? a : getSum(a ^ b, (a & b) << 1);
};
```

### [Missing Number](https://leetcode.com/problems/missing-number/)

XOR: a^b^b = a

```js
var missingNumber = function (nums) {
  let res = nums.length;
  for (let i = 0; i < nums.length; i++) {
    res = res ^ i ^ nums[i];
  }
  return res;
};
```

```js
var missingNumber = function (nums) {
  let n = nums.length,
    total = (n * (n + 1)) / 2;
  return total - nums.reduce((prev, curr) => prev + curr, 0);
};
```

### [Game of Life](https://leetcode.com/problems/game-of-life/)

比特位记录状态值
        next <- current
// 00   0    <- 0
// 01   0    <- 1
// 10   1    <- 0
// 11   1    <- 1

```js
var gameOfLife = function (board) {
  let m = board.length,
    n = board[0].length;
  const count = (x, y) => {
    let cnt = 0;
    for (let i = Math.max(x - 1, 0); i <= Math.min(x + 1, m - 1); i++) {
      for (let j = Math.max(y - 1, 0); j <= Math.min(y + 1, n - 1); j++) {
        cnt += board[i][j] & 1;
      }
    }
    cnt -= board[x][y] & 1; // only count current status
    return cnt;
  };
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      let lives = count(i, j);
      if (board[i][j] === 1 && lives >= 2 && lives <= 3) board[i][j] = 3;
      if (board[i][j] === 0 && lives === 3) board[i][j] = 2;
    }
  }
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      board[i][j] >>= 1; // 右移一位得到 next status
    }
  }
};
```

## 素数

### [Count Primes](https://leetcode.com/problems/count-primes/)

排除范围内所有是质数的数字，剩下的就质数。

```js
var countPrimes = function (n) {
  let isPrime = Array(n).fill(true),
    cnt = 0;
  for (let i = 2; i * i < n; i++) {
    if (!isPrime[i]) continue;
    for (let j = i * i; j < n; j += i) {
      isPrime[j] = false;
    }
  }
  for (let i = 2; i < n; i++) {
    if (isPrime[i]) cnt++;
  }
  return cnt;
};
```

## 指数

### [Pow(x, n)](https://leetcode.com/problems/powx-n/)

```js
var myPow = function (x, n) {
  if (n === 0) return 1;
  let pow = Math.abs(n);
  let res =
    pow % 2 === 0 ? myPow(x * x, pow / 2) : myPow(x * x, (pow - 1) / 2) * x;
  return n > 0 ? res : 1 / res;
};
```

## 手动计算加减乘除

TODO：减法、[乘法](https://leetcode.com/problems/multiply-strings/)

### [Plus One](https://leetcode.com/problems/plus-one/)

```js
var plusOne = function (digits) {
  for (let i = digits.length - 1; i >= 0; i--) {
    if (digits[i] < 9) {
      digits[i]++;
      return digits;
    }
    digits[i] = 0;
  }
  digits.unshift(1);
  return digits;
};
```

### [Add Strings](https://leetcode.com/problems/add-strings/)

模拟加法。

```js
var addStrings = function (num1, num2) {
  let res = "",
    i = num1.length - 1,
    j = num2.length - 1;
  let carry = 0,
    sum = 0;
  while (i >= 0 || j >= 0 || carry !== 0) {
    sum = 0;
    if (i >= 0) sum += parseInt(num1[i--]);
    if (j >= 0) sum += parseInt(num2[j--]);
    sum += carry;
    carry = Math.floor(sum / 10);
    sum %= 10;
    res = sum.toString() + res; // add at front
  }
  return res;
};
```

### [Add to Array-Form of Integer](https://leetcode.com/problems/add-to-array-form-of-integer/)

模拟加法。

```js
var addToArrayForm = function (A, K) {
  let B = K.toString()
      .split("")
      .map((x) => parseInt(x)),
    res = [];
  let carry = 0,
    n = A.length,
    m = B.length;
  let i = n - 1,
    j = m - 1,
    k = 0,
    sum = 0;
  while (i >= 0 || j >= 0 || carry !== 0) {
    sum = 0;
    if (i >= 0) sum += A[i--];
    if (j >= 0) sum += B[j--];
    sum += carry;
    carry = Math.floor(sum / 10);
    sum %= 10;
    res[k++] = sum;
  }
  return res.reverse();
};
```

## 最大公因数 GCD

### [X of a Kind in a Deck of Cards](https://leetcode.com/problems/x-of-a-kind-in-a-deck-of-cards/)

[how to calculate gcd?](https://en.wikipedia.org/wiki/Greatest_common_divisor#Euclid's_algorithm)

```js
function gcd(a, b) {
  return b > 0 ? gcd(b, a % b) : a;
}

var hasGroupsSizeX = function (deck) {
  let map = {};
  for (let card of deck) map[card] = (map[card] || 0) + 1;
  let values = Object.values(map),
    res = 0;
  for (let v of values) res = gcd(v, res);
  return res > 1;
};
```

### [Fraction Addition and Subtraction](https://leetcode.com/problems/fraction-addition-and-subtraction/)

a / b + c / d => (ad + cb) / bd

```js
function gcd(a, b) {
  return b > 0 ? gcd(b, a % b) : a;
}

function add(s1, s2) {
  let [a, b] = s1.split("/").map((x) => parseInt(x)),
    [c, d] = s2.split("/").map((x) => parseInt(x));
  let top = a * d + b * c,
    bottom = b * d,
    sign = "";
  if (top < 0) {
    sign = "-";
    top = -top;
  }
  let divisor = gcd(top, bottom);
  return sign + top / divisor + "/" + bottom / divisor;
}

var fractionAddition = function (expression) {
  let tokens = expression.split(/(?=[+-])/),
    res = "0/1";
  for (let token of tokens) res = add(res, token);
  return res;
};
```

## 中位数

快速选择找中位数 => 找数组中任意第 ith 大的数 eg. Kth Largest Element in an Array.

### [Minimum Moves to Equal Array Elements II](https://leetcode.com/problems/minimum-moves-to-equal-array-elements-ii/)

```js
var minMoves2 = function(nums) {
    if (nums.length === 1) return 0;
    nums.sort((a, b) => a - b);
    let i = 0, j = nums.length - 1, res = 0;
    while (i < j) {
        res += nums[j] - nums[i];
        j--;
        i++;
    }
    return res;
};
```

## 几何

### [Check If It Is a Straight Line](https://leetcode.com/problems/check-if-it-is-a-straight-line/)

三点一线。

y - y1 / x - x1 = y1 - y0 / x1 - x0

=> (x1 - x0) \* (y - y1) = (y1 - y0) \* (x - x1)

=> dx \* (y - y1) = dy \* (x - x1)

```js
var checkStraightLine = function (coordinates) {
  if (coordinates.length === 2) return true;
  let x0 = coordinates[0][0],
    y0 = coordinates[0][1],
    x1 = coordinates[1][0],
    y1 = coordinates[1][1];
  let dx = x1 - x0,
    dy = y1 - y0;
  for (let [x, y] of coordinates) {
    if (dx * (y - y1) !== dy * (x - x1)) return false;
  }
  return true;
};
```

## Others

### [Third Maximum Number](https://leetcode.com/problems/third-maximum-number/)

比大小。

```js
var thirdMax = function(nums) {
  let max1 = -Infinity, max2 = max1, max3 = max1;
  for (let n of nums) {
    if (n === max1 || n === max2 || n === max3) continue;
    if (n > max1) {
      max3 = max2;
      max2 = max1;
      max1 = n;
    } else if (n > max2) {
      max3 = max2;
      max2 = n;
    } else if (n > max3) {
      max3 = n;
    }
  }
  return max3 == -Infinity ? max1 : max3;
};
```

### [Power of Three](https://leetcode.com/problems/power-of-three/)

利用边界值 / 溢出值

```js
var isPowerOfThree = function (n) {
  if (n <= 0) return false;
  let max = 1162261467; // 3^20 = 3486784401 > 2^31 - 1
  return max % n === 0;
};
```

### [Can Place Flowers](https://leetcode.com/problems/can-place-flowers/)

奇偶。

```js
var canPlaceFlowers = function(flowerbed, n) {
    let curr = 1, res = 0;
    for (let i = 0; i < flowerbed.length; i++) {
        if (flowerbed[i] === 1) {
            res += Math.floor((curr - 1) / 2);
            curr = 0;
        } else {
            curr++;
        }
    }
    res += Math.floor(curr / 2);
    return res >= n;
};
```
