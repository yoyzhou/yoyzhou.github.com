---
layout: post
title: "Python模拟登录新浪微薄（使用RSA加密方式和Cookies文件）"
date: 2013-03-18 21:15
comments: true
categories: [Python, WEIBO, RSA2, Encryption, Crawler]
keywords: Python, WEIBO, RSA2, Encryption, Crawler, Scrapy
description: 本文介绍使用Python和RSA2加密方式模拟用户登录新浪微薄,文中使用的代码可以在https://github.com/yoyzhou/weibo_login中找到。
---

本文简单介绍如何使用[PYTHON](http://www.python.org/)模拟用户登录新浪微薄,文中使用的代码可以在[github.com/yoyzhou/weibo_login](https://github.com/yoyzhou/weibo_login)中找到。

###为什么要模拟登录
一般来说获取微薄数据的方式有两种，一种是使用WEIBO官方提供的API接口；另外一种方式就是从WEIBO网站页面上抓取数据。从网面上抓取数据，相对于使用WEIBO API更加灵活，可控性更强，不用受WEIBO API调用次数的限制；但是易受WEIBO页面结构变动的影响，使得程序可靠性低，不适合在生产系统中使用。

> 当然还有一种是由**别人**提供下载的，下载WEIBO数据可以考虑[爬盟中国](http://www.cnpameng.com/)和[数据堂](http://www.datatang.com/)

本文要讲的内容跟页面抓取数据有关，但是像[WEIBO.COM](weibo.com)这样的[SNS](http://en.wikipedia.org/wiki/SNS)网站必须事先登录之后才能访问到她的数据，所以如何模拟用户登录WEIBO，就成为从网页上抓取微薄数据的第一步。

###模拟登录相关资源
目前网上有很多关于模拟用户登录WEIBO的文章:

\+ 使用[HTTPFOX](https://addons.mozilla.org/en-us/firefox/addon/httpfox/)来侦测用户登录WEIBO.COM的过程[[1]](http://blog.csdn.net/yonglaixiazaide/article/details/7923468), [[2]](http://www.jishuziyuan.com/archive/supeercrsky/8016047.html)

\+ 模拟登录PHP[[3]](http://blog.csdn.net/lgg201/article/details/8050606)

\+ Python实现[[4]](http://hi.baidu.com/enmzqbeadvfhiye/item/4018b4e7775cd3edfa42bad3), [[5]](http://www.cnblogs.com/mouse-coder/archive/2013/03/03/2941265.html)

\+ JAVA实现[[6]](http://marspring.mobi/http-client-weibo/)

其中[[4] 新浪微博登录rsa加密方法](http://hi.baidu.com/enmzqbeadvfhiye/item/4018b4e7775cd3edfa42bad3)介绍了使用RSA2加密方式模拟登录，[[5] 模拟新浪微博登录（Python+RSA加密算法）](http://www.cnblogs.com/mouse-coder/archive/2013/03/03/2941265.html)和[[6] httpclient登录新浪微博（非SDK方式）](http://marspring.mobi/http-client-weibo/)分别给出了RSA加密算法模拟登录WEIBO的PYTHON和JAVA实现。

> 关于RSA加密方式，参考维基词条RSA[[zh]](http://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95), [[en]](http://en.wikipedia.org/wiki/RSA)

关注PYTHON实现的读者可以阅读下面关于此事的豆瓣讨论：

> Python模拟新浪微薄登录的豆瓣讨论[[7]](http://www.douban.com/note/201767245/)

###Python+RSA2+Cookies实现模拟登录
由于新浪登录加密方式的改变，参见[[7]](http://www.douban.com/note/201767245/)，这里仅介绍使用RSA加密方法登录，要使用RSA加密方式，必须安装RSA模块，所以：
<!-- more -->

####1.安装PYTHON实现的RSA加密算法模块[python-rsa](https://pypi.python.org/pypi/rsa/3.1.1).

{% codeblock  lang:bash %}
> easy_install rsa
{% endcodeblock %}
python-rsa的文档地址[http://stuvel.eu/files/python-rsa-doc/index.html](http://stuvel.eu/files/python-rsa-doc/index.html)

####2.使用RSA加密用户登录密码

{% codeblock  lang:python %}
def get_pwd_rsa(pwd, servertime, nonce):
    """
        Get rsa2 encrypted password, using RSA module from https://pypi.python.org/pypi/rsa/3.1.1, documents can be accessed at 
        http://stuvel.eu/files/python-rsa-doc/index.html
    """
    #n, n parameter of RSA public key, which is published by WEIBO.COM
    #Hardcoded here but you can also find it from values return from prelogin status above
    weibo_rsa_n = 'EB2A38568661887FA180BDDB5CABD5F21C7BFD59C090CB2D245A87AC253062882729293E5506350508E7F9AA3BB77F4333231490F915F6D63C55FE2F08A49B353F444AD3993CACC02DB784ABBB8E42A9B1BBFFFB38BE18D78E87A0E41B9B8F73A928EE0CCEE1F6739884B9777E4FE9E88A1BBE495927AC4A799B3181D6442443'
    
    #e, exponent parameter of RSA public key, WEIBO uses 0x10001, which is 65537 in Decimal
    weibo_rsa_e = 65537
   
    message = str(servertime) + '\t' + str(nonce) + '\n' + str(pwd)
    
    #construct WEIBO RSA Publickey using n and e above, note that n is a hex string
    key = rsa.PublicKey(int(weibo_rsa_n, 16), weibo_rsa_e)
    
    #get encrypted password
    encropy_pwd = rsa.encrypt(message, key)

    #trun back encrypted password binaries to hex string
    return binascii.b2a_hex(encropy_pwd)
{% endcodeblock %}

####3.验证用户登录是否成功，并且保存Cookies

{% codeblock  lang:python %}
 data = urllib2.urlopen(login_url).read()
 #Verify login feedback, check whether result is TRUE
 patt_feedback = 'feedBackUrlCallBack\((.*)\)'
 p = re.compile(patt_feedback, re.MULTILINE)
        
 feedback = p.search(data).group(1)
        
 feedback_json = json.loads(feedback)
 if feedback_json['result']:
 	cookie_jar2.save(cookie_file,ignore_discard=True, ignore_expires=True) #Save Cookies
 	return 1
 else:
 	return 0
{% endcodeblock %}

###存在的问题
登入过程中返回ERROR-4049:需要输入验证码，这个问题在[[5] 模拟新浪微博登录（Python+RSA加密算法）](http://www.cnblogs.com/mouse-coder/archive/2013/03/03/2941265.html)中也有提到。

###源代码
文中代码片段的源码地址[github.com/yoyzhou/weibo_login](https://github.com/yoyzhou/weibo_login)


###相关阅读

1\. [使用Beautiful Soup抽取网页数据，解析微博用户关注信息][following_ntk_post]，介绍如何使用Beautiful Soup抽取网页数据。

2\. [基于UID的WEIBO信息抓取框架WEIBO_SCRAPY][weibo_scrapy_post]，详细介绍了多线程WEIBO数据抓取框架WEIBO_SCRAPY。


`---EOF---`


[weibo_login_post]: /blog/2013/03/18/sina-weibo-login-simulator-in-python/
[following_ntk_post]:/blog/2013/03/23/extract-data-with-beautifulsoup-taking-weibo-4-example/
[weibo_scrapy_post]: /blog/2013/04/08/weibo-scrapy-framework-with-multi-threading/

