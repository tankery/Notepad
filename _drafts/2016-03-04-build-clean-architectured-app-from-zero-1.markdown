---
layout: post
title: 从零开始构建架构清晰的应用（上）
description: "如何从一个 Fast & Dirty 的应用开始，一步步改造，经过整理、抽象、应用架构，使其符合 Clean Architecture 的标准。"
headline: "如何使手表App兼容Ticwear和Android Wear，以及AW中国版"
categories: development
tags:
  - Clean Architecture
  - MVP
  - 应用架构
comments: true
mathjax: null
featured: true
published: true
---

这上下两篇文章，将从一个 Fast & Dirty 的应用开始，一步步改造，经过整理、抽象、使用架构，使其最终符合 Uncle Bob 描述的 [Clean Architecture][the-clean-architecture] 的标准。

这个架构，将使得应用的UI、数据、应用和业务逻辑得到合适的抽象，并使应用易于测试和扩展。

上半部分，着重介绍这个抽象过程的本身，描述我们为何，以及如何做这些抽象。而下半部分，将会着重介绍一些工具库，使我们的开发更便捷有效率。

<!-- break -->


[the-clean-architecture]: https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html

