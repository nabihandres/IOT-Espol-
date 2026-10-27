# Class 1: Ubuntu 22.04 Installation, Linux Basics & ROS 2 Turtlesim 🐢🐧

This repository provides a comprehensive guide to setting up **Ubuntu 22.04 LTS**, installing **ROS 2 Humble Hawksbill**, mastering core Linux terminal commands, and learning fundamental robotics concepts using the **Turtlesim** simulator.

---

## 📋 Prerequisites
Before starting, ensure you have:
* A working laptop or desktop computer running Windows (for dual-boot setup).
* A USB flash drive with at least **8 GB** capacity (all contents will be erased).
* At least **70 GB** of free, unallocated hard drive space for the Ubuntu system.
* Reliable internet access.

---

## Table of Contents
- [Prerequisites](#-prerequisites)
- [1. PC Setup & Ubuntu 22.04 Installation](#1-pc-setup--ubuntu-2204-installation)
  - [1.1 Create a Bootable Ubuntu USB](#11-create-a-bootable-ubuntu-usb)
  - [1.2 Create a Windows Partition for Ubuntu](#12-create-a-windows-partition-for-ubuntu)
  - [1.3 Install Ubuntu Step-by-Step](#13-install-ubuntu-step-by-step)
- [2. ROS 2 Humble Installation](#2-ros-2-humble-installation)
  - [2.1 Set Locale](#21-set-locale)
  - [2.2 Setup Software Sources and GPG Key](#22-setup-software-sources-and-gpg-key)
  - [2.3 Install ROS 2 Packages](#23-install-ros-2-packages)
  - [2.4 Environment Setup](#24-environment-setup)
  - [2.5 Recommended Additional Tools](#25-recommended-additional-tools)
- [3. Essential Linux Terminal Commands](#3-essential-linux-terminal-commands)
  - [3.1 Quick Reference Table](#31-quick-reference-table)
  - [3.2 Hands-On Terminal Practice](#32-hands-on-terminal-practice)
- [4. Introduction to ROS 2: Nodes & Inspection with Turtlesim](#4-introduction-to-ros-2-nodes--inspection-with-turtlesim)
  - [4.1 Run the Turtlesim Simulator](#41-run-the-turtlesim-simulator)
  - [4.2 Keyboard Control: Teleoperation](#42-keyboard-control-teleoperation)
  - [4.3 Inspecting Active Nodes (`ros2 node list`)](#43-inspecting-active-nodes-ros2-node-list)
  - [4.4 Detailed Node Inspection (`ros2 node info`)](#44-detailed-node-inspection-ros2-node-info)
- [5. Manual Topic Publishing and Telemetry Analysis](#5-manual-topic-publishing-and-telemetry-analysis)
  - [5.1 Manual Circular Motion (`ros2 topic pub`)](#51-manual-circular-motion-ros2-topic-pub)
  - [5.2 Real-Time Monitoring (`ros2 topic echo`)](#52-real-time-monitoring-ros2-topic-echo)
  - [5.3 Frequency Analysis (`ros2 topic hz`)](#53-frequency-analysis-ros2-topic-hz)
- [6. Deliverables and Grading Rubric](#6-deliverables-and-grading-rubric)
- [7. References](#7-references)
- [🤝 Contributing and Contact](#-contributing-and-contact)

---

## 1. PC Setup & Ubuntu 22.04 Installation

> [!NOTE]
> The screenshots below illustrate the standard Ubuntu installation wizard. The process is identical for Ubuntu 22.04 LTS.

🎥 **Video Reference:** [Ubuntu 22.04 Dual Boot Installation Video Guide](https://youtu.be/wIj7sHK0SkQ?si=t1UHW8nSEUa0G_Qe)

---

### 1.1 Create a Bootable Ubuntu USB

1. Download the official **Ubuntu 22.04 LTS Desktop Image** from:  
   👉 [Ubuntu 22.04 Releases](https://www.releases.ubuntu.com/22.04/)  
   Save the `.iso` file in an accessible location (e.g., `Downloads`).
2. Download **Rufus**, a free and open-source USB writing utility:  
   👉 [Download Rufus](https://rufus.ie/)
3. Connect your USB stick, open Rufus, and select your Ubuntu ISO file.
4. Select the partition scheme:
   * Use **GPT** if your computer uses modern UEFI BIOS (most PCs made after 2012).
   * Use **MBR** only if your computer uses legacy BIOS (pre-2010).
5. Click **START** to write the image to the USB.

![Rufus Partition Scheme GPT](https://github.com/user-attachments/assets/e5ade336-5928-41d8-a010-1ce1457fe477)

![Rufus Partition Scheme MBR](https://github.com/user-attachments/assets/61be5d51-1c54-4a1d-a8a2-c614d0f26f5a)

---

### 1.2 Create a Windows Partition for Ubuntu

Before installing, you must free up unallocated hard drive space from Windows:

1. In Windows, press `Win + R`, type `diskmgmt.msc`, and press **Enter** to open **Disk Management**.
2. Identify your main storage volume (usually `C:`). Right-click on it and choose **Shrink Volume**.

![Shrink Volume Option](https://github.com/user-attachments/assets/49e1d836-2f6d-4ae6-bbaf-20f2d9673bd6)

3. In the shrink dialog, allocate approximately **70,000 MB (approx. 70 GB)** for Ubuntu.
4. Click **Shrink**.

![Allocate Shrink Space](https://github.com/user-attachments/assets/b542b080-7cda-4df2-ae56-673ae9da7137)

5. Verify that a black block labeled **Unallocated** has been created with approximately 70 GB.

![Unallocated Space Verification](https://github.com/user-attachments/assets/2d1f5a20-95b5-4162-bf7d-614f5e88a60a)

---

### 1.3 Install Ubuntu Step-by-Step

1. Keep your bootable USB plugged in and restart your computer.
2. While booting, repeatedly press the BIOS/Boot Menu key (usually `F12`, `F2`, `F10`, or `Esc` depending on your motherboard) to open the Boot Menu.
3. Select your **USB drive** from the boot device list.
4. When the installer loads, select **Install Ubuntu**.
5. Choose your keyboard layout and language.

![Ubuntu Language Selection](https://github.com/user-attachments/assets/18bc02bf-06e5-4d9a-b719-6dc802e26027)

![Keyboard Layout](https://github.com/user-attachments/assets/5642eb49-ff5c-467d-a58a-49b915c75556)

6. Choose **Normal Installation** and check *"Install third-party software for graphics and Wi-Fi hardware"*.

![Installation Options](https://github.com/user-attachments/assets/f56a3bc4-ff86-42f4-bcbe-14b01302812d)

> [!WARNING]
> Under **Installation type**, you **MUST** select **"Something else"**. Selecting "Erase disk and install Ubuntu" will delete your Windows operating system and all your personal files.

![Installation Type: Something Else](https://github.com/user-attachments/assets/193cde16-4bf8-44b9-b92b-468bf4d4ea38)

![Partition Table Overview](https://github.com/user-attachments/assets/32ec72cc-3470-4bf2-b903-56c0b9229e98)

7. Find the row marked as **free space** (matching the ~70 GB partition you shrank earlier).
8. Click the **+** (Add) button below the partition list.

![Select Free Space and Click Plus](https://github.com/user-attachments/assets/dad3482e-cc69-4af3-b66b-c087e79a40b5)

9. Configure the root partition popup:
   * **Size:** Keep all available space (~70 GB).
   * **Type for the new partition:** *Logical*.
   * **Location for the new partition:** *Beginning of this space*.
   * **Use as:** *Ext4 journaling file system*.
   * **Mount point:** Select `/`.
10. Click **OK**.

![Create Root Partition](https://github.com/user-attachments/assets/ef7e39ec-a94c-4d3c-b9ed-a47ff1da92f2)

11. Select the new partition and click **Install Now**, then click **Continue** on the confirmation dialog.

![Confirm Install Now](https://github.com/user-attachments/assets/589e3e0f-149e-44ce-81df-ddc6c9a1155e)

12. Select your timezone and configure your username and password.

![Select Timezone](https://github.com/user-attachments/assets/de884502-d309-4dc9-ad59-a80226a09897)

![User Configuration](https://github.com/user-attachments/assets/ff6f17ee-719b-4082-afc6-4b1b97bd5f65)

13. Wait for the installation to finish and click **Restart Now**. When prompted, remove the USB stick and press **Enter**.

![Installation Progress](https://github.com/user-attachments/assets/b151c38a-c157-48e8-a9a3-16133ff9d53a)

![Restart Prompt](https://github.com/user-attachments/assets/354e66d1-a96d-42ad-b1ad-0fbaf8e11196)

> [!NOTE]
> 📷 **Required Screenshot 1 (5 Points):** After logging into Ubuntu, open a terminal (`Ctrl + Alt + T`) and run `lsb_release -a`. Take a full desktop screenshot showing the command output.

<img width="448" height="114" alt="Verificacion de ubuntu " src="https://github.com/user-attachments/assets/9ca06a7a-1ae0-4cad-ad11-ee8dc02b8250" />

---

## 2. ROS 2 Humble Installation

Official reference: [ROS 2 Humble Installation on Ubuntu (Debian Packages)](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)

Open a terminal (`Ctrl + Alt + T`) and follow these steps sequentially:

### 2.1 Set Locale

Ensure your system supports UTF-8:

```bash
sudo apt update && sudo apt install locales -y
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # Verify that settings show UTF-8
```

### 2.2 Setup Software Sources and GPG Key

Enable the Ubuntu Universe repository:

```bash
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
```

Install `curl` and import the official ROS 2 GPG keyring:

```bash
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

Add the ROS 2 repository to your APT sources list:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### 2.3 Install ROS 2 Packages

Update your package index and install the complete ROS 2 Desktop bundle:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y ros-humble-desktop
```

### 2.4 Environment Setup

Source the ROS 2 environment setup file:

```bash
source /opt/ros/humble/setup.bash
```

> [!TIP]
> Automatically source ROS 2 in every terminal by adding it to your `~/.bashrc`:
> ```bash
> echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
> ```

### 2.5 Recommended Additional Tools

**Terminator** is a powerful terminal multiplexer that allows you to split the screen into multiple panes:

```bash
sudo apt install -y terminator
```

---

## 3. Essential Linux Terminal Commands

### 3.1 Quick Reference Table

| Command | Description | Syntax | Practical Example |
| :--- | :--- | :--- | :--- |
| `ls` | Lists files and directories | `ls [options] [path]` | `ls -la ~/` |
| `cd` | Changes directory | `cd [directory]` | `cd ~/ros2_ws` |
| `mkdir` | Creates a new directory | `mkdir [options] [name]` | `mkdir -p project/src` |
| `touch` | Creates an empty file | `touch [filename]` | `touch main.py` |
| `cp` | Copies files or directories | `cp [options] [src] [dst]` | `cp -r folder1 /path/backup` |
| `mv` | Moves or renames files/directories | `mv [src] [dst]` | `mv old.txt new.txt` |
| `rm` | Deletes files or directories | `rm [options] [file/dir]` | `rm -rf obsolete/` |
| `rmdir` | Removes empty directories | `rmdir [directory]` | `rmdir empty_dir` |

### 3.2 Hands-On Terminal Practice

Run the following commands to practice filesystem operations:

```bash
cd ~
mkdir -p workshop_test/sub
cd workshop_test
touch notes.txt file1.txt file2.txt
ls -lh
mv notes.txt summary.txt
mv file1.txt file2.txt sub/
cp -r sub sub_backup
rm sub_backup/file1.txt
rm -rf sub_backup
ls -R ~/workshop_test
```

> [!NOTE]
> 📷 **Required Screenshot 2 (5 Points):** Take a screenshot showing that you have successfully executed the listed terminal commands in your terminal.

<img width="676" height="1015" alt="Comandos Ubuntu" src="https://github.com/user-attachments/assets/800c351e-c5fd-4c10-b1c3-a2bea76627a3" />

---

## 4. Introduction to ROS 2: Nodes & Inspection with Turtlesim

Turtlesim is the official 2D simulator for learning ROS 2 fundamentals.

Ensure Turtlesim is installed:

```bash
sudo apt update && sudo apt install -y ros-humble-turtlesim
```

### 4.1 Run the Turtlesim Simulator

Open **Terminal 1**:

```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtlesim_node
```

A blue graphic window will appear with a turtle in the center.

---

### 4.2 Keyboard Control: Teleoperation

Open **Terminal 2**:

```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtle_teleop_key
```

Use the **arrow keys** to drive the turtle.

> [!IMPORTANT]
> Keep **Terminal 2** selected and focused to capture arrow key strokes.

<img width="752" height="1051" alt="Turtlesim Teleop" src="https://github.com/user-attachments/assets/f81fd28c-29b3-49c5-a1ce-0a7b9c1d29eb" />

---

### 4.3 Inspecting Active Nodes (`ros2 node list`)

Open **Terminal 3**:

```bash
source /opt/ros/humble/setup.bash
ros2 node list
```

Expected output:

```text
/teleop_turtle
/turtlesim
```

> [!NOTE]
> 📷 **Required Screenshot 3 (1 Point):** Screenshot displaying the output of `ros2 node list` showing both active nodes.

<img width="1259" height="637" alt="Turtlesim Nodes" src="https://github.com/user-attachments/assets/9aa7b003-d4a4-49ff-93b7-4baf72c26b9e" />

---

### 4.4 Detailed Node Inspection (`ros2 node info`)

In **Terminal 3**, inspect both nodes:

```bash
ros2 node info /turtlesim
ros2 node info /teleop_turtle
```

> [!NOTE]
> Notice that `/turtlesim` is a **subscriber** to `/turtle1/cmd_vel`, while `/teleop_turtle` is the **publisher**.

> [!NOTE]
> 📷 **Required Screenshot 4 (2 Points):** Screenshot showing the complete output of `ros2 node info /turtlesim` and `ros2 node info /teleop_turtle`.

<img width="1310" height="1410" alt="Turtlesim Nodes Inspection" src="https://github.com/user-attachments/assets/6476ed62-26a3-4626-b23d-eb88a9de5e48" />

---

## 5. Manual Topic Publishing and Telemetry Analysis

### 5.1 Manual Circular Motion (`ros2 topic pub`)

In **Terminal 3**, send continuous velocity commands to move the turtle in a circle at 10 Hz:

```bash
ros2 topic pub -r 10 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

The turtle will immediately begin drawing continuous circular strokes.

> [!NOTE]
> 📷 **Required Screenshot 5 (2 Points):** Screenshot showing the `ros2 topic pub` command executing alongside the Turtlesim window with the turtle moving in a circle.

<img width="1310" height="1410" alt="Turtlesim Circle Move" src="https://github.com/user-attachments/assets/1b6d7c02-7f47-4688-9528-00dca0deeb5c" />

---

### 5.2 Real-Time Monitoring (`ros2 topic echo`)

Open **Terminal 4** to observe velocity messages published over the network:

```bash
source /opt/ros/humble/setup.bash
ros2 topic echo /turtle1/cmd_vel
```

> [!NOTE]
> 📷 **Required Screenshot 6 (1.5 Points):** Screenshot showing streaming `Twist` messages printed by `ros2 topic echo`.

<img width="1310" height="1410" alt="Turtlesim Topic Echo" src="https://github.com/user-attachments/assets/4020fbad-106b-4776-ae34-d210eb2ec6fc" />

---

### 5.3 Frequency Analysis (`ros2 topic hz`)

Open **Terminal 5** to measure publishing frequency:

```bash
source /opt/ros/humble/setup.bash
ros2 topic hz /turtle1/cmd_vel
```

Expected output confirms an average rate around **10.0 Hz**:

```text
average rate: 10.001
	min: 0.098s max: 0.102s std dev: 0.0012s window: 10
```

> [!NOTE]
> 📷 **Required Screenshot 7 (1.5 Points):** Screenshot showing the measured rate in Hz using `ros2 topic hz`.

<img width="1310" height="1410" alt="Turtlesim Topic Hz" src="https://github.com/user-attachments/assets/e0fc4380-2154-4aa1-9e32-5860617c81f6" />

---

## 6. Deliverables and Grading Rubric

Submit your work as a single **.pdf** or **.docx** document containing all 7 required screenshots:

| Item | Criterion | Points |
| :---: | :--- | :---: |
| 1 | **Ubuntu Installation:** `lsb_release -a` desktop screenshot | **5.0 pts** |
| 2 | **Linux Terminal Commands:** Successful command execution screenshot | **5.0 pts** |
| 3 | **Node List:** Output of `ros2 node list` | **2.0 pt** |
| 4 | **Node Info:** Complete output of `ros2 node info` for both nodes | **2.0 pts** |
| 5 | **Circular Movement:** `ros2 topic pub` with circular motion | **2.0 pts** |
| 6 | **Topic Echo:** Output of `ros2 topic echo /turtle1/cmd_vel` | **2.0 pts** |
| 7 | **Topic Hz:** Measured publishing frequency in Hz | **2.0 pts** |

---

## 7. References
- Ubuntu 22.04 LTS Installation: https://youtu.be/wIj7sHK0SkQ?si=t1UHW8nSEUa0G_Qe
- ROS 2 Humble Official Documentation: https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html
- Rufus USB Formatter: https://rufus.ie/
- Terminator Multiplexer: https://shanepark.tistory.com/313

---

# 🤝 Contributing and Contact
Feel free to open an issue or pull request if you find any typo or would like to propose improvements.

✉️ **Teaching Team & Authors:**
1. Maykoll Vanegas - [LinkedIn](https://www.linkedin.com/in/maykoll-steven-vanegas-silva-baa225219/)
