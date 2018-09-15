---
title: 2D-3D-S数据集介绍
date: 2018-09-14 08:25:03
categories:
- 编程
- 数据集
tags:
- 编程
- 数据集
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# 引言

数据集详情地址：http://3Dsemantics.stanford.edu/

该数据集构建的主要目标是为场景理解、深度估计、法线估计、实体检测、分割和场景重建提供研究数据集，推动该领域的发展。

# 相关数据集介绍

现有的基于RGB-D数据集包括NYU Depth v2、SUN RGBD和SceneNN。然而，这些数据集对于处理特定的任务和2.5D的场景有限。

# 采集和处理

## 3D模式

3D模式主要包含点云和mesh。文章利用Matterport Camera得到3D textured Mesh model，然后利用采样方法得到3D点运数据。此外，文章将点运数据与颜色进行对应来生成colored点云数据。

语义标记：文章对点云数据中的每个Mesh和Voxel进行语义信息的标记。语义主要包括ceiling, floor, wall, beam, column, window, door, table, chair, sofa, bookcase, board and clutter。此外，语义信息被投影到RGB图像中。进一步地，文章划分数据为如下室内场景：office, conference room, hallway, auditorium, open space, lobby, lounge, pantry, copy room, storage and WC。每个对象被存储在文件名为“class instanceNum roomType roomNum areaNum”中。

> The dataset contains colored point clouds and textured meshes for each scanned area. 3D semantic annotations for objects and scenes are offered for both modalities, with point-level and face-level labels correspondingly. The annotations were initially performed on the point cloud and then projected onto the closest surface on the 3D mesh model. Faces in the 3D mesh that account for no projected points belong to non-annotated parts of the dataset and are labeled with a default null value. We also provide the tightest axis-aligned bounding box of each object instance and further voxelize it into a 6x6x6 grid with binary occupancy and point correspondence.

该数据集包含每个扫描区域的带有颜色信息的点云和纹理mesh，分别提供点粒度和面粒度的实体和场景的语义信息。最初，语义信息被提取自点云数据，之后映射到最相邻的三维mesh上。不能够被点云映射的面的语义信息置为空值。该数据集也提供了每隔实体样例的tightest axis-aligned bounding box和使用6x6x6二进制的网格来进行体素化表达。

### 技术细节

该数据集包含颜色点云和3D mesh。

#### 颜色点云数据

> The raw colored 3D point clouds along with both object and scene instance-level annotations per point, (tightest) axis-aligned bounding boxes and voxels with binary occupancy and point correpsondence are stored in the Area\_#_PointCloud.mat file. The variables are stored in the form of nested structs

具有实体和场景粒度的语义信息、axis-aligned bounding boex和具有点对应的二进制占据体素的三维点云数据被存储在“Area\_#_PointCloud.mat”文件中。变量被存储为嵌套结构体：

```c
- Area: --> name: the area name as: Area_#
		--> Disjoint-Space: struct with information on the individual spaces in the building.
```

使用matlab读取后结构体显示如下：

![]()

其中，Disjoint_Space结构体为

```C
- Disjoint_Space:	--> name: the name of that space, with per area global index (e.g. conferenceRoom_1, offie_13, etc.)
					-->AlignmentAngle: rotation angle around Z axis, to align spaces based on the CVPR2016 paper *3D Semantic Parsing of Large-Scale Indoor Spaces*. 
					--> color: a unique RGB color value [0,1] for that room, mainly for visualization purposes
					--> object: a struct that contains all objects in that space.
```

使用matlab读取后结构体显示如下：

![]()

object结构体为：

```C
- object:	--> name: the name of the object, wiith per space indexing* (e.g. chair-1, wall_3, clutter_13, etc.)
			--> points: the X,Y,Z coordinates of the 3D points that comprise this object
			--> RGB_color: the raw RGB color value [0,255] associated with each point.
			--> global_name: the name of the object, with per area global index**
			--> Bbox: [Xmin Xmax Ymin Ymax Zmin Zmax] of the object's boudning box
			--> Voxels: [Xmin Xmax Ymin Ymax Zmin Zmax] for each voxel in a 6x6x6 grid
			--> Voxel_Occupancy: binary occupancy per voxel (0: empty, 1: contains points)
			--> Points_per_Voxel: the object points that correspond to each voxel (in XYZ coordinates)
```

![]()

#### 3D Semantic mesh

> The 3D semantic mesh is labeled with instance-level per-face annotations. The mesh is stored in semantic.obj and semantic.mtl where face labels are stored as face's material's name. In Blender, for example, the material label can be retrieved with:

三维语义mesh被使用具有实例粒度的每个面元的注释。mesh被存储在"semantic.obj"和"semantic.mtl"文件中。其中，面元标签被存储在面元的名字。在Blender中，标签可悲解析如下：

```C
mesh_material_idx = mesh.data.polygons[ face_idx ].material_index
material = mesh.data.materials[ mesh_material_idx ]
label = material.name   # final label!
```

标签格式如下：

```C
class_instanceNum_roomType_roomNum_areaNum
```

When using the mesh's color, the material's color should be remapped for the task at hand as the default color is designed for visualizations. One way to encode the label in the color is to set

```C
material.diffuse_color = get_color( labels.index( material.name ) ) 
```

by using labels from /assets/semantic_labels.json and get_color( color ) from /assets/utils.py.

## 2D模态

RGB图像：采集朝向：yaw角约[-180, 180], pitch角约为以0为均值15度为方差的高斯分布，roll角一直是0度。FOV角为以75度为均值和-30度为标准差的半高斯分布。为了过滤掉白墙等图像，文章基于语义信息商来过滤掉近70%的图像。

语义信息商的计算方法如下：首先，对于每一幅图像对13类别的语义信息进行像素统计。再利用像素分布计算香农商。最后，将低于15%的图像过滤掉。

此外，我们保留60%的图像。剩下的25%的图像我们通过半高斯分布采样方法来筛选。如此，文章保证了图像的多样性。

> The dataset contains densely sampled RGB images per scan location. These images were sampled from equirectangular images that were generated per scan location and modality using the raw data captured by the scanner. All images in the dataset are stored in full high-definition at 1080x1080 resolution. For more details on the random sampling of RGB images read section 4.2 in the paper. We provide the camera metadata for each generated image.

> We also provide depth images that were computed on the 3D mesh instead of directly on the 3D mesh, as well as surface normal images. 2D semantic annotations are computed for each image by projecting the 3D mesh labels on the image plane. Due to certain geo- metric artifacts present at the mesh model mainly because of the level of detail in the reconstruction, the 2D annotations occasionally present small local misalignment to the underlying pixels, especially for points that have a short distance to the camera. This issue can be easily addressed by fusing image content with the projected annotations using graphical models. The dataset also includes 3D coordinate encoded images where each pixel encodes the X, Y, Z location of the point in the world coordinate system. Last, an equirectangular projection is also provided per scan location and modality.

该数据集包含了每个采集区域的稠密的RGB图像。作者使用扫描仪输出的全景图像来生成这些图像。在数据集里的所有的图像的分辨率均为1080x1080。细节请参考论文。

此外，数据集也包含深度图像。作者使用三维mesh的语义信息，通过映射到图像平面来得到图像中的语义信息。数据集中每幅图像的三维坐标X,Y,Z均在世界坐标系下。

### 技术细节

#### pose

> The pose files contain camera metadata for each image and are given in the /pose subdirectories. They have filenames which are globally unique due to the fact that camera uuids are not shared between areas. They are stored in json files, and contain

pose文件包含每一幅图像的camera元数据，被给定在“/pose”子文件夹下。他们有一个全局的文件名，被存储在json文件中。

```json
{
  "camera_k_matrix":  # The 3x3 camera K matrix. Stored as a list-of-lists, 
  "field_of_view_rads": #  The Camera's field of view, in radians, 
  "camera_original_rotation": #  The camera's initial XYZ-Euler rotation in the .obj, 
  "rotation_from_original_to_point": 
  #  Apply this to the original rotation in order to orient the camera for the corresponding picture, 
  "point_uuid": #  alias for camera_uuid, 
  "camera_location": #  XYZ location of the camera, 
  "frame_num": #  The frame_num in the filename, 
  "camera_rt_matrix": #  The 4x3 camera RT matrix, stored as a list-of-lists, 
  "final_camera_rotation": #  The camera Euler in the corresponding picture, 
  "camera_uuid": #  The globally unique identifier for the camera location, 
  "room": #  The room that this camera is in. Stored as roomType_roomNum_areaNum 
}
```

#### RGB

> RGB images are in the /rgb folder and contain synthesized but accurate RGB images of the scene.

RGB图像被存储在“/rgb”文件夹中。

#### Depth

> Depth images are stored as 16-bit PNGs and have a maximum depth of 128m and a sensitivity of 1/512m. Missing values are encoded with the value 2^16 - 1. Note that while depth is defined relative to the plane of the camera in the data (z-depth), it is defined as the distance from the point-center of the camera in the panoramics.

#### Global XYZ

> Global XYZ images contain the ground-truth location of each pixel in the mesh. They are stored as 16-bit 3-channel OpenEXR files and a convenience readin function is provided in /assets/utils.py. These can be used for generating point correspondences, e.g. for optical flow. Missing values are encoded as #000000.

#### Normal

> Normals are 127.5-centered per-channel surface normal images. For panoramic images, these normals are relative to the global corodinate system. Since the global coordinate system is impossible to determine from a sampled image, the normal images in /data have their normals defined relative to the direction the camera is facing. The normals axis-color convention is the same one used by NYU RGB-D. Areas where the mesh is missing have pixel color #808080.

#### Semantic

> Semantic images come in two variants, semantic and semantic_pretty. They both include information from the point cloud annotations, but only the semantic version should be used for learning. The labels can be found in assets/semantic_labels.json, and images can be parsed using some of the convenience functions in utils.py. Specifically: The semantic images are encoded as 3-channel 8-bit PNGs which are interpreted as 24-bit base-256 integers which are an index into the labels array in semantic_labels.json.

Let's say that you've loaded the image into memory and it's stored as a numpy array called img and want the label for the pixel at (1500, 2000) which is the leftmost sofa chair in this image. utils.py provides get_index, load_labels and parse_labels for extracting the label information. Here is what your code might look like:

```python
from scipy.misc import imread
from assets.utils import *  # Assets should be downloaded from this repo
labels = load_labels( '/path/to/assets/semantic_labels.json' )

img = imread(  '/path/to/image.png' )
pix = img[ 1500,2000 ]
instance_label = labels[ get_index( pix ) ]
instance_label_as_dict = parse_label( instance_label )
print instance_label_as_dict
```

Gives {'instance_num': 5, 'instance_class': u'sofa', 'room_num': 3, 'room_type': u'office', 'area_num': 3} Here we can see that this is the 5th instance of class 'sofa' in area 3.

Finally, note that pixels where the data is missing are encoded with the color #0D0D0D which is larger than the len( labels ).


