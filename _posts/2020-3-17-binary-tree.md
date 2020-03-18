---
layout: post
title: 二叉树刷题总结
tags: java  Algorithm
---


> 总结记录二叉树刷题报告，常见思路、题目汇总

### 一、二叉树基础知识

二叉树即为只有两个节点的存储结构    

**满二叉树**：所有叶子节点都在最后一层，且总的结点数为2^n - 1;    

**完全二叉树**：所有的叶子节点都在最后一层或者是倒数第二层，且最后一层叶子节点在左边连续，倒数第二层在右边连续。   

**二叉搜索树、二叉排序树**：即左结点一定小于右结点，并且左子树一定都小于右子树的值。官方定义：它或者是一棵空树，或者是具有下列性质的[二叉树](https://baike.baidu.com/item/%E4%BA%8C%E5%8F%89%E6%A0%91/1602879)： 若它的左子树不空，则左子树上所有结点的值均小于它的根结点的值； 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值； 它的左、右子树也分别为[二叉排序树](https://baike.baidu.com/item/%E4%BA%8C%E5%8F%89%E6%8E%92%E5%BA%8F%E6%A0%91/10905079)。  

**二叉树的深度**： 深度即为从根节点遍历到叶子节点的最长路径长度。  

二叉树数据结构类型:    

```java
 static class TreeNode{
        int val;
        TreeNode left = null;
        TreeNode right = null;
        TreeNode(int val){
            this.val = val;
        }
    }
```

### 二、二叉树的遍历：

- 先序遍历 ：根左右 （也可以称之为深度优先遍历）
- 中序遍历 ： 左根右
- 后续遍历 ：左右根
- 层遍历：按层进行遍历

先序、中序、后续的递归遍历代码：    

```java
//先序遍历
public void preOrder(Tree root){
	visited(root);          //根
    preOrder(root->left);   //左
    preOrder(root->left);   //右
}

//中序遍历
public void midOrder(Tree root){
    midOrder(root->left);   //左
    visited(root);          //中
    midOrder(root->left);   //右
}

//后序遍历
public void lastOrder(Tree root){
    lastOrder(root->left);   //左
    lastOrder(root->left);   //右
    visited(root);           //根
}
```

先序、中序、后续、层序遍历的非递归代码：  

```java
 //使用栈进行前序遍历  也就是深度优先遍历。
    public void preTraverse(TreeNode root){

        Stack<TreeNode> s = new Stack<>();
        if (root !=null) s.push(root);
        TreeNode cur;
        while (!s.isEmpty()){
            cur = s.pop();//取出来值。
            System.out.println(cur.val);

            if (cur.right!=null){ //先压右栈？？ 然后就能用了我艹  仔细想想是需要先压左栈 保留火种。。
                s.push(cur.right);
            }
            if (cur.left !=null){
                s.push(cur.left);
            }

        }
    }
	
    //中序遍历
    public void midTraverse(TreeNode root){
        Stack<TreeNode> stack = new Stack<>();

        if (root != null) stack.push(root);
        TreeNode cur;
        while (!stack.isEmpty()){

            while (stack.peek().left!=null){//将左栈压进去。
                stack.push(stack.peek().left);
            }

            while (!stack.isEmpty()){
                cur = stack.pop();
                System.out.println(cur.val); //访问节点。  中

                if (cur.right !=null){ //  右 意思就是每次都将节点保存进去。。
                    stack.push(cur.right);
                    break;
                }
            }

        }

    }

    //后续遍历。  写法与中序差不多，就是在输出条件上加了一个if语句。
    public void postTraverse(TreeNode root){
        Stack<TreeNode> stack = new Stack<>();
        if (root ==null){
            return;
        }
        stack.push(root);
        TreeNode lastPopNode = null;
        while (!stack.isEmpty()){
            while (stack.peek().left !=null){  //压入子树
                stack.push(stack.peek().left);
            }

            while (!stack.isEmpty()){

                //如果右子树已经输出，或者是右子树为空，则输出此节点。
                //比中序遍历多了一个判断是否右子树已经输出，或者为空。
                if (lastPopNode == stack.peek().right || stack.peek().right == null){
                    lastPopNode  = stack.pop();
                    System.out.println(lastPopNode.val);
                }else if (stack.peek().right !=null){ //如果右子树 为空则进行进栈
                    stack.push(stack.peek().right);
                    break;
                }
            }
        }

    }

    //广度优先遍历  使用队列实现
    public void BroadTraverse(TreeNode root){
        if (root ==null) return;

        Queue<TreeNode> queue = new LinkedList<>(); //队列

        queue.add(root);
        while (!queue.isEmpty()){
           TreeNode cur  =queue.remove();
            System.out.println(cur.val);
           if (cur.left!=null){
               queue.add(cur.left);
           }
           if (cur.right != null){
               queue.add(cur.right);
           }
        }

    }
```

### 三、剑指Offer题目

#### 1.镜像二叉树

**题目意思**: 生成二叉树的景象树，即将所有的叶子节点对称交换，如下所示；  

![mirror.png](https://pic.tyzhang.top/images/2020/03/17/mirror.png)

**解题思路**:使用递归将其左右交换实现，注意在递归的时候，都是每一层进行交换。知道交换结束为止  

```java
 public void Mirror(TreeNode root) {
        if (root ==null)
            return;
       if (root.right == null && root.left ==null)
            return;
        else{
            TreeNode tmp; //进行交换子节点，
            tmp = root.right;
            root.right = root.left;
            root.left = tmp;
            
            Mirror(root.left); //然后进行接下来的操作
            Mirror(root.right);
        }
    }
```

#### 2. 判断是否为二叉树子结构

**题目大意**:输入两个二叉树AB，判断A是否为B的子结构。约定空树不是任何一个树的子结构。   

**解题思路**：首先要搞清楚概念，拿下图来说，其中大方框框着的大树的子树，最小的方框为大树的子结构。所以说子结构的范围更小。可有有很多，但是子树的话，一颗大树最多有两个子树。  

[![treeStructure.md.jpg](https://pic.tyzhang.top/images/2020/03/17/treeStructure.md.jpg)](https://pic.tyzhang.top/image/H8o)

判断一个树是否是一棵大树的子树:    

**查找子树**  

```java
  public Boolean subTree(TreeNode root1,TreeNode root2){
        if (root1 ==null)
            return  false;
        if (root2 == null)
            return false;
        return  judge(root1,root2) ||judge(root1.right,root2) || judge(root1.left,root2); //三种情况， 二者完全相同。  左根，右根
    }


    public Boolean judge(TreeNode tree,TreeNode subTree){
        if (subTree ==null)  //有两种情况 一种是先子树遍历完，则表示为其子树   另一种是先被检测的树为空，则表示子树还不为空，表示为false。
            return true;
        if (tree == null)
            return false;

        if (tree.val != subTree.val)
            return false;

        return judge(tree.left,subTree.left) && judge(tree.right,subTree.right);
    }
```

从上面的可以推测出寻找子结构的代码，查找子结构就是比查找子树多了一个判断条件。  

**查找子结构**  

```java
public Boolean subTree(TreeNode root1,TreeNode root2){
        if (root1 ==null)
            return  false;
        if (root2 == null)
            return false;
        return  judge(root1,root2) ||judge(root1.right,root2) || judge(root1.left,root2); //三种情况， 二者完全相同。  左根，右根
    }


    public Boolean judge(TreeNode tree,TreeNode subTree){
        if (subTree ==null)  //有两种情况 一种是先子树遍历完，则表示为其子树   另一种是先被检测的树为空，则表示子树还不为空，表示为false。
            return true;
        if (tree == null)
            return false;

        if (tree.val != subTree.val)
            return judge(tree.left,subTree) || judge(tree.right,subTree);  //与子树相比，在不同的时候，还可以在测试之后的数据是否相同。。

        return judge(tree.left,subTree.left) && judge(tree.right,subTree.right);
    }
```

#### 3.判断数组是否为二叉搜索树后续遍历

**题目大意**：输入一个整数数组，判断该数组是否为二叉搜索树的后续遍历结果，假设输入数组的任何两个数组都不相同    

[![binaryTree.jpg](https://pic.tyzhang.top/images/2020/03/17/binaryTree.jpg)](https://pic.tyzhang.top/image/XO1)

**解题思路**：对于二叉搜索树来说，其中中序遍历为一个有序数列。但是对于后续遍历来说，其最后一个值总是为根，上图的二叉树后续遍历结果为：  

| 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 3    | 8    | 9    | 7    | 12   | 19   | 14   | 10   |

即最后的节点，一定是根节点，又因为左子树的值是小于根节点，二右子树的值是大于根节点，所以可以通过一个循环来进行判断是否有不满足的情况，如果不满足则表示为假。    

然后在通过比较大小 在找到新的子树跟的节点，然后进行递归判断。比如左子树的为7，7后面的值为12，则7就是分界点。为左子树根，而原来跟10前一位就是右子树的根，以此类推。。    

```java
  Boolean judge(int[] array,int start, int root){
        if (start >= root){ //已经检测完了 当左边大于右边的时候，就是递归退出，这个条件需要找好。
            return true;
        }
        int key = array[root];

        int i;
        for (i = start; i < root; i++) { //找到分界点
            if (array[i] > key)
                break;
        }

        for (int j = i; j < root; j++) {
            if (array[j] < key) //找到不成立的点。。
                return false;
        }
        return judge(array,start,i-1) && judge(array,i,root-1); //主要索引，因为i是到了不成立才退出的，所以需要减1.
    }

    public boolean VerifySquenceOfBST(int [] sequence) {
        if (sequence.length ==0 || sequence == null) return false;
        return judge(sequence,0,sequence.length-1);
    }
```

#### 4. 打印出所有路径为给定整数的序列。

**题目大意**：输入一个二叉树的根节点和一个整数，打印一个所有路径值整数和的路径，将遍历到路径保存在数组中，路径表示为根节点到叶子节点的所有路径。  

**解题思路**：对于深度来说，采用深度优先遍历便可以找到对应的深度的值，而对于二叉树来说，深度优先遍历便是先序遍历。所以采用先序遍历来遍历二叉树。然后定义一个以为数组和二维数组来进行数据的保存。 为什么没有判断target<0时进行剪枝，因为可能左节点不满足，但是右节点就已经转为满足。但是可以当target 小于0的时候在最开始进行删减。 但是也可以不注意。。 此处体现了 用完在还回去的思想，还回去是因为此节点已经测试完毕，需要回收。

```java
 private ArrayList<ArrayList<Integer>> result = new ArrayList<ArrayList<Integer>>(); //定义二维数组
    private ArrayList<Integer> list = new ArrayList<>();

    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {

        if(root == null)//这个是返回的标志，因为必须要判断结束。
            return result;
        list.add(root.val);
        target -= root.val; //先进行减。  进来就进行减，然后进行判断是否满足条件
        if(target == 0 && root.left == null && root.right == null) //这个是判断成功的标志。只是增加数据，不返回
            result.add(new ArrayList<>(list));//这个增加二维数组的方式要记住，因为list只是一个地址，对象，所以要新建。
        //先序遍历
        FindPath(root.left, target);
        FindPath(root.right, target);
        list.remove(list.size()-1); //这个节点完成使命以后需要将其删除掉。。。
        return result;
    }
```

#### 5.计算两个叶子节点之间的距离

**题目大意**：给定一个二叉树，每个节点都有权值，计算最大权值的叶子节点到最小权值叶子节点的距离，距离定义为 根节点到叶子节点，即最大权值叶子节点到根，加上 最小权值叶子节点到根的距离。  

**解题思路**：刚开始想到的算法就是题目4中的做法，将所有的叶子结点的距离保存下来，数组中尾部的值变为叶子节点的值，找到最小值和最大值，然后数组长度相加则为距离，但是有个关键点就是，对于二叉树来说，当有一个重复边的时候，需要想减。如下图所示：  其中结点7到结点5的聚类为3+2 -1\*1 = 5 减一的愿意是因为，有一条1-2的共同边，而 7和8的距离为3+3-2\*1 =4  因为 有两条重复的边。当没有重复的边是，其距离为数组长度相加，即7和6的距离为，3+2=5。这一点需要处理。

[![dis6801b6c916610c8b.md.png](https://pic.tyzhang.top/images/2020/03/17/dis6801b6c916610c8b.md.png)](https://pic.tyzhang.top/image/v4p)

 

```java
//参考题目4的写法

ArrayList<ArrayList<Integer>> disList = new ArrayList<ArrayList<Integer>>();
    ArrayList<Integer> roadList = new ArrayList<>();
    public int getDis(TreeNode root) {
         if (root ==null)
            return 0;
        ArrayList<ArrayList<Integer>> res = getRoadlist(root);

        int min=Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;

        int maxIndex = 0;
        int minIndex = 0;
        for (int i = 0; i < res.size() ; i++) {
            int value = res.get(i).get(res.get(i).size() - 1);
           if (value < min){
               min =value;
               minIndex = i;

           }
           if (value > max){
               max = value;
               maxIndex = i;
               
           }
        }
        
        //需要找到有几个重复的跟。
        int pos = 0;
        while (res.get(minIndex).get(pos) == res.get(maxIndex).get(pos)){
            pos ++;
        }

        return  res.get(minIndex).size() + res.get(maxIndex).size() - 2 *pos;
    }

//递归实现，将所有的跟节点到叶子节点的路径用二维数组保存下来。和上面的思想是一致的。
    public ArrayList<ArrayList<Integer>> getRoadlist(TreeNode root){
        if (root ==null)
            return disList;
        roadList.add(root.val);//将其添加进去

        if (root.left ==null && root.right ==null){ //找到叶子节点  然后进行数据的保存。
            disList.add(new ArrayList<>(roadList));
        }
        getRoadlist(root.left);
        getRoadlist(root.right);
        roadList.remove(roadList.size() - 1); //删除该节点
        return disList;
    }
```

#### 6.二叉树的深度

**题目描述**：给定一个二叉树，输出二叉树的深度  

**思路描述**：对于深度来说，其实是很简单的， 但是，刚开始只想到了递归中，一个用完就还的思想来实现，虽然复杂了一点，但是比较好理解  

```java
 int deep = 0;
    int Max = Integer.MIN_VALUE;
    public int TreeDepth(TreeNode root) {
        if (root ==null){
            return 0;
        }
        deep ++; //到了就用
        if (root.left == null && root.left == null){ //判断叶子节点。
           if (deep > Max){
               Max = deep;
           }
        }
       TreeDepth(root.left);
       TreeDepth(root.right);
       deep --; // 遍历完子树以后，开始还回来。 即退出了检测
       return Max;
    }
```

大佬简洁做法：  

刚开始是对于max函数的不理解，当相同时，返回的值为相同的值，即max(1,1) = 1.故初始max(0,0) = 0因为+1便开始计数。一直记录最大值。因为是从底下的子结构开始的，所以到达根节点的时候，就返回深度。

```java
 public int TreeDepth2(TreeNode root) {
        if(root==null){
            return 0;
        }
        int left=TreeDepth2(root.left);
        int right=TreeDepth2(root.right);
        return Math.max(left,right)+1;
    }
```

二者的区别， 有类似于，从上面开始计算，开始从下面开始计算的感觉，就是处理数据在递归开始前，还是在递归开始后。  

#### 7. 将二叉搜索树变成双向链表

**题目大意**：将一个搜索二叉树变成一个有序的双向链表。left指向前节点，right指向后节点  ，返回头节点

**思路描述**：因为是有序的，而二叉搜索树中序遍历为有序，所以刚开始想的是用栈实现中序遍历，然后使用队列将数据保存下来，然后重新组合，这种思路比较简单，但是有大佬递归做法  

```java
//使用栈将二叉树变成 双向链表 使用队列和栈进行操作。
    public TreeNode ConvertOld(TreeNode pRootOfTree) {

        if (pRootOfTree == null) return null;

        Stack<TreeNode> stack = new Stack<>();
        Queue<TreeNode> queue = new LinkedList<>();
        stack.push(pRootOfTree);
		
        //二叉树中序遍历的栈写法。
        while (!stack.isEmpty()){
            while (stack.peek().left !=null){ //压入左栈
                stack.push(stack.peek().left);
            }
            while (!stack.isEmpty()){
                TreeNode cur = stack.pop(); //这个操作要在循环中啊
                queue.add(cur);
                if (cur.right!=null){
                    stack.push(cur.right);
                    break;
                }
            }

        }
		//使用队列进行重新组合 
        pRootOfTree = queue.peek();
        while (true){
            TreeNode cur = queue.remove();
            if (!queue.isEmpty()){
                cur.right = queue.peek();
                queue.peek().left = cur;
            }else {
                cur.right = null;
                break;
            }
        }
        return pRootOfTree;
    }
```

大佬递归做法： 因为是返回头指针，即最小值，所以需要先找到最大值，即使用递归找到向右遍历找到最大值，然后记录下来，其中使用一个pre指针一直指向前面的大值，进行组合。 其中有个判断预计 pre==null 只是为了初始化而使用，之前觉得这种做法比较奢侈，其实很关键。。   

```java
  //将二叉排序树变成 双向链表
    TreeNode pre=null;
    public TreeNode Convert(TreeNode pRootOfTree) { //因为要返回头指针，所以需要逆序来进行操作。
        if(pRootOfTree==null)
            return null;
        Convert(pRootOfTree.right); //先去搞右边的。
        if(pre==null)  //这个就是为了获取前面的那个指针，只会运行一次，我之前就不喜欢这种，为了一次运行而协写代码，其实很必要！！切记。
            pre=pRootOfTree;
        else{
            pRootOfTree.right=pre;
            pre.left=pRootOfTree;
            pre=pRootOfTree;
        }
        Convert(pRootOfTree.left);
        return pre;
    }
```

#### 8. 判断二叉树是否为平衡二叉树

**题目大意**：判断二叉树是否为平衡二叉树

**解题思路**：所谓平衡二叉树也就是左右子树的高度相差不超过1，并且每个子树的左高度相差也不超过1，实现的方法就是不断的进行进行左右子树的高度，然后计算差值，作为判断依据。

```java
public int TreeDepth(TreeNode root) {
        if (root == null) return 0;

        int left = TreeDepth(root.left);
        int right = TreeDepth(root.right);

        return Math.max(left,right) + 1;

    }

    //判断是否是平衡二叉树
    //平衡二叉树 左右子树高度不大于2.即最高高度为1。
    public boolean IsBalanced_Solution(TreeNode root) {
        if (root == null )  //这个很关键，对于空的二叉树也是平衡二叉树。
            return true;

       int left = TreeDepth(root.left);
       int right = TreeDepth(root.right);

       if (Math.abs(left - right) > 1)
           return false;
       else {
           return  IsBalanced_Solution(root.left) && IsBalanced_Solution(root.right); //判断左右子树是否也是平衡二叉树。
           //当为遍历到null 的时候就会返回true， 这样的到底的时候就会出现true ，如果不遍历子树可能会出现 子树不是二叉树的情形。
       }
    }
```

#### 9.判断二叉树是否为对称二叉树

**题目描述**：给定一个二叉树根结点，然后判断是否为镜像二叉树。  

**解题思路**：对于对称二叉树，下图所示即为对称二叉树，刚开始在解题的时候，其实想错了，只想到了左右结点相等， 其实不相等的也会出现。其实这个题目还没有想太明白。。

[![duicheng.png](https://pic.tyzhang.top/images/2020/03/17/duicheng.png)](https://pic.tyzhang.top/image/qon)

```java
 boolean isSymmetrical(TreeNode pRoot)
    {
          return pRoot == null ||  judge(pRoot.left,pRoot.right);

    }

    boolean judge(TreeNode node1,TreeNode node2){ //使用两个结点来进行控制。

        if (node1 == null && node2 == null)
            return  true; //同时为null的时候表示为true
        if (node1 == null || node2 ==null)
            return false; //表示只有一个为空的时候，则不对称。需要返回

        if (node1.val != node2.val)
            return false; //这里使用的是false 
        else //意思是如果相等，则接着判断左子树的右子树 与右子树的左子树是否相等。才有条件接着去验证。
            return judge(node1.right,node2.left) && judge(node1.left,node2.right);
    }
```

#### 10.给定一个节点，返回二叉树中序遍历的下一个结点

**题目描述**：给定一个二叉链表树，和其中的一个节点，找到其中序遍历的下一个结点，其中一个结点不仅包括一左右结点，而且包括其父结点。  

**思路描述**：在刚开始的时候就想到使用栈的中序遍历进行实现，即首先找到根结点，然后使用栈的中序遍历，当找到给定结点时，下一个一定是需要的节点，直接进行返回即可。  

ps:中间出现了一个小错误，就是在找到根结点的时候，没有 分清p.next !=null 的判定条件和p!=null的判定条件的意味，其中，前者表示的p一定不会null，但是后者则可能出现，p==null的情况，因为后面都跟着p=p.next。所以需要注意~~  

```java
//通过pNode可以找到其父节点。
    public TreeLinkNode GetNext(TreeLinkNode pNode)
    {
        if (pNode == null)
            return null;

        TreeLinkNode root;
        TreeLinkNode p = pNode;
        while (p.next!=null){  //p 不能为null  否则会出现错误。  不能p!=null 这样p就可能出现null值。
            p = p.next;
        }
        root = pNode;

        Stack<TreeLinkNode> stack = new Stack<>();

        stack.push(root);
        boolean flag = false;
        while (!stack.isEmpty()){
            while (stack.peek().left != null)
                stack.push(stack.peek().left);
			
            while (!stack.isEmpty()){
                TreeLinkNode cur = stack.pop();
                if (flag)//flag为true 则表示此节点为需要找到的值。
                    return cur;
                if (cur == pNode){
                    flag = true;
                }
				
                //为什么是if ？ 因为在找到一个right节点以后，就必须将其所有的左节点加进去进行遍历。
                if (stack.peek().right !=null){ 
                    stack.push(stack.peek().right);
                    break;
                }
            }
        }

        return  null;

    }


public class TreeLinkNode {
    int val;
    TreeLinkNode left = null;
    TreeLinkNode right = null;
    TreeLinkNode next = null;

    TreeLinkNode(int val) {
        this.val = val;
    }
}
```

#### 11.按照之字形打印二叉树

**题目大意**：实现一个按照之字形打印二叉树，即第一行按照从左到右，第二行按照从右到左，第三行按照从从左往右一次类推。  

**解题思路**：刚开始看到第一反应是按照层序遍历使用queue进行遍历，但是在进行手动模拟的时候总是得不到想要的结果，而且控制条件很不好找，然后忽然想到了Stack，因为是后进先出，所以可以通过控制入栈的顺序来达到对应的效果。即第一行使用先左后右入栈，即在第二行输出时是先右向左，在第二行入栈时，先右后左子树进行入栈，这样在输出的时候是先左后右的，就是题干上面的意思。图示如下：

[![zhi.png](https://pic.tyzhang.top/images/2020/03/18/zhi.png)](https://pic.tyzhang.top/image/LHN)

左边为一个二叉树的结构，输出顺序位1 2 3 4 所示的箭头方向，右边为入栈顺序，通过使用两个栈s1,s2来进行交替入栈，即在输出s1栈数据的同时进行对s2的入栈操作。但是需要主要的是需要控制入栈左右子树的顺序，因为输出的倒序的，所以从左往右的入栈顺序位从右往左，反之亦然。  将出栈的结点保存即可。

具体代码：

```java
 //之字形实现二叉树遍历
    public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> result = new ArrayList<ArrayList<Integer>>();
        ArrayList<Integer> list = new ArrayList<>();

        Stack<TreeNode> first = new Stack<>();
        Stack<TreeNode> second = new Stack<>();

        if (pRoot==null) return result;

        first.push(pRoot);
        while (!first.isEmpty() || !second.isEmpty()){
            TreeNode cur; //记录出栈顺序

            while (!first.isEmpty()){ //为空时则为本层遍历结束
                cur = first.pop();
                list.add(cur.val);

                if (cur.left!=null) second.push(cur.left);
                if (cur.right!=null) second.push(cur.right);
            }

            if (!list.isEmpty()){ //将结果保存
                result.add(new ArrayList<>(list));
                list.clear();
            }
            while (!second.isEmpty()){
                cur = second.pop();
                list.add(cur.val);
                if (cur.right!=null) first.push(cur.right);
                if (cur.left!=null) first.push(cur.left);
            }
            if (!list.isEmpty()){
                result.add(new ArrayList<>(list));
                list.clear();
            }
        }
        return result;
    }
```

#### 12.未完待续~

