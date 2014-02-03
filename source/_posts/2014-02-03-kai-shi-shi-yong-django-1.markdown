---
layout: post
title: "开始使用Django(一)"
date: 2014-02-03 18:24:55 +0800
comments: true
categories: 
---
 在开始学习使用Django之后，第一次遇到的难题就是静态文件（static）的配置，网上资料很多，但也经过多次失败的尝试。所以决心理清这块内容的实际原理。

## Django目录结构
Django本身没有制定标准的目录结构规范，所以我们会看到不同的项目目录结构，当然也对应着不同的配置文件。但对于有代码洁癖的人来说，想从一开始就`最佳实践`。  
在[Django最佳实践](http://yangyubo.com/django-best-practices)中有推荐的布局，如下：  
<img src="/assets/img/django/django-project-dirs.jpg" alt="Django项目推荐布局" title="Django项目推荐布局" width="100%" />
当然目前基于开发比较多的是WEB应用，我们可以将项目（Project）定位为一个平台，应用（App）定位为一个模块。  

其中静态文件夹有两种部署模式：“分布式“、“集中式“。分布式是在各个APP下建立static文件夹并包含img\css\js文件。集中式是在Project路径下建立static文件夹并包含img\css\js文件，即统一管理所有App使用到的静态资源。

## Django如何处理静态文件（基本原理）
在实际的WEB站点中，静态文件一般是单独处理，例如使用naginx作为静态文件服务器或者使用CDN进行静态资源加速。当然Django中也可以这么去处理，Django在开发环境这么去处理静态文件：通过一系列的配置（setting.py指定静态文件的路径，url.py添加url匹配路由），然后交由Django内置的django.views.static.server()来处理。

## Django如何处理静态文件（详细配置）
以下的详细配置，是基于Django1.4，并且准备采用“集中式”静态文件管理方式来阐述的。  

`setting.py`有如下几个重要配置需要注意：  

STATIC_ROOT:  运行collectstatic，所有app中的静态文件包括在STATICFILES_DIRS中指定的都会手机到此目录，这在部署的时候很方便，开发的时候可以忽略此设置。（建议留空！）  

STATIC_URL:  用来配置静态文件请求的URL前缀，例如域名http://www.ooxx.com，并且你的STATIC_URL='/static/'，那么你访问静态文件的URL将类似http://www.ooxx.com/static/img/xixihaha.jpg

ADMIN_MEDIA_PREFIX:  是Django后台管理模块使用的静态文件配置。  

STATICFILES_DIRS:  （默认情况下staticfiles认为静态文件是存放在各app的static目录下）这是静态文件放置的目录，我们如果采用“集中式”管理静态文件，即static文件夹在项目的根目录下和App平级。为了移植方便，可以这么去配置：


    BASE = os.path.dirname(os.path.abspath(__file__))  
    STATICFILES_DIRS = (os.path.join(BASE,'static').replace('\\','/'),)


`url.py`对应也需要配置，在末尾增加：  

    urlpatterns += staticfiles_urlpatterns() #静态文件的路由

上面的配置结束后，我们就可以这么去使用了：
   
    <link rel="stylesheet" href="{{ STATIC_URL }}css/bootstrap/bootstrap.css" />  


-EOF-
