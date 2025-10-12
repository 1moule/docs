# legged_percetive

## 1. 感知

1. t265：
   - 只需要用到libgazebo_ros_p3d.so来获取相机坐标系在gazebo world中的真实位姿，并通过它来修正机体姿态。这个gazebo插件中的两个参数，bodyName和topicName，一个是相机坐标系名字，一个是位姿发布的话题。
   - 在legged estimate包中会订阅t265发布的位姿话题，得到world2sensor的坐标系转换，通过 world2sensor * sensor2odom得到world2odom，再通过odom2world*world2sensor\*sensor2base得到odom2base。最后会把base在odom下的位姿发布到话题（/pose）
2. d435（深度相机）
   - 原理：
     - 深度相机要实现探测物体的位置、深度，实现原理主要有3种：结构光、Tof、双目成像
     - d435使用结构光方案：
       - 激光器外放置光珊，激光通过光栅会发生折射使得激光落在物体表面时会发生位移。
       - 当距离远时位移会变大，当距离近时位移会小（因为折射角不变）。
       - 通过相机检测位移变化就能得到物体的位置和深度信息，复原整个3维空间。
   - d435会发布深度点云信息，可以感知3维空间中的物体。elevation_mapping可以订阅它发布的点云信息，实现高程图的构建（订阅多线雷达发布的点云也可以，因为他们都能够感知3维空间）
3. elevation_mapping
   - 订阅点云、base姿态和协方差（legged estimate发布的）、tf；生成局部高程图并发布到话题
   - [关于elevation_mapping的各种参数](https://github.com/ANYbotics/elevation_mapping)
4. convex_plane_decomposition（凸平面分割）
   - 输入：elevation_mapping构建的高程图。输出：平面分解的结果（PlanarTerrain）
     - 平面分解的结果包括：PlanarRegion（平面区域）的容器、一张gridMap
       - 一个PlanarRegion包含：
         - BoundaryWithInset：带内缩边界的平面外边界
         - bbox2d：平面边界的二维包围盒，用来加速判断点是否在区域附近、加速空间查询
         - transformPlaneToWorld：平面从地形坐标系转换到世界坐标系的坐标系变换

   - [从输入到输出的具体过程](https://zhuanlan.zhihu.com/p/3433060477)




## 2. 控制器

1. 在mpc求解器中加入一个SynchronizedModule：planarTerrainReceiver，用来接收平面分割的结果。这个SynchronizedModule的preSolveRun中会把话题回调中处理得到的planarTerrain和signedDistanceField赋值给leggedInterface的planarTerrainPtr和signedDistanceFieldPtr
   - 回调中：
     - planarTerrain也就是平面分割的结果直接从话题接收的msg获取，用于平面选取
     
     - SignedDistanceField（符号距离场）通过grid_map::SignedDistanceField函数生成，用于碰撞约束
       - 使用平面分割后得到的grid map，使用elevation mapping构建的高程图作为图层。高程图的作用是用来判断哪些地方属于障碍。通过计算grid map上的每个点距离最近的障碍的距离得到SDF
       
         

## 3. Reference Manager

重载modifyReferences函数，修改参考轨迹

1. 根据地形修改状态量参考轨迹
   - 机体姿态：
     - 计算世界系下的地形法向量：向量的x和y为x、y方向上对地形高度微分，z固定等于1表示向量朝上。归一化后得到地形法向量
     - 使用机体yaw构造一个旋转矩阵，将地形法向量旋转到一个yaw与base坐标系相同，roll、pitch水平的坐标系
     - 使用旋转后的地形法向量，atan计算期望俯仰角

   - 机体z方向位置：
     - 地形高度+comHeight/cos（机体俯仰角）
2. 选择落足平面生成落足多边形
   - 首先按照平地上的方式根据期望轨迹计算出一个名义落足点
   - 将名义落足点投影到各个平面，**选择**距离最近的平面作为**落足平面**
   
     - 用getBestPlanarRegionAtPositionInWorld函数
   
       - sortWithBoundingBoxes 计算距离并排序
   
         - 将世界系下的落足点位置转换到平面坐标系下，得到平面系下的落足点位置
         - x、y方向，分别计算 <u>落足点</u> 到 <u>离落足点最近的bbox边界</u> 的距离平方
         - z方向用平面系落足点z位置的平方
         - 计算欧式距离，返回排序后的区域
     - 投影，落足点不在边界上投影点为距离平面系落足点最近的边界点，反之就是自身
       - 计算世界系下，落足点与投影点之间的距离，通过循环迭代出cost最小也就是距离最小的投影（projection）
     - 函数返回迭代出的cost最小的投影，其中用投影点表示projection的位置
   - 在落足平面上**生长**出一个**落足凸多边形**，落足点约束在这个多边形内
   
     - 使用convex_plane_decomposition::growConvexPolygonInsideShape函数，参数如下：
       - parentShape：要生长多边形的 外部图形 的形状
       - center：生长的凸多边形的中心点
       - numberOfVertices：凸多边形的顶点数量
       - growthFactor：每次生长顶点时的步长因子，决定顶点向外生长的速度
   
3. 修改摆动腿轨迹
   - 根据地形高度修改摆动腿z方向轨迹




## 4.Legged Interface

1. 落足点约束
   - 落足点约束在落足多边形内
     - 约束式为：param.a * endEffectorKinematicsPtr_->getPosition(state).front() + param.b>=0。表示落足点在所有的边的内侧
     - 取其中一条边为例：

       - 两个点确定一条边：(A(x_a, y_a)) 和 (B(x_b, y_b))。假设一个任意点P
       - 计算边的法向量n，以及向量AP。两个向量点乘大于等于0，点在边的内侧
       - 向量点乘的式子整理并用符号表达得到：ax+by+c>=0，应用到所有边以及二维落足点也就是param.a*pos+param.b>=0
   - 对触地腿active落足多边形**约束**，这样子mpc算出来的期望状态量就会满足落足点在多边形内，进一步的也就是状态量中关节角度会满足这个约束。因为有硬约束，并且落足点约束是软约束，所以触地腿虽然期望落足点变了，但是腿是不会动的，只有变为摆动腿后腿部关节才会向这个期望关节角度运动
2. 摆动腿足端碰撞约束
   - 计算足端点在grip map中的位置，从符号距离场得到这个点距离最近障碍的距离。约束这个距离>=足端半径就可以避免碰撞
3. 小腿碰撞约束
   - 通过pinocchio给每个小腿创建一个碰撞球，用sdf创建一个碰撞约束

