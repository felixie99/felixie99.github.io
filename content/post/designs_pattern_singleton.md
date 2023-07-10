---
title:       "设计模式--单例模式"
subtitle:    ""
description: ""
date:        2023-07-10T15:39:58+08:00 
author:      ""
image:       ""
tags:        ["cpp"]
categories:  ["Tech" ]
katex:       false
---

当有些对象我们只需要一个的时候，比如线程池，日志系统，连接池等，此时我们就可以用单例模式  

单例模式就是确保一个类在任何情况下都只有一个实例，并提供一个全局访问点  

# 懒汉式单例
顾名思义，非常懒，不用的时候不去初始化，在第一次使用时才进行初始化  

```c++
class lazySingle{
private:
    lazySingle(){}
    ~lazySingle(){}

public:
    static lazySingle& getinstance(){
        static lazySingle obj;
        return &obj;
    };

    lazySingle(lazySingle& obj) = delete;
    lazySingle* opreator=(lazySingle& obj) = delete;
};
```

# 饿汉式单例
即迫不及待，在程序运行时立即初始化
```c++
class eagerSingle{
private:
    eagerSingle(){}
    ~eagerSingle(){}
    static eagerSingle instance;
public:
    static eagerSingle& getinstance(){
        return &instance;
    }

};
```
## 优点
- 饿汉式单例是最简单的一种单例形式，没有任何的锁，执行效率最高
- 线程安全

## 缺点
某些情况下，造成内存浪费，因为对象未被使用的情况下就会被初始化，如果一个项目中的类多达上千个，在项目启动的时候便开始初始化可能并不是我们想要的。

### 参考链接
[单例模式的原理及实现](https://www.bilibili.com/video/BV1Gz4y1d7RJ/?spm_id_from=333.880.my_history.page.click&vd_source=6231c8e862cfb02f2c39776bd6c364a7)