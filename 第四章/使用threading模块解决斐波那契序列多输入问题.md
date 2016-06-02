##使用多线程解决斐波那契序列多输入问题
接下来我们将实践python多线程的使用。任务是并行处理斐波那契序列多输入值。为了把问题说清楚，我们将把输入分成四部分，四个线程分别处理一个输入数据。算法描述如下：

1. 首先使用一个列表存储四个待输入值，这些值将被放入对于线程来说互相锁定的数据结构。
2. 输入值被放入可被锁定的数据结构之后，负责处理斐波那契序列的线程将被告知可以被执行。这时，我们可以使用python线程的同步机制Condition模块（Condition模块对共享变量提供线程之间的同步操作），模块详情请参考：[http://docs.python.org/3/library/threading.html#threading.Condition](http://docs.python.org/3/library/threading.html#threading.Condition)。
3. 当每个线程结束斐波那契序列的计算后，分别把结果存入一个字典。

接下来我们将列出代码，并且讲述其中有趣的地方：

代码开始处我们加入了对编码额外的支持，导入logging, threading和queue模块。此外，我们还定义了我们例子中用到主要数据结构。一个字典，被命名为fibo_dict，将用来存储输入输出数据，输入数据为key，计算结果（输出数据）为值。我们同样定义了一个队列对象，该对象中存储线程间的共享数据（包括读写）。我们把该对象命名为shared\_queue。最后我们定义一个列表模拟程序的四个输入值。代码如下：

```python

    #coding: utf-8
    import logging, threading
    from queue import Queue
    logger = logging.getLogger()
    logger.setLevel(logging.DEBUG)
    formatter = logging.Formatter('%(asctime)s - %(message)s')
    ch = logging.StreamHandler()
    ch.setLevel(logging.DEBUG)
    ch.setFormatter(formatter)
    logger.addHandler(ch)
    fibo_dict = {}
    shared_queue = Queue()
    input_list = [3, 10, 5, 7]
```

接下来的一行代码，我们从threading模块中定义了一个Condition对象，该对象根据一定的条件同步各线程存取资源的操作。

```python

    queue_condition = threading.Condition()
```

使用Condition对象用于控制线程的创建队列。

下一块代码定义了一个很多线程都需要调用的方法，我们把它命名为fibonacci\_task。fibonacci\_task方法接收condition对象作为线程获取shared\_queue中值的协议。方法中，我们使用了with语句（关于更多with语句的用法，请参考[http://docs.python.org/3/reference/compound_stmts.html#with](http://docs.python.org/3/reference/compound_stmts.html#with)）简化控制内容。如果没有with语句，我们则需要显式的使用锁，并且最后释放锁。有了with操作，代码隐式的在代码最开始获得锁，并在代码最后释放锁。fibonacci方法中接下来的是逻辑处理相关代码，告诉当前线程，当shared\_queue为空时，等待。wait()方法是condition中的主要方法之一。线程将一直等待，直到被通知shared\_queue可以被使用。一旦满足shared\_queue可以被使用的条件，当前线程将接收shared\_queue中的值作为输入计算斐波那契序列的值，最后把输入和输出作为key和value存入fibo\_dict字典。最后，我们调用task_done()方法，通知某一个任务已经被分离并执行。代码如下：

```python

    def fibonacci_task(condition):
     with condition:
       while shared_queue.empty():
         logger.info("[%s] - waiting for elements in queue.."
           % threading.current_thread().name)
         condition.wait()
     else:
       value = shared_queue.get()
       a, b = 0, 1
       for item in range(value):
         a, b = b, a + b
         fibo_dict[item] = a    # 这里书中是fibo_dict[value] = a,但是觉得重复赋值没有意义
     shared_queue.task_done()
     logger.debug("[%s] fibonacci of key [%d] with
       result [%d]" %
       (threading.current_thread().name, value,
         fibo_dict[value]))
```

我们定义的第二个函数是queue\_task，该函数被负责计算shared\_queue的值的线程所调用。我们看到condition对象作为获得shared\_queue的协议。input\_list中的每一个值都将被插入到shared\_queue中去。当所有的值都被插入到shared\_queue中后，告知负责计算斐波那契序列的方法shared\_queue已经可以使用。

```python

    def queue_task(condition):
     logging.debug('Starting queue_task...')
     with condition:
       for item in input_list:
         shared_queue.put(item)
         logging.debug("Notifying fibonacci_task threads
           that the queue is ready to consume..")
         condition.notifyAll()
```

接下来我们将创建四个线程等待shared\_queue可以被使用条件。线程将执行target参数作为回调函数，代码如下：

```python

    threads = [threading.Thread(
      daemon=True, target=fibonacci_task,
    args=(queue_condition,)) for i in range(4)]
```

接着我们使用thread对象的start方法开始线程：

```python

    [thread.start() for thread in threads]
```

然后我们创建一个线程处理shared\_queue，然后执行该线程。代码如下：

```python

    prod = threading.Thread(name='queue_task_thread', daemon=True,
     target=queue_task, args=(queue_condition,))
    prod.start()
```

最后，我们多计算斐波那契序列的所有线程都调用join()方法，使用join()方法的目的是，让主线程等待子线程的调用，直到所有子线程执行完毕之后才结束子线程。

```python

    [thread.join() for thread in threads]
```

程序的执行结果如下：

注意到，第一个fibonacci\_task线程被创建和初始化后，它们进入等待状态。同时，queue\_task线程被创建并且生成shared\_queue队列。最后，queue\_task方法告知fibonacci_task线程可以执行它们的任务。

注意到，程序每次执行的过程都不一样，这也是多线程的特性之一。
