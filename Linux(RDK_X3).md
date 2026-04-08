### RDK_X3

### 1.常用命令

#### 1.1.设置

- **`sudo srpi-config`**: 进入srpi-config工具，可以用来连接wifi



***



### 2.GPIO操作

<img src="F:\截屏的图片\Snipaste_2025-07-04_15-07-26.png" alt="Snipaste_2025-07-04_15-07-26" style="zoom:67%;" />

#### 2.1.GPIO的文件操作方式(shell)

##### 2.1.1.文件位置

- 控制 GPIO 的目录位于/sys/class/gpio
  - 里面包含三个文件夹;  **`export`** ，**`unexport`** ,**`gpiochip0`** 
  - **`export`**:  此文件用于通知系统需要导出控制的GPIO引脚编号，即内核空间导出到用户空间，然后才能通过命令控制
  - **`unexport`** :  用于通知系统取消导出，即将你已经导出的GPIO引脚返回内核空间
  - **`gpiochip0`** :  目录保存GPIO的寄存器信息，包括每个寄存器控制引脚的起始编号base,寄存器名称，引脚总数

##### 2.1.2.操作方法

- 引脚导出编号基于CVM(功能名)，比如GPIO105的编号就是105，而不是物理编号(37)或者BCM编码(26)

  - **`sudo echo 105 > /sys/class/gpio/export`**: 将GPIO105从内核空间导出

    - 即对 **`export`** 写入一个105的命令，此时就会进行导出操作

  - **`sudo echo 105 > /sys/class/gpio/unexport`**: 取消GPIO口的导出

    - 将CVM的号码写进 **`unexport`**

  - **`cd  /sys/class/gpio/gpio105`**: 切换到该GPIO口进行操作，同时查看相关信息(使用cat 命令)

    - **`active_low`**: 控制是否将低电平视为"激活"状态，值一般为 **`0`** 或 **`1`** ， **`0`** 为默认值，高电平为激活， **`1`** 代表低电平被激活
    - **`device`**: 指向该gpio口的对应驱动信息(通常是一个符号链接)
    - **`direction`**： 设置和查看GPIO的方向，可以写入 **`in`** 或者 **`out`** ，读取时会显示当前的方向
    - **`edge`** : 设置触发中断的的边沿类型(用于输入模式下触发中断)，可以写入 **`none`** **`rising`** **`falling`** **`both`** 
    - **`value`**: 读写GPIO的当前电平值，可以写入  **`0`** 或 **`1`** (仅当方向是out时)，读取时返回当前的引脚电平(方向为 **`in`** 时)
    - **`power`** : 电源相关，无需管理
    - **`subsystem`**: 指向GPIO子系统的符号链接

  - **`sudo echo out > direction`**: 设置为输出模式，如果想要改为输入模式，将 **`out`** 换为 **`in`** 即可

    - 如果不在当前gpio里面操作则需要完整的文件路径，比如 **`/sys/class/gpio/gpio105/direction`** 

  - **`sudo echo 1 > value`**: 设置为高电平，1 换成 0，则设置为低电平(针对输出模式),

    - 同样如果不在当前文件夹需要指定完整的文件路径，后面的命令同理

    

