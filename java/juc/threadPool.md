### 线程池浅析

* [线程池创建及使用](#create)

* 工作原理概括

* 线程池uml

* [核心组件分析](#compent)

* [核心变量分析](#param)

* 源码分析

* 如何优雅使用

* 推荐书籍

* 扩展点

* 感想

  

> <a id=create>线程池创建及使用</a>

> <a id=compent>核心组件分析</a>

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

  ​	

> <a id=param>核心变量分析</a>

* ctl 包含runState、workCount

* workQueue
* mainLock
* workers
* termination
* keepAliveTime
* corePoolSize
* maximumPoolSize

