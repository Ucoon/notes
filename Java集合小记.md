---
title: Java集合小记
---

# Java集合和数组的区别

1. 数组长度在初始化时指定，意味着只能保持定长的数据；而集合可以保存数量不确定的数据。同时可以保存具有映射关系的数据（即键值对key-value）
2. 数组元素可以是基本类型的值，也可以是对象；集合里只能保存对象（实际上只是保存对象的引用变量），基本数据类型的变量要转换成对应的包装类才能放入集合类中

# Java集合类之间的继承关系

Java的集合类主要由两个接口派生而出：```Collection```和```Map```，```Collection```和```Map```是Java集合框架的根接口

## Collection接口

![Collection集合图](C:\AndroidProject\MyBlogImg\Collection集合图.png)

```Collection```是```Set```，```Queue```和```List```的父接口。```Collection```接口中定义了多种方法可供其子类进行实现，以实现数据操作，参考https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html

### Set集合

Set集合与Collection集合基本相同，没有提供额外的方法。特别的是，Set集合不允许包含相同的元素，如果试图把两个相同的元素加入同一个Set集合中，则添加操作失败，add()返回false，且新元素不会被加入。

### List集合

List集合代表一个元素有序、可重复的集合，集合中每个元素都有其对应的顺序索引。

### Queue集合

Queue集合模拟队列这种数据结构，队列是线性存储结构；特点：先进先出（FIFO，first-in-first-out），新元素插入（offer）到队列的尾部，访问元素（poll）操作会返回队列头部的元素，通常队列不允许随机访问队列中的元素。

## Map接口

![Map集合图](C:\AndroidProject\MyBlogImg\Map集合图.png)

Map保存的是具有映射关系的数据，因此Map集合里保存着两组数，一组保存着Map里的key，另一组值保存着Map里的value，key和value都可以是任何引用类型的数据，Map的key不允许重复

![key-value](C:\AndroidProject\MyBlogImg\key-value.jpg)