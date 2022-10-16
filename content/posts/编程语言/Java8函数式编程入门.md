---
title: Java8函数式编程入门
date: 2021-08-21 09:38:30
categories:
- 编程语言
- Java
tags:
- Java
---

## 一、Lambda表达式

什么是Lambda表达式?

Lambda表达式是一个函数，有入参和出参，它表示一个行为。

Lambda表达式是一个对象，Java中函数是二等公民，只能依附于类存在，Lambda表达式的目标类型被称之为函数接口。

### （一）Lambda表达式与匿名内部类对象

Lambda表达式和匿名内部类对象很像，在正式介绍Lambda表达式之前，先看看一个匿名内部类对象的例子。

一般情况下，接口不可以直接实例化，但可以在new接口的过程中重写接口中的方法来创建一个匿名内部类对象。如下所示：

```java
Runnable anonymousClassesRunnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("我是一个匿名类内部类对象");
    }
};
```

有没有更简单的方式呢？请看下述代码：


```
Runnable lambdaRunnable = () -> System.out.println("我是一个lambda表达式");
```

上述代码中`=`右侧的`() -> System.out.println("我是一个lambda表达式")`是一个`Lambda表达式`。

这个Lambda表达式由三部分组成：

1. `()` : 请求参数，这里为空（对应了上述匿名内部类中run方法的请求参数）
2. `System.out.println("我是一个lambda表达式")`: 方法体（对应了上述匿名内部类中run方法的方法体）
3. `->` : Lambda表达式标识符，连接左侧的请求参数和右侧的方法体

可以看到，Lambda表达式与匿名内部类对象完全等价，它们都在`=`右侧，都可以被赋值给`=`左侧的接口变量，换言之，**Lambda表达式本质上是一个接口的实例化对象**（在Java中，万物皆对象，lambda表达式也不例外）。那么，任意接口都可以是Lambda表达式的目标类型吗？

<!--more-->

### （二） 函数接口

Lambda表达式只能表示一个行为，这就意味着其对应的接口只能有唯一的方法。同时，为了更精确地标识Lambda表达式的目标接口，通常会给对应的接口加上`@FunctionalInterface`注解。这种接口叫做函数接口。

**函数接口是Lambda表达式的目标类型**。

查看一下上述例子中`Runnable`类的源码，如下所示：

```java
package java.lang;

@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
可以看到，Runnable类有以下特性：

1. 只有一个方法
2. 含有@FunctionalInterface注解

所以，Runnable是一个函数接口。

jdk的`java.util.function`包中内置了许多函数接口，常见的如下所示：

| 接口           | 方法              | 使用场景                   |
| -------------- | ----------------- | -------------------------- |
| Predicate<T>   | boolean test(T t) | Strem的filter/match系列    |
| Consumer<T>    | void accept(T t)  | Strem的forEach/peek        |
| Function<T, R> | R apply(T t)      | Strem的map系列/flatMap系列 |
| Supplier<T>    | T get()           | Strem的collect/generate    |


## 二、Stream

Stream基于Lambda表达式，可以对集合进行复杂操作，实现集合的过滤、映射、去重、排序、查找、求值等功能。

### （一）Stream常用API一览

一个有效的Stream操作可以由0到n个**中间操作**和1个**终止操作**组成。

中间操作（intermediate operation）返回Stream对象，可以在Stream操作过程中链式调用。

终止操作（terminal operation）返回空或特定值，在终止操作之前，任何中间操作都不会执行。

#### 1. 中间操作

| 方法     | 请求参数类型  | 返回类型  | 功能 |
| -------- | ------------- | --------- | ---- |
| filter   | Predicate<T>  | Stream<T> | 过滤 |
| map      | Function<T,R> | Stream<R> | 映射 |
| flatMap  | Function<T,R> | Stream<R> | 映射 |
| distinct | void          | Stream<T> | 去重 |
| sorted   | Comparator<T> | Stream<T> | 排序 |

> 其它中间操作：mapToInt、mapToLong、mapToDouble、flatMapToInt、flatMapToLong、flatMapToDouble、peek、limit、skip

#### 2. 终止操作

| 方法      | 请求参数类型 | 返回类型 | 功能 |
| --------- | ------------ | -------- | ---- |
| forEach   | Consumer     | void     | 循环 |
| collect   | Collector    | ...      | 收集 |
| findFirst | void         | Optional | 查找 |
| findAny   | void         | Optional | 查找 |
| anyMatch  | Predicate    | boolean  | 查找 |
| allMatch  | Predicate    | boolean  | 查找 |
| noneMatch | Predicate    | boolean  | 查找 |
| min       | Comparator   | Optional | 求值 |
| max       | Comparator   | Optional | 求值 |
| count     | void         | long     | 求值 |

> 其它终止操作：forEachOrdered、reduce、toArray

### （二）创建Stream对象

在正式介绍Stream用法之前，先定义两种类型的列表，后续的所有实例都是基于这两种列表。

``` java
class Student {
    private String studentId;
    private String name;
    private String gender;
    private int age;
}

class School {
    private String schoolId;
    private String name;
    private List<Student> students;
}

List<Student> syStudents = new ArrayList<>();
List<Student> fdStudents = new ArrayList<>();
List<School> schools = new ArrayList<>();

syStudents.add(new Student("1", "野原新之助", "男", 5));
syStudents.add(new Student("2", "风间彻", "男", 5));
syStudents.add(new Student("3", "樱田妮妮", "女", 4));
syStudents.add(new Student("4", "佐藤正男", "男", 4));
syStudents.add(new Student("5", "阿呆", "男", 4));

fdStudents.add(new Student("1", "图图", "男", 3));
fdStudents.add(new Student("2", "小美", "女", 3));
fdStudents.add(new Student("3", "刷子", "男", 4));
fdStudents.add(new Student("4", "壮壮", "男", 5));
fdStudents.add(new Student("5", "小豆丁", "男", 1));

schools.add(new School("1", "双叶幼稚园", syStudents));
schools.add(new School("2", "翻斗幼儿园", fdStudents));
```

创建Stream对象一般使用`Collection.stream()`方法，如下所示：
```java
Stream<Student> studentStream = syStudents.stream(); //创建一个元素为Student的Stream对象
Stream<School> schoolsStream = schools.stream(); //创建一个元素为Student的Stream对象
```

### （二）过滤

Stream过滤可以使用`filter`方法，过滤掉集合中符合特定条件的元素。这是一个中间操作，一般配合`collect`或`findFirst`等终止操作一起使用。

filter方法接受一个目标类型为Predicate函数接口的lambda表达式作为请求参数。

```java
// 找到双叶幼稚园中所有年龄大于等于4岁的男孩子
List<Student> newStudents = syStudents
        .stream() // 创建一个元素为Student的Stream对象
        .filter(student -> "男".equals(student.gender)) // 过滤得到所有性别为男的小朋友
        .filter(student -> student.age >= 4) // 在上一步过滤的基础上, 再次过滤得到所有年龄大于等于4的小朋友
        .collect(Collectors.toList()); // 收集过滤之后的集合

Assert.assertEquals(3, newStudents.size());
```

### （三）映射

Stream映射可以使用`map`或`flatMap`方法，将集合中的元素映射为另一个元素。这是一个中间操作，一般配合`collect`等终止操作一起使用。

map方法与flatMap方法接受一个目标类型为Function函数接口的lambda表达式作为请求参数。

```java
// 找到双叶幼稚园中所有小朋友的名字
List<String> newStudents = syStudents
        .stream()
        .map(student -> student.name) // 将student对象映射为姓名
        .collect(Collectors.toList());// 收集姓名集合

Assert.assertEquals("[野原新之助, 风间彻, 樱田妮妮, 佐藤正男, 阿呆]", newStudents.toString());
```

```java
// 获得所有幼稚园中的小朋友集合
List<Student> newStudentsFromAllSchool = schools
        .stream()
        .flatMap(school -> school.students.stream()) // 将school对象映射为一个student流, 并将所有的student流合并成一个
        .collect(Collectors.toList()); 

Assert.assertEquals(10, newStudentsFromAllSchool.size());
```


### （四）去重

Stream去重可以使用`distinct`方法，去除集合中的重复元素。这是一个中间操作，一般配合`collect`等终止操作一起使用。

distinct方法没有请求参数, 只有重写集合元素类的equals方法时, distinct去重才有意义。

```java
syStudents.add(new Student("1","蜡笔小新","男", 5));

// 去除双叶幼稚园中重复的小朋友
List<Student> newStudents = syStudents
        .stream()
        .distinct() // 去重, 必须重写equals方法
        .collect(Collectors.toList());//收集去重后的集合

Assert.assertEquals(5, newStudents.size());
```

### （五）排序

Stream排序可以使用`sorted`方法，对集合中的元素进行排序。这是一个中间操作，一般配合`collect`等终止操作一起使用。

sorted方法可以接受一个目标类型为Comparator函数接口的lambda表达式作为请求参数。sorted方法也有一个无参的重载方法，必须保证集合元素类实现实现Comparable接口，才能调用sorted的无参重载方法否则会抛出ClassCastException异常。

```java
// 将双叶幼稚园中所有的小朋友按年龄从小到大顺序重新排序
List<Student> newStudents = syStudents
        .stream()
        .sorted((o1, o2) ->  o1.age - o2.age) // 按年龄从小到大排序
        .collect(Collectors.toList()); //收集排序后的集合

Assert.assertEquals(3, newStudents.get(0).age);
```

### （六）查找

Stream查找也可以使用`findFirst`及`findAny`方法，找到集合中第一个或任意满足特定条件的一个元素。这是一个终止操作，一般配合`filter`中间操作一起使用。

find系列方法没有请求参数, 返回一个Optional对象。

```java
// 查找双叶幼稚园中第一个studentId为1的小朋友
Optional<Student> optionalFirstStudent = syStudents
        .stream()
        .filter(student -> Objects.equals(student.studentId, "1")) // 过滤得到所有studentId为1的小朋友
        .findFirst(); // 在上述过滤的到的集合中找到第一个符合条件的元素
// Optional对象处理
Student firstStudent = optionalFirstStudent.get();

Assert.assertEquals("野原新之助", firstStudent.name);


// 查找双叶幼稚园任意一个3岁的小朋友
Optional<Student> optionalAnyStudent = syStudents
        .stream()
        .filter(student -> Objects.equals(student.age, 3)) // 过滤得到所有年龄为3的小朋友
        .findAny(); // 在上述过滤的到的集合中找到第一个符合条件的元素
// Optional对象处理
Student anyStudent = optionalAnyStudent.orElse(new Student(null, null, null, 0));

Assert.assertEquals("阿呆", anyStudent.name);
```


Stream查找使用`anyMatch`、`allMatch`及`noneMatch`方法，判断集合中的元素是否满足特定条件。这是一个终止操作。

match系列方法接受一个目标类型为Predicate函数接口的lambda表达式作为请求参数。

```java
// 判断双叶幼稚园中是否所有的小朋友的年龄都小于等于5岁
boolean isAllCorrect = syStudents
        .stream()
        .allMatch(student -> student.age <= 5); // 判断是否所有的小朋友的年龄都小于等于5岁

Assert.assertTrue(isAllCorrect);


// 判断双叶幼稚园中是否有任意一个的小朋友是女孩子
boolean isAnyCorrect = syStudents
        .stream()
        .anyMatch(student -> Objects.equals(student.gender, "女")); // 判断有任意一个的小朋友的性别为女

Assert.assertTrue(isAnyCorrect);


// 判断双叶幼稚园中是否没有一个小朋友的年龄大于5岁
boolean isNoneCorrect = syStudents
        .stream()
        .noneMatch(student -> student.age > 5); // 判断是否没有一个小朋友的年龄大于5岁

Assert.assertTrue(isNoneCorrect);
```

### （七）求值

Stream排序可以使用`min`、`max`及`count`等方法，求得集合的最小值、最大值及元素个数等值。这是一个终止操作。

```java
// 计算双叶幼稚园中所有的小朋友的最小年龄
int minAge = syStudents
        .stream()
        .map(student -> student.age)
        .min((o1, o2) ->  o1 - o2)
        .get();

// 计算双叶幼稚园中所有的小朋友的最大年龄
int maxAge = syStudents
        .stream()
        .map(student -> student.age)
        .max((o1, o2) ->  o1 - o2)
        .get();

// 计算双叶幼稚园中所有的小朋友的人数
long count = syStudents
        .stream()
        .count();
        
Assert.assertEquals(3, minAge);
Assert.assertEquals(5, maxAge);
Assert.assertEquals(5, count);
```

## 附录

### 代码实例

```java
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class LambdaDemo {

    List<Student> syStudents = new ArrayList<>();
    List<Student> fdStudents = new ArrayList<>();
    List<School> schools = new ArrayList<>();

    /**
     * 初始化数据
     */
    @Before
    public void initData() {
        syStudents.add(new Student("1", "野原新之助", "男", 5));
        syStudents.add(new Student("2", "风间彻", "男", 5));
        syStudents.add(new Student("3", "樱田妮妮", "女", 4));
        syStudents.add(new Student("4", "佐藤正男", "男", 4));
        syStudents.add(new Student("5", "阿呆", "男", 3));

        fdStudents.add(new Student("1", "图图", "男", 3));
        fdStudents.add(new Student("2", "小美", "女", 3));
        fdStudents.add(new Student("3", "刷子", "男", 4));
        fdStudents.add(new Student("4", "壮壮", "男", 5));
        fdStudents.add(new Student("5", "小豆丁", "男", 1));

        schools.add(new School("1", "双叶幼儿园", syStudents));
        schools.add(new School("2", "翻斗幼儿园", fdStudents));
    }

    /**
     * 展示一个简单的Lambda表达式
     */
    @Test
    public void showLambdaAndAnonymousClass() {
        Runnable anonymousClassRunnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("我是一个匿名类内部类对象");
            }
        };

        Runnable lambdaRunnable = () -> System.out.println("我是一个lambda表达式");

        new Thread(anonymousClassRunnable).start();
        new Thread(lambdaRunnable).start();
    }

    /**
     * 1. 创建Stream对象
     */
    @Test
    public void createStreamObject() {
        Stream<Student> studentStream = syStudents.stream(); //创建一个元素为Student的Stream对象
        Stream<School> schoolsStream = schools.stream(); //创建一个元素为Student的Stream对象
    }

    /**
     * 2. 过滤
     */
    @Test
    public void filter() {
        // 找到双叶幼稚园中所有年龄大于等于4岁的男孩子
        List<Student> newStudents = syStudents
                .stream() // 创建一个元素为Student的Stream对象
                .filter(student -> "男".equals(student.gender)) // 过滤得到所有性别为男的小朋友
                .filter(student -> student.age >= 4) // 在上一步过滤的基础上, 再次过滤得到所有年龄大于等于4的小朋友
                .collect(Collectors.toList()); // 收集过滤之后的集合

        Assert.assertEquals(3, newStudents.size());
    }

    /**
     * 3. 映射
     */
    @Test
    public void map() {
        // 找到双叶幼稚园中所有小朋友的名字
        List<String> newStudents = syStudents
                .stream()
                .map(student -> student.name) // 将student对象映射为姓名
                .collect(Collectors.toList());// 收集姓名集合

        Assert.assertEquals("[野原新之助, 风间彻, 樱田妮妮, 佐藤正男, 阿呆]", newStudents.toString());

        // 获得所有幼稚园中的小朋友集合
        List<Student> newStudentsFromAllSchool = schools
                .stream()
                .flatMap(school -> school.students.stream()) // 将school对象映射为一个student流, 并将所有的student流合并成一个
                .collect(Collectors.toList());// 收集合并后的student集合

        Assert.assertEquals(10, newStudentsFromAllSchool.size());
    }

    /**
     * 4. 去重
     */
    @Test
    public void removeDuplicate() {
        syStudents.add(new Student("1","野原新之助的替身","男", 5));

        // 去除双叶幼稚园中重复的小朋友
        List<Student> newStudents = syStudents
                .stream()
                .distinct() // 去重, 必须重写equals方法
                .collect(Collectors.toList());//收集去重后的集合

        Assert.assertEquals(5, newStudents.size());
    }

    /**
     * 5. 排序
     */
    @Test
    public void sort() {
        // 将双叶幼稚园中所有的小朋友按年龄从小到大顺序重新排序
        List<Student> newStudents = syStudents
                .stream()
                .sorted((o1, o2) ->  o1.age - o2.age) // 按年龄从小到大排序
                .collect(Collectors.toList()); //收集排序后的集合

        Assert.assertEquals(3, newStudents.get(0).age);
    }

    /**
     * 6. 查找
     */
    @Test
    public void findAndMatch() {
        // 查找双叶幼稚园中第一个studentId为1的小朋友
        Optional<Student> optionalFirstStudent = syStudents
                .stream()
                .filter(student -> Objects.equals(student.studentId, "1")) // 过滤得到所有studentId为1的小朋友
                .findFirst(); // 在上述过滤的到的集合中找到第一个符合条件的元素
        // Optional对象处理
        Student firstStudent = optionalFirstStudent.get();

        Assert.assertEquals("野原新之助", firstStudent.name);


        // 查找双叶幼稚园任意一个3岁的小朋友
        Optional<Student> optionalAnyStudent = syStudents
                .stream()
                .filter(student -> Objects.equals(student.age, 3)) // 过滤得到所有年龄为3的小朋友
                .findAny(); // 在上述过滤的到的集合中找到第一个符合条件的元素
        // Optional对象处理
        Student anyStudent = optionalAnyStudent.orElse(new Student(null, null, null, 0));

        Assert.assertEquals("阿呆", anyStudent.name);


        // 判断双叶幼稚园中是否所有的小朋友的年龄都小于等于5岁
        boolean isAllCorrect = syStudents
                .stream()
                .allMatch(student -> student.age <= 5); // 判断是否所有的小朋友的年龄都小于等于5岁

        Assert.assertTrue(isAllCorrect);


        // 判断双叶幼稚园中是否有任意一个的小朋友是女孩子
        boolean isAnyCorrect = syStudents
                .stream()
                .anyMatch(student -> Objects.equals(student.gender, "女")); // 判断任意一个的小朋友的性别为女

        Assert.assertTrue(isAnyCorrect);


        // 判断双叶幼稚园中是否没有一个小朋友的年龄大于5岁
        boolean isNoneCorrect = syStudents
                .stream()
                .noneMatch(student -> student.age > 5); // 判断是否没有一个小朋友的年龄大于5岁

        Assert.assertTrue(isNoneCorrect);

    }

    /**
     * 7. 取值
     */
    @Test
    public void evaluation() {
        // 计算双叶幼稚园中所有的小朋友的最小年龄
        int minAge = syStudents
                .stream()
                .map(student -> student.age)
                .min((o1, o2) ->  o1 - o2)
                .get();
        Assert.assertEquals(3, minAge);

        // 计算双叶幼稚园中所有的小朋友的最大年龄
        int maxAge = syStudents
                .stream()
                .map(student -> student.age)
                .max((o1, o2) ->  o1 - o2)
                .get();
        Assert.assertEquals(5, maxAge);

        // 计算双叶幼稚园中所有的小朋友的人数
        long count = syStudents
                .stream()
                .count();
        Assert.assertEquals(5, count);
    }

    class Student {
        private String studentId;
        private String name;
        private String gender;
        private int age;

        public Student(String studentId, String name, String gender, int age) {
            this.studentId = studentId;
            this.name = name;
            this.gender = gender;
            this.age = age;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Student student = (Student) o;
            return studentId.equals(student.studentId);
        }

        @Override
        public int hashCode() {
            return Objects.hash(studentId);
        }
    }

    class School {
        private String schoolId;
        private String name;
        private List<Student> students;

        public School(String schoolId, String name, List<Student> students) {
            this.schoolId = schoolId;
            this.name = name;
            this.students = students;
        }
    }

}

```