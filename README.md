# llama.cpp deployment for NVIDIA Maxwell GPUs

This repository contains deployment configurations for running [llama.cpp](https://github.com/ggml-org/llama.cpp) with CUDA support on NVIDIA Maxwell architecture GPUs.

## Overview

The `maxwell/` directory contains everything needed to build and deploy llama.cpp server with CUDA support optimized for NVIDIA Maxwell GPUs (compute capability 5.2). The setup includes Docker configurations and docker-compose for running multiple model services simultaneously.

## Features

- **Multi-stage Docker build** for minimal runtime image size
- **CUDA support** optimized for Maxwell architecture (SM_52)
- **Multiple model types**: embeddings, rerankers, and LLMs
- **GPU optimization** configurations for temperature and power management
- **Docker Compose** for orchestrating multiple model containers

## Quick Start

### Prerequisites

- NVIDIA GPU with Maxwell architecture (GTX 900 series, Titan X, etc.)
- NVIDIA Docker runtime (nvidia-container-toolkit)
- Docker and Docker Compose

### Build Docker Image

```bash
# Clone llama.cpp repository
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp

# Copy Dockerfile from this repository
cp /path/to/this/repo/maxwell/Dockerfile ./.devops/llama.cpp.maxwell.Dockerfile

# Build image
docker build -t llama-server-cuda:11.8.0-22.04 -f .devops/llama.cpp.maxwell.Dockerfile .
```

### Configure Environment

Edit `maxwell/.env` to set your model paths and API keys:

```env
# Model paths - adjust to your HuggingFace cache location
BGE_M3_MODEL_PATH=/models/models--ggml-org--bge-m3-Q8_0-GGUF/snapshots/.../bge-m3-q8_0.gguf
BGE_RERANKER_MODEL_PATH=/models/models--gpustack--bge-reranker-v2-m3-GGUF/snapshots/.../bge-reranker-v2-m3-Q8_0.gguf
LLAMA_31_8B_MODEL_PATH=/models/models--bartowski--Meta-Llama-3.1-8B-Instruct-GGUF/snapshots/.../Meta-Llama-3.1-8B-Instruct-Q6_K.gguf

# API keys
API_KEY=your_secret_key
LLAMA_API_KEY=123465

# Volume mapping (host:container)
MODELS_VOLUME=/home/user/.cache/huggingface/hub:/models
```

### Run Services

```bash
cd maxwell
docker compose up -d
```

This will start three services:
- **bge-m3** (port 8080) - Embedding model
- **bge-reranker-v2-m3** (port 8081) - Reranker model
- **llama-31-8b-q6k** (port 8090) - Llama 3.1 8B LLM

## Hardware Setup

### Install NVIDIA Drivers

See `maxwell/NVIDIA.README.md` for detailed instructions:

```bash
# Search for available driver versions
ubuntu-drivers devices

# Install driver (headless, no X11)
sudo apt install --no-install-recommends nvidia-driver-580 nvidia-utils-580
sudo reboot
```

### Install NVIDIA Container Toolkit

```bash
# Follow instructions at:
# https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

# Restart Docker after installation
sudo systemctl restart docker
```

### GPU Optimization

Optimize your GPU for 24/7 operation:

```bash
# Enable persistence mode
sudo nvidia-smi -pm 1

# Limit power consumption (reduces temperature ~10-15°C with ~5-7% performance loss)
sudo nvidia-smi -pl 180
```

## Architecture

### Docker Build

The multi-stage build (`maxwell/Dockerfile`) consists of:

1. **Builder stage**: Compiles llama.cpp with CUDA support
   - Uses `gcc-10`/`g++-10` for compatibility
   - Sets CUDA compute architecture to `52` (Maxwell)
   - Builds only the server binary (tests/examples disabled)

2. **Runtime stage**: Minimal image with:
   - llama-server binary
   - Required shared libraries
   - Environment variables for library paths

### Model Serving

The llama-server supports different model types via command-line flags:

| Model Type | Flags | Example |
|------------|-------|---------|
| Embedding | `--embedding` | bge-m3 |
| Reranker | `--rerank --embedding --pooling rank` | bge-reranker-v2-m3 |
| LLM | (none) | Llama 3.1 |

Common flags:
- `-ngl 99`: Offload all layers to GPU
- `-c 4096`: Context size (tokens)
- `--ubatch-size` / `--batch-size`: Batch processing (for rerankers)
- `--api-key`: API authentication

## Project Structure

```
.
├── CLAUDE.md              # Guidance for AI assistants
├── README.md              # This file
└── maxwell/
    ├── Dockerfile         # Multi-stage Docker build
    ├── .env              # Environment variables for docker-compose
    ├── docker-compose.yml # Orchestration for 3 model containers
    ├── NVIDIA.README.md  # NVIDIA driver installation guide
    ├── LLAMACPP.README.md # Docker build/run commands reference
    └── README_LV.md      # Native llama.cpp build instructions
```

## Model Requirements

Download GGUF models from HuggingFace and place them in your cache directory:

- **bge-m3**: [ggml-org/bge-m3-Q8_0-GGUF](https://huggingface.co/ggml-org/bge-m3-Q8_0-GGUF)
- **bge-reranker-v2-m3**: [gpustack/bge-reranker-v2-m3-GGUF](https://huggingface.co/gpustack/bge-reranker-v2-m3-GGUF)
- **Llama 3.1 8B**: [bartowski/Meta-Llama-3.1-8B-Instruct-GGUF](https://huggingface.co/bartowski/Meta-Llama-3.1-8B-Instruct-GGUF)

## Troubleshooting

### Build fails with CUDA errors

- Ensure NVIDIA drivers are installed: `nvidia-smi`
- Check CUDA toolkit version compatibility
- Verify gcc-10 is installed: `gcc-10 --version`

### Container cannot access GPU

- Install nvidia-container-toolkit
- Restart Docker after installation
- Test with: `docker run --rm --gpus all nvidia/cuda:11.8.0-base-ubuntu22.04 nvidia-smi`

### Out of memory errors

- Reduce `-ngl` value (fewer layers on GPU)
- Reduce `-c` context size
- Use quantized models (Q4_K_M instead of Q8_0)

## License

This repository contains configuration files for deploying llama.cpp, which is licensed under the MIT License. See the [llama.cpp repository](https://github.com/ggml-org/llama.cpp) for details.

## Contributing

This is a deployment configuration repository. For llama.cpp development, see the [upstream repository](https://github.com/ggml-org/llama.cpp).
