---
title: Java正则入门
date: 2021-04-07 18:01:00
categories:
- 编程语言
- Java
tags:
- Java
# description: 以String类中的匹配与替换为入口, 简单介绍Java正则匹配与正则替换
---

## 一、前言

正则表达式一般用于字符串的模式**匹配**与**替换**，Java通过Pattern类与Matcher类原生支持正则表达式。在此基础上，String类封装了正则的细节，提供了一种更便捷的正则操作的方式。

本文将从String类说起，通过它的matches与replace系列方法，介绍[String的匹配与替换](#二、String匹配与替换)。随后，将介绍Java正则中的[Pattern与Matcher](#三、Pattern与Matcher)两大核心类的基本使用方法。最后，将从[String类源码](#四、String正则源码分析)层面，简单分析String类的正则匹配和正则替换是如何通过Pattern类和Matcher类实现的。

有关正则表达式的前置知识介绍，请参照本文[附录](#附录)中的[正则表达式通用知识](#正则表达式通用知识)。

<!--more-->

## 二、String匹配与替换

### （一）String匹配

可以使用String的`matches`方法对字符串进行匹配，它的定义为：`boolean matches(String regex)`，`matches`方法可以用来判断目标字符串是否符合指定的正则表达式(regex)，一个简单的示例如下：

```Java
// 待匹配的目标字符串
String targetString = "<Name>Violet</Name>";
// 正则表达式
String regex = "<Name>.*</Name>";

// 正则匹配
boolean isMatched = targetString.matches(regex);
```

上述示例中，对`"<Name>Violet</Name>"`字符串按照`"<Name>.*</Name>"`这个正则表达式进行匹配，因为目标字符串符合规则，所以匹配成功。

> 部分场景下，如果只需要判断字符串中是否有某个子串，可以直接使用String的`contains`方法，这种方法在匹配是否包含子串是很有效，但在更复杂的匹配规则下就不适用了，此时还是需要使用上述的`matches`方法。

### （二）String替换

可以使用String的`replace`系列方法对字符串进行替换，包括：`replace`、`replaceAll`、`replaceFirst`，它们的定义如下：

1. `String replace(char oldChar, char newChar)`
    **字符**简单替换，将目标字符串中所有的`oldChar`字符替换为`newChar`字符，**不涉及到正则表达式**
2. `String replace(CharSequence target, CharSequence replacement)`
    字符串简单替换，将目标字符串中所有的`target`字符串替换为`replacement`字符串，**不涉及正则表达式**
3. `String replaceAll(String regex, String replacement)`
    字符串**正则**替换，将目标字符串中所有符合的`regex`正则表达式的子串替换为`replacement`
4. `String replaceFirst(String regex, String replacement)`
    字符串**正则**替换，将目标字符串中**第一个**符合的`regex`正则表达式的子串替换为`replacement`

一个简单的示例如下：
```Java
// 待替换的目标字符串
String targetString = "<Name>Violet</Name>";
// 正则表达式
String regex = "<Name>.*</Name>";

// 1. 简单替换所有匹配到的字符
String newString1 = targetString.replace('V','v');  //结果为: "<Name>violet</Name>"
// 2. 简单替换所有匹配到的字符串
String newString2 = targetString.replace("Violet","Violet.Evergarden");  //结果为: "<Name>Violet.Evergarden</Name>"
// 3. 正则替换所有匹配到的字符串
String newString3 = targetString.replaceAll(regex,"<Name>Violet.Evergarden</Name>");  //结果为: "<Name>Violet.Evergarden</Name>"
// 4. 正则替换第一个匹配到的字符串
String newString4 = targetString.replaceFirst(regex,"<Name>Violet.Evergarden</Name>");  //结果为: "<Name>Violet.Evergarden</Name>"
```

## 三、Pattern与Matcher

Pattern与Matcher是Java支持正则表达式的两大核心类，上述[String匹配与替换](#二、String匹配与替换)中的正则匹配与正则替换在底层都是通过Pattern与Matcher来实现的，具体实现过程请参见[String正则源码分析](#四、String正则源码分析)。

一个Pattern与Matcher的基本使用方法的示例如下：

```Java
// 待替换的目标字符串
String targetString = "<Name>Violet</Name>";
// 正则表达式
String regex = "<Name>.*</Name>";

// 以正则表达式字符串regex为基础，定义Pattern
Pattern pattern = Pattern.compile(regex);
// 以Pattern与目标字符串为基础，定义Matcher
// 后续的所有正则匹配与正则替换，都是对Matcher进行操作
Matcher matcher = pattern.matcher(targetString);

// 正则匹配，等同于String.matches(String regex)
boolean isMatched = matcher.matches();
// 正则替换，等同于String.replaceAll(String regex, String replacement)
String newString = matcher.replaceAll("<Name>Violet.Evergarden</Name>");
```

> Pattern与Matcher还有很多其它的用法，此处暂时不做展开，后续如果有需要在此处进行补充。

## 四、String正则源码分析



### （一）String正则匹配源码分析

String的`matches`方法实现如下所示：

```Java
public boolean matches(String regex) {
    return Pattern.matches(regex, this);
}
```

String的`matches`方法很简单，只是简单地调用了`Pattern`类的`matches`方法。

进入到`Pattern`类的`matches`方法，如下所示：

```Java
public static boolean matches(String regex, CharSequence input) {
    Pattern p = Pattern.compile(regex);
    Matcher m = p.matcher(input);
    return m.matches();
}
```

可以发现里面的内容与上一节[Pattern与Matcher](#三、Pattern与Matcher)中的使用示例大同小异：首先根据正则表达式实例化Pattern对象，然后根据Pattern对象与目标字符串实例化Matcher对象，最后调用Matcher对象的matches方法。最终起作用的正是`Matcher`对象的`matches`方法。


### （二）String正则替换源码分析

String的`replaceFirst`与`replaceAll`方法实现如下所示：

```Java
public String replaceFirst(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
}

public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}

```

String的`replace`系列方法也很简单：首先调用`Pattern.compile(regex)`实例化Pattern对象，然后调用Pattern对象的`matcher(this)`实例化Matcher对象（此处的this就是当前字符串)，最后调用Matcher对象的`replaceFirst`与`replaceAll`方法。与[String的正则匹配实现方法](#（一）String正则匹配源码分析)类似，最终起作用的正是`Matcher`对象的`replaceFirst`与`replaceAll`方法。

本小节简单分析了如何通过Pattern类和Matcher类来实现String正则匹配与正则替换，具体实现细节可以继续往下深挖，受限于篇幅，此处就不介绍了（其实是因为我是个懒狗，还没看完，挖个坑，后面有时间补上）。

## 附录

### 正则表达式通用知识

(内容正在快马加鞭赶来...)
