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
