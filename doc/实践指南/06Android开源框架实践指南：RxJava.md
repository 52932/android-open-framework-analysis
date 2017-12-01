# Android开源框架实践指南：RxJava

**关于作者**

>郭孝星，程序员，吉他手，主要从事Android平台基础架构方面的工作，欢迎交流技术方面的问题，可以去我的[Github](https://github.com/guoxiaoxing)提issue或者发邮件至guoxiaoxingse@163.com与我交流。

**文章目录**

- 一 事件模型
- 二 操作符
- 三 线程调度
- 四 应用场景

>A library for composing asynchronous and event-based programs using observable sequences for the Java VM.

要更深入的理解RxJava/RxAndriod的实践原理，我们就要先理解ReactiveX相关基础理论。

>ReactiveX是Reactive Extensions的缩写，它是一个可以使用可观察的数据流进行异步编程的编程接口，它结合了观察者模式，迭代器模式与函数式编程的精华。

-[ReactiveX官方网站](http://reactivex.io/)

说到ReactiveX我们不得不提一下函数式编程，ReactiveX就是函数式编程的一种良好实践。

>函数式编程同命令式编程，面向对象编程一样，也是编程范式的一种，说的是一种编程方法，将函数视为一等公民，将所有计算都描述为一种表达式
求值，函数可以在任何地方定义和调用，就像我们定义变量那样，我们还可以对函数进行任意组合。

更多关于函数式编程的讨论可以参见：

- [函数式编程初探](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)
- [什么是函数式编程思维](https://www.zhihu.com/question/28292740)

ReactiveX的优点：

- 扩展了观察者模式用于支持数据与事件序列，添加了一些操作符用于组合和操作这些序列，而无需去关注底层的机制，例如：线程、同步、线程安全、并发数据结构与非阻塞IO等。
- 函数式编程风格，对可观察数据流使用无副作用的输入输出函数，避免了程序里错综复杂的状态。
- 解决了多层回调的痛点，我们通常对于异步的实现就是"接口+回调"的方式，这在单层异步里是可用的，但是面对嵌套的异步组合就显得十分笨拙。😄

```
那一天人们又回想起了都多层回调与迷之缩进支配的恐惧。☹️

```

## 一 事件模型

RxJava2依然以观察者模式为基本骨架，有两种观察者模式：

- Observable（被观察者）/ Observer（观察者）：不支持背压
- Flowable（被观察者）/ Subscriber（观察者）：支持背压

>背压（Backpressure）指的是在异步场


## 二 操作符

## 三 线程调度


上一篇我们讲了操作符的变换与组合，这一篇我们来讲一下线程调度，还是先贴一下RxJava的事件模型图。

事件的上下游并不总是运行在同一个线程中，一般的，我们通常希望在子线程中处理费时操作，在主线程中更新UI。

默认情况下事件上游在哪个线程中发送事件，事件下游就在哪个线程接收事件。

例如：

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> e) throws Exception {
                Log.d(TAG, "subscribe() - thread: " + Thread.currentThread().getName());
                e.onNext(1);
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d(TAG, "onSubscribe() - thread: " + Thread.currentThread().getName());
            }

            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext() - thread: " + Thread.currentThread().getName());
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "onError() - thread: " + Thread.currentThread().getName());
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete() - thread: " + Thread.currentThread().getName());
            }
        });
```

输出打印

<img src="https://github.com/guoxiaoxing/android-open-framwork-analysis/raw/master/art/rxjava/thread_normal.png"/>

可以看出它们都工作在主线程中。

RxJava内部使用线程池来维护线程，拥有较高的效率。

RxJava指定线程

- subscribeOn()：指定Observable所在线程。
- observeOn()：指定Observer所在线程。

RxJava内置线程模型

- Schedulers.single()：返回一个默认的，共享的，单线程的Scheduler实例，它通常用在那些有较强执行顺序要求的后台线程。
- Schedulers.newThread()：返回一个默认的，共享的Scheduler实例，它创建一个常规的线程。
- Schedulers.trampoline()：返回一个默认的，共享的Scheduler实例，它以FIFO的方式来执行任务。
- Schedulers.io()：返回一个默认的，共享的Scheduler实例，它代表IO操作的线程，通常用于网络，读写文件等IO密集型操作。
- Schedulers.computation()：返回一个默认的，共享的Scheduler实例，它通常用来执行复杂的计算操作。
- AndroidSchedulers.mainThread()：Android主线程


## 四 应用场景

## 附录

- [Rxjava官方文档](http://reactivex.io/intro.html)
- [ReactiveX官方网站](http://reactivex.io/RxJava/2.x/javadoc/overview-summary.html)
- [ReactiveX官方文档中文版](https://mcxiaoke.gitbooks.io/rxdocs/content/Intro.html)