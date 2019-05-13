# 一、Two Sum

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

# 二、Add Two Numbers

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

# 三、Longest Substring Without Repeating Characters

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

# 四、Median of Two Sorted Arrays

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

# 五、Longest Palindromic Substring

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

