

# 树莓派SLAM小车操作指南

> 张子豪  同济大学2018级交通运输工程学院 研究生

[TOC]

# 源代码

> 源代码都在树莓派操作系统中

python源代码：

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/script

C语言源代码

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/src

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/launch



## 安装虚拟机软件VMware

网盘链接：[点我](https://pan.baidu.com/s/1pbDXpgpNQTdTRzzKNWfE1A)    

安装指南：[点我](https://blog.csdn.net/qq_31362105/article/details/80706096)   

许可证密钥：`CG54H-D8D0H-H8DHY-C6X7X-N2KG6 ` 



## 下载虚拟机系统文件

打开Vmware，选择`打开虚拟机`

找到`Ubuntu 64 位.vmx`文件



显示该虚拟机似乎正在使用中......点击`获取所有权`

网络适配器选桥接模式，勾选复制物理网络连接状态

开启虚拟机

我已复制该虚拟机

无法连接虚拟设备：是

确定 确定



## 打开虚拟机Ubuntu系统

用户名CLB，密码123456

查看-自动调整大小-自动适应客户机 自动适应窗口

安装vmware-tools

快捷键：

ctrl + shift + t  新的命令行选项卡

ctrl + alt + t 新的命令行窗口



## 烧录树莓派镜像

下载镜像烧录工具 Etcher或win32img

下载镜像文件，区分3B和3B+

烧录到SD卡中



# 小车开机
将SD卡插到小车上，开机
电压应在12V以上，显示屏显示
等屏幕加载完毕

用飞鼠，在显示屏上设置wifi，电脑和树莓派连到同一个wifi
用ifconfig查看ip地址
或者用局域网ip扫描器查看树莓派ip地址

找到树莓派的ip地址，比如我的是192.168.200.14

在虚拟机中输入

```shell
sudo gedit /etc/hosts
```

> 这个文件显示了域名和网址之间的映射情况。在这个文件里出现的网址，访问的时候不需要使用DNS协议，而是直接读取域名。

把robot前面的ip地址设为树莓派的ip地址

ctrl + s 保存，关闭

在虚拟机中输入

```shel
ssh clbrobot@robot
```

或者在putty里输入

```shell
clbrobot
123456
```





登录之后，输入

```shell
clbrobot@robot~$ roslaunch clbrobot bringup.launch
```

启动主节点



```shell
CLB@CLB sudo gedit ~/.bashrc
```

最后一行，替换成树莓派的ip地址，使树莓派指向主节点

```shell
CLB@CLB source ~/.bashrc
```

让修改生效



```shell
CLB@CLB rosrun rviz rviz
```

启动rviz调试窗口



# 操作机器人



```shell
CLB@CLB ssh clbrobot@robot
```



```shell
clbrobot@robot sudo nano ~/.bashrc
```

确保最后部分是tank和rplidar

> rplidar指的是思岚公司生产的激光雷达



# 快速控制机器人

```shell
clbrobot@robot rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```



# 关机

```shell
clbrobot@robot sudo halt
```

创乐博logo出现在显示屏上，此时就可以关机了



# 重新启动机器人

启动虚拟机

```shell
CLB@CLB ssh clbrobot@robot
```

启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot bringup.launch
```



```shell
clbrobot@robot roscd clbrobot/
```

进入工程目录

```shell
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot$ cd param

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param$ ls
ekf  imu  navigation
```



## 校准IMU

在上一步基础上

```shell
cd imu

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param$ cd imu

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ ls
imu_calib.yaml

clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ sudo nano imu_calib.yaml 
```



```shell
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ rostopic echo /imu/data
打开实时陀螺仪
```



```shell
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/param/imu$ rosrun imu_calib do_calib
将不同面依次朝上，回车进行校准
```



## 角速度校正



```shell
clbrobot@robot $ rosrun rikirobot_nav calibrate_angular.py
CLB@CLB:~$ rosrun rqt_reconfigure rqt_reconfigure 
```

打开imu校准界面

点start_test进行360度转向测试，假如转了365度，就将365/360=1.014填入odom_angular_scale_coreection，重新测试

关掉所有命令行，包括主节点

```shell
clbrobot@robot:~$ roscd clbrobot/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot$ cd launch/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/launch$ sudo nano bringup.launch
```

找到下面这行

```xml
 <param name="angular_scale" value="0.986" />
```

将测好的数值传入value，保存退出



## 线速度校正

ssh到树莓派，启动主节点

```shell
clbrobot@robot~$ rosrun rikirobot_nav calibrate_linear.py
CLB@CLB:~$ rosrun rqt_reconfigure rqt_reconfigure 
```

选择`calibrate linear`

修改odom_linear_scale_correction

```shell
clbrobot@robot:~$ roscd clbrobot/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot$ cd launch/
clbrobot@robot:~/catkin_ws/src/clbrobot_project/clbrobot/launch$ sudo nano bringup.launch
```

打开xml配置文件

修改value值

```xml
 <param name="linear_scale" type="double" value="1.045" />
```

# 创建地图

ssh到树莓派，开启主节点

```shell
clbrobot@robot:~$ roslaunch clbrobot lidar_slam.launch
```

打开rviz调试窗口

```shell
CLB@CLB:~$ rosrun rviz rviz
```

![1551167096948](D:\2018秋季学期\树莓派slam小车\1551167096948.png)



open config

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz



键盘控制机器人

```shell
clbrobot@robot rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

speed调至0.3，turn调至0.5

## 保存地图



```shell
clbrobot@robot:~$ roscd clbrobot
cd maps
ls
./map.sh
ls -l 即可看到更新后的hous.pgm和house.yaml文件
```



# 自动导航



ssh到树莓派，启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot navigate.launch
```



```shell
CLB@CLB rosrun rviz rviz
```

载入

/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/navigate.rviz

# 鼠标构件地图



ssh到树莓派，启动底盘节点

启动slam节点：

```shell
clbrobot@robot:~$ roslaunch clbrobot lidar_slam.launch

CLB@CLB rosrun rviz rviz
```

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

直接用2D Nav Goal选定目标



# 选择区域构建地图

ssh到树莓派，启动底盘节点

启动auto_slam节点：

```shell
clbrobot@robot:~$ roslaunch clbrobot auto_slam.launch

CLB@CLB rosrun rviz rviz
```

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

用工具栏中的Publish Point勾勒一个封闭区域



# 使用hector算法构建地图

ssh到树莓派、启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot hector_slam.launch

CLB@CLB rosrun rviz rviz
```

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

hector算法适用于空旷地带，在狭小空间会有重影

![1551182940839](D:\2018秋季学期\树莓派slam小车\1551182940839.png)

# 多点导航

ssh到树莓派，启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot navigate_multi.launch

CLB@CLB rosrun rviz rviz
```

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/multi_goal.rviz

用publish point工具设定目标点

多点导航源代码：

/home/rikirobot/catkin_ws/src/clbrobot_project/clbrobot/script



# 动态PID参数调节

ssh到树莓派，启动底盘节点

```shell
clbrobot@robot rosrun riki_pid pid_configure

CLB@CLB rosrun rqt_reconfigure rqt_reconfigure
```

源代码：

/home/rikirobot/catkin_ws/src/clbrobot_project/riki_pid

cfg：默认载入参数

src：配置文件

修改之后要重新编译才能生效



# 摄像头寻线

ssh到树莓派，启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot camera.launch

CLB@CLB roslaunch riki_line_follower riki_line.launch
```

摄像头打开，识别红色线

源代码

/home/rikirobot/catkin_ws/src/clbrobot_project/riki_line_follower



# 雷达跟随

ssh到树莓派，启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot camera.launch

clbrobot@robot roslaunch riki_lidar_follower laser_follower.launch
```



源代码

/home/rikirobot/catkin_ws/src/clbrobot_project/riki_lidar_follower



# karto算法构建地图

ssh到树莓派，启动底盘节点

```shell
clbrobot@robot roslaunch clbrobot karto_slam.launch

CLB@CLB rosrun rviz rviz
```

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/slam.rviz

虚拟机端打开键盘控制

```shell
CLB@CLB rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

默认的线速度为0.5，角速度为1.0

因为karto算法较复杂，调整线速度0.3左右，角速度0.5即可

# 安卓手机app与图像监控

ssh到树莓派 ，开启底盘节点

```shell
clbrobot@robot roslaunch clbrobot appcamera.launch
```

手机上安装.apk文件，新建机器人，将localhost改为机器人的ip地址，默认端口11311



# opencv图像处理

源代码：

/home/rikirobot/catkin_ws/src/opencv_apps

查看源码：

```shell
CLB@CLB roscd opencv_apps
cd src
cd nodelet
ls
```



ssh到树莓派

```shell
clbrobot@robot roslaunch clbrobot camera.launch



CLB@CLB roslaunch opencv_apps edge_detection.launch #边缘提取

CLB@CLB roslaunch opencv_apps hough_lines.launch #直线检测
CLB@CLB roslaunch opencv_apps find_contours.launch #检测轮廓
CLB@CLB roslaunch opencv_apps convex_hull.launch #凸包检测
CLB@CLB roslaunch opencv_apps general_contours.launch #椭圆形轮廓检测
CLB@CLB roslaunch opencv_apps people_detect.launch #行人检测
CLB@CLB roslaunch opencv_apps goodfeature_track.launch #尖角点检测
CLB@CLB roslaunch opencv_apps camshift.launch #目标跟踪
CLB@CLB roslaunch opencv_apps fback_flow.launch #物体移动检测
CLB@CLB roslaunch opencv_apps lk_flow.launch #物体移动检测2
CLB@CLB roslaunch opencv_apps simple_flow.launch #物体移动检测3
CLB@CLB roslaunch opencv_apps phase_corr.launch #相位偏移
CLB@CLB roslaunch opencv_apps segment_objects.launch #物体分割
CLB@CLB roslaunch opencv_apps rgb_color_filter.launch #转黑白
CLB@CLB roslaunch opencv_apps hls_color_filter.launch #转黑白
CLB@CLB roslaunch opencv_apps hsv_color_filter.launch #转黑白
```



# 安卓手机app构建地图

手机上下载`Make Map.apk`

ssh到树莓派，开启底盘节点

```shell
clbrobot@robot roslaunch clbrobot lidar_slam.launch

clbrobot@robot roslaunch clbrobot camera.launch
```

打开手机app，设置localhost为机器人ip地址，点connect

# 安卓手机app导航

手机上下载`Map Nav`

ssh到树莓派，开启主节点

```shell
clbrobot@robot roslaunch clbrobot navigate.launch

clbrobot@robot roslaunch clbrobot camera.launch

CLB@CLB rosrun rviz rviz
```

载入/home/riki/catkon_ws/src/rikirobot_project/rikirobot/rviz/navigate.rviz

手机打开APP，设置localhost为机器人ip地址，connect

可以指定机器人的位置set pose，也可以进行导航set goal

设置的时候需要长按，并且在屏幕上拖动指明方向

app上只显示最近的地图，随着位置的移动，地图会跟着更新

同时，小车会自动避开途径的障碍，非常智能。



# vim编辑器常用操作

i 进入编辑模式

esc：退出编辑模式

:w 保存

shift + z + z保存退出

:wq 保存并退出

:q! 强制退出



# 树莓派常用命令



```shell
clbrobot@robot:~$ rostopic echo /scan #如果都是0说明激光雷达没有输出
clbrobot@robot:~$ ls -l /dev/ttyU*
crw-rw---- 1 root dialout 188, 0 2月  28 20:27 /dev/ttyUSB0
crw-rw---- 1 root dialout 188, 1 2月  28 18:44 /dev/ttyUSB1
查看USB设备，一个是底盘，一个是激光雷达
clbrobot@robot:~$ ls -l /dev/riki* #查看软映射

clbrobot@robot:~$ lsusb
Bus 001 Device 006: ID 0bda:5825 Realtek Semiconductor Corp. 
Bus 001 Device 005: ID 10c4:ea60 Cygnal Integrated Products, Inc. CP210x UART Bridge / myAVR mySmartUSB light
Bus 001 Device 004: ID 1a86:7523 QinHeng Electronics HL-340 USB-Serial adapter
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp. SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

```

