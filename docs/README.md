> 世界上并没有完美的程序，但我们并不因此而沮丧，因为写程序本来就是一个不断追求完美的过程。

# 1.2　Java技术体系

&emsp;&emsp;从广义上讲，Clojure、JRuby、Groovy等运行于Java虚拟机上的语言及其相关的程序都属于Java技术体系中的一员。如果仅从传统意义上来看，Sun官方所定义的Java技术体系包括以下几个组成部分：
- Java程序设计语言
- 各种硬件平台上的Java虚拟机
- Class文件格式
- Java API类库
- 来自商业机构和开源社区的第三方Java类库

&emsp;&emsp;我们可以把Java程序设计语言、Java虚拟机、Java API类库这三部分统称为JDK（Java Development Kit），JDK是用于支持Java程序开发的最小环境，在后面的内容中，为了讲解方便，有一些地方会以JDK来代替整个Java技术体系。另外，可以把Java API类库中的Java SE API子集[1]和Java虚拟机这两部分统称为JRE（Java Runtime Environment），JRE是支持Java程序运行的标准环境。图1-2展示了Java技术体系所包含的内容，以及JDK和JRE所涵盖的范围。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30143839_Um3j.png "在这里输入图片标题")

&emsp;&emsp;以上是根据各个组成部分的功能来进行划分的，如果按照技术所服务的领域来划分，或者说按照Java技术关注的重点业务领域来划分，Java技术体系可以分为4个平台，分别为：
- Java Card：支持一些Java小程序（Applets）运行在小内存设备（如智能卡）上的平台。
- Java ME（Micro Edition）：支持Java程序运行在移动终端（手机、PDA）上的平台，对Java API有所精简，并加入了针对移动终端的支持，这个版本以前称为J2ME。
- Java SE（Standard Edition）：支持面向桌面级应用（如Windows下的应用程序）的Java平台，提供了完整的Java核心API，这个版本以前称为J2SE。
- Java EE（Enterprise Edition）：支持使用多层架构的企业应用（如ERP、CRM应用）的Java平台，除了提供Java SE API外，还对其做了大量的扩充[3]并提供了相关的部署支持，这个版本以前称为J2EE。

# 1.4　Java虚拟机发展史

&emsp;&emsp;从1996年初Sun公司发布的JDK 1.0中所包含的Sun Classic VM到今天，曾经涌现、湮灭过许多或经典或优秀或有特色的虚拟机实现，在这一节中，我们先暂且把代码与技术放下，一起来回顾一下Java虚拟机家族的发展轨迹和历史变迁。

## 1.4.1　Sun Classic/Exact VM

&emsp;&emsp;以今天的视角来看，Sun Classic VM的技术可能很原始，这款虚拟机的使命也早已终结。但仅凭它“世界上第一款商用Java虚拟机”的头衔，就足够有让历史记住它的理由。

&emsp;&emsp;1996年1月23日，Sun公司发布JDK 1.0，Java语言首次拥有了商用的正式运行环境，这个JDK中所带的虚拟机就是Classic VM。这款虚拟机只能使用纯解释器方式来执行Java代码，如果要使用JIT编译器，就必须进行外挂。但是假如外挂了JIT编译器，JIT编译器就完全接管了虚拟机的执行系统，解释器便不再工作了。用户在这款虚拟机上执行java-version命令，将会看到类似下面这行输出：
```java
java version"1.2.2"
Classic VM（build JDK-1.2.2-001，green threads,sunwjit）
```
&emsp;&emsp;其中的"sunwjit"就是Sun提供的外挂编译器，其他类似的外挂编译器还有Symantec JIT和shuJIT等。由于解释器和编译器不能配合工作，这就意味着如果要使用编译器执行，编译器就不得不对每一个方法、每一行代码都进行编译，而无论它们执行的频率是否具有编译的价值。基于程序响应时间的压力，这些编译器根本不敢应用编译耗时稍高的优化技术，因此这个阶段的虚拟机即使用了JIT编译器输出本地代码，执行效率也和传统的C/C++程序有很大差距，“Java语言很慢”的形象就是在这时候开始在用户心中树立起来的。

&emsp;&emsp;Sun的虚拟机团队努力去解决Classic VM所面临的各种问题，提升运行效率。在JDK 1.2时，曾在Solaris平台上发布过一款名为Exact VM的虚拟机，它的执行系统已经具备现代高性能虚拟机的雏形：如两级即时编译器、编译器与解释器混合工作模式等。Exact VM因它使用准确式内存管理（Exact Memory Management，也可以叫Non-Conservative/Accurate Memory Management）而得名，即虚拟机可以知道内存中某个位置的数据具体是什么类型。譬如内存中有一个32位的整数123456，它到底是一个reference类型指向123456的内存地址还是一个数值为123456的整数，虚拟机将有能力分辨出来，这样才能在GC（垃圾收集）的时候准确判断堆上的数据是否还可能被使用。由于使用了准确式内存管理，Exact VM可以抛弃以前Classic VM基于handler的对象查找方式（原因是进行GC后对象将可能会被移动位置，如果将地址为123456的对象移动到654321，在没有明确信息表明内存中哪些数据是reference的前提下，虚拟机是不敢把内存中所有为123456的值改成654321的，所以要使用句柄来保持reference值的稳定），这样每次定位对象都少了一次间接查找的开销，提升执行性能。

&emsp;&emsp;虽然Exact VM的技术相对Classic VM来说先进了许多，但是在商业应用上只存在了很短暂的时间就被更为优秀的HotSpot VM所取代，甚至还没有来得及发布Windows和Linux平台下的商用版本。而Classic VM的生命周期则相对长了许多，它在JDK 1.2之前是Sun JDK中唯一的虚拟机，在JDK 1.2时，它与HotSpot VM并存，但默认使用的是Classic VM（用户可用java-hotspot参数切换至HotSpot VM），而在JDK 1.3时，HotSpot VM成为默认虚拟机，但Classic VM仍作为虚拟机的“备用选择”发布（使用java-classic参数切换），直到JDK 1.4的时候，Classic VM才完全退出商用虚拟机的历史舞台，与Exact VM一起进入了Sun Labs Research VM之中。

## 1.4.2　Sun HotSpot VM

&emsp;&emsp;提起HotSpot VM，相信所有Java程序员都知道，它是Sun JDK和OpenJDK中所带的虚拟机，也是目前使用范围最广的Java虚拟机。但不一定所有人都知道的是，这个目前看起来“血统纯正”的虚拟机在最初并非由Sun公司开发，而是由一家名为"Longview Technologies"的小公司设计的；甚至这个虚拟机最初并非是为Java语言而开发的，它来源于Strongtalk VM，而这款虚拟机中相当多的技术又是来源于一款支持Self语言实现“达到C语言50%以上的执行效率”的目标而设计的虚拟机，Sun公司注意到了这款虚拟机在JIT编译上有许多优秀的理念和实际效果，在1997年收购了Longview Technologies公司，从而获得了HotSpot VM。

&emsp;&emsp;HotSpot VM既继承了Sun之前两款商用虚拟机的优点（如前面提到的准确式内存管理），也有许多自己新的技术优势，如它名称中的HotSpot指的就是它的热点代码探测技术（其实两个VM基本上是同时期的独立产品，HotSpot还稍早一些，HotSpot一开始就是准确式GC，而Exact VM之中也有与HotSpot几乎一样的热点探测。为了Exact VM和HotSpot VM哪个成为Sun主要支持的VM产品，在Sun公司内部还有过争论，HotSpot打败Exact并不能算技术上的胜利），HotSpot VM的热点代码探测能力可以通过执行计数器找出最具有编译价值的代码，然后通知JIT编译器以方法为单位进行编译。如果一个方法被频繁调用，或方法中有效循环次数很多，将会分别触发标准编译和OSR（栈上替换）编译动作。通过编译器与解释器恰当地协同工作，可以在最优化的程序响应时间与最佳执行性能中取得平衡，而且无须等待本地代码输出才能执行程序，即时编译的时间压力也相对减小，这样有助于引入更多的代码优化技术，输出质量更高的本地代码。

&emsp;&emsp;在2006年的JavaOne大会上，Sun公司宣布最终会把Java开源，并在随后的一年，陆续将JDK的各个部分（其中当然也包括了HotSpot VM）在GPL协议下公开了源码，并在此基础上建立了OpenJDK。这样，HotSpot VM便成为了Sun JDK和OpenJDK两个实现极度接近的JDK项目的共同虚拟机。

&emsp;&emsp;在2008年和2009年，Oracle公司分别收购了BEA公司和Sun公司，这样Oracle就同时拥有了两款优秀的Java虚拟机：JRockit VM和HotSpot VM。Oracle公司宣布在不久的将来（大约应在发布JDK 8的时候）会完成这两款虚拟机的整合工作，使之优势互补。整合的方式大致上是在HotSpot的基础上，移植JRockit的优秀特性，譬如使用JRockit的垃圾回收器与MissionControl服务，使用HotSpot的JIT编译器与混合的运行时系统。

## 1.4.3　Sun Mobile-Embedded VM/Meta-Circular VM

&emsp;&emsp;Sun公司所研发的虚拟机可不仅有前面介绍的服务器、桌面领域的商用虚拟机，除此之外，Sun公司面对移动和嵌入式市场，也发布过虚拟机产品，另外还有一类虚拟机，在设计之初就没抱有商用的目的，仅仅是用于研究、验证某种技术和观点，又或者是作为一些规范的标准实现。这些虚拟机对于大部分不从事相关领域开发的Java程序员来说可能比较陌生。Sun公司发布的其他Java虚拟机有：

&emsp;&emsp;（1）KVM

&emsp;&emsp;KVM中的K是"Kilobyte"的意思，它强调简单、轻量、高度可移植，但是运行速度比较慢。在Android、iOS等智能手机操作系统出现前曾经在手机平台上得到非常广泛的应用。

&emsp;&emsp;（2）CDC/CLDC HotSpot Implementation

&emsp;&emsp;CDC/CLDC全称是Connected（Limited）Device Configuration，在JSR-139/JSR-218规范中进行定义，它希望在手机、电子书、PDA等设备上建立统一的Java编程接口，而CDC-HI VM和CLDC-HI VM则是它们的一组参考实现。CDC/CLDC是整个Java ME的重要支柱，但从目前Android和iOS二分天下的移动数字设备市场看来，在这个领域中，Sun的虚拟机所面临的局面远不如服务器和桌面领域乐观。

&emsp;&emsp;（3）Squawk VM

&emsp;&emsp;Squawk VM由Sun公司开发，运行于Sun SPOT（Sun Small Programmable Object Technology，一种手持的WiFi设备），也曾经运用于Java Card。这是一个Java代码比重很高的嵌入式虚拟机实现，其中诸如类加载器、字节码验证器、垃圾收集器、解释器、编译器和线程调度都是Java语言本身完成的，仅仅靠C语言来编写设备I/O和必要的本地代码。

&emsp;&emsp;（4）JavaInJava

&emsp;&emsp;JavaInJava是Sun公司于1997年～1998年间研发的一个实验室性质的虚拟机，从名字就可以看出，它试图以Java语言来实现Java语言本身的运行环境，既所谓的“元循环”（Meta-Circular，是指使用语言自身来实现其运行环境）。它必须运行在另外一个宿主虚拟机之上，内部没有JIT编译器，代码只能以解释模式执行。在20世纪末主流Java虚拟机都未能很好解决性能问题的时代，开发这种项目，其执行速度可想而知。

&emsp;&emsp;（5）Maxine VM

&emsp;&emsp;Maxine VM和上面的JavaInJava非常相似，它也是一个几乎全部以Java代码实现（只有用于启动JVM的加载器使用C语言编写）的元循环Java虚拟机。这个项目于2005年开始，到现在仍然在发展之中，比起JavaInJava,Maxine VM就显得“靠谱”很多，它有先进的JIT编译器和垃圾收集器（但没有解释器），可在宿主模式或独立模式下执行，其执行效率已经接近了HotSpot Client VM的水平。

## 1.4.4　BEA JRockit/IBM J9 VM

&emsp;&emsp;前面介绍了Sun公司的各种虚拟机，除了Sun公司以外，其他组织、公司也研发过不少虚拟机实现，其中规模最大、最著名的就是BEA和IBM公司了。

&emsp;&emsp;JRockit VM曾经号称“世界上速度最快的Java虚拟机”（广告词，貌似J9 VM也这样说过），它是BEA公司在2002年从Appeal Virtual Machines公司收购的虚拟机。BEA公司将其发展为一款专门为服务器硬件和服务器端应用场景高度优化的虚拟机，由于专注于服务器端应用，它可以不太关注程序启动速度，因此JRockit内部不包含解析器实现，全部代码都靠即时编译器编译后执行。除此之外，JRockit的垃圾收集器和MissionControl服务套件等部分的实现，在众多Java虚拟机中也一直处于领先水平。

&emsp;&emsp;IBM J9 VM并不是IBM公司唯一的Java虚拟机，不过是目前其主力发展的Java虚拟机。IBM J9 VM原本是内部开发代号，正式名称是"IBM Technology for Java Virtual Machine"，简称IT4J，只是这个名字太拗口了一点，普及程度不如J9。J9 VM最初是由IBM Ottawa实验室一个名为SmallTalk的虚拟机扩展而来的，当时这个虚拟机有一个bug是由8k值定义错误引起的，工程师花了很长时间终于发现并解决了这个错误，此后这个版本的虚拟机就称为K8了，后来扩展出支持Java的虚拟机就被称为J9了。与BEA JRockit专注于服务器端应用不同，IBM J9的市场定位与Sun HotSpot比较接近，它是一款设计上从服务器端到桌面应用再到嵌入式都全面考虑的多用途虚拟机，J9的开发目的是作为IBM公司各种Java产品的执行平台，它的主要市场是和IBM产品（如IBM WebSphere等）搭配以及在IBM AIX和z/OS这些平台上部署Java应用。

## 1.4.5　Azul VM/BEA Liquid VM

&emsp;&emsp;我们平时所提及的“高性能Java虚拟机”一般是指HotSpot、JRockit、J9这类在通用平台上运行的商用虚拟机，但其实Azul VM和BEA Liquid VM这类特定硬件平台专有的虚拟机才是“高性能”的武器。

&emsp;&emsp;Azul VM是Azul Systems公司在HotSpot基础上进行大量改进，运行于Azul Systems公司的专有硬件Vega系统上的Java虚拟机，每个Azul VM实例都可以管理至少数十个CPU和数百GB内存的硬件资源，并提供在巨大内存范围内实现可控的GC时间的垃圾收集器、为专有硬件优化的线程调度等优秀特性。在2010年，Azul Systems公司开始从硬件转向软件，发布了自己的Zing JVM，可以在通用x86平台上提供接近于Vega系统的特性。

&emsp;&emsp;Liquid VM即是现在的JRockit VE（Virtual Edition），它是BEA公司开发的，可以直接运行在自家Hypervisor系统上的JRockit VM的虚拟化版本，Liquid VM不需要操作系统的支持，或者说它自己本身实现了一个专用操作系统的必要功能，如文件系统、网络支持等。由虚拟机越过通用操作系统直接控制硬件可以获得很多好处，如在线程调度时，不需要再进行内核态/用户态的切换等，这样可以最大限度地发挥硬件的能力，提升Java程序的执行性能。

## 1.4.6　Apache Harmony/Google Android Dalvik VM

&emsp;&emsp;这节介绍的Harmony VM和Dalvik VM只能称做“虚拟机”，而不能称做“Java虚拟机”，但是这两款虚拟机（以及所代表的技术体系）对最近几年的Java世界产生了非常大的影响和挑战，甚至有些悲观的评论家认为成熟的Java生态系统有崩溃的可能。

&emsp;&emsp;Apache Harmony是一个Apache软件基金会旗下以Apache License协议开源的实际兼容于JDK 1.5和JDK 1.6的Java程序运行平台，这个介绍相当拗口。它包含自己的虚拟机和Java库，用户可以在上面运行Eclipse、Tomcat、Maven等常见的Java程序，但是它没有通过TCK认证，所以我们不得不用那么一长串拗口的语言来介绍它，而不能用一句“Apache的JDK”来说明。如果一个公司要宣布自己的运行平台“兼容于Java语言”，那就必须要通过TCK（Technology Compatibility Kit）的兼容性测试。Apache基金会曾要求Sun公司提供TCK的使用授权，但是一直遭到拒绝，直到Oracle公司收购了Sun公司之后，双方关系越闹越僵，最终导致Apache愤然退出JCP（Java Community Process）组织，这是目前为止Java社区最严重的一次“分裂”。

&emsp;&emsp;在Sun将JDK开源形成OpenJDK之后，Apache Harmony开源的优势被极大地削弱，甚至连Harmony项目的最大参与者IBM公司也宣布辞去Harmony项目管理主席的职位，并参与OpenJDK项目的开发。虽然Harmony没有经过真正大规模的商业运用，但是它的许多代码（基本上是Java库部分的代码）被吸纳进IBM的JDK 7实现及Google Android SDK之中，尤其是对Android的发展起到了很大的推动作用。

&emsp;&emsp;说到Android，这个时下最热门的移动数码设备平台在最近几年间的发展过程中所取得的成果已经远远超越了Java ME在过去十多年所获得的成果，Android让Java语言真正走进了移动数码设备领域，只是走的并非Sun公司原本想象的那一条路。

&emsp;&emsp;Dalvik VM是Android平台的核心组成部分之一，它的名字来源于冰岛一个名为Dalvik的小渔村。Dalvik VM并不是一个Java虚拟机，它没有遵循Java虚拟机规范，不能直接执行Java的Class文件，使用的是寄存器架构而不是JVM中常见的栈架构。但是它与Java又有着千丝万缕的联系，它执行的dex（Dalvik Executable）文件可以通过Class文件转化而来，使用Java语法编写应用程序，可以直接使用大部分的Java API等。目前Dalvik VM随着Android一起处于迅猛发展阶段，在Android 2.2中已提供即时编译器实现，在执行性能上有了很大的提高。

## 1.4.7　Microsoft JVM及其他

&emsp;&emsp;在十几年的Java虚拟机发展过程中，除去上面介绍的那些被大规模商业应用过的Java虚拟机外，还有许多虚拟机是不为人知的或者曾经“绚丽”过但最终湮灭的。我们以其中微软公司的JVM为例来介绍一下。

&emsp;&emsp;也许Java程序员听起来可能会觉得惊讶，微软公司曾经是Java技术的铁杆支持者（也必须承认，与Sun公司争夺Java的控制权，令Java从跨平台技术变为绑定在Windows上的技术是微软公司的主要目的）。在Java语言诞生的初期（1996年～1998年，以JDK 1.2发布为分界），它的主要应用之一是在浏览器中运行Java Applets程序，微软公司为了在IE3中支持Java Applets应用而开发了自己的Java虚拟机，虽然这款虚拟机只有Windows平台的版本，却是当时Windows下性能最好的Java虚拟机，它在1997年和1998年连续两年获得了《PC Magazine》杂志的“编辑选择奖”。但好景不长，在1997年10月，Sun公司正式以侵犯商标、不正当竞争等罪名控告微软公司，在随后对微软公司的垄断调查之中，这款虚拟机也曾作为证据之一被呈送法庭。这场官司的结果是微软公司赔偿2000万美金给Sun公司（最终微软公司因垄断赔偿给Sun公司的总金额高达10亿美元），承诺终止其Java虚拟机的发展，并逐步在产品中移除Java虚拟机相关功能。具有讽刺意味的是，到最后在Windows XP SP3中Java虚拟机被完全抹去的时候，Sun公司却又到处登报希望微软公司不要这样做[1]。Windows XP高级产品经理Jim Cullinan称：“我们花费了3年的时间和Sun打官司，当时他们试图阻止我们在Windows中支持Java，现在我们这样做了，可他们又在抱怨，这太具有讽刺意味了。”

&emsp;&emsp;我们试想一下，如果当年Sun公司没有起诉微软公司，微软公司继续保持着对Java技术的热情，那Java的世界会变得怎么样呢？.NET技术是否会发展起来？但历史是没有假设的。其他在本节中没有介绍到的Java虚拟机还有（当然，应该还有很多笔者所不知道的）：
- JamVM.
- cacaovm.
- SableVM.
- Kaffe.
- Jelatine JVM.
- NanoVM.
- MRP.
- Moxie JVM.
- Jikes RVM.

# 1.5　展望Java技术的未来

&emsp;&emsp;在2005年，Java语言诞生10周年的SunOne技术大会上，Java语言之父James Gosling做了一场题为“Java技术下一个十年”的演讲。笔者不具备James Gosling博士那样高屋建瓴的视角，这里仅从Java平台中几个新生的但已经开始展现出蓬勃之势的技术发展点来看一下后续1～2个JDK版本内的一些很有希望的技术重点。

## 1.5.1　模块化

&emsp;&emsp;模块化是解决应用系统与技术平台越来越复杂、越来越庞大问题的一个重要途径。无论是开发人员还是产品最终用户，都不希望为了系统中一小块的功能而不得不下载、安装、部署及维护整套庞大的系统。站在整个软件工业化的高度来看，模块化是建立各种功能的标准件的前提。最近几年OSGi技术的迅速发展、各个厂商在JCP中对模块化规范的激烈斗争[1]，都能充分说明模块化技术的迫切和重要。

&emsp;&emsp;在未来的Java平台中，很可能会对模块化提出语法层面的支持。早在2007年，Sun公司就提出过JSR-277：Java模块系统（Java Module System），试图建立Java平台的模块化标准，但受挫于以IBM公司为主导提交的JSR-291：Java SE动态组件支持（Dynamic Component Support for Java SE，这实际就是OSGi R4.1）。由于模块化规范主导权的重要性，Sun公司不能接受一个无法由它控制的规范，在整个Java SE 6期间都拒绝把任何模块化技术内置到JDK之中。在Java SE 7发展初期，Sun公司再次提交了一个新的规范请求文档JSR-294：Java编程语言中的改进模块性支持（Improved Modularity Support in the Java Programming Language），尽管这个JSR仍然没有通过，但是Sun公司已经独立于JCP专家组在OpenJDK里建立了一个名为Jigsaw（拼图）的子项目来推动这个规范在Java平台中转变为具体的实现。Java的模块化之争目前还没有结束，OSGi已经发布到R5.0版本，而Jigsaw从Java 7延迟至Java 8，在2012年7月又不得不宣布推迟到Java 9中发布，从这点看来，Sun在这场战争中处于劣势，但无论胜利者是哪一方，Java模块化已经成为一项无法阻挡的变革潮流。

##1.5.2　混合语言

&emsp;&emsp;当单一的Java开发已经无法满足当前软件的复杂需求时，越来越多基于Java虚拟机的语言开发被应用到软件项目中，Java平台上的多语言混合编程正成为主流，每种语言都可以针对自己擅长的方面更好地解决问题。试想一下，在一个项目之中，并行处理用Clojure语言编写，展示层使用JRuby/Rails，中间层则是Java，每个应用层都将使用不同的编程语言来完成，而且，接口对每一层的开发者都是透明的，各种语言之间的交互不存在任何困难，就像使用自己语言的原生API一样方便[1]，因为它们最终都运行在一个虚拟机之上。

&emsp;&emsp;在最近的几年里，Clojure、JRuby、Groovy等新生语言的使用人数不断增长，而运行在Java虚拟机（JVM）之上的语言数量也在迅速膨胀，图1-4中列举了其中的一部分。这两点证明混合编程在我们身边已经有所应用并被广泛认可。通过特定领域的语言去解决特定领域的问题是当前软件开发应对日趋复杂的项目需求的一个方向。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30160032_WkwA.png "在这里输入图片标题")

&emsp;&emsp;除了催生出大量的新语言外，许多已经有很长历史的程序语言也出现了基于Java虚拟机实现的版本，这样使得混合编程对许多以前使用其他语言的“老”程序员也具备相当大的吸引力，软件企业投入了大量资本的现有代码资产也能很好地保护起来。表1-1中列举了常见语言的JVM实现版本。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30160155_vK55.png "在这里输入图片标题")

&emsp;&emsp;对这些运行于Java虚拟机之上、Java之外的语言，来自系统级的、底层的支持正在迅速增强，以JSR-292为核心的一系列项目和功能改进（如Da Vinci Machine项目、Nashorn引擎、InvokeDynamic指令、java.lang.invoke包等），推动Java虚拟机从“Java语言的虚拟机”向“多语言虚拟机”的方向发展。

## 1.5.3　多核并行

&emsp;&emsp;如今，CPU硬件的发展方向已经从高频率转变为多核心，随着多核时代的来临，软件开发越来越关注并行编程的领域。早在JDK 1.5就已经引入java.util.concurrent包实现了一个粗粒度的并发框架。而JDK 1.7中加入的java.util.concurrent.forkjoin包则是对这个框架的一次重要扩充。Fork/Join模式是处理并行编程的一个经典方法，如图1-5所示。虽然不能解决所有的问题，但是在此模式的适用范围之内，能够轻松地利用多个CPU核心提供的计算资源来协作完成一个复杂的计算任务。通过利用Fork/Join模式，我们能够更加顺畅地过渡到多核时代。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30160955_4kca.png "在这里输入图片标题")

&emsp;&emsp;在Java 8中，将会提供Lambda支持，这将会极大改善目前Java语言不适合函数式编程的现状（目前Java语言使用函数式编程并不是不可以，只是会显得很臃肿），函数式编程的一个重要优点就是这样的程序天然地适合并行运行，这对Java语言在多核时代继续保持主流语言的地位有很大帮助。

&emsp;&emsp;另外，在并行计算中必须提及的还有OpenJDK的子项目Sumatra[2]，目前显卡的算术运算能力、并行能力已经远远超过了CPU，在图形领域以外发掘显卡的潜力是近几年计算机发展的方向之一，例如C语言的CUDA。Sumatra项目就是为Java提供使用GPU（Graphics Processing Units）和APU（Accelerated Processing Units）运算能力的工具，以后它将会直接提供Java语言层面的API，或者为Lambda和其他JVM语言提供底层的并行运算支持。

&emsp;&emsp;在JDK外围，也出现了专为满足并行计算需求的计算框架，如Apache的Hadoop Map/Reduce，这是一个简单易懂的并行框架，能够运行在由上千个商用机器组成的大型集群上，并且能以一种可靠的容错方式并行处理TB级别的数据集。另外，还出现了诸如Scala、Clojure及Erlang等天生就具备并行计算能力的语言。

## 1.5.4　进一步丰富语法

&emsp;&emsp;Java 5曾经对Java语法进行了一次扩充，这次扩充加入了自动装箱、泛型、动态注解、枚举、可变长参数、遍历循环等语法，使得Java语言的精确性和易用性有了很大的进步。在Java 7（由于进度压力，许多改进已推迟至Java 8）中，对Java语法进行了另一次大规模的扩充。Sun（已被Oracle收购）专门为改进Java语法在OpenJDK中建立了Coin子项目[1]来统一处理对Java语法的细节修改，如二进制数的原生支持、在switch语句中支持字符串、“＜＞”操作符、异常处理的改进、简化变长参数方法调用、面向资源的try-catch-finally语句等都是在Coin项目之中提交的内容。

&emsp;&emsp;除了Coin项目之外，在JSR-335（Lambda Expressions for the Java TM Programming Language）中定义的Lambda表达式[2]也将对Java的语法和语言习惯产生很大的影响，面向函数方式的编程可能会成为主流。

## 1.5.5　64位虚拟机

&emsp;&emsp;在几年之前，主流的CPU就开始支持64位架构了。Java虚拟机也在很早之前就推出了支持64位系统的版本。但Java程序运行在64位虚拟机上需要付出比较大的额外代价：首先是内存问题，由于指针膨胀和各种数据类型对齐补白的原因，运行于64位系统上的Java应用需要消耗更多的内存，通常要比32位系统额外增加10%～30%的内存消耗；其次，多个机构的测试结果显示，64位虚拟机的运行速度在各个测试项中几乎全面落后于32位虚拟机，两者大约有15%左右的性能差距。

&emsp;&emsp;但是在Java EE方面，企业级应用经常需要使用超过4GB的内存，对于64位虚拟机的需求是非常迫切的，但由于上述原因，许多企业应用都仍然选择使用虚拟集群等方式继续在32位虚拟机中进行部署。Sun也注意到了这些问题，并做出了一些改善，在JDK 1.6 Update 14之后，提供了普通对象指针压缩功能（-XX:+UseCompressedOops，这个参数不建议显式设置，建议维持默认由虚拟机的Ergonomics机制自动开启），在执行代码时，动态植入压缩指令以节省内存消耗，但是开启压缩指针会增加执行代码数量，因为所有在Java堆里的、指向Java堆内对象的指针都会被压缩，这些指针的访问就需要更多的代码才可以实现，而且并不只是读写字段才受影响，在实例方法调用、子类型检查等操作中也受影响，因为对象实例指向对象类型的引用也被压缩了。随着硬件的进一步发展，计算机终究会完全过渡到64位的时代，这是一件毫无疑问的事情，主流的虚拟机应用也终究会从32位发展至64位，而虚拟机对64位的支持也将会进一步完善。

# 1.6　实战：自己编译JDK

&emsp;&emsp;想要一探JDK内部的实现机制，最便捷的路径之一就是自己编译一套JDK，通过阅读和跟踪调试JDK源码去了解Java技术体系的原理，虽然门槛会高一点，但肯定会比阅读各种书籍、文章更加贴近本质。另外，JDK中的很多底层方法都是本地化（Native）的，需要跟踪这些方法的运作或对JDK进行Hack的时候，都需要自己编译一套JDK。

&emsp;&emsp;现在网络上有不少开源的JDK实现可以供我们选择，如Apache Harmony、OpenJDK等。考虑到Sun系列的JDK是现在使用得最广泛的JDK版本，笔者选择了OpenJDK进行这次编译实战。

## 1.6.1　获取JDK源码

&emsp;&emsp;首先要先明确OpenJDK和Sun/OracleJDK之间，以及OpenJDK 6、OpenJDK 7、OpenJDK 7u和OpenJDK 8等项目之间是什么关系，这有助于确定接下来编译要使用的JDK版本和源码分支。

&emsp;&emsp;从前面介绍的Java发展史中我们了解到OpenJDK是Sun在2006年末把Java开源而形成的项目，这里的“开源”是通常意义上的源码开放形式，即源码是可被复用的，例如IcedTea[1]、UltraViolet[2]都是从OpenJDK源码衍生出的发行版。但如果仅从“开源”字面意义（开放可阅读的源码）上看，其实Sun自JDK 1.5之后就开始以Java Research License（JRL）的形式公布过Java源码，主要用于研究人员阅读（JRL许可证的开放源码至JDK 1.6 Update 23为止）。把这些JRL许可证形式的Sun/OracleJDK源码和对应版本的OpenJDK源码进行比较，发现除了文件头的版权注释之外，其余代码基本上都是相同的，只有字体渲染部分存在一点差异，Oracle JDK采用了商业实现，而OpenJDK使用的是开源的FreeType。当然，“相同”是建立在两者共有的组件基础上的，Oracle JDK中还会存在一些Open JDK没有的、商用闭源的功能，例如从JRockit移植改造而来的Java Flight Recorder。预计以后JRockit的MissionControl移植到HotSpot之后，也会以Oracle JDK专有、闭源的形式提供。

&emsp;&emsp;Oracle的项目发布经理Joe Darcy在OSCON 2011上对两者关系的介绍[3]也证实了OpenJDK 7和Oracle JDK 7在程序上是非常接近的，两者共用了大量相同的代码（如图1-6所示，注意图中提示了两者共同代码的占比要远高于图形上看到的比例），所以我们编译的OpenJDK，基本上可以认为性能、功能和执行逻辑上都和官方的Oracle JDK是一致的。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30162323_UfMv.png "在这里输入图片标题")

&emsp;&emsp;再来看一下OpenJDK 6、OpenJDK 7、OpenJDK 7u和OpenJDK 8这几个项目之间的关系，从图1-7（依然是从Joe Darcy的OSCON 2011演示稿中截取的图片）来看，OpenJDK 7是始于JDK 6时期，当时JDK 6和JDK 6 Update 1已经发布，JDK 7已经开始研发了，所以OpenJDK 7是直接基于正在研发的JDK 7源码建立的。但考虑到OpenJDK 7的状况在当时还不适合实际生产部署，因此在OpenJDK 7 Build 20的基础上建立了OpenJDK 6分支，剥离掉JDK 7新功能的代码，形成一个可以通过TCK 6测试的独立分支。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30162432_HsEg.png "在这里输入图片标题")

&emsp;&emsp;2012年7月，JDK 7正式发布，在OpenJDK中也同步建立了OpenJDK 7 Update项目对JDK 7进行更新升级，以及OpenJDK 8项目开始下一个JDK大版本的研发。按照开发习惯，新的功能或Bug修复通常是在最新分支上进行的，当功能或修复在最新分支上稳定之后会同步到其他老版本的维护分支上。

&emsp;&emsp;OpenJDK 6、OpenJDK 7、OpenJDK 7u和OpenJDK 8的源码都可以在它们相应的网页上找到，在本次编译实践中，笔者选用的项目是OpenJDK 7u，版本为7u6。

&emsp;&emsp;获取OpenJDK源码有两种方式，其中一种是通过Mercurial代码版本管理工具从Repository中直接取得源码（Repository地址： http://hg.openjdk.java.net/jdk7u/jdk7u） ，获取过程如以下代码所示。

&emsp;&emsp;这是最直接的方式，从版本管理中看变更轨迹比看Release Note效果更好。但不足之处是速度太慢，虽然代码总容量只有300 MB左右，但是文件数量太多，在笔者的网络下全部复制到本地需要数小时。另外，考虑到Mercurial不如Git、SVN、ClearCase或CVS之类的版本控制工具那样普及，对于一般读者，建议采用第二种方式，即直接下载官方打包好的源码包，读者可以从Source Bundle Releases页面（地址： http://jdk7.java.net/source.html） 取得打包好的源码，到本地直接解压即可。一般来说，源码包大概一至两个月左右会更新一次，虽然不够及时，但比起从Mercurial复制代码的确方便和快捷许多。笔者下载的是OpenJDK 7 Update 6 Build b21版源码包，2012年8月28日发布，大概99MB，解压后约为339MB。

## 1.6.2　系统需求

&emsp;&emsp;如果可能，笔者建议尽量在Linux、MacOS或Solaris上构建OpenJDK，这要比在Windows平台上容易得多，本章实战中笔者将以Ubuntu 10.10和MacOS X 10.8.2为例进行构建。如果读者一定要在Windows平台上完成编译，可参考本书附录A，该附录是本书第一版中介绍如何在Windows下编译OpenJDK 6的例子，原有的部分内容现在已经过时了（例如安装Plug部分），但还是有一定参考意义，因此笔者没有把它删除掉，而是移到附录之中。

&emsp;&emsp;无论在什么平台下进行编译，都建议读者认真阅读一遍源码中的README-builds.html文档（无论在OpenJDK网站上还是在下载的源码包中都有这份文档），因为编译过程中需要注意的细节非常多。虽然不至于像文档上所描述的“Building the source code for the JDK requires a high level of technical expertise.Sun provides the source code primarily for technical experts who want to conduct research.（编译JDK需要很高的专业技术，Sun提供JDK源码是为了技术专家进行研究之用）”那么夸张，但是如果读者是第一次编译，那有可能会在一些小问题上耗费许多时间。

&emsp;&emsp;在本次编译中采用的是64位操作系统，编译的也是64位的OpenJDK，如果需要编译32位版本，那建议在32位操作系统上进行。在官方文档上写到编译OpenJDK至少需要512MB的内存和600MB的磁盘空间。512MB的内存也许能凑合使用，不过600MB的磁盘空间估计仅是指存放OpenJDK源码所需的空间，要完成编译，600MB肯定是无论如何都不够的，光输出的编译结果就有近3GB（因为有很多中间文件，以及会编译出不同优化级别（Product、Debug、FastDebug等）的虚拟机），建议读者至少保证5GB以上的空余磁盘。

&emsp;&emsp;对系统的最后一点要求就是所有的文件，包括源码和依赖项目，都不要放在包含中文的目录里面，这样做不是一定不可以，只是没有必要给自己找麻烦。

## 1.6.3　构建编译环境

&emsp;&emsp;在MacOS[1]和Linux上构建OpenJDK编译环境比较简单（相对于Windows来说），对于Mac OS，需要安装最新版本的XCode和Command Line Tools for XCode，在Apple Developer网站（https://developer.apple.com/）上可以免费下载，这两个SDK包提供了OpenJDK所需的编译器以及Makefile中用到的外部命令。另外，还要准备一个6u14以上版本的JDK，因为OpenJDK的各个组成部分（Hotspot、JDK API、JAXWS、JAXP……）有的是使用C++编写的，更多的代码则是使用Java自身实现的，因此编译这些Java代码需要用到一个可用的JDK，官方称这个JDK为"Bootstrap JDK"。如果编译OpenJDK 7，Bootstrap JDK必须使用JDK6 Update 14或之后的版本，笔者选用的是JDK7 Update 4。最后需要下载一个1.7.1以上版本的Apache Ant，用于执行Java编译代码中的Ant脚本。

&emsp;&emsp;对于Linux来说，所需要准备的依赖与Mac OS差不多，Bootstrap JDK和Ant都是一样的，在Mac OS中GCC编译器来源于XCode SDK，而Ubuntu中GCC应该是默认安装好的，需要确保版本为4.3以上，如果没有找到GCC，安装binutils即可，在Ubuntu 10.10下编译OpenJDK 7u4所需的依赖可以使用以下命令一次安装完成。
```shell
sudo apt-get install build-essential gawk m4 openjdk-6-jdk
libasound2-dev libcups2-dev libxrender-dev xorg-dev xutils-dev
x11proto-print-dev binutils libmotif3 libmotif-dev ant
```
## 1.6.4　进行编译

&emsp;&emsp;现在需要下载的编译环境和依赖项目都准备齐全了，最后我们还需要对系统的环境变量做一些简单设置以便编译能够顺利通过。OpenJDK在编译时读取的环境变量有很多，但大多都有默认值，必须设置的只有两个：LANG和ALT_BOOTDIR，前者是设定语言选项，必须设置为：
```shell
export LANG=C
```
&emsp;&emsp;否则，在编译结束前的验证阶段会出现一个HashTable内的空指针异常。另外一个ALT_BOOTDIR参数是前面提到的Bootstrap JDK，在Mac OS上笔者设为以下路径，其他操作系统读者对应调整即可。
```shell
export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.7.0_04.jdk/Contents/Home
```
&emsp;&emsp;另外，如果读者之前设置了JAVA_HOME和CLASSPATH两个环境变量，在编译之前必须取消，否则在Makefile脚本中检查到有这两个变量存在，会有警告提示。
```java
unset JAVA_HOME
unset CLASSPATH
```
&emsp;&emsp;其他环境变量笔者就不再一一介绍了，代码清单1-1给出笔者自己常用的编译Shell脚本，读者可以参考变量注释中的内容。

&emsp;&emsp;代码清单1-1　环境变量设置
```java
#语言选项，这个必须设置，否则编译好后会出现一个HashTable的NPE错
export LANG=C
#Bootstrap JDK的安装路径。必须设置
export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.7.0_04.jdk/Contents/Home
#允许自动下载依赖
export ALLOW_DOWNLOADS=true
#并行编译的线程数，设置为和CPU内核数量一致即可
export HOTSPOT_BUILD_JOBS=6
export ALT_PARALLEL_COMPILE_JOBS=6
#比较本次build出来的映像与先前版本的差异。这对我们来说没有意义，
#必须设置为false，否则sanity检查会报缺少先前版本JDK的映像的错误提示。
#如果已经设置dev或者DEV_ONLY=true，这个不显式设置也行
export SKIP_COMPARE_IMAGES=true
#使用预编译头文件，不加这个编译会更慢一些
export USE_PRECOMPILED_HEADER=true
#要编译的内容
export BUILD_LANGTOOLS=true
#export BUILD_JAXP=false
#export BUILD_JAXWS=false
#export BUILD_CORBA=false
export BUILD_HOTSPOT=true
export BUILD_JDK=true
#要编译的版本
#export SKIP_DEBUG_BUILD=false
#export SKIP_FASTDEBUG_BUILD=true
#export DEBUG_NAME=debug
#把它设置为false可以避开javaws和浏览器Java插件之类的部分的build
BUILD_DEPLOY=false
#把它设置为false就不会build出安装包。因为安装包里有些奇怪的依赖，
#但即便不build出它也已经能得到完整的JDK映像，所以还是别build它好了
BUILD_INSTALL=false
#编译结果所存放的路径
export ALT_OUTPUTDIR=/Users/IcyFenix/Develop/JVM/jdkBuild/openjdk_7u4/build
#这两个环境变量必须去掉，不然会有很诡异的事情发生（我没有具体查过这些"诡异的
#事情"，Makefile脚本检查到有这2个变量就会提示警告）
unset JAVA_HOME
unset CLASSPATH
make 2＞＆1|tee $ALT_OUTPUTDIR/build.log
```
&emsp;&emsp;全部设置结束之后，可以输入make sanity来检查我们前面所做的设置是否全部正确。如果一切顺利，那么几秒钟之后会有类似代码清单1-2所示的输出。

&emsp;&emsp;代码清单1-2　make sanity检查
```java
～/Develop/JVM/jdkBuild/openjdk_7u4$make sanity
Build Machine Information:
build machine=IcyFenix-RMBP.local
Build Directory Structure:
CWD=/Users/IcyFenix/Develop/JVM/jdkBuild/openjdk_7u4
TOPDIR=.
LANGTOOLS_TOPDIR=./langtools
JAXP_TOPDIR=./jaxp
JAXWS_TOPDIR=./jaxws
CORBA_TOPDIR=./corba
HOTSPOT_TOPDIR=./hotspot
JDK_TOPDIR=./jdk
Build Directives:
BUILD_LANGTOOLS=true
BUILD_JAXP=true
BUILD_JAXWS=true
BUILD_CORBA=true
BUILD_HOTSPOT=true
BUILD_JDK=true
DEBUG_CLASSFILES=
DEBUG_BINARIES=
……因篇幅关系，中间省略了大量的输出内容……
OpenJDK-specific settings:
FREETYPE_HEADERS_PATH=/usr/X11R6/include
ALT_FREETYPE_HEADERS_PATH=
FREETYPE_LIB_PATH=/usr/X11R6/lib
ALT_FREETYPE_LIB_PATH=
Previous JDK Settings:
PREVIOUS_RELEASE_PATH=USING-PREVIOUS_RELEASE_IMAGE
ALT_PREVIOUS_RELEASE_PATH=
PREVIOUS_JDK_VERSION=1.6.0
ALT_PREVIOUS_JDK_VERSION=
PREVIOUS_JDK_FILE=
ALT_PREVIOUS_JDK_FILE=
PREVIOUS_JRE_FILE=
ALT_PREVIOUS_JRE_FILE=
PREVIOUS_RELEASE_IMAGE=/Library/Java/JavaVirtualMachines/jdk1.7.0_04.jdk/Contents/Home
ALT_PREVIOUS_RELEASE_IMAGE=
```
&emsp;&emsp;检查通过。

&emsp;&emsp;Makefile的Sanity检查过程输出了编译所需的所有环境变量，如果看到"Sanity check passed."，说明检查过程通过了，可以输入"make"执行整个OpenJDK编译（make不加参数，默认编译make all），笔者使用Core i7 3720QM/16GB RAM的MacBook机器，启动6条编译线程，全量编译整个OpenJDK大概需20分钟，编译结束后，将输出类似下面的日志清单所示内容。如果读者之前已经全量编译过，只修改了少量文件，增量编译可以在数十秒内完成。
```shell
#--Build times----------
Target all_product_build
Start 2012-12-13 17:12:19
End 2012-12-13 17:31:07
00:01:19 corba
00:01:15 hotspot
00:00:14 jaxp
00:7:21 jaxws
00:8:11 jdk
00:00:28 langtools
00:18:48 TOTAL
-------------------------
```
&emsp;&emsp;编译完成之后，进入OpenJDK源码下的build/j2sdk-image目录（或者build-debug、build-fastdebug这两个目录），这是整个JDK的完整编译结果，复制到JAVA_HOME目录，就可以作为一个完整的JDK使用，编译出来的虚拟机，在-version命令中带有用户的机器名。
```java
＞./java-version
openjdk version"1.7.0-internal-fastdebug"
OpenJDK Runtime Environment（build1.7.0-internal-fastdebug-icyfenix_2012_12_24_15_57-b00）
OpenJDK 64-Bit Server VM（build 23.0-b21-fastdebug,mixed mode）
```
&emsp;&emsp;在大多数时候，如果我们并不关心JDK中HotSpot虚拟机以外的内容，只想单独编译HotSpot虚拟机的话（例如调试虚拟机时，每次改动程序都执行整个OpenJDK的Makefile，速度肯定受不了），那么使用hotspot/make目录下的Makefile进行替换即可，其他参数设置与前面是一致的，这时候虚拟机的输出结果存放在build/hotspot/outputdir/bsd_amd64_compiler2目录[1]中，进入后可以见到以下几个目录。
```shell
0 drwxr-xr-x 15 IcyFenix staff 510B 12 13 17:24 debug
0 drwxr-xr-x 15 IcyFenix staff 510B 12 13 17:24 fastdebug
0 drwxr-xr-x 15 IcyFenix staff 510B 12 13 17:25 generated
0 drwxr-xr-x 15 IcyFenix staff 510B 12 13 17:24 jvmg
0 drwxr-xr-x 15 IcyFenix staff 510B 12 13 17:24 optimized
0 drwxr-xr-x 584 IcyFenix staff 19K 12 13 17:25 product
0 drwxr-xr-x 15 IcyFenix staff 510B 12 13 17:24 profiled
```
&emsp;&emsp;这些目录对应了不同的优化级别，优化级别越高，性能自然就越好，但是输出代码与源码的差距就越大，难于调试，具体哪个目录有内容，取决于make命令后面的参数。

&emsp;&emsp;在编译结束之后、运行虚拟机之前，还要手工编辑目录下的env.sh文件，这个文件由编译脚本自动产生，用于设置虚拟机的环境变量，里面已经发布了"JAVA_HOME、CLASSPATH、HOTSPOT_BUILD_USER"3个环境变量，还需要增加一个"LD_LIBRARY_PATH"，内容如下：
```shell
LD_LIBRARY_PATH=.:${JAVA_HOME}/jre/lib/amd64/native_threads:${JAVA_HOME}/jre/lib/amd64:
export LD_LIBRARY_PATH
```
&emsp;&emsp;然后执行以下命令启动虚拟机（这时的启动器名为gamma），输出版本号。
```shell
../env.sh
./gamma-version
Using java runtime at:/Library/Java/JavaVirtualMachines/jdk1.7.0_04.jdk/Contents/Home/jre
java version"1.7.0_04"
Java（TM）SE Runtime Environment（build 1.7.0_04-b21）
OpenJDK 64-Bit Server VM（build 23.0-b21，mixed mode）
```
&emsp;&emsp;看到自己编译的虚拟机成功运行起来，很有成就感吧！

## 1.6.5　在IDE工具中进行源码调试

&emsp;&emsp;在阅读OpenJDK源码的过程中，经常需要运行、调试程序来帮助理解。我们现在已经可以编译出一个调试版本HotSpot虚拟机，禁用优化，并带有符号信息，这样就可以使用GDB来进行调试了。据笔者了解，许多对虚拟机了解比较深的开发人员确实就是直接使用GDB加VIM编辑器来开发、修改HotSpot的，不过相信大部分读者更倾向于在IDE环境而不是纯文本的GDB下阅读、跟踪HotSpot源码，因此这节就简单介绍一下“如何在IDE中进行HotSpot源码调试”。

&emsp;&emsp;首先，到NetBeans网站（http://netbeans.org/）上下载最新版的NetBeans，下载时选择支持C/C++开发的那个版本。安装后，新建一个项目，选择“基于现有源代码的C/C++项目”，在源码文件夹中填入OpenJDK目录下hotspot目录的路径，在下面的单选按钮中选择“定制”，如图1-8所示，然后单击“下一步”按钮。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30164710_i1lZ.png "在这里输入图片标题")

&emsp;&emsp;接着，在“指定构建代码的方法”中选择“使用现有的makefile”，并填入Makefile文件的路径（在hotspot/make目录下），如图1-9所示。单击“下一步”按钮，将“构建命令”修改为以下内容：
```shell
${MAKE}-f Makefile clean jvmg
ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.7.0_04.jdk/Contents/Home ARCH_DATA_MODEL=64 LANG=C
```
![输入图片说明](https://static.oschina.net/uploads/img/201703/30164841_Rv52.png "在这里输入图片标题")

&emsp;&emsp;OpenJDK 7u4源码Makefile在终端运行时能正确获取到系统指令集架构为64位，但在NetBeans中却没有取得正确的值，误认为是32位，因此这里必须使用ARCH_DATA_MODEL参数明确指定为64位。另外两个参数ALT_BOOTDIR和LANG的作用前面已经介绍过。单击“完成”按钮，HotSpot项目就这样导入到NetBeans中了。

&emsp;&emsp;不过，这时候HotSpot还运行不起来，因为NetBeans根本不知道编译出来的结果放在哪里、哪个程序是虚拟机的入口等，这些内容都需要明确告知NetBeans。在HotSpot工程上单击右键，在弹出的快捷菜单中选择“属性”，在弹出的对话框中找到“运行”选项，设置运行命令为：
```shell
/Users/IcyFenix/Develop/JVM/jdkBuild/openjdk_7u4/hotspot/build/bsd/bsd_amd64_compiler2/jvmg/gamma Queens
```
&emsp;&emsp;上面的Queens是Makefile脚本自动产生的一段解八皇后问题的Java程序，用于测试虚拟机，这里笔者直接拿来用了，读者完全可以将它替换为自己的Java程序。

&emsp;&emsp;读者在调试Java代码执行时，如果要跟踪具体Java代码在虚拟机中是如何执行的，也许会觉得无从下手，因为目前在HotSpot主流的操作系统上，都采用模板解释器来执行字节码，它与JIT编译器一样，最终执行的汇编代码都是运行期间产生的，无法直接设置断点，所以HotSpot增加了以下参数来方便开发人员调试解释器。
```shell
-XX:+TraceBytecodes-XX:StopInterpreterAt=＜n＞
```
&emsp;&emsp;这组参数的作用是当遇到序号为＜n＞的字节码指令时，便会中断程序执行，进入断点调试。在调试解释器部分代码时，把这两个参数加到gamma后面即可。

&emsp;&emsp;最后，还需要在“环境”窗口中设置环境变量，也就是前面env.sh脚本所设置的那几个环境变量，如图1-10所示。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30165121_hpvY.png "在这里输入图片标题")

&emsp;&emsp;完成以上配置之后，一个可修改、编译、调试的HotSpot工程就完全建立起来了，启动器的执行入口是java.c的main()方法，读者可以设置断点单步跟踪，如图1-11所示。

![输入图片说明](https://static.oschina.net/uploads/img/201703/30165222_Yzbi.png "在这里输入图片标题")

&emsp;&emsp;由于HotSpot的源码比较长，C/C++文件数量也很多，为了便于读者阅读，所以代码清单1-3给出了各个目录中代码的主要用途，供读者参考。

&emsp;&emsp;代码清单1-3　HotSpot源码结构

![输入图片说明](https://static.oschina.net/uploads/img/201703/30165352_kCKe.png "在这里输入图片标题")

![输入图片说明](https://static.oschina.net/uploads/img/201703/30165426_pPqw.png "在这里输入图片标题")