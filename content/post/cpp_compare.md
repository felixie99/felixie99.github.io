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

    struct cmp{
        bool operator()(int const& a, int const& b){
            //可以粗浅理解为 按照 a在前 b在后的序列进行排序
            //return a < b 就代表小的在前
            return a < b;
        }
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
### 2.使用struct cmp来代替operator<

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