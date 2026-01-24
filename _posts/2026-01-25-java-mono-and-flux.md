---
title: Java 响应式编程核心：Mono 与 Flux 指南
author: Ethan
date: 2026-01-25 02:00:00 +0800
categories: [Java, Tutorial]
tags: [java]
---

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

## FAQ
### 1. 什么是背压？
背压，简单来说，就是 **当下游处理不过来时，向上游发出的反馈信号**。
如果没有背压，上游一直输出，下游无法消费，这个时候就会造成溢出（内存溢出OOM）。
一个比较形象的类比，
想象你在参加一个吃包子比赛：
上游 (Publisher)：厨师，负责不停地蒸包子并把它们放到你的盘子里。

下游 (Subscriber)：你，负责把盘子里的包子吃掉。

如果没有背压： 厨师不管你吃得快慢，每秒钟往你盘子里放 10 个包子。你 5 秒钟才吃得下一个。很快，你的盘子堆不下了，包子掉了一地，甚至把你埋了起来（OOM）
如果有背压： 你和厨师之间有一个约定。你会告诉厨师：“我现在胃口好，先给我发 3 个”，或者“我吃不消了，先停 10 秒别放了”。厨师根据你的反馈来调整生产速度。
### 1.1. 背压的工作原理
背压遵循 响应式流规范 (Reactive Streams Specification)，主要通过 Subscription 对象来实现。

具体步骤：

订阅：订阅者向发布者发起订阅。

建立连接：发布者给订阅者发送一个 Subscription（合约）。

请求数据：订阅者调用 subscription.request(n)。这步很关键，它明确告诉上游：“我只要 n 个数据”。

发送数据：发布者收到请求，发送 不多于 n 个 的数据。

循环：订阅者处理完这 n 个后，根据自己的状态，再次调用 request(m)。

### 1.2. Project Reactor (Mono/Flux) 中的背压策略
如果你没有手动控制 request，Flux 默认会帮你处理好。但当数据真的爆满时，你可以通过操作符定义策略：

onBackpressureBuffer()：把溢出的数据先存在一个队列里（类似于快递中转站）。

onBackpressureDrop()：处理不过来的数据直接扔掉（就像太挤了就不让上车了）。

onBackpressureLatest()：只保留最新的那个数据，旧的全部扔掉（比如股票行情，我只关心最新的价格）。

onBackpressureError()：处理不过来就直接报错。

示例代码
```java
Flux.interval(Duration.ofMillis(1))
    .onBackpressureLatest() // 策略：只留最后一个
    .concatMap(data -> Mono.delay(Duration.ofSeconds(1)).thenReturn(data))
    .subscribe(data -> System.out.println("当前最新数据: " + data));
```

### 2. 什么是subscribe，以及代码真实的执行顺序？
以下是一个非常形象的执行例子
```java
public Mono<String> register(User user) {
    // 1. 保存用户（这个需要 return，因为用户要看到结果）
    Mono<String> result = userRepository.save(user);

    // 2. 异步发邮件（不需要 return 给用户，所以手动订阅）
    emailService.sendWelcomeEmail(user)
        .subscribe(
            success -> System.out.println("邮件发送成功"),
            error -> System.err.println("邮件发送失败: " + error.getMessage())
        );

    return result; 
}
```
从代码路径上看， `Mono<String> result`似乎是早于第二步的邮件发送前执行的，但实际上，执行顺序反而是2 -> 1。

因为webflux编程中，1只代表了一个“设计图”，没有真正执行，反而是第二步，由于手动使用了subscribe，会立刻异步启动（在另外一个线程）。

在return 的时候，设计图被返回给了spring，spring在接收到了设计图之后，才会帮你调用subscribe，此时，保存用户的操作才开始执行。