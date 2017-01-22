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

####配置文件
一些 iOS 的子系统有调试工具，可以通过安装一个特殊的配置文件开启。Push Notification 就是一个例子。通常这样安装一个配置文件:
* 放在 web 服务器上面，然后在设备的 Safari 上面下载。
* 放在邮件的附件中，发送给设备的关联账户，然后在 Mail中打开附件中的配置文件。

关于配置文件你可以看 [iPhone in Business](http://www.apple.com/iphone/business/).

#### 查看调试输出
程序通常使用如下机制生成调试日志:
* NSLog
* 打印到stderr(标准错误)
* 系统日志

NSLog 是一个高级 API 用于输出的，在 Objective-c 代码中被广泛使用。NSLog 的具体行为惊人的复杂，随着时间的过去，它已经被改变很多了。谈论他超出了这篇文档的范围。然而，要知道 NSLog 打印输出流，或者系统日志，或者两者都有。如果你理解这两种机制，你可以通过 NSLog 看任何你想查看的。

打印到 stderr 是最常用的实用输出机制之一。考虑到这个话题的重要性，将在降下来的章节中深入讨论。

查看系统日志最简单的方法是在 Xcode 的 Organizer 窗口看控制台输出。如果有人不想安装 Xcode,可以看系统日志使用[iPhone Configureration Utility](https://support.apple.com/business-education)

####控制台输出
很多程序，事实上是很多的系统框架，打印调试信息到 stderr. 这个输出的目的最终由程序决定。他可以重定向 stderr 不管选择的目的是什么。然而，在大多数的情况下，一个程序不会重定向 stderr，输出到哪里由程序启动环境决定。通常是下列之一:
* 如果由普通用户发起一个 GUI 应用程序，系统将重定向打印到 stderr 系统日志的任何消息。你可以使用前面描写的技术查看这些消息。
* 如果你用 Xcode 运行一个程序，你可以看到 stderr 输出在 Xcode 的 调试控制窗口 (选择控制菜单的 Run 菜单去看这个窗口)。

连接到正在运行的程序 (使用 Xcode 的 附加进程菜单或者 GDB 的 attach 命令)不会自动将程序的 Stderr 连接到你的 GDB 窗口。你可以从 GDB 使用技巧这样做在[Technical Note TN2030,‘GDB for MacsBug Veterans](https://developer.apple.com/library/content/navigation/index.html#topic=Technical+Notes&section=Resource+Types)的 ”Seeing stdout and stderr After Attching“ 章节。

####一些汇编需要
尽管现在写大量的汇编语言已经不再寻常，但是依然有必要对他有一些基础的了解。尤其在你调试的时候，尤其在调试那些发生在frameworks 的崩溃，你并没有他们的源代码。这一章讲一些在汇编级别基本的技巧去调试程序。主要讲了怎么去设置断点，读取参数和在支持的架构里访问返回值。

区别是这篇文档中的汇编级别的举例在 iPhone 3GS armv7 架构。然而，大多数情况下，来自不同架构的例子差异并不明显，既是是在 Mac 上。这些例子很容易适配到其他架构，重要的区别是：
* 读取参数
* 得到返回地址

这些在一下的体系结构章节会提到。

>重要: 以下特定架构章节包含一些规则，如果方法没有任何标准参数，或者标准函数返回值，这些规则就并不适用，你需要自己去查阅文档里的细节。

在这种情况下，标准参数包括整数(也适用于寄存器)，枚举，指针(包含数组指针和函数指针)。不标准的参数包括浮点数，向量，结构体，比一个寄存器大的整数，或者在固定参数之后的任何可变参数。

iOS 的详细调用约定，可以看 [iOS ABI Function Call Guide](https://developer.apple.com/library/content/documentation/Xcode/Conceptual/iPhoneOSABIReference/Introduction/Introduction.html).iPhone 模拟器在 32 位架构上可以看 [Mac OS X ABI Function Call Guide](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/LowLevelABI/000-Introduction/introduction.html).

当你看接下来的章节前，理解 GDB 的微妙很重要。因为GDB是源代码调试的心脏。当你在函数中设置断点，GDB 不能在第一个执行指令处，即汇编指令设置断点。但是，它将断点设置在 prologue 后的第一条指令上。从源代码调试级别看下这么做的意义。源代码级别调试器不会想跳过函数的 prologue,然而，在汇编级别的调试，很容易在 prologue 之前读取参数。这是因为在程序指令中参数的位置由叫[ABI](https://en.wikipedia.org/wiki/Application_binary_interface) 的函数决定,即程序二进制接口。但是 prologue 被允许打乱他们的顺序。而且，每一个 prologue 可以以不一样的方式这么做。所以只有一种方法读取参数在 prologue 之后，那就是反汇编 prologue 看看都去哪里了。这通常并不总是很简单，然而这毕竟是额外的工作。 （自己查的:为了实现函数调用， 编译器会在每个子函数头部加入 prologue.GDB执行 step 命令进入子函数时，会跳过prologue，将断点设置在 prologue后的第一条指令上） 

让 GDB 设置断点在函数的第一条指令的最好方法是在函数名称之前加 ”*“。表二展示了例子。

表二：在 prologue 之前和之后
  <pre>(gdb) b CFStringCreateWithFormat	
  	Breakpoint 1 at 0x34427ff8
	(gdb) info break	
	Num Type           Disp Enb Address    What
	1   breakpoint     keep n   0x34427ff8 	<CFStringCreateWithFormat+8>	
	-- The breakpoint is not at the first instruction.
	-- Disassembling the routine shows that GDB has skipped 	the prologue.
	(gdb) x/5i CFStringCreateWithFormat
	0x34427ff0 <CFStringCreateWithFormat>: push    {r2, r3}
	0x34427ff2 <CFStringCreateWithFormat+2>: push    {r7, lr}
	0x34427ff4 <CFStringCreateWithFormat+4>: add r7, sp, #0
	0x34427ff6 <CFStringCreateWithFormat+6>: sub sp, #4
	0x34427ff8 <CFStringCreateWithFormat+8>: add r3, sp, #12
	-- So we use a "*" prefix to disable GDB's 'smarts'.
	(gdb) b *CFStringCreateWithFormat
	Breakpoint 2 at 0x34427ff0
	(gdb) info break
	Num Type           Disp Enb Address    What
	1   breakpoint     keep n   0x34427ff8 	<CFStringCreateWithFormat+8>
	2   breakpoint     keep n   0x34427ff0 	<CFStringCreateWithFormat>
	</pre>
	
	
>重要:因为 iOS 没有办法用命令行运行程序，表二取自 Xcode 的控制台窗口。而且因为控制台窗口不支持字符 '#',用 '--' 表示。如果你在控制台复制这些例子，不要输入这些横线。

于此相反，文档中的其他例子都在Mac OS X 上，所以使用了传统的 GDB 命令行。
最后，如果你要寻找特定的指令，注意 Shark 的帮助(包含在 Xcode 的开发者 工具)有一个指令指南，包含Intel 和 PowerPC 架构。

####AARM
ARM 程序首先把四个参数用寄存器传递，返回地址在寄存器LR里。列表1展示了怎么用 GDB 获取这些值当你停在函数的第一条指令时。

列表1: 在 ARM 中读取参数

|意义| GDB 语法 |
|---------|:-------:|
| 返回地址  |  $lr|
|第一个参数 |  $r0|
|第二个参数 |  $r1|
|第三个参数 |  $r2|
|第四个参数 |  $r3|

函数返回结果在寄存器 R0($r0)中。
因为参数在寄存器中传递，所以没有直接的方法在prologue 之后读取参数。
表三举了例子关于怎么利用这个信息使用 GDB 读取参数。

表三: ARM 的参数
<pre>-- We have to start the program from Xcode. Before we do that, we go to 
-- Xcode's Breakpoints window and set a symbolic breakpoint on 
-- CFStringCreateWithFormat.
GNU gdb 6.3.50-20050815 […]
-- We've stopped after the prologue.
(gdb) p/a $pc
$1 = 0x34427ff8 <CFStringCreateWithFormat+8>
-- Let's see what the prologue has done to the registers 
-- holding our parameters.
(gdb) x/5i $pc-8
0x34427ff0 <CFStringCreateWithFormat>: push    {r2, r3}
0x34427ff2 <CFStringCreateWithFormat+2>: push    {r7, lr}
0x34427ff4 <CFStringCreateWithFormat+4>: add r7, sp, #0
0x34427ff6 <CFStringCreateWithFormat+6>: sub sp, #4
0x34427ff8 <CFStringCreateWithFormat+8>: add r3, sp, #12
-- Hey, the prologue hasn't modified R0..R3, so we're OK.
--
-- first parameter is "alloc"
(gdb) p/a $r0
$2 = 0x3e801d60 <__kCFAllocatorSystemDefault>
-- second parameter is "formatOptions"
(gdb) p/a $r1
$3 = 0x0
-- third parameter is "format"
(gdb) call (void)CFShow($r2)
managed/%@/%@
-- It turns out the prologue has left LR intact as well.
-- So we can get our return address.
--
-- IMPORTANT: Bit zero of the return address indicates that it lies 
-- in a Thumb routine. So when using a return address in LR or on 
-- the stack, always mask off bit zero.
(gdb) p/a $lr & 0xfffffffe
$4 = 0x34427e18 <__CFXPreferencesGetManagedSourceForBundleIDAndUser+48>
-- Now clear the breakpoint and set a new one before the prologue.
(gdb) del 1
(gdb) b *CFStringCreateWithFormat
Breakpoint 2 at 0x34427ff0
(gdb) c
Continuing.

Breakpoint 2, 0x34427ff0 in CFStringCreateWithFormat ()
-- We're at the first instruction. The parameters are guaranteed 
-- to be in the right registers.
(gdb) p/a $pc
$5 = 0x34427ff0 <CFStringCreateWithFormat>
-- first parameter is "alloc"
(gdb) p/a $r0
$6 = 0x3e801d60 <__kCFAllocatorSystemDefault>
-- second parameter is "formatOptions"
(gdb) p/a $r1
$7 = 0x0
-- third parameter is "format"
(gdb) call (void)CFShow($r2)
%@/%@.plist
-- return address is in LR; again, we mask off bit zero
(gdb) p/a $lr & 0xfffffffe
$8 = 0x34427e7c <__CFXPreferencesGetManagedSourceForBundleIDAndUser+148>
(gdb) b *0x34427e7c
Breakpoint 3 at 0x34427e7c
-- Delete the other breakpoint to avoid thread-based confusion.
(gdb) del 2
-- Continue to the return address.
(gdb) c
Continuing.

Breakpoint 3, 0x34427e7c in __CFXPreferencesGetManagedSourceForBundleIDAndUser ()
-- function result
(gdb) p/a $r0
$9 = 0x1062d0
(gdb) call (void)CFShow($r0)
mobile/.GlobalPreferences.plist</pre>

####Intel 32位
32位架构用在 Iphone 模拟器。
32位的程序，参数在栈中传递。程序的第一条执行指令在栈顶包含返回地址。下一层包含第一个参数，再下一层包含第二个参数等等。列表二展示了怎么使用 GDB 获取这些值。

列表二: 获取参数在32位架构

|意义| GDB 语法 |
|---------|:-------:|
|返回地址 |  *(int*)$esp|
|第一个参数| *(int*)($esp + 4)|
|第二个参数| *(int*)($esp + 8)|
|...等等  |  |

函数的返回值在寄存器 EAX($eax).

表4 举例如何利用这个信息读取参数。

表4: 32位架构的参数
<pre>$ # Use the -arch i386 argument to GDB to get it to run the 
$ # 32-bit Intel binary.
$ gdb -arch i386 /Applications/TextEdit.app
GNU gdb 6.3.50-20050815 (Apple version gdb-1346) […]
(gdb) fb CFStringCreateWithFormat
Breakpoint 1 at 0x31ec6d6
(gdb) r
Starting program: /Applications/TextEdit.app/Contents/MacOS/TextEdit 
Reading symbols for shared libraries […]
Breakpoint 1, 0x940e36d6 in CFStringCreateWithFormat ()
(gdb) # We've stopped after the prologue.
(gdb) p/a $pc
$1 = 0x940e36d6 <CFStringCreateWithFormat+6>
(gdb) # However, for 32-bit Intel we don't need to inspect 
(gdb) # the prologue because the parameters are on the stack.
(gdb) # We can access them relative to EBP.
(gdb) #
(gdb) # first parameter is "alloc"
(gdb) p/a *(int*)($ebp+8)
$2 = 0xa0473ee0 <__kCFAllocatorSystemDefault>
(gdb) # second parameter is "formatOptions"
(gdb) p/a *(int*)($ebp+12)
$3 = 0x0
(gdb) # third parameter is "format"
(gdb) call (void)CFShow(*(int*)($ebp+16))
%@
(gdb) # return address is at EBP+4
(gdb) p/a *(int*)($ebp+4)
$4 = 0x940f59fb <__CFXPreferencesGetNamedVolatileSourceForBundleID+59>
(gdb) # Now clear the breakpoint and set a new one before the prologue.
(gdb) del 1
(gdb) b *CFStringCreateWithFormat
Breakpoint 2 at 0x940e36d0
(gdb) c
Continuing.

Breakpoint 2, 0x940e36d0 in CFStringCreateWithFormat ()
(gdb) # We're at the first instruction. We must access 
(gdb) # the parameters relative to ESP.
(gdb) p/a $pc
$6 = 0x940e36d0 <CFStringCreateWithFormat>
(gdb) # first parameter is "alloc"
(gdb) p/a *(int*)($esp+4)
$7 = 0xa0473ee0 <__kCFAllocatorSystemDefault>
(gdb) # second parameter is "formatOptions"
(gdb) p/a *(int*)($esp+8)
$8 = 0x0
(gdb) # third parameter is "format"
(gdb) call (void)CFShow(*(int*)($esp+12))
managed/%@/%@
(gdb) # return address is on the top of the stack
(gdb) p/a *(int*)$esp
$9 = 0x940f52cc <__CFXPreferencesGetManagedSourceForBundleIDAndUser+76>
(gdb) # Set a breakpoint on the return address.
(gdb) b *0x940f52cc
Breakpoint 3 at 0x940f52cc
(gdb) c
Continuing.

Breakpoint 3, 0x940f52cc in __CFXPreferencesGetManagedSourceForBundleIDAndUser ()
(gdb) # function result
(gdb) p/a $eax
$10 = 0x1079d0
(gdb) call (void)CFShow($eax)
managed/com.apple.TextEdit/kCFPreferencesCurrentUser</pre>

####陷阱
下面介绍下在汇编级调试可能遇到的一些问题。

#####其他参数
汇编级别查看参数时，记住以下几点:
* 如果函数是 C++ 成员函数，隐含的第一个参数是 this.
* 如果函数时 objective-C 方法，有两个隐含的参数(查看Objective-C的相关细节)
* 如果一个编译器可以找到一个函数的所有调用者(这通常发生在静态函数身上)，它可以选择将参数以非标准的形式传递。这是一个非常不常见的高效的基于寄存器的 ABI 架构，但是它在 32位程序中比较常见。因此，如果你在32位架构程序上的静态函数上设置断点，当心这令人费解的行为。

####字节序和类型大小
当你用 GDB 检查内存时，如果你用正确的类型大小会容易一些。列表4是 GDB 支持的类型大小总结。

列表4:GDB 类型大小

|大小     |C 类型  | GDB 类型 |记忆     |
|--------|:-------:|:-------:|  -----:|
|1 字节   |    char |  b      |  byte  |
|2 字节   |    short |  h      |   half word |
|4 字节   |    int   |  w      |  word  |
|8 字节   |    long or long long |  g      |  giant  |

当你在少数的系统上调试(所有的iOS设备和模拟器上)这个很重要，因为会出现很奇怪的结果如果你指定错了类型大小
表5展示了这种情况，(在Mac 机器上，和 iOS 设置结果一样)。CFStringCreateWIthCharacters 的第二个和第三个参数指定一个 Unicode 字符数组。每一个元素都是一个Ubichar,16比特的一个数字。当我们在低位运行的系统上时，你必须用正确的类型大小打印这个array,否则就会混乱。

表5: 使用争取的类型大小

<pre>$ gdb
GNU gdb 6.3.50-20050815 (Apple version gdb-1346) […]
(gdb) attach Finder
Attaching to process 4732.
Reading symbols for shared libraries . done
Reading symbols for shared libraries […]
0x00007fff81963e3a in mach_msg_trap ()
(gdb) b *CFStringCreateWithCharacters
Breakpoint 1 at 0x7fff86070520
(gdb) c
Continuing.

Breakpoint 1, 0x00007fff86070520 in CFStringCreateWithCharacters ()
(gdb) # The third parameter is the number of UniChars 
(gdb) # in the buffer pointed to by the first parameter.
(gdb) p (int)$rdx
$1 = 18
(gdb) # Dump the buffer as shorts. Everything makes sense.
(gdb) # This is the string "Auto-Save Recovery".
(gdb) x/18xh $rsi
0x10b7df292: 0x0041  0x0075  0x0074  0x006f  0x002d  0x0053  0x0061  0x0076
0x10b7df2a2: 0x0065  0x0020  0x0052  0x0065  0x0063  0x006f  0x0076  0x0065
0x10b7df2b2: 0x0072  0x0079
(gdb) # Now dump the buffer as words. Most confusing.
(gdb) # It looks like "uAotS-va eeRocevyr"!
(gdb) x/9xw $rsi
0x10b7df292: 0x00750041      0x006f0074      0x0053002d      0x00760061
0x10b7df2a2: 0x00200065      0x00650052      0x006f0063      0x00650076
0x10b7df2b2: 0x00790072
(gdb) # Now dump the buffer as bytes. This is a little less 
(gdb) # confusing, but you still have to remember that it's big 
(gdb) # endian data.
(gdb) x/36xb $rsi
0x10b7df292: 0x41    0x00    0x75    0x00    0x74    0x00    0x6f    0x00
0x10b7df29a: 0x2d    0x00    0x53    0x00    0x61    0x00    0x76    0x00
0x10b7df2a2: 0x65    0x00    0x20    0x00    0x52    0x00    0x65    0x00
0x10b7df2aa: 0x63    0x00    0x6f    0x00    0x76    0x00    0x65    0x00
0x10b7df2b2: 0x72    0x00    0x79    0x00

</pre>

####控制崩溃
在有些情况下使你的程序崩溃是有用的程序控制方式。一个常见的这么做的方法叫做 abort。另外一个方法是使用 __builtin_trap 函数，这生成了一个机器陷阱。表6展示了他是怎么做到的。

表6: __builtin_trap
`int main(int argc, char **argv) {     __builtin_trap();     return 1; }`

如果你在调试器上运行程序，你要停在调用 __builtin_trap 前，否则程序会崩溃并且生成崩溃报告。

>注意:我们建议你在debug 程序中少使用这个技术，在 release 程序中应该使用 abort.

应当注意 iOS 的程序生命周期由用户决定，这意味着 iOS 的应用程序不仅仅是退出。你发布的程序应该在崩溃的地方只使用 abort，它可以防止毁坏用户数据并且可以很好的定位问题。

####Instruments

Instruments 是一个动态跟踪和分析代码的程序。 他运行在 Mac OS X 上允许你的目标程序运行在 Mac OS X ，iOS设备和iphone 模拟器上。

Instruments 主要用在分析调试性能上，也可以用来调试错误。比如，ObjectAlloc 工具帮助你追踪bug。
Instrumemts 的一个好的特性是他可以容易的获得僵尸对象。细节请查看 [Instruments User Guide](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Introduction/Introduction.html).

####CrashReporter
CrashReproter 是一个很有用的调试工具会打印所有程序的崩溃日志。一直启用它，你要做的就是看他的输出。 CrashReporter 细节在[Technical Note TN2151, 'Understanding and Analyzing iPhone OS Application Crash Reports'](https://developer.apple.com/library/content/navigation/index.html#topic=Technical+Notes&section=Resource+Types).

####BSD
BSD 的子系统实现了进程，内存，文件和网路结构，因此对系统中的所有程序都很有用。BSD 实现了一些你可以利用的调试工具。

####内存分配器
默认的内存分配器包含大量的调试工具，你可以通过环境变量使用它。可以看manual page.列表5展示了有用的一个。

列表5:一些有用的内存分配器的环境变量

|变量|概要|
|---|:-----:|
| MallocScribble |用 oxAA 填充分配的内存，ox55填充释放的内存|
| MallocGuardEdges | 大内存分配之前和之后添加保护页|
| MallocStackLogging | 记录每个内存块的分配的回溯协助内存调试工具。如果一个内存块被分配又很快被释放，两条记录都会从日志中清除，有助于减少日志的大小 |
| MallocStackLoggingNoCompact |和 MallocStackLogging 一样但是保留所有日志记录|

默认的内存分配器会记录程序中常见的编程问题。例如。如果你释放了两次同一个内存块，或者释放你从来没有创建过的内存，表7将展示打印信息。圆括号里面的数字是进程ID。

表7: 释放内存时常见的信息
`DummyPhone(1839) malloc: *** error for object 0x826600: double free *** set a breakpoint in malloc_error_break to debug`

你可以在你的程序中使用 GDB 调试调试这一类问题，也可以打断点在 malloc_error_break.到了断点处，你可以使用 GDB 的回溯命令 breakrace 查看最终的调用者。

最后，你可以编程式的查看堆的一致性使用 malloc_zone_check函数，(malloc.h).


####标准 C++ 库
标准 C++ 库支持很多调试的特性：
* 在标准C++ 库中debug 模式下启动设置编译时变量 _GLIBCXX_DEBUG。这个特性在 GCC 4.2或者更高版本下不支持。
* 在 GCC 4.0之前的版本，设置 GLIBCPP_FORCE_NEW 环境变量为1来阻止内存缓存在标准 C++ 库，这使得你可以使用内存调试工具来调试 C++ 内存分配。 GCC 4.0或者之后的版本是默认设置为1的。

####动态链接器(dyld)
dyld 支持一些环境变量的调试工具。都被记录在 manual page。表6列出了一些有用的变量。

|变量|概要|
|-----|:-----:|
|DYLD_IMAGE_SUFFIX | 用这个后缀搜索库|
|DYLD_PRINT_LIBRARIES |下载日志文件|
| DYLD_PRINT_LIBRARIES_POST_LAUNCH | 如上所述，但主要运行main之后|
|DYLD_PRINT_OPTS [1] | 打印启动命令行参数|
|DYLD_PRINT_ENV [1] | 打印启动环境变量|
|DYLD_PRINT_APIS [1] | 打印dyld 调用|
|DYLD_PRINT_BINDINGS [1] | 打印绑定的符号|
|DYLD_PRINT_INITIALIZERS [1] | 打印图片初始化调用|
|DYLD_PRINT_SEGMENTS [1] | 打印段的映射|
|DYLD_PRINT_STATISTICS [1] | 打印启动性能数据|

>注意：这些不适用于 Mac OS X 10.4和之后的版本，适用于所有 iOS 的版本。虽然这些都在 iOS 系统上实现，但是他们中的很多作用有限，因为系统环境限制。

####核心支持

#####Core Foundation
Core Foundation framework CFShow 函数，打印所有 CF 对象到 stderr.你可以在自己的代码这么条用，但是，在 GDB 中调用很有用，表8举例，

表8: GDB 调用 CFShow
<pre>$ gdb /Applications/TextEdit.app
GNU gdb 6.3.50-20050815 (Apple version gdb-1346) […]
(gdb) fb CFRunLoopAddSource
Breakpoint 1 at 0x624dd2f195cfa8
(gdb) r
Starting program: /Applications/TextEdit.app/Contents/MacOS/TextEdit 
Reading symbols for shared libraries […]
Breakpoint 1, 0x00007fff8609bfa8 in CFRunLoopAddSource ()
(gdb) # Check that the prologue hasn't changed $rdi.
(gdb) p/a 0x00007fff8609bfa8
$1 = 0x7fff8609bfa8 <CFRunLoopAddSource+24>
(gdb) p/a $pc
$2 = 0x7fff8609bfa8 <CFRunLoopAddSource+24>
(gdb) x/8i $pc-24
0x7fff8609bf90 <CFRunLoopAddSource>: push   %rbp
0x7fff8609bf91 <CFRunLoopAddSource+1>: mov    %rsp,%rbp
0x7fff8609bf94 <CFRunLoopAddSource+4>: mov    %rbx,-0x20(%rbp)
0x7fff8609bf98 <CFRunLoopAddSource+8>: mov    %r12,-0x18(%rbp)
0x7fff8609bf9c <CFRunLoopAddSource+12>: mov    %r13,-0x10(%rbp)
0x7fff8609bfa0 <CFRunLoopAddSource+16>: mov    %r14,-0x8(%rbp)
0x7fff8609bfa4 <CFRunLoopAddSource+20>: sub    $0x40,%rsp
0x7fff8609bfa8 <CFRunLoopAddSource+24>: mov    %rdi,%r12
(gdb) # Nope. Go ahead and CFShow it.
(gdb) call (void)CFShow($rdi)
<CFRunLoop 0x100115540 [0x7fff70b8bf20]>{
    locked = false, 
    wakeup port = 0x1e07, 
    stopped = false,
    current mode = (none),
    common modes = <CFBasicHash 0x1001155a0 [0x7fff70b8bf20]>{
        type = mutable set, 
        count = 1,
        entries =>
            2 : <CFString 0x7fff70b693d0 [0x7fff70b8bf20]>{
                contents = "kCFRunLoopDefaultMode"
            }
    },
    common mode items = (null),
    modes = <CFBasicHash 0x1001155d0 [0x7fff70b8bf20]>{
        type = mutable set, 
        count = 1,
        entries =>
            0 : <CFRunLoopMode 0x100115670 [0x7fff70b8bf20]>{
                name = kCFRunLoopDefaultMode, 
                locked = false, 
                port set = 0x1f03,
                sources = (null),
                observers = (null),
                timers = (null)
        },
    }
}</pre>

>重要：如果你没有看到 CFShow 的输出，或许在控制台中有。你可以看下控制台的输出信息。
>信息:表8中的输出已经被重新格式化，以便于阅读。

#####僵尸对象

>重要：如果你用 Objective-C 编程。你应该会对 NSZombieEnabled 这个比较感兴趣，在 More Zombies中有描述。

Core Foundation 支持一个叫做 CFZombieLevel 的环境变量。此变量包含一组标志位。列表7展示了当前定义的位。这个可以帮助你追踪 CF的内存管理问题。

列表7: CFZombieLevel 环境变量的标志位定义

|位|意义|
|----|:----:|
|  0 |  释放 CF 内存|
|  1 |  释放 CF 内存时，不释放 CFRuntimeBase|
|  4 |  不释放 保存 CF 对象的内存|
|  7 |  如果设置了，释放的内存使用8..15，否则使用 0xFC|
|  8..15 |  如果设置了7，释放的内存使用这个值|
|  16 |  分配 CF memory|
|  23 |  如果设置了，分配的使用 24..31，否则使用0XCF|
|  23..31 | 如果设置了23，分配的使用这个值|

####应用服务

#####Core Animation
Core Animation 工具可以帮助你测量程序的计算速率和各种图。细节看[Instrumenrs User Guide](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Introduction/Introduction.html).

####Cocoa 和 Cocoa Touch






















 





































