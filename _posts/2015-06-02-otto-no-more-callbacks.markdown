---
layout: post
title: "Otto: No More Callbacks"
date: 2015-06-02T10:21:07+08:00
tags: [tech, java, android]
comments: true
---

Otto: No more callbacks
你要做一个todo app，有一个Activity里面有一个ListView显示你所有的`task`，你的数据存储在服务器。假设你没采用任何的软件架构（MVC、MVP、MVVM等等），每次app打开的时候，你从服务器把数据load下来，load完了以后，通过callback把数据传给Activity，然后显示到listview里面。代码结构大概是这个样子  

```Java
public class TasksActivity extends Activity {
	private ListView mListView;
	//...
	private void loadTasks() {
		TaskModel model = new TaskModel();
		model.setTaskCallback(new TaskCallback() {
			public void onError(String msg, int code) {
				// handle error
			}

			public void onSucceed(List<Task> tasks) {
				updateTaskList(tasks);
			}
		});
		model.loadTasks();
	}

	private void updateTaskList(List<Task> tasks) {
		//Update the task list view
	}
	// other code
}

public class Task {
	// title, label, ..., and their getters and setters.
}

public class TaskModel {
	public void loadTasks() {
		//load tasks from server, when result returns, check if callback is not null and call callback
	}

	private TaskCallback mTaskCallback;
	public void setTaskCallback(TaskCallback callback) {
		this.mTaskCallback = callback;
	}

	public static interface TaskCallback {
		public void onError(String errorMsg, int code);
		public void onSucceed(List<Task> tasks);
	}
}
```

这里，你用的是callback来通知Activity，task已经load好了。这种callback的方式是java里面非常传统的方式，尤其是涉及到异步的时候。这种方式虽然能work，但却是非常麻烦，也非常丑陋的方式。  
首先，你要定义Callback接口，然后在用的地方实现这个接口。  
再次，你要在你的model里面申明一个callback的成员变量，在用的时候，还要判断一下成员变量是不是空的。当然你可以用[NullObject](http://en.wikipedia.org/wiki/Null_Object_pattern) 模式，但这也需要额外的工作。  
最后，当你发现定义的callback接口要变的时候，你要改动的地方可能非常大，因为你可能有很多的地方实现了这个接口，这是非常烦人也是非常容易出错的工作。  
这一来二去，当你的model比较多的时候，你会变得非常烦，这完全就是体力劳动啊！  
那有没有更好地方式来解决这个问题呢？你可能会想到是`Handler`或者是`BroadcastReceiver`，然而他们也不是很好用的东西，都要定义一些东西，判断message，判断action，传递一些引用等等。  
这里介绍一个library叫[Otto](https://github.com/square/otto)，这是Square这个公司的一个开源项目，它是一个EventBus的library。简单的来说，它类似于定义了一套Observer Pattern的便捷的实现方式。你在某一个地方使用`@Subscribe` 表示你要处理某一种事件，在某个地方用post发布这一种事件，那么前面用`@Subscribe`修饰的方法就可以自动得到调用。不用定义接口，不用实现接口，一切都很直观，如你所愿。  
关于Otto的使用[官网](http://square.github.io/otto/)说的非常清楚。  
在这里简单地用Otto重写一下上面的代码，让大家感受一下。  

```Java
public class TasksActivity extends Activity {
	private ListView mListView;
	public void onCreate() {
		OttoHelper.register(this); //对于Subscriber来说，register和unregister是有必要的
	}
	//...
	private void loadTasks() {
		new TaskModel().loadTasks();
	}

	// 用@Subscribe 来表示用这个方法处理某种事件，这种事件就是你的方法的参数。
	// 此外，public void是必须的，方法名可以自己随便取
 	@Subscribe
	public void onTaskLoaded(TaskLoadedEvent event) {
		List<Task> tasks = event.tasks;
		// Update the task list view
	}

	@Subscribe
	public void onTaskLoadError(TaskLoadErrorEvent event) {
		//handler error
	}

	public void onDestroy() {
		OttoHelper.unregister(this)
	}
}

public class Task {
	// title, label, ..., and their getters and setters.
}

public class TaskModel {
	public void loadTasks() {
		// Load task from server
		OttoHelper.post(new TaskLoadedEvent(tasks)); //If load succeed
		OttoHelper.post(new TaskErrorEvent(errorMsg, code); //If load error
	}
}

//Bus 一般是用作单例的，所以有一个helper会很方便
public class OttoHelper {
	private static final Bus sBusInstance = new Bus();
	public void static register(Object obj) {
		sBusInstance.register(obj);
	}

	public void static unregister(Object obj) {
		sBusInstance.register(obj);
	}

	public static void post(Object event) {
		sBusInstance.post(event);
	}

}

public class TaskLoadedEvent {
	public final List<Task> tasks;
	public TaskLoadedEvent(List<Task> tasks) {
		this.tasks = tasks;
	}
}

public class TaskLoadErrorEvent {
	//final field errorMsg and code, and construction, just like TaskLoadedEvent
}
```

需要说明的是，这里只处理了一种事件（load task），所以Otto的优势还不是很明显，然而你依然可以明显感受到的是，`TaskModel`和`TasksActivity`之间的耦合性更弱了。他们之间除了都用了`Task`和`OttoHelper`这两个类以及用来传递事件的Event类，再没有其他共同的东西。  

当事件多了起来，Otto的优势就会非常明显，增加的只是Event这些轻量级的POJO而已。  
非常需要的一点是，最好每一种事件都使用特殊的Event类，千万不要使用常见的类，比如String等等，更不要使用Object，因为Otto判断Subscribe方法所处理的事件是通过方法的参数来判断的。只要 `instance of`为true，那么这个方法就会得到调用。举个例子，在上面的TasksActivity里，假如你有另外一个Subscribe函数，它的参数是Object：  

```Java
public class TasksActivity extends Activity {
	//...
	@Subscribe
	public void onTaskLoaded(TaskLoadedEvent event) {
		List<Task> tasks = event.tasks;
		// Update the task list view
	}

	@Subscribe
	public void onTaskLoadError(TaskLoadErrorEvent event) {
		//handler error
	}

	@Subscribe
	public void onSomeEvent(Object event) {
		//handle some-event
	}
}
```

在上面的事件中，bus post的任何事件，主要TasksActivity没有unregister，那么`onSomeEvent`就会得到调用，比如task load成功了，你post了TaskLoadedEvent，那么除了`onTaskLoaded`会得到调用的之外，`onSomeEvent`也会得到调用，这很可能不是你想要的。  
我曾经就犯了这个愚蠢的错误，更搞笑的是，我在`onSomeEvent`里面直接finish activity了，结果每一次别的事件过来，Activity就退出了，我以为程序奔溃了，找了半天都找不出原因，因为你正常finish activity是不会有error log的，最后终于怀疑到它头上来，然后才解决了这个问题。  
把 `compile 'com.squareup:otto:1.3.7’`加到你的dependencies里面，试用以下，我敢打赌，你再也不会定义callback了，Callbacks should be dead!.  
当然，除了Otto，还有其他的一些library，比如greenbot开源的EventBus，功能貌似比Otto更强，但是我个人选择Otto，因为信仰Square，哈哈。。  

### Disclaimer
这个例子只是用来说明Otto相对于callback的好处，对于一个正常的类似于上面的例子的app，这篇文章中提到的model和activity的交互方式并不是最好的方式，有其他更好的比如Functional Reactive Programming（RxJava）。同时，我们应该使用MVP模式，把Activity当做View，而不是直接在里面调用model。  

如果有任何意见或建议，或者是发现文中有任何问题欢迎留言！  
个人博客：http://www.chriszou.com 欢迎交流！  
