---
layout: post
title: Algorithm：01背包问题总结
tags: java

---

> 一直不怎么会01背包问题，今天进行一次总结。

## [416：分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum)

**题目大意**：给定一个质保函正整数的非空数组，是否可以将这个数组分割为两个子集，使得两个子集的元素和相等。

```shell
实例 1：
输入: [1, 5, 11, 5]
输出: true
解释: 数组可以分割成 [1, 5, 5] 和 [11].


示例 2:
输入: [1, 2, 3, 5]
输出: false
解释: 数组不能分割成两个元素和相等的子集.
```

[**解题思路**](https://leetcode-cn.com/problems/partition-equal-subset-sum/solution/0-1-bei-bao-wen-ti-xiang-jie-zhen-dui-ben-ti-de-yo/)：这个题乍一看没有思路，其实是一个01背包问题，但是需要一点的转化，首先，看题目，分割为两个子集并且元素的和相等，那么这说明了什么？比如10+10 = 20 这时候是相等的，也就是说，当在这个数组中找到和为target = sum/2 的数据就表明是有的。

**是否可以从输入数组中挑选出一些正整数，使得这些数的和 等于 整个数组元素的和的一半。** 另外这个数组的和一定是要为偶数，因为相同数相加一定是偶数。

那么这就转换成了一个dp问题，对于dp\[i][j]来说，i表示第几个数，j表示和，当dp\[len][target]的值就是在0-len中能够找到相加为taget的任意个数元素。

状态转移公式：

num[i] == j  则 dp\[i][j] = true;  因为这时候肯定能从0-i中找到等于j，即num[i]，自己。

j > num[i]  则 dp\[i][j] = dp\[i-1][j] or dp\[i-1][j-nums[i]]  因为要保证 j - nums[i] 大于等于0；

other 则 dp\[i][j] = dp\[i-1][j]  即和上一层同列的值一样。

整个的代码：

```java
class Solution {
    public boolean canPartition(int[] nums) {

        if(nums.length == 0) return false;
        //问题转换成了 从给定的数组中能否找到 几个元素使得相加等于数组中所有数据的相加的一半

        int sum = 0;
        for(int ans:nums)
            sum += ans;
        if((sum & 1) == 1) return false; // 如果是奇数 则直接返回false 因为两个相等的数据相加肯定是偶数

        int len = nums.length;
        int target = sum / 2;
        boolean [][] dp = new boolean[len][target+1]; //因为需要用到target这一列。

        if(nums[0] <= target)
            dp[0][nums[0]] = true; 

        for(int i = 1; i < len; i++ ){
            for(int j=0; j <= target; j++){

                dp[i][j] = dp[i-1][j];// 先抄下来上一层的状态

                if(nums[i] == j){
                    dp[i][j] = true; // 首先自己肯定可以找到的。
                    continue;
                }

                if(j > nums[i]){
                    dp[i][j] = dp[i-1][j - nums[i]] || dp[i-1][j];
                }
            }
        }
        return dp[len - 1][target];
    }
}
```

整个dp的图如下：

![image-20200920161603168](C:\Users\Seven\AppData\Roaming\Typora\typora-user-images\image-20200920161603168.png)

可以看出真的很有规律。只是和上一层的数据有关系。等到最后便能够得到结果，但是对于初始化的设置需要仔细想想。 

从上面的表格可以看出，当一列出现一个true的时候，下面的一列都会变成ture，所有可以进行优化一下，另外可以看出，第一种写法中，当num[i] == j的时候，直接变成true，其实这时候也就是去去 dp\[i-1][0]的值，但是如果我们将dp\[0][0]变成 true，那么就可以将其归一类类，变成下面的一种写法，

j >= num[i] 则  dp\[i][j] = dp\[i-1][j] or dp\[i-1][j-nums[i]]  因为我们把dp\[0][0]变成了True。

另外一种写法：

```java
public class Solution {

    public boolean canPartition(int[] nums) {
        int len = nums.length;
        if (len == 0) {
            return false;
        }
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        if ((sum & 1) == 1) {
            return false;
        }

        int target = sum / 2;
        boolean[][] dp = new boolean[len][target + 1];
        
        // 初始化成为 true 虽然不符合状态定义，但是从状态转移来说是完全可以的
        dp[0][0] = true;

        if (nums[0] == target) {
            dp[0][nums[0]] = true;
        }
        for (int i = 1; i < len; i++) {
            for (int j = 0; j <= target; j++) {
                dp[i][j] = dp[i - 1][j];
                if (nums[i] <= j) {
                    dp[i][j] = dp[i - 1][j] || dp[i - 1][j - nums[i]];
                }
            }

            // 由于状态转移方程的特殊性，提前结束，可以认为是剪枝操作
            if (dp[i][target]) {
                return true;
            }
        }
        return dp[len - 1][target];
    }
}
```

优化为1维的解题：

 **因为当前j的状态只和0-j-1的状态相关，所以就可以倒着进行判断，因为在更新的时候，前面的状态不会被破坏。**

```java
class Solution {
    public boolean canPartition(int[] nums) {
        
        if(nums.length == 0) return false;

        int sum = 0;
        for(int ans: nums)
            sum += ans;
        if((sum & 1) == 1) return false;

        int len = nums.length;
        int target = sum / 2;
        boolean []dp = new boolean[target + 1];
		
        // 通过这两句便已经初始化了数组，这是能够从后面进行更新的基础。
        dp[0] = true;
        if(nums[0] <= target) dp[nums[0]] = true;
        
        
        for(int i = 1; i < len ; i++ ){
            if(dp[target]) return true; // 当target为 true的时候，直接的返回结果。
            // 只需要更新到 j == nums[i]就好，因为之前的已经不能将自己放进去。
            for(int j = target; nums[i] <= j ; j--){
                dp[j] = dp[j] || dp[j - nums[i]]; // 因为当前j的状态只和0-j-1的状态相关，所以就可以倒着进行判断，因为在更新的时候，前面的状态不会被破坏。
            }
        }
        return dp[target];
    }
}
```

## [494. 目标和](https://leetcode-cn.com/problems/target-sum/)

**题目大意**：给定一个非负整数数组，a1, a2, ..., an, 和一个目标数，S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。返回可以使最终数组和为目标数 S 的所有添加符号的方法数。

```shell
输入：nums: [1, 1, 1, 1, 1], S: 3
输出：5
解释：

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

一共有5种方法让最终目标和为3。
```

**解题思路**：这题和上面一题一样，虽然一样看过去不是01背包，但是本质上还是01背包，但是需要进行问题的转化。

假设将数组分内P和N，P为正数子集，N为负数子集，那么问题就转化为 sum(P) - sum(N) = target，又因为 sum(nums) = sum(P) + sum(N)

问题又可以转化为 ,两边同时加上 sum(nums) 变成如下形式。

sum(P) + sum(N)  +  sum(P) - sum(N) = target  +   sum(nums) 

即 2 * sum(P) =  target  +   sum(nums)  = 一个可以提前计算出来的数值。

此时转成了一个01背包问题，即找到数组nums中 不定个元素等于 1/2( target  +   sum(nums)  )。

代码如下，首先是二维数组的计算方式:

```java
public class 目标和494 {
    public static int findTargetSumWays(int[] nums, int S) {
        int sum = 0;
        for (int num : nums) {
            sum += num;
        }
        // 背包容量为整数，sum + S为奇数的话不满足要求
        if (((sum + S) & 1) == 1) {
            return 0;
        }
        // 目标和不可能大于总和
        if (S > sum) {
            return 0;
        }
        sum = (sum + S) >> 1;
        int len = nums.length;
        int[][] dp = new int[len + 1][sum + 1];
        dp[0][0] = 1;

        // 01背包
        // i(1 ~ len)表示遍历（不一定选）了 i 个元素，j(0 ~ sum) 表示它们的和
        for (int i = 1; i <= len; i++) {
            for (int j = 0; j <= sum; j++) {
                // 装不下（不选当前元素）
                if (j - nums[i-1] < 0) {
                    dp[i][j] = dp[i - 1][j];
                    // 可装可不装（当前元素可选可不选）
                } else {
                    dp[i][j] = dp[i - 1][j] + dp[i - 1][j - nums[i-1]]; // 为什么装或者是不装相加？ 因为这是找到所有的情况，那么既然装不装都能满足条件，
                    // 那么就需要将所有的情况都保存下来。就行。
                }
            }
        }

        return dp[len][sum];
    }

    public static void main(String[] args) {
        int [] nums = {1,1,1,1,1};
        System.out.println(findTargetSumWays(nums, 3));
    }
}
```

优化成一维数组：

```java
class Solution {
    public int findTargetSumWays(int[] nums, int S) {

        // 问题怎么就转换成了 sum(p) = (sum(nums) + S)/2  找出这个值就好。
        int len = nums.length;
        if(len == 0) return 0;
        int sum = 0;
        for(int ans:nums) // 统计值。
            sum += ans;
        if (((sum + S) & 1) == 1) { // 如果是奇数直接返回0 因为要除2 如果是个奇数/2 为小数，那么肯定找不到。
            return 0;
        }

        if(sum < S) return 0;

        int target = (sum + S) >> 1;

        int[] dp = new int[target + 1]; 
        dp[0] = 1;
        for(int i = 1; i <= len ; i++){
            for(int j = target;  nums[i-1] <= j; j--){
                dp[j] = dp[j] + dp[j - nums[i-1]]; // 优化成一维， 因为需要统计两个之间的关系。 所以将两种情况都加起来。
            }
        }
        return dp[target];
    }
}
```

本题与上一题不同之处在于，本题是找到有第i个元素和没有i个元素都要加起来，因为是找到所有的情况。而上一题只是单纯的判断是否有效而已。

另外这题也还进行了定义了len+1的dp数组。一般来说，dp[0] 都要是有效的。。

