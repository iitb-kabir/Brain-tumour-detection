# Brain Tumor Detection Environment Setup

This repository contains configuration files to help you set up a Python environment for neuroimaging and deep learning tasks, including support for working with NIfTI files, training models using Keras/TensorFlow, and visualizing with Plotly.

---

## Requirements

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) or [Anaconda](https://www.anaconda.com/)
- Python 3.10.19
- NVIDIA GPU with driver ≥ 520

---

## Complete Verified Dependency Versions

| Package | Version | Source |
|---|---|---|
| Python | 3.10.19 | conda |
| TensorFlow | 2.13.0 | pip |
| tensorflow-estimator | 2.13.0 | pip |
| tensorflow-io-gcs-filesystem | 0.37.1 | pip |
| Keras | 2.13.1 | pip |
| segmentation-models-3D | 1.0.1 | pip |
| classification-models-3D | 1.1.0 | pip |
| efficientnet-3D | 1.0.2 | pip |
| nvidia-cudnn-cu11 | 8.6.0.163 | pip |
| nvidia-cublas-cu11 | 11.11.3.6 | pip |
| cudatoolkit | 11.8.0 | conda |
| numpy | 1.24.3 | pip |
| scipy | 1.15.3 | pip |
| scikit-image | 0.25.2 | pip |
| h5py | 3.16.0 | pip |
| Pillow | 12.1.1 | pip |
| NVIDIA Driver | 535.288.01 | system |
| System CUDA | 12.2 | system |

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

### 4. Install pip dependencies

```bash
pip install tensorflow==2.13.0
pip install keras==2.13.1
pip install segmentation-models-3D==1.0.1 classification-models-3D==1.1.0 efficientnet-3D==1.0.2
pip install nvidia-cudnn-cu11==8.6.0.163
```

---

## GPU Setup (TensorFlow + CUDA)

> ✅ Tested and working on: **2x NVIDIA GeForce RTX 3060**, Driver `535.288.01`, CUDA `12.2`, TensorFlow `2.13.0`

### Step 1: Set environment variables

Run these in your terminal before starting your session:

```bash
export CUDA_HOME=$CONDA_PREFIX
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
export PATH=$CONDA_PREFIX/bin:$PATH
export LD_LIBRARY_PATH=$(python -c "import nvidia.cudnn; import os; print(os.path.dirname(nvidia.cudnn.__file__))")/lib:$LD_LIBRARY_PATH
export SM_FRAMEWORK=tf.keras
```

### Step 2: Make it permanent (recommended)

```bash
echo 'export CUDA_HOME=$CONDA_PREFIX' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export PATH=$CONDA_PREFIX/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=$(python -c "import nvidia.cudnn; import os; print(os.path.dirname(nvidia.cudnn.__file__))")/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
echo 'export SM_FRAMEWORK=tf.keras' >> ~/.bashrc
source ~/.bashrc
```

### Step 3: Verify everything works

```bash
SM_FRAMEWORK=tf.keras python - << 'EOF'
import os
os.environ['SM_FRAMEWORK'] = 'tf.keras'
import tensorflow as tf
import segmentation_models_3D as sm
print("TF:", tf.__version__)
print("GPUs:", tf.config.list_physical_devices('GPU'))
print("segmentation_models_3D OK")
EOF
```

Expected output:

```
Segmentation Models: using `tf.keras` framework.
TF: 2.13.0
GPUs: [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU'),
       PhysicalDevice(name='/physical_device:GPU:1', device_type='GPU')]
segmentation_models_3D OK
```

---

## Manual Patches Required for `classification-models-3D==1.1.0`

> ⚠️ `classification-models-3D 1.1.0` was written for Keras 3.x but must run with Keras 2.13.1 (required by TF 2.13). Several source files need patching after install.

Run all patches in one go (replace `$USER` and `brats` if your username or env name differs):

```bash
PKG=/home/$USER/miniconda3/envs/brats/lib/python3.10/site-packages/classification_models_3D

# 1. Fix keras.src.legacy.backend → keras.backend across all model files
grep -rl "keras.src.legacy.backend" $PKG/models/ | xargs sed -i \
  's/from keras.src.legacy.backend import int_shape/from keras.backend import int_shape/g'

# 2. Fix _DepthwiseConv3D.py
sed -i 's/from keras.src.legacy.backend import _preprocess_padding, _preprocess_conv3d_input/from keras.backend import _preprocess_padding, _preprocess_conv3d_input/' \
  $PKG/models/_DepthwiseConv3D.py
sed -i 's/from keras import InputSpec/from keras.engine.base_layer import InputSpec/' \
  $PKG/models/_DepthwiseConv3D.py
sed -i 's/import keras.ops as kops/import tensorflow as kops/' \
  $PKG/models/_DepthwiseConv3D.py

# 3. Fix efficientnet.py
sed -i 's/from keras.src.utils import file_utils/import os.path as file_utils/' \
  $PKG/models/efficientnet.py
sed -i 's/from keras.src.ops import operation_utils/from keras import backend as operation_utils/' \
  $PKG/models/efficientnet.py

# 4. Fix efficientnet_v2.py and convnext.py
sed -i 's/from keras.src.utils import file_utils/from keras.utils import data_utils as file_utils/' \
  $PKG/models/efficientnet_v2.py
sed -i 's/from keras.src.utils import file_utils/from keras.utils import data_utils as file_utils/' \
  $PKG/models/convnext.py
sed -i 's/import keras.ops as kops/import tensorflow as kops/' \
  $PKG/models/convnext.py

# 5. Comment out Keras-3-only models (efficientnet, efficientnet_v2, convnext) from factory
python3 - << 'PYEOF'
import os
path = os.path.join(os.environ['PKG'], '../classification_models_3D/models_factory.py')
path = os.path.normpath(path)
with open(path, 'r') as f:
    lines = f.readlines()
skip_keywords = ['efficientnet', 'efficientnet_v2', 'convnext', 'eff.', 'eff2.', 'cnext.']
new_lines = []
for line in lines:
    stripped = line.strip()
    if any(kw in stripped for kw in skip_keywords) and not stripped.startswith('#'):
        new_lines.append('#' + line)
    else:
        new_lines.append(line)
with open(path, 'w') as f:
    f.writelines(new_lines)
print("Patched models_factory.py")
PYEOF

# 6. Clear all stale .pyc cache files
find $PKG -name "*.pyc" -delete

echo "All patches applied successfully!"
```

---

## Troubleshooting

### GPU not detected — `Cannot dlopen some GPU libraries`

TF 2.13 needs CUDA 11.8 + cuDNN 8.6, but the conda env may have a different version.

**Fix:** Install cuDNN via pip and set `LD_LIBRARY_PATH` as shown in GPU Setup above.

### `PackagesNotFoundError: cudnn=8.6` via conda

cuDNN 8.6 is not available on conda-forge. Use the pip method (`nvidia-cudnn-cu11==8.6.0.163`) instead.

### `ModuleNotFoundError: No module named 'keras.src.legacy'`

`classification-models-3D` contains Keras 3 internal imports. Run the full patch script in the Manual Patches section above.

### GPUs show `[]` after `segmentation_models_3D` imports fine

The `LD_LIBRARY_PATH` for cuDNN was not exported in this session. Re-run the exports from GPU Setup Step 1, or add them permanently to `~/.bashrc`.

---

## Notes

- `SM_FRAMEWORK=tf.keras` must always be set before importing `segmentation_models_3D`. Add it permanently to `~/.bashrc`.
- The `TF-TRT Warning: Could not find TensorRT` message is harmless and can be ignored unless you plan to use TensorRT optimization.
- NVIDIA driver 535.x supports CUDA 12.2, but TF 2.13 only needs CUDA 11.8 — newer drivers are backward compatible.
- EfficientNet, EfficientNetV2, and ConvNeXt backbones are disabled (Keras 3 only). All other backbones — **ResNet, VGG, DenseNet, SENet, ResNeXt, MobileNet, Inception** — are fully functional.
