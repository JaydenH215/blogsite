---
title: 博客个性化配置
date: 2019-05-10 10:00:00
tags: 
- 博客
- 搭建
- 环境
- 配置

categories:
- 社交
---

Hexo 是一个快速、简洁且高效的博客框架。为了打造心目中的博客，针对next主题修改了以下个性化的配置。
<!-- more --> 

目录
===
<!-- TOC -->

- [配置方式](#配置方式)
- [配置主题（theme）](#配置主题theme)
- [配置风格（scheme）](#配置风格scheme)
- [配置语言（language）](#配置语言language)
- [配置描述（description）](#配置描述description)
- [配置目录（toc）](#配置目录toc)
- [配置头像 （avator）](#配置头像-avator)
- [配置拉条（sidebar）](#配置拉条sidebar)
- [配置社交（social）](#配置社交social)
- [配置打赏](#配置打赏)
- [配置菜单（menu）](#配置菜单menu)
- [配置标签](#配置标签)
- [配置分类](#配置分类)
- [配置评论](#配置评论)
- [配置分享](#配置分享)
- [配置版权](#配置版权)
- [配置脚注（footer）](#配置脚注footer)
- [配置统计](#配置统计)
- [配置搜索](#配置搜索)

<!-- /TOC -->


# 配置方式
搭建的博客过程中用到了以下几种修改方式：

- 修改博客工程根目录下的`_config.yml`文件
- 修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件
- 默认框架不符合审美时候，修改源代码或者增加文件
- 增加插件

# 配置主题（theme）
    
修改博客工程根目录下的`_config.yml`文件

    theme: hexo-theme-next
    
# 配置风格（scheme）

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    scheme: Mist

# 配置语言（language）

修改博客工程根目录下的`_config.yml`文件

    language: zh-CN

# 配置描述（description）

修改博客工程根目录下的`_config.yml`文件

    description: "Stay hungry, Stay foolish."

# 配置目录（toc）

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    toc:
      enable: true
      number: false

# 配置头像 （avator）

将头像图片文件`123.jpg`放进/source/images中，修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    avatar: 
      url: /images/123.jpg


# 配置拉条（sidebar）

    none

# 配置社交（social）

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    social:
      GitHub: https://github.com/jaydenh215 || github

# 配置打赏

将打赏图片文件`wechatpay.jpg`放进/source/images中，修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    reward_settings:
      enable: true
      comment: “如果觉得还不错，请我喝杯咖啡吧~”

    reward:
      wechatpay: /images/wechatpay.jpg

# 配置菜单（menu）

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    menu:
      home: / || home
      tags: /tags/ || tags
      categories: /categories/ || th


# 配置标签

增加一个页面（page）用来汇总`标签`

    hexo new page tags

修改生成的页面文件内容（/source/tags/index.md）

    ---
    title: tags
    date: 2019-05-10 13:49:39
    type: "tags"
    ---

给文章添加`标签`属性

    ---
    title: 博客个性化配置
    date: 2019-05-10 10:00:00
    tags: 
    - 博客
    - 搭建
    - 环境
    - 配置
    ---

# 配置分类

增加一个页面（page）用来汇总`类别`

    hexo new page categories

修改生成的页面文件内容（/source/categories/index.md）

    ---
    title: categories
    date: 2019-05-10 13:44:06
    type: "categories" 
    ---

给文章添加`类别`属性

    ---
    title: 博客个性化配置
    date: 2019-05-10 10:00:00
    categories:
    - 社交
    ---

# 配置评论

next主题集成了许多第三方厂家的评论功能插件，选择比较精简的[Valine](https://theme-next.org/docs/third-party-services/comments-and-widgets)。

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件，主要是`app id`和`app key`。

    valine:
      enable: true 
      appid: xxxxxxxxxx
      appkey: xxxxxxxxxx
      guest_info: nick,mail
      notify: true
      placeholder: Comment here ...

增加评论区之后，右下角会有`Power by Valine`，可以这样删掉：

找到`/themes/hexo-theme-next/layout/_third-party/comments/valine.swig`文件并修改代码

修改前：

    <script>
    var GUEST = ['nick', 'mail', 'link'];
    var guest = '{{ theme.valine.guest_info }}';
    guest = guest.split(',').filter(function(item) {
        return GUEST.indexOf(item) > -1;
    });
    new Valine({
        el: '#comments',
        verify: {{ theme.valine.verify }},
        notify: {{ theme.valine.notify }},
        appId: '{{ theme.valine.appid }}',
        appKey: '{{ theme.valine.appkey }}',
        placeholder: '{{ theme.valine.placeholder }}',
        avatar: '{{ theme.valine.avatar }}',
        meta: guest,
        pageSize: '{{ theme.valine.pageSize }}' || 10,
        visitor: {{ theme.valine.visitor }},
        lang: '{{ theme.valine.language }}' || 'zh-cn'
    });
    </script>

修改后：

    <script>
    var GUEST = ['nick', 'mail', 'link'];
    var guest = '{{ theme.valine.guest_info }}';
    guest = guest.split(',').filter(function(item) {
        return GUEST.indexOf(item) > -1;
    });
    new Valine({
        el: '#comments',
        verify: {{ theme.valine.verify }},
        notify: {{ theme.valine.notify }},
        appId: '{{ theme.valine.appid }}',
        appKey: '{{ theme.valine.appkey }}',
        placeholder: '{{ theme.valine.placeholder }}',
        avatar: '{{ theme.valine.avatar }}',
        meta: guest,
        pageSize: '{{ theme.valine.pageSize }}' || 10,
        visitor: {{ theme.valine.visitor }},
        lang: '{{ theme.valine.language }}' || 'zh-cn'
    });
    //新增
    var infoEle= document.querySelector('#comments .info');
    if (infoEle && infoEle.childNodes && infoEle.childNodes.length > 0) {
        infoEle.childNodes.forEach(function(item) {
        item.parentNode.removeChild(item);
        });
    }
    </script>

# 配置分享

修改博客工程根目录下的`_config.yml`文件

    baidushare: true
        
修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    baidushare: 
      type: button



按照[链接中的方法](https://github.com/hrwhisper/baiduShare)，将static放进`/themes/hexo-theme-next/source`目录下。

修改`baidushare.swig`文件中的代码。

修改前：
    
    .src='//bdimg.share.baidu.com/static/api/js/share.js?cdnversion='+~(-new Date()/36e5)];

修改后：

    .src='/static/api/js/share.js?cdnversion='+~(-new Date()/36e5)]; 
    

# 配置版权

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    creative_commons:
      post: true

修改博客工程根目录下的`_config.yml`文件

    url: https://jaydenh215.github.io/

# 配置脚注（footer）

修改主题目录（/themes/hexo-theme-next）下的`_config.yml`文件

    footer:
      powered:
        enable: false
        
      theme:
        enable: false

# 配置统计

next主题集成了许多第三方厂家的统计功能插件，选择[LeanCloud](https://theme-next.org/docs/third-party-services/statistics-and-analytics)。

完成上述步骤之后，会发现`阅读次数`后面没有数字，那是因为`LeanCloud`和`next`主题还没有联系起来，
需要按照[该博主的方法](https://blog.csdn.net/lijing742180/article/details/87928554)来实现。

# 配置搜索