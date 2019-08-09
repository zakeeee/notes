# Redisson分布式锁

## 加锁过程

1. 执行 `tryLock()`。

```java
@Override
public boolean tryLock() {
    return get(tryLockAsync());
}
```

2. 调用 `tryLockAsync()`。

```java
@Override
public RFuture<Boolean> tryLockAsync() {
    return tryLockAsync(Thread.currentThread().getId());
}
```

3. 调用 `tryLockAsync(long threadId)`。

```java
@Override
public RFuture<Boolean> tryLockAsync(long threadId) {
    return tryAcquireOnceAsync(-1, null, threadId);
}
```

4. 调用 `tryAcquireOnceAsync(long leaseTime, TimeUnit unit, final long threadId)`。

```java
private RFuture<Boolean> tryAcquireOnceAsync(long leaseTime, TimeUnit unit, final long threadId) {
    if (leaseTime != -1) {
        return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_NULL_BOOLEAN);
    }

    RFuture<Boolean> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_NULL_BOOLEAN);

    ttlRemainingFuture.addListener(new FutureListener<Boolean>() {
        @Override
        public void operationComplete(Future<Boolean> future) throws Exception {
            if (!future.isSuccess()) {
                return;
            }

            Boolean ttlRemaining = future.getNow();
            // 获取到了锁
            if (ttlRemaining) {
                scheduleExpirationRenewal(threadId);
            }
        }
    });
    return ttlRemainingFuture;
}
```

5. 异步调用 `scheduleExpirationRenewal(final long threadId)`，并且执行成功后又把自己加入到调度中，这样就可以每隔 `internalLockLeaseTime / 3` 执行一次。

```java
private void scheduleExpirationRenewal(final long threadId) {
    if (expirationRenewalMap.containsKey(getEntryName())) {
        return;
    }

    Timeout task = commandExecutor.getConnectionManager().newTimeout(new TimerTask() {
        @Override
        public void run(Timeout timeout) throws Exception {
            // 更新过期时间
            RFuture<Boolean> future = renewExpirationAsync(threadId);

            future.addListener(new FutureListener<Boolean>() {
                @Override
                public void operationComplete(Future<Boolean> future) throws Exception {
                    expirationRenewalMap.remove(getEntryName());
                    if (!future.isSuccess()) {
                        log.error("Can't update lock " + getName() + " expiration", future.cause());
                        return;
                    }

                    if (future.getNow()) {
                        // 重新把自己添加到调度中去，这样才能周期性地执行
                        scheduleExpirationRenewal(threadId);
                    }
                }
            });
        }

    }, internalLockLeaseTime / 3, TimeUnit.MILLISECONDS);

    if (expirationRenewalMap.putIfAbsent(getEntryName(), new ExpirationEntry(threadId, task)) != null) {
        task.cancel();
    }
}
```

6. 调用 `renewExpirationAsync(long threadId)`，执行了一段 lua 脚本，Redis 执行 lua 脚本是原子性的，这段脚本会把过期时间重新设置为 `internalLockLeaseTime`。

```java
protected RFuture<Boolean> renewExpirationAsync(long threadId) {
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                "return 1; " +
            "end; " +
            "return 0;",
        Collections.<Object>singletonList(getName()),
        internalLockLeaseTime, getLockName(threadId));
}
```

[聊聊redisson的分布式锁](https://juejin.im/post/5ba4b2cc5188255c672eacf2)
