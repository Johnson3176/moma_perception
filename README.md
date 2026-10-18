# moma_perception

机器人点云处理、里程计、建图与全局定位的集成仓库。父仓库本身不是 ROS 功能包，各功能包由 `lm_bringup` 组合启动。

## 当前结构

- `FAST_LIO`：Git 子模块，ROS 包名为 `fast_lio`，负责激光惯性里程计与建图。
- `global_localization`：Git 子模块，负责先验点云地图匹配、初始化位姿与全局位姿输出。
- `lm_bringup`：由父仓库直接维护，负责建图与定位的集成启动、项目配置及 RViz。
- `robot_self_filter`（TODO）：自体点云过滤，计划作为独立子仓库功能包，暂未创建。
- `rolling_cloud_map`（TODO）：局部连续帧点云维护与发布，计划作为独立子仓库功能包，暂未创建。

## 获取与准备

克隆父仓库后，在仓库根目录初始化子模块及嵌套依赖：

```bash
git submodule update --init --recursive
```

将本仓库放在 ROS 1 catkin 工作空间的 `src` 下，安装依赖并编译。使用 Python 3 的 ROS 环境，例如 Noetic：

```bash
cd /path/to/catkin_ws
source /opt/ros/noetic/setup.bash
rosdep install --from-paths src --ignore-src -r -y
python3 -m pip install open3d
catkin_make
source devel/setup.bash
```

Open3D 需安装到定位脚本实际使用的 Python 环境。具体依赖、数据接口与原版算法限制见 `global_localization/README.md`。
工作空间中不要同时保留可被 catkin 扫描到的同名包副本，例如旧工作目录中的 FAST_LIO 或顶层 global_localization。

## 建图

先启动雷达驱动，再运行：

```bash
roslaunch lm_bringup moma_build_map.launch
```

指定退出时保存的 PCD 文件名：

```bash
roslaunch lm_bringup moma_build_map.launch pcd_save_name:=scans.pcd
```

默认点云保存目录为 `FAST_LIO/PCD`。保存需启用配置中的 PCD 保存开关，并让节点正常退出。
建图 launch 同时启动 OctoMap；发布二维投影地图不等于自动保存二维 YAML/图片，二维地图文件需要另行保存。

## 全局定位

先准备三维点云地图、二维地图 YAML 及其引用的图片，再启动雷达驱动并运行：

```bash
roslaunch lm_bringup moma_localize.launch
```

默认地图路径为：

```text
$(find fast_lio)/PCD/scans.pcd
$(find fast_lio)/PCD/scans.yaml
```

也可以显式指定地图：

```bash
roslaunch lm_bringup moma_localize.launch map:=/absolute/map.pcd 2dmap:=/absolute/map.yaml
```

定位入口统一启动 FAST_LIO、原版三个定位节点、三维地图发布器、二维地图服务、点云转 LaserScan 和 `body → base_link` 静态 TF。二维地图话题为 `/prior_map`，三维点云地图话题为 `/map`。
RViz 默认开启，可通过 `rviz:=false` 关闭。不要同时启动 `global_localization.launch`，以免重复发布节点和 TF。

## 配置与使用约定

MID360 建图和定位参数位于 `lm_bringup/config`。部署前核对雷达/IMU 话题、消息类型和外参。
当前原版定位的固定初值及 ICP 参数仍写在 `global_localization/scripts` 中，并非从合并版定位 YAML 读取。

建图与定位 launch 当前使用的 `body → base_link` 外参不同，使用前应按实际机器人核对并统一；其他节点不要重复发布这条 TF。
建图和定位应分别运行，避免重复启动 FAST_LIO。

父仓库只记录子模块的提交编号。修改子模块后，应先在对应子仓库提交并推送，再在父仓库提交版本更新。
