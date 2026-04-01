.. redirect-from::

    Tutorials/Understanding-ROS2-Actions

.. _ROS2Actions:

理解动作
=========

**目标：** 检查 ROS 2 中的动作。

**教程级别：** 初级

**时间：** 15 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

动作是 ROS 2 中的通信类型之一，旨在用于长时间运行的任务。
它们由三部分组成：目标、反馈和结果。

动作建立在主题和服务之上。
它们的功能类似于服务，不同之处在于动作可以被取消。
它们还提供稳定的反馈，而不是返回单个响应的服务。

动作使用客户端-服务端模型，类似于发布者-订阅者模型（在 :doc:`主题教程 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>` 中描述）。
"动作客户端"节点向"动作服务端"节点发送目标，该节点确认目标并返回反馈流和结果。

.. image:: images/Action-SingleActionClient.gif

前提条件
---------

本教程建立在之前教程中介绍的概念之上，如 :doc:`节点 <../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>` 和 :doc:`主题 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>`。

本教程使用 :doc:`turtlesim 软件包 <../Introducing-Turtlesim/Introducing-Turtlesim>`。

一如既往，不要忘记在 :doc:`打开的每个新终端 <../Configuring-ROS2-Environment>` 中 source ROS 2。

任务
----

1 设置
^^^^^^

启动两个 turtlesim 节点，``/turtlesim`` 和 ``/teleop_turtle``。

打开一个新终端并运行：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

打开另一个终端并运行：

.. code-block:: console

  $ ros2 run turtlesim turtle_teleop_key


2 使用动作
^^^^^^^^^^

当您启动 ``/teleop_turtle`` 节点时，您将在终端中看到以下消息：

.. code-block:: console

    Use arrow keys to move the turtle.
    Use G|B|V|C|D|E|R|T keys to rotate to absolute orientations. 'F' to cancel a rotation.

让我们关注第二行，它对应一个动作。
（第一条指令对应 "cmd_vel" 主题，之前在 :doc:`主题教程 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>` 中讨论过。）

请注意，字母键 ``G|B|V|C|D|E|R|T`` 在美式 QWERTY 键盘上围绕 ``F`` 键形成一个"框"（如果您使用的不是 QWERTY 键盘，请参阅 `此链接 <https://upload.wikimedia.org/wikipedia/commons/d/da/KB_United_States.svg>`__ 来跟随操作）。
每个键在 ``F`` 周围的位置对应于 turtlesim 中的那个方向。
例如，``E`` 将把海龟的方向旋转到左上角。

注意运行 ``/turtlesim`` 节点的终端。
每次按下其中一个键时，您都在向 ``/turtlesim`` 节点中的动作服务端发送目标。
目标是让海龟旋转到面向特定方向。
一旦海龟完成旋转，应该显示一条传递目标结果的消息：

.. code-block:: console

    [INFO] [turtlesim]: Rotation goal completed successfully

``F`` 键将在执行过程中取消目标。

尝试按下 ``C`` 键，然后在海龟完成旋转之前按下 ``F`` 键。
在运行 ``/turtlesim`` 节点的终端中，您将看到消息：

.. code-block:: console

  [INFO] [turtlesim]: Rotation goal canceled

不仅客户端（您在 teleop 中的输入）可以停止目标，服务端（``/turtlesim`` 节点）也可以。
当服务端选择停止处理目标时，称为"中止"目标。

尝试按下 ``D`` 键，然后在第一次旋转完成之前按下 ``G`` 键。
在运行 ``/turtlesim`` 节点的终端中，您将看到消息：

.. code-block:: console

  [WARN] [turtlesim]: Rotation goal received before a previous goal finished. Aborting previous goal

此动作服务端选择中止第一个目标，因为它收到了新目标。
它可以选择其他方式，例如拒绝新目标或在第一个目标完成后执行第二个目标。
不要假设每个动作服务端在收到新目标时都会选择中止当前目标。

3 ros2 node info
^^^^^^^^^^^^^^^^

要查看节点提供的动作列表（本例中为 ``/turtlesim``），打开一个新终端并运行命令：

.. code-block:: console

  $ ros2 node info /turtlesim
  /turtlesim
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
      /reset: std_srvs/srv/Empty
      /spawn: turtlesim_msgs/srv/Spawn
      /turtle1/set_pen: turtlesim_msgs/srv/SetPen
      /turtle1/teleport_absolute: turtlesim_msgs/srv/TeleportAbsolute
      /turtle1/teleport_relative: turtlesim_msgs/srv/TeleportRelative
      /turtlesim/describe_parameters: rcl_interfaces/srv/DescribeParameters
      /turtlesim/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
      /turtlesim/get_parameters: rcl_interfaces/srv/GetParameters
      /turtlesim/list_parameters: rcl_interfaces/srv/ListParameters
      /turtlesim/set_parameters: rcl_interfaces/srv/SetParameters
      /turtlesim/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
    Service Clients:

    Action Servers:
      /turtle1/rotate_absolute: turtlesim_msgs/action/RotateAbsolute
    Action Clients:

该命令返回 ``/turtlesim`` 的订阅者、发布者、服务、动作服务端和动作客户端列表。

请注意，``/turtlesim`` 的 ``/turtle1/rotate_absolute`` 动作位于 ``Action Servers`` 下。
这意味着 ``/turtlesim`` 响应 ``/turtle1/rotate_absolute`` 动作并提供反馈。

``/teleop_turtle`` 节点在 ``Action Clients`` 下有名称 ``/turtle1/rotate_absolute``，这意味着它为该动作名称发送目标。
要查看这一点，请运行：

.. code-block:: console

  $ ros2 node info /teleop_turtle
  /teleop_turtle
    Subscribers:
      /parameter_events: rcl_interfaces/msg/ParameterEvent
    Publishers:
      /parameter_events: rcl_interfaces/msg/ParameterEvent
      /rosout: rcl_interfaces/msg/Log
      /turtle1/cmd_vel: geometry_msgs/msg/Twist
    Service Servers:
      /teleop_turtle/describe_parameters: rcl_interfaces/srv/DescribeParameters
      /teleop_turtle/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
      /teleop_turtle/get_parameters: rcl_interfaces/srv/GetParameters
      /teleop_turtle/list_parameters: rcl_interfaces/srv/ListParameters
      /teleop_turtle/set_parameters: rcl_interfaces/srv/SetParameters
      /teleop_turtle/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
    Service Clients:

    Action Servers:

    Action Clients:
      /turtle1/rotate_absolute: turtlesim_msgs/action/RotateAbsolute

4 ros2 action list
^^^^^^^^^^^^^^^^^^

要识别 ROS 图中的所有动作，运行命令：

.. code-block:: console

  $ ros2 action list
  /turtle1/rotate_absolute

这是 ROS 图中目前唯一的动作。
它控制海龟的旋转，正如您之前看到的。
您还已经通过使用 ``ros2 node info <node_name>`` 命令知道此动作有一个动作客户端（``/teleop_turtle`` 的一部分）和一个动作服务端（``/turtlesim`` 的一部分）。

4.1 ros2 action list -t
~~~~~~~~~~~~~~~~~~~~~~~

动作有类型，类似于主题和服务。
要查找 ``/turtle1/rotate_absolute`` 的类型，运行命令：

.. code-block:: console

  $ ros2 action list -t
  /turtle1/rotate_absolute [turtlesim_msgs/action/RotateAbsolute]

每个动作名称右侧的括号中（本例中只有 ``/turtle1/rotate_absolute``）是动作类型 ``turtlesim_msgs/action/RotateAbsolute``。
当您想从命令行或代码执行动作时，您需要这个信息。

5 ros2 action type
^^^^^^^^^^^^^^^^^^

如果您想检查动作的动作类型，运行命令：

.. code-block:: console

  $ ros2 action type /turtle1/rotate_absolute
  turtlesim_msgs/action/RotateAbsolute

6 ros2 action info
^^^^^^^^^^^^^^^^^^

您可以使用以下命令进一步检查 ``/turtle1/rotate_absolute`` 动作：

.. code-block:: console

  $ ros2 action info /turtle1/rotate_absolute
  Action: /turtle1/rotate_absolute
  Action clients: 1
      /teleop_turtle
  Action servers: 1
      /turtlesim

这告诉了我们之前通过在每个节点上运行 ``ros2 node info`` 学到的内容：
``/teleop_turtle`` 节点有一个动作客户端，``/turtlesim`` 节点有一个 ``/turtle1/rotate_absolute`` 动作的动作服务端。

7 ros2 interface show
^^^^^^^^^^^^^^^^^^^^^

在自行发送或执行动作目标之前，您需要的另一个信息是动作类型的结构。

回想一下，您在运行命令 ``ros2 action list -t`` 时确定了 ``/turtle1/rotate_absolute`` 的类型。
在终端中输入以下命令，带上动作类型：

.. code-block:: console

  $ ros2 interface show turtlesim_msgs/action/RotateAbsolute

这将返回：

.. code-block:: text

  # The desired heading in radians
  float32 theta
  ---
  # The angular displacement in radians to the starting position
  float32 delta
  ---
  # The remaining rotation in radians
  float32 remaining

第一个 ``---`` 上方的消息部分是目标请求的结构（数据类型和名称）。
下一部分是结果的结构。
最后一部分是反馈的结构。

8 ros2 action send_goal
^^^^^^^^^^^^^^^^^^^^^^^

现在让我们使用以下语法从命令行发送动作目标：

.. code-block:: console

  $ ros2 action send_goal <action_name> <action_type> <values>

``<values>`` 需要是 YAML 格式。

注意 turtlesim 窗口，并在终端中输入以下命令：

.. code-block:: console

  $ ros2 action send_goal /turtle1/rotate_absolute turtlesim_msgs/action/RotateAbsolute "{theta: 1.57}"
  Waiting for an action server to become available...
  Sending goal:
     theta: 1.57

  Goal accepted with ID: f8db8f44410849eaa93d3feb747dd444

  Result:
    delta: -1.568000316619873

  Goal finished with status: SUCCEEDED


您应该看到海龟正在旋转。


所有目标都有一个唯一 ID，显示在返回消息中。
您还可以看到结果，一个名为 ``delta`` 的字段，它是到起始位置的位移。

要查看此目标的反馈，请在 ``ros2 action send_goal`` 命令中添加 ``--feedback``：

.. code-block:: console

  $ ros2 action send_goal /turtle1/rotate_absolute turtlesim_msgs/action/RotateAbsolute "{theta: -1.57}" --feedback
  Sending goal:
     theta: -1.57

  Goal accepted with ID: e6092c831f994afda92f0086f220da27

  Feedback:
    remaining: -3.1268222332000732

  Feedback:
    remaining: -3.1108222007751465

  …

  Result:
    delta: 3.1200008392333984

  Goal finished with status: SUCCEEDED

您将继续收到反馈，即剩余弧度，直到目标完成。

.. _understanding-actions-ros2-action-echo:

9 ros2 action echo
^^^^^^^^^^^^^^^^^^

要查看动作客户端和动作服务端之间的数据通信，您可以使用以下命令 ``echo`` 动作数据：

.. code-block:: console

  $ ros2 action echo <action_name> <optional arguments/action_type>

``ros2 action echo`` 依赖于动作客户端和服务端的动作内省，默认情况下是禁用的。
要启用它，用户必须在创建动作客户端或服务端后调用 ``configure_introspection``。

启动 ``fibonacci_action_server`` 和 ``fibonacci_action_client``，启用 ``action_server_configure_introspection`` 参数进行演示。

.. code-block:: console

  $ ros2 run action_tutorials_cpp fibonacci_action_server --ros-args -p action_server_configure_introspection:=contents

.. code-block:: console

  $ ros2 run action_tutorials_py fibonacci_action_client --ros-args -p action_client_configure_introspection:=contents

现在我们可以通过 ``ros2 action echo`` 查看 ``fibonacci_action_server`` 和 ``fibonacci_action_client`` 之间的动作通信。

.. code-block:: console

   $ ros2 action echo /fibonacci example_interfaces/action/Fibonacci --flow-style
   interface: GOAL_SERVICE
   info:
     event_type: REQUEST_SENT
     stamp:
       sec: 1742070798
       nanosec: 400435819
     client_gid: [1, 15, 165, 231, 194, 197, 167, 157, 0, 0, 0, 0, 0, 0, 20, 4]
     sequence_number: 1
   request: [{goal_id: {uuid: [230, 96, 12, 6, 100, 69, 69, 70, 220, 205, 135, 251, 210, 2, 231, 110]}, goal: {order: 10}}]
   response: []
   ---
   interface: GOAL_SERVICE
   info:
     event_type: REQUEST_RECEIVED
     stamp:
       sec: 1742070798
       nanosec: 400706446
     client_gid: [1, 15, 165, 231, 194, 197, 167, 157, 0, 0, 0, 0, 0, 0, 20, 4]
     sequence_number: 1
   request: [{goal_id: {uuid: [230, 96, 12, 6, 100, 69, 69, 70, 220, 205, 135, 251, 210, 2, 231, 110]}, goal: {order: 10}}]
   response: []
   ---
   interface: RESULT_SERVICE
   info:
     event_type: REQUEST_SENT
     stamp:
       sec: 1742070798
       nanosec: 401486678
     client_gid: [1, 15, 165, 231, 194, 197, 167, 157, 0, 0, 0, 0, 0, 0, 24, 4]
     sequence_number: 1
   request: [{goal_id: {uuid: [230, 96, 12, 6, 100, 69, 69, 70, 220, 205, 135, 251, 210, 2, 231, 110]}}]
   response: []
   ---
   interface: FEEDBACK_TOPIC
   goal_id:
     uuid: [230, 96, 12, 6, 100, 69, 69, 70, 220, 205, 135, 251, 210, 2, 231, 110]
   feedback:
     sequence: [0, 1, 1]
   ---
   interface: STATUS_TOPIC
   status_list: [{goal_info: {goal_id: {uuid: [230, 96, 12, 6, 100, 69, 69, 70, 220, 205, 135, 251, 210, 2, 231, 110]}, stamp: {sec: 1742070798, nanosec: 401146752}}, status: 2}]
   ---
   ...

.. note::

   此功能在 ``Kilted Kaiju`` 或更高版本上可用。

总结
----

动作类似于服务，允许您执行长时间运行的任务、提供定期反馈，并且可以取消。

机器人系统可能会使用动作进行导航。
动作目标可以告诉机器人前往某个位置。
当机器人导航到该位置时，它可以沿途发送更新（即反馈），然后在到达目的地后发送最终结果消息。

Turtlesim 有一个动作服务端，动作客户端可以向其发送目标以旋转海龟。
在本教程中，您检查了该动作 ``/turtle1/rotate_absolute``，以更好地了解什么是动作以及它们如何工作。

下一步
------

现在您已经涵盖了所有核心 ROS 2 概念。
本系列的最后几篇教程将向您介绍一些工具和技术，使使用 ROS 2 更容易，从 :doc:`../Using-Rqt-Console/Using-Rqt-Console` 开始。

相关内容
--------

您可以在 `这里 <https://design.ros2.org/articles/actions.html>`__ 阅读更多关于 ROS 2 中动作设计决策的信息。
