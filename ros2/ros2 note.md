1. 创建变体（ros1的元包）

   - package.xml

     ```xml
     <?xml version="1.0"?>
     <?xml-model href="http://download.ros.org/schema/package_format2.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
     <package format="2">
       <name>rm_ros2_controllers</name>
       <version>0.0.0</version>
       <description>Meta package that contains package for RoboMaster.</description>
       <maintainer email="maintainer-email">Maintainer Name</maintainer>
       <license>BSD</license>
       <!-- packages in project -->
       <exec_depend>rm_chassis_controllers</exec_depend>
     
       <export>
         <build_type>ament_cmake</build_type>
       </export>
     </package>
     ```

   - cmakelist

     ```
     cmake_minimum_required(VERSION 3.5)
     
     project(rm_ros2_controllers NONE)
     find_package(ament_cmake REQUIRED)
     ament_package()
     ```

     

2. 