# ACMMM21-VIShv
Source codes of ACMMM21 paper: Learning Hierarchical Embeddings for Video Instance Segmentation[https://dl.acm.org/doi/10.1145/3474085.3475342]. 
In this paper, we propose a normalizing flow based generative method for video instance segmentation.
#
![](framework.png)

## Pre-requisites

* Python >= 3.7
* PyTorch >= 1.4
* OpenCV, numpy, imgaug, pillow, tqdm, pyyaml, tensorboardX, scipy, pycocotools (see `requirements.txt` for exact versions in case you encounter issues)

## Basic Setup

1. Clone the repository and append it to the `PYTHONPATH` variable:

   ```bash
   git clone https://github.com/carrierlxk/ACMMM21-VIShv.git
   cd ACMM21-VIShv
   export PYTHONPATH=$(pwd):$PYTHONPATH
   ```

   
2. Download the required datasets from their respective websites and the trained model checkpoints from the given [links](https://pan.baidu.com/s/1Lrv3tBqY9Lcr8wZPEQPO8Q?pwd=0oby), password：0oby. For inference, you only need the validation sets of the target dataset. For training, the table below shows which dataset(s) you will need:

    | Target Dataset        | Datasets Required for Training  | 
    |-----------------------| -------------------------------|
    | [DAVIS](https://data.vision.ee.ethz.ch/csergi/share/davis/DAVIS-2017-Unsupervised-trainval-480p.zip)                 | DAVIS'17, YouTubeVIS, COCO Instance Segmentation, PascalVOC 
    | [YouTube-VIS19](https://competitions.codalab.org/competitions/20128#participate-get_data)           | YouTube-VIS, COCO Instance Segmentation, PascalVOC 
    | [YouTube-VIS21](https://competitions.codalab.org/competitions/20128#participate-get_data)           | YouTube-VIS, COCO Instance Segmentation, PascalVOC 


## Environment Variables

File paths to datasets and model checkpoints are configured using environment variables.

### Required
   
1. `STEMSEG_JSON_ANNOTATIONS_DIR`: To streamline the code, we reorganized the annotations and file paths for every dataset into a standard JSON format. These JSON files can be downloaded from [here](https://omnomnom.vision.rwth-aachen.de/data/STEm-Seg/dataset_jsons/). Set this variable to the directory holding these JSON files.

2. `STEMSEG_MODELS_DIR`: Base directory where models are saved to by default. Only required for training. You can initially point this to any empty directory.

   
   
### Dataset Specific

For inference, you only need to set the relevant variable for the target dataset. For training, since multiple datasets are used, multiple variables will be required (as mentioned below).

#### Video Datasets
   
1. `DAVIS_BASE_DIR`: Set this to the full path of the `JPEGImages/480p` directory for the DAVIS dataset. The image frames for all 60 training and 30 validation videos should be present in the directory. This variable is required for training/inference on DAVIS'19 Unsupervised.
  
2. `YOUTUBE_VIS_BASE_DIR`: Set this to the parent directory of the `train` and `val` directories for the YouTube-VIS dataset. This variable is required for training/inference on YouTube-VIS and also for training for DAVIS.

#### Image Datasets (required only for training)

4. `COCO_TRAIN_IMAGES_DIR`: Set this to the `train2017` directory of the COCO instance segmentation dataset. Remember to use the 2017 train/val split and not the 2014 one. This variable is required for training for DAVIS and YouTube-VIS.
   
5. `PASCAL_VOC_IMAGES_DIR`: Set this to the `JPEGImages` directory of the PascalVOC dataset.This variable is required for training for DAVIS and YouTube-VIS. 
   
6. `MAPILLARY_IMAGES_DIR`: You will need to do two extra things here: (1) Put all the training and validation images into a single directory (18k + 2k = 20k images in total). (ii) Since Mapillary images are very large, we first down-sampled them. The expected size for each image is given in `stemseg/data/metainfo/mapillary_image_dims.json` as a dictionary from the image file name to a (width, height) tuple. Please use OpenCV's `cv2.resize` method with `interpolation=cv2.INTER_LINEAR` to ensure the best consistency between your down-sampled images and the annotations we provide in our JSON file. This variable is required for training for KITTI-MOTS.


## Inference

Assuming the relevant dataset environment variables are correctly set, just run the following commands:

1. DAVIS:

    ```bash
    python stemseg/inference/main.py /path/to/downloaded/checkpoints/davis.pth -o /path/to/output_dir --dataset davis
    ```
    
2. YouTube-VIS:

    ```bash
    python stemseg/inference/main.py /path/to/downloaded/checkpoints/youtube_vis.pth -o /path/to/output_dir --dataset ytvis --resize_embeddings
    ```
    
For each dataset, the output written to `/path/to/output_dir` will be in the same format as that required for the official evaluation tool for each dataset. To obtain visualizations of the generated segmentation masks, you can add a `--save_vis` flag to the above commands.


## Training

1. Make sure the required environment variables are set as mentioned in the above sections. 

2. Run `mkdir $STEMSEG_MODELS_DIR/pretrained` and place the [pre-trained backbone file](https://omnomnom.vision.rwth-aachen.de/data/STEm-Seg/models/mask_rcnn_R_101_FPN_backbone.pth) in this directory.

### DAVIS

The final inference reported in the paper is done using clips of length 16 frames. Training end-to-end with such lengthy clips requires too much GPU VRAM though, so we train in two steps:

1. First we train end-to-end with 8 frame long clips:
   
   ```bash 
   python stemseg/training/main.py --model_dir some_dir_name --cfg davis_1.yaml
   ```
   
2. Then we freeze the encoder network (backbone and FPN) and train only the decoders with 16 frame long clips:

   ```bash 
   python stemseg/training/main.py --model_dir another_dir_name --cfg davis_2.yaml --initial_ckpt /path/to/last/ckpt/from/previous/step.pth
   ```
   
The training code creates a directory at `$STEMSEG_MODELS_DIR/checkpoints/DAVIS/some_dir_name` and places all checkpoints and logs for that training session inside it. For the second step we want to restore the final weights from the first step, hence the additional `--initial_ckpt` argument.

### YouTube-VIS

Here, the final inference was done on 8 frame clips, so the model can be trained in a single step:

```bash 
python stemseg/training/main.py --model_dir some_dir_name --cfg youtube_vis.yaml
```

The training output for this will be placed in `$STEMSEG_MODELS_DIR/checkpoints/youtube_vis/some_dir_name`.

```
@inproceedings{qin2021mm, 
  author = {Qin, Zheyun and Lu, Xiankai and Nie, Xiushan and Zhen, Xiantong and Yin, Yinlong},
  title = {Learning Hierarchical Embeddings for Video Instance Segmentation},
  booktitle = {ACM Multimedia},
  year = {2021}
}
```
