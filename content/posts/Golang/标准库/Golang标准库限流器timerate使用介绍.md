---
title: "Golang标准库限流器timerate使用介绍"
date: 2021-03-03T16:21:52+08:00
draft: true
toc: false
images:
tags: 
  - Go标准库
---

> 本主题为系列文章，分上下两篇。本文主要介绍 `time/rate` 的具体使用方法，另外一篇文章 [《Golang 限流器 time/rate 实现剖析》](https://www.cyhone.com/articles/analisys-of-golang-rate/) 则着重介绍其内部实现原理。

限流器是后台服务中的非常重要的组件，可以用来限制请求速率，保护服务，以免服务过载。
限流器的实现方法有很多种，例如滑动窗口法、Token Bucket、Leaky Bucket 等。

其实 Golang 标准库中就自带了限流算法的实现，即 `golang.org/x/time/rate`。该限流器是基于 Token Bucket(令牌桶) 实现的。

简单来说，令牌桶就是想象有一个固定大小的桶，系统会以恒定速率向桶中放 Token，桶满则暂时不放。
而用户则从桶中取 Token，如果有剩余 Token 就可以一直取。如果没有剩余 Token，则需要等到系统中被放置了 Token 才行。



本文则主要集中介绍下该组件的具体使用方法：

# 构造一个限流器

我们可以使用以下方法构造一个限流器对象：

```go
limiter := NewLimiter(10, 1);
```

这里有两个参数：

1. 第一个参数是 `r Limit`。代表每秒可以向 Token 桶中产生多少 token。Limit 实际上是 float64 的别名。
2. 第二个参数是 `b int`。b 代表 Token 桶的容量大小。

那么，对于以上例子来说，其构造出的限流器含义为，其令牌桶大小为 1, 以每秒 10 个 Token 的速率向桶中放置 Token。

除了直接指定每秒产生的 Token 个数外，还可以用 Every 方法来指定向 Token 桶中放置 Token 的间隔，例如：

```go
limit := Every(100 * time.Millisecond);
limiter := NewLimiter(limit, 1);
```

以上就表示每 100ms 往桶中放一个 Token。本质上也就是一秒钟产生 10 个。

Limiter 提供了三类方法供用户消费 Token，用户可以每次消费一个 Token，也可以一次性消费多个 Token。
而每种方法代表了当 Token 不足时，各自不同的对应手段。

# Wait/WaitN

```go
func (lim *Limiter) Wait(ctx context.Context) (err error)
func (lim *Limiter) WaitN(ctx context.Context, n int) (err error)
```

Wait 实际上就是 `WaitN(ctx,1)`。

当使用 Wait 方法消费 Token 时，如果此时桶内 Token 数组不足 (小于 N)，那么 Wait 方法将会阻塞一段时间，直至 Token 满足条件。如果充足则直接返回。

这里可以看到，Wait 方法有一个 context 参数。
我们可以设置 context 的 Deadline 或者 Timeout，来决定此次 Wait 的最长时间。

# Allow/AllowN

```go
func (lim *Limiter) Allow() bool
func (lim *Limiter) AllowN(now time.Time, n int) bool
```

Allow 实际上就是 `AllowN(time.Now(),1)`。

AllowN 方法表示，截止到某一时刻，目前桶中数目是否至少为 n 个，满足则返回 true，同时从桶中消费 n 个 token。
反之返回不消费 Token，false。

通常对应这样的线上场景，如果请求速率过快，就直接丢到某些请求。

# Reserve/ReserveN

```go
func (lim *Limiter) Reserve() *Reservation
func (lim *Limiter) ReserveN(now time.Time, n int) *Reservation
```

Reserve 相当于 `ReserveN(time.Now(), 1)`。

ReserveN 的用法就相对来说复杂一些，当调用完成后，无论 Token 是否充足，都会返回一个 Reservation * 对象。

你可以调用该对象的 Delay() 方法，该方法返回了需要等待的时间。如果等待时间为 0，则说明不用等待。
必须等到等待时间之后，才能进行接下来的工作。

或者，如果不想等待，可以调用 Cancel() 方法，该方法会将 Token 归还。

举一个简单的例子，我们可以这么使用 Reserve 方法。

```go
r := lim.Reserve()
f !r.OK() {
    // Not allowed to act! Did you remember to set lim.burst to be > 0 ?
    return
}
time.Sleep(r.Delay())
Act() // 执行相关逻辑
```

# 动态调整速率

Limiter 支持可以调整速率和桶大小：

1. SetLimit(Limit) 改变放入 Token 的速率
2. SetBurst(int) 改变 Token 桶大小

有了这两个方法，可以根据现有环境和条件，根据我们的需求，动态的改变 Token 桶大小和速率。

# 相关文章

* [Golang 限流器 time/rate 实现剖析](https://www.cyhone.com/articles/analisys-of-golang-rate/)
* [uber-go 漏桶限流器使用与原理分析](https://www.cyhone.com/articles/analysis-of-uber-go-ratelimit/)


> 本文作者： cyhone
> 本文链接： https://www.cyhone.com/articles/usage-of-golang-rate/
> 版权声明： 本博客所有文章除特别声明外，均采用 BY-NC-SA 许可协议。转载请注明出处！


