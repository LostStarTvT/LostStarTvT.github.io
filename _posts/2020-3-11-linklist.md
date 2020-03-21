---
layout: post
title: 剑指offer解题记录
tags: Algorithm  
---


> 记录剑指offer上面各种比较杂的解题过程。

#### 1. 复杂链表的处理

**题目描述**：输入一个复杂链表，（每个节点有结点值和两个指针，其中一个指向下一个，另外一个指向一个任意结点），返回复制后复杂链表的头指针，返回的结果中不能包含原链表的元素，即复制之后的元素都是新new的。

**解题思路**：刚开始其实没有真正的理解题目的意思，首先，一个链表肯定是连续的，所以通过next指针就能完全的遍历完链表，虽有有random指针，但也不能只有random能访问到而next指针不能访问到，所以现在问题就比较好理解了，通过next复制完全部的结点，然后在将所有的random指针指向对应的结点。实现的步骤包括三步：  

1. 通过next复制所有结点，将新建结点插入到每个节点后面，
2. 通过遍历将random指向复制
3. 拆分链表，

[![complexLinkList.md.jpg](https://pic.tyzhang.top/images/2020/03/17/complexLinkList.md.jpg)](https://pic.tyzhang.top/image/FbH)

**具体的代码**

```java
public RandomListNode CloneNew(RandomListNode pHead)
    {
        if (pHead == null) return null;
        RandomListNode cur = pHead;

        RandomListNode pre;
        while (cur != null){
            RandomListNode newNode = new RandomListNode(cur.label);
            //将其插入到链表中。
            pre = cur.next;
            cur.next = newNode;
            newNode.next = pre;
            cur = pre;
        }

        //复制随机链表
        //
        cur = pHead;
        while (cur!=null ){
            if (cur.random!=null){ //如果不为null
                cur.next.random = cur.random.next; //因为相同的链表就是后面的那个。 cur.random 为原始的数据 .next 就是新建的那个。
            }
           cur =  cur.next.next;
        }

        //拆分链表
        cur = pHead;
        pre = cur.next;
        RandomListNode head = pre;
        while (cur!=null){
            cur.next = pre.next;
            cur = pre.next;
            if (cur ==null){
                break;
            }
            pre.next = cur.next;
            pre = cur.next;
        }
        return head;
    }


class RandomListNode {
    int label;
    RandomListNode next = null;
    RandomListNode random = null;

    RandomListNode(int label) {
        this.label = label;
    }
}
```

#### 2.查找数组中的某值

**题目大意**：给定一个数组，找出该数组中存在超过该数组长度一般的数字，例如{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

**解题思路**：最直观的想法就是调用排序接口，然后相等的数据就会出现在一起，然后使用循环进行遍历，但是这样的话就会非常的简单就解决。另外一种解法就是，利用满足条件的数组性质进行解题，即通过依次遍历，最后得到的数据一定的超过一般的值。  

排序解法：

```java
 public int MoreThanHalfNum_Solution(int [] array) {

        if (array.length == 0) return 0;
        if (array.length == 1) return array[0];
        Arrays.sort(array);

        int temp =array[0];
        int count = 1;
        int halfLength = array.length / 2;
        for (int i = 1; i < array.length; i++) {
            if (temp == array[i]){
                count ++;
                if (count > halfLength)
                    return temp;
            }else {
                temp = array[i];
                count = 1;

            }
        }
        return 0;
    }
```

大佬利用性质解法:同归于尽解法，通过对比相邻数字是否相同，一趟遍历最后一定是改数字。

```java
public int MoreThanHalfNum_SolutionNew(int [] array) {
        if (array.length == 0) return 0;
        int preNum = array[0];
        int count = 1;
        for (int i = 1; i < array.length; i++) {
            if (preNum == array[i]){ //如果相同 则记录下来
                count ++;
            }else {
                count -- ;
                if (count == 0){ //如果为空了。则是该数字
                    preNum = array[i];
                    count = 1;
                }
            }
        }
        count = 0;
        for (int i = 0; i < array.length; i++) { //找到最多的数字，然后进行统计其出现的次数。
            if (preNum == array[i])
                count++;
        }
        return count > array.length/2? preNum:0;
    }
```

#### 3.查找数组中前k小的数据

**题目大意**：对于给定的数组，输出其中前k小的数据，例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。  

**算法思想**：刚开始想的也是直接利用排序算法进行输出.. 然后别人思路也是，但是不是调用系统的api，而是使用自己写的堆排序，通过计算小堆然后实现输出前k小的值。  

排序输出：

```java
 public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer> arrayList = new ArrayList<>();

        if (input.length == 0 || input.length < k) return arrayList;
        Arrays.sort(input);
        for (int i = 0; i < k ;i++) {
            arrayList.add(input[i]);
        }
        return arrayList;
    }
```

大佬写法：还不会写..

#### 4.未完待续~~