---
title: 深入了解Lambda表达式-避免GC的产生
subtitle:
date: 2024-11-11T21:59:16+08:00
slug: 4e4b9ed
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
> 在c#中使用Lambda表达式是很常见的，但是某些情形下Lambda表达可能导致大量的GC产生, 怎么在c#正确的使用Lambda表达式以避免产生过多的GC呢？


### 案例1
> [!example] 没有捕获任何外部变量

```csharp
public class LambdaTest
{
    public void Foo()
    {
        Func<int, int> func = (x) => x + 1;
    }
}
```

***编译生成的IL等效c#代码***

```csharp
public class LambdaTest
{
    private sealed class AnonymousClass
    {
        public static readonly AnonymousClass Ins;
        public static Func<int, int> Func;

        static AnonymousClass()
        {
            Ins = new AnonymousClass();
        }

        public AnonymousClass() { }

        internal int Add(int x)
        {
            return x + 1;
        }
    }

    public void Foo()
    {
        Func<int, int> func = AnonymousClass.Func;
        if (func == null)
        {
            func = new Func<int, int>(AnonymousClass.Ins.Add);
            AnonymousClass.Func = func;
        }
    }
}
```  
> [!quote] 结论
> 当第一次调用到Lambda表达式时，会生成此Lambda表达式对应匿名类的唯一实例，并生成一个相应委托类型的唯一实例。  
> 即：此Lambda表达式不会因为多次调用创建多个实例。
---
{.awesome-hr}

### 案列2
> [!example] 实例方法捕获了外部方法的局部变量

```csharp
public class LambdaTest
{
    public void Foo()
    {
        int y = 1;
        Func<int, int> func = x => x + y;
    }
}
```

***编译生成的IL等效c#代码***

```csharp
public class LambdaTest
{
    private sealed class AnonymousClass
    {
        public int y;

        public int Add(int x)
        {
            return x + y;
        }
    }

    public void Foo()
    {
        var c = new AnonymousClass();
        c.y = 1;
        Func<int, int> func = new Func<int, int>(c.Add);
    }
}
```  
> [!quote] 结论
> 每次调用到Lambda表达式时，会生成此Lambda表达式对应匿名类的一个新实例，并生成一个相应委托类型的实例。  
> 即：此Lambda表达式多次调用会创建多个实例。
---
{.awesome-hr}

### 案列3
> [!example] 实例方法捕获了实例字段

```csharp
public class LambdaTest
{
    int y = 1;
    public void Foo()
    {
        Func<int, int> func = x => x + y;
    }
}
```

***编译生成的IL等效c#代码***

```csharp
public class LambdaTest
{
    private int y;

    public LambdaTest()
    {
        y = 1;
    }

    public void Foo()
    {
        Func<int, int> func = new Func<int, int>(this.Add);
    }

    private int Add(int x)
    {
        return x + y;
    }
}
```  
> [!quote] 结论
> 不会生成新的匿名类，每次调用到Lambda表达式时，会生成一个相应委托类型的实例。  
> 即：此Lambda表达式多次调用会创建多个委托类型实例。
---
{.awesome-hr}

### 案列4
> [!example] 静态方法中的捕获了当前类型静态字段

```csharp
public class LambdaTest
{
    static int y = 1;
    public static void Foo()
    {
        Func<int, int> func = x => x + y;
    }
}
```

***编译生成的IL等效c#代码***

```csharp
public class LambdaTest
{
    private static int y;

    static LambdaTest()
    {
        y = 1;
    }

    public void Foo()
    {
        Func<int, int> func = AnonymousClass.Func;
        if (func == null)
        {
            func = new Func<int, int>(AnonymousClass.Ins.Add);
        }
    }

    private sealed class AnonymousClass
    {
        public static readonly AnonymousClass Ins = new AnonymousClass();
        public static Func<int, int> Func;

        public AnonymousClass() { }
        static AnonymousClass() { }

        public int Add(int x)
        {
            return x + LambdaTest.y;
        }
    }
}
```  
> [!quote] 结论
> 与案例1类似，会产生一个唯一的匿名类与委托实例。  
> 注：此案例将Foo改为实例方法所产生的结果是一样的。 
---
{.awesome-hr}

### 总结
> [!note] 通过上面案例可以得出结论：
> 1. Lambda表达式引用了任何局部变量，每次调用会产生一个新的匿名类型实例和一个委托实例。
> 2. Lambda表达式引用了类里的非静态成员，每次调用只会产生一个委托实例。
> 3. Lambda表达式没有引用如何其他变量或者只引用了静态变量，只会产生全局唯一的匿名类型实例和委托实例。  
- 对于第一个引用局部变量的情况是最需要避免的地方，这种情况是无法避免产生新实例的。  
- 对于第二种情况，生成IL时并没有优化委托实例的创建，如果需要多次调用的情况下，我们完全可以在类里创建一个委托字段保存这个委托从而避免委托实例的多次创建。  
```csharp
public class LambdaTest
{
    int y = 1;
    Func<int, int> funcIns;

    public LambdaTest()
    {
        funcIns = x => x + y;
        // or 这里使用Lambda和使用一个实例方法是一样的
        funcIns = Add;
    }

    public int Add(int x)
    {
        return x + y;
    }
}
```
如上面代码所示，在创建类实例时保存此委托实例，其他用到的地方使用此实例。