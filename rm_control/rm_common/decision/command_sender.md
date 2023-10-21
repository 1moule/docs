**before reading**

- 用控制器控制时，通过rqt向topic发消息

- 当开manual时，就是用遥控器控制时，他是通过command_sender发送消息，

- chassis_gimbal_manaual.cpp为例，里面有一个sendCommand函数，是用来向话题发消息的；这个函数实际上是command_sender里面多个类的实例的sendCommaVel2DCommandSendernd函数的集合

