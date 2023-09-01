# 学习路径

预期时间：耗时约一个月 All In

作者：想学理论的调包侠
链接：https://zhuanlan.zhihu.com/p/632281381
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



严格按照课程表上的日程安排来进行学习了. 一般是如下的计划:

1.  阅读**[xv6 book](https://link.zhihu.com/?target=https%3A//pdos.csail.mit.edu/6.S081/2020/xv6/book-riscv-rev1.pdf)**小书上相对应的章节. 了解这一章节的重点主旨, 同时在xv6 book里会对章节相关的xv6部分源代码进行讲解. 在这一步时, 不求甚解, 只图脑海中有个大致的框架和知道哪几个源代码文件和这章节有关联.
2.  扫读在课程表上重点指出的几个xv6**源代码**文件. 一般来说每节课也就1, 2个源文件, 量不大. 在这个阶段, 主要是看代码文件里不同函数的**呼叫依赖**, 对于具体怎么实现的可以先放一下. 争取做到在阅读完后, 对于一个相关功能, 能在心中大致说出它的一个函数呼叫链, 比如 **a() -> b() -> c() -> a()**.
3.  观看课程视频. 在这个时候, 由于**步骤1** 和 **步骤2** 的准备工作, 你已经对概念和具体实现都有了一定的了解了. 需要的就是听教授的课程把整个内容串起来, 并排疑解惑. 我在听课的时候发现, 很多时候我在阅读xv6 book小书和源代码时心中存下的疑惑, 都会被某些mit同学在课上提出来并被教授解答, 颇让人有心有灵犀一点通的舒畅感. 在听完课程后不要忘记把课程精要笔记读一下, 然后再回头把之前没读懂的源代码部分读完.
4.  完成相对应的实验**lab**. 大部分的lab难度不大, 在hints里都给出了很具体的一步步如何去解决这个问题的步骤. 稳扎稳打的逐步推进即可. 在遇到bug时可以先在代码里加入**printf**语句来看执行顺序 (我个人不怎么喜欢也不擅长用**gdb**来debug), 如果还是debug不成功, 不要浪费时间耗着, 直接来知乎看别人的代码和解析即可. 在看懂别人的代码后再把自己的代码改对, 但不要直接照抄别人的代码.

我个人的节奏一般是, **3**天一个循环♻️. 第一天完成步骤**(1)**和**(2)**, 第二天完成步骤**(3)**, 第三天完成步骤**(4)**, 然后休息一天进入下一个循环.



原作者的循环是指3天做完一个Lab，这其中包含的可能是不止一个Lecture和很多章的xv6 book需要阅读。所以加油吧

# Lecture 1

## Lab1



### 阅读Chapter 1

#### 1.1

这一章大概讲了：

- xv6中提供的系统调用
- shell是如何利用fork-wait来运行命令行命令的。
- shell是如何利用文件系统的接口和文件描述符机制、fork-wait来实现输入重定向和输出重定向的。

这一章涉及到的源代码文件有：

- `user/sh.c`：1、145、58/78

这一章的零碎知识点：

- wait(&status)的返回值是子进程的pid，子进程的退出值value（`exit(int value)`）需要查看status变量得知。
- 可执行文件的格式：ELF格式/PE格式
- exec系统调用
- 为什么会把fork和exec分开？

shell的基本流程：

1. 调用getcmd从用户获取一行输入，并解析。
2. 调用fork，生成一个子进程，在子进程里执行用户指定的命令。子进程会调用runcmd?（需要看了源代码才知道），然后会借助exec来执行相应的文件。
3. 在父进程调用wait，等待子进程执行命令结束。

#### 1.2

这一章大概讲了：

- 文件描述符
- 与I/O相关的系统调用
- 输入输出重定向是如何实现的
- 管道

涉及到的源代码：

- `user/she.c`：151、82

细碎的知识点：

- 获得文件描述符的方式：打开一个文件/设备/Socket/目录/创建管道、复制现存文件描述符。因为Unix系列一切皆文件的思想，所以凡是和I/O有关的都是使用文件描述符的形式。

- 文件描述符的好处是抽离了各种I/O形式的区别，使得他们看起来都是字节流。（the file descriptor interface abstracts away the differences between files, pipes, and devices, making them all look like streams of bytes.）

- 每个进程有一个私有的文件描述符表，所以每个进程的文件描述符空间都是从0开始的。

- Shell总是确保它的0/1/2号文件描述符是打开的。并且传统上这三个依次是标准输入、输出、错误流，Unix下经常利用这个传统来做输入输出重定向。

- 类似cat之类的命令，输入输出重定向并不是他们命令本身实现的，而是由Shell先在子进程完成了输入输出重定向，然后再用exec系统调用启动了命令。

  - cat本身只是很简单的从标准输入读取输入，然后输出到标准输出。是Shell利用重定向，把文件重定向到了标准输入和输出。

  - > cat doesn’t know whether it is reading from a file, console, or a pipe. Similarly cat doesn’t know whether it is printing to a console, a file, or whatever. The use of file descriptors and the convention that file descriptor 0 is input and file descriptor 1 is output allows a simple implementation of cat.

- **新分配的文件描述符，总是当前进程可用描述符中最小的那一个**。这也是Shell利用close和open系统调用实现输入输出重定向的基础。

  - 分配至少有三种方式：open、dup复制、pipe获得管道

  - >  A newly allocated file descriptor is always the lowest numbered unused descriptor of the current process.

- exec系统调用会保留子进程原有的文件描述符表，这个机制也是Shell实现重定向的基础之一。

- fork一个进程的时候，子进程会把父进程的表也复制过来。并且父子进程之间对于同一个文件的偏移量是共享的。

- dup系统调用的作用：

  > The dup system call duplicates an existing fifile descriptor, returning a new one that refers to the same underlying I/O object. Both file descriptors share an offset, just as the file descriptors duplicated by fork do

下面是一个实现cat的输入重定向的例子：

- 先关闭了文件描述符0，所以0变成当前可用的最小文件描述符。
- 然后open了input.txt。文件描述符0会被分配给这个文件。
- 最后用exec启动cat。因为exec会保留文件描述符表，所以之前重定向的操作得以保留。
- 又因为这是在子进程干的事情，所以不改变父进程，也就是Shell的文件描述符表，所以Shell的标准输入还是没变。

```c
char *argv[2];
argv[0] = "cat";
argv[1] = 0;
if(fork() == 0) {
close(0);
open("input.txt", O_RDONLY);
exec("cat", argv);
}
```

到这里我们就可以回答1.1节提出的问题了：为什么要把fork和exec分开？

- 因为这给予了父进程更加灵活的空间，在fork之后，exec之前，子进程实际上执行的还是从父进程复制的指令。这意味着父进程实际上可以以这种方式控制子进程的行为。在这fork和exec这两步之间，给予了父进程更多的空间和灵活性。
- 所以可以利用这种机制，在fork之后，exec之前，先完成输入输出的重定向，然后再去执行新的可执行文件。

#### 1.3

这一节主要讲了：

- 管道
- 将输入和输出重定向到管道的原理
- 父子进程利用管道通信的原理。
- 为什么用管道，而不用文件来作为进程间通信的媒介？

涉及的源代码：

- `user/sh.c`：100

管道本质上是内核中的一个小缓冲区，但是以一对文件描述符的方式暴露给了进程。

管道的性质：

- 从管道的读取端用read()系统调用读取的时候，如果管道是空的，read会阻塞，直到新的数据写入管道，或者所有指向管道写入端的文件描述符都被关闭了（这种情况下read返回0）。
  - 这也解释了为什么父子进程之间利用管道通信时，需要在子进程关闭管道的写入端。

- Shell是如何实现包含管道的命令的，类似：`grep fork sh.c | wc -l`：
  - Shell先fork出一个子进程A。
  - 子进程A创建一个管道，用于连接管道左边的`grep`子进程和右边的`wc`子进程。
  - 然后A再fork出子进程B和C，用于分别执行管道左边的命令和右边的命令。
  - 然后A再wait两次，等待两个子进程都结束。

一个利用管道+重定向实现父子进程通信的例子：

- 为什么要在子进程关闭管道的写入端？

> If no data is available, a read on a pipe waits for either data to be written or for all file descriptors referring to the write end to be closed; in the latter case, read will return 0, just as if the end of a data file had been reached. The fact that read blocks until it is impossible for new data to arrive is one reason that it’s important for the child to close the write end of the pipe before executing wc above: if one of wc ’s file descriptors referred to the write end of the pipe, wc would never see end-of-file.

```c
int p[2];
char *argv[2];
argv[0] = "wc";
argv[1] = 0;
pipe(p);
if(fork() == 0) {
close(0);
dup(p[0]);
close(p[0]);
close(p[1]);
exec("/bin/wc", argv);
} else {
close(p[0]);
write(p[1], "hello world\n", 12);
close(p[1]);
}
```

为什么使用管道，而不使用文件？

- 管道会自动清理自身，而临时文件需要人为清理。
- 管道可以容纳任意长度的字节流，而文件需要消耗足够的硬盘空间。
- 管道本身是实现了生产者消费者问题中阻塞读和阻塞写的，但是文件是没有这个实现的。管道本质上是一个生产者消费者问题，生产者消费者问题就需要生产者和消费者之间的同步（无空间是写入阻塞，无数据时读取阻塞），而文件是不具备这些优势的。

> Pipes have at least four advantages over temporary files in this situation. First, pipes automatically clean themselves up; with the file redirection, a shell would have to be careful to remove /tmp/xyz when done. Second, pipes can pass arbitrarily long streams of data, while file redirection requires enough free space on disk to store all the data. Third, pipes allow for parallel execution of pipeline stages, while the file approach requires the first program to finish before the second starts. Fourth,if you are implementing inter-process communication, pipes’ blocking reads and writes are more efficient than the non-blocking semantics of files

#### 1.4

这一节讲述的是文件系统

涉及的源代码：

- `kernel/stat.h`：
- `user/sh.c`

知识点：

- chdir()系统调用，改变进程当前工作目录的。

- mkdir()系统调用，创建新目录的。

- fstat()系统调用，用于从iNode中获取元数据，也就是文件的状态的。返回一个stat结构体

  - ```c
    #define T_DIR 1 // Directory
    #define T_FILE 2 // File
    #define T_DEVICE 3 // Device
    struct stat {
    int dev; // File system’s disk device
    uint ino; // Inode number
    short type; // Type of file
    short nlink; // Number of links to file
    uint64 size; // Size of file in bytes
    };
    ```

- link系统调用，用于创建硬链接的。

- 将名为b的文件硬链接到一个文件a，a和b底层是同一个iNode，也就是同一个文件。对文件b做的更改，文件a也可以看见，反之亦然。a和b的iNode号是同一个，iNode的引用计数是2。

- unlink系统调用，用于删除一个特定的硬链接的。

- 当文件的引用计数为零，**且没有文件描述符引用该文件时**，这个文件在硬盘上会被Free掉。

  - 示例代码：

  - ```c
    open("a", O_CREATE|O_WRONLY);
    link("a", "b");
    unlink("a");
    ```

- 大部分的Shell命令都是以外部命令的形式存在，除了`cd`。因为`cd`改变的是Shell本身的工作目录，所以如果`cd`也像常规的命令那样，fork出一个子进程去执行，那么`cd`改变的是子进程的工作目录，而不是Shell本身的。而让大部分命令以外部命令的形式存在，使得Shell的扩展性极好，只需要添加新的用户态程序，就相当于给Shell新加了一条外部命令。

  - 这一点其实和Shell执行命令的机制有关：先从`$PATH`中找到命令指定的可执行文件，然后fork出子进程，在子进程利用`exec`去执行那个可执行文件（可能有必要的重定向）。

### Lab

#### 安装qemu，启动xv6

我是在Windows下使用virtual box，Ubuntu 22.04.2 LTS

和知乎上的指导还是有一点区别的。需要额外安装三个依赖：

步骤如下：

```bash
# 按官方指南手册 安装必须的工具链
$ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu 
# 单独移除掉qemu的新版本, 因为不知道为什么build时候会卡壳
$ sudo apt-get remove qemu-system-misc
# 额外安装一个旧版本的qemu
$ wget https://download.qemu.org/qemu-5.1.0.tar.xz
$ tar xf qemu-5.1.0.tar.xz
$ cd qemu-5.1.0
```

到这里为止都和知乎一致

但是当执行到下面的时候，依次出现了如下的几个错误：

```bash
$ ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
```

第一个错误：

<img src="https://i.imgur.com/ARW0nmZ.png" alt="image-20230627114713943" style="zoom: 80%;" />

解决方法：安装`pkg-config`

<img src="https://i.imgur.com/w0I2bf0.png" alt="image-20230627114829378" style="zoom:80%;" />

然后接下来试图重新执行上面的config，出现了第二个错误：

<img src="https://i.imgur.com/Alb7lOM.png" alt="image-20230627114919348" style="zoom:80%;" />

解决方法也是安装对应的依赖，但是对于apt的包的名字我真的无力吐槽了。名字居然不叫glib-2.48是吧。

<img src="https://i.imgur.com/pv1rcJR.png" alt="image-20230627115031303" style="zoom:80%;" />

然后再重试命令，出现了第三个错误：解决方法是绿框。

<img src="https://i.imgur.com/vYEoq7A.png" alt="image-20230627115124095" style="zoom:80%;" />

然后终于可以进行下去了

 <img src="https://i.imgur.com/aLmfoUr.png" style="zoom:80%;" />

然后接下来的步骤就比较一致了：

make去编译qemu的代码需要比较久的时间，在`make`和`sudo make install`之间其实是等待了一段时间的。

克隆仓库的时候，记得最好把目录切换到`~`去。因为前几步的时候工作目录是qemu的源代码目录。

克隆完成之后会发现`xv6-labs-2020`目录是空的，别慌，切换一下分支就有了。

```bash
$ ./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
$ make
$ sudo make install

# 克隆xv6实验仓库
$ git clone git://g.csail.mit.edu/xv6-labs-2020
$ cd xv6-labs-2020
$ git checkout util

# 进行编译
$ make qemu
# 编译成功并进入xv6操作系统的shell
$ xv6 kernel is booting

$ hart 2 starting
$ hart 1 starting
$ init: starting sh
$ (shell 等待用户输入...)
 
# 尝试一下ls指令
```

最后也是成功启动了xv6

另外，退出xv6是先按Ctrl-a，然后再按x。不要按住Ctrl-a不放。

使用`make grade`来对自己的写的Lab代码做测试。

#### sleep

很简单的一个题。但是不知道为什么要让我去看`kernel/sysproc.c`去找sleep系统调用的实现，我感觉我也看不太懂。

- 先检查参数个数，参数个数一定是2，第0个是程序本身的名字，第1个是tick数。

- 使用`atoi()`来将ASCII字符转换为一个整数。然后把整数作为参数传递给已经提供的`sleep(int ticks)`系统调用。
- 别忘了调用`exit`返回即可。
- 在 `Makefile` 文件中添加配置， 在 `UPROGS` 项中最后一行添加 `$U/_sleep\`

> 如果运行命令 `./grade-lab-util sleep` 报 `/usr/bin/env: ‘python’: No such file or directory` 错误，请使用命令 `vim grade-lab-util`，把第一行 `python` 改为 `python3`。如果系统没装 `python3`，请先安装 `sudo apt-get install python3` 。

<img src="https://i.imgur.com/AMLhxvb.png" alt="image-20230627174738305" style="zoom:80%;" />

#### pingpang

父进程发一个字节给子进程，子进程收到之后打印一句"\<pid>: received ping",pid是子进程的pid，然后再把这个字节发回父进程，然后exit。父进程收到了子进程发给他的字节之后，打印一句"<pid\>: received pong"然后exit。

这个应该不算难：

1. pipe出两个管道A和B来
2. 父进程保留A的写端，关闭A的读端。保留B的读端，关闭B的写端。fork一个子进程
3. 子进程则恰好反过来。
4. 然后父进程借助A给子进程发一个字节。然后从B读
5. 子进程收到之后打印ping，把字节通过B发回去，exit。
6. 父进程从B读到之后，打印pang，exit



<img src="https://i.imgur.com/Xq5g4sj.png" alt="image-20230627215245794" style="zoom: 80%;" />

#### primes

算法的基本思路：

1. 每个进程被assign了一个质数, 它会把这个质数打印在屏幕上
2. 它从左手边的进程不断接收数字, 如果这个数字是自己质数的倍数, 就把它过滤掉. 不然就**fork**出一个新的进程, 把这个数字assign并传给那个进程
3. 每个进程都需等待它的子进程结束后才可以退出, 形成了一条进程生命周期依赖链

首先打开提示给出的链接，阅读并观察上面给出的图。由图可见，首先将数字全部输入到最左边的管道，然后第一个进程打印出输入管道的第一个数 `2` ，并将管道中所有 `2` 的倍数的数剔除。接着把剔除后的所有数字输入到右边的管道，然后第二个进程打印出从第一个进程中传入管道的第一个数 `3` ，并将管道中所有 `3` 的倍数的数剔除。接着重复以上过程，最终打印出来的数都为素数。

但是这个过程还需及时关闭不用的文件描述符，否则xv6的资源会耗尽。

所以编码的思路是：

1. 进程A将2-35全部输入管道P1
2. fork出子进程B，B打印从管道读取的第一个数N，这个数肯定是素数。
3. 然后B持续从管道P1读取，每读取到一个数，就判断其是否是N的倍数，是的话肯定不是素数。将不是N的倍数的数，输入管道P2
4. B再fork出子进程C，C从P2读取，然后重复B所做的工作。
5. 所以我们需要写一个函数，这个函数接收一个文件描述符作为参数（是管道的读取端），然后在函数里面，做B和C所做的工作。

# Lecture 2

# Lecture 3

### 阅读Chapter 2

这一章主要讲了xv6是如何实现：

- 进程间的隔离
- 进程间的通信（交互）
- 硬件的复用
- xv6的进程，以及xv6启动时的第一个进程是如何创建的。

聚焦于以宏内核为主流的操作系统。

#### 2.1

为什么不让程序直接与硬件交互？

> if there is more than one application running, the applications must be well-behaved. For example, each application must periodically give up the CPU so that other applications can run. Such a *cooperative* time-sharing scheme may be OK if all applications trust each other and have no bugs. It’s more typical for applications to not trust each other, and to have bugs, so one often wants stronger isolation than a cooperative scheme provides.

#### 2.2 User mode, supervisor mode, and system calls

- RISC-V有三种模式：机器模式、监管者模式、用户模式。
- 运行在机器模式下的指令拥有完全的特权，CPU启动于机器模式，机器模式主要是为了配置计算机。
- 监管者模式也是一种特权模式，可以运行特权指令，但是其特权要低于机器模式。xv6只在机器模式下执行几条指令，就会转变为监管者模式运行。
- 用户模式是非特权模式。如果处于用户模式的进程试图执行特权指令的话，CPU不会执行，并且会立即切换到监管者模式并将其杀死。
- 到底什么是内核？运行于监管者模式，或者说运行于内核空间的软件，就是内核。



CPU会提供将CPU「从用户模式转变为监管者模式，然后从内核指定的进入点进入内核」的特别指令，在RISC-V下是ECALL指令。

一旦切换到监管者模式，内核就可以检查系统调用的参数是否合法，然后决定应用是否被允许执行这个系统调用，然后执行或者拒绝。



为什么让内核来控制内核模式的进入点，而不是让应用程序来决定？为了保证安全，一个恶意的应用程序可能就会在参数合法性检查之后的点进入内核，直接越过参数合法性的检查。

#### 2.3 Kernel organization

这一节讲述宏内核的组织架构。



什么是宏内核？

- 整个操作系统都驻留在内核里，这样所有的系统调用都运行在监管者模式下。这就是宏内核

宏内核的好处是什么？

- 整个OS都带着完全的硬件特权在运行。
- 设计很方便。因为不需要考虑OS的哪个部分其实不需要完全的硬件特权，而是直接无脑给每一个部分完全的特权。

宏内核的坏处是什么？

- OS不同部分之间的接口很复杂，所以OS开发者很容易出错。而宏内核的情况下，内核出错就是崩溃性的，机器会宕机，需要重启。



与宏内核比较相反的是微内核。

- 微内核是最大程度上减少了运行在监管者模式下的内核代码，将大部分的内核代码运行在用户模式下。
- 如下图那样，文件系统之类的模块以一个用户空间的进程的形式运行和存在。
- 作为进程运行的OS服务，叫做一个Server。微内核下大部分模块的服务都以Server的形式存在。
- 内核提供「处于用户空间」的进程间通信的机制，以使得普通进程可以以发送消息的形式，去使用某个Server的服务（发消息==>等待Server响应）。

<img src="https://i.imgur.com/RkXTndn.png" alt="image-20230628154332481" style="zoom:80%;" />

> In a microkernel, the kernel interface consists of a few low-level functions for starting applications, sending messages, accessing device hardware, etc. This organization allows the kernel to be relatively simple, as most of the operating system resides in user-level servers.



xv6从概念上来说是一个宏内核

#### 2.4 Code: xv6 organization

xv6内核的代码在：`kernel/`文件夹。

`kernel/defs.h`定义了模块间接口。

#### 2.5 Process overview

- 进程是OS实行隔离的最小单元。
- 什么是隔离？隔离就是阻止一个进程去监视或者破解另外一个进程的内存、文件描述符、CPU等等。也包括阻止进程去破坏内核本身。
- **内核在实现隔离的过程中，用到的机制有**：
  - 用户/监管者模式标志位
  - 虚拟内存（地址空间）
  - 时间分片

- xv6使用的是页表来实现虚拟内存，将虚拟地址到物理地址的转换。

如下图是xv6进程的内存布局：

> Instructions come first, followed by global variables, then the stack, and finally a “heap” area (for malloc) that the process can expand as needed. 

RISC-V的指针是64位的，但是RISC-V只使用了其中的低39位，而xv6则只使用了39位中的低38位。

在xv6进程的内存布局上，顶部保留了一页给*trapframe*(陷阱帧)和一页给*trampoline*(跳板)。这两个概念会在Chapter 4解释。GPT说，陷阱帧大概是在发生异常、中断或系统调用等情况下，保存当前执行上下文和相关状态的信息。而跳板则是一种机制，用于在不同的执行上下文之间进行转换或传递控制。

<img src="https://i.imgur.com/APRdq4U.png" alt="image-20230628171015545" style="zoom:80%;" />





- xv6的内核为每个进程都维护了其状态，某个进程的状态都被集中在了一个结构体`struct proc(kernel\proc.h)`中。最重要的比如：页表（结合上下文来看存的应该是完整的页表本体？从Chapter 3 来看，应该是指针）、内核栈（(p->kstack）、运行状态。

对于这个结构体，其中的几个字段，直接引用原文：

> p->state indicates whether the process is allocated, ready to run, running, waiting for I/O, or exiting.
>
> p->pagetable holds the process’s page table, in the format that the RISC-V hardware expects. xv6 causes the paging hardware to use a process’s p->pagetable when executing that process in user space. A process’s page table also serves as the record of the addresses of the
>
> physical pages allocated to store the process’s memory.



每个进程都有两个栈：一个用户栈和一个内核栈。

- 当进程在执行用户指令（也即非特权指令）的时候，进程只使用用户栈，内核栈是空的。
- 而当进程进入内核的时候（比如系统调用或者中断），内核指令使用内核栈来执行。此时用户栈中仍旧保存着数据，只是不活跃，没有被使用。

一个进程的线程总是在「使用内核栈」和「使用用户栈」之间来回切换。



为什么需要一个内核栈（而不直接使用用户栈）？

- 内核栈是独立的，是被内核保护的，是不被用户代码操控的。安全
- 因为内核栈是独立的，所以即使一个进程已经破坏了其用户栈，内核还是可以执行下去。



一个进程在调用一个系统调用的时候，发生了什么？

1. 执行ecall指令。
2. ecall指令将特权级提高、将程序计数器的值设置为内核事先定义好的入口点。
3. 入口点的代码切换到内核栈，然后去执行实现了系统调用的那些内核指令。
4. 系统调用执行完毕，内核切换回用户栈。通过执行sret指令返回用户空间。
5. sret指令会将特权级降低，并且恢复执行系统调用指令后的下一条用户指令（把PC的值设置成下一条用户指令的地址即可）



#### 2.6 Code: starting xv6 and the first process

这一小节outline出内核是如何启动和运行第一个进程的。

出现在这一节的机制，都会在后续的章节讲解。



当RISC-V计算机通电的时候，它初始化自己并且运行一个 存储在ROM里的boot loader（引导加载程序）。

 boot loader将xv6内核载入内存。然后，在机器模式下，CPU从`_entry(kernel\entry.S:6)` 开始执行xv6。

xv6启动的时候分页硬件是禁用的，虚拟地址和物理地址是一样的，二者是直接映射。



bootloader将内核加载到物理地址`0x80000000`的地方。之所以不是加载到物理地址0开始的地方，是因为物理地址`0`到`0x80000000`之间包含的是I/O设备。

`_entry`里的指令会设置一个栈，这样xv6才能运行C代码。Xv6在`kernel\start.c:11`中为初始栈`stack0`声明了空间，而`_entry`中的代码会用地址`stack0+4096`来填充栈顶指针寄存器`sp`（XV6的栈是向下增长的）。既然现在内核有个栈了，`_entry`就调用`start.c:21`中的C代码。



函数`Start`执行一些配置工作，这些配置只允许在机器模式下进行，然后再切换到监管者模式。

- RISC-V提供了一个`mret`指令，用于从前一个从监管者模式到机器模式的调用的返回（前一次调用是监管者模式==>机器模式，然后mret用于从机器模式返回到监管者模式）
- 我猜m是machine的意思？

Start在进入监管者模式之前，依次做了哪些事呢？

1. 将记录特权级的寄存器`mstatus`设置为监管者模式
2. 将返回地址设置为`main(kernel\main.c:11)`（将`main`的地址写入寄存器`mepc`）
3. 禁用监管者模式下的虚拟地址（在页表寄存器`satp`写入0）。
4. 将所有中断和异常委托给监管者模式（怎么委托的？）
5. 编程时钟芯片，以生成时钟中断。（为什么需要）

做完这些事情之后，Start调用`mret`返回监管者模式，同时PC会被设置成返回地址，也就是main的地址。

在main初始化了几个设备和子系统之后，它调用`userinit(kernel\proc.c:212)`来创建第一个进程。

这个进程执行一个用RISC-V汇编编写的小程序，`user\initcode.S`。这个小程序重新进入内核，调用`exec`来执行可执行文件`/init`。而init（源代码`user\init.c`）会创建一个新的控制台设备（console device）文件（如果需要），然后将其作为文件描述符0、1、2打开，然后启动一个Shell。然后系统就启动了。

### 阅读源码

读完第二章之后，个人感觉比较难的还是2.6节。前面几节讲述的都比较的通用和概念性。

2.6节则是深入源码

要求阅读的源码：

- `kernel/proc.h`：其中有保存进程状态的结构体`struct proc`
- `kernel/defs.h`：定义了内核模块之间的接口
- `kernel/entry.S`：Bootloader将内核载入之后，CPU运行的第一个程序。调用`Start.c`
- `kernel/main.c`：调用`proc.c`。
- `user/initcode.S`：创建第一个进程init，启动一个Shell
- `user/init.c`

需要浏览（skim）的：

- `kernel/proc.c`
- `kernel/exec.c`



阅读的顺序，个人认为应该是：

1. `kernel/defs.h` ==> `kernel/proc.h`。这里和进程有关
2. 然后按照xv6启动的顺序：`kernel\entry.S` ==> `kernel\start.c:11` ==> `kernel\main.c:11` ==> `kernel\proc.c:212` ==> `user\initcode.S` ==> `user\init.c`。这里的箭头代表调用方向。



`kernel/defs.h`基本上全是按照`.c`文件分好类的函数原型的集合。



`proc.h`主要是一些结构体：

- `struct context`：保存进程的上下文的寄存器的值。
- `struct cpu`：CPU的状态。
- `struct trapframe`：陷阱帧结构体
- `struct proc`：进程状态的结构体。包含：
  - 进程运行状态、是否被Kill、是否在睡眠中、PID
  - 指向内核栈的指针（虚拟地址）、进程内存的大小、用户态的页表（~~就是页表本体~~，从Chapter 3来看，应该是指向第一级页表的指针）
  - 一个陷阱帧结构体、一个context结构体
  - 一个`struct file *`指针数组，用于记录进程打开了哪些文件。
  - 一个`struct inode *`指针变量，用于记录Current Working Directory
  - 一个`char`数组，是这个进程的名字。

真的，还是真的要去读源码，以前学习OS的时候一个个虚拟的概念，现在在眼前都有了一一的实体去对应，原来它真的就在那里。



`_entry.S`是用RISC-V汇编写的一小段汇编代码。在注释中说明了，qemu将内核装载到`0x80000000`，然后让CPU跳转到这个地址。而`_entry.S`的代码，就被放在`0x80000000`。下面是代码原文，我还有一些问题。

- `stack0`是在`start.c`内声明的栈起始地址。每个栈的大小是4096字节。每个CPU一个。
- 假定我们有两个CPU0和CPU1
- 根据注释 `sp = stack0 + (hartid * 4096)`来看，这两个CPU的栈应该是物理地址上连续的两段。hartid就是CPUID(还是CPUID + 1?)
- CPU0的栈起始地址就是stack0，从`stack0`到`stack0+4096`是CPU0的栈空间。
- CPU1的栈起始地址是`stack0+1 * 4096`，从`stack0+4096`到`stack + 4096 + 4096`是CPU1的栈空间。

所以我的问题就是，为什么在用`csrr a1, mhartid`获取了`mhartID`放在a1之后，还需要`addi a1, a1, 1`，让a1自增1？然后再去算栈顶指针。

还是说，我上面的解读其实有点问题，hartID和CPUID其实是恰好相差1的关系？CPU0的栈顶应该是`stack0 + (0+ 1) * 4096`?而CPU1的栈顶应该是 `stack0 + (1+ 1) * 4096`

抛开这个疑问不谈，这段代码干了什么事情还是很清楚的：为每个CPU设置好其栈，然后调用`start`

```assembly
	# qemu -kernel loads the kernel at 0x80000000
        # and causes each CPU to jump there.
        # kernel.ld causes the following code to
        # be placed at 0x80000000.
.section .text
_entry:
	# set up a stack for C.
        # stack0 is declared in start.c,
        # with a 4096-byte stack per CPU.
        # sp = stack0 + (hartid * 4096)
        la sp, stack0 # sp = stack0
        li a0, 1024*4 # 寄存器a0 = 4096
        csrr a1, mhartid #  获取CPUID(mhartid)，保存到寄存器a1。a1 = CPPUID
        addi a1, a1, 1 # a1 = CPUID  + 1。个人猜测这里是为了得到所谓的hartID。hartID比CPUID恰好大1，就跟数组第N个元素在a[N-1]类似？
        mul a0, a0, a1 # a0 = a0(4096) * a1(CPUID + 1)
        add sp, sp, a0 # sp = stack0 + a0.将栈顶指针正确设置好。
	# jump to start() in start.c
        call start
spin:
        j spin
```

然后去扫读了一下`kernel\start.c`，有如下的发现：

- `stack0`实际上是一个`char`数组，它的大小是`4096 * NCPU`，也就是实际上`stack0`是一个大小为4096 * CPU个数的`char`数组。也就是一个这么长的内存，然后用作各个CPU的栈，每个CPU占其中的4096字节。所以在`_entry.S`中设置的`sp`其实可以看做是数组的索引？
  - 我说`sp`是数组索引的意思是，CPU0的栈起始是0(sp = 0)，结束于4095，CPU1的栈起始是4096(sp = 4096)。目前结合下来看应该是这样的？

```c
// entry.S needs one stack per CPU.
__attribute__ ((aligned (16))) char stack0[4096 * NCPU];
```

然后看了一下`start`函数，大部分代码看不太懂（实际上Lecture 3也没有要求我们看`start.c`），但是有注释。大概和阅读xv6 book的时候看到的差不多。start在返回监管者模式之前，做了如下几件事：

- 将`mstatus`寄存器设置成监管者模式
- 将main的地址设置为返回地址
- 禁用页表（禁用分页）
- 将所有中断和异常委托给监管者模式
- 初始化时钟中断（为什么要这一步？）
- 调用`mret`，返回。



接着去阅读了一下`kernel/main.c`。老样子，很多函数不知道是干嘛的，但是有注释，所以我知道它干了什么。

- `main()`针对不同的CPUID有不同的动作。如果是CPUID0，那么让CPU0来负责**初始化**一个console，打开分页，初始化文件表等一大堆初始化工作。而其他的CPU则先轮循等待CPU0初始化好，然后做少量每个CPU都要做的初始化工作。
  - 而创建第一个进程的工作，就是CPU0在做。
- 在做完初始化工作之后，不管是哪个CPU，main中的最后一步都是`scheduler()`，也就是进入调度器



接着去阅读CPU0调用的`userinit()`所在的文件`kernel/proc.c`。Lecture要求我们快速浏览，应该就是看一下这个`userinit()`吧。

- 大概就是先从进程表找一个未使用的项（是一个`struct proc`），然后初始化这个结构体，然后分配一页内存，把`/init`的指令和数据丢进内存区。

接着去阅读`user\initcode.S`本体了，就是`init`进程的汇编。

汇编代码如下：汇编代码应该是按照顺序依次指向了标签`start`和标签`exit`的内容。最后两个标签`init`和`argv`则分别定义了一些字符串内容。

- `start`标签将`init`字符串的地址存入寄存器`a0`，将`argv`的地址存入`a1`。这应该是`exec()`系统调用的两个参数，`a0`保存的是第一个参数「可执行文件路径」的地址，而`a1`保存的则是第二个参数「命令行参数」的地址。然后将`exec()`的系统调用号存入寄存器`a7`，这是`ecall`指令的参数。最后使用`ecall`指令触发系统调用的执行，`ecall`会检查寄存器`a7`的系统调用号。
- `exit`标签是一个无限循环，它所做的事就是无限调用`exit()`系统调用，直到进程退出。
- `init`标签则定义了一个字符串`"/init\0"`，它实际上是要`exec()`执行可执行文件`/init`。也就是exec的第一个参数。
- `argv`标签则定义了另外一个字符串数组`[init, 0]`。这实际上是程序`init`的`main`函数的命令行参数，也就是exec的第二个参数。

所以`initCode.S`所做的事实际上就是一件事：执行系统调用`exec("/init\0", {"init", "0"})`

```assembly
# Initial process that execs /init.
# This code runs in user space.

#include "syscall.h"

# exec(init, argv)
.globl start
start:
        la a0, init
        la a1, argv
        li a7, SYS_exec
        ecall

# for(;;) {exit();}
exit:
        li a7, SYS_exit
        ecall
        jal exit

# char init[] = "/init\0";
init:
  .string "/init\0"

# char *argv[] = { init, 0 };
.p2align 2
argv:
  .long init
  .long 0

```

最后就是看一下`user\init.c`了，这个`.c`文件是`initCode.S`中执行的可执行文件`/init`的源代码。



这个源代码是Linux下的`init`进程的简单实现版。`init`进程是一个特殊的进程，它永远不会退出（肯定，不然假如Shell挂了怎么办？肯定要init负责重启Shell），负责启动其他用户进程，并在其他进程终止时进行清理工作。在正常情况下，init 进程是一直运行的，并且通常具有进程ID 1。

这个进程有必要贴出其源码，并附上我自己写的注释解读。



首先引用GPT的回答：

> Q:为什么会出现wpid不等于pid的情况呢?
>
> 在代码中，`wpid` 不等于 `pid` 的情况可能发生在两种情况下：
>
> 1. 子进程退出：当子进程结束时，父进程通过调用 `wait` 函数等待子进程退出并获取子进程的退出状态。此时，`wait` 函数返回的子进程ID（`wpid`）与之前创建的子进程ID（`pid`）不相等，因为父进程可能同时监控多个子进程的退出情况。在这种情况下，父进程会继续等待其他子进程的退出，而不是重新启动 shell。
> 2. 父进程不存在的子进程退出：在某些情况下，父进程可能在子进程结束之前提前退出。这种情况下，子进程会变成孤儿进程，即没有父进程的进程。孤儿进程会被操作系统的 init 进程接管，并由 init 进程负责回收资源。当父进程不存在时，父进程调用 `wait` 函数会返回一个非负值的子进程ID（`wpid`），但该子进程ID与之前创建的子进程ID（`pid`）不相等。在这种情况下，代码中的注释部分提到"it was a parentless process; do nothing"，即父进程不存在的子进程退出时不做任何处理。
>
> 总之，`wpid` 不等于 `pid` 表示当前返回的子进程ID并非之前创建的子进程ID，可能是其他子进程退出或父进程不存在的子进程退出。在这两种情况下，代码中采取不同的处理方式，保证了程序的正常运行和资源的回收。

```c
// init: The initial user-level program

#include "kernel/types.h"
#include "kernel/stat.h"
#include "kernel/spinlock.h"
#include "kernel/sleeplock.h"
#include "kernel/fs.h"
#include "kernel/file.h"
#include "user/user.h"
#include "kernel/fcntl.h"

char *argv[] = { "sh", 0 };

int
main(void)
{
  int pid, wpid;
    /*
    这里先试图打开一个console设备文件，如果成功打开的话，这个文件会和电脑的键盘和屏幕连接。
    如果打开失败的话，会进入IF，先调用mknod创建一个console设备文件，然后再open它。
    */
  if(open("console", O_RDWR) < 0){
    mknod("console", CONSOLE, 0);
    open("console", O_RDWR);
  }
    /*
    在打开console设备文件之后，利用dup系统调用，把标准输入、标准输出、标准错误，都定向到console设备文件上去
    */
  dup(0);  // stdout
  dup(0);  // stderr

    /*
    然后进入这个循环，这个外层循环是永远不会退出的。意味着init进程是永远不会结束的。
    */
  for(;;){
    printf("init: starting sh\n");
      /*
      从44行到54行
      fork出一个子进程，然后调用exec，启动一个Shell。
      正常情况下子进程去执行Shell了，54行以后的代码，子进程是不会执行的，只有init会执行。
      而不正常的情况下，子进程在52行就exit(1)了，也不会执行54行以后的代码
      所以从54行之后的代码，我们只考虑init执行。
      */
    pid = fork();
    if(pid < 0){
      printf("init: fork failed\n");
      exit(1);
    }
    if(pid == 0){
      exec("sh", argv);
      printf("init: exec sh failed\n");
      exit(1);
    }

    for(;;){
      // this call to wait() returns if the shell exits,
      // or if a parentless process exits.
      wpid = wait((int *) 0);
      if(wpid == pid){
        // the shell exited; restart it.
        break;
      } else if(wpid < 0){
        printf("init: wait returned an error\n");
        exit(1);
      } else {
        // it was a parentless process; do nothing.
      }
    }
      /*
      从55行到68行，是一个子循环。这个子循环在正常情况下，一旦进入，就不会退出。
      因为，正常情况下，只要Shell没有挂，wait返回的wpid就不可能等于pid。因为pid是init进程fork出来的子进程ID。
      而正常情况下，Shell是不会挂的，所以正常情况下，wait返回的wpid，就会是其他进程的pid(父进程被杀死的进程，会由init来接管。)。
      所以，假如wpid == pid，那就说明Shell挂了，init有义务重启Shell，所以需要break跳出子循环，在外层循环重启Shell。
      所以，else的分支，什么都不做。因为那里的情况是，一个（不是由initfork出来的）被init接管的进程退出了。
      */
  }
}
```

Lecture还要求我们快速浏览一下`kernel/exec.c`。看不太懂，看了Lecture再回来看看。

### Lecture

#### 提问汇总

这个提问我觉得很有意思。但是老师的回答断了。

> 学生提问：之前提到，设置处理器中kernel mode的bit位的指令是一条特殊权限指令，那么一个用户程序怎么才能让内核执行任何内核指令？因为现在切换到kernel mode的指令都是一条特殊权限指令了，对于用户程序来说也没法修改那个bit位。
>
> Frans教授：你说的对，这也是我们想要看到的结果。可以这么来看这个问题，首先这里不是完全按照你说的方式工作，在RISC-V中，如果你在用户空间（user space）尝试执行一条特殊权限指令（后面Frans那边的Zoom就断了，等他重新接入，他也没有再继续回答，所以后半段回答是我补充的）用户程序会通过系统调用来切换到kernel mode。当用户程序执行系统调用，会通过ECALL触发一个软中断（software interrupt），软中断会查询操作系统预先设定的中断向量表，并执行中断向量表中包含的中断处理程序。中断处理程序在内核中，这样就完成了user mode到kernel mode的切换，并执行用户程序想要执行的特殊权限指令。

另外一个提问：OS会将进程尽量调度到它之前使用过的CPU上，以减少Cache Miss

> 学生提问：更复杂的内核会不会尝试将进程调度到同一个CPU核上来减少Cache Miss？
>
> Frans教授：是的。有一种东西叫做Cache affinity。现在的操作系统的确非常复杂，并且会尽量避免Cache miss和类似的事情来提升性能。我们在这门课程后面介绍高性能网络的时候会介绍更多相关的内容。

Kernel/User mode在硬件形式上表现为一个Bit的差别，标志位。设置这个标志位的指令是特权指令

> 学生提问：如果kernel mode允许一些指令的执行，user mode不允许一些指令的执行，那么是谁在检查当前的mode并实际运行这些指令，并且怎么知道当前是不是kernel mode？是有什么标志位吗？
>
> Frans教授：是的，在处理器里面有一个flag。在处理器的一个bit，当它为1的时候是user mode，当它为0时是kernel mode。当处理器在解析指令时，如果指令是特殊权限指令，并且该bit被设置为1，处理器会拒绝执行这条指令，就像在运算时不能除以0一样。
>
> 同一个学生继续问：所以，唯一的控制方式就是通过某种方式更新了那个bit？
>
> Frans教授：你认为是什么指令更新了那个bit位？是特殊权限指令还是普通权限指令？（等了一会，那个学生没有回答）。很明显，设置那个bit位的指令必须是特殊权限指令，因为应用程序不应该能够设置那个bit到kernel mode，否则的话应用程序就可以运行各种特殊权限指令了。所以那个bit是被保护的，这样回答了你的问题吗？

内核如何从死循环的进程中夺回控制权？

> 学生提问：当应用程序表现的恶意或者就是在一个死循环中，内核是如何夺回控制权限的？
>
> Frans教授：内核会通过硬件设置一个定时器，定时器到期之后会将控制权限从用户空间转移到内核空间，之后内核就有了控制能力并可以重新调度CPU到另一个进程中。我们接下来会看一些更加详细的细节。

#### 课上其他内容

**内核在实现隔离的过程中，用到的机制有**：

- 用户/监管者模式标志位
- 虚拟内存（地址空间）
- 时间分片



有一种方式可以使得用户的应用程序能够将控制权以一种协同工作的方式转移到内核，这样内核才能提供相应的服务，这种方式是什么？

- 是ECALL指令，借助ECALL可以让应用程序可以将控制权转移给内核。

借助ECALL指令来调用系统调用的时候，实际上发生了什么？（并不是简单地像调用一个普通方法一样直接去调用内核里的那个系统调用，而是有一些其他的事）

> ECALL接收一个数字参数，当一个用户程序想要将程序执行的控制权转移到内核，它只需要执行ECALL指令，并传入一个数字。这里的数字参数代表了应用程序想要调用的System Call（也就是系统调用号）。ECALL会跳转到内核中一个特定的，由内核控制的位置，每一次应用程序执行ECALL指令，应用程序都会通过这个接入点进入到内核中。举个例子，不论是Shell还是其他的应用程序，当它在用户空间执行fork时，**它并不是直接调用操作系统中对应的函数**，而是调用ECALL指令，并将fork对应的数字作为参数传给ECALL。之后再通过ECALL跳转到内核，在内核侧，有一个位于syscall.c的函数syscall，每一个从应用程序发起的系统调用都会调用到这个syscall函数，之后控制权到了syscall函数，syscall函数会检查ECALL的参数，通过这个参数内核可以知道需要调用的是fork。



宏内核的优点和缺点？

- 处于内核模式下的代码数量多，所以出现在内核模式下的bug的数量从概率上来说就会更多。而在内核模式下出bug基本上就是电脑崩溃。
- 优点：各个子模块联系比较紧密，集成度高，可以提供高性能。

微内核的优点和缺点：微内核就是把大部分的代码放在用户态，很多模块以一个运行在用户态下的普通用户进程的形式存在，最小化运行在内核态的代码（内核只维护一些诸如进程间通信、页表之类的最基本的东西）。

- 优点：处于内核模式下的代码行数少，所以在内核模式下出现bug的数量从概率上来说会更少。
- 缺点：模块间通信（或者普通用户程序想使用OS的服务）需要以消息的形式（借助内核的IPC机制），需要一来一回，「跳入内核再跳出内核」分别两次。所以，微内核的性能会差一点。
- 另外一点就是，模块之间的集成度会更低，因为每个模块的服务都是以一个进程存在的，进程之间的隔离性较强（比如无法做到宏内核那样，多个模块共享缓存），所以要获得高性能比较难。

# Lecture 4

### 阅读Chapter 3

这一章讲述xv6的页表。还提到了一个叫做`trampoline page`的东西。

`trampoline page`第一次出现在前一个Chapter的2.5节。中文名似乎叫做跳板页。

> Page tables also provide a level of indirection that allows xv6 to perform a few tricks: mapping the same memory (a trampoline page) in several address spaces, and guarding kernel and user stacks with an unmapped page.

#### 3.1 Paging hardware

**RISC-V的指令都是操纵虚拟地址的。**



xv6运行在`Sv39 RISC-V`上，39的意思是一个64比特的虚拟地址，只有其中的低39位是使用中的（高25位是保留的，未使用的）。

所以虚拟地址的地址空间是$[0, 2^{39}]$，然后根据书上的内容反推，xv6每一页的大小是$2^{12} B = 4 KB$。反推的依据如下：

- 逻辑上来说，一个页表项对应一个物理页（以及同样大小的虚拟页）。所以书上说有2的27次方个项，就说明有这么多页。从而反推，每一页的大小是4KB。

> In this Sv39 configuration, a RISC-V page table is logically an array of $2^{27}$ (134,217,728) *page table entries (PTEs)*.

当然其实往后面多看几行，就发现书上说了一页是4096字节。。



xv6的页表可以看做一个长度为2的27次方的PTE数组。每个PTE包含一个44比特的物理页号（physical page number，PPN），以及一些标志位。

分页的硬件取虚拟地址的高27比特（总长39比特）来作为索引，从页表中找到对应的项，然后把PTE的44比特物理页号和虚拟地址的剩余12位作为低位，拼接到一起，得到一个56比特的物理地址。

- 所以，虚拟地址的长度和物理地址的长度，是可以不一样的？
- xv6的一个56比特的物理地址，高44位来自PPN，低12位来自原始的虚拟地址。
- 低12位实际上是页内偏移量。

> The paging hardware translates a virtual address by using the top 27 bits of the 39 bits to index into the page table to find a PTE, and making a 56-bit physical address whose top 44 bits come from the PPN in the PTE and whose bottom 12 bits are copied from the original virtual address. 

<img src="https://i.imgur.com/WYpbQrb.png" alt="image-20230630200614713" style="zoom:80%;" />



上面的实际上是一个简化过的情况，也就是一个一级页表的情况。但是实际上，xv6的页表是三级的。



xv6的页表是存储在物理内存的3级的树。总共是27比特，要均分成3部分，所以每个部分9比特。

树的根节点是一页4KB的页表页，包含512（2^9）个PTE项（每个PTE项8字节），每一个PTE包含树的第二级的一个页表页的物理地址（存储在PTE的44比特的物理地址部分）。第二级的每个页表页，也是包含512个PTE的页，每个PTE包含一个第三级的页表页。每个第三级的页表页也有512个PTE（也就是实际存储了与虚拟地址对应的物理地址的PTE）。

分页硬件使用27比特中的最高位9比特，找到根节点的某个PTE，然后找到其指向的二级页，然后使用中间的9比特，从二级页找到某个PTE，然后找到其指向的三级页，然后使用最低的9比特，找到要找到那个三级PTE。



其实可以这么理解：第一级是一个长度为512的数组，数组的每一项指向一个长度也为512的数组，而这个数组的每一项，也指向一个长度为512的数组，最后这个数组，每个项是一个真正的PTE。我们先用高9位索引第一个长度为512的数组，找到其指向的第二个长度为512的数组，然后使用中9位索引第二个数组，找到其指向的第三个长度为512的数组，最后使用低9位，从第三个数组找到我们要的PTE项。

<img src="https://i.imgur.com/O8k7qoL.png" alt="image-20230630202108449" style="zoom:80%;" />



也就是说，我们在将一个虚拟地址翻译成物理地址的过程中，总共会遇到三个PTE（每一级各一个）。如果这三个中的任意一个PTE不存在（看PTE的标志位，具体来说是V位），分页硬件就会触发一个缺页异常（*page-fault exception*），然后让内核来处理。



分三级的好处是什么？当连续一大块虚拟地址不存在映射的时候，可以直接省去整页的页表（3级的一页，甚至2级的一页）

> This three-level structure allows a page table to omit entire page table pages in the common case in which large ranges of virtual addresses have no mappings.



PTE的标志位们：像什么V、R、W就不多说了。

- V特殊提一嘴，当V位设置为非法的时候，去引用这一页是会触发缺页异常的。
- R和W就是是否可读和是否可写。
- X的意思是，CPU能否把这一页的内容解释为指令，并执行它们。
- U的意思是，是否可以在用户态访问这一页。如果U未设置，那这一页只允许在内核模式下访问。

这些标志位和其他与分页硬件有关的结构都定义在`kernel/riscv.h`



内核会将页表的根节点（root page-table page）的**物理地址**写入寄存器`satp`。

每个CPU都有它自己的`satp`寄存器，当翻译虚拟地址的时候，使用这个寄存器所指向的页表。



每个CPU都有自己的`satp`寄存器的好处是什么？不同的CPU可以运行不同的进程，每个进程都有自己的地址空间。

或者可以反过来说，因为我们需要不同的CPU可以同时运行不同的进程，而每个进程都有自己的页表，所以我们需要每个CPU都有自己的`satp`寄存器

> Each CPU has its own *satp* so that different CPUs can run different processes, each with a private address space described by its own page table



我们已经知道物理地址、物理内存、虚拟地址是什么了。那虚拟内存又是什么呢？是机制和抽象的集合

> Unlike physical memory and virtual addresses, virtual memory isn’t a physical object, but refers to the collection of abstractions and mechanisms the kernel provides to manage physical memory and virtual addresses.

#### 3.2 Kernel address space

xv6为每一个进程都维护一个页表，以及仅仅一个用于描述内核地址空间的页表。

- 每个进程一个页表 + 内核一个页表

xv6内核的地址空间布局比较特殊，因为内核既要访问物理内存，也要访问各种硬件资源。它会把物理内存和各种硬件资源映射到"predictable virtual addresses"。（`kernel/memlayout.h`）

（最好还是参照最下面的图来阅读）

就QEMU模拟的硬件来说，RAM（也就是模拟的物理内存），是放在了物理地址的`0x80000000`到至少`0x86400000`（这个结束值被xv6叫做`PHYSTOP`，我猜是Physical Stop）



QEMU的模拟也包括了I/O设备（比如硬盘的接口），QEMU将这些设备的接口以内存映射的形式暴露给软件（映射到了物理地址空间中，物理地址小于`0x80000000`的位置）。

内核则通过读写这些特殊物理地址的方式，来与这些I/O设备交互。



内核使用的虚拟地址到物理地址的映射是“直接映射”，也就是说，虚拟地址和其对应的物理地址，是值上相等的。所以实际上，把资源映射到某个虚拟地址，就是映射到了与虚拟地址相等的物理地址上。

就比如内核本身就是被加载到了物理地址的`KERNBASE=0x80000000`，这实际上也是虚拟地址的`0x80000000`。

直接映射的好处就是，简化了内核读写物理内存的代码。很简单，因为物理地址可以直接当虚拟地址用，虚拟地址也可以直接当物理地址用。不需要转换。



也就是说，内核使用的实际上是虚拟地址，只不过这个虚拟地址和与之对应的物理地址的值是相等的。

询问了一下GPT，得到了肯定的回答。

> 在操作系统内核的内部，通常使用直接映射的虚拟地址。直接映射意味着虚拟地址的值和对应的物理地址是相等的，这种映射关系在内核页表中进行管理。
>
> 当操作系统运行在虚拟内存模式下时，物理内存被划分为固定大小的页面，同时也将虚拟地址空间划分为相同大小的页面。内核使用页表来管理虚拟地址到物理地址的映射关系。对于内核空间的页表项，通常采用直接映射的方式，将虚拟地址的高位部分与物理地址的高位部分保持相等。这样，在内核内部执行时，可以直接使用虚拟地址作为物理地址来访问内存，无需进行额外的转换。
>
> 需要注意的是，对于用户空间的页表项，通常采用的是虚拟地址和物理地址不相等的映射方式，因为用户空间的虚拟地址空间和物理地址空间是分开管理的。操作系统通过页表将用户空间的虚拟地址映射到相应的物理地址，这样用户程序就可以访问到自己的内存空间。
>
> 总结起来，在内核的内部，直接映射的虚拟地址用于访问物理内存。而对于用户空间，使用虚拟地址到物理地址的映射关系来实现虚拟内存的机制。



**但是还有一部分内核虚拟地址，不是直接映射的**。

- 跳板页（The trampoline page）：跳板页在虚拟地址空间中被映射到了最顶部。这个映射在用户程序的页表里也是一样的。跳板页的作用会在Chapter 4讨论。但是我们现在可以看到页表的一种有意思的用例：一个物理页（存放量跳板代码）被映射了两次，一次被映射到虚拟地址空间顶部，还有一次直接映射。
- 内核栈们（The kernel stack pages.注意复数）。其实看图也知道，所有的内核栈应该是由内核在统一管理的，每个进程的内核栈都由内核统一管理。每个进程的内核栈都被映射到了虚拟地址空间的顶部，这样xv6就可以在两个相邻的内核栈之间，设置一个Guard Page。这个Guard Page本身是标记为非法的，这样假如内核从某个站Overflow了，就会碰到Guard Page，就会触发一个异常。触发异常比直接越界到其他进程的内核栈去写入要更加好。
  - kstack0和kstack1之间的Guard Page可以作为两个栈之间的警戒线，一旦其中一个写入越界了，就会先触碰到Guard Page，就会触发异常，警示内核。如果没有Guard Page，那溢出的内核栈会直接覆盖另外一个内核栈，导致错误的运算结果。但是因为触碰到Guard Page，导致崩溃，也比错误的运算结果要更好。


当然，内核栈也可以使用直接映射的方式访问，另外一种设计上的选择就是，使用直接映射来使用内核栈。

但是我有一个问题：为什么这种情况下提供Guard Page会涉及未映射的虚拟地址？

>  An alternate design might have just the direct mapping,and use the stacks at the direct-mapped address. In that arrangement, however, providing guard pages would involve unmapping virtual addresses that would otherwise refer to physical memory,which would then be hard to use.

内核将跳板页和Kernel Text标记为R-X（读和执行），因为内核会读取和执行这两个部分的指令。内核将其他页的权限标记为R-W（读和写）。而Guard Page的权限则是Invalid。

<img src="https://i.imgur.com/ftcfomM.png" alt="image-20230630230845583" style="zoom:80%;" />

#### 3.3 Code: creating an address space

大部分用于操纵虚拟地址空间和页表的代码位于`kernel/vm.c`：

- 中心的数据结构是`pagetable_t`，它实际上是一个指向第一级页表页（root page-table page）的指针。这个指针可能指向内核页表，也可能指向一个普通的进程页表。
- 函数`walk`，用于找到给定虚拟地址的PTE。
- 函数`mappages`，用于给新的映射关系添加PTE。
- 以`kvm`开头的函数用于操控内核页表。
- 以`uvm`开头的函数用于操控用户页表。
- 其他的函数是二者共用的。
- `copyout` and `copyin` copy data to and from user virtual addresses provided as system call arguments;~~（虽然我觉得名字是不是应该相互交换一下）~~
  - 阅读源代码的时候，原来，`copyout`指的是从内核复制到用户空间出来，所以是out。因为是From Kernel to User Space，所以是从Kernel里面Out出来。而`copyin`就是反向的，所以是从Kernel外面In进去。




在xv6启动的时候，main会调用`kvminit()`来创建内核页表。这个调用发生在xv6启用分页之前，所以地址是直接引用物理内存的。`kvminit`先分配一页物理页用来保存root page table page，然后调用`kvmmap`添加内核需要的映射关系。

> The translations include the kernel’s instructions and data, physical memory up to PHYSTOP, and memory ranges which are actually devices.

`kvmmap`会调用`mappages`，对于一个虚拟地址的范围（以及其对应的物理地址范围），`mappages`会以页为单位去初始化每个虚拟地址对应的PTE（调用walk找到每个虚拟地址对应的PTE（同一页的所有地址对应同一个PTE），然后去初始化找到的PTE，初始化包含其物理地址，以及相应的标志位）



walk函数做的事就简单了，把27比特分成9+9+9，然后去找到每一级的PTE，要么是指向下一级的PTE，要么是最终的PTE。如果PTE非法，就说明请求的页还未分配，假如`alloc`参数被置位了，那么`walk`会分配一页新的页表页，然后把它的物理地址放入刚才非法的PTE。walk会返回最后一级的PTE的地址。



上述需要要求虚拟地址是直接映射的。



`main`调用`kvminithart`来安装页表，`kvminithart`实际上就是把内核页表第一级的物理地址写入CPU的`satp`寄存器。在这之后CPU就使用内核页表来翻译虚拟地址了。

（所以启用分页实际上就是把第一级页表的地址写入satp寄存器，禁用就是往寄存器里面写0？）



`procinit`（随后被`main`调用）给每个进程分配一个内核栈，它将每个内核栈映射到由KSTACK生成的虚拟地址（并且有Guard Page）。

`kvminit`将PTE们加入内核页表，而main对`kvminithart`的调用将内核页表重新装载到`satp`，这样硬件就知晓了新的PTE们。



所以我有一个问题，PTE到底是Walk加到页表里去的，还是kvminit？





关于TLB，以及TLB作废：



每个RISC-V的CPU都有TLB缓存，里面缓存了一些PTE。每当CPU改变页表（也就是说改变`satp`寄存器的值）的时候，就要将TLB里面缓存的项废除掉。因为TLB里缓存的PTE是前一个进程的PTE，如果后一个进程照旧用的话，可能会把虚拟地址映射到前一个进程的物理地址去，这就破坏了进程之间的隔离性，可能导致一个进程在另外一个进程的物理内存里乱涂乱画。

RISC-V有一个`sfence.vma`指令，它会刷新当前CPU的TLB缓存。所以在`kvminithart`重载了`satp`寄存器之后，xv6就会执行`sfence.vma`。

总之，只要`satp`寄存器的值改变了，就需要将TLB原有的缓存都作废，这一点可以通过`sfence.vma`指令达成。



#### 3.4 Physical memory allocation

xv6使用内核结尾和PHYSTOP之间的物理内存，来做运行时的内存分配（比如给页表、用户内存、内核栈之类的分配内存）

- 内核栈不是由内核统一管理的吗？放在内核地址空间的顶部？

分配和释放都是以4KB一页为单位。

xv6将空闲的页串成一条链表，分配的时候从链表去除一项，释放的时候就将被释放的页加入链表。

#### 3.5 Code: Physical memory allocator

物理内存分配器的代码在`kernel/kalloc.c`。

分配器的数据结构就是一个由可分配的空闲物理页组成的free list。这个free list实际上是由`struct run`组成的一条链表，每个空闲的物理页对应一个`struct run`，这个`struct run`在free list中代表这个物理页。

那么问题来了，分配器从哪来的内存来保存这个数据结构的？每个物理页的`struct run`就存储在这个物理页本身的内存里面。

free list被一个自旋锁保护着。这个在这里只是提一嘴，Chapter 6会将讲锁的事情。

- 看完代码回来：实际上是把一个`struct run`指针和一个自旋锁一起组成了一个结构体。指针负责指向链表。


为什么需要给free list加锁？

我个人觉得free list需要被自旋锁保护着是因为，内存是多个CPU共享的，而每个CPU都有分配和回收内存的需要，这就会导致free list是可能会被多个CPU修改的，所以需要一个锁来保证free list的修改是线程安全的。



`main`调用`kinit`来初始化分配器。`kinit`初始化free list以容纳从内核结尾到PHYSTOP之间的每一页。

`kinit`调用`freerange`，通过逐页调用`kfree`，来将内存加入free list。一开始的时候分配器是没有内存可以分配的，kfree的调用给了它一些。

一个PTE的物理地址部分，只能是4096字节的整数倍（与4096字节对齐）。因为内存的分配是按4KB的页的。所以`freerange`会使用`PGROUNDUP`来确保只释放对齐的物理地址（也就是只释放物理地址是4096的倍数的）。



分配器有时候会把地址当做整数来对待（比如遍历从地址A到地址B范围内的所有页），有时候把地址当指针用（不就是指针吗）。所以分配器里的代码到处都是C的类型转换。



`kfree`函数在释放内存的时候，会把内存里的每个字节都写满1，这样假如代码用了释放后的内存，只能读取到垃圾，读取不到旧的数据。读到垃圾值也许会让程序更快崩溃。

> This will cause code that uses memory after freeing it (uses “dangling references”) to read garbage instead of the old valid contents; hopefully that will cause such code to break faster.



`kfree`应该是前插的？因为原文涉及几个变量，需要看代码才知道，所以这里就直接把原文先贴下面：

- 看完源代码回来：插入和删除都是从头部去做。

> Then kfree prepends the page to the free list: it casts *pa* to a pointer to *struct run*, records the old start of the free list in *r->next*, and sets the free list equal to *r. kalloc* removes and returns the first element in the free list.

#### 3.6 Process address space

每个进程都有自己独立的页表，CPU切换进程的时候也会改变页表（改变`satp`寄存器的值）。一个进程的虚拟地址空间可以从0开始，到一个最大值`MAXVA(kernel/riscv.h：348)`，这里最多是256GB。



当一个进程向xv6请求更多内存时：

1. xv6先调用`kalloc`来分配物理内存页（allocate physical **pages**）
2. 向进程的页表里添加对应的PTE项（如果分配的页数超过一页，会添加多个PTE），PTE项的物理地址部分指向新分配的物理页。（adds **PTEs** to the process’s page table that point to the new physical pages）
3. 设置PTE中的R、W、X、U、V等标志位。

大部分的进程不会使用掉整个用户地址空间的，xv6会把未使用的PTE的V位置空。



页表的好处是什么？

- 第一点，将进程隔离开来，每个进程都有自己私有的内存，并且进程之间无法直接相互访问内存
- 第二点，给进程一个从0开始到最大值的，**连续的**地址空间。尽管在物理层面上进程的物理内存实际上是不连续的。

我觉得值得关注的是第三点：跳板页在物理内存上是只存在一页的，但是利用页表的机制，我们可以把很多个进程的虚拟地址空间的顶部（顶部指数值较高的虚拟地址）都映射到这一页上，同一个物理页可以出现在许多个不同的虚拟地址空间。

> We see here a few nice examples of use of page tables. First, different processes’ page tables translate user addresses to different pages of physical memory, so that each process has private user memory. Second, each process sees its memory as having contiguous virtual addresses starting at zero, while the process’s physical memory can be non-contiguous. Third, the kernel maps a page with trampoline code at the top of the user address space, thus a single page of physical memory shows up in all address spaces.

下面是一个正在运行中的进程的内存布局，stack部分的内容展示的是其刚被`exec`创建出来时，栈里的样子。

栈刚好是一页。

当进程刚被`exec`创建出来的时候，栈里只有以下的内容：

- 栈顶是N个命令行参数本体（每个都是字符串，也就是argv数组里的指针指向的地方）
- 然后是这N个参数的地址（也就是argv数组里的值，这实际上就是argv数组本体）
- 然后是第0个参数的地址的地址（其实就是argv数组的地址）
- 然后是参数个数`argc`
- 然后是`main`的地址，当从exec返回的时候，PC的值会被设置成Main的地址？

xv6同样在栈的底部（顶部？）放置了一个Guard Page，这样可以检测到Overflow的栈。当Overflow发生的时候，会触碰到Guard Page，触发硬件生成一个Page Fault。当然，现代的操作系统应该是会自动给溢出的栈分配更多的内存。

<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230702142603194.png" alt="image-20230702142603194" style="zoom:80%;" />

#### 3.7 Code: sbrk

`sbrk`是用于给进程增加或者缩减内存的。`sbrk`是被函数`growproc(kernel/proc.c)`实现的，`growproc`调用`uvmalloc`或者是`uvmdemalloc`，取决于参数`n`是正的还是负的。`uvmalloc`使用`kalloc`分配物理页，然后使用`mappages`将PTEs添加到用户页表。`uvmdemalloc`调用`uvmunmap`，`uvmunmap`则调用`walk`来找到PTEs，然后`kfree`找到的PTEs指向的物理页。



一个进程的页表不仅用来将虚拟地址翻译成物理地址，也是哪些物理页被分配给这个进程的唯一记录。

#### 3.8 Code: exec

`exec`是创建地址空间的用户部分的。

`exec`使用`namei(kernel/exec.c)`打开一个二进制文件`path`。然后读取文件的ELF头部。Xv6应用使用ELF格式来描述的（`kernel/elf.h`），一个ELF二进制文件包含一个ELF Header（`struct elfhdr(kernel/elf.h:6)`）和一系列至少一个的program section header（`struct proghdr (kernel/elf.h:25)`），每个`proghdr`描述了应用必须被加载到内存的一个部分。Xv6只有一个`proghdr`。



exec读取文件之后的第一步是检查文件的前四个字节（所谓的Magic Number）。一个ELF文件以四字节的Magic Number，四个字节分别是`0x7F、'E'、'L'、'F'`。如果读取到刚才打开的文件的前四个字节是`0x7f,e,l,f`，那么exec就会认为这个二进制文件是符合ELF格式的。



然后`exec`使用`proc_pagetable`分配一个空的（指没有虚拟地址映射的）新页表，然后使用`uvmalloc`给每个ELF段分配内存，然后使用`loadseg`将每个段载入内存。`loadseg`使用`walkaddr`找到刚才分配的物理页的物理地址以写入ELF段，使用`readi`从文件读取。

其实概括一下exec就是按顺序做了下面的几件事：

1. 读取二进制文件，校验其头部的魔数，魔数对得上就认为文件没问题。
2. 分配一个空的页表
3. 给ELF每个段分配物理页
4. 把ELF的每个段写入刚才分配的物理页

以`/init`的程序头部为例：

- 程序头的`filesz`可能会小于`memsz`。分配内存的时候是按照`memsz`分配的，这种时候二者之间的差值使用0来做填充。例子中的`filesz`是2112，而`memsz`是2136，这意味着`uvmalloc`会分配足够容纳2136字节的空间，但是只会从文件读取2112字节，其余字节用0填充。

```bash
# objdump -p _init
user/_init: file format elf64-littleriscv
Program Header:
LOAD off 0x00000000000000b0 vaddr 0x0000000000000000
paddr 0x0000000000000000 align 2**3
filesz 0x0000000000000840 memsz 0x0000000000000858 flags rwx
STACK off 0x0000000000000000 vaddr 0x0000000000000000
paddr 0x0000000000000000 align 2**4
filesz 0x0000000000000000 memsz 0x0000000000000000 flags rw
```

之后`exec`就给栈分配1页的空间，并初始化栈。初始化栈的时候，exec将命令行参数逐个复制到栈的顶部（每个都是一个字符串），在末尾放一个空指针表示参数结束。使用`ustack`记录每个参数的地址。初始情况下的栈的结构其实可以往上翻那个图。

`exec`会在栈的下方放一个Guard Page。这个Page的作用不再赘述。直接引用原文

> so that programs that try to use more than one page will fault. 
>
> This inaccessible page also allows exec to deal with arguments that are too large; in that situation, the copyout (kernel/vm.c:355) function that exec uses to copy arguments to the stack will notice that the destination page is not accessible, and will return -1.



在准备新内存映像的过程中，如果exec检测到类似无效程序段的错误，它会跳到标签bad，释放新映像，并返回-1。

`exec`在确定系统调用一定会成功之前，都必须等待着，不能释放掉旧的镜像（意思就是，在没把握exec一定成功之前，不能释放掉旧的镜像，只能等着，等到有把握的时候再释放）。也就是说，如果旧的镜像已经丢了，这个系统调用不能返回-1。

- 我觉得理由还是很明显的，因为在旧的镜像，`exec()`的后面往往还有一句`exit(1)`。后面这句是用于`exec()`失败的时候传递失败信息的，所以在`exec()`有成功的把握之前，不能释放掉旧的镜像。

> Exec must wait to free the old image until it is sure that the system call will succeed: if the old image is gone, the system call cannot return -1 to it.

在exec中，唯一的错误情况发生在创建镜像的时候，所以一旦创建好镜像，exec就可以提交新的页表，释放旧的页表。



exec中存在的风险：虽然说了一大堆，但是都是没啥用的废话。

因为`exec`会将来自ELF文件中的字节加载到ELF文件指定的地址，而用户或者进程可以在ELF文件里指定任何地址。这就使得ELF文件中指定的地址，可能是一个指向内核的地址。对于一个不做检查的内核来说，后果可能从崩溃，到内核的隔离机制被恶意颠覆和利用。

#### 3.9 Real world

Xv6假设物理地址`0x8000000`处是物理RAM开始的地方，但是现实世界并不是这样的。真正的硬件会将RAM和设备放在不可预知的物理地址，所以在`0x8000000`也许不会有RAM。

在内存容量比较大的机器上，可能会使用RISC-V的超级页特性。其实页的大小就是一个操控内存的粒度的问题，页大粒度就粗，页小一点粒度就细一点。

> On machines with lots of memory it might make sense to use RISC-V’s support for “super pages.” Small pages make sense when physical memory is small, to allow allocation and page-out to disk with fine granularity. For example, if a program uses only 8 kilobytes of memory, giving it a whole 4-megabyte super-page of physical memory is wasteful. Larger pages make sense on machines with lots of RAM, and may reduce overhead for page-table manipulation



什么是Super Page技术？https://www.reddit.com/r/RISCV/comments/v895do/what_are_super_pages_wrt_page_tables/

- **并不是简单的使用更大的单页尺寸**。
- 超级页的，操纵内存的粒度是更粗的，所以内部碎片问题会更加大一点。
- 直接让GPT翻译了一下：

> 以使用SV39为例，页表树共有三个级别。
>
> 通常情况下，第一级PTE指向第二级页表，第二级PTE指向第三级页表，而第三级PTE将虚拟内存的4 KB范围映射到物理内存的4 KB范围。
>
> 然而，任何PTE都可以被标记为叶节点，前提是4位的类型字段不是0000或0001。
>
> 如果第二级PTE是叶节点，则将连续且对齐的2 MB虚拟内存范围映射到连续且对齐的2 MB物理内存范围。
>
> 如果第一级PTE是叶节点，则将连续且对齐的1 GB虚拟内存范围映射到连续且对齐的1 GB物理内存范围。
>
> 超级页的优点在于它们大大减少了进程所需的不同映射数量（对于兆页减少了512倍，对于千兆页减少了262144倍），因此给定数量的TLB条目可以更好地利用，TLB缺失的可能性也更小。它们还使得在TLB缺失时的页表遍历稍微更快一些。
>
> 超级页的缺点在于进程可能并不需要那么多的内存，因此可能会浪费内存空间



xv6内核缺少一个类似`malloc`的分配器，所以Xv6没有办法使用那些复杂的、需要动态分配内存的数据结构。

### 阅读源码

Lecture要求阅读的源码有：

- `kernel/memlayout.h`：关于内核的内存布局
- `kernel/vm.c`：用于操纵虚拟地址空间和页表的代码
- `kernel/kalloc.c`：内存分配器
- `kernel/riscv.h`：定义了分页有关的标志位和结构
- `kernel/exec.c`：exec系统调用的实现



阅读顺序，个人准备按照下面的顺序：

1. `kernel/riscv.h`：标志以及与分页有关的其他硬件有关的结构。
2. `kernel/memlayout.h`：内核的地址空间布局。
3. `kernel/kalloc.c`：然后看一下关于内存分配器的，因为`vm.c`调用了分配器
4. `kernel/vm.c`：看看操纵页表和分配内存的那些东西
5. `kernel/exec.c`：看看exec的实现

####  riscv.h

`riscv.h`中关于页表的部分从314行开始。

首先比较值得关注的是两个宏`PGROUNDUP`和`PGROUNDDOWN`。这两个宏分别用于将给定的地址向上和向下取整到4096的倍数（也就是计算与之最接近的两个4096的倍数）。比如给个3000，向上取是4096，向下是0。

```c
#define PGSIZE 4096 // bytes per page
#define PGSHIFT 12  // bits of offset within a page
#define PGROUNDUP(sz)  (((sz)+PGSIZE-1) & ~(PGSIZE-1))
#define PGROUNDDOWN(a) (((a)) & ~(PGSIZE-1))
/*
如果要写成函数的形式，大概就是:
*/
int PageRoundUp(int size) {
    return (size + PGSIZE -1) & ~(PGSIZE - 1); 
}
int PageRoundDown(int a) {
    return (a & ~(PGSIZE - 1));
}
```

然后另外定义了几个宏，用于方便的查看PTE的标志位。以及方便地给PTE的标志位设置值。

- 大概猜测一下，比如我要查看R位的值，就用PTE_R和PTE去做按位与（`PTE_R & PTE`）应该就可以了？
- 比如我要将PTE设置为可读的时候，那就用PTE_R和PTE去按位或（`PTE_R | PTE`），就可以把R位设置为1。

```c
#define PTE_V (1L << 0) // valid
#define PTE_R (1L << 1)
#define PTE_W (1L << 2)
#define PTE_X (1L << 3)
#define PTE_U (1L << 4) // 1 -> user can access
```

还有一个宏，`PA2PTE`（Physical Address to PTE），用于将给定一个物理地址，**构建其对应的PTE的物理页号部分**（不是找到PTE的索引）。以及另外一个反向宏，`PTE2PA`（这个宏其实就是从PTE里面提取出物理页号）。最后一个宏`PTE_FLAGS(pte)`，用于提取出全部的第十位标志位（`0x3FF`刚好就是10比特1）。

- ~~这两个宏我没看懂其原理。我还想问的是，为什么给定一个物理地址（物理页号 + 页内偏移量），可以反推到其PTE的索引？我只见过给定虚拟地址然后找到PTE的。~~
- `PA2PTE`这个宏，其实是从物理地址里，先右移12位，去掉偏移量，然后提取出物理页号PPN。然后把物理页号左移10位，因为在一个页表项里面，PPN是PTE的高位，最低的10位是标志位，所以要左移10位，腾出低10位的空间。
- `PTE2PA`这个宏，其实就是，给定一个64比特的PTE比特流，先右移10位，去掉低10位的标志位，然后再左移12位，得到物理页号PPN。
- 需要结合下面的PTE的结构来看。宏里的`pte`，指的就是一个64比特的PTE比特流，而不是PTE索引（更何况索引是三级的）。

```c
// shift a physical address to the right place for a PTE.
#define PA2PTE(pa) ((((uint64)pa) >> 12) << 10)

#define PTE2PA(pte) (((pte) >> 10) << 12)

#define PTE_FLAGS(pte) ((pte) & 0x3FF)
```

- <img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230630202108449.png" alt="image-20230630202108449" style="zoom:80%;" />



接下来是三个用于提取虚拟地址的高27位（9+9+9）的三个级别的索引的宏：

- 掩码刚好是9个二级制的1。利用掩码来提取某一级索引的9个比特。**9+9+9三级分别是第2,1,0级**。
- `PXSHIFT(level)`，参数`level`指的是要提取第几级的索引。这个宏根据级数level，计算出应该把虚拟地址右移几位。
  - 原理很简单，因为虚拟地址是三级27位的虚拟页号，拼接上低位12位的页内偏移量。所以假如我要第3级，我就要把虚拟地址右移12+9+9位，才可以把第三级变为最低的9位，然后再用掩码去得到这9位。
- `PX(level, va)`，va是Virtual Address。这个宏的做法很简单，根据要提取的是第几级的索引，把虚拟地址右移`PXSHIFT(level)`位，使得要提取的索引变为最低的9位，然后再用掩码去按位与，即可得到索引的值。

```c
#define PGSHIFT 12
// extract the three 9-bit page table indices from a virtual address.
#define PXMASK          0x1FF // 9 bits
#define PXSHIFT(level)  (PGSHIFT+(9*(level)))
#define PX(level, va) ((((uint64) (va)) >> PXSHIFT(level)) & PXMASK)
```

最后剩下的几个宏：`MAXVA`是最大的虚拟地址，1左移38位。剩下两个`typedef`其实就是说指针是64位的。

```c
// one beyond the highest possible virtual address.
// MAXVA is actually one bit less than the max allowed by
// Sv39, to avoid having to sign-extend virtual addresses
// that have the high bit set.
#define MAXVA (1L << (9 + 9 + 9 + 12 - 1))

typedef uint64 pte_t;
typedef uint64 *pagetable_t; // 512 PTEs
```

在看了后续的代码之后，发现除了上述以外，还有一些汇编指令的函数之类的：

- 用于刷新TLB的`sfence_vma()`
- 用于写入CPU的页表地址寄存器satp的`w_satp()`：现在看来satp应该是Supervisor Address Translation and Protection
- 依次把代码附在下面：
- 此外还有许多读写其他寄存器的函数，也定义在了`riscv.h`

```c
// supervisor address translation and protection;
// holds the address of the page table.
static inline void 
w_satp(uint64 x)
{
  asm volatile("csrw satp, %0" : : "r" (x));
}

static inline uint64
r_satp()
{
  uint64 x;
  asm volatile("csrr %0, satp" : "=r" (x) );
  return x;
}

// flush the TLB.
static inline void
sfence_vma()
{
  // the zero, zero means flush all TLB entries.
  asm volatile("sfence.vma zero, zero");
}

```



#### memlayout.h

有一部分关于各种设备什么什么的。目前都没提到过。

- PLIC
- CLINT
- UART

去掉这些我不太了解的寄存器、中断控制器之类的之后，剩下的部分：

- `TRAMPOLINE`跳板页就真的放在了虚拟地址空间的最顶上。MAXVA - 4KB的地方是起始地址。
- `TRAPFRAME`，**在用户地址空间上**，就和跳板页紧贴在一起。
- 内核地址空间上，内核栈就紧贴在`TRAPFRAME`的下面（多个内核栈集中管理，相邻内核栈之间用Guard Page隔离开）。
- 真就和下面的图一模一样呗。
- 代码里定义的宏`KSTACK(p)`是用于计算内核栈顶地址的。p应该是第几个栈的意思，从0开始计数。

<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230630230845583.png" alt="image-20230630230845583" style="zoom:70%;" />

```c
// the kernel expects there to be RAM
// for use by the kernel and user pages
// from physical address 0x80000000 to PHYSTOP.
#define KERNBASE 0x80000000L
#define PHYSTOP (KERNBASE + 128*1024*1024)

// map the trampoline page to the highest address,
// in both user and kernel space.
#define TRAMPOLINE (MAXVA - PGSIZE)

// map kernel stacks beneath the trampoline,
// each surrounded by invalid guard pages.
#define KSTACK(p) (TRAMPOLINE - ((p)+1)* 2*PGSIZE)

// User memory layout.注意是User Memory Layout，不是Kernel
// Address zero first:
//   text
//   original data and bss
//   fixed-size stack
//   expandable heap
//   ...
//   TRAPFRAME (p->trapframe, used by the trampoline)
//   TRAMPOLINE (the same page as in the kernel)
#define TRAPFRAME (TRAMPOLINE - PGSIZE)
```

#### kalloc.c

这个文件应该实现的是物理页的管理器：

- 物理页的分配和回收、空闲物理页的记录。

首先是一切的基础：记录空闲物理页的Free List是怎么实现的

- `struct run`很奇特。就真的只有指向下一个`run`结构的结构体指针。

- 记录所有空闲物理页的`free list`实际上是`kmem`结构体，里面包含一个空闲页列表，和一个自旋锁。

```c
struct run {
  struct run *next;
};
struct {
  struct spinlock lock;
  struct run *freelist;
} kmem;
```

按照函数之间的相互调用顺序，按照如下的顺序来介绍每个函数：

- `kfree`回收被物理地址`pa`指向的页，这个页正常情况下应该是此前已经被`kalloc()`分配过的页（也就是说，不能回收未分配的页）。除了一种特殊情况：初始化内存分配器的时候，此时所有的页都没有分配，但是需要调用`kfree`把所有的页都插入Free List。
  - 做的事情也很简单，先检查地址是不是合法的，不合法就panic。
  - 地址合法的话，然后把要回收的页用1填满。
  - 然后拿锁，把回收回来的物理页的物理地址，头插（从头部插入）到freelist里面去，最后放锁。

```c
// Free the page of physical memory pointed at by v,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}

```

- `freerange`：`pa_start`是开始的物理地址，`pa_end`是结束的物理地址。`freerange`所做的事应该就是从开始到结束，回收**完全处于这个范围**的所有页。只有一部分与范围重叠的页，不回收，只有完全处于这个范围的页才回收。
  - 举个例子，开始是4095，结束是8193。那么只会回收起始物理地址为4096的那一页。4095所指向的起始地址为0的页、以及8193所指向的起始地址为8192的页，都不会回收，因为这两页都只与给定的范围部分重叠。
  - `freerange`本质上就是把遍历到的物理页，的物理地址，都（先填满1然后）头插入`Freelist`。因为`Kfree`就是做这个的。

```c
void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}
```

- `kinit()`实际上是从内核结束的地方到`PHYSTOP`范围内`freerange`了一遍。`end`就是内核结束的地方，或者说内核结束之后的第一个物理地址。本质上就是把所有物理页（此时全部都是未分配的），的地址，都插入了`freelist`。

```c
void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);
}
```



- 用于分配一页物理内存的`kalloc`就更简单了，拿锁，从`Freelist`移除头部的地址，放锁，返回刚才移除的地址。
  - 如果没有空闲块了，`freelist`本身就是`null`，这个时候`r`就是`null`，直接返回`null`。

```c
// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  release(&kmem.lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}

```

#### vm.c

同样还是按照函数调用的顺序来看。

- `kvminit()`的调用链：`kvminit()` ==> `kvmmap()`
- `kvmmap`的调用链：`kvmmap` ==> `mappages`
- `mappages`的调用链：`mappages` ==> `walk`
- `walk`应该就是调用链的终点了（如果不考虑调用其他源文件的函数的话）

所以从walk开始看：

- 在`RISC-V.h`中，将`uint64`定义为了页表指针和PTE指针，所以walk返回的是PTE的物理地址。

```c
//riscv.h
typedef uint64 pte_t;
typedef uint64 *pagetable_t; // 512 PTEs
```

- 给定一个页表`pagetable`和虚拟地址`va`，以及一个标志位`alloc`。`walk`要做的就是从`pagetable`中找到与`va`对应的PTE，并返回其物理地址。如果`allocate`不等于0，那么创建请求的页表页。
- 直接说循环里吧，循环就是依次获取2/1/0级PTE。
  - 先调用`PX(level, va)`，这个宏定义于`riscv.h`，用于根据请求的级数得到对应的9比特索引。
  - `pagetable[PX(level, va)]`其实就是把`pagetable`当做一个PTE数组了，然后根据上一行计算出的索引找到PTE，记这个PTE为pte。
  - `*pte & PTE_V`用于查看其Valid位，如果合法的话，进入IF分支，让`pagetable`指向下一级的页表，进入下一个循环。
    - IF分支里的`pagetable = (pagetable_t)PTE2PA(*pte);`，其实就是从pte里取出物理地址的部分，物理地址的部分就是下一级的页表的地址。所以这一句话就是让`pagetable`指向下一级页表
  - 如果不合法的话，进入else分支：
    - 如果`alloc==0`或者是`(pagetable = (pde_t*)kalloc()) == 0`（也即分配新的页失败），那么直接return 0.
    - 否则，`pagetable`会指向新分配的一页物理页，清零一下这一页。然后让`pte`指向新分配的页表页，并把`pte`的Valid位置为合法（ `*pte = PA2PTE(pagetable) | PTE_V;`）。

```c
// Return the address of the PTE in page table pagetable
// that corresponds to virtual address va.  If alloc!=0,
// create any required page-table pages.
//
// The risc-v Sv39 scheme has three levels of page-table
// pages. A page-table page contains 512 64-bit PTEs.
// A 64-bit virtual address is split into five fields:
//   39..63 -- must be zero.
//   30..38 -- 9 bits of level-2 index.
//   21..29 -- 9 bits of level-1 index.
//   12..20 -- 9 bits of level-0 index.
//    0..11 -- 12 bits of byte offset within the page.
pte_t *
walk(pagetable_t pagetable, uint64 va, int alloc)
{
  if(va >= MAXVA)
    panic("walk");

  for(int level = 2; level > 0; level--) {
    pte_t *pte = &pagetable[PX(level, va)];
    if(*pte & PTE_V) {
      pagetable = (pagetable_t)PTE2PA(*pte);
    } else {
      if(!alloc || (pagetable = (pde_t*)kalloc()) == 0)
        return 0;
      memset(pagetable, 0, PGSIZE);
      *pte = PA2PTE(pagetable) | PTE_V;
    }
  }
  return &pagetable[PX(0, va)];
}
```

然后看`mappages`：从虚拟地址`va`开始，大小为`size`的虚拟地址空间，这个范围内涉及到的所有的页，给每一页添加PTE。

- 从`va`到`va + size - 1`刚好长度是size（要把va本身算进去，就跟数组下标0到N-1刚好是N个值一样），所以`va + size`这里减了1。
- a和last分别向下取到4096的倍数，就是计算第一页和最后一页的地址。因为我们分配内存都是以页为最小单位分配的，所以要计算va处于哪一页，va + size - 1处于哪一页。
- 然后看for循环：a这个变量其实会依次指向每一虚拟页的开头，而pa则会指向对应的每一物理页的开头（也就是说，物理页也连续分配了？）。
  - 调用`walk`找到对应的PTE，用pte指向它。
  - 如果pte是合法的，那么panic。因为按道理来说他应该是非法的。
  - 然后给pte重新赋值，把pte的物理地址部分置为pa，把标志位V置为合法，等等。
  - a==last的时候跳出循环没什么好说的。
  - 然后移动a和pa，指向下一虚拟页和物理页。
- 我有一个问题，传入的参数`perm`是什么意思？GPT说它是权限位

```c
// Create PTEs for virtual addresses starting at va that refer to
// physical addresses starting at pa. va and size might not
// be page-aligned. Returns 0 on success, -1 if walk() couldn't
// allocate a needed page-table page.
int
mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm)
{
  uint64 a, last;
  pte_t *pte;

  a = PGROUNDDOWN(va);
  last = PGROUNDDOWN(va + size - 1);
  for(;;){
    if((pte = walk(pagetable, a, 1)) == 0)
      return -1;
    if(*pte & PTE_V)
      panic("remap");
    *pte = PA2PTE(pa) | perm | PTE_V;
    if(a == last)
      break;
    a += PGSIZE;
    pa += PGSIZE;
  }
  return 0;
}
```

然后是`kvmmap`：`kvmmap`用于给内核页表添加PTE。只会在开机的时候使用。

```c
// add a mapping to the kernel page table.
// only used when booting.
// does not flush TLB or enable paging.
void
kvmmap(uint64 va, uint64 pa, uint64 sz, int perm)
{
  if(mappages(kernel_pagetable, va, sz, pa, perm) != 0)
    panic("kvmmap");
}
```

最后是`kvminit`：它调用了`kvmmap`很多次。因为内核使用的是直接映射的虚拟地址，所以传递给`kvmmap`的虚拟地址和物理地址是一个值。

- `KERNBASE`是`0x800000L`，也就是内核代码开始的地方。 
- `TRAMPOLINE`是一个用宏定义的值，刚好就是虚拟地址空间最顶部。在`memlayout.h`中（`#define TRAMPOLINE (MAXVA - PGSIZE)`）
- 结合下面的Figure 3.3看更合适

```c
/*
 * create a direct-map page table for the kernel.
 */
void kvminit()
{
  kernel_pagetable = (pagetable_t) kalloc();
  memset(kernel_pagetable, 0, PGSIZE);
  // uart registers。将UART寄存器的地址映射到相同的虚拟地址，并设置可读可写权限。
  kvmmap(UART0, UART0, PGSIZE, PTE_R | PTE_W);
  // virtio mmio disk interface。将virtio MMIO磁盘接口的地址映射到相同的虚拟地址，并设置可读可写权限。
  kvmmap(VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);
  // CLINT。将CLINT（Core Local Interruptor）的地址映射到相同的虚拟地址，并设置可读可写权限。
  kvmmap(CLINT, CLINT, 0x10000, PTE_R | PTE_W);
  // PLIC。将PLIC（Platform-Level Interrupt Controller）的地址映射到相同的虚拟地址，并设置可读可写权限。
  kvmmap(PLIC, PLIC, 0x400000, PTE_R | PTE_W);
    
  // map kernel text executable and read-only。KERNBASE到etext之间的内核代码区域：将内核代码区域的地址映射到相同的虚拟地址，并设置可读可执行权限。
  kvmmap(KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);
  // map kernel data and the physical RAM we'll make use of。etext到PHYSTOP之间的内核数据和物理内存区域：将内核数据和物理内存区域的地址映射到相同的虚拟地址，并设置可读可写权限。
  kvmmap((uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);
    
  // map the trampoline for trap entry/exit to the highest virtual address in the kernel.
    //将trap入口/退出的trampoline映射到内核最高的虚拟地址，并设置可读可执行权限。
  kvmmap(TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
}
```



<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230630230845583.png" alt="image-20230630230845583" style="zoom:80%;" />



然后再看一下`kvminithart`：`main`调用它来安装内核页表

- `w_satp`定义在`riscv.h`中。用于写入satp寄存器。
- h/w page table register指的是硬件页表寄存器，Hardware Page Table Register，H/W PTBR，也就是satp寄存器。这句话的意思是将satp寄存器切换到内核页表
- `sfence_vma`指令用于刷新TLB的，也定义于`riscv.h`

```c
// Switch h/w page table register to the kernel's page table,
// and enable paging.
void
kvminithart()
{
  w_satp(MAKE_SATP(kernel_pagetable));
  sfence_vma();
}
```

 然后是sbrk的实现中会涉及到的几个函数：

`sbrk`由`growproc (kernel/proc.c:239)`实现，而`growproc`则调用了`uvmalloc`和`uvmdealloc`。`uvmdealloc`则调用了`uvmunmap`

阅读源代码的时候发现`uvmalloc`调用了`uvmdealloc`，所以顺序是：`uvmunmap` ==> `uvmdealloc` ==> `uvmalloc`



`uvmunmap`用于从给定页表pagetable中移除从虚拟地址`va`开始的n页映射PTE（一个PTE对应一页物理页和虚拟页之间的映射）

- 要求`va`是页对齐的，否则panic。
- do_free表示在移除PTE的时候是否要释放掉对应的物理页。
- 然后就是for循环，a从取值va开始遍历，依次指向下一**虚拟**页（a += PGSIZE就是指向下一页），直到移除了n个PTE。

但是我有个问题？为什么你都释放PTE了，还不释放对应的内存？问了GPT，我宣布GPT是我爹。很简单，因为这一页可能是共享的，**比如跳板页**就是共享的。

> 然而，有时候可能存在一些特殊情况，需要仅仅解除页面映射而不释放对应的物理内存。这种情况可能发生在以下情形下：
>
> 1. 页面被映射到了多个虚拟地址。当多个虚拟地址映射到同一个物理页面时，如果其中一个虚拟地址解除了映射并释放了物理内存，那么其他虚拟地址的映射将变得无效。因此，在这种情况下，可能只解除映射而不释放物理内存。
> 2. 页面属于共享内存区域。在某些共享内存的实现中，多个进程共享同一个物理页面。当一个进程解除了页面的映射时，其他进程仍然需要访问该页面。因此，只需要解除映射而不释放物理内存。

```c
// Remove npages of mappings starting from va. va must be
// page-aligned. The mappings must exist.
// Optionally free the physical memory.
void
uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free)
{
  uint64 a;
  pte_t *pte;

  if((va % PGSIZE) != 0)
    panic("uvmunmap: not aligned");

  for(a = va; a < va + npages*PGSIZE; a += PGSIZE){
    if((pte = walk(pagetable, a, 0)) == 0)//调用walk找到PTE，用pte去指向
      panic("uvmunmap: walk");
    if((*pte & PTE_V) == 0)//非法的话panic
      panic("uvmunmap: not mapped");
    if(PTE_FLAGS(*pte) == PTE_V)//叶子节点的标志位是不可能只有V位的，R-W-X位至少有一个是非0的，所以panic。
      panic("uvmunmap: not a leaf");
    if(do_free){
      uint64 pa = PTE2PA(*pte);//提取出PTE里的物理页地址部分
      kfree((void*)pa);
    }
    *pte = 0;//将pte清零。即解除虚拟地址与物理地址的映射
  }
}
```



`uvmdealloc`用于将内存大小**向下**调整为newsz。

- 如果新的内存size要大于old，说明不需要向下调整内存大小，直接返回。
- 否则计算出需要减少的页面数`npages`，以及起始虚拟地址`PGROUNDUP(newsz)`。传递1进去表明需要回收物理页。
  - 起始虚拟地址是PGROUNDUP(newsz)，也即对newsz向上对齐。因为向下对齐的话，会多去掉一页，剩下的保留下来的内存会小于newsz字节。
  - 如下图，假如向下对齐newsz，那么去掉的就是2,3,4页，剩下的0，1两页的内存空间是不够newze大小的。所以要向上对齐，保留2号页，这样虽然实际上可用内存比newsz略大半个页，但是可以接受。
  - <img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/ccb5a145c93878de4da66a86e8cddd6.jpg" alt="ccb5a145c93878de4da66a86e8cddd6" style="zoom:67%;" />

```c
// Deallocate user pages to bring the process size from oldsz to
// newsz.  oldsz and newsz need not be page-aligned, nor does newsz
// need to be less than oldsz.  oldsz can be larger than the actual
// process size.  Returns the new process size.
uint64
uvmdealloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz)
{
  if(newsz >= oldsz)
    return oldsz;

  if(PGROUNDUP(newsz) < PGROUNDUP(oldsz)){
    int npages = (PGROUNDUP(oldsz) - PGROUNDUP(newsz)) / PGSIZE;
    uvmunmap(pagetable, PGROUNDUP(newsz), npages, 1);
  }

  return newsz;
}

```

`uvmalloc`：向上增长物理内存到newsz。给页表添加对应的PTE，分配物理页

- 直接说循环：a依次指向虚拟地址从从`PGROUNDUP(oldsz)`开始的每一页虚拟。实际分配的内存可能会比newsz要大小半个页，因为分配是按页分配的。
- 我有个问题，为什么分配mem失败的时候，或者添加PTE失败的时候，要调用`uvmdealloc(pagetable, a, oldsz);`？
  - 需要注意到，这是一个循环，这就意味着新页面的分配是可能会进行多次的，所以分配的失败很有可能不是第一次分配就出现的。更大的可能性是，已经成功分配了几页，但是在分配下一页的时候分配失败了，但是你总不可能摆烂分配到一半就直接返回吧（指就分配这几页糊弄一下直接说分配成功），还是得将你之前分配好的页面也回收掉才行，回滚到完全没有进行新页面分配的状态，才是正确的操作。`uvmdealloc`的调用，概况来说就是，回收此前成功分配的所有页。
  - 举个例子，为了满足请求，需要分配10页，但是你循环到7次的时候分配失败了，那你可能要把这7页也回收回去再return，而不是直接带着分配了的7页return。假如你分配7页就告诉用户说我给你分配好了，用户就以为10页全部分配好了，往第9页写的时候不就越界了？所以不能这么做。

```c
// Allocate PTEs and physical memory to grow process from oldsz to
// newsz, which need not be page aligned.  Returns new size or 0 on error.
uint64
uvmalloc(pagetable_t pagetable, uint64 oldsz, uint64 newsz)
{
  char *mem;
  uint64 a;

  if(newsz < oldsz)
    return oldsz;

  oldsz = PGROUNDUP(oldsz);
  for(a = oldsz; a < newsz; a += PGSIZE){
    mem = kalloc();//分配一页物理页
    if(mem == 0){
      uvmdealloc(pagetable, a, oldsz);//分配失败的话，回收
      return 0;
    }
    memset(mem, 0, PGSIZE);//清零刚才分配的物理页
    if(mappages(pagetable, a, PGSIZE, (uint64)mem, PTE_W|PTE_X|PTE_R|PTE_U) != 0){
        //调用mappages来给虚拟页号a和物理页号mem添加对应的PTE映射。正常情况下mappages的返回值是0，不会进入IF。
      kfree(mem);
      uvmdealloc(pagetable, a, oldsz);//回收本次分配失败之前，成功分配的（可能不止一页）物理页
      return 0;
    }
  }
  return newsz;
}
```

最后再看一下`copyout`和`copyin`吧，前者被`exec`调用了。



`copyout`用于从内核空间将字节流复制到用户的虚拟地址空间。`copyin`则做相反的事，二者思想类似，只看前者吧

- `pagetable`：用户进程的页表
- `dstva`：起始目标虚拟地址
- `len`：要复制的长度。
- 因为字节流可能不止4KB，所以可能会写入不止一页虚拟页，这种情况下，对每一页虚拟页我们找到其对应物理页，然后往里面写东西。
- 基本思想是很简单的：
  - 先根据目标虚拟地址从页表查到PTE，然后找到与虚拟地址对应的物理页（`walkaddr`就在做这件事）
  - 然后往这一页物理页里写东西就可以了。
  - 然后又因为并不是每一页的写入都是从页头开始写入，有可能从物理页的中间开始写入，所以用`memmove`来精确复制制导。
    - `void *memmove(void *dest, const void *src, size_t n);`
  - 循环里就是在做这件事。

```c
// Copy from kernel to user.
// Copy len bytes from src to virtual address dstva in a given page table.
// Return 0 on success, -1 on error.
int
copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)
{
  uint64 n, va0, pa0;

  while(len > 0){
    va0 = PGROUNDDOWN(dstva);//找到下一个虚拟页号
    pa0 = walkaddr(pagetable, va0);//找到对应的物理页
    if(pa0 == 0)
      return -1;
    n = PGSIZE - (dstva - va0);
    if(n > len)
      n = len;
    memmove((void *)(pa0 + (dstva - va0)), src, n);

    len -= n;
    src += n;
    dstva = va0 + PGSIZE;
  }
  return 0;
}
// Look up a virtual address, return the physical address,
// or 0 if not mapped.
// Can only be used to look up user pages.
uint64
walkaddr(pagetable_t pagetable, uint64 va)
{
  pte_t *pte;
  uint64 pa;

  if(va >= MAXVA)
    return 0;

  pte = walk(pagetable, va, 0);
  if(pte == 0)
    return 0;
  if((*pte & PTE_V) == 0)
    return 0;
  if((*pte & PTE_U) == 0)
    return 0;
  pa = PTE2PA(*pte);
  return pa;
}
```

#### exec.c

`exec.c`的代码太长了，就不附上代码了，直接写一下大概的步骤吧：

1. 调用`namei`找到二进制文件的iNode，记错ip。
2. 调用`readi`读取ip，检查ELF头部。
3. 调用`proc_pagetable`分配新的页表，记作PG。
4. 调用`uvmalloc`给二进制文件的每个段分配内存，调用`loadseg`把每一段载入内存。

其中涉及到的东西太多了，不是目前这个进度能够全部看懂的。



`loadseg`可以看看：

### lecture

#### 提问汇总

重点是写入SATP寄存器的指令是特权指令，这是一个进程不能访问另外一个进程的页表的根本原因。

> 学生提问：刚刚说到SATP寄存器会根据进程而修改，我猜每个进程对应的SATP值是由内核保存的？
>
> Frans教授：是的。内核会写SATP寄存器，写SATP寄存器是一条特殊权限指令。所以，用户应用程序不能通过更新这个寄存器来更换一个地址对应表单，否则的话就会破坏隔离性。所以，只有运行在kernel mode的代码可以更新这个寄存器。

物理地址的长度和虚拟地址的长度一定要一样吗？

> 不是，Xv6里虚拟地址实际上是27 +12比特（25比特保留位），而物理地址是44+12比特（44比特物理页号）。二者之间的长度没有必然关系



> 同一个学生：图中的56bit又是根据什么确定的？
>
> Frans教授：这是由硬件设计人员决定的。所以RISC-V的设计人员认为56bit的物理内存地址是个不错的选择。可以假定，他们是通过技术发展的趋势得到这里的数字。比如说，设计是为了满足5年的需求，可以预测物理内存在5年内不可能超过2\^56这么大。或许，他们预测是的一个小得多的数字，但是为了防止预测错误，他们选择了像2^56这么大的数字。这里说的通吗？很多同学都问了这个问题。
>
> 学生提问：如果虚拟内存最多是2\^27（最多应该是2\^39），而物理内存最多是2^56，这样我们可以有多个进程都用光了他们的虚拟内存，但是物理内存还有剩余，对吗？
>
> Frans教授：是的，完全正确。
>
> 学生提问：因为这是一个64bit的机器，为什么硬件设计人员本可以用64bit但是却用了56bit？
>
> Frans教授：选择56bit而不是64bit是因为在主板上只需要56根线。

> 学生提问：当一个进程请求一个虚拟内存地址时，CPU会查看SATP寄存器得到对应的最高一级page table，这级page table会使用虚拟内存地址中27bit index的最高9bit来完成索引，如果索引的结果为空，MMU会自动创建一个page table吗？
>
> Frans教授：不会的，MMU会告诉操作系统或者处理器，抱歉我不能翻译这个地址，最终这会变成一个page fault。如果一个地址不能被翻译，那就不翻译。就像你在运算时除以0一样，处理器会拒绝那样做。
>
> 学生提问：我想知道我们是怎么计算page table的物理地址，是不是这样，我们从最高级的page table得到44bit的PPN，然后再加上虚拟地址中的12bit offset，就得到了完整的56bit page table物理地址？
>
> Frans教授：我们不会加上虚拟地址中的offset，这里只是使用了12bit的0。所以我们用44bit的PPN，再加上12bit的0，这样就得到了下一级page directory的56bit物理地址。这里要求每个page directory都与物理page对齐（也就是page directory的起始地址就是某个page的起始地址，所以低12bit都为0）

关于TLB：TLB会保存虚拟地址到物理地址的映射关系

> 学生提问：3级的page table是由操作系统实现的还是由硬件自己实现的？
>
> Frans教授：这是由硬件实现的，所以3级 page table的查找都发生在硬件中。MMU是硬件的一部分而不是操作系统的一部分。在XV6中，有一个函数也实现了page table的查找，因为时不时的XV6也需要完成硬件的工作，所以XV6有这个叫做walk的函数，它在软件中实现了MMU硬件相同的功能。

既然硬件已经实现了页表的查找，为什么还需要一个walk函数？简而言之就是，因为内核自己有时候也需要将一个虚拟地址转换为物理地址，硬件实现的页表查找是给CPU用的，内核需要自己想办法，所以有了walk函数。

> 学生提问：之前提到，硬件会完成3级 page table的查找，那为什么我们要在XV6中有一个walk函数来完成同样的工作？
>
> Frans教授：非常好的问题。这里有几个原因，首先XV6中的walk函数设置了最初的page table，它需要对3级page table进行编程所以它首先需要能模拟3级page table。另一个原因或许你们已经在syscall实验中遇到了，在XV6中，内核有它自己的page table，用户进程也有自己的page table，用户进程指向sys_info结构体的指针存在于用户空间的page table，但是内核需要将这个指针翻译成一个自己可以读写的物理地址。如果你查看copy_in，copy_out，你可以发现内核会通过用户进程的page table，将用户的虚拟地址翻译得到物理地址，这样内核可以读写相应的物理内存地址。这就是为什么在XV6中需要有walk函数的一些原因。

#### 其他

虚拟地址空间的最大值和物理地址空间的最大值一定要相同吗？

- 不，二者之间没有任何关系，我们甚至可以让虚拟地址空间的最大值大于我们实际拥有的物理内存。
- 实际上在xv6，虽然RISC-V是64比特的寄存器，但是虚拟地址保留了最高的25位没用，虚拟地址只有39位（27虚拟页号 + 12偏移量）。而物理地址则是56位（44页号+ 12偏移量），物理地址的最大值显著大于虚拟地址（这个56是怎么来的？老师的回答是，设计者选的，根据科技的发展趋势预判单机需求的内存不超过2^56字节）。
- 当然，甚至我们实际上会用到的物理地址的范围会远远小于最大值，这取决于主板上最多可以支持多少内存条（和内存容量）。

MMU本身并不存储页表，页表是存储在内存里的，CPU有存储页表物理地址的页表基地址寄存器，CPU借助这个寄存器来告知MMU去哪里找到页表。



页表所维持的映射并不是以单个的虚拟地址映射到单个的物理地址，因为这样光是存储映射本身就可以耗尽全部的内存空间（映射的项数太多了），而是以页为单位映射，单个的虚拟页映射到单个的物理页。也即，映射的粒度是Page，不是Address



物理页号本身，如果再在低12位的偏移量补上0，实际上就是这一个物理页的起始地址。

在进行页表的三级索引的时候，从L0级获取的物理页号，低位补上0，就是L1级页表的物理地址。从第二级获取第三级的物理地址也是一样的，低12位补0。**而且每一个页表就占一页**。



关于地址低于`0x8000000`的那些I/O设备都是什么：

- PLIC是中断控制器（Platform-Level Interrupt Controller）我们下周的课会讲。
- CLINT（Core Local Interruptor）也是中断的一部分。所以多个设备都能产生中断，需要中断控制器来将这些中断路由到合适的处理函数。
- UART0（Universal Asynchronous Receiver/Transmitter）负责与Console和显示器交互。
- VIRTIO disk，与磁盘进行交互。

当与低于`0x8000000`的地址交互时，可以认为是直接在读写设备

## Lecture 5

这一节没有需要看的源代码和xv6 book，但是讲述的是Calling Convention。

什么是Calling Convention？

> Calling Convention（调用约定）是一组规则和约定，用于规定在函数调用过程中，参数的传递方式、寄存器的使用、堆栈的分配等细节。调用约定定义了函数调用方和被调用方之间的接口，确保它们可以正确地进行通信和交互。
>
> 在编程语言中，函数调用涉及到传递参数、执行函数体、返回结果等步骤。不同的编程语言和硬件体系结构可能有不同的调用约定。调用约定规定了参数在哪里传递（寄存器、堆栈）、哪些寄存器需要保存和恢复、如何传递函数的返回值等。
>
> 调用约定的设计旨在最大程度地提高程序的执行效率和可维护性。通过定义统一的规则，编译器可以在生成目标代码时进行优化，而开发人员则可以依赖这些约定编写可移植的代码。
>
> 在RV32和RV64程序中，RISC-V指令集架构定义了两种调用约定：RV32G/RV64G 和 RV32I/RV64I。RV32G/RV64G是基本指令集加上标准通用扩展的调用约定，包括浮点运算指令扩展。而RV32I/RV64I是针对没有浮点运算单元的实现（如软浮点）的调用约定。
>
> 通过使用特定的调用约定，开发人员可以编写与指定的指令集架构和硬件实现兼容的代码，并且可以利用编译器提供的优化功能来提高代码的性能。



阅读材料则描述了用于RV32和RV64程序的C编译器标准，以及两种调用约定：基本指令集加标准通用扩展的约定（RV32G/RV64G），以及针对没有浮点运算单元的实现（如RV32I/RV64I）的软浮点（soft-float）约定。

材料总共就只有四页

### 阅读材料

#### 18.1 C Datatypes and Alignment

有关C的数据类型和对齐：



在RV32和RV64中，`int`都是32比特，而`long`和指针类型都和与一个整数寄存器一样长（也即RV32下32位，RV64下64位）

在RV32和RV64中，`long long`和`double`都是64比特，而`float`是32比特，`long double`则是128比特的长浮点数。



`char`和`unsigned char`都是8比特无符号整数，`unsigned short`是16比特无符号整数。当存储在整数寄存器时（寄存器长度至少32位，所以需要扩展），使用0扩展。

`signed char`是8比特有符号整数，`short`是16比特有符号整数。当存储在整数寄存器时，使用符号扩展。

在RV64下，诸如`int`这样的32比特类型，存储在寄存器里的时候，都使用适当的符号扩展，不改变其值（对于无符号类型来说，也一样）。



在内存中，他们都是对齐的。

<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230707164241710.png" alt="image-20230707164241710" style="zoom:67%;" />

#### 18.2 RVG Calling Convention

RISC-V的调用约定尽可能地使用寄存器来传递参数，总共有8个整数寄存器a0-a7和8个浮点数寄存器fa0-fa7。

如果我们把传递给一个函数的全部参数组成一个结构体来看，那么8个寄存器就是传递这个结构体的前8个域。

第i个（i < 8）浮点数参数放在`fai`，反之第i个整数参数放在`ai`来传递。

但是，当浮点数是联合（Union）的一部分，或者是一个结构体的数组域的一部分，使用整数寄存器来传递（为什么？）

此外，作为可变参数函数的浮点参数（除了在参数列表中明确命名的参数）将在整数寄存器中传递（为什么？）。

GPT说是为了简化实现和保证内存对齐？不知道上课会不会讲？



关于下面的这段话：

因为寄存器的长度都是和指针的长度一致的，也即指针为64位的时候，寄存器也为64位。所以长度小于指针长度的参数，在使用寄存器传递的时候，都保存在寄存器的权重较低的位置。举个例子，RV64，要传递一个`int`的时候，这个`int`会被放置在整数寄存器的低32位存储。

同样的，当长度小于指针长度的参数是使用栈来传递的时候，参数会出现在指针字权重更低的部分，小端存储的。举个例子，RV64的情况下，一个`int`使用栈来传递，它会放在这64位中更低的32位部分上。

> Arguments smaller than a pointer-word are passed in the least-significant bits of argument registers. Correspondingly, sub-pointer-word arguments passed on the stack appear in the lower addresses of a pointer-word, since RISC-V has a little-endian memory system.





当传递长度是指针字长两倍的原始类型参数时（比如32位指针下的`long`，和64位指针字长下的`long double`），要求使用一对偶数-奇数寄存器对（前一个寄存器是偶数编号），其中偶数号的寄存器保存权重较低的那一半。举个例子，RV32下，给函数`void foo(int, long long)`传递参数，第一个`int`被放在整数寄存器`a0`来传递，而第二个`long long`则用寄存器对`a2-a3`来保存，其中a2保存`long long`的较低32位。`a1`未被使用



长度长于两倍指针字长的参数，按引用传递（pass by  Reference）。而不大于两倍指针字长的参数，都是按值传递。



对于哪些没用寄存器传递的参数（比如参数个数太多了，寄存器不够用），剩余的参数就是用栈来传递，栈顶指针`sp`则指向第一个未使用寄存器传递的参数。

原文要结合上文的“概念上的结构体”来理解。如果我们把全部的参数组成一个结构体的话，寄存器负责传递结构体的一部分域，剩余部分使用栈来传递。

> The portion of the conceptual struct that is not passed in argument registers is passed on the stack. The stack pointer sp points to the first argument not passed in a register



关于返回值的存储方式：

- 返回值会被存储在整数寄存器a0和a1，以及浮点寄存器fa0和fa1中。
- 如果返回值的长度是两倍指针字长的（比如RV32下的`long`型），同时使用两个寄存器来存储返回值。
- 浮点类型的返回值特殊一点，只有返回值是原始类型或者是只包含一个或两个浮点值的结构体的成员，才会使用浮点寄存器返回。
- 对于长度超过两倍指针字长的返回值，完全使用内存传递，调用者先分配一块内存，然后把内存的指针传递给被调函数，被调函数将返回值写入这块内存。



在标准的RISC-V调用约定下，栈是向下增长的，栈顶指针永远是16字节对齐的（怎么个对齐法？每次grow 16B？）



除了传递参数和返回值所用的寄存器以外，还有7个整数寄存器`t0-t6`和12个浮点寄存器`ft0-ft11`，这些临时寄存器被归类为易失性寄存器（volatile across calls），所谓易失性就是指在函数调用之间，这些寄存器的内容是不稳定的，会发生变化的。如果调用者之后会使用到这些易失性寄存器的数据，那么就必须自己保存。

还有12个整数寄存器`s0-s11`和12个浮点寄存器`fs0-fs11`，这些寄存器被归类为保留寄存器，所谓保留寄存器（ preserved across calls），就是这些寄存器的内容在一个函数调用开始之前，和这个函数调用结束之后，它的值不能改变。所以如果被调函数使用了这些寄存器，它就必须先保存寄存器的值，并在最后恢复。

> 来自GPT：
>
> 在RISC-V架构中，寄存器分为易失性寄存器和保留寄存器。易失性寄存器是临时寄存器，跨函数调用时其内容可以被改变，因此如果在后续使用中需要保留其内容，调用者需要显式地保存这些寄存器。
>
> 与易失性寄存器相反，保留寄存器是跨函数调用时保持不变的寄存器。整数寄存器s0-s11和浮点寄存器fs0-fs11被归类为保留寄存器，它们的内容在函数调用之间保持不变。如果被调用函数在使用这些寄存器时，必须负责保存和恢复这些寄存器的值，以确保不影响调用者的寄存器内容。

> 原文：
>
> In addition to the argument and return value registers, seven integer registers t0–t6 and twelve floating-point registers ft0–ft11 are temporary registers that are volatile across calls and must be saved by the caller if later used. Twelve integer registers s0–s11 and twelve floating-point registers fs0–fs11 are preserved across calls and must be saved by the callee if used.

总结一下寄存器：

- 传参：a0-a7 fa0 -fa7
- 传返回值：a0-a1 fa0-fa1
- 易失性寄存器：t0-t6 ft0-ft6
- 保留寄存器：s0-s11 fs0-fs11

<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230707203446133.png" alt="image-20230707203446133" style="zoom:67%;" />

#### 18.3 Soft-Float Calling Convention

什么是软浮点：

软浮点顾名思义，就是用软件实现的浮点。一般的PC上都有专门的硬件来实现浮点计算，但是在某些资源受限的硬件上可能没有专门用于浮点运算的硬件，所以用软件来模拟浮点运算。软浮点通过使用整数运算和位操作来执行浮点数的表示、运算和转换。它使用整数寄存器来存储和操作浮点数的各个部分，如符号位、指数和尾数。通过一系列的算法和函数库，软浮点可以对浮点数进行加减乘除、比较和转换等操作。



软浮点调用约定用于缺少浮点计算硬件的RV32和RV64，不使用`f`系列寄存器（也就是浮点寄存器，因为都是用整数运算模拟的）



在软浮点的调用约定中，整数参数的参数传递和返回值传递和之前一节提到的一样，栈的规则也一样。**浮点参数则通过整数寄存器进行传递和返回，使用与相同长度的整数参数相同的规则。**

举例：函数 `double foo(int, double, long double)`

- RV32：第一个参数 `a0` 传递，第二个参数 `a2` 和 `a3` 传递，而第三个参数 `a4` 以引用（by reference）的方式传递。返回值存储在寄存器 `a0` 和 `a1` 中。
- RV64：第一个参数 `a0` 传递，第二个参数 `a1` 传递，而第三个参数使用 `a2` 和 `a3` 寄存器对传递。返回值则存储在寄存器 `a0` 中。

### Lecture

课程目标：RISC-V汇编和处理器、RISC-V的调用约定、栈。

#### 提问

> 学生提问：返回值可以（只单独）放在a1寄存器（不放在a0寄存器）吗？
>
> TA：这是个好问题。我认为理论上是可以的，如果一个函数的返回值是long long型，也就是128bit，我们可以把它放到一对寄存器中。这也同样适用于函数的参数。所以，如果返回值超过了一个寄存器的长度，也就是64bit，我们可以将返回值保存在a0和a1。但是如果你只将返回值放在a1寄存器，我认为会出错。
>
> 学生提问：为什么寄存器不是连续的？比如为什么s1与其他的s寄存器是分开的？
>
> TA：我之前提到过，但是也只是我的猜想，我并不十分确定。因为s1寄存器在RISC-V的Compressed Instruction是可用的，所以它才被分开。
>
> 学生提问：除了Stack Pointer和Frame Pointer，我不认为我们需要更多的Callee Saved寄存器。
>
> TA：s0 - s11都是Callee寄存器，我认为它们是提供给编译器而不是程序员使用。在一些特定的场景下，你会想要确保一些数据在函数调用之后仍然能够保存，这个时候编译器可以选择使用s寄存器。

### 其他内容

这一节的重点个人认为是Stack Frame。

如下图，每个区域是一个栈帧，每执行一次函数调用就会产生一个Stack Frame。函数通过移动Stack Pointer（x86下是%sp寄存器）来完成Stack Frame的空间分配。

栈是向下增长的，创建一个新的Stack Frame的时候，总是对当前的Stack Pointer做减法。

一个函数的Stack Frame包含了保存的寄存器，本地变量，并且，**如果函数的参数多于8个，额外的参数会出现在Stack中**。所以Stack Frame大小并不总是一样。

但是，对于任意一个栈帧，总有两件事是一致的：

- Return address总是会出现在Stack Frame的第一位。
- 指向前一个Stack Frame的指针也会出现在栈中的固定位置（就恰好排在返回地址的后面）

关于栈有两个重要的寄存器：SP和FP，SP指向当前栈顶的位置，FP指向当前所使用的栈帧的Return Address处（指向当前Stack Frame的顶部）。

因为Return Address和前一个栈帧的指针都在栈帧顶的固定位置，所以通过FP寄存器可以直接找到这两个东西。



为什么需要在栈帧里保存前一个栈帧的地址？很简单，因为被调函数返回调用者的时候，栈帧也要切换为调用者的栈帧，而调用者的栈帧就是被调函数的栈帧的前一个栈帧。切换的方式就是将当前FP指向的栈帧的Prev Frame写入FP，同时SP指针也要往上移动一个栈帧。

<img src="https://github.com/huihongxiao/MIT6.S081/raw/master/.gitbook/assets/image%20%28263%29.png" alt="img" style="zoom:80%;" />



关于汇编中，函数的前奏和尾声：

在汇编代码中，函数的最开始你们可以看到Function prologue，之后是函数的本体，最后是Epollgue。这就是一个汇编函数通常的样子。

> 来自GPT：
>
> 
>
> 在汇编语言中，函数前奏（Function Prologue）和函数尾声（Function Epilogue）是在函数调用过程中用于准备和清理函数环境的指令序列。
>
> 函数前奏（Function Prologue）是在调用函数之前执行的指令序列。它的目的是为函数创建一个合适的执行环境，包括保存寄存器状态、分配栈空间和处理参数传递。常见的函数前奏指令包括：
>
> 1. 保存寄存器：将调用者保存的寄存器值压入栈中，以防止函数修改这些寄存器的值。
> 2. 分配栈空间：为函数分配局部变量和临时变量所需的栈空间。
> 3. 设置栈指针：将栈指针调整到正确的位置，以便可以正确访问局部变量和函数参数。
>
> 函数尾声（Function Epilogue）是在函数执行结束后执行的指令序列。它的目的是清理函数的执行环境，并将控制权返回给调用函数。常见的函数尾声指令包括：
>
> 1. 恢复寄存器：从栈中弹出保存的寄存器值，恢复调用函数之前的状态。
> 2. 释放栈空间：释放函数执行期间分配的栈空间。
> 3. 返回指令：使用适当的指令将控制权返回给调用函数，通常是通过返回地址。

## Lecture 6

### 阅读Chapter 4，除了4.6 Page Fault

总共有三种事件会使得CPU搁置当前正在执行的指令，并且强制将控制权转移到处理这种事件的特殊代码：

- 系统调用，当用户程序执行ECALL指令请求OS替它做点什么的时候。
- 异常，当指令（用户程序或者内核）做了某些非法的事情，比如除以0或者使用非法的虚拟地址。
- 设备中断，比如来自硬盘的中断表明其完成了读或者写。

Xv6 Book里面统一称呼为Trap。

一个Trap应该是透明的，在Trap出现时，无论执行的代码是什么，都应该在之后重新开始，并且代码应该完全不知道发生了什么。

典型的Trap处理流程是这样的：

1. Trap迫使控制转移到内核，内核保存寄存器以及其他必要的状态（这样被打断的工作才能恢复执行）。
2. 内核执行合适的Trap处理器代码（比如一个系统调用的实现代码，或者设备驱动的代码）
3. 内核恢复刚才保存的状态，从Trap返回
4. 刚才被打断的代码从它被打断的地方恢复执行

Xv6处理一个Trap分为四个阶段：

1. 由RISC-V CPU执行的硬件行为（比如保存当前CPU状态）

2. 为内核C代码“铺路”的汇编向量（GPT说是指设置好内核代码执行所必需的环境）。

   > 原文是：an assembly “vector” that prepares the way for kernel C code

3. 决定对这个Trap做什么的C Trap Handler代码（ a C trap handler that decides what to do with the trap）
4. 系统调用或者是设备驱动服务程序

内核对于三种陷阱分开做处理，各自都有单独的向量和陷阱处理器。这三种陷阱分别是：

- 来自用户空间的Trap
- 来自内核空间的Trap
- 时钟中断（timer interrupts）

#### 4.1 RISC-V trap machinery

这一小节讲的应该是RISC-V在硬件上的机制？

每个RISC-V的CPU都有一系列的控制寄存器，内核能够写入这些寄存器以告诉CPU如何处理陷阱，或者内核可以读取这些寄存器以获取有关已出现的陷阱的信息。

`riscv.h (kernel/riscv.h:1) `中包含了Xv6用到的寄存器的定义。最重要的的几个寄存器如下：

- `stvec(Supervisor Trap Vector Register)`：内核将Trap Handler的地址写入到这里，RISC-V CPU跳转到此处来处理陷阱
- `sepc(Supervisor Exception Program Counter)`：Trap出现时，内核将当前PC的值保存在这里（因为等下PC的值就被用`stvec`覆盖了）。`sret`指令（从Trap返回的指令）会将`sepc`的值复制给PC。内核可以通过写入`sepc`的方式控制`sret`返回到哪里去。
- `scause(Supervisor Cause Register)`：用于存储最近一次异常或陷阱事件的原因代码。RISC-V放一个数字在这里，是Trap出现的原因代码。
- `sscratch`这个寄存器很奇怪，描述也很奇怪。先把原文放上来，什么是Comes In handy？我还是不知道这个寄存器的作用是什么，GPT说是一个用于临时存储和交换数据的寄存器。

> scause: The RISC-V puts a number here that describes the reason for the trap.
>
> sscratch: The kernel places a value here that comes in handy at the very start of a trap handler.

- `sstatus`：这个寄存器的SIE比特控制是否启用设备中断（即是否关了中断的标志位），如果SIE比特被清除，RISC-V会延迟设备中断直到这个SIE被重新置位。这个寄存器的`SPP`比特记录了Trap是来自用户模式还是内核模式，也就控制了`sret`返回哪个模式。

**这里说的是延迟(defer)对中断的处理，也就是关中断的时候出现的中断不会丢失只会被延迟处理？**

每个CPU核心都有都有它们自己的这一套寄存器。这些寄存器都只能在内核模式下使用。



当陷阱出现的时候，RISC-V硬件对所有的陷阱类型都做了如下的工作（除了时钟中断）：

1. 如果陷阱是一个设备中断，并且`sstatus`的SIE比特被清除了（即设备中断关了），下面全部都不做
2. 清除SIE比特以关闭中断
3. 将PC的值复制到`sepc`
4. 将当前的模式（是User Mode还是Kernel Mode）保存到`sstatus`的SPP比特
5. 设置`scause`以反映陷阱出现的原因
6. 将模式设置为内核模式
7. `stvec`复制到PC
8. 从新的PC开始执行

需要注意的是硬件**并没有**切换到内核页表、保存除了PC以外的其他寄存器的状态、也没有切换到内核栈。这些工作必须由内核来执行。



CPU在一个陷阱期间只做最少的工作的一个理由是，这样可以给内核（软件）更大的灵活性。

#### 4.2 Traps from user space

在用户空间可能出现的陷阱：系统调用、异常、设备中断

从高层来看，来自用户空间的陷阱的处理路径大概是：`uservec (kernel/trampoline.S:16)` ==> `usertrap (kernel/trap.c:37)` 

返回的时候是：`usertrapret (kernel/trap.c:90)` ==> ` userret (kernel/trampoline.S:16)`



然后接下来这段话我没搞懂为什么因果关系成立：

1. 硬件没切换页表，和用户页表必须存在一个`uservec`的映射之间的因果关系
2. 为什么为了指令在切换页表之后可以继续执行，就必须把`userver`在两个页表中映射到相同的位置？

> Because the RISC-V hardware doesn’t switch page tables during a trap, the user page table must include a mapping for *uservec*, the trap vector instructions that `stvec` points to. *uservec* must switch `satp` to point to the kernel page table; in order to continue executing instructions after the switch, *uservec* must be mapped at the same address in the kernel page table as in the user page table.

如果从上一小节，硬件在陷阱出现的时候会做的几件事来看，或许可以解释这两个问题：6和7分别是切换为内核模式和将PC设置为`stvec`，这说明在6之前应该就已经在用户模式下设置好了`stvec`。而`stvec`所指向的正是`uservec`，所以要求在用户模式下也可以访问其地址，才能在用户模式下设置`stvec`寄存器，所以要求用户页表中有指向它的映射。而又因为在`stvec`寄存器中设置的毫无疑问是虚拟地址（分页已经启用了，用户模式下只存在虚拟地址），但是最终我们是在内核模式下执行`uservec`的（在用户模式下设置好的`stvec`，在内核模式下是直接用的），所以要求`uservec`在内核模式下也被映射成相同的虚拟地址，才能保证与用户模式下设置的`stvec`的值一致。



Xv6使用一个包含`uservec`的跳板页（*trampoline* page）来满足这个要求，Xv6将这个跳板页映射到内核页表和每一个用户页表的相同虚拟地址（这个地址是用宏定义的`TRAMPOLINE`），跳板页的内容见`trampoline.S`。（在执行用户代码的时候），`stvec`被设置成`uservec`。这里应该是通过跳板页里的汇编代码来设置的，不是用户进程本身设置的？



当`uservec`启动的时候，全部的32个寄存器中包含的值都是被中断的代码的，但是`uservec`可能需要修改这些寄存器以切换页表和生成保存寄存器值的地址，所以`uservec`需要先想办法腾出一个寄存器来使用。这个时候`sscratch`寄存器就派上用场了，内核预先在`sscratch`放置一个值，然后交换`a0`和`sscratch`的值（`csrrw`指令），这样`a0`原有的值就被保存到了`sscratch`，而`uservec`也就腾出了一个寄存器可以让它随便用。

> Q：为什么不直接就用sscratch寄存器，非得把a0腾出来？
>
> 保持a0不动并直接使用sscratch寄存器来执行操作可能会引入一些问题：
>
> 1. 寄存器约定：RISC-V架构定义了对寄存器的使用约定，如a0寄存器常用于保存函数的参数值。改变这种约定可能会导致代码的可移植性和可读性下降，增加代码维护的困难度。
> 2. 上下文保存和恢复：在中断处理程序（如uservec）中，保留和恢复被中断代码的上下文是非常重要的。通过交换a0和sscratch寄存器的值，可以在保存了用户代码的关键值的同时，提供一个可用的寄存器供中断处理程序使用。这样，在处理完中断后，可以正确地恢复用户代码的上下文。
> 3. 线程隔离：sscratch寄存器通常用于线程间隔离和状态保存。在多线程环境下，每个线程可能需要使用sscratch寄存器来保存自己的状态信息，以确保线程切换时不会相互干扰。直接在中断处理程序中使用sscratch寄存器可能会破坏这种线程隔离的机制，导致错误或不可预测的行为。

在腾出`a0`之后`uservec`就可以去保存这32个寄存器的值了。内核预先放置在`sscratch`寄存器的值实际上就是一个指向`trapframe`的指针。Xv6在创建每个进程的时候，就会为每个进程分配一页作为陷阱帧，并且将它映射到虚拟地址空间的顶部，就在跳板页的正下方。在交换寄存器之后，`a0`里保存的值就是陷阱帧的地址，所以`uservec`可以将32个寄存器的值都存入陷阱帧（a0原有的值则从`sscratch`读取）

`trapframe`包含当前进程的内核栈的指针、当前CPU的hartid、`usertrap`的地址和内核页表的地址。`uservec`检索这些值，将`satp`切换到内核页表，并调用`usertrap`。

> Q：那么Trapframe是即用即分配的吗，还是固定分配一个物理页给所有的进程使用？
>
> GPT：
>
> Trapframe 的具体实现方式可以因操作系统和架构而异，因此可以是即用即分配的，也可以是固定分配一个物理页给所有进程使用。
>
> 1. 即用即分配：在即用即分配的方式下，每当发生异常或中断时，操作系统会动态地分配一个 Trapframe 数据结构，并将相关的处理器状态保存在其中。这样可以节省内存空间，只在需要时分配 Trapframe，并在处理完异常后释放该 Trapframe。这种方式在多任务操作系统中较为常见，每个任务或进程都有自己的 Trapframe。
> 2. 固定分配：在固定分配的方式下，操作系统为每个进程预先分配一个 Trapframe，并将其映射到进程的地址空间中的固定位置。当发生异常或中断时，处理器直接将相关状态保存到该固定位置的 Trapframe 中。这样可以提高处理器状态保存和恢复的效率，但会消耗更多的内存空间。这种方式在一些实时操作系统或嵌入式系统中比较常见。
>
> 无论是即用即分配还是固定分配，Trapframe 都是用于保存处理器状态的数据结构，并在异常或中断处理时起到关键作用。具体的实现方式可以根据操作系统设计和应用需求来选择。

接下来就到了`usertrap`的工作：确定陷阱发生的原因，处理陷阱，返回。

1. 先将`stvec`设置为`kernelvec`，这样当处于内核期间再次出现陷阱时，陷阱可以被内核处理，而不是被`usertrap`处理
2. 再次保存sepc（这次存到哪？），because there might be a process switch in usertrap that could cause sepc to be overwritten.（什么时候会出现？）
3. 如果陷阱是系统调用，`syscall`处理，如果是设备中断，`devintr`处理，否则是异常，杀掉进程。

如果是系统调用，那么要给已保存的PC的值加上4，这样PC在恢复值的时候才会指向ECALL指令的下一条指令。

`usertrap`退出的时候会检查进程是否已经被杀，或者是否应该让出CPU（如果刚才出现的陷阱是时钟中断）



返回用户空间的第一步：调用`usertrapret`。它会设置好所有的控制寄存器，为下一次来自用户空间的陷阱做准备。包括：

- 设置`stvec`指向`uservec`，准备好`uservec`依赖的陷阱帧的域。
- 设置`sepc`为之前保存的值
- 最后调用跳板页的`userret`

`usertrapret`调用`userret`的时候在a0传递了用户页表的指针，在a1传递了陷阱帧的指针（书上这么说的，但是实际上和代码反了，所以我的问题解决了）。`userret`切换页表到用户页表，把**陷阱帧里保存的用户的**`a0`复制到`sscratch`，恢复32个寄存器的值，保存陷阱帧供下次陷阱使用，然后执行`sret`返回用户空间。

但是我有一个问题：为什么先把用户a0存到sscratch，然后从陷阱帧恢复所有的用户寄存器的值，然后再交换a0和sscratch，把TRAPFRAME存到sscratch，但是TRAPFRAME不是放在了a1吗？还是需要看一下代码才知道

> userret copies the trapframe’s saved user a0 to sscratch in preparation for a later swap with TRAPFRAME. From this point on, the only data userret can use is the register contents and the content of the trapframe. Next userret restores saved user registers from the trapframe, does
>
> a final swap of a0 and sscratch to restore the user a0 and save TRAPFRAME for the next trap, and uses sret to return to user space.

看完代码之后，应该是xv6 Book的原文有点小问题：

> usertrapret’s call to userret passes a pointer to the process’s user page table in a0 and TRAPFRAME in a1 (kernel/trampoline.S:88). 

这是代码：注释里明确说明了a0是Trapframe，a1是用户的页表。但是书上说反了。

```assembly
userret:
        # userret(TRAPFRAME, pagetable)
        # switch from kernel to user.
        # usertrapret() calls here.
        # a0: TRAPFRAME, in user page table.
        # a1: user page table, for satp.

        # switch to the user page table.
        csrw satp, a1
        sfence.vma zero, zero

        # put the saved user a0 in sscratch, so we
        # can swap it with our a0 (TRAPFRAME) in the last step.
        ld t0, 112(a0)
        csrw sscratch, t0
```



#### 4.3 Code: Calling system calls

这一小节讲述调用系统调用的时候发生了什么。以调用exec系统调用为例：

用户的代码将给exec的参数放在寄存器a0和a1，将系统调用号放在a7。系统调用号实际上是数组`syscalls`的索引，这个数组是一个函数指针数组。

ECALL指令陷入内核，依次执行`uservec`、`usertrap`，然后执行`syscall`。

`syscall`从**陷阱帧中读取**已保存的a7寄存器的值，也即系统调用号，然后使用系统调用号去索引相应的系统调用。这会导致调用exec的具体实现函数。

当具体实现系统调用的函数返回，`syscall`将系统调用的返回值记录在`p->trapframe->a0`(**不是直接放在a0寄存器**)，这样原始用户空间对exec的调用会返回a0寄存器的值。

如果系统调用号非法，`syscall`返回-1

#### 4.4 Code: System call arguments

用户代码调用系统调用的时候，按照调用约定，最开始的时候是将参数放在寄存器，但是最终内核是从TrapFrame里检索参数的。

函数`argint`, `argaddr`和`argfd`可用于从陷阱帧检索系统调用的第n个参数，分别是以整数、地址、文件描述符的形式。这三个函数都调用了`argraw`来从陷阱帧检索被保存的适当的寄存器。



有些系统调用传递指针作为参数，内核必须使用这些指针来读写用户内存。比如exec系统调用的“命令行参数”这个参数，就是传递给内核的一个字符串指针数组，这些字符串都是存储在用户内存里的，内核必须从用户内存里读取这些字符串参数。

这给内核带来两个需要考虑的地方：

- 有些虚拟地址可能是非法的，或者恶意的，比如恶意欺骗内核访问内核内存。
- 内核在处理这些虚拟地址的时候，`satp`寄存器指向的是内核的页表，所以内核没办法使用常规的load或者store指令来直接从虚拟地址读写。

为了安全地在用户提供的地址传输数据，内核实现了一些函数，比如`fetchstr`。`fetchstr`则调用`copyinstr`来干活。

`copyinstr`则从虚拟地址`srcva`复制最多`max`字节到`dst`，并且需要传入用户的页表`pagetable`。它先调用`walkaddr`得到虚拟地址`srcva`对应的物理地址`pa0`，然后从`pa0`直接复制字节到`dst`（内核使用的是直接映射，所以`dst`就是物理地址）。`walkaddr`则会检查地址的合法性，确保`srcva`是进程的地址空间的一部分。

#### 4.5 Traps from kernel space

Xv6在CPU处于内核态和用户态的时候，对于陷阱寄存器的配置是不一样的。在CPU上运行的是内核时，内核会让`stvec`寄存器指向`kernelvec (kernel/kernelvec.S:10)`。因为内核已经在运行了，所以`kernelvec`无需做切换到内核页表和切换到内核栈的工作，可直接认为`satp`指向了内核页表，栈指针也指向了合法的内核栈。

`kernelvec`会保存被打断的代码的所有寄存器以便恢复。

`kernelvec`将所有寄存器的值保存在被打断的内核线程的栈。主要原因是，陷阱可能导致切换到另外一个线程，这样陷阱实际上返回到了新的线程的栈上。

> This is particularly important if the trap causes a switch to a different thread – in that case the trap will actually return on the stack of the new thread, leaving the interrupted thread’s saved registers safely on its stack.

`kernelvec`在保存好寄存器之后，就会跳转到`kerneltrap`，`kerneltrap`只为设备中断和异常（device interrupts and exceptions）做了处理的准备。对于设备中断，调用`devintr (kernel/trap.c:177)`来检查和处理。如果陷阱不是中断，那就一定是异常，出现在内核的异常对于Xv6的内核是致命错误，内核直接panic然后罢工。



如果是由于时钟中断调用的`kerneltrap`，并且一个进程的内核线程正在运行，那么`kerneltrap`会调用`yield`让出CPU。之后某个时间点，我们之前的线程以及`kerneltrap`会恢复运行。

我有一个问题，原文为什么要强调不是调度线程？应该是因为调度线程本身就是为了调度下一个运行的进程，而时钟中断也是为了达到这样的目的？

> If kerneltrap was called due to a timer interrupt, and a process’s kernel thread is running (rather than a scheduler thread), kerneltrap calls yield to give other threads a chance to run. At some point one of those threads will yield, and let our thread and its kerneltrap resume again. Chapter 7 explains what happens in yield.

`kerneltrap`完事之后需要返回被打断的地方。但是因为可能出现`yield`导致`sepc/sstatus`寄存器之类的被另外一个线程覆盖（比如另外这个线程利用ECALL触发Trap），所以`kerneltrap`开始的时候也会保存一遍这些寄存器。`kerneltrap`返回的时候把这些寄存器恢复，然后返回到`Kernelvec`。`kernelvec`把内核线程的寄存器值从栈里弹出来（最开始保存的），执行`sret`，然后恢复执行被打断的内核代码。



当一个CPU从用户空间进入内核的时候，Xv6会将它的`stvec`设置为`kerneltrap`（`usertrap (kernel/trap.c:29)`）。

问题：这个时间窗口什么时候出现？为什么要关中断？

>  There’s a window of time when the kernel is executing but stvec is set to uservec, and it’s crucial that device interrupts be disabled during that window. Luckily the RISC-V always disables interrupts when it starts to take a trap, and xv6 doesn’t enable them again until after it sets stvec.

#### 4.7 Real world

如果将内核内存映射到每个进程的用户页表中，可以消除特殊的跳板页的需求。同时，这也可以消除从用户空间陷入内核时的页表切换的需要。这进一步允许系统调用在内核中直接使用当前进程的用户内存映射，允许内核代码直接解引用用户指针。许多操作系统已经采用这些思想来提高效率。Xv6避免使用这些思想，以减少由于意外使用用户指针而导致内核中的安全漏洞的可能性，并减少确保用户和内核虚拟地址不重叠所需的一些复杂性



所以跳板页在处理Trap中所扮演的角色是什么？

### 阅读源码

这一节要求阅读的源码不多，总共三个和陷阱有关的。并且调用顺序也挺明确的。

- `uservec (kernel/trampoline.S:16)` ==> `usertrap (kernel/trap.c:37)` 
- `usertrapret (kernel/trap.c:90)` ==> ` userret (kernel/trampoline.S:16)`
- 至于`riscv.h`，应该是定义了内核使用的一系列的控制寄存器

按照Xv6 book的顺序，先看`riscv.h`吧，看看寄存器有哪些，4.1节只提了几个最重要的。

#### riscv.h

第一个是`stvec`，*Supervisor Trap-Vector Base Address*寄存器，陷阱向量基地址寄存器。代码很简单，就只有对这个寄存器的读和写。最低2位是模式。代码是用汇编写的。

- csr：Control State Register。控制状态寄存器。csrw和csrr就是写/读控制状态寄存器。

```c
// Supervisor Trap-Vector Base Address
// low two bits are mode.
static inline void 
w_stvec(uint64 x)
{
  asm volatile("csrw stvec, %0" : : "r" (x));
}

static inline uint64
r_stvec()
{
  uint64 x;
  asm volatile("csrr %0, stvec" : "=r" (x) );
  return x;
}
```

然后是`Sscratch`寄存器：

```c
// Supervisor Scratch register, for early trap handler in trampoline.S.
static inline void 
w_sscratch(uint64 x)
{
  asm volatile("csrw sscratch, %0" : : "r" (x));
}

static inline void 
w_mscratch(uint64 x)
{
  asm volatile("csrw mscratch, %0" : : "r" (x));
}
```

然后是`Scause`寄存器，记录陷阱出现的原因的寄存器。很怪，居然没有写入的代码。

```c
// Supervisor Trap Cause
static inline uint64
r_scause()
{
  uint64 x;
  asm volatile("csrr %0, scause" : "=r" (x) );
  return x;
}
```

然后是*Supervisor Status Register, sstatus*寄存器。使用掩码的形式给状态寄存器写入不同的比特。

- 与这个寄存器挂钩的还有开关中断以及查询中断是否打开的三个函数。
- 开关中断都只改变第1个比特，用当前寄存器的值与SSTATUS_SIE进行按位或即可打开，与~SSTATUS_SIE按位与即可关闭

```c
// Supervisor Status Register, sstatus

#define SSTATUS_SPP (1L << 8)  // Previous mode, 1=Supervisor, 0=User
#define SSTATUS_SPIE (1L << 5) // Supervisor Previous Interrupt Enable
#define SSTATUS_UPIE (1L << 4) // User Previous Interrupt Enable
#define SSTATUS_SIE (1L << 1)  // Supervisor Interrupt Enable
#define SSTATUS_UIE (1L << 0)  // User Interrupt Enable

static inline uint64
r_sstatus()
{
  uint64 x;
  asm volatile("csrr %0, sstatus" : "=r" (x) );
  return x;
}

static inline void 
w_sstatus(uint64 x)
{
  asm volatile("csrw sstatus, %0" : : "r" (x));
}

// enable device interrupts
static inline void
intr_on()
{
  w_sstatus(r_sstatus() | SSTATUS_SIE);
}

// disable device interrupts
static inline void
intr_off()
{
  w_sstatus(r_sstatus() & ~SSTATUS_SIE);
}

// are device interrupts enabled?
static inline int
intr_get()
{
  uint64 x = r_sstatus();
  return (x & SSTATUS_SIE) != 0;
}
```

`sepc`寄存器：也只有读和写两个函数，都是用汇编写的

```c
// machine exception program counter, holds the
// instruction address to which a return from
// exception will go.
static inline void 
w_sepc(uint64 x)
{
  asm volatile("csrw sepc, %0" : : "r" (x));
}

static inline uint64
r_sepc()
{
  uint64 x;
  asm volatile("csrr %0, sepc" : "=r" (x) );
  return x;
}
```

#### trampoline.S==>trap.c

从用户空间进入陷阱的时候，是`uservec (kernel/trampoline.S:16)` ==> `usertrap (kernel/trap.c:37)` 

`trampoline.S`是用于在用户空间和内核空间之间切换的代码。这些代码在内核地址空间和用户虚拟地址空间里被映射到同一个虚拟地址。

`trampoline.S`里面其实就两个使用汇编代码写的函数，`uservec`和`userret`。从用户空间陷入内核的时候调用的是前者。处理完陷阱返回的时候调用的是后者。

直接把解析写在注释里面吧。

最开始的时候内核会在`sscratch`放置用户进程的陷阱帧的指针，然后`uservec`交换a0和`sscratch`，`a0`就指向了陷阱帧。

概括一下，`uservec`依次做了如下几件事：

1. 将除了PC之外剩下的31个寄存器的值保存到陷阱帧
2. 将栈指针寄存器sp切换到内核栈
3. 切换页表，刷新TLB
4. 跳转到usertrap()

```assembly
uservec:    
	#
        # trap.c sets stvec to point here, so
        # traps from user space start here,
        # in supervisor mode, but with a
        # user page table.
        #
        # sscratch points to where the process's p->trapframe is
        # mapped into user space, at TRAPFRAME.
        #
        
	# swap a0 and sscratch
        # so that a0 is TRAPFRAME
        csrrw a0, sscratch, a0 # csrrw 用于存储读取到的值的寄存器, 先读取后写入的寄存器, 用于存储准备写入的内容的寄存器
        # 所以这一句实际上就是，读取sscratch的值同时把a0的值写入sscratch，然后把读取到的sscratch的值写入a0.也就是交换两个寄存器的值

        # save the user registers in TRAPFRAME
        sd ra, 40(a0) # sd是一条存储双字（64位）数据的指令，用于将一个通用寄存器的值存储到内存中的指定地址。
        sd sp, 48(a0) # sd register, offset(Start)。将register的值存储到相对于起始位置Start,偏移量为offset的地方
        sd gp, 56(a0) # 相邻两条指令之间的偏移量之差，大部分恰好是8字节。
        sd tp, 64(a0)
        sd t0, 72(a0)
        sd t1, 80(a0)
        sd t2, 88(a0)
        sd s0, 96(a0)
        sd s1, 104(a0) # 26和28之间偏移量相差2*8字节是因为偏移量112处放的应该是a0原本的值。所以这里跳过了一个寄存器的sd，所以差变成了16
        #这里按理还有一条sd a0 112(a0)。但是由于a0现在指向了陷阱帧，而用户的a0暂存在sscratch。所以在后面才保存了它
        sd a1, 120(a0)
        sd a2, 128(a0)
        sd a3, 136(a0)
        sd a4, 144(a0)
        sd a5, 152(a0)
        sd a6, 160(a0)
        sd a7, 168(a0)
        sd s2, 176(a0)
        sd s3, 184(a0)
        sd s4, 192(a0)
        sd s5, 200(a0)
        sd s6, 208(a0)
        sd s7, 216(a0)
        sd s8, 224(a0)
        sd s9, 232(a0)
        sd s10, 240(a0)
        sd s11, 248(a0)
        sd t3, 256(a0)
        sd t4, 264(a0)
        sd t5, 272(a0)
        sd t6, 280(a0)

	# save the user a0 in p->trapframe->a0
        csrr t0, sscratch # a0原有的值放在了sscratch，然后又把sscratch的值临时放在了t0
        sd t0, 112(a0) # 这里才是将原本a0的值存储在相对于陷阱帧的112偏移量处。也就是27行的地方本来应该做的

        # restore kernel stack pointer from p->trapframe->kernel_sp 
        ld sp, 8(a0) # 所以切换栈的本质是改变sp寄存器的值？这里是切换为内核栈
        
        # make tp hold the current hartid, from p->trapframe->kernel_hartid
        ld tp, 32(a0) # tp, Thread Pointer.存储当前正在运行的线程的ID?还能不能存储别的?

        # load the address of usertrap(), p->trapframe->kernel_trap
        ld t0, 16(a0) # 把usertrap()的地址放入t0，最后会跳转到usertrap()。usertrap()的地址放在陷阱帧里面

        # restore kernel page table from p->trapframe->kernel_satp
        ld t1, 0(a0) # 把内核页表的地址写入t1，然后从t1写入satp寄存器，相当于切换到内核页表。
        csrw satp, t1
        sfence.vma zero, zero # 切换页表就要刷新TLB

        # a0 is no longer valid, since the kernel page
        # table does not specially map p->tf.

        # jump to usertrap(), which does not return
        jr t0

```

然后是`uservec`跳转到的`usertrap`：末尾实际上调用了`usertrapret()`，但是这个会放在下一节看

`usertrap`同时还调用了`devintr()`，这个也在`trap.c`里面，所以一起看了吧。至于`syscall()`，没要求看`syscall.c`，后面再看看。

概况一下`usertrap`做的事：

1. 将`stvec`设置为KernelVec
2. 判断Trap种类，系统调用给syscall处理，设备中断给devintr处理，异常直接杀进程，如果是时钟中断调用yield。
3. 调用usertrapret返回。

> Q:为什么在处理系统调用时会调用 `intr_on()` 来开启中断？
>
> GPT：在处理系统调用时，一些系统调用可能会导致进程进入睡眠状态（比如Sleep）或等待某些事件的发生。为了确保在进程等待期间，其他的中断能够被及时响应，可以在此时开启中断。在处理完与系统调用相关的工作之后，中断可以再次被禁用。

```c
// handle an interrupt, exception, or system call from user space.
// called from trampoline.S
//
void
usertrap(void)
{
  int which_dev = 0;

  if((r_sstatus() & SSTATUS_SPP) != 0) //读取状态寄存器,利用掩码查看被Trap打断之前是什么模式，0是user mode.
    panic("usertrap: not from user mode"); //所以假如查看到非0,就不是从user mode来的

  // send interrupts and exceptions to kerneltrap(),
  // since we're now in the kernel.
  w_stvec((uint64)kernelvec);/* 这里将stvec寄存器设置为了kernelvec的地址。
  因为我们现在处于内核之中，所以假如中途再出现Trap的话，我们就是Trap From Kernel，
  不能再由Uservec来处理，Uservec是处理来自User Mode的Trap的。来自内核的Trap需要让kernelvec去处理
  */

  struct proc *p = myproc(); //myproc()返回的是当前的进程
  
  // save user program counter.
  p->trapframe->epc = r_sepc(); // 将sepc寄存器的值存储到陷阱帧
  
  if(r_scause() == 8){ //读取scause寄存器，查明Trap出现的原因
    // system call

    if(p->killed)
      exit(-1);

    // sepc points to the ecall instruction,
    // but we want to return to the next instruction.
    p->trapframe->epc += 4; //编辑epc，让epc指向ECALL指令的下一条指令

    // an interrupt will change sstatus &c registers,
    // so don't enable until done with those registers.
    intr_on(); //这里我觉得和注释有点矛盾啊，不应该是关中断吗?
      // 嗷，中断在最开始RISC-V硬件关闭的(4.1)，所以这里要打开。因为已经处理完了。

    syscall(); //调用syscall来处理系统调用(syscall.c，后面再看吧)
  } else if((which_dev = devintr()) != 0){ //调用devintr()检查是不是设备中断，是的话刚好顺便检查了
    // ok
  } else { // Exception，直接杀掉进程
    printf("usertrap(): unexpected scause %p pid=%d\n", r_scause(), p->pid);
    printf("            sepc=%p stval=%p\n", r_sepc(), r_stval());
    p->killed = 1;
  }

  if(p->killed)
    exit(-1);

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2) //这里的which_dev在调用devintr()的时候，devintr在中断是时钟中断的时候会返回2。
    yield();

  usertrapret();
}
```

然后是`devintr`：对于无法识别的Trap返回0，这也是`usertrap`利用返回值来判断是否是设备中断，是哪个设备，以及处理是否成功的原因。

当然中断这一部分应该是在Lecture 9讲，Xb6 Book上也没对这里展开，并且这一部分源码目前还有一部分是看不懂的，有关plic的部分。总之先当个黑盒吧。

黑盒：`devintr`会检查当前Trap是否是一个中断，然后处理之，然后返回对应的值表示这是一个什么中断。

```c

// check if it's an external interrupt or software interrupt,
// and handle it.
// returns 2 if timer interrupt,
// 1 if other device,
// 0 if not recognized.
int
devintr()
{
  uint64 scause = r_scause();

  if((scause & 0x8000000000000000L) &&
     (scause & 0xff) == 9){
    // this is a supervisor external interrupt, via PLIC.

    // irq indicates which device interrupted.
    int irq = plic_claim();

    if(irq == UART0_IRQ){
      uartintr();
    } else if(irq == VIRTIO0_IRQ){
      virtio_disk_intr();
    } else if(irq){
      printf("unexpected interrupt irq=%d\n", irq);
    }

    // the PLIC allows each device to raise at most one
    // interrupt at a time; tell the PLIC the device is
    // now allowed to interrupt again.
    if(irq)
      plic_complete(irq);

    return 1;
  } else if(scause == 0x8000000000000001L){
    // software interrupt from a machine-mode timer interrupt,
    // forwarded by timervec in kernelvec.S.

    if(cpuid() == 0){
      clockintr();
    }
    
    // acknowledge the software interrupt by clearing
    // the SSIP bit in sip.
    w_sip(r_sip() & ~2);

    return 2;
  } else {
    return 0;
  }
}
```

#### trap.c==>trampoline.S

从`userTrap`返回的时候，是`usertrapret (kernel/trap.c:90)` ==> ` userret (kernel/trampoline.S:16)`

usertrapret做了如下几件事：

1. 关中断
2. 设置stvec指向uservec
3. 配置陷阱帧
4. 恢复sepc。调用userret，传递给userret页表和陷阱帧。

```c
// return to user space
//
void
usertrapret(void)
{
  struct proc *p = myproc();

  // we're about to switch the destination of traps from
  // kerneltrap() to usertrap(), so turn off interrupts until
  // we're back in user space, where usertrap() is correct.
  intr_off();/*
  因为我们处于将traps的目的地从kerneltrap到usertrap的切换中，并且现在我们还在内核中
  所以先关闭中断，防止出现某个内核中断走向了usertrap的处理路径
  */

  // send syscalls, interrupts, and exceptions to trampoline.S
  w_stvec(TRAMPOLINE + (uservec - trampoline)); //让stvec指向uservec。因为这是来自用户空间的陷阱应该去的地方
    /*
    这里的TRAMPOLINE + (uservec - trampoline)其实可以理解为起始地址 + 偏移量，得到uservec的地址。
    (uservec - trampoline)就相当于uservec相对于TRAMPOLINE的偏移量
    */

  // set up trapframe values that uservec will need when
  // the process next re-enters the kernel.
    //这里设置了uservec会使用的陷阱帧中的值，以便进程下次再进入内核时使用。那么问题来了，第一次uservec使用的值是谁设置的？
  p->trapframe->kernel_satp = r_satp();         // kernel page table
  p->trapframe->kernel_sp = p->kstack + PGSIZE; // process's kernel stack
  p->trapframe->kernel_trap = (uint64)usertrap;
  p->trapframe->kernel_hartid = r_tp();         // hartid for cpuid() 

  // set up the registers that trampoline.S's sret will use
  // to get to user space.
  
  // set S Previous Privilege mode to User.
  unsigned long x = r_sstatus();
  x &= ~SSTATUS_SPP; // clear SPP to 0 for user mode
  x |= SSTATUS_SPIE; // enable interrupts in user mode
  w_sstatus(x);

  // set S Exception Program Counter to the saved user pc.
  w_sepc(p->trapframe->epc);

  // tell trampoline.S the user page table to switch to.
  uint64 satp = MAKE_SATP(p->pagetable);

  // jump to trampoline.S at the top of memory, which 
  // switches to the user page table, restores user registers,
  // and switches to user mode with sret.
  uint64 fn = TRAMPOLINE + (userret - trampoline); //fn是userret的地址。在下一行将其转换成了一个函数指针，然后执行函数fn
    //fn的计算也是采用起始地址TRAMPOLINE + 偏移量(userret - trampoline)
  ((void (*)(uint64,uint64))fn)(TRAPFRAME, satp);
}
```

接着是被`usertrapret`调用的`userret`：

```assembly
.globl userret
userret:
        # userret(TRAPFRAME, pagetable)
        # switch from kernel to user.
        # usertrapret() calls here.
        # a0: TRAPFRAME, in user page table.
        # a1: user page table, for satp.

        # switch to the user page table.
        csrw satp, a1
        sfence.vma zero, zero # 切换到用户页表，并刷新TLB

        # put the saved user a0 in sscratch, so we
        # can swap it with our a0 (TRAPFRAME) in the last step.
        ld t0, 112(a0) # 先把原用户的a0放到sscratch。最后再和a0换回来。此时a0指向陷阱帧了
        csrw sscratch, t0

        # restore all but a0 from TRAPFRAME
        ld ra, 40(a0)
        ld sp, 48(a0)
        ld gp, 56(a0)
        ld tp, 64(a0)
        ld t0, 72(a0)
        ld t1, 80(a0)
        ld t2, 88(a0)
        ld s0, 96(a0)
        ld s1, 104(a0)
        ld a1, 120(a0)
        ld a2, 128(a0)
        ld a3, 136(a0)
        ld a4, 144(a0)
        ld a5, 152(a0)
        ld a6, 160(a0)
        ld a7, 168(a0)
        ld s2, 176(a0)
        ld s3, 184(a0)
        ld s4, 192(a0)
        ld s5, 200(a0)
        ld s6, 208(a0)
        ld s7, 216(a0)
        ld s8, 224(a0)
        ld s9, 232(a0)
        ld s10, 240(a0)
        ld s11, 248(a0)
        ld t3, 256(a0)
        ld t4, 264(a0)
        ld t5, 272(a0)
        ld t6, 280(a0)

	# restore user a0, and save TRAPFRAME in sscratch
        csrrw a0, sscratch, a0 #恢复用户的a0，把陷阱帧存在sscratch
        
        # return to user mode and user pc.
        # usertrapret() set up sstatus and sepc.
        sret

```

#### kerneltrap

Kerneltrap是Kernelvec调用的，用于处理来自内核的Trap——设备中断或者是异常。前者交给驱动，后者直接Panic。

```c
// interrupts and exceptions from kernel code go here via kernelvec,
// on whatever the current kernel stack is.
void 
kerneltrap()
{
  int which_dev = 0;
  uint64 sepc = r_sepc();
  uint64 sstatus = r_sstatus();
  uint64 scause = r_scause();
  
  if((sstatus & SSTATUS_SPP) == 0)
    panic("kerneltrap: not from supervisor mode");
  if(intr_get() != 0)
    panic("kerneltrap: interrupts enabled");

  if((which_dev = devintr()) == 0){ //交给devintr处理，如果devintr返回0，说明是Exception。直接Panic
    printf("scause %p\n", scause);
    printf("sepc=%p stval=%p\n", r_sepc(), r_stval());
    panic("kerneltrap");
  }

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2 && myproc() != 0 && myproc()->state == RUNNING) //如果中断是时钟中断，并且当前进程非空且处于RUNNINg状态。
    yield(); //让出CPU

  // the yield() may have caused some traps to occur,
  // so restore trap registers for use by kernelvec.S's sepc instruction.
  w_sepc(sepc);
  w_sstatus(sstatus);
}
```

### Lecture

#### 提问

关于Memory Mapped IO

> 学生提问：这个问题或许并不完全相关，read和write系统调用，相比内存的读写，他们的代价都高的多，因为它们需要切换模式，并来回捣腾。有没有可能当你执行打开一个文件的系统调用时， 直接得到一个page table映射，而不是返回一个文件描述符？这样只需要向对应于设备的特定的地址写数据，程序就能通过page table访问特定的设备。你可以设置好限制，就像文件描述符只允许修改特定文件一样，这样就不用像系统调用一样在用户空间和内核空间来回捣腾了。
>
> Robert教授：这是个很好的想法。实际上很多操作系统都提供这种叫做内存映射文件（Memory-mapped file access）的机制，在这个机制里面通过page table，可以将用户空间的虚拟地址空间，对应到文件内容，这样你就可以通过内存地址直接读写文件。实际上，你们将在mmap 实验中完成这个机制。对于许多程序来说，这个机制的确会比直接调用read/write系统调用要快的多。



`Sscratch`寄存器，在`uservec`时期保存了陷阱帧的起始地址，然后它会和a0交换，并在接下来的`uservec`中用起始地址a0+偏移量的方式存储剩下的31个寄存器的值。那么问题来了，`sscratch`里的值又是怎么被设置的？

概括来讲，就是上一次OS从内核返回用户空间的时候，`usertrapret`会设置

> 学生提问：当与a0寄存器进行交换时，trapframe的地址是怎么出现在SSCRATCH寄存器中的？
>
> Robert教授：在内核前一次切换回用户空间时，内核会执行set sscratch指令，将这个寄存器的内容设置为0x3fffffe000，也就是trapframe page的虚拟地址。所以，当我们在运行用户代码，比如运行Shell时，SSCRATCH保存的就是指向trapframe的地址。之后，Shell执行了ecall指令，跳转到了trampoline page，这个page中的第一条指令会交换a0和SSCRATCH寄存器的内容。所以，SSCRATCH中的值，也就是指向trapframe的指针现在存储与a0寄存器中。
>
> 
>
> 在内核返回到用户空间时，会恢复所有的用户寄存器。之后会再次执行交换指令，csrrw。因为之前内核已经设置了a0保存的是trap frame地址，经过交换之后SSCRATCH仍然指向了trapframe page地址，而a0也恢复成了之前的数值。最后sret返回到了用户空间。
>
> 你或许会好奇，a0是如何有trapframe page的地址。我们可以查看trap.c代码（如下图），这是内核返回到用户空间的最后的C函数。C函数做的最后一件事情是调用fn函数，传递的参数是TRAMFRAME和user page table。在C代码中，当你调用函数，**第一个参数会存在a0（而第一个参数就是Trapframe的地址）**，这就是为什么a0里面的数值是指向trapframe的指针。fn函数是就是刚刚我向你展示的位于trampoline.S中的代码。
>
> 
>
> 那，如果我们是第一次执行ECALL指令呢？这总没有所谓的前一次内核切换回用户空间吧？
>
> 或许对于这个问题的一个答案是：一台机器总是从内核开始运行的，当机器启动的时候，它就是在内核中。 任何时候，不管是进程第一次启动还是从一个系统调用返回，进入到用户空间的唯一方法是就是执行sret指令。sret指令是由RISC-V定义的用来从supervisor mode转换到user mode。所以，在任何用户代码执行之前，内核会执行fn函数，并设置好所有的东西，例如SSCRATCH，STVEC寄存器。

![](https://github.com/huihongxiao/MIT6.S081/blob/master/.gitbook/assets/image%20%28241%29.png)



为什么要把寄存器的值保存在陷阱帧，为什么不直接保存在用户的栈里？

> 学生提问：寄存器保存在了trapframe page，但是这些寄存器用户程序也能访问，为什么我们要使用内存中一个新的区域（指的是trapframe page），而不是使用程序的栈？
>
> Robert教授：好的，这里或许有两个问题。第一个是，为什么我们要保存寄存器？为什么内核要保存寄存器的原因，是因为内核即将要运行会覆盖这些寄存器的C代码。如果我们想正确的恢复用户程序，我们需要将这些寄存器恢复成它们在ecall调用之前的数值，所以我们需要将所有的寄存器都保存在trapframe中，这样才能在之后恢复寄存器的值。
>
> 另一个问题是，为什么这些寄存器保存在trapframe，而不是用户代码的栈中？这个问题的答案是，我们不确定用户程序是否有栈，必然有一些编程语言没有栈，对于这些编程语言的程序，Stack Pointer不指向任何地址。当然，也有一些编程语言有栈，但是或许它的格式很奇怪，内核并不能理解。比如，编程语言以堆中以小块来分配栈，编程语言的运行时知道如何使用这些小块的内存来作为栈，但是内核并不知道。所以，如果我们想要运行任意编程语言实现的用户程序，内核就不能假设用户内存的哪部分可以访问，哪部分有效，哪部分存在。所以内核需要自己管理这些寄存器的保存，这就是为什么内核将这些内容保存在属于内核内存的trapframe中，而不是用户内存。

#### 以系统调用为例：进入内核

需要特别指出的是，supervisor mode中的代码**并不能读写任意物理地址**（只能读写在内核的页表中合法的地址）。在supervisor mode中，就像普通的用户代码一样，也需要通过page table来访问内存。如果一个虚拟地址并不在当前由SATP指向的page table中，又或者SATP指向的page table中PTE_U=1，那么supervisor mode不能使用那个地址。所以，即使我们在supervisor mode，我们还是受限于当前page table设置的虚拟地址。



当用户空间出现一个Trap的时候，以系统调用为例，其代码的执行流程是：

`ECALL`指令触发一个Trap，在硬件将`Stvec`和`Sepc`寄存器设置好，并将PC设置为`Stvec`的值之后，第一个执行的函数是`uservec`。

`uservec`的工作做完之后，调用`trap.c/usertrap`。`usertrap`则检查`scourse`寄存器查看Trap出现的原因，发现是一个系统调用，于是调用`syscall`函数。`syscall`函数则查看用户进程传入的系统调用号（去找Trapframe里保存的a7寄存器），然后调用相应的系统调用函数。

系统调用函数执行完毕，`return`到`syscall`函数，`syscall`则调用`usertrapret`。

`usertrapret`做一部分工作，然后调用`trampoline.s/userret`函数，`userret`函数会把PC设置成用户进程被中断的地方，然后用户进程继续执行，仿佛什么都没发生。



执行完`ECALL`指令的时候，并没有立即切换到内核页表，此刻页表还是用户页表，这个时候执行的是跳板页里的指令（也就是`uservec`）。此刻寄存器里的值也还是用户进程的值，`uservec`负责把这些寄存器的值保存到用户进程的Trapframe。

ecall并不会切换page table（**但是会将模式切换到内核模式**），这是ecall指令的一个非常重要的特点。**所以这意味着，trap处理代码必须存在于每一个user page table中。因为ecall并不会切换page table，我们需要在user page table中的某个地方来执行最初的内核代码**。而这个trampoline page，是由内核小心的映射到每一个user page table中，**以使得当我们仍然在使用user page table时，内核在一个地方能够执行trap机制的最开始的一些指令**。



**在从内核空间返回用户空间的时候，内核会把`stvec`寄存器的值重新设置好**，用于下一次使用。



Trapframe里除了保存了32个用户寄存器的值，**还保存了内核页表的地址、内核栈顶地址，`usertrap`的地址**。

总结下来，ECALL指令做的事：

1. 将代码从user mode改到supervisor mode。
2. 将程序计数器的值保存在了SEPC寄存器。
3. 跳转到STVEC寄存器指向的指令。也就是跳转到uservec。

然后`uservec`为我们做的事：

1. 保存32个用户寄存器的内容，这样当我们想要恢复用户代码执行时，我们才能恢复这些寄存器的内容。
2. 因为现在我们还在user page table，我们需要切换到kernel page table。
3. 创建或者找到一个kernel stack，并将Stack Pointer寄存器的内容指向那个kernel stack。这样才能给C代码提供栈。
4. 跳转到内核中C代码的某些合理的位置（`usertrap.c`）。

> 这里还有个问题，为什么代码没有崩溃？毕竟我们在内存中的某个位置执行代码，程序计数器保存的是虚拟地址，如果我们切换了page table，为什么同一个虚拟地址不会通过新的page table寻址走到一些无关的page中？看起来我们现在没有崩溃并且还在执行这些指令。有人来猜一下原因吗？
>
> > 学生回答：因为我们还在trampoline代码中，而**trampoline代码在用户空间和内核空间都映射到了同一个地址。**
>
> 完全正确。我不知道你们是否还记得user page table的内容，trampoline page在user page table中的映射与kernel page table中的映射是完全一样的。这两个page table中其他所有的映射都是不同的，只有trampoline page的映射是一样的，因此我们在切换page table时，寻址的结果不会改变，我们实际上就可以继续在同一个代码序列中执行程序而不崩溃。这是trampoline page的特殊之处，它同时在user page table和kernel page table都有相同的映射关系。
>
> 之所以叫trampoline page，是因为你某种程度在它上面“弹跳”了一下，然后从用户空间走到了内核空间。

跳转到`usertrap.c`之后，它为我们做的事：

1. 更改STVEC寄存器。让其指向kernelvec。
   - 因为xv6对于来自内核和来自用户空间的Trap的处理是不一样的，`stvec`指向的又是处理Trap的程序，而如果Trap是来自内核的，那么因为从内核发起的话，程序已经在使用kernel page table。所以当trap发生时，程序执行仍然在内核的话，很多处理都不必存在（比如切换页表，切换到内核栈），所以在内核中执行任何操作之前，usertrap中先将STVEC指向了kernelvec变量，这是内核空间trap处理代码的位置，而不是用户空间trap处理代码的位置。
   - 用我自己的话来讲就是，`usertrap`此时已经处于内核中了，所以`stvec`应该指向另外一个处理来自内核的Trap的处理程序。因为处于内核的时候发生的Trap（也就是来自内核的Trap，Trap发生的时候处于内核态就是来自内核的Trap），和处于用户空间时的Trap，其处理的流程是很不一样的，至少前者可以省略掉切换内核页表和内核栈的步骤，所以当我们处于内核中的时候，`stvec`应该指向一个更加精简的`Trap`处理程序。
2. 把用户的PC的值保存到陷阱帧。
   - 防止出现这种情况：当程序还在内核中执行时，我们可能切换到另一个进程，并进入到那个程序的用户空间，然后那个进程可能再调用一个系统调用进而导致SEPC寄存器的内容被覆盖。所以，我们需要保存当前进程的SEPC寄存器到一个与该进程关联的内存中，这样这个数据才不会被覆盖。
3. 读取SCAUSE寄存器，找到触发Trap的原因。根据原因去执行不同的操作
4. 因为是系统调用，所以操作是调用`syscall`函数

`syscall`函数则从陷阱帧里读取a7寄存器的值，这是系统调用号。然后根据系统调用号，查找对应的系统调用函数具体实现，并调用之。

但是用户传递给系统调用的参数怎么办？syscall函数直接通过trapframe来获取这些参数，就像查看trapframe中的a7寄存器一样，我们可以查看a0寄存器，这是第一个参数，a1是第二个参数，a2是第三个参数。

最后就是具体的系统调用函数去执行了。

在系统调用具体的实现执行完毕后，它会返回`syscall`函数。

而在syscall函数中，有一行值得额外说一下：按照调用约定，返回值是放在a0或者是 a0和a1，这里把Trapframe的a0赋值，就是为了模拟函数的返回值

```c
p->Trapframe -> a0 = syscalls[num];
/*
这里向trapframe中的a0赋值的原因是：所有的系统调用都有一个返回值，比如write会返回实际写入的字节数，而RISC-V上的C代码的习惯是函数的返回值存储于寄存器a0，所以为了模拟函数的返回，我们将返回值存储在trapframe的a0中。之后，当我们返回到用户空间，trapframe中的a0槽位的数值会写到实际的a0寄存器，Shell会认为a0寄存器中的数值是write系统调用的返回值。
*/
```



`syscall`函数则返回`usertrap`函数。

`usertrap`则随后调用了`usertrapret`函数，开始执行离开内核的动作

#### 以系统调用为例：离开内核

`usertrapret`所做的事：

1. 关闭中断。防止出现一个来自内核空间的Trap，走向了`uservec`的处理路径。

> 我们之前在系统调用的过程中是打开了中断的，这里关闭中断是因为我们将要更新STVEC寄存器来指向用户空间的trap处理代码，而之前在内核中的时候，我们指向的是内核空间的trap处理代码（6.6）。我们关闭中断因为**当我们将STVEC更新到指向用户空间的trap处理代码时，我们仍然在内核中执行代码。如果这时发生了一个中断，那么程序执行会走向用户空间的trap处理代码，即便我们现在仍然在内核中**，出于各种各样具体细节的原因，这会导致内核出错。所以我们这里关闭中断。

2. 设置STVEC寄存器，指向trampoline代码`uservec`。
3. 往Trapframe里填入一些值：kernel page table的指针、当前用户进程的kernel stack、usertrap函数的指针（这样uservec才能跳转到这个函数）、从tp寄存器中读取当前的CPU核编号
4. 将`sepc`设置成此前保存的用户的PC的值。因为最后的`sret`指令会将CPU的PC的值设置为sepc。
5. 计算出satp的值，计算出函数指针`userret`的值，以函数指针的形式调用`userret`，传入参数是`satp`和Trapframe的地址。



最后就是跳板页的`trampoline.S/userret`：a0此时存储的是Trapframe的地址（前一部分的函数指针传过来的参数）

1. 切换到用户页表（切换页表的同时也要清理掉TLB缓存项）。用户页表也映射了trampoline page，所以程序还能继续执行而不是崩溃。
2. 恢复用户的所有寄存器的值，值得一提的是**Trapframe里的a0**存储的不再是原本的a0了，而是系统调用的返回值。
3. 交换SSCRATCH寄存器和a0寄存器的值。前面我们看过了SSCRATCH现在的值是系统调用的返回值2，a0寄存器是trapframe的地址。交换完成之后，a0持有的是系统调用的返回值，SSCRATCH持有的是trapframe的地址。之后trapframe的地址会一直保存在SSCRATCH中，直到用户程序执行了另一次trap。现在我们还在kernel中。
4. 执行sret，切换回用户模式，恢复PC的值，并且打开中断。用户程序恢复执行

## Lecture 8

### 阅读4.6 PageFaults

Xv6对于异常的处理很无聊，来自用户空间的异常就直接杀用户进程，来自内核的异常内核就直接Panic。

当CPU无法将一个虚拟地址翻译为物理地址的时候，CPU会生成一个Page Fault。RISC-V有三种缺页异常：分别出现在执行Load、Store、指令译码的时候，无法将虚拟地址翻译为物理地址的时候。

-  load page faults (when a load instruction cannot translate its virtual address)
- store page faults (when a store instruction cannot translate its virtual address)
- instruction page faults (when the address for an instruction doesn’t translate)

`scaus`寄存器中的值会表明出现的是哪一种缺页异常，`stval`寄存器中则包含了无法被翻译的虚拟地址。



CopyOnWrite-Fork的基本思想：最开始让子进程和父进程共享所有的物理页，但是把这些物理页（对应的PTE）标记为只读的。所以，假如父进程或者子进程执行一条Store指令（也就是对某个虚拟地址写入的时候）的时候，CPU抛出一个Page Fault。内核为了响应这个Page Fault，复制一份包含那个触发异常的地址的页。总共两份拷贝，内核把其中一份在父进程的地址空间标记为R/W（修改PTE标志位），然后把另外一份在子进程的地址空间标记为R/W。在更新了页表之后，内核恢复触发异常的进程，然后重新执行触发缺页异常的那条指令。因为内核已经更新了相关的PTE，所以这次重新执行不会再次触发缺页异常。

COW-Fork的评价：

> This COW plan works well for fork, because often the child calls exec immediately after the fork, replacing its address space with a new address space. In that common case, the child will experience only a few page faults, and the kernel can avoid making a complete copy. Furthermore, COW fork is transparent: no modifications to applications are necessary for them to benefit

另外一种结合页表和缺页异常的技术是Lazy Allocation，这个技术分为两步：

1. 当进程调用sbrk，内核就增长虚拟地址空间，但是把新地址在页表中都标记为非法。
2. 当在这些新地址上出现Page Fault的时候，内核分配物理页，然后在页表中映射它。

> Since applications often ask for more memory than they need, lazy allocation is a win: the kernel allocates memory only when the application actually uses it. Like COW fork, the kernel can implement this feature transparently to applications.



再另外一种：*paging from disk*，其实就是Swap。

如果应用程序需要比可用物理RAM更多的内存（是可用，不是总容量），内核可以回收（evict）一些页面：将它们写入磁盘等存储设备，并将其PTE标记为无效。如果应用程序读取或写入被逐出（evicted）的页面，则CPU将出现Page Fault。然后内核可以检查出错的地址，如果地址属于磁盘上的页面，内核会分配一页物理内存，将页面从磁盘读取到该内存，将PTE更新为有效并引用该内存，然后恢复应用程序（并重新执行触发异常的指令）。**为了给来自磁盘的页面腾出空间，内核可能不得不驱逐（evict）出另一个页面**。（驱逐和回收都指evict，也即把页面写入硬盘的行为）

### Lecture

这一小节其实讲的是利用PageFault实现的一些OS的常见特性，比如页的lazy allocation，CopyOnWrite，DemandPaging，memory mapped files

page fault可以让页表中静态的地址映射关系变得动态起来。通过page fault，内核可以更新page table。



第一个问题：当发生page fault时，内核需要什么样的信息才能够响应page fault？

- 出错的虚拟地址，或者是触发page fault的源。当出现page fault的时候，XV6内核会打印出错的虚拟地址，并且这个地址会被保存在STVAL寄存器中。
- 出错的原因，因为我们想要对不同场景的page fault有不同的响应。这个信息存储在`Scause`。
  - 触发Page Fault的原因有很多种，我们需要知道具体到哪一种。
  - 不同的场景是指，比如因为load指令触发的page fault、因为store指令触发的page fault又或者是因为jump指令触发的page fault。
- 是触发page fault的指令的地址。这个地址存放在SEPC（Supervisor Exception Program Counter）寄存器中，并同时会保存在trapframe->sepc中。
  - 因为我们需要在处理完Page Fault之后，重新执行刚才触发Page Fault的指令。理想情况下，修复完page table之后，指令就可以无错误的运行了

page fault和其他的异常使用与系统调用相同的trap机制来从用户空间切换到内核空间。

#### Lazy page allocation

`sbrk`系统调用：参数在Linux上是以字节为单位。**但是上课讲的，则是`sbrk`的参数以页为单位**

> `sbrk` 的机制相对简单，它实际上只是移动进程的堆的结束位置。当调用`sbrk`函数并传递一个正值（增加堆大小）或负值（减小堆大小）作为参数时，操作系统会根据这个值来移动堆的结束位置。这样，进程可以在堆的空闲区域中分配或释放内存，实现动态的内存管理。
>
> 实现 `sbrk` 的基本机制如下：
>
> 1. **初始状态**：进程的堆在开始时是一个空的连续内存区域。堆的起始和结束位置由全局变量或操作系统维护。
> 2. **调用 `sbrk`**：当进程调用 `sbrk` 并传递一个正值 `n` 时，操作系统会将堆的结束位置向前移动 `n` 字节，从而分配更多内存给进程。当传递负值 `-n` 时，堆的结束位置会向后移动 `n` 字节，释放一部分内存。
> 3. **分配内存**：当进程需要分配内存时，可以简单地在当前堆的结束位置分配一块连续的内存区域。堆的结束位置随着分配的变化而移动。
> 4. **释放内存**：当进程释放先前分配的内存块时，这些内存块会变为可用状态。如果释放的内存块位于堆的末尾，堆的结束位置可能会向后移动，使得这块内存被合并到堆的空闲区域中
>
> 堆通常是建立在连续的虚拟地址空间中的一块区域，用于在运行时进行内存分配和释放。虽然这个区域的物理存储可能是分散的，但从进程的角度来看，它是一个连续的虚拟地址范围。



Xv6中的`sbrk`是立即分配的（eager allocation）。

立即分配的意思是，一旦调用了sbrk，内核会立即分配应用程序所需要的**物理内存**（与之相对应的是惰性分配，比如，只先在页表上给涉及到的虚拟地址空间添加对应的PTE，但是PTE是非法的，而不立即就分配对应的物理内存）。

但是实际上，对于应用程序来说很难预测自己需要多少内存，所以通常来说，应用程序倾向于申请多于自己所需要的内存。这意味着，进程的内存消耗会增加许多，但是有部分内存永远也不会被应用程序所使用到。

这也就意味着，如果使用立即分配的话，`sbrk`分配给进程的内存空间很有可能是存在浪费的。



因此这就引入了Lazy Page Allocation 机制。

使用虚拟内存和page fault handler，我们来实现一个惰性分配的机制。

利用惰性分配，改写`sbrk`，核心思想非常简单。

> sbrk系统调基本上不做任何事情，唯一需要做的事情就是提升_p->sz_，将_p->sz_增加n，其中n是需要新分配的内存page数量（就是增加heap）。
>
> 只不过，**内核在这个时间点并不会分配任何物理内存**。
>
> 之后在某个时间点，应用程序使用到了新申请的那部分内存，这时会触发page fault，因为我们还没有将新的内存映射到page table。
>
> 所以，如果我们解析一个大于旧的_p->sz_，但是又小于新的*p->sz*（注，也就是旧的*p->sz* + n）的虚拟地址，我们希望内核能够分配一个物理内存page，并且重新执行指令。
>
> 所以，当我们看到了一个page fault，相应的虚拟地址小于当前_p->sz_，同时大于stack，那么我们就知道这是一个来自于heap的地址，但是内核还没有分配任何物理内存。
>
> 对于这个page fault的处理也很简单：
>
> 在对应的page fault handler中，通过kalloc函数分配一个物理内存page；初始化这个page内容为0；将这个内存page映射到user page table中（添加对应的PTE即可）；最后重新执行指令。

#### Lazy Page Allocation的提问

> 学生提问：在eager allocation的场景，一个进程可能消耗了太多的内存进而耗尽了物理内存资源。如果我们不使用eager allocation，而是使用lazy allocation，应用程序怎么才能知道当前已经没有物理内存可用了？
>
> Frans教授：这是个非常好的问题。从应用程序的角度来看，会有一个错觉：存在无限多可用的物理内存。但是在某个时间点，应用程序可能会用光了物理内存，之后如果应用程序再访问一个未被分配的page，但这时又没有物理内存，这时内核可以有两个选择，我稍后会介绍更复杂的那个。你们在lazy lab中要做的是，返回一个错误并杀掉进程。因为现在已经OOM（Out Of Memory）了，内核也无能为力，所以在这个时间点可以杀掉进程。

#### Zero Fill On Demand

这个和Lazy Allocation有点类似。

我们使用一个例子来说明什么是Zero Fill On Demand。

假如我们定义了一个巨大的矩阵，这个矩阵所占据的内存可能有好几页，甚至好几十页，但是这个矩阵的所有元素的初始值都是0。

那我们真的要为这个矩阵分配几十页的物理页，然后把这几十页的物理页都填充为0，然后映射到相应的虚拟页上去吗？

我们完全不必这样，我有如此多的内容全是0的page，在物理内存中，我只需要分配一个物理page，这个page的内容全是0。然后将所有虚拟地址空间的全0的虚拟page（也就是矩阵全0的页）都map到这一个物理page上，**并且把这一物理页设置为只读的**。这样至少在程序启动的时候能节省大量的物理内存分配。

当然，这里的Mapping应该不允许对于这个page执行写操作，因为所有的虚拟地址空间page都期望page的内容是全0，所以这里的PTE都是只读的。

而之后在某个时间点，应用程序尝试写虚拟页中的一个page时，比如说需要更改一两个变量的值，会触发page fault。

我们需要做的是，在物理内存中申请一个新的内存page，将其内容设置为0，因为我们预期这个内存的内容为0。之后我们需要更新这个page的mapping关系，首先PTE要设置成可读可写，然后将其指向新的物理page。这里相当于更新了PTE，之后我们可以重新执行指令。



这样做的好处是，节省内存且程序可以启动的更快，因为你只需要分配一个内容全是0的物理page。所有的全0的虚拟page都可以映射到这一个物理page上。



但是也有代价，因为PageFault是一个Trap，需要陷入内核来处理，它的开销比一条Load或者Store指令要大得多。

####  Copy On Write Fork

 Copy On Write概括一下基本思想就是，我在该Copy的时候我不Copy，我先把要Copy的东西设置为只读，等到某一方要对这个东西进行写入的时候，我再真的去Copy一个副本，让写入者对着副本去写入。这样做的好处就是，我可以赌，这一页只会被读取不会被写入，赌成功了就大大减小开销。



以Fork-Exec场景为例：

当Shell处理指令时，它会通过fork创建一个子进程。fork会创建一个Shell进程的拷贝，所以这时我们有一个父进程（原来的Shell）和一个子进程。Shell的子进程执行的第一件事情就是调用exec运行一些其他程序，比如运行echo。现在的情况是，fork创建了Shell地址空间的一个完整的拷贝，而exec做的第一件事情就是丢弃这个地址空间，取而代之的是一个包含了echo的地址空间。这里看起来有点浪费。 

所以，我们最开始有了一个父进程的虚拟地址空间，然后我们有了子进程的虚拟地址空间。在物理内存中，XV6中的Shell通常会有4个page，当调用fork时，基本上就是创建了4个新的page，并将父进程page的内容拷贝到4个新的子进程的page中。

但是之后，一旦调用了exec，我们又会释放这些page，并分配新的page来包含echo相关的内容。所以对于这个特定场景有一个非常有效的优化：当我们创建子进程时，与其创建，分配并拷贝内容到新的物理内存，其实我们可以直接共享父进程的物理内存page。所以这里，我们可以设置子进程的PTE指向父进程对应的物理内存page。为了确保进程间的隔离性，我们可以将这里的父进程和子进程的PTE的标志位都设置成只读的。

在某个时间点，当我们需要更改内存的内容时，我们会得到page fault。因为父进程和子进程都会继续运行，而父进程或者子进程都可能会执行store指令来更新一些全局变量，这时就会触发page fault，因为现在在向一个只读的PTE写数据。

在得到page fault之后，我们需要拷贝相应的物理page。假设现在是子进程在执行store指令，那么我们会分配一个新的物理内存page，然后将page fault相关的物理内存page拷贝到新分配的物理内存page中，并将新分配的物理内存page映射到子进程。这时，新分配的物理内存page只对子进程的地址空间可见，所以我们可以将相应的PTE设置成可读写，并且我们可以重新执行store指令。实际上，对于触发刚刚page fault的物理page，因为现在只对父进程可见，相应的PTE对于父进程也变成可读写的了。

所以现在，我们拷贝了一个page，将新的page映射到相应的用户地址空间，并重新执行用户指令。



概括一下就是：

父进程A，假如有4个页，A Fork出一个子进程B。如果不使用CopyOnWrite，那么Fork会分配四个物理页给B，把A的4个页都复制一遍。

但是Exec会随即丢弃掉刚刚才复制的四个物理页。这就造成了浪费。

优化的方案就是CopyOnWrite Fork。Fork只复制父进程A的页表，并且把**父进程和子进程**的页表里的PTE都标记为只读。这样子进程就相当于共享了父进程全部的物理页。

假如子进程B要对其中某一页做写入，那么对只读页的写入会触发Page Fault，我们处理Page Fault的方式是，把触发Page Fault的那一页复制一遍，映射到子进程的页表里去。新分配的页映射到子进程里去，并设置为可以写，而旧的页，在父进程的页表里，也设置为可以写入的。而其他的页，仍然是父子进程共享的，并且仍然还是只读的。然后再重新执行刚才触发Page Fault的指令

这样，后续执行Exec丢弃的物理页就少了许多，减少了浪费。并且拷贝页表比拷贝物理页快多了，所以程序的速度会快一点。

#### CopyOnWrite提问

> 学生提问：对于一些没有父进程的进程，比如系统启动的第一个进程，它会对于自己的PTE设置成只读的吗？还是设置成可读写的，然后在fork的时候再修改成只读的？
>
> Frans教授：这取决于你。实际上在lazy lab之后，会有一个copy-on-write lab。在这个lab中，你自己可以选择实现方式。当然最简单的方式就是将PTE设置成只读的，当你要写这些page时，你会得到一个page fault，之后你可以再按照上面的流程进行处理。
>
> 我个人认为，按照统一的行为逻辑来，是写代码最方便的。
>
> 
>
> 学生提问：因为我们经常会拷贝用户进程对应的page，内存硬件有没有实现特定的指令来完成拷贝，因为通常来说内存会有一些读写指令，但是因为我们现在有了从page a拷贝到page b的需求，会有相应的拷贝指令吗？
>
> Frans教授：x86有硬件指令可以用来拷贝一段内存。但是RISC-V并没有这样的指令。当然在一个高性能的实现中，所有这些读写操作都会流水线化，并且按照内存的带宽速度来运行。

Page Fault的触发条件是“向一个只读的页写入”，但是向一个只读的页写入，并不等价于“这一页是一个CopyOnWrite页”，因为它也有可能是一个Lazy Allocation页，二者都只是向一个只读页写入的特例情况之一。

那么内核在处理Page Fault的时候，如何区分，现在是一个copy-on-write fork的场景，而不是应用程序在向一个正常的只读地址写数据？

- 简短一点的回答就是，在PTE里设置相关的标志位，来表明这是一个copy-on-write page

> Frans教授：内核必须要能够识别这是一个copy-on-write场景。几乎所有的page table硬件都支持了这一点。对于PTE的标志位，我之前介绍过第0bit到第7bit，但是没有介绍最后两位RSW。这两位保留给supervisor software使用，supervisor softeware指的就是内核。内核可以随意使用这两个bit位。所以可以做的一件事情就是，将bit8标识为当前是一个copy-on-write page。
>
> 当内核在管理这些page table时，对于copy-on-write相关的page，内核可以设置相应的bit位，这样当发生page fault时，我们可以发现如果copy-on-write bit位设置了，我们就可以执行相应的操作了。否则的话，比如说lazy allocation，我们就做一些其他的处理操作。
>
> 在copy-on-write lab中，你们会使用RSW在PTE中设置一个copy-on-write标志位。

#### CopyOnWrite Lab细节（对物理页要进行引用计数了）

> 在copy-on-write lab中，还有个细节需要注意。目前在XV6中，除了trampoline page外，一个物理内存page只属于一个用户进程。trampoline page永远也不会释放，所以也不是什么大问题。但是对于这里的物理内存page，现在有多个用户进程或者说多个地址空间都指向了相同的物理内存page，举个例子，当父进程退出时我们需要更加的小心，因为我们要判断是否能立即释放相应的物理page。如果有子进程还在使用这些物理page，而内核又释放了这些物理page，我们将会出问题。那么现在释放内存page的依据是什么呢？
>
> 我们需要对于每一个物理内存page的引用进行计数，当我们释放虚拟page时，我们将物理内存page的引用数减1，如果引用数等于0，那么我们就能释放物理内存page。所以在copy-on-write lab中，你们需要引入一些额外的数据结构或者元数据信息来完成引用计数。

#### Demand Paging

需求分页（Demand Paging）是一种虚拟内存管理技术，用于有效地管理系统中的内存资源。它的核心思想是，不是一开始就将整个程序或进程所需的所有页面加载到物理内存中，而是根据需要逐页加载。



就是说，我们并不那么eager地把整个程序的数据段和代码段都加载到物理内存里去，而是再等等，直到应用程序实际需要这些指令的时候再加载内存里去。因为，第一，我们可能并不需要完整的二进制程序，二八定律决定了我们可能只需要其中的一小部分，如果把整个二进制程序都加载进去，那么可能其中大部分的指令根本就没被执行过，放在那里占用物理内存而已。



那么Demand Paging的具体实行方式是？

以exec为例，在虚拟地址空间中，我们为text和data分配好地址段，但是相应的PTE并不对应任何物理内存page。对于这些PTE，我们只需要将valid bit位设置为0即可。当CPU开始执行`exec`的指令时，位于地址0的指令是会触发第一个page fault的指令，因为我们还没有真正的加载内存。

对于这里触发的Page Fault，我们首先要得知这些Page是on-demand page。我们需要在某个地方记录了这些page对应的程序文件（应该就在触发Page Fault的Page的PTE里？），我们在page fault handler中需要从程序文件中读取page数据，加载到内存中；之后将内存page映射到page table；最后再重新执行指令。



在最坏的情况下，用户程序使用了text和data中的所有内容，那么我们将会在应用程序的每个page都收到一个page fault。但是如果我们幸运的话，用户程序并没有使用所有的text区域或者data区域，那么我们一方面可以节省一些物理内存，另一方面我们可以让exec运行的更快（因为不需要为整个程序分配内存）



这是Demand Paging的一个方面

Demand Paging的另外一个方面是，假设内存已经耗尽了或者说OOM了，这个时候如果得到了一个page fault，需要从文件系统拷贝中拷贝一些内容到内存中，但这时你又没有任何可用的物理内存page，这其实回到了之前的一个问题：在lazy allocation中，如果内存耗尽了该如何办？

选择是回收page（evict page）。比如说将部分内存page中的内容写回到文件系统再回收page。一旦你回收并释放了page，那么你就有了一个新的空闲的page，你可以使用这个刚刚空闲出来的page，分配给刚刚的page fault handler，再重新执行指令。



也就是Swap机制。利用页面置换来给On-Demand Page腾出物理页空间。

优先选择非脏的页来替换，因为这样可以不用写回硬盘，直接覆盖原有内容即可。

选择的策略是LRU。利用PTE的Access Bit和Dirty Bit我们可以实现一个LRU。



时钟算法：一种基于模拟LRU算法的页面置换算法。

- 基本思想：每当需要一个物理页的时候，时钟指针沿着顺时针方向依次检查表盘上的每一页。
- 对于检查到的这一页：
  - 如果它的Access Bit是1，那么我们清零这个Bit，同时指针指向下一页。
  - 如果它的Access Bit是0，那么我们选择这一页用于置换。检查其Dirty Bit，看是否需要写回硬盘。然后替换
  - 运气不好的情况下，指针转一整圈，回到最初的起点，这个时候表盘上的每一页的Access Bit都被清零了。所以起点那一页也是，我们选择起点的那一页。

####  Memory Mapped Files

将文件的内容映射到进程的地址空间，使得文件的内容可以被当作内存中的数据来访问和操作。

memory mapped files的核心思想是，将完整或者部分文件加载到内存中，这样就可以通过内存地址相关的load或者store指令来操纵文件。

`mmap`系统调用：

```c
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
/*从文件描述符对应的文件的偏移量的位置开始，映射长度为len的内容到虚拟内存地址VA，同时我们需要加上一些保护，比如只读或者读写。*/
```

假定内核实现`mmap`采用的是eager的方式（实际上是Lazy的，但是Lazy本质上也是把eager要做的操作延后了而已，所以我们这里讲eager的）。

内核会从文件的offset位置开始，将数据拷贝到内存，设置好PTE指向物理内存的位置。之后应用程序就可以使用load或者store指令来修改内存中对应的文件内容。

当完成操作之后，会有一个对应的unmap系统调用，参数是虚拟地址（VA），长度（len），来表明应用程序已经完成了对文件的操作，在unmap时间点，我们需要将dirty block写回到文件中。我们可以很容易的找到哪些block是dirty的，因为它们在PTE中的dirty bit为1。



当然，实际上是Lazy的方式实现的，你不会立即将文件内容拷贝到内存中，而是先记录一下这个PTE属于这个文件描述符。相应的信息通常在VMA结构体中保存，VMA全称是Virtual Memory Area。



## Lecture 9

> 在计算机系统中，中断是一种机制，用于暂停 CPU 正在执行的当前任务，转而去处理一些紧急事件或外部请求。以下是一些可能触发中断的情况：
>
> 1. **硬件中断**：来自计算机硬件设备的信号，如时钟中断（定时器中断）、I/O 设备中断（键盘、鼠标、磁盘）、网络设备中断等。
> 2. **异常**：也称为陷阱或故障，是非预期的程序行为，可能导致程序中断执行。例如，访问无效内存地址、除以零等。
> 3. **系统调用**：当应用程序需要访问操作系统提供的服务或资源时，会触发系统调用，这通常通过软中断（software interrupt）实现。
> 4. **外部中断**：来自外部设备、外部事件或其他处理器的信号。例如，通过中断控制器（如 PIC 或 APIC）触发的中断，或者外部硬件设备的信号。
> 5. **软中断**：由程序内部触发的中断，允许程序通过软件中断指令来显式地触发中断。这可以用于调度、进程间通信等。
> 6. **NMI（非屏蔽中断）**：这是一种特殊类型的中断，通常用于处理严重问题，例如硬件故障或系统崩溃。

### 阅读Chapter 5 Interrupts and device drivers

驱动程序是操作系统中管理特定设备的代码：它配置设备硬件，告诉设备执行操作，处理产生的中断，并与可能正在等待设备I/O的进程进行交互。

需要OS的注意力的设备一般会生成一个中断，内核陷阱处理代码识别设备何时发出中断，并调用驱动程序的中断处理程序（`devintr`）。



许多驱动程序在两个上下文中执行代码，分别是运行在一个内核线程的上半部分（*top half*），和运行在中断时间的下半部分（*bottom half*）。

上半部分通过想让设备执行I/O的系统调用（比如read和write）来调用，这部分的代码会请求硬件执行一个操作（如读取磁盘块），然后等待操作完成。

最终设备完成操作，产生一个中断，驱动的中断处理程序就是下半部分。下半部分确定已完成的操作是什么，并适当唤醒处于等待的进程，告诉硬件执行下一个操作。

#### 5.1 Code: Console input

以控制台驱动程序（console Driver）为例。console的驱动通过连接到RISC-V的UART串行端口接收人类手打的字符。控制台驱动程序一次积累一行输入，并处理特殊的输入字符，例如退格。用户进程（例如shell）使用read系统调用从控制台获取输入行。

驱动程序所使用的UART硬件是由QEMU模拟的16550芯片，而QEMU模拟的UART则连接到了键盘和显示屏。

UART硬件对软件表现为一系列内存映射的寄存器（ a set of *memory-mapped* control registers），也就是说有一部分的物理地址对应的并不是RAM，而是UART设备。Load和Store指令在这些地址上执行的时候实际上是与设备交互而不是RAM。

UART的起始地址是0x10000000,，定义为`UART0 (kernel/memlayout.h:21)`。

UART的控制寄存器的宽度都是一字节。他们的地址相对于UART0的偏移量见`(kernel/uart.c:22)`。



Xv6的`main`调用 `consoleinit (kernel/console.c:184)`来初始化UART硬件。`consoleinit`将UART配置为当收到一字节的输入的时候，发出一个 receive interrupt，每当完成一字节的输出的时候，发出一个 transmit complete interrupt.



Xv6的Shell借助由`init.c`文件描述符来从控制台读取，对read的调用会穿过内核到达`consoleread`，`consoleread`等待输入到达（通过中断）并被缓存在缓冲区`cons.buf`，将输入复制到用户空间，并在一整行到达之后返回用户进程。如果用户还没输入满一行，那么进行读取的进程会在Sleep等待（啥意思？）

> If the user hasn’t typed a full line yet, any reading processes will wait in the sleep call (kernel/console.c:98) (Chapter 7 explains the details of sleep)

当用户输入一个字符，UART硬件会让RISC-V生成一个中断，中断激活xv6的陷阱处理程序，陷阱处理程序调用`devintr`，`devintr`查看`scause`寄存器发现这是一个来自外部设备的中断，然后它询问PLIC告知哪个设备中断了，如果是UART，`devintr`调用`uartintr`。

`uartintr`从UART硬件读取等待的输入字符并递交给`consoleintr`，它并不会等待新的字符，因为新的字符输入会产生新的中断。`consoleintr`的工作则是将输入缓冲到`cons.buf`，直到累计的输入满一行。当新的一行到达，`consoleintr`唤醒一个等待中的`consoleread`（如果有）。

一旦醒来，`consoleread`就会在`cons.buf`发现一整行的输入，将它复制到用户空间，然后返回。

#### 5.2 Code: Console output

一个对与控制台连接的文件描述符的`write`系统调用，最终会到达`uartputc(kernel/uart.c:87)`。设备驱动维护了一个缓冲区`uart_tx_buf`，这样写入的进程不需要等待UART完成传输，而是`uartputc`将字符追加到缓冲区，然后调用`uartstart`开始设备传输，然后返回。`uartputc`会等待的唯一情况只有缓冲区满的情况。

每次UART完成一个字节的发生，它产生一个中断，`uartintr`调用`uartstart`，`uartstart`检查设备真正完成了传输，并传递给设备下一个缓存的字符。所以假如一个进程写入多个字节，那么第一个字节是由`uartputc`调用`uartstart`发送的，剩下的字节则是由`uartintr`调用`uartstart`发送的。



一个通用的模式是通过缓冲区和中断的方式解耦进程的活动和设备的活动。即便没有进程等待读取，控制台驱动也可以处理输入，进程不需要等待设备也可以直接发送输出。这样可以提高性能

#### 5.3 Concurrency in drivers

`consoleintr`和`consoleread`都加了锁，这是为了并发安全。

有三种并发的风险：

- 不同CPU上的两个进程同时调用`consoleread`
- 硬件可能会在CPU已经进入`consoleread`的时候要求CPU发出一个UART中断
- 硬件在`consoleread`正在执行的时候在另外一个CPU上发出另外一个中断。

当一个进程等待设备输入时，中断可能在不同的进程（甚至没有进程运行）时到达，因此中断处理程序不能依赖于当前被中断的进程或代码的状态。

#### 5.4 Timer interrupts

Xv6使用时钟中断来维持它的时钟以及在计算密集型进程之间切换。

时钟中断来自与每个RISC-V CPU绑定的时钟硬件上，Xv6编码这个时钟适当它周期性地中断CPU。

RISC-V要求时钟中断在机器模式下运行，并且不使用分页，并且使用单独的一系列控制寄存器。所以在机器模式下运行普通的内核代码是不现实的。结果就是，Xv6对时钟中断的处理完全独立于上述处理Trap的机制



在`start.c`中机器模式下执行的代码，先于main，为接收时钟中断做好设置。一部分工作是编程好CLINT(core-local interruptor），让它在特定的延迟之后产生中断。另外一部分工作是设置好一个类似陷阱帧的暂存区，以帮助时钟中断处理程序暂存好寄存器和CLINT寄存器的地址。最终，`start`设置`mtvec`寄存器指向`timervec`，并启用时钟中断。



一个时钟中断可以出现在任何时候，无论当前是用户代码还是内核正在执行，内核也没有办法在关键操作的时候关闭时钟中断。所以时钟中断处理程序必须保证以一种不会干扰到被中断的内核代码的方式干活。基本的策略就是，让处理程序请求RISC-V抛出一个软中断，然后立即返回。RISC-V将软中断提交给内核，以普通的Trap处理机制处理，并且允许内核禁用它们。

问题：所以这个策略到底是什么逻辑？

 机器模式下的时钟中断向量`timevecc (kernel/kernelvec.S:93)`将寄存器的值保存在暂存区，告诉CLINT什么时候产生下一次时钟中断，叫RISC-V产生一个软中断，恢复寄存器，然后返回。

#### 5.5 Real world

Xv6允许在内核中执行时以及在执行用户程序时进行设备和定时器中断。定时器中断迫使定时器中断处理程序进行线程切换（调用`yield`），即使在内核中执行也是如此。



UART使用的是编程的I/O。但是这样性能不高，更高性能的是DMA方式，DMA硬件直接把输入的数据写入内存，直接从内存读取输出的数据。DMA设备的驱动会将数据直接写入RAM，然后写入控制寄存器以告知设备去处理准备好的数据。



由于中断的开销很大，所以一些高速设备，使用一些技巧以减少中断的需求：

- 为一批量的输入或输出请求，发出一个中断，而不是为每个请求发一次中断
- 设备驱动完全禁用中断，周期性地检查设备（其实就是轮循）是否需要注意力。轮循对于运行速度很快的设备来说是很好的，但是假如设备大部分时候是闲置状态，就是浪费CPU时间。
- 有些设备依据当前设备的负载情况，动态地在轮循和中断之间切换

UART驱动程序先把输入数据复制到内核的缓冲区，然后从内核的缓冲区复制到用户的缓冲区。数据率低的时候可以这样，但是数据率高的时候这种双重复制会极大降低性能。所以有些OS直接在设备硬件和用户空间缓冲区之间传输数据。

### 阅读源码

要求阅读的有：

- `kernel/kernelvec.S`：负责处理来自内核的Trap。这里应该需要看一下Timevec
- `kernel/plic.c`：
- `kernel/console.c`：控制台驱动
-  `kernel/uart.c` ：QEMU模拟的UART芯片。连接键盘和显示屏
- `kernel/printf.c`：

因为这一部分读的时候有点乱，所以，按照每一小节提到的顺序来读吧。

#### 5.1提到的

UART各个寄存器相对于UART地址UART0的偏移量见(kernel/uart.c:22)，所以先读(kernel/uart.c:22)。

大概包含下面几个寄存器，以及一些用于定义状态的宏

- RHR：*receive holding register*，存放输入的字节
- THR：*transmit holding register*，存放输出的字节
- IER：*interrupt enable register*
- FCR：*FIFO control register*
- ISR：*interrupt status register*
- LCR：*line control register*
- LSR：*line status register*



Xv6的main调用了consoleinit，consoleinit调用了uartinit。

uartinit主要做了下面几件事：

1. 关中断。
2. 设置波特率。设置字长为1字节。重置FIFO控制寄存器
3. 开中断（这样收到输入和完成输出时，发出中断），初始化uart的锁（用于uart的并发控制）。

cons结构体是console结构体，包含一个控制台的自旋锁，一个缓冲区`cons.buf`，几个索引变量记录当前读写位置之类的。

```c
struct {
  struct spinlock lock;
  
  // input
#define INPUT_BUF 128
  char buf[INPUT_BUF];
  uint r;  // Read index。最好理解成读取了多少个字符
  uint w;  // Write index。最好理解成外部输入了多少个字符？
  uint e;  // Edit index
} cons;

void
consoleinit(void)
{
  initlock(&cons.lock, "cons");

  uartinit();

  // connect read and write system calls
  // to consoleread and consolewrite.
    //将控制台设备的读和写系统调用连接到consoleread和consolewrite函数
  devsw[CONSOLE].read = consoleread;
  devsw[CONSOLE].write = consolewrite;
}
```

另外提到的就是`consoleread`：对于控制台的读取（`read()`）最终会到达`consoleread`。

```c
#define C(x)  ((x)-'@')  // Control-x。这个宏用于定义各种特殊的输入，比如Ctrl-x + D，指代各类char

// user read()s from the console go here.
// copy (up to) a whole input line to dst.
// user_dist indicates whether dst is a user
// or kernel address.
int
consoleintr(int user_dst, uint64 dst, int n)
{
  uint target;
  int c;
  char cbuf;

  target = n;//target用于保存n的初始值，也就是预期要复制的目标字节数?
  acquire(&cons.lock);
  while(n > 0){
    // wait until interrupt handler has put some
    // input into cons.buffer.
    while(cons.r == cons.w){//这里使用一个轮循加睡眠来等待中断处理程序将输入放到buffer。
        //cons.r == cons.w意味着外部输入的字符数与已经读取的字符数相等，所以进入睡眠?
        //输入是被consoleintr放进去的，consoleintr并不是放一个字符就唤醒consoleintr，
        //很有可能是放进去一行再唤醒，只不过consoleread是一个字符一个字符复制的
      if(myproc()->killed){
        release(&cons.lock);
        return -1;
      }
      sleep(&cons.r, &cons.lock);
    }

    c = cons.buf[cons.r++ % INPUT_BUF]; //c是新放入的输入字符，Uart是一个字节一个字节传输的，所以刚好就是一个char。

    if(c == C('D')){  // end-of-file。其实是Ctrl-x D
      if(n < target){
        // Save ^D for next time, to make sure
        // caller gets a 0-byte result.
        cons.r--;//EOF不计入读取的字符个数，所以这里自减1（上面++了一次）
      }
      break;
    }

    // copy the input byte to the user-space buffer.
    cbuf = c;
    if(either_copyout(user_dst, dst, &cbuf, 1) == -1)//如果复制失败就break
      break;

    dst++;//将一个字符复制到dst之后，dst++让dst指向下一个位置。
    --n;//每复制一个字符，n--。n可以理解成剩余需要复制的字节数。

    if(c == '\n'){
      // a whole line has arrived, return to
      // the user-level read().
      break;
    }
  }
  release(&cons.lock);

  return target - n;//返回实际上读取的字节数?
}
```

`consoleread`调用的`either_copyout(proc.c)`：

```c
// Copy to either a user address, or kernel address,
// depending on usr_dst.
// Returns 0 on success, -1 on error.
int
either_copyout(int user_dst, uint64 dst, void *src, uint64 len)
{
  struct proc *p = myproc();//获取当前进程
  if(user_dst){//如果是用户空间的地址，调用copyout来从内核空间复制到用户空间
    return copyout(p->pagetable, dst, src, len);
  } else {//否则直接在内核空间里复制就行了
    memmove((char *)dst, src, len);
    return 0;
  }
}
```

然后是提到的`uartintr`。这个函数在`devintr`被调用，当`devintr`查看scause寄存器，发现是外部设备中断时，向PLIC询问是哪个设备，如果是UART，就调用`uartintr`。这里先摘取`devintr`的一部分代码：

```c
uint64 scause = r_scause();

  if((scause & 0x8000000000000000L) &&
     (scause & 0xff) == 9){
    // this is a supervisor external interrupt, via PLIC.

    // irq indicates which device interrupted.
    int irq = plic_claim();

    if(irq == UART0_IRQ){
      uartintr();
    } 
```

然后是`uartintr`，`uartintr`调用了`consoleintr`，`consoleintr`的工作是在cons.buffer累计缓冲一行输入，然后再唤醒`consoleread`做处理（把输入从buffer复制到别的地址去，比如用户空间的缓冲区）。`consoleread`一旦被唤醒，就能在`cons.buffer`看见一整行的输入。

`uartintr`则调用`uartgetc`从UART的RHR寄存器读取一个字节的输入。寄存器的读取则是与之前的类似，都是一个定义好的宏。

所以下一个阅读的是`consoleintr`。

```c
// read one input character from the UART.
// return -1 if none is waiting.
int
uartgetc(void)
{
  if(ReadReg(LSR) & 0x01){
    // input data is ready.
    return ReadReg(RHR); //返回读取到的RHR寄存器的内容。RHR寄存器存放输入的一个字节
  } else {
    return -1;
  }
}

// handle a uart interrupt, raised because input has
// arrived, or the uart is ready for more output, or
// both. called from trap.c.
void
uartintr(void)
{
  // read and process incoming characters.
  while(1){
    int c = uartgetc();
    if(c == -1)
      break;//持续不断读取下一个输入字节，并传递给consoleintr，直到没有输入字节为止
    consoleintr(c);
  }

  // send buffered characters.
  acquire(&uart_tx_lock);
  uartstart();
  release(&uart_tx_lock);
}
```

`consoleintr`负责将输入累计缓冲在`cons.buffer`，满一行的时候唤醒`consoleread`去做处理。

```c
// the console input interrupt handler.
// uartintr() calls this for input character.
// do erase/kill processing, append to cons.buf,
// wake up consoleread() if a whole line has arrived.
//
void
consoleintr(int c)
{
  acquire(&cons.lock);

  switch(c){
  case C('P'):  // Print process list. 特殊字符Ctrl-x + P
    procdump();
    break;
  case C('U'):  // Kill line. 特殊字符Ctrl-x + U
    while(cons.e != cons.w &&
          cons.buf[(cons.e-1) % INPUT_BUF] != '\n'){
      cons.e--;
      consputc(BACKSPACE);
    }
    break;
  case C('H'): // Backspace
  case '\x7f':
    if(cons.e != cons.w){
      cons.e--;
      consputc(BACKSPACE);
    }
    break;
  default:
    if(c != 0 && cons.e-cons.r < INPUT_BUF){
      c = (c == '\r') ? '\n' : c;

      // echo back to the user.这里应该是回显到显示屏？
      consputc(c);

      // store for consumption by consoleread().把C放入缓冲区
      cons.buf[cons.e++ % INPUT_BUF] = c;

      if(c == '\n' || c == C('D') || cons.e == cons.r+INPUT_BUF){//如果累计满一行，就唤醒consoleread()
        // wake up consoleread() if a whole line (or end-of-file)
        // has arrived.
        cons.w = cons.e;
        wakeup(&cons.r);
      }
    }
    break;
  }
  
  release(&cons.lock);
}
```

由于这一节并没有显式提到`uartstart`，把他们放在下一节

#### 5.2 Code: Console output提到的

对于控制台的write系统调用最终会到达`uartputc`，`uartputc`维护了一个缓冲区(uart_tx_buf)，进程只要往里面写东西就行了，不需要等UART发送完。`uartputc`代表进程往缓冲区里追加东西，然后调用`uartstart`开始传输。当缓冲区满的时候，`uartputc`等待。

`uartstart`所做的事很简单，就是在死循环里，不断把下一个字符放到寄存器THR里面。

```c
// the transmit output buffer.
struct spinlock uart_tx_lock;//这其实就是一个生产者消费者问题，锁用于同步和互斥，而两个int变量，一个是从哪里读，一个是写到哪里去
#define UART_TX_BUF_SIZE 32
char uart_tx_buf[UART_TX_BUF_SIZE];
int uart_tx_w; // write next to uart_tx_buf[uart_tx_w++]
int uart_tx_r; // read next from uart_tx_buf[uar_tx_r++]

// add a character to the output buffer and tell the
// UART to start sending if it isn't already.
// blocks if the output buffer is full.
// because it may block, it can't be called
// from interrupts; it's only suitable for use
// by write().
void
uartputc(int c)
{
  acquire(&uart_tx_lock);

  if(panicked){
    for(;;)
      ;
  }

  while(1){
    if(((uart_tx_w + 1) % UART_TX_BUF_SIZE) == uart_tx_r){
      // buffer is full.
      // wait for uartstart() to open up space in the buffer.
      sleep(&uart_tx_r, &uart_tx_lock);
    } else {
      uart_tx_buf[uart_tx_w] = c;//写入字符c到缓冲区
      uart_tx_w = (uart_tx_w + 1) % UART_TX_BUF_SIZE;//缓冲区看做是一个回环，所以更新的时候取余，而不是只uart_tx_w++。
      uartstart();//开始传输
      release(&uart_tx_lock);
      return;
    }
  }
}
// if the UART is idle, and a character is waiting
// in the transmit buffer, send it.
// caller must hold uart_tx_lock.
//注意，调用者必须持有uart_tx_lock，所以uartstart没有Acquire的操作。
// called from both the top- and bottom-half.
void
uartstart()
{
  while(1){
    if(uart_tx_w == uart_tx_r){
      // transmit buffer is empty.
      return;
    }
    
    if((ReadReg(LSR) & LSR_TX_IDLE) == 0){
      // the UART transmit holding register is full,
      // so we cannot give it another byte.
      // it will interrupt when it's ready for a new byte.
      return;
    }
    
    int c = uart_tx_buf[uart_tx_r];//读取出下一个要输出的字符c
    uart_tx_r = (uart_tx_r + 1) % UART_TX_BUF_SIZE;//更改下一个待读取的位置
    
    // maybe uartputc() is waiting for space in the buffer.
    wakeup(&uart_tx_r);//唤醒可能等待空闲位置的uartputc()
    
    WriteReg(THR, c);//把字符写入存放输出的寄存器THR
  }
}
```

我觉得有必要对`userintr`做一些进一步的说明：它其实做了两件事，第一件事是在死循环中，持续不断地从输入字符寄存器RHR读取一个字符，然后传递给`consoleintr`，直到没有更多输入读取为止，break跳出死循环。然后开始做第二件事：调用`uartstart`开始传输输出的字符。

所以对`uartintr`的调用实际上是既处理了输入，也处理了输出，所以注释里说"handle a uart interrupt"，其实是既处理了输入引发的中断，也处理了输出引发的中断。说白了就是，`uartintr`不管三七二十一，直接处理一遍输入一遍输出，两件事一起做了就完事了，管你的中断到底是哪一种原因引发的。

```c
// handle a uart interrupt, raised because input has
// arrived, or the uart is ready for more output, or
// both. called from trap.c.
void
uartintr(void)
{
  // read and process incoming characters.
  while(1){
    int c = uartgetc();
    if(c == -1)
      break;//持续不断读取下一个输入字节，并传递给consoleintr，直到没有输入字节为止(consoleintr则把字符放入cons.buffer)
    consoleintr(c);
  }

  // send buffered characters.将缓冲在uart_tx_buf的字符逐个输出到外界。注意一下输入和输出并没有共用缓冲区，输入用的是cons.buffer
  acquire(&uart_tx_lock);
  uartstart();
  release(&uart_tx_lock);
}
```

#### 5.4

阅读一下`kernelvec.S`。其实`kernelvec`这个函数和之前的uservec很类似，只不过uservec是把寄存器的值都放在了Trapframe，而kernelvec把寄存器的值都保存在内核的栈里面。

kernelvec其实就是一个保持寄存器到内核栈==>调用kerneltrap() ==>恢复寄存器的值==>sret返回。

不过既然调用了kerneltrap，那就把kerneltrap也一并摆上来。其实kerneltrap做的事和Usertrap大体是相同的，从`scause`寄存器读取Trap的原因，然后分情况调用不同的处理程序就好。只有中断和异常两类，异常直接Panic

```c
// interrupts and exceptions from kernel code go here via kernelvec,
// on whatever the current kernel stack is.
void 
kerneltrap()
{
  int which_dev = 0;
  uint64 sepc = r_sepc();
  uint64 sstatus = r_sstatus();
  uint64 scause = r_scause();
  
  if((sstatus & SSTATUS_SPP) == 0)
    panic("kerneltrap: not from supervisor mode");
  if(intr_get() != 0)
    panic("kerneltrap: interrupts enabled");

  if((which_dev = devintr()) == 0){
    printf("scause %p\n", scause);
    printf("sepc=%p stval=%p\n", r_sepc(), r_stval());
    panic("kerneltrap");
  }

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2 && myproc() != 0 && myproc()->state == RUNNING)
    yield();

  // the yield() may have caused some traps to occur,
  // so restore trap registers for use by kernelvec.S's sepc instruction.
  w_sepc(sepc);
  w_sstatus(sstatus);
}
```



```c
        # interrupts and exceptions while in supervisor
        # mode come here.
        #
        # push all registers, call kerneltrap(), restore, return.
        #
.globl kerneltrap
.globl kernelvec
.align 4
kernelvec:
        // make room to save registers.
        addi sp, sp, -256

        // save the registers.
        sd ra, 0(sp)
        sd sp, 8(sp)
        sd gp, 16(sp)
        sd tp, 24(sp)
        sd t0, 32(sp)
        sd t1, 40(sp)
        sd t2, 48(sp)
        sd s0, 56(sp)
        sd s1, 64(sp)
        sd a0, 72(sp)
        sd a1, 80(sp)
        sd a2, 88(sp)
        sd a3, 96(sp)
        sd a4, 104(sp)
        sd a5, 112(sp)
        sd a6, 120(sp)
        sd a7, 128(sp)
        sd s2, 136(sp)
        sd s3, 144(sp)
        sd s4, 152(sp)
        sd s5, 160(sp)
        sd s6, 168(sp)
        sd s7, 176(sp)
        sd s8, 184(sp)
        sd s9, 192(sp)
        sd s10, 200(sp)
        sd s11, 208(sp)
        sd t3, 216(sp)
        sd t4, 224(sp)
        sd t5, 232(sp)
        sd t6, 240(sp)

	// call the C trap handler in trap.c
        call kerneltrap

        // restore registers.
        ld ra, 0(sp)
        ld sp, 8(sp)
        ld gp, 16(sp)
        // not this, in case we moved CPUs: ld tp, 24(sp)
        ld t0, 32(sp)
        ld t1, 40(sp)
        ld t2, 48(sp)
        ld s0, 56(sp)
        ld s1, 64(sp)
        ld a0, 72(sp)
        ld a1, 80(sp)
        ld a2, 88(sp)
        ld a3, 96(sp)
        ld a4, 104(sp)
        ld a5, 112(sp)
        ld a6, 120(sp)
        ld a7, 128(sp)
        ld s2, 136(sp)
        ld s3, 144(sp)
        ld s4, 152(sp)
        ld s5, 160(sp)
        ld s6, 168(sp)
        ld s7, 176(sp)
        ld s8, 184(sp)
        ld s9, 192(sp)
        ld s10, 200(sp)
        ld s11, 208(sp)
        ld t3, 216(sp)
        ld t4, 224(sp)
        ld t5, 232(sp)
        ld t6, 240(sp)

        addi sp, sp, 256

        // return to whatever we were doing in the kernel.
        sret
```

然后是这一小节提到的`timevec`：

#### 单独阅读一下plic.c

> Platform Level Interrupt Controller (PLIC)是一种硬件设备，用于处理中断控制和分发，特别是在多处理器系统中。它是为了支持RISC-V架构而设计的，但也可在其他体系结构中使用。
>
> PLIC的主要功能是协调和分发中断请求，以确保它们被正确地传递给目标处理器核心。在多处理器系统中，可能存在多个处理器核心和多个外部设备，这些设备可能会发出中断请求。PLIC负责将中断请求路由到适当的处理器核心。

`plic.c`应该是在`trap.c`的`deviceintr`里引用了其中的方法，所以要求阅读一下。这里先把`deviceintr`中引用了下面的两个函数，第一个是用于查看是哪个设备发出的外部中断，第二个是用于告诉PLIC，刚才那个设备发出的中断已经处理完成了。

```c
// ask the PLIC what interrupt we should serve.
int
plic_claim(void)
{
  int hart = cpuid();
  int irq = *(uint32*)PLIC_SCLAIM(hart);
  return irq;
}

// tell the PLIC we've served this IRQ.
void
plic_complete(int irq)
{
  int hart = cpuid();
  *(uint32*)PLIC_SCLAIM(hart) = irq;
}
```

#### printf.c

`printf.c`大部分的函数都调用了`console.c`中的`consputc`。`consputc`用于把一个字符发送给uart（实际上就是把那个字符写到THR寄存器去）。

`pushoff`不知道是干嘛的，在`locking.c`里面，先放着吧。`uartputc_sync`其实就做了一件事，把字符写入THR寄存器，只不过在等待THR变空的时候是采用轮循的方式。

```c
// send one character to the uart.
// called by printf, and to echo input characters,
// but not from write().
//
void
consputc(int c)
{
  if(c == BACKSPACE){//如果是退格，那就用空格符去覆盖前一个字符，这样就表现为退格
    // if the user typed backspace, overwrite with a space.
    uartputc_sync('\b'); uartputc_sync(' '); uartputc_sync('\b');
  } else {
    uartputc_sync(c);
  }
}

// alternate version of uartputc() that doesn't 
// use interrupts, for use by kernel printf() and
// to echo characters. it spins waiting for the uart's
// output register to be empty.
void
uartputc_sync(int c)
{
  push_off();

  if(panicked){
    for(;;)
      ;
  }

  // wait for Transmit Holding Empty to be set in LSR.
  while((ReadReg(LSR) & LSR_TX_IDLE) == 0)
    ;
  WriteReg(THR, c);

  pop_off();
}
```

然后稍微解释一下printf的基本思想：

直接解释for循环吧：

- 如果c不是百分号，说明c就是一个普通的字符，那就直接把c传递给uart，由uart负责显示到屏幕上去，直接下一个循环。
- 否则c是百分号，那就说明c的下一个字符是一个格式化字符（%d、%s之类的，c是前面的%），那就先让c指向下一个字符，然后对c进行Switch Case：
  - 

```c
void
printf(char *fmt, ...)
{
  va_list ap;
  int i, c, locking;
  char *s;

  locking = pr.locking;
  if(locking)
    acquire(&pr.lock);

  if (fmt == 0)
    panic("null fmt");

  va_start(ap, fmt);
  for(i = 0; (c = fmt[i] & 0xff) != 0; i++){
    if(c != '%'){
      consputc(c);
      continue;
    }
    c = fmt[++i] & 0xff;
    if(c == 0)//到达结尾，跳出循环
      break;
    switch(c){
    case 'd':
      printint(va_arg(ap, int), 10, 1);
      break;
    case 'x':
      printint(va_arg(ap, int), 16, 1);
      break;
    case 'p':
      printptr(va_arg(ap, uint64));
      break;
    case 's':
      if((s = va_arg(ap, char*)) == 0)
        s = "(null)";
      for(; *s; s++)//把%s对应的字符串里面的字符一个个地传递给uart，由uart显示在屏幕上。
        consputc(*s);
      break;
    case '%':
      consputc('%');
      break;
    default:
      // Print unknown % sequence to draw attention.
      consputc('%');
      consputc(c);
      break;
    }
  }

  if(locking)
    release(&pr.lock);
}
```

### Lecture

不管是系统调用，还是Page Fault，还是中断，使用的都是相同的。

但是中断与系统调用相比又存在三个小的差别：

- 中断的产生与当前CPU正在执行的程序是没有关系的，而系统调用则发生在当前进程的上下文。
- 生成中断的设备和CPU之间是并行的在运行的。网卡自己处理收到的包，然后在合适的时间发出一个中断，这些行为的同时CPU也在运行。
- 需要对外部设备进行编程。



这一节课回答两个问题：

- console中的提示符“$ ”是如何显示出来的
- 如果你在键盘输入“ls”，这些字符是怎么最终在console中显示出来的。

这二者还不太一样，因为$是Shell的输出，而ls是用户敲击键盘输入之后再显示的。

#### 有关设备驱动

大部分的驱动都分为两个部分：

- bottom half：底部通常是中断处理程序，当一个中断到达CPU的时候，CPU就会调用这个中断的中断处理程序。中断处理程序并不与特定进程相关，它就单纯负责处理中断。
- top half：顶部是对外提供的接口，用户进程或者是内核的其他部分可以调用的接口。

将底部和顶部之间关联在一起（也是使得顶部和底部解耦的东西），是一个Buffer（缓冲区），top部分的代码会从队列中读写数据，而Interrupt handler（bottom部分）同时也会向队列中读写数据。这里的队列可以将并行运行的设备和CPU解耦开来。



这里还需要补充一个知识点：Memory Mapped IO（内存映射的IO）和Memory Mapped File（内存映射的文件）是两码事

前者指的是把外部设备的寄存器和状态映射到物理内存的地址空间中，这样读写外部设备的寄存器就可以表现得像在读写内存。无需使用特定的 I/O 指令，就可以通过读取或写入这些内存地址来与设备进行通信。

在xv6下，外部设备的地址是被映射到了`0x800000`以下的部分。OS只需要普通的Load和Store指令，就可以读写外部设备的寄存器。

#### 有关UART的提问

> 学生提问：如果你写入数据到Transmit Holding Register，然后再次写入，那么前一个数据不会被覆盖掉吗？
>
> Frans教授：这是我们需要注意的一件事情。我们通过load将数据写入到这个寄存器中，之后UART芯片会通过串口线将这个Byte送出。当完成了发送，UART会生成一个中断给内核，这个时候才能再次写入下一个数据。所以内核和设备之间需要遵守一些协议才能确保一切工作正常。上图中的UART芯片会有一个容量是16的FIFO，但是你还是要小心，因为如果阻塞了16个Byte之后再次写入还是会造成数据覆盖。

#### $和ls是怎么显示在屏幕上的？

“\$ ”和“ls”还不太一样，“$ ”是Shell程序的输出，而“ls”是用户通过键盘输入之后再显示出来的。

对于“$ ”来说，实际上就是设备会将字符传输给UART的寄存器，UART之后会在发送完字符之后产生一个中断。而线路的另一端是另外一个UART，它连接了Console，将字符显示到屏幕上。

对于“ls”，键盘连接了UART的输入线路，当在键盘上按下一个按键，UART芯片会将按键字符通过串口线发送到另一端的UART芯片。另一端的UART芯片先将数据bit合并成一个Byte，之后再产生一个中断，并告诉处理器这里有一个来自于键盘的字符。之后Interrupt handler会处理来自于UART的字符。



#### 中断的改进

对于（如果使用中断）极高频率产生中断的外部设备，使用轮循可能是更优解。这样可以节省高频处理中断的开销。

> 如果你有一个高性能的设备，例如你有一个千兆网卡，这个网卡收到了大量的小包，网卡每秒可以生成1.5Mpps，这意味着每一个微秒，CPU都需要处理一个中断，这就超过了CPU的处理能力。那么当网卡收到大量包，并且处理器不能处理这么多中断的时候该怎么办呢？
>
> 这里的解决方法就是使用polling。除了依赖Interrupt，CPU可以一直读取外设的控制寄存器，来检查是否有数据。对于UART来说，我们可以一直读取RHR寄存器，来检查是否有数据。现在，CPU不停的在轮询设备，直到设备有了数据。

其实也可以说还有另外一种解法，那就是批处理，先累计一定的Packet，然后每一批Packet发出一个中断，而不是到达一个Packet就发出一个中断。



# Lecture 10

### 阅读Chapter 6

> The word *concurrency* refers to situations in which multiple instruction streams are interleaved, due to multiprocessor parallelism, thread switching, or interrupts

锁的坏处是：他们会降低性能，因为锁会将并发的操作串行化。

这章讲述为什么需要锁，xv6是怎么实现锁的，Xv6是怎么使用锁的。

#### 6.1 Race conditions

> A race condition is a situation in which a memory location is accessed concurrently, and at least one access is a write

> When we say that a lock protects data, we really mean that the lock protects some collection of invariants that apply to the data. Invariants are properties of data structures that are maintained across operations. Typically, an operation’s correct behavior depends on the invariants being true when the operation begins. The operation may temporarily violate the invariants but must reestablish them before finishing. 
>
> The race condition we examined above happened **because a second CPU executed code that depended on the list invariants while they were (temporarily) violated.**
>
>  Proper use of a lock ensures that only one CPU at a time can operate on the data structure in the critical section, so that no CPU will execute a data structure operation when the data structure’s invariants do not hold

我们可以把锁看做是将涉及临界区的并发操作串行化了，一次只执行一个，所以数据结构的不变性得以保持。也可以把锁看做是保证了操作的原子性，这样后一个操作永远不会看到前一个操作的部分完成的更新，永远只能看见前一个操作的完整完成的更新。

> You can think of a lock as *serializing* concurrent critical sections so that they run one at a time, and thus preserve invariants (assuming the critical sections are correct in isolation). You can also think of critical sections guarded by the same lock as being atomic with respect to each other, so that each sees only the complete set of changes from earlier critical sections, and never sees partially-completed updates.

锁降低性能：

> Although correct use of locks can make incorrect code correct, locks limit performance. For example, if two processes call kfree concurrently, the locks will serialize the two calls, and we obtain no benefit from running them on different CPUs

#### 6.2 Code: Locks

Xv6有两种锁：Spinlock和Sleep Lock，自旋锁和睡眠锁。

自旋锁是一个`struct spinlock`

其实现：利用原子指令`amoswap`实现5和6行，使得5和6行是一个原子操作

`amoswap`的意思是`Atomic Swap`

```c
void
acquire(struct spinlock *lk) // does not work!
{
    for(;;) {
        if(lk->locked == 0) {
            lk->locked = 1;
            break;
        }
    }
}
```

关于`amoswap`指令：简单来讲就是交换寄存器r和地址a的值，原子性的交换。

```assembly
amoswap r, a # 读取地址a处的值，将寄存器r保存的值放入地址a处，同时把读取到的值放回寄存器r。原子性地实现整个过程
```

这个指令使用硬件来阻止其他CPU在它读写a的时候访问a。



Xv6的Acquire使用C库的`__sync_lock_test_and_set`调用，本质上是xv6的`amoswap`指令。这个调用的返回值是`locked`的旧值，同时把1和`locked`交换。把这个调用放在一个`while`循环的条件里，这样假如返回值是0，那么我们知道锁是空闲的，并且把`locked`置为了1，拿到了锁。如果返回值是1，我们知道锁被占用，将锁的`locked`的值和1交换也恰好没有改变`locked`原有的值。

> The acquire function wraps the swap in a loop, retrying (spinning) until it has acquired the lock. Each iteration swaps one into lk->locked and checks the previous value; if the previous value is zero, then we’ve acquired the lock, and the swap will have set lk->locked to one. If the previous value is one, then some other CPU holds the lock, and the fact that we atomically swapped one into lk->locked didn’t change its value.



Xv6的`release`使用C库的`__sync_lock_release`指令来将`locked`的值清零，这个指令本质上也是一个`amoswap`指令。同样也是为了保证清零的原子性。

#### 6.3 Code: Using locks

使用锁的一些基本原则：什么时候锁是必需的

> First, any time a variable can be written by one CPU at the same time that another CPU can read or write it, a lock should be used to keep the two operations from overlapping. Second, remember that locks protect invariants: if an invariant involves multiple memory locations, typically all of them need to be protected by a single lock to ensure the invariant is maintained.

有的简易的内核使用一个锁在多处理器上运行，这个锁在进入内核之前必须被获取，在退出内核的时候必须被释放。许多单处理器OS使用这种方法来使得内核可以在多处理器的系统上运行，这种方法有时候叫做a “big kernel lock”。

但是这种方法牺牲了并行性：一次实际上只能有一个CPU在内核里执行。

>  If the kernel does any heavy computation, it would be more efficient to use a larger set of more fine-grained locks, so that the kernel could execute on multiple CPUs simultaneously.



Xv6粗粒度的锁的例子：Xv6的物理页分配器，Kalloc就只有一个Freelist和一个锁，假如多个CPU有分配物理页的需求，那就只能串行化。并且因为Acquire是在不断轮循自旋的，所以实际上等待中的CPU在空转浪费CPU时间。如果对Freelist的锁的竞争比较激烈的话，就可能浪费很多的CPU时间。

提高性能的一种方式：使用多个Freelist，每个Freelist一个锁，这样可以允许真正的并发分配。



Xv6细粒度的锁的例子：Xv6对每个文件都有一个单独的锁，这样使用不同文件的不同进程之间不需要等待彼此。

#### 6.4 Deadlock and lock ordering

> If a code path through the kernel must hold several locks at the same time, it is important that all code paths acquire those locks in the same order. If they don’t, there is a risk of *deadlock*.
>
> To
>
> avoid such deadlocks, all code paths must acquire locks in the same order. The need for a global lock acquisition order means that locks are effectively part of each function’s specification: callers must invoke functions in a way that causes locks to be acquired in the agreed-on order.

文件系统包含Xv6最长的锁链：

> For example, creating a file requires simultaneously holding a lock on the directory, a lock on the new file’s inode, a lock on a disk block buffer, the disk driver’s vdisk_lock, and the calling process’s p->lock. To avoid deadlock, file-system code always acquires locks in the order mentioned
>
> in the previous sentence.

有的时候也并没有那么容易能够提前知道正确的锁的获取顺序是什么，有的时候可能根本没有正确的顺序，原因有：



> Honoring a global deadlock-avoiding order can be surprisingly difficult. Sometimes the lock order conflicts with logical program structure, e.g., perhaps code module M1 calls module M2, but the lock order requires that a lock in M2 be acquired before a lock in M1. Sometimes the identities of locks aren’t known in advance, perhaps because one lock must be held in order to discover the identity of the lock to be acquired next. This kind of situation arises in the file system as it looks up successive components in a path name

#### 6.5 Locks and interrupt handlers

如果有一个（自旋）锁被中断处理程序使用了，那么**CPU永远不能在持有这个锁的时候开启中断**，不然有死锁的风险。

举个例子：一个自旋锁L，被用于某个系统调用，也被用于一个中断。用户进程A调用这个系统调用，刚拿到锁，然后这时候一个中断出现了，A的CPU被中断，CPU上的中断处理程序也要先获取锁L才能继续。这种情况下L永远不会被释放，因为A才能释放它，但是A没办法返回CPU继续运行，除非中断处理程序返回，但是中断处理程序没办法获取到L，没办法返回。

简单一点说就是，中断处理程序要锁才能继续，所以它需要A放锁。但是A想要放锁需要返回CPU运行才能放锁，但是因为中断处理程序没办法返回（因为拿不到锁），所以A也没办法放锁。死锁。

**当然，这种情况下，只要中断处理程序和A不在同一个CPU上，就不会死锁，A在其中一个CPU上运行然后放锁，中断处理程序在另外一个CPU上自旋然后拿到锁**。

Xv6则更加保守，当一个CPU获取任意一个锁，Xv6就一定会把这个CPU上的中断关闭。

> Xv6 is more conservative: when a CPU acquires any lock, xv6 always disables interrupts on that CPU. Interrupts may still occur on other CPUs, so an interrupt’s acquire can wait for a thread to release a spinlock; just not on the same CPU.

这句话应该这么说：当一个CPU持有至少一个自旋锁的时候，Xv6就会把这个CPU上的中断关闭。反面上来说，当且仅当CPU持有0个自旋锁的时候，这个CPU上的中断才会被重新开启。

因为考虑到可能存在临界区里嵌套另外一个临界区的情况，也就存在一个CPU持有2个及以上自旋锁的情况，而XV6只有在自旋锁的持有数到达0的时候才会重新开启中断，所以在Acquire和release里，Xv6都分别调用了`push_off`和`pop_off`，它们会检查当前CPU自旋锁的持有数。并且在持有数到达0的时候，`pop_off`会恢复当前CPU在最外层临界区时的，中断开关状态。

注意一下`pop_off`并不是无脑开启中断，而是恢复在进入最外层的临界区时，中断的启用状态，也就是说，当你进入最外层临界区时中断是关闭的，那么`pop_off`在CPU的自旋锁持有数为0的时候，也是把中断恢复为关闭的状态。

> acquire calls push_off (kernel/spinlock.c:89) and release calls pop_off (kernel/spinlock.c:100) to track the nesting level of locks on the current CPU. When that count reaches zero, pop_off restores the interrupt enable state that existed at the start of the outermost critical section.



必须在关中断之后拿锁，也并且在放锁之后开中断。因为我们的语义是，**持有锁的时候中断必须是关着的**。这也就意味着，拿锁之前就要关中断，这样拿到的时候中断才是关着的，否则会有一个小的时间窗口，上，持有锁且开着中断，这个时间窗口上就可能死锁。

同样的，放锁之后才能开中断，因为放锁完成之前锁都算是持有的状态，必须保证中断关着。只有放锁之后才不算持有，才能开中断。

原文如下：

> It is important that acquire call push_off strictly before setting lk->locked (kernel/spin lock.c:28). If the two were reversed, there would be a brief window when the lock was held with interrupts enabled, and an unfortunately timed interrupt would deadlock the system. Similarly, it is important that release call pop_off only after releasing the lock (kernel/spinlock.c:66).

#### 6.6 Instruction and memory ordering

一般我们会很自然地认为指令的执行顺序就和源代码中语句出现的顺序是一致的，但是实际上CPU是有可能会乱序执行指令以提高性能的。CPU和编译器会在保证不改变结果的前提下乱序执行代码，但是这种乱序执行可能会在多核情况下导致错误的结果。

> Compilers and CPUs follow rules when they re-order to ensure that they don’t change the results of correctly-written serial code. However, the rules do allow re-ordering that changes the results of concurrent code, and can easily lead to incorrect behavior on multiprocessors. The CPU’s ordering rules are called the *memory model*.

多线程的情况下，指令重排可能会导致临界区的读写排在了release的后面，导致先释放锁再做了修改。

概括性地说就是，指令重排可能会把临界区给挪到Acquire和Release的外面去，导致错误的结果。

所以Xv6为了保证Acquire和release之间的指令不被乱序执行，在Acquire和release中使用`__sync_synchronize()`。`__sync_synchronize()`是一个内存屏障（*memory barrier*）。内存屏障告诉CPU和编译器不要做指令重排

#### 6.7 Sleep locks

> Sometimes xv6 needs to hold a lock for a long time.  Holding a spinlock that long would lead to waste if another process wanted to acquire it, since the acquiring process would waste CPU for a long time while spinning. Another drawback of spinlocks is that a process cannot yield the CPU while retaining a spinlock

Xv6的自旋锁的一个很明显的弊端就是，Acquire在获取锁的过程中，是没办法让出CPU的（看了源代码就知道，Acquire的第一行就是关中断，中断都关了是没办法让出CPU的）。

Xv6的自旋锁还有一个弊端，就是持有自旋锁的进程是没办法让出CPU的（因为Acquire关掉了当前CPU的中断，所以时钟中断也不存在，所以持有锁的进程不会被时钟中断赶下CPU）。

但是我们想这样做，让持有锁但是在等待（比如等待硬盘）的进程暂时让出CPU来，这样其他进程可以运行（而目前我们的自旋锁是不行的）。自旋锁不能这样做，这样做同样也违反了Xv6对于持有自旋锁时必须关闭中断的语义（因为让进程让出CPU需要时钟中断）。



所以Xv6有另外一种锁，睡眠锁，允许进程在等待获取锁（也就是正在Acquiring中）的时候能让出CPU，以及进程在持有锁的时候也可以让出CPU（以及允许中断）

> Thus we’d like a type of lock that yields the CPU while waiting to acquire, and allows yields (and interrupts) while the lock is held.
>
> Xv6 provides such locks in the form of *sleep-locks*. acquiresleep (kernel/sleeplock.c:22) yields the CPU while waiting, using techniques that will be explained in Chapter 7

因为睡眠锁会让中断开启，所以睡眠锁是不能够用于中断处理程序的（中断处理程序只能用自旋锁）。因为acquiresleep可能会让出CPU，所以睡眠锁不能用于被自旋锁保护的临界区的内部（因为自旋锁保护的临界区，用自旋锁来保护本身就是意味着不想让出CPU的，自旋锁是不会让出CPU的，这时候再在临界区里面再来一个睡眠锁，就违背了使用自旋锁不愿意让出CPU的意图了）。

自旋锁则可以用于睡眠锁保护的临界区的内部（因为自旋锁只是会浪费CPU时间而已）。或者可以这么说，既然你可以用睡眠锁，那就意味者即使中途放弃CPU也是可以保证程序的准确性的，那么假如我再加一个自旋锁，连CPU都不让出来，那程序也不可能会反过来出错。

> Because sleep-locks leave interrupts enabled, they cannot be used in interrupt handlers. Be cause acquiresleep may yield the CPU, sleep-locks cannot be used inside spinlock critical sections (though spinlocks can be used inside sleep-lock critical sections).
>
> Spin-locks are best suited to short critical sections, since waiting for them wastes CPU time; sleep-locks work well for lengthy operations.

### 阅读源代码spinlock.h和spinlock.c

`spinlock.h`里就一个锁的结构体`struct spinlock`的声明：

`cpu`和`name`都要求在持有了自旋锁之后才能赋值，在Acquire应该可以看到

```c
// Mutual exclusion lock.
struct spinlock {
  uint locked;       // Is the lock held?

  // For debugging:
  char *name;        // Name of lock.
  struct cpu *cpu;   // The cpu holding the lock.
};

```

然后就是`spinlock.c`：`initlock`应该就是类似一个构造器的作用，用于获取一个新的锁（new 一个），并且给这个锁一个名字。

```c
void
initlock(struct spinlock *lk, char *name)
{
  lk->name = name;
  lk->locked = 0;
  lk->cpu = 0;
}
```

`Acquire`：

```c
// Acquire the lock.
// Loops (spins) until the lock is acquired.
void
acquire(struct spinlock *lk)
{
  push_off(); // disable interrupts to avoid deadlock.
  if(holding(lk))//如果当前CPU已经持有这个锁了，重复的Acquire触发Panic
    panic("acquire");

  // On RISC-V, sync_lock_test_and_set turns into an atomic swap:
  //   a5 = 1
  //   s1 = &lk->locked
  //   amoswap.w.aq a5, a5, (s1)
  while(__sync_lock_test_and_set(&lk->locked, 1) != 0)//实际上拿到锁的地方是这里，从这里往下的操作都是持有锁的操作
    ;

  // Tell the C compiler and the processor to not move loads or stores
  // past this point, to ensure that the critical section's memory
  // references happen strictly after the lock is acquired.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();
//这个函数会生成一条内存屏障的指令，确保在这个函数调用之前的所有内存访问操作都完成，且在这个函数调用之后的内存访问操作不会被重排序到这个函数调用之前。
  // Record info about lock acquisition for holding() and debugging.
  lk->cpu = mycpu();
}
```

`release`：

```c
// Release the lock.
void
release(struct spinlock *lk)
{
  if(!holding(lk))
    panic("release");

  lk->cpu = 0;

  // Tell the C compiler and the CPU to not move loads or stores
  // past this point, to ensure that all the stores in the critical
  // section are visible to other CPUs before the lock is released,
  // and that loads in the critical section occur strictly before
  // the lock is released.
  // On RISC-V, this emits a fence instruction.
  __sync_synchronize();

  // Release the lock, equivalent to lk->locked = 0.
  // This code doesn't use a C assignment, since the C standard
  // implies that an assignment might be implemented with
  // multiple store instructions.
  // On RISC-V, sync_lock_release turns into an atomic swap:
  //   s1 = &lk->locked
  //   amoswap.w zero, zero, (s1)
  __sync_lock_release(&lk->locked);//锁实际上是在这里释放的，从这里往上，都是还持有锁的。

  pop_off();
}
```

Acquire和release两个内存屏障把整个临界区夹在中间，保证了临界区中的代码可能会被重排，但是一定不会被重排到拿锁之前，或者放锁之后，只能在两个屏障之间的空间重排。应该是这个意思？

- 所谓的拿锁之前就是指`while(__sync_lock_test_and_set(&lk->locked, 1) != 0)`跳出循环之前
- 所谓的放锁之后，就是指 `__sync_lock_release(&lk->locked);`将locked置为0之后。

接下来是`push_off`和`pop_off`，一个被Acquire调用来关中断，另外一个被`release`调用来开中断。

`pushoff`和`popoff`是成对出现的，N次`pushoff`就需要对应N次`popoff`来解除这N次`pushoff`的效应。

因为一次Acquire里面调用一次`push_off`，而每Acquire一次，持有的自旋锁数目就加一，所以`push_off`的调用层数和持有的自旋锁数是相等的，所以可以理解成`push_off`的调用层数，就是当前持有的自旋锁数目。

`popoff`则可以理解成，相应地在`release`释放一个自旋锁的时候，调用`popoff`相应地将当前持有的自旋锁数目减一。

`mycpu()->noff`其实就是*push_off()* 的调用层数，也可以说是持有的自旋锁数目。

```c
// push_off/pop_off are like intr_off()/intr_on() except that they are matched:
// it takes two pop_off()s to undo two push_off()s.  Also, if interrupts
// are initially off, then push_off, pop_off leaves them off.

void
push_off(void)
{
  int old = intr_get();

  intr_off();//关闭中断
  if(mycpu()->noff == 0)
    mycpu()->intena = old;
  mycpu()->noff += 1; //调用层数自加1
}

void
pop_off(void)
{
  struct cpu *c = mycpu();
  if(intr_get())
    panic("pop_off - interruptible");
  if(c->noff < 1)
    panic("pop_off");
  c->noff -= 1; //层数自减1
  if(c->noff == 0 && c->intena) //如果持有锁的数目到达0,并且CPU的中断最开始是开启的
    intr_on();//那么就恢复中断的开启状态(否则就保持关闭的状态不变即可,所以没有else,因为else就是什么都不做)
}
```

最后是用于检查当前CPU是否持有某个锁的`holding`：

- `holding`只在`Acquire`和`release`里被调用了。而在`Acquire`中的调用是在拿锁之前检查的，所以`holding`要求关闭中断。

```c
// Check whether this cpu is holding the lock.
// Interrupts must be off.
int
holding(struct spinlock *lk)
{
  int r;
  r = (lk->locked && lk->cpu == mycpu());
  return r;
}

```

### Lecture

> 如果内核中只有一把大锁，我们暂时将之称为big kernel lock。基本上所有的系统调用都会被这把大锁保护而被序列化。系统调用会按照这样的流程处理：一个系统调用获取到了big kernel lock，完成自己的操作，之后释放这个big kernel lock，再返回到用户空间，之后下一个系统调用才能执行。这样的话，如果我们有一个应用程序并行的调用多个系统调用，这些系统调用会串行的执行，因为我们只有一把锁。所以通常来说，例如XV6的操作系统会有多把锁，这样就能获得某种程度的并发执行。如果两个系统调用使用了两把不同的锁，那么它们就能完全的并行运行。



Case-Study，关于生产者消费者的一些数据不变性（也可以类比到ArrayList的实现上的不变性）

- 锁的存在就是为了保证UART芯片里的这些不变性。

> 当我们传输数据时，写指针会指向传输缓存的下一个空闲槽位，而读指针指向的是下一个需要被传输的槽位。这是我们对于并行运算的一个标准设计，它叫做消费者-生产者模式。
>
> 所以现在有了一个缓存，一个写指针和一个读指针。读指针的内容需要被显示，写指针接收来自例如printf的数据。我们前面已经了解到了锁有多个角色。第一个是保护数据结构的特性不变，数据结构有一些不变的特性，例如**读指针需要追赶写指针**；**从读指针到写指针之间的数据是需要被发送到显示端**；**从写指针到读指针之间的是空闲槽位**（从写指针到缓冲区结尾，再回环过去到读指针），锁帮助我们维护了这些特性不变

RISC-V上的原子性的TestAndSet：amoswap（atomic memory swap）

> 这个指令接收3个参数，分别是address，寄存器r1，寄存器r2。这条指令会先锁定住address，将address中的数据保存在一个临时变量中（tmp），之后将r1中的数据写入到地址中，之后再将保存在临时变量中的数据写入到r2中，最后再对于地址解锁。
>
> 原子指令的实现其实就是将一个软件锁转变为了一个硬件锁的实现：
>
> - 多个处理器共用一个内存控制器，内存控制器可以支持这里的操作，比如给一个特定的地址加锁，然后让一个处理器执行2-3个指令，然后再解锁。因为所有的处理器都需要通过这里的内存控制器完成读写，所以内存控制器可以对操作进行排序和加锁。
> - 如果内存位于一个共享的总线上，那么需要总线控制器（bus arbiter）来支持。总线控制器需要以原子的方式执行多个内存操作。
> - 如果处理器有缓存，那么缓存一致性协议会确保对于持有了我们想要更新的数据的cache line只有一个写入者，相应的处理器会对cache line加锁，完成两个操作。

#### 提问

为什么release函数中不直接使用一个store指令将锁的locked字段写为0？

- 因为没有人可以保证`store`指令是一个原子指令。

> 这里的问题是，对于很多人包括我自己来说，经常会认为一个store指令是一个原子操作，但实际并不总是这样，这取决于具体的实现。例如，对于CPU内的缓存，每一个cache line的大小可能大于一个整数，那么store指令实际的过程将会是：首先会加载cache line，之后再更新cache line。所以对于store指令来说，里面包含了两个微指令。

在acquire函数的最开始，会先关闭中断。为什么？

> 简而言之就是，存在这样一种情况：如果这里只有一个CPU的话，假如我们有一个锁A，这个锁正在被当前用户进程持有，然后这时候来了一个中断，CPU的控制权转移到中断成立程序，然后这个程序也需要先获取锁A才能继续下去，然后就死锁了。
>
> 

## Lecture 11/13

### 阅读Chapter 7 Scheduling(边阅读源码)

任何操作系统都可能运行比计算机拥有的CPU数目更多的进程，因此需要一个方法来在进程之间时间共享CPU。

理想情况下，共享对用户进程是透明的。

一种常见的方法是，通过将进程多路复用到硬件CPU上，为每个进程提供一种认为它有自己的虚拟CPU的错觉。本章解释了xv6是如何实现这种多路复用的

#### 7.1 Multiplexing

Xv6通过从一个进程切换CPU到另外一个进程来实现复用。

在两种情况下，Xv6会进行这种切换：

- Xv6的*sleep*和*wakeup*机制下，当一个进程在等待设备或者管道I/O，或者等待子进程退出，或者在Sleep系统调用中等待的时候
- 周期性的强制性切换，以应对计算密集型的长时间计算而不休眠的进程。

这种切换使得进程产生了自己有专属CPU的错觉



实现多路复用带来的挑战：

- 如何从一个进程切换到另外一个进程？虽然上下文切换的概念很简单，但是实现很困难
- 如何让这种切换对用户进程透明？即，如何以一种透明的方式来实现？Xv6借助了时钟中断
- 许多CPU可能会并发地做进程切换，需要加锁以消除竞态条件。
- 当一个进程退出的时候它的所有资源都必须回收，但是它没办法自己去做全部的回收工作（比如没办法在使用内核栈的时候回收栈）
- 多核机器上的每个核都必须记住自己正在执行的是哪个进程，这样系统调用才能对正确的进程的内核状态产生影响。
- *sleep*和*wakeup*机制实现很麻烦，需要避免竞态条件

#### 7.2 Code: Context switching

从一个用户进程切换到另外一个用户进程的步骤：

1. 一个user-kernel切换，切换到老用户进程的内核线程
2. 上下文切换到当前CPU的调度器线程
3. 上下文切换到新用户进程的内核线程
4. 一个Trap返回用户进程

Xv6的调度器为每个核都有一个专用的调度器线程（专用的寄存器值和专用的栈）。调度器线程不会使用旧的进程的内核栈，因为这个旧的进程有可能会被另外一个CPU同时执行，另外那个CPU同时执行的时候有可能会使用旧的进程的内核栈，所以调度器线程不能使用旧的进程的内核栈，否则可能出现两个CPU核同时使用一个内核栈的情况，直接寄。

举个例子，进程A刚被从CPU0上踢下来，调度器线程上去，A转头就去CPU1运行了，那如果调度器线程使用A的内核栈，直接寄。

>  The xv6 scheduler has a dedicated thread (saved registers and stack) per CPU because it is not safe for the scheduler execute on the old process’s kernel stack: some other core might wake the process up and run it, and it would be a disaster to use the same stack on two different cores.



从一个线程切换到另外一个线程，涉及保存旧进程的CPU寄存器集，以及恢复新线程的预先保存好的寄存器集的值。因为PC的值和栈指针`sp`的值都会被保存和恢复，所以CPU在切换线程的同时，也会切换栈以及切换它正在执行的代码。



函数`swtch`为内核线程的切换执行保存和回复寄存器的工作。`swtch`其实不知道它做的是上下文切换，它只是保存和恢复一些寄存器的值（也即上下文），它主观上是不知道自己其实做的是内核线程的上下文切换的工作的。

每当到了一个进程放弃CPU的时候，它的内核线程会调用`swtch`来保存它自己的上下文，然后return到调度器的上下文。

每个上下文被保存在`struct context(kernel/proc.h:2)`，而`struct context`自己则被包含在一个进程的`struct proc`或者一个CPU的`struct cpu`中。（从源代码可知，Xv6的调度器为每个核都有一个专用的调度器线程，这个调度线程的上下文就保存在`struct cpu`中）



`swtch.S`里就只有`swtch`的实现：从源代码里也可以看到，`swtch`就只是做了一些load和store的工作，它并不知道其他的信息。

`swtch`是函数`void swtch(struct context *old, struct context *new)`的实现，这个函数把旧进程的上下文保存在`old`，然后从`new`里恢复新进程的上下文。

寄存器`a0`是函数的第一个参数，`a1`是第二个参数。还记得之前学习的调用约定么？

```c
# Context switch
#
#   void swtch(struct context *old, struct context *new);
#
# Save current registers in old. Load from new.	
.globl swtch
swtch:
        sd ra, 0(a0)
        sd sp, 8(a0)
        sd s0, 16(a0)
        sd s1, 24(a0)
        sd s2, 32(a0)
        sd s3, 40(a0)
        sd s4, 48(a0)
        sd s5, 56(a0)
        sd s6, 64(a0)
        sd s7, 72(a0)
        sd s8, 80(a0)
        sd s9, 88(a0)
        sd s10, 96(a0)
        sd s11, 104(a0)

        ld ra, 0(a1)
        ld sp, 8(a1)
        ld s0, 16(a1)
        ld s1, 24(a1)
        ld s2, 32(a1)
        ld s3, 40(a1)
        ld s4, 48(a1)
        ld s5, 56(a1)
        ld s6, 64(a1)
        ld s7, 72(a1)
        ld s8, 80(a1)
        ld s9, 88(a1)
        ld s10, 96(a1)
        ld s11, 104(a1)
        
        ret
```

在之前讲Trap的时候，Trap有一种可能就是时钟中断，然后调用`yield`。`yield`会调用`sched`，然后`sched`调用`swtch`来保存当前上下文，然后切换到调度器的上下文（预先保存在`cpu->scheduler`）。

这里从`yield(proc.c:520)`开始看代码：

首先附上`struct proc`和`struct cpu`：

```c

enum procstate { UNUSED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  struct proc *parent;         // Parent process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.【当前运行在这个CPU的进程是谁?】
  struct context context;     // swtch() here to enter scheduler().【调度线程的上下文保存在这里】
  int noff;                   // Depth of push_off() nesting.【push_off的嵌套层数，也可以说是运行在当前CPU上的进程持有的锁的数目】
  int intena;                 // Were interrupts enabled before push_off()?【在第一次调用push_off之前,中断是开启的还是关闭的?】
};//intena用于在spinlock的Release中，恢复CPU的中断状态时使用，去查看pup_off(spinlock.c)的代码
```

```c
// Give up the CPU for one scheduling round.
void
yield(void)
{
  struct proc *p = myproc();
  acquire(&p->lock); //拿到当前进程的proc结构体的锁。只有持有锁的时候才能去访问和修改proc结构体
  p->state = RUNNABLE;//把进程的状态改为RUNNABLE
  sched();//切换上下文
  release(&p->lock);
}
```

`yield`调用`sched`：必须要持有当前进程的`proc`的锁才能调用。

```c
// Switch to scheduler.  Must hold only p->lock
// and have changed proc->state. Saves and restores
// intena because intena is a property of this
// kernel thread, not this CPU. It should
// be proc->intena and proc->noff, but that would
// break in the few places where a lock is held but
// there's no process.
void
sched(void)
{
  int intena;
  struct proc *p = myproc();

  if(!holding(&p->lock))
    panic("sched p->lock");
  if(mycpu()->noff != 1)//CPU持有自旋锁的时候,正常情况下中断是关闭的,不可能yield-->sched，所以这里一定是出错了。
    panic("sched locks");
  if(p->state == RUNNING)
    panic("sched running");
  if(intr_get())
    panic("sched interruptible");

  intena = mycpu()->intena;
  swtch(&p->context, &mycpu()->context);//&mycpu()->context保存的是调度器线程的上下文，所以这里是将上下文切换到了调度器线程
  mycpu()->intena = intena;
}
```

`Swtch`只保存由被调用者保存的寄存器（callee-saved registers），调用者保存的寄存器（caller-saved registers）则由调用的C代码保存在栈上。`Swtch`**不保存PC的值，但是保存和回复ra寄存器的值**。ra寄存器保存了`swtch`被调用时的返回地址。`swtch`保存旧的进程的ra，然后从新的进程的 `struct context` 恢复上下文，恢复了ra，这个ra是由前一次`swtch`（新进程作为旧进程时）保存的。因此，当`swtch`恢复上下文完毕，返回的时候，恰好就返回到了新的进程此前调用`swtch`的地方。**此外，swtch返回的时候栈也切换了**（`sp`寄存器的值保存和恢复）。

 举个例子，进程A和进程B说明恢复`ra`的妙用，他们的代码分别是：

```c
//A:
swtch();
a = 1;
//B:
swtch();
b = 2;
```

假如进程A先运行，然后调用了`swtch`，`swtch`会让返回地址`ra`指向`a = 1`。

中间其他的进程运行了一段时间，终于轮到了进程B运行，B调用`swtch`，`swtch`保存B的上下文，然后回复A的上下文。

恢复完毕的时候，`ra`寄存器存储的返回地址恰好就是`a = 1`这一行的地址。然后`swtch`执行`ret`返回，`ret`指令把PC的值置为`ra`。

这样进程A就如同完全没有被中断过一样，只是简单的做了一个`swtch`函数调用，然后从函数调用返回，按顺序继续执行了下一行代码。仿佛什么都没有发生过一样。或者说，A就刚好从它之前中断的地方继续运行下去了。



在yield的例子里，调度器线程在回复了新的进程的上下文之后（调度器对`swtch`的调用会保存调度器的上下文，同时恢复新进程的上下文），`ret`就直接返回了新进程当初中断的地方。

 以进程A利用yield切换到进程B为例：

进程A调用yield，yield调用`swtch(A,Scheduler)`，`swtch`保存A的内核线程上下文，同时将上下文切换到调度器线程，调度器线程调用`swtch(Scheduler, B)`，`swtch`将调度器的上下文保存在`cpu->context`，同时恢复B的上下文，ret返回进程B。而下一次恢复调度器上下文的时候，使用的正是前一次调度器调用`swtch`为自己保存的上下文。**本次调度器为自己保存的上下文，其实就是下一次yield调用swtch，swtch为调度器恢复的上下文**。

#### 7.3 Code: Scheduling

调度器以一种每个CPU都有的特殊线程的形式存在（可以看做是一种绑定的关系），每个调度器线程都在运行`scheduler`函数。

> The scheduler exists in the form of a special thread per CPU, each running the scheduler function.

这个函数负责选出下一个运行的进程。一个想要放弃CPU的进程，必须拿到自己的锁，更新自己的进程运行状态，释放它持有的任意锁，然后调用`sched`。Yield、Sleep和exit都遵从了这个约定。

> A process that wants to give up the CPU must acquire its own process lock p->lock, release any other locks it is holding, update its own state (p->state), and then call sched.

下面是调度器运行的代码，最外面是一个死循环，里面先遍历`proc`数组，找到第一个状态为`p->state == RUNNABLE`的进程，然后改变其状态为`RUNNING`，然后将当前CPU的`c->proc = p`置为找到的这个进程。然后调用`swtch`切换到找到的进程。这时候其实调度器线程就已经被中断了，所以当调度器线程下一次恢复运行的时候，它是从它调用`swtch`的下一行代码接着运行的，**而不是直接从头开始下一个循环**。

并且我们需要注意到，在遍历到任意一个进程之前，我们都先拿到了这个进程的`p->lock`，这是为了防止出现竞态条件。

举个例子，另外一个CPU的调度器线程也在运行的情况下，两个CPU找到了同一个为`RUNNABLE`的进程，然后同时运行它，出现两个CPU运行同一个进程的情况（加了锁，就只有一个CPU最终会运行这个进程，因为另外一个CPU的调度器线程在拿到锁之后，检查其状态会发现是`RUNNING`而不是`RUNNABLE`）。

那我有一个问题，这个锁是调度器拿到的，然后调度器切换线程之后，新的进程是在哪里释放这个锁的？而且要求新的进程在跳转回调度器之前重新拿回锁，又是从哪里拿回来的？

这个问题的解答我写在`scheduler`代码的下面： 

```c
struct proc proc[NPROC]; //proc是全局变量
// Per-CPU process scheduler.
// Each CPU calls scheduler() after setting itself up.
// Scheduler never returns.  It loops, doing:
//  - choose a process to run.
//  - swtch to start running that process.
//  - eventually that process transfers control
//    via swtch back to the scheduler.
void
scheduler(void)
{
  struct proc *p;
  struct cpu *c = mycpu();
  
  c->proc = 0;
  for(;;){
    // Avoid deadlock by ensuring that devices can interrupt.
    intr_on();
    
    int found = 0;
    for(p = proc; p < &proc[NPROC]; p++) {
      acquire(&p->lock);
      if(p->state == RUNNABLE) {
        // Switch to chosen process.  It is the process's job
        // to release its lock and then reacquire it
        // before jumping back to us.
        p->state = RUNNING;
        c->proc = p;
        swtch(&c->context, &p->context);

        // Process is done running for now.
        // It should have changed its p->state before coming back.
        c->proc = 0;

        found = 1;
      }
      release(&p->lock);
    }
    if(found == 0) {
      intr_on();
      asm volatile("wfi");
    }
  }
}
```

这个问题的答案我们要结合`Yield`和`Scheduler`的代码来一起看。并且要举一个具体的例子来说明

首先我们知道，会触发调度器调度的，是时钟中断。而一个时钟中断类型的Trap，会使得CPU去运行`Yield`：

我们记`yield`之前运行的进程是进程A（也就是旧进程），调度器调度之后运行的进程是进程B（也就是新进程）。

为了方便，**我们假设只有机器上这两个进程**。

```c
// Give up the CPU for one scheduling round.
void
yield(void)
{
  struct proc *p = myproc();
  acquire(&p->lock); //拿到当前进程的proc结构体的锁。只有持有锁的时候才能去访问和修改proc结构体
  p->state = RUNNABLE;//把进程的状态改为RUNNABLE
  sched();//切换到调度器的上下文
  release(&p->lock);
}
```

在`yield`的第7行，将A的进程状态改为`RUNNABLE`之后，`yield`调用`sched`，`sched`调用`swtch`切换到调度器的上下文。这时候`yield`的`release(&p->lock)`是**没有机会被执行的**。因为直接到调度器的上下文去了。这里更准确的说法是，实际上中断的地方是`sched`内部调用`swtch`的那一行，但是`sched`会返回`yield`。所以我们如果把`sched`看成一个整体，在`yield`中就是被中断在了第8/9行。



调度器则做了下面的事：拿到B的`proc`结构体锁，找到B，把B的进程状态设置为`RUNNING`，然后把CPU的`proc`设置成B的`proc`，然后就调用`swtch`，切换到了B的上下文。

注意到，这里**调度器的**`release(&p->lock);`也是没有机会执行的，更准确一点说，是第5行往后的都还没执行。

```c
acquire(&p->lock);
if(p->state == RUNNABLE) {
p->state = RUNNING;
c->proc = p;
swtch(&c->context, &p->context);
c->proc = 0;
found = 1;
}
release(&p->lock);
```

我们知道`swtch`执行的最后一条指令是`ret`，它会返回B前一次被中断的地方。

而B上一次运行的时候，被中断的地方是哪？**就是yield的第8行，也就是进程A被中断的地方**。为什么？A之所以能够使用CPU，是因为B也像A一样，调用了`yield`，把CPU让给了A，所以B也在`yield`的第8行被中断了。因此，切换到进程B的上下文的时候，B执行的第一行代码，就会是`yield`的`release(&p->lock)`，也就把刚才调度器所`Acquire`的锁给释放掉了。然后B才返回了用户态。



然后在下一个时钟中断到来的时候，此时CPU上运行的是B，B的内核线程从`yield`的第一行开始执行，`yield`的第6行`acquire(&p->lock);`把B的`proc`锁又拿回来，然后`yield`调用`swtch`切换到调度器线程。

同样的，`swtch`执行的最后一条指令是`ret`，它会返回调度器线程被中断的地方，**也就是上文说的，调度器没来得及执行的第6行**。

然后调度器从它中断的地方继续，依次执行`c->proc = 0;`、`found = 1;`、`release(&p->lock);`，把`yield`拿到的B的`proc`锁放掉，然后进入下一个循环。

下一个循环调度器拿到A的锁，调用`swtch`，切换到A的上下文，CPU从A中断的地方继续开始运行，也就是`yield`的第9行（更准确的说法是`sched`的最后一行，紧接着执行的才是`yield`的第9行），把A的锁放掉。



那又有一个问题了，那就是A执行完`yield`之后，它是怎么返回用户空间（user space）的？

这个问题，去看Trap那一章的源代码，以及Trap是如何返回用户空间的就好了。

大概就是从`yield`返回之后，紧接着执行的是`usertrapret`，然后`usertrapret`会调用`trapret`，`trapret`是汇编写的，它的最后一条指令是`sret`，返回用户空间。在`Trap.c`中，`yield`之前的代码是B（的内核线程）执行的，从`yield`返回，之后的代码，是A（的内核线程）执行的。



妈的，一切串起来的感觉真他妈的爽。



只有一种情况下，调度器对`swtch`的调用不会把控制流转到`sched`，那就是一个新的进程第一次被调度的时候。会转到Forkret去。这个后面再说吧，应该有一节是将进程的创建的。Forkret的存在就是为了放掉调度器线程拿到的锁。

```c
// A fork child's very first scheduling by scheduler()
// will swtch to forkret.
void
forkret(void)
{
  static int first = 1;

  // Still holding p->lock from scheduler.
  release(&myproc()->lock);

  if (first) {
    // File system initialization must be run in the context of a
    // regular process (e.g., because it calls sleep), and thus cannot
    // be run from main().
    first = 0;
    fsinit(ROOTDEV);
  }

  usertrapret();
}
```

#### 7.4 Code: mycpu and myproc

> Xv6 maintains a *struct cpu* for each CPU (kernel/proc.h:22), which records the process currently running on that CPU (if any), saved registers for the CPU’s scheduler thread, and the count of nested spinlocks needed to manage interrupt disabling. 

`mycpu`这个函数返回当前CPU的`struct cpu`，因为RISC-V给每个CPU一个编号（*hartid*），这个编号实际上就是数组的索引（`struct cpu cpus[NCPU];`，cpus的索引）。每个CPU的编号会被保存在这个CPU的`tp`寄存器，所以能够读取这个寄存器来获取CPUID，从而索引到数组的对应项。

>  The function mycpu (kernel/proc.c:60) returns a pointer to the current CPU’s struct cpu. RISC-V numbers its CPUs, giving each a *hartid*. Xv6 ensures that each CPU’s hartid is stored in that CPU’s tp register while in the kernel. This allows mycpu to use tp to index an array of cpu structures to find the right one.

在CPU启动的时候，还处于机器模式，`mstart`会将CPUID设置在tp寄存器。`usertrapret`则会在从内核跳转到用户空间的时候保存`tp`的值，而`uservec`则会在相反的过程中恢复`tp`寄存器的值。

> *mstart* sets the tp register early in the CPU’s boot sequence, while still in machine mode (kernel/start.c:46).
>
> *usertrapret* saves tp in the trampoline page, because the user process might modify tp. Finally,*uservec* restores that saved tp when entering the kernel from user space

`cpuid()`在执行的时候必须关闭中断，防止出现这样的情况：内核线程已经读取了`tp`寄存器的值准备返回（准备执行第8行），然后这个线程突然被时钟中断，调度到另外一个CPU，然后再返回（执行`return id`）。这个时候`cpuid()`返回的CPUID是错误的值。

为了避免类似的竞态条件，Xv6要求在完成对`struct cpu`的使用之后再开启中断。

`myproc()`的原理就简单了，**关中断**，获取当前CPU的`struct cpu`，获取结构体里的`struct proc`，开中断并返回。

```c
// Per-CPU state.
struct cpu {
  struct proc *proc;          // The process running on this cpu, or null.【当前运行在这个CPU的进程是谁?】
  struct context context;     // swtch() here to enter scheduler().【调度线程的上下文保存在这里】
  int noff;                   // Depth of push_off() nesting.【push_off的嵌套层数，也可以说是运行在当前CPU上的进程持有的锁的数目】
  int intena;                 // Were interrupts enabled before push_off()?【在第一次调用push_off之前,中断是开启的还是关闭的?】
};//intena用于在spinlock的Release中，恢复CPU的中断状态时使用，去查看pup_off(spinlock.c)的代码

// Must be called with interrupts disabled,
// to prevent race with process being moved
// to a different CPU.
int
cpuid()
{
  int id = r_tp();
  return id;
}

// Return this CPU's cpu struct.
// Interrupts must be disabled.
struct cpu*
mycpu(void) {
  int id = cpuid();
  struct cpu *c = &cpus[id];
  return c;
}

// Return the current struct proc *, or zero if none.
struct proc*
myproc(void) {
  push_off();
  struct cpu *c = mycpu();
  struct proc *p = c->proc;
  pop_off();
  return p;
}
```



#### 7.5 Sleep and wakeup

>  Xv6 uses one called *sleep* and *wakeup*, which allow one process to sleep waiting for an event and another process to wake it up once the event has happened. *Sleep* and *wakeup* are often called *sequence coordination* or *conditional synchronization* mechanisms

本小节先简单的引入了信号量，并利用信号量 + 轮循来解决生产者消费者问题。

然后为了解决消费者长期轮循忙等浪费CPU时间的问题，引入简单的Sleep和wakeup机制：

> Sleep(chan) sleeps on the arbitrary value chan, called the *wait channel*. Sleep puts the calling process to sleep, releasing the CPU for other
>
> work.
>
> Wakeup(chan) wakes all processes sleeping on chan (if any), causing their sleep calls to return. If no processes are waiting on chan, wakeup does nothing.

在改进之后的代码长这样：

```c
void
V(struct semaphore *s)
{
    acquire(&s->lock);
    s->count += 1;
    wakeup(s);
    release(&s->lock);
}

void
P(struct semaphore *s)
{
    while(s->count == 0){
        sleep(s);
    }
    acquire(&s->lock);
    s->count -= 1;
    release(&s->lock);
}
```

但是这样不行，存在所谓的*lost wake-up* problem：

假如`P`发现`count == 0`成立，然后此时正处于13行和14行之间的时候（比如恰好处于跳转指令`jne`），这时候另外一个CPU上的`V`把`count`加了1，然后调用`wakeup`，但是此时`wakeup`发现，没有进程处于睡眠，于是它什么都没做。然后`P`才执行了14行的`sleep`。

这就导致了一个问题：P本来要等待的那个V操作已经发生了。假如后续一直没有新的V操作的话，P操作只能永远等待下去（即便`count != 0`已经成立了）。

> This causes a problem: P is asleep waiting for a V call that has already happened. Unless we get lucky and the producer calls V again, the consumer will wait forever even though the count is non-zero.



这个问题的发生是由于，V在不恰当的时候的发生，导致了【P只在count == 0的时候sleep】的不变性被破坏了。被破坏的原因是P在检查`count`时并不能保证其他的V不能修改`count`。也就是说完全可能存在，在P检查完之后还没去Sleep的瞬间，`count`的值又被V改成非零，但是P已经检查完了认为其应该去Sleep了。

所以我们必须保证P在检查的时候，count的值不能被其他的V改变。

或者说，要保证检查和睡眠的作为一个整体的原子性。

再或者说，强制一个与P操作并发发生的V操作，必须在P将自身沉睡之后再进行。

但是，仅仅是简单地将P改成下面这样是不行的，很显然的死锁：一旦P睡眠，它就持有锁去睡了，V根本不可能拿到锁，因为P在睡眠没办法释放。

```c
void
P(struct semaphore *s)
{
    acquire(&s->lock); //Deadlock
    while(s->count == 0){
        sleep(s); //P holds the lock while it sleeps, so V will block forever waiting for the lock.
    }
    s->count -= 1;
    release(&s->lock);
}
```



为了修复这个问题，我们给Sleep的添加上一条：必须把锁传递给Sleep，Sleep负责将进程睡眠和将锁释放。

**一旦进程被wakeup，Sleep重新Acquire锁，然后再返回。**

懒得写那么清楚，直接引用原文吧：

> the caller must pass the *condition lock* to sleep so it can release the lock after the calling process is marked as asleep and waiting on the sleep channel.
>
> **The lock will force a concurrent V to wait until P has finished putting itself to sleep**, so that the wakeup will find the sleeping consumer and wake it up. 
>
> Once the consumer is awake again **sleep reacquires the lock before returning**.

改进之后的生产者消费者问题：

```c
void
V(struct semaphore *s)
{
    acquire(&s->lock);
    s->count += 1;
    wakeup(s);
    release(&s->lock);
}

void
P(struct semaphore *s)
{
    while(s->count == 0){
        sleep(s, &s->lock);
    }
    acquire(&s->lock);
    s->count -= 1;
    release(&s->lock);
}
```

**我们需要Sleep原子性地释放锁和将进程睡眠。是原子性地做这两件事**

> The fact that P holds s->lock prevents V from trying to wake it up between P’s check of c->count and its call to sleep. 
>
> Note, however, that **we need sleep to atomically release s->lock and put the consuming process to sleep.**



这一节讲述了Sleep-Wakeup机制之后，下一节就是Sleep和Wakeup的实现代码了



#### 7.6 Code: Sleep and wakeup

Sleep的基本思路就是把调用Sleep的进程（也即当前进程）标记为`SLEEPING`，然后接着调用`sched`来释放CPU。

Wakeup的基本思路就是找到一个在给定等待通道上睡眠的进程，然后将其标记为`RUNNABLE`。

Sleep和Wakeup的调用者可以使用任何双方相互约定好的（mutually convenient）数字来作为等待通道。Xv6则经常使用某个内核数据结构的地址作为通道。



Sleep的代码还是很简单的。比较值得注意的就是Sleep是如何实现【原子性地释放传入的锁`lk`和让进程去睡眠的】。

具体的原理已经在英文注释里写了：只要获取了当前进程的`Proc -> lock`，就可以保证我们不会错过任何Wakeup，因为Wakeup也需要先获取这个锁才能继续。所以我们可以安全地把`lk`释放掉。

> Now that sleep holds p->lock, it is safe to release lk: some other process may start a call to wakeup(chan), but wakeup will wait to acquire p->lock, and thus will wait until sleep has finished putting the process to sleep, keeping the wakeup from missing the sleep.

```c
/*
proc.h
*/
enum procstate { UNUSED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };

// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  struct proc *parent;         // Parent process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};

/*
proc.c
*/
// Atomically release lock and sleep on chan.
// Reacquires lock when awakened.
void
sleep(void *chan, struct spinlock *lk)
{
  struct proc *p = myproc();
  
  // Must acquire p->lock in order to
  // change p->state and then call sched.
  // Once we hold p->lock, we can be
  // guaranteed that we won't miss any wakeup
  // (wakeup locks p->lock),
  // so it's okay to release lk.
  if(lk != &p->lock){  //DOC: sleeplock0
    acquire(&p->lock);  //DOC: sleeplock1
    release(lk);
  }//这里的if判断: 当传入的lk就是p->lock时，无需重复获取p->lock了(否则会死锁)，也不能释放lk(因为lk是p->lock)。所以判断一下。

  // Go to sleep.
  p->chan = chan;
  p->state = SLEEPING;

  sched();
/*在进程被唤醒之后，进程会从这里开始运行*/
  // Tidy up.
  p->chan = 0;

  // Reacquire original lock.
  if(lk != &p->lock){
    release(&p->lock);
    acquire(lk);
  }
}
```

然后就是`Wakeup`的实现：很简单，直接遍历整个Proc数组，把处于睡眠状态并且通道对的上号的进程的状态标记为`RUNNABLE`，看代码即可。

> It acquires the p->lock of each process it inspects, both because it may manipulate that process’s state and **because p->lock ensures that sleep and wakeup do not miss each other**. When wakeup finds a process in state *SLEEPING* with a matching *chan*, it changes that process’s state to *RUNNABLE*. The next time the scheduler runs, it will see that the process is ready to be run.

```c
// Wake up all processes sleeping on chan.
// Must be called without any p->lock.
void
wakeup(void *chan)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    acquire(&p->lock);
    if(p->state == SLEEPING && p->chan == chan) {
      p->state = RUNNABLE;
    }
    release(&p->lock);
  }
}
```

有的时候会有多个进程同时在同一个Channel上睡眠，而一次Wakeup调用会把他们都唤醒（都标记为`RUNNABLE`）。这种情况下，这些进程的其中一个会第一个运行起来，然后拿到锁。其他被唤醒的进程则会先暂时阻塞在Acquire处自旋，直到第一个进程释放锁，他们中的之一拿到锁，剩余的继续自旋。第二个拿到锁的会检查循环条件，如果满足循环条件就会再一次进入循环，再次Sleep。其余进程同理。



关于Sleep-Wakeup机制的评价：

> No harm is done if two uses of sleep/wakeup accidentally choose the same channel: they will see spurious wakeups, but looping as described above will tolerate this problem. Much of the charm of sleep/wakeup is that it is both lightweight (no need to create special data structures to act as sleep channels) and provides a layer of indirection (callers need not know which specific process they are interacting with)

#### 7.7 Code: Pipes(pipe.c，但是课程要求里没列出)

这一小节来看看`piperead`和`pipewrite`是怎么实现的。



每个管道由一个`struct pipe`来代表，里面包含一个锁和一个缓冲区。缓冲区我们看做是一个环形的缓冲区。`nread`和`nwrite`分别代表读取的总字节数和写入的总字节数。因为我们把缓冲区看做是回环的，所以在写入`buf[PIPESIZE-1]`之后，下一个字节写入的地方是`buf[0]`。但是`nread`和`nwrite`是不会回环的（也即不会在到达PIPESIZE之后归零）。

因此：

- 缓冲区满的条件：nwrite ==nread+PIPESIZE。已写入的字节数比已读取的字节数多PIPESize，未读取的字节刚好全部占满了缓冲区。
- 缓冲区空的条件：nwrite == nread。已写入的字节数等于已读取的字节数，缓冲区里面没有哪一个字节是没被读取过的，所以缓冲区的每一个字节都是可以覆盖的，也就是说缓冲区是空的。
- 下一个要读取的字节在：`buf[nread % PIPESIZE]`
- 下一个写入的位置在：`buf[nwrite % PIPESIZE]`

```c

#define PIPESIZE 512

struct pipe {
  struct spinlock lock;
  char data[PIPESIZE];
  uint nread;     // number of bytes read
  uint nwrite;    // number of bytes written
  int readopen;   // read fd is still open
  int writeopen;  // write fd is still open
};
```

然后是`pipewrite`的实现：

1. 拿到管道的锁。如果此时有并发的piperead，会阻塞在Acquire处自旋。
2. 遍历`addr[0]`到`addr[n-1]`，将每一个字节写入管道的缓冲。
   1. 如果管道缓冲区满了，那就先唤醒沉睡中的读者，然后去睡眠，等待被某个读者唤醒。

```c
int
pipewrite(struct pipe *pi, uint64 addr, int n)
{
  int i;
  char ch;
  struct proc *pr = myproc();

  acquire(&pi->lock);
  for(i = 0; i < n; i++){
    while(pi->nwrite == pi->nread + PIPESIZE){  //DOC: pipewrite-full
        //管道满了
      if(pi->readopen == 0 || pr->killed){
        release(&pi->lock);
        return -1;
      }
      wakeup(&pi->nread);//叫醒沉睡的读者
      sleep(&pi->nwrite, &pi->lock); //写者去睡,sleep会把pi->lock释放
    }
    if(copyin(pr->pagetable, &ch, addr + i, 1) == -1) 
        /*这里copyin的作用是把用户空间的一个字节读取到变量ch里。
        因为ch是处于内核空间的局部变量，所以需要调用cpoyin来将addr数组的一个字节复制到ch,
        而不是简单的ch = addr[i].这样是跨地址空间的赋值,是没办法依靠等号来实现的
        ch处于内核的地址空间，而addr处于用户进程pr的地址空间.跨地址空间的赋值是没办法简单使用等号来实现的
        */
      break; 
    pi->data[pi->nwrite++ % PIPESIZE] = ch; //把ch写入到管道的缓冲区.
    /*其实ch这个变量是可以省略的，
    直接用copyin写入到pi->data[pi->nwrite++ % PIPESIZE]理论上也是可以的,
    类似:copyin(pr->pagetable, &(pi->data[pi->nwrite++ % PIPESIZE]), addr + i, 1) == -1.
    也就是直接用pi->data[pi->nwrite++ % PIPESIZE]替换ch.
    只不过可读性上相比来说就差了一大截了。
    */
  }
  wakeup(&pi->nread);//写入完毕之后,唤醒可能有的沉睡的读者，让他们去读取
  release(&pi->lock);//放掉管道的锁
  return i;//返回写入的字节数
}
```



`piperead`的基本思想也是类似的：

1. 拿锁。如果管道缓冲区是空的，那就陷入沉睡。
2. 否则从管道里逐个字节地，**从内核空间**复制n个字节到用户空间。

调用`copyout`的理由同样是，`ch`和`pi`是处于内核空间的变量，而`addr`是处于用户空间的虚拟地址，二者之间的赋值是跨地址空间的，所以**不能使用简单的`=`来实现跨地址空间的赋值**（不能通过简单的 `addr[i] =pi->data[pi->nread++ % PIPESIZE] `来将字节复制到用户空间，**因为等号两侧的变量处于不同的地址空间** ）。

```c
int
piperead(struct pipe *pi, uint64 addr, int n)
{
  int i;
  struct proc *pr = myproc();
  char ch;

  acquire(&pi->lock);
  while(pi->nread == pi->nwrite && pi->writeopen){  //DOC: pipe-empty
    if(pr->killed){
      release(&pi->lock);
      return -1;
    }
    sleep(&pi->nread, &pi->lock); //DOC: piperead-sleep
  }
  for(i = 0; i < n; i++){  //DOC: piperead-copy
    if(pi->nread == pi->nwrite)
      break;
    ch = pi->data[pi->nread++ % PIPESIZE];
    if(copyout(pr->pagetable, addr + i, &ch, 1) == -1)
      break;
  }
  wakeup(&pi->nwrite);  //DOC: piperead-wakeup
  release(&pi->lock);
  return i;
}
```

最后一段懒得做笔记了，直接贴过来：

> The pipe code uses separate sleep channels for reader and writer (pi->nread and pi->nwrite); this might make the system more efficient in the unlikely event that there are lots of readers and writers waiting for the same pipe. The pipe code sleeps inside a loop checking the sleep condition; if there are multiple readers or writers, all but the first process(除了第一个以外全部的进程) to wake up will see the condition is still false and sleep again.

#### 7.8 Code: Wait, exit, and kill

因为在子进程死亡的时候（调用`exit`的时候），父进程可能已经调用过`wait`了，也可能还在做别的事，然后后面才调用的`wait`。在后一种情况下，我们必须保证后续的`wait`不会错过已经死亡的子进程。换言之，即使是在子进程死亡很久以后，父进程才调用了`wait`，父进程也需要能观察到子进程已经死亡了。

xv6记录子进程的死亡直到父进程调用wait观察到为止的方式是，让`exit`将调用者（调用exit者）将进程的状态置为`ZOMBIE`。子进程一直保持在`ZOMBIE`状态，直到父进程的`wait`注意到了它，将它的状态改为`UNUSED`，将子进程的退出状态复制，返回子进程的PID给父进程。



如果父进程提前于子进程结束的话，父进程会将子进程移交给`init`（父进程退出会调用exit，由exit负责），由`init`来扮演父进程的角色。因此每个子进程都有一个父进程来负责清理。



首先看`wait`的实现：

wait的基本思想很简单，扫描一遍整个进程表，如果找到某一个子进程处于`ZOMBIE`状态，就把它的资源和它的`Proc`结构体释放掉，将它的退出状态复制到`addr`指向的变量，然后`return`它的PID。

如果`wait`找到的子进程都不处于`ZOMBIE`状态，它就先陷入睡眠，等待其中某个子进程`exit`，然后再次扫描。

`wait`获取锁的顺序是先获取它自己的锁，然后再试图获取子进程的锁。所以为了避免死锁，Xv6的其他部分也要遵循同样的获取顺序。



`wait`判断进程表中某个进程是否是自己的子进程的方法是，查看其`np->parent`域是否与自己相等（这个域见`struct proc(proc.h)`的定义）。但是这里有一个地方很值得注意：**` if(np->parent == p)`的判断，是在`acquire(&np->lock)`**之前的，也就是说，父进程在查看子进程`proc`的`parent`域之前，是没有持有锁的。

这破坏了使用锁的原则（共享变量必须被锁保护），为什么要这么做？

因为`wait`是遍历了整个进程表，这个进程表里存储的是**所有的**进程，**所以很有可能会存在某个进程np，它是当前进程的父进程**。如果先`acquire(&np->lock)`的话，就会违反“先拿父进程的锁，再拿子进程的锁”的原则（因为当前进程`p`是`np`的子进程），会造成死锁。

另一方面，不拿锁就去检查`np->parent`也是安全的，因为`np->parent`只有可能被`np`的父进程修改。如果当前进程就是`np`的父进程，那么除了当前进程不会有第三个进程修改，当前进程就是唯一一个可能修改`np->parent`的。如果当前进程不是`np`的父进程，那更没关系了，因为当前进程只关心自己的子进程。

```c
// Wait for a child process to exit and return its pid.
// Return -1 if this process has no children.
int
wait(uint64 addr)
{
  struct proc *np;
  int havekids, pid;
  struct proc *p = myproc();

  // hold p->lock for the whole time to avoid lost
  // wakeups from a child's exit().
  acquire(&p->lock);

  for(;;){
    // Scan through table looking for exited children.
    havekids = 0;
    for(np = proc; np < &proc[NPROC]; np++){
      // this code uses np->parent without holding np->lock.
      // acquiring the lock first would cause a deadlock,
      // since np might be an ancestor(of p), and we already hold p->lock(i.e, the child's lock, p-> lock).
      if(np->parent == p){
        // np->parent can't change between the check and the acquire()
        // because only the parent changes it, and we're the parent.
        acquire(&np->lock);
        havekids = 1;
        if(np->state == ZOMBIE){
          // Found one.
          pid = np->pid;
          if(addr != 0 && copyout(p->pagetable, addr, (char *)&np->xstate,
                                  sizeof(np->xstate)) < 0) {
              /*这里使用copyout，不使用 *addr = np->xstate的理由也是一样的：
              proc结构体是内核空间的东西, addr是父进程地址空间的虚拟地址.
              跨地址空间复制值不能使用等号
              */
            release(&np->lock);
            release(&p->lock);
            return -1;
          }
          freeproc(np); //释放掉np的proc结构体
          release(&np->lock);
          release(&p->lock);
          return pid;
        }
        release(&np->lock);
      }
    }

    // No point waiting if we don't have any children.
    if(!havekids || p->killed){
      release(&p->lock);
      return -1;
    }
    
    // Wait for a child to exit.
    sleep(p, &p->lock);  //DOC: wait-sleep
  }
}
```



然后是`exit`：`exit`记录调用者的退出状态码，将其进程状态标记为`zombie`，释放一些资源，把调用者的子进程给`init`，唤醒父进程以防有父进程在wait，然后让出CPU。

`exit`在将进程设置为`ZOMBIE`并且唤醒父进程的时候，必须持有父进程的锁。因为父进程在醒来之前必须拿到自己的锁，而子进程此时持有这个锁，就可以防止父进程在`exit`设置状态的中途醒来，导致Lost Wakeup问题。 `exit`还要持有进程本身的锁，防止父进程在`exit`将进程设置为`ZOMBIE`之后，让出CPU之前，Free掉子进程。

> The exiting process must hold its parent’s lock while it sets its state to ZOMBIE and wakes the parent up, since the parent’s lock is the condition lock that guards against lost wakeups in wait. The child must also hold its own p->lock, since otherwise the parent might see it in state ZOMBIE and free it while it is still running

同样的，`exit`获取锁的顺序同样是先父后子，以避免死锁。



`exit`里存在的调用链：

- 调用`wakeup1`来唤醒父进程，没有使用`Wakeup`。
- 调用`reparent`来将可能有的子进程移交给`init`

先看`wakeup1`：要求`exit`在持有`proc ->lock`的前提下调用。传入的参数就是父进程的`proc`结构体的指针。思想也很简单，就是把父进程的状态设置为`RUNNABLE`（如果父进程处于`SLEEPING`）。

```c
// Wake up p if it is sleeping in wait(); used by exit().
// Caller must hold p->lock.
static void
wakeup1(struct proc *p)
{
  if(!holding(&p->lock))
    panic("wakeup1");
  if(p->chan == p && p->state == SLEEPING) {
    p->state = RUNNABLE;
  }
}
```

然后是`reparent`：用于将子进程都移交给`init`进程。传入的参数是要移交子进程的那个父进程。

- 其实基本思想也很简单，直接遍历整个进程表。对于遍历到的那个进程，如果其父进程是传入的参数`p`，那么就将其父进程改为`initproc`。

```c
// Pass p's abandoned children to init.
// Caller must hold p->lock.
void
reparent(struct proc *p)
{
  struct proc *pp;

  for(pp = proc; pp < &proc[NPROC]; pp++){
    // this code uses pp->parent without holding pp->lock.
    // acquiring the lock first could cause a deadlock
    // if pp or a child of pp were also in exit()
    // and about to try to lock p.
    if(pp->parent == p){
      // pp->parent can't change between the check and the acquire()
      // because only the parent changes it, and we're the parent.
      acquire(&pp->lock);
      pp->parent = initproc;
      // we should wake up init here, but that would require
      // initproc->lock, which would be a deadlock, since we hold
      // the lock on one of init's children (pp). this is why
      // exit() always wakes init (before acquiring any locks).
      release(&pp->lock);
    }
  }
}
```

最后就可以看一下`exit`：

- 总共调用了两次`wakeup1`，一次用于唤醒`init`（因为可能有子进程），另外一次用于唤醒父进程。

```c
// Exit the current process.  Does not return.
// An exited process remains in the zombie state
// until its parent calls wait().
void
exit(int status)
{
  struct proc *p = myproc();

  if(p == initproc)
    panic("init exiting");

  // Close all open files.
  for(int fd = 0; fd < NOFILE; fd++){
    if(p->ofile[fd]){
      struct file *f = p->ofile[fd];
      fileclose(f);
      p->ofile[fd] = 0;
    }
  }

  begin_op();
  iput(p->cwd);
  end_op();
  p->cwd = 0;

  // we might re-parent a child to init. we can't be precise about
  // waking up init, since we can't acquire its lock once we've
  // acquired any other proc lock. so wake up init whether that's
  // necessary or not. init may miss this wakeup, but that seems
  // harmless.
  acquire(&initproc->lock);
  wakeup1(initproc);
  release(&initproc->lock);

  // grab a copy of p->parent, to ensure that we unlock the same
  // parent we locked. in case our parent gives us away to init while
  // we're waiting for the parent lock. we may then race with an
  // exiting parent, but the result will be a harmless spurious wakeup
  // to a dead or wrong process; proc structs are never re-allocated
  // as anything else.
  acquire(&p->lock);
  struct proc *original_parent = p->parent;
  release(&p->lock);
  
  // we need the parent's lock in order to wake it up from wait().
  // the parent-then-child rule says we have to lock it first.
  acquire(&original_parent->lock);

  acquire(&p->lock);

  // Give any children to init.
  reparent(p);

  // Parent might be sleeping in wait().
  wakeup1(original_parent);

  p->xstate = status;
  p->state = ZOMBIE;

  release(&original_parent->lock);

  // Jump into the scheduler, never to return.
  sched();
  panic("zombie exit");
}
```



`Kill`所做的事很简单，就只是把`proc -> killed`置为1，并且将受害者进程唤醒（如果它在Sleep）。它并不会显式地立即把进程清理掉。因为受害者进程可能正在执行一些很敏感的要求原子性的操作（比如一系列要求原子性的硬盘写入，这样才能让文件系统处于正确的状态），所以`Kill`不会立即将受害者清理掉。

最终受害者是会进入内核或者从内核返回的，无论那种情况，`usertrap`中的代码都会检查`p->killed`，如果被置为1了就调用`exit`，这样受害者进程就会退出。如果受害者进程正在用户空间运行，那么至少它会因为时钟中断的出现而进入内核（所以不会出现受害者进程永远不进出内核的情况），然后调用`exit`退出。



> Some xv6 sleep loops do not check p->killed because the code is in the middle of a multistep system call that should be atomic. The virtio driver (kernel/virtio_disk.c:242) is an example: it does not check p->killed because a disk operation may be one of a set of writes that are all
>
> needed in order for the file system to be left in a correct state. A process that is killed while waiting for disk I/O won’t exit until it completes the current system call and usertrap sees the killed flag.

#### 7.9 Real world

Xv6只实现了RR，但是现实的OS还有许多其他复杂的调度策略。

 

优先级反转：当高优先级进程和低优先级进程共享一个锁时，如果低优先级的进程持有锁时，会阻塞高优先级的进程优先运行。

- 解决方法：优先级继承，先让低优先级但是持有锁的进程暂借高优先级进程的优先级，使得锁能够尽快释放。

车队效应（*convoys*）：许多高优先级的进程等待一个持有共享锁的低优先级进程释放锁，这些高优先级的进程就像被堵在高速上的车队。



几种处理 "lost wakeups" 问题的方法：

- 原始的 Unix 内核中的`sleep`简单地禁用了中断，这在单 CPU 系统中是可行。因为在单 CPU 系统中，禁用中断使得Sleep在CPU上运行的时候不会被打断，这样就不存在Sleep还没执行完就来了一个Wakeup的情况，因为Sleep一定可以执行完。
- 由于Xv6运行在多处理器系统上，它为`sleep`添加了一个显式的锁（p -> lock）。FreeBSD 的 "msleep" 采取了相同的方法
- Linux 内核的`sleep`使用一个显式的进程队列，称为等待队列（wait queue），而不是使用等待通道（wait channel）；这个队列有其自己的内部锁。



Xv6的Wakeup采用的方式是变量整个进程表来找到Channel匹配的进程，比较低效。一个更好的方法是，将睡眠在同一个Channel上的进程都放到一个队列里去，这就是等待队列。

许多线程库管Sleep-Wakeup这种玩意叫条件变量，这种语境下Sleep和Wakeup分别叫Wait和Signal。这些机制都有一个共同点：睡眠的条件被某种锁保护着，这个锁在Sleep的时候会自动释放掉。

> Scanning the entire process list in wakeup for processes with a matching chan is inefficient. A better solution is to replace the chan in both sleep and wakeup with a data structure that holds a list of processes sleeping on that structure, such as Linux’s wait queue.



Wakeup的实现是唤醒所有在这个Channel上睡眠的进程，假如进程不止一个的话，这些进程就需要竞争条件锁。

这种现象叫 *thundering herd*。 指的是当一个共享资源被唤醒后，所有等待该资源的进程都会被唤醒，并且它们会同时竞争检查资源是否可用。这种情况可能会导致不必要的竞争，浪费系统资源，并可能降低系统性能。

为了避免这个问题，大部分的条件变量针对`wakeup`有两个原语：`signal`和`broadcast`。前者只唤醒一个进程，后者唤醒全部的等待进程。



信号量的计数通常都可以人为赋予特殊的意义的。所赋予的意义有助于同步的实现。

> Semaphores are often used for synchronization. The count typically corresponds to something like the number of bytes available in a pipe buffer or the number of zombie children that a process has. Using an explicit count as part of the abstraction avoids the “lost wakeup” problem: there is an explicit count of the number of wakeups that have occurred. 



一个真正的操作系统在寻找可用的`struct proc`的时候是从Freelist寻找，只需要耗费常数时间（头部就是一个可用的），而Xv6则在`allocproc`中使用了线性时间的查找。

### Lecture 11

**所有的内核线程都共享内核的地址空间**。内核线程的作用之一是执行来自用户进程的系统调用请求。

> XV6内核共享了内存，并且XV6支持内核线程的概念，对于每个用户进程都有一个内核线程来**执行来自用户进程的系统调用**。所有的内核线程都共享了内核内存，所以XV6的内核线程的确会共享内存。

抢占式线程调度的基本实现方法，配合时钟中断：

> 在每个CPU核上，都存在一个硬件设备，它会定时产生中断。XV6与其他所有的操作系统一样，将这个中断传输到了内核中，并将程序运行的控制权从用户空间代码切换到内核中的中断处理程序。
>
> 位于内核的定时器中断处理程序，会自愿的将CPU出让（yield）给线程调度器，并告诉线程调度器说，你可以让一些其他的线程运行了。这里的出让其实也是一种线程切换，它会保存当前线程的状态，并在稍后恢复。
>
> 这里的基本流程是，定时器中断将CPU控制权给到内核，内核再自愿的出让CPU。
>
> 有趣的是，在XV6和其他的操作系统中，线程调度是这么实现的：定时器中断会强制的将CPU控制权从用户进程给到内核，这里是pre-emptive scheduling，之后内核会代表用户进程（注，**实际是内核中用户进程对应的内核线程会代表用户进程出让CPU**），使用voluntary scheduling。



需要注意的是，一个用户进程，它的用户空间的上下文是被保存在了Trapframe里，**而它的内核线程的上下文则被保存在了Context结构体里面**。

> 如果程序执行了一个系统调用或者因为响应中断走到了内核中，那么相应的用户空间状态会被保存在程序的trapframe中，同时属于这个用户程序的**内核线程被激活**。所以首先，用户的程序计数器，寄存器等等被保存到了trapframe中，之后CPU被切换到内核栈上运行，实际上会走到trampoline和usertrap代码中。之后内核会运行一段时间处理系统调用或者执行中断处理程序。在处理完成之后，如果需要返回到用户空间，trapframe中保存的用户进程状态会被恢复。

#### 先不考虑调度器，单纯看两个进程之间的切换

教授先讲述的是，没有考虑调度器在其中运行的角色的，简化一点的情况：

两个线程A和B之间的切换，实际上是：A的用户空间=>A的内核线程=>调度器线程=>B的内核线程=>返回B的用户空间。

但是这里，教授先讲述的是省略了其中的调度器线程部分的情况。

- A的内核线程在切换到B的内核线程的时候，A的内核线程的上下文实际上是被保存在了`context`结构体里面。而A的用户空间的上下文则保存在Trapframe。
- 而B的内核线程的恢复，就是从B对应的`context`结构体里面把寄存器的值回填到CPU里去。
- 所以目前为止有两类寄存器：用户寄存器存在trapframe中，内核线程的寄存器存在`context`结构体中。

> 在定时器中断程序中，如果XV6内核决定从一个用户进程切换到另一个用户进程，那么首先在内核中第一个进程的内核线程会被切换到第二个进程的内核线程。之后再在第二个进程的内核线程中返回到用户空间的第二个进程，这里返回也是通过恢复trapframe中保存的用户进程状态完成。

举个例子：C语言编译器CC，切换到Shell里的ls命令之间的切换

当XV6从CC程序的内核线程切换到LS程序的内核线程时：

1. XV6会首先会将CC程序的内核线程的内核寄存器保存在一个context对象中。
2. 类似的，因为要切换到LS程序的内核线程，那么LS程序现在的状态必然是RUNABLE，表明LS程序之前运行了一半。这同时也意味着LS程序的用户空间状态已经保存在了对应的trapframe中，更重要的是，**LS程序的内核线程对应的内核寄存器也已经保存在对应的context对象中**。所以接下来，XV6会**恢复LS程序的内核线程的context对象，也就是恢复内核线程的寄存器。**
3. 之后LS会**继续在它的内核线程栈上，完成它的中断处理程序**（注，假设之前LS程序也是通过定时器中断触发的pre-emptive scheduling进入的内核）。
4. 然后通过恢复LS程序的trapframe中的用户进程状态，返回到用户空间的LS程序中。
5. 最后恢复执行LS。

这里想强调一下3处，ls的内核线程恢复之后，也会从它被中断的地方继续完成下去。其实我们可以想象，ls的内核线程被中断的地方，和CC是一个地方，都是切换线程的那个临界点的那里（大概就是汇编写的函数`Swch`那里）。

#### 然后把调度器加上去

然后再把调度器线程也考虑进去。

完整的故事应该是这样的：

假如我们有两个线程A和B：

1. 一个定时器中断强迫CPU从用户空间进程切换到内核，trampoline代码将A的用户寄存器保存于用户进程A对应的trapframe对象中；
2. 之后在内核中运行usertrap，来实际执行相应的中断处理程序。这时，CPU正在进程A的内核线程和内核栈上，执行内核中的C代码；
3. A的内核线程决定让出CPU，它会做很多工作，但是最后它会调用swtch函数（switch 是C 语言关键字，因此这个函数命名为swtch 来避免冲突）
4. swtch函数会保存A**对应内核线程的寄存器**至context对象。但是，swtch函数**并不是直接从一个内核线程切换到另一个内核线程**，而是切换到本CPU的调度器线程（调度器线程的Context保存在模拟的CPU的结构体里面）。
5. 调度器线程运行`Schedule`函数：假定它找到的是B，然后它做如下工作。
   1. 先保存自己的寄存器到调度器线程的context对象
   2. 找到进程B之前保存的context，恢复其中的寄存器。也就是恢复B的内核线程
   3. 因为进程B在进入RUNABLE状态之前，如进程A一样，必然也调用了swtch函数。所以之前的swtch函数会被恢复，并返回到进程B所在的系统调用或者中断处理程序中（注，因为B进程之前调用swtch函数必然在系统调用或者中断处理程序中）。
   4. 不论是系统调用还是中断处理程序，在从用户空间进入到内核空间时会保存用户寄存器到trapframe对象。所以当内核程序执行完成之后，trapframe中的用户寄存器会被恢复。
   5. Trapframe中的寄存器恢复之后，最后用户进程B就恢复运行了

每个CPU都有自己的调度器线程，调度器线程也是一种内核线程，它也有自己的context对象。

在XV6的代码中，context对象总是由swtch函数产生，所以context总是保存了内核线程在执行swtch函数时的状态。**当我们在恢复一个内核线程时，对于刚恢复的线程所做的第一件事情就是从之前的swtch函数中返回**（也就是那个原则：总是从自己被打断的地方捡起来继续执行）。



在Scheduler函数中，锁的作用有哪些：

> 首先，出让CPU涉及到很多步骤，我们需要将进程的状态从RUNNING改成RUNABLE，我们需要将进程的寄存器保存在context对象中，并且我们还需要停止使用当前进程的栈。所以这里至少有三个步骤，而这三个步骤需要花费一些时间。所以锁的第一个工作就是**在这三个步骤完成之前，阻止任何一个其他核的调度器线程看到当前进程**。锁这里**确保了三个步骤的原子性**。从CPU核的角度来说，三个步骤要么全发生，要么全不发生。
>
> 第二，当我们开始要运行一个进程时，p->lock也有类似的保护功能。当我们要运行一个进程时，我们需要将进程的状态设置为RUNNING，我们需要将进程的context移到RISC-V的寄存器中。但是，如果在这个过程中，发生了中断，从中断的角度来说进程将会处于一个奇怪的状态。比如说进程的状态是RUNNING，但是又还没有将所有的寄存器从context对象拷贝到RISC-V寄存器中。所以，如果这时候有了一个定时器中断将会是个灾难，因为我们可能在寄存器完全恢复之前，从这个进程中切换走。而从这个进程切换走的过程中，将会保存不完整的RISC-V寄存器到进程的context对象中。所以我们希望启动一个进程的过程也具有原子性。在这种情况下，切换到一个进程的过程中，也需要获取进程的锁以确保其他的CPU核不能看到这个进程。同时在切换到进程的过程中，还需要关闭中断，这样可以避免定时器中断看到还在切换过程中的进程。（注，这就是为什么468行需要加锁的原因）



为什么`swtch`只保存和加载寄存器？

> 线程除了寄存器以外的还有很多其他状态，它有变量，堆中的数据等等，但是所有的这些数据都在内存中，并且会保持不变。我们没有改变线程的任何栈或者堆数据。所以线程切换的过程中，**处理器中的寄存器是唯一的不稳定状态**，且需要保存并恢复。**而所有其他在内存中的数据会保存在内存中不被改变，所以不用特意保存并恢复。**

> 学生提问：为什么不能将context对象保存在进程对应的trapframe中？
>
> Robert教授：context可以保存在trapframe中，因为每一个进程都只有一个内核线程对应的一组寄存器，我们可以将这些寄存器保存在任何一个与进程一一对应的数据结构中。对于每个进程来说，有一个proc结构体，有一个trapframe结构体，所以我们可以将context保存于trapframe中。但是或许出于简化代码或者让代码更清晰的目的，trapframe还是只包含进入和离开内核时的数据。而context结构体中包含的是在内核线程和调度器线程之间切换时，需要保存和恢复的数据。

老师提问：为什么RISC-V中有32个寄存器，但是swtch函数中只保存并恢复了14个寄存器？

> 学生回答：因为switch是按照一个普通函数来调用的，对于有些寄存器，swtch函数的调用者默认swtch函数会做修改，所以调用者已经在自己的栈上保存了这些寄存器，当函数返回时，这些寄存器会自动恢复。所以swtch函数里只需要保存Callee Saved Register就行。（注，详见5.4）
>
> 老师回答：
>
> 完全正确！因为swtch函数是从C代码调用的，所以我们知道**Caller Saved Register会被C编译器保存在当前的栈上**。Caller Saved Register大概有15-18个，而我们在swtch函数中只需要处理C编译器不会保存，但是对于swtch函数又有用的一些寄存器。所以在切换线程的时候，我们只需要保存Callee Saved Register。



#### Context Switching术语的解释

> 当人们在说context switching，他们通常说的是从一个线程切换到另一个线程，因为在切换的过程中需要先保存前一个线程的寄存器，然后再恢复之前保存的后一个线程的寄存器，这些寄存器都是保存在context对象中。在有些时候，context switching也指从一个用户进程切换到另一个用户进程的完整过程。偶尔你也会看到context switching是指从用户空间和内核空间之间的切换。对于我们这节课来说，context switching主要是指一个内核线程和调度器线程之间的切换。

#### 提问：Linux是如何实现进程内的多线程的（Xv6里一个进程只有一个线程）

省流版：可以认为Linux下的每个线程都是一个完整的进程，而一个进程中的多个线程，本质上是共享同一块内存的多个独立进程。

> Robert教授：Linux是支持一个进程包含多个线程，Linux的实现比较复杂，或许最简单的解释方式是：几乎可以认为Linux中的每个线程都是一个完整的进程。Linux中，我们平常说一个进程中的多个线程，本质上是共享同一块内存的多个独立进程。所以Linux中一个进程的多个线程仍然是通过一个内存地址空间执行代码。如果你在一个进程创建了2个线程，那基本上是2个进程共享一个地址空间。之后，调度就与XV6是一致的，也就是针对每个进程进行调度。

#### 提问：一个进程中的多个线程会有相同的page table？

一个进程内的多个线程，他们的页表的内容肯定是一样的，至于是每个线程独立页表，还是所有线程共享页表，我个人认为出于节省空间的考虑，应该是后者。

但是由于每个线程也有自己的私有存储（比如局部变量，比如栈），所以出于线程隔离的安全考虑，应该不是完全共享页表的，或许是共享某些页表条目？

> 是的，如果你在Linux上，你为一个进程创建了2个线程，我不确定它们是不是共享同一个的page table，还是说它们是不同的page table，但是内容是相同的。
>
> 学生提问：有没有原因说这里的page table要是分开的？
>
> Robert教授：我不知道Linux究竟用了哪种方法。

#### 提问：第一次Swtch调用是哪里来的？

> 学生提问：当调用swtch函数的时候，实际上是从一个线程对于switch的调用切换到了另一个线程对于switch的调用。所以线程第一次调用swtch函数时，需要伪造一个“另一个线程”对于switch的调用，是吧？因为也不能通过swtch函数随机跳到其他代码去。
>
> Robert教授：是的。我们来看一下第一次调用switch时，“另一个”调用swtch函数的线程的context对象。proc.c文件中的allocproc函数会被启动时的第一个进程和fork调用，allocproc会设置好新进程的context



### Lecture 13

首先先强调的一点是，在`swtch`上的锁的释放有一点特殊，这也是得益于`swtch`的特殊性：在调用switch函数之前，总是会先获取线程对应的用户进程的锁。一个进程先获取自己的锁，然后调用switch函数切换到调度器线程，调度器线程再释放进程锁。

要强调的点就是，进程获取的自己的锁，其实是由`swtch`释放的。

实际上的代码顺序更像这样：

1. 一个进程出于某种原因想要进入休眠状态，比如说出让CPU或者等待数据，它会先获取自己的锁；
2. 之后进程将自己的状态从RUNNING设置为RUNNABLE；
3. 之后进程调用switch函数，其实是调用sched函数在sched函数中再调用的switch函数；
4. switch函数将当前的线程切换到调度器线程；
5. 调度器线程之前也调用了switch函数，现在恢复执行会从自己的switch函数返回；
6. 返回之后，调度器线程会释放刚刚出让了CPU的进程的锁 

在进程切换的最开始，进程先获取自己的锁，并且直到调用switch函数时也不释放锁。而另一个线程，也就是调度器线程会在进程的线程完全停止使用自己的栈之后，再释放进程的锁。释放锁之后，就可以由其他的CPU核再来运行进程的线程



XV6中，不允许进程在执行switch函数的过程中，持有任何其他的锁（这个禁止加锁是通过在`sched`的代码里先做一些条件检查来实现的。）。所以，进程在调用switch函数的过程中，必须要持有p->lock（也就是进程对应的proc结构体中的锁），但是**同时又不能持有任何其他的锁**。



举个例子说明为什么需要这样：

- 假如我们有两个进程A和B，只有一个CPU，还有一个额外的锁L。并且A的内核线程持有了L。
- A执行`Swtch`让出CPU，调度器选择了B的内核线程恢复。这会导致进程 A 持有了锁，但是进程 A 又不在运行
- B恰好也因为需要执行一个系统调用而需要获取锁L。假如L是自旋锁，那么B会一直在CPU上自旋，Acquire会一直占用CPU自旋。
- 而 A 又不在运行，且因为B的Acquire一直占用CPU（并且Acquire是关闭了中断的），A没有机会使用CPU并释放掉L（因为定时器中断被关掉了），死锁形成。

> 学生提问：难道定时器中断不会将CPU控制切换回进程P1从而解决死锁的问题吗？
>
> Robert教授：首先，所有的进程切换过程都发生在内核中，所有的acquire，switch，release都发生在内核代码而不是用户代码。实际上XV6允许在执行内核代码时触发中断，如果你查看trap.c中的代码你可以发现，如果XV6正在执行内核代码时发生了定时器中断，中断处理程序会调用yield函数并出让CPU。
>
> 但是在之前的课程中我们讲过**acquire函数在等待锁之前会关闭中断，否则的话可能会引起死锁**（注，详见10.8），所以我们不能在等待锁的时候处理中断。所以如果你查看XV6中的acquire函数，你可以发现函数中第一件事情就是关闭中断，之后再“自旋”等待锁释放。你或许会想，为什么不能先“自旋”等待锁释放，再关闭中断？因为这样会有一个短暂的时间段锁被持有了但是中断没有关闭，在这个时间段内的设备的中断处理程序可能会引起死锁。
>
> 所以不幸的是，当我们在自旋等待锁释放时会关闭中断，进而阻止了定时器中断并且阻止了进程P2将CPU出让回给进程P1

#### Sleep&Wakeup

sleep和wakeup函数只是接收表示了sleep channel的64bit数值，它们**并不关心这个数值代表什么**。当我们调用sleep函数时，我们通过一个sleep channel表明我们等待的特定事件，当调用wakeup时我们**传入相同的数值**来表明想唤醒哪个线程。



什么是Lost Wakeup（唤醒丢失？）：简而言之就是，在进程A调用Sleep将自己的进程状态设置为`SLEEPING`之前，但是已经`release`放锁之后，另外一个进程B调用了Wakeup意图唤醒A，这时Wakeup函数会找不到A（因为A的状态还是`RUNNING`），而A也会照常进入`SLEEPING`状态，A就错过了一次被唤醒的机会。

原因本质上是，存在一个锁已经被释放了但是A还是没有`SLEEPING`的时间窗口。

消除Lost Wakeup的方法就是，原子性地将进程设置成SLEEPING状态，同时释放锁。这样就不存在锁被释放了但是进程还没有进入到SLEEPING状态的时间窗口。

具体的消除措施就是，Sleep在释放条件锁**之前**，先获取进程锁，Wakeup也是先获取进程锁再检查进程是否是`SLEEPING`。这样就可以确保Wakeup要么是在进程被Sleep标记为`SLEEPING`之后，才能去试图唤醒这个进程。

#### exit系统调用

在XV6中，一个进程如果退出的话，我们需要释放用户内存，释放page table，释放trapframe对象，将进程在进程表单中标为REUSABLE，这些都是典型的清理步骤。当进程退出或者被杀掉时，有许多东西都需要被释放。

这里会产生的两大问题：

- 首先我们不能直接单方面的摧毁另一个线程，因为：另一个线程可能正在另一个CPU核上运行，并使用着自己的栈；也可能另一个线程正在内核中持有了锁；也可能另一个线程正在更新一个复杂的内核数据，如果我们直接就把线程杀掉了，我们可能在线程完成更新复杂的内核数据过程中就把线程杀掉了。我们不能让这里的任何一件事情发生。
- 另一个问题是，即使一个线程调用了exit系统调用，并且是自己决定要退出。它仍然持有了运行代码所需要的一些资源，例如它的栈，以及它在进程表单中的位置。当它还在执行代码，它就不能释放正在使用的资源。所以我们需要一种方法让线程能释放最后几个对于运行代码来说关键的资源。

`exit`所做的事：

- 释放进程的内存和page table，关闭已经打开的文件。
- 如果要退出的这个进程，它有子进程，那么**需要设置这些子进程的父进程为`init`进程**
- 唤醒可能调用了wait的父进程（不管有没有，无脑唤醒父进程就是了，反正无害的）。
- 将进程的状态设置为`ZOMBIE`，也就是僵尸进程。这个时候进程不在会被运行，但是资源还没有被完全地释放。
- 调用`sched`，也就是让调度器接手

#### wait系统调用

当一个进程调用了wait系统调用，它会扫描进程表，找到父进程是自己且状态是ZOMBIE的进程。从上一节可以知道，这些子进程已经在exit函数中几乎都把资源释放地差不多了。所以在这里，由父进程调用的`freeproc`函数，来完成释放进程资源的最后几个步骤。

`freeproc`做的事：

- 回收陷阱帧的物理页（把物理页添加到Freelist里去）
- 回收页表
- 把进程表中与子进程对应的`proc`结构体项，每个域都清零，把进程状态修改为`UNUSED`。



问题：如果父进程不回收呢？那么谁回收？

根据GPT的回答来说，父进程不回收的话，就没有人会回收了，所以父进程必须定期回收。

> 在Unix-like操作系统中，当一个进程执行了`exit`系统调用后，它会变成一个僵尸进程（Zombie Process）。这种状态发生在子进程比父进程更早地终止，但是父进程还没有来得及收集子进程的退出状态。
>
> 虽然僵尸进程本身不再执行任何代码，但它仍然在进程表中存在，以便父进程可以在之后通过`wait`或`waitpid`系统调用来获取子进程的退出状态。父进程调用这些系统调用后，操作系统会释放僵尸进程的资源，包括进程表中的条目和相关的资源，如进程号等。
>
> 因此，僵尸进程的资源由其父进程负责释放。父进程可以定期地调用`wait`或`waitpid`来回收已经终止的子进程，并且在成功收集到子进程退出状态后，操作系统会将僵尸进程从进程表中移除，释放其相关资源。**如果父进程没有及时回收僵尸进程，那么系统会继续保留这些僵尸进程的资源，这可能会导致系统的进程表资源耗尽。**
>
> 所以，要避免出现大量的僵尸进程，父进程应该定期地调用`wait`或`waitpid`来回收子进程，并释放它们的资源。



这也解释了，为什么要把终止的进程的子进程，托管给`init`进程，因为这样`init`就可以调用`wait`来回收他们的资源。

> 在Unix中，**对于每一个退出的进程，都需要有一个对应的wait系统调用**，这就是为什么当一个进程退出时，它的子进程需要变成init进程的子进程。**init进程的工作就是在一个循环中不停调用wait，因为每个进程都需要对应一个wait，这样它的父进程才能调用freeproc函数，并清理进程的资源。**

当父进程完成了清理进程的所有资源，子进程的状态会被设置成UNUSED。之后，fork系统调用才能重用进程在进程表单的位置。

#### 提问：Wait相关

> 学生提问：在exit系统调用中，为什么需要在重新设置父进程之前，先获取当前进程的父进程？
>
> Robert教授：这里其实就是在防止一个进程和它的父进程同时退出。通常情况下，一个进程exit，它的父进程正在wait，一切都正常。但是也可能一个进程和它的父进程同时exit。所以当子进程尝试唤醒父进程，并告诉它自己退出了时，父进程也在退出。这些代码我一年前还记得是干嘛的，现在已经记不太清了。它应该是处理这种父进程和子进程同时退出的情况。如果不是这种情况的话，一切都会非常直观，子进程会在后面通过wakeup函数唤醒父进程。
>
> 学生提问：为什么我们在唤醒父进程之后才将进程的状态设置为ZOMBIE？难道我们不应该在之前就设置吗？
>
> Robert教授：正在退出的进程会先获取自己进程的锁，同时，因为父进程的wait系统调用中也需要获取子进程的锁，所以父进程并不能查看正在执行exit函数的进程的状态。这意味着，正在退出的进程获取自己的锁到它调用sched进入到调度器线程之间（注，因为调度器线程会释放进程的锁），父进程并不能看到这之间代码引起的中间状态。所以这之间的代码顺序并不重要。大部分时候，如果没有持有锁，exit中任何代码顺序都不能工作。因为有了锁，代码的顺序就不再重要，因为父进程也看不到进程状态。

#### Kill系统调用

kill系统调用**不能就直接停止目标进程的运行**。实际上，在XV6和其他的Unix系统中，kill系统调用基本上不做任何事情。

原因：在wait一节已经说过，那就是我们无法确定被Kill的那个进程，在它被Kill的时候正在做什么。比如它也有可能正在完成磁盘写入，如果Kill直接就把这个进程给杀死，那么写入就是部分完成的，会破坏操作的原子性。



Kill只做了下面的事情：先扫描进程表，找到目标进程。然后**只是**将进程的proc结构体中`killed`标志位设置为1。如果进程正在`SLEEPING`状态，将其设置为`RUNNABLE`。这里只是将killed标志位设置为1，**并没有停止进程的运行**。所以kill系统调用本身还是很温和的。

而目标进程运行到内核代码中能安全停止运行的位置时，会检查自己的killed标志位，如果设置为1，目标进程会自愿的执行exit系统调用。

- 大部分的可以安全停止运行的位置，应该都在`trap.c`里。

#### 提问：Kill能否杀掉所有进程

> 学生提问：为什么一个进程允许kill另一个进程？这样一个进程不是能杀掉所有其他进程吗？
>
> Robert教授：如果你在MIT的分时复用计算机Athena上这么做的话，他们可能会开除你。在XV6中允许这么做是因为，XV6这是个教学用的操作系统，任何与权限相关的内容在XV6中都不存在。在Linux或者真正的操作系统中，每个进程都有一个user id或多或少的对应了执行进程的用户，一些系统调用使用进程的user id来检查进程允许做的操作。所以在Linux中会有额外的检查，调用kill的进程必须与被kill的进程有相同的user id，否则的话，kill操作不被允许。所以，在一个分时复用的计算机上，我们会有多个用户，我们不会想要用户kill其他人的进程，这样一套机制可以防止用户误删别人的进程。



## Lecture 14

### 阅读Chapter 8 File System(除了Logging相关的节，在下一节课)

文件系统的目的是什么？存储和组织数据，持久化数据以便重启后还能访问。



文件系统的挑战：

- 文件系统的硬盘上的数据结构必须要能反映目录和文件的树形结构。
- 必须能够记录存储一个文件内容的块是哪些。必须记录哪些块是Free的
- 支持*crash recovery*。

> The file system must support *crash recovery*. That is, if a crash (e.g., power failure) occurs, the file system must still work correctly after a restart. The risk is that a crash might interrupt a sequence of updates and leave inconsistent on-disk data structures (e.g., a block that is both
>
> used in a file and marked free).

- 支持并发。
- 因为硬盘访问速度指数级别的低于内存访问，所以文件系统需要在内存给热点块维护一个缓存。

#### 8.1 Overview

自底向上：

- 硬盘层读写硬盘块
- 缓冲层缓存硬盘块，并且还要同步对他们的访问，也即确保一次只有一个内核进程能够修改存储在某个特定块的数据
- 日志层：将更上层对多个块的更新整合为一个事务，确保即便发生崩溃时，事务的原子性（也即全部或者零）。
- inode层没什么好说的
- 目录层负责实现目录的
- 路径名层负责提供层次化的路径名，以及路径的解析服务
- 文件描述符层把Unix的文件资源抽象出来，对应用层编程提供的接口。

<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230724154626239.png" alt="image-20230724154626239" style="zoom:67%;" />





文件系统在硬盘上的结构：

- 文件系统不使用Block 0，因为那里是Bootloader
- Block1是*Super Block*（超级块），存储了文件系统本身的元数据。
  - 有很多，比如单个块的大小，文件系统以块为单位的总大小，iNode的个数，等等
- 从Block2开始，放的是Log
- Log后面是一个iNode数组（借用一下内存的数组的概念），每个块会有多个INode。
- INodes后面是Bit Map，用于跟踪哪些块在使用中，哪些块是Free的
- 剩余就是数据块

**超级块是被一个单独的程序填充的，`mkfs`，用于建造和初始化一个文件系统**。



<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230724155759677.png" alt="image-20230724155759677" style="zoom:67%;" />



接下来自顶向上从Buffer Cache Layer开始逐层讲述。很容易会注意到，下层选择的良好抽象是如何减轻上层的设计负担的。

> Look out for situations where **well-chosen abstractions at lower layers ease the design of higher ones.**



超级块在Xv6上是一个`struct superblock`：没什么特别需要解释的。

```c
// Disk layout:
// [ boot block | super block | log | inode blocks |
//                                          free bit map | data blocks]
//
// mkfs computes the super block and builds an initial file system. The
// super block describes the disk layout:
struct superblock {
  uint magic;        // Must be FSMAGIC
  uint size;         // Size of file system image (blocks)
  uint nblocks;      // Number of data blocks
  uint ninodes;      // Number of inodes.
  uint nlog;         // Number of log blocks
  uint logstart;     // Block number of first log block
  uint inodestart;   // Block number of first inode block
  uint bmapstart;    // Block number of first free map block
};

#define FSMAGIC 0x10203040
```



#### 8.2 Buffer cache layer

Buffer Cache负责两个工作：

1. synchronize access to disk blocks to ensure that only one copy of a block is in memory and that only one kernel thread at a time uses that copy
2. cache popular blocks so that they don’t need to be re-read from the slow disk. 

代码在`bio.c`



Buffer cache layer对外暴露的两个接口：

- `bread`：获取一个*struct buf*，这个*buf*在内存中存放了一个能被读写的，某个硬盘Block的副本。

> a *buf* containing a copy of a block which can be read or modified in memory

- `bwrite`：将一个被修改过的Buffer写回硬盘

一个内核线程在使用完某个Buffer以后，调用`brelse`来释放这个Buffer。Buffer Cache层对每个Buffer都使用了一个Sleep Lock来确保一次只能有一个内核线程使用每个Buffer（以及对应的每个硬盘块）。

`bread`返回一个锁上的Buffer，而`brelse`释放那个锁。



Buffer Cache层维护的是固定个数的Buffers，每个Buffer缓存一个Disk Block。所以这就意味着存在缓存替换，因为缓存的个数是有限的，所以当我们要把某个不在缓存的Disk Block读取进来，而缓存用完时，必然需要替换掉某个现有缓存。

Buffer Cache使用LRU算法作为缓存替换策略



#### 8.3 Code: Buffer cache

Buffer Cache是一个由`struct buf`组成的双向链表。`struct buf`的结构在`buf.h`。

- 每个Disk Block的大小是1024字节，也即1KB。所以每个*buf*中的缓冲区(*buf.data*)的大小也是1KB。
- `lock`用于同步和串行化并发的访问。

```c
/*fs.h*/
#define BSIZE 1024  // block size
/*buf.h */
struct buf {
  int valid;   // has data been read from disk?
  int disk;    // does disk "own" buf?
  uint dev;    //设备号?
  uint blockno; //块号?
  struct sleeplock lock;
  uint refcnt; //应该是引用计数,Reference Count
  struct buf *prev; // LRU cache list
  struct buf *next; //指针没什么好说的,构建链表用的, prev是previous,前向指针.
  uchar data[BSIZE]; //真正存放Disk Block数据的地方
};
```

在Xv6启动的时候，`main`调用了`binit`来初始化一个长度为*NBUF*的`struct buf`数组。这个数组才是实际上的缓存，但是外部对于Buffer Cache的访问，都是通过链表的方式（去引用`bcache.head`），**而不是直接去访问那个数组**。所以一开始才说Buffer Cache是一个双向链表。

之所以使用这种链表配合数组的方式，应该是为了方便地实现LRU。

**他这里的双向链表是没有使用一头一尾的两个哨兵节点的（head其实是尾哨兵节点），并且是一个回环的，也就是说，最后一个节点的Next会回环指向第一个节点，而第一个节点的Prev会回环指向最后一个节点**。

而且这里代码的构造方式很奇怪，我习惯上是让Next从左往右指，但是这里Next是从右往左指的，`head`变成了最后一个节点。。

举个例子，1,2,3的插入：

- Next指针方向上：head <- 1 <- 2 <- 3
- Prev指针方向上：head -> 1 -> 2 -> 3

**也正因为这种回环链表的结构(以及它违反我的习惯的指针指向)，假如我们新插入一个Buffer是插入在head.next的话，那么head.next就刚好指向了最近使用过的Buffer**，同样的道理，随着我们不断插入新的节点，第一次插入的Buffer就会被逐渐挤压到离head越来越远，也就意味着它是最近最少使用的那个Buffer，**head.prev就刚好指向了它！非常的妙**。LRU就这么实现了！

以一种形象化的，动态的方式来说就是，随着新的节点不断在head右边挤进去，最右侧的那个节点会被挤得越来越靠右，越来越靠右就意味着它越来越久没使用。所以我们可以以“离head的距离”的远近来衡量其最近使用的频率。因为假如一个靠右的节点再次被使用，它会被移除然后重新添加到`head.next`去。

所以假如我们用尽了30个Buffer，需要选一个做替换的时候，我们直接选择`head.prev`指向的那个Buffer，在数组`bcache.buf`里找到`head.prev`那个对应的*struct buf*单元，直接覆盖就行了！

```c
/*parameters.h*/
#define MAXOPBLOCKS  10  // max # of blocks any FS op writes
#define NBUF         (MAXOPBLOCKS*3)  // size of disk block cache
/*bio.c*/
//Buffer Cache
struct {
  struct spinlock lock;
  struct buf buf[NBUF];

  // Linked list of all buffers, through prev/next.
  // Sorted by how recently the buffer was used.
  // head.next is most recent, head.prev is least.
  struct buf head;
} bcache;

void
binit(void)
{
  struct buf *b;

  initlock(&bcache.lock, "bcache");

  // Create linked list of buffers
  bcache.head.prev = &bcache.head; //最开始的时候，它自己就既是第一个节点，也是最后一个节点，所以prev和next都指回来了。
  bcache.head.next = &bcache.head;
  for(b = bcache.buf; b < bcache.buf+NBUF; b++){
    b->next = bcache.head.next; 
    b->prev = &bcache.head; //此时b是第一个节点，其prev要回环指向最后一个节点head(如果把图画出来，head是next指针的终点)
    initsleeplock(&b->lock, "buffer");
    bcache.head.next->prev = b; 
    bcache.head.next = b; //head是最后一个节点，其next指针应该指向新的第一个节点
  }
}
```

然后是`bread`：`bread`调用了`bget`来为给定扇区获取一个Buffer，如果需要从硬盘读取，`bread`调用`virtio_disk_rw` 

可以看到`bread`本身还是比较简单的。先调用`bget`获取一个*struct buf*，然后检查其是否合法，不合法就从硬盘读。最后返回。

```c
// Return a locked buf with the contents of the indicated block.
struct buf*
bread(uint dev, uint blockno)
{
  struct buf *b;

  b = bget(dev, blockno);
  if(!b->valid) {
    virtio_disk_rw(b, 0);
    b->valid = 1;
  }
  return b;
}
```

`bget`则会遍历一遍表来找具有匹配扇区号和设备号的那个Buffer。如果没找到就说明这是第一次分配，分配一个新的Buffer给它。然后返回。

先释放掉`bcache.lock`然后再去获取单个Buffer的SleepLock是安全的，因为那个Buffer的引用计数是1，会阻止其他Disk Block去使用这个Buffer。

```c
// Look through buffer cache for block on device dev.
// If not found, allocate a buffer.
// In either case, return locked buffer.
static struct buf*
bget(uint dev, uint blockno)
{
  struct buf *b;

  acquire(&bcache.lock);

  // Is the block already cached?
  for(b = bcache.head.next; b != &bcache.head; b = b->next){ //遍历一遍。因为指针是回环的,所以一直遍历到到底会回到head
    if(b->dev == dev && b->blockno == blockno){ //如果设备号和扇区号都匹配上了,就是找到了
      b->refcnt++; //引用计数 ++
      release(&bcache.lock);
      acquiresleep(&b->lock);
      return b;//拿到这个Buffer的睡眠锁，然后return b
    }
  }

  // Not cached.
  // Recycle the least recently used (LRU) unused buffer.
  for(b = bcache.head.prev; b != &bcache.head; b = b->prev){ //从最右边的开始遍历,因为离得越远使用频率越低
    if(b->refcnt == 0) {
      b->dev = dev;
      b->blockno = blockno;
      b->valid = 0; /*将合法位置为0,用以说明Buffer里面的内容是前任Block的，要去硬盘读取现任的。
        确保bread会去硬盘读取，而不是直接使用原有的Buffer内容*/
      b->refcnt = 1;
      release(&bcache.lock);
      acquiresleep(&b->lock);
      return b;
    }
  }
  panic("bget: no buffers");
}
```

一旦调用了`bread`拿到了一个Buffer以后，调用者就独占这个Buffer。如果调用者修改了Buffer里面的内容，它在**返还Buffer之前必须**调用`bwrite`把缓冲区的内容写回硬盘。`Bwrite`也调用`virtio_disk_rw`来与硬盘硬件交流。

代码就没什么好说的了。

```c
// Write b's contents to disk.  Must be locked.
void
bwrite(struct buf *b)
{
  if(!holdingsleep(&b->lock))
    panic("bwrite");
  virtio_disk_rw(b, 1);
}
```

在使用完Buffer打算返还的时候，调用`brelse`。`brelse`把SleepLock释放，然后把Buffer放到链表的最右侧。

```c
// Release a locked buffer.
// Move to the head of the most-recently-used list.
void
brelse(struct buf *b)
{
  if(!holdingsleep(&b->lock))
    panic("brelse");

  releasesleep(&b->lock);

  acquire(&bcache.lock);
  b->refcnt--; //引用计数减一
  if (b->refcnt == 0) { //如果引用计数==0说明没人需要它了
    // no one is waiting for it.
    //这里是把b的前一个节点以及b的后一个节点连接起来,不然链表就在b的位置断了
      //举个例子, A <- b <- C <-.... 。把b挪到最右侧，还要把A和C给连起来。
    b->next->prev = b->prev; //A就是b.next,C就是b.prev。这里就是A.prev = C
    b->prev->next = b->next; // C.next = A
    //下面几行就是把b放到最右侧去。因为是个回环双向链表,所以b.prev要回环到head,同理head.next回环到b
    b->next = bcache.head.next; 
    b->prev = &bcache.head;
    bcache.head.next->prev = b;
    bcache.head.next = b;
  }
  
  release(&bcache.lock);
}
```

关于这一点，原文在最后一段也有叙述，这里直接贴上来：

>  Moving the buffer causes the list to be ordered by how recently the buffers were used (meaning released): the first buffer in the list is the most
>
> recently used, and the last is the least recently used. The two loops in *bget* take advantage of this: the scan for an existing buffer must process the entire list in the worst case, but checking the most recently used buffers first (starting at *bcache.head* and following next pointers) will reduce scan time when there is good locality of reference. The scan to pick a buffer to reuse picks the least recently used buffer by scanning backward (following prev pointers).



#### 8.7 Code: Block allocator

Xv6的Block分配器维护了一个BitMap，每个Block对应一个Bit。0是Free，1是Used。

`mkfs`会将BitMap上与Boot扇区，超级块、Log块、iNode块以及BitMap块对应的Bit都置为1



两个函数：`balloc`分配，`bfree`回收。

`BBLOCK`是一个宏：对于某个块号为b的Block ，计算其在BitMap上对应的Bit所在的块号。

- 基本思想就是：BitMap起始块号 + b / 单个BitMap Block上的Bit数。除法计算其在第几块BitMap Block，再加上第0块BitMap Block的块号，就是要得到的块号。

```c
// Block of free map containing bit for block b
#define BBLOCK(b, sb) ((b)/BPB + sb.bmapstart)
```

`balloc`外层循环是遍历全部的*BitMap Block*（所以b不是每个循环自加1，而是`b += BPB`），然后对于每个遍历到的Block，在内层循环检查*BitMap*上的Bit是否是0。是的话，说明找到了一个*Free Block*，把这个Bit置为1，同时清零与这个Bit对应的*Data Block*。

**其实需要注意一下的是，`bzero`没有调用`bwrite`把清零写回硬盘，也就是说，清零的实际上是与*Free Block*对应的内存里的Buffer** 。其实也没必要把清零写回硬盘，因为清零只是准备工作罢了，真正要写的东西根本还没开始写。

> The outer loop reads each block of bitmap bits. The inner loop checks all BPB bits in a single bitmap block

```c
/*fs.h*/
// Bitmap bits per block
#define BPB           (BSIZE*8) //BSIZE是以字节为单位的，BitMap是Bit为单位，需要换算
#define BSIZE 1024  // block size
/*fs.c*/
// Allocate a zeroed disk block.
static uint
balloc(uint dev)
{
  int b, bi, m;
  struct buf *bp;

  bp = 0;
    /*
    b可以看做是第b块Data Block。而不是块号为b的Data Block
    因为BitMap上的第N个Bit，对应的应该就是第N个Data Block的使用情况
    所以很自然的，第0到第[BPB - 1]块Data Block，他们的使用情况都记录在第0个 BitMap Block上
    这也是为什么b是以BPB为单位加的
    因为第 0 到 BPB - 1号Data Block对应的BitMap Block是第0块
    然后第 BPB 到 2BPB - 1 号Data Block对应第1块 BitMap Block
    而我们只需要用b + DataBlock的起始块号(b + sb.)，就可以知道第b块Data Block的实际块号是什么
    */
  for(b = 0; b < sb.size; b += BPB){ 
    bp = bread(dev, BBLOCK(b, sb)); //把块号为b对应的BitMap Block读取到buf缓存。
    for(bi = 0; bi < BPB && b + bi < sb.size; bi++){  //对于bp里的每一个比特对应的那块Block，检查其是否是Free的
      m = 1 << (bi % 8);
      if((bp->data[bi/8] & m) == 0){  // Is block free?
        bp->data[bi/8] |= m;  // Mark block in use.
        log_write(bp);
        brelse(bp); //释放掉缓存Bit Map Block的那个Buffer。
        bzero(dev, b + bi); //把找到的那一块清零(实际上清零的是与这一块Disk Block对应的Buffer)
        return b + bi;
      }
    }
    brelse(bp);
  }
  panic("balloc: out of blocks");
}

// Zero a block.
static void
bzero(int dev, int bno)
{
  struct buf *bp;

  bp = bread(dev, bno);
  memset(bp->data, 0, BSIZE);
  log_write(bp);
  brelse(bp);
}
```

`bfree`就没什么好说的，先通过`bread`把块号为`b`的*Block*对应的*BitMap Block*读取进来，然后清零掉对应的Bit。

> Bfree (kernel/fs.c:90) finds the right bitmap block and clears the right bit. Again the exclusive use implied by bread and brelse avoids the need for explicit locking.

但是我有个问题，为什么这里不要把`bp`缓冲的内容写回去？

```c
// Free a disk block.
static void
bfree(int dev, uint b)
{
  struct buf *bp;
  int bi, m;

  bp = bread(dev, BBLOCK(b, sb));
  bi = b % BPB;
  m = 1 << (bi % 8);
  if((bp->data[bi/8] & m) == 0)
    panic("freeing free block");
  bp->data[bi/8] &= ~m;
  log_write(bp);
  brelse(bp);
}
```

**balloc和bfree都要在一个事务里调用**

#### 8.8 Inode layer

`INode`这个词本身可以有两种意思，一种是指“在硬盘上的INode”，另外一种指的是“在内存里的INode”，在内存里的iNode其实就是从硬盘读取的，相当于硬盘INode的缓存。

> the on-disk data structure containing a file’s size and list of data block numbers. 
>
> Or an in-memory inode, which contains a copy of the on-disk inode as well as extra information needed within the kernel.

INode都是集中在INode Block的，并且每个INode的大小都是一样的，所以给定一个数字N，很容易找到在硬盘上的第N个INode的位置。



硬盘上的INode使用一个`struct dinode`来定义：

- `type`：区分文件、目录、设备文件。type = 0表明INode为Free状态
- `nlink`： the number of directory entries that refer to this inode。文件系统中，指向这个Inode的链接数。

```c
/*fs.h*/
#define NDIRECT 12

// On-disk inode structure
struct dinode {
  short type;           // File type
  short major;          // Major device number (T_DEVICE only)
  short minor;          // Minor device number (T_DEVICE only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  uint addrs[NDIRECT+1];   // Data block addresses。数据块的地址的集合。地址是Data Block的块号
    //The addrs array records the block numbers of the disk blocks holding the file’s content.
};
```

在内存里活跃的iNode使用一个 `struct inode`来定义：

- 内核仅在有C指针指向一个iNode时，在内存中保留它。
- 当引用计数到达0时，内核会清理掉这个INode。

```c
// in-memory copy of an inode
struct inode {
  uint dev;           // Device number
  uint inum;          // Inode number
  int ref;            // Reference count
  struct sleeplock lock; // protects everything below here
  int valid;          // inode has been read from disk?

  short type;         // copy of disk inode.从这里开始就是和struct dinode里的一模一样的域了
  short major;
  short minor;
  short nlink;
  uint size;
  uint addrs[NDIRECT+1];
};
```

为了获取和释放一个指向内存中的`inode`指针，分别要调用`iget`和`iput`。因为直接类似 `struct inode * ptr = xxxx `并不会自动给`inode`的引用计数加一，所以需要调用函数，函数会这么做。

> Pointers to an inode can come from file descriptors, current working directories, and transient kernel code such as exec.

下面是`iget`：

- `iget`不从硬盘读取。所以如果是复用数组中一个旧的`inode`，会把其`valid`域置为0。
- `iget`**也不锁住inode**。
- `iget`不提供对`inode`的互斥访问，因为只有这样才能允许多个指针指向内存中的同一个iNode。

文件系统的许多地方都依赖于`iget`的这种非互斥的特点：

>  Many parts of the file-system code depend on this behavior of iget(), both to hold long-term references to inodes (as open files and current directories) and to prevent races while avoiding deadlock in code that manipulates multiple inodes (such as pathname lookup)

`icache`就是Inode的Cache，由一个自旋锁和一个`struct inode`数组构成。数组的每个元素就是一个缓存的`dinode`。

`icache`里的数组的每一个`struct inode`应该都是在声明`icache`的时候就已经自动分配好了，后面都是在复用最开始分配好的`struct Inode`?

```c
/*param.h*/
#define NINODE       50  // maximum number of active i-nodes

/*fs.c*/
struct {
  struct spinlock lock;
  struct inode inode[NINODE];
} icache;


// Find the inode with number inum on device dev
// and return the in-memory copy. Does not lock
// the inode and does not read it from disk.
static struct inode*
iget(uint dev, uint inum)
{
  struct inode *ip, *empty;

  acquire(&icache.lock);

  // Is the inode already cached?
  empty = 0;
  for(ip = &icache.inode[0]; ip < &icache.inode[NINODE]; ip++){
      /*
      循环里同时做了两件事:
      找到一个设备号和块号都能对得上的缓存Inode
      找到第一个为空的(引用计数 == 0)的空槽
      */
    if(ip->ref > 0 && ip->dev == dev && ip->inum == inum){ //引用计数大于0，设备号和块号都对得上,就说明找到了已存在的缓存inode
      ip->ref++; //引用计数 ++
      release(&icache.lock); //释放icache的锁然后返回
      return ip;
    }
    if(empty == 0 && ip->ref == 0)    // Remember empty slot.
      empty = ip;
  }

  // Recycle an inode cache entry.
  if(empty == 0) //Empty == 0说明遍历一整个数组都没找到一个Free的Inode缓存槽。
    panic("iget: no inodes");
//没找到空槽会直接Panic，下面的代码也不会运行。只有在不Panic的时候，下面的代码才会执行
  ip = empty;
  ip->dev = dev;
  ip->inum = inum;
  ip->ref = 1;
  ip->valid = 0; //因为iget并不会从硬盘去读取对应的那个dinode，所以要把valid置为0。
  release(&icache.lock);

  return ip;
}
```

然后是`iput`：

如果内存中的`inode`本身的引用计数到1，**并且对应的硬盘上的Inode的链接数到0(ip->nlink == 0)，**`iput`会Free掉硬盘上的Inode。

而其他情况下，`iput`就只是简单的让内存中的`struct inode`的引用计数`ref --`而已。并不会删掉`struct inode`，也不会让`struct inode`复用给另外一个inode。

```c
// Drop a reference to an in-memory inode.
// If that was the last reference, the inode cache entry can
// be recycled.
// If that was the last reference and the inode has no links
// to it, free the inode (and its content) on disk.
// All calls to iput() must be inside a transaction in
// case it has to free the inode.
void
iput(struct inode *ip)
{
  acquire(&icache.lock);

  if(ip->ref == 1 && ip->valid && ip->nlink == 0){ // 内存中还有一个引用，文件系统中已经没有引用了(nlink == 0)
    // inode has no links and no other references: truncate and free.

    // ip->ref == 1 means no other process can have ip locked,
    // so this acquiresleep() won't block (or deadlock).
    acquiresleep(&ip->lock);

    release(&icache.lock);

    itrunc(ip);
    ip->type = 0;
    iupdate(ip);
    ip->valid = 0;

    releasesleep(&ip->lock);

    acquire(&icache.lock);
  }

  ip->ref--; //引用计数减1。实际上如果if()分支不满足条件的话，iput就只做了这件事
  release(&icache.lock);
}
```

`iupdate`在后面看过了，但是后文并没有提到`itrunc`，所以这里补上：

- `itrunc`用于将某个iNode截断（也就是把文件截断，也就是直接抛弃文件的内容）。
- 必须要持有这个INode的锁，这个很好理解。
- 因为调用了`bfree`，所以也一并放上来。
- 不过似乎前面的章节有一些信息是没有提到的，比如并没有提到`addrs`数组里的前12个是直接指针，最后一个是间接指针。
- 所以还是放到8.9节吧

```c
// Truncate inode (discard contents).
// Caller must hold ip->lock.
void
itrunc(struct inode *ip)
{
    /* 
    Itrunc (kernel/fs.c:410) starts by 
    freeing the direct blocks(kernel/fs.c:416-421), 
    then the ones listed in the indirect block (kernel/fs.c:426-429), 
    and finally the indirect block itself
    */
  int i, j;
  struct buf *bp;
  uint *a;

  for(i = 0; i < NDIRECT; i++){ //遍历一遍addrs数组，调用bfree释放掉每一块由直接指针指向的Data Block
    if(ip->addrs[i]){
      bfree(ip->dev, ip->addrs[i]);
      ip->addrs[i] = 0; //把数组里的值变成0
    }
  }

  if(ip->addrs[NDIRECT]){ //因为ip->addrs的最后一个单元是一个二重指针，所以不能放在上面那个循环里来释放。
      //if()指的是如果使用了间接指针的话
    bp = bread(ip->dev, ip->addrs[NDIRECT]); //读取间接指针指向的那个Disk Block，那个Block里全都是直接指针(Data Block的块号)
    a = (uint*)bp->data; //把Buffer里缓存数据的data数组转换为一个uint数组,这样data的每个单元存储的就是一个块号
    for(j = 0; j < NINDIRECT; j++){ //遍历整个data数组
      if(a[j])
        bfree(ip->dev, a[j]);
    }
    brelse(bp);
      //下面释放掉间接指针本身
    bfree(ip->dev, ip->addrs[NDIRECT]);
    ip->addrs[NDIRECT] = 0; 
  }

  ip->size = 0;
  iupdate(ip);
}
```

在Xv6的Inode代码中总共有四种类似的锁机制：

- `icache.lock`控制对icache整体的并发访问，防止出现同一个`inode`被接连两个进程一前一后使用等情况。
- 每个在内存中的`struct inode`有一个睡眠锁，用来串行化多个进程对同一个`struct inode`的访问，保证对`inode`以及对应的文件的互斥访问。
- 一个`struct inode`的`ref > 0`的时候，xv6会保证这个`struct Inode`不会用于缓存另外一个`dinode`。
- 一个处于硬盘上的Inode，当其链接数大于零时，Xv6不会Free掉它。



因为调用`iget`返回的`struct inode`里面不一定是有内容的。所以在真正使用`iget`返回的`struct inode`之前，代码要先调用`ilock`。`ilock`会给`inode`加锁（这样其他进程就没办法同时使用这个INode），并且会从硬盘里读取`dinode`，如果之前还没读过（也即`valid == 0`）

多个进程可以通过`iget`持有对同一个`struct inode`的指针，但是一次只能有一个进程能拿到它的锁。

`ilock`的代码还是很容易读懂的：

- `IBLOCK`是一个宏，用于计算第`i`个iNode在硬盘上是第几块Block（块号是多少）。
- `ilock`对于已经读取过硬盘的`struct inode`来说，就只做了拿到睡眠锁这一件事。

`iunlock`和`ilock`配对，用来解锁的。

```c
/*fs.h*/

// Block containing inode i
#define IBLOCK(i, sb)     ((i) / IPB + sb.inodestart) //sb:Super Block，超级块
/*fs.c*/
// Lock the given inode.
// Reads the inode from disk if necessary.
void
ilock(struct inode *ip)
{
  struct buf *bp;
  struct dinode *dip;

  if(ip == 0 || ip->ref < 1)
    panic("ilock");

  acquiresleep(&ip->lock); //拿到ip的睡眠锁

  if(ip->valid == 0){ //如果之前没从硬盘读过，那就读一下
    bp = bread(ip->dev, IBLOCK(ip->inum, sb)); //把硬盘里的INode读取到一个Buffer里
    dip = (struct dinode*)bp->data + ip->inum%IPB;
    ip->type = dip->type;
    ip->major = dip->major;
    ip->minor = dip->minor;
    ip->nlink = dip->nlink;
    ip->size = dip->size;
    memmove(ip->addrs, dip->addrs, sizeof(ip->addrs));
    brelse(bp);
    ip->valid = 1;
    if(ip->type == 0)
      panic("ilock: no type");
  }
}
// Unlock the given inode.
void
iunlock(struct inode *ip)
{
  if(ip == 0 || !holdingsleep(&ip->lock) || ip->ref < 1)
    panic("iunlock");

  releasesleep(&ip->lock);
}
```



inode 缓存的主要工作其实是同步多进程对同一个Inode的访问，缓存只是其次要工作。

`struct inode`缓存是直写法的（Write Through），这意味着编辑过缓存的inode的代码，要立即调用`iupdate`，把它写回硬盘。

>  The inode cache is *write-through*, which means that code that modifies a cached inode must immediately write it to disk with iupdate.

```c
// Copy a modified in-memory inode to disk.
// Must be called after every change to an ip->xxx field
// that lives on disk, since i-node cache is write-through.
// Caller must hold ip->lock.
void
iupdate(struct inode *ip)
{
  struct buf *bp;
  struct dinode *dip;

  bp = bread(ip->dev, IBLOCK(ip->inum, sb)); //把与ip对应的Inode所在的Disk Block读取到一个Buffer bp。
  dip = (struct dinode*)bp->data + ip->inum%IPB; //从bp里找到那个Inode
    //依次更新dinode的type,major等域
  dip->type = ip->type;
  dip->major = ip->major;
  dip->minor = ip->minor;
  dip->nlink = ip->nlink;
  dip->size = ip->size;
    
    /*
    这里利用memmove是把iNode里那个存储数据块的数组更新了，相当于利用遍历数组实现拷贝。
    类似for(int i = 0; i < 数组长度; i ++) 
    	dip->addrs[i] =  ip->addrs[i]
    只不过是用memmove做了这个工作而已。
    */
  memmove(dip->addrs, ip->addrs, sizeof(ip->addrs)); // memmove(dest, src, Length in Byte)
  log_write(bp); //把更改写入Log就认为是写回硬盘了。应该是？
  brelse(bp);
}
```

#### 8.9 Code: Inodes

要创建一个新的Inode，调用`ialloc`。`ialloc`会遍历硬盘上的全部INode Block，找到其中一个空闲的（`dinode.type == 0`），将其标记为传入的类型`type`，然后返回一个内存缓存的`struct Inode`（调用`iget`来得到）。

- 关于为什么`inum`是从1开始的，GPT给的解释是在大多数文件系统里Inode都从1开始编号，0号Inode一般有特殊用途。
- 因为`bp`是自带睡眠锁的，所以可以保证当当前进程在读取bp中的`dinode`时，其他进程如果想读取这一块Disk Block会被阻塞，因此不会存在多个进程同时看到同一个`dinode`是Free状态的。

```c
// Allocate an inode on device dev.
// Mark it as allocated by  giving it type type.
// Returns an unlocked but allocated and referenced inode.
struct inode*
ialloc(uint dev, short type) //设备号, 文件类型
{
  int inum;
  struct buf *bp;
  struct dinode *dip;

  for(inum = 1; inum < sb.ninodes; inum++){ //遍历每一个Inode，从代码反推Inode应该是从1开始计数的?
    bp = bread(dev, IBLOCK(inum, sb)); //读取第inum个 dinode 所在的Disk Block，并缓存在一个Buffer bp里面
    dip = (struct dinode*)bp->data + inum%IPB; //找到第inum个 dinode,用dip指向它
    if(dip->type == 0){  // a free inode dip -> type == 0为空闲
      memset(dip, 0, sizeof(*dip)); //把dip这个dinode所在的内存全部清零
      dip->type = type; //设置其文件类型为给定的type
      log_write(bp);   // mark it allocated on the disk
      brelse(bp); //释放掉Buffer
      return iget(dev, inum); //调用iget得到dip在内存里的struct inode缓存，并返回给调用者
    }
    brelse(bp);
  }
  panic("ialloc: no inodes");
}
```

然后是`iget`，因为在上一节已经看过了，所以这里直接引用原文吧：

> Iget (kernel/fs.c:243) looks through the inode cache for an active entry (ip->ref > 0) with the desired device and inode number. If it finds one, it returns a new reference to that inode(kernel/fs.c:252-256). As iget scans, it records the position of the first empty slot (kernel/fs.c:257-258), which it uses if it needs to allocate a cache entry



然后是`ilock`和`iunlock`，也看过了直接贴原文

> Code must lock the inode using ilock before reading or writing its metadata or content. ilock (kernel/fs.c:289) uses a sleep-lock for this purpose. Once ilock has exclusive access to the inode, it reads the inode from disk (more likely, the buffer cache) if needed. The function iunlock (kernel/fs.c:317) releases the sleep-lock, which may cause any processes sleeping to be woken up.



`Iput`也看过了：

> Iput (kernel/fs.c:333) releases a C pointer to an inode by decrementing the reference count(kernel/fs.c:356). If this is the last reference, the inode’s slot in the inode cache is now free and can be re-used for a different inode.
>
> If iput sees that there are no C pointer references to an inode and that the inode has no links to it (occurs in no directory), then the inode and its data blocks must be freed. Iput calls itrunc to truncate the fifile to zero bytes, freeing the data blocks; sets the inode type to 0 (unallocated); and writes the inode to disk (kernel/fs.c:338).

为什么像`read`这样的系统调用也可能会成为对某个文件的最后一个引用？举个例子，`read`读的文件，被删掉了，`read`就是最后一个引用了。`read`结束这个文件应该就会被删除。

> iput() can write to the disk. This means that any system call that uses the file system may write the disk, because the system call may be the last one having a reference to the file. Even calls like read() that appear to be read-only, may end up calling iput(). This, in turn, means that even read-only system calls must be wrapped in transactions if they use the file system.



`Iput`与崩溃之间的挑战：

`Iput`在一个文件的链接数降到0的时候并不会立即就把这个文件截断，因为此时对这个文件的引用数可能大于1，有些进程可能还持有一个指向内存中的`inode`的引用。但是假如在最后一个进程关闭对这个文件的文件描述符之前，发生了崩溃，就会导致这个文件实际上在硬盘上仍然存在，但是却没有任何一个目录项指向它。

> iput() doesn’t truncate a file immediately when the link count for the file drops to zero, because some process might still hold a reference to the inode in memory: a process might still be reading and writing to the file, because it successfully opened it. But, if a crash happens before the last process closes the file descriptor



解决这个问题有两种办法：

- 第一种：在故障恢复时，重启之后，文件系统扫描整个文件系统，去寻找那些被标记为已经分配，但是却没有目录项指向它们的Disk Block。如果找到了，就把这些块释放掉。
- 第二种：文件系统在硬盘上用一个List记录下（比如记在超级块）那些引用计数不为0，但是链接数为0的文件（以记录INode 号的形式记下来）。正常情况下，当某个文件引用计数将至0，文件系统就把这个文件截断，同时把List里对应的项去掉。而在发生崩溃的情况下，恢复的时候只要把List上的每个文件都Free掉即可。

但是Xv6两个都没实现

#### 8.10 Code: Inode content

<img src="https://raw.githubusercontent.com/CorneliaStreet1/NewPicBed0/master/image-20230727081012941.png" alt="image-20230727081012941" style="zoom:80%;" />

12个直接指针，直接指向某个Data Block，一个一重间接指针。所以`addrs`的大小是`NDIRECT + 1`，NDIRECT == N-Direct，N个直接指针 + 1个间接指针。这种设计可以同时兼顾小文件和大文件。

```c
/*fs.h*/
#define NDIRECT 12

// On-disk inode structure
struct dinode {
  short type;           // File type
  short major;          // Major device number (T_DEVICE only)
  short minor;          // Minor device number (T_DEVICE only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  uint addrs[NDIRECT+1];   // Data block addresses
};
```

`bmap`负责管理这些直接指针间接指针之类的，给其他更高一级的函数提供接口。

`bmap`返回在`iNode ip`中第`bn `个Data Block的硬盘块号（Disk Block Number）

```c
// Inode content
//
// The content (data) associated with each inode is stored
// in blocks on the disk. The first NDIRECT block numbers
// are listed in ip->addrs[].  The next NINDIRECT blocks are
// listed in block ip->addrs[NDIRECT].

// Return the disk block address of the nth block in inode ip.
// If there is no such block, bmap allocates one.
static uint
bmap(struct inode *ip, uint bn) //返回第bn个数据块的硬盘块号
{
  uint addr, *a;
  struct buf *bp;

  if(bn < NDIRECT){ //如果bn < NDIRECT,就在直接指针的范围内.直接去索引addr[bn],然后按需分配数据数据块
    if((addr = ip->addrs[bn]) == 0) //未分配过，分配一下
      ip->addrs[bn] = addr = balloc(ip->dev); //把balloc返回的块号同时赋值给addr和addrs[bn]
    return addr; //返回
  }
  bn -= NDIRECT; //假如是间接块的话，第bn个数据块，就是间接块里的第 bn - NDIRECT 个数据块,所以这里bn要减去NDIRECT

  if(bn < NINDIRECT){
    // Load indirect block, allocating if necessary.
    if((addr = ip->addrs[NDIRECT]) == 0) //如果间接指针未分配
      ip->addrs[NDIRECT] = addr = balloc(ip->dev); //那就分配一个指针块，让间接指针 ip->addrs[NDIRECT] 指向分配的这一块
    bp = bread(ip->dev, addr); //把刚才分配的指针读取到Buffer
    a = (uint*)bp->data; //addr指向的那一块实际上是存储了256个直接指针的指针块,所以data实际上存储了(未分配的)256个块号
    if((addr = a[bn]) == 0){ //分配
      a[bn] = addr = balloc(ip->dev);
      log_write(bp);
    }
    brelse(bp); //释放掉Buffer,返回addr
    return addr;
  }
    //bn超范围了
  panic("bmap: out of range");
}
```

`bmap`是被`readi`和`writei`调用的：

- 二者都会做边界检查。当起点越过文件结尾时，`readi`会直接返回，而`Writei`会返回-1
- `write`在写入的时候会按需扩展文件
- 下面这个哪有检查？？？

> Both readi and writei begin by checking for *ip->type == T_DEV*. This case handles special devices whose data does not live in the file system; we will return to this case in the file descriptor layer.

```c
// Read data from inode.
// Caller must hold ip->lock.
// If user_dst==1, then dst is a user virtual address;
// otherwise, dst is a kernel address.
int
readi(struct inode *ip, int user_dst, uint64 dst, uint off, uint n) //user_dst是一个布尔值，表示dst是否是一个用户空间虚拟地址
{
  uint tot, m;
  struct buf *bp;

  if(off > ip->size || off + n < off) // 偏移量 > 文件的结尾 或者 偏移量 + 读取的字节数 < 偏移量(n < 0),直接返回0
    return 0;
  if(off + n > ip->size) //要读取的字节数n，大于，以off起点到文件结尾为终点的字节数，那么实际读取的字节数就是起点到结尾的差
    n = ip->size - off;

    //读,tot是total,已读取的总字节数
  for(tot=0; tot<n; tot+=m, off+=m, dst+=m){ 
    bp = bread(ip->dev, bmap(ip, off/BSIZE)); 
    m = min(n - tot, BSIZE - off%BSIZE); //m是单次循环读取的字节数
    if(either_copyout(user_dst, dst, bp->data + (off % BSIZE), m) == -1) { //把读取到的字节复制到dst指向的位置
      brelse(bp);
      break;
    }
    brelse(bp);
  }
  return tot;
}
```

`writei`：

```c
// Write data to inode.
// Caller must hold ip->lock.
// If user_src==1, then src is a user virtual address;
// otherwise, src is a kernel address.
int
writei(struct inode *ip, int user_src, uint64 src, uint off, uint n)
{
  uint tot, m;
  struct buf *bp;

  if(off > ip->size || off + n < off) //边界检查，起点是否超过了文件结尾 || n < 0
    return -1;
  if(off + n > MAXFILE*BSIZE) //偏移量(起点) + 要写入的字节数 > 文件结尾的最大值。换句话说就是从off开始写n个字节会超出文件的最大大小
    return -1;

  for(tot=0; tot<n; tot+=m, off+=m, src+=m){
    bp = bread(ip->dev, bmap(ip, off/BSIZE));
    m = min(n - tot, BSIZE - off%BSIZE);
    if(either_copyin(bp->data + (off % BSIZE), user_src, src, m) == -1) {
      brelse(bp);
      break;
    }
    log_write(bp);
    brelse(bp);
  }

  if(n > 0){
    if(off > ip->size)
      ip->size = off;
    // write the i-node back to disk even if the size didn't change
    // because the loop above might have called bmap() and added a new
    // block to ip->addrs[].
    iupdate(ip);
  }

  return n;
}
```

`stati`：把`struct inode`的信息复制到一个`struct stat`，其实就是暴露给`stat`系统调用的接口

```c
// Copy stat information from inode.
// Caller must hold ip->lock.
void
stati(struct inode *ip, struct stat *st)
{
  st->dev = ip->dev;
  st->ino = ip->inum;
  st->type = ip->type;
  st->nlink = ip->nlink;
  st->size = ip->size;
}
```

#### 8.11 Code: directory layer

目录的Inode类型是`T_DIR`，目录是一个特殊的文件，目录里的数据是一系列的目录项（a sequence of directory entries）。每个目录项是一个 `struct dirent`，包含一个名字和iNode号。每个名字最长不超过14字节。iNode号是0的目录项是Free的目录项

> Directory entries with inode number zero are free.

```c
/*fs.h*/

// Directory is a file containing a sequence of dirent structures.
#define DIRSIZ 14

struct dirent {
  ushort inum;
  char name[DIRSIZ];
};
```

`dirlookup`：从给定的目录寻找具有给定名字的目录项。如果找到了，调用`iget`返回一个**无锁的**指向对应文件的INode的指针，同时将`poff`的偏移量设置在目录中，这个目录项开始的地方。

```c
// Look for a directory entry in a directory.
// If found, set *poff to byte offset of entry.
struct inode*
dirlookup(struct inode *dp, char *name, uint *poff)
{
  uint off, inum;
  struct dirent de;

  if(dp->type != T_DIR)
    panic("dirlookup not DIR");

  for(off = 0; off < dp->size; off += sizeof(de)){  //以sizeof(de)为步长，这样恰好off每次指向一个目录项的起始字节
      //从偏移量为off的地方开始，读取sizeof(de)个字节，放到de里去。也就刚好读到一个完整的目录项
    if(readi(dp, 0, (uint64)&de, off, sizeof(de)) != sizeof(de))
      panic("dirlookup read");
    if(de.inum == 0) //因为使用iNode number 0来表示空闲
      continue;
    if(namecmp(name, de.name) == 0){ 
      // entry matches path element
      if(poff)
        *poff = off; //把被poff指向的那个变量设置到指向目录项的第一个字节
      inum = de.inum;
      return iget(dp->dev, inum); //获取与那个目录项对应的文件的INode，并返回给调用者
    }
  }

  return 0;
}
```

`dirlink`往给定目录里添加一个目录项。如果目录项的名字已经存在，返回一个Error。

代码还是很简单的。

```c
// Write a new directory entry (name, inum) into the directory dp.
int
dirlink(struct inode *dp, char *name, uint inum)
{
  int off;
  struct dirent de;
  struct inode *ip;

  // Check that name is not present.
  if((ip = dirlookup(dp, name, 0)) != 0){
    iput(ip); //因为if里调用了dirlookup,dirlookup调用了iget,所以这里要调用iput
    return -1;
  }

  // Look for an empty dirent.
  for(off = 0; off < dp->size; off += sizeof(de)){
    if(readi(dp, 0, (uint64)&de, off, sizeof(de)) != sizeof(de))
      panic("dirlink read");
    if(de.inum == 0)
      break;
  }    //跳出循环的时候，off要么指向一个空项的第一个字节，要么指向目录的末尾。不管哪种情况都会继续写入新的Entry

  strncpy(de.name, name, DIRSIZ); 
  de.inum = inum;
  if(writei(dp, 0, (uint64)&de, off, sizeof(de)) != sizeof(de))
    panic("dirlink");

  return 0;
}
```

#### 8.12 Code: Path names

`namei`用于解析给定的路径`path`，返回对应的Inode。`nameiparent`是`namei`的一个变体，不同的是它返回的是父目录的Inode，把最后一个元素复制到`name`里。两个函数实际上都调用的是`namex`来完成真正的工作，调用传参仅有一个标志位参数不一样。标志位为1时表示返回父目录的`inode`节点，为0时表示返回最终的路径元素的`inode`节点

```c
struct inode*
namei(char *path)
{
  char name[DIRSIZ];
  return namex(path, 0, name);
}
struct inode*
nameiparent(char *path, char *name)
{
  return namex(path, 1, name);
}
```

`namex`：

`Namex`先觉得从哪里开始做路径解析，如果路径以`/`开始，那就从根目录开始解析，否则从当前目录开始解析。`Namex`在后一种情况下会调用`idup`，所以先从`idup`开始看：`idup`就是很简单地将`Inode`的引用计数加一。

问题：为什么不需要先获取 ip的睡眠锁？难道是因为要求调用`idup`之前需要持有那个锁？

```c
// Increment reference count for ip.
// Returns ip to enable ip = idup(ip1) idiom.
struct inode*
idup(struct inode *ip)
{
  acquire(&icache.lock);
  ip->ref++;
  release(&icache.lock);
  return ip;
}
```

然后是`skipelem`，`namex`在循环条件里调用它：调用`skip_element`的作用就是跳过一个路径元素（所谓的一个元素指的应该是夹在斜杠之间的字符串），看注释举的例子就很明白了。

```c
// Copy the next path element from path into name.
// Return a pointer to the element following the copied one.
// The returned path has no leading slashes,
// so the caller can check *path=='\0' to see if the name is the last one.
// If no name to remove, return 0.
//
// Examples:
//   skipelem("a/bb/c", name) = "bb/c", setting name = "a"
//   skipelem("///a//bb", name) = "bb", setting name = "a"
//   skipelem("a", name) = "", setting name = "a"
//   skipelem("", name) = skipelem("////", name) = 0
//
static char*
skipelem(char *path, char *name)
{
  char *s;
  int len;
    //如果我们以 path = ///a//b 为例
  while(*path == '/') //处理斜杠连续出现至少两次的情况
    path++; //循环三次,path依次变为//a//b ==> /a//b ==> a//b。
    // 【果path指针指向斜杠字符，就一直向后移动，直到指针指向非斜杠字符或者字符串结束】
  if(*path == 0) //处理skipelem("", name) 的情况
    return 0;
  s = path; //到这里path = a//b。【s将指向路径中的第一个非斜杠字符。】
    
  while(*path != '/' && *path != 0)//【这里用于寻找下一组斜杠开始的位置】
    path++; //只循环一次,path 由 a//b 变为 //b。
    
  len = path - s; /*
  s和path的长度之差.实际上就是一个路径元素的长度。 path指针的地址在s指针的后面。
  因为在s出现之后，path在上一行的循环 path++ 了几次，所以path的值是大于s的。
  所以是用path - s去算长度的差，尽管path指向的字符串是更短的那个，但是path的地址更大。
  */
  if(len >= DIRSIZ) //路径元素的长度大于最大限制，只取其前DIRSIZ个字节
    memmove(name, s, DIRSIZ);
  else { //否则不超过限制,直接把名字复制到name里去
    memmove(name, s, len);
    name[len] = 0; // 给字符串添加一个 '\0' 的结尾
  }
  while(*path == '/') //找到下一个非斜杠的起始位置。path由 //b 变为 b
    path++;
  return path; //返回 path = b，并且此时name[] = "a\0"
}
```

对于`namex`我有个问题，29行的if，假如满足条件的话，`*path == '\0`就成立，但是这样不是应该不满足循环条件吗？

```c
/*param.h*/
#define ROOTDEV       1  // device number of file system root disk

/*fs.h*/
#define ROOTINO  1   // root i-number

/*fs.c*/

// Look up and return the inode for a path name.
// If parent != 0, return the inode for the parent and copy the final
// path element into name, which must have room for DIRSIZ bytes.
// Must be called inside a transaction since it calls iput().
static struct inode*
namex(char *path, int nameiparent, char *name) //nameiparent其实一个布尔值。为1的时候把父目录的Inode返回，把最后一个元素复制到name
{
  struct inode *ip, *next;

  if(*path == '/')
    ip = iget(ROOTDEV, ROOTINO);
  else
    ip = idup(myproc()->cwd);

  while((path = skipelem(path, name)) != 0){
    ilock(ip); //这里锁住ip不是为了并发安全，而是为了保证ip里的内容一定是从硬盘里读取的
    if(ip->type != T_DIR){
      iunlockput(ip);
      return 0;
    }
    if(nameiparent && *path == '\0'){ 
      // Stop one level early.
      iunlock(ip);
      return ip;
    }
    if((next = dirlookup(ip, name, 0)) == 0){ //从与ip对应的目录里寻找到名字为name数组的那个inode(其实就是下一级目录的inode)
      iunlockput(ip);
      return 0;
    }
    iunlockput(ip);
    ip = next; //ip变成下一级目录的inode了，然后进入下一个循环
  }
  if(nameiparent){ //这里代表没有一次都进入while循环(比如path = ""),但是nameiparent != 0的情况
    iput(ip);
    return 0;
  }
  return ip;
}
```

`namex`是允许多个内核进程并发调用它，因为它是对路径中出现的每个目录独立上锁的，所以对不同的目录的搜寻是可以并发进行的。

> Namex locks each directory in the path separately so that lookups in different directories can proceed in parallel

但是这样的并发也带来了一些问题：比如一个内核线程正在搜索一个路径名的时候，另外一个线程改变了目录树的结构。再比如，一个线程在搜寻一个路径的时候，这个路径上的某个目录可能被另外一个线程删除了。

> For example, while one kernel thread is looking up a pathname another kernel thread may be changing the directory tree by unlinking a directory. A potential risk is that a lookup may be searching a directory that has been deleted by another kernel thread and its blocks have been re-used for another directory or file.

但是xv6避免了这种静态条件，一是对每个目录加锁可以串行化对同一个目录的并发访问。二是因为存在inode的引用计数这种东西，使得即使一个目录已经被删除了，但是因为另外一个进程还在引用对应的iNode，引用计数不为零。xv6也暂时不会删除它。



另外一个风险是死锁，比如当搜寻`"."`时，`next`与`ip`指向同一个inode。解决这个问题的做法是先释放当前锁，再让ip = next。

不过为什么？我感觉代码里好像没体现？

#### 8.13 File descriptor layer

Unix系的一个很酷的方面就是，Unix下的大部分的资源都以文件的形式存在，包含设备、控制台、管道、真正的文件。文件描述符层就是实现这个统一的接口的层。



Xv6给每个进程它自己的打开文件表，或者说文件描述符表。每个打开的文件用一个`struct file`来代表，`struct file`实际上是一个管道或者inode，外加偏移量的包装。

`proc.h`中的代码有体现每个进程有自己的打开文件表这一点，这里只截取了有关的代码。

然后再补充一点关于`struct inode`以及`struct dinode`里的`major`和`minor`域是什么。

- `major`域，应该是叫做主要设备号吧？用于给设备分类，可以说是类编号。回答这个设备是哪一类设备，比如是硬盘还是打印机
- `minor`域，应该是叫做次要设备号吧？用于区分具体某一类下的哪一个设备。回答这个设备是某某类下的哪一个设备。

```c
/*param.h*/
#define NOFILE       16  // open files per process

/*proc.h*/
struct proc {
    //文件描述符实际上就是ofile数组的索引
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
};

/*fs.h*/

// On-disk inode structure
struct dinode {
  short type;           // File type
  short major;          // Major device number (T_DEVICE only)
  short minor;          // Minor device number (T_DEVICE only)
  short nlink;          // Number of links to inode in file system
  uint size;            // Size of file (bytes)
  uint addrs[NDIRECT+1];   // Data block addresses
};

/*file.h*/
struct file {
  enum { FD_NONE, FD_PIPE, FD_INODE, FD_DEVICE } type;
  int ref; // reference count
  char readable; //打开的目的是用于读吗?
  char writable; //打开的目的是用于写吗?
  struct pipe *pipe; // FD_PIPE
  struct inode *ip;  // FD_INODE and FD_DEVICE
  uint off;          // FD_INODE
  short major;       // FD_DEVICE
};

// in-memory copy of an inode
struct inode {
  uint dev;           // Device number
  uint inum;          // Inode number
  int ref;            // Reference count
  struct sleeplock lock; // protects everything below here
  int valid;          // inode has been read from disk?

  short type;         // copy of disk inode
  short major;
  short minor;
  short nlink;
  uint size;
  uint addrs[NDIRECT+1];
};
```

每次对`open`的调用都会创建一个新的`struct file`。如果多个进程独立地打开同一个文件，那么不同的实例（`struct file`）会有不同的读写偏移量。

另一方面，同一个`struct file`能够在一个进程的打开文件表里出现多次，也可以在多个进程的文件表里出现。举个例子，比如一个进程利用`open`打开一个文件，然后使用`dup`创建别名，或者Fork出一个子进程与子进程共享。一个文件（`struct file`）能够打开用于读或者写，`readable`和`writeable`两个域就是来追踪这一点的。



整个系统打开的文件都被保存在一个全局的文件表里面，这个表是`struct ftable`。与之对应的函数有：

- 分配文件：`filealloc`
- 创建一个副本引用：`filedup`
- 释放一个引用：`fileclose`
- 读写文件：`fileread`，`filewrite`

```c
/*param.h*/
#define NOFILE       16  // open files per process
#define NFILE       100  // open files per system

/*file.c*/
struct {
  struct spinlock lock;
  struct file file[NFILE];
} ftable;
```

那么接下来就来看看这几个函数：

分配的很简单， 遍历一遍`ftable`，找到第一个未被引用（引用计数为0）的单元，把其引用计数置为1然后返回其地址即可。

```c
// Allocate a file structure.
struct file*
filealloc(void)
{
  struct file *f;

  acquire(&ftable.lock);
  for(f = ftable.file; f < ftable.file + NFILE; f++){
    if(f->ref == 0){
      f->ref = 1;
      release(&ftable.lock);
      return f;
    }
  }
  release(&ftable.lock);
  return 0;
}
```

`filedup`也很简单，引用计数提高1，直接返回传入的参数。

```c
// Increment ref count for file f.
struct file*
filedup(struct file *f)
{
  acquire(&ftable.lock);
  if(f->ref < 1)
    panic("filedup");
  f->ref++;
  release(&ftable.lock);
  return f;
}

```

`fileclose`则复杂一点，引用计数减少1，如果引用计数变为0了，就关掉对应的文件或者管道。

```c
// Close file f.  (Decrement ref count, close when reaches 0.)
void
fileclose(struct file *f)
{
  struct file ff;

  acquire(&ftable.lock);
  if(f->ref < 1)
    panic("fileclose");
  if(--f->ref > 0){ //如果f的引用数减1之后还大于0，就不需要释放资源。--f表达式的值是自减之后的值
    release(&ftable.lock);
    return;
  }
    //以下都是f->ref自减1之后不大于0的时候才会执行的
  ff = *f;
  f->ref = 0;
  f->type = FD_NONE;
  release(&ftable.lock);
 //根据类型释放掉对应的资源
  if(ff.type == FD_PIPE){
    pipeclose(ff.pipe, ff.writable);
  } else if(ff.type == FD_INODE || ff.type == FD_DEVICE){
    begin_op();
    iput(ff.ip);
    end_op();
  }
}
```

函数`filestat`，`fileread`，`filewrite`分别实现了文件的stat, read,write操作。

`filestate`只允许对`inode`进行调用，并且调用了`stati`。`fileread`和`filewrite`则检查操作是否被打开时的模式允许（比如以读取模式打开的是不允许`filewrite`），然后把调用传递给管道或者`inode`的相应读写的实现。如果读写的文件是一个`inode`，`fileread`和`filewrite`使用I/O偏移量作为操作的偏移量，并且前移它。

管道没有偏移量的概念。

inode的函数都要求调用者来处理加锁的事。加锁的好处是读和写的偏移量的更新是原子的，这样对同一个文件的同时的多次写入不会彼此之间覆盖（虽然他们的写入可能是相互穿插的，但是不会彼此覆盖或者说重叠）

`filestate`：

```c
// Get metadata about file f.
// addr is a user virtual address, pointing to a struct stat.
int
filestat(struct file *f, uint64 addr)
{
  struct proc *p = myproc();
  struct stat st; //用于临时存储从f的inode读取的元数据
  
  if(f->type == FD_INODE || f->type == FD_DEVICE){
    ilock(f->ip);
    stati(f->ip, &st); //读取元数据，放在st
    iunlock(f->ip);
    if(copyout(p->pagetable, addr, (char *)&st, sizeof(st)) < 0) //从st跨地址空间从内核复制到addr指向的用户空间
      return -1;
    return 0;
  }
  return -1;
}
```

`fwrite`：

```c
// Read from file f.
// addr is a user virtual address.
int
fileread(struct file *f, uint64 addr, int n) //n应该是要读取的字节数
{
  int r = 0;

  if(f->readable == 0) //检查open()的时候是不是以可以读取的模式打开的
    return -1;

  if(f->type == FD_PIPE){ //管道读
    r = piperead(f->pipe, addr, n); //返回值是从管道读取的字节数
  } else if(f->type == FD_DEVICE){
    if(f->major < 0 || f->major >= NDEV || !devsw[f->major].read) //!devsw[f->major].read的意思是没有可以用于读取的函数
      return -1;
    r = devsw[f->major].read(1, addr, n);//这里devsw[f->major].read就是在调用设备的读取函数
  } else if(f->type == FD_INODE){ //inode的话就调用inode的readi了
    ilock(f->ip);
    if((r = readi(f->ip, 1, addr, f->off, n)) > 0)
      f->off += r; //增加偏移量
    iunlock(f->ip);
  } else {
    panic("fileread");
  }

  return r; //返回读取的字节数
}
```

`filewrite`：

```c
// Write to file f.
// addr is a user virtual address.
int
filewrite(struct file *f, uint64 addr, int n) //n是要写入的字节数
{
  int r, ret = 0; //

  if(f->writable == 0)
    return -1;

  if(f->type == FD_PIPE){ //管道调用pipewrite
    ret = pipewrite(f->pipe, addr, n);
  } else if(f->type == FD_DEVICE){
    if(f->major < 0 || f->major >= NDEV || !devsw[f->major].write) //!devsw[f->major].write表示没有可以用于write的函数
      return -1;
    ret = devsw[f->major].write(1, addr, n); //调用设备的write
  } else if(f->type == FD_INODE){ //inode就写入inode了
    // write a few blocks at a time to avoid exceeding
    // the maximum log transaction size, including
    // i-node, indirect block, allocation blocks,
    // and 2 blocks of slop for non-aligned writes.
    // this really belongs lower down, since writei()
    // might be writing a device like the console.
    int max = ((MAXOPBLOCKS-1-1-2) / 2) * BSIZE;
    int i = 0;
    while(i < n){
      int n1 = n - i;
      if(n1 > max)
        n1 = max;

      begin_op();
      ilock(f->ip);
      if ((r = writei(f->ip, 1, addr + i, f->off, n1)) > 0)
        f->off += r;
      iunlock(f->ip);
      end_op();

      if(r < 0)
        break;
      if(r != n1)
        panic("short filewrite");
      i += r;
    }
    ret = (i == n ? n : -1);
  } else {
    panic("filewrite");
  }

  return ret;
}
```

#### 8.14 Code: System calls

有了更低的几层所提供的函数之后，大部分系统调用的实现就显得很容易。

都在`kernel/sysfile.c`。这一小节挑了几个

> There are a few calls that deserve a closer look.

`sys_link`和`sys_unlink`用于给某个inode添加和移除引用。

`syslink`的第一步是调用`argstr`来获取系统调用的两个参数，一个`old`和一个`new`。并且假定`old`存在且不是一个目录。`new`则是`old`的另外一个路径。

所以先把`argstr`以及它涉及的调用链中的函数看一下：

- `argstr`将长度为一个字的第n个参数作为字符串来获取（其实就是说把第n个参数寄存器的值作为字符串的指针来解读），然后把字符串复制到`buf`，复制的最大长度是`max`字节。
- `argstr`则先调用`argaddr`来拿到字符串的地址（也就是寄存器里存的地址）。而`argaddr`又调用`argraw`来获取地址，`argraw`则是直接从陷阱帧里面读取。不是真的从寄存器里面读取，是因为处理中断的时候用户线程的参数已经被丢到`Trapframe`里去了。
- 获取到地址之后，调用`fetchstr`把字符串复制到`buf`所指向的**内核空间**（`fetchstr`会调用`copyinstr`的）。为什么要复制，而不是直接返回指针？我个人的解释是因为`addr`指向的字符串实际上是处于用户的地址空间的，所以`addr`指针是没办法直接被内核使用的。

```c
// Fetch the nth word-sized system call argument as a null-terminated string.
// Copies into buf, at most max.
// Returns string length if OK (including nul), -1 if error.
int
argstr(int n, char *buf, int max)
{
  uint64 addr;
  if(argaddr(n, &addr) < 0) //拿到字符串的地址,存储在局部变量addr
    return -1;
  return fetchstr(addr, buf, max); //拿到字符串本体,复制到buf里去。
}

// Fetch the nul-terminated string at addr from the current process.
// Returns length of string, not including nul, or -1 for error.
int
fetchstr(uint64 addr, char *buf, int max)
{
  struct proc *p = myproc();
  int err = copyinstr(p->pagetable, buf, addr, max);
  if(err < 0)
    return err;
  return strlen(buf);
}

// Retrieve an argument as a pointer.
// Doesn't check for legality, since
// copyin/copyout will do that.
int
argaddr(int n, uint64 *ip)
{
  *ip = argraw(n); //从陷阱帧里读取
  return 0;
}

static uint64
argraw(int n)
{
  struct proc *p = myproc();
  switch (n) {
  case 0:
    return p->trapframe->a0;
  case 1:
    return p->trapframe->a1;
  case 2:
    return p->trapframe->a2;
  case 3:
    return p->trapframe->a3;
  case 4:
    return p->trapframe->a4;
  case 5:
    return p->trapframe->a5;
  }
  panic("argraw");
  return -1;
}
```

在看完`sys_link`是如何获取到新的路径`new`和旧的路径`old`之后，我们可以继续往下看：

创建链接**要求新的路径和旧的路径处于同一个设备上**，因为inode号只有在同一个设备上才有独一无二的意义。

> The new parent directory must exist and be on the same device as the existing inode: inode numbers only have a unique meaning on a single disk. If an error like this occurs, sys_link must go back and decrement ip->nlink.

```c
// Create the path new as a link to the same inode as old.
uint64
sys_link(void)
{
  char name[DIRSIZ], new[MAXPATH], old[MAXPATH];
  struct inode *dp, *ip;

  if(argstr(0, old, MAXPATH) < 0 || argstr(1, new, MAXPATH) < 0) //把新路径和旧路径给取过来
    return -1;

  begin_op(); //事务的开始
  if((ip = namei(old)) == 0){ //找到旧路径的文件本体的inode
    end_op();
    return -1;
  }

  ilock(ip);
  if(ip->type == T_DIR){ //检查old是否是一个文件，如果是一个路径的话直接结束事务并且return
    iunlockput(ip);
    end_op();
    return -1;
  }

  ip->nlink++; //增加old的inode的链接数
  iupdate(ip); //更新inode到硬盘上
  iunlock(ip);

  if((dp = nameiparent(new, name)) == 0) //找到new的父目录的inode。把新的文件名(也就是new的最后一个路径元素)存入name
    goto bad;
  ilock(dp);
  if(dp->dev != ip->dev || dirlink(dp, name, ip->inum) < 0){ 
      //dp->dev != ip->dev检查new和old是否处于同一个设备上
      //dirlink(dp, name, ip->inum)则是往new的父目录里写入一个新的Directory Entry
    iunlockput(dp);
    goto bad;
  }
  iunlockput(dp); //放回ip和dp
  iput(ip);

  end_op(); //事务结束

  return 0;
//如果哪一步出错了，回滚
bad:
  ilock(ip);
  ip->nlink--; //回滚链接数的值
  iupdate(ip); //更新回硬盘
  iunlockput(ip); 
  end_op();
  return -1;
}
```

`Sys_Link`给现存的`inode`取一个新名字，但是`create`函数则是给一个新的`inode`取一个新名字。`create`是三个文件创建系统调用的泛化：带有`O_CREATE`Flag的Open，和创建新目录的`mkdir`，和创建新设备文件的`mkdev`。

`create`在`name`已经存在的时候，其表现取决于它被用于哪个系统调用：

- 如果是被用于Open，并且name本身是一个常规的文件，那么认为创建成功。否则认为失败
- 

```c
static struct inode*
create(char *path, short type, short major, short minor) //路径，类型，主设备号，次设备号
{
  struct inode *ip, *dp;
  char name[DIRSIZ];

  if((dp = nameiparent(path, name)) == 0) //得到父目录的inode，存入dp
    return 0;

  ilock(dp);

  if((ip = dirlookup(dp, name, 0)) != 0){ //如果name已经在目录dp中存在了
    iunlockput(dp);
    ilock(ip);
    if(type == T_FILE && (ip->type == T_FILE || ip->type == T_DEVICE))  //要创建的是一个文件，并且name是文件。就认为成功，返回ip
      return ip;
    iunlockput(ip); 
    return 0; //否则返回0代表失败
  }

    // 接下来的代码只有在name不存在的时候会执行
  if((ip = ialloc(dp->dev, type)) == 0) //先分配一个inode
    panic("create: ialloc");

  ilock(ip); //更新一下inode的数据,然后调用iupdate写回硬盘什么的
  ip->major = major;
  ip->minor = minor;
  ip->nlink = 1;
  iupdate(ip);

    /*
    如果要创建的是一个目录的话，那么需要添加的目录项有:
    往父目录dp里添加一项指向name的项
    往name目录ip里添加一项指向自身的"." 和一项指向父目录的 ".."
    */
  if(type == T_DIR){  // Create . and .. entries.
    dp->nlink++;  // for ".." 这里是把父目录的iNode链接数 ++了,不是name的inode
    iupdate(dp); //更新父目录的inode到硬盘上
    // No ip->nlink++ for ".": avoid cyclic ref count.("."的引用数不需要++,避免循环引用)
      
    if(dirlink(ip, ".", ip->inum) < 0 || dirlink(ip, "..", dp->inum) < 0) // 往name的inode里写入"."和".."的对应目录项
        /*
        "."对应的是 <".", name的inode number> 的映射目录项
        ".." 对应的是 <"..", 父目录的inode number> 的映射目录项
        */
      panic("create dots");
  }

    //不管name是一个目录还是一个普通文件，都需要往其上一级的目录添加一个目录项
  if(dirlink(dp, name, ip->inum) < 0) //往父目录里写入<name, ip->num>的目录项
    panic("create: dirlink");

  iunlockput(dp);

  return ip;
}
```

接下来看一下`sys_open`的实现：

```c
// Allocate a file descriptor for the given file.
// Takes over file reference from caller on success.
static int
fdalloc(struct file *f)
{
  int fd;
  struct proc *p = myproc();

  for(fd = 0; fd < NOFILE; fd++){ //基本思路就是遍历当前进程的proc结构体里的ofile数组，找到一个值为空的，也就是没使用的
    if(p->ofile[fd] == 0){
      p->ofile[fd] = f;
      return fd;
    }
  }
  return -1;
}

uint64
sys_open(void)
{
  char path[MAXPATH];
  int fd, omode;
  struct file *f;
  struct inode *ip;
  int n;

  if((n = argstr(0, path, MAXPATH)) < 0 || argint(1, &omode) < 0) //同样的,拿到两个用户传入的参数
    return -1;

  begin_op();

  if(omode & O_CREATE){ //如果Flag是O_CREATE,就调用create
    ip = create(path, T_FILE, 0, 0);
    if(ip == 0){
      end_op();
      return -1;
    }
  } else {
    if((ip = namei(path)) == 0){ //否则就是打开既有的文件，调用namei找到其inode
      end_op();
      return -1;
    }
    ilock(ip); //锁上拿到的inode
    if(ip->type == T_DIR && omode != O_RDONLY){ 
        // 假如类型是目录，Flag不是只读的话,那就说明有问题(因为针对目录的修改是有专门的系统调用的,比如mkdir).
      iunlockput(ip);
      end_op();
      return -1;
    }
  }

  if(ip->type == T_DEVICE && (ip->major < 0 || ip->major >= NDEV)){ //对设备类型的文件做检查
    iunlockput(ip);
    end_op();
    return -1;
  }

  if((f = filealloc()) == 0 || (fd = fdalloc(f)) < 0){ //分配一个file structure 给f,同时分配对应的文件描述符。
      //fdalloc的实现过于简单，不做太多注释了，直接贴在open的上面
    if(f) //如果分配文件成功，但是分配文件描述符失败的话，就需要关闭这个struct file f。
      fileclose(f);
    iunlockput(ip);
    end_op();
    return -1;
  }

  if(ip->type == T_DEVICE){
    f->type = FD_DEVICE;
    f->major = ip->major;
  } else {
    f->type = FD_INODE;
    f->off = 0; //初始偏移量为0
  }
    //设置Flag和对应的指针之类的
  f->ip = ip;
  f->readable = !(omode & O_WRONLY);
  f->writable = (omode & O_WRONLY) || (omode & O_RDWR);

  if((omode & O_TRUNC) && ip->type == T_FILE){
      //如果Flag里有截断的话，就调用itrunc去截断文件
    itrunc(ip);
  }

  iunlock(ip);
  end_op();

  return fd; //返回得到的文件描述符
}
```



最后再来看看和管道挂钩的系统调用吧：`sys_pipe`

```c
uint64
sys_pipe(void)
{
  uint64 fdarray; // user pointer to array of two integers
  struct file *rf, *wf;
  int fd0, fd1;
  struct proc *p = myproc();

  if(argaddr(0, &fdarray) < 0) //获取用户进程传递的参数，指向一个int[]的指针。存入fdarry
    return -1;
  if(pipealloc(&rf, &wf) < 0) //分配一个管道,pipealloc会调用filealloc，在ftable中添加与管道的读取端和写入端分别对应的struct file项
    return -1;
  fd0 = -1;
  if((fd0 = fdalloc(rf)) < 0 || (fd1 = fdalloc(wf)) < 0){ //给管道的写入端和读取端各自分配一个文件描述符
    if(fd0 >= 0) //如果fd0分配成功而fd1分配失败
      p->ofile[fd0] = 0; //就只关闭fd1
      //读取端和写入端对应的struct file都需要关闭
    fileclose(rf);
    fileclose(wf);
    return -1;
  }
  if(copyout(p->pagetable, fdarray, (char*)&fd0, sizeof(fd0)) < 0 ||
     copyout(p->pagetable, fdarray+sizeof(fd0), (char *)&fd1, sizeof(fd1)) < 0){
    p->ofile[fd0] = 0;
    p->ofile[fd1] = 0;
    fileclose(rf);
    fileclose(wf);
    return -1;
  }
  return 0;
}
```

### Lecture

一个文件（inode）只能在link count为0的时候被删除。实际的过程可能会更加复杂，实际中还有一个openfd count，也就是当前打开了文件的文件描述符计数。一个文件只能在**这两个计数器都为0的时候**才能被删除。



Sector和Block：

- sector通常是磁盘驱动可以读写的最小单元，它过去通常是512字节。
- block通常是操作系统或者文件系统视角的数据。它由文件系统定义，在XV6中它是1024字节。所以XV6中一个block对应两个sector。通常来说一个block对应了一个或者多个sector。

Directory Entry的结构：

每一个目录包含了directory entries，每一条entry都有固定的格式：

- 前2个字节包含了目录中文件或者子目录的inode编号，
- 接下来的14个字节包含了文件或者子目录名。

所以每个entry总共是16个字节。

在XV6中，使用inode中的type字段来标识inode是否空闲，这个字段同时也会用来表示inode是一个文件还是一个目录。

创建并写入一个新文件会发生的事：

- 创建文件：
  1. 分配一个Inode，将其标记为占用（通过类型字段）
  2. 填充Inode里的元数据
  3. 往这个文件所在的父目录的目录项里添加与这个文件对应的Directory Entry

- 写入文件：
  1. 分配数据块，往数据块里写东西。
  2. 更新Inode，指向刚才分配的数据块，更新Inode里的文件大小字段，最后一次修改日期字段。



一个Disk Block只能在buffer cache中出现一次：

如果buffer cache中有两份block 33的cache将会出现问题。假设一个进程要更新inode19，另一个进程要更新inode20。如果它们都在处理block 33的cache，并且cache有两份，那么第一个进程可能持有一份cache并先将inode19写回到磁盘中，而另一个进程持有另一份cache会将inode20写回到磁盘中，**并将inode19的更新覆盖掉。**



既然sleep lock是基于spinlock实现的，为什么对于block cache，我们使用的是sleep lock而不是spinlock？

> 学生回答：因为磁盘的操作需要很长的时间。

是的，这里其实有多种原因。对于spinlock有很多限制，其中之一是加锁时中断必须要关闭。所以如果使用spinlock的话，当我们对block cache做操作的时候需要持有锁，那么我们就永远也不能从磁盘收到数据（因为中断被关了，CPU收不到）。或许另一个CPU核可以收到中断并读到磁盘数据，但是如果我们只有一个CPU核的话，我们就永远也读不到数据了。出于同样的原因，也不能在持有spinlock的时候进入sleep状态（因为Sleep是系统调用，系统调用也会触发中断，或者叫软中断）。所以这里我们使用sleep lock。sleep lock的优势就是，**我们可以在持有`SleepLock`锁的时候不关闭中断**。我们可以在磁盘操作的过程中持有锁，我们也可以长时间持有锁。当我们在等待sleep lock的时候，我们并没有让CPU一直空转，我们通过sleep将CPU出让出去了。

### 补上对Sleeplock.c的阅读

基本上可以理解成，这里有二重锁，一是外面一重的自旋锁，二是内层的`Locked`字段。当要修改`Locked`字段获得锁的时候，先要去拿到外层的自旋锁。如果内层的锁已经被拿到了，那就调用Sleep，原子性地释放掉外层的自旋锁并陷入睡眠。

而，要释放SleepLock的时候，除了要修改`Locked`字段，还要调用Wakeup唤醒可能存在的，睡眠中等待这个SleepLock的其他进程。

```c
// Long-term locks for processes
struct sleeplock {
  uint locked;       // Is the lock held?
  struct spinlock lk; // spinlock protecting this sleep lock
  
  // For debugging:
  char *name;        // Name of lock.
  int pid;           // Process holding lock
};

void
acquiresleep(struct sleeplock *lk)
{
  acquire(&lk->lk);
  while (lk->locked) {
      //放到自旋锁并进入睡眠状态。
    sleep(lk, &lk->lk);
  }
  lk->locked = 1;
  lk->pid = myproc()->pid;
  release(&lk->lk);
}

void
releasesleep(struct sleeplock *lk)
{
  acquire(&lk->lk);
  lk->locked = 0;
  lk->pid = 0;
  wakeup(lk);
  release(&lk->lk);
}
```



## Lecture 15

### 阅读Chapter 8 有关Logging的章节

#### 8.4 Logging layer

我们为什么需要Logging？

很简单，因为许多文件系统的操作往往是设计多次对硬盘的写入的，一旦发生崩溃，不完整的写入（a subset of the writes）会导致文件系统处于一个不一致的状态。

> One of the most interesting problems in file system design is crash recovery. The problem arises because many file-system operations involve multiple writes to the disk, and a crash after a subset of the writes may leave the on-disk file system in an inconsistent state.



这样的问题的举例：比如导致已分配但是未被引用的Disk Block的出现（类似内存泄漏），比如实际上已经被标记为Free但是还被某个inode引用的Disk Block。

这样的问题的后果：

> The latter is relatively benign, but an inode that refers to a freed block is likely to cause serious problems after a reboot. After reboot, the kernel might allocate that block to another file, and now we have two different files pointing unintentionally to the same block. If xv6 supported multiple
>
> users, this situation could be a security problem, since the old file’s owner would be able to read and write blocks in the new file, owned by a different user.

Xv6使用的Logging设计是这样的：一个形式很简单的Logging

一个Xv6系统调用，它并不会直接就去写入硬盘的文件系统的数据结构（不会直接就奔文件系统，往里面写东西）。相反，它先往硬盘上写一个Log，这个Log里包含了对于它想做的所有硬盘写入的描述，一旦它在Log里记下了它全部的写入，它再往硬盘上写入一个*Commit Log*，这个*Commit Log*表明这个Log包含了一个完整的操作。然后，在这个时间点上，系统调用再去真正地把那些写入给写进硬盘文件系统。如果这次写入也成功完成，系统调用就把它刚才写的Log给抹除掉。



> An xv6 system call does not directly write the on-disk file system data structures. Instead, it places a description of all the disk writes it wishes to make in a *log* on the disk. Once the system call has logged all of its writes, it writes a special *commit* record to the disk indicating that the log contains a complete operation. At that point the system call copies the writes to the on-disk file system data structures. After those writes have completed, the system call erases the log on disk.



如果系统崩溃并且重启了，那么（在运行任何进程之前）文件系统就可以按照如下步骤，借助Log从崩溃中恢复：

- 对于有Commit Log的Log，按照Log里的描述Replay一遍里面的操作
- 对于不完整的Log，直接忽略，擦除掉它。

> -  If the log is marked as containing a complete operation, then the recovery code copies the writes to where they belong in the on-disk file system. 
> - If the log is not marked as containing a complete operation, the recovery code ignores the log. The recovery code finishes by erasing the log.



为什么这样的Log能够解决文件系统操作期间的崩溃问题？

很简单，对于单个的操作，我们可以把崩溃发生的时机分为三类：写入Commit Log前、已经写入Commit Log，但是还没有完成对文件系统的写入、文件系统的写入也完成了。

- 第一类下，由于文件系统本身并未改变，而Log由于没有Commit Log，会被直接擦除。所以硬盘的状态就和那个操作根本没发生的时候是一模一样的
- 第二类下，借助Log可以Replay一遍崩溃的操作。而对于文件系统同一个地方写入同样的内容，这个操作又是幂等的，所以Replay不会造成结果的改变。
- 第三类下，那不就是已经完成了吗，有什么好说的。

不管是哪种情况，日志都让操作是原子性的：崩溃恢复之后，操作的全部写入要么全部发生，要么全部都不发生。

> -  If the crash occurs before the operation commits, then the log on disk will not be marked as complete, the recovery code will ignore it, and the state of the disk will be as if the operation had not even started. 
> - If the crash occurs after the operation commits, then recovery will replay all of the operation’s writes, perhaps repeating them if the operation had started to write them to the on-disk data structure. 
>
> In either case, the log makes operations atomic with respect to crashes: after recovery, either all of the operation’s writes appear on the disk, or none of them appear.

#### 8.5 Log design

Log驻留在硬盘上一个已知的位置，在Super Block中指定。Log包含一个头部块，其后是一系列的Logged Block（ a sequence of updated block copies）。头部块包含一个扇区号的数组，每个Logged Block一个。头部块的数组要么是0，表示在Log中没有事务，要么是非0，表示Log中包含一个已提交的包含同样个数的Logged Block的事务。

Xv6在一个事务提交之后写入头部块，不会在事务提交之前写入头部块。在将Logged Block复制回文件系统之后，将数字置为0。因此一个事务中途的Crash会导致一个为0的数字，事务提交之后的Crash则是非零的数字。



为了允许不同进程并发地执行文件系统操作，日志系统会将多个系统调用的写入累计在一起，放入一个事务中。所以单单一个事务提交可能就包含多个系统调用的写入。

为了避免在事务之间拆分系统调用，日志记录系统只有在没有文件系统调用正在进行时才会提交。

将多个事务一起提交的想法称为组提交（*group commit*）。组提交减少了磁盘操作的数量，因为它将提交的固定成本分摊到多个操作上。组提交还同时为磁盘系统提供了更多的并发写入，也许允许磁盘在单次磁盘旋转期间将它们全部写入。



Xv6在硬盘上使用固定数量的空间来容纳Log。一个事务中的系统调用的写入的总和必须能够在这个空间中放得下。

这也就导致了系统调用一次性能写入的Disk Block的个数是不能超过Log空间的上限的。但是这对大部分的系统调用来说不算问题。但是对于`write`这样可能写入大文件的系统调用来说，Xv6会将其划分成多个更小的能够放得下的批量写入。

另外一个后果就是，日志系统不会允许一个系统调用开始，除非它能保证系统调用的写入能够放入Log的剩余空间中。

#### 8.6 Code: logging

首先是Log Header的结构：

> The header block contains an array of sector numbers, one for each of the logged blocks, and the count of log blocks.

```c
/*parama.h*/
#define MAXOPBLOCKS  10  // max # of blocks any FS op writes
#define LOGSIZE      (MAXOPBLOCKS*3)  // max data blocks in on-disk log

/*log.c*/

// Contents of the header block, used for both the on-disk header block
// and to keep track in memory of logged block# before commit.
struct logheader {
  int n; // Logged Block的数目?
  int block[LOGSIZE]; // 应该是被Update的Data Block的块号的数组
};
```

然后是log本体：包含一个自旋锁，用于保证并发安全。

```c
struct log {
  struct spinlock lock;
  int start;
  int size;
        // log.outstanding counts the number of system calls that have reserved log space
  int outstanding; // how many FS sys calls are executing.
  int committing;  // in commit(), please wait.
  int dev; //设备号吧
  struct logheader lh; //Log header
};
struct log log;

void
initlog(int dev, struct superblock *sb)
{
  if (sizeof(struct logheader) >= BSIZE)
    panic("initlog: too big logheader");

  initlock(&log.lock, "log");
  log.start = sb->logstart;
  log.size = sb->nlog;
  log.dev = dev;
  recover_from_log();
}
```

在文件系统的系统调用中典型的Log用法：

```c
begin_op();
...
bp = bread(...);
bp->data[...] = ...;
log_write(bp);
...
end_op();
```

按照这个框架依次看看：



`begin_op`：

代码本身很简单，如果Log系统正在提交的话，就陷入睡眠。否则检查一下剩余的空闲Log空间是否足够。因为`begin_op`是假定每次文件系统的系统调用都会使用`MAXOPBLOCKS`的空间（也就是保守地做最坏的情况估计，虽然我不知道它实际上会用掉多少，但是我知道它最多可以用掉多少），所以`if`里的条件是用`(log.outstanding+1)*MAXOPBLOCKS`。

> Incrementing log.outstanding  both reserves space and prevents a commit from occuring during this system call
>
> 因为只有在 log.outstanding 到达0的时候，才会写入Commit Log，所以 log.outstanding +=1 （使得log.outstanding  > 0）阻止了这个系统调用期间的Commit

```c
// called at the start of each FS system call.
void
begin_op(void)
{
  acquire(&log.lock);
  while(1){
    if(log.committing){
      sleep(&log, &log.lock);
    } else if(log.lh.n + (log.outstanding+1)*MAXOPBLOCKS > LOGSIZE){
      // this op might exhaust log space; wait for commit.
      sleep(&log, &log.lock);
    } else {
      log.outstanding += 1;    // reserves space and prevents a commit from occuring during this system call
      release(&log.lock);
      break;
    }
  }
}
```

然后是`log_write`： `log_write`作为`bwrite`的代理，可以代替`bwrite`的使用。调用者已经编辑过了某个Data Block对应的Buffer的`buf`的`data`数组， `log_write`只是将其对应的块号（扇区号和块号是一一对应的，只不过一个是物理意义，一个是逻辑意义）记录下来，并且调用`bpin`将这块`buf`给固定在内存里，防止被替换出去（这一块缓存必须一直待在内存里直到Commit）。

`log_write`**并不会做写入硬盘的工作，写入硬盘的工作是`Commit/write_log`在做**

为什么需要固定缓冲块直到Commit，不让它被替换出去（替换出去实际上就是写回硬盘原有位置）？

> 如果在事务提交之前就允许缓存的Block被替换出去，那么可能会导致如下问题：
>
> - 因为我们知道事务是原子性地，所以里面的操作要么全部发生要么全部不发生。假如中途就写入之后，事务就崩溃了怎么办？会导致数据一致性的问题。而且日志的原则就是，在写入Commit Log之后，才能把更改统一写入文件系统。这样做违背了这个原则
> - 另外一个就是，如果崩溃了，这样做会让回滚起来比较麻烦。因为“在Commit Log之后才统一把更改写回文件系统”是让日志得以工作的很重要的原因，因为这个原则的存在，使得文件系统的更改只有两种可能：完全的更改（Commit Log的写入就标志了完全的更改），或者完全不更改（Commit Log未写入就标志了完全不更改，因为不完整的日志会直接被忽略），但是假如我们破坏这个原则，允许在事务中途就写回文件系统，那么就存在一种部分更改的状态，崩溃的时候Commit Log还没写，但是有一部分更改已经实际写入了文件系统。
> - 

另外这里还使用了一种叫做*absorption*（合并）的技术：如果在同一事务中对同一磁盘块进行多次写入，传统的做法是将每次写入都记录在日志中，并在事务提交时将它们逐个写回磁盘。然而，通过使用"absorption"优化技术，文件系统能够检测到多次写入同一磁盘块的情况，并只记录该磁盘块的最终状态。换句话说，它将多个写入操作合并为一个操作，只需要将块写回磁盘一次，从而节省了日志空间并提高了性能。

具体而言，就是假如一个块被同一个事务写入了多次，那么这多次写入都分配在同一个Log Slot。

> log_write notices when a block is written multiple times during a single transaction, and **allocates that block the same slot in the log**

```c
// Caller has modified b->data and is done with the buffer.
// Record the block number and pin in the cache by increasing refcnt.
// commit()/write_log() will do the disk write.
//
// log_write() replaces bwrite(); a typical use is:
//   bp = bread(...)
//   modify bp->data[]
//   log_write(bp)
//   brelse(bp)
void
log_write(struct buf *b)
{
  int i;

  if (log.lh.n >= LOGSIZE || log.lh.n >= log.size - 1)
    panic("too big a transaction");
  if (log.outstanding < 1)
    panic("log_write outside of trans");

  acquire(&log.lock);
  for (i = 0; i < log.lh.n; i++) {
      //如果是之前已经写过一次以上的Block，那么这里会找到对应的Log Slot,直接break,在后续很自然地就日志合并了
    if (log.lh.block[i] == b->blockno)   // log absorbtion
      break;
  }
    //假如是第一次写入的Block的话，i会等于log.lh.n。所以下面可以通过检查这一点来判断是否是新的Block
  log.lh.block[i] = b->blockno;
  if (i == log.lh.n) {  // Add new block to log?
    bpin(b); // 实现放下面了。把新的固定住
    log.lh.n++;
  }
  release(&log.lock);
}

/*bio.c*/
void
bpin(struct buf *b) {
  acquire(&bcache.lock);
  b->refcnt++;
  release(&bcache.lock);
}
```

然后是与`begin_op`对应的`end_op`：

`begin_op`对应地会把 `log.outstanding` 减少1。如果 ` log.outstanding `到达0了，就调用`Commit()`来提交。

```c
// called at the end of each FS system call.
// commits if this was the last outstanding operation.
void
end_op(void)
{
  int do_commit = 0;

  acquire(&log.lock);
  log.outstanding -= 1;
  if(log.committing)
    panic("log.committing");
  if(log.outstanding == 0){
    do_commit = 1;
    log.committing = 1; //这样后续到来的begin_op就会Sleep
  } else {
    // begin_op() may be waiting for log space,
    // and decrementing log.outstanding has decreased
    // the amount of reserved space.
    wakeup(&log);
  }
  release(&log.lock);

  if(do_commit){
    // call commit w/o holding locks, since not allowed
    // to sleep with locks.
    commit();
    acquire(&log.lock);
    log.committing = 0; // 这样被唤醒的begin_op就不会再一次Sleep
    wakeup(&log); // 唤醒begin_op
    release(&log.lock);
  }
}
```

`Commit`比较简单，复杂的地方在其调用的几个函数：

- `write_log`将log从内存写入硬盘的Log区
- `write_head`更新硬盘里的Log Header
- `install_trans`将数据真正地写入文件系统里。

```c
static void
commit()
{
  if (log.lh.n > 0) {
    write_log();     // Write modified blocks from cache to log
    write_head();    // Write header to disk -- the real commit【在这之后发生崩溃，只会导致Log的Replay】
    install_trans(); // Now install writes to home locations【读Log，把更改写入到文件系统里该写的地方】
    log.lh.n = 0;
    write_head();    // Erase the transaction from the log【文件系统里也成功写入了，该把Log擦除了】
  }
}
```

所以接下来依次看看这几个函数：直接写注释里了

```c
 // Copy modified blocks from cache to log.
static void
write_log(void)
{
  int tail;

  for (tail = 0; tail < log.lh.n; tail++) {
    struct buf *to = bread(log.dev, log.start+tail+1); // log block 
    struct buf *from = bread(log.dev, log.lh.block[tail]); // cache block
    memmove(to->data, from->data, BSIZE); //复制data数组,也就是数据本体
    bwrite(to);  // write the log
    brelse(from);
    brelse(to);
  }
}
```

```c
// Write in-memory log header to disk.
// This is the true point at which the
// current transaction commits.
static void
write_head(void)
{
  struct buf *buf = bread(log.dev, log.start); //读取硬盘里的Log Header到buf
  struct logheader *hb = (struct logheader *) (buf->data); //转换一下类型
  int i;
  hb->n = log.lh.n;
  for (i = 0; i < log.lh.n; i++) { //循环遍历复制一遍block[]数组
    hb->block[i] = log.lh.block[i];
  }
    //把Log header写回硬盘去 
  bwrite(buf);
  brelse(buf);
}
```

```c
// Copy committed blocks from log to their home location
static void
install_trans(void)
{
  int tail;

  for (tail = 0; tail < log.lh.n; tail++) { 
    struct buf *lbuf = bread(log.dev, log.start+tail+1); // read log block
    struct buf *dbuf = bread(log.dev, log.lh.block[tail]); // read dst
    memmove(dbuf->data, lbuf->data, BSIZE);  // copy block to dst
    bwrite(dbuf);  // write dst to disk
    bunpin(dbuf); //这里为什么需要unpin dbuf?是因为dst被log_write bpin()过吗?我觉得有点奇怪
    brelse(lbuf);
    brelse(dbuf);
  }
}
```

在系统启动的时候，在第一个用户进程运行之前，`fsinit`会初始化文件系统，同时也会做相应的日志恢复工作。`fsinit`调用`initlog`， `initlog`则调用`recover_from_log`来做可能的崩溃恢复。

```c
void
initlog(int dev, struct superblock *sb)
{
  if (sizeof(struct logheader) >= BSIZE)
    panic("initlog: too big logheader");

  initlock(&log.lock, "log");
  log.start = sb->logstart;
  log.size = sb->nlog;
  log.dev = dev;
  recover_from_log();
}

static void
recover_from_log(void)
{
  read_head();
  install_trans(); // if committed, copy from log to disk
  log.lh.n = 0;
  write_head(); // clear the log
}
```

### Lecture

文件系统的崩溃问题并不出在操作的顺序问题上，不使用Logging保证原子性，无论怎样调整操作顺序，总会出现问题。问题并不在于操作的顺序，而在于我们这里有多个写磁盘的操作，这些操作必须作为一个原子操作出现在磁盘上。



日志系统的特点：

- 首先，它可以确保文件系统的系统调用是原子性的。比如你调用create/write系统调用，这些系统调用的效果是要么完全出现，要么完全不出现，这样就避免了一个系统调用只有部分写磁盘操作出现在磁盘上。
- 其次，它支持快速恢复（Fast Recovery）。在重启之后，我们不需要做大量的工作来修复文件系统，只需要非常小的工作量。
- 最后，原则上来说，它可以非常的高效，尽管我们在XV6中看到的实现不是很高效。





Log的基本工作方式和基本思想：当需要更新文件系统时，我们并不是更新文件系统本身。而是先把我们要做的更新给写入到Log里面去。

在讲述Xv6的Log工作方式之前，先说明一下Xv6的Log在硬盘上的结构是怎么样的：

我们将整个Log区分为两部分：一部分是Log Header，另外一部分是真正写入Log的地方。在下图中，由红框圈定的是Header，N可以看做是代表有效的log block的数量，而那个由块号构成的数组，则与每个Disk Block（Log Block）一一对应（也即每个log block的实际对应的block编号）。比如第1个Disk Block的Home Location就在块号数组的第1号单元。

<img src="https://i.imgur.com/i0CmBxD.png" alt="image-20230820145040803" style="zoom:80%;" />



接下来举个简单的例子说明：

假设我们在内存中缓存了bitmap block，也就是block 45。当需要更新bitmap时，我们并不是直接写block 45，而是将数据写入到log中（比如写入Disk Block数组的2号单元），并记录这个更新应该写入到block 45（在块号数组的2号单元写一个45）。对于所有的写 block都会有相同的操作。

任何一次写操作都是先写入到log，我们并不是直接写入到block所在的位置，而总是先将写操作写入到log中。

当我们在log中存储了所有写block的内容时，就将内容真正意义上写入文件系统，只需要将block从log分区移到文件系统分区。我们知道第一个操作该写入到block 45，我们会直接将数据从log写到block45，第二个操作该写入到block 33，我们会将它写入到block 33，依次类推。

一旦完成了，就可以清除log。清除log实际上就是将属于同一个文件系统的操作的个数设置为0。



所以Xv6的Log的流程是这样的：

1. 写Log
2. 提交Log（通过将N置为对应的个数）
3. 安装Log（指写回文件系统）
4. 清除Log（把N置为0）



那么在开机的时候，文件系统会检查N的大小，如果N等于0，就说明没有回复工作需要做。如果N大于0，就从流程的第3步开始执行，修复文件系统（也就是重新把Log写回文件系统，然后清除Log）。

`log_write`也强调一点：**它不是把Log写入硬盘的地方**，它只是往Log header的块号数组里面添加了相应的块号。比如我修改了33号Block，调用`Log_write`只是往Log Header（在内存里的！！！）的块号数组里添加了一个33，而33号Block本身的内容，要在`write_log`里才会被写入Log。

而`Write_log`是被`end_op`调用的，所以要在事务的结束开始的时候，33号Block的内容才会写入硬盘里的Log。

LogHeader本身，也不是在这里写入了硬盘，而是在`write_head`。

关于`write_log`函数再强调几点：

- 调用`bread`不一定是真的从硬盘里读那一块，也可能是直接找了内存里缓存的。而在这里就是第二种情况。
- 它的思想就是用`bread`读取内存里修改过的Block以及一个LogBlock，然后调用`bwrite`写入LogBlock。

`write_log`这个位置并不是Commit Point，我们还没有commit，现在我们只是将block存放在了log中。如果我们在这个位置也就是在write_head之前crash了，那么最终的表现就像是transaction从来没有发生过。

`write_head`完成的地方才是Commit Point。就算是`Write_head`的中途，都不是完成了Commit。



当前的Xv6的日志系统还面临哪些挑战？

- 第一个，需要pin和unpin位于buffer cache中的block。为了不违反Write Ahead Rule
- 第二个，Log空间的问题。文件系统操作必须适配log的大小。Xv6的Log空间只有30个Block大小，但是Xv6的单个操作不会超过这个上限。对于连续写入大文件的操作，`file_write(write真正调用的函数)`会把一个写操作会被分割成多个小一些的写操作，每一个小的写操作通过独立的transaction写入。这样文件系统本身不会陷入不正确的状态中（因为我们对文件系统的要求就是保持一致性，而不是保持正确性，文件是可以丢的，但是文件系统是不能不一致的，这还是最终要回到Log的目的上来，Log的目的并不是保证正确性，而是保持一致性）。
- 第二点五个，还需要注意，因为block在落盘之前需要在cache中pin住，所以buffer cache的尺寸也要大于log的尺寸。

第三个挑战比较重量级：事务的并发。

先构造一个场景：

> 假设我们有一段log，和两个并发的执行的transaction，其中transaction t0在log的前半段记录，transaction t1在log的后半段记录。可能我们用完了log空间，但是任何一个transaction**都还没完成**。现在我们能提交任何一个transaction吗？我们不能，因为这样的话我们就提交了一个部分完成的transaction，这违背了write ahead rule，log本身也没有起到应该的作用。所以必须要保证多个并发transaction加在一起也适配log的大小。

Xv6对这个问题的解决就是限制同时并发的文件系统操作的个数，在begin_op中先检查Log空间够不够，然后再决定是现在就开始一个新的事务，还是先让它Sleep一会。

这里另外还要提的一个东西就是Group Commit，也就是说Xv6的Log并不一定是一个事务提交一次的，因为Log分区本身的大小是足以放下多个事务的，所以它是可以按顺序容纳多个事务，然后一起提交的。



关于`begin_op`再提几点：

- 不能在Log正在Commiting的时候开始新的事务，这时候`begin_op`会Sleep，等待被唤醒。
- 如果加上当前的事务，Log空间就不够用的话，那么`begin_op`也会Sleep，等待目前进行的事务提交腾出空间。
- ，如果当前操作可以继续执行，需要将log的outstanding字段加1，最后再退出函数并执行文件系统操作。



然后是对应的`end_op`：

- 最开始首先会对log的outstanding字段减1，因为一个transaction正在结束。
- 同样的，也不能在Log系统正在Commiting的时候结束一个事务。并且这种可能性是不存在的，所以假如出现的话Xv6就Panic
- 如果当前操作是整个并发操作的最后一个的话（log.outstanding == 0），接下来立刻就会执行commit；
- 如果当前操作不是整个并发操作的最后一个的话，我们需要唤醒在begin_op中sleep的操作，让它们检查是不是能运行。





#### 提问

> 学生提问：如果一个进程向磁盘写了一些数据，但是在commit之前进程出现了故障，假设故障之后进程退出了，这样会有问题吗？
>
> Frans教授：简单回答是没问题，因此磁盘不会被更新，所以效果就像文件系统操作没有发生过一样。并且进程并不能在故障后恢复，唯一能在故障之后还能保持的是保存在磁盘中的状态。（注，应该是没有理解问题。进程通过write系统调用成功写入的数据，就算在成功落盘之前进程异常退出了，内核还是会写入到磁盘中，前提是内核还在运行。）

#### 为什么要log_writ调用了bpin?

因为假如不固定的话，有可能会导致内存里的Block缓存处于缓存替换的原因，被提前写回文件系统。**这是违反Write AheadRule的**

Xv6的Log是Write Ahead Log，也即明确规定了，对文件系统的写入不能先于对Log的写入，任何对文件系统所作出的更改，都必须



## Lecture 16

### Journaling the Linux ext2fs Filesystem (1998)

The paper uses "journal" to refer to the same idea that xv6 calls a "log". The file system described in the paper is now called ext3.

To help you read the paper, try to answer the following question for yourself:

Mid-way down the left column on page 6, the Journaling paper says "However, until we have finished syncing those buffers, we cannot delete the copy of the data in the journal." Give a concrete example in which removing this rule would lead to disaster.



尽管日志文件系统的事务的大部分术语和技术都是来自数据库的，但是传统的数据库的事务和文件系统的事务有很大的不同。

最大的两个区别之一是，数据库的事务是有Abort的，但是文件系统的事务是没有的，在我们开始正式对文件系统做出修改的时候，我们可以确定那个改变是能够合法完成的（因为已经写入了Commit Log了，但是数据库即使写入了Commit，也可以Abort，应该是这个意思？）



第二个不同之处是，文件系统的事务的生命周期通常都是很短的，而数据库则会存在那种长时间持续的事务。这意味着，文件系统可以让很多个事务按照严格的顺序一一提交到硬盘去，而不会太影响性能。但是假如存在个别事务持续时间很久，就不能这样，否则会影响整体性能（在它后面的短时事务要等待它很久），所以就必须让事务们以不会相互冲突的任意顺序去提交。

基于这两点，甚至可以做一个更大的改进——合并事务。合并事务就是，我们"every so often"地（应该说每隔一段时间，不一定是周期的）创建一个新的事务，并且允许所有的文件系统服务调用都将他们所做的更改添加到这个单一的全局复合事务里面，**而不是为每一个文件系统更新创建一个独立的事务**。



这样做的好处是，对于那些频繁被更改的Block（比如BitMap所在的Block），在一个复合事务的生命期间，它们的更改只需要提交到硬盘一次。

> Any  block which is updated many times during the life  of a compound transaction need only be committed  to disk once.

而对于什么时候提交当前的复合事务，开始一个新的事务的决定，应该由用户来控制。毕竟这是一个影响系统性能的tradeoff：一个复合事务等待越久，那么能够合并的文件系统操作就越多，长期来看需要的IO操作越少（意思是100个操作，如果每10个合并一次需要10次IO，如果等待时间拉长10倍，100次合并，只用一次IO）。但是，长时间等待的复合事务其实违背了“文件系统的事务的生命周期通常都是很短的”的特点，更长的事务需要占用更多的内存和硬盘，并且如果中途出现崩溃的话，丢失掉的东西就更多。

> The longer a commit waits, the more  filesystem operations can be merged together in the  log and so less IO operations are required in the long term. 
>
> However, longer commits tie up larger  amounts of memory and disk space, and leave a larger window for loss of updates if a crash occurs. 
>
> They may also lead to storms of disk activity which make filesystem response times less predictable.

### Lecture

Xv6的Logging System的性能差在哪了？

XV6中的任何一个例如create/write的系统调用，**需要在整个transaction完成之后才能返回**。所以在创建文件的系统调用返回到用户空间之前，它需要完成所有end_op包含的内容，这包括了：

- 将所有更新了的block写入到log
- 更新header block
- 将log中的所有block写回到文件系统分区中
- 清除header block

这里每个系统调用需要等待它包含的所有写磁盘结束，XV6的系统调用对于写磁盘操作来说是同步的（synchronized）

#### 因为总是区分不清Write Back和Write Through，特此补充

> **写回缓存（Write Back Cache）**： 在写回缓存策略中，当CPU执行写操作时，数据首先被写入缓存，而不是立即写入主存。这意味着缓存中的数据可能与主存中的数据不同步。只有当缓存行被替换出缓存时，才会将被修改的数据写回主存。这种方式可以减少主存写入的次数，从而提高写入操作的效率。
>
> 优点：
>
> - 减少主存写入操作，提高写入性能。
> - 减少了对主存带宽的压力，可以在一定程度上提高系统整体性能。
>
> 缺点：
>
> - 数据不一致性风险：缓存中的数据可能与主存不同步，如果在数据被写回主存之前发生系统崩溃或其他异常情况，可能导致数据一致性问题。
> - 需要额外的硬件支持来管理缓存中的脏数据（已被修改但尚未写回主存的数据）。
>
> **写直通缓存（Write Through Cache）**： 在写直通缓存策略中，当CPU执行写操作时，数据会同时写入缓存和主存，确保缓存中的数据与主存保持同步。这样可以保证数据一致性，但写入操作可能会稍微变慢，因为要同时更新缓存和主存。
>
> 优点：
>
> - 数据一致性：缓存中的数据与主存保持同步，减少了数据不一致性的风险。
> - 简化硬件：不需要额外的硬件支持来管理脏数据。
>
> 缺点：
>
> - 写入性能相对较低：每次写入操作都需要写入缓存和主存，可能导致写入操作的速度较慢。
> - 增加主存带宽压力：频繁的写入操作可能会占用主存带宽，影响系统整体性能。

#### ext3 FS的Log结构

内存作为硬盘的缓存是Write Back类型的。文件系统本身也是基于Inode思想的，目录也是层次化的树形结构。会有bitmap block来表明每个data block是被分配的还是空闲的。在磁盘的一个指定区域，会保存log。



ext3维护了一些事务信息，每个事务的信息有：

- 一个序列号
- 一系列该transaction修改的block编号。这些block编号指向的是在Memory中的block，因为任何修改最初都是在Memory中完成。
- 以及一系列的handle，handle对应了系统调用，并且这些系统调用是transaction的一部分，会读写cache中的block

每个时刻都只有一个活跃的事务，但是Log上可能有多个有效事务（所谓有效事务就是有Commit Block但是还没写回文件系统的事务）。



ext3的Log分区的结构：

最开始是一个Log分区的Super Block（与整个文件系统的Super Block区分开），Super Block里包含了Log分区中，第一个有效的事务的起始位置和序列号。全部的Log Block是Log分区内的一段固定大小的连续Block，就像Xv6的30个Block。

- 起始位置：磁盘上log分区的block编号
- 序列号：就是事务的序列号

每个事务在Log分区上的结构：

- descriptor block：其中包含了log数据对应的实际block编号。与XV6中的header block很像，有一个由块号组成的数组，记录了中间的每个数据块的Home Location。
- 中间是更新过的数据块（或者是BitMap块，Inode块）
- 当一个transaction完成并commit了，会有一个commit block。也就是所谓的Commit Log
- descriptor block和commit block会以一个32bit的魔法数字作为起始，以区分三种不同类型的Block。
  - **如果Data Block本身的数据的前32比特就以Magic Number起始呢**？


> logging系统需要能区分一个以魔法数字作为起始的descriptor block和一个以魔法数字作为起始的data block。ext3是这样做的，当它向log写一个block时，如果这个block既不是descriptor block也不是commit block，但是又以魔法数字作为起始，**文件系统会以0替换前32bit，并在transaction的descriptor block中为该data block设置一个bit。这个bit表示，对应的data block本来是以魔法数字作为起始，但是现在我们将其替换成了0。而恢复软件会检查这个bit位，在将block写回到文件系统之前，会用魔法数字替换0。**
>
> **因此，在log中，除了descriptor和commit block，不会有其他的block以这32bit的魔法数字作为起始。所以我们不会有模棱两可的判断，如果一个commit block之后的block以魔法数字作为起始，那么它必然是一个descriptor block。**

<img src="https://i.imgur.com/J4qx4jy.png" alt="image-20230821204833865" style="zoom: 67%;" />



整个Log分区的结构：整个Log分区中可能有多个transaction，commit block之后可能会跟着下一个transaction的descriptor block，data block和commit block。所以log可能会很长并包含多个transaction。我们可以认为super block中的起始位置和序列号属于最早的，排名最靠前的，并且是有效的transaction。

**并且Log分区的Log Blocks是一个环形数组，** 其思想类似ArrayList。**只不过没有扩容，只有空间复用**。这也是为什么我们需要在Log Super Block里记录第一个有效事务的起始位置，因为它并不总是在Log Blocks的第一个Block处开始。

如下图，第三个有效事务占据的硬盘空间将会从灰色框的左边界开始占据。

<img src="https://i.imgur.com/aGk15w1.png" alt="image-20230821210419523" style="zoom:67%;" />



> Log中会有多个transaction，但是一个时间只有一个正在进行的transaction。当前正在进行的transaction只存在于内存中，对应的系统调用只会更新内存中的文件系统block。当ext3决定结束当前正在进行的transaction，它会做两件事情：首先开始一个新的transaction，这将会是下一个transaction；其次将刚刚完成的transaction写入到磁盘中，这可能要花一点时间。
>
> 所以完整的故事是，磁盘上的log分区有一系列旧的transaction，这些transaction已经commit了，除此之外，还有一个位于内存的正在进行的transaction。在磁盘上的transaction，只能以log记录的形式存在，并且还没有写到对应的文件系统block中。**logging系统在后台会从最早的transaction开始，将transaction中的data block写入到对应的文件系统中**。当整个transaction的data block都写完了，之后logging系统才能释放并重用log中的空间。所以log其实是个循环的数据结构，如果用到了log的最后，logging系统会从log的最开始位置重新使用。

#### ext3是如何提高Logging性能的?

ext3通过3种方式提升了性能：

- 首先，它提供了异步的（asynchronous）系统调用，也就是说系统调用在写入到磁盘之前就返回了，系统调用只会更新缓存在内存中的block，并不用等待写磁盘操作。不过它可能会等待读磁盘。
- 第二，它提供了批量执行（batching）的能力，可以将多个系统调用打包成一个transaction。
- 最后，它提供了并发（concurrency）。

简而言之：异步、批量处理、并发



异步：

首先是异步的系统调用。这表示系统调用修改完位于内存中的block之后就返回，并不会触发写磁盘。这样做的优势是系统调用能够快速的返回（因为不用等待写入磁盘了）。同时它也使得I/O可以并行的运行，也就是说应用程序可以调用一些文件系统的系统调用，但是应用程序可以很快从系统调用中返回并继续运算，与此同时**文件系统在后台会并行的完成之前的系统调用所要求的写磁盘操作**。这被称为I/O concurrency，如果没有异步系统调用，很难获得I/O concurrency，或者说很难同时进行磁盘操作和应用程序运算，因为同步系统调用中，应用程序总是要等待磁盘操作结束才能从系统调用中返回。

另一个异步系统调用带来的好处是，它使得大量的批量执行变得容易。

异步系统调用的缺点是**系统调用的返回并不能表示系统调用应该完成的工作实际完成了。**原因很简单，因为系统调用的返回不意味着这一次调用写入的数据，已经成功地写回了硬盘，内核的文件系统可能并没有来得及把它从内存里复制到硬盘上（更准确一点，复制到Log，然后从Log复制到文件系统）。

> 所以这就带来了一个很简单的问题：当Write系统调用返回的时候，程序突然崩溃了，它写入的东西就一定在硬盘里了吗？
>
> 我对这个问题的回答是：
>
> 对于Xv6来说，回答是肯定的。因为Xv6下的文件系统的系统调用都是同步的，这表示，当Write系统调用返回的时候，它一定已经完成了对硬盘的写入。
>
> 而对于Linux来说，回答是不能确定。因为Write系统调用是异步的，它返回只能说明它成功地将东西写入了内存。而将内存里的东西复制回硬盘的工作，是内核在后台负责的。而我们并不能确定Write返回的那一刻，内核是否在做这个工作。并且，程序突然崩溃这个事也要分情况讨论，如果只是程序本身崩溃了，内核还在正常运转，那么东西就应该是能保住的，如果内核都崩溃了，那东西就很可能保不住了（如果运气真的巨好，也可能东西保住了）。

所以为了解决这个问题，文件系统一个系统调用，叫做`fsync`，所有的UNIX都有这个系统调用。这个系统调用接收一个文件描述符作为参数，它会告诉文件系统去完成所有的与该文件相关的写磁盘操作，在所有的数据都确认写入到磁盘之后，`fsync`才会返回。

- `fsync`，立刻去完成这个文件在内存与硬盘之间的同步工作，不要等啦！就这个意思。其实也可以说就是一个`flush`，它`flush`了所有文件相关的写磁盘操作到了磁盘中。
- 但是其实许多程序并不会调用fsync，并且乐于获得异步系统调用带来的高性能。



批量执行（批处理）：

在任何时候，ext3只会有一个open transaction。ext3中的任何一个transaction可以包含多个不同的系统调用。

所以ext3的Log系统实际上的工作方式是：

它首先会宣告要开始一个新的transaction，接下来的几秒所有的系统调用都是这个大的transaction的一部分。我认为默认情况下，ext3每5秒钟都会创建一个新的transaction，所以每个transaction都会包含5秒钟内的系统调用，这些系统调用都打包在一个transaction中。在5秒钟结束的时候，ext3会commit这个包含了可能有数百个更新的大transaction。

为什么它提高了性能？

- 它在多个系统调用之间分摊了transaction带来的固有的损耗。固有损耗，比如磁盘的寻道时间、旋转时间。比如写transaction的descriptor block和commit block。现在可以对一批系统调用只做一次这些工作，而不是每个系统调用做一次这样的工作。
- 它可以更容易触发write absorption。经常会有这样的情况，你有一堆系统调用最终在反复更新相同的一组磁盘block。举个例子，如果我创建了一些文件，我需要分配一些inode，inode或许都很小只有64个字节，一个block包含了很多个inode，所以同时创建一堆文件只会影响几个block的数据。类似的，如果我向一个文件写一堆数据，我需要申请大量的data block，我需要修改表示block空闲状态的bitmap block中的很多个bit位，如果我分配到的是相邻的data block，它们对应的bit会在同一个bitmap block中，所以我可能只是修改一个block的很多个bit位。所以一堆系统调用可能会反复更新一组相同的磁盘block。通过batching，多次更新同一组block会先快速的在内存的block cache中完成，之后在transaction结束时，一次性的写入磁盘的log中。这被称为write absorption，它可以极大的减少写磁盘的总时间（因为对同一个块的多次更改，我们只用往文件系统里写入它的最终状态就好了）。
- 最后就是disk scheduling。假设我们要向磁盘写1000个block，不论是在机械硬盘还是SSD（机械硬盘效果会更好），一次性的向磁盘的连续位置写入1000个block，要比分1000次每次写一个不同位置的磁盘block快得多。我们写log就是向磁盘的连续位置写block。通过向磁盘提交大批量的写操作，可以更加的高效。



并发：

第一种并发，是多个系统调用之间的并发。

ext3允许多个系统调用同时执行，所以我们可以有并行（没错就是并行，不是并发）执行的多个不同的系统调用。在ext3决定关闭并commit当前的transaction之前，系统调用不必等待其他的系统调用完成，它可以直接修改作为transaction一部分的block。许多个系统调用都可以并行的执行，并向当前transaction增加block。在XV6中，如果当前的transaction还没有完成，新的系统调用不能继续执行。而在ext3中，大多数时候多个系统调用都可以更改当前正在进行的transaction。

第二种并发，是多个事务之间的并发。

这种并发是，可以有多个不同状态的transaction同时存在。所以尽管只有一个open transaction可以接收系统调用，但是其他之前的transaction可以并行的写磁盘。这里可以并行存在的不同transaction状态包括了：

- open transaction
- 若干个正在commit到log的transaction，我们并不需要等待这些transaction结束。当之前的transaction还没有commit并还在写log的过程中，新的系统调用仍然可以在当前的open transaction中进行。
- 若干个正在从Memory中向文件系统block写数据的transaction
- 若干个正在被释放的transaction，也就是可以被清除的Log。

新的系统调用不必等待旧的transaction提交到log或者写入到文件系统。对比之下，XV6中新的系统调用就需要等待前一个transaction完全完成。



在ext3下，文件系统的系统调用流程：

1. 调用start函数。得到一个handle，它某种程度上唯一识别了当前系统调用。
2. 通过get获取block在buffer中的缓存，同时告诉handle这个block需要被读或者被写。如果你需要更改多个block，类似的操作可能会执行多次。
3. 当这个系统调用结束时，它会调用stop函数，并将handle作为参数传入。

stop函数并不会导致transaction的commit，它只是告诉logging系统，当前的transaction少了一个正在进行的系统调用。transaction只能在所有已经开始了的系统调用都执行了stop之后才能commit。



ext3 transaction commit步骤：

1. 当我们决定停止当前活跃的事务时，我们需要阻止新的系统调用的执行。当我们正在commit一个transaction时，我们不会想要有新增的系统调用，我们只会想要包含已经开始了的系统调用，所以我们需要阻止新的系统调用。当然，这会在短时间内损失性能
2. 等待包含在transaction中的已经开始了的系统调用们结束。
3. 一旦transaction中的所有系统调用都完成了，也就是完成了更新Memory中的数据，那么就可以开始一个新的transaction，并且让在第一步中等待的系统调用继续执行。开始一个新的事务。
4. 在Memory中更新相应的descriptor，data和commit block
5. 把descriptor block，data block写入硬盘的Log分区。
6. 前两步中的写log结束后，写入commit block。
7. 只有写入commit block结束之后 ，当前transaction才到达了commit point。如果crash发生在写commit block之前，那么transaction中的写操作在crash并重启时会丢失。
8. 将Data Block写回文件系统
9. 第8步结束之后，我们才能复用这个transaction在Log中占用的空间。



这些步骤都是内核的线程在后台完成的。

复用，如下图：只有T1先结束，T2的空间才能被复用到。因为Log是一个连续的写入

**如果复用的话，也许需要修改Super Block的指向，比如T1被复用了，那么T2才是第一个有效事务**

<img src="https://i.imgur.com/ziIHmvl.png" style="zoom:80%;" />



至于崩溃之后的恢复，是比较简单的：

因为对于以Magic Number起始的Data Block，我们会将这32比特替换为0，并在恢复（和写入文件系统）的时候替换回去，所以在log中，除了descriptor和commit block，不会有其他的block以这32bit的魔法数字作为起始。

所以恢复软件会从super block指向的位置开始一直扫描，直到：

- 某个commit block之后的一个block并不是descriptor block
- 或者某个commit block之后是descriptor block，但是根据descriptor block找到的并不是一个commit block

这时，恢复软件会停止扫描，并认为最后一个有效的commit block是log的结束位置。对于部分完成的事务（也即没有Commit Block），会完全忽略，因为它不是一个原子性的事务，是一个部分完成的事务，没有包含这个事务全部的写入操作。

之后恢复软件会回到log的最开始位置，并将每个log block写入到文件系统的实际位置，直到走到最后一个有效的commit block。之后才是启动剩下的操作系统，并且运行普通的程序。在恢复完成之前，是不能运行任何程序的，因为这个时候文件系统并不是有效的。

简而言之就是三步走：

1. 从Log Super Block指向的起始位置开始，扫描一遍（可能会回环）， 找到最后一个有效的Transaction，以及全部的有效Transaction。
2. 从第一个事务，到最后一个，按顺序把他们恢复到文件系统里去
3. 启动剩下的OS，这时才可以开始运行普通的程序。因为第2步完成之前，FS是不一致的。

#### 提问：对比Xv6的Log

> 学生提问：XV6相比这里的log机制，缺少了什么呢？
>
> Robert教授：XV6主要缺失的是在log中包含多个transaction的能力，**在XV6的log中最多只会有一个transaction**，所以在XV6中缺少了并发的能力。比如说当我在执行transaction T7的系统调用时，ext3可以同时向磁盘提交T6，而**这在XV6中这是不可能的，因为log只保存了一个transaction**。所以我们必须先完成一个transaction的所有工作，之后才能开始下一个transaction。所以XV6是简单且正确的，但是缺少了并发的能力。
>
> 学生提问：但是在XV6我还是可以有多个transaction，只是说不能异步的执行它们，对吗？
>
> Robert教授：这里其实有点模糊，**XV6实际上允许在一个transaction中包含多个系统调用（注，详见15.8），所以XV6有一些并发和batching的能力，但是当XV6决定要commit一个transaction时，在完全完成这个transaction之前，是不能执行任何新的系统调用的。**因为直到前一个transaction完全完成，并没有log空间来存放新的系统调用。**所以XV6要么是在运行一些系统调用，要么是在commit transaction，但是它不能同时干这两件事情，而ext3可以同时干这两件事情。**

#### 提问：为什么新transaction需要等前一个transaction中系统调用执行完成？

这个我个人的理解是，因为每一个与文件系统有关的系统调用，原则上都要保证这个系统调用的原子性的。而如果我们把同一个系统调用所产生的效应，分裂到一新一旧两个事务中，那就使得两个事务中都包含了一个部分完成的系统调用，这是破坏了原子性的。

而且，新的事务要比旧的事务5秒钟结束，这就会导致同一个系统调用所产生的操作，它生效的时间会有延迟，这个时间差可能会导致问题。

#### 提问：虚假Commit Block

> 学生提问：当你在讨论crash的时候，你有一个图是T8正在使用之前释放的T5的空间，如果T8在crash的时候还没有commit，并且T5的commit block正好在T8的descriptor block所指定的位置，这样会不会不正确的表明T8已经被commit了（注，这时T8有一个假的commit block）？
>
> 简而言之，序列号对不上
>
> Robert教授：让我尝试画出这里的场景。首先我们有一个古老的transaction T5，因为log的循环特性，在顺序上T8位于T5之前。因为T5已经被释放了，T8正在蚕食T5的log空间。假设T8没有完成commit，但是如果完成commit的话，T8的commit block会写到T5的commit block位置。T8并没有能写入commit block，T8前面所有的block都写入了，但是最后跟的是T5的commit block。这里的答案是，descriptor block和commit block都有transaction的序列号，所以T8的descriptor block里面的序列号是8，但是T5的commit block里面的序列号是5，所以两者不能匹配。



> 学生提问：我们可以在transaction T8开始的时候就知道它的大小吗？
>
> 简而言之，事务在关闭之后可以知道多大，关闭之前没办法预言未来。
>
> Robert教授：这是个复杂的问题。当T8作为活跃的transaction开始时，系统调用会写入数据，这时文件系统并不知道T8有多大。当文件系统开始commit T8时，是知道T8有多大的，因为文件系统只会在T8中所有的系统调用都结束之后才commit它，而在那个时间点，文件系统知道所有的写操作，所以就知道T8究竟有多大。除此之外，descriptor block里面包含了所有block的实际编号，所以当写入transaction的第一个block，也就是descriptor block时，logging系统知道T8会包含多少个block。

#### 提问：ext3的三种模式

> 学生提问：log中的data block是怎么写入到文件系统中的？
>
> Robert教授：这个问题有多个答案。对于data block，ext3有三种模式，但是我只记得两个，journaled data和ordered data（注，第三种是writeback）。当你在配置ext3文件系统时，你需要告诉Linux你想要哪种模式。
>
> journaled data：文件内容就是写入到log中，如果你向一个文件写数据，这会导致inode更新，log中会包含文件数据和更新了的inode，也就是说任何更新了的block都会记录在log中。这种方法非常慢，因为数据需要先写在log中，再写到文件系统中。所以journaled data很直观，但是很慢。
>
> ordered data是最流行的模式，它不会将文件数据写入到log中，只会将metadata block，例如inode，目录block，写入到log中，**文件的内容会直接写到文件系统的实际位置中**。所以这种模式要快得多，因为你不用将文件内容写两次。但是它也会导致更多的复杂性，因为你不能随时写入文件内容。假设你执行一个写操作导致一个新的block被分配给一个文件，并将包含了新分配block编号的inode写入到log中并commit，在实际写入文件内容至刚刚分配的data block之前发生crash。在稍后的恢复流程中，你将会看到包含了新分配的block编号的inode，但是对应data block里面的内容却属于之前使用了这个data block的旧的文件。如果你运行的是一个类似Athena的多用户系统，那么可能就是一个用户拥有一个文件，其中的内容又属于另一个用户已经删除的文件，如果我们不是非常小心的处理写入数据和inode的顺序就会有这样的问题。
>
> ext3的ordered data通过先写入文件内容到磁盘中，再commit修改了的inode来解决这里的问题。如果你是个应用程序，你写了一个文件并导致一个新的文件系统data block被分配出来，文件系统会将新的文件内容写到新分配的data block中，之后才会commit transaction，进而导致inode更新并包含新分配的data block编号。如果在写文件数据和更新inode之间发生了crash，你也看不到其他人的旧的数据，因为这时就算有了更新了的data block，但是也没关系，因为现在不仅inode没有更新，连bitmap block也没更新，相应的data block还处于空闲状态，并且可以分配给其他的程序，你并不会因此丢失block。这里的效果就是我们写了一个data block但是最终并没有被任何文件所使用。

WriteBack：

> **写回（write-back）模式：** 在写回模式下，文件系统将数据写入文件所在的块，但不会立即更新对应的inode信息。这意味着文件数据可以更快地写入，因为不需要等待inode的更新，但在系统崩溃时可能会导致部分数据的丢失。

journal： Data Block也写入Log，和Xv6一样。

Ordered：文件内容直接写回文件系统，然后再将元数据Commit到Log。

### Virtual Memory Primitives for User Programs

