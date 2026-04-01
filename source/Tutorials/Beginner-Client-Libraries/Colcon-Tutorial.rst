.. _Colcon:

.. redirect-from::

    Colcon-Tutorial
    Tutorials/Colcon-Tutorial

使用 ``colcon`` 构建软件包
==========================

.. contents:: 目录
   :depth: 2
   :local:

**目标：** 使用 ``colcon`` 构建 ROS 2 工作空间。

**教程级别：** 初级

**时间：** 20 分钟

这是一个关于如何使用 ``colcon`` 创建和构建 ROS 2 工作空间的简短教程。
这是一个实践教程，不是为了替代核心文档。

背景
----

``colcon`` 是 ROS 构建工具 ``catkin_make``、``catkin_make_isolated``、``catkin_tools`` 和 ``ament_tools`` 的迭代版本。
有关 colcon 设计的更多信息，请参阅 `此文档 <https://design.ros2.org/articles/build_tool.html>`__。

源代码可以在 `colcon GitHub 组织 <https://github.com/colcon>`__ 中找到。

Prerequisites
-------------

Install colcon
^^^^^^^^^^^^^^

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

        $ sudo apt install python3-colcon-common-extensions

  .. group-tab:: macOS

    .. code-block:: console

        $ python3 -m pip install colcon-common-extensions

  .. group-tab:: Windows

    .. code-block:: console

        $ pip install -U colcon-common-extensions


安装 ROS 2
^^^^^^^^^^^

要构建示例，您需要安装 ROS 2。

按照 :doc:`安装说明 <../../Installation>` 进行操作。

.. attention:: 如果从 deb 包安装，本教程需要 :ref:`桌面版安装 <linux-install-debs-install-ros-2-packages>`。

基础
----

ROS 工作空间是一个具有特定结构的目录。
通常有一个 ``src`` 子目录。
在该子目录中是 ROS 包的源代码所在位置。
通常该目录开始时是空的。

colcon 执行源外构建。
默认情况下，它将在 ``src`` 目录的同级位置创建以下目录：

* ``build`` 目录是存储中间文件的地方。
  对于每个包，将创建一个子文件夹，在其中调用 CMake 等。
* ``install`` 目录是每个包安装的位置。
  默认情况下，每个包将安装到单独的子目录中。
* ``log`` 目录包含有关每次 colcon 调用的各种日志信息。

.. note:: 与 catkin 相比，没有 ``devel`` 目录。

创建工作空间
^^^^^^^^^^^^

首先，创建一个目录（``ros2_ws``）来包含我们的工作空间：

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

       $ mkdir -p ~/ros2_ws/src
       $ cd ~/ros2_ws

  .. group-tab:: macOS

    .. code-block:: console

       $ mkdir -p ~/ros2_ws/src
       $ cd ~/ros2_ws

  .. group-tab:: Windows

    .. code-block:: console

       $ md \dev\ros2_ws\src
       $ cd \dev\ros2_ws

此时工作空间包含一个空的 ``src`` 目录：

.. code-block:: bash

    .
    └── src

    1 directory, 0 files

添加一些源代码
^^^^^^^^^^^^^^

让我们将 `examples <https://github.com/ros2/examples>`__ 仓库克隆到工作空间的 ``src`` 目录中：

.. code-block:: console

    $ git clone https://github.com/ros2/examples src/examples -b {REPOS_FILE_BRANCH}

现在工作空间应该有 ROS 2 示例的源代码：

.. code-block:: bash

    .
    └── src
        └── examples
            ├── CONTRIBUTING.md
            ├── LICENSE
            ├── rclcpp
            ├── rclpy
            └── README.md

    4 directories, 3 files

Source 底层环境
^^^^^^^^^^^^^^^

重要的是我们要 source 现有 ROS 2 安装的环境，它将为我们的工作空间提供示例包所需的构建依赖。
这是通过 source 二进制安装或源代码安装提供的设置脚本来实现的，即另一个 colcon 工作空间（参见 :doc:`安装 <../../Installation>`）。
我们将此环境称为 **底层（underlay）**。

我们的工作空间 ``ros2_ws`` 将是现有 ROS 2 安装之上的 **覆盖层（overlay）**。
通常，当您计划对少量包进行迭代时，建议使用覆盖层，而不是将所有包放在同一个工作空间中。

构建工作空间
^^^^^^^^^^^^

.. attention::

   要在 Windows 上构建包，您需要在 Visual Studio 环境中，有关更多详细信息，请参阅 :ref:`构建 ROS 2 代码 <windows-dev-build-ros2>`。

在工作空间的根目录下，运行 ``colcon build``。
由于 ``ament_cmake`` 等构建类型不支持 ``devel`` 空间的概念，并且需要安装包，colcon 支持 ``--symlink-install`` 选项。
这允许通过更改 ``source`` 空间中的文件（例如 Python 文件或其他非编译资源）来更改已安装的文件，以便更快地迭代。

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ colcon build --symlink-install

  .. group-tab:: macOS

    .. code-block:: console

      $ colcon build --symlink-install

  .. group-tab:: Windows

    .. code-block:: console

      $ colcon build --merge-install

    Windows 不允许长路径，因此 ``merge-install`` 会将所有路径合并到 ``install`` 目录中。
    在 Windows 上，您需要特殊权限才能创建符号链接，因此默认情况下不使用 ``--symlink-install``。
    要使用它，您需要以管理员身份运行命令或在系统设置中启用开发者模式。

.. tip::

   运行 ``colcon build`` 可能会冻结受 CPU、RAM 和 I/O 限制的系统（例如 Raspberry Pi）的屏幕和鼠标，因此使用 ``--executor sequential`` 参数逐个构建包而不是使用并行可能会很有用。
   有关更多参数，请参阅 `colcon 文档 <https://colcon.readthedocs.io/en/released/reference/executor-arguments.html>`_。

构建完成后，我们应该看到 ``build``、``install`` 和 ``log`` 目录：

.. code-block:: bash

    .
    ├── build
    ├── install
    ├── log
    └── src

    4 directories, 0 files

.. _colcon-run-the-tests:

运行测试
^^^^^^^^

要对我们刚刚构建的包运行测试，运行以下命令：

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ colcon test

  .. group-tab:: macOS

    .. code-block:: console

      $ colcon test

  .. group-tab:: Windows

    请记住使用 ``x64 Native Tools Command Prompt for VS 2019`` 执行以下命令，因为我们要构建工作空间。

    .. code-block:: console

      $ colcon test --merge-install

    您还需要在这里指定 ``--merge-install``，因为我们在上面的构建中使用了它。

.. _colcon-tutorial-source-the-environment:

Source 环境
^^^^^^^^^^^

当 colcon 成功完成构建后，输出将在 ``install`` 目录中。
在使用任何已安装的可执行文件或库之前，您需要将它们添加到您的路径和库路径中。
colcon 将在 ``install`` 目录中生成 bash/bat 文件以帮助设置环境。
这些文件将把所有必需的元素添加到您的路径和库路径中，并提供包导出的任何 bash 或 shell 命令。

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

       $ source install/setup.bash

  .. group-tab:: macOS

    .. code-block:: console

       $ . install/setup.bash

  .. group-tab:: Windows

    In a Windows command line interface:

    .. code-block:: console

       $ call install\setup.bat

    Or with Powershell:

    .. code-block:: console

       $ install\setup.ps1

Try a demo
^^^^^^^^^^

With the environment sourced, we can run executables built by colcon.
Let's run a subscriber node from the examples:

.. code-block:: console

    $ ros2 run examples_rclcpp_minimal_subscriber subscriber_member_function

In another terminal, let's run a publisher node (don't forget to source the setup script):

.. code-block:: console

    $ ros2 run examples_rclcpp_minimal_publisher publisher_member_function

You should see messages from the publisher and subscriber with numbers incrementing.

Create your own package
-----------------------

colcon uses the ``package.xml`` specification defined in `REP 149 <https://reps.openrobotics.org/rep-0149/>`__ (`format 2 <https://reps.openrobotics.org/rep-0140/>`__ is also supported).

colcon supports multiple build types.
The recommended build types are ``ament_cmake`` and ``ament_python``.
Also supported are pure ``cmake`` packages.

An example of an ``ament_python`` build is the `ament_index_python package <https://github.com/ament/ament_index/tree/{REPOS_FILE_BRANCH}/ament_index_python>`__ , where the setup.py is the primary entry point for building.

A package such as `demo_nodes_cpp <https://github.com/ros2/demos/tree/{REPOS_FILE_BRANCH}/demo_nodes_cpp>`__ uses the ``ament_cmake`` build type, and uses CMake as the build tool.

For convenience, you can use the tool ``ros2 pkg create`` to create a new package based on a template.
A full description of creating a package and how to use ``ros2 pkg create`` is in the upcoming tutorial :doc:`create a package <./Creating-Your-First-ROS2-Package>`.

.. note:: For ``catkin`` users, this is the equivalent of ``catkin_create_package``.

设置 ``colcon_cd``
------------------

``colcon_cd`` 命令允许您快速将 shell 的当前工作目录更改为包的目录。
例如，``colcon_cd some_ros_package`` 会快速将您带到目录 ``~/ros2_ws/src/some_ros_package``。
要设置 ``colcon_cd``，您需要运行以下命令来修改您的 shell 启动脚本：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ echo "source /usr/share/colcon_cd/function/colcon_cd.sh" >> ~/.bashrc
        $ echo "export _colcon_cd_root=/opt/ros/{DISTRO}/" >> ~/.bashrc

   .. group-tab:: macOS

      .. code-block:: console

        $ echo "source /usr/local/share/colcon_cd/function/colcon_cd.sh" >> ~/.bashrc
        $ echo "export _colcon_cd_root=~/ros2_install" >> ~/.bashrc

   .. group-tab:: Windows

      Not yet available

Depending on the way you installed ``colcon_cd`` and where your workspace is, the instructions above may vary, please refer to `the documentation <https://colcon.readthedocs.io/en/released/user/installation.html#quick-directory-changes>`__ for more details.
To undo this in Linux and macOS, locate your system's shell startup script and remove the appended source and export commands.

设置 ``colcon`` tab 补全
------------------------

``colcon`` 命令支持 bash 和类似 bash 的 shell 的命令补全。
必须安装 ``colcon-argcomplete`` 包，并且 `可能需要一些设置 <https://colcon.readthedocs.io/en/released/user/installation.html#enable-completion>`__ 才能使其工作。

技巧
----

* 如果您不想构建特定的包，则在目录中放置一个名为 ``COLCON_IGNORE`` 的空文件，它将不会被索引。

* 如果您想避免在 CMake 包中配置和构建测试，可以传递：``--cmake-args -DBUILD_TESTING=0``。

* 如果您想从包中运行单个特定测试：

  .. code-block:: console

     $ colcon test --packages-select YOUR_PKG_NAME --ctest-args -R YOUR_TEST_IN_PKG

设置 ``colcon`` mixins
----------------------

各种命令行选项写起来很繁琐和/或难以记住。

例如，要将 CMake 构建类型更改为 debug，通常使用：

.. code-block:: console

    $ colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug

为了使常用命令行选项更容易调用，此仓库提供了这些"快捷方式"。

要安装默认的 colcon mixins，运行以下命令：

.. code-block:: console

    $ colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml
    $ colcon mixin update default

然后，尝试使用 ``debug`` mixin：

.. code-block:: console

    $ colcon build --mixin debug

有关更多详细信息，请参阅 `colcon mixin 仓库 <https://github.com/colcon/colcon-mixin-repository>`__。
