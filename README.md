# IoT & Robotics Workshop - ROS 2 Humble & Linux Fundamentals 🤖🚀

This repository contains complete, step-by-step educational materials designed to introduce students to Linux operating system essentials, ROS 2 middleware architecture, and practical robot control using the Turtlesim simulator.

## 📅 Workshop Syllabus & Modules

| Module | Day | Description | Guide Link |
| :--- | :--- | :--- | :--- |
| **Class 1** | Tuesday | **Linux Fundamentals & ROS 2 Turtlesim CLI**<br>Ubuntu 22.04 installation guidelines, terminal commands, node inspection (`ros2 node list/info`), manual circular motion publishing (`ros2 topic pub`), message monitoring (`echo`), and frequency measurement (`hz`). | [Open Class 1 Guide ➜](Class_1_Linux_and_Turtlesim.md) |
| **Class 2** | Wednesday | **Inter-Node Communication with Python & Turtlesim**<br>Workspace scaffolding (`ros2_ws`), Python package creation (`turtle_controller`), odometry subscriber node (`odom_listener.py`), and geometric trajectory publisher (`triangle_mover.py`) forming a triangular pattern. | [Open Class 2 Guide ➜](Class_2_ROS2_Python_Communication.md) |
| **Class 3** | Thursday | **Web Interfaces for Robot Monitoring & Teleoperation**<br>Full-stack robotics web architecture, `rosbridge_suite` WebSocket communication, `web_video_server` MJPEG streaming, Canvas 2D grid map visualization, sensor telemetry (IMU/LiDAR), and Nav2 autonomous goal dispatching. | [Open Class 3 Guide ➜](Class_3_ROS2_Web_Interface.md) |

---

## 📋 General Prerequisites
Before starting the sessions, ensure that you have:
* **Operating System:** [Ubuntu 22.04 LTS (Jammy Jellyfish)](https://releases.ubuntu.com/22.04/) with ~70 GB disk space.
* **Middleware:** [ROS 2 Humble Hawksbill](https://docs.ros.org/en/humble/Installation.html).
* **Simulators:** `ros-humble-turtlesim` and Gazebo Classic 11 (`ros-humble-gazebo-*`).
* **Web & Navigation Packages:** `ros-humble-rosbridge-server`, `ros-humble-web-video-server`, `ros-humble-navigation2`, `ros-humble-nav2-bringup`.
* **Build Tools:** `python3-colcon-common-extensions`.

---

## 📂 Repository Layout

```text
IOT-Espol-/
├── README.md                               # Course overview & navigation (this file)
├── Class_1_Linux_and_Turtlesim.md          # Complete Tuesday tutorial
├── Class_2_ROS2_Python_Communication.md   # Complete Wednesday tutorial
├── Class_3_ROS2_Web_Interface.md           # Complete Thursday tutorial
└── assets/                                 # Screenshots and media assets
```

---

## 🤝 Contributing and Contact
Feel free to open an issue or pull request if you find any typo or would like to propose improvements.

✉️ **Teaching Team & Authors:**
1. Maykoll Vanegas - [LinkedIn](https://www.linkedin.com/in/maykoll-steven-vanegas-silva-baa225219/)
2. Hector La Mota - [LinkedIn](https://www.linkedin.com/in/h%C3%A9ctor-la-mota-5a9b7a2a8/)
3. Carolina Sánchez - [LinkedIn](https://www.linkedin.com/in/carolinas%C3%A1nchezcevallos/)
