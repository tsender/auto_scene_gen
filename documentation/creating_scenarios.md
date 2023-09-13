# ROS Interface for Creating and Running Scenarios with the AutomaticSceneGeneration Plugin

One of the primary functions of this interface is to provide a set of tools to request AutoSceneGenWorkers to create various scenarios for off-road AV testing, conduct the simulated navigation task, and report back what the vehicle's actions were. The interface we developed for this task requires an AutoSceneGenWorkerRef, an AutoSceneGenScenarioBuilder, and an AutoSceneGenClient node.

## Terminology

In the context of the AutoSceneGen ecosystem, a *scene* refers to the physical static operating environment for the vehicle (this includes the geometric structure and any textural properties like weather conditions). The *environment* describes both the static and dynamic elements of the entire operating environment (including any dynamic actors). And the *scenario* combines the static and dynamic environment descriptions with the task assigned to the vehicle/agent and any constraints that may be imposed on its behavior.

Whether we are discussing the scene, environment, or scenario, all of these objects are composed of a collection of attributes that define that particular object. We will decompose these attributes into various categories, with the main ones being:
- Strutural Attributes: These attributes define the geometry of all structural objects that compose the scene. This includes the ground plane, static obstacles, water bodies, etc.
  - We specifically decompose these attributes into those pertaining to the landscape (or ground plane) and structural scene actors (SSAs), which are static structural objects like trees, bushes, rocks, etc.
- Textural Attributes: These attributes enhance the visual realism of the environment by describing visual effects. These attributes include weather intensities.
- Operational Attributes: These attributes describe operational constraints on the vehicle's behavior  (e.g., go/no-go zones, speed ranges, etc.), can define the vehicle's task, as well as describe the behavior of other agents.

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

Please refer to the `AutoSceneGenWorkerRef` class in auto_scene_gen_core/client_node.py for more details. You may also create child classes if you need to add additional functionality for your own needs.

## AutoSceneGenScenarioBuilder

This is the base class for creating scenarios. This platform currently only supports creating scenarios for a navigation task for a single vehicle, and all scenarios must be defined via the ROS interface. All classes mentioned in this section can be found in auto_scene_gen_core/scenario_builder.py, and all message/service definitions can be found in LINK.

### Scenario Attributes

As mentioned above, scenarios are composed of a collection of various attributes. Before we can create a scenario in code, we must first define groups of scenario attributes (e.g., structural attributes, textural attributes, etc.), the types of attributes that each group contains, and the allowed value (or range of values) that each attribute can take on. The `ScenarioAttribute` class is the lowest level for defining any type of attribute, and takes as input the name of the attribute and the allowed value (or range of values). The `ScenarioAttributeGroup` class is a base class used for defining groups of attributes. All attribute groups that we define inherit from this class. This interface provides several base scenario attribute groups.

#### Structural Scene Actor Attributes
- Class Name: `StructuralSceneActorAttributes`
- Description: Describes the geometric attributes related to SSAs. While we could make a separate class for each type of SSA, right now we choose the simple route of having only one group that applies to all SSAs.
- Attributes:
  - `x`: The x-coordinate for the SSA [m]
  - `y`: The y-coordinate for the SSA [m]
  - `yaw`: The yaw angle for the SSA [deg]
  - `scale`: The scale factor for the SSA (applies to all three dimensions) multilying the max_scale parameter for the particular SSA (see StructuralSceneActorConfig)

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
When working with SSAs, we need both the `StructuralSceneActorAttributes` object as well as a special `SctructuralSceneActorConfig` instance for each SSA. This objects holds additional information needed to 

### Creating Scenarios
To create a scenario, we first create instances of the `ScenarioAttributeGroups` discussed above. Then we can create an instance of the `AutoSceneGenScenarioBuilder` class which requires the following parameters:
- `landscape_nominal_size`: The size of the nominal landscape in [m]. See documentation for the [AutoSceneGenLandscape](https://github.com/tsender/AutomaticSceneGeneration/blob/main/Documentation/actors.md) actor for more details.
- `landscape_subdivisions`: The number of times the triangles in the nominal landscape mesh should be subdivided. See documentation for the [AutoSceneGenLandscape](https://github.com/tsender/AutomaticSceneGeneration/blob/main/Documentation/actors.md) actor for more details.
- `landscape_border`: The minimum allowed amount of padding for the landscape border. See documentation for the [AutoSceneGenLandscape](https://github.com/tsender/AutomaticSceneGeneration/blob/main/Documentation/actors.md) actor for more details.
- `ssa_attr`: The StructuralSceneActorAttributes instance to use
- 

## AutoSceneGenClient Node
