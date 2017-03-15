---
#2 Introduction 简介
---

OSGi is arguably one of the best specifications in the Java world. It is a no compromise specification for a component framework that is based on what we today know are the best practices. It extends the type safety first model of Java to hold true over time when the modules morph into sometimes unrecognizable shapes. It provides a solid foundation to build applications that can run anywhere, the original promise of Java; offering these impressive features while remaining as simple as possible, albeit no simpler.

OSGi可以说是Java世界中最好的规范之一。基于我们现在所知最好的实践，它是一个对于模块框架的no compromise规范。它拓展了Java类型安全模型以应对随着时间的推移，当模型发生无法识别的状态时确保其是真实有效的。它为构建可以在任何地方运行的程序（Java最初的承诺）提供了坚实的基础。它在提供这些令人印象深刻的特点的同时尽可能保证简单，虽然没有更简单。

For some, the previous paragraph may come as a surprise because OSGi has had its share of people complaining about it. Surprisingly, the biggest complaint is often the Class Not Found Exception, which is always a perfect sign that people try to push a round peg in a too small square hole, and with all their might. You only see those exceptions when you're not doing engineering but when you are hacking. If you run head on into the walls that OSGi installs and it is giving you a headache, then just look around and find the elegant and easy to use doors: services.

对于一些人来说，上一段可能会带来惊喜，因为已经有人抱怨它了。不过令人惊喜的是，关于OSGi最大的抱怨通常是Class Not Found Exception。这通常是个完美的信号，有人试图以他们全部的力量去想将一个圆桩推进一个过小的方形的洞里。如果你是去hack它而不是真的去做一些工程的话，你只会看到那些异常。如果你一头扎进OSGi Install所堆砌的墙中，你首先会头疼，然后看看周围，会发现那些优雅且易于使用的入口：Services（服务）。

Though this is all true, I do not claim that OSGi is trivial to use; triviality has a way to clash with large system that must evolve over many years. The software profession has a brutal industry that lures us with the siren song of a 'few hours work' to devour us while trying to main gigantic hairballs. As Fred Brooks already said so many years ago in his seminal book 'The Mythical Man Month', there is no silver bullet. Even OSGi will require hard work to build evolvable systems. And though we cannot make building complex systems easy, bnd can at least make it easier (and considerably more fun).

虽然这些都是真的，我并不是说OSGi是不重要的使用；那些演化多年巨型的系统，碎片化必然是造成崩溃的原因之一。软件行业是一个残酷的产业that lures us with the siren song of a ‘few hours work’ to devour us while trying to main gigantic hairballs。正如弗雷德·布鲁克斯在多年前在他的开创性着作“人月神话”中所说的：，没有银弹。即使是OSGi也需要艰苦的工作才能构建出可演化的系统。虽然我们，bnd，不能使构建复杂系统变成得简单，但至少可以尝试让它更简单（且赋予更多的乐趣）。

##2.1 A Bit of History 一些历史

When we started building the OSGi specifications around 1998 it became clear we needed metadata to describe the bundles. We chose the manifest since this was already a well defined resource in the JAR, why invent the wheel? Actually, it was Anselm Baird, a Sun employee at the time, who had come up with this idea in his Java Embedded Server (JES). Basically, the manifest was a property file with benefits. We still consider this choice for embedded metadata an excellent idea. Looking at Maven where the metadata is separated over a surprising number of files. It is clear that the reduction in the number of files and potential errors caused by out-of-sync metadata is quite large. My favorite example of this additional complexity is when the SHA1 files necessary in Maven were also signed, an easy mistake but induced by unnecessary complexity. An OSGi bundle is a completely self described artifact.

当我们在1998年开始构建OSGi规范时，我们需要用元数据（metadata）来描述bundle。我们选择了Manifest，因为它已经是JAR中一个定义明确的资源，我们为什么要重新发明车轮呢？事实上，当时Sun员工的Anselm Baird，他在他的Java嵌入式服务器（JES）中提出了这个想法。基本上来说，清单（Manifest）是一个属性文件同时它有许多好处。直至今日，我们仍然认为嵌入式元数据的这个选择是一个好主意。看看Maven吧，其中元数据需要在惊人数量的文件中分离。显然，由不同步元数据引起的文件数量和潜在错误的减少是相当大的。我个人最喜欢的这个额外复杂性的例子是当Maven中需要的SHA1文件也被签名，一个容易的错误，但由不必要的复杂性引起。 而OSGi bundle则是一个完全自我描述的组件。

##2.2 Transitive Dependencies 传递依赖
So who got bnd started? Well, when we wrote our bundles in the dawn of the new millenium we quickly discovered that transient dependencies, well, eh, sucked. Several of us had great experience in Object Oriented design, and though objects clearly had become mainstream they frustratingly had not provided the level of reusability that some had dreamed of. As Brian Foote shows, we thought we were building shiny castles but in the end built a big ball of mud. Where our predecessors of the structured programming era always hammered on low coupling and high cohesion we had ignored their wise lessons and got ourselves tangled up in a nice mess. Trying to reuse a class too often dragged in more unwanted stuff than the cat does in the whole year.

那么，是谁创建的bnd呢？在2000年初，我们在写bundles的时候发现依赖传递是一个很棘手的问题，有精通面向对象设计模式的人发现虽然面向对象已经成为主流，但是它的代码重用性并没有达到我们的预期水平。就像Brian Foote说的那样，我们认为自己正在建造闪亮的城堡，最后却建立了一个大泥团。在结构化编程时代，前人们总是致力于实现程序的高内聚低耦合性，我们却忽视了这一点，以至让我们的程序陷入一片混乱。为了更多地重用一个类，把很多不想要的东西全部拖到一块儿来了。

The advantage of Java was that it had found an elegant solution to the scourge of transient dependencies: Java interfaces. An interface elegantly broke transitive dependencies while maintaining type safety at the cost of an indirection. A cost anyone should gladly bear for hairballs are quite distasteful. Surprisingly, while Java offered us this elegant solution to break transitive dependencies, most people in our industry were blindly committing themselves to the same route, just on another level, which actually would make the consequences even worse.

Java的优点是它找到了一个优雅的方案来收拾传递依赖这个祸害：Java接口。接口优雅地打破了传递性依赖，同时以间接性为代价来维持类型的安全性。每个人都很愉快地接受了这个小代价是十分令人讨厌的。令人惊讶的是，我们行业中的大多数人都在使用Java提供的这种解决方案。从另一方面来说，这样会使结果变得更加糟糕。

What we discovered was that we could use the idea of interfaces in an object oriented environment also for modules. An interface is an instance of contract based programming and we found out that we could use the same between modules. The idea behind the OSGi service is that you do not let modules directly interact but that you use reified conduits between the modules so that you keep the modules, which are implementations, unconnected. What better concept in Java to express this contract between modules is there but the concept of a package?

我们发现接口不仅适用于面向对象编程，还适用于模块。接口是基于契约编程的一个实例，在模块之间也可以使用它。OSGi服务背后的理念是，不让模块直接交互，而是让模块之间使用合法的管道进行交互，以便保持模块的独立性。在Java中包是表达模块之间的这种契约的最佳方式。

##2.3 Packages 包

Originally devised as the Java modules, Java packages are heavily underappreciated in the Java world. Even highly expert users sometimes only appreciate the namescoping of the package, brushing over any accessibility issues arising out of the use of multiple classloaders. They are wrong, a package is a perfect module. That said, it is a perfect encapsulation to describe the contract that governs the collaboration between modules. An interface has a too small granularity for such a collaboration contract since it often requires additional interfaces like listeners and also helper classes. Packages, although they are too fine grained for deployment modules, are eminently suitable to define a contract to given the collaboration between deployment modules.

在Java世界中，最初被设计为Java模块的Java包被大大地低估。即使是专家级用户有时也只是高度赞赏包的命名约束，忽略了由于使用过多类加载器引起的任何其它问题。但是他们错了，一个包是一个完美的模块。也就是说，它完美地封装了描述管理模块之间的协作契约。对于这样的协作契约，接口的粒度太小了，因为它通常需要一些附加接口，例如监听器和辅助类。对于部署模块来说，包虽然小，但是它们非常适合用来定义部署模块之间协作的契约。

So we had come up with the surprisingly well fitting dependency model: 'Look Ma, No Coupling!' But then using it became quite painful because it required a lot of tedious house keeping, tracking which packages you were using, of what version. Just bluntly expressing your dependency on an implementation sometimes felt as attractive as the next shot of heroin for a junkie. Now, housekeeping and chores in general are quite low on the list of reasons to give meaning to live for most software developers, and we were no exception. Not only are we in general horrible at chores, we tend to make an amazing amount of mistakes doing them. Which, in general is not good software engineering. Fortunately, we largely live in a virtual world where it is easy to let the computer do the chores. And this is the raison d'etre of bnd: take the chores out of OSGi and focus on the fun parts. And maybe even more important, 'Do Not Repeat Yourself'.

我们想出了非常好的依赖模型：“看，没有耦合！” 但是使用它变得相当痛苦，因为它需要很多复杂的空间来保存和跟踪你使用了哪些包以及使用了包的什么版本。直白地说，有时你对实现的依赖就像瘾君子对下一口海洛因的依赖。现在，一般的杂务对于大多数软件开发者来说意义是很低的，我们也不例外。通常，我们不仅讨厌琐事，而且在做这些琐事的过程中有犯更多的错误的倾向。幸运的是，我们大部分人生活在一个虚拟世界里，很容易让计算机来做这些杂务。这也就是bnd的存在理由：把OSGi中的杂务拿出去，让人们专注于有趣的部分。也许更重要的是，“不要重复自身”。

##2.4 BTool
The early version of bnd was called btool and was incepted at Ericsson, where it was used to automate the generation of the manifest in the ebox project. Then, in 2001, btool was used to automate the build process of the specifications, the reference implementations, and the test suites at the OSGi Alliance. One day there was a bit of confusion about the pedigree of the btool, and it was showing its age; it was time for a rewrite and bnd was born.

bnd的早期版本被称为btool，在爱立信中，它被用于为ebox项目自动生成清单。然后，2001年，在OSGi联盟中，btool被用于OSGi规范，参考实现和测试程序组的自动构建过程。有一天，人们觉得btool的血统混乱，年龄也大了，是时候重写它了，所以bnd诞生了。

The name came from bundle, then removing he vowels, and cutting it to three characters to reduce the typing on the command line. The pronunciation: we actually have no clue, whatever your preference is. _B AND D_, _bind_, whatever. You want to write it as bnd, Bnd, or BND, be our guest. We prefer to write it like bnd since we love the symmetry.

Bndde的名字来源于bundle，删除元音字母取其前三个字符，以减少在命令行上的打字。发音：实际上我们也不确定，可以根据你自己的喜好去称呼它，例如B和D，绑定等等什么的都可以。作为我们的客户，你把它写成bnd，Bnd或BND都可以。我们更喜欢把它写成bnd，因为我们热爱对称性。

Over the years bnd gained a lot of power because we needed a tool at the OSGi Alliance that could not just create good bundles, it also was heavily used in the test cases that required bad bundles. Of course this was an excellent test range for bnd and this synergy is still true today.  

多年来，bnd从OSGi中获得了很多的力量，因为在OSGi联盟中需要的这样一个工具，该工具不仅能创建好的bundle，也能创建大量坏的bundle来当测试用例。当然，这对bnd来说是一个很好试验，到目前为止这种协作还在继续。

##2.5 Eclipse

In 2003 Eclipse decided to adopt the OSGi specifications and this generated some interesting discussions. To the OSGi crowd it was crystal clear that packages were the future, the Eclipse crowd, coming from a traditional transitive direct hard module dependency world, this future was more fuzzy. We lost, and Require-Bundle and Fragment-Host are the souvenirs of that battle. However, this was not the only disagreement, unfortunately we also quarreled about how to develop bundles (sorry, plugins) and there we lost again with PDE and P2 the landmarks remembering us of this lost fight. Our experience with bnd had taught us that one should not try to maintain the manifest by hand, just like one should not try to write class files with a hex editor. The manifest is a readable format but it was never intended to be human writable except for emergencies. To simplify the runtime (important in an embedded world) we never made the manifest easy to write. For example, when you specify the packages you want to export it is very useful to use wildcards to include a number of sub-packages. However, in runtime this would force the framework to traverse all classes to make the list of total packages of which it could then select the packages based on the wildcards. There is also an inevitable duplication of information as well as derived information which makes it impossible to not create evil redundancy.

在2003年，Eclipse决定采用OSGi规范，这引发了一系列有趣的讨论。对于OSGi人群来说，很明显包就是未来。然而对于这群来自传统传递性模块依赖世界的Eclipse人群来说，这一未来十分模糊。在让Eclipse也使用bnd的这场战役中，我们失败了，Require-Bundle和Fragment-Host标记是这场战斗的纪念品。然而，不幸的是，这不是我们唯一意见不同的地方，在争论如何开发bundle（对不起，插件）的过程中，我们再次输给了PDE和P2，这些标志让我们想起这场失败的战役。多年的开发bnd经验告诉我们，不应该去尝试手动维护清单，就像不应该尝试用十六进制编辑器编写类文件。虽然清单是一种可读的格式文件，但除了在紧急情况下永远不要尝试去手动编写它。为了简化runtime（这在嵌入式世界中很重要），我们从来没有想过要使清单容易编写。例如，当您指定要导出的包时，使用通配符包含多个子包非常有用。然而，在运行时，这将强制框架遍历所有类来制作总的包列表，然后它可以基于通配符选择包。同时不可避免地会产生重复的信息和导出信息，这使得系统不可能不产生令人讨厌的冗余。

Unfortunately, the PDE guys insisted on 'manifest first'. In their model, the manifest drives the building of the artifact instead of representing the output of a build process. That is, if you require a bundle in your manifest, then PDE will place this on the classpath of the bundle you're building. If you import a package in the manifest, PDE will find a bundle exporting that package and place it on the classpath (praying there is only one bundle that exports it in the target).

不幸的是，PDE那群人坚持“清单优先”。在他们的模型中，清单用来驱动构件的构建，而不是用来展示构建过程的输出。也就是说，如果你需要在清单中有一个bundle，那么PDE会将它放在你正在构建的bundle的类路径中。如果在清单中导入一个包，PDE将找到一个包导出该包并将其放在类路径上（祈祷在目标平台中只有一个bundle导出了这个包）。

##2.6 Inside the OSGi Alliance 在OSGi联盟内

Inside the OSGi Alliance we were in a bind (pun intended) because PDE was not suitable for our own build. We used bnd in ant (it was also an ant plugin) but we missed the joy of Eclipse. This caused bnd to develop a split personality. Originally it was a JAR generator based on a small recipe but for our build we needed project & workspace concepts. We toyed with the idea to split it into a bld and a bnd tool but in the end even we committed the sin against modularity of low cohesion and kept these two tools in one out of laziness.

在OSGi联盟内部，我们遇到了困难（双关意图），因为PDE不适合我们自己的构建。我们在ant中使用bnd（它也是一个ant插件），但我们错过了使用Eclipse的快乐，这导致了bnd发展出了分裂的特性，最初它是一个基于小偏方的JAR生成器，但对于构建，我们需要项目和工作区概念。抱着玩玩的想法，我们把bnd分成一个bld和一个bnd工具。但最终我们犯了模块之间低内聚的罪，让这两个工具中的一个处于闲置状态。

Obviously we also developed PDE envy because Eclipse was actually awfully nice except for it. After toying with the idea of using the Eclipse metadata (.classpath) and finding out that this was only possible if we included the complete Eclipse IDE in an offline build, we embarked on developing an Eclipse plugin. Just not a good idea. So we reversed the model, and developed a library that had its own internal, uncoupled, model of a build. You see, the problem is that most build environments are quite pedantic and strongly optimized for their primary goal. Ant, maven, gradle, et. al. are just not easy to use inside an IDE because they are very stream driven: start, process, stop. In this model there is very little incentive to optimize incremental building and event notifications for important changes. An IDE is the reverse, it is start, build, build, build, ..., build, build, stop. For performance, it is crucial to optimize the building out of incremental changes to keep the IDE responsive. It is also crucial to send out events when important things happen. So we decided to pursue the middle ground: a model of projects and workspaces that was as uncoupled of the real world as possible but providing the hooks to use it in all popular build tools, either command line tools or IDEs. This is rather well captured in the expression: "The one tool that bnd's them all".

显然，我们不是那么喜欢PDE，因为Eclipse除了PDE之外都是非常好的。当我们着手开发一个Eclipse插件时，在这个过程中有了使用Eclipse元数据（.classpath）的想法，我们发现唯一可能的方式就是包括完整的Eclipse IDE离线构建进来，只是这不是一个好主意。所以我们扭转了模型，开发了一个有自己的内部的，非耦合的构建的模型库。你看，主要的问题其实是大多数构建环境都十分固执，强烈地想要优化其主要目标。像ant，maven，gradle这些构建工具只是在IDE中不容易被使用，因为他们属于流驱动：启动，进程，停止。在这个模型中，不是特别鼓励优化增量构建和重大事件通知功能。IDE刚好是是相反的，它的模型是：启动，构建，构建，构建，...，构建，构建，停止。在性能方面，对增量构建做出响应对IDE来说很重要，同样，对重大事件的响应也是至关重要的。因此，我们决定追求中间层：projects&workspaces模型，尽可能与现实世界脱钩，同时向外提供一个钩子，让它在所有的主流构建工具、命令行工具或IDE都能够被使用。所以对于这个模型更好的描述是：“一个把所有都联系起来的工具”。

##2.7 bndtools

Then one day we heard that Neil Bartlett also had started a plugin called bndtools. He had used the open source library of bnd and started to work on creating a pleasant to use friendly environment. Interestingly, he had created a continuous builder for bnd so that every save operation automatically build the bundles, something we had considered in our plugin but had been too afraid to do out of performance fear. Neil, however, was developing a lot of stuff that was already in bnd, he only used the JAR packager and manifest generator. After beating him up, which even took a special trip to the UK where we worked for 8 hours in a hotel lounge, he surrendered and thus our long term fruitful collaboration was born.

有一天，我们听说Neil Bartlett开发了一个名为bndtools的插件。他使用bnd的开源库开始创造一个用户友好的使用环境。有趣的是，他为bnd创建了一个连续的构建器，这样每个保存操作都会触发bundle的自动构建，其实之前在我们也考虑过在插件中加这个功能，但是由于害怕出现性能问题就放弃了。然而，Neil正在开发的很多东西bnd中已经有了，他只使用了bnd中的JAR打包器和清单生成器。为了让他开发的工具更加完善，我们特地去英国和他在一个酒店休息室里一起工作了8小时，最后我们达成了长期有效合作关系。

Currently bnd(tools) is managed through a Github organization called bndtools and has its home site at http://bndtools.org. It is split in a number of repositories: bndtools, bnd, bndtools.rt and a number of associated support repositories. Though the collaboration between bndtools and bnd is awfully close, we are fanatic about separating the projects, our goal is still to allow Jetbrains to use bnd in IntelliJ without forcing them to eat any Eclipsisms. And it is always fun when hear someone claiming to successfully integrate bndlib in a product or tool, raving how uncoupled it was.

目前，bnd（tools）通过一个名为bndtools的Github组织进行管理，其主站点位于http://bndtools.org。它被分为多个存储库：bndtools，bnd，bndtools.rt和一些关联的支持存储库。虽然bndtools和bnd之间的合作非常密切，但是仍然热衷于分离项目，我们的目标是允许Jetbrains和IntelliJ使用bnd，而不强迫他们使用eclipse相关的插件。当听到有人声称在一个产品或工具中成功集成bndlib时，总会觉得很有意思。

##2.8 Manual 手册

What you are now reading is the manual of bnd, which is the result of a tremendous amount of work, and will be a work in progress for a long time to be. Though much of bnd's development was indirectly supported by the OSGi Alliance it is and always will be an open source project. This gave the authors the license to scratch their itches and not worry too much about documenting the nifty things they developed to scratch. Though the most important aspects were documented, it was sparse and not overly well organized. Though we always hoped someone with a gift for documenting would come along, fall in love with bnd, and write the perfect documentation, this somehow failed. After 16 years, we find it is time to take up this task ourselves, still praying that we will get support from bnd's surprisingly large (to some) group of users, don't hesitate. This is the reason this manual is a github repositories. Contributing is trivial, clone the bndtools/bnd.manual repo, edit the markdown text (you can even do this on the github web), save it, and create a pull request. Don't (always) ask what bnd can do for you, ask what you can do for bnd's users ...

你现在正在阅读的是bnd用户手册，这本手册是一个巨大的工程，这个工程在以后还会持续很长一段时间。尽管OSGi联盟在很大程度上间接支持了bnd的发展，但bnd始终都会是一个开源项目。这给了作者去安心开发bnd却不用担心写文档的权利。虽然bnd重要的特征都有相应文档，但是这些文档十分稀少且没有被归整起来。我们总是希望有一个人疯狂爱上bnd，为它写出完美的文档，并把它作为礼物送给我们，不知道为什么这个愿望一直没能实现。16年后，我们发现是时候自己去承担起这个任务了，但是我们仍然祈祷得到bnd的广大用户的支持，所以如果你想为我们贡献文档，千万不要犹豫，因为这是本手册是一个github库。贡献文档的过程十分简单，首先克隆bndtools/bnd.manual repo，然后编辑markdown文本（你甚至可以在github网页上编辑它），保存之后创建一个pull request就可以啦。不要一直问什么是bnd可以为你做些什么，首先问你能为bnd的用户做些什么...

Not sure why we wrote such a long introduction, the facebook generation seems have an attention span of 5 lines, so we are probably among ourselves dear reader, congratulations on your stamina!

不知道为什么我们写了这么长的介绍，现在使用facebook的这一代人的注意力持久力似乎只有5行。所以亲爱的读者们，恭喜你拥有特别棒的耐力！
