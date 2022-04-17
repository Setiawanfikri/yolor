# YOLOR
implementation of paper - [You Only Learn One Representation: Unified Network for Multiple Tasks](https://arxiv.org/abs/2105.04206)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1NIghFeKQGFRVRPh553JaQXeIMlI1xh95#)

[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/you-only-learn-one-representation-unified/real-time-object-detection-on-coco)](https://paperswithcode.com/sota/real-time-object-detection-on-coco?p=you-only-learn-one-representation-unified)

![Unified Network](https://github.com/WongKinYiu/yolor/blob/main/figure/unifued_network.png)

<img src="https://github.com/WongKinYiu/yolor/blob/main/figure/performance.png" height="480">

To get the results on the table, please use [this branch](https://github.com/WongKinYiu/yolor/tree/paper).

## Training

Install YOLOR dependencies
<details><summary> <b>Expand</b> </summary>

* clone YOLOR repository
      
      <pre><code>!git clone https://github.com/roboflow-ai/yolor
      %cd yolor
      !git reset --hard eb3ef0b7472413d6740f5cde39beb1a2f5b8b5d1</code></pre>
  
* Install necessary dependencies
      
      <pre><code>!pip install -qr requirements.txt</code></pre>
  
* Install Mish CUDA
  
      <pre><code>!git clone https://github.com/JunnYu/mish-cuda
      %cd mish-cuda
      !git reset --hard 6f38976064cbcc4782f4212d7c0c5f6dd5e315a8
      !python setup.py build install
      %cd ..</code></pre>
  
* Install PyTorch Wavelets
      
      <pre><code>!git clone https://github.com/fbcotter/pytorch_wavelets
      %cd pytorch_wavelets
      !pip install .
      %cd ..</code></pre>

</details>

Colab environment
<details><summary> <b>Expand</b> </summary>
  
```
git clone https://github.com/WongKinYiu/yolor
cd yolor

# pip install required packages
pip install -qr requirements.txt

# install mish-cuda if you want to use mish activation
# https://github.com/thomasbrandon/mish-cuda
# https://github.com/JunnYu/mish-cuda
git clone https://github.com/JunnYu/mish-cuda
cd mish-cuda
python setup.py build install
cd ..

# install pytorch_wavelets if you want to use dwt down-sampling module
# https://github.com/fbcotter/pytorch_wavelets
git clone https://github.com/fbcotter/pytorch_wavelets
cd pytorch_wavelets
pip install .
cd ..
```

</details>

Prepare COCO dataset
<details><summary> <b>Expand</b> </summary>

```
cd /yolor
bash scripts/get_coco.sh
```

</details>

Prepare pretrained weight
<details><summary> <b>Expand</b> </summary>

```
cd /yolor
bash scripts/get_pretrain.sh
```

</details>

## Testing

[`yolor_p6.pt`](https://drive.google.com/file/d/1Tdn3yqpZ79X7R1Ql0zNlNScB1Dv9Fp76/view?usp=sharing)

```
python test.py --data data/coco.yaml --img 1280 --batch 32 --conf 0.001 --iou 0.65 --device 0 --cfg cfg/yolor_p6.cfg --weights yolor_p6.pt --name yolor_p6_val
```

You will get the results:

```
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.52510
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.70718
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.57520
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.37058
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.56878
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.66102
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.39181
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.65229
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.71441
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.57755
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.75337
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.84013
```

## Training

Single GPU training:

```
python train.py --batch-size 8 --img 1280 1280 --data coco.yaml --cfg cfg/yolor_p6.cfg --weights '' --device 0 --name yolor_p6 --hyp hyp.scratch.1280.yaml --epochs 300
```

Multiple GPU training:

```
python -m torch.distributed.launch --nproc_per_node 2 --master_port 9527 train.py --batch-size 16 --img 1280 1280 --data coco.yaml --cfg cfg/yolor_p6.cfg --weights '' --device 0,1 --sync-bn --name yolor_p6 --hyp hyp.scratch.1280.yaml --epochs 300
```

Training schedule in the paper:

```
python -m torch.distributed.launch --nproc_per_node 8 --master_port 9527 train.py --batch-size 64 --img 1280 1280 --data data/coco.yaml --cfg cfg/yolor_p6.cfg --weights '' --device 0,1,2,3,4,5,6,7 --sync-bn --name yolor_p6 --hyp hyp.scratch.1280.yaml --epochs 300
python -m torch.distributed.launch --nproc_per_node 8 --master_port 9527 tune.py --batch-size 64 --img 1280 1280 --data data/coco.yaml --cfg cfg/yolor_p6.cfg --weights 'runs/train/yolor_p6/weights/last_298.pt' --device 0,1,2,3,4,5,6,7 --sync-bn --name yolor_p6-tune --hyp hyp.finetune.1280.yaml --epochs 450
python -m torch.distributed.launch --nproc_per_node 8 --master_port 9527 train.py --batch-size 64 --img 1280 1280 --data data/coco.yaml --cfg cfg/yolor_p6.cfg --weights 'runs/train/yolor_p6-tune/weights/epoch_424.pt' --device 0,1,2,3,4,5,6,7 --sync-bn --name yolor_p6-fine --hyp hyp.finetune.1280.yaml --epochs 450
```

## Inference

[`yolor_p6.pt`](https://drive.google.com/file/d/1Tdn3yqpZ79X7R1Ql0zNlNScB1Dv9Fp76/view?usp=sharing)

```
python detect.py --source inference/images/horses.jpg --cfg cfg/yolor_p6.cfg --weights yolor_p6.pt --conf 0.25 --img-size 1280 --device 0
```

You will get the results:

![horses](https://github.com/WongKinYiu/yolor/blob/main/inference/output/horses.jpg)

## Citation

```
@article{wang2021you,
  title={You Only Learn One Representation: Unified Network for Multiple Tasks},
  author={Wang, Chien-Yao and Yeh, I-Hau and Liao, Hong-Yuan Mark},
  journal={arXiv preprint arXiv:2105.04206},
  year={2021}
}
```

## Acknowledgements

<details><summary> <b>Expand</b> </summary>

* [https://github.com/AlexeyAB/darknet](https://github.com/AlexeyAB/darknet)
* [https://github.com/WongKinYiu/PyTorch_YOLOv4](https://github.com/WongKinYiu/PyTorch_YOLOv4)
* [https://github.com/WongKinYiu/ScaledYOLOv4](https://github.com/WongKinYiu/ScaledYOLOv4)
* [https://github.com/ultralytics/yolov3](https://github.com/ultralytics/yolov3)
* [https://github.com/ultralytics/yolov5](https://github.com/ultralytics/yolov5)

</details>
