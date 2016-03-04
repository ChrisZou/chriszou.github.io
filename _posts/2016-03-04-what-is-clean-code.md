---
layout: post
title: '《Clean Code》阅读：在大师们的眼中，什么样的Code才是Clean Code？'
date: 2016-03-04T07:23:47+08:00
comments: true
changyan_sid: what-is-clean-code
---


前几天开始看Uncle Bob的《Clean Code》，在第一章里，作者讨论了这样的一个问题：什么样的code才是Clean code。对于这个问题，一千个Programmer可能会有一千个答案，所以作者请教了6个著名的专家，问问他们对于这个问题的看法，以下三个是我认为不那么虚的，比较有操作参考意义的回答：  

#### 1. Bjarne Stroustrup, C++之父

> I like my code to be elegant and efficient. The logic should be straightforward to make it hard for bugs to hide, the dependencies minimal to ease maintenance, error handling complete according to an articulated strategy, and per- formance close to optimal so as not to tempt people to make the code messy with unprinci- pled optimizations. Clean code does one thing well.  

这里的几个重点是：

- Elegant：意味着你的代码看起来很舒服
- Efficient：这个可能跟Bjarne Stroustrup的领域有关，毕竟使用C++的地方大多是对效率要求很高的领域。
- Error handling complete：完备的错误处理机制，这意味着关注细节。
- **Does one thing well**：就是我们所说的，Single Responsibility！

#### 2. Dave Thomas, 出版公司The Pragmatic Bookshelf Co-founder，《Pragmatic Programmer》（中文叫程序员修炼之道）、《Programming Ruby》作者。

> Clean code can be read, and enhanced by a developer other than its original author. It has unit and acceptance tests. It has meaningful names. It provides one way rather than many ways for doing one thing. It has minimal dependencies, which are explicitly defined, and pro- vides a clear and minimal API. Code should be literate since depending on the language, not all necessary information can be expressed clearly in code alone.  

这里的几个重点是：

- Can be read and enhanced by others
- **Unit and acceptance tests**
- Meaningful names
- Minimal dependency, **minimal API**

这里的test和Minimal API是我们应该注意的东西，

#### 3. Ron Jeffries 《Extreme Programming Installed》 《Extreme Programming Adventures in C#》的作者

> In recent years I begin, and nearly end, with Beck’s rules of simple code. In priority order, simple code:  
• Runs all the tests;  
• Contains no duplication;  
• Expresses all the design ideas that are in the system;  
• Minimizes the number of entities such as classes, methods, functions, and the like.  
... Of these, I focus mostly on duplication. When the same thing is done over and over, it’s a sign that there is an idea in our mind that is not well represented in the code. I try to figure out what it is. Then I try to express that idea more clearly.  
Expressiveness to me includes meaningful names, and I am likely to change the names of things several times before I settle in.... I also look at whether an object or method is doing more than one thing. If it’s an object, it probably needs to be broken into two or more objects. If it’s a method, I will always use the Extract Method refactoring on it, resulting in one method that says more clearly what it does, and some submethods saying how it is done...  

在这里，Ron Jeffries甚至把test放在第一位。
至于作者的最后一条：Minimizes the number of entities such as classes, methods, functions, and the like。我认为作者应该是有前提的。不然的话，如果以那个为标准，那么所有的代码都放到一个类，一个方法里面好了，这样明显是不行的。而且作者后面自己也提到，会把一个大的类分成两个小的，会用extract method来分解一个很长的方法。总体来说，应该大多数人都会认同：几个小的类比一个大的类要好，几个小的方法比一个大的方法要好。

下面3个回答相对来说就比较虚了，大家看看就好。

#### 4. Grady Booch， 《Object Oriented Analysis and Design with Applications》作者

> Clean code is simple and direct. Clean code reads like well-written prose. Clean code never obscures the designer’s intent but rather is full of crisp abstractions and straightforward lines of control.  

简单直接，如诗一般， straightforward lines of control，说了等于没说。abstraction，相对来说还是比较实在的。

#### 5. Michael Feathers, 《Working Effectively with Legacy Code》作者

> I could list all of the qualities that I notice in clean code, but there is one overarching quality that leads to all of them. Clean code always looks like it was written by someone who cares. There is nothing obvious that you can do to make it better. All of those things were thought about by the code’s author, and if you try to imagine improvements, you’re led back to where you are, sitting in appreciation of the code someone left for you—code left by some- one who cares deeply about the craft.

这里的重点是Care。但是什么样的代码是被author care过得呢？这里没有明确的指标可以依据，只能靠我们自己去判断。

#### 6. Ward Cunningham, Wiki发明人，Fit发明人，eXtreme Programming共同发明人. Design Patterns、OO思想领袖。The godfather of all those who care about code.

> You know you are working on clean code when each routine you read turns out to be pretty much what you expected. You can call it beautiful code when the code also makes it look like the language was made for the problem.

“Pretty much what you expected”，但是每个人expect的代码的样子都是不一样的，一个初学者expected的代码的样子可能相对也会比较初级，难道那个也能称作clean code吗？I might have a different opinion.

大家觉得，什么样的代码才是好代码呢？
