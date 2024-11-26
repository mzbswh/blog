---
title: Task-异步与多线程
subtitle:
date: 2024-11-22T12:34:08+08:00
slug:
draft: false
author:
  name: mzbswh
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - c#
  - 异步
  - Async
  - 同步上下文
  - SynchronizationContext
categories:
  - c#
collections:
  - c#研究室
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

```c#
public class AsyncTest
{
    public async Task FooAsync()
    {
        Console.WriteLine("ThreadId=" + Thread.CurrentThread.ManagedThreadId);
        await Task.Delay(1000);
        Console.WriteLine("ThreadId=" + Thread.CurrentThread.ManagedThreadId);
        await Task.Delay(1000);
        Console.WriteLine("ThreadId=" + Thread.CurrentThread.ManagedThreadId);
    }
}
```

> [!QUESTION] 问题
> 运行上面的异步函数, 发现打印的线程id依次为:  
> - ThreadId=1
> - ThreadId=5
> - ThreadId=5
>
> 也就是说当第一个等待完成后, 当前的线程发生了改变, 线程改变背后的原理是什么呢?

## 线程同步上下文

在上一篇[Task基本实现原理中]({{< ref "02_Task-基本实现原理" >}})中, await等待完成后, 会通过`SynchronizationContext`里的`post`方法
来驱动状态机向下执行, 线程的改变实际就与这个`SynchronizationContext`相关。

### 1. 什么是线程同步上下文

`SynchronizationContext`是.NET框架中提供的一个抽象类, 主要用于管理线程间的任务调度。它允许开发者定义任务应该在哪个线程或执行上下文中运行, 从而实现线程切换和上下文捕获。

在多线程编程中，`SynchronizationContext`的作用尤为重要，特别是在需要将任务调度到特定线程（如 UI 线程）或在特定上下文中运行的场景下。

`SynchronizationContext`提供了调度任务的方法，主要有 **Send(同步)** 和 **Post(异步)**。

异步方法中的 `await`会捕获当前的`SynchronizationContext`, 以确保后续代码能够在正确的上下文中继续运行。

### 2. 默认同步上下文

如果没有设置当前线程的同步上下文, 则会使用默认的上下文, 默认上下文的核心实现代码:
```c#
public partial class SynchronizationContext
{
    public SynchronizationContext() { }

    public static SynchronizationContext? Current => Thread.CurrentThread._synchronizationContext;

    public virtual void Send(SendOrPostCallback d, object? state) => d(state);

    public virtual void Post(SendOrPostCallback d, object? state)
        => ThreadPool.QueueUserWorkItem(static s => s.Key(s.Value), new KeyValuePair<SendOrPostCallback, object?>(d, state), preferLocal: false);
}
```
核心的两个方法就是`Send`和`Post`
- `Send`: 在当前线程同步调用回调方法, 会阻塞当前线程直到方法执行完成。
- `Post`: 异步调用, 不会阻塞当前线程, 如上面默认的实现是从线程池中取一个线程执行回调。

### 3. 自定义同步上下文
回到最开始的问题, 我们通过实现自定义的同步上下文来实现`await`结束后回到主线程执行后续的代码逻辑。
```c#
partial class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Main ThreadId=" + Thread.CurrentThread.ManagedThreadId);
        SynchronizationContext.SetSynchronizationContext(new CustomSynchronizationContext());
        AsyncTest t = new AsyncTest();

        _ = t.FooAsync();

        while (true)
        {
            while (CustomSynchronizationContext.Actions.Count > 0)
            {
                var action = CustomSynchronizationContext.Actions[0];
                CustomSynchronizationContext.Actions.RemoveAt(0);
                action();
            }
            Thread.Sleep(100);
        }
    }
}

public class AsyncTest
{
    public async Task FooAsync()
    {
        Console.WriteLine("async1 ThreadId=" + Thread.CurrentThread.ManagedThreadId);
        await Task.Delay(1000);
        Console.WriteLine("async2 ThreadId=" + Thread.CurrentThread.ManagedThreadId);
        await Task.Delay(1000);
        Console.WriteLine("async3 ThreadId=" + Thread.CurrentThread.ManagedThreadId);
    }
}

public class CustomSynchronizationContext : SynchronizationContext
{
    public static List<Action> Actions = new List<Action>();

    public override void Send(SendOrPostCallback d, object state)
    {
        Post(d, state);
    }

    public override void Post(SendOrPostCallback d, object state)
    {
        Console.WriteLine("Post callback");
        lock (Actions)
        {
            Actions.Add(() => d(state));
        }
    }
}
```
上述代码里, 自定义同步上下文继承了默认同步上下文, 并重写了`Send`与`Post`方法。  
当`Send`或`Post`回调时, 把回调存入一个列表里, 并在主线程的While循环里取出回调并执行。

运行上述代码, 可以看到控制台打印依次如下:
1. Main ThreadId=1
2. async1 ThreadId=1
3. Post callback
4. async2 ThreadId=1
5. Post callback
6. async3 ThreadId=1

这样就实现了异步代码`Await`执行完成后回到了主线程里执行后续的代码。