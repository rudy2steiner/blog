---
<!-- title: How to contribute to Kubernetes if you have a full-time job -->
title: 如果你有全职工作，如何为Kubernetes做贡献
toc: true
tags: [K8s,翻译]
---
你甚至可以在业余时间参与最大的开源项目之一的内部工作。

<!-- You can work on the internals of one of the largest open source projects, even in your spare time. -->
2018年10月，我开始为[Kubernetes](https://opensource.com/resources/what-is-kubernetes) (K8s)做贡献，那时我在IBM的产品安全事件响应团队工作。我被分布式系统所吸引，但日常工作中接触不到，所以我的导师，孙林，建议我在空闲时间为开源的分布式系统做贡献。我开始对k8s感兴趣，再也没有回头。
<!-- I started contributing to Kubernetes (K8s) in October 2018, when I was working on the Product Security Incident Response Team at IBM. I was drawn to distributed systems, but I couldn't work with them in my day job, so my mentor, Lin Sun, suggested I contribute to open source distributed systems in my spare time. I became interested in K8s and have never looked back! -->

我主要在Kubernetes测试、存储和发布特别兴趣组内工作。我最为人所知的代码贡献是在Saad Ali的带领下实现了[插件管理](https://github.com/kubernetes/kubernetes/pull/73891)。插件管理器是一个管理插件注册/注销的控制器。它给Kubelet插件注册在注册插件（如容器存储接口或设备插件）失败场景下，添加了重试和指数退避逻辑。
<!-- I've worked mainly in the Kubernetes special interest groups (SIGs) sig-testing, sig-storage, and sig-release. My most notable code contribution was implementing the plugin manager under the lead of Saad Ali. The plugin manager is a controller that manages plugin registration/unregistration. This gives Kubelet plugin registration retry and exponential backoff logic in cases where registration of plugins (like CSI or device plugin) fails. -->

2019年1月，我成为k8s的会员，并于2019年7月作为开源贡献和开发倡导者加入IBM开放技术组，目前我正在为[Knative](https://knative.dev)，一个基于Kubernetes的无服务器负载部署和管理平台，做贡献。

<!-- I became a member of K8s in January 2019 and joined the IBM Open Technology team as an open source contributor and developer advocate in July 2019, where I currently contribute to Knative, a Kubernetes-based platform for deploying and managing serverless workloads. -->

现在Kubernetes非常欢迎贡献者，如果你想跟着我的足迹，在空闲时间开始贡献，继续阅读。
<!-- New Kubernetes contributors are more than welcome, so if you would like to follow my path and start contributing in your spare time, continue reading. -->

<!-- ## Skills you need to start contributing to Kubernetes  -->
## 开始为Kubernetes做贡献需要的技能

### 任意的版本控制系统(如git,svn)
<!-- Any version control system (e.g. git, svn) -->

K8s的源代码托管在Github，了解如何使用Git或者其它版本控制系统是非常重要的。首先，使自己熟悉这些[常见命令](https://git-scm.com/docs/giteveryday)。

<!-- Skills you need to start contributing to Kubernetes
Any version control system (e.g. git, svn)
K8s' source code is hosted on GitHub, so it's important that you know how to work with Git or another version control system. To get started, familiarize yourself with these common commands. -->

### Golang
<!-- Golang -->
我有C/C++的使用背景，但在开始前我并不了解Golang。如果你熟悉面向对象编程，Golang应该相当容易上手。我建议在学习K8s代码库的同时[学习Go](https://opensource.com/article/18/11/learning-golang)。我的同事黄伟向我介绍[Ultimate Go编程](https://www.oreilly.com/library/view/ultimate-go-programming/9780135261651/)系列，它极大地帮助了我。
<!-- I come from a C/C++ background and did not know any Golang before I started. If you are familiar with object-oriented programming, Golang should be fairly easy to pick up. I suggest learning Go while you're learning the K8s code base. My coworker Wei Huang introduced me to the Ultimate Go Programming video series, which helped me immensely. -->

### 其它要求
<!-- Other requirements -->

在你的提交可以被合并到K8s代码库之前，必须签署[贡献者许可协议](https://github.com/kubernetes/community/blob/master/CLA.md)（CLA）。还要注意社区准则、编码准则、如何设置你的开发环境以及其它Kubernetes Github代码库中的“[开始贡献之前](https://github.com/kubernetes/community/tree/master/contributors/guide#before-you-get-started)”描述的事。

<!-- Before your commits can be merged into the K8s code base, you must sign the Contributor License Agreement (CLA). Also, be aware of the community guidelines, code of conduct, how to set up your dev environment, and other things described in "before you get started" in Kubernetes' GitHub repo. -->

## 如何选取你的第一个Kubernetes问题
<!-- ## How to pick your first Kubernetes issue -->

在Kubernetes中找事做的首要的方法是查看开放问题列表，你可以用标签,如好的第一个问题，需要帮组，过滤问题列表,这些标签表明这个问题对新手非常友好。偶尔问题会被打上错误标签,也许问题的技术难度被低估了，并被错误的标记为好的第一个问题。所以一个“好的第一个问题”比你预料的更复杂也不必惊讶。

![](https://opensource.com/sites/default/files/uploads/kubernetes-issues.png)
<!-- The first way to find things to work on in Kubernetes is to look at the list of open issues. You can filter the list of issues by tag, such as "good first issue" and "help wanted," which indicate the issue is newcomer-friendly. Occasionally, an issue can be mislabeled; maybe the issue's technical complexity is underestimated, and it's mislabeled as "good first issue." So, don't be surprised if a "good first issue" seems more complicated than you expect. -->

第二个找事做的方法是搜索代码库里的“TODO”。这里有上百个低优先级的改善。学习代码库和从某人的“TODO”列表移除某一项是非常好的学习方法-双赢。
<!-- The second way to find things to work on is to search for "TODO"s in the codebase. There are hundreds of TODOs for lower priority improvements. It's a great way to learn the codebase and cross an item off of someone's TODO list—a win-win! -->

## 如何处理问题
<!-- ## How to work on an issue -->

阅读任何与之相关的问题或拉取请求（PRs）来帮助你理解上下文和问题本身。如果某个问题描述不清楚，在投入时间之前，确保联系问题的创建者以获得更清晰的理解。如果它是一个错误，验证你是否能复现错误。请注意这可能会花费很多时间,我花了很多时间去设置环境以尝试复现错误。

<!-- Read any linked issues or pull requests (PRs) to help you understand an issue's context and problem. If the issue description is not clear, make sure to reach out to the issue creator to get a clearer understanding before investing time in it. If it's a bug, verify that you can reproduce the bug. Note that this can take a significant investment of time; I have spent a lot of time setting up my environment to try to reproduce bugs. -->

一旦你有解决方案的想法，在拉取请求之前最好在Slack上联系问题的创建者，验证你的方法。如果在一周之内没有收到反馈，直接发起拉取请求这样他可以对具体方案做评论。

<!-- Once you have an idea for a solution, it is a good idea to reach out to the issue creator on Slack to verify your approach before making a PR. If you have not heard back from the creator within a week, go ahead a make a PR so the person can review it with a concrete solution. -->

起初我非常困惑就一个问题发起讨论之后，需要多快提交PR。因为我不得不在全职工作以外的时间为K8s工作。我还在思考这会如何影响我的工作-生活平衡。在浏览了问题和其它拉取请求后，我发现我需要在两周内提供PR或者更新问题的状态。在我看来，如果你有一个初始的、没做测试的实现，也可以提交初步的PR，这样可以尽快得到是否步入正轨的反馈。

<!-- At first, I was confused about was how quickly I should submit a PR after calling dibs on an issue. Since I had to work on K8s outside my fulltime job, I also wondered how this would affect my work-life balance. After looking through issues and other PRs, I discovered I needed to provide a PR or status update within at least two weeks. If you have an initial implementation that's not tested yet, in my opinion, it's still okay to make the initial PR so you can get feedback as soon as possible about whether you are on the right track. -->

在开发你的方案的时，确保添加了单元测试或集成测试以验证错误已修复或者特性和预期的一样。K8s使用名为[Prow](https://github.com/kubernetes/test-infra/tree/master/prow)的持续集成/持续开发（CI/CD）系统，它会为有 /ok-to-test 标签的PR运行所有的单元和集成测试。CI任务不会自动执行如果你不是Kubernetes项目的成员。这种情况下，我建议你先本地测试，然后在相关的特别兴趣组Slack频道找Kubernetes Github组织成员去你的PR评论 /ok-to-test。

<!-- While developing your solution, make sure to add unit tests or integration tests to verify the bug is fixed or that the feature works as intended. K8s uses a continuous integration/continuous development (CI/CD) system called Prow, which runs all unit and integration tests for a PR if it has the /ok-to-test label. The CI jobs will not run automatically if you're not a member of the Kubernetes project. In that case, I suggest that you run the test locally first, then ask on the relevant SIG Slack channel for a member of the Kubernetes GitHub org to comment /ok-to-test on your PR -->

## 如何加入Kubernetes社区

<!-- ## How to join the Kubernetes community -->

对于每天的交流，[Kubernetes Slack](http://slack.k8s.io)非常适合直接和其他贡献者联系并请教问题。我建议你加入自己最感兴趣的特别兴趣组（如客户端、存储、测试等）频道。
<!-- For everyday communication, the Kubernetes Slack is great for direct messaging other contributors and asking questions. I suggest joining the SIG channels you are most interested in (such as sig-cli, sig-storage, sig-testing, etc.). -->

KubeCons是与其他贡献者面对面交流的好地方。[2019](https://events19.linuxfoundation.org/events/kubecon-cloudnativecon-north-america-2019/)和[2020](https://events19.linuxfoundation.org/events/kubecon-cloudnativecon-north-america-2020/)年9月在北美有一次，3/4月在[欧洲](https://events19.linuxfoundation.org/events/kubecon-cloudnativecon-europe-2020/)有一次，5月在[亚洲](https://events19.lfasiallc.com/events/kubecon-cloudnativecon-open-source-summit-china-2020/)有一次。 我强烈推荐有抱负的新手和经验丰富的贡献者参与下面的两项活动：

<!-- KubeCons are a great place to meet other contributors face-to-face. There's one in North America in November 2019 and 2020, one in Europe in March/April, and one in Asia in July. I highly recommend the following two events for all aspiring, new, and seasoned contributors: -->

* Kubernetes贡献者峰会：在KubeCon正式开始的前一天，一整天活动都是免费的。既有为新人也有为贡献者准备的聚焦于学习和开发的工作坊。

<!-- * Kubernetes Contributor Summit: This is a free, all-day event the day before KubeCon officially starts. There are workshops for both new and current contributors that are focused on learning and development. -->

* 交流和辅导环节：在这里，你可以和许多包括Kubernetes在内的CNCF项目中资深的开源人士会面。你将和其他两位结对探讨技术和社区问题，甚至可以就选取的问题做结对编程。

<!-- * Networking + Mentoring Session: This is a place to meet with experienced open source veterans across many CNCF projects, including Kubernetes. You will be paired with two other people in a pod-like setting to explore technical issues and community questions, and you can even do pair-programming on a problem of your choice. -->

## 其他贡献方式

<!-- ## Other ways to contribute -->

除了贡献代码，还有许多其它的贡献方式。审查代码很重要，因为这可以帮助审查者和维护者减轻负担并提供多样的观点。也是学习K8s代码库的最佳方法之一。

<!-- There are many other ways to contribute besides writing code. Reviewing code is important, as it helps existing reviewers and maintainers with their review workload and provides a diversity of opinions. It's also one of the best ways to learn the K8s codebase! -->

你还可以加入发布小组作为角色的影子，为Kubernetes发布流程做贡献。访问[角色手册](https://github.com/kubernetes/sig-release/tree/master/release-team/role-handbooks)学习更多发布团队中的不同角色。我曾作为测试基础架构的影子领导两个版本发布直到我们自动化了整个流程，并取消了这个角色。

<!-- You can also contribute to the Kubernetes release process by joining the release team as a shadow for a role. Access the role handbooks to learn more about the different roles on the release team. I shadowed the Test Infra lead for two releases until we automated the process and eliminated the role. -->

成为一个影子教会我很多，包括CI/CD自动化和一个高度可视化的、影响全世界公司的产品背后的项目管理。发布团队的持续贡献以自动化任务和消除所有角色的手动任务来达到更好的CI/CD给我留下了特别深刻的印象。

<!-- Being a shadow has taught me a lot, including about the CI/CD automation and project management behind a highly visible product that impacts companies all around the world. I was especially impressed by the continuing effort of the release team to automate tasks and eliminate manual tasks for all roles to achieve better CI/CD. -->

如果你对成为一个影子感兴趣，查看正式的[申请流程](https://github.com/kubernetes/sig-release/blob/master/release-team/shadows.md)。
<!-- If you're interested in being a shadow, check out the formal process for applying. -->

如果你在空余时间参与K8s，一个需要注意的问题是：每周有发布团队会议，但是不要求影子所有的会议都参加。会议通常在美国太平洋时区的工作时间进行，持续半小时。幸运地是我的工作很灵活足以让我每周预定2～3次会议室参加发布团队的会议。


<!-- One note if you're working on Kubernetes in your spare time: There are weekly release team meetings, but shadows are not required to attend all of them. The meetings usually happen during work hours in the US Pacific time zone and last a half hour. Fortunately, my job was flexible enough for me to book a conference room two or three times a week to attend meetings with the release team. -->

## 如果你有全职工作，如何管理时间
<!-- ## How to manage your time when you have a full-time job -->

 每周设定用于贡献K8s的小时数非常有帮助，因为非常容易陷入开发活动中。当你工作和生活变得忙的时候，暂时停止对K8s的贡献也非常重要。k8s开发社区的宕机时间通常和一个发布剪裁的代码冻结期一致。
 对你来说，这是短暂休息的好时机，因为在这期间其它贡献者基本不会参与贡献。你不太可能得到PR或者Slack消息的回复。

<!-- It's helpful to set a number of hours per week you want to dedicate to K8s contribution, as it is very easy to get sucked into development activities. It's also important to take a break from contributing to K8s when your day job or life gets busy. There are downtimes in the K8s development community that usually coincide with the release code freeze period before a release cut. This would be a great time for you to take a break—because other contributors are less involved during this period, you are less likely to get a reply on your PRs or Slack messages. -->

让你的团队和经理了解你的新爱好是很好的事，因为分享你的知识可能会有所帮助。如果你的团队或者公司使用Kubernetes（或者任何容器技术），会因为你了解K8s的原理而受益。如果不，与你的团队分享Kubernetes中工业界的趋势、软件工程设计模式和架构设计决策也是非常棒的。考虑举办这些话题的每月讲座，介绍你在kubeCon或其它会议中学到的内容。

<!-- It's also good to let your team and manager at your job know about your new hobby since it can be beneficial to share your knowledge. If your team or company uses Kubernetes (or any container technologies), it will benefit from your knowing the internals of K8s. If not, it's still great to share trends in the industry, software engineering design patterns, and architectural design decisions in Kubernetes with your development team. Consider hosting monthly talks on those topics or presenting what you learn at KubeCon and other conferences. -->


## 总结
<!-- ## Summary -->

为K8s做贡献是我最有意义的经历之一，编程以前只是一份工作，但是现在它也是一个爱好。如果你工作不是特别苛刻，我强烈建议你试一试。

<!-- Contributing to K8s is one of the most rewarding experiences I've ever had. Programming used to be just a job, but now it's also a hobby. If you have a day job that isn't too demanding, I highly recommend trying it out. -->

## 资源

<!-- ## Resources -->
* [Kubernetes Slack](http://slack.k8s.io)（邀请自己）
* [贡献者指南](https://github.com/kubernetes/community/tree/master/contributors/guide)
* [贡献者备忘录](https://github.com/kubernetes/community/blob/master/contributors/guide/contributor-cheatsheet/README.md)
* [社区成员角色](https://github.com/kubernetes/community/blob/master/community-membership.md)
* IBM的[Kubernetes教程](https://developer.ibm.com/series/kubernetes-learning-path/)
* [Ultimate Go编程](https://www.oreilly.com/library/view/ultimate-go-programming/9780135261651/)视频系列
* [企业中的Kubernetes](https://learning.oreilly.com/library/view/kuberentes-in-the/9781492043270/)，由Brad Topol、Jake Kitchener和 Michael Elder

<!-- * Kubernetes Slack (invite yourself!)
* Contributor guide
* Contributor cheatsheet
* Community membership roles
* Kubernetes tutorials from IBM
* Ultimate Go Programming video series
* Kubernetes in the Enterprise by Brad Topol, Jake Kitchener, and Michael Elder -->
