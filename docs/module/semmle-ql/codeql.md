# CodeQL

https://semmle.com/codeql

## How does CodeQL work

### Create database

1. 提取代码库中每个源文件的单一关系表示
    - 对于编译型语言，通过监视构建过程提取。每次调用编译器来处理源文件时，都会生成该文件的副本，并收集关于源代码的所有相关信息。
    - 对于解释型语言，提取器直接在源代码上运行，解析依赖以提供准确的代码库表示。
    - 对于多语言代码库，数据库每次生成一种语言。
2. 提取之后，分析所需的所有数据被导入到单个目录中，即CodeQL数据库。

### Run queries

用专门设计的面向对象查询语言--QL,对数据库执行一个或多个查询。

### Interpret query results

将查询结果以更可解释的方式展示。

## CodeQL for Visual Studio Code

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
    - Make sure you include the submodules, either by using git clone --recursive, or using by git submodule update --init --remote after cloning.
    - Use git submodule update --remote regularly to keep the submodules up to date.
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

## CodeQL CLI

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