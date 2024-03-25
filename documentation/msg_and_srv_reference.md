This page contains the custom message and service definitions within the `auto_scene_gen_msgs` package.

Quick Links
- [Home Page](https://github.com/tsender/auto_scene_gen)
- [Messages Reference](#autoscenegen-messages-reference)
- [Services Reference](#autoscenegen-services-reference)

# AutoSceneGen Messages Reference

## [LandscapeDescription.msg](../auto_scene_gen_msgs/msg/LandscapeDescription.msg)

Contains the description for creating an AutoSceneGenLandscape.

**Field**               | **Type**          | **Description**
------------------------|-------------------|----------------
nominal_size            | float32           | The side-length [m or cm] of the nominal landscape along the X and Y dimensions (this is a square landscape).
subdivisions            | float32           | The number of times the two base triangles in the nominal landscape should be subdivided. The landscape will have 2^subdivisions triangles along each edge. Each vertex in the mesh will be spaced nominal_size/(2^subdivisions) [m or cm] apart in a grid.
border                  | float32           | (Optional) Denotes the approximate length to extend the nominal landscape in [m or cm]. <br>Using this will border the nominal landscape by ceil(Border/VertexSpacing) vertices in the four XY Cartesional directions, where VertexSpacing is discussed above.

## [OdometryWithoutCovariance.msg](../auto_scene_gen_msgs/msg/OdometryWithoutCovariance.msg)

This represents an estimate of a position and velocity in free space without covariance. The pose in this message should be specified in the coordinate frame given by header.frame_id. The twist in this message should be specified in the coordinate frame given by the child_frame_id.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
header                  | std_msgs/Header       | Includes the frame id of the pose parent.
child_frame_id          | string                | Frame id the pose points to. The twist is in this coordinate frame.
pose                    | geometry_msgs/Pose    | Estimated pose that is typically relative to a fixed world frame.
twist                   | geometry_msgs/Twist   | Estimated linear and angular velocity relative to child_frame_id.

## [PhysXControl.msg](../auto_scene_gen_msgs/msg/PhysXControl.msg)

Message for sending basic PhysX control commands to a PhysX vehicle.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
header                  | std_msgs/Header       | Message header.
longitudinal_velocity   | float32               | Vehicle's longitudinal velocity in [cm/s or m/s].
steering_angle          | float32               | Range = [-MaxSteeringAngle, +MaxSteeringAngle] in [deg].
handbrake               | bool                  | false = disengaged, true = engaged.

## [SceneCaptureSettings.msg](../auto_scene_gen_msgs/msg/SceneCaptureSettings.msg)

Message for specifying scene capture settings for the AutoSceneGenWorker. The camera used has a 60 deg. FOV.

**Field**                 | **Type**            | **Description**
--------------------------|---------------------|----------------
image_size                | uint32              | The image size in pixels. All images are square.
draw_annotations          | bool                | Indicates if the scene captures should contain annotations.
goal_sphere_thickness     | float32             | (Annotation) Goal sphere thickness in [cm].
goal_shere_color          | std_msgs/ColorRGBA  | (Annotation) Goal sphere color.
ortho_aerial              | bool                | Draw orthographic aerial view.
perspective_aerial        | bool                | Draw perspective aerial view.
aerial_padding            | uint32[]            | Padding in [m] to apply to the base orthographic width.
front_aerial              | bool                | Draw perspective aerial-like view from the front.
left_front_aerial         | bool                | Draw perspective aerial-like view from the left-front.
left_aerial               | bool                | Draw perspective aerial-like view from the left.
left_rear_aerial          | bool                | Draw perspective aerial-like view from the left-rear.
rear_aerial               | bool                | Draw perspective aerial-like view from the rear.
right_rear_aerial         | bool                | Draw perspective aerial-like view from the right rear.
right_aerial              | bool                | Draw perspective aerial-like view from the right.
right_front_aerial        | bool                | Draw perspective aerial-like view from the right-front.
vehicle_start_pov         | bool                | Draw perspective view of the vehicle's POV at its starting location.
vehicle_start_rear_aerial | bool                | Draw perspective 3rd person rear aerial view of the vehicle at the starting location.

## [SceneDescription.msg](../auto_scene_gen_msgs/msg/SceneDescription.msg)

This message contains information regarding the scene description.

**Field**               | **Type**                                  | **Description**
------------------------|-------------------------------------------|----------------
landscape               | auto_scene_gen_msgs/LandscapeDescription          | The landscape description.
sunlight_inclination    | float32                                           | The angle the sunlight makes with the horizontal [deg].
sunlight_yaw_angle      | float32                                           | The yaw angle the sunlight is pointing in (i.e., the angle the shadow will be cast in) [deg].
ssa_array               | auto_scene_gen_msgs/StructuralSceneActorLayout[]  | An array defining all of the types of structural scene actors and their attributes from which to place in the UE4 scene.

## [StatusCode.msg](../auto_scene_gen_msgs/msg/StatusCode.msg)

Indicates the current status of some entity.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
status                  | uint8                 | The current status.<br>OFFLINE = 0<br>ONLINE_AND_READY = 1<br>ONLINE_AND_RUNNING = 2

## [StructuralSceneActorLayout.msg](../auto_scene_gen_msgs/msg/StructuralSceneActorLayout.msg)

This message specifies all of the attributes for the actors of a specific structural scene actor (SSA) subclass that we wish to place in a UE4 scene.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
path_name               | string                | UE path name for the SSA subclass.
num_instances           | uint16                | For verifications purposes, this is used to determine the number of instances.
visible                 | bool[]                | Indicates if the actor is visible in the game. Toggling this is more efficient than adding/removing elements when creating the UE4 scene.
cast_shadow             | bool[]                | Indicates if the actor can cast a shadow in the game.
x                       | float32[]             | X coordinates in [cm or m].
y                       | float32[]             | Y coordinates in [cm or m].
yaw                     | float32[]             | Yaw angles in [deg].
scale                   | float32[]             | The mesh's scale (applies to all 3 axes).


## [VehicleNodeOperatingInfo.msg](../auto_scene_gen_msgs/msg/VehicleNodeOperatingInfo.msg)

This message is published by the AutoSceneGenClient to tell all registered vehicle nodes important operating information. This info includes: if all vehicle nodes for that worker are ready, where the nodes should save their internal data and how much data to save. Each array has the same length and all data is organized according to the order of the worker ID in the 'worker_ids' field.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
worker_ids              | uint8[]               | AutoSceneGenWorker IDs.
scenario_numbers        | uint32[]              | List of the scenario numbers being run on each AutoSceneGenWorker, listed in the same order as worker_ids.
ok_to_run               | bool[]                | Indicates if it is okay for the vehicle nodes to run, listed in the same order as worker_ids.
save_dirs               | string[]              | List of directories to save data to, listed in the same order as worker_ids.
save_minimal            | bool[]                | List of booleans to indicate if the vehicle node should only save the least amount of data required (and skip any time-consuming analyses), listed in the same order as worker_ids.

## [VehicleStatus.msg](../auto_scene_gen_msgs/msg/VehicleStatus.msg)

This message contains the enable status of an AutoSceneGenVehicle.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
enabled                 | bool                  | Indicates if the vehicle is currently enabled in the UE game.
preempted               | bool                  | Indicates if the vehicle was disabled preemptively (only applies if enabled is False).

# AutoSceneGen Services Reference

## [AnalyzeScenario.srv](../auto_scene_gen_msgs/srv/AnalyzeScenario.srv)

This service message is used to convey useful information about the vehicle's last run.

### Request
**Field**               | **Type**                                        | **Description**
------------------------|-------------------------------------------------|----------------
worker_id               | uint8                                           | AutoSceneGenWorker ID sending the request.
scenario_number         | uint16                                          | Scenario number that the AutoSceneGenWorker just executed.
termination_reason      | uint8                                           | Reason for ending the simulation. See table below for possible values.
vehicle_trajectory      | auto_scene_gen_msgs/OdometryWithoutCovariance[] | The vehicle's trajectory from start to goal.
vehicle_sim_time        | float32                                         | Total length of vehicle simulation time (from when the first control input was received).

Possible Termination Reasons
**Reason**                      | **Value**     | Description
------------------------        |---            |---
REASON_SUCCESS                  | 0             | Vehicle reached the goal safely within the allotted time.
REASON_VEHICLE_COLLISION        | 1             | Vehicle crashed into a non-traversable obstacle (any form of contact counts as collision).
REASON_VEHICLE_FLIPPED          | 2             | Vehicle turned or flipped over.
REASON_SIM_TIMEOUT              | 3             | Simulation timeout expired.
REASON_VEHICLE_IDLING_TIMEOUT   | 4             | Vehicle idling timeout experied.
REASON_VEHICLE_STUCK_TIMEOUT    | 5             | Vehicle got stuck enroute (e.g., due to a collision) and could not recover within a certain timeout expired. Note: we separate stuck from having flipped over.

Idling and Getting Stuck
- We define *idling* as commanding near-zero velocity while undergoing near-zero velocity.
- We define *being stuck* as commanding non-zero velocity while undergoing near-zero velocity.

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
received                | bool                  | Indicates the AutoSceneGenClient received the AnalyzeScenario request.

## [NotifyReady.srv](../auto_scene_gen_msgs/srv/NotifyReady.srv)

This service message is used for each vehicle node to notify the AutoSceneGenClient that it is ready for the next scenario. These notifications allow the client to ensure all nodes are ready before proceeding on to the next scenario.

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
worker_id               | uint8                 | AutoSceneGenWorker ID this node is associated with.
node_name               | string                | Name of the registered node.
last_scenario_number    | uint32                | Last scenario number that was just ran.
request_rerun           | bool                  | Indicate if the vehicle node wants to rerun the previous scenario (e.g., due to a problem it encountered).
reason_for_rerun        | string                | Reason for requesting a rerun
 
### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
received                | bool                  | Indicates the AutoSceneGenClient received the notification.
accepted                | bool                  | Indicates the AutoSceneGenClient accepted the request. False means it was ignored (since it accepted the first request).

## [RegisterVehicleNode.srv](../auto_scene_gen_msgs/srv/RegisterVehicleNode.srv)

This service message is used for each vehicle node to register itself with the AutoSceneGenClient. Registering guarantees the AutoSceneGenClient will not proceed until all registered vehicle nodes have indicated they are ready to proceed with a NotifyReady request.

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
worker_id               | uint8                 | AutoSceneGenWorker ID this node is associated with.
node_name               | string                | Name of the node to register.

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
received                | bool                  | Indicates the AutoSceneGenClient received the notification.

## [RunScenario.srv](../auto_scene_gen_msgs/srv/RunScenario.srv)

This service message is used to define the scenario that the AutoSceneGenWorker should run. RunScenario requests are to be submitted by an external entity from UE4 which we refer to as an AutoSceneGenClient.

### Request
**Field**                     | **Type**              | **Description**
------------------------------|-----------------------|----------------
scenario_number               | int32                               | Mainly just used to check on progress.
sim_timeout_period            | float32                             | Maximum amount of time [s] to let the simulation run before terminating. Set to -1 to disable feature.
vehicle_idling_timeout_period | float32                             | Maximum amount of time [s] the vehicle can idle (once it began moving) before terminating the simulation. Set to -1 to disable feature.
vehicle_stuck_timeout_period  | float32                             | Maximum amount of time [s] the vehicle can be "stuck", like on an obstacle, before terminating the simulation. Set to -1 to disable feature.
max_vehicle_roll              | float32                             | Max allowed vehicle roll angle [deg]. Simulation will end if this threshold is met.
max_vehicle_pitch             | float32                             | Max allowed vehicle pitch angle [deg]. Simulation will end if this threshold is met.
allow_collisions              | bool                                | If true, then the simulator will not terminate the simulation if the vehicle touches a non-traversable obstacle. If false, then the simulation will terminate with reason REASON_VEHICLE_COLLISION (see AnalyzeScenario.srv) if the vehicle touches a non-traversable obstacle.
vehicle_start_location        | geometry_msgs/Point                 | Vehicle start location in [cm]. The Z location is ignored and will be populated by the AutoSceneGenWorker in UE4.
vehicle_start_yaw             | float32                             | Vehicle starting yaw angle in [deg].
vehicle_goal_location         | geometry_msgs/Point                 | Vehicle goal location in [cm]. The Z location is ignored and will be populated by the AutoSceneGenWorker in UE4.
goal_radius                   | float32                             | If vehicle is within this distance in [cm] of the goal location, then we assume the vehicle succeeded.
scene_description             | auto_scene_gen_msgs/SceneDescription | The scene description.
take_scene_capture            | bool                                | Indicates if we should take scene captures to send back to the client.
scene_capture_only            | bool                                | Indicates if we should only take scene captures after creating the scene (the scneario will NOT be run).
scene_capture_settings        | auto_scene_gen_msgs/SceneCaptureSettings | Scene capture settings.

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
received                | bool                  | Indicates the AutoSceneGenWorker received the RunScenario request

## [WorkerIssueNotification.srv](../auto_scene_gen_msgs/srv/WorkerIssueNotification.srv)

This services is to be invoked by the AutoSceneGenWorker to inform the AutoSceneGenClient of an issue.

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
worker_id               | uint8                 | Worker ID sending the notification
issue_id                | uint8                 | Issue at hand.<br>ISSUE_ROSBRIDGE_INTERRUPTED = 0<br>ISSUE_PROBLEM_CREATING_SCENE = 1
message                 | string                | Message string, if any message needs to be relaid to the AutoSceneGen client. Can be empty.

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
received                | bool                  | Indicates the AutoSceneGenClient received the notification.
