---
layout: post
title: 系统设计面试之短链接
tags: 
    - system design
    - shortly
    - interview
---
> 本文翻译自GitHub上有关系统设计的repo：[system design primer](https://github.com/donnemartin/system-design-primer)里面关于面试系统设计之短链接，6w+的star，短链接这种系统设计，我想大家都有去了解或者有自己做过类似的系统设计，系统设计关键在于严谨的设计思路和方法论，国外的程序员和国内的程序员有很大的差别，差别在于国外的程序员有很科学的方法论，就好比MBA的教育，在国内是混圈子拿文凭，而国外的MBA教育除了混圈子以外，还教会你一种MBA领域语言和一种科学的方法论。

# 设计 Pastebin.com (或者 Bit.ly)

*Note: 为了避免重复，当前文档直接链接到[系统设计主题](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#系统设计主题的索引)的相关区域，请参考链接内容以获得综合的讨论点、权衡和替代方案。*

**设计 Bit.ly** - 是一个类似的问题，区别是pastebin需要存储的是paste的内容，而不是原始的未短化的url。

## 第一步：概述用例和约束

> 收集这个问题的需求和范畴。
> 问相关问题来明确用例和约束。
> 讨论一些假设。

因为没有面试官来明确这些问题，所以我们自己将定义一些用例和约束。

### 用例

#### 我们将问题的范畴限定在如下用例

* **用户** 输入一段文本，然后得到一个随机生成的链接
  * 过期设置
    * 默认的设置是不会过期的
    * 可以选择设置一个过期的时间
* **用户** 输入一个paste的url后，可以看到它存储的内容
* **用户** 是匿名的
* **Service** 跟踪页面分析
  * 一个月的访问统计
* **Service** 删除过期的pastes
* **Service** 需要高可用

#### 超出范畴的用例

* **用户** 可以注册一个账户
  * **用户** 通过验证邮箱
* **用户** 可以用注册的账户登录
  * **用户** 可以编辑文档
* **用户** 可以设置可见性
* **用户** 可以设置短链接

### 约束和假设

#### 状态假设

* 访问流量不是均匀分布的
* 打开一个短链接应该是很快的
* pastes只能是文本
* 页面访问分析数据可以不用实时
* 一千万的用户量
* 每个月一千万的paste写入量
* 每个月一亿的paste读取量
* 读写比例在10:1

#### 计算使用

**向面试官说明你是否应该粗略计算一下使用情况。**

* 每个paste的大小
  * 每一个paste 1 KB
  * `shortlink` - 7 bytes
  * `expiration_length_in_minutes` - 4 bytes
  * `created_at` - 5 bytes
  * `paste_path` - 255 bytes
  * 总共 = ~1.27 KB
* 每个月新的paste内容在12.7GB
  * (1.27 * 10000000)KB/月的paste
  * 三年内将近450GB的新paste内容
  * 三年内3.6亿短链接
  * 假设大部分都是新的paste，而不是需要更新已存在的paste
* 平均4paste/s的写入速度
* 平均40paste/s的读取速度

简单的转换指南:

* 2.5百万 req/s
* 1 req/s = 2.5百万 req/m
* 40 req/s = 1亿 req/m
* 400 req/s = 10亿 req/m

## 第二步：创建一个高层次设计

> 概述一个包括所有重要的组件的高层次设计

![Imgur](http://i.imgur.com/BKsBnmG.png)

## 第三步：设计核心组件

> 深入每一个核心组件的细节

### 用例：用户输入一段文本，然后得到一个随机生成的链接

我们可以用一个[关系型数据库](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#关系型数据库管理系统rdbms)作为一个大的哈希表，用来把生成的url映射到一个包含paste文件的文件服务器和路径上。

为了避免托管一个文件服务器，我们可以用一个已有的托管的**对象存储**，比如Amazon的S3或者[NoSQL文档类型存储](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#文档类型存储)。

有一个替代作为一个大的哈希表的关系型数据库的方案，我们可以用[NoSQL键值存储](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#键-值存储)。我们需要讨论[选择SQL或NoSQL之间的权衡](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#sql-还是-nosql)。下面的讨论是使用关系型数据库方法。

* **客户端** 发送一个创建paste的请求到作为一个[反向代理](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#反向代理web-服务器)启动的**Web服务器**。
* **Web服务器** 转发请求给 **写接口** 服务器
* **写接口** 服务器执行如下操作：
  * 生成一个唯一的url
    * 检查这个url在**SQL数据库**里面是否是唯一的
    * 如果这个url不是唯一的，生成另外一个url
    * 如果我们支持自定义url，我们可以使用用户提供的url（也需要检查是否重复）
  * 把生成的url存储到**SQL数据库**的`pastes`表里面
  * 存储paste的内容数据到**对象存储**里面
  * 返回生成的url

**向面试官阐明你需要写多少代码**

`pastes`表可以有如下结构：

```sql
shortlink char(7) NOT NULL
expiration_length_in_minutes int NOT NULL
created_at datetime NOT NULL
paste_path varchar(255) NOT NULL
PRIMARY KEY(shortlink)
```

我们将在`shortlink`字段和`created_at`字段上创建一个[数据库索引](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#使用正确的索引)，用来提高查询的速度（避免因为扫描全表导致的长时间查询）使其存储的数据保存在内存中，从内存里面顺序读取1MB的数据需要大概250微秒，而从SSD上读取则需要花费4倍的时间，从硬盘上则需要花费80倍的时间。<sup><a href=https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#每个程序员都应该知道的延迟数>1</a></sup>

为了生成唯一的url，我们可以：

* 使用[**MD5**](https://en.wikipedia.org/wiki/MD5)来哈希用户的IP地址+时间戳
  * MD5是一个普遍用来生成一个128-bit长度的哈希值的一种哈希方法
  * MD5是一致分布的
  * 或者我们也可以用MD5哈希一个随机生成的数据
* 用[**Base 62**](https://www.kerstner.at/2012/07/shortening-strings-using-base-62-encoding/) 编码MD5哈希值
  * 对于urls，使用Base 62编码 `[a-zA-Z0-9]`是比较合适的
  * 对于每一个原始输入只会有一个hash结果，Base 62是确定的（不涉及随机性）
  * Base 64是另外一个流行的编码方案，但是对于urls，会因为额外的`+`和`-`字符串而产生一些问题
  * 以下[Base 62伪代码](http://stackoverflow.com/questions/742013/how-to-code-a-url-shortener) 执行的时间复杂度是O(k)，k是数字的数量=7：

```python
def base_encode(num, base=62):
    digits = []
    while num > 0
      remainder = modulo(num, base)
      digits.push(remainder)
      num = divide(num, base)
    digits = digits.reverse
```

* 取输出的前7个字符，结果会有62^7个可能的值，应该足以满足在3年内处理3.6亿个短链接的约束：

```python
url = base_encode(md5(ip_address+timestamp))[:URL_LENGTH]
```

我们将会用一个公开的[**REST风格接口**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#表述性状态转移rest)：

```shell
$ curl -X POST --data '{ "expiration_length_in_minutes": "60", \
    "paste_contents": "Hello World!" }' https://pastebin.com/api/v1/paste
```

Response:

```json
{
    "shortlink": "foobar"
}
```

用于内部通信，我们可以用[RPC](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#远程过程调用协议rpc)。

### 用例：用户输入一个paste的url后可以看到它存储的内容

* **客户端** 发送一个获取paste请求到 **Web Server**
* **Web Server** 转发请求给 **读取接口** 服务器
* **读取接口** 服务器执行如下操作：
  * 在**SQL数据库**检查这个生成的url
    * 如果这个url在**SQL数据库**里面，则从**对象存储**获取这个paste的内容
    * 否则，返回一个错误页面给用户

REST API：

```shell
curl https://pastebin.com/api/v1/paste?shortlink=foobar
```

Response:

```json
{
    "paste_contents": "Hello World",
    "created_at": "YYYY-MM-DD HH:MM:SS",
    "expiration_length_in_minutes": "60"
}
```

### 用例： 服务跟踪分析页面

因为实时分析不是必须的，所以我们可以简单的**MapReduce** **Web Server**的日志，用来生成点击次数。

```python
class HitCounts(MRJob):

    def extract_url(self, line):
        """Extract the generated url from the log line."""
        ...

    def extract_year_month(self, line):
        """Return the year and month portions of the timestamp."""
        ...

    def mapper(self, _, line):
        """Parse each log line, extract and transform relevant lines.

        Emit key value pairs of the form:

        (2016-01, url0), 1
        (2016-01, url0), 1
        (2016-01, url1), 1
        """
        url = self.extract_url(line)
        period = self.extract_year_month(line)
        yield (period, url), 1

    def reducer(self, key, values):
        """Sum values for each key.

        (2016-01, url0), 2
        (2016-01, url1), 1
        """
        yield key, sum(values)
```

### 用例： 服务删除过期的pastes

为了删除过期的pastes，我们可以直接搜索**SQL数据库**中所有的过期时间比当前时间更早的记录，
所有过期的记录将从这张表里面删除（或者将其标记为过期）

## 第四步：扩展这个设计

> 给定约束条件，识别和解决瓶颈。

![Imgur](http://i.imgur.com/4edXG0T.png)

**Important: 不要简单的从最初的设计直接跳到最终的设计**

说明您将迭代地执行这样的操作：1)**Benchmark/Load 测试**，2)**Profile** 出瓶颈，3)在评估替代方案和权衡时解决瓶颈，4)重复前面，可以参考[在AWS上设计一个可以支持百万用户的系统](../scaling_aws/README.md)这个用来解决如何迭代地扩展初始设计的例子。

重要的是讨论在初始设计中可能遇到的瓶颈，以及如何解决每个瓶颈。比如，在多个**Web服务器**上添加**负载平衡器**可以解决哪些问题？ **CDN**解决哪些问题？**Master-Slave Replicas**解决哪些问题? 替代方案是什么和怎么对每一个替代方案进行权衡比较？

我们将介绍一些组件来完成设计，并解决可伸缩性问题。内部的负载平衡器并不能减少杂乱。

*为了避免重复的讨论*， 参考以下[系统设计主题](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#系统设计主题的索引)获取主要讨论要点、权衡和替代方案：

* [DNS](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#域名系统)
* [CDN](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#内容分发网络cdn)
* [负载均衡器](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#负载均衡器)
* [水平扩展](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#水平扩展)
* [反向代理（web 服务器）](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#反向代理web-服务器)
* [应用层](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#应用层)
* [缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#缓存)
* [关系型数据库管理系统 (RDBMS)](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#关系型数据库管理系统rdbms)
* [SQL write master-slave failover](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#故障切换)
* [主从复制](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#主从复制)
* [一致性模式](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#一致性模式)
* [可用性模式](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#可用性模式)

**分析存储数据库** 可以用比如Amazon Redshift或者Google BigQuery这样的数据仓库解决方案。

一个像Amazon S3这样的**对象存储**，可以轻松处理每月12.7 GB的新内容约束。

要处理每秒40 *平均*读请求(峰值更高)，其中热点内容的流量应该由**内存缓存**处理，而不是数据库。**内存缓存**对于处理分布不均匀的流量和流量峰值也很有用。只要副本没有陷入复制写的泥潭，**SQL Read Replicas**应该能够处理缓存丢失。

对于单个**SQL Write Master-Slave**，*平均*每秒4paste写入(峰值更高)应该是可以做到的。否则，我们需要使用额外的SQL扩展模式:

* [联合](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#联合)
* [分片](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#分片)
* [非规范化](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#非规范化)
* [SQL调优](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#SQL调优)

我们还应该考虑将一些数据移动到**NoSQL数据库**。

## 额外的话题

> 是否更深入探讨额外主题，取决于问题的范围和面试剩余的时间。

### NoSQL

* [键值存储](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#键-值存储)
* [文档存储](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#文档类型存储)
* [列型存储](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#列型存储)
* [图数据库](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#图数据库)
* [sql 还是 nosql](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#sql-还是-nosql)

### 缓存

* 在哪缓存
  * [客户端缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#客户端缓存)
  * [CDN缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#cdn-缓存)
  * [Web 服务器缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#web-服务器缓存)
  * [数据库缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#数据库缓存)
  * [应用缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#应用缓存)
* 缓存什么
  * [数据库查询级别的缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#数据库查询级别的缓存)
  * [对象级别的缓存](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#对象级别的缓存)
* 何时更新缓存
  * [缓存模式](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#缓存模式)
  * [直写模式](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#直写模式)
  * [回写模式](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#回写模式)
  * [刷新](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#刷新)

### 异步和微服务

* [消息队列](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#消息队列)
* [任务队列](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#任务队列)
* [背压](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#背压)
* [微服务](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#微服务)

### 通信

* 讨论权衡:
  * 跟客户端之间的外部通信 - [HTTP APIs following REST](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#表述性状态转移rest)
  * 内部通信 - [RPC](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#远程过程调用协议rpc)
* [服务发现](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#服务发现)

### 安全

参考[安全](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#安全).

### 延迟数字

见[每个程序员都应该知道的延迟数](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#每个程序员都应该知道的延迟数).

### 持续进行

* 继续对系统进行基准测试和监控，以在瓶颈出现时解决它们
* 扩展是一个迭代的过程
