# CodeQL

https://semmle.com/codeql

## How does CodeQL work

### 一 Create database

1. 提取代码库中每个源文件的单一关系表示
    - 对于编译型语言，通过监视构建过程提取。每次调用编译器来处理源文件时，都会生成该文件的副本，并收集关于源代码的所有相关信息。
    - 对于解释型语言，提取器直接在源代码上运行，解析依赖以提供准确的代码库表示。
    - 对于多语言代码库，数据库每次生成一种语言。
2. 提取之后，分析所需的所有数据被导入到单个目录中，即CodeQL数据库。

### Run queries

用专门设计的面向对象查询语言--QL,对数据库执行一个或多个查询。

### Interpret query results

将查询结果以更可解释的方式展示。

## 二 CodeQL for Visual Studio Code

### Installing the extension

3种方式

1. Go to the Visual Studio Code Marketplace in your browser and click Install.
2. In the Extensions view (Ctrl+Shift+X or Cmd+Shift+X), search for CodeQL, then select Install.
3. Download the CodeQL VSIX file. Then, in the Extensions view, click More actions > Install from VSIX, and select the CodeQL VSIX file.

### Configuring access to the CodeQL CLI

CodeQL使用CodeQL CLI来编译和运行查询，细节另外介绍。CodeQL CLI [下载](https://github.com/github/codeql-cli-binaries/releases/latest/download/codeql.zip)

CodeQL CLI 在Linux/Mac叫做 `codeql`，在Windows中叫做 `codeql.cmd`

需要在vscode CodeQL Extension配置其路径（codeQL.cli.executablePath）

### Setting up a CodeQL workspace

使用CodeQL时，需要访问标准CodeQL库。不然无法查询。两种方式：

1. use the “starter” workspace(建议)
2. add the CodeQL libraries and queries to an existing workspace.

#### Using the “starter” workspace

1. Clone the https://github.com/github/vscode-codeql-starter/ repository to your computer:
    - Make sure you include the submodules, either by using `git clone --recursive`, or using by `git submodule update --init --remote` after cloning.
    - Use `git submodule update --remote` regularly to keep the submodules up to date.
2. In VS Code, use the File > Open Workspace option to open the vscode-codeql-starter.code-workspace file from your checkout of the workspace repository.

#### Updating an existing workspace for CodeQL

Dowload [the CodeQL libraries](https://github.com/semmle/ql)

1. Select **File** > **Add Folder** to Workspace, and choose your local checkout of the `Semmle/ql` repository.
2. Create one new folder per target language, using either the **New Folder** or **Add Folder** to Workspace options, to hold custom queries and libraries.
3. Create a `qlpack.yml` file in each target language folder. This tells the CodeQL CLI the target language for that folder and what its dependencies are. (The master branch of Semmle/ql already has these files.) CodeQL will look for the dependencies in all the open workspace folders, or on the user’s search path.

### Choosing a database

关于如何创建测试目标的database，看 CodeQL Cli

打开vscode，选择CodeQL扩展，可以看到 **DATABASES** 和 **QUERY HISTORY**，并且DATABASES有一个 **+** 符号，点击并选择database
 

### Run a query

vscode 打开 `*.ql` 右键 **CodeQL: Run Query**

## 三 CodeQL CLI

前一节的vscode的大部分工作最后都是调用CodeQL CLI完成的

### Setting up the CodeQL CLI

```shell
# Download the CodeQL CLI zip package
wget https://github.com/github/codeql-cli-binaries/releases/latest/download/codeql.zip
# Create a new CodeQL directory, For example
mkdir $HOME/codeql-home
# Obtain a local copy of the CodeQL queries, For example
git clone https://github.com/Semmle/ql  $HOME/codeql-home/codeql-repo
git clone https://github.com/github/codeql-go/ $HOME/codeql-home/codeql-go-repo
# Extract the zip archive
unzip codeql.zip
mv codeql $HOME/codeql-home/codeql-cli
# validate
$HOME/codeql-home/codeql-cli/codeql resolve languages
# outputs
## cpp (/home/pwd/codeql-home/codeql-cli/cpp)
## go (/home/pwd/codeql-home/codeql-cli/go)
## ...
## python (/home/pwd/codeql-home/codeql-cli/python)
$HOME/codeql-home/codeql-cli/codeql resolve qlpacks
# outputs
## codeql-cpp (/home/pwd/codeql-home/codeql-repo/cpp/ql/src)
## codeql-cpp-tests (/home/pwd/codeql-home/codeql-repo/cpp/ql/test)
## ...
## legacy-upgrades (/home/pwd/codeql-home/codeql-cli/legacy-upgrades)
```

### Create a database

```shell
codeql database create <database> --language=<language-identifier>
```

必须指定

- `<database>`: 新建数据库的路径，不能是一个已有的文件夹
- `--language`: 为其创建数据库的语言的标识符, `codeql resolve languages` 查看支持的语言

可选

- `--source-root`: 数据库创建中使用的主要源文件的根文件夹。默认情况下，该命令假设当前目录是源根目录
- `--command`: 用于编译型语言，调用编译器的构建命令，比如`--command='make all'`。如果未指定该命令，CodeQL将尝试使用内置的autobuilder自动检测构建系统。

### Running codeql database analyze


```shell
codeql database analyze <database> <queries> --format=<format> --output=<output>
```

必须指定

- `<database>`: database的路径
- `<queries>`: 运行 ql文件，如果是目录会递归搜索执行
- `--format`: 生成结果的文件格式. 支持 csv, sarif-latest, sarifv1, sarifv2, sarifv2.1.0, graphtext, dgml.
- `--output`: 生成结果到目录

### Upgrade database

```shell
codeql database upgrade <database>
```

### Others

见 https://help.semmle.com/codeql/codeql-cli/commands.html


## 四 CodeQL for C/C++

### Basic C/C++ query example

查找冗余的if语句

```c
if (error) { }
```

查询语句

```sql
import cpp

from IfStmt ifstmt, Block block
where ifstmt.getThen() = block
  and block.getNumStmt() = 0
select ifstmt, "This 'if' statement is redundant."
```


`import cpp` 导入C/C++标准查询库

`from IfStmt ifstmt, Block block `	定义变量，语法是 `<type> <variable name>`. 这里定义了 **IfStmt** 类型`ifstmt`， **Block** 类型`block`

`where ifstmt.getThen() = block and block.getNumStmt() = 0`	条件. `ifstmt.getThen() = block` block必须是if语句的分支并且这个block中statements数量为0

`select ifstmt, "This 'if' statement is redundant."` 定义每个匹配报告的内容, 格式 `select <program element>, "<alert message>"`

### the CodeQL libraries for C/C++ 

关于C/C++标准查询库

https://help.semmle.com/QL/learn-ql/cpp/introduce-libraries-cpp.html

### Tutorial: Function classes

#### 寻找静态函数

```sql
import cpp

from Function f
where f.isStatic()
select f, "This is a static function."
```

#### 寻找未被调用的函数

```sql
from Function f
where not exists(FunctionCall fc | fc.getTarget() = f)
select f, "This function is never called."
```

#### 排除使用函数指针引用的函数

```sql
import cpp

from Function f
where not exists(FunctionCall fc | fc.getTarget() = f)
  and not exists(FunctionAccess fa | fa.getTarget() = f)
select f, "This function is never called, or referenced with a function pointer."
```

#### 寻找特定函数

```sql
import cpp

from FunctionCall fc
where fc.getTarget().getQualifiedName() = "sprintf"
  and not fc.getArgument(1) instanceof StringLiteral
select fc, "sprintf called with variable format string."
```

### Tutorial: Expressions, types and statements 

Stmt - C/C++ statements

- Loop
    - WhileStmt
    - ForStmt
    - DoStmt
- ConditionalStmt
    - IfStmt
    - SwitchStmt
- TryStmt
- ExprStmt - expressions used as a statement; for example, an assignment
- Block - { } blocks containing more statements

#### 查找赋值为0的表达式

```sql
import cpp

from AssignExpr e
where e.getRValue().getValue().toInt() = 0
select e, "Assigning the value 0 to something."
```

#### 查找赋值0给IntegralType变量的表达式

注意 `Type.getUnspecifiedType()`. 它会把 `typedef` 展开成底层类型

```sql
import cpp

from AssignExpr e
where e.getRValue().getValue().toInt() = 0
  and e.getLValue().getType().getUnspecifiedType() instanceof IntegralType
select e, "Assigning the value 0 to an integer."
```

#### 在'for'循环初始化中查找赋值为0

由于`for`的初始化通常是一个`Stmt`而不是`Expr`, 使用`getEnclosingStmt` 获取Expr最接近的Smt（类 AssignExpr 被类 ExprStmt 封装）

```sql
import cpp

from AssignExpr e, ForStmt f
// the assignment is in the 'for' loop initialization statement
where e.getEnclosingStmt() = f.getInitialization()
  and e.getRValue().getValue().toInt() = 0
  and e.getLValue().getType().getUnspecifiedType() instanceof IntegralType
select e, "Assigning the value 0 to an integer, inside a for loop initialization."
```

#### 在'for'循环的循环体中查找赋值为0

`*`表示0次或多次，即这个 Stmt 本身，它的 父Stmt，父Stmt的父Stmt 等等

```sql
import cpp

from AssignExpr e, ForStmt f
// the assignment is in the for loop body
where e.getEnclosingStmt().getParentStmt*() = f.getStmt()
  and e.getRValue().getValue().toInt() = 0
  and e.getLValue().getType().getUnderlyingType() instanceof IntegralType
select e, "Assigning the value 0 to an integer, inside a for loop body."
```

### Tutorial: Conversions and classes

Conversions, 所有转换都会更改表达式的类型。它们可以是隐式转换(由编译器生成)，也可以是显式转换(由用户请求)。

- Expr
    - Conversion
    - Cast
        - CStyleCast
        - StaticCast
        - ConstCastReinterpretCast
        - DynamicCast
    - ArrayToPointerConversion
    - VirtualMemberToFunctionPointerConversion


Classes

- Type
    - UserType—includes classes, typedefs, and enums
        - Class—a class or struct
            - Struct—a struct, which is treated as a subtype of Class
            - TemplateClass—a C++ class template

## Tutorial: Analyzing data flow in C/C++ 

### Local data flow

单个函数内部的数据流。在 `DataFlow` 模块中定义了类 `Node`, 表示数据可以经过的任何元素。Node 分 `ExprNode` 和 `ParameterNode`。

predicates `asExpr` and `asParameter`

```cpp
class Node {
  /** Gets the expression corresponding to this node, if any. */
  Expr asExpr() { ... }

  /** Gets the parameter corresponding to this node, if any. */
  Parameter asParameter() { ... }

  ...
}

```

或者使用 predicates `exprNode` 和 `parameterNode`

```cpp
/**
 * Gets the node corresponding to expression `e`.
 */
ExprNode exprNode(Expr e) { ... }

/**
 * Gets the node corresponding to the value of parameter `p` at function entry.
 */
ParameterNode parameterNode(Parameter p) { ... }
```

`localFlowStep(Node nodeFrom, Node nodeTo)`, 可以使用'*'和'+'递归，`localFlow` 相当于 `localFlowStep*`。比如

```cpp
DataFlow::localFlow(DataFlow::parameterNode(source), DataFlow::exprNode(sink))
```

### Using local taint tracking

局部污点跟踪通过包含非保存值的流步骤扩展了局部数据流。例如下面，这种情况下malloc的参数被会被污染。

```cpp
int i = tainted_user_input();
some_big_struct *array = malloc(i * sizeof(some_big_struct));
```

本地污点跟踪在模块 `TaintTracking` 中, `localTaintStep(DataFlow::Node nodeFrom, DataFlow::Node nodeTo)` 意味着（?）从节点`nodeFrom`到节点`nodeTo`有一个立即污染传递边，`localTaint` 想当于 `localTaintStep*`

```
TaintTracking::localTaint(DataFlow::parameterNode(source), DataFlow::exprNode(sink))
```

### Using global data flow

全局数据流跟踪整个程序中的数据流，因此比本地数据流更强大。但是，全局数据流不如本地数据流精确，而且执行分析通常需要大量的时间和内存。

使用全局数据流库扩展类 `DataFlow::Configuration`，如下所示

```cpp
import semmle.code.cpp.dataflow.DataFlow

class MyDataFlowConfiguration extends DataFlow::Configuration {
  MyDataFlowConfiguration() { this = "MyDataFlowConfiguration" }

  override predicate isSource(DataFlow::Node source) {
    ...
  }

  override predicate isSink(DataFlow::Node sink) {
    ...
  }
}
```

定义以下配置

- `isSource` 定义数据可能流自何处
- `isSink` 定义数据可能流向某处
- `isBarrier` 可选，限制数据流
- `isBarrierGuard` 可选，限制数据流
- `isAdditionalFlowStep` 可选，添加额外的流步骤

使用谓词`hasFlow(DataFlow::Node source, DataFlow::Node sink)`进行数据流分析:

```sql
from MyDataFlowConfiguration dataflow, DataFlow::Node source, DataFlow::Node sink
where dataflow.hasFlow(source, sink)
select source, "Data flow to $@.", sink, sink.toString()
```

### Using global taint tracking

全局污点跟踪使用附加的非保存值的步骤扩展了全局数据流。通过扩展`TaintTracking::Configuration`类来使用全局污染跟踪库，具体如下:

```cpp
import semmle.code.cpp.dataflow.TaintTracking

class MyTaintTrackingConfiguration extends TaintTracking::Configuration {
  MyTaintTrackingConfiguration() { this = "MyTaintTrackingConfiguration" }

  override predicate isSource(DataFlow::Node source) {
    ...
  }

  override predicate isSink(DataFlow::Node sink) {
    ...
  }
}
```

### Examples

查找所有传入 fopen 参数1 的表达式:

```sql
import cpp
import semmle.code.cpp.dataflow.DataFlow

from Function fopen, FunctionCall fc, Expr src
where fopen.hasQualifiedName("fopen")
  and fc.getTarget() = fopen
  and DataFlow::localFlow(DataFlow::exprNode(src), DataFlow::exprNode(fc.getArgument(0)))
select src
```

查找传入 fopen 参数1 的 publi parameter (这个函数接收的参数，比如`main(int argc, char **argv, char **envp)`中的argc， argv, envp):

```sql
import cpp
import semmle.code.cpp.dataflow.DataFlow

from Function fopen, FunctionCall fc, Parameter p
where fopen.hasQualifiedName("fopen")
  and fc.getTarget() = fopen
  and DataFlow::localFlow(DataFlow::parameterNode(p), DataFlow::exprNode(fc.getArgument(0)))
select p
```

查找对格式字符串没有硬编码的格式化函数的调用。

```sql
import semmle.code.cpp.dataflow.DataFlow
import semmle.code.cpp.commons.Printf

from FormattingFunction format, FunctionCall call, Expr formatString
where call.getTarget() = format
  and call.getArgument(format.getFormatParameterIndex()) = formatString
  and not exists(DataFlow::Node source, DataFlow::Node sink |
    DataFlow::localFlow(source, sink) and
    source.asExpr() instanceof StringLiteral and
    sink.asExpr() = formatString
  )
select call, "Argument to " + format.getQualifiedName() + " isn't hard-coded."
```


```sql
import semmle.code.cpp.dataflow.DataFlow

class EnvironmentToFileConfiguration extends DataFlow::Configuration {
  EnvironmentToFileConfiguration() { this = "EnvironmentToFileConfiguration" }

  override predicate isSource(DataFlow::Node source) {
    exists (Function getenv |
      source.asExpr().(FunctionCall).getTarget() = getenv and
      getenv.hasQualifiedName("getenv")
    )
  }

  override predicate isSink(DataFlow::Node sink) {
    exists (FunctionCall fc |
      sink.asExpr() = fc.getArgument(0) and
      fc.getTarget().hasQualifiedName("fopen")
    )
  }
}

from Expr getenv, Expr fopen, EnvironmentToFileConfiguration config
where config.hasFlow(DataFlow::exprNode(getenv), DataFlow::exprNode(fopen))
select fopen, "This 'fopen' uses data from $@.",
  getenv, "call to 'getenv'"
```