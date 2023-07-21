# VideoINR

This repository contains the official implementation for VideoINR introduced in the following paper:

[**VideoINR: Learning Video Implicit Neural Representation for Continuous
Space-Time Super-Resolution**](http://zeyuan-chen.com/VideoINR/) \
[Zeyuan Chen](http://zeyuan-chen.com/),
[Yinbo Chen](https://yinboc.github.io/),
Jingwen Liu
[Xingqian Xu](https://scholar.google.com/citations?user=s1X82zMAAAAJ&hl=en),
[Vidit Goel](https://vidit98.github.io/),
[Zhangyang Wang](https://express.adobe.com/page/CAdrFMJ9QeI2y),
[Humphrey Shi](https://www.humphreyshi.com/),
[Xiaolong Wang](https://xiaolonw.github.io/)

**CVPR 2022**

You can find more visual results and a brief introduction to VideoINR at our
[project page](http://zeyuan-chen.com/VideoINR/).

## Method Overview

<img src="images/pipeline.png" width=100%>

Two consecutive input frames are concatenated and encoded as a discrete feature map. Based on the feature, the spatial and temporal implicit neural representations decode a 3D space-time coordinate to a motion flow vector. We then sample a new feature vector by warping according to the motion flow, and decode it as the RGB prediction of the query coordinate.

## Citation

If you find our work useful in your research, please cite:

```bibtex
@inproceedings{chen2022vinr,
  author    = {Chen, Zeyuan and Chen, Yinbo and Liu, Jingwen and Xu, Xingqian and Goel, Vidit and Wang, Zhangyang and Shi, Humphrey and Wang, Xiaolong},
  title     = {VideoINR: Learning Video Implicit Neural Representation for Continuous Space-Time Super-Resolution},
  journal   = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition},
  year      = {2022},
}
```

## Environmental Setup

The code is tested in:

- Windows 10
- Python 3.11
- Pytorch 2.0.1 (latest)
- torchvision 0.15.2
- pytorch-cuda 11.8
- [Deformable Convolution v2](https://arxiv.org/abs/1811.11168). Following
  [Zooming Slowmo](https://github.com/Mukosame/Zooming-Slow-Mo-CVPR-2020),
  we adopt [CharlesShang's DCNv2](https://github.com/CharlesShang/DCNv2)
  in the submodule *[now updated to [lucasjinreal's DCNv2_latest
  ](https://github.com/lucasjinreal/DCNv2_latest)]*.

It is **highly** recommended to use [Anaconda](https://www.anaconda.com/download) to build this environment:

```sh
# install dependencies
conda env create -f environment.yml
conda activate videoinr
```

This will install all conda and pip dependencies, including the local module
DCNv2.

### Note

- Must have CUDA-compatible GPUs with latest drivers installed.
- **No need** to install CUDA toolkit on system, packages in environment.yml are
  sufficient.
- **Windows**: Must install MSVC on system
  (VS Build Tools 2017-2022 for CUDA 11.8).
  Must also install nvcc (uncomment cuda-nvcc dependency in environment.yml).
- **Linux**: Must install gcc & g++ (uncomment gxx dependency in environment.yml).

For an optimal installation please [update conda and install and set the new
dependency solver libmamba](https://www.anaconda.com/blog/a-faster-conda-for-a-growing-community).

## Demo

1. Download the pre-trained model from [google drive
   ](https://drive.google.com/file/d/1JW_-ef_oHAmPZkssiXvOspIAn_poymK9/view?usp=sharing).

2. Convert your video of interest to a sequence of images.
   This process can be completed by many apps, *e.g.* ffmpeg and AdobePR.
   The folder that contains this image sequence should have a structure as follows:

    ```txt
    data_path
    ├── img_1.png
    ├── img_2.png
    ├── ...
    ├── img_n.png
    ```

3. Using VideoINR for performing space-time super-resolution.
   You can adjust up-sampling scales by setting different `space_scale` and `time_scale`.

    ```sh
    python demo.py --space_scale 4 --time_scale 8 --data_path [YOUR_DATA_PATH]
    ```

4. The output would be three folders including low-resolution images, bicubic-upsampling images, and the results of VideoINR.

## Preparing Dataset

We use the [Adobe240 dataset](https://www.cs.ubc.ca/labs/imager/tr/2017/DeepVideoDeblurring/) for training.

1. Download the zip file [here](https://www.cs.ubc.ca/labs/imager/tr/2017/DeepVideoDeblurring/DeepVideoDeblurring_Dataset_Original_High_FPS_Videos.zip) which contains the original high FPS videos.

2. In order to extract frames of each video to a separated folder, change `videoFolder` to where you save the extracted frames and `frameFolder` to `DATASET_PATH` in `generate_frames_from_adobe240fps.py` and run it. This would automatically split the data into train/test/val set.

    ```sh
    python generate_frames_from_adobe240fps.py
    ```

## Training

1. Configure training settings, which can be found at options/train. The default training setting can be found at train_zsm.yml. You need to change a few lines in the config file in order to run successfully in your machine:
    - **name & mode (Line 12 & 13)**: As mentioned in the paper, we adopt a two-stage training strategy, so there exists two different modes for training set. `Adobe` and `Adobe_a` (refer to Line 47 in data/init.py). `Adobe` fixs the down-sampling scale to 4 while `Adobe_a` randomly samples down-sampling scales in [2, 4]. For the first stage (0 - 450000 iterations), we set the name & mode to `Adobe`. For the second stage (450000 - 600000 iterations), we set the name & mode to `Adobe_a`.
    - **dataroot_GT & dataroot_LQ (Line 17 & 18)**: Path to the Adobe240 dataset. Set them as `DATASET_PATH/train` (dataroot_LQ is not used in current implementation)
    - **models & training_state (Line 47 & 48)**: Path to where you want to save the model parameters and training state (for restart training).

2. Run training code. The default setting needs four RTX 2080Ti for training. Note that for applying the two-stage training strategy, you might have to run train.py twice.

    ```sh
    python train.py -opt options/train/train_zsm.yml
    ```

## Additional Note

Throughout the training process, we calculate the loss by summing distances of all pixels between prediction and ground-truth. However, this can be unreasonable for stage 2 (450000 - 600000 iterations) since the ground-truth images have different resolutions, resulting in different loss scales. Using mean distances for the loss value in stage 2 can be helpful for the final model performance.

Thank @[sichun233746](https://github.com/sichun233746) very much for his testing!

## Acknowledgments

Our code is built on [Zooming-Slow-Mo-CVPR-2020](https://github.com/Mukosame/Zooming-Slow-Mo-CVPR-2020) and [LIIF](https://github.com/yinboc/liif). Thank the authors for sharing their codes!
