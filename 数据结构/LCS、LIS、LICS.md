# 一、最长公共子串

问题：有两个字符串str1和str2，求出两个字符串中最长公共子串长度。

例子：str1=acbcbcef，str2=abcbced，则str和str2的最长公共子串为bcbce，最长公共子串长度为5。

**思路：**

1、把两个字符串分别以行和列组成一个二维矩阵。

2、比较二维矩阵中每个点对应行列字符中否相等，相等的话值设置为1，否则设置为0。

3、通过查找出值为1的最长对角线就能找到最长公共子串。

矩阵：

![](http://mycsdnblog.work/201919151944-Q.png)

从上图可以看到，str1和str2共有5个公共子串，但最长的公共子串长度为5。

为了进一步优化算法的效率，我们可以再计算某个二维矩阵的值的时候顺便计算出来当前最长的公共子串的长度，这样就避免了后续查找对角线长度的操作了。修改后的二维矩阵如下：

![](http://mycsdnblog.work/201919151945-i.png)

代码：

```java
    private static String subStr(String str1, String str2) {
        int row = str1.length();
        int col = str2.length();
        int[][] result = new int[row][col];
        int maxLength = 0;
        int end = 0;
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (str1.charAt(i) == str2.charAt(j)){
                    if (i == 0 || j == 0){
                        result[i][j] = 1;
                    }else {
                        result[i][j] = result[i - 1][j - 1] + 1;
                    }
                }
                if (result[i][j] > maxLength){
                    maxLength = result[i][j];
                    end = j;
                }
            }
        }
        return str2.substring(end - maxLength + 1, end + 1);
    }
```

通过记录最大长度，以及最大长度的位置，来截取最后的字符串

# 二、最长公共子序列（LCS）

子序列可以不连续，所以在状态转移方程上有所改变：

![](http://mycsdnblog.work/201919152028-0.png)

如果两个对应字符相等，那么依然是C[i-1,j-1]+1，如果不相同的话，为了保持上一个相同的状态，那么就需要复制上一个状态，上一个状态在哪里？就是在当前位置的左侧和上侧，这两个里面取最大的即可。

![](http://mycsdnblog.work/201919152102-b.png)

代码：

```java
private static int subStr(String str1, String str2) {
    int row = str1.length() + 1;
    int col = str2.length() + 1;
    int[][] result = new int[row][col];
    for (int i = 1; i < row; i++) {
        for (int j = 1; j < col; j++) {
            if (str1.charAt(i - 1) == str2.charAt(j - 1)){
                result[i][j] = result[i - 1][j - 1] + 1;
            }else {
                result[i][j] = Math.max(result[i-1][j],result[i][j-1]);
            }
        }
    }
    return result[row - 1][col - 1];
}
```

通过result数组，回溯法找到所有子序列。

# 三、最长递增子序列（LIS）

# 四、最长递增公共子序列