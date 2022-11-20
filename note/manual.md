## 1. 流程

### 1. main.cpp

1. 定义了这个节点的主函数，这个节点的运行实际是循环运行一个run函数，run函数是写在manual_base.h中的一个虚函数
2. main函数中会根据robot type的不同，然后调用不同的run，
   - 步兵：chassi_gimble_shooter_cover_manual
   - 英雄：chassis_gimble_shooter_manual
   - 工程：engineer_manual

## 2. 代码结构

### 1. manual

- chassi_gimble_shooter_cover_manual继承chassi_gimble_shooter_manual
- chassi_gimble_shooter_manual、engineer_manual继承chassi_gimble_manual
- chassis_gimble_manual、dart_manual继承manual_base



### 2. 英雄manual为例，详解

1. 大部分为manual_base中声明的虚函数
2. 其余为写键位，本质是改变msg的内容，然后通过sendCommand发布到话题上

## 2.  manual_base

1. run()：
   - 从串口读取裁判系统的数据并放到buffer中
   - 从buffer读取裁判系统的数据
   - checkReferee() 检查裁判系统，实际是检查机器人血量
   - checkSwitch()：先检查遥控器开还是关并执行对应函数，然后检查right_switch的状态，然后根据state_是RC还是PC，执行对应的函数
   - sendCommand：发送指令(将msg发布到topic)
   - controller_manager_.update()：更新controller manager，会根据情况开启/关闭相应的控制器
2. checkSwitch()

- remoteControlTurnOff()：stop main controllers and calibration controllers，main controllers是一个vector容器，容器中有所有主要的控制器，这个函数会把这些控制器停掉；校准控制器也停掉
- remoteControlTurnOn()：start main controllers

3. updateRc()

- manual_base中的，只检查left switch的状态，同样通过update()，（update定义在input_event.h）

4. updatePc()

- checkkeyboard()，检查键盘

5. sendCommand()，调用commandsender的sendcommand，向话题发布消息
6. robotDie()，如果遥控开启，把所有控制器stop

## 3. input_event.h

1. boost::function<>：

```c++
int main() 
{ 
  boost::function<int (const char*)> f = std::atoi; 
  std::cout << f("1609") << std::endl; 
  f = std::strlen; 
  std::cout << f("1609") << std::endl; 
} 

//这里定义了一个函数指针f，他可以指向所有符合这个签名的函数；在这个例子里也就是符合：返回int，传入参数为const char*的函数
```

2. boost::bind()：

```c++
void add(int i, int j) 
{ 
  std::cout << i + j << std::endl; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), boost::bind(add, 10, _1));
}
    
  //std::for_each()要求第三个参数为一个一元函数（即只有一个参数），而add()为二元函数；这时我们就可以通过boost::bind
    
  //boost::bind()第一个参数为函数名；第二、三个参数是传给add的参数；boost::bind的第三个参数为占位符，有_1、_2、_3、...，有几个占位符，那么别人调用这个boost::bind时，得到的就是一个几元函数
    
  //v中的元素通过占位符传到bind，再传到add
    
  //改变占位符的位置，可以改变他们传到add中作为参数时的位置
```



## 2. 写键位method

### 1. 流程

1. 在头文件中定义Inputevent类型变量
2. 在构造函数中调用变量的setfalling、setrising之类的函数，这个决定了在什么情况下会调用这个键位的函数
   - setRising：刚按下那一刻调用一次
   - setFalling：刚松开时调用一次
   - setActiveHigh：在持续按住的过程中一直调用
   - setActiveLow：在持续松开时一直调用
3. checkKeyboard()：调用变量的update函数
4. 剩下的就是按键函数的逻辑