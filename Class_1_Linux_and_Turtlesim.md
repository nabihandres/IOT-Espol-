# Class 1: Linux Fundamentals and Getting Started with ROS 2 Turtlesim 🐢🐧

This guide provides a detailed, step-by-step walkthrough to set up your environment in **Ubuntu 22.04 LTS**, master essential Linux terminal commands (Bash), and take your first steps in the **ROS 2 Humble** ecosystem using the **Turtlesim** simulator. You will learn how to inspect node architectures, publish and monitor topics in real time, and control the motion of a virtual robot.

## [For more details, see the presentation slides](https://docs.google.com/presentation/d/1Yl-QCOvlGdIBSREwMPxfj50a_lk7E9fm8FWiSNKZ6dg/edit)

---

## 📋 Prerequisites
Before starting, ensure your environment meets the following requirements:

* **Operating System:** [Ubuntu 22.04 LTS (Jammy Jellyfish)](https://releases.ubuntu.com/22.04/) installed on disk or on a bootable USB drive.
* **Minimum Storage:** Approximately **70 GB** of unallocated storage space for the Ubuntu partition.
* **Middleware:** [ROS 2 Humble Hawksbill](https://docs.ros.org/en/humble/Installation.html) installed on your system.
* **Terminal Tools:** Standard Bash terminal (`Ctrl + Alt + T`).

---

## Table of Contents
- [Prerequisites](#-prerequisites)
- [1. Ubuntu Preparation and Installation](#1-ubuntu-preparation-and-installation)
  - [1.1 Installation Considerations and Dual Boot](#11-installation-considerations-and-dual-boot)
  - [1.2 Verifying the Ubuntu Installation](#12-verifying-the-ubuntu-installation)
- [2. Essential Linux Terminal Commands](#2-essential-linux-terminal-commands)
  - [2.1 Quick Reference Table](#21-quick-reference-table)
  - [2.2 Hands-On File and Directory Management Exercise](#22-hands-on-file-and-directory-management-exercise)
- [3. Introduction to ROS 2: Nodes and Inspection with Turtlesim](#3-introduction-to-ros-2-nodes-and-inspection-with-turtlesim)
  - [3.1 Source the ROS 2 Environment](#31-source-the-ros-2-environment)
  - [3.2 Install the Turtlesim Package](#32-install-the-turtlesim-package)
  - [3.3 Launch the Turtlesim Simulator](#33-launch-the-turtlesim-simulator)
  - [3.4 Keyboard Control: Teleoperation](#34-keyboard-control-teleoperation)
  - [3.5 Inspecting Active Nodes (`ros2 node list`)](#35-inspecting-active-nodes-ros2-node-list)
  - [3.6 Detailed Node Inspection (`ros2 node info`)](#36-detailed-node-inspection-ros2-node-info)
- [4. Manual Topic Publishing and Analysis from Terminal](#4-manual-topic-publishing-and-analysis-from-terminal)
  - [4.1 Manual Circular Motion (`ros2 topic pub`)](#41-manual-circular-motion-ros2-topic-pub)
  - [4.2 Monitoring Published Data (`ros2 topic echo`)](#42-monitoring-published-data-ros2-topic-echo)
  - [4.3 Analyzing Publishing Frequency (`ros2 topic hz`)](#43-analyzing-publishing-frequency-ros2-topic-hz)
- [5. Deliverables and Grading Rubric](#5-deliverables-and-grading-rubric)
- [🤝 Contributing and Contact](#-contributing-and-contact)

---

## 1. Ubuntu Preparation and Installation

This section covers critical guidelines for setting up the host operating system required for robotics development.

### 1.1 Installation Considerations and Dual Boot

For in-lab sessions, Ubuntu should run directly on the machine's hardware:

1. **Bootable Drive Creation:** Flash the official Ubuntu 22.04 LTS ISO onto a USB drive (using software such as Rufus or BalenaEtcher) **before** coming to class.
2. **Storage Allocation:** Reserve an unallocated disk partition of approximately **70 GB** on the target computer.

> [!WARNING]
> **Extreme Caution with Windows Partitions:** If configuring a dual-boot setup alongside Windows, make sure to choose *"Install Ubuntu alongside Windows Boot Manager"* or configure partitions manually under *"Something else"*. Formatting the wrong partition could overwrite your Windows operating system and lead to irreversible data loss.

### 1.2 Verifying the Ubuntu Installation

Once booted into Ubuntu 22.04, launch a new terminal window by pressing:

```text
Ctrl + Alt + T
```

Run the following commands to verify distribution details and the Linux kernel version:

```bash
lsb_release -a
uname -r
```

The output should confirm `Ubuntu 22.04 LTS (Jammy)`:

```text
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.x LTS
Release:        22.04
Codename:       jammy
```

> [!NOTE]
> 📷 **Required Screenshot 1 (5 Points):** Take a full screenshot showing your Ubuntu desktop and the terminal displaying the output of `lsb_release -a`.

![Screenshot: Successful Ubuntu 22.04 Installation](assets/class1_ubuntu_installed.png)

---

## 2. Essential Linux Terminal Commands

Proficiency with ROS 2 relies heavily on comfortable use of the Linux Command Line Interface (CLI). Below is a summary of standard directory and file manipulation commands.

### 2.1 Quick Reference Table

| Command | Description | General Syntax | Practical Example |
| :--- | :--- | :--- | :--- |
| `ls` | Lists directory contents and file metadata | `ls [options] [path]` | `ls -la` |
| `cd` | Changes current working directory | `cd [directory]` | `cd ~/Documents` |
| `mkdir` | Creates one or more directories | `mkdir [options] [name]` | `mkdir -p my_folder/sub` |
| `touch` | Creates an empty file or updates timestamps | `touch [filename]` | `touch main.py` |
| `cp` | Copies files or directories | `cp [options] [source] [dest]` | `cp -r folder1 /backup/` |
| `mv` | Moves or renames files and directories | `mv [source] [dest]` | `mv old.txt new.txt` |
| `rm` | Deletes files or directories | `rm [options] [file/dir]` | `rm -rf obsolete_folder` |
| `rmdir` | Removes empty directories only | `rmdir [dir_name]` | `rmdir empty_folder` |

### 2.2 Hands-On File and Directory Management Exercise

Execute the following sequential commands in your terminal to practice path navigation and file management:

```bash
# 1. Navigate to the user home directory
cd ~

# 2. Create a practice folder hierarchy and enter it
mkdir -p linux_workshop/practice
cd linux_workshop

# 3. Create sample test files
touch notes.txt file1.txt file2.txt

# 4. List files with human-readable permissions and details
ls -lh

# 5. Rename a file
mv notes.txt final_report.txt

# 6. Move files into the subfolder
mv file1.txt file2.txt practice/

# 7. Copy the subfolder recursively as a backup
cp -r practice practice_backup

# 8. Delete files and directories
rm practice_backup/file1.txt
rm -rf practice_backup
```

Verify your final directory layout:

```bash
ls -R ~/linux_workshop
```

> [!NOTE]
> 📷 **Required Screenshot 2 (5 Points):** Capture a screenshot showing that you have successfully executed the listed terminal commands (`mkdir`, `touch`, `mv`, `ls`, etc.).

![Screenshot: Linux Terminal Commands Execution](assets/class1_terminal_commands.png)

---

## 3. Introduction to ROS 2: Nodes and Inspection with Turtlesim

**Turtlesim** is the standard educational simulator for ROS 2. It provides a visual, lightweight platform to understand how **nodes** communicate via **topics** and how velocity commands (`Twist`) affect physical motion.

### 3.1 Source the ROS 2 Environment

Before executing any ROS 2 command, you must source its environment setup file in the current terminal:

```bash
source /opt/ros/humble/setup.bash
```

> [!TIP]
> You can add this line to your `~/.bashrc` file by running `echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc` so that ROS 2 is automatically initialized in every newly opened terminal.

### 3.2 Install the Turtlesim Package

If you are working on a fresh or minimal Ubuntu installation, ensure Turtlesim is installed:

```bash
sudo apt update
sudo apt install -y ros-humble-turtlesim
```

Verify that the executables are available:

```bash
ros2 pkg executables turtlesim
```

The output should list `turtlesim_node` and `turtle_teleop_key`, among others.

---

### 3.3 Launch the Turtlesim Simulator

This lab requires working with **multiple independent terminal windows**:

```text
Terminal 1 → Graphical simulator (Turtlesim)
Terminal 2 → Keyboard teleoperation node
Terminal 3 → Inspection commands and topic publishing
```

Open **Terminal 1** and start the visual turtle simulation:

```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtlesim_node
```

A blue window featuring a turtle centered on screen will appear:

```text
[INFO] [turtlesim]: Starting turtlesim with node name /turtlesim
[INFO] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
```

> [!IMPORTANT]
> Keep **Terminal 1** running. Closing this terminal or pressing `Ctrl + C` will immediately stop the simulation.

---

### 3.4 Keyboard Control: Teleoperation

Open **Terminal 2** to run the keyboard teleoperation node:

```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtle_teleop_key
```

You will see the control banner in the console:

```text
Reading from keyboard
---------------------------
Use arrow keys to move the turtle.
Use G|B|V|C|D|E|R|T keys to rotate to absolute orientations. 'F' to cancel a rotation.
```

Use the **arrow keys** (Up, Down, Left, Right) to drive the turtle around the canvas.

> [!IMPORTANT]
> **Terminal 2** must remain active and focused for keyboard strokes to be captured and published to the turtle.

> [!NOTE]
> 📷 **Optional Screenshot:** Turtlesim window showing colored path lines after driving the turtle with teleoperation.

![Screenshot: Turtlesim Teleoperation](assets/class1_teleop_motion.png)

---

### 3.5 Inspecting Active Nodes (`ros2 node list`)

Open **Terminal 3** to inspect the active ROS graph without interrupting the simulator.

Execute the command below to list all currently running nodes:

```bash
source /opt/ros/humble/setup.bash
ros2 node list
```

The expected output will list both active nodes:

```text
/teleop_turtle
/turtlesim
```

> [!NOTE]
> 📷 **Required Screenshot 3 (1 Point):** Screenshot clearly showing the output of `ros2 node list` displaying the two active nodes (`/teleop_turtle` and `/turtlesim`).

![Screenshot: Active Nodes List](assets/class1_node_list.png)

---

### 3.6 Detailed Node Inspection (`ros2 node info`)

The command `ros2 node info` reveals the internal communication interfaces of a given node: Publishers, Subscribers, Service Servers/Clients, and Action Servers.

#### A. Inspecting the Simulator Node `/turtlesim`

In **Terminal 3**, run:

```bash
ros2 node info /turtlesim
```

Observe the reported interfaces:

```text
/turtlesim
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
  Service Servers:
    ...
```

> [!NOTE]
> Notice that `/turtlesim` is a **subscriber** to the `/turtle1/cmd_vel` topic, expecting messages of type `geometry_msgs/msg/Twist`.

#### B. Inspecting the Teleoperation Node `/teleop_turtle`

In the same **Terminal 3**, run:

```bash
ros2 node info /teleop_turtle
```

Observe that this node is the **publisher**:

```text
/teleop_turtle
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  ...
```

> [!NOTE]
> 📷 **Required Screenshot 4 (2 Points):** Screenshot showing the full console output of `ros2 node info /turtlesim` and `ros2 node info /teleop_turtle`.

![Screenshot: Node Info Inspection](assets/class1_node_info.png)

---

## 4. Manual Topic Publishing and Analysis from Terminal

ROS 2 allows you to publish data directly into topics from the command line without writing custom code.

### 4.1 Manual Circular Motion (`ros2 topic pub`)

Let us publish velocity messages directly to `/turtle1/cmd_vel` to command the turtle to move continuously in a circular path.

In **Terminal 3**, execute:

```bash
ros2 topic pub -r 10 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```

**Breakdown of command arguments:**
* `pub`: The action to publish a message.
* `-r 10`: Publishes periodically at a rate of **10 Hz** (10 times per second).
* `/turtle1/cmd_vel`: Destination topic name.
* `geometry_msgs/msg/Twist`: Message type definition.
* `"{linear: {x: 2.0...}, angular: {z: 1.8}}"`: YAML payload assigning forward linear velocity in X and rotational angular velocity around the Z axis.

> [!NOTE]
> Look at the Turtlesim window immediately: the turtle will start drawing a continuous circle.

> [!NOTE]
> 📷 **Required Screenshot 5 (2 Points):** Screenshot displaying both the terminal executing `ros2 topic pub` and the Turtlesim window showing the turtle moving in a circle.

![Screenshot: Circular Motion with ros2 topic pub](assets/class1_topic_pub_circle.png)

---

### 4.2 Monitoring Published Data (`ros2 topic echo`)

While the previous command continues running in **Terminal 3**, open **Terminal 4** to monitor messages on this topic in real time:

```bash
source /opt/ros/humble/setup.bash
ros2 topic echo /turtle1/cmd_vel
```

The terminal will print incoming `Twist` messages continuously:

```yaml
linear:
  x: 2.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 1.8
---
```

> [!NOTE]
> 📷 **Required Screenshot 6 (1.5 Points):** Screenshot showing the active streaming output of `ros2 topic echo /turtle1/cmd_vel`.

![Screenshot: Monitoring with ros2 topic echo](assets/class1_topic_echo.png)

---

### 4.3 Analyzing Publishing Frequency (`ros2 topic hz`)

In robotics, confirming that sensors and controllers publish data at expected frequencies is essential.

Open **Terminal 5** (or temporarily stop `echo` with `Ctrl + C`) and calculate the publishing rate:

```bash
source /opt/ros/humble/setup.bash
ros2 topic hz /turtle1/cmd_vel
```

After a brief sampling period, the terminal will report:

```text
average rate: 10.002
	min: 0.098s max: 0.102s std dev: 0.00114s window: 12
average rate: 9.998
	min: 0.098s max: 0.103s std dev: 0.00121s window: 23
```

The `average rate` is around **10.0 Hz**, exactly matching the `-r 10` parameter specified in the `pub` command.

> [!NOTE]
> 📷 **Required Screenshot 7 (1.5 Points):** Screenshot showing the publishing frequency in Hertz displayed by `ros2 topic hz`.

![Screenshot: Frequency Analysis with ros2 topic hz](assets/class1_topic_hz.png)

---

## 5. Deliverables and Grading Rubric

Your final submission must consist exclusively of a single **.pdf** or **.docx** document compiling all requested screenshots with clear titles, your full name, and date.

### Grading Rubric (Total: 10 Points)

| Assessment Criteria | Description | Points |
| :--- | :--- | :---: |
| **Nodes Running Correctly** | Proper simultaneous execution of `turtlesim_node` and `turtle_teleop_key` | **2.0 pts** |
| **Correct Use of `ros2 node list`** | Accurate command execution showing both active nodes | **1.0 pt** |
| **Correct Use of `ros2 node info`** | Complete output and inspection for both `/turtlesim` and `/teleop_turtle` | **2.0 pts** |
| **Functional Circular Movement** | Continuous circular motion achieved using `ros2 topic pub` | **2.0 pts** |
| **Use of `ros2 topic echo`** | Live inspection showing published `Twist` velocity messages | **1.5 pts** |
| **Use of `ros2 topic hz`** | Output demonstrating message publishing rate in Hz | **1.5 pts** |
| **Total** | **Maximum Workshop Score** | **10.0 pts** |

---

# 🤝 Contributing and Contact
Contributions and suggestions are always welcome! If you have recommendations to improve this guide, please feel free to fork the repository and submit a pull request.

✉️ **Teaching Team & Developers:**
1. Maykoll Vanegas - [LinkedIn](https://www.linkedin.com/in/maykoll-steven-vanegas-silva-baa225219/)
2. Hector La Mota - [LinkedIn](https://www.linkedin.com/in/h%C3%A9ctor-la-mota-5a9b7a2a8/)
3. Carolina Sánchez - [LinkedIn](https://www.linkedin.com/in/carolinas%C3%A1nchezcevallos/)

**AIROS - ESPOL** 🇪🇨
