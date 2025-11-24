# BrepGen: 具有结构化潜在几何的 B-rep 生成扩散模型 (SIGGRAPH 2024)

[![arXiv](https://img.shields.io/badge/📃-arXiv%20-red.svg)](https://arxiv.org/abs/2401.15563)
[![webpage](https://img.shields.io/badge/🌐-Website%20-blue.svg)](https://brepgen.github.io) 
[![Youtube](https://img.shields.io/badge/📽️-Video%20-orchid.svg)](https://www.youtube.com/xxx)

*[Xiang Xu](https://samxuxiang.github.io/), [Joseph Lambourne](https://www.research.autodesk.com/people/joseph-george-lambourne/),
[Pradeep Jayaraman](https://www.research.autodesk.com/people/pradeep-kumar-jayaraman/), [Zhengqing Wang](https://www.linkedin.com/in/zhengqing-wang-485854241/?originalSubdomain=ca), [Karl Willis](https://www.karlddwillis.com/), and [Yasutaka Furukawa](https://yasu-furukawa.github.io/)*

![alt BrepGen](resources/teaser.jpg)

> 我们提出了一种基于扩散的生成方法，可直接输出 CAD B-rep。BrepGen 使用一种新颖的结构化潜在几何来编码 CAD 几何和拓扑。采用自顶向下的生成方法来对面、边和顶点进行去噪。


## 环境要求

### 测试环境
- Linux
- Python 3.9
- CUDA 11.8 
- PyTorch 2.2 
- Diffusers 0.27


### 依赖项

安装 PyTorch 和其他依赖项：
```
conda create --name brepgen_env python=3.9 -y
conda activate brepgen_env

pip install -r requirements.txt
pip install chamferdist
```

如果 `chamferdist` 安装失败，可以尝试以下几种方法：

- 如果出现 CUDA 版本不匹配错误，请尝试设置 `CUDA_HOME` 环境变量指向 CUDA 安装文件夹。该文件夹的 CUDA 版本必须与 PyTorch 的版本匹配，即 11.8。

- 尝试[从源码构建](https://github.com/krrish94/chamferdist?tab=readme-ov-file#building-from-source)。

按照[此处](https://github.com/AutodeskAILab/occwl)的说明安装 OCCWL。
如果 conda 卡在"Solving environment..."，可以尝试两种方法：

- 按照 occwl 的 README 建议，尝试使用 `mamba`。

- 安装 pythonOCC：https://github.com/tpaviot/pythonocc-core?tab=readme-ov-file#install-with-conda 并手动安装 occwl：`pip install git+https://github.com/AutodeskAILab/occwl`。

## 数据
下载 [ABC](https://archive.nyu.edu/handle/2451/43778) STEP 文件（100 个文件夹）。 

下载 [Furniture Data](https://drive.google.com/drive/folders/1WpV_rgJDXEkBoWaQsqEoO9Ir8CABI8oP?usp=sharing)。JSON 文件包含指向原始 STEP 文件的对象 UID。 

需要从 STEP 文件中提取面、边和顶点。  

处理 B-rep（在 ```data_process``` 文件夹下）：

    sh process.sh


去除重复的 CAD 模型（在 ```data_process``` 文件夹下，默认为 ```6 bit```）：

    sh deduplicate.sh

您可以下载去重后的文件：[DeepCAD](https://drive.google.com/drive/folders/1N_60VCZKYgPviQgP8lwCOVXrzu9Midfe?usp=drive_link) 和 [ABC](https://drive.google.com/drive/folders/1bA90Rz5EcwaUhUrgFbSIpgdJ0aeDjy3v?usp=drive_link)。



## 训练 
训练表面和边缘 VAE（使用 wandb 记录日志）：

    sh train_vae.sh

训练潜在扩散模型（将路径更改为之前训练的 VAE）：

    sh train_ldm.sh

```--cf``` 用于 Furniture 数据集的分类器自由训练。 

```--data_aug``` 在训练期间随机旋转 CAD 模型（可选）。




## 生成和评估
从高斯噪声随机生成 B-rep，将保存 STEP 和 STL 文件：

    python sample.py --mode abc

这将加载 ```eval_config.yaml``` 中的设置。请确保将模型路径更新到正确的文件夹。 

运行此脚本进行评估（将路径更改为生成的数据文件夹，至少需要 3,000 个样本）：

    sh eval.sh
    
这将计算 JSD、MMD 和 COV 分数。还请下载[测试集](https://drive.google.com/drive/folders/1kqxSDkS2gUN9_qpuWotFDhl4t7czbfOc?usp=sharing)的采样点云。


## 预训练检查点
我们还提供了在不同数据集上训练的各个检查点。 
| **源数据集** |  |   |                                                 
|--------------------|-----------| -----------|
| DeepCAD | [vae 模型](https://drive.google.com/drive/folders/1UZYqJ2EmTjzeTcNr_NL3bPpU4WrufvQa?usp=drive_link) |   [潜在扩散模型](https://drive.google.com/drive/folders/1jonuCzoTBFOKKlnaoGlbmhT6YlnH0lma?usp=drive_link) |
| ABC | [vae 模型](https://drive.google.com/drive/folders/18Ib9L0kpFf4ylZIRTCYFhXZB_GVIUm53?usp=drive_link) |   [潜在扩散模型](https://drive.google.com/drive/folders/1hv7ZUcU-L3J0LiONK60-TEh7sAN0zfve?usp=drive_link) |


## 致谢
本研究部分得到了 NSERC Discovery Grants with Accelerator Supplements、DND/NSERC Discovery Grant Supplement、NSERC Alliance Grants 和 John R. Evans Leaders Fund (JELF) 的支持。我们还要感谢 Onshape 的支持以及对公开可用 CAD 模型的访问。


## 引用
如果您在研究中发现我们的工作有用，请引用以下论文
```
@article{xu2024brepgen,
  title={BrepGen: A B-rep Generative Diffusion Model with Structured Latent Geometry},
  author={Xu, Xiang and Lambourne, Joseph G and Jayaraman, Pradeep Kumar and Wang, Zhengqing and Willis, Karl DD and Furukawa, Yasutaka},
  journal={arXiv preprint arXiv:2401.15563},
  year={2024}
}
```