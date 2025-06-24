# 1- Installing ROS

If you haven't already installed ROS instructions listed here:

For Ubuntu 20.04 LTS, [ROS-noetic](http://wiki.ros.org/noetic/Installation/Ubuntu) is official supported version

## Set up your sources.list
* `sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros-noetic.list'`

## Set up your keys
* `sudo apt update` 
* `sudo apt install curl -y`
* `curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -`

## Update your package index
* `sudo apt update`

## Install ROS Noetic (Desktop-Full version)
* `sudo apt install ros-noetic-desktop-full -y`

## Install rosdep
* `sudo apt install python3-rosdep -y`

## Install rosdep
* `sudo rosdep init`
* `rosdep update`

## Environment Setup
`echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc`

`source ~/.bashrc`

## Install rosinstall tools (optional)
* `sudo apt install python3-rosinstall python3-rosinstall-generator python3-wstool build-essential -y`

## Test ROS Installation
* `roscore`

In another terminal:
* `rosrun turtlesim turtlesim_node`


# 2-Setting up your workspace
We suggest use python catkin tools to build ROS projects. Make sure you have the python catkin tools installed:
  * `sudo apt install python3-catkin-tools`
  
For more information you can see: http://catkin-tools.readthedocs.io/en/latest/installing.html

Now, create a workspace folder:
  * `$ mkdir -p ~/catkin_ws/src`  
  * `$ cd ~/catkin_ws/`
  
We suggest you use a fresh work space for this simulator to prevent possible cross-contamination. Instructions on using multiple work spaces can be found here:
  * https://roboticsbackend.com/ros-multiple-catkin-workspaces/

Then, just build an empty project by:
  * `$ catkin build`

This is just to see you can build projects using python catkin tools. NOTE: If you used `catkin_make` before you cannot build, first do `catkin_make clean`, then `catkin build`.

# 3-Setup symlinks
Now `cd` into your ROS ws folder, and make the needed symlinks between the simulator and the work space

* `cd ~/catkin_ws/`
* `ln -s ~/AICode/agri-fly/AIFS_ROS/hiperlab_common/ src/`
* `ln -s ~/AICode/agri-fly/AIFS_ROS/hiperlab_components/ src/`
* `ln -s ~/AICode/agri-fly/AIFS_ROS/hiperlab_rostools/ src/`
* `ln -s ~/AICode/agri-fly/AIFS_ROS/hiperlab_hardware/ src/`
* `ln -s ~/AICode/agri-fly/AIFS_ROS/makeProjects_AIFS.sh `

Then we need to link the source files for common & components:
* `ln -s ~/AICode/agri-fly/Common/ src/hiperlab_common/src`
* `ln -s ~/AICode/agri-fly/Components/ src/hiperlab_components/src`

# 4-Setup AirSim Root in buildfile
In `AIFS_ROS/hiperlab_rostools/CMakeLists.txt`,
Set the `AIRSIM_ROOT` to the folder you store airsim on your computer. 
i.e.:
* `set(AIRSIM_ROOT /home/manhhv/AICode/agri-fly/AIFS_AirSim)`


# 5-Build the workspace
Finally, in `~/catkin_ws` folder, run
* `sh makeProjects_AIFS.sh` 

Before continuing source your new `setup.*sh` file:
  * `source devel/setup.bash`

To make sure your workspace is properly overlayed by the setup script, make sure `ROS_PACKAGE_PATH` environment variable includes the directory you're in.
  * `$ echo $ROS_PACKAGE_PATH`

By echo you have to see something like:
`/home/manhhv/catkin_ws/src/hiperlab_common:/home/manhhv/catkin_ws/src/hiperlab_components:/home/manhhv/catkin_ws/src/hiperlab_rostools:/home/manhhv/catkin_ws/src/hiperlab_hardware:/opt/ros/kinetic/share`



--------------------------------------------------------------------------------------------
# Quick startup guide

# 1- Start the simulator in Unity
* Start Unity Hub, import the argricultral world via selecting the folder `AIFS_AirSim\Unity\UnityDemo`, and then hit the `OK` button.

* Click on the new project which showed up in the Unity Hub menu to open it in Unity.

* In the bottom pane, click on `Projects->Assets->Scenes`. Then, double-click on `SimModeSelector`.

* Hit the `Play` button to start.

* Select the scene you would like the drone to fly in. At this point you should see the drone droping and landing on the ground.

# 2- Run the ROS simulator
* Locate the launch file located at `hiperlab_rostools->launch->agrifly.launch`.
 
* Create a `trajectory.txt` file that contains your desired trajectory setpoints with the following format:
  ```
  x1,y1,z1
  x2,y2,z2
  x3,y3,z3
  ...
  ```
  Where `X` points forward, `Y` left, and `Z` up. For example, the file content should look like:
  ```
  0.0,0.0,4.0
  10.0,0.0,4.0
  15.0,0.0,4.0
  ...
  ```
  for the drone to fly a straight line trajectory at 4 meters.
  
 * Point to the trajctory file location in `agrifly.launch` by replacing `"/default/path/to/trajectory.txt"`.
 
 * Start the simulator by running `roslaunch hiperlab_rostools agrifly.launch`. You will be prompted to hit 's' button on the keyboard twice to start the flight controller. You should now see the drone flying through the scene following the defined trajectory.
 
 * To end the simulation, kill the ROS notes from the terminal before you stop the Unity simulation.

# 3- Record bag files
* Copy and paste `rosbag_record_airsim.sh` to places you want to record data at.
* Run 'rosbag_record_airsim.sh' to record ROS bag files.

# Running the simulation with seperate ROS nodes
Instead of using the launch file, you may also start each node seperately.

* Start ROS by running `roscore`
  
* Start the AirSim Bridge node by running `rosrun hiperlab_rostools air_sim_bridge`. The AirSim Bridge receives synthesized camera images from Unity and convert it to ROS compatible messages. Add 1 or 2 arguments of integer 0-10 (camera types specified in ImageCaptureBase.hpp) to specify which camera type(s) to be sent to ROS from Unity. Ex: `rosrun hiperlab_rostools air_sim_bridge 3 0` would return DepthVis (3) and Scene (0) to ros publishers depthImage and rgbImage respectively. 

* If your hardware is powerful enough/you experiment is not sensative to delay in synthesized graph generation, we suggest you use real-time ros simulator which is synced with a wall clock. Start the simulator node by running `rosrun hiperlab_rostools simulator 1` change `1` to other vehicle IDs if you are not running with the default vehicle.

* If your hardware is limited/you sense significant delay in image generation, we suggest you use simulation-time ros simulator which is synced with a simulation clock. Start the sync-simulator node by running `rosrun hiperlab_rostools sync_simulator 1` change `1` to other vehicle IDs if you are not running with the default vehicle.

* Start the RAPPIDS planner and controller node `rosrun hiperlab_rostools quad_rappids_planner_controller 1`. Again, change `1` to your vehicle IDs if you are running with a different vehicle.   

* Following the instruction given by the RAPPIDS node and hit 's' button on the keyboard multiple times to start the flight controller.

* To end the simulation, kill the ROS nodes (in the same order you ran them) first before you stop the Unity simulated world.


