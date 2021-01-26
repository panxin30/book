---
description: >-
  写笔记，汇整成书籍形式方便查阅或分享，不希望造成额外的费用或时间成本，每次编辑书籍内容都能自动更新。解决方案：GitBook CLI + GitHub
  Pages + GitHub Actions
---

# github及gitbook入门

## 一、Github

#### 创建个人主页

GitHub 为每一个用户分配了一个二级域名&lt;user-id&gt;.github.io，用户为自己的二级域名创建主页很容易，只要在托管空间下创建一个名为&lt;user-id&gt;.github.io的版本库，向其master分支提交网站静态页面即可，其中网站首页为index.html。  
访问网址：[https://panxin30.github.io/](https://panxin30.github.io/)

#### 创建项目主页

还可以为每个项目设置主页，项目主页也通过此二级域名进行访问。

访问地址：[https://panxin30.github.io/book/](https://panxin30.github.io/book/)

为项目启用项目主页很简单，只需要在项目版本库中创建一个名为gh-pages的分支，并向其中添加静态网页即可。也就是说如果项目的Git版本库中包含了名为gh-pages分支的话，则表明该项目提供静态网页构成的主页，可以通过网址http://&lt;user-id&gt;[.github.io/](http://.github.io/)&lt;project-name&gt;访问到。

## 二、gitbook

#### 1. 安装gitbook

#### 参考：[https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md](https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md)

#### 2. 如果被墙，切换为淘宝镜像再安装

*  切换为使用国内速度较快的淘宝镜像。

```text
npm config set registry=http://registry.npm.taobao.org -g
```

3. 开始写书

**方法一**、直接在gitbook上新建space，然后点击integrations，整合github其中一个仓库，编辑gitbook会自动同步到github对应仓库。担心是gitbook被墙，在国内访问较慢。数据不会丢，因为用的github仓库存储。github上可以直接编辑 [https://abu.gitbook.io/notes](https://abu.gitbook.io/notes)

**方法二**： 目标：在github新建一个仓库叫book，gitclone到本地，添加book.json，使用gitbook install book目录/ 安装插件，使用gitbook build book目录/构建静态资源，上传到panxin30.github.io或者book所在仓库的gh-pages分支（这就是github pages）。 缺点：不会自动更新，修改了书籍的源代码后需要再按照上面的操作来一次。前提是已经在本地安装了gitbook

**方法三**、使用github actions实现博客自动化部署 gitbook建立新书，关联到github仓库，给这个仓库添加actions，actions自动提交变更到gh-pages

以方法二为基础添加actions这一步

#### 1. 设定GitHub Access Token

为了让GitHub Actions 能自动帮我们发布GitBook 成果到GitHub Pages，必须授权GitHub 操作我们的Repository。作法就是设定Access Token。

这里都是在GitHub 网页上操作，按照以下步骤即可：

**2. 产生一个GitHub Personal Access Token：**

1. 点右上角帐号的头像-&gt;选择`Settings`-&gt;左边列表选择最底下的`Developer settings`-&gt;下个页面的左边列表选择`Personal access tokens`。
2. 点击`Generate new token`按钮。
3. 输入Token的描述，权限勾选`repo:status`和`public_repo`两个项目。
4. 点最下面的`Generate token`按钮。
5. 这时候页面上会显示一组Token，复制下来。**注意！产生的Token内容只会在此时显示一次，之后无法再查到，如果忘记Token就只能重新操作产生一次**。

**3. 到Repository 将刚刚的Token 设定成Secret：**

1. 到想要自动发布的Repository -&gt;选择`Settings`-&gt;左边列表选则`Secrets`-&gt;点`New secret`按钮。
2. 「Name」栏位填`GH_ACCESS_TOKEN`，「Value」栏位贴上刚刚复制的Token。
3. 点`Add secret`按钮，设定就完成了。

**4. GitHub Actions Workflow**

万事俱备，只剩下设定GitHub Actions，让它能自动帮我们制书和发布到GitHub Pages。

回到资料夹，新增一个`.github/workflows/build.yml`档案：

```text
$ mkdir -p .github/workflows
$ vi .github/workflows/build.yml
```

这里完全照抄即可，只需要把`USER_NAME`和`USER_EMAIL`设定更换成你的Git User Name和Email：

```text
name: Build my gitbook and deploy to gh-pages                                                                                            
  
on:
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  build:
    name: Build Gitbook
    runs-on: ubuntu-latest
    steps:
      # Check out the repo first
      - name: Checkout code
        uses: actions/checkout@v2
      # Run this action to publish gitbook
      - name: Publish
        uses: tuliren/publish-gitbook@v1.0.0
        with:
          # specify either github_token or personal_token
          github_token: 23e77fcdfda67b264737527285c2049f268c9f2a
          # personal_token: ${{ secrets.PERSONAL_TOKEN }}
```

上面是一个GitHub Actions的设定档，称为一个「workflow」。里面用到官方的[checkout action](https://github.com/marketplace/actions/checkout)，这几乎是每个workflow的起手式。使用[https://github.com/marketplace/actions/publish-gitbook](https://github.com/marketplace/actions/publish-gitbook) 选择的一个gitbook相关的别人做好的镜像,负责将markdown档制成GitBook静态网站，并自动将网站档案commit到`gh-pages`branch。

**5. 将Workflow 档推上GitHub，触发自动发布到GitHub Pages**

将刚刚新增的workflow 档进行commit 和push：

```text
$ git add .github/workflows/build.yml
$ git commit -m "add workflow file"
$ git push
```

回到GitHub Repository 页面，点「Actions」tab，会看到有一个workflow 任务被自动触发执行中：

![](../.gitbook/assets/image%20%281%29.png)

等到执行完毕变成绿勾勾，会看到自动建立了`gh-pages`branch并commit GitBook静态网站的档案：

测试有没有自动更新

## 三、gitbook 插件

gitbook插件可以解决一些网站不太方便的地方，如侧边栏导航不能收缩，自带搜索不支持中文等。

插件安装、使用方法：

> 1、 在`book.json`的plugins参数中添加插件名。  
> 2、终端使用`gitbook install`来安装插件，或使用NPM命令来单独安装：`npm install gitbook-plugin-插件名`。  
> 3、重新打包就能看见效果。

_**注意：**_  
1、插件一定先要在`book.json`文件里面plugins中才能生效，如果只是安装了插件，而没配置的话是不会生效的。  
2、gitbook命令安装慢，而且是全部插件都安装一遍，如果只安装一个插件的话建议使用NPM命令安装。

还有很多可用插件，具体如下：

1. 信息框\(`flexible-alerts`\)
2. 阅读统计（`pageview-count`）
3. 侧边栏宽度可调节（`splitter`）
4. 页脚版权（`page-copyright`）
5. 打赏功能（`donate`）
6. 分享当前页面（`sharing-plus`）
7. 修改标题栏图标（`custom-favicon`）
8. 复选框（`todo`）
9. 显示图片名称（`image-captions`）
10. 目录折叠（`toggle-chapters`）
11. 分章节展示（`multipart`）
12. 插入 `Logo`（`insert-logo`）
13. `Google` 分析（`ga`）
14. 返回顶部（`back-to-top-button`）
15. 代码添加行号和复制按钮（`code`）
16. 高级搜索，支持中文（`search-pro`）
17. 添加 `Github` 图标（`github`）



#### 左侧目录可折叠

**2.2.1 chapter-fold**

支持多层目录，点击导航栏的标题名就可以实现折叠扩展。  
在`book.json`的plugins参数中添加插件名：

```text
{
    "plugins": ["chapter-fold"]
}
```

然后使用`npm install gitbook-plugin-chapter-fold`命令安装插件。  
_**注意：**_要想目录折叠，`SUMMARY.md`目录应该如下：

```text
* [项目介绍](README.md)

* [tcp说明](doc/http/tcp/tcp说明.md)
    * [udp说明](doc/http/tcp/udp/udp说明.md)
* [html](doc/html/readme.md)
    * [HTML5-特性说明](doc/html/HTML5-特性说明.md)
```

如下写法会产生bug，导致CSS是收缩的，不能展开，效果如上面的动图：

```text
* CSS 
    * [说明](doc/css/readme.md)
```

**2.2.2 expandable-chapters**

这个插件也是左侧目录折叠的插件，不同的是可以解决`chapter-fold`插件的bug，怎么写都会折叠目录  
在`book.json`的plugins参数中添加插件名：

```text
{
    "plugins": [
         "expandable-chapters"
    ]
}
```

安装命令：`npm install gitbook-plugin-expandable-chapters`  
_**注意：**_这个插件也有问题，就是如下写法的，需要点击箭头才能展开收缩菜单：

```text
* [tcp说明](doc/http/tcp/tcp说明.md)
    * [udp说明](doc/http/tcp/udp/udp说明.md)
```

解决的办法是和`chapter-fold`插件一起用，互补一下各自的问题就完美解决了：

```text
"plugins": [
    "expandable-chapters",
    "chapter-fold",
]
```

#### 2.5 回到顶部按钮

在`book.json`的plugins参数中添加插件名和配置信息：

```text
{
    "plugins": [
        "back-to-top-button"
    ],
}
```

使用`npm install gitbook-plugin-back-to-top-button`命令安装插件。

#### 3.2 右上角添加github图标

在`book.json`的plugins参数中添加插件名和配置信息：

```text
{
    "plugins": [ 
        "github" 
    ],
    "pluginsConfig": {
        "github": {
            "url": "https://github.com/zhangjikai"
        }
    }
}
```

然后使用`npm install gitbook-plugin-github`命令安装插件。  
_**注意：**_  
如果使用npm命令安装后报错`GitBook doesn't satisfy the requirements of this plugin: >=4.0.0-alpha.0`.  
请使用`gitbook install`来安装.  
或者`npm uninstall gitbook-plugin-github`卸载后，使用`npm i gitbook-plugin-github@2.0.0`安装，然后查看是否还报错。

#### 3.3 edit-link在线编辑文件

`book.json`中插件名和配置信息：

```text
{
    "plugins": ["edit-link"],
    "pluginsConfig": {
      "edit-link": {
            "base": "//github.com/yulilong/book/edit/master",
            "label": "编辑此页面"
       }
    }
}
```

使用`npm i gitbook-plugin-edit-link`命令安装插件。

点击编辑按钮，即可跳转到github仓库在线编辑这个文件。

#### 3.6 prism代码块颜色插件\(测试没有通过\)

插件地址：[https://github.com/gaearon/gi...](https://github.com/gaearon/gitbook-plugin-prism)  
此插件需要禁用gitbook自带的`highlight`插件。  
`book.json`中插件名和配置信息：

```text
{
    "plugins": ["prism", "-highlight"],
    "pluginsConfig": {
      "prism": {
        "css": [
          "prismjs/themes/prism-okaidia.css"
        ]
      }
    }
}
```

使用`npm install gitbook-plugin-prism`命令安装插件。  
  
更多的颜色参考：[https://github.com/gaearon/gi...](https://github.com/gaearon/gitbook-plugin-prism)

_**注意：**_代码块的语言标注比如`JS`,`CSS`,如果标注一个插件不认识的语言，在运行打包命令`gitbook build .`这个插件会报错，提示不认识这个语言，这里需要注意一下。

