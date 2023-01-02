---
title: 主题Icarus定制化
date: 2019-09-27 03:55:50
tags: 
- hexo
- icarus
top: 0
---
博客迁移后，主题也从nexT切换成icarus，我的一些定制化配置。所有改动都上传到我的fork的仓库里了，地址： [https://github.com/skybcyang/hexo-theme-icarus](https://github.com/skybcyang/hexo-theme-icarus)
<!-- more -->

### 语言设置为中文
编辑hexo目录下的`_config.xml`文件

    language: zh-CN

### 导航栏标题设置
编辑`themes/icarus/`目录下的`_config.xml`文件

    menu:
        Home: /
        Archives: /archives
        Categories: /categories
        Tags: /tags
        About: /about
改为

    menu:
        主页: /
        分类: /categories
        Leetcode: /categories/leetcode
        Github: /categories/github
        随笔: /categories/essay
        游戏: /categories/game
        摄影: /categories/images
        归档: /archives
        标签: /tags
        关于: /about


### 三列改为两列
把所有wedgit改到左边，改变布局比例
首先在`themes/icarus/_config.yml`修改所有的wedget到left对齐
但是文章区域比较窄，修改`themes/icarus/layout/layout.ejs`

把layout/common/widget.ejs中side_column_class()的case 2
的返回值改成return 'is-4-tablet is-4-desktop is-3-widescreen';就好啦！

### 添加评论
现在想要添加gitment作为评论的体系，

### 添加分享
`themes/icarus/_config.yml`

    share:
        # Share plugin name
        type: sharejs

### 添加目录
确保`themes/icarus/_config.yml`中有

    widgets:
        -
            # Widget name
            type: toc
            # Where should the widget be placed, left or right
            position: left

然后在文章头部添加标签

    toc: true


### 文章置顶功能
在`themes/icarus/_config.yml`配置

    index_generator:
        path: ''
        per_page: 10
        order_by:
            top: -1
            date: -1

修改`/⁨node_modules⁩/hexo-generator-index⁩/lib⁩/generator.js`

    var paginationDir = config.pagination_dir || 'page';

    // added code
    posts.data = posts.data.sort(function(a, b) {
    if(a.top && b.top) {
        if(a.top == b.top) return b.date - a.date;
        else return b.top - a.top;
    }
    else if(a.top && !b.top) {
        return -1;
    }
    else if(!a.top && b.top) {
        return 1;
    }
    else return b.date - a.date;
    });
    // end

    var path = config.index_generator.path || '';

修改模板中的post.md，添加top属性并设置默认值为0

    /scaffolds/post.md
    ---
    title: {{ title }}
    date: {{ date }}
    tags:
    top: 0
    ---

最后，根据大家自己的喜好在前端添加标签咯~

    /themes/icarus/layout/common/article.ejs

    <% if (post.top>0) { %>
    <i class="fas fa-arrow-alt-circle-up" style="color:#3273dc"></i>
    <span class="level-item" style="color:#3273dc">&nbsp;置顶</span>
    <% } %>


### 添加阅读计数


### 增加看板娘
在hexo目录下执行

    npm install --save hexo-helper-live2d

后来觉得不好看就卸掉了

    npm uninstall hexo-helper-live2d

### 修改社交链接

Icarus的默认社交链接有`Github`,`Facebook`,`Twiter`,`Dribbble`,`RSS`
需要定制化图标是文件


### 注释最近文章的图片展示
我觉得最近文章的图片展示太花里胡哨了，想要去掉
在`themes/icarus/layout/widget/recetn_posts.ejs`中注释掉

    <!-- <% if (!has_config('article.thumbnail') || get_config('article.thumbnail') !== false) { %>
    <a href="<%- url_for((post.link?post.link:post.path)) %>" class="media-left">
        <p class="image is-64x64">
            <img class="thumbnail" src="<%= get_thumbnail(post) %>" alt="<%= post.title %>">
        </p>
    </a>
    <% } %> -->

### 展示侧边栏定制化
主题默认的是在所有页面都展示相同的侧边栏，我觉得有些信息冗余，比如说在阅读文章时就不需要展示友链信息。
所以得对不同页面下的侧边栏展示做一个定制化。

### 添加自定义图标
在个人介绍侧边栏中的社交链接图标如facebook、twitter等平时几乎不用，所以换成了平时常用的网易云、微博等链接，图标定义如下

- weibo
- cloudmusic
- steam

这些都是有对应的图标的，都在对应体系中
原先是添加了微博的图标，但是微博没事让我关注一些奇怪的博主，于是我决定卸载微博的app，在博客中也去掉微博的链接


## 后记
后来不用了，改成了新主题Fluid，除了改改主题颜色背景图等，也不再关注博客功能了，思想改了吧，还是应该以输出内容为主。