.. redirect-from::

    Tutorials/Configuring-ROS2-Environment

.. _ConfigROS2:

Configuring environment
=======================

**目标：** 本教程将向您展示如何准备 ROS 2 环境。

**教程级别：** 初级

**时间：** 5 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

ROS 2 依赖于使用 shell 环境组合工作空间的概念。
"工作空间"是 ROS 术语，指的是您在系统上使用 ROS 2 进行开发的位置。
核心 ROS 2 工作空间称为底层（underlay）。
后续的本地工作空间称为覆盖层（overlay）。
在使用 ROS 2 开发时，您通常会同时激活多个工作空间。

组合工作空间使针对不同版本的 ROS 2 或不同软件包集合进行开发变得更加容易。
它还允许在同一台计算机上安装多个 ROS 2 发行版（或"distros"，例如 Dashing 和 Eloquent）并在它们之间切换。

这是通过每次打开新 shell 时 source 设置文件，或者将 source 命令一次性添加到 shell 启动脚本来实现的。
如果不 source 设置文件，您将无法访问 ROS 2 命令，也无法找到或使用 ROS 2 软件包。
换句话说，您将无法使用 ROS 2。

前提条件
--------

在开始这些教程之前，请按照 ROS 2 :doc:`../../Installation` 页面上的说明安装 ROS 2。

本教程中使用的命令假设您按照操作系统的二进制包安装指南进行操作（Linux 为 deb 包）。
如果您是从源代码构建的，仍然可以继续，但设置文件的路径可能会有所不同。
如果您从源代码安装，您也将无法使用 ``sudo apt install ros-<distro>-<package>`` 命令（在初级教程中经常使用）。

如果您使用的是 Linux 或 macOS，但还不熟悉 shell，`这个教程 <https://www.linux.com/training-tutorials/bash-101-working-cli/>`__ 会对您有所帮助。

任务
----

1 Source 设置文件
^^^^^^^^^^^^^^^^^^

您需要在打开的每个新 shell 中运行此命令才能访问 ROS 2 命令，如下所示：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ source /opt/ros/{DISTRO}/setup.bash


      如果您不使用 bash，请将 ``.bash`` 替换为您的 shell。
      可能的值有：``setup.bash``、``setup.sh``、``setup.zsh``。

   .. group-tab:: macOS

      .. code-block:: console

        $ . ~/ros2_install/ros2-osx/setup.bash

   .. group-tab:: Windows

      .. code-block:: console

        $ call C:\dev\ros2\local_setup.bat

.. note::
    具体命令取决于您安装 ROS 2 的位置。
    如果遇到问题，请确保文件路径指向您的安装位置。

2 将 source 命令添加到 shell 启动脚本
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果您不想每次打开新 shell 时都 source 设置文件（跳过任务 1），可以将命令添加到 shell 启动脚本中：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ echo "source /opt/ros/{DISTRO}/setup.bash" >> ~/.bashrc

      要撤销此操作，请找到系统的 shell 启动脚本并删除追加的 source 命令。

   .. group-tab:: macOS

      .. code-block:: console

        $ echo "source ~/ros2_install/ros2-osx/setup.bash" >> ~/.bash_profile

      要撤销此操作，请找到系统的 shell 启动脚本并删除追加的 source 命令。

   .. group-tab:: Windows

      仅适用于 PowerShell 用户，在"我的文档"中创建一个名为"WindowsPowerShell"的文件夹。
      在"WindowsPowerShell"中，创建文件"Microsoft.PowerShell_profile.ps1"。
      在文件中粘贴：

      .. code-block:: console

        $ C:\dev\ros2_{DISTRO}\local_setup.ps1

      PowerShell 每次打开新 shell 时都会请求运行此脚本的权限。
      要避免此问题，您可以运行：

      .. code-block:: console

        $ Unblock-File C:\dev\ros2_{DISTRO}\local_setup.ps1

      要撤销此操作，请删除新的"Microsoft.PowerShell_profile.ps1"文件。

3 检查环境变量
^^^^^^^^^^^^^^

Source ROS 2 设置文件将设置运行 ROS 2 所需的多个环境变量。
如果您在查找或使用 ROS 2 软件包时遇到问题，请确保使用以下命令正确设置了环境：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ printenv | grep -i ROS

   .. group-tab:: macOS

      .. code-block:: console

        $ printenv | grep -i ROS

   .. group-tab:: Windows

      .. code-block:: console

        $ set | findstr -i ROS

检查 ``ROS_DISTRO`` 和 ``ROS_VERSION`` 等变量是否已设置。

::

  ROS_VERSION=2
  ROS_PYTHON_VERSION=3
  ROS_DISTRO={DISTRO}

如果环境变量设置不正确，请返回您遵循的安装指南中的 ROS 2 软件包安装部分。
如果您需要更具体的帮助（因为环境设置文件可能来自不同的地方），您可以从社区`获取答案 <https://robotics.stackexchange.com/>`__。

3.1 ``ROS_DOMAIN_ID`` 变量
~~~~~~~~~~~~~~~~~~~~~~~~~~

有关 ROS 域 ID 的详细信息，请参阅 `域 ID <../../Concepts/Intermediate/About-Domain-ID>` 文章。

确定 ROS 2 节点组的唯一整数后，可以使用以下命令设置环境变量：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ export ROS_DOMAIN_ID=<your_domain_id>

      要在 shell 会话之间保持此设置，可以将命令添加到 shell 启动脚本中：

      .. code-block:: console

        $ echo "export ROS_DOMAIN_ID=<your_domain_id>" >> ~/.bashrc

   .. group-tab:: macOS

      .. code-block:: console

        $ export ROS_DOMAIN_ID=<your_domain_id>

      要在 shell 会话之间保持此设置，可以将命令添加到 shell 启动脚本中：

      .. code-block:: console

        $ echo "export ROS_DOMAIN_ID=<your_domain_id>" >> ~/.bash_profile

   .. group-tab:: Windows

      .. code-block:: console

        $ set ROS_DOMAIN_ID=<your_domain_id>

      如果您想在 shell 会话之间使其永久生效，还可以运行：

      .. code-block:: console

        $ setx ROS_DOMAIN_ID <your_domain_id>

3.2 ``ROS_AUTOMATIC_DISCOVERY_RANGE`` 变量
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

默认情况下，ROS 2 通信不限于本地主机。
``ROS_AUTOMATIC_DISCOVERY_RANGE`` 环境变量允许您限制 ROS 2 发现范围。
使用 ``ROS_AUTOMATIC_DISCOVERY_RANGE`` 在某些情况下很有帮助，例如在教室中，多个机器人可能发布到同一主题，导致异常行为。
有关更多详细信息，请参阅 :ref:`改进的动态发现 <ImprovedDynamicDiscovery>`。

总结
----

ROS 2 开发环境在使用前需要正确配置。
这可以通过两种方式完成：在打开的每个新 shell 中 source 设置文件，或将 source 命令添加到启动脚本中。

如果您在使用 ROS 2 定位或使用软件包时遇到任何问题，首先应该检查环境变量，并确保它们设置为您预期的版本和发行版。

下一步
------

现在您已经有了一个可用的 ROS 2 安装，并且知道如何 source 其设置文件，您可以开始使用 :doc:`turtlesim 工具 <./Introducing-Turtlesim/Introducing-Turtlesim>` 学习 ROS 2 的方方面面。
