.. redirect-from::

    Tutorials/Creating-Your-First-ROS2-Package

.. _CreatePkg:

Creating a package
==================

**目标：** 使用 CMake 或 Python 创建新包，并运行其可执行文件。

**教程级别：** 初级

**时间：** 15 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

1 什么是 ROS 2 包？
^^^^^^^^^^^^^^^^^^^

包是 ROS 2 代码的组织单元。
如果您希望能够安装您的代码或与他人共享，那么您需要将其组织在包中。
通过包，您可以发布您的 ROS 2 作品，并允许其他人轻松构建和使用它。

ROS 2 中的包创建使用 ament 作为其构建系统，使用 colcon 作为其构建工具。
您可以使用 CMake 或 Python 创建包，这是官方支持的，尽管存在其他构建类型。

2 ROS 2 包由什么组成？
^^^^^^^^^^^^^^^^^^^^^^

ROS 2 Python 和 CMake 包各自有自己的最低必需内容：

.. tabs::

   .. group-tab:: CMake

      * ``CMakeLists.txt`` file that describes how to build the code within the package
      * ``include/<package_name>`` directory containing the public headers for the package
      * ``package.xml`` file containing meta information about the package
      * ``src`` directory containing the source code for the package

   .. group-tab:: Python

      * ``package.xml`` file containing meta information about the package
      * ``resource/<package_name>`` marker file for the package
      * ``setup.cfg`` is required when a package has executables, so ``ros2 run`` can find them
      * ``setup.py`` containing instructions for how to install the package
      * ``<package_name>`` - a directory with the same name as your package, used by ROS 2 tools to find your package, contains ``__init__.py``

The simplest possible package may have a file structure that looks like:

.. tabs::

   .. group-tab:: CMake

      .. code-block:: console

        my_package/
             CMakeLists.txt
             include/my_package/
             package.xml
             src/

   .. group-tab:: Python

      .. code-block:: console

        my_package/
              package.xml
              resource/my_package
              setup.cfg
              setup.py
              my_package/


3 工作空间中的包
^^^^^^^^^^^^^^^^

单个工作空间可以包含任意数量的包，每个包都在自己的文件夹中。
您还可以在一个工作空间中拥有不同构建类型的包（CMake、Python 等）。
您不能有嵌套的包。

最佳实践是在工作空间中有一个 ``src`` 文件夹，并在其中创建您的包。
这使工作空间的顶层保持"干净"。

一个简单的工作空间可能如下所示：

.. code-block:: console

  workspace_folder/
      src/
        cpp_package_1/
            CMakeLists.txt
            include/cpp_package_1/
            package.xml
            src/

        py_package_1/
            package.xml
            resource/py_package_1
            setup.cfg
            setup.py
            py_package_1/
        ...
        cpp_package_n/
            CMakeLists.txt
            include/cpp_package_n/
            package.xml
            src/


前提条件
--------

按照 :doc:`上一个教程 <./Creating-A-Workspace/Creating-A-Workspace>` 中的说明，您应该已经有一个 ROS 2 工作空间。
您将在此工作空间中创建您的包。


任务
----

1 创建包
^^^^^^^^

首先，:doc:`source 您的 ROS 2 安装 <../Beginner-CLI-Tools/Configuring-ROS2-Environment>`。

让我们使用您在 :ref:`上一个教程 <new-directory>` 中创建的工作空间 ``ros2_ws`` 来创建新包。

在运行包创建命令之前，请确保您在 ``src`` 文件夹中。

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ cd ~/ros2_ws/src

   .. group-tab:: macOS

     .. code-block:: console

       $ cd ~/ros2_ws/src

   .. group-tab:: Windows

     .. code-block:: console

       $ cd \ros2_ws\src

在 ROS 2 中创建新包的命令语法为：

.. tabs::

   .. group-tab:: CMake

      .. code-block:: console

        $ ros2 pkg create --build-type ament_cmake --license Apache-2.0 <package_name>

   .. group-tab:: Python

      .. code-block:: console

        $ ros2 pkg create --build-type ament_python --license Apache-2.0 <package_name>

在本教程中，您将使用可选参数 ``--node-name`` 和 ``--license``。
``--node-name`` 选项在包中创建一个简单的 Hello World 类型可执行文件，``--license`` 声明包的许可证信息。

在终端中输入以下命令：

.. tabs::

   .. group-tab:: CMake

      .. code-block:: console

        $ ros2 pkg create --build-type ament_cmake --license Apache-2.0 --node-name my_node my_package

   .. group-tab:: Python

      .. code-block:: console

        $ ros2 pkg create --build-type ament_python --license Apache-2.0 --node-name my_node my_package

现在您将在工作空间的 ``src`` 目录中拥有一个名为 ``my_package`` 的新文件夹。

运行命令后，您的终端将返回消息：

.. tabs::

   .. group-tab:: CMake

      .. code-block:: console

        going to create a new package
        package name: my_package
        destination directory: /home/user/ros2_ws/src
        package format: 3
        version: 0.0.0
        description: TODO: Package description
        maintainer: ['<name> <email>']
        licenses: ['Apache-2.0']
        build type: ament_cmake
        dependencies: []
        node_name: my_node
        creating folder ./my_package
        creating ./my_package/package.xml
        creating source and include folder
        creating folder ./my_package/src
        creating folder ./my_package/include/my_package
        creating ./my_package/CMakeLists.txt
        creating ./my_package/src/my_node.cpp

   .. group-tab:: Python

      .. code-block:: console

        going to create a new package
        package name: my_package
        destination directory: /home/user/ros2_ws/src
        package format: 3
        version: 0.0.0
        description: TODO: Package description
        maintainer: ['<name> <email>']
        licenses: ['Apache-2.0']
        build type: ament_python
        dependencies: []
        node_name: my_node
        creating folder ./my_package
        creating ./my_package/package.xml
        creating source folder
        creating folder ./my_package/my_package
        creating ./my_package/setup.py
        creating ./my_package/setup.cfg
        creating folder ./my_package/resource
        creating ./my_package/resource/my_package
        creating ./my_package/my_package/__init__.py
        creating folder ./my_package/test
        creating ./my_package/test/test_copyright.py
        creating ./my_package/test/test_flake8.py
        creating ./my_package/test/test_pep257.py
        creating ./my_package/my_package/my_node.py

您可以看到新包自动生成的文件。

2 构建包
^^^^^^^^

将包放在工作空间中特别有价值，因为您可以通过在工作空间根目录中运行 ``colcon build`` 一次构建多个包。
否则，您必须单独构建每个包。

返回工作空间的根目录：

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

现在您可以构建您的包：

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ colcon build

  .. group-tab:: macOS

    .. code-block:: console

      $ colcon build

  .. group-tab:: Windows

    .. code-block:: console

      $ colcon build --merge-install

    Windows 不允许长路径，因此 ``merge-install`` 会将所有路径合并到 ``install`` 目录中。

回顾上一个教程，您的 ``ros2_ws`` 中还有 ``ros_tutorials`` 包。
您可能已经注意到，运行 ``colcon build`` 也构建了 ``turtlesim`` 包。
当您的工作空间中只有几个包时，这没问题，但当有很多包时，``colcon build`` 可能需要很长时间。

下次只构建 ``my_package`` 包，您可以运行：

.. code-block:: console

    $ colcon build --packages-select my_package

3 Source setup 文件
^^^^^^^^^^^^^^^^^^^

要使用新包和可执行文件，首先打开一个新终端并 source 您的主 ROS 2 安装。

然后，在 ``ros2_ws`` 目录中，运行以下命令来 source 您的工作空间：

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ source install/local_setup.bash

  .. group-tab:: macOS

    .. code-block:: console

      $ . install/local_setup.bash

  .. group-tab:: Windows

    .. code-block:: console

      $ call install/local_setup.bat

现在您的工作空间已添加到您的路径中，您将能够使用新包的可执行文件。

4 使用包
^^^^^^^^

要运行您在包创建期间使用 ``--node-name`` 参数创建的可执行文件，请输入命令：

.. code-block:: console

  $ ros2 run my_package my_node

这将向您的终端返回一条消息：

.. tabs::

   .. group-tab:: CMake

      .. code-block:: console

        hello world my_package package

   .. group-tab:: Python

      .. code-block:: console

        Hi from my_package.

5 检查包内容
^^^^^^^^^^^^

在 ``ros2_ws/src/my_package`` 中，您将看到 ``ros2 pkg create`` 自动生成的文件和文件夹：

.. tabs::

   .. group-tab:: CMake

      .. code-block:: console

        CMakeLists.txt  include  package.xml  src

      ``my_node.cpp`` 位于 ``src`` 目录中。
      这是您将来所有自定义 C++ 节点的存放位置。

   .. group-tab:: Python

      .. code-block:: console

        my_package  package.xml  resource  setup.cfg  setup.py  test

      ``my_node.py`` 位于 ``my_package`` 目录中。
      这是您将来所有自定义 Python 节点的存放位置。

6 自定义 package.xml
^^^^^^^^^^^^^^^^^^^

您可能已经注意到，创建包后的返回消息中，``description`` 和 ``license`` 字段包含 ``TODO`` 注释。
这是因为包描述和许可证声明不会自动设置，但如果您想发布包，这些是必需的。
``maintainer`` 字段可能也需要填写。

从 ``ros2_ws/src/my_package``，使用您喜欢的文本编辑器打开 ``package.xml``：

.. tabs::

   .. group-tab:: CMake

    .. code-block:: xml

     <?xml version="1.0"?>
     <?xml-model
        href="http://download.ros.org/schema/package_format3.xsd"
        schematypens="http://www.w3.org/2001/XMLSchema"?>
     <package format="3">
      <name>my_package</name>
      <version>0.0.0</version>
      <description>TODO: Package description</description>
      <maintainer email="user@todo.todo">user</maintainer>
      <license>TODO: License declaration</license>

      <buildtool_depend>ament_cmake</buildtool_depend>

      <test_depend>ament_lint_auto</test_depend>
      <test_depend>ament_lint_common</test_depend>

      <export>
        <build_type>ament_cmake</build_type>
      </export>
     </package>

   .. group-tab:: Python

    .. code-block:: xml

     <?xml version="1.0"?>
     <?xml-model
        href="http://download.ros.org/schema/package_format3.xsd"
        schematypens="http://www.w3.org/2001/XMLSchema"?>
     <package format="3">
      <name>my_package</name>
      <version>0.0.0</version>
      <description>TODO: Package description</description>
      <maintainer email="user@todo.todo">user</maintainer>
      <license>TODO: License declaration</license>

      <test_depend>ament_copyright</test_depend>
      <test_depend>ament_flake8</test_depend>
      <test_depend>ament_pep257</test_depend>
      <test_depend>python3-pytest</test_depend>

      <export>
        <build_type>ament_python</build_type>
      </export>
     </package>

如果 ``maintainer`` 行没有自动为您填充，请输入您的姓名和电子邮件。
然后，编辑 ``description`` 行以概括包：

.. code-block:: xml

  <description>Beginner client libraries tutorials practice package</description>

然后，更新 ``license`` 行。
您可以 `在此处 <https://opensource.org/licenses/alphabetical>`__ 了解更多关于开源许可证的信息。
由于此包仅用于练习，使用任何许可证都是安全的。
我们将使用 ``Apache-2.0``：

.. code-block:: xml

  <license>Apache-2.0</license>

编辑完成后不要忘记保存。

在 license 标签下方，您将看到一些以 ``_depend`` 结尾的标签名称。
这是您的 ``package.xml`` 列出其对其他包的依赖项的地方，以便 colcon 搜索。
``my_package`` 很简单，没有任何依赖项，但您将在接下来的教程中看到此空间被使用。

.. tabs::

   .. group-tab:: CMake

      您现在完成了！

   .. group-tab:: Python

      ``setup.py`` 文件包含与 ``package.xml`` 相同的 description、maintainer 和 license 字段，因此您也需要设置它们。
      它们在两个文件中需要完全匹配。
      version 和 name（``package_name``）也需要完全匹配，并且应该在两个文件中自动填充。

      使用您喜欢的文本编辑器打开 ``setup.py``。

      .. code-block:: python

       from setuptools import find_packages, setup

       package_name = 'my_py_pkg'

       setup(
        name=package_name,
        version='0.0.0',
        packages=find_packages(exclude=['test']),
        data_files=[
            ('share/ament_index/resource_index/packages',
                    ['resource/' + package_name]),
            ('share/' + package_name, ['package.xml']),
          ],
        install_requires=['setuptools'],
        zip_safe=True,
        maintainer='TODO',
        maintainer_email='TODO',
        description='TODO: Package description',
        license='TODO: License declaration',
        tests_require=['pytest'],
        entry_points={
            'console_scripts': [
                    'my_node = my_py_pkg.my_node:main'
            ],
          },
       )

      编辑 ``maintainer``、``maintainer_email`` 和 ``description`` 行以匹配 ``package.xml``。

      不要忘记保存文件。


总结
----

您已经创建了一个包来组织您的代码，并使其易于他人使用。

您的包已自动填充必要的文件，然后您使用 colcon 构建它，以便您可以在本地环境中使用其可执行文件。

下一步
------

接下来，让我们向包中添加一些有意义的内容。
您将从简单的发布者/订阅者系统开始，您可以选择使用 :doc:`C++ <./Writing-A-Simple-Cpp-Publisher-And-Subscriber>` 或 :doc:`Python <./Writing-A-Simple-Py-Publisher-And-Subscriber>` 编写。
