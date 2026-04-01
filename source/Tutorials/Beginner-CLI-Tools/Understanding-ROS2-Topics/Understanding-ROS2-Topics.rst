.. redirect-from::

    Tutorials/Topics/Understanding-ROS2-Topics

.. _ROS2Topics:

理解主题
=========

**目标：** 使用 rqt_graph 和命令行工具检查 ROS 2 主题。

**教程级别：** 初级

**时间：** 20 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

ROS 2 将复杂系统分解为许多模块化节点。
主题是 ROS 图的重要组成部分，充当节点交换消息的总线。

.. image:: images/Topic-SinglePublisherandSingleSubscriber.gif

一个节点可以向任意数量的主题发布数据，同时可以订阅任意数量的主题。

.. image:: images/Topic-MultiplePublisherandMultipleSubscriber.gif

主题是数据在节点之间移动的主要方式之一，因此也是系统不同部分之间移动数据的主要方式之一。


前提条件
---------

:doc:`上一篇教程 <../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>` 提供了一些有用的节点背景信息，这些信息在这里会用到。

一如既往，不要忘记在 :doc:`打开的每个新终端 <../Configuring-ROS2-Environment>` 中 source ROS 2。

任务
----

1 设置
^^^^^^

现在您应该已经熟悉启动 turtlesim 了。

打开一个新终端并运行：

.. code-block:: console

    $ ros2 run turtlesim turtlesim_node

打开另一个终端并运行：

.. code-block:: console

    $ ros2 run turtlesim turtle_teleop_key

回顾 :doc:`上一篇教程 <../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>`，这些节点的名称默认为 ``/turtlesim`` 和 ``/teleop_turtle``。


2 rqt_graph
^^^^^^^^^^^

在本教程中，我们将使用 ``rqt_graph`` 来可视化变化的节点和主题，以及它们之间的连接。

:doc:`turtlesim 教程 <../Introducing-Turtlesim/Introducing-Turtlesim>` 告诉您如何安装 rqt 及其所有插件，包括 ``rqt_graph``。

要运行 rqt_graph，打开一个新终端并输入命令：

.. code-block:: console

    $ ros2 run rqt_graph rqt_graph

您也可以通过打开 ``rqt`` 并选择 **Plugins** > **Introspection** > **Node Graph** 来打开 rqt_graph。

.. image:: images/rqt_graph.png

您应该看到上面的节点和主题，以及图形外围的两个动作（暂时忽略它们）。
如果将鼠标悬停在中心的主题上，您将看到如上图所示的颜色高亮。

该图描述了 ``/turtlesim`` 节点和 ``/teleop_turtle`` 节点如何通过主题相互通信。
``/teleop_turtle`` 节点正在向 ``/turtle1/cmd_vel`` 主题发布数据（您输入的用于移动海龟的按键），而 ``/turtlesim`` 节点订阅该主题以接收数据。

当检查具有许多节点和主题以多种不同方式连接的更复杂系统时，rqt_graph 的高亮功能非常有帮助。

rqt_graph 是一个图形化检查工具。
现在我们来看看一些用于检查主题的命令行工具。


3 ros2 topic list
^^^^^^^^^^^^^^^^^

在新终端中运行 ``ros2 topic list`` 命令将返回系统中当前活动的所有主题的列表：

.. code-block:: console

  $ ros2 topic list
  /parameter_events
  /rosout
  /turtle1/cmd_vel
  /turtle1/color_sensor
  /turtle1/pose

``ros2 topic list -t`` 将返回相同的主题列表，这次在括号中附加了主题类型：

.. code-block:: console

  $ ros2 topic list -t
  /parameter_events [rcl_interfaces/msg/ParameterEvent]
  /rosout [rcl_interfaces/msg/Log]
  /turtle1/cmd_vel [geometry_msgs/msg/Twist]
  /turtle1/color_sensor [turtlesim_msgs/msg/Color]
  /turtle1/pose [turtlesim_msgs/msg/Pose]

这些属性，特别是类型，是节点知道它们在主题上传递相同信息的方式。

如果您想知道所有这些主题在 rqt_graph 中的位置，可以取消选中 **Hide:** 下的所有复选框：

.. image:: images/unhide.png

但是，暂时保持选中这些选项以避免混淆。

4 ros2 topic echo
^^^^^^^^^^^^^^^^^

要查看主题上发布的数据，请使用：

.. code-block:: console

    $ ros2 topic echo <topic_name>

由于我们知道 ``/teleop_turtle`` 通过 ``/turtle1/cmd_vel`` 主题向 ``/turtlesim`` 发布数据，让我们使用 ``echo`` 来检查该主题：

.. code-block:: console

    $ ros2 topic echo /turtle1/cmd_vel

起初，这个命令不会返回任何数据。
这是因为它在等待 ``/teleop_turtle`` 发布内容。

返回运行 ``turtle_teleop_key`` 的终端并使用箭头键移动海龟。
同时观察运行 ``echo`` 的终端，您将看到为您的每个动作发布的位置数据：

.. code-block:: console

  linear:
    x: 2.0
    y: 0.0
    z: 0.0
  angular:
    x: 0.0
    y: 0.0
    z: 0.0
    ---

现在返回 rqt_graph 并取消选中 **Debug** 复选框。

.. image:: images/debug.png

``/_ros2cli_26646`` 是我们刚刚运行的 ``echo`` 命令创建的节点（数字可能不同）。
现在您可以看到发布者正在通过 ``cmd_vel`` 主题发布数据，并且有两个订阅者订阅了它。

5 ros2 topic info
^^^^^^^^^^^^^^^^^

主题不一定只是一对一通信；它们可以是一对多、多对一或多对多。

另一种查看方式是运行：

.. code-block:: console

  $ ros2 topic info /turtle1/cmd_vel
  Type: geometry_msgs/msg/Twist
  Publisher count: 1
  Subscription count: 2

5.1 ros2 topic info --verbose
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

要获取有关主题的更详细信息，可以使用 ``--verbose``（或 ``-v``）标志：

.. code-block:: console

  $ ros2 topic info /turtle1/cmd_vel --verbose

这将返回额外的详细信息，包括：

- 发布者和订阅者的节点名称和命名空间
- 主题类型
- QoS 配置文件

.. code-block:: console

  Type: geometry_msgs/msg/Twist

  Publisher count: 1

  Node name: teleop_turtle
  Node namespace: /
  Topic type: geometry_msgs/msg/Twist
  Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
  Endpoint type: PUBLISHER
  GID: 24.ba.3e.e7.c1.51.bb.46.21.41.de.36.1b.14.73.5e
  QoS profile:
    Reliability: RELIABLE
    History (Depth): KEEP_LAST (7)
    Durability: VOLATILE
    Lifespan: Infinite
    Deadline: Infinite
    Liveliness: AUTOMATIC
    Liveliness lease duration: Infinite

  Subscription count: 2

  Node name: _ros2cli_300492
  Node namespace: /
  Topic type: geometry_msgs/msg/Twist
  Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
  Endpoint type: SUBSCRIPTION
  GID: cc.4d.98.79.29.91.fe.25.8a.0a.c9.03.db.1a.ec.81
  QoS profile:
    Reliability: RELIABLE
    History (Depth): KEEP_LAST (5)
    Durability: VOLATILE
    Lifespan: Infinite
    Deadline: Infinite
    Liveliness: AUTOMATIC
    Liveliness lease duration: Infinite

  Node name: turtlesim
  Node namespace: /
  Topic type: geometry_msgs/msg/Twist
  Topic type hash: RIHS01_9c45bf16fe0983d80e3cfe750d6835843d265a9a6c46bd2e609fcddde6fb8d2a
  Endpoint type: SUBSCRIPTION
  GID: 9c.33.59.38.b2.f2.42.47.69.1b.7f.0e.5e.1d.86.f5
  QoS profile:
    Reliability: RELIABLE
    History (Depth): KEEP_LAST (7)
    Durability: VOLATILE
    Lifespan: Infinite
    Deadline: Infinite
    Liveliness: AUTOMATIC
    Liveliness lease duration: Infinite

6 ros2 interface show
^^^^^^^^^^^^^^^^^^^^^

节点使用消息通过主题发送数据。
发布者和订阅者必须发送和接收相同类型的消息才能通信。

我们之前运行 ``ros2 topic list -t`` 后看到的主题类型让我们知道每个主题上使用的消息类型。
回想一下 ``cmd_vel`` 主题的类型是：

.. code-block:: console

    geometry_msgs/msg/Twist

这意味着在 ``geometry_msgs`` 包中有一个名为 ``Twist`` 的 ``msg``。

现在我们可以对此类型运行 ``ros2 interface show <msg_type>`` 来了解其详细信息。
具体来说，消息期望的数据结构。

.. code-block:: console

    $ ros2 interface show geometry_msgs/msg/Twist

这将返回：

.. code-block:: text

    # This expresses velocity in free space broken into its linear and angular parts.
        Vector3  linear
                float64 x
                float64 y
                float64 z
        Vector3  angular
                float64 x
                float64 y
                float64 z

这告诉您 ``/turtlesim`` 节点期望一个包含两个向量 ``linear`` 和 ``angular`` 的消息，每个向量有三个元素。
如果您回想一下我们通过 ``echo`` 命令看到 ``/teleop_turtle`` 传递给 ``/turtlesim`` 的数据，它的结构是相同的：

.. code-block:: console

  linear:
    x: 2.0
    y: 0.0
    z: 0.0
  angular:
    x: 0.0
    y: 0.0
    z: 0.0
    ---

7 ros2 topic pub
^^^^^^^^^^^^^^^^

现在您已经了解了消息结构，可以使用以下命令直接从命令行向主题发布数据：

.. code-block:: console

    $ ros2 topic pub <topic_name> <msg_type> '<args>'

``'<args>'`` 参数是您将传递给主题的实际数据，采用您在上一节中发现的结构。

使用 ``pub`` 命令主要有四种方式，如下所示。
但是，Windows 不支持 ``c.`` 和 ``d.`` 中描述的自动完成功能。

a. **发布字典字符串**：

  要向主题发布数据，您需要以 YAML 字符串的形式传递数据。

  .. code-block:: console

    $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

  但是，如果您只是更改线速度或角速度，则不需要指定整个消息，只需指定要更改的值。

  例如，如果您想将线速度更改为 2.0 并保持角速度为 1.8，可以执行以下操作：

  .. code-block:: console

    $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0}, angular: {z: 1.8}}"

b. **发布空消息**：

  .. code-block:: console

    $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist

  这将以 1 Hz 的频率发布消息类型的默认值。
  在这种情况下，这等效于以下命令：

  .. code-block:: console

    $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}" --rate 1

c. **使用自动完成**：

  您可以通过以下方式触发终端的自动完成功能：

  .. code-block:: console

    $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist <TAB>
    --keep-alive
    --max-wait-time-secs
    --node-name
    --once
    --print
    --qos-depth
    --qos-durability
    --qos-history
    --qos-liveliness
    --qos-liveliness-lease-duration-seconds
    --qos-profile
    --qos-reliability
    --rate
    --spin-time
    --stdin
    --times
    --use-sim-time
    --wait-matching-subscriptions
    --yaml-file
    -1
    -n
    -p
    -r
    -s
    -t
    -w
    \'linear:\^J\ \ x:\ 0.0\^J\ \ y:\ 0.0\^J\ \ z:\ 0.0\^Jangular:\^J\ \ x:\ 0.0\^J\ \ y:\ 0.0\^J\ \ z:\ 0.0\^J\'

所有选项将在输入选项的前几个字符后按 :kbd:`tab` 键自动完成。
但是，主题消息原型只有在输入 ``\'<TAB>`` 后才会自动完成。

这是因为终端不将单引号识别为自动完成字符串的一部分。
因此需要使用 ``\'`` 进行转义才能被识别为字符串的一部分。

最终自动完成的字符串将如下所示：

  .. code-block:: bash

    ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist 'linear:
      x: 0.0
      y: 0.0
      z: 0.0
    angular:
      x: 0.0
      y: 0.0
      z: 0.0
    '

  此字符串是可编辑的，您可以根据需要更改消息类型的值。

d. **使用原始自动完成的字符串**：

  如上所述，``geometry_msgs/msg/Twist`` 的自动完成字符串如下所示：

  .. code-block:: console

    \'linear:\^J\ \ x:\ 0.0\^J\ \ y:\ 0.0\^J\ \ z:\ 0.0\^Jangular:\^J\ \ x:\ 0.0\^J\ \ y:\ 0.0\^J\ \ z:\ 0.0\^J\'

  这可以直接在命令行中代替 yaml 字符串使用。

  .. code-block:: console

    $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist \'linear:\^J\ \ x:\ 0.0\^J\ \ y:\ 0.0\^J\ \ z:\ 0.0\^Jangular:\^J\ \ x:\ 0.0\^J\ \ y:\ 0.0\^J\ \ z:\ 0.0\^J\'


海龟（以及通常它所模拟的真实机器人）需要稳定的命令流才能连续运行。
因此，要让海龟移动并保持移动，您可以使用以下字典字符串：

.. code-block:: console

  $ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

.. image:: images/pub_stream.png

有时您可能只想向主题发布一次数据（而不是连续发布）。
要只发布一次命令，请添加 ``--once`` 选项。

.. code-block:: console

  $ ros2 topic pub --once -w 2 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

``--once`` 是一个可选参数，意思是"发布一条消息然后退出"。

``-w 2`` 是一个可选参数，意思是"等待两个匹配的订阅"。
这是必需的，因为我们同时有 turtlesim 和主题 echo 订阅。

您将在终端中看到以下输出：

.. code-block:: console

  Waiting for at least 2 matching subscription(s)...
  publisher: beginning loop
  publishing #1: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=2.0, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=1.8))

您将看到您的海龟像这样移动：

.. image:: images/pub_once.png


您可以刷新 rqt_graph 以图形方式查看正在发生的事情。
您将看到 ``ros2 topic pub ...`` 节点（``/_ros2cli_30358``）正在通过 ``/turtle1/cmd_vel`` 主题发布，该主题现在被 ``ros2 topic echo ...`` 节点（``/_ros2cli_26646``）和 ``/turtlesim`` 节点接收。

.. image:: images/rqt_graph2.png

最后，您可以在 ``pose`` 主题上运行 ``echo`` 并重新检查 rqt_graph：

.. code-block:: console

  $ ros2 topic echo /turtle1/pose

.. image:: images/rqt_graph3.png

您可以看到 ``/turtlesim`` 节点也正在发布到 ``pose`` 主题，新的 ``echo`` 节点已订阅该主题。

当发布带有时间戳的消息时，``pub`` 有两种方法可以用当前时间自动填充它们。
对于带有 ``std_msgs/msg/Header`` 的消息，可以将 header 字段设置为 ``auto`` 来填充 ``stamp`` 字段。

.. code-block:: console

  $ ros2 topic pub /pose geometry_msgs/msg/PoseStamped '{header: "auto", pose: {position: {x: 1.0, y: 2.0, z: 3.0}}}'

如果消息不使用完整的 header，而只是有一个类型为 ``builtin_interfaces/msg/Time`` 的字段，可以将其设置为值 ``now``。

.. code-block:: console

  $ ros2 topic pub /reference sensor_msgs/msg/TimeReference '{header: "auto", time_ref: "now", source: "dumy"}'

8 ros2 topic hz
^^^^^^^^^^^^^^^

您还可以使用以下命令查看数据发布的速率：

.. code-block:: console

    $ ros2 topic hz /turtle1/pose
    average rate: 59.354
      min: 0.005s max: 0.027s std dev: 0.00284s window: 58

它将返回 ``/turtlesim`` 节点向 ``pose`` 主题发布数据的速率数据。

回想一下，您使用 ``ros2 topic pub --rate 1`` 将 ``turtle1/cmd_vel`` 的发布速率设置为稳定的 1 Hz。
如果您使用 ``turtle1/cmd_vel`` 而不是 ``turtle1/pose`` 运行上述命令，您将看到反映该速率的平均值。

.. Note:: 该速率反映的是 ``ros2 topic hz`` 命令创建的订阅上的接收速率，可能会受到平台资源和 QoS 配置的影响，可能与发布者速率不完全匹配。

9 ros2 topic bw
^^^^^^^^^^^^^^^

可以使用以下命令查看主题使用的带宽：

.. code-block:: console

    $ ros2 topic bw /turtle1/pose
    Subscribed to [/turtle1/pose]
    1.51 KB/s from 62 messages
        Message size mean: 0.02 KB min: 0.02 KB max: 0.02 KB

它返回发布到 ``/turtle1/pose`` 主题的带宽利用率和消息数量。

.. Note:: 带宽反映的是 ``ros2 topic bw`` 命令创建的订阅上的接收速率，可能会受到平台资源和 QoS 配置的影响，可能与发布者的带宽不完全匹配。

10 ros2 topic find
^^^^^^^^^^^^^^^^^^

要列出给定类型的可用主题列表，请使用：

.. code-block:: console

    $ ros2 topic find <topic_type>

回想一下 ``cmd_vel`` 主题的类型是：

.. code-block:: console

    geometry_msgs/msg/Twist

使用 ``find`` 命令可以输出给定消息类型时可用的主题：

.. code-block:: console

    $ ros2 topic find geometry_msgs/msg/Twist
    /turtle1/cmd_vel

11 清理
^^^^^^^

此时您将有大量节点在运行。
不要忘记在每个终端中输入 ``Ctrl+C`` 来停止它们。

总结
----

节点通过主题发布信息，这允许任意数量的其他节点订阅并访问该信息。
在本教程中，您使用 rqt_graph 和命令行工具检查了多个节点之间通过主题的连接。
您现在应该对数据如何在 ROS 2 系统中移动有了很好的了解。

下一步
------

接下来，您将通过教程 :doc:`../Understanding-ROS2-Services/Understanding-ROS2-Services` 了解 ROS 图中的另一种通信类型。
