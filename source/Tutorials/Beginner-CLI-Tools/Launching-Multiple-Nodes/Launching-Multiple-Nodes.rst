.. redirect-from::

    Tutorials/Launch/CLI-Intro

.. _ROS2Launch:

启动节点
========

**目标：** 使用命令行工具一次性启动多个节点。

**教程级别：** 初级

**时间：** 5 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

在大多数入门教程中，您一直为每个新节点打开新的终端。
随着您创建更复杂的系统，同时运行越来越多的节点，打开终端和重新输入配置细节变得乏味。

启动文件允许您同时启动和配置多个包含 ROS 2 节点的可执行文件。

使用 ``ros2 launch`` 命令运行单个启动文件将一次性启动您的整个系统——所有节点及其配置。

前提条件
---------

在开始这些教程之前，请按照 ROS 2 :doc:`../../../Installation/` 页面上的说明安装 ROS 2。

本教程中使用的命令假设您遵循了操作系统的二进制包安装指南（Linux 的 deb 包）。
如果您从源代码构建，您仍然可以继续，但设置文件的路径可能会有所不同。
如果您从源代码安装，您也无法使用 ``sudo apt install ros-<distro>-<package>`` 命令（在初级教程中经常使用）。

如果您使用 Linux 并且还不熟悉 shell，`这个教程 <https://www.linux.com/training-tutorials/bash-101-working-cli/>`__ 会对您有所帮助。

一如既往，不要忘记在 :doc:`打开的每个新终端 <../Configuring-ROS2-Environment>` 中 source ROS 2。

任务
----

运行启动文件
^^^^^^^^^^^^

打开一个新终端并运行：

.. code-block:: console

   $ ros2 launch turtlesim multisim.launch.py

此命令将运行以下启动文件：

.. literalinclude:: launch/multisim.launch.py
   :language: python

.. note::

  上面的启动文件是用 Python 编写的，但您也可以使用 XML 和 YAML 创建启动文件。
  您可以在 :doc:`../../../How-To-Guides/Launch-file-different-formats` 中看到这些不同 ROS 2 启动格式的比较。

这将运行两个 turtlesim 节点：

.. image:: images/turtlesim_multisim.png

暂时不用担心这个启动文件的内容。
您可以在 :doc:`ROS 2 启动教程 <../../Intermediate/Launch/Launch-Main>` 中找到更多关于 ROS 2 启动的信息。

（可选）控制 Turtlesim 节点
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

现在这些节点正在运行，您可以像控制任何其他 ROS 2 节点一样控制它们。
例如，您可以通过打开两个额外的终端并运行以下命令让海龟朝相反的方向移动：

在第二个终端中：

.. code-block:: console

   $ ros2 topic pub  /turtlesim1/turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

在第三个终端中：

.. code-block:: console

   $ ros2 topic pub  /turtlesim2/turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: -1.8}}"

运行这些命令后，您应该会看到类似以下的内容：

.. image:: images/turtlesim_multisim_spin.png

总结
----

到目前为止，您所做的重要之处在于使用一个命令运行了两个 turtlesim 节点。
一旦您学会编写自己的启动文件，您将能够使用 ``ros2 launch`` 命令以类似的方式运行多个节点——并设置它们的配置。

有关 ROS 2 启动文件的更多教程，请参阅 :doc:`主启动文件教程页面<../../Intermediate/Launch/Launch-Main>`。

下一步
------

在下一篇教程中，:doc:`../Recording-And-Playing-Back-Data/Recording-And-Playing-Back-Data`，您将了解另一个有用的工具 ``ros2 bag``。
