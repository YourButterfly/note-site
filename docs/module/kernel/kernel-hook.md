# kernel hook

## makefile obj-

```txt
obj-$(CONFIG_FOO) += foo.o $(CONFIG_FOO)可以为y(编译进内核) 或m(编译成模块)。如果CONFIG_FOO不是y 和m,那么该文件就不会被编译联接了
```
