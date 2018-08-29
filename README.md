# EAST: An Efficient and Accurate Scene Text Detector

### Introduction
This is a tensorflow re-implementation of [EAST: An Efficient and Accurate Scene Text Detector](https://arxiv.org/abs/1704.03155v2).
The features are summarized blow:

+ Only **RBOX** part is implemented.
+ A fast Locality-Aware NMS in C++ provided by the paper's author.
+ The pre-trained model provided achieves **80.83** F1-score on ICDAR 2015
	Incidental Scene Text Detection Challenge using only training images from ICDAR 2015 and 2013.
  see [here](http://rrc.cvc.uab.es/?ch=4&com=evaluation&view=method_samples&task=1&m=29855&gtv=1) for the detailed results.
+ Differences from original paper
	+ Use ResNet-50 rather than PVANET
	+ Use dice loss (optimize IoU of segmentation) rather than balanced cross entropy
	+ Use linear learning rate decay rather than staged learning rate decay
+ Speed on 720p (resolution of 1280x720) images:
	+ Now
		+ Graphic card: GTX 1080 Ti
		+ Network fprop: **~50 ms**
		+ NMS (C++): **~6ms**
		+ Overall: **~16 fps**
	+ Then
		+ Graphic card: K40
		+ Network fprop: ~150 ms
		+ NMS (python): ~300ms
		+ Overall: ~2 fps
		
Thanks for the author's ([@zxytim](https://github.com/zxytim)) help!
Please cite his [paper](https://arxiv.org/abs/1704.03155v2) if you find this useful.

### Contents
1. [Installation](#installation)
2. [Download](#download)
3. [Prepare dataset/pretrain](#dataset)
4. [Test](#train)
5. [Train](#test)
6. [Examples](#examples)
7. [Hmean](#hmean)

### Installation
1. Any version of tensorflow version > 1.0 should be ok.

### Download
1. Models trained on ICDAR 2013 (training set) + ICDAR 2015 (training set): [BaiduYun link](http://pan.baidu.com/s/1jHWDrYQ) [GoogleDrive](https://drive.google.com/open?id=0B3APw5BZJ67ETHNPaU9xUkVoV0U)
2. Resnet V1 50 provided by tensorflow slim: [slim resnet v1 50](http://download.tensorflow.org/models/resnet_v1_50_2016_08_28.tar.gz)

### Prepare dataset/pretrain weight
1. dataset 
+ -- trainset  ./data/ocr/train/img_###.jpg and img_###.txt (mixed) 
+ -- testset   ./data/ocr/test/img_###.jpg (jpg only)
2. pretrained  
-- if pretrained, use '--resume' in run.sh and set 49491.ckpt in /tmp/east_icdar2015_resnet_v1_50_rbox. and ./east_icdar2015_resnet_v1_50_rbox
3. if not pretrained , not use '--resume', and download resnet_v1_50.ckpt at ./

### Train
If you want to train the model, you should provide the dataset path, in the dataset path, a separate gt text file should be provided for each image
and run

```
python multigpu_train.py --gpu_list=0 --input_size=512 --batch_size_per_gpu=14 --checkpoint_path=/tmp/east_icdar2015_resnet_v1_50_rbox/ \
--text_scale=512 --training_data_path=/data/ocr/icdar2015/ --geometry=RBOX --learning_rate=0.0001 --num_readers=24 \
--pretrained_model_path=/tmp/resnet_v1_50.ckpt
```

If you have more than one gpu, you can pass gpu ids to gpu_list(like --gpu_list=0,1,2,3)

**Note: you should change the gt text file of icdar2015's filename to img_\*.txt instead of gt_img_\*.txt(or you can change the code in icdar.py), and some extra characters should be removed from the file.
See the examples in training_samples/**


### Test
First, modify checkpoint_step in eval.py
run
```
python eval.py --test_data_path=/tmp/images/ --gpu_list=0 --checkpoint_path=/tmp/east_icdar2015_resnet_v1_50_rbox/ \
--output_dir=/tmp/
```

a text file will be then written to the output path.


### Examples
Here are some test examples on icdar2015, enjoy the beautiful text boxes!
![image_1](demo_images/img_2.jpg)
![image_2](demo_images/img_10.jpg)
![image_3](demo_images/img_14.jpg)
![image_4](demo_images/img_26.jpg)
![image_5](demo_images/img_75.jpg)

### F1 score 
+ modify step_###_model.ckpt in eval.py
+ sh test.sh  >>> raise two dirs in ./result/
   ./result contain {dir1: res_img.txt    dir2: res_img.jpg with box to visualization}
+ cd result/### && zip -r submit.zip ./* 
+ submit at [ICDAR15 My method website](http://rrc.cvc.uab.es/?ch=4&com=mymethods&task=1)
