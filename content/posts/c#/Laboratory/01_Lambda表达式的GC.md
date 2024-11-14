---
title: Lambda表达式的GC
subtitle:
date: 2024-11-11T21:59:16+08:00
slug: 4e4b9ed
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

> [!question] 问题背景
> 在c#中使用Lambda表达式是很常见的，但是某些情形下Lambda表达式可能导致大量的GC产生, 如何在c#正确的使用Lambda表达式以避免产生过多的GC呢？


## 案例1
> [!example] 没有捕获任何外部变量的Lambda 表达式

```csharp
public class LambdaTest
{
    public void Foo()
    {
        Func<int, int> func = (x) => x + 1;
    }
}
```

***编译后的等效c#代码***

```
public class Test
{
    // 匿名内部类
    private class AnonymousNestedClass
    {
        // 缓存匿名类单例
        public static readonly AnonymousNestedClass _anonymousInstance;

        // 缓存委托实例
        public static Func<int, int> _func;

        static AnonymousNestedClass()
        {
            _anonymousInstance = new AnonymousNestedClass();
        }

        internal int AnonymousMethod(int x)
        {
            return x + 1;
        }
    }

    public void Foo()
    {
        // 这里是编译器的一个优化，委托实例是单例
        if (AnonymousNestedClass._func == null)
        {
            AnonymousNestedClass._func = 
                new Func<int, int>(AnonymousNestedClass._anonymousInstance.AnonymousMethod);
        }

        Func<int, int> func = AnonymousNestedClass._func;
    }
}
```


CASE 1 没有捕获任何外部变量的Lambda 表达式
public class Test
{
    public void Foo()
    {
        Func<int, int> func = x => x + 1;
    }
}
CASE 2 捕获了外部方法局部变量的Lambda 表达式
public class Test
{
    public void Foo()
    {
        int y = 1;
        Func<int, int> func = x => x + y;
    }
}
CASE 3 实例方法中捕获了实例字段的Lambda 表达式
public class Test
{
    private int _y = 1;
    public void Foo()
    {
        Func<int, int> func = x => x + _y;
    }
}
CASE 4 静态方法中的捕获了当前类型静态字段的Lambda 表达式
public class Test
{
    private static int _y = 1;
    public static void Bar()
    {
        Func<int, int> func = x => x + _y;
    }
}