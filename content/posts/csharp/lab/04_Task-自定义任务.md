---
title: Task-自定义任务
subtitle:
date: 2024-11-26T12:34:08+08:00
slug:
draft: true
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

> [!Note] 前言
> 本篇会研究如何在c#里实现自定义的Task, 一般来说c#默认的Task基本能实现绝大部分的异步需求, 为什么需要自定义`Task`呢？
> - 性能优化: 如果你的任务是高频率的小任务，使用 `ValueTask` 或自定义的轻量级任务类型可以减少内存分配和 GC 压力。
> - 灵活性：你可以根据需要设计任务的行为，比如自定义错误处理、超时机制、取消支持等。
> - 并发控制：有时你需要控制任务的执行方式，比如限制最大并发数，或者为任务设置特定的优先级，这时自定义类型可能会更适合。
> - 组合和调度：在复杂的异步操作中，可能需要将多个任务组合在一起执行并管理任务的依赖关系。通过自定义任务类型，你可以更好地控制任务的执行顺序和逻辑。

## 1. Task Type[^1]
[^1]: [c# Task Type Doc](https://github.com/dotnet/roslyn/blob/main/docs/features/task-types.md) [extending the async methods in c#](https://devblogs.microsoft.com/premier-developer/extending-the-async-methods-in-c/)

任务类型的组成:
1. 一个`class`或`Struct`并带有`System.Runtime.CompilerServices.AsyncMethodBuilderAttribute`属性。   
```c#
[AsyncMethodBuilder(typeof(MyTaskMethodBuilder<>))]
class MyTask<T>
{
    // 为了支持`Await`, `task type`必须有一个可访问的`GetAwaiter`方法返回`awaiter type`。
    public Awaiter<T> GetAwaiter();
}
```
2. Awaiter type: 一个可等待的类型必须实现`INotifyCompletion`并满足如下条件
```c#
class Awaiter<T> : INotifyCompletion
{
    // 判断异步操作是否已经完成
    public bool IsCompleted { get; }
    // 获取异步执行结果
    public T GetResult();
    // 注册一个回调函数，当异步操作完成时，调用这个回调。
    public void OnCompleted(Action completion);
}
```
3. Builder Type: 一个与`Task`对应的`class`或`Struct`, builder type最多只能有一个类型参数且不能嵌套在泛型类中。有以下`public`方法, 对于非泛型的builer type, `SetResult`方法没有参数:
```c#
class MyTaskMethodBuilder<T>
{
    public static MyTaskMethodBuilder<T> Create();

    public void Start<TStateMachine>(ref TStateMachine stateMachine)
        where TStateMachine : IAsyncStateMachine;

    public void SetStateMachine(IAsyncStateMachine stateMachine);
    public void SetException(Exception exception);
    public void SetResult(T result);

    public void AwaitOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : INotifyCompletion
        where TStateMachine : IAsyncStateMachine;

    public void AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : ICriticalNotifyCompletion
        where TStateMachine : IAsyncStateMachine;

    public MyTask<T> Task { get; }
}
``` 

## 2. 自定义任务实现

```c#
static async Task Main(string[] args)
{
    var result = await FooAsync();
    Console.WriteLine($"result={result}");
}

public static async MyTask<int> FooAsync()
{
    Console.WriteLine("mytask exec 1");
    await Task.Delay(1000);
    Console.WriteLine("mytask exec 2");
    await Task.Delay(1000);
    Console.WriteLine("mytask exec 3");
    return 999;
}

[AsyncMethodBuilder(typeof(MyTaskMethodBuilder<>))]
public class MyTask<T>
{
    private MyAwaiter<T> _awaiter;
    public T Value;
    public bool IsCompleted { get; private set; }

    public MyAwaiter<T> GetAwaiter()
    {
        Console.WriteLine("MyTask GetAwaiter");
        return _awaiter ??= new MyAwaiter<T>()
        {
            Task = this,
        };
    }

    public void SetResult(T result)
    {
        Console.WriteLine($"MyTask SetResult result={result}");
        Value = result;
        IsCompleted = true;
        _awaiter?.RunCallback();
        _awaiter = null;
    }
}

public class MyAwaiter<T> : INotifyCompletion
{
    public bool IsCompleted => Task.IsCompleted;
    public MyTask<T> Task { get; set; }

    private Action _continuation;

    public T GetResult()
    {
        Console.WriteLine("MyAwaiter GetResult");
        return Task.Value;
    }

    public void OnCompleted(Action completion)
    {
        Console.WriteLine("MyAwaiter OnCompleted");
        _continuation = completion;
    }

    public void RunCallback()
    {
        Console.WriteLine("MyAwaiter RunCallback");
        _continuation?.Invoke();
    }
}

public class MyTaskMethodBuilder<T>
{
    public static MyTaskMethodBuilder<T> Create()
    {
        Console.WriteLine($"MyTaskMethodBuilder Create type={typeof(T).Name}");
        return new MyTaskMethodBuilder<T>()
        {
            Task = new MyTask<T>()
        };
    }

    public void Start<TStateMachine>(ref TStateMachine stateMachine)
        where TStateMachine : IAsyncStateMachine
    {
        Console.WriteLine($"MyTaskMethodBuilder Start");
        stateMachine.MoveNext();
    }

    public void SetStateMachine(IAsyncStateMachine stateMachine)
    {
        Console.WriteLine($"MyTaskMethodBuilder SetStateMachine stateMachine={stateMachine}");
    }

    public void SetException(Exception exception)
    {
        Console.WriteLine($"MyTaskMethodBuilder SetException exception={exception}");
    }

    public void SetResult(T result)
    {
        Console.WriteLine($"MyTaskMethodBuilder SetResult result={result}");
        Task.SetResult(result);
    }

    public void AwaitOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : INotifyCompletion
        where TStateMachine : IAsyncStateMachine
    {
        Console.WriteLine($"MyTaskMethodBuilder AwaitOnCompleted");
        awaiter.OnCompleted(stateMachine.MoveNext);
    }

    public void AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : ICriticalNotifyCompletion
        where TStateMachine : IAsyncStateMachine
    {
        Console.WriteLine($"MyTaskMethodBuilder AwaitUnsafeOnCompleted");
        Console.WriteLine($"{awaiter.GetType().Name}");
        awaiter.OnCompleted(stateMachine.MoveNext);
    }

    public MyTask<T> Task { get; private set; }
}
```

上面的代码展示了一个简单的自定义任务实现, 运行代码可以看到如下打印:     
1. MyTaskMethodBuilder Create type=Int32
2. MyTaskMethodBuilder Start
3. mytask exec 1
4. MyTaskMethodBuilder AwaitUnsafeOnCompleted
5. TaskAwaiter
6. MyTask GetAwaiter
7. MyAwaiter OnCompleted
8. mytask exec 2
9. MyTaskMethodBuilder AwaitUnsafeOnCompleted
10. TaskAwaiter
11. mytask exec 3
12. MyTaskMethodBuilder SetResult result=999
13. MyTask SetResult result=999
14. MyAwaiter RunCallback
15. MyAwaiter GetResult
16. result=999

逐步分析打印:       
1. `Main`方法状态机执行`var result = await FooAsync();` -> 创建一个`MyTaskMethodBuilder`类型。
2. 执行`FooAsync`对应的状态机的`Start`方法。
3. 执行`FooAsync`的`Console.WriteLine("mytask exec 1");`
4. 执行`FooAsync`的第一个`await Task.Delay(1000);`, 判断该任务未完成, 执行`MyTaskMethodBuilder`里的`AwaitUnsafeOnCompleted`方法。
5. `AwaitUnsafeOnCompleted`里打印了该方法传入的`awaiter`类型, 为`TaskAwaiter`, 注意到这是c#默认的Task对应的Awaiter, `await`后面等待的是什么任务类型, 状态机的后续驱动
就由该任务类型来调用执行。
6. 这里在获取自定义的`MyAwaiter`, 因为上一步`FooAsync`状态机执行完成了, 进入了异步的等待状态, 所以这一步其实是回到了`Main`方法对应的状态机, 因为`await`后是我们的自定义任务类型, 所以获取了自定义的`MyAwaiter`。
7. 还是`Main`方法的状态机执行, 判断到`MyAwaiter`的`IsCompleted`没有完成, 因此`Main`方法的状态机调用了`MyAwaiter`的`OnCompleted`方法, 并传入`Main`状态机的`MoveNext`方法作为`FooAsync`方法执行完成的回调。
8. `FooAsync`的第一个`await`执行完成, 由默认的Task执行`FooAsync`的`MoveNext`方法。
9. 同4
10. 同5
11. 同8
12. `FooAsync`执行完成, 对应最后的`return 999;`, 状态机调用`MyTaskMethodBuilder`的`SetResult`方法。