# Just Add $\pi$! Pose Induced Video Transformers for Understanding Activities of Daily Living. 

[[Paper]](https://arxiv.org/abs/2311.18840) [[Pretrained models]](https://github.com/dominickrei/pi-vit/?tab=readme-ov-file#testing)

![intro](intro_graphic.png)

This is the official code for the CVPR 2024 paper titled "Just Add $\pi$! Pose Induced Video Transformers for Understanding Activities of Daily Living"

[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/just-add-p-pose-induced-video-transformers/action-classification-on-toyota-smarthome)](https://paperswithcode.com/sota/action-classification-on-toyota-smarthome?p=just-add-p-pose-induced-video-transformers) [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/just-add-p-pose-induced-video-transformers/action-recognition-in-videos-on-ntu-rgbd)](https://paperswithcode.com/sota/action-recognition-in-videos-on-ntu-rgbd?p=just-add-p-pose-induced-video-transformers) [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/just-add-p-pose-induced-video-transformers/action-recognition-in-videos-on-ntu-rgbd-120)](https://paperswithcode.com/sota/action-recognition-in-videos-on-ntu-rgbd-120?p=just-add-p-pose-induced-video-transformers)

## Installation
First, create a conda environment and activate it:
```
conda create -n pivit python=3.7 -y
source activate pivit
```

Then, install the following packages:
- torch & torchvision `pip install torch===1.8.1+cu111 torchvision===0.9.1+cu111 -f https://download.pytorch.org/whl/torch_stable.html`
- [fvcore](https://github.com/facebookresearch/fvcore/): `pip install 'git+https://github.com/facebookresearch/fvcore'`
- PyAV: `conda install av -c conda-forge`
- misc: `pip install simplejson einops timm psutil scikit-learn opencv-python tensorboard`

Lastly, build the codebase by running:
```
git clone https://github.com/dominickrei/pi-vit
cd pi-vit
python setup.py build develop
```

## Data preparation
We make use of the following action recognition datasets for evaluation: [Toyota Smarthome](https://project.inria.fr/toyotasmarthome/), [NTU RGB+D](https://rose1.ntu.edu.sg/dataset/actionRecognition/), and [NTU RGB+D 120](https://rose1.ntu.edu.sg/dataset/actionRecognition/). Download the datasets from their respective sources and structure their directories in the following formats.

### Smarthome
```
├── Smarthome
    ├── mp4
        ├── Cook.Cleandishes_p02_r00_v02_c03.mp4
        ├── Cook.Cleandishes_p02_r00_v14_c03.mp4
        ├── ...
    ├── skeletonv12
        ├── Cook.Cleandishes_p02_r00_v02_c03_pose3d.json
        ├── Cook.Cleandishes_p02_r00_v14_c03_pose3d.json
        ├── ...
```

### NTU RGB+D
```
├── NTU
    ├── rgb
        ├── S001C001P001R001A001_rgb.avi
        ├── S001C001P001R001A001_rgb.avi
        ├── ...
    ├── skeletons
        ├── S001C001P001R001A001.skeleton.npy
        ├── S001C001P001R001A001.skeleton.npy
        ├── ...
```
* By default, the NTU skeletons are in MATLAB format. We convert them into Numpy format using code provided in https://github.com/shahroudy/NTURGB-D/tree/master/Python

### Preparing CSVs
After downloading and preparing the datasets, prepare the CSVs for training, testing, and validation splits as `train.csv`, `test.csv`, and `val.csv`. The format of each CSV is:
```
path_to_video_1,path_to_video_1_skeleton,label_1
path_to_video_2,path_to_video_2_skeleton,label_2
...
path_to_video_N,path_to_video_N_skeleton,label_N
```

## Usage
We provide configs for training $\pi$-ViT on Smarthome and NTU in [configs/](configs/). Please update the paths in the config to match the paths in your machine before using.

### Training
Download the necessary pretrained models (Kinetics-400 for Smarthome and SSv2 for NTU) from [this link](https://github.com/facebookresearch/TimeSformer?tab=readme-ov-file#model-zoo) and update `TRAIN.CHECKPOINT_FILE_PATH` to point to the downloaded model.

For example to train $\pi$-ViT on Smarthome using 8 GPUs run the following command:

`python tools/run_net.py --cfg configs/Smarthome/PIViT_Smarthome.yaml NUM_GPUS 8`

### Testing
| Model | Dataset | mCA | Top-1 | Downloads |
| --- | --- | --- | --- | --- |
$\pi$-ViT | Smarthome CS | 72.9 | - | [HuggingFace](https://huggingface.co/datasets/dreilly/pi-vit-checkpoints/resolve/main/SH_CS_pivit.pyth) |
$\pi$-ViT | Smarthome CV2 | 64.8 | - | [HuggingFace](https://huggingface.co/datasets/dreilly/pi-vit-checkpoints/resolve/main/SH_CV2_pivit.pyth) |
$\pi$-ViT | NTU-120 CS | - | 91.9 | [HuggingFace](https://huggingface.co/datasets/dreilly/pi-vit-checkpoints/blob/main/NTU120_CS_pivit.pyth) |
$\pi$-ViT | NTU-120 CSetup | - | 92.9 | [HuggingFace](https://huggingface.co/datasets/dreilly/pi-vit-checkpoints/blob/main/NTU120_CSet_pivit.pyth) |
$\pi$-ViT | NTU-60 CS | - | 94.0 | [HuggingFace](https://huggingface.co/datasets/dreilly/pi-vit-checkpoints/blob/main/NTU60_CS_pivit.pyth) |
$\pi$-ViT | NTU-60 CV | - | 97.9 | [HuggingFace](https://huggingface.co/datasets/dreilly/pi-vit-checkpoints/blob/main/NTU60_CV_pivit.pyth) |

After downloading a pretrained model, evaluate it using the command:

`python tools/run_net.py --cfg configs/Smarthome/PIViT_Smarthome.yaml NUM_GPUS 8 TEST.CHECKPOINT_FILE_PATH /path/to/downloaded/model TRAIN.ENABLE False`

## Setting up skeleton features for $\pi$-ViT

During training, the 3D-SIM module in $\pi$-ViT requires extracted features from a pre-trained sketon action recognition model. This means that for every video in the training set, there must be a corresponding feature vector associated with it. The features should be stored in the directory indicated by the config option  `EXPERIMENTAL.HYPERFORMER_FEATURES_PATH`.

$\pi$-ViT expects a directory containing a single HDF5 file for each video in the training dataset. For example, the directory structure for Smarthome should look like this:
```
├── /path/to/hyperformer_features
        ├── Cook.Cleandishes_p02_r00_v02_c03.h5
        ├── Cook.Cleandishes_p02_r00_v14_c03.h5
        ├── ...
```

Where `Cook.Cleandishes_p02_r00_v02_c03.h5` is a HDF5 file containing a single dataset named `data` with a shape of `400x216`. We provide a minimal example to demonstrate saving a feature vector in the format $\pi$-ViT expects:
```
skeleton_features = np.random.rand(400, 216)

with h5py.File('random_tensor.h5', 'w') as f:
    f.create_dataset('data', data=tensor)
```

Due to the large size of the skeleton feature datasets we do not upload them here, instead we provide the Hyperformer models pre-trained on Toyota-Smarthome in `hyperformer_models/`. NTU trained models, and details for executing the Hyperformer model, are available [here](https://github.com/ZhouYuxuanYX/Hyperformer).

## Citation & Acknowledgement
```
@article{reilly2024pivit,
    title={Just Add $\pi$! Pose Induced Video Transformers for Understanding Activities of Daily Living},
    author={Dominick Reilly and Srijan Das},
    booktitle={Proceedings of the Conference on Computer Vision and Pattern Recognition (CVPR)}
    year={2024}
}
```
Our primary contributions can be found in:
- [train_net.py](tools/train_net.py), [pivit.py](timesformer/models/pivit.py), [pivit_modules.py](timesformer/models/pivit_modules.py), [losses.py](timesformer/models/losses.py), [smarthome.py](timesformer/datasets/smarthome.py), [ntu.py](timesformer/datasets/ntu.py)

This repository is built on top of [TimeSformer](https://github.com/facebookresearch/TimeSformer).
