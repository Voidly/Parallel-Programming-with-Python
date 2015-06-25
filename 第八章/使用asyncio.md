我们可以定义asyncio是一个Python中驱动异步编程的模块。ayncio模块使用下列组合来实现异步编程：
- Event loop: asyncio模块允许每个进程一个事件循环。
- Coroutines(协程): asyncio的官方文档中指出，coroutine是遵循一定规则的发生器。它最吸引人的特点是在执行的时候能够暂停等待外部处理，当外部处理完成后又可以从原来的位置恢复执行。
- Futures: Futures代表尚未完成的processing。
- Tasks: 是asyncio.Future的子类，用于管理coroutines。

除了这些机制，asyncio还为应用开发提供了很多其他机制，比如传输和协议，可以使用TCP、SSL、UDP和管道进行通信。关于asyncio更多的信息请查看 https://docs.python.org/3.4/library/asyncio.html。

### 理解coroutines和rutures
为了在asyncio中定义coroutine，我们使用@asyncio.coroutine装饰器。为了执行一个操作I/O或者其他可能阻塞循环事件的计算，我们必须使用`yield from`语法来暂停coroutine。但是暂停和恢复的机制怎样工作？Coroutine和asyncio.Future对象一起工作。我们可以总结操作如下：
- Coroutine初始化，asyncio.Future在内部实例化或者作为参数传给coroutine。
- 到达coroutine使用yield from的地方，coroutine暂停来等待yield from引发的计算。yield from实例等待yield from\<coroutine or asyncio.Future or asyncio Task\>构建。
- 当yield from引发的计算结束，coroutine执行与coroutine关联的asyncio.Future对象的set_result(\<result\>)方法，通知事件循环coroutine可以被恢复。


#### 使用coroutine和asyncio.Future
下面是使用coroutine和asyncio.Future对象的一些例子：
	import asyncio
	@asyncio.coroutine
	def sleep_coroutine(f):
		yield from asyncio.sleep(2)
		f.set_result("Done!")
在上述代码中，定义了一个协程sleep_coroutine，它接收一个asyncio.Future对象作为参数。在sleep_coroutine中，asyncio.sleep(2)会让协程睡眠2秒，asyncio.sleep已经和asyncio兼容。
在主函数中创建asyncio.Future对象，创建event loop对象。
	if __name__ == '__main__':
		future = asyncio.Future()
		loop = asyncio.get_event_loop()
		loop.run_until_complete(sleep_coroutine(future))

> event loop执行的时候，任务和协程才会执行。

在最后一行，loop.run_until_complete(sleep_coroutine(future))，很明显就是运行直到sleep_coroutine结束。

#### 使用asyncio.Task
asyncio.Task是asyncio.Future的子类，目的是管理协程。以下是一个例子，多个asyncio.Task将会在事件循环中被创建和分派。

	import asyncio
	@asyncio.coroutine
	def sleep_coro(name, seconds=1):
		print("[%s] coroutine will sleep for %d second(s)…"
				% (name, seconds))
		yield yfrom asyncio.sleep(seconds)
		print("[%s] done!" % name)

sleep_coro协程会接收两个参数，name用来标识协程，seconds用来定义睡眠时间。

在主函数中，定义了一个包含三个asyncio.Task对象的列表：

	if __name__ == '__main__':
		tasks = [asyncio.Task(sleep_coro('Task-A', 10)),
				asyncio.Task(sleep_coro('Task-B')),
				asyncio.Task(sleep_coro('Task-C'))]
		loop.run_until_complete(asyncio.gather(*tasks))

程序的运行结果如下：
![](图片链接地址)

值得注意的是，程序的输出表明任务执行的顺序和申明的顺序一致，它们都不能阻塞event loop。

#### 使用和asyncio不兼容的库

asyncio是python新加入的模块，一些库还不能很好的兼容。我们重新实现之前章节的例子asyncio_task_sample.py，用time.sleep替换asyncio.sleep。运行结果如下：

![](图片链接地址)

