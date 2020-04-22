---
layout: post
title: Java：基础知识
tags: java
---


> 本篇主要介绍java的一些常用的基础知识，例如输入输出。


### 单行输入多个数据

在不知道数组长度的情况下输入数据。 

```java
 public static void main(String[] args) {

        // 在不知道数组长度的情况下进行数组的输入
        Scanner sc=new Scanner(System.in);
        String[] nums = null;
        nums = sc.nextLine().split(" "); //读取一行。在输入回车的时候进行停止输入
        int num[]=new int[nums.length];
        for(int i=0;i<num.length;i++){
            num[i]=Integer.valueOf(nums[i]);
        }
        
        for (int n:num) {
            System.out.println(n);
        }
   }
```
进行输入的结果显示:  
![样例显示](/image/inputLine.png)

### 指定数组长度然后进行循环输入

在进行刷题的时候，常用的指定数组的长度，然后进行输入数组的数据

```java
 Scanner sc = new Scanner(System.in);
    public static void main(String[] args) {
        System.out.println("输入：");
        Scanner sc = new Scanner(System.in);
        int m = sc.nextInt();
        int n = sc.nextInt();
        int[] num1 = new int[m];
        int[] num2 = new int[n];
        // 换成其他数据类型也一样，其他数值类型就修改int跟nextInt就可以了，
        //String就把nextInt()换成next()
        for(int i = 0; i < m; i ++) {
            num1[i] = sc.nextInt();  // 一个一个读取
        }
        for(int i = 0; i < n; i ++) {
            num2[i] = sc.nextInt();
        }
        System.out.println("输出：");
        System.out.println(Arrays.toString(num1));
        System.out.println(Arrays.toString(num2));
    }
```
进行输入的结果显示：  
![输入多个](/image/inputMulit.png)

以上两种可以满足大多数进行输入的情况。