## 1. 流程

### 1. 连接nuc

1. 与nuc连到同一个wifi（nuc已经配网，连了604的wifi），通过进入网关（192.168.1.1）查看nuc的ip
2. 打开终端输入命令行

```
ssh dynamicx@nuc的ip
```

3. 在nuc的终端输入命令行

```
roscore

mon launch rm_config rm_hw.launch

mon launch rm_config load_controllers.launch
```

4. 在自己的终端打开rqt

### 2. 出现问题：

1. 打开rqt后，运行plugin，终端显示无法加载

solution：在.bashrc复制粘贴阵雨哥发过来的那个.bashrc里面的ros那一段，然后根据nuc的ip修改ROS_MASTER_URI，因为这些工具是通过网络连接的，

2. rqt中打开shooter_controller，改变mode的值，发现没有反应

solution：在nuc的终端catkin clean，再重新catkin build；阵雨哥说这里是因为rm_msgs出了点问题，叫我重新传功能包，后面敏源哥说不用，因为调试的那台步兵上面是已经有整套的源码了的，只要catkin clean删掉build跟devel的文件，然后重新生成可执行文件就行，需要传功能包只有在写了新的源码需要传到nuc上时才有

3. 要发现各种异常，比如电路、机械的锅（没有扎线、某块板没装好之类的）