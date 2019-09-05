# AQS

![](https://ws2.sinaimg.cn/large/006tNc79ly1fzm5tzq08gj31f80u0nju.jpg)

![](https://ws3.sinaimg.cn/large/006tNc79ly1fzm5wessyfj31r20u0wwp.jpg)

### CountDownLatch
![](https://ws3.sinaimg.cn/large/006tNc79ly1fzm81o09w5j316u0u0an5.jpg)
给定一个计数器

```
    private static int threadCount = 200;

    private static CountDownLatch countDownLatch = new CountDownLatch(threadCount);
```

- countDownLatch.countDown(): 每执行一次减一
- countDownLatch.await(): 等待内部计数减到零
- countDownLatch(10,TimeUtil.MILLISECONDS): 设置每个方法的超时时间,如果超时则直接结束


### Semaphore
![](https://ws2.sinaimg.cn/large/006tNc79ly1fzmazn4dm2j31990u01kx.jpg)

```
 private static Semaphore semaphore = new Semaphore(3);
```

设置可以操作的线程数,可以进行并发访问控制
 - semaphore.release(): 释放一个许可
 -  semaphore.acquire(): 获取一个许可
 - semaphore.tryAcquire(): 尝试获取一个许可
 - semaphore.tryAcquire(int permits, long timeout, TimeUnit unit):尝试获取多个许可,并且设置超时时间

### CyclicBarrier

![](https://ws1.sinaimg.cn/large/006tNc79ly1fzmcl3e6l5j31640u0qm0.jpg)

```
 private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
```

-  await():等待指定线程数,如果达到则立刻执行
- await(long timeout, TimeUnit unit):如果等待超时则会抛出异常