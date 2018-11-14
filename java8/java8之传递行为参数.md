# 通过行为参数传递代码  

首先，论述这样一个需求，现在有一个农民种了漫山遍野的苹果，农民正在为一件事情而头疼，他需要根据颜色、重量、大小来筛选苹果，由于今年收成尤其的好，苹果太多，筛选起来非常的费时费力，作为程序员的你，有什么好的办法来帮助这个农民呢。

## 第一种方式

定义一个筛选苹果的方法，遍历所有的苹果，我们挑选出绿色的苹果。现在这个方法应该是这样的：

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
List<Apple> result = new ArrayList<Apple>();
for(Apple apple: inventory){
if( "green".equals(apple.getColor() ) {
result.add(apple);
}
}
return result;
}
```

按照这种方式，有个明显的缺点，那就是如果农民需要筛选红色苹果，此时你又需要另外写一个筛选红色苹果的方法。重复代码太多了。

## 第二种方式

定义一个筛选苹果的方法，将苹果颜色作为参数，这样我们就可以真正的实现农民根据颜色来筛选苹果了，现在这个方法是这样的：

```java
public static List<Apple> filterGreenApples(List<Apple> inventory, String color) {
List<Apple> result = new ArrayList<Apple>();
for(Apple apple: inventory){
if(color.equals(apple.getColor() ) {
result.add(apple);
}
}
return result;
}
```

按照这个方式，虽然实现了按照颜色来筛选苹果的工作，但是如果农民说，我想要筛选重量>150克的苹果，这时候就不满足了，按照上面的方式，只能继续复制上面的代码，把参数改成重量。这样代码又重复了。

## 第三种方式

定义一个筛选苹果的方法，将苹果颜色和重量作为参数，并传入一个识别是筛选重量还是颜色的参数。现在这个方式是这样的：

```java
public static List<Apple> filterApples(List<Apple> inventory, String color,
int weight, boolean flag) {
List<Apple> result = new ArrayList<Apple>();
for (Apple apple: inventory){
if ( (flag && apple.getColor().equals(color)) ||
(!flag && apple.getWeight() > weight) ){
result.add(apple);
}
}
return result;
}
```

这个方法虽然解决了，按颜色和重量来动态筛选苹果，但是代码太难看了，而且并没有更深层次的抽象，若此时农民说需要按照大小来筛选苹果，就没辙了。

### 更深层次的抽象

我们可以把以上三种方式，定义为一种筛选苹果的行为，而这种行为其实就是一个抽象方法。这个抽象方法的作用就是筛选苹果。可以通过不同的实现类来实现不同的筛选苹果的方式，现在代码是这样的。

定义一个筛选苹果的抽象方法

```java
public interface ApplePredicate{
boolean test (Apple apple);
}
```

选出绿色苹果

```java
public class AppleGreenColorPredicate implements ApplePredicate{
public boolean test(Apple apple){
return "green".equals(apple.getColor());
}
}
```

选出重的苹果

```java
public class AppleHeavyWeightPredicate implements ApplePredicate{
public boolean test(Apple apple){
return apple.getWeight() > 150;
}
}
```

筛选苹果

```java
public static List<Apple> filterApples(List<Apple> inventory,
ApplePredicate p){
List<Apple> result = new ArrayList<>();
for(Apple apple: inventory){
if(p.test(apple)){
result.add(apple);
}
}
return result;
}
```

这就是典型的策略模式，实现一个抽象方法，来达到不同的行为，然后，我们将实现类作为参数传递给方法，来实现苹果的筛选，实际上，可以理解为，将不同的行为传递给了方法，只不过实现类是不同行为的载体，这就是行为参数传递代码的意思。不难发现，这种方式需要我们定义非常多的实现类，怎么才能用最少的实现类来实现不同的行为呢？可以用匿名内部类的方式

## 匿名内部类

将上的方式改造下，用匿名内部类的方式作为参数传递，代码是这样的

```java
filterApples(appleList, new ApplePredicate() {
@Override
public boolean test(Apple apple) {
return apple.getWeight() > 150;
}
});
```

通过这种方式我们就可以省略了接口的实现类，直接传递一个匿名内部类就行了。

## lambda表达式

```java
filter(appleList, apple1 -> apple1.getWeight() > 150);
```

你看，使用lambda表达式多么方便，一行代码就搞定。

## 最后的方式

通过以上的方式，我们已经实现对苹果的筛选，并实现了抽象化，但是我们更希望的是不仅仅是苹果，而是任何东西，接下来代码是这样的：

抽象行为

```java
public interface Predicate<T>{
boolean test(T t);
}
```

筛选方法

```java
public static <T> List<T> filter(List<T> list, Predicate<T> p){
List<T> result = new ArrayList<>();
for(T e: list){
if(p.test(e)){
result.add(e);
}
}
return result;
}
```

调用

```java
List<Apple> redApples =
filter(inventory, (Apple apple) -> "red".equals(apple.getColor()));

List<Integer> evenNumbers =
filter(numbers, (Integer i) -> i % 2 == 0);
```

## 总结

我们可以把第一种、第二种、第三种方式理解为值参数化，而对于，传递类、匿名内部类、lambda表达式，成为行为参数化，相比于值参数化，行为参数化更加灵活，重复代码更少。而lambda表达式其实就是对匿名内部类的一种优化，使得我们的代码更加简单、简洁、易读。
