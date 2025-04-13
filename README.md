

---

# 🧠 Azure IoT Edge with Raspberry Pi 3B+

This guide walks you through setting up **Azure IoT Edge** on a **Raspberry Pi 3B+** and on a **simulated device** for development and testing purposes.

---

## 🔧 Prerequisites

### 📦 Hardware
- Raspberry Pi 3B+
- 5V 2.5A Micro USB Power Supply
- Micro SD Card

### 💻 Software
- Azure IoT Hub
- Azure Container Registry
- **VS Code** with the following extensions:
  - Azure Account  
  - Azure IoT Edge  
  - Azure IoT Toolkit  
  - Docker  
  - Python (with pip)
- Environment variables set for:
  - `python`
  - `pip`
  - `wheels`
- `iotedgehubdev` installed

---

## 🍓 Raspberry Pi Setup

### 1. Install the Operating System

- Download **Raspbian Stretch** from [raspberrypi.org](https://www.raspberrypi.org/downloads/raspbian/)
  - Choose:
    - **Lite**: Command-line only
    - **Desktop**: GUI OS
    - **Desktop + Recommended Software**: For development
- Flash the image using [Balena Etcher](https://www.balena.io/etcher/)

---

### 2. First Boot & Configuration

- Default Credentials:
  - **Username:** `pi`
  - **Password:** `raspberry`

#### Optional: Enable Wi-Fi
```bash
sudo raspi-config
# Network Options > Wi-Fi > Set country, SSID, and passkey
```

#### Optional: Enable SSH
```bash
sudo raspi-config
# Interfacing Options > SSH > Enable
```

#### Check Network Configuration
```bash
ifconfig
```

#### Update the OS
```bash
sudo apt update && sudo apt full-upgrade -y
```

---

## 🐳 Install Container Runtime

### 1. Install Moby (Docker Engine for IoT Edge)

```bash
curl -L https://aka.ms/moby-engine-armhf-latest -o moby_engine.deb && sudo dpkg -i ./moby_engine.deb
curl -L https://aka.ms/moby-cli-armhf-latest -o moby_cli.deb && sudo dpkg -i ./moby_cli.deb
sudo apt-get install -f
```

---

## 🔐 Install IoT Edge Security Daemon

```bash
curl -L https://aka.ms/libiothsm-std-linux-armhf-latest -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb
curl -L https://aka.ms/iotedged-linux-armhf-latest -o iotedge.deb && sudo dpkg -i ./iotedge.deb
sudo apt-get install -f
```

---

## 🔗 Register Raspberry Pi with Azure IoT Hub

1. Go to the **Azure Portal**
2. Add a new IoT Edge Device
3. Copy the generated **Connection String**

### Configure IoT Edge

```bash
sudo nano /etc/iotedge/config.yaml
# Replace <ADD DEVICE CONNECTION STRING HERE> under manual provisioning
```

### Restart the Daemon

```bash
sudo systemctl restart iotedge
systemctl status iotedge  # check status
```

### Verify Setup

```bash
sudo iotedge list  # should list running modules
```

---

## 🖥️ Simulated Device Setup (For Dev & Testing)

> This section is for setting up a simulated IoT Edge device on Windows. (Linux supported too)

### 1. Enable Containers on Windows

- Go to **Windows Features**  
- Enable **Containers**  
- Restart your system

### 2. Install Docker

- Download from [Docker Desktop](https://www.docker.com/products/docker-desktop)  
- Switch to **Linux Containers** mode

---

### 3. Register Simulated Device

- Add a new IoT Edge Device in Azure
- Copy the Connection String

### 4. Install IoT Edge Daemon on Windows

Open **PowerShell ISE** as Administrator and run:

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression;
Install-SecurityDaemon -Manual -ContainerOs Linux -DeviceConnectionString '<connection-string>'
```

Check Daemon & Modules:
```powershell
Get-Service iotedge
iotedge list
docker images
```

---

## 🧪 Develop & Deploy IoT Edge Solution

### 📁 Create a New Solution in VS Code

1. **Command Palette** → `Azure IoT Edge: New IoT Edge Solution`
2. Select:
   - Language: **Python**
   - Module name: e.g., `SampleModule`
   - Replace `localhost:5000` in `module.json` with your Azure Container Registry:
     ```
     iotedgeraspberrypi.azurecr.io/samplemodule
     ```

### 🐳 Deploy to Simulated Device

```text
Right-click deployment.template.json → Build and Push IoT Edge Solution
Right-click deployment.template.json → Generate IoT Edge Deployment Manifest
Right-click deployment.amd64.json → Create Deployment for Single Device
```

Monitor messages:
```text
Right-click device in Azure IoT Hub extension → Start Monitoring D2C Message
```

---

### 🍓 Deploy to Raspberry Pi

1. Set platform to `arm32v7`
2. Build and push the module
3. Generate deployment manifest
4. Deploy `deployment.arm32v7.json` to the Pi

Monitor D2C messages from the Pi as you did with the simulated device.

---

## 📌 Notes

- Use `Azure IoT Hub: Stop monitoring D2C messages` from the Command Palette to stop monitoring.
- On resource-constrained devices, set `OptimizeForPerformance` to `false` in `config.yaml`.

---
