.. redirect-from::

    Tutorials/Ros2bag/Recording-And-Playing-Back-Data

.. _ROS2Bag:

Recording and playing back data
===============================

**目标：** 记录主题、服务和动作上发布的数据，以便您可以随时重放和检查。

**教程级别：** 初级

**时间：** 20 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

``ros2 bag`` 是一个命令行工具，用于记录 ROS 2 系统中主题、服务和动作上发布的数据。
它累积在任意数量的主题、服务和动作上传递的数据，然后将其保存到数据库中。
然后您可以重放数据以重现测试和实验的结果。
记录主题、服务和动作也是分享您的工作并允许他人重现的好方法。


前提条件
--------

您应该在常规 ROS 2 设置中安装了 ``ros2 bag``。

如果您需要安装 ROS 2，请参阅 :doc:`安装说明 <../../../Installation>`。

本教程讨论了之前教程中涵盖的概念，如 :doc:`节点 <../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>`、:doc:`主题 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>`、:doc:`服务 <../Understanding-ROS2-Services/Understanding-ROS2-Services>` 和 :doc:`动作 <../Understanding-ROS2-Actions/Understanding-ROS2-Actions>`。
它还使用 :doc:`turtlesim 包 <../Introducing-Turtlesim/Introducing-Turtlesim>`、:doc:`服务内省演示 <../../Demos/Service-Introspection>` 和 :doc:`动作内省演示 <../../Demos/Action-Introspection>`。

一如既往，不要忘记在 :doc:`您打开的每个新终端 <../Configuring-ROS2-Environment>` 中 source ROS 2。


管理主题数据
------------

1 设置
^^^^^^

您将在 ``turtlesim`` 系统中记录键盘输入以便稍后保存和重放，因此首先启动 ``/turtlesim`` 和 ``/teleop_turtle`` 节点。

打开一个新终端并运行：

.. code-block:: console

    $ ros2 run turtlesim turtlesim_node

打开另一个终端并运行：

.. code-block:: console

    $ ros2 run turtlesim turtle_teleop_key

让我们也创建一个新目录来存储保存的记录，这是一个好习惯：

.. tabs::

    .. group-tab:: Linux

        .. code-block:: console

            $ mkdir bag_files
            $ cd bag_files

    .. group-tab:: macOS

        .. code-block:: console

            $ mkdir bag_files
            $ cd bag_files

    .. group-tab:: Windows

        .. code-block:: console

            $ md bag_files
            $ cd bag_files


2 选择主题
^^^^^^^^^^

``ros2 bag`` 可以记录发布到主题的消息数据。
要查看系统主题列表，打开一个新终端并运行命令：

.. code-block:: console

  $ ros2 topic list
  /parameter_events
  /rosout
  /turtle1/cmd_vel
  /turtle1/color_sensor
  /turtle1/pose

在主题教程中，您了解到 ``/turtle_teleop`` 节点在 ``/turtle1/cmd_vel`` 主题上发布命令，使 turtlesim 中的海龟移动。

要查看 ``/turtle1/cmd_vel`` 发布的数据，运行命令：

.. code-block:: console

    $ ros2 topic echo /turtle1/cmd_vel

起初不会显示任何内容，因为 teleop 没有发布数据。
返回运行 teleop 的终端并选择它使其处于活动状态。
使用箭头键移动海龟，您将在运行 ``ros2 topic echo`` 的终端上看到正在发布的数据。

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


3 记录主题
^^^^^^^^^^

3.1 记录单个主题
~~~~~~~~~~~~~~~~

要记录发布到主题的数据，使用命令语法：

.. code-block:: console

    $ ros2 bag record --topics <topic_name>

在您选择的主题上运行此命令之前，打开一个新终端并移动到您之前创建的 ``bag_files`` 目录，因为 rosbag 文件将保存在您运行它的目录中。

运行命令：

.. code-block:: console

    $ ros2 bag record --topics /turtle1/cmd_vel
    [INFO] [rosbag2_storage]: Opened database 'rosbag2_2019_10_11-05_18_45'.
    [INFO] [rosbag2_transport]: Listening for topics...
    [INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/cmd_vel'
    [INFO] [rosbag2_transport]: All requested topics are subscribed. Stopping discovery...

现在 ``ros2 bag`` 正在记录 ``/turtle1/cmd_vel`` 主题上发布的数据。
返回 teleop 终端并再次移动海龟。
移动并不重要，但尝试做一个可识别的模式，以便稍后重放数据时查看。

.. image:: images/record.png

按 :kbd:`Ctrl-C` 停止记录。

数据将累积到一个新的 bag 目录中，名称格式为 ``rosbag2_year_month_day-hour_minute_second``。
此目录将包含 ``metadata.yaml`` 以及录制格式的 bag 文件。

3.2 记录多个主题
~~~~~~~~~~~~~~~~

您还可以记录多个主题，以及更改 ``ros2 bag`` 保存的 bag 目录名称。

Run the following command:

.. code-block:: console

  $ ros2 bag record -o subset --topics /turtle1/cmd_vel /turtle1/pose
  [INFO] [rosbag2_storage]: Opened database 'subset'.
  [INFO] [rosbag2_transport]: Listening for topics...
  [INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/cmd_vel'
  [INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/pose'
  [INFO] [rosbag2_transport]: All requested topics are subscribed. Stopping discovery...

``-o`` 选项允许您为 bag 目录选择一个唯一的名称。
后面的字符串，在本例中是 ``subset``，是 bag 目录名称。

要一次记录多个主题，只需在 ``--topics`` 后列出每个主题，用空格分隔。
在这种情况下，上面的命令输出确认正在记录两个主题。

您可以移动海龟，完成后按 :kbd:`Ctrl-C`。

.. note::

    您可以在命令中添加另一个选项 ``-a``，它会记录系统上的所有主题。

3.3 将录制拆分为多个文件
~~~~~~~~~~~~~~~~~~~~~~~~

您还可以根据录制持续时间或文件大小将录制拆分为多个文件。
``-d <max_bag_duration>`` 确保每个文件在开始写入新文件之前只持续 ``<max_bag_duration>`` 秒，或者 ``-b <max_bag_size>`` 确保每个文件的文件大小不超过 ``<max_bag_size>`` 字节。
这可以防止文件过大和笨重，并防止在录制操作在某个时刻损坏时丢失所有数据。

运行以下命令至少 15 秒，允许写入三个 5 秒的 bag 文件：

.. code-block:: console

    $ ros2 bag record -o subset_split -d 5 --topics /turtle1/cmd_vel /turtle1/pose
    [INFO] [rosbag2_recorder]: Press SPACE for pausing/resuming
    [INFO] [rosbag2_recorder]: Listening for topics...
    [INFO] [rosbag2_recorder]: Event publisher thread: Starting
    [INFO] [rosbag2_recorder]: Recording...
    [INFO] [rosbag2_recorder]: Subscribed to topic '/turtle1/cmd_vel'
    [INFO] [rosbag2_recorder]: Subscribed to topic '/turtle1/pose'
    [INFO] [rosbag2_recorder]: All requested topics are subscribed. Stopping discovery...
    [INFO] [rosbag2_cpp]: Writing remaining messages from cache to the bag. It may take a while
    [INFO] [rosbag2_cpp]: Writing remaining messages from cache to the bag. It may take a while
    [INFO] [rosbag2_cpp]: Writing remaining messages from cache to the bag. It may take a while

完成后按 :kbd:`Ctrl-C`。
您应该找到一个 ``subset_split`` 目录，其中包含这些文件：``subset_split_0.mcap``、``subset_split_1.mcap`` 等等。

4 检查主题数据
^^^^^^^^^^^^^^

您可以通过运行以下命令查看有关录制的详细信息：

.. code-block:: console

    $ ros2 bag info <bag_name>

在 ``subset`` bag 录制上运行此命令将返回信息列表：

.. code-block:: console

    $ ros2 bag info subset
    Files:             subset_0.mcap
    Bag size:          228.5 KiB
    Storage id:        mcap
    Duration:          48.47s
    Start:             Oct 11 2019 06:09:09.12 (1570799349.12)
    End                Oct 11 2019 06:09:57.60 (1570799397.60)
    Messages:          3013
    Topic information: Topic: /turtle1/cmd_vel | Type: geometry_msgs/msg/Twist | Count: 9 | Serialization Format: cdr
                       Topic: /turtle1/pose | Type: turtlesim_msgs/msg/Pose | Count: 3004 | Serialization Format: cdr
    Services:          0
    Service information:
    Actions:           0
    Action information:

或者，您也可以对单个文件调用 ``ros2 bag info``，例如 ``subset_split/subset_split_0.mcap``，它只会显示该部分录制的信息；在这种情况下，是前 5 秒。

5 播放主题数据
^^^^^^^^^^^^^^

5.1 播放单个 bag
~~~~~~~~~~~~~~~~

在重放 bag 之前，在运行 teleop 的终端中输入 :kbd:`Ctrl-C`。
然后确保您的 turtlesim 窗口可见，以便您可以看到 bag 文件的运行情况。

输入命令：

.. code-block:: console

    $ ros2 bag play subset
    [INFO] [rosbag2_player]: Set rate to 1
    [INFO] [rosbag2_player]: Adding keyboard callbacks.
    [INFO] [rosbag2_player]: Press SPACE for Pause/Resume
    [INFO] [rosbag2_player]: Press CURSOR_RIGHT for Play Next Message
    [INFO] [rosbag2_player]: Press CURSOR_UP for Increase Rate 10%
    [INFO] [rosbag2_player]: Press CURSOR_DOWN for Decrease Rate 10%
    Progress bar enabled at 3 Hz.
    Progress bar [?]: [R]unning, [P]aused, [B]urst, [D]elayed, [S]topped
    [INFO] [rosbag2_player]: Playback until timestamp: -1


    ====== Playback Progress ======
    [1751923361.427372456] Duration 0.00/48.47 [R]

您的海龟将遵循您在录制时输入的相同路径（虽然不是 100% 准确；turtlesim 对系统时间的微小变化很敏感）。

.. image:: images/playback.png

因为 ``subset`` 文件记录了 ``/turtle1/pose`` 主题，所以 ``ros2 bag play`` 命令不会在您运行 turtlesim 时退出，即使您没有移动。

这是因为只要 ``/turtlesim`` 节点处于活动状态，它就会定期在 ``/turtle1/pose`` 主题上发布数据。
您可能在上面的 ``ros2 bag info`` 示例结果中注意到，``/turtle1/cmd_vel`` 主题的 ``Count`` 信息只有 9；这是我们在录制时按箭头键的次数。

请注意，``/turtle1/pose`` 的 ``Count`` 值超过 3000；在录制时，该主题上发布了 3000 次数据。

要了解位置数据发布的频率，您可以运行命令：

.. code-block:: console

    $ ros2 topic hz /turtle1/pose

5.2 播放多个 bag
~~~~~~~~~~~~~~~~~

有时，将所需的录制主题分配到多个录制中是相关的，作为分配录制工作负载的一种方式。
例如，我们可以将 ``/turtle1/cmd_vel`` 和 ``/turtle1/pose`` 分别录制到各自的 bag 中。

创建两个终端实例。
在第一个中，运行以下命令：

.. code-block:: console

    $ ros2 bag record -o subset_cmd_vel --topics /turtle1/cmd_vel

在第二个终端中，运行此命令：

.. code-block:: console

    $ ros2 bag record -o subset_pose --topics /turtle1/pose

像之前一样移动海龟，完成后用 :kbd:`Ctrl-C` 结束两个录制。

要让这两个录制以正确的时间并行播放，请为要包含的每个 bag 调用带有 ``-i <bag_name>`` 的 ``ros2 bag play``。
在这种情况下，运行：

.. code-block:: console

    $ ros2 bag play -i subset_cmd_vel -i subset_pose

这将一起播放 ``subset_cmd_vel`` 和 ``subset_pose`` 录制，播放同步以重现消息的原始顺序。
如果使用，可选参数 ``--message-order {received,sent}`` 决定消息是根据接收还是发布的时间排序（默认为接收）。
这也适用于播放单个 bag。

管理服务数据
------------

1 设置
^^^^^^

您将记录 ``introspection_client`` 和 ``introspection_service`` 之间的服务数据，然后稍后显示和重放相同的数据。
要记录服务客户端和服务器之间的服务数据，必须在节点上启用 ``Service Introspection``。

让我们启动 ``introspection_client`` 和 ``introspection_service`` 节点并启用 ``Service Introspection``。
您可以在 :doc:`服务内省演示 <../../Demos/Service-Introspection>` 中查看更多详细信息。

打开一个新终端并运行 ``introspection_service``，启用 ``Service Introspection``：

.. code-block:: console

    $ ros2 run demo_nodes_cpp introspection_service --ros-args -p service_configure_introspection:=contents

打开另一个终端并运行 ``introspection_client``，启用 ``Service Introspection``：

.. code-block:: console

    $ ros2 run demo_nodes_cpp introspection_client --ros-args -p client_configure_introspection:=contents

2 检查服务可用性
^^^^^^^^^^^^^^^^

``ros2 bag`` 只能记录可用服务的数据。
要查看系统服务列表，打开一个新终端并运行命令：

.. code-block:: console

  $ ros2 service list
  /add_two_ints
  /introspection_client/describe_parameters
  /introspection_client/get_parameter_types
  /introspection_client/get_parameters
  /introspection_client/get_type_description
  /introspection_client/list_parameters
  /introspection_client/set_parameters
  /introspection_client/set_parameters_atomically
  /introspection_service/describe_parameters
  /introspection_service/get_parameter_types
  /introspection_service/get_parameters
  /introspection_service/get_type_description
  /introspection_service/list_parameters
  /introspection_service/set_parameters
  /introspection_service/set_parameters_atomically

要检查客户端和服务上是否启用了 ``Service Introspection``，运行命令：

.. code-block:: console

  $ ros2 service echo --flow-style /add_two_ints
  info:
    event_type: REQUEST_SENT
    stamp:
      sec: 1713995389
      nanosec: 386809259
    client_gid: [1, 15, 96, 219, 162, 1, 108, 201, 0, 0, 0, 0, 0, 0, 21, 3]
    sequence_number: 133
  request: [{a: 2, b: 3}]
  response: []
  ---

您应该看到服务通信。

3 记录服务
^^^^^^^^^^

要记录服务数据，支持以下选项。
服务数据可以与主题同时记录。

要记录特定服务：

.. code-block:: console

  $ ros2 bag record --service <service_names>

要记录所有服务：

.. code-block:: console

  $ ros2 bag record --all-services

运行命令：

.. code-block:: console

  $ ros2 bag record --service /add_two_ints
  [INFO] [1713995957.643573503] [rosbag2_recorder]: Press SPACE for pausing/resuming
  [INFO] [1713995957.662067587] [rosbag2_recorder]: Event publisher thread: Starting
  [INFO] [1713995957.662067614] [rosbag2_recorder]: Listening for topics...
  [INFO] [1713995957.666048323] [rosbag2_recorder]: Subscribed to topic '/add_two_ints/_service_event'
  [INFO] [1713995957.666092458] [rosbag2_recorder]: Recording...

现在 ``ros2 bag`` 正在记录 ``/add_two_ints`` 服务上发布的服务数据。
要停止录制，在终端中输入 :kbd:`Ctrl-C`。

数据将累积到一个新的 bag 目录中，名称格式为 ``rosbag2_year_month_day-hour_minute_second``。
此目录将包含 ``metadata.yaml`` 以及录制格式的 bag 文件。

4 检查服务数据
^^^^^^^^^^^^^^

您可以通过运行以下命令查看有关录制的详细信息：

.. code-block:: console

  $ ros2 bag info <bag_file_name>
  Files:             rosbag2_2024_04_24-14_59_17_0.mcap
  Bag size:          15.1 KiB
  Storage id:        mcap
  ROS Distro:        rolling
  Duration:          9.211s
  Start:             Apr 24 2024 14:59:17.676 (1713995957.676)
  End:               Apr 24 2024 14:59:26.888 (1713995966.888)
  Messages:          0
  Topic information:
  Service:           1
  Service information: Service: /add_two_ints | Type: example_interfaces/srv/AddTwoInts | Event Count: 78 | Serialization Format: cdr

5 播放服务数据
^^^^^^^^^^^^^^

在重放 bag 文件之前，在运行 ``introspection_client`` 的终端中输入 :kbd:`Ctrl-C`。
当 ``introspection_client`` 停止运行时，``introspection_service`` 也会停止打印结果，因为没有传入的请求。

从 bag 文件重放服务数据将开始向 ``introspection_service`` 发送请求。

输入命令：

.. code-block:: console

  $ ros2 bag play --publish-service-requests <bag_file_name>
  [INFO] [1713997477.870856190] [rosbag2_player]: Set rate to 1
  [INFO] [1713997477.877417477] [rosbag2_player]: Adding keyboard callbacks.
  [INFO] [1713997477.877442404] [rosbag2_player]: Press SPACE for Pause/Resume
  [INFO] [1713997477.877447855] [rosbag2_player]: Press CURSOR_RIGHT for Play Next Message
  [INFO] [1713997477.877452655] [rosbag2_player]: Press CURSOR_UP for Increase Rate 10%
  [INFO] [1713997477.877456954] [rosbag2_player]: Press CURSOR_DOWN for Decrease Rate 10%
  [INFO] [1713997477.877573647] [rosbag2_player]: Playback until timestamp: -1

您的 ``introspection_service`` 终端将再次开始打印以下服务消息：

.. code-block:: console

  [INFO] [1713997478.090466075] [introspection_service]: Incoming request
  a: 2 b: 3

这是因为 ``ros2 bag play`` 将 bag 文件中的服务请求数据发送到 ``/add_two_ints`` 服务。

我们还可以在 ``ros2 bag play`` 播放时内省服务通信以验证 ``introspection_service``。

在 ``ros2 bag play`` 之前运行此命令以查看 ``introspection_service``：

.. code-block:: console

  $ ros2 service echo --flow-style /add_two_ints

您可以看到来自 bag 文件的服务请求和来自 ``introspection_service`` 的服务响应。

.. code-block:: console

  info:
    event_type: REQUEST_RECEIVED
    stamp:
      sec: 1713998176
      nanosec: 372700698
    client_gid: [1, 15, 96, 219, 80, 2, 158, 123, 0, 0, 0, 0, 0, 0, 20, 4]
    sequence_number: 1
  request: [{a: 2, b: 3}]
  response: []
  ---
  info:
    event_type: RESPONSE_SENT
    stamp:
      sec: 1713998176
      nanosec: 373016882
    client_gid: [1, 15, 96, 219, 80, 2, 158, 123, 0, 0, 0, 0, 0, 0, 20, 4]
    sequence_number: 1
  request: []
  response: [{sum: 5}]

.. _record-play-data-action:

管理动作数据
------------

1 设置
^^^^^^

您将记录 ``fibonacci_action_client`` 和 ``fibonacci_action_server`` 之间的动作数据，然后稍后显示和重放相同的数据。
要记录动作客户端和服务器之间的动作数据，必须在节点上启用 ``Action Introspection``。

让我们启动 ``fibonacci_action_client`` 和 ``fibonacci_action_server`` 节点并启用 ``Action Introspection``。
您可以在 :doc:`动作内省演示 <../../Demos/Action-Introspection>` 中查看更多详细信息。

打开一个新终端并运行 ``fibonacci_action_server``，启用 ``Action Introspection``：

.. code-block:: console

  $ ros2 run action_tutorials_py fibonacci_action_server --ros-args -p action_server_configure_introspection:=contents

打开另一个终端并运行 ``fibonacci_action_client``，启用 ``Action Introspection``：

.. code-block:: console

  $ ros2 run action_tutorials_cpp fibonacci_action_client --ros-args -p action_client_configure_introspection:=contents

2 检查动作可用性
^^^^^^^^^^^^^^^^

``ros2 bag`` 只能记录可用动作的数据。
要查看系统动作列表，打开一个新终端并运行命令：

.. code-block:: console

  $ ros2 action list
  /fibonacci

要检查动作上是否启用了 ``Action Introspection``，运行命令：

.. code-block:: console

  $ ros2 action echo --flow-style /fibonacci
  interface: GOAL_SERVICE
  info:
    event_type: REQUEST_SENT
    stamp:
      sec: 1744917904
      nanosec: 760683446
    client_gid: [1, 15, 165, 231, 234, 109, 65, 202, 0, 0, 0, 0, 0, 0, 19, 4]
    sequence_number: 1
  request: [{goal_id: {uuid: [81, 55, 121, 145, 81, 66, 209, 93, 214, 113, 255, 100, 120, 6, 102, 83]}, goal: {order: 10}}]
  response: []
  ---
  ...

3 记录动作
^^^^^^^^^^

要记录动作数据，支持以下选项。
动作数据可以与主题和服务同时记录。

要记录特定动作：

.. code-block:: console

  $ ros2 bag record --action <action_names>

要记录所有动作：

.. code-block:: console

  $ ros2 bag record --all-actions

运行命令：

.. code-block:: console

  $ ros2 bag record --action /fibonacci
  [INFO] [1744953225.214114862] [rosbag2_recorder]: Press SPACE for pausing/resuming
  [INFO] [1744953225.218369761] [rosbag2_recorder]: Listening for topics...
  [INFO] [1744953225.218386223] [rosbag2_recorder]: Event publisher thread: Starting
  [INFO] [1744953225.218580294] [rosbag2_recorder]: Recording...
  [INFO] [1744953225.725417634] [rosbag2_recorder]: Subscribed to topic '/fibonacci/_action/cancel_goal/_service_event'
  [INFO] [1744953225.727901848] [rosbag2_recorder]: Subscribed to topic '/fibonacci/_action/feedback'
  [INFO] [1744953225.729655213] [rosbag2_recorder]: Subscribed to topic '/fibonacci/_action/get_result/_service_event'
  [INFO] [1744953225.731315612] [rosbag2_recorder]: Subscribed to topic '/fibonacci/_action/send_goal/_service_event'
  [INFO] [1744953225.735061252] [rosbag2_recorder]: Subscribed to topic '/fibonacci/_action/status'
  ...

现在 ``ros2 bag`` 正在记录 ``/fibonacci`` 动作的动作数据：目标、结果和反馈。
要停止录制，在终端中输入 :kbd:`Ctrl-C`。

数据将累积到一个新的 bag 目录中，名称格式为 ``rosbag2_year_month_day-hour_minute_second``。
此目录将包含 ``metadata.yaml`` 以及录制格式的 bag 文件。

4 检查动作数据
^^^^^^^^^^^^^^

您可以通过运行以下命令查看有关录制的详细信息：

.. code-block:: console

  $ ros2 bag info <bag_file_name>
  Files:             rosbag2_2025_04_17-22_20_40_0.mcap
  Bag size:          20.7 KiB
  Storage id:        mcap
  ROS Distro:        rolling
  Duration:          9.019568080s
  Start:             Apr 17 2025 22:20:47.263125070 (1744953647.263125070)
  End:               Apr 17 2025 22:20:56.282693150 (1744953656.282693150)
  Messages:          0
  Topic information:
  Services:          0
  Service information:
  Actions:           1
  Action information:
    Action: /fibonacci | Type: example_interfaces/action/Fibonacci | Topics: 2 | Service: 3 | Serialization Format: cdr
      Topic: feedback | Count: 9
      Topic: status | Count: 3
      Service: send_goal | Event Count: 4
      Service: cancel_goal | Event Count: 0
      Service: get_result | Event Count: 4

5 播放动作数据
^^^^^^^^^^^^^^

在重放 bag 文件之前，在运行 ``fibonacci_action_client`` 的终端中输入 :kbd:`Ctrl-C`。
当 ``fibonacci_action_client`` 停止运行时，``fibonacci_action_server`` 也会停止打印结果，因为没有传入的请求。

从 bag 文件重放动作数据将开始向 ``fibonacci_action_server`` 发送请求。

输入命令：

.. code-block:: console

  $ ros2 bag play --send-actions-as-client <bag_file_name>
  [INFO] [1744953720.691068674] [rosbag2_player]: Set rate to 1
  [INFO] [1744953720.702365209] [rosbag2_player]: Adding keyboard callbacks.
  [INFO] [1744953720.702409447] [rosbag2_player]: Press SPACE for Pause/Resume
  [INFO] [1744953720.702423063] [rosbag2_player]: Press CURSOR_RIGHT for Play Next Message
  [INFO] [1744953720.702431404] [rosbag2_player]: Press CURSOR_UP for Increase Rate 10%
  [INFO] [1744953720.702437677] [rosbag2_player]: Press CURSOR_DOWN for Decrease Rate 10%
  Progress bar enabled at 3 Hz.
  Progress bar [?]: [R]unning, [P]aused, [B]urst, [D]elayed, [S]topped
  [INFO] [1744953720.702577680] [rosbag2_player]: Playback until timestamp: -1


  ====== Playback Progress ======
  [1744953656.281683207] Duration 9.02/9.02 [R]

您的 ``fibonacci_action_server`` 终端将再次开始打印以下服务消息：

.. code-block:: console

  [INFO] [1744953720.815577088] [fibonacci_action_server]: Executing goal...
  [INFO] [1744953720.815927050] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1])
  [INFO] [1744953721.816509658] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2])
  [INFO] [1744953722.817220270] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3])
  [INFO] [1744953723.817876426] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3, 5])
  [INFO] [1744953724.818498515] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3, 5, 8])
  [INFO] [1744953725.819182228] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3, 5, 8, 13])
  [INFO] [1744953726.820032562] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3, 5, 8, 13, 21])
  [INFO] [1744953727.820738690] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3, 5, 8, 13, 21, 34])
  [INFO] [1744953728.821449308] [fibonacci_action_server]: Feedback: array('i', [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55])

这是因为 ``ros2 bag play`` 将 bag 文件中的动作目标请求数据发送到 ``/fibonacci`` 动作。

我们还可以在 ``ros2 bag play`` 播放时内省动作通信以验证 ``fibonacci_action_server``。

在 ``ros2 bag play`` 之前运行此命令以查看 ``fibonacci_action_server``。
您可以看到来自 bag 文件的动作目标请求和来自 ``fibonacci_action_server`` 的服务响应：

.. code-block:: console

  $ ros2 action echo --flow-style /fibonacci
  interface: STATUS_TOPIC
  status_list: [{goal_info: {goal_id: {uuid: [34, 116, 225, 217, 48, 121, 146, 36, 240, 98, 99, 134, 55, 227, 184, 72]}, stamp: {sec: 1744953720, nanosec: 804984321}}, status: 4}]
  ---
  interface: GOAL_SERVICE
  info:
    event_type: REQUEST_RECEIVED
    stamp:
      sec: 1744953927
      nanosec: 957359210
    client_gid: [1, 15, 165, 231, 190, 254, 1, 50, 0, 0, 0, 0, 0, 0, 19, 4]
    sequence_number: 1
  request: [{goal_id: {uuid: [191, 200, 153, 122, 221, 251, 152, 172, 60, 69, 94, 20, 212, 160, 40, 12]}, goal: {order: 10}}]
  response: []
  ---
  interface: GOAL_SERVICE
  info:
    event_type: RESPONSE_SENT
    stamp:
      sec: 1744953927
      nanosec: 957726145
    client_gid: [1, 15, 165, 231, 190, 254, 1, 50, 0, 0, 0, 0, 0, 0, 19, 4]
    sequence_number: 1
  request: []
  response: [{accepted: true, stamp: {sec: 1744953927, nanosec: 957615866}}]
  ---
  interface: STATUS_TOPIC
  status_list: [{goal_info: {goal_id: {uuid: [191, 200, 153, 122, 221, 251, 152, 172, 60, 69, 94, 20, 212, 160, 40, 12]}, stamp: {sec: 1744953927, nanosec: 957663383}}, status: 2}]
  ---
  interface: FEEDBACK_TOPIC
  goal_id:
    uuid: [191, 200, 153, 122, 221, 251, 152, 172, 60, 69, 94, 20, 212, 160, 40, 12]
  feedback:
    sequence: [0, 1, 1]
  ---
  ...

总结
----

您可以使用 ``ros2 bag`` 命令记录 ROS 2 系统中主题、服务和动作上传递的数据。
无论您是与他人分享工作还是内省自己的实验，这都是一个值得了解的好工具。

下一步
------

您已完成"初级：CLI 工具"教程！
下一步是攻克"初级：客户端库"教程，从 :doc:`../../Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace` 开始。

相关内容
--------

关于 ``ros2 bag`` 的更详细说明可以在 `这里 <https://github.com/ros2/rosbag2>`__ 的 README 中找到。
有关服务录制和播放的更多信息可以在 `这里 <https://github.com/ros2/rosbag2/blob/{DISTRO}/docs/design/rosbag2_record_replay_service.md>`__ 的设计文档中找到。
有关动作录制和播放的更多信息可以在 `这里 <https://github.com/ros2/rosbag2/blob/{DISTRO}/docs/design/rosbag2_record_replay_action.md>`__ 的设计文档中找到。
有关 QoS 兼容性和 ``ros2 bag`` 的更多信息，请参阅 :doc:`../../../How-To-Guides/Overriding-QoS-Policies-For-Recording-And-Playback`。
