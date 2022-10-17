---
title: C++常用数据结构的简单使用
date: 2021-10-21 14:09:23
categories:
- 数据结构
tags: 
- 数据结构
- C++
---

## 一、常用数据结构一览

| 数据结构          | 用途             |
| ----------------- | ---------------- |
| [vector](#vector) | 动态数组         |
| [stack](#stack)   | 先入后出的线性表 |
| [queue](#queue)   | 先入先出的线性表 |
| [map](#map)       | 键值对元素集合   |
| [set](#set)       | 无序非重复集合   |

> 持续完善与更新中...

<!--more-->

<!-- tab vector -->

## vector

### 1.导入vector

vector是STL中的一个容器类，其位于std命名空间。

```c++
#include <vector>
using namespace std;
```

### 2.定义vector

vector使用一个泛型来指定vector元素的数据类型。

```c++
// 定义元素数据类型为int的vector
vector<int> int_vector;

// 定义元素数据类型为string的vector(在使用string之前先include)
vector<string> string_vector; 
```

### 3.初始化vector

定义vector时不加任何赋值语句，其本身已经初始化为一个空的vector。

与普通数组的初始化方式一样，vector也可以使用`{元素1, 元素2, ... 元素n}`的形式初始化多个元素，如下所示：

```c++
vector<int> int_vector = {1, 2, 3};
```

### 4.使用vector

```c++
// 通过下标取值
int first_element = int_vector[0];

// 遍历（普通方式）
int current_vector_element;
for (int i = 0; i < int_vector.size(); i++) {
    current_vector_element = int_vector[i];
}

// 遍历（迭代器方式）
vector<int>::iterator iterator;
for (iterator = int_vector.begin(); iterator != int_vector.end(); iterator++) {
    current_vector_element = *iterator;
}

// 在末尾增加元素
int_vector.push_back(4);

// 删除末尾的元素
int_vector.pop_back();
```

<!-- endtab -->



<!-- tab stack -->

## stack

// TODO

<!-- endtab -->



<!-- tab queue -->

## queue

// TODO

<!-- endtab -->



<!-- tab map -->

## map

### 1.导入map

map是STL中的一个容器类，其位于std命名空间。

```c++
#include <map>
using namespace std;
```

### 2.定义map

map使用两个泛型来指定map元素中键与值的数据类型。

```c++
// 定义键为string类型且值为int类型的map
map<string,int> string_int_map; 
```

### 3.初始化map

map可以使用`{ {键1,值1}, {键2,值2}, ... {键n,值n} }`的形式初始化多个元素，如下所示：

```c++
map<string,int> string_int_map  = {{"a",1}, {"b",2}};
```

### 4.使用map

```c++
// 遍历（迭代器方式）
string current_key;
int current_value;
map<string,int>::iterator iterator;
for (iterator = string_int_map.begin(); iterator != string_int_map.end(); iterator++) {
    current_key = iterator->first;
    current_value = iterator->second;
}
```

<!-- endtab -->



<!-- tab set -->

## set

// TODO

<!-- endtab -->
