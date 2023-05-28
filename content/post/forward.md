---
title:       "C++ forward关键字"
subtitle:    ""
description: ""
date:        2023-05-27T16:46:33+08:00
author:      ""
image:       ""
tags:        ["cpp"]
categories:  ["Tech" ]
---

# C++ forward关键字
**forward的作用:当某函数需要将其一个或多个实参连同类型不变地转发给其他函数时，需要保持被转发实参的所有性质，包括实参类型是否是const的以及实参数是左值还是右值**  

### 引用参数
编写一个函数如下
```c++
template <typename F, typename T1, typename T2>
void filp1(F f, T1, T2)
{
    f(t2, t1);
}
```  
这个函数一般情况下可以正常工作，但当我们希望用它调用一个接受引用参数的函数时就会出现问题：
```c++
void f(int v1, int& v2)
{
    cout << v1 << " " << ++v2 << endl;
}
```  
此时使用
```c++
f(42,i)
```
能够正常改变i的值
但当调用filp1时则无效
```c++
flip(f, j, 42); //通过filp1调用f不会改变j
```  
因为\<j,42>传入模版参数后 \<T1,T2> 分别被推导为 \<int, int>，即模板参数被实例化为
```c++
void flip(void(*fcn)(int, int&), int t1, int t2);
```
故j的值被拷贝到t1中，f中的引用参数被绑定到t1，而非j，从而其改变不会影响j

### 定义能保持类型信息的函数参数  
重写函数，使其参数能够保持给定参数的“左值性”，同时也希望保持参数的const属性  
**通过将一个函数参数定义为一个指向模板类型参数的右值引用，我们可以保持其对应实参数的所有类型信息。**  
而使用引用参数（无论是左值还是右值）使得我们可以保持const属性，因为在引用类型中的const是底层的  
我们将函数参数定义为T1&& 和T2&&通过引用折叠就可以保持返转实参的左值/右值属性
```c++
template <typename F, typename T1, typename T2>
void flip2(F t, T1 &&t1, T2 &&t2)
{
    f(t2, t1); 
}
```
此时如果调用flip2(f,j,42)，模版参数就会被实例化为
```c++
void flip2(void(*fcn)(int, int&), int&, int)
```
>> 如果一个函数参数是指向模板类型参数的右值引用(T &&)， 它对应的实参的const属性和左值/右值属性将得到保持

### 右值引用
flip2能够很好的解决左值引用的问题，但不能接受右值引用参数的函数
```c++
void g(int &&i,int& j)
{
    cout << i << " " << j << endl;
}
```
如果我们试图用flip2调用g， 则参数t2会被传递给g的右值引用参数
```c++
flip2(g, i, 42); //错误：不能从一个左值实例化int&&
```
因为这样调用时，模板参数实例化仍为
```c++
void flip2(void(*fcn)(int, int&), int&, int)
```
**调用std::forward保持类型信息**
我们可以使用一个名叫forward的新标准库设施来传递flip2的参数，它能保持原始参数实参的类型。  
forward必须通过显式实参来调用，forward返回该显式实参类型的右值引用。即forward<T>的返回类型是T&&。  

通常我们使用forward传递那些定义为模板类型参数的右值引用的函数参数，通过返回类型上的引用折叠，forward可以保持给定实参的左值/右值属性：
```c++
template <typename Type> intermediary(Type &&arg)
{
    finalFcn(std::forward<Type>(arg));
}
```
**forward传递**
上述函数中使用Type作为forward的显式模板实参数类型，它是从arg推断出来的  
如果arg是一个模板类型参数的右值引用(Type &&)， Type将表示传递给arg的实参的所有类型信息。  
如果arg是一个右值(如42），则Type是一个普通（非引用)类型(int),forward\<Type>将返回Type &&(int&&)。   
如果arg是一个左值(int), 则通过引用折叠, Type本身是一个左值引用类型\<int &>，forward返回Type && (int& &&)， 再通过折叠，仍然返回左值引用类型int &。  
则翻转函数可以写成
```c++
template <typename F, typename T1, typename T2>
void flip(F f, T1 &&t1, T2 &&t2)
{
    f(std::forward<T2>(t2), std::forward<T1>(t1));
}