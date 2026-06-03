ROS工作空间命名原则，习惯用后缀 _ws 

ROS2是话题的通讯，通过话题发送和订阅，两个节点通过同一个话题进行信息收发，确保话题名字一样

wpr机器人仿真资源克隆(作为一个单独的功能包，放置在src里面) ： git clone https://gitee.com/s-robot/wpr_simulation2.git



### 1.ROS2工作空间分布

```
# 标准 ROS2 工作空间结构

ROS2_WS/                              # 工作空间根目录
│
│
│  # 在工作空间下需要手动创建src，将功能包放置于src目录下，返回工作空间进行构件
│
├── src/                              ← 📦 所有ROS包放在这里（手动创建），包很多，但是里面的节点可以跨包通信
│   ├── topic_pkg/                    ← 包1：话题通信包
│   ├── service_pkg/                  ← 包2：服务通信包
│   ├── action_pkg/                   ← 包3：动作通信包
│   └── ...                           ← 其他包
│
├── build/                            ← 编译临时文件（自动生成，可删除）
│   ├── topic_pkg/
│   ├── service_pkg/
│   └── ...
│
├── install/                          ← 安装文件（自动生成，可删除）
│   ├── topic_pkg/
│   ├── service_pkg/
│   ├── _local_setup_util_ps1.py
│   ├── _local_setup_util_sh.py
│   ├── local_setup.bash
│   ├── local_setup.ps1
│   ├── local_setup.sh
│   ├── local_setup.zsh
│   └── setup.bash
│
└── log/                              ← 日志文件（自动生成，可删除）
    └── ...


ROS包内目录结构

topic_pkg/                            # 包根目录
│
├── CMakeLists.txt                    ← 构建配置文件（必需）
├── package.xml                       ← 包描述文件（必需）
├── setup.py                          ← Python安装配置（仅纯Python包需要），混合开发，使用CMakList.txt就足够了
│
├── include/                          ← C++头文件目录
│   └── topic_pkg/                    ← 与包名同名的子目录
│       ├── publisher.hpp
│       ├── subscriber.hpp
│       └── msg_utils.hpp
│
├── src/                              ← C++源文件目录
│   ├── publisher_node.cpp
│   ├── subscriber_node.cpp
│   └── msg_utils.cpp
│
├── scripts/                          ← Python可执行脚本，逻辑需手写，注意添加可执行权限（chmod +x xxx.py）
│   ├── publisher_node.py
│   ├── subscriber_node.py
│   └── data_processor.py
│
│  # 脚本文件，可以直接通过 ros2 run 来进行单独运行，也可以通过启动文件来进行运行
|  # 注意，ros2 run只能运行单个节点
│
├── topic_pkg/                        ← Python模块库（与包名同名）
│   ├── __init__.py                   ← 标记为Python包，名字强制不能更改，一般空白
│   ├── publisher_module.py           ← 自己写的可以重复导入的模块
│   ├── subscriber_module.py		  ← 自己写的可以重复导入的模块
│   └── utils/                        ← 子模块，可以是第三方库
│       ├── __init__.py
│       └── helper.py
│
├── launch/                           ← 手动创建.py启动文件，逻辑需手写，注意只有python文件需要
│   ├── topic_launch.py               
│   └── full_system_launch.py
│ 
│  # 启动文件可以分为多个启动文件，比如说调试的时候不需要启动订阅者，
│  # 测试的时候则需要启动订阅者，完全上线的时候则需要启动所有节点
│  # 通过启动文件一次性调用多个节点，并且配置好，单次调用脚本大多用于测试，指令： ros2 launch
│  # 启动文件虽然采用python语言，但是他可以用来启动所有类型的执行脚本包括c/c++
│ 
| 
├── msg/                              ← 自定义消息类型
│   ├── CustomMsg.msg				  # 比如说机器人的位置（x/y/z 浮点）+ 姿态（roll/pitch/yaw 浮点）+ 状态码（整数） 
│   └── SensorData.msg
│  # 适用于异步流式传输，类比于串口,不等待回传，一直发
|  
├── srv/                              ← 自定义服务类型
│   ├── ComputeSum.srv
│   └── ProcessData.srv
│  # 适用于同步传输，客户端 / 服务器模式，单次查询、执行一次计算、触发一个瞬时操作
|  # 客户端必须等待服务端响应后才能继续执行，是阻塞式通信
|
├── action/                           ← 自定义动作类型
│   └── Navigate.action
│  # 长时间运行、可抢占、可反馈的异步任务（目标 - 反馈 - 结果模式），
|  # 客户端发送目标坐标，服务端实时反馈当前位置、剩余距离等进度，任务完成后返回最终状态，也可中途取消。
|
|
├── config/                           ← 手动创建.yaml配置文件，参数需手写
│   ├── params.yaml
│   └── node_config.yaml
│
├── rviz/                             ← RViz配置
│   └── view_config.rviz
│
├── test/                             ← 手动创建测试文件，测试逻辑需手写
│   ├── test_publisher.cpp
│   ├── test_subscriber.cpp
│   └── test_python_node.py
│
└── resource/                         ← 资源文件，纯 Python 包由ros2 pkg create自动生成，无需手动修改
    └── ament_index/
        └── resource_index/
            └── ...
                    
            
运行前必须加载ROS包：
cd ~/ros2_ws(切换工作空间)
source install/setup.bash(加载ROS包环境变量)
            
常规检查指令：
echo $ROS_DISTRO          # 检查ROS版本
echo $AMENT_PREFIX_PATH	  # 这条命令用于打印 AMENT_PREFIX_PATH 环境变量的值，这个变量告诉系统去哪里寻找已安装的包。
  (举例-返回：/home/wenjun/ros2_ws/install/topic_pkg:/opt/ros/humble)-使用冒号 : 分隔路径
  
创建 C++ 包（生成 CMakeLists.txt + package.xml + include/）
ros2 pkg create topic_pkg --build-type ament_cmake --dependencies rclcpp std_msgs
创建 Python 包（生成 CMakeLists.txt + package.xml + setup.py + resource/）
ros2 pkg create topic_pkg --build-type ament_python --dependencies rclpy std_msgs  
  
自动生成文件：
 CMakeLists.txt
 package.xml
 setup.py
 resource/
 include/topic_pkg/
  
手动生成文件：
 src/
 scripts/
 topic_pkg/  
 launch/
 msg/
 srv/
 action/
 config/
 rviz/
 test/
 
 
```

### 2.如何运行和部署节点代码

#### 2.1.**常用命令行：**

```
# 1. 创建工作空间
mkdir -p ~/ROS2_test_WS/src
cd ~/ROS2_test_WS

# 2. 创建包（一次性带齐参数）包括c/c++和python所需要的依赖包和库
cd src
ros2 pkg create test_pkg \
  --build-type ament_cmake \         
  --dependencies rclcpp std_msgs \  
  --license Apache-2.0

# 3. 查看包结构
ls test_pkg/
# 应该看到：CMakeLists.txt  package.xml  src/  include/

# 4. 返回工作空间根目录编译
cd ~/ROS2_test_WS
colcon build  --packages-select (包名)   # 每次修改完脚本需要编译一下，指定功能包，如果不指定就直接：colcon build

# 5. 设置环境
source install/setup.bash			    # 每次修改完脚本需要设置一下

# 6. 开始运行节点
#------第一种方式：依次手动运行发布者节点和订阅者节点---------
ros2 run test_pkg publisher_node							 # ros2 run 包名称 节点名称

#------第二种方式：直接启动你编写好的launch文件---------
ros2 launch test_pkg pub_sub.launch.py 					     # ros2 run 包名称 启动文件名称  



```

#### 2.2.**发布者节点基本代码(`.py`)：**

```python
#!/usr/bin/env python3            # 注意这里必须要加上，否则脚本无法运行
# -*- coding: utf-8 -*-           # 指定脚本编码为 UTF-8（支持中文）

import rclpy                      # ROS 2 Python 编程的核心库，负责节点初始化、自旋、通信等核心功能
from rclpy.node import Node
from std_msgs.msg import String   # 所有 ROS 2 节点的基类，封装了创建发布者 / 订阅者 / 定时器等核心方法
from std_msgs.msg import Int32    # ROS 2 内置的标准消息类型（String/Int32 等），无需自己定义
import time

class PublisherNode(Node):           	     #继承父类，(父类)，Node封装了ROS2的节点功能
    """发布者节点：定期发布消息到话题"""
    
    def __init__(self):
        super().__init__('publisher_node')   # super()为调用父类代码的方法 super().父类方法名('参数')
        
        # 创建发布者，发布到 'test_topic' 话题
        # String类型消息
        self.string_publisher = self.create_publisher(    #实例属性
            String,          # 参数1：消息类型
            'test_topic',    # 参数2：话题名
            10   			 # 参数3：队列大小
        )
        '''
        create_publisher 是父类 Node 提供的“工厂方法” —— 你调用它时，它会根据你传入的参数（消息类型、话题名、队列大小），
        创建并返回一个 “发布者对象”，你把这个 “发布者对象” 赋值给 self.string_publisher 这个实例属性，相当于 贴个标签，
        存到自定义节点的实例里；同时这个发布者对象自带 publish() 方法,最后可以通过self.string_publisher.publish()发射,
        所以贴上string标签后调用发布者对象的publish()方法就可以传入string类型的消息参数了
        '''
 
        # 也可以发布其他类型的消息，比如Int32
        self.int_publisher = self.create_publisher(
            Int32, 
            'counter_topic', 
            10
        )
        
        # 创建定时器，每1秒发布一次消息
        self.timer = self.create_timer(1.0, self.timer_callback)  
        
        self.counter = 0
        self.get_logger().info('发布者节点已启动，开始发布消息...')
    
    def timer_callback(self):                 #回调函数，前面通过定时器会定时调用此函数，频率依据定时器设置
        """定时器回调函数，定期发布消息"""
        # 将类实例化，String本身是调用ROS库里面的类(from std_msgs.msg import String)
        # string_msg = String()等于是将这个类实例化，string_msg为实例化的标签
        # string_msg.data 等同于 String().data
        string_msg = String()    
        string_msg.data = f'Hello ROS2! 消息编号: {self.counter}'
        self.string_publisher.publish(string_msg)
        # 调用日志对象的info级别的输出方法，把括号里的字符串打印出来。
        self.get_logger().info(f'发布字符串: "{string_msg.data}"')
        
        # 发布整数消息
        int_msg = Int32()
        int_msg.data = self.counter
        self.int_publisher.publish(int_msg)
        self.get_logger().info(f'发布整数: {int_msg.data}')   
        
        self.counter += 1

def main(args=None):
    """主函数"""
    rclpy.init(args=args)    # 初始化ROS2的底层库，包括节点通信
    
    # 创建节点，可以说是实例化
    publisher_node = PublisherNode()
    
    try:
        # 运行节点
        # rclpy.spin 的作用是让节点实例进入 “自旋状态”，持续处理该实例的回调函数（比如定时器、订阅者回调）
        # 参数传入实例
        rclpy.spin(publisher_node) 
    except KeyboardInterrupt:
        pass
    finally:
        # 清理
        publisher_node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

#### 2.3.**订阅者节点基本代码(`.py`)：**

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from std_msgs.msg import Int32

class SubscriberNode(Node):
    """订阅者节点：订阅话题并处理接收到的消息"""
    
    def __init__(self):
        super().__init__('subscriber_node')
        
        # 创建订阅者，订阅 'test_topic' 话题
        self.string_subscription = self.create_subscription(
            String,
            'test_topic',
            self.string_callback,
            10  # 队列大小
        )
        
        # 订阅整数话题
        self.int_subscription = self.create_subscription(
            Int32,
            'counter_topic',
            self.int_callback,
            10
        )
        
        # 防止订阅者被垃圾回收（ROS2的quirk）
        self.string_subscription
        self.int_subscription
        
        self.get_logger().info('订阅者节点已启动，等待消息...')
    
    def string_callback(self, msg):
        """字符串消息回调函数"""
        self.get_logger().info(f'收到字符串消息: "{msg.data}"')
        
        # 这里可以添加处理逻辑
        # 例如：解析消息、控制机器人、保存数据等
    
    def int_callback(self, msg):
        """整数消息回调函数"""
        self.get_logger().info(f'收到整数消息: {msg.data}')
        
        # 处理整数消息
        if msg.data % 5 == 0:
            self.get_logger().warn(f'计数达到 {msg.data} 的倍数！')

def main(args=None):
    """主函数"""
    rclpy.init(args=args)
    
    # 创建节点
    subscriber_node = SubscriberNode()
    
    try:
        # 运行节点
        rclpy.spin(subscriber_node)
    except KeyboardInterrupt:
        pass
    finally:
        # 清理
        subscriber_node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

#### 2.4.**`package.xml`  核心配置文件结构分布** 

- 功能包的基础信息（名称、版本、作者、许可证等）；

  编译 / 运行依赖（比如需要用到的 ROS2 库、第三方库）；

  构建工具（比如用`ament_cmake`还是`setuptools`构建）；

  其他元信息（描述、导出配置等）

**配置文件：**

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>test_pkg</name>
  <version>0.0.0</version>
  <!-- 注释：描述功能包作用 -->
  <description>Topic communication test package (Python)</description>   
  <maintainer email="wenjun@todo.todo">wenjun</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>           <!-- 注释：编译c/c++ -->
  <buildtool_depend>ament_cmake_python</buildtool_depend>    <!-- 注释：编译python -->

  <depend>rclcpp</depend>       <!-- 注释：对C/C++的支持 -->
   <depend>rclpy</depend>       <!-- 注释：对python的支持 -->
  <depend>std_msgs</depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>

```

#### 2.5.**`CMakeList.txt`  核心配置文件结构分布** 

- `package.xml`：声明依赖、功能包元信息（“告诉系统需要什么”）；

  `CMakeLists.txt`：定义编译 / 安装规则（“告诉系统怎么构建、把文件装到哪里”）。

**配置文件：**

```cmake
cmake_minimum_required(VERSION 3.5)   # 1. 指定CMake最低版本要求
project(test_pkg)                     # 2. 定义项目名称（必须和package.xml中的name一致）



# 设置C++标准（如果有C++代码）
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)  # 强制要求符合C++14标准，否则报错


# 查找依赖
find_package(ament_cmake REQUIRED)        # ROS2核心构建工具
find_package(ament_cmake_python REQUIRED) # ROS2 Python构建扩展
find_package(rclpy REQUIRED)              # ROS2 Python客户端库

# C++依赖（混合开发）
find_package(rclcpp REQUIRED)       # ROS2 C++客户端库
find_package(std_msgs REQUIRED)     # 标准消息库（C++/Python都用）

# ==================== Python部分 ====================
# 安装Python脚本
install(PROGRAMS                    # PROGRAMS表示安装 “可执行程序 / 脚本”
  scripts/publisher_node.py         # 脚本路径
  scripts/subscriber_node.py        # 脚本路径
  DESTINATION lib/${PROJECT_NAME}   # 安装目标路径
)

# 由于没有写C++脚本，所以没有添加C++执行部分

# ==================== Launch文件安装 ====================
# 安装launch目录下的所有.launch.py文件
install(DIRECTORY launch/
  DESTINATION share/${PROJECT_NAME}/launch
)


# ==================== 测试 ====================
if(BUILD_TESTING)  # 如果开启编译测试（默认开启）
  find_package(ament_lint_auto REQUIRED)  # 查找代码检查依赖
  ament_lint_auto_find_test_dependencies() # 自动查找测试依赖并运行代码检查（比如语法检查、格式检查）
endif()

ament_package()  # 收尾指令
```

#### 2.6.**`launch.py`** 基础启动脚本文件框架

```python
# 1. 导入必需的Launch模块
from launch import LaunchDescription    
from launch_ros.actions import Node

# 2. 定义启动函数（固定命名：generate_launch_description）
def generate_launch_description():
    # 3. 定义要启动的节点
    # 发布者节点
    publisher_node = Node(
        package='test_pkg',        # 功能包名（必须和package.xml一致）
        executable='publisher_node.py',  # 要运行的可执行文件（脚本名）,必须加入.py后缀
        name='publisher_node',     # 节点名（可选，覆盖代码中super().__init__的名字）
        output='screen'            # 日志输出到终端（关键！否则看不到节点打印）
    )
    
    # 订阅者节点
    subscriber_node = Node(
        package='test_pkg',
        executable='subscriber_node.py',  # 必须加入.py后缀
        name='subscriber_node',
        output='screen'
    )
    
    # 4. 组装启动描述，返回所有要启动的节点
    return LaunchDescription([
        publisher_node,
        subscriber_node
    ])
```

- 如果使用 **`launch`** 来进行启动需要配置 **`CMakeList`** 

  - ```cmake
    # ==================== Launch文件安装 ====================
    # 安装launch目录下的所有.launch.py文件
    install(DIRECTORY launch/
      DESTINATION share/${PROJECT_NAME}/launch
    )
    ```
  
    
  
- **进阶脚本框架：**

  - 话题重映射

  - ```py
    publisher_node = Node(
        package='test_pkg',
        executable='publisher_node',
        name='publisher_node',
        output='screen',
        remappings=[     # 话题重映射
            ('test_topic', 'new_test_topic'),  # (原话题名, 新话题名)
            ('counter_topic', 'new_counter_topic')
        ]
    )
    ```

  - 启动文件直接设置节点参数,比如频率，用于设置不同的启动文件内的节点通讯频率，无需重复修改节点文件

  - ```python
    publisher_node = Node(
        package='test_pkg',
        executable='publisher_node',
        name='publisher_node',
        output='screen',
        parameters=[{'publish_frequency': 2.0}]  # 字典形式传参数  2.0HZ
    )
    
    ```

#### 2.7.XML文件格式(适用于快速编写package.xml)

```xml
<标签名>内容</标签名>      <!-- 内容赋值给标签 -->

<标签名 属性名称="内容">   <!-- 内容赋值给标签里面的属性，可以同时赋值多个属性 --> 
</标签名>     			  <!-- 必须加入结束标签符号 -->
```

| 符号 | 名称          | 作用                 |
| ---- | ------------- | -------------------- |
| `<`  | 小于号/尖括号 | 标签开始             |
| `>`  | 大于号/尖括号 | 标签结束             |
| `</` | 斜杠+尖括号   | 结束标签             |
| `/>` | 尖括号+斜杠   | 自闭合标签（无内容） |

- 举例解释：

- ```xml
  <build_depend>message_generation</build_depend>  <!-- ROS1固定包名，用于自定义消息格式-->
  ```

  - `<build_depend>` → **开始标签**（标签名是 `build_depend`）
  - `message_generation` → **标签内容**（文本值）
  - `</build_depend>` → **结束标签**（`/` 表示结束）
  - **标签必须成对出现（自闭合标签除外）**
  - **标签名区分大小写**

- 常见标签：

  | 标签类型         | 核心作用                                                     | 适用场景                                        |
  | ---------------- | ------------------------------------------------------------ | ----------------------------------------------- |
  | `<build_depend>` | 仅编译阶段需要的依赖（比如代码生成器、编译工具）             | `rosidl_default_generators`（msg/srv 编译）     |
  | `<exec_depend>`  | 仅运行阶段需要的依赖（比如运行时加载接口的库）               | `rosidl_default_runtime`（msg/srv 运行）        |
  | `<depend>`       | 同时包含「编译依赖 + 运行依赖」（ROS2 对旧版 ROS1 的兼容标签） | 同时需要编译 + 运行的核心库（如 rclpy、rclcpp） |

- 嵌套结构

  - ```xml
    <link name="base_link">
        <visual>
            <geometry>
                <!-- 只有加了这一句，它才是长方体 -->
                <box size="1.0 1.0 0.2"/>
            </geometry>
        </visual>
    </link>
    ```

    

#### 2.8.CMakeList.txt文件格式

```
# ================= 1. 版本和项目 =================
cmake_minimum_required(VERSION 3.5)   # 命令(参数名 参数值)
project(test_pkg)                     # 命令(项目名)

# ================= 2. 设置变量 =================
set(CMAKE_CXX_STANDARD 14)            # set(变量名 值)
# set(变量名 值) 是赋值命令

# ================= 3. 查找依赖包 =================
find_package(ament_cmake REQUIRED)    # 命令(包名 选项)
# REQUIRED 表示找不到就报错

# ================= 4. 安装文件 =================
install(PROGRAMS                      # 命令(选项 参数...)
  scripts/publisher_node.py           # 多行参数，缩进只是为了可读性
  DESTINATION lib/${PROJECT_NAME}     # ${} 表示变量引用
)

# ================= 5. 条件判断 =================
if(BUILD_TESTING)                     # if(条件)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()                               # endif() 结束条件

# ================= 6. 收尾 =================
ament_package()                       # ROS2 特有的收尾命令
```



#### 2.9.自定义消息格式(msg)配置

- 首先在功能包里面创建**`msg`**文件夹，在文件夹里面创建 **`txt`** 文件，同时后缀改为( **`.msg`** )，本质仍然是txt格式，所以任意编辑器都可以编辑

- **`.msg`** 文件编写举例：

- ```
  # RobotState.msg 举例：复合数据结构
  int32 id          # 机器人ID
  float32 x         # x坐标
  float32 y         # y坐标
  string status     # 状态（正常/异常）
  bool is_moving    # 是否移动
  ```

- **同时需要增加依赖声明标签在 `package.xml` 文件里面，用于编译依赖和运行依赖**

- ```xml
  <build_depend>rosidl_default_generators</build_depend>        <!-- 编译.msg生成Python/C++接口 -->
  <depend>rosidl_default_runtime</depend>                       <!-- 运行时加载自定义msg,运行依赖 -->
  <member_of_group>rosidl_interface_packages</member_of_group>  <!-- 声明为消息接口包，让自己被功能包识别  -->
  ```

- **需要继续配置 `CMakeList.txt` 文件**

- ```cmake
  find_package(std_msgs REQUIRED)                   # 显式查找std_msgs
  
  # ========== 自定义.msg配置（ROS2核心） ==========
  # 1. 声明自定义msg文件
  set(msg_files
    "msg/Robot.msg"  # 路径要写完整（msg文件夹下的Robot.msg）
  )
  
  # 2. 生成自定义消息（ROS2的核心指令）
  rosidl_generate_interfaces(${PROJECT_NAME}
    ${msg_files}
    DEPENDENCIES std_msgs  # 依赖的基础消息包
    ADD_LINTER_TESTS       # 可选：开启msg语法检查
  )
  
  # 3. 声明消息运行时依赖（让其他节点能找到自定义消息）
  ament_export_dependencies(rosidl_default_runtime)
  ```

- 验证配置是否成功：

  - ```bash
    # 进入工作空间
    cd ROS2_test_WS   
    # 编译这个工作空间下的test_pkg这个功能包
    colcon build --packages-select test_pkg   
    # 加载编译后的环境变量（每次新开终端都要执行）
    source install/setup.bash
    # 运行创建的.msg文件，查看是否输出你需要的数据类型名称
    ros2 interface show test_pkg/msg/Robot
    ```

    ![image-20260319185308136](C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260319185308136.png)

##### 

- 在python安装脚本的地方，新增加两个文件路径

```cmake
# ==================== Python部分 ====================
# 安装Python脚本
install(PROGRAMS                    # PROGRAMS表示安装 “可执行程序 / 脚本”
  scripts/publisher_node.py         # 脚本路径
  scripts/subscriber_node.py        # 脚本路径
  scripts/publisher_robot_node.py           # 针对msg信息测试的发布者脚本路径 (新增)
  scripts/subscriber_robot_node.py          # 针对msg信息测试的订阅者脚本路径(新增)
  DESTINATION lib/${PROJECT_NAME}   # 安装目标路径
)
```

- **发布者代码**：

- ```python
  #!/usr/bin/env python3
  import rclpy
  from rclpy.node import Node
  from test_pkg.msg import Robot
  
  class RobotPublisher(Node):
      def __init__(self):
          super().__init__('publisher_robot_node')
          self.Robot_publisher = self.create_publisher(
              Robot,               # 消息类型，自定义的Robot.msg类型
              '/Robot_status',     # 话题名
              10                   # 队列大小
          )
          self.Robot_timer = self.create_timer(
              1.0,
              self.publisher_robot_msg,    # 定时触发的回调函数
          )
          self.robot_id = 1   # 在这里就初始化了机器人ID
      # 自定义回调函数，开始给消息赋值
      def publisher_robot_msg(self):
          # 开始准备构造自己定义的消息对象，这里直接硬编码直接赋值
          # 正常是接收外部数据，比如直接从全局/配置文件读取，或者作为类的实例变量，先存在实例，等数据到了就赋值给实例变量
          Robot_msg = Robot()   # 构造一下消息对象
          Robot_msg.id = self.robot_id
          Robot_msg.x = 0.5
          Robot_msg.y = 0.8
          Robot_msg.status = "runing" 
          Robot_msg.is_moving = True
          # 发送消息
          self.Robot_publisher.publish(Robot_msg)
          # 打印日志（方便调试）
          self.get_logger().info(f"发布机器人状态：ID={Robot_msg.id}, X={Robot_msg.x:.2f}, 状态={Robot_msg.status}")
          self.robot_id += 1
  
  def main(args=None):
      # 初始化 ROS2 上下文
      rclpy.init(args=args)
      # 创建发布者节点
      publisher = RobotPublisher()
  
      try:
          # 运行节点
          # rclpy.spin 的作用是让节点实例进入 “自旋状态”，持续处理该实例的回调函数（比如定时器、订阅者回调）
          # 参数传入实例
          rclpy.spin(publisher)  # 保持节点运行（阻塞）
      except KeyboardInterrupt:
          pass
      # 清理
      publisher.destroy_node()
      rclpy.shutdown()
         
  
  if __name__ == '__main__':
      main()        
  
  ```

- **订阅者代码：**

- ```python
  #!/usr/bin/env python3
  import rclpy
  from rclpy.node import Node
  from test_pkg.msg import Robot  # 导入自定义消息
  
  class RobotSubscriber(Node):
      def __init__(self):
          super().__init__('subscriber_robot_node')
          # 1. 创建订阅者：订阅话题「/robot_status」，收到消息后调用 callback 函数
          self.subscription = self.create_subscription(
              Robot,
              '/Robot_status',
              self.robot_msg_callback,  # 回调函数（处理收到的消息）
              10  # 队列大小
          )
          self.subscription  # 防止未使用变量警告
  
      def robot_msg_callback(self, msg):
          # 2. 解析自定义消息的字段（直接通过 msg.字段名 获取）
          self.get_logger().info("\n接收到机器人数据：")
          self.get_logger().info(f"ID：{msg.id}")
          self.get_logger().info(f"坐标（X,Y）：({msg.x:.2f}, {msg.y:.2f})")
          self.get_logger().info(f"运行状态：{msg.status}")
          self.get_logger().info(f"是否移动：{msg.is_moving}")
  
  def main(args=None):
      rclpy.init(args=args)
      subscriber = RobotSubscriber()
      try:
          # 运行节点
          # rclpy.spin 的作用是让节点实例进入 “自旋状态”，持续处理该实例的回调函数（比如定时器、订阅者回调）
          # 参数传入实例
          rclpy.spin(subscriber)  # 保持节点运行（阻塞）
      except KeyboardInterrupt:
          pass
  
      subscriber.destroy_node()
      rclpy.shutdown()
         
  
  if __name__ == '__main__':
      main()
  ```

  

#### 3.0.SRV(service)服务定义修改，包括双向、同步、请求

- **双向通信**：有请求必有响应，不像话题（msg）是单向发消息；

- **同步 / 异步可选**：**默认同步（客户端等服务端回复）**，也可异步调用；

- **一对一**：一个请求对应一个响应，适合 “一次性操作”。

- **`.srv`** 文件编写举例：文件命名规定不允许有下划线，只能由字母 / 数字组成

- ```
  # 请求：查询哪个关节的状态（字符串）
  string joint_name
  ---
  # 响应：关节位置（浮点）+ 是否正常（布尔）
  float64 position
  bool is_normal
  ```

  - 或者：

  - ```
    int64 a
    int64 b
    ---        
    int64 sum
    ```

  - **`---`** 分隔符：这是最关键的部分。它将文件分为上下两半

    - **上半部分（请求 Request）**：客户端发送给服务器的数据。这里是 `a` 和 `b`，意思是“请帮我算一下 a 加 b"。
    - **下半部分（响应 Response）**：服务器处理完后返回给客户端的数据。这里是 `sum`，意思是“这是计算结果 sum"。

  - 实际就是定义发送和响应分别采用什么数据，响应使用 **`sum`** 表示计算总和，那么客户端发送 **`a 和 b`** ，那么服务器就应该响应 **` a + b`** 的和

- 修改 **`package.xml`** ，下面的接口smg/srv/action通用

- ```xml
  <!-- ========== 接口相关（msg/srv/action通用） ========== -->
  <build_depend>rosidl_default_generators</build_depend>         <!-- 编译.msg生成Python/C++接口 -->
  <exec_depend>rosidl_default_runtime</exec_depend>              <!-- 运行时加载自定义msg -->
  <member_of_group>rosidl_interface_packages</member_of_group>   <!-- 声明为消息接口包 -->
  ```

  - `exec_depend` 这个标签名代表仅运行阶段需要的依赖（比如运行时加载接口的库）

- 修改**`CMakeList.txt`** 

- ```cmake
  
  # ========== 自定义.配置（ROS2核心） ==========
  # 1. 声明自定义msg文件
  set(msg_files
    "msg/Robot.msg"  # 路径要写完整（msg文件夹下的Robot.msg）
  )
  
  # 2. 声明自定义srv文件（配置srv）
  set(srv_files
    "srv/Check_status.srv"  # 示例：srv文件夹下的AddTwoInts.srv，替换为你的srv文件名
  )
  
  
  # 2. 生成自定义消息以及服务（ROS2的核心指令）
  rosidl_generate_interfaces(${PROJECT_NAME}
    ${msg_files}
    ${srv_files}（配置srv）
    DEPENDENCIES std_msgs  # 依赖的基础消息包
    ADD_LINTER_TESTS       # 可选：开启msg语法检查
  )
  
  # 3. 声明消息运行时依赖（让其他节点能找到自定义消息）
  ament_export_dependencies(rosidl_default_runtime)
  
  
  # ==================== Python部分 ====================
  # 安装Python脚本
  install(PROGRAMS                    # PROGRAMS表示安装 “可执行程序 / 脚本”
    scripts/publisher_node.py         # 脚本路径
    scripts/subscriber_node.py        # 脚本路径
    scripts/publisher_robot_node.py             # 针对msg信息测试的发布者脚本路径
    scripts/subscriber_robot_node.py            # 针对msg信息测试的订阅者脚本路径
    scripts/check_status_server.py              # 服务端节点（Python）（配置srv）建议全部小写
    scripts/check_status_client.py              # 客户端节点（Python）（配置srv）建议全部小写
    DESTINATION lib/${PROJECT_NAME}   # 安装目标路径
  )
  ```

- **服务端代码：**

- ```py
  #!/usr/bin/env python3
  import rclpy
  from rclpy.node import Node
  from test_pkg.srv import Check_status # 导入Check_status接口
  
  class Checkstatus(Node):
      def __init__(self):
          # 初始化父类(Node) 
          # 内部是ROS2节点名（用于网络识别），与脚本名/CMake无强制关联（建议统一命名）
          super().__init__('check_status_server')  
  
          # 开始创建服务对象
          self.srv = self.create_service(
              Check_status,                 # srv配置文件名称
              'get_status',                 # 服务名称
              self.check_status_callback    # 回调函数
          )
          self.get_logger().info("关节状态查询服务已经启动！")   # 打印日志    
  
          # 模拟关节数据库，用字典，字典key不能重复，所以joint1和joint2不一样
          self.joint_database = {
              'joint1':{'position':0.5,'is_normal':True},
              'joint2':{'position':1.2,'is_normal':True},
              'joint3':{'position':-0.8,'is_normal':False},
          }
  
      # 回调函数两个变量传参，一个是客户端请求(request)，一个是服务端响应(response)
      def check_status_callback(self,request,response):       
          joint_name = request.joint_name                      # 服务端接受客户端请求并且存储在本地用于比对是否正确
          self.get_logger().info(f'收到查询请求：关节名 = {joint_name}')  # 打印请求信息
  
          if joint_name in self.joint_database:    #判断请求的信息是否符合规范，即服务端拥有的数据类型
              # 开始给这个response属性赋值进行响应
              response.position = self.joint_database[joint_name]['position']
              response.is_normal  = self.joint_database[joint_name]['is_normal']    
              # 打印日志
              self.get_logger().info(f'查询成功：位置 = {response.position}，状态正常 = {response.is_normal}')  
          else:
              response.position = 0.0
              response.is_normal = False
              self.get_logger().warn(f'查询失败：关节{joint_name}不存在！')
  
          return response
      
  
  def main(args=None):
      rclpy.init(args=args)
      Checkstatus_server = Checkstatus()  # 创建实例
      rclpy.spin(Checkstatus_server)      # 开始运行
      Checkstatus_server.destroy_node() 
      rclpy.shutdown()
  
  if __name__ == '__main__':
      main()
  
  ```

- **客户端代码：** 

- ```python
  #!/usr/bin/env python3
  import sys  # 用于读取命令行传入的关节名参数
  import rclpy
  from rclpy.node import Node
  from test_pkg.srv import Check_status  # 导入和服务端相同的接口
  
  # 定义客户端节点类，继承Node
  class CheckStatusClient(Node):
      def __init__(self):
          # 初始化节点
          # 内部是ROS2节点名（用于网络识别），与脚本名/CMake无强制关联（建议统一命名）
          super().__init__('check_status_client')
          
          # 创建客户端对象：接口类型和服务名必须和服务端完全一致！
          # 接口：Check_status，服务名：get_status（和服务端的'get_status'对应）
          self.cli = self.create_client(Check_status, 'get_status')
          
          # 等待服务端上线（避免客户端先启动找不到服务）
          # timeout_sec=1.0：每隔1秒检查一次服务是否可用
          while not self.cli.wait_for_service(timeout_sec=1.0):
              self.get_logger().info('服务端未上线，等待中...')
          
          # 创建请求对象（用于存放要发送给服务端的参数）
          self.req = Check_status.Request()
  
      # 定义发送请求的函数：传入关节名，返回服务端响应
      def send_request(self, joint_name):
          # 给请求对象赋值：设置要查询的关节名（对应srv里的string joint_name）
          self.req.joint_name = joint_name
          
          # 异步发送请求（ROS2推荐方式，非阻塞）
          self.future = self.cli.call_async(self.req)
          
          # 等待服务端返回响应（阻塞，直到收到结果）
          rclpy.spin_until_future_complete(self, self.future)
          
          # 返回响应结果
          return self.future.result()
  
  # 主函数：程序入口
  def main(args=None):
      # 初始化ROS2环境
      rclpy.init(args=args)
      
      # 检查命令行参数：确保用户传入了要查询的关节名
      # sys.argv[0]是脚本名，sys.argv[1]是第一个参数（关节名）
      if len(sys.argv) != 2:
          print('用法：ros2 run test_pkg check_status_client.py <关节名>')
          print('示例：ros2 run test_pkg check_status_client.py joint1')
          return
      
      # 创建客户端节点实例
      client_node = CheckStatusClient()
      
      # 发送请求：传入命令行参数中的关节名
      response = client_node.send_request(sys.argv[1])
      
      # 打印服务端返回的结果
      client_node.get_logger().info(
          f'===== 关节状态查询结果 =====\n'
          f'查询关节：{sys.argv[1]}\n'
          f'关节位置：{response.position}\n'
          f'是否正常：{response.is_normal}'
      )
      
      # 销毁节点，关闭ROS2环境
      client_node.destroy_node()
      rclpy.shutdown()
  
  # 程序入口
  if __name__ == '__main__':
      main()
  
  
  ```

- **开始启用服务端和客户端，必须先开启服务端，保障客户端查询时服务端可以及时响应**

- ```bash
  cd ~/ROS2_test_WS
  colcon build --packages-select test_pkg     # 编译指定功能包
  source install/setup.bash                   # 加载环境变量
  ros2 run test_pkg check_status_server.py    # 启动服务端节点
  
  # 查询存在的关节
  ros2 run test_pkg check_status_client.py joint1
  
  # 查询不存在的关节（测试异常情况）
  ros2 run test_pkg check_status_client.py joint4
  ```

  

### 3.使用 `wpr` 机器人仿真包思路解析

- 新开终端通过 **`source ~/wpr_ws/install/setup.bash`** 加载环境，然后调用 **`ros2 launch wpr_simulation2 wpb_simple.launch.py`** 启动 **`wpb`** 机器人开启仿真环境。

- 现在我需要获取这个功能包的相关话题名方便进行仿真控制

  - 常用命令：(**如果没有启动仿真环境是无法查到相应的话题**)

    - 查询话题：**`ros2 topic list`** 
    - 查询服务：**`ros2 service list`** 
    - 查询动作：**`ros2 action list`** 
    - 查看特定动作类型：**`ros2 action type <动作名称>`** 
    - 启动简单环境：        **`ros2 launch wpr_simulation2 wpb_simple.launch.py`**  

  - 新开一个终端，使用 **`source ~/wpr_ws/install/setup.bash`** 加载环境变量，一定要指定工作区，防止其他工作区干扰

  - 然后调用 **`ros2 topic list -t`** 查询当前功能包所有的话题，**`-t`** ： 查询消息类型，比如话题信息(**`msg`**)、服务信息(**`srv`**)

  - 以 **`/cmd_vel [geometry_msgs/msg/Twist]`**  为例

    - **`cmd_vel`** 是话题名，**`geometry_msgs`** 是ROS2系统里面的消息包属于ROS 自带的“标准件库”，**`msg`** 是代表话题消息，**`Twist`** 是消息名称

  - **`cmd_vel`** 是标准的移动机器人控制接口，现在通过命令 **`ros2 interface show geometry_msgs/msg/Twist`** 来查询这个接口的消息格式

    - 输出：

    - ```
      Vector3  linear
      	float64 x
      	float64 y
      	float64 z
      Vector3  angular
      	float64 x
      	float64 y
      	float64 z
      ```

    - 现在已经得到了消息话题的格式，可以通过这个格式去控制机器人

- **开始编写发布者节点代码：**

- ```python
  #!/usr/bin/env python3
  # -*- coding: utf-8 -*-
  
  import rclpy
  from rclpy.node import Node
  from geometry_msgs.msg import Twist  #  导入wpr功能包自定义的消息格式
  import time
  
  
  class Publisher_control_Node(Node):
  
      def __init__(self):
          super().__init__('control_wpbrobot')
  
          # 创建发布器
          self.publisher = self.create_publisher(
              Twist,          # wpr功能包自定义的消息格式
              '/cmd_vel',     # 话题名称
              10              # 队列大小
          )
  
          # 首次调用记录启动时间
          self.start_time = time.time()
  
          # 定时器（10ms）
          self.timer = self.create_timer(0.01, self.timer_callback)
  
          self.get_logger().info('控制节点启动...')
  
      def timer_callback(self):
          twist_msg = Twist()     # 创建一个消息对象用来进行发布
  
          # 当前运行时间
          elapsed_time = time.time() - self.start_time
  
          # ===== 控制逻辑 =====
          if elapsed_time < 2.0:        # 时间在2s以内x的线速度保持在0.5m/s
              # 第一阶段
              twist_msg.linear.x = 0.5  
  
          elif elapsed_time < 10.0:      # 时间在2s-10s之内，调整x的线速度，增加Z轴角速度的修改
              # 第二阶段（2s~10s）
              twist_msg.linear.x = 0.2
              twist_msg.angular.z = 0.5
  
          else:                          # 超过10s就发布0的指令，让机器人停止
              # 停止
              twist_msg.linear.x = 0.0
              twist_msg.angular.z = 0.0
  
          # 发布
          self.publisher.publish(twist_msg)
  
          # 打印日志（简化版）
          self.get_logger().info(
              f"time={elapsed_time:.2f}s | vx={twist_msg.linear.x:.2f} | wz={twist_msg.angular.z:.2f}"
          )
  
  
  def main(args=None):
      rclpy.init(args=args)
       # 创建节点，可以说是实例化
      node = Publisher_control_Node()
  
      try:
          rclpy.spin(node)
      except KeyboardInterrupt:
          pass
      finally:
          node.destroy_node()
          rclpy.shutdown()
  
  
  if __name__ == '__main__':
      main()
  ```

- 修改 **`CMakeList.txt`** 的python脚本目录

- ```cmake
  # ==================== Python部分 ====================
  # 安装Python脚本
  install(PROGRAMS                    # PROGRAMS表示安装 “可执行程序 / 脚本”
    scripts/control_wpbrobot.py         # 脚本路径
    DESTINATION lib/${PROJECT_NAME}   # 安装目标路径
  )
  ```

  

#### 3.1.使用激光雷达仿真

- 常见的激光雷达话题是用  "/scan"表示的，使用命令 **`ros2 interface show sensor_msgs/msg/LaserScan`** 开始查看激光雷达的话题的消息格式有哪些

- ```python
  # Single scan from a planar laser range-finder
  #
  # If you have another ranging device with different behavior (e.g. a sonar
  # array), please find or create a different message, since applications
  # will make fairly laser-specific assumptions about this data
  
  # timestamp(时间戳) 表示本次扫描中接收到第一束激光反射信号的时间
  std_msgs/Header header              # 传感器消息必带的"消息头"，作为传感器的身份证，header为变量名
  	builtin_interfaces/Time stamp
  		int32 sec                   # 秒
  		uint32 nanosec				# 纳秒
  	string frame_id         	    # 激光雷达所在的坐标系，可以区分不同传感器
                               # the first ray in the scan.
                               #
                               # in frame frame_id, angles are measured around
                               # the positive Z axis (counterclockwise, if Z is up)
                               # with zero angle being forward along the x axis
  
  float32 angle_min            # 扫描起始角度 [rad]
  float32 angle_max            # 扫描结束角度 [rad]
  float32 angle_increment      # 每次测量之间的角度增量 [rad]
  
  float32 time_increment       # 每个测量之间的时间（秒）              
                     
  float32 scan_time            # 完整一圈扫描所需时间
  
  float32 range_min            # 雷达最小量程
  float32 range_max            # 雷达最大量程
  
  # 核心数据
  float32[] ranges             # 实测距离 [m]
                               # (Note:不在量程内的数据直接丢弃)
  float32[] intensities        # # 信号强度（可选）
                               # device does not provide intensities, please leave
                               # the array empty.
  
  ```

  - 每个传感器都有一张身份证（header)

    ```
    header（身份证）
    ├── stamp（时间戳）时间戳 = 秒 + 纳秒
    │    ├── sec ：秒
    │    └── nanosec ：纳秒
    └── frame_id ：坐标系名字（例如 laser、base_link、odom 等）
    ```

    - **`frame_id`** 是每个传感器的坐标系名称用来区分不同的传感器，而 **`stamp`** 时间戳一般每个传感器是不一样的，不同传感器需要统一时间戳，时间戳 = 秒 + 纳秒，注意纳秒需要换算为秒， 秒 = 纳秒 / 10 亿

  - 角度信息
    - 激光扫描通常是扇形或者 360°
    - **angle_min → angle_max** 表示扇形范围
    - **angle_increment** 表示数组中相邻两个测距值的角度差

- 使用 **`RViz2`** 进行数据可视化，首先通过命令 **`ros2 topic echo /scan | grep frame_idid`** 查询仿真环境里面的坐标系是什么
  - 这里返回了 **` frame_id: laser_link`** ，所以在 **`RViz2`** 左侧的 **`gloabl option`** 坐标系里面选择 **`laser_link`** ，然后左下角添加话题，点击 **`Add`** ，在 **`By topic`** 里面选择我们的雷达数据话题 **`/scan`** ，添加完毕就会显示出来，然后将配置保存下来(**`Save Config As`** )，然后自定义命名即可，下次点开选择一个配置即可。
  - **`RViz2`** 可以适配几乎所有雷达数据显示(3d雷达需要配置插件)，所有2D雷达的话题消息格式都遵循同一个协议：**`LaserScan`** ，是 ROS 生态系统的 **通用标准接口 **  

#### 3.2.利用激光雷达测距实现障碍物转弯

- 首先我们需要进行测距，针对激光雷达真前方的距离测算，我们应该选择雷达的 **`ranges`**  数组中间位置，我们以360°平面雷达举例，数组一共有360个(假设分辨率为1°)，雷达的正前方刚好是180°，角度从右往左增加，一共360°

- **编写节点：** 

- ```python
  #!/usr/bin/env python3
  # -*- coding: utf-8 -*-
  
  import rclpy
  from rclpy.node import Node
  from sensor_msgs.msg import LaserScan   # 导入激光雷达的话题消息格式
  from geometry_msgs.msg import Twist     # 导入WPB的控制速度和角度的消息格式
  
  class ObstacleAvoid(Node):
      def __init__(self):
          super().__init__('obstacle_avoid_node')
  
          # 订阅雷达 /scan
          self.scan_sub = self.create_subscription(
              LaserScan, '/scan', self.scan_callback, 10
          )
  
          # 发布速度 /cmd_vel，给定一个初始速度
          self.cmd_pub = self.create_publisher(Twist, '/cmd_vel', 2)
  
          # 障碍物距离阈值（小于这个值就拐弯）
          self.obstacle_dist = 0.4  # 单位：米
  
      def scan_callback(self, msg):
          # --------------------------
          # 1. 取正前方的雷达数据
          # --------------------------
          # 正前方是 ranges 数组中间位置
          mid = len(msg.ranges) // 2
          front_dist = msg.ranges[mid]
  
          # 过滤无效值
          if front_dist == float('inf') or front_dist < 0.1:
              front_dist = 999
  
          self.get_logger().info(f'正前方距离：{front_dist:.2f} m')
  
          # --------------------------
          # 2. 控制逻辑：有障碍就拐弯
          # --------------------------
          twist = Twist()
  
          if front_dist < self.obstacle_dist:
              # 障碍 → 停止前进 + 左转
              twist.linear.x = 0.0
              twist.angular.z = 0.4  # 左转
              self.get_logger().info("⚠️ 障碍物！左转避让")
          else:
              # 无障碍 → 直走
              twist.linear.x = 0.2   # 前进
              twist.angular.z = 0.0
  
          # 发布速度
          self.cmd_pub.publish(twist)
  
  def main(args=None):
      rclpy.init(args=args)
      node = ObstacleAvoid()
      rclpy.spin(node)
      node.destroy_node()
      rclpy.shutdown()
  
  if __name__ == '__main__':
      main()
  ```

  

#### 3.3.IMU

| 传感器                           | 测量物理量                   | 典型用途                               |
| -------------------------------- | ---------------------------- | -------------------------------------- |
| **加速度计**（Accelerometer）    | 三轴线性加速度（含重力）     | 检测运动状态、倾斜角度、振动           |
| **陀螺仪**（Gyroscope）          | 三轴角速度(大多是使用四元数) | 测量旋转速率、姿态变化                 |
| **磁力计**（Magnetometer，可选） | 三轴地磁场强度               | 辅助航向角（偏航角）解算，构成**AHRS** |

##### 3.3.1.四元数

- **四元数 = 用来表示 3D 旋转的数学工具** 它比欧拉角更稳定、不会卡死、计算更快，但确实不直观。

  - 欧拉角：用 **3 个角度（俯仰、偏航、滚转）** 描述旋转，很直观，但会出现**万向锁**(两个轴会有概率重合导致计算时自由度丢失)。

  - 四元数：用 **4 个数字 (w, x, y, z)** 描述旋转，不直观，但**数学干净、不会锁死**。

- **一个 3D 旋转，本质上只有两件事：**

  - **绕哪条轴旋转？**（单位向量：x, y, z）

  - **转多少角度？**（θ）

- **四元数就是把这两件事打包成 4 个数：**

  - $$q = w + xi + yj + zk$$

  - **其中：**

    - $$w$$ 跟**角度**有关

    - $$x,y,z$$ 跟**旋转轴**有关

  - **标准写法：**

​		$$q = \left( \cos\frac{\theta}{2},\ \sin\frac{\theta}{2}\cdot v_x,\ \sin\frac{\theta}{2}\cdot v_y,\ \sin\frac{\theta}{2}\cdot v_z \right)$$

​		\> 记法：(实部 w, 虚部 x, 虚部 y, 虚部 z)

- **例：**绕 Y 轴转 90°

  - 旋转轴：Y 轴 → (0,1,0)

  - 角度：θ = 90° = π/2

​	$$\begin{aligned} w &= \cos(\theta/2) = \cos45^\circ = \frac{\sqrt{2}}{2} \\ x &= \sin(\theta/2)\cdot 0 = 0 \\ y &= \sin(\theta/2)\cdot 1 = \frac{\sqrt{2}}{2} \\ z &= 0 \end{aligned}$$

​	所以四元数：

​	$$q = \left(\frac{\sqrt{2}}{2},\ 0,\ \frac{\sqrt{2}}{2},\ 0\right)$$

- **四元数如何控制一个坐标它绕指定轴旋转固定角度计算出新的坐标**

  - 这是核心公式，记住就行：

  - $$p' = q \cdot p \cdot q^{-1}$$
    - $$p$$：要旋转的点，写成四元数：$$p=(0, x, y, z)$$
    - $$q$$：旋转四元数(主要部分，包含了绕哪个轴旋转，转的角度大小)
    - $$q^{-1}$$：四元数的逆（单位四元数直接取虚部反号即可）

​	单位四元数满足 $$|q|=1$$(模长等于1，不会被拉伸)，所以：

​	$$q^{-1} = (w, -x, -y, -z)$$

- **四元数乘法规则**

  - 四元数有三个虚单位：
    - $$i^2 = j^2 = k^2 = ijk = -1$$

  - 乘法顺序：
    - $$\begin{aligned} ij &= k,\quad ji=-k\\ jk &= i,\quad kj=-i\\ ki &= j,\quad ik=-j \end{aligned}$$

  - 两个四元数相乘：
    - $$q_1 = (w_1,x_1,y_1,z_1)\\ q_2 = (w_2,x_2,y_2,z_2)$$
      - $$\begin{aligned} w &= w_1w_2 - x_1x_2 - y_1y_2 - z_1z_2\\ x &= w_1x_2 + x_1w_2 + y_1z_2 - z_1y_2\\ y &= w_1y_2 - x_1z_2 + y_1w_2 + z_1x_2\\ z &= w_1z_2 + x_1y_2 - y_1x_2 + z_1w_2 \end{aligned}$$

- **快速看懂一个四元数的技巧**

  - 给你一个四元数 $$q=(w,x,y,z)$$

  1. 先看 $$w$$
     1. $$w≈1$$ → 几乎没旋转
     2. $$w≈0$$ → 旋转接近 180°

  2. 再看 (x,y,z)：这就是**旋转轴方向**。

  3. 模长必须接近 1        $$w^2+x^2+y^2+z^2 \approx 1$$不是 1 就不是单位四元数，要归一化。

- **最常用公式汇总**

  - 角度+轴 → 四元数
    - $$\begin{aligned} w &= \cos(\theta/2)\\ x &= \sin(\theta/2)\cdot v_x\\ y &= \sin(\theta/2)\cdot v_y\\ z &= \sin(\theta/2)\cdot v_z \end{aligned}$$

  - **四元数 → 轴角**

    - $$\theta = 2\arccos(w)$$

    - $$(x,y,z) = \frac{\text{虚部}}{\sin(\theta/2)}$$

- **旋转点**
  - $$p' = qpq^{-1}$$

- **连续旋转**

  - 先转 $$q_1$$ 再转 $$q_2$$

  - $$q = q_2 q_1$$

​	（注意顺序不能反）



##### 3.3.2.IMU获取数据

- 新开终端通过 **`source ~/wpr_ws/install/setup.bash`** 加载环境，然后调用 **`ros2 launch wpr_simulation2 wpb_simple.launch.py`** 启动 **`wpb`** 机器人开启仿真环境。

- 开始调用 **`ros2 topic list -t`** 查看一下IMU的话题格式

  - 话题格式如下：

  - ```
    /imu/data [sensor_msgs/msg/Imu]
    ```

  - 使用命令查看话题格式内容：**`ros2 interface show sensor_msgs/msg/Imu`**

  - ```
    # This is a message to hold data from an IMU (Inertial Measurement Unit)
    #
    # Accelerations should be in m/s^2 (not in g's), and rotational velocity should be in rad/sec
    #
    # If the covariance of the measurement is known, it should be filled in (if all you know is the
    # variance of each measurement, e.g. from the datasheet, just put those along the diagonal)
    # A covariance matrix of all zeros will be interpreted as "covariance unknown", and to use the
    # data a covariance will have to be assumed or gotten from some other source
    #
    # If you have no estimate for one of the data elements (e.g. your IMU doesn't produce an
    # orientation estimate), please set element 0 of the associated covariance matrix to -1
    # If you are interpreting this message, please check for a value of -1 in the first element of each
    # covariance matrix, and disregard the associated estimate.
    
    # 这是一条用于存储来自惯性测量单元(IMU)数据的消息。
    # 加速度应以m/s2为单位(而非g)，旋转速度应以rad/sec为单位
    # 如果测量的协方差已知，则应填写(如果你只知道)
    # 每个测量值的方差，例如来自数据表，只需将这些值沿对角线排列
    # 全为零的协方差矩阵将被解释为“协方差未知”，并使用数据协方差必须假设或从其他来源获取
    # 如果你对其中一个数据元素没有估计值(例如，你的IMU没有产生一个
    # (方向估计)，请将相关协方差矩阵的第0个元素设为-1。
    # 如果您正在解释此消息，请检查每个元素的第一个值是否为-1。
    # 协方差矩阵，且忽略相关估计值。
    
    std_msgs/Header header
    	builtin_interfaces/Time stamp
    		int32 sec
    		uint32 nanosec
    	string frame_id
    
    geometry_msgs/Quaternion orientation       # 四元数：描述角度旋转
    	float64 x 0
    	float64 y 0
    	float64 z 0
    	float64 w 1
    float64[9] orientation_covariance # Row major about x, y, z axes  方向协方差矩阵
    
    geometry_msgs/Vector3 angular_velocity     # 角速度
    	float64 x
    	float64 y
    	float64 z
    float64[9] angular_velocity_covariance # Row major about x, y, z axes   角速度协方差矩阵
    
    geometry_msgs/Vector3 linear_acceleration  #  线性加速度
    	float64 x
    	float64 y
    	float64 z
    float64[9] linear_acceleration_covariance # Row major x, y z     线性加速度协方差矩阵
    
    ```

- 编写节点程序：

- ```python
  #!/usr/bin/env python3
  # -*- coding: utf-8 -*-
  
  import time
  import rclpy
  from rclpy.node import Node
  from sensor_msgs.msg import LaserScan
  from geometry_msgs.msg import Twist
  from sensor_msgs.msg import Imu
  
  class ImuScanLogger(Node):
      def __init__(self):
          super().__init__('imu_scan_wpbrobot')
  
          # 订阅IMU数据 /Imu
          self.imu_sub = self.create_subscription(
              Imu, '/imu/data', self.imu_callback, 10
          )
  
          # 订阅雷达 /scan
          self.scan_sub = self.create_subscription(
              LaserScan, '/scan', self.scan_callback, 10
          )
  
          # 防止订阅器被垃圾回收（可选，但建议添加）
          self.imu_sub  # 无需赋值，仅占位
          self.scan_sub
  
          # ========================
          # 100ms 打印控制
          # ========================
          self.last_print_time = time.time()
          self.update_interval = 0.1  # 100ms
          
          # 数据缓冲区
          self.latest_imu = None
          self.latest_scan = None
  
      # 激光雷达回调函数,里面做了初步处理，只取正前方中间的传感距离
      def scan_callback(self,msg):
          # --------------------------
          # 1. 取正前方的雷达数据
          # --------------------------
          # 正前方是 ranges 数组中间位置
          mid = len(msg.ranges) // 2
          front_dist = msg.ranges[mid]  
          # 过滤无效值
          if front_dist == float('inf') or front_dist < 0.1:
              front_dist = 999.0
          self.latest_scan = front_dist
  
  
  
      # 回调函数
      def imu_callback(self, msg):
          self.latest_imu = msg         # 数据缓冲，储存到另一个对象
  
      # ========================
      # 统一打印（100ms一次）
      # ========================
      def print_data(self):
          current_time = time.time()
          if current_time - self.last_print_time < self.update_interval:
              return
          self.last_print_time = current_time
  
          # --------------------------
          # 打印雷达
          # --------------------------
          if self.latest_scan is not None:
              self.get_logger().info(f'正前方距离：{self.latest_scan:.2f} m')
  
          # --------------------------
          # 打印IMU
          # --------------------------
          if self.latest_imu is not None:
              msg = self.latest_imu
              imu_list = [0.0] * 10             # 创建一个列表 [0,0,0,0,0,0,0,0,0,0]，内部用浮点数初始化
              imu_list[0] = msg.orientation.x
              imu_list[1] = msg.orientation.y
              imu_list[2] = msg.orientation.z
              imu_list[3] = msg.orientation.w
              imu_list[4] = msg.angular_velocity.x
              imu_list[5] = msg.angular_velocity.y
              imu_list[6] = msg.angular_velocity.z
              imu_list[7] = msg.linear_acceleration.x
              imu_list[8] = msg.linear_acceleration.y
              imu_list[9] = msg.linear_acceleration.z
  
              self.get_logger().info(
                  f'姿态四元数：[{imu_list[0]:.4f}, {imu_list[1]:.4f}, {imu_list[2]:.4f}, {imu_list[3]:.4f}] | '
                  f'角速度：[{imu_list[4]:.4f}, {imu_list[5]:.4f}, {imu_list[6]:.4f}] | '
                  f'加速度：[{imu_list[7]:.4f}, {imu_list[8]:.4f}, {imu_list[9]:.4f}]'
              )
  
  
  def main(args=None):
      rclpy.init(args=args)
      node = ImuScanLogger()
  
      # 主循环：10ms轮询，100ms打印
      try:
          while rclpy.ok():
              rclpy.spin_once(node, timeout_sec=0.001)
              node.print_data()
      except KeyboardInterrupt:
          node.get_logger().info('节点已手动关闭')
      finally:
          node.destroy_node()
          rclpy.shutdown()
  
  if __name__ == '__main__':
      main()
  
  
  
  ```

  - 回调函数里面不放入打印，容易导致时间延迟，统一在外围100ms来进行打印

#### 3.4.SLAM

> 常用命令：`pkill -f ros` 删除所有ros进程

##### 3.4.1.wpr的slam启动文件集成

> 核心： 跨功能包启动，启动 **`wpr`** 仿真包里面的 **`slam`** 导航(在仿真已经编译和加载环境成功的前提下)

- 直接复制官方集成slam启动文件(增加了遥控漫游功能)：

- ```python
  #!/usr/bin/env python3
  import os
  from ament_index_python.packages import get_package_share_directory   # 跨包运行核心依赖
  from launch import LaunchDescription                 				  # 启动文件的 “总容器”
  from launch.actions import IncludeLaunchDescription    				  # 启动别人写好的 launch 文件。
  from launch.launch_description_sources import PythonLaunchDescriptionSource
  from launch_ros.actions import Node
  
  def generate_launch_description():
      # 查找功能包路径，去相应的功能包里面找对应的启动文件，用于跨包启动
      # os.path.join ：进入这个launch文件
      launch_file_dir = os.path.join(get_package_share_directory('wpr_simulation2'), 'launch')
  
      # gazebo_cmd启动 Gazebo 仿真环境
      gazebo_cmd = IncludeLaunchDescription(
          # robocup_home.launch.py里面包含：Gazebo 场景，家具,墙壁,机器人,激光雷达,里程计
          PythonLaunchDescriptionSource(
              os.path.join(launch_file_dir, 'robocup_home.launch.py')  
          )
      )
  	# 启动 SLAM 建图节点
      slam_cmd = Node(
          # 建图功能包
          package="slam_toolbox",
          executable="sync_slam_toolbox_node",  # 建图程序。
          parameters=[{
              "use_sim_time": True,             # 用仿真时间，不然建图失败
              "base_frame": "base_footprint",
              "odom_frame": "odom",                 
              "map_frame": "map",             # 机器人坐标系，固定写法不用改。
              "resolution": 0.025,  			# 调整至2.5厘米分辨率 (相比于原来的5cm更清晰)
          }]
      )
  
      # 可视化工具
      rviz_cmd = Node(
          package='rviz2',
          executable='rviz2',
          name='rviz2',
          # 加载官方配置好的文件 自动显示：地图、雷达、模型、坐标
          arguments=['-d', os.path.join(get_package_share_directory('wpr_simulation2'), 'rviz', 'slam.rviz')]
      )
  
      # 启动键盘遥控
      keyboard_teleop_cmd = Node(
          package='wpr_simulation2',
          executable='keyboard_vel_cmd',
          output='screen',
          prefix='xterm -e',                  # 弹出新终端
          parameters=[{'use_sim_time': True}]
      )
  
      # 创建一个 启动列表
      ld = LaunchDescription()
      
      # 把要启动的功能加进去
      ld.add_action(gazebo_cmd)
      ld.add_action(slam_cmd)
      ld.add_action(rviz_cmd)
      ld.add_action(keyboard_teleop_cmd)
  	
      # 执行启动列表
      return ld
  ```
  
  - **`keyboard_teleop_cmd`** 和 **`slam_cmd`** 以及 **`rviz_cmd`** 的语法是常见的launch启动写法，启动功能包的对应节点，而 **`gazebo_cmd`** 是直接调用了仿真包里面现有的启动文件



##### 3.4.2.SLAM地图序列化和反序列化

- <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260403171230476.png" alt="image-20260403171230476" style="zoom:50%;" />

- 这里右侧的 **`/home/wenjun/maps_wpr_ws/map1`** 是保存的序列化文件绝对路径，如果需要反序列化也需要绝对路径，这里需要提前自己创建一个文件目录用来保存，否则会保存失败

- **`Serialize map`** ：序列化地图

  - 保存 **完整的位姿图（Pose Graph）+ 所有关键帧激光数据 + 元数据**。**可以下次加载进来继续建图、修改、优化、合并地图**。

  - 文件会保存为 **`.posegraph`**  和 **` .data`** 这两个文件格式。
    - **`.posegraph`** :  **可读的文本文件**（可以用文本编辑器直接打开），位姿图 + 元数据，地图的骨架
    - **` .data`** : 原始激光数据，作为二进制文件，不可读，所有关键帧的原始激光扫描数据，传感器标定参数，以及其他辅助数据，比如里程计和IMU数据

  - 相比软件操作，还可以通过命令行（依据下面的格式）进行序列化保存：

  - ```bash
    ros2 service call /slam_toolbox/deserialize_map slam_toolbox/srv/DeserializePoseGraph "{filename: '/home/wenjun/maps_wpr_ws/map1'}"
    ```

- **`Deserialize`** ： 反序列化地图

  - 导入保存好的序列化地图，在原有的基础上继续进行建图

  

##### 3.4.3.SLAM导航加载栅格地图

> 我们常用的用于定位和导航地图是栅格地图，栅格地图保存后不能用于继续建图

<img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260403172919382.png" alt="image-20260403172919382" style="zoom:50%;" />

- 保存为 **`.pgm `** 和 **` .yaml`** 两种格式的文件，

  -  **`.pgm（Portable Gray Map）`** : 便携式灰度图像格式，包含了2D栅格地图数据
    - 每个像素代表一个栅格单元的状态：
    - **空闲区域（通常用白色/高值表示）**
    - **障碍物（通常用黑色/低值表示）**
    - **未知区域（通常用灰色/中间值表示）**

  - **`.yaml`** : 地图的配置文件
    - **分辨率**（resolution）：每个像素代表实际多少米
    - **原点坐标**（origin）：地图的起始位置 [x, y, yaw]
    - **占用阈值**（occupied_thresh）：判定为障碍物的阈值
    - **自由阈值**（free_thresh）：判定为空闲区域的阈值
    - **图像文件路径**：指向对应的 **`.pgm`** 文件

- 保存下来的栅格地图文件可以直接发布出去，然后利用**`rivz`**可视化来查看效果，**`rivz`** 本质上就是订阅了**`/map`** 话题

  - **方法一：修改启动文件**

  - ```python
    # 注释掉slam建图节点,停止实时建图，不再消耗 CPU 计算位姿与栅格更新
    # slam_cmd = Node(
    #     package="slam_toolbox",
    #     executable="sync_slam_toolbox_node",
    #     parameters=[{
    #         "use_sim_time": True,
    #         "base_frame": "base_footprint",
    #         "odom_frame": "odom",
    #         "map_frame": "map",
    #         "resolution": 0.025,  # 新增：2.5厘米分辨率 (更清晰)
    #     }]
    # )
    
    
    # 3. 新增：map_server加载栅格地图
    map_server_cmd = Node(
        package='nav2_map_server',    # 指定节点所属的功能包。Nav2 将地图服务独立为此包
        executable='map_server',
        name='map_server',
        output='screen',			  # 将节点的标准输出（print/log）直接打印到终端
        parameters=[				  # 开始定义传递给节点的参数列表。
            {'yaml_filename': '/home/wenjun/maps_wpr_ws/map1.yaml'},  # 加载地图，里面传入路径
            {'use_sim_time': True}        # 仿真时间,告诉节点不使用系统真实时间，而是订阅 /clock 话题获取仿真时间。在 Gazebo/Webots 或带时钟源的环境中必须开启，否则时间戳不同步会导致 Rviz 不显示。
        ]
    )
    
    # 4. 新增：生命周期管理器（激活地图）
    lifecycle_cmd = Node(
        package='nav2_lifecycle_manager',
        executable='lifecycle_manager',   # 管理器可执行文件。它的作用是集中控制其他 Lifecycle Node 的状态迁移。
        # 管理器自身的节点名。加 _map 后缀是为了与可能存在的其他管理器（如控制 AMCL 的）区分开，避免命名冲突。
        name='lifecycle_manager_map',     
        output='screen',  				  # 将节点的标准输出（print/log）直接打印到终端
        parameters=[
            {'use_sim_time': True},
             # 自动激活，核心参数：Launch 启动后，管理器会自动向列表中的节点发送状态切换指令
            {'autostart': True},     	 
            {'node_names': ['map_server']}     # 传入的参数名字要求和map_server_cmd里面的name一样
        ]
    )
    ```

  - **底层逻辑运作：**

  -  **`map_server`**(**只读不写**)： 是一个生命周期节点(**`lifecycle`**)需要配合  **`lifecycle_manager`** 使用

    1. **用来读取 `.yaml` 配置进行解析获取 4 个关键元数据：**

       - `resolution`（分辨率：1个像素=多少米）
       - `origin`（原点：地图在 ROS 世界坐标系中的 `[x, y, yaw]`）
       - `occupied_thresh` / `free_thresh`（黑白阈值）
       - `image`（指向 `.pgm` 图片的路径）

    2. **读取 `.pgm` 地图文件，将灰度图像转换成内存中的 二维整型数组**

       - `0` = 空闲（白色）
       - `100` = 障碍（黑色）
       - `-1` = 未知（灰色）

    3. **打包发布**：将二维数组 + 元数据封装成 ROS 2 标准消息类型 **`nav_msgs/msg/OccupancyGrid`** (Nav2架构)，并**持续循环发布到 `/map` 话题**。

       > SLAM 是“画家”（实时扫描、计算、绘制）；`map_server` 是“画廊播放器”（读取已经画好装裱的画作，循环展示给观众）。你注释掉 SLAM 完全正确，因为现在你只需要“展示旧画”，不需要“实时作画”。

  - **`lifecycle_manager(本质是一个状态机)`** ： 用于自动化唤醒生命周期节点，一般的生命周期节点不会自动唤醒，有着严格的唤醒顺序，按依赖顺序自动转换所有 Nav2 节点

    **生命节点的常见状态：`lifecycle_manager` 的作用就是**自动化这个唤醒过程

    | 状态                     | 含义                                                 | 此时 `map_server` 在干嘛？         |
    | :----------------------- | :--------------------------------------------------- | :--------------------------------- |
    | `unconfigured`（未配置） | 节点进程刚启动，未加载任何参数                       | 等待指令                           |
    | `inactive`（未激活）     | 已加载 yaml/pgm 到内存，数据就绪，但**不对外发数据** | 地图已读入内存，处于待命状态       |
    | `active`（已激活）       | 开始正常工作，向外发布话题                           | **开始向 `/map` 循环发送地图数据** |

    - 你配置了 **`autostart: true`** 和 **`node_names: ['map_server']`**

    - Launch 启动后，Manager 会自动通过 ROS 2 底层服务（**`/map_server/change_state`**）依次发送： `unconfigured` → **`configure` → `inactive` → `activate` → `active`*

  

  - **方法二：命令行直接修改**

  - ```bash
    # 杀死slam建图节点
    pkill -f slam_toolbox
    
    # 启动 map_server 加载地图，这里会进行等待激活我们的生命节点，需要依靠我们的生命周期管理器
    ros2 run nav2_map_server map_server --ros-args \
    -p yaml_filename:="/home/wenjun/maps_wpr_ws/map1.yaml" 
    -p use_sim_time:=true
    
    # 单开一个终端，加载环境后启动生命周期管理器激活地图
    ros2 run nav2_lifecycle_manager lifecycle_manager --ros-args \
    -p autostart:=true \
    -p node_names:="[map_server]" \
    -p use_sim_time:=true
    ```
    
    - **`nav2_map_server`** ： 		地图管理器
    - **`nav2_lifecycle_manager`** ： **生命周期管理器，自动管理 Nav2 所有节点的启动、配置、激活、关闭**
      - **`node_names`** ： 这里传入你需要管理的节点，这里是地图管理节点

#### 3.5.NAV2实现路径规划(通过NAV2启动文件)

##### 3.5.1.**首先加载栅格地图(上述有描述)**

- ```bash
  # 加载slam相关启动文件，通常包含slam建模和rviz和gazebo
  ros2 launch wpr_user slam.launch.py
  ```

##### 3.5.2.**删除slam建图程序，方便导入现有的栅格图**

- ```bash
  pkill -f slam_toolbox
  ```

##### 3.5.3.**启动定位并且加载地图**

- ```bash
  ros2 launch nav2_bringup localization_launch.py \
  map:=/home/wenjun/maps_wpr_ws/map1.yaml \
  use_sim_time:=true
  ```

- **启动文件源码：**

- ```python
  #!/usr/bin/env python3
  #
  
  import os
  from ament_index_python.packages import get_package_share_directory
  from launch import LaunchDescription
  from launch.actions import IncludeLaunchDescription
  from launch.launch_description_sources import PythonLaunchDescriptionSource
  from launch.substitutions import LaunchConfiguration
  from launch_ros.actions import Node
  
  def generate_launch_description():
      # 实用仿真时间
      use_sim_time = LaunchConfiguration('use_sim_time', default='true')
      
      # 给定一个路径，用来查找仿真包的相关环境启动文件
      launch_file_dir = os.path.join(get_package_share_directory('wpr_simulation2'), 'launch')
  
      # 启动一个家庭的仿真环境
      home_cmd = IncludeLaunchDescription(
          PythonLaunchDescriptionSource(
              os.path.join(launch_file_dir, 'robocup_home.launch.py')
          )
      )
  	# 行为树文件（用于导航逻辑）
      behavior_tree = os.path.join(
          get_package_share_directory('wpr_simulation2'),
          'config',
          'behavior_tree.xml'
      )
  	# 导入地图配置文件，如果自己有现成的配置文件，又不想动这个源代码，那么可以在命令里面加入参数
      # 比如：map:=/home/wenjun/maps_wpr_ws/map1.yaml 
      map_file = os.path.join(
          get_package_share_directory('wpr_simulation2'),
          'maps',
          'map.yaml'
      )
      
  	# 导入NAV2的全参数文件，通常会有默认参数
      nav_param_file = os.path.join(
          get_package_share_directory('wpr_simulation2'),
          'config',
          'nav2_params.yaml'
      )
  	# 核心：官方的启动目录
      nav2_launch_dir = os.path.join(
          get_package_share_directory('nav2_bringup'), 
          'launch'
      )
  
      # 核心：启动完整 Nav2 导航，主要是打开bringup_launch.py
      navigation_cmd = IncludeLaunchDescription(
          PythonLaunchDescriptionSource([nav2_launch_dir, '/bringup_launch.py']),
          launch_arguments={
              'map': map_file,
              'use_sim_time': use_sim_time,
              'params_file': nav_param_file}.items(),
      )
  	# 打开rviz的虚拟环境，用来演示导航效果
      rviz_cmd = Node(
              package='rviz2',
              executable='rviz2',
              name='rviz2',
              arguments=['-d', [os.path.join(get_package_share_directory('wpr_simulation2'), 'rviz', 'navi.rviz')]]
          )
  
      ld = LaunchDescription()
  
      # Add the commands to the launch description
      ld.add_action(home_cmd)
      ld.add_action(navigation_cmd)
      ld.add_action(rviz_cmd)
  
      return ld
  
  ```

- **启动文件里面调用的核心NAV2文件源码：`bringup_launch.py`** ，包含下面几个部分

  - **`map_server`**（地图服务）
  - **`AMCL`**（定位）
  - **`controller`**（路径跟踪）
  - **`planner`**（全局路径规划）
  - **`recoveries`**（异常恢复）
  - **`bt_navigator`**（行为树导航）
  - **`lifecycle manager`**（生命周期管理）
  -  **`tf`坐标转换**

  

##### 3.5.4.**再启动导航的路径规划器和控制器(必须加`map:=empty`，不抢地图)**

- ```
  ros2 launch nav2_bringup navigation_launch.py \
  use_sim_time:=true \
  map:=empty
  ```

- 

##### 3.5.5.RViz 关键操作（补全 TF 链)

- **点 2D Pose Estimate 按钮**：在 RViz 地图上，找到机器人真实位置，点一下并拖拽方向（和机器人朝向一致）
  - 这一步是给 AMCL 初始位姿，AMCL 收到后会立刻发布`map→odom`的变换，TF 链直接补全

- **点 2D Goal Pose 按钮**：在地图上点目标点，路径会正常规划，机器人自动沿路径行驶

- <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260413171337491.png" alt="image-20260413171337491" style="zoom:50%;" />





##### 3.5.6.rivz路径可视化

- 在 `rivz` 实现可视化，添加路径读取path的话题内容： **`Add → Path → topic: /plan`**  
  - <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260413112305022.png" alt="image-20260413112305022" style="zoom: 33%;" />

  



#### 3.6.NAV2路径规划相关功能层级结构分析

**层级：**根基 → 基础 → 核心 → 调度 → 保障 → 管理

##### 3.6.1.根基-`tf`坐标变换链

- **变换链：**通过 `AMCL` 发布的 `map` 坐标系推断到 `base_link`，

  - 通过验证不同坐标系之间的变换去判断哪个节点出现问题了

  - ```
    map（地图坐标系）
      ←发布者：【AMCL发布】
    odom（里程计坐标系）
      ←发布者：【仿真/实机里程计发布（Gazebo/底盘驱动）】
    base_footprint（机器人底盘）
      ←发布者：【机器人模型发布】
    base_link（机器人主体）
      ↓ 【发布者：robot_state_publisher（URDF模型）】
    雷达/摄像头等传感器
    ```

- 典型 `tf` 坐标链终端错误

  - <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260413213016855.png" alt="image-20260413213016855" style="zoom:33%;" />

  - 图片为tf坐标正常变换的效果，如果出现错误就是 `tf` 坐标转换链断了

##### 3.6.2.基础-map server地图服务器和ACML算法

1. map_server（地图服务器）

- 作用：**加载离线地图**，发布 `/map` 话题
- 依赖：只需要 TF 的 `map` 坐标系
- 谁用它：AMCL、planner、bt_navigator
- 注意：**全局只能有 1 个**，多开就冲突（**之前踩的坑**）

2. AMCL（激光定位算法）

- 作用：**机器人定位核心**，匹配雷达 + 地图，计算机器人在地图里的位置
- 核心贡献：**发布 `map → odom` 变换**（补上 TF 最关键的一段）
- 依赖：map_server 的地图、雷达数据、TF坐标变换链
- **之前的警告：`Please set the initial pose` = 没给初始位置，无法定位**

##### 3.6.3.核心-全局路径规划器和路径控制器

1. planner（全局路径规划器）

- 作用：**算大路线**（从起点到终点的整体路径，不考虑动态实时避障）
- 依赖：map_server 地图、AMCL 定位、TF
- 输出：一条全局的粗路径

2. controller（局部控制器 / 路径跟踪）

- 作用：**司机开车**，沿着全局路径走，**实时动态避障**，控制机器人轮子转速
- 依赖：planner 的路径、雷达、TF、AMCL 定位
- 输出：机器人运动指令 `/cmd_vel`

##### 3.6.4.调度-bt_navigator（行为树导航）

- 作用：**Nav2 的大脑**，统一调度 planner、controller、recoveries
- 流程：
  1. 收到目标点 → 调用 planner 算路线
  2. 调用 controller 执行行走
  3. 遇到障碍 → 调用 recoveries 恢复
  4. 到达终点 → 停止
- 所有导航指令（`RViz` 点 2D Goal）都是发给它
  - <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260413213922561.png" alt="image-20260413213922561" style="zoom: 50%;" />

##### 3.6.5.保障-recoveries（异常恢复）

- 作用：机器人**卡住、迷路、撞墙**时，自动执行恢复动作
- 例如：原地旋转、后退、重新规划路径
- 调度：完全由 `bt_navigator` 控制

##### 3.6.6.管理（统一启停）- **lifecycle manager（生命周期管理器）**

- 作用：Nav2 所有节点都是**生命周期节点**（不能直接跑，需要激活）
- 工作：**按顺序启动、配置、激活** map_server/AMCL/planner/controller
- 核心：**防止组件乱启动、资源冲突**,   **一个管理器可以管理系统中所有的生命周期节点**
- 之前的问题：混用官方管理器 + 自己写的管理器 → 节点打架





#### 3.7.功能启动顺序(严格对照)

> 官方启动文件也是按照这个顺序，必须先有**里程计** → 再启动**定位** → 最后启动**导航**

1. **1.启动仿真**
   发布机器人模型、里程计、雷达 → 生成 `odom`（里程计） → `base_footprint`（机器人地盘） TF

2. **2.启动 map_server**
   加载地图，发布 `/map` 话题，**注意：如果加载已经建好的地图无需再进行建模，那么要考虑地图被覆盖的风险，启动项不要再重复加载**

3. **3.启动 AMCL + 生命周期管理器**
   激活定位节点 → 等待初始位姿

4. **4.RViz 给初始位姿**
   AMCL 开始工作 → 发布 `map` → `odom` → TF 链**完整**

5. **5.启动 planner + controller**
   规划器、控制器就绪，依赖地图、定位、TF

6. **6.RViz 点目标点**
   `bt_navigator` 收到指令 → 调用 `planner` 算全局路径

7. **7.机器人行走**
   `controller` 跟踪路径 + 实时避障 → 发布运动指令

8. **8.异常处理**
   卡住 → `recoveries` 自动恢复

9. **9.到达终点**
   `bt_navigator` 结束任务





#### 3.8.NAV2设置多个巡航点(插件专属`wpr_simulation2`仿真包)

> 地图上面设置巡航点，方便快捷，无需手动定位坐标

- 首先将路径规划相关的功能全部启动，具体顺序依据3.5

##### 3.8.1. 安装插件并编译

```bash
cd ~/wpr_ws/src
# 直接克隆过来
git clone https://gitee.com/s-robot/wp_map_tools.git
# 进入插件执行文件目录
cd ~/wpr_ws/src/wp_map_tools/scripts/
# 运行执行文件来安装插件
./install_for_humble.sh`

cd ~/wpr_ws/

colcon build
```

##### 3.8.2.导入地图数据

- 由于这个插件是 `wpr`仿真包专属插件，所以地图文件必须放入到 `wpr_simulation2` 的 `maps` 文件夹里面

<img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260414215712247.png" alt="image-20260414215712247" style="zoom:50%;" />

- 如果不想要从这里加载，可以从他的启动文件 **` ros2 launch wp_map_tools add_waypoint_sim.launch.py`**里面修改,地方如下图所示
  - <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260414221509897.png" alt="image-20260414221509897" style="zoom:50%;" />

##### 3.8.3.启动程序添加航点

- 启动命令：

  - ```bash
    ros2 launch wp_map_tools add_waypoint_sim.launch.py
    ```

- 进入 `rviz` 来选择多个航点
- <img src="C:\Users\23629\AppData\Roaming\Typora\typora-user-images\image-20260414220324667.png" alt="image-20260414220324667" style="zoom:67%;" />

- 最后依靠以下命令生成配置文件  **`waypoints.yaml`** 

  - ```bash
    ros2 run wp_map_tools wp_saver
    ```

  - 会生成到用户的主文件夹里面



##### 3.8.4.写节点依靠配置文件实现循环导航

- **配置文件( `waypoints.yaml`)：**

  - ```yaml
    Waypoints_Num: 3
    Waypoint_1:
      Type: Waypoint
      Name: 1
      Pos_x: -2.5771
      Pos_y: 2.26442
      Pos_z: 0
      Ori_x: 0
      Ori_y: 0
      Ori_z: 0
      Ori_w: 1
    Waypoint_2:
      Type: Waypoint
      Name: 2
      Pos_x: 2.38092
      Pos_y: -1.81623
      Pos_z: 0
      Ori_x: 0
      Ori_y: 0
      Ori_z: 0
      Ori_w: 1
    Waypoint_3:
      Type: Waypoint
      Name: 3
      Pos_x: -0.0376959
      Pos_y: -2.55577
      Pos_z: 0
      Ori_x: 0
      Ori_y: 0
      Ori_z: 0
      Ori_w: 1
    ```

    

- **节点代码：**

  - ```python
    #!/usr/bin/env python3
    import os
    import yaml
    import rclpy
    from ament_index_python.packages import get_package_share_directory
    from rclpy.action import ActionClient
    from rclpy.node import Node
    from nav2_msgs.action import NavigateToPose   # Nav2 官方单点导航动作
    #  ROS 2 的基础功能包,用于描述物体在空间中的位置、姿态、速度等几何信息, 包含：x, y, z 坐标 + 四元数朝向
    from geometry_msgs.msg import PoseStamped  
    
    class MultiPointCruise(Node):
        def __init__(self):
            super().__init__('multi_point_cruise_node')
            self.get_logger().info("✅ 多航点循环巡航节点启动（稳定版）")
    
            #  连接 Nav2 导航服务，单点导航       客户端 vs 服务端
            # NavigateToPose：动作类型（单点导航）  /navigate_to_pose：动作名称（Nav2 提供的服务端） 
            # action_client ： 客户端
            self.action_client ： 客户端 = ActionClient(self, NavigateToPose, '/navigate_to_pose')
            while not self.action_client.wait_for_server(timeout_sec=1.0):
                self.get_logger().info("等待 Nav2 导航服务...")
    
            # 读取 waypoints.yaml 加载航点
            pkg_share_dir = get_package_share_directory('wpr_user')  # 读取文件路径
            self.waypoint_file = os.path.join(pkg_share_dir, 'config', 'waypoints.yaml') # 完整拼接路径
            self.waypoints = self.load_waypoints()
            self.current_idx = 0
            self._destroyed = False
    
        # 加载航电函数，用来读取 yaml 并返回航点列表
        def load_waypoints(self):
            # 文件内容转成字典
            with open(self.waypoint_file, 'r', encoding='utf-8') as f:
                data = yaml.safe_load(f)
            waypoint_num = data['Waypoints_Num']
            waypoints = []
            for i in range(1, waypoint_num + 1):
                # 依次读取：Waypoint_1、Waypoint_2、Waypoint_3
                wp = data[f'Waypoint_{i}']
                # 创建一个导航目标点消息
                pose = PoseStamped()
                # 坐标系：map 地图坐标系
                pose.header.frame_id = "map"
                # 时间戳
                pose.header.stamp = self.get_clock().now().to_msg()
                # 赋值 x y 坐标 ，z 固定为 0（平面导航）
                pose.pose.position.x = float(wp['Pos_x'])
                pose.pose.position.y = float(wp['Pos_y'])
                pose.pose.position.z = 0.0
                # 朝向角度：四元数
                pose.pose.orientation.x = 0.0
                pose.pose.orientation.y = 0.0
                pose.pose.orientation.z = float(wp['Ori_z'])
                pose.pose.orientation.w = float(wp['Ori_w'])
                # 将点写入列表
                waypoints.append(pose)
                self.get_logger().info(f"✅ 加载航点 {i}")
            return waypoints
    
        def goto_next(self):
            if self._destroyed or not self.waypoints:
                return
    
            idx = self.current_idx
            pose = self.waypoints[idx]
            self.get_logger().info(f"🚗 前往航点 {idx+1}")
    
            # 创建导航动作目标
            goal = NavigateToPose.Goal()
            # 实例化导航动作，将pose赋值进去
            goal.pose = pose
    
            # 异步发送导航目标
            self.future = self.action_client.send_goal_async(goal)
            
            # 注册回调函数，发送完成 → 自动调用 on_goal_response
            self.future.add_done_callback(self.on_goal_response)
    
        def on_goal_response(self, future):
            # 得到发送导航目标的回复，会返回 accepted = True 和 accepted = False 
            goal_handle = future.result()
            # 如果返回不接受说明还没有到达导航点，直接拒绝
            if not goal_handle.accepted:
                self.get_logger().error("任务被拒绝")
                return
            # 异步等待机器人的动作句柄(goal_handle)返回数值，不会卡死程序
            self.result_future = goal_handle.get_result_async()
            self.result_future.add_done_callback(self.on_result)
    
        def on_result(self, future):
            if self._destroyed:
                return
            # 到达！自动下一个
            self.get_logger().info(f"✅ 已到达航点 {self.current_idx+1}")
            # 准备下一个航点，序号往后面推
            self.current_idx = (self.current_idx + 1) % len(self.waypoints)
            self.goto_next()
    
        def destroy_node(self):
            self._destroyed = True
            super().destroy_node()
    
    def main(args=None):
        rclpy.init(args=args)
        node = MultiPointCruise()
        try:
            node.goto_next()
            rclpy.spin(node)
        except KeyboardInterrupt:
            node.get_logger().info("🛑 安全退出")
        finally:
            node.destroy_node()
            rclpy.shutdown()
    
    if __name__ == '__main__':
        main()
    ```

    



### 4.建模与仿真

#### 4.1.机器人建模(URDF)

> URDF (**Unified Robot Description Format**) —————— **统一机器人描述格式**

- URDF是机器人操作系统（ROS）中用于描述机器人结构、外观和物理属性的标准文件格式，基于 `xml` 文件格式，后缀为 `.urdf`，通常在 **`Gazebo`** 或者 **`Rviz`** 中将 URDF 文件解析为图形化的机器人模型。

  - 如果非仿真环境，那么使用 URDF 结合 Rviz 直接显示感知的真实环境信息

  -  如果是仿真环境，那么需要使用 URDF 结合 **`Gazebo`** 搭建仿真环境，并结合 **`Rviz`**  显示感知的虚拟环境信息

- URDF 标签分类：

  - link 连杆标签 
  - joint 关节标签 
  - gazebo 集成gazebo需要使用的标签 
  - robot 根标签，类似于 launch文件中的launch标签

- 非常严格的层级结构：`robot` 为最高层级，`link` 和 `joint` 以及 `gazebo` 为同一层级，挂载在 `robot` 的下一个层级

  - ```xml
    <!-- 第 0 层：只有这一层是顶层 -->
    <robot name="我的机器人">
    
        <!-- 第 1 层：必须直接挂在 robot 下面 -->
        <link name="连杆A">
            <!-- 第 2 层：link 的子标签，不能直接写在 robot 下 -->
            <visual>
                <geometry>
                    <box size="1 1 1"/>
                </geometry>
            </visual>
        </link>
    
        <!-- 第 1 层：也是直接挂在 robot 下面 -->
        <joint name="关节1" type="fixed">
            <parent link="连杆A"/>
            <child link="连杆B"/>
        </joint>
    
        <!-- 第 1 层：gazebo 标签也挂在 robot 下 -->
        <gazebo reference="连杆A">
            <material>Gazebo/Blue</material>
        </gazebo>
    
    </robot
    ```

    

##### 4.1.1. URDF -- ` <link/>`

- 用于描述机器人某个部件(也即刚体部分)的外观和物理属性，比如: 机器人底座、轮子、激光雷达、摄像头...每一个部件都对应一个 link, 在 link 标签内，可以设计该部件的形状、尺寸、颜色、惯性矩阵、碰撞参数等一系列属性

- **主要的属性子标签(层级仅次于 `link`)**

  - `<visual>` (视觉属性)

    `<collision> `(碰撞属性)

    `<inertial> `(惯性属性)

- **中间层级的标签**：

  -  `<geometry>` ：用来联接几何形状标签
  - `<origin>` ： 用来设置坐标系与杆件几何中心的偏移量与倾斜弧度
    -  属性1: `xyz`=x偏移 y偏移 z偏移 
    - 属性2: `rpy`=x翻滚 y俯仰 z偏航 (单位是弧度)

  - `<material>` ：专门负责**外观颜色或纹理**，它只能存在于 `<visual>` 标签内部

- **底层标签**——**几何形状子标签**(必须在属性标签的下面通过挂载 `<geometry>` 标签来联通几何形状的标签)

  - `<box>` (长方体):
    - `size="长 宽 高"`：定义长方体的尺寸。
  - `<cylinder>` (圆柱体):
    - `radius="半径"`：定义圆柱体的半径。
    - `length="长度"`：定义圆柱体的高度。
  - `<sphere>` (球体):
    - `radius="半径"`：定义球体的半径。
  - `<mesh>` (网格):
    - `filename="文件路径"`：引用一个外部的 3D 模型文件（如 `.stl` 或 `.dae`），用于创建复杂、不规则的形状。

  - 举例：

  - ```xml
    <link name="my_link">
        <visual>
            <geometry>
                <cylinder radius="1" length="0.5"/> <!-- 形状在这里 -->
            </geometry>
        </visual>
    </link>
    ```

    

- **完整层级结构图：**

- ```
  <link name="示例连杆">
  │
  ├── 🟢 <visual> (视觉属性 - 第一层级)
  │   │
  │   ├── 🔵 <origin> (中间层级 - 位置/姿态)
  │   │   └── 属性: xyz (坐标), rpy (欧拉角)
  │   │
  │   ├── 🔵 <geometry> (中间层级 - 几何容器)
  │   │   │
  │   │   └── 🟠 几何形状 (底层标签 - 三选一)
  │   │       ├── <box size="x y z"/>
  │   │       ├── <cylinder radius="r" length="l"/>
  │   │       ├── <sphere radius="r"/>
  │   │       └── <mesh filename="路径"/>
  │   │
  │   └── 🔵 <material> (中间层级 - 材质/颜色)
  │       └── <color rgba="r g b a"/>
  │
  ├── 🟡 <collision> (碰撞属性 - 第一层级)
  │   │
  │   ├── 🔵 <origin> (中间层级 - 位置/姿态)
  │   │
  │   └── 🔵 <geometry> (中间层级 - 几何容器)
  │       └── 🟠 (同上，定义碰撞的具体形状)
  │
  └── 🔴 <inertial> (惯性属性 - 第一层级)
      │
      ├── 🔵 <origin> (中间层级 - 定义质心位置)
      │
      ├── <mass value="质量"/>
      │
      └── <inertia .../> (惯性矩阵)
  ```

  

##### **4.1.2. URDF --** `<joint/>`

- **用于描述机器人两个部件（Link）之间的连接关系**。
  - 它定义了两个连杆之间是如何相对运动的（例如：是固定死的、旋转的、还是滑动的）。
  - 每一个 `<joint>` 必须包含两个核心引用：**parent（父连杆）** 和 **child（子连杆）**。
- **主要的属性子标签（层级仅次于 joint）**
  - `<parent>`：定义父级连杆（参考系）。
  - `<child>`：定义子级连杆（被移动的部件）。
  - `<calibration>`：（可选）关节的参考位置校准。
  - `<limit>`：（重要）定义运动的物理限制（如角度范围、速度、力矩）。
  - `<dynamics>`：（可选）定义物理属性，如摩擦和阻尼。
  - `<safety_controller>`：（可选）安全控制器设置。
  - `<mimic>`：（可选）模仿其他关节的运动。

- **中间层级的标签**：
  -  `<geometry>` ：用来联接几何形状标签
  - `<axis>`：专门负责定义**运动轴向**（仅在非固定关节中有效）。
    - 属性：`xyz` = "x y z"：定义关节旋转或移动的轴向量（通常是 `1 0 0` 代表绕X轴，`0 0 1` 代表绕Z轴）

- **底层标签**——关节类型（通过 type 属性定义，必须在 joint 标签的开始处定义）：
  - `<joint ... type="类型">`
  - `revolute` (旋转关节)：
    - 描述：绕轴旋转，有角度限制（由 `<limit>` 定义）。
    - 示例：`<joint name="joint1" type="revolute">`
  - `continuous` (连续旋转关节)：
    - 描述：绕轴无限旋转（如轮子），忽略 `<limit>` 中的角度限制。
    - 示例：`<joint name="wheel_joint" type="continuous">`
  - `prismatic` (滑动关节)：
    - 描述：沿轴滑动，有距离限制。
    - 示例：`<joint name="piston" type="prismatic">`
  - `fixed` (固定关节)：
    - 描述：不可运动，将两个连杆刚性连接。
    - 示例：`<joint name="base_link" type="fixed">`
  - `floating` (浮动关节)：
    - 描述：允许在6个自由度上自由运动（x, y, z 移动 + 绕轴旋转）。
  - `planar` (平面关节)：
    - 描述：在垂直于轴的平面上移动，并绕轴旋转。



- 完整层级结构图：

- ```
  <joint name="示例关节" type="类型(revolute/fixed/...)">
  │
  ├── 🟢 <parent> (主要属性 - 父连杆)
  │   └── 属性: link="父连杆名称"
  │
  ├── 🟡 <child> (主要属性 - 子连杆)
  │   └── 属性: link="子连杆名称"
  │
  ├── 🔵 <origin> (中间层级 - 位置/姿态)
  │   └── 属性: xyz="x y z", rpy="r p y"
  │       (定义关节轴心相对于父连杆的位置)
  │
  ├── 🔵 <axis> (中间层级 - 运动轴向)
  │   └── 属性: xyz="1 0 0"
  │       (定义关节是绕哪个轴转动或移动，默认是X轴)
  │
  ├── 🔴 <limit> (主要属性 - 运动限制)
  │   ├── 属性: lower="下限" (弧度或米)
  │   ├── 属性: upper="上限" (弧度或米)
  │   ├── 属性: effort="最大力/力矩"
  │   └── 属性: velocity="最大速度"
  │
  └── 🟣 <dynamics> (主要属性 - 物理特性)
      ├── 属性: damping="阻尼系数"
      └── 属性: friction="摩擦系数"
  ```

  





































