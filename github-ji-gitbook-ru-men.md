# github及gitbook入门

一、Github

二、gitbook

gitbook 插件

gitbook插件可以解决一些网站不太方便的地方，如侧边栏导航不能收缩，自带搜索不支持中文等。

插件安装、使用方法：

> 1、 在`book.json`的plugins参数中添加插件名。  
> 2、终端使用`gitbook install`来安装插件，或使用NPM命令来单独安装：`npm install gitbook-plugin-插件名`。  
> 3、重启服务或者重新打包就能看见效果。  
> 4、如果使用`gitbook install`安装的很慢，建议使用`npm init`初始化一个package.json文件，然后每个包通过npm命令安装，以后就可以通过`npm install`一键快速安装依赖包了。

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
下过如下图：

![clipboard.png](https://segmentfault.com/img/bVbvzyh)  
点击编辑按钮，即可跳转到github仓库在线编辑这个文件。

#### 3.6 prism代码块颜色插件

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
效果如下图：  
![clipboard.png](https://segmentfault.com/img/bVbvpGH)

更多的颜色参考：[https://github.com/gaearon/gi...](https://github.com/gaearon/gitbook-plugin-prism)

_**注意：**_代码块的语言标注比如`JS`,`CSS`,如果标注一个插件不认识的语言，在运行打包命令`gitbook build .`这个插件会报错，提示不认识这个语言，这里需要注意一下。

