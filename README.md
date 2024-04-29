# ROS2 Interface for the AutomaticSceneGeneration UE4 plugin

## Description

This repo contains the base ROS2 interface for interacting with the [AutomaticSceneGeneration](https://github.com/tsender/AutomaticSceneGeneration) plugin for UE4.

This interface contains two ROS packages:
1. `auto_scene_gen_core`: Contains the main ROS nodes and various objects needed to interact with an AutoSceneGenWorker and AutoSceneGenVehicle in UE4.
2. `auto_scene_gen_msgs`: Contains the custom message and service definitions.

Even though the ROS code you write will depend on this repo, we think it's best that you simply add the two ament packages from this repo into your custom repo. One reason for this is because these repos are fairly lightweight. But more importantly, in case you choose to modify the message definitions or any part of the provided ROS nodes, it will be easier to compile and run your code if everything is in one location.

Below are some important documentation links explaining how to use this repo:
- [Message and Service Definitions](documentation/msg_and_srv_reference.md)
- [Creating and Running Scenarios](documentation/creating_scenarios.md)
- [Creating Vehicle Nodes](documentation/creating_vehicle_nodes.md)

For a full working example of using this interface with the AutomaticSceneGeneration plugin, please see our other repo [bacre_2D](https://github.com/tsender/bacre_2D) which was developed for our research paper below.

### Citation

If you use our work in an academic context, we would greatly appreciate it if you used the following citation:

```
@article{SenderRAL2024_BACRE,
  author={Sender, Ted and Brudnak, Mark and Steiger, Reid and Vasudevan, Ram and Epureanu, Bogdan},
  journal={IEEE Robotics and Automation Letters}, 
  title={A Regret-Informed Evolutionary Approach for Generating Adversarial Scenarios for Black-Box Off-Road Autonomy Systems}, 
  year={2024},
  volume={9},
  number={6},
  pages={5354-5361},
  keywords={Trajectory;Closed box;Testing;Costs;Navigation;Complexity theory;Vehicle dynamics;Planning under uncertainty;simulation and animation;software, middleware and programming environments},
  doi={10.1109/LRA.2024.3387109}
}
```

The journal article can be found [here](https://ieeexplore.ieee.org/document/10496154).

## Software Requirements and Installation

### Supported Systems
This repo was written and tested with ROS2 Foxy on Ubuntu 20.04 in a custom docker image, but it should work on Foxy and up. We only support the use of native Ubuntu.

### Dependencies
While there are a handful of external libraires needed for the Python 3 and C++ code to compile and run, it is highly recommended that you simply use our custom docker image as it contains the minimum amount of software libraries that you may need. You can download our docker image with the tag `tsender/tensorflow:gpu-focal-foxy` (you may need to login to your docker account from the command line to pull the image). But if you wish to add more libraries for your own code, you can inspect the [original docker image](https://github.com/tsender/dockerfiles/blob/main/tensorflow_foxy/Dockerfile) for everything it contains.

For reference, these are the libraries we rely on that are not commonly included in standard C++ or Python 3 installations:
- (C++) The `date` library https://github.com/HowardHinnant/date, used for logging purposes
- (Python 3) `paramiko`, used for ssh

As mentioned in the [AutomaticSceneGeneration](https://github.com/tsender/AutomaticSceneGeneration) documentation, we utilize the `ROSIntegration` UE4 plugin for ROS communication, which further requires use of the `rosbridge_suite` repo (which will run on Ubuntu). As far as the Ubuntu dependencies is concerned, you will need to use the `ros2` branch on @tsender's fork of [rosbridge_suite](https://github.com/tsender/rosbridge_suite/tree/ros2) because the authors of `rosbridge_suite` have not yet accepted the PR https://github.com/RobotWebTools/rosbridge_suite/pull/824 (please feel free to contribute to the PR in any way).

### Installation

1. Download the provided docker image or add all of its libraries to your Ubuntu system.
2. Let's put all of our code in a common folder: `mkdir ~/auto_scene_gen_ws`
3. Let's clone and build rosbridge_suite
   ```
   cd ~/auto_scene_gen_ws/
   mkdir rosbridge_suite
   cd rosbridge_suite/
   git clone https://github.com/tsender/rosbridge_suite.git src
   source /opt/ros/foxy/setup.bash
   colcon build
   ```
3. Create a new ROS workspace as the main location for all of your code. Let's assume it has the file path `~/auto_scene_gen_ws/my_workspace/`.
4. Copy the two `auto_scene_gen_*` packages into `~/auto_scene_gen_ws/my_workspace/src/`.
5. Let's build our new workspace. You only need to do this step if your workspace has not been built before. Note, we use the rosbridge_suite installation from above as an underlay.
   ```
   cd ~/auto_scene_gen_ws/
   source rosbridge_suite/install/setup.bash # rosbridge_suite is our underlay
   cd ~/auto_scene_gen_ws/my_workspace/
   colcon build
   ```
6. In all new terminals you open, make sure to source the overlay (i.e., your main workspace) before working.
   ```
   source ~/auto_scene_gen_ws/my_workspace/install/setup.bash
   ```

## Known Problems

1. My custom AutoSceneGenClient node is no longer able to send RunScenario requests to the AutoSceneGenWorkers in UE4.
   - To the best of our knowledge, this is a problem with the rosbridge_suite code. Unfortunately we do not know what causes this problem. In our experience, it occurs randomly after a few hours of running and we have been unable to find a repeatable sequence of events that causes this behavior. The only known solution is to shutdown and restart the affected rosbridge_suite nodes.