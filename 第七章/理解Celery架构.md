Celery架构基于可插拔组件和用*message transport(broker)*实现的消息交换机制。具体的如下图所示：
![](图片链接地址)

现在，让我们详细的介绍Celery的每个组件。

### 处理任务

在上图中的*client*组件，有创建和分派任务到brokers的方法。
分析如下示例代码来演示通过使用@app.task装饰器来定义一个任务，它可以被一个Celery应用的实例访问，下面代码展示了一个简单的`Hello World app`：

	@app.task
	def hello_world():
		return "Hello I'm a celery task"


> 任何可以被调用的方法或对象都可以成为任务

正如我们之前所说，有各种类型的任务：同步、异步、周期和计划任务。当我们调用任务，它返回类型AsyncResult的实例。AsyncResult对象中可以查看任务状态，当任务结束后可以查看返回结果。然而，为了利用这个机制，另一个叫做*result backend*的组件必须启动，这将在本章的后面讲解。为了分派任务，我们可以用下面这些方法：
- delay(arg, kwarg=value) : 它会调用apply_aync方法。
- apply_async((arg,), {'kwarg': value}) : 该方法可以为任务设置很多参数，一些参数如下。
 - countdown : 默认任务是立即执行，该参数设置经过countdown秒之后执行。
 - expires : 代表经过多长时间终止。
 - retry : 如果连接或者发送任务失败，该参数可是重试。
 - queue : 任务队列。
 - serializer : 数据格式，其他还有json、yaml等等
 - link : 连接一个或多个即将执行的任务。
 - link_error : 连接一个或多个执行失败的任务。
- apply((arg,), {'kwarg': value}) : 在本地进程以同步的方式执行任务，因而阻塞直到结果就绪。

> Celery 提供了查看任务状态的机制，这在跟踪进程的真实状态非常有用。更多的关于内建任务状态的资料请查看http://celery.readthedocs.org/en/latest/reference/celery.states.html

### 发现消息传输(broker)

broker是Celery中的核心组件，通过broker可以发送和接受消息，来完成和workers的通信。Celery支持大量的brokers。然而，对于某些broker，不是所有的Celery机制都实现了。实现功能最全的是**RabbitMQ**和**Redis**。在本书中，我们将采用Redis作为broker。broker提供在不同客户端应用之间通信的方法，客户端应用发送任务，workers执行任务。可以有多台带有broker的机器等待接收消息，然后发送消息给workers。


### 理解workers

Workers负责执行接收到的任务。Celery提供了一系列的机制，我们可以选择最合适的方式来控制workers的行为。这些机制如下：
- 并发模式：例如进程、线程、协程(Eventlet)和Gevent
- 远程控制：利用该机制，可以通过高优先级队列发送消息到某个特定的worker来改变行为，包括runtime。
- 撤销任务：利用该机制，可以指挥一个或多个workers来忽略一个或多个任务。

更多的特性可以在运行时设定或者改变。比如，worker在某一段时间执行的任务数，worker在哪个队列消费等等。软玉worker更多的信息可以参考  http://docs.celeryproject.org/en/latest/userguide/workers.html#remote-control 

### 理解result backends

Result backend组件存储任务的状态和任务返回给客户端应用的结果。Celery支持的result backend之中，比较出彩的有 RabbitMQ, Redis, MongoDB, Memcached。每个result backend都有各自的优缺点，详见  http://docs.celeryproject.org/en/latest/userguide/tasks.html#task-result-backends

现在，我们对Celery架构有了一个大概的认识。下面我们建立一个开发环境来实现一些例子。