---
layout: post
title: 递归
tags: Algorithm
---


> 本篇主要介绍一下递归，然后加上几个例子进行演示。

#### 递归三大要素

- 明白这个函数要干什么 (在进行回调的时候，内部的细节可以没有想好，但是在使用的时候是直接进行使用的。)

>  我们先不管函数里面的代码什么，而是要先明白，你这个函数是要用来干什么。 

- 寻找递归结束条件

>  所谓递归，就是会在函数内部代码中，调用这个函数本身，所以，我们必须要找出**递归的结束条件**，不然的话，会一直调用自己，进入无底洞。也就是说，我们需要找出**当参数为啥时，递归结束，之后直接把结果返回**，请注意，这个时候我们必须能根据这个参数的值，能够**直接**知道函数的结果是什么。 

- 找出函数的等价关系式

>  第三要素就是，我们要**不断缩小参数的范围**，缩小之后，我们可以通过一些辅助的变量或者操作，使原函数的结果不变。 

[参考链接](https://mp.weixin.qq.com/s/mJ_jZZoak7uhItNgnfmZvQ)    

#### 反转单链表

> 反转单链表，例如1->2->3->4。反转为4->3->2->1

思路主要是使用递归的方法进行链表的反转。代码如下。

首先需要输入链表的个数m，然后输入m个链表元素形成链表。

```java
package day3;

import java.util.Scanner;

/**
 * @author Seven
 * @description 进行测试递归的类
 * @create 2019-11-16 15:27
 **/

public class recursion {


    static Node reverseNode(Node head){ //进行反转链表
        if (head.next == null){ // 递归结束条件
            return head;
        }
        // 以下是等价关系式
        Node newNode = reverseNode(head.next);
        // 改变 1，2节点的指向。
        // 通过 head.next获取节点2
        Node t  = head.next; //不能直接使用newNode的指针。
        // 让 2 的 next 指向 2
        t.next = head;
        // 1 的 next 指向 null.
        head.next = null;
        System.out.println(newNode.next); //通过输出地址可以发现new的地址是没有在变化的。
        return newNode;
    }

    public static void main(String[] args) {

        // 以下是进行java中链表的输入。
        Scanner sc = new Scanner(System.in);
        int m = sc.nextInt(); // 获取链表的长度

        Node head = new Node(); //定义头指针
        Node p = head;   //使用p q指针进行创建链表
        int n = sc.nextInt(); //输入链表中第一个数据
        head.data = n;

        for (int i = 1; i <m ; i++) {// 输入1-m个数据
            n = sc.nextInt();
            Node q = new Node();
            p.next = q;
            q.data = n;
            p = q;
        }

        head = reverseNode(head);
        System.out.println(head.next);

        while (head != null){
            System.out.println(head.data);
            head = head.next;
        }
    }

    static class Node{
        int data;
        Node next;
    }
}
```

整个递归的形式如下图所示：

1. 表示刚进入链表的时候的情况
2. 表示在进行到循环到结束时候进行的操作。
3. 表示进行一次指针调整，即将指针反转、
4. 表示在进行3以后的整个链表的状态。其中newNode是一直指向newNode的。而一直进行改变的就是Head  和head->next的指针。一直返回newNode都是指向尾结点。

![递归图示](/image/listReverse.png)