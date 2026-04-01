.. redirect-from::

    Tutorials/Writing-A-Simple-Cpp-Publisher-And-Subscriber

.. _CppPubSub:

Writing a simple publisher and subscriber (C++)
===============================================

**目标：** 使用 C++ 创建并运行发布者和订阅者节点。

**教程级别：** 初级

**时间：** 20 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

:doc:`节点 <../Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>` 是通过 ROS 图进行通信的可执行进程。
在本教程中，节点将以字符串消息的形式通过 :doc:`主题 <../Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics>` 相互传递信息。
这里使用的示例是一个简单的"发布者"和"订阅者"系统；一个节点发布数据，另一个节点订阅主题以便接收该数据。

这些示例中使用的代码可以在 `这里 <https://github.com/ros2/examples/tree/{REPOS_FILE_BRANCH}/rclcpp/topics>`__ 找到。

前提条件
--------

在之前的教程中，您学习了如何 :doc:`创建工作空间 <./Creating-A-Workspace/Creating-A-Workspace>` 和 :doc:`创建包 <./Creating-Your-First-ROS2-Package>`。

任务
----

1 创建包
^^^^^^^^

打开一个新终端并 :doc:`source 您的 ROS 2 安装 <../Beginner-CLI-Tools/Configuring-ROS2-Environment>`，以便 ``ros2`` 命令可以工作。

导航到在 :ref:`之前的教程 <new-directory>` 中创建的 ``ros2_ws`` 目录。

回想一下，包应该在 ``src`` 目录中创建，而不是工作空间的根目录。
因此，导航到 ``ros2_ws/src``，并运行包创建命令：

.. code-block:: console

    $ ros2 pkg create --build-type ament_cmake --license Apache-2.0 cpp_pubsub

您的终端将返回一条消息，验证您的包 ``cpp_pubsub`` 及其所有必要文件和文件夹的创建。

导航到 ``ros2_ws/src/cpp_pubsub/src``。
回想一下，这是任何 CMake 包中包含可执行文件的源文件所在的目录。


2 编写发布者节点
^^^^^^^^^^^^^^^^

通过输入以下命令下载示例 talker 代码：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

            $ wget -O publisher_lambda_function.cpp https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_publisher/lambda.cpp

   .. group-tab:: macOS

      .. code-block:: console

            $ wget -O publisher_lambda_function.cpp https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_publisher/lambda.cpp

   .. group-tab:: Windows

      In a Windows command line prompt:

      .. code-block:: console

            $ curl -sk https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_publisher/lambda.cpp -o publisher_lambda_function.cpp

      Or in powershell:

      .. code-block:: console

            $ curl https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_publisher/lambda.cpp -o publisher_lambda_function.cpp

现在将有一个名为 ``publisher_lambda_function.cpp`` 的新文件。
使用您喜欢的文本编辑器打开该文件。

.. code-block:: C++

    #include <chrono>
    #include <memory>
    #include <string>

    #include "rclcpp/rclcpp.hpp"
    #include "std_msgs/msg/string.hpp"

    using namespace std::chrono_literals;

    /* 此示例创建 Node 的子类，并使用高级的 C++11 lambda
    * 函数来缩短回调语法，但代价是使代码
    * 乍一看更难理解。 */

    class MinimalPublisher : public rclcpp::Node
    {
    public:
      MinimalPublisher()
      : Node("minimal_publisher"), count_(0)
      {
        publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
        auto timer_callback =
          [this]() -> void {
            auto message = std_msgs::msg::String();
            message.data = "Hello, world! " + std::to_string(this->count_++);
            RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
            this->publisher_->publish(message);
          };
        timer_ = this->create_wall_timer(500ms, timer_callback);
      }

    private:
      rclcpp::TimerBase::SharedPtr timer_;
      rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
      size_t count_;
    };

    int main(int argc, char * argv[])
    {
      rclcpp::init(argc, argv);
      rclcpp::spin(std::make_shared<MinimalPublisher>());
      rclcpp::shutdown();
      return 0;
    }

2.1 检查代码
~~~~~~~~~~~~

代码顶部包含您将使用的标准 C++ 头文件。
在标准 C++ 头文件之后是 ``rclcpp/rclcpp.hpp`` 包含，它允许您使用 ROS 2 系统的最常见部分。
最后是 ``std_msgs/msg/string.hpp``，它包含您将用于发布数据的内置消息类型。

.. code-block:: C++

    #include <chrono>
    #include <memory>
    #include <string>

    #include "rclcpp/rclcpp.hpp"
    #include "std_msgs/msg/string.hpp"

    using namespace std::chrono_literals;

这些行表示节点的依赖项。
回想一下，依赖项必须添加到 ``package.xml`` 和 ``CMakeLists.txt`` 中，您将在下一节中执行此操作。

下一行通过从 ``rclcpp::Node`` 继承来创建节点类 ``MinimalPublisher``。
代码中的每个 ``this`` 都指向该节点。

.. code-block:: C++

    class MinimalPublisher : public rclcpp::Node

公共构造函数将节点命名为 ``minimal_publisher`` 并将 ``count_`` 初始化为 0。
在构造函数内部，发布者使用 ``String`` 消息类型、主题名称 ``topic`` 和所需的队列大小进行初始化，以在备份事件中限制消息数量。
接下来，声明一个名为 ``timer_callback`` 的 `lambda 函数 <https://en.cppreference.com/w/cpp/language/lambda>`_。
它对当前对象 ``this`` 执行引用捕获，不接受输入参数并返回 void。
``timer_callback`` 函数创建一个类型为 ``String`` 的新消息，使用所需的字符串设置其数据并发布它。
``RCLCPP_INFO`` 宏确保每条发布的消息都打印到控制台。
最后，``timer_`` 被初始化，这导致 ``timer_callback`` 函数每秒执行两次。

.. code-block:: C++

    public:
      MinimalPublisher()
      : Node("minimal_publisher"), count_(0)
      {
        publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
        auto timer_callback =
          [this]() -> void {
            auto message = std_msgs::msg::String();
            message.data = "Hello, world! " + std::to_string(this->count_++);
            RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
            this->publisher_->publish(message);
          };
        timer_ = this->create_wall_timer(500ms, timer_callback);
      }

在类的底部是计时器、发布者和计数器字段的声明。

.. code-block:: C++

    private:
      rclcpp::TimerBase::SharedPtr timer_;
      rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
      size_t count_;

在 ``MinimalPublisher`` 类之后是 ``main``，节点实际执行的地方。
``rclcpp::init`` 初始化 ROS 2，``rclcpp::spin`` 开始处理来自节点的数据，包括来自计时器的回调。

.. code-block:: C++

    int main(int argc, char * argv[])
    {
      rclcpp::init(argc, argv);
      rclcpp::spin(std::make_shared<MinimalPublisher>());
      rclcpp::shutdown();
      return 0;
    }

2.2 添加依赖项
~~~~~~~~~~~~~~

返回上一级到 ``ros2_ws/src/cpp_pubsub`` 目录，那里已经为您创建了 ``CMakeLists.txt`` 和 ``package.xml`` 文件。

使用文本编辑器打开 ``package.xml``。

正如 :doc:`之前的教程 <./Creating-Your-First-ROS2-Package>` 中提到的，确保填写 ``<description>``、``<maintainer>`` 和 ``<license>`` 标签：

.. code-block:: xml

      <description>Examples of minimal publisher/subscriber using rclcpp</description>
      <maintainer email="you@email.com">Your Name</maintainer>
      <license>Apache-2.0</license>

在 ``ament_cmake`` buildtool 依赖项之后添加新行，并粘贴与您的节点 include 语句对应的以下依赖项：

.. code-block:: xml

    <depend>rclcpp</depend>
    <depend>std_msgs</depend>

这声明了包在构建和执行代码时需要 ``rclcpp`` 和 ``std_msgs``。

确保保存文件。

2.3 CMakeLists.txt
~~~~~~~~~~~~~~~~~~

现在打开 ``CMakeLists.txt`` 文件。
在现有依赖项 ``find_package(ament_cmake REQUIRED)`` 下方，添加以下行：

.. code-block:: cmake

    find_package(rclcpp REQUIRED)
    find_package(std_msgs REQUIRED)

之后，添加可执行文件并将其命名为 ``talker``，以便您可以使用 ``ros2 run`` 运行节点：

.. code-block:: cmake

    add_executable(talker src/publisher_lambda_function.cpp)
    target_link_libraries(talker PUBLIC rclcpp::rclcpp ${std_msgs_TARGETS})

最后，添加 ``install(TARGETS...)`` 部分，以便 ``ros2 run`` 可以找到您的可执行文件：

.. code-block:: cmake

  install(TARGETS
    talker
    DESTINATION lib/${PROJECT_NAME})

您可以通过删除一些不必要的部分和注释来清理 ``CMakeLists.txt``，使其如下所示：

.. code-block:: cmake

  cmake_minimum_required(VERSION 3.5)
  project(cpp_pubsub)

  # Default to C++14
  if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
  endif()

  if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
  endif()

  find_package(ament_cmake REQUIRED)
  find_package(rclcpp REQUIRED)
  find_package(std_msgs REQUIRED)

  add_executable(talker src/publisher_lambda_function.cpp)
  target_link_libraries(talker PUBLIC rclcpp::rclcpp ${std_msgs_TARGETS})

  install(TARGETS
    talker
    DESTINATION lib/${PROJECT_NAME})

  ament_package()

您现在可以构建包、source 本地设置文件并运行它，但让我们先创建订阅者节点，以便您可以看到完整系统的运行。

3 编写订阅者节点
^^^^^^^^^^^^^^^^

返回 ``ros2_ws/src/cpp_pubsub/src`` 创建下一个节点。
在终端中输入以下代码：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

            $ wget -O subscriber_lambda_function.cpp https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_subscriber/lambda.cpp

   .. group-tab:: macOS

      .. code-block:: console

            $ wget -O subscriber_lambda_function.cpp https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_subscriber/lambda.cpp

   .. group-tab:: Windows

      In a Windows command line prompt:

      .. code-block:: console

            $ curl -sk https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_subscriber/lambda.cpp -o subscriber_lambda_function.cpp

      Or in powershell:

      .. code-block:: console

            $ curl https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclcpp/topics/minimal_subscriber/lambda.cpp -o subscriber_lambda_function.cpp

检查以确保这些文件存在：

.. code-block:: console

    publisher_lambda_function.cpp  subscriber_lambda_function.cpp

使用文本编辑器打开 ``subscriber_lambda_function.cpp``。

.. code-block:: C++

    #include <memory>

    #include "rclcpp/rclcpp.hpp"
    #include "std_msgs/msg/string.hpp"

    class MinimalSubscriber : public rclcpp::Node
    {
    public:
      MinimalSubscriber()
      : Node("minimal_subscriber")
      {
        auto topic_callback =
          [this](std_msgs::msg::String::UniquePtr msg) -> void {
            RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
          };
        subscription_ =
          this->create_subscription<std_msgs::msg::String>("topic", 10, topic_callback);
      }

    private:
      rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
    };

    int main(int argc, char * argv[])
    {
      rclcpp::init(argc, argv);
      rclcpp::spin(std::make_shared<MinimalSubscriber>());
      rclcpp::shutdown();
      return 0;
    }

3.1 检查代码
~~~~~~~~~~~~

订阅者节点的代码与发布者的代码几乎相同。
现在节点被命名为 ``minimal_subscriber``，构造函数使用节点的 ``create_subscription`` 函数来执行回调。

没有计时器，因为订阅者只是在数据发布到 ``topic`` 主题时做出响应。

``topic_callback`` 函数接收通过主题发布的字符串消息数据，并使用 ``RCLCPP_INFO`` 宏简单地将其写入控制台。

回想 :doc:`主题教程 <../Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics>`，发布者和订阅者使用的主题名称和消息类型必须匹配才能允许它们通信。

.. code-block:: C++

    public:
      MinimalSubscriber()
      : Node("minimal_subscriber")
      {
        auto topic_callback =
          [this](std_msgs::msg::String::UniquePtr msg) -> void {
            RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
          };
        subscription_ =
          this->create_subscription<std_msgs::msg::String>("topic", 10, topic_callback);
      }

此类中唯一的字段声明是订阅。

.. code-block:: C++

    private:
      rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;

The ``main`` function is exactly the same, except now it spins the ``MinimalSubscriber`` node.
For the publisher node, spinning meant starting the timer, but for the subscriber it simply means preparing to receive messages whenever they come.

Since this node has the same dependencies as the publisher node, there's nothing new to add to ``package.xml``.

3.2 CMakeLists.txt
~~~~~~~~~~~~~~~~~~

Reopen ``CMakeLists.txt`` and add the executable and target for the subscriber node below the publisher's entries.

.. code-block:: cmake

  add_executable(listener src/subscriber_lambda_function.cpp)
  target_link_libraries(listener PUBLIC rclcpp::rclcpp ${std_msgs_TARGETS})

  install(TARGETS
    talker
    listener
    DESTINATION lib/${PROJECT_NAME})

Make sure to save the file, and then your pub/sub system should be ready.

.. _cpppubsub-build-and-run:

4 Build and run
^^^^^^^^^^^^^^^
You likely already have the ``rclcpp`` and ``std_msgs`` packages installed as part of your ROS 2 system.
It's good practice to run ``rosdep`` in the root of your workspace (``ros2_ws``) to check for missing dependencies before building:

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

            $ rosdep install -i --from-path src --rosdistro {DISTRO} -y

   .. group-tab:: macOS

      rosdep only runs on Linux, so you can skip ahead to next step.

   .. group-tab:: Windows

      rosdep only runs on Linux, so you can skip ahead to next step.


Still in the root of your workspace, ``ros2_ws``, build your new package:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ colcon build --packages-select cpp_pubsub

  .. group-tab:: macOS

    .. code-block:: console

      $ colcon build --packages-select cpp_pubsub

  .. group-tab:: Windows

    .. code-block:: console

      $ colcon build --merge-install --packages-select cpp_pubsub

Open a new terminal, navigate to ``ros2_ws``, and source the setup files:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ . install/setup.bash

  .. group-tab:: macOS

    .. code-block:: console

      $ . install/setup.bash

  .. group-tab:: Windows

    .. code-block:: console

      $ call install/setup.bat

Now run the talker node.
The terminal should start publishing info messages every 0.5 seconds, like so:

.. code-block:: console

     $ ros2 run cpp_pubsub talker
     [INFO] [minimal_publisher]: Publishing: "Hello World: 0"
     [INFO] [minimal_publisher]: Publishing: "Hello World: 1"
     [INFO] [minimal_publisher]: Publishing: "Hello World: 2"
     [INFO] [minimal_publisher]: Publishing: "Hello World: 3"
     [INFO] [minimal_publisher]: Publishing: "Hello World: 4"

Open another terminal, source the setup files from inside ``ros2_ws`` again, and then start the listener node.
The listener will start printing messages to the console, starting at whatever message count the publisher is on at that time:

.. code-block:: console

     $ ros2 run cpp_pubsub listener
     [INFO] [minimal_subscriber]: I heard: "Hello World: 10"
     [INFO] [minimal_subscriber]: I heard: "Hello World: 11"
     [INFO] [minimal_subscriber]: I heard: "Hello World: 12"
     [INFO] [minimal_subscriber]: I heard: "Hello World: 13"
     [INFO] [minimal_subscriber]: I heard: "Hello World: 14"

Enter ``Ctrl+C`` in each terminal to stop the nodes from spinning.

Summary
-------

You created two nodes to publish and subscribe to data over a topic.
Before compiling and running them, you added their dependencies and executables to the package configuration files.

Next steps
----------

Next you'll create another simple ROS 2 package using the service/client model.
Again, you can choose to write it in either :doc:`C++ <./Writing-A-Simple-Cpp-Service-And-Client>` or :doc:`Python <./Writing-A-Simple-Py-Service-And-Client>`.

Related content
---------------

There are several ways you could write a publisher and subscriber in C++; check out the ``minimal_publisher`` and ``minimal_subscriber`` packages in the `ros2/examples <https://github.com/ros2/examples/tree/{REPOS_FILE_BRANCH}/rclcpp/topics>`_ repo.
