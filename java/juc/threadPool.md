# 线程池浅析
* [问题](#questions)
* [线程池创建及使用](#create)
* [工作原理概括](#general)
* [线程池生命周期](#lifetime)
* [线程池uml](#uml)
* [核心组件分析](#compent)
* [核心变量分析](#param)
* [源码分析](#source)
* [如何优雅使用](#use)
* [推荐书籍](#book)


>  <a id=questions>问题</a>

* ctl的作用
* 为什么Worker要实现AQS而不使用显示锁
* BlockQueue中add/remove offer/poll put/take 区别
* 中断机制
* 线程池生命周期
* 线程池容量如何动态伸缩

>  <a id=create>线程池创建及使用</a>



>  <a id=general>工作原理概括</a>



![线程池工作原理概括(2)](https://gitee.com/zzz_123456/picgo/raw/main/image/线程池工作原理概括(2).png)




>  <a id=lifetime>线程池生命周期</a>



![threadPoolLifetime](https://gitee.com/zzz_123456/picgo/raw/main/image/threadPoolLifetime.png)



>  <a id=uml>线程池uml</a>



![image-20210804134139734](https://gitee.com/zzz_123456/picgo/raw/main/image/image-20210804134139734.png)





>  <a id=compent> 核心组件分析</a>

* `ThreadFactory`

  * `DefaultThreadFactory`,创建的线程拥有相同的group、name、priority

* `RejectedExecutionHandler`

  默认四种实现：

  * DiscardPolicy

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            }
    ```

  * DiscardOldestPolicy

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                if (!e.isShutdown()) {
                    e.getQueue().poll();
                    e.execute(r);
                }
            }
    ```

  * AbortPolicy

    ```java
     public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                throw new RejectedExecutionException("Task " + r.toString() +
                                                     " rejected from " +
                                                     e.toString());
            }
    ```

  * CallerRunsPolicy

    ```java
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
                if (!e.isShutdown()) {
                    r.run();
                }
            }
    ```

* `RunnableFuture`

>  <a id=param>核心变量分析</a>

* ctl 包含runState、workCount

* workQueue

* mainLock

* workers

* keepAliveTime

* allowCoreThreadTimeOut

* corePoolSize

* maximumPoolSize

> <a id=source> 源码分析</a>

* execute(Runnable command)
* addWorker(Runable firstTask, boolean core)
* runWorker(Worker w)
* getTask()
* processWorkerExit(Worker w, boolean completeAbruptly)
* shutdown()
* shutdownNow()
* interruptIdleWorkers(boolean onlyOne)
* interruptWorkers()
* tryTerminate()

> <a id=use>如何优雅地使用</a>

* 线程池大小的设置
* 任务队列大小的设置
* 自定义失败策略
* 不同任务使用不同的线程池


> <a id=book>推荐书籍</a>

![image-20210804112051353](https://gitee.com/zzz_123456/picgo/raw/main/image/image-20210804112051353.png)



![image-20210804112130655](https://gitee.com/zzz_123456/picgo/raw/main/image/image-20210804112130655.png)
