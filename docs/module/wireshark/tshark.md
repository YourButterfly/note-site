# tshark

tshark - 转储，分析网络流量

<https://www.wireshark.org/docs/man-pages/tshark.html>

```shell
tshark [ -i <capture interface>|- ] [ -f <capture filter> ] [ -2 ] [ -r <infile> ] [ -w <outfile>|- ] [ options ] [ <filter> ]

tshark -G [ <report type> ] [ --elastic-mapping-filter <protocols> ]
```

## 描述

Tshrak 是一个网络协议分析工具，可以从实时网络捕获数据包数据，也可以从先前保存的捕获文件中读取数据包，进而将这些包的解码形式输出到标准输出，或者将这些包写入文件。

捕获文件的默认格式是pcapng


