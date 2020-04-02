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
