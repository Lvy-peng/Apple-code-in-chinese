iOS 调试魔法
===


[原文地址](https://developer.apple.com/library/content/technotes/tn2239/_index.html)

翻译人:pzm

iOS 有很多秘密的调试工具，包括环境变量，设置，程序等等。这篇文档主要描述这些工具。如果你是一名 iOS 的开发者，可以看下这些可能被你遗漏掉的让你爽歪歪的方法。

## 简介
苹果系统包含苹果工程师添加的一些调试工具，以便于我们开发和调试特殊场景。这些工具在系统软件中，你可以使用他们去调试你的程序。这篇文档讲一下比较有用的工具。调试文档记录在别的地方的情况下，会对工具做一个概述，并且会有一个指向这个文档的链接。

>这份总结并不详细，不包括所有的调试工具。

这篇文档中的细节可能包含不同的平台和不同的版本，你可能会遇到一些平台上的差异，或者新旧系统上的。一些已知的重要改动文档中会提。

>本文中提到的调试工具并不是不变的，苹果有权随着 OS 的更新对其进行更改或者更新。过去发生过，以后也很可能发生。这些东西只是调试通用的，你发布app的时候不能依赖本文介绍的这些功能。

除了二进制的兼容问题，你应该记住 iOS 的程序必须遵守各种法律规定和 APP Store 的审核规则。

>提示: 如果你也开发 Mac OS X,你可以阅读[Technical Note TN2124, ‘Mac OS X Debugging Magic’](https://developer.apple.com/library/content/technotes/tn2124/_index.html)

此文包含一些调试术语，如果你刚开始学习这些，那下面的这些你应该查阅下:
* GDB 是系统主要的调试工具，[Beguging with GDB]（https://developer.apple.com/library/content/navigation/）了解 GDB 可以看这里。

* Xcode 是苹果的 IDE。 它包含复杂惊喜的调试器。是一个 GDB 的包装。了解更多的 Xcode 看 苹果文档 [Reference Library](https://developer.apple.com/library/content/navigation/) 中的 "Tools & Languages" 章节。

这篇文档没有提到性能调试，如果你想调试下性能，你可以看 [Getting Started with Performace] （链接竟然有问题-。-）


## 基础
接下来的几章讲调试工具的细节，很多工具使用相似的技术开启，关闭和查看日志。这涨讲述下这些相似的技术。

###开启调试工具
有些调试工具默认开启,然而，也有很多的工具需要使用下面的方法去开启。

####环境变量
很多情况下你可以通过设置一个特殊的环境变量开启调试。你可以使用 Xcode 的可执行文件的监察器，例1如下:

例1：在 Xcode 中设置环境变量

![Figure1 icon](https://developer.apple.com/library/content/technotes/tn2239/Art/tn2239_XcodeEnv.png)

####设置
有些调试工具的使用要使用一些特殊的设置。你可以在 Xcode 中配置一个命令行参数。例2展示了这么做：

例2: 在 Xcode 中设置命令行参数
![Figure2 icon]https://developer.apple.com/library/content/technotes/tn2239/Art/tn2239_XcodeArgs.png)

####可调用接口
很多系统框架包含接口调试信息的打印输出。GDB 可能设计这些接口是可调用的，或者他们只是现有的调试时有用的 API。表1举例怎么用 GDB 调用调试接口。很明显，他调用 CFBundleGetAllBundles 得到程序加载的所有 bundles 表，然后调用 CGShow 打印出来。

表1:GDB 调用调试接口

	`(gdb) call (void)CFShow((void *)CFBundleGetAllBundles())
	<CFArray 0x10025c010 [0x7fff701faf20]>{type = mutable-small, count = 59, values = (		
    0 : CFBundle 0x100234d00 </System/Library/Frameworks/CoreData.framework> …
    […]
    12 : CFBundle 0x100237790 </System/Library/Frameworks/Security.framework> …
    […]
    23 : CFBundle 0x100194eb0 </System/Library/Frameworks/CoreFoundation.framework> …
    […]
	)}`
	
>当这样调用接口时， GDB 必须要知道方法的返回类型 (因为一些返回类型可能由传入的参数决定)。如果你调用一个方法没有调试符号，你要通过添加一个属性来告诉 GDB 返回类型。例如， 表1列出了返回类型是 CFBundleGetAllBundles 强转成 (void *) 和 CFShow 强转成 (void)。

同样的，如果你调用了没有标准参数的方法，你需要列出你的参数确保 GDB 正确传参。
如果你没有看到方法的输出，你可以看下控制台日志，会在设置调试输出章节介绍。

>重要:如果你在你自己的代码上使用这个技巧，应当注意他不对所有的 static 方法生效。因为编译器的过程优化可能造成 static 方法不同于程序二进制接口，这称为二进制方法，这个方法不能保证可靠的被 GDB 调用。 

实际上，这个只适用于intel 32位，iPhone模拟器上。


	

	
















