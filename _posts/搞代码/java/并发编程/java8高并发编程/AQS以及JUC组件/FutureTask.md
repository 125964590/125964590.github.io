 # FutureTask
 ![](https://ws1.sinaimg.cn/large/006tKfTcly1g0iwr1vqxbj31410l6421.jpg)

 该方法可以接受另一个线程中的结果，在获取结果的过程中将持持续等待

 ```
 @Log4j2
public class FeatureTask1 {

    private static ExecutorService executorService = Executors.newCachedThreadPool();

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        final int i = 1;
        FutureTask<String> stringFutureTask = new FutureTask<>(() -> {
            log.info("in the thread");
            Thread.sleep(5000);
            return "lol:" + i;
        });
        executorService.execute(stringFutureTask);
        Thread.sleep(1000);
        log.info("sleep 1000 ms");
        String s = stringFutureTask.get();
        log.info("str: {}", s);
        log.info("finish");
        executorService.shutdown();
    }
}
 ```