# 一、算法思想

## 1.1 双指针

### 1.1.1 Two Sum II - Input array is sorted（167）

```java
package com.problem167;

class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int i = 0, j = numbers.length - 1;
        int[] result = new int[2];
        int temp;
        while (i < j){
            temp = numbers[i] + numbers[j];
            if (temp == target){
                result[0] = i + 1;
                result[1] = j + 1;
                break;
            }else if (temp < target){
                i++;
            }else{
                j--;
            }
        }
        return result;
    }
}
```

### 1.1.2 Sum of Square Numbers（633）

```java
package com.problem633;

class Solution {
    public boolean judgeSquareSum(int c) {
        int i = 0, j = (int) Math.sqrt(c);
        int temp;
        while (i <= j){
            temp = i * i + j * j;
            if (temp == c){
                return true;
            }else if (temp < c){
                i++;
            }else {
                j--;
            }
        }
        return false;
    }
}
```

### 1.1.3 Reverse String（344）

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

### 1.1.4 Reverse Vowels of a String（345）

```java
package com.problem345;

import java.util.Arrays;
import java.util.List;

class Solution {
    public String reverseVowels(String s) {
        List<Character> list = Arrays.asList('a','e','i','o','u','A','E','I','O','U');
        char[] chars = s.toCharArray();
        int i = 0, j = s.length() - 1;
        char temp;
        while (i < j){
            if (list.contains(chars[i]) && list.contains(chars[j])){
                temp = chars[i];
                chars[i] = chars[j];
                chars[j] = temp;
                i++;j--;
            }else if (!list.contains(chars[j])){
                j--;
            }else if (!list.contains(chars[i])){
                i++;
            }
        }
        return new String(chars);
    }
}
```

### 1.1.5 Valid Palindrome（125）

```java
class Solution {
    public boolean isPalindrome(String s) {
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
        s = sb.toString();
        int i = 0, j = s.length() - 1;
        while (i < j){
            if (s.charAt(i) != s.charAt(j)){
                return false;
            }else {
                i++;
                j--;
            }
        }
        return true;
    }
}
```

### 1.1.6 Valid Palindrome II（680）

**双指针，遇到不相等的字符就分别判断s[i,j-1]或s[i+1,j]是不是回文字符串**

```java
package com.problem680;


class Solution {

    public boolean validPalindrome(String s) {
        int i = 0, j = s.length() - 1;
        while (i < j){
            if (s.charAt(i) != s.charAt(j)){
                return judge(s,i,j - 1) || judge(s,i + 1,j);
            }
            i++;j--;
        }
        return true;
    }

    private boolean judge(String s, int i, int j) {
        while (i < j){
            if (s.charAt(i++) != s.charAt(j--)){
                return false;
            }
        }
        return true;
    }
}
```

### 1.1.8 *Merge Sorted Array（88）

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

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int index = m - 1;
        int index2 = n - 1;
        int total = m + n -1;
        while (index >= 0 || index2 >= 0){
            if (index < 0){
                nums1[total--] = nums2[index2--];
            }else if (index2 < 0){
                nums1[total--] = nums1[index--];
            }else if (nums1[index] < nums2[index2]){
                nums1[total--] = nums2[index2--];
            }else {
                nums1[total--] = nums1[index--];
            }
        }
    }
}
```

### 1.1.9 Linked List Cycle（141）

```java
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null){
        return false;
    }
    ListNode slow = head;
    ListNode fast = head;
    while (slow != null && fast != null){
        slow = slow.next;
        if (fast.next == null){
            return false;
        }
        fast = fast.next.next;
        if (slow == fast){
            return true;
        }
    }
    return false;
}
```

### 1.1.10 通过删除字母匹配到字典里最长单词

[524. Longest Word in Dictionary through Deleting](https://leetcode.com/problems/longest-word-in-dictionary-through-deleting/)

```java
package com.problem524;

import java.util.List;

class Solution {
    public String findLongestWord(String s, List<String> d) {
        String result = "";
        for (String s1 : d){
            int l1 = result.length();
            int l2 = s1.length();
            if (l1 > l2 || (l1 == l2 && s1.compareTo(result) < 0)){
                continue;
            }
            if (judge(s1,s)){
                result = s1;
            }
        }
        return result;
    }

    private boolean judge(String s1, String s) {
        int i = 0, j = 0;
        while (i < s1.length() && j < s.length()){
            if (s1.charAt(i) == s.charAt(j)){
                i++;
            }
            j++;
        }
        return i == s1.length();
    }
}
```

### 1.1.11 判断子序列

[392. Is Subsequence](https://leetcode.com/problems/is-subsequence/)

```java 
package com.problem392;

class Solution {
    public boolean isSubsequence(String s, String t) {
        int i = 0,j = 0;
        while (i < s.length() && j < t.length()){
            if (s.charAt(i) == t.charAt(j)){
                i++;
            }
            j++;
        }
        return i == s.length();
    }
}
```

## 1.2 排序

### 1.2.1 Kth Largest Element in an Array（215）

注：求解Kth Element问题，可以使用快排、堆、排序等方法来完成。

> **排序**

```java
public int findKthLargest(int[] nums, int k) {
    Arrays.sort(nums);
    return nums[nums.length - k];
}
```

时间复杂度为：O(nlogn)，空间复杂度O(1)。

> **堆**

```java
public static int findKthLargest3(int[] nums, int k) {
    //小顶堆
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    for (int i : nums){
        heap.add(i);
        if (heap.size() > k){
            heap.poll();
        }
    }
    return heap.peek();
}
```

时间复杂度O(nlogk)，空间复杂度O(k)

> **快速排序**

因为快速排序每回都可以确定一个元素的最终位置，所以通过比较枢轴与length-k的位置大小来决定对哪一部分进行递归。

```java
private static void sort(int[] nums, int start, int end,int k) {
    int i = start;
    int j = end;
    if (start < end){
        int temp = nums[start];
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
```

时间复杂度O(n)，空间复杂度O(1)

### 1.2.2 Top K Frequent Elements（347）

桶排序

```java
package com.problem347;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        List<Integer> list = new ArrayList<>();
        HashMap<Integer,Integer> map = new HashMap<>();
        for (int i : nums){
            map.put(i, map.getOrDefault(i, 0) + 1);
        }
        ArrayList<Integer>[] buckets = new ArrayList[nums.length + 1];
        for (int i : map.keySet()){
            int index = map.get(i);
            if (buckets[index] == null){
                buckets[index] = new ArrayList<>();
            }
            buckets[index].add(i);
        }
        for (int i = buckets.length - 1; i >= 0; i--) {
            if (buckets[i] != null){
                if (list.size() == k){
                    break;
                }
                if (buckets[i].size() <= (k - list.size())){
                    list.addAll(buckets[i]);
                }else {
                    list.addAll(buckets[i].subList(0, k - list.size()));
                }
            }
        }
        return list;
    }
}
```

### 1.2.3 Sort Characters By Frequency（451）

```java
package com.problem451;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

class Solution {
    
    public  String frequencySort(String s) {
        HashMap<Character,Integer> map = new HashMap<>();
        for (char c : s.toCharArray()){
            map.put(c, map.getOrDefault(c, 0) + 1);
        }
        List<Character>[] buckets = new ArrayList[s.length() + 1];
        for (char c : map.keySet()){
            int index = map.get(c);
            if (buckets[index] == null){
                buckets[index] = new ArrayList<>();
            }
            for (int i = 0; i < index; i++) {
                buckets[index].add(c);
            }
        }
        StringBuilder sb = new StringBuilder();
        for (int i = buckets.length - 1; i >= 0; i--) {
            if (buckets[i] != null){
                for (char c : buckets[i]) {
                    sb.append(c);
                }
            }
        }
        return sb.toString();
    }
}
```

### 1.2.4 Sort Colors（75）

思路：三向切分快速排序

快速排序在实际应用中会面对大量具有重复元素的数组。例如加入一个子数组全部为重复元素，则对于此数组排序就可以停止，但快排算法依然将其切分为更小的数组。这种情况下快排的性能尚可，但存在着巨大的改进潜力。（从O(nlgn)提升到O(n)）

  一个简单的想法就是将数组分为三部分：小于当前切分元素的部分，等于当前切分元素的部分，大于当前切分元素的部分。

  E.W.Dijlstra（对，就是Dijkstra最短路径算法的发明者）曾经提出一个与之相关的荷兰国旗问题（一个数组中有分别代表红白蓝三个颜色的三个主键值，将三个主键值排序，就得到了荷兰国旗的颜色排列）。

  他提出的算法是： 对于每次切分：从数组的左边到右边遍历一次，维护三个指针，其中lt指针使得元素（arr[0]-arr[lt-1]）的值均小于切分元素；gt指针使得元素（arr[gt+1]-arr[N-1]）的值均大于切分元素；i指针使得元素（arr[lt]-arr[i-1]）的值均等于切分元素，（arr[i]-arr[gt]）的元素还没被扫描，切分算法执行到i>gt为止。每次切分之后，位于gt指针和lt指针之间的元素的位置都已经被排定，不需要再去处理了。之后将（lo,lt-1）,（gt+1,hi）分别作为处理左子数组和右子数组的递归函数的参数传入，递归结束，整个算法也就结束。

三向切分的示意图：

![](http://mycsdnblog.work/201919081447-Z.png)

```java
package com.problem75;

class Solution {
    public void sortColors(int[] nums) {
        sortBy3Way(nums,0,nums.length - 1);
    }

    private void sortBy3Way(int[] nums, int i, int j) {
        int low = i,high = j;
        int index = low + 1;
        if (i < j){
            int temp = nums[low];
            while (index <= j){
                if (nums[index] < temp){
                    swap(nums, low++, index++);
                }else if (nums[index] > temp){
                    swap(nums, index, high--);
                }else {
                    index++;
                }
            }
            sortBy3Way(nums, i, low - 1);
            sortBy3Way(nums, high + 1, j);
        }
    }

    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

## 1.3 贪心

### 1.3.1 Assign Cookies（455）

思路：首先将g和s排序，然后从大到小进行分配

```java
package com.problem455;

import java.util.Arrays;

class Solution {
    public int findContentChildren(int[] g, int[] s) {
        Arrays.sort(g);
        Arrays.sort(s);

        int i = g.length - 1;
        int j = s.length - 1;
        int count = 0;
        while (i >= 0 && j >= 0){
            if (g[i] <= s[j]){
                count++;
                j--;
            }
            i--;
        }
        return count;
    }
}
```

### 1.3.2 无重叠区间

[435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)

和56题类似，只不过56题是对重叠区间进行合并：a=[1,3]，b=[2,4]，只要a[1]>=b[0]，那么就可以合并，所以取右边最大的值就可以得到最终的区间，先将`intervals`按第一列排序，然后进行合并。

这道题是删除最小的区间数，使剩下的区间不再重叠，那么每回就让右边取最小，统计可以合并的**最少**区间数即可。

```java
package com.problem435;

import java.util.Arrays;
import java.util.Comparator;

class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        int row = intervals.length;
        int result = 0;
        Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
        int i = 0;
        while (i < row){
            int right = intervals[i][1];
            int j = i + 1;
            while (j < row && intervals[j][0] < right){
                right = Math.min(right, intervals[j][1]);
                result++;
                j++;
            }
            i = j;
        }
        return result;
    }
}
```

贪心：

先计算**最多**能组成的不重叠区间个数，然后用区间总个数减去不重叠区间的个数。

在每次选择中，区间的结尾最为重要，选择的区间结尾越小，留给后面的区间的空间越大，那么后面能够选择的区间个数也就越大。

按区间的结尾进行排序，每次选择结尾最小，并且和前一个区间不重叠的区间。

```java
public int eraseOverlapIntervals2(int[][] intervals) {
    int row = intervals.length;
    if (row == 0){
        return 0;
    }
    Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));
    int result = 1;
    int right = intervals[0][1];
    for (int i = 1; i < row; i++) {
        if (intervals[i][0] < right){
            continue;
        }
        right = intervals[i][1];
        result++;
    }
    return row - result;
}
```

### 1.3.3 用最少数量的箭引爆气球

[452. Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)

和无重叠区间一样，直接计算无重叠区间的个数，即为最少数量的箭，只不过这里边界值相等也算重叠！

```java
package com.problem452;

import java.util.Arrays;
import java.util.Comparator;

class Solution {
    public int findMinArrowShots(int[][] points) {
        int row = points.length;
        if (row == 0){
            return 0;
        }
        Arrays.sort(points, Comparator.comparingInt(a -> a[1]));
        int result = 1;
        int right = points[0][1];
        for (int i = 1; i < row; i++) {
            if (points[i][0] <= right){
                continue;
            }
            right = points[i][1];
            result++;
        }
        return result;
    }
}
```

### 1.3.4 根据身高重建队列

[406. Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/)

为了使插入操作不影响后续的操作，身高较高的学生应该先做插入操作，否则身高较小的学生原先正确插入的第 k 个位置可能会变成第 k+1 个位置。

身高 h 降序、个数 k 值升序，然后将某个学生插入队列的第 k 个位置中。

```java
package com.problem406;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public int[][] reconstructQueue(int[][] people) {
        if (people == null || people.length == 0 || people[0].length == 0){
            return new int[0][0];
        }
        Arrays.sort(people, ((o1, o2) -> o1[0] == o2[0] ? o1[1] - o2[1] : o2[0] - o1[0]));
        List<int[]> list = new ArrayList<>();
        for (int[] p : people){
            list.add(p[1],p);
        }
        return list.toArray(new int[list.size()][]);
    }
}
```

### 1.3.5 买卖股票的最佳时机

[121. Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

要想收益最大，那么就需要找到数组中的最小值和最大值，且最大值的下标必须比最小值的下标大，这样就得到了最大收益。

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
        for (int i : prices){
            min = Math.min(min, i);
            result = Math.max(result, i - min);
        }
        return result;
    }
}
```

### 1.3.6 买卖股票的最佳时机Ⅱ

[122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

对于 [a, b, c, d]，如果有 a <= b <= c <= d ，那么最大收益为 d - a。而 d - a = (d - c) + (c - b) + (b - a) ，因此当访问到一个 prices[i] 且 prices[i] - prices[i-1] > 0，那么就把 prices[i] - prices[i-1] 添加到收益中。

```java
package com.problem122;

class Solution {
    public int maxProfit(int[] prices) {
        int result = 0;
        for (int i = 1; i < prices.length; i++) {
            if (prices[i] > prices[i - 1]){
                result += (prices[i] - prices[i - 1]);
            }
        }
        return result;
    }
}
```

### 1.3.7 种花问题

[605. Can Place Flowers](https://leetcode.com/problems/can-place-flowers/)

思路：找到所有满足条件的位置，然后判断是否大于等于n。

```java
package com.problem605;

class Solution {
    public boolean canPlaceFlowers(int[] flowerbed, int n) {
        int len = flowerbed.length;
        //找到所有满足条件的位置
        int count = 0;
        for (int i = 0; i < len; i++) {
            if (flowerbed[i] == 1){
                continue;
            }
            int pre = i == 0 ? 0 : flowerbed[i - 1];
            int next = i == len - 1 ? 0 : flowerbed[i + 1];
            if (pre == 0 && next == 0){
                count++;
                flowerbed[i] = 1;
            }
        }
        return count >= n;
    }
}
```

### 1.3.8 判断子序列

[392. Is Subsequence](https://leetcode.com/problems/is-subsequence/)

直接判断字符串s中的每个字符是否按顺序在字符串t中出现

```java
public boolean isSubsequence(String s, String t) {
    int index = -1;
    for (char c : s.toCharArray()){
        index = t.indexOf(c, index + 1);
        if (index == -1){
            return false;
        }
    }
    return true;
}
```

### 1.3.9 非递减数列

[665. Non-decreasing Array](https://leetcode.com/problems/non-decreasing-array/)

思路：在出现 nums[i] < nums[i - 1] 时，需要考虑的是应该修改数组的哪个数，使得本次修改能使 i 之前的数组成为非递减数组，并且 **不影响后续的操作** 。优先考虑令 nums[i - 1] = nums[i]，因为如果修改 nums[i] = nums[i - 1] 的话，那么 nums[i] 这个数会变大，就有可能比 nums[i + 1] 大，从而影响了后续操作。还有一个比较特别的情况就是 nums[i] < nums[i - 2]，修改 nums[i - 1] = nums[i] 不能使数组成为非递减数组，只能修改 nums[i] = nums[i - 1]。

[2，3，3，2，4]

[1，4，2，3]

```java
package com.problem665;

class Solution {
    public boolean checkPossibility(int[] nums) {
        int count = 0;
        if (nums.length <= 1){
            return true;
        }
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] < nums[i - 1]){
                count++;
                if (i - 2 >= 0 && nums[i] < nums[i - 2]){
                    nums[i] = nums[i - 1];
                }else {
                    nums[i - 1] = nums[i];
                }
            }
        }
        return count <= 1;
    }
}
```

### 1.3.10 最大子序合

[53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

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

### *1.3.11 划分字母区间

[763. Partition Labels](https://leetcode.com/problems/partition-labels/)

思路：利用hashmap记录字符串中每个字母最后出现的位置，然后再对字符串进行遍历，设置指针right指向子串的结束位置，设置指针left指向子串的开始位置，那么最后分割的子串长度就为`right-left+1`，通过比较当前元素下标和right的大小来进行划分：

- 如果当前扫描的元素的最后一次出现的下标比right大，就说明子字符串还需要延长，就刷新right
- 如果当前扫描的元素的下标已经达到了right，就说明这个子字符串已经找到了

```java
package com.problem763;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

class Solution {
    public List<Integer> partitionLabels(String S) {
        HashMap<Character,Integer> map = new HashMap<>();
        List<Integer> list = new ArrayList<>();
        if (S.length() == 0){
            return list;
        }
        for (int i = 0; i < S.length(); i++) {
            map.put(S.charAt(i), i);
        }

        int right = map.get(S.charAt(0));
        int left = 0;
        for (int i = 0; i < S.length(); i++) {
            right = Math.max(right, map.get(S.charAt(i)));
            if (i >= right){
                list.add(right - left + 1);
                left = right + 1;
            }
        }
        return list;
    }
}
```

### 1.3.12 跳跃游戏

[55. Jump Game](https://leetcode.com/problems/jump-game/)

> Given an array of non-negative integers, you are initially positioned at the first index of the array.
>
> Each element in the array represents your maximum jump length at that position.
>
> Determine if you are able to reach the last index.
>
> **Example 1:**
>
> ```
> Input: [2,3,1,1,4]
> Output: true
> Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.
> ```
>
> **Example 2:**
>
> ```
> Input: [3,2,1,0,4]
> Output: false
> Explanation: You will always arrive at index 3 no matter what. Its maximum
>              jump length is 0, which makes it impossible to reach the last index.
> ```

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

## 1.4 二分查找

### 1.4.1 x的平方根

[69. Sqrt(x)](https://leetcode.com/problems/sqrtx/)

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
        return end;
    }
}
```

### 1.4.2 寻找比目标字母大的最小字母

[744. Find Smallest Letter Greater Than Target](https://leetcode.com/problems/find-smallest-letter-greater-than-target/)

```java
package com.problem744;

class Solution {
    public char nextGreatestLetter(char[] letters, char target) {
        int[] nums = new int[26];
        for (char c : letters){
            nums[c - 'a'] = 1;
        }
        int index = target - 'a';
        int start = index + 1;
        while (true){
            if (start >= nums.length){
                start = 0;
            }
            if (nums[start] != 0){
                return (char) ('a' + start);
            }
            start++;
        }
    }

    public char nextGreatestLetter2(char[] letters, char target) {
        int len = letters.length;
        int low = 0;
        int high = len - 1;
        while (low <= high){
            int mid = (low + high) / 2;
            if (letters[mid] <= target){
                low = mid + 1;
            }else {
                high = mid - 1;
            }
        }
        return high < len - 1 ? letters[high + 1] : letters[0];
    }
}
```

### *1.4.3 有序数组中的单一元素

[540. Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/)

> Given a sorted array consisting of only integers where every element appears exactly twice except for one element which appears exactly once. Find this single element that appears only once.
>
> **Example 1:**
>
> ```
> Input: [1,1,2,3,3,4,4,8,8]
> Output: 2
> ```
>
> **Example 2:**
>
> ```
> Input: [3,3,7,7,10,11,11]
> Output: 10
> ```
>
> **Note:** Your solution should run in O(log n) time and O(1) space.

如果没有时间复杂度要求的话可以用异或来做，时间复杂度为O(n)：

```java
package com.problem540;

class Solution {
    public int singleNonDuplicate(int[] nums) {
        int result = 0;
        for (int i : nums){
            result ^= i;
        }
        return result;
    }
}
```

既然有序，那么可以采用折半查找的方法，找到中间位置mid后，分别计算左右两边元素个数：

`left = mid - low，right = high - mid`，然后通过比较nums[mid]与nums[mid+1]、nums[mid-1]是否相等来确定折半查找的区间：

如果nums[mid]==nums[mid+1]，那么right需要自减一次，然后判断right是否为偶数：

- 如果right为偶数，那么说明出现一次的数只能在左半部分，则折半查找的区间在[low，mid-1]
- 如果right为奇数，那么说明出现一次的数就在右半部分，则折半查找的区间为[mid + 2，high]。为什么加2，因为nums[mid]与nums[mid+1]相等，所以要从mid+2开始查找。

同理，如果nums[mid]==nums[mid-1]，那么left需要自减一次，然后判断left是否为偶数：

- 如果left为偶数，那么说明出现一次的数只能在右半部分，则折半查找的区间在[mid-1，high]
- 如果left为奇数，那么说明出现一次的数就在左半部分，则折半查找的区间为[low，mid-2]。为什么减2，因为nums[mid]与nums[mid-1]相等，所以要从mid-2开始查找。

```java
public static int singleNonDuplicate(int[] nums) {
    int low = 0;
    int high = nums.length - 1;
    while (low < high){
        int mid = (low + high) / 2;
        int left = mid - low;
        int right = high - mid;
        if (nums[mid] == nums[mid + 1]){
            right--;
            if (right % 2 == 0){
                high = mid - 1;
            }else {
                low = mid + 2;
            }
        }else if (nums[mid] == nums[mid - 1]){
            left--;
            if (left % 2 == 0){
                low = mid + 1;
            }else {
                high = mid - 2;
            }
        }else {
            return nums[mid];
        }
    }
    return nums[low];
}
```

### *1.4.4 第一个错误版本

[278. First Bad Version](https://leetcode.com/problems/first-bad-version/)

> You are a product manager and currently leading a team to develop a new product. Unfortunately, the latest version of your product fails the quality check. Since each version is developed based on the previous version, all the versions after a bad version are also bad.
>
> Suppose you have `n` versions `[1, 2, ..., n]` and you want to find out the first bad one, which causes all the following ones to be bad.
>
> You are given an API `bool isBadVersion(version)` which will return whether `version` is bad. Implement a function to find the first bad version. You should minimize the number of calls to the API.
>
> **Example:**
>
> ```
> Given n = 5, and version = 4 is the first bad version.
> 
> call isBadVersion(3) -> false
> call isBadVersion(5) -> true
> call isBadVersion(4) -> true
> 
> Then 4 is the first bad version. 
> ```

思路：为了尽量减少isBadVersion的调用，所以采用折半查找的方式来解决，先判断mid是否错误，如果mid是错误版本，那么第一个错误版本的范围就是[low、mid]，为什么不是mid-1，因为有可能mid刚好是第一个错误版本，**所以，正因为这样也决定的循环条件为low < high，而不能low <= high**；如果mid不是错误版本，那么第一个错误版本的范围只能在[mid+1，high]，最终当low与high相等的时候就找到了第一个错误版本。

> 注：因为n的范围可能会很大，那么`mid = (low+high)/2`可能会越界，所以使用`mid = low + (high - low) / 2`

```java
package com.problem278;

public class Solution extends VersionControl{

    public int firstBadVersion(int n) {
        int low = 1;
        int high = n;
        while (low < high){
            int mid = low + (high - low) / 2;
            if (isBadVersion(mid)){
                high = mid;
            }else {
                low = mid + 1;
            }
        }
        return low;
    }
}
```

### 1.4.5 选择数组的最小值

[153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

> Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.
>
> (i.e.,  `[0,1,2,4,5,6,7]` might become  `[4,5,6,7,0,1,2]`).
>
> Find the minimum element.
>
> You may assume no duplicate exists in the array.
> **Example 1:**
> ```
> Input: [3,4,5,1,2] 
> Output: 1
> ```
> **Example 2:**
>
> ```
> Input: [4,5,6,7,0,1,2]
> Output: 0
> ```

思路：通过比较mid与high位置元素的大小来判断最小值在哪个区间内：

如果nums[mid] > nums[high]，那么说明最小值肯定在[mid+1，high]这个区间内

如果nums[mid] == nums[high]，此时最小值无法判断在哪个区间，那么只能缩小high的值，以便下一次进行判断

如果nums[mid] < nums[high]，此时最小值只能存在于区间[low，mid]，为什么不是mid-1？

例如：[1，0，1]

```java
package com.problem153;

class Solution {
    public int findMin(int[] nums) {
        int low = 0;
        int high = nums.length - 1;
        while (low < high){
            int mid = (low + high) / 2;
            if (nums[mid] > nums[high]){
                low = mid + 1;
            }else if (nums[mid] == nums[high]){
                high--;
            }else {
                high = mid;
            }
        }
        return nums[low];
    }
}
```

### 1.4.6 在数组中查找元素的第一个和最后一个位置

[34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

> Given an array of integers `nums` sorted in ascending order, find the starting and ending position of a given `target` value.
>
> Your algorithm's runtime complexity must be in the order of *O*(log *n*).
>
> If the target is not found in the array, return `[-1, -1]`.
>
> **Example 1:**
>
> ```
> Input: nums = [5,7,7,8,8,10], target = 8
> Output: [3,4]
> ```
>
> **Example 2:**
>
> ```
> Input: nums = [5,7,7,8,8,10], target = 6
> Output: [-1,-1]
> ```

思路：采用折半查找的思路，找比目标值小的第一个元素和找比目标值大的第一个元素

```java
package com.problem34;

class Solution {

    public int[] searchRange(int[] nums, int target) {
        //折半查找第一个比target大的数字
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
        //折半查找第一个比target小的数字
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

## *1.5 分治

分治算法的基本思想是将一个规模为N的问题分解为K个规模较小的子问题，这些子问题相互独立且与原问题性质相同。求出子问题的解，就可得到原问题的解。即一种分目标完成程序算法，简单问题可用二分法完成。

当我们求解某些问题时，由于这些问题要处理的数据相当多，或求解过程相当复杂，使得直接求解法在时间上相当长，或者根本无法直接求出。对于这类问题，我们往往先把它分解成几个子问题，找到求出这几个子问题的解法后，再找到合适的方法，把它们组合成求整个问题的解法。如果这些子问题还较大，难以解决，可以再把它们分成几个更小的子问题，以此类推，直至可以直接求出解为止。这就是**分治策略**的基本思想。

分治法解题的一般步骤：

（1）分解，将要解决的问题划分成若干规模较小的同类问题；

（2）求解，当子问题划分得足够小时，用较简单的方法解决；

（3）合并，按原问题的要求，将子问题的解逐层合并构成原问题的解。

### 1.5.1 为运算表达式设计优先级

[241. Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)

> Given a string of numbers and operators, return all possible results from computing all the different possible ways to group numbers and operators. The valid operators are `+`, `-` and `*`.
>
> **Example 1:**
>
> ```
> Input: "2-1-1"
> Output: [0, 2]
> Explanation: 
> ((2-1)-1) = 0 
> (2-(1-1)) = 2
> ```
>
> **Example 2:**
>
> ```
> Input: "2*3-4*5"
> Output: [-34, -14, -10, -10, 10]
> Explanation: 
> (2*(3-(4*5))) = -34 
> ((2*3)-(4*5)) = -14 
> ((2*(3-4))*5) = -10 
> (2*((3-4)*5)) = -10 
> (((2*3)-4)*5) = 10
> ```

遇到运算符就将字符分割成两部分，直到划分的结果中不包含运算符，也就是找到了运算数，把该运算数返回，然后根据运算符计算表达式的结果，最后将结果返回到上一层递归。

```java
package com.problem241;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<Integer> diffWaysToCompute(String input) {
        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < input.length(); i++) {
            char c = input.charAt(i);
            if (c == '+' || c == '-' || c == '*'){
                List<Integer> left = diffWaysToCompute(input.substring(0, i));
                List<Integer> right = diffWaysToCompute(input.substring(i + 1));
                for (int num1 : left){
                    for (int num2 : right){
                        switch (c){
                            case '+':
                                result.add(num1 + num2);
                                break;
                            case '-':
                                result.add(num1 - num2);
                                break;
                            case '*':
                                result.add(num1 * num2);
                                break;
                        }
                    }
                }
            }
        }
        if (result.size() == 0){
            result.add(Integer.valueOf(input));
        }
        return result;
    }
}
```

### 1.5.2 不同的二叉搜索树

[95. Unique Binary Search Trees II](https://leetcode.com/problems/unique-binary-search-trees-ii/)

> Given an integer *n*, generate all structurally unique **BST's** (binary search trees) that store values 1 ... *n*.
>
> **Example:**
>
> ```
> Input: 3
> Output:
> [
>   [1,null,3,2],
>   [3,2,null,1],
>   [3,1,null,null,2],
>   [2,1,3],
>   [1,null,2,null,3]
> ]
> Explanation:
> The above output corresponds to the 5 unique BST's shown below:
> 
>    1         3     3      2      1
>     \       /     /      / \      \
>      3     2     1      1   3      2
>     /     /       \                 \
>    2     1         2                 3
> ```

```java
package com.problem95;

import com.TreeNode;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<TreeNode> generateTrees(int n) {
        if (n < 1){
            return new ArrayList<>();
        }
        return solve(1, n);
    }

    public List<TreeNode> solve(int start, int end){
        List<TreeNode> result = new ArrayList<>();
        for (int i = start; i <= end; i++) {
            List<TreeNode> leftTree = solve(start, i - 1);
            List<TreeNode> rightTree = solve(i + 1, end);
            for (TreeNode left : leftTree){
                for (TreeNode right : rightTree){
                    TreeNode root = new TreeNode(i);
                    root.left = left;
                    root.right = right;
                    result.add(root);
                }
            }
        }
        if (start > end){
            result.add(null);
            return result;
        }
        return result;
    }
}
```

## 1.6 搜索

深度优先搜索和广度优先搜索广泛运用于树和图中，但是它们的应用远远不止如此。

### 1.6.1 BFS

每一层遍历的节点都与根节点距离相同。设 di 表示第 i 个节点与根节点的距离，推导出一个结论：

对于先遍历的节点 i 与后遍历的节点 j，有 di <= dj。利用这个结论，可以求解最短路径等

**最优解** 问题：第一次遍历到目的节点，其所经过的路径为最短路径。应该注意的是，使用 BFS 只能求解无权图的最短路径，无权图是指从一个节点到另一个节点的代价都记为 1。

#### 1.6.1.1 岛屿的个数

[200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

> Given a 2d grid map of `'1'`s (land) and `'0'`s (water), count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.
>
> **Example 1:**
>
> ```
> Input:
> 11110
> 11010
> 11000
> 00000
> 
> Output: 1
> ```
>
> **Example 2:**
>
> ```
> Input:
> 11000
> 11000
> 00100
> 00011
> 
> Output: 3
> ```

采用BFS的思路，先找到一个不为0的点，那么就找到了一个岛屿，然后遍历其四周，把四周都置为0，直到全部为0，则继续寻找下一个不为0的点。

```java
package com.problem200;

import javafx.util.Pair;

import java.util.LinkedList;

class Solution {
    public int numIslands(char[][] grid) {
        int row = grid.length;
        if (row == 0){
            return 0;
        }
        int col = grid[0].length;
        int result = 0;
        LinkedList<Pair<Integer,Integer>> queue = new LinkedList<>();
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (grid[i][j] == '1'){
                    queue.add(new Pair<>(i, j));
                    result++;
                    while (!queue.isEmpty()){
                        Pair<Integer,Integer> cur = queue.poll();
                        int x = cur.getKey();
                        int y = cur.getValue();
                        if (grid[x][y] == '0'){
                            continue;
                        }
                        grid[x][y] = '0';
                        Pair<Integer,Integer> up = new Pair<>(x - 1, y);
                        Pair<Integer,Integer> down = new Pair<>(x + 1, y);
                        Pair<Integer,Integer> left = new Pair<>(x, y - 1);
                        Pair<Integer,Integer> right = new Pair<>(x, y + 1);
                        if (inGrid(up,grid) && grid[up.getKey()][up.getValue()] == '1'){
                            queue.add(up);
                        }
                        if (inGrid(down,grid) && grid[down.getKey()][down.getValue()] == '1'){
                            queue.add(down);
                        }
                        if (inGrid(left,grid) && grid[left.getKey()][left.getValue()] == '1'){
                            queue.add(left);
                        }
                        if (inGrid(right,grid) && grid[right.getKey()][right.getValue()] == '1'){
                            queue.add(right);
                        }
                    }
                }
            }
        }
        return result;
    }

    private boolean inGrid(Pair<Integer, Integer> pair, char[][] grid) {
        return pair.getKey() >= 0 && pair.getKey() < grid.length && pair.getValue() >= 0 && pair.getValue() < grid[0].length;
    }

}
```

#### *1.6.1.2 完全平方数

[279. Perfect Squares](https://leetcode.com/problems/perfect-squares/)

> Given a positive integer *n*, find the least number of perfect square numbers (for example, `1, 4, 9, 16, ...`) which sum to *n*.
>
> **Example 1:**
>
> ```
> Input: n = 12
> Output: 3 
> Explanation: 12 = 4 + 4 + 4.
> ```
>
> **Example 2:**
>
> ```
> Input: n = 13
> Output: 2
> Explanation: 13 = 4 + 9.
> ```

可以将每个整数看成图中的一个节点，如果两个整数之差为一个平方数，那么这两个整数所在的节点就有一条边。

要求解最小的平方数数量，就是求解从节点 n 到节点 0 的最短路径。

```java
package com.problem279;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.Queue;

/**
 * @Author: 98050
 * @Time: 2019-06-23 19:37
 * @Feature:
 * 采用BFS的思路来解决，可以将每个整数看成图中的一个节点，如果两个整数之差为一个平方数，那么这两个整数所在的节点就有一条边。
 * 要求解最小的平方数数量，就是求解从节点 n 到节点 0 的最短路径。
 */
public class Solution2 {

    public int numSquares(int n) {
        if (n <= 1){
            return n;
        }
        List<Integer> squareList = generateSquareList(n);
        LinkedList<Integer> queue = new LinkedList<>();
        boolean[] grid = new boolean[n + 1];
        queue.add(n);
        int count = 0;
        while (!queue.isEmpty()){
            int size = queue.size();
            count++;
            while (size-- > 0){
                int now = queue.poll();
                for (int num : squareList){
                    int temp = now - num;
                    if (temp < 0){
                        break;
                    }
                    if (temp == 0){
                        return count;
                    }
                    if (grid[temp]){
                        continue;
                    }
                    grid[temp] = true;
                    queue.add(temp);
                }
            }
        }
        return -1;
    }


    private List<Integer> generateSquareList(int n) {
        List<Integer> list = new ArrayList<>();
        for (int i = 1; i < n; i++) {
            if (i * i <= n){
                list.add(i * i);
            }else {
                break;
            }
        }
        return list;
    }
}
```

#### 



### 1.6.2 DFS

从一个节点出发，使用 DFS 对一个图进行遍历时，能够遍历到的节点都是从初始节点可达的，DFS 常用来求解这种 **可达性** 问题。

#### 1.6.2.1 岛屿的个数

[200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

> Given a 2d grid map of `'1'`s (land) and `'0'`s (water), count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.
>
> **Example 1:**
>
> ```
> Input:
> 11110
> 11010
> 11000
> 00000
> 
> Output: 1
> ```
>
> **Example 2:**
>
> ```
> Input:
> 11000
> 11000
> 00100
> 00011
> 
> Output: 3
> ```

采用DFS的思路，找到第一个不为0的点，开始进行深度优先遍历，也就是找到一个连通分量，遍历过程中把所有不为0的点，都变为0.

```java
package com.problem200;

/**
 * @Author: 98050
 * @Time: 2019-06-22 20:32
 * @Feature: 采用DFS来解决
 */
public class Solution2 {

    public int numIslands(char[][] grid) {
        int row = grid.length;
        if (row == 0){
            return 0;
        }
        int result = 0;
        int col = grid[0].length;
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (grid[i][j] == '1'){
                    result++;
                    dfs(grid,i,j);
                }
            }
        }
        return result;
    }

    private void dfs(char[][] grid, int i, int j) {
        if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0'){
            return;
        }
        grid[i][j] = '0';
        dfs(grid, i - 1, j);
        dfs(grid, i + 1, j);
        dfs(grid, i, j - 1);
        dfs(grid, i, j + 1);
    }

}
```

#### 1.6.2.2 岛屿的最大面积

[695. Max Area of Island](https://leetcode.com/problems/max-area-of-island/)

> Given a non-empty 2D array `grid` of 0's and 1's, an **island** is a group of `1`'s (representing land) connected 4-directionally (horizontal or vertical.) You may assume all four edges of the grid are surrounded by water.
>
> Find the maximum area of an island in the given 2D array. (If there is no island, the maximum area is 0.)
>
> **Example 1:**
>
> ```
> [[0,0,1,0,0,0,0,1,0,0,0,0,0],
>  [0,0,0,0,0,0,0,1,1,1,0,0,0],
>  [0,1,1,0,1,0,0,0,0,0,0,0,0],
>  [0,1,0,0,1,1,0,0,1,0,1,0,0],
>  [0,1,0,0,1,1,0,0,1,1,1,0,0],
>  [0,0,0,0,0,0,0,0,0,0,1,0,0],
>  [0,0,0,0,0,0,0,1,1,1,0,0,0],
>  [0,0,0,0,0,0,0,1,1,0,0,0,0]]
> ```
>
> Given the above grid, return 6. Note the answer is not 11, because the island must be connected 4-directionally.
>
> **Example 2:**
>
> ```
> [[0,0,0,0,0,0,0,0]]
> ```
>
> Given the above grid, return 0.
>
> **Note:** The length of each dimension in the given `grid` does not exceed 50.

```java
package com.problem695;

class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int row = grid.length;
        if (row == 0){
            return 0;
        }
        int col = grid[0].length;
        int result = 0;
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (grid[i][j] == 1){
                    result = Math.max(result, dfs(grid,i,j));
                }
            }
        }
        return result;
    }

    private int dfs(int[][] grid, int i, int j) {
        int result = 0;
        if (i >= 0 && i < grid.length && j >= 0 && j < grid[0].length && grid[i][j] != 0){
            result++;
            grid[i][j] = 0;
            return result + dfs(grid, i - 1, j) + dfs(grid, i + 1, j) + dfs(grid, i, j - 1) + dfs(grid, i, j + 1);
        }else {
            return result;
        }
    }

}
```

#### 1.6.3.3 朋友圈

[547. Friend Circles](https://leetcode.com/problems/friend-circles/)

> There are **N** students in a class. Some of them are friends, while some are not. Their friendship is transitive in nature. For example, if A is a **direct** friend of B, and B is a **direct** friend of C, then A is an **indirect** friend of C. And we defined a friend circle is a group of students who are direct or indirect friends.
>
> Given a **N\*N** matrix **M** representing the friend relationship between students in the class. If M[i][j] = 1, then the ith and jth students are **direct** friends with each other, otherwise not. And you have to output the total number of friend circles among all the students.
>
> **Example 1:**
>
> ```
> Input: 
> [[1,1,0],
>  [1,1,0],
>  [0,0,1]]
> Output: 2
> Explanation:The 0th and 1st students are direct friends, so they are in a friend circle. 
> The 2nd student himself is in a friend circle. So return 2.
> ```
>
> 
>
> **Example 2:**
>
> ```
> Input: 
> [[1,1,0],
>  [1,1,1],
>  [0,1,1]]
> Output: 1
> Explanation:The 0th and 1st students are direct friends, the 1st and 2nd students are direct friends, 
> so the 0th and 2nd students are indirect friends. All of them are in the same friend circle, so return 1.
> ```
>
> 
>
> **Note:**
>
> 1. N is in range [1,200].
> 2. `M[i][i]` = 1 for all students.
> 3. If `M[i][j]` = 1, then `M[j][i]` = 1.

思路：根据好友的传递性，那么通过一个同学，那么就可以把他的朋友圈都标记为0，那么朋友圈的个数就是dfs的次数。先从第一个同学开始，即`M[0][0]`，把`M[0][0]`置为0，然后开始遍历与他互为朋友的同学，例如找到一个朋友`M[0][k]`，那么就把`M[0][k]`，`M[k][0]`都置为0，接着继续从k同学开始dfs，最终就把朋友圈都标记为0。

```java
package com.problem547;

class Solution {
    public int findCircleNum(int[][] M) {
        int row = M.length;
        if (row == 0){
            return 0;
        }
        int result = 0;
        for (int i = 0; i < row; i++) {
            if (M[i][i] == 1){
                dfs(M, i, i);
                result++;
            }
        }
        return result;
    }

    private void dfs(int[][] m, int i, int j) {
        if (i < 0 || i >= m.length || j < 0 || j >= m[0].length || m[i][j] == 0){
            return;
        }
        m[i][j] = 0;
        for (int k = 0; k < m[0].length; k++) {
            if (m[i][k] == 1){
                m[i][k] = 0;
                m[k][i] = 0;
                dfs(m, k, k);
            }
        }
    }
}
```

#### *1.6.3.4 被围绕的区域

[130. Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

> Given a 2D board containing `'X'` and `'O'` (**the letter O**), capture all regions surrounded by `'X'`.
>
> A region is captured by flipping all `'O'`s into `'X'`s in that surrounded region.
>
> **Example:**
>
> ```
> X X X X
> X O O X
> X X O X
> X O X X
> ```
>
> After running your function, the board should be:
>
> ```
> X X X X
> X X X X
> X X X X
> X O X X
> ```
>
> **Explanation:**
>
> Surrounded regions shouldn’t be on the border, which means that any `'O'` on the border of the board are not flipped to `'X'`. Any `'O'` that is not on the border and it is not connected to an `'O'` on the border will be flipped to `'X'`. Two cells are connected if they are adjacent cells connected horizontally or vertically.

`思路：从边界上的“O”出发进行深度搜索，将图中所有与边界上的“O”连通的“O”（包括边界上的“O”）都用“*”替代，那么图中所有“*”组成的区域其实是无法被“X”包围的，那么图中剩余的“O”就完全被“X”包围了。所以在深度搜索完成后，将图中所有“*”的点用“O”替换，所有“O”的点用“X”替换。`

```java
package com.problem130;

class Solution {
    public void solve(char[][] board) {
        int row = board.length;
        if (row == 0){
            return;
        }
        int col = board[0].length;

        for (int i = 0; i < row; i++) {
            dfs(board,i,0);
            dfs(board,i,col - 1);
        }
        for (int i = 0; i < col; i++) {
            dfs(board,0,i);
            dfs(board,row - 1,i);
        }

        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (board[i][j] == 'O'){
                    board[i][j] = 'X';
                }
                if (board[i][j] == '*'){
                    board[i][j] = 'O';
                }
            }
        }
    }

    private void dfs(char[][] board, int i, int j) {
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length || board[i][j] != 'O'){
            return;
        }
        board[i][j] = '*';
        dfs(board, i - 1, j);
        dfs(board, i + 1, j);
        dfs(board, i, j - 1);
        dfs(board, i, j + 1);
    }


}
```

#### *1.6.3.5 太平洋大西洋水流问题

[417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)

> Given an `m x n` matrix of non-negative integers representing the height of each unit cell in a continent, the "Pacific ocean" touches the left and top edges of the matrix and the "Atlantic ocean" touches the right and bottom edges.
>
> Water can only flow in four directions (up, down, left, or right) from a cell to another one with height equal or lower.
>
> Find the list of grid coordinates where water can flow to both the Pacific and Atlantic ocean.
>
> **Note:**
>
> 1. The order of returned grid coordinates does not matter.
> 2. Both *m* and *n* are less than 150.
>
>  
>
> **Example:**
>
> ```
> Given the following 5x5 matrix:
> 
>   Pacific ~   ~   ~   ~   ~ 
>        ~  1   2   2   3  (5) *
>        ~  3   2   3  (4) (4) *
>        ~  2   4  (5)  3   1  *
>        ~ (6) (7)  1   4   5  *
>        ~ (5)  1   1   2   4  *
>           *   *   *   *   * Atlantic
> 
> Return:
> 
> [[0, 4], [1, 3], [1, 4], [2, 2], [3, 0], [3, 1], [4, 0]] (positions with parentheses in above matrix).
> ```

思路：与`被围绕的区域`类似，从边界出发开始dfs+记忆搜索。用两个额外的boolean矩阵来标识可以到达太平洋和大西洋的位置，然后找两个矩阵中重合的点即为最终结果。

```java
package com.problem417;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> pacificAtlantic(int[][] matrix) {
        List<List<Integer>> result = new ArrayList<>();
        int row = matrix.length;
        if (row == 0){
            return result;
        }
        int col = matrix[0].length;

        boolean[][] flagA = new boolean[row][col];
        boolean[][] flagB = new boolean[row][col];

        for (int i = 0; i < row; i++) {
            dfs(matrix,i,0,flagA);
            dfs(matrix,i,col - 1,flagB);
        }
        for (int i = 0; i < col; i++) {
            dfs(matrix,0,i,flagA);
            dfs(matrix,row - 1,i,flagB);
        }

        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (flagA[i][j] && flagB[i][j]){
                    result.add(Arrays.asList(i,j));
                }
            }
        }

        return result;
    }

    private void dfs(int[][] matrix, int i, int j, boolean[][] flag) {
        if (flag[i][j]){
            return;
        }
        flag[i][j] = true;
        if (i - 1 >= 0 && matrix[i][j] <= matrix[i - 1][j]){
            dfs(matrix, i - 1, j,flag);
        }
        if (i + 1 < matrix.length && matrix[i][j] <= matrix[i + 1][j]){
            dfs(matrix, i + 1, j,flag);
        }
        if (j - 1 >= 0 && matrix[i][j] <= matrix[i][j - 1]){
            dfs(matrix, i, j - 1,flag);
        }
        if (j + 1 < matrix[0].length && matrix[i][j] <= matrix[i][j + 1]){
            dfs(matrix, i, j + 1,flag);
        }
    }


}
```

### *1.6.3 回溯

回溯法属于DFS，普通DFS主要解决可达性问题，而回溯主要解决排列组合问题。

因为回溯不是立即返回，而要继续求解，因此在程序实现时，需要注意对元素的标记问题：

- 在访问一个新元素进入新的递归调用时，需要将新元素标记为已经访问，这样才能在继续递归调用时不用重复访问该元素；
- 但是在递归返回时，需要将元素标记为未访问，因为只需要保证在一个递归链中不同时访问一个元素，可以访问已经访问过但是不在当前递归链中的元素。

#### 1.6.3.1 字符串的全排列

> 剑指Offer

> 输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。

注意：当添加一个元素的时候要判断这个字符是否等于前一个字符，如果等于且前一个字符还未访问，那么就跳过这个字符。

```java
package com.example.problem27;

import java.util.ArrayList;

/**
 * @Author: 98050
 * @Time: 2019-06-30 14:41
 * @Feature:
 */
public class Solution2 {

    public ArrayList<String> Permutation(String str) {
        ArrayList<String> result = new ArrayList<>();
        if (str.length() == 0){
            return result;
        }
        boolean[] tag = new boolean[str.length()];
        solve(str,new StringBuilder(),result,tag);
        return result;
    }

    private void solve(String str, StringBuilder sb, ArrayList<String> result, boolean[] tag) {
        if (sb.length() == str.length()){
            result.add(sb.toString());
            return;
        }
        for (int i = 0; i < tag.length; i++) {
            if ((i != 0 && str.charAt(i) == str.charAt(i - 1) && !tag[i - 1]) || tag[i]){
                continue;
            }
            tag[i] = true;
            sb.append(str.charAt(i));
            solve(str, sb, result, tag);
            sb.deleteCharAt(sb.length() - 1);
            tag[i] = false;
        }
    }
}
```

#### 1.6.3.2 数字键盘组合

[17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

> Given a string containing digits from `2-9` inclusive, return all possible letter combinations that the number could represent.
>
> A mapping of digit to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.
>
> ![img](http://upload.wikimedia.org/wikipedia/commons/thumb/7/73/Telephone-keypad2.svg/200px-Telephone-keypad2.svg.png)
>
> **Example:**
>
> ```
> Input: "23"
> Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
> ```

```java
package com.problem17;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-06-30 14:23
 * @Feature:
 */
public class Solution3 {
    public List<String> letterCombinations(String digits) {
        List<String> num = Arrays.asList("abc","def","ghi","jkl","mno","pqrs","tuv","wxyz");
        List<String> result = new ArrayList<>();
        if (digits.length() == 0){
            return result;
        }
        solve(result, new StringBuilder(),digits,num);
        return result;
    }

    private void solve(List<String> result, StringBuilder sb, String digits, List<String> num) {
        if (sb.length() == digits.length()){
            result.add(sb.toString());
            return;
        }
        int index = digits.charAt(sb.length()) - '0' - 2;
        String string = num.get(index);
        for (char c : string.toCharArray()){
            sb.append(c);
            solve(result, sb, digits, num);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

#### *1.6.3.3 复原IP地址

[93. Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/)

> Given a string containing only digits, restore it by returning all possible valid IP address combinations.
>
> **Example:**
>
> ```
> Input: "25525511135"
> Output: ["255.255.11.135", "255.255.111.35"]
> ```

思路：假设已经放置了一或两个点使得无法摆放其他点来生成有效IP地址。这时应该做什么？ 回溯。也就是说，回到之前，改变上一个摆放点的位置。并试着继续。如果依然不行，则继续 回溯。 

注意：

- 每一次最多截取字符的长度为3，所以`i <= 2`

- 考虑字符串“010010”，不能出现010这种情况，所以在回溯的时候要进行判断

```java
package com.problem93;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<String> restoreIpAddresses(String s) {
        List<String> result = new ArrayList<>();
        solve(0,new StringBuilder(),result,s);
        return result;
    }

    private void solve(int k, StringBuilder sb, List<String> result, String s) {
        if (k == 4 || s.length() == 0){
            if (k == 4 && s.length() == 0){
                result.add(sb.toString());
            }
            return;
        }
        for (int i = 0; i < s.length() && i <= 2; i++) {
            if (i != 0 && s.charAt(0) == '0'){
                break;
            }
            String temp = s.substring(0, i + 1);
            if (Integer.valueOf(temp) <= 255){
                if (sb.length() != 0){
                    temp = "." + temp;
                }
                sb.append(temp);
                solve(k + 1, sb, result, s.substring(i + 1));
                sb.delete(sb.length() - temp.length(), sb.length());
            }
        }
    }
}
```

#### 1.6.3.4 单词搜索

[79. Word Search](https://leetcode.com/problems/word-search/)

> Given a 2D board and a word, find if the word exists in the grid.
>
> The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.
>
> **Example:**
>
> ```
> board =
> [
>   ['A','B','C','E'],
>   ['S','F','C','S'],
>   ['A','D','E','E']
> ]
> 
> Given word = "ABCCED", return true.
> Given word = "SEE", return true.
> Given word = "ABCB", return false.
> ```

思路：常见的dfs，但是因为每个字母只能使用一次，所以需要记忆搜索，同时再加回溯就可以解决问题

```java
package com.problem79;

class Solution {
    public boolean exist(char[][] board, String word) {
        int row = board.length;
        if (row == 0){
            return false;
        }
        int col = board[0].length;
        boolean[][] tag = new boolean[row][col];
        for (int i = 0; i < row; i++) {
            for (int j = 0; j < col; j++) {
                if (dfs(0,i,j,tag,board,word)){
                    return true;
                }
            }
        }
        return false;
    }

    private boolean dfs(int k, int i, int j, boolean[][] tag, char[][] board, String word) {
        if (k == word.length()){
            return true;
        }
        if (i < 0 || i >= board.length || j < 0 || j >= board[0].length || tag[i][j] || board[i][j] != word.charAt(k)){
            return false;
        }
        tag[i][j] = true;
        boolean result = dfs(k + 1, i - 1, j, tag, board, word) || dfs(k + 1, i + 1, j, tag, board, word) || dfs(k + 1, i, j - 1, tag, board, word) || dfs(k + 1, i, j + 1, tag, board, word);
        tag[i][j] = false;
        return result;
    }

}
```

#### 1.6.3.5 二叉树的所有路径

[257. Binary Tree Paths](https://leetcode.com/problems/binary-tree-paths/)

> Given a binary tree, return all root-to-leaf paths.
>
> **Note:** A leaf is a node with no children.
>
> **Example:**
>
> ```
> Input:
> 
>    1
>  /   \
> 2     3
>  \
>   5
> 
> Output: ["1->2->5", "1->3"]
> 
> Explanation: All root-to-leaf paths are: 1->2->5, 1->3
> ```

回溯法：

```java
package com.problem257;

import com.TreeNode;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-07-02 10:41
 * @Feature:
 */
public class Solution2 {

    public List<String> binaryTreePaths(TreeNode root) {
        List<String> result = new ArrayList<>();
        if (root == null){
            return result;
        }
        List<Integer> list = new ArrayList<>();
        dfs(root,list,result);
        return result;
    }

    private void dfs(TreeNode root, List<Integer> list, List<String> result) {
        if (root != null){
            list.add(root.val);
            if (root.left == null && root.right == null){
                result.add(build(list));
            }else {
                dfs(root.left, list, result);
                dfs(root.right, list, result);
            }
            list.remove(list.size() - 1);
        }
    }

    private String build(List<Integer> list) {
        StringBuilder sb = new StringBuilder();
        for (int i : list){
            sb.append(i).append("->");
        }
        sb.delete(sb.length() - 2, sb.length());
        return sb.toString();
    }
}
```

代码改进，使用String，直接省去最后删除节点的步骤

```java
package com.problem257;

import com.TreeNode;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<String> binaryTreePaths(TreeNode root) {
        List<String> result = new ArrayList<>();
        if (root == null){
            return result;
        }
        dfs(root,"",result);
        return result;
    }


    private void dfs(TreeNode root, String string, List<String> result) {
        if (root != null) {
            string += root.val;
            if (root.left == null && root.right == null) {
                result.add(string);
                return;
            }else {
                string += "->";
                dfs(root.left, string, result);
                dfs(root.right, string, result);
            }
        }

    }
}
```

层次遍历：

```java
package com.problem257;

import com.TreeNode;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-07-02 10:52
 * @Feature:
 */
public class Solution3 {

    public List<String> binaryTreePaths(TreeNode root) {
        LinkedList<String> paths = new LinkedList();
        if (root == null) {
            return paths;
        }

        LinkedList<TreeNode> node_stack = new LinkedList();
        LinkedList<String> path_stack = new LinkedList();
        node_stack.add(root);
        path_stack.add(Integer.toString(root.val));
        TreeNode node;
        String path;
        while (!node_stack.isEmpty()) {
            node = node_stack.pollFirst();
            path = path_stack.pollFirst();
            if ((node.left == null) && (node.right == null)) {
                paths.add(path);
            }
            if (node.left != null) {
                node_stack.add(node.left);
                path_stack.add(path + "->" + node.left.val);
            }
            if (node.right != null) {
                node_stack.add(node.right);
                path_stack.add(path + "->" + node.right.val);
            }
        }
        return paths;
    }
}
```

#### *1.6.3.6 全排列

[46. Permutations](https://leetcode.com/problems/permutations/)

> Given a collection of **distinct** integers, return all possible permutations.
>
> **Example:**
>
> ```
> Input: [1,2,3]
> Output:
> [
>   [1,2,3],
>   [1,3,2],
>   [2,1,3],
>   [2,3,1],
>   [3,1,2],
>   [3,2,1]
> ]
> ```

需要注意的地方是，当满足输出条件的时候，一定要重新构list将其放入结果集中，否则到最后回溯完成就都为空了。

```java
package com.problem46;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author: 98050
 * @Time: 2019-07-02 11:38
 * @Feature:
 */
public class Solution2 {

    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        if (nums.length == 0){
            return result;
        }
        boolean[] tag = new boolean[nums.length];
        solve(new ArrayList<>(),nums,result,tag);
        return result;
    }

    private void solve(ArrayList<Integer> integers, int[] nums, List<List<Integer>> result, boolean[] tag) {
        if (integers.size() == nums.length){
            result.add(new ArrayList<>(integers));
            return;
        }
        for (int i = 0; i < tag.length; i++) {
            if (tag[i]){
                continue;
            }
            tag[i] = true;
            integers.add(nums[i]);
            solve(integers, nums, result, tag);
            tag[i] = false;
            integers.remove(integers.size() - 1);
        }
    }

}
```

#### 1.6.3.7 含有相同元素的全排列

[47. Permutations II](https://leetcode.com/problems/permutations-ii/)

> Given a collection of numbers that might contain duplicates, return all possible unique permutations.
>
> **Example:**
>
> ```
> Input: [1,1,2]
> Output:
> [
>   [1,1,2],
>   [1,2,1],
>   [2,1,1]
> ]
> ```

因为重复的数字交换位置没有意义，所以可以先将nums先排序，然后再加一个判断条件，即：如果当前元素与前一个元素相同，并且前一个元素没有被访问


```java
package com.problem47;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> permuteUnique(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);
        boolean[] visited = new boolean[nums.length];
        if (nums.length == 0){
            return result;
        }
        solve(new ArrayList<>(),nums,visited,result);
        return result;
    }

    private void solve(ArrayList<Integer> integers, int[] nums, boolean[] visited, List<List<Integer>> result) {
        if (integers.size() == nums.length){
            result.add(new ArrayList<>(integers));
            return;
        }
        for (int i = 0; i < visited.length; i++) {
            if ((i != 0 && nums[i] == nums[i - 1] && !visited[i - 1]) || visited[i]){
                continue;
            }
            visited[i] = true;
            integers.add(nums[i]);
            solve(integers, nums, visited, result);
            integers.remove(integers.size() - 1);
            visited[i] = false;
        }
    }
}
```

#### *1.6.3.8 组合

[77. Combinations](https://leetcode.com/problems/combinations/)

> Given two integers *n* and *k*, return all possible combinations of *k* numbers out of 1 ... *n*.
>
> **Example:**
>
> ```
> Input: n = 4, k = 2
> Output:
> [
>   [2,4],
>   [3,4],
>   [2,3],
>   [1,2],
>   [1,3],
>   [1,4],
> ]
> ```

因为求的是组合，所以在回溯的时候要进行剪枝，限定回溯的范围，所以每次递归要限定开始位置，返回的时候在原来的基础上继续递归，而不是从头开始


```java
package com.problem77;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<List<Integer>> combine(int n, int k) {
        List<List<Integer>> result = new ArrayList<>();
        solve(1,n,k, new ArrayList<>(),result);
        return result;
    }

    private void solve(int start, int end, int k, ArrayList<Integer> integers, List<List<Integer>> result) {
        if (integers.size() == k){
            result.add(new ArrayList<>(integers));
            return;
        }
        for (int j = start; j <= end; j++) {
            integers.add(j);
            solve(j + 1, end, k, integers, result);
            integers.remove(integers.size() - 1);
        }
    }
}
```

#### 1.6.3.9 组合求和

[39. Combination Sum](https://leetcode.com/problems/combination-sum/)

> Given a **set** of candidate numbers (`candidates`) **(without duplicates)** and a target number (`target`), find all unique combinations in `candidates` where the candidate numbers sums to `target`.
>
> The **same** repeated number may be chosen from `candidates` unlimited number of times.
>
> **Note:**
>
> - All numbers (including `target`) will be positive integers.
> - The solution set must not contain duplicate combinations.
>
> **Example 1:**
>
> ```
> Input: candidates = [2,3,6,7], target = 7,
> A solution set is:
> [
>   [7],
>   [2,2,3]
> ]
> ```
>
> **Example 2:**
>
> ```
> Input: candidates = [2,3,5], target = 8,
> A solution set is:
> [
>   [2,2,2,2],
>   [2,3,3],
>   [3,5]
> ]
> ```

注意：可以重复使用元素，所以使用贪心策略，一个数先用到底，不满足条件的话再回溯。


```java
package com.problem39;

import java.util.ArrayList;
import java.util.List;

class Solution {
    

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> result = new ArrayList<>();
        solve(new ArrayList<>(),candidates,0,target,result);
        return result;
    }

    private void solve(ArrayList<Integer> integers, int[] candidates, int start,int target, List<List<Integer>> result) {
        if (0 == target){
            result.add(new ArrayList<>(integers));
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] <= target) {
                integers.add(candidates[i]);
                solve(integers, candidates, i, target - candidates[i], result);
                integers.remove(integers.size() - 1);
            }
        }
    }
}
```

#### 1.6.3.10 含有相同元素的组合求和

[40. Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

> Given a collection of candidate numbers (`candidates`) and a target number (`target`), find all unique combinations in `candidates` where the candidate numbers sums to `target`.
>
> Each number in `candidates` may only be used **once** in the combination.
>
> **Note:**
>
> - All numbers (including `target`) will be positive integers.
> - The solution set must not contain duplicate combinations.
>
> **Example 1:**
>
> ```
> Input: candidates = [10,1,2,7,6,1,5], target = 8,
> A solution set is:
> [
>   [1, 7],
>   [1, 2, 5],
>   [2, 6],
>   [1, 1, 6]
> ]
> ```
>
> **Example 2:**
>
> ```
> Input: candidates = [2,5,2,1,2], target = 5,
> A solution set is:
> [
>   [1,2,2],
>   [5]
> ]
> ```

```java
package com.problem40;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(candidates);
        boolean[] tag = new boolean[candidates.length];
        solve(new ArrayList<>(),candidates,0,target,result,tag);
        return result;
    }

    private void solve(ArrayList<Integer> integers, int[] candidates, int start, int target, List<List<Integer>> result,boolean[] tag) {
        if (target == 0){
            result.add(new ArrayList<>(integers));
        }
        for (int i = start; i < candidates.length; i++) {
            if (i > 0 && candidates[i] == candidates[i - 1] && !tag[i - 1]){
                continue;
            }
            if (candidates[i] <= target){
                integers.add(candidates[i]);
                tag[i] = true;
                solve(integers, candidates, i + 1, target - candidates[i], result,tag);
                tag[i] = false;
                integers.remove(integers.size() - 1);
            }
        }
    }
}
```

#### 1.6.3.11 1~9数字的组合求和

> Find all possible combinations of ***k*** numbers that add up to a number ***n***, given that only numbers from 1 to 9 can be used and each combination should be a unique set of numbers.
>
> **Note:**
>
> - All numbers will be positive integers.
> - The solution set must not contain duplicate combinations.
>
> **Example 1:**
>
> ```
> Input: k = 3, n = 7
> Output: [[1,2,4]]
> ```
>
> **Example 2:**
>
> ```
> Input: k = 3, n = 9
> Output: [[1,2,6], [1,3,5], [2,3,4]]
> ```

```java
package com.problem216;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<List<Integer>> combinationSum3(int k, int n) {
        List<List<Integer>> result = new ArrayList<>();
        solve(new ArrayList<>(),1,k,n,result);
        return result;
    }

    private void solve(ArrayList<Integer> integers, int start, int k, int target, List<List<Integer>> result) {
        if (integers.size() == k && target == 0){
            result.add(new ArrayList<>(integers));
            return;
        }
        for (int i = start; i <= 9; i++) {
            if (i <= target){
                integers.add(i);
                solve(integers, i + 1, k, target - i, result);
                integers.remove(integers.size() - 1);
            }
        }
    }
}
```

#### 1.6.3.12 子集

[78. Subsets](https://leetcode.com/problems/subsets/)

> Given a set of **distinct** integers, *nums*, return all possible subsets (the power set).
>
> **Note:** The solution set must not contain duplicate subsets.
>
> **Example:**
>
> ```
> Input: nums = [1,2,3]
> Output:
> [
>   [3],
>   [1],
>   [2],
>   [1,2,3],
>   [1,3],
>   [2,3],
>   [1,2],
>   []
> ]
> ```

```java
package com.problem78;

import java.util.ArrayList;
import java.util.List;

class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        solve(new ArrayList<>(),0,nums,result);
        result.add(new ArrayList<>());
        return result;
    }

    private void solve(ArrayList<Integer> integers, int start, int[] nums, List<List<Integer>> result) {
        for (int i = start; i < nums.length; i++) {
            integers.add(nums[i]);
            result.add(new ArrayList<>(integers));
            solve(integers, i + 1, nums, result);
            integers.remove(integers.size() - 1);
        }
    }
}
```

#### 1.6.3.13 含有相同元素的子集

[90. Subsets II](https://leetcode.com/problems/subsets-ii/)

> Given a collection of integers that might contain duplicates, **nums**, return all possible subsets (the power set).
>
> **Note:** The solution set must not contain duplicate subsets.
>
> **Example:**
>
> ```
> Input: [1,2,2]
> Output:
> [
>   [2],
>   [1],
>   [1,2,2],
>   [2,2],
>   [1,2],
>   []
> ]
> ```

```java
package com.problem90;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);
        boolean[] tag = new boolean[nums.length];
        solve(new ArrayList<>(),nums,tag,0,result);
        result.add(new ArrayList<>());
        return result;
    }

    private void solve(ArrayList<Integer> integers, int[] nums, boolean[] tag, int start, List<List<Integer>> result) {
        for (int i = start; i < nums.length; i++) {
            if (i > 0 && nums[i - 1] == nums[i] && !tag[i - 1]){
                continue;
            }
            tag[i] = true;
            integers.add(nums[i]);
            result.add(new ArrayList<>(integers));
            solve(integers, nums, tag, i + 1, result);
            tag[i] = false;
            integers.remove(integers.size() - 1);
        }
    }


}
```

#### 1.6.3.14 分割回文串

[131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

> Given a string *s*, partition *s* such that every substring of the partition is a palindrome.
>
> Return all possible palindrome partitioning of *s*.
>
> **Example:**
>
> ```
> Input: "aab"
> Output:
> [
>   ["aa","b"],
>   ["a","a","b"]
> ]
> ```

## 1.7 动态规划

### *1.7.1 不同的二叉搜索树

[96. Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/)

> Given *n*, how many structurally unique **BST's** (binary search trees) that store values 1 ... *n*?
>
> **Example:**
>
> ```
> Input: 3
> Output: 5
> Explanation:
> Given n = 3, there are a total of 5 unique BST's:
> 
>    1         3     3      2      1
>     \       /     /      / \      \
>      3     2     1      1   3      2
>     /     /       \                 \
>    2     1         2                 3
> ```

![1561196630964](assets/1561196630964.png)

就跟斐波那契数列一样，我们把n = 0 时赋为1，因为空树也算一种二叉搜索树，那么n = 1时的情况可以看做是其左子树个数乘以右子树的个数，左右字数都是空树，所以1乘1还是1。那么n = 2时，由于1和2都可以为跟，分别算出来，再把它们加起来即可。n = 2的情况可由下面式子算出：

dp[2] =  dp[0] * dp[1]　　　(1为根的情况)

　　　　+ dp[1] * dp[0]　　  (2为根的情况)

同理可写出 n = 3 的计算方法：

dp[3] =  dp[0] * dp[2]　　　(1为根的情况)

　　　　+ dp[1] * dp[1]　　  (2为根的情况)

 　　　  + dp[2] * dp[0]　　  (3为根的情况)

```java
package com.problem96;

class Solution {
    
    public  int numTrees(int n) {
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;

        for (int i = 2; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                dp[i] += dp[j] * dp[i - 1 - j];
            }
        }
        return dp[n];
    }
}
```

### 1.7.2 完全平方数

[279. Perfect Squares](https://leetcode.com/problems/perfect-squares/)

> Given a positive integer *n*, find the least number of perfect square numbers (for example, `1, 4, 9, 16, ...`) which sum to *n*.
>
> **Example 1:**
>
> ```
> Input: n = 12
> Output: 3 
> Explanation: 12 = 4 + 4 + 4.
> ```
>
> **Example 2:**
>
> ```
> Input: n = 13
> Output: 2
> Explanation: 13 = 4 + 9.
> ```

思路：将小于n的所有完全平方数找出来，然后转换成背包问题

```java
package com.problem279;

import java.util.ArrayList;
import java.util.List;

class Solution {

    public int numSquares(int n) {
        if (n <= 1){
            return n;
        }
        List<Integer> squareList = generateSquareList(n);
        int[] result = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            int min = Integer.MAX_VALUE;
            for (int num : squareList){
                if (i < num){
                    break;
                }
                min = Math.min(min, result[i - num] + 1);
            }
            result[i] = min;
        }
        return result[n];
    }

    private List<Integer> generateSquareList(int n) {
        List<Integer> list = new ArrayList<>();
        for (int i = 1; i < n; i++) {
            if (i * i <= n){
                list.add(i * i);
            }else {
                break;
            }
        }
        return list;
    }
}
```

## 1.8 数学

# 二、数据结构

## 2.1 链表

## 2.2 树

## 2.3 栈和队列

## 2.4 哈希表

## 2.5 字符串

## 2.6 数组与矩阵

## 2.7 图

## 2.8 位运算