# 相机、imu联合标定

## 1. 什么是标定

1. 标定，主要是指使用标准的计量仪器对所使用仪器的准确度（精度）进行检测，检测它是否符合标准。一般用于精密度较高的仪器。标定也可以认为是校准。

## 2. 为什么要标定

1. 确定仪器或[测量系统](https://baike.baidu.com/item/测量系统)的输入—输出关系，赋予仪器或测量系统分度值；
2. 确定仪器或测量系统的静态特性指标；
3. 消除系统误差，改善仪器或系统的[精确度](https://baike.baidu.com/item/精确度/1612899)。
4. 在科学测量中，标定是一个不容忽视的重要步骤。

## 3. 步骤

1. 相机标定
2. imu标定
3. 相机、imu联合标定

（步骤1、2可以同步进行）

## 4. 准备工作

### 1. 软件方面

- Linux系统，例如Ubuntu 18.04
- ROS环境，例如ROS Melodic
- [RealSense SDK](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md) 与 [realsense-ros-wrapper](https://github.com/IntelRealSense/realsense-ros)
- [kalibr](https://github.com/ethz-asl/kalibr)
- [code_utils](https://github.com/gaowenliang/code_utils) 与 [imu_utils](https://github.com/gaowenliang/imu_utils)
- 其他相关依赖

### 2. 硬件方面

- Intel Realsense D435i相机
- 标定板一个，例如打印好的AprilTag或棋盘格

**注：**

使用kalibr生成标定板的命令格式如下，运行后可以得到一个pdf文件。需要使用原始尺寸打印出来，并且这里的参数要记好，后面需要用。

```bash
kalibr_create_target_pdf --type apriltag --nx [NUM_COLS] --ny [NUM_ROWS] --tsize [TAG_WIDTH_M] --tspace [TAG_SPACING_PERCENT]
```



## 5. 使用Kalibr标定camera

1. 

## 6. 使用imu_utils标定imu

### 1. 运行D435i相机

1. 指定参数运行

```bash
roslaunch realsense2_camera rs_camera.launch \
    unite_imu_method:="linear_interpolation" \
    enable_gyro:=true \
     enable_accel:=true
```

2. 修改rs_camera.launch文件，然后直接运行

### 2. 录制imu数据包

