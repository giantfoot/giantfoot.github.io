**72. 编辑距离**

给定两个单词 word1 和 word2，计算出将 word1 转换成 word2 所使用的最少操作数 。

你可以对一个单词进行如下三种操作：

1. 插入一个字符
2. 删除一个字符
3. 替换一个字符

示例 1:
```
输入: word1 = "horse", word2 = "ros"
输出: 3
解释:
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
示例 2:

输入: word1 = "intention", word2 = "execution"
输出: 5
解释:
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```
实现代码：
```
class Solution {
    public int minDistance(String word1, String word2) {
        if (word1 == null || word2 == null) {
          return 0;
        }
        int l1 = word1.length();
        int l2 = word2.length();
        if (l1 == 0 || l2 == 0) {
          return l1 + l2;
        }

        int[][] dp = new int[l1+1][l2+1];
        for (int i=0; i<l2+1; i++) {
          dp[0][i] = i;
        }
        for (int j=0; j<l1+1; j++) {
          dp[j][0] = j;
        }
        for (int i=1; i<=l1; i++) {
          for (int j=1; j<=l2; j++) {
            dp[i][j] = Math.min(Math.min(dp[i-1][j],dp[i][j-1]),word1.charAt(i-1)==word2.charAt(j-1) ? dp[i-1][j-1]-1 : dp[i-1][j-1]) + 1;
          }
        }
        return dp[l1][l2];
    }
}
```
