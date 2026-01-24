---
title: Java 响应式编程核心：Mono 与 Flux 指南
author: Ethan
date: 2026-01-25 02:00:00 +0800
categories: [Java, Tutorial]
tags: [java]
---
# Java 响应式编程核心：Mono 与 Flux 指南

## 1. 核心概念：什么是响应式流？

在响应式编程中，数据不再是被“拿”过来的，而是像水一样流过来的。

* **Flux (多元素流)**：代表 0 到 N 个元素的异步序列。你可以把它看作一条**传送带**，上面会陆续送来零件。
* **Mono (单元素流)**：代表 0 到 1 个元素的异步序列。你可以把它看作一个**快递盒**，打开后要么有一个零件，要么是空的。

### 核心定律：不订阅，无事发生

**Nothing happens until you subscribe.**
你写的 `map`, `filter`, `flatMap` 只是在“铺设水管”，如果你不打开水龙头（订阅），一滴水（数据）都不会流过。

---

## 2. 为什么 `return` 就能执行？

在 Spring WebFlux（如 Controller 层）中，你通常只写 `return` 而不写 `.subscribe()`，但代码依然运行了。这是因为：

* **框架代劳**：Spring WebFlux 充当了“最终订阅者”。
* **执行链路**：
1. 你交出一张“水管设计图”（`return Mono/Flux`）。
2. Spring 拿到后，在底层自动调用了 `.subscribe()`。
3. 数据开始流动，最终通过 HTTP 响应发给用户。



> **注意：** 永远不要在 Controller 内部“断开”这个链条。如果你没有 `return` 某个流，那么那个流里的逻辑（如数据库保存）永远不会被触发。

---

## 3. Subscribe 的不同用法

如果你在非 Web 环境（如测试或后台异步任务）中，需要手动触发流。

### A. 简单触发

只启动流，不关心结果。

```java
source.subscribe();

```

### B. 处理数据 (onNext)

拿到数据后进行处理。

```java
source.subscribe(data -> {
    System.out.println("收到数据: " + data);
});

```

### C. 全功能订阅 (处理数据 + 错误 + 完成)

最完整的监听方式。

```java
source.subscribe(
    data -> System.out.println("数据: " + data),     // 正常处理
    err -> System.err.println("出错: " + err),      // 异常处理 (类似 catch)
    () -> System.out.println("处理完毕！")           // 完成通知 (类似 finally)
);

```

---

## 4. 关键操作符：Map vs FlatMap

这是新手最容易混淆的地方：

* **Map (一对一变换)**：
* **动作**：把苹果切成片。
* **场景**：把 `User` 对象转换成 `UserResponse` 对象。
* **代码**：`.map(user -> user.getName())`


* **FlatMap (一对多/异步变换)**：
* **动作**：把订单 ID 拿去查数据库，返回一个新的订单详情流。
* **场景**：当你需要调用另一个返回 `Mono` 或 `Flux` 的异步方法时。
* **代码**：`.flatMap(id -> repository.findById(id))`



---

## 5. 与 Go 异步 (go func) 的区别

如果你熟悉 Go 语言，可以对比理解：

| 特性 | Go `go func(){}` | Java `Mono/Flux` |
| --- | --- | --- |
| **风格** | **命令式**：像写同步代码一样写异步。 | **声明式**：像搭积木一样拼装逻辑流。 |
| **状态** | 有独立的协程栈，变量随调随用。 | 无状态回调链，必须在流中传递变量。 |
| **背压** | 需通过 Channel 手动控制。 | **原生支持**：下游可以告诉上游慢点发。 |
| **报错** | `if err != nil` 直接处理。 | 必须用 `onErrorResume` 等操作符处理。 |

---

## 6. 避坑指南 (Checklist)

1. **别忘了 Return**：方法里定义的 Mono/Flux 一定要 return 或者是作为返回流的一部分。
2. **避免多次订阅**：一个流通常只应该被订阅一次。
3. **不要在流里用阻塞代码**：千万不要在 `map` 或 `flatMap` 里用 `Thread.sleep()` 或同步的数据库驱动，这会瞬间卡死整个系统的事件环。

---

## 7. 常用名词中英文对照 (Terminology)

* **异步 (Asynchronous)**: 不原地等待结果。
* **非阻塞 (Non-blocking)**: 线程不挂起。
* **操作符 (Operator)**: 处理流的方法（map, filter等）。
* **背压 (Backpressure)**: 流量控制机制。
* **副作用 (Side Effect)**: 不影响数据本身的额外动作（如 `doOnNext`）。

