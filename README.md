## Installation and Configuration Guide for Ollama with CUDA Support on Debian 12

### Prerequisites

- Debian 12 (Bookworm)
- Supported NVIDIA GPU (e.g., RTX 5070)
- Root or `sudo` access

---

### 1. Uninstall Old NVIDIA Drivers (Optional but Recommended)

```bash
sudo apt-get remove --purge '^nvidia-.*' -y
sudo apt autoremove -y
sudo apt autoclean -y
```

---

### 2. Install Compilation Dependencies

```bash
sudo apt update
sudo apt install build-essential dkms mokutil openssl git curl -y
```

---

### 3. Install NVIDIA Drivers

```bash
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/575.51.02/NVIDIA-Linux-x86_64-575.51.02.run
chmod +x NVIDIA-Linux-x86_64-575.51.02.run
sudo ./NVIDIA-Linux-x86_64-575.51.02.run --no-kernel-modules
```

---

### 4. Build and Sign Open-Source NVIDIA Kernel Modules

```bash
git clone https://github.com/NVIDIA/open-gpu-kernel-modules.git
cd open-gpu-kernel-modules
git checkout 575.51.02
make modules -j$(nproc)
sudo make modules_install
sudo depmod
```

#### Create and Import MOK Key for Secure Boot (if enabled)

```bash
mkdir ~/kernel-signing && cd ~/kernel-signing
openssl genrsa -out MOK.priv 2048
openssl req -new -x509 -key MOK.priv -out MOK.pem -days 36500 -subj "/CN=NVIDIA Kernel Module Signing/"
openssl x509 -outform DER -in MOK.pem -out MOK.der
sudo mokutil --import MOK.der
# Reboot and enroll the key via the Secure Boot menu
```

#### Sign Modules After Reboot

```bash
sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 \
  ~/kernel-signing/MOK.priv \
  ~/kernel-signing/MOK.der \
  /lib/modules/$(uname -r)/kernel/drivers/video/nvidia.ko
sudo modprobe nvidia
```

---

### 5. Install CUDA Toolkit 12.9

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt install cuda-toolkit-12-9 -y
```

#### Add to ~/.bashrc

```bash
export PATH=/usr/local/cuda-12.9/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-12.9/lib64:$LD_LIBRARY_PATH
source ~/.bashrc
```

---

### 6. Install Ollama with CUDA (Binary Version)

```bash
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
sudo chmod +x /usr/bin/ollama
```

---

### 7. Start the Ollama Server

```bash
ollama serve
```

Check that the NVIDIA card is properly detected:

```
time=... source=types.go:130 msg="inference compute" ... name="NVIDIA GeForce RTX 5070 Ti"
```

---

### 8. Run a Model

```bash
ollama run llama3
```

---

### References

- https://ollama.com/
- https://developer.nvidia.com/cuda-downloads
- https://github.com/NVIDIA/open-gpu-kernel-modules
