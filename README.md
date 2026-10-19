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
