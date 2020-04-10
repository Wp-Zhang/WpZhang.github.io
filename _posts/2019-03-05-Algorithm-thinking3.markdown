---
layout:     post
title:      求最大公约数和最小公倍数
subtitle:   "算法思考"
date:       2019-03-05 04:33:00
author:     "Wp.Zhang"
header-img: "img/post-bg-2015.jpg"
tags:
    - Algorithm
---
### 题目：求两数的最大公约数和最小公倍数

---

>关键在于求最大公约数，求出最大公约数后就可以算出最小公倍数



#### 方法一

&nbsp; &nbsp; &nbsp; &nbsp; 简单的暴力寻找：设p为两数m,n中的较小数，设置死循环：

1. 若p同时整除m和n，则p为m和n的最大公约数，退出循环；

2. 否则p-1，返回步骤1



##### C++代码：

```cpp

#include <iostream>



using namespace std;



int getGCD_method1(int m, int n){

	int p = m > n ? n : m;

	while (m % p != 0 || n % p != 0) p--;

	return p;

}



int main(){

	int m = 6, n = 9;

	cout << getGCD_method1(m, n) << endl;

}

```



#### 方法二

&nbsp; &nbsp; &nbsp; &nbsp; 辗转相除法：设置死循环(使用递归也可以)：

1. 取两数中的较大数为m，较小数为n；

2. 若m%n为0，则n为最大公约数，退出循环；

3. 将原来的两个数替换为n和m%n，返回步骤1



##### C++代码-循环

```cpp

#include <iostream>



using namespace std;



int getGCD_method2(int m, int n){

	while (1){

		if (m < n){

			m = m+n;

			n = m - n;

			m = m - n;

		}

		if (m % n == 0) return n;

		else{

			int tmp = m % n;

			m = n;

			n = tmp;

		}

	}

}



int main(){

	int m = 6, n = 9;

	cout << getGCD_method2(m, n) << endl;

}

```



##### C++代码-递归

```cpp

#include <iostream>



using namespace std;



int getGCD_method3(int m, int n){

	if (m < n){

		m = m+n;

		n = m - n;

		m = m - n;

	}

	if (m % n == 0) return n;

	else getGCD_method3(n, m % n);

}



int main(){

	int m = 6, n = 9;

	cout << getGCD_method3(m, n) << endl;

}

```



##### 两种方法直观比较：

![](/img/in-post/old/GOD&LCM_20190305140314184779.png)



### 求最小公倍数

##### C++代码：

```cpp

int getLCM(int m, int n){

	return m * n / getGCD_method2(m, n);

}

```