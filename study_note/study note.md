1. **在工作空间下catkin_make编译软件包报错：has built by "catkin_build"**
     1.catkin clean 2.catkin init

2. #Resource not found: loam_velodyne
   #ROS path [0]=/opt/ros/noetic/share/ros
   #ROS path [1]=/home/chen/catkin_ws/src
   #ROS path [2]=/home/chen/catkin_ws1/src
   #ROS path [3]=/opt/ros/noetic/share
   #The traceback for the exception was written to the log file

1. cause:没有catkin build
   solution:catkin build
2. catkin build后还是这样
cause:没有source     ；soution:source一下

3. **git clone到工作空间后，catkin_make报错：**
   The specified source space "/home/chen/catkin_loam/src" does not exist
   cause:

4. **g++编译时报错，（.h文件和.cpp文件都存在）**
   GazeboRosVelodyneLaser.cpp:35:10: fatal error: velodyne_gazebo_plugins/GazeboRosVelodyneLaser.h: No such file or directory
      35 | #include <velodyne_gazebo_plugins/GazeboRosVelodyneLaser.h>
         |          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   compilation terminated.
   ##solution:
   回工作空间catkin_make就编译好了，在工作空间下 /devel/lib目录下有.so文件

gazebo插件的cpp文件在工作空间catkin_make后生成.so文件，.so文件就是插件
将.so文件移动到 /opt/ros/noetic/lib然后gazebo就可以用这个插件了

5. **关于激光雷达这些插件在gazebo的使用**

1. 要下载好模型（xacro文件），还有插件（.cpp文件，编译后生成.so文件）
2. 要在机器人的模型的urdf里面加入这个模型的插件
3. rviz里面我这里用的是VLP-16的插件，怎么在smb机器人的rviz里面add这个插件呢？明天解决。good

6. **更换清华源**

1. 在清华镜像站右下角点使用帮助，在里面选ubuntu，就可以看到需要copy的东西了

#还源流程
1.备份原来的源	sudo cp /etc/apt/sources.list /etc/apt/sources_init.list
2.更换源    sudo gedir /etc/apt/sources.list
3.更新源    sudo apt-get update
4.更新软件    sudo apt-get upgrade  
#修改bashrc文件
vim ~/.bashrc

7. **rviz中laserscan显示激光**
   （1）看那几个坐标系，发现有一个rslidar，这个是激光雷达，所以安装了他的sdk
   sudo apt-get install ros-noetic-rslidar-sdk
    (2)最后pointcloud2显示出了激光雷达，laserscan也有，因为smb那个包里面有pointcloud_to_laserscan
    大功告成！！！！
8. **clion显示not found “ros/ros.h”**
    #因为没有写cmakelist，加上：add_executable(功能包名 src/.cpp文件)
9. **解压tar.gz格式的压缩包**
   命令行：tar -zxvf xxx.tar.gz
10. 删除当前目录下的所有内容
   sudo rm -rf *
11. clion左侧边，显示commit标签

 `setting` => `Version Control` => `Commit` => `Use non-modal commit interface`
