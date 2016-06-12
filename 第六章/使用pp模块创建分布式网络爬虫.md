目前我们已经使用pp组件在本机上实现多进程并发，接下来我们将在分布式环境下使用pp组件，分布式硬件环境如下：    

- Iceman-Thinkpad-X220: Ubuntu 13.10
- Iceman-Q47OC-500P4C： Ubuntu 12.04 LTS
- Asgard-desktop: Elementary OS

我们将在如上列举的三台机器上测试pp组件在分布式环境下的使用。对此，我们实现了分布式网络爬虫。web_crawler_pp_cluster.py方法中，将input_list列举的URL分发到本地以及远端进程执行，web_crawler_pp_cluster.py中的回调函数将组织这些URL以及以及通过它们找到的前三个连接(URL)。     

让我们分析代码，一步步理解怎样实现上述功能。首先import使用到的库以及定义数据结构。然后定义input_list列表用于存放入口URL和result_dict存放爬取结果。代码如下所示：     

接下来分析aggregate_results方法，该方法还是作为回调函数，相对于前一小结的aggreate_results方法，改变不多。返回meesage的格式的发生了变化，aggregate_resullt方法的传入参数变成tuple，分别保存执行该方法的PID号，hostname和找到的前三个URL。代码如下：    

接下来定义crawl_task方法，作为Server类方法。和之前章节的crawl_task方法功能类似，依据传入的RUL获取网页上的其他链接(RUL),唯一的不同是返回值是tuple，代码如下：   

在main方法和callback方法定义之后，我们需要初始化Server类实例，以至于能够在分布式环境下执行网络爬虫任务。我们注意到pp.Server类有三个参数，第一个参数是执行Server类方法的IP或hostname，我们的例子中，除了本机之外，还需要定义另外两台机器的IP和hostname，定义如下所示：    

pp.Server类的初始化如下：    

我们注意到初始化有三个参数，第一个参数不同的是被赋值为1，使得pp组件创建一个本地进程，如果需要的话，把其他任务转发到远端机器上执行。第二个参数是之前定义的ppservers。第三个参数定义socket连接超时时间，通常为了测试目的，我们把timeout时间设置很长，防止因为网络超时而导致socket连接终止。    

Server类创建之后，我们将遍历url_list，并提交crawl_task方法。    

相对于之前的之前章节，变化是传入的组件较之于斐波那契序列不同。    

我们需要等待网络爬虫执行完毕，得到爬虫结果。     

执行程序之前，我们需要执行远端机器上的ppserver.py模块。Python ppserver.py -a -d 为shell命令。-a选项用于设置自动发现，使得client自动发现网络中没有设置ip的server。-d选项设置debug模式，显示程序执行过程中产生的log。     

接下来我们定义输出格式：    
 
1. 首先，下面的屏幕显示main节点的执行信息，包括执行和分发远端任务。    
2. 然后，执行ppserver.py脚本，执行任务信息如下屏幕内容所示。       
3. 在之前屏幕中，我们打印了有趣的统计信息，包括在远端机器上执行的任务数量，每个人物执行的时间和每个远端机器上的任务总数。截图上反应的另外一个信息是，我们应该限制callback函数任务量，因为callback函数都是在main节点上执行，这可能会成为成为整个系统的瓶颈，当然也取决于不同的应用场景。     
4. 下面的截图是的debug模式下ppserver.py脚本在iceman-Q470C-500P4C机器上的的执行日志。     
5. 下面截图是debug模式下ppserver.py脚本在asgard-desktop机器上的执行日志。      



