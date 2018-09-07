---
title: Mac/Linux-Shell-批量重命名的方法总览
date: 2018-06-05 17:46:27
categories:
- 编程
- Shell
tags:
- 编程
- Shell
copyright:
---

# 删除所有的 .bak 后缀

```bash
rename 's/\.bak$//' *.bak
```

# 把 .JPEG 文件后缀修改为 .jpg

```bash
rename 's/\.JPEG$/\.jpg/' *.JPEG
```

# 把所有文件的文件名改为小写

```bash
rename 'y/A-Z/a-z/' *
```

# 将 abcd.jpg 重命名为 abcd_efg.jpg

```bash
for var in *.jpg; do mv "$var" "${var%.jpg}_efg.jpg"; done
```

# 将 abcd_efg.jpg 重命名为 abcd_lmn.jpg

```bash
for var in *.jpg; do mv "$var" "${var%_efg.jpg}_lmn.jpg"; done
```

# 把文件名中所有小写字母改为大写字母

```bash
for var in `ls`; do mv -f "$var" `echo "$var" |tr a-z A-Z`; done
```

# 把格式 \*\_?.jpg 的文件改为 \*\_0?.jpg

```bash
for var in `ls *_?.jpg`; do mv "$var" `echo "$var" |awk -F '_' '{print $1 "_0" $2}'`; done
```

# 把文件名的后四个字母变为 vzomik

```bash
for var in `ls`; do mv -f "$var" `echo "$var" |sed 's/....$/vzomik/'`; done
```

# 把.txt变成.txt_bak 的后缀

```bash
ls *.txt|xargs -n1 -i{} mv {} {}_bak
# xargs -n1 –i{} 类似for循环，-n1意思是一个一个对象的去处理，-i{} 把前面的对象使用{}取代，mv {} {}_bak 相当于 mv 1.txt 1.txt_bak

$ find ./*.txt -exec mv {} {}_bak \;  

# 这个命令中也是把{}作为前面find出来的文件的替代符，后面的”\”为”;”的脱意符，不然shell会把分号作为该行命令的结尾.
```

# 批量替换文件名

```bash
#!/bin/bash
oldext="JPG"
newext="jpg"
dir=$(eval pwd)
for file in $(ls $dir | grep .$oldext)
        do
        name=$(ls $file | cut -d. -f1)
        mv $file ${name}.$newext
        done
echo "change JPG=====>jpg done!"
```

1.变量oldext和newext分别指定旧的扩展名和新的扩展名。dir指定文件所在目录；

2.“ls $dir | grep .$oldext”用来在指定目录dir中获取扩展名为旧扩展名的所有文件；

3.在循环体内先利用cut命令将文件名中“.”之前的字符串剪切出来，并赋值给name变量；接着将当前的文件名重命名为新的文件名。

通过这个脚本，所有照片的扩展名都成功修改。为了使这个脚本更具有通用型，我们可以增加几条read命令实现脚本和用户之间的交互。改进版的脚本如下：

```bash
#!/bin/bash
read -p "old extension:" oldext
read -p "new extension:" newext
read -p "The directory:" dir
cd $dir
for file in $(ls $dir | grep .$oldext)
        do
        name=$(ls $file | cut -d. -f1)
        mv $file ${name}.$newext
        echo "$name.$oldext ====> $name.$newext"
        done
echo "all files has been modified."
```

修改后的脚本可以批量修改任意扩展名。

# 参考

- http://edsionte.com/techblog/archives/category/shell%e7%bc%96%e7%a8%8b
- https://my.oschina.net/musings/blog/380939