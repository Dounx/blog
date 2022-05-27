---
layout: post
title: Redis 分布式锁
categories: redis
---

之所以会需要使用 Redis 锁，是因为需要让用户的某一种操作同一段时间只发生一次，或者串行化这类操作。

假设用户会进行 A 操作，A 和 A 就不能同时进行。

那么利用 Redis 锁就可以实现这种效果（锁住 A 对应的 Key），而数据库就不行。

其实在串行化的时候，使用队列可能会比用 Redis 分布锁会更好。但是用在防抖的时候，用 Redis 分布式锁是挺不错的。

串行化和防抖的区别主要是在获取 Redis 锁的时候，前者需要等待，后者则不需要直接返回 false 即可。

利用 Redis 的 setnx 实现一个分布式锁其实很简单，在一般项目中足够用了（当然官方推荐使用 RedLock）。

Redis 分布式锁的两个关键步骤是获得锁和释放锁。

获得锁需要以下步骤：

1. setnx key uuid
2. expire key timeout

释放锁需要以下步骤：

1. 检查 key 的 uuid 是否为当前 uuid
2. 若是，则 del key

上述获得锁和释放锁的操作都需要保证原子性，所以使用 Lua 脚本编写并执行。

然后这个 timeout 的时间应设置为锁内代码可能的最大运行时间。

当然也可以在锁内代码执行的时候，起一个守护进程，如果没有运行完则 extend 锁，但是这种方式实现起来会很复杂。

根据业务需求，获取锁的时候可以选择是否等待。

简单实现：https://github.com/Dounx/distributed_mutex

## 参考资料

1. https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/
