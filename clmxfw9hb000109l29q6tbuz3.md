---
title: "Arts -- 2023.9.24"
datePublished: Sun Sep 24 2023 12:32:20 GMT+0000 (Coordinated Universal Time)
cuid: clmxfw9hb000109l29q6tbuz3
slug: arts-2023924
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ylveRpZ8L1s/upload/e909a81fcaa207c644edc431ac6c1ea3.jpeg
tags: arts

---

# Algorithm

本周刷的算法是：[57\. 插入区间](https://leetcode.cn/problems/insert-interval/description/)

这道算法对我来说很有意义，为什么？ 因为这一道算法让我意识到我之前写的算法都是「混」过来的，理解根本不到位，怎么解决？重复的刷之前写过的重要的算法，仔细思考

```xml
给你一个 无重叠的 ，按照区间起始端点排序的区间列表。

在列表中插入一个新的区间，你需要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。

示例 1：

输入：intervals = [[1,3],[6,9]], newInterval = [2,5]
输出：[[1,5],[6,9]]
示例 2：

输入：intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
输出：[[1,2],[3,10],[12,16]]
解释：这是因为新的区间 [4,8] 与 [3,5],[6,7],[8,10] 重叠。
示例 3：

输入：intervals = [], newInterval = [5,7]
输出：[[5,7]]
示例 4：

输入：intervals = [[1,5]], newInterval = [2,3]
输出：[[1,5]]
示例 5：

输入：intervals = [[1,5]], newInterval = [2,7]
输出：[[1,7]]
```

解答：

首先来看一下下面的图

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695558640974/84846299-26a3-44b5-8a4d-b94b23d58c43.png align="center")

可以发现，分为 3 部分

* 左边不重叠的区间
    
* 中间重叠的区间
    
* 右边不重叠的区间
    

那么主要的问题就是，这几个区间的如何进行区分 ？

首先左边不重叠的区间，我们可以很容易想到： 绿区间的 **右边** &lt; 蓝区间的**左边**。

然后呢, 如何区分中间的重叠区间？首先我们需要明白，如果是已经经过了上面的判断到达需要判断是不是重叠的时候，隐含的条件就是 `已经存在至少一个重复的部分` 所以我们主要的目标就是这个重叠的区间在什么时候结束，看上图我们发现在 8 位置的时候刚好结束重叠，那么我们就明白了 这里的条件就是：经过上一个条件的过滤 + 绿区间的 **左边** &lt;= 蓝区间的 **右边** (我们这里需要的是重复的部分, 绿区间的 **左边** &gt;= 蓝区间的 **右边** 这个条件是刚好不重复的条件，所以需要取反) 。

最后一部分也是相对简单的，只要到能到这里那么就一定是 右边不重叠的部分了。

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        // 结果集合，当插入的数组完全没有交集的时候就需要  intervals.length + 1 长度的数组
        // 但是也会又重复的情况，所以我们需要定义一个 index 计算一下有几个
        int[][] res = new int[intervals.length + 1][2];
        int index = 0;
         // 工具人
        int i = 0;
        while (i < intervals.length && intervals[i][1] < newInterval[0]) {
            res[index++] = intervals[i++];
        }
        // 循环判断，最小的左边界和最大的右边界
        while (i < intervals.length && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        res[index++] = newInterval;
        while (i < intervals.length) {
            res[index++] = intervals[i++];
        }
        // 放入几次就取对应的长度的数组
        return Arrays.copyOf(res, index);
    }
}
```

# Review

[https://www.quora.com/What-are-some-of-the-most-basic-things-every-programmer-should-know](https://www.quora.com/What-are-some-of-the-most-basic-things-every-programmer-should-know)

这篇文章分享的是作者的一些编码上的见解

1. 测试非常重要，就比如我上线 伙伴匹配系统的时候，我本地稍微测试了一下对应的接口，但是没做线上测试，导致上线之后出现了很多问题：不能正常显示队伍、不能修改队伍信息...
    
2. 写的不好的代码最大的代价就是维护，我现在还没有很深的体会
    
3. 代码的格式很重要。虽然 Java 不强制要求格式，但是我的第一位编程老师给我们说格式非常重要，我现在代码的格式还算规范
    
4. 世界上不存在 5 分钟的工作。总是至少需要半天时间
    

# Technique/Tips

找 bug 不要局限在国内，要善于使用英文进行搜索，或许会有意想不到的结果（毕竟写代码的大部分都说英文）

可以使用 github desktop 进行代码上传，但是好像不能正常的添加 .ignore 的文件，所以我们可以使用 idea 上面的 git 工具进行 commit 自己需要被 push 的代码，其他的文件，比如 application-prod.yml 文件不要上传到 GitHub 上面

# Share

有时候我感觉很累很累，干某件事非常反人性，但是最近我看来一本书「纳瓦尔宝典」里面就提到了，其实很多时候我们感觉很累，很难受其实主要是因为我们的在干某一件事情之前，大脑就会向是非常累、难受，所以就算我们开始做事情的时候我们就会陷入到内耗。但是在「纳瓦尔宝典」之中作者就说，其实大脑在欺骗我们，干这些事情的时候其实并没有非常的难受，在这本书之中，作者就说你可以尝试洗一洗冷水澡，会发现皮肤没有碰到冷水的时候自己是「最冷的」，反而碰到之后就没这么冷。

所以，我就通过这本书，明白了其实很多事情其实没什么，比如每周读一读英文文章