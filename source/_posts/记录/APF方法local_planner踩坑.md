---
title: APF方法的local planner 
description: 人工势场法（Artificial Potential Field）在全局规划与局部规划的应用
date: 2020-09-14 15:19:00
author: zhangyuanes
categories: 记录
mathjax: ture
tags:
  - SLAM
  - 人工势场法
---

## 人工势场法（Artificial Potential Field）

移动机器人在给定初始点和最终点的情况下，在不与障碍物发生碰撞的情况下执行由自动路径规划模块规划出的路径到达最终点。

在实际情况下，部分或全部环境未知，路径规划通常使用基于行为的方法（Behavior based methods）和使用智能控制技术的方法（methods using Intelligent Control Techniques）。

已知环境中的路径规划包括生成供机器人遵循的路径，前提是该环境的整体（静态或动态）都是已知的。在这种情况下，方法可以被大致分为两类：最小化代价方法，势场法。最小化代价方法中一个很典型的算法就是的启发式A*算法。

人工势场法是势场法中避障规划路径的常见方法。人工势场法在障碍物周围使用斥力场来使得机器人离开（不碰撞），目标点周围使用吸引力场来吸引机器人。因此，机器人受到的总力等于总场势梯度的负值（矢量）。这样的合力驱动机器人朝着目标点的位置下降，直到其到达目标点或者停止（可能陷入局部最小，类似于梯度下降）。

人工势场法的主要问题以下几点：

1. 距离很近的障碍物之间没有通道（不能通过）；
2. 在狭小环境下，机器人可能陷入平衡位置，并且会振荡或以闭环运行；
3. GNRON（Goals nonreachable with obstacle near by）无法到达障碍物附近的目标。


## 手动实现APF

### 思路

在激光的回调函数中调用，拿到Lansecan的数据，数据结构如下：

```bash
sensor_msgs/LaserScan
angle_min (float32) scan 的开始角度 [弧度]
angle_max (float32) scan 的结束角度 [弧度]
angle_increment (float32) 测量的角度间的距离 [弧度]
time_increment (float32) 测量间的时间 [秒] – 如果扫描仪在移动,这将用于插入 3D 点的位置
scan_time (float32) 扫描间的时间 [秒]
range_min (float32) 最小的测量距离 [米]
range_max (float32) 最大的测量距离 [米]
```

在world中拿到goal（目标点）和机器人所在位置坐标，分别为goal_（x，y），pose_（x，y）。然后计算goal与pose的欧式距离r，再使用atan2（dy，dx）计算出目标点相对当前位置的角度theta_goal，单位 [弧度]，其中dy = goal_（y）-pose_（y），dx同理。这里theta_goal是在世界坐标系中的，而非机器人坐标系。

```c++
pose_x = goal_.pose.position.x - pose_.pose.position.x;
pose_y = goal_.pose.position.y - pose_.pose.position.y;
float theta_goal = atan2(pose_y, pose_x);
float r = sqrt(pow(pose_x, 2) + pow(pose_y, 2));
```

### 引力

引力场计算公式：

$$
U_{att}(q) = \frac12\xi r^2 
$$

引力场下的引力计算为：

$$
F_{att} = -\Delta U_{att}(q) = \xi r
$$

这里将系数设置为GOAL_FORCE（default=1），需要进行世界坐标系的xy分解：

```c++
//常量定义
#define PI 3.14
#define GOAL_FORCE 1	  // 目标吸引力因子
...
pose_x = goal_.pose.position.x - pose_.pose.position.x;
pose_y = goal_.pose.position.y - pose_.pose.position.y;
float theta_goal = atan2(pose_y, pose_x);
float r = sqrt(pow(pose_x, 2) + pow(pose_y, 2));
float goal_force = GOAL_FORCE * r;
float goal_x = goal_force * cos(theta_goal);
float goal_y = goal_force * sin(theta_goal);
```

这里实现的上是冗余的，所以可以进行简化，直接写成引力计算的公式，但是后面需要可视化引力，这里就保留较为复杂的写法。

### 斥力

斥力场计算公式：

$$
U_{rep}(q) = \frac12 \eta \left( \frac{1}{\rho (q,q_{obs})} - \frac1{\rho_0} \right)^2 \rho ^n(q,q_{goal}), \rho (q,q_{obs}) \leq \rho_0
$$

$$
U_{rep}(q) = 0, \rho (q,q_{obs}) > \rho_0
$$

这里需要设置两个系数，参数n和阈值p0,在原有的斥力场中加上目标点和机器人距离的影响，来缓解GNRON问题。参数n一般设置为2，阈值p0是障碍物的影响半径，如果机器人距离障碍物一定距离，即使可以看见障碍物，也对机器人没有影响。

在斥力影响不为0时刻的斥力计算公式：

$$
F_{rep}(q) = -\Delta U_{rep}(q) = F_{rep1}n_{OR} + F_{req2}n_{RG} , \rho (q,q_{obs}) \leq \rho_0
$$

$$
F_{rep1}=\eta \left( \frac1{\rho (q,q_{obs})} - \frac1{\rho_0} \right) \frac{\rho ^n(q,q_{goal})}{\rho (q,q_{obs})},F_{req2}=\frac{n}{2}\eta \left( \frac1{\rho (q,q_{obs})} - \frac1{\rho_0} \right)^2\rho ^{n-1}(q,q_{goal})
$$

这里对标在激光中的实现还是有些问题，所以采用了一种另一种处理方式：

1. 遍历激光数据，找到距离机器人最近的障碍物距离 temp_min，以及其对应的range[i]的下标 temp_min_i。
2. 然后计算这个最近点对机器人的斥力。采用传统斥力公式。

```c++
//常量定义
#define MAX_RANGE 0.3 //机器人探测半径，也可以理解为障碍物影响距离
#define ROBOT_RADIUS 0.10 //机器人半径，默认机器人为圆形
#define REPULSE_FORCE 0.1  // 斥力因子
...
float temp_min = MAX_RANGE; //获得与机器人最近的障碍物距离来控制速度减小
int temp_min_i = -1; //对应最小距离的range下标
// 计算激光探测到的力（障碍物的斥力），分解到x，y方向（世界坐标系）
float repulsion_resultant_x = 0;
float repulsion_resultant_y = 0;
// laser_msg_这里是激光数据的数据结构，定义为sensor_msgs::LaserScan
int total_points = (laser_msg_.angle_max - laser_msg_.angle_min) / laser_msg_.angle_increment;
for (int i = 0; i < total_points; i++)
{
  if (laser_msg_.ranges[i] > MAX_RANGE || laser_msg_.ranges[i] == 0)
    continue;
  if (laser_msg_.ranges[i] > ROBOT_RADIUS)
  {
    float r_laser = laser_msg_.ranges[i] - ROBOT_RADIUS;
    if (r_laser < temp_min) {
      temp_min = r_laser;
      temp_min_i = i;
    }
  }
}
// 如果存在最近障碍物，将最近障碍物角度 laser_radian_global 转换到世界坐标系：先计算机器人坐标系坐标偏转 laser_radian，加上yaw角度（这里yaw角度放在了位姿的z轴中，因为机器人不存在高度上的移动，所以简化和充分利用pose数据结构将yaw放在了 pose_.pose.position.z中）
// 之后使用斥力公式计算，这里为了表示方向相反而乘上了负号
if (temp_min_i != -1) {
  float laser_radian = laser_msg_.angle_min + laser_msg_.angle_increment * temp_min_i;
  float laser_radian_global = laser_radian + pose_.pose.position.z;
  repulsion_resultant_x = -REPULSE_FORCE * cos(laser_radian_global) / pow(temp_min, 2);
  repulsion_resultant_y = -REPULSE_FORCE * sin(laser_radian_global) / pow(temp_min, 2);
}
```

### 合力

力的合成高中物理，无需赘述。合成之后如果合力的方向与目标引力的方向一致，可能落入局部最小，需要给予扰动力让其逃逸。

```c++
// 计算在x，y轴方向上的合力：斥合力+吸引力, attention: *r
float force_x = repulsion_resultant_x * r + goal_x;
float force_y = repulsion_resultant_y * r + goal_y;
// x，y分量合成合力
// force = sqrt(pow(force_x, 2) + pow(force_y, 2));
theta = atan2(force_y, force_x);
if(theta_goal == theta){
  theta += 0.1;
}
```

此外机器人移动发布的消息类型为cmd_vel需要指定线速度和角速度，所以需要进行合力到速度的映射。角速度需要优先考虑，因为机器人的旋转是其改变方向避开障碍物的第一步，这里就是计算出合力角度 theta 和 yaw的差值角度，假设时间间隔dt = 1来将其直接设置为角速度 angular.z；线速度如果变动不光滑在初步实现demo的时候也不用太过在意，直接写死简化处理方式（注释代码段，这个地方其实也有bug，因为速度的线性减小会逐步累加而导致最后速度很小，需要额外增加恢复函数）。

```c++
// 角度变化并放缩到-PI到PI之间
float radian_speed = theta - pose_.pose.position.z;
if (radian_speed > PI) {
  radian_speed = radian_speed - 2*PI;
}
if(radian_speed < -PI){
  radian_speed = 2*PI + radian_speed;
}
geometry_msgs::Twist vel_msg;
	// float cur_vx = 0.5;
	// float new_vx = cur_vx * temp_min / MAX_RANGE; // 越靠近物体速度越小
	vel_msg.linear.x = 0.35;
	vel_msg.linear.y = 0;
	vel_msg.linear.z = 0;
	vel_msg.angular.x = 0;
	vel_msg.angular.y = 0;
	vel_msg.angular.z = radian_speed; // 角度充当角速度，dt=1
	vel_pub_.publish(vel_msg);
```

### 遇到的问题

在初步需要实现代码到系统中的时候遇到了许多问题和坑：

- 不需要发布点云——最开始的时候将激光数据发布到点云，后来发现这个回调只需要计算合力大小与方向即可，不需要发布点云，删去这个部分代码
- atan2方法的参数为y，x——注意这里的顺序是先y后x，我不小心写成了atan2（x，y）导致角度计算不正确，花了好大力气一步步审查代码才发现，函数使用请细心；
- 计算出theta为世界坐标系中的弧度值——因为LaserScan中的angle_min与angle_max都是在世界坐标系下的弧度值，且已知angle_increment可以直接计算，但是不能直接使用这个做x，y分解的角度，因为引力是分解到世界坐标系中的，所以斥力以应该如此，需要在theta与yaw做运算和之后才行
- 启发式算法以计算角度差值代替角速度值
- 线速度与角速度的平滑函数很复杂，之前尝试使用加速度与角加速度去实现速度映射，发现不可行，机器人的运动无法预测。
- 善用可视化——在最开始遇到机器人的运动无法预测没有想到可视化，只是将数据打印了出来，然后使用rqt工具观察，发现角速度的变化是正常的，而合力很小，斥力很大，大的离谱，这时候合力对机器人的运动起不到影响作用，斥力直接影响合力方向，后面查看代码时候发现是这一段：

```c++
for (int i = 0; i < total_points; i++){
			if(scan_filtered.ranges[i] > MAX_RANGE || scan_filtered.ranges[i] == 0)	
				continue;
			if(scan_filtered.ranges[i] > ROBOT_RADIUS){
				float r_laser = scan_filtered.ranges[i] - ROBOT_RADIUS;
				if(r_laser >0){
					repulsion_resultant_x += -REPULSE_FORCE * cos(scan_filtered.angle_min + scan_filtered.angle_increment * i) / pow(r_laser, 2);
					repulsion_resultant_y += -REPULSE_FORCE * sin(scan_filtered.angle_min + scan_filtered.angle_increment * i) / pow(r_laser, 2);
				}
			}
		}
```

上一段代码是有几个错误的，包括之前说的计算出theta为世界坐标系中的弧度值，这里使用机器人坐标系的角度分解是错误的。但是我想说的重点在斥力计算的部分：这里代码含义为只要在机器人探测范围内有效的障碍物的粒子点都参与斥力计算，而且斥力是加和（+=）的，导致在机器人距离物体很近的时候，粒子很多，每个粒子都提供斥力，累和数值很大，所以后之后绝得将其修改为了一个粒子的影响。
这个问题是在进行引力、斥力和合力的可视化之后才发现的，可视化的部分代码如下：将力的数据处理成PoseStamped格式进行发布，只显示方向，大小从topic输出的数值来看。pose中最重要的方向orientation，将theta转化为四元数输出，同理对引力theta_goal、斥力atan2(repulsion_resultant_y, repulsion_resultant_x)也是一样的做法。

```c++
std_msgs::Float32 force_msg;
force_msg.data = force_x;
force_x_pub_.publish(force_msg);
force_msg.data = force_y;
force_y_pub_.publish(force_msg);
force_msg.data = radian_speed;
w_pub_.publish(force_msg);
geometry_msgs::PoseStamped force_dir_msg;
force_dir_msg.header.frame_id = "map";
force_dir_msg.header.stamp = ros::Time::now();
force_dir_msg.pose.position.x = pose_.pose.position.x;
force_dir_msg.pose.position.y = pose_.pose.position.y;
force_dir_msg.pose.orientation = tf::createQuaternionMsgFromRollPitchYaw(0, 0, theta);
force_direction_pub_.publish(force_dir_msg);
force_dir_msg.pose.orientation = tf::createQuaternionMsgFromRollPitchYaw(0, 0, theta_goal);
att_force_direction_pub_.publish(force_dir_msg);
// 斥力有才发布
if (repulsion_resultant_y != 0 || repulsion_resultant_x != 0) {
  force_dir_msg.pose.orientation = tf::createQuaternionMsgFromRollPitchYaw(0, 0, atan2(repulsion_resultant_y, repulsion_resultant_x));
  rep_force_direction_pub_.publish(force_dir_msg);
}
```

在rviz中展示的效果如下：
![apf-rviz](/medias/apf-rviz.jpg)