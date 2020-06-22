# 一、在行列都排好序的矩阵中找指定的数

```java
package com.example.problem1;

import java.util.Scanner;

/**
 * @Author: 98050
 * @Time: 2019-08-02 15:22
 * @Feature:
 */
public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int N = sc.nextInt();
        int M = sc.nextInt();
        int K = sc.nextInt();

        int[][] nums = new int[N][M];
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < M; j++) {
                nums[i][j] = sc.nextInt();
            }
        }

        int i = N - 1;
        int j = 0;
        boolean tag = false;
        while (i >= 0 && j < M){
            if (nums[i][j] == K){
                tag = true;
                break;
            }else if (nums[i][j] > K){
                i--;
            }else {
                j++;
            }
        }
        if (tag){
            System.out.println("Yes");
        }else {
            System.out.println("No");
        }
    }
}
```

# 二、最长的可整合子数组长度

```java
package com.example.problem2;

import java.io.Serializable;
import java.util.*;

/**
 * @Author: 98050
 * @Time: 2019-08-02 15:35
 * @Feature:
 */
public class Main {

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int length = sc.nextInt();
        Set<Integer> set = new TreeSet<Integer>();
        for (int i = 0; i < length; i++) {
            set.add(sc.nextInt());
        }
        Integer[] nums = new Integer[set.size()];
        set.toArray(nums);
        int result = 0;
        int len = 1;
        int pre = nums[0];

        for (int i = 1; i < nums.length; i++) {
            if (nums[i] - 1 == pre){
                len++;
            }else {
                len = 0;
            }
            result = Math.max(result, len);
            pre = nums[i];
        }
        System.out.println(result);
    }
}
```

