# Python基本

## 数据结构



https://www.learnfk.com/python-data-structure/python-stack.html



poco.exceptions.PocoNoSuchNodeException: Cannot find any visible node by query UIObjectProxy of "com.netease.cloudmusic:id/musicInfoList/com.netease.cloudmusic:id/musicListItemContainer[0]>com.netease.cloudmusic:id/songRank"

## python多线程

多线程类似于同时执行多个不同程序，多线程运行有如下优点：

- 使用线程可以把占据长时间的程序中的任务放到后台去处理。
- 用户界面可以更加吸引人，这样比如用户点击了一个按钮去触发某些事件的处理，可以弹出一个进度条来显示处理的进度
- 程序的运行速度可能加快
- 在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。在这种情况下我们可以释放一些珍贵的资源如内存占用等等。

线程在执行过程中与进程还是有区别的。每个独立的进程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

每个线程都有他自己的一组CPU寄存器，称为线程的上下文，该上下文反映了线程上次运行该线程的CPU寄存器的状态。

指令指针和堆栈指针寄存器是线程上下文中两个最重要的寄存器，线程总是在进程得到上下文中运行的，这些地址都用于标志拥有线程的进程地址空间中的内存。

- 线程可以被抢占（中断）。
- 在其他线程正在运行时，线程可以暂时搁置（也称为睡眠） -- 这就是线程的退让。

Python中使用线程有两种方式：函数或者用类来包装线程对象。

### 函数式

调用thread模块中的start_new_thread()函数来产生新线程。语法如下:

```
thread.start_new_thread ( function, args[, kwargs] )

```

- function - 线程函数。
- args - 传递给线程函数的参数,他必须是个tuple类型。
- kwargs - 可选参数。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import thread
import time
 
# 为线程定义一个函数
def print_time( threadName, delay):
   count = 0
   while count < 5:
      time.sleep(delay)
      count += 1
      print "%s: %s" % ( threadName, time.ctime(time.time()) )
 
# 创建两个线程
try:
   thread.start_new_thread( print_time, ("Thread-1", 2, ) )
   thread.start_new_thread( print_time, ("Thread-2", 4, ) )
except:
   print "Error: unable to start thread"
 
while 1:
   pass
```

### 模块

使用Threading模块创建线程，直接从threading.Thread继承，然后重写__init__方法和run方法：

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import threading
import time
 
exitFlag = 0
 
class myThread (threading.Thread):   #继承父类threading.Thread
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):                   #把要执行的代码写到run函数里面 线程在创建后会直接运行run函数 
        print "Starting " + self.name
        print_time(self.name, self.counter, 5)
        print "Exiting " + self.name
 
def print_time(threadName, delay, counter):
    while counter:
        if exitFlag:
            (threading.Thread).exit()
        time.sleep(delay)
        print "%s: %s" % (threadName, time.ctime(time.time()))
        counter -= 1
 
# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)
 
# 开启线程
thread1.start()
thread2.start()
 
print "Exiting Main Thread"
```

### 线程同步

如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数据的正确性，需要对多个线程进行同步。

使用Thread对象的Lock和Rlock可以实现简单的线程同步，这两个对象都有acquire方法和release方法，对于那些需要每次只允许一个线程操作的数据，可以将其操作放到acquire和release方法之间。如下：

多线程的优势在于可以同时运行多个任务（至少感觉起来是这样）。但是当线程需要共享数据时，可能存在数据不同步的问题。

考虑这样一种情况：一个列表里所有元素都是0，线程"set"从后向前把所有元素改成1，而线程"print"负责从前往后读取列表并打印。

那么，可能线程"set"开始改的时候，线程"print"便来打印列表了，输出就成了一半0一半1，这就是数据的不同步。为了避免这种情况，引入了锁的概念。

锁有两种状态——锁定和未锁定。每当一个线程比如"set"要访问共享数据时，必须先获得锁定；如果已经有别的线程比如"print"获得锁定了，那么就让线程"set"暂停，也就是同步阻塞；等到线程"print"访问完毕，释放锁以后，再让线程"set"继续。

经过这样的处理，打印列表时要么全部输出0，要么全部输出1，不会再出现一半0一半1的尴尬场面。

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import threading
import time
 
class myThread (threading.Thread):
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):
        print "Starting " + self.name
       # 获得锁，成功获得锁定后返回True
       # 可选的timeout参数不填时将一直阻塞直到获得锁定
       # 否则超时后将返回False
        threadLock.acquire()
        print_time(self.name, self.counter, 3)
        # 释放锁
        threadLock.release()
 
def print_time(threadName, delay, counter):
    while counter:
        time.sleep(delay)
        print "%s: %s" % (threadName, time.ctime(time.time()))
        counter -= 1
 
threadLock = threading.Lock()
threads = []
 
# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)
 
# 开启新线程
thread1.start()
thread2.start()
 
# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)
 
# 等待所有线程完成
for t in threads:
    t.join()
print "Exiting Main Thread"
```

### 线程优先级队列（Queue）

Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。

Queue模块中的常用方法:



- Queue.qsize() 返回队列的大小
- Queue.empty() 如果队列为空，返回True,反之False
- Queue.full() 如果队列满了，返回True,反之False
- Queue.full 与 maxsize 大小对应
- Queue.get([block[, timeout]])获取队列，timeout等待时间
- Queue.get_nowait() 相当Queue.get(False)
- Queue.put(item, block=True, timeout=None) 写入队列，timeout等待时间
- Queue.put_nowait(item) 相当 Queue.put(item, False)
- Queue.task_done() 在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号
- Queue.join() 实际上意味着等到队列为空，再执行别的操作

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import Queue
import threading
import time
 
exitFlag = 0
 
class myThread (threading.Thread):
    def __init__(self, threadID, name, q):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.q = q
    def run(self):
        print "Starting " + self.name
        process_data(self.name, self.q)
        print "Exiting " + self.name
 
def process_data(threadName, q):
    while not exitFlag:
        queueLock.acquire()
        if not workQueue.empty():
            data = q.get()
            queueLock.release()
            print "%s processing %s" % (threadName, data)
        else:
            queueLock.release()
        time.sleep(1)
 
threadList = ["Thread-1", "Thread-2", "Thread-3"]
nameList = ["One", "Two", "Three", "Four", "Five"]
queueLock = threading.Lock()
workQueue = Queue.Queue(10)
threads = []
threadID = 1
 
# 创建新线程
for tName in threadList:
    thread = myThread(threadID, tName, workQueue)
    thread.start()
    threads.append(thread)
    threadID += 1
 
# 填充队列
queueLock.acquire()
for word in nameList:
    workQueue.put(word)
queueLock.release()
 
# 等待队列清空
while not workQueue.empty():
    pass
 
# 通知线程是时候退出
exitFlag = 1
 
# 等待所有线程完成
for t in threads:
    t.join()
print "Exiting Main Thread"
```

运行结果

```
Starting Thread-1
Starting Thread-2
Starting Thread-3
Thread-1 processing One
Thread-2 processing Two
Thread-3 processing Three
Thread-1 processing Four
Thread-2 processing Five
Exiting Thread-3
Exiting Thread-1
Exiting Thread-2
Exiting Main Thread
```

### 观察加锁和不加锁的差别

#### 加锁时的代码

```
# -*-* encoding:UTF-8 -*-
# author : shoushixiong
# date   : 2018/11/22
import threading
import time
list = [0,0,0,0,0,0,0,0,0,0,0,0]
class myThread(threading.Thread):
    def __init__(self,threadId,name,counter):
        threading.Thread.__init__(self)
        self.threadId = threadId
        self.name = name
        self.counter = counter
    def run(self):
        print "开始线程:",self.name
        # 获得锁，成功获得锁定后返回 True
        # 可选的timeout参数不填时将一直阻塞直到获得锁定
        # 否则超时后将返回 False
        threadLock.acquire()
        print_time(self.name,self.counter,list.__len__())
        # 释放锁
        threadLock.release()
    def __del__(self):
        print self.name,"线程结束！"
def print_time(threadName,delay,counter):
    while counter:
        time.sleep(delay)
        list[counter-1] += 1
        print "[%s] %s 修改第 %d 个值，修改后值为:%d" % (time.ctime(time.time()),threadName,counter,list[counter-1])
        counter -= 1
threadLock = threading.Lock()
threads = []
# 创建新线程
thread1 = myThread(1,"Thread-1",1)
thread2 = myThread(2,"Thread-2",2)
# 开启新线程
thread1.start()
thread2.start()
# 添加线程到线程列表
threads.append(thread1)
threads.append(thread2)
# 等待所有线程完成
for t in threads:
    t.join()
print "主进程结束！"
```

输出结果为

```
开始线程: Thread-1
开始线程: Thread-2
[Thu Nov 22 16:04:13 2018] Thread-1 修改第 12 个值，修改后值为:1
[Thu Nov 22 16:04:14 2018] Thread-1 修改第 11 个值，修改后值为:1
[Thu Nov 22 16:04:15 2018] Thread-1 修改第 10 个值，修改后值为:1
[Thu Nov 22 16:04:16 2018] Thread-1 修改第 9 个值，修改后值为:1
[Thu Nov 22 16:04:17 2018] Thread-1 修改第 8 个值，修改后值为:1
[Thu Nov 22 16:04:18 2018] Thread-1 修改第 7 个值，修改后值为:1
[Thu Nov 22 16:04:19 2018] Thread-1 修改第 6 个值，修改后值为:1
[Thu Nov 22 16:04:20 2018] Thread-1 修改第 5 个值，修改后值为:1
[Thu Nov 22 16:04:21 2018] Thread-1 修改第 4 个值，修改后值为:1
[Thu Nov 22 16:04:22 2018] Thread-1 修改第 3 个值，修改后值为:1
[Thu Nov 22 16:04:23 2018] Thread-1 修改第 2 个值，修改后值为:1
[Thu Nov 22 16:04:24 2018] Thread-1 修改第 1 个值，修改后值为:1
[Thu Nov 22 16:04:26 2018] Thread-2 修改第 12 个值，修改后值为:2
[Thu Nov 22 16:04:28 2018] Thread-2 修改第 11 个值，修改后值为:2
[Thu Nov 22 16:04:30 2018] Thread-2 修改第 10 个值，修改后值为:2
[Thu Nov 22 16:04:32 2018] Thread-2 修改第 9 个值，修改后值为:2
[Thu Nov 22 16:04:34 2018] Thread-2 修改第 8 个值，修改后值为:2
[Thu Nov 22 16:04:36 2018] Thread-2 修改第 7 个值，修改后值为:2
[Thu Nov 22 16:04:38 2018] Thread-2 修改第 6 个值，修改后值为:2
[Thu Nov 22 16:04:40 2018] Thread-2 修改第 5 个值，修改后值为:2
[Thu Nov 22 16:04:42 2018] Thread-2 修改第 4 个值，修改后值为:2
[Thu Nov 22 16:04:44 2018] Thread-2 修改第 3 个值，修改后值为:2
[Thu Nov 22 16:04:46 2018] Thread-2 修改第 2 个值，修改后值为:2
[Thu Nov 22 16:04:48 2018] Thread-2 修改第 1 个值，修改后值为:2
主进程结束！
Thread-1 线程结束！
Thread-2 线程结束！
```

  

#### 不加锁时

同样是上面实例的代码，注释以下两行代码：

```
threadLock.acquire()
threadLock.release()
```

输出结果为：

```
开始线程: Thread-1
开始线程: Thread-2
[Thu Nov 22 16:09:20 2018] Thread-1 修改第 12 个值，修改后值为:1
[Thu Nov 22 16:09:21 2018] Thread-2 修改第 12 个值，修改后值为:2
[Thu Nov 22 16:09:21 2018] Thread-1 修改第 11 个值，修改后值为:1
[Thu Nov 22 16:09:22 2018] Thread-1 修改第 10 个值，修改后值为:1
[Thu Nov 22 16:09:23 2018] Thread-1 修改第 9 个值，修改后值为:1
[Thu Nov 22 16:09:23 2018] Thread-2 修改第 11 个值，修改后值为:2
[Thu Nov 22 16:09:24 2018] Thread-1 修改第 8 个值，修改后值为:1
[Thu Nov 22 16:09:25 2018] Thread-2 修改第 10 个值，修改后值为:2
[Thu Nov 22 16:09:25 2018] Thread-1 修改第 7 个值，修改后值为:1
[Thu Nov 22 16:09:26 2018] Thread-1 修改第 6 个值，修改后值为:1
[Thu Nov 22 16:09:27 2018] Thread-2 修改第 9 个值，修改后值为:2
[Thu Nov 22 16:09:27 2018] Thread-1 修改第 5 个值，修改后值为:1
[Thu Nov 22 16:09:28 2018] Thread-1 修改第 4 个值，修改后值为:1
[Thu Nov 22 16:09:29 2018] Thread-2 修改第 8 个值，修改后值为:2
[Thu Nov 22 16:09:29 2018] Thread-1 修改第 3 个值，修改后值为:1
[Thu Nov 22 16:09:30 2018] Thread-1 修改第 2 个值，修改后值为:1
[Thu Nov 22 16:09:31 2018] Thread-2 修改第 7 个值，修改后值为:2
[Thu Nov 22 16:09:31 2018] Thread-1 修改第 1 个值，修改后值为:1
[Thu Nov 22 16:09:33 2018] Thread-2 修改第 6 个值，修改后值为:2
[Thu Nov 22 16:09:35 2018] Thread-2 修改第 5 个值，修改后值为:2
[Thu Nov 22 16:09:37 2018] Thread-2 修改第 4 个值，修改后值为:2
[Thu Nov 22 16:09:39 2018] Thread-2 修改第 3 个值，修改后值为:2
[Thu Nov 22 16:09:41 2018] Thread-2 修改第 2 个值，修改后值为:2
[Thu Nov 22 16:09:43 2018] Thread-2 修改第 1 个值，修改后值为:2
主进程结束！
Thread-1 线程结束！
Thread-2 线程结束！
```

# selenium

## 安装

### 安装selenium

```
conda install selenium
```

### 下载浏览器驱动

当selenium升级到3.0之后，对不同的浏览器驱动进行了规范。如果想使用selenium驱动不同的浏览器，必须单独下载并设置不同的浏览器驱动。

各浏览器下载地址：

Firefox浏览器驱动：[geckodriver](https://link.zhihu.com/?target=https%3A//github.com/mozilla/geckodriver/releases)

Chrome浏览器驱动：[chromedriver](https://link.zhihu.com/?target=https%3A//sites.google.com/a/chromium.org/chromedriver/home) , [taobao备用地址](https://link.zhihu.com/?target=https%3A//npm.taobao.org/mirrors/chromedriver)

IE浏览器驱动：[IEDriverServer](https://link.zhihu.com/?target=http%3A//selenium-release.storage.googleapis.com/index.html)

Edge浏览器驱动：[MicrosoftWebDriver](https://link.zhihu.com/?target=https%3A//developer.microsoft.com/en-us/microsoft-edge/tools/webdriver)

Opera浏览器驱动：[operadriver](https://link.zhihu.com/?target=https%3A//github.com/operasoftware/operachromiumdriver/releases)

PhantomJS浏览器驱动：[phantomjs](https://link.zhihu.com/?target=http%3A//phantomjs.org/)

注：部分浏览器驱动地址需要科学上网。

#### 下载chromedriver

1.打开chrome 输入 “chrome://version/”来查看chrome版本 

2.访问驱动下载地址，查找相应版本的chromedriver

3.解压获得chromedriver.exe

##### 添加驱动的方法1

1、将下载的chromedriver.exe（2.46）放到（复制或移动）至Python的安装目录下，与python.exe文件相同目录下。查看Python的安装目录（cmd输入命令where python）。测试代码如下

```
from selenium import webdriver
import time

driver = webdriver.Chrome()
driver.get('https://www.baidu.com/')
print(driver.title)
time.sleep(5)
driver.quit()
```

##### 添加驱动的方法2

设置浏览器驱动
设置浏览器的地址非常简单。 我们可以手动创建一个存放浏览器驱动的目录，如： C:\driver , 将下载的浏览器驱动文件（例如：chromedriver、geckodriver）丢到该目录下。

我的电脑–>属性–>系统设置–>高级–>环境变量–>系统变量–>Path，将“C:\driver”目录添加到Path的值中。

<<<<<<< HEAD
## 交互操作

|             |                  |
| ----------- | ---------------- |
| clear()     | 清除内容         |
| click()     | 模拟鼠标点击操作 |
| send_keys() | 向输入框输入     |
| submit()    | 提交表单         |
|             |                  |

鼠标事件

ActionChains可以完成简单的交互行为，例如鼠标移动，鼠标点击事件，键盘输入，以及内容菜单交互。

|                           |                |
| ------------------------- | -------------- |
| context_click()           | 鼠标右击       |
| double_click()            | 鼠标双击       |
| move_to_element(目标元素) | 移动到某个元素 |
|                           |                |
|                           |                |
|                           |                |
|                           |                |
|                           |                |
|                           |                |



|                             |        |
| --------------------------- | ------ |
| send_keys(Keys.ENTER)       | 回车键 |
| send_keys(Keys.BACK_SPACE)  | 删除键 |
| send_keys(Keys.SPACE)       | 空格键 |
| send_keys(Keys.CONTROL,‘a’) | 全选   |
| send_keys(Keys.CONTROL,‘c’) | 复制   |
| send_keys(Keys.CONTROL,‘v’) | 粘贴   |
|                             |        |
|                             |        |
|                             |        |



=======
>>>>>>> e94ab2aa461f1372fa11366cff1dfac678b18db7
## 简单的示例及分析

### 代码 

先看这段代码

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

driver = webdriver.Chrome()
driver.get("http://www.python.org")
assert "Python" in driver.title
elem = driver.find_element_by_name("q")
elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
assert "No results found." not in driver.page_source
driver.close()
```

### 分析

`selenium.webdriver` 模块提供了所有 WebDriver的实现，现在支持的WebDriver的实现有 Firefox,Ie,Chrome,Remote,`Keys`类提供了键盘的代码（回车,ALT,F1等等）

```
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
```

先创建一个Firefox实例

```
 driver = webdriver.Firefox()
```

`driver.get`方法会导向给定的URL的页面，WebDriver会等待页面完全加载完(就是`onload`函数被触发了),才把程序的控制权交给你的测试或者脚本。 但是如果你的页面用了太多的AJAX，那么这个机制就没什么卵用了，因为它不知道页面到底是什么时候加载完。

```
driver.get("http://www.python.org")
```

WebDrive提供了一系统类似于`find_element_by_*`的方法来寻找页面元素，例如，我们利用`find_element_by_name`方法，通过元素的`name`属性来定位一个文本输入框元素。 

```
elem = driver.find_element_by_name("q")
```

接着我们发送了一些字符，类似于用键盘直接输入。特殊的键盘符我们可以导入`selenium.webdriver.common.keys`,然后用`Keys`类来表示:

```
elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
```

提交页面之后我们应该确认一下是否有返回，为了确定有东西返回，我们在这里下一个断言:

```
assert "No results found." not in driver.page_source
```

最后浏览器窗口被关闭了，你也可以调用`quit`方法来代替`close`，区别在于`quit`会退出整个浏览器，而`close`只会关闭一个标签，但是如果浏览器只有一个标签，那么这两个方法完全一样，都会关闭整个浏览器。

```
  driver.close()
```

### unittest代码实例

Selenium经常被用来写测试用例，它本身的包不提供测试的工具或者框架。我们可以用Python的单元测试模块来编写测试用例。下面是一个用`unittest`模块改进后的例子，是对 `python.org` 函数搜索功能的测试:

```
import unittest
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

class PythonOrgSearch(unittest.TestCase):

    def setUp(self):
        self.driver = webdriver.Chrome()

    def test_search_in_python_org(self):
        driver = self.driver
        driver.get("http://www.python.org")
        self.assertIn("Python", driver.title)
        elem = driver.find_element_by_name("q")
        elem.send_keys("pycon")
        elem.send_keys(Keys.RETURN)
        assert "No results found." not in driver.page_source


    def tearDown(self):
        self.driver.close()

if __name__ == "__main__":
    unittest.main()
```

测试用例是继承了 unittest.TestCase 类，继承这个类表明这是一个测试类。setUp 方法是初始化的方法，这个方法会在每个测试类中自动调用。每一个测试方法命名都有规范，必须以 test 开头，会自动执行。最后的 tearDown 方法会在每一个测试方法结束之后调用。这相当于最后的析构方法。在这个方法里写的是 close 方法，你还可以写 quit 方法。不过 close 方法相当于关闭了这个 TAB 选项卡，然而 quit 是退出了整个浏览器。当你只开启了一个 TAB 选项卡的时候，关闭的时候也会将整个浏览器关闭。





测试用例类继承了`unittest.TestCase`类，这表明这个类是一个测试用例：

```
class PythonOrgSearch(unittest.TestCase):
```

`setUp`函数进行了初始化，你将要在这个类里编写的所有测试方法都要先调用这个方法，接着我们创建了一个`Firefox WebDriver`实例：

```
def setUp(self):
    self.driver = webdriver.Chrome()
```

接下来是测试用例的方法，它应该总是以字符'test'开始. 方法的第一行 本地引用了 `setUp`方法中创建的driver对象：

```
def test_search_in_python_org(self):
    driver = self.driver
```

`driver.get`方法会导向给定的URL的页面，WebDriver会等待页面完全加载完(就是`onload`函数被触发了),才把程序的控制权交给你的测试或者脚本。 但是如果你的页面用了太多的AJAX，那么这个机制就没什么卵用了，因为它不知道页面到底是什么时候加载完。

```
 driver.get("http://www.python.org")
```

下一行是个断言，确认页面标题里是否有'Python'这个单词:

```
self.assertIn("Python", driver.title)
```

WebDrive提供了一系统类似于`find_element_by_*`的方法来寻找页面元素，例如，我们利用`find_element_by_name`方法，通过元素的`name`属性来定位一个文本输入框元素。 

```
  elem = driver.find_element_by_name("q")
```

接着我们发送了一些字符，类似于用键盘直接输入。特殊的键盘符我们可以导入`selenium.webdriver.common.keys`,然后用`Keys`类来表示:

```

elem.send_keys("pycon")
elem.send_keys(Keys.RETURN)
```

提交页面之后我们应该确认一下是否有返回，为了确定有东西返回，我们在这里下一个断言:

```
  assert "No results found." not in driver.page_source
```

最后浏览器窗口被关闭了，你也可以调用`quit`方法来代替`close`，区别在于`quit`会退出整个浏览器，而`close`只会关闭一个标签，但是如果浏览器只有一个标签，那么这两个方法完全一样，都会关闭整个浏览器。

```
 driver.close()
```

Final lines are some boiler plate code to run the test suite:

```
if __name__ == "__main__":
    unittest.main()
```

## 等待

## 均线

很简单，举个例子，比如最短的一个叫 5 日均线，就是把最近 5 天的收盘价加起来除以 5，得到一个点，然后把这些点依次连接，就变成一条线。

所以均线是股票收盘价的移动平均线。以此类推，我们把 5 日换成 10 日、30 日、60 日、120 日……就能得到不同天数的均线。

按天数的多少，均线分别可以反映股价短期（5 日、10 日）、中期（30 日、60 日）、长期（120 日以上）的趋势。

### 均线的两种主要用法

1.看股价趋势

当短中长均线全部向下移动，说明大家卖出意愿比较强，看空的人占上风。一旦短期均线开始向上移动，意味着看多的人逐渐占据上风，随着短期均线上穿中长期均线，最终股价形成上涨趋势。

举个例子，图中的红色 5 日均线开始向上移动，然后穿过蓝色 30 日均线，形成「金叉」，这说明人们开始看好这支股票，通常这也是一个很好的买入时机。

相反，当短中长均线全部向上移动，说明大家买入意愿比较强，看多的人占上风。一旦短期均线开始向下移动，意味着短期看空的人逐渐占据上风，随着短期均线下穿中长期均线，最终股价形成下跌趋势。

比如我们举个例子，当图中的相对短期的红色 5 日均线，向下穿过相对长期的蓝色 30 日均线，形成「死叉」，说明大家都不看好这支股票，股价也逐步向下运行，此时不但不应该买入，持有股票的人也要择机抛出。



2.看阻力和支撑

均线的第二层含义就是，它代表了市场上买入股票者的平均成本，比如 5 日均线代表了市场上 5 天内买入者的平均成本。均线所在的点位往往是十分重要的支撑或阻力位。

因为股价在下跌过程中，遇到市场平均成本，会引起投资者心理的变化，进一步影响到大家的行为。

比如 250 日均线代表了一年内买入者平均成本。当股价跌破 250 日均线，意味着股价低于一年内买入者的平均成本。相对便宜的价格，会吸引大量投资者进场购买股票，股价因此得到支撑。

举个例子，图中的股票价格从最高的 32.64 元处暴跌，在触及绿色的 250 日线后，开始止跌回升

最后同样要补充一点，均线和 K 线一样，也包含不同周期。假如我们分析的是周 K 线图，那么原来的 10 日、30 日均线，当然也就变成了 10 周、30 周均线。

相对日均线，周均线代表了一个更大级别的趋势，这两个指标同样可以配合使用。比如日均线和周均线同时发出信号，信号可靠性大大增加，买卖的准确率就比较高。



现在很多Web应用都在使用AJAX技术。浏览器载入一个页面时，页面内的元素可能是在不同的时间载入的，这会加大定位元素的困难程度，因为元素不在DOM里，会抛出`ElementNotVisibleException`异常，使用`waits`，我们就可以解决这个问题。Waiting给(页面)动作的执行提供了一些时间间隔-通常是元素定位或者其他对元素的操作。

Selenium WebDriver提供了两类`waits`- 隐式和显式。显式的`waits`会让WebDriver在更深一步的执行前等待一个确定的条件触发，隐式的`waits`则会让WebDriver试图定位元素的时候对DOM进行指定次数的轮询。

### 显式Waits

显式的`waits`等待一个确定的条件触发然后才进行更深一步的执行。最糟糕的的做法是`time.sleep()`，这指定的条件是等待一个指定的时间段。 这里提供一些便利的方法让你编写的代码只等待需要的时间，`WebDriverWait`结合`ExpectedCondition`是一种实现的方法：

```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("http://somedomain/url_that_delay_loading")
try:
    element = WebDriverWait(driver,10).until(
        EC.presence_of_element_located((By.ID,"myDynamicElement"))
    )
finally:
    driver.quit()
```

这段代码会等待10秒，如果10秒内找到元素则立即返回，否则会抛出`TimeoutException`异常，WebDriverWait默认每500毫秒调用一下`ExpectedCondition`直到它返回成功为止。`ExpectedCondition`类型是布尔的，成功的返回值就是true,其他类型的`ExpectedCondition`成功的返回值就是 `not null`

### 隐式Waits

当我们要找一个或者一些不能立即可用的元素的时候，隐式`waits`会告诉WebDriver轮询DOM指定的次数，默认设置是0次。一旦设定，WebDriver对象实例的整个生命周期的隐式调用也就设定好了。

```
from selenium import webdriver

driver = webdriver.Chrome()
driver.implicitly_wait(10) # seconds
driver.get("http://somedomain/url_that_delays_loading")
myDynamicElement = driver.find_element_by_id('myDynamicElement')
```

## 元素选择



### 动态定位

**xpath使用**

contains(a,b) 如果 a 中含有字符串 b，则返回 True,反之 False

starts-with(a,b) 如果 a 中以字符串 b 开头，则返回 True,反之 False

ends-with(a,b) 如果 a 中以字符串 b 结尾，则返回 True,反之 False

如：<div  id = "but_1234">aaaaaa.<div>

driver.find_element_by_xpath("//div[contains(@id,"but")]")

driver.find_element_by_xpath("//div[starts-with(@id,"but")]")



### 根据文本内容定位

我们会经常遇到这样的标题中带着文字，而这个文字内容又是唯一的，那么我们为什么不直接根据文字定位呢，有网友告诉我说直接定位文字经常会定位不到，我不知道为什么会这样，但确实会遇到定位不到，（有知道原因的欢迎留言，或者有其他更好方法的）不过不防先试试这种定位方法

```


xpath

driver.findElement(By.xpath("//span[contains(text(),'hello')]"))  包含匹配

driver.findElement(By.xpath("//span[text()='新闻']"))     绝对匹配

```



### 由父节点定位子节点

对以下代码：

```
<html>
<body>
<div id="A">
<!--父节点定位子节点-->
<div id="B">
<div>parent to child</div>
</div>
</div>
</body>
</html>
```

想要根据 B节点 定位无id的子节点，代码示例如下：

```
# -*- coding: utf-8 -*-
from selenium import webdriver
driver = webdriver.Firefox()
driver.get('D:\\py\\AutoTestFramework\\src\\others\\test.html')
# 1.串联寻找
print driver.find_element_by_id('B').find_element_by_tag_name('div').text
# 2.xpath父子关系寻找
print driver.find_element_by_xpath("//div[@id='B']/div").text
# 3.css selector父子关系寻找
print driver.find_element_by_css_selector('div#B>div').text
# 4.css selector nth-child
print driver.find_element_by_css_selector('div#B div:nth-child(1)').text
# 5.css selector nth-of-type
print driver.find_element_by_css_selector('div#B div:nth-of-type(1)').text
# 6.xpath轴 child
print driver.find_element_by_xpath("//div[@id='B']/child::div").text
driver.quit()
```

### 定位兄弟节点

页面代码：

```
<html>
<body>
<div id="A">
<!--下面两个节点用于兄弟节点定位-->
<div>brother 1</div>
<div id="D"></div>
<div>brother 2</div>
</div>
</body>
</html>
```

怎么通过 D节点 定位其哥哥节点呢？看代码示例：

```python
# -*- coding: utf-8 -*-
from selenium import webdriver
driver = webdriver.Firefox()
driver.get('D:\\Code\\py\\AutoTestFramework\\src\\others\\test.html')
# 1.xpath,通过父节点获取其哥哥节点
print driver.find_element_by_xpath("//div[@id='D']/../div[1]").text
# 2.xpath轴 preceding-sibling
print driver.find_element_by_xpath("//div[@id='D']/preceding-sibling::div[1]").text
driver.quit()
```

源码与 3 一致，要想通过 D节点 定位其弟弟节点，看代码示例：

```python
# -*- coding: utf-8 -*-
from selenium import webdriver
driver = webdriver.Firefox()
driver.get('D:\\Code\\py\\AutoTestFramework\\src\\others\\test.html')
# 1.xpath，通过父节点获取其弟弟节点
print driver.find_element_by_xpath("//div[@id='D']/../div[3]").text
# 2.xpath轴 following-sibling
print driver.find_element_by_xpath("//div[@id='D']/following-sibling::div[1]").text
# 3.xpath轴 following
print driver.find_element_by_xpath("//div[@id='D']/following::*").text
# 4.css selector +
print driver.find_element_by_css_selector('div#D + div').text
# 5.css selector ~
print driver.find_element_by_css_selector('div#D ~ div').text
driver.quit()
```



## 页面对象

```
import unittest
from selenium import webdriver
import page

class PythonOrgSearch(unittest.TestCase):
    """一个简单展示页面对象如何工作的类"""

    def setUp(self):
        self.driver = webdriver.Chrome()
        self.driver.get("http://www.python.org")

    def test_search_in_python_org(self):
        """
        测试 python.org网站的搜索功能。搜索一个单词“pycon”然后验证某些结果会展示出来。
        注意这个测试不会在搜索结果页里寻找任何细节文本，它只会验证结果为非空
        """

        #载入主页面，这个例子里是 Python.org的首页
        main_page = page.MainPage(self.driver)
        #检查页面的标题是否包含"python"单词
        assert main_page.is_title_matches(), "python.org title doesn't match."
        #将搜索框的文本设置为"pycon"
        main_page.search_text_element = "pycon"
        main_page.click_go_button()
        search_results_page = page.SearchResultsPage(self.driver)
        #验证结果页非空
            assert search_results_page.is_results_found(), "No results found."

    def tearDown(self):
        self.driver.close()

if __name__ == "__main__":
    unittest.main()
```

https://python-selenium-zh.readthedocs.io/zh_CN/latest/6.%E9%A1%B5%E9%9D%A2%E5%AF%B9%E8%B1%A1/





## 一些常用函数

### 断言

```
 assertIn(key, container, message)
```

**参数**：assertIn()接受以下三个参数的说明：

- **key**：在给定容器中检查其存在性的字符串
- **container**：在其中搜索关键字符串的字符串
- **message**：作为测试消息失败时显示的消息的字符串语句。

## 一些常见问题

### sendkey没法输入中文

\#方式1，在中文前加入u
\# driver.find_element_by_class_name(“s_ipt”).send_keys(u’测试’)

### 跳转后获取当前页面

```
time.sleep(1)
driver.switch_to.windows(driver.window_handles[0])
```

方式二

```
time.sleep(1)
search_windows= driver.current_windows_handle
```



# AIRTEST

### 调试模式

```
*#558#*
```

## 安装相关

### 安装poco

先安装一个示例APP

http://top.gdl.netease.com/poco-res/poco-demo-unity-game-android.zip

```
adb install c:/NetScramble_1.1.apk 

```

## 非AirtestIDE中安装

### 环境安装

```
# 安装Airtest框架
pip install airtest

# 安装Poco框架；编写了Poco语句就需要安装
pip install pocoui

# 安装airtest-selenium框架；编写了airtest-selenium语句就需要安装
pip install airtest-selenium
```



### 设备连接

```
# 在auto_setup接口传入devices参数
auto_setup(__file__,logdir=True,devices=["android://127.0.0.1:5037/127.0.0.1:7555"])
```

##### **1.连接Android手机**

```
# 什么都不填写，默认取当前连接中的第一台手机
Android:///
# 连接本机默认端口连的一台设备号为79d03fa的手机
Android://127.0.0.1:5037/79d03fa
# 用本机的adb连接一台adb connect过的远程设备，注意10.254.60.1:5555其实是serialno
Android://127.0.0.1:5037/10.254.60.1:5555
```



https://mp.weixin.qq.com/s/znYi-eCifeMXfce9GDpW-w

### log保存

```
# 在auto_setup接口里设置logdir，用于保存log内容
# logdir可以传入具体的log保存路径，或者是True，传入True表示在当前项目目录下生成log内容

auto_setup(__file__,logdir=True)
```

<<<<<<< HEAD
## 常用操作

### ADB

除了一些封装的ADB语句

还可以通过shell()方法补充ADB shell

```
android = Android()
shell("ls")
shell("pm list packages -3")
```

元素匹配

```
#基本选择器
poco("star_single",type="Image")
#相对选择器
poco("plays").child("playBasic").offspring("star_single")
#空间顺序选择器
name0 = poco("Content").child(type="Text")[0].get_name()
name1 = poco("Content").child(type="Text")[1].get_name()
name2 = poco("Content").child(type="Text")[2].get_name()

```



### 基础操作

|                           |                                                              |      |
| ------------------------- | ------------------------------------------------------------ | ---- |
| start_app()               | 启动APP                                                      |      |
| stop_app()                | 关闭APP                                                      |      |
| click()                   | 单击                                                         |      |
| rclick()                  | 右键点击                                                     |      |
| double_click()            | 双击                                                         |      |
| long_click()              | 长按                                                         |      |
| text()                    | 输入内容                                                     |      |
| wait                      | 等待，poco(‘控件地址’).wait(2).click()，控件出现就点击，最多等待2秒 |      |
| sleep                     | 等待，不进行操作                                             |      |
| swipe                     | 滑动，swipe([0.2, -0.2], duration=1)以45度角滑动，持续1秒    |      |
| drag                      | 拖拽，poco(text=‘需要拖动位置’).drag_to(poco(text=‘目标位置’)) |      |
| focus (local positioning) | 局部定位，poco(‘控件地址’).focus(‘center’).click()，点击控件中间位置 |      |
| attr(‘text’)              | 通过给定的属性名检索ui元素的属性。如果属性不存在，则返回none（属性有visible、text、type、pos、name等） |      |
| snopshoot                 | 截图，所截内容可在报告中展示                                 |      |
| wait_for_appearance       | 等待元素出现                                                 |      |
| wait_for_disappearance    | 等待元素消失                                                 |      |
|                           |                                                              |      |

### 断言

|                     |                              |      |
| ------------------- | ---------------------------- | ---- |
| assert              | 判断内容                     |      |
| assert_exists()     | 判断元素是否存在             |      |
| assert_not_exists() | 判断元素是否不存在           |      |
| assert_equal()      | 判断实际内容与预期是否一致   |      |
| assert_not_equal()  | 判断实际内容与预期是否不一致 |      |

### 录屏操作

```
adb = ADB(serialno="emulator-5554")
recorder = Recorder(adb)

# 开启录屏
recorder.start_recording(max_time=10)

# 结束录屏
recorder.stop_recording(output="test.mp4")
```

### 输出报告

```
simple_report(__file__,filepath,logpath,logfile,output)
```

=======
>>>>>>> e94ab2aa461f1372fa11366cff1dfac678b18db7


## 示例

### 随手写的计算器按键测试

```python
# -*- encoding=utf8 -*-
__author__ = "11146243"
__title__="计算器"
__desc__ = "ES_0001_数字按键显示"
# 从api中import接口后，就能直接使用airtest的各个接口了
from airtest.core.api import *
# 引入androidPoco方法
from poco.drivers.android.uiautomation import AndroidUiautomationPoco
poco = AndroidUiautomationPoco(use_airtest_input=True, screenshot_each_action=False)
# 引用自定义的公共方法
#using("../../common/common.air")
#from common import *
# 初始化设备
auto_setup(__file__)


# 自定义变量
calculator="com.android.bbkcalculator"


# 自定义方法
def AppStart():
    # 启动APP
    stop_app(calculator)
    time.sleep(2.0)
    start_app(calculator)
    time.sleep(5.0)
AppStart()


#清除之前的数字
poco("com.android.bbkcalculator:id/clr").click()
assert poco("com.android.bbkcalculator:id/formula").get_text() ==None
#遍历数字
for i in range(10):
    #点击数字键
    poco("com.android.bbkcalculator:id/digit_"+str(i)).click()
    #查看是否获得相同结果
    assert poco("com.android.bbkcalculator:id/formula").get_text() ==str(i)
    time.sleep(1.0)
    #点击清除
    poco("com.android.bbkcalculator:id/clr").click()
    assert poco("com.android.bbkcalculator:id/formula").get_text() ==None
    time.sleep(1.0)

```

## 元素定位

基本选择器

```
poco("star_single",type="Image")
```

相对选择器

```
poco("plays").child("playBasic").offspring("star_single")
```

offspring:寻找所有后代节点中有用的

sibling:获取当前节点的兄弟节点

parent:父节点

遍历

```python
items=poco("com.netease.cloudmusic:id/musicInfoList").child("com.netease.cloudmusic:id/musicListItemContainer")
for item in items:
    print_obj=item.offspring("com.netease.cloudmusic:id/songRank").get_text()+": "+ \
    item.offspring("com.netease.cloudmusipoco("<Root>")c:id/songName").get_text()
    print(print_obj)
```



## 打包

pyinstaller

-F, –onefile	打包一个单个文件，如果你的代码都写在一个.py文件的话，可以用这个，如果是多个.py文件就别用
-D, –onedir	打包多个文件，在dist中生成很多依赖文件，适合以框架形式编写工具代码，我个人比较推荐这样，代码易于维护
-K, –tk	在部署时包含 TCL/TK
-a, –ascii	不包含编码.在支持Unicode的python版本上默认包含所有的编码.
-d, –debug	产生debug版本的可执行文件
-w,–windowed,–noconsole	使用Windows子系统执行.当程序启动的时候不会打开命令行(只对Windows有效)
-c,–nowindowed,–console	
使用控制台子系统执行(默认)(只对Windows有效)

pyinstaller -c  xxxx.py

pyinstaller xxxx.py --console

-s,–strip	可执行文件和共享库将run through strip.注意Cygwin的strip往往使普通的win32 Dll无法使用.
-X, –upx	如果有UPX安装(执行Configure.py时检测),会压缩执行文件(Windows系统中的DLL也会)(参见note)
-o DIR, –out=DIR	指定spec文件的生成目录,如果没有指定,而且当前目录是PyInstaller的根目录,会自动创建一个用于输出(spec和生成的可执行文件)的目录.如果没有指定,而当前目录不是PyInstaller的根目录,则会输出到当前的目录下.
-p DIR, –path=DIR	设置导入路径(和使用PYTHONPATH效果相似).可以用路径分割符(Windows使用分号,Linux使用冒号)分割,指定多个目录.也可以使用多个-p参数来设置多个导入路径，让pyinstaller自己去找程序需要的资源
–icon=<FILE.ICO>	
将file.ico添加为可执行文件的资源(只对Windows系统有效)，改变程序的图标  pyinstaller -i  ico路径 xxxxx.py

–icon=<FILE.EXE,N>	将file.exe的第n个图标添加为可执行文件的资源(只对Windows系统有效)
-v FILE, –version=FILE	将verfile作为可执行文件的版本资源(只对Windows系统有效)
-n NAME, –name=NAME	可选的项目(产生的spec的)名字.如果省略,第一个脚本的主文件名将作为spec的名字

打包poco和airtest的情况下，发现会找不到adb.exe，推测是pyinstaller打包会只打包py文件，而airtest需要调用adb.exe，所以需要想办法将adb.exe加入到打包路径

解决方法一：（成功）

```
打包技巧:(必须做这步不然打包后是运行不了的)
第一步: 找到你的pyinstaller库的目录的hooks
D:\Program Files (x86)\Python38\Lib\site-packages\PyInstaller
第二步:
下载我搞好的文件:https://www.lanzoui.com/itK5jj4zvhg
解压后,把里面的两个文件放入hooks文件夹中
如果你之前有打包过,那请把你的项目文件夹中的
build/dist文件夹都删掉
还有 xxx.spec文件也删掉,
然后命令行打包
打包代码: pyinstaller -F main.py
————————————————
版权声明：本文为CSDN博主「小主早安」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/xiaoxiamimm/article/details/114260224
```

然后还需要设置logdir



方法二：未测试

```
pyinstaller -F -p path main.py
```

此处path是第三方库的路径

## 常见问题

### 获取不到UI树

猜测：会沿用上一次dump出的页面树节点的缓存进行查找，目前在打开程序前先杀死程序再打开暂时未出现问题

```
from airtest.core.android.android import *
android = Android()
calculator = "com.android.bbkcalculator"
android.shell("am force-stop "+calculator)
```

结果后来发现好像不太行，然后再去研究有没有方法重启，发现AndroidUiautomationPoco中可以设置force_restart

whether always restart the poco-service-demo running on Android device or not

*class*`AndroidUiautomationPoco`**(***device=None***,** *using_proxy=True***,** *force_restart=False***,** *use_airtest_input=False***,** ***options***)**[[source\]](https://poco.readthedocs.io/en/latest/_modules/poco/drivers/android/uiautomation.html#AndroidUiautomationPoco)

- **device** (`Device`) – `airtest.core.device.Device` instance provided by `airtest`. leave the parameter default and the default device will be chosen. more details refer to `airtest doc`
- **using_proxy** (`bool`) – whether use adb forward to connect the Android device or not
- **force_restart** (`bool`) – whether always restart the poco-service-demo running on Android device or not
- **options** – see [`poco.pocofw.Poco`](https://poco.readthedocs.io/en/latest/source/poco.pocofw.html#poco.pocofw.Poco)

**设置force_restart=True,重启poco_service进行读取**

https://poco.readthedocs.io/en/latest/source/poco.drivers.android.uiautomation.html?highlight=AndroidUiautomationPoco#poco.drivers.android.uiautomation.AndroidUiautomationPoco

# ADB

查看当前包名

```
adb shell dumpsys window | findstr mCurrentFocus
```

# Django

## Mysql安装

https://www.runoob.com/w3cnote/windows10-mysql-installer.html

## 安装

新建project后结构如下

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

mysite/是一个容器，可以改名，不影响

**manage.py** 连接到Django project

第二个mysite是python package的名字

**mysite/__init__.py** 空文件，启动的py文件

**mysite/settings.py**  Settings/configuration for this Django project

**mysite/urls.py** The URL declarations for this Django project;

**mysite/asgi.py**  An entry-point for ASGI-compatible web servers to serve your project

**mysite/wsgi.py**  An entry-point for WSGI-compatible web servers to serve your project.

## 创建第一个APP

```
python manage.py startapp polls
```

### 创建第一个view

在view中

```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

然后设置路由

在polls下面创建urls.py文件

```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

然后回到project目录下的urls.py中添加include()

```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

The [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) function allows referencing other URLconfs. Whenever Django encounters [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

The idea behind [`include()`](https://docs.djangoproject.com/en/4.0/ref/urls/#django.urls.include) is to make it easy to plug-and-play URLs. Since polls are in their own URLconf (`polls/urls.py`), they can be placed under “/polls/”, or under “/fun_polls/”, or under “/content/polls/”, or any other path root, and the app will still work.

为了更好的理解，举例如下：

```
# app下的urls.py
#----------------------------------------------------
from django.urls import path
from . import views

urlpatterns = [
    path('/record', views.record_name)
]

# pro下的urls.py
#----------------------------------------------------
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('record/', include('record.urls')),  # 转向record应用的urls.py
    path('blog/', include('blog.urls')),  # 转向blog应用的urls.py
]
```

注：
 当访问的URL第一层为/record/xxxx时，为跳转到record应用下的urls.py文件下路由
 当访问的URL第一层为/blog/xxxx时，为跳转到blog应用下的urls.py文件下路由

### 数据库配置



### 创建模型

在这个投票应用中，需要创建两个模型：问题 `Question` 和选项 `Choice`。`Question` 模型包括问题描述和发布时间。`Choice` 模型有两个字段，选项描述和当前得票数。每个选项属于一个问题。

这些概念可以通过一个 Python 类来描述。按照下面的例子来编辑 `polls/models.py` 文件：

```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

每个模型被表示为 [`django.db.models.Model`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/instances/#django.db.models.Model) 类的子类。每个模型有许多类变量，它们都表示模型里的一个数据库字段。

每个字段都是 [`Field`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.Field) 类的实例 - 比如，字符字段被表示为 [`CharField`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.CharField) ，日期时间字段被表示为 [`DateTimeField`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.DateTimeField) 。这将告诉 Django 每个字段要处理的数据类型。

每个 [`Field`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.Field) 类实例变量的名字（例如 `question_text` 或 `pub_date` ）也是字段名，所以最好使用对机器友好的格式。你将会在 Python 代码里使用它们，而数据库会将它们作为列名。

你可以使用可选的选项来为 [`Field`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.Field) 定义一个人类可读的名字。这个功能在很多 Django 内部组成部分中都被使用了，而且作为文档的一部分。如果某个字段没有提供此名称，Django 将会使用对机器友好的名称，也就是变量名。在上面的例子中，我们只为 `Question.pub_date` 定义了对人类友好的名字。对于模型内的其它字段，它们的机器友好名也会被作为人类友好名使用。

定义某些 [`Field`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.Field) 类实例需要参数。例如 [`CharField`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.CharField) 需要一个 [`max_length`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.CharField.max_length) 参数。这个参数的用处不止于用来定义数据库结构，也用于验证数据，我们稍后将会看到这方面的内容。

[`Field`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.Field) 也能够接收多个可选参数；在上面的例子中：我们将 `votes` 的 [`default`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.Field.default) 也就是默认值，设为0。

注意在最后，我们使用 [`ForeignKey`](https://docs.djangoproject.com/zh-hans/4.0/ref/models/fields/#django.db.models.ForeignKey) 定义了一个关系。这将告诉 Django，每个 `Choice` 对象都关联到一个 `Question` 对象。Django 支持所有常用的数据库关系：多对一、多对多和一对一。



## blog

### 编写博客模型代码



```
#blog/models.py

from django.db import models
from django.contrib.auth.models import User


class Category(models.Model):
    """
    django 要求模型必须继承 models.Model 类。
    Category 只需要一个简单的分类名 name 就可以了。
    CharField 指定了分类名 name 的数据类型，CharField 是字符型，
    CharField 的 max_length 参数指定其最大长度，超过这个长度的分类名就不能被存入数据库。
    当然 django 还为我们提供了多种其它的数据类型，如日期时间类型 DateTimeField、整数类型 IntegerField 等等。
    django 内置的全部类型可查看文档：
    https://docs.djangoproject.com/en/2.2/ref/models/fields/#field-types
    """
    name = models.CharField(max_length=100)


class Tag(models.Model):
    """
    标签 Tag 也比较简单，和 Category 一样。
    再次强调一定要继承 models.Model 类！
    """
    name = models.CharField(max_length=100)


class Post(models.Model):
    """
    文章的数据库表稍微复杂一点，主要是涉及的字段更多。
    """

    # 文章标题
    title = models.CharField(max_length=70)

    # 文章正文，我们使用了 TextField。
    # 存储比较短的字符串可以使用 CharField，但对于文章的正文来说可能会是一大段文本，因此使用 TextField 来存储大段文本。
    body = models.TextField()

    # 这两个列分别表示文章的创建时间和最后一次修改时间，存储时间的字段用 DateTimeField 类型。
    created_time = models.DateTimeField()
    modified_time = models.DateTimeField()

    # 文章摘要，可以没有文章摘要，但默认情况下 CharField 要求我们必须存入数据，否则就会报错。
    # 指定 CharField 的 blank=True 参数值后就可以允许空值了。
    excerpt = models.CharField(max_length=200, blank=True)

    # 这是分类与标签，分类与标签的模型我们已经定义在上面。
    # 我们在这里把文章对应的数据库表和分类、标签对应的数据库表关联了起来，但是关联形式稍微有点不同。
    # 我们规定一篇文章只能对应一个分类，但是一个分类下可以有多篇文章，所以我们使用的是 ForeignKey，即一
    # 对多的关联关系。且自 django 2.0 以后，ForeignKey 必须传入一个 on_delete 参数用来指定当关联的
    # 数据被删除时，被关联的数据的行为，我们这里假定当某个分类被删除时，该分类下全部文章也同时被删除，因此     # 使用 models.CASCADE 参数，意为级联删除。
    # 而对于标签来说，一篇文章可以有多个标签，同一个标签下也可能有多篇文章，所以我们使用 
    # ManyToManyField，表明这是多对多的关联关系。
    # 同时我们规定文章可以没有标签，因此为标签 tags 指定了 blank=True。
    # 如果你对 ForeignKey、ManyToManyField 不了解，请看教程中的解释，亦可参考官方文档：
    # https://docs.djangoproject.com/en/2.2/topics/db/models/#relationships
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    tags = models.ManyToManyField(Tag, blank=True)

    # 文章作者，这里 User 是从 django.contrib.auth.models 导入的。
    # django.contrib.auth 是 django 内置的应用，专门用于处理网站用户的注册、登录等流程，User 是 
    # django 为我们已经写好的用户模型。
    # 这里我们通过 ForeignKey 把文章和 User 关联了起来。
    # 因为我们规定一篇文章只能有一个作者，而一个作者可能会写多篇文章，因此这是一对多的关联关系，和 
    # Category 类似。
    author = models.ForeignKey(User, on_delete=models.CASCADE)
```

首先是 `Category` 和 `Tag` 类，它们均继承自 `models.Model` 类，这是 django 规定的。`Category` 和 `Tag` 类均有一个`name` 属性，用来存储它们的名称。由于分类名和标签名一般都是用字符串表示，因此我们使用了 `CharField` 来指定 `name` 的数据类型，同时 `max_length` 参数则指定 `name` 允许的最大长度，超过该长度的字符串将不允许存入数据库。除了 `CharField` ，django 还为我们提供了更多内置的数据类型，比如时间类型 `DateTimeField`、整数类型 `IntegerField` 等等。

`Post` 类也一样，必须继承自 `models.Model` 类。文章的数据库表稍微复杂一点，主要是列更多，我们指定了这些列：

- `title`。这是文章的标题，数据类型是 `CharField`，允许的最大长度 `max_length = 70`。
- `body`。文章正文，我们使用了 `TextField`。比较短的字符串存储可以使用 `CharField`，但对于文章的正文来说可能会是一大段文本，因此使用 `TextField` 来存储大段文本。
- `created_time`、`modified_time`。这两个列分别表示文章的创建时间和最后一次修改时间，存储时间的列用 `DateTimeField` 数据类型。
- `excerpt`。文章摘要，可以没有文章摘要，但默认情况下 `CharField` 要求我们必须存入数据，否则就会报错。指定 `CharField` 的 `blank=True` 参数值后就可以允许空值了。
- `category` 和 `tags`。这是分类与标签，分类与标签的模型我们已经定义在上面。我们把文章对应的数据库表和分类、标签对应的数据库表关联了起来，但是关联形式稍微有点不同。我们规定一篇文章只能对应一个分类，但是一个分类下可以有多篇文章，所以我们使用的是 `ForeignKey`，即一对多的关联关系。且自 django 2.0 以后，ForeignKey 必须传入一个 on_delete 参数用来指定当关联的数据被删除时，被关联的数据的行为，我们这里假定当某个分类被删除时，该分类下全部文章也同时被删除，因此使用 models.CASCADE 参数，意为级联删除。

而对于标签来说，一篇文章可以有多个标签，同一个标签下也可能有多篇文章，所以我们使用 `ManyToManyField`，表明这是多对多的关联关系。同时我们规定文章可以没有标签，因此为标签 tags 指定了 `blank=True`。

`author`。文章作者，这里 `User` 是从 django.contrib.auth.models 导入的。django.contrib.auth 是 django 内置的应用，专门用于处理网站用户的注册、登录等流程。其中 `User` 是 django 为我们已经写好的用户模型，和我们自己编写的 `Category` 等类是一样的。这里我们通过 `ForeignKey` 把文章和 `User`关联了起来，因为我们规定一篇文章只能有一个作者，而一个作者可能会写多篇文章，因此这是一对多的关联关系，和 `Category` 类似。



#### 多对多和一对多

##### ForeignKey

`ForeignKey` 表明一种一对多的关联关系。比如这里我们的文章和分类的关系，一篇文章只能对应一个分类，而一个分类下可以有多篇文章。反应到数据库表格中，它们的实际存储情况是这样的：

##### ManyToManyField

`ManyToManyField` 表明一种多对多的关联关系，比如这里的文章和标签，一篇文章可以有多个标签，而一个标签下也可以有多篇文章。反应到数据库表格中，它们的实际存储情况是这样的：



- [django ForeignKey 简介](https://docs.djangoproject.com/en/2.2/topics/db/models/#relationships)
- [django ForeignKey 详细示例](https://docs.djangoproject.com/en/2.2/topics/db/examples/many_to_one/)
- [django ManyToManyField 简介](https://docs.djangoproject.com/en/2.2/topics/db/models/#many-to-many-relationships)
- [django ManyToManyField 详细示例](https://docs.djangoproject.com/en/2.2/topics/db/examples/many_to_many/)

### 迁移数据库

```
python manage.py makemigrations 
```

```
python manage.py migrate
```

http://download.yingjiesheng.com/dlb2021/nanjingbank.pdf

<<<<<<< HEAD


## restframework

### 

=======
>>>>>>> e94ab2aa461f1372fa11366cff1dfac678b18db7
## 运行

```
python manage.py runserver
```

<<<<<<< HEAD
## 参考网址

https://www.zmrenwu.com/courses/django-rest-framework-tutorial/materials/103/

https://www.byincd.com/bobjiang/article-0196/
=======
>>>>>>> e94ab2aa461f1372fa11366cff1dfac678b18db7
