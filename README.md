# 在PX4中添加MID360、D435i进行仿真
记录搭建仿真环境的过程
我使用的环境是：
- Ubuntu20.04
- ROS Noetic

# MID360+PX4仿真
在gazebo + px4 + mid360进行仿真
## 1.安装Livox-SDK2
参考repo：`https://github.com/Livox-SDK/Livox-SDK2.git` 进行安装

**快速安装**
```shell
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd ./Livox-SDK2/
mkdir build
cd build
cmake .. && make -j
sudo make install
```


## 2.编译livox_ros_driver2
mid360需要使用livox_ros_driver2进行驱动，livox_ros_driver不兼容  
参考repo:`https://github.com/Livox-SDK/livox_ros_driver2.git`进行安装  
**快速安装**
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

## 3.安装livox_laser_simulation
构建gazebo仿真插件
```shell
# 也是在工作目录中创建功能包，如果之前创建过，则直接进入即可
cd ~/catkin_ws/src
git clone https://github.com/qiurongcan/Mid360_imu_sim.git
cd ..
catkin_make
# 最好在~/.bashrc文件中全局激活工作空间路径
source devel/setup.bash
# 验证
roslaunch livox_laser_simulation livox_simulation.launch
# 或者使用带有IMU版本的
roslaunch livox_laser_simulation mid360_IMU_platform.launch
```
**注意：** 这里我只是用了mid360的扫描文件，其他型号的文件被我删除了，需要部署其他雷达则参考原仓库即可
`https://github.com/Livox-SDK/livox_laser_simulation.git`

**注意2：** 如果使用IMU版本的这里的雷达输出只有以下两个话题，标准版只有一个话题：  
- /scan sensor_msgs/PointCloud
- /lidar/imu
如果需要使用Fast_lio,这个点云格式是没有办法直接使用的，需要进行格式转化，参考repo: `https://github.com/qiurongcan/Mid360_imu_sim.git` 中的方法

【但是这个不重要hhh】主要用的是他的插件，MID360在后面进行重新构建了

## 4.组装MID360+px4无人机
**[默认安装好PX4环境]**
v1.13版本之前的px4和之后的文件布局略有不同

**v1.13.0版本包括之前**
需要将 `Mid360` 文件夹复制到路径 `PX-Autopilot/Tools/sitl_gazebo/models/` 下  

**v1.14.0之后的版本**
需要将 `Mid360` 文件夹复制到路径 `PX-Autopilot/Tools/simulation/gazebo-classic/sitl_gazebo-classic/models/` 下


**注意！！！！！！！！**  
**注意！！！！！！！！**  
**注意！！！！！！！！**  
需要修改 `Mid360/Mid360.sdf` 文件中读取csv的路径，**见71行**，一定要修改为你电脑的路径，这也才能成功执行
```xml
70          <downsample>1</downsample>
71          <csv_file_name>/home/amov/PX4-Autopilot/Tools/sitl_gazebo/models/Mid360/livox_mid40/scan_mode/mid360.csv</csv_file_name>
72          <ros_topic>/scan</ros_topic>
```

这个有**iris**无人机，所以无需重复添加  
之后再这里创建一个文件夹，如下
1. 首先创建一个 `iris_mid360文` 件夹
2. 创建两个文件`model.config` 和 `iris_mid360.sdf`  
- **model.config**
```xml
<?xml version="1.0"?>
<model>
  <name>3DR Iris with mid360</name>
  <version>1.0</version>
  <sdf version='1.5'>iris_mid360.sdf</sdf>

  <author>
   <name>qiurongcan</name>
   <email>qrc18760035045@163.com</email>
  </author>

  <description>
    mid360
  </description>
</model>
```

- **iris_mid360.sdf**
```xml
<?xml version="1.0" ?>
<sdf version="1.5">
  <model name='iris_mid360'>
    <include>
      <uri>model://iris</uri>
    </include>
    <include>
      <uri>model://Mid360</uri>
      <pose>0 0 0.05 0 0 0</pose>
    </include>
    <joint name="livox_joint" type="fixed">
      <child>Mid360::livox_base</child>
      <parent>iris::base_link</parent>
      <axis>
        <xyz>0 0 1</xyz>
        <limit>
          <upper>0</upper>
          <lower>0</lower>
        </limit>
      </axis>
    </joint>
  </model>
</sdf>

```
如果想偷懒直接复制 `iris_mid360` 文件夹到这个目录即可  

最后在`PX4-AutoPilot/launch/mavros_posix_sitl.launch` 文件中修改无人机的型号为新替换的这个即可
## 5.验证
```shell
# terminal 1 运行后弹出一个带有mid360的无人机模型
roslaunch px4 mavros_posix_sitl.launch

# terminal 2 查看话题并查看输出
rostopic list | grep /scan
rostopic echo /scan
```

# D435i+PX4仿真
在gazebo + px4 + D435i进行仿真
## 1.安装realsense仿真驱动

## 2.拷贝仿真插件

## 3.组装D435i+px4无人机

## 4.验证


# PX4+MID360+D435i无人机



