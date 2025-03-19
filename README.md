# Object Reconstruction Project Setup Guide

This guide provides detailed instructions on setting up, building, and running the Object Reconstruction wrapper for capturing and creating mesh files using Realsense and BundleSDF on a Windows 11 system using Docker and WSL 2.

---

## 1. Prerequisites

Before starting, ensure you have the following installed on your system:

- **Windows 11** with [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install) enabled.
- **Python 3.10**: Download and install it from the [official Python website](https://www.python.org/downloads/).
- **Docker Desktop** with WSL integration: Download and install it from [here](https://www.docker.com/products/docker-desktop/). Ensure that WSL integration is enabled in Docker settings.
- **Visual Studio Code (VSCode)** (optional) with the [Remote - WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl).
- **VcXsrv X Server** for GUI-based applications: Download it [here](https://sourceforge.net/projects/vcxsrv/).
- **NVIDIA GPU Driver and CUDA Toolkit** for WSL2: Install following [NVIDIA's official guide](https://docs.nvidia.com/cuda/wsl-user-guide/index.html).

---

## 2. Environment Setup

### a. Install WSL 2

To enable WSL 2 on Windows 11:

1. **Open PowerShell as Administrator**: Right-click on the Start button and select "Windows PowerShell (Admin)".

2. **Enable the WSL Feature**:

   ```powershell
   wsl --install
   ```

   This command will install the latest Ubuntu distribution by default. 

3. **Set WSL 2 as the Default Version**:

   ```powershell
   wsl --set-default-version 2
   ```

4. **Verify the Installation**:

   ```powershell
   wsl --list --verbose
   ```

   Ensure that your distribution is set to version 2.

### b. Install Docker Desktop with WSL 2 Integration

1. **Download Docker Desktop**: Get the installer from the [Docker Desktop for Windows page](https://docs.docker.com/desktop/install/windows-install/).

2. **Run the Installer**: Follow the on-screen instructions. During installation:

   - Ensure that the option "Use the WSL 2 based engine" is selected.
   - After installation, Docker Desktop will prompt you to enable WSL integration. Select your preferred WSL 2 distributions to integrate with Docker.

3. **Verify Docker Installation**:

   Open a new terminal and run:

   ```powershell
   docker --version
   ```

   This should display the Docker version installed.

### c. Install NVIDIA Drivers and CUDA for WSL 2

1. **Install NVIDIA Drivers**:

   - Download and install the NVIDIA CUDA-enabled driver for WSL from the [NVIDIA website](https://developer.nvidia.com/cuda/wsl).

2. **Install CUDA Toolkit**:

   - Open your WSL 2 terminal (e.g., Ubuntu) and run:

     ```bash
     wget https://developer.download.nvidia.com/compute/cuda/12.8.1/local_installers/cuda_12.8.1_570.124.06_linux.run
     sudo sh cuda_12.8.1_570.124.06_linux.run
     ```

     Follow the on-screen prompts during the installation process.

3. **Verify CUDA Installation**:

   Run:

   ```bash
   nvcc --version
   ```

   This should display the CUDA compiler version.

### d. Install NVIDIA Container Toolkit in WSL 2

1. **Set Up the Package Repository**:

   ```bash
   curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
   curl -s -L https://nvidia.github.io/libnvidia-container/stable/ubuntu$(lsb_release -rs)/$(arch)/nvidia-container-toolkit.list | \
     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
     sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
   ```

2. **Install the NVIDIA Container Toolkit**:

   ```bash
   sudo apt-get update
   sudo apt-get install -y nvidia-container-toolkit
   ```

3. **Restart the Docker Daemon**:

   ```bash
   sudo systemctl restart docker
   ```

4. **Verify the Installation**:

   ```bash
   sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
   ```

   This command should display your GPU information.

### e. Install VcXsrv X Server

1. **Download VcXsrv**: Get the installer from [SourceForge](https://sourceforge.net/projects/vcxsrv/).

2. **Install VcXsrv**: Follow the on-screen instructions.

3. **Install X11 Utilities in WSL**:

   ```bash
   sudo apt-get install x11-apps
   ```


4. **Configure Display Settings**:

   - Open PowerShell and run:

     ```powershell
     $env:DISPLAY="localhost:0.0"
     ```

   - To make this setting permanent, add the above line to your PowerShell profile script.

5. **Launch VcXsrv**:

   - Run "XLaunch" from the Start menu.
   - Select "Multiple windows" and click "Next".
   - Choose "Start no client" and click "Next".
   - Ensure "Disable access control" is checked and click "Finish".

6. **Verify VcXsrv Installation**:

   - Run "XLaunch" again (make sure icon visible in tray).
   - Open WSL in terminal and run:

     ```bash
     xclock
     ```

   - If the clock appears, VcXsrv is successfully installed.


---

## 3. Running the Object Reconstruction Pipeline

### a. Set up Virtual Environment and Install Dependencies

1. **Open a powershell terminal in the Object Reconstruction directory**

2. **Create a Virtual Environment**:

   ```powershell
   python -m venv venv
   ```

3. **Activate the Virtual Environment**:

   ```powershell
   .\venv\Scripts\activate
   ```

4. **Upgrade pip**:

   ```powershell
   python -m pip install --upgrade pip
   ```

5. **Install Dependencies**:

   ```powershell
   pip install -r requirements.txt
   ```

### b. Process Input Data

1. **Run the segmentation mask generation script**:

   ```powershell
   python rec_con_mask.py
   ```

2. **Run XMem segmentation**:

   ```powershell
   cd XMem
   python eval.py
   cd ..
   ```

3. **Organize Files**:

   ```powershell
   cd input
   mv video1 masks
   rmdir /s /q JPEGImages
   rmdir /s /q Annotations
   cd ..
   ```

### c. Build and Run the Docker Container

1. **Build the Docker Image**:

   ```powershell
   docker build --network host -t nvcr.io/nvidian/bundlesdf .
   ```

Note: Make sure XLaunch is running in Windows before running the following scripts.

2. **Run the Docker Container**:

   ```powershell
   bash run_container.sh
   ```

3. **Build the Project Inside the Container**:

   ```bash
   bash build.sh
   ```

### c. Run Object Reconstruction

1. **Perform 3D Reconstruction**:

   ```powershell
   bash run.sh
   ```
---

