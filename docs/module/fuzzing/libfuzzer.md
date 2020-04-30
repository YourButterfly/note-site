# libfuzzer

## libfuzzer for gcc

[Add Better Support for AFL/gcc](https://github.com/google/clusterfuzz/issues/920)

### 问题

```cpp
Your application is linked against incompatible ASan runtimes.
```

可能是link导致的,按照下面的说法，静态编译版本的项目同时link了libasan.so也会报错

```c
// ref https://lists.gnu.org/archive/html/libtool/2017-01/msg00008.html
Hello list,

I was getting the following error message after building with
CFLAGS='-fsanitize=address -static-libasan' LDFLAGS=-static-libasan
and trying to run the generated executable.

==5331==Your application is linked against incompatible ASan runtimes.


Please note that the project also contains a shared library, which libtool was always linking while ignoring the "-static-libasan" flag. So the library was linking to "libasan.so" while the binaries were linked with the static runtime.

The attached patch on the generated libtool script solved the issue to me. Feel free to use it upstream, if it makes sense.

Thanks,
Dimitris
```

确定了，确实是这个问题

```cmake
target_compile_options(you_target PUBLIC -static-libasan)
```
