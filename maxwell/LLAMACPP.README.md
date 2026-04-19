

# Установка llama.cpp

## Установить cuda toolkit
https://developer.nvidia.com/cuda-downloads
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-2

## Установить nvidia-cuda-toolkit
sudo apt install nvidia-cuda-toolkit -y
nvcc --version

## билдим llama.cpp
git clone git@github.com:ggml-org/llama.cpp.git 
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp/

gcc old version fixes build error.

sudo apt install gcc-10 g++-10
cmake -B build \
  -DGGML_CUDA=ON \
  -DCMAKE_CUDA_ARCHITECTURES=52 \
  -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/gcc-10 \
  -DCMAKE_CXX_COMPILER=/usr/bin/g++-10
cmake --build build --config Release -j$(nproc)

./llama.cpp/build/bin/llama-server -m  /home/l-vanin/.cache/huggingface/hub/models--ggml-org--bge-m3-Q8_0-GGUF/snapshots/9eba04c5d75ba5a1595e45de734d36bef4e5cb98/bge-m3-q8_0.gguf -ngl 99 -c 4096 --host 0.0.0.0 --port 8081 --api-key "your_secret_key" --embedding












