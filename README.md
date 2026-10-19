# RL manipulation

Isaac Sim 5.1 workspace for developing robot manipulation tasks. The current
scene contains a Franka Panda arm, a table, and a cube, with a ROS 2 OmniGraph
for joint commands and joint-state feedback. A custom extension loads scenes
from the command line.

Run tasks with `pixi run <task>`:

| Task | Description |
| --- | --- |
| `isaacsim` | Launch Isaac Sim with the project's extension directory available. |
| `isaacsim-default-scene` | Launch Isaac Sim and automatically open the default manipulation scene. |
| `ros-build` | Build the ROS workspace and its required source dependencies using system Jazzy. |
| `ros-run-moveit` | Build the ROS workspace, then start MoveIt, RViz, ros2_control, and the Panda arm/gripper controllers. |

## ROS workspace

All ROS packages live under `ros_ws/src`. Upstream repositories are pinned Git
submodules; `rl_manipulation_bringup` contains only project-specific integration.

| Reused package | Source | Purpose |
| --- | --- | --- |
| `moveit_resources_panda_description` | `moveit_resources` submodule | Panda URDF, meshes, joint limits, and finger mimic model. |
| `moveit_resources_panda_moveit_config` | `moveit_resources` submodule | SRDF, kinematics, OMPL planning settings, ros2_control descriptions, MoveIt controller mapping, and RViz configuration. |
| `topic_based_ros2_control` | Submodule | Hardware plugin that exchanges joint commands and feedback with Isaac Sim over ROS topics. |
| `controller_manager`, `hardware_interface`, `controller_interface` | System Jazzy | Load hardware and run controllers. |
| `joint_trajectory_controller` | System Jazzy | Execute arm trajectories through `/panda_arm_controller/follow_joint_trajectory`. |
| `gripper_controllers` | System Jazzy | Position-controlled gripper action at `/panda_hand_controller/gripper_cmd`. |
| `joint_state_broadcaster` | System Jazzy | Publish controller-side state on `/controller_joint_states`. |
| `moveit_configs_utils`, `moveit_ros_move_group` | System Jazzy | Assemble configuration and provide motion planning/execution. |
| `moveit_planners_ompl`, `moveit_kinematics`, `moveit_simple_controller_manager` | System Jazzy | Planning, inverse kinematics, and action-based controller integration. |
| `robot_state_publisher`, `tf2_ros` | System Jazzy | Publish robot transforms and the scene's Panda base placement. |
| `xacro`, `launch_ros`, `rviz2`, `moveit_ros_visualization` | System Jazzy | Expand robot descriptions, launch processes, and provide interactive planning UI. |

No separate Pixi environment is required. Isaac Sim uses Pixi's Python 3.11;
ROS builds and nodes use `/opt/ros/jazzy` and system Python 3.12. The Pixi ROS tasks
remove simulator-specific paths before sourcing Jazzy and the workspace.
Install system Jazzy, colcon, MoveIt, ros2_control, and ros2_controllers first.

Initialize dependencies and build:

```bash
git submodule update --init --recursive
pixi run ros-build
```

If system dependencies are missing, install them with rosdep (from the repository root):

```bash
rosdep install --rosdistro jazzy --ignore-src -y --from-paths \
  ros_ws/src/rl_manipulation_bringup \
  ros_ws/src/topic_based_ros2_control \
  ros_ws/src/moveit_resources/panda_description \
  ros_ws/src/moveit_resources/panda_moveit_config
```

Launch the simulator and press **Play**, then launch controllers in another terminal:

```bash
pixi run isaacsim-default-scene
# In another terminal:
pixi run ros-run-moveit
```

Use `pixi run ros-run-moveit rviz:=false` to omit RViz. The ROS nodes use simulation
time from `/clock`. Isaac Sim publishes `/isaac/joint_states` and receives
`/isaac/joint_command`; MoveIt, RViz, and robot_state_publisher consume that feedback.
Controller-broadcast states use a separate topic to prevent a
feedback loop. The second finger follows the first through the topic hardware's
mimic configuration.

From a system ROS terminal, inspect the running controllers:

```bash
source /opt/ros/jazzy/setup.bash
source ros_ws/install/setup.bash
ros2 control list_controllers
ros2 action list
```

The static `world` → `panda_link0` transform matches the current scene's base
position `(-0.5, 0, 1)`; update the launch file if you move the robot. The table
and cube are not yet added to MoveIt's collision scene, so inspect planned paths
before executing them. This integration targets simulation, not physical Franka hardware.
