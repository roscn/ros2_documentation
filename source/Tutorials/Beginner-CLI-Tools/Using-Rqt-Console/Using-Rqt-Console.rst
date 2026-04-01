.. redirect-from::

    Tutorials/Rqt-Console/Using-Rqt-Console

.. _rqt_console:

使用 ``rqt_console`` 查看日志
=============================

**目标：** 了解 ``rqt_console``，一个用于检查日志消息的工具。

**教程级别：** 初级

**时间：** 5 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

``rqt_console`` 是一个用于检查 ROS 2 中日志消息的 GUI 工具。
通常，日志消息会显示在您的终端中。
使用 ``rqt_console``，您可以随时间收集这些消息，以更详细和有组织的方式查看它们，过滤它们，保存它们，甚至重新加载保存的文件以在不同时间进行检查。

节点使用日志以各种方式输出有关事件和状态的消息。
它们的内容通常是为了用户的信息。

前提条件
---------

您需要安装 :doc:`rqt_console 和 turtlesim <../Introducing-Turtlesim/Introducing-Turtlesim>`。

一如既往，不要忘记在 :doc:`打开的每个新终端 <../Configuring-ROS2-Environment>` 中 source ROS 2。


任务
----

1 设置
^^^^^^

在新终端中使用以下命令启动 ``rqt_console``：

.. code-block:: console

    $ ros2 run rqt_console rqt_console

``rqt_console`` 窗口将打开：

.. image:: images/console.png

控制台的第一部分是系统日志消息显示的地方。

中间部分您可以选择通过排除严重级别来过滤消息。
您还可以使用右侧的加号按钮添加更多排除过滤器。

底部部分用于高亮显示包含您输入字符串的消息。
您也可以向此部分添加更多过滤器。

现在在新终端中使用以下命令启动 ``turtlesim``：

.. code-block:: console

    $ ros2 run turtlesim turtlesim_node

2 rqt_console 上的消息
^^^^^^^^^^^^^^^^^^^^^^

要生成 ``rqt_console`` 显示的日志消息，让我们让海龟撞墙。
在新终端中，输入下面的 ``ros2 topic pub`` 命令（在 :doc:`主题教程 <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>` 中详细讨论）：

.. code-block:: console

    $ ros2 topic pub -r 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0,y: 0.0,z: 0.0}}"

由于上述命令以稳定的速率发布主题，海龟持续撞墙。
在 ``rqt_console`` 中，您将看到相同的消息以 ``Warn`` 严重级别反复显示，如下所示：

.. image:: images/warn.png

在运行 ``ros2 topic pub`` 命令的终端中按 ``Ctrl+C`` 以停止海龟撞墙。

3 日志级别
^^^^^^^^^^

ROS 2 的日志级别按严重程度排序：

 1. Fatal
 2. Error
 3. Warn
 4. Info
 5. Debug

每个级别表示什么没有确切的标准，但可以安全地假设：

* ``Fatal`` 消息表示系统将终止以试图保护自己免受损害。
* ``Error`` 消息表示不一定会损坏系统但阻止其正常运行的重大问题。
* ``Warn`` 消息表示可能代表更深层次问题的意外活动或不理想结果，但不会直接损害功能。
* ``Info`` 消息表示作为系统按预期运行的可视验证的事件和状态更新。
* ``Debug`` 消息详细说明系统执行的整个逐步过程。

默认级别是 ``Info``。
您只会看到默认严重级别和更严重级别的消息。

通常，只有 ``Debug`` 消息被隐藏，因为它们是唯一比 ``Info`` 严重程度低的级别。
例如，如果将默认级别设置为 ``Warn``，您只会看到严重级别为 ``Warn``、``Error`` 和 ``Fatal`` 的消息。

3.1 设置默认日志级别
~~~~~~~~~~~~~~~~~~~~

您可以在首次运行 ``/turtlesim`` 节点时使用重映射设置默认日志级别。
在终端中输入以下命令：

.. code-block:: console

    $ ros2 run turtlesim turtlesim_node --ros-args --log-level WARN

现在您将看不到上次启动 ``turtlesim`` 时控制台中出现的初始 ``Info`` 级别消息。
这是因为 ``Info`` 消息的优先级低于新的默认严重级别 ``Warn``。

总结
----

如果您需要仔细检查系统中的日志消息，``rqt_console`` 会非常有帮助。
您可能出于多种原因需要检查日志消息，通常是找出问题所在以及导致该问题的一系列事件。

下一步
------

下一篇教程将教您如何使用 :doc:`ROS 2 Launch <../Launching-Multiple-Nodes/Launching-Multiple-Nodes>` 一次启动多个节点。
