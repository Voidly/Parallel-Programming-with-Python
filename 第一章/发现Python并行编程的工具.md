##发现Python并行编程的工具

由Guido Van Rossum创造的语言，是一种多泛型的，多用途的语言。由于它非常简单且易于维护，被世界各处广泛接受。它也被称为含有电池的语言。它有广泛的模块使其用起来更流畅。在并行编程中，Python有简化实现的内置和外部模块。这是基于Python3.X的。

###Python线程模块

Python线程模块提供了一个抽象层次的模块_thread，这是一个低层次的模块。当开发一个基于线程的并行系统的艰巨任务时，它为程序员提供了一些函数来帮助程序员的开发。线程模块的官方文档可以在<http://docs.python.org/3/library/
threading.html?highlight=threading#module-threadin>找到。

###Python多进程模块

多进程模块旨在为基于进程的并行的使用提供一个简单的API。这个模块与线程模块类似，它简化了进程间的交替而没有什么难度。在Python社区中，基于进程的方法很流行，因为它是在回答出现在Python中CPU-Bound threads和GIL的使用的问题时的一个解决方案。多进程模块的官方文档可以在<http://docs.python.org/3/library/multiprocessing.html?highlight=multi
processing#multiprocessing>找到。

###并行Python模块

并行Python模块是外部提供的，它提供了丰富的API，这些API利用进程的方法创建并行和分布式系统。这个模块是轻量级并且易安装的，它与其他的Python程序一起集成的。并行模块可以在<http://parallelpython.com>找到。在那么多特性中，我们着重强调以下几点：

* 最优配置的自动检测
* 运行时可以改变多个工作进程
* 动态的负载均衡
* 容错性
* 自发现计算资源

###Celery-分布式任务队列

Celery是一个用于创建分布式系统的极其优秀的模块，并且拥有很好的文档。它在并发形式上使用了至少三种不同类型的方法来执行任务：multiprocessing, Eventlet,和 Gevent。这项工作将会集中精力在多进程的方法的使用上。而且，一个与另一个的连接就是配置问题，这被留下来作为一个研究，以便读者能够建立一个与他/她的实验的一个比较。

Celery模块可以在官方的项目页面<http://celeryproject.org>得到。


