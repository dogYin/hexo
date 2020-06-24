---
title: 线性表算法练习
date: 2020-06-22 14:33:13
tags:
    - algorithms
    - 线性表
categories:
    - algorithms
top_img: https://cdn.jsdelivr.net/gh/dogYin/imgOSS/img/algorithms.jpg
cover: https://cdn.jsdelivr.net/gh/dogYin/imgOSS/img/algorithms.jpg
---
线性表：只能在前后两个方向访问(数组，链表，队列，栈)

数组：拥有连续内存空间和相同数据类型的线性表
为什么数组从0开始：0代表当前数据的便宜量，如果从1开始寻址算法多了k-1的操作，对于操作系统来说就多了一次指令
数组寻址算法：a[i]_adr = base_adr + i * data_type_size (从0开始)
              a[k]_adr= base_adr+ (k-1) * data_type_size(从1开始)
二维数组寻址算法：a[i]_adr = base_adr + i * data_type_size 
链表：通过指针将多个零散的内存块连接在一起