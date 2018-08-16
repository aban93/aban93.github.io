---
title: Lambda表达式
date: 2018-08-14 00:08:31
tags: Lambda
categories: Java 8
---
# 简言
由于之前一直使用的都是Java 7,对Java 8的一些新特性不甚了解。最近Aban学习了一下Java 8的一些新特性，在这里简单总结、分享一下关于Lambda表达式的一些东西。如有错误，还望指正。
## 行为参数化的思想
在使用Lambda表达式之前，我们应该先理解一个重要概念----行为参数化。
行为参数化，简单来说，就是你准备好一个代码块，却不去执行它，这个代码块以后可以被程序其他部分调用。更通俗的说，就是把方法(你的代码)作为参数传递给另一个方法。
<!-- more -->
![img1](/images/1.png)
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
### Supplier
java.util.function.Supplier<T>接口定义了一个get的抽象方法，它没有参数，返回一个泛型T的对象，这类似于一个工厂方法。
比如返回一个Apple对象。
``` bash
public interface Supplier<T> {
    T get();
}

public static <T> T getObject(Supplier<T> s) {
    return s.get();
}

Apple apple = () -> new Apple();
```
## 方法引用
我们上面写到的Lambda表达式是很方便的，但确实它们可以再简洁一点，比如根据苹果重量对集合进行排序，Lambda表达式是这样的：
``` bash
apples.sort((Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```
使用<em><b> 方法引用 </b></em>和 java.util.Comparator.comparing 可以写成这样子：
``` bash
apples.sort(comparing(Apple::getWeight));
```
### 基本格式
方法引用显式地指明调用的方法的名称，使代码的<b>可读性更好</b>。
当你想要使用方法引用时，目标引用放在分隔符 : : 前，方法名称放在后面：
![img4](/images/4.png)
下面给出了一些Java 8中方法引用的例子：
![img5](/images/5.png)
### 如何构建
方法引用主要有三类。
1. 指向<em>静态方法</em>的方法引用(例如Integer 的 parseInt 方法)
``` bash
Integer :: parseInt
```
2. 指向<em>任意类型实例方法</em>的方法引用(例如String 的 length 方法)
``` bash
String :: length  //实例为方法参数
```
3. 指向<em>现有对象的实例方法</em>的方法引用(假设你有一个局部变量transaction，为Transaction类型，它支持实例方法getValue，就可以写成下面这样)
``` bash
Transaction :: getValue  //实例为外部对象
```
第2钟和第3钟乍一看有点晕，其实第二种方法引用的思想就是你在引用一个对象的方法，这个对象本身是lambda的一个参数；第三种方法引用是你再调用一个已经存在的外部对象的方法。
### 构造函数引用
上面我展示了如何创建方法引用，其实我们也可以对类的构造函数做类似的事情。
我们可以利用 <b>ClassName :: new</b> 的形式构建一个构造函数的引用。假设一个构造函数没有参数，它试合Supplier的签名() -> Apple:
``` bash
Supplier<Apple> supplier = Apple :: new;
Apple apple = supplier.get();
```
在使用方法引用之前它是这样的：
``` bash
Supplier<Apple> supplier = () -> new Apple();
Apple apple = supplier.get();
```
如何你的构造函数是有参数的，比如签名是Apple(Integer weight),那么它就适合Function接口的签名：
``` bash
Function<Integer,Apple> func = Apple :: new;
Apple apple = func.apply(100);
```
这就等价于：
``` bash
Function<Integer,Apple> func = (weight) -> new Apple(weight);
Apple apple = func.apply(100);
```
## Lambda和方法引用实战
接了下来我们继续研究前面的一个例子——按照苹果重量给Apple列表排序，我会展示从原始粗暴的状态到更加简明状态的过程，而且会用到前面提到的概念和功能：行为参数化、匿名类、Lambda表达式和方法引用。
### 行为参数化——传递代码
Java 8的API已经为我们提供了一个List可用的sort方法，我们可以直接使用。我们可以看下sort方法的签名：
``` bash
void sort(Comparator<? super E> c)
```
它需要一个Comparator对象来比较两个Apple，所以第一个方案可以是这样的：
``` bash
public class AppleComparator implements Comparator<Apple>{

	@Override
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}

}

apples.sort(new AppleComparator());
```
### 使用匿名类
我们可以使用匿名类来改进，而不是实现一个Comparator却只实例化一次：
``` bash
apples.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```
### 使用Lambda表达式
使用匿名类的方案还挺啰嗦的，既然我们了解了Lambda表达式，它可以用更轻量级的语法来<em>传递代码</em>。我们需要记住这一点：<b>在需要函数式接口的地方可以使用Lambda表达式，抽象方法的签名描述了Lambda表达式的签名</b>。
Comparator接口抽象方法的签名是符合这种形式的——(T,T) -> int。所以我们改进后的方案是这样的：
``` bash
apples.sort((Apple a1,Apple a2) 
              -> a1.getWeight().compareTo(a2.getWeight())
);
```
其实，Java的编译器是可以根据Lambda出现的上下文来推断Lambda表达式参数的类型的：
``` bash
apples.sort((a1,a2) -> a1.getWeight().compareTo(a2.getWeight()));
```
Comparator接口其实有一个comparing的静态方法，可以接受一个Function，并返回一个Comparator对象。像下面这样：
``` bash
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());
```
这时候我们的代码就可以更简洁一点了：
``` bash
import static java.util.Comparator.comparing;

apples.sort(comparing(a -> a.getWeight()));
```
### 使用方法引用
最后，我们可以使用方法引用来完成最终解决方案：
``` bash
apples.sort(comparing(Apple::getWeight));
```
# 总结
在了解了Lambda表达式的和方法引用的用法之后，你就可以自己去尝试用Lambda表达式去简化一些代码了(你可以自己去练习一下)。不过用于传递Lambda表达式的Comparator、Function、Predicate等函数式接口提供了允许你进行复合的方法。这意味着你可以把多个简单的Lambda复合成复杂的表达式。有兴趣的童鞋可以自己去了解下，这里不再详细讲解。