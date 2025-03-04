# Experimental MoveIt 2 - Drake Integration

> [!NOTE]
> Experimental and will continue to have breaking changes until first release.

`moveit_drake` brings together the vertical ROS integration of the [MoveIt 2](https://moveit.ai/) motion planning framework, with the Mathematical Programming interface within [Drake](https://drake.mit.edu/).
This allows the user to setup motion planning as an optimization problem within ROS, with the rich specification of constraints and costs provided by `drake`.


## Features

- Exposes [`KinematicTrajectoryOptimization`](https://drake.mit.edu/doxygen_cxx/classdrake_1_1planning_1_1trajectory__optimization_1_1_kinematic_trajectory_optimization.html) implementation in `drake` as a motion planner.
- Exposes [`TOPPRA`](https://drake.mit.edu/doxygen_cxx/classdrake_1_1multibody_1_1_toppra.html) implementation in `drake` as a trajectory post-processing adapter.

## Docker Workflow (Preferred and tested)

### Requirements
Docker and Docker Compose - Follow the instructions [here](https://docs.docker.com/engine/install/ubuntu/).
Also, run the [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/).

### Steps
The following steps clone and build the base image that you will require to test/build/run/develop with the repo.

```bash
git clone https://github.com/moveit/moveit_drake.git
cd moveit_drake
docker compose build
```

This should give you an image with `drake` and `moveit2`.
Next, create a container with the following and create shell access.

```bash
docker compose up
docker compose exec -it moveit_drake bash
```

Follow [instructions](#build-moveit_drake) below to build `moveit_drake`


## Local Installation

### Install Drake

[Follow these instructions](https://drake.mit.edu/installation.html)

### Build `moveit_drake`

Follow the [MoveIt Source Build](https://moveit.ros.org/install-moveit2/source/) instructions to set up a ROS 2 workspace with MoveIt from source.

Open a command line and navigate to your workspace:

```bash
cd ${WORKSPACE}/src
```

Clone this repo to your workspace, including upstream dependencies:

```bash
git clone https://github.com/moveit/moveit_drake.git
vcs import < moveit_drake/moveit_drake.repos
rosdep install -r --from-paths . --ignore-src --rosdistro ${ROS_DISTRO} -y
```

Configure and build the workspace (this will take some time, as it builds MoveIt):

```bash
cd ${WORKSPACE}
colcon build --event-handlers desktop_notification- status- --cmake-args -DCMAKE_BUILD_TYPE=Release --parallel-workers 1
```


## Examples

The planning pipeline testbench compares `moveit_drake` planners with existing MoveIt planners such as OMPL and Pilz.

```bash
ros2 launch moveit_drake pipeline_testbench.launch.py
```

This interactive example shows constrained planning using the Drake KTOpt planner.

```bash
ros2 launch moveit_drake constrained_planning_demo.launch.py
```


## Configuration

### Use another robot

By default, Drake uses descriptions located within the [drake_models](https://github.com/RobotLocomotion/models) package. Modify the `drake_robot_description` parameter if you want to use a different description:
```yaml
drake_robot_description: "package://drake_models/pr2_description/urdf/pr2_simplified.urdf"
```

In case you want to use other robots, you can specify the global path to the description package using the `external_robot_description` parameter:
```yaml
external_robot_description: ["/home/user/xarm_ws/src/xarm_ros2/xarm_description"]
```
Drake will crawl down the directory tree starting at the given path. You can also specify multiple paths in the array.

If you already provide the transform `world->base_frame` you might also want to leave the parameter `base_frame` empty.


## Development

### Formatting

We use [pre-commit](https://moveit.ros.org/documentation/contributing/code/#pre-commit-formatting-checks) to format the code in this repo.

Within the container, you can run the following command to format the code:

```bash
cd src/moveit_drake
pre-commit run -a
```

### Some helper commands
To rebuild only the `moveit_drake` package:

```bash
rm -rf build/moveit_drake install/moveit_drake
colcon build --packages-select moveit_drake
```

## Known issues

### .stl support

Unfortunately, Drake does not support `.stl` files (11/28/2024, see [drake#19408](https://github.com/RobotLocomotion/drake/issues/19408)).
We're working around this by replacing the `.stl` files in the urdf string with `.obj` files in the plugin implementations.
Make sure that the moveit config you're using contains the relevant `.stl` files.
If it doesn't, take a look into the scripts/ directory.
We've provided a simple python script to add additional `.obj` files for given `.stl` files. Usage:

```
./scripts/convert_stl_to_obj.py /PATH/TO/YOUR/MESH/DIR
```
Don't forget to rebuild your description package so the `.obj` files are copied into the workspace's install directory.
