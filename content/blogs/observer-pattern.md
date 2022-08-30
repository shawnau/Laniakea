---
title: "观察者模式"
date: 2022-08-30T16:56:33+08:00
draft: false
description: C# Observer Pattern Tutorial
draft: false
hideToc: false
enableToc: true
enableTocContent: true

tags:
- Observer
categories:
- C#
series:
- Design Pattern
---
观察者 - 被观察者是一个常见的设计模式.

<!--more-->

这里考虑一种场景: 被观察者是报刊, 观察者是订阅报刊的人. 一旦有新闻, 订阅者都可以收到报刊的通知.

## 定义消息, 观察者和被观察者
### 消息与EventArgs
第一步是定义消息的格式, 以及观察者和被观察者的行为.
```csharp
public class NotifyEventArgs : EventArgs
{
    public string Info { get; set; } = string.Empty;
    public DateTime Time { get; set; }
}
```
首先定义消息, 这里继承了`EventArgs`. C#在语言上就提供了对消息的的支持, 其中[`EventArgs`](https://docs.microsoft.com/en-us/dotnet/api/system.eventargs?view=net-6.0) 包装了消息, [`EventHandler`](https://docs.microsoft.com/en-us/dotnet/api/system.eventhandler?view=net-6.0) 包装了一个`delegate`, 抽象了处理`EventArg`的逻辑, 也就是观察者收到消息之后需要做的事情. 下一小节在观察者的接口中就定义了这个`delegate`的行为.

为了简单起见, 这里消息就只包含两个内容: 消息本身以及消息发出的时间.
### 观察者接口
```csharp
public interface IObserver
{
    public void NotifyEventHandler(object? notifier, NotifyEventArgs args);
}
```
观察者只需要做一件事, 就是收到消息之后做他想做的事情. 这里函数签名与`EventHandler`一致, 被观察者会把它自身传给第一个参数供观察者使用, 第二个参数是`NotifyEventArgs`, 就是消息内容.
### 被观察者接口
```csharp
public interface IObservable
{
    public void Register(IObserver observer);
    public void OnNotified(NotifyEventArgs args);
}
```
被观察者需要做两件事:
1. `Register`: 订阅观察者 
2. 需要通知观察者的时候, 调用`OnNotified`, 并且传入消息

## 被观察者实现
```csharp
public class Observable : IObservable
{
    internal event EventHandler<NotifyEventArgs> NotifyEventHandlers = delegate {};

    public virtual void OnNotified(NotifyEventArgs args) => NotifyEventHandlers(this, args);

    public void Register(IObserver observer)
    {
        this.NotifyEventHandlers += observer.NotifyEventHandler;
    }

    public void Publish()
    {
        var args = new NotifyEventArgs 
        {
            Info = $"[Breaking News!]", 
            Time = DateTime.Now 
        };

        OnNotified(args);
    }

    public override string ToString() => "New York Times";
}
```
1. `NotifyEventHandlers` 可以视为一个集合, 保存了所有观察者注册的行为(`delegate`), 在需要发送消息的时候, 所有行为都会被依次调用, 这样观察者们会被依次通知.
2. `OnNotified`调用`NotifyEventHandlers`里注册的行为.
3. `Register`将观察者的行为添加到`NotifyEventHandlers`中.
4. `Publish`构造`EventArgs`并触发`OnNotified`, 发送消息给所有观察者.

## 观察者实现
```csharp
public class SubscriberAlice : IObserver
{
    public string Name = "Alice";

    public void NotifyEventHandler(object? notifier, NotifyEventArgs args)
    {
        Console.WriteLine($"{this.Name} received {args.Info} from {notifier?.ToString()} at {args.Time}.");
    }
}

public class SubscriberBob : IObserver
{
    public string Name = "Bob";

    public void NotifyEventHandler(object? notifier, NotifyEventArgs args)
    {
        Console.WriteLine($"{this.Name} received {args.Info} from {notifier?.ToString()} at {args.Time}.");
    }
}
```
构造了两个观察者, `Alice`和`Bob`, 他们做的事情就是把收到的消息打印出来.

## 启动程序
```csharp
class Program
{
    static void Main(string[] args)
    {
        // 初始化被观察者
        var notifier = new Observable();
        
        // 初始化观察者
        var alice = new SubscriberAlice();
        var bob = new SubscriberBob();

        // 注册观察者
        notifier.Register(alice);
        notifier.Register(bob);

        // 通知观察者
        notifier.Publish();
    }
}
```

程序输出
```txt
Alice received [Breaking News!] from New York Times at 8/30/2022 3:37:14 PM.
Bob received [Breaking News!] from New York Times at 8/30/2022 3:37:14 PM.
```

### Reference
[Observer Design Pattern | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/events/observer-design-pattern)
[How to: Raise and Consume Events | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/events/how-to-raise-and-consume-events)
