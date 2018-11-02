---
title: Hello Hexo
date: 2018-06-02 11:33:45
categories: 前端
tags:
  - Hexo
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## 什么是 Hexo? ##
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。[视频教程](https://www.youtube.com/watch?v=bCj0iVVqkSg)

## 安装 ##
安装 Hexo 只需几分钟时间，若您在安装过程中遇到问题或无法找到解决方式，请 [提交问题](https://github.com/hexojs/hexo/issues)，我会尽力解决您的问题。

### 安装前提 ###
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
 - [Node.js](https://nodejs.org/)
 - [Git](https://git-scm.com/)

如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。
``` bash
$ npm install -g hexo-cli
```
如果您的电脑中尚未安装所需要的程序，请根据以下安装指示完成安装。
>    **Mac 用户**
>
>    您在编译时可能会遇到问题，请先到 App Store 安装 Xcode，Xcode 完成后，启动并进入 **Preferences -> Download -> Command Line Tools -> Install** 安装命令行工具。

### 安装 Git ###
 - Windows：下载并安装 [git](https://git-scm.com/download/win)。
 - Mac：使用 [Homebrew](http://mxcl.github.com/homebrew/), [MacPorts](http://www.macports.org/) ：brew install git;或下载 [安装程序](http://sourceforge.net/projects/git-osx-installer/) 安装。
 - Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
 - Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`

>    **Windows 用户**
>
>    由于众所周知的原因，从上面的链接下载 git for windows 最好挂上一个代理，否则下载速度十分缓慢。也可以参考 [这个页面](https://github.com/waylau/git-for-win)，收录了存储于百度云的下载地址。

### 安装 Node.js ###
安装 Node.js 的最佳方式是使用 [nvm](https://github.com/creationix/nvm)。

cURL:
``` bash
$ curl https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

Wget:
``` bash
$ wget -qO- https://raw.github.com/creationix/nvm/v0.33.11/install.sh | sh
```

安装完成后，重启终端并执行下列命令即可安装 Node.js。
``` bash
$ nvm install stable
```

或者您也可以下载 [安装程序](http://nodejs.org/) 来安装。
>    **Windows 用户**
>
>    对于windows用户来说，建议使用安装程序进行安装。安装时，请勾选 `Add to PATH` 选项。
>    另外，您也可以使用 `Git Bash`，这是 git for windows 自带的一组程序，提供了 Linux 风格的 shell，在该环境下，您可以直接用上面提到的命令来安装 Node.js。
>    打开它的方法很简单，在任意位置单击右键，选择 “Git Bash Here” 即可。由于 Hexo 的很多操作都涉及到命令行，您可以考虑始终使用 `Git Bash` 来进行操作。

### 安装 Hexo ###
所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。
``` bash
$ npm install -g hexo-cli
```

## 建站 ##
安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。[视频教程](https://www.youtube.com/watch?v=iJv4N5EdKJ4)
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

新建完成后，指定文件夹的目录如下：
``` 
    .
    ├── _config.yml
    ├── package.json
    ├── scaffolds
    ├── source
    |   ├── _drafts
    |   └── _posts
    └── themes
```

目录和文件的描述如下：
 - **_config.yml：** 网站的 [配置](https://hexo.io/docs/configuration) 信息，您可以在此配置大部分的参数。
 - **package.json：** 应用程序的信息。[EJS](http://embeddedjs.com/), [Stylus](http://learnboost.github.io/stylus/) 和 [Markdown](http://daringfireball.net/projects/markdown/) renderer 已默认安装，您可以自由移除。
 - **scaffolds：** [模版](https://hexo.io/docs/writing#Scaffolds) 文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。
 - **source：** 资源文件夹是存放用户资源的地方。除 _posts 文件夹之外，开头命名为 _ (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去。
 - **themes：** [主题](https://hexo.io/docs/themes) 文件夹。Hexo 会根据主题来生成静态页面。

## 配置 ##
您可以在 `_config.yml` 中修改大部份的配置。
### 网站 ###
| 参数          | 描述                                                                                            |
| ------------- | ----------------------------------------------------------------------------------------------- |
| `title`       | 网站标题                                                                                        |
| `subtitle`    | 网站副标题                                                                                      |
| `description` | 网站描述                                                                                        |
| `author`      | 您的名字                                                                                        |
| `language`    | 网站使用的语言                                                                                  |
| `timezone`    | 网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：`America/New_York`、 `Japan` 和 `UTC` 。 |

其中，`description` 主要用于SEO，告诉搜索引擎一个关于您站点的简单描述，通常建议在其中包含您网站的关键词。`author` 参数用于主题显示文章的作者。

### 网址 ###
| 参数                 | 描述                                                    | 默认值                      |
| -------------------- | ------------------------------------------------------- |                             |
| `url`                | 网址                                                    |                             |
| `root`               | 网站根目录                                              |                             |
| `permalink`          | 文章的 [永久链接](https://hexo.io/docs/permalinks) 格式 | `:year/:month/:day/:title/` |
| `permalink_defaults` | 永久链接中各部分的默认值                                |                             |

>    网站存放在子目录
>
>    如果您的网站存放在子目录中，例如 `http://yoursite.com/blog`，则请将您的 `url` 设为 `http://yoursite.com/blog` 并把 `root` 设为 `/blog/`。
 	
### 目录 ###
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
	
>    提示
>
>    如果您刚刚开始接触 Hexo，通常没有必要修改这一部分的值。

### 文章 ###
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

>    相对地址
>
>    默认情况下，Hexo 生成的超链接都是绝对地址。例如，如果您的网站域名为 `example.com`，您有一篇文章名为 `hello`，那么绝对链接可能像这样：`http://example.com/hello.html`，它是绝对于域名的。
>    相对链接像这样：`/hello.html，也就是说，无论用什么域名访问该站点，都没有关系，这在进行反向代理时可能用到。通常情况下，建议使用绝对地址。

### 分类 & 标签 ###
| 参数               | 描述     | 默认值        |
| ------------------ | -------- |               |
| `default_category` | 默认分类 | uncategorized |
| `category_map`     | 分类别名 | 	        |
| `tag_map`          | 标签别名 | 	        |

### 日期 / 时间格式 ###
Hexo 使用 [Moment.js](http://momentjs.com/) 来解析和显示时间。

| 参数          | 描述     | 默认值     |
| ------------- | -------- |            |
| `date_format` | 日期格式 | YYYY-MM-DD |
| `time_format` | 时间格式 | H:mm:ss    |

### 分页 ###
| 参数             | 描述                                | 默认值 |
| ---------------- | ----------------------------------- |        |
| `per_page`       | 每页显示的文章量 (0 = 关闭分页功能) | 10     |
| `pagination_dir` | 分页目录                            | page   |

### 扩展 ###
| 参数     | 描述                                  |
| -------- | ------------------------------------- |
| `theme`  | 当前主题名称。值为 `false` 时禁用主题 |
| `deploy` | 部署部分的设置                        | 	 	

## 命令 ##
### init ###
``` bash
$ hexo init [folder]
```
新建一个网站。如果没有设置 `folder`，Hexo 默认在目前的文件夹建立网站。

### new ###
``` bash
$ hexo new [layout] <title>
```
新建一篇文章。如果没有设置 `layout` 的话，默认使用 [\_config.yml](https://hexo.io/docs/configuration) 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。查看[Writing](https://hexo.io/docs/writing.html)

### server ###
``` bash
$ hexo server
```
启动服务器。默认情况下，访问网址为： [http://localhost:4000/](http://localhost:4000/) 。查看[Server](https://hexo.io/docs/server.html)

| 选项               | 描述                           |
| ------------------ | ------------------------------ |
| `-p`, `--port`     | 重设端口                       |
| `-s`, `--static`   | 只使用静态文件                 |
| `-l`, `--log`      | 启动日记记录，使用覆盖记录格式 |


### generate ###
``` bash
$ hexo generate
or
$ hexo g
```
生成静态文件。

| 选项             | 描述                   |
| ---------------- | ---------------------- |
| `-d`, `--deploy` | 文件生成后立即部署网站 |
| `-w`, `--watch`  | 监视文件变动           |

查看[Generating](https://hexo.io/docs/generating.html)

### deploy ###
``` bash
$ hexo deploy
or
$ hexo d
```
部署网站。查看[Deployment](https://hexo.io/docs/deployment.html)
| 选项               | 描述                     |
| ------------------ | ------------------------ |
| `-g`, `--generate` | 部署之前预先生成静态文件 |

### migrate ###
``` bash
$ hexo migrate <type>
```
从其他博客系统 [迁移内容](https://hexo.io/zh-cn/docs/migration)。

### clean ###
``` bash
$ hexo clean
```
清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。
在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。

### list ###
``` bash
$ hexo list <type>
```
列出网站资料。

### version ###
``` bash
$ hexo version
```
显示 Hexo 版本。

### Options ###
#### Safe mode ####
``` bash
$ hexo --safe
```
在安全模式下，不会载入插件和脚本。当您在安装新插件遭遇问题时，可以尝试以安全模式重新执行。

#### Debug mode ####
``` bash
$ hexo --debug
```
在终端中显示调试信息并记录到 `debug.log`。当您碰到问题时，可以尝试用调试模式重新执行一次，并 [提交调试信息到 GitHub](https://github.com/hexojs/hexo/issues/new)。

#### Silent mode ####
``` bash
$ hexo --silent
```
隐藏终端信息。
