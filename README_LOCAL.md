# Local Installation Guide - Chatterbox TTS Server

This guide documents the complete setup process for installing the Chatterbox TTS Server on a local machine, including resolving the ONNX build issues that occur with Python 3.12.

## Prerequisites

- Ubuntu/Debian Linux system
- Python 3.12
- NVIDIA GPU (optional, for GPU acceleration)
- Git

## Step 1: Install System Dependencies

Install essential build tools and libraries:

```bash
sudo apt update
sudo apt install -y build-essential cmake git python3-dev python3-pip libprotobuf-dev protobuf-compiler libssl-dev libffi-dev libbz2-dev libreadline-dev libsqlite3-dev pkg-config libhdf5-dev
```

## Step 2: Create and Activate Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

## Step 3: Upgrade pip

```bash
pip install --upgrade pip
```

## Step 4: Install Compatible ONNX Version First

This prevents the build issues with older ONNX versions and Python 3.12:

```bash
pip install "onnx>=1.15.0" --only-binary=all
```

## Step 5: Install s3tokenizer Without Dependencies

This avoids dependency conflicts:

```bash
pip install --force-reinstall --no-deps s3tokenizer
```

## Step 6: Install Requirements Without Dependency Resolution

```bash
pip install -r requirements.txt --no-deps
```

## Step 7: Install Missing Web Framework Dependencies

```bash
pip install --only-binary=all starlette pydantic anyio sniffio idna certifi charset-normalizer urllib3 click h11 httptools python-dotenv uvloop watchfiles websockets
```

## Step 8: Install Remaining Dependencies

```bash
pip install --only-binary=all filelock fsspec networkx setuptools sympy markupsafe conformer diffusers resemble-perth transformers more-itertools typeguard cffi argbind descript-audiotools einops pre-commit pillow audioread decorator joblib lazy-loader msgpack numba pooch scikit-learn scipy soxr
```

## Step 9: Fix Protobuf Version Conflict

```bash
pip install "protobuf>=4.25.1,<5.0"
```

## Step 10: Install GPU Support (Optional)

If you want GPU acceleration, replace the CPU-only PyTorch with CUDA version:

### Check GPU and CUDA Version

```bash
nvidia-smi
```

### Uninstall CPU PyTorch

```bash
pip uninstall torch torchvision torchaudio -y
```

### Install CUDA PyTorch

For CUDA 12.4 (adjust version based on your CUDA version):

```bash
pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cu124
```

## Step 11: Verify Installation

### Test Basic Imports

```bash
python -c "from chatterbox.tts import ChatterboxTTS; print('Chatterbox TTS import successful')"
python -c "import fastapi, uvicorn; print('FastAPI and Uvicorn import successful')"
```

### Test GPU Detection (if GPU support installed)

```bash
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}'); print(f'GPU: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else \"N/A\"}')"
```

### Test GPU Operations

```bash
python -c "import torch; device = torch.device('cuda' if torch.cuda.is_available() else 'cpu'); x = torch.randn(3, 3).to(device); print(f'GPU test on {device} successful')"
```

## Known Issues and Solutions

### ONNX Build Failure with Python 3.12

**Problem**: ONNX fails to build from source due to pybind11 compatibility issues with Python 3.12.

**Solution**: Install a newer pre-built ONNX version first (Step 4), then install dependencies without resolution (Step 6).

### Protobuf Version Conflicts

**Problem**: Different packages require incompatible protobuf versions.

**Solution**: Install a compatible middle-ground version (Step 9).

### RTX 5090 Compatibility Warning

**Problem**: PyTorch 2.6.0 shows warnings about RTX 5090's newer compute capability (sm_120).

**Status**: Operations work but may not be fully optimized. Wait for newer PyTorch versions for full compatibility.

## Expected Warnings

You may see these warnings which are normal:

1. `pkg_resources is deprecated` - From resemble-perth package
2. `CUDA capability sm_120 is not compatible` - For RTX 5090 users
3. Dependency conflict messages - These don't prevent functionality

## File Structure

After installation, your project should look like:

```
Chatterbox-TTS-Server/
├── requirements.txt (CPU-only versions)
├── requirements-nvidia.txt
├── requirements-rocm.txt
├── README.md
├── README_LOCAL.md (this file)
└── venv/ (your virtual environment)
```

## Alternative Requirements Files

If the main `requirements.txt` causes issues, try:

- `requirements-nvidia.txt` - For NVIDIA GPU setups
- `requirements-rocm.txt` - For AMD GPU setups

## Troubleshooting

### If imports fail:

1. Check that all dependencies installed successfully
2. Verify virtual environment is activated
3. Try installing missing packages individually

### If GPU not detected:

1. Verify NVIDIA drivers are installed: `nvidia-smi`
2. Check CUDA version compatibility
3. Ensure you installed the CUDA version of PyTorch

### If build errors occur:

1. Ensure all system dependencies are installed (Step 1)
2. Try using `--only-binary=all` flag to avoid source builds
3. Clear pip cache: `pip cache purge`

## Performance Notes

- CPU-only installation will work but be slower for inference
- GPU acceleration significantly improves performance
- RTX 5090 users may see compatibility warnings but basic functionality works
- For production use, consider using conda/mamba instead of pip for better dependency management