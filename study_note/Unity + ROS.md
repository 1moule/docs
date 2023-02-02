## 1. 安装软件

[可参考文章1](https://www.guyuehome.com/40908)

[文章2](https://www.guyuehome.com/41415)

### 1. 安装Unity Hub

[步骤](https://docs.unity3d.com/hub/manual/InstallHub.html?_ga=2.165915582.1081426319.1674648491-1250109943.1673670199#install-hub-linux)

### 2. 安装Unity Editor

1. 直接在Unity Hub里面安装即可
2. 添加licenses，选择personal那个即可
2. 确保有足够磁盘空间

### 3. 安装ROS

## 2. 配置Unity

1. 参考文章2

## 3. 配置本地

1. 拉Unity-Robotics-Hub，这个是demo项目

   ```bash
   git clone https://github.com/Unity-Technologies/Unity-Robotics-Hub.git
   ```

2. 拉ROS-TCP-Endpoint，这是跟unity沟通必备的

   ```bash
   git clone https://github.com/Unity-Technologies/ROS-TCP-Endpoint.git
   ```

## 4. 导入urdf

1. 将urdf放到工程目录下的Assets目录下
2. 将.xacro转换成.urdf

```
rosrun xacro xacro xxx.urdf.xacro > xxx.urdf
```

## 5. 导入FBX

1. 将fusion360模型导出为OBJ文件
2. 使用blender将OBJ转换为FBX
3. 将FBX文件添加到project的Assets目录下
4. （若材质球无法编辑）选中模型，在Inspector 将location从use embedded materials 改成use external materials(legacy)

