# ROS Interface for Creating Scenarios with the AutomaticSceneGeneration Plugin

One of the primary functions of this interface is to provide a set of tools to request AutoSceneGenWorkers to create various scenarios for off-road AV testing, conduct the simulated navigation task, and report back what the vehicle's actions were. The interface we developed for this task requires an AutoSceneGenWorkerRef, an AutoSceneGenScenarioBuilder, and an AutoSceneGenClient node.

## Terminology

In the context of the AutoSceneGen ecosystem, a *scene* refers to the physical static operating environment for a system (this includes the geometric structure and any textural properties like weather conditions). The *environment* describes both the static and dynamic elements of the entire operating environment (including any dynamic actors). And the *scenario* combines the static and dynamic environment descriptions with the task assigned to the vehicle/agent and any constraints that may be imposed on its behavior.

Whether we are discussing the scene, environment, or scenario, all of these objects are composed of a collection of attributes that define that particular object. We will decompose these attributes into various categories, with the main ones being:
- Strutural Attributes: These attributes define the geometry of all structural objects that compose the scene. This includes the ground plane, static obstacles, water bodies, etc.
- Textural Attributes: These attributes enhance the visual realism of the environment by describing visual effects. These attributes include weather intensities.
- Operational Attributes: These attributes describe operational constraints on the test system's behavior  (e.g., go/no-go zones, speed ranges, etc.), can define the system's task, as well as describe the behavior of other agents.

## AutoSceneGenWorkerRef

Every AutoSceneGenWorker that we interact with via ROS needs a *reference* or a *proxy* object to keep track of important information regarding what that worker's status is and what it is doing. Below is a list of just some of the items this object keeps track of:
- The corresponding worker ID
- The worker's status
- If the worker is registered with the AutoSceneGenClient
- The AutoSceneGenVehicleNodes registered to control the corresponding AutoSceneGenVehicle
- If the AutoSceneGenVehicleNodes are ready to proceed (i.e., their internal states have been reset)
- The current scenario number being run
- The latest RunScenario request the worker is using
- The AnalyzeScenario request resulting from running the requested scenario
- If the worker is actively running a scenario

Please refer to the `AutoSceneGenWorkerRef` class in auto_scene_gen_core/client_node.py for more details.

## AutoScenegenScenarioBuilder

This is the base class for creating scenarios. Before we can discuss the details, we must first describe how a scenario is defined. 

## AutoSceneGenClient Node
