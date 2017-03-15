---
# 5 Guided Tour Workspace & Project导览
---


Since bnd is by design headless, the best way to get start is to use one of the IDEs like [bndtools][1]. They have tutorials and an IDE is a more pleasant place than a command line. So if you just want to learn OSGi, please go away, this chapter is not for you! bndlib relates to OSGi like the ASM byte code manipulation library relates to Java. Sometimes incredibly useful but in general something you do not want to touch, and obviously not the way to learn Java.

由于bnd没有图形界面，如果想要学习它，最好从学习它的IDE入手，比如bndtools 。因为IDE有教程，会比使用命令行更加方便。如果你只是想学习OSGi，这一章应该不适合你！因为bndlib与OSGi有关，就像ASM字节码操作库与Java相关一样，虽然有时ASM很有用，但是你并不会想要通过学习ASM去学习Java。

Assuming we're now only left with the blue collar workers of our industry: people that need to maintain builds or that must do JAR engineering. First a word of warning, the fact that bndlib provides a function is in no way an advertisement to use that function. bndlib grew up together with the OSGi specifications and has been used to build the Reference Implementations (RI) and Test Compatibility Kits (TCK). Though this has a lot of benefits for you, the disadvantage is that it also has to support all the bad parts of the specifications, and even sometimes must be able to create erroneous situations so we could create test cases. And we also had to handle the situations caused by the mess of non-modular bundles out there.

接下来，我们就假定继续留下来阅读本章节的是我们产业中的蓝领工人：那些需要维持软件的构建或者需要做JAR工程的人们。再次提醒，bndlib没有提供那些供广告人使用的功能。bndlib是与OSGi规范共同发展起来的，并被用于构建参考实现（RI）和测试兼容性套件（TCK）。所以它不仅支持OSGi规范中好的部分，也支持OSGi规范中所有坏的部分。 甚至有时它必须支持创建错误的示例来让我们创建测试用例。同时，我们也需要处理由那些非模块化JAR包造成的混乱。

So to make it crystal clear: the fact that a function is in bndlib does not mean it is intended to be used. This section contains a whole bunch of things you wish you never had to touch, and if you do OSGi properly, you will only see a tiny fraction of bndlib. That said, when the unprepared JARs hit the OSGi framework, it is nice to have bndlib as backup.

事实上，一个函数放在bndlib并不意味着它会被使用，bndlib中包含了大量你从未接触过的东西。通过OSGi，你只会看到bndlib的冰山一角。也就是说，当你需要把普通的jar包放到OSGi框架中，bndlib可以提供很好的支持。 

This tour uses the bnd command line bnd to demonstrate much of the inner details of bndlib.

此导览使用bnd命令行演示了bndlib中的大部分内部细节。 

bnd became the Swiss army knife to manipulate JARs and gained a lot of function you _should never attempt to use_. However, this chapter will give you an overview of what is in bnd.

bnd已经成为了操控JARs的利刃，并且拥有许多你永远都不会尝试去使用的功能。但是，本章将为您概述bnd中的所有内容。

bnd has lots of commands to try out aspects of bndlib. The [install section](120-install.md) shows how to install bnd.

bnd有针对bndlib各个方面的命令。在[安装部分](120-install.md)介绍了如何安装bnd。

##5.1 Wrapping 打包

A common use of bnd is to wrap a JAR file; there are still too many files out there that have no OSGi metadata.This lack of metadata makes them kind of useless as a bundle since it will be fully isolated from the other bundles. In OSGi, you must explicitly indicate what packages should be imported and what packages should be exported, by default everything is private. Though this can be quite frustrating sometimes, a rather strong argument can be made for the long term benefits of this strategy.

bnd的一个常见用途是打包JAR文件，因为仍然有很多的文件没有OSGi元数据。元数据的缺乏使它们不能和其他的bundle联系起来，成为了无用的bundle。在OSGi中，必须明确指出应该导入哪些包以及应导出哪些包，默认情况下，所有包都是私有的。虽然这有时可能相当令人沮丧，但是据论证这种策略是有长远好处的。

The OSGi metadata is stored in the _manifest_ of the JAR file. The manifest is a like a properties file stored in `META-INF/MANIFEST.MF` in the JAR file. The manifest format is [specified by Sun/Oracle][4]. Though the manifest is a text file and human readable it has a number of quirks that makes it hard to edit for humans. So even though it is text, the intention of the manifest was never to be done by hand, bnd like tools were always the intention. Tools can provide many useful defaults, handle expansions, and verify consistency,

OSGi元数据存储在JAR文件的清单中。清单类似于存储在META-INF/MANIFEST.MFJAR文件中的属性文件。清单格式由Sun / Oracle指定。虽然清单是人类可读的文本文件，但是它有一些怪癖，使得很难进行人为编辑。因此，即使清单是文本，但它也不是用来手动编辑的，我们更倾向于使用像bnd这样的工具。工具可以提供许多有用的默认值，并且可以处理扩展和验证一致性，

So let's use bnd to wrap the [javax.activation.jar][2] file even though it is already wrapped for us in the [Enterprise Bundle Repository][3]. Let's download it:

现在让我们使用bnd来打包javax.activation.jar文件吧，即使它已经在我们的企业bundle库中打包好了。首先让我们下载它：

	$ mkdir jar
	$ bnd copy http://repo1.maven.org/maven2/javax/activation/activation/1.1.1/activation-1.1.1.jar \
		jar/javax.activation-1.1.1.jar
	
bnd provides a convenience function to print out the contents of a JAR. The --uses option shows the packages and what other packages they use. 

bnd提供了一个很方便的功能来打印JAR的内容。-uses选项显示包及其依赖的其他包。

	$ bnd print --uses jar/javax.activation-1.1.1.jar 
	[USES]
	com.sun.activation.registries            []
	com.sun.activation.viewers               [javax.activation]
	javax.activation                         [com.sun.activation.registries]

##5.2 Simplistic 简单的应用

In the simplistic case our goal is to provide a manifest that makes all the packages in the source JAR available to the other bundles (exports) and that imports anything the classes in the JAR require from the external world (imports). bnd is controlled by a _bnd_ file. This file is a standard Java properties file. In this properties file we can provide headers, instructions, and macros to bnd. In general, a header starts with an upper case, and an instruction with a minus sign ('-'). The format of the headers and instructions is defined by the OSGi specifications. The headers specifically try to follow the structure of an OSGi header. Since exports are defined with the `Export-Package` header and imports are defined by the `Import-Package` header we can create a bnd file that looks very much like the desired manifest, except for the instruction that defines our class path. We could also use the URL directly on the classpath but this would download the JAR everytime we ran bnd. So `javax.activation.bnd` should look like:

在简单的情况下，我们的目标是提供一个清单，使源JAR包中的所有packages对其他bundles可用（exports），并且导入JAR包中类需要的所有packages（imports）。bnd由bnd文件控制,此文件是标准的Java属性文件。在这个属性文件中，我们可以向bnd提供header,instruction和macros. 通常，header以大写字母开头，instruction以“-”号开头。header和instruction的格式是由OSGi规范定义，所以我们的headers也遵循OSGi header的结构。因为exports是用Export-Package header定义的，exports是由Import-Package header 定义的，所以我们可以创建一个除了定义类路径的instruction,其他方面都非常像我们所需清单的bnd文件。我们还可以直接在类路径上使用URL，但是这将会导致我们每次运行bnd时都会重新下载这个jar包。综上javax.activation.bnd应该如下所示：

	-classpath:		jar/javax.activation-1.1.1.jar
	Export-Package: *

##5.3 Properties Format 属性格式
The casing of the headers is important, even though OSGi headers are case insensitive. That is Export-Package is not the same for bnd as Export-package.

Headers的大小写很重要，虽然OSGi headers不区分大小写。也就是说Export-Package和Export-package在bnd中是不同的。

The properties format we use in bnd is darned flexibile. A header is recognized when you start the line with a header name (a word without spaces) and then:

我们在bnd中使用的属性格式十分灵活，当您使用不含空格的单词开始一行时，会被识别为header name。具体如下：

* Any white space (space, tab)
* A  colon (':'), white space around the colon is ignored
* An equal sign ('='), white space around the equal sign is ignored  
 

* 任何空白（空格，制表符）
* 冒号（'：'），冒号周围的空格被忽略
* 等号（'='），等号周围的空格被忽略
 
Though lines can be as long as you want, it is better for your brains to keep them short. You can break a line with a backslash ('\\') followed by a newline ('\n'). Be careful, the newline *must* follow the backslash or you will get whatever comes after the backslash in the header. Some examples that all set the variable `foo` to 3:

虽然一行可以你想写多长就写多长，但是最好时控制它们在合适的长度以便于阅读。你可以用反斜杠（'\'）和换行符（'\ n'）分隔一行。但是需要注意的是，新的一行必须要紧接着反斜杠，否则你将得到header中反斜杠之后的任何内容。下面是设置变量foo为3的例子：

	foo				= 	3
	foo:3
	foo 3
	foo \
	3

Since bnd files are property files, you cannot repeat a property. Later properties will completely overwrite earlier ones and there is no order between properties.

由于bnd文件是属性文件，因此不能写重复属性。否则后面的属性将完全覆盖早先的属性，并且属性之间没有顺序。

##5.4 Running bnd 运行bnd

We can now wrap the source JAR with the following command:

我们现在可以使用以下命令打包源JAR包：

	$ bnd javax.activation.bnd

bnd has many commands, you can find out more about them with the`bnd help [sub]` command. In this case we rely on bnd detecting a bnd file and doing the right thing. bnd is also prepared to show the manifest of any bundle, so let's take a look at the bundle we just generated:

bnd有很多命令，你可以通过bnd help [sub]命令来获取它们。上面这种情况，我们是依靠bnd检测bnd文件并做相应动作。bnd也可以用来显示任何bundle的清单，让我们来看看我们刚刚生成的bundle清单文件：

	$ bnd javax.activation.jar
	[MANIFEST javax.activation]
	Bnd-LastModified                         1407229057278                           
	Bundle-ManifestVersion                   2                                       
	Bundle-Name                              javax.activation                   
	Bundle-SymbolicName                      javax.activation                   
	Bundle-Version                           0                                       
	Created-By                               1.8.0 (Oracle Corporation)              
	Export-Package                           com.sun.activation.registries;version="0.0.0",
	                                         com.sun.activation.viewers;uses:="javax.activation";version="0.0.0",
	                                         javax.activation;version="0.0.0"
	Manifest-Version                         1.0                                     
	Require-Capability                       osgi.ee;filter:="(&(osgi.ee=JavaSE)(version=1.4))"
	Tool                                     Bnd-2.4.0.201408050759                  

##5.5 Output 输出

When we create the JAR, we can also specify the output file. By default, bnd will store JAR under the same name as the bnd file, but then with a `.jar` extension. You can override the default from the command line:

当我们创建JAR时，我们还可以指定输出文件。默认情况下，bnd将以与bnd文件相同的名称存储JAR，然后加上.jar扩展名。通过命令行可以修改默认值：

	$ mkdir bundle
	$ bnd do --output bundle javax.activation.bnd
	$ ls bundle
	javax.activation.jar

You can either specify a directory or a file name. However, you can also specify the output in the bnd file itself:

你也可以在bnd文件中指定输出目录或文件：

	-output: 		bundle

##5.6 The Manifest 清单文件

It is clear that bnd has been busy. It detected 3 packages (com.sun.activation.registries, com.sun.activation.viewers, and javax.activation) and since we exported all, they appear in the manifest as exports. Observant readers will have noticed that we used _wildcards_ ('*'). The OSGi specifications are not so generous, every package must be explicitly listed so that the manifest contains all the information required to process a bundle. This, again, has many long term macro advantages. It's however bnd's goal in life to provide the convenience in this model and that is the reason that bnd supports wildcards in virtual any situation where they are usable. A wildcard requires a _domain_ to be effective, you can only select something out of a set, its domain. In the case of `Export-Package`, its domain is the _class path_. This class path is set by the `-classpath` instruction. In this case the activation jar.

很明显bnd在生成清单的过程中做了很多事，它检测到3个包（com.sun.activation.registries，com.sun.activation.viewers和javax.activation），由于我们导出了所有包，所以它们在manifest的exports中都显示出来了。细心的读者将会注意到我们在bnd中使用了通配符（'*'），但是OSGi规范并不支持这么做，在OSGi中每个导出包都必须明确列出，以便清单把处理一个bundle所需的全部信息包含进来。强调一下，OSGi这样做是有很多长期好处的。然而，这也是bnd存在的目的，为OSGi模型提供更加方便的支持，因此在任何bnd可以使用的情况下，都可以使用通配符。通配符是在一定域范围内有效的，您只能从集合中选择某个域。在Export-Package中，其域是类路径，这个类路径由-classpath instruction设置。

##5.7 Export Package 导出包

Currently our package is exported at version 0.0.0. This is not a very useful version, we therefore need to version the exported packages. Why? Well, this is the contract of modularity. To get the benefits of privacy (the privilege to change whenever we feel the need) comes the cost of compatibility for the parts we share. If you export a package, you intend it to be used by others. These others are an uncontrollable bunch that will use your _revision_ (the actual artifact with a given digest, i.e. the instance of the _program_) but will expect updates in the future with new functionality and bug fixes. 

目前我们的软件包以0.0.0版本导出， 这不是一个非常有用的版本，因此我们需要给导出的包一个特定的版本。为什么？因为，这就相当于模块化的合同。为了获得私有性（按照我们所需去改变特权），我们不得不付出部分的兼容性成本。如果你导出一个包，那就是希望这个包被别人所使用。但是别人的使用行为是不可控的，他们可能使用你程序的某个修订版本，同时还会希望这个程序更新一些新功能和修复一些bug.

Since a software _program_ is an ephemeral product that changes over time these others do not become dependent on the revision, nope, they become dependent on the _program_. This creates two problems. 

由于软件程序是一种随时间变化的临时产品，这些使用的人不会依赖于版本修订。他们只依赖于软件本身。这就产生两个问题：

* How do they make sure they do not get linked in runtime to a revision that is older than they they compiled against?
* How do they know if a revision is backward compatible?

* 它们如何确保它们在运行时不会链接到比他们编译的版本更旧的版本？
* 他们如何知道修订版本是否向后兼容？

The answer to these two questions is given to us by [semantic versioning][5], a prescriptive syntax for versions. Though semantic versions work for bundles, they really shine for packages. The basic idea is that when you compile against a version, this version is recorded in the OSGi metadata as an _range_ on the imported package. The floor of this range is the version in the revision use to compile against, the ceiling indicates the first version that would not be compatible anymore. For example, `[1.2,2)` indicates that any version that is larger or equal than 1.2 and less than 2 is compatible. Therefore, 1.5 would be compatible. Since OSGi supports version ranges on the Import-Package header it will be able to control what revisions get linked and how.

语义版本解决了这两个问题，它是版本的规范语法。虽然语义版本适用于bundle，但它们真的是为packages量身定做。基本思想是，当你针对版本进行编译时，此版本将作为导入包的范围记录在OSGi元数据中。这个范围的底部是修订版本中用于编译的版本，天花板表示不再兼容的第一个版本。例如，[1.2,2)指示大于或等于1.2且小于2的任何版本是兼容的。因此，1.5将是兼容的。由于OSGi支持导入包头上的版本范围，它将能够控制什么版本链接和如何链接。

Such a version scheme of course only works if you agree how to use the version syntax, this should explain the 'semantic' in semantic versioning. Therefore, the bnd/OSGi versions have a strictly defined syntax:

这样的版本方案当然只有在您同意如何使用版本语法时才有效，这应该解释语义版本化中的“语义”。因此，bnd / OSGi版本具有严格定义的语法：

* major – For breaking change
* minor – For changes that break the provider of the package but are backward compatible for the consumer of the package. See [semantic versions][5] for more detail.
* micro – Backward compatible changes, e.g. small bug fixes or documentation changes
* qualifier – Identifying the build 

* 主版本 - 用于重大变化
* 次版本 - 用于破坏包的提供程序但对包的使用者向后兼容的更改。有关更多详细信息，请参阅语义版本。
* 微 - 向后兼容的更改，例如小错误修复或文档更改
* qualifier - 标识构建

Long story, but it is at the heart of what we're trying to achieve: evolving large systems in a anti-fragile way.

说来话长，但它正是我们努力想要做到的核心：向强健的大型系统发展。

Back to the exported packages. We can add the version to the export-package by _decorating_ the Export-Package header: 

返回到导出包，我们可以通过装饰 Export-Package header来将版本添加到export-package ：
  
	-classpath:		jar/javax.activation-1.1.1.jar
	Export-Package: *;version=1.1.1

Let's take another look:

让我们再看一看：
	
	$ bnd javax.activation.bnd
	$ bnd javax.activation.jar
	[MANIFEST javax.activation]
	...
	Export-Package                           com.sun.activation.registries;version="1.1.1",
	                                         com.sun.activation.viewers;uses:="javax.activation";version="1.1.1",
	                                         javax.activation;version="1.1.1"
	...

Looking at the manifest we can see the _uses_ directive; this directive is the primary reason you should use bnd. To prevent wiring the wrong bundles together, the OSGi Framework must know which packages use what other packages in their API, this list is reflected in the uses constraints. It allows the OSGi Framework to handle multiple versions of the same package in one VM and not create Class Cast Exceptions ... Very powerful but it is impossible to maintain this list by hand and it would be prohibitively expensive in runtime.

在此清单中，我们可以看到uses指令; 这个指令是你应该使用bnd的主要原因。为了防止将错误的包连接在一起，OSGi框架必须知道哪些包在其API中使用了什么其他包，该列表反映在使用约束中。它允许OSGi框架在一个VM中处理同一个包的多个版本，而不是创建类转换异常...非常强大，但是我们不可能手动维护这个列表，这样太代价高昂了。

##5.8 Private Package 私有包

An exported package is a _commitment_ to any future users out there. This commitment will restrict your freedom to make changes since users can get pretty nasty if you make something that will not work with their existing software. In contrast, a package that is not exported is private; since no external user can have knowledge about this package we have complete freedom to do whatever we want in a future revision without getting hollering users on the phone.

导出包是对未来用户的承诺。这个承诺将限制你进行自由更改，因为如果你做的改动让他们无法使用现有的软件，会让用户感到十分讨厌。相反，未导出的包是私有的; 因为没有外部用户可以有关于这个包的知识，我们有完全自由，做任何我们想要的在未来的修订，而不会接到客户的抱怨电话。

In our example javax.activation JAR we should therefore not export the `com.sun.*` packages since they are actually private. The Private-Package header has the same syntax as the Export-Package header but will not export the packages, it will keep them hidden in the bundle. Unlike the Export-Package header, it is not necessary to decorate the packages with a version since nobody can depend on them.

在我们的示例javax.activation JAR中，我们不应该导出com.sun.*包，因为它们实际上是私有的。Private-Package标头具有与Export-Package header相同的语法，但不会导出包，它会将它们隐藏在包中。与Export-Package header不同，没有必要使用版本来装饰包，因为没有人可以依赖它们。

	-classpath:		jar/javax.activation-1.1.1.jar
	Export-Package: javax.activation;version=1.1.1
	Private-Package: com.sun.activation.*

	$ bnd javax.activation.bnd
	$ bnd javax.activation.jar
	[MANIFEST javax.activation]
	...
	Export-Package          javax.activation;version="1.1.1"
	Private-Package         com.sun.activation.registries,
	                        com.sun.activation.viewers
	...

The Private-Package has no meaning to OSGi, if you want to not have this header in your manifest then you can also use `-private-package` instead.

Private-Package对OSGi没有意义，如果你不想在你的清单中有这个header，那么你也可以使用-private-package。

##5.9 Bundle Version Bundle版本

The `Bundle-Version` header obviously provides the version. Default it will use version 0, so we should override it. We could just add the 1.1.1 version from our source JAR, however, this would make it impossible to to distinguish the different variations we might make over time. A good tool is to use the ${tstamp} macro for the _qualifier_. The qualifier is the last part of the version. A macro provides a textual replacement, bnd has hundreds of macros (including access to Javascript) and you can define your own. The ${tstamp} macro provides a time stamp.

显而易见，Bundle-Version是用来标明版本的，默认版本为0，所以我们应该覆盖它。我们可以从源JAR添加1.1.1版本，但是，我们将无法区分不同时间的相同版本。一个好的办法是是使用$ {tstamp} macro作为限定符。限定符是版本的最后一部分。macro提供了一个文本替换，bnd有数百个marco（包括访问Javascript），你可以定义自己的marco。这里的$ {tstamp} macro是用来提供时间戳的。

	-classpath:		jar/javax.activation-1.1.1.jar
	Export-Package: javax.activation;version=1.1.1
	Private-Package: com.sun.activation.*
	Bundle-Version:	1.1.1.${tstamp}

##5.10 DNRY or the Benefit of Macros DNRY和Macro的好处

The 'Do Not Repeat Yourself' mantra encodes one of the more important lessons of software engineering. You should always strive to define a 'thing' or a piece of knowledge in only one place. In our little example, we actually repeat ourselves with the version 1.1.1:

“不要重复自己”是软件工程中的最重要的经验之一。你应该努力只在一个地方定义一个“事物”或一块知识。在我们的小例子中，我们实际上重复自己的版本1.1.1：

* The classpath
* The export package
* The version
* The output

* 类路径
* 导出包
* 版本
* 输出


bnd has a macro processor on board that is a life saver if you are addicted to DNRY (we are). This macro processor has access to all properties in the bnd file, including headers and instructions. Therefore `${Bundle-Version}` refers to whatever value the Bundle Version is set to. However, in this case the Bundle Version also contains the time stamp in the qualifier, which we do not need in the other places. That is, we do not want the output file name to contain the qualifier. 

bnd有一个macro处理器，如果你沉迷于DNRY, 这个处理器将会成为你的救星。此处理器可访问bnd文件中的所有属性，包括header和instructor。因此，${Bundle-Version}指向的是Bundle Version设置的任何值。但是，在这种情况下，Bundle版本还包含时间戳限定符，我们在其他地方不需要时间戳。也就是说，我们不希望输出文件名包含限定符。

bnd also contains a large number of built-in macros that provide common utilities. The `${versionmask;mask;version}` is for example such a utility for picking out the parts of a version, bumping it, as well as normalizing it. Normalizing (making sure all parts are present, leading zeros removed, etc.) is crucial to keep things workable.

Bnd还包含大量提供常见实用程序的内置marco, 例如${versionmask;mask;version}用于筛选出版本的一部分来使它更加规范。规范化（确保所有部分都存在，前面的零被删除等）是保持事物可行的关键。

We can try out the macro from the command line, the bnd command has a `macro` sub command:

我们可以从命令行尝试macro，bnd命令有一个macro子命令：

	$ bnd macro "version;===;1.2.3.q" => 1.2.3
	$ bnd macro "version;+00;1.2.3.q" => 2.0.0
	$ bnd macro "version;+00;0"       => 1.0.0
	$ bnd macro "version;=;1.2.3.q"   => 1

To reuse the Bundle Version in the class path, the output, and the export package:  

在类路径，导出包和导出包中重用Bundle版本：

	-classpath:		jar/javax.activation-${versionmask;===;${Bundle-Version}}.jar

	Export-Package: javax.activation;version=${versionmask;===;${Bundle-Version}}
	Private-Package: com.sun.activation.*
	Bundle-Version:	1.1.1.${tstamp}
   
This of course looks awkward and hardly DNRY. A better solution is to create an intermediate variable. Variables are always lower case (if they started with an upper case, they would end up in the manifest). Variables can be referred to with the macro syntax of `${}`.

当然这看起来是一个很糟糕的DNRY。一个更好的解决方案是创建一个中间变量，这个变量总是小写（如果以大写开头，它们将在清单中结束），变量可以使用macro语法来引用${}。

	v:               1.1.1

	-classpath:		 jar/javax.activation-${v}.jar
	-output:		 bundle
	
	Bundle-Version:	 ${v}.${tstamp}
	Export-Package:  javax.activation;version=${v}
	Private-Package: com.sun.activation.*

##5.11 Description 描述

Maven Central is quickly moving towards a million revisions organized in 200.000 programs. It should be clear that we need to organize this huge pile of software. One way is to make sure you document your programs appropriately. It is highly recommended to have a short one paragraph description in each bundle so that sites like [jpm](https://jpm4j.org) can provide search facilities and useful presentation. Adding a description to our bundle is easy:

Maven Central正在快速地对200万个程序进行一百万次修订。需要明确的是，要组织这大一堆软件，其中一种方法是确保你的程序中有适当的文档。因此强烈建议在每个bundle中有一段简短的描述，使像jpm这样的网站可以提供搜索工具和有用的演示。向我们的bundle中添加描述很简单，如下：

	v:               1.1.1

	-classpath:		 jar/javax.activation-${v}.jar
	-output:		 bundle
	
	Bundle-Description: \
		A wrapped version of javax.activation ${v} JAR from Oracle. \
		This bundle is downloaded from maven and wrapped by bnd.
	Bundle-Version:	 ${v}.${tstamp}
	Export-Package:  javax.activation;version=${v}
	Private-Package: com.sun.activation.*

For longer descriptions you can continue on the next line with a backslash ('\') followed by a newline. Unlike the manifest, it does not has to start with a space though indenting the text for the next lines is usually a good practice. You cannot use newlines or other markup in the Bundle-Description.

对于更长的描述，您可以使用反斜杠（”\”）后跟换行符来开始下一行。与清单不同的是，它并不需要从一个空格开始，虽然缩进下一行的文本通常是一个好的做法。你不能在Bundle-Description中使用换行符或其他标记。

That said, a good Bundle-Description is a single paragraph with a few short sentences.

也就是说，一个好的Bundle-Description是一个有几个短句的简单段落。 

##5.12 Include Resources 包含资源

The Bundle-Description is an excellent way to provide a short description, but what about a longer description with examples of usage, caveats, configuration requirements, etc? It would be nice to include a `readme` file in the bundle that indexing sites could use. For example, take a look at [Xray](http://jpm4j.org/#!/p/sha/C9256D19F155D699ED0D18559BBB80441AC68FF7//0.0.0?tab=readme). You can achieve this effect by including a `readme.md` file in the root of your bundle.

Bundle-Description是一个提供简短描述的好方法，但是对于用例，注意事项，配置要求等更长的描述呢？如果在bundle中包含一个可以供索引站点使用的readme文件是非常棒的，例如Xray。 你可以通过在bundle根目录下包含一个readme.md文件来实现此效果。

In bnd it is possible to include any resource from anywhere in the file system (actually, any URL as well).

在bnd中，可以包括文件系统中任何地方的任何资源（实际上，任何URL也可以）。

	-includeresource: readme.md 

The resource is of course not always in the proper place, it is therefore possible to make the output name different from the file path:

资源当然不总是在合适的位置，因此可以让输出名称与文件路径不同：

	-includeresource: readme.md=doc/readme.md

In this case, the readme could benefit from the bnd macro support. For example, it could then contain the actual version of the artifact or use any of the other macros available. Preprocessing can be indicated by enclosing the clause with curly braces ('{' and '}').  

在这种情况下，readme文件可以从bnd macro支持中受益。例如，它可以包含artifact的实际版本或使用任何其他可用的marco。可以通过用大括号（'{'和'}'）包含子句来指示预处理。

	-includeresource: {readme.md=doc/readme.md}
	
Another option is to encode the text for the readme in the bnd file using the literal option (though this is not such a good idea for any decent readme file since it is hard to use newlines):

另一个选项是使用literal选项对bnd文件中的readme的文本进行编码（虽然这对于任何合适的readme文件并不是一个好主意，因为很难使用换行符）：

	-includeresource: readme.md;literal=${unescape;#JAF\nThis is the Java Activation Framework}

The `-includeresource` instruction is quite powerful, there are many more options to recurse directories, filter, etc. See [-includeresource](../macros/includeresource.html).

-includeresource instruction十分强大，有更多的选项来递归目录，过滤器等等。请参阅-includeresource。

##5.13 Import Package 导入包

So far, we've ignored the imported packages because the javax.activation JAR only depends on java.*; java.* packages are not imported, the OSGi Framework will always provide access to them. Let's add another JAR, the javax.mail jar that uses javax.activation. This JAR is already a bundle, which is really good. Except for this exercise, so we copy and strip the OSGi metadata 

到目前为止，我们忽略了导入包，因为javax.activation JAR只依赖于java.; java.包不需要导入，OSGi框架会始终提供对它们的访问。现在让我们添加另一个JAR，即使用javax.activation的javax.mail jar。这个JAR已经是一个bundle，这真是棒极了。做这个练习之前，我们首先需要下载这个jar包并移除它所有的OSGi清单元数据：

	$ bnd copy --strip http://repo1.maven.org/maven2/com/sun/mail/javax.mail/1.5.2/javax.mail-1.5.2.jar \
	   > jar/javax.mail-1.5.2.jar
	$ bnd print --uses jar/javax.mail-1.5.2.jar 
	[USES]
	com.sun.mail.auth          [javax.crypto, ... ]
	com.sun.mail.handlers      [javax.activation, ... ]
	com.sun.mail.iap           [com.sun.mail.util, ... ]
	com.sun.mail.imap          [com.sun.mail.imap.protocol, ...]
	com.sun.mail.imap.protocol [com.sun.mail.iap, ... ]
	com.sun.mail.pop3          [javax.mail, ... ]
	com.sun.mail.smtp          [com.sun.mail.util, ... ]
	com.sun.mail.util          [javax.mail,  ... ]
	com.sun.mail.util.logging  [javax.mail, ...]
	javax.mail                 [javax.mail.event, ... ]
	javax.mail.event           [javax.mail]
	javax.mail.internet        [javax.mail, ... ]
	javax.mail.search          [javax.mail.internet, javax.mail]
	javax.mail.util            [javax.mail.internet, javax.activation]

The javax.mail bundle leverages the Java Activation Framework (JAF) embedded in our previous javax.activation bundle. One of the most useful features in bnd is that it can analyze the class files to find the package dependencies and turn them into an Import-Package header. If bnd finds the exported packages with a version on the class path then it will automatically use the exported versions to calculate the import version range. That is, if the package is exported as 1.1.1, then bnd will import it in general as `[1.1,2)`.

javax.mail包利用嵌入在javax.activation包中的Java激活框架（JAF）。bnd中最有用的功能之一是它可以通过分析类文件来找到包依赖项并将其转换为Import-Package header。如果bnd发现导出的包在类路径上有一个版本，那么它将自动使用导出的版本来计算导入版本范围。也就是说，如果包导出为1.1.1，那么bnd将导入为[1.1,2)。

So lets wrap the javax.mail package, add to `javax.mail.bnd`:

所以让我们打包javax.mail包，添加以下内容到javax.mail.bnd：

	v:          1.5.2
	
	-classpath: \
		jar/javax.activation-1.1.1.jar, \
		jar/javax.mail-${v}.jar
	
	Bundle-Version: ${v}
	Bundle-Description: \
	  An OSGi wrapped version of the javax.mail library downloaded from maven.
	Export-Package: javax.mail.*
	Private-Package: com.sun.mail.*

bnd contains a special `-i/--impexp` option to print the imports and exports of a bundle. So lets make the bundle and see:

bnd可以用-i/--impexp选项来打印导入和导出包，接下来，让我们制作一个bundle来尝试一下：
 
	$ bnd javax.mail.bnd
	$ bnd print -i javax.mail.jar
	[IMPEXP]
	Import-Package
	  javax.activation                       
	  javax.crypto                           
	  javax.crypto.spec                      
	  javax.mail.event                       {version=[1.5,2)}
	  javax.mail.search                      {version=[1.5,2)}
	  javax.mail.util                        {version=[1.5,2)}
	  javax.net                              
	  javax.net.ssl                          
	  javax.security.auth.callback           
	  javax.security.auth.x500               
	  javax.security.sasl                    
	  javax.xml.transform                    
	  javax.xml.transform.stream             
	Export-Package
	  javax.mail                             {version=1.5}
	  javax.mail.event                       {version=1.5, imported-as=[1.5,2)}
	  javax.mail.internet                    {version=1.5}
	  javax.mail.search                      {version=1.5, imported-as=[1.5,2)}
	  javax.mail.util                        {version=1.5, imported-as=[1.5,2)}

First, we notice that there are quite a few imports that have no import range. This is not good but unfortunately Java's VM package versions are more or less absent and are clearly not semantically versioned. A correctly setup system ensures that the correct execution environment is used.

首先，我们注意到有相当多import包没有import范围。这样十分不好，但不幸的是Java的VM的包版本或多或少没有版本或者没有规范的语义版本。正确的设置系统可确保使用正确的执行环境。

However, we also have in this list javax.activation, for this package we do have an export, the problem is that we included the original JAR on the class path and not the bundle we generated.

然而，我们在这个列表中也有javax.activation，对于这个包，我们有一个导出，问题是我们在类路径中包括原始的JAR，而不是我们生成的bundle。
 
	v:          1.5.2
	
	-classpath: \
		bundle/javax.activation.jar, \
		jar/javax.mail-${v}.jar
	
	Bundle-Version: ${v}
	Bundle-Description: \
	  An OSGi wrapped version of the javax.mail library downloaded from maven.
	Export-Package: javax.mail.*
	Private-Package: com.sun.mail.*

And now build + print:

现在build + print：

	$ bnd javax.mail.bnd; bnd print -i javax.mail.jar
	[IMPEXP]
	Import-Package
	  javax.activation                       {version=[1.1,2)}
	  javax.crypto                           
	  ...

Yes! Now the javax.activation package is imported with a range of `[1.1,2)`. You now may wonder why not `[1.1.1,2)`? Well, the reason is that changes in the micro part of the version should not make a difference in API. If you include the micro part in the import range then it turns out that the overall system becomes very volatile, small changes become large quickly. Ignoring the micro part in the import range is like a bit of oil in the engine ... However, if you feel uncomfortable with this lubricant then it is possible to override this.

现在javax.activation包的导入范围变成了[1.1,2)。你现在可能想知道为什么不是[1.1.1,2)呢？原因是微版本的变化不应该让API产生变化。如果在导入范围中包括微版本，那么结果是整个系统变得非常不稳定，小变化迅速变大。在导入范围中忽略微版本，就像是发动机油，如果你对这种油感到不满意，那么你可以换掉它。

##5.14 Imports 导入

We got the Import-Package header in the manifest but we never specified it. The reason is that bnd has a default Import-Package set to '*'.  There is a very good reason for this, it works almost exactly as it should work. Any time you feel the desire to muck around with the imports you should realize that you're not doing the right thing. That said, it is the unfortunate truth that sometimes doing the least worst thing is the best option ...

我们在清单中获得了Import-Package header，但我们从未指定它, 因为bnd默认Import-Package为'*'。这样做的理由是它能够完成我们需要的功能。任何时候，你觉得想要import一些包，你应该意识到，你不是在做一件正确的事情。也就是说，有时做最少的不正确的事情是最好的选择...

Assume we want to provide a version range on some of the imported packages because these packages could also be provided by bundles, they do not have to come from the JVM. For this, we can add the following header that will decorate the javax.net.ssl package:

假设我们想对一些导入包提供一个版本范围，因为这些包也可以由bundle提供，它们不必来自JVM。为此，我们可以添加以下header来修饰javax.net.ssl包：

	Import-Package: javax.net.*;version=1.1, *

You can decorate any package, including packages specified with a wildcard. The domain of the Import-Package is all packaged that are referred to by any class inside the bundle. If there are for example imports 

您可以修饰任何包，包括用通配符指定的包。导入包的作用域是bundle中所有引用它的类。

##5.15 Remove Headers 删除Headers

The `removeheaders` header uses a _selector_ expression. A selector expression consists of a number of comma separated clauses. A clause contains a name (which can be wildcarded) and some optional instructions. A selector is always applied to a _domain_, in the case of `-removeheaders` the domain is all the headers in the manifest. The clauses in the selector are applied one by one in their declaration order to the domain. If a clause matches a member of the domain then the member is removed from the set, and the next member is again matched against the first clause. A suffix of ':i' indicates a case insensitive match.

Removeheaders header采用了选择表达式。选择表达式由多个逗号分隔的子句组成。子句包含一个名称（可以是通配符）和一些可选指令。选择器始终应用于域，在-removeheaders域是清单中的所有标头的情况下。选择器中的子句会按其声明顺序一个一个地应用于域。如果一个子句与域的成员匹配，则该成员从集合中删除，并且下一个成员再次与第一个子句匹配。'：i'的后缀表示不区分大小写的匹配。

This ordering is important because selector clauses can use negation. When a clause starts with an exclamation mark ('!') then a matching member is further ignored. Therefore, the following selector will never match the Bundle-Name, the first clause already threw any header that starts with `Bundle-` out:

这种排序很重要，因为selector子句可以使用否定。当子句以惊叹号（'！'）开头时，将忽略此匹配成员。因此，以下选择器将永远不匹配Bundle-Name，因为第一个子句已经忽略了任何以Bundle-开头的header：

	-removeheaders: !Bundle-*, Bundle-Name:i

The `Require-Capability` header ensures that the runtime has at least a Java 1.4 VM by requiring an OSGi execution environment. bnd calculates this requirement based on the compiler version used to compile the classes in the JAR. We can override this requirement or disable it with the `-noee=true` instruction.

Require-Capability header可确保OSGi运行环境中至少具有一个Java 1.4虚拟机。bnd利用JAR中类的编译器版本来计算此要求。我们可以使用-noee=true指令来禁用它。

##5.16 Includes 包含

We now have 2 files: javax.activation.bnd and javax.mail.bnd. If we would like to add some more descriptive headers like Bundle-Vendor, Bundle-DocURL, and Bundle-Copyright then we would have to add them to both files, another case of DNRY. Since this is quite common, we can use includes.

我们现在有2个文件：javax.activation.bnd和javax.mail.bnd。如果我们想添加一些更具描述性的header，例如Bundle-Vendor，Bundle-DocURL和Bundle-Rights，我们想要将它们添加到这两个文件中，我们可以使用includes指令。

	-include: common.bnd

In common.bnd we can then place:

在common.bnd，我们可以设置：

	Bundle-Vendor: Oracle (wrapped by bnd)
	Bundle-Copyright: (c) Oracle, All Rights Reserved

These headers are then automatically placed in the manifests of our target bundles.

然后这些header将会被自动放置在我们的目标bundle的清单中。

A common mistake with the `-include` instruction is that it is repeated several times. Since bnd files are property files, only one of them survives. However, the `-include` instruction accepts multiple files/URLs to both property files or manifest files. You can also indicate that it is ok to not have a certain file by prefixing it with a minus sign ('-').

-include指令的常见错误是它被重复多次。因为bnd文件是属性文件，所以只能存在一个。但是，-include指令接受属性文件或清单文件的多个文件/ URL。您还可以通过在其前面加上减号（' - '）指示没有某个文件的确定。

	-include common.bnd, -other.bnd

Always use Unix like paths, using the forward slash ('/') as file separator, also on Windows. It is highly recommended to always use relative paths since any absolute references makes your builds unportable.

bnd一直使用类似Unix的路径，所以在Windows上也使用正斜杠（'/'）作为文件分隔符。强烈建议使用相对路径，因为任何绝对引用将使您的构建不可移植。

##5.17 Sub bnd Files bnd子文件

In our progression towards wrapping javax.activation and javax.mail we have created an unmanaged ordering; we can only build javax.mail once the javax.activation is created. For this reason, bnd can also create a control file that steers _sub_ bnd files. The control file can also contain common properties, though these can be overridden by the sub files. Place the following content in javax.bnd:

在我们的打包javax.activation和javax.mail的过程中，我们创建了一个非托管排序; 我们只能在创建javax.activation后构建javax.mail。为此，bnd还可以创建一个控制文件，用于控制sub bnd文件。控制文件也可以包含公共属性，虽然这些属性可以被子文件覆盖。将以下内容放在javax.bnd中：

	Bundle-Vendor:       Oracle (wrapped by bnd)
	Bundle-Copyright:    (c) Oracle, All Rights Reserved
	-output:             bundle
	-sub: \
		javax.activation.bnd, \
		javax.mail.bnd

We can now build the jars with a simple command:

我们现在可以用一个简单的命令构建jar：

	$ rm bundle/*
	$ bnd javax.bnd
	$ ls bundle
	javax.activation.jar    javax.mail.jar

##5.18 Merging 合并

The basic model of bnd is to collect packages from the classpath and assemble them in a bundle. This pull model is quite different from the more common push model in builds where it is harder to include packages from other projects. However, bnd's model makes it quite easy to create a bundle out of multiple JARs. So lets add a new bnd file that merges javax activation and mail.

bnd的基本模型是从类路径收集包并将它们组合在一个bundle中。这个拉模型与更常见的推模型是非常不同的，在推模型中很难包括来自其他项目的包。然而，bnd的模型可以很容易从多个JAR中创建一个bundle。因此，我们添加一个新的bnd文件，它合并javax activation和mail。

Actually, javax.activation and javax.mail are really bad citizens in an OSGi world. They use class loading hacks all over the place that make a mockery out of modularity. Part of the problem is that they really require certain private directories to be available from the client's class loader. In those cases it is sometimes necessary to make sure the layout of the bundles is exactly the same as the source bundles. 

实际上，在OSGi世界中javax.activation和javax.mail真的不是好公民。他们使用类加载到处攻击，真是对模块化的嘲弄。部分问题是，他们真的需要客户端类加载器中提供的某些私有目录。在这些情况下，有时需要确保bundle的布局与源bundle完全相同。

The best way to achieve this is to _unroll_ the source bundles in the target bundle. You can unroll a JAR by prefixing it with a commercial at sign ('@') in an include resource operation. Lets get started on a javax.mail.all.bnd file:

实现这一点的最佳方法是在目标bundle 中展开源bundle。你可以通过在包含资源操作中以符号（'@'）为前缀来展开JAR。让我们以javax.mail.all.bnd文件开始：

	-includeresource:   \
		@jar/javax.activation.jar, \
		@jar/javax.mail.jar     

This instruction combines the two JARs into one.

该指令会将两个JAR组合成一个。

##5.19 Alternative Wrap 选择性打包

##5.20 Errors & Warnings 错误和警告

##5.21 Failing 失败

##5.22 Settings 设置

##5.23 System Commands 系统命令

##5.24 Upto 目标

bnd is the Swiss army knife of OSGi, it is used for creating and working with OSGi bundles. Its primary goal is take the pain out of developing bundles. With OSGi you are forced to provide additional metadata in the JAR's manifest to verify the consistency of your "class path". This metadata must be closely aligned with the class files in the bundle and the policies that a company has about versioning. Maintaining this metdata is an error prone chore because many aspects are redundant.

bnd是OSGi的瑞士军刀，它用于创建和使用OSGi包。它的主要目标是消除开发bundle的痛苦。使用OSGi，你必须在JAR的清单中提供额外的元数据来验证“类路径”的一致性。此元数据必须与bundle中的类文件和公司关于版本控制的策略紧密契合。维护这个元数据是一个容易发生错误的事情，因为许多方面是多此一举。

bnd's raison d'etre is therefore to remove the chores and use the redundancy to create the manifest from the class files instead of maintaining it by hand. The core task is therefore to analyze the class files and find any dependencies. These dependencies are then merged with ''instructions'' supplied by the user. For example, adding a version to all imported packages from a specific library can be specified as:

bnd存在的理由是删除杂项，并使用冗余从类文件创建清单，而不是手动维护。因此，bnd的核心任务是分析类文件并从中找到所有依赖关系。这些依赖关系随后与用户提供的“指令”合并。例如，从特定库中向所有导入包中添加一个版本可以被指定为：

    Import-Package: com.library.*; version = 1.21

The OSGi manifest must explicitly mention a package, bnd allows the use of wildcards. bnd contains many more such conveniences. bnd roots are about 10 years old and bnd has therefore a large number of functions that remove such chores. These range from simplifying the use of OSGi Declarative Services, working with Spring and Blueprint, WAR and WAB files, version analysis, project dependencies, and much more.

OSGi清单必须明确声明每一个包，bnd允许使用通配符。bnd包含许多这样的便利。bnd大约存在有10年了，因此bnd有大量的功能来除移除这样的杂务。例如，简化OSGi声明式服务的使用，与Spring和Blueprint协作，WAR和WAB文件，版本分析，项目依赖性等等。

Over time bnd started to appear in many different incarnations. It is an an ant task, a command line utility, and a bundle for Eclipse. Other projects have used bndlib to create a maven plugin, bndtools and Sigil both Eclipse IDEs, and others. By keeping the core library small and uncoupled (bnd has no external connections except Java 5), it is easy to embed the functionality in other projects.

随着时间的推移开始出现在许多不同的化身。它可以是一个Ant任务，一个命令行程序或者是一个Eclipse bundle。也有其他项目使用bndlib来创建maven插件，bndtools和Sigil both Eclipse IDEs等等。由于bnd保持小核心库和低耦合（bnd没有外部连接，除了Java 5），所以很容易将它的功能嵌入到其他项目中。

##5.25 Workflow 工作流
Traditionally, JAR files were made with the JDK jar tool, the jar ant task, or the Maven packager. All these tools share the same concept. The developer creates a directory image of the jar by copying files to a directory; this directory is then jarred. This model can be called the ''push'' model. Obviously this method works well.

通常来讲，JAR文件是利用JDK jar tool、jar Ant task或者Maven打包程序生成的。这些工具使用了同一模式：开发人员将文件复制到指定目录，然后打成jar包。这种模型被称为“推”模型。显然，这是个不错的模型。

bnd works differently, it uses the ''pull'' model. Instructions in the bnd file describe the contents of the desired JAR file without writing this structure to disk. The contents from the output can come from the class path or from anywhere in the file system. For example, the following instruction includes the designated packages in the JAR:

bnd工作方式不同，它使用了“拉”模型。bnd文件中的指令描述所需的JAR文件的内容，而不将此结构写入磁盘。输出中的内容可以来自类路径或来自文件系统中的任何位置。例如，以下指令包括JAR中指定的包：

  Private-Package: com.example.*
 
bnd can create a JAR from packages the sources, directories or other JAR files. You never have to copy files around, the instructions that Bnd receives are sufficient to retrieve the files from their original location, preprocessing or filtering when required.

bnd可以从包，源文件，目录或其他JAR文件创建一个JAR。你不必复制文件，Bnd接收的指令足以让它从原始位置检索文件，在需要时还可以进行预处理或过滤。

The Jar is constructed from 3 different arguments:

这个Jar由3个不同的参数构成：

    Export-Package
    Private-Package
    Include-Resource

Private-Package and Export-Package contain ''instructions''. Instructions are patterns + attributes and directives, looking like normal OSGi attributes and directives. For example:

私有包和导出包包含“指令”。指令是模式+属性和指令，看起来像正常的OSGi属性和指令。例子如下：

    Export-Package: com.acme.*;version=1.2

Each instruction is applied to each package on the classpath in the definition order. That is, if an earlier instruction matches, the later instruction never gets a chance to do its work. If an instruction matches its attributes and properties are applied to the packages. The difference between the Private-Package argument and the Export-Package arguments is that the export version selects the packages for export. If the packages overlap between the two, the export wins.

每个指令按照定义顺序应用于类路径上的每个包。也就是说，如果较早的指令匹配，则稍后的指令永远不会有机会进运行。如果指令与其属性匹配并且属性应用于包。Private-Package参数和Export-Package参数之间的区别是导出版本选择要导出的包。如果包在两者之间重叠，则导出获胜。

An instruction can also be negative when it starts with a '!'. In that case the package is excluded from the selection. For example:

当指令以“！”开头时，它为负。在这种情况下，包将从选择中被排除。例如：

    Export-Package: !com.acme.impl, com.acme.*;version=1.2

Note that the instructions are applied in order. If the ! instruction was at the end in the previous example, it would not have done its work because the com.acme.* would already have matched.

需要注意的是，指令会按顺序应用。如果！指令在上一个例子的结尾，它不会起作用，因为com.acme。*已经先于它匹配了。

The Include-Resource argument can be used to copy resources from the file system in the JAR. This is useful for licenses, images, etc. The instructions in the argument can be a directory, a file, or an inline JAR. The default JAR path is the the root for a directory or the filename for a file. The path can be overridden. Instructions that are enclosed in curly braces, like {license.txt}, are pre-processed, expanding any macros in the file.

在JAR中可以用Include-Resource参数从文件系统复制资源。这对许可证，图片等很有用。参数中的指令可以是目录，文件或内联JAR。默认JAR路径是根目录或文件的文件名。路径可以被覆盖。包含在大括号中的指令（例如{license.txt}）会被预处理，以展开文件中包含的任何macro。

Once the JAR is created, the bnd program analyzes the classes and creates an import list with all the packages that are not contained in the jar but which are referred to. This import list is matched against the Import-Package instructions. Normally, the Import-Package argument is *; all referred packages will be imported. However, sometimes it is necessary to ignore an import or provide attributes on the import statement. For example, make the import optional or discard the import:

一旦创建了JAR，bnd程序将分析这些类，并创建一个导入列表，此列表将列出所有未包含在jar中但在jar中被引用的包。此导入列表与导入包指令相匹配。通常，导入包参数是*; 所有引用包将被导入。但是，有时需要忽略一些导入或给提供属性。例如，使导入可选或丢弃导入：

    Import-Package: !com.acme.*, *;resolution:=optional

The arguments to bnd are normal given as a set of properties. Properties that begin with an upper case are copied to the manifest (possibly after processing). Lower case properties are used for macro variables but are not set as headers in the manifest.

bnd的参数是作为一组属性给出的。以大写开头的属性将复制到清单（可能经历了一系列处理）。小写属性用于宏变量，不会在清单文件中被设为header.

After the JAR is created, the bnd program will verify the result. This will check the resulting manifest in painstaking detail.

创建JAR之后，bnd程序将验证结果。这一过程将会检查生成的清单的细节。

The bnd program works on a higher level than the traditional jarring; this might take some getting used to. However, it is much more elegant to think in packages than that it is to think in files. The fact that the bnd understands the semantics of a bundle, allows it to detect many errors and also allows bundles to be created with almost no special information. 

相比于传统的jar,bnd程序运行在一个更高的层次，这可能需要花一些时间去习惯它。考虑包比考虑文件更加优雅，bnd理解bundle的语义允许它检测许多错误，并且允许创建bundle几乎没有特殊信息。

bnd will not create an output file if none of the resources is newer than an existing output file.

如果没有资源更新，bnd将不会创建输出文件。

The program is available in several forms: command line, ant task, maven plugin, and an Eclipse plugin.

该程序有几种形式：命令行，ant任务，maven插件和Eclipse插件。

##Tips 建议
There are some common pitfalls that can be prevented by following the tips:

下面是一些规避常见的陷阱的建议：

* Keep it simple. bnd's defaults are pretty good and not specifying is usually the best solution. KISS!
* Think packages ... yes it feels redundant to specify the packages that are in your source directory but your artifact will get a life of its own over time. Many IDEs and build tools restricted us to one artifact per project but bnd allows many artifacts, allowing the choice of granularity to you. As OSGi's packages can be easily refactored you can design the contents of your artifacts depending on the deployment needs. Think packages!
* Private is always better than export, only use export when you absolute need it.
* Not versioning an exported package is at your own peril. Sorry, that is false, it is at the peril of your users.
* Do not use the Bundle-ClassPath, if you need to include whole JARs, see the @ option at Include-Resource
* If you do not understand a header, remove it
* If you have a problem, make an example that is as small as possible and send it to [me](mailto:Peter.Kriens@aQute.biz).


* 把事情简单化。bnd的默认值相当不错，没有特别的指定通常是最好的解决方案。真棒
* 以包为单位思考 ...是的，在源文件目录具体定义一个包感觉也许有些多余，但随着时间的推移你的artifact将因为包而获得生命。许多IDE和构建工具将每个项目限制为一个 artifact，但bnd允许许多 artifact，允许你自己选择粒度。由于OSGi的包可以轻松被重构，因此您可以根据部署需要设计artifact的内容。想想packages！
* 私有总是比导出更好，只有当你绝对需要它时才使用导出。
* 不对导出的包进行版本控制对你是不利的。对不起，这是假的，相反它对你的用户是不利的。
* 如果需要包含整个JAR，请不要使用Bundle-ClassPath，请参阅Include-Resource中的@选项
* 如果有某个不理解的header，移除它就好了
* 如果你有什么问题，做一个尽可能小的例子发送给我

[1]: http://www.bndtools.org
[2]: http://repo1.maven.org/maven2/javax/activation/activation/1.1.1/activation-1.1.1.jar
[3]: http://ebr.springsource.com/repository/app/
[4]: http://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html#JARManifest
[5]: 170-versioning.html
