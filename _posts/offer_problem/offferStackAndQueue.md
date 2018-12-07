---
title: 剑指offer总结（栈与队列）
date: 2018-07-17 13:45:13
tags:
- 剑指offer
- java
- python
- 栈和队列
categories: Offer Problems
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/offer.jpg
---

# <center>剑指offer总结——栈与队列</center>

### 面试题9：用两个栈实现队列 
#### 题目描述
用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。 
#### 解决方案
栈的属性是先进后出的，而队列是先进先出，使用两个栈，队列push操作直接将数据push到第一个栈中；pop队列操作时，需要将stack1中的数据push到stack2中，变换stack中存储数据的首位方向，然后stack2弹出栈顶操作就相当于弹出stack1的栈底。push时间复杂度为$O()1$，pop时间复杂度为$O(n)$。

- java解法
```java
public class Solution {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();
    
    public void push(int node) {
        stack1.push();
    }
    
    public int pop() {
        if (stack2.isEmpty){
            while (!stack1.isEmpty()){
                stack2.push(stack1.pop);
            }
        }
        return stack2.pop();
    }
}
```

### 面试题30：包含min函数的栈 
#### 题目描述
定义栈的数据结构，请在该类型中实现一个能够得到栈中所含最小元素的min函数。在该栈中，使用min，push，及pop的时间复杂度都是$O(1)$。
#### 解决方案
使用一个栈存储数据，另一个辅助栈存储最小值数据。当push的时候，将push数据与当前辅助站的top值比较，如果该值比top值小，就将数据push到辅助站中，若该值比top值大，就将top值入栈，这样就记录了每一次入栈操作的最小值。pop时将两个栈的数据都pop即可。

- java解法

```java
import java.util.Stack;
import java.util.*;
import java.lang.Exception;

public class Solution {  
    public Stack<Integer> dataStack = new Stack<>();
    public Stack<Integer> minStack = new Stack<>();
    public void push(int node) {
        dataStack.push(node);
        if (minStack != null){
            if (minStack.top() < node) minStack.push(node);
            else minStack.push(minStack.top());
        }
        else{
            minStack.push(node);
        }
    }
    
    public void pop() throws NullPointerException {
        if ((!data.isEmpty()) && (!minData.isEmpty())){
            data.pop();
            minData.pop();
        }
        else 
            throw new NullPointerException();
    }
    
    public int top() {
        if (dataStack.isEmpty) return Integer.MIN_VALUE;
        return dataStack.peek();
    }
    
    public int min() {
        if (minStack.isEmpty) return Integer.MIN_VALUE;
        return minStack.peek();
    }
}
```

### 面试题31：栈的压入、弹出序列 
#### 题目描述
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）。
#### 解决方案
使用一个辅助栈存储当前栈中的值，第二个序列的第一个数字为栈顶，因此，此时的栈应该为从序列一中入栈，直到找到popA[0]=pushA[i]，然后循环到下一个pop值
当popA[j]为栈顶元素时，就将该元素出栈；如果不是，就把pushA的下一个元素入栈，直到把下一个需要弹出的数字入栈为止。
如果所有的元素均已入栈仍然没有找到下一个应该弹出的数字，则返回false。时间复杂度为$O(n)$，空间复杂度也为$O(n)$。

- java解法
```java
public boolean IsPopOrder(int [] pushA, int [] popA) {
    if (pushA == null || popA == null || pushA.length != popA.length)
        return false;
    Stack<Integer> stack = new Stack<>();
    int i = 0, j = 0;
    while (i < pushA.length || j < popA.length){
        while (i < pushA.length && (stack.isEmpty() || popA[j] != stack.peek())){
            stack.push(pushA[i]);
            i++;
        }
        if (j >= popA.length || popA[j] != stack.peek()) return false;
        stack.pop();
        j++;
    }
    if (stack.isEmpty() && i == j) return true;
    return false;
}
```
### 面试题32：从上到下打印二叉树 
#### 题目描述
从上往下打印出二叉树的每个节点，同层节点从左至右打印。
#### 解决方案
用一个队列存储当前层的节点，弹出队列顶点节点，判断当前节点是否有左右子节点，有就入队，并将当前节点的值打印出来。再弹出下一个队列顶点节点，直至队列为空。时间复杂度为$O(n)$。

这道题是二叉树的内容，并且涉及到使用队列。

- java解法
```java
public ArrayList<Integer> printFromTopToBottom(TreeNode root){
    ArrayList<Integer> ans = new ArrayList<>();
    if (root == null) return ans;
    Queue<TreeNode> queue = new Queue<>();
    queue.offer(root);
    while(!queue.isEmpty()){
        TreeNode temp = queue.poll();
        if (temp.left != null) queue.offer(temp.left);
        if (temp.right != null) queue.offer(temp.right);
        ans.add(temp);
    }
    return ans;
}
```
### 题目变形一：分行从上到下打印二叉树
#### 题目描述
从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。
#### 解决方案
此题和上一题类似，需要使用队列来保存将要打印的节点，为了把二叉树的每一行单独打印到一行，在层次遍历的过程中，需要使用一个变量记录当前层的节点数。

- java解法
```java
ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
    ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
    if (pRoot == null) return ans;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(pRoot);
    while (!queue.isEmpty()){
        ArrayList<Integer> list = new ArrayList<>();
        int left = 0, right = queue.size();
        while (left < right){
            TreeNode temp = queue.poll();
            list.add(temp.val);
            if (temp.left != null) queue.offer(temp.left);
            if (temp.right != null) queue.offer(temp.right);
            left++;
        }
        ans.add(list);     
    }
    return ans;
}
```
### 题目变形二：分行从上到下打印二叉树
#### 题目描述
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。
#### 解决方案
此题和上一题变形题类似，使用之字形打印时，需要两个栈来维护之字形的结构，奇数层存入第一个栈，偶数层存入第二个栈，当遍历完当前层的所有节点时，交换这两个栈，继续遍历下一层。

LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
可以仅使用一个LinkedList，类似于双向链表的数据结构存储节点数据，因为它既可以当做队列，实现从左到右打印，也可以当做栈，实现从右到左打印。因此可以在奇数层时作为队列，偶数层时作为栈。
这里采用第一种方法。

- java解法
```java
public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
    ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
    if (pRoot == null) return ans;
    
    Stack<TreeNode> stack1 = new Stack<>();
    Stack<TreeNode> stack2 = new Stack<>();
    stack1.add(pRoot);
    boolean leftToRight = true;
    while(!stack1.isEmpty() || !stack2.isEmpty()){
        ArrayList<Integer> layer = new ArrayList<>();
        TreeNode temp = null;
        if (leftToRight){
            while(!stack1.isEmpty()){
                temp = stack1.pop();
                if (temp  != null){
                    layer.add(temp.val);
                    stack2.push(temp.left);
                    stack2.push(temp.right);
                }
            }
        }else{
            while(!stack2.isEmpty()){
                temp = stack2.pop();
                if (temp != null){
                    layer.add(temp.val);
                    stack1.push(temp.right);
                    stack1.push(temp.left);
                }
            }
        }
        if (layer.length != 0){
            ans.add(layer);
            leftToRight = (!leftToRight);
        }
    }
    return ans;
}
```






