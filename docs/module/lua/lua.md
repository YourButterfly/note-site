# lua

## syntax

require("<模块名>")

```lua
require("<模块名>")
-- 用来加载模块
local v = require("<模块名>")
-- 可以给加载的模块定义一个别名变量

-- Lua启动时，会以环境变量 LUA_PATH 的值来初始全局变量 package.path，用于require搜索 Lua 文件

-- 加载C
local path = "/usr/local/lua/lib/libluasocket.so"
-- 或者 path = "C:\\windows\\luasocket.dll"，这是 Window 平台下
local f = assert(loadlib(path, "luaopen_socket"))
f()  -- 真正打开库
```

## cgilua

### installing cgilua

```shell
luarocks install cgilua
# Lua包管理工具Luarocks
```

### api

#### Server API

Server API (SAPI)允许抽象一系列内部web服务器细节，并允许在WSAPI上使用CGILua。

Kepler是WSAPI的参考实现，目前支持Apache、Microsoft IIS和Xavante作为Web服务器，支持CGI、FastCGI作为WSAPI连接器。

SAPI.Request

```lua
SAPI.Request.getpostdata ([n])
    获取一块post数据. 可选参数n指定n字节的读入数据 (无参数则使用默认块大小).
    以lua String的形式返回这块post数据

SAPI.Request.servervariable (varname)
    获取服务器环境变量的值. 参数varname可以是定义的CGI变量之一，尽管不是所有服务器都实现了全部的变量集。这集合包括:
    AUTH_TYPE - I如果服务器支持用户身份验证，并且脚本受到保护, 这就是用于验证用户的特定协议验证方法.
    CONTENT_LENGTH - 客户端提供的content自身的长度.
    CONTENT_TYPE - 对于附加信息的查询，如HTTP POST和PUT，这是数据的content类型。
    GATEWAY_INTERFACE - 此服务器遵守的CGI规范的修订. 格式: CGI/revision
    PATH_INFO - client提供的额外路径信息. 换句话说，脚本可以通过它们的虚拟路径名访问，在路径的末尾加上额外的信息。 额外信息作为PATH_INFO发送. 如果信息来自URL，它将在传递给CGI脚本时，被服务器解码
    PATH_TRANSLATED - 服务器提供了PATH_INFO的翻译版本, 包含virtual-to-physical的映射
    QUERY_STRING - 引用此脚本的URL中的“?”后面的信息. 它不应该被解码，不管什么方式. 当有查询信息时，无论命令行解码与否，都应该设置此变量。
    REMOTE_ADDR - 发起请求的远程主机地址.
    REMOTE_HOST - The hostname making the request. If the server does not have this information, it should set REMOTE_ADDR and leave this unset.
    REMOTE_IDENT - If the HTTP server supports RFC 931 identification, then this variable will be set to the remote user name retrieved from the server. Usage of this variable should be limited to logging only.
    REMOTE_USER - If the server supports user authentication, and the script is protected, this is the username they have authenticated as.
    REQUEST_METHOD - 请求方法. For HTTP, this is "GET", "HEAD", "POST", etc.
    SCRIPT_NAME - A virtual path to the script being executed, used for self-referencing URLs.
    SERVER_NAME - The server's hostname, DNS alias, or IP address as it would appear in self-referencing URLs.
    SERVER_PORT - The port number to which the request was sent.
    SERVER_PROTOCOL - The name and revision of the information protcol this request came in with. Format: protocol/revision
    SERVER_SOFTWARE - The name and version of the web server software answering the request (and running the gateway). Format: name/version
```

SAPI.Response

```lua
SAPI.Response.contenttype (header)
    Sends the Content-type header to the client. The given header should be in the form "type/subtype". 在使用SAPI.Response.write发送任何输出之前，必须调用这个函数。
    Returns nothing.
SAPI.Response.errorlog (message)
    Generates error output using the given string or number.
    Returns nothing.
SAPI.Response.header (header, value)
    Sends a generic header to the client. The first argument must be the header name, such as "Set-Cookie". The second argument should be its value. 这个函数不应该用来代替 SAPI.Response.contenttype 或者 the SAPI.Response.redirect.
    Returns nothing.
SAPI.Response.redirect (url)
    Sends the Location header to the client. The given url should be a string.
    Returns nothing.
SAPI.Response.write (...)
    Generates output using the given arguments. The arguments must be strings or numbers.
    Returns nothing.
```

## luadec

### installing luadec

```shell
git clone https://github.com/viruscamp/luadec
cd luadec
git submodule update --init lua-5.1
cd lua-5.1
make linux
cd ../luadec
make LUAVER=5.1
```

### luadec usage

```shell
# decompile lua binary file:  
luadec abc.luac  
#decompile lua source file for testing and comparing:  
luadec abc.lua  
# disassemble lua source or binary  
luadec -dis abc.lua  
# -pn print nested functions structure, could be used by -fn  
luadec -pn test.lua
# -f decompile only specific nested function  
luadec -f 0_1 test.lua  
# -ns donot process sub functions  
luadec -ns -f 0_1 test.lua  
# -fc perform a instruction-by-instruction compare for each function  
luadec -fc test.lua  
```

### issue 1

```shell
$ ./luadec/luadec -pn  workspace/lua.luac
./luadec/luadec: workspace/lua.luac: bad header in precompiled chunk
```

solution 1

```shell
# http://sourceforge.net/projects/unluac/?source=directory
java -jar unluac_2015_06_13.jar workspace/lua.luac > myfile_decompiled.lua
```
