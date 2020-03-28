Java 的线程池

Java 的线程池可以通过 ThreadPoolExecutor 类进行实现。 ThreadPoolExecutor 的参数的最多的构造函数为 ThreadPoolExecutor(int coolPoolSize, int maximumPoolSize, long keeAliveTime, TimeUnit unit, BlockQuque<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler); 其中
>1. coolPoolSize: 表示的是线程池中核心线程的数量
>2. maximunPoolSize: 表示的是线程池中线程的最大数量
>3. keeAliveTime: 线程池中空闲线程的存活最大时间, 0 表示永久存在
>4. unit: 线程池中线程存活最大时间的单位, 比如: 秒, 分钟等
>5. workQueue: 线程池中存放任务的地方
>6. threadFactory: 线程工厂, 可以用来生产线程池中的线程的的配置, 比如线程的名字, 线程的权重等
>8. handler: 拒绝策略, 当我们的线程池中的线程都处于忙碌状态, 缓存队列也满了, 这时还不断有任务进来，那么这些任务如何处理。

线程池的工作原理: 
>1. 初始状态, 用户每提交一个任务, 线程池就会创建一个线程进行处理。 
>2. 随着用户的不断提交任务, 线程会不断增加, 当数量达到了核心线程数, 后续的任务都会被放到任务队列。 等到线程池中有空闲的线程, 就会到任务队列中取任务。
>3. 当线程池达到了核心线程数, 任务队列也达到了上限, 那么线程池会继续创建线程, 直到数量达到了最大线程数量
>4. 当线程池的线程数达到了最大线程数量, 任务队列也满了, 这是的任务的就靠拒绝策略进行处理。 如: 直接抛弃, 将任务队列中最老的任务扔掉, 在用户线程执行任务。
>5. 当线程池的开始空闲时, 线程池中的空闲线程达到了配置的时间, 就会被消除。

Java 中除了可以通过上面的自定义线程池, 还可以 Executors 提供的方法, 自动创建一些特殊的线程池
>1. Executors.newCachedThreadPool() 等于 new ThreadPoolExecutor(0, Integer.MAX_ VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
核心数为 0, 最大的线程数 Integer.MAX_VALUE, 线程存活时间 60s, 队列为 SynchronousQueue(内部没有维护存储任务的空间, 也就是容量为 0)。 因为核心线程数为 0, 那么进来的任务都会先到工作队列里面，但是现在的工作队列容量为0, 也就是任务来了, 就会创建一个线程进行处理(当然是线程池中没有空闲的线程), 空闲线程在 60s 后就自动消除

>2. Executors.newFixedThreadPool(int nThreads) 等于 new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
核心数和最大线程数都为 nThread, 存活时间为 0, 也就是永久存在, 工作队列为LinkedBlockingQueue, 可以看做是一个无限的队列。 任务来了, 就创建一个线程, 当线程达到 nThread, 后续的任务就会放到队列，队列是无限的。 也就是这个线程池的线程的线程一旦达到了核心线程数，就不会减少。

3. Executors.newSingleThreadExecutor()。 等于 Executors.newFixedThreadPool(1)
核心和最大线程数都为 1, 存活时间永久, 队列为无限队列。 也就是整个线程池只有一个线程用来处理任务。 该线程池确保了在任意一个时刻只有一个任务会被执行。

4. Executors.newScheduledThreadPool(int corePoolSize) 等同于 new ThreadPoolExecutor(corePoolSize, Integer.MAX_VALUE, 0L, TimeUnit.MILLISECONDS, new DelayedWorkQueue());
核心数是 corePoolSize, 最大线程数为 Interger.MAX_VALUE, 线程存活时间 0，也就是永久存在， 工作队列 DelayedWorkQueue, 定时执行任务，当任务的执行时间到来时，ScheduedExecutor 就会真正启动一个线程，其余时间 ScheduledExecutor 都是在轮询任务的状态