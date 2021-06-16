### Redis实现分布式锁

#### 什么是分布式锁

分布式系统中，多个进程

#### 分布式锁的实现

分布式锁实现的关键是在分布式应用服务器外，搭建一个存储服务器，存储锁信息，可以使用Redis。

实现分布式锁有几个关键点：

1. 互斥性：在任意时刻，只能有一个客户端能持有锁。
2. 不会发生死锁：即使一个客户端在持有锁的期间崩溃而没有释放锁，也能保证后续其他客户端能加锁。
3. 具有容错性：只要大部分Redis节点正常运行，客户端就可以加锁和解锁。
4. 解铃还须系铃人：加锁和解锁必须是同一个客户端，客户端不能把其他客户端加的锁给解了。

#### 加锁

```java
jedis.set(lockKey, requestId, "NX", "EX", expireTIme);
// lockKey 锁的名字
// requestId 请求ID，标识当前线程，只有当前线程才能解锁
// "NX" 加上这个参数相当于setnx命令
// "EX" 这个参数表示设置expireTime秒后过期
// expireTime 过期时间
```

错误示例1

```java
Long result = jedis.setnx(lockKey, requestId);
if (result == 1) {
    // 若在这里程序突然崩溃，则无法设置过期时间，将发生死锁
    jedis.expire(lockKey, expireTime);
}
```

错误示例2

```java
public static boolean wrongGetLock2(Jedis jedis, String lockKey, int expireTime) {
 
    long expires = System.currentTimeMillis() + expireTime;
    String expiresStr = String.valueOf(expires);
 
    // 如果当前锁不存在，返回加锁成功
    if (jedis.setnx(lockKey, expiresStr) == 1) {
        return true;
    }
 
    // 如果锁存在，获取锁的过期时间
    String currentValueStr = jedis.get(lockKey);
    if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {
        // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间
        String oldValueStr = jedis.getSet(lockKey, expiresStr);
        if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
            // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才有权利加锁
            return true;
        }
    }
        
    // 其他情况，一律返回加锁失败
    return false;
}
```

这段代码存在的问题：

1. 由于是客户端自己生成过期时间，所以需要强制要求分布式下每个客户端的时间必须同步
2. 当锁过期的时候，如果多个客户端同时执行`jedis.getSet()`方法，那么虽然最终只有一个客户端可以加锁，但是这个客户端的锁的过期时间可能被其他客户端覆盖
3. 锁不具备拥有者标识，即任何客户端都可以解锁

#### 解锁

```java
public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
 
    String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
    Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
 
    if (Long.valueOf(1L).equals(result)) {
        return true;
    }
    return false;
 
}
```

这段代码首先获取锁对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）。使用Lua脚本保证了上述操作的原子性。

错误示例1

```java
public static void wrongReleaseLock1(Jedis jedis, String lockKey) {
    jedis.del(lockKey);
}
```

这种方式没有判断锁的拥有者，导致任何客户端都能解锁，即使这把锁不是它的。

错误示例2

```java
public static void wrongReleaseLock2(Jedis jedis, String lockKey, String requestId) {
        
    // 判断加锁与解锁是不是同一个客户端
    if (requestId.equals(jedis.get(lockKey))) {
        // 若在此时，这把锁突然不是这个客户端的，则会误解锁
        jedis.del(lockKey);
    }
}
```

这种方法看起来跟正确示例一样，但是如果在调用del命令的时候，锁突然过期了，同时另一个客户端尝试获取锁成功，客户端执行del命令就会把其他客户端的锁解除。

也可使用Redission实现：

```java
// 最常见的使用方法
RLock lock = redisson.getLock("lockKey");
lock.lock();
// 执行业务
lock.unlock();
 
//另外Redisson还通过加锁的方法提供了leaseTime的参数来指定加锁的时间。超过这个时间后锁便自动解开了。
 
// 加锁以后10秒钟自动解锁
// 无需调用unlock方法手动解锁
lock.lock(10, TimeUnit.SECONDS);
 
// 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
   try {
     ...
   } finally {
       lock.unlock();
   }
}
```

#### Redis分布式锁的缺点

在哨兵模式下，客户端1对某个master节点写入了锁信息，此时会异步复制给slave节点。但是这个过程中，一旦master节点宕机，主备切换，salve节点变成了master节点。于是客户端2尝试加锁的时候，在新的master节点上能加锁成功。



#### 基于Redis分布式锁实现秒杀

秒杀系统，是典型的短时大量突发访问类问题。对这类问题，有三种优化思路：

1. 写入内存而不是硬盘
2. 异步处理而不是同步处理
3. 分布式处理

Redis能满足上述三点，因此，用Redis能轻松实现秒杀系统。

实现思路：

先将商品总数存入Redis，服务器通过请求Redis获得下单资格，通过Lua脚本实现，由于Redis是单线程模型， Lua可以保证多个命令的原子性。

Redis接收到下单请求时先判断商品剩余总数是否大于0，如果大于0则商品库存减一，返回1 ，否则返回0 。

可以使用`script load`命令将Lua脚本提前缓存在Redis，然后调用`evalsha`，比直接调用`eval`节省宽带。

下单成功后订单信息进入MQ，订单处理模块异步从MQ中获取订单信息处理。

Redis的QPS达到10w+，若还不能满足需求，可使用Redis集群，下单请求按照某种算法发送到不同的Redis服务。