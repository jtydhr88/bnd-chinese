---
order: 140
title: Best practices
layout: default
---

## Workspaces

The idea of a workspace is that it is cohesive. It contains a set of shared bundles and some of those bundles are exported to a repository. Other bundles are imported from repositories. This is the basic model of modularity.

worksapce的设计理念是它应该是有凝聚力的。它包含一组可被共享的bundles，其中一些bundle是可以导出到repository。而其他bundle是从repository导入的。这是模块化的基本模型。


This strongly implies that you should NOT share cnf between workspace, just like you should not share private fields from classes nor private classes from packages. So the idea is that your organizations has a number of workspaces that contain the code that has a strong relation. For example a product, your company service APIs, or shared base libraries. Since all these projects are in one workspace you get a lot of ease of use like refactoring and immediate feedback.

这意味着你不应该在workspace之间共享cnf，就像你不应该在类中共享私有字段，也不应该在包中共享私有类。所以例如，您的组织有许多工作空间包含具有强关系的代码。例如产品，您的公司服务API或共享库。由于所有这些项目都在一个工作区，你得到很多便利，如重构和即时反馈。

That said, there is often a desire to have some of the information, like the repositories in a shared place because you do not want to maintain it multiple times. Since bnd is completely based on inherited properties this is not that hard. The top of the properties are coming from the cnf/build.bnd file. Since this is a bnd file, you can actually include another bnd file there. Since this can be included via a URL, you can refer it to a file on your git repository. For convenience, you could make it refer to master but a better way is to make it refer to an actual commit. The reason is that if you checkout your project 5 years from now, it is unlikely that your build will be ok with the latest version of the included bnd file.

也就是说，经常有一个愿望是，你不想多次维护有一些被放在一个公共区域的repo的信息。因为bnd完全基于继承的属性，所以这不是那么难做到。property的顶部来自cnf / build.bnd文件。因为这是一个bnd文件，实际上你可以包括另一个bnd文件。它可以通过一个URL，比如你可以将它引用到你的git存储库上的一个文件。为了方便起见，你可以让它引用master，但更好的方法是让它引用一个实际提交。原因是如果你切换到一个5年前的版本，你的构建是不可能与被包含的bnd文件的最新版本保持一致。


	build.bnd:

	-include: https://examplegit.com/foo/bar/master/shared/shared.bnd

A more defined way to handle this long term versioning problem is to use git modules. They have  a bad name but as far as I understand their bad name is because people don’t like that they are not automatically updated, which is exactly what you want when you want to build your project 5 years from now. Git modules require an explicit command to upgrade it to the latest or another version. The parent git repository stores the commit at which it is linked. With git modules you could make a subdirectory in cnf and then include a shared file from there:

我们可以使用git模块来处理这个长期版本化问题。他们有一个难听的名字，但据我所知，他们这个难听名字是因为人们不喜欢他们不会自动更新，这是你想要的，当你想要建立你的项目5年后。 Git模块需要一个明确的命令将其升级到最新或另一个版本。父git repo则存储它提交的链接。使用git模块，您可以在cnf中创建一个子目录，然后从中包含一个共享文件：

	cnf
		shared/
			.git/
			shared.bnd
		build.bnd

	build.bnd:
	-include shared/shared.bnd

So to summarize, share workspaces, not projects. Use continuous integration that publishes your bundles to a repository that is shared with all. Share one bnd file with a git module.

简而言之，共享workspaces，而不是projects，使用持续集成将捆绑包发布到与所有共享的repo。通过git模块共享一个bnd文件。
