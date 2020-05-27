---
layout: post
title: Python at Netflix
tags: 
    - python
    - netflix
---
> 众所周知，Netflix 是 Java 大厂，它在 GitHub 上开源了很多 Java 项目，其中就有用 Java 语言实现的网关项目[zuul](https://github.com/Netflix/zuul)，但是一个大厂往往不会被语言限制，它还开源了很多 go, python, ruby 的项目，这篇文章是 Netflix 在 Medium 上的技术博客专栏里面写[关于 Python 在 Netflix 的应用]((https://medium.com/netflix-techblog/python-at-netflix-bba45dae649e)), 从这篇文章你可以大概看出 Netflix 的 Python 技术栈以及技术团队划分。

本片文章由 Netflix 的 Pythonistas 讲述，由 Amjith Ramanujam 协调，并由 Ellen Livengood 编辑。

![python](../assets/images/python.png)

由于我们很多人要准备去 PyCon，我们想分享一下 Netflix 应用 Python 的例子。 我们在完整的内容服务生命周期中使用了 Python，从决定什么样的内容一路到操作 CDN 上的内容为 1.48 亿会员服务。 我们使用很多开源 Python 项目并对这些项目做了些贡献，其中一些在下面会提到。 如果您对这些感兴趣，可以访问我们的[招聘网站](https://jobs.netflix.com/search?q=python)或者在 PyCon 上找到我们。 我们已经向 [PyLadies Auction](https://us.pycon.org/2019/events/auction/) 捐了几张 Netflix Originals 系列海报，并期待在那里见到你们。

# Open Connect

[Open Connect](https://openconnect.netflix.com/en/) 是 Netflix 的内容分发网络（CDN）。 一个在你按遥控器上的 Play 之前，对 Netflix 基础设施发生的所有事情（例如，你是否已经登录？你有什么计划？你看过什么？所以我们可以推荐新标题到给你，你想看什么？）的简单但不精确的思考，发生在亚马逊网络服务（AWS）中，而之后发生的一切（例如，视频流）都发生在 Open Connect 网络中。 内容就已经放置在尽可能靠近最终用户的 Open Connect CDN 的服务器网络上，从而改善了客户的流媒体体验，并降低了 Netflix 和我们的网络服务提供商（ISP）合作伙伴的成本。

设计，构建和操作这个 CDN 基础设施需要各种软件系统，其中很多都是用 Python 编写的。构成 CDN 大部分的网络设备主要由Python应用程序管理。这些应用程序跟踪我们的网络设备的库存：哪些设备，哪些型号，哪些硬件组件，位于哪些站点。这些设备的配置由几个其他系统控制，包括实际来源，设备配置应用和备份。用于收集运行状况和其他运行数据的设备交互是另一个 Python 应用程序。Python 长期以来一直是网络领域中流行的编程语言，因为它是一种直观的语言，使得工程师可以快速解决网络问题。后来又开发了大量有用的库，使得该语言更适合学习和使用。

# Demand Engineering

[Demand Engineering](https://www.linkedin.com/pulse/what-demand-engineering-aaron-blohowiak/) 负责 Netflix 云的[区域故障转移](https://opensource.com/article/18/4/how-netflix-does-failovers-7-minutes-flat)，流量分配，容量运营和车队效率。我们很自豪地说我们团队的工具主要是用 Python 造的。编排故障转移的服务使用 numpy 和 scipy 来执行数值分析，[boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) 用于更改我们的AWS基础架构，[rq](https://python-rq.org/)用于运行异步工作负载，我们将它们全部包含在 Flask API 的薄层中。进入[bpython](https://bpython-interpreter.org/) shell 即时解决问题的能力已经不止一次地挽救了这一天。

我们是 Jupyter Notebooks 和 [nteract](https://nteract.io/)的重度用户，用来分析操作数据和原型[可视化工具](https://github.com/nteract/nteract/tree/master/packages/data-explorer)，帮助我们检测容量回归。

# CORE

CORE团队在我们的告警和统计分析工作中使用 Python。 我们依靠许多统计和数学库（numpy，scipy，ruptures，pandas）来帮助在我们的告警系统指示问题时自动分析 1000 个相关信号。 我们开发了一个时间序列关联系统，用于团队内部和外部，以及一个分布式的 worker 系统，用于并行化大量分析工作，以快速提供结果。

Python 也是我们通常用于自动化任务，数据探索和清理的工具，和用于可视化工作的便捷资源。

# Monitoring, alerting and auto-remediation

Insight Engineering 团队负责构建和运行用于操作洞察，告警，诊断和自动修复的工具。随着 Python 的日益普及，该团队现在支持他们大多数服务的 Python 客户端。一个例子是 [Spectator](https://github.com/Netflix/spectator-py)  Python 客户端库，一个用于检测代码用于记录空间时序性指标的库。我们构建 Python 库用来跟其他 Netflix 平台级服务进行交互。除了库，[Winston](https://medium.com/netflix-techblog/introducing-winston-event-driven-diagnostic-and-remediation-platform-46ce39aa81cc) 和 [Bolt](https://medium.com/netflix-techblog/introducing-bolt-on-instance-diagnostic-and-remediation-platform-176651b55505)产品也使用 Python 框架（Gunicorn + Flask + Flask-RESTPlus）构建。

# Information Security

信息安全团队使用 Python 为 Netflix 实现了许多高杠杆目标：安全自动化，风险分类，自动修复和漏洞识别等等。 我们有许多成功的 Python 开源项目，包括 [Security Monkey](https://github.com/Netflix/security_monkey)（我们团队最活跃的开源项目）。通过使用 [Bless](https://github.com/Netflix/bless)，我们利用 Python 来保护我们的 SSH 资源。通过使用 [Repokid](https://github.com/Netflix/repokid)，我们的基础架构安全团队利用 Python 来帮助进行IAM权限调整。通过使用 [Lemur](https://github.com/Netflix/lemur)，我们使用 Python 来帮助生成 TLS 证书。

我们最近的一些项目包括 Prism：一个批处理框架，可帮助安全工程师测量铺设的道路采用，风险因素，和识别源代码中的漏洞。我们目前为 Prism 提供 Python 和 Ruby 库。[Diffy](https://medium.com/netflix-techblog/netflix-sirt-releases-diffy-a-differencing-engine-for-digital-forensics-in-the-cloud-37b71abd2698)取证分类工具完全用 Python 编写。通过使用 Lanius，我们还使用 Python 来检测敏感数据。

# Personalization Algorithms

我们在更广泛的[个性化机器学习基础设施](https://www.slideshare.net/FaisalZakariaSiddiqi/ml-infra-for-netflix-recommendations-ai-nextcon-talk)中广泛使用 Python 来训练 Netflix 体验的关键方面的一些机器学习模型：从我们的[推荐算法](https://research.netflix.com/research-area/recommendations)到[艺术品个性化](https://medium.com/netflix-techblog/artwork-personalization-c589f074ad76)到[营销算法](https://medium.com/netflix-techblog/engineering-to-scale-paid-media-campaigns-84ba018fb3fa)。例如，一些算法使用 TensorFlow，Keras 和 PyTorch 学习深度神经网络，XGBoost 和 LightGBM 来学习渐变 Boosted 决策树或 Python 中更广泛的科学技术栈（例如 numpy，scipy，sklearn，matplotlib，pandas，cvxpy）。因为我们不断尝试新的方法，我们使用 Jupyter Notebooks 来驱动我们的许多实验。我们还开发了许多高级库，用来帮助将这些库与我们的其他[生态系统](https://www.slideshare.net/FaisalZakariaSiddiqi/ml-infra-for-netflix-recommendations-ai-nextcon-talk)集成（例如数据访问，事实记录和特征提取，模型评估和发布）。

# Machine Learning Infrastructure

除个性化外，Netflix 还将机器学习应用于整个公司的数百个用例。其中许多应用程序都是由 [Metaflow](https://www.youtube.com/watch?v=XV5VGddmP24) 提供支持的，这是一个可以轻松地从原型阶段到生产阶段执行ML项目的 Python 框架。

Metaflow 推出了 Python 的极限：我们利用良好的并行化和优化的 Python 代码来获取 10Gbps 的数据，处理内存中数亿个数据点，并协调数万个 CPU 内核的计算。

# Notebooks

我们是 Netflix 公司中 Jupyter Notebooks 的狂热用户，我们之前已经写过[这项投资的原因和性质](https://medium.com/netflix-techblog/notebook-innovation-591ee3221233)。

但是 Python 在我们提供这些服务方面发挥着重要作用。 当我们需要开发，调试，探索和原型化与 Jupyter 生态系统的不同交互时，Python 是一种主要语言。 我们使用 Python 构建 Jupyter 服务器的自定义扩展，来允许我们代表用户管理记录，归档，发布和克隆 notebooks 等任务。我们通过不同的 Jupyter 内核为用户提供了许多版本的 Python，并使用 Python 管理这些内核规范的部署。

# Orchestration

Big Data Orchestration 团队负责提供所有服务和工具来安排和执行 ETL 和 Adhoc 管道。

业务流程服务的许多组件都是用 Python 编写的。从我们的调度程序开始，它使用 [Jupyter Notebooks](https://jupyter.org/) 和 [papermill](https://papermill.readthedocs.io/en/latest/) 来提供模板化的作业类型（Spark，Presto，...）。这使我们的用户能够以标准化和简便的方式表达需要执行的工作。你可以在[此处](https://medium.com/netflix-techblog/scheduling-notebooks-348e6c14cfd6)查看有关此主题的更多详细信息。 在需要人工干预的情况下，我们一直使用 notebooks 作为真实的 runbooks - 例如：重新启动过去一小时内失败的所有内容。

在内部，我们还构建了一个完全用 Python 编写的事件驱动平台。我们已经从许多系统中创建了事件流，这些系统统一到一个工具中。这允许我们定义过滤事件的条件，以及对它们作出反应或路由的动作。 因此，我们能够解耦微服务并了解数据平台上发生的所有事情。

我们的团队还构建了 [pygenie](https://github.com/Netflix/pygenie) 客户端，该客户端与联合作业执行服务 [Genie](https://netflix.github.io/genie/) 交互。 在内部，我们对此库有额外的扩展，可应用业务约定并与 Netflix 平台集成。 这些库是用户以编程方式与大数据平台中的工作进行交互的主要方式。

最后，我们团队致力于为 [papermill](https://papermill.readthedocs.io/en/latest/) 和 [scrapbook](https://nteract-scrapbook.readthedocs.io/en/latest/) 开源项目做出贡献。我们在那里的工作既包括我们自己的用例也包括外部用例。这些努力在开源社区中获得了很大的吸引力，我们很高兴能够为这些共享项目做出贡献。

# Experimentation Platform

用于实验的科学计算团队正在为科学家和工程师创建一个分析AB测试和其他实验的平台。 科学家和工程师可以在三个方面，数据，统计和可视化方面贡献新的创新。

Metrics Repo 是一个基于 [PyPika](https://pypika.readthedocs.io/en/latest/) 的 Python 框架，允许贡献者编写可重用的参数化 SQL 查询。它是任何新分析的切入点。

因果模型库是一个 Python＆R 框架，供科学家们为因果推理贡献新模型。 它利用 [PyArrow](https://arrow.apache.org/docs/python/) 和 [RPy2](https://rpy2.readthedocs.io/en/version_2.8.x/)，以便可以使用任何一种语言无缝地计算统计数据。

可视化库基于 [Plotly](https://plot.ly/)。由于 Plotly 是一种广泛采用的可视化规范，因此有许多工具可以让贡献者生成我们平台可以使用的输出。

# Partner Ecosystem

Partner Ecosystem group 正在扩展其对 Python 的使用，以便在设备上测试 Netflix 应用程序。 Python 正在形成新的 CI 基础架构的核心，包括控制我们的业务流程服务器，控制 Spinnaker，测试用例查询和过滤，以及在设备和容器上安排测试运行。使用 TensorFlow 在 Python 中进行额外的运行后分析，以确定哪些测试最有可能在哪些设备上显示问题。

# Video Encoding and Media Cloud Engineering

我们的团队负责编码（和重新编码）Netflix 目录，并利用机器学习来深入了解该目录。
我们在大约 50 个项目中使用 Python，例如 [vmaf](https://github.com/Netflix/vmaf/blob/master/resource/doc/references.md) 和 [mezzfs](https://medium.com/netflix-techblog/mezzfs-mounting-object-storage-in-netflixs-media-processing-platform-cda01c446ba)，我们使用名为 [Archer](https://medium.com/netflix-techblog/simplifying-media-innovation-at-netflix-with-archer-3f8cbb0e2bcb) 的媒体地图缩减平台构建[计算机视觉解决方案](https://medium.com/netflix-techblog/ava-the-art-and-science-of-image-discovery-at-netflix-a442f163af6)，并且我们将Python用于许多内部项目。
我们还开源了一些工具来简化Python项目的开发/分发，比如 [setupmeta](https://pypi.org/project/setupmeta/) 和 [pickley](https://pypi.org/project/pickley/)。

# Netflix Animation and NVFX

Python 是我们用于创建 Animated 和 VFX 内容的所有主要应用程序的行业标准，因此不用说我们正在大量使用它。 我们与Maya 和 Nuke 的所有集成都在 Python 中，而我们的 Shotgun 工具大部分也在 Python 中。 我们刚刚开始在云中使用我们的工具，并期望部署我们自己的许多自定义 Python AMIs/ 容器。

# Content Machine Learning, Science & Analytics

内容机器学习团队广泛使用Python来开发机器学习模型，这是预测所有内容的受众大小，收视率和其他需求指标的核心。
