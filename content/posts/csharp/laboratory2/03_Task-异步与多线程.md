---
title: Task-异步与多线程
subtitle:
date: 2024-11-22T12:34:08+08:00
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
  - c#,异步,Async,Await,同步上下文,SynchronizationContext
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

在[Task基本实现原理中]({{< ref "posts/csharp/laboratory2/02_Task-基本实现原理" >}})