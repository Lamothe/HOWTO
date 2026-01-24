# Initialize Container
Create and enter a clean Ubuntu 24.04 container.

```
distrobox create --image ubuntu:24.04 --name rocm7
distrobox enter rocm7
```

# Install ROCm 7.2
```
wget https://repo.radeon.com/amdgpu-install/7.2/ubuntu/noble/amdgpu-install_7.2.70200-1_all.deb
sudo apt install ./amdgpu-install_7.2.70200-1_all.deb
sudo apt update
sudo apt install python3-setuptools python3-wheel
sudo usermod -a -G render,video $LOGNAME # Add the current user to the render and video groups
sudo apt install rocm
```

It think that I also did installed the driver.
```
wget https://repo.radeon.com/amdgpu-install/7.2/ubuntu/noble/amdgpu-install_7.2.70200-1_all.deb
sudo apt install ./amdgpu-install_7.2.70200-1_all.deb
sudo apt update
sudo apt install linux-headers-generic
#sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo apt install amdgpu-dkms
```

# System Prerequisites
Install Python system tools. python3-venv is strictly required to comply with PEP 668 (managed environments).

```
sudo apt update
sudo apt install -y python3-venv python3-pip git wget
```

# Create Virtual Environment
Do not use the global pip (system-wide). Create an isolated environment to prevent permission errors and package collisions.
```
# Create the environment named 'trellis_env'
python3 -m venv trellis_env

# Activate the environment
source trellis_env/bin/activate

# Upgrade base tools inside the venv
pip install --upgrade pip setuptools wheel ninja
```

# Install ROCm 7.2 Wheels
```
# Install PyTorch, Vision, Audio, and Triton for ROCm 7.2 (Python 3.12)
pip3 install --no-cache-dir \
  https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/torch-2.9.1%2Brocm7.2.0.lw.git7e1940d4-cp312-cp312-linux_x86_64.whl \
  https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/torchvision-0.25.0%2Brocm7.2.0.gitaa35ca19-cp312-cp312-linux_x86_64.whl \
  https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/torchaudio-2.9.0%2Brocm7.2.0.gite3c6ee2b-cp312-cp312-linux_x86_64.whl \
  https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/triton-3.5.1%2Brocm7.2.0.gita272dfa8-cp312-cp312-linux_x86_64.whl
```

# Clean up
```
distrobox delete rocm7
distrobox rm rocm7
```
