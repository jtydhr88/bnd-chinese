---
#10 Versioning 版本控制
---


Versioning is probably the most painful part of developing real software. Where toys and prototypes can be developed ignoring evolution, real software requires a migration path to an unknown future.

版本控制可能是开发软件时遇到最头疼的事了。玩具或者原型的发展也许可以忽略演变，但是真实的软件需要一个面向未来的迁移路线。

The OSGi has defined a versioning policy that is described in the [Semantic Versioning whitepaper](http://www.osgi.org/wiki/uploads/Links/SemanticVersioning.pdf). bnd fully supports this model and provides many shortcuts. The goal of bnd is remove any manual work from versioning bundles as well as packages.

OSGi已经在语义版本控制白皮书中定义了版本控制策略。bnd完全支持此模型，并提供了许多快捷方式。bnd的目标是让bundle和包的版本控制完全自动化。

The key concept to version in OSGi is the ''package''. Bundles are an ''aggregate'' and therefore must move as fast as the fastest moving exported packages they contain. For example, if a bundle contains two exported packages `foo` and `bar` and `foo` is not changed but `bar` has a major change, then the bundle version needs to also have a major change. This requires an unnecessary update for a bundle that only depended on `foo`. Aggregating dependencies increases the fan out of the transitive dependencies. The result is that systems can only evolve when everything is updated simultaneously. The result is that the system as a whole becomes brittle.

OSGi版本的关键概念是“包”。bundle是聚合体，所以它应该与它迁移最快的导出包保持相同的迁移速度。例如，如果一个bundle包含两个导出的包foo，bar，其中foo没有更改，但是bar有一个重大的改变，那么bundle的版本也需要有一个重大的改变。这将导致仅依赖foo的bundle进行一次不必要的更新。聚合依赖关系会增加扇区的传递性依赖，这将会导致系统只有在同时更新的时候才能发展，系统整体变的脆弱。

In contrast, versioning the packages and using Import-Package, bundles can be refactored and versioned independently.

相比之下，版本化软件包并使用Import-Package，bundle可以独立重构和版本化。

## 10.1 Best Practices 最佳实践

* Version on packages, not on bundles
* Create a packageinfo file with "version 1.0" in each package directory and maintain it meticuously for each change to the package. Any change that breaks consumers, increment major. Changes that break providers (if any), increment minor. Other changes micro. bnd will then properly version this package when exported
* If you provide an API (as defined in te Semantic Versioning whitepaper) export the API package even if it is not in your project and put the `provide:=true` directive on the package export, e.g. `Export-Package: org.osgi.service.event; provide:=true`.
* Every time when you release a bundle to the external world, bump the minor part of the bundle  version. For internal development releases, bump the micro part of the bundle version.

* 给包指定版本，而不是bundles
* 如果您提供一个API（如“语义版本控制白皮书”中定义的那样），那么你需要导出这个API的包即使你的项目用不到它，并将该provide:=true指令放到导出包上，例如：Export-Package: org.osgi.service.event; provide:=true。
* 每次当你对外发布一个bundle，触发的是bundle的次版本。对于内部开发发布来说，触发的是bundle的微版本。

## 10.2 Versions in OSGi OSGi版本
A version in OSGi has 4 parts:

OSGi中的一个版本有4个部分：

  major        1
  minor        1.1
  micro        1.1.1
  qualifier    1.1.1.qualifier

  主版本 1
  次版本 1.1
  微版本 1.1.1
  限定符 1.1.1.qualifier
	
To survive versioning, one must have a ''version policy''. A version policy puts semantics on the version numbers. The ''recommended'' policy in OSGi is changing the part when there is:

为了拯救版本控制，一个软件必须要有一个“版本控制策略”，版本策略将语义体现在版本号上。OSGi推荐的版本控制策略是根据不同的改变变更版本的不同部分：

  major        a breaking change
  minor        a backward compatible changes
  micro        a bug fix (no API change)
  qualifier    a new build

  主版本		   破坏前后兼容性变化
  次版本  	   向后兼容性变化
  微版本		   bug的修复（非API变化）
  限定符        新的构建

In OSGi, the decision was taken to have a single export version. The import statement allows a version range to be set. For example:

在OSGi中，决定采用单一的导出版本。import语句允许设置版本范围。例如：

    Export-Package: com.acme.foo; version=1.0.2
    Import-Package: com.acme.bar; version="[1,2)"

The semantic versioning white paper introduces two terms that are orthogonal to the imports and exports as well as implementing or delegating:

语义版本白皮书介绍了正交的两个术语：

* Provide an API - Provide the functionality defined in an API
* Consume an API - Consume the functionality defined in an API

* 提供API - 提供API中定义的功能
* 消费API​​ - 消费API中定义的功能

Provide and consume is orthogonal to implementing an interface and delegating. For example, the Configuration Admin service has the `ConfigurationAdmin` interface that is implemented by the Provider of an API but the `ConfigurationListener` interface is implemented by the Consumer of the API.

提供和消费正交于接口的实现与委派。例如，Configuration Admin service有`ConfigurationAdmin`这个接口，它被一个API的提供者所实现，但是它的另一`ConfigurationListener`接口被一个API的消费者所实现。

The reason for the providers and consumer terms is that version policies are different. A change in an API almost always affects the provider but with careful API design it is often possible to make a change backward compatible for consumers.

用提供者和消费者这两个术语的原因是因为它们的版本控制策略是不同的。一个API的改变总是会对提供者造成影响，但是通过仔细的API设计，通常可以让这个改变对消费者向后兼容。

In bnd, whenever you have to provide an import range, you can use modifiers to create a range out of a single version:

在bnd,无论何时你想要提供一个导入的范围，你可以用修饰符从单个版本中创建范围：

* `@1.2.3` – Creates an import up to the next major version: `[1.2.3,2.0.0)`.
* `1.2.3@` – Creates an import up to the next minor version: `[1.2.3,1.3.0)`.
* `=1.2.3` – Creates a range that only accepts that version: `[1.2.3,1.2.3]`.

* `@1.2.3` – 创建一个导入从当前版本到下一个主版本：`[1.2.3,2.0.0)`.
* `1.2.3@` – 创建一个导入从当前版本到下一个次版本：`[1.2.3,1.3.0)`.
* `=1.2.3` – 创建一个导入只接受当前版本：`[1.2.3,1.2.3]`.

## 10.3 Versioning Packages 包的版本控制
If you have a package that is containing implementation code that is supposed to be directly used by the consumers then this is a ''library''.

如果你有一个包含实现代码的包，你希望它直接被消费者所使用，那这个包就被称为一个'库'。

A library package is not an API that can be implemented by other bundles, it is the implementation. Then the versioning of library packages is relatively straightforward: Any change that breaks a consumer of that package must increment the major version number. For example, if the popular ASM library would add a method to the `MethodVisitor` class then it must increment the major version number of the `org.objectweb.asm` package because all existing consumers of this library would then be broken.

一个库包不是一个API，它不能被其他bundles所实现，它是一个实现。那么库包的版本控制相对来说比较简单：任何破坏这个包消费者的改变都应该增加主版本号。例如，如果流行的ASM库想向MethodVisitor类添加一个方法，则它必须增加`org.objectweb.asm`包的主版本号，因为该库的所有现有的消费者将会被破坏。

If however a package contains an API that is provided and consumed by others the situation is more complex. In such a case, the provider should export the API package and the consumers should import it.

然而，如果你有一个包含由其他人提供和使用的API的包，情况就会比较复杂，在这种情况下，提供者应该导出API包，消费者应该导入它。

bnd explicitly allows the inclusion of packages that come from other projects. It is just good practice to include an API package in your bundle if you are the provider of that API. However, this means that maintaining the version of the package in the manifest is ''wrong'', it would have to be maintained in several places, which is very error prone.

bnd明确允许包含来自其他项目的包。如果您是该API的提供者，那么在您的软件包中包含一个API包就是很好的做法。但是，这意味着在清单中维护包的版本是“错误的”，它必须在几个地方进行维护，这很容易出错。

For this reason, bnd provides a way to store the version of the package together with the package itself. One with annotations and one without when annotations are not possible.

因此，bnd提供了一种存储包的版本以及包本身的方法。一个有注释，一个没有注释当注释不可用时。

The @Version annotation is placed on the package. Since Java 5 it is possible to create a package-info.java file that can be used to annotate packages:

把@Version注释放在包上。从Java 5开始，就可以创建一个用于注释软件包的package-info.java文件：

    package-info.java:
        @Version("1.2.0.${build}")
        package com.example;

        import aQute.bnd.annotation.Version;

A non-annotation based alternative is the `packageinfo` file. When bnd scans the Java archives it will look in each package for this packageinfo file. The format of this file is very simple:

一个基于非注释的选择是 `packageinfo`文件。当bnd扫描java存档时，它会查看每个包中的packageinfo文件，这个文件的格式非常简单：

    packageinfo:
        version 1.2.0.v${build}

In either case, the value for the version may contain macros.

在任一情况下，版本的值都可能包含宏。

If you now export the package (from any bundle that has the package on its class path), it will be properly versioned.

如果你现在想导出这个包（从任何在class path上包含这个package的bundle）,它将会被正确地版本化。

    bnd.bnd:
        build = ${tstamp}
        Export-Package: com.example.*

The resulting manifest will look like:

结果清单将会如下所示：

    Manifest:
        Export-Package: com.example; version=1.2.0.v201010101010

If you export a a package from another bundle, bnd will also look in the manifest of that other bundle for a version.

如果你从其他bundle中导出一个包，bnd也会从其他bundle的清单中找到其版本。

Using packageinfo (or the @Version annotation) is highly recommended.

强烈推荐使用packageinfo（或者@Version注释）

## 10.4 When Package Version Differ 当包版本不同时

The package version for package p can come from the following places (in order of increasing priority):

一个包的版本可能来自以下位置（按优先级顺序排列）：

* `p/packageinfo`
* `p/package-info.java` with an annotation
* Manifest of the source bundle
* Manifest of the bundle under construction, i.e. the bnd.bnd file

* `p/packageinfo`
* `p/package-info.java`的版本注释
* 源bundle的清单
* 正在构建的bundle的清单，即bnd.bnd文件

The bnd warning means that bnd finds multiple definitions of the version for p and they are not the same. So I assume you set the version in the manifest of the bundle under construction and in one of those other places. The best is to remove the version from your bnd.bnd file. The by far absolute best way is to only set the version of the package in the package directory (either `packageinfo` or `package-info.java` with an annotation).

bnd警告意味着bnd找到包版本的多个定义，并且它们不一样。所以我假设你将版本设置在正在构建的bundle的清单中和上述的其他一个位置。那么你最好是从bnd.bnd文件中删除版本。到目前为止最佳方式绝对是只在包目录中设置包的版本（packageinfo或者package-info.java版本注释）。

## 10.5 Import Version Policy 导入版本策略
If you import a package bnd will look at the exported version of that package. This version is not directly suitable for the import because it is usually too specific, it needs a policy to convert this export version to an import version.

如果你导入一个包,bnd会查找这个包的导出版本。但是这个版本不适合直接导入，因为它太具体了，需要一个策略将导出版本转化为导入版本。

An importer that provides the functionality of an API package is much closer tied to that package than a client. The whitepaper recommends binding to the major.minor part of the version for a provider. That is, any change in the minor part of the version breaks the compatibility. This makes sense, the provider of an API must implement the contract and is therefore not backward compatible for any change in the API. A consumer of the API only has to be bound to the major part because it is much more relaxed for the backward compatibility.

提供API包功能的导入者比客户端更紧密地与该包相关。白皮书推荐绑定提供者的次版本。 这是因为，该版本的次要部分的任何更改会破坏兼容性。这是有道理的，API的提供者必须实现合同，因此API的任何更改不会向后兼容。API的消费者只需要绑定到主版本，因为它对于向后兼容性要更加宽松。

For example, a new method is added to an interface that is implemented by the provider of the API. Old clients have no visibility of this method because when they compiled it did not exist. However, the provider of the API must be modified to implement this method otherwise more modern clients would break.

例如，如果一个接口中增加了一个新方法，提供者实现了这个API。老的客户端是看不见这个方法的，因为它们编译的时候不存在这个方法。然而，这个API的提供者必须修改来实现这个方法，否者更多的新的客户端将会被破坏。

This asymmetry creates the need for two version policies:

这种不对称形式需要两个版本策略：

    -provider-policy :    ${range;[==,=+)}
    -consumer-policy :    ${range;[==,+)}

The given values are the defaults. The value of the version policy will be used calculate the import based on the exported package. The `${range}` macro provides a convenient shortcut to do this using a version mask.

给定的值是默认的。基于导出包的版本策略值将会被用于计算导入范围。${range}宏利用版本面具提供了一个方便的快捷方式来做到这一点。

For example, a bundle that implements the OSGi Event Admin service can use the following bnd file:

例如，一个实现了OSGi Event Admin service的bundle可以用以下的bnd文件：

    bnd.bnd:
        Private-Package: com.example.impl.event

The resulting manifest would look like:

得到的清单将如下所示：

    Manifest:
        Import-Package:  org.osgi.service.event; version="[1.1,2)", ...
        ...

How does bnd know if a bundle is a provider or a consumer of a specific package? Well, the default is the consumer policy but this can be overridden with the `provide:=true` directive that works on the `Import-Package` clauses as well as on the `Export-Package` clauses. 

bnd如何知道bundle是一个特定包的提供者还是消费者呢？其实，默认的是消费者策略，但是这个可以通过覆盖 `provide:=true` 指令来进行修改，这个方法对`Import-Package`和`Export-Package` 子句都有效。

The `provide:` directive indicates to bnd that the given package contains API that is provided by this bundle. The (strongly) recommended way is to put the `provide:=true` directive on the `Export-Package` header, even if the package comes from another bundle. This way the bundle contains a copy of the package that is by default imported with the proper provider policy range.

`provide:`指令告诉bnd这个给定包含有的API是被这个bundle提供的。强烈推荐把`provide:=true` 指令放在`Export-Package`header上，即使这个包来自于其他bundle.用这种方法，这个bundle会包含这个包的副本，在默认情况下，这个副本会根据合适的提供者策略范围被导入进来。


For example, an implementation of the OSGi Event Admin specification could use the following bnd file:

例如，OSGi Event Admin的实现可以用如下bnd文件：

    bnd.bnd:
        Export-Package:  org.osgi.service.event; provide:=true
        Private-Package: com.example.impl.event

The resulting manifest would look like:

得到的清单将如下所示：

    Manifest:
        Export-Package:  org.osgi.service.event; version=1.1
        Import-Package:  org.osgi.service.event; version="[1.1,1.2)", ...
        ...

If for some reason it is not desirable to export the API package in the implementation bundle, then the `provide:` directive can also be applied on the `Import-Package` header:

如果由于一些原因在实现中导出API包是不可取的，那么`provide:`指令可以应用在`Import-Package`header上：

    bnd.bnd
        Import-Package: org.osgi.service.event; provide:=true, *
        Private-Package: com.example.impl.event

The resulting manifest would look like:

得到的清单将如下所示：

    Manifest:
        Import-Package:  org.osgi.service.event; version="[1.1,1.2)", ...
        ...

## 10.6 Substitution 置换
A key aspect of OSGi is that a package can be both imported and exported. The reason is that this feature allows a framework more leeway during resolving without creating multiple unconnected class spaces.

OSGi的一个关键方面是包既可以导入又可以导出。之所以重要是由于此功能允许框架在解析过程中有更多的余地，而不用创建多个未连接的类空间。

After the bundle has been created and analyzed bnd will see if an exported package is eligible for import. An export is automatically imported when the following are true:

在bundle被创建和分析之后，bnd会去看导出的包是否有资格导入。如果满足以下条件，导出将自动导入：

* There exists at least one reference to the exported package from a private package
* The exported package has no references to any private packages
* The exported package does not have a `-noimport:` directive.

* 私有包中至少含有对这个导出的包的一个引用
* 这个导出包没有引用任何私有包
* 这个导出包没有`-noimport:`指令

If a package is imported it will use the version as defined by the version policy.

如果一个包被导入，那么它将使用版本策略定义的版本。

## 10.7 Versioning Bundles bundle版本控制
Versioning bundles usually requires bumping the version every time it is placed in a repository. When package versioning is used, the bundle version is only important for tracking an artifact.

bundle版本控制通常需要在每次将其存入库时触发版本。当使用包版本控制时，bundle版本只对跟踪artifact非常重要。

## 10.8 Why does bnd use only major and minor version component in import-package headers? 为什么在bnd import-package headers中只使用主要和次要版本模块？

The micro is left out because it generates a lot of unnecessary releases, this is similar to the maven release process. If you connect everything 100%, you cannot move anything unless all its dependencies are moved at the same time. We actually tried in the OSGi build to use micro version changes for default methods in Java 8 but found that it just creates an enormous ripple through effect in the build. Not depending on the micro version is a lubricant that does not kill any bundle out there that depends on you.

微版本被遗漏的原因是它产生了大量的不必要版本，这与maven的发布过程类似。如果连接100％的所有内容，除非所有的依赖项都被同时移动，否则你不能移动任何东西。实际上，我们在OSGi构建中尝试使用Java 8中微版本更改的默认方法，但发现它只是在构建中效果创建了巨大的波动。不依赖于微版本是一种润滑剂，它不会杀死依赖于你的任何bundle。

This should not be a problem because a micro version is a deployment issue since the semantic versioning should be used for APIs and a micro change is not visible in the API. 

这不应该不能算一个问题，因为微版本是一个部署问题，语义版本控制应该用于API，微版本的变化影响不到API。

That said, this is bnd so obviously you can override it. You can override the default version policy is:

这是bnd，所以显然你可以覆盖默认版本策略：

	-provider-policy = ${range;[==,=+)}
	-consumer-policy = ${range;[==,+)}

Just set '===' instead of '==' for the floor version in your pom.xml in the <configuration> section and you should be ok.

只要在你的pom.xml的floor版本中用'==='代替'==' 部分，就可以使用微版本了。

	<configuration>
		<_provider-policy>${range;[===,=+)}</_provider-policy>
		<_consumer-policy>${range;[===,+)}</_consumer-policy>
	</configuration>

