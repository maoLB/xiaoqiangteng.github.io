---
title: semap论文
date: 2018-03-25 11:07:29
categories:
- 学术
- semap
tags:
- 学术
- semap
copyright:
---

# SeMap: 数字化室内语义地图低成本自动化构建方法

## 场景图重建

使用Colmap工具来获取场景图重建结果

## [Colmap使用方法](https://colmap.github.io/tutorial.html#quickstart)

场景图通过[Feature Detection and Extraction](https://colmap.github.io/tutorial.html#feature-detection-and-extraction)和[Feature Matching and Geometric Verification](https://colmap.github.io/tutorial.html#feature-matching-and-geometric-verification)两步完成。

首先使用GUI来操作。

### [打开Colmap](https://colmap.github.io/tutorial.html#data-structure)

（1）Windows: COLMAP.bat

Mac: COLMAP.app

Linux可选为：./src/exe/colmap gui

（2）新建项目-GUI操作

File > New project

在此步，首先确定生成的数据库保存位置，还要确定输入图片位置。为了方便起见，能够通过File > Save project来保存整个项目。

File > Open project为打开现有项目。

（3）新建项目-命令行操作

colmap gui 或 colmap gui --project_path path/to/project.ini

（4）例子

```
/path/to/project/...
+── images
│   +── image1.jpg
│   +── image2.jpg
│   +── ...
│   +── imageN.jpg
+── database.db
+── project.ini
```

其中，/path/to/project/images为图片路径，/path/to/project/database.db为数据库保存路径，/path/to/project/project.ini为项目配置保存路径。

### Feature Detection and Extraction

(1) GUI操作

Processing > Extract features

> 如果导入现有的特征提取的文件，需有如下文件：</br >
> NUM_FEATURES 128 </br >
> X Y SCALE ORIENTATION D_1 D_2 D_3 ... D_128</br >
> ...</br >
> X Y SCALE ORIENTATION D_1 D_2 D_3 ... D_128</br >
> where X, Y, SCALE, ORIENTATION are floating point numbers and D_1…D_128 values in the range 0…255. The file should have NUM_FEATURES lines with one line per feature. For example, if an image has 4 features, then the text file should look something like this:</br >
> 4 128</br >
> 1.2 2.3 0.1 0.3 1 2 3 4 ... 21</br >
> 2.2 3.3 1.1 0.3 3 2 3 2 ... 32</br >
> 0.2 1.3 1.1 0.3 3 2 3 2 ... 2</br >
> 1.2 2.3 1.1 0.3 3 2 3 2 ... 3</br >

### Feature Matching and Geometric Verification

Processing > Match features

### 查看结果

（1）[Database Management](https://colmap.github.io/tutorial.html#database-management)

Processing > Manage database

（2）[Database Format](https://colmap.github.io/database.html#database-format)

SQLite database file

The database contains the following tables:

- cameras
- images
- keypoints
- descriptors
- matches
- inlier_matches

Cameras and Images： The relation between cameras and images is 1-to-N. 

Keypoints and Descriptors：The detected keypoints are stored as row-major float32 binary blobs, where the first two columns are the X and Y locations in the image, respectively. 

Matches：Feature matching stores its output in the matches table and geometric verification in the inlier_matches table. COLMAP only uses the data in inlier_matches for reconstruction. Every entry in the two tables stores the feature matches between two unique images, where the pair_id is the row-major, linear index in the upper-triangular match matrix, generated as follows:</br >

```python
def image_ids_to_pair_id(image_id1, image_id2):
    if image_id1 > image_id2:
        return 2147483647 * image_id2 + image_id1
    else:
        return 2147483647 * image_id1 + image_id2
```

and image identifiers can be uniquely determined from the pair_id as:

``` python
def pair_id_to_image_ids(pair_id):
    image_id2 = pair_id % 2147483647
    image_id1 = (pair_id - image_id2) / 2147483647
    return image_id1, image_id2
```

The pair_id enables efficient database queries, as the matches tables may contain several hundred millions of entries. This scheme limits the maximum number of images in a database to 2147483647 (maximum value of signed 32-bit integers), i.e. image_id must be smaller than 2147483647.

The binary blobs in the matches tables are row-major uint32 matrices, where the left column are zero-based indices into the features of image_id1 and the second column into the features of image_id2. The column cols must be 2 and the rows column specifies the number of feature matches.