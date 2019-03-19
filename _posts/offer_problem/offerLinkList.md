---
title: 剑指offer刷题总结（链表）
date: 2018-07-15 10:00:59
tags:
- 剑指offer
- java
- python
- 链表
categories: Offer Problems
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/offer.jpg
---

# <center>剑指offer总结——链表</center>

### 面试题6：从尾到头打印链表  
#### 题目描述
输入一个链表，按链表值从尾到头的顺序返回一个ArrayList。 
#### 解决方案
解法一：使用栈的形式，将链表从头到尾入栈，最后依次出栈，时间复杂度及空间复杂度均为$O(n)$
解法二：递归

- java解法一：

```java
  public ArrayList<Integer> printListFromTailToHead(ListNode listNode){
      ArrayList<Integer> ans = new ArrayList<>();
      if (listNode == null) return ans;
  
      Stack<Integer> stack = new Stack<>();
      while (listNode != null){
          stack.push(listNode.val);
          listNode = listNode.next;
      }
      while (!stack.isEmpty()){
          ans.add(stack.pop());
      }
      return ans;
  
  }
```

- java解法二：

```java
  public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
      ArrayList<Integer> ans = new ArrayList<Integer>();
      if(listNode == null) return ans;
      ListNode pHead = listNode;
      if(pHead != null){
          ans = printList(pHead, ans);
      }
      return ans;
  }
  public ArrayList<Integer> printList(ListNode pHead, ArrayList<Integer> ans){
      if(pHead.next != null){
          ans = printList(pHead.next, ans);
      }
      ans.add(pHead.val);
      return ans;
  }
```

- python解法：使用python insert函数

```python
  def printListFromTailToHead(self, listNode):
  
      if not listNode:
          return []
      arr = []
      head = listNode
      while head:
          arr.insert(0, head.val)
          head = head.next
      return arr
```
### 面试题18：删除链表的节点  
#### 题目描述
给定一个单向链表的头指针的一个节点指针，实现一个函数在$O(1)$时间删除该节点
#### 解决方案
常用的做法是遍历链表，找到目标节点，其时间复杂度为$O(n)$，不满足要求。要删除该节点，不一定非要找到目标节点的上一节点，只需要找到目标节点的下一节点，将其赋值给目标节点就可以了。

- 链表有多个节点，要删除的不是尾节点需要$O(1)$的时间；
- 链表只有一个结点，删除头结点（也是尾结点）需要$O(1)$的时间；
- 链表有多个节点，要删除的是尾节点需要$O(n)$的时间

并且在删除操作前，需要判断target节点是否确实在链表中，这样的查找需要$O(n)$的时间，但此处因为时间复杂度的要求，不加以判断。

- java解法

```java
  public void deleteNode(ListNode pHead, ListNode target){
      if(root == null || target == null) return;
      if(target.next != null){
          ListNode temp = target.next;
          target.val = temp.val;
          target.next = temp.next;
          temp = null;
      }
      else if(pHead == target){
          pHead = null;
          target = null;
      }
      else{
          ListNode head = pHead;
          while(head.next != target)
              head = head.next;
          head.next = null;
          target = null;
      }
  }
```

### 链表中倒数第k个节点 
#### 题目描述
输入一个链表，输出该链表中倒数第k个结点。 
#### 解决方案
类似于滑动窗的做法，使用两个指针，让第一个指针先走k-1步，再让第一第二个指针同时走，当第一个指针走到链表尾部的时候，第二个指针正好指向倒数第K个结点。其时间复杂度为$O(n)$,空间复杂度为$O(1)$

- java解法
```java
  public ListNode FindKthToTail(ListNode head,int k) {
      ListNode first = new ListNode(0);
      if(head == null || k <= 0){
          return first.next;
      }
      first = head;
      ListNode second = head;
      while(--k > 0){
          first = first.next;
      }
      if (first == null){
          return first;
      }
      while(first.next != null){
          first = first.next;
          second = second.next;
      }
      return second;
  }
```
### 面试题23：链表中环的入口节点 
#### 题目描述
给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。
#### 解决方案
使用一个快指针，步长为2，慢指针，步长为1。如果链表中有环，两个指针在更新的过程中都不可能为null，当快慢指针第一次相遇时，根据其更新次数是相等的，然后快指针的路程是慢指针的2倍的关系，列出等式，然后让快指针从头结点开始走，慢指针从当前位置以相同的步长开始走。当他们相遇时，相遇的节点就是环的入口节点。时间复杂度为$O(n)$，空间复杂度为$O(1)$

- java解法
```java
  public ListNode EntryNodeOfLoop(ListNode pHead){
      if(pHead == null || pHead.next == null || pHead.next.next == null) return null;
      ListNode slow = pHead.next, fast = slow.next;
      while(fast != slow){
          slow = slow.next;
          fast = fast.next.next;
          if(fast == null || slow == null){
              return null;
          }
      }
      fast = pHead;
      while(fast != slow){
          fast = fast.next;
          slow = slow.next;
      }
      return fast;
}
```

### 面试题24：反转链表 

#### 题目描述

输入一个链表，反转链表后，输出新链表的表头。 

#### 解决方案

使用两个指针更新链表，在循环内部使用一个temp Node保存当前结点的next结点，然后将head节点的next指向pre节点，此时完成了head节点的断链以及指向新链。然后将pre指向当前节点head， head指向其下一个节点temp。时间复杂度为$O(n)$，空间复杂度为$O(1)$

- java解法

```java
  public ListNode ReverseList(ListNode head) {
      if(head == null) return head;
      ListNode pre = null;
      while(head != null){
          ListNode temp = head.next;
          head.next = pre;
          pre = head;
          head = temp;
      }
      return pre;
  }
```

### 面试题25：合并两个排序的链表 

#### 题目描述

输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。 

#### 解决方案

创建一个头节点，使用while循环，遍历两个list，进行判断：

当头结点的下一个节点为list1的节点时，需要满足的条件为list2此时为null或（list1不为null且list1的小于list2的值）充分利用逻辑语句的短路性质，list2的判断方法相同。时间复杂度为$O(n)$,空间复杂度为$O(1)$

- java解法

```java
  public ListNode Merge(ListNode list1,ListNode list2) {
      if(list1 == null) return list2;
      if(list2 == null) return list1;
  
      ListNode head = new ListNode(0);
      ListNode ans = head;
  
      while(list1 != null || list2 != null){
          if(list2 == null || (list1 != null && list1.val < list2.val)){
              ans.next = list1;
              list1 = list1.next;
          }else if(list1 == null || (list2 != null && list1.val >= list2.val)){
              ans.next = list2;
              list2 = list2.next;
          }
          ans = ans.next;
      }
      return head.next;
  }
```

### 面试题35：复杂链表的复制 

#### 题目描述

输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）

```java
public class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
```

复杂链表示例：

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/772795.jpg" width=50%/>
</center>

#### 解决方案
有一个巧妙的方法，使用链表结构本身来记录其sbiling指针的位置，分成三个步骤 

- 根据原始链表的每个结点N创建对应的N’，并把N’连在N的后面。  如下图： 

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/76851960.jpg" width=75%/>
</center>

- 根据原来的记录，N节点的random指向S节点，因此将复制出来的节点N'的random指向其对应的S'节点，如图所示

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/6780607.jpg" width=75%/>
</center>

- 将长链表拆分为两个短链表，将奇数位置的节点连接起来就是原链表，偶数位置的链表连接起来就是复制的链表。如图所示

<center>
<img src="https://github.com/niuyuanyuanna/BlogImages/raw/master/offerProblems/92955680.jpg" width=75%/>
</center>

这种方法的总体时间复杂度为$O(n)$,空间复杂度也为$O(n)$

- java解法

```java
  public RandomListNode Clone(RandomListNode pHead){
      if(pHead == null) return pHead;
      RandomListNode p = pHead, q = pHead, g = pHead;
      while(p != null){
          RandomListNode copy = new RandomListNode(p.label);
          copy.next = p.next;
          p.next = copy;
          p = copy.next;
      }
      while(q != null){
          if(q.random != null){
              q.next.random = q.random.next;
          }
          q = q.next.next;
      }
      RandomListNode ans = new RandomListNode(0);
      RandomListNode f = ans;
  
      while(g != null){                         //注意最后分开两个链表的方式，用局部变量
          RandomListNode temp = g.next;
          g.next = temp.next;
          temp.next = f.next;
          f.next = temp;
          f = f.next;
          g = g.next;
      }
      return ans.next;
  }
```

### 面试题52：两个链表的第一个公共节点 
#### 题目描述
输入两个链表，找出它们的第一个公共结点。
#### 解决方案

首先找到两个链表的长度，其长度的差值d+1就是链表公共节点的个数，然后然长的链表先走d步，然后短链表和长链表一起走，直到两个链表相遇，第一次相遇的节点就是他们相等的节点。时间复杂度$O(n)$,空间复杂度$O(1)$

- java解法

```java
  public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
      if(pHead1 == null || pHead2 == null) return null;
      int count1 = 0, count2 = 0;
      ListNode p = pHead1, q = pHead2, p1 = p, q1 = q;
      while(p != null || q != null){
          if(p != null){
              count1++;
              p = p.next;
          }
          if(q != null){
              count2++;
              q = q.next;
          }
      }
      if(count1 > count2) return findFirstNode(p1, q1, count1, count2);
      else return findFirstNode(q1, p1, count2, count1);
  }
  
  public ListNode findFirstNode(ListNode p, ListNode q, int count1, int count2){
      int diff = count1 - count2;
      while(--diff >= 0){
          p = p.next;
      }
      while(p != null){
          if(p == q) return p;
          p = p.next;
          q = q.next;
      }
      return p;
  }
```

### 面试题62：圆圈中最后剩下的数字

#### 题目描述

击鼓传花，所有人围成一个圈，编号为0~n-1，共n人，指定一个数字m，每次循环喊数从0~m-1，喊到m-1的淘汰，直到剩下最后一个，返回这个人的编号。

#### 解决方案

这是Josephuse环问题，可以用经典的解法：使用环形链表模拟圆圈，创建有m个节点的环形链表，然后每次在链表中删除第n个节点，但总体的时间复杂度为$O(m*n)$,空间复杂度为$O(n)$

一种创新性解法是使用根据题目本身的数学关系，用数学归纳法，建立公式求解，最终实现时间复杂度为$O(n)$，空间复杂度为$O(1)$

问题描述：n个人（编号0~(n-1))，从0开始报数，报到(m-1)的退出，剩下的人 继续从0开始报数。求胜利者的编号。
我们知道第一个人(编号一定是m%n-1)出列之后，剩下的n-1个人组成了一个新的约瑟夫环（以编号为k=m%n的人开始）:
$$
k, k+1,k+2, \ldots ,n-2,n-1,\quad 0,1,2, \ldots, k-2
$$
并且从k开始报0，现在我们把他们的编号做一下转换：

$$
\begin{aligned}
&k     --> 0 \\
&k+1   --> 1 \\
&k+2   --> 2 \\
& \cdots  \\
& \cdots  \\
&k-2   --> n-2 \\
&k-1   --> n-1 
\end{aligned}
$$

变换后成为了(n-1)个人报数的子问题，假如知道这个子问题的解：如$x$是最终的胜利者，根据上面的表把$x$变回去刚好就是$n$个人情况的解。变回去的公式为：
$$
x'=(x+k)%n
$$
令$f[i]$表示$i$个人玩游戏报$m$退出最后胜利者的编号，最后的结果是$f[n]$。

递推公式
$$
f[1]=0;\\
\vdots \\
f[i] = f([i-1] + m) \%i;\quad(i>1)
$$
有了这个公式，我们要做的就是从1-n顺序算出f[i]的数值，最后结果是f[n]。 因为实际生活中编号总是从1开始，我们输出f[n]+1。

- java解法

```java
  int LastRemaining_Solution(unsigned int n, unsigned int m)
  {
      if(n==0)
          return -1;
      if(n==1)
          return 0;
      else
          return (LastRemaining_Solution(n-1,m)+m)%n;
  }
```

  