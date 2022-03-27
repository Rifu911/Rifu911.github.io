---
layout: posts
date: 2020-02-27 13:17:18
title: 使用Gitee 和 hexo 搭建个人博客
categories: 教程
tags: [Gitee, hexo, 个人博客]
---



### 一、搭建步骤

##### 1）安装node.js

这个没什么好说的，直接官网下载，next,next

<!-- more -->

##### )2 安装git

同样简单，然后就是配置环境等，如果不会可以参考我的另一篇学习笔记部分

https://rifu520.gitee.io/2018/10/10/Git/

##### 3) 安装hexo

> npm install -g hexo-cli

##### 4) 安装好hexo 之后，找个空文件夹，执行以下命令

> hexo init
>
> npm install

这里安装时间比较长，耐心等待

### 二、关于目录

##### 1) 安装完hexo 之后，它的目录结构是这样的

##### ![微信截图_20181012110406](C:\Users\Lifu\Desktop\微信截图_20181012110406.png)

解释一下各个文件的作用

##### config.yml

博客的配置文件，博客的名称、关键词、作者、语言、博客主题...设置都在里面。

##### package.json

应用程序信息，新添加的插件内容也会出现在这里面，我们可以不修改这里的内容。

##### scaffolds

scaffolds就是脚手架的意思，这里放了三个模板文件，分别是新添加博客文章（posts）、新添加博客页（page）和新添加草稿（draft）的目标样式。

这部分可以修改的内容是，我们可以在模板上添加比如categories等自定义内容

##### source

source是放置我们博客内容的地方，里面初始只有两个文件夹，一个是drafts（草稿），一个posts（文章），但之后我们通过命令新建tags（标签）还有categories（分类）页后，这里会相应地增加文件夹。

##### themes

放置主题文件包的地方。Hexo会根据这个文件来生成静态页面。

初始状态下只有landscape一个文件夹，后续我们可以添加自己喜欢的。

### 三、指令的使用

##### `init`

新建一个网站。

```
hexo init <folder>
```

##### `new`

新建文章或页面。

```
hexo new <layout> "title"
```

这里的`<layout>`对应我们要添加的内容，如果是`posts`就是添加新的文章，如果是`page`就是添加新的页面。

默认是添加`posts`。

然后我们就可以在对应的posts或drafts文件夹里找到我们新建的文件，然后在文件里用Markdown的格式来写作了。

##### `generate`

生成静态页面

```
hexo generate
```

也可以简写成

```
hexo g
```

##### `deploy`

将内容部署到网站

```
hexo deploy
```

也可以简写成

```
hexo -d
```

##### `publish`

发布内容，实际上是将内容从drafts（草稿）文件夹移到posts（文章）文件夹。

```
hexo publish <layout> <filename>
```

##### `server`

启动服务器，默认情况下，访问网站为`http://localhost:4000/`

```
hexo server
```

也可以简写成

```
hexo s
```



### 四、拓展

##### 1）添加分类

新建分类页面

```
hexo new page categories
```

给分类页面添加类型

我们在source文件夹中的categories文件夹下找到index.md文件，并在它的头部加上type属性。

```
---
title: 文章分类
date: 2017-05-27 13:47:40
type: "categories"   #这部分是新添加的
---
```

给模板添加分类属性

现在我们打开scarffolds文件夹里的post.md文件，给它的头部加上`categories:`，这样我们创建的所有新的文章都会自带这个属性，我们只需要往里填分类，就可以自动在网站上形成分类了。

```
title: {{ title }}
date: {{ date }}
categories:
tags:
```

给文章添加分类

现在我们可以找到一篇文章，然后尝试给它添加分类

```
layout: posts
title: 写给小白的express学习笔记1： express-static文件静态管理
date: 2018-06-07 00:38:36
categories: 学习笔记
tags: [node.js, express]
```

##### 2）创建标签

创建"标签"页的方式和创建“分类”一样。

- 新建“标签”页面

  ```
  hexo new page tags
  ```

- 给标签页面添加类型

  我们在source文件夹中的tags文件夹下找到index.md文件，并在它的头部加上type属性。

  ```
  title: tags
  date: 2018-08-06 22:48:29
  type: "tags" #新添加的内容
  ```

- 给文章添加标签

  有两种写法都可以，第一种是类似数组的写法，把标签放在中括号`[]`里，用英文逗号隔开

  ```
  layout: posts
  title: gitee 和 hexo 搭建个人博客
  date: 2018-06-07 00:38:36
  categories: 学习笔记
  tags: [node.js, express]
  ```

  第二种写法是用`-`短划线列出来

  ```
  layout: posts
  title: 写给小白的express学习笔记1： express-static文件静态管理
  date: 2018-06-07 00:38:36
  categories: 学习笔记
  tags: 
  - node.js
  - express
  ```



# 使用主题

hexo有很多开源的主题，我选了[NexT](https://github.com/iissnan/hexo-theme-next)，开始只是觉得很简洁清爽，后来发现它的功能挺齐全的，提前解决了很多搭建过程中会遇到的问题。这里强烈推荐一下。

#### 安装

我是用的`git clone`的方法，文档中还有其他方法

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

#### 设置主题

在**hexo根目录**下的配置文件config.yml里设置主题

```
theme: next
```

#### 配置主题

接下来我们就可以来按需配置主题内容了，所有内容都在**themes/next**文件夹下的config.yml文件里修改。

官方文档里写的是有些配置需要将一部分代码添加到配置文件中，但其实不用，我们逐行看配置文件就会发现，有很多功能都已经放在配置文件里了，只是注释掉了，我们只需要取消注释，把需要的相关信息补全即可使用

##### 菜单栏 `menu` 

原生菜单栏有主页、关于、分类、标签等数个选项，但是在配置文件中是注释掉的状态，这里我们自行修改注释就行

```
menu:
  home: / || home
  # about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  # schedule: /schedule/ || calendar
  # sitemap: /sitemap.xml || sitemap
  # commonweal: /404/ || heartbeat
```

注意点：

- 如果事先没有通过`hexo new page <pageName>`来创建页面的话，即使在配置文件中取消注释，页面也没法显示
- 我们也可以添加自己想要添加的页面，不用局限在配置文件里提供的选择里
-  `||`后面是fontAwesome里的文件对应的名称
-  `menu_icons`记得选`enable: true`（默认应该是`true`）



```
menu:
  home: / || home
  # about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  # schedule: /schedule/ || calendar
  # sitemap: /sitemap.xml || sitemap
  # commonweal: /404/ || heartbeat
```

##### 主题风格 `schemes` 

主题提供了4个，我们把想要选择的取消注释，其他三个保持注释掉的状态即可。

- `Muse`



  ![img](https:////upload-images.jianshu.io/upload_images/9240001-5e7193faf3720112.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

  image-20180809164700600

- Mist



  ![img](https:////upload-images.jianshu.io/upload_images/9240001-dbd774ea0be0fe87.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

  image-20180809164749052

- Pisces



  ![img](https:////upload-images.jianshu.io/upload_images/9240001-327385996d44bb02.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

  image-20180809164925685

- Gemini



  ![img](https:////upload-images.jianshu.io/upload_images/9240001-0e58f7644c380210.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

  image-20180809165023401

选择主题后也可以自定义，不过我还没摸清楚有哪些地方可以自定义，等弄清楚了我再来更新。

##### 底部建站时间和图标修改

修改主题的配置文件：

```
footer:
  # Specify the date when the site was setup.
  # If not defined, current year will be used.
  since: 2018

  # Icon between year and copyright info.
  icon: snowflake-o

  # If not defined, will be used `author` from Hexo main config.
  copyright:
  # -------------------------------------------------------------
  # Hexo link (Powered by Hexo).
  powered: false

  theme:
    # Theme & scheme info link (Theme - NexT.scheme).
    enable: false
    # Version info of NexT after scheme info (vX.X.X).
    # version: false
```

我在这部分做了这样几件事：

- 把用户的图标从小人`user`改成了雪花`snowflake-o` 
-  `copyright`留空，显示成页面`author`即我的名字
-  `powered: false`把hexo的授权图片取消了
-  `theme: enable:false` 把主题的内容也取消了

这样底部信息比较简单。

