# Creating and Working with AutoSceneGenVehicleNodes

Quick Links:
- [Home Page](https://github.com/tsender/auto_scene_gen)
- [Message and Service Definitions](msg_and_srv_reference.md)
- [Creating and Running Scenarios](creating_scenarios.md)

Once you know how to [create scenarios](https://github.com/tsender/auto_scene_gen/blob/main/documentation/creating_scenarios.md) using the provided ROS interface, you will then want to create ROS nodes that can control the AutoSceneGenVehicle operating in Unreal Engine. Because we expect that you will be creating many different scenarios, one after the other, and do not want to have to restart your vehicle nodes each time to reset them, we developed a special vehicle node, called an AutoSceneGenVehicleNode, that can seamlessly interact with an AutoSceneGenVehicle, the corresponding AutoSceneGenWorker, and the correpsonding AutoSceneGenClient. To offer the most flexibility, we provide both a Python and a C++ implementation for the AutoSceneGenVehicleNode, and we will provide some templated code demonstrating how to properly work with these classes.

## Abbreviations

Instead of always writing the prefix "AutoSceneGen", we will often use a shorthand when referring to the various components in the AutoSceneGen ecosystem. The shorthand only applies if it makes based on the context.
- Client: Refers to an AutoSceneGenClient, which is a Python ROS node that determines the scenarios one or more AutoSceneGenWorkers should run.
- Worker: Refers to an instance of an AutoSceneGenWorker, which resides in Unreal Engine.
- Vehicle: Refers to an instance of an AutoSceneGenVehicle, which resides in Unreal Engine.
- Vehicle Node: Refers to an instance of an AutoSceneGenVehicleNode, which is a Python or C++ ROS node within the autonomy stack controlling an AutoSceneGenVehicle.

## ROS Objects

Lists any publishers, subscribers, clients, services, and or timers monitored by this node. All instances of `<asg_client_name>` in the below topic names get replaced with the AutoSceneGenClient's name from the `asg_client_name` parameter. All instances of `<wid>` get replaced by the ID from the `wid` parameter. All instances of `<vehicle_name>` get replaced with the name from the `vehicle_name` parameter.

**Parameters**
- `vehicle_name`: The name given to the corresponding AutoSceneGenVehicle inside Unreal Engine
- `wid`: The AutoSceneGenWorker ID for the associated AutoSceneGenVehicle
- `asg_client_name`: The name of the AutoSceneGenClient managing the associated worker
- `asg_client_ip_addr`: The IP address of the computer where the AutoSceneGenClient resides. Leave empty if the client is on the same machine.
- `ssh_username`: The SSH username for sending files between this computer and the computer where the AutoSceneGenClient lives. If empty, then we will use the current username.
- `b_debug_mode`: In case you want to test some of your vehcle's code and want the node to be free from this interface, then set this flag to true. Note: this is still a bit of an experimental feature, I have barely tested/used it, and it may get improved or removed in the future.

**Subscribers:**
- Clock Sub
  - Topic: `/clock<wid>`
  - Type: `rosgraph_msgs/Clock`
  - Description: Subscribes to the clock topic coming from the ROSIntegration plugin in Unreal Engine where the vehicle is.
- Client Status Sub
  - Topic: `/<asg_client_name>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the client's status.
- Worker Status Sub
  - Topic: `/asg_worker<wid>/status`
  - Type: `auto_scene_gen_msgs/StatusCode`
  - Description: Subscribes to the worker's status.
- Vehicle Status Sub
  - Topic: `/<asg_client_name>/<vehicle_name>/status`
  - Type: `auto_scene_gen_msgs/VehicleStatus`
  - Description: Subscribes to the vehicle's status.
- Vehicle Node Operating Info Sub
  - Topic: `/<asg_client_name>/vehicle_node_operating_info`
  - Type: `auto_scene_gen_msgs/VehicleNodeOperatingInfo`
  - Description: Subscribes to the vehicle node operating info published by the client.
- Scene Description Sub
  - Topic: `/asg_worker<wid>/scene_description`
  - Type: `auto_scene_gen_msgs/SceneDescription`
  - Description: Subscribes to the current scene description being ran on the worker.
 
**Clients:**
- Register Node Client
  - Topic: `/<asg_client_name>/services/register_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Client for registering with the client.
- Unregister Node Client
  - Topic: `/<asg_client_name>/services/unregister_vehicle_node`
  - Type: `auto_scene_gen_msgs/RegisterVehicleNode`
  - Description: Client for unregistering with the client.
- Notify Ready Client
  - Topic: `/<asg_client_name>/services/notify_ready`
  - Type: `auto_scene_gen_msgs/NotifyReady`
  - Description: Client for notifying to the client that this vehicle node is ready. Techncally, we send service requests, but we treat these requests as notifications.
 
**Timers**
- Register Node Timer
  - Timer Callback: `register_node_timer_cb`
  - Description: This callback runs every second and ensures that the vehicle node is registered with the client (once it comes online) and then makes sure that its `NotifyReady` requests are received.

## General Node Workflow

Assuming that the `b_debug_mode` parameter is set to false, then the veicle node's operation will be as follows:
1. Load ROS parameters and create ROS objects (see above).
2. Wait for the client to come online.
3. Register with the client.
4. Send a `NotifyReady` request to the client and verify it was received.
5. Follow any custom callbacks that you created and function like a normal node (see ).
6. Perform a reset procedure when the vehicle is disabled (see below).
7. Go back to step 4 and repeat until the node is destroyed.
8. Prior to getting destroyed, perform a shutdown procedure.

### Customizing the Node

The `AutoSceneGenVehicleNode` is the base class from which all of your ROS nodes will inherit from because this node abstracts away all of the overhead needed to interactwith the entire AutoSceneGen ecosystem. For the most part, you can treat your child classes like regular ROS nodes - you can add more parameters in the constructor and create however many callbacks you need in each node.

Since this entire platform is built on creating an automated system in which scenarios can be created/executed repeatedly without forcing the user to restart Unreal Engine or their ROS nodes (unless a critical problem arises), part of the overhead embedded in this interfaces does result in some minor modifications to your ROS nodes. The following codeblock must be placed at the very top of *every* callback to ensure your code does not execute prematurely:

Python:
```
if not self.vehicle_ok():
    return
```

C++:
```
if (!vehicle_ok())
    return;
```

### Logging

The `AutoSceneGenVehicleNode` class provides a logging function that makes it easier to log information to the console and to a separate log file. The log message will always be logged via the ROS logger and by default will be written to the log file. All messages are prepended with a date, timestamp, and log level, for example `[2023-09-17 23:30:09.599643] [INFO]`. See the `log` function for more details. The base class will create a temporary log file and it will automatically be saved to the save directory on the client's computer.

### Saving Node Data

In a number of situations, you will want your vehicle nodes to save their data so you can later analyze it, create plots/figures, etc. The `AutoSceneGenVehicleNode` class provides a `save_node_data` function that automatically gets called during the reset process (see below) allowing you to save any data before the node's internal state is reset. The `AutoSceneGenVehicleNode` class also contains three variables that will be needed when saving your data. Of these variables, `save_dir` and `b_save_minimal` are set by the client.
- `save_dir`: This string denotes the directory on the client's computer where all of this node's data will be saved to.
- `temp_save_dir`: This string denotes a temporary directory on the vehicle node's computer where it can save data to temporarily before copying it to the `save_dir` directory on the client computer. This will come in handy when the vehicle node does not reside on the same computer as the client.
- `b_save_minimal`: This boolean indicates how much data the vehicle node should save. Even though you may wish to save as much data as possible, saving lage data files consumes a lot of time and creates a bottleneck as it holds up the client from running another scenario until the node finihes saving all of its data. Hence, we provide the option (if you choose to use it) to let you specify how much data you wish to save. If this value is true, then the node should only save the least amount of data necessry for your application. If the value is false, then it can save as much data as you need.

For Python vehicle nodes, you may find the following functions to be helpful:
- `save_file_on_remote_asg_client`
- `copy_file_from_remote_asg_client`
- `save_pickle_object_on_asg_client`
- `load_pickle_object_from_asg_client`

For C++ vehicle nodes, you may find the following functions to be helpful:
- `save_file_on_remote_asg_client`

### Requesting a Rerun

Since no simulation is perfect, it may be useful to rerun a scenario if you are able to detect if any problems that occurred that would not normally occur in the real vehicle/system. For example, even though ROS2's QoS feature allows you to configure the communication quality between nodes, due to the ROS pipeline between Unreal Engine and rosbridge, there may be added latency or hiccups that prevent things from running smoothly. The `check_for_rerun_request` function is built into the reset process (see below) and it provides you with the opportunity to internally monitor various things and request a rerun as desired

To use this feature, first override the `check_for_rerun_request` function with any verification checks you wish to perform. Then, if a check fails, do the following to request a rerun:
1. Set the `request_rerun` field in the `NotifyReady` request to true.
2. Set the `reason_for_rerun` field in the `NotifyReady` request field to explain the reason for the rerun request.

The node will automatically reset these two fields after submitting the `NotifyReady` request.

### The Reset Procedure

Once the simulation terminates, all of the corresponding vehicle nodes must reset their internal state so they can be ready for the next simulation. Internally, the vehicle nodes monitor the vehicle's "OK status". The OK status returns true if all of the following conditions are true:
- The vehicle node is registered with the client
- The vehicle node's most recent `NotifyReady` request was proceesed by the client
- The vehicle in Unreal Engine is enabled
- The client has informed the vehicle nodes that is okay for them to run

Once the OK status changes to false, the reset procedure is triggered (all of this takes place inside the vehicle status callback):
1. The vehicle disabled timestamp is recorded.
2. If the incoming vehicle status message was not preempted (i.e, the simulation was not aborted due to a detected issue), then:
   - The `save_node_data` function will be called allowing the node to save any internal data to a save directory on the computer where the client resides.
   - The function `check_for_rerun_request` will be called allowing the node to request a rerun of the last scenario if it detected a potential problem that should not have occurred (this is left to the user to decide).
   - The vehicle node's log file will get saved to the save directory on the computer where the client resides.
3. The vehicle node's temporary save directory will be reset.
4. The `reset` function will be called in which the vehicle node should reset all internal variables that need to be reset.
5. The vehicle node will send a `NotifyReady` request to the client informing that it is ready for the next scenario.

### Shutdown Procedure

Before a vehicle node is destroyed, you need to call the `shutdown` function on the node to unregister it from the client. The procedure to [spin a vehicle node](#spinning-a-vehicle-node) will show you how to spin the node and make sure it gets shutdow properly.

## Clock Time

AutoSceneGenVehicleNodes do not use the built-in clock that comes with ROS nodes because it is expected that you may be running multiple Unreal simulations in parallel. Although ROS nodes let you control the ROS clock time via external time source, they only allow you to publish on the `/clock` topic. Running multiple simulations in parallel requires multiple independent clock topics, and so we must instead use our own version of this time source mechanism. The ROSIntegration fork required by this platform provides an extra parameter allowing you to specify the clock topic. You will need to set the topic name to `/clock<wid>`, where `<wid>` is replaced by the ID of the worker running in the Unreal simulation.

Both the Python and C++ `AutoSceneGenVehicleNode` classes provide two class variables for access to a system clock `system_clock` and the simulation clock `sim_clock`. Assuming the clock topics are configured correctly, then the base class will automatically align the node's simulation clock with the time coming from the simulation. 

**IMPORTANT**: Whenever you create timers, you will need to pass along the appropriate clock, `system_clock` or `sim_clock`, so your timers run correctly (most of the time you will likely just be using the `sim_clock`).

## Spinning a Vehicle Node

Both Python and C++ versions of the `AutoSceneGenVehicleNode` classes provides a helper function for spinning a vehicle node called `spin_vehicle_node`. Due to the need for keeping the sim clock as up-to-date as ppssible, this requires that all vehicle nodes use a multithreaded executor. We can configure how many threads we want the executor to use when calling `spin_vehicle_node`. For demonstration purposes. let's say we created a custom vehicle node class called `MyVehicleNode` in the package `my_package`.

Here is the Python version of the `main` function for creating and spinning our vehicle node:
```
from my_package.my_vehicle_node import MyVehicleNode
from auto_scene_gen_core.vehicle_node import spin_vehicle_node

def main(args=None):
    # Code to initalize any parameters for our vehicle node

    rclpy.init(args=args)
    node = MyVehicleNode("my_vehicle_node", <other_parameters>)
    spin_vehicle_node(node, num_threads=2)
```

Here is the C++ version of the main function for creating and spinning our vehicle node. Note, unlike in Python, a try-catch block in C++ cannot catch any events that a signal handler would catch, such as a signal interrupt (ctrl-c). To ensure our vehicle node shuts down properly, we must declare our vehicle node as a global variable in the executable file and configure a custom `signal_handler` function. I do not know of any other way to shutdown the node properly while accounting for signal handling.
```
#include "rclcpp/rclcpp.hpp"
#include <csignal>
#include "auto_scene_gen_core/vehicle_node.hpp"
#include "my_package/my_vehicle_node.hpp"

// Needs to be a global variable so we can use with signal handling
std::shared_ptr<MyVehicleNode> node;

void signal_handler(int signal_num)
{
    if (signal_num == SIGINT)
        node->log(MyVehicleNode::LOG_INFO, "Keyboard interrupt. Shutting down...");
    else
        node->log(MyVehicleNode::LOG_ERROR, "Signal handler. Value = " + std::to_string(signal_num));
    node->shutdown();
    rclcpp::shutdown();
    exit(signal_num);
}

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);

    // Manually make signal handler so we can perform desired logging
    rclcpp::uninstall_signal_handlers();
    struct sigaction sig_action;
    sigemptyset(&sig_action.sa_mask);
    sig_action.sa_handler = signal_handler;
    sig_action.sa_flags = 0;
    sigaction(SIGINT, &sig_action, nullptr);
    sigaction(SIGFPE, &sig_action, nullptr);
    sigaction(SIGBUS, &sig_action, nullptr);
    sigaction(SIGABRT, &sig_action, nullptr);
    sigaction(SIGSEGV, &sig_action, nullptr);

    // Code to initialize any parameters for our vehicle node

    node = std::make_shared<MyVehicleNode>("my_vehicle_node", <other_parameters>);
    auto_scene_gen_core::spin_vehicle_node(node, 2);
    return 0;
}
```

## Compiling a C++ AutoSceneGenVehicleNode

When creating a C++ subclass of `AutoSceneGenVehicleNode`, your CMakeLists.txt file should have the following `find_package` commands:
```
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(auto_scene_gen_msgs REQUIRED)
find_package(auto_scene_gen_core REQUIRED)
find_package(Boost COMPONENTS system filesystem REQUIRED) # This is needed since auto_scene_gen_core cannot export them for us
```
The last line is critical. Unfortunatey, I was unable to get the `auto_scene_gen_core` package to export the two boost libraries. Consequently, every `ament_cmake` package that depends on `auto_scene_gen_core` muust find those two boost libraries for that package to compile.

Fortunately, creating your executable does not require linking against boost. You can create an executable as expected with the lines
```
add_executable(test_vehicle_node src/test_vehicle_node.cpp)
ament_target_dependencies(test_vehicle_node rclcpp auto_scene_gen_msgs auto_scene_gen_core)
```
