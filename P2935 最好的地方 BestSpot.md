这是即hdu2066后的第3题Floyd题目, 第二题因为过水而没什么特别要注意的地方于是就略过了. 这道题因为遇到了一些特别的情况于是来写一下

---

#### 题目描述 (此题为USACO的题目 原题是英文)
约翰拥有P(1<=P<=500)个牧场.贝茜特别喜欢其中的F个.所有的牧场 由C(1 < C<=8000)条双向路连接，第i路连接着ai,bi，需要1(1<=Ti< 892)单 位时间来通过.

作为一只总想优化自己生活方式的奶牛，贝茜喜欢自己某一天醒来，到达所有那F个她喜欢的 牧场的平均需时最小.那她前一天应该睡在哪个牧场呢？请帮助贝茜找到这个最佳牧场.

此可见，牧场10到所有贝茜喜欢的牧场的平均距离最小，为最佳牧场.　　

#### 输入输出格式
##### 输入格式
* Line 1: Three space-separated integers: P, F, and C
* Lines 2..F+1: Line i+2 contains a single integer: F_i
* Lines F+2..C+F+1: Line i+F+1 describes cowpath i with three space-separated integers: a_i, b_i, and T_i

##### 输出格式
* Line 1: A single line with a single integer that is the best pasture in which to sleep. If more than one pasture is best, choose the smallest one.

#### 输入输出样例
##### 输入样例#1：
13 6 15 
11 
13 
10 
12 
8 
1 
2 4 3 
7 11 3 
10 11 1 
4 13 3 
9 10 3 
2 3 2 
3 5 4 
5 9 2 
6 7 6 
5 6 1 
1 2 4 
4 5 3 
11 12 3 
6 10 1 
7 8 7 
##### 输出样例#1：
10

#### 样例的说明
            1*--[4]--2--[2]--3
                     |       |
                    [3]     [4]
                     |       |
                     4--[3]--5--[1]---6---[6]---7--[7]--8*
                     |       |        |         |
                    [3]     [2]      [1]       [3]
                     |       |        |         |
                    13*      9--[3]--10*--[1]--11*--[3]--12*


```
      * * * * * * Favorites * * * * * *
 Potential      Pasture Pasture Pasture Pasture Pasture Pasture     Average
Best Pasture       1       8      10      11      12      13        Distance
------------      --      --      --      --      --      --      -----------
    4              7      16       5       6       9       3      46/6 = 7.67
    5             10      13       2       3       6       6      40/6 = 6.67
    6             11      12       1       2       5       7      38/6 = 6.33
    7             16       7       4       3       6      12      48/6 = 8.00
    9             12      14       3       4       7       8      48/6 = 8.00
   10             12      11       0       1       4       8      36/6 = 6.00 ** BEST
   11             13      10       1       0       3       9      36/6 = 6.00
   12             16      13       4       3       0      12      48/6 = 8.00

Thus, presuming these choices were the best ones (a program would have to check all of them somehow), the best place to sleep is pasture 10.
```

这一题跟hdu2066不同的是, **在初始化的时候要对每个原点初始化成0而不是1(也就是坐标i==j的时候)**

为什么呢, 这道题是要求一个点 到所有喜欢的点的路径和的最小值. 

题目并没有说这个点不能是"喜欢的点", 因此这就会用到自己到自己的距离.(但这很明显是0啊).. 但是如果上面初始化如果不对原点进行特判初始化成0的话,进行Floyd算法的过程, 会让这些原点的路径改变(从INF变成一个莫名其妙的数)

拿输入数据中的一条边作为例子 10 11 1
在Floyd算法中, 当i为10,k为11,j为10的时候. 就会使得10到10的路径发生变化了(如果初始成INF)  

```
if(dp[10][11] + dp[11][10] < dp[10][10])
{
   dp[10][10] = dp[10][11] + dp[11][10];
}
```  
因为题目说明,是双向边 因此10到11和11到10路径都是1, 而dp[10][10]初始化成INF的话 就会大于左边的1 + 1, 因此dp[10][10]就变成了2 以至于影响到后面求最小值的
结果

至于后面的最小值计算, 求出每个点到某个点的最短路. 比较每个点到所有喜欢的点的路径和 选最小的即可.

完整代码
```c
#include <stdio.h>
#define Max 99999999
int read()
{
    char ch = getchar();
    int f = 1;
    int x = 0;
    while(ch < '0' || ch > '9'){if(ch == '-')f = 0;ch = getchar();}
    while(ch >= '0' && ch <= '9'){x = x * 10 + ch - '0';ch = getchar();}
    return f?x:x*-1;
}
int main()
{
    int p = read(),f = read(),c = read();
    int dp[501][501] = {};
    int fav[500] = {};
    
    for(int i = 1;i <= p;i ++)
    {
        for(int j = 1;j <= p;j ++)
        {
            if(i == j)
            {
                dp[i][j] = 0;
            }
            else
            {
                dp[i][j] = Max;    
            }
        }
    }
    for(int i = 0;i < f;i ++)
    {
        fav[i] = read();
    }
    
    for(int i = 0;i < c;i ++)
    {
        int a  = read(),b = read(),c = read();
        if(dp[a][b] > c)
        {
            dp[a][b] = c;
            dp[b][a] = c;
        }
    }
    
    for(int k = 1;k <= p;k ++)
    {
        for(int i = 1;i <= p;i ++)
        {
            if(i != k)
            {
                for(int j = 1;j <= p;j ++)
                {
                    if(dp[i][k] + dp[k][j] < dp[i][j])
                    {
                        dp[i][j] = dp[i][k] + dp[k][j];
                        dp[j][i] = dp[i][j];
                    }
                }
            }
        }
    }
    
    double min = 99999;
    int ans = -1;
    for(int i = 1;i <= p;i ++)
    {
        double count = 0;
        for(int j = 0;j < f;j ++)
        {
            count += dp[i][fav[j]];
        }
        count /= f;
        if(count < min)
        {
            min = count;
            ans = i;
        }
    }
    printf("%d",ans);

    return 0;
}
```

这道题又让我对Floyd题目稍微涨了涨姿势>.< 初始化的时候并不是什么时候都能无脑最大值