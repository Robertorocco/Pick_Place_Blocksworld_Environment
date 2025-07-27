# Pick and Place in a Blocksworld Environment using HTN Planning and MoveIt

https://private-user-images.githubusercontent.com/182740140/467962562-0d6c1e1e-ce1d-46a5-9862-abccb1980fe2.mp4?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTI4MzIxMDMsIm5iZiI6MTc1MjgzMTgwMywicGF0aCI6Ii8xODI3NDAxNDAvNDY3OTYyNTYyLTBkNmMxZTFlLWNlMWQtNDZhNS05ODYyLWFiY2NiMTk4MGZlMi5tcDQ_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzE4JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcxOFQwOTQzMjNaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT04NDE2NmU0MWQxNDBkNTM3ZjdkMmMzNmU1ZGZiNjlkODA4YTJmZmFlZDQzYjFkZGM1ZDg0N2ExMWVjYmRhNjBiJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.PxCx_JmYlHH0dlaoo6u91ZOYYkAP6NSsMMZrTjmxWW8

---

## 🤖 Project Overview


The goal of this project is to enable a **Universal Robots UR5e** robotic manipulator to autonomously perform pick & place tasks in a blocksworld-like environment. The system takes an initial configuration of blocks within a **Gazebo** world and a specified symbolic goal state. It then computes and executes the sequence of physical actions required to move the blocks to their target locations, including stacking them on top of one another.

The entire execution pipeline is built on **ROS 2** and integrates high-level symbolic planning with low-level motion planning and execution.

## 📄 Documentation

Full documentation of the project is aviable in PDF format

**[PDF Documentation](./Documentation.pdf)**

## Core Technologies

* **Robot**: Universal Robots UR5e
* **Simulation**: Gazebo for blocksworld building
* **Planning Paradigms**: Hierarchical Task Network (HTN), MoveIt Task Constructor (MTC), Replanning strategy

## ✨ Key Features & Pipeline

The system operates through a series of distinct, high-level steps:

### 1. Symbolic State Generation

* **Gazebo to ROS Bridge**: A custom Gazebo plugin (`PoseRelayPlugin`) was developed to read the geometric poses of all blocks in the simulation. This plugin publishes the block positions and orientations on a ROS 2 topic (`/object_pose`).
* **Geometric to Symbolic Conversion**: An algorithm converts the continuous 3D coordinates of the blocks into a discrete, symbolic state. It uses heuristics based on the blocks' known height to determine predicates like `on(A, B)` or `on(A, table)`. This removes the need to manually specify the initial state for the planner.

### 2. Hierarchical Task Network (HTN) Planning

* **High-Level Task Decomposition**: An HTN planner is used to compute the sequence of actions. The planner decomposes abstract tasks (e.g., `move_blocks`) into simpler, primitive actions (e.g., `pickup`, `unstack`, `stack`) using predefined methods.
* **Plan Publication**: Once decomposition is complete, the planner generates a linear sequence of primitive actions. This plan is serialized and published to a ROS 2 topic (`/pyhop_plan`) for the motion controller.

### 3. Motion Execution with MoveIt Task Constructor (MTC)

* **Symbolic-to-Motion Mapping**: A dedicated ROS 2 node (`mtc_node.cpp`) subscribes to the plan topic and partitions the actions into sequential pick-and-place pairs.
* **Task Execution**: Each pair is executed as a composite task within the **MoveIt Task Constructor** (MTC). MTC handles the complex sequence of sub-tasks, including moving to grasp, lifting, attaching the object, and placing it at a target location.
* **Collision Management**: The system dynamically updates MoveIt's `PlanningScene` with block locations from Gazebo and uses an `AllowedCollisionMatrix` to prevent false-positive collisions during planning.

### 4. Gripper Integration: Virtual and Realistic

* **Gripper-less "Fake Hand"**: The initial implementation works without a physical gripper. A "fake hand" (a virtual link) was created to satisfy MoveIt's requirements. Grasping is simulated by virtually attaching the object's frame to the fake hand's frame.
* **Realistic Gripper Integration**: The project was extended to integrate a **Robotiq 2-Finger 85 Gripper**. This involved re-enabling gravity and using the gripper's physical dynamics, which provides more stable placements at the cost of higher computational load in Gazebo.

### 5. Replanning for Failure Recovery

* **State Monitoring**: A manager node supervises the execution to handle failures.
* **Failure Detection**: Before executing each task, the system checks if the *actual* symbolic state (from Gazebo) matches the *expected* state from the original plan.
* **Recovery**: If a mismatch is detected, the current task is terminated. The planner is re-invoked to compute a new plan from the current state, allowing the system to recover from errors and adapt.
  
## :clipboard: Requirements
This package **requires MoveIt2 to be pre-installed** on your system. 
:point_right: Follow the [official MoveIt2 installation guide](https://moveit.picknik.ai/humble/doc/tutorials/getting_started/getting_started.html) to set it up properly.
> :bangbang: This package will **not work** without a working MoveIt installation.

# :hammer: Build
1. Clone the repository in the `src` folder of your ROS 2 workspace.  If you want to only clone the content files without creating the repo folder, use
```
git clone https://github.com/Robertorocco/Pick_Place_Blocksworld_Environment.git
```
2. Build the workspace
```
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
```
3. Source the setup files
```
source ~/ros2_ws/install/setup.bash
```

# :whale: Alternative Setup with Docker (No Local MoveIt2 Installation Required)
If you'd prefer not to install MoveIt2 manually, you can run this project inside a Docker container with all necessary dependencies pre-configured.

## :gear: Prerequisites
Docker installed on your system. You can get it from the [official Docker website](https://docs.docker.com/get-docker/). 

### :bulb: Notes: 
Superuser permissions are requested. If you want to avoid invoking sudo you can add the currently logged-in user to the docker group


### \:hammer\_and\_wrench: Using the Prebuilt Docker Image

1. Clone the repository containing Dockerfile and scripts to build and tun the image into a local folder:

```
git clone https://github.com/Robertorocco/MoveIt-Docker.git
cd MoveIt-Docker
```
2. Inside the terminal of the cloned repo make all the bash files executable:
```sh
chmod +x *.sh
```

3. Build the image: 

```
./docker_build_image.sh <IMAGE_NAME>
```
where <IMAGE_NAME> is a name for the image you want to build. The build will take at least 1 hour.
If you have up to 16Gb of RAM you and want to speed building of the image you can substitute inside the Dockerfile the following line:
```
MAKEFLAGS="-j4 -l1" colcon build --executor sequential --cmake-args -DCMAKE_BUILD_TYPE=Release
```
with:  
```
 colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
```

4. Run the container:

```
./docker_run_container.sh <IMAGE_NAME> <CONTAINER_NAME> <FOLDER_NAME>
```
where <IMAGE_NAME> is the name of the image you have just built, while <CONTAINER_NAME> is a name for the container hosting the image. <FOLDER_NAME> is the name of the local folder you want to mount inside the container ROS2 ws. In order to run the application it must correspond to the name of the folder where **Pick_Place_Blocksworld_Environment** has been cloned.


5. Once inside the container, build and source the workspace:

```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

### :bulb: Notes

To connect to the container from another terminal, run
```
./docker_connect.sh 
```
and select the container you want to connect to.


# :rocket: Launching the Application

## :white_check_mark: Usage
1. Run the RVIZ environment and Gazebo simulation through the launch file:
```
ros2 launch ur_description setup.launch.py 
```
Gazebo simulation starts in the "PAUSED" state. To make it play click on the bottom left "Play" button. You have 5 seconds (a default timeout value difficult to change) to start the Gazebo program, connect the controller and play the simulation.

2. Open a second terminal and run the MoveIt Task Constructor and Move It planner using the following executable:
```
ros2 run mtc_package manager
```
3. Once the console shows the message from `mtc_node` saying **"Parsing the plan..."**, in another terminal you can launch the **HTN Planner**, which will generate the sequence of tasks to be performed, with the following:

```
ros2 launch blocksword_planner htn_launch.py 
``` 

## :warning: Warning
During the launch of the Gazebo environment you could face a list of errors similar to:
```diff
- [GUI] [Err] [SystemPaths.cc:378] Unable to find file with URI [model://robotiq_description/meshes/visual/2f_85/robotiq_base.dae]

```
This is caused by the unsuccessful update of the GZ_SIM_RESOURCE_PATH environment variable. To fix the error use after sourcing the setup files:
```
export GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:~/ros2_ws/src/ros2_robotiq_gripper/
```

