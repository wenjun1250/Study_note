## Python使用

> 特定：严格缩进原则而不是 C 的 {}
>
> 一般不需要 **`;`** 结尾

### 1.基础使用

#### 1.1.变量

- **变量赋值**：直接写 变量名 = 值，无需类型声明。

- **基本类型**：

  - 整数：**`a`** = 10
  - 浮点数：**`b`** = 3.14
  - 字符串：**`s`** = "hello"
  - 布尔值：**`flag`** = True

- **类型转换**：用 int()、float()、str() 等函数转换类型。

  - **与 C 的对比**：

    - **C**：**`int a = 10`**;

    - **Python**：**`a = 10`**

#### 1.2.运算符

- **算术**：+ ,  - ,  * ,  / ,  //（整除）,  % ,  **（幂, 区别于 C的 ^）。
  - / : 返回浮点数
  - // : 返回整数
- **比较**：==, !=, >, <, >=, <=。
- **逻辑**：and, or, not（C 用 &&, ||, !）。
- **位运算**：&, |, ^, ~, <<, >>（与 C 相同）。

#### 1.3.条件语句

- **`if`** 语句

```python
if a > 0:
    printf("正数")
elif a == 0:
    printf("零")
else:
    printf("负数")
```

- **`while`** 语句

```python
while a < 5:
    a += 1
    if a == 1:
        continue # 跳过此次循环
    if a >= 3:  
        break;   # 对于 while语句可以通过 break和continue 这两个关键字来跳出循环或者跳过此次循环
```

- **`for`** 语句

```python
for i in range(5): 
    print(i)  # 遍历 0 到 4
    
words = ['cat', 'window', 'defenestrate']   # 创建一个列表
for w in words[:]:   # 使用切片创建副本，这里的副本只是为了做判断，使用过后自动销毁  [:]为切片语法
    if len(w) > 6:
        words.insert(0, w) # 这里运行完的结果就是遍历到长度大于6的时候即 defenestrate ，将这个元素插到开头的位置
        				   # 然后此时列表就变成了['defenestrate', 'cat', 'window', 'defenestrate']
# 这里如果判断的时候不用切片会造成列表混乱，程序会乱，可能造成无限循环

```

#### 1.4.函数

- 定义函数

```python
def add(a, b):
    return a + b

```

- **可变参数**：**`*args`**（接收多个参数为元组），**`**kwargs`**（接收键值对为字典）。
  - **`*`**: 解包列表或元组
  - **`**`** :  解包字典（关键字参数）

```python
def my_function(*args):
    print("接收到的参数是一个元组：", args)
    
def my_function(**kwargs):
    print("接收到的参数是一个字典：", kwargs)

```

- 关键字: **`global`** ，如果外部定义了全局变量，函数内部需要修改它，那么就需要声明全局变量

```python
# 全局变量
frame_count = 0
fps = 0.0

def calculate_fps(timer):
    global frame_count,fps
    fps = frame_count      # 帧率 = 过去1s的帧数
    frame_count = 0        # 重置计数
    
```



#### 1.5.指针

- 不会显示的暴露指针，指针通常隐藏在赋值下面

```python
a = 10
b = a    # 这里实际上 b 就指向了 a的地址，当10变为20，那么b也会变为20

```

#### 1.6.数据结构

##### 1.6.1.列表

- **例子**: **`lst = [1,2,3]`**

- **操作**:
  - 索引：**`lst[0]`**（从 0 开始）
  - 切片：**`lst[1:3]`**（取第 1 到第 2 个元素） lst[:] ( 拷贝整个列表)
  - 添加：**`lst.append(4)`**  在末尾添加一个元素
  - 添加：**`lst.insert(index, value)`**：在指定位置插入元素
  - 删除：**`lst.remove(2)`**  删除匹配的值

##### 1.6.2.元组

- **例子**：**`tup = (1,2,3)`**

- 特点: 不可变(类似于 c 语言的 const)，访问方式类似列表

##### 1.6.3.集合

- **例子**：**`s = {1,2,3}`**
- **操作**:  **`s.add(4)`**(添加一个元素，这里是添加4这个值), **`s.remove(2) `**(删除指定的值) . 支持交并差运算，即并集，交集，超集等等
- **特点**: 
  - 无序性：元素没有顺序，不能通过索引访问
  - 唯一性：自动去重，不允许重复元素
  - 可变性：可以添加或删除元素（但元素本身必须是可变类型）

##### 1.6.4.字典

- **例子**：**`d = {"name": "Alice", "age": 25}`**
- **操作**：
  - 访问：**`d["name"]`**
  - 添加：**`d["key"] = "value"`**
  - 删除：**`del d["key"]`**
- **特点**: 一个键对应一个值，可以用键来索引，
  - 键（key）必须是**不可变类型** （如字符串、数字、元组）
  - 值（value）可以是任意类型

##### 1.6.5.字符串

- **例子**:  **` s = "Hello,world"`**  ， 单双引号作用相同

  - 三引号可以进行多行字符串

  - ```python
    s = '''line1
    line2'''
    ```

- 转义字符:  用 \ 表示，如 \n（换行）、\t（制表符）。 r 表示忽略转义字符

- 

- 

#### 1.7.类和对象

```python
class Dog:
    def __init__(self, name):  #  Python 中的构造函数（初始化方法）。 当创建类的新实例时，会自动调用它
        self.name = name 	   # 实例属性  self 表示实例自身，用来绑定属性。name 是传入的参数。
    def bark(self):  		   # 定义一个用户函数，即自定义函数
        print(f"{self.name} says woof!") # 开头的 f 表示这是一个 f-string ，它允许你在字符串中嵌入表达式（如变量、运算等）。
        
# 创建两个不同的狗实例
dog1 = Dog("Buddy")
dog2 = Dog("Max")

# 调用它们的 bark 方法
dog1.bark()  # 输出：Buddy says woof!
dog2.bark()  # 输出：Max says woof!
```

- **`with`** 语句：



#### 1.8.继承

```python
class Puppy(Dog):
    def bark(self):  # 重写方法
        print(f"{self.name} says yip!")
        
```

#### 1.9.模块化的测试代码

- **`if __name__ == "__main__": `** 语句用于区分 Python 脚本是被直接运行还是被作为模块导入到其他脚本中。

  - 如果我自行编写了一个 **`my_module.py`** 文件，里面包含了你自己写的模块，你是可以单独测试的

    ```python
    def add(a, b):
        return a + b
    
    # 测试代码或主程序逻辑
    if __name__ == "__main__":
        print("Running my_module.py directly")
        result = add(2, 3)
        print(f"Result of add(2, 3): {result}")
    ```

  - 这里的 **` print("Running my_module.py directly")`** 代码以及后面的代码如果被导入到其他名称的文件则不会执行，比如 **`main.py`** 文件

  - **`main`** 换成文件作为模块导入的文件名称，然后执行时就不会执行测试代码，无需做删减
  
  - 这里如果直接执行测试文件一样会执行配置文件的

### 2.ROS2_Python代码拆解

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import rclpy
from rclpy.node import Node
from std_msgs.msg import String
from std_msgs.msg import Int32
import time

class PublisherNode(Node):
    """发布者节点：定期发布消息到话题"""
    
    def __init__(self): 						 # 这是实例方法
        super().__init__('publisher_node')       # 调用父类代码
        
        # 创建发布者，发布到 'test_topic' 话题
        # String类型消息
        self.string_publisher = self.create_publisher(  	# 利用self来进行实例属性赋值
            String, 
            'test_topic', 
            10  # 队列大小
        )
        
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
    
    def timer_callback(self):
        """定时器回调函数，定期发布消息"""
        # 发布字符串消息
        string_msg = String()
        string_msg.data = f'Hello ROS2! 消息编号: {self.counter}'
        self.string_publisher.publish(string_msg)
        self.get_logger().info(f'发布字符串: "{string_msg.data}"')
        
        # 发布整数消息
        int_msg = Int32()
        int_msg.data = self.counter
        self.int_publisher.publish(int_msg)
        self.get_logger().info(f'发布整数: {int_msg.data}')
        
        self.counter += 1

def main(args=None):
    """主函数"""
    rclpy.init(args=args)
    
    # 创建节点
    publisher_node = PublisherNode()
    
    try:
        # 运行节点
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































