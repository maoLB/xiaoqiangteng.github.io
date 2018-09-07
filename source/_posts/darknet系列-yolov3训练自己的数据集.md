---
title: '[darknet系列]yolov3训练自己的数据集'
date: 2018-06-09 11:55:39
mathjax: true
categories:
- 编程
- 深度学习
- yolo
tags:
- 编程
- 深度学习
- yolo
copyright:
---

# 环境设置

darknet版本： 2018年6月 [yolov3](https://pjreddie.com/darknet/yolo/)

系统配置：

```bash
ubuntu 16.04
12GB Titan GPU
```

数据集：ILSVRC2015 (ILSVRC2015转VOC数据格式详见：http://mapstec.com/2018/04/05/ILSVRC2015%E6%95%B0%E6%8D%AE%E9%9B%86%E8%BD%ACVOC2007%E6%95%B0%E6%8D%AE%E9%9B%86%E6%A0%BC%E5%BC%8F/)

# darknet配置

下载工程:

```bash
git clone https://github.com/pjreddie/darknet
```

修改Makefile,参考我的makefile文件：

```bash
GPU=1
CUDNN=0
OPENCV=1
OPENMP=0
DEBUG=0

ARCH= -gencode arch=compute_30,code=sm_30 \
      -gencode arch=compute_35,code=sm_35 \
      -gencode arch=compute_50,code=[sm_50,compute_50] \
      -gencode arch=compute_52,code=[sm_52,compute_52]
#      -gencode arch=compute_20,code=[sm_20,sm_21] \ This one is deprecated?

# This is what I use, uncomment if you know your arch and want to specify
# ARCH= -gencode arch=compute_52,code=compute_52

VPATH=./src/:./examples
SLIB=libdarknet.so
ALIB=libdarknet.a
EXEC=darknet
OBJDIR=./obj/

CC=gcc
NVCC=/usr/local/cuda-8.0/bin/nvcc 
AR=ar
ARFLAGS=rcs
OPTS=-Ofast
LDFLAGS= -lm -pthread 
COMMON= -Iinclude/ -Isrc/
CFLAGS=-Wall -Wno-unused-result -Wno-unknown-pragmas -Wfatal-errors -fPIC

ifeq ($(OPENMP), 1) 
CFLAGS+= -fopenmp
endif

ifeq ($(DEBUG), 1) 
OPTS=-O0 -g
endif

CFLAGS+=$(OPTS)

ifeq ($(OPENCV), 1) 
COMMON+= -DOPENCV
CFLAGS+= -DOPENCV
LDFLAGS+= `pkg-config --libs opencv` 
COMMON+= `pkg-config --cflags opencv` 
endif

ifeq ($(GPU), 1) 
COMMON+= -DGPU -I/usr/local/cuda/include/
CFLAGS+= -DGPU
LDFLAGS+= -L/usr/local/cuda/lib64 -lcuda -lcudart -lcublas -lcurand
endif

ifeq ($(CUDNN), 1) 
COMMON+= -DCUDNN 
CFLAGS+= -DCUDNN
LDFLAGS+= -lcudnn
endif

OBJ=gemm.o utils.o cuda.o deconvolutional_layer.o convolutional_layer.o list.o image.o activations.o im2col.o col2im.o blas.o crop_layer.o dropout_layer.o maxpool_layer.o softmax_layer.o data.o matrix.o network.o connected_layer.o cost_layer.o parser.o option_list.o detection_layer.o route_layer.o upsample_layer.o box.o normalization_layer.o avgpool_layer.o layer.o local_layer.o shortcut_layer.o logistic_layer.o activation_layer.o rnn_layer.o gru_layer.o crnn_layer.o demo.o batchnorm_layer.o region_layer.o reorg_layer.o tree.o  lstm_layer.o l2norm_layer.o yolo_layer.o
EXECOBJA=captcha.o lsd.o super.o art.o tag.o cifar.o go.o rnn.o segmenter.o regressor.o classifier.o coco.o yolo.o detector.o nightmare.o darknet.o
ifeq ($(GPU), 1) 
LDFLAGS+= -lstdc++ 
OBJ+=convolutional_kernels.o deconvolutional_kernels.o activation_kernels.o im2col_kernels.o col2im_kernels.o blas_kernels.o crop_layer_kernels.o dropout_layer_kernels.o maxpool_layer_kernels.o avgpool_layer_kernels.o
endif

EXECOBJ = $(addprefix $(OBJDIR), $(EXECOBJA))
OBJS = $(addprefix $(OBJDIR), $(OBJ))
DEPS = $(wildcard src/*.h) Makefile include/darknet.h

#all: obj backup results $(SLIB) $(ALIB) $(EXEC)
all: obj  results $(SLIB) $(ALIB) $(EXEC)


$(EXEC): $(EXECOBJ) $(ALIB)
	$(CC) $(COMMON) $(CFLAGS) $^ -o $@ $(LDFLAGS) $(ALIB)

$(ALIB): $(OBJS)
	$(AR) $(ARFLAGS) $@ $^

$(SLIB): $(OBJS)
	$(CC) $(CFLAGS) -shared $^ -o $@ $(LDFLAGS)

$(OBJDIR)%.o: %.c $(DEPS)
	$(CC) $(COMMON) $(CFLAGS) -c $< -o $@

$(OBJDIR)%.o: %.cu $(DEPS)
	$(NVCC) $(ARCH) $(COMMON) --compiler-options "$(CFLAGS)" -c $< -o $@

obj:
	mkdir -p obj
backup:
	mkdir -p backup
results:
	mkdir -p results

.PHONY: clean

clean:
	rm -rf $(OBJS) $(SLIB) $(ALIB) $(EXEC) $(EXECOBJ) $(OBJDIR)/*

```

编译darknet：

```bash
make
```

配置完以后可以下载作者的预训练模型测试一下：

```bash
wget https://pjreddie.com/media/files/yolov3.weights
```

下载之后用图片进行测试：

```bash
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg
```

# 制作自己的数据集

本教授使用Imagenet数据集作为训练数据集，详细制作过程参考：http://mapstec.com/2018/04/05/ILSVRC2015%E6%95%B0%E6%8D%AE%E9%9B%86%E8%BD%ACVOC2007%E6%95%B0%E6%8D%AE%E9%9B%86%E6%A0%BC%E5%BC%8F/

# 数据处理

按darknet的说明编译好后，接下来在darknet-master/scripts文件夹中新建文件夹VOCdevkit，然后将整个VOC2007文件夹都拷到VOCdevkit文件夹下。

然后，需要利用scripts文件夹中的voc_label.py文件生成一系列训练文件和label，具体操作如下：

首先需要修改voc_label.py中的代码，这里主要修改数据集名，以及类别信息，我的是VOC2007，并且所有样本用来训练，没有val或test，有1000类目标，因此按如下设置：

```python
import xml.etree.ElementTree as ET
import pickle
import os
from os import listdir, getcwd
from os.path import join

#sets=[('2012', 'train'), ('2012', 'val'), ('2007', 'train'), ('2007', 'val'), ('2007', 'test')]
sets = [('2007', 'train')]

# classes = ["aeroplane", "bicycle", "bird", "boat", "bottle", "bus", "car", "cat", "chair", "cow", "diningtable", "dog", "horse", "motorbike", "person", "pottedplant", "sheep", "sofa", "train", "tvmonitor"]
classes = ["tench","goldfish","great_white_shark","tiger_shark","hammerhead","electric_ray","stingray","cock","hen","ostrich","brambling","goldfinch","house_finch","junco","indigo_bunting","robin","bulbul","jay","magpie","chickadee","water_ouzel","kite","bald_eagle","vulture","great_grey_owl","European_fire_salamander","common_newt","eft","spotted_salamander","axolotl","bullfrog","tree_frog","tailed_frog","loggerhead","leatherback_turtle","mud_turtle","terrapin","box_turtle","banded_gecko","common_iguana","American_chameleon","whiptail","agama","frilled_lizard","alligator_lizard","Gila_monster","green_lizard","African_chameleon","Komodo_dragon","African_crocodile","American_alligator","triceratops","thunder_snake","ringneck_snake","hognose_snake","green_snake","king_snake","garter_snake","water_snake","vine_snake","night_snake","boa_constrictor","rock_python","Indian_cobra","green_mamba","sea_snake","horned_viper","diamondback","sidewinder","trilobite","harvestman","scorpion","black_and_gold_garden_spider","barn_spider","garden_spider","black_widow","tarantula","wolf_spider","tick","centipede","black_grouse","ptarmigan","ruffed_grouse","prairie_chicken","peacock","quail","partridge","African_grey","macaw","sulphur-crested_cockatoo","lorikeet","coucal","bee_eater","hornbill","hummingbird","jacamar","toucan","drake","red-breasted_merganser","goose","black_swan","tusker","echidna","platypus","wallaby","koala","wombat","jellyfish","sea_anemone","brain_coral","flatworm","nematode","conch","snail","slug","sea_slug","chiton","chambered_nautilus","Dungeness_crab","rock_crab","fiddler_crab","king_crab","American_lobster","spiny_lobster","crayfish","hermit_crab","isopod","white_stork","black_stork","spoonbill","flamingo","little_blue_heron","American_egret","bittern","crane","limpkin","European_gallinule","American_coot","bustard","ruddy_turnstone","red-backed_sandpiper","redshank","dowitcher","oystercatcher","pelican","king_penguin","albatross","grey_whale","killer_whale","dugong","sea_lion","Chihuahua","Japanese_spaniel","Maltese_dog","Pekinese","Shih-Tzu","Blenheim_spaniel","papillon","toy_terrier","Rhodesian_ridgeback","Afghan_hound","basset","beagle","bloodhound","bluetick","black-and-tan_coonhound","Walker_hound","English_foxhound","redbone","borzoi","Irish_wolfhound","Italian_greyhound","whippet","Ibizan_hound","Norwegian_elkhound","otterhound","Saluki","Scottish_deerhound","Weimaraner","Staffordshire_bullterrier","American_Staffordshire_terrier","Bedlington_terrier","Border_terrier","Kerry_blue_terrier","Irish_terrier","Norfolk_terrier","Norwich_terrier","Yorkshire_terrier","wire-haired_fox_terrier","Lakeland_terrier","Sealyham_terrier","Airedale","cairn","Australian_terrier","Dandie_Dinmont","Boston_bull","miniature_schnauzer","giant_schnauzer","standard_schnauzer","Scotch_terrier","Tibetan_terrier","silky_terrier","soft-coated_wheaten_terrier","West_Highland_white_terrier","Lhasa","flat-coated_retriever","curly-coated_retriever","golden_retriever","Labrador_retriever","Chesapeake_Bay_retriever","German_short-haired_pointer","vizsla","English_setter","Irish_setter","Gordon_setter","Brittany_spaniel","clumber","English_springer","Welsh_springer_spaniel","cocker_spaniel","Sussex_spaniel","Irish_water_spaniel","kuvasz","schipperke","groenendael","malinois","briard","kelpie","komondor","Old_English_sheepdog","Shetland_sheepdog","collie","Border_collie","Bouvier_des_Flandres","Rottweiler","German_shepherd","Doberman","miniature_pinscher","Greater_Swiss_Mountain_dog","Bernese_mountain_dog","Appenzeller","EntleBucher","boxer","bull_mastiff","Tibetan_mastiff","French_bulldog","Great_Dane","Saint_Bernard","Eskimo_dog","malamute","Siberian_husky","dalmatian","affenpinscher","basenji","pug","Leonberg","Newfoundland","Great_Pyrenees","Samoyed","Pomeranian","chow","keeshond","Brabancon_griffon","Pembroke","Cardigan","toy_poodle","miniature_poodle","standard_poodle","Mexican_hairless","timber_wolf","white_wolf","red_wolf","coyote","dingo","dhole","African_hunting_dog","hyena","red_fox","kit_fox","Arctic_fox","grey_fox","tabby","tiger_cat","Persian_cat","Siamese_cat","Egyptian_cat","cougar","lynx","leopard","snow_leopard","jaguar","lion","tiger","cheetah","brown_bear","American_black_bear","ice_bear","sloth_bear","mongoose","meerkat","tiger_beetle","ladybug","ground_beetle","long-horned_beetle","leaf_beetle","dung_beetle","rhinoceros_beetle","weevil","fly","bee","ant","grasshopper","cricket","walking_stick","cockroach","mantis","cicada","leafhopper","lacewing","dragonfly","damselfly","admiral","ringlet","monarch","cabbage_butterfly","sulphur_butterfly","lycaenid","starfish","sea_urchin","sea_cucumber","wood_rabbit","hare","Angora","hamster","porcupine","fox_squirrel","marmot","beaver","guinea_pig","sorrel","zebra","hog","wild_boar","warthog","hippopotamus","ox","water_buffalo","bison","ram","bighorn","ibex","hartebeest","impala","gazelle","Arabian_camel","llama","weasel","mink","polecat","black-footed_ferret","otter","skunk","badger","armadillo","three-toed_sloth","orangutan","gorilla","chimpanzee","gibbon","siamang","guenon","patas","baboon","macaque","langur","colobus","proboscis_monkey","marmoset","capuchin","howler_monkey","titi","spider_monkey","squirrel_monkey","Madagascar_cat","indri","Indian_elephant","African_elephant","lesser_panda","giant_panda","barracouta","eel","coho","rock_beauty","anemone_fish","sturgeon","gar","lionfish","puffer","abacus","abaya","academic_gown","accordion","acoustic_guitar","aircraft_carrier","airliner","airship","altar","ambulance","amphibian","analog_clock","apiary","apron","ashcan","assault_rifle","backpack","bakery","balance_beam","balloon","ballpoint","Band_Aid","banjo","bannister","barbell","barber_chair","barbershop","barn","barometer","barrel","barrow","baseball","basketball","bassinet","bassoon","bathing_cap","bath_towel","bathtub","beach_wagon","beacon","beaker","bearskin","beer_bottle","beer_glass","bell_cote","bib","bicycle-built-for-two","bikini","binder","binoculars","birdhouse","boathouse","bobsled","bolo_tie","bonnet","bookcase","bookshop","bottlecap","bow","bow_tie","brass","brassiere","breakwater","breastplate","broom","bucket","buckle","bulletproof_vest","bullet_train","butcher_shop","cab","caldron","candle","cannon","canoe","can_opener","cardigan","car_mirror","carousel","carpenters_kit","carton","car_wheel","cash_machine","cassette","cassette_player","castle","catamaran","CD_player","cello","cellular_telephone","chain","chainlink_fence","chain_mail","chain_saw","chest","chiffonier","chime","china_cabinet","Christmas_stocking","church","cinema","cleaver","cliff_dwelling","cloak","clog","cocktail_shaker","coffee_mug","coffeepot","coil","combination_lock","computer_keyboard","confectionery","container_ship","convertible","corkscrew","cornet","cowboy_boot","cowboy_hat","cradle","crane","crash_helmet","crate","crib","Crock_Pot","croquet_ball","crutch","cuirass","dam","desk","desktop_computer","dial_telephone","diaper","digital_clock","digital_watch","dining_table","dishrag","dishwasher","disk_brake","dock","dogsled","dome","doormat","drilling_platform","drum","drumstick","dumbbell","Dutch_oven","electric_fan","electric_guitar","electric_locomotive","entertainment_center","envelope","espresso_maker","face_powder","feather_boa","file","fireboat","fire_engine","fire_screen","flagpole","flute","folding_chair","football_helmet","forklift","fountain","fountain_pen","four-poster","freight_car","French_horn","frying_pan","fur_coat","garbage_truck","gasmask","gas_pump","goblet","go-kart","golf_ball","golfcart","gondola","gong","gown","grand_piano","greenhouse","grille","grocery_store","guillotine","hair_slide","hair_spray","half_track","hammer","hamper","hand_blower","hand-held_computer","handkerchief","hard_disc","harmonica","harp","harvester","hatchet","holster","home_theater","honeycomb","hook","hoopskirt","horizontal_bar","horse_cart","hourglass","iPod","iron","jack-o-lantern","jean","jeep","jersey","jigsaw_puzzle","jinrikisha","joystick","kimono","knee_pad","knot","lab_coat","ladle","lampshade","laptop","lawn_mower","lens_cap","letter_opener","library","lifeboat","lighter","limousine","liner","lipstick","Loafer","lotion","loudspeaker","loupe","lumbermill","magnetic_compass","mailbag","mailbox","maillot","maillot","manhole_cover","maraca","marimba","mask","matchstick","maypole","maze","measuring_cup","medicine_chest","megalith","microphone","microwave","military_uniform","milk_can","minibus","miniskirt","minivan","missile","mitten","mixing_bowl","mobile_home","Model_T","modem","monastery","monitor","moped","mortar","mortarboard","mosque","mosquito_net","motor_scooter","mountain_bike","mountain_tent","mouse","mousetrap","moving_van","muzzle","nail","neck_brace","necklace","nipple","notebook","obelisk","oboe","ocarina","odometer","oil_filter","organ","oscilloscope","overskirt","oxcart","oxygen_mask","packet","paddle","paddlewheel","padlock","paintbrush","pajama","palace","panpipe","paper_towel","parachute","parallel_bars","park_bench","parking_meter","passenger_car","patio","pay-phone","pedestal","pencil_box","pencil_sharpener","perfume","Petri_dish","photocopier","pick","pickelhaube","picket_fence","pickup","pier","piggy_bank","pill_bottle","pillow","ping-pong_ball","pinwheel","pirate","pitcher","plane","planetarium","plastic_bag","plate_rack","plow","plunger","Polaroid_camera","pole","police_van","poncho","pool_table","pop_bottle","pot","potters_wheel","power_drill","prayer_rug","printer","prison","projectile","projector","puck","punching_bag","purse","quill","quilt","racer","racket","radiator","radio","radio_telescope","rain_barrel","recreational_vehicle","reel","reflex_camera","refrigerator","remote_control","restaurant","revolver","rifle","rocking_chair","rotisserie","rubber_eraser","rugby_ball","rule","running_shoe","safe","safety_pin","saltshaker","sandal","sarong","sax","scabbard","scale","school_bus","schooner","scoreboard","screen","screw","screwdriver","seat_belt","sewing_machine","shield","shoe_shop","shoji","shopping_basket","shopping_cart","shovel","shower_cap","shower_curtain","ski","ski_mask","sleeping_bag","slide_rule","sliding_door","slot","snorkel","snowmobile","snowplow","soap_dispenser","soccer_ball","sock","solar_dish","sombrero","soup_bowl","space_bar","space_heater","space_shuttle","spatula","speedboat","spider_web","spindle","sports_car","spotlight","stage","steam_locomotive","steel_arch_bridge","steel_drum","stethoscope","stole","stone_wall","stopwatch","stove","strainer","streetcar","stretcher","studio_couch","stupa","submarine","suit","sundial","sunglass","sunglasses","sunscreen","suspension_bridge","swab","sweatshirt","swimming_trunks","swing","switch","syringe","table_lamp","tank","tape_player","teapot","teddy","television","tennis_ball","thatch","theater_curtain","thimble","thresher","throne","tile_roof","toaster","tobacco_shop","toilet_seat","torch","totem_pole","tow_truck","toyshop","tractor","trailer_truck","tray","trench_coat","tricycle","trimaran","tripod","triumphal_arch","trolleybus","trombone","tub","turnstile","typewriter_keyboard","umbrella","unicycle","upright","vacuum","vase","vault","velvet","vending_machine","vestment","viaduct","violin","volleyball","waffle_iron","wall_clock","wallet","wardrobe","warplane","washbasin","washer","water_bottle","water_jug","water_tower","whiskey_jug","whistle","wig","window_screen","window_shade","Windsor_tie","wine_bottle","wing","wok","wooden_spoon","wool","worm_fence","wreck","yawl","yurt","web_site","comic_book","crossword_puzzle","street_sign","traffic_light","book_jacket","menu","plate","guacamole","consomme","hot_pot","trifle","ice_cream","ice_lolly","French_loaf","bagel","pretzel","cheeseburger","hotdog","mashed_potato","head_cabbage","broccoli","cauliflower","zucchini","spaghetti_squash","acorn_squash","butternut_squash","cucumber","artichoke","bell_pepper","cardoon","mushroom","Granny_Smith","strawberry","orange","lemon","fig","pineapple","banana","jackfruit","custard_apple","pomegranate","hay","carbonara","chocolate_sauce","dough","meat_loaf","pizza","potpie","burrito","red_wine","espresso","cup","eggnog","alp","bubble","cliff","coral_reef","geyser","lakeside","promontory","sandbar","seashore","valley","volcano","ballplayer","groom","scuba_diver","rapeseed","daisy","yellow_ladys_slipper","corn","acorn","hip","buckeye","coral_fungus","agaric","gyromitra","stinkhorn","earthstar","hen-of-the-woods","bolete","ear","toilet_tissue"]

def convert(size, box):
    dw = 1./(size[0])
    dh = 1./(size[1])
    x = (box[0] + box[1])/2.0 - 1
    y = (box[2] + box[3])/2.0 - 1
    w = box[1] - box[0]
    h = box[3] - box[2]
    x = x*dw
    w = w*dw
    y = y*dh
    h = h*dh
    return (x,y,w,h)

def convert_annotation(year, image_id):
    in_file = open('VOCdevkit/VOC%s/Annotations/%s.xml'%(year, image_id))
    out_file = open('VOCdevkit/VOC%s/labels/%s.txt'%(year, image_id), 'w')
    tree=ET.parse(in_file)
    root = tree.getroot()
    size = root.find('size')
    w = int(size.find('width').text)
    h = int(size.find('height').text)

    for obj in root.iter('object'):
        difficult = obj.find('difficult').text
        cls = obj.find('name').text
        if cls not in classes or int(difficult)==1:
            continue
        cls_id = classes.index(cls)
        xmlbox = obj.find('bndbox')
        b = (float(xmlbox.find('xmin').text), float(xmlbox.find('xmax').text), float(xmlbox.find('ymin').text), float(xmlbox.find('ymax').text))
        bb = convert((w,h), b)
        out_file.write(str(cls_id) + " " + " ".join([str(a) for a in bb]) + '\n')

wd = getcwd()

for year, image_set in sets:
    if not os.path.exists('VOCdevkit/VOC%s/labels/'%(year)):
        os.makedirs('VOCdevkit/VOC%s/labels/'%(year))
    image_ids = open('VOCdevkit/VOC%s/ImageSets/Main/%s.txt'%(year, image_set)).read().strip().split()
    list_file = open('%s_%s.txt'%(year, image_set), 'w')
    for image_id in image_ids:
        list_file.write('%s/VOCdevkit/VOC%s/JPEGImages/%s.JPEG\n'%(wd, year, image_id))
        convert_annotation(year, image_id)
    list_file.close()

#os.system("cat 2007_train.txt 2007_val.txt 2012_train.txt 2012_val.txt > train.txt")
#os.system("cat 2007_train.txt 2007_val.txt 2007_test.txt 2012_train.txt 2012_val.txt > train.all.txt")


```

修改好后在该目录下运行命令：python voc_label.py，之后则在文件夹scripts\VOCdevkit\VOC2007下生成了文件夹lable。这里包含了类别和对应归一化后的位置（i guess，如有错请指正）。同时在scripts\下应该也生成了train_2007.txt这个文件，里面包含了所有训练样本的绝对路径。

修改.data文件，以cfg/voc.data为例：

```bash
  1 classes= 20
  2 train  = <path-to-voc>/train.txt
  3 valid  = <path-to-voc>2007_test.txt
  4 names = data/voc.names
  5 backup = backup
```

使用自己的路径替换<path-to-voc>，例如我的配置如下：

```bash
classes= 1000
train  = /home/teng/programmings/semap/darknet/scripts/2007_train.txt
names = /home/teng/programmings/semap/darknet/data/voc.names
backup = /home/teng/programmings/semap/darknet/backup
```

修改.names文件，即训练的label：

```bash
tench
goldfish
great_white_shark
...
bolete
ear
toilet_tissue
```

# 配置文件修改

做好了上述准备，就可以根据不同的网络设置（cfg文件）来训练了。在文件夹cfg中有很多cfg文件，应该跟caffe中的prototxt文件是一个意思。这里以yolo-voc.cfg为例，主要修改参数如下：

```bash
[net]
# Testing
# batch=1
# subdivisions=1    #训练时候把上面Testing的参数注释
# Training
batch=64
subdivisions=32     #这个参数根据自己GPU的显存进行修改，显存不够就改大一些
...                 #因为训练时每批的数量 = batch/subdivisions
...
...
learning_rate=0.001  #根据自己的需求还有训练速度学习率可以调整一下
burn_in=1000
max_batches = 30000  #根据自己的需求还有训练速度max_batches可以调整一下
policy=steps
steps=10000,20000    #跟着max_batches做相应调整
...
...
...
[convolutional]
size=1
stride=1
pad=1
filters=30         #filters = 3*(classes + 5)
activation=linear

[yolo]
mask = 0,1,2
anchors = 10,13,  16,30,  33,23,  30,61,  62,45,  59,119,  116,90,  156,198,  373,326
classes=5          #修改类别数
num=9
jitter=.3
ignore_thresh = .5
truth_thresh = 1
random=1           #显存小的话 =0
#这个文件的最下面有3个YOLO层，这里我才放上来了一个，这三个地方的classes做相应修改
#每个YOLO层的上一层的convolutional层的filters也要修改
```

# 开始训练

下载预训练模型（权重）：

```bash
wget https://pjreddie.com/media/files/darknet53.conv.74
```

训练：

```bash
./darknet detector train cfg/voc.data cfg/yolov3-voc.cfg darknet53.conv.74
```

# 训练过程参数详解

Region xx: cfg文件中yolo-layer的索引；

Avg IOU:当前迭代中，预测的box与标注的box的平均交并比，越大越好，期望数值为1；

Class: 标注物体的分类准确率，越大越好，期望数值为1；

obj: 越大越好，期望数值为1；

No obj: 越小越好；

.5R: 以IOU=0.5为阈值时候的recall; recall = 检出的正样本/实际的正样本

0.75R: 以IOU=0.75为阈值时候的recall;

count:正样本数目。

# 参考：

- https://pjreddie.com/darknet/yolo/
- https://blog.csdn.net/lilai619/article/details/79695109
- https://zhuanlan.zhihu.com/p/35490655
- https://blog.csdn.net/ch_liu23/article/details/53558549




