## 1. 牛顿欧拉 动力学建模

​	轮腿有三个机构：驱动轮、摆杆、机体，对各个机构的平动、转动分别使用牛二定律进行力学分析得到$$x、\theta、\phi$$的二阶导数的表达式，这些表达式只包含机器人的物理参数以及状态量和控制量，最终可以得到状态空间方程$$\dot x=Ax+Bu$$

### 1.1 经典力学分析

1. 对驱动轮：

   - 平动
     $$
     m_w\ddot x=N_f-N \tag{1.1}
     $$

   - 转动
     $$
     I_w \frac{\ddot x}{R}=T-N_fR \tag{1.2}
     $$
     ​	驱动轮产生的扭矩一部分使驱动轮平动，一部分使驱动轮自身转动。平动牵引力等于地面对驱动轮的摩擦力。

     ​	由1.1得到$$N_f=m_w\ddot x+N$$，带入1.2得到
     $$
     I_w \frac{\ddot x}{R}=T-(m_w\ddot x+N)R \\
     \frac{I_w }{R}\ddot x+m_wR\ddot x=T-NR \\
     \ddot x=\frac{T-NR}{\frac{I_w }{R}+m_wR} \tag{1.3}
     $$

2. 对摆杆

   - 平动

     - 水平方向
       $$
       N-N_M=m_p\frac{\partial^2}{\partial t^2}(x+L\sin\theta)\\
       N-N_M=m_p\frac{\partial}{\partial t}(\dot x+L\cos\theta\dot \theta)\\
       N-N_M=m_p(\ddot x-L\sin\theta\dot \theta^2+L\cos\theta\ddot \theta) \tag{1.4}
       $$

     - 竖直方向
       $$
       P-P_M-m_pg=m_p\frac{\partial^2}{\partial t^2}(L\cos\theta)\\
       P-P_M-m_pg=-m_p\frac{\partial}{\partial t}(L\sin\theta\dot \theta)\\
       P-P_M-m_pg=-m_p(L\cos\theta\dot\theta^2+L\sin\theta\ddot \theta)\\\tag{1.5}
       $$
       

   - 转动（使摆杆逆时针转的力为正）

     将驱动轮、机体对摆杆的力分别分解为水平、竖直方向后得到4个力（$$P、P_M、N、N_M$$），将这4个力再分别分解为沿杆方向和垂直于杆的方向。沿杆方向的力被抵消，而垂直于杆的方向的力会使摆杆发生旋转，这些力产生的扭矩为 力*转动半径
     $$
     I_p\ddot \theta=(P\sin \theta L-N\cos \theta L)+(P_M\sin \theta L_M-N_M\cos\theta L_M)-T+T_p\tag{1.6}
     $$
     ​	因为当驱动轮输出扭矩时，摆杆会受到一个大小相等符号相反的扭矩，因此要减T。髋关节电机输出的扭矩直接就是摆杆受到的扭矩，因此加Tp

3. 对机体

   - 平动

     - 水平方向
       $$
       N_M=M\frac{\partial^2}{\partial t^2}(x+(L+L_M)\sin\theta-l\sin\phi)\\
       N_M=M\frac{\partial}{\partial t}(\dot x+(L+L_M)\cos\theta\dot \theta-l\cos\phi\dot \phi)\\
       N_M=M(\ddot x-(L+L_M)\sin\theta\dot \theta^2+(L+L_M)\cos\theta\ddot \theta+l\sin\phi\dot \phi^2-l\cos\phi\ddot\phi) \tag{1.7}
       $$

     - 竖直方向
       $$
       P_M-Mg=M\frac{\partial^2}{\partial t^2}((L+L_M)\cos\theta+l\cos\phi)\\
       P_M-Mg=-M\frac{\partial}{\partial t}((L+L_M)\sin\theta\dot \theta+l\sin\phi\dot \phi)\\
       P_M=Mg-M((L+L_M)\cos\theta\dot\theta^2+(L+L_M)\sin\theta\ddot \theta+l\cos\phi\dot\phi^2+l\sin\phi\ddot\phi)\\\tag{1.8}
       $$

   - 转动
     $$
     I_M\ddot\phi=T_p+N_Ml\cos\phi+P_Ml\sin\phi \tag{1.9}
     $$

4. 求解

   - 将1.7带入1.4得到
     $$
     N=m_p(\ddot x-L\sin\theta\dot \theta^2+L\cos\theta\ddot \theta)+M(\ddot x-(L+L_M)\sin\theta\dot \theta^2+(L+L_M)\cos\theta\ddot \theta+l\sin\phi\dot \phi^2-l\cos\phi\ddot\phi) \tag{1.10}
     $$
     将1.8带入1.5得到
     $$
     P=-m_p(L\cos\theta\dot\theta^2+L\sin\theta\ddot \theta)+m_pg+Mg-M((L+L_M)\cos\theta\dot\theta^2+(L+L_M)\sin\theta\ddot \theta+l\cos\phi\dot\phi^2+l\sin\phi\ddot\phi) \tag{1.11}
     $$
     到这里就得到了$$P、N、P_M、N_M$$的表达式，他们只与状态量有关

   - 将$$P、N、P_M、N_M$$的表达式以及1.3、1.6、1.9一共7个式子丢到matlab进行符号表达式计算，matlab代码如下。
   
     ```matlab
        % 所需符号量
         syms theta theta1 theta2; % theta1=dTheta, theta2=ddTheta
         syms x x1 x2;
         syms phi phi1 phi2;
         syms T Tp N P Nm Pm Nf t;
         
         % 机器人结构参数
         syms R L Lm l mw mp M Iw Ip Im g;
         
         % 进行物理计算
         Nm=M*(x2+(L+Lm)*(theta2*cos(theta)-theta1^2*sin(theta))-l*(phi2*cos(phi)-phi1^2*sin(phi)));
         Pm=M*g+M*((L+Lm)*(-theta1^2*cos(theta)-theta2*sin(theta))-l*(phi1^2*cos(phi)+phi2*sin(phi)));
         N=Nm+mp*(x2+L*(theta2*cos(theta)-theta1^2*sin(theta)));
         P=Pm+mp*g+mp*L*(-theta1^2*cos(theta)-theta2*sin(theta));
         
         equ1=x2-(T-N*R)/(Iw/R+mw*R);
         equ2=(P*L+Pm*Lm)*sin(theta)-(N*L+Nm*Lm)*cos(theta)-T+Tp-Ip*theta2;
         equ3=Tp+Nm*l*cos(phi)+Pm*l*sin(phi)-Im*phi2;
         
         % solve函数求解x2,theta2,phi2的符号表达式
         [x2,theta2,phi2]=solve(equ1,equ2,equ3,x2,theta2,phi2);
         
         % 求得雅克比矩阵即非线性模型的A、B矩阵
         Ja=jacobian([theta1;theta2;x1;x2;phi1;phi2],[theta theta1 x x1 phi phi1]);
         Jb=jacobian([theta1;theta2;x1;x2;phi1;phi2],[T Tp]);
         
         % 使用sbus进行变量替换，得到线性化后的状态空间方程A、B矩阵
         A=vpa(subs(Ja,[theta theta1 x x1  phi phi1],[0 0 x 0 0 0]));
         B=vpa(subs(Jb,[theta theta1 x x1  phi phi1],[0 0 x 0 0 0]));
     ```
     
     使用matlabFunction将A、B阵的计算生成matlab函数，再使用matlab coder生成c代码，最终得到可以传入结构参数，返回A、B矩阵的c函数。



## 2.  状态观测

1. 一些符号相关：
   - 规定机器人朝前的方向为x正方向。
   - （五连杆）：所有的joint，当角度为0时，其坐标系都是x轴沿机器人朝前的方向，逆时针旋转角度为正，反之为负。在计算vmc时要给电机角度加上offset，令后关节大腿水平时角度为0，逆时针转时角度增加为锐角；前关节大腿水平时角度为pi，顺时针转时角度减小为钝角。因为左右腿上的关节坐标系方向一致，因此处理是一样的。这样处理后经过vmc算出来，当腿向后摆时theta就为正，腿竖直时theta为0
   - （串联腿）：
   - 机体向朝前的方向往下俯时，phi要为负，上仰时要为正
2. 一共有4个状态量：摆杆角度、角速度；位移、速度；机体角度、角速度
   - 摆杆角度、角速度通过VMC计算得到，vmc得到的是相对机体的摆角，叠加机体倾角后得到广义摆角
   - 机体角度通过imu获取，注意imu获取的角度、角速度符号，必须满足机体向朝前的方向往下俯时，phi为负，上仰时为正
   - 速度通过轮速运动学解算得到，位移通过速度积分得到。可以在速度小于一定阈值（即基本静止）时积分位移，运动时令位移为0，实现运动时控速度，静止时能定住。



### 3. 离地检测

