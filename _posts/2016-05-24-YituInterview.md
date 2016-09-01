---
title: "依图科技面试"
tags: [面试,算法]
---

昨天去依图科技面试，特此总结。

## 一面

### 判断围棋棋盘合法性

一面的第一道也是唯一一道题目...

先普及一发关于棋子的气的围棋规则（摘自百度百科）：

> **棋子的气**  
一个棋子在棋盘上，与它直线紧邻的空点是这个棋子的“气”。
直线紧邻的点上如果有同色棋子存在，这些棋子就相互连接成一个不可分割的整体。
直线紧邻的点上如果有异色棋子存在，此处的气便不存在。棋子如失去所有的气，就不能在棋盘上存在。

举个例子，比如下图中红点所在位置即为黑棋的气，若这些位置均被白棋占领，则黑棋失去所有的气，不能存在。存在不应该存在的棋的棋盘将被判定为不合法，否则棋盘合法。  


![](http://i.imgur.com/B98iehJ.jpg)  


输入：

{% highlight c++ %}
vector<string> board,int m,int n
//分别表示棋盘，棋盘长，棋盘宽，其中'o'表示白棋，'x'表示黑棋，'.'表示没有棋
{% endhighlight %}

输出：

	bool值表示棋盘是否合法

**思路：**

搜索整个棋盘，对每个点进行搜索（广搜/深搜），如是黑棋，则沿着上下左右搜索到气的存在，则这片棋子为合法的，接下去搜索。

## 二面

### 斐波那契数列第n项的O(logn)求法（写出完整代码）

斐波那契数列第n项可用矩阵幂运算求解：

![](http://i.imgur.com/oqMjotV.png)  

矩阵的n次方可在O(logn)时间内完成.

具体代码如下(摘自网络):

{% highlight c++ %}
#include<iostream>  
#include<string>  
using namespace std;  

//定义2×2矩阵；  
struct Matrix2by2  
{  
  //构造函数  
	Matrix2by2  
  (  
    long m_00,  
    long m_01,  
    long m_10,  
    long m_11  
  )  
  :m00(m_00),m01(m_01),m10(m_10),m11(m_11)
  {
  }

  //数据成员
  long m00;
  long m01;
  long m10;
  long m11;
};

//定义2×2矩阵的乘法运算
Matrix2by2 MatrixMultiply(const Matrix2by2& matrix1,const Matrix2by2& matrix2)
{
  Matrix2by2 matrix12(1,1,1,0);
  matrix12.m00 = matrix1.m00 * matrix2.m00 + matrix1.m01 * matrix2.m10;
  matrix12.m01 = matrix1.m00 * matrix2.m01 + matrix1.m01 * matrix2.m11;
  matrix12.m10 = matrix1.m10 * matrix2.m00 + matrix1.m11 * matrix2.m10;
  matrix12.m11 = matrix1.m10 * matrix2.m01 + matrix1.m11 * matrix2.m11;
  return matrix12;
}


//定义2×2矩阵的幂运算
Matrix2by2 MatrixPower(unsigned int n)
{
  Matrix2by2 matrix(1,1,1,0);
  if(n == 1)
  {
    matrix = Matrix2by2(1,1,1,0);
  }
  else if(n % 2 == 0)
  {
    matrix = MatrixPower(n / 2);
    matrix = MatrixMultiply(matrix, matrix);
  }
  else if(n % 2 == 1)
  {
    matrix = MatrixPower((n-1) / 2);
    matrix = MatrixMultiply(matrix, matrix);
    matrix = MatrixMultiply(matrix, Matrix2by2(1,1,1,0));
  }
  return matrix;
}

//计算Fibnacci的第n项
long Fibonacci(unsigned int n)
{
  if(n == 0)
    return 0;
  if(n == 1)
	  return 1;

	Matrix2by2 fibMatrix = MatrixPower(n-1);
	return fibMatrix.m00;
}

int main()
{
  cout<<"Enter A Number:"<<endl;
  unsigned int number;
  cin>>number;
  cout<<Fibonacci(number)<<endl;
  return 0;
}
{% endhighlight %}

### 数组循环移位  

长度为size的数组，循环右移k位，要求时间复杂度O(n),空间复杂度O(1),讲出思路即可.

**方法1**

首先reverse整个数组，然后将左边k位reverse，右边n-k位reverse，结束.

**方法2**

求size与k的最大公约数，记为m,令i=size/m.

1. 从第1位开始，将第1位元素覆盖掉第(1+k)%size位元素，第(1+k)%size位元素覆盖掉第(1+2*k)%size位元素，做i次覆盖，恰好为1个循环
2. 从第2位开始，重复以上步骤...
3. ...
3. 从第m位开始，重复以上步骤.

结束.

## 三面

三面为技术总监周健，主要是聊一下价值观啊知识储备什么的，过程很轻松。


## 总结

依图科技作为一家很有实力的创业公司，在实习生面试的把关也算是相对严格，面试基本上就是考察算法掌握以及编程能力，题目难度比起国内大公司如BAT还是难挺多的，不过考察的知识也相对集中，容易复习和做准备。

公司现在规模还相对较小，只有五六十人的技术团队，而且团队成员也都比较年轻，但是整个团队的未来还是很光明的。
