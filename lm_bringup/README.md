# lm_bringup：项目集成入口

当前使用 global_localization 原版三节点功能包，位于工作区顶层。本包由 moma_perception 父仓库维护，与 FAST_LIO 同级。
不要让 moma_slam1 中的同名包同时进入 catkin 扫描范围。

## 定位

先启动雷达驱动，再运行：

```bash
roslaunch lm_bringup moma_localize.launch map:=/absolute/map.pcd
```

二维地图、点云转扫描和 body→base_link 静态 TF 始终启动：

```bash
roslaunch lm_bringup moma_localize.launch map:=/absolute/map.pcd 2dmap:=/absolute/map.yaml
```

默认 PCD/YAML 路径沿用 fast_lio/PCD/scans.pcd、scans.yaml，目前未附带这些地图。
二维 YAML 引用的 PGM 也需存在；二维地图与 PCD 需在同一 map 坐标下对齐。
启动文件组合 FAST_LIO、PCD 发布器、global_localization 原版三个节点和可选 RViz，不启动雷达驱动。
直接启动 global_localization.py、transform_fusion.py、publish_initial_pose.py，不 include 定位包 launch；RViz 使用本包 rviz/moma_localize.rviz 独立配置。
原版脚本使用固定话题 /cloud_registered、/Odometry、/map，不读取合并版定位 YAML。自动初始化的固定值仍在原版脚本中。
RViz 的 /initial_pose_used 显示项已关闭，原版不发布该话题。
MID360 专用配置位于 config/mid360_localize.yaml，部署前按真实设备核对外参。
body→base_link 沿用旧定位 launch 的 -0.32 0 -0.79 0 0 0，通过 body_to_base_link 参数覆盖。
body→base_link 由此 launch 统一发布，其他节点不要重复发布。定位不发布 odom→robot_foot_init。

## 建图

```bash
roslaunch lm_bringup moma_build_map.launch pcd_save_name:=scans.pcd
```

项目建图 launch、OctoMap launch、MID360 项目配置和建图 RViz 已迁入本包。
FAST_LIO 算法源码未修改，保存目录仍遵循现有实现，为 FAST_LIO/PCD。
建图沿用原建图静态外参和 OctoMap 参数；与定位分开启动，避免重复里程计。

## Open3D 运行依赖

在目标 ROS Python 3 环境中执行，不是在当前 Windows 工作区中安装：

```bash
source /opt/ros/noetic/setup.bash
python3 -m pip install open3d
python3 -c "import rospy, ros_numpy, open3d; print(open3d.__version__)"
```

若使用虚拟环境，需确保 roslaunch 启动脚本使用的 Python 能导入上述模块。
Open3D 尚未在目标 ROS 环境安装或验证；仅安装在其他 Python 环境不会生效。

## 验证范围

已检查 XML/YAML、文件引用和定位数值/状态测试。
当前环境未提供 ROS/catkin/Open3D，未执行真实编译、roslaunch、ICP 或 RViz 渲染。
