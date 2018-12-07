---
title: 剑指offer刷题总结（树）
date: 2018-07-16 09:43:59
tags:
- 剑指offer
- java
- python
- 二叉树
categories: Offer Problems
top_img: https://github.com/niuyuanyuanna/BlogImages/raw/master/background/offer.jpg
---

# <center>剑指offer总结——树</center>

### 面试题7：重建二叉树 

#### 题目描述

输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

#### 解决方案

根据二叉树先序遍历的特征，找到根节点为先序遍历的第一个节点，在中序遍历的数组中找到该节点，则在中序遍历中，该节点的左半部分为左子树的节点，又半部分为右子树的节点，分而治之，以递归的形式，分别找到根节点的左子树和右子树。算法时间复杂度为$O(nloogn)$，空间复杂度为$O(n)$，因为每个节点都会被递归调用。

- java解法
```java
public TreeNode reConstructBinaryTree(int [] pre,int [] in) {
    if (pre == null || in == null || pre.length != in.length) return null;
    int preL = 0, preR = pre.length - 1, inL = 0, inR = in.length - 1;
    return findNode(pre, preL, preR, in, inL, inR);
}

public TreeNode findNode(int[] pre, int preL, int preR, int[] in, int inL, int inR){
    if (preL > preR || inL >inR) return null;
    TreeNode root = new TreeNode(pre[preL]);
    for (int i = inL; i <= inR; i++){
        if (pre[preL] == in[i]){
            root.left = findNode(pre, preL + 1, preL + i - inL, in, inL, i - 1);
            root.right = findNode(pre, preL + i - inL + 1, preR, in, i+1, inR);
        }
    }
    return root;
}
```

### 面试题8：二叉树的下一个节点 
#### 题目描述
给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，同时包含指向父结点的指针。
#### 解决方案
二叉树的下一个节点共有三种情况

- 该节点有右子节点，此时中序遍历的下一个节点就是该节点右子树沿左分支向下，知直到找到最后一个左孩子，这个左孩子就是其下一个节点
- 该节点没有有子节点，此时沿父节点向上查找，直到查找到一个父节点，父节点的左孩子是该节点的分支
- 该节点为null，返回null
平均时间复杂度为$O(h)$，h为数的深度。

- java解法
```java
public TreeLinkNode GetNext(TreeLinkNode pNode){
    TreeLinkNode ans = new TreeLinkNode(0);
    if(pNode == null) return null;
    if(pNode.right != null){
        ans = pNode.right;
        while(ans.left != null){
            ans = ans.left;
        }
        return ans;
    }
    while(pNode.next != null){
        ans = pNode.next;
        if(ans.left == pNode) return ans;
        pNode = ans;
    }
    return null;
}
```
### 面试题26：树的子结构 
#### 题目描述
输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）。
#### 解决方案
使用递归的方法判断B是否是A的子树，在当前节点的判断需要分为三种情况，首先是A和B是否相同，然后判断A的左子树是否和B相同，A的右子树是否和B相同。时间复杂度为$O(n)$，n为A的节点数。

- java解法
需要充分利用判断条件的短路性质
```java
public boolean HasSubtree(TreeNode root1,TreeNode root2) {
    if (root2 == null || root1 == null) return false;
    return isSubtree(root1, root2) || HasSubtree(root1.left, root2) || HasSubtree(root1.right, root2);
}
private boolean isSubtree(TreeNode root1, TreeNode root2){
    if (root2 == null) return true;
    if (root1 == null) return false;
    if (root1.val == root2.val) 
        return isSubtree(root1.left, root2.left) && isSubtree(root1.right, root2.right);
    return false;
}
```

### 面试题27：二叉树的镜像 
#### 题目描述
操作给定的二叉树，将其变换为源二叉树的镜像。
例如：
    	     8
    	    /  \
    	  6    10
    	 / \    / \
    	5  7   9  11
镜像二叉树为：
    	     8
    	    /  \
    	  10    6
    	 / \    / \
    	11  9   7  5


#### 解决方案
使用递归的方法，类似于二叉树的层次遍历，依次翻转二叉树的左右子节点。时间复杂度为$O(n)$，空间复杂度为$O(n)$

- java解法
```java
public void mirror(TreeNode root){
    if (root == null) return;
    if (root.left != null || root.right != null){
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;
        mirror(root.left);
        mirror(root.right);
    }
}
```
### 面试题28：对称的二叉树 
#### 题目描述
请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。
#### 解决方案
使用递归的方法，分别判断当前节点的左孩子是否等于对比节点的右孩子值，当前节点的右孩子是否等于对比节点的左孩子值。时间复杂度和空间复杂度均为$O(n)$。

- java解法
```java
public boolean isSymmetrical(TreeNode pRoot){
    if (pRoot == null) return true;
    return isSym(pRoot.left, pRoot.right);
}
public boolean isSym(TreeNode pNode, TreeNode qNode){
    if (pNode == null) return qNode == null;
    if (qNode == null || pNode.val != qNode.val) return false;
    return (isSym(pNode.left, qNode.right) && isSym(pNode.right, qNode.left));
}
```

### 面试题32：从上到下打印二叉树 
#### 题目描述
从上往下打印出二叉树的每个节点，同层节点从左至右打印。
#### 解决方案
用一个队列存储当前层的节点，弹出队列顶点节点，判断当前节点是否有左右子节点，有就入队，并将当前节点的值打印出来。再弹出下一个队列顶点节点，直至队列为空。时间复杂度为$O(n)$。

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

### 面试题33：二叉搜索树的后序遍历序列 
#### 题目描述
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同
#### 解决方案
采用分而治之的思想，二叉树的后序遍历是将根节点置于数组最后，BST将剩余数组分为两部分，前面是左子树，其数值均比根节点的数值小，后面部分为右子树，其数值均比根节点的数值大，如果后面部分有比根节点数值小的数字，则不是一个BST的后序遍历。递归判断左右子序列是否是后序遍历。时间复杂度为$O(logn)$。

- java解法
```java
public boolean VerifySquenceOfBST(int [] sequence) {
    if (sequence.length == 0) return false;
    return (verifySequence(sequence, 0, sequence.length - 1));
}
public boolean verifySequence(int[] sequence, int l, int r){
    if (l >= r) return true;
    int separater = l;
    while (separater < r){
        if (sequence[separater] > sequence[r]){
            break;
        }
        ++separater;
    }
    for (int i = separater; i < r; i++){
        if (sequence[i] < sequence[r]) return false;
    }
    return verifySequence(sequence, l, separater-1) && verifySequence(sequence, separater, r-1);
    
}
```
### 面试题34：二叉树中和为某一值的路径 
#### 题目描述
输入一颗二叉树的根节点和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。(注意: 在返回值的list中，数组长度大的数组靠前)
#### 解决方案
因为需要从根节点出发到达叶节点，因此采用先序遍历的思想。访问到某一节点时，将该节点添加到路径上，累加该节点的值，如果路径中的节点值之和正好为输入整数，且该节点为叶节点，则当前路径符合要求。当前节点访问完成后，递归回到其父节点。时间复杂度为$O(n)$。

- java解法
```java
public class Solution{
    private ArrayList<ArrayList<Integer>> ans = new ArrayList<>();
    private ArrayList<Integer> list = new ArrayList<>();
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target){
        if (root == null) return ans;
        list.add(root.val);
        target -= root.val;
        if (target == 0 && root.left == null && root.right == null)
            ans.add(new ArrayList<Integer>(list));
        FindPath(root.left, target);
        FindPath(root.right, target);
        list.remove(list.size() - 1);
        return ans;
    }
}
```



### 面试题36：二叉搜索树与双向链表 

#### 题目描述
输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。 

#### 解决方案

转换为排序链表则考虑使用中序遍历，将二叉树变为排序链表。例如BST为：

    	     10
    	    /  \
    	  6    14
    	 / \    / \
    	4  8   12 16
采用中序遍历的结果应为：
$$
4\leftrightarrow 6\leftrightarrow 8\leftrightarrow 10\leftrightarrow 12\leftrightarrow 14\leftrightarrow 16
$$
为了减少指针的变换次数，并让操作更加简单，在转换成排序双向链表时，原先指向左子结点的指针调整为链表中指向前一个结点的指针，原先指向右子结点的指针调整为链表中指向下一个结点的指针。例如对于上面的值为6的节点，调整之后，它的前一个节点\左孩子是4，后一个节点\右孩子是8。
当遍历到根结点时，把树分为三个部分：根结点，根的左子树和根的右子树。如上图的二叉排序树，就分成了根结点10以结点6为根的左子对和以结点14为根的右子树。从变换的链表中可以看到，应当把结点10
的left指针指向结点8，把结点8的right指针指向结点10，由于采用中序遍历，当遍历到结点10时，结点10的左子树已经转化为一个有序的双向链表，而结点8是这个已经转化的双向链表的尾结点，所以应该用一个变量last_node来保存最后一个结点的指针，以便在与根结点连续时使用。然后把这个变量last_node的值更新为指向根结点10。对于结点10的右子树，采取相似的操作。至于具体的实现，只需要对所有的子树递归地执行上述操作即可。

- java解法
```java
public class Solution {
    public TreeNode lastNode = null;
    public TreeNode realHead = null;

    public TreeNode Convert(TreeNode pRootOfTree) {
        if (pRootOfTree == null) return pRootOfTree;
        convertNode(pRootOfTree);
        return realHead;
    }
     
    private void convertNode(TreeNode pNode) {
        if(pNode == null) return;
        
        if (pRootOfTree.left != null){
            convertNode(pNode.left);
        }
        
        if (lastNode == null) {
            lastNode = pNode;
            realHead = pNode;
        } else {
            lastNode.right = pNode;
            pNode.left = lastNode;
            lastNode = pNode;
        }
        if (pNode.right != null){
            convertNode(pNode.right);
        }
    }
}
```

### 面试题37：序列化二叉树  
#### 题目描述
请实现两个函数，分别用来序列化和反序列化二叉树 。

#### 解决方案

二叉树的序列化从根节点开始，相应的反序列化在根节点的数值读出来的时候就可以开始，因此可以根据前序遍历的顺序来序列化二叉树，当遇到空指针的时候就使用"$"来代替该处的value，用“,”把不同的节点数值分隔开。

反序列化的时候每次从流中读取一个字符，以前序遍历的顺序重建二叉树。

- java解法

```java
import java.util.*;
public class Solution {
    public int index = -1;
	public String Serialize(TreeNode root){
        if (root == null) return "$,";
        StringBuffer ans = new StringBuffer();
        ans.append(root.val).append(",");
        ans.append(Serialize(root.left));
        ans.append(Serialize(root.right));
        return ans.toString();
    }

    public TreeNode Deserialize(String str){
        if (str.equals("")) return null;
        String[] strList = str.split(",");       
        return deserialize(strList);      
    }

    public TreeNode deserialize(String[] strList){
        ++index;
        if (index >= strList.length) return null;
        TreeNode root = null;
        if (!strList[index].equals("$")){
            root = new TreeNode(Integer.parseInt(strList[index]));
            root.left = deserialize(strList);
            root.right = deserialize(strList);
        }
        return root;
    }
}
```


### 面试题40：最小的k个数  
#### 题目描述
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。 

#### 解决方案

如果将数组排序后取前面k个数可以解决问题，但时间复杂度为$O(nlogn)$。

- 方法一：时间复杂度$O(n)$

  基于Partition函数来实现，如果基于数组中的第k个数字来调整，使比第k个数小的所有数字位于数组左半部分，比第k个数大的所有数字位于数组右半部分。调整后可以直接找到这k个数字，但这k个数字不一定是排序的。且这种方法会更改原始数组的排序。

```java
public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
    ArrayList<Integer> ans = new ArrayList<>();
    if (input.length == 0 || k <= 0 || input.length < k) return ans;
    
    int start = 0, end = input.length - 1;
    int index = Partition(input, start, end);
    while(index != k - 1){
        if(index > k - 1)
            index = Partition(input, start, index - 1);
        else
            index = Partition(input, index + 1, end);
    }
    for (int i = 0; i < k; i++){
        ans.add(input[i]);
    } 
    return ans;
}

public int Partition(int[] input, int start, int end){
    if (start >= end) return start;
    int index = (int)(Math.random()*(end - start + 1)) + start;
    swap(input, index, end);
    int small = start - 1;
    for (index = start; index < end; index++){
        if (input[index] < input[end]){
            small++;
            if(input[small] != input[index])
                swap(input, small, index);      
        }
    }
    small++;
    swap(input, small, end);
    return small;
} 

public void swap(int[] input, int i, int j){
    int temp = input[i];
    input[i] = input[j];
    input[j] = temp;
}
```

- 方法二：时间复杂度$O(klogn)$

  适合处理海量数据的算法。
  先创建一个大小为k的数据容器来存储最小的k个数字，接下来每次从输入的n个整数中读入一个数。如果容器中已有的数字少于k个，则直接把这次读入的整数放入容器之中；如果容器中己有k 数字了，也就是容器己满，此时我们不能再插入新的数字而只能替换已有的数字。找出这己有的k 个数中的最大值，然后1在这次待插入的整数和最大值进行比较。如果待插入的值比当前己有的最大值小，则用这个数替换当前已有的最大值：如果待插入的值比当前已有的最大值还要大，那么这个数不可能是最小的k个整数之一，于是我们可以抛弃这个整数。

  因此当容器满了之后，要做3 件事情： 

  - 在k 个整数中找到最大数
  - 有可能在这个容器中删除最大数
  - 有可能要插入一个新的数字

  使用一个二叉树作为容器，在O(logk）时间内实现这三步操作。
  由于每次都需要找到k个整数中的最大值，因此考虑用最大堆。在最大堆中，根节点的值总是大于子树中的值。可以在$O(1)$的时间内查找到最大值，但需要$O(logk)$时间完成删除和插入。这种解法没有修改原始输入。

```java

```

### 面试题54：二叉搜索树的第K个节点
#### 题目描述
给定一颗二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）    中，按结点数值大小顺序第三小结点的值为4。
#### 解决方案
BST中序遍历找到第K个节点就是返回值。时间复杂度为$O(k)$。

- java解法
```java
public class Solution {
    int index = 0;
    TreeNode KthNode(TreeNode pRoot, int k){
        if(pRoot == null || k <= 0){
            return null;
        }
        TreeNode node = KthNode(pRoot.left, k);
        if(node != null) return node;
        index++;
        if(k == index) return pRoot;
        node = KthNode(pRoot.right, k);
        if(node != null) return node;
        return null;
    }
}
```

### 面试题55：二叉树的深度  
#### 题目描述
输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。
#### 解决方案
递归调用，判断左子树的深度和右子树的深度，返回深度时+1

- java解法
```java
public int TreeDepth(TreeNode root) {
    if (root == null) return 0;
    int l = TreeDepth(root.left), r = TreeDepth(root.right);
    return 1+ (l>r?l:r);
}
```
###  面试题55：二叉树的深度——平衡二叉树
#### 题目描述
输入一棵二叉树，判断该二叉树是否是平衡二叉树。
#### 解决方案
后序遍历的方式遍历整棵二叉树，遍历某节点的左右子节点之后，根据左右子节点的深度判断其是否为平衡二叉树，且得到当前节点的深度。遍历到最后节点的时候就可以判断其是否为平衡二叉树。这种方法每个节点只遍历一次。

- java解法
```java
public boolean IsBalanced_Solution(TreeNode root){
    if (root == null || isBalanced(root) != -1) return true;
    else return false;
}
public int depthOfBST(TreeNode root){
    if (root == null) return 0;
    int leftDepth = depthOfBST(root.left);
    if (leftDepth == -1) return false;
    int rightDepth = depthOfBST(root.right);
    if (rightDepth == -1) return false;
    if (Math.abs(leftDepth - rightDepth) <= 1)
        return 1 + (leftDepth > rightDepth ? leftDepth : rightDepth);
    return -1;
}
```

### 面试题68：树中两个节点的最低公共祖先 
#### 题目描述
输入两个树节点，求他们的最低公共祖先。
#### 解决方案


给定二叉搜索树时，因为二叉搜索树已近排序，位于左子树的节点都比右子树的节点小。从树的根节点开始，和两个输入的节点进行比较。如果当前节点的值比两个树节点的值都大，则目标节点一定是位于当前节点的左子树中，若小就位于右子树。从上到下遍历，找到的第一个在两个输入节点的值之间的节点就是最低的公共祖先。

- java解法
```java
public TreeNode findFirstCommon(TreeNode pNode, TreeNode qNode，TreeNode root){
    
}
```

假设给定的树为普通树，且子树没有指向父节点的指针，那么应该记录从根节点到这两个节点的路径，然后转换成找到链表中的最后一个公共节点问题。

- java解法

```java
/*
 * 获取两个节点的最低公共祖先
 */
public TreeNode getLastCommonParent(TreeNode root, TreeNode p1, TreeNode p2) {
    //path1和path2分别存储根节点到p1和p2的路径（不包括p1和p2）
    List<TreeNode> path1 = new ArrayList<TreeNode>();
    List<TreeNode> path2 = new ArrayList<TreeNode>();
    List<TreeNode> tmpList = new ArrayList<TreeNode>();

    getNodePath(root, p1, tmpList, path1);
    getNodePath(root, p2, tmpList, path2);
    //如果路径不存在，返回空
    if (path1.size() == 0 || path2.size() == 0)
        return null;

    return getLastCommonParent(path1, path2);
}

// 获取根节点到目标节点的路径
public void getNodePath(TreeNode root, TreeNode target, List<TreeNode> tmpList, List<TreeNode> path) {
    //鲁棒性
    if (root == null || root == target)
        return;
    tmpList.add(root);
    List<TreeNode> children = root.children;
    for (TreeNode node : children) {
        if (node == target) {
            path.addAll(tmpList);
            break;
        }
        getNodePath(node, target, tmpList, path);
    }

    tmpList.remove(tmpList.size() - 1);
}
//将问题转化为求链表最后一个共同节点
private TreeNode getLastCommonParent(List<TreeNode> p1, List<TreeNode> p2) {
    TreeNode tmpNode = null;
    for (int i = 0; i < p1.size(); i++) {
        if (p1.get(i) != p2.get(i))
            break;
        tmpNode = p1.get(i);
    }

    return tmpNode;
}
```



