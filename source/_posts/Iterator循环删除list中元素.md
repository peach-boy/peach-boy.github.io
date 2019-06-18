---
title: Iterator循环删除list中元素
date: 2019-06-18 16:49:39
categories: java基础
tags:
---

### 一、业务场景
当我们需要剔除list中的某些元素时，通常的做法就是循环遍历list,然后符合条件时remove掉该元素。
### 二、实现逻辑
    for (Iterator<String> iterator = list.iterator(); iterator.hasNext(); ) {
            String str=iterator.next();
            if (Objects.equals("one",str)){
                iterator.remove();
            }
        }
    // or
    Iterator<String> iterator = list.iterator();
        while(iterator.hasNext()){
            String str=iterator.next();
            if (Objects.equals("one",str)){
                iterator.remove();
            }


### 三、代码分析
使用iterator遍历list同时删除元素，不会因为删除元素改变size而造成异常