---
order: 120
title: How to install bnd
layout: default
---

bnd is not a single product, it is a library (bndlib) used in many different software build environments. It runs inside Maven, ant, gradle, Eclipse, sbt, and maybe one day in Intellij. To install bnd, you will have to install these tools. 

bnd不是一个单独的产品，它是一个在不同软件构建环境中使用的库（bndlib）。它可以运行在Maven，Ant，Gradle，Eclipse，sbt当中，或者未来的某一天你也可以在IntelliJ看到它。你需要安装以下的工具去使用bnd。

<div>
<ul class="property-index">

{% for tool in site.tools %}<li><a href="{{ tool.url | prepend: site.github.url }}">{{tool.title}}</a> {{tool.summary}}</li>
{% endfor %}

</ul>
</div>

## Command Line 命令行
That said, there is also a command line version of bnd, providing an easy way to try out its many features. You can install bnd through [jpm][1]. You first have to install jpm, fortunately, this is well documented at [jpm install][1]. Installing jpm has the other advantage that it provides some useful commands to work with a repository that has all of maven central. 

事实上，还有一个命令行版本的bnd，它提供了一个简单的方法可以让你去尝试许多的功能。你可以通过jpm来安装bnd。首先，你需要安装 jpm，幸运的是，已经有了一篇好的文档来教你如何安装jpm了。安装jpm也有一些其他的好处，比如它提供了一些有用的命令来处理maven中心仓库。

With jpm installed, you can now install the latest version of bnd as follows:

当你安装完jpm，你就可以通过以下命令来安装bnd的最新版本。

	$ sudo jpm install bnd@*
	$ bnd version
	2.4.0.201406271227
	$

## Libraries 库
The binaries are available on [Cloudbees][4]. The latest version can be found at:

这个包现在被放在了Cloudbees上。你可以在下面的链接找到它的最新版。

	https://bndtools.ci.cloudbees.com/job/bnd.master/lastSuccessfulBuild/artifact/

However, Cloudbees currently also contains the previously released libraries.

与此同时，Cloudbees也包括之前的发布版。
	
If you are a maven user, you can find many version in central. The coordinates are:

如果你是一个maven用户，你可以在中央库中找到许多的版本。代码如下：

		<dependency>
			<groupId>biz.aQute.bnd</groupId>
			<artifactId>biz.aQute.bndlib</artifactId>
			<version>....</version>
		</dependency>


## Source Code 源代码
bnd is maintained at [github][3]. If you want to change the code, just clone it and modify it. In general we accept pull requests, and often even highly appreciate them.

我们现在在这个github库去维护bnd。如果你想去改其中的代码，克隆然后修改就可以了。一般来说，我们是接受pull request并且高度赞赏他们。

## Manual 手册
The manual is also on [github][5]. If you see an improvement, do not hesitate to clone the repo and create a pull request. Improvements are bug corrections but we also like short articles about how to do do something with bnd.

这个手册也被放到了github上。如果你对它有什么改进的想法，请不要犹豫，克隆这个库，然后发个Pull request过来。改进可以是修改Bug，同时我们也欢迎任何关于bnd的小文章。

## Communication Settings 连接设置

If you're behind a firewall that requires proxies or you use repositories that require authentication see [-connection-settings].

如果你处于一个防火墙之下且需要代理，或是你使用那些需要认证的仓库，请见-connection-settings.

[1]: http://www.jpm4j.org
[2]: http://jpm4j.org/#!/md/install
[3]: https://github.com/bndtools/bnd
[4]: https://bndtools.ci.cloudbees.com/job/bnd.master/lastSuccessfulBuild/artifact/
[5]: https://github.com/bndtools/bnd.manual
[6]: /instructions/conditionalpackage.html
[-connection-settings]: /instructions/connection-settings
