# DeePoint: Visual Pointing Recognition and Direction Estimation, ICCV 2023

This repository provides an implementation of our paper [DeePoint: Visual Pointing Recognition and Direction Estimation](https://openaccess.thecvf.com/content/ICCV2023/html/Nakamura_DeePoint_Visual_Pointing_Recognition_and_Direction_Estimation_ICCV_2023_paper.html) in ICCV 2023. If you use our code and data please cite our paper.

Please note that this is research software and may contain bugs or other issues – please use it at your own risk. If you experience major problems with it, you may contact us, but please note that we do not have the resources to deal with all issues.


```
@InProceedings{Nakamura_2023_ICCV,
	author    = {Shu Nakamura and Yasutomo Kawanishi and Shohei Nobuhara and Ko Nishino},
	title     = {DeePoint: Visual Pointing Recognition and Direction Estimation},
	booktitle = {Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)},
	month     = {October},
	year      = {2023},
}
```

![DeePoint Architecture](./img/DeePoint_architecture.jpg)

## Prerequisites
We tested our code with `python3.10` with external libraries including:
- `numpy`
- `opencv-python`
- `opencv-contrib-python`
- `torch`
- `torchvision`
- `pytorch-lightning`

For example:
```bash
pip install -r requirements.txt
conda install -c conda-forge PyOpenGL
conda install pytorch==1.13.1 torchvision==0.14.1 torchaudio==0.13.1 pytorch-cuda=11.7 -c pytorch -c nvidia
```

Please refer to [environment/pip_freeze.txt](environment/pip_freeze.txt) for the specific versions we used.  
You can also use `singularity` to replicate our environment:
```bash
singularity build environment/deepoint.sif environment/deepoint.def
singularity run --nv environment/deepoint.sif
```

## Usage

### Demo
You can download the pretrained model from [here](https://drive.google.com/file/d/1I887Y_G27sPf6QaFfMDTJoHVcTR-pTR_/view?usp=drive_link).
Download it and save the file as `models/weight.ckpt`.

You can apply the model on your video and visualize the result by running the script below.
```
python3 src/demo.py movie=./demo/example.mp4 lr=l ckpt=./models/weight.ckpt
```
- The video has to contain one person within the frame and the person had better shows the whole body in the frame.
- You need to specify the pointing hand (left or right) for visualization.
- Since this script uses OpenGL (`PyOpenGL` and `glfw`) to draw an 3D arrow, you need to have an window system for this to work.
	- We use the script that were used in [Gaze360](https://github.com/erkil1452/gaze360) for drawing 3D arrows.

### DP Dataset

![Examples of the DP Dataset](./img/DPDataset_examples.jpg)

You can download the DP Dataset from Google Drive. [link](https://drive.google.com/drive/folders/1W_49HId_2FLFH0X9Ry8QiTTyaVt2Y0ks)

#### License
The DP Dataset is distributed under the [Creative Commons Attribution-Noncommercial 4.0 International License (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/).

#### Training with the DP Dataset
After downloading the dataset, follow the instructions in [data/README.md](data/README.md).
The structure of `data` directory then should look like below:
```
deepoint
├── data
│   ├── README.md
│   ├── mount_frames.sh
│   ├── frames
│   │   ├── 2023-01-17-livingroom
│   │   └── ...
│   ├── labels
│   │   ├── 2023-01-17-livingroom
│   │   └── ...
│   └── keypoints
│       ├── collected_json.pickle
│       └── triangulation.pickle
└── ...
```
After downloading the dataset, run
```
python src/main.py task=train
```
to train the model. Refer to `conf/model` for configurations of the model.

#### Evaluation with the DP Dataset
After training, you can evaluate the model by running:

```
python src/main.py task=test ckpt=./path/to/the/model.ckpt
```


## Troubleshooting

### `PyOpenGL` error
1. shader GLSL version is not supported.
```bash
Traceback (most recent call last):
  File "/home/claire/Documents/Human_robot_interaction/deepoint4MLLM/src/demo.py", line 11, in <module>
    from draw_arrow import WIDTH, HEIGHT
  File "/home/claire/Documents/Human_robot_interaction/deepoint4MLLM/src/draw_arrow.py", line 23, in <module>
    VERTEX_SHADER = shaders.compileShader(
  File "/home/claire/miniconda3/envs/deepoint/lib/python3.10/site-packages/OpenGL/GL/shaders.py", line 235, in compileShader
    raise ShaderCompilationError(
OpenGL.GL.shaders.ShaderCompilationError: ("Shader compile failure (0): b'0:1(10): error: GLSL 3.30 is not supported. Supported versions are: 1.10, 1.20, 1.30, 1.40, 1.00 ES, and 3.00 ES\\n'", [b'#version 330\nlayout(location = 0) in vec4 position;\nout vec2 UV;\nvoid main()\n{\n  UV = position.xy*0.5+0.5;\n  gl_Position = position;\n}\n'], GL_VERTEX_SHADER)
```
Solution:

1. 通过设置环境变量 MESA_GL_VERSION_OVERRIDE=3.3，可以强制使用OpenGL 3.3版本
```bash
export MESA_GL_VERSION_OVERRIDE=3.3
```

## pytorch error
1. openpifpaf库或PyTorch的C++扩展
```bash
Error executing job with overrides: ['movie=./demo/example.mp4', 'lr=l', 'ckpt=./models/weight.ckpt']
Traceback (most recent call last):
  ...
  File "/home/claire/miniconda3/envs/deepoint/lib/python3.10/ctypes/__init__.py", line 374, in __init__
    self._handle = _dlopen(self._name, mode)
OSError: /home/claire/miniconda3/envs/deepoint/lib/python3.10/site-packages/openpifpaf/_cpp.so: undefined symbol: _ZN2at4_ops5zeros4callEN3c108ArrayRefINS2_6SymIntEEENS2_8optionalINS2_10ScalarTypeEEENS6_INS2_6LayoutEEENS6_INS2_6DeviceEEENS6_IbEE

Set the environment variable HYDRA_FULL_ERROR=1 for a complete stack trace.
```