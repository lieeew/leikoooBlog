---
title: "Arts--2023.10.1"
datePublished: Sun Oct 01 2023 14:09:58 GMT+0000 (Coordinated Universal Time)
cuid: cln7jgsem000509ijgd2m7w4h
slug: arts-2023101
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ylveRpZ8L1s/upload/8c9ca933d653c99782e2e36edc799fbe.jpeg

---

# Algorithm

题目： [**划分字母区间**](https://leetcode.cn/problems/partition-labels/description) 给你一个字符串 `s` 。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。注意，划分结果需要满足：将所有划分结果按顺序连接，得到的字符串仍然是 `s` 。返回一个表示每个字符串片段的长度的列表。

```markdown
示例 1：
输入：s = "ababcbacadefegdehijhklij"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca"、"defegde"、"hijhklij" 。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 这样的划分是错误的，因为划分的片段数较少。 
示例 2：

输入：s = "eccbbbbdec"
输出：[10]
```

要注意这里面有一个关键点是 `仅由小写英文字母组成` 这就告诉我们有优化解法，就是可以使用 下面的写法

```java
        String char = "abcdefgh";
        char[] chars = char.toCharArray();
        int[] edge = new int[26];
        for (int i = 0; i < chars.length; i++) {
            edge[chars[i] - 'a']++;
        }
```

一个最关键的一点就是，`chars[i] - 'a'` 当我们的 'a' - 'a' 时就会得到数字 0 , 'b' - 'a' = 1 , 'c' - 'a' = 2 ..... 这个技巧在优化 **小写字符串** 非常有用这样的操作就可以记录字符串包含的字符有哪些。

回到本题上，解法之中包含这么一段代码：

```java
        String char = "abcdefgh";
        char[] chars = char.toCharArray();
        int[] edge = new int[26];
        for (int i = 0; i < chars.length; i++) {
            edge[chars[i] - 'a'] = i;
        }
```

那么这样操作之后 `edge` 就变成了什么？

> 变成了一个数值集合，每一个位置上都是最后一次出现对应字母的索引位置

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695714072876/af8ab394-3d55-420c-8dcc-cfba9cfe476e.png align="center")

结合网上的图解

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695711205305/3a7b11fb-95fd-4dfc-a381-61957cbd4c0f.png align="center")

```java
class Solution {
    public List<Integer> partitionLabels(String S) {
        List<Integer> list = new LinkedList<>();
        int[] edge = new int[26];
        char[] chars = S.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            edge[chars[i] - 'a'] = i;
        }
        int idx = 0;
        // 极端情况下，当我们的整字符串就是答案的话，那整个 list 的长度就是 i - (-1) 
        // 其他情况下就需要 i - last 
        int last = -1;
        for (int i = 0; i < chars.length; i++) {
            idx = Math.max(idx, edge[chars[i] - 'a']);
            if (i == idx) {
                list.add(i - last);
                last = i;
            }
        }
        return list;
    }
}
```

# Review

[Should I Rust or should I go?](https://kerkour.com/should-i-rust-or-should-i-go) 这篇文章讲解了 Rust 和 Go 的对比，作者比较支持 Go

文章解释到，Rust 更新的新的特型太多了，2022 到 2023 年就发布了 **31** 个版本，相比于 Go 只发布了 8 个版本 Python 就发布了 3 个版本，新特性过犹不及

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1695715253617/d6074b5d-aa36-45c0-ab06-a9f5d0eea462.png align="center")

而且 Rust 要 async 相对复杂，需要在原本的代码上修改很多东西 .

所以我要开始卷 Go 了（bushi）

# Technique/Tips

报错的时候要仔细看报错信息

要是 IDEA 犯病就把 .idea 文件删除之后重新打开就好了

# Share

人都是不知道满足的，要知足常乐。

在增广贤文之中有一句话：儿孙自有儿孙福，莫为儿孙作马牛。**人生不满百，常怀千岁忧。**

而且还要知道进退，有一些 bug 可以放一下等自己切换思维可能一下子就解决了也说不定

要知道合理放松自己，玩玩游戏什么的，但是要明白一件事，游戏时玩不完的慢慢的玩而不是迫切的升级打怪，要分清主次