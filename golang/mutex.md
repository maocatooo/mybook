---
title: Golang mutex
date: 2023-01-03
tags: [Golang]
---

参考 ： 

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives

https://www.bilibili.com/video/BV1hv411x7we


#### mutex

sync.Mutex 由两个字段 state 和 sema 组成。其中 state 表示当前互斥锁的状态, 而 sema 是用于控制锁状态的信号量

```go

type Mutex struct {
    state int32
    sema  uint32
}
```
![mutex](./images/mutex.png)

state 最低三位表示mutexLocked, mutexWoken,  mutexStarving

 - mutexLocked 表示互斥锁的锁定状态；
 - mutexWoken — 表示从正常模式被从唤醒；
 - mutexStarving — 当前的互斥锁进入饥饿状态；

其余位置表示当前互斥锁上等待的 Goroutine 个数


#### 正常模式
一个尝试加锁的Goroutine会先自旋几次, 尝试通过原子操作获取锁, 若获取不到锁, 就通过信号量排队等待(先进先出), 
当锁释放是, 第一个排队等待的Goroutine会被唤醒, 与处于自旋状态,且没有在排队的Goroutine争取锁, 被唤醒的Goroutine没有获取锁会返回队列排在第一位,当一个Goroutine处于等待时长超过1ms,当前状态切换为饥饿状态

#### 饥饿模式
mutex的所有权直接从执行Unlock的Goroutine交给等待队列头部的Goroutine, 后来者不会自旋且不会尝试获得锁, 直接在队列排队等待获取锁

切换正常模式：
  - 当等待者获取锁之后, 且等待时长小于1ms
  - 当前等待者是队列种最后一个获取锁的Goroutine

##### 为什么需要自旋

避免频繁的挂起, 唤醒Goroutine带来的开销

##### 为什么等待需要和自旋争取/为什么自旋的Goroutine争取锁更有优势
处于自旋的Goroutine正在CPU上运行, 可以避免不必要的上下文切换, 处于自旋的Goroutine可能有很多个, 队列种被唤醒的Goroutine只有一个, 所以被唤醒的Goroutine大概率拿不到锁


#### 触发自旋的条件

1. 锁已经被占用，且锁不处于饥饿状态

2. 累计的自旋次数，小于最大自旋次数（active_spin=4）

3. cpu 核数大于1

4. 有空闲的P

5. 当前 goroutine 锁挂载的P下，本地运行G队列为空


sync.Mutex 有两种模式

- 正常模式和饥饿模式。

我们需要在这里先了解正常模式和饥饿模式都是什么以及它们有什么样的关系

在正常模式下, 锁的等待者会按照先进先出的顺序获取锁。但是刚被唤起的 Goroutine 与新创建的 Goroutine 竞争时, 大概率会获取不到锁, 为了减少这种情况的出现, 一旦 Goroutine 超过 1ms 没有获取到锁, 它就会将当前互斥锁切换饥饿模式, 防止部分 Goroutine 被『饿死』

在饥饿模式中, 互斥锁会直接交给等待队列最前面的 Goroutine。新的 Goroutine 在该状态下不能获取锁、也不会进入自旋状态, 它们只会在队列的末尾等待。如果一个 Goroutine 获得了互斥锁并且它在队列的末尾或者它等待的时间少于 1ms, 那么当前的互斥锁就会切换回正常模式

