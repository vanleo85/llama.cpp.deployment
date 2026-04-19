
# Сборка Docker образа

git clone https://github.com/ggml-org/llama.cpp.git 
cd llama.cpp
cp ../maxwell/Dockerfile ./.devops/llama.cpp.maxvell.Dockerfile
docker build -t llama-server-cuda:11.8.0-22.04 -f .devops/llama.cpp.maxvell.Dockerfile .

docker run --name bge-m3 --gpus all -d -p 8080:8080 \
  -v /home/l-vanin/.cache/huggingface/hub:/models \
  llama-server-cuda:11.8.0-22.04 \
  -m /models/models--ggml-org--bge-m3-Q8_0-GGUF/snapshots/9eba04c5d75ba5a1595e45de734d36bef4e5cb98/bge-m3-q8_0.gguf \
  -ngl 99 -c 4096 --host 0.0.0.0 --port 8080 --api-key "your_secret_key" --embedding

docker run --name bge-reranker-v2-m3 --gpus all -d -p 8081:8080 \
  -v /home/l-vanin/.cache/huggingface/hub:/models \
  llama-server-cuda:11.8.0-22.04 \
  -m /models/models--gpustack--bge-reranker-v2-m3-GGUF/snapshots/3093af03b1a635e67b084b1d8c03c5f5e020fd05/bge-reranker-v2-m3-Q8_0.gguf \
  -ngl 99 --host 0.0.0.0 --port 8080 --api-key "your_secret_key" -c 4096 --ubatch-size 4096 --batch-size 4096 -np 1 --rerank --embedding --pooling rank

docker run --name llama-31-8b-q6k --gpus all -d -p 8090:8080 \
  -v /home/l-vanin/.cache/huggingface/hub:/models \
  llama-server-cuda:11.8.0-22.04 \
  -m /models/models--bartowski--Meta-Llama-3.1-8B-Instruct-GGUF/snapshots/bf5b95e96dac0462e2a09145ec66cae9a3f12067/Meta-Llama-3.1-8B-Instruct-Q6_K.gguf \
  -ngl 99 -c 8192 --host 0.0.0.0 --port 8080 --api-key "your_secret_key"








