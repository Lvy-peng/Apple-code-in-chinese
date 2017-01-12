理解和分析应用程序崩溃报告
===

[原文地址](https://developer.apple.com/library/content/technotes/tn2151/_index.html#//apple_ref/doc/uid/DTS40008184)

翻译人：pzm 翻译时间：2017/1/12

当应用程序崩溃时，崩溃报告对于理解程序为什么会崩溃起到了很重要的作用。这篇文档包含了重要的信息，关于我们如何去符号化，理解和解释崩溃报告。

###概述
当程序崩溃时，设备会生成并保存一份崩溃报告。崩溃报告描述了程序在什么情况下终止。很多情况下都包含完整的线程的调用栈，对于调试应用程序信息很有用。你可以看这些崩溃报告理解你的程序的崩溃点，并且尝试去修复他们。

在分析报告中的调用栈时应先对他们进行符号化。符号化会将内存地址转变为函数名和行数。如果你从 Xcode 的 Devices 窗口拿到设备的崩溃日志，那 Xcode 很快就会自动在几秒钟之内进行符号化。否则你就需要自己符号化这些崩溃日志。细节可以参考 [Symbolicating Crash Reports](https://developer.apple.com/library/content/technotes/tn2151/_index.html#//apple_ref/doc/uid/DTS40008184-CH1-SYMBOLICATION).

内存告警报告不同于其他的崩溃报告，它没有现成的调用栈。当内存告警导致了崩溃时，你需要查看你的内存使用情况和收到内存告警时的调用函数。这个文档指出了几个内存管理的参考或许你会觉得有用。

###捕获崩溃和低内存报告
[Debugging Deployed iOS Apps](https://developer.apple.com/library/content/qa/qa1747/_index.html) 讨论了怎样去恢复崩溃和内存报告通过 iOS 设备。

[Analyzing Crash Reports](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/AnalyzingCrashReports/AnalyzingCrashReports.html) 在 [App Distribution Guide](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/Introduction/Introduction.html) 讨论了怎么去	怎么样去看从 TestFlight 下载了应用程序的测试用户手机中收集到的崩溃报告。

###符号化崩溃报告
符号化是一个解析调用栈转化为源文件方法和函数名的过程。如果没有符号化就很难定位到产生崩溃的地址。

>低内存报告不需要被符号化

> macOS 的崩溃报告通常在生成的时候被符号化或者部分符号化。这一部分我们主要探讨 iOS , watchOS, 和 tvOS 上的崩溃报告，总的来说过程与 macOS 相似。

* 1 崩溃报告和符号化过程的概览
  ![process icon](https://developer.apple.com/library/content/technotes/tn2151/Art/tn2151_crash_flow.png)
  
 1.编译器将你的源代码转化为二进制码，同时生成调试信息，当源代码被调用时映射源代码的每一行为机器的二进制码。依赖 Debug Infofermation Format  编译设置，这些调试符号保存包含二进制在 DSYM 文件中。默认情况下，debug 的应用程序在二进制程序中包含调试符号，然而 release 版本的应用程序在 DSYM 文件中保存调试符号信息，这样是为了减少二进制文件的大小。
 
 调试符号文件和应用程序的二进制文件被捆绑在一起，通过构建 UUID 。每一个构建应用程序都会生成一个新的 UUID，唯一的标识这一次构建。即使构建相同的源代码，使用一样的编译设置，都会生成不同的 UUID. 调试符号的文件在随后的构建里，即使是相同的源文件也不会与其他的构建文件混淆。

2. 将应用程序归档以进行分开时，Xcode 将收集应用程序的文件通过 DSYM 文件并且在你的 home 文件夹里保存他们。你可以在已经归档的 Xcode 组织者的应用程序中在 "Archived" 分类下找到你的程序。创建一个存档更多信息可以参考[App Distribution Guide](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/Introduction/Introduction.html)

>重要：从测试者，app 审核的人和用户中符号化崩溃文件，你必须保留文件在你发布的应用程序中。



















