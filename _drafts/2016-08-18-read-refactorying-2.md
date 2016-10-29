---
layout: post
title: 阅读《重构》（二）- 准备重构
description: "总结归纳《重构：改善既有代码的设计》这本书的重点内容，第二部分，重构的准备工作"
categories:
  - development
  - reading
tags:
  - 重构
  - 设计模式
  - 改善既有代码的设计
  - 代码坏味道
comments: true
mathjax: null
featured: true
published: true
---

这一系列文章将会总结归纳《重构：改善既有代码的设计》这本书的重点内容。

本篇文章，对应书的三、四章。首先通过列举各类代码的“坏味道”来告诉我们什么时候应该开始重构，然后通过测试用例，为重构做准备。

<!-- more -->

## 代码的坏味道

> 如果尿布臭了，就换掉它。

作者以这样的一个 Beck 奶奶的“人生哲理”开头，告诉我们，重构的时机就是我们发现代码“有坏味道”时。

但这个坏味道，并不是一个精确的时机，而是需要一定的经验和直觉。当然，通过某些迹象，我们就能开始思考需不需要重构了。

1. **Duplicated Code**
2. **Long Method**
3. **Large Class**
4. **Long Parameter List**
5. **Divergent Change**
6. **Shotgun Surgery**
7. **Feature Envy**
8. **Data Clumps**
9. **Primitive Obsession**
10. **Switch Statements**
11. **Parallel Inheritance Hierarchies**
12. **Lazy Class**
13. **Speculative Generality**
14. **Temporary Field**
15. **Message Chains**
16. **Middle Man**
17. **Inappropriate Intimacy**
18. **Alternative Classes with Different Interfaces**
19. **Incomplete Library Class**
20. **Data Class**
21. **Refused Bequest**
22. **Comments**



## 构筑测试体系


