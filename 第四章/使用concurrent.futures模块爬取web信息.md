 ##使用concurrent.futures模块爬取web信息

下面的章节会实现一个并行的web爬虫.

在实现时,我们会应道concurrent.futures模块中一个很有意思的类,叫做ThreadPoolExecutor 在上一章节的例子中,我们分析了parallel\_fibonacci.py是如何实现并发的,它只是以最原始的方式来使用进程,在某一特定的时候需要我们手工来创建和初始化一个个线程. 然而在大型程序中还想这样手工管理线程就太困难了. 在开发大型程序时,我们常常要用到线程池机制. 线程池是一种用于在进程中管理预先创建的多个线程的一种数据结构. 使用线程池的目的是为了复用线程,这样就可以避免不断的创建线程所照成的资源浪费.

基本上和前一章节一样, 我们将会设计一个算法,该算法分阶段地执行一些任务,并且这些任务也会相互影响. 下面,让我们分析一下这个并发网络爬虫的代码

在导入必要的模块,并设置好日志文件后,我们使用内置模块re来创建一个正则表达式(re模块的完整文档可以在http://docs.python.org/3/howto/regex.html中找到). 我们会使用该正则表达式来过滤爬取阶段返回的连接集合. 相关代码如下所示:

```python
html_link_regex = \
re.compile('<a\s(?:.*?\s)*?href=[\'"](.*?)[\'"].*?>')
```

接下来我们创建一个同步队列来模拟输入数据. 然后我们创建一个名为result\_dict的字典实例. In this, we will correlate the URLs and their respective links as a list structure. 相关代码如下:

```python
urls = queue.Queue()
urls.put('http://www.google.com')
urls.put('http://br.bing.com/')
urls.put('https://duckduckgo.com/')
urls.put('https://github.com/')
urls.put('http://br.search.yahoo.com/')
result_dict = {}
```

再接下来我们定义一个名为group\_urls\_task的函数,该函数用于从同步队列中抽取出URL并存入result\_dict的key值中. 另一个应该留意的细节是,我们调用Queue的get方法是,带了两个参数,第一个参数为True表示阻塞其他线程访问这个同步队列,第二个参数是0.05表示阻塞的超时事件,这样就防止出现由于同步队列中没有元素而等待太长事件的情况出现. 毕竟,在某些情况下,你不会想化太多的时间来等待新元素的到来. 相关代码如下:

```python
def group_urls_task(urls):
    try:
        url = urls.get(True, 0.05)
        result_dict[url] = None
        logger.info("[%s] putting url [%s] in dictionary..." % (
            threading.current_thread().name, url))
    except queue.Empty:
        logging.error('Nothing to be done, queue is empty')
```

现在我们需要有一个在爬行阶段执行的任务,该任务将每个url作为参数传递给一个名为crawl\_task的函数. 当将URL所指页面中的所有连接都保存下里之后,爬行阶段就算是完成了. 爬行过程中会返回一个元组,且该元组的第一个元素就是传递給crawl\_task函数的URL参数. 在这个步骤中,会从URL所指页面中抽取出一个连接的列表. 获取URL所指网页的内容需要用到request模块(关于request模块的官方文档请参见https://pypi.python.org/pypi/requests )

```python
def crawl_task(url):
    links = []
    try:
        request_data = requests.get(url)
        logger.info("[%s] crawling url [%s] ..." % (
            threading.current_thread().name, url))
        links = html_link_regex.findall(request_data.text)
    except:
        logger.error(sys.exc_info()[0])
        raise
    finally:
        return (url, links)
```

进一步分析代码,我们会发现创建了一个concurrent.futures模块中定义的ThreadPoolExecutor对象(关于ThreadPoolExecutor对象的详细信息,请参见 http://docs.python.org/3.3/library/concurrent.futures.html#concurrent.futures.ThreadPoolExecutor)在这个ThreadPoolExecutor对象的构造函数中有一个名为max\_workers的参数,该参数决定了该executor所包含的线程池中的线程数. Within the stage of removal of the URLs from the synchronized queue and insertion of keys into result\_dict, the choice was between using three worker threads.(*这一段不知道怎么翻译*) 该数量可以根据问题的大小而改变. 定义完ThreadPoolExecutor之后,我们还使用with语句来保证结束的清理动作会被执行. 这些清理动作会在超出with语句的作用域时被执行. 在ThreadPoolExecutor对象的作用域内,我们遍历同步队列并且通过ThreadPoolExecutor对象的submit方法来将同步队列作为包含URL的队列引用传递給group\_urls\_task函数. 总之,submit方法接受一个要执行的回调函数及其参数并返回一个Future对象,该Future对象会在未来的某个时候执行该回调函数. 就我们的例子中,该回调函数就是group\_urls\_task,而参数就是同步队列的引用. 然后线程池中的线程就会并行且异步地执行Future对象中预定的函数. 相关代码如下:

```python
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as\
    group_link_threads:
    for i in range(urls.qsize()):
        group_link_threads.submit(group_urls_task, urls)
```

随后,我们还要再创建一个ThreadPoolExecutor对象, 不过这一次我们使用上一阶段中group\_urls\_task所产生的key值作为参数来执行爬行的动作. 这一次我们所使用的代码有些不同.
```python
future_tasks = {crawler_link_threads.submit(crawl_task, url): url  
    for url in result_dict.keys()}
```

我们映射了一个名为future\_tasks的临时字典对象. 该字段对象包含了submit方法所创建的Future对象,且创建这些Future对象时所使用的参数是result\_dict中的每个URL. 也就是说,根据result\_dict中的每个key,我们创建了future\_tasks中的每个任务. 映射完这个字典对象后,我们还需要搜集这些Future对象执行的结果. 搜集执行结果的方法是,使用concurrent.futures.as\_completed(fs,timeout=None)函数来循环遍历执行futre\_tasks中的各个对象, concurrent.futures.as\_completed(fs, timeout=None)方法会返回一个Future对象的迭代器. 这样我们可以遍历得到这些Future对象的执行结果. 在ThreadPoolExecutor的最后,我们在每个爬行线程中都调用了Future对象的result()方法. 在我们这个例子中,该方法返回结果元组. 这样我们最终得到的future\_tasks结果如下所示.

又一次,我们可以发现每个线程池中的线程执行是乱序的,但这不重要,重要的是,result\_dict中输出的内容就是最终结果.
