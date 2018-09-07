---
title: 基于hexo+github免费搭建个人博客
date: 2018-03-04 10:47:33
categories: 
- 其他
- hexo
tags:
- hexo
---

# 一、Hexo简介

Hexo是一款基于Node.js的静态博客框架，依赖少易于安装使用，可以方便的生成静态网页托管在GitHub和Heroku上，是搭建博客的首选框架。这里我们选用的是GitHub。Hexo同时也是GitHub上的开源项目，参见：[hexojs/hexo](https://github.com/hexojs/hexo) 如果想要更加全面的了解Hexo，可以到其官网 [Hexo](https://hexo.io/) 了解更多的细节，因为Hexo的创建者是台湾人，对中文的支持很友好，可以选择中文进行查看。

# 搭建步骤

# 三、环境配置

## 1. 本机环境配置 

1. 安装Node.js

	下载[Node.js](https://nodejs.org/en/download/),注意安装Node.js会包含环境变量及npm的安装，安装后，检测Node.js是否安装成功，在命令行中输入 node -v. 检测npm是否安装成功，在命令行中输入npm -v
	
2. 安装Hexo

- Hexo就是我们的个人博客网站的框架， 这里需要自己在电脑常里创建一个文件夹，可以命名为Blog，Hexo框架与以后你自己发布的网页都在这个文件夹中。创建好后，进入文件夹中,使用npm命令安装Hexo，输入：
	
	```bash
	npm install -g hexo #等待一会就会完成下载安装。
	hexo init #该命令会在目标文件夹内建立网站所需要的所有文件
	npm install #安装依赖包
	```
	
- 到这里本地博客就搭建好了。执行以下命令（在你博客的对应文件夹路径下）:
	
	```bash
	hexo generate # Or hexo g
	hexo server   # Or hexo s
	```

- 在浏览器输入http://localhost:4000/ 就可以进行查看了。当然这个博客是本地的，别人是无法访问的，之后我们需要部署到GitHub上。常用的Hexo 命令:

	```bash
	npm install hexo -g #安装Hexo
	npm update hexo -g #升级
	hexo init #初始化博客
	# 命令简写
	hexo n "我的博客" == hexo new "我的博客" #新建文章
	hexo g == hexo generate #生成
	hexo s == hexo server #启动服务预览
	hexo d == hexo deploy #部署
	hexo server #Hexo会监视文件变动并自动更新，无须重启服务器
	hexo server -s #静态模式
	hexo server -p 5000 #更改端口
	hexo server -i 192.168.1.1 #自定义 IP
	hexo clean #清除缓存，若是网页正常情况下可以忽略这条命令
	```

## 2. git环境配置

1. 注册Github账号并新建仓库

	- 注册过程就不多说了，注册完成之后需要新建一个仓库。需要注意的是新创建的仓库的名字，必须是username.github.io。例如我的username是XXX，那么新创建的仓库的名字便是XXX.github.io。
	
2. 配置SSH Key

- 这一步不是必须的，配置SSH Key的话之后每次更新博客就不用都输入用户名和密码，可以方便一些。
	
	(1)检查本机上是否已经存在SSH Key。打开终端，输入如下命令：
	
	```bash
	cd .ssh
	ls -la
	```
	检查终端输出的文件列表中是否已经存在id_rsa.pub 或 id_dsa.pub 文件，如果文件已经存在，则直接进入第三步。
	
	(2)创建一个SSH Key。在终端输入如下命令:
	
	```bash
	ssh-keygen -t rsa -C "your_email@example.com"
	```
		按下回车，让你输入文件名，直接回车会创建使用默认文件名的文件(推荐使用默认文件名)，然后会提示你输入两次密码，可以为空。
	
	(3)添加SSH Key到Github
	
		如果你没有指定文件名（也就是使用的默认文件名），那么你的.ssh文件夹下，应该有一个id_rsa.pub文件了，打开该文件，复制里面的文本。然后登录Github，点击右上角头像右边的三角图标，点击Settings，然后在左边菜单栏点击SSH and GPG keys，点击New SSH key，Title 随便填一个，在Key栏填入你复制的内容，点击Add SSH key，就添加成功了。

	(4)检验SSH Key是否配置成功。在终端输入如下命令:
	
	```bash
	ssh -T git@github.com
	```
	
		如果出现:
	
	```bash
	Are you sure you want to continue connecting (yes/no)? 
	```
	
		请输入yes再按回车。如果最后出现:
	
	```bash
	Hi username! You've successfully authenticated, but GitHub does not provide shell access.
	```

		就说明你的SSH Key配置成功了。

## 3. 同步本地博客到Github

1. 上面只是在本地预览，接下来要做的就是就是推送网站，也就是发布网站，让我们的网站可以被更多的人访问。在设置之前，需要解释一个概念，在blog根目录里的_config.yml文件称为站点配置文件.
	
2. 我们的Hexo与GitHub关联起来，打开站点的配置文件_config.yml，翻到最后修改为：
	
	```
	deploy: 
	type: git
	repo: 这里填入你之前在GitHub上创建仓库的完整路径，记得加上 .git
	branch: master参考如下：
	```
	
- 例子：
	
	```bash
	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	type: git
	repo: https://github.com/xiaoqiangteng/xiaoqiangteng.github.io.git
	brabch: master
   ```
  
- 保存站点配置文件。其实就是给hexo d 这个命令做相应的配置，让hexo知道你要把blog部署在哪个位置，很显然，我们部署在我们GitHub的仓库里。最后安装Git部署插件，输入命令：

	```bash
  	npm install hexo-deployer-git --save
  	```
  
3. 这时，我们分别输入三条命令：
  
  	```bash
	hexo clean 
	hexo g 
	hexo d
	```
	
	其实第三条的 hexo d 就是部署网站命令，d是deploy的缩写。完成后，打开浏览器，在地址栏输入你的放置个人网站的仓库路径

4. 发布新的博客

	- 既然博客已经搭建好了，那么不发几篇博文有就没有意义了，使用下面的命令来新建一篇叫做”brightloong”的文章。
	
	```bash
	hexo new 'brightloong'
	```
	
	- 命令执行之后，你会在你文件博客根目录的source/_post目录下找到你刚刚新建的md后缀的文件，hexo博客是使用markdown语法来书写的，如果不熟悉markdown语法可以快速的看一下[markdown](https://www.appinn.com/markdown/)语法说明.
	
	> 注意：在冒号后面一定要加上一个空格，否则在生成静态文件的时候会报错，并且也不能将其成功推送到github。
	
	```md
	---
	title: brightloong #文章标题
	date: 2017-02-24 12:03:12 #创建时间
	tags: #文章标签，如果有多个标签可以使用[1,2,3]的形式，还有其他形式自己摸索吧
	---
	#这之后是正文
	```
	
	- 文章编写好之后，只用以下命令生成静态文件并推送到github上，执行完成后打开自己的博客页面，是不是发现刚刚编写的文章出现了；如果你想删除某一篇文章，那么在source/_post目录下找到对应的文章将其删除后，同样执行一下命令就OK了。
	
5. 站点配置文件_config.yml

	- 站点配置文件_config.yml是在你博客保存目录的根目录下，注意将它与主题配置文件进行区分，我使用的主题是Next主题。下面我先介绍下站点配置文件，我将一些主要的配置做了注释，如果你想了解更多的配置的含义和作用，请访问[Hexo官方教程](https://hexo.io/docs/configuration.html)查看。
	
	```yml
	# Hexo Configuration
	## Docs: https://hexo.io/docs/configuration.html
	## Source: https://github.com/hexojs/hexo/
	# Site
	title: BrightLoong's Blog #博客的标题
	subtitle: #子标题
	description: Remember what should be remembered, and forget what should be forgotten.Alter what is changeable, and accept what is mutable. #博客描述，可以是一段你喜欢的话，也可以是你博客的描述，只要你开心就好。
	author: BrightLoong #作者
	language: zh-Hans #语言（我使用的是简体中文）
	timezone: #时区（默认使用电脑时间）
	##之下的保持默认就好，没有什么需要更改的
	# URL
	## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' #and root as '/child/'
	url: https://brightloong.github.io
	root: /
	permalink: :year/:month/:day/:title/
	permalink_defaults:
	# Directory
	source_dir: source #source目录
	public_dir: public
	tag_dir: tags #标签目录
	archive_dir: archives 
	category_dir: categories #分类目录
	code_dir: downloads/code
	i18n_dir: :lang
	skip_render: static/** #注意这个属性（跳过渲染），你暂时不用配置，我之后会讲到，这个也是我遇到的坑
	##之下的保持默认就好，没有什么需要更改的
	# Writing
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
	  auto_detect: false
	  tab_replace:
	# Category & Tag
	default_category: uncategorized
	category_map:
	tag_map:
	# Date / Time format
	## Hexo uses Moment.js to parse and display date
	## You can customize the date format as defined in
	## http://momentjs.com/docs/#/displaying/format/
	date_format: YYYY-MM-DD
	time_format: HH:mm:ss
	# Pagination
	## Set per_page to 0 to disable pagination
	per_page: 10
	pagination_dir: page
	# Extensions
	## Plugins: https://hexo.io/plugins/
	## Themes: https://hexo.io/themes/
	theme: next #你设置的主题，接下来我会说到这个
	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	  type: git
	  repository: https://github.com/BrightLoong/BrightLoong.github.io.git
	  branch: master
  ```
  
## 设置专属域名
  
博客搭建好后，我们可以通过之前设置好的GitHub仓库地址来访问，比如：http://XXX.github.io，而且GitHub是免费替我们托管的的，如果我们想要设置自己的专属的域名，我们可以去阿里云购买域名，我们点击添加记录，设置主机记录为@，类型为A，到IP 192.30.252.153（固定值）。按照如上设置完成之后， 可能不会立即生效，等个几分钟，在./source目录下新建文件CNAME（没有后缀名），文件中写上我们要绑定的域名，例如: XXX.com.部署到GitHub上。这时就可以通过http://XXX.com访问.
  
# 四、Hexo配置

## 主题设置

搭建自己的博客，最吸引人的莫过于那千变万化的主题了，大家可以在[Hexo官网](https://hexo.io/themes/)上看到无数漂亮、大方、简洁的主题。本人使用的是简洁的[Next主题](https://github.com/iissnan/hexo-theme-next)，你可以选择你喜欢的下载下来，将其解压放入themes目录中，比如我的目录是.\themes，然后修改我在上面提到的站点配置文件中的theme属性，为你刚刚放入themes目录中文件的名字（最好是对解压文件修改一个名字，否则名字可能会比较长，我把我下载下来的主题改文了next）,做完这些之后并不代表你完成了，你还需要参考你所下载的主题所说的配置步骤进行相关的配置，由于不同的主题配置过程也尽不相同，大家根据自己下载的主题去配置，我在这里只说我使用的Next主题如何配置。

```yml
theme: next
```

>注意：从下面开始所说的都是Next主题的相关配置。

如果你使用的和我一样，也是Next的主题，那么你最好还是看官方提供[Next使用文档](http://theme-next.iissnan.com/)，并且文档是中文版的,我也仅仅是讲一些容易被忽略的配置，以及我使用的配置，以及在使用过程中遇到的问题;至于如何更换头像，添加分类和标签页面、切换主题样式（Next主题包含3中样式）之类的，大家还是照着官方的做更好。

1. 配置网站图标 

	如何让网站前能显示自己想要的图标，我当时也是找了很久，最后发现是在主题配置文件（我的是F:\myblog\themes\next_config.yml）的最前面，有一个favicon属性，我把一个名字叫favicon.ico的图片放到了F:\myblog\source下，然后配置如下：
	
	```yml
	favicon: /favicon.ico
	```
	
2. 首页显示阅读全文按钮

	首页的文章是不是默认展开了，显示出了整篇文章，怎么才能显示出如下的阅读全文的按钮。在主题配置文件中找到auto_excerpt属性进行配置:
	
	```yml
	auto_excerpt:
 	 	enable: true #改写为true
	   length: 150 #默认展示的高度
	```
	
	你也可以在自己的博文中添加\<!--more-->来决定在首页展示到什么位置（我就喜欢用这种方式），这个标签后的内容就不会展示到首页啦。
	
## 修改文章内链接文本样式

修改文件 themes\next\source\css\_common\components\post\post.styl，在末尾添加如下css样式，：

```css
// 文章内链接文本样式
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  border-bottom: 1px solid #0593d3;
  &:hover {
    color: #fc6423;
    border-bottom: none;
    border-bottom: 1px solid #fc6423;
  }
}
```

其中选择.post-body 是为了不影响标题，选择 p 是为了不影响首页“阅读全文”的显示样式,颜色可以自己定义。

## 在每篇文章末尾统一添加“本文结束”标记

