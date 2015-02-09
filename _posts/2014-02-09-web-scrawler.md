---
layout: post
title: 爬虫总结
category: web_crawler
---

本文的目标是简单的讲述下在做爬虫的时候一般思路， 以及遇到过的坑

##总览
本质上爬虫就是模拟浏览器所做的动作， 在一般的网站浏览器都是发送**Get/Post**来获取数据，当然也有部分网站也在使用**ROA风格**  
这时，有可能就需要用到其他的HTTP方法。包括所谓的**Ajax技术**或者**动态加载效果**, 后者大多是通过前者实现。
##技巧

###利用好浏览器

####Network
大多时候(几乎所有时候)制作爬虫都是在**构造URL或表单**，或者是重定向到其他网页，那么浏览器的network就是很好的东西。  
network是个类似于浏览器日志的东西, 会记录操作。

####关闭JS
另外一个很头疼的问题是，网页中的数据是HTML中就有的， 还是通过Ajax加载？在这种情况下，我选择关闭浏览器的JS渲染，  
**在通常情况下**，如果有, HTML，如果无, Ajax。当然也有例外:  

*	 数据存在于HTML中，但是仍然无， 这种情况多半是数据是需要通过js组件呈现(某吧某段时间)

所以综上在判断数据来源的时候一定搜索一遍源码。

####XPath、CSSPath
chrome浏览器现在新增了选中某个标签返回XPath、CSSPath的功能，但是基本的[XPath]的知识也是需要的。

###选择好方式

####浏览器扩展
浏览器扩展最大的好处是， 可以免除很多麻烦的登录，可以直接利用浏览器作为现成的JS渲染。  
缺陷也是蛮大的， 无法丢数据库。抢课啥的， 可以完全胜任。

####少数页面
简单情况下， 选择一个优秀的Http库 + HTMLparser就足够了.

*	Java: [HttpClient] + [Jsoup]  
*	Python: [requests] + [BeautifulSoup]  
* Ruby: [Net::HTTP] + [nokogiri]  
* Node: 说实话node是个很有诱惑力的平台， 用JQuery来获取数据(大雾！)  
* 用天生异步构建一个高效的爬虫库不难， 但是无赖本人时间不够。。希望大家去玩一玩

####大型爬虫框架
*	Java: [WebMagic]  
*	Python: [Scrapy]

##疑难杂症

###动态加载
很多人看见动态加载的页面就想用**selenium**(请自动百度), 本文开头也阐述了即使是JS渲染的页面，  
本质上还是基本的HTTP动作, 对于动态加载, 我一般的处理方法为, 打开network面板，同时让页面，  
实现加载, 在network中看请求的URL和参数, 然后伪造一份。

###反反爬虫策略
很多网站都有反爬虫策略， 我遇到过的包括:

*	判断User-Agent有无, (有些网站即使没有User-agent但是返回的是过期数据，注意！)
*	判断Host的来源, 这种一般是下载的时候。
*	最小间隔时间, (这种情况要么，买代理， 要么扩大间隔时间) 

##最后的考虑
在很多情况下, 我们爬网站就是爬下数据, 所以对数据的敏感性很重要, 特别是在构造URL的时候, 组装form的时候.
所以在组装的时候不要想当然, 一定要有根据.

###为自己打个广告
如果有任何相关的问题, 工作推荐, 请务必发我邮箱.谢谢~

###感谢
最后的最后感谢开源，让开发更简单。

[XPath]: "http://www.w3school.com.cn/xpath/index.asp"
[HttpClient]: "http://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient"
[Jsoup]: "http://mvnrepository.com/artifact/org.jsoup/jsoup"
[requests]: "https://github.com/kennethreitz/requests"
[BeautifulSoup]: "https://github.com/bdoms/beautifulsoup"
[Net::HTTP]: "http://www.ruby-doc.org/stdlib-2.2.0/libdoc/net/http/rdoc/index.html"
[nokogiri]: "http://www.nokogiri.org/"
[WebMagic]: "https://github.com/code4craft/webmagic"
[Scrapy]: "http://scrapy.org/"
{% include references.md %}
