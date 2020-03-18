---
layout: post
title: 剑指offer解题记录
tags: java  Algorithm  
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

