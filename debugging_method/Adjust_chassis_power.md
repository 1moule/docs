# 1.before adjusting

1. 领悟rm_control/rm_common/include/rm_common/decision/power_limit.h
2. 领悟rm_controllers/rm_chassis_controller/src/chassis_base.cpp中的powelimit函数

# 2. method

1. 打开plotjuggler，听/controller/chassis_controller/command/power_liimt,以及/power_heat_data/chassis_power
    (1)command中的power_limit是manual通过command_sender发送的，chassis_power中的是从裁判系统中返回的当前底盘功率
2. 平地起步、急刹、匀速，保证功率接近功率上限
    （1）当功率过小时，将k2调小，因为这个改变的是一元二次方程的c，k2变小，c变小，图像下移，方程的根即缩放系数变大；过大同理
    （2）平地上k2对功率的影响更大，可由公式看出，因为k2是速度的平方的系数，k1是扭矩的系数；由p=fv，平地上速度大而扭矩小，故改变k2影响更大
3. 在主控上切换多个功率上限，重复2，保证每个功率上限下，都接近而不超
4. 爬坡
    （1）当爬坡时功率过小时，将k1调小，从公式理解，k1那一项变小，那么功率那一项变大，输出力矩更大，吃的功率就更多
    （2）k1是扭矩的系数，因为爬坡时速度小而扭矩大，所以k1的影响更大

# 3. others

1. command中的power limit是由manual调用command sender发送的，而command sender会调用rm_common中的power_limit.h来给power limit赋值
2. 关于赋的值的大小，可看power_limit.md
