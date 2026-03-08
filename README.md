# Brain Tumor Detection Environment Setup

This repository contains configuration files to help you set up a Python environment for neuroimaging and deep learning tasks, including support for working with NIfTI files, training models using Keras/TensorFlow, and visualizing with Plotly.

---

## Requirements

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or [Anaconda](https://www.anaconda.com/)
- Python 3.11.7

---

## Setup using Conda (`tumor.yml`)

### 1. Save the file as `tumor.yml`

Make sure `tumor.yml` is in your project directory.

### 2. Create the environment

```bash
conda env create -f tumor.yml
```

### 3. Activate the environment

```bash
conda activate brats
```

---

## GPU Setup (TensorFlow + CUDA)

> ✅ Tested and working on: **2x NVIDIA GeForce RTX 3060**, Driver `535.288.01`, CUDA `12.2`, TensorFlow `2.13.0`

### Working Configuration

| Component | Version |
|---|---|
| OS | Ubuntu (linux-64) |
| NVIDIA Driver | 535.288.01 |
| System CUDA | 12.2 |
| TensorFlow | 2.13.0 |
| nvidia-cudnn-cu11 | 8.6.0.163 |
| nvidia-cublas-cu11 | 11.11.3.6 |
| Conda env name | `brats` |
| GPUs | 2x NVIDIA GeForce RTX 3060 (12GB each) |

### Step 1: Install cuDNN and cuBLAS via pip

```bash
pip install nvidia-cudnn-cu11==8.6.0.163
```

This also installs `nvidia-cublas-cu11==11.11.3.6` automatically.

### Step 2: Set environment variables

Run these in your terminal before starting your session:

```bash
export CUDA_HOME=$CONDA_PREFIX
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
export PATH=$CONDA_PREFIX/bin:$PATH
export LD_LIBRARY_PATH=$(python -c "import nvidia.cudnn; import os; print(os.path.dirname(nvidia.cudnn.__file__))")/lib:$LD_LIBRARY_PATH
```

### Step 3: Make it permanent (recommended)

Add the exports to your `~/.bashrc` so you don't need to re-run them every session:

```bash
echo 'export CUDA_HOME=$CONDA_PREFIX' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export PATH=$CONDA_PREFIX/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$(python -c "import nvidia.cudnn; import os; print(os.path.dirname(nvidia.cudnn.__file__))")/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

### Step 4: Verify GPU detection

```bash
python -c "import tensorflow as tf; print('TF:', tf.__version__); print('GPUs:', tf.config.list_physical_devices('GPU'))"
```

Expected output:

```
TF: 2.13.0
GPUs: [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU'),
       PhysicalDevice(name='/physical_device:GPU:1', device_type='GPU')]
```

---

## Troubleshooting

### GPU not detected — `Cannot dlopen some GPU libraries`

This is caused by a **CUDA version mismatch** between your conda environment and TensorFlow.

| Component | Required |
|---|---|
| TensorFlow 2.13 | CUDA 11.8 + cuDNN 8.6 |
| Your NVIDIA driver | Must be ≥ 520 (supports CUDA 11.x) |

**Fix:** Follow the GPU Setup steps above. The key is installing `nvidia-cudnn-cu11==8.6.0.163` via pip and pointing `LD_LIBRARY_PATH` to its lib directory.

### `PackagesNotFoundError: cudnn=8.6` via conda

cuDNN 8.6 is not available on conda-forge. Use the pip method described in GPU Setup instead.

---

## Notes

- The `TF-TRT Warning: Could not find TensorRT` message is harmless and can be ignored unless you plan to use TensorRT optimization.
- Your NVIDIA driver (535.x) supports CUDA 12.2, but TF 2.13 only needs CUDA 11.8. This is fine — newer drivers are **backward compatible** with older CUDA toolkits.
