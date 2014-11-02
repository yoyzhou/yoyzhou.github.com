---
layout: post
title: "基于UID的WEIBO信息抓取框架WEIBO_SCRAPY"
date: 2013-04-08 20:55
comments: true
categories: [Python, WEIBO, Scrapy, Threading, Framework]
keywords: Python, WEIBO, Crawler, Threading, Framework, 微博, 抓取, 多线程, 框架, UID, SINA
description: 本文介绍基于微博UID的SINA微博信息抓取框架WEIBO_SCRAPY。 WEIBO_SCRAPY是一个PYTHON实现的，使用多线程抓取WEIBO信息的框架。WEIBO_SCRAPY框架给用户提供WEIBO的模拟登录和多线程抓取微博信息的接口，让用户只需关心抓取的业务逻辑，而不用处理棘手的WEIBO模拟登录和多线程编程。
---

本文介绍基于微博UID的SINA微博信息抓取框架WEIBO\_SCRAPY。 WEIBO\_SCRAPY是一个PYTHON实现的，使用多线程抓取WEIBO信息的框架。WEIBO\_SCRAPY框架给用户提供WEIBO的模拟登录和多线程抓取微博信息的接口，让用户只需关心抓取的业务逻辑，而不用处理棘手的WEIBO模拟登录和多线程编程。

###WEIBO\_SCRAPY
WEIBO\_SCRAPY是一个PYTHON实现的，使用基于微博UID的方式从WEIBO.COM页面抓取信息的框架。框架以微博UID为最小单位，为每一个UID分配一个抓取线程，因此每一个UID相当于一个**抓取任务**。WEIBO\_SCRAPY内置微博模拟登录和多线程框架，让用户只需关注以微博UID为基础的抓取业务逻辑。具体的说，使用WEIBO\_SCRAPY用户只需要重载WEIBO\_SCRAPY的**抓取任务**方法为自己的抓取逻辑，即可实现多线程地抓取微博信息（详见下文`WEIBO_SCRAPY的实现`小节）。

###WEIBO\_SCRAPY的功能
1\. 微博模拟登录

2\. 多线程抓取框架

3\. **抓取任务**接口

4\. 抓取参数配置

###WEIBO\_SCRAPY的实现

#### 1 微博模拟登录
请参考拙文[Python模拟登录新浪微薄（使用RSA加密方式和Cookies文件）][weibo_login]

#### 2 多线程抓取框架
WEIBO\_SCRAPY使用[Queue][python_queue]实现多线程抓取框架，Queue中的元素为微博用户的UID，每一个抓取线程消费Queue中的UID，抓取线程在获得UID之后交给**抓取任务**处理该UID。框架实现`从某一个UID开始`和`从文件中加载UID列表`两种抓取方式，无论何种方式都有可能返回在抓取过程中新获得的UID(通过解析抓取的页面获得)，并加入到Queue中，对于从单一某个UID开始的方式，返回新获得的UID是推荐的做法，不然Queue中就只有开始UID一个抓取任务。

> The Queue module implements multi-producer, multi-consumer queues. It is especially useful in threaded programming when information must be exchanged safely between multiple threads. 

关于更多Python多线程编程的知识，请参考拙文[Python线程同步机制: Locks, RLocks, Semaphores, Conditions, Events和Queues][threading]

以下是WEIBO\_SCRAPY多线程框架的实现代码，分别是抓取线程和抓取的主进程，在第二段代码中抓取的主进程生成用户指定数目的抓取进行执行抓取任务：

抓取线程

{% codeblock  lang:python %}

class scrapy_threading(threading.Thread):
    """Thread class to handle scrapy task"""
    
    def __init__(self, task, wanted):
        threading.Thread.__init__(self)
        self.do_task = task
        self.wanted = wanted
        
    def run(self):
        global visited_uids
        global task_queue
        global scraped
        global lock
    
        while scraped < self.wanted:
            
            #crawl info based on each uid
            if task_queue:
              
                uid = task_queue.get()
                
                if uid in visited_uids: #already crawled
                    task_queue.task_done()
                
                else:
                    try:
                        gains = self.do_task(uid)
                        
                        #per debug
                        wow = '{0: <25}'.format('[' + time.asctime() + '] ') + ' uid_' + '{0: <12}'.format(uid)
                        print wow
                        for uid in gains:
                            task_queue.put(uid)
                        
                        #signals that queue job is done
                        task_queue.task_done()
                        
                        #counting scrapied number
                        with lock:
                            scraped += 1
                            #per debug
                            print 'scraped: ' + str(scraped)
                            
                    except Exception, e:
                        print e
                        pass
                        
            else:
                time.sleep(30)
{% endcodeblock %}

抓取的主进程

{% codeblock lang:python %}
def scrapy(self):
        
        login_status = login(self.login_username, self.login_password, self.cookies_file)
    
        if login_status:
            
            if self.start_uid:
                task_queue.put(self.start_uid)
            
            elif self.uids_file:
                uids_list = self.__load_uids__()
                for uid in uids_list:
                    task_queue.put(uid)
                
            else: #start uid or uids file is needed
                raise Exception('ERROR: Start uid or uids file is needed.') 
           
            #spawn a pool of threads, and pass them queue instance 
            for _ in range(self.thread_number):
                st = scrapy_threading(self.scrapy_do_task, self.wanted)
                st.setDaemon(True)
                st.start()
                
            
            task_queue.join()
                
{% endcodeblock %}

#### 3 抓取任务接口

{% codeblock lang:python %}
  def scrapy_do_task(self, uid=None):
        '''
        User needs to overwrite this method to perform uid-based scrapy task.
        @param uid: weibo uid
        @return: a list of uids gained from this task, optional
        '''
        #return []
        pass
{% endcodeblock %}
以上是抓取任务的接口，WEIBO\_SCRAPY暴露scrapy\_do\_task方法，用户只需要继承WEIBO\_SCRAPY的scrapy类并且重写scrapy\_do\_task方法为自己的抓取业务逻辑。
像前面提到过的一样，抓取任务是基于微博用户的UID的，所以scrapy\_do\_task方法有一个uid参数。

#### 4 抓取参数配置
WEIBO\_SCRAPY提供简单易用的账户信息配置和抓取参数配置，在`scrapy.ini`文件中即可轻松的完成参数的配置，以下是一个配置文件的样本：

{% codeblock lang:python %}
[login_account_info]
#account info for logining weibo 
login_username = ur_login_account_name_here
login_uid = weibo_uid_of_login_account_here
login_password = account_password_here
cookies_file = weibo_cookies.dat

[scrapy_settings]
thread_number = 50
wanted = 100000
#only one property of below 2 is required, and start_uid takes advantage of uids_file
#also note that arguments from constructor will overwrite this two properties 
start_uid = 1197161814
uids_file =
{% endcodeblock %}
用户也可以在scrapy类的构造器中指定配置文件的位置。

###WEIBO\_SCRAPY的使用
使用WEIBO\_SCRAPY用户只需要继承WEIBO\_SCRAPY的scrapy类并且重写scrapy\_do\_task方法为自己的抓取业务逻辑。

以下示例代码可在[weibo_scrapy/example.py][example]中找到。

{% codeblock lang:python %}
#!/usr/bin/env python
#coding=utf8

from weibo_scrapy import scrapy


class my_scrapy(scrapy):
    
    def scrapy_do_task(self, uid=None):
         '''
        User needs to overwrite this method to perform uid-based scrapy task.
        @param uid: weibo uid
        @return: a list of uids gained from this task, optional
        '''
         super(my_scrapy, self).__init__(**kwds)
         
         #do what you want with uid here, note that this scrapy is uid based, so make sure there are uids in task queue, 
         #or gain new uids from this function
         print 'WOW...'
         return 'replace this string with uid list which gained from this task'
     
if __name__ == '__main__':
    
    s = my_scrapy(start_uid = '1197161814')
    s.scrapy()
    
{% endcodeblock %}



###项目源代码
WEIBO\_SCRAPY项目的源代码地址：[weibo_scrapy][weibo_scrapy]，[Fork it][fork]。

###相关阅读
1\. [Python模拟登录新浪微薄（使用RSA加密方式和Cookies文件）][weibo_login]

2\. [使用Beautiful Soup抽取网页数据，解析微博用户关注信息][following_ntk]


`---EOF---`


<!-- PUT reference-style links below-->
[weibo_login]: /blog/2013/03/18/sina-weibo-login-simulator-in-python/
[python_queue]: http://docs.python.org/2/library/queue.html
[threading]:/blog/2013/02/28/python-threads-synchronization-locks/
[example]:https://github.com/yoyzhou/weibo_scrapy/blob/master/example.py
[weibo_following_ntk_scrapy]:https://github.com/yoyzhou/weibo_scrapy/blob/master/weibo_following_ntk_scrapy.py
[following_ntk]:/blog/2013/03/23/extract-data-with-beautifulsoup-taking-weibo-4-example/
[weibo_scrapy]: https://github.com/yoyzhou/weibo_scrapy
[fork]:https://github.com/yoyzhou/weibo_scrapy/fork






