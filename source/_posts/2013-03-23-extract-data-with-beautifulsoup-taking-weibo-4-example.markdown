---
layout: post
title: "使用Beautiful Soup抽取网页数据，解析微博用户关注信息"
date: 2013-03-23 17:14
comments: true
categories: [Python, WEIBO, BeautifulSoup, Social Network, Data Analysis]
keywords: Python, WEIBO, BeautifulSoup, Social Network, Data Analysis
description: 本文介绍了Beautiful Soup，PYTHON实现的HTML/XML标记的解析器；简要描述了Beautiful Soup的安装以及使用；最后以抽取微博用户关注信息为例详细的演示了如何使用Beautiful Soup。
---

本文介绍了Beautiful Soup，PYTHON实现的HTML/XML标记的解析器；简要描述了Beautiful Soup的安装以及使用；最后以抽取微博用户关注信息为例详细的演示了如何使用Beautiful Soup。

###什么是Beautiful Soup
[Beautiful Soup][bs4]是用PYTHON实现的HTML/XML标记的解析器。它提供简单和通用的方法对HTML/XML语法树进行浏览（navigating），搜索（searching）以及修改（searching）。它甚至可以针对不规范的标记生成语法树，可以大大地减少开发人员的时间。

> Beautiful Soup is an HTML/XML parser for Python that can turn even invalid markup into a parse tree. It provides simple, idiomatic ways of navigating, searching, and modifying the parse tree. It commonly saves programmers hours or days of work. <cite>[Beautiful Soup][bs3]</cite>

###安装Beautiful Soup
安装Beautiful Soup很简单，如果你已经安装过pip或者easy_install,如果您还没有安装过Python安装工具，建议您参考[Install Python Setuptools/Distribute for Python2 and Python3][easy_install]。

{% codeblock  lang:bash %}
sudo easy_install beautifulsoup4
{% endcodeblock %}
需要注意的是：BeautifulSoup4支持Python 2.x和3.x，而BeautifulSoup3只支持Python 2.x，Beautiful Soup官网建议大家应该使用BeautifulSoup4而不是BeautifulSoup3。

> Beautiful Soup 3 only works on Python 2.x, but Beautiful Soup 4 also works on Python 3.x. Beautiful Soup 4 is faster, has more features, and works with third-party parsers like lxml and html5lib. You should use Beautiful Soup 4 for all new projects.

###使用Beautiful Soup
首先，导入Beautiful Soup。

{% codeblock  lang:python %}
from bs4 import BeautifulSoup
{% endcodeblock %}
注意如果你使用的是BeautifulSoup3，那么导入语句可能是：

{% codeblock  lang:python %}
from BeautifulSoup import BeautifulSoup
{% endcodeblock %}
然后，使用BeautifulSoup为你生成标记语言的语法树。

{% codeblock  lang:python %}
soup = BeautifulSoup(open('my.html'))
{% endcodeblock %}
得到了语法树soup之后，就可以调用相应的接口浏览，搜索和修改你的标记文件。比如下面的语句搜索my.html中所有'action-type'是'user_item'的div：

{% codeblock  lang:python %}
soup.findAll('div', attrs={'action-type' : 'user_item'})
{% endcodeblock %}

上面简单介绍了Beautiful Soup的安装和使用，更多Beautiful Soup的文档请参考官方文档[bs3][]和[bs4][]。下面我们以从WEIBO.COM页面中解析出用户的关注信息为例，介绍Beautiful Soup的使用。

###Beautiful Soup实例：解析微博用户的关注信息
社交网络中的关注信息（followings）是用户对什么人/东西感兴趣的一种表达，从关注信息中可以得到用户的兴趣偏好，又因为关注信息有用户自己维护，所以相对于粉丝（followers）信息更能体现个人偏好。以微博来说，关注就是用户关注的人，一般认为用户是根据自己的兴趣爱好出发有选择的关注帐号。

微博中有两种关注，我的关注和他人的关注，由于这两种关注的页面结构不同，所以在解析的时候需要分别对待，但是分析的过程是同理的，只是在抽取数据是的页面标签不一样，使用上面的Beautiful Soup工具，抽取时标签的定位会很容易，这就是使用Beautiful Soup带来的好处。

####1.使用浏览器的`Inspect Element`功能理解页面的结构

最新版的Chrome和Firefox都自身内置有`Inspect Element`功能，在编写代码时，可能要经常的使用它来定位要寻找的页面元素。Chrome浏览器`Inspect Element`的使用请参考[Chrome Developer Tools - Elements Panel][chrome-inspect-element]。
<!-- more -->

####2.从页面中提取用户关注的HTML字段，构建Beautiful Soup对象

通过上面一步的分析，我们大致了解用户关注列表的页面结构，接下来就把页面文件/流导入到Beautiful Soup中，让它为我们生成页面结构树。可是当我们查看新浪微博页面源码的时候，情况却不是这样的，我们发现页面源码中很多信息并不是以HTML元素的形式呈现，而是以plain文本形式放到了页面的Javacript脚本里面，这时就更加凸显了Beautiful Soup的伟大之处了，只要我们在Script里面找到了相应的代码（实际上是json格式存放的数据），抽取出来再导入到Beautiful Soup中，这个问题也就迎刃而解。下面的正则表达式用来从页面中提取Script中的json数据并且使用其中的HTML字段生成soup：

{% codeblock  lang:python %}
def parse_followings(page_content):
    '''
    @param page_content: html page file or response stream from Internet
    '''

    #reguler expression to extract json data which contains html info
    patt_view = '<script>STK && STK.pageletM && STK.pageletM.view\((.*)\)</script>'
    patt = re.compile(patt_view, re.MULTILINE)
   
    weibo_scripts = patt.findall(page_content)
    
    for script in weibo_scripts: 
	view_json = json.loads(script)
        
        if 'html' in view_json and view_json['pid'] == 'pl_relation_hisFollow':
            html = view_json['html']
            soup = BeautifulSoup(html)	#WOW...we got the soup
		
{% endcodeblock %}

####3.通过Beautiful Soup获取需要的页面元素，并从中抽取感兴趣的信息

通过上一步获得following的HTML信息，导入到Beautiful Soup中，接下来就使用Beautiful Soup抽取信息了。如何去定位元素当然有很多方式，结合`Inspect Element`,可以很容易的做到，获得元素之后就可以抽取需要的信息了。

{% codeblock  lang:python %}
	
#all the followings, search according element type(li) and attributes
friendollowings = soup.findAll('li', attrs={'class':'clearfix S_line1', 'action-type' : 'itemClick'})

for user_item in friendollowings:
    
    action_data = user_item.get('action-data')
    user_info = {}
    for field in action_data.split("&"):
        field_name, field_value = field.split('=')
        user_info[field_name] = field_value
    
    
    for  info in  [more for more in user_item('div') if isinstance(more, Tag)]:
        class_name = info['class']
        if class_name == 'name':
            user_info['name'] = clean_content(info.a.text)
            user_info['address'] = clean_content(info.span.text)
        elif class_name == 'connect':
            user_info['connect'] = clean_content(info.text)
        elif class_name == 'face mbspace': #face image
            user_info['face'] = info.a.img['src']
        elif class_name == 'weibo':
            pass
            #user_info['lasttweet'] = clean_content(info.text)
    
{% endcodeblock %}

###结束语
文中阐述了如何使用Beautiful Soup抽取微博页面的用户关注信息，最后有几点需要提出：
1.模拟用户登录

结合本人的上一篇文章[Python模拟登录新浪微博（使用RSA加密方式和Cookies文件）][weibo-login]，可以实现模拟用户登录之后发送用户关注的URL请求`http://weibo.com/{uid}/follow`获得用户关注的页面；

2.分页抽取

解析页面`http://weibo.com/{uid}/follow`可以获取用户关注的总页面数，通过`http://weibo.com/{uid}/follow?page={page_num}`可以实现分页抽取。

###相关阅读

1\. [Python模拟登录新浪微薄（使用RSA加密方式和Cookies文件）][weibo_login_post],使用RSA2加密方式模拟登录SINA微博。

2\. [基于UID的WEIBO信息抓取框架WEIBO_SCRAPY][weibo_scrapy_post]，详细介绍了多线程WEIBO数据抓取框架WEIBO_SCRAPY。



`---EOF---`


<!-- PUT reference-style links below-->
[bs3]: http://www.crummy.com/software/BeautifulSoup/bs3/documentation.html
[bs4]: http://www.crummy.com/software/BeautifulSoup/
[easy_install]: /blog/2012/08/12/install-python-setuptools-slash-distribute-for-both-python2-and-python3/
[kaifulee_followings]: /images/kaifulee_followings.png
[chrome-inspect-element]: https://developers.google.com/chrome-developer-tools/docs/elements
[weibo-login]: /blog/2013/03/18/sina-weibo-login-simulator-in-python/
[weibo_login_post]: /blog/2013/03/18/sina-weibo-login-simulator-in-python/
[following_ntk_post]:/blog/2013/03/23/extract-data-with-beautifulsoup-taking-weibo-4-example/
[weibo_scrapy_post]: /blog/2013/04/08/weibo-scrapy-framework-with-multi-threading/


