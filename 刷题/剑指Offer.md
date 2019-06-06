# 一、二维数组查找

思路：从左下角开始搜索，当前值比目标值大，那么就向上移动，比目标值小就向右移动。

或者从右上角开始搜索，当前值比目标值大就向左移动，比目标值小就向下移动

```java
package com.example.problem1;

public class Solution {
    public boolean Find(int target, int [][] array) {
        int row = array.length;
        if (row == 0){
            return false;
        }
        int col = array[0].length;
        int i = row - 1;
        int j = 0;
        while (i >= 0 && j < col){
            if (array[i][j] == target){
                return true;
            }else if (array[i][j] > target){
                i --;
            }else {
                j ++;
            }
        }
        return false;
    }
}
```

# 二、替换空格

思路：统计空格的个数，然后计算替换后字符串的总长度，最后设置两个指针，一个指向原来字符串的末尾，另外一个指向“新字符串”的末尾，从后往前碰到空格就替换，不是空格的话就直接赋值。

```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
    	int length = str.length();
    	//1.统计空格的个数
        int count = 0;
        for (int i = 0; i < length; i++) {
            if (str.charAt(i) == ' '){
                count++;
            }
        }
        //2.计算替换后字符串的长度
        int newLength = length + count * 2;
        str.setLength(newLength);
        //3.从后往前扫描，进行替换
        int i = length - 1;
        int j = newLength - 1;
        while (i >= 0){
            if (str.charAt(i) == ' '){
                str.setCharAt(j--, '0');
                str.setCharAt(j--, '2');
                str.setCharAt(j--, '%');
                i--;
            }else {
                str.setCharAt(j--,str.charAt(i));
                i--;
            }
        }
        return str.toString();
    }
}
```

# 三、从尾到头打印链表

思路：先递归到链表末尾，返回的时候将值添加到链表当中。

```java
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        solve(listNode,list);
        return list;
    }

    private void solve(ListNode listNode, ArrayList<Integer> list) {
        if (listNode == null){
            return;
        }
        solve(listNode.next, list);
        list.add(listNode.val);
    }
}
```

# 四、重建二叉树

思路：根据前序序列可以在中序序列中对二叉树进行划分。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，那么根节点就是1，在中序序列中进行划分：1的左半部分就是左子树，1的右半部分就是右子树，然后递归划分即可。完成上述过程使用递归，设置指针i、j指向前序遍历序列的首尾，设置指针k、l指向中序遍历序列的首尾；当i > j 或者 k > l的时候说明该节点的左子树或者右子树为空。

```java
package com.example.problem4;


import com.example.TreeNode;

public class Solution {
    public TreeNode reConstructBinaryTree(int [] pre, int [] in) {
        return rebuild(pre,0,pre.length - 1,in,0,in.length - 1);
    }

    private TreeNode rebuild(int[] pre, int i, int j, int[] in, int k, int l) {
        if (i > j || k > l){
            return null;
        }

        TreeNode root = new TreeNode(pre[i]);
        for (int m = k; m <= l; m++) {
            if (in[m] == pre[i]){
                root.left = rebuild(pre, i + 1, (m - k) + i, in, k, m - 1);
                root.right = rebuild(pre, (m - k) + i + 1, j, in, m + 1, l);
                break;
            }
        }
        return root;
    }
}
```

# 五、两个栈实现队列

思路：

> **两个栈实现队列**
>
> ​	入队：将元素都放入栈1
>
> ​	出队：判断栈2是否为空，空的话讲栈1的元素全部放到栈2中，然后弹出栈顶元素，如果不为空，	           那么直接读取栈2的栈顶元素即可。
>
> **两个队列实现栈**
>
> ​	入栈：将元素放入队列1
>
> ​	出栈：判断队列1中的元素个数是否等于1，等于的话就直接出队，不等于的话就将队列1中的元素		   出队放入到队列2中，直到队列1中只剩一个元素就停止，然后队列1出队，然后再把队列2中		   的元素放回到队列1中。

```java
package com.example.problem5;

import java.util.Stack;

public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    public void push(int node) {
        stack1.push(node);
    }

    public int pop() {
        int result;
        if (stack2.isEmpty()){
            while (!stack1.isEmpty()){
                stack2.push(stack1.pop());
            }
            result = stack2.pop();
        }else {
            result = stack2.pop();
        }
        return result;
    }
}
```

# 六、旋转数组的最小值

思路：

采用二分法解答这个问题， 

  mid = low + (high - low)/2 

  需要考虑三种情况： 

  (1)array[mid] > array[high]: 

  出现这种情况的array类似[3,4,5,6,0,1,2]，此时最小数字一定在mid的右边。 

  low = mid + 1 

  (2)array[mid] == array[high]: 

  出现这种情况的array类似 [1,0,1,1,1]   或者[1,1,1,0,1]，此时最小数字不好判断在mid左边 

  还是右边,这时只好一个一个试 ， 

  high = high - 1 

  (3)array[mid] < array[high]: 

  出现这种情况的array类似[2,2,3,4,5,6,6],此时最小数字一定就是array[mid]或者在mid的左 

  边。因为右边必然都是递增的。 

  high = mid 

  **注意这里有个坑：如果待查询的范围最后只剩两个数，那么mid** **一定会指向下标靠前的数字**  

  比如 array = [4,6] 

  array[low] = 4 ;array[mid] = 4 ; array[high] = 6 ; 

  如果high = mid - 1，就会产生错误， 因此high = mid 

  但情形(1)中low = mid + 1就不会错误

```java
package com.example.problem6;

public class Solution {
    
    public int minNumberInRotateArray(int [] array) {
        int length = array.length;
        int low = 0;
        int high = length - 1;
        while (low < high){
            int mid = (low + high) / 2;
            if (array[mid] > array[high]){
                low = mid + 1;
            }else if (array[mid] == array[high]){
                high--;
            }else {
                high = mid;
            }
        }
        return array[low];
    }
}
```

# 七、斐波那契数列

```java
package com.example.problem7;

public class Solution {
    public int Fibonacci(int n) {
        int[] result = new int[n + 1];
        result[0] = 0;
        if (n >= 1){
            result[1] = 1;
        }
        for (int i = 2; i < n + 1; i++) {
            result[i] = result[i - 1] + result[i - 2];
        }
        return result[n];
    }
}
```

# 八、跳台阶

```java
package com.example.problem8;

public class Solution {
    public int JumpFloor(int target) {
        if (target == 0 || target == 1){
            return 1;
        }
        return JumpFloor(target - 1) + JumpFloor(target - 2);
    }
}
```

# 九、变态跳台阶

普通思路：

```java
package com.example.problem9;

public class Solution {
    public int JumpFloorII(int target) {
        int[] result = new int[target + 1];
        if (target < 3){
            return target;
        }
        result[0] = 1;
        result[1] = 1;
        result[2] = 2;
        for (int i = 3; i < target + 1; i++) {
            for (int j = 0; j < i; j++) {
                result[i] += result[j];
            }
        }
        return result[target];
    }
}
```

高级：

因为n级台阶，第一步有n种跳法：跳1级、跳2级、到跳n级

跳1级，剩下n-1级，则剩下跳法是f(n-1)

跳2级，剩下n-2级，则剩下跳法是f(n-2)

 所以f(n)=f(n-1)+f(n-2)+...+f(1)

 因为f(n-1)=f(n-2)+f(n-3)+...+f(1)

 所以f(n)=2*f(n-1)

```java
public class Solution {
    public int JumpFloorII(int target) {
        if (target < 3){
            return target;
        }
        return 2*JumpFloorII(target - 1);
    }
}
```

# 十、矩形覆盖

```java
package com.example.problem10;

public class Solution {
    public int RectCover(int target) {
        if (target < 4){
            return target;
        }
        return RectCover(target - 1) + RectCover(target - 2);
    }
}
```

# 十一、二进制中1的个数

陷入死循环的方法：

```java
public int NumberOf1(int n) {
    int count = 0;
    while (n != 0){
        if ((n & 1) == 1) {
            count++;
        }
        n = n>>1;
    }
    return count;
}
```

因为负数在右移的时候高位补的是1，所以就陷入死循环了

转换思路，设置一个tag标记，让tag依次向左移，然后再与n做与运算

```java
public int NumberOf1(int n) {
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
```

什么时候停止呢?就是tag位超出边界。

另一种写法：

```java
public int NumberOf12(int n) {
    int count = 0;
    for (int i = 0; i < 32; i++) {
        if ((n >> i & 1) != 0){
            count++;
        }
    }
    return count;
}
```

最优解：

```class
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count = 0;
        while (n != 0){
            count++;
            n = n & (n - 1);
        }
        return count;
    }
}
```



# 十二、数值的整数次方

关键：考虑指数是负数的情况

```java
package com.example.problem12;

/**
 * @author 98050
 */
public class Solution {
    public double Power(double base, int exponent) {
        boolean tag = false;
        if (exponent < 0){
            tag = true;
            exponent = -exponent;
        }
        double result = 1;
        for (int i = 0; i < exponent; i++) {
            result *= base;
        }
        return tag ? 1 / result : result;
    }
}
```

# 十三、调整数值顺序使奇数位于偶数前面

思路：因为奇数在前偶数在后，所以碰到奇数就往前移动直到它的前一个数是奇数停止。

```java
package com.example.problem13;

public class Solution {
    public void reOrderArray(int [] array) {
        for (int i = 0; i < array.length; i++) {
            if (array[i] % 2 != 0){
                //向前移动
                int j = i;
                while (j > 0 && array[j - 1] % 2 == 0){
                    int temp = array[j];
                    array[j] = array[j - 1];
                    array[j - 1] = temp;
                    j--;
                }
            }
        }
    }
}
```

# 十四、链表中倒数第k个结点

思路：快慢指针，注意k大于链表长度的情况，所以在返回的时候需要进行判断

```java
package com.example.problem14;

import com.example.ListNode;

public class Solution {

    public ListNode FindKthToTail(ListNode head, int k) {
        ListNode slow = head;
        ListNode fast = head;
        int count = 0;
        while (fast != null){
            if (count < k){
                fast = fast.next;
                count++;
            }else {
                fast = fast.next;
                slow = slow.next;
            }
        }
        return count == k ? slow : null;
    }
}
```

# 十五、反转链表

思路一：使用头插法

```java
package com.example.problem15;

import com.example.ListNode;

public class Solution {

    public ListNode ReverseList(ListNode head) {
        ListNode root = new ListNode(0);
        while (head != null){
            ListNode temp = head;
            head = head.next;
            temp.next = root.next;
            root.next = temp;
        }
        return root.next;
    }
}
```

思路二：递归

递归出口：链表为空或者当前结点的下一个结点为空

每一次递归相当于保存了一条从当前结点到链表尾的链表。例如链表：1->2->3->4->5，递归到最后，返回5
第二次返回的时候当前head指向的节点是4，4->next->next指向的就是5，将4放在5的后面，5->4，将4后面的链断开

```java
public ListNode ReverseList2(ListNode head) {
    if (head == null || head.next == null){
        return head;
    }
    ListNode temp = ReverseList2(head.next);
    head.next.next = head;
    head.next = null;
    return temp;
}
```

# 十六、合并两个排序链表

思路：依次比较构建新链表就可以。

普通写法：

```java
public ListNode Merge(ListNode list1, ListNode list2) {
    ListNode head = new ListNode(0);
    ListNode temp = head;
    while (list1 != null && list2 != null){
        if (list1.val < list2.val){
            head.next = list1;
            list1 = list1.next;
        }else {
            head.next = list2;
            list2 = list2.next;
        }
        head = head.next;
    }
    if (list1 != null){
        head.next = list1;
    }
    if (list2 != null){
        head.next = list2;
    }
    return temp.next;
}
```

高级写法：

```java
public ListNode Merge(ListNode list1, ListNode list2) {
        if (list1 == null){
            return list2;
        }
        if (list2 == null){
            return list1;
        }
        ListNode head;
        if (list1.val < list2.val){
            head = list1;
            head.next = Merge(list1.next, list2);
        }else {
            head = list2;
            head.next = Merge(list1, list2.next);
        }
        return head;
    }
```

# 十七、树的子结构

思路：在A树中找到与B树根结点相同的点，然后再进行比较。

```java
package com.example.problem17;

import com.example.TreeNode;

public class Solution {
    public boolean HasSubtree(TreeNode root1, TreeNode root2) {
        if (root2 == null || root1 == null){
            return false;
        }
        boolean result = false;
        if (root1.val == root2.val){
            result = compare(root1,root2);
        }
        if (!result){
            return HasSubtree(root1.left, root2) || HasSubtree(root1.right, root2);
        }else {
            return result;
        }
    }

    private boolean compare(TreeNode root1, TreeNode root2) {
        if (root2 == null){
            return true;
        }
        if (root1 == null){
            return false;
        }
        if (root1.val != root2.val){
            return false;
        }
        return compare(root1.left, root2.left) && compare(root1.right, root2.right);
    }
}
```

# 十八、二叉树的镜像

思路：先序或者后序遍历，交换根节点的左右子树

```java
package com.example.problem18;

import com.example.TreeNode;

public class Solution {
    public void Mirror(TreeNode root) {
        if (root == null){
            return;
        }
        Mirror(root.left);
        Mirror(root.right);
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;
    }
}
```

# 十九、顺时针打印矩阵

思路：输出第一行，然后将数组的第一行删除，将数组逆时针旋转90度，然后再输出第一行。

```java
package com.example.problem19;


import java.util.ArrayList;
public class Solution {

    public ArrayList<Integer> printMatrix(int [][] matrix) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        int row = matrix.length;
        if (row == 0){
            return result;
        }
        int[][] temp = matrix;
        while (temp.length != 0) {
            //1.输出第一行
            printf(temp, result);
            //2.删除第一行
            temp = delete(temp);
            //3.旋转
            temp = route(temp);
        }
        return result;
    }

    private int[][] route(int[][] temp) {
        int row = temp.length;
        if (row == 0){
            return new int[0][0];
        }
        int col = temp[0].length;
        int[][] result = new int[col][row];
        for (int i = 0; i < col; i++) {
            for (int j = 0; j < row; j++) {
                result[i][j] = temp[j][col - 1 - i];
            }
        }
        return result;
    }

    private int[][] delete(int[][] temp) {
        int row = temp.length;
        if (row == 0){
            return new int[0][0];
        }
        int col = temp[0].length;
        int[][] result = new int[row - 1][col];
        for (int i = 0; i < row - 1; i++) {
            for (int j = 0; j < col; j++) {
                result[i][j] = temp[i + 1][j];
            }
        }
        return result;
    }

    private void printf(int[][] temp, ArrayList<Integer> result) {
        for (int i = 0; i < temp[0].length; i++) {
            result.add(temp[0][i]);
        }
    }
}
```

# 二十、包含min函数的栈

思路：使用两个栈，栈1正常使用，栈2用来保存最小值。

当元素进栈的时候，先压入栈1，然后判断当前元素是否比栈2的栈顶元素小，小的话就压入栈2

当元素出栈的时候，栈1直接弹出栈顶元素，然后该值与栈2的栈顶元素是否相等，相等的话栈2弹出栈顶元素

获取最小值时，直接读取栈2的栈顶元素

```java
package com.example.problem20;

import java.util.Stack;

public class Solution {

    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    public void push(int node) {
        stack1.push(node);
        if (stack2.isEmpty()){
            stack2.push(node);
        }else if (node < stack2.peek()){
            stack2.push(node);
        }
    }
    
    public void pop() {
        if (stack2.peek().equals(stack1.pop())){
            stack2.pop();
        }
    }
    
    public int top() {
        return stack1.peek();
    }
    
    public int min() {
        return stack2.peek();
    }
}
```

# 二十一、栈的压入、弹出序列

思路：入栈序列中的数和出栈序列的数进行比较，不相等就入栈，当入栈序列遍历完成后比较栈中剩余数的出栈序列与popA中剩余数的序列是否一致

```java
package com.example.problem21;

import java.util.Stack;

/**
 * @author 98050
 */
public class Solution {
    //入栈：12345
    //出栈：45321
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        Stack<Integer> stack = new Stack<Integer>();
        int j = 0;
        for (int i = 0; i < pushA.length; i++) {
            if (pushA[i] != popA[j]){
                stack.push(pushA[i]);
            }else {
                j++;
            }
        }
        for (int i = j; i < popA.length; i++) {
            if (popA[i] != stack.pop()){
                return false;
            }
        }
        return true;
    }
}
```

# 二十二、从上往下打印二叉树

树的层次遍历

```java
package com.example.problem22;

import com.example.TreeNode;

import java.util.ArrayList;
import java.util.LinkedList;

public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        if (root == null){
            return result;
        }
        LinkedList<TreeNode> queue = new LinkedList<TreeNode>();
        queue.add(root);
        while (!queue.isEmpty()){
            TreeNode treeNode = queue.poll();
            result.add(treeNode.val);
            if (treeNode.left != null){
                queue.add(treeNode.left);
            }
            if (treeNode.right != null){
                queue.add(treeNode.right);
            }
        }
        return result;
    }
}
```

# 二十三、二叉搜索树的后序遍历序列

思路：后序遍历序列，那么最后一个节点就是根节点，所以可以根据最后一个节点将数组分割为两部分，前一部分全部小于根节点，后一部分全部大于根节点，满足条件继续递归判断即可。

```java
package com.example.problem23;

public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
        int length = sequence.length;
        if (length == 0){
            return false;
        }
        return judge(sequence,0,length - 1);
    }

    private boolean judge(int[] sequence, int start, int end) {
        if (start >= end){
            return true;
        }
        int index = end - 1;
        while (index >= start && sequence[index] > sequence[end]){
            index--;
        }
        for (int i = start; i <= index; i++) {
            if (sequence[i] > sequence[end]){
                return false;
            }
        }
        return judge(sequence, start, index) && judge(sequence, index + 1, end - 1);
    }
}
```

# 二十四、二叉树中和为某一值的路径

思路：对树进行先序遍历，每遍历一个节点就用target的值减去当前节点的值，直到当前节点值与target相等，就找到一条路径。用一个list来保存遇到的节点，满足条件的时候就把list放入最终的结果集result中。

注意：每一次递归返回的时候要删除list中最后一个节点，这样就实现了路径保存

```java
package com.example.problem24;

import com.example.TreeNode;

import java.util.ArrayList;

public class Solution {
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
        ArrayList<ArrayList<Integer>> result = new ArrayList<ArrayList<Integer>>();
        if (root == null){
            return result;
        }
        ArrayList<Integer> temp = new ArrayList<Integer>();
        solve(root,target,temp,result);
        return result;
    }

    private void solve(TreeNode root, int target, ArrayList<Integer> temp, ArrayList<ArrayList<Integer>> result) {
        if (root == null){
            return;
        }
        temp.add(root.val);
        if (root.left == null && root.right == null && target == root.val){
            result.add(new ArrayList<Integer>(temp));
        }
        solve(root.left, target - root.val, temp, result);
        solve(root.right, target - root.val, temp, result);
        temp.remove(temp.size() - 1);
    }


}
```

# 二十五、复杂链表的复制

思路一：记录每一个节点random所指向的节点，然后重新赋值

```java
public RandomListNode Clone3(RandomListNode pHead) {
    if (pHead == null) {
        return null;
    }
    RandomListNode p = pHead;
    RandomListNode newHead = new RandomListNode(pHead.label);
    RandomListNode q = newHead;
    HashMap<RandomListNode, RandomListNode> map = new HashMap<RandomListNode, RandomListNode>();
    map.put(p,q);
    while (p.next != null){
        RandomListNode node = new RandomListNode(p.next.label);
        q.next = node;
        p = p.next;
        q = q.next;
        map.put(p, q);
    }
    p = pHead;
    q = newHead;

    while (q != null){
        q.random = map.get(p.random);
        p = p.next;
        q = q.next;
    }
    return newHead;
}
```

思路二：

- 将原来每个节点都复制一份放在原来节点的后边
- 根据原来的节点random指针来更新复制后节点random指针的值
- 将新旧链表进行拆分

```java
package com.example.problem25;

import com.example.RandomListNode;

public class Solution {
    public RandomListNode Clone(RandomListNode pHead)
    {
        RandomListNode temp = pHead;
        if (temp == null){
            return null;
        }
        //1.将每一个节点都复制放在原来节点的后边
        while (temp != null){
            RandomListNode node = new RandomListNode(temp.label);
            RandomListNode next = temp.next;
            temp.next = node;
            node.next = next;
            temp = next;
        }
        //2.更新random指针
        temp = pHead;
        while (temp != null){
            RandomListNode next = temp.next.next;
            if (temp.random != null){
                temp.next.random = temp.random.next;
            }
            temp = next;
        }
        //3.链表拆分
        temp = pHead;
        RandomListNode result = temp.next;
        while (temp != null){
            RandomListNode p = temp.next;
            temp.next = p.next;
            p.next = p.next == null ? null : p.next.next;
            temp = temp.next;
        }
        return result;
    }
}
```

# 二十六、二叉搜索树与双向链表

思路：利用树的中序遍历，定义头尾节点，一开始都指向树最左边的节点，然后依次将节点插入到链表尾部就可以，最后返回头节点。

递归版本：

```java
package com.example.problem26;

import com.example.TreeNode;

public class Solution {

    TreeNode left = null;
    TreeNode right = null;
    public TreeNode Convert(TreeNode pRootOfTree) {
       if (pRootOfTree == null){
           return null;
       }
       Convert(pRootOfTree.left);
       if (right == null){
           right = left = pRootOfTree;
       }else {
           right.right = pRootOfTree;
           pRootOfTree.left = right;
           right = pRootOfTree;
       }
       Convert(pRootOfTree.right);
       return left;
    }
}
```

非递归版本：

```java
public TreeNode Convert2(TreeNode pRootOfTree) {
    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode left = null;
    TreeNode right = null;
    while (!stack.isEmpty() || pRootOfTree != null){
        if (pRootOfTree != null){
            stack.push(pRootOfTree);
            pRootOfTree = pRootOfTree.left;
        }else {
            TreeNode temp = stack.pop();
            if (right == null){
                left = right = temp;
            }else {
                right.right = temp;
                temp.left = right;
                right = temp;
            }
            pRootOfTree = temp.right;
        }
    }
    return left;
}
```

# 二十七、字符串全排列

思路：https://blog.csdn.net/lyj2018gyq/article/details/87702078

```java
package com.example.problem27;

import java.util.ArrayList;
import java.util.Collections;


public class Solution {
    public ArrayList<String> Permutation(String str) {
        ArrayList<String> result = new ArrayList<String>();
        if (str.length() == 0){
            return result;
        }
        char[] temp = str.toCharArray();
        solve(temp,0,result);
        Collections.sort(result);
        return result;
    }

    private void solve(char[] temp, int i, ArrayList<String> result) {
        if (i == temp.length){
            result.add(new String(temp));
        }
        ArrayList<Character> list = new ArrayList<Character>();
        for (int j = i; j < temp.length; j++) {
            if (!list.contains(temp[j])) {
                list.add(temp[j]);
                swap(temp, i, j);
                solve(temp, i + 1, result);
                swap(temp, i, j);
            }
        }
    }

    private void swap(char[] temp, int i, int j) {
        char c = temp[i];
        temp[i] = temp[j];
        temp[j] = c;
    }
}
```

# 二十八、数组中出现次数超过一半的数字

思路：使用选举法，确定一个候选人，票数初始值为1，遍历数组，遇到相同的数字票数加一，不同的话就票数减一，当票数为0时重新确定候选人。最后统计数组中等于候选人的个数。

```java
package com.example.problem28;


public class Solution {
    public int MoreThanHalfNum_Solution(int [] array) {
        if (array.length == 0){
            return 0;
        }
        int candidate = array[0];
        int vote = 1;
        for (int i = 1; i < array.length; i++) {
            if (array[i] == candidate){
                vote++;
            }else {
                vote--;
                if (vote == 0){
                    candidate = array[i];
                    vote = 1;
                }
            }
        }
        int count = 0;
        for (int t : array){
            if (t == candidate){
                count++;
            }
        }
        return count > array.length / 2 ? candidate : 0;
    }
}
```

# 二十九、最小的k个数

思路：建小顶堆，然后输出前k个数

```java
package com.example.problem29;

import java.util.ArrayList;

public class Solution {
    

    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        if (k > input.length || k == 0 || input.length == 0){
            return result;
        }
        int length = input.length;
        //1.建堆
        for (int i = length / 2; i >= 0; i--) {
            heap(input,i,input.length - 1);
        }
        //2.调整
        for (int i = length - 1; i > length - 1 - k; i--) {
            int temp = input[0];
            input[0] = input[i];
            input[i] = temp;
            result.add(input[i]);
            heap(input,0,i - 1);
        }
        return result;
    }

    private void heap(int[] input, int start, int end) {
        int root = start;
        int left = 2 * root + 1;
        while (left <= end){
            if (left + 1 < end && input[left] > input[left + 1]){
                left ++;
            }
            if (input[root] > input[left]){
                int temp = input[root];
                input[root] = input[left];
                input[left] = temp;
            }
            root = left;
            left = 2 * root + 1;
        }
    }
}
```

# 三十、连续子数组的最大和

```java
package com.example.problem30;

public class Solution {
    public int FindGreatestSumOfSubArray(int[] array) {
        int length = array.length;
        if (length == 0){
            return 0;
        }
        int sum = 0;
        int result = array[0];
        for (int i = 0; i < array.length; i++) {
            if (sum + array[i] > array[i]){
                sum += array[i];
            }else {
                sum = array[i];
            }
            result = Math.max(result, sum);
        }
        return result;
    }
}
```

# 三十一、整数中1出现的次数

思路一：普通思路

```java
package com.example.problem31;

/**
 * @author 98050
 */
public class Solution {
    public int NumberOf1Between1AndN_Solution(int n) {
        int count = 0;
        for (int i = 1; i <= n; i++) {
            count += solve(i);
        }
        return count;
    }

    private int solve(int n) {
        int count = 0;
        while (n != 0){
            if (n % 10 == 1){
                count++;
            }
            n = n / 10;
        }
        return count;
    }

}
```

思路二：归纳总结

1. 如果第i位（自右至左，从1开始标号）上的数字为0，则第i位可能出现1的次数由更高位决定（若没有高位，视高位为0），等于更高位数字*当前位数的权重10^(i-1)。 

2. 如果第i位上的数字为1，则第i位上可能出现1的次数不仅受更高位影响，还受低位影响（若没有低位，视低位为0），等于更高位数字*当前位数的权重10^(i-1) +（低位数字+1）。 

3. 如果第i位上的数字大于1，则第i位上可能出现1的次数仅由更高位决定（若没有高位，视高位为0），等于（更高位数字+1) *当前位数的权重10^(i-1)。

```java
public int NumberOf1Between1AndN_Solution2(int n) {

    int result = solve2(n,1);
    return result;
}

private int solve2(int n,int x) {
    int high = n,low,curr,temp,i=1;
    int total = 0;
    while (high != 0){
        //取高位
        high = (int) (n / Math.pow(10,i));
        temp = (int) (n % Math.pow(10,i));
        curr = (int) (temp / Math.pow(10,i - 1));
        low = (int) (temp % Math.pow(10,i - 1));
        if (curr == x){
            total += high * Math.pow(10,i - 1) + low + 1;
        }else if (curr < x){
            total += high * Math.pow(10,i - 1);
        }else {
            total += (high + 1) * Math.pow(10,i - 1);
        }
        i++;
    }
    return total;
}
```

# 三十二、把数组排成最小的数

思路：定义比较规则

如果ab > ba，那么a > b

如果ab == ba ，那么a = b

如果ab < ba，那么a < b

举例：

a = 3，b = 32

332(ab) > 323(ba)，那么a > b

a = 3，c = 321

3321(ac) > 3213(ca)，那么a > c

b = 32 , c = 321

32321(bc) > 32132(cb)，那么b > c

最后顺序：3 32 321

```java
package com.example.problem32;

import java.util.ArrayList;
import java.util.List;

public class Solution {
    public String PrintMinNumber(int [] numbers) {
        int length = numbers.length;
        if (length == 0){
            return "";
        }
        List<String> list = new ArrayList<String>();
        for (int i : numbers){
            list.add(String.valueOf(i));
        }
        list.sort((c1,c2) -> {
            String temp1 = c1 + c2;
            String temp2 = c2 + c1;
            return temp1.compareTo(temp2);
        });
        StringBuilder result = new StringBuilder();
        for (String s : list){
            result.append(s);
        }
        return result.toString();
    }
}
```

# 三十三、丑数

思路一：判断每一个数是否是丑数，直到满足条件

```java
public int GetUglyNumber_Solution(int index) {
    List<Integer> list = new ArrayList<>();
    if (index == 0){
        return 0;
    }
    for (int i = 1; i < 7; i++) {
        list.add(i);
    }
    int num = 7;
    while (list.size() <= index){
        if (solve(num)){
            list.add(num);
        }
        num++;
    }
    return list.get(index - 1);
}

private boolean solve(int num) {
    while (num % 2 == 0){
        num /= 2;
    }
    while (num % 3 == 0){
        num /= 3;
    }
    while (num % 5 == 0){
        num /= 5;
    }
    return num == 1;
}
```

超时！！！！！！！！！！！！！！！！

思路二：  首先从丑数的定义知道，一个丑数的因子只有2,3,5，那么丑数p = 2 ^ x * 3 ^ y * 5 ^ z，换句话说一个丑数一定由另一个丑数乘以2或者乘以3或者乘以5得到，那么我们从1开始乘以2,3,5，就得到2,3,5三个丑数，在从这三个丑数出发乘以2,3,5就得到4，6,10,6，9,15,10,15,25九个丑数，我们发现这种方法会得到重复的丑数，而且我们题目要求第N个丑数，这样的方法得到的丑数也是无序的。通过维护三个队列，每次都取出三个队列队头最小的放入丑数数组，然后将该数乘以2，3，5分别放入相应的队列。

```java
public int GetUglyNumber_Solution2(int index) {
    if (index < 7){
        return index;
    }
    ArrayList<Integer> list = new ArrayList<>();
    list.add(1);
    LinkedList<Integer> queue2 = new LinkedList<>();
    LinkedList<Integer> queue3 = new LinkedList<>();
    LinkedList<Integer> queue5 = new LinkedList<>();
    queue2.add(2);
    queue3.add(3);
    queue5.add(5);
    int min = 0;
    while (list.size() < index){
        if (!queue2.isEmpty() && !queue3.isEmpty() && ! queue5.isEmpty()) {
            min = Math.min(queue2.getFirst(), Math.min(queue3.getFirst(), queue5.getFirst()));
            list.add(min);
        }
        if (!queue2.isEmpty() && queue2.getFirst() == min){
            queue2.pollFirst();
        }
        if (!queue3.isEmpty() && queue3.getFirst() == min){
            queue3.pollFirst();
        }
        if (!queue5.isEmpty() && queue5.getFirst() == min){
            queue5.pollFirst();
        }
        queue2.addLast(2 * min);
        queue3.addLast(3 * min);
        queue5.addLast(5 * min);
    }
    return list.get(index - 1);
}
```

代码优化：用三个指针代替队列，移动指针

```java
public int GetUglyNumber_Solution3(int index) {
    if (index < 7){
        return index;
    }
    int p2 = 0,p3 = 0, p5 = 0;
    int curr = 1;
    List<Integer> list = new ArrayList<>();
    list.add(curr);
    int min = 0;
    while (list.size() < index){
        min = Math.min(list.get(p2) * 2, Math.min(list.get(p3) * 3, list.get(p5) * 5));
        list.add(min);
        if (list.get(p2) * 2 == min){
            p2++;
        }
        if (list.get(p3) * 3 == min){
            p3++;
        }
        if (list.get(p5) * 5 == min){
            p5++;
        }
    }
    return list.get(index - 1);
}
```

# 三十四、第一个只出现一次的字符

思路：使用map进行统计，但是要保存读取的顺序，所以使用`LinkedHashMap`。

```java
package com.example.problem34;

import java.util.LinkedHashMap;

public class Solution {
    public int FirstNotRepeatingChar(String str) {
        int length = str.length();
        if (length == 0){
            return -1;
        }
        LinkedHashMap<Character,Integer> map = new LinkedHashMap<>();
        for (char c : str.toCharArray()){
            if (map.keySet().contains(c)){
                map.put(c, map.get(c) + 1);
            }else {
                map.put(c, 1);
            }
        }
        for (int i = 0; i < length; i++) {
            if (map.get(str.charAt(i)) == 1){
                return i;
            }
        }
        return -1;
    }
}
```

# 三十五、数组中的逆序对

思路：归并排序，在归并的过程中统计逆序对

```java
package com.example.problem35;

public class Solution {
    public int InversePairs(int [] array) {
        int length = array.length;
        if (length == 0){
            return 0;
        }
        int[] temp = new int[length];
        int count = solve(array,0,length - 1,temp);
        return count;
    }

    private int solve(int[] array, int start, int end, int[] temp) {
        if (start >= end){
            return 0;
        }
        int mid = (start + end) / 2;
        int left = solve(array, start, mid, temp) % 1000000007;
        int right = solve(array, mid + 1, end, temp) % 1000000007;
        int count = merge(array,start,mid,end,temp) % 1000000007;
        return (left + right + count) % 1000000007;
    }

    private int merge(int[] array, int start, int mid, int end, int[] temp) {
        int i = mid;
        int j = end;
        int result = 0;
        int index = end;
        while (i >= start && j > mid){
            if (array[i] > array[j]){
                result += (j - mid);
                if (result > 1000000007){
                    result %= 1000000007;
                }
                temp[index--] = array[i--];
            }else {
                temp[index--] = array[j--];
            }
        }
        while (i >= start){
            temp[index--] = array[i--];
        }
        while (j > mid){
            temp[index--] = array[j--];
        }
        for (int k = start; k <= end; k++) {
            array[k] = temp[k];
        }
        return result;
    }
}
```

# 三十六、两个链表的第一个公共节点

思路一：借助一个辅助list，两次遍历

```java
package com.example.problem36;

import com.example.ListNode;

import java.util.ArrayList;

public class Solution {

    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        ArrayList<ListNode> listNodes = new ArrayList<>();
        while (pHead1 != null){
            listNodes.add(pHead1);
            pHead1 = pHead1.next;
        }
        while (pHead2 != null){
            if (listNodes.contains(pHead2)){
                return pHead2;
            }
            pHead2 = pHead2.next;
        }
        return null;
    }
}
```

思路二：双指针

```java
public ListNode FindFirstCommonNode2(ListNode pHead1, ListNode pHead2) {
    ListNode p1 = pHead1;
    ListNode p2 = pHead2;
    while (p1 != p2){
        p1 = p1 == null ? pHead2 : p1.next;
        p2 = p2 == null ? pHead1 : p2.next;
    }
    return p1;
}
```

# 三十七、数字在排序数组中出现的次数

思路：排序数组，那么就用折半查找，查找第一个比目标值小的元素下标，然后再查找第一个比目标值大的元素下标，然后相减。

```java
package com.example.problem37;

public class Solution {
    
    public int GetNumberOfK(int [] array , int k) {
       int start = 0;
       int end = array.length - 1;
       int index1 = 0;
       while (start <= end){
           int mid = (start + end) / 2;
           if (array[mid] >= k){
               end = mid - 1;
           }else {
               start = mid + 1;
           }
       }
        // 保存k开始的位置
       index1 = start;
       start = 0;
       end = array.length - 1;
       while (start <= end){
           int mid = (start + end) / 2;
           if (array[mid] <= k){
               start = mid + 1;
           }else {
               end = mid - 1;
           }
       }
       //end中保存k结束的位置
       return end - index1 + 1;
    }
}
```

# 三十八、二叉树的深度

递归版本

```java
package com.example.problem38;

import com.example.TreeNode;

public class Solution {
    public int TreeDepth(TreeNode root) {
        if (root == null){
            return 0;
        }
        return Math.max(TreeDepth(root.left), TreeDepth(root.right)) + 1;
    }
}
```

非递归版本：层次遍历

```java
import java.util.LinkedList;
public class Solution {
    public int TreeDepth(TreeNode root) {
        LinkedList<TreeNode> queue = new LinkedList<>();
        if (root == null){
            return 0;
        }
        queue.addLast(root);
        int high = 0;
        while (!queue.isEmpty()){
            int count = queue.size();
            while (count > 0){
                TreeNode temp = queue.pollFirst();
                if (temp.left != null){
                    queue.addLast(temp.left);
                }
                if (temp.right != null){
                    queue.addLast(temp.right);
                }
                count--;
            }
            high++;
        }
        return high;
    }
}
```

# 三十九、平衡二叉树

思路一：分别计算左右子树的高度，高度差超过2就返回false，否则就依次判断。

```java
package com.example.problem39;

import com.example.TreeNode;

public class Solution {

    public boolean IsBalanced_Solution(TreeNode root) {
        if (root == null){
            return true;
        }
        int temp = Math.abs(TreeDepth(root.left) - TreeDepth(root.right));
        if (temp < 2){
            return IsBalanced_Solution(root.left) && IsBalanced_Solution(root.right);
        }else {
            return false;
        }
    }

    public int TreeDepth(TreeNode root) {
        if (root == null){
            return 0;
        }
        return Math.max(TreeDepth(root.left), TreeDepth(root.right)) + 1;
    }
}
```

思路二：对思路一代码进行改进，在计算树高时，判断左右子树高度差，如果大于1那么就直接返回-1即可。这样就变为从下往上遍历，不会重复遍历节点。

```java
public class Solution {
    public boolean IsBalanced_Solution(TreeNode root) {
        return TreeDepth(root) != -1;
    }

    public int TreeDepth(TreeNode root) {
        if (root == null){
            return 0;
        }
        int left = 0;
        int right = 0;
        left = TreeDepth(root.left);
        if (left == -1){
            return -1;
        }
        right = TreeDepth(root.right);
        if (right == -1){
            return -1;
        }
        return Math.abs(left - right) > 1 ? -1 : 1 + Math.max(left, right);
    }
}
```

# 四十、数组中只出现一次的数字

思路一：使用HashMap进行统计

```java
package com.example.problem40;

import java.util.ArrayList;
import java.util.HashMap;

//num1,num2分别为长度为1的数组。传出参数
//将num1[0],num2[0]设置为返回结果
public class Solution {
    public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
        HashMap<Integer,Integer> map = new HashMap<>();
        for (int i : array){
            if (map.keySet().contains(i)){
                map.put(i, map.get(i) + 1);
            }else {
                map.put(i, 1);
            }
        }
        ArrayList<Integer> list = new ArrayList<>();
        for (int i : array){
            if (map.get(i) == 1){
                list.add(i);
            }
        }
        num1[0] = list.get(0);
        num2[0] = list.get(1);
    }
}
```

思路二：使用异或运算，因为相同的数异或的结果为0，那么可以将数组中的数字做异或运算，那么最终结果就是两个只出现一次的数字，然后根据异或的结果1所在的最低位，把数字分成两半，每一半里都还有只出现一次的数据和成对出现的数据，然后分别对两个数组做异或运算。

```java
//num1,num2分别为长度为1的数组。传出参数
//将num1[0],num2[0]设置为返回结果
public class Solution {
    public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
        int result = 0;
        for (int i : array){
            result ^= i;
        }
        int firstI = findFirstI(result);
        for (int i : array){
            if (check(i,firstI)){
                num1[0] ^= i;
            }else {
                num2[0] ^= i;
            }
        }
    }

    private boolean check(int i, int firstI) {
        return ((i >> firstI) & 1) == 1;
    }

    private int findFirstI(int result) {
        int count = 0;
        while ((result & 1) != 1){
            result = result >> 1;
            count ++;
        }
        return count;
    }
}
```

# 四十一、和为S的连续正数序列

思路：定义快慢指针，通过求和公式计算两个指针之间数字的和，与S相等的话就将区间内的数字放入list中，然后快指针或者慢指针随便移动一个即可；如果比S小，那么快指向前针移动；比S大，慢指针向前移动

注：这样可以保证序列内有序且序列间有序

```java
package com.example.problem41;

import java.util.ArrayList;
public class Solution {
    public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
        ArrayList<ArrayList<Integer>> result = new ArrayList<>();
        int low = 1;
        int high = 2;
        while (low < high){
            int temp = (low + high) * (high - low + 1) / 2;
            if (temp == sum){
                ArrayList<Integer> list = new ArrayList<>();
                for (int i = low; i <= high; i++) {
                    list.add(i);
                }
                result.add(list);
                high++;
            }else if (temp < sum){
                high++;
            }else {
                low++;
            }
        }
        return result;
    }
}
```

# 四十二、和为S的两个数

思路：首尾指针，进行比较移动

```java
package com.example.problem42;

import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> FindNumbersWithSum(int [] array,int sum) {
        ArrayList<Integer> result = new ArrayList<>();
        int min = Integer.MAX_VALUE;
        int low = 0;
        int high = array.length - 1;
        while (low < high){
            int temp = array[low] + array[high];
            int temp2 = array[low] * array[high];
            if (temp == sum){
                if (temp2 < min) {
                    result.add(array[low]);
                    result.add(array[high]);
                    min = temp2;
                }
                high--;
            }else if (temp < sum){
                low++;
            }else {
                high--;
            }
        }
        return result;
    }
}
```

# 四十三、左旋转字符串

思路：字符序列S=”abcXYZdef”为例

第一步：S = "cbaXYZdef"

第二步：S="cbafedZYX"

第三步：S="XYZdefabc"

```java
package com.example.problem43;

public class Solution {
    public String LeftRotateString(String str,int n) {
        int length = str.length();
        if (length == 0){
            return "";
        }
        char[] array = str.toCharArray();
        reverse(array,0,n - 1);
        reverse(array, n,length - 1);
        reverse(array, 0, length - 1);
        return new String(array);
    }

    private void reverse(char[] array, int low, int high) {
        while (low < high){
            char temp = array[low];
            array[low] = array[high];
            array[high] = temp;
            low++;
            high--;
        }
    }
}
```

# 四十四、反转单词顺序列

思路一：可以使用额外空间，那么将字符串根据空格分割，然后再进行拼接

```java
package com.example.problem44;

public class Solution {

    public String ReverseSentence(String str) {
        String[] temp = str.split(" ");
        if (temp.length == 0){
            return str;
        }
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = temp.length - 1; i >= 0; i--) {
            if (i != 0) {
                stringBuilder.append(temp[i] + " ");
            }else {
                stringBuilder.append(temp[i]);
            }
        }
        return stringBuilder.toString();
    }
}
```

思路二：先反转每个单词，然后再反转整个句子

```java
public String ReverseSentence2(String str) {
    char[] temp = str.toCharArray();
    int pre = 0;
    for (int i = 0; i < temp.length; i++) {
        if (temp[i] == ' '){
            reverse(temp, pre, i - 1);
            pre = i + 1;
        }else if (i == temp.length - 1){
            reverse(temp, pre, temp.length - 1);
        }
    }
    reverse(temp, 0, temp.length - 1);
    return new String(temp);
}

private void reverse(char[] array, int low, int high) {
    while (low < high){
        char temp = array[low];
        array[low] = array[high];
        array[high] = temp;
        low++;
        high--;
    }
}
```

# 四十五、扑克牌顺子

思路：长度为5的顺子，那么最大值和最小值的差值不可能超过5，且保证除0外没有重复牌

```java
package com.example.problem45;

import java.util.ArrayList;

public class Solution {
    public boolean isContinuous(int [] numbers) {
        if (numbers.length == 0){
            return false;
        }
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        ArrayList<Integer> list = new ArrayList<>();
        for (int i : numbers){
            if (i != 0) {
                if (list.contains(i)) {
                    return false;
                } else {
                    list.add(i);
                    max = Math.max(max, i);
                    min = Math.min(min, i);
                }
            }
        }
        if (max - min >= 5){
            return false;
        }else {
            return true;
        }
    }
}
```

# 四十六、孩子们的游戏

思路：约瑟夫环，用list来模拟游戏过程

```java
package com.example.problem46;

import java.util.ArrayList;
import java.util.List;

public class Solution {
    public int LastRemaining_Solution(int n, int m) {
        if (n == 0 || m == 0){
            return -1;
        }
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            list.add(i);
        }
        int index = 0;
        while (list.size() > 1){
            index = (index + m - 1) % list.size();
            list.remove(index);
        }
        return list.get(0);
    }
}
```

```java
public int LastRemaining_Solution2(int n, int m) {
    if (n == 0 || m == 0){
        return -1;
    }
    int[] nums = new int[n];
    int i = -1,step = 0,count = n;
    while (count > 0){
        i++;
        if (i >= n){
            i = 0;
        }
        if (nums[i] == -1){
            continue;
        }
        step++;
        if (step == m){
            nums[i] = -1;
            step = 0;
            count--;
        }
    }
    return i;
}
```

# 四十七、求和

递归：

```java
package com.example.problem47;

/**
 * @author 98050
 */
public class Solution {
    public int Sum_Solution(int n) {
        int result = n;
        if (n != 0){
            result += Sum_Solution(n - 1);
        }
        return result;
    }
}
```

```java
    public int Sum_Solution2(int n) {
        int res = n;
        boolean b = res > 0 && (res += Sum_Solution2(n - 1)) > 0;
        return res;
    }
```

# 四十八、另类加法

思路：

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
package com.example.problem48;

public class Solution {
    public int Add(int num1,int num2) {
        while (num2 != 0){
            int temp = num1 ^ num2;
            num2 = (num2 & num1) << 1;
            num1 = temp;
        }
        return num1;
    }
}
```

# 四十九、把字符串转换成整数

```java
package com.example.problem49;


public class Solution {

    public int StrToInt(String str) {
        int length = str.length();
        if (length == 0){
            return 0;
        }
        int result = 0;
        int lex = 0;
        for (int i = length - 1; i >= 0; i--) {
            char c = str.charAt(i);
            if (c >= '0' && c <= '9'){
                result += (int) Math.pow(10, lex) * (str.charAt(i) - '0');
                lex ++;
                System.out.println(result);
            }else if ((c == '+' || c == '-') && i == 0){
                result = c == '-' ? -result : result;
            }else {
                return 0;
            }
        }
        return result;
    }
}
```

# 五十、数组中重复的数字

思路一：万能的map

```java
package com.example.problem50;

import java.util.LinkedHashMap;

public class Solution {
    // Parameters:
    //    numbers:     an array of integers
    //    length:      the length of array numbers
    //    duplication: (Output) the duplicated number in the array number,length of duplication array is 1,so using duplication[0] = ? in implementation;
    //                  Here duplication like pointor in C/C++, duplication[0] equal *duplication in C/C++
    //    这里要特别注意~返回任意重复的一个，赋值duplication[0]
    // Return value:       true if the input is valid, and there are some duplications in the array number
    //                     otherwise false
    public boolean duplicate(int numbers[],int length,int [] duplication) {
        LinkedHashMap<Integer,Integer> map = new LinkedHashMap<>();
        if (length == 0){
            return false;
        }
        for (int num : numbers){
            if (map.keySet().contains(num)){
                map.put(num, map.get(num) + 1);
            }else {
                map.put(num, 1);
            }
        }
        for (int num : numbers){
            if (map.get(num) > 1){
                duplication[0] = num;
                return true;
            }
        }
        return false;
    }
}
```

思路二：因为数组内的数字都在0~n-1范围内，。所以可以利用现有数组设置标志，当一个数字被访问过后，可以设置对应位上的数 + n，之后再遇到相同的数时，会发现对应位上的数已经大于等于n了，那么直接返回这个数即可。

```java
public boolean duplicate2(int numbers[],int length,int [] duplication) {
    if (length == 0){
        return false;
    }
    for (int i = 0; i < length; i++) {
        int temp = numbers[i];
        if (temp >= length){
            temp -= length;
        }
        int temp2 = numbers[temp];
        if (temp2 <= length){
            numbers[temp] += length;
        }else {
            duplication[0] = temp;
            return true;
        }
    }
    return false;
}
```

# 五十一、构建乘积数组

思路一：时间复杂度O(n^2)

```java
package com.example.problem51;

public class Solution {
    public int[] multiply(int[] A) {
        int length = A.length;
        int[] result = new int[length];
        if (length == 0){
            return result;
        }
        for (int i = 0; i < length; i++) {
            int temp = 1;
            for (int j = 0; j < length; j++) {
                if (i != j){
                    temp *= A[j];
                }
            }
            result[i] = temp;
        }
        return result;
    }
}
```

思路二：构建二维矩阵

**B[i]的值可以看作下图的矩阵中每行的乘积。** 

下三角用连乘可以很容求得，上三角，从下向上也是连乘。 

因此我们的思路就很清晰了，先算下三角中的连乘，即我们先算出B[i]中的一部分，然后倒过来按上三角中的分布规律，把另一部分也乘进去。

![](http://mycsdnblog.work/201919111326-X.png)

O(n)

```java
public int[] multiply2(int[] A) {
    int length = A.length;
    int[] result = new int[length];
    if (length == 0){
        return result;
    }
    result[0] = 1;
    //计算下三角连乘
    for (int i = 1; i < length; i++) {
        result[i] = result[i - 1] * A[i - 1];
    }
    int temp = 1;
    for (int i = length - 2; i >= 0; i--) {
        temp *= A[i + 1];
        result[i] *= temp;
    }
    return result;
}
```

# 五十二、正则表达式匹配

# 五十三、表示数值的字符串

```java
package com.example.problem53;


public class Solution {
    
    public boolean isNumeric(char[] str) {
        //注意："-.123"是合法的
        boolean tag1 = false; //小数点
        boolean tag2 = false; //正负号
        boolean tag3 = false; // E或e
        int length = str.length;
        if (length == 0){
            return false;
        }
        for (int i = 0; i < length; i++) {
            //1.正负号的出现规则：出现在首位，其后必须是数字或者小数点；不在首位，那么必须出现在e或E的后边
            //2.小数点的出现规则：只能出现一次，且其后必须为数字，前面的可以是数字或者正负号，而且小数点不能出现在E或e的后面
            //3.E或e只能出现一次，其后可以是数字或者正负号，其前必须是数字
            if (str[i] == '.'){
                if (tag3 || tag1){
                    return false;
                }else {
                    boolean t = i + 1 < length && str[i + 1] >='0' && str[i + 1] <= '9';
                    boolean t2 = i - 1 >= 0 && ((str[i - 1] >='0' && str[i - 1] <= '9') || (str[i - 1] =='+' || str[i - 1] == '-'));
                    if (t && t2){
                        tag1 = true;
                    }else {
                        return false;
                    }
                }
            }else if (str[i] == '+' || str[i] == '-'){
                if (!tag2){
                    if (i == 0){
                        tag2 = true;
                    }else if (str[i - 1] == 'e' || str[i - 1] == 'E'){
                        tag2 = true;
                    }else {
                        return false;
                    }
                }else if (str[i - 1] == 'e' || str[i - 1] == 'E'){
                    continue;
                }else {
                    return false;
                }
            }else if (str[i] == 'e' || str[i] == 'E'){
                if (tag3){
                    return false;
                }else {
                    if (i + 1 < length &&i - 1 >= 0 && str[i-1] >= '0' && str[i-1] <= '9'){
                        tag3 = true;
                    }else {
                        return false;
                    }
                }
            }else if (str[i] < '0' || str[i] > '9'){
                return false;
            }
        }
        return true;
    }
}
```

# 五十四、字符流中第一个不重复的字符

思路一：使用一个链表，每次插入的时候都进行判断，如何重复那就删除list中对应的字符

```java
package com.example.problem54;

import java.util.LinkedList;

public class Solution {

    LinkedList<Character> list = new LinkedList<>();

    //Insert one char from stringstream
    public void Insert(char ch)
    {
        if (list.size() > 0 && list.contains(ch)){
            list.remove(list.indexOf(ch));
        }else {
            list.add(ch);
        }
    }
  //return the first appearence once char in current stringstream
    public char FirstAppearingOnce()
    {
        if (list.size() != 0){
            return list.getFirst();
        }else {
            return '#';
        }
    }
}
```

# 五十五、链表中环的入口结点

思路一：使用list

```java
package com.example.problem55;

import com.example.ListNode;

import java.util.ArrayList;

public class Solution {

    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
        ArrayList<ListNode> list = new ArrayList<>();
        while (pHead != null){
            if (list.contains(pHead)){
                return pHead;
            }else {
                list.add(pHead);
            }
            pHead = pHead.next;
        }
        return null;
    }
}
```

思路二：快慢指针

```java
public ListNode EntryNodeOfLoop2(ListNode pHead){
    if (pHead.next == null){
        return null;
    }
    ListNode p1 = pHead;
    ListNode p2 = pHead;
    while (p1 != null && p2 != null){
        p1 = p1.next;
        p2 = p2.next.next;
        if (p1 == p2){
            p1 = pHead;
            while (p1 != p2){
                p1 = p1.next;
                p2 = p2.next;
            }
            return p1;
        }
    }
    return null;
}
```

思路三：断链法

```java
public ListNode EntryNodeOfLoop3(ListNode pHead){
    ListNode pre = pHead;
    ListNode walk = pre.next;
    if (walk == null){
        return null;
    }
    while (walk != null){
        pre.next = null;
        pre = walk;
        walk = walk.next;
    }
    return pre;
}
```

具体讲解：https://blog.csdn.net/lyj2018gyq/article/details/88683291

# 五十六、删除链表中重复的结点

思路一：新增一个头结点，然后定义两个指针，快指针用来扫描重复结点，慢指针用来记录重复结点的前一个不重复结点。

```java
package com.example.problem56;

import com.example.ListNode;

public class Solution {
    public ListNode deleteDuplication(ListNode pHead)
    {
        ListNode node = new ListNode(-1);
        node.next = pHead;
        ListNode p1 = node;
        ListNode p2 = pHead;
        while (p2 != null){
            ListNode next = p2.next;
            if (next != null && next.val == p2.val){
                while (p2 != null && next.val == p2.val){
                    p2 = p2.next;
                }
                p1.next = p2;
            }else {
                p2 = p2.next;
                p1 = p1.next;
            }
        }
        return node.next;
    }
}
```

思路二：使用map对链表进行统计，然后构造新链表

```java
public ListNode deleteDuplication2(ListNode pHead){
    LinkedHashMap<Integer,Integer> map = new LinkedHashMap<>();
    while (pHead != null){
        if (map.keySet().contains(pHead.val)){
            map.put(pHead.val, map.get(pHead.val) + 1);
        }else {
            map.put(pHead.val, 1);
        }
        pHead = pHead.next;
    }

    ListNode node = new ListNode(0);
    ListNode head = node;
    for (Integer integer : map.keySet()){
        if (map.get(integer) == 1){
            node.next = new ListNode(integer);
            node = node.next;
        }
    }
    node.next = null;
    return head.next;
}
```

# 五十七、二叉树的下一个结点

思路一：通过该结点找到树的根节点，然后中序遍历二叉树，最后找到该节点的下一个节点

```java
package com.example.problem57;

import com.example.TreeLinkNode;

import java.util.ArrayList;

public class Solution {

    public TreeLinkNode GetNext(TreeLinkNode pNode)
    {
        TreeLinkNode temp = pNode;
        //1.先找根节点
        while (pNode.next != null){
            pNode = pNode.next;
        }
        //2.中序遍历
        ArrayList<TreeLinkNode> list = new ArrayList<>();
        InOrder(list,pNode);
        int index = list.indexOf(temp);
        if (index < list.size() - 1){
            return list.get(index + 1);
        }else {
            return null;
        }
    }

    private void InOrder(ArrayList<TreeLinkNode> list, TreeLinkNode pNode) {
        if (pNode == null){
            return;
        }
        InOrder(list,pNode.left);
        list.add(pNode);
        InOrder(list,pNode.right);
    }
}
```

思路二：

![](http://mycsdnblog.work/201919121335-o.png)

**1、若该节点存在右子树：则下一个节点为右子树最左子节点（如图节点     B     ）**   

**2、若该节点不存在右子树：这时分两种情况：**   

​     **2.1      该节点为父节点的左子节点，则下一个节点为其父节点（如图节点     D     ）**   

​     **2.2      该节点为父节点的右子节点，则沿着父节点向上遍历，直到找到一个节点的父节点的左子节点为该节点，则该节点的父节点下一个节点（如图节点 I ，沿着父节点一直向上查找找到B（B为其父节点的左子节点），则B 的父节点A为下一个节点）。**

```java
public TreeLinkNode GetNext2(TreeLinkNode pNode){
    if (pNode == null){
        return null;
    }
    if (pNode.right != null){
        pNode = pNode.right;
        while (pNode.left != null){
            pNode = pNode.left;
        }
        return pNode;
    }else if (pNode.next != null && pNode.next.left == pNode){
        return pNode.next;
    }else if (pNode.next != null && pNode.next.right == pNode){
        while (pNode.next != null && pNode.next.left != pNode){
            pNode = pNode.next;
        }
        return pNode.next;
    }
    return null;
}
```

# 五十八、对称的二叉树

思路一：递归判断左右子树，左子树的右节点和右子树的左节点，左子树的左节点和右子树的右节点

```java
package com.example.problem58;

import com.example.TreeNode;

public class Solution {
    boolean isSymmetrical(TreeNode pRoot)
    {
        if (pRoot == null){
            return true;
        }
        return compare(pRoot.left,pRoot.right);
    }

    private boolean compare(TreeNode left, TreeNode right) {
        if (left == null && right == null){
            return true;
        }
        if (left != null && right != null){
            return left.val == right.val && compare(left.right, right.left) && compare(left.left, right.right);
        }
        return false;
    }
}
```

思路二：利用树的层次遍历，首先将根节点左右孩子入队，然后左右孩子出队，进行判断。入队的时候按顺序入队：左子树的右节点、右子树的左节点、左子树的左节点、右子树的右节点

```java
boolean isSymmetrical2(TreeNode pRoot){
    if (pRoot == null){
        return true;
    }
    LinkedList<TreeNode> queue = new LinkedList<>();
    queue.addLast(pRoot.left);
    queue.addLast(pRoot.right);
    while (!queue.isEmpty()){
        TreeNode left = queue.pollFirst();
        TreeNode right = queue.pollFirst();
        if (left == null && right == null){
            continue;
        }
        if (left == null || right == null){
            return false;
        }
        if (left.val != right.val){
            return false;
        }
        queue.addLast(left.left);
        queue.addLast(right.right);
        queue.addLast(left.right);
        queue.addLast(right.left);

    }
    return true;
}
```

# 五十九、按之字形顺序打印二叉树

思路：二叉树层次遍历，出队的次数就是每一层的节点数，当遇到偶数层的时候将list反转一下

```java
package com.example.problem59;

import com.example.TreeNode;

import java.util.ArrayList;
import java.util.Collections;
import java.util.LinkedList;

public class Solution {
    public ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        LinkedList<TreeNode> queue = new LinkedList<>();
        ArrayList<ArrayList<Integer> > result = new ArrayList<>();
        if (pRoot == null){
            return result;
        }
        int level = 0;
        queue.addLast(pRoot);
        while (!queue.isEmpty()){
            level++;
            int count = queue.size();
            ArrayList<Integer> list = new ArrayList<>();
            while (count > 0){
                TreeNode node = queue.pollFirst();
                list.add(node.val);
                if (node.left != null){
                    queue.addLast(node.left);
                }
                if (node.right != null){
                    queue.addLast(node.right);
                }
                count--;
            }
            if (level % 2 == 0){
                Collections.reverse(list);
            }
            result.add(list);
        }
        return result;
    }

}
```

# 六十、把二叉树打印成多行

思路：层次遍历

```java
package com.example.problem60;

import com.example.TreeNode;

import java.util.ArrayList;
import java.util.Collections;
import java.util.LinkedList;


public class Solution {
    ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        LinkedList<TreeNode> queue = new LinkedList<>();
        ArrayList<ArrayList<Integer> > result = new ArrayList<>();
        if (pRoot == null){
            return result;
        }
        queue.addLast(pRoot);
        while (!queue.isEmpty()){
            int count = queue.size();
            ArrayList<Integer> list = new ArrayList<>();
            while (count > 0){
                TreeNode node = queue.pollFirst();
                list.add(node.val);
                if (node.left != null){
                    queue.addLast(node.left);
                }
                if (node.right != null){
                    queue.addLast(node.right);
                }
                count--;
            }
            result.add(list);
        }
        return result;
    }
    
}
```

# 六十一、序列化二叉树

思路：二叉树的遍历和重建

```java
package com.example.problem61;

import com.example.TreeNode;

public class Solution {
    String Serialize(TreeNode root) {
        String result = "";
        if (root == null){
            result += "#,";
            return result;
        }
        result += root.val + ",";
        result += Serialize(root.left);
        result += Serialize(root.right);
        return result;
  }

    TreeNode Deserialize(String str) {
       String[] nums = str.split(",");
       return build(nums);
  }

    int index = -1;
    private TreeNode build(String[] nums) {
        index++;
        if (index >= nums.length){
            return null;
        }
        TreeNode root = null;
        if (!"#".equals(nums[index])){
            root = new TreeNode(Integer.valueOf(nums[index]));
            root.left = build(nums);
            root.right = build(nums);
        }
        return root;
    }


}
```

# 六十二、二叉搜索树的第k个节点

思路1：中序遍历二叉树，然后直接返回第k个节点

```java
package com.example.problem62;

import com.example.TreeNode;

import java.util.ArrayList;
import java.util.List;

public class Solution {
    TreeNode KthNode(TreeNode pRoot, int k)
    {
        List<TreeNode> list = new ArrayList<>();
        InOrder(pRoot,list);
        if (k - 1 < 0 || k - 1 >= list.size()){
            return null;
        }
        return list.get(k - 1);
    }

    private void InOrder(TreeNode pRoot, List<TreeNode> list) {
        if (pRoot == null){
            return;
        }
        InOrder(pRoot.left, list);
        list.add(pRoot);
        InOrder(pRoot.right, list);
    }


}
```

思路2：中序遍历，非递归，

```java
TreeNode KthNode2(TreeNode pRoot, int k){
    Stack<TreeNode> stack = new Stack<>();
    int count = 0;
    while (!stack.isEmpty() || pRoot != null){
        if (pRoot != null){
            stack.push(pRoot);
            pRoot = pRoot.left;
        }else {
            TreeNode node = stack.pop();
            count++;
            if (count == k){
                return node;
            }
            pRoot = node.right;
        }
    }
    return null;
}
```

思路3：中序遍历，递归

```java
int count = 0;
TreeNode KthNode3(TreeNode pRoot, int k){
    if (pRoot == null){
        return null;
    }
    TreeNode node = KthNode3(pRoot.left, k);
    if (node != null){
        return node;
    }
    if (++count == k){
        return pRoot;
    }
    node = KthNode3(pRoot.right, k);
    if (node != null){
        return node;
    }
    return null;
}
```

# 六十三、数据流中的中位数

思路1：插入排序

```java
package com.example.problem63;


import java.util.ArrayList;
import java.util.List;

public class Solution {

    List<Integer> list = new ArrayList<>();
    
    public void Insert(Integer num) {
        list.add(num);
        int temp = num;
        for (int i = 0; i <= list.size() - 1; i++) {
            if (list.get(i) > temp){
                int j = list.size() - 1;
                while (j > i){
                    list.set(j, list.get(j - 1));
                    j--;
                }
                list.set(j, temp);
                break;
            }
        }
    }

    public Double GetMedian() {
        int index = list.size();
        if (index % 2 != 0){
            return Double.valueOf(list.get(index / 2 + 1));
        }else {
            return (list.get(index / 2) + list.get(index / 2 - 1)) / 2.0;
        }
    }


}
```

思路2：将中位数的两端分别存储，使用两个堆：大顶堆和小顶堆。大顶堆用来存放中位数左边的数据，小顶堆用来存放中位数右边的数据

元素先插大顶堆，然后判断：

1.如果两个堆的元素个数相差超过1，那么将大顶堆的堆顶插入到小顶堆中

2.如果大顶堆的堆顶元素大于小顶堆的堆顶元素，那么将大顶堆的堆顶插入到小顶堆中

目的：

1.两个堆的元素个数不超过1

2.大顶堆的元素都比小顶堆的元素小

取中位数：

如果两个堆中元素的个数相等，那么将两个堆的堆顶出栈

否则返回个数大的那个

```java
package com.example.problem63;

import java.util.Comparator;
import java.util.PriorityQueue;

/**
 * @Author: 98050
 * @Time: 2019-04-15 11:37
 * @Feature:
 */
public class Solution2 {

    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    public void Insert(Integer num) {
        maxHeap.add(num);
        if (maxHeap.size() - minHeap.size() > 1){
            minHeap.add(maxHeap.poll());
        }
        if (maxHeap.size() != 0 && minHeap.size() != 0){
            if (maxHeap.peek() > minHeap.peek()){
                minHeap.add(maxHeap.poll());
            }
        }
    }

    public Double GetMedian() {
        int count1 = minHeap.size();
        int count2 = maxHeap.size();
        if (count1 == count2 && count1 != 0){
            return (minHeap.peek() + maxHeap.peek()) / 2.0;
        }else if (count1 > count2){
            return Double.valueOf(minHeap.peek());
        }else if (count1 < count2){
            return Double.valueOf(maxHeap.peek());
        }else {
            return (double) 0;
        }
    }
}
```

# 六十四、滑动窗口的最大值

思路一：

```java
package com.example.problem64;

import java.util.ArrayList;

public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        ArrayList<Integer> list = new ArrayList<>();
        if (size == 0 || size > num.length){
            return list;
        }
        for (int i = 0; i <= num.length - size; i++) {
            list.add(find(num,i,i+size));
        }
        return list;
    }

    private Integer find(int[] num, int start, int end) {
        int max = num[start];
        for (int i = start; i < end; i++) {
            max = Math.max(max, num[i]);
        }
        return max;
    }
}
```

思路二：

```java
public ArrayList<Integer> maxInWindows2(int [] num, int size){
    ArrayList<Integer> list = new ArrayList<>();
    LinkedList<Integer> queue = new LinkedList<>();
    if (size == 0 || size > num.length){
        return list;
    }
    int index = 0;
    for (int i = 0; i < num.length; i++) {
        index = i - size + 1;
        if (queue.isEmpty()){
            queue.addLast(i);
        }else if (index > queue.peekFirst()){
            queue.pollFirst();
        }
        while (!queue.isEmpty() && num[queue.peekLast()] <= num[i]){
            queue.pollLast();
        }
        queue.addLast(i);
        if (index >= 0){
            list.add(num[queue.peekFirst()]);
        }
    }
    return list;
}
```

# 六十五、矩阵中的路径

```java
package com.example.problem65;

public class Solution {
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str)
    {
        int[] flag = new int[matrix.length];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (find(matrix,rows,cols,i,j,str,0,flag)){
                    return true;
                }
            }
        }
        return false;
    }

    private boolean find(char[] matrix, int rows, int cols, int i, int j, char[] str, int k, int[] flag) {
        int index = i * cols + j;
        if (i >= rows || i < 0 || j >= cols || j < 0 || matrix[index] != str[k] || flag[index] == 1){
            return false;
        }
        if (k == str.length - 1){
            return true;
        }
        flag[index] = 1;
        boolean tag = find(matrix, rows, cols, i - 1, j, str, k + 1, flag) ||
                      find(matrix, rows, cols, i + 1, j, str, k + 1, flag) ||
                      find(matrix, rows, cols, i, j - 1, str, k + 1, flag) ||
                      find(matrix, rows, cols, i, j + 1, str, k + 1, flag) ;
        if (tag){
            return true;
        }
        flag[index] = 0;
        return false;
    }


}
```

# 六十六、机器人的运动范围

```java
package com.example.problem66;

public class Solution {
    public int movingCount(int threshold, int rows, int cols)
    {
        int[][] result = new int[rows][cols];
        return solve(result,rows,cols,0,0,threshold);
    }

    private int solve(int[][] result, int rows, int cols, int i, int j, int threshold) {
        if (i >= rows || i < 0 || j >= cols || j < 0 || result[i][j] == - 1 || sum(i) + sum(j) > threshold){
            return 0;
        }
        result[i][j] = -1;
        return solve(result, rows, cols, i + 1, j, threshold) +
                solve(result, rows, cols, i - 1, j, threshold) +
                solve(result, rows, cols, i, j + 1, threshold) +
                solve(result, rows, cols, i, j - 1, threshold) + 1;
    }

    private int sum(int i) {
        int result = 0;
        while (i > 0){
            result += (i % 10);
            i /= 10;
        }
        return result;
    }


}
```

