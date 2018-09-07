---
title: ILSVRC2015数据集转VOC2007数据集格式
date: 2018-04-05 15:48:06
categories:
- 编程
- 深度学习
- 数据集制作
tags:
- 编程
- 深度学习
copyright:
---

# 下载imageNet2015数据集

image-net官网下载：image-net.org

# VOC2007数据格式介绍

```bash
├── Annotations
│   ├── 000001.xml
│   └── 000002.xml
|   |__ ...
├── ImageSets
│   └── Main
│       ├── test.txt
│       ├── train.txt
│       └── val.txt
└── JPEGImages
    ├── 000001.jpg
    └── 000002.jpg
    |__ ...
```

JPEGImages文件夹中包含了PASCAL VOC所提供的所有的图片信息，包括了训练图片和测试图片。

Annotations文件夹中存放的是xml格式的标签文件，每一个xml文件都对应于JPEGImages文件夹中的一张图片。

xml文件的具体格式如下：（对于2007_000392.jpg）:

```xml

<annotation>
	<folder>VOC2012</folder>                           
	<filename>2007_000392.jpg</filename>                               //文件名
	<source>                                                           //图像来源（不重要）
		<database>The VOC2007 Database</database>
		<annotation>PASCAL VOC2007</annotation>
		<image>flickr</image>
	</source>
	<size>					                           //图像尺寸（长宽以及通道数）						
		<width>500</width>
		<height>332</height>
		<depth>3</depth>
	</size>
	<segmented>1</segmented>		                           //是否用于分割（在图像物体识别中01无所谓）
	<object>                                                           //检测到的物体
		<name>horse</name>                                         //物体类别
		<pose>Right</pose>                                         //拍摄角度
		<truncated>0</truncated>                                   //是否被截断（0表示完整）
		<difficult>0</difficult>                                   //目标是否难以识别（0表示容易识别）
		<bndbox>                                                   //bounding-box（包含左下角和右上角xy坐标）
			<xmin>100</xmin>
			<ymin>96</ymin>
			<xmax>355</xmax>
			<ymax>324</ymax>
		</bndbox>
	</object>
	<object>                                                           //检测到多个物体
		<name>person</name>
		<pose>Unspecified</pose>
		<truncated>0</truncated>
		<difficult>0</difficult>
		<bndbox>
			<xmin>198</xmin>
			<ymin>58</ymin>
			<xmax>286</xmax>
			<ymax>197</ymax>
		</bndbox>
	</object>
</annotation>
```

ImageSets存放的是每一种类型的challenge对应的图像数据。
在ImageSets下有四个文件夹：

其中Action下存放的是人的动作（例如running、jumping等等，这也是VOC challenge的一部分）

Layout下存放的是具有人体部位的数据（人的head、hand、feet等等，这也是VOC challenge的一部分）

Main下存放的是图像物体识别的数据，总共分为20类。

Segmentation下存放的是可用于分割的数据。

在这里主要考察Main文件夹。

Main文件夹下包含了20个分类的\*\*\*_train.txt、\*\*\*_val.txt和\*\*\*_trainval.txt。

这些txt中的内容都差不多如下：

```txt
000012
000017
000023
000026
000032
...
```

表示图像的name。

# ILSVRC2015数据集介绍

```bash
├── Annotations
│   └── CLS-LOC
│       └── train
│           ├── n01440764
│           │   ├── n01440764_18.xml
│           │   └── n01440764_37.xml
|           |   |__ ...
│           └── n01443537
│               ├── n01443537_16.xml
│               └── n01443537_2.xml
|               |__ ...
├── Data
│   └── CLS-LOC
│       ├── test
│       ├── train
│       │   ├── n01440764
│       │   │   ├── n01440764_36.JPEG
│       │   │   └── n01440764_37.JPEG
|       |       |__ ...
│       │   └── n01443537
│       │       ├── n01443537_16.JPEG
│       │       └── n01443537_2.JPEG
|       |       |__ ...
│       └── val
├── ImageSets
│   └── CLS-LOC
│       ├── test.txt
│       ├── train_cls.txt
│       ├── train_loc.txt
│       └── val.txt
└── devkit
    ├── COPYING
    ├── data
    │   └── map_clsloc.txt
    |   |__ ...
    ├── evaluation
    │   └── VOCreadxml.m
    |   |__ ...
    └── readme.txt
```

ILSVRC2015_devkit\devkit\data\map_clsloc.txt文件描述了类别对应关系。

# 转换方法

## 图片JPEG转jpg格式

VOC2007中的原图片的数据格式为.jpg格式，而ILSVRC2015数据集中的原始图片为.JPEG格式。考虑到有些深度神经网络只支持.jpg格式的图片。因此，首先将.JPEG格式图片转换为.jpg格式的图片。转换代码可参考如下：

```matlab
clc;
clear all;
 
maindir = '/Users/xiaoqiangteng/Downloads/imageset/JPEGImages1/';
subdir =  dir( maindir ); % 遍历所有子文件夹
 
for i = 1 : length( subdir )
    if( isequal( subdir( i ).name, '.' ) || ...
        isequal( subdir( i ).name, '..' ) || ...
        ~subdir( i ).isdir )   % 过滤空文件夹
        continue;
    end
     
    subdirpath = fullfile( maindir, subdir( i ).name, '*.JPEG' ); %subdir( i ).name = 'n00007846'; subdirpath = S:\ImageNet\JPEGImages1\n00007846\*.JPEG;
    images = dir( subdirpath );
     
    for j = 1 : length( images )
        imagepath = fullfile(maindir, subdir( i ).name, images( j ).name);
        imgdata = imread( imagepath);
        subdirpath1 = strcat(maindir, subdir( i ).name);
        subdirpath = strcat(subdirpath1, '/');
        jpgPath = [subdirpath, images( j ).name(1:end-5), '.jpg'];
        imwrite(imgdata, jpgPath, 'mode','lossless');
        delete(imagepath, images( j ).name);
    end
end
```

待解决的问题：

函数imwrite(imgdata, jpgPath);默认参数会改变图片大小。

imwrite(imgdata, jpgPath, 'mode','lossless');加入特定模式后，图片损坏。

## 转换XML文件

参考这篇http://blog.csdn.net/samylee/article/details/51201744，他是将每个图片的数据写成了一个txt文件，然后用txt转化为xml文件。我模仿这种方法，那么我得先获得txt文件，所以现在的第一步是要将我的imageNet的annotation，即xml文件转化为参考博客所提到的txt文件。

### XML文件转txt文件

```matlab
%createtxt.m
clc;
clear all;
 
RootPath = '/Users/xiaoqiangteng/Downloads/imageset/';
[dir_name, count, label] = importDataFiles(RootPath);

path_image = '/Users/xiaoqiangteng/Downloads/imageset/JPEGImages2/';%原始图片文件夹
path_xml = '/Users/xiaoqiangteng/Downloads/imageset/Annotations1/';
path_label = '/Users/xiaoqiangteng/Downloads/imageset/labels/';%生成的txt文件夹
subdir = dir(path_xml);

for i = 3 : length( subdir )
    if( isequal( subdir( i ).name, '.' ) || ...
        isequal( subdir( i ).name, '..' ) || ...
        ~subdir( i ).isdir )   % 过滤空文件夹
        continue;
    end
      
    subdirpath = fullfile(path_xml, subdir( i ).name, '*.xml');
    xml_files1 = dir( subdirpath );
    
    %XML文件排序
    xml_files2 = [];
    int_xml_files = [];
    int_xml = [];
    xml_files = [];
    length_xml = length(xml_files1);
    for k = 1:length(xml_files1)
        xml_files2(k).name = xml_files1(k).name(11:end-4);
        int_xml_files(k) = str2num(xml_files2(k).name);
    end
    int_xml = sort(int_xml_files);
    
    for ii = 1:length(xml_files1)
        xml_files3 = strcat(subdir( i ).name, '_');
        xml_files4 = strcat(xml_files3, num2str(int_xml(ii)));
        xml_files(ii).name = strcat(xml_files4, '.xml');
    end
    
    mkdir(path_label, subdir( i ).name);
    pathtxt1 = strcat(path_label, subdir( i ).name);
    pathtxt2 = strcat(pathtxt1, '/');
    
    
    % 遍历XML文件
    for j = 1 : length(xml_files)
        disp(j);
         try
            pathtxt = [pathtxt2 xml_files( j ).name(1:end-4) '.txt'];
            subdir_xml = fullfile(path_xml, subdir( i ).name, xml_files( j ).name);
            str = fileread(subdir_xml);
            v = xml_parse( str );
            xmin = v.object.bndbox.xmin;
            ymin = v.object.bndbox.ymin;
            xmax = v.object.bndbox.xmax;
            ymax = v.object.bndbox.ymax;
            filename = v.filename;
            fid = fopen(pathtxt,'wt');
            fprintf(fid,'%s%s',filename,'.JPEG');
            fprintf(fid,'%c',' ');
            fprintf(fid,'%s', label{i-2});
            fprintf(fid,'%c',' ');
            fprintf(fid,'%c',xmin);
            fprintf(fid,'%c',' ');
            fprintf(fid,'%c',ymin);
            fprintf(fid,'%c',' ');
            fprintf(fid,'%c',xmax);
            fprintf(fid,'%c',' ');
            fprintf(fid,'%c',ymax);
            fclose(fid);
         catch
%             delete_image1 = strcat(path_image, subdir( i ).name);
%             delete_image2 = strcat(delete_image1, '/');
%             delete_image = [delete_image2, xml_files( j ).name(1:end-4), '.JPEG'];
%             delete(delete_image);
             delete_xml1 = strcat(path_xml, subdir( i ).name);
             delete_xml2 = strcat(delete_xml1, '/');
             delete_xml = [delete_xml2, xml_files( j ).name(1:end-4), '.xml'];
             delete(delete_xml);
             disp('Wrong');
         end
    end
end
```

importDataFiles(RootPath)函数：

```matlab
function [dir_name, count, label]=importDataFiles(RootPath)
DirOutput = dir(fullfile(RootPath));
SimpleName = {DirOutput(3:end).name}';
LenSimFile = length(SimpleName);

for i=1:LenSimFile
    fileName = fullfile(RootPath,SimpleName{i});
    switch SimpleName{i}
        case 'map_clsloc.txt'
            [dir_name, count, label] = textread(fileName,'%s%d%s');
    end    
end

end
```

改代码通过遍历xml文件来生成txt文件。原因在于原始图片文件夹内的图片多余对应的xml文件。

可能存在的问题：

（1）以上matlab代码通过使用XML 函数来解析XML文件，即xml_parse()函数。需要先下载该函数的工具包，下载地址：https://cn.mathworks.com/matlabcentral/fileexchange/4278-xml-toolbox?focused=5055046&tab=function

但是该工具包在高版本的matlab已不支持，请尝试低版本的Matlab。楼主使用matlab 2014a版本，可运行。

（2）待解决的问题

以上代码仅仅支持XML中存在一个object对象。若存在多个object对象，即会报错，运行catch语句块，将不能够读取的xml文件从原文件夹中删除，以此来保证xml文件的数量同txt文件的数量相同。但是，该问题应该很好能够解决。

### TXT转XML

接下来就可以进行将txt转化为pascal voc格式的xml文件了，在当前目录下创建一个Annotations的文件夹，代码如下：

```matlab
%writeanno.m
clc;
clear all;
 
path_image = '/home/teng/programmings/datasets/imagenet/imagenet/JPEGImages/';
path_label = '/home/teng/programmings/datasets/imagenet/imagenet/labels/';
path_xml = '/home/teng/programmings/datasets/imagenet/imagenet/Annotations/';
subdir = dir(path_label);

for i = 3 : length( subdir )
    if( isequal( subdir( i ).name, '.' ) || ...
        isequal( subdir( i ).name, '..' ) || ...
        ~subdir( i ).isdir )
        continue;
    end
      
    subdirpath = fullfile(path_label, subdir( i ).name, '*.txt');
    txt_files = dir( subdirpath );
    
    mkdir(path_xml, subdir( i ).name);
    for j = 1:length(txt_files)
        disp(i, j)
        path_label_dir1 = strcat(path_label, subdir( i ).name);
        path_label_dir = strcat(path_label_dir1, '/');
        msg = textread(strcat(path_label_dir, txt_files(j).name(1:end-4),'.txt'),'%s');
        clear rec;
        path_xml_subdir1 = strcat(path_xml, subdir( i ).name);
        path_xml_subdir = strcat(path_xml_subdir1, '/');
        path = [path_xml_subdir txt_files(j).name(1:end-4) '.xml'];
        fid=fopen(path,'w');
        rec.annotation.folder = 'VOC2007';%数据集名
        rec.annotation.filename = strcat(txt_files(j).name(1:end-4), '.JPEG');
        rec.annotation.source.database = 'The VOC2007 Database';    
        rec.annotation.source.annotation = 'PASCAL VOC2007';    
        rec.annotation.source.image = 'flickr';
        rec.annotation.source.flickrid = '0';
        rec.annotation.owner.flickrid = 'I do not know';    
        rec.annotation.owner.name = 'I do not know';

        path_image_subdir1 = strcat(path_image, subdir( i ).name);
        path_image_subdir = strcat(path_image_subdir1, '/');
        img = imread([path_image_subdir txt_files(j).name(1:end-4) '.JPEG']);
        rec.annotation.size.width = int2str(size(img,2));
        rec.annotation.size.height = int2str(size(img,1));
        rec.annotation.size.depth = int2str(size(img,3));

        rec.annotation.segmented = '0';  
        rec.annotation.object.name = msg{2};   
        rec.annotation.object.pose = 'Left';    
        rec.annotation.object.truncated = '1';    
        rec.annotation.object.difficult = '0';    
        rec.annotation.object.bndbox.xmin = msg{3};%坐标x1
        rec.annotation.object.bndbox.ymin = msg{4};%坐标y1
        rec.annotation.object.bndbox.xmax = msg{5};%坐标x2
        rec.annotation.object.bndbox.ymax = msg{6};%坐标y2
        writexml(fid,rec,0);
        fclose(fid);
    end   
end
```

writexml函数：

```matlab
%writexml.m
function xml = writexml(fid,rec,depth)

fn=fieldnames(rec);
for i=1:length(fn)
    f=rec.(fn{i});
    if ~isempty(f)
        if isstruct(f)
            for j=1:length(f)            
                fprintf(fid,'%s',repmat(char(9),1,depth));
                a=repmat(char(9),1,depth);
                fprintf(fid,'<%s>\n',fn{i});
                writexml(fid,rec.(fn{i})(j),depth+1);
                fprintf(fid,'%s',repmat(char(9),1,depth));
                fprintf(fid,'</%s>\n',fn{i});
            end
        else
            if ~iscell(f)
                f={f};
            end       
            for j=1:length(f)
                fprintf(fid,'%s',repmat(char(9),1,depth));
                fprintf(fid,'<%s>',fn{i});
                if ischar(f{j})
                    fprintf(fid,'%s',f{j});
                elseif isnumeric(f{j})&&numel(f{j})==1
                    fprintf(fid,'%s',num2str(f{j}));
                else
                    error('unsupported type');
                end
                fprintf(fid,'</%s>\n',fn{i});
            end
        end
    end
end
```

## 移动目录内所有文件夹内的文件至上层目录

```bash
#! /bin/bash
function read_dir(){
for file in `ls $1`       #注意此处这是两个反引号，表示运行系统命令
do
if [ -d $1"/"$file ]  #注意此处之间一定要加上空格，否则会报错
then
echo $1$file
read_dir $1"/"$file
else
mv $1"/"$file /home/teng/programmings/datasets/imagenet/imagenet/Annotations
fi
done
}   

function delete_dir(){
for file in `ls $1`
do
if [ -d $1"/"$file ]
then
echo $1$"/"file
rm -rf $1"/"$file
fi
done
}

    #读取第一个参数
read_dir $1
delete_dir $1
```

这一步的目的在于将Annotations文件夹内的所有xml文件置于一个目录下。

## imageSets文件夹

代码如下：

```matlab
%createimagesets.m
clc;
clear all;
  
file = dir('/home/teng/programmings/datasets/imagenet/imagenet/Annotations/');
len = length(file)-2;


num_trainval=sort(randperm(len, floor(9*len/10)));%trainval集占所有数据的9/10，可以根据需要设置
num_train=sort(num_trainval(randperm(length(num_trainval), floor(5*length(num_trainval)/6))));%train集占trainval集的5/6，可以根据需要设置
num_val=setdiff(num_trainval,num_train);%trainval集剩下的作为val集
num_test=setdiff(1:len,num_trainval);%所有数据中剩下的作为test集
path = '/home/teng/programmings/datasets/imagenet/imagenet/ImageSets/Main/';


fid=fopen(strcat(path, 'trainval.txt'),'a+');
for i=1:length(num_trainval)
    s = sprintf('%s',file(num_trainval(i)+2).name);
    fprintf(fid,[s(1:length(s)-4) '\r\n']);
end
fclose(fid);


fid=fopen(strcat(path, 'train.txt'),'a+');
for i=1:length(num_train)
    s = sprintf('%s',file(num_train(i)+2).name);
    fprintf(fid,[s(1:length(s)-4) '\r\n']);
end
fclose(fid);


fid=fopen(strcat(path, 'val.txt'),'a+');
for i=1:length(num_val)
    s = sprintf('%s',file(num_val(i)+2).name);
    fprintf(fid,[s(1:length(s)-4) '\r\n']);
end
fclose(fid);


fid=fopen(strcat(path, 'test.txt'),'a+');
for i=1:length(num_test)
    s = sprintf('%s',file(num_test(i)+2).name);
    fprintf(fid,[s(1:length(s)-4) '\r\n']);
end
fclose(fid);
```

这样所需的文件夹我们都已备齐，将imageSets，Annotations和JPEGiImage文件夹分别放入voc数据集的对应位置，在这之前先将其原来的文件夹删除。

# 参考：

1. https://blog.csdn.net/xbcReal/article/details/51259558
2. https://blog.csdn.net/samylee/article/details/51201744
3. https://blog.csdn.net/sinat_30071459/article/details/50723212