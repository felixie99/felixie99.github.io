---
title:       "bit operator"
subtitle:    ""
description: ""
date:        2018-06-04
author:      ""
image:       ""
tags:        ["cpp_note"]
categories:  ["" ]
---

# 二进制位运算


### 不用加减乘除做加法
'无进位和'与'异或运算'规律相同, '进位'和'与运算'规律相同（需左移一位，注意c++中的负数左移结果不定)
[参考链接](https://leetcode.cn/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/solutions/210882/mian-shi-ti-65-bu-yong-jia-jian-cheng-chu-zuo-ji-7/)

### 寻找最右侧的1
n & (~n + 1)