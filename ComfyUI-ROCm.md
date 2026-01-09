# Comfy on ROCm

## Setup
```
mkdir -p ~/Applications
cd ~/Applications
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.4 --force-reinstall
```

## Test
```
python3 -c 'import torch; print(f"Torch available: {torch.cuda.is_available()}"); print(f"Torch version: {torch.__version__}")'
```
If you see "/opt/amdgpu/share/libdrm/amdgpu.ids: No such file or directory", ignore it.

## Update
```
cd ~/Applications/ComfyUI
git pull
```

## Run
```
cd ~/Applications/ComfyUI
source .venv/bin/activate
python main.py
```
