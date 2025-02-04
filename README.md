# tractor_sim

![Build - noetic](https://img.shields.io/:Build-noetic-yellowgreen.svg)
[![license - Apache-2.0](https://img.shields.io/:license-Apache2.0-yellowgreen.svg)](https://opensource.org/licenses/Apache-2.0)

Package for simulating the Autonomous Tractor. For further project information, see the GRAVL main repository wiki: ([link](https://github.com/olinrobotics/gravl/wiki))

## Setup
Dependencies:
+ [GRAVL](https://github.com/olinrobotics/gravl) - for teleop and testing code
+ Follow setup instructions on the GRAVL wiki if you haven't already ([reference](https://github.com/olinrobotics/gravl/wiki))
+ [tractor_sim_packages](https://github.com/olinrobotics/tractor_sim_packages) - packages for simulated localization and odometry
+ [velodyne_simulator](https://bitbucket.org/DataspeedInc/velodyne_simulator/src/master/) - packages for simulated 3D lidar

Installation:
```bash
cd ~/catkin_ws/src
git clone https://github.com/SNU-SF4/tractor_sim
git clone https://github.com/SNU-SF4/gravl.git
git clone https://github.com/SNU-SF4/state_controller.git

# install essential packages
rosdep install --from-paths src --ignore-src -r -y --os=ubuntu:focal
sudo apt install ros-noetic-ackermann-msgs ros-noetic-mavros-msgs ros-noetic-gps-common

# install velodyne related packages
sudo apt install ros-noetic-velodyne-description ros-noetic-velodyne-gazebo-plugins ros-noetic-velodyne-simulator

cd ~/catkin_ws/
catkin build
```

To use the models included in this repo, copy the contents of the folder to `~/.gazebo/models`
```bash
cd ~/catkin_ws/src/tractor_sim/tractor_sim_gazebo/models
cp -R . ~/.gazebo/models
```

## Usage
To run simulation:
+ `roslaunch tractor_sim_gazebo bringup.launch`

## ROS Topics
+ `/cmd_vel` - twist command for driving the tractor
+ `/cmd_hitch` - pose command for setting height of box blade ([reference](https://github.com/olinrobotics/state_controller/blob/master/README.md))

#### Sensor Topics
+ `/camera/image_raw` - camera feed
+ `/gps/fix` - rtk gps data
+ `/imu/data` - imu data
+ `/scan` - downward facing LiDar data

#### Other Topics
+ `/odometry/filtered` - tractor odometry
+ `/laser_pointcloud` - generated pointcloud

## Troubleshooting
If using the main launch file doesn't work for you, you can just run individual launch files to suit your needs
To run simulation without external dependencies:
+ `roslaunch tractor_sim_gazebo bringup_min.launch`

To run rviz:
+  `roslaunch tractor_sim_description tractor_sim_rviz.launch`

To test code:
+ `rosrun gravl yourscript.py` (ex: `rosrun gravl LidarFollower.py`)

To run simulation (without teleop, hitch controls, localization, or any extra code running)
+ `roslaunch tractor_sim_gazebo tractor_sim.launch`

To run teleop (following https://github.com/olinrobotics/gravl/wiki/Kubo:-Overview):
+ `roslaunch gravl teleop.launch`
+ `roslaunch gravl mainstate.launch`

To setup localization:
+ `roslaunch gravl localization.launch`

To setup hitch commands:
+ `roslaunch tractor_sim_gazebo hitch.launch`


### Parameters
+ **tractor_delay** - either 0 or 1. If set to 1, then tractor will react instantaneously to commands. If set to 1, the tractor will simulate the delayed response time of the real tractor (Default: 1)
+ **max_acceleration** - maximum acceleration for the tractor Note: only works when tractor_delay is 0 (Default: 0.4)
+ **max_steering_angle_velocity** - maximum steering velocity Note: only works when tractor_delay is set to 0 (Default: 1)

## Known Issues
+ Running the launch file will throw the warning
```
[WARN] [1590601461.920958, 0.000000]: Controller Spawner couldn't find the expected controller_manager ROS interface.
```
if the controller spawner loads before gazebo loads, which is likely. To get around this, in the launch file the controller spawner's node has `respawn="true"`. Ideally, this should wait for gazebo to finish loading before starting the node so that the warning doesn't pop up, but this works for now.
