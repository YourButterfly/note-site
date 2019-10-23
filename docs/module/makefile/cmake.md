# cmake

cmake 语法

## install 指令（主要是生成Makefile中的install target）

```cmake
install(FILES flie DESTINATION dir_path) #执行make install时，把file拷贝到dir_path
install(PROGRAMS file DESTINATION dir_path) #执行make install时，把file拷贝到dir_path,并给予file可执行权限
INSTALL(TARGETS  ylib ylib_s
    #RUNTIME DESTINATION xxx
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)# 安装libylib.so到lib目录，安装libylib_s.a到lib目录，RUNTIME 是安装可执行文件到xxx目录，注意这个指令有个坑，我后面会说明这个
```

## configure_file

```cmake
configure_file(fileA fileB @ONLY)
#把fileA 复制并重命名为fileB,此时，fileA中的@var@的值会被替换为cmakelists.txt 中var的值。@ONLY是只转换@va@这种变量
```
