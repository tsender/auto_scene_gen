# ROS Interface for Creating Scenarios with the AutomaticSceneGeneration Plugin

One of the primary functions of this interface is to provide a set of tools to request AutoSceneGenWorkers to create various scenarios for off-road AV testing, conduct the simulation navigation task, and report back what the vehicle's actions were. The interface we developed for this task requires an AutoSceneGenWorker reference, a scenario buuilder, and an AutoSceneGenClient node.

## AutoSceneGenWorker Reference

Every AutoSceneGenWorker that we interact with via ROS needs a *reference* or a *proxy* object to keep track of important information regarding what that worker's status is and what it is doing. Below is a list of the more important pieces of information this object keeps track of:
- The corresponding worker ID
- The worker's status
- If the the worker registered with the AutoSceneGenClient
- The AutoSceneGenVehicleNodes registered to control the corresponding AutoSceneGenVehicle
- If the AutoSceneGenVehicleNodes are ready to proceed (i.e., their internal states have been reset)
- The current scenario number being run
- The latest RunScenario request the worker is using
- The AnalyzeScenario request resulting from running the requested scenario
- If the worker is actively running a scenario

Please refer to the `AutoSceneGenWorkerRef` class in auto_scene_gen_core/client_node.py for more details.

## Scenario Builder


## AutoSceneGenClient Node
