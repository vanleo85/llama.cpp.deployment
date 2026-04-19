# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains deployment configurations and documentation for running llama.cpp with CUDA support on NVIDIA GPUs. It focuses on embedding models (bge-m3), rerankers (bge-reranker-v2-m3), and LLM inference (Llama 3.1) via Docker containers.

## Architecture

### Docker-based Deployment

The project uses a multi-stage Docker build (`maxwell/Dockerfile`) that:
1. **Builder stage**: Compiles llama.cpp with CUDA support using gcc-10/g++-10 (required for compatibility)
2. **Runtime stage**: Creates a minimal runtime image with just the llama-server binary and shared libraries

Key build flags:
- `-DGGML_CUDA=ON`: Enables CUDA support
- `-DCMAKE_CUDA_ARCHITECTURES=52`: Sets CUDA compute capability (adjust for your GPU)
- `-DCMAKE_CUDA_HOST_COMPILER=/usr/bin/gcc-10`: Uses older gcc version for build compatibility

### Model Serving

The llama-server binary supports multiple model types:
- **Embedding models**: `--embedding` flag (e.g., bge-m3)
- **Rerankers**: `--rerank --embedding --pooling rank` flags (e.g., bge-reranker-v2-m3)
- **LLMs**: No special flags needed (e.g., Llama 3.1)

Common server flags:
- `-ngl 99`: Offload all layers to GPU
- `-c 4096` or `-c 8192`: Context size
- `--ubatch-size` / `--batch-size`: Batch processing for rerankers
- `--api-key "KEY"`: API authentication
- `--host 0.0.0.0 --port 8080`: Network binding

## Build and Deploy

### Build Docker Image
```bash
# From llama.cpp source directory
cp ../maxwell/Dockerfile ./.devops/llama.cpp.maxvell.Dockerfile
docker build -t llama-server-cuda:11.8.0-22.04 -f .devops/llama.cpp.maxvell.Dockerfile .
```

### Run Container (examples from maxwell/LLAMACPP.README.md)
```bash
# Embedding model (bge-m3)
docker run --name bge-m3 --gpus all -d -p 8080:8080 \
  -v /home/l-vanin/.cache/huggingface/hub:/models \
  llama-server-cuda:11.8.0-22.04 \
  -m /models/models--ggml-org--bge-m3-Q8_0-GGUF/snapshots/.../bge-m3-q8_0.gguf \
  -ngl 99 -c 4096 --host 0.0.0.0 --port 8080 --api-key "your_secret_key" --embedding

# Reranker (bge-reranker-v2-m3)
docker run --name bge-reranker-v2-m3 --gpus all -d -p 8081:8080 \
  -v /home/l-vanin/.cache/huggingface/hub:/models \
  llama-server-cuda:11.8.0-22.04 \
  -m /models/.../bge-reranker-v2-m3-Q8_0.gguf \
  -ngl 99 --host 0.0.0.0 --port 8080 --api-key "your_secret_key" \
  -c 4096 --ubatch-size 4096 --batch-size 4096 -np 1 --rerank --embedding --pooling rank
```

## Host Setup

### NVIDIA Drivers and CUDA Toolkit

See `maxwell/NVIDIA.README.md` and `maxwell/README_LV.md` for:
- Installing NVIDIA drivers (version 580 example)
- Installing CUDA toolkit 13.2
- Installing nvidia-container-toolkit for Docker GPU support
- GPU optimization: persistence mode and power limits

### GPU Optimization
```bash
# Enable persistence mode
sudo nvidia-smi -pm 1

# Limit power consumption (reduces temperature ~10-15°C with ~5-7% performance loss)
sudo nvidia-smi -pl 180
```

## Key Files

- `maxwell/Dockerfile`: Multi-stage Docker build for llama.cpp with CUDA
- `maxwell/LLAMACPP.README.md`: Docker build and run commands
- `maxwell/NVIDIA.README.md`: NVIDIA driver and CUDA installation instructions
- `maxwell/README_LV.md`: Host system setup and llama.cpp native build
