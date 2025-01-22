---
title: 如何写出好的代码？
subtitle:
date: 2025-01-22T20:26:16+08:00
slug: f56894b
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
  - code science
categories:
  - misc
collections:
  - code science
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
### 1. 逻辑设计
代码是否设计良好？这种设计是否适合当前系统？是否具备良好的可扩展性与可维护性？是否考虑了全局设计和兼容现有业务细节? 是否考虑边界条件和并发控制? 是否分层清晰、模块化合理、高内聚?
### 2. 功能完整
代码实现的行为与作者的期望是否相符？代码实现的交互界面是否对用户友好？代码实现是否满足功能需求? 实现上有没有需求的理解偏差?
### 3. 复杂性
是否有重复可简化的复杂逻辑，代码复杂度是否过高? 如果将来有其他开发者使用这段代码，他能很快理解吗？是否逻辑清晰、易理解，避免使用奇淫巧技，避免过度拆分?  
KISS 原则（Keep It Simple, Stupid）: 尽可能让代码保持简单，不要让设计和实现变得不必要的复杂。  
DRY 原则（Don't Repeat Yourself）: 避免代码重复，提高代码的复用性。
### 4. 测试
这段代码是否有正确的、设计良好的自动化测试？测试用例的代码行覆盖率和分支覆盖率?
### 5. 编码规范
在为变量、类名、方法等命名时，开发者使用的名称是否清晰易懂？命名、注释、领域术语、架构分层、日志打印、代码样式等是否符合规范? 参数是否全校验？变量null判断？分支处理逻辑？容错？资源是否全释放？
### 6. 注释
所有的注释是否都一目了然？
### 7. 文档
开发者是否同时更新了相关文档？
### 8. 安全隐患与性能隐患
是否存在数据安全隐患？是否存在损害性能的隐患？