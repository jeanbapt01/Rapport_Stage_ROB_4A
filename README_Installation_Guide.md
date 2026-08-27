# Installation & Setup Guide: Unitree LeRobot & XR Teleoperation Framework

This guide details the step-by-step setup for **Unitree LeRobot**, **XR Teleoperation**, **Inspire Hand SDK**, **On-Robot Streaming**, and **NVIDIA Isaac-GR00T**.

---

## Table of Contents
1. [Unitree LeRobot Configuration](#1-unitree-lerobot-configuration)
2. [XR Teleoperation Setup (Host Laptop)](#2-xr-teleoperation-setup-host-laptop)
3. [SSL Certificate Configuration for XR Devices](#3-ssl-certificate-configuration-for-xr-devices)
4. [Inspire Hand SDK Configuration (Host Laptop)](#4-inspire-hand-sdk-configuration-host-laptop)
5. [On-Robot Configuration (Unitree G1)](#5-on-robot-configuration-unitree-g1)
6. [How to Perform Teleoperation & Record Training Data](#6-how-to-perform-teleoperation--record-training-data)
7. [Data Processing & LeRobot Conversion](#7-data-processing--lerobot-conversion)
8. [NVIDIA Isaac-GR00T Installation](#8-nvidia-isaac-gr00t-installation)

---

## 1. Unitree LeRobot Configuration
*Reference: [unitree_lerobot GitHub](https://github.com/unitreerobotics/unitree_lerobot/tree/main)*

### Clone the Repository
```bash
# New installation
git clone --recurse-submodules https://github.com/unitreerobotics/unitree_lerobot.git
cd unitree_lerobot

# If already downloaded without submodules:
git submodule update --init --recursive
```

### Create and Configure the Conda Environment
```bash
conda create -y -n unitree_lerobot python=3.10
conda activate unitree_lerobot

# Install essential core dependencies
conda install pinocchio -c conda-forge
conda install ffmpeg=7.1.1 -c conda-forge
```

### Install LeRobot and Unitree SDK2
```bash
# Install the LeRobot package in editable mode
pip install -e .

# Install Unitree SDK2 Python bindings
cd ~
git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
cd unitree_sdk2_python
pip install -e .
```

---

## 2. XR Teleoperation Setup (Host Laptop)
*Reference: [xr_teleoperate GitHub](https://github.com/unitreerobotics/xr_teleoperate/tree/main)*

### Environment Setup
```bash
conda create -n tv python=3.10 pinocchio=3.1.0 numpy=1.26.4 -c conda-forge
conda activate tv
```

### Clone and Install Submodules
```bash
cd ~
git clone https://github.com/unitreerobotics/xr_teleoperate.git
cd xr_teleoperate

# Shallow clone submodules to speed up download
git submodule update --init --depth 1

# Install televuer module
cd teleop/televuer
pip install -e .

# Install teleimager module
cd ../teleimager
pip install -e . --no-deps

# Install dex retargeting module
cd ../robot_control/dex-retargeting/
pip install -e .
```

### Install Requirements and Resolve Conflicts
```bash
cd ../../../ # Make sure you are at the root of /xr_teleoperate

pip install -r requirements.txt

# Manually resolve pinning conflicts
pip uninstall vuer params-proto aiohttp aiohttp-cors -y
pip install "params-proto == 2.13.0"
pip install "vuer[all] == 0.0.60" --no-deps
pip install aiohttp == 3.10.5 aiohttp-cors killport waterbear
```

---

## 3. SSL Certificate Configuration for XR Devices
Configure SSL certificates for the `televuer` module to ensure secure HTTPS / WebRTC streaming from XR headsets.

### Option A: For Apple Vision Pro (AVP)
#### 1. Generate certificate files:
```bash
cd ~/xr_teleoperate/teleop/televuer

openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 365 -out rootCA.pem -subj "/CN=xr-teleoperate"
openssl genrsa -out key.pem 2048
openssl req -new -key key.pem -out server.csr -subj "/CN=localhost"
```

#### 2. Create the configuration extension file (`server_ext.cnf`):
```ini
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
IP.1 = 192.168.1.10
IP.2 = 192.168.1.
IP.3 = 192.168.123.164
IP.4 = 192.168.123.2
```

#### 3. Sign the certificate and open firewall:
```bash
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out cert.pem -days 365 -sha256 -extfile server_ext.cnf
sudo ufw allow 8012
```

#### 4. Transfer `rootCA.pem` to Apple Vision Pro:
*   **Mac Users:** Use AirDrop.
*   **Non-Mac Users (Snapdrop):**
    1. Connect both host PC and AVP to the same local Wi-Fi.
    2. Open [snapdrop.net](https://snapdrop.net) on both PC and AVP Safari.
    3. Transfer `rootCA.pem` and install the profile in AVP Settings.

### Option B: For Meta Quest / Pico
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem
```

### Global Certificate Deployment
```bash
mkdir -p ~/.config/xr_teleoperate/
cp cert.pem key.pem ~/.config/xr_teleoperate/
```

---

## 4. Inspire Hand SDK Configuration (Host Laptop)
*Ensure hands are reachable over local network (Right default: `192.168.123.211`, Left default: `192.168.123.210`).*

### Environment Setup & Installation
```bash
conda create -n inspire_hand python=3.10 -y
conda activate inspire_hand

cd ~
git clone https://github.com/NaCl-1374/inspire_hand_ws
cd inspire_hand_ws

pip install -r requirements.txt
git submodule init
git submodule update

sudo apt install -y bison cmake build-essential

# Install SDK submodules
cd unitree_sdk2_python
pip install -e .

cd ../inspire_hand_sdk
conda install pyqt
pip install -e . --no-deps

pip install pymodbus==3.6.9
pip install pyserial colorcet pyqtgraph
```

### Build CycloneDDS C Library
```bash
cd ~
git clone https://github.com/eclipse-cyclonedds/cyclonedds -b 0.10.2
cd cyclonedds
mkdir build && cd build
sudo cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
sudo cmake --build . --target install
sudo ldconfig

export CYCLONEDDS_HOME=/usr/local
```

### SDK Verification Test
```bash
cd ~/inspire_hand_ws/inspire_hand_sdk/example
python Headless_driver_l.py
```

---

## 5. On-Robot Configuration (Unitree G1)

### 1. Connection & Network Configuration
*   Connect Ethernet cable directly from host laptop to Unitree G1.
*   Configure static IP on host (e.g., `192.168.123.X`).
*   SSH into onboard computer:
```bash
ping 192.168.123.164
ssh unitree@192.168.123.164
```

### 2. Environment Setup (On-Robot)
```bash
# Run on Host Laptop to push Miniconda:
cd ~
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh
scp Miniconda3-latest-Linux-aarch64.sh unitree@192.168.123.164:~

# Run inside Robot SSH terminal:
mkdir -p ~/miniconda3 
mv ~/Miniconda3-latest-Linux-aarch64.sh ~/miniconda3/miniconda.sh 
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3 
source ~/miniconda3/bin/activate

conda create -n teleimager python=3.10 -y
conda activate teleimager

cd ~
git clone https://github.com/silencht/teleimager
cd teleimager
pip install -e .
bash setup_uvc.sh
pip install aiohttp aiortc
sudo apt-get install -y libusb-1.0-0-dev libuvc-dev cython3 
pip install git+https://github.com/pupil-labs/pyuvc.git
```

#### Synchronize SSL Certificates:
```bash
# On Host Laptop:
scp ~/xr_teleoperate/teleop/televuer/key.pem ~/xr_teleoperate/teleop/televuer/cert.pem unitree@192.168.123.164:~/teleimager

# On Robot SSH:
cd ~/teleimager
mkdir -p ~/.config/xr_teleoperate/
cp cert.pem key.pem ~/.config/xr_teleoperate/
```

### 3. Camera Discovery and Server Running
```bash
# Scan and generate camera configuration
teleimager-server --cf

# Configure ~/teleimager/cam_config_server.yaml as needed, then start server:
teleimager-server
```

### 4. On-Robot Inspire Hand Setup
Repeat **Section 4: Inspire Hand SDK Configuration** natively inside the robot SSH shell.

---

## 6. How to Perform Teleoperation & Record Training Data

Open **3 separate terminals** on your Host PC:

### Terminal 1: Robot (Hand Driver Connection)
```bash
ssh unitree@192.168.123.164
source ~/miniconda3/bin/activate
conda activate inspire_hand

unset ROS_VERSION ROS_PYTHON_VERSION AMENT_PREFIX_PATH PYTHONPATH LD_LIBRARY_PATH ROS_LOCALHOST_ONLY ROS_DISTRO
python ~/inspire_hand_ws/inspire_hand_sdk/example/Headless_driver_r.py
```

### Terminal 2: Robot (WebRTC Camera Streaming Server)
```bash
ssh unitree@192.168.123.164
source ~/miniconda3/bin/activate
conda activate teleimager

unset ROS_VERSION ROS_PYTHON_VERSION AMENT_PREFIX_PATH PYTHONPATH LD_LIBRARY_PATH ROS_LOCALHOST_ONLY ROS_DISTRO
cd teleimager
teleimager-server
```

### Terminal 3: Host PC (Teleoperation Master Core & Recorder)
```bash
unset ROS_VERSION ROS_PYTHON_VERSION AMENT_PREFIX_PATH PYTHONPATH LD_LIBRARY_PATH ROS_LOCALHOST_ONLY ROS_DISTRO

conda activate tv
cd ~/xr_teleoperate/teleop

PYTHONPATH=. python -m teleop_hand_and_arm --input-mode hand --arm G1_29 --ee inspire_ftp --motion --image-server 192.168.123.164 --record
```

Open headset browser to: `https://192.168.123.x:8012/?ws=wss://192.168.123.x:8012`

---

## 7. Data Processing & LeRobot Conversion

### 1. Interactive Data Labeling & Editing
```bash
cd ~/unitree_lerobot/data_editor
conda activate unitree_lerobot
pip install PyQt5
python data_editor_EN.py
```

### 2. Format Structuring & Conversion
```bash
# Sort folders
python unitree_lerobot/utils/sort_and_rename_folders.py --data_dir $HOME/datasets/task_name

# Convert raw JSON to LeRobot format
python unitree_lerobot/utils/convert_unitree_json_to_lerobot.py     --raw-dir $HOME/datasets     --repo-id your_hf_username/repo_task_name     --robot_type Unitree_G1_Dex3     --push_to_hub
```

---

## 8. NVIDIA Isaac-GR00T Installation

```bash
sudo apt update && sudo apt install -y git-lfs
git lfs install

git clone --recurse-submodules https://github.com/NVIDIA/Isaac-GR00T
cd Isaac-GR00T

curl -LsSf https://astral.sh/uv/install.sh | sh
sudo apt-get install -y ffmpeg

uv run python -c "import gr00t; print('GR00T installed successfully')"
uv run huggingface-cli login
uv run bash scripts/patch_triton_cuda13.sh
```
