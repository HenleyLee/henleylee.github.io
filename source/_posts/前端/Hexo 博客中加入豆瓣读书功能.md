---
title: Hexo 博客中加入豆瓣读书功能
categories: 前端
tags:
  - Hexo
abbrlink: efcad6e3
date: 2019-12-18 11:33:45
---

在 [Hexo](https://hexo.io/) 博客个性化定制中，加入豆瓣读书界面是一个很不错的功能，那么是怎么做到的呢？其实很简单，我们只需要加入一个 `hexo-douban` 模块即可。

## 简介 ##
**`hexo-douban`**是一个在 [Hexo](https://hexo.io/) 页面中嵌入豆瓣个人主页的小插件。

GitHub：[https://github.com/mythsman/hexo-douban](https://github.com/mythsman/hexo-douban)

## 安装 ##
使用时需要先安装依赖模块，在 `Hexo` 站点根目录下执行以下命令：
```bash
$ npm install hexo-douban --save
```

## 安装 ##
然后在 `Hexo` 站点根目录配置文件 `_config.xml` 中(不是主题的配置文件)添加如下配置：
```yaml
douban:
  user: mythsman
  builtin: false
  book:
    title: 'This is my book title'
    quote: 'This is my book quote'
  movie:
    title: 'This is my movie title'
    quote: 'This is my movie quote'
  game:
    title: 'This is my game title'
    quote: 'This is my game quote'
  timeout: 10000 
```
参数说明：
 - **`user：`**你的豆瓣ID。打开豆瓣，登入账户，然后在右上角点击 “个人主页” ，这时候地址栏的URL大概是这样：“https://www.douban.com/people/xxxxxx/” ，其中的”xxxxxx”就是你的个人ID了。
 - **`builtin：`**是否将生成页面的功能嵌入 `hexo s` 和 `hexo g` 中，默认是 `false`，另一可选项为 `true` (1.x.x版本新增配置项)。
 - **`title：`**该页面的标题。
 - **`quote：`**写在页面开头的一段话，支持 `html` 语法.
 - **`timeout：`**爬取数据的超时时间，默认是 10000ms，如果在使用时发现报了超时的错(ETIMEOUT)可以把这个数据设置的大一点。

> 由于 `hexo-douban` 是默认抓取豆瓣读书、豆瓣电影以及豆瓣游戏的，如果只想要其中一部分，可以把其它部分在上述配置文件中去掉即可。

## 启动 ##
只需要在 `Hexo` 站点根目录下执行以下命令：`hexo clean` && `hexo douban -bgm` && `hexo g` && `hexo s` 即可。注意其中开启 `hexo-douban` 的命令中，如果不加参数，那么默认参数为 `-bgm`，`-bgm` 代表的是 `book`、`game`、`movie` 三个参数，如果只需要其中的一部分就只带你想要的那些参数。

> **`需要注意的是：`**通常大家都喜欢用 `hexo d` 来作为 `hexo deploy` 命令的简化，但是当安装了 `hexo douban` 之后，就不能用 `hexo d` 了，因为 `hexo douban` 跟 `hexo deploy` 的简写都是 `hexo d`。由于 `hexo douban` 的简写指令与 `hexo deploy` 的简写指令 `hexo d` 冲突，因此在进行二者部署的时候，只能都打全名而不能打简写形式。

## 测试 ##
如果配置都没问题之后，只需要在站点目录下测试 [`http://localhost:4000/books`](http://localhost:4000/books) 或者 [`http://localhost:4000/movies`](http://localhost:4000/movies) 等，如果看到页面了就说明成功了。

## 部署 ##
如果测试没有问题，就可以在菜单栏中添加菜单了，打开主题配置文件 `_config.xml`，找到菜单项配置，可以选择性的添加下面内容：
```yaml
menu:
  Home: /
  Archives: /archives
  Books: /books     #This is your books page
  Movies: /movies   #This is your movies page
  Games: /games   #This is your games page
```

注意添加完成之后按钮并不是中文的，这是由于在 `languages` 文件夹下面的 `zh-CN`（中文语言配置文件）没有添加上述对应的中文参数信息，所以中文配置文件也需要主动添加。

## 升级 ##
由于作者会不定期更新一些功能或者修改一些 Bug，所以如果想使用最新的特性，可以用下面的方法来更新：
 - 修改 `package.json` 内 `hexo-douban` 的版本号至最新
 - 重新安装最新版本 `npm install hexo-douban --save`

或者使用 `npm install hexo-douban --update --save` 直接更新。


