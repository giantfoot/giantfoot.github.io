# 动态规划总结及刷题记录


> 什么样的问题适合用动态规划来解决呢？

- 实际上，动态规划作为一个非常成熟的算法思想，很多人对此已经做了非常全面的总结。我把这部分理论总结为“**一个模型三个特征**”。

- 多阶段决策最优解模型：
  > 我们一般是用动态规划来解决最优问题。而解决问题的过程，需要经历多个决策阶段。每个决策阶段都对应着一组状态。然后我们寻找一组决策序列，经过这组决策序列，能够产生最终期望求解的最优值。

- 三个特征：最优子结构、无后效性和重复子问题。
  - 最优子结构
    - 最优子结构指的是，问题的最优解包含子问题的最优解。反过来说就是，我们可以通过子问题的最优解，推导出问题的最优解。如果我们把最优子结构，对应到我们前面定义的动态规划问题模型上，那我们也可以理解为，后面阶段的状态可以通过前面阶段的状态推导出来。
  - 无后效性
    - 无后效性有两层含义，第一层含义是，在推导后面阶段的状态的时候，我们只关心前面阶段的状态值，不关心这个状态是怎么一步一步推导出来的。第二层含义是，某阶段状态一旦确定，就不受之后阶段的决策影响。无后效性是一个非常“宽松”的要求。只要满足前面提到的动态规划问题模型，其实基本上都会满足无后效性。
  -重复子问题
    - 如果用一句话概括一下，那就是，不同的决策序列，到达某个相同的阶段时，可能会产生重复的状态。

- 解决动态规划问题，一般有两种思路。我把它们分别叫作，**状态转移表法**和**状态转移方程法**。
  - 状态转移表法
    - 一般能用动态规划解决的问题，都可以使用回溯算法的暴力搜索解决。所以，当我们拿到问题的时候，我们可以先用简单的回溯算法解决，然后定义状态，每个状态表示一个节点，然后对应画出递归树。从递归树中，我们很容易可以看出来，是否存在重复子问题，以及重复子问题是如何产生的。以此来寻找规律，看是否能用动态规划解决。找到重复子问题之后，接下来，我们有两种处理思路，第一种是直接用回溯加“备忘录”的方法，来避免重复子问题。从执行效率上来讲，这跟动态规划的解决思路没有差别。第二种是使用动态规划的解决方法，状态转移表法。

    - 我们先画出一个状态表。状态表一般都是二维的，所以你可以把它想象成二维数组。其中，每个状态包含三个变量，行、列、数组值。我们根据决策的先后过程，从前往后，根据递推关系，分阶段填充状态表中的每个状态。最后，我们将这个递推填表的过程，翻译成代码，就是动态规划代码了。尽管大部分状态表都是二维的，但是如果问题的状态比较复杂，需要很多变量来表示，那对应的状态表可能就是高维的，比如三维、四维。那这个时候，我们就不适合用状态转移表法来解决了。一方面是因为高维状态转移表不好画图表示，另一方面是因为人脑确实很不擅长思考高维的东西。

  - 状态转移方程法
      - 状态转移方程法有点类似递归的解题思路。我们需要分析，某个问题如何通过子问题来递归求解，也就是所谓的最优子结构。根据最优子结构，写出递归公式，也就是所谓的状态转移方程。有了状态转移方程，代码实现就非常简单了。一般情况下，我们有两种代码实现方法，一种是递归加“备忘录”，另一种是迭代递推。
      状态转移方程是解决动态规划的关键。如果我们能写出状态转移方程，那动态规划问题基本上就解决一大半了，而翻译成代码非常简单。但是很多动态规划问题的状态本身就不好定义，状态转移方程也就更不好想到。

**LeetCode**

[崔添翼 背包九讲](https://blog.csdn.net/yandaoqiusheng/article/details/84782655)

[知乎优秀答案](https://www.zhihu.com/question/23995189/answer/1094101149)

> 主要题目：
```
第 5 题、
第 53 题、
第 300 题、
第 72 题、
第 1143 题、
第 62 题、
第 63 题、
背包问题（第 416 题，第 494 题）、
硬币问题（第 322 题、第 518 题）、
打家劫舍问题（做头两题即可）、
股票问题、
第 96 题、
第 139 题、
第 10 题、
第 91 题、
第 221 题。
```

**494. 目标和**

给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。
返回可以使最终数组和为目标数 S 的所有添加符号的方法数。

```
输入: nums: [1, 1, 1, 1, 1], S: 3
输出: 5
解释:

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

一共有5种方法让最终目标和为3。

注意:

数组非空，且长度不会超过20。
初始的数组的和不会超过1000。
保证返回的最终结果能被32位整数存下。
```
解题思路：
> 这道题给定了各种范围，明显很适合使用状态转移表法，但由于数组的和存在负数，所以要特殊处理，给sum加上1000的冗余值，得到20 X 2000的二维数组，然后初始化第一行数据，接着对数据里的每个可能的值进行赋值，最后得出结果。

> 优化：不需要对二维数组的每个数据进行赋值，添加筛选条件，减少赋值操作。

优化前实现代码：
```
public class Solution {
    public int findTargetSumWays(int[] nums, int S) {
        int length = nums.length;
        int[][] dp = new int[length][2001];
        for (int i=0; i<=2001; i++) {
            if (nums[0] == i) {
                dp[0][1000+i]++;
                dp[0][1000-i]++;
            }
        }
        for (int i=1; i<length; i++) {
            for (int sum=0; sum <= 2000; sum++) {
                dp[i][sum] = (sum-nums[i] >= 0 ? dp[i-1][sum-nums[i]] : 0) + (sum+nums[i] <= 2000 ? dp[i-1][sum+nums[i]] : 0);
            }
        }
        return (S > 1000 || S < -1000) ? 0 : dp[length-1][S+1000];
    }
}
```
优化后的代码：
```
public class Solution {
    public int findTargetSumWays(int[] nums, int S) {
        int length = nums.length;
        int[][] dp = new int[length][2001];
        for (int i=0; i<=2001; i++) {
            if (nums[0] == i) {
                dp[0][1000+i]++;
                dp[0][1000-i]++;
            }
        }
        for (int i=1; i<length; i++) {
            for (int sum=0; sum <= 2000; sum++) {
                if ((sum-nums[i] > 0 && dp[i-1][sum-nums[i]] > 0) || (sum+nums[i] <= 2000 && dp[i-1][sum+nums[i]] > 0)) {
                    dp[i][sum] = (sum-nums[i] >= 0 ? dp[i-1][sum-nums[i]] : 0) + (sum+nums[i] <= 2000 ? dp[i-1][sum+nums[i]] : 0);
                }
            }
        }
        return (S > 1000 || S < -1000) ? 0 : dp[length-1][S+1000];
    }
}
```


**5.最长回文子串**

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例 1：
```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

解题思路1 ：
>暴力破解不可取，对暴力破解进行优化-动态规划，处理过的方案存储在数组里，后面直接使用。
数组boolean[i][j] sign记录起始坐标 i 和 结束坐标 j之间的子字符串是否为回文字符串，那么sign[i][j] = sign[i+1][j-1] && s.charAt(i) == s.charAt(j)，其中长度等于1和2的时候要特殊处理。

实现代码1：
```
class Solution {
    public String longestPalindrome(String s) {
        int length = s.length();
        boolean[][] sign = new boolean[length][length];
        int maxLength = 0;
        String maxStr = "";
        for (int i=1; i<=length; i++) {
          for (int start=0; start<length; start++) {
            int end = start + i - 1;
            if (end >= length) {
              break;
            }
            sign[start][end] = (i==1 || i==2 || sign[start+1][end-1]) && s.charAt(start) == s.charAt(end);
            if (sign[start][end] && i > maxLength) {
              maxLength = i;
              maxStr = s.substring(start, end+1);
            }
          }
        }
        return maxStr;
    }
}
```
解题思路2 ：
>中心扩展法。整个数组包括间隙共2n-1个元素，每个元素两边的元素相同的子字符串为回文字符串，同理，中心前后坐标对分别是(0,1)(0,2),(1,2)(1,3),(2,3)(2,4),(3,4)(3,5)，即：(n,n+1),(n,n+2)两种情况，依次遍历后即可得出结果

实现代码2：
```
class Solution {
    public String longestPalindrome(String s) {
        int length = s.length();
        char[] chars = s.toCharArray();
        String maxStr = "";
        if (length > 0) {
            maxStr = s.substring(0,1);
        }
        int maxLength = 0;
        int start = 0;
        int end = 0;
        int[] ends = new int[2];
        for (int i=0; i<length-1; i++) {
          ends[0] = i+1;
          ends[1] = i+2;
          for (int num : ends) {
            start = i;
            end = num;
            while (start >= 0 && end <= length-1) {
              if (chars[start] == chars[end]) {
                if (end-start > maxLength) {
                  maxLength = end - start;
                  maxStr = s.substring(start, end+1);
                }
                start--;
                end++;
                continue;
              } else {
                break;
              }
            }
          }
        }
        return maxStr;
    }
}
```

**53. 最大子序和**

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
```
输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```

解题思路1：
>暴力破解的优化版，记录计算过的组合，但若是数组过大仍然会超出内存，此法不可取

实现代码1：
```
class Solution {
    public int maxSubArray(int[] nums) {
      int length = nums.length;
      int maxNum = Integer.MIN_VALUE;
      int[][] dp = new int[length][length];
      for (int i=0; i<length; i++) {
        dp[i][i] = nums[i];
        if (maxNum < nums[i]) {
          maxNum = nums[i];
        }
      }
      for (int i=0; i<length-1; i++) {
        for (int j=i+1; j<length; j++) {
          dp[i][j] = dp[i][j-1] + nums[j];
          if(dp[i][j] > maxNum) {
            maxNum = dp[i][j];
          }
        }
      }
      return maxNum;
    }
}
```

解题思路2：
>优化版动态规划，通过修改原数组值来记录计算过的最大值，从左向右不断后移,

实现代码2：
```
class Solution {
    public int maxSubArray(int[] nums) {
      int length = nums.length;
      int maxNum = nums[0];
      for (int i=1; i<length; i++) {
        if (nums[i-1] > 0) {
          nums[i] += nums[i-1];
        }
        maxNum = Math.max(maxNum, nums[i]);
      }
      return maxNum;
    }
}
```

解题思路3：

状态转移方程：
```
        dp[i−1]+nums[i], if dp[i−1]≥0
dp[i]=         
        nums[i], if dp[i−1]<0
​
```

实现代码3：
```
public class Solution {

    public int maxSubArray(int[] nums) {
        int len = nums.length;
        if (len == 0) {
            return 0;
        }
        int[] dp = new int[len];
        dp[0] = nums[0];
        for (int i = 1; i < len; i++) {
            if (dp[i - 1] >= 0) {
                dp[i] = dp[i - 1] + nums[i];
            } else {
                dp[i] = nums[i];
            }
        }
        // 最后不要忘记全部看一遍求最大值
        int res = dp[0];
        for (int i = 1; i < len; i++) {
            res = Math.max(res, dp[i]);
        }
        return res;
    }
}

压缩空间优化：
public int maxSubArray(int[] nums) {
    if (nums == null) {
        return 0;
    }
    int max = nums[0];    // 全局最大值
    int subMax = nums[0];  // 前一个子组合的最大值,状态压缩
    for (int i = 1; i < nums.length; i++) {
        if (subMax > 0) {
            // 前一个子组合最大值大于0，正增益
            subMax = subMax + nums[i];
        } else {
            // 前一个子组合最大值小于0，抛弃前面的结果
            subMax = nums[i];
        }
        // 计算全局最大值
        max = Math.max(max, subMax);
    }

    return max;
}


```
