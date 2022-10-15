---
title:       "cpp_compare"
subtitle:    ""
description: ""
date:        2022-10-14
author:      ""
image:       ""
tags:        ["cpp_note"]
categories:  ["" ]
---

# 自定义cmp函数

### c++ 默认使用operator<()来进行排序  

```c++
    class Student{
        int score;

        bool operator<()(int const& stu) const{
            return this->score < stu.score;
        }
    }

    /*或者是*/
    //修正 这么写不太必要
    // struct cmp{
    //     bool operator()(int const& a, int const& b){
    //         //可以粗浅理解为 按照 a在前 b在后的序列进行排序
    //         //return a < b 就代表小的在前
    //         return a < b;
    //     }
    // }

    bool operator()(int const& a, int const& b){
        //可以粗浅理解为 按照 a在前 b在后的序列进行排序
        //return a < b 就代表小的在前
        return a < b;
    }

```

### 1.我们可以通过重写operator<()来进行自定义排序  

比如在写一个自定义结构时，我们可以将它的operator<进行重写
```c++
    class State{
    public:
        int id;
        double prob_from_start;
        State() : State(0,0.0){};

        State(int id,double prob_from_start){
            this->id = id;
            this->prob_from_start = prob_from_start;
        }

        //参数外面的const修饰整个函数 ： this-> 值 不能修改
        bool operator<(State const& a) const{
            //参数里面的const 修饰State：State值不能修改
            return this->prob_from_start > a.prob_from_start;
        }
    };

```

### 2.自定义比较函数
```c++
class Solution{
    class State{
    public:
        int id;
        double prob_from_start;
        State() : State(0,0.0){};

        State(int id,double prob_from_start){
            this->id = id;
            this->prob_from_start = prob_from_start;
        }
    };

    //注意这里比较函数我们放在class内 需要static  如果放在class外 就不需要static
    static bool operator(State const& a, State const& b) {
        //参数里面的const 修饰State：State值不能修改
        return a.prob_from_start > b.prob_from_start;
    }
}
```

##### 为什么放在class内需要static，否则不需要？

因为sort传入的是一个函数指针，只关心传入的参数量

放在class内，这个函数就属于class，它就会默认有一个this指针，一共有三个参数

而传到sort里面时 只传了两个形参，没有传this  

把函数加上一个static后，因为static是没有this指针的 所以就不需要传入this

而把它放在class外后，这个函数就只是一个单独的函数  就不会默认有this指针，一共有两个参数

> 注意：因为static没有this指针，所以static不可以调成员函数，成员函数能调static
### 3.声明比较类 

#### 类似于priority_queue这样的结构，只能传入类型，所以只能使用声明比较类

```c++
    struct Edge{
        int p1;
        int p2;
        int manhattan;
        Edge(int p1, int p2,int manhattan): p1(p1),p2(p2),manhattan(manhattan){

        }
    };
    // 按照边的权重从小到大排序
    struct cmp{
        bool operator()(Edge& a,Edge& b){
            return a.manhattan > b.manhattan;
        }
    };

    // 核心数据结构，存储「横切边」的优先级队列
    priority_queue<Edge, vector<Edge>, cmp> pq;
```

### 注意事项 

> 注意：
> greater表示从大到小排序，less表示从小到大排序
> 但是priority_queue中是相反的
