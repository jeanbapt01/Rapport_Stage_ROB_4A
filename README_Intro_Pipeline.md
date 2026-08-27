# Industrial Imitation Learning Pipeline for Humanoid Robotics

An end-to-end industrial imitation learning framework for humanoid robotics, unifying multi-vendor teleoperation, physics simulation, dataset processing, and policy training/inference.

---

## 1. Project Goal & Overview
The objective of this repository is to establish a unified industrial imitation learning pipeline tailored for humanoid robots (specifically the 29-DoF Unitree G1 platform). This repository aggregates multiple core technologies and open-source frameworks into a single unified workspace to streamline data collection, dataset conversion, policy training, and evaluation.

### Aggregated Repositories
* **XR Teleoperation:** [unitreerobotics/xr_teleoperate](https://github.com/unitreerobotics/xr_teleoperate) (VR teleoperation via Apple Vision Pro, Meta Quest, Pico)
* **Unitree LeRobot:** [unitreerobotics/unitree_lerobot](https://github.com/unitreerobotics/unitree_lerobot) (Unitree integration & utilities for LeRobot)
* **Hugging Face LeRobot:** [huggingface/lerobot](https://github.com/huggingface/lerobot) (Core imitation learning policies and dataset structures)

---

## 2. Technical Pipeline Workflow

1. **Environment Initialization:** Environment setup using isolated virtual environments (Conda and `uv`).
2. **Teleoperation & Control:** Real-time teleoperation with Apple Vision Pro (AVP) or Meta Quest headsets.
3. **Data Recording:** Episode recording conducted across both simulation and real-world execution.
4. **Data Conversion & Preprocessing:**
   - Filtering and deletion of faulty recordings.
   - Format conversion from Unitree raw JSON into Hugging Face LeRobot format.
   - Detailed sub-step/sub-goal segmentation required when targeting GR00T-compliant formats.
5. **Policy Training:** Training execution using **Isaac-GR00T**.
6. **Inference & Benchmarking:** Inference deployment tested on a fixed-base 29-DoF Unitree G1 humanoid robot.

---

## 3. Implementation Status & Roadmap

### Validated Features (Current Baseline)
- [x] VR Teleoperation setup using Apple Vision Pro / Meta Quest.
- [x] Data recording routines in simulation and real-world environments.
- [x] Data preprocessing, deletion of bad recordings, and format conversions.
- [x] GR00T sub-step dataset division.
- [x] Model training execution with ACT and Isaac-GR00T.
- [x] Inference evaluation on a fixed Unitree G1 robot (29 DoF).

### Future Roadmap & Next Steps
- [ ] Transition to **NVIDIA Teleoperation** to broaden multi-robot compatibility beyond the G1.
- [ ] Integration of an advanced data pipeline: **Data Augmentation / NVIDIA Mimic / NVIDIA Cosmos**.

---

## 4. Hardware Requirements (HW Needed)
* **Humanoid Robot:** Unitree G1 (Fixed setup, 29 DoF).
* **End-Effectors:** Inspire FTP Dexterous Hands (Left: `192.168.123.210`, Right: `192.168.123.211`).
* **XR Headsets:** Apple Vision Pro (AVP), Meta Quest, or Pico.
* **Host PC:** High-performance PC with discrete NVIDIA GPU (x86_64 architecture).
* **Onboard Compute:** Unitree G1 Onboard Computer (AArch64 architecture).
* **Cameras:** Head-mounted Intel RealSense and wrist cameras.

---

## 5. Software Stack & Versions

### Virtual Environments Note
Due to strict version pinning and specific C++ library bindings across subsystems, components are organized into isolated environments:
* **`unitree_lerobot`**: Python 3.10, `pinocchio`, `ffmpeg==7.1.1`
* **`tv`**: Python 3.10, `pinocchio==3.1.0`, `numpy==1.26.4`, `params-proto==2.13.0`, `vuer[all]==0.0.60`, `aiohttp==3.10.5`
* **`inspire_hand`**: Python 3.10, `pymodbus==3.6.9`, `cyclonedds==0.10.2`
* **`teleimager`** (On-Robot): Python 3.10, `pyuvc`, `aiortc`
* **`Isaac-GR00T`**: Managed via `uv`, `torchcodec==0.8.0` (requires `ffmpeg` v4 to v7), NVIDIA Isaac Sim 5.0, NVIDIA Isaac-GR00T (vN1.7).
