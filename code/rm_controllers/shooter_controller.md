# 1. standard.cpp

## 1. normalize()

1. M_PI是c++中的一个宏，就是圆周率pi
2. push_per_rotation是转一周拨出的弹的数量
3. 2pi/push_per_rotation就是拨出一颗蛋转的角度
4. ctrl_trigger.setcommand()就是用控制器控制trigger的位置，应该是为了消除累积的位置误差

## 3. push()

1. (time-last_shoot_time).toSec() >= 1. / cmd_.hz

toSec()是把单位转换成秒；cmd.hz是rqt设的发射频率，倒数就是发射的周期（两次发射间的时间间隔）

2. ```c++
   ctrl_trigger_.setCommand(ctrl_trigger_.command_struct_.position_ - 2. * M_PI / static_cast<double>(push_per_rotation_));
                            
   //让trigger电机逆时针转一个拨差（拨一颗子弹所需转动的角度，也就是一个拨差）
   
   //setcommand()之后，会把这个command存到.command_struct_.position_
   ```

## 4. block()

1. ```
   if (std::abs(ctrl_trigger_.command_struct_.position_ - ctrl_trigger_.joint_.getPosition()) <
           config_.anti_block_threshold ||
       (time - last_block_time_).toSec() > config_.block_overtime)
   ```

这是判断是否反转成功的if语句，如果command中的位置跟当前的位置的误差小于anti_block_threshold（说明反转完成了），或者当前离卡弹的时刻过了block_overtime，那就说明反转完成，退出block