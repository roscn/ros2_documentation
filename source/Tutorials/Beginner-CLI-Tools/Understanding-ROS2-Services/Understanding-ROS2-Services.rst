.. redirect-from::

    Tutorials/Services/Understanding-ROS2-Services

.. _ROS2Services:

理解服务
=========

**目标：** 使用命令行工具了解 ROS 2 中的服务。

**教程级别：** 初级

**时间：** 10 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

服务是 ROS 图中节点的另一种通信方式。
服务基于调用-响应模型，与主题的发布者-订阅者模型不同。
虽然主题允许节点订阅数据流并获得持续更新，但服务仅在客户端专门调用时提供数据。

.. image:: images/Service-SingleServiceClient.gif

.. image:: images/Service-MultipleServiceClient.gif

前提条件
---------

本教程中提到的一些概念，如 :doc:`节点 <../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>` 和 :doc:`主题 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>`，已在系列的前几篇教程中介绍过。

您需要 :doc:`turtlesim 软件包 <../Introducing-Turtlesim/Introducing-Turtlesim>`。

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

2 ros2 service list
^^^^^^^^^^^^^^^^^^^

在新终端中运行 ``ros2 service list`` 命令将返回系统中当前活动的所有服务的列表：

.. code-block:: console

  $ ros2 service list
  /clear
  /kill
  /reset
  /spawn
  /teleop_turtle/describe_parameters
  /teleop_turtle/get_parameter_types
  /teleop_turtle/get_parameters
  /teleop_turtle/list_parameters
  /teleop_turtle/set_parameters
  /teleop_turtle/set_parameters_atomically
  /turtle1/set_pen
  /turtle1/teleport_absolute
  /turtle1/teleport_relative
  /turtlesim/describe_parameters
  /turtlesim/get_parameter_types
  /turtlesim/get_parameters
  /turtlesim/list_parameters
  /turtlesim/set_parameters
  /turtlesim/set_parameters_atomically

您将看到两个节点都有名称中包含 ``parameters`` 的相同六个服务。
ROS 2 中几乎每个节点都有这些构建参数的基础设施服务。
下一篇教程将详细介绍参数。
在本教程中，将不讨论参数服务。

现在，让我们专注于 turtlesim 特有的服务：``/clear``、``/kill``、``/reset``、``/spawn``、``/turtle1/set_pen``、``/turtle1/teleport_absolute`` 和 ``/turtle1/teleport_relative``。
您可能还记得在 :doc:`使用 turtlesim、ros2 和 rqt <../Introducing-Turtlesim/Introducing-Turtlesim>` 教程中使用 rqt 与其中一些服务交互。


3 ros2 service type
^^^^^^^^^^^^^^^^^^^

服务有类型，描述服务的请求和响应数据如何结构化。
服务类型的定义与主题类型类似，不同之处在于服务类型有两个部分：一个用于请求的消息，另一个用于响应的消息。

要查找服务的类型，请使用命令：

.. code-block:: console

  $ ros2 service type <service_name>

让我们看看 turtlesim 的 ``/clear`` 服务。
在新终端中，输入命令：

.. code-block:: console

  $ ros2 service type /clear
  std_srvs/srv/Empty

``Empty`` 类型意味着服务调用在发出请求时不发送数据，在接收响应时不接收数据。

3.1 ros2 service list -t
~~~~~~~~~~~~~~~~~~~~~~~~

要同时查看所有活动服务的类型，可以在 ``list`` 命令后附加 ``--show-types`` 选项，缩写为 ``-t``：

.. code-block:: console

  $ ros2 service list -t
  /clear [std_srvs/srv/Empty]
  /kill [turtlesim_msgs/srv/Kill]
  /reset [std_srvs/srv/Empty]
  /spawn [turtlesim_msgs/srv/Spawn]
  ...
  /turtle1/set_pen [turtlesim_msgs/srv/SetPen]
  /turtle1/teleport_absolute [turtlesim_msgs/srv/TeleportAbsolute]
  /turtle1/teleport_relative [turtlesim_msgs/srv/TeleportRelative]
  ...

4 ros2 service info
^^^^^^^^^^^^^^^^^^^

要查看特定服务的信息，请使用命令：

.. code-block:: console

  $ ros2 service info <service_name>

这将返回服务类型以及服务客户端和服务端的数量。

例如，您可以找到 ``/clear`` 服务的客户端和服务端数量：

.. code-block:: console

   $ ros2 service info /clear
   Type: std_srvs/srv/Empty
   Clients count: 0
   Services count: 1

4.1 ros2 service info --verbose
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
要获取有关服务的更多详细信息，可以在 ``info`` 命令后附加 ``--verbose``（或 ``-v``）选项：

.. code-block:: console

  $ ros2 service info --verbose <service_name>

例如：您可以获取有关 ``/clear`` 服务的详细信息：

.. code-block:: console

  $ ros2 service info --verbose /clear

这将返回额外的信息，包括服务服务端的节点名称和命名空间，以及底层中间件（RMW）实现细节。
对于开发者来说，一个重要信息是基于 DDS 的 RMW 实现（connextdds、cyclone、fastrtps）的 ``Endpoint count`` 将为 2，因为 DDS 为每个服务服务端创建两个端点（一个用于请求，一个用于响应）。

.. code-block:: console

    Type: std_srvs/srv/Empty
    Clients count: 0
    Services count: 1
    Node name: turtlesim
    Node namespace: /
    Service type: std_srvs/srv/Empty
    Service type hash: RIHS01_5888399dedec5ccc85ea6451949fd2c9f97bfdf963f9a588821639fcd31b5d19
    Endpoint type: SERVER
    Endpoint count: 2
    GIDs:
    - Request Reader : 01.0f.93.f0.07.92.53.47.00.00.00.00.00.00.13.04
    - Response Writer : 01.0f.93.f0.07.92.53.47.00.00.00.00.00.00.14.03
    QoS profiles:
    - Request Reader :
          Reliability: RELIABLE
          History (Depth): KEEP_LAST (10)
          Durability: VOLATILE
          Lifespan: Infinite
          Deadline: Infinite
          Liveliness: AUTOMATIC
          Liveliness lease duration: Infinite
    - Response Writer :
          Reliability: RELIABLE
          History (Depth): KEEP_LAST (10)
          Durability: VOLATILE
          Lifespan: Infinite
          Deadline: Infinite
          Liveliness: AUTOMATIC
          Liveliness lease duration: Infinite

而对于基于非 DDS 的 RMW 实现（如 ``rmw_zenoh_cpp``），``Endpoint count`` 将为 1，因为它使用单个端点处理请求和响应。

.. code-block:: console

    Type: std_srvs/srv/Empty
    Clients count: 0
    Services count: 1
    Node name: turtlesim
    Node namespace: /
    Service type: std_srvs/srv/Empty
    Service type hash: RIHS01_5888399dedec5ccc85ea6451949fd2c9f97bfdf963f9a588821639fcd31b5d19
    Endpoint type: SERVER
    Endpoint count: 1
    GID: 59.b0.ea.78.57.3c.52.b4.c6.e9.af.44.22.3d.7c.f5
    QoS profile:
      Reliability: RELIABLE
      History (Depth): KEEP_LAST (10)
      Durability: VOLATILE
      Lifespan: Infinite
      Deadline: Infinite
      Liveliness: AUTOMATIC
      Liveliness lease duration: Infinite

如果您想了解更多关于不同 RMW 实现的信息，请参阅 :doc:`关于不同中间件供应商 <../../../Concepts/Intermediate/About-Different-Middleware-Vendors>` 概念。

5 ros2 service find
^^^^^^^^^^^^^^^^^^^

如果您想查找特定类型的所有服务，可以使用命令：

.. code-block:: console

  $ ros2 service find <type_name>

例如，您可以像这样找到所有 ``Empty`` 类型的服务：

.. code-block:: console

  $ ros2 service find std_srvs/srv/Empty
  /clear
  /reset

6 ros2 interface show
^^^^^^^^^^^^^^^^^^^^^

您可以从命令行调用服务，但首先需要了解输入参数的结构。

.. code-block:: console

  $ ros2 interface show <type_name>

在 ``/clear`` 服务的类型 ``Empty`` 上尝试这个：

.. code-block:: console

  $ ros2 interface show std_srvs/srv/Empty
  ---

``---`` 将请求结构（上方）与响应结构（下方）分开。
但是，正如您之前学到的，``Empty`` 类型不发送或接收任何数据。
所以，自然地，它的结构是空白的。

让我们检查一个具有发送和接收数据类型的服务，如 ``/spawn``。
从 ``ros2 service list -t`` 的结果中，我们知道 ``/spawn`` 的类型是 ``turtlesim_msgs/srv/Spawn``。

要查看 ``/spawn`` 服务的请求和响应参数，运行命令：

.. code-block:: console

  $ ros2 interface show turtlesim_msgs/srv/Spawn
  float32 x
  float32 y
  float32 theta
  string name # Optional.  A unique name will be created and returned if this is empty
  ---
  string name

``---`` 行上方的信息告诉我们调用 ``/spawn`` 所需的参数。
``x``、``y`` 和 ``theta`` 决定生成海龟的 2D 位姿，``name`` 显然是可选的。

行下方的信息在这种情况下不是您需要知道的，但它可以帮助您了解从调用中获得的响应的数据类型。

7 ros2 service call
^^^^^^^^^^^^^^^^^^^

现在您已经知道什么是服务类型、如何查找服务的类型以及如何查找该类型参数的结构，您可以使用以下命令调用服务：

.. code-block:: console

  $ ros2 service call <service_name> <service_type> <arguments>

``<arguments>`` 部分是可选的。
例如，您知道 ``Empty`` 类型的服务没有任何参数：

.. code-block:: console

  $ ros2 service call /clear std_srvs/srv/Empty

此命令将清除 turtlesim 窗口中您的海龟绘制的任何线条。

.. image:: images/clear.png

现在让我们通过调用 ``/spawn`` 并设置参数来生成一只新海龟。
从命令行调用服务时输入的 ``<arguments>`` 需要使用 YAML 语法。

输入命令：

.. code-block:: console

  $ ros2 service call /spawn turtlesim_msgs/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"
  requester: making request: turtlesim_msgs.srv.Spawn_Request(x=2.0, y=2.0, theta=0.2, name='')

  response:
  turtlesim_msgs.srv.Spawn_Response(name='turtle2')

您将看到这种方法式的视图显示正在发生的事情，然后是服务响应。

您的 turtlesim 窗口将立即更新为新生成的海龟：

.. image:: images/spawn.png

8 ros2 service echo
^^^^^^^^^^^^^^^^^^^

要查看服务客户端和服务服务端之间的数据通信，您可以使用以下命令 ``echo`` 服务：

.. code-block:: console

  $ ros2 service echo <service_name | service_type> <arguments>

``ros2 service echo`` 依赖于服务客户端和服务端的服务内省，默认情况下是禁用的。
要启用它，用户必须在创建服务客户端或服务端后调用 ``configure_introspection``。

启动 ``introspection_client`` 和 ``introspection_service`` 服务内省演示。

.. code-block:: console

  $ ros2 launch demo_nodes_cpp introspect_services_launch.py

打开另一个终端并运行以下命令以启用 ``introspection_client`` 和 ``introspection_service`` 的服务内省。

.. code-block:: console

  $ ros2 param set /introspection_service service_configure_introspection contents
  $ ros2 param set /introspection_client client_configure_introspection contents

现在我们可以通过 ``ros2 service echo`` 查看 ``introspection_client`` 和 ``introspection_service`` 之间的服务通信。

.. code-block:: console

  $ ros2 service echo --flow-style /add_two_ints
   info:
     event_type: REQUEST_SENT
     stamp:
       sec: 1709408301
       nanosec: 423227292
     client_gid: [1, 15, 0, 18, 250, 205, 12, 100, 0, 0, 0, 0, 0, 0, 21, 3]
     sequence_number: 618
   request: [{a: 2, b: 3}]
   response: []
   ---
   info:
     event_type: REQUEST_RECEIVED
     stamp:
       sec: 1709408301
       nanosec: 423601471
     client_gid: [1, 15, 0, 18, 250, 205, 12, 100, 0, 0, 0, 0, 0, 0, 20, 4]
     sequence_number: 618
   request: [{a: 2, b: 3}]
   response: []
   ---
   info:
     event_type: RESPONSE_SENT
     stamp:
       sec: 1709408301
       nanosec: 423900744
     client_gid: [1, 15, 0, 18, 250, 205, 12, 100, 0, 0, 0, 0, 0, 0, 20, 4]
     sequence_number: 618
   request: []
   response: [{sum: 5}]
   ---
   info:
     event_type: RESPONSE_RECEIVED
     stamp:
       sec: 1709408301
       nanosec: 424153133
     client_gid: [1, 15, 0, 18, 250, 205, 12, 100, 0, 0, 0, 0, 0, 0, 21, 3]
     sequence_number: 618
   request: []
   response: [{sum: 5}]
   ---

总结
----

节点可以使用 ROS 2 中的服务进行通信。
与主题——一种单向通信模式，节点发布信息供一个或多个订阅者消费——不同，服务是一种请求/响应模式，客户端向提供服务的节点发出请求，服务处理请求并生成响应。

通常您不希望使用服务进行连续调用；主题甚至动作会更适合。

在本教程中，您使用命令行工具来识别、检查和调用服务。

下一步
------

在下一篇教程中，:doc:`../Understanding-ROS2-Parameters/Understanding-ROS2-Parameters`，您将学习配置节点设置。

相关内容
--------

查看 `这个教程 <https://discourse.ubuntu.com/t/call-services-in-ros-2/15261>`_；这是一个使用 Robotis 机械臂的 ROS 服务的优秀实际应用。
