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
3. Builder Type: 一个与`Task`对应的`class`或`Struct`