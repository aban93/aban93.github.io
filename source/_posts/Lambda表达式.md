---
title: Lambda表达式
date: 2018-08-14 00:08:31
tags: Lambda
categories: Java 8
---
# 简言
由于之前一直使用的都是Java 7,对Java 8的一些新特性不甚了解。最近Aban学习了一下Java 8的一些新特性，在这里简单总结、分享一下。如有错误，还望指正。
## 行为参数化的思想
在使用Lambda表达式之前，我们应该先理解一个重要概念----行为参数化。
行为参数化，简单来说，就是你准备好一个代码块，却不去执行它，这个代码块以后可以被程序其他部分调用。更通俗的说，就是把方法(你的代码)作为参数传递给另一个方法。
![img1](/images/1.png)
<!-- more -->
![img2](/images/2.png)
比如，将代码块传递给另一个方法，稍后去执行它。这个方法的行为就基于那块代码被参数化了。例如，你要处理一个集合，你一可能会写一个方法：
* 对列表中的每个元素做“某件不可描述的事情”
* 在列表处理完后做“另一件不可描述的事情”
* 遇到错误时可以做“另外一件不可描述的事情”
### 初试牛刀
我们接下来看一个例子，然后展示一些让代码灵活的最佳做法。(<em><font color="brown">例子较长，请耐心观看~~</font></em>)
有一个农场仓库，里面有很多苹果，我们用一个list集合来表示。
1. 农民伯伯想要找出所有的绿苹果，听起来很简单：
``` bash
public static List<Apple> filterGreenApples(List<Apple> apples) {
	List<Apple> greenApples = new ArrayList<Apple>();
	for(Apple apple: apples) {
		if("green".equals(apple.getColor())) {
			greenApples.add(apple);
		}
	}
	return greenApples;
}
```
我们检出所有的绿苹果之后，农民伯伯改变注意了，想要找出各种颜色苹果：绿色，红色，黄色......
2. 再展身手，将颜色作为参数
针对农民伯伯的需求，我们可以把颜色作为参数，这样会更加灵活一点：
``` bash
public static List<Apple> filterApples(List<Apple> apples,String color) {
	List<Apple> result = new ArrayList<Apple>();
	for(Apple apple: apples) {
		if(apple.getColor().equals(color)) {
			result.add(apple);
		}
	}
	return result;
}
```
我们再复杂一点，农民伯伯又说，要是能区分轻的苹果和重的苹果就好了，大于100克的算是重的苹果。作为有职业操守的程序员，我们早可以想到农民伯伯可能要改变重量，于是又有了下面的方法：
``` bash
public static List<Apple> filterApples(List<Apple> apples,int weight) {
	List<Apple> result = new ArrayList<Apple>();
	for(Apple apple: apples) {
		if(apple.getWeight() > weight) {
			result.add(apple);
		}
	}
	return result;
}
```
虽然有了解决方案，但这有点令人失望，因为中间有很多重复的代码。
3. 第三次尝试
我们可以把筛选颜色和重量弄到一个方法里面，定义一个标识来判断是筛选颜色还是重量：
``` bash
public static List<Apple> filterApples(List<Apple> apples,String color,int weight,boolean flag) {
	List<Apple> result = new ArrayList<Apple>();
	for(Apple apple: apples) {
	    if((flag && apple.getColor().equals(color)) ||
		    (!flag && apple.getWeight() > weight)) {
		    result.add(apple);
	    }
	}
	return result;
}
```
你可以这么用：
``` bash
List<Apple> greenApples = filterApples(apples,"green",0,true);
List<Apple> heavyApples = filterApples(apples,"",100,false);
```
但说句实话，这个方案实在糟糕透了，如果需要组合属性做更复杂的查询，或者有更加复杂的需求，可能需要更加冗长复杂的代码(<em><font color="brown">你身为程序员的职业操守呢？？？</font></em>)
### 运用行为参数化
从上面可以看到，我们需要一种更好的方法来应对变化的需求，需要更高层次的抽象。
我们可以根据Apple的属性，来返回一个boolean值，我们称之为<b>谓词</b>(即一个返回boolean值的函数)
首先，我们定义一个接口来对选择标准建模：
``` bash
public interface ApplePredicate {
	boolean test(Apple apple);
}
```
然后，我们就可以用ApplePredicate的多个实现来代表不同的选择标准：
``` bash
public class AppleColorPredicate implements ApplePredicate{
	//选出绿苹果
	@Override
	public boolean test(Apple apple) {
	   return "green".equals(apple.getColor());
	}
}
```
``` bash
public class AppleWeightPredicate implements ApplePredicate{
	//选出重的苹果
	@Override
	public boolean test(Apple apple) {
	   return apple.getWeight() > 100;
	}
}
```
最后，我们的filter方法看起来是这样的：
``` bash
public static List<Apple> filterApples(List<Apple> apples,ApplePredicate p){
    List<Apple> result = new ArrayList<Apple>();
    for (Apple apple : apples) {
        if(p.test(apple)) {
        result.add(apple);
    }
}
	return result;
}
```
这里，filterApples方法需要接受ApplePredicate对象，对Apple做条件测试。filterApples方法的行为取决于通过ApplePredicate对象传递的代码，换句话说，我们把filterApples方法的行为参数化了！(<em><font color="brown">身为程序员的你终于有了有一丝丝尊严！！！</font></em>)
由此，我们对行为参数化有了更精确的解释：<b>让方法接受多种行为作为参数，并在内部使用，来完成不同的行为。</b>
### 进阶
1. 匿名类
上面的例子虽然最后算是找到一个<em><font color="brown">还算好一点</font></em>的方案,但还是有点费劲。我们可以使用匿名内部类，它允许你随用随建。
``` bash
List<Apple> redApples = filterApples(apples,new ApplePredicate() {
        //筛选红苹果
        @Override
        public boolean test(Apple apple) {
	        return "red".equals(apple.getColor());
        }
});
```
2. 使用Lambda表达式
但匿名类还是不够好，它往往很笨重，占用很多空间；而且很多程序员觉得它用起来很费解。(<em><font color="brown">唉，处女座就是麻烦...</font></em>)
在Java 8中，可以用Lambda表达式写成下面的样子：
``` bash
List<Apple> result = filterApples(apples,(Apple apple) -> "red".equals(apple.getColor()));
```
## Lambda表达式
Lambda表达式的基本语法是：
``` bash
(parameters) -> expression
```
或(请注意语句的花括号)
``` bash
(parameters) -> { statements; }
```
比如我们利用Lambda表达式比较两个苹果的重量：
![img3](/images/3.png)
这个Lambda表达式有三个部分：
* 参数列表 —— 这里采用了Comparator中compare方法的参数,两个Apple
* 箭头 —— 箭头 -> 把参数列表和Lambda主体分隔开
* Lambda主体 —— 比较两个Apple的重量，表达式就是Lambda的返回值了
### 什么时候可以使用Lambda
Lambda表达式是可以在<b>函数式接口</b>上使用的。<b>函数式接口</b>就是只定义一个抽象方法的接口。比如：
``` bash
public interface Predicate<T>{
    boolean test (T t);
}

public interface Comparator<T>(){
    int compare(T o1,T o2);
}

public interface Runnable{
    void run();
}
```
其实，Lambda表达式允许你直接以内联的形式为函数式接口的抽象方法提供实现，并<em>把整个表达式作为函数式接口的实例</em>(确切来说，是函数式接口的一个具体实现的实例)。
### Lambda表达式的具体使用
1. 如果我们想要从一个文件中读取一行所需的内容，可以定义这样的方法：
``` bash
public static String processFile() throws IOException {
    try (BufferedReader br = 
            new BufferedReader(new FileReader("data.txt"))){
        return br.readLine();
    }
}
```
那如果要读取两行呢，这时候我们就应该记起来行为参数化。
2. 使用函数式接口来传递行为
前面已经说过，Lambda仅可用于上下文是函数式接口的情况。我们需要创建一个匹配BufferedReader -> String，还可以抛出异常的接口。
``` bash
public interface BufferedReaderProcessor {
    String process(BufferedReader br) throws IOException;
}
```
现在，我们可以把这个接口作为processFile方法的参数：
``` bash
public static String processFile(BufferedReaderProcessor p) throws Exception{
        try (BufferedReader br = 
              new BufferedReader(new FileReader("data.txt"))){
            return p.process(br);
        }
}
```
3. 传递Lambda
现在我们就可以通过传递不同的Lambda重用processFile方法，以不同方式处理文件。
* 处理一行:
``` bash
String oneLine = processFile((BufferedReader br) -> br.readLine());
```
* 处理两行:
``` bash
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
## 几种函数式接口
Java 8中常用的函数式接口有三个:Predicate,Consumer,Function。这里我们简单介绍使用一下，具体使用有兴趣可以自己实践下。
### Predicate
java.util.function.Predicate<T>接口定义了一个名叫test的抽象方法，它接受泛型T对象，并返回一个boolean。在需要表示一个涉及类型T的布尔表达式时，可以使用这个接口。比如，你可以定义一个接受String对象的Lambda表达式。
``` bash
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) { 
    List<T> results = new ArrayList<>(); 
    for(T s: list){ 
        if(p.test(s)){ 
            results.add(s); 
        } 
    } 
    return results; 
} 

Predicate<String> predicate = (String s) -> !s.isEmpty(); 
List<String> nonEmpty = filter(listOfStrings, predicate); 
```
### Consumer
java.util.function.Consumer<T>接口定义了一个名叫accept的抽象方法,它接受泛型T，没有返回值(void)。如果需要访问类型T的对象，并对其执行某些操作，可以使用这个接口。
比如定义一个forEach方法，接受一个Integer类型的列表，并对每个元素执行打印操作。
``` bash
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

public static <T> void forEach(List<T> list,Consumer<T> c) {
    for(T i: list) {
        c.accept(i);
    }
}

forEach(
    Arrays.asList(1,2,3,4,5),(Integer i) -> System.out.println(i)
);
```
### Function
java.util.function.Function<T, R>接口定义了一个叫作apply的方法，它接受一个泛型T的对象，并返回一个泛型R的对象。如果需要定义一个Lambda，将输入的信息映射到输出，可以使用这个接口(比如提取苹果的重量，或把字符串映射为它的长度)。
比如，定义一个map方法，将一个String列表映射到包含每个String长度的Integer列表。
``` bash
@FunctionalInterface 
public interface Function<T, R>{ 
    R apply(T t); 
} 

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {   
    List<R> result = new ArrayList<>(); 
    for(T s: list){ 
        result.add(f.apply(s)); 
    } 
    return result; 
} 

// [6,3,6]
List<Integer> list = map(
		Arrays.asList("lambda","int","action"),
		(String s) -> s.length()
	);
```