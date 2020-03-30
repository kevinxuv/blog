---
layout: post
title: Python Guide
tags: 
    - python
---
![python](https://cdn-images-1.medium.com/max/1600/1*PPIp7twJJUknfohZqtL8pQ.png)

学习Python语言，可以通过经典书籍、技术视频、网络教程来学习，除了看书以外也需要多看优秀的开源项目，然后再多加练习，遇到问题多 Google，总结来讲就是：多看好书，多看优秀开源项目源代码，多练习，遇到问题多 Google。

## 特性

- 高级语言（当然是相对于汇编这种更加底层的语言来说）
- 面向对象
- 函数式编程
- 强大的社区以及丰富的开源库
- 动态语言

## 应用

我们可以用 `python` 做很多事情，比如：

- 网络开发（ [Django](https://www.djangoproject.com/), [Flask](https://flask.palletsprojects.com/en/1.1.x/)）
- 机器学习（ [Tensorflow](https://www.tensorflow.org/), [PyTorch](https://pytorch.org/)）
- 科学计算 ( [numpy](https://numpy.org/), [pandas](https://pandas.pydata.org/) )
- GUI编程（ [PyQt](https://www.riverbankcomputing.com/software/pyqt/intro), [Tkinter](https://docs.python.org/3/library/tkinter.html), [Kivy](https://kivy.org/) ）
- 网络爬虫 ( [Scrapy](https://scrapy.org/), [Selenium](https://selenium-python.readthedocs.io/) )

是的，跟其它语言（比如：Java、ruby 等）一样，`python` 基本上可以应用在很多方面

## 哪些大厂在用

国外

- YouTube
- [Instagram](https://www.youtube.com/watch?v=28jA0uP7W6w)
- Dropbox
- [Netflix](https://zhuanlan.zhihu.com/p/64624687)

国内

- 豆瓣
- 饿了么
- 知乎

这里没有列一些巨大厂（比如：国外的 FLAG, 国内的 BAT），因为对这些巨大厂来讲，语言不会是阻碍其发展的因素，而且其中大部分不以 `python` 为主要语言（比如：Facebook - PHP，Google - Java， Amazon - Java，腾讯和百度 - C/C++，阿里 - Java），列出来的都是以 `python` 作为主要开发语言的公司，`Youtube` 最早就是用 `python` 开发的，不过 `YouTube` 在被 `Google` 收购后，现在已经开始在用 `Go` 重写，`Dropbox`作为 [Guido](https://en.wikipedia.org/wiki/Guido_van_Rossum) 退休公司，现在也在用 `rust`, `Go` 重写部分项目，`Netflix` 也是一个巨大厂级别的公司，从 `Netflix` GitHub 上开源的项目以及技术博客看它使用的大多数语言是：Java、python、Go等，国内豆瓣和知乎是以 `Python` 作为主要开发语言，不过豆瓣也在用 `GO` 重写部分项目，饿了么在被阿里收购前后，就在迁移到 `Java` 上面。

## Books

python的经典书籍有很多

入门书籍

- 《python核心编程》

进阶书籍

- 《python cookbook》, 适用于 python3, [电子版](https://python3-cookbook.readthedocs.io/zh_CN/latest/index.html)
- 《Intermediate Python》, [gitbook 地址](https://eastlakeside.gitbooks.io/interpy-zh/content/)
- 《effective python》, 作者 [Brett Slatkin](https://www.linkedin.com/in/bslatkin/) 就职于 Google，主要负责基础设施的开发工作。

网络教程

- [geeksforgeeks python tutorials](https://www.geeksforgeeks.org/python-programming-language/) 强烈推荐 `geek for geek` 的 `python` 教程， 非常全面。
- [The Hitchhiker’s Guide to Python!](https://docs.python-guide.org/) 主要的作者是非常流行的 Python 开源 http client 库 [requests](https://github.com/requests/requests) 的作者 [Kenneth Reitz](https://github.com/kennethreitz)
- [learn python the hard way](https://learnpythonthehardway.org/book/)
- [python for you and me](https://pymbook.readthedocs.io/en/latest/) 作者 [Kushal Das](https://github.com/kushaldas) 是 python software foundation 的 director

## Projects

Python因为本身就是开源的，所以社区很活跃，当然至少2是这样的，2-3由于放弃了向上兼容，导致了一些问题，以至于社区的活跃度上也会受到影响，但是这不影响优秀的项目开源， 可以参考 [awesome python](https://github.com/vinta/awesome-python)。列举了非常好又流行的项目：

- [flask](https://github.com/pallets/flask), 非常优秀的实现了 WSGI 协议的 web framwork。
- [django](https://github.com/django/django), 非常流行的 Python web framwork, 社区很强大。
- [tornado](https://github.com/tornadoweb/tornado), 实现了nonblocking 网络I/O实现的 Python 网络库。
- [requests](https://github.com/requests/requests), python http client 事实上的标准库。
- [gunicorn](https://github.com/benoitc/gunicorn), 非常强大的 WSGI app container，借鉴于 ruby 的 Unicorn 项目。
- [sentry](https://github.com/getsentry/sentry), 非常强大的用于问题跟踪的工具。

## Specification

- [PEP20 - The Zen of Python](https://www.python.org/dev/peps/pep-0020/) python 之禅，也可以解读为python语言的设计哲学。
- [PEP8](https://www.python.org/dev/peps/pep-0008/) 作为 Python 官方的规范，讲的很清楚。
- [Google python style](https://github.com/google/styleguide/blob/gh-pages/pyguide.md)，Google 的编码规范也是非常值得参考的。

## Community

我想大部分人，遇到问题应该首先会选择 Google，一般语言问题的解答 [Stackoverflow](https://stackoverflow.com/) 在 Google 的搜索排名中一般比较靠前，而且对于新手，大部分问题都可以从 **Stackoverflow** 上找到答案，顺便你可以没事看看其他同学有关 Python 的提问 [Stackoverflow python](https://stackoverflow.com/questions/tagged/python)

## 总结

无论学习哪种语言，个人觉得比较好的方法是：Read more good books and awesome open source, practice more and google more.
