---
layout: post
title: 'RxJava源码分析（一）：Observable的几种创建方式'
date: 2016-06-14T08:36:47+08:00
comments: true
post_id: rxjava-source-reading-1-creating-observable
---

相信写Android的，写Java的人应该都听说过RxJava这个革命性的东西了，很多人也了解这个东西怎么使用。网上也有很多优秀的介绍性的文章，如果你还不了解这个东西，或者是不熟悉它的用法的话，我会建议先看[这个系列](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)的文章，然后再看[这篇](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)。尤其是后者绝对强烈推荐，其实我个人对RxJava的看法，刚开始只是觉得这个东西写起来语法非常简洁，非常优雅，可以在很大程度上面减少一些外部变量的使用，非常Functional。但是等到我看了那篇文章以后，我才发现，我只是发挥了RxJava不到一成的功力，获得了它不到一成的优点。它还有那么强大的力量，隐藏在冰山水下。另外，国内也有广受好评的[扔物线](https://github.com/rengwuxian)写的[给Android开发者的RxJava详解](http://gank.io/post/560e15be2dca930e00da1083)。这些足够让你对RxJava有一个比较深入的了解，此外，[扔物线](https://github.com/rengwuxian)的文章也有一定程度的原理讲解和源码分析，可谓真正的深入浅出。

Anyway，这里我们就不介绍RxJava的用法和特点了，我们简单看看它的源代码。

先看第一部分，Observable的创建。现在大部分介绍RxJava的文章、尤其是介绍RxJava在Android上面的使用的文章，都往往会结合第三方的library，比如[Retrofit](http://square.github.io/retrofit/)。里面介绍Observable的使用方式是通过Retrofit的返回结果来获得一个Observable，再操作这个Observable。这容易给我们一种错觉，就好像我们自己从来不用，也不能创建Observable一样。然后其实不是这样的，我们完全可以自己创建Observable。常见的创建Observable的方式有以下几种：

1. `Observable.just(...)`  
比如：

```java
Observable.just("xiaochuang"); //Create an Observable that has only one item: "xiaochuang"
Observable.just("xiaochuang", "handsome"); //Create an Observable that has 2 items: "xiaochuang" and "handsome"
Observable.just("xiaochuang", "is", "handsome"); //Create an Observable that has 3 items: "xiaochuang", "is" and "handsome"
//...you can pass in up to 10 items
```

2. `Observable.from(an_array_or_iterable);`  
比如：

```java
Observable.from(new String[]{"xiaochuang", "is", "handsome"});  //Create an Observable that get data from an array

List<String> items = new ArrayList<>();
items.add("xiaochuang");
items.add("is");
items.add("handsome");
Observable.from(items);  //Create an Observable that get data from the List "items"
```

3. `Observable.create(OnSubscribe);`  
这个就最灵活也最强大了，使用这种方法，你可以把任何东西变成Observable。比如：

```java
Observable.create(new OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("xiaochuang");  //你想要给最终的Subscriber通知什么数据，就在这里调用subscriber的onNext
                subscriber.onNext("is");
                subscriber.onNext("handsome");
                subscriber.onCompleted();
            }
        }); //Create an Observable that emit 3 Strings: "xiaochuang", "is" and "handsome"
```
我个人使用这种方式的常见场景，就是把现有的基于`Callback`的API转换成Observable的形式，比如，你有如下的网络请求方法：

```java
public class UserModel {
    public void loadUsers(Callback callback) {
        //...
    }
    public interface Callback {
        void onSuccess(List<User> users);
        void onFailure(int code, String msg);
    }
}
```
Client使用这个`UserModel#loadUsers`的时候，必须传进来一个`Callback`，如果你想添加一个返回Observable的`loadUsers()`方法，可以使用这里说的`Observable.create()`方法来创建一个Observable：

```java
public class UserModel {
  public void loadUsers(Callback callback) {
    //...
  }

  public Observable<List<User>> loadUsers() {
      return Observable.create(new OnSubscribe<List<User>>() {
          @Override
          public void call(final Subscriber<? super List<User>> subscriber) {
              loadUsers(new Callback() {
                  @Override
                  public void onSuccess(List<User> users) {
                      if (subscriber.isUnsubscribed()) return;  //这里要记得判断是否unsubscribe了

                      subscriber.onNext(users);
                      subscriber.onCompleted();
                  }

                  @Override
                  public void onFailure(int code, String msg) {
                      if (subscriber.isUnsubscribed()) return;  //这里要记得判断是否unsubscribe了

                      subscriber.onError(new RequestFailureException(code, msg)); //这里的RequestFailureException是自己定义的一个Exception，封装了return code和error msg等信息
                  }
              });
          }
      });
  }

  //Callback interface definition
}
```

下面，我们来看下这几种方式创建Observable的源代码。  
首先是`Observable.just()`这种方式。从使用的角度来看，好像一个参数的`just()`和多个参数的`just()`没什么区别，但是看代码以后你会发现，他们的还是略有不同的：

```java
public final static <T> Observable<T> just(final T value) {
    return ScalarSynchronousObservable.create(value);
}

public final static <T> Observable<T> just(T t1, T t2) {  
    return from(Arrays.asList(t1, t2));  //其它多个参数的just方法也是类似的，先变成List，然后调用from(List);
}


```
对于只有一个参数的`Observable.just()`，调用了`ScalarSynchronousObservable.create()`，同时将传给`just()`的value传给了 `ScalarSynchronousObservable.create()` 它的代码如下：

```java
public final class ScalarSynchronousObservable<T> extends Observable<T> {
    public static final <T> ScalarSynchronousObservable<T> create(T t) {
        return new ScalarSynchronousObservable<T>(t);
    }
}
```

它只是直接调用 `ScalarSynchronousObservable` 的构造方法，把传入的这个value传给这个构造方法，创建了一个 `ScalarSynchronousObservable` 对象。再来看看 `ScalarSynchronousObservable` 的构造方法：

```java
public final class ScalarSynchronousObservable<T> extends Observable<T> {
    private final T t;

    protected ScalarSynchronousObservable(final T t) {
        super(new OnSubscribe<T>() {
            @Override
            public void call(Subscriber<? super T> s) {
                s.onNext(t);
                s.onCompleted();
            }
        });
        this.t = t;
    }
```

可以看到，它只是`new`了一个 `OnSubscribe` 对象，然后传给父类的构造方法，同时将传入的value保存起来。  
也就是说，最终，这种方法是调用了Observable的构造方法，传给了一个`OnSubscribe`实例：

```java
public class Observable<T> {

    final OnSubscribe<T> onSubscribe;
    protected Observable(OnSubscribe<T> f) {
        this.onSubscribe = f;
    }
}
```

这个构造方法做的事情很简单，就是把传入的`OnSubscribe`对象保存起来而已。实事上，这是Observable唯一的构造方法。所以大家可想而知，最后肯定是调到这里来的，接下来我们就看看，每种方式都是怎么最后调到这里来的，以及为什么要这么做。

我知道到这里，你很想知道这个`OnSubscribe`到底是什么东西，它跟Observable是什么关系，跟`Subscriber`又是什么关系，跟`Observer`双是什么关系？但是这是先不讲这个东西。先来证实下我们的结论，那就是，所有的创建Observable的方式最终都是调用了Observable的这个只有一个`OnSubscribe`参数的构造方法。所以下面我们看看有两个参数的`Observable.just(T t1, T t2)`;

```java
public final static <T> Observable<T> just(T t1, T t2) {
    return from(Arrays.asList(t1, t2));
}
```

可见，这个方法的实现是先把这两个参数转化成一个`List`，然后调用`Observable.from(Iterable iterable)`;  
其它有3个、4个。。。10个参数的`just()`也是一样的实现，而`Observable.from(Iterable iterable)`的实现是：

```java
public final static <T> Observable<T> from(Iterable<? extends T> iterable) {
    return create(new OnSubscribeFromIterable<T>(iterable));
}
```

这里是实现是，用传入的`iterable`创建了一个 `OnSubscribeFromIterable` , 然后调用`Observable.create(OnSubscribe onSubscribe)`; 从这里我们知道，`OnSubscribeFromIterable`应该是实现了`OnSubscribe`这个`interface`的一个concrete class。看定义：

```java
public final class OnSubscribeFromIterable<T> implements OnSubscribe<T> {

    final Iterable<? extends T> is;

    public OnSubscribeFromIterable(Iterable<? extends T> iterable) {
        if (iterable == null) {
            throw new NullPointerException("iterable must not be null");
        }
        this.is = iterable;
    }
}
```

证实了我们的猜测。再回到 `create(new OnSubscribeFromIterable<T>(iterable))`，看看`create()`的代码：

```
public final static <T> Observable<T> create(OnSubscribe<T> f) {
    return new Observable<T>(hook.onCreate(f));
}
```

这里的`hook.onCreate(f)`的定义如下:

```
public abstract class RxJavaObservableExecutionHook {
    public <T> OnSubscribe<T> onCreate(OnSubscribe<T> f) {
        return f;
    }
}
```

这里，`onCreate()`的具体实现其实不重要，重要的是它的返回值，一个`OnSubscribe`。所以说，这种创建Observable的方法，最终也是调用到了Observable只有一个`OnSubscribe`的构造方法。

剩下的`Observable.from(Iterable iterable)`，我们分析过。而`Observable.from(T[] array)`:

```
public final static <T> Observable<T> from(T[] array) {
    return from(Arrays.asList(array));
}
```

可见只是调用了`Observable.from(Iterable iterable)`。最后的`Observable.create(OnSubscribe)`我们也分析过。

综上所述，所有的创建Observable的方法最终都会调到Observable唯一的构造方法：

```
public class Observable<T> {
    final OnSubscribe<T> onSubscribe;

    protected Observable(OnSubscribe<T> f) {
        this.onSubscribe = f;
    }
}
```
对于`Observable.just(T t)`来说，是通过`ScalarSynchronousObservable.create(T t);` `ScalarSynchronousObservable` 调用 `ScalarSynchronousObservable` 的构造方法，`ScalarSynchronousObservable`再通过`super`去调`Observable`的构造方法。而对于`Observable.just(T t1, T t2, ...)`、`Observable.from(T[] array)`、`Observable.from(Iterable iterable)`都会调到`Observable.create(OnSubscribe onSubscribe)`，然后`Observable.create(OnSubscribe onSubscribe)`再调到Observable的构造方法。
它们之间的调用关系图如下：

![](http://jietu-10024907.file.myqcloud.com/lzuqmlbvitvrdebqxlpfinnqclfactct.jpg)

不同的创建方法，有不同的`OnSubscribe`的实现。对于`Observable.just(T t)`来说，是在`ScalarSynchronousObservable`的构造方法里面自己new的一个`OnSubscribe`：

```
protected ScalarSynchronousObservable(final T t) {
    super(new OnSubscribe<T>() {
        @Override
        public void call(Subscriber<? super T> s) {
            s.onNext(t);
            s.onCompleted();
        }
    });
}
```
对于`Observable.just(T t1, T t2, ...)`、`Observable.from(T[] array)`、`Observable.from(Iterable iterable)`来说，使用的是`OnSubscribeFromIterable`：

```
public final class OnSubscribeFromIterable<T> implements OnSubscribe<T> {

    final Iterable<? extends T> is;

    public OnSubscribeFromIterable(Iterable<? extends T> iterable) {
        if (iterable == null) {
            throw new NullPointerException("iterable must not be null");
        }
        this.is = iterable;
    }

    @Override
    public void call(final Subscriber<? super T> o) {
        final Iterator<? extends T> it = is.iterator();
        if (!it.hasNext() && !o.isUnsubscribed())
            o.onCompleted();
        else
            o.setProducer(new IterableProducer<T>(o, it));
    }
}
```

那么现在剩下的唯一的问题是，`OnSubscribe` 到底是什么样的一个东西？看代码：

```
/**
 * Invoked when Observable.subscribe is called.
 */
public interface OnSubscribe<T> extends Action1<Subscriber<? super T>> {
    // cover for generics insanity
}

/**
 * A one-argument action.
 */
public interface Action1<T> extends Action {
    void call(T t);
}
```

可以看到，`OnSubscribe` 是一个interface，继承自`Action1`，而`Action1`又继承自一个`Action`。在RxJava里面，一个`Action`可看作一个类似于`Runnable`的接口定义，`Action`有11个子类，分别是`Action0`、`Action1`。。。`Action9`、`ActionN`，他们都只有一个 `call()` 方法，这个`call()`方法的参数个数在这几个`Action`的子接口里面分别是0个、1个。。。9个和不限个数的多个。

话说回来，`OnSubscribe` 继承自`Action1`，它只有一个`call(T t)`方法，通过泛型的声明可以看出，`OnSubscribe` 的这个`call()`方法的参数是一个`Subscriber`。这个跟我们前面看到的是一样的。我们前面看到，这个类的对象是传给Observable的构造方法，用来构造出一个Observable。那它跟Observable是什么关系呢？通过这个类的doc可以看出，以及Observable的构造方法的doc

```
public class Observable<T> {

    final OnSubscribe<T> onSubscribe;

    /**
     * Creates an Observable with a Function to execute when it is subscribed to.
     * <p>
     * <em>Note:</em> Use {@link #create(OnSubscribe)} to create an Observable, instead of this constructor,
     * unless you specifically have a need for inheritance.
     *
     * @param f
     *            {@link OnSubscribe} to be executed when {@link #subscribe(Subscriber)} is called
     */
    protected Observable(OnSubscribe<T> f) {
        this.onSubscribe = f;
    }
}
```

可以看出，其实`OnSubscribe`的作用就是，当用它构造出来的Observable的 `subscribe()`方法被调用的时候，它的`call()`方法会被调用。这个可以通过`subscribe()`方法的源代码得到验证：

```
public final Subscription subscribe(Subscriber<? super T> subscriber) {
     return Observable.subscribe(subscriber, this);
 }

 private static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {
     //...
     // if not already wrapped
     if (!(subscriber instanceof SafeSubscriber)) {
         // assign to `observer` so we return the protected version
         subscriber = new SafeSubscriber<T>(subscriber);
     }
     try {
         // allow the hook to intercept and/or decorate
         hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber);
         return hook.onSubscribeReturn(subscriber);
     } catch (Throwable e) {
         //...
     }
 }
```

关键的一行是 `hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber);` 其中 `hook.onSubscribeStart(observable, observable.onSubscribe)`的实现很简单，

```
public <T> OnSubscribe<T> onSubscribeStart(Observable<? extends T> observable, final OnSubscribe<T> onSubscribe) {
    // pass-thru by default
    return onSubscribe;
}
```

这里直接把传入的`OnSubscribe`给return了，所以上面的那行代码相当于： `onSubscribe.call(subscriber);`，同时传给`call()`方法的`subscriber()`就是我们在`subscribe()`的时候传入的`subscriber()`。(Well, not exactly)，验证了我们前面的结论。最后，再回过头去看前面几种创建Observable方法的`OnSubscribe`的实现的话，你会发现，他们都是在某个点相应的去调传入的`subscriber`的`onNext()`、`onComplete()`或`onError()`。这个估计你也猜得到，就是我们为什么`subscribe`的时候，如果有数据的话，会在`onNext()`收到数据，`onComplete()`或`onError()`会得到调用的原因。

这里有一个比较有意思的点是，为什么要把observable的构造方法设置为protected，然后通过工厂方法来创建？为什么不直接把constructor直接设为public，让外面可以访问到呢？这个其实在[Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683)的第一条就讲到了。使用[factory method](https://en.wikipedia.org/wiki/Factory_method_pattern)有以下的好处：

1. 可读性更高: `Observable.from()`、`Observable.just()`方法等等除了使用起来更方便之外，可读性也比自己创建自应的`OnSubscribe`然后传给Observable的构造方法要高很多。
2. 实现缓存: 不用每次都创建一个新的对象。当然这点在我们看到的例子里面没有相应的应用。
3. 可以返回子类，方便以后拓展，比如说`just(T t)`方法返回的是Observable的子类 `ScalarSynchronousObservable`，而不是Observable本身。  

此外，我觉得还有个原因就是实现像代码里面的hook这种功能，给你进一步控制Observable创建的过程。比如说在hook里面打log，实现类似decorator pattern的作用等等。

这篇小文简单分析了一下几种创建Observable的方法的源码，当然Observable创建的方法有[很多](https://github.com/ReactiveX/RxJava/wiki/Creating-Observables)，这些都算是RxJava的Operators的一部分。但无论是哪种方式，可以预见的是，它们都会构建出一个OnSubscribe对象，然后传给Observable的构造方法。在这个OnSubscribe对象的`call(Subscriber subscriber)`方法里面，它需要在某个点通过某种方式调用传入的`subscriber` 的 `onNext()`，`onComplete()`或`onError()`方法。按照这个思路去看其它的Observable创建方法的代码，应该会清楚一些。