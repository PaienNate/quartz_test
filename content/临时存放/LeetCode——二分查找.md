---
layout: post
date: 2022-02-12
updated: 2022-02-12
title: LeetCode-二分查找
tags: 
- LeetCode
category: 数据结构
---
# LeetCode——二分查找

2024-02-08注：二分查找作为数据结构经典内容，在考研中也有提及。饶是如此，长久不用也让我印象不太深刻了。

最近没啥心情打游戏，寻思去Leetcode练练手。

总结下来就是一句：人要勇敢的面对自己真的很废的事实（吐槽）

跟着点开了一个学算法的题目集，然后打开了二分查找：

## 二分查找的介绍

**复杂度分析**

- 时间复杂度：O(log n)，其中 n 是数组的长度。
- 空间复杂度：O(1)。

## 自己的思路（第一次）

思考了一下，二分查找就是取数组中间那个值，然后比较目标和中间的值，如果大，就把中间的值作为左值，如果小，就作为右值。

所以先定义三个常量：

```java
        int begin = 0;
        //结束下标
        int end = nums.length - 1;
        //中间值下标
        int middle = 0;
```

然后再定义一个total:

```java
int total = end - begin + 1;
```

这个total代表的就是剩下的那部分数组里面有多少个元素。

接下来就是取这个中间的值，还要注意数组下标和总数的关系。总数 = 数组最大下标 + 1.

当时琢磨了半天（脑补某傻子掰着手指头折腾了十来分钟），大概成型了一下：

数组里总数是total，获取中间的就是total/2,但是total分奇数和偶数两种可能，奇数好办，获取中间那个就行，偶数就要获取“靠右的那个”，这样下次还是奇数。

同时还要注意到，总数total除以的值和数组下标的关系（我经常倒腾不清这个）

| n (0) | n+1(1) | n+2(2) | n+3(3) | n+4(4) |
| ----- | ------ | ------ | ------ | ------ |
| n (1) | 2 (2)  | 3 (3)  | 4 (4)  | 5 (5)  |

以上面这个表为例，奇数情况下，获取中间的就是用总数total加1然后除以2，此时获得的是第三个元素（中间），在整体数组里，是从开头处向右挪移两个的那个元素。（begin + 3 - 1)

偶数情况下（去掉最后那一列），直接除以2，此时你在总数里拿到的是第二个元素，不过我想获取右侧的元素，所以给它加1，获取第三个元素，换算到数组下标里，还要减掉，就变成了：

(begin + 2 + 1 - 1)

也就是直接除以2就可以了。

于是乎就变成了这样子：

```java
        if(total%2 == 0)
        {
            //偶数情况下，中间值认定成靠右的那个，具体解释上面说了
            middle = begin + total/2;
        }
        else{
            //奇数情况下，中间值认定为中间那个，就是起始值 + (总数 - 1)/2，具体解释上面有
            middle = begin + (total- 1)/2;
        }
        if(target == nums[middle])
        {
            //如果相等，那下标就找到了
            return middle;
        }
        else if(target > nums[middle])
        {
            //目标大于，修改起始为middle
            begin = middle;
            //末尾不变，并重复该过程
        }
        else{
            //目标小于，修改末尾为middle
            end = middle;
        }
```

琢磨了一下应该没啥毛病，然后套上一个循环。现在的问题是判断不存在：

如果二分法分了半天还是不存在，那最后会变成两个元素，因为偶数个元素用取右边值的方法，如果是2个值，它永远都比较不了第一个。**（到现在复盘，我想或许它应该中间值认定为靠左的那个，或许这个问题就解决了）**

所以如果有两个的话，那就比较一下和第一个是否相等，如果不等，说明不存在，返回-1：

```java
        //当total为2时，还要比对它和左边的大小，如果左边都不等，则判断不存在
        if(total==2)
        {
            if(target == nums[middle - 1])
            {
                return middle -1;
            }
            else{
                return -1;
            }
        }
```

上面这个问题，显然，当元素只有一个的时候，也会发生这样的问题，同样判断：

```java
        if(total == 1)
        {
            if(nums[0] == target)
            {
                return 0;
            }
            else
            {
                return -1;
            }
        }
```

于是合计下来，代码写的乱的一批，变成了这么一个样子：

```java
class Solution {
    public int search(int[] nums, int target) {
        //开始下标
        int begin = 0;
        //结束下标
        int end = nums.length - 1;
        //中间值下标
        int middle = 0;
        //每次都计算区间内的总数
        while(true)
        {
        System.out.println("begin=" + begin + "end=" + end + "middle=" + middle);
        int total = end - begin + 1;
        if(total == 1)
        {
            if(nums[0] == target)
            {
                return 0;
            }
            else
            {
                return -1;
            }
        }
        System.out.println("total=" + total);
        //如果total没有了，说明二分到头了
        if(total%2 == 0)
        {
            //偶数情况下，中间值认定成靠右的那个，就是起始值 + 二分之一的总数
            middle = begin + total/2 ;
        }
        else{
            //奇数情况下，中间值认定为中间那个，就是起始值 + (总数 - 1)/2
            middle = begin + (total- 1)/2;
        }
        if(target == nums[middle])
        {
            return middle;
        }
        else if(target > nums[middle])
        {
            //目标大于，修改起始为middle
            begin = middle;
            //末尾不变，并重复该过程
        }
        else{
            //目标小于，修改末尾为middle
            end = middle;
        }
        //当total为2时，还要比对它和左边的大小，如果左边都不等，则判断不存在
        if(total==2)
        {
            if(target == nums[middle - 1])
            {
                return middle -1;
            }
            else{
                return -1;
            }
        }
        }
        
    }
}
```

最后通过用时8ms，可以说是极为的low了，超越了10%的用户。接下来分析一下题解是怎么干的：

## 正儿八经的题解思路

题解的参考代码长这样子：

```java
class Solution {
    public int search(int[] nums, int target) {
        int low = 0, high = nums.length - 1;
        while (low <= high) {
            int mid = (high - low) / 2 + low;
            int num = nums[mid];
            if (num == target) {
                return mid;
            } else if (num > target) {
                high = mid - 1;
            } else {
                low = mid + 1;
            }
        }
        return -1;
    }
}

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/binary-search/solution/er-fen-cha-zhao-by-leetcode-solution-f0xw/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

它用low和high表示了开始和结束。

### 0. 逻辑上的失误

逻辑上的失误有2：

1. 我当时脑子里转着“偶数的时候取右边的值”这件事，直接导致了最后整体合不拢（2个的时候取右边的值，无尽死循环，而明显还没有比较最左边的值，所以当时就应该将思路逆转过来，从‘为什么会导致2这个偶数特殊入手，将其改成取左边的值，就不会出现这样的死循环了）
2. 我直接将middle值作为开始/结束，实际上这个值已经不需要参与比较了，所以应该以middle-1/+1来作为新的起点/终点。不过从某种意义上说，如果当时把偶数取右边的值，再这样想的话，比较起来恐怕会很复杂，我估计我就更折腾不清了。一个逻辑失误，将会引发一堆失误啊。

### 1. 精妙的舍弃

我用了能否整除2来进行奇偶判断，而忽略了“地板除”的存在。在题解中，其使用了float换int的精度丢失，直接舍弃了小数点部分。这样就 不需要再判断几个if。

### 2. 统一比较单位

这里的单位指的是下标和总数。我一直致力于换算总数和下标，结果参考代码大笔一挥——换算干啥，反正只要走一套体系就不会出错：

于是乎使用(high-low)获取的就是下标之间的差值，然后除以2加上到开始的距离，就是中间的mid。之后让中间的值和target比较。去掉了一大堆奇怪的换算之后，代码变得简洁美观。

### 3.更完善的循环

我写的循环使用了死循环然后全部塞上返回值的方法。由于逻辑上的失误，循环更加没有办法想到像题解这样的方案。

题解的方案也很简单，如果你用中间值比较目标值，比我大（说明在mid左边）就把开始设置成mid位-1,比我小（说明在mid右边）就把结束设置成mid位+1（不需要再比较mid了，因为已经比较过），这样一路比较下去。

同时对两种极值的情况：

1 3（找2），比较到最后一次，中间值取1，比较一次小于目标值2，此时low要加1（代表1号元素），此时high和low都是1，再比较一次，high=mid-1，此时high变成0了，不满足循环，跳出。

1 3（找4），比较到最后一次，中间值取1，比较一次小于目标值4，此时low要加1（代表1号元素），此时high和low都是1，再比较一次，还是小于，low再加1，此时low变成2了，不满足循环，跳出。

这种方案比我的方案要巧妙非常多。

在力扣的评论里，同时强调了=的重要性：当数组只有一个值的时候：

5 找 5 如果第一次没有等于，直接就low=0 high=0 任意判定都等于-1了。

#### 小细节：

力扣作者labuladong提醒了一句：

```
另外声明一下，计算 mid 时需要防止溢出，
代码中 left + (right - left) / 2 就和 (left + right) / 2 的结果相同，
但是有效防止了 left 和 right 太大直接相加导致溢出。
链接：https://leetcode-cn.com/problems/binary-search/solution/er-fen-cha-zhao-xiang-jie-by-labuladong/
```

## 力扣示例题目

你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品的最新版本没有通过质量检测。由于每个版本都是基于之前的版本开发的，所以错误的版本之后的所有版本都是错的。

假设你有 n 个版本 [1, 2, ..., n]，你想找出导致之后所有版本出错的第一个错误的版本。

你可以通过调用 bool isBadVersion(version) 接口来判断版本号 version 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。


示例 1：

```java
输入：n = 5, bad = 4
输出：4
```

解释：

```java 
调用 isBadVersion(3) -> false 
调用 isBadVersion(5) -> true 
调用 isBadVersion(4) -> true
所以，4 是第一个错误的版本。
```

示例 2：

```java
输入：n = 1, bad = 1
输出：1
```


提示：

1 <= bad <= n <= 2(31次方） - 1

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/first-bad-version
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 解题

显然这就是让你用二分查找查找的，不过稍加变种，原本的二分查找是查到就行，现在换成了查到"左右两边缩小到一个点"。

于是乎摸出来如下代码：

```java
/* The isBadVersion API is defined in the parent class VersionControl.
      boolean isBadVersion(int version); */

public class Solution extends VersionControl {
    //摸一个适应的二分查找出来
    public int BinarySearch(int versionnum)
    {
        int begin = 0;
        //根据题目来看，它的要求里，versionnum不需要进行-1转换
        int end = versionnum - 1;
        while(begin<=end)
        {
            int middle = begin + (end-begin)/2;
             //判断的时候要转换成那个n的数值
            if(isBadVersion(middle + 1))
            {
                //从它后面的都是错的，判定前半部分即可
                end = middle - 1;
                //System.out.println("1begin = " + begin + "end = " + end);
            }
            else
            {
                //如果这里是对的，那错的一定在后面
                begin = middle + 1;//0 1 2 3 4
                //System.out.println("2begin = " + begin + "end = " + end);
            }
        }
        return begin + 1;
    }

    public int firstBadVersion(int n) {
        return BinarySearch(n);

    }
}
```

这个代码略微不完美的地方是，其实一开始begin和end用1和n就可以了。我机械化的写了0和n-1，其实完全没有任何必要。