# AESbyJAVA
开发环境为IntelliJ IDEA 15.0.2

jre1.8.0_121 为运行环境，如果电脑里有64位的java环境可以删掉，运行AES.jar

AES可以导入到Intellij IDEA或Eclipse里运行，主类为SwingAES.java


文件加密及 SWING 界面设计 16bytes 对齐及文件加密的实现 我们知道，UTF-8 一个中文占 3 个字节，而一个数字占 1 个字节，一个英文字母也 占一个字节，所以为了保证加密解密结果的一致性，对文件字节的对齐就非常的重要。
 AES 加密要求一次加密 16bytes，如果输入不足 16bytes，我们的加密代码会补齐 0x00， 解密后在控制台输出是没有问题，但如果对文件进行加解密，例如对 txt 文本进行加密， 如果不足 16bytes，补齐以后，尾部会有空格（0x00）。这看起来可以接受，但如果对其 他的数据进行加密，则有可能因为解密结果和原加密结果不一样而失败。这是失败的加 密算法实现。所有我们需要对不足 16bytes 的数据进行加密，解密后则去掉多出的部分。 通过 JAVA 的文件输入输出流，我们可以把文件以 byte 读入写出。通过对加密文件 添加头部，可以保证只需要设置 AES，及输入输出，就可以对文件进行解密。头文件包 括两个 16bytes 加密信息，一个为加密时，文件读取的次数，这用于解密时，保证 CBC 解密的准确性；另一个为最后的补全字节数，这用于保证解密后解密文件和加密文件的 一致性。整个流程如下： 
 
 
图 2.1 文件 的加密解密 
SWING 界面的设计 界面使用了 SWING 设计，在设计时，预留了信息传递的接口。
 
 
  
图 2.2 文件 的加密解密 
三、加密性能优化 输入输出流及代码的一些优化 一开始以每次 16bytes 的速度读取文件，同时进行加密，解密，速度，282MB 的视 频文件时间为加密 60 多秒，解密时间为 70 多秒，这显然是不可以接受的。增大输入输 出流的大小后，加解密的时间明显变快 
这里有趣的地方在于，16000bytes读取的加密时间比16000bytes的加密时间居然长， 其实后面的测试表明，继续增大输入输出流，文件加密时间会继续增加，所以说明，在 16e2bytes 到 16e5bytes 之前大小的输入输出流对加密解密时间的影响不大，此时主要的 瓶颈在于加密解密算法消耗的时间。 在代码上，尽量采用的一些高性能的写法，比如使用 System.arraycopy()代替 for 循 环进行数组拷贝，据对比3，for 循环最慢，约为 clone 方法的 2 倍，约为 System.arraycopy 的 4~5 倍； 经过测试，并不是所有的方法和变量都写为静态的，运行速度是最快的，这里我也 不明白为什么，可能和 Java 的虚拟机运行机制有关系，因为有些对象、变量，及用及销 毁，反而会让整个程序更快，篇幅有限，而且这里不是讨论重点，以后可能会研究一下。 这里只说明，在代码上，也进行了一些特别的优化，让整个加密解密更快一些。 
 
  
                                       
多线程优化 

上一小节提到除了输入输出流会影响加密解密速度外，最影响加解密速度是算法本 身，因为算法每次只能加解密 16bytes，所以，多线程加密解密是提高速度的最好的方 法。 每个线程都是独立的加密，而且输入的数据流大小均为 16bytes 的倍数。如果最后 一次不为 16bytes 的倍数，则单独进行加密。多线程加密让加密程序有了加密 G 为单位 文件的能力，下面有一份不同线程下，不同数据流下的对 1.5GB（1,619,124,684bytes） 电影进行 128 位 ECB 加密解密测试的结果 测试机器 CPU 为 i7-4800MQ 8GB 内存，250G 固态硬盘 
每次读取字节数 线程数 加密时间 解密时间 
9*16e6 bytes 9 28.361 30.798 
9*16e5 bytes 9 23.987 26.25 
9*16e4 bytes 9 24.183 26.912 
9*16e6 bytes 6 29.206 31.535 
9*16e5 bytes 6 26.463 28.668 
9*16e4 bytes 6 23.899 27.168 
9*16e6 bytes 3 33.8 35.25 
9*16e5 bytes 3 29.327 30.343 
9*16e4 bytes 3 25.643 27.157 
      可以看到，多线程效果明显，顺便说一句，曾经我要测试单线程的速度，但由于太 慢，测试到了一半变放弃了，单线程速度可以参考之前的 282MB 测试数据。 
 
 
参考资料 
 
AES： 密码编码学与网络安全（第六版）中国工信出版集团 教学课程 PPT 第 5 章高级加密标准 aes 加密算法 java 代码实现 http://blog.csdn.net/vk5176891/article/details/43796153 维基百科 高级加密标准词条 
 
JAVA： Java 核心技术 卷Ⅰ（原书第 9 版） 机械工业出版社 
