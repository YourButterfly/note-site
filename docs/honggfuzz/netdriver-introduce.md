# Honggfuzz NetDriver

如果我们要实现一个相当健壮的TCP服务的fuzzing，为了实现这个目标需要做到哪些

1. 能与现有TCP服务集成的一小段代码，比如说，一个能够被Apache的HTTP链接的静态或动态库，（这里可以使httpd这个二进制程序）
2. 将来自内存数组和长度标识的输入转成TCP流的技术
3. 当输入缓冲区没有任何数据时，有方法去通知TCP 服务端结束连接；否则TCP服务端将一直挂起，或者超时中断连接，或者超过fuzzer设定的timeout
4. 如果许多相同的TCP服务器同时在相同的系统上运行，配置相同(或者默认配置)，那么它们也会绑定到数字上相同的TCP端口。当然，只有当我们打算同时Fuzz同一个TCP服务器的多个实例时，这才重要(使用honggfuzz可以轻松做到什么)

最近在Honggfuzz的代码库中引入的Honggfuzz网络驱动程序就试图实现这些。

静态库 [libfhnetdriver/libhfnetdriver.a] [源代码](https://github.com/google/honggfuzz/blob/master/libhfnetdriver/netdriver.c)

驱动程序将自己作为目标(提供符号)插入到最初由libFuzzer作者实现的接口中:

```c
int LLVMFuzzerTestOneInput(const uint8_t* buf, size_t len);
```

这种选择使得不仅可以在honggfuzz中使用该驱动程序，还可以在AFL和libFuzzer中使用该驱动程序。从现在开始，Apache HTTPD项目需要做的唯一修改可以归结为以下差异:

```patch
--- a/server/main.c
+++ b/server/main.c
@@ -484,8 +484,11 @@ static void usage(process_rec *process)
destroy_and_exit_process(process, 1);
}

-int main(int argc, const char * const argv[])
-{
+#ifdef HFND_FUZZING_ENTRY_FUNCTION
+ HFND_FUZZING_ENTRY_FUNCTION(int argc, const char *const *argv) {
+#else
+ int main(int argc, const char *const *argv) {
+#endif
char c;
int showcompile = 0, showdirectives = 0;
const char *confname = SERVER_CONFIG_FILE;
```

应用这个补丁之后，**main()**(属于Apache HTTPD 中server/main.c)会被编译器hfuzz-cc中的宏替代（'-D'定义宏），这个扩展宏对HonggFuzz NetDriver有特殊意义--识别为Apache HTTPD服务器代码的原始入口点的位置

在将TCP server 链接静态库libhfnetdriver.a(这一步由hfuzz-cc/*编译器执行)之后，fuzz引擎会运行自己的main函数，然后在一个单独的线程中运行TCP服务端代码

之前提到的TCP server 接受到输入结束的信号问题（比如说在fuzzing输入缓冲中不再有数据时，如何通知到TCP server），可以通过向其建立的tcp流中发送TCP FIN 包。在现代操作系统中，shutdown(sock, SHUT_WR)这个syscall完成了这个工作

我们要解决的最后一个要求是，能够使用相同的TCP端口启动多个TCP服务器。很遗憾，setsockopt(SO_REUSEPORT)不能用在这，因为混乱的输入可以分发到随机的fuzz进程的实例，不过这里有一个很好的方法：

    Linux network namespaces，正如它的名字所说，只能用在linux操作系统中。在不同的网络名称空间中运行每个新的TCP服务器允许它绑定到任何它希望的TCP端口，因为每个服务器都将看到属于自己的全新loopback接口。NetDriver 利用Linux namespace的代码可以在HonggFuzz的libhfcommin库中发现。

一旦TCP server 启动并接受新的TCP连接，net dirver 将：

    在你选择的Fuzzer的准备阶段调用LLVMFuzzerTestOneInput()接口；
    连接TCP server，
    向建立的TCP连接发送（send()）输入,
    使用shutdown(sock, SHUT_WR) 通知TCP server 没有更多的输入数据，
    等待TCP server向我们发送数据，直到它关闭TCP连接的一端为止。
    关闭客户端的TCP连接点，
    重复上述过程

下面的命令集将帮助您开始使用NetDriver来Fuzz您的第一个项目(至少对于Apache HTTPD来说)。

```shell
$ ( cd httpd && ./hfuzz.compile_and_install.asan.sh )
$ honggfuzz -v -Q -f IN/ -w ./httpd.wordlist -- ./httpd/httpd -X -f /home/jagger/fuzz/apache/dist/conf/httpd.conf.h2
...
...
Honggfuzz Net Driver (pid=21726): Waiting for the TCP server process to start accepting TCP connections at 127.0.0.1:8080. Sleeping for 1 second....
Honggfuzz Net Driver (pid=21726): The TCP server process is ready to accept connections at 127.0.0.1:8080. TCP fuzzing starts now!
Size:9378 (i,b,hw,edge,ip,cmp): 0/0/0/4643/110/56364, Tot:0/0/0/4643/110/56364
Size:40712 (i,b,hw,edge,ip,cmp): 0/0/0/1971/104/21274, Tot:0/0/0/6614/214/77638
...
```

在这个honggfuzz目录中可以找到自定义编译脚本，以及Apache HTTPD服务器所需的所有补丁、配置和初始语料库文件。
我认为将网络驱动程序与libFuzzer和AFL fuzzing设置集成起来应该是一项相对简单的任务。