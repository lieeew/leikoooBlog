---
title: "Arts--2023.10.8"
datePublished: Mon Oct 09 2023 05:11:38 GMT+0000 (Coordinated Universal Time)
cuid: clnifrano000109l8gt538a0z
slug: arts-2023108
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ylveRpZ8L1s/upload/8d782589b6d871a52cf7be9f3537fe06.jpeg

---

# Algorithm

本周题目：[LeetCode134. 加油站](https://leetcode.cn/problems/gas-station/description/)

```xml
在一条环路上有 n 个加油站，其中第 i 个加油站有汽油 gas[i] 升。

你有一辆油箱容量无限的的汽车，从第 i 个加油站开往第 i + 1 个加油站需要消耗汽油 cost[i] 升。你从其中的一个加油站出发，开始时油箱为空。

给定两个整数数组 gas 和 cost ，如果你可以按顺序绕环路行驶一周，则返回出发时加油站的编号，否则返回 -1 。如果存在解，则 保证 它是 唯一 的。

示例 1:

输入: gas = [1,2,3,4,5], cost = [3,4,5,1,2]
输出: 3
解释:
从 3 号加油站(索引为 3 处)出发，可获得 4 升汽油。此时油箱有 = 0 + 4 = 4 升汽油
开往 4 号加油站，此时油箱有 4 - 1 + 5 = 8 升汽油
开往 0 号加油站，此时油箱有 8 - 2 + 1 = 7 升汽油
开往 1 号加油站，此时油箱有 7 - 3 + 2 = 6 升汽油
开往 2 号加油站，此时油箱有 6 - 4 + 3 = 5 升汽油
开往 3 号加油站，你需要消耗 5 升汽油，正好足够你返回到 3 号加油站。
因此，3 可为起始索引。
示例 2:

输入: gas = [2,3,4], cost = [3,4,3]
输出: -1
解释:
你不能从 0 号或 1 号加油站出发，因为没有足够的汽油可以让你行驶到下一个加油站。
我们从 2 号加油站出发，可以获得 4 升汽油。 此时油箱有 = 0 + 4 = 4 升汽油
开往 0 号加油站，此时油箱有 4 - 3 + 2 = 3 升汽油
开往 1 号加油站，此时油箱有 3 - 3 + 3 = 3 升汽油
你无法返回 2 号加油站，因为返程需要消耗 4 升汽油，但是你的油箱只有 3 升汽油。
因此，无论怎样，你都不可能绕环路行驶一周
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1696499505756/8d6f4510-4867-44b7-b826-7952f549d6ad.png align="center")

这个问题是关于「贪心」这个东西非常有意识，使用这个东西「可能」会得到\*\*最优解，\*\*所以我们也不用刻意的使用贪心算法

回到这道题目，我们经过每一个加油站我们就会油变多、变少，由于是每过一个就会有增加有减少，所以我们可以用一个变量 curNum 来记录一下，如果经过了一个站点这个变量 curNum 变成负数，那么我们就需要意识到从这个站点出发是行不通的，我们需要在尝试下一个站点作为起点。当我们把所有的站点的 curTotal 相加之后，会得到总的耗油量 totalNum，当这个 totalNum &lt; 0 时证明汽车不能绕环路行驶一周。

这个还需要考虑边界问题，需要注意一下范围

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int curSum = 0;
        int totalSum = 0;
        int start = 0;
        for (int i = 0; i < gas.length; i++) {
            // 这个油是不会凭空减少的，我们使用油箱里面的油就只有开到下一处
            // 所以我们需要把每一个加起来，而不是只添加一次
            curSum += gas[i] - cost[i];
            // 这个就不需要重置，就算是负数也没有问题，负数的问题就是不能走完整的一圈
            totalSum += gas[i] - cost[i];
            // 当前累加rest[i]和 curSum一旦小于0
            if (curSum < 0) {  
                // 更新起始位置为i+1 
                start = i + 1; 
                // curSum从0开始 
                curSum = 0;     
            }
        }
        // 说明怎么走都不可能跑一圈了
        if (totalSum < 0) return -1; 
        return start;
    }
}
```

> 仔细读题！！！

# Review

这篇文章太牛了：[https://github.com/ruanyf/weekly/blob/master/docs/issue-271.md](https://github.com/ruanyf/weekly/blob/master/docs/issue-271.md)

# Technique/Tips

安装应用要在 D 盘。

代码不要放在有中文目录的路径下面。

及时做好代码上传，要不然有我哭的时候。

# Share

**重要的是方向而不是目标**，更多的时候我们是**不能**够到达我们向往的地方、实现我们的目标，但是这些都不重要，就像当我们得到一个东西，我们并不会就此满足。比如 游戏里面的角色，我们第一件事情想到的不是满足，而是接着去干强化他们、升级他们，当我们得到了，我们也不一定有自己现象的快乐。所以我们更重要的是「当下」，是我们前进的方向，只要是方向是对的哪怕付出再多的时间、精力都不会被浪费！

而且很多东西，不是你一下子就能看到结果的，很多事情都需要坚持下去。古话说的好：行百里者半九十，我们要引以为戒！