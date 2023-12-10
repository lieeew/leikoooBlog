---
title: "Arts-2023/12/10"
datePublished: Sun Dec 10 2023 03:13:26 GMT+0000 (Coordinated Universal Time)
cuid: clpywu3fp000008jl4mp23ylq
slug: arts-20231210
tags: arts

---

# **Algorithm**

[https://leetcode.com/problems/insert-interval/description/](https://leetcode.com/problems/insert-interval/description/)

```bash
You are given an array of non-overlapping intervals intervals where intervals[i] = [starti, endi] represent the start and the end of the ith interval and intervals is sorted in ascending order by starti. You are also given an interval newInterval = [start, end] that represents the start and end of another interval.
​
Insert newInterval into intervals such that intervals is still sorted in ascending order by starti and intervals still does not have any overlapping intervals (merge overlapping intervals if necessary).
​
Return intervals after the insertion.
​
 
​
Example 1:
​
Input: intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
Example 2:
​
Input: intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]
Explanation: Because the new interval [4,8] overlaps with [3,5],[6,7],[8,10].
```

首先遍历，一般情况下 intervals 当 interval\[1\] &lt; newInterval\[0\] 的时候现在是在左边的情况。当我们 newInterval\[1\] &lt; interval\[0\] 的时候在右边，其他的情况下 newInterval 进行修改的情况。其他情况比如，这个在 **最末尾** ，所以需要一个 flag 比如 inserted

![image-20231209210638892](https://leeikoooo-1313589692.cos.ap-beijing.myqcloud.com/imgs/image-20231209210638892-1702130034900-1.png align="left")

```bash
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> res = new ArrayList<>();
        boolean inserted = false;
​
        // Edge case. empty intervals, or newInterval comes before. return merged
        if (intervals.length == 0 || newInterval[1] < intervals[0][0]) {
            res.add(newInterval);
            res.addAll(Arrays.asList(intervals));
            return res.toArray(new int[res.size()][2]);
        }
​
        for (int[] interval : intervals) {
            if (newInterval[0] > interval[1]) {
                // left interval
                res.add(interval);
            } else if (newInterval[1] < interval[0]) {
                // right interval
                if (!inserted) {
                    res.add(newInterval);
                    inserted = true;
                }
                res.add(interval);
            } else {
                // other situation we should not add its to res 
                newInterval[0] = Math.min(interval[0], newInterval[0]);
                newInterval[1] = Math.max(interval[1], newInterval[1]);
            }
        }
        // when newInterval in the end 
        if (!inserted) {
            res.add(newInterval);
        }
        res.toArray(new int[res.size()][2]);
        return res.toArray(new int[res.size()][2]);
    }
​
```

# **Review**

[Best Practices for Error Handling and Exception Management in Java](https://archive.ph/2KPL7)

Java 中的异常处理是确保报错的稳定性、可靠性

1. 需要打印异常，比如可以使用 SL4J 框架
    

```bash
try {
    // Code that may throw an exception
} catch (Exception e) {
    logger.error("An error occurred", e);
}
```

1. 需要捕获特定的异常信息，而不是直接使用 `Exception`
    

```bash
try {
    // Code that may throw specific exceptions
} catch (IOException e) {
    // Handle IOException
} catch (SQLException e) {
    // Handle SQLException
} 
```

1. **提前**抛出对应的异常
    

```bash
if (input == null) {
    throw new IllegalArgumentException("Input cannot be null");
}
```

1. 不要只是 catch 住异常，而什么都不干
    

```bash
try {
    // Code that may throw an exception
} catch (Exception e) {
    // Don't leave this block empty
    logger.error("An error occurred", e);
}
```

1. 可以在异常出现的地方，有其他的处理方式
    

```bash
try {
    // Code that may interact with an external service
} catch (ServiceUnavailableException e) {
    // Continue with an alternative approach
    logger.warn("Service is unavailable. Using alternative approach.");
}
```

1. `finally` 需要关闭，不管是否会发生异常
    

```bash
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("file.txt"));
    // Read from the file
} catch (IOException e) {
    // Handle exception
} finally {
    if (br != null) {
        try {
            br.close();
        } catch (IOException e) {
            // Handle close exception
        }
    }
}
```

1. 需要捕获 stream 流里面的异常
    

```bash
List<String> filenames = Arrays.asList("file1.txt", "file2.txt");
filenames.stream()
    .map(filename -> {
        try {
            return readFileContents(filename);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    })
    .forEach(System.out::println);
```

1. 使用 assert 检测参数是符合要求，不符合要求会直接爆 `AssertionError`
    

```bash
public int factorial(int n) {
    assert n >= 0 : "Input must be a non-negative integer";
    
    // Calculate factorial
}
```

1. 使用 Optional 替代 null
    

```bash
Optional<String> getName() {
    // Return Optional.empty() instead of null
}
```

1. 使用 Try-With-Resources 可以自动关闭流
    

```bash
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    // Read from the file
} catch (IOException e) {
    // Handle exception
}
```

# **Technique/Tips**

* 可以直接查看上一次自己的进入的目录
    

```bash
cd -
```

* 执行 history 命令
    

```bash
$ history
$ !666 
```

# **Share**

少看中文社交媒体，多看国外的文章（因为看不懂，所以不会浪费太多时间）