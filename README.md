

# gazebo仿真环境下的rtabmap建图与导航





# 介绍

本项目是基于机器人阿杰的ros教程下的rtabmap建图与导航教程，各位小伙伴在学习完机器人阿杰的hector和gmapping 2d栅格建图以及movebase导航后，可以继续学习本项目的rtabmap 3d点云建图，相比于传统的 **Gmapping** 或 **Hector SLAM** 这种 2D 栅格建图方案，**RTAB-Map (Real-Time Appearance-Based Mapping)** 的 3D 点云建图在技术逻辑和实际应用上有着本质的区别。





## 一.rtabmap3d 建图与 2d建图特点比较

| **特性**       | **Hector SLAM** | **Gmapping**       | **RTAB-Map**                   |
| -------------- | --------------- | ------------------ | ------------------------------ |
| **维度**       | 2D 栅格         | 2D 栅格            | **3D 点云 + 2D 栅格**          |
| **传感器需求** | 高频激光雷达    | 激光雷达 + 里程计  | **RGB-D 相机/激光/IMU**        |
| **回环检测**   | 无              | 有（较弱）         | **极强（基于视觉词袋）**       |
| **计算开销**   | 极低            | 中等               | 较高（涉及图像处理）           |
| **适用场景**   | 室内、地面平稳  | 室内、大部分机器人 | **室内外、复杂地形、三维重建** |

### 主要优点：

**安全性更高**：3D 建图转换出的 2D 地图能包含更多高度信息。例如，它能发现地上的细小电线或突出的桌面（2D 激光雷达可能会因为扫在桌腿之间而误认为那是空地）。

例如：左中右分别是仿真环境，gmapping 2d栅格地图，rtabmap 3d转2d地图



<img src="https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602180915158.png" width="50%"> <img src="https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602180925904.png" alt="image-20260218092533777" style="zoom: 80%;" />

<img src="https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602180927423.png" alt="image-20260218092722294" style="zoom: 67%;" />

### 定位能力更强：

当机器人迷路时，RTAB-Map 的重定位（Relocalization）成功率远高于 2D SLAM，因为它看到的“世界”更具体。





## 二.下载noetic版本下的rtabmap

```
sudo apt update
sudo apt install ros-noetic-rtabmap-ros
```

![image-20260218095338452](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602180953731.png)

可以在图中文件夹下查看下载的源代码



## 三.启动gazebo仿真环境

```
roslaunch wpr_simulation wpb_stage_robocup.launch
```

![image-20260218095743940](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602180957358.png)



## 四.启动rtabmap建图

```
roslaunch wpr_simulation rtabmap_wpb_stage_robocup.launch
```

![image-20260218101458408](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181014743.png)

```
<?xml version="1.0"?>
<launch>
  <arg name="rgb_topic"               default="/kinect2/qhd/image_color_rect" />
  <arg name="depth_topic"             default="/kinect2/sd/image_depth_rect" />
  <arg name="camera_info_topic"       default="/kinect2/qhd/camera_info" />
  
  <arg name="frame_id"                default="base_footprint"/>
  <arg name="rtabmap_viz"             default="true" />
  <arg name="use_sim_time"            default="true"/>
  <arg name="approx_sync"             default="true"/> 

  <include file="$(find rtabmap_launch)/launch/rtabmap.launch">
    <arg name="rtabmap_viz"        value="$(arg rtabmap_viz)" />
    <arg name="use_sim_time"       value="$(arg use_sim_time)"/>
    <arg name="frame_id"           value="$(arg frame_id)"/>
    
    <arg name="rgb_topic"          value="$(arg rgb_topic)" />
    <arg name="depth_topic"        value="$(arg depth_topic)" />
    <arg name="camera_info_topic"  value="$(arg camera_info_topic)" />
    
    <arg name="visual_odometry"    value="false"/>
    <arg name="odom_topic"         value="/odom"/>
    
    <arg name="approx_sync"        value="$(arg approx_sync)"/>
    
    <arg name="args"               value="--delete_db_on_start"/>
  </include>

  <node pkg="tf" type="static_transform_publisher" name="camera_correction_final" 
      args="0 0 0 -1.5708 0 -1.95 kinect2_dock kinect2_head_frame 10" />

</launch>
```

------

```
此段代码做了tf坐标变换，运行rostopic echo /kinect2/qhd/image_color_rect/header，可以查看到摄像头话题的消息头，可以直接看到它自称属于哪个坐标系
```

![image-20260218112109261](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181121598.png)

## 五.启动rviz查看

```
rviz
```

![image-20260218112242950](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181122319.png)

设置一些设置查看3d点云地图

```
rosrun wpr_simulation keyboard_vel_ctrl
操控机器人移动建立完整的3d点云地图
```

![68a61216629db801a1a71a9dcdb0e30a](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181126229.png)

建好的3d点云地图会自动保存到数据库的db文件，如果需要利用该地图进行movebase导航需要先转为2d地图

## 六.查看3d点云地图以及转为2d栅格地图

```
rtabmap-databaseViewer ~/.ros/rtabmap.db
```

![image-20260218121042198](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181210947.png)

![image-20260218121137823](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181211489.png)

依次点击update all neigubor -> view 3d map 

![image-20260218121308624](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181213182.png)

然后勾选此三项

![image-20260218121411947](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181214498.png)

接着先关闭database，然后重新打开，然后生成2d map，就可以看到原来灰色的导出2d 地图选项就可以启用了

![image-20260218121627345](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181216565.png)

将导出的2d栅格地图文件剪切复制到wpr_simulation/maps文件夹下

![image-20260218121801280](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181218567.png)

注意修改launch中的map地图的名称

```
roslaunch wpr_simulation wpb_navigation.launch
```

![image-20260218122104197](https://picter1.oss-cn-wuhan-lr.aliyuncs.com/img/202602181221579.png)

运行launch文件后即可利用rtabmap转成的2d栅格地图进行导航
