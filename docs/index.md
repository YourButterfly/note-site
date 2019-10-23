# 随笔和笔记

--to神莱

## 环境

```txt
os:         linux 64位
tools:      pip3 mkdocs mkdocs-material git
language:   python3
```

## go

### python pip3 安装

略

### mkdocs mkdocs-material安装

```shell
sudo pip3 install mkdocs mkdocs-material
```

mkdocs-material 是主题

### mkdocs 搭建

git clone

```shell
git clone https://github.com/YourButterfly/note-site.git
```

解释一下项目目录

```txt
.
├── docs //md文档，用来写的
│   ├── index.md
│   └── pdf
│       ├── introduce.md
│       └── page-boxes.md
├── LICENSE //不用管
└── mkdocs.yml // mkdoc的配置文件，大部分已经弄好了

2 directories, 5 files

```

介绍一下mkdocs命令

```shell
mkdocs -h

Commands:
  build      Build the MkDocs documentation，这里不需要
  gh-deploy  Deploy your documentation to GitHub Pages，推送到github，已经配置好了，直接用‘mkdocs gh-deploy’
  new        Create a new MkDocs project，这里不需要
  serve      Run the builtin development server，预览页面
```

### 编写文档

1. 写之前pull一下，在项目主目录（note-site）里`git pull`，防止冲突
2. 写
    - 在docs下创建目录，比如pdf，作为一个类别
    - 在docs/pdf下创建你的文档，md格式，也可以继续创建目录，作为你的tag
3. 配置
    - 配置mkdocs.yml，配置见下
4. 推送
    - 保存一下项目git add，git commit ，git push
    - 推送到gh-deploy分支上，直接`mkdocs gh-deploy`

配置mkdocs.yml

照着加就行

```md

nav:
        - 首页: index.md
        - 文件格式:
                - PDF:
                        - 简介: pdf/introduce.md
                        - PageBox: pdf/page-boxes.md
```

预览

```shell
mkdocs serve
```

推送

```shell
$ mkdocs gh-deploy
INFO    -  Cleaning site directory
INFO    -  Building documentation to directory: /home/pwd/myMkDocs/first-project/site
INFO    -  Copying '/home/pwd/myMkDocs/first-project/site' to 'gh-pages' branch and pushing to GitHub.
Username for 'https://github.com': yourbutterfly
Password for 'https://yourbutterfly@github.com':
INFO    -  Your documentation should shortly be available at: https://YourButterfly.github.io/note-site/
```
