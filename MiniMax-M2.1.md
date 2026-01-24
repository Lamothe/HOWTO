# MiniMax M2.1 running on the Strix Halo using llama.cpp

I've only tested this on:
* Strix Halo with 128 GB Unified Memory
* MiniMax M2.1 95 GB model (MiniMaxAI_MiniMax-M2.1-IQ3_XS)
* Fedora 43 with stock ROCm 6.4.x

# Memory Configuration in BIOS
* `iGPU Memory Configuration` = `Custom`
* `iGPU Memory` = `4 GB` (Required for 16 streams + Display Buffer (1080p) + Kernel Context)

# Setup
## Configure memory for Strix Halo
```
echo "🔧 Configuring Kernel Parameters for 128GB Strix Halo..."
sudo grubby --update-kernel=ALL --args='amdgpu.gttsize=131072 ttm.pages_limit=33554432'
sudo grubby --update-kernel=ALL --args='amd_iommu=off'

CURRENT_GTT=$(sudo dmesg | grep -i "GTT memory ready" | tail -n 1)

echo "-----------------------------------------------------"
if [[ "$CURRENT_GTT" == *"131072M"* ]]; then
    echo "🚀 GOOD NEWS: GTT memory is ready."
else
    echo "⚠️  ACTION REQUIRED: REBOOT NEEDED"
    echo "   GTT Memory is NOT 128GB yet."
    echo "   Please reboot now."
fi
echo "-----------------------------------------------------"
```
## Download
```
# Download the repo
mkdir -p ~/Projects
cd ~/Projects
https://github.com/ggml-org/llama.cpp.git
cd ~/Projects/llama.cpp

# Download the model
MODEL_FILE_FILTER="*IQ3_XS*"

hf download "bartowski/MiniMaxAI_MiniMax-M2.1-GGUF" \
    --include "*IQ3_XS*" \
    --local-dir ~/models \
    --local-dir-use-symlinks False
```

# Build

## CPU (~14 tokens/sec)
```
cd ~/Projects/llama.cpp

# Remove the build cache
rm -rf build

# Don't use the GPU
export HIP_VISIBLE_DEVICES=-1

# Configure for CPU
cmake -S . -B build \
    -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j $(nproc)
```

## Vulkan (~32 tokens/sec)
```
cd ~/Projects/llama.cpp

# Clean everything (CMake caches are stubborn)
rm -rf build

# Configure
cmake -S . -B build \
    -DGGML_VULKAN=1 \
    -DCMAKE_BUILD_TYPE=Release

# 3. Compile
cmake --build build --config Release -j $(nproc)
```

## ROCm (Not working)
```
cd ~/Projects/llama.cpp

# Remove the build cache
rm -rf build

# Autodetect the GPU
unset HIP_VISIBLE_DEVICES
export GGML_CUDA_FORCE_HOST_MALLOC=1

# Configure for ROCm (gfx1151 for Strix Halo)
cmake -S . -B build \
    -DGGML_HIP=ON \
    -DAMDGPU_TARGETS=gfx1151 \
    -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release -j $(nproc)
```

# Install
```
sudo cmake --install build --prefix /usr/local
```

# Run
## 4096 Token Context Window
```
export LD_LIBRARY_PATH=/usr/local/lib64:/usr/lib64/rocm/lib:/usr/lib64
export HSA_OVERRIDE_GFX_VERSION=11.5.1 # Only required for ROCm
export LLAMA_ARG_HOST=0.0.0.0 # Allow remote access

/usr/local/bin/llama-server \
    -m ~/models/MiniMaxAI_MiniMax-M2.1-IQ3_XS/MiniMaxAI_MiniMax-M2.1-IQ3_XS-00001-of-00003.gguf \
    -c 4096 \
    -ngl 99 \
    --port 8080
```
## Large Context Window
```
# Start the 'Big Brain' mode
cd ~/Projects/llama.cpp/build
/usr/local/bin/llama-server \
    -m ~/models/MiniMaxAI_MiniMax-M2.1-IQ3_XS/MiniMaxAI_MiniMax-M2.1-IQ3_XS-00001-of-00003.gguf \
    -c 81920 \
    -ctk q8_0 -ctv q8_0 \
    -ngl 99 \
    --port 8080
```
