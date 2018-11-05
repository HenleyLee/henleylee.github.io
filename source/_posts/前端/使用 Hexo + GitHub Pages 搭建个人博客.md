---
title: 使用 Hexo + GitHub Pages 搭建个人博客
date: 2018-06-03 10:25:30
categories: 前端
tags:
  - Hexo
---

## 搭建 Node.js 环境 ##
Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。Node.js 可以在非浏览器环境下，解释运行 JS 代码。

在 [Node.js 官网](https://nodejs.org/en/) 下载安装包，保持默认设置，一路 Next 即可快速完成安装。

安装完成后打开命令提示符，输入 `node -v`、`npm -v`，正确输出版本号则说明 Node.js 环境配置成功！

> 为什么要搭建 Node.js 环境？ - 因为 Hexo 博客系统是基于 Node.js 编写的。

## 搭建 Git 环境 ##
Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。

在 [Git 官网](https://git-scm.com/) 下载安装包，保持默认设置，一路 Next 即可快速完成安装。

安装完成后在电脑桌面点击右键，打开 `Git Bush Here`，输入 `git --version`，正确输出版本号则说明 Git 环境配置成功！

> 为什么要搭建 Git 环境？ - 因为需要把本地的网页和文章等提交到 GitHub 上。

## 创建 GitHub 配置 ##
GitHub 是一个面向开源及私有软件项目的托管平台，因为只支持 Git 作为唯一的版本库格式进行托管，故名 GitHub。

GitHub Pages 本用于介绍托管在GitHub的项目，不过，由于他的空间免费稳定，用来做搭建一个博客再好不过了。

### 创建仓库 ###
在 [GitHub 官网](https://github.com/) 注册一个账号，完成注册后，在我们的 GitHub 上面右上角的 `New repository` 来创建一个仓库。

> **注意：** 仓库名必须遵守相应格式：`username.github.io`，这样子在访问主页的时候直接用 `username.github.io` 就能访问。`username` 是你注册的 GitHub 用户名。

### GitHub Pages ###
仓库创建完成后，开始设置我们的 GitHub Pages。打开我们刚刚创建的仓库，然后点开 `Settings`，移到 `GitHub Pages`，点击 `Choose a theme` 进入主题选择页面，选择一个主题，然后点击 `Select theme` 即可。

完成上面的配置后，访问 `your_username.github.io`，如果可以正常访问，那么 Github 的配置已经结束了。。

## 安装配置 Hexo ##
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

Hexo 详细介绍请阅读 [Hello Hexo](./hello-hexo/) 或 [Hexo 官网](https://hexo.io/) 。】

### 安装 Hexo ###
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
 - [Node.js](https://nodejs.org/)
 - [Git](https://git-scm.com/)

如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。
``` bash
$ npm install -g hexo-cli
```

安装完成后打开 `Git Bush Here`，输入 `hexo version`，正确输出版本号则说明 Hexo 环境配置成功！

### 初始化 Hexo ###
安装 Hexo 完成后，请执行下列命令来初始化 Hexo，Hexo 将会在指定文件夹中新建所需要的文件：
``` bash
$ hexo init Blog
$ cd Blog
$ npm install
```
也可以直接 `clone` 前面所创建的 `your_username.github.io` 替代 `Blog` 目录。

完成上面的初始化工作后，指定文件夹的目录如下：
```
.
├── .deploy             # 需要部署的文件
├── node_modules        # Hexo插件
├── public              # 生成的静态网页文件
├── scaffolds           # 模板
├── source              # 博客正文和其他源文件，404、favicon、CNAME 都应该放在这里
| ├── _drafts           # 草稿
| └── _posts            # 文章
├── themes              # 主题
├── _config.yml         # 全局配置文件
└── package.json        # npm 依赖等
```

### 运行本地 Hexo 服务 ###
``` bash
$ hexo server
or
$ hexo s
```
通过 `hexo server` 命令启动服务器。默认情况下，访问网址为： [http://localhost:4000/](http://localhost:4000/) 。如果 [http://localhost:4000/](http://localhost:4000/) 能够正常访问，则说明 Hexo 本地博客已经搭建起来了，只是本地哦，别人看不到的。

## 配置 ##
您可以在站点主目录的 `_config.yml` 文件中修改大部份的配置。

### Site ###
| 参数          | 描述                                                                                            |
| ------------- | ----------------------------------------------------------------------------------------------- |
| `title`       | 网站标题                                                                                        |
| `subtitle`    | 网站副标题                                                                                      |
| `description` | 网站描述                                                                                        |
| `author`      | 您的名字                                                                                        |
| `language`    | 网站使用的语言                                                                                  |
| `timezone`    | 网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：`America/New_York`、 `Japan` 和 `UTC` 。 |

其中，`description` 主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author` 参数用于主题显示文章的作者。

### URL ###
| 参数                 | 描述                                                    | 默认值                      |
| -------------------- | ------------------------------------------------------- |                             |
| `url`                | 网址                                                    |                             |
| `root`               | 网站根目录                                              |                             |
| `permalink`          | 文章的 [永久链接](https://hexo.io/docs/permalinks) 格式 | `:year/:month/:day/:title/` |
| `permalink_defaults` | 永久链接中各部分的默认值                                |                             |

>    如果您的网站存放在子目录中，例如 `http://yoursite.com/blog`，则请将您的 `url` 设为 `http://yoursite.com/blog` 并把 `root` 设为 `/blog/`。
 	
### Directory ###
| 参数           | 描述                                                                                       | 默认值           |
| -------------- | ------------------------------------------------------------------------------------------ |                  |
| `source_dir`   | 资源文件夹，这个文件夹用来存放内容。                                                       | `source`         |
| `public_dir`   | 公共文件夹，这个文件夹用于存放生成的站点文件。                                             | `public`         |
| `tag_dir`      | 标签文件夹                                                                                 | `tags`           |
| `archive_dir`  | 归档文件夹                                                                                 | `archives`       |
| `category_dir` | 分类文件夹                                                                                 | `categories`     |
| `code_dir`     | Include code 文件夹                                                                        | `downloads/code` |
| `i18n_dir`     | 国际化（i18n）文件夹                                                                       | `:lang`          |
| `skip_render`  | 跳过指定文件的渲染，您可使用 [glob 表达式](https://github.com/isaacs/node-glob)来匹配路径。|                  |
	
>    提示：如果您刚刚开始接触 Hexo，通常没有必要修改这一部分的值。

### Writing ###
| 参数                | 描述                                 | 默认值     |
| ------------------- | ------------------------------------ |            |
| `new_post_name`     | 新文章的文件名称                     | 	:title.md |
| `default_layout`    | 预设布局                             | 	post      |
| `auto_spacing`      | 在中文和英文之间加入空格             | 	false     |
| `titlecase`         | 把标题转换为 title case              | 	false     |
| `external_link`     | 在新标签中打开链接                   |	true      |
| `filename_case`     | 把文件名称转换为 (1) 小写或 (2) 大写 | 	0         |
| `render_drafts`     | 显示草稿                             |	false     |
| `post_asset_folder` | 启动 Asset 文件夹                    | 	false     |
| `relative_link`     | 把链接改为与根目录的相对位址         | 	false     |
| `future`            | 显示未来的文章                       |	true      |
| `highlight`         | 代码块的设置 	                     |            |

>    默认情况下，Hexo 生成的超链接都是绝对地址。例如，如果您的网站域名为 `example.com`，您有一篇文章名为 `hello`，那么绝对链接可能像这样：`http://example.com/hello.html`，它是绝对于域名的。
>    相对链接像这样：`/hello.html，也就是说，无论用什么域名访问该站点，都没有关系，这在进行反向代理时可能用到。通常情况下，建议使用绝对地址。

### Category & Tag ###
| 参数               | 描述     | 默认值        |
| ------------------ | -------- |               |
| `default_category` | 默认分类 | uncategorized |
| `category_map`     | 分类别名 | 	        |
| `tag_map`          | 标签别名 | 	        |

### Date / Time format ###
Hexo 使用 [Moment.js](http://momentjs.com/) 来解析和显示时间。

| 参数          | 描述     | 默认值     |
| ------------- | -------- |            |
| `date_format` | 日期格式 | YYYY-MM-DD |
| `time_format` | 时间格式 | H:mm:ss    |

### Pagination ###
| 参数             | 描述                                | 默认值 |
| ---------------- | ----------------------------------- |        |
| `per_page`       | 每页显示的文章量 (0 = 关闭分页功能) | 10     |
| `pagination_dir` | 分页目录                            | page   |

### Extensions ###
| 参数     | 描述                                  |
| -------- | ------------------------------------- |
| `theme`  | 当前主题名称。值为 `false` 时禁用主题 |
| `deploy` | 部署部分的设置                        | 	 

## 更改 Hexo 主题 ##
官方主题库：[https://hexo.io/themes/](https://hexo.io/themes/)

Hexo 主题非常多，推荐使用 [`NexT`](https://github.com/iissnan/theme-next-docs) 为主题，请阅读 NexT 的官方文档(http://theme-next.iissnan.com/) ，5 分钟快速安装。

### 下载主题 ###
在终端窗口下，定位到 Hexo 站点主目录下。使用 `Git` checkout 代码：
``` bash
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用主题 ###
与所有 Hexo 主题启用的模式一样。当克隆/下载完成后，打开站点主目录下的 `_config.yml` 文件，找到 `theme` 字段，并将其值更改为 `next`。
``` yml
theme: next
```
到此，NexT 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前， 我们最好使用 `hexo clean` 来清除 Hexo 的缓存。

### 验证主题 ###
首先启动 Hexo 本地站点，并开启调试模式（即加上 `--debug`），整个命令是 `hexo s --debug`。 在服务启动的过程，注意观察命令行输出是否有任何异常信息，如果你碰到问题，这些信息将帮助他人更好的定位错误。 当命令行输出中提示出：
``` bash
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```
此时即可使用浏览器访问 [http://localhost:4000/](http://localhost:4000/) ，检查站点是否正确运行。

>  当你看到站点的外观与下图所示类似时即说明你已成功安装 `NexT` 主题。这是 NexT 默认的 Scheme —— Muse。更的信息请查看 [NexT 文档](http://theme-next.iissnan.com/getting-started.html)

本人的博客是使用的 [Matery](https://github.com/blinkfox/hexo-theme-matery) 主题。

## 安装 Hexo 插件 ##
### 代码高亮 ###
由于 Hexo 自带的代码高亮主题显示不好看，所以主题中使用到了 [hexo-prism-plugin](https://github.com/ele828/hexo-prism-plugin) 的 Hexo 插件来做代码高亮，安装命令如下：
```bash
npm i -S hexo-prism-plugin
```

然后，修改 Hexo 根目录下`_config.yml`文件中`highlight.enable`的值为`false`，并新增`prism`插件相关的配置，主要配置如下：
```yml
highlight:
  enable: false

prism_plugin:
  mode: 'preprocess'    # realtime/preprocess
  theme: 'tomorrow'
  line_number: false    # default false
  custom_css:
```

### 站内搜索 ###
安装 [hexo-generator-search](https://github.com/wzpan/hexo-generator-search) 在 `Hexo` 中实现搜索功能，在站点的根目录下执行以下命令：
```bash
npm install hexo-generator-search --save
```

在 Hexo 站点根目录下的`_config.yml`文件中，新增以下的配置项：
```yml
search:
  path: search.xml
  field: post
  content: true
```

### RSS 订阅(可选) ###
安装 [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed) 在 `Hexo` 中实现 `RSS 订阅`功能，安装命令如下：
```bash
npm install hexo-generator-feed --save
```

在 Hexo 站点根目录下的 `_config.yml` 文件中，新增以下的配置项：
```yml
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: ' '
  order_by: -date
```

### 中文链接转拼音(可选) ###
如果你的文章名称是中文的，那么 Hexo 默认生成的永久链接也会有中文，这样不利于 `SEO`，且 `gitment` 评论对中文链接也不支持。我们可以用[hexo-permalink-pinyin](https://github.com/viko16/hexo-permalink-pinyin) Hexo 插件使在生成文章时生成中文拼音的永久链接。

安装命令如下：
```bash
npm i hexo-permalink-pinyin --save
```

在 Hexo 根目录下的`_config.yml`文件中，新增以下的配置项：
```yml
permalink_pinyin:
  enable: true
  separator: '-' # default: '-'
```

> **注**：除了此插件外，[hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink) 插件也可以生成非中文的链接。

## 部署 Hexo 到 GitHub ##
Hexo 提供了快速方便的一键部署功能，让您只需一条命令就能将网站部署到服务器上。
```bash
$ hexo deploy
or
$ hexo d
```

### 安装 Git ###
安装 [hexo-deployer-git](hexo-deployer-git)。
```bash
$ npm install hexo-deployer-git --save
```

### 修改配置 ###
在 Hexo 站点根目录下的 `_config.yml` 文件中，修改以下的配置项：
```yml
deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]
```

| 参数      | 描述                                                                                              |
| --------- | ------------------------------------------------------------------------------------------------- |
| `repo`    | 仓库(Repository)地址                                                                              |
| `branch`  | 分支名称。如果您使用的是 GitHub 或 GitCafe 的话，程序会尝试自动检测。                             |
| `message` | 自定义提交信息(默认为 Site updated: &#123;&#123; now&#40;'YYYY-MM-DD HH:mm:ss'&#41; &#125;&#125;) |

### 部署 ###
配置完成后，您可执行下列的其中一个命令，让 Hexo 在生成完毕后自动部署网站，两个命令的作用是相同的。
```bash
$ hexo generate --deploy
or
$ hexo deploy --generate
```

上面的命令等同于：
```bash
$ hexo generate
$ hexo deploy
```
> **注意：** 在通过上面的命令部署之前，最好通过 `hexo clean` 命令先清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

> Hexo 除了可以部署到 `GitHub` 上之外，还可以部署到 `Heroku`、`Rsync`、`OpenShift`、`FTPSync` 等平台。
> Hexo 生成的所有文件都放在 `public` 文件夹中，您可以将它们复制到您喜欢的地方。
> 不同平台的具体配置和部署信息可查看 [Deployment](https://hexo.io/docs/deployment)

## 相关教程 ##
[Git 教程](https://git-scm.com/book/)
[Hexo 教程](https://hexo.io/docs/)
[Hexo 主题](https://hexo.io/themes/)
