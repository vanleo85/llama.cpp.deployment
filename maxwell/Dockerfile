FROM nvidia/cuda:11.8.0-devel-ubuntu22.04 AS builder

RUN apt update && apt install -y \
    build-essential \
    cmake \
    git \
    gcc-10 g++-10 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build
COPY . .

RUN export LIBRARY_PATH="$LIBRARY_PATH:/usr/local/cuda/lib64/stubs" && \
    cmake -B build \
                    -DGGML_CUDA=ON \
                    -DCMAKE_BUILD_TYPE=Release \
                    -DLLAMA_BUILD_EXAMPLES=OFF \
                    -DLLAMA_BUILD_TESTS=OFF \
                    -DCMAKE_CUDA_ARCHITECTURES=52 \
                    -DCMAKE_EXE_LINKER_FLAGS="-Wl,--allow-shlib-undefined" \
                    -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/gcc-10 \
                    -DCMAKE_CXX_COMPILER=/usr/bin/g++-10 && \
    cmake --build build --config Release -j$(nproc)

FROM nvidia/cuda:11.8.0-runtime-ubuntu22.04 AS runtime

RUN apt-get update && apt-get install -y \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY --from=builder --chmod=755 /build/build/bin/llama-server* .
COPY --from=builder --chmod=755 /build/build/bin/*.so* /usr/local/lib/

RUN ldconfig

ENV LLAMA_ARG_HOST=0.0.0.0

ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

ENTRYPOINT ["./llama-server"]

