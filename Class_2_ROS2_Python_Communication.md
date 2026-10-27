# Class 2: Inter-Node Communication in ROS 2 Using Python and Turtlesim 🐢📐

This hands-on workshop walks you through building a modular **ROS 2 Humble** solution using **Python (`rclpy`)**. You will learn how to create a ROS 2 workspace, scaffold a package from scratch, and develop two decoupled nodes that communicate over topics: an odometry subscriber (`odom_listener.py`) and a geometric trajectory publisher (`triangle_mover.py`) to steer the turtle along a **triangular** path.

## [For more details, see the workshop slides](https://docs.google.com/presentation/d/1Yl-QCOvlGdIBSREwMPxfj50a_lk7E9fm8FWiSNKZ6dg/edit)

---

## 📋 Prerequisites
Before starting this workshop, ensure your development environment satisfies:

* **Operating System:** [Ubuntu 22.04 LTS](https://releases.ubuntu.com/22.04/)
* **Middleware:** [ROS 2 Humble](https://docs.ros.org/en/humble/Installation.html)
* **Packages Installed:** `ros-humble-turtlesim` and build tools (`python3-colcon-common-extensions`)
* **Recommended Code Editor:** Visual Studio Code or any lightweight text editor (Nano, Gedit).

---

## Table of Contents
- [Prerequisites](#-prerequisites)
- [1. System Architecture](#1-system-architecture)
- [2. Workspace Creation and Directory Structure](#2-workspace-creation-and-directory-structure)
  - [2.1 Create the Workspace (`ros2_ws`)](#21-create-the-workspace-ros2_ws)
  - [2.2 Create the `turtle_controller` Package](#22-create-the-turtle_controller-package)
  - [2.3 Generated Package Layout](#23-generated-package-layout)
- [3. Developing the Subscriber Node: `odom_listener.py`](#3-developing-the-subscriber-node-odom_listenerpy)
  - [3.1 Understanding the `/turtle1/pose` Topic](#31-understanding-the-turtle1pose-topic)
  - [3.2 Full Source Code of `odom_listener.py`](#32-full-source-code-of-odom_listenerpy)
- [4. Developing the Publisher Node: `triangle_mover.py`](#4-developing-the-publisher-node-triangle_moverpy)
  - [4.1 Geometry of the Triangular Pattern](#41-geometry-of-the-triangular-pattern)
  - [4.2 Full Source Code of `triangle_mover.py`](#42-full-source-code-of-triangle_moverpy)
- [5. Package Configuration and Compilation](#5-package-configuration-and-compilation)
  - [5.1 Configure Dependencies in `package.xml`](#51-configure-dependencies-in-packagexml)
  - [5.2 Register Console Entry Points in `setup.py`](#52-register-console-entry-points-in-setuppy)
  - [5.3 Building with Colcon](#53-building-with-colcon)
  - [5.4 Sourcing the Local Workspace](#54-sourcing-the-local-workspace)
- [6. Execution, Testing, and System Validation](#6-execution-testing-and-system-validation)
  - [6.1 Recommended Execution Order](#61-recommended-execution-order)
  - [6.2 Verifying Active Nodes and Topics](#62-verifying-active-nodes-and-topics)
  - [6.3 Visualizing the Node Graph with `rqt_graph`](#63-visualizing-the-node-graph-with-rqt_graph)
- [7. Deliverables and Grading Rubric](#7-deliverables-and-grading-rubric)
- [🤝 Contributing and Contact](#-contributing-and-contact)

---

## 1. System Architecture

In this workshop, we will implement the foundational ROS 2 communication paradigm: **Publisher / Subscriber**.

The communication topology and message flow operate as follows:

```text
┌───────────────────────┐                               ┌───────────────────────┐
│    Publisher Node     │ ──── [/turtle1/cmd_vel] ────> │       Simulator       │
│  (triangle_mover.py)  │     (geometry_msgs/Twist)     │    (turtlesim_node)   │
└───────────────────────┘                               └───────────────────────┘
                                                                    │
                                                           [/turtle1/pose]
                                                         (turtlesim/msg/Pose)
                                                                    ▼
                                                        ┌───────────────────────┐
                                                        │    Subscriber Node    │
                                                        │   (odom_listener.py)  │
                                                        └───────────────────────┘
```

1. **`triangle_mover`**: Computes linear and angular velocity commands to generate 3 equal sides and 3 turns of an equilateral triangle, publishing them to `/turtle1/cmd_vel`.
2. **`turtlesim_node`**: Receives velocity commands, moves the turtle in the canvas, and continuously publishes its real-time 2D pose to `/turtle1/pose`.
3. **`odom_listener`**: Subscribes to `/turtle1/pose` and outputs coordinates $(X, Y)$ and orientation $(\theta)$ to the console.

---

## 2. Workspace Creation and Directory Structure

In ROS 2, all custom development happens inside a **Workspace**.

### 2.1 Create the Workspace (`ros2_ws`)

Open a terminal and set up the workspace root directory along with its required `src` folder:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

### 2.2 Create the `turtle_controller` Package

Inside the `src` folder, invoke `ros2 pkg create` to scaffold a Python-based ROS 2 package (`ament_python`):

```bash
ros2 pkg create --build-type ament_python turtle_controller --dependencies rclpy geometry_msgs turtlesim
```

**Breakdown of command arguments:**
* `--build-type ament_python`: Declares that this package contains Python scripts and modules.
* `turtle_controller`: The name of the package.
* `--dependencies rclpy geometry_msgs turtlesim`: Automatically includes dependencies in the manifest: Python ROS 2 client library (`rclpy`), standard velocity messages (`geometry_msgs`), and simulator interfaces (`turtlesim`).

### 2.3 Generated Package Layout

Verify the generated package structure:

```bash
ls -R ~/ros2_ws/src/turtle_controller
```

The directory tree will look like this:

```text
ros2_ws/
└── src/
    └── turtle_controller/
        ├── package.xml               # Package metadata and dependencies
        ├── setup.py                  # Installation script and console entry points
        ├── setup.cfg                 # Install configuration
        ├── resource/
        │   └── turtle_controller
        └── turtle_controller/        # Python module folder
            └── __init__.py
```

---

## 3. Developing the Subscriber Node: `odom_listener.py`

### 3.1 Understanding the `/turtle1/pose` Topic

The Turtlesim simulator publishes the turtle's odometry through the `turtlesim/msg/Pose` message type. You can inspect its fields directly from the terminal:

```bash
source /opt/ros/humble/setup.bash
ros2 interface show turtlesim/msg/Pose
```

Expected output:

```text
float32 x               # Horizontal coordinate
float32 y               # Vertical coordinate
float32 theta           # Orientation angle in radians [-pi to pi]
float32 linear_velocity # Instantaneous linear velocity
float32 angular_velocity# Instantaneous angular velocity
```

### 3.2 Full Source Code of `odom_listener.py`

Create a new file for the subscriber node inside the package module directory:

```bash
touch ~/ros2_ws/src/turtle_controller/turtle_controller/odom_listener.py
```

Open the file in your text editor and insert the following code:

```python
#!/usr/bin/env python3
"""
Odometry Subscriber Node for Turtlesim
Author: AIROS - ESPOL
Description: Subscribes to /turtle1/pose and logs position and orientation coordinates.
"""

import rclpy
from rclpy.node import Node
from turtlesim.msg import Pose


class OdomListener(Node):
    def __init__(self):
        # 1. Initialize the node with the name 'odom_listener'
        super().__init__('odom_listener')

        # 2. Subscribe to the '/turtle1/pose' topic
        # Parameters: Message Type, Topic Name, Callback Function, QoS Queue Size
        self.subscription = self.create_subscription(
            Pose,
            '/turtle1/pose',
            self.pose_callback,
            10
        )
        self.subscription  # Prevent unused variable warning

        self.get_logger().info('Node [odom_listener] initialized and listening to /turtle1/pose...')

    def pose_callback(self, msg: Pose):
        """
        Callback triggered asynchronously whenever a new Pose message arrives.
        """
        # Format the log output for clear console reading
        log_message = (
            f"Pose -> X: {msg.x:.2f} m | Y: {msg.y:.2f} m | "
            f"Orientation (Theta): {msg.theta:.2f} rad ({msg.theta * 180.0 / 3.14159:.1f}°)"
        )
        self.get_logger().info(log_message)


def main(args=None):
    rclpy.init(args=args)
    node = OdomListener()

    try:
        # Keep processing incoming message callbacks
        rclpy.spin(node)
    except KeyboardInterrupt:
        node.get_logger().info('Shutting down odom_listener node...')
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

> [!NOTE]
> This node relies on an event-driven *callback* function named `pose_callback`. There is no need to manually poll the topic: whenever Turtlesim publishes a new pose, ROS 2 triggers this callback automatically.

---

## 4. Developing the Publisher Node: `triangle_mover.py`

### 4.1 Geometry of the Triangular Pattern

To trace an **equilateral triangle**, the turtle alternates between two sequential motions:

1. **Move straight:** Travel forward with a constant linear velocity for duration $T_{\text{line}}$.
2. **Turn in place:** Rotate about its Z-axis by an exterior angle of **120°** ($\frac{2\pi}{3} \approx 2.0944\text{ rad}$) for duration $T_{\text{turn}}$.
3. **Repeat:** Complete this cycle 3 times to form 3 equal sides and return to the starting heading.

```text
               /\
              /  \
      Side 1 /    \ Side 2
            /  /\  \
           /  120°  \
          /__________\
             Side 3
```

### 4.2 Full Source Code of `triangle_mover.py`

Create a new file for the publisher node:

```bash
touch ~/ros2_ws/src/turtle_controller/turtle_controller/triangle_mover.py
```

Open the file and paste the following state-machine implementation:

```python
#!/usr/bin/env python3
"""
Triangular Trajectory Publisher Node for Turtlesim
Author: AIROS - ESPOL
Description: Publishes Twist commands to /turtle1/cmd_vel to navigate the turtle
             in an equilateral triangular pattern.
"""

import math
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist


class TriangleMover(Node):
    def __init__(self):
        super().__init__('triangle_mover')

        # 1. Create velocity publisher to /turtle1/cmd_vel
        self.publisher_ = self.create_publisher(Twist, '/turtle1/cmd_vel', 10)

        # 2. Timer setup: runs at 10 Hz (every 0.1 seconds)
        self.timer_period = 0.1
        self.timer = self.create_timer(self.timer_period, self.timer_callback)

        # Motion parameters
        self.linear_speed = 2.0   # m/s
        self.line_duration = 2.0  # seconds moving straight (distance = 4.0 m)

        # Exterior angle turn: 120° = (2 * pi / 3) rad = 2.0944 rad
        self.turn_duration = 2.0  # seconds turning
        self.angular_speed = (2.0 * math.pi / 3.0) / self.turn_duration  # rad/s

        # State tracking: 'FORWARD' (straight), 'TURN' (rotation), 'FINISHED' (stop)
        self.state = 'FORWARD'
        self.state_time = 0.0
        self.sides_completed = 0
        self.total_sides = 3

        self.get_logger().info('Node [triangle_mover] started. Tracing triangular trajectory...')

    def timer_callback(self):
        msg = Twist()

        if self.state == 'FORWARD':
            # Drive forward
            msg.linear.x = self.linear_speed
            msg.angular.z = 0.0
            self.state_time += self.timer_period

            # Check if current side is finished
            if self.state_time >= self.line_duration:
                self.state = 'TURN'
                self.state_time = 0.0
                self.get_logger().info(f'Side {self.sides_completed + 1} finished. Turning 120 degrees...')

        elif self.state == 'TURN':
            # Rotate in place
            msg.linear.x = 0.0
            msg.angular.z = self.angular_speed
            self.state_time += self.timer_period

            # Check if 120-degree turn is complete
            if self.state_time >= self.turn_duration:
                self.sides_completed += 1
                self.state_time = 0.0

                if self.sides_completed < self.total_sides:
                    self.state = 'FORWARD'
                    self.get_logger().info(f'Starting side {self.sides_completed + 1}...')
                else:
                    self.state = 'FINISHED'
                    self.get_logger().info('Triangle trajectory completed successfully!')

        elif self.state == 'FINISHED':
            # Bring robot to a full stop
            msg.linear.x = 0.0
            msg.angular.z = 0.0
            self.publisher_.publish(msg)
            # Optional: To repeat indefinitely, uncomment the following two lines:
            # self.sides_completed = 0
            # self.state = 'FORWARD'
            return

        # Publish velocity command
        self.publisher_.publish(msg)


def main(args=None):
    rclpy.init(args=args)
    node = TriangleMover()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        node.get_logger().info('Stopping triangle_mover node...')
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

---

## 5. Package Configuration and Compilation

For ROS 2 to register and run these Python scripts as system executables, we must register them in the package configuration files.

### 5.1 Configure Dependencies in `package.xml`

Open `~/ros2_ws/src/turtle_controller/package.xml` and ensure that the following dependency tags are present:

```xml
  <depend>rclpy</depend>
  <depend>geometry_msgs</depend>
  <depend>turtlesim</depend>
```

*(These tags are automatically created if you included `--dependencies` during package creation).*

### 5.2 Register Console Entry Points in `setup.py`

Open `~/ros2_ws/src/turtle_controller/setup.py`.

Find the `entry_points` dictionary and declare both nodes inside `'console_scripts'`:

```python
    entry_points={
        'console_scripts': [
            'odom_listener = turtle_controller.odom_listener:main',
            'triangle_mover = turtle_controller.triangle_mover:main',
        ],
    },
```

> [!IMPORTANT]
> Verify comma placement and quotations carefully. Any syntax error in `setup.py` will cause the compilation process to fail.

### 5.3 Building with Colcon

Navigate to your workspace root and compile with `colcon build`:

```bash
cd ~/ros2_ws
colcon build --symlink-install
```

The `--symlink-install` option creates symbolic links directly to your Python source scripts. This ensures that future code edits in `.py` files take effect immediately without requiring another rebuild.

The compilation summary should finish cleanly:

```text
Starting >>> turtle_controller
Finished <<< turtle_controller [1.2s]

Summary: 1 package finished [1.4s]
```

### 5.4 Sourcing the Local Workspace

To make the newly compiled nodes accessible in your terminal session:

```bash
source ~/ros2_ws/install/setup.bash
```

Confirm that ROS 2 detects your new executables:

```bash
ros2 pkg executables turtle_controller
```

Expected output:

```text
turtle_controller odom_listener
turtle_controller triangle_mover
```

---

## 6. Execution, Testing, and System Validation

To coordinate all processes, open **three separate terminals**.

### 6.1 Recommended Execution Order

#### Terminal 1: Launch the Simulator

```bash
source /opt/ros/humble/setup.bash
ros2 run turtlesim turtlesim_node
```

> The Turtlesim window will appear with the turtle at center coordinates.

---

#### Terminal 2: Run the Odometry Subscriber Node (`odom_listener`)

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run turtle_controller odom_listener
```

> The console will immediately start streaming the current rest pose of the turtle.

---

#### Terminal 3: Run the Trajectory Publisher Node (`triangle_mover`)

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run turtle_controller triangle_mover
```

> [!NOTE]
> In the Turtlesim window, observe the turtle drawing the 3 sides of the triangle. Simultaneously, **Terminal 2** will log updating coordinates $(X, Y, \theta)$ in real time.

> [!NOTE]
> 📷 **Required Screenshot 1:** Turtlesim window showing the completed triangular path drawn on canvas.

![Screenshot: Triangular Pattern in Turtlesim](assets/class2_triangle_path.png)

> [!NOTE]
> 📷 **Required Screenshot 2:** Console of `odom_listener` displaying live streaming pose telemetry.

![Screenshot: Odometry Subscriber Output](assets/class2_odom_console.png)

---

### 6.2 Verifying Active Nodes and Topics

Open a **Terminal 4** to verify inter-node communication integrity:

```bash
source /opt/ros/humble/setup.bash

# Verify that all 3 nodes are active
ros2 node list
```

Expected output:

```text
/odom_listener
/triangle_mover
/turtlesim
```

List active topics:

```bash
ros2 topic list
```

Expected output:

```text
/parameter_events
/rosout
/turtle1/cmd_vel
/turtle1/color_sensor
/turtle1/pose
```

> [!NOTE]
> 📷 **Required Screenshot 3:** Terminal output showing the results of `ros2 node list` and `ros2 topic list`.

![Screenshot: Node and Topic Verification](assets/class2_verification_topics.png)

---

### 6.3 Visualizing the Node Graph with `rqt_graph`

To verify active publish/subscribe connections visually:

```bash
ros2 run rqt_graph rqt_graph
```

An interactive window will pop up showing the communication edges between `/triangle_mover`, `/turtlesim`, and `/odom_listener`.

> [!NOTE]
> 📷 **Required Screenshot 4:** The computation graph displayed by `rqt_graph`.

![Screenshot: RQt Graph Visualization](assets/class2_rqt_graph.png)

---

## 7. Deliverables and Grading Rubric

### Required Deliverables

1. **Two Source Code Files:**
   * `odom_listener.py`
   * `triangle_mover.py`
2. **One Online YouTube Video:**
   * A link to a clear video demonstrating the turtle successfully completing the triangular trajectory on screen alongside the running terminals.

---

### Grading Rubric (Total: 10 Points)

| Assessment Criteria | Description | Points |
| :--- | :--- | :---: |
| **Subscriber Node (`odom_listener.py`)** | Correctly implemented subscriber to `/turtle1/pose` logging clean pose data to console | **3.0 pts** |
| **Publisher Node (`triangle_mover.py`)** | Robust publisher node generating clean triangular motion via `/turtle1/cmd_vel` | **3.0 pts** |
| **Effective Communication & Organization** | Clean package layout, successful compilation, and verified graph topology via `ros2 topic list` / `rqt_graph` | **2.0 pts** |
| **Demonstration Video on YouTube** | Clear video demonstrating the complete system running without errors | **2.0 pts** |
| **Total** | **Maximum Workshop Score** | **10.0 pts** |

---

# 🤝 Contributing and Contact
Contributions and suggestions are always welcome! If you have recommendations to improve this workshop, please feel free to fork the repository and submit a pull request.

✉️ **Teaching Team & Developers:**
1. Maykoll Vanegas - [LinkedIn](https://www.linkedin.com/in/maykoll-steven-vanegas-silva-baa225219/)
2. Hector La Mota - [LinkedIn](https://www.linkedin.com/in/h%C3%A9ctor-la-mota-5a9b7a2a8/)
3. Carolina Sánchez - [LinkedIn](https://www.linkedin.com/in/carolinas%C3%A1nchezcevallos/)

**AIROS - ESPOL** 🇪🇨
