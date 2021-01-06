### Executor-Future

  #### [UML](#uml)

#### [核心参数](#param)

  #### [RejectedExecutionHandler](#executionHandler)

  #### [ThreadFactory](#threadFactory)

  #### [Hook](#hook)

  #### [ThreadPool](#threadPool)

  #### [状态机](#state)

  #### [核心代码](#code)

#### [流程图](#stream)

* <a id=uml>UML</a>

  ![image-20210104141625894](https://gitee.com/zzz_123456/picgo/raw/main/image/image-20210104141625894.png)



* <a id=param>核心参数</a>

  ```java
  // 高3位保存线程池状态 低29位保存线程数量
  private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
  
  private final BlockingQueue<Runnable> workQueue;
  
  private final ReentrantLock mainLock = new ReentrantLock();
   
  private final HashSet<Worker> workers = new HashSet<Worker>();
      
  private final Condition termination = mainLock.newCondition();
  
  private int largestPoolSize;
      
  private long completedTaskCount;
    
  private volatile ThreadFactory threadFactory;
  
  private volatile RejectedExecutionHandler handler;
  //allowCoreThreadTimeOut为true或者workerCount超过corePoolSize时 等待从task队列中获取task的超时时间
  private volatile long keepAliveTime;
  // 如果为false(默认值)，核心线程即使在空闲时也保持活动。如果为true，核心线程使用keepAliveTime超时等待工作
  private volatile boolean allowCoreThreadTimeOut;
    
  private volatile int corePoolSize;
  
  private volatile int maximumPoolSize;
  
  private static final RejectedExecutionHandler defaultHandler =
          new AbortPolicy();
  
  private static final RuntimePermission shutdownPerm =
        new RuntimePermission("modifyThread");
  private final AccessControlContext acc;
  ```




* <a id="threadFactory">ThreadFactory</a>
  
  * `DefaultThreadFactory`
  
    ​	所有线程具有相同的group,priority
  
  * `PrivilegedThreadFactory`

* <a id="executionHandler">RejectedExecutionHandler</a>

  ```java
  public interface RejectedExecutionHandler {
  
      void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
  }
  ```

  默认四种实现:

  * CallerRunsPolicy

    <font color="grey">如果线程池未关闭，由提交任务的线程来执行</font>

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                if (!e.isShutdown()) {
                    r.run();
                }
            }
    ```

    

  * AbortPolicy

    <font color="grey">默认拒绝策略，直接抛出异常`RejectedExecutionHandler`</font>

    ```java
     public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                throw new RejectedExecutionException("Task " + r.toString() +
                                                     " rejected from " +
                                                     e.toString());
            }
    ```

    

  * DiscardPolicy

    <font color="grey">直接抛弃该任务</font>

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            }
    ```

    

  * DiscardOldestPolicy

    <font color="grey">如果线程池未关闭，抛弃最早的任务，提交该任务</font>

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                if (!e.isShutdown()) {
                    e.getQueue().poll();
                    e.execute(r);
                }
            }
    ```

    

* <a id="threadPool">ThreadPool</a>

  1. <font color=grey>jdk自带的几种线程池(Executors),各有弊端，最好使用自定义线程池`new ThreadPoolExecutor()`</font>
  2. <font color=grey>`newFixedThreadPool`、`newSingleThreadExecutor`使用无界的LinkedBlockingQueue造成堆积的请求处理队列造成oom</font>
  3. <font color=grey>`newCachedThreadPool`、`newScheduledThreadPool`线程最大数为Integer.MAX_VALUE造成oom</font>

  默认几种实现:

  * newFixedThreadPool

    ```java
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
            return new ThreadPoolExecutor(nThreads, nThreads,
                                          0L, TimeUnit.MILLISECONDS,
                                          new LinkedBlockingQueue<Runnable>(),
                                          threadFactory);
        }
    ```

    

  * newSingleThreadExecutor

    ```java
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
            return new FinalizableDelegatedExecutorService
                (new ThreadPoolExecutor(1, 1,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>(),
                                        threadFactory));
        }
    ```

    

  * newCachedThreadPool

    ```java
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
            return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                          60L, TimeUnit.SECONDS,
                                          new SynchronousQueue<Runnable>(),
                                          threadFactory);
        }
    ```

    

  * newScheduledThreadPool

    ```java
    public static ScheduledExecutorService newScheduledThreadPool(
                int corePoolSize, ThreadFactory threadFactory) {
            return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
        }
    
     public ScheduledThreadPoolExecutor(int corePoolSize,
                                           ThreadFactory threadFactory) {
            super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
                  new DelayedWorkQueue(), threadFactory);
        }
    ```

    

* <a id="state">状态机</a>

  ![image-20210105103000838](https://gitee.com/zzz_123456/picgo/raw/main/image/image-20210105103000838.png)

  * RUNNING 运行状态，接受task，执行排队中task
  * SHUTDOWN 不接受新task，执行排队中task
  * STOP 不接受新task，不执行排队中task
  * TIDYING 所有task终止，`workerCount`为0，转化为该状态后，执行hook `terminated()`
  * TERMINATED hook`terminated()`执行完毕

* <a id=hook>Hook</a>

  * beforeExecute `Worker`开始执行前执行

    ```java
    protected void beforeExecute(Thread t, Runnable r) { }
    ```

  * afterExecute  `Worker`执行完时执行

    ```java
    protected void afterExecute(Runnable r, Throwable t) { }
    ```

  * terminated 线程池状态转化为`TIDYING`时执行

    ````java
     protected void terminated() { }
    ````

* <a id=code>核心代码(线程池)</a>
  * `submit`

    ```java
      public Future<?> submit(Runnable task) {
             if (task == null) throw new NullPointerException();
             RunnableFuture<Void> ftask = newTaskFor(task, null);
          	// ThreadPoolExecutor中实现
             execute(ftask);
             return ftask;
         }
     public <T> Future<T> submit(Runnable task, T result) {
             if (task == null) throw new NullPointerException();
             RunnableFuture<T> ftask = newTaskFor(task, result);
             execute(ftask);
             return ftask;
         }
     public <T> Future<T> submit(Callable<T> task) {
             if (task == null) throw new NullPointerException();
             RunnableFuture<T> ftask = newTaskFor(task);
             execute(ftask);
             return ftask;
         }
    ```

   

   * `newTaskFor`

     ```java
      protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
           return new FutureTask<T>(runnable, value);
       }
      protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
           return new FutureTask<T>(callable);
       }
     ```
     
  * `execute(Runnable command)` 

       <font color=grey>submit task实际执行execute</font>

     ```java
      public void execute(Runnable command) {
                 if (command == null)
                     throw new NullPointerException();
                 int c = ctl.get();
             // 1.第一种情况 线程数小于corePoolSize,新增worker线程并启动执行task
                 if (workerCountOf(c) < corePoolSize) {
                     if (addWorker(command, true))
                         return;
                     c = ctl.get();
                 }
             //2.将task加入阻塞队列中
                 if (isRunning(c) && workQueue.offer(command)) {
                     int recheck = ctl.get();
                     if (! isRunning(recheck) && remove(command))
                         reject(command);
                     else if (workerCountOf(recheck) == 0)
                         addWorker(null, false);
                 }
           //3.线程池非RUNNING状态或task队列已满，添加worker线程，此时线程池最大数量为maximumPoolSize
                 else if (!addWorker(command, false))
                   reject(command);
             }
     ```

   

  * `addWorker(Runnable firstTask,boolean core)`

    <font color=grey> 线程池创建Worker线程</font>
    
    ```java
     private boolean addWorker(Runnable firstTask, boolean core) {
            retry:
            for (;;) {
                int c = ctl.get();
                int rs = runStateOf(c);
    
                // SHUTDOWN状态不接受task，只执行排队中的task
                if (rs >= SHUTDOWN &&
                    ! (rs == SHUTDOWN &&
                       firstTask == null &&
                       ! workQueue.isEmpty()))
                    return false;
    
                for (;;) {
                    int wc = workerCountOf(c);
                    // 线程数不能超过2^29 - 1 不能超过核心线程数或最大线程数
                    if (wc >= CAPACITY ||
                        wc >= (core ? corePoolSize : maximumPoolSize))
                        return false;
                    if (compareAndIncrementWorkerCount(c))
                        break retry;
                    // 1.cas更新ctl失败，说明ctl发生改变，
                    // 2.重新获取ctl
                    // 3.如果线程池状态改变，进入外层循环
                    // 4.否则是线程数量发生改变进入内层循环
                    c = ctl.get();  // Re-read ctl
                    if (runStateOf(c) != rs)
                        continue retry;
                    
                }
  	        }
         	//........
    	// 1.创建新worker线程
         	// 2.加入到线程池中Set<Worker>
       	// 3.启动worker线程 执行runWorker(this)
      }
    ```

  * `runWorker(Worker w)` 运行Worker线程

    <font color=grey>`Worker`线程运行时，循环从task队列中获取task并执行</font>

    ```java
      final void runWorker(Worker w) {
              Thread wt = Thread.currentThread();
              Runnable task = w.firstTask;
              w.firstTask = null;
              w.unlock(); // allow interrupts
              boolean completedAbruptly = true;
              try {
                  // 1.获取创建worker时的首个task，没有task则调用getTask()从task阻塞队列中获取
                  // 2.获取task之后执行task，发生异常跳出循环
                  // 3.获取不到task结束循环
                  while (task != null || (task = getTask()) != null) {
                      w.lock();
                      if ((runStateAtLeast(ctl.get(), STOP) ||
                           (Thread.interrupted() &&
                            runStateAtLeast(ctl.get(), STOP))) &&
                          !wt.isInterrupted())
                          wt.interrupt();
                      try {
                          beforeExecute(wt, task);
                   		// ......
                          task.run();
                          // ......
                          } finally {
                              afterExecute(task, thrown);
                          }
                      } finally {
                          task = null;
                          w.completedTasks++;
                          w.unlock();
                      }
                  }
                  completedAbruptly = false;
              } finally {
        		// 4.退出worker线程
                  processWorkerExit(w, completedAbruptly);
            }
          }
    ```
    
    
    
  * `getTask()` 

    <font color=grey>worker线程从Task队列中获取task </font>

    获取不到task的情况(导致worker线程结束的一种情况): 

    1. SHUTDOWN状态，workQueue为空
    2. STOP及之后状态
    3. 获取task超时 或 workerCount超过maximumPoolSize 且还有其他worker或workQueue为空

    ```java
    private Runnable getTask() {
              boolean timedOut = false; // Did the last poll() time out?
              for (;;) {
                  int c = ctl.get();
                  int rs = runStateOf(c);
                  // SHUTDOWN状态，排队的task全部执行完 或者STOP状态之后 workerCount减1
                  if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
                      decrementWorkerCount();
                      return null;
                  }
                  int wc = workerCountOf(c);
                  // Are workers subject to culling?
                  boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
      			// workerCount超过maximumPoolSize或者worker等待获取task超时
                 	// workerCount减1,即将在调用getTask()的地方(runWorker)关闭当前worker
                  if ((wc > maximumPoolSize || (timed && timedOut))
                      && (wc > 1 || workQueue.isEmpty())) {
                      if (compareAndDecrementWorkerCount(c))
                          return null;
                      continue;
                  }
                  try {
                      Runnable r = timed ?
                          workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                          workQueue.take();
                      if (r != null)
                          return r;
                      timedOut = true;
                  } catch (InterruptedException retry) {
                    timedOut = false;
                  }
          }
      }
    ```

    

    

  * `processWorkerExit(Worker w,boolean completedAbruptly)`

    <font color=grey> 关闭worker线程</font>

    ```java
     private void processWorkerExit(Worker w, boolean completedAbruptly) {
         // workerCount调整发生在getTask()中，异常时workerCount未调整
            if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
                decrementWorkerCount();
    
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                completedTaskCount += w.completedTasks;
                //1.从Set<Worker>中移除worker
                workers.remove(w);
            } finally {
                mainLock.unlock();
            }
    		//2.尝试关闭线程池
            tryTerminate();
    
            int c = ctl.get();
            if (runStateLessThan(c, STOP)) {
                if (!completedAbruptly) {
                    // min需要的最小Worker数量
                    // allowCoreThreadTimeOut为true时，completedAbruptly为false说明worker获取task超时导致worker结束，说明task队列为空
                    int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
                    if (min == 0 && ! workQueue.isEmpty())
                        min = 1;
                    if (workerCountOf(c) >= min)
                        return; // replacement not needed
                }
                // 3.线程池处于可以执行task的状态中
                // 3.1 当前worker异常，重新添加一个worker
                // 3.2 当前线程数量小于需要的最小的线程数量
              // 3.3 添加一个新的worker
                addWorker(null, false);
          }
        }
    ```

    

  * `tryTerminate()` 

    <font color=grey>尝试关闭线程池</font>

    ```java
    final void tryTerminate() {
            for (;;) {
                int c = ctl.get();
                if (isRunning(c) ||
                    runStateAtLeast(c, TIDYING) ||
                    (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
                    return;
                if (workerCountOf(c) != 0) { // Eligible to terminate
                    interruptIdleWorkers(ONLY_ONE);
                    return;
                }
    
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // 1.STOP状态或者SHUTDOWN状态且workQueue为空
                    // 2.workerCount为0
                    // 3.满足以上两种条件 更新为TIDYING状态 执行Hook terminated()
                   
                    if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                        try {
                            terminated();
                        } finally {
                             // 4.terminated()执行完更新为TERMINATED状态
                            ctl.set(ctlOf(TERMINATED, 0));
                            // 5.唤醒调用了awaitTerminated()的地方
                            termination.signalAll();
                        }
                        return;
                    }
                } finally {
                    mainLock.unlock();
            }
                // else retry on failed CAS
        }
        }
    ```

  * `shutdown和shutdownNow`

    作用：

    1. 更新线程池状态
    2. 中断所有worker线程
    3. 清空workQueue，并返回未完成的task(只有shutdownNow执行该步骤)
    4. 调用tryTerminated尝试关闭线程池

    ```java
     public void shutdown() {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                checkShutdownAccess();
                advanceRunState(SHUTDOWN);
                interruptIdleWorkers();
                onShutdown(); // hook for ScheduledThreadPoolExecutor
            } finally {
                mainLock.unlock();
            }
            tryTerminate();
        }
    
     public List<Runnable> shutdownNow() {
            List<Runnable> tasks;
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                checkShutdownAccess();
                advanceRunState(STOP);
                interruptWorkers();
                tasks = drainQueue();
            } finally {
                mainLock.unlock();
          }
            tryTerminate();
          return tasks;
        }
    ```

  

  * interruptIdleWorkers(boolean onlyOne)

    <font color=grey>中断worker线程,shutdown、shutdownNow等地方调用</font>

    ```java
      private void interruptIdleWorkers(boolean onlyOne) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                for (Worker w : workers) {
                    Thread t = w.thread;
                    // 如果当前worker线程正在运行，tryLock无法获取到锁无法中断worker
                    // runWorker中存在对STOP状态下确保worker中断的操作
                    // SHUTDOWN状态则不需要确保worker中断，SHUTDOWN状态需要worker完成所有workQueue中task，workQueue为空，worker获取不到task，会自动关闭
                    if (!t.isInterrupted() && w.tryLock()) {
                        try {
                            t.interrupt();
                        } catch (SecurityException ignore) {
                        } finally {
                            w.unlock();
                        }
                    }
                    if (onlyOne)
                        break;
                }
          } finally {
                mainLock.unlock();
            }
        }
    ```

  

* <a id=stream>流程图</a>

  ![线程池工作流程](https://gitee.com/zzz_123456/picgo/raw/main/image/线程池工作流程.jpg)