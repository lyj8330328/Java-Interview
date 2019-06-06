# 1、Two Sum

```java
package com.problem1;

import java.util.HashMap;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] result = new int[2];
        HashMap<Integer,Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (map.keySet().contains(target - nums[i])){
                result[0] = i;
                result[1] = map.get(target - nums[i]);
            }else {
                map.put(nums[i],i);
            }
        }
        return result;
    }
}
```

# 2、Add Two Numbers

```java 
package com.problem2;

import com.ListNode;

class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        int pre = 0;
        ListNode p1 = l1;
        ListNode p2 = l2;
        ListNode result = new ListNode(0);
        ListNode p3 = result;
        int temp = 0;
        while (p1 != null || p2 != null){
            temp += p1 == null ? 0 : p1.val;
            temp += p2 == null ? 0 : p2.val;
            temp += pre;
            pre = temp / 10;
            temp %= 10;
            p3.next = new ListNode(temp);
            p3 = p3.next;
            p1 = p1 == null ? null : p1.next;
            p2 = p2 == null ? null : p2.next;
            temp = 0;
        }
        if (pre != 0){
            p3.next = new ListNode(pre);
            p3 = p3.next;
        }
        p3.next = null;
        return result.next;

    }
}
```

# 3、Longest Substring Without Repeating Characters

```java
package com.problem3;

import java.util.HashMap;
import java.util.LinkedList;

class Solution {

    public int lengthOfLongestSubstring(String s) {
        int result = 0;
        int length = s.length();
        int j = 0;
        LinkedList<Character> list = new LinkedList<>();
        while (j < length){
            if (!list.contains(s.charAt(j))){
                list.addLast(s.charAt(j));
                result = Math.max(result, list.size());
                j++;
            }else {
                list.removeFirst();
            }
        }
        return result;
    }

    public int lengthOfLongestSubstring2(String s) {
        HashMap<Character,Integer> map = new HashMap<>();
        int result = 0;
        int length = s.length();
        for (int i = 0,j = 0; i < length; i++) {
            if (map.containsKey(s.charAt(i))){
                j = Math.max(map.get(s.charAt(i)),j);
            }
            result = Math.max(result, i - j + 1);
            map.put(s.charAt(i), i + 1);
        }
        return result;
    }
}
```

# 4、Median of Two Sorted Arrays

```java
package com.problem4;

class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length;
        int n = nums2.length;
        if (m > n){
            int[] temp = nums1;
            nums1 = nums2;
            nums2 = temp;

            int temp2 = m;
            m = n;
            n = temp2;
        }
        int imin = 0,imax = m,half = (m+n+1)/2;
        while (imin <= imax){
            int i = (imax + imin) / 2;
            int j = half - i;
            if (i > imin && nums1[i - 1] > nums2[j]){
                imax = i - 1;
            }else if (i < imax && nums2[j-1] > nums1[i]){
                imin = i + 1;
            }else {
                //找左边的最大值
                int leftMax = 0;
                if (i == 0){
                    leftMax = nums2[j - 1];
                }else if (j == 0){
                    leftMax = nums1[i - 1];
                }else {
                    leftMax = Math.max(nums1[i - 1],nums2[j - 1]);
                }
                if ((m + n) % 2 == 1){
                    return leftMax;
                }

                //找到右边的最小值
                int rightMin = 0;
                if (i == m){
                    rightMin = nums2[j];
                }else if (j == n){
                    rightMin = nums1[i];
                }else {
                    rightMin = Math.min(nums1[i],nums2[j]);
                }
                return (leftMax + rightMin) / 2.0;
            }
        }
        return 0;
    }
}
```

思路：https://blog.csdn.net/lyj2018gyq/article/details/85770457

# 5、Longest Palindromic Substring

```java
package com.problem5;

class Solution {
    public String longestPalindrome(String s) {
        int length = s.length();
        if (length == 0){
            return s;
        }
        int start = 0;
        int maxLength = 0;
        boolean[][] result = new boolean[length][length];
        for (int i = 0; i < length; i++) {
            result[i][i] = true;
            if (i < length - 1 && s.charAt(i) == s.charAt(i + 1)){
                result[i][i+1] = true;
                start = i;
                maxLength = 2;
            }
        }

        for (int len = 3; len <= length; len++){
            for (int i = 0; i <= length - len ; i++) {
                int j = i + len - 1;
                if (s.charAt(i) == s.charAt(j) && result[i+1][j-1]){
                    result[i][j] = true;
                    start = i;
                    maxLength = len;
                }
            }
        }
        if (start == 0 && maxLength == 0){
            return s.charAt(length - 1) + "";
        }else {
            return s.substring(start, start + maxLength);
        }
    }
}
```

思路：https://blog.csdn.net/lyj2018gyq/article/details/83827877

# 7、Reverse Integer

```java
package com.problem7;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public int reverse(int x) {
        long result = 0;
        boolean tag = false;
        if (x < 0){
            tag = true;
            x = -x;
        }
        List<Integer> list = new ArrayList<>();
        while (x != 0){
            list.add(x % 10);
            x /= 10;
        }
        for (int i = list.size() - 1; i >= 0; i--) {
            result += Math.pow(10, i) * list.get(list.size() - i - 1);
        }
        if (tag){
            result = -result;
        }
        if (result > Integer.MAX_VALUE || result < Integer.MIN_VALUE){
            return 0;
        }
        return (int) result;
    }

    /**
     * pop operation:
     * pop = x % 10;
     * x /= 10;
     *
     * push operation:
     * result = result * 10 + pop;
     * @param x
     * @return
     */
    public int reverse2(int x) {
        long result = 0;
        boolean tag = false;
        if (x < 0){
            tag = true;
            x = -x;
        }
        while (x != 0){
            result = result * 10 + x % 10;
            x /= 10;
        }
        if (tag){
            result = -result;
        }
        if (result > Integer.MAX_VALUE || result < Integer.MIN_VALUE){
            return 0;
        }
        return (int) result;
    }
}
```

# 8、String to Integer (atoi)

```java
package com.problem8;

class Solution {

    /**
     * 1.先找到第一个非空格字符的下标index,然后进行判断，如果是+或者-，那么就保存一下正负，然后index后移
     * 2.开始转换，从index开始必须是数字才能转换，否则退出，所以每一次循环要进行判断
     * 3.在循环过程中可能导致数字越界，越long类型的也有可能，所以要提前判断
     * 4.最终再一次判断
     * @param str
     * @return
     */
    public int myAtoi(String str) {
        long result = 0;
        boolean tag = false;
        if (str.length() == 0){
            return 0;
        }
        int index = 0;
        for (int i = 0; i < str.length(); i++) {
            if (str.charAt(i) != ' '){
                index = i;
                break;
            }
        }

        char first = str.charAt(index);
        if (first == '+' || first =='-'){
            tag = first == '-';
            index++;
            if (index >= str.length()){
                return 0;
            }
        }

        if (str.charAt(index) >= '0' && str.charAt(index) <= '9'){
            while (index < str.length() && (str.charAt(index) >= '0' && str.charAt(index) <= '9')){
                if (result > Integer.MAX_VALUE){
                    return tag ? Integer.MIN_VALUE : Integer.MAX_VALUE;
                }
                result = result * 10 + str.charAt(index) - '0';
                index++;
            }
        }else {
            return 0;
        }
        result = tag ? -result : result;
        if (result > Integer.MAX_VALUE){
            return Integer.MAX_VALUE;
        }else if (result < Integer.MIN_VALUE){
            return Integer.MIN_VALUE;
        }
        return (int) result;
    }
}
```

# 11、Container With Most Water

```java
package com.problem11;

class Solution {
    public int maxArea(int[] height) {
        int length = height.length;
        int result = 0;
        int i = 0, j = length - 1;
        while (i < j){
            int temp = Math.min(height[i],height[j]) * (j - i);
            result = Math.max(result, temp);
            if (height[i] < height[j]){
                i++;
            }else {
                j--;
            }
        }
        return result;
    }
}
```

# 13、Roman to Integer

```java
package com.problem13;

import java.util.HashMap;

class Solution {

    public static int romanToInt(String s) {
        HashMap<Character,Integer> map = new HashMap<>();
        map.put('I', 1);
        map.put('V', 5);
        map.put('X', 10);
        map.put('L', 50);
        map.put('C', 100);
        map.put('D', 500);
        map.put('M', 1000);
        int length = s.length();
        int result = 0;
        int temp1 = 0;
        int temp2 = 0;
        int index = 0;
        while (index < length){
            temp1 = map.get(s.charAt(index));
            if (index + 1 < length) {
                temp2 = map.get(s.charAt(index+1));
            }
            if (temp1 >= temp2){
                result += temp1;
                index++;
            }else {
                result += (temp2 - temp1);
                index+=2;
            }
            temp2 = 0;
        }
        return result;
    }

}
```

# 14、Longest Common Prefix

纵向扫描即可

```java
package com.problem14;

class Solution {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 0){
            return "";
        }
        for (int i = 0; i < strs[0].length(); i++) {
            char c = strs[0].charAt(i);
            for (int j = 1; j < strs.length; j++) {
                if (i == strs[j].length() || strs[j].charAt(i) != c){
                    return strs[0].substring(0, i);
                }
            }
        }
        return strs[0];
    }
}
```

# 15、3Sum

```java
package com.problem15;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] > 0){
                break;
            }
            int temp = 0 - nums[i];
            int start = i + 1;
            int end = nums.length - 1;
            while (start < end){
                int temp2 = nums[start] + nums[end];
                if (temp2 == temp){
                    List<Integer> list = new ArrayList<>();
                    list.add(nums[i]);
                    list.add(nums[start]);
                    list.add(nums[end]);
                    if (!result.contains(list)) {
                        result.add(list);
                    }
                    while (start < end && nums[start] == nums[start + 1]){
                        start++;
                    }
                    while (start < end && nums[end] == nums[end - 1]){
                        end--;
                    }
                    start++;
                    end--;
                }else if(temp2 > temp){
                    end--;
                }else {
                    start++;
                }
            }
        }
        return result;
    }
}
```

# 17、Letter Combinations of a Phone Number

普通版本：做笛卡儿积

```java
package com.problem17;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {

    public static void main(String[] args) {
        System.out.println(letterCombinations("23"));
    }

    public static List<String> letterCombinations(String digits) {
        List<String> num = Arrays.asList("abc","def","ghi","jkl","mno","pqrs","tuv","wxyz");
        List<String> result = new ArrayList<>();
        for (int i = 0; i < digits.length(); i++) {
            int temp = digits.charAt(i) - '0';
            String temp2 = num.get(temp - 2);
            result = solve(result,temp2);
        }
        return result;
    }

    private static List<String> solve(List<String> result, String str) {
        List<String> temp = new ArrayList<>();
        if (result.size() == 0){
            for (int i = 0; i < str.length(); i++) {
                temp.add(String.valueOf(str.charAt(i)));
            }
        }else {
            for (String s : result) {
                for (int j = 0; j < str.length(); j++) {
                    temp.add(s + str.charAt(j));
                }
            }
        }
        return temp;
    }
}
```

递归版本：

```java
package com.problem17;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution2 {

    public static void main(String[] args) {
        System.out.println(letterCombinations("23"));
    }

    public static List<String> letterCombinations(String digits) {
        List<String> num = Arrays.asList("abc","def","ghi","jkl","mno","pqrs","tuv","wxyz");
        List<String> result = new ArrayList<>();
        if (digits.length() == 0){
            return result;
        }
        solve(result, "",digits,num);
        return result;
    }

    private static void solve(List<String> result, String cur, String digits, List<String> num) {
        if (cur.length() == digits.length()){
            result.add(cur);
            return;
        }
        int index = digits.charAt(cur.length()) - '0' - 2;
        String string = num.get(index);
        for (int i = 0; i < string.length(); i++) {
            solve(result, cur + string.charAt(i), digits, num);
        }
    }

}
```

# 19、Remove Nth Node From End of List

```java
package com.problem19;

import com.ListNode;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        int count = 0;
        ListNode p1 = head;
        ListNode p2 = head;
        while (p2 != null){
            if (count < n + 1){
                p2 = p2.next;
                count++;
            }else {
                p1 = p1.next;
                p2 = p2.next;
            }
        }
        if (count == n + 1){
            p1.next = p1.next.next;
            return head;
        }else {
            return head.next;
        }
    }
}
```

# 20、Valid Parentheses

```java
package com.problem20;

import java.util.*;

class Solution {
    public boolean isValid(String s) {
        Map<Character,Character> map = new HashMap<>(3);
        Stack<Character> stack = new Stack<>();
        map.put('(', ')');
        map.put('[', ']');
        map.put('{', '}');
        for (int i = 0; i < s.length(); i++) {
            if (map.containsKey(s.charAt(i))){
                stack.push(s.charAt(i));
            }else {
                char c = stack.isEmpty() ? '#' : stack.pop();
                if (c == '#' || map.get(c) != s.charAt(i)){
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }
}
```

# 21、Merge Two Sorted Lists

```java
package com.problem21;

import com.ListNode;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null){
            return l2;
        }
        if (l2 == null){
            return l1;
        }
        ListNode head;
        if (l1.val < l2.val){
            head = l1;
            head.next = mergeTwoLists(l1.next, l2);
        }else {
            head = l2;
            head.next = mergeTwoLists(l1,l2.next);
        }
        return head;
    }
}
```

# 22、Generate Parentheses

```java
package com.problem22;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        getAll(new char[2 * n],0,result);
        return result;
    }

    private void getAll(char[] chars, int i, List<String> result) {
        if (i == chars.length){
            if (valid(chars)) {
                result.add(new String(chars));
            }
            return;
        }else {
            chars[i] = '(';
            getAll(chars, i + 1, result);
            chars[i] = ')';
            getAll(chars, i + 1, result);
        }
    }

    private boolean valid(char[] chars) {
        int count = 0;
        for (char c: chars){
            if (c == '('){
                count++;
            }else if (c == ')'){
                count--;
            }if (count < 0){
                return false;
            }
        }
        return count == 0;
    }
}
```

```java
package com.problem22;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-05-19 22:46
 * @Feature:
 */
public class Solution2 {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        solve(result,"",0,0,n);
        return result;
    }

    private void solve(List<String> result, String s, int start, int end, int n) {
        if (s.length() == 2 * n){
            result.add(s);
            return;
        }
        if (start < n){
            solve(result, s + "(", start + 1, end, n);
        }
        if (end < start){
            solve(result, s + ")", start, end + 1, n);
        }
    }
}
```

# 26、Remove Duplicates from Sorted Array

```java
package com.problem26;

class Solution {
    public int removeDuplicates(int[] nums) {
        int length = nums.length;
        if (length == 0){
            return 0;
        }
        int i = 0;
        int pre = nums[i];
        for (int j = 1; j < length; j++) {
            if (nums[j] != pre){
                pre= nums[j];
                i++;
                swap(nums,i,j);
            }
        }
        return i+1;
    }

    private void swap(int[] nums, int i, int j) {
        if (nums[i] != nums[j]) {
            nums[i] = nums[i] ^ nums[j];
            nums[j] = nums[i] ^ nums[j];
            nums[i] = nums[i] ^ nums[j];
        }
    }
}
```

# *28、Implement strStr()

思路一：暴力破解

```java
package com.problem28;

class Solution {
    public int strStr(String haystack, String needle) {
        if (needle.length() == 0){
            return 0;
        }
        for (int i = 0; i < haystack.length(); i++) {
            if (haystack.charAt(i) == needle.charAt(0)){
                int j = i + 1;
                for (int k = 1; k < needle.length(); k++) {
                    if (j < haystack.length() && needle.charAt(k) == haystack.charAt(j)){
                        j++;
                    }
                }
                if (j - i == needle.length()){
                    return i;
                }
            }
        }
        return -1;
    }
}
```

思路二：找到第一个相同字符的位置，然后在主串中截取等长的子串进行比较

```java
public int strStr2(String haystack, String needle) {
        if (needle.length() == 0){
            return 0;
        }
        for (int i = 0; i < haystack.length(); i++) {
            if (haystack.charAt(i) == needle.charAt(0)){
                if (i + needle.length() - 1< haystack.length()) {
                    if (haystack.substring(i, i + needle.length()).equals(needle)) {
                        return i;
                    }
                }
            }
        }
        return -1;
    }
```

思路三：KMP算法

# *29、Divide Two Integers

```java
package com.problem29;

class Solution {

    public int divide(int dividend, int divisor) {
        if (dividend == 0){
            return 0;
        }
        long result = 0;
        boolean tag = true;
        if ((dividend < 0 && divisor > 0) || (dividend > 0 && divisor < 0)){
            tag = false;
        }
        long dividendTemp = dividend;
        dividendTemp = dividendTemp < 0 ? -dividendTemp : dividendTemp;
        long divisorTemp = divisor;
        divisorTemp = divisorTemp < 0 ? -divisorTemp : divisorTemp;
        while (dividendTemp >= divisorTemp){
            int k = 1;
            long t = divisorTemp;
            while (dividendTemp >= t){
                dividendTemp -= t;
                result += k;
                t += t;
                k += k;
            }
        }
        if (tag){
            if (result > Integer.MAX_VALUE){
                return Integer.MAX_VALUE;
            }else {
                return (int) result;
            }
        }else {
            result = -result;
            if (result < Integer.MIN_VALUE){
                return Integer.MAX_VALUE;
            }else {
                return (int) result;
            }
        }
    }
}
```

# 33、Search in Rotated Sorted Array

```java
package com.problem33;

class Solution {
    public int search(int[] nums, int target) {
        int length = nums.length;
        int i = 0,j = length - 1;
        while (i <= j){
            int mid = (i + j) / 2;
            if (nums[mid] == target){
                return mid;
            }else if (nums[mid] < nums[j]){
                //右半部分有序
                if (target <= nums[j] && target > nums[mid]){
                    i = mid + 1;
                }else {
                    j = mid - 1;
                }
            }else{
                //左半部分有序
                if (target >= nums[i] && target < nums[mid]){
                    j = mid - 1;
                }else {
                    i = mid + 1;
                }
            }
        }
        return -1;
    }
}
```

# 34、Find First and Last Position of Element in Sorted Array

```java
package com.problem34;

class Solution {
    
    public int[] searchRange(int[] nums, int target) {
        //折半查找第一个比target大的数字
        //折半查找第一个比target小的数字
        int[] result = new int[2];
        int length = nums.length;
        double temp = target + 0.5;
        int start = 0;
        int end = length - 1;
        while (start <= end){
            int mid = (start + end) / 2;
            if (nums[mid] > temp){
                end = mid - 1;
            }else if (nums[mid] <= temp){
                start = mid + 1;
            }
        }
        result[1] = end;

        temp = target - 0.5;
        start = 0;
        end = length - 1;
        while (start <= end){
            int mid = (start + end) / 2;
            if (nums[mid] <= temp){
                start = mid + 1;
            }else if (nums[mid] > temp){
                end = mid - 1;
            }
        }
        result[0] = start;

        if (result[1] >= result[0]){
            return result;
        }else {
            return new int[]{-1,-1};
        }
    }
}
```

# 36、Valid Sudoku

``` Java
class Solution {
    public boolean isValidSudoku(char[][] board) {
         
        /**
         * 1.确保每一个子数独中没有重复的数字
         * 2.确保每一行没有重复的数字
         * 3.确保每一列没有重复的数字
         */
        HashMap<Integer,Integer>[] row = new HashMap[9];
        HashMap<Integer,Integer>[] col = new HashMap[9];
        HashMap<Integer,Integer>[] map = new HashMap[9];
        for (int i = 0; i < 9; i++) {
            map[i] = new HashMap<Integer,Integer>(9);
            row[i] = new HashMap<Integer,Integer>(9);
            col[i] = new HashMap<Integer,Integer>(9);
        }
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                char c = board[i][j];
                if (c != '.'){
                    int index = c - '0';
                    int boxIndex = (i / 3 ) * 3 + j / 3;
                    row[i].put(index, row[i].getOrDefault(index, 0) + 1);
                    col[j].put(index, col[j].getOrDefault(index, 0) + 1);
                    map[boxIndex].put(index, map[boxIndex].getOrDefault(index, 0) + 1);

                    if (row[i].get(index) > 1 || col[j].get(index) > 1 || map[boxIndex].get(index) > 1){
                        return false;
                    }
                }
            }
        }
        return true;
    }
}
```

**重点是如何确定子数独：int boxIndex = (i / 3 ) * 3 + j / 3;**

# 38、Count and Say

```java
package com.problem38;

import java.util.ArrayList;
import java.util.List;

class Solution {

    public String countAndSay(int n) {
        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("11");
        StringBuilder sb = new StringBuilder();
        for (int i = 2; i < n; i++) {
            String temp = list.get(i - 1);
            char pre = temp.charAt(0);
            int count = 1;
            for (int j = 1; j < temp.length(); j++) {
                if (temp.charAt(j) != pre){
                    sb.append(count).append(pre);
                    pre = temp.charAt(j);
                    count = 1;
                }else {
                    count++;
                }
            }
            if (count >= 1){
                sb.append(count).append(pre);
            }
            list.add(sb.toString());
            sb = new StringBuilder();
        }
        return list.get(n - 1);
    }
}
```

# 46、Permutations

```java
package com.problem46;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        if (nums.length == 0){
            return result;
        }
        solve(nums,0,result);
        return result;
    }

    private void solve(int[] nums, int i, List<List<Integer>> result) {
        if (i == nums.length){
            ArrayList<Integer> list = new ArrayList<>();
            for (int num : nums) {
                list.add(num);
            }
            result.add(list);
            return;
        }
        for (int j = i; j < nums.length; j++) {
            swap(nums,i,j);
            solve(nums, i + 1, result);
            swap(nums,i,j);
        }
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

# 49、Group Anagrams

思路一：HashMap+排序

```java
package com.problem49;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

class Solution {

    public  List<List<String>> groupAnagrams(String[] strs) {
        List<List<String>> result = new ArrayList<>();
        int length = strs.length;
        if (length == 0){
            return result;
        }
        HashMap<String,List<String>> map = new HashMap<>();
        for (int i = 0; i < length; i++) {
            String temp = strs[i];
            String temp2 = sort(temp);
            if (map.containsKey(temp2)){
                map.get(temp2).add(temp);
            }else {
                ArrayList<String> list = new ArrayList<>();
                list.add(temp);
                map.put(temp2, list);
            }
        }
        for (String key : map.keySet()){
            result.add(map.get(key));
        }
        return result;
    }

    private  String sort(String temp) {
        char[] array = temp.toCharArray();
        for (int i = 1; i < array.length; i++) {
            for (int j = 0; j < array.length - i; j++) {
                if (array[j] > array[j + 1]){
                    char c = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = c;
                }
            }
        }
        return new String(array);
    }


}
```

思路二：统计计数

```java
package com.problem49;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-05-23 10:23
 * @Feature:
 */
public class Solution2 {
    public  List<List<String>> groupAnagrams(String[] strs) {
        int length = strs.length;
        int[] count = new int[26];
        HashMap<String,List<String>> map = new HashMap<>();
        for (int i = 0; i < length; i++) {
            Arrays.fill(count, 0);
            String temp = strs[i];
            for (int j = 0; j < temp.length(); j++) {
                count[temp.charAt(j) - 'a']++;
            }
            StringBuilder sb = new StringBuilder();
            for (int j = 0; j < 26; j++) {
                sb.append(count[j]);
            }
            if (!map.containsKey(sb.toString())){
                map.put(sb.toString(), new ArrayList<>());
            }
            map.get(sb.toString()).add(temp);
        }
        return new ArrayList<>(map.values());
    }
}
```

# 50、Pow(x, n)

```java
package com.problem50;

class Solution {
    public double myPow(double x, int n) {
        if (n == 0){
            return 1.0;
        }
        double result = 1;
        long t = n;
        boolean tag = false;
        if (n < 0) {
            t = -n;
            tag = true;
        }
        for (long i = t; i != 0 ; i /= 2) {
            if (i % 2 != 0){
                result *= x;
            }
            x *= x;
        }
        if (tag){
            result = 1 / result;
        }
        return result;
    }
}
```

# 53、Maximum Subarray

```java
package com.problem53;

class Solution {
    public int maxSubArray(int[] nums) {
        int length = nums.length;
        if (length == 0){
            return 0;
        }
        int result = nums[0];
        int temp = 0;
        for (int num : nums) {
            if (temp > 0) {
                temp += num;
            } else {
                temp = num;
            }
            result = Math.max(result, temp);
        }
        return result;
    }
}
```

# 54、Spiral Matrix

思路一：

```java
package com.problem54;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();
        while (matrix.length > 0) {
            //1.输出第一行
            for (int i = 0; i < matrix[0].length; i++) {
                result.add(matrix[0][i]);
            }
            //2.删除第一行
            matrix = deleteFirstLine(matrix);
            //3.逆时针旋转90°
            matrix = rotate(matrix);
        }
        return result;
    }

    private int[][] rotate(int[][] matrix) {
        int row = matrix.length;
        if (row == 0){
            return new int[0][0];
        }
        int col = matrix[0].length;
        int[][] temp = new int[col][row];
        for (int i = 0; i < col; i++) {
            for (int j = 0; j < row; j++) {
                temp[i][j] = matrix[j][col - i - 1];
            }
        }
        return temp;
    }

    private int[][] deleteFirstLine(int[][] matrix) {
        int[][] temp = new int[matrix.length - 1][matrix[0].length];
        for (int i = 1; i < matrix.length; i++) {
            for (int j = 0; j < matrix[0].length; j++) {
                temp[i - 1][j] = matrix[i][j];
            }
        }
        return temp;
    }
}
```

思路二：比较难想出，了解

```java
package com.problem54;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-05-23 10:58
 * @Feature:
 */
public class Solution2 {

    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> list = new ArrayList<>();
        int row = matrix.length;
        if (row == 0){
            return list;
        }
        int col = matrix[0].length;
        int layers = (Math.min(row, col) - 1) / 2 + 1;
        for (int i = 0; i < layers; i++) {
            //从左往右
            for (int j = i; j < col - i; j++) {
                list.add(matrix[i][j]);
            }
            //右上到右下
            for (int j = i + 1; j < row - i; j++) {
                list.add(matrix[j][col - i - 1]);
            }
            //右到左
            for (int j = col - i - 2; j >= i && row - i - 1 != i; j--) {
                list.add(matrix[row- i - 1][j]);
            }
            //左下到左上
            for (int j = row - i - 2; j > i && col- i - 1 != i ; j--) {
                list.add(matrix[j][i]);
            }
        }
        return list;
    }
}
```

# 55、 Jump Game

思路：使用贪心策略，即移动每个节点所代表的最大步长，然后找出能到达的最大位置，而且当前节点的位置一定不能大于所能到达的最大位置，最后比较最大值和数组长度的大小来进行判断

```java
class Solution {
    public boolean canJump(int[] nums) {
        int length = nums.length;
        if (length == 1){
            return true;
        }
        int start = 0;
        int end = 0;
        while (start <= end && end < length - 1){
            end = Math.max(end,start + nums[start]);
            start++;
        }
        return end >= length - 1;
    }
}
```

# 56、Merge Intervals

思路：什么情况下进行合并？a=[1,3]，b=[2,4]，只要a[1]>=b[0]，那么就可以合并。先将`intervals`按第一列排序，然后进行合并。

```java
package com.problem56;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

class Solution {

    public int[][] merge(int[][] intervals) {
        int row = intervals.length;
        List<int[]> list = new ArrayList<>();
        if (row == 0){
            return new int[0][0];
        }
        Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
        int i = 0;
        while (i < row){
            int left = intervals[i][0];
            int right = intervals[i][1];
            int j = i + 1;
            while (j < row && intervals[j][0] <= right){
                right = Math.max(right, intervals[j][1]);
                j++;
            }
            i = j - 1;
            list.add(new int[]{left,right});
            i++;
        }
        return list.toArray(new int[list.size()][]);
    }
}
```

# 58、Length of Last Word

```java
package com.problem58;

class Solution {
    public int lengthOfLastWord(String s) {
        String[] temp = s.split(" ");
        return temp.length == 0 ? 0 : temp[temp.length - 1].length();
    }

    public int lengthOfLastWord2(String s) {
        String str = s.trim();
        int length = str.length();
        if (length == 0){
            return 0;
        }
        int i = length - 1;
        while (i >= 0){
            if (str.charAt(i) == ' '){
                break;
            }
            i--;
        }
        if (i < 0){
            return length;
        }else {
            return length - i - 1;
        }
    }
}
```

# 66、Plus One

```java
package com.problem66;

class Solution {
    public static int[] plusOne(int[] digits) {
        int pre = 0;
        int length = digits.length;
        if (length == 0){
            return digits;
        }
        for (int i = length - 1; i >= 0; i--) {
            int temp;
            if (i == length - 1) {
               temp = digits[i] + 1 + pre;
            }else {
                temp = digits[i] + pre;
            }
            pre = temp / 10;
            temp = temp % 10;
            digits[i] = temp;
        }
        if (pre > 0){
            int[] result = new int[length + 1];
            result[0] = pre;
            System.arraycopy(digits, 0, result, 1, length);
            return result;
        }
        return digits;
    }
}
```

# 69、Sqrt(x)

思路一：从2到x/2之间查找

```java
package com.problem69;

class Solution {
    
    public int mySqrt(int x) {
        if (x == 1 || x == 0){
            return x;
        }
        int base = x / 2;
        long start = 2;
        while (start <= base){
            long temp = start * start;
            if (temp == x){
                return (int) start;
            }else if (temp < x){
                start++;
            }else if (temp > x || temp > Integer.MAX_VALUE){
                break;
            }
        }
        return (int) (start - 1);
    }
}
```

思路二：既然涉及到了在有序数组中的查找，那么就要使用折半查找

```java
package com.problem69;


class Solution {
    public  int mySqrt(int x) {
        if (x == 1 || x == 0){
            return x;
        }
        int end = x / 2;
        int start = 2;
        while (start <= end){
            int mid = (start + end) / 2;
            int temp = x / mid;
            if (temp == mid){
                return temp;
            }else if (temp < mid){
                end = mid - 1;
            }else {
                start = mid + 1;
            }
        }
        return start - 1;
    }
}
```

**在这里一开始可能会使用`mid*mid`来与x进行比较，但是`mid*mid`很容易越界，所以要进行思路改变，用`x/mid`来与`mid`进行比较。**

# 70、Climbing Stairs

```java
package com.problem70;

class Solution {
    public int climbStairs(int n) {
        if (n <= 2){
            return n;
        }
        int[] result = new int[n + 1];
        result[0] = 0;
        result[1] = 1;
        result[2] = 2;
        for (int i = 3; i <= n; i++) {
            result[i] = result[i - 1] + result[i - 2];
        }
        return result[n];
    }

    public int climbStairs2(int n) {
        if (n == 0 || n == 1){
            return 1;
        }
        return climbStairs(n - 1) + climbStairs(n - 2);
    }
}
```

# 88、Merge Sorted Array

```java
package com.problem88;

class Solution {
    
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = 0,j = 0;
        while (j < n && i < (m + n)){
            if (nums1[i] <= nums2[j] && i < m){
                i++;
            }else if (i > m){
                nums1[i] = nums2[j];
                i++;j++;
            }else {
                //插入nums2[j]
                if (m - i >= 0) {
                    System.arraycopy(nums1, i, nums1, i + 1, m - i);
                }
                nums1[i] = nums2[j];
                j++;
                m++;
            }
        }
    }
}
```

# *92、Reverse Linked List II

思路一：将链表中的值保存到数组中，然后将m和n之间的数字逆序重新赋值

```java
public ListNode reverseBetween(ListNode head, int m, int n) {
    List<Integer> list = new ArrayList<>();
    ListNode p = head;
    while (p != null){
        list.add(p.val);
        p = p.next;
    }
    p = head;
    int count = 0;
    int index = n - 1;
    while (p != null){
        if (count >= m - 1 && count <= n - 1 && index >= m - 1){
            p.val = list.get(index--);
        }
        p = p.next;
        count++;
    }
    return head;
}
```

思路二：重新构建链表，将m前的节点使用尾插法插入到一个新链表中，m和n之间的节点使用头插法插入，n之后的节点使用尾插法插入。因为m和n之间是使用头插法，所以要额外定义一个指针用来扫描当前链表，即找到当前链表的最后一个节点，这样方便将n之后的节点插入。

**注：新构建的链表最后一定要指向null**

```java
public ListNode reverseBetween2(ListNode head, int m, int n) {
    m -= 1;
    n -= 1;
    ListNode p1 = new ListNode(0);
    ListNode head1 = p1;
    ListNode last = head1;


    ListNode index = head;
    ListNode temp;

    int count = 0;
    while (index != null){
        temp = index.next;
       if (count < m){
           head1.next = index;
           head1 = head1.next;
           index = temp;
       }
       if (count >= m && count <= n){
           index.next = head1.next;
           head1.next = index;
           index = temp;
       }
       if (count > n){
           last.next = index;
           last = last.next;
           index = temp;
       }
        if (count <= n && last.next != null) {
            last = last.next;
        }
       count++;
    }
    last.next = null;
   return p1.next;
}
```

# 101、Symmetric Tree

递归比较左右子树，左子树的左节点和右子树的右节点，左子树的右节点和右子树的左节点。

递归版本：

```java
package com.problem101;


import com.TreeNode;

class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null){
            return true;
        }
        return compare(root.left,root.right);
    }

    private boolean compare(TreeNode left, TreeNode right) {
        if (left == null && right == null){
            return true;
        }

        if (left != null && right != null){
            return left.val == right.val && compare(left.left, right.right) && compare(left.right, right.left);
        }
        return false;
    }
}
```

非递归版本（层次遍历）：

```java
package com.problem101;

import com.TreeNode;

import java.util.LinkedList;
import java.util.Queue;
import java.util.Stack;

/**
 * @Author: 98050
 * @Time: 2019-05-25 17:32
 * @Feature:
 */
public class Solution2 {
    public boolean isSymmetric(TreeNode root) {
        if (root == null){
            return true;
        }
        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.addLast(root.left);
        queue.addLast(root.right);
        while (!queue.isEmpty()){
            TreeNode left = queue.pollFirst();
            TreeNode right = queue.pollFirst();
            if (left == null && right == null){
                continue;
            }
            if ((left == null || right == null) || left.val != right.val){
                return false;
            }
            queue.addLast(left.left);
            queue.addLast(right.right);
            queue.addLast(left.right);
            queue.addLast(right.left);
        }
        return true;
    }
}
```

# 104. Maximum Depth of Binary Tree

```java
package com.problem104;

import com.TreeNode;

class Solution {
    public int maxDepth(TreeNode root) {
        if (root == null){
            return 0;
        }
        int left = maxDepth(root.left);
        int right = maxDepth(root.right);
        return Math.max(left, right) + 1;
    }
}
```

# 108. Convert Sorted Array to Binary Search Tree

```java
package com.problem108;

import com.TreeNode;

class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if (nums.length == 0){
            return null;
        }
        return build(nums,0,nums.length - 1);
    }

    private TreeNode build(int[] nums, int start, int end) {
        if (start > end){
            return null;
        }
        if (start == end){
            return new TreeNode(nums[start]);
        }
        int mid = (start + end) / 2;
        TreeNode root = new TreeNode(nums[mid]);
        root.left = build(nums, start, mid - 1);
        root.right = build(nums, mid + 1, end);
        return root;
    }
}
```

# 118. Pascal's Triangle

```java
package com.problem118;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>> result = new ArrayList<>();
        for (int i = 0; i < numRows; i++) {
            result.add(new ArrayList<>());
        }
        for (int i = 0; i < numRows; i++) {
            for (int j = 0; j <= i; j++) {
                if (j == 0){
                    result.get(i).add(1);
                }else {
                    int temp1 = j > result.get(i - 1).size() - 1 ? 0 : result.get(i - 1).get(j);
                    int temp2 = j - 1 < 0 ? 0 : result.get(i - 1).get(j - 1);
                    result.get(i).add(temp1 + temp2);
                }
            }
        }
        return result;
    }
}
```

# 121. Best Time to Buy and Sell Stock

找最小值，然后拿当前值减去最小值得到利润，保存最大的利润

```java
package com.problem121;

class Solution {
    public int maxProfit(int[] prices) {
        int result = 0;
        int length = prices.length;
        if (length == 0){
            return result;
        }
        int min = prices[0];
        for (int i = 0; i < length; i++) {
            min = Math.min(min, prices[i]);
            result = Math.max(result, prices[i] - min);
        }
        return result;
    }
}
```

# 122、Best Time to Buy and Sell Stock II

思路：贪心，只要利润大于0，就卖掉，**同时要重置最小值**

```java
package com.problem122;

class Solution {
    public int maxProfit(int[] prices) {
        int result = 0;
        int length = prices.length;
        if (length == 0){
            return result;
        }
        int min = Integer.MAX_VALUE;
        for (int i = 0; i < length; i++) {
            if (prices[i] > min){
                result += prices[i] - min;
                min = Integer.MAX_VALUE;
            }
            min = Math.min(min, prices[i]);
        }
        return result;
    }
}
```

# 125、Valid Palindrome

```java
package com.problem125;

class Solution {
    public  boolean isPalindrome(String s) {
        String string;
        StringBuilder sb = new StringBuilder();
        for (char c : s.toCharArray()){
            if ((c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z')){
                string = String.valueOf(c);
                sb.append(string.toLowerCase());
            }else if (c >= '0' && c <= '9'){
                sb.append(c);
            }
        }
        String temp1 = sb.toString();
        String temp2 = sb.reverse().toString();
        return temp1.equals(temp2);
    }
}
```

```java
public  boolean isPalindrome2(String s) {
    int length = s.length();
    s = s.toLowerCase();
    int i = 0,j = s.length() - 1;
    while (i < j){
        while (i < length && (s.charAt(i) < 'a' || s.charAt(i) > 'z') && (s.charAt(i) < '0' || s.charAt(i) > '9')){
            i++;
        }

        while (j > 0 && (s.charAt(j) < 'a' || s.charAt(j) > 'z') && (s.charAt(j) < '0' || s.charAt(j) > '9')){
            j--;
        }
        if (i > j){
            return true;
        }
        if (s.charAt(j) != s.charAt(i)){
            return false;
        }else {
            i++;j--;
        }
    }
    return true;
}
```

# 136、Single Number

```java
package com.problem136;

class Solution {
    public int singleNumber(int[] nums) {
        int result = 0;
        for (int i : nums){
            result ^= i;
        }
        return result;
    }
}
```

# 141、Linked List Cycle

https://blog.csdn.net/lyj2018gyq/article/details/88683291

```java
package com.problem141;

import com.ListNode;

import java.util.ArrayList;
import java.util.List;

public class Solution {
    public boolean hasCycle(ListNode head) {
        List<ListNode> listNodes = new ArrayList<>();
        while (head != null){
            if (listNodes.contains(head)){
                return true;
            }else {
                listNodes.add(head);
            }
            head = head.next;
        }
        return false;
    }
}
```

```java
public boolean hasCycle2(ListNode head) {
    if (head == null || head.next == null){
        return false;
    }
    ListNode slow = head;
    ListNode fast = head.next;
    while (slow != fast){
        if (fast == null || fast.next == null){
            return false;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    return true;
}
```

# 155、Min Stack

```java
package com.problem155;

import java.util.Stack;

class MinStack {

    Stack<Integer> stack = new Stack<>();
    Stack<Integer> min = new Stack<>();

    /** initialize your data structure here. */
    public MinStack() {
    }
    
    public void push(int x) {
        stack.push(x);
        if (min.isEmpty()){
            min.push(x);
        }else {
            if (x <= min.peek()){
                min.push(x);
            }
        }
    }
    
    public void pop() {
        if (stack.pop().equals(min.peek())){
            min.pop();
        }
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return min.peek();
    }
}
```

# 160、Intersection of Two Linked Lists

```java
package com.problem160;

import com.ListNode;

import java.util.ArrayList;
import java.util.List;

public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        List<ListNode> list = new ArrayList<>();
        while (headA != null){
            list.add(headA);
            headA = headA.next;
        }
        while (headB != null){
            if (list.contains(headB)){
                return headB;
            }
            headB = headB.next;
        }
        return null;
    }
}
```

```java
public ListNode getIntersectionNode2(ListNode headA, ListNode headB) {

    ListNode p1 = headA;
    ListNode p2 = headB;

    while (p1 != p2){
        p1 = p1 == null ? headB : p1.next;
        p2 = p2 == null ? headA : p2.next;
    }
    return p1;
}
```

# 166、Fraction to Recurring Decimal

# 169、Majority Element

```java
package com.problem169;

class Solution {
    public int majorityElement(int[] nums) {
        int length = nums.length;
        if (length == 0){
            return 0;
        }
        int vote = 1;
        int candidate = nums[0];
        for (int i = 1; i < length; i++) {
            if (nums[i] == candidate){
                vote++;
            }else {
                vote--;
            }
            if (vote == 0){
                vote = 1;
                candidate = nums[i];
            }
        }
        int count = 0;
        for (int i : nums){
            if (i == candidate){
                count++;
            }
        }
        if (count > length / 2){
            return candidate;
        }else {
            return 0;
        }
    }
}
```

# 171、Excel Sheet Column Number

```java
package com.problem171;

class Solution {
    public int titleToNumber(String s) {
        int length = s.length();
        if (length == 0){
            return 0;
        }
        if (length == 1){
            return s.charAt(0) - 'A' + 1;
        }
        int result = 0;
        for (int i = length - 1; i >= 0; i--) {
            int c = s.charAt(i) - 'A' + 1;
            result += c * Math.pow(26, length - i - 1);
        }
        return result;
    }
}
```

# *172、Factorial Trailing Zeroes

不可能去计算出n的阶乘，因为会越界。尾数中0的个数其实是由n中包含5的个数所决定

```java
package com.problem172;

class Solution {

    public int trailingZeroes(int n) {
        if (n < 5){
            return 0;
        }
        int count = 0;
        while (n >= 5){
            count += n / 5;
            n /= 5;
        }
        return count;
    }

}
```

# 189、Rotate Array

```java
package com.problem189;

class Solution {
    public void rotate(int[] nums, int k) {
        int last = nums.length - 1;
        for (int i = 0; i < k; i++) {
            int index = last - 1;
            change(nums, 0, index);
            change(nums, index + 1, nums.length - 1);
            change(nums, 0, nums.length - 1);
        }
        //1 2 3
        //5 6 7 1 2 3 4
    }

    private void change(int[] nums, int i, int j) {
        int temp;
        while (i >= 0 && j < nums.length && i < j){
            temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
            i++;j--;
        }
    }
}
```

# *190、Reverse Bits

求-7二进制

1.先将-7绝对值转换成二进制，得00000111

2.然后求该二进制数的反码，得11111000

3.最后为第二步得到的二进制数+1，结果为11111001

普通思路：求出n的二进制字符串，然后根据正负补全至32位，然后字符串反转，再转换成整数。但是当n为负数的时候，直接转是不对的，会超出Integer的最大值，所以需要借助Long类型，先直接转为无符号的long类型，然后减1，再按位取反。

```java
package com.problem190;

public class Solution {
    
    // you need treat n as an unsigned value
    public static int reverseBits(int n) {
        String str = Integer.toBinaryString(n);
        StringBuilder temp = new StringBuilder();
        if (str.length() < 32 && n > 0){
            for (int i = 0; i < 32 - str.length(); i++) {
                temp.append("0");
            }
        }else if (str.length() < 32 && n < 0){
            for (int i = 0; i < 32 - str.length(); i++) {
                temp.append("1");
            }
        }
        temp.append(str);
        String result = temp.reverse().toString();
        if (result.charAt(0) == '1'){
            return (int) - ~(Long.valueOf(result, 2) - 1);
        }else {
            return Integer.valueOf(result, 2);
        }
    }
}
```

高级思路：

```java
public int reverseBits2(int n) {
    int result = 0;
    for (int i = 0; i <= 32; i++) {
        // 1. 将给定的二进制数,由低到高位逐个取出
        // 1.1 右移 i 位,
        int tmp = n >> i;
        // 1.2  取有效位
        tmp = tmp & 1;
        // 2. 然后通过位运算将其放置到反转后的位置.
        tmp = tmp << (31 - i);
        // 3. 将上述结果再次通过运算结合到一起
        result |= tmp;
    }
    return result;
}
```

# 191、Number of 1 Bits

```Java
package com.problem191;

public class Solution {

    public int hammingWeight(int n) {
        int tag = 1;
        int count = 0;
        while (tag != 0){
            if ((tag & n) != 0) {
                count++;
            }
            tag = tag << 1;
        }
        return count;
    }
}
```

# 198、House Robber

https://blog.csdn.net/lyj2018gyq/article/details/84501461

```java
class Solution {
    public int rob(int[] nums) {
        int length = nums.length;
        if (length == 0){
            return 0;
        }
        if (length == 1){
            return nums[0];
        }
        if (length == 2) {
            return Math.max(nums[0],nums[1]);
        }
        nums[1] = Math.max(nums[0], nums[1]);
        for (int i = 2; i < length; i++) {
            nums[i] = Math.max(nums[i - 1],nums[i] + nums[i - 2]);
        }
        return nums[length - 1];
    }
}
```

# *202、Happy Number

**所有不快乐数的数位平方和计算，最后都会进入 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 的循环中**，**如果发现计算出来的数之前曾经出现过，就说明已经进入了不快乐循环，此时返回 `False`。**

```java
package com.problem202;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public boolean isHappy(int n) {
        List<Integer> list = new ArrayList<>();
        while (!list.contains(n)){
            list.add(n);
            n = solve(n);
        }
        return n == 1;
    }

    private int solve(int n) {
        int result = 0;
        while (n != 0){
            int temp = (n % 10);
            result += temp * temp;
            n /= 10;
        }
        return result;
    }
}
```

# 204、Count Primes

```java
package com.problem204;

import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

class Solution {

    public int countPrimes(int n) {
        List<Integer> list = new ArrayList<>();
        for (int i = 2; i < n; i++) {
            if (judge(i)){
                list.add(i);
            }
        }
        return list.size();
    }

    private boolean judge(int i) {
        for (int j = 2; j * j < i; j++) {
            if (i % j == 0){
                return false;
            }
        }
        return true;
    }
}
```

# *206、Reverse Linked List

```java
package com.problem206;

import com.ListNode;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null){
            return head;
        }
        ListNode root = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return root;
    }

    public ListNode reverseList2(ListNode head) {
        ListNode root = new ListNode(0);
        ListNode temp;
        while (head != null){
            temp = head;
            head = head.next;
            temp.next = root.next;
            root.next = temp;
        }
        return root.next;
    }
}
```

# 215、Kth Largest Element in an Array

利用快排的思想，通过比较k与枢轴下标的大小，然后确定对哪一部分进行递归！！！！！！！！！！

```java
package com.problem215;

import org.junit.Test;

import java.util.Arrays;

class Solution {
    public int findKthLargest(int[] nums, int k) {
        Arrays.sort(nums);
        return nums[nums.length - k];
    }

    public int findKthLargest2(int[] nums, int k) {
        int length = nums.length;
        int start = 0, end = length - 1;
        sort(nums,start,end,k);
        return nums[length - k];
    }

    private void sort(int[] nums, int start, int end,int k) {
        int i = start;
        int j = end;
        if (start < end){
            int temp = nums[start + (end - start)/2];
            while (i != j){
                while (i < j && nums[j] >= temp){
                    j--;
                }
                if (i < j){
                    nums[i] = nums[j];
                    i++;
                }
                while (i < j && nums[i] <= temp){
                    i++;
                }
                if (i < j){
                    nums[j] = nums[i];
                    j--;
                }
            }
            nums[i] = temp;
            if (i == nums.length - k){
                return;
            }
            if (i > nums.length - k){
                sort(nums, start, i - 1, k);
            }
            if (i < nums.length - k) {
                sort(nums, i + 1, end, k);
            }
        }
    }
}
```

快排的改进版本：

```Java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        
        return helper(nums,0,nums.length - 1,nums.length - k);
    }
    
    private int helper(int[] nums,int s,int e,int m){
        if(s >= e){
            return nums[s];
        }
        
        int l = s;
        int r = e;
        int pivot = nums[s + (e - s) / 2];
        while(l <= r){
            while(l <= r && nums[l] < pivot){
                l++;
            }
            
            while(l <= r && nums[r] > pivot){
                r--;
            }

            if(l <= r){
                int tmp = nums[l];
                nums[l] = nums[r];
                nums[r] = tmp;
                l++;
                r--;
            }
        }
        
        if(m <= r){
            return helper(nums,s,r,m);
        }else if(m >= l){
            return helper(nums,l,e,m);
        }else{
            return nums[m];
        }
    }
    
}
```

# 217、Contains Duplicate

使用list.contains其实复杂度就达到了O(n^2)，先排序，再遍历O(nlogn)

```java
package com.problem217;

import java.util.Arrays;

class Solution {
    public boolean containsDuplicate(int[] nums) {
        Arrays.sort(nums);
        for (int i = 0; i < nums.length - 1; i++) {
            if (nums[i] == nums[i+1]){
                return true;
            }
        }
        return false;
    }
}
```

# 234、Palindrome Linked List

找到链表的中间位置，然后将后半部分链表通过头插法构建一个新链表，最后与前半部分链表比较即可。

```java
package com.problem234;

import com.ListNode;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {

    public boolean isPalindrome(ListNode head) {
        //1 2 2 1  1 1 2 1 1
        if (head == null){
            return false;
        }
        //1.计算链表长度
        int length = 0;
        ListNode p1 = head;
        while (p1 != null){
            length++;
            p1 = p1.next;
        }

        if (length == 1){
            return true;
        }
        if (length == 2){
            return head.val == head.next.val;
        }
        //2.找中间位置
        int start;
        if (length % 2 == 0){
            start = length / 2 + 1;
        }else {
            start = length / 2 + 2;
        }
        int count = 1;
        ListNode p2 = head;
        while (count < start){
            p2 = p2.next;
            count++;
        }
        //3.头插法反转start位置后的链表
        ListNode newHead = new ListNode(0);
        ListNode temp;
        while (p2 != null){
            temp = p2;
            p2 = p2.next;
            temp.next = newHead.next;
            newHead.next = temp;
        }
        //4.比较两个链表
        ListNode p3 = newHead.next;
        while (head != null && p3 != null){
            if (head.val != p3.val){
                return false;
            }
            head = head.next;
            p3 = p3.next;
        }
        return true;
    }
}
```

# 237、Delete Node in a Linked List

删除单链表中的指定结点，其只能从该结点开始访问。使用复制的方法，将后一个结点的值赋给前一个，然后把最后一个结点干掉。

```java
package com.problem237;

import com.ListNode;

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public void deleteNode(ListNode node) {
        ListNode last = null;
        while (node.next != null){
            if (node.next.next == null){
                last = node;
            }
            node.val = node.next.val;
            node = node.next;
        }
        last.next = null;
    }
}
```

# 242、Valid Anagram

思路一：hashmap统计然后比较

```java
package com.problem242;

import java.util.HashMap;

class Solution {
    public boolean isAnagram(String s, String t) {
        HashMap<Character,Integer> map = new HashMap<>();
        for (char c : s.toCharArray()){
            if (map.containsKey(c)){
                map.put(c, map.get(c) + 1);
            }else {
                map.put(c, 1);
            }
        }
        HashMap<Character,Integer> map2 = new HashMap<>();
        for (char c : t.toCharArray()){
            if (map2.containsKey(c)){
                map2.put(c, map2.get(c) + 1);
            }else {
                map2.put(c, 1);
            }
        }
        if (map.keySet().size() != map2.keySet().size()){
            return false;
        }
        for (char c : map.keySet()){
            if (!map.get(c).equals(map2.get(c))){
                return false;
            }
        }
        return true;
    }
}
```

思路二：

```java
public boolean isAnagram2(String s, String t) {
    int[] result = new int[26];
    int[] result2 = new int[26];
    if (s.length() != t.length()){
        return false;
    }
    for (int i = 0; i < s.length(); i++) {
        int index = s.charAt(i) - 'a';
        int index2 = t.charAt(i) - 'a';
        result[index]++;
        result2[index2]--;
    }

    for (int i = 0; i < 26; i++) {
        int num = result[i] + result2[i];
        if (num != 0){
            return false;
        }
    }
    return true;
}
```

# 268、Missing Number

```java
package com.problem268;

class Solution {
    public int missingNumber(int[] nums) {
        int sum = (1 + nums.length) * nums.length / 2;
        int temp = 0;
        for (int i : nums){
            temp+=i;
        }
        return sum - temp;
    }
}
```

# *283、Move Zeroes

思路一：将所有非0数字向前移动，时间复杂度O(n^2)

```java
package com.problem283;

class Solution {
    public void moveZeroes(int[] nums) {
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] != 0){
                int j = i;
                while (j - 1 >= 0 && nums[j - 1] == 0){
                    int temp = nums[j - 1];
                    nums[j - 1] = nums[j];
                    nums[j] = temp;
                    j--;
                }
            }
        }
    }
}
```

思路二：不用直接移动元素，将非0数字依次复制到数组前边，然后统计0的个数count，将数组最后count个位置置0

```java
public void moveZeroes2(int[] nums) {
    int index = 0;
    int count = 0;
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] != 0){
            nums[index] = nums[i];
            index++;
        }else {
            count++;
        }
    }
    for (int i = nums.length - count - 1; i < nums.length; i++) {
        nums[i] = 0;
    }
}
```

# 326、Power of Three

```java
package com.problem326;

class Solution {
    public boolean isPowerOfThree(int n) {
        return Integer.toString(n, 3).matches("^10*$");
    }

    public boolean isPowerOfThree2(int n) {
        if (n < 1){
            return false;
        }
        while (n % 3 == 0){
            n /= 3;
        }
        return n == 1;
    }
}
```

# 344、Reverse String

```java
package com.problem344;

class Solution {
    public void reverseString(char[] s) {
        int length = s.length;
        int i = 0, j = length - 1;
        char c;
        while (i < j){
            c = s[i];
            s[i] = s[j];
            s[j] = c;
            i++;
            j--;
        }
    }
}
```

# 350、Intersection of Two Arrays II

HashMap计数

```java
package com.problem350;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        HashMap<Integer,Integer> map = new HashMap<>();
        for (int i : nums1){
            if (map.containsKey(i)){
                map.put(i, map.get(i) + 1);
            }else {
                map.put(i, 1);
            }
        }
        List<Integer> list = new ArrayList<>();
        for (int i : nums2){
            if (map.containsKey(i) && map.get(i) > 0){
                list.add(i);
                map.put(i, map.get(i) - 1);
            }
        }
        int[] result = new int[list.size()];
        for (int i = 0; i < list.size(); i++) {
            result[i] = list.get(i);
        }
        return result;
    }
}
```

# 371、Sum of Two Integers

首先看十进制是如何做的： 5+7=12，三步走 

第一步：相加各位的值，不算进位，得到2。 

第二步：计算进位值，得到10. 如果这一步的进位值为0，那么第一步得到的值就是最终结果。 

第三步：重复上述两步，只是相加的值变成上述两步的得到的结果2和10，得到12。  

同样我们可以用三步走的方式计算二进制值相加： 5-101，7-111 

第一步：相加各位的值，不算进位，得到010，二进制每位相加就相当于各位做异或操作，101^111。  

第二步：计算进位值，得到1010，相当于各位做与操作得到101，再向左移一位得到1010，(101&111)<<1。  

第三步重复上述两步， 各位相加 010^1010=1000，进位值为100=(010&1010)<<1。      

继续重复上述两步：1000^100 = 1100，进位值为0，跳出循环，1100为最终结果。  

```java
package com.problem371;

class Solution {
    public int getSum(int a, int b) {
         while (b != 0){
             int temp = a ^ b;
             b = (a & b) << 1;
             a = temp;
         }
         return a;
    }

}
```

# 387、First Unique Character in a String

```java
package com.problem387;

import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;

class Solution {
    public int firstUniqChar(String s) {
        LinkedHashMap<Character,Integer> map = new LinkedHashMap<>();
        for (char c : s.toCharArray()){
            if (map.containsKey(c)){
                map.put(c, map.get(c) + 1);
            }else {
                map.put(c, 1);
            }
        }
        for (char c : s.toCharArray()){
            if (map.get(c) == 1){
                return s.indexOf(c);
            }
        }
        return -1;
    }
    
    public int firstUniqChar2(String s) {
        int[] result = new int[26];
        int index;
        for (char c : s.toCharArray()){
            index = c - 'a';
            result[index]++;
        }
        for (int i = 0; i < s.length(); i++) {
            if (result[s.charAt(i) - 'a'] == 1){
                return i;
            }
        }
        return -1;
    }
}
```

# 412、Fizz Buzz

```java
package com.problem412;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<String> fizzBuzz(int n) {
        List<String> result = new ArrayList<>();
        for (int i = 1; i <= n; i++) {
            if (i % 3 == 0 && i % 5 == 0){
                result.add("FizzBuzz");
            }else if (i % 5 == 0){
                result.add("Buzz");
            }else if (i % 3 == 0){
                result.add("Fizz");
            }else {
                result.add(String.valueOf(i));
            }
        }
        return result;
    }
}
```

