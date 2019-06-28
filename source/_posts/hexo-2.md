---
title: 换电脑后Hexo更新
date: 2019-05-07 11:58:18
tags: "Hexo"
categories: "Hexo"
---

现在大部分时间使用公司电脑，于是搞一下。
主要参考这个[解决办法](https://www.zhihu.com/question/21193762)，master分支上是生成的页面文件，hexo分支上是源文件，设置hexo为默认分支。
遇到的第一个问题就是多个git账号，参考这个[解决办法](https://www.jianshu.com/p/b02645fff791)，主要思路是在ssh中增加一个config文件。

```
# 别名，可以随便取
Host github
    User git
    Hostname github.com
    IdentityFile ~/.ssh/github_id_rsa

Host gitlab
    User git
    Hostname gitlab.xxx.com
    IdentityFile ~/.ssh/id_rsa
```

这里有个小小的坑，就是之前设置了git config —global，在hexo deploy时会使用全局配置的用户名进行，需要在hexo的项目中配置局部设置。

```shell
$ git config user.name "A"
$ git config user.email "A@gmail.com"
```

这样就会使用配置后的局部用户名进行push。

其次就是不要用hexo init命令。原因是当前目录已经建立了git仓库环境, hexo init会覆盖到当前的git环境，重建一个新的，这样和我们的私有Hexo源码仓库脱离了联系。
最后，由于在yml文件已经配置了deploy的branch是master，所以执行hexo d时还是会将页面文件保存至master，而源文件需要手动push到hexo分支上。

一些hexo常用命令：
`hexo new newpage  # 新建`
`hexo g  # 生成`
`hexo s  # 本地预览`
`hexo d  # 部署`

另外，在mac上发现了一款markdown神器——[typora](https://typora.io/)
注意一下，直接使用回车会有空行，使用shift+回车就可以没有空行了。

一些typora常用快捷键：
超链接  `⌘ + k`
代码块 ``⌃ + ` ``

ps1: [mac技术符号输入](https://sspai.com/post/36242)
ps2: 如果两个重音符`` ` ``中间的内容也包含重音符，则在最外层需要用**两个连续重音符**夹起来