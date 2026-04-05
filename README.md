<!--
 * @@Descripttion: lanhuazhiyue于2026年3月28日更新了部分说明
 * @@version: 
 * @@encoding: utf-8
-->

# Livox MID360 + IMU仿真 可用于Fast LIO
A package to provide plug-in for [Livox Series LiDAR](https://www.livoxtech.com).

## 发布点云类型
```shell
livox_ros_driver2/CustomMsg
```
## Environment
我自己电脑测试的环境如下
- ROS(=Noetic)
- Gazebo11
- Ubuntu(20.04)

## 安装Livox_SDK2依赖
**快速安装**
```shell
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd ./Livox-SDK2/
mkdir build
cd build
cmake .. && make -j
sudo make install
```

## 安装livox_ros_driver2依赖
```shell
# 在主目录下创建一个工作空间
mkdir -p catkin_ws/src
cd catkin/src
git clone https://github.com/Livox-SDK/livox_ros_driver2.git ws_livox/src/livox_ros_driver2
cd livox_ros_driver2
# 我是用的ROS1，如果使用ROS2参考原仓库安装
./build.sh ROS1
# 激活全局路径
source ../../devel/setup.bash
# 验证，没报错的话基本上是没问题
roslaunch livox_ros_driver2 [launch file]
```

## Usage

**克隆仓库并编译, 注意这个功能包和`livox_ros_driver2`放在一个工作空间下，如果不在一个空间编译会报错**
```shell
cd ~/ws_livox/src
git clone git@github.com:lanhuazhiyue/livox_laser_simulation_Mid360.git
cd ..
catkin_make
```

<span style="color: red;">
💡 **注意：**
编译成功会在 ws_livox/devel/lib文件夹下生成*liblivox_laser_simulation.so*文件，gazebo中能出现点云都依赖于此链接库
</span>

> 相比于官方仿真库，原作者主要1. 在meshes文件夹加入了*mid360.dae*和*mid360_rules.dae*两个文件分别用于外形显示和碰撞检测； 2. 修改了urdf文件夹中*livox_mid360.xacro*文件的内容； 新增了urdf文件夹中*mid360_IMU_platform.xacro*文件; 3. 修改了src文件夹下*livox_points_plugin.cpp*文件内的部分代码，让点云的输出类型从PointCloud -> PointCloud2 -> livox_ros_drive2.CustomMsg，从而可以供给fast-lio直接使用
>

> 相比于原作者，根据github上面的issue将*src/livox_points_plugin.cpp*中关于minDist的定义强行赋值为0了，避免在碰撞检测阶段就“扭曲”近距离的射线，最后的输出仍然按用户设定值来输出。

```cpp
// minDist = rangeElem->Get<double>("min");
minDist = 0.0
```


**运行代码**
```shell
roslaunch livox_laser_simulation mid360_IMU_platform.launch
```
在`mid360_IMU_platform.launch`文件中，可以注释最后的rviz这几行，不显示rviz,也可以取消注释显示rviz  
可以手动在gazebo中添加物体【长方形、圆柱...】,达到一个更好的演示效果
```xml
  <node pkg="robot_state_publisher" type="robot_state_publisher" name="robot_state_publisher">
    <param name="publish_frequency" type="double" value="30.0" />
  </node>

  <!-- RViz -->
  <!-- <arg name="rviz" default="true"/>
  <node name="rviz" pkg="rviz" type="rviz" args="-d $(find livox_laser_simulation)/rviz/livox_simulation.rviz"/> -->

```
**查看话题**
此时会有两个话题
```shell 
# 打开一个新的终端
rostopic list
# 输出如下 >>>>>
/scan livox_ros_driver2/CustomMsg
/livox/imu
```
**这个雷达的数据类型Fast_LIO是可以直接使用的**


## Parameters(only for display , and example by avia)

- laser_min_range: 0.1  // min detection range
- laser_max_range: 200.0  // max detection range
- horizontal_fov: 70.4   //°
- vertical_fov: 77.2    //°
- ros_topic: scan // topic in ros
- samples: 24000  // number of points in each scan loop
- downsample: 1 // we can increment this para to decrease the consumption

