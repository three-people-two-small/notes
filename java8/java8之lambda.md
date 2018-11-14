# lambda表达式

先来个例子：

```java
// 按重量升序排列
Comparator<Apple> byWeight = new Comparator<Apple>(){
 @Override
 public int compare(Apple o1, Apple o2) {
 return o1.getWeight().compareTo(o2.getWeight());
 }
 };
    appleList.sort(byWeight);
```

这是一个将苹果按重量升序排列的例子，为了实现排序接口，这里用了匿名内部类的方式。而我的理解是，其实lambda表达式就是对匿名内部类的一种简化，下面是用lambda表达式实现上述的例子

```java
// 按重量升序排列
Comparator<Apple> byWeight = (o1,o2)->o1.getWeight().compareTo(o2.getWeight());
appleList.sort(byWeight);
```

发现，用lambda表达式的方式的确是精简了不少的代码。lambda的语法到底是怎么样的？

## lambda组成

1. 参数列表：(o1, o2)就是参数，代表两个Apple对象

2. 箭头：将参数和主体分开，用箭头表示。

3. Lambda主体：比较两个Apple的重量，可以理解为接口方法的内部行为，整个表达式结果就是返回值。

## 如何使用

一言以蔽之，我们可以把lambda表达式作为行为参数传递给目标方法，而这个行为参数必须是函数式接口，那么什么是函数式接口呢？函数式接口就是只有一个抽象方法的接口。

```java
appleList.sort((o1,o2)->o1.getWeight().compareTo(o2.getWeight()));
```

## 方法引用

方法引用并不是方法调用，它是一种方法标记，表示当发生方法调用的时候，标记要执行的方法。

```java
// 1
IntFunction<Integer> a = Integer::valueOf;
Integer b = a.apply(1);
System.out.println(b);
```

如上述例子，Integer::valueOf 就是方法引用，我们来看把这段方法引用转成lambda表达式是这样的

```java
IntFunction<Integer> a = int1->Integer.valueOf(int1);
```

方法引用主要下面几类

1. 指向静态方法的方法引用：如上面的Integer.valueOf方法

2. 指向任意类型的实例方法：如String的length方法，方法引用格式为String::length

3. 指向现有对象的实例方法

```java
// 现有对象
Apple apple = new Apple();
apple.setWeight(200);
Supplier<Integer> supplier = apple::getWeight;
```

4. 构造函数：可以用类的名称和关键字new构建一个引用，ClassName::new

```java
Supplier<Apple> supplier = Apple::new;
Apple apple = supplier.get();
```

综合以上对方法引用的概述，对于lambda表达式用方法引用确实是能让代码更加的简洁，但是对于构造方法用方法引用，似乎没什么必要。

最后，我们的排序的代码可以简洁到只有一行

```java
appleList.sort(Comparator.comparing(Apple::getWeight));
```