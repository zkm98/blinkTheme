title: 贝塞尔曲线的学习一：基本n阶贝塞尔曲线和求导
author: zkm98
categories: 算法
tags:
  - c++
  - 算法
date: 2021-06-02 10:26:00

---
## 简介
    本文主要是从贝塞尔曲线的定义和求导下手进行简单贝塞尔曲线的学习和整理。

## 1.基本定义

    贝塞尔曲线是参数化曲线（Parametric Curves）的一种，其n阶次曲线具有如下的形式：

$$C(t) = \sum_{i=0}^{n}B_{i,n}(t)p_i$$
$$t\in[0,1]$$

    所以写成矩阵的形式有：

$$C(t) =[B_{0,n}(t),B_{1,n}(t),...,B_{n,n}(t)][P_0^T,P_1^T,...,P_n^T]^T$$

    其中出现了一个名为贝塞尔曲线参数的符号：
    $$B_{i,n}(t))$$
    我们将他定义为

$$B_{i,n}(t)=\frac{n!}{i!(n-i)!}(1-t)^{n-i}t^i$$

    其中其实大部分的内容，我们只需要使用一个n的向量计算出就好。

```cpp
/*vector<int> factorial*/
Point at(double t){
    Point ans={0,0};
    for(int i=0;i<=n;i++){
        ans += factorial[n]/factorial[i]/factorial[n-i]*mypow(1-t,n-i)*mypow(t,i);
    }
    return ans;
}
```



## 2. 求导

    因为大部分的时候，我们只需要使用一阶导数和二阶导数，这里只去简单去推导其中两个内容。也是为我们去工业使用时候，比较减少计算量的一个方法。

#### 2.1 一阶导数

    根据1中内容，我们已知其中对t求导时，应该考虑的主要在贝塞尔曲线参数的t求导。所以我们可以得到下面的求导过程：

$$\frac{\partial{B_{i,n}(t)}}{\partial{t}}=\frac{n!}{i!(n-i)!}(i(1-t)^{n-i}t^{i-1}-(n-i)(1-t)^{n-i-1}t^i)$$

    由于在头部和尾部就仅仅需要其中一个部分，我们可以有以下的实现：

```cpp
/*vector<int> factorial*/
Point dev(double t){
    Point ans={0,0};
    for(int i=0;i<=n;i++){
        ans += factorial[n]/factorial[i]/factorial[n-i]*(
            i*mypow(1-t,n-i)*mypow(t,i-1)-
            (n-i)*mypow(1-t,n-i-1)mypow(t,i));
    }
    return ans;
}
```

#### 2.2 二阶导数

    二阶导数是基于一阶导数的二次实现，基本上的求导过程和上述内容差不多，也是对贝塞尔曲线参数的求导：

$$\frac{\partial ^2{B_{i,n}(t)}}{\partial{t^2}}=\frac{\partial \frac{n!}{i!(n-i)!}(i(1-t)^{n-i}t^{i-1}-(n-i)(1-t)^{n-i-1}t^i)}{\partial t} $$

$$\frac{\partial ^2{B_{i,n}(t)}}{\partial{t^2}}=\frac{n!}{i!(n-i)!}(i(i-1)(1-t)^{n-i}t^{i-2}-2i(n-i)(1-t)^{n-i-1}t^{i-1}+(n-i)(n-i-1)(1-t)^{n-i-2}t^i)$$

    基于上述的公式，我们再次更新我们的代码为：

```cpp
/*vector<int> factorial*/
Point dev2(double t){
    Point ans={0,0};
    for(int i=0;i<=n;i++){
        ans += factorial[n]/factorial[i]/factorial[n-i]*(
              i*(i-1)*mypow(1-t,n-i)*mypow(t,i-2)
             -2*i*(n-i)*mypow(1-t,n-i-1)mypow(t,i-1)
        	 +(n-i)*(n-i-1)mypow(1-t,n-i-2)*mypow(t,i));
    }
    return ans;
}
```



## 总结

    上述内容主要是介绍了如何去简单的运算贝塞尔曲线，以及它的一阶导数和二阶导数。