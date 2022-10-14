---
title:       "const int* 与 int const* 的区别"
subtitle:    ""
description: ""
date:        2018-06-04
author:      ""
image:       ""
tags:        ["cpp_note"]
categories:  ["" ]
---

作者：王国潇
链接：https://www.zhihu.com/question/443195492/answer/1723886545
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

不管const写成如何，读懂别人写的const和*满天飞的类型的金科玉律是const默认作用于其左边的东西，否则作用于其右边的东西：  

> const applies to the thing left of it. If there is nothing on the left then it applies to the thing right of it.[1]   

例如  
> const int*  

const只有右边有东西，所以const修饰int成为常量整型，然后*再作用于常量整型。所以这是a pointer to a constant integer（指向一个整型，不可通过该指针改变其指向的内容，但可改变指针本身所指向的地址）  

> int const *  

再看这个，const左边有东西，所以const作用于int，*再作用于int const所以这还是 a pointer to a constant integer（同上）  

> int* const  

这个const的左边是*，所以const作用于指针（不可改变指向的地址），所以这是a constant pointer to an integer，可以通过指针改变其所指向的内容但只能指向该地址，不可指向别的地址。  

> const int* const  

这里有两个const。左边的const 的左边没东西，右边有int那么此const修饰int。右边的const作用于*使得指针本身变成const（不可改变指向地址），那么这个是a constant pointer to a constant integer，不可改变指针本身所指向的地址也不可通过指针改变其指向的内容。  

> int const * const  

这里也出现了两个const，左边都有东西，那么左边的const作用于int，右边的const作用于*，于是这个还是是a constant pointer to a constant integerint const * const *懒得分析了，照葫芦画瓢，a pointer to a constant pointer to a constant integer，其实就是指向上边那个的东西的指针。  

> int const * const * const  

上边的指针本身变成const，a constant pointer to a constant pointer to a constant integer。  

再扯两句，因为const的作用机理和阅读习惯，码农之间产生了两种写const的流派，一个是所谓的western const style也即把const写在西边（左边），例如const int，符合人类语言习惯；  

另一个流派叫eastern const style，遵照上边的规则把const写在东边（右边），例如int const。从代码可读性易维护性出发，强烈推荐把const写在右边（包括自己学的时候老师如是安利。。。），可以跟指针的作用范围很好地统一起来不至于混乱。  

update 2021-02-17：应 @m9lee 加了引用出处[1] https://stackoverflow.com/questions/5503352/const-before-or-const-after