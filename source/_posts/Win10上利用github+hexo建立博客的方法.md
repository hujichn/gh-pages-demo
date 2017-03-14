---
title: Win10上利用github+hexo建立博客的方法
date: 2016-04-06 16:55:47
categories: other
tags: Blog
---

这几天心血来潮在Github上建了个博客，纠结了半天才倒腾好，希望以后自己能多总结下学的知识，先从建立博客开始咯~

<!--more-->

author: [@Huji][1]


## 1. 安装环境

Windows 10 64位系统

---

## 2. 创建自己的GitHub Pages

- 首先注册[GitHub][2]账号，用户名如hujichn

- 参考[教程][3]创建`GitHub Pages`。

- 访问上述教程时，注意选择左边创建`User or organization site`。

![User or organization site.png-9.8kB][4]

- 第一步：Create a repository，注意name必须和用户名一致如hujichn.github.io

- 在What git client are you using？这一步，如果没有下载过`Github for Windows`，可以选择`I don't know`，后面会出现下载链接，点击下载即可。这个下载的是在线安装包，可能会慢的感人，我后来找了一个[离线安装包][5] [^lixian]，双击运行GitHub.application即可安装。

![githubwindows.png-31.7kB][6]

---

## 3. 安装Hexo

- 第一步是安装[Node.js][7]，根据自己的电脑选择合适的版本下载即可安装。 安装完成后添加Path环境变量，使npm命令生效。

```
C:\Program Files\nodejs\node_modules\npm
```

安装完后在cmd中分别输入

```
node -v
npm -v
```

如果都显示了版本号，那么就安装成功了。

- GitHub Windows安装好之后，桌面上应该有一个Git Shell，双击打开，输入以下命令换源，否则安装hexo速度感人。
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

![changeSourse.png-3.2kB][8]

然后再输入以下两行命令：
```
cnpm install hexo-cli -g
cnpm install hexo --save
```

安装完成后，在GitHub默认路径下建立一个hexo的文件夹，如
`C:\Users\Huji\Documents\GitHub\hexo`。

在Git Shell中输入命令`cd hexo`切换到该文件夹下。再分别输入以下命令：

```
hexo init
cnpm install
```

安装相关插件（复制以下命令，在Shell窗口里右键粘贴即可）

```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
```

安装完成后，输入以下命令后，在浏览器中打开`localhost:4000`即可看到博客在本地的效果。在测试主题的一些效果的时候可以用这个命令在本地查看，等效果满意了再deploy到GitHub。
```
hexo s -g
```

---

## 4. 利用Hexo写博客并将Hexo产生的静态网页部署到GitHub

- 更改博客的设置信息，在hexo文件夹下，打开`_config.yml`文件，输入以下代码 [^config_code]，并进行相应修改。
```
#博客名称
title: 我的博客
#副标题
subtitle: 一天进步一点
#简介
description: 记录生活点滴
#博客作者（请修改）
author: John Doe
#博客语言
language: zh-Fans
#时区
timezone:

#博客地址,与申请的GitHub一致（请修改）
url: http://elfwalk.github.io
root: /
#博客链接格式
permalink: :year/:month/:day/:title/
permalink_defaults:

source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:

default_category: uncategorized
category_map:
tag_map:

#日期格式
date_format: YYYY-MM-DD
time_format: HH:mm:ss

#分页，每页文章数量
per_page: 5
pagination_dir: page

#博客主题
theme: landscape

#发布设置
deploy: 
  type: git
  #elfwalk改为你的github用户名（请修改）
  repository: https://github.com/elfwalk/elfwalk.github.io.git
  branch: master
```

- 在Git Shell中切换到hexo目录，输入以下命令即可创建一篇名为test的新文章，会有一个test.md出现在`hexo\source\_posts`这个目录中。
```
hexo new “test”
```

打开后看到默认会有title，date，tags等等信息，要写的文章内容可以在这个md中进行修改，可以先在后面随便打几个字作为正文。

![test.md][9]

- 接下来在Git Shell中输入以下命令，即可设置好身份信息
```
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
```

- 最终，使用以下命令即可将生成的网页部署到Github上。
```
hexo deploy -g
```
访问`http://userName.github.io`就可以看到博客啦。

也可以使用下面的命令先在本地生成预览
```
hexo g
hexo s
```

在浏览器输入localhost:4000就可以看到本地生成的网页了，这样万一有问题可以改好了再提交到github上。

---

## 5. 更换主题，使用第三方插件增加评论，阅读次数统计等功能。

主题有很多好看的，这里选择了一个比较简单大方的Next主题。参考[教程][10]即可完成配置。

 **注意的地方**：

- 教程中的`站点配置文件`是指`hexo`目录下的`_config.yml`，`主题配置文件`是指`hexo\themes\next`目录下的`_config.yml`。

- 教程中设置头像时修改的应该是`主题配置文件`，设置本地头像图片可以放在`hexo\source\images\`目录下，也可以放在`\hexo\themes\next\source\uploads`，要注意放置的位置要和配置相对应。

- 教程中设置第三方服务的全都应该填在`主题配置文件`，这里面作者已经给写好了，只是都注释掉了，只要去掉注释，填写相应的内容即可。

- 阅读统计量功能里面，Next主题文件已经不需要修改，从`配置LeanCloud`开始看即可。

- duoshuo评论功能打开之后，可以在`主题配置文件`中打开UA开关，这样会评论旁边会显示User使用的电脑环境等信息。usr_id可以在多说的网站，通过点击自己的用户名得到。

![UA.png-10.4kB][11]

- 设置头像翻转啊等一些动态效果：登录多说后台->设置，可以在自定义文本里面修改一些词语，在基本设置->自定义CSS中可以添加如下CSS代码 [^css]，实现后效果如图。

![comment.png-17.9kB][12]

```
/*UA Start*/
span.this_ua {
    background-color: #ccc!important;
    border-radius: 4px;
    padding: 0 5px!important;
    margin: 0 1px!important;
    border: 1px solid #BBB!important;
    color: #fff;

    /*text-transform: Capitalize!important;
    float: right!important;
    line-height: 18px!important;*/
}
.this_ua.platform.Windows{
    background-color: #39b3d7!important;
    border-color: #46b8da!important;
}
.this_ua.platform.Linux {
    background-color: #3A3A3A!important;
    border-color: #1F1F1F!important;
}
.this_ua.platform.Ubuntu {
    background-color: #DD4814!important;
    border-color: #DD4814!important;
}
.this_ua.platform.Mac {
    background-color: #666666!important;
    border-color: #666666!important;
}
.this_ua.platform.Android {
    background-color: #98C13D!important;
    border-color: #98C13D!important;
}
.this_ua.platform.iOS {
    background-color: #666666!important;
    border-color: #666666!important;
}
.this_ua.browser.Chrome{
    background-color: #EE6252!important;
    border-color: #EE6252!important;
}
.this_ua.browser.Chromium{
    background-color: #EE6252!important;
    border-color: #EE6252!important;
}
.this_ua.browser.Firefox{
    background-color: #f0ad4e!important;
    border-color: #eea236!important;
}
.this_ua.browser.IE{
    background-color: #428bca!important;
    border-color: #357ebd!important;
}
.this_ua.browser.Edge{
    background-color: #428bca!important;
    border-color: #357ebd!important;
}
.this_ua.browser.Opera{
    background-color: #d9534f!important;
    border-color: #d43f3a!important;
}
.this_ua.browser.Maxthon{
    background-color: #7373B9!important;
    border-color: #7373B9!important;
}
.this_ua.browser.Safari{
    background-color: #666666!important;
    border-color: #666666!important;
}
.this_ua.sskadmin {
    background-color: #00a67c!important;
    border-color: #00a67c!important;
}
/*UA End*/
/*Head Start*/
#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current {
    border: 0px;
    color: #6D6D6B;
    text-shadow: none;
    background: #F3F3F3;
}

#ds-thread #ds-reset .ds-highlight {
    font-family: Microsoft YaHei, "Helvetica Neue", Helvetica, Arial, Sans-serif;
    ;font-size: 100%;
    color: #6D6D6B !important;
}

#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current:hover {
    color: #696a52;
    background: #F2F2F2;
}

#ds-thread #ds-reset a.ds-highlight:hover {
    color: #696a52 !important;
}

#ds-thread {
    padding-left: 15px;
}

#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset #ds-hot-posts {
    overflow: visible;
}

#ds-thread #ds-reset .ds-post-self {
    padding: 10px 0 10px 10px;
}

#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset .ds-post-self {
    border: 0 !important;
}

#ds-reset .ds-avatar, #ds-thread #ds-reset ul.ds-children .ds-avatar {
    top: 15px;
    left: -20px;
    padding: 5px;
    width: 36px;
    height: 36px;
    box-shadow: -1px 0 1px rgba(0,0,0,.15) inset;
    border-radius: 46px;
    background: #FAFAFA;
}

#ds-thread .ds-avatar a {
    display: inline-block;
    padding: 1px;
    width: 32px;
    height: 32px;
    border: 1px solid #b9baa6;
    border-radius: 50%;
    background-color: #fff !important;
}

#ds-thread .ds-avatar a:hover {
}

#ds-thread .ds-avatar > img {
    margin: 2px 0 0 2px;
}

#ds-thread #ds-reset .ds-replybox {
    box-shadow: none;
}

#ds-thread #ds-reset ul.ds-children .ds-replybox.ds-inline-replybox a.ds-avatar,
#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar {
    left: 0;
    top: 0;
    padding: 0;
    width: 32px !important;
    height: 32px !important;
    background: none;
    box-shadow: none;
}

#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar img {
    width: 32px !important;
    height: 32px !important;
    border-radius: 50%;
}

#ds-reset .ds-replybox a.ds-avatar,
#ds-reset .ds-replybox .ds-avatar img {
    padding: 0;
    width: 32px !important;
    height: 32px !important;
    border-radius: 5px;
}

#ds-reset .ds-avatar img {
    width: 32px !important;
    height: 32px !important;
    border-radius: 32px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.22);
    -webkit-transition: .8s all ease-in-out;
    -moz-transition: .4s all ease-in-out;
    -o-transition: .4s all ease-in-out;
    -ms-transition: .4s all ease-in-out;
    transition: .4s all ease-in-out;
}

.ds-post-self:hover .ds-avatar img {
    -webkit-transform: rotateX(360deg);
    -moz-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    transform: rotate(360deg);
}

#ds-thread #ds-reset .ds-comment-body {
    -webkit-transition-delay: initial;
    -webkit-transition-duration: 0.4s;
    -webkit-transition-property: all;
    -webkit-transition-timing-function: initial;
    background: #F7F7F7;
    padding: 15px 15px 15px 47px;
    border-radius: 5px;
    box-shadow: #B8B9B9 0 1px 3px;
    border: white 1px solid;
}

#ds-thread #ds-reset ul.ds-children .ds-comment-body {
    padding-left: 15px;
}

#ds-thread #ds-reset .ds-comment-body p {
    color: #787968;
}

#ds-thread #ds-reset .ds-comments {
    border-bottom: 0px;
}

#ds-thread #ds-reset .ds-powered-by {
    display: none;
}

#ds-thread #ds-reset .ds-comments a.ds-user-name {
    font-weight: normal;
    color: #3D3D3D !important;
}

#ds-thread #ds-reset .ds-comments a.ds-user-name:hover {
    color: #D32 !important;
}

#ds-thread #ds-reset #ds-bubble {
    display: none !important;
}

#ds-thread #ds-reset #ds-hot-posts {
    border: 0;
}

#ds-reset #ds-hot-posts .ds-gradient-bg {
    background: none;
}

#ds-thread #ds-reset .ds-comment-body:hover {
    background-color: #F1F1F1;
    -webkit-transition-delay: initial;
    -webkit-transition-duration: 0.4s;
    -webkit-transition-property: all;
    -webkit-transition-timing-function: initial;
}
/*Head End*/
```

## 6. 新建about，categories页面以及插入外链图片或者视频

- 新建about, categories页面，在Git Shell中切换到hexo目录输入以下命令：
```
hexo new page "about"
hexo new page "categories"
```

- 命令输入完之后，会生成categories文件夹，里面会有index.md的文件，打开文件，修改为：
```
---
title: categories
date: 2016-04-06 17:21:47
type: "categories"
---
```

- 在about页面可以插入图片，写文字等等，这里介绍下插入外链图片的方法。
  - 首先，[注册七牛网][13]，注册好后要实名认证，然后就可以新建一个公开空间，点击内容管理，就可以上传文件并获得图片的外链了。
  - markdown中插入图片的方法这里顺便一提：

```
![图片名称](外链地址)
```

- 插入视频通常是在视频网站选择嵌入，复制其提供的html代码，直接粘贴进博客的md文件中即可，因为markdown是支持html语法的~

![insert_vedio](http://7xsoqo.com1.z0.glb.clouddn.com/hexo%2Finsert_vedio.png)


## 7. 更新的相关操作

- 更新hexo：
```
npm update -g hexo
```

- 更新主题：
cd 到主题文件夹，执行命令：
```
git pull
```

- 更新插件：
```
npm update
```

## 8. 增加脚注功能footnotes

在Cmd Markdown里面写东西的时候是支持脚注的，但是Hexo默认的markdown渲染插件好像不支持，所以只好换了一个，步骤如下：

- 在hexo目录下输入
```
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it --save
```

- 不过此时的hexo-renderer-markdown-it还是用不了脚注，我们需要加上plugin，这里写了一句footnote的安装，其他的也可以干脆一起装了，还可以支持emoji表情哦~~ :smile:
```
cd node_modules/hexo-renderer-markdown-it/
npm install markdown-it-footnote --save
```

- 装好之后编辑Hexo的配置文件`_config.yml`，插入以下句子即可。
```
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: false
    linkify: true
    typographer: true
    quotes: '“”‘’'
  plugins:
    - markdown-it-footnote
    - markdown-it-sup
    - markdown-it-sub
    - markdown-it-abbr
    - markdown-it-emoji
  anchors:
    level: 2
    collisionSuffix: 'v'
    permalink: true
    permalinkClass: header-anchor
    permalinkSymbol: ¶
```

## 9. 数学公式后面有竖线的问题解决

这个问题在[Issues][14]里已经有人提出来，作者给出了解决方案，为了方便也在这里记录下：

- 首先，编辑next的主题配置文件`_config.yml`，在里面找到mathjax的部分，替换为以下内容：
```
# MathJax Support
mathjax:
  enable: true
  cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```

- 打开`\hexo\themes\next\layout\_scripts\third-party\mathjax.swig`，将其内容替换为：
```
{% if theme.mathjax.enable %}
  <script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        inlineMath: [ ['$','$'], ["\\(","\\)"]  ],
        processEscapes: true,
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
      }
    });
  </script>

  <script type="text/x-mathjax-config">
    MathJax.Hub.Queue(function() {
      var all = MathJax.Hub.getAllJax(), i;
      for (i=0; i < all.length; i += 1) {
        all[i].SourceElement().parentNode.className += ' has-jax';
      }
    });
  </script>
  <script type="text/javascript" src="{{ theme.mathjax.cdn }}"></script>
{% endif %}
```

测试一下，应该可以啦~
$$\sum_{i=1}^n a_i=0$$

[^lixian]: http://www.yaozeyuan.online/?p=104

[^config_code]: http://blog.csdn.net/jzooo/article/details/46781805

[^css]: http://wsgzao.github.io/post/duoshuo/


  [1]: https://hujichn.github.io/
  [2]: https://github.com/
  [3]: https://pages.github.com/
  [4]: http://static.zybuluo.com/huji/k9w3r3pthtflj27qc8293ty1/User%20or%20organization%20site.png
  [5]: http://pan.baidu.com/s/1Hkewm
  [6]: http://static.zybuluo.com/huji/4v6i1chihhfmsg3kfzwqd3uy/githubwindows.png
  [7]: https://nodejs.org/en/
  [8]: http://static.zybuluo.com/huji/qy08e7rtzpt9hd4ym5rii0zh/changeSourse.png
  [9]: http://static.zybuluo.com/huji/m66fsq13ywwzj3xdrzzvwrg1/QQ%E6%88%AA%E5%9B%BE20160406160448.png
  [10]: http://theme-next.iissnan.com/getting-started.html
  [11]: http://static.zybuluo.com/huji/91xx7uzlhaptef9tv1x66x65/UA.png
  [12]: http://static.zybuluo.com/huji/dgjg98df8rcrj3vo58f7dr9x/comment.png
  [13]: https://portal.qiniu.com/signup?code=3lo4whaobe9g2
  [14]: https://github.com/iissnan/hexo-theme-next/issues/752