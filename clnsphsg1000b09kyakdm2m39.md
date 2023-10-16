---
title: "Arts  - 2023/10/15"
datePublished: Mon Oct 16 2023 09:41:53 GMT+0000 (Coordinated Universal Time)
cuid: clnsphsg1000b09kyakdm2m39
slug: arts-20231015
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/hGV2TfOh0ns/upload/ee7729700826336aba6d850960375854.png

---

# Algorithm

[LeetCode77](https://leetcode.com/problems/combinations/description/) 这道题目考察的是**回溯**

```xml
给定两个整数 n 和 k，返回 1 ... n 中所有可能的 k 个数的组合。例如，输入n=4，k=2，则输出：
[[2,4], [3,4], [2,3], [1,2], [1,3], [1,4]]
```

首先我们先来区分一下「回溯」和「递归」之间的区别

* `递归策略`：先与意中人制造偶遇，然后了解人家的情况，然后约人家吃饭，有好感之后尝试拉人家的手，没有拒绝就表白。
    
* `回溯策略`：先统计周围所有的单身女孩，然后一个一个表白， 被拒绝就说“我喝醉了”，然后就当啥也没发生，继续找下一个。
    

**回溯的解题模板**

```java
    void backtracking(参数) {
        if (终止条件) {
            存放结果;
            return;
        }
        for (选择本层集合中元素（画成树，就是树节点孩子的大小）){
            处理节点;
            backtracking();
            回溯，撤销处理结果;
        }
    }
```

这里有一个很难理解的地方就是我们为什么还需有一个**撤销处理结果的操作 ？**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697370774080/cbeb77b2-b2cb-49d6-92a3-e1f55e103f77.png align="center")

我们可以看到，我们收集每个结果不是针对叶子结点的，而是针对`树枝`的，比如最上层我们首先选了 1，下层如果选 2，结果就是 {1,2}，如果下层选了 3，结果就是 {1,3}，依次类推。现在的问题是当我们得到第一个结果 {1, 2} 之后，怎么得到第二个结果 {1,3} 呢？  
继续观察纵横图，可以看到，我可以在得到 {1,2} 之后将 2 撤掉，再继续取3，这样就得到了{1,3}，同理可以得到 {1,4}，之后当前层就没有了，我们可以将 1 撤销，继续从最上层取 2 继续进行。  
这里对应的代码操作就是先将第一个结果放在临时列表 path 里，得到第一个结果 {1,2} 之后就将 path 里的内容放进结果列表 resultList 中，之后，将 path 里的 2 撤销掉， 继续寻找下一个结果 {1.3} ,然后继续将path放入resultLit，然后再撤销继续找。

这样我们就能写出最后的代码了：

```java
  public List<List<Integer>> combine(int n, int k) {
    List<List<Integer>> resultList = new ArrayList<>();
    if (k <= 0 || n < k) {
      return resultList;
    }
    Deque<Integer> path = new ArrayDeque<>();
    dfs(n, k, 1, path, resultList);
    return resultList;
  }

  /**
   * 循环调用的方法
   *
   * @param n n
   * @param k k
   * @param beginIndex 开始回溯的索引
   * @param path 对应暂存集合
   * @param resultList 结果集合
   */
  public void dfs(
      int n, int k, int beginIndex, Deque<Integer> path, List<List<Integer>> resultList) {
    // 循环结束条件是 path.size() == k
    if (path.size() == k) {
      resultList.add(new ArrayList<>(path));
      return;
    }
    for (int i = beginIndex; i <= n; i++) {
      path.addLast(i);
      dfs(n, k, i + 1, path, resultList);
      path.removeLast();
    }
  }
```

为什么有 `i + 1` 这种操作 ？

因为我们的要继续向右进行遍历，当遍历到最深的位置时，我们就需要不断地向右取值放入到 path 之中，其他深度位置同理，我们都需要这样的操作

```java
for (int i = startIndex; i <= n; i++) {
     dfs(n, k, i + 1, path, res);
 }
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697371974496/3533025d-a1a3-4315-baba-608ce3d95933.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1697425828529/60de71b6-f257-443b-a4c9-59383c7bba10.png align="center")

# Review

英文文章

[https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)

善用「老代码」，不要动不动就全部重新写，动不动就看别人写的代码不顺眼自己写。对于一个新的产品我们需要利用「老代码」，而不是把之前经过大量的时间和经历写好的逻辑弃之不顾，另辟山头。

这种推到重写的方法，可能会导致新公司的破产！

# Technique/Tips

报错了，仔细看报错信息，要明白最离谱的 `bug`是没有报错信息的 `bug`

要善于利用 ChatGPT，但不要夸大 ChatGPT，他现在只是 LLM 而没有到达 AGI 的水平 [不要过分夸大 ChatGPT](https://github.com/ruanyf/weekly/blob/master/docs/issue-248.md)

# Share

人会被环境影响，但是作为适应环境的典范「人类」是不是在自己不能改变的时候，改变一下自己呢？

就算我们生活在很乱的环境，我们是不是也能适应呢? 当我身处优良环境的时候的表现或许也不尽人意，所有不要总是在抱怨环境，不要重视在不断内耗自己，更重要的是向前冲！