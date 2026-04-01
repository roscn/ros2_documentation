.. redirect-from::

    Tutorials/Workspace/Creating-A-Workspace

.. _ROS2Workspace:

Creating a workspace
====================

**目标：** 创建一个工作空间并学习如何为开发和测试设置覆盖层。

**教程级别：** 初级

**时间：** 20 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

工作空间是一个包含 ROS 2 包的目录。
在使用 ROS 2 之前，有必要在您计划工作的终端中 source 您的 ROS 2 安装工作空间。
这使 ROS 2 的包可供您在该终端中使用。

您还可以选择 source 一个"覆盖层"——一个次要工作空间，您可以在其中添加新包，而不会干扰您正在扩展的现有 ROS 2 工作空间或"底层"。
您的底层必须包含覆盖层中所有包的依赖项。
覆盖层中的包将覆盖底层中的包。
也可以有多层底层和覆盖层，每个后续覆盖层使用其父底层的包。


前提条件
---------

* :doc:`ROS 2 安装 <../../../Installation>`
* :doc:`colcon 安装 <../Colcon-Tutorial>`
* `git 安装 <https://git-scm.com/book/en/v2/Getting-Started-Installing-Git>`__
* :doc:`turtlesim 安装 <../../Beginner-CLI-Tools/Introducing-Turtlesim/Introducing-Turtlesim>`
* 已安装 :doc:`rosdep <../../Intermediate/Rosdep>`
* 了解基本终端命令（`这里是 Linux 指南 <https://www2.cs.sfu.ca/~ggbaker/reference/unix/>`__）
* 您选择的文本编辑器

任务
----

1 Source ROS 2 环境
^^^^^^^^^^^^^^^^^^^

您的主 ROS 2 安装将作为本教程的底层。
（请记住，底层不一定是主 ROS 2 安装。）

根据您安装 ROS 2 的方式（从源代码或二进制文件）以及您使用的平台，您的确切 source 命令会有所不同：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ source /opt/ros/{DISTRO}/setup.bash

   .. group-tab:: macOS

      .. code-block:: console

        $ . ~/ros2_install/ros2-osx/setup.bash

   .. group-tab:: Windows

      请记住使用 ``x64 Native Tools Command Prompt for VS 2019`` 来执行以下命令，因为我们要构建一个工作空间。

      .. code-block:: console

        $ call C:\dev\ros2\local_setup.bat

如果这些命令对您不起作用，请查阅您遵循的 :doc:`安装指南 <../../../Installation>`。

.. _new-directory:

2 创建新目录
^^^^^^^^^^^^

最佳实践是为每个新工作空间创建一个新目录。
名称并不重要，但让它指示工作空间的用途会很有帮助。
让我们选择目录名称 ``ros2_ws``，表示"开发工作空间"：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ mkdir -p ~/ros2_ws/src
        $ cd ~/ros2_ws/src

   .. group-tab:: macOS

      .. code-block:: console

        $ mkdir -p ~/ros2_ws/src
        $ cd ~/ros2_ws/src

   .. group-tab:: Windows

     .. code-block:: console

       $ md \ros2_ws\src
       $ cd \ros2_ws\src


另一个最佳实践是将工作空间中的任何包放入 ``src`` 目录中。
上面的代码在 ``ros2_ws`` 内部创建了一个 ``src`` 目录，然后导航到其中。


3 克隆示例仓库
^^^^^^^^^^^^^^

在克隆之前，请确保您仍在 ``ros2_ws/src`` 目录中。

在其余的初级开发者教程中，您将创建自己的包，但现在您将练习使用现有包组合工作空间。

如果您完成了 :doc:`初级：CLI 工具 <../../Beginner-CLI-Tools>` 教程，您将熟悉 ``turtlesim``，它是 `ros_tutorials <https://github.com/ros/ros_tutorials/>`__ 中的包之一。

一个仓库可以有多个分支。
您需要检出针对您安装的 ROS 2 发行版的分支。
克隆此仓库时，添加 ``-b`` 参数，后跟该分支。

在 ``ros2_ws/src`` 目录中，运行以下命令：

.. code-block:: console

  $ git clone https://github.com/ros/ros_tutorials.git -b {DISTRO}

现在 ``ros_tutorials`` 已克隆到您的工作空间中。
``ros_tutorials`` 仓库包含 ``turtlesim`` 包，我们将在本教程的其余部分使用它。
此仓库中的其他包未构建，因为它们包含 ``COLCON_IGNORE`` 文件。

到目前为止，您已经用示例包填充了工作空间，但它还不是一个功能完整的工作空间。
您需要先解决依赖关系，然后构建工作空间。


4 解决依赖关系
^^^^^^^^^^^^^^

在构建工作空间之前，您需要解决包依赖关系。
您可能已经拥有所有依赖项，但最佳实践是每次克隆时都检查依赖关系。
您不希望在长时间等待后构建失败，才发现缺少依赖项。

从工作空间的根目录（``ros2_ws``），运行以下命令：

.. tabs::

   .. group-tab:: Linux

      如果您仍在包含 ``ros_tutorials`` 克隆的 ``src`` 目录中，请确保运行 ``cd ..`` 返回到工作空间（``ros2_ws``）。

      .. code-block:: console

        $ cd ..
        $ rosdep install -i --from-path src --rosdistro {DISTRO} -y

   .. group-tab:: macOS

      rosdep 仅在 Linux 上运行，因此您可以跳到"5 使用 colcon 构建工作空间"部分。

   .. group-tab:: Windows

      rosdep 仅在 Linux 上运行，因此您可以跳到"5 使用 colcon 构建工作空间"部分。

如果您在 Linux 上从源代码或二进制存档安装 ROS 2，则需要使用其安装说明中的 rosdep 命令。
这里是 :ref:`从源代码安装的 rosdep 部分 <linux-development-setup-install-dependencies-using-rosdep>` 和 :ref:`二进制存档的 rosdep 部分 <linux-install-binary-install-missing-dependencies>`。

如果您已经拥有所有依赖项，控制台将返回：

.. code-block:: text

  #All required rosdeps installed successfully

包在 package.xml 文件中声明其依赖项（您将在下一个教程中了解更多关于包的信息）。
此命令遍历这些声明并安装缺少的依赖项。
您可以在另一个教程中了解更多关于 ``rosdep`` 的信息（即将推出）。

5 使用 colcon 构建工作空间
^^^^^^^^^^^^^^^^^^^^^^^^^^

从工作空间的根目录（``ros2_ws``），您现在可以使用以下命令构建包：

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ colcon build
      Starting >>> turtlesim
      Finished <<< turtlesim [5.49s]

      Summary: 1 package finished [5.58s]


  .. group-tab:: macOS

    .. code-block:: console

      $ colcon build
      Starting >>> turtlesim
      Finished <<< turtlesim [5.49s]

      Summary: 1 package finished [5.58s]


  .. group-tab:: Windows

    .. code-block:: console

      $ colcon build --merge-install
      Starting >>> turtlesim
      Finished <<< turtlesim [5.49s]

      Summary: 1 package finished [5.58s]

    Windows 不允许长路径，因此 ``merge-install`` 会将所有路径合并到 ``install`` 目录中。

.. note::

  ``colcon build`` 的其他有用参数：

  * ``--packages-up-to`` 构建您想要的包及其所有依赖项，但不构建整个工作空间（节省时间）
  * ``--symlink-install`` 使您不必在每次调整 Python 脚本时重新构建
  * ``--event-handlers console_direct+`` 在构建时显示控制台输出（否则可以在 ``log`` 目录中找到）
  * ``--executor sequential`` 逐个处理包而不是使用并行处理

构建完成后，在工作空间根目录（``~/ros2_ws``）中输入命令。
您将看到 colcon 创建了新目录：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ ls
        build  install  log  src

   .. group-tab:: macOS

      .. code-block:: console

        $ ls
        build  install  log  src

   .. group-tab:: Windows

      .. code-block:: console

        $ dir
        build  install  log  src

``install`` 目录是工作空间 setup 文件所在的位置，您可以使用它来 source 您的覆盖层。


6 Source 覆盖层
^^^^^^^^^^^^^^^

在 source 覆盖层之前，非常重要的一点是打开一个新终端，与您构建工作空间的终端分开。
在构建工作空间的同一终端中 source 覆盖层，或同样在 source 覆盖层的地方构建，可能会产生复杂的问题。

在新终端中，将您的主 ROS 2 环境 source 为"底层"，这样您就可以在其上构建覆盖层：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ source /opt/ros/{DISTRO}/setup.bash

   .. group-tab:: macOS

      .. code-block:: console

        $ . ~/ros2_install/ros2-osx/setup.bash

   .. group-tab:: Windows

      在这种情况下，您可以使用普通的命令提示符，因为我们不会在此终端中构建任何工作空间。

      .. code-block:: console

        $ call C:\dev\ros2\local_setup.bat

进入工作空间的根目录：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ cd ~/ros2_ws

   .. group-tab:: macOS

      .. code-block:: console

        $ cd ~/ros2_ws

   .. group-tab:: Windows

     .. code-block:: console

       $ cd \ros2_ws

在根目录中，source 您的覆盖层：

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ source install/local_setup.bash

  .. group-tab:: macOS

    .. code-block:: console

      $ . install/local_setup.bash

  .. group-tab:: Windows

    .. code-block:: console

      $ call install\setup.bat

.. note::

  Source 覆盖层的 ``local_setup`` 只会将覆盖层中可用的包添加到您的环境中。
  ``setup`` 会 source 覆盖层以及创建它的底层，允许您利用两个工作空间。

  因此，先 source 主 ROS 2 安装的 ``setup``，然后 source ``ros2_ws`` 覆盖层的 ``local_setup``，就像您刚才做的那样，
  与只 source ``ros2_ws`` 的 ``setup`` 是一样的，因为它包含了其底层的环境。

现在您可以从覆盖层运行 ``turtlesim`` 包：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

但是，您如何判断这是覆盖层的 turtlesim 在运行，而不是主安装的 turtlesim？

让我们修改覆盖层中的 turtlesim，这样您就可以看到效果：

* 您可以单独于底层在覆盖层中修改和重新构建包。
* 覆盖层优先于底层。


7 修改覆盖层
^^^^^^^^^^^^

您可以通过编辑 turtlesim 窗口上的标题栏来修改覆盖层中的 ``turtlesim``。
为此，在 ``~/ros2_ws/src/ros_tutorials/turtlesim/src`` 中找到 ``turtle_frame.cpp`` 文件。
使用您喜欢的文本编辑器打开 ``turtle_frame.cpp``。

找到函数 ``setWindowTitle("TurtleSim");``，将值 ``"TurtleSim"`` 更改为 ``"MyTurtleSim"``,并保存文件。

返回到您之前运行 ``colcon build`` 的第一个终端，然后再次运行它。

返回到第二个终端（已 source 覆盖层）并再次运行 turtlesim：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

您将看到 turtlesim 窗口上的标题栏现在显示"MyTurtleSim"。

.. image:: images/overlay.png

尽管您之前在此终端中 source 了主 ROS 2 环境，但 ``ros2_ws`` 环境的覆盖层优先于底层的内容。

要查看您的底层是否仍然完好，请打开一个全新的终端并仅 source 您的 ROS 2 安装。
再次运行 turtlesim：

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

.. image:: images/underlay.png

您可以看到覆盖层中的修改实际上并没有影响底层中的任何内容。


总结
----

在本教程中，您将主 ROS 2 发行版安装 source 为底层，并通过在新工作空间中克隆和构建包来创建覆盖层。
覆盖层被前置到路径中，并优先于底层，正如您在修改 turtlesim 时所看到的那样。

建议使用覆盖层来处理少量包，这样您就不必将所有内容放在同一个工作空间中，也不必在每次迭代时重新构建庞大的工作空间。

下一步
------

现在您已经了解了创建、构建和 source 自己工作空间的详细信息，您可以学习如何 :doc:`创建自己的包 <../Creating-Your-First-ROS2-Package>`。

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ source /opt/ros/{DISTRO}/setup.bash

   .. group-tab:: macOS

      .. code-block:: console

        $ . ~/ros2_install/ros2-osx/setup.bash

   .. group-tab:: Windows

      Remember to use a ``x64 Native Tools Command Prompt for VS 2019`` for executing the following commands, as we are going to build a workspace.

      .. code-block:: console

        $ call C:\dev\ros2\local_setup.bat

Consult the :doc:`installation guide <../../../Installation>` you followed if these commands don't work for you.

.. _new-directory:

2 Create a new directory
^^^^^^^^^^^^^^^^^^^^^^^^

Best practice is to create a new directory for every new workspace.
The name doesn't matter, but it is helpful to have it indicate the purpose of the workspace.
Let's choose the directory name ``ros2_ws``, for "development workspace":

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ mkdir -p ~/ros2_ws/src
        $ cd ~/ros2_ws/src

   .. group-tab:: macOS

      .. code-block:: console

        $ mkdir -p ~/ros2_ws/src
        $ cd ~/ros2_ws/src

   .. group-tab:: Windows

     .. code-block:: console

       $ md \ros2_ws\src
       $ cd \ros2_ws\src


Another best practice is to put any packages in your workspace into the ``src`` directory.
The above code creates a ``src`` directory inside ``ros2_ws`` and then navigates into it.


3 Clone a sample repo
^^^^^^^^^^^^^^^^^^^^^

Ensure you're still in the ``ros2_ws/src`` directory before you clone.

In the rest of the beginner developer tutorials, you will create your own packages, but for now you will practice putting a workspace together using existing packages.

If you went through the :doc:`Beginner: CLI Tools <../../Beginner-CLI-Tools>` tutorials, you'll be familiar with ``turtlesim``, one of the packages in `ros_tutorials <https://github.com/ros/ros_tutorials/>`__.

A repo can have multiple branches.
You need to check out the one that targets your installed ROS 2 distro.
When you clone this repo, add the ``-b`` argument followed by that branch.

In the ``ros2_ws/src`` directory, run the following command:

.. code-block:: console

  $ git clone https://github.com/ros/ros_tutorials.git -b {DISTRO}

Now ``ros_tutorials`` is cloned in your workspace.
The ``ros_tutorials`` repository contains the ``turtlesim`` package, which we'll use in the rest of this tutorial.
The other packages in this repository are not built because they contain a ``COLCON_IGNORE`` file.

So far you have populated your workspace with a sample package, but it isn't a fully-functional workspace yet.
You need to resolve the dependencies first and then build the workspace.


4 Resolve dependencies
^^^^^^^^^^^^^^^^^^^^^^

Before building the workspace, you need to resolve the package dependencies.
You may have all the dependencies already, but best practice is to check for dependencies every time you clone.
You wouldn't want a build to fail after a long wait only to realize that you have missing dependencies.

From the root of your workspace (``ros2_ws``), run the following command:

.. tabs::

   .. group-tab:: Linux

      If you're still in the ``src`` directory with the ``ros_tutorials`` clone, make sure to run ``cd ..`` to move back up to the workspace (``ros2_ws``).

      .. code-block:: console

        $ cd ..
        $ rosdep install -i --from-path src --rosdistro {DISTRO} -y

   .. group-tab:: macOS

      rosdep only runs on Linux, so you can skip ahead to section "5 Build the workspace with colcon".

   .. group-tab:: Windows

      rosdep only runs on Linux, so you can skip ahead to section "5 Build the workspace with colcon".

If you installed ROS 2 on Linux from source or the binary archive, you will need to use the rosdep command from their installation instructions.
Here are the :ref:`from-source rosdep section <linux-development-setup-install-dependencies-using-rosdep>` and the :ref:`binary archive rosdep section <linux-install-binary-install-missing-dependencies>`.

If you already have all your dependencies, the console will return:

.. code-block:: text

  #All required rosdeps installed successfully

Packages declare their dependencies in the package.xml file (you will learn more about packages in the next tutorial).
This command walks through those declarations and installs the ones that are missing.
You can learn more about ``rosdep`` in another tutorial (coming soon).

5 Build the workspace with colcon
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

From the root of your workspace (``ros2_ws``), you can now build your packages using the command:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ colcon build
      Starting >>> turtlesim
      Finished <<< turtlesim [5.49s]

      Summary: 1 package finished [5.58s]


  .. group-tab:: macOS

    .. code-block:: console

      $ colcon build
      Starting >>> turtlesim
      Finished <<< turtlesim [5.49s]

      Summary: 1 package finished [5.58s]


  .. group-tab:: Windows

    .. code-block:: console

      $ colcon build --merge-install
      Starting >>> turtlesim
      Finished <<< turtlesim [5.49s]

      Summary: 1 package finished [5.58s]

    Windows doesn't allow long paths, so ``merge-install`` will combine all the paths into the ``install`` directory.

.. note::

  Other useful arguments for ``colcon build``:

  * ``--packages-up-to`` builds the package you want, plus all its dependencies, but not the whole workspace (saves time)
  * ``--symlink-install`` saves you from having to rebuild every time you tweak python scripts
  * ``--event-handlers console_direct+`` shows console output while building (can otherwise be found in the ``log`` directory)
  * ``--executor sequential`` processes the packages one by one instead of using parallelism

Once the build is finished, enter the command in the workspace root (``~/ros2_ws``).
You will see that colcon has created new directories:

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ ls
        build  install  log  src

   .. group-tab:: macOS

      .. code-block:: console

        $ ls
        build  install  log  src

   .. group-tab:: Windows

      .. code-block:: console

        $ dir
        build  install  log  src

The ``install`` directory is where your workspace's setup files are, which you can use to source your overlay.


6 Source the overlay
^^^^^^^^^^^^^^^^^^^^

Before sourcing the overlay, it is very important that you open a new terminal, separate from the one where you built the workspace.
Sourcing an overlay in the same terminal where you built, or likewise building where an overlay is sourced, may create complex issues.

In the new terminal, source your main ROS 2 environment as the "underlay", so you can build the overlay "on top of" it:

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ source /opt/ros/{DISTRO}/setup.bash

   .. group-tab:: macOS

      .. code-block:: console

        $ . ~/ros2_install/ros2-osx/setup.bash

   .. group-tab:: Windows

      In this case you can use a normal command prompt, as we are not going to build any workspace in this terminal.

      .. code-block:: console

        $ call C:\dev\ros2\local_setup.bat

Go into the root of your workspace:

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ cd ~/ros2_ws

   .. group-tab:: macOS

      .. code-block:: console

        $ cd ~/ros2_ws

   .. group-tab:: Windows

     .. code-block:: console

       $ cd \ros2_ws

In the root, source your overlay:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ source install/local_setup.bash

  .. group-tab:: macOS

    .. code-block:: console

      $ . install/local_setup.bash

  .. group-tab:: Windows

    .. code-block:: console

      $ call install\setup.bat

.. note::

  Sourcing the ``local_setup`` of the overlay will only add the packages available in the overlay to your environment.
  ``setup`` sources the overlay as well as the underlay it was created in, allowing you to utilize both workspaces.

  So, sourcing your main ROS 2 installation's ``setup`` and then the ``ros2_ws`` overlay's ``local_setup``, like you just did,
  is the same as just sourcing ``ros2_ws``'s ``setup``, because that includes the environment of its underlay.

Now you can run the ``turtlesim`` package from the overlay:

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

But how can you tell that this is the overlay turtlesim running, and not your main installation's turtlesim?

Let's modify turtlesim in the overlay so you can see the effects:

* You can modify and rebuild packages in the overlay separately from the underlay.
* The overlay takes precedence over the underlay.


7 Modify the overlay
^^^^^^^^^^^^^^^^^^^^

You can modify ``turtlesim`` in your overlay by editing the title bar on the turtlesim window.
To do this, locate the ``turtle_frame.cpp`` file in ``~/ros2_ws/src/ros_tutorials/turtlesim/src``.
Open ``turtle_frame.cpp`` with your preferred text editor.

Find the function ``setWindowTitle("TurtleSim");``, change the value ``"TurtleSim"`` to ``"MyTurtleSim"``, and save the file.

Return to the first terminal where you ran ``colcon build`` earlier and run it again.

Return to the second terminal (where the overlay is sourced) and run turtlesim again:

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

You will see the title bar on the turtlesim window now says "MyTurtleSim".

.. image:: images/overlay.png

Even though your main ROS 2 environment was sourced in this terminal earlier, the overlay of your ``ros2_ws`` environment takes precedence over the contents of the underlay.

To see that your underlay is still intact, open a brand new terminal and source only your ROS 2 installation.
Run turtlesim again:

.. code-block:: console

  $ ros2 run turtlesim turtlesim_node

.. image:: images/underlay.png

You can see that modifications in the overlay did not actually affect anything in the underlay.


Summary
-------
In this tutorial, you sourced your main ROS 2 distro install as your underlay, and created an overlay by cloning and building packages in a new workspace.
The overlay gets prepended to the path, and takes precedence over the underlay, as you saw with your modified turtlesim.

Using overlays is recommended for working on a small number of packages, so you don't have to put everything in the same workspace and rebuild a huge workspace on every iteration.

Next steps
----------

Now that you understand the details behind creating, building and sourcing your own workspace, you can learn how to :doc:`create your own packages <../Creating-Your-First-ROS2-Package>`.
