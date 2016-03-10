---
layout: post
title: '命名是件大事情'
date: 2016-03-10T08:56:04+08:00
comments: true
changyan_sid: naming-is-no-little-thing
---

这是《Clean code》第二章的笔记总结，关于程序变量、方法、类的命名的一些建议。文章的内容还是非常有帮助的。很多的建议虽然我们都听过，然而看了其中的例子，你就会发现，那些建议都并不是一句空话，这样写和那样写，可读性就是有很大不同。

### 使用的名字要表现出它的作用
不好的命名：`int d;  // elapsed time in days.`  
好的命名：    `int elapsedTimeInDays;`  

很多时候，为了更好的可读性，可以使用一个类来封装一下基础类型，比如一个扫雷程序，游戏中每个格子可以用一个int[]来表示，其中数组中的元素表示了这个格子的坐标、状态等其他信息。下面的函数用于获取棋局中所有被点开的格子。

```
public List<int[]> getFlaggedCells() {
	List<int[]> flaggedCells = new ArrayList<int[]>();
	for (int[] cell : gameBoard)
		if (cell[STATUS_VALUE] == FLAGGED)
			flaggedCells.add(cell);
	return flaggedCells;
}
```
这时候可以用一个类Cell来封装一下表示每个格子的int[], 程序将变成下面的样子：

```
public List<Cell> getFlaggedCells() {
	List<Cell> flaggedCells = new ArrayList<Cell>();
	for (Cell cell : gameBoard)
		if (cell.isFlagged())
			flaggedCells.add(cell);
	return flaggedCells;
}
```
很明显的看到，可读性提高了不止一点点。

### 避免歧义
如果一个词已经有现成含义，而被命名的目的又不是那个含义，那么最好避免这种命名。  
比如有一组accounts，如果用来存储的container不是List，那么就不要命名为`accountList`，直接`accounts`就好了。

### 如果两个东西的作用不一样，那么命名上也应该明显的区分开
不好的命名：

```java
public static void copyChars(char a1[], char a2[]) {
	for (int i=0; i< a1.length; i++) {
		a2[i] = a1[i];
	}
}
```
好的命名：

```
public static void copyChars(char source[], char destination[]) {
	for (int i=0; i< source.length; i++) {
		destination[i] = source[i];
	}
}
```

### 使用可以被搜索到的命名
除了少数特殊情况（比如for循环里面)，不要用一个字母的命名。不要使用Magic Number。尽量让使用的名字可以被搜索到。  
对比：  
不好的命名：  

```
for(int j=0; j<34; j++) {
	s += (t[j] * 4)/5;
}
```
好的命名：

```
int realDaysPerIdealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for (int j=0; j < NUMBER_OF_TASKS; j++) {
	int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
	int realTaskWeeks = (realdays / WORK_DAYS_PER_WEEK);
	sum += realTaskWeeks;
}
```

### 类名和方法名
类名应该是名词，或者是名词短语。类名不应该是动词。  
方法名应该是动词或者是动词短语，动词后面是名字，表示动作的对象。  
可以考虑用工厂方法来提高构造函数的可读性，比如：  
`Complex fulcrumPoint = Complex.fromRealNumber(23.0);`  
比  
`Complex fulcrumPoint = new Complex(23.0);`  
的可读性就要好很多，有必要的话，可以将Constructor设置为private，以强制使用factory method来创建一个类。  

最后一段鸡汤的话，拿过来总结一下:  

> People are also afraid of renaming things for fear that some other developers will object. We do not share that fear and find that we are actually grateful when names change (for the better). Most of the time we don’t really memorize the names of classes and methods. We use the modern tools to deal with details like that so we can focus on whether the code reads like paragraphs and sentences, or at least like tables and data structure (a sentence isn’t always the best way to display data). You will probably end up surprising some- one when you rename, just like you might with any other code improvement. Don’t let it stop you in your tracks.

遇到不好的名字，大胆的重命名吧！
