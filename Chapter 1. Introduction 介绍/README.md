Chapter 1. Introduction 介绍
========================

##License 协议

Activiti 基于 [Apache V2](http://www.apache.org/licenses/LICENSE-2.0.txt) 协议

##Download 下载

[http://activiti.org/download.html](http://activiti.org/download.html)

##Sources 源码

发布包里包含大部分的已经打好 jar 包的源码。如果想找到并构建完整的源码库，请参考 wiki [构建发布包](http://docs.codehaus.org/display/ACT/Developers+Guide#DevelopersGuide-Buildingthedistribution)

##Required software 需要的软件

###JDK 6+

JDK 需要 6 或者以上的版本 。在 [Oracle Java SE 下载页面](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 进行下载，安装完成后 执行 `java -version` 命令行来检验是否成功。 

### Eclipse Indigo and Juno

下载 [eclipse](http://www.eclipse.org/downloads/) 进行安装，版本选  Indigo 或者 Juno。详见 [Chapter 12. Eclipse Designer-安装一节](https://github.com/waylau/activiti-5.x-user-guide/blob/master/Chapter%2012.%20Eclipse%20Designer/Installation%20%E5%AE%89%E8%A3%85.md)

##Reporting problems 报告问题

任何一个自觉的开发者都应该看看 [如何聪明的提出问题](http://www.catb.org/~esr/faqs/smart-questions.html)。
看完之后，你可以在[用户论坛](http://forums.activiti.org/en/viewforum.php?f=3)上进行提问和评论， 或者在[JIRA问题跟踪系统](http://jira.codehaus.org/browse/ACT) 中创建问题。

*注意*

*虽然 Activiti 已经托管在 GitHub 上了，但是问题不应该提交到GitHub 的问题跟踪系统上。如果你想报告一个问题， 不要创建一个GitHub 的问题，而是应该使用 [JIRA问题跟踪系统](http://jira.codehaus.org/browse/ACT)*

*译者注：如果对本翻译有疑问，可以在[https://github.com/waylau/activiti-5.x-user-guide/issues](https://github.com/waylau/activiti-5.x-user-guide/issues) 提问。*

##Experimental features 试验性功能

标记为 [EXPERIMENTAL] 的章节表示功能尚未稳定。

所有包名中包含 .impl. 的类都是内部实现类，都是不保证稳定的。 不过，如果用户指南把哪些类列为配置项，那么它们可以认为是稳定的。

##Internal implementation classes 内部实现类

在 jar 包中，所有包名中包含.impl.（比如：org.activiti.engine.impl.pvm.delegate）的类都是实
现类， 它们应该被视为流程引擎内部的类。对于这些类和接口都不能够保证其稳定性。
