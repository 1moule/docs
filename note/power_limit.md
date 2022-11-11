# 1. 过程

## 1. referee.cpp

### 1. Referee::unpack(...)

1. 当读到裁判系统的数据并且解包后，会把is_online的值设为true

### 2. SuperCapacitor::receiveCallBack(...)

1. 当接收到超级电容的数据后，调用这个回调函数，这个回调函数会把data_.is_online设为true

## 2. power_limit.h

### 1. getLimitPower()

1. 经过1.1和1.2之后，两个is_online都设为了true（假设id为hero）

2. 当game_progress_=1时，直接return30，否则继续往下判断

3. 当从裁判系统读到的底盘功率限制（就是主控上设置的功率限制）大于120时，limit_power_等于burst_power。

4. else，根据state_进行不同的动作

5. test()：limit_power_赋值为0

6. burst()：如果超级电容的power大于一个阈值，

   （1）小陀螺模式：limit_power_等于主控上设置的功率限制+extra_power（hero的是30）；
   （2）else：等于burst_power（参数文件中设置）

7. normal()：等于主控上设置的功率限制

8. charge()：等于主控上的功率限制*0.85

9. 如果state_不是burst并且