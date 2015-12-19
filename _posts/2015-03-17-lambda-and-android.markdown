---
layout: post
title:  Java 8的Lambda及其在Android 开发中的应用
date:   2015-03-17 13:51:09 UTC
tags: [tech, java, android]
comments: true
---

上次在盆友圈发了一张照片

![lambda_example](http://chriszou.com/images/lambda_example.png)

上面的两段代码是完全等效的，但是代码行数从11行降低到了一行，更不用说在第一段代码里面，我在run方法的前后以及内部都没有加入任何的空行。由此可以看出，使用lambda可以让你的Java代码在某些情况下达到何等的简洁。
那么问题来了。。。
#### 什么叫lambda呢？
Java 8 给我们带来了lambda，然而在Oracle的文档中，我没有找到lambda的定义，wikipedia里面也没有找到适合Java中的lambda的定义。写这篇文章的时候，我在[这里](http://www.drdobbs.com/jvm/lambda-expressions-in-java-8/240166764) 看到一篇很好的介绍lambda的文章，它里面给了一个定义，我觉得还挺合适的。

> A lambda expression is a block of code with parameters.

#### lambda的写法
首先列举一个完整的lambda expression：

```Java
(int a, int b) -> {
	System.out.println("Performing add operation...");
	return a+b;
}
```

一个lambda expression由三部分组成：
- 参数：（int a, int b）是这个lambda expression的参数部分，包括参数类型和参数名
- 箭头：->
- 代码块：就是用"{}"包含着的那两句代码。

上面说的是一个完整的lambda表达式，在很多情况下，很多东西是可以省略的。比如说，当系统可以根据context自动推断出参数的类型的时候，参数类型是可以神略的。这样的话就可以写成：

```Java
(a, b) -> {
	System.out.println("Performing add operation...");
	return a+b;
}
```

系统怎么自动推断出参数类型的呢？ 在下面我们就可以看到。
再比如，如果只有一个参数，而参数的类型有可以自动判断，那么连“()”也是可以省略的，那么久写成了：

```Java
a -> {
	System.out.println("Performing add operation...");
	return a+a;
}
```

再再比如，如果代码块里面只有一行代码，那么“{}”也是可以省略的，那么久写成了：

```Java
a ->
	return a+a;
```

是的，可以写在同一行

```Java
a -> return a+a;
```

让我们更进一步，在这里，“return”其实也是没必要的。

```Java
a -> a+a;
```

Great, 如果没有参数的话，是不是就可以写成：

```Java
-> a+a
```

呢？
很可惜，如果没有参数，那么前面的"()"是必须存在的。也就是说，必须写成：

```Java
()-> a+a
```

#### lambda的用法
实际上，如果你直接把上面的代码放到你的编辑器里面，你的IDE是会报错的，因为lambda是不能这样使用的。lambda的使用永远要跟一个叫做`Functional Interface`的东西绑定在一起。什么叫`Functional Interface`呢？`Functional Interface`也是Java8 中引入的概念，是的，是为了lambda。我们知道java中的`interface`，而`Functional Interface`就是一个“只有一个抽象方法”的`interface`。比如`Runnable` 这个interface就只有一个run方法，那么它就是一个`Functional Interface`。
那么或许你要问了，什么叫只有一个**抽象**方法的interface？interface中的方法不都是抽象的吗？Well，在java8以前是这样的，而在java8中，Java引进了*default method*的概念，就是带有具体实现的方法：

```java
public interface DuckInterface {
	public void walksLikeADuck();
	public default void talksLikeADuck() {
		System.out.println("Quack!");
	}
}
```

在上面的例子中，talksLikeADuck就是一个default method，注意到这个方法的定义中有一个“default”修饰符。相信很多人会觉得这是个非常有用的feature，我也是这样觉得的。
再说回Functional Interfacel，lambda，刚刚说到，lambda必须和Functional Interface配套使用，那怎么配套使用呢？
以安卓里面的View.OnClickListener为例（它也是个Functional Interface）如果没有lambda，我们经常会这样使用的。

```Java
View.OnClickListener onClickListener = new View.OnClickListener() {
	@Override
	public void onClick(View view) {
		handleClick();
	}
});
findViewById(R.id.someView).setOnClickListener(onClickListener);
```

或者直接使用匿名内部类：

```Java
findViewById(R.id.someView).setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View view) {
		handleClick();
	}
});
```

在上面的5行代码中，有用的其实只有`handleClick()`，而我们却必须用5行代码去处理，这是非常繁琐的。在Java 8以前的世界，这样的代码非常多。有一个专门的名词来称呼这种性质的代码，叫“boilerplate code”，我不知到中文叫什么。。。
现在好了，有了lambda，我们可以这样写:

```Java
View.OnClickListener onClickListener = view -> handleClick();
findViewById(R.id.someView).setOnClickListener(onClickListener);
```

匿名内部类的版本：

```Java
findViewById(R.id.someView).setOnClickListener(view -> handleClick());
```

是不是瞬间觉得简洁优雅了？
从上面的例子可以看到，lambda其实就相当于简化了Functional Interface的实例的创建。当然，从真正意义上来讲，lambda的意义不止这么一点点，只不过从使用的角度来看，你可以这样看待。

#### 从lambda到Functional Programming
这里稍微讨论一下关于lambda的其他一些特性。
在上面的例子中

```Java
View.OnClickListener onClickListener = view -> handleClick();
findViewById(R.id.someView).setOnClickListener(onClickListener);
```

这里，你既可以吧onClickListener看作是OnClickListener的一个instance，也可以把它看做一个代码块，后面的那句`findViewById(R.id.someView).setOnClickListener(onClickListener);`就相当于是吧这个代码块传给了view.setOnClickListener()这个函数。也就是说，从某种意义上来讲，你可以把lambda看作是可以相互传递的代码块。而传递代码块，是Functional Programming（一下简称FP）非常重要的一个特征，虽然说这两者其实没有什么对等关系。因为FP的本质特征是，运行一段代码并不会改变事物的状态，也就是说，没有side-effect。而lambda里面是可以调用所在的类的成员方法的、也可以访问和修改所在类的成员变量的。
话说回来，关于FP我也不是了解的很多，我本身并没有多少FP的经验，虽然对Ruby有一定了解，但Ruby也只是“可以比较好的进行”FP而已，也不是纯粹的FP语言。纯粹的FP语言是List（包括Scheme，Clojure）、Haskell、ML等等这些。关于FP，Robert Fowler（就是《Clean code》和《The Clean Coder》的作者）有一个讲得很好的视频在[这里](https://www.youtube.com/watch?v=7Zlp9rKHGD4)。
刚刚讲到，lambda的代码块可以访问所在类的成员变量和成员方法，那对于局部变量？
我们知道，方法内部定义的匿名类是可以访问所在方法的final局部变量的，作为Functional Interface的简写方式，lambda在这点上面跟匿名类保持了一致。也就是说，lambda可以访问定义它的那个方法的final局部变量。而在Java8里面，lambda还可以访问所谓“Effectively final”的局部变量。所谓“Effectively final”的局部变量，就是说除了在定义的时候给了一个初始值以为，在没有改变过她的值的那些局部变量：

```Java
int age = 26;	//在这里，age就是Effectively final的局部变量
Runnable r = () -> System.out.println("My age is "+age);
new Thread(r).start();
```

#### 在Android开发中的应用
可是！！！Android只支持Java 7啊？怎么办？莫急，要相信网友的力量，已经有人开发了gradle的插件，可以将java 8中的labmda表达式在编译出来的bytecode里面给它转化成Java 7兼容的代码。猛戳[这里](https://github.com/evant/gradle-retrolambda)，使用方法那个页面都用，在这里就不赘述了。

#### Disclaimer
文中有很多英文的单词短语这些，是因为找不到合适的中文翻译，或者是感觉翻译过来就有点变了味道了，so。。。
一篇文章分了4天来写，我也是醉了。。。
