## 加法运算

对两个输入参数做加法运算，但是不能使用 “+” 运算符 。



### 解法思路

看到这样的问题，能想到的只有位运算，问题是怎么算？相信大家小学刚学习加法的时候，对于一下子不能得到答案的题，肯定会在草稿纸上列竖式，从右向左算，同一列对下来的数字相加如果超过 10，那么肯定要在下面写两个数字相加后的个位数，然后往前进一位，下一位运算时就要加上这个进位，用这样的方式直到最后算出结果。这题的思路也是一样的，只不过有两点不一样，第一，10 进制变成了 2 进制，第二，我们不再是在草稿纸上列竖式，而是要写成计算机看得懂的代码，这就得借助我们的位运算了，因为 2 进制表示的数中只会出现 0 和 1，你可以把这两个数看成是 true 和 false，这样更好理解，我们可以先通过异或塞选出不用进位的情况，然后再用与运算和左移运算计算出进位的情况，迭代更新出最后的结果。

### 参考代码

```
public int plus(int a, int b) {
    int aTemp = 0, bTemp = 0;
    while (b != 0) {
        aTemp = a ^ b;
        bTemp = (a & b) << 1;
        a = aTemp;
        b = bTemp;
    }

    return a;
}
```



## 交换两数



如何在不创建临时变量的情况下进行交换两个数？

### 解法思路

异或的常见应用，很简单，但是注意思考角度从位出发，而不是数，这点很重要。

### 参考代码

```
public void swap(int a, int b) {
    a ^= b; // a 中存放两数互异的点位
    b ^= a; // 取反 b 中不同于 a 的点位，也就是实现了 b = a
    a ^= b; // 取反 a 中不同于 b 的点位，也就是实现了 a = b
}
```



## 数A 转换成 数B

如果把 A 转换成 B ，需要改变多少位？

### 解法思路

异或的简单应用，两个数做异或的结果就是两个数差异所在，然后只需计算这个结果中有多少个 1 即可。

### 参考代码

```
public int convertA2B(int A, int B) {
    int n = A ^ B;
    int count = 0;
    while (n != 0) {
        n = n & (n - 1); // n - 1 是将 n 的最低位为零
        count++;
    }

    return count;
}
```



## 只出现一次的数

LeetCode 第 136 号问题：给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

### 解法思路

异或的三个点顺下来，就可以很清楚地解这道题：

异或运算和乘法一样，位置和运算顺序不影响最后结果：`a^b^c = b^c^a` 
两个相同的数做异或运算结果为零：`a^a = 0` 
任何数和零做异或结果还是这个数本身：`a^0 = a`

### 参考代码

```
public int findOnce(int[] arr) {
    int result = 0;
    for (int i = 0; i < arr.length; ++i) {
        result ^= arr[i];
    }

    return result;
}
```



## 只出现一次的数2

LeetCode 第 137  号问题：给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现了三次。找出那个只出现了一次的元素。

### 解法思路

这题的难点在于 3 次，如果把数组里面的数字就当作数字本身来看的话，很难找到突破口；如果想到了位运算，那就要有一个概念就是位运算是基于位的，而不是基于数的，在这个问题中，所有的 bit 的出现次数只会有两种情况，`3*n，3*n + 1`,这里的 n 是任意整数，假设你遍历数组，其实会有一个中间态就是 `3*n + 2` 存在，对于除这个数以外的其他数，过程大概是 `3*n + 1` -> `3*n + 2` -> `3*n`，我们只要记录的就是 `3*n + 1` 和 `3*n + 2`的情况，因为 `3*n` 其实就是一个初始状态（n=0）,记不记录和我们最后要返回的答案无关，而记录 `3*n + 2` 是为了恢复一些 bit 从 `3*n + 2` 到 `3*n`

### 参考代码

```
public int singleNumber(int[] nums) {
    int ones = 0, twos = 0;
    for (int i = 0; i < nums.length; ++i) {
        // 找出当前的 3n + 1
        ones = (nums[i] ^ ones) & (~twos);
        // 找出当前的 3n + 2
        twos = (nums[i] ^ twos) & (~ones);
    }

    return ones;
}
```



## 只出现一次的两个数



LeetCode 第 260  号问题：给定一个整数数组 `nums`，其中恰好有两个元素只出现一次，其余所有元素均出现两次。找出只出现一次的那两个元素。

### 解法思路

这题的主要难点是如何把两个数给拆出来，如果直接运用异或算法，我们最后得到的结果是两个数做异或的结果，关键点是如何基于这个异或的结果来找到这两个数，有一点很重要的就是，异或的结果为 1 的点位只会出现在其中一个数中，我们可以用其中一个为 1 的点位作为判断依据，这个点位存在的所有数在一起做异或，这个点位不存在的所有数一起做异或，这样就把这个问题拆解成了两个 problem 3。

### 参考代码

```
public int[] singleNumber(int[] nums) {
    if (nums == null || nums.length == 0) {
        return new int[2];
    }

    int different = 0;
    for (int i : nums) {
        different ^= i;
    }

    different &= -different;
    int[] ans = {0, 0};
    for (int i : nums) {
        if ((different & i) == 0) {
            ans[0] ^= i;
        } else {
            ans[1] ^= i;
        }
    }

    return ans;
}
```