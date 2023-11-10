# 中国大学生工程实践与创新能力大赛 HITSZ四队代码仓库

## 介绍

这是一台按照给定任务完成物料搬运的智能机器人（简称：机器人）。该机器人能够通过扫描二维码领取搬运任务，在指定的工业场景内行走与避障，并按任务要求将物料搬运至指定地点并精准摆放（色环）。

![IMG](https://mitcher-1316637614.cos.ap-nanjing.myqcloud.com/test/IMG_0607(1).png)

## 开发日志

- 2022-10-26 加入仓库

- 2022-10-29 实现了通过OpenMV向STM32发送抓取顺序
  思路：
  1. 先通过识别二维码，获取抓取顺序qr_seq[3] (一个含有三个元素的列表，第一个元素代表第一个要抓取的物料的颜色)
  2. 识别上层货架上的物料的颜色，获取红绿蓝的cx坐标，cx_r, cx_g, cx_b）,根据坐标大小可以判断左中右关系，定义一个列表color_seq[3]
  列表的第一个元素是是红色对应的位置，第二个元素是是绿色对应的位置，第三个元素是是蓝色对应的位置（比如红色的cx最大，那么这个元素的值就是3）
  3. 定义一个抓取顺序列表grab_seq[3]，seq第i个元素的获取方式为，检索qr_seq的第i个元素，如果第一个元素是n，那么检索color_seq第n个元素的值，并作为抓取顺序列表seq第i个元素的值。(i=1,2,3;n=1,2,3)
  最后，向单片机发送抓取顺序grab_seq的前三个元素

    > “1”为红色，“2”为绿色，“3”为蓝色
    >
    > seq=1 红绿蓝（1，2，3） 左中右
    >
    > seq=2 红蓝绿（1，3，2） 左右中
    >
    > seq=3 蓝绿红（3，2，1） 右中左
    >
    > seq=4 蓝红绿（3，1，2） 右左中
    >
    > seq=5 绿红蓝（2，1，3） 中左右
    >
    > seq=6 绿蓝红（2，3，1） 中右左

- 2022-10-29 完善了定时器中断的代码，消灭了几个bug，目前huart1分配给电机，huart2分配给OpenMV，huart3分配给OPS。uart4分配给舵机（**2023.1已经在此更新串口分配**）

- 2022-10-31 在OpenMV的代码中增加了对LCD显示屏的支持，现在可以在显示屏上显示二维码的内容了，FinalTask2修改了FinalTask1中获取字符串的很笨的一种方法(map)

- 2022-10-31 修正了定时器中断代码和运动代码，串口接收中断尚待debug，目前无法进入串口回调函数

- 2022-11-02 修正串口中断代码和小车运动代码，修正cubemx配置，小车可以运动到指定地点。

- ~~2022-11-14 升级舵机固件，下载[新版资料](https://www.feetech.cn/service.html),将huart2改为舵机串口~~

- 2022-11-15 写了Servo_Demo，成功控制多个舵机，但动作尚未完善，目前发现机械结构虚位比较大

- 2022-11-16 [点击这里理解HAL_Uart_Transmit](https://controllerstech.com/uart-transmit-in-stm32/)

- 2022-12-18 更新了相关注释

- 2023-1-1 结束摸鱼，开始更新代码

> 重新接了电机的线：
>
> ID=1的电机对应右后的轮子 
>
> ID=2的电机对应左前的轮子，速度设为正值时会反转
>
> ID=3的电机对应右前的轮子，
>
> ID=4的电机对应左后的轮子，速度设为正值时会反转

- 2023-1-2 将main函数中关于定位的部分算法搬移到了action头文件中，并修改了pid_x函数中的PID公式，改用为增量式PID
  $$
  \Delta u = Kp·[e(k)-e(k-1)]+Ki·e(k)
  $$

- 2023-1-3 setMotorSpeed()函数中向串口发送数据后增加了延时，解决了发送不出去的情况

- 2023-1-4 修改了CubeMX中设置的优先级，优先级顺序改为：滴答计时器 > 串口通信 > 定时器中断，解决了串口只能控制一个电机的bug，以及在定时器中断中用HAL_Delay()会卡死的bug，并且测试了码盘数据，码盘处于可用的状态，接下来最关键的就是通过码盘解算出速度控制机器人

- 2023-1-5 如果以机器人作为坐标系的原点，那么机器人的前方是x轴正方向，机器人的左方是y轴正方向，机器人的上方是z轴的正方向，即采用右手系。今天修正了麦轮解算中的这一错误。此外，精简代码库，重新命名了motor头文件中的函数，将action定位模块的相关函数封装到了头文件，明天重点解决运动解算部分

  现在定义如下：

  floatdata[3]是机器人的y轴坐标

  -floatdata[4]是机器人的x轴坐标（注意负号）

  floatdata[0]是机器人的航向角，逆时针方向，范围是0-360°

  然而为了方便，move2position里面的angle应该是-180°到180°之间的输入，w_loss()函数已经进行了相关转换。
  
- 2023-1-6 **代码风格说明**：头文件的.h文件应该include你在对应的.c文件里面所需要的所有文件，而.c文件只需要include相应的头文件，extern的变量统一放入.c文件，删去了一些不必要include的头文件

- 2023-1-7 初步实现定位算法，后期要改进（stop_flag的应用，位置的精确度，定位的速度有待提升）

- 2023-1-9 提高定位速度和精度，并且实现了连续不同坐标的定位（keil里面要将Optimization设置为Level-0，不然move2position会进入死循环，具体原因要看汇编卡住的地方），下一步争取实现LCD屏幕显示。

- 2023-1-12 

  1. 增加了对LCD显示的支持

  2. 已知问题，芯片上电后使用按键复位，芯片不会再进入串口接收中断

     解决办法，重新上电
  
- 2023-1-16 实现openmv读取二维码获得顺序并前往指定位置

- 2023-1-17 完善了OpenMV和STM32的通讯机制，实现了OpenMV的颜色识别（单独识别上层和下层）并传给STM32，目前已知的最大问题，OpenMV识别受光线影响大（或者阈值采集地不准确）。很难才识别出来，下一步计划先完善舵机代码

- 2023-1-19

  舵机 

  1，顺时针，减小

  2，顺时针，减小

  3，顺时针，增大
  
- 2023-1-25

  OpenMv会在扫描二维码之后先向STM32发送颜色顺序，以便后来在半成品区和粗加工区放置色块（通讯未测试，但是移动位置和代码框架已经写好）

- 2023-2-20

  **huart1分配给电机，huart2分配给OpenMV，huart3分配给OPS，huart4分配给舵机**

- 2023-2-21
  
  1，**<u>舵机的RS232 rx 接 rx, tx 接 tx</u>**
  
  2，完成机械臂Grab的初步框架，大致测量各个动作的起始位置和结束位置，大致写出整个流程的运行逻辑；
  
  3，遗留问题：1.设置舵机速度为逆时针（反方向）时，不能输入如”-2“这样的负数，应该填入什么待解决

- 2023-2-24

  1，舵机控制板连接单片机时电平选择**5V**，用上位机调试时选择**3V3**
  
  2，PWM舵机：采用通用定时器TIM2和TIM3，四通道输出，选择PWM通用模式，对应引脚：
  
   ![image-20230224154348515](https://mitcher-1316637614.cos.ap-nanjing.myqcloud.com/test/image-20230224154348515.png)
  
   ![image-20230224154620660](https://mitcher-1316637614.cos.ap-nanjing.myqcloud.com/test/image-20230224154620660.png)
  
  3，重新上电和Reset还是有区别的。今天发现重新上电之后没有跑初始化的程序（具体表现是SMS舵机没有运行)，不知道属不属于bug。
  
  4，进度：完成了PWM舵机的驱动，并且测定了SMS舵机初始化的参数。
  
  5，添加对GitHub仓库的镜像支持。
  
  
  
  - 2023-3-6
  
  1. 放置物料（上层）的基本流程已写完，但由于硬件装配不牢固原因无法精确动作，需要重新设计或装配硬件
  2. 控制机械臂的pwm舵机计划改成飞特舵机（问题：PWM舵机角度的取值范围？）
  3. reset后舵机仍旧可以回到初试姿态重新开始执行程序。
  
- 2023-3-9

  1. 控制底下圆盘的舵机设为编号**4**。

  2. 各舵机的调参角度范围：（一定不能超过这些范围，尤其是高亮的！）

     1）1号舵机：==1300==~2600；

     2）2号舵机：1500~==2600==；

     3）3号舵机：1600~3000；

     4）4号舵机：800~3700；

  3. 特别注意4号舵机，在机械臂还未展开时，若云台转至正中间位置（即机械臂指向正前方），机械臂会与物料产生干涉。建议将初始位置设为斜的角度，且保证机械臂回收时避免将机械臂移到正前方位置。