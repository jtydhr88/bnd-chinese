---
title: Guided Tour
layout: default
---

The purpose of this com.acme.prime is to show build administrators how to setup _workspaces_ and what features bndlib provides to automate common tasks. This section is not intended to be used by people wanting to learn OSGi, please use a bndtools tutorial for this. 

这个com.acme.prime项目的目的是用来显示构建管理员如何设置 workspaces 以及bndlib提供的一些自动执行常见任务的功能。本节不推荐给那些希望学习OSGi的人使用，请使用bndtools教程。

This section will go through the process of creating a workspace and a few projects while explaining what functions are useful in each phase. It will remain at a rather high level to keep the text flowing, details can be found in the different reference sections.

本节将介绍如何去创建一个workspace以及相关几个项目的过程，同时说明在每个阶段中哪些函数是很有用的。我们会尽量使它的流畅性保持在一定程度上，而其中的详细信息你可以在其他的参考章节找到。

Since this section provides a tool independent view of bndlib, we use the [bnd command line application](120-install.html) to demonstrate the features. Though this is an excellent way to show the low level functionality (the porcelain in git terms), it is not the normal way bndlib is used. In general, a build tool like gradle, ant, make, or maven drives this process from the command line an IDE like bndtools handles the user interaction. So please, please, do not take this com.acme.prime as a guide how to create a build. However handy bnd is, it falls far short of a real build tool like for example gradle.

由于本节我们提供了一个独立的bndlib的工具视图，我们将使用bnd command line application来演示其中的特性。虽说这是一个很好的方式来显示底层的功能（the porcelain in git terms），但希望你能明白的是，它并不是使用bndlib的正常的方式。一般来说，一个构建工具，比如radle, ant, make, 或是 maven，他们会从命令行推动着这个过程，同时一个IDE，比如Bndtools则会去处理用户交互行为。所以，请万千注意，不用去将com.acme.prime作为指导去创建一个构建行为。事实上，它本来也远远没有达到一个真正的构建工具的水平，如gradle。

## Workspace

A workspace is a directory with the following mandatory files/directories:

workspace是一个必须具有以下目录结构的文件夹：

	./
		cnf/
			build.bnd
			ext/

That's all! So let's create one:

现在让我们来创建一个：

	$ bnd add workspace com.acme.prime
	$ cd com.acme.prime
	$ ls
	cnf

The `cnf` directory is the 'magic' directory that makes a directory a workspace, just like the `.git` directory does for git. 

cnf 是一个具有“魔法”的目录，因为它能将一个目录变成了一个workspace，好比.git 目录对于git一样。

In the `cnf` directory you find the following files:

在cnf目录中，你能找到如下几个文件：

	$ ls cnf
	bnd.bnd         build.bnd       ext

The `build.bnd` is part of the magic to make something a workspace, it contains *your* workspace properties. The `ext` directory contains more properties. The `bnd.bnd` is the last piece of magic, it makes bndlib recognize the `cnf` directory as a project.

build.bnd是“魔法”的第一部分，它可以让一个普通的文件夹变成workspace，它包含了你的workspace的配置文件。ext文件夹则包含了更多的配置文件。bnd.bnd是魔法的最后一部分，它可以让bndlib将cnf文件夹识别为一个项目。

### Naming 命名

Why such a long name for the workspace? Wouldn't just `tour` suffice? Well, glad you asked. If you work with bnd(tools) for some time you will find that you will get many different workspaces since a workspace is another level of _modularity_. You can see a workspace as a _cohesive_ set of bundles; just like any module it can import and export. Just like any other module, it imports and exports the thing it encapsulates. For example, a bundle imports and exports packages. In case of the workspace this is the _bundle_. A workspace imports and exports bundles through repositories. 

为什么要为一个workspace起那么长的名字？为什么不仅仅是个前缀？是的，很高兴你能这么问。如果你使用bnd(tools)一段时间，你会发现你会用到许多不同的workspaces，因为一个workspace是另一个层次的 模块化。您可以将一个workspace视为一组相关的bundles;就像任何一个模块一样，它都可以import或是export；或是像其他的模块，它会imports或是export那些它封装好了的东西。举个例子，一个bundle可以imports或是exports 包(packages)。这时workspace就是一个bundle，它可以通过repositories去import或export bundles。

A good module has cohesion; this means that its constituents have a rather strong relation. Since they also tend to come from the same organization they will have very similar bundle symbolic names. Since some of these bundles will escape in the wild it is always beneficial if you can quickly find the source of that bundle. Therefore naming the workspace with the shared prefix of the bundle symbolic names of its constituents is highly recommended.

一个好的module应该是有凝聚力的; 这意味着其中每个成员都具有相当强的关系。由于他们可能于来自同一个组织，他们最好有非常相似的bundle符号名称。Since some of these bundles will escape in the wild，如果你可以快速找到这个bundle来自何处，那该多好啊。因此，我们强烈建议使用bundle的共享前缀来为工作空间的命名。

That said, bndlib does not enforce this guideline in any way, unlike project names. You can name your workspace `foo` if you want to.

不过，bndlib不以任何方式强制执行这个标准，除非是project名称。你可以为你的workspace起任何名字，比如foo。

### Properties 

A workspace so created is quite empty. However, if we look in the `cnf` directory then we can see the `build.bnd` file. This file  is 100% for you ... Any property, instruction, or header specified in here is available in anything you build in this workspace; all other bnd files will inherit everything from this properties file. In this file should add for example the headers Bundle-Vendor and Bundle-Copyright. However, using the macro language we can also add custom variables and macros that are useful across the workspace.

当一个workspace被创建出来的时候它里面什么都没有。但是如果我们去看看cnf文件夹，我们会发现build.bnd文件。这个文件就是为你准备的…此处指定的任何属性，指令或header specified都可以在此workspace中构建的任何内容中使用。所有其他的bnd文件都将会继承这个配置文件中的任何东西。比如，你应该在这个文件里面添加Bundle-Vendor和Bundle-Copyright。但是，使用宏语言（macro language），我们还可以添加在整个workspace中有用的自定义变量和宏。

## Plugins

And important aspect of the workspace is that it hosts _plugins_. A plugin is an extension to bndlib that gets loaded when the workspace is opened. Plugins provide a lot of different functions in bndlib. You can see the currently loaded plugins with bnd:

对于workspace来说，一个重要的方面是托管插件(plugins)。Plugin是bndlib的扩展，它在workspace打开时加载。在bndlib中，插件可以提供很多不同的功能。你看到bnd是如何加载plugins的。

	$ bnd plugins
	000 Workspace [com.acme.prime]
	001 java.util.concurrent.ThreadPoolExecutor@a4102b8[Running, ...]
	002 java.util.Random@11dc3715
	003 Maven [m2=...]
	004 Settings[/Users/aqute/.bnd/settings.json]
	005 bnd-cache
	006 ResourceRepositoryImpl [... ]
	007 aQute.bnd.osgi.Processor

The plugins you see are the built-in plugins of bnd itself, they always are available. However, the purpose of plugins is to extend the base functionality. As almost everything, the set of external plugins is managed through an instruction, which is a property in the Workspace.

你看到的这些插件是bnd自己构建出来的，通常来说他们都是可用的。但是插件的目的是为了拓展基础功能。作为几乎所有内容，外部插件集是通过指令管理的，其中的property被放进了Workspace中。

Before bndlib reads the `build.bnd` file to read the workspace properties, it first reads all the files with a `.bnd` extension in the `cnf/ext` folder. The purpose of this folder is to manage setups for plugins. We can add additional plugins by their name. You can see a list of built-in plugins with the add plugin command:

在bndlib读取build.bnd文件以获取workspace properties之前，它首先读取cnf/ext文件夹中所有以.bnd为拓展名文件。这个文件夹的目的是管理plugins的设置。

	$ bnd add plugin
	Type                           Description
	ant                            aQute.bnd.plugin.ant.AntPlugin
	blueprint                      aQute.lib.spring.SpringXMLType
	eclipse                        aQute.bnd.plugin.eclipse.EclipsePlugin
	filerepo                       aQute.lib.deployer.FileRepo
	git                            aQute.bnd.plugin.git.GitPlugin
	gradle                         aQute.bnd.plugin.gradle.GradlePlugin
	...

An interesting plugin is the Eclipse plugin that will prepare any projects for Eclipse. We could also add the git plugin that will make sure the proper .gitignore files are in place.

Eclipse plugin是一个有趣的plugin，它可以为eclipse 准备任何项目。我们也可以添加git插件去添加.gitignore文件等。

	$ bnd add plugin eclipse
	$ bnd add plugin git
	$ bnd plugins
	...
	007 aQute.bnd.osgi.Processor
	008 EclipsePlugin
	009 GitPlugin
	
Since you likely need to maintain this build it is good to know how this is stored. If you look in the `cnf/ext` directory you should see an `eclipse.bnd` and a `git.bnd` file:

如果你知道这些是如何存储的，你可以更好的去维护他们。在cnf/ext文件夹，你能看到eclipse.bnd 和git.bnd文件。

	$ ls cnf/ext
	eclipse.bnd			git.bnd

The `eclipse.bnd` file contains the following:

eclipse.bnd文件包含了以下内容：

	$ more cnf/ext/eclipse.bnd
	#
	# Plugin eclipse setup
	#
	-plugin.eclipse = aQute.bnd.plugin.eclipse.EclipsePlugin

So how does this work? When the workspace is opened bndlib will first read all the bnd files in the `cnf/ext` directory in alphabetical order. After that, it will read the `build.bnd` file. The idea of the `cnf/ext` files is that they should not be touched by you, the `build.bnd` file is, however, all yours. You can override any previous setting in the `build.bnd` file since it is read last. 

它是如何工作的呢？当一个workspace被打开时，bndlib首先会去以字母排序读取所有的cnf/ext文件夹下的bnd文件。然后，他会读取build.bnd文件。cnf／ext的设计理念是他不应该被你去修改，你应该去修改build.bnd文件。但是，因为它是最后一个被读取的，所以你在build.bnd文件修改的任何属性都会覆盖之前的设置。

As you can see, the `eclipse.bnd` file defines the property `-plugin.eclipse`. In most cases that a value should be settable in different places, bndlib uses _merged properties_. When bndlib loads the plugins, it actually gets the property `-plugin`, merged with any other property that has a key that starts with `-plugin.` (ordered alphabetically). This allows you to add to a merged property anywhere in the many places in bndlib where you can set properties.

正如你所看到的，eclipse.bnd文件定义了属性-plugin.eclipse。大多数情况这个值可以在不同的地方去设置，bndlib会去合并这些属性。当bndlib加载plugins时，它实际上去获取属性 -plugin，合并所有以-plugin开始的属性（以字母排序）。这意味着它允许你在bndlib的很多地方去添加一个属性。

So now we can try to build the workspace:

现在我们可以试着去构建一下workspace：

	$ bnd build
	No Projects

Which makes some sense ...

明白了吗？

## Project

An empty workspace is not so useful, let's add a project. 

一个空的workspace是没什么用处的，所以让我们添加一个project。

	$ bnd add project com.acme.prime.hello
	$ ls -a com.acme.prime.hello/
	.               .gitignore      bin_test        src
	..              .project        bnd.bnd         test
	.classpath      bin             generated

This classic layout defines separate source folders for the main code and the test code. The `generated` directory is a temporary directory, it contains the artifacts produced by this build. 

这是一个典型的项目结构，它会去定义一些主代码和测试代码的源文件夹。而generated文件夹则是一个临时文件夹，里面包含由此构建生成的文件。

## Setup




## Changing the Layout
This is a classical Eclipse layout, with a separate `src` and `test` folder. However, this is not baked into bndlib, it is possible to, for example, use the maven layout with the `src/main/java`, `src/test/java`, and `target` directories. We can try this out with the maven plugin.

它是一个典型的Eclipse项目结构，其中包含了src和test文件夹。但是，bndlib是不会限制你去修改项目结构的，例如，我们可以试着去使用maven插件，去使用maven的项目结构 src/main/java, src/test/java和target。

	$ bnd add plugin maven
	$ more cnf/ext/maven.bnd
	-plugin.maven = aQute.bnd.plugin.maven.MavenPlugin

	-outputmask = ${@bsn}-${versionmask;===S;${@version}}.jar
	
	src=src/main/java
	bin=target/classes
	testsrc=src/test/java
	testbin=target/test-classes
	target-dir=target

The maven plugin adds a number of properties that are recognized by bndlib and used appropriately. 

maven插件添加了许多可以被bndlib识别并使用的属性。

