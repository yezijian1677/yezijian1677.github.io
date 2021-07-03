+++
author = "augenye"
title = "Java Concurrent"
date = "2021-07-02"
description = "并发基础知识"
categories = [
    "技术"
]
tags = [
    "juc","并发"
]
+++
### 基础概念

**上下文切换**：任务执行一个时间片后会切换到下一个任务。但是在切换之前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。

