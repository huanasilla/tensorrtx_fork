# TensorRTx

TensorRTx aims to implement popular deep learning networks with tensorrt network definition APIs. As we know, tensorrt has builtin parsers, including caffeparser, uffparser, onnxparser, etc. But when we use these parsers, we often run into some "unsupported operations or layers" problems, especially some state-of-the-art models are using new type of layers.

So why don't we just skip all parsers? We just use TensorRT network definition APIs to build the whole network, it's not so complicated.

I wrote this project to get familiar with tensorrt API, and also to share and learn from the community.

All the models are implemented in pytorch first, and export a weights file xxx.wts, and then use tensorrt to load weights, define network and do inference. Some pytorch implementations can be found in my repo [Pytorchx](https://github.com/wang-xinyu/pytorchx), the remaining are from polular open-source pytorch implementations. 

# News

- `22 May 2020`. A new branch [trt4](https://github.com/wang-xinyu/tensorrtx/tree/trt4) created, which is using TensorRT 4 API. Now the master branch is using TensorRT 7 API. But only `yolov4` has been migrated to TensorRT 7 API for now. The rest will be migrated soon. And a tutorial for `migarating from TensorRT 4 to 7` provided.

## Tutorials

- [A guide for quickly getting started, taking lenet5 as a demo.](./tutorials/getting_started.md)
- [Migrating from TensorRT 4 to 7](./tutorials/migrating_from_tensorrt_4_to_7.md)

## Test Environment

1. GTX1080 / Ubuntu16.04 / cuda10.0 / cudnn7.6.5 / tensorrt7.0.0 / nvinfer7.0.0 / opencv3.3

## How to run

Each folder has a readme inside, which explains how to run the models inside.

## Models

Following models are implemented.

|Name | Description |
|-|-|
|[lenet](./lenet) | the simplest, as a "hello world" of this project |
|[alexnet](./alexnet)| easy to implement, all layers are supported in tensorrt |
|[googlenet](./googlenet)| GoogLeNet (Inception v1) |
|[inception](./inceptionv3)| Inception v3 |
|[mnasnet](./mnasnet)| MNASNet with depth multiplier of 0.5 from the paper |
|[mobilenetv2](./mobilenetv2)| MobileNet V2 |
|[mobilenetv3](./mobilenetv3)| V3-small, V3-large. |
|[resnet](./resnet)| resnet-18, resnet-50 and resnext50-32x4d are implemented |
|[senet](./senet)| se-resnet50 |
|[shufflenet](./shufflenetv2)| ShuffleNetV2 with 0.5x output channels |
|[squeezenet](./squeezenet)| SqueezeNet 1.1 model |
|[vgg](./vgg)| VGG 11-layer model |
|[yolov3](./yolov3)| darknet-53, weights from yolov3 authors, pytorch implementation from [ayooshkathuria/pytorch-yolo-v3](https://github.com/ayooshkathuria/pytorch-yolo-v3) |
|[yolov3-spp](./yolov3-spp)| darknet-53, weights from [ultralytics/yolov3](https://github.com/ultralytics/yolov3) |
|[yolov4](./yolov4)| CSPDarknet53, weights from [AlexeyAB/darknet](https://github.com/AlexeyAB/darknet#pre-trained-models), pytorch implementation from [ultralytics/yolov3](https://github.com/ultralytics/yolov3) |
|[retinaface](./retinaface)| resnet-50, weights from [biubug6/Pytorch_Retinaface](https://github.com/biubug6/Pytorch_Retinaface) |

## Tricky Operations

Some tricky operations encountered in these models, already solved, but might have better solutions.

|Name | Description |
|-|-|
|BatchNorm| Implement by a scale layer, used in resnet, googlenet, mobilenet, etc. |
|MaxPool2d(ceil_mode=True)| use a padding layer before maxpool to solve ceil_mode=True, see googlenet. |
|average pool with padding| use setAverageCountExcludesPadding() when necessary, see inception. |
|relu6| use `Relu6(x) = Relu(x) - Relu(x-6)`, see mobilenet. |
|torch.chunk()| implement the 'chunk(2, dim=C)' by tensorrt plugin, see shufflenet. |
|channel shuffle| use two shuffle layers to implement `channel_shuffle`, see shufflenet. |
|adaptive pool| use fixed input dimension, and use regular average pooling, see shufflenet. |
|leaky relu| I wrote a leaky relu plugin, but PRelu in `NvInferPlugin.h` can be used, see yolov3. |
|yolo layer v1| yolo layer is implemented as a plugin, see yolov3. |
|yolo layer v2| three yolo layers implemented in one plugin, see yolov3-spp. |
|upsample| replaced by a deconvolution layer, see yolov3. |
|hsigmoid| hard sigmoid is implemented as a plugin, hsigmoid and hswish are used in mobilenetv3 |
|retinaface output decode| implement a plugin to decode bbox, confidence and landmarks, see retinaface. |
|mish| mish activation is implemented as a plugin, mish is used in yolov4 |

## Speed Benchmark

| Models | Device | BatchSize | Mode | Input Shape(HxW) | FPS |
|-|-|:-:|:-:|:-:|:-:|
| YOLOv3(darknet53) | Xavier | 1 | FP16 | 320x320 | 55 |
| YOLOv3-spp(darknet53) | Xeon E5-2620/GTX1080 | 1 | FP32 | 256x416 | 94 |
| YOLOv4(CSPDarknet53) | Xeon E5-2620/GTX1080 | 1 | FP16 | 608x608 | 35.7 |
| YOLOv4(CSPDarknet53) | Xeon E5-2620/GTX1080 | 4 | FP16 | 608x608 | 40.9 |
| YOLOv4(CSPDarknet53) | Xeon E5-2620/GTX1080 | 8 | FP16 | 608x608 | 41.3 | 
| RetinaFace(resnet50) | TX2 | 1 | FP16 | 384x640 | 15 |
| RetinaFace(resnet50) | Xeon E5-2620/GTX1080 | 1 | FP32 | 928x1600 | 15 |

Detection net FPS test including inference and nms time, excluding image preprocess time.

Help wanted, if you got speed results, please add an issue or PR.

Thanks @Kmarconi for yolov3(darknet53) speed test.

## Acknowledgments & Contact

Currently, This repo is funded by Alleyes-THU AI Lab([aboutus in Chinese](http://www.alleyes.com.cn/aboutus.html)). We are based in Tsinghua University, Beijing, and seeking for talented interns for CV R&D. Contact me if you are interested.

Any contributions, questions and discussions are welcomed, contact me by following info.

E-mail: wangxinyu_es@163.com

WeChat ID: wangxinyu0375
