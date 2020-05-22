
## cURL是做什么的?

cURL是一个项目,它的主要目的和重点是制造两种产品:

-   命令行工具, curl

-   C API的传输库, libcurl

工具和库都使用因特网协议, 对指定的URL资源进行因特网传输.

任何与互联网协议传输相关的东西,都可以被看作是curl的业务.与之无关的事情应该 *要不起*,留给其他项目和产品.

同样重要的是,cURL和libcurl还考虑试图避免处理传输的实际数据.例如,它没有关于HTML的知识,或者任何关于常用HTTP传输的其他知识内容,但是它知道如何通过HTTP,传输这种数据.

这两种产品不仅经常用于为互联网连接的世界,驱动数千或数百万的脚本和应用程序,而且还广泛用于服务器测试、协议处理和尝试新事物.

该库用于任何需要网络传输的嵌入式设备:汽车娱乐、电视、蓝光播放器、机顶盒、打印机、路由器、游戏系统等.

### 命令行工具

自然运行curl命令行,默认情况下,Daniel从来没有考虑过,在向终端的stdout上输出数据."一切都是管道"的标准UNIX哲学的咒语是丹尼尔相信的东西. curl类似于"cat"或其他Unix工具之一; 它向stdout发送数据,以便易于与其他工具链接在一起以完成您想要的操作. 这也是为什么, 几乎所有允许从文件读取或写入文件的curl选项,都具有选择对stdout或从stdin进行读取的能力.

遵循Unix命令行工具的工作方式,它应该在命令行上支持多个URL也绝不是问题.

命令行工具被设计成从脚本,或其他自动化手段一起完美地工作.它不具有任何其他GUI或UI,而仅仅是文本进和文本出.

### libcurl库

当命令行工具首先出现时,网络引擎在2000年被拆卸并转换为一个库,我们今天仍然拥抱这个概念,自2000年8月libcurl 7.1的引入. 从那时起,命令行工具就是一层薄薄的逻辑,用来在库周围制作一个工具,完成所有繁重的工作.

libcurl被设计成可用于任何想要,将客户端文件传输能力添加到其软件、任何平台、任何体系结构和任何目的的任何人.libcurl也得到了极为宽松的许可,以避免成为障碍.

libcurl是用传统的和保守的C语言编写的.当然有其他语言,去[捆绑库](bindings.zh.md)了解人们的创建.