**300. 最长上升子序列**

>感觉很难，还没理解透彻

给定一个无序的整数数组，找到其中最长上升子序列的长度。

示例:

```
输入: [10,9,2,5,3,7,101,18]
输出: 4
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
```
说明:

  - 可能会有多种最长上升子序列的组合，你只需要输出对应的长度即可。
  - 你算法的时间复杂度应该为 O(n2) 。

实现代码：
```
class Solution {
    public int lengthOfLIS(int[] nums) {
      if (nums == null || nums.length == 0) {
        return 0;
      }
      int length = nums.length;
      int[] dp = new int[length];
      int result = 0;
      dp[0] = 1;
      for (int i=1; i<length; i++) {
        int maxNum = 0;
        for (int j=0; j<i; j++) {
          if (nums[i] > nums[j]) {
            maxNum = Math.max(dp[j], maxNum);
          }
        }
        dp[i] = maxNum + 1;
        result = Math.max(result, maxNum);
      }
      return result;
}
```
