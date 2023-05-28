---
title:       "C++ move关键字"
subtitle:    ""
description: ""
date:        2023-05-27T20:30:39+08:00
author:      ""
image:       ""
tags:        ["cpp"]
categories:  ["Tech" ]
---

# C++  move关键字
**不能将一个右值引用直接绑定到一个左值上，但我们可以通过调用move新标准库函数来获取绑定到左值上的右值引用**
使用move函数可以返回指定对象的右值引用
```c++
int &&rr3 = std::move(rr1); //ok
```
move调用告诉编译器：我们有一个左值，但我们希望像一个右值一样处理它。   

**调用move就意味着承诺：除了对rr1赋值或销毁它外，我们将不再使用它。即不能对移后源对象(rr1)的值做任何假设**

### std::move是如何定义的
```c++
template <typename T>
typename remove_reference<T>::type&& move(T&& t)
{
    return static_cast<typename remove_reference<T>::type&&>(t);
} 
```
move的函数模板参数T&&是一个指向模板类型参数的右值引用。  
此参数可以与任何类型的实参匹配。特别是，我们即可以传递给move一个左值，也可以传递给move一个右值。
```c++
string s1("hi"), s2;
s2 = std::move(string("bye!")); //正确：从一个右值移动数据
s2 = std::move(s1); //正确：但在赋值之后，s1的值是不确定的，这也是为什么不应该使用移后源对象
```

### std::move是如何工作的
**在一个赋值中**,传递给move的实参是string的构造函数的右值结果------string("bye!")。  
所以move的工作过程是：
- 推断出T的类型是string
- 因此,remove_reference用string进行实例化
- remove_reference\<string>的type成员是string
- move的返回类型是string&&
- move的函数参数t的类型为string &&。
因此，这个调用实例化move\<string>，即函数
```c++
string&& move(string &&t)
``` 
函数体返回static_cast\<string&&>(t)。t的类型已经是string&&, 于是类型转换什么都不做。   

**第二个赋值中**，传递给move的实参是一个左值。  
所以move的工作过程是：
- 推断出T的类型是string&
- 因此，remove_reference用string&进行实例化
- remove_reference\<string&>的type成员是string
- move的返回类型仍是string&&
- move的函数参数t实例化为string& &&， 会折叠为string&,即
```c++
string&& move(string &t)
```
函数体返回static_cast\<string&&>(t)。在此情况下，t的类型为string&, cast将其转换为string&&。

