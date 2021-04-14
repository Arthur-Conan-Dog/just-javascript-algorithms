# Greedy

## Questions

### [Gas Station](https://leetcode.com/problems/gas-station/)

update position if current tank < 0; check if total tank is < 0; When total tank is positive, it means we have enough gas to over all the previous path.

```js
var canCompleteCircuit = function (gas, cost) {
  let tank = 0,
    total = 0,
    pos = 0;
  for (let i = 0; i < gas.length; i++) {
    tank += gas[i] - cost[i];
    total += gas[i] - cost[i];
    if (tank < 0) {
      tank = 0;
      pos = i + 1;
    }
  }
  return total >= 0 ? pos : -1;
};
```

why pos = i + 1 ? => tank += gas[i] - cost[i] < 0 表明 gas[i] - cost[i] 一定为负值，不能作为起始点，故直接跳至下一个加油站 i + 1。
