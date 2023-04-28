---
layout: post
title: "RxSwift底层原理及结合MVVM架构在项目中的应用"
date: 2020-12-10
tags: iOS
---

`RxSwift` 是 [ReactiveX](https://github.com/ReactiveX) 家族的重要一员, `ReactiveX` 是 `Reactive Extensions` 的缩写，一般简写为 `Rx`。`ReactiveX` 官方给`Rx`的定义是：`Rx`是一个使用可观察数据流进行异步编程的编程接口。

`RxSwift` 是 `Rx` 为 `Swift` 语言开发的一门函数响应式编程语言， 它可以代替`iOS`系统的 `Target Action` / `代理` / `闭包` / `通知` / `KVO`,同时还提供`网络`、`数据绑定`、`UI事件处理`、`UI的展示和更新`、`多线程`……

RxSwift：它只是基于 `Swift` 语言的 `Rx` 标准实现接口库，所以 `RxSwift` 里不包含任何 `Cocoa` 或者 `UI` 方面的类。
RxCocoa：是基于 `RxSwift` 针对于 `iOS` 开发的一个库，它通过 `Extension` 的方法给原生的比如 `UI` 控件添加了 `Rx` 的特性，使得我们更容易订阅和响应这些控件的事件

## 基本概念
---
要想充分理解RXSwift核心逻辑，那么首先必须要知道RXSwift里包含哪几个角色，以及它们的职责。

#### 命令式编程
命令式编程的主要思想是关注计算机执行的步骤，即一步一步告诉计算机先做什么再做什么

#### 响应式编程
响应式编程是一种和事件流有关的编程模式，关注导致状态值改变的行为事件，一系列事件组成了事件流。

#### 为什么要用它
* 开发过程中，状态以及状态之间依赖过多, RxSwift更加有效率地处理事件流，而无需显式去管理状态。在命令式编程中，状态变化是最难跟踪，最头痛的事。这个也是最重要的一点。

* 减少变量的使用，由于它跟踪状态和值的变化，因此不需要再申明变量不断地观察状态和更新值。

* 提供统一的消息传递机制，将Swift中的通知，action，KVO以及其它所有UIControl事件的变化都进行监控，当变化发生时，就会传递事件和值。

* 当值随着事件变换时，可以使用map，filter，reduce等函数便利地对值进行变换操作。

####被观察者(Observable)
它主要负责***产生事件***，实质上就是一个可被监听的序列(`Sequence`)。

`Observable<T> `这个类就是**Rx框架的基础**，我们称它为**可观察序列**。
>Observable<T> ` ==异步产生==>` event(element : T) 

![被观察者(Observable)类继承关系](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2020/12/06/16072382564981.png)

####观察者(Observer)
它主要负责**监听事件**然后对这个事件**做出响应**，或者说`任何响应事件的行为都是观察者`。

![观察者(Observer)类继承关系](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2020/12/06/16072383368134.png)


####订阅者(Subscriber)
事件的最终处理者

####管道(Sink)
**Observer 和 Observable 沟通的桥梁：**Sink相当与一个加工者，可以将源事件流转换成一个新的事件流，如果将事件流比作水流，事件的传递过程比作水管，那么Sink就相当于水管中的一个转换头。

![管道(Sink)类继承关系](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2020/12/06/16072384059160.png)


##代码解析
---
接下来我们结合以下很简单的代码来分析，逐步揭开RXSwift的神秘面纱。

```
//1:创建序列
 let ob = Observable<Any>.create { (observer) -> Disposable in
            // 3:发送信号
            observer.onNext("测试OnNext")
            observer.onCompleted()
            observer.onError(NSError.init(domain: "error！", code: 000, userInfo: nil))
            return Disposables.create()
        }
 
//2:订阅信息
 let _ = ob.subscribe(onNext: { (text) in
            print("订阅到:\(text)")    //text从哪里来的？
        }, onError: { (error) in
            print("error: \(error)")    //error从哪里来的？
        }, onCompleted: {
            print("完成")
        }) {
            print("销毁")
        }

```
在这里我们主要关注下
**create 闭包什么时候执行，**
**subscribe 闭包又是什么时候执行的**
也就是3->2这条线

####Create方法
```
 public static func create(_ subscribe: @escaping (AnyObserver<E>) -> Disposable) -> Observable<E> {
        return AnonymousObservable(subscribe)
    }
```
我们看到create 函数， 返回了一个`AnonymousObservable`实例，接着我们进入`AnonymousObservable`看它里面具体内容

```
// AnonymousObservable  Class
final fileprivate class AnonymousObservable<Element> : Producer<Element> {
    typealias SubscribeHandler = (AnyObserver<Element>) -> Disposable

    let _subscribeHandler: SubscribeHandler

    init(_ subscribeHandler: @escaping SubscribeHandler) {
        _subscribeHandler = subscribeHandler
    }

    override func run<O : ObserverType>(_ observer: O, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where O.E == Element {
        let sink = AnonymousObservableSink(observer: observer, cancel: cancel)
        let subscription = sink.run(self)
        return (sink: sink, subscription: subscription)
    }
}
```
在这里我们再次回顾下**Observable集成体系**
**（父类）**
**`ObservableConvertibleType`（完全的抽象）**
|
**`ObservableType`（ 处理subscribe）**
|
**`Observable`（处理 asObservable）**
|
**`Producer`（重载subscribe）**
|
**`AnonymousObservable`（处理run）**
**（子类）**

**PS：**由上面我们能看出，如果想自定义`Observable`通常只需要继承Producer， 并实现run方法就可以了。

在这里我们看到**Run方法**里面涉及了类`AnonymousObservableSink`,它作为Observer 和 Observable的衔接的桥梁我们看到**它本身遵守`ObseverType`协议**，与此同时**实现了`run`方法**。
> 那也就是说，**`sink`从某种程度来说也是`Observable`**
通过sink就可以完成从Observable到Obsever的转变。

总结下`create`方法主要工作：
* 创建`AnonymousObservable`对象，
* 用`_subscribeHandler `保存了闭包
* 写了`run`方法在内部创建了`AnonymousObservableSink `


####Subscribe方法
```
 public func subscribe(_ on: @escaping (Event<E>) -> Void)
        -> Disposable {
            let observer = AnonymousObserver { e in
                on(e)
            }
            return self.asObservable().subscribe(observer)
    }
```

上述代码只是入口，下面的才是核心**（Producer里面）**

```
// Producer Class
  override func subscribe<O : ObserverType>(_ observer: O) -> Disposable where O.E == Element {
        if !CurrentThreadScheduler.isScheduleRequired {
            // The returned disposable needs to release all references once it was disposed.
            let disposer = SinkDisposer()
            let sinkAndSubscription = run(observer, cancel: disposer)
            disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

            return disposer
        }
        else {
            return CurrentThreadScheduler.instance.schedule(()) { _ in
                let disposer = SinkDisposer()
                let sinkAndSubscription = self.run(observer, cancel: disposer)
                disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

                return disposer
            }
        }
    }
```
这里面我们看到了`Producer`调用了自己的`run`方法，而`AnonymousObservableSink`作为其子类重写了该方法，我们先去看下子类是如何写的。

```
// AnonymousObservable Class
    override func run<O : ObserverType>(_ observer: O, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where O.E == Element {
        let sink = AnonymousObservableSink(observer: observer, cancel: cancel)
        let subscription = sink.run(self)
        return (sink: sink, subscription: subscription)
    }
```

之前提到过`AnonymousObservableSink`，**注意`Sink`是持有`Observer`的**，从这也可以看出来
>**`Observerable`的`run`方法触发`Sink`的`run`方法**

接下来就要关注

`AnonymousObservableSink`方法。

```
    func run(_ parent: Parent) -> Disposable {
        return parent._subscribeHandler(AnyObserver(self))
    }
```

在这里我们再一次见到了`subscribeHandler`，这个`subscribeHandler`就是之前最开始的闭包！

```
 Observable<String>.create { observer -> Disposable in
            observer.onNext("测试")
            return Disposables.create()
        }
```
至此我们知道了create闭包是什么执行的了，接下来我们自然的把目光锁定到实体类`AnyObserver`，看看它里面究竟是如何实现的。

```
public struct AnyObserver<Element> : ObserverType {
    /// The type of elements in sequence that observer can observe.
    public typealias E = Element
    
    /// Anonymous event handler type.
    public typealias EventHandler = (Event<Element>) -> Void

    private let observer: EventHandler

    /// Construct an instance whose `on(event)` calls `eventHandler(event)`
    ///
    /// - parameter eventHandler: Event handler that observes sequences events.
    public init(eventHandler: @escaping EventHandler) {
        self.observer = eventHandler
    }
    
    /// Construct an instance whose `on(event)` calls `observer.on(event)`
    ///
    /// - parameter observer: Observer that receives sequence events.
    public init<O : ObserverType>(_ observer: O) where O.E == Element {
        self.observer = observer.on
    }
    
    /// Send `event` to this observer.
    ///
    /// - parameter event: Event instance.
    public func on(_ event: Event<Element>) {
        return self.observer(event)
    }

    /// Erases type of observer and returns canonical observer.
    ///
    /// - returns: type erased observer.
    public func asObserver() -> AnyObserver<E> {
        return self
    }
}
```
在这里我们看到其**内部的`Observer`其实是一个`EventHandler`**，并且在初始化的时候把外部传过来的`AnonymousObservableSink.on`赋值给了这个`Observer`，也就是说**`observer.onNext("测试")`最终会触发`AnonymousObservableSink.on`事件**

接着我们看下`AnonymousObservableSink`的`on`的具体实现

```
// AnonymousObservableSink.on

    func on(_ event: Event<E>) {
        #if DEBUG
            _synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { _synchronizationTracker.unregister() }
        #endif
        switch event {
        case .next:
            if _isStopped == 1 {
                return
            }
            forwardOn(event)
        case .error, .completed:
            if AtomicCompareAndSwap(0, 1, &_isStopped) {
                forwardOn(event)
                dispose()
            }
        }
    }
```
>**`AnonymousObservableSink`的`on`会调用其内部的`forwardOn`**

```
// Sink.forwardOn
    final func forwardOn(_ event: Event<O.E>) {
        #if DEBUG
            _synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { _synchronizationTracker.unregister() }
        #endif
        if _disposed {
            return
        }
        _observer.on(event)
    }

```
在这里还记得这个`_observer`的实体吗，看到下面代码你一定会回忆起来，
```
  public func subscribe(_ on: @escaping (Event<E>) -> Void)
        -> Disposable {
            let observer = AnonymousObserver { e in
                on(e)
            return self.asObservable().subscribe(observer)
    }
```
对，没错，这个`_observer`就是我们最初传进来的`subscribe`闭包的实体类`AnonymousObserver `~!

接着我们继续看下这个`AnonymousObserver`的`on`方法又是如何实现的
>**继承关系：`AnonymousObserver`->`ObserverBase`->`ObserverType`**

```
// ObserverBase.on
    func on(_ event: Event<E>) {
        switch event {
        case .next:
            if _isStopped == 0 {
                onCore(event)
            }
        case .error, .completed:
            if AtomicCompareAndSwap(0, 1, &_isStopped) {
                onCore(event)
            }
        }
    }
```
`ObserverBase.on`会触发`onCore`方法，看下子类的实现
```
// AnonymousObserver.onCore
    override func onCore(_ event: Event<Element>) {
        return _eventHandler(event)
    }
```
这里的`_eventHandler`还记得吗？它就是最初传进来的**订阅闭包**即：
```
            .subscribe { event in
               print(event.element)
        }
```

##总结一下
---

#### Observable-Create阶段：
* 创建`AnonymousObservable`
* 保存闭包(`subscribeHandler`)


#### Observable-Subscribe阶段：
* 创建`AnonymousObserver`
* 调用自己（`AnonymousObservable`）的`run`方法:（`AnonymousObserver`作为参数）
`AnonymousObservable`重写了`run`，它在方法里面创建了`AnonymousObservableSink`并在`sink`里保存了这个刚创建的`AnonymousObserver`
* 调用`AnonymousObservableSink`的`run`：`run`方法里用到`AnonymousObservable`的`_subscribeHandler`并传入`AnyObserver`，这里`AnonymousObservableSink.on`赋值给了`AnyObserver`内部的`EventHandler`成员`observer`

#### 执行阶段：
`AnyObserver.on` ----> `AnonymousObservableSink.on` ----> `AnonymousObservableSink.forwardOn` ----> `AnonymousObserver.on `---> `AnonymousObserver.onCore` ----> `_eventHandler(event)`


#### PS：
可以看出`Sink`在不同的阶段有着不同的身份

>* **Sink充当Observable**
> `Obsevable.subscribe` ---> `Obsevable.run` ----> `Sink.run `
这个过程通过Sink建立Obsevable和 Observer的联系

> * **Sink充当Observer**
> `AnyObserver.on` ----> `Sink.on` ----> `Observer.on` ---> `Observer.OnCore`


## 项目中的应用

RxSwift+Moya+MVVM

#### 对现有网络请求的改造

由于之前网络请求返回的都是一个对象，接入RxSwift，最好将返回的对象定义为`Observable`类型，这样我们的业务模块才能方便的订阅返回的数据

```
extension Reactive where Base: ZPMNetworkAgent {
    public func request<T: HandyJSON>(_ path: String, params: [String: Any]? = nil, modelType: T.Type) -> Single<T?> {
        return Single.create { [weak base] single in

            let cancle = base?.getCAPIURL(path, params: params, success: { (request) in


                let jsonData = request.responseJSONObject
                var model: T?
                if let json = jsonData, let jsonDic = json as? [String: Any] {
                    model = T.deserialize(from: jsonDic)
                }

                single(.success(model))
            }, failure: { (request) in
                single(.error(ZPMNetError().transError(error: (request.error as! NSError))))
            })

            return Disposables.create {
                cancle?.stop()
            }
        }
    }
}
```

#### 统一代码风格
ViewModel是MVVM架构模式与MVC架构模式最大的区别点。MVVM架构模式把业务逻辑从controller集中到了ViewModel中，方便进行单元测试和自动化测试

ViewModel的业务模型如下：

![](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2020/12/06/16072417690478.jpg)

viewmodel相当于是一个黑盒子，封装了业务逻辑，进行输入和输出的转换。
其中View、Model与MVC架构模式下负责的任务相同。controller由于业务逻辑移到了Viewmodel中，它本身担起了中间调用者角色，负责把View和Viewmodel绑定在一起。


```
protocol ViewModelType {
    associatedtype Input
    associatedtype Output
    
    func transform(input: Input) -> Output
}
```


## 推荐链接

[RxSwift社区](https://community.rxswift.org/)
[参考项目](https://github.com/khoren93/SwiftHub)




