---
# 6 Concepts 概念
---

One of the more surprising things we've learned about modularity in the last decade is how much of a fractal pattern it actually is. All the way from CPU machine instructions all the way up to a large distributed system you can see the pattern of encapsulation to keep as much details as possible confined to the internals of the module. Strangely enough, as an industry we seem to have a hard time learning the lessons from the lower layers where we over time learned the rules to the layers above.

在过去十年中，随着对模块化的深入学习，我们震惊于它实际所拥有的分形模式。从CPU机器指令一直到大型分布式系统，你都可以看到把细节限制在模块内部的封装模式。在软件工程中，我们很难从被封装的底层学到经验，一般都是从被封装层之上学习规则，例如被封装的模块是如何运作的，如何运用这些规则去创建新的模块等。

Java was developed the early 90's as a state of the art language based on Object Oriented concepts. It did apply the rules of modularity on the method and class level and to pioneered with providing a higher level module with their concept of _packages_. However, our software industry did their utmost to ignore this concept, helped by the fact that it did have some real shortcomings.

在90年代初，Java作为基于面向对象概念最先进的语言出现。它确实在方法和类的级别上运用了模块化规则，并率先提出以包为模块化标准的理念。然而，我们的软件行业极力忽略这个理念，因为它确实有一些缺点。

The key shortcoming was that it was hard to keep classes private since they required public visibility when they crossed the package boundary. This collided with the granularity of the package, in general you needed to break an application in many different packages to stay sane as a developer; with the consequence that a lot of private code was forced to become public.

关键的缺点是，当人们要在包外使用包中的类时，他们需要类是公共可见的，这样就很难保持类的私有性。这与包的粒度相冲突，作为一个开发者，通常你需要把一个应用程序分成很多个不同的包以保持程序的健壮，但是这有导致了很多私有代码被迫公开。

Java did provide a higher level module: the JAR. However, this was a concept outside the language, it was only visible in runtime to be used by the class loaders. Interestingly, it was then used in runtime security but it was kept outside the language that still desperately tried to front a single name space. Problems showed up as class cast exceptions, package private access violations in the same package, and getting the wrong class. It is amazing how brittle the class path model of Java really is.

Java提供了一个更高级别的模块：JAR。然而，这是一个语言之外的概念，它只在运行时被类装载器使用时可见。有趣的是，JAR被用于保证运行的安全性，它与java语言无关但是仍然想获得一个单一的命名空间。这时候问题就来了，包在同一个包中的私有访问冲突，并得到错误的类，此时会显示为类转换异常，。多么令人惊讶，Java的类路径模型真是脆弱。

In search for a model that would allow the management of hundreds of thousands of devices running Java the OSGi decided to leverage the JAR but turn it in to a proper module by adding encapsulation. Since in modularity a module tends to export/import entities that have a lower granularity (classes export/import methods, packages import/export classes), the OSGi solution was to export/import packages.

在搜索一个能够管理成千上万个运行Java的设备模型时，OSGi决定利用JAR，但是会通过添加封装将其转换为适当的模块。在模块化过程中，因为一个模块倾向于导出/导入具有较低粒度的实体（类导出/导入方法，包导入/导出类），所以OSGi解决方案是导出/导入包。

Package imports/export was a much maligned choice because the maligners did not understand that some of the lessons learned from object oriented programming, where the class is the module, also applied to the other kinds of modules. The greatest problem with Object Oriented was that direct object references created highly coupled systems that were very brittle. Direct references aggregate many constraints that are only important for a specific implementation. This problem was not fully solved until we got Java _interfaces_. Java interfaces allow a user of that interface and an implementer of that interface to only couple on the relevant details necessary for the collaboration; any implementation details on either side of the interface are not constraining the other side in any way. 

导入/导出包是一个不怀好意的选择，因为恶意者没有从面向对象编程中获得的一些经验教训，在面向对象编程中，类是模块，也被用于其他类型的模块中。面向对象的最大的问题是直接对象引用创建了非常脆弱的高耦合系统。直接引用聚合了许多只对特定实现重要的约束。这个问题直到我们得到Java接口才完全解决。Java接口允许该接口的用户和该接口的实现者仅耦合在协作所需的相关细节上; 接口的实现细节不以任何方式约束影响用户。

What the maligners do not see is that we face the exact same problem when we turn JAR into modules. The maven model only supports direct references between modules and they aggregate implementation constraints in the same way as object references did. The solution the OSGi came up with is to treat a package as a _specification_, just like a Java interface specifies the behavior of an implementation class. 

恶意者没有看到的是，当我们将JAR转换为模块时，我们面临着完全相同的问题。maven模型只支持模块之间的直接引用，它们以与对象引用相同的方式聚合实现约束。OSGi提出的解决方案是将包视为规范，就像Java接口指定实现类的行为一样。

In this way a bundle (the OSGi JAR module) could provide an implementation for a (specification) package and other bundles could consume this implementation. Again, this is similar to the Java interface for classes. 

这样，bundle（OSGi JAR模块）可以为（规范）包提供实现，其他bundle可以使用这个实现。同样，这类似于类的Java接口。

In the Java world we learned rather slowly that interfaces are great but that you have this pesky problem of how to get an instance for that interface if you should not be coupled to a specific implementation? Java created a a number of quite horrendous _factory_ models for almost each subsystem, a factory model that too often bled into the actual collaborative API. For example, the `javax.persistence` package does not only contain the classes to work with JPA, it also contains a surprisingly large API to manage the life cycle of the provider. Factories also heavily abused class loaders for a purpose they were clearly not designed for. In Java 6 Sun tried to stop the hemorrhaging with the Service loader but forgot that class loaders are not designed for this purpose and baked them right into the API. 

在Java世界中，我们十分缓慢地学习了解到接口是伟大的，那么问题来了，如果你不为接口绑定一个特定的实现，那么你如何获得一个接口的实例？Java几乎为每个子系统都创建了一些非常可怕的工厂模型，这是一个工厂模型经常渗入实际的协作API。例如，javax.persistence包不仅包含与JPA一起使用的类，它还包含一个惊人的大型API来管理程序的生命周期。工厂模式还严重滥用类加载器，但是这显然不是类加载器被设计的目的。在Java 6 Sun尝试使用服务加载器停止出血，但忘记类加载器不是为此目的设计，并将它们直接放入API。

Spring brought dependency injection to the masses that allowed the consumers and providers to stay away from each other but it then introduced a massive coupling in the XML that contained too many details about the provider implementations since it specified the provider class names. 

Spring向群众提供了依赖注入，允许消费者和提供者彼此远离，但是随后在XML中引入了大量的耦合，包含了太多关于提供者实现的细节，因为它指定了提供者类名。

The solution OSGi came up with in 1998 was to use a _service broker_. A broker makes sure that consumers and providers were bound together once they are available. This is the only true modular model that allows all implementation details to remain private inside the module. 

1998年，OSGi提出的解决方案是使用服务代理，代理确保消费者和提供者在可用时绑定在一起。这是唯一真正的模块化模型，允许所有实现细节在模块内保持私有。

A the consequence of this model is that services, an implementation object implementing a Java interface from a specification package, cannot be assumed to be always there. Where Spring calculates the dependencies of its beans to initialize a system in the right order, a broker negotiates when a service is available since it does know any of its implementation details. One of those implementation details is when that service will be available. The broker is therefore dynamic, it can react to changing circumstances, another even more both maligned and misunderstood OSGi feature.

这个模型的结果是不能假设实现了某个特定接口的实现类（服务）是一直存在的。Spring通过计算其bean的依赖关系来达到以正确的顺序初始化系统的目的，因为它知道任何的实现细节，所以它就像一个代理可以去协商服务何时可用。其中一个实现细节是什么时候该服务是可用的。因此，代理是动态的，它可以响应不断变化的情况，这是另一个会让人诋毁和误解OSGi功能的存在了。

The hardest part of OSGi is to look at it with a clean slate. The core model is surprisingly simple, elegant, and extremely powerful. The hard part is that it tends to conflict (as it should!) with all the bad habits in existing code bases that grew up thinking class loaders were simple extension mechanisms.

OSGi最难的部分是用全新的方式来看待它。它的核心模式惊人的简单，优雅，同时又非常强大。困难的部分是，它倾向于与现有代码库中所有坏习惯（越来越多人认为类加载器只是简单的扩展机制）有冲突。

##6.1 Bundles

So to summarize, a bundle is a JAR that imports and exports a set of packages. These imports and exports are bound to other bundles when they are resolved, allowing multiple versions of the same package. Once a bundle is started, it can  then communicate with the external world at its own discretion; it can also get and register services to collaborate with other bundles. The following picture depicts the model of an OSGi application:

总而言之，bundle是一个JAR，用于导入和导出一组包。这些导入和导出在解析时绑定到其他bundle，允许同一个包的多个版本。一旦一个bundle启动了，它就可以自己决定与外部世界沟通; 它也可以通过获得和注册服务来与其他bundle协作。下图描述了OSGi应用程序的模型：

<embed class="illustration" src="/img/chapters/concepts-bundles.svg" />

The symbols used are defined in the OSGi specifications. The rectangle with rounded corners is a bundle, the triangle is a service (it always points in the dependency direction, i.e. the Reporter bundle depends on the sensor service to be there. Input and output is depicted with the corresponding flow chart symbol.

上图所使用的符号都在OSGi规范中定义好了。具有圆角的矩形是一个bundle，三角形是一个服务（它总是指向依赖方向，即Reporter bundle依赖传感器服务存在）。输入和输出用相应的流程图符号描述。

In a perfect OSGi world, packages are just a minor detail. Until then, we also need some way to show package imports and package exports. In the OSGi specification, we use an open rectangle for imported packages and and black rectangle for exported packages. Private packages are depicted with a grey rectangle, see:

在一个完美的OSGi世界中，包只是一个小细节。在那之前，我们还需要一些方法来显示包导入和包导出。在OSGi规范中，我们对导入的包使用一个打开的矩形，对于导出的包使用黑色矩形。私有包用灰色矩形描述，请参阅：

<embed class="illustration" src="/img/chapters/concepts-packages.svg" />

##6.2 Components 组件

There is often a confusion of terminology, like, is a bundle a component? We are guilty of not always using the terminology consistent. However, in the past few years it has become very clear that the OSGi declarative services provides a programming model that should have been incorporated in the OSGi framework from the beginning. DS allows you to make any object _active_ with a simple annotation. This object can automatically be registered as a service. If there are dependencies on other services then you can easily specify those dependencies with an annotation on a set method.

时常会出现术语上的混乱，比如说，bundle是一个组件吗？我们有罪，不应该总是使用不一致的术语。然而，在过去几年中变得非常清楚的是，OSGi声明性服务提供了一套从一开始就应该被并入OSGi框架中的编程模型。DS允许你使用简单的注释激活任何对象，此对象可以自动注册为服务。如果它依赖其他服务，那么你可以轻松地在set方法上使用注释指定这些依赖。

In DS, the implementation class is called the _component_ and in this document we follow this lead. Therefore, in general a bundle consists of a number of components. 

在DS中，实现类被称为组件，在本文档中我们遵循这个说法。因此，一般来说，一个bundle由多个组件组成。

#6.3 The Workspace 工作空间

The bndlib _workspace_ is an encapsulation of a set of cohesive _projects_, where a project exports zero or more bundles via _repositories_. A repository provides access to set of bundles exported by some means, likely from other workspaces. A repository can be on the local file system or a remote system like JPM or Maven central. 

bndlib工作空间是一组内聚项目的封装，其中项目可以通过仓库导出零个或多个bundle 。仓库同时也提供对其他bundle（可能来自其他工作空间）的访问。仓库可以在本地文件系统或远程系统上（如JPM或Maven中央仓库）。

Projects can depend on other projects in the workspace or import bundles from the repositories. This is depicted in the following figure.

项目可以依赖于工作空间中的其他项目或从仓库中导入的bundle，如下图所示。

<embed class="illustration" src="/img/chapters/concepts-workspace.svg" />

A workspace is a single directory, just like a git workspace it encompasses all its sub directories. Though the name of the workspace directory is free to choose, it is highly recommended to use a naming strategy. In practice you will create many different workspaces and having a naming strategy will significantly simplify the handling of these workspaces.

一个工作空间是一个单目录，就像一个git工作区，它包含所有的子目录。虽然工作空间目录的名称可以自由选择，但强烈建议使用命名策略。实际上，您将创建许多不同的工作空间，并且使用命名策略将大大简化这些工作空间的处理。

On the same root level in the workspace as the `cnf` directory, bndlib expects the projects. Yes, projects *must* reside in the root level. The reason is again, simplicity. We will later discuss how the bundle's symbolic name is derived from the project's directory name. Since projects can depend on each other, bndlib maintains a workspace repository of projects that it derives from the top level directories. Some people desperately want to use hierarchies of projects (often because that is how they used to work before). However, even people patching bndlib to make it hierarchical admit that the simple linear model is actually working quite well. The reason it works so well is that a workspace is not supposed to hold every single bundle that your organization produces. It is intended to be a cohesive set of between 10-20 up to a couple of hundred projects.

项目目录需要像cnf，bndlib目录一样在工作空间的根目录下。是的，项目必须存放在根目录下。再次说下原因，因为简单。我们稍后将讨论bundle的符号名称是如何从项目的目录名称派生而来。由于项目可以依赖于彼此，bndlib维护了一个从顶层目录派生的项目的工作库。有些人迫切地想使用项目的层次结构（通常是因为这是他们以前的工作方式）。然而，人们通过给bndlib打补丁，使它结构化并承认简单的线性模型实际上也能很好运行。它运行得很好的原因是工作空间不应该包含您的组织生产的每一个bundle。它倾向于包含一组内聚的10-20到100上下的项目。

