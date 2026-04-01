.. redirect-from::

    Tutorials/Understanding-ROS2-Nodes

.. _ROS2Nodes:

Understanding nodes
===================

**目标：** 了解 ROS 2 中节点的功能以及与节点交互的工具。

**教程级别：** 初级

**时间：** 10 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

1 ROS 2 图
^^^^^^^^^^

在接下来的几篇教程中，您将学习一系列构成所谓"ROS (2) 图"的核心 ROS 2 概念。

ROS 图是一个同时处理数据的 ROS 2 元素网络。
如果您将所有可执行文件及其之间的连接映射出来并进行可视化，它将包含所有可执行文件及其之间的连接。

2 ROS 2 中的节点
^^^^^^^^^^^^^^^^

ROS 中的每个节点都应该负责单一、模块化的目的，例如控制轮电机或发布来自激光测距仪的传感器数据。
每个节点都可以通过主题、服务、动作或参数与其他节点发送和接收数据。

.. image:: images/Nodes-TopicandService.gif

一个完整的机器人系统由许多协同工作的节点组成。
在 ROS 2 中，单个可执行文件（C++ 程序、Python 程序等）可以包含一个或多个节点。

前提条件
---------

:doc:`之前的教程 <../Introducing-Turtlesim/Introducing-Turtlesim>` 向您展示了如何安装这里使用的 ``turtlesim`` 软件包。

一如既往，不要忘记在 :doc:`打开的每个新终端 <../Configuring-ROS2-Environment>` 中 source ROS 2。

任务
----

1 ros2 run
^^^^^^^^^^

``ros2 run`` 命令从软件包启动可执行文件。

.. code-block:: console

  $ ros2 run <package_name> <executable_name>

要运行 turtlesim，请打开一个新终端，并输入以下命令：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

turtlesim 窗口将打开，正如您在 :doc:`上一篇教程 <../Introducing-Turtlesim/Introducing-Turtlesim>` 中看到的那样。

这里，软件包名称是 ``turtlesim``，可执行文件名称是 ``turtlesim_node``。

但是，我们还不知道节点名称。
您可以使用 ``ros2 node list`` 来查找节点名称

2 ros2 node list
^^^^^^^^^^^^^^^^

``ros2 node list`` 将显示所有正在运行的节点的名称。
当您想要与节点交互，或者当您的系统运行许多节点并需要跟踪它们时，这特别有用。

在 turtlesim 仍在另一个终端中运行时打开一个新终端，并输入以下命令。
终端将返回节点名称：

.. code-block:: console

  $ ros2 node list
  /turtlesim

打开另一个新终端并使用以下命令启动 teleop 节点：

.. code-block:: console

  $ ros2 run turtlesim turtle_teleop_key

这里，我们再次引用 ``turtlesim`` 软件包，但这次我们的目标是名为 ``turtle_teleop_key`` 的可执行文件。

返回运行 ``ros2 node list`` 的终端并再次运行它。
您现在将看到两个活动节点的名称：

.. code-block:: console

  $ ros2 node list
  /turtlesim
  /teleop_turtle

2.1 重映射
~~~~~~~~~~

`重映射 <https://design.ros2.org/articles/ros_command_line_arguments.html#name-remapping-rules>`__ 允许您将默认节点属性（如节点名称、主题名称、服务名称等）重新分配为自定义值。
在上一篇教程中，您在 ``turtle_teleop_key`` 上使用了重映射来更改 cmd_vel 主题并定位到 **turtle2**。

现在，让我们重新分配 ``/turtlesim`` 节点的名称。
在一个新终端中，运行以下命令：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node --ros-args --remap __node:=my_turtle

由于您再次在 turtlesim 上调用 ``ros2 run``，另一个 turtlesim 窗口将打开。
但是，现在如果您返回运行 ``ros2 node list`` 的终端，并再次运行它，您将看到三个节点名称：

.. code-block:: console

    /my_turtle
    /turtlesim
    /teleop_turtle

3 ros2 node info
^^^^^^^^^^^^^^^^

既然您知道了节点的名称，您可以使用以下命令访问有关它们的更多信息：

.. code-block:: console

  $ ros2 node info <node_name>

要检查您最新的节点 ``my_turtle``，运行以下命令：

.. code-block:: console

  $ ros2 node info /my_turtle
  /my_turtle
    Subscribers:
      /parameter_events: rcl_interfaces/msg/ParameterEvent
      /turtle1/cmd_vel: geometry_msgs/msg/Twist
    Publishers:
      /parameter_events: rcl_interfaces/msg/ParameterEvent
      /rosout: rcl_interfaces/msg/Log
      /turtle1/color_sensor: turtlesim_msgs/msg/Color
      /turtle1/pose: turtlesim_msgs/msg/Pose
    Service Servers:
      /clear: std_srvs/srv/Empty
      /kill: turtlesim_msgs/srv/Kill
      /my_turtle/describe_parameters: rcl_interfaces/srv/DescribeParameters
      /my_turtle/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
      /my_turtle/get_parameters: rcl_interfaces/srv/GetParameters
      /my_turtle/list_parameters: rcl_interfaces/srv/ListParameters
      /my_turtle/set_parameters: rcl_interfaces/srv/SetParameters
      /my_turtle/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
      /reset: std_srvs/srv/Empty
      /spawn: turtlesim_msgs/srv/Spawn
      /turtle1/set_pen: turtlesim_msgs/srv/SetPen
      /turtle1/teleport_absolute: turtlesim_msgs/srv/TeleportAbsolute
      /turtle1/teleport_relative: turtlesim_msgs/srv/TeleportRelative
    Service Clients:

    Action Servers:
      /turtle1/rotate_absolute: turtlesim_msgs/action/RotateAbsolute
    Action Clients:

``ros2 node info`` 返回订阅者、发布者、服务和动作的列表。
即与该节点交互的 ROS 图连接。

现在尝试在 ``/teleop_turtle`` 节点上运行相同的命令，看看它的连接与 ``my_turtle`` 有何不同。

您将在接下来的教程中了解更多关于 ROS 图连接概念，包括消息类型。

总结
----

节点是 ROS 2 的基本元素，在机器人系统中服务于单一、模块化的目的。

在本教程中，您通过运行可执行文件 ``turtlesim_node`` 和 ``turtle_teleop_key`` 来使用 ``turtlesim`` 软件包中创建的节点。

您学习了如何使用 ``ros2 node list`` 发现活动节点名称，以及如何使用 ``ros2 node info`` 检查单个节点。
这些工具对于理解复杂、真实机器人系统中的数据流至关重要。

下一步
------

现在您已经了解了 ROS 2 中的节点，您可以继续学习 :doc:`主题教程 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>`。
主题是连接节点的通信类型之一。

相关内容
--------

:doc:`../../../Concepts` 页面为节点的概念添加了更多细节。
