# Creating and Working with AutoSceneGenVehicleNodes

Once you know how to [create scenarios](https://github.com/tsender/auto_scene_gen/blob/main/documentation/creating_scenarios.md) using the provided ROS interface, you will then want to create ROS nodes that can control the AutoSceneGenVehicle operating in Unreal Engine. Because we expect that you will be creating many different scenarios, one after the other, and do not want to have to restart your vehicle nodes each time to reset them, we developed a special vehicle node, an AutoSceneGenVehicleNode, that can seamlessly interact with an AutoSceneGenWorker (and its corresponding AutoSceneGenVehicle), and the correpsonding AutoSceneGenClient. To offer the most flexibility, we provide both a Python and a C++ implementation for the AutoSceneGenVehicleNode, and we will provide some templated code demonstrating how to properly work with these classes.

## ROS Objects

Lists any publishers, subscribers, clients, services, and or timers monitored by this node. All instances of `<asg_client_name>` in the below topic names get replaced with the AutoSceneGenClient's name from the `asg_client_name` parameter. All instances of `<wid>` get replaced by the ID from the `wid` parameter. All instances of `<vehicle_name>` get replaced with the name from the `vehicle_name` parameter.

**Parameters**
- `vehicle_name`: The name given to the corresponding AutoSceneGenVehicle inside Unreal Engine
- `wid`: The AutoSceneGenWorker ID for the associated AutoSceneGenVehicle
- `asg_client_name`: The name of the AutoSceneGenClient managing the associated worker
- `asg_client_ip_addr`: The IP address of the computer where the AutoSceneGenClient resides. Leave empty if the client is on the same machine.
- `ssh_username`: The SSH username for sending files between this computer and the computer where the AutoSceneGenClient lives. If empty, then we will use the current username.
- `b_debug_mode`: In case you want to test some of your vehcle's code and want the node to be free from this interface, then set this flag to true. Note: this is still a bit of an experimental feature, and it may get improved or removed in the future.

**Subscribers:**
- Clock Sub
  - Topic: `/clock<wid>`
  - Type: `rosgraph_msgs/Clock`
  - Description: Subscribes to the clock topic coming from the ROSIntegration plugin in Unreal Engine where the AutoSceneGenVehicle is
- Client Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenClient's status
- Worker Status Sub
  - Topic: `/asg_worker<wid>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenWorker's status
- Vehicle Status Sub
  - Topic: `/<asg_client_name>/<vehicle_name>/status`
  - Type: `auto_scene_gen_msgs/VehicleStatus`
  - Description: Subscribes to the AutoSceneGenVehicle's status
- Vehicle Node Operating Info Sub
  - Topic: `/<asg_client_name>/vehicle_node_operating_info`
  - Type: `auto_scene_gen_msgs/VehicleNodeOperatingInfo`
  - Description: Subscribes to the vehicle node operating info published by the AutoSceneGenClient
- Scene Description Sub
  - Topic: `/asg_worker<wid>/scene_description`
  - Type: `auto_scene_gen_msgs/SceneDescription`
  - Description: Subscribes to the current scene description being ran on the AutoSceneGenWorker
 
**Clients:**
- Register Node Client
  - Topic: `/<asg_client_name>/services/register_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Client for registering with the AutoSceneGenClient
- Unregister Node Client
  - Topic: `/<asg_client_name>/services/unregister_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Client for unregistering with the AutoSceneGenClient
- Notify Ready Client
  - Topic: `/<asg_client_name>/services/notify_ready`
  - Type: `auto_scene_gen_msgs/NotifyReady`
  - Description: Client for notifying to the AutoSceneGenClient that this vehicle node is ready. Techncally, we send service requests, but we treat these requests as notifications.
 
**Timers**
- Register Node Timer
  - Timer Callback: `register_node_timer_cb`
  - Description: This callback runs every second and ensures that the vehicle node is registered with the AutoSceneGenClient (once it comes online) and then makes sure that its `NotifyReady` requests are received.

### General Node Workflow

Assuming that the `b_debug_mode` parameter is set to false, then the veicle node's operation will be as follows:
1. Load ROS parameters and create ROS objects (see above).
2. Wait for the AutoSceneGenClient to come online.
3. Register itself with the AutoSceneGenClient.
4. Send an initial `NotifyReady` request to the AutoSceneGenClient and verify it was received.
5. Follow any custom callbacks that you created and function like a normal node.
6. When the AutoSceneGenVehicle

### Customizing the Node


### Logging


### Clock Time
