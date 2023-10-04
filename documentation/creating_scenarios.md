# ROS Interface for Creating and Running Scenarios with the AutomaticSceneGeneration Plugin

Quick Links:
- [Home Page](https://github.com/tsender/auto_scene_gen)
- [Message and service definitions](https://github.com/tsender/auto_scene_gen/blob/main/documentation/msg_and_srv_reference.md)
- [Creating vehicle nodes](https://github.com/tsender/auto_scene_gen/blob/main/documentation/creating_vehicle_nodes.md)

One of the primary functions of this interface is to provide a set of tools to request AutoSceneGenWorkers to create various scenarios for off-road AV testing, conduct the simulated navigation task, and report back what the vehicle's actions were. The interface we developed for this task requires an AutoSceneGenWorkerRef, an AutoSceneGenScenarioBuilder, and an AutoSceneGenClient node. This particular interface only supports the use of Python (using our provided classes).

## Terminology

In the context of the AutoSceneGen ecosystem, a *scene* refers to the physical static operating environment for the vehicle (this includes the geometric structure and any textural properties like weather conditions). The *environment* describes both the static and dynamic elements of the entire operating environment (including any dynamic actors). And the *scenario* combines the static and dynamic environment descriptions with the task assigned to the vehicle/agent and any constraints that may be imposed on its behavior.

Whether we are discussing the scene, environment, or scenario, all of these objects are composed of a collection of attributes that define that particular object. We will decompose these attributes into various categories, with the main ones being:
- Strutural Attributes: These attributes define the geometry of all structural objects that compose the scene. This includes the ground plane, static obstacles, water bodies, etc.
  - We specifically decompose these attributes into those pertaining to the landscape (or ground plane) and structural scene actors (SSAs), which are static structural objects like trees, bushes, rocks, etc.
- Textural Attributes: These attributes enhance the visual realism of the environment by describing visual effects. These attributes include weather intensities.
- Operational Attributes: These attributes describe operational constraints on the vehicle's behavior  (e.g., go/no-go zones, speed ranges, etc.), can define the vehicle's task, as well as describe the behavior of other agents.

## Abbreviations

Instead of always writing the prefix "AutoSceneGen", we will often use a shorthand when referring to the various components in the AutoSceneGen ecosystem. The shorthand only applies if it makes based on the context.
- Worker: Refers to an instance of an AutoSceneGenWorker, which resides in Unreal Engine.
- Vehicle: Refers to an instance of an AutoSceneGenVehicle, which resides in Unreal Engine.
- Vehicle Node: Refers to an instance of an AutoSceneGenVehicleNode, which is a Python or C++ ROS node within the autonomy stack controlling an AutoSceneGenVehicle.

## AutoSceneGenWorkerRef

Every AutoSceneGenWorker that we interact with via ROS needs a *reference* or a *proxy* object to keep track of important information regarding what that worker's status is and what it is doing. Below is a list of some of the items this object keeps track of:
- The corresponding worker ID
- The worker's status
- If the worker is registered with the client
- The vehicle nodes registered to control the corresponding vehicle
- If the vehicle nodes are ready to proceed (i.e., their internal states have been reset)
- The current scenario number being run
- The latest RunScenario request the worker is using
- The AnalyzeScenario request resulting from running the requested scenario
- If the worker is actively running a scenario

Please refer to the `AutoSceneGenWorkerRef` class in `auto_scene_gen_core/client_node.py` for more details. You may also create child classes if you need to add additional functionality for your own needs.

## AutoSceneGenScenarioBuilder

This is the base class for creating scenarios. This platform currently only supports creating scenarios for a navigation task for a single vehicle, and all scenarios must be defined via the ROS interface. All classes mentioned in this section can be found in `auto_scene_gen_core/scenario_builder.py`, and all message/service definitions can be found in LINK.

### Scenario Attributes

As mentioned above, scenarios are composed of a collection of various attributes. Before we can create a scenario in code, we must first define groups of scenario attributes (e.g., structural attributes, textural attributes, etc.), the types of attributes that each group contains, and the allowed value (or range of values) that each attribute can take on. The `ScenarioAttribute` class is the lowest level for defining any type of attribute, and takes as input the name of the attribute and the allowed value (or range of values). The `ScenarioAttributeGroup` class is a base class used for defining groups of attributes. All attribute groups that we define inherit from this class. This interface provides several base scenario attribute groups.

#### Structural Scene Actor Attributes

- Class Name: `StructuralSceneActorAttributes`
- Description: Describes the geometric attributes related to SSAs. While we could make a separate class for each type of SSA, right now we choose the simple route of having only one group that applies to all SSAs.
- Attributes:
  - `x`: The x-coordinate for the SSA [m]
  - `y`: The y-coordinate for the SSA [m]
  - `yaw`: The yaw angle for the SSA [deg]
  - `scale`: The scale factor for the SSA (applies to all three dimensions) multilying the max_scale parameter for the particular SSA (see `StructuralSceneActorConfig`)

#### Textural Attributes

- Class Name: `TexturalAttributes`
- Description: Describes the textural attributes of the environment.
- Attributes:
  - `sunlight_inclination`: The angle the sun makes with the horizontal plane [deg]
  - `sunlight_yaw`: The angle in which the sun is pointing in [deg]
 
#### Operational Attributes

- Class Name: `OperationalAttributes`
- Description: Describes the operational attributes for the scenario.
- Attributes:
  - `start_location_x`: The x-coordinate for the vehicle's starting position [m]
  - `start_location_y`: The y-coordinate for the vehicle's starting position [m]
  - `start_yaw`: The yaw angle for the vehicle's starting position [deg]
  - `goal_location_x`: The x-coordinate for the vehicle's goal position [m]
  - `goal_location_y`: The y-coordinate for the vehicle's goal position [m]
  - `goal_radius`: The goal radius [m]
  - `sim_timeout_period`: The simulation timeout period [s]
  - `vehicle_idling_timeout_period`: The maximum amount of time the vehicle can idle [s]
  - `vehicle_stuck_timeout_period`: The maximum amount of time the vehicle can be stuck [s]
  - `max_vehicle_roll`: The maximum allowed roll angle for the vehicle [deg]
  - `max_vehicle_pitch`: The maximum allowed pitch angle for the vehicle [deg]

### Structural Scene Actor Config

When working with SSAs, we will need two types of objects: the `StructuralSceneActorAttributes` object discussed above and a `StructuralSceneActorConfig` object. For every SSA we will need a `StructuralSceneActorConfig` object that contains additional information needed to populate the scene. Creating a `StructuralSceneActorConfig` requires the following parameters:
- `blueprint_directory`: The directory to find the Blueprint in the UE project, starts with "/Game/"
- `blueprint_name`: The name of the Blueprint (excluding extensions)
- `num_instances`: The number of instances that can be placed in the game
- `max_scale`: The maximum scale factor (we keep this separate from the scene attributes in case the user wants more control)
- `ssa_type`: The type of SSA, e.g., tree or bush
The first four parameters are used by the AutoSceneGenClient node when populating the `RunScenario` request.

### Creating Scenarios

The `AutoSceneGenScenarioBuilder` class is the main class used to create scenarios and it requires the following parameters:
- `landscape_nominal_size`: The size of the nominal landscape in [m]. See documentation for the [AutoSceneGenLandscape](https://github.com/tsender/AutomaticSceneGeneration/blob/main/Documentation/actors.md) actor for more details.
- `landscape_subdivisions`: The number of times the triangles in the nominal landscape mesh should be subdivided. See documentation for the [AutoSceneGenLandscape](https://github.com/tsender/AutomaticSceneGeneration/blob/main/Documentation/actors.md#autoscenegenlandscape) actor for more details.
- `landscape_border`: The minimum allowed amount of padding for the landscape border. See documentation for the [AutoSceneGenLandscape](https://github.com/tsender/AutomaticSceneGeneration/blob/main/Documentation/actors.md#autoscenegenlandscape) actor for more details.
- `ssa_attr`: The `StructuralSceneActorAttributes` instance to use
- `ssa_config`: A list of `` objects for the various SSAs that can be placed in the scene
- `b_ssa_casts_shadow`: Indicates if the SSAs cast a shadow in the game
- `b_allow_collisions`: Indicate if simulation keeps running in the case of vehicle collisions
- `txt_attr`: The `TexturalAttributes` instance to use
- `opr_attr`: The `OperationalAttributes` instance to use
- `start_obstacle_free_radius`: Obstacle-free radius [m] around the start location
- `goal_obstacle_free_radius`: Obstacle-free radius [m] around the goal location

The `AutoSceneGenScenarioBuilder` class provides a number of basic methods that may be useful in your own applications. The functions worth discussing are shown below:
- `create_default_run_scenario_request`
  - Calling this function will create and return an `auto_scene_gen_msgs/RunScenario` request using the default attribute values provided in the scene builder parameters.
- `create_default_scene_description_msg`:
  - This function is internally called by `create_default_run_scenario_request` and creates a `auto_scene_gen_msgs/SceneDescription` with the default parameters.
- `get_unreal_engine_run_scenario_request`:
  - When we create the scene/scenario via code, we do so assuming a right-handed north-west-up coordinate frame with meters as the base positional unit of measurement. However, UE uses a left-handed coordinate system and uses centimeters. Calling this function will return the equivalent `RunScenario` request so that UE an properly create and run the scenario.
- `get_unreal_engine_scene_description`:
  - This function is called internally by `get_unreal_engine_run_scenario_request`, and performs the conversion on the `auto_scene_gen_msgs/SceneDescription` message instance passed to it.
- `is_scenario_request_feasible`:
  - Since it is important to create scenarios that are feasible for the test system, it may be useful to have a custom function that evaluates if a given `RunScenario` request satisfies your feasibility criteria. In the base class this function simply returns true. But this function can be overriden in a child scenario builder class.

There are several other functions provided by the `AutoSceneGenScenarioBuilder` class that may be useful. Please look through the source code for their description. 

Depending on your needs, you are welcome to add features to the UE4 plugin and to this interface. However, extending the scene building functionality requires that you are able to modify *all* components in the scene building pipeline, including:
- The AutoSceneGenWorker
- The AutoSceneGenScenarioBuilder and/or your new child class (including many of the provided functions)
- The `auto_scene_gen_msgs/RunScenario` request definition in both the ROS and UE4 version of the `auto_scene_gen_msgs` package.

## AutoSceneGenClient Node

The AutoSceneGenClient ROS node provides the base functionality needed to interact with the AutoSceneGenWorker that operates in Unreal Engine. Internally, this node manages a variety of things that allow you to seamlessly create various scenarios in Unreal Engine, observe and analyze the test vehicle's behavior from that scenario, and then repeat the process.

### ROS Objects

Lists any publishers, subscribers, clients, services, and or timers monitored by this node. All instances of `<asg_client_name>` in the below topic names get replaced with the name provided by the `asg_client_name` parameter. All instances of `<wid>` are place holders for a worker ID.

**Publishers:**
- Client Status Pub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Publishes the client node's status.
- Vehicle Node Operating Info Pub
  - Topic: `/<asg_client_name>/vehicle_node_operating_info`
  - Type: `auto_scene_gen_msgs/VehicleNodeOperatingInfo`
  - Description: Publishes important operating information to all registered vehicle nodes.
- Scene Description Pubs (one for each worker)
  - Topic: `/asg_worker<wid>/scene_description`
  - Type: `auto_scene_gen_msgs/SceneDescription`
  - Description: Publishes the most recent scene description for the associated worker.

**Subscribers:**
- Worker Status Subs
  - Topic: `/asg_worker<wid>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the associated worker's status.
 
**Clients:**
- Run Scenario Clients (one for each worker)
  - Topic: `/asg_worker<wid>/services/run_scenario`
  - Type: `auto_scene_gen_msgs/RunScenario`
  - Description: Requests a specific scenario for the worker to create and execute.
 
**Services:**
- Analyze Scenario Service
  - Topic: `/<asg_client_name>/services/analyze_scenario`
  - Type: `auto_scene_gen_msgs/AnalyzeScenario`
  - Description: Service for accepting `AnalyzeScenario` requests back from the workers.
- Register Vehicle Node Service
  - Topic: `/<asg_client_name>/services/register_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Service for vehicle nodes to register themselves with the client node.
- Unregister Vehicle Node Service
  - Topic: `/<asg_client_name>/services/unregister_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Service for vehicle nodes to unregister themselves with the client node.
- Notify Ready Service
  - Topic: `/<asg_client_name>/services/notify_ready`
  - Type: `auto_scene_gen_msgs/NotifyReady`
  - Description: Service for vehicle nodes to indicate to the client node that they are ready to proceed.
- Worker Issue Notification Service
  - Topic: `/<asg_client_name>/services/worker_issue_notification`
  - Type: `auto_scene_gen_msgs/WorkerIssueNotification`
  - Description: Service for workers to indicate to the client node of any issues they encountered (e.g., rosbrige interruption).
 
**Timers**
- Main Loop Timer
  - Timer Callback: `main_loop_timer_cb`
  - Description: This is the main loop that runs until the node is destroyed. When it is running, it will publish the online status for the client, publish any vehicle node operating info for any vehicle nodes, publish the current scene description being ran in the managed workers, check/ensure the workers received their latest `RunScenario` request, and then call `main_step(wid: int)` and passes in the current worker ID being processed. The `main_step` function is what you will need to override and is your primary entry point for creating and analyzing scenarios. The `main_loop_timer_cb` function will cycle through all managed worker IDs so you don't have to.

### Creating and Customizing the AutoSceneGenClient Node

The `AutoSceneGenClient` class provided in `auto_scene_gen_core/cient_node.py` is the base client. You will need to create a child class that inherits from `AutoSceneGenClient` and then customize it to your needs. The base class requires the following parameters:
- `node_name`: The name of the ROS node
- `main_dir`: The main directory for storing data on your computer
- `asg_client_name`: The AutoSceneGenClient's name, which is used for creating the appropriate ROS topics (this does not need to match the ROS node name)
- `num_vehicle_nodes`: Number of AutoSceneGenVehicleNodes in the AutoScenegenVehicle's the autonomy stack
- `num_workers`: Number of AutoSceneGenWorkers to keep track of
- `base_wid`: The base, or starting, AutoSceneGenWorker ID
- `worker_class`: A class or subclass instance of AutoSceneGenWorkerRef, used for managing AutoSceneGenWorkers
- `scenario_builder`: A class or subclass instance of AutoSceneGenScenarioBuilder, used for creating scenarios

Due to the structure of the `AutoSceneGenClient` class, you can only customize what happens inside the constructor and inside the `main_step` function. The `main_step(wid: int)` function is called in each iteration that `main_loop_timer_cb` runs and is passed the ID of the current AutoSceneGenWorker being processed. The `main_step` function is where you should place your main processing code for creating and analyzing scenarios with the various workers. Based on how the various components in the entire workflow interact, there is a recommended way to configure the `main_step` function so that your code will operate as intended. Below is the minimilistic recommendation (you must follow this framework for everything to work properly):
```
def main_step(wid: int):
  worker = self.workers[wid]
  
  # Worker must be registered
  if not worker.b_registered_with_asg_client:
    return

  # Ensure all vehicle nodes are registered
  if len(worker.registered_vehicle_nodes) != self.num_vehicle_nodes:
    return

  # Ensure all vehicle nodes are ready to proceed before doing anything
  if not self.are_all_vehicle_nodes_ready(wid):
    return

  # Create a scenario...

  # Submit the RunScenario request
  self.submit_run_scenario_request(wid)

  # Block until ready to analyze scenario
  if worker.b_waiting_for_analyze_scenario_request:
    return

  # Analyze the latest scenario...

  # Check for some stopping criterion...
```

### Saving Vehicle Node Data

In a number of situations, you will want your vehicle nodes to save their data so you can later analyze the data, create plots/figures, etc. The `AutoSceneGenWorkerRef` class has two variables that you can set to control where your vehicle nodes save their data and to what extent. It is recommended that you set these variables before you submit each `RunScenario` request to ensure the vehicle nodes receive the updated information in their `VehicleNodeOperatingInfo` messages.
- `vehicle_node_save_dir`: The directory for the vehicle nodes associated with this worker's vehicle to save their data to. The directory must be on the same computer where the client resides and you should make sure it exists. Leave the string empty if you do not want the vehicle nodes to save any data.
- `b_save_minimal`: In some cases, you may want your vehicle nodes to save a lot of data so you can observe all of the decisions that the autonomy stack made (e.g., image classifications, LiDAR point clouds, etc.). However, saving lots of data consumes a lof of time and decreases the number of scenarios that can be run in a given period (becasue the client must wait for every vehicle node to submit a `NotifyReady` request before it can proceed). This variable gives you control over how much data you may want your vehicle nodes to save. Setting the variable to true means you want your vehicle nodes to only save the least amount of data. Setting this variable to false measn you want your nodes to save as much data as you desire (keep in mind this incurs a long waiting time before you can run another scenario).

### Logging

The AutoSceneGenClient makes extensive use of logging to help you identify potential errors. We provide a custom `log` function that allows you to log messages to the console and to a separate log file. The base class creates a log file at the file path `<main_dir>/asg_log.txt` where `<main_dir>` is replaced by the directory assigned to the client in the constructor. All messages are prepended with a date, timestamp, and log level, for example `[2023-09-17 23:30:09.599643] [INFO]`. Most operations within the base class write their messages to this log file, and it is recommended for you to do the same in your child class. Note, these log files can easily reach 10000+ or 100000+ lines, depending on how many scenarios you run and how much additional logging you add.
