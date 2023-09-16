# Creating and Working with AutoSceneGenVehicleNodes

Once you know how to [create scenarios](https://github.com/tsender/auto_scene_gen/blob/main/documentation/creating_scenarios.md) using the provided ROS interface, you will then want to create ROS nodes that can control the AutoSceneGenVehicle operating in Unreal Engine. Because we expect that you will be creating many different scenarios, one after the other, and do not want to have to restart your vehicle nodes each time to reset them, we developed a special vehicle node, an AutoSceneGenVehicleNode, that can seamlessly interact with an AutoSceneGenWorker (and its corresponding AutoSceneGenVehicle), and the correpsonding AutoSceneGenClient. To offer the most flexibility, we provide both a Python and a C++ implementation for the AutoSceneGenVehicleNode, and we will provide some templated code demonstrating how to properly work with these classes.

## ROS Objects

Lists any publishers, subscribers, clients, services, and or timers monitored by this node. All instances of `<asg_client_name>` in the below topic names get replaced with the AutoSceneGenClient's name from the `asg_client_name` parameter. All instances of `asg_workerX` are just place holders for the topic substring associated with an AutoSceneGenWorker (e.g., `asg_worker0`, `asg_worker1`, etc.).

**Parameters**
- `vehicle_name`: The name given to the corresponding AutoSceneGenVehicle inside Unreal Engine
- `wid`: The AutoSceneGenWorker ID for the associated AutoSceneGenVehicle
- `asg_client_name`: The name of the AutoSceneGenClient managing the associated worker
- `asg_client_ip_addr`: The IP address of the computer where the AutoSceneGenClient resides. An empty string means the client is on the same machine.
- `ssh_username`: The SSH username for sending files between this computer and the computer where the AutoSceneGenClient lives. An empty string means to use the current username.
- `b_debug_mode`: In case you want to test some of your vehcle's code and want the node to be free from this interface, then set this flag to true. Note: this is still a bit of an experimental feature, and it may get improved or removed in the future.

**Subscribers:**
- Clock Sub
  - Topic: `/clock<wid>` (Here, the `<wid>` gets replaced by the value form the `wid` parameter)
  - Type: `rosgraph_msgs/Clock`
  - Description: Subscribes to the clock topic coming from the ROSIntegration plugin in Unreal Engine where the AutoSceneGenVehicle is
- AutoSceneGenClient Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenClient's status
- AutoSceneGenClient Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenClient's status
- AutoSceneGenClient Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenClient's status
- AutoSceneGenClient Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenClient's status
- AutoSceneGenClient Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the AutoSceneGenClient's status

**Clients:**
- Run Scenario Clients (one for each worker)
  - Topic: `/<asg_workerX>/services/run_scenario`
  - Type: `auto_scene_ge_msgs/RunScenario`
  - Description: Requests a specific scenario for the AutoSceneGenWorker to create and execute
 
**Services:**
- Analyze Scenario Service
  - Topic: `/<asg_client_name>/services/analyze_scenario`
  - Type: `auto_scene_gen_msgs/AnalyzeScenario`
  - Description: Service for accepting `AnalyzeScenario` requests back from the AutoSceneGenWorkers
- Register Vehicle Node Service
  - Topic: `/<asg_client_name>/services/register_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Service for AutoSceneGenVehicleNodes to register themselves with this client node
- Unregister Vehicle Node Service
  - Topic: `/<asg_client_name>/services/unregister_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Service for AutoSceneGenVehicleNodes to unregister themselves with this client node
- Notify Ready Service
  - Topic: `/<asg_client_name>/services/notify_ready`
  - Type: `auto_scene_gen_msgs/NotifyReady`
  - Description: Service for AutoSceneGenVehicleNodes to indicate to this client node that they are ready to proceed
- Worker Issue Notification Service
  - Topic: `/<asg_client_name>/services/worker_issue_notification`
  - Type: `auto_scene_gen_msgs/WorkerIssueNotification`
  - Description: Service for AutoSceneGenWorkers to indicate to this client node of any issues they encountered (e.g., rosbrige interruption)
 
**Timers**
- Main Loop Timer
  - Timer Callback: `main_loop_timer_cb`
  - Description: This is the main loop that runs until the node is destroyed. When it is running, it will publish the online status for the client, publish any vehicle node operating info for any AutoSceneGenVehicleNodes, publish the current scene description being ran in the managed AutoSceneGenWorkers, check/ensure the AutoSceneGenWorkers received their latest `RunScenario` request, and then call `main_step(wid: int)` and passes in the current worker ID being processed. The `main_step` function is what you will need to override and is your primary entry point for creating and analyzing scenarios. The `main_loop_timer_cb` function will cycle through all managed worker IDs so you don't have to.
