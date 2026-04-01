.. redirect-from::

    Tutorials/Writing-A-Simple-Py-Publisher-And-Subscriber

.. _PyPubSub:

Writing a simple publisher and subscriber (Python)
==================================================

**目标：** 使用 Python 创建并运行发布者和订阅者节点。

**教程级别：** 初级

**时间：** 20 分钟

.. contents:: 目录
   :depth: 2
   :local:

背景
----

在本教程中，您将创建 :doc:`节点 <../Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>`，它们通过 :doc:`主题 <../Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics>` 以字符串消息的形式相互传递信息。
这里使用的示例是一个简单的"发布者"和"订阅者"系统；
一个节点发布数据，另一个节点订阅主题以便接收该数据。

这些示例中使用的代码可以在 `这里 <https://github.com/ros2/examples/tree/{REPOS_FILE_BRANCH}/rclpy/topics>`__ 找到。

前提条件
--------

在之前的教程中，您学习了如何 :doc:`创建工作空间 <./Creating-A-Workspace/Creating-A-Workspace>` 和 :doc:`创建包 <./Creating-Your-First-ROS2-Package>`。

建议对 Python 有基本了解，但并非完全必要。

任务
----

1 创建包
^^^^^^^^

打开一个新终端并 :doc:`source 您的 ROS 2 安装 <../Beginner-CLI-Tools/Configuring-ROS2-Environment>`，以便 ``ros2`` 命令可以工作。

导航到在 :ref:`之前的教程 <new-directory>` 中创建的 ``ros2_ws`` 目录。

回想一下，包应该在 ``src`` 目录中创建，而不是工作空间的根目录。
因此，导航到 ``ros2_ws/src``，并运行包创建命令：

.. code-block:: console

  $ ros2 pkg create --build-type ament_python --license Apache-2.0 py_pubsub

您的终端将返回一条消息，验证您的包 ``py_pubsub`` 及其所有必要文件和文件夹的创建。

2 编写发布者节点
^^^^^^^^^^^^^^^^

导航到 ``ros2_ws/src/py_pubsub/py_pubsub``。
回想一下，此目录是一个 `Python 包 <https://docs.python.org/3/tutorial/modules.html#packages>`__，与它所在的 ROS 2 包同名。

通过输入以下命令下载示例 talker 代码：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ wget https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_publisher/examples_rclpy_minimal_publisher/publisher_member_function.py

   .. group-tab:: macOS

      .. code-block:: console

        $ wget https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_publisher/examples_rclpy_minimal_publisher/publisher_member_function.py

   .. group-tab:: Windows

      In a Windows command line prompt:

      .. code-block:: console

            $ curl -sk https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_publisher/examples_rclpy_minimal_publisher/publisher_member_function.py -o publisher_member_function.py

      Or in powershell:

      .. code-block:: console

            $ curl https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_publisher/examples_rclpy_minimal_publisher/publisher_member_function.py -o publisher_member_function.py

现在将在 ``__init__.py`` 旁边有一个名为 ``publisher_member_function.py`` 的新文件。

使用您喜欢的文本编辑器打开该文件。

.. code-block:: python

  import rclpy
  from rclpy.executors import ExternalShutdownException
  from rclpy.node import Node

  from std_msgs.msg import String


  class MinimalPublisher(Node):

      def __init__(self):
          super().__init__('minimal_publisher')
          self.publisher_ = self.create_publisher(String, 'topic', 10)
          timer_period = 0.5  # seconds
          self.timer = self.create_timer(timer_period, self.timer_callback)
          self.i = 0

      def timer_callback(self):
          msg = String()
          msg.data = 'Hello World: %d' % self.i
          self.publisher_.publish(msg)
          self.get_logger().info('Publishing: "%s"' % msg.data)
          self.i += 1


  def main(args=None):
      try:
          with rclpy.init(args=args):
              minimal_publisher = MinimalPublisher()

              rclpy.spin(minimal_publisher)
      except (KeyboardInterrupt, ExternalShutdownException):
          pass


  if __name__ == '__main__':
      main()


2.1 检查代码
~~~~~~~~~~~~

注释后的第一行代码导入 ``rclpy``，以便可以使用其 ``Node`` 类。

.. code-block:: python

  import rclpy
  from rclpy.executors import ExternalShutdownException
  from rclpy.node import Node

下一个语句导入节点用于结构化其在主题上传递的数据的内置字符串消息类型。

.. code-block:: python

  from std_msgs.msg import String

这些行表示节点的依赖项。
回想一下，依赖项必须添加到 ``package.xml`` 中，您将在下一节中执行此操作。

接下来，创建了 ``MinimalPublisher`` 类，它继承自（或是其子类）``Node``。

.. code-block:: python

  class MinimalPublisher(Node):

接下来是类的构造函数的定义。
``super().__init__`` 调用 ``Node`` 类的构造函数并为其提供节点名称，在本例中为 ``minimal_publisher``。

``create_publisher`` 声明节点在名为 ``topic`` 的主题上发布类型为 ``String``（从 ``std_msgs.msg`` 模块导入）的消息，并且"队列大小"为 10。
队列大小是必需的 QoS（服务质量）设置，如果订阅者接收消息的速度不够快，则限制排队消息的数量。

接下来，创建一个计时器，回调每 0.5 秒执行一次。
``self.i`` 是回调中使用的计数器。

.. code-block:: python

  def __init__(self):
      super().__init__('minimal_publisher')
      self.publisher_ = self.create_publisher(String, 'topic', 10)
      timer_period = 0.5  # seconds
      self.timer = self.create_timer(timer_period, self.timer_callback)
      self.i = 0

``timer_callback`` 创建一个附加了计数器值的消息，并使用 ``get_logger().info`` 将其发布到控制台。

.. code-block:: python

  def timer_callback(self):
      msg = String()
      msg.data = 'Hello World: %d' % self.i
      self.publisher_.publish(msg)
      self.get_logger().info('Publishing: "%s"' % msg.data)
      self.i += 1

Lastly, the main function is defined.

.. code-block:: python

  def main(args=None):
      try:
          with rclpy.init(args=args):
              minimal_publisher = MinimalPublisher()

              rclpy.spin(minimal_publisher)
      except (KeyboardInterrupt, ExternalShutdownException):
          pass


首先初始化 ``rclpy`` 库，然后创建节点，然后"spin"节点以便调用其回调。

2.2 添加依赖项
~~~~~~~~~~~~~~

返回上一级到 ``ros2_ws/src/py_pubsub`` 目录，那里已经为您创建了 ``setup.py``、``setup.cfg`` 和 ``package.xml`` 文件。

使用文本编辑器打开 ``package.xml``。

正如 :doc:`之前的教程 <./Creating-Your-First-ROS2-Package>` 中提到的，确保填写 ``<description>``、``<maintainer>`` 和 ``<license>`` 标签：

.. code-block:: xml

  <description>Examples of minimal publisher/subscriber using rclpy</description>
  <maintainer email="you@email.com">Your Name</maintainer>
  <license>Apache-2.0</license>

在上述行之后，添加与您的节点导入语句对应的以下依赖项：

.. code-block:: xml

  <exec_depend>rclpy</exec_depend>
  <exec_depend>std_msgs</exec_depend>

这声明了包在执行代码时需要 ``rclpy`` 和 ``std_msgs``。

确保保存文件。

2.3 添加入口点
~~~~~~~~~~~~~~

打开 ``setup.py`` 文件。
同样，将 ``maintainer``、``maintainer_email``、``description`` 和 ``license`` 字段与您的 ``package.xml`` 匹配：

.. code-block:: python

  maintainer='YourName',
  maintainer_email='you@email.com',
  description='Examples of minimal publisher/subscriber using rclpy',
  license='Apache-2.0',

在 ``entry_points`` 字段的 ``console_scripts`` 括号内添加以下行：

.. code-block:: python

  entry_points={
          'console_scripts': [
                  'talker = py_pubsub.publisher_member_function:main',
          ],
  },

不要忘记保存。

2.4 检查 setup.cfg
~~~~~~~~~~~~~~~~~~

``setup.cfg`` 文件的内容应该自动正确填充，如下所示：

.. code-block:: ini

  [develop]
  script_dir=$base/lib/py_pubsub
  [install]
  install_scripts=$base/lib/py_pubsub

这只是告诉 setuptools 将您的可执行文件放在 ``lib`` 中，因为 ``ros2 run`` 会在那里查找它们。

您现在可以构建包、source 本地设置文件并运行它，但让我们先创建订阅者节点，以便您可以看到完整系统的运行。

3 编写订阅者节点
^^^^^^^^^^^^^^^^

返回 ``ros2_ws/src/py_pubsub/py_pubsub`` 创建下一个节点。
在终端中输入以下代码：

.. tabs::

   .. group-tab:: Linux

      .. code-block:: console

        $ wget https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_subscriber/examples_rclpy_minimal_subscriber/subscriber_member_function.py

   .. group-tab:: macOS

      .. code-block:: console

        $ wget https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_subscriber/examples_rclpy_minimal_subscriber/subscriber_member_function.py

   .. group-tab:: Windows

      In a Windows command line prompt:

      .. code-block:: console

            $ curl -sk https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_subscriber/examples_rclpy_minimal_subscriber/subscriber_member_function.py -o subscriber_member_function.py

      Or in powershell:

      .. code-block:: console

            $ curl https://raw.githubusercontent.com/ros2/examples/{REPOS_FILE_BRANCH}/rclpy/topics/minimal_subscriber/examples_rclpy_minimal_subscriber/subscriber_member_function.py -o subscriber_member_function.py

现在目录应该包含这些文件：

.. code-block:: console

  __init__.py  publisher_member_function.py  subscriber_member_function.py

3.1 检查代码
~~~~~~~~~~~~

使用文本编辑器打开 ``subscriber_member_function.py``。

.. code-block:: python

  import rclpy
  from rclpy.executors import ExternalShutdownException
  from rclpy.node import Node

  from std_msgs.msg import String


  class MinimalSubscriber(Node):

      def __init__(self):
          super().__init__('minimal_subscriber')
          self.subscription = self.create_subscription(
              String,
              'topic',
              self.listener_callback,
              10)
          self.subscription  # prevent unused variable warning

      def listener_callback(self, msg):
          self.get_logger().info('I heard: "%s"' % msg.data)


  def main(args=None):
      try:
          with rclpy.init(args=args):
              minimal_subscriber = MinimalSubscriber()

              rclpy.spin(minimal_subscriber)
      except (KeyboardInterrupt, ExternalShutdownException):
          pass


  if __name__ == '__main__':
      main()

订阅者节点的代码与发布者的代码几乎相同。
构造函数使用与发布者相同的参数创建订阅者。
回想 :doc:`主题教程 <../Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics>`，发布者和订阅者使用的主题名称和消息类型必须匹配才能允许它们通信。

.. code-block:: python

  self.subscription = self.create_subscription(
      String,
      'topic',
      self.listener_callback,
      10)

订阅者的构造函数和回调不包含任何计时器定义，因为它不需要。
它的回调在收到消息后立即被调用。

回调定义只是将信息消息打印到控制台，以及它收到的数据。
回想一下，发布者定义了 ``msg.data = 'Hello World: %d' % self.i``

.. code-block:: python

  def listener_callback(self, msg):
      self.get_logger().info('I heard: "%s"' % msg.data)

``main`` 定义几乎完全相同，用订阅者替换发布者的创建和旋转。

.. code-block:: python

  minimal_subscriber = MinimalSubscriber()

  rclpy.spin(minimal_subscriber)

由于此节点与发布者具有相同的依赖项，因此无需向 ``package.xml`` 添加新内容。
``setup.cfg`` 文件也可以保持不变。


3.2 添加入口点
~~~~~~~~~~~~~~

重新打开 ``setup.py`` 并在发布者的入口点下方添加订阅者节点的入口点。
``entry_points`` 字段现在应该如下所示：

.. code-block:: python

  entry_points={
          'console_scripts': [
                  'talker = py_pubsub.publisher_member_function:main',
                  'listener = py_pubsub.subscriber_member_function:main',
          ],
  },

Make sure to save the file, and then your pub/sub system should be ready.

4 Build and run
^^^^^^^^^^^^^^^
You likely already have the ``rclpy`` and ``std_msgs`` packages installed as part of your ROS 2 system.
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

      $ colcon build --packages-select py_pubsub

  .. group-tab:: macOS

    .. code-block:: console

      $ colcon build --packages-select py_pubsub

  .. group-tab:: Windows

    .. code-block:: console

      $ colcon build --merge-install --packages-select py_pubsub

Open a new terminal, navigate to ``ros2_ws``, and source the setup files:

.. tabs::

  .. group-tab:: Linux

    .. code-block:: console

      $ source install/setup.bash

  .. group-tab:: macOS

    .. code-block:: console

      $ . install/setup.bash

  .. group-tab:: Windows

    .. code-block:: console

      $ call install/setup.bat

Now run the talker node.
The terminal should start publishing info messages every 0.5 seconds, like so:

.. code-block:: console

  $ ros2 run py_pubsub talker
  [info] [minimal_publisher]: publishing: "hello world: 0"
  [info] [minimal_publisher]: publishing: "hello world: 1"
  [info] [minimal_publisher]: publishing: "hello world: 2"
  [info] [minimal_publisher]: publishing: "hello world: 3"
  [info] [minimal_publisher]: publishing: "hello world: 4"
  ...

Open another terminal, source the setup files from inside ``ros2_ws`` again, and then start the listener node.
The listener will start printing messages to the console, starting at whatever message count the publisher is on at that time, like so:

.. code-block:: console

  $ ros2 run py_pubsub listener
  [INFO] [minimal_subscriber]: I heard: "Hello World: 10"
  [INFO] [minimal_subscriber]: I heard: "Hello World: 11"
  [INFO] [minimal_subscriber]: I heard: "Hello World: 12"
  [INFO] [minimal_subscriber]: I heard: "Hello World: 13"
  [INFO] [minimal_subscriber]: I heard: "Hello World: 14"

Enter ``Ctrl+C`` in each terminal to stop the nodes from spinning.

Summary
-------

You created two nodes to publish and subscribe to data over a topic.
Before running them, you added their dependencies and entry points to the package configuration files.

Next steps
----------

Next you'll create another simple ROS 2 package using the service/client model.
Again, you can choose to write it in either :doc:`C++ <./Writing-A-Simple-Cpp-Service-And-Client>` or :doc:`Python <./Writing-A-Simple-Py-Service-And-Client>`.

Related content
---------------

There are several ways you could write a publisher and subscriber in Python; check out the ``minimal_publisher`` and ``minimal_subscriber`` packages in the `ros2/examples <https://github.com/ros2/examples/tree/{REPOS_FILE_BRANCH}/rclpy/topics>`_ repo.
