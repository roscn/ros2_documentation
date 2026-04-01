.. redirect-from::

    Tutorials/Parameters/Understanding-ROS2-Parameters

.. _ROS2Params:

理解参数
=========

**目标：** 学习如何在 ROS 2 中获取、设置、保存和重新加载参数。

**教程级别：** 初级

**时间：** 5 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

参数是节点的配置值。
您可以将参数视为节点设置。
节点可以将参数存储为整数、浮点数、布尔值、字符串和列表。
在 ROS 2 中，每个节点维护自己的参数。
有关参数的更多背景信息，请参阅 :doc:`概念文档 <../../../Concepts/Basic/About-Parameters>`。

前提条件
---------

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


2 ros2 param list
^^^^^^^^^^^^^^^^^

要查看属于您节点的参数，打开一个新终端并输入命令：

.. code-block:: console

  $ ros2 param list
  /teleop_turtle:
    qos_overrides./parameter_events.publisher.depth
    qos_overrides./parameter_events.publisher.durability
    qos_overrides./parameter_events.publisher.history
    qos_overrides./parameter_events.publisher.reliability
    scale_angular
    scale_linear
    use_sim_time
  /turtlesim:
    background_b
    background_g
    background_r
    qos_overrides./parameter_events.publisher.depth
    qos_overrides./parameter_events.publisher.durability
    qos_overrides./parameter_events.publisher.history
    qos_overrides./parameter_events.publisher.reliability
    use_sim_time


您可以看到节点命名空间 ``/teleop_turtle`` 和 ``/turtlesim``，后面是每个节点的参数。

参数的命名空间和其名称使用点分隔，例如 ``parameter_events.publisher.depth``。

每个节点都有 ``use_sim_time`` 参数；它不是 turtlesim 特有的。

根据它们的名称，``/turtlesim`` 的参数似乎使用 RGB 颜色值决定 turtlesim 窗口的背景颜色。

要确定参数的类型，您可以使用 ``ros2 param get``。


3 ros2 param get
^^^^^^^^^^^^^^^^

要显示参数的类型和当前值，请使用命令：

.. code-block:: console

  $ ros2 param get <node_name> <parameter_name>

您也可以通过省略节点名称来查询所有节点中的参数：

.. code-block:: console

  $ ros2 param get <parameter_name>

让我们找出 ``/turtlesim`` 参数 ``background_g`` 的当前值：

.. code-block:: console

  $ ros2 param get /turtlesim background_g
  Integer value is: 86

现在您知道 ``background_g`` 保存一个整数值。

如果您在 ``background_r`` 和 ``background_b`` 上运行相同的命令，您将分别获得值 ``69`` 和 ``255``。

您也可以检查所有节点中的 ``use_sim_time`` 参数：

.. code-block:: console

  $ ros2 param get use_sim_time

这将显示设置了此参数的每个节点的参数值。

4 ros2 param set
^^^^^^^^^^^^^^^^

要在运行时更改参数的值，请使用命令：

.. code-block:: console

  $ ros2 param set <node_name> <parameter_name> <value>

让我们更改 ``/turtlesim`` 的背景颜色：

.. code-block:: console

  $ ros2 param set /turtlesim background_r 150
  Set parameter successful

您的 turtlesim 窗口的背景应该会改变颜色：

.. image:: images/set.png

使用 ``set`` 命令设置参数只会更改当前会话中的参数，而不是永久更改。
但是，您可以保存设置并在下次启动节点时重新加载它们。

5 ros2 param dump
^^^^^^^^^^^^^^^^^

您可以使用以下命令查看节点所有当前参数值：

.. code-block:: console

  $ ros2 param dump <node_name>

该命令默认打印到标准输出（stdout），但您也可以将参数值重定向到文件中以保存以备后用。
要将 ``/turtlesim`` 参数的当前配置保存到文件 ``turtlesim.yaml`` 中，输入命令：

.. code-block:: console

  $ ros2 param dump /turtlesim > turtlesim.yaml

您将在 shell 运行的当前工作目录中找到一个新文件。
如果您打开此文件，您将看到以下内容：

.. code-block:: YAML

  /turtlesim:
    ros__parameters:
      background_b: 255
      background_g: 86
      background_r: 150
      qos_overrides:
        /parameter_events:
          publisher:
            depth: 1000
            durability: volatile
            history: keep_last
            reliability: reliable
      use_sim_time: false

如果您将来想用相同的参数重新加载节点，转储参数会很方便。

6 ros2 param load
^^^^^^^^^^^^^^^^^

您可以使用以下命令从文件向当前运行的节点加载参数：

.. code-block:: console

  $ ros2 param load <node_name> <parameter_file>

要将使用 ``ros2 param dump`` 生成的 ``turtlesim.yaml`` 文件加载到 ``/turtlesim`` 节点的参数中，输入命令：

.. code-block:: console

  $ ros2 param load /turtlesim turtlesim.yaml
  Set parameter background_b successful
  Set parameter background_g successful
  Set parameter background_r successful
  Set parameter qos_overrides./parameter_events.publisher.depth failed: parameter 'qos_overrides./parameter_events.publisher.depth' cannot be set because it is read-only
  Set parameter qos_overrides./parameter_events.publisher.durability failed: parameter 'qos_overrides./parameter_events.publisher.durability' cannot be set because it is read-only
  Set parameter qos_overrides./parameter_events.publisher.history failed: parameter 'qos_overrides./parameter_events.publisher.history' cannot be set because it is read-only
  Set parameter qos_overrides./parameter_events.publisher.reliability failed: parameter 'qos_overrides./parameter_events.publisher.reliability' cannot be set because it is read-only
  Set parameter use_sim_time successful

.. note::

  只读参数只能在启动时修改，之后不能修改，这就是为什么"qos_overrides"参数有一些警告。

.. _LoadParameterFileOnNodeStartup:

7 在节点启动时加载参数文件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

要使用保存的参数值启动相同的节点，请使用：

.. code-block:: console

  $ ros2 run <package_name> <executable_name> --ros-args --params-file <file_name>

这与您用来启动 turtlesim 的命令相同，只是添加了标志 ``--ros-args`` 和 ``--params-file``，后跟您要加载的文件。

停止您运行的 turtlesim 节点，并尝试使用保存的参数重新加载它，使用：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node --ros-args --params-file turtlesim.yaml

turtlesim 窗口应该照常出现，但具有您之前设置的紫色背景。

.. note::

  当在节点启动时使用参数文件时，所有参数，包括只读参数，都将被更新。

总结
----

节点具有参数来定义其默认配置值。
您可以从命令行 ``get`` 和 ``set`` 参数值。
您还可以将参数设置保存到文件中，以便在以后的会话中重新加载。

下一步
------

回到 ROS 2 通信方法，在下一篇教程中，您将学习 :doc:`动作 <../Understanding-ROS2-Actions/Understanding-ROS2-Actions>`。
