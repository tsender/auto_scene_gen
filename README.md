# ROS2 Interface for the AutomaticSceneGeneration UE4 plugin

## Description

This repo contains the base ROS2 interface for interacting with the [AutomaticSceneGeneration](https://github.com/tsender/AutomaticSceneGeneration) plugin for UE4.

## Software Requirements

**Supported Systems:**
- Ubuntu and ROS2 Foxy+
  - This repo was written and tested with ROS2 Foxy on Ubuntu 20.04, but it should work on Foxy and up.

**Usage Dependencies:**
While this interface doesn't have dependencies to external libraries (other than standard ROS2, C++, and Python 3 libraries), when you use the
3. [rosbridge_suite](https://github.com/tsender/rosbridge_suite/tree/main): Required by the ROSIntegration plugin. Use the `main` branch on @tsender's fork because the authors of `rosbridge_suite` have not yet accepted accepted the PR https://github.com/RobotWebTools/rosbridge_suite/pull/824 (please feel free to contribute to the PR in any way).

## Overview

This interface contains two ROS packages:
1. `auto_scene_gen_msgs`: Contains the custom message and service definitions. See LINK for a detailed description of these messages and services.
2. `auto_scene_gen_core`: Contains the main ROS nodes and various objects needed to interact with an AutoSceneGenWorker and AutoSceneGenVehicle in UE4.

