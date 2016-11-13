---
layout: post
title: "Elm语言初体验"
date: 2016-11-13T23:24:02+08:00
comments: true
post_id: a-glimpse-of-elm
---

这个周末两天一直在看[Elm](http://elm-lang.org/)，这是一门函数式编程语言。所谓[函数式编程](https://en.wikipedia.org/wiki/Functional_programming)（Functional Programming，FP），跟[面向对象编程](https://en.wikipedia.org/wiki/Object-oriented_programming)（Object  Oriented，OO)是同一种类型的概念，是一种编程模式。具体什么叫函数式编程，它有什么特征，有什么优势，这里不作具体解释。只放出一个大胆的预言，就是在未来，尤其是在Server端，将会是函数式编程的天下。  
Anyway，Elm是一门函数式编程语言，它是一门静态的，强类型的语言，目前主要targeting在Web端，因此它有一个compiler可以编译成JS。  
目前，编译成JS的语言已经有非常多了，如[CoffeeScript](http://coffeescript.org/)，[TypeScript](https://www.typescriptlang.org/)，[PureScript](http://www.purescript.org/)等等，甚至很多现有的general purpose语言现在也有了一些工具，用来把它们编译成JS，如Clojure 有[ClojureScript](http://clojurescript.org/)，Scala有[Scalajs](https://www.scala-js.org/)，Ruby有[Opal](http://opalrb.org/)等等等等。  
为什么有这么多的新语言编译成JS，又有这么多工具把其它语言编译成JS呢？这些说白了其实都是想取代JS，这说明一个问题，那就是JS不够好，这个跟JVM上面有这么多语言原因是类似的。  
Elm也是出于同样的原因，它想要发明一种全新的Web语言，用于取代JS。那在这么多语言中，Elm的优势是什么？  
经过这两天的接触，我了解到它的优势有以下几个：  

1. No Runtime Error。”Undefined is not a function” 有没有遇到过？如果你做过一段时间前端开发，你不可能没有遇到过这个问题。类似的Runtime Error还有非常多。然而，Elm声称可以完成避免Runtime Error！这绝对是无可比拟的一个优势。美国的一家公司[NoRedLink](https://www.noredink.com/)声称，他们的产品上线一年多，目前遇到的Runtime Error是0个！这个还是非常有诱惑力的。
2. 纯粹的函数式编程，Immutable data和pure function，让你的代码变得干净，0耦合，不易出错，容易重构，易读，易测。这些都是JS界目前大力推进的方向啊，不然要[immutable.js](https://facebook.github.io/immutable-js/)干吗？要[Ramda](http://ramdajs.com/)干吗？要[RxJs](https://github.com/Reactive-Extensions/RxJS)干吗？
3. 有一个“官方”的[App architecture](https://guide.elm-lang.org/architecture/)，这是一个非常优秀的architecture，有多优秀？[Redux](http://redux.js.org/)大家都知道，现在其实已经是Web app实际上的标准architecture，然而Redux的思想其实是从Elm的architecture学来的！

总之，我觉得[这个视频](https://www.youtube.com/watch?v=txxKx_I39a8)里面有一句话总结得非常好，那就是Elm是现在整个JS界的发展趋势，然而JS还有很长一段路要走。

那Elm完美的吗？当然不是，我这两天接触下来，发现它仍然处于比较年轻的状态，虽然官方声称它已经production ready，也确实有好一些公司已经用于正式的产品中。然而我相信都不是很容易的事情。目前来说，我觉得有以下几个问题需要极需改善：

1. 社区太小，这可能是目前最大的问题，在国内更是如此。如果你在百度上面搜”Elm“，出来的结果基本都不是Elm语言相关的。当然这也不赖Elm，因为你在百度上搜其它东西，基本出来的结果也不是你想要的。
2. 函数式编程与OO或[过程式编程](https://en.wikipedia.org/wiki/Procedural_programming)差别太大，上手真不是很容易，这会在很大的程度上影响到它的采用率。
3. Elm语言和编译器本身也有一些东西有等完善，如目前还不支持[String interpolation](https://en.wikipedia.org/wiki/String_interpolation)，[elm reactor](https://github.com/elm-lang/elm-reactor)(类似于[webpack-dev-server](https://webpack.github.io/docs/webpack-dev-server.html)）不支持hot reload，其实之前是支持的，后来发现了bug，就去掉了。Time travel debugger也是，本来是有的，后来发现有bug又去掉了。当前这些都不是很严重的问题，比如hot reload只是节省了你手动刷新页面的功力而已。然而从这些”原来有，后来发现有bug又去掉“这些事情中就能说明这个东西还不是很稳定的问题。

接下来打算在公司的内部小工具上面试用一下，因为Elm文件是可以跟JS交互的，所以应该不是太大的问题，大不了再用回JS嘛。但是函数式编程作为未来的发展趋势，还是要了解一下的。所以，总体来说，我觉得这是一门非常值得学习的语言。

如果你对Elm感兴趣，欢迎加我微信(875156226)，共同交流。
