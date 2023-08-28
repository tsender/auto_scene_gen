# AutoSceneGen Messages Reference

## LandscapeDescription.msg

Contains the description for creating an AutoSceneGenLandscape.

**Field**               | **Type**          | **Description**
------------------------|-------------------|----------------
nominal_size            | float32           | The side-length [m or cm] of the nominal landscape along the X and Y dimensions (this is a square landscape).
subdivisions            | float32           | The number of times the two base triangles in the nominal landscape should be subdivided. The landscape will have 2^subdivisions triangles along each edge. Each vertex in the mesh will be spaced nominal_size/(2^subdivisions) [m or cm] apart in a grid.
border                  | float32           | (Optional) Denotes the approximate length to extend the nominal landscape in [m or cm]. <br>Using this will border the nominal landscape by ceil(Border/VertexSpacing) vertices in the four XY Cartesional directions, where VertexSpacing is discussed above.

## OdometryWithoutCovariance.msg

This represents an estimate of a position and velocity in free space without covariance. The pose in this message should be specified in the coordinate frame given by header.frame_id. The twist in this message should be specified in the coordinate frame given by the child_frame_id.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
header                  | std_msgs/Header       | Includes the frame id of the pose parent.
child_frame_id          | string                | Frame id the pose points to. The twist is in this coordinate frame.
pose                    | geometry_msgs/Pose    | Estimated pose that is typically relative to a fixed world frame.
twist                   | geometry_msgs/Twist   | Estimated linear and angular velocity relative to child_frame_id.

## PhysXControl.msg

Message for sending basic PhysX control commands to a PhysX vehicle.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
header                  | std_msgs/Header       | Message header.
longitudinal_velocity   | float32               | Vehicle's longitudinal velocity in [cm/s or m/s].
steering_angle          | float32               | Range = [-MaxSteeringAngle, +MaxSteeringAngle] in [deg].
handbrake               | bool                  | false = disengaged, true = engaged.

## SceneCaptureSettings.msg

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

## SceneDescription.msg

This message contains information regarding the scene description.

**Field**               | **Type**                                  | **Description**
------------------------|-------------------------------------------|----------------
landscape               | auto_scene_gen_msgs/LandscapeDescription          | The landscape description.
sunlight_inclination    | float32                                           | The angle the sunlight makes with the horizontal [deg].
sunlight_yaw_angle      | float32                                           | The yaw angle the sunlight is pointing in (i.e., the angle the shadow will be cast in) [deg].
ssa_array               | auto_scene_gen_msgs/StructuralSceneActorLayout[]  | An array defining all of the types of structural scene actors and their attributes from which to place in the UE4 scene.

## StatusCode.msg

Indicates the current status of some entity.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
status                  | uint8                 | The current status.<br>OFFLINE = 0<br>ONLINE_AND_READY = 1<br>ONLINE_AND_RUNNING = 2

## StructuralSceneActorLayout.msg

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


## VehicleNodeOperatingInfo.msg

This message is published by the AutoScenegen client to tell all registered vehicle nodes important operating information. This info includes: if all vehicle nodes for that worker are ready, where the nodes should save their internal data and how much data to save. Each array has the same length and all data is organized according to the order of the worker ID in the 'worker_ids' field.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
worker_ids              | uint8[]               | AutoSceneGen worker IDs.
scenario_numbers        | uint32[]              | List of the scenario numbers being run on each AutoSceneGen worker, listed in the same order as worker_ids.
ok_to_run               | bool[]                | Indicates if it is okay for the vehicle nodes to run, listed in the same order as worker_ids.
save_dirs               | string[]              | List of directories to save data to, listed in the same order as worker_ids.
save_minimal            | bool[]                | List of booleans to indicate if the vehicle node should only save the least amount of data required (and skip any time-consuming analyses), listed in the same order as worker_ids.

## VehicleStatus.msg

This message contains the enable status of an AutoSceneGen vehicle.

**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
enabled                 | bool                  | Indicates if the vehicle is currently enabled in the UE game.
preempted               | bool                  | Indicates if the vehicle was disabled preemptively (only applies if enabled is False).

# AutoSceneGen Services Reference

# AnalyzeScenario.srv

This service message is used to convey useful information about the vehicle's last run.

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
worker_id               | uint8                 | AutoSceneGen worker ID sending the request.
scenario_number         | uint16                | Scenario number that the AutoSceneGenWorker just executed.
termination_reason      | uint8                 | Reason for ending the simulation. See table below for possible values.
vehicle_trajectory      | auto_scene_gen_msgs/OdometryWithoutCovariance[] | The vehicle's trajectory from start to goal.
vehicle_sim_time        | float32               | Total length of vehicle simulation time (from when the first control input was received).

Possible Termination Reasons
**Reason**                      | **Value**     | Description
------------------------        |---            |---
REASON_SUCCESS                  | 0             | Vehicle reached the goal safely within the alloted time.
REASON_VEHICLE_COLLISION        | 1             | Vehicle crashed into a non-traversable obstacle (any form of contact counts as collision).
REASON_VEHICLE_FLIPPED          | 2             | Vehicle turned or flipped over.
REASON_SIM_TIMEOUT              | 3             | Simulation timeout expired.
REASON_VEHICLE_IDLING_TIMEOUT   | 4             | Vehicle idling timeout experied.
REASON_VEHICLE_STUCK_TIMEOUT    | 5             | Vehicle got stuck enroute, such as on a rock or collided with an obstacle and cannot recover. Note: we separate stuck from having flipped over.


### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
received                | bool                  | Indicates the AutoSceneGen client received the AnalyzeScenario request.

# NotifyReady.srv

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------

# RegisterVehicleNode.srv

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------

# RunScenario.srv

### Request
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------

### Response
**Field**               | **Type**              | **Description**
------------------------|-----------------------|----------------
