.. redirect-from::

    Tutorials/Turtlesim/Introducing-Turtlesim

.. _Turtlesim:

使用 ``turtlesim``、``ros2`` 和 ``rqt``
==========================================

**目标：** 安装并使用 turtlesim 软件包和 rqt 工具，为接下来的教程做准备。

**教程级别：** 初级

**时间：** 15 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

Turtlesim 是一个用于学习 ROS 2 的轻量级模拟器。
它展示了 ROS 2 在最基本层面上所做的工作，让您了解以后使用真实机器人或机器人仿真时将要做什么。

ros2 工具是用户管理、检查和与 ROS 系统交互的方式。
它支持多个命令，针对系统及其操作的不同方面。
您可以用它来启动节点、设置参数、监听主题等等。
ros2 工具是 ROS 2 核心安装的一部分。

rqt 是 ROS 2 的图形用户界面（GUI）工具。
在 rqt 中完成的所有操作都可以在命令行中完成，但 rqt 提供了一种更友好的方式来操作 ROS 2 元素。

本教程涉及 ROS 2 的核心概念，如节点、主题和服务。
所有这些概念将在后续教程中详细阐述；目前，您只需设置工具并熟悉它们。

前提条件
---------

之前的教程 :doc:`../Configuring-ROS2-Environment` 将向您展示如何设置环境。

任务
----

1 安装 turtlesim
^^^^^^^^^^^^^^^^^

一如既往，首先在新终端中 source 您的设置文件，如 :doc:`之前的教程 <../Configuring-ROS2-Environment>` 中所述。

为您的 ROS 2 发行版安装 turtlesim 软件包：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ sudo apt update
        $ sudo apt install ros-{DISTRO}-turtlesim

   .. group-tab:: macOS

      只要您安装 ROS 2 的归档文件包含 ``ros_tutorials`` 仓库，您应该已经安装了 turtlesim。

   .. group-tab:: Windows

      只要您安装 ROS 2 的归档文件包含 ``ros_tutorials`` 仓库，您应该已经安装了 turtlesim。

要检查软件包是否已安装，请运行以下命令，该命令应返回 turtlesim 可执行文件的列表：

.. code-block:: console

  $ ros2 pkg executables turtlesim
  turtlesim draw_square
  turtlesim mimic
  turtlesim turtle_teleop_key
  turtlesim turtlesim_node

2 启动 turtlesim
^^^^^^^^^^^^^^^^^

要启动 turtlesim，请在终端中输入以下命令：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node
  [INFO] [turtlesim]: Starting turtlesim with node name /turtlesim
  [INFO] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]

在命令下方，您将看到来自节点的消息。
在那里您可以看到默认海龟的名称和它生成的坐标。

模拟器窗口应该会出现，中间有一只随机的海龟。

.. image:: images/turtlesim.png


3 使用 turtlesim
^^^^^^^^^^^^^^^^^

打开一个新终端并再次 source ROS 2。

现在您将运行一个新节点来控制第一个节点中的海龟：

.. code-block:: console

  $ ros2 run turtlesim turtle_teleop_key

此时您应该有三个打开的窗口：一个运行 ``turtlesim_node`` 的终端、一个运行 ``turtle_teleop_key`` 的终端和 turtlesim 窗口。
排列这些窗口，以便您可以看到 turtlesim 窗口，同时保持运行 ``turtle_teleop_key`` 的终端处于活动状态，这样您就可以控制 turtlesim 中的海龟。

使用键盘上的箭头键控制海龟。
它将在屏幕上移动，使用其附带的"画笔"绘制到目前为止所经过的路径。

.. note::

  按下箭头键只会使海龟移动一小段距离然后停止。
  这是因为，实际上，如果操作员与机器人的连接丢失，您不希望机器人继续执行指令。

您可以使用相应命令的 ``list`` 子命令查看节点及其关联的主题、服务和动作：

.. code-block:: console

  $ ros2 node list
  $ ros2 topic list
  $ ros2 service list
  $ ros2 action list

您将在接下来的教程中了解更多关于这些概念的知识。
由于本教程的目标只是对 turtlesim 进行总体概述，您将使用 rqt 调用一些 turtlesim 服务并与 ``turtlesim_node`` 交互。

4 安装 rqt
^^^^^^^^^^

打开一个新终端来安装 ``rqt`` 及其插件：

.. tabs::

  .. group-tab:: Ubuntu

    .. code-block:: console

      $ sudo apt update
      $ sudo apt install '~nros-{DISTRO}-rqt*'

  .. group-tab:: RHEL

    .. code-block:: console

      $ sudo dnf install 'ros-{DISTRO}-rqt*'

  .. group-tab:: macOS

    在 macOS 上安装 ROS 2 的标准归档文件包含 ``rqt`` 及其插件，因此您应该已经安装了 ``rqt``。

  .. group-tab:: Windows

    在 Windows 上安装 ROS 2 的标准归档文件包含 ``rqt`` 及其插件，因此您应该已经安装了 ``rqt``。

要运行 rqt：

.. code-block:: console

  $ rqt

5 使用 rqt
^^^^^^^^^^

首次运行 rqt 时，窗口将是空白的。
不用担心；只需从顶部菜单栏中选择 **Plugins** > **Services** > **Service Caller**。

.. note::

  rqt 可能需要一些时间来定位所有插件。
  如果您点击 **Plugins** 但看不到 **Services** 或任何其他选项，您应该关闭 rqt 并在终端中输入命令 ``rqt --force-discover``。

.. image:: images/rqt.png

使用 **Service** 下拉列表左侧的刷新按钮，确保 turtlesim 节点的所有服务都可用。

点击 **Service** 下拉列表查看 turtlesim 的服务，然后选择 ``/spawn`` 服务。

5.1 尝试 spawn 服务
~~~~~~~~~~~~~~~~~~~

让我们使用 rqt 调用 ``/spawn`` 服务。
从它的名称可以猜出，``/spawn`` 将在 turtlesim 窗口中创建另一只海龟。

通过双击 **Expression** 列中空的单引号之间，为新海龟指定一个唯一的名称，例如 ``turtle2``。
您可以看到此表达式对应于 **name** 的值，类型为 **string**。

接下来输入一些有效的坐标来生成新海龟，例如 ``x = 1.0`` 和 ``y = 1.0``。

.. image:: images/spawn.png

.. note::

  如果您尝试生成一个与现有海龟同名的新海龟，例如默认的 ``turtle1``，您将在运行 ``turtlesim_node`` 的终端中收到一条错误消息：

  .. code-block:: console

    [ERROR] [turtlesim]: A turtle named [turtle1] already exists

要生成 ``turtle2``，您需要点击 rqt 窗口右上角的 **Call** 按钮来调用服务。

如果服务调用成功，您应该看到一只新海龟（同样具有随机设计）在您为 **x** 和 **y** 输入的坐标处生成。

如果您刷新 rqt 中的服务列表，您还会看到现在有与新海龟相关的服务 ``/turtle2/...``，除了 ``/turtle1/...``。

5.2 尝试 set_pen 服务
~~~~~~~~~~~~~~~~~~~~~

现在让我们使用 ``/set_pen`` 服务为 ``turtle1`` 设置一个独特的画笔：

.. image:: images/set_pen.png

**r**、**g** 和 **b** 的值在 0 到 255 之间，设置 ``turtle1`` 绘制的画笔颜色，**width** 设置线条的粗细。

要让 ``turtle1`` 用明显的红色线条绘制，将 **r** 的值更改为 255，将 **width** 的值更改为 5。
更新值后别忘了调用服务。

如果您返回运行 ``turtle_teleop_key`` 的终端并按箭头键，您将看到 ``turtle1`` 的画笔已更改。

.. image:: images/new_pen.png

您可能也注意到了，没有办法移动 ``turtle2``。
那是因为 ``turtle2`` 没有遥操作节点。

6 重映射
^^^^^^^^

您需要第二个遥操作节点来控制 ``turtle2``。
但是，如果您尝试运行与之前相同的命令，您会注意到这个命令也控制 ``turtle1``。
改变这种行为的方法是重映射 ``cmd_vel`` 主题。

在一个新终端中，source ROS 2，然后运行：

.. code-block:: console

  $ ros2 run turtlesim turtle_teleop_key --ros-args --remap turtle1/cmd_vel:=turtle2/cmd_vel


现在，当此终端处于活动状态时，您可以移动 ``turtle2``，当另一个运行 ``turtle_teleop_key`` 的终端处于活动状态时，可以移动 ``turtle1``。

.. image:: images/remap.png

7 关闭 turtlesim
^^^^^^^^^^^^^^^^

要停止模拟，您可以在 ``turtlesim_node`` 终端中输入 ``Ctrl + C``，在 ``turtle_teleop_key`` 终端中输入 ``q``。

总结
----

使用 turtlesim 和 rqt 是学习 ROS 2 核心概念的好方法。

下一步
------

现在您已经启动并运行了 turtlesim 和 rqt，并且了解了它们的工作原理，让我们通过下一个教程 :doc:`../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes` 深入了解第一个 ROS 2 核心概念。

相关内容
--------

turtlesim 软件包可以在 `ros_tutorials <https://github.com/ros/ros_tutorials/tree/{REPOS_FILE_BRANCH}/turtlesim>`_ 仓库中找到。
