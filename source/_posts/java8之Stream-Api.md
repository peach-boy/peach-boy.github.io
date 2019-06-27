---
title: java 8之Stream Api解析
date: 2019-06-18 16:50:30
categories: java基础
tags:
---

### 一、流操作之中间操作简介
中间操作主要是用来对Stream做出相应转换及限制流，实际上是将源Stream转换为一个新的Stream，以达到需求效果。


### 二、常用中间操作方法

|操作|类型|返回类型|操作参数|函数描述符|作用|
|-----|-----|----|------|-------|-------|
|filter|中间|Stream<T>|Predicate<T>   |T -> boolean|返回参数函数为true的元素的流|
|map   |中间|Stream<T>|Function<T, R> |T -> R      |参数函数会被应用到每个元素上，并将其映射成一个新的元素|
|flatMap|中间|Stream<T>|Function<T, R> |T -> R      |flatmap 方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。|
|limit |中间|Stream<T>|               ||返回前n个元素的流|
|skip  |中间|Stream<T>|               ||返回一个跳过前n个元素的流（不足n个元素，返回一个空流）|
|sorted|中间|Stream<T>|Comparator<T>  |(T, T) -> int|按照传入的Comparator安排续|
|distinct|中间|Stream<T>|             |            |根据流所生成元素的hashCode 和 equals 方法，实现去重后的流|

### filter方法
使用实例：遍历出相同的字符串
```java
 Stream.of("a", "a", "b")
                .filter((s) -> s.startsWith("a"))
                .forEach(System.out::println);
  //return:"a","a"
```
### map方法
类似的有：mapToDouble，mapToInt，mapToLong。
使用实例：求和处理
```java
Stream.of(1, 1, 1)
               .mapToInt(Integer::intValue)
               .sum();
       //return:3
```
```java
Long l=Stream.of(1L,2L,4L)
                .mapToLong(Long::longValue)
                .sum();
//return:7
```
### flatMap方法
使用实例： 给 定 单 词 列 表"Hello","World" ，你想要返回列表 "H","e","l", "o","W","r","d" ，做法就是先分割，在去重，返回字符串数组。
```java
  List<String> uniqueCharacters =
                Stream.of("heelo","adafa")
                        .map(w -> w.split(""))
                        .flatMap(Arrays::stream)
                        .distinct()
                        .collect(Collectors.toList());
```
### limit方法和skip方法
使用实例：取前n个元素
```java
 Stream.of(1, 2, 3, 4, 5)
	   .limit(3)
	    .forEach(i -> System.out.println(i));
//return 1,2,3
```
使用实例：跳过前3个元素
```java
 Stream.of(1, 2, 3, 4, 5)
	   .skip(3)
	    .forEach(System.out::println);
//return 4,5
```
### distinct方法
使用实例：去重操作
```java
Stream.of("a", "b", "c", "c")
                .distinct()
                .forEach(System.out::println);
 //return:a,b,c
```



