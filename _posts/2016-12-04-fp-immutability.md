---
layout: post
title: '函数式编程的那些特性(一): Immutability'
date: "2016-12-04T18:59:32+08:00"
comments: true
post_id: fp-immutability
---

> FBI warning:  
> 这又是一个系列的文章（Hopefully），介绍Functional Programming的一些特性，以及Why there are so awesome! 目测将cover以下的一些话题：  

> 1. Immutability
> 2. Pure function
> 3. No side-effect
> 4. High order function
> 5. Curried Function
> 6. Pattern matching
> 7. Effect managements.(Monads, STM，...)
> 8. Rethink OOP
> 9. ......

> 目前没有办法确定更新频率，甚至没有办法保证一定会更新下去，只能说我尽量努力一个星期更新一遍，but I can’t give you my word.
> 另外，我相应你已经发现了，这些文章（Like every other posts）将会充斥着各种英文单词或句子。部分是为了装逼，部分是因为当我写到那里的时候，就想这么表达而已，感觉很带劲。所以，慎重关注，不喜还请轻喷！

这几天一直在折腾Elm，这门函数式编程语言，应该说，感觉真是酸爽。使用一门新的语言，在一个小的Community里面，还真的会有不少的问题，不少的坑。这些坑被你踩到的时候，可能还不能很快的得到解决，你要费一番功夫，你要Google得时间长一点，你要耐心的去看每一个相关的github issue、pull request，甚至相关的源码。此外，IDE的支持也是个大问题，可能每一个编辑器/IDE，都有不足的地方.比如说对于Elm开发，我现在的解决方案是，同时使用WebStorm(for editing files)和Atom（for linting），利用Mac的分屏功能，一边一个，这想想都觉得搞笑啊！但是，问到问题，解决问题，我想这也是程序员的一大乐趣吧！

Anyway，这个系列的文章将介绍一下[函数式编程](https://en.wikipedia.org/wiki/Functional_programming)（[Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)，以下简称FP）的一些特性。当然首先要解释的问题是，什么是函数式编程？这个概念我相信大家已经听烂了，但是究竟什么叫函数式编程呢？我们知道这个概念是跟面向对象编程一样，是一种编程模式，但是它跟OO有什么不同呢？有些人可能知道这是比OO更早出现的一种编程模式，但是为什么它没有像OO一想，如今得到如此广泛的应用？又是什么原因，最近这几年的编程界，大家又都在讨论这个东西？

我们一个一个问题来看，首先看看FP在[Wikipedia的定义](https://en.wikipedia.org/wiki/Functional_programming)：

> In computer science, functional programming is a programming paradigm—a style of building the structure and elements of computer programs—that treats computation as the evaluation of mathematical functions and **avoids changing-state and mutable data**. It is a declarative programming paradigm, which means programming is done with expressions[1] or declarations[2] instead of statements. In functional code, the **output value of a function depends only on the arguments** that are input to the function, so calling a function f twice with the same value for an argument x will produce the same result f(x) each time. **Eliminating side effects**, i.e. changes in state that do not depend on the function inputs, can make it much easier to understand and predict the behavior of a program, which is one of the key motivations for the development of functional programming.

其实我认为，上面的描述基本把FP的核心给概括到了，那就是：

1. Immutable data，数据是不可变的（Immutability）
2. 一个函数的输出只依赖于这个函数的输入(pure function)
3. 任何一个函数都没有side effect：一个函数除了操纵它的输入，然后返回输出之外，再也不做其它的事情。

这里提到三个特性，基本上每一种函数式编程语言都有比这三个多出很多的特性和概念，而且这些特性和概念在每一种函数式编程语言里面基本都是类似的，然而我认为，这三个特性才是函数式编程最根本的特性，是跟其它编程模式（OO、过程式编程）最截然不同的地方。

单单把这三个特性列出来，听起来有点虚，有点理论，有点模糊，那么接下来我用具体的例子来说明一个，这三个到底是什么样的东东，对于我们习惯了OO的人来说，写起代码来有什么样的一种感受。这里先介绍第一个：

### Immutability

这个包括两层含义：1. 没有变量，只有常量，任何一个“变量”一旦被赋值，就不能再次被赋值。类似于所以的变量都是用final(in Java)或const(in C/C++/JS/etc)修饰过了的；2. 数据一旦被创建出来，就不能被修改了。比如一个数组被创建出来以后，就不能再增、删、改里面的元素了；一个Map（Hash、Dictionary）被创建出来以后，里面的key-value pair也是不能再变的的，也不能add key，也不能remove key，一个对象/Struct一旦被创建，里面的每一个field都不能被改变了。这个特性听起来有点奇怪，什么要这么设定呢？这个特性对我们编程有什么影响呢？

#### 变量的噩梦

看看以下的代码：

```
age = 18    
happy_new_year()
print(age)  //What is the output
```

上面这段小代码，对于非FP来说，`age`是可以变的，所以你必须去看happy\_new\_year里面的代码，才知道`age`最后到底是什么样的值。而对于FP来说，你不用去管`happy_new_year()`里面的代码是什么样的，你就能确定`age`的值一定是18。因为变量`age`一旦被赋值，就不会被改变了。  
所以，从这个简单的例子就可以看出，FP可以极大的提高代码的可读性（可理解性）。  
现在，想象一下，你最近接触的OO代码里面有多少可变的成员变量？为了控制某一个地方的逻辑而专门设置的状态变量？多少可变的临时变量？这些变量给你造成了多大的思想负担？你要翻遍所以可能改变这个变量的地方，又有哪些public方法会改变这个变量？甚至外面又有哪些地方调用了这个方法，你可能都要找清楚，有的时候一个变量可能不会被改变，但是你还是要去找一遍，因为你不知道这个值不会被改变，所以你翻遍了所有的地方，结果发现，这个值没有被改变。。。回想一下你就会发现，这简直就是一个噩梦！  
也许你会说，对于哪些初始化以后不会被改变的变量，你可以用final来修饰一下，但是你回想一下，你有多少次会真的这么去做呢？多数人用final，是因为定义常量或者是一个临时变量要被某个匿名内部类引用到，所以不得不用final。再说，就算你是个优秀的程序员（如果一个程序员会因为考虑到一个变量初始化以后不会被改变而特地加一个final，那我认为他确实是个优秀的程序员！），就算你看过[Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683)，你会记得该加final的地方加final，那又有多大比例人能像你这么优秀呢？再说，这还是解决不了那些确实会被改变的变量引起的谜之代码，这依然是个噩梦！
而且，噩梦还不会这里结束，想象一下如果`happy_new_year `这个方法是异步的，里面起了一个新的线程去改变`age`的值呢？在这种情况下，`age`的值就更难判断了，虽然你说，可以用一些加锁的方式去做同步，但是你不能否认，写好多线程的代码并不是那么容易的。而且，这些额外的控制代码，只会让代码的可读性进一步的降低。所以毫不夸张的说，这简单就是升级版的噩梦！

#### 可变的数据（结构）

可变的数据结构也是同样的道理。我相信以下的代码很多安卓的同学都写过：

```java
public class SomeActivity extends Activity {
	private List<...> mDataList = new ArrayList<>();
	//....
	void someMethod() {
		ListView listView = (ListView)findViewById(R.id.some_list_view)
		mDataList = ...  //一个List类型的成员变量。
		YourAdapter adater = new YourAdapter(...., mDataList)
		listView.setAdapter(adapter);
	}
}
```

如果你不是写安卓的孩子，这里简单解释一下，以上的代码就是在界面上显示一个列表。列表里面的每一项对应于`mDataList`这个List里面的一项数据，`adater`就是一个将`mDataList`里面的每一项数据转变成`ListView`里面的一行的工具。但是你发现某些时候，你的`ListView`上面显示的数据与你预期的可能不一样，或许多了一些，或许少了一些，或许某一项是错误的，你费了好多时间调试下来，发现原来是在某些不经意的地方，你修改了`mDataList`里面的值。但是这个`mDataList`怎么会影响到`ListView`呢？哦，原来你的`adater`里面也有一个data list，那个data list跟你Actvity里面的`mDataList`引用的是同一个对象，类似于这样的实现：

```java
public class YourAdapter extends BaseAdapter {
	private List<...> dataList; //
	public BaseAdapter(..., List<...> data) {
		...
		this.dataList = data;
	}
}
```

这种情况其实是非常的counter-intuitive的，一般来说当你看到

```Java
	YourAdapter adapter = new YourAdapter(...., mDataList)
	listView.setAdapter(adapter);
```

的时候，你不会再想到`adater`里面的数据跟外面的`mDataLis`t还有关系，你会以为`adapter`取了`mDataList`当时那一刻的数据，后面除非你手动往`adapter`里面增删数量，不然里面的数据是不会变的。所以如果你的Adapter也是这种情况，那你就得无时无刻不意识到，`mDataList`跟`adapter`里面的`dataList`是同一个对象，要非常小心的修改里面的数据，这无疑给人增加了非常大的心里负担。此外，这个`mDataList`会不会在某个地方被传给另外一个对象？那个对象会不会在某个地方修改了这个List？它又会不会再次传给别人？？？更糟糕的是，一般来说你的代码不是你一个人在维护，其它人知道这个`mDataList`跟`adapter`之间的关系吗？如果他不清楚，在某个地方无意间修改了`mDataList`的值怎么办？如果他写了一段代码，传给了别的对象怎么办？同时，`adapter`本身会不会有一个方法return这个list给别人，那个人会不会修改这个List？？？？？？我相信你看到这里，你自己头都大了。但是你别急，因为还有更糟糕的情况，那就是很多时候你需要异步的修改`mDataList`里面的值，这个时候又需要加锁，需要同步，从而代码可读性进一步降低，这又回到上面那个升级版的噩梦了。  
当然，像上面的情况，解决的办法也有很多，其中一个比较简单的是在你的`adapter`里面不要持有跟外面传进去的list同一个引用，而是自己new一个，当外面把数据传进来的时候，把里面的数据自己copy一份。这样，外面对`mDataList`的修改就不会影响到`adapter`了，类似于下面这样：

```Java
public class YourAdapter extends BaseAdapter {
	private final List<...> dataList = new ArrayList<>(); //我用final我自豪！
	public BaseAdapter(..., List<...> data) {
		...
		if (data != null) {
			dataList.addAll(data);
		}
	}
}
```

这个其实也是[Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683)里面推荐的Best Practices之一(Item39: Make defensive copies when needed)。Again，这需要一个程序员有比较高的觉悟，需要他有这方面的意识，知道这里有个坑需要注意，还需要他有比较高的水平（比如看过Effective Java并且会照着做）。觉悟、意识、水平，这些都是很难控制的事情，也不是很容易、不是一时半会就能培养出来的事情。  
总之，Mutability is Evil！
![](http://upload-images.jianshu.io/upload_images/63600-a61cb2bd29e3b1a6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/63600-1293b7404165c364.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


那FP在这里的作用是怎么样的呢？那就是他解决了这个问题。因为里面的数据、“变”量都是不可变的，所以上面所说的问题统统不存在！想想，这是不是极大的提高了代码的可读性，可理解性，极大的减少了人的心理负担？
因为`age`是不可变的，所以无论`happy_new_year()`里面的代码是怎么样的，你都不用关心，你可以立刻的肯定，`print(age)`的输出一定是18，无论有多少个线程，都是这样的。  
因为`mDataList`是不可变的，所以就算`adapter`持有的跟`mDataList`同一个引用也没关系，你永远不用担心`mDataList`会改变，从而也不用担心对它的改变会不会影响到`adapter`，无论`mDataList`被传给多少个其它对象都无所谓，无论有多少个线程都无所谓。  
那如果我需要改变`age`的值怎么办（比如27天以后你就老了一岁）？如果我需要往`mDataList`里面加数据怎么办？答案很简单，你创建另外一个“变”量来存放它们改变以后的值，然后在需要使用这些新值的地方用那个新的“变”量就好了。

```
new_age = age + 1
List<...> newDataList = mDataList.add(...);
```

我相信看到这里，有一个强烈的反应马上出现在你心里：这太夸张了，这岂不是需要额外增加非常多的对象？这岂不是会极大的增加GC的负担？  
对于这个问题，我的答案是，这个担心完全是没必要的，多数的FPL(L for Language）都会在底层实现解决这个问题，比如共用内存。比如在上面的`newDataList`和`mDataList`中，这两个List共用`mDataList`现有的内存空间，只不过`newDataList`多了一个元素的内存而已。因为这片内存是不可变的，所以这个共用完全是安全的。当然不同的语言、不同的情况可能会有不同的方式来处理这个问题。总之，目前还没有哪种FPL说因为immutability而导致GC负担太重的问题。

“但是我总有一些地方确实需要“变量”啊，我要用来表示对象当前的状态、系统当前的状态，当前用户的状态，这些东西没有变量怎么行呢？”
你有这个疑问，是因为你的思想还停留在OO的编程模式中，在FP中是不会这样去思考问题的，FP is about transforming data, not about maintaining state. There is much less "state" stuff in FP.
不明白?那就跟着这个系列，后面你将看到，没有“state”的编程是怎么样一种体验。

## 小结
这篇文章介绍了FP的immutability特性，为什么mutability这么糟糕，为什么immutability这么有必要，不知道讲得是不是够清楚，有任何问题、疑问、反对意见，欢迎留言！
