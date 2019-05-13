# 11、盛水最多的容器

思路：面积取决于两个指针之间的距离和最小的高度，如果高度较大的一侧向内移动的话，那么最终面积一定减小，如果移动小的一侧，那么虽然距离减小，但是可能高度会增加，所以最后面积会变大。

```java
package leetcode.problem11;

class Solution {
    public int maxArea(int[] height) {
        int length = height.length;
        int i = 0;
        int j = length - 1;
        int max = 0;
        while (i < j){
            int h = Math.min(height[i],height[j]);
            max = Math.max(max, h * (j - i));
            if (height[i] < height[j]){
                i++;
            }else {
                j--;
            }
        }
        return max;
    }
}
```

# 43、字符串相乘

思路一：按照最基本的乘法运算进行。提供两个方法：字符串加法和字符串乘法。

乘法的时候注意最后一位，不能直接计算然后反转，必须反正过后将第一位与前面计算的结果反转后进行拼接

```java
import java.util.ArrayList;
import java.util.List;

class Solution {
   public static String multiply(String num1, String num2) {
       if ("0".equals(num1) || "0".equals(num2)){
            return "0";
        }
        List<String> list = new ArrayList<>();
        for (int i = num2.length() - 1; i >= 0 ; i--) {
            StringBuilder temp = new StringBuilder(compute(num1, num2.charAt(i)));
            for (int j = 0; j < num2.length() - i - 1; j++) {
                temp.append("0");
            }
            list.add(temp.toString());
        }
        String result = "";
        for (int i = 0; i < list.size(); i++) {
            //System.out.println(list.get(i));
            result = add(result,list.get(i));
            //System.out.println("result:" + result);
        }
        return result;
    }

    private static String add(String result, String s) {
        int tag = 0;
        StringBuilder sb = new StringBuilder();
        int i = result.length() - 1;
        int j = s.length() - 1;
        while (i >= 0 || j >= 0){
            int t1 = 0,t2 = 0;
            if (i >= 0){
                t1 = result.charAt(i) - '0';
            }
            if (j >= 0){
                t2 = s.charAt(j) - '0';
            }
            int temp = t1 + t2 + tag;
            if (temp > 9){
                temp -= 10;
                tag = 1;
            }else {
                tag = 0;
            }
            sb.append(temp);
            i--;j--;
        }
        if (tag != 0){
            sb.append(tag);
        }
        return sb.reverse().toString();
    }


    private static String compute(String num1, char charAt) {
        int tag = 0;
        int length = num1.length();
        StringBuilder sb = new StringBuilder();
        for (int i = length - 1; i >= 1; i--) {
            int temp = (num1.charAt(i) - '0') * (charAt - '0') + tag;
            if (temp > 9){
                tag = temp / 10;
                temp %= 10;
            }else {
                tag = 0;
            }
            sb.append(temp);
        }
        int temp = (num1.charAt(0) - '0') * (charAt - '0') + tag;
        return temp + sb.reverse().toString();
    }
}
```

# 46、全排列

```java


import java.util.*;

class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        int length = nums.length;
        if (length == 0){
            return result;
        }
        solve(nums,0,result);
        return result;
    }

    private void solve(int[] nums, int k, List<List<Integer>> result) {
        if (k == nums.length){
            List<Integer> list = new ArrayList<>();
            for (int num : nums){
                list.add(num);
            }
            result.add(list);
            return;
        }
        Set<Integer> set = new HashSet<>();
        for (int i = k; i < nums.length; i++) {
            if (!set.contains(nums[i])) {
                set.add(nums[i]);
                swap(nums, k, i);
                solve(nums, k + 1, result);
                swap(nums, k, i);
            }
        }
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

# 48、旋转图像

![](http://mycsdnblog.work/201919282227-w.png)

规律：
旋转90°即：A[0,0] 转到 A[0,n] 位置；A[0,n] 转到 A[n,n] 位置；A[n,n] 转到 A[n,0] 位置；A[n,0] 转到 A[0,0] 位置。然后依次类推

上一步操作的是最外层的一层 环，我们只需要一层层往里执行相同的操作，最终即可完成整个矩阵的旋转

假设矩阵是 n*n 的，那么我们对 n/2 个环执行旋转即可完成

对于任一层的环，假如其实索引为 start，终止索引为 end，那么左上右下四个点分别可有表示为：`A[start][start]`，`A[start][end]`，`A[end][start]`，`A[end][end]`

某层环内的循环规律即 `A[start][start->end]`，`A[start->end][end]`，`A[end->start][start]`，`A[end->start][end]`。箭头表示递变情况



# 66、加一

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class Solution {
    public int[] plusOne(int[] digits) {
        List<Integer> list = new ArrayList<>();
        int length = digits.length;
        int tag = 0;
        int last = digits[length - 1];
        if (last + 1 >= 10){
            list.add(last + 1 - 10);
            tag = 1;
        }else {
            list.add(last + 1);
        }
        for (int i = length - 2; i >= 0 ; i--) {
            int temp = digits[i] + tag;
            if (temp >= 10){
                tag = 1;
                temp -= 10;
            }else {
                tag = 0;
            }
            list.add(temp);
        }
        
        if (tag != 0){
            list.add(tag);
        }
        int[] result = new int[list.size()];
        for (int i = 0; i < list.size(); i++) {
            result[i] = list.get(list.size() - i - 1);
        }
        return result;
    }
}
```

# 67、二进制求和

从后往前扫描，设置一个tag表示进位，取出每一位进行运算，相加的时候要加上进位，然后判断结果值是否大于1，大于的话要计算进位，同时也要将结果减2，如果不产生进位，那么就将tag位置0，因为最后需要判断tag位中是否还保存了进位。

```java
class Solution {
    public String addBinary(String a, String b) {
        // 进位
        int tag = 0;
        StringBuilder sb = new StringBuilder();
        int i = a.length() - 1;
        int j = b.length() - 1;
        while (i >= 0 || j >= 0){
            int t1 = 0,t2 = 0;
            if (i >= 0) {
                 t1 = a.charAt(i) - '0';
            }
            if (j >= 0) {
                 t2 = b.charAt(j) - '0';
            }
            int temp = t1 + t2 + tag;
            if (temp > 1){
                temp -= 2;
                tag = 1;
            }else {
                tag = 0;
            }
            sb.append(temp);
            i--;
            j--;
        }
        if (tag != 0){
            sb.append(tag);
        }
        return sb.reverse().toString();
    }
}
```

