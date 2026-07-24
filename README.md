# 🤖 Phase 2: The Software Brain (ROS 2 Core Architecture)

Welcome to Phase 2 of the **IEEE RAS: From Simulation to Reality** workshop! In this phase, we transition from basic Linux setup into the core of the ROS 2 framework. 

This repository guide walks you through creating a hybrid `ament_cmake` package, writing Python publisher/subscriber nodes, writing launch files, and debugging your robot's software.

---

## 🧠 Quick Concepts Dictionary
*   **Node:** A mini-program that performs a specific task (e.g., controlling a motor).
*   **Topic:** A named "radio channel" where data is sent (e.g., `/chatter_topic`).
*   **Publisher:** A node that broadcasts messages to a topic.
*   **Subscriber:** A node that tunes into a topic to read messages.
*   **Workspace:** The main folder (`ros2_ws`) where all your ROS 2 projects live.
*   **Colcon:** The tool we use to build/compile our workspace.

---

## 🛠️ Step 1: Create Workspace & Package
Open a new terminal (`Ctrl + Alt + T`) and run these commands to set up your folder structure. We are using an `ament_cmake` package, which allows us to mix C++ and Python code.

```bash
# 1. Create the workspace and go into the 'src' folder
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# 2. Create the package
ros2 pkg create --build-type ament_cmake my_first_package --dependencies rclpy std_msgs

# 3. Go into the package and create directories for our scripts and launch files
cd my_first_package
mkdir src launch urdf

🗣️ Step 2: Write the Publisher (Talker)

Create a new file named talker.py inside the src folder.

#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('my_talker_node')
        self.publisher_ = self.create_publisher(String, 'chatter_topic', 10)
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello from Robot!'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    node = MinimalPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

👂 Step 3: Write the Subscriber (Listener)

Create a new file named listener.py inside the src folder.

#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalSubscriber(Node):
    def __init__(self):
        super().__init__('my_listener_node')
        self.subscription = self.create_subscription(
            String, 'chatter_topic', self.listener_callback, 10)

    def listener_callback(self, msg):
        self.get_logger().info(f'I heard: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    node = MinimalSubscriber()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

⚠️ CRITICAL: Make your scripts executable!

Linux needs permission to run these files. If you are on Windows/WSL, you also
need to fix line endings. Run these commands:

# Give execution permissions
chmod +x ~/ros2_ws/src/my_first_package/src/*.py

# Fix Windows CRLF line endings to Linux LF line endings (Fixes the \r error)
sudo apt update && sudo apt install dos2unix
dos2unix ~/ros2_ws/src/my_first_package/src/*.py

⚙️ Step 4: Configure CMakeLists.txt

Open CMakeLists.txt (located in ~/ros2_ws/src/my_first_package/). Scroll down to
just above the ament_package() line at the very bottom, and add this:

# Tell CMake to install our Python src as executables
install(PROGRAMS
  src/talker.py
  src/listener.py
  DESTINATION lib/${PROJECT_NAME}
)

# Tell CMake to install our launch and urdf folders
install(DIRECTORY
  launch
  urdf
  DESTINATION share/${PROJECT_NAME}
)

🚀 Step 5: Write the Launch File

Why open multiple terminals when you can open one? Create a new file named
chatter.launch.py inside the launch folder.

from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_first_package',
            executable='talker.py',
            name='talker'
        ),
        Node(
            package='my_first_package',
            executable='listener.py',
            name='listener'
        )
    ])

🏗️ Step 6: Build & Run!

Whenever you change CMakeLists.txt, package.xml, or add a new launch file, you
must rebuild your workspace.

# 1. Go to the root of your workspace
cd ~/ros2_ws

# 2. Build the packages
colcon build --symlink-install

# 3. Source the workspace (MUST do this in every new terminal!)
source install/setup.bash

# 4. Run the launch file!
ros2 launch my_first_package chatter.launch.py

🔍 Step 7: Introspection (Debugging)

Leave your launch file running, open a new terminal, run source
~/ros2_ws/install/setup.bash, and try these commands:

  - ros2 node list (Lists all running nodes)
  - ros2 topic list (Lists all active topics)
  - ros2 topic info /chatter_topic (Shows message type and publisher/subscriber
    count)
  - ros2 topic echo /chatter_topic (Prints the actual data streaming over the
    topic)

🏆 THE CHALLENGE: URDF Arm Visualization

Your Mission: Use a Launch file to display a 3D digital robot arm in RViz!

1.  Download the provided robot_arm.urdf file and place it in your
    ~/ros2_ws/src/my_first_package/urdf/ folder.
2.  Rebuild your workspace (colcon build --symlink-install).
3.  Create a new launch file called arm_visualize.launch.py.

💡 Launch File Hints: You need to launch three specific nodes. Here is the
framework to help you!
# 1. Create the workspace and go into the 'src' folder
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

# 2. Create the package
ros2 pkg create --build-type ament_cmake my_first_package --dependencies rclpy std_msgs

# 3. Go into the package and create directories for our scripts and launch files
cd my_first_package
mkdir src launch urdf

🗣️ Step 2: Write the Publisher (Talker)

Create a new file named talker.py inside the src folder.

#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('my_talker_node')
        self.publisher_ = self.create_publisher(String, 'chatter_topic', 10)
        self.timer = self.create_timer(1.0, self.timer_callback)

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello from Robot!'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    node = MinimalPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

👂 Step 3: Write the Subscriber (Listener)

Create a new file named listener.py inside the src folder.

#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalSubscriber(Node):
    def __init__(self):
        super().__init__('my_listener_node')
        self.subscription = self.create_subscription(
            String, 'chatter_topic', self.listener_callback, 10)

    def listener_callback(self, msg):
        self.get_logger().info(f'I heard: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    node = MinimalSubscriber()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()

⚠️ CRITICAL: Make your scripts executable!

Linux needs permission to run these files. If you are on Windows/WSL, you also
need to fix line endings. Run these commands:

# Give execution permissions
chmod +x ~/ros2_ws/src/my_first_package/src/*.py

# Fix Windows CRLF line endings to Linux LF line endings (Fixes the \r error)
sudo apt update && sudo apt install dos2unix
dos2unix ~/ros2_ws/src/my_first_package/src/*.py

⚙️ Step 4: Configure CMakeLists.txt

Open CMakeLists.txt (located in ~/ros2_ws/src/my_first_package/). Scroll down to
just above the ament_package() line at the very bottom, and add this:

# Tell CMake to install our Python src as executables
install(PROGRAMS
  src/talker.py
  src/listener.py
  DESTINATION lib/${PROJECT_NAME}
)

# Tell CMake to install our launch and urdf folders
install(DIRECTORY
  launch
  urdf
  DESTINATION share/${PROJECT_NAME}
)

🚀 Step 5: Write the Launch File

Why open multiple terminals when you can open one? Create a new file named
chatter.launch.py inside the launch folder.

from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_first_package',
            executable='talker.py',
            name='talker'
        ),
        Node(
            package='my_first_package',
            executable='listener.py',
            name='listener'
        )
    ])

🏗️ Step 6: Build & Run!

Whenever you change CMakeLists.txt, package.xml, or add a new launch file, you
must rebuild your workspace.

# 1. Go to the root of your workspace
cd ~/ros2_ws

# 2. Build the packages
colcon build --symlink-install

# 3. Source the workspace (MUST do this in every new terminal!)
source install/setup.bash

# 4. Run the launch file!
ros2 launch my_first_package chatter.launch.py

🔍 Step 7: Introspection (Debugging)

Leave your launch file running, open a new terminal, run source
~/ros2_ws/install/setup.bash, and try these commands:

  - ros2 node list (Lists all running nodes)
  - ros2 topic list (Lists all active topics)
  - ros2 topic info /chatter_topic (Shows message type and publisher/subscriber
    count)
  - ros2 topic echo /chatter_topic (Prints the actual data streaming over the
    topic)

🏆 THE CHALLENGE: URDF Arm Visualization

Your Mission: Use a Launch file to display a 3D digital robot arm in RViz!

1.  Download the provided robot_arm.urdf file and place it in your
    ~/ros2_ws/src/my_first_package/urdf/ folder.
2.  Rebuild your workspace (colcon build --symlink-install).
3.  Create a new launch file called arm_visualize.launch.py.

💡 Launch File Hints: You need to launch three specific nodes. Here is the
framework to help you!
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node
import xacro

def generate_launch_description():
    # 1. Find the path to the URDF file
    xacro_file = os.path.join(get_package_share_directory('my_first_package'),'urdf', 'robot_arm.urdf.xacro') 
    doc = xacro.process_file(xacro_file)
    robot_description = {'robot_description': doc.toxml()}
    

    return LaunchDescription([
        # Node 1: Robot State Publisher (Broadcasts the robot's links to ROS)
        Node(
            package='robot_state_publisher',
            executable='robot_state_publisher',
            parameters= [{'robot_description': robot_description}]
        ),
        
        # Node 2: Joint State Publisher GUI (Gives you sliders to move the joints!)
        Node(
            package='joint_state_publisher_gui',
            executable='joint_state_publisher_gui'
        ),

        # Node 3: RViz2 (The 3D Visualization tool)
        Node(
            package='rviz2',
            executable='rviz2'
        )
    ])

Created for the IEEE RAS Robotics Workshop Series.



