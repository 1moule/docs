# IMU

## 1. 什么是imu

1. 惯性测量单元，是测量物体三轴姿态角以及加速度的装置
2. 一般，一个IMU包含了三个单轴的加速度计和三个单轴的陀螺

## 2. imu功能

1. 加速度计检测物体在载体坐标系统独立三轴的加速度信号
2. 陀螺仪检测载体相对于导航坐标系的角速度信号
3. 对这些信号进行处理后，可以解算出物体的姿态

## 3. 注意

1. IMU提供的是一个相对的定位信息，它的作用是测量相对于起点物体所运动的路线，所以它并不能提供你所在的具体位置的信息
2. 因此，它常常和GPS一起使用，当在某些GPS信号微弱的地方时，IMU就可以发挥它的作用，可以让汽车继续获得绝对位置的信息，不至于“迷路”。

## 4. imu特性

1. **更新频率高**，工作频率可以达到100Hz以上
2. **短时间内的推算精度高**

## 5. 不同精度、价格对应的应用场景

![image](https://img-blog.csdnimg.cn/20190910201426711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3podWlxaXV6aHVveXVlNTgz,size_16,color_FFFFFF,t_70)