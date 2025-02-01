<div align="center">
    <img src="./assets/org_icon.png" alt="smalldoges" width="100%">
</div>

<div align="center">

[![arXiv](https://img.shields.io/static/v1?label=arXiv&message=2412.11834&color=B31B1B&logo=arXiv)](https://arxiv.org/abs/2412.11834)
[![huggingface](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Models-FFD21E)](https://huggingface.co/collections/SmallDoge/doge-slm-679cc991f027c4a3abbded4a)
[![License: Apache-2.0](https://img.shields.io/badge/License-Apache--2.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

<h4>

**Small Doges is under construction, let's develop together!**

English | [简体中文](./README_zh.md)

</h4>

</div>


## About

<div align="center">
    <img src="./assets/doge_architecture.png" alt="drawing" width="600"/>
</div>

As shown in the figure, the sequence transformation part of the Doge architecture uses `Dynamic Mask Attention`, which can be understood as using self-attention related to value states during training, and using state-space without past state decay during inference, to solve the problem of existing Transformers or SSMs getting lost in long text. The state transformation part of Doge uses `Cross Domain Mixture of Experts`, which consists of dense linear layers and sparse embedding layers, and can additionally increase sparse parameters to continue training from dense weight checkpoints without retraining the entire model, thereby reducing the cost of continuous iteration of the model. In addition, Doge also uses `RMSNorm` and `Residual` with learnable parameters to adapt the gradient range of deep models.

**Dynamic Mask Attention Module**

![DMAttn](./assets/dmattn.png)
![DMAttn](./assets/mqar.png)

**Cross Domain Mixture of Experts Module**

![CDMoE](./assets/cdmoe.png)
![CDMoE](./assets/merm.png)

We also hope to use open-source tools and frameworks as much as possible to simplify the process from data processing to model training, so that beginners can easily understand and use them.


## Requirements

- Windows or Linux
- NVIDIA GPU
- Python 3.10+
- PyTorch 2.0+
- CUDA 11.8+

We highly recommend that you install the latest version of PyTorch and CUDA for optimal performance.

Of course, you can also use the open-source [Docker PyTorch](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/pytorch) image to avoid the hassle of configuring the environment.

```bash
docker pull nvcr.io/nvidia/pytorch:24.12-py3
docker run --privileged --gpus all -it --name PyTorch --shm-size=32g -p 8888:8888 -p 6006:6006 --ulimit memlock=-1 --ulimit stack=67108864 -v <your code path>:/workspace -v <your datasets path>:/workspace/Doge/datasets nvcr.io/nvidia/pytorch:24.12-py3
```

- `pip install transformers`: The core framework for all subsequent work.
- `pip install datasets sentencepiece boto3`: Used to download and process datasets.
- `pip install accelerate`: Used for distributed training.
- `pip install trl`: Used for fine-tuning with reinforcement learning.


## Installation

```bash
git clone https://github.com/SmallDoges/small-doge.git
cd small-doge
pip install -e .
```


## Usage

We have written a [notebook](./examples/notebook.ipynb) and [training guide](./recipes/doge/README.md) (still in progress) to demonstrate the entire process of datasets processing, model training, and model evaluation. You can also use the models that have been released independently.


## Models Released

### Doge-CheckPoint

![wsd_scheduler](./assets/wsd_scheduler.png)

Doge uses `wsd_scheduler` as the training scheduler, which divides the learning rate into three stages: `warmup`, `stable`, and `decay`. It allows us to continue training on any new dataset from any checkpoint in the `stable stage` without spikes of the training.

Here are the initial learning rates required to continue training at each checkpoint:

- **[Doge-20M](https://huggingface.co/SmallDoge/Doge-20M-checkpoint)**: 8e-3
- **[Doge-60M](https://huggingface.co/SmallDoge/Doge-60M-checkpoint)**: 6e-3
- **[Doge-160M](https://huggingface.co/SmallDoge/Doge-160M-checkpoint)**: 4e-3
- **Doge-320M**: 2e-3

| Model | Learning Rate | Schedule | Warmup Steps | Stable Steps |
|-------|---------------|----------|--------------|--------------|
| [Doge-20M]((https://huggingface.co/SmallDoge/Doge-20M-checkpoint)) | 8e-3 | wsd_scheduler | 800 | 6400 |
| [Doge-60M](https://huggingface.co/SmallDoge/Doge-60M-checkpoint) | 6e-3 | wsd_scheduler | 1600 | 12800 |
| [Doge-160M](https://huggingface.co/SmallDoge/Doge-160M-checkpoint) | 4e-3 | wsd_scheduler | 2400 | 19200 |
| Doge-320M | 2e-3 | wsd_scheduler | 3200 | 25600 ||

### Doge-SLM

**Pre-Training**:
| Model | Training Data | Steps | Content Length | Tokens | LR | Batch Size | Precision |
|---|---|---|---|---|---|---|---|
| [Doge-20M](https://huggingface.co/SmallDoge/Doge-20M) | [HuggingFaceTB/smollm-corpus](https://huggingface.co/datasets/HuggingFaceTB/smollm-corpus) | 8k  | 2048 | 4B | 8e-3 | 0.5M | bfloat16 |
| [Doge-60M](https://huggingface.co/SmallDoge/Doge-60M) | [HuggingFaceTB/smollm-corpus](https://huggingface.co/datasets/HuggingFaceTB/smollm-corpus) | 16k  | 2048 | 16B | 6e-3 | 1M | bfloat16 |

**Evaluation**:
| Model | MMLU | TriviaQA | ARC-E | ARC-C | PIQA | HellaSwag | OBQA | Winogrande | tokens / s on CPU |
|---|---|---|---|---|---|---|---|---|---|
| [Doge-20M](https://huggingface.co/SmallDoge/Doge-20M) | 25.43 | 0.03 | 36.83 | 22.78 | 58.38 | 27.25 | 25.60 | 50.20 | 142 |
| [Doge-60M](https://huggingface.co/SmallDoge/Doge-60M) | 26.41 | 0.18 | 50.46 | 25.34 | 61.43 | 31.45 | 28.00 | 50.75 | 62 |

> All evaluations are done using five-shot settings, without additional training on the benchmarks.

**SFT**:
| Model | Training Data | Epochs | Content Length | LR | Batch Size | Precision |
|---|---|---|---|---|---|---|
| [Doge-20M-Instruct-SFT](https://huggingface.co/SmallDoge/Doge-20M-Instruct-SFT) | [HuggingFaceTB/smoltalk](https://huggingface.co/datasets/HuggingFaceTB/smoltalk) | 2 | 2048 | 8e-4 | 0.25M | bfloat16 |
| [Doge-60M-Instruct-SFT](https://huggingface.co/SmallDoge/Doge-60M-Instruct-SFT) | [HuggingFaceTB/smoltalk](https://huggingface.co/datasets/HuggingFaceTB/smoltalk) | 2 | 2048 | 6e-4 | 0.25M | bfloat16 |

**DPO**:
| Model | Training Data | Epochs | Content Length | LR | Batch Size | Precision |
|---|---|---|---|---|---|---|
| [Doge-20M-Instruct](https://huggingface.co/SmallDoge/Doge-20M-Instruct) | [HuggingFaceH4/ultrafeedback_binarized](https://huggingface.co/datasets/HuggingFaceH4/ultrafeedback_binarized) | 2 | 1024 | 8e-5 | 0.125M | bfloat16 |
| [Doge-60M-Instruct](https://huggingface.co/SmallDoge/Doge-60M-Instruct) | [HuggingFaceH4/ultrafeedback_binarized](https://huggingface.co/datasets/HuggingFaceH4/ultrafeedback_binarized) | 2 | 1024 | 6e-5 | 0.125M | bfloat16 |

**Environment**:
- Image: nvcr.io/nvidia/pytorch:24.12-py3
- Hardware: 1x NVIDIA RTX 4090
- Software: Transformers, TRL


## Citation

If you use this codebase, or otherwise find our work valuable, please cite our paper:

```bibtex
@misc{shi2024wonderfulmatrices,
      title={Wonderful Matrices: Combining for a More Efficient and Effective Foundation Model Architecture}, 
      author={Jingze Shi and Bingheng Wu},
      year={2024},
      eprint={2412.11834},
      archivePrefix={arXiv},
      primaryClass={cs.LG},
      url={https://arxiv.org/abs/2412.11834}, 
}
```
