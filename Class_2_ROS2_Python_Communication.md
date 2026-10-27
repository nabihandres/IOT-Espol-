# Class 2: Inter-Node Communication in ROS 2 with Turtlesim 🐢📐

This guide walks you through building a modular **ROS 2 Humble** package using **Python (`rclpy`)**. Following the course curriculum, you will learn how to create a workspace, build a custom package, and develop two interconnected nodes communicating via topics: an odometry subscriber that computes cumulative path length (`odom_listener.py`) and a geometric trajectory controller (`triangle_stepper.py`) that navigates an **equilateral triangle** pattern using real-time pose feedback.

---

## 📋 Prerequisites
Before starting, ensure you have:
* **Operating System:** [Ubuntu 22.04 LTS](https://releases.ubuntu.com/22.04/)
* **Middleware:** [ROS 2 Humble](https://docs.ros.org/en/humble/Installation.html)
* **Simulator:** `ros-humble-turtlesim` installed (`sudo apt install ros-humble-turtlesim`)
* **Build Tools:** `colcon` build system (`sudo apt install python3-colcon-common-extensions`)

---

## Table of Contents
- [Prerequisites](#-prerequisites)
- [1. Workshop Objectives & Architecture](#1-workshop-objectives--architecture)
- [2. Workspace Creation](#2-workspace-creation)
- [3. Creating the ROS 2 Package](#3-creating-the-ros-2-package)
- [4. Launching the Turtlesim Simulator](#4-launching-the-turtlesim-simulator)
- [5. Developing Subscriber Node: `odom_listener.py`](#5-developing-subscriber-node-odom_listenerpy)
  - [5.1 What Does This Node Do?](#51-what-does-this-node-do)
  - [5.2 Full Source Code](#52-full-source-code)
  - [5.3 Detailed Code Breakdown](#53-detailed-code-breakdown)
- [6. Developing Publisher Node: `triangle_stepper.py`](#6-developing-publisher-node-triangle_stepperpy)
  - [6.1 What Does This Node Do?](#61-what-does-this-node-do)
  - [6.2 Full Source Code](#62-full-source-code)
  - [6.3 Detailed Code Breakdown](#63-detailed-code-breakdown)
- [7. Package Configuration & Compilation](#7-package-configuration--compilation)
  - [7.1 Dependency Configuration in `package.xml`](#71-dependency-configuration-in-packagexml)
  - [7.2 Registering Executables in `setup.py`](#72-registering-executables-in-setuppy)
  - [7.3 Granting Execution Permissions](#73-granting-execution-permissions)
  - [7.4 Building with Colcon and Sourcing](#74-building-with-colcon-and-sourcing)
- [8. Running and Testing the Nodes](#8-running-and-testing-the-nodes)
  - [8.1 Multi-Terminal Execution](#81-multi-terminal-execution)
  - [8.2 Visualizing the Node Graph (`rqt_graph`)](#82-visualizing-the-node-graph-rqt_graph)
- [9. Deliverables and Grading Rubric](#9-deliverables-and-grading-rubric)
- [🤝 Contributing and Contact](#-contributing-and-contact)

---

## 1. Workshop Objectives & Architecture

The objective of this hands-on workshop is to develop two ROS 2 Python nodes communicating through topics to steer Turtlesim in a precise **equilateral triangular pattern** using odometry feedback:

```text
┌─────────────────────────┐                                 ┌─────────────────────────┐
│     Publisher Node      │ ──── [/turtle1/cmd_vel] ─────>  │        Simulator        │
│  (triangle_stepper.py)  │      (geometry_msgs/Twist)      │     (turtlesim_node)    │
└─────────────────────────┘                                 └─────────────────────────┘
             ▲                                                           │
             │                                                           │
             └─────────────── [/turtle1/pose] ───────────────────────────┤
                           (turtlesim/msg/Pose)                          ▼
                                                            ┌─────────────────────────┐
                                                            │     Subscriber Node     │
                                                            │   (odom_listener.py)    │
                                                            └─────────────────────────┘
                                                                         │
                                                               [/turtle1/path_length]
                                                                 (std_msgs/Float32)
```

* **`triangle_stepper.py`**: Computes the coordinates of 3 vertices of an equilateral triangle based on the initial spawn point. It turns towards each vertex, pauses for stability, and moves straight using feedback from `/turtle1/pose`.
* **`odom_listener.py`**: Subscribes to `/turtle1/pose`, continuously computes the Euclidean distance between consecutive points, and publishes the cumulative path length to `/turtle1/path_length`.

---

## 2. Workspace Creation

In ROS 2, custom packages must be developed inside a dedicated **Workspace**:

```bash
mkdir -p ~/ros2_ws/src
```

This creates the folder `ros2_ws` in your home directory along with the mandatory `src` subfolder where all source packages reside.

Now initialize the workspace by building it once:

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

The `colcon build` command scans `src/` and generates the `build/`, `install/`, and `log/` directories.

---

## 3. Creating the ROS 2 Package

Navigate into `src` and create the `turtle_controller` package with Python build type (`ament_python`) and required dependencies:

```bash
cd ~/ros2_ws/src
ros2 pkg create turtle_controller --build-type ament_python --dependencies rclpy geometry_msgs turtlesim std_msgs
```

Expected package layout:

```text
turtle_controller/
├── package.xml
├── setup.cfg
├── setup.py
└── turtle_controller/
    └── __init__.py
```

---

## 4. Launching the Turtlesim Simulator

Open a new terminal (**Terminal 1**) and launch Turtlesim:

```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtlesim_node
```

Keep this window open throughout the workshop.

---

## 5. Developing Subscriber Node: `odom_listener.py`

Navigate into the package's Python module directory:

```bash
cd ~/ros2_ws/src/turtle_controller/turtle_controller
touch odom_listener.py
chmod +x odom_listener.py
```

### 5.1 What Does This Node Do?

* Subscribes to the `/turtle1/pose` topic where Turtlesim publishes odometry.
* Calculates the total distance traveled by the turtle by accumulating incremental Euclidean distances between consecutive points (`math.hypot(dx, dy)`).
* Publishes this cumulative distance to `/turtle1/path_length` as `std_msgs/msg/Float32`.
* Prints real-time distance metrics to the console.

### 5.2 Full Source Code

Open `odom_listener.py` and insert the following code:

```python
import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose
from std_msgs.msg import Float32
import math


class OdomListener(Node):
    def __init__(self):
        super().__init__('odom_listener')
        self.subscription = self.create_subscription(
            Pose,
            '/turtle1/pose',
            self.pose_callback,
            10
        )
        self.publisher = self.create_publisher(Float32, '/turtle1/path_length', 10)

        self.last_pose = None
        self.total_distance = 0.0

    def pose_callback(self, msg):
        if self.last_pose is not None:
            dx = msg.x - self.last_pose.x
            dy = msg.y - self.last_pose.y
            increment = math.hypot(dx, dy)
            self.total_distance += increment

        self.last_pose = msg

        msg_out = Float32()
        msg_out.data = self.total_distance
        self.publisher.publish(msg_out)

        self.get_logger().info(f"Total distance traveled: {self.total_distance:.2f} units")


def main(args=None):
    rclpy.init(args=args)
    node = OdomListener()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

```

### 5.3 Detailed Code Breakdown

```python
import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose
from std_msgs.msg import Float32
import math
```
* Imports essential ROS 2 classes, the `Pose` message, the `Float32` standard message, and the math library.

```python
class OdomListener(Node):
    def __init__(self):
        super().__init__('odom_listener')
        self.subscription = self.create_subscription(Pose, '/turtle1/pose', self.pose_callback, 10)
        self.publisher = self.create_publisher(Float32, '/turtle1/path_length', 10)
```
* Defines node `odom_listener`.
* Creates a subscriber to `/turtle1/pose` calling `self.pose_callback`.
* Creates a publisher on `/turtle1/path_length` to broadcast cumulative travel distance.

```python
    def pose_callback(self, msg):
        if self.last_pose is not None:
            dx = msg.x - self.last_pose.x
            dy = msg.y - self.last_pose.y
            increment = math.hypot(dx, dy)
            self.total_distance += increment

        self.last_pose = msg
```
* On every pose update, calculates the Euclidean displacement $\Delta d = \sqrt{\Delta x^2 + \Delta y^2}$ and adds it to `self.total_distance`.

```python
        msg_out = Float32()
        msg_out.data = self.total_distance
        self.publisher.publish(msg_out)
        self.get_logger().info(f"Total distance traveled: {self.total_distance:.2f} units")
```
* Packages and publishes the cumulative distance and logs it to terminal.

---

## 6. Developing Publisher Node: `triangle_stepper.py`

Create the publisher node file:

```bash
cd ~/ros2_ws/src/turtle_controller/turtle_controller
touch triangle_stepper.py
chmod +x triangle_stepper.py
```

### 6.1 What Does This Node Do?

* Uses odometry feedback from `/turtle1/pose` to guide the turtle along an **equilateral triangle** of side length $L = 3.0$ units.
* Automatically computes 3 vertices $(P_1, P_2, P_3)$ relative to the initial spawn position $(x_0, y_0)$:
  $$P_1 = (x_0, y_0)$$
  $$P_2 = (x_0 + L, y_0)$$
  $$P_3 = \left(x_0 + \frac{L}{2}, y_0 + \frac{L \sqrt{3}}{2}\right)$$
* Executes a sequential state machine:
  * **`turn`**: Rotates in place until heading matches the angle towards the next vertex (`math.atan2(dy, dx)`).
  * **`pause`**: Halts for 0.3 seconds to stabilize orientation.
  * **`move`**: Drives forward along the side until the distance reached equals $L$.
* Continues cycling through vertices in a loop.

### 6.2 Full Source Code

Open `triangle_stepper.py` and insert the following code:

```python
import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose
from geometry_msgs.msg import Twist
import math


class TriangleStepper(Node):
    def __init__(self):
        super().__init__('triangle_stepper')
        self.pose = None
        self.state = 'turn'
        self.origin = None
        self.target = None
        self.angle_target = None
        self.pause_time = None

        self.subscription = self.create_subscription(Pose, '/turtle1/pose', self.update_pose, 10)
        self.publisher = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)
        self.timer = self.create_timer(0.1, self.control_loop)

        self.L = 3.0
        self.epsilon = 0.01
        self.vertices = []
        self.vertex_index = 0

    def update_pose(self, msg):
        self.pose = msg

    def control_loop(self):
        if not self.pose:
            return

        msg = Twist()

        # Initialize triangle vertices once using current position as anchor
        if not self.vertices:
            x0, y0 = self.pose.x, self.pose.y
            p1 = (x0, y0)
            p2 = (x0 + self.L, y0)
            p3 = (x0 + self.L / 2.0, y0 + (self.L * math.sqrt(3)) / 2.0)
            self.vertices = [p1, p2, p3]
            self.origin = p1
            self.target = p2
            self.vertex_index = 1
            self.state = 'turn'
            return

        dx = self.target[0] - self.pose.x
        dy = self.target[1] - self.pose.y
        angle_to_target = math.atan2(dy, dx)
        angle_diff = self.normalize_angle(angle_to_target - self.pose.theta)

        if self.state == 'turn':
            if abs(angle_diff) > self.epsilon:
                msg.angular.z = 1.0 * angle_diff
            else:
                msg.angular.z = 0.0
                self.state = 'pause'
                self.pause_time = self.get_clock().now()
                self.origin = (self.pose.x, self.pose.y)

        elif self.state == 'pause':
            now = self.get_clock().now()
            if (now - self.pause_time).nanoseconds / 1e9 > 0.3:
                self.state = 'move'

        elif self.state == 'move':
            traveled = math.hypot(self.pose.x - self.origin[0], self.pose.y - self.origin[1])
            if traveled < self.L:
                msg.linear.x = 1.5
            else:
                msg.linear.x = 0.0
                self.state = 'turn'
                self.vertex_index = (self.vertex_index + 1) % 3
                self.target = self.vertices[self.vertex_index]

        self.publisher.publish(msg)

    def normalize_angle(self, angle):
        return math.atan2(math.sin(angle), math.cos(angle))


def main(args=None):
    rclpy.init(args=args)
    node = TriangleStepper()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

```

### 6.3 Detailed Code Breakdown

```python
    def control_loop(self):
        if not self.pose:
            return
```
* Periodic callback running at 10 Hz (every 0.1s). Waits until the first pose message has arrived from `/turtle1/pose`.

```python
        if not self.vertices:
            x0, y0 = self.pose.x, self.pose.y
            p1 = (x0, y0)
            p2 = (x0 + self.L, y0)
            p3 = (x0 + self.L / 2.0, y0 + (self.L * math.sqrt(3)) / 2.0)
            self.vertices = [p1, p2, p3]
```
* Geometry setup: Constructs the 3 equilateral vertices relative to initial spawn coordinates.

```python
        dx = self.target[0] - self.pose.x
        dy = self.target[1] - self.pose.y
        angle_to_target = math.atan2(dy, dx)
        angle_diff = self.normalize_angle(angle_to_target - self.pose.theta)
```
* Determines the required steering angle towards the target vertex and normalizes the angle difference between $[-\pi, \pi]$.

```python
        if self.state == 'turn':
            if abs(angle_diff) > self.epsilon:
                msg.angular.z = 1.0 * angle_diff
            else:
                self.state = 'pause'
```
* Proportional steering controller aligns the turtle with the target direction before moving forward.

```python
        elif self.state == 'move':
            traveled = math.hypot(self.pose.x - self.origin[0], self.pose.y - self.origin[1])
            if traveled < self.L:
                msg.linear.x = 1.5
            else:
                self.vertex_index = (self.vertex_index + 1) % 3
                self.target = self.vertices[self.vertex_index]
```
* Moves forward until the displacement from the side origin equals $L=3.0$, then advances target vertex to $(index + 1) \pmod 3$.

---

## 7. Package Configuration & Compilation

### 7.1 Dependency Configuration in `package.xml`

Open `~/ros2_ws/src/turtle_controller/package.xml` and ensure all required dependencies are present inside the `<package>` block:

```xml
  <depend>rclpy</depend>
  <depend>geometry_msgs</depend>
  <depend>turtlesim</depend>
  <depend>std_msgs</depend>
```

### 7.2 Registering Executables in `setup.py`

Open `~/ros2_ws/src/turtle_controller/setup.py` and register both console scripts in `entry_points`:

```python
from setuptools import find_packages, setup

package_name = 'turtle_controller'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='anthony',
    maintainer_email='anthony@todo.todo',
    description='Turtlesim controller package for triangular motion',
    license='Apache-2.0',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'odom_listener = turtle_controller.odom_listener:main',
            'triangle_mover = turtle_controller.triangle_stepper:main',
        ],
    },
)
```

> [!NOTE]
> The alias `'triangle_mover = turtle_controller.triangle_stepper:main'` allows running the executable using `ros2 run turtle_controller triangle_mover`.

### 7.3 Granting Execution Permissions

Ensure both Python scripts have execution permissions:

```bash
cd ~/ros2_ws/src/turtle_controller/turtle_controller
chmod +x odom_listener.py
chmod +x triangle_stepper.py
```

### 7.4 Building with Colcon and Sourcing

Navigate to workspace root and compile:

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

Verify that both executables are registered:

```bash
ros2 pkg executables turtle_controller
```

Output:
```text
turtle_controller odom_listener
turtle_controller triangle_mover
```

---

## 8. Running and Testing the Nodes

### 8.1 Multi-Terminal Execution

#### Terminal 1: Launch Simulator
```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtlesim_node
```

#### Terminal 2: Run Odometry Subscriber Node
```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run turtle_controller odom_listener
```

You will see cumulative distance output printed:
```text
[INFO] [odom_listener]: Total distance traveled: 0.00 units
[INFO] [odom_listener]: Total distance traveled: 1.45 units
...
```

#### Terminal 3: Run Triangle Motion Publisher Node
```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run turtle_controller triangle_mover
```

The turtle will orient itself, move forward 3 units, turn towards the apex vertex, drive 3 units, turn towards base, and complete an equilateral triangle repeatedly.

---

### 8.2 Visualizing the Node Graph (`rqt_graph`)

In **Terminal 4**, launch the ROS graph visualizer:

```bash
source /opt/ros/humble/setup.bash
ros2 run rqt_graph rqt_graph
```

Verify that `/triangle_mover` publishes to `/turtle1/cmd_vel`, `/turtlesim` publishes to `/turtle1/pose`, and `/odom_listener` receives `/turtle1/pose` and publishes `/turtle1/path_length`.

---

## 9. Deliverables and Grading Rubric

### Deliverables
1. Source code file: `odom_listener.py`
2. Source code file: `triangle_mover.py` (or `triangle_stepper.py`)
3. A short YouTube video clearly showing the turtle completing the triangular pattern while both nodes are running.

### Grading Rubric (Total: 10 Points)

| Assessment Criteria | Description | Points |
| :--- | :--- | :---: |
| **Subscriber Node (`odom_listener.py`)** | Subscribes to `/turtle1/pose`, calculates cumulative distance, and logs data | **3.0 pts** |
| **Publisher Node (`triangle_mover.py`)** | Robust node driving turtle in triangular pattern using feedback from odometry | **3.0 pts** |
| **Communication & Organization** | Verified topic communication (`ros2 topic list` / `rqt_graph`) and clean package layout | **2.0 pts** |
| **Demonstration Video** | Clear video demonstrating the complete system running without errors | **2.0 pts** |
| **Total** | **Maximum Workshop Score** | **10.0 pts** |

---

# 🤝 Contributing and Contact
Feel free to open an issue or pull request if you find any typo or would like to propose improvements.

✉️ **Teaching Team & Authors:**
1. Maykoll Vanegas - [LinkedIn](https://www.linkedin.com/in/maykoll-steven-vanegas-silva-baa225219/)
