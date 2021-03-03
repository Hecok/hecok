---
title: Talk Something about C++
author: NiKoJJ
date: 2021-02-17 14:00:00 +0800
categories: [Python,Django]
tags: [models]
---


### I.引用的概念
&emsp;&emsp;引用是C++中新引入的一个概念，其工作原理通常可以用指针来类比，但有不同于指针；其表现形式类似于变量，但在某些场合下又不同于变量。引用仅是一个名称而已，也可以成为别名。在建立引用时，总是让引用对应某一个目标，这个过程叫做**引用的初始化**，`经过初始化的引用就是目标的一个别名，对引用的操作实际上就是对目标的操作。`但是注意，引用不是变量不占用存储空间，而且被引用的目标必须先有定义，引用仅仅是别名而已，离开了目标它将没有意义。

```cpp
int i;  //定义变量
int &r=i; //建立引用
```
### II.引用的操作
&emsp;&emsp;由于引用只是变量的另一种符号化的表示，其和指针或变量都是不相同的，它不占有实际的存储空间，因此引用是没有地址的。那么能否对引用进行求地址的操作呢？回答是肯定的。`如果对引用求地址，将得到引用的目标的地址。`

```cpp
#include<iostream>
using namespace std;

int main()
{
    int i=3;
    int &r=i;  //r为变量的引用
    cout<<"the value of i :"<<i<<endl;
    cout<<"the value of r :"<<r<<endl;
    cout<<"the address of i :"<<&i<<endl;
    cout<<"the address of r :"<<&r<<endl;  //返回引用所指向目标变量的地址
    return 0;
}
结果：
the value of i :3
the value of r :3
the address of i :0x61fe1c
the address of r :0x61fe1c
```
&emsp;&emsp;观察上面的代码不难发现，引用的初始化有点儿类似赋值操作，但是它和赋值操作有本质的区别。引用的初始化是将引用维系在一个目标上，引用一旦被初始化，这个维系的关系就不再改变；而对初始化过的引用进行赋值操作，只是对引用的目标的赋值，而不是将引用转移到另一个目标上去，见如下代码：

```cpp
#include<iostream>
using namespace std;

int main()
{
    int *p;
    int num1,num2;
    int &r=num1;  //对num1的引用
    p=&num1;  //指针变量指向num1

    num1=2;
    num2=3;

    cout<<"the value of num1 :"<<num1<<endl;
    cout<<"the value of num2 :"<<num2<<endl;
    cout<<"指针变量p："<<p<<endl;  //输出num1的地址
    cout<<"求取引用的地址："<<&r<<endl;

    p=&num2;  //指针变量指向num2
    r=num2;  //对引用进行赋值操作,使得num1=num2

    cout<<"the value of num1 :"<<num1<<endl;
    cout<<"the value of num2 :"<<num2<<endl;
    cout<<"指针变量p:"<<p<<endl; //输出的则为num2的地址
    cout<<"求取对引用赋值后的地址："<<&r<<endl;  //输出仍是所指向目标num1的地址
    return 0;
}
结果：
the value of num1 :2
the value of num2 :3
指针变量p：0x61fe18
求取引用的地址：0x61fe18

the value of num1 :3
the value of num2 :3
指针变量p:0x61fe1c
求取对引用赋值后的地址：0x61fe18
```
&emsp;&emsp;为了方面比较地址上述代码引入了指针变量，可见，引用和指针有着很大的差别。指针更加灵活，它是一个变量，可以对它进行赋值操作，但是这一点对引用则不适用，同时通过上面对引用的赋值操作可以看到，`对初始化过的引用进行赋值操作只是对引用的目标的赋值，而不是将引用转移到另一个目标上去，`这一点与指针是截然不同的！
### III.利用引用传递参数
&emsp;&emsp;引用是可以传递参数的，传递引用给函数和传递指针给函数的效果是相同的，所传递的是实际参数本身，而不是作用域内建立的变量。现在我们用`传递变量形参swap1()`、`传递指针swap2()`、`传递引用swap3()`来编写swap()函数。

```c
#include<iostream>
#include<typeinfo>
using namespace std;

void swap1(int x,int y) //传递变量形参，那么这里的x y实际是mian()函数里a b的副本，所以对x y的操作都不能引起实际参数的变化
{//这里实际是值传递，只是将main()函数里的a b值赋值给swap1()函数里的x y,那么原本mian()函数里的的a b并不会改变
    int temp;
    temp = x;
    x = y;
    y = temp;

    cout << x << " " << y << endl;
}

void swap2_1(int *x,int *y)
{ //这里只是对形参地址进行了交换，对实际实参的值并没有影响(x=&a,y=&b)
    int *temp;
    temp = x;
    x = y;
    y = temp;
}

void swap2_2(int *x,int *y) 
{ 
 /*x y均为指针变量即地址，那么*x *y为x y各自地址所对应的值；
   值得注意的是在调用swap2()函数时——swap2(&x,&y)，直接传递的是main()函数里x y变量地址，那么在函数内部对形式参数的操作会体现到实际参数里；
   形参和实参指向相同的地址，导致随着形参的改变实参也会跟着改变，即被调用函数中对形参指针所指向的地址中内容的任何改变都会影响到实参。*/
    int temp;
    temp = *x;
    *x = *y;
    *y = temp;
}

void swap3(int &x,int &y)
{ //x y直接替代了a b ,即对a b本身进行操作
    int temp;
    temp = x;
    x = y;
    y = temp;
}

int main()
{
    int a = 2;
    int b = 3;
    cout << "原始地址：" << &a << " " << &b << endl;
    //swap1(a,b);  
    //swap2(&a,&b);  //调用函数
    swap3(a, b);
    cout << a << " " << b << endl;
    cout << "操作之后：" << &a << " " << &b << endl;
}
```
- **分析：**
&emsp;&emsp;那么我们来分析这几个函数,函数swap1()所传递的是变量形参，x、y是实际参数的一个副本，对x、y的所有操作都不能引起实际参数的变化，起不到交换的作用；函数swap2()和swap3()由于传递的都是实际参数的地址，所以在函数内部对形式参数的操作会体现在实际参数中。

- 传递指针与传递引用的比较：

 1. 传递指针的方式需要在函数体内反复使用求职运算符“*”，使得程序的编写和阅读都容易产生错误；
 2. 传递引用的方式具有与传递指针方式一样的效果，其程序的可读性往往比指针传递好，在函数体内就像传值方式一样简单、易读，可以达到一些传值方式无法达到的效果。
### IV.利用引用返回值
&emsp;&emsp;在讨论这个话题之前，我们不妨先来看两个函数，如下：
```cpp
int f1(int i)
{
    result=i+1;
    return result;
}

int &f2(int i)
{
    result=i+1;
    return result;
}
```
- **分析：**
&emsp;&emsp;对于一般的函数`f1()`而言，当一个函数返回一个值时，需要在函数的栈空间中生成一个临时副本，存放变量result的值，当函数返回给主函数时，会把临时变量中的值赋给所定义的变量得到结果。但是当函数通过引用返回时，则不会生成返回值的副本，当函数`f2()`返回main（）函数时，会直接将全局变量result的值赋给所定义的目标变量，从而使目标变量得到结果。
&emsp;&emsp;那么通过上述比较分析，可以明显发现通过值返回会有两次赋值过程，而通过引用返回时，由于不生成临时变量作为副本，也就省去了至少两次赋值操作的时间，可以明显提高程序执行效率和空间利用率。
### V.Ending... ...
&emsp;&emsp;以上内容仅仅是我对引用的一点点认知，有错误的地方还望给予指正。希望可以帮助到初学引用的同学。指针和引用使用比较灵活，有些地方也是不好理解的，一步一步理解就行辽，哈哈哈哈..... ...