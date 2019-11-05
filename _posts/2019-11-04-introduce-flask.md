---
layout: post
title: Introduce Flask
tags: 
    - python
    - web framework
    - flask
    - wsgi
---
![flask](http://flask.palletsprojects.com/en/1.1.x/_images/flask-logo.png)

> [flask](http://flask.palletsprojects.com/en/1.1.x/) 是 [Armin Ronacher](https://github.com/mitsuhiko) 开源在 [GitHub](https://github.com/pallets/flask) 上实现了 python [wsgi](https://wsgi.readthedocs.io/en/latest/index.html) 协议的轻量级 [micro framework](http://flask.palletsprojects.com/en/1.1.x/foreword/#what-does-micro-mean)，截止目前，已经有 4w+ star，具体数字已经超过python领域前辈 [Django](https://github.com/django/django)，至少从GitHub的数据来讲，flask已经是当前非常流行的 python web 框架了。

# 起源

关于项目的起源，可以从 Armin 在 [2016 SF python](https://www.youtube.com/watch?v=1ByQhAM5c1I) 的分享的几个原因：

1. 我想开发一个可以分布式部署的软件
2. 最初我是想开发一个python版本的 [phpBB](https://www.phpbb.com/)
3. 受到用工具集去开发一个类似 [trac](https://trac.edgewall.org/) 的网站的启发，而不是用Django的方式
4. 不强加一些配置在框架用户层面，而是让开发者能够自己掌控配置（我想这肯定是针对Django的）

而对于 flask 在 GitHub 或者说开源世界受到如此欢迎，Armin 用了一个词来表达他的感受：`scarily popular`, 同时又说了 `I am not sure why`，他也讲到 flask 跟 Django 在设计上的区别：你可以利用Django做很多事情，但是用flask做的就很少，flask更像一个 `fixed box`，他并不想要使得这个box变的很大。我想这就是flask和Django最大的区别，当然你可以从这个视频里面看到更多的来自Armin关于2者的区别。

在我看来，Django 是一个一站式开发的框架，你可以在 Django 的世界里面找到 `everything you need`，但是你要用 flask 做相同的事情的话，你需要借助 flask 的 extension（可以从 [Awesome Flask](https://github.com/humiaozuzu/awesome-flask) 这个GitHub项目找到这些插件），这是俩者的优点，也是俩者的缺点，缺点和优点的判断，取决于开发者的使用场景，假如你想要开发一个 web 后台，Django 生态可能更合适，你不必造重复的轮子，Django 生态里面的 library 都是由 Django 社区维护，从而使得其项目质量有一定的保证，而相同的事情，你用 flask 去做，你需要接受的是 flask 的插件都是开发者贡献的开源项目，项目的质量是无法保证的，而假如你仅仅是需要开发一个 [micro service](https://en.wikipedia.org/wiki/Microservices)，或者 web api，并且你更喜欢用一些流行的其它 python library 做其它事情，比如，你想用流行的orm框架  [sqlalchemy](https://www.sqlalchemy.org/) 或者 [peewee](http://docs.peewee-orm.com/en/latest/) 等，你不喜欢用django自带的orm，那选择flask这种轻量的框架会更合适，而最关键的评估标准就是你用它们开发的投入产出比。

flask 的流行，在我看来，可以从宏观和微观的角度来看：

## 宏观

### python 的流行

flask 的流行受益于 [python](https://en.wikipedia.org/wiki/Python_(programming_language)) 的流行，随着AI领域的发展，标志性的事件就是 Google 的 [AlphaGo](https://en.wikipedia.org/wiki/AlphaGo) 挑战李世石那一战，AI一战成名，AI也越来越受到人们的关注，新的技术发展，往往需要从实验室走出，走进工业生产，走进人们的日常生活，当AI被大众所熟悉的时候，说明这项技术已经走出了实验室，开始进入人们的生活，再发展下去，AI will be everywhere，在互联网领域，很多大佬在他们的学生阶段都是做AI研究的，比如大家熟知的李开复先生，他在CMU研究的领域就是计算机语音识别，而他的导师就是大名鼎鼎的AI领域的先驱图灵奖得主 [Raj Reddy](https://en.wikipedia.org/wiki/Raj_Reddy)，还有大家熟悉的沈向洋、陆奇、洪小文，都是李开复先生的师弟，就是由这些人前赴后继的努力，才有如今的AI能应用到人们的生活当中，如今已经到了AI研究落地收获的季节。

作为AI领域（机器学习、计算机视觉，计算机语音识别等）的原生语言 python，也就受益于AI的发展，从最新的 [TIOBE](https://www.tiobe.com/tiobe-index/) 排名，python 排名第三，曾经一度（非常短暂）排名第一，但是python在企业开发（排除AI）领域的发展却远不如第一名 Java，这也就是 Java 排名第一的原因，Java 在企业开发领域一直是霸主的地位，python 一度想在企业开发领域发力，但是受限于语言的设计以及社区力量的原因，并没有扩大 python 在企业开发领域的份额，python3 已经是社区最大的努力，力求跟 Java 在企业开发领域一博，到目前为止，始终无法撼动 Java 在企业开发领域的地位，但是随着计算机领域的发展，未来是不可知的，十年河东十年河西，在不远的将来，或许 python 能从局部突破然后打破 Java 的在企业开发领域的地位也不无可能，或许这2种语言会被新的语言所淘汰也不无可能，还是那句话：未来是不可知的。

### 移动互联网时代

互联网从 pc 发展到到 mobile，web 系统的架构也已经更多的趋向于前后端分离的架构，Java 的 `JSP`，C# 的 `ASP`，早期的 php，Django 老的版本等这些前后端一起的技术架构早已经被淘汰，虽然 flask 也有自己的模版引擎 jinja ，但是这不是必须的，你只需要利用 flask 来实现 [web server](https://en.wikipedia.org/wiki/Web_server) 跟你的应用通信，把你的接口通过 http 协议暴露给你的端使用即可，虽然那些古老的框架也进行了演化，从而支持前后端分离，但是 flask 这种天生只关注后端实现的框架可能就更适合来实现后端服务接口。

## 微观

### 微服务的流行

从微观的角度看，flask的流行又受益于[微服务](https://en.wikipedia.org/wiki/Microservices)在企业中的应用，企业应用架构从单点架构，到SOA服务化，到更细粒度的微服务，再到现在流行的 `service mesh`，从避免单点故障，到分布式架构提倡的为了避免系统雪崩而允许局部故障，都需要把业务通过微服务进行更细粒度的拆分解耦，从而提升系统的业务稳定，但是系统的总体的稳定性其实是下降了，这时候一些中间件就可以单独从原先的系统里面抽象出来成为单独的系统组件，并且还需要支持高可用、高并发的能力，比如：`auth`，`api gateway`，`session`，`permission`，`log`，`trace`，`metrics`，而微服务越来越专注于业务本身逻辑实现，从而使得开发者对框架的需求从大而全到专而精，开发者的需求就是需要框架做一件简单的事情，开发一个微服务，通过网络暴露接口给其它微服务，而微服务之间通过 `rpc` 调用通信，flask作为一个轻量级的实现了 wsgi 协议的框架，很容易用来开发基于http通信的微服务，以至于它提供基于 [jinja](https://jinja.palletsprojects.com/en/2.10.x/) 的模版功能都不需要用到，这个时候轻量级的框架带来的是方便快捷，并且在性能上也相较于Django这种大而全的框架要好（可以参考 [web framework benchmarks](https://www.techempower.com/benchmarks/))。

# 优点

Armin 在 [2016 SF python](https://www.youtube.com/watch?v=1ByQhAM5c1I&t=330s) 也从他作为作者的角度去讲flask在各个方面的优点（具体可以仔细看视频），从我的角度看，除了他讲的那几点以外，还有几个优点，首先是一个优秀的开源项目共通的优点：

1. 文档丰富（better documented）
2. 持续维护和更新，flask开发者或者团队还在不断对项目进行维护和更新（bug fix or new feature etc），flask 已经发布了`1.0`版本
3. 使用的人多，意味着你遇到的大部分问题都可以 Google 到 or [stackoverflow](https://stackoverflow.com/questions/tagged/flask) 到，that is very important to me.
4. 源代码质量高，假如你遇到问题，通过 Google 都找不到答案时（这种情况很少），你就不得不通过阅读源代码去找到解决方法，这时候源代码的质量就非常重要

使用上的优点：

1. 快速开发`API`，有多快呢，看看 flask 的 [demo](http://flask.palletsprojects.com/en/1.1.x/quickstart/#a-minimal-application) 就知道
2. 丰富的接口，`request` 的封装，`error handle`，基于 thread local 的 `g`，hooks: `before_request`, `after_request`等等，让你只需要关注自己的业务实现，而不用关心 wsgi 协议的抽象和封装
3. 方便的路由管理，你可以方便的使用 [Blueprints](http://flask.palletsprojects.com/en/1.1.x/blueprints/#blueprints) 管理你的路由

# 缺点

我想唯一的缺点就是：

1. 虽然支持 python3，但是不支持 `asyncio`，这点 Armin 在视频中也提到这点

但他也讲到一个[方案](https://www.youtube.com/watch?v=1ByQhAM5c1I&t=279s) 去实现一个 flask 与 websocket 通信的方案，也提到 [Django channel](https://channels.readthedocs.io/en/latest/) 的 [asgi](https://asgi.readthedocs.io/en/latest/) 实现方案。

# 最后

flask 是一个已经被很多公司应用到生产环境的架构，正是由于它给开发者带来的方便和稳定，才促使它成为一个拥有4w+ star 的流行框架。
