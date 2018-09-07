---
title: Hexo使用攻略-添加分类及标签
date: 2018-03-04 00:55:23
categories: 
- 其他
- hexo
tags:
- hexo
---


# Hexo使用攻略-添加分类及标签

## 创建“分类”选项

###生成“分类”页并添加tpye属性

打开命令行，进入博客所在文件夹。执行命令:
	
```
$ hexo new page categories
```
成功后会提示：
```
INFO  Created: ~/Documents/blog/source/categories/index.md
```
	
根据上面的路径，找到index.md这个文件，打开后默认内容是这样的：
	
```
---
title: 文章分类
date: 2017-05-27 13:47:40
type: "categories"
---
```
	
保存并关闭文件。

### 给文章添加“categories”属性

打开需要添加分类的文章，为其添加categories属性。下方的categories: web前端表示添加这篇文章到“web前端”这个分类。注意：hexo一篇文章只能属于一个分类，也就是说如果在“- web前端”下方添加“-xxx”，hexo不会产生两个分类，而是把分类嵌套（即该文章属于 “- web前端”下的 “-xxx ”分类）。

```
---
title: jQuery对表单的操作及更多应用
date: 2017-05-26 12:12:57
categories: 
- web前端
---
```

至此，成功给文章添加分类，点击首页的“分类”可以看到该分类下的所有文章。当然，只有添加了categories: xxx的文章才会被收录到首页的“分类”中。

注意：如果有启用多说 或者 Disqus 评论，默认页面也会带有评论。需要关闭的话，请添加字段 comments 并将值设置为 false，如：

```
title: 分类
date: 2014-12-22 12:39:04
type: "categories"
comments: false
---
```

或

### 设置分类列表

在我们编辑文章的时候，直接在categories:项填写属于哪个分类，但如果分类是中文的时候，路径也会包含中文。
比如分类我们设置的是：

```
categories: 编程
```

那在生成页面后，分类列表就会出现编程这个选项，他的访问路径是：

```
那在生成页面后，分类列表就会出现编程这个选项，他的访问路径是：
```

如果我们想要把路径名和分类名分别设置，需要怎么办呢？

打开根目录下的配置文件_config.yml，找到如下位置做更改：

```
# Category & Tag
default_category: uncategorized
category_map:
	编程: programming
	生活: life
	其他: other
tag_map:
```

在这里category_map:是设置分类的地方，每行一个分类，冒号前面是分类名称，后面是访问路径。

可以提前在这里设置好一些分类，当编辑的文章填写了对应的分类名时，就会自动的按照对应的路径来访问。

## 创建“标签”选项

### 生成“标签”页并添加tpye属性

打开命令行，进入博客所在文件夹。执行命令

```
$ hexo new page tags
```

成功后会提示：

```
INFO  Created: ~/Documents/blog/source/tags/index.md
```

根据上面的路径，找到index.md这个文件，打开后默认内容是这样的：

```
---
title: 标签
date: 2017-05-27 14:22:08
---
```

添加type: "tags"到内容中，添加后是这样的：

```
---
title: 文章分类
date: 2017-05-27 13:47:40
type: "tags"
---
```

保存并关闭文件。

### 给文章添加“tags”属性

打开需要添加标签的文章，为其添加tags属性。下方的tags:下方的\- jQuery \- 表格
\- 表单验证就是这篇文章的标签了

```
---
title: jQuery对表单的操作及更多应用
date: 2017-05-26 12:12:57
categories: 
- web前端
tags:
- jQuery
- 表格
- 表单验证
---
```

至此，成功给文章添加分类，点击首页的“标签”可以看到该标签下的所有文章。当然，只有添加了tags: xxx的文章才会被收录到首页的“标签”中。

## 新建页面的模板

打开scaffolds/post.md文件，在tages:上面加入categories:,保存后，重新执行hexo n 'name'命令，会发现新建的页面里有categories:项了。

scaffolds目录下，是新建页面的模板，执行新建命令时，是根据这里的模板页来完成的，所以可以在这里根据你自己的需求添加一些默认值。

## 菜单中添加链接

编辑主题的 _config.yml ，将 menu 中的 categories: /categories 注释去掉，如下:

```
menu:
  home: /
  categories: /categories
  archives: /archives
  tags: /tags
```

在主题配置文件中添加分类选项

在主题配置文件:

```
themes/_config.yml
```

中添加以下代码（#号后为注释内容）:

```
menu:
  主页: /
  所有文章: /archives
  技巧经验: /categories/技巧经验     # 博客首页展示文本： 访问路径/自定义归档名称
  资料总结: /categories/资料总结
```
