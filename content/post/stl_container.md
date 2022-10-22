---
title:       "stl_container"
subtitle:    ""
description: ""
date:        2022-10-22
author:      ""
image:       ""
tags:        ["cpp_note"]
categories:  ["" ]
---

# STL容器

## 顺序容器



## 关联容器

![示意图](/img/stl_container_1.png)
关联容器步支持顺序容器的位置相关的操作，例如push_front,push_back.  

关联容器也不支持构造函数或插入操作这些接受一个元素值和一个数量值的操作



### 标准的STL关联式容器

#### 1.使用map

map是key-value的集合，所有元素都是pair，有序。不允许重复的键值，multimap允许重复的键值

应用场景：  

1.将名字映射到电话号码  

2.将单词与单词的个数关联

#### 2.使用set

set的key = value。有序，不允许重复的键值，multiset允许重复的键值。

应用场景：

当只是想知道一个值是否存在时，set是最有用的。

> set和map均为有序容器，缺省条件下，按照键值从小到大排序。  

### 3.multimap、multiset  
容器multimap和multiset允许多个元素拥有相同的关键字  

### 非标准的STL关联式容器


## STL容器的插入、删除、遍历和查找操作性能对比    
没有讨论stack、queue和priority_queue，是因为它们底层是使用deque或者vector实现的。
### 插入  
[转载自breaksoftware的csdn博客](https://blog.csdn.net/breaksoftware/article/details/82947838)  
#### 头部插入
结论：  
- deque的性能是最好的，其次是unordered_set（神一般的存在）。

- 当元素个数比较多时，vector的性能是最差的。

- forward_list要优于list。

- 除了unordered_set，其他关联容器性能都比非关联容器（除了vector）要差。

- 当元素个数比较多（大概大于2500）时，set的性能要高于map。multi系列的都要优于非multi系列（除了unordered_set）。比如multimap优于map；unordered_multimap优于unordered_map。

- 当元素个数比较少（大概小于256）时，有序关联容器的性能比无序（unordered）关联容器要高（除了unordered_set）。
### 中间插入  
结论：  
- 当元素个数小于256时，vector的效率是最高的。但是随着元素个数增多，vector将变成性能最差的容器。

- forward_list、list和deque在不同元素个数时表现的都很优异。

- set容器是所有关联容器中性能最好的。

#### 尾部插入
结论：  
- 在尾部插入时，vector的性能是最好的。其他两个场景下，vector的性能都是最差的。但是在中间插入场景，容器元素个数小于256时，vector还是最优的。但是之后衰退严重。

- deque在头部和尾部插入元素场景下性能优异。

- list和forward_list在中间插入元素场景下性能优异。

- 在关联容器中，只有在头部插入场景下的unordered_set性能极其优异。

- 当元素个数较多时，set的性能要优于map。

### 删除  
[转载自breaksoftware的csdn博客](https://blog.csdn.net/breaksoftware/article/details/82948224) 
#### 头部删除  
结论：
- vector的性能始终最差
- 除了vector，非关联容器性能都优于关联容器
- 除了vector，set和map的性能最差  

#### 中间删除  
结论：
- vector效率持续糟糕
- list和forward_list性能最优
- deque和其他关联容器效率相似，比较低效  

#### 尾部删除  
结论：
- vector表现最优，其次是deque和list
- map的性能要优于set
- set在元素个数超过3000左右后，效率仅优于forward_list

#### 最终结论
- vector只有在尾部删除时性能最优。在头部和中间删除时，性能始终是最差的。

- forward_list在头部和中间删除时，性能是非常好的。但是在尾部删除时，性能极其差。

- 中间删除时，性能最高的是list和forward_list。deque在这个场景下表现很平庸，和其他关联容器差不多。

- 头部和尾部删除时，deque性能非常优异。

### 遍历

#### 从前往后  
结论：  

- 元素个数大于8000左右时，vector效率是最好的。

- 元素个数小于8000左右时，list效率是最好的。

- 元素个数大于200左右时，unordered_multiset效率是最差的。

- 元素个数小于200左右时，deque效率是最差的。主要原因是开始时一次高耗时操作，但是之后每次操作耗时均不多（线的变化率）。  

#### 从后往前  
结论：

-  vector在各个方向的遍历效率均比较优秀。
-  list在从前往后遍历时比deque优秀。
-  deque在从后向前遍历时比list优秀。
-  关联容器的遍历效率没有非关联容器高。  

### 查找  
因为非关联容器的查找只能通过遍历，其效率和关联容器的查找没法比。所以我们只比较关联容器  
结论：

-   unordered系列容器比ordered系列容器效率高
-   ordered系列容器中，map系列比set系列对应的容器效率高。比如map比set优，multimap比multiset优
-   set效率最差
-   unordered_multiset最优。