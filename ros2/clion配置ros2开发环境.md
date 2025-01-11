# clion配置ros2开发环境

### 1. 传统做法

[这是官方文档](https://www.jetbrains.com/help/clion/ros2-tutorial.html)

（下面使用一个随机创建的软件包为例子）

1. 创建一个软件包

   ```
   ros2 pkg create --build-type ament_cmake cpp_pubsub
   ```

2. 编译

   ```
   colcon build --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -G Ninja
   ```

   注意这里需要安装Ninja

   ```
   sudo apt-get install Ninja-build
   ```

   编译后会在工作空间下的build目录下生成一个compile_commands.jason文件，需要从该文件打开工作空间

3. 在clion打开工作空间后，默认project root是在build下，需要修改

   - 在主页面上方找到Tools | Compilation Database | Change Project Root 
   - 修改为工作空间的路径

4. 编译单个软件包

   - 在软件包目录下新建一个文件，名为cmake_commands.bat

   - 将下面内容复制到cmake_commands.bat文件中，注意修改路径，并修改文件权限

     ```bash
     #构建命令
     /usr/bin/cmake /home/ros2/ros2_ws/src/cpp_pubsub -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -G Ninja -DCMAKE_INSTALL_PREFIX=/home/ros2/ros2_ws/src/cpp_pubsub
     #编译命令
     /usr/bin/cmake --build /home/ros2/ros2_ws/build/cpp_pubsub -- -j8 -l8
     #安装命令
     /usr/bin/cmake --install /home/ros2/ros2_ws/build/cpp_pubsub
     ```

   - 修改文件权限

     ```bash
     sudo chmod 777 cmake_commands.bat
     ```

   - clion中找到Settings | Build, Execution, Deployment | Custom Build Targets，按下图操作

     ![Creating a custom build target for CMake commands](https://resources.jetbrains.com/help/img/idea/2023.3/cl_ros2_customtool.png)

   - 找到主页面上方Run | Edit Configurations，选择Target，这里的Target就是刚才设置的Custom Build Targets

     ![Custom configuration](https://resources.jetbrains.com/help/img/idea/2023.3/cl_ros2_custom_config.png)

5. 编译整个工作空间

   -  找到Settings | Tools | External Tools 

   - 将下图中的内容全部照抄

     ![External tool for colcon build](https://resources.jetbrains.com/help/img/idea/2023.3/cl_ros2_colcon_build_external_tool.png)

     完成后，在clion编译后，在compile_commands.jason中可以看到所有软件包，此时cpp文件也可以找到对应cmakelist



### 2. 通过添加顶层CMakelists

创建两个工作空间dev_ws、build_ws

dev_ws将添加顶层cmakelist，用于代码开发；build_ws通过软链接链接dev_ws中的所有包，并使用colcon build编译



原因：添加顶层cmakelist后，dev_ws中使用colcon build会有问题。为了兼顾二者，选择在build_ws中通过软链接dev_ws中的包，使用colcon build编译。



dev_ws：

1. 克隆仓库

   ```bash
   git clone git@github.com:kai-waang/colcon-toplevel-cmake.git /opt/ros/scripts/cmake
   ```

2. 将克隆下来的包复制到工作空间目录下

   ```bash
   cp /opt/ros/scripts/cmake/toplevel.cmake <path_to_workspace>/CMakeLists.txt
   ```

3. 配置clion

   ![image](/home/ros2/Pictures/Screenshot from 2024-12-23 21-16-14.png)

build_ws：

```bash
# 进入 build_ws/src 目录
cd ~/build_ws/src

# 遍历 dev_ws/src 目录下的所有包，并创建符号链接
for dir in ~/ros2_ws/src/*; do
    if [ -d "$dir" ]; then
        ln -s "$dir" .
    fi
done
```

