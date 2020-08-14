---
layout: post
title: Algorithm：基本的算法思想
tags: java
---


> 苦算法久已。所以要好好总结一下

##  目录
* 目录
{:toc}
五大基本算法思想，其实都不怎么会。。

# 动态规划

动态规划也就是最优的子结构，但是要找准最优的子结构是解决什么问题，不能生搬硬套。。

# 贪心算法

贪心算法思想也就是不从宏观出发，只是单纯的从当前角度的触发， 即最优子结构，还有就是

数学符号只是为了推出其规律使得问题更具有一般性，为什么会排斥？

对于动态规范和贪心算法思想的应用，有个很主要的思想就是0-1背包问题。

# 回溯算法

对于回溯算法来说，其实也就是暴力的变体，即如何通过暴力遍历得到所有的值，然后选择合适的内容，在进行使用的时候要想清楚三个条件：

1. 自己的目标
2. 限制条件
3. 可选择的条件

在进行写代码的时候要想清楚以上的三个条件，然后好好想想自己到底要干啥。想清楚以后再进行写，避免总是尝试编码。。

回溯算法也就是遍历所有的解空间，然后寻找合适的解，当找到合适的解以后，便需要将其添加到解空间。

另外，回溯法有个重要的思想就是用完这个选择以后需要删除这个选择，然后做下一种选择。即用完就还的思想。

```python
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

那么什么时候使用回溯法呢？ 如果看到所有、全部的字样，便是要使用回溯法进行遍历解空间。还有一个点需要注意，有些回溯法需要设置一个book[]数组，即使用过的元素在之后都不能再使用，不然会出现很多无用的结果，还有就是设置条件的时候也要多注意，巧妙的设计回退条件能够事半功倍。

例题说明N的全排列，输入一个数字N然后输出其全排列，比如1-3的全排列。 即123 132

```java
public class AllNumber {

    // 使用深度优先搜索进行寻找n的全排列。  res 表示结果  book 表示是否已经被使用。
    static  void  dfs(int step, int [] book, int [] res){

        // 1. 目标
        if (step == res.length){
            System.out.println(Arrays.toString(res));
            return;
        }
		
        // 2.选择列表。 i-res.length 
        for (int i = 1; i <= res.length - 1; i++) {
            if (book[i] == 0){ // 3.book[] 为限制条件
                res[step] = i;
                book[i] = 1; // 4.做选择
                dfs(step + 1,book,res);
                book[i] = 0; // 5.撤销选择
            }
        }
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int [] book = new int[n + 1]; //可选列表。 book就是标记的意思。
        int [] res = new int[n + 1];  //存储结果
        dfs(1,book,res);
    }
}
```

```shell
// 输出结果。
3
[0, 1, 2, 3]
[0, 1, 3, 2]
[0, 2, 1, 3]
[0, 2, 3, 1]
[0, 3, 1, 2]
[0, 3, 2, 1]
```

[93. 复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses/) 

**题目大意：**给定一个全是数字的字符串，输出其所有可能组成的IP地址。

```java
输入: "25525511135"
输出: ["255.255.11.135", "255.255.111.35"]
```

以下的思想就是使用回溯法进行输出。

```java
class Solution {
    
    List<String> result;
    public List<String> restoreIpAddresses(String s) {
        result = new ArrayList<>();
        dfs(0,"",0,s);
        return result;
    }

    // left 表示开始的起点，当left == s.length 的时候，表示满足。 
    // step 表示递归的深度， res表示路径， s表示选择列表， left 表示限制条件
    public void dfs(int step, String res,int left, String s){
        
        // 1. 目标
        if(left > s.length()) return;
        if(step == 4){ // 目标
            if(s.length() == left){
                result.add(res);
            }
            return;
        }else{
			
            // 2.做选择
            for(int i=1;i <= 3 && left + i <= s.length();i++){

                String str = s.substring(left,left + i); // 3. 限制条件 使用窗口的方式进行限制条件，所以不需要进行标记
                int num = Integer.parseInt(str);
                if(str.equals(num + "") && num <= 255){ // 3.限制条件
                    String newRes = res.equals("") ? str : res + "." + str;
                    dfs(step+1,newRes,left+i,s); // 3.因为限制条件是窗口右移，所有返回以后就自动的撤销了限制条件。
                }
            }
        }

    }
}
```

[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/) 

**题目大意：** 给定一个九宫格的键盘，在输入n个字母以后，输出其字典序字母全排列， 这题也是包含全部的字样，所以也是使用回溯方法，即输出所有的解空间。

```java
class Solution {
  public Map<Character, String> init(){
        Map<Character, String> map = new HashMap<>();
        map.put('2',"abc");
        map.put('3',"def");
        map.put('4',"ghi");
        map.put('5',"jkl");
        map.put('6',"mno");
        map.put('7',"pqrs");
        map.put('8',"tuv");
        map.put('9',"wxyz");
        return map;
    }
    
    // 既然是这样那么就需要 将数组保存下来。
    public List<String> letterCombinations(String digits) {

        N = digits.length();
        List<String> avail = new ArrayList<>();
        if(N == 0) return avail;

        Map<Character, String> map = init();
        for (int i = 0; i < digits.length(); i++) {
            avail.add(map.get(digits.charAt(i)));
        }
        dfs(0,new StringBuilder(),avail);
        return res;
    }

    int N=0;
    List<String> res = new ArrayList<>();
    
    // avail 表示可以用的数据。 s 表示此时的结果，即路径
   public void  dfs(int step, StringBuilder s, List<String> avail){

        if (step == N){ // 1. 目标  到这个层则直接的将数据添加。
            res.add(s.toString());
            return;
        }
        // 2. 做选择 使用此时层的数据。
        String str = avail.get(step); //从数组中获取当前使用的数据
        for (int i = 0; i < str.length(); i++) {
            s.append(str.charAt(i)); // 从尾部添加数据，然后 从尾部删除数据。
            dfs(step + 1, s ,avail );
            s.deleteCharAt(s.length() -1); // 因为都传递给下去，所以删除尾结点很关键。 如果是删除头节点的话，容易出现错误。
        }
    }
}
```

[22. 括号生成](https://leetcode-cn.com/problems/generate-parentheses/)

题目大意：又是生成所有的有效的括号的组合，这有时候就需要使用栈去判断。

```shell
输入：n = 3
输出：[
       "((()))",
       "(()())",
       "(())()",
       "()(())",
       "()()()"
     ]
```

解题思路，观察可以发现，有效的组合最左边肯定是(，最右边肯定是 )，当n=3的时候，共有3\*2个( )，所以中间有 () ()的全拍了，即 (3-1)\*2个括号的排列，所以step == (3-2）\*2就可以，选择总共有2个，比如说2个)，当n=4的时候，总共有3个)可以选择。

```java
class Solution {
    // 可以选择的列表，
     public Map<Integer,Character> init(){
        Map<Integer,Character> map = new HashMap<>();
        map.put(0,'(');
        map.put(1,')');
        return map;
    }

    public List<String> generateParenthesis(int n) {
        STEP = 2*n - 2 ; // 减一乘2   当 n= 3的时候，只需要使用 3个进行排序
        N = n -1;
        if ( n == 0) return res;
        if ( n == 1){
            res.add("()");
            return res;
        }

        book.put('(',0); //记录已经使用了多少次， 比如n=3的使用 能用2此，n=4的时候，能用3次。
        book.put(')',0);
        // 初始化直接加上左括号
        dfs(0,new StringBuilder("("),init());
        // System.out.println(res);
        return res;
    }

    List<String> res = new ArrayList<>(); // 存储结果。
    Map<Character, Integer> book = new HashMap<>();
    int STEP = 0; //递归深度
    int N = 0;  // 能够使用的列表。
    public void dfs(int step, StringBuilder s, Map<Integer,Character> map){
        if (step == STEP){ // 1.目标
            // 结尾加上右括号。
            s.append(")"); 
            // 需要判断此时的括号是否有效
            if (isValid(s)){
                res.add(s.toString());
            }
            s.deleteCharAt(s.length() - 1); // 最后添加的也要删除，不然会越来越多。。
            return;
        }
        //2. 做选择 如果出现了两次 那么就不能再使用。
        for (int i = 0; i < map.size(); i++) {
            char c = map.get(i);
            int count = book.get(c);
            if (count < N){ // 3.限制条件
                s.append(map.get(i));
                book.put(c,++count); // 4.使用选择
                dfs(step+1,s,map);
                book.put(c,--count); // 4.撤销选择
                s.deleteCharAt(s.length() - 1); //删除新添加的值。
            }
        }
    }
	
    // 使用栈判断括号列表是否有效。
    public boolean isValid(StringBuilder s){
        Stack<Character> stack = new Stack<>();

        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (c == ')'  && !stack.isEmpty() && stack.peek() == '(' ){
                stack.pop();
            }else {
                stack.push(c);
            }
        }
        return stack.isEmpty();
    }
}
```

[51. N皇后](https://leetcode-cn.com/problems/n-queens/) 

经典的回溯算法，主要就是要遍历出来所有的结果，另外有一点需要说明，要深刻理解step这个变量，他其实是控制走到哪一层的关键，即首先是找出第一行满足条件的位置，然后找出第2行满足的位置，知道找到第n-1行的位置，然后在此返回即每一层只需要遍历完每一行的位置即可，而不是没有用的功能，之前写成step用，直接每一层都都是寻找n*n上的位置，就重复了很多，另外需要多理解基础数据类型的数组，char\[][]二维数组是没有想到的。

Java中表示字符串的两种方法

| 操作             | 字符串数组      | Java字符串       |
| ---------------- | --------------- | ---------------- |
| 声明             | char[]          | String s         |
| 根据索引访问字符 | a[i]            | s.charAt(i)      |
| 获取字符串长度   | a.length        | s.length()       |
| 表示方法转换     | s.toCharArray() | s= new String(a) |

```java
class Solution {
   // n个皇后放在 n*n的棋盘上，找到所有能够防止的位置。
    public List<List<String>> solveNQueens(int n) {
        N = n;
        result = new ArrayList<>();
        if (n == 0) return result;

        char [][] mark = new char[n][n]; //存储对应的棋盘

        for (int i = 0; i < N ; i++) { // 使用char型数组真是太秀了吧。。
            for (int j = 0; j < N; j++) {
               mark[i][j] =  '.';
            }
        }
        dfs(0,mark);
        // System.out.println(count);
        // System.out.println(result);
        return result;
    }
    List<List<String>> result; //存储结果

    int N = 0;  //总共的棋盘

    void dfs(int step, char[][] mark ){
        if (step == N){ // 结束条件。 如果前三个能够完成，然后这个其实也是会
            List<String> res = new ArrayList<>();
            for (int i = 0; i < step; i++) { // 这是怎么做到的？
                res.add(String.valueOf(mark[i])); // 因为mark是个二维数组，所以mark[1]就是将一整行输入录入进去。
            }
            result.add(res);
            return;
        }

        // step 表示传递过来的 列数，不能双循环，这样会重复很多的东西。。
        for (int j = 0; j < N; j++) {
            if (mark[step][j] == '.' && isValid(step,j,mark)){
                mark[step][j] = 'Q';
                dfs(step + 1,  mark);
                mark[step][j] = '.';
            }
        }

    }

    // 判断当前位置是否有效
    private boolean isValid(int i, int j,char [][] mark) {
        // 在grid进行米字型遍历，如果没有则成功，
        // 向右遍历
        int x = i;
        int y = j;
        while (y+1 < N){ // 判断右边
            y++;
            if (mark[x][y] == 'Q') return false;
        }

        x= i; y = j;
        while (y-1 >= 0){//判断左边
            y--;
            if (mark[x][y] == 'Q') return false;
        }

        x = i; y = j;
        while (x-1 >=0){ // 判断上边
            x--;
            if (mark[x][y] == 'Q') return false;
        }

        x = i; y = j;
        while (x+1 < N){ //判断下边
            x++;
            if (mark[x][y] == 'Q') return false;
        }

        x = i; y = j;
        while (x-1 >=0 && y-1>=0){ // 左上
            x--;
            y--;
            if (mark[x][y] == 'Q') return false;
        }

        x = i; y = j; // 左下
        while (x + 1 < N && y-1 >=0){
            x++;
            y--;
            if (mark[x][y] == 'Q') return  false;
        }

        x = i; y = j;
        while (x+1 <N && y+1< N){
            x++;
            y++;
            if (mark[x][y] == 'Q') return  false;
        }

        x = i; y = j;
        while (x-1>=0 && y+1 < N){
            x--;
            y++;
            if (mark[x][y] == 'Q') return  false;
        }
        return true;
    }
}
```

# 分治算法

其实也就是将一个大问题分解成多个小问题，然后依次求解小问题，最后整个大的问题也被解决，主要的运用就是归并排序，快速排序，递归也算是一种分治思想的实现。

# 枚举算法

相当于是暴力破解的思想，然后将所有的值去试试，例题就是 百钱买百鸡问题，就是将所有的可能的值尝试一遍。然后找出最优解。 相当于是回溯算法？

# 图的遍历

对于图来说，说复杂也不复杂，主要就是想清楚，另外就是图存储的使用。图的遍历又不一样，虽然和回溯算法框架是类似的，但是要知道遍历的二维数组是不一样的，不过都是利用了深搜的思想。另外图的话是为了遍历所有的节点，而每一次进入递归都是要将该结点访问或者是保存起来，表示遍历解说，然后根据便找到下一个结点。 另外step可以认为就是当前遍历的节点。

```java
public class 图的深度优先遍历 {

    static int X;
    static int Y;
    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);
        
        Y = X = 5; // 输入行和列。 因为肯定正方形的。所以是相同的值。
        int [][] grid = new int[X][Y];
        book = new int[X]; // 标记哪个定点已经遍历了。
        for (int i = 0; i < X; i++) {
            for (int j = 0; j < Y; j++) {
                if ( i == j) grid[i][j] = 0;
                else grid[i][j] = -1; // -1 表示此路不通。
            }
        }
        // 输入边。
        grid[0][1] = 1;
        grid[0][2] = 1;
        grid[1][3] = 1;
        grid[2][4] = 1;
        grid[0][4] = 1;
        List<Integer> res = new ArrayList<>();

        // 然后就是深度优先遍历了。
        dfs(0,grid,res);
        System.out.println(res);
    }
    // 标记数组
    static int [] book; // book[i] == 1 表示i节点已经被遍历过。

    // step 表示的就是当前满足条件访问到的节点， grid 表示就是需要遍历的图， res 表示遍历的结果。
    static void dfs(int step, int [][] grid, List<Integer> res){ // 首先是按照第一个定点进行遍历，然后进行
        // 因为是遍历，所以需要先直接将元素添加进来。
        res.add(step);
        if ( res.size() == X){ // 如果将所有的数据遍历了，便可以直接的返回。
            return;
        }
        for (int i = 0; i < X; i++) {
            // 第 i 条数据没有被访问，并且有路才能接着访问。
            if (book[i] == 0 && grid[step][i] == 1){ // 如果有路并且没有没有访问过则直接的走过去
                book[i] = 1; // 标记当前的节点已经被访问。
                dfs(i,grid,res);
            }
        }
    }
}
```

# 常用的解题技巧

1. 双指针法，左右指针或者是数组下标进行维护
2. 逆序数组遍历，对于遍历来说，有些数组的遍历使用逆序遍历效果更好。
3. 最小栈使用方法。
4. 暴力破解方法。
5. 快慢指针做法，比如解决链表的环。 快速找到俩表的中间位置
6. 处理链表的时候 先加个头指针，然后就可以归一化的处理，即更好的处理
7. 类比于Map中的思想，如果使用头插法会好点。
8. 对于有规律的数据，为了加快速度可以使用打表的方法进行加快速度，即先将所有的结果存储，或者使用数组，或者使用Map进行存储。
9. 如果有说是循环数组，那么就直接将数组直接的加上去，成为一个2n的数组，具体的操作就是使用将循环变成2n，但是下标使用 i%n模拟循环。

# 常用概念

回文：对于回文来说，aba 这种对称的数据。

素数、也叫做质数：

> ```
> “素数”，又称“质数”，是指：
> 除1和其自身之外，没有其它约数的正整数
> 如 2，3，5，7，11，13，17，19，23，29，31，37，41，43，47，...
> ```

奇数、偶数

> 奇数（odd）指不能被2整除的整数 ，数学表达形式为：2k+1。
>
> 偶数是能够被2所整除的整数。

素数打表算法：

```java

final static int maxn = 1000000 + 10;
// 素数
static boolean[] isp = new boolean[maxn];

for (int i = 0; i < maxn; i++) {
    isp[i] = true;
}
isp[0] = false;
isp[1] = false;

// 素数的倍数就不是素数，一并处理
for (int i = 2; i < maxn; i++) {
    if (isp[i] == true) {
        for (int j = i + i; j < maxn; j += i) {
            isp[j] = false;
        }
    }
}
```

不超过A，则表示小于等于A，那么等于A的也算。要看好题目的意思，对于写出来一个能用的程序其实也是很重要的。不能啥都不知道。。

字典序： 对于字母来说，相当于是字典中数据的排列方式，类似于英语单词的编排方式。比如说 ab 就是排在 b之前，那么在计算中，1是排在2之前， 那么12也是排在2之前。 这次又是不知道的题意。。操 	

对于一些数据来说，找规律的方法很好用，不要老是想着找出所有的数据，然后进行直接取出来，其实不用这样，可以试试找规律，这些人写的代码效率又高又优美这才是一个程序员应该具有的素质。

说白了刷题训练的也就是自己解决实际问题的能力。每日一刷已经成为习惯。。

学会处理String字符串具有很大的帮助，常用的API，比如charAt()  subString()等等函数，另外在使用这些函数的时候，最好先String s = str.subString(0,1)这种方式进行使用，因为这样会保存下来，之后在进行处理的时候，比较方便，还有就是尽量少进行删除操作，如果不行可以使用滑动窗口的方法进行获取值。

## 二维数组

```java
double [][] a = new double [M][N]
```

上面的便称为M*N数组，我们约定，第一维为行数，第二维为列数，其内部如下所示

```java
public static void main(String[] args) {
    int [][] a =  {{1,2,3},
                   {4,5,6},
                   {7,8,9},
                   {10,11,12}};
    System.out.println(a[0][1]); //2 表示输出第0行的第一列。 即 2
}
```

比如以上就是一个4*3的数组，即M=4，N=3的数组，其实也相当于M表示的就是一维数组中的元素的个数，而3表示每个元素内部的元素的数据。

## 最小栈

活用最小栈可以很方便的解决问题，最小栈也就是说对于数组进行线性遍历，进栈的标准为 如果小于栈顶的元素则进行入栈，如果碰到大于栈顶的元素，则直接的弹出元素，直到栈顶的元素不大于此时的元素，这种可以解决最小子串的问题。

[739. 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

**题目大意**：给定一组温度，需要计算出如果要找到更高的温度，至少需要等几天？ 其实也即是找到数组中最近的下一个大的值，如果没有则输出0，此时就很适合使用最小栈。维护一个里面都是小值的栈，但是只是存储下标，因为需要更新对应的下标的值。

在存储下标还是值这个问题，可以根据具体的场景去分析，但是最好还是使用下标，因为对于下标来说，更具有唯一性，而数组中的值可能是相同的，

```java
class Solution {
    public int[] dailyTemperatures(int[] T) {
        
        if(T.length == 0) return T;
        int [] res = new int[T.length];
        Stack<Integer> stack = new Stack<>();
        stack.push(0);
        int index = 0;
        for (int i = 1; i < T.length ; i++) {

            while (!stack.isEmpty() && T[stack.peek()] < T[i]){
                // 当 发现有大的值的时候。
                int cur = stack.pop(); // 需要记录当时的值， 因为栈中存储的数据就是 下标。
                res[cur] =  i - cur; // 记录此时的值。
                index ++;
            }
            stack.push(i);
        }
        stack.clear();
        return  res;
    }
}
```

[496. 下一个更大元素 I](https://leetcode-cn.com/problems/next-greater-element-i/)

题目大意：找到数组中的下一个更大的元素，与上面题目的解题思路其实是差不多的，也是维护一个最小的栈，然后输出其下标，

```shell
输入: nums1 = [4,1,2], nums2 = [1,3,4,2].
输出: [-1,3,-1]
解释:
    对于num1中的数字4，你无法在第二个数组中找到下一个更大的数字，因此输出 -1。
    对于num1中的数字1，第二个数组中数字1右边的下一个较大数字是 3。
    对于num1中的数字2，第二个数组中没有下一个更大的数字，因此输出 -1。
```
具体的做法就是先将nums中的所有元素的下一个更小值存储在Map中，然后在对num1进行遍历取值输出。
```java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {

        // 使用栈去维护一个Map，然后通过Map直接的进行选择。 不用直接慢慢的找到。，
        
        if(nums1.length == 0) return nums1; // 如果为空则直接的返回

        int [] res = new int[nums1.length];
        Stack<Integer> stack = new Stack<>();
        HashMap<Integer,Integer> map = new HashMap<>(); // 存储数据。
        
        stack.push(nums2[0]);
        for(int i=1;i<nums2.length;i++){
            
            // 如果栈不为空， 并且当前元素也大于当前栈顶，此时就会直接的的加入 
            while(!stack.isEmpty() && nums2[i] > stack.peek()){
               map.put(stack.pop(),nums2[i]); //
            }
            stack.push(nums2[i]);
        }

        while (!stack.empty())
            map.put(stack.pop(), -1);

        for(int i = 0; i < nums1.length ; i++){
            res[i] = map.get(nums1[i]);
        }

        return res;
    }
}
```

## && ||截断功能

利用好这个阶段功能，将简单的判断放在前面，复杂的判断方法后面，这样前面的简单的判断如果不满足就会直接的阶段，然后后面的复杂操作就会直接跳过，这样就会实现了加快计算的结果，

## String常用API

```java
public class String{
    	// 创建一个空字符串
    	String()
            
        // 字符串长度
    	int length()
            
        // 第i个字符
        int charAt(int i)
            
        // p第一次出现的位置，如果没有则返回-1
        int indexOf(String p)
            
        // p在i个字符后第一次出现的位置，如果没有则返回-1
        int indexOf(String p,int i)
            
        // 将t附在该字符串末尾
        String concat(String t)
            
        // 该字符串的子字符串（第i到j-1个字符）
        String subString(int i,int j)
            
        // 使用delim分隔符切分字符串
        String[] split(String delim)
            
        // 比较字符串
        int compareTo(String t)
            
        // 该字符串的值和t的值是否相同
        boolean equals(String t)
            
        // 散列值。
        int hashCode()
}
```

一般来说以上的就是比较常用的api，如果还有就是StringBuilder的使用方式。还有就是 +符号，+符号可以直接的组装String字符串。

```java
public final class StringBuilder{
	// 新建一个字符串
    public StringBuilder()
    // 在末尾添加一个字符
    public StringBuilder append(char[] str)
	
    public StringBuilder appendCodePoint(int codePoint)
	// 删除从start到end的字符
    public StringBuilder delete(int start, int end)
	// 删除指定index的字符，一般可以用在 size()-1 即删除末尾的字符
    public StringBuilder deleteCharAt(int index)
	// 进行字符的代替
    public StringBuilder replace(int start, int end, String str)
	// 插入字符
    public StringBuilder insert(int offset, double d)
	// 将字符进行反转。
    public StringBuilder reverse()
}
```

## 基本数据类型

**byte：**

- byte 数据类型是8位、有符号的，以二进制补码表示的整数；
- 最小值是 **-128（-2^7）**；
- 最大值是 **127（2^7-1）**；
- 默认值是 **0**；
- byte 类型用在大型数组中节约空间，主要代替整数，因为 byte 变量占用的空间只有 int 类型的四分之一；
- 例子：byte a = 100，byte b = -50。

**short：**

- short 数据类型是 16 位、有符号的以二进制补码表示的整数
- 最小值是 **-32768（-2^15）**；
- 最大值是 **32767（2^15 - 1）**；
- Short 数据类型也可以像 byte 那样节省空间。一个short变量是int型变量所占空间的二分之一；
- 默认值是 **0**；
- 例子：short s = 1000，short r = -20000。

**int：**

- int 数据类型是32位、有符号的以二进制补码表示的整数；
- 最小值是 **-2,147,483,648（-2^31）**；
- 最大值是 **2,147,483,647（2^31 - 1）**；
- 一般地整型变量默认为 int 类型；
- 默认值是 **0** ；
- 例子：int a = 100000, int b = -200000。

**long：**

- long 数据类型是 64 位、有符号的以二进制补码表示的整数；
- 最小值是 **-9,223,372,036,854,775,808（-2^63）**；
- 最大值是 **9,223,372,036,854,775,807（2^63 -1）**；
- 这种类型主要使用在需要比较大整数的系统上；
- 默认值是 **0L**；
- 例子： long a = 100000L，Long b = -200000L。
  "L"理论上不分大小写，但是若写成"l"容易与数字"1"混淆，不容易分辩。所以最好大写。

**float：**

- float 数据类型是单精度、32位、符合IEEE 754标准的浮点数；
- float 在储存大型浮点数组的时候可节省内存空间；
- 默认值是 **0.0f**；
- 浮点数不能用来表示精确的值，如货币；
- 例子：float f1 = 234.5f。

**double：**

- double 数据类型是双精度、64 位、符合IEEE 754标准的浮点数；
- 浮点数的默认类型为double类型；
- double类型同样不能表示精确的值，如货币；
- 默认值是 **0.0d**；
- 例子：double d1 = 123.4。

**boolean：**

- boolean数据类型表示一位的信息；
- 只有两个取值：true 和 false；
- 这种类型只作为一种标志来记录 true/false 情况；
- 默认值是 **false**；
- 例子：boolean one = true。

**char：**

- char类型是一个单一的 16 位 Unicode 字符；
- 最小值是 **\u0000**（即为0）；
- 最大值是 **\uffff**（即为65,535）；
- char 数据类型可以储存任何字符；
- 例子：char letter = 'A';。

对于数据的溢出与整理需要时刻的注意，即int更大的范围为long， flaot更大的数据为double。对于各种ASCII码来说，可以直接使用对应的码进行相减。比如判断A的 char a='A';直接等于就行，但是需要注意，a > A即小写的字母的ASCII码更大。

## 封装类型

左边原始类，右边封装类。

| boolean | Boolean   |
| ------- | --------- |
| char    | Character |
| byte    | Byte      |
| short   | Short     |
| int     | Integer   |
| long    | Long      |
| float   | Float     |
| double  | Double    |

   其中使用的最多的就是Integer.parseInt()，将string类型转成int数据，还有就是Integer.MAX_VALUE和Integer.MIN_VALUE。

## 线性获取栈中的最小数据

做题时候发现，通过设置可以线性的获取到栈中的最小值，实现的方式就是设置一个min变量一直保存最小值，每次进栈的时候都会与当前最小值比较，如果比最小值更小，则先将当前最小值入栈，然后在入栈当前最小值，并且将min更新为当前最小值。在出栈的时候，会先检测当前出栈是否为最小值，如果是最小值，则出栈两次，第二次出栈的就是最小值， 并且更新min。 另外一种更简单的做法就是维护一个最小栈，也是类似的思想，出栈时候比较栈顶元素与最小值。但是更加的浪费，这种只需要维护一个栈就行。

```java
    class MinStack {
        Stack<Integer> stack = new Stack<>();
        int min = 0;
        public MinStack() {
        }

        // 入栈
        public void push(int x) {
            // 只用一个栈进行实现。
            if (stack.isEmpty()){
                min = x;

            if (min >= x){ // 将上一个小的值入栈，因为栈的操作是有序的，所以如果发现有一个更小的值，那么就直接将其入栈，在出栈的时候，判断出栈的是否为最小值，如果
                // 是最小值则 出栈两次，第二次弹出栈的就是更小的值。
                stack.push(min);
                min = x;
            }
            stack.push(x);
        }

        // 出栈
        public void pop() {
            if (stack.peek() == min){
                min = stack.pop();
            }
            stack.pop();
        }

        public int top() {
            return stack.peek();
        }

        public int getMin() {
            return min;
        }
    }
```

