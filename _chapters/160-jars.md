---
# 9 Generating JARs 生成JAR包
---

This is about generating JARs

这章的主要内容是如何利用bnd生成JAR包

This is a simple example of wrapping a jar with bnd. The basic idea is to create a recipe (a .bnd file) that collects the different resources in the right way.

下面是一个简单的例子，它将阐述如何用bnd打JAR包。其中的基本思想是创建一个清单文件（例如 .bnd文件）来收集不同的资源。

For example, you want to wrap the WebSocket server from https://github.com/TooTallNate/Java-WebSocket. Download the binary and the sources in Websocket.jar, Websocket-src.zip. Once you have these files, the following code will create the org.websocket bundle

例如，你想打包https://github.com/TooTallNate/Java-WebSocket上的WebSocket服务器。下载Websocket.jar和Websocket-src.zip文件，下面的代码将帮助你创建org.websocket bundle。



    # Wrapped version of Github project TooTallNate/Java-WebSocket
    Bundle-SymbolicName: org.websocket
    Bundle-DocURL: https://github.com/TooTallNate/Java-WebSocket
    Bundle-License: https://github.com/TooTallNate/Java-WebSocket/blob/8ef67b46ecc927d5521849dcc2d85d10f9789c20/LICENSE
    Bundle-Description: This repository contains a barebones \ 
     WebSocket server and client implementation written \ 
     in 100% Java. The underlying classes are implemented \ 
     using the Java ServerSocketChannel and SocketChannel \ 
     classes, which allows for a non-blocking event-driven model \ 
     (similar to the WebSocket API for web browsers). \ 
     Implemented WebSocket protocol versions are: Hixie 75, \ 
     Hixie 76, Hybi 10, and Hybi 17

    # websocket does not define a version yet :-(
    Bundle-Version: 1.0.2

    -includeresource: @jar/WebSocket.jar, OSGI-OPT/src=@jar/WebSocket-src.zip
    -exportcontents: org.java_websocket


The documentation headers are optional but very important, just spent the minute to document them since you'll be grateful later.


文档的headers你可以选择不写，但是它也是非常重要的。你只要花几分钟就可以写完，之后会为你带来很多便利。

If the target project does not have a version, makeup a version and maintain it. Notice that in general the recipe will only be used once for each version, it is normally not used in continuous integration builds. However, you normally use it to convert the next version of the project. Crisp versioning is important.

如果需要打包的目标项目没有写版本，我们可以为它指定一个版本并进行维护。需要注意的是，通常情况下，每个版本的清单只能使用一次，不适合在连续集成构建中使用。有时你可以用它来把项目转换到下一个版本，清晰的版本控制很重要。

The `Include-Resource` statement unrolls the jars we downloaded in the root of the JAR and in `OSGI-OPT`. Since the source code is in the `src` directory in the  `WebSocket-src.zip` file, we put it in the new JAR under `OSGI-OPT/src`. This convention is supported by all IDEs to give you direct access to the bundle's source code. Since the binary and the source are kept together, you always have the correct source code available, and usually automatically. It is so convenient that once you're used to this it is hard to imagine a life without source code.

在Include-Resource声明中，我们展开了下载在根目录和OSGI-OPT目录下的JAR包。由于源代码在WebSocket-src.zip中的src目录中，我们将其放在新的JAR下OSGI-OPT/src目录中。所有IDE都支持此约定，以便您直接访问该bundle的源代码。由于二进制代码和源代码放在一起，因此你始终可以使用正确的源代码，通常是自动的。这很方便，一旦你习惯了，很难想象没有源代码的生活。

The binaries and sources are not in the final jar but bnd does not yet know what needs to be exported. This can be indicated with the `-exportcontents` instruction. It has the same syntax as `Export-Package` but does not copy from the classpath, it only applies the instruction to the content of the final JAR.

二进制文件和源代码不在最终生成的jar中，但是bnd现在还不知道需要导出什么。这可以用-exportcontents指示来指定。它与Export-Package具有相同的语法，但不从类路径复制，它只将指令应用于最终JAR包。



## 9.1 Project 项目
The easiest way to build these wrappers is to create a project in bndtools called wrappers and create a bnd descriptor for each one. They are then automatically build (look in generated) and you get a lot of help editing the bnd files.

打包最简单的方法是用bndtools创建一个项目，然后在这个项目中创建一个bnd描述文件。它就会自动进行构建并打包，同时用bndtools还可以很方便地编辑bnd文件。

You can also make an ant file but this is not described here (volunteer?). The other possibility is to use bnd from the [[CommandLine]]. In this case the command is:

你也可以选着在项目中加一个ant文件，但是在这里就不细说了。另一种可能的做法是在命令行使用bnd打包。上面的例子可以使用如下命令：

    bnd websocket.bnd

## 9.2 Manifest 清单
Applying this recipe gives the following manifest in a JAR named `org.websocket.jar`:

使用上面的配方可以给名为org.websocket.jar的JAR生成以下的清单

    Manifest-Version: 1.0
    Bnd-LastModified: 1338190175969
    Bundle-Description: This repository contains a barebones WebSocket serve
     r and client implementation written in 100% Java. The underlying classe
     s are implemented using the Java ServerSocketChannel and SocketChannel 
     classes, which allows for a non-blocking event-driven model (similar to
     the WebSocket API for web browsers). Implemented WebSocket protocol ve
     rsions are: Hixie 75, Hixie 76, Hybi 10, and Hybi 17
    Bundle-DocURL: https://github.com/TooTallNate/Java-WebSocket
    Bundle-License: https://github.com/TooTallNate/Java-WebSocket/blob/8ef67
     b46ecc927d5521849dcc2d85d10f9789c20/LICENSE
    Bundle-ManifestVersion: 2
    Bundle-Name: org.websocket
    Bundle-SymbolicName: org.websocket
    Bundle-Version: 1.0.2
    Created-By: 1.6.0_27 (Apple Inc.)
    Export-Package: org.java_websocket;version="1.0.2"
    Include-Resource: @jar/WebSocket.jar, OSGI-OPT/src=@jar/WebSocket-src.zip
    Private-Package: org.java_websocket.handshake,org.java_websocket.drafts,
     org.java_websocket.exceptions,org.java_websocket.util,org.java_websocke
     t.framing
    Tool: Bnd-1.51.0

## 9.3 Extra entries on the Classpath 类路径中的额外条目
One of the great features of bnd is to use export version from other versions to generate the import ranges. This feature requires that the other JARs are on the classpath. In bndtools you can use the -buildpath. However, you always add entries on the class path per bnd descriptor with the -classpath instruction:

bnd的一个重要功能是使用其他版本的导出版本来生成导入范围。此功能要求其他JAR位于类路径上。在bndtools中，您可以使用-buildpath。但是，如果您需要在bnd描述文件上添加条目，则要使用-classpath指令：

    -classpath: dependency.jar, other.jar


