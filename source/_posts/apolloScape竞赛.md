---
title: apolloScape竞赛
date: 2018-09-09 11:48:32
categories:
- 编程
- 竞赛
tags:
- 编程
- 竞赛
copyright:
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

# 背景介绍

百度ApolloScape重磅发布了自动驾驶开放数据集。自动驾驶开发测试中，海量、高质的真实数据是必不可缺的“原料”。但是，少有团队有能力开发并维持一个适用的自动驾驶平台，定期校准并收集新数据。

据介绍，Apollo开放平台此次发布的ApolloScape不仅开放了比Cityscapes等同类数据集大10倍以上的数据量，包括感知、仿真场景、路网数据等数十万帧逐像素语义分割标注的高分辨率图像数据，进一步涵盖更复杂的环境、天气和交通状况等。从数据难度上来看，ApolloScape数据集涵盖了更复杂的道路状况（例如，单张图像中多达162辆交通工具或80名行人），同时开放数据集采用了逐像素语义分割标注的方式，是目前环境最复杂、标注最精准、数据量最大的自动驾驶数据集。

Apollo开放平台还将与加州大学伯克利分校在CVPR 2018（IEEE国际计算机视觉与模式识别会议）期间联合举办自动驾驶研讨会（Workshop on Autonomous Driving），并将基于ApolloScape的大规模数据集定义了多项任务挑战，为全球自动驾驶开发者和研究人员提供共同探索前沿领域技术突破及应用创新的平台。

# 参考一：[PoseNet implementation for self-driving car localization using Pytorch on Apolloscape dataset](https://capsulesbot.com/blog/2018/08/24/apolloscape-posenet-pytorch.html)

This article covers the very beginning of the journey and includes the reading and visualization of the Apolloscape dataset for localization task. Implement PoseNet [2] architecture for monocular image pose prediction and visualize results. I use Python and Pytorch for the task.

NOTE: If you want to jump straight to the code here is the GitHub repo. It’s is still an ongoing work where I intend to implement Vidloc [7], Pose Graph Optimization [3,8] and Structure from Motion [9] pipelines for Apolloscape Dataset in the context of the localization task.

## Apolloscape Pytorch Dataset

For Pytorch I need to have a Dataset object that prepares and feeds the data to the loader and then to the model. I want to have a robust dataset class that can:

- support stereo and mono images
- support train/validation splits that came along with data or generate a new one
- support pose normalization
- support different pose representations (needed mainly for visualization and experiments with loss functions)
- support filtering by record id
- support general Apolloscape folder structure layout

I am not putting here the full listing of the Apolloscape dataset and concentrate solely on how to use it and what data we can get from it. For the full source code, please refer to the Github file datasets/apolloscape.py.

Here how to create a dataset:

```python
from datasets.apolloscape import Apolloscape
from torchvision import transforms

# Path to unpacked data folders
APOLLO_PATH = "./data/apolloscape"

# Resize transform that is applied on every image read
transform = transforms.Compose([transforms.Resize(250)])

apollo_dataset = Apolloscape(root=os.path.join(APOLLO_PATH), road="zpark-sample",
                             transform=transform, train=True, pose_format='quat',
                             stereo=True)
print(apollo_dataset)
```

output:

```python
Dataset: Apolloscape
    Road: zpark-sample
    Record: None
    Train: None
    Normalize Poses: False
    Stereo: True
    Length: 1499 of 1499
    Cameras: ['Camera_2', 'Camera_1']
    Records: ['Record001', 'Record002', 'Record003', 'Record004', 'Record006', 'Record007', 'Record008', 'Record009', 'Record010', 'Record011', 'Record012', 'Record013', 'Record014']
```

POLLO_PATH is a folder with unpacked Apolloscape datasets, e.g. \$APOLLO_PATH/road02_seg or \$APOLLO_PATH/zpark. Download data from Apolloscape page and unpack iot. Let’s assume that we’ve also created a symlink ./data/apolloscape that points to $APOLLO_PATH folder.

We can view the list of available records with a number of data samples in each:

```python
# Show records with numbers of data points
recs_num = apollo_dataset.get_records_counts()
recs_num = sorted(recs_num.items(), key=lambda kv: kv[1], reverse=True)
print("Records:")
print("\n".join(["\t{} - {}".format(r[0], r[1]) for r in recs_num ]))
```

output:

```python
Records:
	Record008 - 122
	Record007 - 121
	Record006 - 121
	Record012 - 121
	Record001 - 121
	Record009 - 121
	Record010 - 121
	Record003 - 121
	Record013 - 120
	Record004 - 120
	Record002 - 120
	Record011 - 120
	Record014 - 50
```

We can draw a route for one record with a sampled camera image:

```python
from utils.common import draw_record

# Draw path of a record with a sampled datapoint
record = 'Record008'
draw_record(apollo_dataset, record)
plt.show()
```

output:

Alternatively, we can see all records at once in one chart:

```python
# Draw all records for current dataset
draw_record(apollo_dataset)
plt.show()
```

output:

Another option is to see it in a video:

```python
from utils.common import make_video

# Generate and save video for the record
outfile = "./output_data/videos/video_{}_{}.mp4".format(apollo_dataset.road, apollo_dataset.record)
make_video(apollo_dataset, outfile=outfile)
```

Output (cut gif version of the generated video):

For the PoseNet training we will use mono images with zero-mean normalized poses and camera images center-cropped to 250px:

```python
# Resize and CenterCrop
transform = transforms.Compose([
    transforms.Resize(260),
    transforms.CenterCrop(250)
])

# Create train dataset with mono images, normalized poses, enabled cache_transform
train_dataset = Apolloscape(root=os.path.join(APOLLO_PATH), road="zpark-sample",
                             transform=transform, train=True, pose_format='quat',
                             normalize_poses=True, cache_transform=True,
                             stereo=False)

# Draw path of a single record (mono with normalized poses)
record = 'Record008'
draw_record(apollo_dataset, record)
plt.show()
```

Output:

Implemented Apolloscape Pytorch dataset also supports cache_transform option which is when enabled saves all transformed pickled images to a disk and retrieves it later for the subsequent epochs without the need to redo convert and transform operations every image read event. Cache saves up to 50% of the time during training time though it’s not working with image augmentation transforms like torchvision.transforms.ColorJitter.

Also, we can get the mean and the standard deviation that we need later to recover true poses translations:

```python
poses_mean = train_dataset.poses_mean
poses_std = train_dataset.poses_std
print('Translation poses_mean = {} in meters'.format(poses_mean))
print('Translation poses_std  = {} in meters'.format(poses_std))
```

Output:

```python
Translation poses_mean = [  449.95782055 -2251.24771214    40.17147932] in meters
Translation poses_std  = [123.39589457 252.42350964   0.28021513] in meters
```

You can find all mentioned examples in Apolloscape_View_Records.ipynb notebook.

And now let’s turn to something useful and more interesting, for example, training PoseNet deep convolutional network to regress poses from camera images.

## PoseNet localization task

参考：[PoseNet implementation for self-driving car localization using Pytorch on Apolloscape dataset](https://capsulesbot.com/blog/2018/08/24/apolloscape-posenet-pytorch.html)

A Pytorch implementation of the PoseNet model using a mono image:

```python
import torch
import torch.nn.functional as F

class PoseNet(torch.nn.Module):

    def __init__(self, feature_extractor, num_features=128, dropout=0.5,
                 track_running_stats=False, pretrained=False):
        super(PoseNet, self).__init__()
        self.dropout = dropout
        self.feature_extractor = feature_extractor
        self.feature_extractor.avgpool = torch.nn.AdaptiveAvgPool2d(1)
        fc_in_features = self.feature_extractor.fc.in_features
        self.feature_extractor.fc = torch.nn.Linear(fc_in_features, num_features)

        # Translation
        self.fc_xyz = torch.nn.Linear(num_features, 3)

        # Rotation in quaternions
        self.fc_quat = torch.nn.Linear(num_features, 4)

    def extract_features(self, x):
        x_features = self.feature_extractor(x)
        x_features = F.relu(x_features)
        if self.dropout > 0:
            x_features = F.dropout(x_features, p=self.dropout, training=self.training)
        return x_features

    def forward(self, x):
        x_features = self.extract_features(x)
        x_translations = self.fc_xyz(x_features)
        x_rotations = self.fc_quat(x_features)
        x_poses = torch.cat((x_translations, x_rotations), dim=1)
        return x_poses
```

For further experiments I’ve also implemented stereo version (currently it’s simply processes two images in parallel without any additional constraints), option to switch off stats tracking for BatchNorm layers and Kaiming He normal for weight initialization [4]. Full source code is here models/posenet.py

## PoseNet Loss Functions

For more details on where it came from and intro to Bayesian Deep Learning (BDL) you can refer to an excellent post by Alex Kendall where he explains different types of uncertainties and its implications to the multi-task models. And even more results you can find in papers “Multi-task learning using uncertainty to weigh losses for scene geometry and semantics.” [5] and “What uncertainties do we need in Bayesian deep learning for computer vision?.” [6].

Pytorch implementation for both versions of a loss function is the following:

```python
class PoseNetCriterion(torch.nn.Module):
    def __init__(self, beta = 512.0, learn_beta=False, sx=0.0, sq=-3.0):
        super(PoseNetCriterion, self).__init__()
        self.loss_fn = torch.nn.L1Loss()
        self.learn_beta = learn_beta
        if not learn_beta:
            self.beta = beta
        else:
            self.beta = 1.0
        self.sx = torch.nn.Parameter(torch.Tensor([sx]), requires_grad=learn_beta)
        self.sq = torch.nn.Parameter(torch.Tensor([sq]), requires_grad=learn_beta)

    def forward(self, x, y):
        # Translation loss
        loss = torch.exp(-self.sx) * self.loss_fn(x[:, :3], y[:, :3])
        # Rotation loss
        loss += torch.exp(-self.sq) * self.beta * self.loss_fn(x[:, 3:], y[:, 3:]) + self.sq
        return loss
```

If learn_beta param is False it’s a simple weighted sum version of the loss and if learn_beta is True it’s using sx and sq params with enabled gradients that trains together with other network parameter with the same optimizer.

## PoseNet Training Implementation Details

Now let’s combine it all to the training loop. I use torch.optim.Adam optimizer with learning rate 1e-5, ResNet34 pretrained on ImageNet as a feature extractor and 2048 features on the last FC layer before pose regressors.

```python 
from torchvision import transforms, models
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
from datasets.apolloscape import Apolloscape
from utils.common import save_checkpoint
from models.posenet import PoseNet, PoseNetCriterion

APOLLO_PATH = "./data/apolloscape"

# ImageNet normalization params because we are using pre-trained
# feature extractor
normalize = transforms.Normalize(mean=[0.485, 0.456, 0.406],
                                     std=[0.229, 0.224, 0.225])

# Resize data before using
transform = transforms.Compose([
    transforms.Resize(260),
    transforms.CenterCrop(250),
    transforms.ToTensor(),
    normalize
])

# Create datasets
train_dataset = Apolloscape(root=os.path.join(APOLLO_PATH), road="zpark-sample",
    transform=transform, normalize_poses=True, pose_format='quat', train=True, cache_transform=True, stereo=False)
val_dataset = Apolloscape(root=os.path.join(APOLLO_PATH), road="zpark-sample",
    transform=transform, normalize_poses=True, pose_format='quat', train=False, cache_transform=True, stereo=False)

# Dataloaders
train_dataloader = DataLoader(train_dataset, batch_size=80, shuffle=True)
val_dataloader = DataLoader(val_dataset, batch_size=80, shuffle=True)

# Select primary device
if torch.cuda.is_available():
    device = torch.device('cuda')
else:
    device = torch.device('cpu')

# Create pretrained feature extractor
feature_extractor = models.resnet34(pretrained=True)

# Num features for the last layer before pose regressor
num_features = 2048

# Create model
model = PoseNet(feature_extractor, num_features=num_features, pretrained=True)
model = model.to(device)

# Criterion
criterion = PoseNetCriterion(stereo=False, learn_beta=True)
criterion = criterion.to(device)

# Add all params for optimization
param_list = [{'params': model.parameters()}]
if criterion.learn_beta:
    # Add sx and sq from loss function to optimizer params
    param_list.append({'params': criterion.parameters()})

# Create optimizer
optimizer = optim.Adam(params=param_list, lr=1e-5, weight_decay=0.0005)

# Epochs to train
n_epochs = 2000

# Main training loop
val_freq = 200
for e in range(0, n_epochs):
    train(train_dataloader, model, criterion, optimizer, e, n_epochs, log_freq=0,
         poses_mean=train_dataset.poses_mean, poses_std=train_dataset.poses_std,
         stereo=False)
    if e % val_freq == 0:
        validate(val_dataloader, model, criterion, e, log_freq=0,
            stereo=False)

# Save checkpoint
save_checkpoint(model, optimizer, criterion, 'zpark_experiment', n_epochs)
```

A little bit simplified train function below with error calculation that is used solely for logging purposes:

```python
def train(train_loader, model, criterion, optimizer, epoch, max_epoch,
          log_freq=1, print_sum=True, poses_mean=None, poses_std=None,
          stereo=True):

    # switch model to training
    model.train()

    losses = AverageMeter()

    epoch_time = time.time()

    gt_poses = np.empty((0, 7))
    pred_poses = np.empty((0, 7))

    end = time.time()
    for idx, (batch_images, batch_poses) in enumerate(train_loader):
        data_time = (time.time() - end)

        batch_images = batch_images.to(device)
        batch_poses = batch_poses.to(device)

        out = model(batch_images)
        loss = criterion(out, batch_poses)

        # Training step
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        losses.update(loss.data[0], len(batch_images) * batch_images[0].size(0) if stereo
                else batch_images.size(0))

        # move data to cpu & numpy
        bp = batch_poses.detach().cpu().numpy()
        outp = out.detach().cpu().numpy()
        gt_poses = np.vstack((gt_poses, bp))
        pred_poses = np.vstack((pred_poses, outp))

        # Get final times
        batch_time = (time.time() - end)
        end = time.time()

        if log_freq != 0 and idx % log_freq == 0:
            print('Epoch: [{}/{}]\tBatch: [{}/{}]\t'
                  'Time: {batch_time:.3f}\t'
                  'Data Time: {data_time:.3f}\t'
                  'Loss: {losses.val:.3f}\t'
                  'Avg Loss: {losses.avg:.3f}\t'.format(
                   epoch, max_epoch - 1, idx, len(train_loader) - 1,
                   batch_time=batch_time, data_time=data_time, losses=losses))


    # un-normalize translation
    unnorm = (poses_mean is not None) and (poses_std is not None)
    if unnorm:
        gt_poses[:, :3] = gt_poses[:, :3] * poses_std + poses_mean
        pred_poses[:, :3] = pred_poses[:, :3] * poses_std + poses_mean

    # Translation error
    t_loss = np.asarray([np.linalg.norm(p - t) for p, t in zip(pred_poses[:, :3], gt_poses[:, :3])])

    # Rotation error
    q_loss = np.asarray([quaternion_angular_error(p, t) for p, t in zip(pred_poses[:, 3:], gt_poses[:, 3:])])

    if print_sum:
        print('Ep: [{}/{}]\tTrain Loss: {:.3f}\tTe: {:.3f}\tRe: {:.3f}\t Et: {:.2f}s\t{criterion_sx:.5f}:{criterion_sq:.5f}'.format(
            epoch, max_epoch - 1, losses.avg, np.mean(t_loss), np.mean(q_loss),
            (time.time() - epoch_time), criterion_sx=criterion.sx.data[0], criterion_sq=criterion.sq.data[0]))
```

validate function is similar to train except model.eval()/model.train() modes, logging and error calculations. Please refer to /utils/training.py on GitHub for full-versions of train and validate functions.

The training converges after about 1-2k epochs. On my machine, with GTX 1080 Ti it takes about 22 seconds per epoch on ZPark sample train dataset with 2242 images pre-processed and scaled to 250x250 pixels. Total training time – 6-12 hours.

## PoseNet Results on Apolloscape dataset. ZPark sample road.

After 2k epochs of training, the model was managed to get a prediction of pose translation with a mean 40.6 meters and rotation with a mean 1.69 degrees.

## Further development

Established results are far from one that can be used in autonomous navigation where a system needs to now its location within accuracy of 15cm. Such precision is vital for a car to act safely, correctly predict the behaviors of others and plan actions accordingly. In any case, it’s a good baseline and building blocks of the pipeline to work with Apolloscape dataset that I can develop and improve further.

There many things to try next:

- Use temporal nature of a video.
- Rely on geometrical features of stereo cameras.
- Pose graph optimization techniques.
- Loss based on 3D reprojection errors.
- Structure from motion methods to build 3D map representation.

And what’s more importantly, all above-mentioned methods need no additional information but that we already have in ZPark sample road from Apolloscape dataset.

## References

- 1. Kendall, Alex, and Roberto Cipolla. “Geometric loss functions for camera pose regression with deep learning.” (2017).
- 2. Kendall, Alex, Matthew Grimes, and Roberto Cipolla. “Posenet: A convolutional network for real-time 6-dof camera relocalization.” (2015).
- 3. Brahmbhatt, Samarth, et al. “Mapnet: Geometry-aware learning of maps for camera localization.” (2017).
- 4. He, Kaiming, et al. “Delving deep into rectifiers: Surpassing human-level performance on imagenet classification.” (2015).
- 5. Kendall, Alex, Yarin Gal, and Roberto Cipolla. “Multi-task learning using uncertainty to weigh losses for scene geometry and semantics.” (2017).
- 6. Kendall, Alex, and Yarin Gal. “What uncertainties do we need in bayesian deep learning for computer vision?.” (2017).
- 7. Clark, Ronald, et al. “VidLoc: A deep spatio-temporal model for 6-dof video-clip relocalization.” (2017).
- 8. Calafiore, Giuseppe, Luca Carlone, and Frank Dellaert. “Pose graph optimization in the complex domain: Lagrangian duality, conditions for zero duality gap, and optimal solutions.” (2015).
- 9. Martinec, Daniel, and Tomas Pajdla. “Robust rotation and translation estimation in multiview reconstruction.” (2007).

# 参考

- [PoseNet implementation for self-driving car localization using Pytorch on Apolloscape dataset](https://capsulesbot.com/blog/2018/08/24/apolloscape-posenet-pytorch.html)





