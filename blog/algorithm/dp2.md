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
