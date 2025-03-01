---
title: Task-基本实现原理
subtitle:
date: 2024-11-17T19:34:08+08:00
slug: 0b5f289
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
  - Task
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

> [!TIP] 前言
> C#中的异步编程是一种处理长时间运行任务的方式，可以避免阻塞主线程，从而提升应用程序的响应性和性能。异步也可以使回调的写法更加简单明了和线性化, 可以避免嵌套多层的"回调地狱"。本文主要介绍异步背后的一些基本实现原理。

## 异步基本写法

```csharp
public class AsyncTest
{
    public async Task FooAsync()
    {
        Console.WriteLine("step1");
        await Task.Delay(1000);
        Console.WriteLine("step2");
        await Task.Delay(1000);
        Console.WriteLine("step3");
    }

    public void Foo()
    {
        _ = FooAsync();
        Console.WriteLine("执行完成");
    }
}
```  

上面的代码展示了一个非常简单的一个异步写法, 先打印`Step1`, 等1秒后打印`Step2`, 再等1秒打印`Step3`。

> [!QUESTION] 在执行Foo时，可以观察到控制台的打印依次为 `Step1`-`执行完成`-`Step2`-`Step3`, 为什么会是这样的执行顺序呢？

## 异步的实现原理

`AsyncTest`编译为IL时对应的等效c#代码：

```csharp
public class AsyncTest
{
    private sealed class FooAsyncStateMachine : IAsyncStateMachine
    {
        public int state;
        public AsyncTaskMethodBuilder taskBuilder;
        public AsyncTest asyncTest;
        private int _a;
        private TaskAwaiter _awaiter;

        public FooAsyncStateMachine()
        {
            state = -1;
            taskBuilder = AsyncTaskMethodBuilder.Create();
        }

        void IAsyncStateMachine.MoveNext()
        {
            int num = state;
            try
            {
                TaskAwaiter awaiter;
                if (num != 0)
                {
                    if (num == 1)
                    {
                        // step6(第三次MoveNext): 第二个等待完成
                        awaiter = _awaiter;
                        _awaiter = default;
                        awaiter.GetResult();
                        // step7: 执行 a+=2; 全部执行完成
                        Console.WriteLine("step3")
                        return;
                    }
                    // step1(第一次MoveNext): Console.WriteLine("step1"); await Task.Delay(1000);
                    Console.WriteLine("step1");
                    awaiter = Task.Delay(1000).GetAwaiter();
                    if (!awaiter.IsCompleted)
                    {
                        // step2: 第一个等待未完成, 状态=0 return
                        num = state = 0;
                        _awaiter = awaiter;
                        var stateMachine = this;
                        taskBuilder.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine);
                        return;
                    }
                }
                else
                {
                    // step3(第二次MoveNext): 第一个等待完成
                    awaiter = _awaiter;
                    _awaiter = default;
                    num = state = -1;
                }
                awaiter.GetResult();
                // step4: 执行 Console.WriteLine("step2"); await Task.Delay(1000);
                Console.WriteLine("step2");
                awaiter = Task.Delay(1000).GetAwaiter();
                if (!awaiter.IsCompleted)   // 会立即判断任务是否完成
                {
                    // step5: 第二个等待未完成, 状态=1 return
                    num = state = 1;
                    _awaiter = awaiter;
                    var stateMachine = this;
                    taskBuilder.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine);
                    return;
                }
            }
            catch (Exception exception)
            {
                state = -2;
                taskBuilder.SetException(exception);
                return;
            }
            state = -2;
            taskBuilder.SetResult();
        }

        [DebuggerHidden]
        void IAsyncStateMachine.SetStateMachine(IAsyncStateMachine stateMachine)
        {
            taskBuilder.SetStateMachine(stateMachine);
        }
    }

    [AsyncStateMachine(typeof(FooAsyncStateMachine))]
    public Task FooAsync()
    {
        var stateMachine = new FooAsyncStateMachine();
        stateMachine.asyncTest = this;
        stateMachine.taskBuilder.Start(ref stateMachine);
        return stateMachine.taskBuilder.Task;
    }

    public AsyncTest()
    {
    }
}
```

**可以看到, c#背后将异步代码编译生成为了一个状态机, 异步的执行实际就是驱动状态机的执行。**

==上述代码的注释即为状态机的运行步骤顺序。==[info]

`Foo`执行实际执行流程:
- 执行`FooAsync`
- 创建`FooAsyncStateMachine`状态机器类
- `AsyncTaskMethodBuilder`启动状态机
- 状态机执行到第一个`await Task.Delay(1000)`时返回
- `FooAsync`方法执行完成并返回了一个`Task`
- `Foo`方法就继续往下执行`Console.WriteLine("执行完成");`

> [!QUESTION] 怎么让`Foo`全部执行完后才执行`Console.WriteLine("执行完成");`呢？

将`Foo`代码改为如下形式:

```csharp
public async void Foo()
{
    await FooAsync();
    Console.WriteLine("执行完成");
}
```

上述代码将`Foo`改为了异步形式, 并对`FooAsync`添加对等待。同`FooAsync`, 此时`Foo`方法也被编译为了一个状态机, 只有当`FooAsync`全部执行完后, 即`FooAsync`状态机执行到`SetResult()`时, `FooAsync`被标记为执行完成, 并驱动`Foo`状态机继续往下执行。

## 状态机的驱动[^dotnetRuntime]

[^dotnetRuntime]: [dotnet runtime source code](https://github.com/dotnet/runtime/tree/main/src/libraries)

#### 1. 启动状态机

参考上述状态机代码, 可以发现状态机的启动是通过`AsyncTaskMethodBuilder`调用`Start()`方法。`AsyncTaskMethodBuilder`部分内部实现:
```csharp
public struct AsyncTaskMethodBuilder
{
    public void Start<TStateMachine>(ref TStateMachine stateMachine) where TStateMachine : IAsyncStateMachine =>
        AsyncMethodBuilderCore.Start(ref stateMachine);

    public void AwaitOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : INotifyCompletion
        where TStateMachine : IAsyncStateMachine =>
        AsyncTaskMethodBuilder<VoidTaskResult>.AwaitOnCompleted(ref awaiter, ref stateMachine, ref m_task);

    public void AwaitUnsafeOnCompleted<TAwaiter, TStateMachine>(
        ref TAwaiter awaiter, ref TStateMachine stateMachine)
        where TAwaiter : ICriticalNotifyCompletion
        where TStateMachine : IAsyncStateMachine =>
        AsyncTaskMethodBuilder<VoidTaskResult>.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine, ref m_task);
}
```
可以看到`Start()`方法实际调用了`AsyncMethodBuilderCore`里的`Start()`, `AsyncMethodBuilderCore`是一个静态类, 以下是部分内部实现:
```c#
internal static class AsyncMethodBuilderCore // debugger depends on this exact name
{
    public static void Start<TStateMachine>(ref TStateMachine stateMachine) where TStateMachine : IAsyncStateMachine
    {
        if (stateMachine == null) // TStateMachines are generally non-nullable value types, so this check will be elided
        {
            ThrowHelper.ThrowArgumentNullException(ExceptionArgument.stateMachine);
        }

        Thread currentThread = Thread.CurrentThread;

        // Store current ExecutionContext and SynchronizationContext as "previousXxx".
        // This allows us to restore them and undo any Context changes made in stateMachine.MoveNext
        // so that they won't "leak" out of the first await.
        ExecutionContext? previousExecutionCtx = currentThread._executionContext;
        SynchronizationContext? previousSyncCtx = currentThread._synchronizationContext;

        try
        {
            stateMachine.MoveNext();
        }
        finally
        {
            // The common case is that these have not changed, so avoid the cost of a write barrier if not needed.
            if (previousSyncCtx != currentThread._synchronizationContext)
            {
                // Restore changed SynchronizationContext back to previous
                currentThread._synchronizationContext = previousSyncCtx;
            }

            ExecutionContext? currentExecutionCtx = currentThread._executionContext;
            if (previousExecutionCtx != currentExecutionCtx)
            {
                ExecutionContext.RestoreChangedContextToThread(currentThread, previousExecutionCtx, currentExecutionCtx);
            }
        }
    }
}
```
这里会保存当前线程的**执行上下文**与**同步上下文**, 将状态机移动到下一步后, 还原`stateMachine.MoveNext()`对上下文做的任何改变。  
有关上下文的介绍会在之后的文章讲解。

#### 2. 等待异步任务完成

状态机启动后, 遇到await的任务时, 如果判断任务未完成, 会调用`taskBuilder.AwaitUnsafeOnCompleted`或`taskBuilder.AwaitOnCompleted`方法。   

这两个方法主要区别在与`AwaitUnsafeOnCompleted`不会捕获`ExecutionContext`, 这会减少一些不必要的开销, 大多数的异步场景不依赖`ExecutionContext`。  

```c#
async Task Example()
{
    await Task.Delay(1000).ConfigureAwait(true); // 捕获上下文
}
```
代码`Task.Delay(1000)`的`AwaitUnsafeOnCompleted`调用链:
 - AsyncTaskMethodBuilder<T>.AwaitUnsafeOnCompleted(ref awaiter, ref stateMachine, ref m_task)
 - TaskAwaiter.UnsafeOnCompletedInternal(ta.m_task, box, continueOnCapturedContext: true);
 - Task.UnsafeSetContinuationForAwait(stateMachineBox, continueOnCapturedContext);
 - 创建`TaskContinuation tc = new SynchronizationContextAwaitTaskContinuation(syncCtx, stateMachineBox.MoveNextAction, flowExecutionContext: false);`

#### 3. 完成后调用

在上面的最后一个调用中, 构造了一个`TaskContinuation`类, 并将`stateMachineBox.MoveNextAction`传类进去。  
当`Task.Delay`任务完成时, 调用链如下:
 - Task.TrySetResult()
 - Task.FinishContinuations()
 - Task.RunContinuations();
 - TaskContinuation.Run()

最后一步就是运行上一步等待时创建的`TaskContinuation`, 部分实现如下:
```c#
/// <summary>Inlines or schedules the continuation.</summary>
/// <param name="task">The antecedent task, which is ignored.</param>
/// <param name="canInlineContinuationTask">true if inlining is permitted; otherwise, false.</param>
internal sealed override void Run(Task task, bool canInlineContinuationTask)
{
    // If we're allowed to inline, run the action on this thread.
    if (canInlineContinuationTask &&
        m_syncContext == SynchronizationContext.Current)
    {
        RunCallback(GetInvokeActionCallback(), m_action, ref Task.t_currentTask);
    }
    // Otherwise, Post the action back to the SynchronizationContext.
    else
    {
        TplEventSource log = TplEventSource.Log;
        if (log.IsEnabled())
        {
            m_continuationId = Task.NewId();
            log.AwaitTaskContinuationScheduled((task.ExecutingTaskScheduler ?? TaskScheduler.Default).Id, task.Id, m_continuationId);
        }
        RunCallback(GetPostActionCallback(), this, ref Task.t_currentTask);
    }
    // Any exceptions will be handled by RunCallback.
}
```
这里最终就是调用`SynchronizationContext`的`Post()`方法并将状态机的`MoveNext`方法传入。

同步上下文主要就是`Send`和`Post`这两个方法, 一个同步调用一个异步调用, 这里不做详细介绍, 在后续的文章讲解。

状态机的整个驱动流程大概就是这些步骤, 具体代码可以直接参考[官方源码](https://github.com/dotnet/runtime/tree/main/src/libraries)[^dotnetRuntime]
