---
title:       "虚继承"
subtitle:    ""
description: ""
date:        2023-06-12T20:14:08+08:00 
author:      ""
image:       ""
tags:        ["cpp"]
categories:  ["Tech" ]
katex:       false
---

## 菱形继承
虚继承的引入是因为出现菱形继承的时候调用重复部分的数据会引起二义性

贴个代码：
```c++
#include <iostream> 
#include <iomanip>

using namespace std;

class A{
public:
        virtual void foo() { cout << "A" << endl; }
        int m_a;
        int m_aa;
};

class B :public A{
public:
        virtual void foo() { cout << "B" << endl; }
        int m_b;
};

class C :public A{
public:
        virtual void foo() { cout << "C" << endl; }
        int m_c;
};

class D :public B, public C{
public:
        virtual void foo() { cout << "D" << endl; }
        int m_d;
};

void print_mem(void* c, unsigned len) {
    for (unsigned i = 0; i < len; i++) {
        if (i % 8 != 0)
            cout << ' ';

        cout << setfill('0') << setw(2) << hex << (unsigned)*((unsigned char*)(c) + i);

        if (i % 8 == 7 && i != len - 1) {
            cout << endl;
        }
    }
    cout << endl;
}

int main(){
        D d;

        d.B::m_a = 0x0b0a;
        d.B::m_aa = 0x0baa;
        d.C::m_a = 0x0c0a;
        d.C::m_aa = 0x0caa;
        d.m_b = 0x0b;
        d.m_c = 0x0c;
        d.m_d = 0x0d;

        print_mem(&d, sizeof(d));
        return 0;
}
```

此时输出结果为：
![没有虚继承的情况下](/img/virtual_inherit1.jpg)
来分析一下他们的内存布局
（注意内存对齐）
![内存布局图](/img/nonvirtural.jpg)
可以发现，当D继承B,C时，d对象中就会有两个A的副本，那么如果d调用m_a时，就会造成二义性，这就是「菱形继承」问题

那么怎么解决呢----「虚继承」

## 虚继承

我们使用 B 和 C 分别虚继承 A， 看一下代码：
```c++
#include <iostream> 
#include <iomanip>

using namespace std;

class A{
public:
        virtual void foo() { cout << "A" << endl; }
        int m_a;
        int m_aa;
};

class B :virtual public A{
public:
        virtual void foo() { cout << "B" << endl; }
        int m_b;
};

class C :virtual public A{
public:
        virtual void foo() { cout << "C" << endl; }
        int m_c;
};

class D :public B, public C{
public:
        virtual void foo() { cout << "D" << endl; }
        int m_d;
};

void print_mem(void* c, unsigned len) {
    for (unsigned i = 0; i < len; i++) {
        if (i % 8 != 0)
            cout << ' ';

        cout << setfill('0') << setw(2) << hex << (unsigned)*((unsigned char*)(c) + i);

        if (i % 8 == 7 && i != len - 1) {
            cout << endl;
        }
    }
    cout << endl;
}

int main(){
        D d;

        d.B::m_a = 0x0b0a;
        d.B::m_aa = 0x0baa;
        d.C::m_a = 0x0c0a;
        d.C::m_aa = 0x0caa;
        d.m_b = 0x0b;
        d.m_c = 0x0c;
        d.m_d = 0x0d;

        print_mem(&d, sizeof(d));
        return 0;
}
```
运行结果如下：
![有虚继承的情况下](/img/virtual_inherit3.jpg)
可以注意到此时d对象的内存布局发生了改变
在上面B和C的虚函数指针下没有了A的数据， 但是在最后又多了一个指针，并且A的数据放在了最后  

同时也注意到对象b和对象c中原本a的数据，也变成了虚函数指针 + a的数据

那么此时的内存布局是这样:
![有虚继承的情况下](/img/virtual.jpg)